# Projektinställningar

Projektinställningarna <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i Chloros kan du konfigurera alla aspekter av bildbehandling, kalibreringsmåldetektering, multispektrala indexberäkningar och exportalternativ för ditt projekt. Dessa inställningar sparas med ditt projekt och kan sparas som mallar för återanvändning i flera projekt.

## Öppna projektinställningar

Så här öppnar du projektinställningar:

1. Öppna ett projekt i Chloros
2. Klicka på fliken **Projektinställningar**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i vänster sidofält
3. Inställningspanelen visar alla tillgängliga konfigurationsalternativ organiserade efter kategori

***

## Målidentifiering

Dessa inställningar styr hur Chloros identifierar och bearbetar kalibreringsmål i dina bilder.

### Minsta kalibreringsprovområde (px)

* **Typ**: Tal
* **Intervall**: 0 till 10 000 pixlar
* **Standard**: 25 pixlar
* **Beskrivning**: Ställer in det minsta område (i pixlar) som krävs för att ett detekterat område ska betraktas som ett giltigt kalibreringsmålprov. Mindre värden detekterar mindre mål men kan öka antalet falska positiva resultat. Större värden kräver större, tydligare målområden för detektering.
* **När ska du justera**:
  * Öka om du får falska detekteringar på små bildartefakter.
  * Minska om dina kalibreringsmål verkar små i dina bilder och inte detekteras.

### Minsta målkluster (0–100)

* **Typ**: Tal
* **Intervall**: 0 till 100
* **Standard**: 60
* **Beskrivning**: Styr klustertröskeln för gruppering av liknande färgade områden vid detektering av kalibreringsmål. Högre värden kräver att fler liknande färger grupperas tillsammans, vilket resulterar i en mer konservativ måldetektering. Lägre värden tillåter större färgvariation inom en målgrupp.
* **När ska du justera**:
  * Öka om kalibreringsmål delas upp i flera detekteringar.
  * Minska om kalibreringsmål med färgvariation inte detekteras fullständigt.

***

## Bearbetning

Dessa inställningar styr hur Chloros bearbetar och kalibrerar dina bilder.

### Vignettkorrigering

* **Typ**: Kryssruta
* **Standard**: Aktiverad (markerad)
* **Beskrivning**: Tillämpar vignettkorrigering för att kompensera för linsens mörkare kanter i bildens ytterkanter. Vignettering är ett vanligt optiskt fenomen där hörnen och kanterna på en bild verkar mörkare än mitten på grund av linsens egenskaper.
* **När ska du inaktivera**: Inaktivera endast om din kamera/lins-kombination redan har tillämpat vignettkorrigering, eller om du vill korrigera vignetteringen manuellt i efterbearbetningen.

### Reflektanskalibrering/vitbalans

* **Typ**: Kryssruta
* **Standard**: Aktiverad (markerad)
* **Beskrivning**: Aktiverar automatisk reflektanskalibrering med hjälp av detekterade kalibreringsmål i dina bilder. Detta normaliserar reflektansvärdena i hela din dataset och säkerställer konsekventa mätningar oavsett ljusförhållanden.
* **När ska du inaktivera**: Inaktivera endast om du vill bearbeta råa, okalibrerade bilder eller om du använder ett annat kalibreringsarbetsflöde.

### Debayer-metod

* **Typ**: Rullgardinsmeny
* **Alternativ**:
  * Hög kvalitet (snabbare) – För närvarande det enda tillgängliga alternativet
* **Standard**: Hög kvalitet (snabbare)
* **Beskrivning**: Väljer den demosaicing-algoritm som används för att konvertera råa sensordata i Bayer-mönster till fullfärgsbilder. Metoden ”Hög kvalitet (snabbare)” ger en optimal balans mellan bearbetningshastighet och bildkvalitet.
* **Obs!**: Ytterligare debayer-metoder kan läggas till i framtida versioner av Chloros.

### Minsta omkalibreringsintervall

