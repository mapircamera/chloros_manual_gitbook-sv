# Bearbetningspipeline

Chloros 1.3.0 använder en bearbetningspipeline med fyra trådar som fungerar som ett stegvis uppbyggt löpande band. Varje tråd hanterar en särskild fas i arbetsflödet, vilket innebär att flera bilder samtidigt kan befinna sig i olika stadier av bearbetningen.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Pipelinens arkitektur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Varje bild passerar genom alla fyra trådarna i tur och ordning. Med Chloros+ multitrådsbearbetning upptar flera bilder olika trådar samtidigt – medan tråd 3 bearbetar en bild kan tråd 1 detektera nästa, tråd 2 kalibrera en annan och tråd 4 skriva en färdig bild till disken.

Framsteg rapporteras per tråd och strömmas via Server-Sent Events (backend publicerar dem på `/api/events`). I CLI:s realtidsvisning av framsteg är de fyra stegen märkta **Detektering, Analys, Bearbetning, Export**.***

## Trådinformation

### Tråd 1: Detektering

**Syfte**: Ladda bilder och detektera kalibreringsmål.

* Läser bildfiler från disken – Survey3-paren `.raw`+`.jpg`, LATTICE `.tif`/`.tiff`-bilder samt `.dng`
* Extraherar EXIF-metadata (GPS, kameramodell, tidsstämplar, exponering)
* Detekterar kalibreringsmål: ArUco-märkta målgeometrier för LATTICE-bildtagningar, samt den klassiska paneldetektorn för Survey3-kalibreringsmålsbilder
* Utdata: bilddata + metadata + resultat från måldetektering

I huvudsak en I/O- och CPU-bunden tråd.

### Tråd 2: Kalibrering

**Syfte**: Beräkna kalibreringsparametrar utifrån detekterade mål.

* Beräknar reflektanskalibreringskoefficienter från målbilder
* Beräknar parametrar för vignetteringskorrigering
* Fastställer kalibreringskurvor per band
* Utdata: kalibreringsparametrar för varje bild

En CPU-bunden beräkningstråd. Vid en Survey3-körning eller en blandad körning väntar tråd 3 på den när reflektanskalibrering är aktiverad, så att dess koefficienter är klara innan någon bild bearbetas. Vid en ren LATTICE-körning körs tråd 3 under detekteringen: varje bildruta startar när dess målankare är fastställt. En bildram tar det tidsmässigt närmaste målet på vardera sidan, så den väntar tills varje bildram från dess kamera som ligger närmare i tiden än dess nuvarande närmaste mål har skannats. Produkterna är desamma, men levereras tidigare.

### Tråd 3: Bearbetning (GPU)

**Syfte**: Tillämpa korrigeringar och beräkna vegetationsindex.**Detta är den mest beräkningsintensiva tråden.*** **Debayering**: konverterar RAW-Bayer-data till flerkanaliga bilder
  * Standard (snabb, medelhög kvalitet) — standardinställning, `--debayer standard`
  * Texture Aware (långsam, högsta kvalitet) — endast Chloros+, `--debayer texture-aware`, använder en AI/ML-modell för brusreducering
  * LATTICE mono (M3M)-bildtagningar är enkelbandsbilder: demosaik- och vitbalansstegen hoppas över för dessa (med ett loggmeddelande på en rad), medan eventuella M3C/Bayer-bilder i samma körning fortfarande genomgår dessa steg
* **Vignettkorrigering**: tillämpar korrigering av objektivvignettering över hela bilden
* **Reflektanskalibrering**: tillämpar kalibreringskoefficienter för att konvertera till reflektansvärden
* **Indexberäkning**: beräknar vegetationsindex (NDVI, NDRE, GNDVI, …)
* Utdata: bearbetade bilddata redo för export

Denna tråd drar störst nytta av GPU-acceleration, och det är den tråd som [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) finjusterar.

### Tråd 4: Export

**Syfte**: Skriva ut bearbetade bilder till disk.

