# Bildlager

I rullgardinsmenyn Bildlager i Chloros Image Viewer kan du snabbt växla mellan olika versioner av samma bild – från originalbilderna till bearbetade reflektansutdata och beräknade indexbilder.

## Vad är bildlager?

I Chloros avser **lager** de olika bildutdata som finns tillgängliga för en enskild källbild. När du bearbetar bilder skapar Chloros flera versioner:

* **Originalbilder** (JPG- och RAW-filer från din kamera)
* **Reflektanskalibrerade** utdata (om reflektanskalibrering var aktiverad)
* **Målbilder** (om bilden innehåller kalibreringsmål)
* **Indexbilder** (NDVI, NDRE, GNDVI, etc. om index var konfigurerade)

Med **rullgardinsmenyn för lagerval** längst upp till höger i bildvisaren kan du omedelbart växla mellan dessa versioner utan att lämna visaren.***

## Tillgängliga lagertyper

### JPG

* Den ursprungliga JPG-förhandsvisningsbilden från din kamera
* Alltid tillgänglig för alla bilder
* Obearbetad, precis som den togs av kameran
* Snabbast att ladda och visa

**När ska du visa:**

* Snabb förhandsgranskning av originalbilden
* Kontrollera bildkomposition och bildutsnitt
* Verifiera bildkvaliteten före bearbetning

### RAW (Original)

* De ursprungliga RAW-sensordata från din kamera
* Debayered utan efterbearbetning
* Högre bitdjup än JPG (vanligtvis 12-bitars eller 14-bitars sensordata)

**När ska man visa:**

* Kontrollera kvaliteten på den ursprungliga sensordatan
* Kontrollera om det finns sensorproblem eller artefakter
* Jämföra resultat före och efter bearbetning

### RAW (Mål)

* Visas endast för bilder som identifierats som innehållande kalibreringsmål
* Visar den ursprungliga RAW-bilden med detekterat mål
* Används för att verifiera att måldetekteringen lyckades

**När ska man visa:**

* Bekräfta att kalibreringsmålen detekterades korrekt
* Kontrollera målbildens kvalitet
* Felsöka kalibreringsproblem

{% hint style="info" %}
**Målskikt**: Detta skikt visas endast i rullgardinsmenyn för bilder som innehåller kalibreringsmål. Vanliga bilder har inte detta alternativ.
{% endhint %}

### RAW (reflektans)

* Den kalibrerade reflektansutgångsbilden
* Vignettkorrigerad (om aktiverat vid bearbetning)
* Reflektans kalibrerad med hjälp av måldata (om aktiverat)
* Multibands TIFF med alla kamerakanaler
* Pixelvärden representerar procentuell reflektans (vid användning av procentläge)
* Redo att bearbetas med [Index/LUT Sandbox](index-lut-sandbox.md)

**När ska man visa:**

* Granska kalibrerade resultat
* Verifiera kalibreringskvaliteten
* Kontrollera pixelvärden för vetenskaplig noggrannhet
* Jämföra med originalet för att se kalibreringseffekter

{% hint style="success" %}
**Rekommenderat**: Använd RAW (reflektans)-lager när du kontrollerar pixelvärden för vetenskapliga mätningar och analyser.
{% endhint %}

### RAW (NDVI-index)... och liknande

* Beräknad bild av vegetationsindex (NDVI i detta exempel)
* Indexnamnet ändras beroende på vilket index som konfigurerades under bearbetningen
* Exempel: RAW (NDVI-index), RAW (NDRE-index), RAW (GNDVI-index) osv.
* Enbandsgråskalebild som visar indexberäkningsresultaten
* Ett lager visas för varje index som konfigurerats i projektinställningarna

**Möjliga indexnamn:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI-index)
* RAW (EVI-index)
* RAW (SAVI-index)
* Och många fler... (se [Formler för multispektrala index](../project-settings/multispectral-index-formulas.md))

**När ska man visa:**

* Granska resultat av indexberäkningar
* Kontrollera indexvärdesintervall
* Identifiera områden av intresse
* Verifiera indexbilder innan de används i GIS eller analys

***

## Använda lagerväljaren

### Öppna rullgardinsmenyn

1. Öppna en bild i helskärmsläge (klicka på valfri miniatyrbild i bildvisaren)
2. Leta reda på **lagermenyn** i det övre högra hörnet av visaren
3. Menyn visar det lager som för närvarande är valt (t.ex. &quot;JPG&quot;)
4. Klicka på menyn för att se alla tillgängliga lager

### Byta lager

1. Klicka på lagermenyn för att öppna listan
2. Alla tillgängliga lager för den aktuella bilden visas
3. Klicka på valfritt lagernamn för att byta till den versionen
4. Bilden uppdateras omedelbart för att visa det valda lagret

