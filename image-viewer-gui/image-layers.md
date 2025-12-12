# Bildlager

Med rullgardinsmenyn Bildlager i Chloros Bildvisare kan du snabbt växla mellan olika versioner av samma bild – från originalbilderna till bearbetade reflektansutdata och beräknade indexbilder.

## Vad är bildlager?

I Chloros avser **lager** de olika bildutdata som finns tillgängliga för en enskild källbild. När du bearbetar bilder skapar Chloros flera versioner:

* **Originalbilder** (JPG- och RAW-filer från din kamera)
* **Reflektanskalibrerade** utdata (om reflektanskalibrering var aktiverad)
* **Målbilder** (om bilden innehåller kalibreringsmål)
* **Indexbilder** (NDVI, NDRE, GNDVI, etc. om index var konfigurerade)

Med **rullgardinsmenyn för lagerval** längst upp till höger i bildvisaren kan du växla mellan dessa versioner utan att lämna visaren.

***

## Tillgängliga lagertyper

### JPG

* Den ursprungliga JPG-förhandsvisningsbilden från din kamera
* Alltid tillgänglig för alla bilder
* Obearbetad, som den fångats av kameran
* Snabbaste att ladda och visa

**När ska du visa den:**

* Snabb förhandsgranskning av originalbilden
* Kontrollera bildkomposition och bildutsnitt
* Kontrollera bildkvaliteten före bearbetning

### RAW (original)

* Originaldata från kamerans RAW-sensor
* Debayerad utan efterbearbetning
* Högre bitdjup än JPG (vanligtvis 12-bitars eller 14-bitars sensordata)

**När ska du visa den:**

* Kontrollera kvaliteten på originaldata från sensorn
* Kontrollera om det finns problem med sensorn eller artefakter
* Jämföra resultat före och efter bearbetning

### RAW (mål)

* Visas endast för bilder som identifierats som innehållande kalibreringsmål
* Visar originalbilden i RAW-format med detekterat mål
* Används för att verifiera att måldetekteringen lyckades

**När ska du visa:**

* Bekräfta att kalibreringsmålen detekterades korrekt
* Kontrollera målbildens kvalitet
* Felsöka kalibreringsproblem

{% hint style=&quot;info&quot; %}
**Målskikt**: Detta skikt visas endast i rullgardinsmenyn för bilder som innehåller kalibreringsmål. Vanliga bilder har inte detta alternativ.
{% endhint %}

### RAW (reflektans)

* Den kalibrerade reflektansutgångsbilden
* Vignettkorrigerad (om aktiverad i bearbetningen)
* Reflektans kalibrerad med hjälp av måldata (om aktiverad)
* Multiband TIFF med alla kamerakanaler
* Pixelvärden representerar procentuell reflektans (vid användning av procentläge)
* Redo att bearbetas med [Index/LUT Sandbox](index-lut-sandbox.md)

**När ska man visa:**

* Inspektera kalibrerade resultat
* Verifiera kalibreringskvaliteten
* Kontrollera pixelvärden för vetenskaplig noggrannhet
* Jämföra med originalet för att se kalibreringseffekter

{% hint style=&quot;success&quot; %}
**Rekommenderat**: Använd RAW-lagret (Reflectance) när du kontrollerar pixelvärden för vetenskapliga mätningar och analyser.
{% endhint %}

### RAW (NDVI Index)... och liknande

* Beräknad vegetationsindexbild (NDVI i detta exempel)
* Indexnamnet ändras beroende på vilket index som konfigurerades under bearbetningen.
* Exempel: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI-index) osv.
* Enkelbandsgråskalebild som visar indexberäkningsresultat
* Ett lager visas för varje index som konfigurerats i projektinställningarna

**Möjliga indexnamn:**

* RAW (NDVI-index)
* RAW (NDRE-index)
* RAW (GNDVI-index)
* RAW (OSAVI-index)
* RAW (EVI-index)
* RAW (SAVI-index)
* Och många fler... (se [Multispektrala indexformler](../project-settings/multispectral-index-formulas.md))

**När ska man visa:**

* Granska indexberäkningsresultat
* Kontrollera indexvärdesintervall
* Identifiera områden av intresse
* Verifiera indexbilder innan de används i GIS eller analys

***

## Använda lagerväljaren

### Öppna rullgardinsmenyn

1. Öppna en bild i helskärmsläge (klicka på valfri miniatyrbild i bildvisaren)
2. Leta reda på **lagermenyn** i det övre högra hörnet av visaren
3. Menyn visar det aktuella valda lagret (t.ex. &quot;JPG&quot;)
4. Klicka på menyn för att se alla tillgängliga lager

### Byta lager

1. Klicka på lagermenyn för att öppna listan
2. Alla tillgängliga lager för den aktuella bilden visas.
3. Klicka på valfritt lagernamn för att växla till den versionen.
4. Bilden uppdateras omedelbart för att visa det valda lagret.

