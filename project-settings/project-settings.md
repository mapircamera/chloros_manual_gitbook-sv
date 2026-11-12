# Projektinställningar

I sidopanelen ”Projektinställningar” <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

i Chloros kan du konfigurera alla aspekter av bildbearbetning, detektering av kalibreringsmål, beräkningar av multispektrala index samt exportalternativ för ditt projekt. Dessa inställningar sparas tillsammans med ditt projekt och kan sparas som mallar för återanvändning i flera projekt.

## Öppna projektinställningarna

Så här öppnar du projektinställningarna:

1. Öppna ett projekt i Chloros


2. Klicka på fliken **Projektinställningar** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

i vänster sidopanel
3. Inställningspanelen visar alla tillgängliga konfigurationsalternativ sorterade efter kategori

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->



{% hint style="info" %}
**Inställningar som är beroende av andra inställningar är gråmarkerade.** När en överordnad inställning gör en annan inställning omöjlig (till exempel om du avmarkerar *Reflektanskalibrering / vitbalans* blir *Exportera reflektans* omöjligt), inaktiveras den beroende kontrollen och dess verktygstips anger vilken inställning som måste ändras.
{% endhint %}

***

## Visning

### Upplösning för bildminiatyrer

* **Typ**: Rullgardinsmeny
* **Alternativ**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Standard**: Standard (512 px)
* **Beskrivning**: Upplösning (längsta sidan, i pixlar) vid vilken miniatyrerna i bildrutnätet renderas. Högre värden ser skarpare ut när man zoomar in, men laddas långsammare och använder mer minne. Full upplösning motsvarar den ursprungliga bildstorleken.
* **Obs**: Endast för visning — detta påverkar aldrig bearbetningen eller exporterade filer.

***

## Måldetektering

Dessa inställningar styr hur Chloros e upptäcker och bearbetar kalibreringsmål i dina bilder. Båda är endast aktiva när **Reflektanskalibrering / vitbalans** är aktiverat (annars är de gråmarkerade, eftersom måldetekteringen hoppas över helt).

### Minsta kalibreringsprovområde (px)

* **Typ**: Tal
* **Intervall**: 0 till 10 000 pixlar
* **Standard**: 25 pixlar
* **Beskrivning**: Ställer in den minsta ytan (i pixlar) som krävs för att ett upptäckt område ska betraktas som ett giltigt kalibreringsmålprov. Lägre värden upptäcker mindre mål men kan öka antalet falska positiva resultat. Högre värden kräver större, tydligare målområden för detektering.
* **När ska du justera**:
  * Öka värdet om du får falska detekteringar på små bildartefakter
  * Minska värdet om dina kalibreringsmål verkar små i dina bilder och inte detekteras

### Minsta målkluster (0–100)

* **Typ**: Tal
* **Intervall**: 0 till 100
* **Standardvärde**: 60
* **Beskrivning**: Styr tröskelvärdet för klustring vid gruppering av områden med liknande färg vid detektering av kalibreringsmål. Högre värden kräver att fler liknande färger grupperas tillsammans, vilket resulterar i en mer konservativ måldetektering. Lägre värden tillåter större färgvariation inom en målgrupp.
* **När ska du justera**:
  * Öka värdet om kalibreringsmål delas upp i flera detekteringar
  * Minska värdet om kalibreringsmål med färgvariationer inte detekteras fullständigt

***

## Bearbetning

Dessa inställningar styr hur Chloros bearbetar och kalibrerar dina bilder.

### Vignetteringskorrigering

* **Typ**: Kryssruta
* **Standard**: Aktiverad (markerad)
* **Beskrivning**: Tillämpar vignetteringskorrigering för att kompensera för mörkare kanter på bilderna. Vignettering är ett vanligt optiskt fenomen där hörn och kanter på en bild framstår som mörkare än mitten på grund av objektivets egenskaper.
* **Bieffekt**: Denna knapp väljer också vilken *okalibrerad reservprodukt* som en körning skriver ut (se nedan).

### Reflektanskalibrering / vitbalans

