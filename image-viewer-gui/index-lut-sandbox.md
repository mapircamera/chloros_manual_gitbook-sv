# Index/LUT-sandlåda

Index/LUT Sandbox är det interaktiva arbetsområdet i sidopanelen i Chloros Image Viewer. Du väljer en formel, kopplar kamerans kanaler till den, färglägger den med en toning och justerar värdeintervallet – och bilden uppdateras i realtid medan du gör det. Sedan version 1.2.0 kan du också **spara det du har skapat**, för en enskild bild eller för hela projektet, utan att behöva bearbeta om.

## Vad Sandboxen är till för

| Index/LUT Sandbox (interaktiv)        | Projektbearbetning (batch)       |
| -------------------------------------- | -------------------------------- |
| En bild i taget, omedelbar återkoppling  | Hela datauppsättningen i ett svep     |
| Experimentellt och iterativt             | Förkonfigurerade inställningar          |
| Renderar i realtid; sparar endast när du ber om det  | Skriver alltid ut produktfiler      |
| Perfekt för att hitta rätt inställningar | Bäst när inställningarna är slutgiltiga |

{% hint style="success" %}
**Det vanliga arbetsflödet**: finjustera i Sandbox tills visualiseringen ser ut som du vill, och exportera sedan antingen direkt från Sandbox eller kopiera samma index- och LUT-inställningar till [Projektinställningar](../project-settings/project-settings.md) så att nästa bearbetningsomgång bakar in dem i varje bild.
{% endhint %}

***

## Öppna Sandbox

1. Klicka på en bild i rutnätet — den öppnas i helskärmsläge i fliken **Bildvisare** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Klicka på ikonen **Bildvisare** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> för att dra ut vänster sidofält om det inte redan är öppet
3. Välj ett multibandslager från lagermenyn längst upp till höger — **RAW (Reflectance)** är det vanligaste valet, eftersom indexvärden som beräknas utifrån kalibrerad reflektans är jämförbara mellan olika bilder

Sidopanelen visar, uppifrån och ner:

* bildens namn och kameramodell
* knappen **Exportera/Spara bild(er)** — visas när Index eller LUT är markerat
* kryssrutorna **Index**och**LUT**
* panelen för indexkonfiguration
* panelen **Cursorvärden** med avläsning, histogram och GSD-kontroll

{% hint style="warning" %}
**Ej tillgängligt för monokameror.** På en LATTICE M3M-bild med ett enda band är båda kryssrutorna inaktiverade, med verktygstipset _”Ej tillgängligt för mono (M3M)-sensorer”_ — ett multibandindex är odefinierat på ett enda band. För att beräkna index från M3M-kameror ska du kombinera två eller flera till en justerad flerbandsstapel och använda LATTICE-indexmotorn.
{% endhint %}

***

## Tillämpa ett index