**Snabbt byte:**

* Rullgardinsmenyn kommer ihåg ditt senaste val
* När du navigerar till nästa bild försöker Chloros visa samma lagertyp
* Om det lagret inte finns på nästa bild används JPG som standard

### Lagertillgänglighet

Alla lager är inte tillgängliga för varje bild:

**Alltid tillgängliga:*** ✅ JPG (varje bild har en JPG-förhandsgranskning)

**Villkorligt tillgängliga:**

* ⚠️ RAW (Original) – Endast om bilden togs i RAW- eller RAW+JPG-läge
* ⚠️ RAW (Mål) – Endast om bilden innehåller upptäckta kalibreringsmål
* ⚠️ RAW (reflektans) – Endast efter bearbetning med reflektanskalibrering aktiverad
* ⚠️ RAW (\[Index] Index) – Endast efter bearbetning med konfigurerade index

***

## Lagerbeständighet

### Navigera mellan bilder

När du navigerar till en annan bild (med piltangenterna eller genom att klicka på miniatyrer):**Lagerinställningen bevaras:**

* Om du visar &quot;RAW (Reflektans)&quot; visas nästa bild som &quot;RAW (Reflektans)&quot; (om tillgängligt)
* Om du visar &quot;RAW (NDVI Index)&quot;, visas nästa bild som &quot;RAW (NDVI Index)&quot; (om tillgängligt)
* Om samma lager inte finns, används JPG som standard

**Exempel på arbetsflöde:**

1. Öppna bild 1, växla till RAW (NDVI Index)
2. Tryck på → för att visa bild 2
3. Bild 2 visar automatiskt lagret RAW (NDVI Index)
4. Fortsätt navigera – alla bilder visar NDVI-lagret
5. Mycket effektivt för att granska indexresultat över många bilder

***

## Vanliga arbetsflöden

### Arbetsflöde 1: Före/efter-jämförelse

**Mål**: Jämför originalbilden med den kalibrerade bilden

1. Öppna den bearbetade bilden i bildvisaren
2. Välj **RAW (Original)** från rullgardinsmenyn
3. Notera vinjetteringen och de okalibrerade värdena
4. Byt till **RAW (Reflectance)** från rullgardinsmenyn
5. Jämför – vinjettering borttagen, värden kalibrerade

### Arbetsflöde 2: Granskning av index

**Mål**: Granska snabbt NDVI-resultat över hela datasetet

1. Öppna den första bearbetade bilden
2. Välj **RAW (NDVI Index)** i rullgardinsmenyn
3. Använd →-piltangenten för att navigera till nästa bild
4. NDVI-lagret kvarstår automatiskt
5. Fortsätt genom alla bilder och kontrollera NDVI-mönstren
6. Byt till **RAW (NDRE Index)** för att jämföra

### Arbetsflöde 3: Verifiering av mål

**Mål**: Verifiera att alla målbilder har detekterats korrekt

1. Navigera till en målbild
2. Välj **RAW (Target)** från rullgardinsmenyn
3. Verifiera att kalibreringsmålen är tydligt synliga och detekterade
4. Navigera till nästa målbild
5. Upprepa verifieringen för alla mål

### Arbetsflöde 4: Inspektion av pixelvärden

**Mål**: Kontrollera reflektansvärden för vetenskaplig noggrannhet

1. Öppna den bearbetade bilden
2. Välj **RAW (Reflectance)**-lagret
3. Aktivera läget **Pixel Percent** (knappen i verktygsfältet uppe till höger)
4. Flytta markören över vegetationsområden
5. Kontrollera att pixelvärdena ligger inom förväntade intervall (30–70 % för NIR, 5–15 % för Red)
6. Kontrollera att mark- och vattenområdena har lämpliga värden

***

## Förstå pixelvärden per lager

Olika lager visar olika intervall för pixelvärden:

### JPG-lager

* **Intervall**: 0–255 (8-bitars)
* **Betydelse**: Visningsvärden, gammakorrigerade
* **Användning**: Endast visuell inspektion, inte för vetenskapliga mätningar

### RAW (Original)

* **Intervall**: 0–65535 (16-bitars)
* **Betydelse**: Råa digitala siffror från sensorn
* **Användning**: Kontroll av sensorns prestanda, inte kalibrerade

### RAW (reflektans)

* **Intervall**: 0–65 535 (16-bitars TIFF) eller 0,0–1,0 (32-bitars procent)
* **Betydelse**: Kalibrerad reflektans i procent
* **Användning**: Vetenskapliga mätningar och analyser**För 16-bitars TIFF:**Dela med 65 535 för att få reflektansen i procent**För 32-bitars procent:** Värdena representerar direkt procent (0,5 = 50 % reflektans)

