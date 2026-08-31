# Lägga till filer i ett projekt

När du har skapat eller öppnat ett projekt i Chloros är nästa steg att lägga till dina multispektrala bilder för att påbörja bearbetningen. Fliken ”<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">” i filbläddraren gör det enkelt att importera bilder och hantera din datamängd.

## Öppna filbläddraren

1. Öppna eller skapa ett projekt i Chloros
2. Klicka på ikonen **Filbläddraren** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> i vänster sidofält
3. I panelen Filbläddraren visas projektets fillista

{% hint style="info" %}
**Filtyper som stöds**:

* **Survey3W / Survey3N**: RAW+JPG-par och JPG-bilder (RAW+JPG rekommenderas)
* **LATTICE**: `.tif` / `.tiff`-bildtagningar — tagna med kamerastyrningen Chloros eller via en LATTICE-hub
* **Ljussensordata**: `.daq`-inspelningar (DAQ-U/M/E) och DAQ-M `.csv`-loggar över nedåtriktat ljus — importerade tillsammans med bilder för att driva reflektanskalibrering
{% endhint %}

***

## Lägga till bilder i ditt projekt

Det finns två huvudsakliga sätt att lägga till bilder i ditt projekt:

### Metod 1: Lägg till filer

Använd det här alternativet för att importera enskilda bildfiler eller ett litet urval av filer.

1. Klicka på knappen **”Lägg till filer”** (<img src="../.gitbook/assets/image (3).png" alt="" data-size="line">) högst upp i panelen Filbläddraren
2. Navigera till mappen som innehåller dina bilder
3. Markera en eller flera bildfiler (håll ned **Ctrl** för att markera flera filer)
4. Klicka på **”Öppna”** för att importera de markerade filerna

### Metod 2: Lägg till mapp

Använd det här alternativet för att importera alla bilder från en mapp på en gång. Du kan välja **flera mappar** i en och samma dialogruta.

1. Klicka på knappen **&quot;Lägg till mapp&quot;** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> högst upp i panelen Filbläddraren
2. Navigera till och välj den eller de mappar som innehåller bilderna från din inspelningssession
3. Klicka på **&quot;Välj mapp&quot;** för att importera alla bilder som stöds

{% hint style="info" %}
**Filer som inte går att ladda rapporteras.** Om en mapp innehåller filer som Chloros känner igen men inte kan ladda, visas en varning – bilderna försvinner inte tyst från rutnätet.
{% endhint %}

***

## Importera LATTICE-inspelningsmappar

