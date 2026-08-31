# API : Python SDK

{% hint style="info" %}
**Letar du efter den fullständiga API?** Den här sidan är en praktisk handledning. Alla offentliga klasser, metoder, exakta signaturer och exempel som går att kopiera och klistra in finns i [SDK-referensen](reference/sdk-reference.md), som är optimerad för AI-assistenter.**Arbetar du med en AI-assistent?** Klistra in denna URL i chatten så att den får den fullständiga, aktuella Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Varje sida i denna handbok finns tillgänglig som rå markdown under dess småbokstavs-slug + `.md`, och hela handboken är indexerad på `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` på PyPI) styr alla funktioner som skrivbordsappen kan utföra från Python: batchbearbetning av bilder, styrning av LATTICE-kameror och -arrayer i realtid, DAQ-sessioner med ljussensorer samt automatisering av sparade projekt. Det är ett tunt lager över samma lokala backend som GUI och CLI använder (HTTP på `127.0.0.1:5000`), så beteendet är identiskt på alla tre gränssnitten.

## Installation

Installationen sker i två steg: först Chloros-paketet för skrivbordet (det tillhandahåller backend för bearbetning och hårdvaruruntimes), sedan Python-paketet.

**Steg 1 — Installera Chloros.** Windows: kör installationsprogrammet för skrivbordet (standardväg `C:\Program Files\MAPIR\Chloros\`) från sidan [Nedladdning](download.md). Linux: installera paketet `.deb` ([Linux Installation](linux/linux-installation.md)).**Steg 2 — Installera SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Du behöver kanske inte ens pip: varje installationsprogram levereras med ett matchande SDK-wheel. Installationsprogrammet Windows installerar det automatiskt i ditt system Python; Linux `.deb` placerar den i `/usr/lib/chloros/sdk/` och visar det exakta `pip install --user`-kommandot. PyPI uppdateras vid release-byggnader, så `pip install chloros-sdk` stämmer överens med den senaste stabila versionen.

**Steg 3 — Logga in en gång per maskin:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Inloggningsuppgifterna cachelagras i `~/.chloros/` (båda plattformarna). På Windows kan du på motsvarande sätt logga in via fliken Användar<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">er i skrivbordsappen. SDK kräver ett betalt Chloros+-abonnemang — se [Licenskrav](#license-requirement) nedan.

| Krav | Detaljer |
| --- | --- |
| **Chloros installerat** | Windows: installationsprogram för skrivbordet; Linux: `.deb`-paket (innehåller backend-binären) |
| **Python** | 3.7 eller senare (utvecklat/testat på 3.10) |
| **Operativsystem** | Windows 10/11 64-bitars, Ubuntu 22.04 LTS eller nyare, eller NVIDIA Jetson (JetPack 6) |
| **Licens** | Aktivt Chloros+-inloggning, valfri betald nivå (Copper eller högre) |

## 60 sekunders framgång

Ett enda anrop skapar ett projekt, importerar en mapp, konfigurerar bearbetningen och kör pipelinen – samtidigt som backend-delen startas automatiskt om den inte redan körs:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(På Linux, använd Linux-sökvägar: `/home/user/drone_images/flight001`. SDK fungerar på samma sätt på båda plattformarna.)

Bearbetar du en LATTICE-insamlingsmapp? Använd det LATTICE-anpassade verktyget – det tillämpar rätt standardinställningar (ingen detektering av panelmål, standard-debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` – fullständig kontroll över bearbetningskedjan

För allt som kräver mer än en enda rad, använd `ChlorosLocal`. Det startar backend vid första användningen (`auto_start_backend=True`), skapar och konfigurerar projekt, övervakar framstegen och returnerar en sammanfattning efter körningen.

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

{% hint style="info" %}
Behåll standardvärdet `http://127.0.0.1:5000` istället för att ersätta det med `localhost` — på Windows `localhost` omvandlas först till `::1` och tar cirka 2 sekunder per förfrågan mot backend som endast stöder IPv4.
{% endhint %}

Använd det som en kontextmanager för garanterad upprensning:

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` accepterar följande nyckelord: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` och `custom_settings`. De viktigaste värdena:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

De LATTICE-specifika reglagen (`input_level`, `radiometric_output`, samt familjen `array_alignment*`) dokumenteras med sina fullständiga värdetabeller i [SDK-referensen](reference/sdk-reference.md#supported-values).

### Övervaka förloppet

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Läsa sammanfattningen efter körningen – och upptäcka tomma körningar

När körningen är klar bifogar `process()` backendens bearbetningssammanfattning som `result["summary"]`. Varje post i `summary["hints"]` är en fullständig mening som förklarar allt som är värt att notera — till exempel varför en körning gav noll utdata — och varje ledtråd skickas också ut på nytt som en Python `UserWarning`, så tomma körningar diagnostiseras automatiskt även om du aldrig granskar ordlistan:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` genereras inte när en körning inte producerar några bilder.** Detta är det enda stället där SDK och CLI medvetet skiljer sig åt: `chloros-cli process` behandlar ”produkter begärdes, inga skrevs ut” som ett fel och avslutas med ett värde som inte är noll, medan SDK avslutas normalt och rapporterar tillståndet via `summary` / hints. Om din pipeline ska avbrytas vid en tom körning, kontrollera detta själv – granska `summary` (eller räkna filerna i projektmappen) istället för att förlita dig på ett undantag.
{% endhint %}

## Smart Connect — live-hårdvara

Tre hjälpfunktioner öppnar beständiga sessioner i backendens hårdvarupool – samma pool som GUI:n använder, så att SDK-skript kan samexistera med skrivbordsappen utan att konkurrera om seriella portar eller nätverksbandbredd. Alla tre startar automatiskt en lokal backend om ingen är igång.

### Enskild LATTICE-kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Synkroniserad uppsättning — `connect_array`

`connect_array` är den rekommenderade startpunkten för riggar med flera kameror. Den kör samma smart-prep-flöde som GUI: nätverksanalys, automatisk val av synkroniseringsnivå, PTP-tidssynkronisering, val av pixelformat per kamera, AE-seeding och aktivering av GPIO-trigger. Den **första serien är master** (den avfyrar hårdvarutriggerpulsen); resten är slavar.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Lägg till `smart=True` till valfri arrayinspelning för att vänta på att den automatiska exponeringen ska stabiliseras på alla kameror innan utlösning sker. För inspelningslägen (Enstaka / Kontinuerlig / Intervall / Snabbaste), inspelningsenheter, burst-to-video och matrisjustering, se [SDK-referens](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ-ljussensor — `connect_daq_sensor`

Utan argument detekterar `connect_daq_sensor()` automatiskt överföringssättet (prioritet: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Varje ram innehåller 135-punkts `spectrum` (W/m²/nm vid kalibrering), en `is_saturated`-flagga och CIE `x`, `y`, `z`. För att ange en specifik sensor eller transport – det pålitliga valet på värddatorer med flera nätverksgränssnitt, där Ethernet-autodetektering kan missa en fungerande DAQ-E vid första försöket – skickar man en explicit hint:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Observera att profiler för kapacitetskorrigering (`cap_id`) **inte** är en SDK-inställning – välj dem istället via `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` istället.

### Sparade projekt — `open_project`

Ett sparat Chloros-projekt behåller sin anslutna hårdvara (`cameras.json` + `sensors.json` tillsammans med `project.json`), och `chloros_sdk.open_project(path)` kan återansluta allt på en gång och styra inspelningar efter enhetsnamn. Se [Projektautomatisering](reference/sdk-reference.md#project-automation--chlorosproject) i referensen.

## Vad en installation som endast använder pip ger

Kontrollera tillgänglighetsflaggorna på modulnivå innan du använder hårdvaruytor:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

På en värd med **endast** `pip install chloros-sdk` och utan Chloros-skrivbordspaket:

* `ChlorosLocal`, `process_folder` och `process_lattice_capture` fungerar **inte** — de behöver den bakomliggande binärfilen som medföljer i skrivbordsinstallationsprogrammet.
* Smart-connect-hjälpprogrammen (`connect_camera`, `connect_array`, `connect_daq_sensor`) är rena HTTP-klienter, så de fungerar mot en backend på en annan maskin — men de medföljande backendarna binder endast till loopback, så du måste vidarebefordra porten själv (t.ex. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) och skicka vidare `backend_url="http://127.0.0.1:5000"` tillsammans med `auto_start_backend=False`. Se [Remote-Backend Mode](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* De direkt-hårdvarubaserade LATTICE-klasserna (`LatticeCamera`, `CameraPool`, …) kan importeras, men kräver Arena SDK-körmiljön från skrivbordspaketet — utan den är `CAMERA_AVAILABLE` `False`.
* `daq_sdk` (klasserna för direkt DAQ) ingår i desktop-installationen, men inte i PyPI-paketet, så `DAQ_AVAILABLE` är `False` på en värd som endast använder pip — styr istället DAQ-sensorer via `connect_daq_sensor()` mot en (tunnlad) backend.

## Licenskrav

Åtkomst till SDK kräver en aktiv Chloros+-inloggning på någon av de betalda nivåerna — **Copper eller högre**(Copper / Bronze / Silver / Gold); den kostnadsfria nivån Iron har ingen åtkomst till SDK/CLI. Kontrollen sker**på serversidan**: varje SDK-förfrågan måste innehålla både en aktiv session och en betald nivå, annars returnerar backend-systemet `403` / `PLAN_UPGRADE_REQUIRED` (genererad som `ChlorosLicenseError` av `ChlorosLocal`, och som `ChlorosConnectError` av `connect_*`-hjälpfunktionerna). En utloggad anropare får istället `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) istället – att köra `chloros-cli login` på nytt åtgärdar det första fallet men inte det andra.

Offlineanvändning fungerar inom planens respitperiod: nivån läses från cacheminnet för servervalidering (5 minuter) eller från cacheminnet för signerade, maskinbundna licenser (30 dagar för månadsplaner; till prenumerationens utgångsdatum för årsplaner). När respitperioden löper ut övergår planen till gratisversionen och åtkomsten via SDK stoppas tills enheten når servern en gång. `chloros-cli status` förblir tillgängligt på gratisnivån så att orsaken alltid syns. Se [Chloros+ Inloggning](chloros+-login.md).

## Undantag

Fånga basklassen för att hantera ”allt som gick fel med Chloros”:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

Alla pipeline-undantag (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) härstammar från `ChlorosError`. En sak att tänka på: `ChlorosConnectError` — genereras endast av `connect_camera` / `connect_array` / `connect_daq_sensor` — härstammar från vanliga `Exception`, **inte** från `ChlorosError`, så `except ChlorosError` kommer inte att fånga upp det. Den fullständiga hierarkin finns i [SDK-referensen](reference/sdk-reference.md#exceptions).

## Se även

* [SDK-referens](reference/sdk-reference.md) — den fullständiga API-ytan, optimerad för AI-assistenter.
* [CLI-referens](reference/cli-reference.md) — varje CLI-underkommando motsvarar ett SDK-anrop.
* [Ladda ner](download.md) — installationsprogram för Windows och Linux.