1. Markera rutan **Index** högst upp i sidofältet
2. Välj kamerans filter från rullgardinsmenyn till vänster (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Välj en indexformel från den högra rullgardinsmenyn – 27 inbyggda formler, plus eventuella anpassade formler som du har sparat
4. Formeln visas som matematisk formel nedan, med en tom cirkel vid varje bandplats. **Dra en färgad kanalcirkel till en plats** för att koppla den
5. När alla platser som formeln använder är kopplade uppdateras bilden och visar indexvärden
6. För muspekaren över bilden för att läsa av värden; panelen **Pekarvärden** lägger till en indexrad med värdet under muspekaren

Dubbelklicka på en kopplad plats för att rensa den. En ofullständig formel är ett normalt tillstånd under dragningen, inte ett fel – bilden uppdateras helt enkelt inte förrän formeln är komplett.

Kanalcirklarna är färgkodade: röd = Red, grön = Green, blå = Blue, orange = Orange, cyan = Cyan, lila = NIR, magenta = RE. Samma färger används för kanalpunkterna och histogramkurvorna i panelen Markörvärden.

### Exempel på NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

För den fullständiga formelreferensen – alla tre förinställningslistorna och vilka namn som fungerar var – se [Multispektrala indexformler](../project-settings/multispectral-index-formulas.md).

### Med ”Index” markerat men utan LUT

Bilden ritas i **gråskala**, utsträckt mellan de två tröskelvärdena. Detta är avsiktligt: indexbilden består av skalära data, och gråskala är den mest korrekta återgivningen av den. Lägg till en LUT när du vill ha färg.***

## Arbeta med LUT:er (uppslagstabeller)

En **uppslagstabell** mappar indexvärden till färger: mat in NDVI 0,65, få ut en viss grön nyans. Den ändrar inte data – den ändrar hur du tolkar den.

### Lägga till en LUT

1. Klicka på knappen **&quot;+ Lägg till LUT&quot;** under formeln <img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">
2. Välj en färggradient
3. Ställ in minimivärde och maximivärde för beskärningen
4. Välj ett beskärningsläge
5. Markera rutan **LUT** i sidofältet för att rendera den

LUT-rutan förblir inaktiverad tills en LUT faktiskt har konfigurerats i indexet.

### Välja en färggradient

Håll muspekaren över **gradientfältet**för att öppna listan med förinställningar — Chloros levereras med**sju** förinställda gradienter:

| # | Gradient                            | Form                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Gul → Green (**standard**)  | Divergerande — stämmer överens med den vanliga uppfattningen om vegetation, grönt = friskt |
| 2 | Lila → Gul → Green             | Divergerande, med en tydlig nedre del                                  |
| 3 | Brun → Vit → Blue                | Divergerande kring en ljus mittpunkt                                   |
| 4 | Svart → Lila → Rosa → Ljusgult | Sekventiell, mörkt till ljust                                           |
| 5 | Red → Gult → Blue                 | Avvikande kring en ljus mittpunkt                                   |
| 6 | Lila → Blue → Green → Gul      | Sekventiell, mörk till ljus                                           |
| 7 | Orange → Vit → Lila             | Divergerande runt en ljus mittpunkt                                   |

En **divergerande**gradient placerar en neutral färg i mitten av fönstret, vilket fungerar bra när mittpunkten har en specifik betydelse (ett tröskelvärde, ett basdatum). En**sekventiell** gradient löper monotont från mörkt till ljust, vilket fungerar bra för en kvantitet som endast har &quot;mer&quot; och &quot;mindre&quot;.

Varje förinställning har sju färgsteg. Klicka på en förinställning så uppdateras bilden omedelbart (när rutan för LUT är markerad).

### Redigera färgstegen

Under gradientfältet finns en rad med färgprover, ett per färgsteg:

* **Ändra en färg**: klicka på en färgprovbit för att öppna färgväljaren (färghjul, RGB/HSV-reglage eller en hexkod som `#FF0000`)
* **Lägg till ett steg**: klicka på**+**-knappen i slutet av raden – ett vitt steg läggs till
* **Ta bort ett steg**:**dubbelklicka** på färgprovet
* **Spara en redigerad gradient**: klicka på spara-ikonen bredvid gradientfältet för att lägga till din redigerade gradient i listan med förinställningar så att du kan välja den igen

Den gradient du har konfigurerat för ett index sparas tillsammans med det indexet i projektets inställningar, så den bevaras även när projektet stängs och öppnas igen.

**Färre steg**ger tydliga zoner som kan tolkas som en klassificering;**fler steg** ger mjuka, nästan fotografiska övergångar. Tre till fem steg passar presentationsbilder och klassificeringskartor; sex till tio passar allmän analys; femton eller fler passar detaljerad granskning och publikationsdiagram.

### Ställa in värdeintervallet

Tröskelreglaget är ett **reglage med två handtag**som sträcker sig från −1 till +1, med en redigerbar textruta i varje ände för exakta värden och en**AUTO**-knapp.

* Dra i något av handtagen, eller skriv in ett tal i rutan och tryck på Enter
* **AUTO**ställer in intervallet till den**2:a och 98:e percentilen** av bildens giltiga indexvärden – en bra utgångspunkt som bortser från extremvärden. Chloros avrundar resultatet adaptivt, till 4 decimaler för ett mycket snävt intervall, 3 för ett snävt intervall och 2 i övriga fall
* Alla manuella justeringar har företräde framför AUTO tills du trycker på AUTO igen

Exempel på NDVI-fönster:

| Mål                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Visa allt                                | −1,0 | 1,0 |
| Endast vegetation, exkludera mark och vatten | 0,2  | 0,9 |
| Endast frisk vegetation                 | 0,5  | 0,9 |
| Betona stress                        | 0,2  | 0,5 |

Genom att begränsa fönstret ökar kontrasten inom ditt intresseområde och allt annat hamnar utanför intervallet — där **klippningsläget** avgör vad som händer med det.***

## Klippningslägen

När en pixels indexvärde faller utanför min/max-fönstret avgör klippningsläget hur den ska ritas.

| Rullgardinsmeny                  | Lagrad värde      | Pixlar utanför intervallet ritas som                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum &amp; Maximum** (standard) | `clip`            | Den närmaste ändfärgen i gradienten — värden under minimum får den första färgen, värden över maximum får den sista |
| **Transparent bakgrund**      | `transparent`     | Helt transparent (verklig alfa)                                                                                                  |
| **Indexbakgrund**| `indexColor`      | Gråskala, utsträckt över bildens**hela** indexintervall, så att strukturer utanför intervallet fortfarande syns i grått                |
| **Originalbakgrund**         | `backgroundColor` | Själva den underliggande bilden, så att färgöverlägget ligger ovanpå den verkliga scenen                                                |

| Läge                       | Bäst för                               | Utseende                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimum &amp; Maximum**      | Fullständig datavisning, vetenskaplig analys | Varje pixel färglagd                      |
| **Transparent bakgrund** | GIS-överlägg, isolering av ett värdeintervall   | Färg inuti fönstret, inget utanför |
| **Indexbakgrund**       | Betoningen bevaras samtidigt som datakontexten bibehålls    | Färg inuti, grått utanför               |
| **Originalbakgrund**    | Rapporter och presentationer              | Färg inuti, fotografi utanför         |

{% hint style="info" %}
**Pixlar utan data är alltid transparenta, i alla lägen.** En pixel vars index inte är ändligt (en division med 0/0) eller är exakt −1,0 eller +1,0 (mättnadsvärden, där det ena bandet visar noll medan det andra inte gör det) behandlas som utan data snarare än som ett extremvärde. Detta håller utbrända högdagrar och döda skuggor borta från din färgskala istället för att återge dem som de mest extrema värdena i bilden. Samma regel avgör vilka pixlar som matas in i AUTO-tröskelvärdena och indexhistogrammet, så att alla tre stämmer överens.
{% endhint %}

Transparensen bevaras när exporten sparas som PNG. Den kan inte återges i JPG.

***

## Avläsa värden medan du finjusterar

Panelen **Cursor Values** under konfigurationspanelen fungerar som mätinstrument för Sandbox:

* För markören över bilden och avläs källvärdena per kanal, samt indexvärdet i sin egen rad
* Aktivera **INDEX**-knappen ovanför histogrammet för att se fördelningen av indexvärden i bilden, där dina två klipptrösklar visas som orange streckade linjer och markörens värde som en vit linje — detta är det snabbaste sättet att välja ett fönster som faktiskt innehåller dina data
* Aktivera **CURSOR** för att se markeringslinjer vid värdena under pekaren
* Zooma in mer än 60× (mindre om en GSD-blockstorlek är inställd) för att markera enskilda visade pixlar med ett flytande värde

En praktisk rutin:

1. Notera värdena över frisk vegetation, stressad vegetation, bar mark och vatten
2. Titta på var dessa kluster befinner sig på indexhistogrammet
3. Ställ in min/max för att avgränsa det kluster du är intresserad av
4. Välj ett beskärningsläge — _Original Background_ behåller scenen synlig runt omkring

***

## Exportera från Sandbox

Allt ovan är en liveförhandsvisning tills du sparar det. Knappen **Exportera/spara bild(er)** högst upp i sidofältet öppnar ett fönster som glider över sidofältet (istället för att täcka bilden, så att du fortfarande kan se vad du fattar beslut om).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Alternativ

| Alternativ                          | Effekt                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Tillämpa på aktuell bild**      | Sparar exakt den bild som visas, med dessa inställningar                                                                                                |
| **Tillämpa på alla projektbilder** | Kör om samma konfiguration på varje bild i projektet. Bilder utan de band som detta index behöver hoppas över och behandlas inte som fel |
| **Index-/LUT-gradientfält**      | Skriver även ut en separat bild med förklarande text per export, där värdeintervallet är märkt                                                                     |
| **Indexhistogram**             | Skriver även ut en separat histogrambild per export, som visar dataens min-/maxvärden och klipptrösklarna                                               |

Om **GSD-blockstorleken** på bildfliken är högre än 1, visas detta i fönstret innan du bekräftar: exporten sparar det du ser, inklusive blockmedelvärde. Ställ först tillbaka GSD-reglaget till 1 om du vill ha full upplösning.

### Var filerna sparas

Varje klick på **Exportera**skapar en**ny mapp som aldrig återanvänds**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Exempel: `Sandbox_Exports/NDVI_LUT_001/`, sedan `Sandbox_Exports/NDVI_LUT_002/` för nästa körning. Numreringen genereras genom att skanna vad som redan finns på disken, så den bevaras även vid omstarter och mappar som du tar bort manuellt. Ingenting skrivs någonsin över – hela poängen med Sandboxen är att jämföra ett försök med det senaste.

Innehåll i mappen, per bild:

| Fil                                                   | Innehåll                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Den renderade bilden, pixel för pixel precis som den visades i visningsprogrammet |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Sidfilen med gradientfältet, om så begärs                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Sidfilen med indexhistogrammet, om så begärs                  |

De två sidobilderna skrivs alltid ut i **full upplösning**, även när huvudbilden är blockgenomsnittsberäknad: en blockstorlek motsvarar en skärmupplösning, och båda sidobilderna visar de verkliga indexvärdena per pixel. De visar också mer information än versionerna på skärmen — båda markerar sträckningsfönstret _och_ de verkliga min-/maxvärdena för data, så att en sparad legend fortfarande är läsbar månader senare utan att projektet är öppet.

### Förlopp och resultat

En export av hela projektet tar några minuter, så körningen rapporterar tillbaka via en live-kanal för förloppet istället för att blockera:

* En förloppsindikator visar `current / total` och den fil som skrivs
* När den är klar visar fönstret hur många bilder som exporterades, hur många som hoppades över och sökvägen till utmatningsmappen
* Bilder som hoppats över listas med orsaken (upp till fem visas, därefter en rad med ”+N fler”). Den vanligaste orsaken är ett lager som saknar de kanaler som detta index behöver
* Om **ingen** bild i projektet kan använda indexet rapporterar körningen ett fel istället för att lämna en tom mapp

Endast en sandbox-export får köras åt gången. Att starta en andra medan en pågår nekas med ett tydligt meddelande istället för att låta två körningar konkurrera om samma projektfil.

### Rutnätet hämtar körningen

Varje avslutad körning visas som en egen knapp i verktygsfältet [bildrutnät](image-grid.md), märkt `<IndexName> <Index|LUT> <NNN>`. Så här jämför du körningar: exportera två gånger med olika gradienter eller tröskelvärden, och växla sedan mellan de två knapparna i rutnätet.

***

## Anpassade indexformler (Chloros+)

{% hint style="info" %}
**Var man skapar dem**: i sidopanelen i Sandbox eller i**Projektinställningar** före bearbetning. Båda skriver till samma lista på projektnivå.
{% endhint %}

1. Öppna kalkylatorn för anpassade formler från rullgardinsmenyn för indexformler (kräver inloggning med ett giltigt Chloros+-abonnemang)
2. Skriv formeln med hjälp av **band-slot-symbolerna** `x`, `y`, `z`, `a`, `b`, `c` — inte bandnamn
3. Tillgängliga operatörer: `+`, `-`, `*`, `/`, `^` och `()` för gruppering
4. Tillgängliga funktioner: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Namnge och spara den – den visas längst ner i formelmenyn och du kopplar dess platser genom att dra kanalcirklar, precis som med en inbyggd förinställning

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Anpassade formler finns endast i GUI.** Alternativet CLI/SDK `--indices` utökar de 22 inbyggda förinställningsnamnen och hoppar tyst över allt annat, inklusive dina anpassade formler. För att bearbeta en anpassad formel i batch, konfigurera den i Projektinställningar och kör bearbetningen, eller använd exportfunktionen ”Tillämpa på alla projektbilder” i Sandbox.
{% endhint %}

***

## Felsökning

### ”Det här lagret har inte de kanaler som detta index kräver”

Formeln läser av en kanalposition som det aktuella lagret inte har — till exempel ett index med tre platser på en fil med en eller två kanaler. Byt till ett flerbandslager (reflektans eller debayered), eller välj ett index som passar din kameras filter.

### ”Kunde inte nå bildbehandlingsbackenden”

Backenden svarar inte. Kontrollera fliken Logg; om backend-modulen startar om återställs Sandbox automatiskt så snart den är tillbaka.

### Bilden förändrades inte när jag drog en cirkel

Formeln är ännu inte komplett. En ofullständig formel behandlas som ett normalt tillstånd mitt i en dragning – ingenting renderas och inget rapporteras som ett fel. Fyll i alla fält som formeln använder.

### Hela bilden är enfärgad

Ditt klippfönster ligger troligen långt utanför dataområdet. Tryck på **AUTO**för att anpassa det till den 2:a eller 98:e percentilen, eller aktivera**INDEX**-histogrammet för att se var data faktiskt ligger.

### De exporterade färgerna stämmer inte överens med vad jag såg

Det borde de göra – exportvägen är en avsiktlig spegling av liveförhandsvisningen, inklusive alfa i klippningsläget, och blockmedelvärdesberäkningen tillämpas _efter_ färgläggningen, precis som visningsprogrammet gör. Om de skiljer sig åt, kontrollera att GSD-blockstorleken inte har ändrats mellan visning och export.

***

## Nästa steg

* [**Bildlager**](image-layers.md) — vilket lager indexet ska köras på, och vad dess värden betyder
* [**Öppna en bild i helskärmsläge**](opening-an-image-full-screen.md) — detaljerad beskrivning av marköravläsning, histogram och GSD-kontroll
* [**Formler för multispektrala index**](../project-settings/multispectral-index-formulas.md) — alla förinställningar, på alla ytor
* [**Projektinställningar**](../project-settings/project-settings.md) — att spara de inställningar du har hittat i en bearbetningskörning
