# API: Python SDK

{% hint style="info" %}
**Letar du efter den fullständiga API:n?** Den här sidan är en praktisk handledning. Alla offentliga klasser, metoder, exakta signaturer och exempel som går att kopiera och klistra in finns i [SDK-referensen](reference/sdk-reference.md), som är optimerad för AI-assistenter.**Arbetar du med en AI-assistent?** Klistra in den här URLn i chatten så att den får tillgång till den fullständiga, aktuella Chloros 1.3.0-API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Varje sida i denna handbok finns tillgänglig som rå markdown under dess slug med små bokstäver + `.md`, och hela handboken är indexerad på `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` på PyPI) styr allt som desktop-appen kan göra från Python: batchbearbetning av bilder, styrning av LATTICE-kameror och -arrayer i realtid, DAQ-sessioner med ljussensorer samt automatisering av sparade projekt. Det är ett tunt lager över samma lokala backend som GUI och CLI använder (HTTP på `127.0.0.1:5000`), så beteendet är identiskt på alla tre gränssnitten.

## Installation

SDK:et levereras med Chloros-paketet för skrivbordet, som även innehåller backend-komponenterna för bildbearbetning och hårdvarukörmiljöer, och versionen stämmer alltid överens med den.

**Steg 1 — Installera Chloros.** Windows: kör installationsprogrammet för skrivbordet (standardväg `C:\Program Files\Chloros\`) från [Nedladdningssidan](download.md). Linux: installera paketet `.deb` med `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Linux-installation](linux/linux-installation.md)).**Steg 2 — Kontrollera SDK** (Python 3.7+). Vanligtvis behöver inget installeras:

* **Windows:** installationsprogrammet installerar det medföljande SDK-wheelet i ditt systems Python (om ingen Python är installerad hoppar det över detta steg).
* **Linux:** `.deb` installerar SDK:et för systemets `python3` i `/usr/lib/python3/dist-packages`. Kravpaketen, `python3-numpy` och `python3-requests`, rekommenderas av paketet: `apt install` inkluderar dem, medan `dpkg -i` inte gör det.
* **En virtualenv** ser inte systemets paket; installera det medföljande wheel-paketet i den: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` på Linux, eller `chloros_sdk-*.whl` i mappen `sdk` i Windows-installationsmappen (`C:\Program Files\Chloros\sdk\` som standard).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

På en dator utan Chloros-paket, vars SDK-drivrutin styr en backend på en annan dator (se [Vad en ren pip-installation ger](#what-a-pip-only-install-gets)), installera det från PyPI: `pip install chloros-sdk`. PyPI uppdateras vid varje release, så det stämmer överens med den senaste stabila versionen.

**Steg 3 — Logga in en gång per dator:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Inloggningsuppgifterna cachelagras i `~/.chloros/` (båda plattformarna). I Windows kan du istället logga in via fliken Användare <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> i skrivbordsappen. SDK kräver ett betalt Chloros+-abonnemang — se [Licenskrav](#license-requirement) nedan.

| Krav | Detaljer |
| --- | --- |
| **Chloros installerat** | Windows: installationsprogram för skrivbordet; Linux: `.deb`-paket (innehåller backend-binären och SDK) |
| **Python** | 3.7 eller högre (utvecklat/testat på 3.10) |
| **Operativsystem** | Windows 10/11 64-bitars, Ubuntu 22.04 LTS eller nyare, eller NVIDIA Jetson (JetPack 6) |
| **Licens** | Aktivt Chloros+-konto, valfri betald nivå (Copper eller högre) |

## Klart på 60 sekunder

Ett enda anrop skapar ett projekt, importerar en mapp, konfigurerar bearbetningen och kör pipelinen – samtidigt som backend-programmet startas automatiskt om det inte redan körs:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(På Linux använder du Linux-sökvägar: `/home/user/drone_images/flight001`. SDK:et fungerar på samma sätt på båda plattformarna.)

Bearbetar du en LATTICE-inspelningsmapp? Använd den LATTICE-anpassade wrappern – den tillämpar rätt standardinställningar (ingen detektering av panelmål, standard-debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` – fullständig kontroll över pipelinen

