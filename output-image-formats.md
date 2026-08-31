---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Bildformat för utdata

Chloros exporterar bearbetade produkter i fyra filformat. Välj format i Projektinställningar (GUI), med `--format` (CLI) eller med `export_format` (SDK). CLI och SDK accepterar exakt de strängar som anges nedan.

| Formatsträng | Tillägg | Pixeltyp | Pixelintervall | Anmärkningar |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(standard)* | `.tif` | uint16 digitalt tal | 0 – 65535 | Rekommenderas för fotogrammetri/GIS. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % reflektans. Vissa program kan inte läsa TIFF-filer med flyttalsvärden; filerna blir större. |
| `PNG (8-bit)` | `.png` | uint8 digitalt tal | 0 – 255 | Förlustfri komprimering, lämplig för visning på webben och visualisering. |
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

Kameramappen är `LATT-<sensor>-<lens>-F<filter>` för LATTICE och `<model>_<filter>` (t.ex. `Survey3N_RGN`) för Survey3. **Varje exporterad produkt behåller källfilens namn – det är mappen som identifierar produkten, inte ett filnamnstillägg.** Se [Var utdata hamnar](reference/cli-reference.md) i CLI-referensen för de fullständiga reglerna.

## LATTICE-produkter (insamlings- och exportnivåer)

En LATTICE-råram fördelas till alla begärda produkter i ett enda steg. Varje produkttyp har sin egen växlingsfunktion (kryssrutor i användargränssnittet, eller CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, alla är aktiverade som standard):

| Nivå | Innehåll | Datatyp |
| --- | --- | --- |
| `raw` | Bayer-data direkt från sensorn (monokromkameror: det enda bandet). Bearbetningen utgår alltid från rådata. | Som inspelat |
| `debayered` | Linjär demosaikering — 3-kanals för M3C, 1-kanals gråskala för M3M. | Linjär DN |
| `radiance` | Absolut spektral strålningsintensitet från hela den radiometriska kedjan, i **W/m²/sr/nm**. Skrivs alltid som 32-bitars TIFF (`tiff32/Radiance_Images/`), oavsett valt exportformat. | float32 |
| `reflectance` | Reflektans ρ, där **DN 32768 = ρ 1,0 (100 %)** med utrymme upp till ρ 2,0. Pix4D-kompatibel. | uint16 |
| `preview` | Visningsklar rendering: RGB = vitbalans + gamma; multispektral = falskfärgssträckning. | 8-bitars visning |

## Avläsning av reflektansvärden för pixlar

Reflektansen lagras som ett heltalsvärde, och **det DN-värde som motsvarar ρ = 1,0 (100 % reflektans) beror på källkameran**:

| Källkamera | ρ = 1,0 är DN | Hur man avgör |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (utrymme upp till ρ 2,0) | XMP-taggen `Chloros:PixelScale=32768` är inpräglad i filen. |
| Survey3 | `65535` (klippt vid ρ 1,0) | Inga `Chloros:*` XMP-taggar – den avsaknaden är tecknet. |

**Läs av XMP-taggen `Chloros:PixelScale` och dividera med den** istället för att anta ett konstant värde. Taggen är definierad i uint16-domänen, så den förblir `32768` i utdataformat som skalar om — normalisera först den lagrade datatypen tillbaka till uint16 (×257 från 8-bit, ×65535 från float32).

{% hint style="warning" %}
**Ett fall har ingen skala, enligt designen.** När en 8-bitars källinspelning (BayerRG8) skrivs som 8-bitars TIFF, klipper pipelinen av till 0–255 istället för att skala om, så ingen skala beskriver filen — Chloros utelämnar medvetet `Chloros:PixelScale` där. Om taggen saknas i en LATTICE-reflektansfil ska du inte utgå från att det finns en skala; exportera istället om i 16-bitars eller 32-bitars format.
{% endhint %}

För de fullständiga reglerna (inklusive de MicaSense-kompatibla taggarna), se **”Läsa reflektanspixlar”** i [CLI-referensen](reference/cli-reference.md).