* Skriver utdatafiler i det valda formatet — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Bäddar in metadata i utdatafilerna (GPS, tidsstämplar, bearbetningsparametrar) — inklusive bearbetningstaggarna `Chloros:*`, som från och med version 1.3.0 skrivs in i alla bearbetningslägen, både för Survey3 och LATTICE
* Organiserar utdata under projektmappen som `<camera>/<format>/<Product>_Images/` — till exempel `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Exporterade filer behåller källfilens namn; mappen identifierar produkten.**
* För LATTICE-inspelningar kan en kältram delas upp i flera produkter (Debayered, Preview, Radiance, Reflectance, Index), var och en i sin egen produktmapp. Indexprodukter beräknas utifrån den reflektansprodukt som körningen skapade (om ingen sådan finns, utifrån en annan produkt som körningen skapade, till exempel radians). Radiance- och reflektansprodukter från multispektrala M3C-kameror kan också få en klämmask per pixel i `<format>/clamp_masks/` (se [Utgångsbildformat](../output-image-formats.md#clamp-masks))
* Utdata: slutliga filer på disken

I huvudsak en I/O-bunden tråd — SSD-lagring förbättrar prestandan märkbart.

***

## Under huven: Exekutorer

Inom tråd 3 parallelliseras arbetet per bild enligt följande:

* **GPU-strategier**(`GPU_SINGLE`, `GPU_PARALLEL`) använder en pool av**arbetsprocesser**, var och en en separat OS-process med sitt eget CUDA-sammanhang. Chloros startar varje arbetsprocess genom att starta sitt eget installerade backend-program på nytt – inte via Pythons `multiprocessing`, vars start av arbetsprocesser inte fungerar inuti den paketerade applikationen
* **`CPU_PARALLEL`** använder en trådpool (Pythons `ThreadPoolExecutor`) — NumPy och OpenCV frigör GIL, så trådar räcker
* Jetson-enheter med 8 GB eller mindre delat RAM-minne hoppar över exekutorn helt och hållet och bearbetar sekventiellt inom processen
* Texture Aware på en GPU med mindre än 7 GB VRAM körs också sekventiellt — brusreduceringsmodellen ryms inte mer än en gång

Chloros använder inget distribuerat ramverk från tredje part (såsom Ray). Se [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) för information om hur strategin och antalet arbetare väljs.

***

## Sekventiell vs. pipelined bearbetning

### Fritt läge (sekventiellt)

I gratisversionen av Chloros bearbetas bilderna **en i taget**, sekventiellt genom alla fyra stegen:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI:n visar en förenklad förloppsindikator i gratisversionen; dess sekventiella faser visas som **Måldetektering**och sedan**Bearbetning**.

### Chloros+-läge (pipeline)

Med en Chloros+-licens arbetar alla fyra trådar **samtidigt** med olika bilder:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI:ns förloppsindikator visar de fyra stegen; håll muspekaren över den för att se förloppet per tråd. I CLI visas samma fyra steg i realtid som **Detektering, Analys, Bearbetning, Export**.

{% hint style="info" %}
**En etikett, två namn.** CLI:n kallar steg 3 för _Processing_. Backendens förloppsflöde i premiumläge – det som GUI:ns förloppsindikator visar – betecknar samma steg som _Calibrating_. Det är samma tråd som utför samma arbete (Tråd 3: debayer, korrigeringar, index).
{% endhint %}

{% hint style="success" %}
**Pipelined-bearbetning med Chloros+** kan vara 3–5 gånger snabbare än sekventiell bearbetning, beroende på din hårdvara och datamängdens storlek. Hastighetsökningen är störst på system med snabba GPU:er och SSD-enheter.
{% endhint %}

***

## Tråd 4: Exportförlopp

Exporttråden har sin egen förloppsspårning, som du kan avfråga separat:

**Kommandoradsgränssnitt (CLI):**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Bearbetningen är klar när tråd 4 når 100 %.

{% hint style="info" %}
**En körning som inte skriver ut några bilder betraktas som misslyckad.**Vid framgång rapporterar `chloros-cli process` hur många bildprodukter som skrevs ut (`Image products written: N`). Om produkter begärdes och**inga**skrevs ut — endast `project.json` och `calibration_data.json` — skriver CLI:et ut `Processing finished but wrote no image products.` och**avslutas med ett värde som inte är noll**, med angivande av projektmappen och de vanligaste orsakerna (inmatningsmappen kändes inte igen som en inspelning – kontrollera layouten och `--input-level` – eller så var alla begärda produkter olämpliga för dessa kameror). Skript kan förlita sig på avslutningskoden.
{% endhint %}

***

## Samband med dynamisk beräkningsanpassning

[Dynamisk beräkningsanpassning](dynamic-compute-adaptation.md) påverkar främst **tråd 3 (bearbetning)**:

* **`GPU_PARALLEL`**: Tråd 3 kör flera bilder genom GPU:n samtidigt med hjälp av `fused_gpu`-pipeline
* **`GPU_SINGLE`**: Tråd 3 serialiserar GPU-åtkomsten med en semafor medan arbetsprocesser överlappar I/O, med hjälp av `fused_gpu` eller den minneseffektiva pipelinen `tiled_gpu`
* **`CPU_PARALLEL`**: Tråd 3 använder CPU-baserad bearbetning med multitrådad parallellitet

Tråd 3:s GPU-minnesallokering växer också när trådarna 1 och 2 avslutas — se [Dynamisk GPU-minnesallokering](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Nästa steg

* [Dynamisk beräkningsanpassning](dynamic-compute-adaptation.md) — Hur Chloros väljer den optimala strategin för din hårdvara
* [NVIDIA Jetson-guide](../linux/nvidia-jetson-guide.md) — Plattformsspecifikt pipelinebeteende på Jetson
* [Övervakning av bearbetningen](../processing-images-gui/monitoring-the-processing.md) — Övervakning av förloppet via GUI
* [Referens för CLI](../reference/cli-reference.md) — `process`, `export-status`, avslutningskoder och utdatalayout