* **Typ**: Tal
* **Intervall**: 0 till 3 600 sekunder
* **Standard**: 0 sekunder
* **Beskrivning**: Ställer in det minsta tidsintervallet (i sekunder) mellan användning av kalibreringsmål. När det är inställt på 0 kommer Chloros att använda alla upptäckta kalibreringsmål. När det är inställt på ett högre värde kommer Chloros endast att använda kalibreringsmål som är separerade med minst detta antal sekunder, vilket minskar bearbetningstiden för datamängder med frekventa kalibreringsmål.
* **När ska justeras**:
  * Ställ in på 0 för maximal kalibreringsnoggrannhet när ljusförhållandena varierar.
  * Öka (t.ex. till 60–300 sekunder) för snabbare bearbetning när ljuset är konstant och du har frekventa kalibreringsmålbilder.

### Ljusgivarens tidszonsförskjutning

* **Typ**: Tal
* **Intervall**: -12 till +12 timmar
* **Standard**: 0 timmar
* **Beskrivning**: Anger tidszonsförskjutningen (i timmar från UTC) för ljussensorns datatidsstämplar. Detta används vid bearbetning av PPK-datafiler (Post-Processed Kinematic) för att säkerställa korrekt tidssynkronisering mellan bildtagningar och GPS-data.
* **När ska du justera**: Ställ in detta på din lokala tidszonsförskjutning om dina PPK-data använder lokal tid istället för UTC. Till exempel:
  * Pacific Time: -8 eller -7 (beroende på sommartid)
  * Eastern Time: -5 eller -4 (beroende på sommartid)
  * Central European Time: +1 eller +2 (beroende på sommartid)

### Tillämpa PPK-korrigeringar

* **Typ**: Kryssruta
* **Standard**: Inaktiverad (avmarkerad)
* **Beskrivning**: Aktiverar användningen av PPK-korrigeringar (Post-Processed Kinematic) från MAPIR DAQ-inspelare som innehåller GPS (GNSS). När denna funktion är aktiverad kommer Chloros att använda alla .daq-loggfiler som innehåller exponeringspinndata i din projektkatalog och tillämpa exakta geolokaliseringskorrigeringar på dina bilder.
* **Krav**: .daq-loggfil med exponeringsstiftposter måste finnas i din projektkatalog
* **När ska funktionen aktiveras**: Vi rekommenderar att du alltid aktiverar PPK-korrigering om du har exponeringsfeedbackposter i din .daq-loggfil.

### Exponeringsstift 1

* **Typ**: Rullgardinsmeny
* **Synlighet**: Endast synlig när &quot;Tillämpa PPK-korrigeringar&quot; är aktiverat OCH exponeringsdata är tillgängliga för stift 1
* **Alternativ**:
  * Kameramodellnamn som upptäckts i projektet
  * &quot;Använd inte&quot; – Ignorera detta exponeringsstift
* **Standard**: Väljs automatiskt baserat på projektkonfigurationen
* **Beskrivning**: Tilldelar en specifik kamera till exponeringsstift 1 för PPK-tidssynkronisering. Exponeringsstiftet registrerar den exakta tidpunkten när kamerans slutare utlöses, vilket är avgörande för en korrekt PPK-geolokalisering.
* **Automatiskt val**:
  * Enkel kamera + enkel stift: Väljer automatiskt kameran
  * En kamera + två stift: Stift 1 tilldelas automatiskt till kameran
  * Flera kameror: Manuell val krävs

### Exponeringsstift 2

* **Typ**: Rullgardinsmeny
* **Synlighet**: Endast synlig när &quot;Tillämpa PPK-korrigeringar&quot; är aktiverat OCH exponeringsdata är tillgängliga för stift 2
* **Alternativ**:
  * Kameramodellnamn som upptäckts i projektet
  * &quot;Använd inte&quot; – Ignorera denna exponeringsstift