För allt som kräver mer än en enda rad, använd `ChlorosLocal`. Det använder en backend som redan körs (den från skrivbordsappen), eller startar den installerade vid första användningen (`auto_start_backend=True`; i Windows från mappen `backend` i Chloros-installationsmappen, `C:\Program Files\Chloros\backend\` som standard), skapar och konfigurerar projekt, övervakar framsteg och returnerar en sammanfattning efter körningen.

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
Behåll standardvärdet `http://127.0.0.1:5000` istället för att ersätta det med `localhost` — i Windows omvandlas `localhost` först till `::1` och tar cirka 2 sekunder per begäran mot backend som endast stöder IPv4.
{% endhint %}

Använd den som en kontextmanager. När du lämnar blocket avslutas denna instans användning av backend-servern, inte backend-servern själv: den fortsätter att köras för nästa skript och stängs av automatiskt efter 30 minuters inaktivitet. `shutdown_backend()` avslutar den instans som startats.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` accepterar följande nyckelord: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` och `custom_settings`. De viktigaste värdena:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

De LATTICE-specifika reglagen (`input_level`, `radiometric_output`, familjen `array_alignment*`) dokumenteras med sina fullständiga värdetabeller i [SDK-referensen](reference/sdk-reference.md#supported-values).

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

När körningen är klar bifogar `process()` backendens bearbetningssammanfattning som `result["summary"]`. Varje post i `summary["hints"]` är en fullständig mening som förklarar allt som är värt att notera — till exempel varför en körning inte gav något resultat — och varje ledtråd skickas också ut på nytt som en Python-kod (`UserWarning`), så tomma körningar diagnostiseras automatiskt även om du aldrig granskar ordlistan:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` utlöses inte när en körning inte genererar några bilder.** Detta är det enda stället där SDK:t och CLI:t medvetet skiljer sig åt: `chloros-cli process` behandlar ”produkter begärdes, inga skrevs” som ett fel och avslutas med ett värde som inte är noll, medan SDK:t återgår normalt och rapporterar tillståndet via `summary` / tips. Om din pipeline ska avbrytas vid en tom körning, kontrollera detta själv – granska `summary` (eller räkna filerna i projektmappen) istället för att förlita dig på ett undantag.
{% endhint %}

## Smart Connect — live-hårdvara

Tre hjälpfunktioner öppnar beständiga sessioner i backendens hårdvarupool — samma pool som CLI, andra skript och GUI:ns kameraflikar använder, så att SDK-skript kan samexistera med dem utan att konkurrera om seriella portar eller nätverksbandbredd. Alla tre startar automatiskt en lokal backend om ingen är igång. GUI:ns DAQ-panel ingår inte i poolen: stäng en ljussensor i GUI:n innan du öppnar den från ett skript.

### Enstaka LATTICE-kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Relativa sökvägar pekar på den plats där ditt skript körs (ändrat i 1.3.0).** Med en lokal backend, en relativ `output_dir` (inklusive standardvägen `"output"`) — och de andra sökvägarna som backenden öppnar (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — omvandlas utifrån ditt skripts arbetskatalog, så att `cam.capture("output/")` skriver till `<current folder>/output/`. I version 1.2.0 tolkade en installerad backend dem i sin egen installationsmapp och gav felmeddelandet ”Åtkomst nekad”. Vid användning av en fjärrbackend skickas sökvägarna precis som de skrivits in.
{% endhint %}

### Synkroniserad matris — `connect_array`

`connect_array` är den rekommenderade startpunkten för riggar med flera kameror. Den kör samma smart-prep-flöde som GUI: nätverksanalys, automatisk val av synkroniseringsnivå, PTP-tidssynkronisering, val av pixelformat per kamera, AE-seeding och aktivering av GPIO-trigger. Den **första serienumreringen är mastern** (den avfyrar hårdvarutriggerpulsen); de övriga är slavar.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Lägg till `smart=True` till valfri array-inspelning för att vänta på att den automatiska exponeringen ska stabiliseras på alla kameror innan utlösning sker.

Nytt i version 1.3.0: `ArraySession` ger dig även synkroniserade bildrutor direkt, utan att skriva filer:

* `arr.grab()` returnerar en bildruta per kamera, alla från en utlösning, som numpy-matriser (`{serial: array}`); `.group_id` är den delade utlösaren och `.errors` namnger alla kameror som utelämnats.
* `arr.frame_stream(fps=5.0)` ger en sådan uppsättning per tick, för livebearbetning i din egen pipeline.
* `arr.continuous("output/")` kör backendens egen kontinuerliga inspelning (GUI:ns kontinuerliga läge) och returnerar en `ContinuousHandle` tillsammans med `status()`, `wait()` och `stop()` – använd detta istället för en inspelningsslinga när du behöver spara varje utlösning med hög frekvens.

För inspelningslägen (Enstaka / Kontinuerlig / Intervall / Snabbast), inspelningsenheter, burst-to-video och matrisjustering, se [SDK-referensen](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ-ljussensor — `connect_daq_sensor`

Utan argument detekterar `connect_daq_sensor()` automatiskt transportprotokollet (prioritet: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Varje ram innehåller 135-punkts `spectrum` (W/m²/nm vid kalibrering), en `is_saturated`-flagga samt CIE `x`, `y`, `z`. För att ange en specifik sensor eller transport, ange en explicit hint:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Observera att profiler för cap-korrigering (`cap_id`) **inte** är en SDK-inställning — välj dem istället via `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Sparade projekt — `open_project`

Ett sparat Chloros-projekt behåller sin anslutna hårdvara (`cameras.json` + `sensors.json` tillsammans med `project.json`), och `chloros_sdk.open_project(path)` kan återansluta sina kameror, matriser och ljussensorer på en gång och styra inspelningar efter enhetsnamn. Ljussensorerna återansluts via en körande Chloros-backend, med sina sparade inställningar och kapacitet. Se [Projektautomatisering](reference/sdk-reference.md#project-automation--chlorosproject) i referensen.

## Vad en installation med enbart pip ger

Kontrollera tillgänglighetsflaggorna på modulnivå innan du använder hårdvaruytor:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

På en värd med **endast** `pip install chloros-sdk` och utan Chloros-skrivbordspaket:

* `ChlorosLocal`, `process_folder` och `process_lattice_capture` fungerar **inte** – de behöver den bakomliggande binärfilen som medföljer i skrivbordsinstallationsprogrammet.
* Smart-connect-hjälpprogrammen (`connect_camera`, `connect_array`, `connect_daq_sensor`) är rena HTTP-klienter, så de fungerar mot en backend på en annan maskin — men de medföljande backend-programmen binder endast till loopback, så du måste själv vidarebefordra porten (t.ex. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) och skicka `backend_url="http://127.0.0.1:5000"` tillsammans med `auto_start_backend=False`. Se [Fjärr-backend-läge](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* LATTICE-klasserna för direkt hårdvaruanslutning (`LatticeCamera`, `CameraPool`, …) kan importeras, men kräver Arena SDK-körmiljön från skrivbordspaketet — utan den är `CAMERA_AVAILABLE` lika med `False`.
* DAQ-sensorer styrs endast via en (tunnlad) backend — `connect_daq_sensor()`, eller ett projekts `proj.sensors`; backenden öppnar sensorn på sin egen maskin.

## AI-assistenter — `chloros-mcp`

SDK:et innehåller `chloros-mcp`, en MCP-server som gör det möjligt för Claude Desktop, Claude Code, Cursor och andra MCP-klienter att styra Chloros: bearbeta bilder och köra LATTICE-kameror, arrayer och DAQ-ljussensorer. Dess verktyg täcker vad detta SDK gör. Det installeras tillsammans med SDK, så för Claude Code består installationen av en enda rad:

```bash
claude mcp add chloros -- chloros-mcp
```

Övriga klienter: [Använda Chloros med AI-assistenter](ai-assistants.md). Alternativ och alla verktyg: [MCP-server (AI-assistenter)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Licenskrav

För åtkomst till SDK krävs en aktiv Chloros+-inloggning på valfri betald nivå – **Copper eller högre**(Copper / Bronze / Silver / Gold); den kostnadsfria nivån Iron har ingen tillgång till SDK/CLI. Kontrollen sker**på serversidan**: varje SDK-förfrågan måste innehålla både en aktiv session och ett betalt abonnemang, annars returnerar backend-systemet `403` / `PLAN_UPGRADE_REQUIRED` (genererade som `ChlorosLicenseError` av `ChlorosLocal`, och som `ChlorosConnectError` av hjälpfunktionerna `connect_*`). En utloggad anropare får istället `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) istället — att köra `chloros-cli login` på nytt löser det första fallet men inte det andra.

Offlineanvändning fungerar inom planens respitperiod: nivån läses från cacheminnet för servervalidering (5 minuter) eller från cacheminnet för signerade, maskinbundna licenser (30 dagar för månadsplaner; till prenumerationens utgångsdatum för årsplaner). När respitperioden löper ut övergår planen till gratisnivån och SDK-åtkomsten avbryts tills datorn ansluter till servern en gång. `chloros-cli status` förblir tillgängligt på gratisnivån så att orsaken alltid är synlig. Se [Chloros+ inloggning](chloros+-login.md).

## Undantag

Fånga basklassen för att hantera ”allt som gått fel i Chloros”:

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

Alla undantag i pipelinen (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) härstammar från `ChlorosError`. En sak att tänka på: `ChlorosConnectError` — genereras endast av `connect_camera` / `connect_array` / `connect_daq_sensor` — härstammar från vanliga `Exception`, **inte** från `ChlorosError`, så `except ChlorosError` kommer inte att fånga upp det. Den fullständiga hierarkin finns i [SDK-referensen](reference/sdk-reference.md#exceptions).

## Se även

* [SDK-referensen](reference/sdk-reference.md) — den fullständiga API-ytan, optimerad för AI-assistenter.
* [CLI-referens](reference/cli-reference.md) — varje CLI-underkommando motsvarar ett SDK-anrop.
* [Använda Chloros med AI-assistenter](ai-assistants.md) — `chloros-mcp`-konfigurations- och promptrecept.
* [Nedladdning](download.md) — installationsprogram för Windows och Linux.
