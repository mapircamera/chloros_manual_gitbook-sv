# Öppna en bild i helskärmsläge

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>En bild öppnad i helskärmsläge, med lagerväljaren uppe till höger</p></figcaption></figure>

Chloros Image Viewer är ett gränssnitt i helskärmsläge för visning, granskning och mätning av dina bilder. Det är här du läser av **verkliga pixelvärden** — DN per kanal, reflektans i procent eller strålningsintensitet i W/m²/sr/nm — istället för den utsträckta förhandsvisningen som skärmen visar.

## Öppna bildvisaren

### Från filbläddraren

1. Öppna fliken **Filbläddraren** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klicka på valfri **miniatyrbild** i [bildrutnätet](image-grid.md)
3. Bilden öppnas i helskärmsläge på fliken **Bildvisaren**

Bilden öppnas i det produktlager som rutnätet visade. Om rutnätet är inställt på `RAW (Reflectance)` är det det lagret du hamnar på.

### Öppna sidopanelen för bildvisaren

Klicka på **bildvisarens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">-ikon i vänster sidopanel för att fälla ut analyspanelen. Den innehåller, uppifrån och ner:

* bildens namn och kameramodell
* knappen **Exportera/spara bild(er)** (endast när ett index eller en LUT är aktivt)
* kryssrutorna **Index**och**LUT** samt konfigurationspanelen för index – se [Index/LUT-sandlåda](index-lut-sandbox.md)
* panelen **Markörvärden**: avläsning per kanal, lagerhistogram och GSD-reglaget***

## Navigera och zooma

### Bläddra bland bilder

* **Nästa bild**: knappen → eller tangenten**→** (högerpil)
* **Föregående bild**: knappen ← eller tangenten**←** (vänsterpilen)
* **Hoppa till en specifik bild**: gå tillbaka till rutnätet och klicka på dess miniatyrbild

Zoom och panorering bibehålls när du växlar mellan bilder, så att du kan bläddra igenom en bildserie samtidigt som du stannar kvar på samma del av bilden.

### Zooma

Zoomningen styrs med **mushjulet**, i steg om 15 %, och är förankrad vid markören – punkten under pekaren förblir under pekaren. Omfånget begränsas av bildens och fönstrets storlek: du kan inte zooma ut mer än att bilden anpassas till fönstret, och den övre gränsen bestäms av bildens ursprungliga upplösning.

Det finns inga särskilda zoomknappar i helskärmsvisaren. (I rutnätet ändrar **Ctrl + `+` / `−`** ändrar storleken på miniatyrerna – en annan funktion.)

### Panorera vid zoomning

Klicka och håll ned vänster musknapp över bilden och dra. Panoreringen är begränsad så att bilden inte kan dras utanför skärmen.

### Granskning per pixel vid hög förstoring

När den effektiva förstoringen överstiger **60×** ritar Chloros en markeringsruta runt den enskilda visade pixeln under markören och visar ett flytande värde bredvid den.

&quot;Den ”effektiva” förstoringen räknar utifrån GSD-blockstorleken: med en blockstorlek på 8 visas markeringen vid 7,5× zoom istället för 60×, eftersom en visad pixel redan motsvarar 8 × 8 källpixlar. Zooma ut till under tröskelvärdet så försvinner markeringen.

### Tangentbordsgenvägar

