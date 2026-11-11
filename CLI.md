# CLI: Kommandoraden

> **Fullständig referens:**[CLI-referens](reference/cli-reference.md) dokumenterar**varje flagga för varje underkommando** och är optimerad för AI-assistenter — klistra in dess URL i din assistent och be om ett fungerande kommando: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tips för AI-verktyg:** alla sidor i denna handbok är tillgängliga som ren Markdown genom att lägga till `.md` till URL:en (t.ex. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), och `https://mapir.gitbook.io/chloros/llms.txt` indexerar hela handboken för användning med stora språkmodeller (LLM).

> **Vill du att din assistent ska köra Chloros själv?** `chloros-mcp`, en MCP-server installerad med Python-SDK:et, fungerar med Claude Desktop, Claude Code, Cursor och andra MCP-klienter: se [Använda Chloros med AI-assistenter](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Vad CLI
:n är

`chloros-cli` är kommandoradsgränssnittet till samma bearbetningsmotor som Chloros
-skrivbordsappen använder. Det är en tunn HTTP
-klient över Chloros
-backenden (en lokal server på `127.0.0.1:5000`) — de flesta kommandon startar backenden automatiskt, så ett enda `chloros-cli process …`-anrop är allt ett skript behöver. Backenden fortsätter sedan att köras för nästa kommando: den avslutas vid `chloros-cli logout` (såvida inte appen är öppen eller en körning eller en kompilering pågår), vid `chloros-cli backend stop`, eller automatiskt efter 30 minuter utan klient, körning eller ansluten enhet. `chloros-cli backend status` visar detta.

Det körs på **Windows
10/11 (x64)**och**Linux
(x86_64 samt NVIDIA Jetson arm64 på JetPack 6)**, i valfri terminal, utan krav på grafiskt gränssnitt. Kontrollera din installation med:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Kommandofamiljerna i korthet:

* **Bearbetning och konto** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 språk — se [Stödda språk](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (endast Linux
/Jetson)
* **Live-hårdvara** — `lattice` (LATTICE-kamerastyrning, över 45 underkommandon), `daq pool-*` (DAQ-ljussensorer), `time-sync` (PTP)
* **Automatisering** — `project` (kör ett sparat Chloros
-projekt utan grafiskt gränssnitt, inklusive YAML-insamlingsrecept)

Globala alternativ som är värda att känna till: `--port N` (backend-port, standard `5000`), `-v/--verbose`, `--restart` (tvinga omstart av backend), `--backend-exe PATH`. Se [CLI
-referensen](reference/cli-reference.md) för en fullständig lista.

***

## Installation

CLI
:n **ingår i Chloros
-installationsprogrammet** på alla plattformar — det finns ingen separat nedladdning av CLI
:n. Hämta installationsprogrammet från sidan [Nedladdning](download.md).

### Windows


Installationsprogrammet placerar CLI
:et i:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

och lägger till den mappen i ditt system `PATH` en gång (en uppgradering tar också bort dubbla poster som lämnats kvar av tidigare installationsprogram) — **öppna en ny terminal**efter installationen så att den uppdaterade `PATH` hämtas. Installationsprogrammet placerar även startskript (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) i installationsroten samt en**Chloros
CLI
** genväg i Start-menyn, som båda öppnar en terminal med `chloros-cli` redo att användas.

### Linux


Installera `.deb` för din arkitektur med `apt` (inte `dpkg -i`, som hoppar över de rekommenderade kraven för Python
SDK
):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Detta installerar `chloros-cli` till `/usr/bin/chloros-cli` (finns redan i `PATH`), backend- och CLI
-programmen som packats upp i `/usr/lib/chloros/backend/` och `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` är ett startprogram som startar backend-programmet), Arena SDK
-runtime som backend-programmet behöver för LATTICE-kameror samt Python
-SDK
för systemet `python3`. Se [Linux
-installation](linux/linux-installation.md) för mer information.

### Verifiera

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Inloggning och licensiering

Åtkomst till CLI
(och Python
-SDK
) kräver ett **betalt Chloros
+-abonnemang**– alla betalda nivåer har det; den kostnadsfria nivån har det inte. Begränsningen tillämpas**på serversidan** av backend-systemet, inte av CLI
-binären: ett utloggat anrop avvisas med `401 AUTH_REQUIRED`, och ett inloggat anrop på gratispaketet med `403 PLAN_UPGRADE_REQUIRED`, oavsett om det kommer från `chloros-cli`, SDK
eller en egenutvecklad HTTP
-klient. Uppgradera på [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Logga in **en gång per maskin**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Lösenord med specialtecken**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` förvrängs av skalet (CLI
:n upptäcker detta vid ett 401-fel och gör automatiska omförsök, men enkla citattecken undviker problemet helt).
{% endhint %}

Sessionen cachelagras i `~/.chloros/user_session.json` och fortsätter att fungera offline under planens respitperiod (30 dagar för månadsabonnemang, fram till utgångsdatumet för årsabonnemang). `chloros-cli status` fungerar även utan ett betalt abonnemang, så orsaken till ett avslag är alltid synlig.

{% hint style="danger" %}
**Ska du schemalägga headless-arbete? Logga in först.**Ett kommando som startar en backend (`process`, `status`, `export-status`, …) som körs**utan cachad session**misslyckas inte omedelbart – det övergår till en interaktiv `Email:` / `Password:`-prompt på stdin. Ett obevakat cron-jobb eller CI-steg kommer därför att**hänga sig i väntan på inmatning**. Kör `chloros-cli login EMAIL 'PASSWORD'` en gång på maskinen innan du schemalägger något.
{% endhint %}

***

## Din första bearbetningskörning

Rikta `process` mot en mapp med insamlade data — det upptäcker automatiskt Survey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` eller en blandning:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Framstegsströmmar i realtid per pipeline-tråd (detektering, analys, bearbetning, export), och en lyckad körning avslutas med en rapport om hur många bildprodukter som skrevs: `Image products written: N`, följt av `(plus M clamp masks)` när körningen även skrev [klämmasker](output-image-formats.md#clamp-masks) (dessa räknas inte som produkter).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Om en Chloros
-backend redan körs (till exempel desktop-appens), använder `process` den och öppnar sitt projekt i den. Den avvisar medan appen är öppen på ett projekt som appen har öppnat: kör från appen, eller stäng det projektet. Om en tidigare körning fortfarande pågår väntar den upp till 15 sekunder på den innan den avvisar. `--restart` tvingar fram en ny backend. I Linux
startas aldrig en backend som körs som en systemd-tjänst om på nytt: `process` använder den som den är, och där den annars skulle ha startat om den (`--restart`, eller en import som inte hittade något), skriver den ut `sudo systemctl restart <unit>` och avslutas `1`.

### Var utdata hamnar

`process` skriver till en **projektmapp**, inte till din inmatningsmapp:

* Utan `-o`: projektet skapas under din standardprojektmapp (delad med GUI; hantera den med `get-project-folder` / `set-project-folder`; om ingen är angiven, `Documents/MAPIR/Chloros Projects` i din användarmapp), namngiven med `-n/--project-name` eller en tidsstämpel (`YYYYMMDD_HHMMSS`) om inget anges.
* Med `-o PATH`: den mappen **är** projektmappen. Om den redan innehåller en `project.json` skapas istället en `_1`/`_2`… med suffix skapas istället för att skriva över. En relativ `PATH` anges relativt den mapp du kör `chloros-cli` från.

{% hint style="info" %}
**Relativa sökvägar (ändrat i 1.3.0).** Varje sökväg som backend öppnar — `-o`/`--output` vid vilket kommando som helst (inklusive standardkommandot `output` för inspelningskommandon), `--input`, `--burst-dir`, `--profile`, `--save` och `--mask` — tolkas utifrån den mapp som du kör `chloros-cli` från, så att `-o captures/` skriver till `<current folder>/captures/`. I version 1.2.0 löste backend-modulen dem i sin egen installationsmapp, vilket misslyckades med felmeddelandet ”Åtkomst nekad”. Mot en fjärrbackend (`CHLOROS_BACKEND_URL`) skickas sökvägarna precis som de skrivits in och löses på den datorn.
{% endhint %}

{% hint style="warning" %}
**Windows
: håll projektmappen kort såvida inte långa sökvägar är aktiverade.** När långa sökvägar är avstängda i Windows
(Windows
standardinställning) får ingen filväg vara längre än 260 tecken, och produkterna placeras upp till cirka 130 tecken under projektmappen. `process` kontrollerar detta innan det startar: en projektmapp som är för djup avvisas med meddelandet **”Projektmappen är för djup för den här datorn”**, den längsta projektmapp som passar samt hur man aktiverar långa sökvägar — ingenting bearbetas. Använd en kortare `-o` (eller `set-project-folder`), eller aktivera Windows
långa sökvägar (installationsprogrammets alternativ **Aktivera Windows
långa sökvägar**, markerat som standard) och starta om Chloros
.
{% endhint %}

Inom projektet grupperas produkterna **efter kamera och därefter efter filformat**:

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

Kameramappen är `LATT-<sensor>-<lens>-F<filter>` för LATTICE (vilket stämmer överens med bildens EXIF-data `Model`) och `<model>_<filter>` (t.ex. `Survey3N_RGN`) för Survey3
. Formatmappen följer `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` eller `tiff32` för `TIFF (32-bit, Percent)`.

För en multispektral M3C-kamera kan även en mapp med namnet `clamp_masks/` förekomma i en formatmapp, bredvid produktmapparna (till exempel `tiff16/clamp_masks/Reflectance_Calibrated_Images/` och `tiff32/clamp_masks/Radiance_Images/`). Den innehåller maskerna per pixel för strålnings- och reflektansprodukter där bandavblandningen har klämt fast ett negativt värde till noll — se [Klämmasker](output-image-formats.md#clamp-masks). Ingen produktmapp innehåller någonsin en mask.

{% hint style="info" %}
**Varje exporterad produkt behåller namnet på KÄLLFILEN.**En strålnings-export av `capture_..._raw.tif` heter fortfarande `capture_..._raw.tif` – den finns bara i `tiff32/Radiance_Images/`.**Mappen identifierar produkten, inte filnamnet**, så använd glob för katalogen, inte för suffixet `*radiance*`.
{% endhint %}

### De alternativ du faktiskt kommer att använda

| Flagga | Standard | Vad den gör |
| --- | --- | --- |
| `-o, --output PATH` | standardprojektmapp | Projektmappens plats (se ovan). |
| `-n, --project-name NAME` | tidsstämpel | Projektnamn. |
| `--format FMT` | `TIFF (16-bit)` | Ett av `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros
skalar inte upp: på bilder som helt består av 8-bitarsdata skrivs ett djupare format som `TIFF (8-bit)`, och detta anges en gång. |
| `--indices NAME [NAME ...]` | ingen | Vegetationsindex att exportera (se [Vegetationsindex](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neural debayer, långsammare, högsta kvalitet (Chloros
+, kräver en CUDA-GPU: utan en sådan använder körningen `standard`; avvisas för LATTICE-bildtagningar, som alltid får `standard`). |
| `--vignette / --no-vignette` | på | Vignettkorrigering. |
| `--reflectance / --no-reflectance` | på | Reflektanskalibrering; för LATTICE är detta även växlingsknappen för reflektansprodukten. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Tvinga startpunkten för bearbetningskedjan för LATTICE-TIFF-filer. |

För allt annat — inställning av måldetektering, PPK, exponeringspinnar, flaggor för matrisjustering — se avsnittet [`process` i CLI
-referensen](reference/cli-reference.md).

***

## Välja vad som ska exporteras (LATTICE-produkter)

LATTICE-bearbetningen fördelas till **alla tillämpliga produkter i ett enda steg**. Fyra produktspecifika växlingsalternativ är alla**aktiverade som standard**; använd formuläret `--no-` för att inaktivera ett:

| Väljare | Produkt |
| --- | --- |
| `--debayered` | Linjär demosaik → `Debayered_Images/`: kalibreringspaketets mörka offset avlägsnas, därefter en kantmedveten demosaik, utan linearitetskorrigering, flat field eller unmix — samma fil som en bild sparar på sin `debayered`-nivå. Behålls i sensorns ursprungliga orientering (sparas i XMP). |
| `--preview` | Förhandsgranskning (vitbalans + gamma; falskfärgssträckning för multispektral) → `Preview_Images/` |
| `--radiance` | float32 strålningsintensitet, W/m²/sr/nm → `Radiance_Images/` (alltid `tiff32/`) |
| `--reflectance` | uint16 reflektans, Pix4D-kompatibel → `Reflectance_Calibrated_Images/` |

RGB
-huvudkameror avger alltid endast debayered + förhandsgranskning — strålning/reflektans per band är inte meningsfullt för en bredbandssensor, så dessa växlingsalternativ har ingen effekt för dem. Survey3
`.raw` ignorerar växlingsalternativen och följer standardvägen för reflektans/mål.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (standard `auto`) väljer reflektansreferensen: `auto` gör ett QA-godkänt [kalibreringsmål](calibration-targets.md) inom bildrutan till den absoluta referensen och faller tillbaka till DAQ-ljussensorns nedåtriktade delningsvärde (ρ = π·L/E) när inget mål finns närvarande (ett mål som klarar kvalitetskontrollen med en flagga konkurrerar med DAQ om det uppmätta värdet); `target` är strikt (ingen DAQ-ersättning); `daq` är DAQ-auktoritativt. Mätningar av mål per enhet kan tillhandahållas med `--target-reflectance-dir`.

{% hint style="info" %}
**Avläsning av reflektanspixlar:**DN-värdet som betyder ρ = 1,0 är**per källa** — LATTICE-filer stämplar `Chloros:PixelScale=32768` i XMP; Survey3
-filer använder 65535 och har ingen `Chloros:PixelScale`-tagg (de har dock andra `Chloros:*`-bearbetningstaggar). Läs av taggen och dividera med den istället för att anta ett konstant värde. Detaljer och det enda avsiktliga specialfallet utan skala finns i [CLI
-referensen](reference/cli-reference.md).
{% endhint %}

**Bearbetningen börjar alltid från `raw`.** Härledda produkter (export av debayering/strålning/reflektans) matas aldrig tillbaka genom bearbetningskedjan — att importera dem på nytt och bearbeta dem skulle innebära att kalibreringsberäkningarna tillämpas två gånger, så Chloros
hoppar över dem och meddelar detta. `--input-level` är den avsiktliga nödutgången när du verkligen behöver tvinga fram en startpunkt.

***

## När en körning misslyckas

Från och med version 1.2.0 misslyckas `process` tydligt istället för att ”lyckas” utan att visa något resultat:

* En körning som **begärde produkter men inte skrev ut några**— endast `project.json` och `calibration_data.json` — skriver ut `Processing finished but wrote no image products.` och**avslutas med ett värde som inte är noll**, så att skript kan upptäcka det. De vanligaste orsakerna: inmatningsmappen kändes inte igen som en inspelning (kontrollera layouten och `--input-level`), eller så var alla begärda produkter otillämpliga för dessa kameror (t.ex. att begära radians/reflektans från kameror som endast har RGB
).
* En **avsiktlig körning endast med metadata** (alla produktalternativ avstängda, ingen `--indices`) räknas fortfarande som lyckad — en tom bildutmatning är det korrekta resultatet i det fallet.
* Kör om med `--verbose` och kontrollera backend-loggen efter raderna `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, som förklarar vilka kameror som hoppats över.
* I Windows
, där långa sökvägar är inaktiverade, avvisas en projektmapp som är för djup för gränsen på 260 tecken **innan** bearbetningen påbörjas, och den längsta projektmappen som ryms används istället (se [Var utdata hamnar](#where-the-outputs-land)).
* Inspelningskommandon (`lattice capture`, `lattice array-capture` och de andra inspelningsfamiljerna) anger namnen på alla filer som de inte kunde skriva och avslutas med `1`; upprepade inspelningar avbryts när enheten är full.

Avslutningskoder: `0` framgång · `1` generellt fel · `2` argumentfel · `130` avbrutet med Ctrl+C.

***

## Vegetationsindex

Kör `--indices` med ett eller flera förinställda namn; varje index hamnar i sin egen `<INDEX>_Index_Images/`-mapp:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

De 22 förinställda namnen som `process --indices` accepterar:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Ett namn som ännu inte finns med i projektet läggs till. Ett namn som inte är en förinställning, eller som kamerornas band inte kan bilda, hoppas över med en varning som anger namnet. Om CLI
:n inte kan läsa projektets aktuella indexlista lämnar den listan oförändrad och visar en varning istället för att ersätta den.

{% hint style="warning" %}
**Det finns tre indexlistor – blanda inte ihop dem.**Rullgardinsmenyn Projektinställningar i GUI:n har 27 formler (lägger till `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` – dessa fem finns endast i GUI och gäller**inte** för `--indices`). Kommandot live/offline `lattice index --preset` använder en egen separat lista med 22 förinställningar. Formler och bandberäkningar finns dokumenterade i [Formler för multispektrala index](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ-ljussensorer: En snabb introduktion

`daq pool-*`-familjen styr MAPIR
s DAQ-spektralsensorer (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) genom backendens permanenta pool – GUI, CLI
och SDK
delar alla ett live-handtag. **`pool-*` är den DAQ-väg som stöds i den medföljande CLI
:n**; andra `daq`-underkommandon som du kan stöta på är en MAPIR
-intern yta endast för källkod och avslutas med ett explicit felmeddelande som hänvisar dig till `pool-*`.

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

`pool-record` utan `--duration` körs tills `pool-record --stop`; standardutmatningskatalogen är `~/Documents/DAQ Live View/` **på backend-maskinen**. Profilen för kap-korrigering väljs vid anslutningstillfället (`--cap-id`, standard för backend `sunshine_cosine`) och kan bytas ut i realtid med `pool-set-cap` — profiler för maximalvärden och sensorns kalibrerade mätområde beskrivs i DAQ-kapitlen i denna handbok.

{% hint style="warning" %}
**DAQ-E på en värd med flera nätverkskort:** den första automatiska upptäckten av `pool-connect --eth` efter uppstart kan misslyckas även om sensorn är i gott skick. `--eth-host <ip-or-hostname>` är den tillförlitliga varianten — använd den när upptäckten inte ger något resultat.
{% endhint %}

***

## LATTICE-kameror, PTP och projektautomatisering

`lattice`-familjen (över 45 underkommandon) täcker LATTICE-kamerahantering från början till slut: upptäckt, enstaka bildtagningar, beständiga synkroniserade matriser med GUI:ns smart-prep-anslutningsflöde, liveförhandsgranskning i webbläsaren, inriktning, indexberäkningar och diagnostik av värd-NIC. Ett smakprov:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Vid sidan av detta: `chloros-cli time-sync` rapporterar om den PTP-grandmaster som Chloros
-värden kör (LATTICE-kameror och DAQ-E-sensorer fungerar som slavar till denna för tidsstämplar mellan enheter), och `chloros-cli project` öppnar ett sparat Chloros
-projekt och styr dess kameror, arrayer och sensorer utan grafiskt gränssnitt – inklusive skriptbaserade YAML-insamlingsrecept.

Dessa tre familjer (`lattice`, `project`, `daq pool-*`) är också de enda som stöder `CHLOROS_BACKEND_URL` för styrning av en **fjärrstyrd** backend; kärnkommandona riktar sig alltid mot den lokala maskinen.

Fullständiga genomgångar finns i LATTICE-kapitlen i denna handbok; alla flaggor finns i [CLI
-referensen](reference/cli-reference.md).

***

## Felsökning: Topp 5

| Symptom | Lösning |
| --- | --- |
| `Login required`, eller ett schemalagt jobb hänger sig vid en `Email:`-prompt | Kör `chloros-cli login EMAIL 'PASSWORD'` en gång på den här datorn – kommandon utan en cachad sessionsprompt körs interaktivt istället för att avbrytas direkt. |
| `backend unreachable` | Starta Chloros
-skrivbordsappen, eller starta backend själv: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` på Windows
(mappen `backend` i din installationsmapp; den finns inte i `PATH`), `chloros-backend` på Linux
. `chloros-cli backend status` visar om en körs; `lattice`/`project`/`daq pool-*` startar aldrig någon. Om du pekar dem mot en fjärrbackend, kontrollera `CHLOROS_BACKEND_URL`. |
| Arrayanslutning blockerad: `FRAMES WILL DROP` / `Reduce ROI to enable` | Värd-NIC:s mottagningsring återställd till standardinställningar — den vanligaste orsaken till att en tidigare fungerande rigg vägrar ansluta, vanligtvis efter en uppdatering av NIC-drivrutinen. Kör `chloros-cli lattice network --fix` från en **terminal med utökade behörigheter** (eller ställ in `ReceiveBufferLen=256`, `PendingReceives=64`); se avsnittet *Host NIC Setup &amp; Tuning* i referensmaterialet. |
| `daq`-underkommandot avslutas: ”kräver det fullständiga DAQ-paketet…” | Använd kommandona `daq pool-*`, som styr samma sensor via backend: anslutning, strömning, inspelning, val av kapacitet och, för en DAQ-E, dess nätverksinställningar, omstart och firmwareuppdateringar (eller `chloros_sdk.connect_daq_sensor()` från Python
). |
| Jetson visar en varning om swap-utrymme före stora mappar | Lägg till filbaserat swap-utrymme — CLI
:n visar exakt vilka `fallocate`/`swapon`-kommandon som ska köras. |

***

## Få hjälp

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Alla flaggor, alla underkommandon:** [CLI
-referens](reference/cli-reference.md)
* **Motsvarighet i Python
:** [Python
SDK
](api-python-sdk.md) och [SDK Reference](reference/sdk-reference.md)
* **Support:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