* **Typ**: Kryssruta
* **Standard**: Aktiverad (markerad)
* **Beskrivning**: Aktiverar reflektanskalibrering — utifrån detekterade kalibreringsmål inom bildrutan och/eller nedåtriktade data från DAQ-ljussensorn, beroende på kamera och vad som är tillgängligt. Detta normaliserar reflektansvärdena i hela din datamängd och säkerställer konsekventa mätningar oavsett ljusförhållanden.
* **När funktionen är inaktiverad**: Måligenkänningen hoppas över helt, och**ingen reflektansprodukt kan genereras av någon kamera** – varken de målstyrda i Survey3 eller de DAQ-styrda i LATTICE. De beroende inställningarna (*Exportera reflektans*, *Minsta omkalibreringsintervall* och tröskelvärdena för måligenkänning) är gråmarkerade.

### Okalibrerade reservprodukter: Exportera sensorns respons / Exportera vignettkorrigerad

* **Typ**: Två kryssrutor
* **Standardinställningar**: Båda aktiverade (markerade)
* **Beskrivning**: När en bildruta inte kan kalibreras med avseende på reflektans (inget kalibreringsmål hittades eller reflektanskalibreringen är avstängd) skrivs den istället ut som en *okalibrerad reservprodukt*. **Exakt en av de två reservprodukterna finns per körning, för varje kameramodell**, vald via *Vignettkorrigering*-omkopplaren:
  * Vignettkorrigering **på**→ `Vignette_Corrected_Images/` (styrs av**Exportera vignettkorrigerat**)
  * Vignettkorrigering **av**→ `Sensor_Response_Images/` (styrs av**Exportera sensorns respons**)
* Den reservprodukt som inte används är gråmarkerad. Om du avmarkerar den som används förhindras att den filen skrivs ut överhuvudtaget.

### LATTICE-exportprodukter

För projekt som innehåller LATTICE-bildtagningar fördelas varje importerad LATTICE-bildram till alla aktiverade **och tillämpliga**produkter i ett enda bearbetningspass. Fyra kryssrutor styr fördelningen (alla är som standard**på**):

| Inställning | Utmatningsmapp | Vad den exporterar |
| --- | --- | --- |
| **Exportera debayered** | `Debayered_Images/` | Den linjära debayered-bilden: kalibreringspaketets mörka offset har tagits bort, följt av en kantmedveten demosaikering, utan linearitetskorrigering, flat field eller unmix — samma fil som en bild sparar på sin debayered-nivå. Bevaras i sensorns ursprungliga orientering (registrerad i XMP). Gäller för RGB- och multispektrala kameror. |
| **Exportera förhandsgranskning** | `Preview_Images/` | Förhandsgranskningen på skärmen. RGB = vitbalans (DAQ-ljuskälla om tillgänglig, annars gråskala) + gamma; multispektral = falskfärgskorrigering. |
| **Exportera strålning** | `Radiance_Images/` | Spektral strålning i Float32, enhet W/m²/sr/nm. Endast multispektralt (M3C/M3M) – gäller inte för RGB-originalbilder. Skrivs alltid som 32-bitars TIFF oavsett inställningen för *Kalibrerat bildformat*. |
| **Exportera reflektans**| `Reflectance_Calibrated_Images/` | Uint16-reflektans, skalad så att**32768 = reflektans 1,0** (stämplad som XMP `Chloros:PixelScale`). Endast multispektralt, skrivs ut när en matchande `.daq`-nedåtriktad post (eller ett QA-godkänt mål inom bilden) täcker bilden. |

