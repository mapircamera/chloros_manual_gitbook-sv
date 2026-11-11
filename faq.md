---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Vanliga frågor

<details>

<summary>Kan jag bearbeta bilder från kameror som inte är av märket MAPIR med Chloros?</summary>

Nej, Chloros stöder endast bearbetning av bilder från MAPIR-kameror — serierna Survey3 och LATTICE. Se listan över [kameramodeller som stöds](supported-cameras.md) för mer information. Vi erbjuder bearbetning av bilder från andra kameror i MAPIR-molnet, se hela listan [här](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Stöder Chloros LATTICE-kameror?</summary>

Ja. Chloros 1.3.0 stöder LATTICE M3C- och M3M-kameramoduler från början till slut: **livekontroll**— upptäck, anslut, förhandsgranska och ta bilder från fliken Kameror i GUI:n, `chloros-cli lattice`, eller Python-SDK, inklusive synkroniserade flerkamerasystem med PTP-tidssynkronisering — samt**fullständig radiometrisk bearbetning** av bilder (rådata → debayering → strålningsintensitet → reflektans → index). Se [Kameror som stöds](supported-cameras.md) och [LATTICE-guiden](lattice/README.md).

</details>

<details>

<summary>Kan jag kalibrera mina bilder för reflektans utan ett kalibreringsmål?</summary>

**Survey3:** Nej. Utan en bild av kalibreringsmålet som tagits i samband med att bilderna utan målet togs kommer du inte att kunna koppla bildens pixelvärden till en känd reflektansprocent. Om du dessutom inte inkluderar loggen från en MAPIR-ljussensor kommer omgivningsljusets spektrum inte att mätas, och reflektansresultaten blir inte korrekta.

**LATTICE:** Ja. Reflektansen kan relateras till den nedåtriktade irradians som mäts av en DAQ-ljussensor istället för en panel (ρ = π·L/E). När ett QA-godkänt mål *finns* i bildramen blir det som standard den absoluta referensen (`--reflectance-source auto`), såvida det inte godkänns med en flagga, då målet och DAQ-kedjorna konkurrerar om det uppmätta poängtalet. Ett undantag: ”F988-reflektansen kalibreras med hjälp av en reflektanspanel i scenen: bandet ligger utanför DAQ-ljussensorns kalibrerade område, så Chloros tillämpar din senaste panelavläsning och behåller den mellan panelavläsningarna.” Se [Kalibreringsmål](calibration-targets.md).

</details>

<details>

<summary>Behöver jag en DAQ-ljussensor?</summary>

Inte för strålning: LATTICE-strålningsdata exporteras från varje kameras fabriksradiometriska kalibrering och kräver varken en DAQ-sensor eller ett mål. För **reflektans**behöver du en referens för omgivningsljuset – antingen en DAQ-ljussensors nedåtriktade mätning eller ett kalibreringsmål i bildrutan. Med en DAQ-sensor kan du ta fram kalibrerad reflektans**utan att placera några paneler i scenen**. Inspelade `.daq`-filer matchas automatiskt med dina bilder utifrån tidsstämpeln. Se [Kalibreringsmål](calibration-targets.md) och [CLI-referensen](reference/cli-reference.md).

</details>

<details>

<summary>Kan jag använda Chlorost med en AI-assistent (Claude, ChatGPT, etc.)?</summary>

Ja — den här manualen och CLI/SDK är utformade för just det:

* Hela manualens index finns tillgängligt på `https://mapir.gitbook.io/chloros/llms.txt` så att AI-assistenter kan hitta varje sida.
* Varje sidas råa Markdown-kod finns tillgänglig på sidans URL i gemener med `.md` tillagt (till exempel `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-referensen](reference/cli-reference.md) och [SDK-referensen](reference/sdk-reference.md) är skrivna för användning av stora språkmodeller (LLM): exakta flaggor, standardvärden, avslutningssemantik och kommandon som går att kopiera och klistra in.

Se [AI-assistenter](ai-assistants.md) för information om hur du kopplar din assistent till Chloros.

</details>

<details>

<summary>Vart hamnar mina bearbetade utdatafiler?</summary>

Produkterna sparas i projektmappen, grupperade efter kamera och sedan efter filformat:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **kameramapp** — `LATT-<sensor>-<lens>-F<filter>` för LATTICE, `<model>_<filter>` (t.ex. `Survey3N_RGN`) för Survey3
* **formatmapp** — `tiff16`, `tiff8`, `png8`, `jpg8` eller `tiff32`
* **produktmappar** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (alltid under `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — endast LATTICE multispektral: pixelspecifika masker bredvid produktmapparna, se nedan**Exporterade filer behåller källfilens namn — mappen identifierar produkten, inte ett filnamnstillägg.**Med CLI skapas projektmappen under din standardprojektmapp (`Documents/MAPIR/Chloros Projects` om du inte har ändrat den) såvida du inte anger `-o`; ett relativt `-o` tolkas relativt den mapp du kör kommandot från. Observera att en körning av `chloros-cli process` som begärde produkter men inte skrev ut några visar `Processing finished but wrote no image products.` och**avslutas med ett värde som inte är noll**, så att skript kan upptäcka detta. Se [Utdataformat](output-image-formats.md) och [CLI-referensen](reference/cli-reference.md).

</details>

<details>

<summary>Vad är mapparna clamp_masks och *_clampmask.tif-filerna?</summary>

Nytt i 1.3.0. Radian- och reflektansprodukterna från en multispektral LATTICE M3C-kamera genomgår en 3×3-bandsavblandning, och där det ger ett negativt värde i ett band klipper Chlorosper det värdet till noll. En produkt med minst en klippt pixel får en enkelbandsmask av typen `uint8` som markerar vilka pixlar och band som har klippts (bit 0 = produktens R-kanal, bit 1 = G, bit 2 = B). Maskerna sparas i en `clamp_masks/`-mapp **bredvid** produktmapparna — till exempel `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — och produktens `Chloros:UnmixClampMask` XMP-tagg namnger dess mask (eller anger `none`).

De är inte bilder av scenen: Chloros egen import hoppar över dem, och eftersom de aldrig ligger i en produktmapp kan du överföra en produktmapp till Pix4D eller Metashape precis som den är. Se [Clamp-masker](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Varför säger Chloros: ”Projektmappen är för djup för den här datorn”?</summary>

Detta gäller endast Windows. Windows levereras med långa sökvägar avstängda, och då får ingen filväg vara längre än 260 tecken. Chloros-produkterna ligger upp till cirka 130 tecken under projektmappen, så Chloros avvisar en djupare projektmapp innan bearbetningen påbörjas – istället för att köra programmet och inte skriva ut något, vilket kunde hända före version 1.3.0. Meddelandet anger den längsta projektmappen som passar.

Åtgärda det på något av följande sätt:

* Använd en kortare projektmapp (**Arbetsmappen** i Projektinställningar, eller `-o` / `chloros-cli set-project-folder` för CLI).
* Aktivera långa sökvägar i Windows och starta om Chloros. Alternativet **Aktivera långa sökvägar i Windows** i Chloros-installationsprogrammet (markerat som standard) gör detta; kör installationsprogrammet igen om du avmarkerade det.

Se [Starta bearbetningen](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Kan jag redigera mina bilder innan de bearbetas i Chloros?</summary>

Nej. Chloros utgår från att indata inte har modifierats. Ändra inte filnamnen.

</details>

<details>

<summary>Kan jag ställa in mina MAPIR Survey3-kameror på automatisk exponering och bearbeta bilderna i Chloros?</summary>

Nej. Survey3-bilddatauppsättningar måste ha en fast/låst exponering, så ingen automatisk slutartid eller automatisk ISO. Alla bilder från samma kameramodell måste ha identisk slutartid och ISO (exponering).

LATTICE-kameror har inte denna begränsning: Chloros styr deras exponering i realtid (Smart AE), och varje bild registrerar den exponering och förstärkning som faktiskt användes, vilket den radiometriska bearbetningskedjan tar hänsyn till.

</details>

<details>

<summary>Kan Chlorosa bearbeta eller analysera ortomosaikbilder?</summary>

Nej. Endast enskilda bilder från MAPIR-kameror stöds, inte sammanfogade bilder som en ortomosaikkarta.

</details>

<details>

<summary>Hur kan jag påskynda måldetekteringssteget i Chloros?</summary>

Om du i filbläddrarens tabell förväljer målbilderna i den högra kolumnen kommer Chloros att endast söka efter kalibreringsmål i dessa bilder, vilket avsevärt påskyndar bearbetningen.

</details>

<details>

<summary>Om jag ska ladda upp mina bilder till <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> bör jag då bearbeta dem i Chloros innan uppladdningen?</summary>

Om du planerar att ladda upp till vår onlinebehandlingsplattform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) ska du inte redigera bilderna innan du laddar upp dem. Cloud utför samma bearbetning och mer därtill.

</details>

<details>

<summary>Kommer MAPIR någonsin att stödja funktionen X? Jag önskar verkligen att MAPIR erbjöd X.</summary>

Vi är alltid intresserade av att få feedback på våra produkter. Om du upptäcker ett problem med våra produkter eller har ett förslag på hur vi kan förbättra dem, vänligen [KONTAKTA OSS](https://www.mapir.camera/community/contact) för att dela med dig av dina tankar. Största delen av vår forskning och utveckling styrs av att lyssna på våra kunders största behov.

</details>

<details>

<summary>Finns Chloros tillgängligt för Linux?</summary>

Ja! Chloros 1.3.0 stöder Linux amd64 (x86_64) och arm64 (NVIDIA Jetson JetPack 6) via `.deb`-paket. CLI och Python-SDK stöds fullt ut på Linux, inklusive styrning av LATTICE-kameror och DAQ-sensorer i realtid. Det finns inget grafiskt gränssnitt (GUI) för Linux – all interaktion sker via [CLI](CLI.md) eller [Python-SDK](api-python-sdk.md). Se [Översikt över Linux](linux/linux-overview.md) för mer information.

</details>

<details>

<summary>Kan jag köra Chloros på NVIDIA Jetson?</summary>

Ja! Chloros stöder NVIDIA Jetson **Orin-familjen** – Orin Nano, Orin NX och AGX Orin – som kör JetPack 6. Äldre modeller (Jetson Nano, TX2, Xavier NX) kan inte köra JetPack 6 och stöds inte av det aktuella paketet. Chloros identifierar automatiskt din Jetson-modell och optimerar bearbetningsstrategin. Jetson-paketet använder JetPack 6:s CUDA-version av PyTorch, så bearbetningen sker på Jetsons GPU. Se [NVIDIA Jetson-guiden](linux/nvidia-jetson-guide.md) för instruktioner om installation och driftsättning.

</details>

<details>

<summary>Optimerar Chloros automatiskt för min hårdvara?</summary>

Ja! Chloros innehåller [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) som automatiskt känner av din CPU, GPU, RAM och (på Jetson) temperatursensorer. Därefter väljer det den optimala bearbetningsstrategin — från `GPU_PARALLEL` på GPU:er med stort minne till `GPU_SINGLE` på mindre GPU:er och vidare till `CPU_PARALLEL` på system utan en CUDA-GPU som PyTorch kan använda. Ingen manuell konfiguration behövs.

</details>

<details>

<summary>Vad är bearbetningspipeline med 4 trådar?</summary>

Chloros använder en 4-trådig pipelinerad arkitektur för Chloros+-användare: Tråd 1 (Detektering) laddar bilder och detekterar kalibreringsmål, Tråd 2 (Kalibrering) beräknar reflektanskalibrering, Tråd 3 (Bearbetning) utför GPU-accelererad debayering och indexberäkning, och Tråd 4 (Export) skriver utdatafiler. Flera bilder kan befinna sig i olika trådar samtidigt för maximal genomströmning. Se [Bearbetningspipeline](processing-architecture/processing-pipeline.md) för mer information.

</details>

<details>

<summary>Hur kör jag diagnostik på min Chloros-installation?</summary>

Använd kommandot `selftest` för att köra ett 7-stegs smottest: version, porttillgänglighet, uppstart av backend, API-anslutning (`/api/test`), systeminformation (`/api/system-info` — GPU/CUDA/PyTorch), förekomst av brusreduceringsmodell samt CUDA- och brusreduceringsberedskap:

```bash
chloros-cli selftest
```

Detta är särskilt användbart på Linux-system för att verifiera GPU- och CUDA-konfigurationen. På en Jetson innebär `CUDA: False` vanligtvis att JetPack 6:s CUDA- och cuDNN-bibliotek saknas: installera dem med `sudo apt install nvidia-jetpack`.

</details>