| Tangent                             | Var       | Åtgärd                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Helskärm | Nästa bild                          |
| **←**                           | Helskärm | Föregående bild                      |
| **Ctrl + R**                    | Helskärm | Återställ index/LUT-sandlådan         |
| **Ctrl + `+`**/**Ctrl + `=`** | Rutnät        | Större miniatyrbilder (4 px per tryck)  |
| **Ctrl + `−`**                  | Rutnät        | Mindre miniatyrbilder (4 px per tryck) |***

## Markörvärden

Flytta markören över bilden så visar panelen **Markörvärden** värdet för varje kanal under den.

{% hint style="success" %}
**Detta är filens verkliga värden.** Arbetytan på skärmen är en 8-bitars utsträckt förhandsgranskning och kan inte tillhandahålla dem, så Chloros hämtar värdena från den faktiska produktfilen för avläsningen. Det är därför en 12-bitars råbild visar värden över 255, och varför ett float32-radiance-lager visar fysiska enheter.
{% endhint %}

### Vad kolumnerna betyder

Panelen anpassas efter det lager du tittar på:

| Lager du tittar på              | Visade kolumner    | Anmärkningar                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Reflektans                        | **DN**och**%** | Procentvärdet beräknas utifrån filens egen skala — se nedan                                      |
| Strålning                           | **W/m²/sr/nm**   | Fysiska värden av typen float; ingen DN-kolumn, eftersom ett DN-värde är meningslöst här                           |
| Rådata / Debayered / förhandsgranskning / JPG    | **DN**           | Heltalsvärden                                                                         |
| 32-bitars procentuell reflektans export | endast **%**       | Det lagrade flyttalsvärdet är inte ett DN, så att avrunda det till ett heltal skulle resultera i ett meningslöst `0` eller `1` |

Varje rad är märkt med kanalnamnet för kamerans filter — `Red / Green / NIR` för RGN, `Orange / Cyan / NIR` för OCN, `NIR / Green / Blue` för NGB, `Red / Green / Blue` för RGB samt det enskilda bandnamnet för RE-, NIR och mono-M3M-kameror. Varje etikett har en färgad prick som motsvarar de kanalcirklar som används i indexformelredigeraren.

Sparade **index- och LUT-**bilder är ett specialfall: de innehåller färgkartkomponenter istället för spektralband, så deras rader märks med `Red / Green / Blue` (eller `Index` för en enkelkanalig indexfil) istället för med kamerans filternamn.

När ett index är aktivt i sandlådan visas en extra rad under kanalerna som visar **indexvärdet** vid markören, tillsammans med indexets namn och en vit prick som motsvarar dess markör på histogrammet.

### Reflektansprocenten använder varje fils egen skala

{% hint style="warning" %}
**Antag inte att 65535 = 100 %.** Chloros lagrar reflektans i olika skalor beroende på vilken kamera som genererat den, och visningsprogrammet avgör vilken som är korrekt för varje fil.
{% endhint %}

| Källa                  | DN som motsvarar reflektans 1,0 | Hur den identifieras                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | XMP-taggen `Chloros:PixelScale=32768` skrivs in i varje LATTICE-export av reflektans. Tack vare det dubbla utrymmet kan filen innehålla ρ över 1,0 utan att klippas |
| **Survey3**|**65535**                      | Ingen Chloros XMP-skalningstagg — Survey3-kalibreringen skriver ρ × dtype-max och klipper vid 1,0                                                               |

Visningsprogrammet, index-/LUT-sandlådan och indexexporten beräknar alla skalan genom samma enda implementering, så ett värde du läser vid markören är samma värde som används i indexberäkningen.

Två konsekvenser som är värda att känna till:

* En **32-bitars procent**TIFF lagrar DN/65535 som ett flyttal, och en**8-bitars** PNG/JPG-export lagrar DN × 255/65535 — visningsprogrammet konverterar båda tillbaka innan procentvärdet visas.
* Ett fall går inte att återställa: en **8-bitars TIFF-export av en 8-bitars källbild** beskärs till 0–255 istället för att skalas om, och har avsiktligt ingen skalningsetikett. För dessa filer visar panelen endast DN, utan procentkolumn. Detta är det ärliga svaret, inte ett fel.***

## Lagrets histogram

Under markörraderna finns ett realtidshistogram för det lager du tittar på, i **256 bin**. Som standard ritas en sammansatt kurva, viktad `(R + 2G + B) / 4` — samma mätutrymme som LATTICE-kamerans histogram använder. Om du aktiverar**RGB** ersätts den med kurvor per kanal i kanalernas färger, additivt blandade så att överlappningar förblir läsbara. Monolager ritar alltid den enda kurvan.

Den horisontella axeln är i lagrets egen enhet:

| Lager       | Axelenhet  | Axelns maximivärde                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Reflektans | procent    | 125 % — produktens headroom tillåter ρ över 1,0           |
| Strålning    | W/m²/sr/nm | Bildens egen toppvärde, avrundat uppåt till två signifikanta siffror |
| 8-bitarsdata  | DN         | 255                                                        |
| 12-bitarsdata | DN         | 4095                                                       |
| 16-bitarsdata | DN         | 65535                                                      |

När axeln är i DN och hamnar på ett av dessa tre tak, vet Chloros även bitdjupet för det du tittar på.

Tre knappar finns ovanför histogrammet:

| Knapp     | Standard | Effekt                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CURSOR** | På      | Ritar markeringslinjer på histogrammet vid de exakta värdena som visas i raderna ovan, så att du kan se var pixeln under markören befinner sig i bildens fördelning. I läget RGB finns det en markör per kanal i sin egen färg; i övriga fall en enda vit markör vid det kombinerade värdet |
| **INDEX**| På      | Visas endast när ett index är aktivt. Växlar histogrammet från källbanden till**indexvärdesfördelningen**, där de två klipptrösklarna ritas som orange streckade linjer och markörens indexvärde som en vit linje                                                          |
| **RGB**| Av     | Växlar från den kombinerade kurvan till kurvor per kanal. På en monosensor visar denna knapp**MONO** och är inaktiverad — det finns bara en kanal att visa                                                                                                                                  |

Histogrammet beräknas utifrån de **block du kan se**, inte utifrån källpixlarna bakom dem: ändra GSD-blockstorleken så beräknas fördelningen om, så att histogrammet, markören och den visade bilden alltid stämmer överens.***

## GSD-blockstorlek

Längst ner på panelen finns kontrollen **GSD (px)**: en inmatningsruta, ett skjutreglage från**1 till 256**och en**RESET**-knapp.

Den gör den _visade_ bilden grov genom att beräkna medelvärdet av ett N × N-block av källpixlar till en visad pixel. `1` är den ursprungliga upplösningen.

* Det påverkar **helskärmsvyn, rutnätets miniatyrbilder, markörens avläsning och båda histogrammen** – allt som visar bilden har samma grundupplösning.
* Det gäller **endast visningen**. Bearbetning och export påverkas inte. Det enda undantaget är avsiktligt: en export via [Index/LUT Sandbox](index-lut-sandbox.md) sparar det du tittar på, så den behåller den aktuella blockstorleken, och exportpanelen varnar dig när blockstorleken överstiger 1.
* Värdet lagras **per projekt** som `viewer_display.gsd_bin` i `project.json`, så det bevaras även när programmet stängs och öppnas igen.
* Markörens avläsning visar blocket, inte källpixeln, när blockstorleken är större än 1 – det visade värdet är medelvärdet för blocket under markören.

{% hint style="info" %}
**Varför ”blockstorlek” och inte centimeter per pixel?** Ett värde i cm/px kräver en höjd över marken. EXIF-informationen för en enskild bildruta innehåller GPS-höjd över medelhavsnivån, inte över den terräng som kameran var riktad mot, så Chloros kommer inte att visa ett markavstånd som det inte kan beräkna på ett korrekt sätt. Blockstorleken i källpixlar är samma reservlösning som molnverktygen i MAPIR använder när markprovavståndet är okänt.
{% endhint %}

***

## Bildtyper du kan visa

Lagermenyn längst upp till höger i visningsprogrammet listar alla versioner av den aktuella bilden. Vilka poster som visas beror på kameran och på vad som har bearbetats — se [Bildlager](image-layers.md) för den fullständiga listan och hur menyn fungerar.

### Survey3

* **JPG** — kamerans egen förhandsgranskningsfil
* **RAW (Original)** — källfilen `.RAW`, avbayererad för visning, inga korrigeringar
* **RAW (Target)** — en bildruta som identifierats som innehållande ett kalibreringsmål
* **RAW (reflektans)** — den kalibrerade reflektansprodukten (65535 = ρ 1,0)
* **Vignettkorrigerad**/**Sensorsvar** — den okalibrerade reservprodukten
* **Vitbalanserad** — den vitbalanserade produkten
* **RAW (`<INDEX>`-index)**och**`<INDEX>` LUT** — beräknade indexbilder

