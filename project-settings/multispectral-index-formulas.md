---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formler för multispektrala index

Nedanstående indexformler använder en kombination av genomsnittliga transmittansintervall för filtret Survey3:

<table><thead><tr><th align="center">Survey3-filterfärg</th><th width="196.199951171875" align="center">Survey3 Filternamn</th><th width="159.800048828125" align="center">Transmissionsintervall (FWHM)</th><th align="center">Genomsnittlig transmission</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>När dessa formler används kan namnet sluta på &quot;\_1&quot; eller &quot;\_2&quot;, vilket motsvarar vilket NIR-filter, antingen NIR1 eller NIR2, som användes.

***

## EVI – Förbättrat vegetationsindex

Detta index utvecklades ursprungligen för användning med MODIS-data som en förbättring jämfört med NDVI genom att optimera vegetationssignalen i områden med högt bladarealindex (LAI). Det är mest användbart i regioner med högt LAI där NDVI kan mättas. Det använder det blåa reflektansområdet för att korrigera för bakgrundssignaler från marken och för att minska atmosfäriska influenser, inklusive aerosolspridning.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI-värden bör ligga mellan 0 och 1 för vegetationspixlar. Ljusa objekt såsom moln och vita byggnader, tillsammans med mörka objekt såsom vatten, kan resultera i onormala pixelvärden i en EVI-bild. Innan du skapar en EVI-bild bör du maskera bort moln och ljusa objekt från reflektansbilden, och eventuellt sätta en tröskel för pixelvärdena från 0 till 1.

_Referens: Huete, A., et al. &quot;Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.&quot; Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Skogstäckningsindex 1

Detta index skiljer skogskronor från andra typer av vegetation med hjälp av multispektrala reflektansbilder som inkluderar ett rött kantband.

$$
FCI1 = Red * RedEdge
$$

Skogsområden kommer att ha lägre FCI1-värden på grund av trädens lägre reflektans och förekomsten av skuggor inom skogskronorna.

_Referens: Becker, Sarah J., Craig S.T. Daughtry och Andrew L. Russ. &quot;Robust forest cover indices for multispectral images.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 – Skogstäckningsindex 2

Detta index skiljer skogskronor från andra typer av vegetation med hjälp av multispektrala reflektansbilder som inte inkluderar ett rött kantband.

$$
FCI2 = Red * NIR
$$

Skogsområden kommer att ha lägre FCI2-värden på grund av trädens lägre reflektans och förekomsten av skuggor i trädkronorna.

_Referens: Becker, Sarah J., Craig S.T. Daughtry och Andrew L. Russ. &quot;Robust forest cover indices for multispectral images.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI – Globalt miljöövervakningsindex

Detta icke-linjära vegetationsindex används för global miljöövervakning utifrån satellitbilder och försöker korrigera för atmosfäriska effekter. Det liknar NDVI men är mindre känsligt för atmosfäriska effekter. Det påverkas av bar mark; därför rekommenderas det inte för användning i områden med gles eller måttligt tät vegetation.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Där:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referens: Pinty, B., och M. Verstraete. GEMI: ett icke-linjärt index för övervakning av global vegetation från satelliter. Vegetation 101 (1992): 15-20._

***

## GARI – Green Atmosfäriskt resistent index

Detta index är mer känsligt för ett brett spektrum av klorofyllkoncentrationer och mindre känsligt för atmosfäriska effekter än NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gammakonstanten är en viktningsfunktion som beror på aerosolförhållandena i atmosfären. ENVI använder värdet 1,7, vilket är det rekommenderade värdet enligt Gitelson, Kaufman och Merzylak (1996, sidan 296).

_Referens: Gitelson, A., Y. Kaufman och M. Merzylak. &quot;Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI – Green Klorofyllindex

Detta index används för att uppskatta klorofyllhalten i blad hos ett brett spektrum av växtarter.

$$
GCI = {NIR \over Green} - 1
$$

Att ha breda NIR och gröna våglängder ger en bättre förutsägelse av klorofyllhalten samtidigt som det möjliggör högre känslighet och ett högre signal-brusförhållande.

_Referens: Gitelson, A., Y. Gritz och M. Merzlyak. ”Relationships Between Leaf Chlorophyll Content and Spectral Reflectance and Algorithms for Non-Destructive Chlorophyll Assessment in Higher Plant Leaves.” Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Bladindex