LATTICE-inspelningar sparas med **en undermapp per exportnivå** – till exempel `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — med den motsvarande nedåtgående filen `.daq` i rotkatalogen:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Välj mappen i rotkatalogen för bilderna** (`output/` ovan). När den valda mappen i sig inte innehåller några bilder men har undermappar, går Chloros automatiskt in i dem — undermapparna på den nivån och rotmappen `.daq` hämtas alla på en gång.**Så här importeras bildserier:*** Varje bildserie importeras som en **enskild bild**, grupperad per bildserie (inte en post per nivå). De andra nivåerna i samma bildserie visas som visningslägen för just den bilden.
* **Bearbetningen börjar alltid från den obearbetade bilden.** De andra nivåerna är synliga, men endast `raw` matas någonsin genom bearbetningskedjan — att återbearbeta en redan bearbetad produkt skulle innebära att korrigeringarna tillämpas två gånger, så Chloros avvisas. En återimporterad export kan aldrig ta en bildseriens råbildsplats.
* En bildseriemapp som sparats **utan** råbildsimporter visas normalt, men bearbetningen hoppar över den och detta anges i loggen. (Flaggan CLI `--input-level` kan tvinga fram en startpunkt för detta fall – se [referensen för CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**LATTICE-hubbsessioner** importeras på samma sätt: peka på Lägg till mapp i sessionsmappen som kopierats från hubben (den innehåller `raw/` plus `previews/`), tillsammans med eventuella DAQ-M `.csv`-loggar. Om kamerans eller DAQ:ns kalibrering ännu inte finns i cacheminnet på din dator hämtar Chloros den automatiskt utifrån serienumret vid importen (kräver internetanslutning en gång).***

## Förstå tabellen i filbläddraren

När bilderna har importerats visas de i en tabell med följande kolumner:

### Filnamn

* Ursprungligt filnamn från kameran
* Behåller kamerans namngivningskonvention (t.ex. IMG\_0001.RAW eller capture\_20260816\_101500\_SN213800234\_raw.tif)

### Tidsstämpel

* Datum och tidpunkt då bilden togs
* Hämtas från bildens EXIF-metadata
* Används för matchning av ljussensorer, PPK-synkronisering och schemaläggning av kalibreringsmål

### Kameramodell

* Automatiskt detekterad kamera- och filterkonfiguration
* Exempel på Survey3: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* LATTICE-exempel: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Används för att tillämpa korrekta bearbetningsprofiler

### Målkolumn (kryssruta)

* Markera denna ruta för bilder som innehåller kalibreringsmål
* När minst en bild är markerad, **skannas endast de markerade bilderna** efter mål
* Se [Välja målbilder](choosing-target-images.md) för mer information

### Visa bildmetadata

Om du klickar på växlingsknappen i det övre högra hörnet ovanför tabellen visas den valda bildens metadata i bildrutnätet.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Ljussensorfiler i ditt projekt

* Filerna `.daq` och `.csv` visas i filbläddraren men är inte klickbara bilder — de tillhandahåller nedåtriktad irradians för reflektanskalibrering.
* Varje importerad `.daq`/`.csv`-fil listas under **Projektinställningar → DAQ-ljussensor**, där du kan granska den diffusorkåpskorrigering som gäller för varje fil. Se [Justera projektinställningar](adjusting-project-settings.md).
* Inspelningar som du gör på fliken **Ljussensorer** läggs automatiskt till i det öppna projektet – ingen manuell import behövs.***

## Hantera filer i ditt projekt

### Ta bort filer

Så här tar du bort oönskade bilder från ditt projekt:

1. Markera en eller flera bilder i tabellen i filbläddraren
2. Klicka på knappen **”Ta bort markerade”** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Bekräfta borttagningen (filerna raderas inte från hårddisken, utan tas endast bort från projektet)

### Sortering och filtrering

* **Sortera efter kolumn**: Klicka på valfri kolumnrubrik för att sortera bilderna
* **Sortering efter tidsstämpel**: Användbart för att organisera bildserier i kronologisk ordning
* **Filter efter kameramodell**: Gruppera bilder efter kameratyp om du använder flera kameror***

## Bildförhandsgranskning

### Visa hela bilden

Klicka på valfri bildminiatyr i filbläddraren för att visa den i huvudförhandsgranskningsområdet:

1. Bilden visas i det centrala förhandsgranskningsfönstret
2. Använd zoomreglagen för att granska bilddetaljer
3. Navigera mellan bilder med piltangenterna

### Snabbnavigering

* **Föregående bild**: Klicka på vänsterpilen eller tryck på ←-tangenten
* **Nästa bild**: Klicka på högerpilen eller tryck på →-tangenten
* **Zooma in/ut**: Använd mushjulet eller zoomknapparna
* **Panorera**: Klicka och dra på bilden när du har zoomat in***

## Hantering av dubbletter

Chloros upptäcker och ignorerar automatiskt dubbletter:

* Filer med identiska filnamn hoppas över
* Förhindrar oavsiktlig dubbelbearbetning
* Varningsmeddelande visas när dubbletter upptäcks

{% hint style="warning" %}
**Viktigt**: Byt inte namn på eller ändra dina originalbildfiler innan du importerar dem. Chloros förlitar sig på originalfilnamn och metadata för korrekt bearbetning.
{% endhint %}

***

## Datamängder med flera kameror

Om ditt projekt innehåller bilder från flera MAPIR-kameror:

1. Chloros identifierar automatiskt varje kameramodell — Survey3, LATTICE eller en blandning
2. Varje kameratyp bearbetas med sin lämpliga kalibreringsprofil
3. Filbläddraren visar kameramodellen i kolumnen Kameramodell
4. Varje kamera får sin egen utdatamappstruktur när den bearbetas

**Exempelscenarier**: Survey3W RGN + Survey3N OCN-konfiguration med två kameror, eller en LATTICE-konfiguration med en RGB-huvudkamera och flera smalbandsmoduler***

## Rekommenderade metoder

### Organisera före import

* Spara kalibreringsmålbilderna i samma mapp som kartläggningsbilderna
* Spara varje fotograferingssessionens `.daq` / `.csv`-ljussensorfiler tillsammans med bilderna från den sessionen
* Behåll den ursprungliga mappstrukturen från din kamera/SD-kort/hub
* Blanda inte datamängder från olika sessioner i ett och samma projekt

### Filnamngivning

* Behåll de ursprungliga filnamnen från kameran (IMG\_0001.RAW, capture\_..., etc.)
* Byt inte namn på filerna före import
* De ursprungliga namnen innehåller viktig metadata

### Kalibreringsmålbilder

* Inkludera alltid 1–2 kalibreringsmålbilder per session (Survey3; för LATTICE kan en DAQ-inspelning ersätta detta – se [Välja målbilder](choosing-target-images.md))
* Ta bilder av kalibreringsmålen före och efter fotograferingssessionen
* Placera målen under samma ljusförhållanden som fotograferingsområdet
* Markera målbilderna med kryssrutan ”Target”

***

## Vanliga problem och lösningar

### Bilder visas inte efter import

**Möjliga orsaker:**

* Filformatet stöds inte (se listan över stödda filtyper högst upp på denna sida)
* Bilderna kommer från kameror som inte är av typen MAPIR (se [Stödda kameror](../supported-cameras.md))
* Skadade filer eller ofullständig överföring från SD-kortet

**Lösning**: Kontrollera att filformatet och kameramodellen är kompatibla, och granska varningen vid filinläsningen för att se exakt vilka filer som misslyckades

### Kameramodellen upptäcks inte

**Möjliga orsaker:**

* Ändrade EXIF-metadata
* Bilder som redigerats i extern programvara
* Ofullständig filöverföring

**Lösning**: Importera om de ursprungliga, oförändrade filerna från kameran/SD-kortet

### Saknade tidsstämplar

**Möjliga orsaker:**

* Kamerans klocka är inte korrekt inställd
* EXIF-data har tagits bort av extern programvara

**Lösning**: Kontrollera att kamerans tidsinställningar var korrekta vid fotograferingen

### Projektet har öppnats på nytt och rapporterar saknade filer

Om källfilerna har flyttats eller raderats sedan projektet öppnades senast, visar Chloros **vilka** filer som saknas istället för att öppna ett tomt rutnät. Återställ filerna till deras ursprungliga sökvägar, eller ta bort de saknade posterna och importera om.***

## Nästa steg

När dina filer har importerats:

1. **Granska fillistan** – Se till att alla bilder har laddats korrekt
2. **Kontrollera kameramodellerna** – Kontrollera att kamerorna har identifierats korrekt
3. **Markera målbilder** – Se [Välja målbilder](choosing-target-images.md)
4. **Justera inställningarna** – Konfigurera bearbetningsalternativen i [Projektinställningar](adjusting-project-settings.md)
5. **Starta bearbetningen** – Se [Starta bearbetningen](starting-the-processing.md)

För detaljerad information om projektkonfiguration, se [Justera projektinställningar](adjusting-project-settings.md).
