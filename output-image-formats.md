---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Utdataformat

Chloros exporterar bearbetade produkter i fyra filformat. Välj format i projektinställningarna (GUI), med `--format` (CLI) eller med `export_format` (SDK). CLI och SDK accepterar exakt de strängar som anges nedan.

| Formatsträng | Filändelse | Pixeltyp | Pixelintervall | Anmärkningar |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(standard)* | `.tif` | uint16-tal | 0 – 65535 | Rekommenderas för fotogrammetri/GIS. |
| `TIFF (8-bit)` | `.tif` | uint8-tal | 0 – 255 | För alla 8-bitarsinspelningar, som endast erbjuder 8-bitarsformat: Chloros uppskalar inte bitdjupet. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % reflektans. Vissa program kan inte läsa TIFF-filer med flyttalsvärden; filerna blir större. |
| `PNG (8-bit)` | `.png` | uint8-tal | 0 – 255 | Förlustfri komprimering, lämplig för visning på webben och visualisering. |
| `JPG (8-bit)` | `.jpg` | uint8-tal | 0 – 255 | Komprimering med förlust, minsta filstorlek. |

## Var utdatafilerna sparas

Produkterna sparas i projektmappen, grupperade efter kamera och sedan efter filformat:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kameramappen är `LATT-<sensor>-<lens>-F<filter>` för LATTICE och `<model>_<filter>` (t.ex. `Survey3N_RGN`) för Survey3. **Varje exporterad produkt behåller källfilens namn – det är mappen som identifierar produkten, inte ett filnamnstillägg.** Se [Var utdata placeras](reference/cli-reference.md) i CLI-referensen för de fullständiga reglerna.

## LATTICE-produkter (insamlings- och exportnivåer)

En råbild från LATTICE fördelas till alla begärda produkter i ett enda steg. Varje produkttyp har sin egen aktiveringsknapp (kryssrutor i GUI eller CLI-kommandona `--debayered` / `--preview` / `--radiance` / `--reflectance`, alla aktiverade som standard):

| Nivå | Innehåll | Datatyp |
| --- | --- | --- |
| `raw` | Bayer-data direkt från sensorn (monokameror: det enda bandet). Bearbetningen utgår alltid från rådata. | Som inspelat |
| `debayered` | Linjär demosaikering — 3-kanals för M3C, 1-kanals gråskala för M3M: kalibreringspaketets mörkförskjutning avlägsnas, därefter en kantmedveten demosaikering, utan linearitetskorrigering, flat field eller unmix. En `process`-export skriver samma fil som en bildtagning sparar på denna nivå, i alla bearbetningslägen. | Linjär DN |
| `radiance` | Absolut spektral strålning från den fullständiga radiometriska kedjan, i **W/m²/sr/nm**. Skrivs alltid som 32-bitars TIFF (`tiff32/Radiance_Images/`), oavsett valt exportformat. | float32 |
| `reflectance` | Reflektans ρ, där **DN 32768 = ρ 1,0 (100 %)** med utrymme upp till ρ 2,0. Pix4D-kompatibel. | uint16 |
| `preview` | Visningsklar rendering: RGB-värden = vitbalans + gamma; multispektral = falskfärgskorrigering. | 8-bitars visning |

### Orientering