* **Standard**: Väljs automatiskt baserat på projektkonfigurationen
* **Beskrivning**: Tilldelar en specifik kamera till exponeringsstift 2 för PPK-tidssynkronisering vid användning av en dubbelkamerakonfiguration.
* **Automatiskt val**:
  * Enkel kamera + enkel stift: Stift 2 ställs automatiskt in på &quot;Använd inte&quot;
  * Enkel kamera + två stift: Stift 2 ställs automatiskt in på &quot;Använd inte&quot;
  * Flera kameror: Manuell val krävs
* **Obs**: Samma kamera kan inte tilldelas både stift 1 och stift 2 samtidigt.

***

## Index

Med dessa inställningar kan du konfigurera multispektrala index för analys och visualisering.

### Lägg till index

* **Typ**: Specialpanel för indexkonfiguration
* **Beskrivning**: Öppnar en interaktiv panel där du kan välja och konfigurera multispektrala vegetationsindex (NDVI, NDRE, EVI, etc.) som ska beräknas under bildbearbetningen. Du kan lägga till flera index, var och en med sina egna visualiseringsinställningar.
* **Tillgängliga index**: Systemet innehåller över 30 fördefinierade multispektrala index, inklusive:
  * NDVI (normaliserat vegetationsindex)
  * NDRE (normaliserad skillnad RedEdge)
  * EVI (förbättrat vegetationsindex)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Och många fler (se [Multispektrala indexformler](multispectral-index-formulas.md) för en fullständig lista)
* **Funktioner**:
  * Välj bland fördefinierade indexformler
  * Konfigurera visualiseringsfärggradienter (LUT – Look-Up Tables)
  * Ställ in tröskelvärden för analys
  * Skapa anpassade indexformler

### Anpassade formler (Chloros+ Funktion)

* **Typ**: Array av anpassade formeldefinitioner
* **Beskrivning**: Gör det möjligt att skapa och spara anpassade multispektrala indexformler med hjälp av bandmatematik. Anpassade formler sparas med dina projektinställningar och kan användas precis som inbyggda index.
* **Så här skapar du**:
  1. I indexkonfigurationspanelen letar du efter alternativet för anpassade formler.
  2. Definiera din formel med hjälp av bandidentifierare (t.ex. NIR, Red, Green, Blue).
  3. Spara formeln med ett beskrivande namn.
* **Formelsyntax**: Standardmatematiska operationer stöds, inklusive:
  * Aritmetik: `+`, `-`, `*`, `/`
  * Parenteser för operationsordning
  * Bandreferenser: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Exportera

Dessa inställningar styr formatet och kvaliteten på exporterade bearbetade bilder.

### Kalibrerat bildformat

* **Typ**: Rullgardinsmeny
* **Alternativ**:
  * **TIFF (16-bitars)** – Okomprimerat 16-bitars TIFF-format
  * **TIFF (32-bitars, procent)** – 32-bitars flyttalsformat TIFF med reflektansvärden i procent
  * **PNG (8-bitars)** - Komprimerat 8-bitars PNG-format
  * **JPG (8-bitars)** - Komprimerat 8-bitars JPEG-format
* **Standard**: TIFF (16-bitars)
* **Beskrivning**: Väljer filformat för att spara bearbetade och kalibrerade bilder.
* **Formatrekommendationer**:
  * **TIFF (16-bitars)**: Rekommenderas för vetenskaplig analys och professionella arbetsflöden. Bevarar maximal datakvalitet utan komprimeringsartefakter. Bäst för multispektral analys och vidare bearbetning i GIS-programvara.
  * **TIFF (32-bitars, procent)**: Bäst för arbetsflöden som kräver reflektansvärden i procent (0–100 %). Ger maximal precision för radiometriska mätningar.
  * **PNG (8-bitars)**: Bra för visning på webben och allmän visualisering. Mindre filstorlekar med förlustfri komprimering, men reducerat dynamiskt omfång.
  * **JPG (8-bitars)**: Minsta filstorlekar, bäst för förhandsgranskning och endast visning på webben. Använder förlustrik komprimering som inte är lämplig för vetenskaplig analys.

***

