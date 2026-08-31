---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Kompatibla kameror

Chloros bearbetar bilder från två MAPIR-kameraserier på **alla plattformar** (Windows, Linux amd64 och Linux arm64/Jetson):

* **Survey3** — Survey3W (bred) och Survey3N (smal) kameror. Inmatning: `RAW+JPG`.
* **LATTICE**— M3C- och M3M-multispektrala kameramoduler. Inmatning: `.tif`/`.tiff`-bildtagningar. LATTICE-kameror kan även**styrs i realtid** från Chloros — via fliken Kameror i GUI (Windows) eller `chloros-cli lattice` / Python och SDK (Windows och Linux) – inklusive synkroniserade flerkamerasystem. Se [LATTICE-guiden](lattice/).

Bearbetningspipeline accepterar även `.dng`-inmatningsfiler.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Tillverkare</th><th width="250">Kameramodell</th><th width="138">Filtermodell</th><th width="187">Bildtyp</th></tr></thead><tbody><tr><td><strong>Tillverkare</strong></td><td><strong>Kameramodell</strong></td><td><strong>Filtermodell</strong></td><td><strong>Bildtyp</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

LATTICE-serien är ett modulärt multispektralt kamerasystem baserat på Sonys IMX265-sensor med global slutare (3,1 MP, 3,45 µm pixlar). Varje kamera lagrar sin identitet som en modellsträng:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros visar den med prefixet `LATT-` (till exempel `LATT-M3M-L41-F550`), och modellsträngen styr allt nedströms — sensorprofil, bandlayout och kalibrering bestäms automatiskt; det finns inget att konfigurera per kamera. Objektivnumret är **det horisontella synfältet i grader**: `L41` = smalt 41°, `L87` = vidvinkel 87°.

Det finns två sensorkonfigurationer:

| Konfiguration | Sensor      | Filtertyp                           | Band per kamera                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayer-färg | Trippelbandpass                       | 3 spektralband från en enda exponering                                 |
| **M3M**       | Monokrom  | Enkelt smalbandigt interferensfilter | 1 kalibrerat band — kombinera flera M3M-kameror för vegetationsindex |

### M3C (Bayer) filteralternativ

| Filter | Band (namn @ centrum nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### M3M (mono) filterkatalog — 23 SKU:er

F-numret är artikelnumret; det uppmätta bandet (stämplat på varje kalibrerad export) är filterskanningen per parti:

| Artikelnummer    | Centrum (nm, uppmätt) | FWHM kanter (nm) | Bredd (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominellt)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775,1                 | 754–796         | 42         |
| F808   | 810,3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominellt)           | 867–893         | 26         |
| F905   | — (nominellt)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_&quot;Bandkanter mäts som värden för halvmaximal bredd från MAPIR:s filterskanningar per parti — samma värden som Chloros stämplar in i varje kalibrerad export.&quot;_ &quot;— (nominellt)&quot; = ingen skanning av partiet ännu; för dessa SKU:er är det angivna centrumet SKU-numret och bredden är tillverkarens uppgift.

† ”Reflektansen för F988 kalibreras med hjälp av en reflektanspanel i scenen: bandet ligger utanför DAQ-ljussensorns kalibrerade område, så Chloros använder din senaste panelavläsning och behåller den mellan panelavläsningarna.” Se [Kalibreringsmål](calibration-targets.md).

För livekamerastyrning, matriser, nätverkskonfiguration och den radiometriska bearbetningskedjan, se [LATTICE-guiden](lattice/).
