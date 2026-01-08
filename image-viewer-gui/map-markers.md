# Kartmarkörer

Fliken Karta visar dina bilder på en interaktiv 2D-karta baserad på deras GPS-koordinater. Detta ger en geografisk översikt över din bildtagning och hjälper dig att visualisera den rumsliga täckningen. Det är också användbart när du först importerar dina bilder för att snabbt ta bort bilder som du inte behöver bearbeta.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Öppna fliken Karta

1. Öppna eller skapa ett projekt i Chloros
2. Importera bilder som innehåller GPS-metadata
3. Klicka på fliken **Karta** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> i vänster sidofält.
4. Kartan visar markörer vid varje bilds GPS-position.

{% hint style="info" %}
**GPS krävs**: Endast bilder med inbäddade GPS-koordinater i EXIF-metadata visas på kartan. Se till att GPS är aktiverat på din kamera när du tar bilder.
{% endhint %}

***

## Justera bilder från fliken Karta

Fliken **Karta** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> har samma funktioner för att lägga till  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  och ta bort  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  filknapparna som fliken [**Filbläddraren**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Den visar också samma projektfiltabell, men med olika kolumnrubriker:

### Filnamn

* Originalfilnamn från kameran
* Behåller kamerans namngivningskonvention (t.ex. IMG\_0001.RAW)

### Latitud

* Bildens latitud

### Longitud

* Bildens longitud

### Höjd

* Bildens höjd

{% hint style="info" %}
Om du klickar på tabellens kolumnrubriker sorteras även raddata
{% endhint %}

***

## Bildmarkörer

Varje bild med GPS-data representeras av en markör på kartan:

### Markörvisning

* Markörerna anger de exakta GPS-koordinaterna där varje bild togs.
* Klustrade markörer kan grupperas när du zoomar ut.
* Zooma in för att se enskilda bilders platser.

{% hint style="success" %}
SUPERZOOM: När du når den maximala zoomnivån från kartkortsleverantören förstoras kartkortet vid ytterligare zoomning, så att du kan se markörer som ligger nära varandra.
{% endhint %}

### Förhandsgranskning vid muspekning

* **Håll muspekaren** över en markör för att se en miniatyrförhandsgranskning av den bilden.
* Detta möjliggör snabb visuell identifiering utan att lämna kartvyn.
* Användbart för att hitta specifika bilder inom en stor bildsession.

***

## Kartkakel-leverantörer

{% hint style="success" %}
**Automatisk val**: Chloros väljer automatiskt den kartleverantör som ger den bästa zoomnivån för din aktuella kartposition. Du kan växla mellan leverantörer manuellt om du vill.
{% endhint %}

Fliken Karta stöder två kartleverantörer för bakgrundskartbilder:

### Google Maps

* Standardbilder från satellit och karta från Google.
* Bäst för allmän täckning över hela världen.

### ESRI

* Satellit- och flygbilder från ESRI ArcGIS.
* Ger ofta bilder med högre upplösning i vissa regioner.

***

## Kartkakeltyper

Du kan välja kartlagstyp (från vänster till höger):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="line">### Terräng

Visar höjdprofiler och kartkakel med detaljer (vägar etc.)

### Karta

Visar standardkartkakel (lägre bandbredd) med detaljer (vägar etc.)

### Satellit

Visar detaljerade satellitkartkakel (högre bandbredd)

### Hybrid

Visar satellitkartkakel med extra detaljer (vägar etc.)

***

## Kartnavigering

### Zoomkontroller

* **Zooma in/ut**: Använd musens rullhjul eller zoomknapparna.
* **Helskärm**: Visa kartan i helskärmsläge.

### Panoreringkontroller

* **Panorera**: Klicka och dra för att flytta runt på kartan.

***

## Användningsfall

### Visualisering av flygväg

* Visa täckningsområdet för drönarens bildtagning
* Identifiera luckor i bildtäckningen
* Verifiera flygvägens genomförande

### Granskning av markundersökning

* Se den rumsliga fördelningen av markbaserade bilder
* Lokalisera kalibreringsmålbilder i förhållande till undersökningsområdet
* Planera ytterligare bildtagningsplatser

### Kvalitetskontroll

* Identifiera snabbt bilder som tagits på oväntade platser.
* Verifiera GPS-noggrannheten i hela datasetet.
* Korsreferera bildplatser med fältanteckningar.

***

## Felsökning

### Inga markörer visas

**Möjliga orsaker:**

* Bilderna innehåller inte GPS-metadata.
* GPS var inaktiverat på kameran under fotograferingen.
* EXIF-data har tagits bort av extern programvara.

**Lösning**: Kontrollera att GPS är aktiverat på kameran och importera originalfilerna på nytt.

### Markörer på fel plats

**Möjliga orsaker:**

* Kamerans GPS hade dålig satellitfix.
* GPS-avvikelse under fotograferingen.

**Lösning**: Detta är vanligtvis ett problem med fotograferingstidpunkten. Överväg att använda PPK/RTK GPS för precisionsapplikationer.