* RGB-huvudkameror avger debayered + förhandsgranskning; strålning/reflektans hoppas över för dessa eftersom de inte är tillämpliga.
* Bitdjupet för debayered/förhandsgranskning följer inställningen *Kalibrerat bildformat*; strålning är alltid float32.
* Survey3-bearbetning påverkas inte av dessa fyra växlar.
* Radiance- och reflektansprodukter från multispektrala M3C-kameror kan också skriva en **klämmask** per pixel i en `clamp_masks/`-mapp bredvid produktmapparna, där den registrerar var bandavblandningen klippte ett negativt värde till noll. Se [Utgångsbildformat](../output-image-formats.md#clamp-masks).

Samma fyra växlingsalternativ finns i headless-läge som `chloros-cli process --debayered / --preview / --radiance / --reflectance`. De ersatte den gamla CLI-flaggan `--radiometric-output`, som inte längre finns.

{% hint style="warning" %}
**Om alla tillämpliga produkter stängs av misslyckas körningen.** Från och med version 1.2.0 rapporterar en bearbetningskörning som begärde produkter men inte skrev någon bildprodukt ett fel, och CLI-programmet avslutas med ett värde som inte är noll, istället för att rapportera en tyst framgång. Loggen anger vilken produkt som inte kunde skrivas och varför. En avsiktlig körning med endast metadata (inget begärt) betraktas fortfarande som lyckad.
{% endhint %}

### Reflektanskälla (projektinställning, ställs in via CLI)

Projektet lagrar även vilken **reflektansreferens** som LATTICE-reflektansprodukten använder. Det finns ingen särskild kontroll i inställningspanelen; värdet lagras i projektkonfigurationen som `Processing → "Target reflectance source"` och ställs in med `chloros-cli process --reflectance-source {auto,target,daq}` (SDK:s `configure()` har inget nyckelord för detta):

* **`auto`** (standard): ett kalibreringsmål inom bildrutan som klarar kvalitetskontrollen (QA) blir den absoluta referensen, med fallback till DAQ:s nedåtriktade delningsvärde (ρ = πL/E) när inget mål finns eller om kvalitetskontrollen misslyckas. Ett mål som klarar kvalitetskontrollen med en flagga (en panel som fallit bort eller ett band som inte fungerar) prioriteras inte automatiskt: kedjorna mål, mål+DAQ och enbart DAQ testas i ordning efter deras uppmätta poäng på uteslutna paneler.
* **`target`**: strikt målstyrd reflektans — ingen DAQ-ersättning.
* **`daq`**: DAQ-avgörande reflektans; mål inom ramen används inte som referens.

Det lagrade värdet matchas utan hänsyn till versaler och gemener, och några stavningar accepteras som alias: `target`, `target_image`, `empirical` och `empirical_line` betyder alla **mål**; `daq`, `dls`, `light_sensor` och `sensor` betyder alla**daq**. Allt annat – inklusive en saknad nyckel – tolkas som**auto**.**Uppmätta** målsökningar per enhet söks upp utifrån målenhetens serienummer/QR, som `<serial>.csv`, på tre ställen: i den katalog som anges med `--target-reflectance-dir` (lagrad som `Processing → "Target reflectance dir"`), i projektets egen `target_reflectance/`-mapp, samt sökvägen i miljövariabeln `CHLOROS_TARGET_REFLECTANCE_DIR`. Om det inte finns någon mätt skanning för den enheten används istället den nominella publicerade kurvan för målmodellen.

### Debayer-metod

* **Typ**: Rullgardinsmeny
* **Alternativ**:
  * Standard (snabb, medelhög kvalitet)
  * Texturmedveten (långsam, högsta kvalitet) \[Chloros+]
* **Standard**: Standard (snabb, medelhög kvalitet)
* **Beskrivning**: Väljer den demosaicing-algoritm som används för att omvandla rådata från sensorer med Bayer-mönster till fullfärgsbilder. Metoden ”Standard (snabb, medelhög kvalitet)” ger en optimal balans mellan bearbetningshastighet och bildkvalitet. Metoden ”Texturmedveten (långsam, högsta kvalitet)” \[Chloros+] använder en högkvalitativ kantmedveten debayer i kombination med en AI/ML-brusreduceringsmodell som tar bort nästan allt debayeringbrus. Modellen ”Texture Aware” kräver GPU-minne (VRAM) för att köras. Vi rekommenderar att du använder den när du har >4 GB VRAM tillgängligt för snabbare bearbetning.
* **När raden överhuvudtaget är en rullgardinsmeny**: rullgardinsmenyn med två alternativ visas endast när**båda** villkoren är uppfyllda — du är inloggad med ett giltigt Chloros+-abonnemang, **och** projektet innehåller inga LATTICE-inspelningar. I annat fall visas raden som ren text med texten `Standard (Fast, Medium Quality)` utan några valalternativ.
* **LATTICE-anmärkning**: Det finns ingen LATTICE-tränad Texture Aware-modell, och bearbetningskedjan tvingar fram standarddemosaikering för LATTICE-bilder oavsett det lagrade värdet. Om du lägger till en LATTICE-mapp i ett projekt där Texture Aware redan var valt, återställer Chloros inställningen till Standard istället för att behålla ett inaktuellt värde i `project.json`.

### Minsta omkalibreringsintervall

* **Typ**: Tal
* **Intervall**: 0 till 3 600 sekunder
* **Standardvärde**: 0 sekunder
* **Beskrivning**: Ställer in det minsta tidsintervallet (i sekunder) mellan användning av kalibreringsmål. När värdet är 0 kommer Chloros att använda varje upptäckt kalibreringsmål. När värdet är högre kommer Chloros endast att använda kalibreringsmål som ligger minst detta antal sekunder från varandra, vilket minskar bearbetningstiden för datamängder med frekventa registreringar av kalibreringsmål.
* **När ska inställningen justeras**:
  * Ställ in på 0 för maximal kalibreringsnoggrannhet när ljusförhållandena varierar
  * Öka värdet (t.ex. till 60–300 sekunder) för snabbare bearbetning när ljusförhållandena är konstanta och du har frekventa bilder av kalibreringsmål

### Tidszonsavvikelse för ljussensor

* **Typ**: Tal
* **Intervall**: -12 till +12 timmar
* **Standard**: 0 timmar
* **Beskrivning**: Anger tidszonsförskjutningen (i timmar från UTC) för tidsstämplar i ljussensordata, vilket används vid matchning av ljussensorloggar mot bildtagningstidpunkter. Nyare `.daq`-inspelningar har sin egen tidszonsinformation, så detta behövs främst för äldre loggar som spelats in i lokal tid.

### Tillämpa PPK-korrigeringar

* **Typ**: Kryssruta
* **Standard**: Inaktiverad (avmarkerad)
* **Beskrivning**: Aktiverar användningen av PPK-korrigeringar (Post-Processed Kinematic) från MAPIR DAQ-inspelare som innehåller GPS (GNSS). När detta är aktiverat kommer Chloros att använda alla .daq-loggfiler som innehåller exponeringsvärden i din projektkatalog och tillämpa exakta geolokaliseringskorrigeringar på dina bilder.
* **Krav**: En .daq-loggfil med exponeringstapp-poster måste finnas i din projektkatalog
* **När ska funktionen aktiveras**: Det rekommenderas att alltid aktivera PPK-korrigering om du har exponeringstapp-poster i din .daq-loggfil.

### Exponeringsstift 1

* **Typ**: Rullgardinsmeny
* **Synlighet**: Synlig endast när ”Tillämpa PPK-korrigeringar” är aktiverat OCH exponeringsdata finns tillgängliga för stift 1
* **Alternativ**:
  * Kameramodellnamn som upptäckts i projektet
  * ”Använd inte” – Ignorera detta exponeringsstift
* **Standard**: Väljs automatiskt baserat på projektkonfigurationen
* **Beskrivning**: Tilldelar en specifik kamera till exponeringsstift 1 för PPK-tidssynkronisering. Exponeringsstiftet registrerar den exakta tidpunkten då kamerans slutare utlöses, vilket är avgörande för korrekt PPK-geolokalisering.
* **Beteende vid automatisk val**:
  * En kamera + en stift: Kameran väljs automatiskt
  * En kamera + två stift: Stift 1 tilldelas automatiskt till kameran
  * Flera kameror: Manuellt val krävs

### Exponeringsstift 2

* **Typ**: Rullgardinsmeny
* **Synlighet**: Synlig endast när ”Tillämpa PPK-korrigeringar” är aktiverat OCH exponeringsdata finns tillgängliga för stift 2
* **Alternativ**:
  * Kameramodellnamn som har identifierats i projektet
  * ”Använd inte” – Ignorera denna exponeringspinne
* **Standard**: Väljs automatiskt utifrån projektkonfigurationen
* **Beskrivning**: Tilldelar en specifik kamera till exponeringspinne 2 för PPK-tidssynkronisering vid användning av en konfiguration med två kameror.
* **Beteende vid automatiskt val**:
  * En kamera + en stift: Stift 2 ställs automatiskt in på ”Använd inte”
  * En kamera + två stift: Stift 2 ställs automatiskt in på ”Använd inte”
  * Flera kameror: Manuellt val krävs
* **Obs**: Samma kamera kan inte tilldelas både stift 1 och stift 2 samtidigt.

***

## DAQ-ljussensor

Detta avsnitt visas i Projektinställningar och listar alla DAQ-nedåtriktade filer i projektet — `.daq`-inspelningar och DAQ-M `.csv`-loggar. Inspelningar som gjorts på fliken Ljussensorer läggs automatiskt till i det öppna projektet.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Varje rad visar filen, sensormodellen och den diffusorkåpskorrigering som faktiskt gäller för den filen. Ovanför raderna finns en enda projektomfattande kontroll:

### Överskrivning av kåp (alla filer)

* **Typ**: Rullgardinsmeny
* **Alternativ**: `Auto` samt de diffusionskåpskorrigeringsprofiler som gäller för de sensortyper som finns i projektet
* **Standard**: Auto
* **Lagras som**: `Processing → "DAQ cap id"` (standard `auto`)
* **Beskrivning**: `Auto` använder den registrerade kåpan för varje fil (Sunshine-kåpan antas om inget har registrerats, men DAQ-U, DAQ-M och DAQ-E levereras utan kåpa och Chloros kan inte se vilken som är monterad, därför varnar bearbetningen för ett antaget lock: välj det lock som var monterat). Att välja ett specifikt lock åsidosätter **alla** nedåtriktade filer i projektet utom en DAQ-E-S:s, vars diffusor ingår i dess kalibrering; ett projekt som endast består av DAQ-E-S visar ingen rullgardinsmeny: råa inspelningar korrigeras med den, och inspelningar som redan har ett lock omrefereras (den inspelade korrigeringen ångras och den valda tillämpas).
* **Viktigt**: Det valda locket måste stämma överens med det lock som fysiskt var monterat under inspelningen. Varken sensorn eller programvaran kan upptäcka det fysiska locket – ett felaktigt lock-ID korrigerar spektren felaktigt.

Det finns avsiktligt **en** projektomfattande inställning istället för rullgardinsmenyer per fil: inställningen gäller för alla nedåtriktade källor i projektet.

***

## Arrayjustering

Detta avsnitt visas **endast** när minst en bild i projektet innehåller den modul-till-modul-justeringstransformation som LATTICE-matriser stämplar vid bildtagningstillfället (XMP-taggarna `Chloros:Alignment*`). Här visas hur många bilder som har justeringstaggar, vilken kamera som är referenskamera (`REF`-märket) samt en tabell över antalet bilder per kamera.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Tillämpa arrayjustering

* **Typ**: Kryssruta
* **Standard**: Aktiverad (markerad)
* **Lagras som**: `Processing → "Array alignment"`
* **Beskrivning**: Förvränger varje bearbetad produkt (debayered / förhandsgranskning / strålning / reflektans / index) till matrisens gemensamma referensgeometri med hjälp av den transformation som stämplades vid tagningen. Av = exportera i ursprunglig geometri per sensor.

### Beskär till gemensam överlappning

* **Typ**: Kryssruta (endast aktiv när *Tillämpa matrisjustering* är aktiverat)
* **Standard**: Aktiverat (markerat)
* **Lagras som**: `Processing → "Array alignment crop"`
* **Beskrivning**: Beskär justerade exporter till det område som alla kameramoduler delar, så att varje band har samma täckningsområde. Av behåller hela sensorns yta (svart fyllning utanför källan).

### Omprovtagning

* **Typ**: Rullgardinsmeny (endast aktiv när *Tillämpa matrisjustering* är aktiverat)
* **Alternativ**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Standard**: Bilineär
* **Lagras som**: `Processing → "Array alignment interpolation"`
* **Beskrivning**: Interpolering som används vid justeringsförvrängningen. *Närmaste* bevarar exakta källvärden (ingen blandning mellan pixlar) för strikt radiometrisk analys; *Bilineär* är bäst för kartläggning och visuell användning.

Samma tre alternativ finns utan prefix som `chloros-cli process --array-alignment`, `--array-alignment-crop` och `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Index

Med dessa inställningar kan du konfigurera multispektrala index för analys och visualisering.

### Lägg till index

* **Typ**: Panel för specialkonfiguration av index
* **Beskrivning**: Öppnar en interaktiv panel där du kan välja och konfigurera multispektrala vegetationsindex (NDVI, NDRE, EVI osv.) som ska beräknas under bildbearbetningen. Du kan lägga till flera index, vart och ett med sina egna visualiseringsinställningar.
* **Tillgängliga index**: Rullgardinsmenyn i grafiskt gränssnitt innehåller**27** fördefinierade multispektrala indexformler (se [Multispektrala indexformler](multispectral-index-formulas.md) för den fullständiga listan, inklusive vilka namn som även accepteras av CLI/SDK `--indices`-alternativet).
* **Funktioner**:
  * Välj bland fördefinierade indexformler
  * Dra kamerans filterkanaler till formelns bandplatser
  * Konfigurera färgövergångar för visualisering (LUT – Look-Up Tables)
  * Ställ in tröskelvärden och beskärningslägen
  * Skapa anpassade indexformler
* **Obs**: Index beräknas inte för enkelbands-mono-kameror av typen LATTICE M3M — flerbandsindex är odefinierade på ett band. Survey3 och LATTICE M3C påverkas inte.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Varje index du lägger till renderar sin formel som matematik, med en färgad cirkel per bandplats: röd = Red, grön = Green, blå = Blue, orange = Orange, cyan = Cyan, lila = NIR, magenta = RE. Dra en cirkel från raden ovanför formeln till en plats för att koppla den; dubbelklicka på en kopplad plats för att rensa den. Indexet beräknas endast en gång per plats där formeln använder en kanal.

### Anpassade formler (Chloros+-funktion)

* **Typ**: Array med definitioner av anpassade formler
* **Tillgänglighet**: Kräver inloggning med ett giltigt Chloros+-abonnemang.
* **Beskrivning**: Gör det möjligt att skapa och spara anpassade multispektrala indexformler med hjälp av bandmatematik. Anpassade formler sparas tillsammans med dina projektinställningar och kan användas precis som inbyggda index.
* **Så här skapar du**:
  1. Öppna kalkylatorn för anpassade formler i konfigurationspanelen för index
  2. Skriv formeln med hjälp av **band-slot-symbolerna**, inte bandnamn
  3. Spara formeln med ett beskrivande namn – den visas då längst ner i formelmenyn, och du drar kamerans kanalcirklar till dess platser precis som med en inbyggd förinställning
* **Formelsyntax**:
  * Bandplatser: `x`, `y`, `z`, `a`, `b`, `c` — sex positioner som du mappar till verkliga kanaler genom att dra
  * Operatörer: `+`, `-`, `*`, `/`, `^` och `()` för gruppering
  * Funktioner: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Varför symboler istället för bandnamn**: en formel skriven som `(y-x)/(y+x)` fungerar på vilken kamera som helst, eftersom drag-och-släpp-mappningen avgör om `y` är 850 nm NIR för ett RGN-filter eller 808 nm NIR för ett OCN-filter. De inbyggda förinställningarna lagras på samma sätt — se [Formler för multispektrala index](multispectral-index-formulas.md) för den exakta symbolformen för alla 27.
* **Var de fungerar**: anpassade formler sparas tillsammans med projektinställningarna och kan användas både i [Index/LUT-sandlådan](../image-viewer-gui/index-lut-sandbox.md) och vid bearbetning. De accepteras**inte** av CLI/SDK-namnlistan `--indices`, som endast utökar de 22 inbyggda förinställningsnamnen.

***

## Export

Dessa inställningar styr formatet och kvaliteten på exporterade bearbetade bilder.

### Kalibrerat bildformat

* **Typ**: Rullgardinsmeny
* **Alternativ**:
  * **TIFF (16-bit)** – Okomprimerat 16-bitars TIFF-format
  * **TIFF (8-bit)** – Okomprimerat 8-bitars TIFF-format
  * **TIFF (32-bit, procent)** – 32-bitars TIFF med flyttalsvärden där reflektansvärdena anges i procent
  * **PNG (8-bit)** – Komprimerat 8-bitars PNG-format
  * **JPG (8-bit)** – Komprimerat 8-bitars JPEG-format
* **Standard**: TIFF (16-bit); ett projekt som helt består av 8-bitarsbilder visar TIFF (8-bit) och erbjuder endast 8-bitarsformaten
* **Beskrivning**: Väljer filformat för att spara bearbetade och kalibrerade bilder. De exporterade filerna placeras i en undermapp för respektive format inuti varje kameras mapp (`tiff16`, `tiff32`, `png8`, `jpg8`), med en `<Product>_Images/`-mapp per produkt. Exporterade filer behåller källfilnamnet – det är mappen, inte filnamnstillägget, som identifierar produkten.
* **Formatrekommendationer**:
  * **TIFF (16-bitars)**: Rekommenderas för vetenskaplig analys och professionella arbetsflöden. Bevarar maximal datakvalitet utan komprimeringsartefakter. Bäst för multispektral analys och vidare bearbetning i GIS-programvara.
  * **TIFF (32-bitars, procent)**: Bäst för arbetsflöden som kräver reflektansvärden uttryckta i procent (0–100 %). Ger maximal precision för radiometriska mätningar.
  * **PNG (8-bit)**: Bra för visning på webben och allmän visualisering. Mindre filstorlekar med förlustfri komprimering, men reducerat dynamiskt omfång.
  * **JPG (8-bit)**: Minsta filstorlekar, bäst endast för förhandsgranskning och visning på webben. Använder förlustbefläckad komprimering som inte är lämplig för vetenskaplig analys.
* **Obs**: LATTICE-radiansexporteras alltid som 32-bitars float-TIFF, oavsett denna inställning.

***

## Spara projektmall

Med den här funktionen kan du spara dina aktuella projektinställningar som en återanvändbar mall.

* **Typ**: Textinmatning + Spara-knapp
* **Beskrivning**: Ange ett beskrivande namn för din inställningsmall och klicka på spara-ikonen. Mallen lagrar alla dina aktuella projektinställningar (måldetektering, bearbetningsalternativ, index, och exportformat) för enkel återanvändning i framtida projekt. Mallarna sparas i mappen `Project Templates/` i din projektsparmapp och kan även väljas eller exporteras från huvudmenyn (*Välj mall* / *Spara mall* / *Exportera mall*).
* **Användningsfall**:
  * Skapa mallar för olika kamerasystem (RGB, multispektralt, NIR)
  * Spara standardkonfigurationer för specifika grödor eller analysflöden
  * Dela enhetliga inställningar inom ett team
* **Så här använder du**:
  1. Konfigurera alla önskade projektinställningar
  2. Ange ett mallnamn (t.ex. ”RedEdge Survey3 NDVI Standard”)
  3. Klicka på spara-ikonen
  4. Mallen kan nu laddas när du skapar nya projekt

***

## Spara projektmapp

Denna inställning anger var nya projekt sparas som standard.

* **Typ**: Visning av katalogväg + redigeringsknapp
* **Standard (Windows)**: `Documents\MAPIR\Chloros Projects` i mappen Dokument — vanligtvis `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`, eller där OneDrive eller en roamingprofil har flyttat mappen Dokument
* **Standard (Linux)**: `~/Documents/MAPIR/Chloros Projects`
* **Beskrivning**: Visar den aktuella standardkatalogen där nya Chloros-projekt skapas. Klicka på redigeringsikonen för att välja en annan katalog. Denna inställning sparas som en enda textrad i `~/.chloros/working_directory.txt` — i Windows är det `C:\Users\<Username>\.chloros\working_directory.txt`. Om den filen saknas, eller pekar på en sökväg som inte längre finns, faller Chloros tillbaka till standardinställningen ovan. CLI-gränssnittet läser och skriver samma fil, så `chloros-cli` och GUI-gränssnittet är alltid överens om var projekten ska ligga.
* **Projektmallar** finns i en undermapp med namnet `Project Templates/` i denna katalog.
* **När ska du ändra**:
  * Ställ in på en nätverksenhet för teamsamarbete
  * Byt till en enhet med mer lagringsutrymme för stora datamängder
  * Organisera projekt efter år, kund eller projekttyp i olika mappar
* **Obs**: Att ändra denna inställning påverkar endast NYA projekt. Befintliga projekt förblir på sina ursprungliga platser.
* **Windows-sökvägslängd**: om inte Windows-långa sökvägar är aktiverade (installationsprogrammets alternativ **Aktivera Windows-långa sökvägar**), avvisas en projektmapp som är djupare än cirka 130 tecken före bearbetning, eftersom dess produkter skulle överskrida Windows gräns på 260 tecken för sökvägar. Se [Starta bearbetningen](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Inställningarnas beständighet

Ett Chloros-projekt är en **mapp**. Alla projektinställningar sparas i `project.json` inuti den; ansluten hårdvara sparas tillsammans med dem i `cameras.json` och `sensors.json`, så att kameror och ljussensorer återansluts när ett projekt öppnas på nytt. När du öppnar ett projekt på nytt återställs alla inställningar exakt som du lämnade dem. Sparade projekt kan även köras utan grafiskt gränssnitt med `chloros-cli project` eller SDK:s `open_project`.

### Inställningshierarki

Inställningarna tillämpas i följande ordning:

1. **Systemstandardvärden** – Inbyggda standardvärden definierade av Chloros

2. **Mallinställningar** – Om du laddar en mall när du skapar ett projekt
3. **Inställningar för sparade projekt** – Inställningar som sparats tillsammans med projektfilen
4. **Manuella justeringar** – Alla ändringar du gör under den aktuella sessionen

### Inställningar och bildbearbetning

Bearbetningsinställningarna läses in när en bearbetningskörning startar. Att ändra en inställning påverkar inte i efterhand produkter som redan finns på disken – kör bearbetningen igen för att tillämpa de nya inställningarna. Några inställningar påverkar aldrig bearbetningen alls:

* Upplösning för bildminiatyrer (endast för visning)
* Spara projektmall
* Spara projektmapp

***

## Referens för konfigurationsnycklar

För automatisering (CLI `--config`, SDK `configure` eller direktläsning av `project.json`) är detta de exakta nycklarna under `Project Settings`:

| Nyckelväg | Typ | Standard |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | tal mellan 0 och 10 000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | tal 0–100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | tal 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | tal -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | cap-profil-id eller `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | lista över indexkonfigurationer | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Nycklarna `Array alignment` skrivs ut första gången avsnittet Array Alignment renderas eller när ett automatiseringsanrop ställer in dem. Så länge de saknas använder pipelinen samma värden som visas ovan (`true`, `true`, bilineär), så ett projekt.json utan dem beter sig på exakt samma sätt som ett med dem.

### Nycklar lagrade i `project.json` utan kontroll i inställningspanelen

Dessa finns under samma `Project Settings`-träd och läses av bearbetningen, men du hittar ingen widget för dem i sidofältet:

| Nyckelväg | Typ | Standard | Ställs in av |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Åsidosätter hur LATTICE-inmatnings-TIFF-filer tolkas; `auto` härleder från varje fils `Chloros:ProcessingLevel` XMP-tagg samt antalet kanaler. Ignoreras för Survey3-inspelningar `.raw`. Avsiktligt ingen GUI-inställning — ”auto” är korrekt i alla normala fall. |
| `Processing → Target reflectance dir` | sökvägssträng | `""` | `chloros-cli process --target-reflectance-dir`, eller projektets mål-API |
| `Processing → Target reflectance config` | ordlista med kamerans serienummer som nyckel | `{}` | Registrering av ett mål inom bildrutan (läge `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | sökvägssträng | `""` | SDK `process_folder(daq_log_path=…)`. Pekar på en `.daq`-inspelning eller en mapp med sådana |
| `Target Detection → Minimum calibration target squares` | nummer | `4` | Äldre standard; ingen kontroll och ingen CLI-flagga |
| `UI → Grid thumbnail size` | nummer | `160` | Bildrutnätets egen zoomreglage för miniatyrbilder |

Två visningsinställningar lagras **på högsta nivå i `project.json`**, helt utanför `Project Settings`, eftersom de avser visningsstatus snarare än bearbetningsinställningar:

| Nyckelväg | Typ | Standardvärde | Ställs in av |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | heltal 1–256 | `1` | Bildflikens GSD-reglage (px) — se [Öppna en bild i helskärmsläge](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Rekommenderade metoder

1. **Börja med standardinställningarna**: Standardinställningarna fungerar bra för de flesta MAPIR-kamerasystem och vanliga arbetsflöden.
2. **Skapa mallar**: När du har optimerat inställningarna för ett specifikt arbetsflöde eller en specifik kamera ska du spara dem som en mall för att säkerställa enhetlighet mellan olika projekt.
3. **Testa innan fullständig bearbetning**: När du experimenterar med nya inställningar ska du testa dem på en liten delmängd bilder innan du bearbetar hela din datamängd.
4. **Dokumentera dina inställningar**: Använd beskrivande mallnamn som anger kamerasystem, bearbetningstyp och avsedd användning (t.ex. ”Survey3\_RGB\_NDVI\_Agriculture”).
5. **Val av exportformat**: Välj exportformat utifrån din slutliga användning:
   * Vetenskaplig analys → TIFF (16-bitars eller 32-bitars)
   * GIS-bearbetning → TIFF (16-bitars)
   * Snabb visualisering → PNG (8-bitars)
   * Delning på webben → JPG (8-bitars)

***

För mer information om multispektrala index i Chloros, se sidan [Formler för multispektrala index](multispectral-index-formulas.md).
