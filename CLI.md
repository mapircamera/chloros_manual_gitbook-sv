# CLI : Kommandorad

> **Fullständig referens:**[CLI Reference](reference/cli-reference.md) dokumenterar**varje flagga för varje underkommando** och är optimerad för AI-assistenter — klistra in dess URL i din assistent och be om ett fungerande kommando: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tips för AI-verktyg:** vilken sida som helst i denna manual finns tillgänglig som ren Markdown genom att lägga till `.md` till dess URL (t.ex. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), och `https://mapir.gitbook.io/chloros/llms.txt` indexerar hela handboken för användning i stora språkmodeller (LLM).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## VadCLI
är

`chloros-cli` är ett kommandoradsgränssnitt till samma bearbetningsmotor som desktop-appenChloros
använder. Det är en tunnHTTP
-klient som kommunicerar med backend-servernChloros
(en lokal server på `127.0.0.1:5000`) — de flesta kommandon startar backend-servern automatiskt, så ett enda `chloros-cli process …`-anrop är allt som behövs i ett skript.

Den körs på **Windows
10/11 (x64)**och**Linux
(x86_64 samt NVIDIA Jetson arm64 på JetPack 6)**, i valfri terminal, utan att något grafiskt gränssnitt krävs. Kontrollera din installation med:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Kommandofamiljerna i korthet:

* **Bearbetning och konto** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 språk — se [Stödda språk](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (endastLinux
/Jetson)
* **Live-hårdvara** — `lattice` (LATTICE-kamerastyrning, över 45 underkommandon), `daq pool-*` (DAQ-ljussensorer), `time-sync` (PTP)
* **Automatisering** — `project` (köra ett sparatChloros
-projekt utan gränssnitt, inklusive YAML-insamlingsrecept)

Globala alternativ som är värda att känna till: `--port N` (backend-port, standard `5000`), `-v/--verbose`, `--restart` (tvinga omstart av backend), `--backend-exe PATH`. Se [CLI
Referens](reference/cli-reference.md) för en fullständig lista.

***

## Installation

CLI
**ingår iChloros
-installationsprogrammet** på alla plattformar — det finns ingen separatCLI
-nedladdning. Hämta installationsprogrammet från [Nedladdning](download.md)-sidan.

###Windows


Installationsprogrammet placerarCLI
i:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

och lägger till den mappen i ditt system `PATH` — **öppna en ny terminal**efter installationen så att den uppdaterade `PATH` hämtas. Installationsprogrammet placerar även startskript (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) i installationskatalogen samt en**Chloros
CLI
** genväg i Start-menyn, som var och en öppnar en terminal med `chloros-cli` redo att användas.

###Linux


Installera `.deb` för din arkitektur:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Detta installerar `chloros-cli` till `/usr/bin/chloros-cli` (redan på `PATH`) och backend till `/usr/lib/chloros/chloros-backend`, tillsammans med Arena-SDK
-runtime som krävs för LATTICE-kameror. Se [Linux
Installation](linux/linux-installation.md) för mer information.

### Verifiera

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Inloggning och licensiering

CLI
(ochPython
SDK
) kräver ett **betaltChloros
+-abonnemang**— alla betalda nivåer har det; den kostnadsfria nivån har det inte. Begränsningen tillämpas**på serversidan** av backend, inte avCLI
-programmet: ett samtal från en utloggad användare avvisas med `401 AUTH_REQUIRED`, och ett inloggat anrop på gratispaketet med `403 PLAN_UPGRADE_REQUIRED`, oavsett om det kommer från `chloros-cli`,SDK
eller en egenutveckladHTTP
-klient. Uppgradera på [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Logga in **en gång per maskin**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Lösenord med specialtecken**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` förvrängs av skalet (CLI
en upptäcker detta vid en 401-felkod och gör automatiska omförsök, men enkla citattecken undviker problemet helt).
{% endhint %}

Sessionen cachas i `~/.chloros/user_session.json` och fortsätter att fungera offline under abonnemangets respitperiod (30 dagar för månadsabonnemang, fram till utgångsdatumet för årsabonnemang). `chloros-cli status` fungerar även utan ett betalt abonnemang, så orsaken till ett avslag är alltid synlig.

{% hint style="danger" %}
**Ska du schemalägga headless-arbete? Logga in först.**Ett kommando som startar en backend (`process`, `status`, `export-status`, …) som körs**utan cachelagrad session**misslyckas inte omedelbart – det övergår till en interaktiv `Email:` / `Password:`-prompt på stdin. Ett obevakat cron-jobb eller CI-steg kommer därför att**hänga sig i väntan på inmatning**. Kör `chloros-cli login EMAIL 'PASSWORD'` en gång på maskinen innan du schemalägger något.
{% endhint %}

***

## Din första bearbetningskörning

Peka `process` mot en mapp med inspelningar — det upptäcker automatisktSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` eller en blandning:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Förloppsströmmar visas live per pipeline-tråd (Detektering, Analys, Bearbetning, Export), och en lyckad körning avslutas med en rapport om hur många bildprodukter som har skrivits (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Var utdata hamnar

`process` skriver till en **projektmapp**, inte till din inmatningsmapp:

* Utan `-o`: projektet skapas under din standardprojektmapp (delad med GUI; hantera den med `get-project-folder` / `set-project-folder`, alternativt `~/Chloros Projects`), och namnges med `-n/--project-name` eller en tidsstämpel (`YYYYMMDD_HHMMSS`) om det utelämnas.
* Med `-o PATH`: den mappen **är** projektmappen. Om den redan innehåller en `project.json` skapas istället en systermapp med suffixet `_1`/`_2`… istället för att den gamla skrivs över.

Inom projektet grupperas produkterna **efter kamera, därefter efter filformat**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kameramappen är `LATT-<sensor>-<lens>-F<filter>` för LATTICE (motsvarar bildens EXIF-data `Model`) och `<model>_<filter>` (t.ex. `Survey3N_RGN`) förSurvey3
. Formatmappen följer `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` eller `tiff32` för `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Varje exporterad produkt behåller källfilens namn.**En Radiance-export av `capture_..._raw.tif` heter fortfarande `capture_..._raw.tif` — den finns bara i mappen `tiff32/Radiance_Images/`.**Mappen identifierar produkten, inte filnamnet**, så använd glob för katalogen, inte för suffixet `*radiance*`.
{% endhint %}

### De alternativ du faktiskt kommer att använda

| Flagga | Standard | Vad den gör |
| --- | --- | --- |
| `-o, --output PATH` | standardprojektmapp | Projektmappens plats (se ovan). |
| `-n, --project-name NAME` | tidsstämpel | Projektnamn. |
| `--format FMT` | `TIFF (16-bit)` | Ett av `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | inga | Vegetationsindex att exportera (se [Vegetationsindex](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neural debayer, långsammare, högsta kvalitet (Chloros
+, NVIDIA GPU). |
| `--vignette / --no-vignette` | på | Vignettkorrigering. |
| `--reflectance / --no-reflectance` | på | Reflektanskalibrering; för LATTICE är detta även omkopplaren för reflektansprodukten. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Tvinga startpunkten för bearbetningskedjan för LATTICE-TIFF-filer. |

För allt annat — inställning av måldetektering, PPK, exponeringsmarkörer, flaggor för matrisjustering — se avsnittet [`process` i referenshandboken förCLI
](reference/cli-reference.md).

***

## Välja vad som ska exporteras (LATTICE-produkter)

LATTICE-bearbetningen fördelas till **alla tillämpliga produkter i ett enda steg**. Fyra produktspecifika omkopplare är alla**aktiverade som standard**; använd formuläret `--no-` för att inaktivera en:

| Väljare | Produkt |
| --- | --- |
| `--debayered` | Linjär demosaik → `Debayered_Images/` |
| `--preview` | Förhandsgranska (vitbalans + gamma; falskfärgssträckning för multispektral) → `Preview_Images/` |
| `--radiance` | float32 strålningsintensitet, W/m²/sr/nm → `Radiance_Images/` (alltid `tiff32/`) |
| `--reflectance` | uint16 reflektans, Pix4D-kompatibel → `Reflectance_Calibrated_Images/` |

RGB
huvudkameror avger alltid endast debayered + förhandsgranskning — strålning/reflektans per band är inte meningsfullt för en bredbandssensor, så dessa växlingsalternativ har ingen effekt för dem.Survey3
`.raw` ignorerar växlingsalternativen och följer standardvägen för reflektans/mål.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (standard `auto`) väljer reflektansreferensen: `auto` skapar ett QA-godkänt [kalibreringsmål](calibration-targets.md) i bildrutan som absolut referens och faller tillbaka till DAQ-ljussensorns nedåtriktade delningsvärde (ρ = π·L/E) när inget mål är närvarande; `target` är strikt (ingen DAQ-ersättning); `daq` är DAQ-auktoritativ. Mätningar av mål per enhet kan tillhandahållas med `--target-reflectance-dir`.

{% hint style="info" %}
**Avläsning av reflektanspixlar:**DN-värdet som betyder ρ = 1,0 är**per källa** — LATTICE-filer stämplar `Chloros:PixelScale=32768` i XMP;Survey3
-filer använder 65535 (och innehåller inga `Chloros:*`-taggar). Läs av taggen och dividera med den istället för att anta en konstant. Detaljer och det enda avsiktliga specialfallet utan skala finns i [CLI
-referensen](reference/cli-reference.md).
{% endhint %}

**Bearbetningen börjar alltid från `raw`.** Härledda produkter (export av debayering/strålning/reflektans) matas aldrig tillbaka genom pipelinen — att återimportera dem och bearbeta dem skulle innebära att kalibreringsberäkningarna tillämpas två gånger, såChloros
hoppar över dem och meddelar detta. `--input-level` är den avsiktliga nödutgången när du verkligen behöver tvinga fram en startpunkt.

***

## När en körning misslyckas

Från och med version 1.2.0 misslyckas `process` med tydlig feedback istället för att ”lyckas” utan att visa något resultat:

* En körning som **begärde produkter men inte skrev ut några**— endast `project.json` och `calibration_data.json` — skriver ut `Processing finished but wrote no image products.` och**avslutas med ett värde som inte är noll**, så att skript kan upptäcka det. De vanligaste orsakerna: inmatningsmappen kändes inte igen som en inspelning (kontrollera layouten och `--input-level`), eller så var alla begärda produkter otillämpliga för dessa kameror (t.ex. att begära radians/reflektans från kameror som endast harRGB
).
* En **avsiktlig körning endast med metadata** (alla produkter avaktiverade, inget `--indices`) räknas fortfarande som lyckad — en tom bildutmatning är det korrekta resultatet i det fallet.
* Kör om med `--verbose` och kontrollera backend-loggen efter rader med `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, som förklarar vilka kameror som hoppats över.

Utgångskoder: `0` framgång · `1` generellt fel · `2` argumentfel · `130` avbrutet med Ctrl+C.

***

## Vegetationsindex

Kör `--indices` med ett eller flera förinställda namn; varje index hamnar i sin egen `<INDEX>_Index_Images/`-mapp:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

De 22 förinställda namnen som `process --indices` accepterar:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Det finns tre indexlistor – blanda inte ihop dem.**I rullgardinsmenyn ”Projektinställningar” i GUI:n finns 27 formler (lägger till `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` – dessa fem finns endast i GUI och gäller**inte** för `--indices`). Kommandot live/offline `lattice index --preset` använder en egen separat lista med 22 förinställningar. Formler och bandberäkningar finns dokumenterade i [Formler för multispektrala index](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ-ljussensorer: En snabb översikt

`daq pool-*`-familjen styrMAPIR
DAQ-spektralsensorer (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) via backendens permanenta pool – GUI:n,CLI
ochSDK
delar alla ett live-handtag. **`pool-*` är den DAQ-sökväg som stöds i den medföljandeCLI
**; andra `daq`-underkommandon som du kanske ser refereras till är enMAPIR
-intern yta endast för källkod och avslutas med ett explicit fel som hänvisar dig till `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` utan `--duration` körs fram till `pool-record --stop`; standardutmatningskatalogen är `~/Documents/DAQ Live View/` **på backend-maskinen**. Profilen för kap-korrigering väljs vid anslutningstillfället (`--cap-id`, standard för backend `sunshine_cosine`) och kan bytas ut i realtid med `pool-set-cap` — profiler för kapacitetskorrigering och sensorns kalibrerade mätområde beskrivs i DAQ-kapitlen i denna handbok.

{% hint style="warning" %}
**DAQ-E på en värddator med flera nätverkskort:** den första automatiska upptäckten av `pool-connect --eth` efter uppstart kan misslyckas även med en felfri sensor. `--eth-host <ip-or-hostname>` är den tillförlitliga varianten — använd den när upptäckten inte ger något resultat.
{% endhint %}

***

## LATTICE-kameror, PTP och projektautomatisering

`lattice`-familjen (över 45 underkommandon) täcker LATTICE-kamerans funktioner från början till slut: upptäckt, enstaka bildtagningar, beständiga synkroniserade matriser med GUI:ns smart-prep-anslutningsflöde, liveförhandsgranskning i webbläsaren, justering, indexberäkningar och värd-NIC-diagnostik. Ett smakprov:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Vid sidan av detta: `chloros-cli time-sync` rapporterar om den PTP-grandmaster som körs på värddatornChloros
(LATTICE-kameror och DAQ-E-sensorer fungerar som slavar till denna för enhetsöverskridande tidsstämplar), och `chloros-cli project` öppnar ett sparatChloros
-projekt och styr dess kameror, arrayer och sensorer utan grafiskt gränssnitt – inklusive skriptbaserade YAML-inspelningsrecept.

Dessa tre familjer (`lattice`, `project`, `daq pool-*`) är också de enda som stöder `CHLOROS_BACKEND_URL` för styrning av en **fjärransluten** backend; kärnkommandona riktar sig alltid mot den lokala maskinen.

Fullständiga genomgångar finns i LATTICE-kapitlen i denna handbok; alla flaggor finns i [CLI
Referens](reference/cli-reference.md).

***

## Felsökning: Topp 5

| Symptom | Lösning |
| --- | --- |
| `Login required`, eller ett schemalagt jobb hänger sig vid en `Email:`-prompt | Kör `chloros-cli login EMAIL 'PASSWORD'` en gång på den här maskinen – kommandon utan en cachad session körs interaktivt istället för att avbrytas omedelbart. |
| `backend unreachable` | Starta skrivbordsappenChloros
eller kör backend-binären direkt (`chloros-backend`). Om du pekar `lattice`/`project`/`daq pool-*` mot en fjärrbackend, kontrollera `CHLOROS_BACKEND_URL`. |
| Array-anslutning blockerad: `FRAMES WILL DROP` / `Reduce ROI to enable` | Värd-NIC:s mottagningsring återställd till standardinställningar — den vanligaste orsaken till att en rigg som tidigare fungerade vägrar att ansluta, vanligtvis efter en uppdatering av NIC-drivrutinen. Kör `chloros-cli lattice network --fix` från en **terminal med utökade behörigheter** (eller ställ in `ReceiveBufferLen=256`, `PendingReceives=64`); se avsnittet *Host NIC Setup &amp; Tuning* i referensmaterialet. |
| `daq`-underkommandot avslutas: ”kräver hela DAQ-paketet…” | Förväntat på levererade versioner — den kompileradeCLI
levereras endast med `daq pool-*`-familjen, som täcker anslutning, strömning, inspelning och val av kap. Använd `pool-*` (eller `chloros_sdk.connect_daq_sensor()` frånPython
). |
| Jetson visar en varning om swap innan stora mappar | Lägg till filbaserad swap —CLI
visar exakt vilka `fallocate`/`swapon`-kommandon som ska köras. |

***

## Få hjälp

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Alla flaggor, alla underkommandon:** [CLI
Referens](reference/cli-reference.md)
* **Motsvarighet iPython
:** [Python
SDK
](api-python-sdk.md) och [SDK
Referens](reference/sdk-reference.md)
* **Support:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
