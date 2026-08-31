---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Vanliga frågor

<details>

<summary>Kan jag bearbeta bilder från kameror som inte är av märket MAPIR med Chloros?</summary>

Nej, Chloros stöder endast bearbetning av bilder från MAPIR-kameror — serierna Survey3 och LATTICE. Se listan över [kameramodeller som stöds](supported-cameras.md) för mer information. Vi erbjuder bearbetning av andra kameror på MAPIR Cloud, se hela listan [här](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Stöder Chloros LATTICE-kameror?</summary>

Ja. Chloros 1.2.0 stöder LATTICE M3C- och M3M-kameramoduler från början till slut: **livekontroll**— upptäck, anslut, förhandsgranska och ta bilder från fliken Kameror i GUI:n, `chloros-cli lattice` eller Python SDK, inklusive synkroniserade multikamerauppsättningar med PTP-tidssynkronisering — samt**fullständig radiometrisk bearbetning** av bilder (rådata → debayering → strålningsintensitet → reflektans → index). Se [Kameror som stöds](supported-cameras.md) och [LATTICE-guiden](lattice/README.md).

</details>

<details>

<summary>Kan jag kalibrera mina bilder för reflektans utan ett kalibreringsmål?</summary>

**Survey3:** Nej. Utan en bild av kalibreringsmålet som tagits i samband med att bilderna utan målet togs kommer du inte att kunna koppla bildens pixelvärden till en känd reflektansprocent. Om du inte heller inkluderar loggen från en MAPIR-ljussensor kommer omgivningsljusets spektrum inte att mätas, och reflektansresultaten blir inte korrekta.**LATTICE:** Ja. Reflektansen kan relateras till den nedåtriktade irradians som mäts av en DAQ-ljussensor istället för en panel (ρ = π·L/E). När ett QA-godkänt mål *finns* i bildramen blir det som standard den absoluta referensen (`--reflectance-source auto`). Ett undantag: ”F988-reflektansen kalibreras med hjälp av en reflektanspanel i scenen: bandet ligger utanför DAQ-ljussensorns kalibrerade område, så Chloros använder din senaste panelavläsning och behåller den mellan panelavläsningarna.” Se [Kalibreringsmål](calibration-targets.md).

</details>

<details>

<summary>Behöver jag en DAQ-ljussensor?</summary>

Inte för strålningsintensitet: LATTICE-strålningsdata exporteras från varje kameras fabriksinställda radiometriska kalibrering och kräver varken en DAQ-sensor eller ett kalibreringsmål. För **reflektans**behöver du en referens för det omgivande ljuset – antingen en nedåtriktad mätning från en DAQ-ljussensor eller ett kalibreringsmål i bildrutan. Med en DAQ-sensor kan du ta fram kalibrerad reflektans**utan att placera några paneler i scenen**. Inspelade `.daq`-filer matchas automatiskt med dina bilder via tidsstämpel. Se [Kalibreringsmål](calibration-targets.md) och [CLI-referensen](reference/cli-reference.md).

</details>

<details>

<summary>Kan jag använda Chloros med en AI-assistent (Claude, ChatGPT osv.)?</summary>

Ja – den här manualen och CLI/SDK är utformade för just det:

* Hela manualens index finns på `https://mapir.gitbook.io/chloros/llms.txt` så att AI-assistenter kan hitta varje sida.
* Varje sidas råa Markdown-kod finns tillgänglig på dess småbokstavssida URL med `.md` tillagt (till exempel `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-referensen](reference/cli-reference.md) och [SDK-referensen](reference/sdk-reference.md) är skrivna för användning av LLM: exakta flaggor, standardvärden, avslutningssemantik och kommandon som går att kopiera och klistra in.

Se [AI-assistenter](ai-assistants.md) för information om hur du pekar din assistent mot Chloros.

</details>

<details>

<summary>Vart hamnar mina bearbetade utdatafiler?</summary>

Produkterna sparas i projektmappen, grupperade efter kamera och sedan efter filformat:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **kameramapp** — `LATT-<sensor>-<lens>-F<filter>` för LATTICE, `<model>_<filter>` (t.ex. `Survey3N_RGN`) för Survey3
* **formatmapp** — `tiff16`, `tiff8`, `png8`, `jpg8` eller `tiff32`
* **produktmappar** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (alltid under `tiff32`), `<INDEX>_Index_Images/`**Exporterade filer behåller källfilens namn — mappen identifierar produkten, inte ett filnamnstillägg.**Med CLI skapas projektmappen bredvid inmatningsmappen, såvida du inte anger `-o`. Observera att en körning av `chloros-cli process` som begärde produkter men inte skrev ut några skriver ut `Processing finished but wrote no image products.` och**avslutas med ett värde som inte är noll**, så att skript kan upptäcka detta. Se [Utgångsbildformat](output-image-formats.md) och [CLI-referensen](reference/cli-reference.md).

</details>

<details>

<summary>Kan jag redigera mina bilder innan bearbetning i Chloros?</summary>

Nej. Chloros förutsätter att indata inte har ändrats. Ändra inte filnamnen.

</details>

<details>

<summary>Kan jag ställa in mina MAPIR och Survey3-kameror på automatisk exponering och bearbeta bilderna i Chloros?</summary>

Nej. Bilddatauppsättningar från Survey3 måste ha en fast/låst exponering, så ingen automatisk slutartid eller automatisk ISO. Alla bilder från samma kameramodell måste ha identisk slutartid och ISO (exponering).

LATTICE-kameror har inte denna begränsning: Chloros styr exponeringen i realtid (Smart AE), och varje bild registrerar den exponering och förstärkning som faktiskt användes, vilket den radiometriska bearbetningskedjan tar hänsyn till.

</details>

<details>

<summary>Kan Chloros bearbeta eller analysera ortomosaikbilder?</summary>

Nej. Endast enskilda MAPIR-kamerabilder stöds, inte sammanfogade bilder som en ortomosaikkarta.

</details>

<details>

<summary>Hur kan jag påskynda målidentifieringssteget i Chloros?</summary>

Om du i filbläddrarens tabell förväljer målbilderna i den högra kolumnen kommer Chloros att endast söka efter kalibreringsmål i dessa bilder, vilket avsevärt påskyndar bearbetningen.

</details>

<details>

<summary>Om jag ska ladda upp mina bilder till <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> bör jag då bearbeta dem i Chloros innan uppladdningen?</summary>

Om du planerar att ladda upp till vår onlinebehandlingsplattform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) ska du inte redigera bilderna innan du laddar upp dem. Cloud utför samma bearbetning och mer därtill.

</details>

<details>

<summary>Kommer MAPIR någonsin att stödja funktionen X? Jag önskar verkligen att MAPIR erbjöd X.</summary>

Vi är alltid intresserade av att få feedback på våra produkter. Om du upptäcker ett problem med våra produkter eller har ett förslag på hur vi kan förbättra dem, vänligen [KONTAKTA OSS](https://www.mapir.camera/community/contact) för att dela med dig av dina tankar. Största delen av vår forskning och utveckling styrs av att vi lyssnar på våra kunders största behov.

</details>

<details>

<summary>Finns Chloros tillgängligt för Linux?</summary>

Ja! Chloros 1.2.0 stöder Linux amd64 (x86_64) och arm64 (NVIDIA Jetson JetPack 6) via `.deb`-paket. CLI och Python SDK stöds fullt ut på Linux, inklusive styrning av LATTICE-kamera och DAQ-sensor i realtid. Det finns inget grafiskt gränssnitt för Linux — all interaktion sker via [CLI](CLI.md) eller [Python SDK](api-python-sdk.md). Se [Linux Översikt](linux/linux-overview.md) för mer information.

</details>

<details>

<summary>Kan jag köra Chloros på NVIDIA Jetson?</summary>

Ja! Chloros stöder NVIDIA Jetson-plattformar, inklusive Jetson Nano, Orin Nano, Orin NX och AGX Orin med JetPack 6. Chloros identifierar automatiskt din Jetson-modell och optimerar bearbetningsstrategin. Se [NVIDIA Jetson-guiden](linux/nvidia-jetson-guide.md) för instruktioner om installation och driftsättning.

</details>

<details>

<summary>Optimerar Chloros automatiskt för min hårdvara?</summary>

Ja! Chloros innehåller [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) som automatiskt identifierar din CPU, GPU, RAM och (på Jetson) temperatursensorer. Därefter väljer den den optimala bearbetningsstrategin – från `GPU_PARALLEL` på system med stort minne till `GPU_SINGLE` på enheter med begränsade resurser och till `CPU_PARALLEL` på system utan NVIDIA-GPU. Ingen manuell konfiguration behövs.

</details>

<details>

<summary>Vad är 4-trådsbearbetningspipeline?</summary>

Chloros använder en 4-trådig pipelinerad arkitektur för Chloros+-användare: Tråd 1 (detektering) laddar bilder och detekterar kalibreringsmål, tråd 2 (kalibrering) beräknar reflektanskalibrering, tråd 3 (bearbetning) utför GPU-accelererad debayering och indexberäkning, och tråd 4 (export) skriver utdatafiler. Flera bilder kan befinna sig i olika trådar samtidigt för maximal genomströmning. Se [Bearbetningspipeline](processing-architecture/processing-pipeline.md) för mer information.

</details>

<details>

<summary>Hur kör jag diagnostik på min Chloros-installation?</summary>

Använd kommandot `selftest` för att köra ett 7-stegs smottest: version, porttillgänglighet, uppstart av backend, API-anslutning (`/api/test`), systeminformation (`/api/system-info` — GPU/CUDA/PyTorch), förekomst av brusreduceringsmodell och CUDA + brusreduceringsberedskap:

```bash
chloros-cli selftest
```

Detta är särskilt användbart på Linux/Jetson-system för att verifiera GPU- och CUDA-konfigurationen.

</details>
