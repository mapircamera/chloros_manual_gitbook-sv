# Kartmarkörer

Fliken ”Karta” visar dina bilder på en interaktiv 2D-karta utifrån deras GPS-koordinater. Den ger dig en geografisk översikt över en fotograferingssession och är det snabbaste sättet – direkt efter importen – att ta bort bilder som du inte vill bearbeta.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Så här öppnar du fliken ”Karta”

1. Öppna eller skapa ett projekt i Chloros
2. Importera bilder som innehåller GPS-metadata
3. Klicka på fliken **Karta** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> i vänster sidofält
4. Kartan visar en markör vid varje bilds GPS-position

{% hint style="info" %}
**GPS krävs**: endast bilder med GPS-koordinater i sina EXIF-metadata visas på kartan. En bild utan koordinater finns fortfarande kvar i projektet och bearbetas som vanligt – den har bara ingen markör.
{% endhint %}

***

## Justera bilder från fliken ”Karta”

Fliken **Karta**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> har samma knappar för att lägga till <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> och ta bort <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> filer som fliken [**Filbläddraren**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Den visar samma projektfillista, med geografiska kolumner:

| Kolumn        | Innehåll                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Namn**      | Filnamnet så som det hämtades från kameran                             |
| **Latitud**  | Decimala grader, sex decimaler                                |
| **Längdgrad** | Decimala grader, sex decimaler                                |
| **Höjd**  | Meter, en decimal – `-` när bilden saknar höjdinformation |

{% hint style="info" %}
Klicka på valfri kolumnrubrik för att sortera efter den; klicka igen för att vända på ordningen.
{% endhint %}

{% hint style="warning" %}
**Höjden är höjden över havsytan, inte höjden över marken.** Värdet hämtas från bildens EXIF-tagg `GPSAltitude`, som avser medelhavsnivån. Det är inte flyghöjden över terrängen, och Chloros kommer inte att beräkna ett markprovavstånd utifrån detta – över ett fält som ligger 300 m över havsnivån registrerar en drönare på 100 m AGL ungefär 400 m här. Använd kolumnen för att upptäcka avvikande värden och bekräfta en jämn flyghöjd, inte som ett AGL-mått.
{% endhint %}

***

## Bildmarkörer

Varje bild med GPS-data får en markör vid sina koordinater.

### Visning av markörer

* Markörerna placeras vid de exakta koordinaterna som registrerats för varje bild
* Markörer som ligger nära varandra kan visuellt överlappa varandra när du zoomar ut – zooma in för att skilja dem åt
* Valda och markerade markörer visas ovanför de övriga

### Förhandsgranskning vid muspekning

* **Håll muspekaren** över en markör för att visa en miniatyrbild av den bilden med dess filnamn
* **Klicka**på en markör för att markera bilden och**fästa** popup-fönstret – det förblir öppet tills du klickar någon annanstans. Medan ett popup-fönster är fäst kommer det inte att ersättas av ett annat när du håller muspekaren över andra markörer
* Detta är det snabba sättet att hitta en specifik bild i en stor session utan att lämna kartan

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Fliken Karta visar alla geotaggade bilder i projektet</p></figcaption></figure>### Superzoom

{% hint style="success" %}
**SUPERZOOM**: när du når den maximala zoomnivån som bildleverantören har bilder för, fortsätter rutorna att förstoras istället för att stanna, så att du kan skilja markörer som ligger nästan ovanpå varandra.
{% endhint %}

* Superzoom aktiveras endast när du befinner dig **vid** leverantörens maximala zoomnivå för den platsen och rutorna har laddats klart. Under den nivån fungerar zoomningen som vanligt
* Intervallet är **1× till 32×** utöver leverantörens egen maximala nivå
* En indikator i hörnet visar den aktuella superzoomen som en procentandel, och en **×**-knapp bredvid den återför dig till normal zoom med ett klick
* När du zoomar ut går det alltid vidare till själva kartan, så du kan aldrig fastna i superzoom
* Om du zoomar och panorerar medan du är i superzoom överförs den resulterande förskjutningen tillbaka till kartan, så att det område utanför centrum som du flyttat till fortsätter att begära rutor istället för att bli tomt
* Markörer ritas som vektorelement istället för att rasteriseras, så att de förblir skarpa på alla superzoomnivåer

***

## Kartrutsleverantörer

{% hint style="success" %}
**Automatiskt val**: Chloros väljer den ruttjänst som erbjuder den bästa zoomnivån för just där dina bilder befinner sig. Du kan byta manuellt när som helst.
{% endhint %}

| Leverantör        | Anmärkningar                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Brett globalt täckningsområde; stöder alla fyra karttyper                                                                                                            |
| **Esri ArcGIS**| Ofta flygbilder med högre upplösning i vissa regioner. Karttypen**Terrain** erbjuds inte för Esri och dess knapp är inaktiverad när Esri är valt |***

## Kartrutor

Välj kartlagerstyp med knapparna (från vänster till höger):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Typ                 | Visar                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Terräng**          | Höjdskuggning med kartdetaljer (vägar, etiketter). Endast Google       |
| **Karta**              | Standardrutor för gatukartor — alternativet med lägst bandbredd              |
| **Satellit**        | Detaljerade satellitbilder, inga etiketter — alternativet med högst bandbredd |
| **Hybrid** (standard) | Satellitbilder med vägar och etiketter överlagrade                |

Fliken ”Karta” öppnas i läget **Hybrid**. Ditt val överförs till leverantörsväxlingen där leverantören stöder detta.***

## Kartnavigering

* **Zoom**: musens rullhjul eller zoomknapparna på kartan
* **Panorera**: klicka och dra
* **Helskärm**: helskärmsknappen förstorar kartan till hela fönstret***

## Användningsfall

### Granskning av flygväg

* Se täckningsområdet för en drönarsession på ett ögonblick
* Upptäck luckor där en flygning uteblev
* Kontrollera att flygningen följde det planerade mönstret

### Granskning av markundersökning

* Se hur markbaserade bilder är fördelade
* Lokalisera kalibreringsmålramar i förhållande till undersökningsområdet
* Bestäm var ytterligare bilder behövs

### Kvalitetskontroll

* Hitta bilder som tagits på oväntade platser och ta bort dem innan bearbetning
* Sortera efter höjd för att upptäcka en bild som tagits på fel höjd, eller en där GPS-positioneringen var dålig
* Jämför bildernas positioner med fältanteckningarna

***

## Felsökning

### Inga markörer visas

**Möjliga orsaker**

* Bilderna innehåller inga GPS-metadata
* GPS var inaktiverat på kameran under fotograferingen
* EXIF-data raderades av annan programvara före import

**Vad du ska göra**: Kontrollera att GPS är aktiverat på kameran och importera originalfilerna på nytt. Du kan kontrollera om en specifik fil har koordinater genom att leta efter den i filtabellen på fliken Karta – en bild utan koordinater har ingen rad där.

### Markörerna ligger på fel plats

**Möjliga orsaker**: dålig satellitposition vid fotograferingstillfället eller GPS-avvikelse under sessionen.**Åtgärd**: detta är ett problem som uppstod vid fotograferingstillfället och som Chloros inte kan korrigera i efterhand. För precisionsarbete, använd ett PPK/RTK-GPS-arbetsflöde – se inställningen**Tillämpa PPK-korrigeringar** i [Projektinställningar](../project-settings/project-settings.md).

### Kartan är tom eller kartrutorna slutar laddas

Kartruteleverantörerna är onlinetjänster. Om kartrutorna slutar komma in, kontrollera maskinens nätverksanslutning och försök sedan byta leverantör. Om du hade zoomat in extremt mycket, tryck på återställningsknappen **×** för att återgå till en normal zoomnivå och låt kartan begära in kartrutorna på nytt.***

## Relaterade sidor

* [**Bildrutnät**](image-grid.md) – samma bilduppsättning som miniatyrerna
* [**Öppna en bild i helskärmsläge**](opening-an-image-full-screen.md) – granska en bild i detalj
* [**Lägga till filer i ett projekt**](../processing-images-gui/adding-files-to-a-project.md) — knapparna för att lägga till/ta bort filer som finns på den här fliken
