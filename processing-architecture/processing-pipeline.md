# Bearbetningspipeline

Chloros 1.1.0 använder en bearbetningspipeline med fyra trådar som fungerar som ett stegvis uppbyggt löpande band. Varje tråd hanterar en separat fas i bearbetningsflödet, vilket gör det möjligt att bearbeta flera bilder samtidigt i olika steg.

***

## Pipelinearkitektur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Varje bild går igenom alla fyra trådarna i ordning. Med Chloros+ multitrådsbehandling kan flera bilder befinna sig i olika trådar samtidigt – medan tråd 3 bearbetar en bild kan tråd 1 detektera nästa, tråd 2 kalibrera en annan och tråd 4 skriva en tidigare bearbetad bild till disken.

***

## Tråddetaljer

### Tråd 1: Detektering

**Syfte**: Ladda bilder och detektera kalibreringsmål.

* Läser bildfiler från disken (RAW, JPG)
* Extraherar EXIF-metadata (GPS, kameramodell, tidsstämplar, exponering)
* Detekterar ArUco-kalibreringsmål i markerade målbilder
* Utdata: bilddata + metadata + resultat från måldetektering

Detta är främst en I/O- och CPU-bunden tråd.

### Tråd 2: Kalibrering

**Syfte**: Beräkna kalibreringsparametrar från detekterade mål.

* Beräknar reflektanskalibreringskoefficienter från målbilder
* Beräknar parametrar för vignettkorrigering
* Fastställer kalibreringskurvor per band
* Utdata: kalibreringsparametrar för varje bild

Detta är en CPU-bunden beräkningstråd.

### Tråd 3: Bearbetning (GPU)

**Syfte**: Tillämpa korrigeringar och beräkna vegetationsindex.**Detta är den mest beräkningsintensiva tråden.*** **Debayering**: Konverterar RAW-data i Bayer-mönster till flerkanaliga bilder
  * Standard (snabb, medelhög kvalitet) — standard
  * Texture Aware (långsam, högsta kvalitet) — endast Chloros+, använder AI/ML-brusreducering
* **Vignettkorrigering**: Tillämpar linsvignettkorrigering över hela bilden
* **Reflektanskalibrering**: Tillämpar kalibreringskoefficienter för att konvertera till reflektansvärden
* **Indexberäkning**: Beräknar vegetationsindex (NDVI, NDRE, GNDVI, etc.)
* Utdata: bearbetade bilddata redo för export

Denna tråd drar mest nytta av GPU-acceleration. Systemet [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) optimerar främst denna tråds beteende.

### Tråd 4: Export

**Syfte**: Skriva ut bearbetade bilder till disk.

* Skriver utdatafiler i det valda formatet (TIFF 16-bit, TIFF 32-bit %, PNG, JPG)
* Bäddar in EXIF-metadata i utdatafilerna (GPS, tidsstämplar, bearbetningsparametrar)
* Organiserar utdata i undermappar efter kameramodell
* Utdata: slutliga filer på disken

Detta är främst en I/O-bunden tråd. SSD-lagring förbättrar prestandan för tråd 4 avsevärt.

***

## Sekventiell vs. pipelined bearbetning

### Fritt läge (sekventiellt)

I gratisversionen av Chloros bearbetas bilderna **en i taget**, sekventiellt genom alla fyra stegen:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI:ns förloppsindikator visar två steg: Målidentifiering och Bearbetning.

### Chloros+-läge (pipeline)

Med en Chloros+-licens arbetar alla fyra trådar **samtidigt** med olika bilder:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI:s förloppsindikator visar fyra steg: Detektering, Analys, Kalibrering, Export. Håll muspekaren över förloppsindikatorn för att se förloppet per tråd.

{% hint style="success" %}
**Pipelined-bearbetning med Chloros+** kan vara 3–5 gånger snabbare än sekventiell bearbetning, beroende på din hårdvara och datasetets storlek. Hastighetsökningen är störst på system med snabba GPU:er och SSD:er.
{% endhint %}

***

## Tråd 4: Exportförlopp

I Chloros 1.1.0 har exporttråden (Tråd 4) sin egen dedikerade förloppsspårning. Du kan övervaka exportförloppet separat:

**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Bearbetningen är klar när tråd 4 når 100 %.

***

## Samband med dynamisk beräkningsanpassning

Systemet [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) påverkar främst **tråd 3 (bearbetning)**:

* **`GPU_PARALLEL`**-strategi: Tråd 3 kör flera bilder genom GPU:n samtidigt med hjälp av `fused_gpu`-pipeline
* **`GPU_SINGLE`**-strategi: Tråd 3 bearbetar en bild i taget med hjälp av den minneseffektiva `tiled_gpu`-pipeline
* **`CPU_PARALLEL`**-strategi: Tråd 3 använder CPU-baserad bearbetning med multitrådad parallellitet

Tråd 3:s GPU-minnesallokering ändras också dynamiskt när trådarna 1 och 2 slutförs — se [Dynamisk GPU-minnesallokering](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Nästa steg

* [Dynamisk beräkningsanpassning](dynamic-compute-adaptation.md) — Hur Chloros väljer den optimala strategin för din hårdvara
* [NVIDIA Jetson-guide](../linux/nvidia-jetson-guide.md) — Plattformsspecifikt pipelinebeteende på Jetson
* [Övervakning av bearbetningen](../processing-images-gui/monitoring-the-processing.md) — Övervakning av framsteg via GUI