### LATTICE

LATTICE-bilderna använder samma rullgardinsmeny, med namnen på pipeline-nivåerna:

| Lager                 | Vad det innehåller                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Original)**    | Den ursprungliga RAW-bilden som den tagits                                     |
| **RAW (Debayered)**   | Den linjära, debayered bilden                                           |
| **RAW (Förhandsgranskning)**     | Förhandsgranskningen på skärmen — falskfärgssträckning för multispektrala kameror |
| **Vitbalanserad**    | Förhandsvisningen för RGB-huvudkameror (vitbalans + gamma)   |
| **RAW (Strålning)**    | Float32 spektral strålning i W/m²/sr/nm                              |
| **RAW (Reflektans)** | uint16-reflektans, 32768 = ρ 1,0                                    |

Strålning och reflektans finns endast för multispektrala bilder: en RGB-huvudkamera har ingen radiometri per band, så dessa lager genereras inte för den.

***

## Tillämpning av index och LUT

Tillämpa multispektrala index och färg-LUT:er från sidofältet:

1. Öppna **Bildvisaren** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i sidofältet
2. Markera **Index**

3. Välj kamerans filter och en indexformel, och dra sedan kanalcirklarna till formelns fält
4. Lägg till en LUT och välj en gradient, tröskelvärden och ett klippningsläge
5. Läs av värdena vid markören och spara resultatet med **Export/Save Image(s)**Se [Index/LUT Sandbox](index-lut-sandbox.md) för en fullständig genomgång.***