`raw`- och `debayered`-filer behåller alltid sensorns **ursprungliga** orientering; kamerans inställning för spegling/rotation tillämpas endast på de andra produkterna (förhandsgranskning, strålningsvärde, reflektans, index). Vid array-inspelningar behålls varje medlems filer i sin ursprungliga orientering såvida du inte anger en orientering (`--orient` i CLI, `orient=True` i SDK). Varje fil registrerar den orientering som hör till dess pixlar i XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` – och `ChlorosOrientationApplied` anger om dess egna pixlar bär denna information (`false` för råa och debayrade filer). Chloros Image Viewer anpassar RAW- och debayered-filer enligt den posten endast för visning; själva filerna skrivs aldrig om. En fil utan sådan post (skriven före version 1.3.0 eller med ett annat verktyg) visas som den är lagrad.

## Klämmasker

Strålnings- och reflektansprodukterna från en multispektral M3C-kamera genomgår en 3×3-bandsavblandning. Om avblandningen ger ett negativt värde i ett band, klipps det värdet till noll (livevisningar klipps på exakt samma sätt). Nytt i version 1.3.0 är att en produkt där minst en pixel har klippts får en tillhörande **klämmask** som anger exakt vilka pixlar och band det gäller:

* **Där:**i en `clamp_masks/`-mapp**bredvid** produktmapparna, aldrig inuti en sådan: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`, och `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Vad:** en `uint8`-TIFF-fil med ett enda band på produktens eget pixelrutnät (inklusive orientering, ROI, binning och matrisjustering). Bit 0 (`mask & 1`) anges där bandet i produktens R-kanal klipptes, bit 1 (`& 2`) G-kanalen, bit 2 (`& 4`) B-kanalen. Andelen klippta pixlar per band stämplas också på produkten som `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` och `Chloros:BandBUnmixClampFraction`.
* **Så här anges det i produkten:** XMP-taggen `Chloros:UnmixClampMask` innehåller maskens sökväg relativt produktens mapp (`../clamp_masks/<product folder>/<name>_clampmask.tif`, alltid separerade med `/`), `none` när ingen pixel klipptes bort (ingen mask skrivs), eller `unavailable` när en mask inte kunde skrivas. En produkt utan tagg har aldrig genomgått avblandningen (RGB- och monokameror, rådata och debayered-nivåer).

Eftersom ingen produktmapp innehåller en mask kan du överföra en produktmapp som till exempel `Reflectance_Calibrated_Images/` till Pix4D, Metashape eller annan fotogrammetriprogramvara precis som den är, inklusive undermappar – överför produktmappen, inte formatmappen ovanför den. Chloros egen import hoppar över clamp-masker. CLI-verktyget räknar dem separat: `Image products written: N (plus M clamp masks)`.

## Indexprodukter

Varje index hamnar i sin egen `<INDEX>_Index_Images/`-mapp (t.ex. `NDVI_Index_Images/`).

* **Heltalsformat**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) lagrar en**linjär sträckning per bild**, inte en −1…1-mappning: DN 0 är den bildrutan lägsta indexvärde och full skala dess högsta. Filen innehåller `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` och `Chloros:IndexMax`, alltså även `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Eftersom varje bildruta har sina egna ändpunkter ska bildrutor jämföras som värden, inte som DN.
* **`TIFF (32-bit, Percent)`** lagrar själva indexvärdena som float32, utan omskalning — det bästa valet för kvantitativt arbete över en flygning.

Nytt i 1.3.0: en indexprodukt beskriver **sig själv** istället för att upprepa källans rådata-taggar. Dess GPS- och kamerataggar kopieras från den produkt den beräknades utifrån (reflektansprodukten om sådan finns), och sedan:

| Tagg | Värde |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (så att filen aldrig importeras på nytt som en inspelning) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / den använda formeln |
| `Chloros:IndexLevel` | `reflectance` eller `radiance` — vad indexet beräknades utifrån (saknas när inget av dem förekommer) |
| `Chloros:IlluminationInvariant` | `True` för ett index beräknat utifrån reflektans, `False` utifrån strålningsintensitet eller sensorvärden |
| `Chloros:ChlorosPipelineVersion` | den Chloros-version som skapade indexet |

`Chloros:PixelScale` har tagits bort från indexprodukterna — använd `IndexMin` / `IndexMax` ovan. Det fullständiga receptet för återläsning finns under ”En heltalindexraster är en kontraststräckning per bild” i [CLI-referensen](reference/cli-reference.md).

## Läsa in reflektansvärden för pixlar

Reflektansen lagras som ett heltal (DN), och **det DN-värde som motsvarar ρ = 1,0 (100 % reflektans) beror på källkameran**:

| Källkamera | ρ = 1,0 är DN | Hur man avgör |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (utrymme upp till ρ 2,0) | XMP-taggen `Chloros:PixelScale=32768` är inbäddad i filen. |
| Survey3 | `65535` (klippt vid ρ 1,0) | Ingen `Chloros:PixelScale`-tagg – avsaknaden är tecknet. (Produkter från Survey3 har andra `Chloros:*`-bearbetningstaggar, såsom `Chloros:CalibrationApplied`, i alla bearbetningslägen från och med version 1.3.0.) |

**Läs av `Chloros:PixelScale` XMP-taggen och dividera med dess värde** istället för att anta en konstant. Taggen är definierad i uint16-domänen, så den förblir `32768` i utdataformat som skalar om – normalisera först den lagrade datatypen tillbaka till uint16 (×257 från 8-bit, ×65535 från float32).

{% hint style="warning" %}
**8-bitarsinspelningar.** Exportdata från en 8-bitarsinspelning är 8-bitars. Dess reflektans omskalas och märks med `Chloros:PixelScale=32768`, så att skalan bibehålls; 8-bitars DN-produkter och reflektansfiler som skrivits före version 1.3.0 beskärs till 0–255 och har ingen skalmärkning. Om märkningen saknas på en LATTICE-reflektansfil ska du inte förutsätta någon skala; exportera istället om den till 16-bitars eller 32-bitars format.
{% endhint %}

För de fullständiga reglerna (inklusive de MicaSense-kompatibla taggarna), se **”Läsa reflektanspixlar”** i [CLI-referensen](reference/cli-reference.md).
