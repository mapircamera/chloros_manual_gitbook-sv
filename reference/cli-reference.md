# Chloros CLI Referens

**Version:**

1.2.0**Skapad:**2026-07-29 19:19 ·**Reviderad:** 2026-08-30**Målgrupp:** Optimerad för användning av stora språkmodeller (LLM); läsbar för människor.**Omfattning:** Alla användarvända underkommandon för `chloros-cli`, med alternativ och exempel som kan kopieras och klistras in.

Detta dokument är den fullständiga referensen för kommandoradsverktyget `chloros-cli` som medföljer MAPIR Chloros. Det är avsiktligt uttömmande så att en LLM (eller människa) kan sammanställa vilket stödt arbetsflöde som helst från listorna nedan utan att behöva granska källkoden.

Om du bara behöver det viktigaste, gå till:
- [Fem minuters snabbstart](#five-minute-quickstart)
- [Arbetsflöde för första anslutning av LATTICE-kamera](#lattice-camera-first-connect-workflow)
- [Arbetsflöde för första anslutning av DAQ-sensor](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Inspelningslägen, inspelningsenheter och offline-bearbetning](#capture-modes-recorders--offline-reprocess)

---

## Konventioner

- Alla kommandon har prefixet `chloros-cli`. På Windows är binärfilen `chloros-cli.exe`; på Linux/Jetson är den `chloros-cli`.
- Valfria argument visas som `--flag`. Obligatoriska positionsargument visas utan parenteser.
- Om ett standardvärde anges används det värdet om flaggan utelämnas.
- CLI är en tunn HTTP-klient över Chloros-backend (Flask-server på `127.0.0.1:5000`). Backenden startas automatiskt-startas av de flesta kommandon. `CHLOROS_BACKEND_URL=<url>` pekar på kommandofamiljerna **`lattice`**,**`project`**och**`daq pool-*`** till en fjärrbackend – kärnkommandona (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) låser medvetet fast `http://127.0.0.1:<port>` och ignorerar det (IPv4-literalen undviker Windows:s straff på ~2 sekunder per begäran för `localhost`→`::1`). Se [Miljövariabler](#environment-variables).
- Inloggning med ett Chloros+-konto krävs för alla SDK/CLI-anrop (kör `chloros-cli login` en gång per maskin; cachas i `~/.chloros/`).
- Exemplen använder Linux-sökvägar; på Windows ersätt `/home/user/...` med `C:/Users/.../...`.

---

## Översikt på högsta nivå

```
chloros-cli [global options] COMMAND [command options]
```

### Globala alternativ

| Flagga | Beskrivning |
| --- | --- |
| `--backend-exe PATH` | Åsidosätt den automatiskt upptäckta bakgrundskörbara filen. |
| `--port N` | Port för backend-programmet HTTP (standard: `5000`). |
| `-v, --verbose` | Aktivera detaljerad utdata. |
| `--restart` | Tvinga omstart av backend (avslutar alla körande `backend_server.py`). |
| `--version` | Visa version (`Chloros CLI 1.2.0`). |
| `--help` | Visa hjälp på högsta nivå. |

### Kommandoindex

| Kommando | Syfte |
| --- | --- |
| [`process`](#chloros-cli-process) | Bearbeta en mapp med Survey3- eller LATTICE-inspelningar från början till slut. |
| [`login`](#chloros-cli-login) | Autentisera den här datorn med ett Chloros+-konto. |
| [`logout`](#chloros-cli-logout) | Rensa cachade inloggningsuppgifter. |
| [`status`](#chloros-cli-status) | Visa aktuell licens-/autentiseringsstatus. |
| [`export-status`](#chloros-cli-export-status) | Visar exportförloppet för Live Thread-4 under en `process`-körning. |
| [`language`](#chloros-cli-language) | Ställ in eller visa CLI-visningsspråk (38 stöds). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Standardprojektmapp (delad med GUI). |
| [`update`](#chloros-cli-update) | Sök efter och installera CLI-uppdateringar (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Systemdiagnostik + rökprov. |
| [`time-sync`](#chloros-cli-time-sync) | PTP-grandmaster-status / -kontroll. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-kamerastyrning och bildtagning (över 45 underkommandon). |
| [`daq`](#chloros-cli-daq) | Styrning av DAQ-spektralsensor (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Öppna och kör ett sparat Chloros-projekt (kameror + DAQ:er). |

---

## Installation

`chloros-cli` ingår i Chloros-installationsprogrammet för skrivbordet på alla plattformar som stöds — det finns ingen separat nedladdning av CLI. När du installerar plattformspaketet läggs `chloros-cli` till din `PATH` tillsammans med skrivbordsappen och den bakomliggande binärfilen som den använder.

Senaste nedladdningar: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Installationsprogrammet innehåller även praktiska startskript (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) som öppnar ett färdigt CLI-skal; dessa beskrivs i [CLI Användarhandbok](../CLI.md) och beskrivs inte separat här.

### Windows (.exe)

1. Ladda ner installationsprogrammet Windows från nedladdningssidan.
2. Kör `Chloros-Setup-x.y.z.exe` och följ guiden. Standardinstallationsvägen är `C:\Program Files\Chloros\` (CLI placeras i `C:\Program Files\Chloros\cli\`, som installationsprogrammet lägger till i PATH).
3. Öppna en ny terminal (`cmd.exe`, PowerShell eller Windows Terminal) så att den uppdaterade `PATH` hämtas.

```powershell
chloros-cli --version
```

Installationsprogrammet lägger automatiskt till `chloros-cli.exe` i ditt systems `PATH` och inkluderar Arena SDK-körmiljön som krävs för LATTICE-kameror.

### Linux amd64 (.deb)

För Ubuntu 22.04 LTS eller nyare / Debian-baserade x86_64-arbetsstationer.

> **Ubuntu 20.04 stöds inte.** Paketets beroendelista härleds från
> vad backend faktiskt länkar mot, och det inkluderar `libc6 (>= 2.34)`;
> focal levereras med glibc 2.31. `apt` avbryter installationen hellre än att låta den misslyckas vid
> körning.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

.deb-paketet installerar:
- `chloros-cli` till `/usr/bin/chloros-cli`
- Den kompilerade backend-modulen till `/usr/lib/chloros/chloros-backend`
- Arena SDK-körmiljön (för LATTICE-kameror)
- Denoiser-modeller, kalibreringspaket och konfiguration av uppdateringskanaler

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Samma uppbyggnad som amd64-deb-filen,deb, med en CUDA-byggnad anpassad för Jetson Orin / Orin NX / Orin Nano.

### Autentisera en gång per maskin

Varje plattform kräver en engångsinloggning med Chloros+ innan SDK/CLI-anropen ska fungera:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Inloggningsuppgifterna cachelagras i `~/.chloros/user_session.json`.

### Kontrollera installationen

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Chloros+-abonnemang krävs.**CLI kräver en aktiv Chloros+ -abonnemang.**Copper**är instegsnivån Chloros+ — alla betalda Chloros+-nivåer har tillgång till CLI/SDK; endast den kostnadsfria**Iron**-nivån som inte har det. (Plan-ID-tabell: `0`=Iron/gratis, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Uppgradera på [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Denna lägsta gräns upprätthålls av backend-systemet, inte bara av CLI: en SDK/CLI-flaggad begäran utan betald plan avvisas med `403 PLAN_UPGRADE_REQUIRED`, oavsett om den kommer från `chloros-cli`, Python SDK eller en egenutvecklad HTTP-klient. En utloggad användare får istället `401 AUTH_REQUIRED`. Åtkomsten fungerar offline under planens respitperiod (30 dagar per månad, till utgångsdatumet för årsabonnemang) och upphör när den löper ut; `chloros-cli status` fortsätter att fungera så att orsaken syns (det är den SDK/CLI-rutten som är undantagen från nivåbegränsningen — `GET /api/license-status`).

---

## Fem minuters snabbstart

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Bearbeta en mapp med bilder genom hela Chloros-pipeline (måldetektering → kalibrering → vignett → reflektans → indexexport).

### Översikt

```
chloros-cli process INPUT [OPTIONS]
```

### Positionsargument

| Argument | Beskrivning |
| --- | --- |
| `INPUT` | Sökväg till inmatningsmappen som innehåller `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) eller `.dng`-filer. |

### Allmänna alternativ

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `-o, --output PATH` | en ny mapp med tidsstämpel under din standardprojektväg (`~/Chloros Projects` om inte annat anges) | Projektmapp att skapa eller återanvända. Om mappen redan innehåller en `project.json`, skapas istället en `_1`/`_2` skapas istället för att skriva över den befintliga. |
| `-n, --project-name NAME` | auto (tidsstämpel) | Projektnamn. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` använder en Chloros+ neural debayer; långsammare men med högre kvalitet. |
| `--vignette / --no-vignette` | `--vignette` | Vignettkorrigering. |
| `--reflectance / --no-reflectance` | `--reflectance` | Reflektanskalibrering (använder panelmål om sådant finns, NIST-kalibrering per serienummer för LATTICE). För LATTICE multispektral fungerar detta även som **produkt** – se [Exportalternativ per produkt](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | av | Tillämpa PPK GNSS-korrigeringar från sidecar-filer. |
| `--exposure-pin-1 MODEL` | av | Fäst en Survey3 dubbelkamerariggens ”pin-1”-modell. |
| `--exposure-pin-2 MODEL` | av | Fäst ”pin-2”-modellen. |
| `--recal-interval SECONDS` | 0 | Tvinga omkörning av kalibreringsberäkningar var N:te sekund av inspelningstiden. |
| `--timezone-offset HOURS` | lokal | Åsidosätt tidszonsförskjutningen som är inbäddad i utdatametadata. |
| `--format FORMAT` | `TIFF (16-bit)` | En av `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | ingen | Vegetationsindex (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Tvinga pipelinens startpunkt för LATTICE TIFF-filer (Survey3 .raw påverkas inte). Även den bakdörr som gör att en inspelning utan **raw** kan bearbetas överhuvudtaget — se [Hur en inspelningsmapp ser ut](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | på | Skicka ut den linjära debayered-produkten (`Debayered_Images`). Se [Exportalternativ per produkt](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | på | Skickar ut förhandsgranskningen (`Preview_Images`): RGB = vitbalans (DAQ-ljuskälla om tillgänglig, annars gråvärld) + gamma; multispec = falskfärgssträckning. |
| `--radiance / --no-radiance` | på | Skicka ut strålningsvärde av typen float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referens för LATTICE-reflektansprodukten: `auto` = QA-godkänt mål inom ramen är den absoluta referensen, DAQ-nedåtriktad (ρ = π·L/E) som reserv; `target` = strikt (ingen DAQ-ersättning); `daq` = DAQ-auktoritativ. Se [Exportalternativ per produkt](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | ingen | Katalog med **mätta** målreflektansskanningar (`<serial>.csv`); faller tillbaka till de nominella T3/T4P-spektren vid avsaknad. |
| `--array-alignment / --no-array-alignment` | på | LATTICE-matriser: tillämpa modul-tillmodul-till-modul-inriktning som är angiven i varje bilds `Chloros:Alignment*` XMP-fil till varje bearbetad produkt (debayered / förhandsgranskning / radianse / reflektans / index). Ingen åtgärd för bilder utan taggarna. |
| `--array-alignment-crop / --no-array-alignment-crop` | beskärning | Beskär justerade exportfiler till matrisens gemensammaöverlappningsområde så att alla moduler delar samma fotavtryck; `--no-…` behåller hela sensorns arbetsyta (svart fyllning utanför källan). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Omprovtagning för justeringsförvrängningen. `nearest` bevarar exakta käll-DN-värden (ingen blandning av radiometriska värden mellan pixlar). |

### Alternativ för måldetektering

| Flagga | Beskrivning |
| --- | --- |
| `--min-target-size PIXELS` | Minsta panelmålstorlek (px) för detektorn. |
| `--target-clustering 0-100` | Klusterkänslighet. |
| `--target / --targets` | Behandla ingångsmappen som enbart målpaneler (hoppa över undersökningsdetektering). |

### Exempel

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Exportalternativ per produkt (LATTICE multispektral)

LATTICE-bearbetningen fördelas till **alla tillämpliga produkter i ett enda steg**. De fyratyp — `--debayered`, `--preview`, `--radiance`, `--reflectance` — är alla**aktiverade som standard**; använd formuläret `--no-<type>` för att inaktivera en. RGB-huvudkameror avger endast debayered + förhandsgranskning (ingen strålning/reflektion per band), så `--radiance`/`--reflectance` har ingen effekt för dem. Växlingsalternativen ignoreras för Survey3 och `.raw` (som följer standardvägen för reflektans/mål). *(Den gamla flaggan `--radiometric-output {reflectance,radiance,sensor-response}` har **tagits bort** och ersatts av dessa växlingsalternativ; det finns ingen `sensor-response`-nivå längre.)*

| Produkt | Utdata | Behövs DAQ-nedåtriktad strålning? |
| --- | --- | --- |
| `--debayered` | Linjär demosaik (`Debayered_Images`). | Nej. |
| `--preview` | Förhandsgranskning (`Preview_Images`): RGB = vitbalans + gamma; multispektral = falskfärgssträckning. | Nej. |
| `--radiance` | float32 W/m²/sr/nm från den fullständiga radiometriska kedjan (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 reflektans ρ (`32768` = 1,0), Pix4D-kompatibel. | **Ja**, såvida inte ett QA-godkänt mål inom ramen förankrar det (se nedan). |

`--reflectance-source` väljer reflektansreferensen:**`auto`**(standard) gör ett QA-godkänt mål inom bilden till den**absoluta referensen**— de målankrade empiriska linjekedjorna kors-värderas på uteslutna paneler och den uppmätta vinnaren tillämpas — med fallback till DAQ:s nedåtriktade delningsvärde (ρ = π·L/E) när inget mål finns eller QA misslyckas;**`target`**är strikt (ingen DAQ-ersättning);**`daq`**väljer bort det DAQ-auktoritativa beteendet. Målgeometrin (ArUco / fast ROI / remsa) hämtas från projektets målkonfiguration; `--target-reflectance-dir DIR` behåller**mätta** skanningar (`<serial>.csv`) som hämtas utifrån målenhetens serienummer/QR-kod, med de nominella T3/T4P-spektren som reserv.

DAQ-reflektansvägen löser automatiskt ut **tidsstämpelmatchnad nedåtriktad strålning**från en inspelad**`.daq`**(DAQ-U/M/E)**eller en DAQ-M-inbyggd `.csv`**som finns tillsammans med bildmaterialet. Om ett kalibreringspaket per kamera eller DAQ inte finns cachelagrat lokalt,**hämtar pipelinen det automatiskt från AWS** vid första användningen (kräver internetanslutning en gång; cachas under `~/.chloros/`).

#### Läsa reflektanspixlar (Pix4D / Metashape / dina egna skript)

Reflektansen lagras som ett heltal (DN), och **det DN-värde som motsvarar ρ = 1,0 beror på källkameran**:

| Källa | ρ = 1,0 är | Hur man avgör |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (utrymme upp till ρ 2,0) | XMP-taggen `Chloros:PixelScale=32768` är instämplad i filen. |
| Survey3 | `65535` (avskuren vid ρ 1,0) | Inga `Chloros:*` XMP-taggar — den avsaknaden *är* tecknet. |

**Läs av `Chloros:PixelScale` och dividera med det** istället för att anta en konstant. Taggen är definierad i uint16-domänen, så den förblir `32768` i utdataformat som skalar om — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` och `TIFF (32-bit, Percent)` är alla självbeskrivande (normalisera först den lagrade datatypen tillbaka till uint16: ×257 från 8-bit, ×65535 från float).

> **Ett fall har ingen skalning, enligt design.** När en 8-bitars källinspelning (BayerRG8) skrivs som 8-bitars TIFF, *klipper* pipelinen till 0..255 istället för att omskala, så varje värde över ρ≈0,008 fltill 255 och ingen skalning beskriver filen. Chloros utelämnar medvetet både `Chloros:PixelScale` och tuplen `MicaSense:RadiometricCalibration` där, och loggar varför. **Om taggen saknas i en LATTICE-reflektansfil, ska du inte anta någon skala — exportera om med 16-bitars eller 32-bitars** istället för att dela upp pixlar som aldrig var delbara.

#### EXIF överförs till exporten

`process` kopierar källbildens **GPS-block och dess ExifIFD** till varje produkt, så en
export innehåller `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` och
`CameraSerialNumber` tillsammans med georefereringen.

**`FocalLength` är inte valfritt för fotogrammetri.** Pix4D beräknar markprovavståndet utifrån
brännvidd plus höjd; om taggen saknas faller programmet tillbaka till en helt felaktig skala. Vid en
flygning över en apelsinlund med 49 bilder förvandlade den saknade taggen ett område på 411 m × 160 m till ett rekonstruerat
47,8 km × 13 km – en ortobild på 455 MP bestående mestadels av nodata, vilket sedan tolkades som ett problem med bildrutor och
ett BigTIFF-problem innan någon kontrollerade GSD. Om din ortofoto får en osannolik
skala, kör först `exiftool -FocalLength` på den exporterade produkten.

Kopian är avsiktligt **inte** `-all:all`: IFD0:ss strukturtaggar stör LATTICE-utdata när de
kopieras, och `ExifImageWidth` / `ExifImageHeight` utesluts eftersom de beskriver
*källans* bildtagning — en export som någonsin har ändrats i storlek skulle annars innehålla dimensioner
som strider mot dess egen raster. XMP skrivs direkt istället för att kopieras, eftersom ExifTool
kasserar XMP-taggar från samma anrop när XMP-blocket kopieras (vilket skulle leda till att kalibreringstaggarna MAPIR
förloras).

### Var utdata placeras

Produkterna skrivs **i projektmappen, grupperade efter kamera och sedan efter filformat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kameramappen är `LATT-<sensor>-<lens>-F<filter>` för LATTICE (vilket stämmer överens med bildens EXIF-data
`Model`) och `<model>_<filter>` för Survey3 — två kameror som delar sensor och filter men har olika
objektiv har separata träd, eftersom vinjettering, synfält och distorsion skiljer sig åt. Formatmappen
följer `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` eller `tiff32` till
`TIFF (32-bit, Percent)`.

> **Varje exporterad produkt behåller namnet på KÄLLFILEN.** En Radiance-export av
> `capture_…_raw.tif` heter fortfarande `capture_…_raw.tif` — den finns bara i
> `tiff32/Radiance_Images/`. **Mappen identifierar produkten, inte filnamnet**, så en glob-sökning
> efter `*radiance*.tif` ger inget resultat; sök istället i katalogen.

### Ljussensorinspelningar — kalibrerade `.daq` + `.csv`

`process` hanterar även `.daq`-inspelningarna i din inmatningsmapp, och det **inte**
behöver några bilder för att göra det: en DAQ-U / DAQ-M / DAQ-E som flygs på egen hand är en fullständig
insamling, och en mapp som endast innehåller `.daq`-filer är en giltig inmatning.

En DAQ kan spelas in **utan** sin kalibrering — det är vad de allmänna
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) inspelarna
(`record_daq.py`) gör som standard: de skriver ut råa sensorräkningsvärden och märker filen så att
Chloros hämtar den sensorns fabrikskalibrering **via serienummer** (först lokal cache,
sedan MAPIR Cloud) och tillämpar den. `process` skriver ut resultatet igen:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` innehåller en rad per avläsning: UTC-tidsstämpel, integrationstid, total effekt,
fotopisk/skotopisk lux, PPFD (och dess uppdelning i blått/grönt/rött), toppvåglängd, sedan det
fulla spektrumet på sensorns eget våglängdsraster. `.daq` importeras på nytt utan att
kalibreras en andra gång.

Vid lyckat resultat rapporterar körningen `Light-sensor products written: N (calibrated .daq + .csv)`.
Det som står inom parentes beskriver vad som faktiskt skrevs ut, så det lyder
`(RAW COUNTS — this sensor has no calibration bundle)` för en sensor utan paket och
`(N calibrated, M raw counts)` för en mapp som innehåller båda. Backendens egna
rubriker `[DAQ-EXPORT]` och `[RUN-SUMMARY]` härleder sin formulering på samma sätt — ingen av
de tre kan kalla en rå export för kalibrerad.

En DAQ-U-/DAQ-M-/DAQ-E-inspelning vars kalibreringspaket inte kan hämtas — du är
offline, eller så finns ingen kalibrering registrerad för den sensorn — **hoppas över med en anledning** på en
`[DAQ-EXPORT]`-raden och skrivs aldrig ut som en ”kalibrerad” fil som innehåller råa mätvärden.
Anslut till internet och kör om. Orsaken är den som läsaren faktiskt
fastställt för den filen (oläsbart schema, inget paket, ett skrivfel), och körningssammanfattningen
listar **skilda** orsaker — tjugo filer som hoppats över av en orsak visas som en
orsak, inte tjugo upprepningar av den.

#### DAQ-A-inspelningar exporteras som råa räknevärden

**DAQ-A**-familjen är äldre än systemet med bundlar per serienummer och har ingen kalibreringsbunt
att hämta — den kalibreras istället i fält mot ett reflektansmål, vilket är
anledningen till att den aldrig behövt någon. Att avvisa dessa inspelningar gjorde att de inte hade någon möjlighet att få ut sina
siffrorna ut alls, så de exporteras under ett **annat namn**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Ett annat filnamn istället för en flagga inuti filen, eftersom uppgiften måste klara av
att skickas vidare via e-post som ett rent namn. `.csv`-rubriken anger
`raw spectral sensor counts (NOT irradiance)` och varnar för att värdena är jämförbara
**inom** filen – vilket är precis vad målbaserad kalibrering använder dem till – och
inte mellan olika sensorer. De effektberoende fotometriska kolumnerna (total effekt, fotopisk och
skotopisk lux, PPFD) skrivs ut som **NULL** istället för att integreras från räknevärden, och
sammanfattningen anger `RAW COUNTS`, så ”exporterade” värden i en logg kan inte tolkas som irradians.

Äldre **v1.01 / v1.02**-inspelningar (en DAQ-A-SD skriver dessa) innehåller ingen epok per avläsning,
utan endast filens skrivtid. Bild↔nedåtriktad matchare avvisar dem fortfarande — att matcha en
bildram mot en skrivtid skulle ge osynliga fel — men exportören läser dem, och
CSV skriver ut `clock=daq_created_on` så att produkten anger vilken klocka den använder.

### Anmärkningar

- `process` upptäcker automatiskt om din mapp är Survey3, LATTICE eller en blandning.
- Förloppsströmmar via Server-Sent Events; CLI visar realtidsförlopp per tråd (detektering, analys, bearbetning, export).
- För Linux/Jetson kontrollerar CLI swap-minnet och kan visa en varning innan bearbetning av stora mappar. Den texturmedvetna debayern tillämpar också automatiskt en GPU-frekvensbegränsning på Jetson-enheter med låg strömförbrukning (Nano, Orin Nano).
- Vid lyckat resultat rapporterar körningen hur många bildprodukter den skrev (`Image products written: N`).

#### En körning som inte skriver ut några bilder misslyckas

Om du begärde bildprodukter och körningen skrev ut **inga** — endast `project.json` och
`calibration_data.json` — behandlar `process` detta som ett misslyckande: det skriver ut
`Processing finished but wrote no image products.` och **avslutas med ett värde större än noll**, så att ett skript kan
upptäcka det. Meddelandet anger projektmappen och de vanligaste orsakerna:

- inmatningsmappen kändes inte igen som en inspelning (kontrollera layouten och `--input-level`), eller
- alla begärda produkter hoppades över eftersom de inte var tillämpliga för dessa kameror (t.ex. vid begäran om
  radianse/reflektans från kameror som endast stöder RGB).

Kör om med `--verbose` och kontrollera backend-loggen efter rader med `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
som förklarar utelämnanden per kamera som annars inte når ut till CLI:s utdata.

En avsiktlig körning med enbart metadata – alla produkter avstängda och utan `--indices` – är fortfarande en
**framgång**, eftersom en tom bildutmatning är det korrekta resultatet i det fallet.

Detsamma gäller en **körning enbart med ljussensor**: en mapp med `.daq`-inspelningar har per definition inga bilder att exportera
, och körningen bedöms utifrån de kalibrerade `.daq` / `.csv`-filerna som den skrev istället.

---

## `chloros-cli login`

Autentisera den här datorn med ett Chloros+-molnkonto. Inloggningsuppgifterna cachelagras säkert i `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Exempel

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (genom att ta bort, eller duplicerar delar av lösenordet). Vid ett 401-fel gör CLI automatiskt ett nytt försök med `$$` tillagt igen, sedan med en avduplicerad halva av lösenordet; om ett nytt försök lyckas loggar det in dig in och visar den korrekta syntaxen med enkla citattecken som ska användas nästa gång.

> **Användning utan grafiskt gränssnitt/med skript: ingen cachad session innebär en interaktiv prompt, inte ett snabbt misslyckande.** Alla kommandon som startar en backend (`process`, `status`, `export-status`, `time-sync`, …) som körs utan en cachad licens/session hamnar i en interaktiv `Email:` / `Password:`-prompt på stdin innan de fortsätter. Ett obevakat jobb utan cachelagrad session kommer därför att hänga sig i väntan på inmatning — kör `chloros-cli login EMAIL PASSWORD` en gång per maskin innan du schemalägger headless-arbete.

---

## `chloros-cli logout`

Rensar den cachelagrade sessionen och tvingar fram en ny inloggning vid nästa anrop.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Visar aktuell licensnivå (Iron/Copper/Bronze/Silver/Gold), autentiserad användare och antal enhetsbindningar.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Hämtar aktuell status för Thread-4-exporten. Kan säkert anropas **under** en `process`-körning från ett annat skal.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Ställ in visningsspråket för CLI (38 språk stöds, inklusive CJK, RTL och indiska språk). Faller smidigt tillbaka till engelska på äldre konsoler som inte kan återge skriptet.

```
chloros-cli language [LANG_CODE] [--list]
```

### Exempel

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Kommandon för projektmapp

Dessa hanterar standardplatsen för projektmappen (delad med GUI).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux / Endast Jetson. Kontrollerar `version_url` från `/etc/chloros/update.conf` och erbjuder att ladda ner och installera den matchande versionen `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

På Linux/Jetson utför även CLI en **automatisk uppdateringskontroll vid varje uppstart** (icke-blockerande, fördröjer aldrig kommandot): den läser `/etc/chloros/update.conf`, cachar resultatet i 1 timme i `~/.chloros/update_cache.json` och visar `Update available: vX.Y.Z / Run: chloros-cli update` när en nyare version finns. Hoppa tyst över vid eventuella fel och i Windows.

---

## `chloros-cli selftest`

Kör ett rökprov i sju steg: version, porttillgänglighet, uppstart av backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), förekomst av brusreduceringsmodell, CUDA+brusreduceringsberedskap.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP-grandmaster-status och -styrning. Värden Chloros kör PTP-grandmastern; LATTICE-kameror och DAQ-E-enheter fungerar som slavar till den för tidsstämplar mellan enheter.

| Underkommando | Beskrivning |
| --- | --- |
| `status` | Visa grandmaster-status, BMCA-prioriteringar och klockidentitet. |
| `peers` | Lista över slavar som upptäcks via Delay_Req (kameror + DAQ-E-sensorer). |
| `cameras` | PTP-status per kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Starta om grandmaster-processen. |
| `set-priority --priority1 N --priority2 N` | Åsidosätt BMCA-prioriteringar. |

### Exempel

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE-kamerastyrning. Varje underkommando dirigeras via Chloros-backend; backend äger kamerapoolen, så efterföljande CLI-anrop återanvänder samma öppna handtag.

### Vanliga alternativ (gemensamma för de flesta underkommandon)

| Flagga | Beskrivning |
| --- | --- |
| `-d, --device N` | Kameraindex (standard: 0). |
| `-s, --serial SN` | Specifikt serienummer; åsidosätter `--device`. |
| `--serials SN1,SN2,…` | Serienummer separerade med kommatecken för drift med flera kameror. |
| `--all` | Kör på alla upptäckta kameror. |
| `--exposure US` | Exponeringstid i mikrosekunder. |
| `--gain DB` | Förstärkning i dB. |
| `--pixel-format FMT` | t.ex. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Bilddimensioner. |
| `--preset {default,high_quality,high_speed,triggered}` | Tillämpa en förinställning. Alla körs fritt förutom `triggered`, som aktiverar kameran vid en hårdvarusignal på linje 2 – om ingen signal skickas till den linjen kommer den att vänta i evighet istället för att ta en bild. |
| `-o, --output DIR` | Utmatningskatalog (standard: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-paketstorlek. `auto` kör ICMP+GVSP-sonder; `jumbo` = 9000; `standard` = 1500. |

### Arbetsflöde för första anslutning av LATTICE-kameror

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referens för underkommandon

#### Upptäckt och information

| Underkommando | Syfte |
| --- | --- |
| `lattice info` | Visa lista över anslutna kameror (tillverkare, modell, serienummer, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analysera värdsystemet för optimal kamerakonfiguration. `--no-discover` hoppar över kameraupptäckten (snabbare, endast NIC-analys). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Kontrollera/korrigera nätverkskortets inställningar; uppskatta bandbredd/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Nätverkskapacitet för backend med stabilt schema + rekommendation om array (returnerar `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` behåller den begärda upplösningen men begränsar mål-FPS — läs `recommended.recommended_target_fps` och skicka det som anslutningsmål; betrakta det som en framgång, inte som ett fel. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Vad-händer-om-analys utan att öppna kameror. **`--n-active` är det totala antalet kameror på nätverket, inte bara i denna matris**— öka det när fristående kameror strömmar samtidigt, eller när nätverksbudgeten beräknas utifrån en efterfrågan som underskattar antalet (standard: `len(--models)`). Visar alltid det sammanlagda `Wire budget:` (MB/s som krävs jämfört med kollisionssäkert tak) och `Max cameras:`, och markerar `** OVER-SUBSCRIBED**` när arrayen överbelastar nätverket — se [Array fps &amp; burst-modell](#array-fps--burst-model). |
| `lattice gpu` | Visa GPU-status. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Kontrollera eller uppdatera kamerans firmware. Det lokala valet av `.fwa` är fastställt: filen i `firmware/<MODEL_PREFIX>/` som matchar byggets `MIN_FIRMWARE_VERSION` flashas när den finns (endast den högsta versionen som reserv), så en nyare leverantörsbild som lagrats på disken är inaktiv tills den fästningen ändras — avsiktligt når nyare utgåvor enheterna via det signerade AWS-manifestet, vilket är att föredra när nyare. |
| `lattice presets [--apply NAME]` | Visa eller tillämpa kamerainställningar. |
| `lattice status` | Kamerastatus i realtid. |

#### Inspelning

| Underkommando | Syfte |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Enstaka bildruta. **Sparar alla exporttyper som standard** (`--processing all`); se [Exportnivåer för bildtagning](#capture-export-levels-the-all-default). `--levels` sparar en explicit delmängd (åsidosätter `--processing`); `--force-daq` skriver det tilldelade DAQ-värdet som en `.daq`-sidecar även vid en-only-inspelning. `--jpeg-quality` = JPEG kvalitet 1–100 (standard 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Strömma till disk tills Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Webbläsarbaserad live-förhandsvisning i MJPEG. `--ae-damping` ställer in dämpning av automatisk exponering (0,4–100). |

#### Sensorkalibrering

| Underkommando | Syfte |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Läs/skriv valfri GenICam-nod. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Exponering &amp; AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Förstärkning och automatisk förstärkning. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Sensor-ROI och binning. |
| `lattice format [--set FMT] [--list]` | Pixelformat. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hårdvaru-/mjukvarutrigger. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (inga flaggor = engångs-vitbalans) | Vitbalansoperationer. Endast RGB/Bayer-kameror; en no-op (hoppas över) på mono-M3M. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB visar färgpipeline. `natural` (standard) är den billiga live-bearbetningen; `enhanced` lägger till defringe + vibrance + CLAHE lokal kontrast för det fullständiga hub-parity-utseendet till ungefär dubbla bearbetningskostnaden per bildruta, vilket ger en lägre **live**-bildfrekvens — sparade inspelningar får alltid den fullständiga bearbetningen oavsett. Endast RGB/Bayer-kameror; hoppas över på mono M3M. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Visa mättnad/kontrast (RGB-filterkameror). Hoppas över på mono M3M. |
| `lattice filter [--set NAME] [--list]` | Ställ in kamerans filtermodell (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Mät ström-/värmenoder; växla till lågströmsläge. |

#### Kalibrering och sensorer

| Underkommando | Syfte |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrera utifrån ett reflektansmål. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Inbyggda kommandon för sensor för nedåtriktat ljus. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Tillämpa vignetteringskorrigering på befintliga bilder. |

#### Flera kameror (transienta sessioner)

| Underkommando | Syfte |
| --- | --- |
| `lattice multi-info` | Lista alla kameror med synkroniseringsroller. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | En synkroniserad bildruta från varje kamera. Sparar **alla exporttyper som standard**när en permanent matris är ansluten; är den tillfälliga reservlösningen utan array**endast debayered** (kör `array-connect` först för resten). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Strömma synkroniserade bildrutor (tillfällig). |
| `lattice multi-test [--count N]` | Test av GPIO-synkroniseringstid. |
| `lattice multi-detect [--line LINE] [--json]` | Automatisk detektering av GPIO-master/slav-koppling. |

#### Justering

| Underkommando | Syfte |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — plus reglage för detektor/matchare `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-reglage `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombination av flera bildrutor `[--averaging mean\|median\|inlier_weighted]`, geometriska begränsningar `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, rumsliga begränsningar `[--roi X0,Y0,X1,Y1] [--mask PATH]` och överskrivningar per slav `[--per-cam-override SN:KEY=VALUE]` (upprepningsbar) | Beräkna inriktningsprofil från livekameror. `--prefilter` har som standardinställning `gradient` (kantkarta; överensstämmer med GUI/array-inriktaren — kanterna bevaras över spektralbanden). `--matcher flann` lönar sig vid över ~5000 drag; `--averaging median` är robust mot en dålig bildtagning, `inlier_weighted` viktar efter antal matchningar; `--lock-scale` projicerar till närmaste rotation (ingen skalning), `--lock-axis` nollställer en translationskomponent; `--mask` gäller för alla kameror (använd `--per-cam-override` för kameraspecifika inställningar, t.ex. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` vägrar att spara en kalibrering vars RMS-värde för omprojektionen överskrider gränsvärdet. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Ta en inriktad multibandsbild. `--bit-depth` anpassar sig som standard till kameran; `--no-crop` behåller hela bilden (fyller ut med svart); `--interpolation` (standard `linear`) och `--border-mode`/`--border-value` (standardinställning `constant`/0) styr CPU-warp – GPU-banan är oavsett detta bilineär. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Strömjusterade flerbandsramar (samma warp-reglage som `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Visa profilinformation. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Ändra lagerordning. |

#### Index / Vegetationsberäkningar

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Fullständig flagguppsättning: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (upprepningsbar), `--capture-level raw|debayered|radiance|reflectance|unknown` (åsidosätter den inspelningsnivå som registrerats i källan TIFF; standard: läses från TIFF-metadata), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Med `--live` gäller även justeringsreglagen för vridning: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **`--channel`-symbolerna är skiftlägeskänsliga.** Symbolsidan måste stämma exakt överens med förinställningarnas kanalnamn (förinställningarna använder gemener, t.ex. NDVI = `red`,`nir` — kontrollera `--list-presets`), och bandsidan måste stämma överens med ett bandnamn i den anpassade stapeln (eller vara ett 0-baserat bandindex i offline-läge). `--channel red=Red_660 --channel nir=NIR_850` fungerar; `--channel RED=660` misslyckas med ett `channel_map missing entries`-fel.

#### Persistenta anslutningar (Smart-Prep, GUI-motsvarande flöde)

Dessa kommandon håller kameror öppna i backend-poolen över flera CLI-anrop.

| Underkommando | Syfte |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Lägg till en kamera i poolen (enstaka kamera, ingen matris). |
| `lattice cam-disconnect [--serial SN] [--all]` | Frigör. |
| `lattice cam-list` | Visa kameror i poolen. |
| **`lattice array-connect`**|**Anslut en beständig synkroniserad array (DEN rekommenderade startpunkten).** Kör hela smart-prep-flödet i GUI. |
| `lattice array-disconnect [--array-id ID] [--all]` | Frigör en array. |
| `lattice array-list` | Visa anslutna arrayer. |
| `lattice array-status [--array-id ID]` | Live-fps, PTP, senaste fel. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | En synkroniserad inspelning från live-matrisen — Enstaka / Kontinuerlig / Intervall / Snabbaste. **Standardinställningen är `all`** (en fil per tillämplig exporttyp per kamera). Kameror som hoppats över (t.ex. RGB, som exkluderats från strålningsintensitet/reflektans) rapporteras med `Skipped: SN:<serial> (<reason>)`; den DAQ-avläsning som används för reflektans sparas tillsammans med och rapporteras med `DAQ: <path>`. Se [Inspelningslägen, inspelare och offline-bearbetning](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Spela in den kombinerade indexvyn i realtid som video/GIF (övervakningskvalitet; kräver att den kombinerade strömmen är öppen). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Raw-Bayer-bildserie med hög bildfrekvens (analyskvalitet; ombearbetas offline). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Bearbeta en sparad råbildserie till kalibrerad(a) video(er). |

##### `array-connect`-alternativ

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatisk upptäckt av alla LATTICE-kameror (kräver ≥2) | Den första i serien är MASTER. Om detta utelämnas filtreras upptäckten till LATTICE (`TRI032*`) och ansluter alla. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkroniseringslinje. |
| `--target-fps F` | auto | Master-triggers avfyrningshastighet. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Åsidosätt nivåväljaren. |
| `--wire-ceiling-mbps MB_PER_S` | automatisk detektering | **Värdens kontinuerliga bandbreddsbudget, i MB/s — det värde som hela arraytilldelningen hänger på.** Sänk det när arrayen rapporterar GVSP-korrupta ramar: det automatiska värdet härleds från nätverkskortets angivna länkhastighet, vilket överskattar USB-adaptrar, smala PCIe-banor och belastade delade nätverk. Lagras i projektets array-insamlingsblock, så en återöppning / CLI / SDK återställer det. Se [Arrayhälsa](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Hårdvarubinning. |
| `--no-recommend` | off | Hoppa över nätverksanalyssteget. |
| `--no-ptp` | av | Inaktivera PTP (tidsstämplar mellan kameror är då **inte** jämförbara). |

### Smart-AE / Smart-Capture

LATTICE-matriser kör kontinuerlig AE i bakgrunden så snart de är anslutna, men det tar en stund för en nyinställd scen att konvergera. `array-capture --smart` är den **praktiska lösningen**: den väntar tills AE har stabiliserats på alla kameror i arrayen och utlöser sedan bildtagningen. Använd den när du byter scen mitt i en session.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Standardinställningen för stabiliseringspolicyn är konservativ: 5 sekunders timeout, 1,5 sekunders stabilitetsfönster, ±5 % tolerans för exponeringsspridning. Justera via SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) om du behöver ett annat beteende från automatiseringen.

### Exportnivåer för bildtagning (standardinställningen för `all`)

Från och med denna version är `lattice capture`, `lattice multi-capture` och `lattice array-capture` **som standard inställda på `--processing all`** — en sparad fil per exporttyp som gäller för varje kamera, vilket motsvarar beteendet ”Capture All” i användargränssnittet. Nivåerna är:

| Nivå | Utdata | Gäller för |
| --- | --- | --- |
| `raw` | Enkanalig Bayer (monokameror: det enda bandet) direkt från sensorn. | Alla kameror. |
| `debayered` | 3-kanals BGR-demosaik (monokromkameror: 1-kanals gråskala). | Alla kameror. |
| `radiance` | float32 W/m²/sr/nm via hela den radiometriska kedjan. | Endast multispektrala (M3C/M3M) — **hoppas över för RGB-filterkameror**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-kompatibel. | Endast multispektralt, och **endast när en DAQ är bunden + kameran är kalibrerad**; annars hoppas över. |
| `preview` / `display` | Fullständig GUI-förhandsgranskningskedja (CCM + WB + gamma enligt kamerans profil). `lattice capture` namnger detta `preview`; `array-capture`/`multi-capture` använd `display`. | Alla kameror. |

Ange en enda nivå för att spara just den (`--processing debayered`). När du begär `all` hoppas nivåer som inte gäller för en viss kamera över (och rapporteras), men ger inte fel – en frånkopplad eller okalibrerad kamera får fortfarande `raw` / `debayered` / `preview`.

För varje reflektansbild skrivs den DAQ-avläsning av nedåtriktat ljus som faktiskt används till en **`.daq`**-sidokar bredvid bilden (så att inspelningen kan bearbetas på nytt senare) och redovisas på en `DAQ:`-rad.

### Hur en inspelningsmapp ser ut

Varje exporttyp hamnar i sin **egen undermapp** under `-o`, så en inspelning med flera nivåer blandar aldrig ihop olika typer:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` är inspelningens tidsstämpel och `<serial>` kamerans serienummer, så en synkroniserad grupp delar en
tidsstämpel mellan kamerorna. **Observera den enda asymmetrin:** nivån `display` lagras i en mapp
med namnet `preview/` medan filerna själva behåller `_display` i namnet — mapp och suffix skiljer sig åt
endast för den nivån. Okända nivåer faller tillbaka till en mapp med sitt eget namn, och om undermappen
inte kan skapas skrivs filen till utdatarotkatalogen istället för att gå förlorad.

**Ombearbetning av en inspelningsmapp:**peka `chloros-cli process` mot**rotkatalogen för captures**
(`output/`). `process` importerar normalt endast den mapp du anger, men när den mappen inte innehåller några
bilder men har undermappar går den automatiskt ned i hierarkin — så rotmappens undermappar på samma nivå och själva
rotmappen `.daq` hämtas alla på en gång. Varje nivå i en bildserie importeras som en enda bild, där
de andra nivåerna är tillgängliga som lägen, istället för en bild per nivå.

Det går också att namnge en **undermapp på en nivå** direkt (t.ex. `output/raw/`). Om du gör det lämnas roten
`.daq` kvar, så kopiera eller peka in DAQ-avläsningen tillsammans med den när duhärleder en radiometrisk
produkt från `raw/` — annars har tidsstämpeln inget att jämföra mot.

**Bearbetningen börjar alltid från `raw`.** Inom varje inspelning är den råa bilden källan till bearbetningskedjan;
`debayered`, `radiance`, `reflectance` och `preview` ingår som visningslägen men matas aldrig
tillbaka genom pipelinen. Om en härledd produkt bearbetas på nytt skulle vignettering, CCM och
strålningsberäkningar som redan är inbakade i dess pixlar, så Chloros avvisas istället för att
bearbetas två gånger. Två konsekvenser som är värda att känna till:

- Renderingarna `index/` och `composite/` **bearbetas aldrig** bearbetas. De är utdata, inte inspelningar —
  en NDVI LUT-rendering har ingen meningsfull radiance-tolkning.
- En inspelningsmapp som exporteras **utan** `raw` (t.ex. `array-capture --processing reflectance`) har
  ingen giltig pipelinekälla. Dessa inspelningar importeras och visas normalt, men `process` hoppar över
  dem och meddelar detta:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Om du verkligen behöver driva igenom en härledd produkt – en hubbsession som fångats in med
  `demosaic` aktiverat, eller en äldre mapp – tvingar `--input-level {raw,debayered,processed}` in
  ingångspunkten och åsidosätter hoppet. Den flaggan är den avsiktliga nödutgången; `auto` (standard)
  bearbetar aldrig en inspelning som saknar rådata.

### Hoppade över inspelningar i arrayer med blandade filter

När du blandar RGB och multispektrala kameror i ett array sparar `array-capture --processing radiance` (eller `reflectance`) de multispektrala bilderna och **hoppar över** RGB-kamerorna — strålningsvärdet per Bayer-pixel är inte meningsfullt för en bredbandssensor. CLI skriver ut varje sparad fil (med dess exportnivå), varje `.daq` som skrivits och varje hopp explicit, så är antalet filer inte överraskande:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Hoppa över-orsakstoken följer mönstret `<level>-not-applicable-to-rgb-cam`. Reflektans kan också hoppas över med `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, och med `dls-uncalibrated-band-<nm>` när bandet till största delen ligger utanför DAQ-ljussensorns radiometriskt kalibrerade intervall (~374–974 nm) – bland de levererade SKU:erna är det endast F988, vars stödda arbetsflöde är arbetsflödet för reflektanspaneler.

Använd `--processing debayered` (eller `display`) för att inkludera alla kameror oavsett filtertyp, eller standardinställningen `all` för att få alla tillämpliga nivåer per kamera i ett enda steg.

---

## Inspelningslägen, inspelningsenheter och offline-bearbetning

Alla dessa fungerar på en **persistent array** (kör `array-connect` först). De speglar GUI-panelen för inspelning.

### `array-capture`-lägen

`array-capture` är ett enda kommando med fyra slutarlägen samt en uppsättning exportalternativ:

| Läge | Flagga | Beteende |
| --- | --- | --- |
| **Enstaka** *(standard)* | (ingen) | En synkroniserad inspelningsgrupp, därefter avslutas. |
| **Kontinuerligt** | `--continuous` | På varandra följande genomgångar tills `Ctrl+C`, `--count N` eller `--duration S`. |
| **Intervall** | `--interval S` | Ett pass var `S` sekunder (räknat från början av varje pass), samma gränser. |
| **Snabbaste** | `--fastest` | Endast rådata + det tilldelade DAQ-värdet + den kombineradeindexkomposit; hoppar över beräkningarna för strålning/reflektans/visning så att bilden visas snabbt. Innebär `--processing raw --force-daq`. Bearbeta de sparade `.daq`-data till kalibrerade produkter senare. |

Exportalternativ (kan kombineras med valfritt läge; alla delar samma GUI/SDK-ändpunkt):

| Flagga | Effekt |
| --- | --- |
| `--processing LEVEL` | En enda exportnivå, eller `all` (standard). |
| `--levels L1,L2,…` | Explicit delmängd av exporttyper (t.ex. `raw,radiance,reflectance`); **åsidosätter `--processing`**. |
| `--aligned` / `--no-aligned` | Warpa varje medlems icke-råa export till arrayens [justeringsprofil](#alignment) (samregistrerad). Rådata förblir oförvrängda men bär med sig transformationen i metadata. Faller tillbaka till ojusterad (med en varning) om arrayen saknar profil. |
| `--index` / `--no-index` | Spara / hoppa över överlägget med vegetationsindex per kamera där ett sådant är konfigurerat. Standard: rendera det. |
| `--force-daq` | Spara den tilldelade DAQ/DLS-avläsningen som en `.daq`-sidokar även om ingen vald nivå kräver det (t.ex. en inspelning som endast innehåller rådata), så att bildrutorna kan bearbetas om till reflektans/index offline. |
| `--smart` | Vänta tills AE har stabiliserats på alla kameror innan utlösning (se [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF-pixelkomprimering. `deflate` (standard) = förlustfri zlib L1 + horisontell prediktor, ~4,1 MB per bildruta i full upplösning; `none` = okomprimerat, ~5× snabbare skrivhastighet vid ~6,3 MB per bildruta — använd för maximal kontinuerlig hastighet när hårddisken tillåter det. Båda är förlustfria och läses identiskt vid import. |

> **Enskild skrivning TIFF + modellen för kontinuerlig hastighet.**Bilderna skrivs i**ett**TIFF-filpass som innehåller pixlar + XMP + IFD0 Märke/Modell (uppmätt på full upplösning Mono12: 36 ms komprimerat / 6,5 ms okomprimerat, jämfört med ~148 ms för den gamla metoden med skrivning följt av omskrivning med ExifTool); det enda återstående arbetet med ExifTool (finjustering av EXIF-sub-IFD) körs i en asynkron bakgrundsprosess, och en bild är färdig och klar för import även om den aldrig körs. Observera att DEFLATE-komprimering håller Python GIL, så komprimerade skrivningar kan**inte**parallelliseras över de enskildakamera-skrivtrådar — kontinuerlig inspelning med 8 kameror i full upplösning vid sensorhastighet (~10,4 fps) kräver `--compression none`**och** en disk av NVMe-klass (~500 MB/s kontinuerlig skrivhastighet). Samma inställning exponeras som `compression` på `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — kombinerad indexering av video/GIF (övervakningsklass)

Spelar in allt som **den kombinerade live-vyn** visar till en `.avi` (och valfritt en `.gif`). Eftersom den hämtar från den sammansatta live-bilden måste den kombinerade strömmen vara öppen (t.ex. att matrisen förhandsgranskas i GUI:n) för att bildrutorna ska kunna registreras. Den avläser förloppet var 2:e sekund och avslutas vid `--duration`, `Ctrl+C` eller när inspelaren avslutas automatiskt.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `--array-id ID` | endast array | Målarray (uteslut om endast en är ansluten). |
| `-o, --output DIR` | `output` | Utmatningskatalog (backend-lokal). |
| `--fps F` | `10` | Inspelningsbildfrekvens. |
| `--duration S` | tills Ctrl+C | Stoppas automatiskt efter `S` sekunder. |
| `--gif` | av | Skriv även en animerad GIF. |
| `--gif-only` | av | Skriv endast en GIF (ingen `.avi`). |

### `array-burst` — rå-Bayer-bildserie med hög bildfrekvens (analysklass)

Läser direkt från synkroniserad gruppbuffert i bildserien — **ingen kalibreringskedja, inget exiftool, ingen livevisning behövs** — så det körs med kamerans fulla bildhastighet. Skriver råa bildrutor + ett manifest per bildruta + en `.daq` per distinkt DLS-avläsning under `<output>/bursts/<base>/`. Bearbeta om offline (nästa kommando), eller skicka `--build` för att göra det omedelbart vid stopp.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `--array-id ID` | endast array | Målarray. |
| `-o, --output DIR` | `output` | Utdatakatalog (burst hamnar i `<DIR>/bursts/<base>/`). |
| `--duration S` | tills Ctrl+C | Automatisk stopp efter `S` sekunder. |
| `--max-frames N` | obegränsad | Automatisk stopp efter `N` råa bildrutor. |
| `--build` | av | Efter stopp, bearbeta omedelbart bursten på nytt (samma som `array-build-video`). |
| `--products …` | `combined:index` | Med `--build`: vilka videoklipp som ska skapas (se nedan). |
| `--fps F` | `10` | Med `--build`: bildhastighet (fps) för utgångsvideo. |
| `--save-tiffs` | av | Med `--build`: spara även bildkalibrerade TIFF-filer för varje bildruta. |
| `--gif` | av | Med `--build`: skriv även animerade GIF-filer. |

### `array-build-video` — ombearbeta en sparad bildserie offline

Tidsmatchar varje råbild till närmaste sparade `.daq`-avläsning och kör den genom **samma strålnings-/reflektans-/indexkedja som importpipeline**, vilket resulterar i en eller flera videor.

`--products` är en kommaseparerad lista med `kind:level`-objekt, där `kind` ∈ `per_cam` | `combined` och `level` ∈ `radiance` | `reflectance` | `index`. Ett `level` (ingen `kind:`) har som standardvärde `per_cam`. Standardvärdet är `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `--burst-dir DIR` | (obligatoriskt) | Sökväg till burst-mappen (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level`-lista, enligt ovan. |
| `--fps F` | `10` | Utgångsvideons bildhastighet (fps). |
| `--save-tiffs` | av | Spara ävenramkalibrerade TIFF-filer tillsammans med videon/videorna. |
| `--gif` | av | Skriv även animerade GIF-filer. |

> **Välj rätt inspelningsprogram.** `array-record` är *övervakningsklass* — den spelar in den sammansatta livebilden precis som den visas och kräver att strömmen är öppen. `array-burst` → `array-build-video` är *analysklass*-klass* — den sparar råa sensordata med full hastighet och rekonstruerar därefter kalibrerade videor av strålningsintensitet/reflektans/index, utan att någon livevisning krävs.

### Mono (M3M) enkelbandskameror

**M3M**-serien är den monokroma motsvarigheten till Bayer**M3C**: ett smalbandigt interferensfilter per kamera (`M3M-<lens>-F<wavelength>`, t.ex. `M3M-L87-F685`), så sensorn levererar ett**enda gråskaleband** utan Bayer-mosaik. Det finns inget att avmosaikera, ingen kanalöverskridande överhörning att separera och ingen vitbalans att ställa in — hela RGB-färgbehandlingskedjan gäller helt enkelt inte.

Vad det innebär för CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**upptäcker en monokamera och**hoppar över den med ett enradigt meddelande** istället för att tillämpa meningslösa inställningar. De fungerar fortfarande normalt med en RGB/Bayer M3C-kamera i samma session.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** fungerar fortfarande — strålningsintensitet och reflektans är *bandvisa* radiometriska kartor och är perfekt definierade för ett band. Monobilder har en **identitets**-sensorresponsmatris (ingen 3×3-avblandning), så planet passerar genom kalibreringsberäkningarna oförändrat.
- **En enskild monokamera kan inte generera ett vegetationsindex.**NDVI/NDRE/etc. kräver minst två band (t.ex. Red + NIR). För att få ett index från monohårdvara ska du rikta**flera** M3M-kameror mot olika våglängder, sammanfoga dem till en multibandsstapel och indexera *den*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel`-symbolerna måste **exakt** stämma överens med förinställningens kanalnamn (skiljer på versaler och gemener; NDVI är gemener `red`, `nir` — se `--list-presets`), och bandnamnet anger ett band i den anpassade stapeln (offline-läget accepterar även 0-baserade bandindex, t.ex. `--channel red=0 --channel nir=1`).

Det som skiljer dem åt i hela stapeln är tokenet `M3M` i modellsträngen (det förekommer aldrig i en `M3C`-sträng), som visas i GUI:n som `is_mono`.

---

## Konfiguration och inställning av värd-NIC (LATTICE-uppsättningar)

LATTICE-kameror strömmar GVSP via värdens Ethernet-kort, så för uppsättningar med flera kameror är kortets**drivrutin**och**mottagningsringens storlek** lika viktiga som länkhastigheten. Felaktiga inställningar visas som en `FRAMES WILL DROP`-/`Reduce ROI to enable`-port i panelen Arrayinställningar (och i `lattice network-analysis` / SDK:s `analyze_array_network()`), även när kamerorna i sig fungerar som de ska.

### USB 10GbE-kort — Realtek RTL8157 (&quot;Realtek USB 10GbE Family Controller&quot;)

| Post | Nödvändigt värde | Varför det är viktigt |
| --- | --- | --- |
| **Drivrutinsversion**|**≥ v10.67 (jan 2026)**, INF `rtump64x64sta.inf` | Den äldre**2016**-drivrutinen (v10.65, `rtump64x64.inf`) hanterar avstängning felaktigt och orsakar buggarmed**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**vid avstängning/omstart/viloläge. Övergången hänger sig (~5 minuters timeout), användaren tvingar fram en avstängning, och upprepade oriktiga avstängningar**skadar WMI-databasen**(PowerShell/verktyg börjar misslyckas med `Invalid class`) och**blockerar USB-stacken** vid nästa uppstart (nätverkskortet aktiveras inte; USB-enheter slutar att räknas upp). Uppdatera från realtek.com (eller dongelleverantören) innan du förlitar dig på korrekta omstarter. |
| **Mottagningsbuffertar**— nyckelord `ReceiveBufferLen` |**256**(drivrutinens max) | Nätverkskortets RX-ring. Drivrutinens standardvärde på**32**lämnar endast ~0,26 MB användbar ring – alldeles för liten för en multikamera-burst – så arraypanelen rapporterar `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` och anslutningarna blockeras. Vid**256**är ringen stor (**~13,5 MB uppmätt på labbets 10GbE-värd**), vilket ger RX-pipeline verkligt utrymme för GVSP-burst med flera kameror. (Huruvida en given konfiguration faktiskt *ansluts* avgörs av två kontroller — den **drain-aware**tillträdeskontrollen och kontrollen av**aggregerad överteckning** — inte en ren jämförelse mellan burst och ring; se [Array fps &amp; burst-modell](#array-fps--burst-modell).) |
| **Mottagande URB:er**— nyckelord `PendingReceives` |**64** (max) | USB-begäranblock under överföring; ökas tillsammans med mottagningsbuffertar för att absorbera burstar. |
| **Jumbo Frame** — nyckelord `*JumboPacket` | **9014** | Krävs för GVSP-paket på 9000 byte (6× färre paket/ram än 1500). |

> ⚠️ **En uppdatering av nätverkskortdrivrutinen ÅTERSTÄLLER dessa avancerade egenskaper till standardvärdena.**Efter uppdatering eller byte av nätverkskortdrivrutinen måste du**tillämpa** `ReceiveBufferLen=256` och `PendingReceives=64` på nytt, annars kommer arraypanelen att stängas av igen trots att ”ingenting har ändrats i hårdvaran”. Detta är den främsta orsaken till att en rigg som tidigare fungerade plötsligt vägrar att ansluta.

Tillämpa från en **förhöjd** PowerShell (ersätt med ditt nätverkskortnamn, t.ex. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` gäller USB 10GbE-adaptrar.** Nu detekteras adaptertotypen och rätt nyckelord för mottagningsringen ställs in: `*ReceiveBuffers`→2048 för PCIe-nätverkskort (Intel I219, m.fl.), eller `ReceiveBufferLen`→256 + `PendingReceives`→64 för Realtek **USB** 10GbE-kontroller (som inte exponerar `*ReceiveBuffers`). Målvärdena begränsas till det maximivärde som varje drivrutin rapporterar (`NumericParameterMaxValue`), så det skrivs aldrig in något värde utanför intervallet. Kör det från en **förmågad** terminal; precis som alla registerbaserade justeringar träder ändringen i kraft efter en omstart av nätverkskortet eller en omstart av datorn. De manuella `Set-NetAdapterAdvancedProperty`-kommandona ovan är fortfarande ett bra alternativ – de tillämpas direkt (omkopplar nätverkskortet) utan omstart.

### Nätverksgrunder (alla LATTICE-länkar)

- **Adressering:** länklokal `169.254.0.0/16` (GigE Vision LLA). Värddatorn tilldelas en statisk adress `169.254.x.x/16`; kameror och DAQ-E tilldelar sig själva adresser inom samma intervall. Inget DHCP eller gateway krävs.
- **Paketstorlek:**föredra jumbo (9000), men låt den automatiska-proben bestämma storleken – den mäter om vid varje anslutning och ser redan förbi kamerans ICMP-begränsning på 1500 byte via en GVSP-prob, så den landar på jumbo där kabeln faktiskt klarar det. Använd `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` endast när du vet bättre än sonden, och föredra per kommando framför permanent: en fast inställning hoppar över sonden, så om vägen faktiskt inte klarar 9000**varje** insamling tidsutgår med `SC_ERR_TIMEOUT -1011` (se [Miljövariabler](#environment-variables)).
- **RX-ringen skalar med `ReceiveBufferLen`:**vid standardvärdet `32` är den användbara ringen ~0,26 MB (för liten för någon multi-kameraserie); vid det maximala värdet `256` är den stor (~13,5 MB uppmätt på labbets 10GbE-värd), vilket ger verkligt utrymme. Huruvida en konfiguration ansluts avgörs sedan av den dräneringsmedvetna tillträdeskontrollen**och** den sammanlagda kontrollen av överteckning nedan – inte en ren jämförelse mellan burst och-ring-jämförelse.

### Array-fps och burst-modell

Hur man läser panelen Array-inställningar (och `lattice analyze-array` / SDK:s `analyze_array_network`):

- **Burst summeras per kamera i varje kameras faktiska pixelformat.**Mono**M3M**-kameror strömmar**Mono12 (2 B/px)**;**M3C**-Bayer-kameror strömmar 8- eller 12-bitars (TRI032S sänder tyst BayerRG12 även när BayerRG8 begärs). Så en bild i full upplösning från fyra kameror är**~12,6 MB om alla är 8-bitars, men ~25 MB med tre 12-bitars monokameror**. Projektionen avgör varje kameras format utifrån dess modell (identitetscache), så att burst-värdet stämmer överens med vad kabeln faktiskt överför — inte ett generellt antagande om BayerRG8.
- **En USB-Ethernet-adapter har en övre gräns på 200 MB/s oavsett vad som står på typskylten.** Effektivitetstabellen som omvandlar en länkhastighet till ett kontinuerligt värde är härledd från PCIe; ett USB-nätverkskort anger sin *Ethernet* länkhastighet men begränsas av USB-bussen och dess drivrutin. En USB 10GbE-dongel brukade uppnå ~1063 MB/s ”kontinuerligt” — ett värde som aldrig verifierades — och den resulterande takten förvrängde 6–18 % av ramarna samtidigt som den fortfarande rapporterade ett normalt mål-fps. USB-anslutna nätverkskort är nu begränsade till **200 MB/s** som ett absolut värde (begränsningen ligger i bussen, så den skalar inte med vad som står på typskylten; en USB 1 GbE-adapter uppnår ~80 MB/s och påverkas inte). `wire_ceiling_source` i kapacitetsregistret anger detta i ord, och `nic_is_usb` markerar det. Åsidosätt båda med `--wire-ceiling-mbps`.
- **Admittansen är drain-medveten, inte hel-burst-vs-ring.** En samtidig burst behöver endast passa in i den *transient backlog* = `max(0, Σ per-cam arrival − host drain) × emit_window`, inte hela bursten. I en struktur med snabb värd och långsamma kameror (en **PCIe**10G-värd + 4× 1 GbE-kameror: ankomst ≈ 320 MB/s, tömning ≈ 1063 MB/s) töms värden snabbare än kamerorna fylls, backlog ≈ 0, så**tillåter**sim-emit i full upplösning detta trots att 25 MB-bursten överskrider 13,5 MB-ringen. Anslut samma fyra kameror till en**USB**10GbE-adapter och avläsningen blir 200 MB/s, inte 1063 – ankomsten överstiger den, och förlusten visar sig som korrupta bildrutor snarare än som en lägre bildfrekvens. På en 1 GbE-värd gör kamerornas DLThr-gräns på 31,25 MB/s att ankomsten överstiger utmatningen → den**blockerar** (för *denna* typ av block, minska ROI eller använd binning ≥ 2). Tillträde sker via en av **två** anslutningsgrindar — den andra är den sammanlagda kontrollen av överteckning nedan.
- **Den beräknade bildfrekvensen (fps) är ett konservativt tak för seriell hämtning.**Värdens hämtningsslinga hämtar för närvarande varje kameras buffert**seriellt**(~ett utsändningsfönster per kamera vardera), så cykeln begränsas av `max(readout+emit, N × emit)` där utsändningen per kamera begränsas till kamerans**åtkomstlänk**(1 GbE ≈ 80 MB/s), inte värdens upplänk. För en uppsättning med 4 kameror i full upplösning och 12 bitar blir det**~2,8 fps**, vilket stämmer överens med de uppmätta värdena på ~2,7–3,0. fps är avsiktligt**exponeringsoberoende**, så i svagt upplysta scener kan det faktiska värdet sjunka något under takvärdet när exponeringstiden förlängs. Den seriella hämtningen är den verkliga fps-begränsaren; att parallellisera den skulle höja takvärdet mot hastigheten för enstaka utsändningar.
- **Aggregerad överteckning är en hård anslutningsblockerare.**Bandbreddstilldelningen per kamera har en lägsta gräns på**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), så när minimivärdet når sin gräns kan den sammanlagda efterfrågan (`per_cam × N`) överskrida det**kollisionssäkra nätverksgränsvärdet**(`sustained × sim_emit_factor`). Praktiska maximalaupplösning på 1 GbE:**6 kameror vid 1500 MTU, 9 med jumbo**. Detta tak beror enbart på kabeln och golvet – det är**oberoende av ramstorlek**, så**binning och mindre ROI hjälper INTE** (de minskar antalet byte per *ram*, inte antalet GevSCPD-styrda byte per *sekund*); de enda lösningarna är färre kameror, jumbo-ramar fråntill slut, eller ett snabbare nätverkskort. Symptomet skulle vara paketförlust i GVSP, inte en gradvis minskning av fps, så `analyze-array` nollställer siffrorna för uppnåelig fps och visar `**OVER-SUBSCRIBED**`, och `array-connect` med en fastställd upplösning **vägrar att ansluta** (walk-down-funktionen sorterar annars bort ramar, vilket inte heller löser denna typ av blockering). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` nedgraderar avvisningen till en tydlig varning för testarbete — se [Miljövariabler](#environment-variables).

### Arrayhälsa — vilket delsystem tappar bildrutor

En ansluten arrays `GET /api/camera/array/<array_id>/capability` bär ett aktivt
`health`-block, som omvärderas i ett rullande **10-sekunders** fönster. Det delar upp bildruteförlusten
i de två orsakerna som kräver motsatta åtgärder, istället för att rapportera en ”ofullständig”
förlustfrekvens som inte specificerar någon av dem:

| Fält | Vad det betyder | Vilket delsystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriell anslutning) | Ramen **anlände men var strukturellt felaktig**— GVSP-paketförlust. |**Nätverk**: kabelbudget, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (per seriell port) | Ramen **kom aldrig alls**— kameran utlöste inte, eller så skickades inget ut. |**Trigger / synk**: M8-kabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Den sämsta kamerans hastighet för respektive. | — |
| `per_cam_rate_pct` | Kombinerad andel ofullständiga bilder per kamera (båda orsakerna tillsammans). | — |
| `stable_for_seconds` | Hur länge varje kamera har legat under 0,01 %. | — |

Över 5 % loggar backend-systemet en `[array-health <id>] WARN`-rad som anger uppdelningen — vid
det första avsteget, vid en förändring av allvarlighetsgraden, en gång per minut så länge det kvarstår, och en gång när
det åtgärdas. Den felaktiga halvan skriver ut `[gvsp-corrupt <SN>]` vid den första träffen per kamera och
orsak, därefter en sammanställning var 60:e sekund. Varje utvärdering hamnar fortfarande i backend-loggfilen;
räknarna rör sig vid varje buffert oavsett vad som skrivs ut.

Samma post rapporterar det nummer som hela tilldelningen hänger på:

| Fält | Vad det betyder |
| --- | --- |
| `wire_ceiling_mbps` | Värdens gällande kontinuerliga bandbreddsbudget, MB/s. |
| `wire_ceiling_source` | Varifrån det numret kommer, i ord — t.ex. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` eller `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` när `--wire-ceiling-mbps` (eller GUI:ns **Wire Budget**-fält) ställer in det. |
| `nic_is_usb` | `true` för en USB-Ethernet-adapter — se gränsen på 200 MB/s ovan. |

**Tolkning:** ett värde på `gvsp_corrupt_rate_pct` som inte är noll och `never_arrived_rate_pct` på 0
betyder att triggning och kabelsynkronisering är perfekta och att 100 % av förlusten ligger på nätverksvägen
— sänk `--wire-ceiling-mbps` och anslut på nytt. Det omvända mönstret pekar istället på
synkroniseringskabeln eller utlösningsledningen.

> **`--target-fps` är inte reglaget för korrupta ramar.** GevSCPD-takten skrivs
> en gång vid anslutning, så att sänka utlösningsfrekvensen ändrar arbetscykeln och inte
> burstfrekvensen för samtidig sändning. En uppmätt 5×-begränsning gav ingen förbättring;
> att sänka trådens tak från 240 till 200 MB/s minskade andelen korrupta ramar för samma rigg från 10,4 %
> till 0,00 %.

> **Automatisk krympning mitt i strömmen är inte tillgänglig i TRI032S-firmware.** En pågående array
> kan inte åtgärda detta själv; koppla bort och anslut på nytt så att anslutningstidsväljaren kan
> planera om med det nya taket.

### Symptom → åtgärd

| Symptom (Arrayinställningar / anslutning / `analyze_array_network`) | Orsak | Åtgärd |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` återställs till 32 (vanligtvis efter en drivrutinsuppdatering) | Ställ in `ReceiveBufferLen`→256, `PendingReceives`→64; öppna panelen igen (starta om backend om den har cachat den gamla ringstorleken) |
| Omstart/avstängning hänger sig; senare `Invalid class` WMI-fel, nätverkskortet aktiveras inte, USB-enheter saknas | Gammal Realtek USB 10GbE-drivrutin från 2016 → BSOD `0x9F` → tvingad avstängning-avstängningar | Uppdatera adapterdrivrutinen till ≥ v10.67 (2026), och tillämpa sedan ovanstående inställningar för mottagningsringen på nytt |
| Anslutningen lyckas men returnerar en upplösning lägre än den ursprungliga | Smart-prep krymper automatiskt ramen för att passa kabeln | Uppgradera länken / acceptera krympningen / `--force-tier slip-emit-and-capture` |
| Arrayen rapporterar en korrekt mål-fps men levererar bara en bråkdel av den; `health.gvsp_corrupt_rate_pct` icke-noll, `never_arrived_rate_pct` 0 | Värdens beräknade kabelkapacitet överskattar vad den faktiskt klarar av (typiskt för en USB-Ethernet-adapter, en smal PCIe-bana eller en delad nätverksstruktur) | Anslut på nytt med ett lägre `--wire-ceiling-mbps` och kontrollera hälsoblocket på nytt. **Inte** `--target-fps` — GevSCPD-pacing är fastställd vid anslutning |
| Kameror saknas i publicerade grupper; `health.never_arrived_rate_pct` icke-noll, `gvsp_corrupt_rate_pct` 0 | Trigger-/synkroniseringsväg — kamerorna utlöses inte, det är inte ett nätverksproblem | Kontrollera M8-synkroniseringskabeln och `--line`; bekräfta att alla enheter är aktiverade (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` överskridet i `analyze-array`, eller anslutningsvägran med fastställd upplösning (`array over-subscribes the wire`) | Den sammanlagda efterfrågan per kamera (minst 8 MB/s × N kameror) överskrider det kollisionssäkra taketsäkerhetsgränsen — 6 kameror i full upplösning på 1 GbE @1500 MTU, 9 med jumbo | Färre kameror, jumbo-ramar från ändpunkt till ändpunkt eller ett snabbare nätverkskort. **ROI/binning hjälper INTE** (taket är oberoende av ramstorlek). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` åsidosätter på testbänken (accepterar paketförlusten) |

---

## `chloros-cli daq`

Kommandon för spektralsensorer. Två klasser:
- **`pool-*`**— smala HTTP-klienter som styr sensorn via backendens permanenta pool.**Detta är den stödda vägen, och den enda som finns i den levererade CLI.** Backenden äger transporten, så GUI:t, CLI- och SDK-skripten delar alla ett aktivt handtag istället för att tävla om den seriella porten.
- **Allt annat**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — direkt hårdvaruåtkomst, dokumenterad nedan för fullständighetens skull. Dessa kräver paketet `daq` Python, vilket**inte ingår i någon levererad artefakt**: den kompilerade versionen av CLI utesluter det (`scripts/Build-CLI.ps1` sätter `--nofollow-import-to=daq`, och transportfilerna `pyserial` / `bleak` / `zeroconf` med den), och PyPI-paketet SDK innehåller den inte heller. De körs endast från en källkodsutcheckning, så betrakta dem som en MAPIR-intern utvecklingsväg snarare än något att sträva efter.
- **`discover` / `list`** ligger någonstans mittemellan: de är direkta hårdvarukommandon från en källkodsutcheckning, men i en färdig levererad version faller de tillbaka till `pool-discover` och backenden utför skanningen. Så skanningen fungerar överallt – vilket är viktigt eftersom det är det enda sättet att ta reda på en DAQ-M:s BLE-MAC.

> **`chloros-cli daq --help`** (samt `-h` / `help`) listar `pool-*`-underkommandona — hjälpinformationen dirigeras medvetet till poolklienten så att den återspeglar de kommandon som faktiskt körs. Om du anropar ett direkt-hårdvaru-underkommando på en levererad version avslutas det med ett uttryckligt felmeddelande som anger vilket paket som saknas och hänvisar dig tillbaka till `pool-*`; ingenting misslyckas tyst. (`discover` / `list` är undantaget — de omdirigeras till `pool-discover` och fungerar helt enkelt.)
>
> **Allt en kund behöver är tillgängligt via `pool-*`** – anslut, strömma, spela in kalibrerade `.daq`-filer och byt kap-profiler. DAQ-enheten kan även styras från Python med `chloros_sdk.connect_daq_sensor()`, som använder samma sammanslagna sökväg.

### Arbetsflöde för första anslutning av DAQ-sensor

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*`-referens

| Underkommando | Syfte |
| --- | --- |
| `daq pool-connect` (smart-detect) | Öppna en sensor i backend-poolen. |
| `daq pool-connect --port PORT` | DAQ-U på en specifik seriell port. |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC-adress skannas automatiskt. |
| `daq pool-connect --mac MAC` | DAQ-M via BLE på en känd MAC-adress (förutsätter `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E via Ethernet på en känd värd. |
| `daq pool-connect --eth` | DAQ-E via Ethernet, värd upptäckt automatiskt (mDNS + ARP som reserv; fungerar från en tom ARP-cache på Windows och Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Justera integrationsfönster / AE-status. |
| `daq pool-connect --no-stream` | Anslut men starta inte strömningen ännu (återuppta med `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil för Cap-korrigering. Standardinställningen i backend är `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Sök igenom alla transportenheter efter sensorer som du kan ansluta till, utan att ansluta. **Så här hur du hittar en DAQ-M:s BLE-MAC.** `daq discover` / `daq list` dirigeras automatiskt hit i de levererade versionerna. Sensorer som redan är öppna i poolen listas inte — en ansluten DAQ-M slutar sända — så använd `pool-list` för dem. |
| `daq pool-list` | Visa alla sensorer i backend-poolen. |
| `daq pool-disconnect --sensor-id ID [--all]` | Släpp. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De senaste N spektrumramarna. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Återuppta/pausa strömning. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Starta/stoppa en .DAQ-inspelning. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Byt kapacitanskorrigeringsprofil under körning. |

### Direkt-hårdvaru-underkommandon (endast källkodsutcheckning — ingår inte i levererade versioner)

> Anges för fullständighetens skull. Dessa kräver paketet `daq` Python samt `pyserial` / `bleak` / `zeroconf`, varav inget ingår i den kompilerade versionen CLI eller PyPI-versionen SDK — de körs endast från en MAPIR-källkodsutdragning. **Om du använder en släppt Chloros-version, använd istället kommandona `pool-*` ovan**; de täcker anslutning, strömning, inspelning och val av kap.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Öppna, anslut till och kör ett sparat Chloros-projekt (en mapp med `cameras.json` + `sensors.json` + `project.json`). Allt går via backend så att GUI och CLI ger identiska hårdvarustatus.

### Referens för underkommandon

| Underkommando | Syfte |
| --- | --- |
| `project open PATH` | Skriv ut projektets enhetsmanifest (kameror, matriser, sensorer). |
| `project devices PATH [--reconnect]` | Lista eller-kör upptäckt. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Anslut alla sparade kameror / matriser / sensorer. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Enstaka bildtagning från en namngiven kamera eller matris. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | N-bilders serie från en namngiven kamera eller bildserie (`-n/--count` standard 5; `-i/--interval` sekunder mellan bilderna, standard 0). Kameragruppserier avduplicerar upprepade synkroniserade grupper (föråldringskontroll) så att en delvis cyklisk grupp inte kan returnera N kopior av en bildruta; skriver ut resultat per iteration. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Strömning till disk via ett backend-jobb. `--poll-interval` = sekunder mellan `/stats`-avfrågningar (standardvärde 2,0). |
| `project sensor read PATH NAME [--json]` | Senaste spektrumramen. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Spela in .daq. |
| `project run PATH RECIPE.yaml` | Kör ett YAML/JSON-insamlingsrecept. `--dry-run` validerar utan att köra. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Beräkna inriktning för en matris — se [flaggtabellen nedan](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Skriv ut aktuell inriktningsprofil. |
| `project align clear PATH NAME` | Rensa den cachade profilen. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Justera en slavs transformation. |
| `project align export PATH NAME --to FILE` | Spara profilen till JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Ladda en sparad profil. |

#### `project align calibrate` Alternativ

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Justeringsmetod. **Dessa stavningar skiljer sig från `lattice align-calibrate`**, som använder kortformerna `orb` / `akaze` / `phase`; de två kommandona är inte utbytbara för denna flagga. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformera modellen så att den passar. |
| `--frames N` | `1` | Synkroniserade bildrutor till medelvärdet. |
| `--reference SN` | huvudkameran | Referenskamerans serienummer; alla övriga medlemmar förvrängs mot den. |
| `--max-features N` | `5000` | Övre gräns för antal ORB-egenskaper. |
| `--ratio-threshold F` | `0.75` | Lowes förhållandestest. |
| `--ransac-threshold-px F` | `3.0` | RANSAC tröskelvärde för inliers. |
| `--min-matches N` | `15` | **Kvalitetsgräns** — avvisa lösningen om antalet matchande inliers understiger detta värde. |
| `--max-reproj-err-px F` | `4.0` | **Kvalitetsgräns** — avvisa lösningen om RMS-felet vid omprojektion överstiger detta värde. |
| `--checkerboard RxC` | — | Brädgeometri för `--method checkerboard`, t.g. `9x6`. |
| `--name PROFILE` | tom | Profilnamn inbäddat i den sparade JSON. **Inte matrisnamnet** — det är positions-`NAME`. |

De två kvalitetskontrollerna är anledningen till att en kalibrering kan lyckas med lösningen men ändå
avvisa sparandet: en profil som misslyckas med någon av dem skulle tyst felregistrera varje
senare inspelning, så den avvisas istället för att sparas.

### Exempel

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Recept-DSL

`project run RECIPE.yaml` accepterar en YAML- eller JSON-fil som beskriver en sekvens av åtgärder:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Åtgärder som stöds: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Åtgärden `burst` kräver `name` (obligatoriskt), `count` (standardvärde 5), `interval` (sekunder, standardvärde 0), `output`, `format`, och `settings` (samma inställningsform per kamera som `apply`); array-serier använder samma nysynkroniserade grupp-watchdog som `project burst`.

Kör följande:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Miljövariabler

| Variabel | Effekt |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Åsidosätter backend-inställningen URL (standard: `http://127.0.0.1:5000`) — **respekteras endast av kommandofamiljerna `lattice`, `project` och `daq pool-*`.** Kärnkommandona (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) kopplar till `http://127.0.0.1:<port>` och ignorerar denna variabel (IPv4-värdet kringgår straffet på ~2 sstraff per begäran), så de riktar sig alltid mot den lokala maskinen. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` nedgraderar avvisningen av anslutningar vid överteckning av arrayen (sammanlagd efterfrågan per kamera &gt; kollisionssäkert trådtak med `pin_resolution`) till en tydlig varning med fortsättning, vilket accepterar GVSP-paketförlust. Endast för användning i testmiljö — se [Array fps &amp; burst-modell](#array-fps--burst-modell). |
| `CHLOROS_CLI_MODE` | Ställs in av själva CLI; instruerar backend att aktivera parallellbearbetning. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` hoppar över GVSP-fallback-sonden (endast ICMP-resultat). **Detta inaktiverar jumbo, det dämpar inte bara loggen** — kameran svarar endast på DF-pingar upp till 1500 på varje väg, så denna sond är det enda som kan upptäcka jumbo. Sparar ~1 s per kamera per anslutning; kostar ~1,45× kabeltaket om nätverket *skulle* ha kunnat hantera jumbo. SDK varnar när du ställer in den. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fastställer GVSP-paketstorleken till N byte; hoppar över sonderingen helt. Använd helst per kommando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) istället för att ställa in det permanent: en fastställd storlek slutar anpassa sig till nätverket framför den, och att fastställa 9000 på en väg som inte kan hantera jumbo gör att **varje** insamling att gå över tiden med `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Åsidosätt Nuitkas extraheringskatalog för enstaka filer. CLI använder automatiskt `/mnt/ssd/tmp` om den finns. |

---

## Avslutningskoder

| Kod | Betydelse |
| --- | --- |
| `0` | Lyckades. |
| `1` | Allmänt fel (de flesta underkommandofel). |
| `2` | Argumentfel. |
| `130` | Avbruten med Ctrl+C. |

---

## Tips för felsökning

- **&quot;Inloggning krävs&quot;** → Kör `chloros-cli login EMAIL PASSWORD` en gång på den här datorn.
- **&quot;backend unreachable&quot;** → Starta skrivbordsappen Chloros, eller kör backend-binären direkt (`chloros-backend`), eller kontrollera `CHLOROS_BACKEND_URL` vid fjärranslutning.
- **`lattice`-kommandon misslyckas med meddelandet ”LATTICE-kameradrivrutiner hittades inte”** → Arena SDK-runtime är inte installerat; CLI levereras med `win32api` inkluderat i Windows, men C-körmiljön ingår i GUI-installationsprogrammet.
- **Array connect / Array Settings visar &quot;FRAMES WILL DROP&quot; eller &quot;Reduce ROI to enable&quot;** → Värddatorns nätverkskortets mottagningsring är för liten (återställs vanligtvis till 32 efter en uppdatering av nätverkskortdrivrutinen). Se [Konfiguration och inställning av värddatorns nätverkskort](#host-nic-setup--tuning-lattice-arrays) — ställ in `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Datorn hänger sig vid omstart/avstängning, sedan WMI `Invalid class` / nätverkskortet aktiveras inte / USB-enheter saknas** → Föråldrad drivrutin för USB 10GbE-adapter orsakar `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Uppdatera drivrutinen för adaptern — se [Konfiguration och finjustering av värd-NIC](#host-nic-setup--tuning-lattice-arrays).
- **Jetson-swap-varning** → Lägg till filbaserad swap; CLI visar exakt samma kommandon som `fallocate` / `swapon`.
- **DAQ-direktkommandon saknas** → Förväntat: den medföljande `chloros-cli` utesluter medvetet paketet `daq`, så endast `pool-*` finns (PyPI SDK innehåller det inte heller). Använd `pool-*`, som styr samma sensor via backend, eller `chloros_sdk.connect_daq_sensor()` från Python.

---

## Se även

- [Python SDK-referens](sdk-reference.md) — programmatisk motsvarighet till varje CLI-kommando.
- [DAQ-sensorguide](../daq/README.md) — sensorspecifik kabeldragning + kalibrering.
- Online-dokumentation: `https://mapir.gitbook.io/chloros/cli`