## Felsökning

### Bilden går inte att öppna

**Möjliga orsaker**: filen har flyttats eller raderats efter importen; produkten har aldrig skrivits ut; otillräckligt minne för en mycket stor bild.**Åtgärder**:

1. Kontrollera att lagrets fil fortfarande finns i projektets utdatastruktur
2. Öppna filen i en extern bildvisare för att bekräfta att den är intakt
3. Stäng andra program för att frigöra minne

### Bilden är svart, vit eller har helt orimliga färger

**Möjliga orsaker**: bildsträckningen har inget att arbeta med (en nästan konstant bildram); ett float32-lager med ovanliga värden; ett index som inte genererade giltiga data.**Vad du ska göra**:

1. Läs av markörvärdena – om varje kanal ligger på eller nära noll ligger problemet i data, inte i visningen
2. Kontrollera histogrammet: en enskild topp i ena änden indikerar att ramen är beskuren eller tom
3. Kontrollera bearbetningsloggen för den körning som genererade lagret

### Värdena ser felaktiga ut

**Möjliga orsaker**: du befinner dig på ett annat lager än du tror; du jämför en procentandel med ett rått DN-värde; du jämför en LATTICE-fil med en Survey3-fil med samma divisor.**Vad du ska göra**:

1. Kontrollera vilket lager som är valt i rullgardinsmenyn – panelens enheter följer lagret
2. För reflektans, använd kolumnen **%** istället för att själv dividera DN-värdet; om du måste dividera, använd den filens `Chloros:PixelScale` (32768 för LATTICE, saknas betyder 65535 för Survey3)
3. Ställ tillbaka GSD-blockstorleken till 1 — över 1 läser du ett blockgenomsnitt, inte en pixel
4. Kontrollera att reflektanskalibreringen faktiskt kördes för den ramen; en okalibrerad reservprodukt (Sensor Response / Vignette Corrected) är inte reflektans

***

## Nästa steg

* [**Bildlager**](image-layers.md) — varje lagernamn, om det finns, och vad dess värden betyder
* [**Index/LUT-sandlåda**](index-lut-sandbox.md) — skapa, finjustera och exportera indexvisualiseringar
* [**Kartmarkörer**](map-markers.md) — samma bilduppsättning på en karta
* [**Multispektrala indexformler**](../project-settings/multispectral-index-formulas.md) — indexreferensen

För arbetsflödet för bearbetning, se [Bearbeta bilder (GUI)](../processing-images-gui/adding-files-to-a-project.md).
