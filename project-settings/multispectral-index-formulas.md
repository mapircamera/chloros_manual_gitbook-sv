---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formler för multispektrala index

Indexformlerna nedan använder en kombination av genomsnittliga transmissionsintervall för Survey3-filter:

<table><thead><tr><th align="center">Survey3-filterfärg</th><th width="196.199951171875" align="center">Survey3-filternamn</th><th width="159.800048828125" align="center">Transmissionsintervall (FWHM)</th><th align="center">Genomsnittlig transmission</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re-RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

När dessa formler används kan namnet sluta på ”\_1” eller ”\_2”, vilket motsvarar vilket NIR-filter – antingen NIR1 eller NIR2 – som användes.

För LATTICE M3C-kameror (Bayer trippelbandpass) använder samma indexmotor M3C-filterbanden:

| M3C-filter | Band 1 (centrum/FWHM) | Band 2 (centrum/FWHM) | Band 3 (centrum/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3M-kameror är enkelbandskameror (ett smalbandsfilter per kamera), så multibandsindex beräknas inte för en enskild M3M-bild. För att beräkna index med M3M ska du kombinera två eller flera kameror till en justerad multibandsstapel och använda LATTICE-indexmotorn (`chloros-cli lattice index` eller GUI:ns realtidsindexberäknare).

***

## Var varje indexnamn fungerar

I Chloros finns **tre** indexytor, och deras förinställda listor är inte identiska. Använd detta avsnitt för att kontrollera om ett namn fungerar där du planerar att använda det.

| Var du befinner dig | Vilken lista gäller | Antal |
| --- | --- | --- |
| Projektinställningar → Index → Lägg till index (GUI) | Yta 1 | 27 |
| Bildvisare [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Yta 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Yta 2 | 22 |
| SDK `process_folder(indices=[...])` | Yta 2 | 22 |
| `chloros-cli lattice index --preset` | Yta 3 | 22 (en annan 22) |
| Fliken Kameror – liveindexberäknare | Yta 3 | 22 (en annan 22) |

Surface 1 och 2 arbetar med **en bild i taget från en kamera**, med hjälp av symbolplatserna `x`/`y`/`z`(/`a`) kopplade till den kamerans filterkanaler. Yta 3 arbetar med en**justerad multibandsstapel** — flera LATTICE-kameror samregistrerade till en kub — och hänvisar till kanaler med namn i gemener.

### 1. GUI-projektinställningar / Bildvisarens sandlåda-rullgardinsmeny — 27 formler

Rullgardinsmenyn listar dem i följande ordning (det är i infogningsordning, inte alfabetisk ordning):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

I GUI drar du kamerans filterkanaler till formelns bandplatser, så att vilken formel som helst kan användas med vilken bandtilldelning som helst som din kamera stöder. Anpassade formler som du har sparat läggs till under denna lista.

De **fem formlerna som endast finns i GUI** — de som CLI-/SDK-listan `--indices` inte accepterar — är implementerade enligt följande:

| Förinställning endast för GUI | Formel (såsom implementerad) | Platser |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (fyra platser) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Den avsedda mappningen för varje formel anges i ett eget avsnitt längre ner på denna sida (till exempel förväntar sig GARI att x=Green, y=NIR, z=Blue, a=Red). GARI är den enda formeln i Chloros som använder en fjärde plats.

### 2. CLI / SDK `--indices` namnutvidgning — 22 förinställningar

Alternativet `chloros-cli process --indices` (och SDK-parametern `indices`) accepterar följande förinställningsnamn:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Okända indexnamn hoppas över utan meddelande.** Ett namn som inte finns med i denna lista (inklusive de fem formlerna som endast finns i GUI:n: `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` samt alla anpassade formler som du har sparat i GUI) utelämnas med endast en loggmeddelande – körningen fortsätter utan det indexet, och körningen i sig rapporteras fortfarande som lyckad. Meddelandet skrivs ut som:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Namn jämförs utan hänsyn till versaler och gemener efter att mellanslag har tagits bort, så `ndvi`, `NDVI` och ` NDVI ` är samma förinställning. En förinställning hoppas också över om den kräver ett band som kamerans filter inte tillhandahåller.
{% endhint %}

De exakta formlerna som de är implementerade (symbolerna `x`/`y`/`z` är bandplatser; standardmappningen visas per förinställning):

| Förinställning | Formel (såsom den är implementerad) | Standardfilter | Platser (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)*(x/y)` | RGN | Red, Green, NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Hur ett förinställt namn omvandlas till bandpositioner

När du anger ett rent namn som `NDVI` måste Chloros avgöra vilken kanal i vilken fil varje symbol avser. Programmet använder denna tabell, som kopplar en filterkod till arraypositionen för varje kanal:

| Filterkod | Kanal → arrayindex |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` accepteras som ett alias för Orange, även 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

Förinställningens **standardfilter** (kolumnen ”Standardfilter” ovan) används när projektet innehåller bilder med det filtret. Om så inte är fallet skannar Chloros igenom de filter som faktiskt finns i projektet i ordningen `RGN, OCN, NGB, RGB, RE, NIR` och väljer det första som kan tillhandahålla alla kanaler som förinställningen behöver. Om inget kan det görs, utelämnas förinställningen för den körningen. Det är därför som `NDVI`, som begärs för en datamängd som endast består av OCN, fortfarande ger ett meningsfullt resultat – det kopplas till OCN:s Orange- och NIR-positioner.

LATTICE M3C-modellsträngar innehåller filtret med prefixet `F` (`LATT-M3C-L41-FRGN`), men prefixet tas bort när filterkoden läses av från bilden, så en FRGN-kamera tolkar genom raden `RGN` ovan och behöver ingen särskild hantering.

### 3. LATTICE-indexmotor (`lattice index --preset`, live-indexberäknare) — 22 förinställningar

LATTICE-motorn arbetar med inriktade multibandsstaplar (live-matriser eller exporterade multibands-TIFF-filer) och använder kanalnamn med gemener (`red`, `green`, `blue`, `red_edge`, `nir`). Dess lista över förinställningar skiljer sig från de två ovanstående:

| Förinställning | Formel | Kanaler |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | röd, NIR |
| GNDVI | `(nir - green) / (nir + green)` | grönt, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | blått, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | rödkant, NIR |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | blå, grön, nära infraröd |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | röd, nära infraröd |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | röd, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | röd, nir |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | blå, röd, nir |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | röd, NIR |
| CVI | `nir * red / green**2` | röd, grön, NIR |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | röd, NIR |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | röd, NIR |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | blå, röd, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | röd, grön, blå |
| NGRDI | `(green - red) / (green + red)` | röd, grön |
| VARI | `(green - red) / (green + red - blue)` | röd, grön, blå |
| TGI | `green - 0.39*red - 0.61*blue` | röd, grön, blå |
| EXG | `2*green - red - blue` | röd, grön, blå |
| CIRE | `(nir / red_edge) - 1` | röd\_kant, NIR |
| CIGREEN | `(nir / green) - 1` | grön, NIR |
| NDWI | `(green - nir) / (green + nir)` | grön, nir |

Kör `chloros-cli lattice index --list-presets` för att skriva ut denna tabell från din installerade version, och `--list-gradients` för de tillgängliga färggradienterna. Kanalsymbolerna är skiftlägeskänsliga och måste stämma överens med förinställningarnas namn i gemener (t.ex. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI – Klorofyllvegetationsindex

Detta index är känsligt för bladens klorofyllhalt över ett brett spektrum av krontätheter. Det använder NIR-, röda och gröna band.

$$
CVI = NIR * {Red \over Green^{2}}
$$

I GUI och i CLI-/SDK-listan över förinställningar är symbolformeln `(z/y)*(x/y)` där x=Red, y=Green, z=NIR (banden hos en RGN-kamera); förinställningen i LATTICE-indexmotorn är `nir * red / green**2`. Samma index beräknas för varje yta.

_Referens: Vincini, M., E. Frazzi och P. D’Alessio. ”A Broad-Band Leaf Chlorophyll Vegetation Index at the Canopy Scale.” Precision Agriculture 9 (2008): 303–319._

***

## ENDVI – Förbättrat normaliserat vegetationsindex (Enhanced Normalized Difference Vegetation Index)

Detta index använder den blå kanalen utöver NIR och grönt, och är populärt för kameror med NGB-filter där det blå bandet ersätter det röda.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Implementeringen sker med symbolformeln `((x+y)-(2*z))/((x+y)+(2*z))` – tilldela kamerans NIR- och Green-kanaler till x/y-platserna och Blue till z (för en NGB-kamera: x=NIR, y=Green, z=Blue).

***

## EVI – Enhanced Vegetation Index

Detta index utvecklades ursprungligen för användning med MODIS-data som en förbättring jämfört med NDVI genom att optimera vegetationssignalen i områden med högt bladarealindex (LAI). Det är mest användbart i regioner med högt LAI där NDVI kan mättas. Det använder det blåa reflektansområdet för att korrigera för bakgrundssignaler från marken och för att minska atmosfäriska påverkan, inklusive aerosolspridning.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI-värdena bör ligga mellan 0 och 1 för vegetationspixlar. Ljusa objekt såsom moln och vita byggnader, liksom mörka objekt såsom vatten, kan ge upphov till avvikande pixelvärden i en EVI-bild. Innan du skapar en EVI-bild bör du maskera bort moln och ljusa objekt från reflektansbilden och, om så önskas, sätta ett tröskelvärde för pixelvärdena mellan 0 och 1.

_Referens: Huete, A., et al. ”Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.” Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Skogstäckningsindex 1

_Endast GUI – finns inte som CLI- eller SDK-förinställning `--indices`._

Detta index skiljer skogskronor från andra typer av vegetation med hjälp av multispektrala reflektansbilder som inkluderar ett rött kantband.

$$
FCI1 = Red * RedEdge
$$

Skogsområden kommer att ha lägre FCI1-värden på grund av trädens lägre reflektans och förekomsten av skuggor inom trädkronorna.

_Referens: Becker, Sarah J., Craig S.T. Daughtry och Andrew L. Russ. ”Robust forest cover indices for multispectral images.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Skogstäckningsindex 2

_Endast GUI – finns inte som CLI- eller SDK-förinställning `--indices`._

Detta index skiljer skogskronor från andra typer av vegetation med hjälp av multispektrala reflektansbilder som inte innehåller ett ”red edge”-band.

$$
FCI2 = Red * NIR
$$

Skogsområden kommer att ha lägre FCI2-värden på grund av trädens lägre reflektans och förekomsten av skuggor inom trädkronorna.

_Referens: Becker, Sarah J., Craig S.T. Daughtry och Andrew L. Russ. ”Robust forest cover indices for multispectral images.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Global Environmental Monitoring Index

_Endast GUI – finns inte som CLI- eller SDK-förinställning `--indices`._

Detta icke-linjära vegetationsindex används för global miljöövervakning utifrån satellitbilder och syftar till att korrigera för atmosfäriska effekter. Det liknar NDVI, men är mindre känsligt för atmosfäriska effekter. Det påverkas av bar mark; därför rekommenderas det inte för användning i områden med gles eller måttligt tät vegetation.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Där:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referens: Pinty, B., och M. Verstraete. ”GEMI: a Non-Linear Index to Monitor Global Vegetation From Satellites”. Vegetation 101 (1992): 15–20._

***

## GARI – Green Atmospherically Resistant Index

_Endast GUI – finns inte som förinställning för CLI/SDK `--indices`._

Detta index är känsligare för ett brett spektrum av klorofyllkoncentrationer och mindre känsligt för atmosfäriska effekter än NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gammakonstanten är en viktningsfunktion som beror på aerosolförhållandena i atmosfären. ENVI använder värdet 1,7, vilket är det rekommenderade värdet enligt Gitelson, Kaufman och Merzylak (1996, sidan 296).

_Referens: Gitelson, A., Y. Kaufman och M. Merzylak. ”Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.” Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green Chlorophyll Index

Detta index används för att uppskatta klorofyllhalten i blad hos ett brett spektrum av växtarter.

$$
GCI = {NIR \over Green} - 1
$$

Att ha breda våglängder inom NIR- och det gröna spektrumet ger en bättre förutsägelse av klorofyllhalten samtidigt som det möjliggör högre känslighet och ett bättre signal-brusförhållande.

_Referens: Gitelson, A., Y. Gritz och M. Merzlyak. ”Relationships Between Leaf Chlorophyll Content and Spectral Reflectance and Algorithms for Non-Destructive Chlorophyll Assessment in Higher Plant Leaves.” Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Leaf Index

Detta index utformades ursprungligen för användning med en digital RGB-kamera för att mäta vetehäckning, där de röda, gröna och blå digitala värdena (DN) varierar mellan 0 och 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI-värdena varierar mellan -1 och +1. Negativa värden representerar mark och icke-levande föremål, medan positiva värden representerar gröna blad och stjälkar.

_Referens: Louhaichi, M., M. Borman och D. Johnson. ”Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.” Geocarto International 16, nr 1 (2001): 65–70._

***

## GNDVI – Green Normalized Difference Vegetation Index

Detta index liknar NDVI, med den skillnaden att det mäter det gröna spektrumet från 540 till 570 nm istället för det röda spektrumet. Detta index är mer känsligt för klorofyllkoncentrationen än NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referens: Gitelson, A., och M. Merzlyak. ”Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.” Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimized Soil Adjusted Vegetation Index

Detta index utformades ursprungligen med hjälp av färg-infrarödfotografering för att förutsäga kvävebehovet hos majs. Det liknar OSAVI, men ersätter det röda bandet med det gröna.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referens: Sripada, R., m.fl. ”Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.” Doktorsavhandling, North Carolina State University, 2005._

***

## GRVI – Green Ratio Vegetation Index

Detta index är känsligt för fotosynteshastigheten i skogskronorna, eftersom reflektanserna för grönt och rött påverkas starkt av förändringar i bladpigmenten.

$$
GRVI = {NIR \over Green }
$$

_Referens: Sripada, R., et al. ”Flygfotografering i färg och infrarött för att fastställa kvävebehovet tidigt under säsongen hos majs.” Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green Soil Adjusted Vegetation Index

Detta index utformades ursprungligen med hjälp av färg-infrarödfotografering för att förutsäga kvävebehovet hos majs. Det liknar SAVI, men ersätter det röda bandet med det gröna.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referens: Sripada, R., m.fl. ”Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.” Doktorsavhandling, North Carolina State University, 2005._

***

## LAI – bladarealindex

Detta index används för att uppskatta lövverkstäckningen och för att prognostisera grödans tillväxt och avkastning. ENVI beräknar grönt LAI med hjälp av följande empiriska formel från Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Där EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Höga LAI-värden ligger vanligtvis mellan cirka 0 och 3,5. När scenen innehåller moln och andra ljusa inslag som ger upphov till mättade pixlar kan LAI-värdena dock överstiga 3,5. Helst bör du maskera bort moln och ljusa inslag från scenen innan du skapar en LAI-bild.

_Referens: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde och A. Thomsen. ”Flygburna multispektrala data för kvantifiering av bladareaindex, kvävekoncentration och fotosyntetisk effektivitet inom jordbruket.” Remote Sensing of Environment 81, nr 2–3 (2002): 179–193._

***

## LCI – bladklorofyllindex

_Endast GUI – finns inte som CLI/SDK-förinställning i `--indices`._

Detta index används för att uppskatta klorofyllhalten i högre växter och är känsligt för variationer i reflektans som orsakas av klorofyllabsorption.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referens: Datt, B. ”Remote Sensing of Water Content in Eucalyptus Leaves.” Journal of Plant Physiology 154, nr 1 (1999): 30–36._

***

## MNLI – Modifierat icke-linjärt index

Detta index är en vidareutveckling av det icke-linjära indexet (NLI) som införlivar det jordjusterade vegetationsindexet (SAVI) för att ta hänsyn till markbakgrunden. ENVI använder ett värde på 0,5 för justeringsfaktorn för trädkronans bakgrund (_L_).

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referens: Yang, Z., P. Willis och R. Mueller. ”Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.” Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modified Soil Adjusted Vegetation Index 2

Detta index är en enklare version av MSAVI-indexet som föreslagits av Qi m.fl. (1994), vilket utgör en förbättring av Soil Adjusted Vegetation Index (SAVI). Det minskar markbruset och ökar det dynamiska omfånget för vegetationssignalen. MSAVI2 baseras på en induktiv metod som inte använder ett konstant _L_-värde (som hos SAVI) för att framhäva frisk vegetation.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referens: Qi, J., A. Chehbouni, A. Huete, Y. Kerr och S. Sorooshian. ”A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – Modified Simple Ratio

Detta index är en modifiering av det enkla NIR/Red-förhållandet, utformat för att linjärisera dess samband med biofysiska parametrar, och är känsligare än NDVI vid högre vegetationstäthet.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Referens: Chen, J. ”Evaluation of Vegetation Indices and a Modified Simple Ratio for Boreal Applications.” Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – Normalized Difference RedEdge

Detta index liknar NDVI, men jämför kontrasten mellan NIR och RedEdge istället för Red, vilket ofta upptäcker vegetationsstress tidigare.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalized Difference Vegetation Index

Detta index är ett mått på frisk, grön vegetation. Kombinationen av dess normaliserade differensformel och användningen av klorofyllens områden med högst absorption och reflektans gör det robust under ett brett spektrum av förhållanden. Det kan dock mättas vid tät vegetation när LAI-värdet blir högt.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Värdet för detta index varierar mellan -1 och 1. Det vanliga intervallet för grön vegetation är 0,2 till 0,8.

_Referens: Rouse, J., R. Haas, J. Schell och D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Tredje ERTS-symposiet, NASA (1973): 309–317._

***

## NLI – Icke-linjärt index

Detta index utgår från att sambandet mellan många vegetationsindex och biofysiska ytparametrar är icke-linjärt. Det lineariserar samband med ytparametrar som tenderar att vara icke-linjära.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referens: Goel, N., och W. Qin. ”Influences of Canopy Architecture on Relationships Between Various Vegetation Indices and LAI and Fpar: A Computer Simulation.” Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimerat jordjusterat vegetationsindex

Detta index baseras på det jordjusterade vegetationsindexet (SAVI). Det använder ett standardvärde på 0,16 för justeringsfaktorn för trädkronans bakgrund. Rondeaux (1996) fastställde att detta värde ger större variation i jordförhållandena än SAVI vid låg vegetationstäckning, samtidigt som det uppvisar ökad känslighet vid en vegetationstäckning på över 50 %. Detta index används bäst i områden med relativt gles vegetation där marken syns genom trädkronorna.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referens: Rondeaux, G., M. Steven och F. Baret. ”Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormaliserat differensvegetationsindex

Detta index använder skillnaden mellan våglängder i det nära infraröda och röda spektrumet, tillsammans med NDVI, för att framhäva frisk vegetation. Det är okänsligt för effekterna av marken och solens betraktningsvinkel.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referens: Roujean, J., och F. Breon. ”Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.” Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – jordjusterat vegetationsindex

Detta index liknar NDVI, men det dämpar effekterna av jordpixlar. Det använder en justeringsfaktor för trädkronans bakgrund, _L_, som är en funktion av vegetationstätheten och ofta kräver förkunskaper om vegetationsmängderna. Huete (1988) föreslår ett optimalt värde på _L_=0,5 för att ta hänsyn till variationer i markbakgrunden av första ordningen. Detta index används bäst i områden med relativt gles vegetation där marken syns genom trädkronorna.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referens: Huete, A. ”A Soil-Adjusted Vegetation Index (SAVI).” Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – Transformed Difference Vegetation Index

Detta index är användbart för övervakning av vegetationstäcket i stadsmiljöer. Det mättas inte på samma sätt som NDVI och SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referens: Bannari, A., H. Asalhi och P. Teillet. ”Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” I Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, volym 5 (2002)._

***

## VARI – Visible Atmospherically Resistant Index

Detta index baseras på ARVI och används för att uppskatta andelen vegetation i en bild med låg känslighet för atmosfäriska effekter.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referens: Gitelson, A., et al. ”Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.” International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Wide Dynamic Range Vegetation Index

Detta index liknar NDVI, men använder en viktningskoefficient (_a_) för att minska skillnaden mellan bidragen från signalerna i det nära infraröda och röda spektrumet till NDVI. WDRVI är särskilt effektivt i scener med måttlig till hög vegetationstäthet när NDVI överstiger 0,6. NDVI tenderar att plana ut när vegetationsandelen och bladareaindexet (LAI) ökar, medan WDRVI är mer känsligt för ett bredare spektrum av vegetationsandelar och för förändringar i LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Viktningskoefficienten (_a_) kan variera mellan 0,1 och 0,2. Ett värde på 0,2 rekommenderas av Henebry, Viña och Gitelson (2004).

_Referenser_

_Gitelson, A. ”Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.” Journal of Plant Physiology 161, nr 2 (2004): 165–173._

_Henebry, G., A. Viña och A. Gitelson. ”The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis.” Gap Analysis Bulletin 12: 50–56._