Detta index utformades ursprungligen för användning med en digital RGB-kamera för att mäta vetehäckning, där de röda, gröna och blå digitala siffrorna (DN) varierar mellan 0 och 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI-värdena varierar mellan -1 och +1. Negativa värden representerar mark och icke-levande objekt, medan positiva värden representerar gröna blad och stjälkar.

_Referens: Louhaichi, M., M. Borman och D. Johnson. &quot;Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.&quot; Geocarto International 16, nr 1 (2001): 65-70._

***

## GNDVI – Green Normaliserat vegetationsindex

Detta index liknar NDVI, förutom att det mäter det gröna spektrumet från 540 till 570 nm istället för det röda spektrumet. Detta index är mer känsligt för klorofyllkoncentrationen än NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referens: Gitelson, A., och M. Merzlyak. &quot;Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.&quot; Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Optimerat jordjusterat vegetationsindex

Detta index utformades ursprungligen med hjälp av färg-infraröd fotografering för att förutsäga kvävebehovet för majs. Det liknar OSAVI, men ersätter det gröna bandet med det röda.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referens: Sripada, R., et al. &quot;Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.&quot; Doktorsavhandling, North Carolina State University, 2005._

***

## GRVI - Green Vegetationsindex baserat på förhållandet

Detta index är känsligt för fotosynteshastigheten i skogskronorna, eftersom gröna och röda reflektansvärden påverkas starkt av förändringar i bladpigmenten.

$$
GRVI = {NIR \over Green }
$$

_Referens: Sripada, R., et al. &quot;Aerial Color Infrared Photography for Determining Early In-season Nitrogen Requirements in Corn.&quot; Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Jordjusterat vegetationsindex

Detta index utformades ursprungligen med hjälp av färg-infraröd fotografering för att förutsäga kvävebehovet för majs. Det liknar SAVI, men ersätter det gröna bandet med det röda.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referens: Sripada, R., et al. &quot;Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.&quot; Doktorsavhandling, North Carolina State University, 2005._

***

## LAI – Bladarealindex

Detta index används för att uppskatta lövverkstäckningen och för att prognostisera grödans tillväxt och avkastning. ENVI beräknar grönt LAI med hjälp av följande empiriska formel från Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Där EVI är:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Höga LAI-värden ligger vanligtvis mellan cirka 0 och 3,5. Men när scenen innehåller moln och andra ljusa objekt som ger upphov till mättade pixlar kan LAI-värdena överstiga 3,5. Helst bör du maskera bort moln och ljusa objekt från scenen innan du skapar en LAI-bild.

_Referens: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde och A. Thomsen. ”Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture.” Remote Sensing of Environment 81, nr 2-3 (2002): 179-193._

***

## LCI – Bladklorofyllindex

Detta index används för att uppskatta klorofyllhalten i högre växter och är känsligt för variationer i reflektans orsakade av klorofyllabsorption.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referens: Datt, B. &quot;Fjärranalys av vatteninnehåll i eukalyptusblad.&quot; Journal of Plant Physiology 154, nr 1 (1999): 30-36._

***

## MNLI – Modifierat icke-linjärt index

Detta index är en förbättring av det icke-linjära indexet (NLI) som införlivar det jordjusterade vegetationsindexet (SAVI) för att ta hänsyn till jordbakgrunden. ENVI använder ett värde på 0,5 för justeringsfaktorn för trädkronans bakgrund (_L_).

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referens: Yang, Z., P. Willis och R. Mueller. &quot;Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.&quot; Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modifierat jordjusterat vegetationsindex 2

Detta index är en enklare version av indexet MSAVI som föreslagits av Qi et al (1994), vilket förbättrar det jordjusterade vegetationsindexet (SAVI). Det minskar jordbruset och ökar det dynamiska omfånget för vegetationssignalen. MSAVI2 baseras på en induktiv metod som inte använder ett konstant _L_-värde (som i SAVI) för att framhäva frisk vegetation.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referens: Qi, J., A. Chehbouni, A. Huete, Y. Kerr och S. Sorooshian. &quot;A Modified Soil Adjusted Vegetation Index.&quot; Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE – Normaliserad skillnad RedEdge