## Spara projektmall

Med den här funktionen kan du spara dina aktuella projektinställningar som en återanvändbar mall.

* **Typ**: Textinmatning + Spara-knapp
* **Beskrivning**: Ange ett beskrivande namn för din inställningsmall och klicka på spara-ikonen. Mallen lagrar alla dina aktuella projektinställningar (målidentifiering, bearbetningsalternativ, index och exportformat) för enkel återanvändning i framtida projekt.
* **Användningsfall**:
  * Skapa mallar för olika kamerasystem (RGB, multispektral, NIR)
  * Spara standardkonfigurationer för specifika grödtyper eller analysarbetsflöden
  * Dela enhetliga inställningar inom ett team
* **Så här använder du funktionen**:
  1. Konfigurera alla önskade projektinställningar.
  2. Ange ett mallnamn (t.ex. ”RedEdge Survey3 NDVI Standard”).
  3. Klicka på ikonen Spara.
  4. Mallen kan nu laddas när du skapar nya projekt.

***

## Spara projektmapp

Denna inställning anger var nya projekt sparas som standard.

* **Typ**: Visning av katalogväg + knappen Redigera
* **Standard**: `C:\Users\[Username]\Chloros Projects`
* **Beskrivning**: Visar den aktuella standardkatalogen där nya Chloros-projekt skapas. Klicka på redigeringsikonen för att välja en annan katalog.
* **När ska du ändra**:
  * Ställ in på en nätverksenhet för teamsamarbete.
  * Ändra till en enhet med mer lagringsutrymme för stora datamängder.
  * Organisera projekt efter år, kund eller projekttyp i olika mappar.
* **Obs**: Att ändra denna inställning påverkar endast NYA projekt. Befintliga projekt förblir på sina ursprungliga platser.

***

## Inställningars beständighet

Alla projektinställningar sparas automatiskt med din projektfil (`.mapir`-projektformat). När du öppnar ett projekt igen återställs alla inställningar exakt som du lämnade dem.

### Inställningshierarki

Inställningarna tillämpas i följande ordning:

1. **Systemstandarder** – Inbyggda standarder definierade av Chloros
2. **Mallinställningar** – Om du laddar en mall när du skapar ett projekt
3. **Sparade projektinställningar** – Inställningar som sparats med projektfilen
4. **Manuella justeringar** – Alla ändringar du gör under den aktuella sessionen

### Inställningar och bildbehandling

De flesta inställningsändringar (särskilt i kategorierna Bearbetning och Export) utlöser ombearbetning av bilder för att återspegla de nya inställningarna. Vissa inställningar är dock &quot;endast för export&quot; och kräver inte omedelbar ombearbetning:

* Spara projektmall
* Arbetsmapp
* Kalibrerat bildformat (gäller vid export)

***

## Bästa praxis

1. **Börja med standardinställningarna**: Standardinställningarna fungerar bra för de flesta MAPIR-kamerasystem och typiska arbetsflöden.
2. **Skapa mallar**: När du har optimerat inställningarna för ett specifikt arbetsflöde eller en specifik kamera sparar du dem som en mall för att säkerställa konsistens mellan olika projekt.
3. **Testa innan fullständig bearbetning**: När du experimenterar med nya inställningar testar du på en liten delmängd av bilderna innan du bearbetar hela datasetet.
4. **Dokumentera dina inställningar**: Använd beskrivande mallnamn som anger kamerasystem, bearbetningstyp och avsedd användning (t.ex. &quot;Survey3\_RGB\_NDVI\_Agriculture&quot;).
5. **Val av exportformat**: Välj exportformat baserat på din slutliga användning:
   * Vetenskaplig analys → TIFF (16-bitars eller 32-bitars)
   * GIS-bearbetning → TIFF (16-bitars)
   * Snabb visualisering → PNG (8-bitars)
   * Webbdela → JPG (8-bitars)

***

För mer information om multispektrala index i Chloros, se sidan [Multispektrala indexformler](multispectral-index-formulas.md).