**Snabb växling:**

* Rullgardinsmenyn kommer ihåg ditt senaste val.
* När du navigerar till nästa bild försöker Chloros visa samma lagertyp.
* Om det lagret inte finns på nästa bild används JPG som standard.

### Lager tillgänglighet

Alla lager är inte tillgängliga för varje bild:

**Alltid tillgängligt:**

* ✅ JPG (varje bild har en JPG-förhandsgranskning)

**Villkorligt tillgängligt:**

* ⚠️ RAW (Original) - Endast om bilden togs i RAW- eller RAW+JPG-läge
* ⚠️ RAW (Mål) – Endast om bilden innehåller upptäckta kalibreringsmål
* ⚠️ RAW (Reflektans) – Endast efter bearbetning med reflektanskalibrering aktiverad
* ⚠️ RAW (\[Index] Index) – Endast efter bearbetning med konfigurerade index

***

## Lagerbeständighet

### Navigera mellan bilder

När du navigerar till en annan bild (med piltangenterna eller genom att klicka på miniatyrbilder):

**Lagerinställningen bevaras:**

* Om du visar &quot;RAW (Reflektans)&quot; visas nästa bild som &quot;RAW (Reflektans)&quot; (om tillgänglig)
* Om du visar &quot;RAW (NDVI Index)&quot; visas nästa bild som &quot;RAW (NDVI Index)&quot; (om tillgänglig)
* Om samma lager inte finns, visas JPG som standard.

**Exempel på arbetsflöde:**

1. Öppna bild 1, växla till RAW (NDVI Index).
2. Tryck på → för att visa bild 2.
3. Bild 2 visar automatiskt RAW (NDVI Index)-lagret.
4. Fortsätt navigera – alla bilder visar NDVI-lagret.
5. Mycket effektivt för att granska indexresultat för många bilder.

***

## Vanliga arbetsflöden

### Arbetsflöde 1: Jämförelse före/efter

**Mål**: Jämföra originalbilden med den kalibrerade bilden

1. Öppna den bearbetade bilden i Image Viewer.
2. Välj **RAW (Original)** i rullgardinsmenyn.
3. Notera vinjetteringen och de okalibrerade värdena.
4. Byt till **RAW (Reflectance)** i rullgardinsmenyn.
5. Jämför – vignettering borttagen, värden kalibrerade.

### Arbetsflöde 2: Granskning av index

**Mål**: Granska snabbt NDVI-resultaten för hela datasetet.

1. Öppna den första bearbetade bilden.
2. Välj **RAW (NDVI Index)** från rullgardinsmenyn.
3. Använd → piltangenten för att navigera till nästa bild.
4. NDVI-lagret kvarstår automatiskt.
5. Fortsätt genom alla bilder och kontrollera NDVI-mönster.
6. Växla till **RAW (NDRE Index)** för att jämföra.

### Arbetsflöde 3: Målverifiering

**Mål**: Verifiera att alla målbilder har detekterats korrekt.

1. Navigera till en målbild.
2. Välj **RAW (Target)** från rullgardinsmenyn.
3. Verifiera att kalibreringsmålen är tydligt synliga och detekterade.
4. Navigera till nästa målbild.
5. Upprepa verifieringen för alla mål.

### Arbetsflöde 4: Pixelvärdesinspektion

**Mål**: Kontrollera reflektansvärden för vetenskaplig noggrannhet.

1. Öppna den bearbetade bilden.
2. Välj **RAW (Reflectance)**-lagret.
3. Aktivera **Pixel Percent**-läget (knappen i verktygsfältet uppe till höger).
4. Flytta markören över vegetationsområden.
5. Kontrollera att pixelvärdena ligger inom förväntade intervall (30–70 % för NIR, 5–15 % för Red).
6. Kontrollera att jord- och vattenområden har lämpliga värden.

***

## Förstå pixelvärden per lager

Olika lager visar olika pixelvärdesintervall:

### JPG-lager

* **Intervall**: 0–255 (8 bitar)
* **Betydelse**: Visar värden, gammakorrigerade
* **Användning**: Endast visuell inspektion, inte för vetenskapliga mätningar

### RAW (original)

* **Intervall**: 0–65535 (16 bitar)
* **Betydelse**: Råa digitala siffror från sensorn
* **Användning**: Kontroll av sensorns prestanda, inte kalibrerad

### RAW (reflektans)

* **Intervall**: 0–65 535 (16-bitars TIFF) eller 0,0–1,0 (32-bitars procent)
* **Betydelse**: Kalibrerad procentuell reflektans
* **Användning**: Vetenskapliga mätningar och analyser

**För 16-bitars TIFF:** Dela med 65 535 för att få procentuell reflektans **För 32-bitars procent:** Värdena representerar direkt procent (0,5 = 50 % reflektans)

