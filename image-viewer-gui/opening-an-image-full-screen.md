# Öppna en bild i helskärmsläge

Bildvisaren Chloros har ett särskilt gränssnitt för helskärmsvisning som gör det möjligt att visa, analysera och bearbeta dina multispektrala bilder. Oavsett om du tittar på originalbilder eller bearbetade resultat erbjuder bildvisaren kraftfulla verktyg för granskning och analys.

## Öppna bildvisaren

### Från filbläddraren

Det vanligaste sättet att öppna en bild i bildvisaren:

1. Se till att du befinner dig på fliken **Filbläddraren** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klicka på valfri **bildminiatyr** i bildrutnätet
3. Bilden öppnas i **huvudförhandsgranskningsområdet** (mitt på skärmen)
4. Bilden är nu laddad och redo för visning i helskärmsläge

### Öppna fliken Bildvisare

När en bild har laddats i förhandsgranskningsområdet:

1. Klicka på **Bildvisare** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i vänster sidofält
2. Fliken Bildvisare öppnas och visar den valda bilden i helskärmsläge
3. Avancerade visnings- och analysverktyg blir tillgängliga i vänster sidofält

***

## Översikt över gränssnittet i bildvisaren

### Huvudvisningsområde

Den största delen av skärmen visar din bild:

* **Full upplösning**: Bilder visas i ursprunglig upplösning
* **Zoombar**: Använd kontrollerna eller mushjulet för att zooma
* **Panorerbar**: Klicka och dra för att flytta runt när du har zoomat
* **Bildförhållandet bibehålls**: Bilderna skalas proportionellt***

## Visningsalternativ

### Grundläggande bildnavigering

#### Bläddra igenom bilder

Navigera genom din bildsamling med hjälp av kortkommandon eller knappar:

* **Nästa bild**: Klicka på →-knappen eller tryck på**→** (högerpilen)
* **Föregående bild**: Klicka på ←-knappen eller tryck på**←** (vänsterpilen)
* **Hoppa till en specifik bild**: Gå tillbaka till filbläddraren och klicka på önskad miniatyrbild

#### Zoomkontroller

Justera förstoringen för att granska bilddetaljer:

**Zooma in:*** Klicka på **+** (plus)-knappen
* Tryck på **+**- eller**=**-tangenten
* Rulla mushjulet **uppåt**

**Zooma ut:*** Klicka på **−** (minus)-knappen
* Tryck på **−** (minus)-tangenten
* Rulla mushjulet **nedåt**

#### Panorera vid zoomning

När du har zoomat in bortom skärmstorleken:

1. Flytta muspekaren över bilden
2. Klicka och **håll ned vänster musknapp**

3.**Dra** för att flytta bilden
4. Släpp för att avsluta panoreringen

**Alternativ**: Använd piltangenterna för att panorera i små steg***

## Granskning av pixelvärden

### Visa pixelvärden vid markören

När du flyttar muspekaren över bilden visas pixelvärden i realtid:**Plats för värdevisning:*** **Flytande siffra och röd linje i index-LUT-gradientlegenden på höger sida*** **När du zoomar in ytterligare, flytande värde nära markören och markerad pixel*** Visar värden för pixel **under markören eller markerad*** Uppdateras när du flyttar musen

***

## Bildtyper du kan visa

### JPG

**JPG-bilder från kamera:**

* Visar JPG-data som förhandsgranskning
* Visar ursprungliga, okorrigerade värden
* Användbart för att kontrollera bildkvaliteten före bearbetning

### RAW (Original)

### RAW (Reflektans)

**Efter bearbetning:**

* Vignettering korrigerad
* Reflektans kalibrerad
* Multiband TIFF (Red, Green, NIR, etc.)
* Vetenskapliga data redo för analys

### RAW (Index)

**NDVI, NDRE, GNDVI, etc. (\_NDVI.tif-filer):**

* Enbandsgråskalebilder
* Pixelvärden representerar indexberäkningsresultat
* Intervall vanligtvis -1 till +1 för normaliserade index
* Kan tillämpa färg-LUT:er för visualisering

***

## Index- och LUT-tillämpning

Tillämpa multispektrala index och färg-Look-Up Tables:

1. Leta upp **Index/LUT Sandbox**i**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sidomenyn
2. Välj vegetationsindex (NDVI, NDRE, etc.)
3. Välj multispektralformel eller skapa en egen anpassad (endast Chloros+)
4. Tillämpa färg-LUT-gradient för visualisering
5. Justera värdeintervall och tröskelvärden

Se [Index/LUT Sandbox](index-lut-sandbox.md) för detaljerade instruktioner.

***

## Kortkommandon

### Navigering

* **→** (högerpil): Nästa bild
* **←** (vänsterpil): Föregående bild
* **Home**: Första bilden i listan
* **End**: Sista bilden i listan

### Zooma

* **+**eller**=**: Zooma in
* **−**: Zooma ut
* **Musens rullhjul**: Zooma in/ut***

### Verifiera indexberäkningar

Kontrollera att indexen har beräknats korrekt:

1. Öppna NDVI eller annan indexbild
2. Kontrollera vegetationsområden:
   * **NDVI**: Bör visa 0,4–0,9 för friska växter
   * **NDRE**: Högre värden för kraftig tillväxt
   * **GNDVI**: Liknar NDVI men är klorofyllkänsligt
3. Kontrollera icke-vegetation:
   * **Jord**: Nära 0 eller något negativt
   * **Vatten**: Negativa värden (-0,5 till 0)***

## Felsökning av visningsproblem

### Bilden öppnas inte

**Möjliga orsaker:**

* Filen har skadats under bearbetningen
* Filformatet stöds inte
* Otillräckligt minne för stor bild

**Lösningar:**

1. Försök öppna i en extern visare för att kontrollera filens integritet
2. Kontrollera att filformatet stämmer överens med förväntad typ
3. Stäng andra program för att frigöra minne
4. Försök med en mindre/annan bild

### Bilden visas svart eller vit

**Möjliga orsaker:**

* Värdeintervall utanför visningskapaciteten
* 32-bitars flyttalsbild med ovanliga värden
* Fel i indexberäkningen

**Lösningar:**

1. Kontrollera pixelvärdena – om alla är mycket låga eller mycket höga, justera visningsintervallet
2. Försök öppna i QGIS eller liknande med automatisk intervalljustering
3. Kontrollera felsökningsloggen från bearbetningen för fel

### Pixelvärdena verkar felaktiga

**Möjliga orsaker:**

* Visar fel bild (original vs bearbetad)
* Kalibreringen tillämpades inte korrekt
* Ljussensordata ingick inte i indata
* Procentläget växlade felaktigt

**Lösningar:**

1. Kontrollera att du tittar på den bearbetade utdata (kontrollera filnamnssuffixet)
2. Kontrollera statusen för procentlägesknappen
3. Jämför med bilder som du vet är korrekta från samma dataset

***

## Nästa steg

Nu när du kan visa bilder i helskärmsläge:

* [**Bildlager**](image-layers.md) – Lär dig mer om multibandsvisualisering
* [**Index/LUT-sandlåda**](index-lut-sandbox.md) – Tillämpa anpassade index och färgkartläggning
* [**Formler för multispektrala index**](../project-settings/multispectral-index-formulas.md) – Förstå tillgängliga index

För bearbetningsflödet, se:

* [**Bearbeta bilder (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Komplett bearbetningsguide