### RAW (indexbilder)

* **Intervall**: Varierar beroende på index (vanligtvis -1,0 till +1,0 för normaliserade index)
* **Betydelse**: Resultat av indexberäkning
* **Exempel**:
  * NDVI: -1 till +1 (vegetation vanligtvis 0,4 till 0,9)
  * NDRE: -1 till +1 (stressdetektering)
  * EVI: 0 till 1 (förbättrad vegetation)

***

## Tips och bästa praxis

### Effektiv växling mellan lager

* **Tangentbordsgenvägar**: Det finns inga tangentbordsgenvägar för lager, men navigationspilarna (←/→) fungerar för alla lager
* **Konsekventa arbetsflöden**: Välj ett lager (t.ex. NDVI) och granska hela datasetet innan du byter till ett annat
* **Snabba jämförelser**: Växla mellan Original och Reflektans för att verifiera bearbetningskvaliteten

### Prestandaöverväganden

* **JPG laddas snabbast**: Använd för snabb navigering genom många bilder
* **RAW-lager laddas långsammare**: Högre upplösning och bitdjup
* **Indexlager**: Liknande hastighet som Reflektans-lager
* **Första laddningen är långsam**: Efterföljande visningar av samma lager cachas och går snabbare

### Kvalitetskontroll

* **Kontrollera alltid RAW (Original)**: Kontrollera källdatakvaliteten innan du litar på de bearbetade resultaten
* **Jämför lager**: Använd lagerväxling för att verifiera att bearbetningen fungerade korrekt
* **Kontrollera indexintervall**: Använd läget Pixelprocent med indexlager för att verifiera att värdena är rimliga***

## Felsökning

### Lager inte tillgängligt

**Problem**: Förväntat lager visas inte i rullgardinsmenyn**Möjliga orsaker:**

* Bilden har inte bearbetats (endast JPG och RAW (Original) tillgängliga)
* Reflektanskalibrering var inaktiverad under bearbetningen
* Specifikt index var inte konfigurerat i projektinställningarna
* Bilden är en ren målbild (inga index genereras för mål)

**Lösningar:**

1. Kontrollera att bilden har bearbetats (kontrollera utdatamappen för bearbetade filer)
2. Kontrollera projektinställningarna för att bekräfta att index har konfigurerats
3. Bearbeta om med önskade index aktiverade

### Fel lager visas

**Problem**: Bilden öppnas i ett oväntat lager**Orsak**: Lagerinställningen från föregående bild har överförts, men det lagret finns inte i den aktuella bilden**Lösning**: Chloros faller automatiskt tillbaka till JPG när det önskade lagret inte är tillgängligt – detta är normalt beteende

### Kan inte se kalibreringsmål

**Problem**: RAW-lagret (mål) visar inte måldetektering**Möjliga orsaker:**

* Mål detekterades inte under bearbetningen
* Bilden innehåller faktiskt inga mål
* Inställningarna för måldetektering är för strikta

**Lösningar:**

1. Kontrollera felsökningsloggen efter meddelanden om &quot;Mål hittat&quot;
2. Kontrollera att bilden faktiskt innehåller synliga kalibreringsmål
3. Justera inställningarna för måldetektering i projektinställningarna
4. Se [Välja målbilder](../processing-images-gui/choosing-target-images.md)

***

## Relaterade funktioner

### Verktyg i bildvisaren

När du visar ett lager kan du använda:

* **Zoomkontroller**: Förstora för att granska detaljer
* **Panorera**: Klicka och dra för att flytta runt i den zoomade bilden
* **Pixelvärdesgranskning**: Se värden vid markörens position
* **Navigeringspilar**: Flytta mellan bilder medan du behåller lagret
* **Pixelprocentläge**: Växla mellan DN- och procentvisning

Se [Öppna en bild i helskärmsläge](opening-an-image-full-screen.md) för fullständig dokumentation om bildvisaren.

### Index/LUT-sandlåda

För interaktivt indexprovning och visualisering:

* **Indexberäkning i realtid**: Testa olika indexformler
* **LUT-färgkartläggning**: Tillämpa färgövergångar på gråskaliga index
* **Exportera visualiseringar**: Spara färgade indexbilder

Se [Index/LUT-sandlåda](index-lut-sandbox.md) för mer information.

***

## Nästa steg

Nu när du förstår bildlager:

* [**Öppna en bild i helskärmsläge**](opening-an-image-full-screen.md) – Komplett guide till Image Viewer
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Interaktiv indexvisualisering
* [**Formler för multispektrala index**](../project-settings/multispectral-index-formulas.md) – Referens över tillgängliga index
* [**Avsluta bearbetningen**](../processing-images-gui/finishing-the-processing.md) – Förstå bearbetade resultat