### RAW (indexbilder)

* **Område**: Varierar beroende på index (vanligtvis -1,0 till +1,0 för normaliserade index)
* **Betydelse**: Resultat av indexberäkning
* **Exempel**:
  * NDVI: -1 till +1 (vegetation vanligtvis 0,4 till 0,9)
  * NDRE: -1 till +1 (stressdetektering)
  * EVI: 0 till 1 (förbättrad vegetation)

***

## Tips och bästa praxis

### Effektiv byte av lager

* **Kännedom om kortkommandon**: Det finns inga kortkommandon för lager, men navigeringspilarna (←/→) fungerar för alla lager
* **Konsekventa arbetsflöden**: Välj ett lager (t.ex. NDVI) och granska hela datasetet innan du byter till ett annat
* **Snabba jämförelser**: Växla mellan Original och Reflektans för att verifiera bearbetningskvaliteten

### Prestandahänsyn

* **JPG laddas snabbast**: Används för snabb navigering genom många bilder
* **RAW-lager laddas långsammare**: Högre upplösning och bitdjup
* **Indexlager**: Liknande hastighet som reflektanslager
* **Första laddningen är långsam**: Efterföljande visningar av samma lager cachelagras och är snabbare

### Kvalitetsverifiering

* **Kontrollera alltid RAW (Original)**: Kontrollera källdatakvaliteten innan du litar på bearbetade resultat
* **Jämför lager**: Använd lagerbyte för att verifiera att bearbetningen fungerade korrekt
* **Kontrollera indexintervall**: Använd Pixel Percent-läget med indexlager för att verifiera att värdena är rimliga

***

## Felsökning

### Skiktet är inte tillgängligt

**Problem**: Det förväntade skiktet visas inte i rullgardinsmenyn.

**Möjliga orsaker:**

* Bilden har inte bearbetats (endast JPG och RAW (Original) är tillgängliga).
* Reflektanskalibrering var inaktiverad under bearbetningen.
* Specifikt index var inte konfigurerat i projektinställningarna.
* Bilden är en målbild (inga index genererade för mål).

**Lösningar:**

1. Kontrollera att bilden har bearbetats (kontrollera utdatamappen för bearbetade filer).
2. Kontrollera projektinställningarna för att bekräfta att index har konfigurerats.
3. Bearbeta om med önskade index aktiverade.

### Felaktigt lager visas

**Problem**: Bilden öppnas i ett oväntat lager.

**Orsak**: Lagerinställningen från föregående bild har överförts, men det lagret finns inte i den aktuella bilden.

**Lösning**: Chloros faller automatiskt tillbaka till JPG när det önskade lagret inte är tillgängligt – detta är normalt.

### Kan inte se kalibreringsmål

**Problem**: RAW-lagret (mål) visar inte måldetektering

**Möjliga orsaker:**

* Mål upptäcktes inte under bearbetningen
* Bilden innehåller faktiskt inga mål
* Inställningarna för måldetektering är för strikta

**Lösningar:**

1. Kontrollera felsökningsloggen för meddelanden om ”Mål hittat”
2. Kontrollera att bilden faktiskt innehåller synliga kalibreringsmål
3. Justera inställningarna för måldetektering i projektinställningarna
4. Se [Välja målbilder](../processing-images-gui/choosing-target-images.md)

***

## Relaterade funktioner

### Bildvisningsverktyg

När du visar ett lager kan du använda:

* **Zoomkontroller**: Förstora för att granska detaljer
* **Panorera**: Klicka och dra för att flytta runt i den zoomade bilden
* **Pixelvärdesinspektion**: Se värden vid markörens position
* **Navigeringspilar**: Flytta mellan bilderna medan du behåller lagret
* **Pixelprocentläge**: Växla mellan DN- och procentvisning

Se [Öppna en bild i helskärmsläge](opening-an-image-full-screen.md) för fullständig dokumentation om bildvisaren.

### Index/LUT Sandbox

För interaktiv indexering och visualisering:

* **Indexberäkning i realtid**: Testa olika indexformler
* **LUT-färgkartläggning**: Tillämpa färgövergångar på gråskalaindex
* **Exportera visualiseringar**: Spara färgade indexbilder

Se [Index/LUT Sandbox](index-lut-sandbox.md) för mer information.

***

## Nästa steg

Nu när du förstår bildlager:

* [**Öppna en bild i helskärmsläge**](opening-an-image-full-screen.md) – Komplett guide till Image Viewer
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Interaktiv indexvisualisering
* [**Multispektrala indexformler**](../project-settings/multispectral-index-formulas.md) – Referens för tillgängliga index
* [**Avsluta bearbetningen**](../processing-images-gui/finishing-the-processing.md) – Förstå bearbetade utdata
