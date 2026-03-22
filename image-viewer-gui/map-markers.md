# Kartmarkörer

Fliken ”Karta” visar dina bilder på en interaktiv 2D-karta utifrån deras GPS-koordinater. Detta ger en geografisk översikt över din fotograferingssession och hjälper dig att visualisera den geografiska täckningen. Det är också användbart när du först importerar dina bilder för att snabbt ta bort bilder som du inte behöver bearbeta.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Öppna fliken Karta

1. Öppna eller skapa ett projekt i Chloros
2. Importera bilder som innehåller GPS-metadata
3. Klicka på fliken **Karta** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> i vänster sidofält
4. Kartan visar markörer vid varje bilds GPS-position

{% hint style="info" %}
**GPS krävs**: Endast bilder med inbäddade GPS-koordinater i EXIF-metadata visas på kartan. Se till att GPS är aktiverat på din kamera under fotograferingen.
{% endhint %}

***

## Justera bilder från fliken Karta

Fliken **Karta**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> har samma lägg till  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  och ta bort  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  filer som fliken [**Filbläddraren**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Den visar också samma lista över projektfiler, men med andra kolumnrubriker:

### Filnamn

* Ursprungligt filnamn från kameran
* Behåller kamerans namngivningskonvention (t.ex. IMG\_0001.RAW)

### Latitud

* Bildens latitud

### Longitud

* Bildens longitud

### Höjd

* Bildens höjd

{% hint style="info" %}
Om du klickar på tabellens kolumnrubriker sorteras också raddata
{% endhint %}

***

## Bildmarkörer

Varje bild med GPS-data representeras av en markör på kartan:

### Markörvisning

* Markörerna anger de exakta GPS-koordinaterna där varje bild togs
* Markörer som ligger nära varandra kan grupperas när du zoomar ut
* Zooma in för att se enskilda bilders platser

{% hint style="success" %}
SUPER-ZOOM: När du når den maximala zoomnivån från kartleverantören förstoras rutan ytterligare vid ytterligare zoomning, vilket gör att du kan se markörer som ligger nära varandra.
{% endhint %}

### Förhandsgranskning vid muspekning

* **Håll muspekaren** över en markör för att se en miniatyrbild av den bilden
* Detta möjliggör snabb visuell identifiering utan att lämna kartvyn
* Användbart för att hitta specifika bilder inom en stor fotograferingssession

***

## Kartrutsleverantörer

{% hint style="success" %}
**Automatiskt val**: Chloros väljer automatiskt den rutleverantör som ger den bästa zoomnivån för din aktuella kartposition. Du kan manuellt växla mellan leverantörer om du vill.
{% endhint %}

Fliken Karta stöder två rutleverantörer för bakgrundskartbilderna:

### Google Maps

* Standardbildmaterial från satellit och kartor från Google
* Bäst för allmän täckning över hela världen

### ESRI

* Satellit- och flygbilder från ESRI ArcGIS
* Ger ofta bilder med högre upplösning i vissa regioner

***

## Kartrutor

Du kan välja typ av kartlager (från vänster till höger):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terräng

Visar höjdprofiler och kartrutor med detaljer (vägar etc.)

### Karta

Visar standardkartrutor (lägre bandbredd) med detaljer (vägar etc.)

### Satellit

Visar detaljerade (högre bandbredd) satellitkartrutor

### Hybrid

Visar satellitkartrutor med extra detaljer (vägar etc.)

***

## Kartnavigering

### Zoomkontroller

* **Zooma in/ut**: Använd musens rullhjul eller zoomknapparna
* **Helskärm**: Visa kartan i helskärmsläge

### Panoreringskontroller

* **Panorera**: Klicka och dra för att flytta runt på kartan***

## Användningsfall

### Visualisering av flygväg

* Visa täckningsområdet för drönarinspelningar
* Identifiera luckor i bildtäckningen
* Verifiera att flygvägen har följts

### Granskning av markundersökning

* Se den rumsliga fördelningen av markbaserade bilder
* Lokalisera kalibreringsmålbilder i förhållande till undersökningsområdet
* Planera ytterligare inspelningsplatser

### Kvalitetskontroll

* Identifiera snabbt bilder som tagits på oväntade platser
* Verifiera GPS-noggrannheten i hela datasetet
* Jämför bildplatser med fältanteckningar

***

## Felsökning

### Inga markörer visas

**Möjliga orsaker:**

* Bilderna innehåller inte GPS-metadata
* GPS var inaktiverat på kameran under fotograferingen
* EXIF-data har tagits bort av extern programvara

**Lösning**: Kontrollera att GPS är aktiverat på din kamera och importera originalfilerna på nytt

### Markörer på fel plats

**Möjliga orsaker:**

* Kamerans GPS hade dålig satellitfix
* GPS-avvikelse under fotograferingen

**Lösning**: Detta är vanligtvis ett problem som uppstår vid fotograferingstillfället; överväg att använda PPK/RTK-GPS för precisionsapplikationer
