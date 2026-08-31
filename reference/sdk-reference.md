# Chloros Python SDK Referens

**Version:**

1.2.0**Skapad:**2026-07-29 19:19 ·**Uppdaterad:** 2026-08-30**Paket:** `chloros-sdk` (PyPI)**Målgrupp:** Optimerad för användning av stora språkmodeller (LLM); läsbar för människor.**Omfattning:** Alla offentliga klasser, funktioner och hjälpfunktioner som exponeras av `import chloros_sdk`, med exempel som går att kopiera och klistra in och som täcker bildbehandling, styrning av enstaka kameror, synkroniserade arrayer, DAQ-sensorer och projektautomatisering.

Om du bara vill ha det viktigaste, gå till:
- [Installation och snabbstart](#installation)
- [Smart-Connect för LATTICE-kameror](#smart-connect-for-lattice-cameras)
- [DAQ-sensorsessioner](#daq-sensor-sessions)
- [Projektautomatisering](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Arkitekturen på 60 sekunder

SDKen är ett tunnPythonsskikt ovanpå backend-systemet Chloros (samma Flask-server som används av desktop-GUI:n och CLI). För automatisering importerar du `chloros_sdk` och anropar metoder på hög nivå; bakom kulisserna omvandlas varje anrop till en HTTP-förfrågan till den lokala backend-servern på port 5000 — `http://127.0.0.1:5000/api/...` (medvetet inte `localhost`, som först omdirigeras till `::1` på Windows och tar cirka 2 sekunder per begäran mot en backend som endast stöder IPv4). Backenden hanterar hårdvarupoolen — kameror, DAQ-sensorer, justeringsprofiler, bildbuffertar — så att SDK-skript kan samexistera med GUI:t utan att tävla om seriella portar eller nätverkskortets bandbredd.

Det finns tre gränssnitt som dukommer att använda:

1. **`ChlorosLocal` + fria funktioner** (`process_folder`, `process_lattice_capture`) — Bildbehandlingspipeline. Kör en hel mapp genom kalibrering / debayer / indexexport med ett enda anrop till Python.
2. **Smart-connect-hanterare** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Öppna en beständig backend-session för live-hårdvara. Samma ”smart-prep”-flöde som i GUI: nätverkssond, automatisk val av nivå, PTP, AE-seeding, GPIO-triggerkonfiguration.
3. **`ChlorosProject` / `open_project`** — Ladda ett sparat projekt (mapp med `cameras.json` + `sensors.json` + `project.json`), anslut allt på en gång och kör inspelningar med namngivna handtag.

Ytorna 1 och 2 **startar automatiskt en lokal backend** om ingen redan lyssnar (samma medföljande binärfil som GUI/CLI startar) — så ett enkelt skript fungerar från ett nytt skal utan att du behöver starta en backend först. Ange `auto_start_backend=False` för att avaktivera detta (t.ex. när du pekar på en fjärrbackend, som aldrig startas). Se [Automatisk start av backend](#backend-auto-start). Surface 3 beter sig annorlunda: `open_project()` tar inga `auto_start_backend`-parametrar, och `connect_all()` startar aldrig en back— den söker efter `http://127.0.0.1:5000` en gång och, om inget svarar, faller den tyst tillbaka till direkt (backend-fri) `lattice_sdk`-enhetsstyrning. Endast `proj.process()` och `stream(..., overlays=True)` skapar en `ChlorosLocal()` vid behov (vilket sker automatiskt vid start).

Alla tre är åtkomstbegränsade: kör `chloros-cli login` en gång på maskinen, eller logga in via skrivbordets grafiska gränssnitt. Anrop till SDK utan en giltig session genererar `ChlorosAuthenticationError`.

Krav:
- Python 3.7+ (enligt paketets specifikation; utvecklat/testat på 3.10)
- Chloros Desktop installerat lokalt (backend-binären medföljer i installationsprogrammet)
- Aktiv inloggning på Chloros+. Miniminivån för SDK / CLI är **Copper**eller högre (Copper / Bronze / Silver / Gold); den kostnadsfria**Iron**-nivån har ingen åtkomst till SDK / CLI. Detta tillämpas**på serversidan**: varje begäran med flaggan SDK / CLI måste innehålla både en aktiv session och ett betalt abonnemang, annars returnerar backend-systemet `403` med `error_code: PLAN_UPGRADE_REQUIRED` (visas som `ChlorosLicenseError` av `ChlorosLocal`, och som `ChlorosConnectError` av hjälpfunktionerna `connect_*`). En utloggad användare får istället `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) istället — de två är olika eftersom en ny körning av `chloros-cli login` åtgärdar den första men inte den andra.
- Offlineanvändning stöds inom planens respitperiod: nivån läses från cacheminnet för servervalidering (5 min) eller den signerade, maskinbundna licenscachen (30 dagar för månadsabonnemang, till abonnemangets utgångsdatum för årsabonnemang). När denna respitperiod löper ut övergår abonnemanget till gratisversionen och åtkomsten till SDK / CLI upphör tills maskinen kan nå servern en gång. `chloros-cli status` (`GET /api/license-status`) förblir tillgänglig på gratisnivån så att orsaken syns – det är den enda vägen SDK / CLI som är undantagen från nivåbegränsningen.
- Windows 10/11 64-bitars, **Ubuntu 22.04 LTS eller nyare**, eller Jetson (JetPack 6). Ubuntu 20.04 stöds**inte**: `.deb`:s beroenden härrör från vad backend länkar mot, inklusive `libc6 (>= 2.34)`, och Focal levereras med glibc 2.31.

---

## Installation

Python SDK är ett tunt Python-lager ovanpå backend-modulen Chloros. För allt utöver några få DAQ-arbetsflöden behöver du **det lokalt installerade Chloros-paketet** (Windows installer eller Linux `.deb`) — det är det som tillhandahåller backend-binären, Arena-SDK-runtime för LATTICE-kameror och kalibreringspaketen.

Senaste nedladdningar: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Steg 1 — Installera plattformspaketet Chloros

#### Windows (.exe)

1. Ladda ner `Chloros-Setup-x.y.z.exe` från nedladdningssidan.
2. Kör installationsprogrammet och följ guiden. Standardinstallationsvägen är `C:\Program Files\MAPIR\Chloros\`.
3. Starta Chloros minst en gång och logga in med ditt Chloros+-konto.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Steg 2 — Installera Python SDK

**Chloros-installationsprogrammet levereras med ett matchande SDK-wheel.** Varje Windows-installationsprogram och Linux .deb placerar en `chloros_sdk-X.Y.Z-py3-none-any.whl` på disken som exakt matchar GUI-/CLI-/backend-versionen. Du behöver inte hålla koll på PyPI för att hålla dig synkroniserad.

#### Windows

Installationsprogrammet kör automatiskt`pip install` mot det medföljande wheel-paketet med hjälp av ditt systemPythont (`py.exe`-startprogrammet föredras, faller tillbaka på `python -m pip`). Ingen åtgärd krävs — `import chloros_sdk` fungerar i din Python-miljö efter en lyckad installation. Om det inte finns någon Python på datorn hoppar installationsprogrammet tyst över detta steg och GUI + CLI fortsätter att fungera.

#### Linux (.deb)

.deb-paketet placerar wheel-filen i `/usr/lib/chloros/sdk/`. `postinst` visar det exakta kommandot — PEP 668-distributioner tillåter som standard inte globala pip-skrivningar, så vi utför ingen automatisk installation:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

För Jetson-installationer i air-gapped-miljöer sker detta helt offline – wheel-filen finns redan på disken.

#### Offentlig PyPI

För värdar som endast använder pip (inget Chloros-skrivbordspaket installerat; arbetsflöden med endast fjärrbackend eller DAQ):

```bash
pip install chloros-sdk
```

PyPI uppdateras vid installationer av release-versioner, så det publicerade wheel-paketet matchar den senaste stabila versionen. Utvecklingsversioner (t.ex. `1.1.4.dev1`) levereras endast via det medföljande installationsprogrammet.

#### Verifiera

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ prenumeration krävs.** Alla SDK-anrop kräver en aktiv Chloros+ inloggning. Kör `chloros-cli login user@example.com 'YourPassword'` en gång per maskin; inloggningsuppgifterna sparas i `~/.chloros/`.

### Behöver jag skrivbordspaketet?

Enbart pip-paketet är **inte** tillräckligt för de flesta arbetsflöden. Här är vad varje SDK-yta behöver:

| SDK-yta | Behöver Desktop-paketet? | Varför |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ja** | Startar backend-binären automatiskt på `/usr/lib/chloros/chloros-backend` (Linux) eller `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ja**(lokalt)**/ Nej**(fjärr) | Rena HTTP-klienter via backend. Lokal backend → skrivbordspaket krävs. Fjärrbackend → `backend_url=`**genom en tunnel** (se Läget för fjärrbackend — medföljande backends binder endast till loopback). |
| `ChlorosProject` / `open_project` | **Ja** | Kör sparade projekt via backenden. |
| Direkta LATTICE-klasser (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ja** | Kräver den inbyggda runtime-miljön för Arena-SDK, som ingår i desktop-paketet. Annars är `CAMERA_AVAILABLE` identiskt med `False` vid import. |
| Direkta DAQ-klasser (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Nej** | Ren Python via pyserial/bleak/zeroconf. En miljö som endast använder pip kan styra DAQ:er från början till slut. |

### Fjärr-backend-läge (värd som endast använder pip, via tunnel)

> **Den medföljande backenden är inte nåbar via LAN.** Produktions
> -versioner binder endast till loopback (båda loopback-familjerna) och avvisar kategoriskt det
> enda icke-loopback-läget (`CHLOROS_CLOUD_MODE`), så
> `backend_url="http://<lan-ip>:5000"` **kan inte fungera mot en installerad
> Chloros** — det mönstret har endast fungerat mot en source/dev-
> backend. För att driva en backend på en annan maskin, vidarebefordra dess loopback-
> port själv och peka SDK mot tunneln:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Headless-/CI-/robotikvärdar kan ha en maskin med fullständig skrivbordsinstallation som ”Chloros-server” och `pip install chloros-sdk` överallt annars — men transporten mellan dem sker via den ovan nämnda, användararrangerade tunneln, inte via en direkt LAN-URL

> **Känd begränsning — `ChlorosLocal` stöder inte enbart pip.** `ChlorosLocal(backend_url=BACKEND)` löser för närvarande en lokal backend-binärfil i sin konstruktor *innan* URLen undersöks, och genererar felet `ChlorosBackendError` (”Chloros-backend hittades inte…”) när inget skrivbordspaket är installerat — även om en fjärrbackend är tillgänglig. Endast smart-connect-gränssnittet ovan (`connect_camera` / `connect_array` / `connect_daq_sensor`, plus `analyze_array_network` och hjälpfunktionerna `list_*` / `discover_*`) fungerar från en värd med endast pip.

### Arbetsflöde enbart för DAQ (värd enbart med pip)

Om du endast behöver DAQ-sensorer och inte använder LATTICE-kameror eller bildbehandling är pip-paketet fristående:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Ingen backend, inget .deb-paket och ingen inloggning via Chloros+ krävs för direkt hårdvarubaserad DAQ-användning.

---

## Snabbstart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Index över toppnivåAPIer

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Bildbehandling — `ChlorosLocal`

Den övergripande pipeline-klassen. Startar backend vid första användningen, skapar och konfigurerar projekt, övervakar framsteg och returnerar sammanfattningar efter körning.

### Konstruktor

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

### Metoder

| Metod | Beskrivning |
| --- | --- |
| `create_project(project_name, camera=None)` | Skapa ett nytt projekt (valfritt med en kameramall som `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importerar RAW-/TIF-/JPG-/DNG-bilder **och `.daq`-ljussensorinspelningar**. Returnerar `count` (bilder) och `scan_count` (inspelningar). Visar en varning endast om mappen inte innehåller något av dessa. |
| `export_light_sensor(daq=True, csv=True)` | Skriv kalibrerade `.daq` + `.csv` för varje ljussensorinspelning i projektet, till `<project>/Light Sensor/`. Se [Ljussensorinspelningar](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Ställ in bearbetningsreglagen. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Kör bearbetningskedjan. Returnerar `{"status": "complete", "async": False}`, plus en `summary`-nyckel när backend tillhandahåller en — se [Sammanfattning och tips efter körning](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Kontrollera backendens status. |
| `logout()` | Rensa cachade autentiseringsuppgifter. |
| `shutdown_backend()` | Avsluta backend (om SDK -started). |
| `discover_cameras()` | Upptäck LATTICE-kameror **via denna instans backend** (`/api/camera/discover`). Returnerar en lista med ordböcker (`serial`, `model`, `ip`, …) — samma form som GUI/CLI . Tom lista om inga hittas eller om backend inte kan nås. |
| `camera_capture(output_dir, format="tiff", **settings)` | Fånga en enskild bildruta**genom backend**(startas automatiskt av detta handtag) så att den får samma förberedelse som GUI/CLI (12-bitars standard, återanvändning av pool, inbäddade kalibrerings- metadata). Bestäm målet med `serial=` eller `device_index=`; skicka vidare `exposure`/`gain`/`pixel_format`/`preset` som `**settings`. Returnerar den äldre metadatadiktionen (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Genererar överlagrade sammansatta förhandsgranskningsbilder från en sammanslagen kamera — tunn MJPEG-klient via backendens `/api/camera/<serial>/stream-annotated`-väg (zebra / rutnät / hårkors / histogram / peaking / punkt ritad på serversidan). `decode=True` genererar BGR-matriser; `False` genererar råa JPEG-byte. Kan även nås per projekt som `ChlorosProject.stream(overlays=True)`. |

Använd som kontextmanager för garanterad rensning:

```python
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

### Ljussensorinspelningar — kalibrerade `.daq` + `.csv`

En DAQ-U / DAQ-M / DAQ-E kan spelas in **utan** sitt kalibreringspaket. Det är
vad de offentliga [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
inspelarna (`record_daq.py`) gör som standard: de skriver ut råa sensordata och märker
filen så att Chloros hämtar sensorns fabrikskalibrering **via serienummer** — först från den lokala cachen
och sedan från MAPIR Cloud — och tillämpar den vid import.

Chloros skriver ut resultatet som två produkter per inspelning, under
`<project>/Light Sensor/`:

| Produkt | Vad det är |
| --- | --- |
| `<name>_calibrated.daq` | Det ombearbetningsbara arkivet — samma schema som en liveinspelning, men nu med angivelse av det paket som genererade den. Om den importeras på nytt kalibreras den **inte** en andra gång. |
| `<name>_calibrated.csv` | Spektral irradians i W/m²/nm på sensorns eget våglängdsnät, en rad per avläsning, plus fotometriska kolumner (total effekt, fotopisk/skotopisk lux, PPFD och dess uppdelning i blått/grönt/rött, toppvåglängd). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Endast sensorer utan paket (DAQ-A).** Råa spektrala sensortal — *inte* irradians. Se nedan. |

`process()` utför denna export som ett av sina steg. Det kräver **inte** bilder:
en ljussensor som flygs på egen hand är ett förstklassigt arbetsflöde, och ett sådant projekt har noll
bilder per definition.

**DAQ-A-inspelningar exporteras som råa värden.** DAQ-A-familjen är äldre än systemet med
serienummerbundlar och har ingen bunt att hämta — den kalibreras istället i fält mot ett
reflektansmål istället, vilket är anledningen till att den aldrig behövt någon. Dessa inspelningar exporteras
under stammen `_raw` istället för `_calibrated`: ett annat filnamn istället för en flagga
inuti filen, eftersom uppgiften måste klara att skickas via e-post som ett rent namn. Rubriken
`.csv` anger `raw spectral sensor counts (NOT irradiance)` och varnar för att
värdena är jämförbara **inom** filen — precis vad målbaserad kalibrering använder
dem till — och inte mellan olika sensorer. De effektberoende fotometriska kolumnerna (total effekt,
fotopisk/skotopisk lux, PPFD) returneras som **NULL** istället för att integreras utifrån räknevärden.

En DAQ-U / DAQ-M / DAQ-E vars paket helt enkelt inte kunde hämtas **hoppas fortfarande över**,
och skrivs inte ut i råformat: där finns paketet och ”anslut om och bearbeta om” är ett konkret råd.

Äldre **v1.01 / v1.02**-inspelningar (en DAQ-A-SD skriver dessa) har ingen epok per avläsning,
utan endast filens skrivtid. Bild↔nedåtriktad matchare vägrar fortfarande att — att matcha en
ram mot en skrivtid skulle ge osynliga fel — men exportören läser dem, och
CSV skriver ut `clock=daq_created_on` så att produkten anger vilken klocka den använder.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

En inspelning vars kalibreringspaket inte kan hämtas (offline, eller en sensor utan
kalibrering i filen) rapporteras under `skipped` **med orsaken**. Den skrivs aldrig
ut som en ”kalibrerad” fil som innehåller råa räknevärden — anslut till internet och
kör om, så slutförs exporten.

### Återkopplingar om förlopp

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Sammanfattning och tips efter körning

När processen är klar hämtar `process()` `GET /api/processing-summary` och bifogar huvudtexten som `result["summary"]`. Hämtningen sker efter bästa förmåga och blockerar aldrig ett lyckat retur — om sammanfattningen inte är tillgänglig faller `process()` tillbaka till den vanliga `{"status": "complete", "async": False}`-formen. Varje post i `summary["hints"]` — fullständiga meningar med föreslagna åtgärder, t.ex. varför en körning gav nollutdata — skickas också ut på nytt som en Python `UserWarning`, så körningar med nollutdata är självdiagnostiserande även om du aldrig granskar ordlistan:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` är den maskinläsbara delen:

| Nyckel | Vad den räknar |
| --- | --- |
| `models` | Kameragrupper i körningen. |
| `images_in_groups` | Källbilder i dessa grupper. |
| `targets_found` | Detekterade reflektansmål. |
| `images_calibrated` | Bilder som körningen kalibrerade. |
| `exported_files` | **Bildproduktfiler som körningen skapade.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Ljussensorregistreringar, räknade separat avsiktligt – de kommer från ett annat steg och finns även för körningar utan bilder alls, så att inkludera dem skulle få en körning som endast omfattar datainsamling att se ut som om den exporterade bilder. |

Vid sidan av dessa: `summary["output_dirs"]` (varje katalog som skrivits till),
`summary["light_sensor_export"]`, `summary["stopped"]` (gäller när användaren avbröt
körningen, så att partiella räkningar inte tolkas som en avslutad körning som underpresterade), och
`summary["groups"]` (uppdelningen per grupp).

`exported_files` registreras av pipelinen **medan den skriver**, inte genom att skanna av
projektets bildobjekt i efterhand. Parallell- och GPU-strategierna bygger sina egna bildobjekt
objekt (i arbetarsubprocesser för GPU-vägarna), så den gamla skanningen rapporterade
`0 file(s) written` för varje sådan körning och skickade sedan ut tipset om nollexporter — vid körningar
där allt hade fungerat. Om du skapar ett skript baserat på detta nummer rapporterar en felfri parallellkörning nu
ett värde som inte är noll.

Hoppade ljussensorer rapporterar den orsak som läsaren faktiskt fastställde för varje fil – ett
oläsbart schema, ett saknat paket, ett skrivfel – **deduplicerat**, så att tjugo filer
som hoppades över på grund av en orsak räknas som en orsak istället för tjugo upprepningar av den.

> **`process()` utlöses inte när en körning inte producerar några bilder.** Detta är det enda stället där SDK och
> CLI medvetet skiljer sig åt: `chloros-cli process` behandlar ”produkter begärdes, inga
> skrivna” som ett fel och avslutas med ett värde som inte är noll, medan SDK avslutas normalt och rapporterar
> tillståndet via `summary` / hints. Om din pipeline ska avbrytas vid en tom körning, kontrollera den
> själv – granska `summary` (eller räkna filerna i projektmappen) istället för att förlita dig på
> frånvaron av ett undantag. Vanliga orsaker är en inmatningsmapp som inte kändes igen som en
> inspelning och produkter som hoppats över eftersom de inte var tillämpliga för de närvarande kamerorna (t.ex. strålningsvärden från kameror som endast stöder RGB
> ).

### Bekvämlighetsfunktioner

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Stödda värden

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrisk utdata (LATTICE-pipeline för multispektral data)

`process`-pipelinensLATTICE-multispektrala (M3C/M3M) exportnivå — `reflectance` (standard), `radiance`, `sensor-response` eller `all` (alla tillämpliga lägen per bild) — motsvarar projektets bearbetningsinställning **”Radiometrisk utdata”**. `configure()` har ett särskilt nyckelord för detta:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Den avancerade utvägen — att skriva in projektets `"Radiometric output"`-nyckel via `custom_settings` — fungerar fortfarande, men kom ihåg att den ersätter hela inställningsblocket (se varningen nedan):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (standardinställningen) dividerar kamerans strålning med **tidsstämpelmatchad DAQ-nedstrålning**, som automatiskt beräknas utifrån en inspelad `.daq` (DAQ-U/M/E)**eller en DAQ-M-inbyggd `.csv`**som finns tillsammans med bildmaterialet; eventuella kalibreringspaket per kamera eller DAQ som saknas lokalt**hämtas automatiskt från AWS** vid första användningen. CLI visar detta som produktväljare per typ på `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **ersätter** hela blocket med beräknade inställningar (det kringgår `configure()`:s övriga nyckelord och validering enligt designen). När du använder det ska du inkludera alla `Project Settings`-nycklar som är viktiga för dig, precis som i exemplet ovan.

---

## Smart-Connect för LATTICE-kameror

Persistenta backend-sessioner för live-hårdvara. Samma slutpunkter som GUI:n använder, så beteendet är identiskt på SDK / CLI / GUI.

### Enstaka kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### `connect_camera()`-signatur

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession`-metoder

| Metod | Beskrivning |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Läser GenICam-noder; returnerar `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Skriver noder med vänligt namn (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Ta en **enskild** bild. Returnerar en lista med ett element som består av bildmetadata-diktionärer. (Serietagning/tagning av flera bilder har tagits bort — anropa `capture()` i en loop om du behöver en serie.) |
| `disconnect()` | Frigör från poolen. Ingen åtgärd om vi är anslutna till en redan öppen session. |

`capture()`-exportkontroller (samma modell som arrayen + GUI):

- `processing` / `levels` — `processing="all"` sparar alla tillämpliga exporttyper; `levels=["raw","radiance"]` sparar endast dessa (åsidosätter `processing`). Utelämna båda för backend-standardinställningen.
- `force_daq=True` — spara den tilldelade DAQ/DLS-avläsningen som en `.daq`-sidecar även vid en ren rådata-insamling, så att ramen senare kan ombearbetas till reflektans/index. Ingen åtgärd om ingen DAQ är länkad.

### Synkroniserad matris — `ArraySession` (Smart-Prep)

`connect_array` är **den rekommenderade startpunkten** för uppsättningar med flera kameror. Den kör hela Smart-Prep-flödet via GUI i bakgrunden:

1. **Nätverksanalys** (`/api/camera/array/recommend`) — hittar den största bildstorleken som passar sim-emit-nivån utan att bildrutor tappas bort.
2. **Automatisk val av nivå** — `sim-capture-sim-emit` om kabeln klarar det; annars `sim-capture-ftd-stagger` eller `slip-emit-and-capture`.
3. **Automatisk minskning**— minskar ramstorleken utan varning / ökar binningen när kabeln inte klarar den begärda upplösningen.**Detta säkerhetsnät täcker inte aggregerad överteckning**: för många kameror för kabeln kan inte åtgärdas genom att minska ramarna — se [Överteckning](#over-subscription-the-per-cam-floor).
4. **PTP aktiverat**som standard — tidsstämplar mellan kameror hamnar på en gemensam klocka med en avvikelse på**~1 ms**. Samtidig exponering sker via M8-hårdvarutriggaren (**&lt; 100 µs** mellan moduler), inte via PTP: PTP synkroniserar *tidsstämplar*, inte exponeringar.
5. **Automatiskval** — RGB-kameror → `BayerRG8`, multispektrala → `BayerRG12`.
6. **AE-seeding** — tar en ögonblicksbild av varje kameras aktuella AE-tillstånd så att anslutningen inte återställer exponeringen mitt i en bildserie.
7. **GPIO-triggerkonfiguration** — `connect_array` aktiverar varje kamera (`TriggerMode=On`, `TriggerSource=Line2`) så att masterkameraens puls styr slavkamerorna via M8-kabeln. Detta steg gäller endast för en array: en enskild kamera som öppnas med `LatticeCamera` körs istället i fritt läge.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()` Signatur

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`force_tier`-värden:
- `"sim-capture-sim-emit"` — verkligt simultant (alla kameror avfyras vid samma klockkant).
- `"sim-capture-ftd-stagger"` — flexibel tidsförskjutning (kamerorna avfyras vid något förskjutna tidpunkter så att paketen serialiseras på ledningen).
- `"slip-emit-and-capture"` — sekventiell insamling per kamera (ingen tidssynkronisering; enda alternativet när ingen ramstorlek passar sim).

`wire_ceiling_mbps` åsidosätter **värdens kontinuerliga nätverksbandbredd** i MB/s — det enda
siffran som hela arraytilldelningen baseras på. Lämna inställningen på `None` för att använda det automatiskt detekterade
värdet. Sänk värdet när arrayen rapporterar GVSP-korrupta ramar: det automatiska värdet härleds
från nätverkskortets angivna länkhastighet, vilket överskattar USB-adaptrar, smala PCIe-banor och
upptagna delade nätverk — och överskattningen visar sig som korrupta ramar snarare än som en
synligt långsam länk. Värdet sparas i projektets array-insamlingsblock, så en
återöppning eller ett senare `connect_array` återställer det precis som vilken annan array-inställning som helst.
Se [Array Health](#array-health--which-subsystem-is-losing-frames).

#### Överteckning (minimigränsen per kamera)

Sim-emit-pacing tilldelar varje kamera en andel av den kollisionssäkra bandbreddsbudgeten, med en lägsta gräns på **8 MB/s per kamera**(`per_cam_floor_bps`). När `N × floor` överskrider det kollisionssäkra taket**övertecknar arrayen bandbredden**— felmoden är GVSP-paketförlust, inte en lägre bildfrekvens — och det finns ingen lösning genom att minska bildstorleken:**binning och ROI minskar antalet byte per bildruta, inte de reglerade byte per sekund**som den aggregerade kontrollen jämför. Praktiska takvärden för full upplösning på en 1 GbE-värd:**6 kameror @ 1500 MTU, 9 med jumbo-ramar** (`max_cams_collision_safe` i analyssvaret anger gränsvärdet för din anslutning). Åtgärder: färre kameror, jumbo-ramar fråntill-till, eller ett snabbare nätverkskort.

- Svaren `analyze_array_network()` och `/api/camera/array/connect` innehåller `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` och `per_cam_floor_bps`. När `oversubscribed` är sant nollställer projektionen **fps-fälten** (`achievable_fps_max` / `fps_bright` / `fps_dark`) istället för att rapportera en missvisande hastighet som är långsam men fungerar.
- `POST /api/camera/array/connect` accepterar en `pin_resolution`-kroppsparameter (**endast HTTP — inte en SDK-kwarg**; `connect_array` exponerar den inte). Fastställning avlägsnar säkerhetsnätet för nedtrappning av binning, så en övertecknad anslutning med `pin_resolution` inställt**avvisas kategoriskt** med ett felmeddelande som anger alla möjliga åtgärder. Utan pinning fortsätter anslutningen med nedskärningen men varnar för att minskningen inte kan rensa aggregatet.
- Nödutväg för testmiljö: ställ in `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` i backendensmiljö för att nedgradera avvisningen till en tydlig varning — du ansluter ändå och accepterar paketförlusten.

#### Arrayhälsa — vilket delsystem tappar ramar

`GET /api/camera/array/<array_id>/capability` bär ett aktivt `health`-block på en
ansluten array, omvärderat i ett rullande **10-sekunders** fönster. Den delar upp ramförlusten
i de två orsakerna som kräver motsatta åtgärder, istället för en ”ofullständig” frekvens som
inte specificerar någon av dem:

| Fält | Vad det betyder | Vilket delsystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriell port) | Ramen **anlände men var strukturellt felaktig**— GVSP-paketförlust. |**Nätverk**: kabelkapacitet, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (per seriell port) | Ramen **kom aldrig alls**— kameran utlöste inte, eller så lämnade ingenting den. |**Utlösare/synkronisering**: M8-kabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Sämsta kamerans andel för varje. | — |
| `per_cam_rate_pct` | Kombinerad andel ofullständiga bilder per kamera (båda orsakerna tillsammans). | — |
| `stable_for_seconds` | Hur länge varje kamera har legat under 0,01 %. | — |

Tillsammans med `health` anger samma post det värde som hela tilldelningen hänger på:

| Fält | Vad det betyder |
| --- | --- |
| `wire_ceiling_mbps` | Värdens gällande kontinuerliga bandbredd, MB/s. |
| `wire_ceiling_source` | Varifrån siffran kommer, i ord — t.ex. `USB-capped 200 MB/s (was theoretical 1062; …)` eller `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` när `wire_ceiling_mbps=` ställde in den. |
| `nic_is_usb` | `true` för en USB-Ethernet-adapter. |

Det finns inget SDK-wrapper för denna ändpunkt — läs den direkt:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Avläsning:** ett värde som inte är noll `gvsp_corrupt_rate_pct` med `never_arrived_rate_pct` på 0 betyder
att triggning och kabelsynkronisering är perfekta och att 100 % av förlusten ligger på nätverksvägen — sänk
`wire_ceiling_mbps` och anslut på nytt. Det omvända mönstret pekar istället på synkroniseringskabeln eller
triggerledningen.

> **`target_fps` är inte indikatorn för korrupta ramar.** GevSCPD-takten ställs in en gång vid
> anslutning, så att sänka triggerfrekvensen ändrar arbetscykeln och inte
> burstfrekvensen för samtidig sändning. En uppmätt 5× sänkning av efterfrågan gav ingen förbättring, medan
> en sänkning av kabelns tak från 240 till 200 MB/s tog samma rigg från 10,4 % korrupta ramar till
> 0,00 %.

> **Automatisk krympning mitt i strömmen är inte tillgänglig i TRI032S-firmware.** En pågående array kan inte
> åtgärda detta själv; koppla bort och anslut på nytt så att anslutningstidsväljaren planerar om utifrån
> det nya taket.

En **USB-Ethernet-adapter begränsas till 200 MB/s** av proben oavsett dess
typskylt: effektivitetstabellen som omvandlar en länkhastighet till ett kontinuerligt värde är
hämtad från PCIe, och ett USB-nätverkskort anger sin Ethernet-länkhastighet samtidigt som det begränsas av
USB-bussen och dess drivrutin. Begränsningen är absolut, inte en bråkdel — en USB 1 GbE-adapter
uppnår ~80 MB/s och påverkas inte.

#### `ArraySession`-metoder

| Metod | Beskrivning |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | En synkroniserad inspelningsgrupp. Returnerar en `CaptureResult` (lista över ramordböcker + `.skipped`). Exportkontroller nedan. |
| `capture(..., smart=True)` | **Smart inspelning** — väntar tills AE har stabiliserats på alla kameror, sedan utlöses. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Snabbaste inspelning: endast rådata + den tilldelade DAQ-avläsningen (+ det fria kombinerade indexet). Speglar GUI-knappen ”Fastest Capture”. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Enstaka / Kontinuerlig / Intervall i en avgränsad slinga. Returnerar `list[CaptureResult]`.**Kräver `count` och/eller `duration_s`** för att avslutas (SDKen har ingen Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Starta inspelning av den kombinerade indexvyn i realtid till video/GIF → `RecorderHandle`. En sammansatt inspelare per array. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Starta en rå-Bayer-bildserie med hög bildfrekvens → `RecorderHandle`. Bearbeta om offline med `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Bearbeta en sparad råbildsserie offline till kalibrerad video. Blockerar tills processen är klar (`wait=True`) och returnerar `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Avfråga ett offline-byggjobb: `{running, result, error, burst_dir}`. |
| `disconnect()` | Frigör hela arrayen. |

`capture()` exportkontroller (samma slutpunkt som GUI/CLI använder):

- `processing` / `levels` — `processing="all"` (eller `levels=["raw","radiance",…]`) sparar alla tillämpliga exporttyper per kamera; ett enda `processing`-värde sparar endast den nivån.
- `aligned=True` — förvränger varje medlems icke-råa export till arrayens [justeringsprofil](#array-alignment) (samregistrerad); rådata förblir oförskjutna men bär med sig transformationen i metadata. Faller tillbaka till oalignerad (med en varning som visas i resultatets `alignment`) om matrisen saknar profil.
- `render_index=False` — hoppa över överlägget för vegetationsindex per kamera; standardinställningen renderar det där det är konfigurerat.
- `force_daq=True` — spara den tilldelade DAQ/DLS-avläsningen som en `.daq`-sidecar även när ingen vald nivå behöver den.

**TIFF komprimering (endast reglaget HTTP):**`ArraySession.capture()` skickar ingen `compression`-nyckel, så backendens standardinställning gäller — `POST /api/camera/array/capture` läser in en `compression`-kroppsparametern, `"deflate"` som standard (förlustfri zlib L1 + horisontell prediktor, ~4,1 MB per bildruta i full upplösning). `"none"` skriver okomprimerat (~6,3 MB/bildruta) med en**~5× snabbare skrivhastighet** — båda är förlustfria och läses identiskt vid import. SDK exponerar inga kwarg för detta; genvägen är `chloros-cli lattice array-capture --compression none` eller rå HTTP. DEFLATE håller också GIL:en Python, så komprimerade skrivningar kan inte parallelliseras över skrivtrådarna per kamera — kontinuerlig inspelning i full upplösning med 8 kameror vid sensorhastighet kräver `compression: "none"`. Detaljer: [CLI Referens → array-capture](cli-reference.md).**Överskrivningar vid export per medlem (endast HTTP):**samma slutpunkt accepterar även `exclude_serials` (lista — ta bort medlemmar från den sparade uppsättningen; matrisen utlöses fortfarande som en synkroniserad grupp och uteslutna medlemmar returneras i `excluded`), `serial_levels` (`{serial: [level tokens]}`-överskrivningar på kameranivå) och `serial_index` (`{serial: bool}`-överskrivningar av indexöverlägg per kamera). Dessa är GUI-paritetsparametrar och**ännu inte **SDK-kwargs**; medlemmar som saknas i kartorna faller tillbaka till de array-omfattande `levels` / `render_index`.

##### Granska hoppade kammer — `CaptureResult.skipped`

`ArraySession.capture()` returnerar ett `CaptureResult`, vilket är en underklass till `list`: iterera det, indexera det, `len()` det — alla befintliga mönster fortsätter att fungera. Ny kod kan inspektera attributet `.skipped` för att se vilka kammar som uteslöts och varför. Det vanligaste fallet är RGB-kameror i en blandad filtermatris när du begär `processing="radiance"` eller `"reflectance"` — strålning per Bayer-matris är meningslöst för en bredbandssensor, så backend-systemet hoppar över dessa kameror istället för att producera nonsens.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Orsaktoken följer mönstret `<level>-not-applicable-to-rgb-cam` (en post per utelämnad nivå, var och en med `level`). De reflektansspecifika hoppningarna är `reflectance-skipped-no-fresh-dls` (ingen ny nedåtriktad avläsning tillgänglig), `reflectance-skipped-bound-daq-unavailable (…)` (den bundna DAQ:en kunde inte nås) och `dls-uncalibrated-band-<nm>` — bandet ligger till största delen utanför DAQ-ljussensorns radiometriskt kalibrerade intervall (~374–974 nm), varför den absoluta DAQ-baserade reflektansuppdelningen avvisas och ramen nedgraderas tydligt till sensorns respons. Bland de levererade SKU:erna är det endast F988 som utlöser detta; den kamerans stödda väg är arbetsflödet med reflektanspaneler.

`processing`-nivåer:

| Nivå | Utgång |
| --- | --- |
| `"raw"` | Enkanalig Bayer (monokameror: det enda bandet) direkt från sensorn. |
| `"debayered"` *(standardinställning för SDK)* | 3-kanals BGR via bilineär demosaik (monokromkameror: 1-kanals gråskala). |
| `"radiance"` | float32 W/m²/sr/nm via hela den radiometriska kedjan. Endast multispektralt — RGB-kameror hoppas över. |
| `"reflectance"` | uint16 0..32768 (Pix4D-kompatibel); kräver en aktiv DAQ-koppling för absolut referens. Endast multispektral. |
| `"display"` | Hela kedjan matchar förhandsvisningen i GUI (CCM + WB + gamma enligt kamerans profil). |
| `"all"` | **En fil per tillämplig nivå** för varje kamera (överensstämmer med standardinställningen ”Capture All”/CLI i GUI). Den returnerade filen `CaptureResult` innehåller då ett bilddikt per `(cam, level)`, med nivån i varje dikt; icke tillämpliga nivåer visas i `.skipped`. Den DAQ-avläsning som används för varje reflektansbild sparas som en `.daq`-sidecar. |

> **Obs! – Standardvärdet skiljer sig från det som anges i CLI.** `ArraySession.capture()` har som standardvärde `processing="debayered"`; kommandot `chloros-cli lattice array-capture` har som standardvärde `processing="all"`. Ange `processing="all"` explicit från SDK för att spegla CLI /GUI-sparandet på flera nivåer.

### Inspelningslägen och inspelningsenheter

Matrisyta speglar GUI-inspelningspanelen: Enstaka / Kontinuerlig / Intervall / Snabbaste slutarlägen, plus två inspelningsalternativ (live-kompositvideo och rå bildserie → ombearbetning offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**är SDKs kontinuerliga/intervall-loop. Eftersom det inte finns något `Ctrl+C` för att avbryta den från ett skript,**måste** du skicka `count` och/eller `duration_s` (den stannar när någon av dem nås). `interval_s` mäts från början av varje genomgång (i enlighet med GUI:n). Återstående kwargs skickas direkt vidare till `capture()`.
- **`record`** är *av övervakningsklass*: den fångar den kombinerade indexkompositen i realtid så som den visas, så den kombinerade strömmen måste vara öppen för att bildrutor ska kunna registreras. En kompositinspelare per array (utlöser ett fel om en redan körs).
- **`burst` → `build_video`** är *analysklass*: `burst` skriver råa bildrutor + ett manifest per bildruta + en `.daq` per distinkt DLS-avläsning under `<output>/bursts/<base>/` vid inspelningsslingans fulla hastighet (ingen kedja, inget exiftool, ingen live-visning). `build_video` tidsanpassar varje bildruta till närmaste `.daq` och körkör importpipelinens kedja för strålning/reflektans/index. `products` är en lista över `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (standard: det kombinerade indexet). `burst().stop()` startar också automatiskt en ”best-effort”-konstruktion av det kombinerade indexet, som returneras som `build_job` i stopp-resultatet.

#### `RecorderHandle`

Returneras av `ArraySession.record()` och `ArraySession.burst()`. Använd den som en kontextmanager för att automatiskt avbryta vid utgång ur omfattningen, eller kör den manuellt.

| Medlem | Beskrivning |
| --- | --- |
| `job_id` | Backend-jobb-id (str). |
| `kind` | `"composite"` (från `record`) eller `"raw"` (från `burst`). |
| `start_stats` | Dict som returneras av anropet `start`. |
| `result` | `None` under körning; den slutliga stoppresultatdiktionen när stoppet har genomförts. |
| `stats(timeout=10.0)` | Live-jobbstatistik (skrivna bildrutor, uppnådd bildhastighet, förfluten tid). |
| `stop(timeout=60.0)` | Stoppa inspelaren; returnerar och cachar det slutliga resultatet. Idempotent (ett andra anrop returnerar det cachade resultatet). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Ansluta till en redan ansluten array — `attach_array`

Om arrayen redan är igång (GUI:n har öppnat den, eller en tidigare SDK-session har anropat `connect_array`), använd `attach_array` för att hämta ett handtag till den istället för att ansluta på nytt. `connect_array` ger alltid felmeddelandet &quot;Kameran  finns<sn> redan i arrayen<id>” i den situationen, eftersom en POST-begäran med `/array/connect` för en medlem ipool inte är idempotent; `attach_array` läser `/api/camera/array/list` och matchar antingen via array_id eller serienummer.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Mönster: SDK-skript som körs parallellt med skrivbordsgränssnittet bör först försöka med `attach_array` och sedan fallera till `connect_array` om det ännu inte finns någon array i poolen.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Viktigt — avslutning av context-manager kopplar INTE bort anslutningen.**`ArraySession.disconnect()` skickar alltid en POST-begäran till `/array/disconnect`; det finns ingen ”attached-not-owned”-kontroll som för `CameraSession` / `DAQSensorSession`. Om du delar resurser med GUI:n och intevill riva ner arrayen vid scope-exit,**använd inte `with`-blocket** — spara handtaget i en vanlig variabel och hoppa över det explicita `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Hjälpmedel för nätverksanalys

Användbart innan du öppnar arrayen — visar om dina föreslagna inställningar kommer att passa:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` är en av `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (annars `error`). `auto_capped_fps` innebär att den begärda upplösningen endast passar RX-ringen vid en begränsad utlösningsfrekvens – behåll upplösningen och skicka `target_fps=result["recommended"]["recommended_target_fps"]` till `connect_array` (se [Exempel 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Hur man tolkar projektionen** (samma modell som panelen Arrayinställningar i GUI):

- **Burst (`frame_bytes_total`) summeras per kamera i varje kameras faktiska pixelformat.**Mono**M3M**-kameror strömmar Mono12 (2 B/px) oavsett vilket `pixel_format`-värde du anger, så en bildram med full upplösning från 4 kameror är**~25 MB** med tre monokameror, inte de ~12,6 MB som antagandet om enbart 8-bitarsformat ger. Backend avgör varje kameras format utifrån dess modell.
- **Admittans (`burst_fits_nic_ring`) är dräneringsmedveten**, inte helburst-vs-ring: sim-emit passar när värden tömmer RX-ringen snabbare än kamerorna fyller den. En 10G-värd + 1 GbE-kameror**tillåter** full upplösning även när bursten överskrider ringen; en 1 GbE-värd blockerar (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` är ett konservativt tak för seriell hämtning** — `max(readout+emit, N×emit)` med utsändning per kamera begränsad till 1 GbE-kameralänken, oberoende av exponering. T.ex. ~2,8 fps för en 4-kameras array med full upplösning och 12 bitar (stämmer överens med runtime-mätningarna på ~2,7–3,0). Fullständig modell: [CLI Referens → Array fps &amp; burst-modell](cli-reference.md#array-fps--burst-model).
- **Överskrivning (`oversubscribed: true`) innebär att N × lägsta gränsen per kamera överskrider den kollisionssäkra övre gränsen** — fps-fälten (`achievable_fps_max` / `fps_bright` / `fps_dark`) visar 0, och automatisk krympning/binning kan inte åtgärda det (de minskar antalet byte per bildruta, inte antalet byte per sekund). Lösningarna är färre kameror, jumbo-ramar eller ett snabbare nätverkskort; `max_cams_collision_safe` rapporterar taket (6 kameror med full upplösning på 1 GbE @ 1500 MTU, 9 med jumbo). Svaret innehåller även `aggregate_demand_bps`, `collision_safe_ceiling_bps`, och `per_cam_floor_bps` (8 MB/s). Se [Överteckning](#over-subscription-the-per-cam-floor).

### Upptäckt och listning

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

LATTICE-matriser kör kontinuerlig AE i bakgrunden så snart de är anslutna, men det tar en stund för en nyinsatt scen att konvergera. **Smart-capture** är den praktiska lösningen: den avläser varje kamerasexponering, väntar tills arrayen är stabil över ett fönster och utlöser sedan bildtagningen. Det motsvarar GUI: desktop-appens ”smart”-knapp för bildtagning anropar samma backend-ändpunkt.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

När du styr via `ChlorosProject` (nästa avsnitt) får du fler inställningsmöjligheter:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Smart-AE-inställningen är konservativ som standard. Skärp `exposure_tolerance_pct` för noggrant radiometriskt arbete; lätta på inställningen för snabbt föränderliga scener där du bara vill ha ”nära nog”.

---

## DAQ-sensorsessioner

Persistent backend-pool för spektralsensorer (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Speglar kamerans yta: smart-detect, återanvändning av poolen, idempotent anslutning.

### Smart-Detect (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioritet: Ethernet → BLE → USB. Ange valfri explicit hint för att låsa transporten.

### Fastställd transport

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### `DAQSensorSession`-metoder

| Metod | Beskrivning |
| --- | --- |
| `status(timeout=10.0)` | Sammanfattning av poolpost (strömnings-/inspelningsstatus, våglängdsområde, kalibrerings-SHA, integrationstid, frame_avg, AE-status). |
| `latest(n=1, timeout=10.0)` | Returnerar upp till N senaste spektrumramar. |
| `stream_start()` / `stream_stop()` | Återuppta / pausa strömning (handtaget förblir öppet). |
| `record_start(output_dir=None, device_name=None)` | Starta inspelning av en .daq-fil. Returnerar filvägen. Avvisas för DAQ-U/M utan ett AWS-kalibreringspaket (DAQ-E undantaget). |
| `record_stop()` | Avbryt inspelning. Returnerar `{path, rows}`. |
| `disconnect()` | Frigör från poolen. Ingen effekt för anslutna men icke-ägda handtag. |

> **Kapacitetskorrigeringsprofiler (`cap_id`) är inte en SDK-reglage.** `connect_daq_sensor()` / `DAQSensorSession` exponerar ingen `cap_id`-parameter eller `set_cap`-metod. Välj en flotta-cap-korrigeringsprofil viaCLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) eller backendens `/api/daq`-HTTP-rutter (`/api/daq/connect` och `/api/daq/<id>/cap-id` accepterar `cap_id`).

### Upptäckt — att hitta en adress att ansluta till

`discover_daq_sensors()` skannar USB / BLE / ETH efter sensorer som du *skulle kunna* öppna. Det är DAQ-motsvarigheten till `discover_lattice_cameras()`, och det enda sättet att få fram en **DAQ-M:s BLE-MAC** — en DAQ-E har ett värdnamn och en DAQ-U en COM-port, men en MAC-adress är varken tryckt på enheten eller listad av operativsystemet.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Fält | Beskrivning |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-port / BLE-MAC / värdnamn — vidarebefordras till `connect_daq_sensor` som `port=` / `mac=` / `eth_host=`. |
| `display` | Läsbar etikett. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, eller `None` för en port som skanningen inte kan identifiera (USB-serieadaptrar är omöjliga att skilja åt utan en sond, så okända poster visas istället för att döljas). |
| `extra` | Detaljer per transport (BLE-annonserat namn, USB-tillverkare, DAQ-E ip/fw/…). Tomma värden utelämnas. |

| Parameter | Standard | Beskrivning |
| --- | --- | --- |
| `transports` | alla tre | Sekvens (eller CSV-sträng) som begränsar skanningen. Värt att ange när du vet vad du vill ha — BLE är den långsamma delen. |
| `scan_timeout` | 5 | Sökfönster per transport i sekunder; backend begränsar till 1–20. |
| `timeout` | 60.0 | HTTP-tak för hela anropet (precis som på andra ställen i SDK). |
| `auto_start_backend` | `True` | Starta en lokal backend om ingen körs. Starts aldrig för en fjärr-`backend_url`. |

> **Sensorer som redan är öppna i poolen visas inte.** En ansluten BLE-enhet slutar sända ut information och en öppen COM-port kan inte undersökas, så upptäckten listar vad som är *tillgängligt för anslutning*. Ett tomt resultat direkt efter att du har anslutit något är förväntat — använd `list_daq_sensors()` för det du redan har. Transportprotokoll vars skanning inte kan köras (ingen bleak/zeroconf installerad) hoppas över istället för att generera ett fel, så en maskin utan Bluetooth får fortfarande sina USB- och ETH-svar.

### Lista

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Samtidig användning med GUI/CLI

Om GUI redan har en sensor öppen, returnerar ett anrop av `connect_daq_sensor(port="COM3")` från Python ett handtag märkt `already_connected=True`. Sessionens `disconnect()` är då en no-op, så ditt SDK-skript river inte sensorn ur GUI:n när programmet avslutas.

### Direkt-hårdvaruklasser (Ingen backend)

`daq_sdk` återexporteras av `chloros_sdk`, så du kan även styra sensorer från början till slut i processen utan backend:

> **Tillgänglighet:**`daq_sdk` levereras med Chloros-installationen för skrivbordet,**inte** med PyPI-paketet — `pip install chloros-sdk` ger dig `lattice_sdk` men lämnar kvar `chloros_sdk.DAQ_AVAILABLE == False`. Kontrollera den flaggan innan du använder dessa klasser; på en värd som endast använder pip ska du istället styra sensorn via [`connect_daq_sensor()`](#daq-sensor-sessions) istället, vilket inte kräver några lokala transportbibliotek.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Använd helst smart-connect-vägen (`connect_daq_sensor`) när du vill ha delat ägande med GUI; använd de direkta klasserna för skript utan grafiskt gränssnitt som äger sensorn exklusivt.

---

## Projektautomatisering — `ChlorosProject`

Ett sparat Chloros-projekt är en mapp som innehåller `cameras.json` + `sensors.json` + `project.json`. `open_project` laddar manifestet, och `connect_all` kopplar upp alla sparade enheter med sina sparade inställningar — samma hårdvarutillstånd som GUI:n skulle åstadkomma.

### Minimalt exempel

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Eller som en kontextmanager:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject`-metoder

| Metod | Beskrivning |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Upptäck och anslut alla sparade enheter. Returnerar en anslutningsrapport per klass. Använder en körande backend om en sådan lyssnar på `127.0.0.1:5000`; i annat fall faller den tyst tillbaka till direkt (backend-fri) `lattice_sdk`-enhetsstyrning — den startar aldrig en backend. |
| `disconnect_all()` | Avsluta allt. |
| `capture_all(output_dir=".")` | En bildruta från varje kamera + matris + spektrum från varje sensor. |
| `stream(camera, overlays=False, fps=10.0)` | Generator som genererar BGR-ramar från en namngiven kamera (eller array). `overlays=False` är en direkt `lattice_sdk`-hämtningsslinga (matriser genererar `{serial: frame}`-dikt). `overlays=True` dirigeras via `ChlorosLocal.camera_stream()` → backendens `/api/camera/<serial>/stream-annotated` MJPEG-ström, där kamerans sparade `ui.overlay`-block skickas vidare som sökparametrar. Kräver backend-läge och en **fristående kamera**: en kamera i direktläge genererar `RuntimeError` (backenden kaninte hämta en kamera som denna process äger) och en array genererar `NotImplementedError` (överlagrar komposit per kamera — strömmar en medlem efter namn). Motsvarighet för engångsbruk: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Kör justering på varje array som för närvarande är ansluten. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Kör kalibrerings-/indexeringspipeline på projektets bilder (omsluter `ChlorosLocal.process`; dessa fyra är de **enda** godkända kwargs — `indices=` etc. ger upphov till `TypeError`; ställ in index via `ChlorosLocal.configure()`). Konstruerar en `ChlorosLocal()` på ett lat sätt, vilket automatiskt startar en backend. |

Attribut:
- `proj.cameras` — `Dict[str, CameraHandle]` med nyckel baserad på namn OCH serienummer.
- `proj.arrays` — `Dict[str, ArrayHandle]` indexerad efter namn OCH array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` sorterad efter namn OCH slot_id.
- `proj.config` — `project.json["config"]` ordlista.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Bearbetningsnivåer.** `capture()`, `grab()` och `frame_stream()` tar alla samma `processing`-
token, och kedjan är kumulativ — varje nivå kör allt ovanför sig:

| Nivå | Utgång | Anmärkningar |
| --- | --- | --- |
| `raw` | 1-kanals Bayer, sensornativ | Ingen demosaik. Överlagringar är inte tillgängliga på denna nivå. |
| `debayered` | 3-kanals BGR (**standard**) | Bilineär demosaik. Den enda nivån som fungerar utan backend-läge. |
| `radiance` | float32, W/m²/sr/nm | Fullständig radiometrisk kedja: demosaik + 3×3-avblandning (multispektral) + DSNU + flat-field + NIST-skala, där exponering × förstärkning har dividerats bort så att värdena är absoluta. |
| `reflectance` | uint16, 32768 = 1,0 | Strålning dividerad med nedåtriktad irradians (ρ = π·L/E). Kräver en DLS/DAQ-avläsning — se anmärkningen nedan. |
| `display` | 8-bitars sRGB-liknande | GUI-motsvarande rendering: CCM + vitbalans + gamma via kamerans aktiva färgprofil. |

Allt annat än `debayered` kräver backend-läge; en kamera i direktläge genererar
`NotImplementedError`. `reflectance` kräver en användbar nedåtriktad avläsning — bildrutan hämtar
automatiskt in den samlade DAQ:n i kamerans DLS-plats, men utan någon bunden DAQ vägrar kedjan
reflektansutgången och markerar ärligt nedgraderingen i de returnerade metadata istället för att tyst
lämna tillbaka en sämre produkt.

> **Reflektans DN-skala – kod den inte in.** LATTICE-reflektans använder `32768` = ρ 1,0 och markerar
> XMP `Chloros:PixelScale=32768`; Survey3 reflektans använder `65535` = ρ 1,0 och innehåller inga
> `Chloros:*`-taggar. Läs av taggen och dividera med den. Den är definierad i uint16-domänen, så den förblir
> `32768` för varje format som skalar om (16-bitars TIFF, 8-bitars PNG /JPG, 32-bitars procent) — normalisera
> först den lagrade datatypen tillbaka till uint16 (×257 från 8-bitars, ×65535 från float). Det enda undantaget:
> en 8-bitars källinspelning som skrivs som 8-bitars TIFF *klipps*, inte skalas om, så ingen skala beskriver
> den — Chloros utelämnar `PixelScale` och MicaSense-tuplen helt i det fallet. Behandla en saknad
> tagg i en LATTICE-reflektansfil som ”ingen giltig skala”, inte som ett standardvärde.

> **EXIF överförs till exporten.** `process()` kopierar källbildens GPS-block
> **och dess ExifIFD** till varje produkt, så exporterna innehåller `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` och `CameraSerialNumber` samt
> georefereringen. `FocalLength` är det som Pix4D använder för att beräkna markprovavståndet utifrån – utan det
> faller rekonstruktionen tillbaka till en helt felaktig skala (i ett uppmätt fall förvandlades en 411 m stor plats
> till en på 47,8 km). Kopian är medvetet inte `-all:all`: IFD0:s strukturtaggar stör
> LATTICE-utdata, och `ExifImageWidth`/`Height` utesluts eftersom de beskriver källans
> bildtagning snarare än den exporterade rasterbilden.

Underflaggor för inspelningsstadiet (gäller de radiometriska nivåerna — `radiance`, `reflectance`, `display`):

| Flagga | Standard | Betydelse |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + NIST-radiometrisk skala. |
| `apply_white_balance` | `True` | WB LUT. DLS-medveten när en DAQ är kopplad till kameran. |
| `apply_index` | `False` | Utvärdering av vegetationsindex. |
| `index_expression` | `None` | Åsidosätt formel. Tomtom → aktiverar indexet automatiskt. |
| `annotated` | `False` | Överlagring av GUI-dekorationer (zebra/rutnät/peaking). Ej tillgängligt för `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Returtypen är `CapturePathMap`, inte `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` är `Dict[str, Union[str, List[str]]]`: en en-nivå
> `processing` ger varje serienummer en väg, medan en flernivåvariant (`"all"`, eller en
> explicit `levels`-lista) ger den den **ordnade listan** över alla produkter som sparats för den
> kameran. En live-komposit, om en sådan strömmas, placeras under den extra
> `"combined"`-nyckeln snarare än under en serienummer. Kod som förutsätter `str` slutar fungera vid
> listformen utan att någon typkontroll invänder — anteckningen angav `Dict[str, str]`
> en tid efter att listformen släpptes, vilket är anledningen till att aliaset finns. Normalisera
> när du vill ha den platta formen:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Arrayjustering

`ArrayHandle` exponerar hela justeringsytan. Profiler är som standard endast sessionsbundna — anropa `export_alignment()` explicit för att spara dem.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Justering vid anslutning

`connect_all(align=...)` kan automatiskt justera varje array vid anslutning:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Återgår till `project.json["config"]["auto_align_on_connect"]` om inget anges.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Direkt hårdvara (utan backend)

När du vill ha noll beroende av backend (CI, headless-robotar, inbäddad), importera `lattice_sdk` och `daq_sdk` direkt — båda återexporteras av `chloros_sdk`. Skydda med `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` ingår i PyPI-paketet (men kräver att Arena-SDK-runtime finns installerat), medan `daq_sdk` endast levereras med desktop-installationen.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Förinställningar och utlösaren

Tre av de fyra förinställningarna **free-run**: kameran exponerar kontinuerligt och en
`capture()` returnerar nästa bildruta. `triggered` är undantaget — den aktiverar
kameran för en hårdvarukant på linje 2, så den fångar ingenting förrän en sådan anländer.

| Förinställning | Utlösare | Används när |
| --- | --- | --- |
| `default` | free-run | allmänt bruk |
| `high_speed` | free-run | 8-bitars, max 60 fps, kort exponering |
| `high_quality` | frilöp | 12-bitars, ingen fps-begränsning — det vanliga valet för stillbilder |
| `triggered` | **förberedd, linje 2** | kameran är ansluten via en M8-synkroniseringskabel och något annat utlöser den |

Om du väljer `triggered` (eller själv ställer in `trigger_mode="On"`) utan att något
styr linje 2, kommer varje `capture()` kommer att gå i timeout — vilket är korrekt, eftersom du bad
kameran att vänta. SDK förklarar detta när det händer; se
[SC_ERR_TIMEOUT under inspelning](#direct-hardware-backend-free).

> **Obs! — ”GVSP-probe” / `SC_ERR_TIMEOUT -1011`-meddelanden vid anslutning är inga fel.**&gt; Vid anslutning försöker SDK förhandla fram**jumbo-ramar** (9000-byte GVSP-paket) för högre genomströmning. På en direkt punkt-till-punkt-NIC-länk (t.ex. en länk-lokal `169.254.x.x`-adress) kan nätverket vanligtvis inte hantera jumbo-ramar, så denna sond går ut och loggar rader som:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Detta är den **inbyggda reservlösningen**: SDK återgår automatiskt till standardpaket på 1500 byte och kameran fortsätter att ansluta som vanligt (de följande `[chunk-enable …]`-raderna ingår i den normala anslutningssekvensen). Inspelningen fungerar fortfarande.
>
> Du kan hoppa över denna sond, men **det är inte bara en logg-dämpare — den stänger av jumbo-ramar.** Kameran svarar på ”Don&#x27;t-Fragment”-pingar endast upp till 1500 byte oavsett hur bra ditt nätverk är, så ping-testet i sig kan aldrig upptäcka jumbo-ramar; denna sond är det enda som kan göra det. Inaktiverar du den kommer kameran skickar standardpaket på 1500 byte för alltid, oavsett nätverk:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Det lönar sig endast i ett nätverk som du *vet* inte klarar jumbo, där det sparar ungefär en sekund i anslutningstid per kamera. Eftersom det är en verklig avvägning snarare än en kosmetisk förändring, anger nu ”SDK” detta när du använder funktionen:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Låt det vara som det är om du inte har en anledning.** Om funktionen är aktiverad mäter varje anslutning om det nätverk du faktiskt har: anslut till en switch som stöder jumbo-paket så upptäcker nästa anslutning jumbo-paket automatiskt, utan att du behöver konfigurera något eller starta om.
>
> Om du *vill* ha jumbo-genomströmningen, aktivera jumbo från ändpunkt till ändpunkt (NIC MTU 9000 + en switch som släpper igenom dem), eller lås det med `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` när du vet att länken stöder det — men föredra ett `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` per kommando framför att ställa in det permanent, eftersom en fastställd storlek hoppar över sondningen och slutar anpassa sig till nätverket framför den. **Varje** enhet i vägen måste kunna vidarebefordra jumbo-paket – inklusive eventuella PoE-splitters eller -injektorer, vilket oftast är anledningen till att en annars jumbo-kompatibel installation inte kan hantera dem.

> **`SC_ERR_TIMEOUT -1011` under `capture()` / `grab*()` är ett annat problem – det är ett verkligt fel.**&gt; Anmärkningen ovan gäller endast `-1011` som loggats av**connect-time probe**. Samma fel som uppstår vid en**inspelning** innebär att kameran anslöt utan problem men inte skickar några bilder:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Det som avslöjar problemet är en kamera vars *kontroll*kanal fungerar som den ska – upptäckten fungerar, inställningarna och `[chunk-enable …]`-skrivningarna lyckas alla – medan *varje* bildram går över tiden.
>
> **Den vanligaste orsaken är att kameran är inställd på en hårdvarutrigger.** Med `trigger_mode="On"` och `trigger_source="Line2"` sänder kameran ingenting alls förrän en elektrisk flank anländer på M8-synkroniseringskabeln. Om du inte har någon kabel som driver den linjen, väntar varje bildtagning i evighet. Kameran är inte trasig och nätverket fungerar som det ska – den gör exakt vad den blivit beordrad att göra.
>
> `CameraSettings()` och `default` / `high_speed` / `high_quality` förinställer frilöpning, och en bildtagning som går ut medan den är aktiverad förklarar sig själv istället för att skriva ut ett naket `-1011`. `PRESETS["triggered"]` aktiverar Line2, enligt design.
>
> För att tvinga en kamera till friläge:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Om den fortfarande går i timeout med `trigger_mode="Off"`, levererar kameran verkligen inte data — skicka loggen och `ip link show` till oss.

#### Färgprofiler (liveförhandsgranskning av RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` väljer skärmens färgprofil för **förhandsvisningen** på RGB-kameror (multispec-kameror ignorerar inställningen):

| Profil | Betydelse |
| --- | --- |
| `raw` | Kringgå den radiometriska kedjan helt. |
| `linear` | DSNU + flat + WB, ingen CCM, ingen gamma. |
| `natural` | Linjär + uppmätt CCM + sRGB-gamma, endast med den enkla finishen (kromatisk utjämning + desaturering av höjdpunkter) — den realistiska standardinställningen. |
| `enhanced` | `natural` plus den fullständiga Hub-Parity-finishen (defringe, vibrance, CLAHE lokal kontrast). Rikare utseende till ungefär **dubbla bearbetningskostnaden per bildruta**, vilket ger en lägre LIVE-bildfrekvens. |
| `custom_temp` | `natural` men vitbalansen låst till `custom_cct_k` Kelvin (DLS ignoreras; begränsad till 2000–10000 K på backend-sidan-sidan). |

Profilen är en hastighets-/utseendeknapp som **endast gäller för liveförhandsgranskning**: sparade bilder får alltid den fulla, rika finishen oavsett vilken profil som valts, så att välja `natural` för att spara inramtid sänker inte kvaliteten på det som sparas på disken. En okänd profil höjer `ValueError`; när en chloros-backend är tillgänglig skickas ändringen även via POST till den så att nästa förhandsgranskningsram återspeglar den (användare av direct-SDK utan backend får ändå inställningsändringen).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Monokameror (M3M) och `Calibration`

En monokrom **M3M**-kamera (`M3M-<lens>-F<wavelength>`) är enkelbands: ett gråskalplan, ingen Bayer-mosaik, ingen 3×3 spektral-crosstalk-matris. `Calibration` känner igen den och visar en `is_mono`-flagga. Reflektansen gäller fortfarande som en radiometrisk karta per band (avblandningen är identitetsmatrisen), men multibandsberäkningar på en enda kamera ger meningsfulla resultat istället för nonsens:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

För att skapa ett vegetationsindex från monokromatisk hårdvara, kombinera flera M3M-kameror med olika våglängder till en justerad multibandsstapel (se [Arrayjustering](#array-alignment)) och beräkna indexet över hela stapeln istället för på en enda kamera.

DAQ direkt-läge:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` godkända nycklar**— exakt `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; utfasad till förmån för `cap_id`), `filter_model` (DAQ-M) och `cap_id` (alla DAQ-typer; `None`/`""`/`"none"` = ren sensor, ingen kondensatorkorrigering). Okända nycklar**ignoreras utan meddelande** — t.ex. `{"integration_time": 64}` gör ingenting (det måste vara `integration_time_ms`). Returnerar `{"applied": [...], "errors": {...}}` och genererar aldrig ett undantag.

`chloros_sdk` återexporterar endast den kärnyta som används ovan. Den fullständiga offentliga APIen för `daq_sdk` (22 namn) lägger till följande — importera dem direkt från `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Undantag

Fånga basklassen för att hantera ”allt som gick fel i Chloros&quot;:

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

> `ChlorosAuthenticationError` och `ChlorosConfigurationError` exporteras på högsta nivå tillsammans med resten; de kan också importeras från `chloros_sdk.exceptions`, som visas.

Hierarki:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Exempel från början till slut

### 1. Bearbeta en mapp med en anpassad förloppsindikator

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Live LATTICE-matris → Reflektans + DAQ-referens

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projektdriven insamlingskampanj

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Bildström från flera kameror → NumPy-pipeline

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skript för inspelning direkt på hårdvaran utan backend

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Funktionskontroll före anslutning av en 4-kamerasuppsättning

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Motsvarighet till inspelningsrecept (ren Python)

CLIs recept-DSL har en direkt motsvarighet i Python:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Automatisk start av backend

Smart-Connect-ingångspunkterna — `connect_camera`, `connect_array`, `connect_daq_sensor` och `discover_lattice_cameras` — är tunna HTTP-klienter som utgår från att att en backend lyssnar på `127.0.0.1:5000` (standardURL för smart-connect-gränssnittet). Om GUI:n eller CLI redan körs, finns det en. Från ett rent skript kanske det inte finns någon – därför **startar dessa funktioner automatiskt-startar den medföljande backend-binären** (utan fönster, på samma sätt som `ChlorosLocal` gör) innan de anropas för första gången, och väntar sedan upp till `backend_startup_timeout` på att den ska starta.

Regler:

- **Endast en lokal URL någonsin startas.** En `backend_url` som pekar på `localhost` / `127.0.0.1` / `[::1]` är tillåten; alla andra värdar antas vara någon annansmaskin och startas aldrig.
- **Backenden lämnas igång för återanvändning** (precis som vid CLI) — det sker ingen implicit avstängning när ditt skript avslutas. Om du kör skriptet igen återanvänds den aktiva backenden.
- **Välj bort med `auto_start_backend=False`** vid något av dessa anrop (t.ex. när du har pekat på en fjärrbackend eller om du själv hanterar backendens livscykel).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Om den medföljande binären inte kan hittas eller startas, genererar det efterföljande anropet HTTP ett åtgärdbart, **plattform-anpassat** `ChlorosConnectError` istället för en ren spårningsrapport om avvisad anslutning — på Windows hänvisar det dig till skrivbordsappen eller ett `chloros-cli`-kommando; på Linux (utan GUI) hänvisar det dig till ett `chloros-cli`-kommandot eller `.deb`.

---

## Miljö och rubriker

SDK markerar varje backend-HTTP-anrop med `X-Chloros-Client: sdk`. Backenden tillämpar licensreglerna för SDK / CLI (inloggning **och** ett betalt Chloros+-abonnemang krävs) istället för GUI:ns kostnadsfria nivå. Detta ställs in automatiskt vid import – dubehöver inte göra någonting.

`http://localhost` och `http://127.0.0.1` identifieras som den lokala backend-miljön. Anrop till andra värdar (t.ex. din egen analystjänst) påverkas inte.

Åsidosätt backend-URLen genom att ange `backend_url=` (eller `api_url=` på `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(En `backend_url` som inte är loopback når endast en source/dev-backend — medföljande backends binder endast loopback; se Remote-Backend Mode för tunnelmönstret.)

---

## Versionshantering och kompatibilitet

- Versionen SDK exponeras som `chloros_sdk.__version__`.
- SDK låser beteendet till den medföljande backend-versionen. Att blanda en äldre SDK med en nyare backend fungerar vanligtvis (framåtkompatibla slutpunkter), men att blanda en nyare SDK med en äldre backend kan orsaka `404`-fel på nya slutpunkter — uppgradera skrivbordsappen så att den stämmer överens.
- Smart-Connect-gränssnittet (`connect_camera` / `connect_array` / `connect_daq_sensor`) och nätverksanalys-ändpunkten returnerar stabila JSON-scheman; nya fält läggs till.

---

## Tips för felsökning

- **`ChlorosAuthenticationError: Login required`** → Kör `chloros-cli login EMAIL PASSWORD` en gång på den här datorn, eller logga in via skrivbordsappen Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-connect-anropen startar automatiskt en lokal backend, så detta meddelande visas endast när den medföljande binärfilen inte kan hittas eller startas (t.ex. en värd som endast använder pip och saknar skrivbordspaket). Meddelandet är plattformsberoende: på Windows öppnar du skrivbordsappen eller kör valfritt `chloros-cli`-kommando; på Linux kör du ett `chloros-cli`-kommando (inget GUI finns) eller installerar `.deb`. För en fjärrbackend, ange `backend_url=` (och `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** vid import → `lattice_sdk` kunde inte laddas (vanligtvis är Arena-SDK-körnings-DLL:er inte installerade). Ytan utanför kameran fungerar fortfarande.
- **Array connect returnerar en upplösning som är lägre än den ursprungliga**→ Backendens smart-prep krymper automatiskt bildstorleken så att den passar överföringsvägen. Använd `analyze_array_network()` för att se varför, och uppgradera sedan länken, acceptera krympningen eller skicka `force_tier="slip-emit-and-capture"` för sekventiell inspelning. Krympningen säkerhetsnätet täcker**inte** aggregerad överteckning (`oversubscribed: true`, fps-fält 0): för många kameror för nätverkskabeln kan inte åtgärdas med binning/ROI — minska antalet kameror, aktivera jumbo-ramar eller byt till ett snabbare nätverkskort (se [Överteckning](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` rapporterar att nätverkskortets mottagningsring är mycket liten (~0,26 MB) / anslutningsgrindar med ”FRAMES WILL DROP”** → Värd-nätverkskortets mottagningsring är inställd på standardvärdet (återställs ofta till 32 efter en uppdatering av nätverkskortdrivrutinen). På en Realtek USB 10GbE-adapter ska du ställa in `ReceiveBufferLen=256` och `PendingReceives=64` (höjd), och sedan starta om backend så att den läser av ringen på nytt. Fullständig procedur: [CLI Referens → Konfiguration och inställning av värd-NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Värden hänger sig vid omstart/avstängning, senare WMI-fel `Invalid class` / nätverkskortet aktiveras inte** → Föråldrad USB 10GbE-drivrutin orsakar `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Uppdatera nätverkskortets drivrutin till en aktuell version (≥ 2026) och tillämpa inställningarna för mottagningsringen på nytt. Se [CLI Referens → Konfiguration och inställning av värd-NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Reflektans avvisad** → En aktiv DAQ måste vara kopplad till kameran (eller arrayen) för reflektans i absolut skala. Koppla antingen via GUI eller använd `processing="radiance"` (W/m²/sr/nm) som inte kräver en kopplad sensor.
- **`smart=True`-insamlingen tar längre tid än förväntat** → AE-konvergensen beror på scenens dynamik; skärp inställningen för `exposure_tolerance_pct` eller förkorta `stability_window_s` om du vill ha en snabbare (mindre stabil) utlösare.

---

## Se även

- [Referens för CLI](cli-reference.md) — varje CLI-underkommando motsvarar ett SDK-anrop.
- [DAQ-sensorguide](../daq/README.md) — sensorspecifika regler för anslutning, kalibrering och registrering.
- Online-dokumentation: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