Detta index liknar NDVI men jämför kontrasten mellan NIR och RedEdge istället för Red, vilket ofta upptäcker vegetationsstress tidigare.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normaliserat vegetationsindex

Detta index är ett mått på frisk, grön vegetation. Kombinationen av dess normaliserade differensformulering och användningen av klorofyllens områden med högst absorption och reflektans gör det robust under ett brett spektrum av förhållanden. Det kan dock mättas i förhållanden med tät vegetation när LAI blir högt.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Värdet på detta index varierar mellan -1 och 1. Det vanliga intervallet för grön vegetation är 0,2 till 0,8.

_Referens: Rouse, J., R. Haas, J. Schell och D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Tredje ERTS-symposiet, NASA (1973): 309–317._

***

## NLI – Icke-linjärt index

Detta index utgår från att förhållandet mellan många vegetationsindex och ytans biofysiska parametrar är icke-linjärt. Det lineariserar förhållanden med ytparametrar som tenderar att vara icke-linjära.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referens: Goel, N. och W. Qin. &quot;Influences of Canopy Architecture on Relationships Between Various Vegetation Indices and LAI and Fpar: A Computer Simulation.&quot; Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI – Optimerat markjusterat vegetationsindex

Detta index baseras på det markjusterade vegetationsindexet (SAVI). Det använder ett standardvärde på 0,16 för justeringsfaktorn för trädkronans bakgrund. Rondeaux (1996) fastställde att detta värde ger större variation i marken än SAVI vid låg vegetationstäckning, samtidigt som det visar ökad känslighet för vegetationstäckning över 50 %. Detta index används bäst i områden med relativt gles vegetation där marken syns genom trädkronorna.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referens: Rondeaux, G., M. Steven och F. Baret. &quot;Optimization of Soil-Adjusted Vegetation Indices.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI – Renormalized Difference Vegetation Index

Detta index använder skillnaden mellan våglängder i det nära infraröda och röda spektrumet, tillsammans med NDVI, för att markera frisk vegetation. Det är okänsligt för effekterna av marken och solens betraktningsvinkel.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referens: Roujean, J., och F. Breon. &quot;Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.&quot; Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI – Jordjusterat vegetationsindex

Detta index liknar NDVI, men det dämpar effekterna av markpixlar. Det använder en justeringsfaktor för trädkronans bakgrund, _L_, som är en funktion av vegetationstätheten och ofta kräver förkunskaper om vegetationsmängder. Huete (1988) föreslår ett optimalt värde på _L_=0,5 för att ta hänsyn till variationer i markbakgrunden av första ordningen. Detta index används bäst i områden med relativt gles vegetation där marken syns genom trädkronorna.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referens: Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – Transformed Difference Vegetation Index

Detta index är användbart för övervakning av vegetationstäcket i stadsmiljöer. Det mättas inte på samma sätt som NDVI och SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referens: Bannari, A., H. Asalhi och P. Teillet. ”Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” I Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, volym 5 (2002)._

***

## VARI – Synligt atmosfärsresistent index

Detta index baseras på ARVI och används för att uppskatta andelen vegetation i en scen med låg känslighet för atmosfäriska effekter.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referens: Gitelson, A., et al. &quot;Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Vegetationsindex med brett dynamiskt omfång

Detta index liknar NDVI, men använder en viktningskoefficient (_a_) för att minska skillnaden mellan bidraget från signalerna i det nära infraröda och röda spektrumet till NDVI. WDRVI är särskilt effektivt i scener med måttlig till hög vegetationstäthet när NDVI överstiger 0,6. NDVI tenderar att plana ut när vegetationsfraktionen och bladarealindexet (LAI) ökar, medan WDRVI är mer känsligt för ett bredare intervall av vegetationsfraktioner och för förändringar i LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Viktningskoefficienten (_a_) kan variera mellan 0,1 och 0,2. Ett värde på 0,2 rekommenderas av Henebry, Viña och Gitelson (2004).

_Referenser_

_Gitelson, A. &quot;Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.&quot; Journal of Plant Physiology 161, nr 2 (2004): 165–173._

_Henebry, G., A. Viña och A. Gitelson. ”The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis.” Gap Analysis Bulletin 12: 50–56._
