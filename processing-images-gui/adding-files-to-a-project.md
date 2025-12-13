# Lägga till filer till ett projekt

När du har skapat eller öppnat ett projekt i Chloros är nästa steg att lägga till dina multispektrala bilder för att påbörja bearbetningen. Filbläddraren<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> gör det enkelt att importera bilder och hantera din dataset.

## Öppna filbläddraren

1. Öppna eller skapa ett projekt i Chloros
2. Klicka på ikonen **Filbläddraren** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> i vänster sidofält
3. Panelen Filbläddraren visar projektets fillista

{% hint style=&quot;info&quot; %}
**Filtyper som stöds**: Chloros stöder RAW+JPG- och JPG-bildfiler från MAPIR Survey3W och Survey3N kameror. Endast RAW+JPG rekommenderas.
{% endhint %}

***

## Lägga till bilder till ditt projekt

Det finns två huvudsakliga sätt att lägga till bilder till ditt projekt:

### Metod 1: Lägg till filer

Använd det här alternativet för att importera enskilda bildfiler eller ett litet urval av filer.

1. Klicka på knappen **”Lägg till filer”** högst upp i panelen Filbläddraren.
2. Navigera till mappen som innehåller dina bilder.
3. Välj en eller flera bildfiler (håll ned **Ctrl** för att välja flera filer).
4. Klicka på **”Öppna”** för att importera de valda filerna.

### Metod 2: Lägg till mapp

Använd det här alternativet för att importera alla bilder från en mapp på en gång.

1. Klicka på knappen **&quot;Lägg till mapp&quot;** längst upp i filbläddrarpanelen.
2. Navigera till och välj mappen som innehåller bilderna från din fotograferingssession.
3. Klicka på **&quot;Välj mapp&quot;** för att importera alla bilder som stöds från den mappen.

***

## Förstå filbläddrarens tabell

När bilderna har importerats visas de i en tabell med följande kolumner:

### Miniatyrbild

* Liten förhandsgranskning av varje bild.
* Klicka på miniatyrbilden för att visa hela bilden i huvudförhandsgranskningsområdet.

### Filnamn

* Ursprungligt filnamn från kameran.
* Behåller kamerans namngivningskonvention (t.ex. IMG\_0001.RAW).

### Tidsstämpel

* Datum och tid då bilden togs.
* Extraherad från bildens EXIF-metadata.
* Används för PPK-synkronisering och kalibreringsmåldetektering

### Kameramodell

* Automatiskt detekterad kamera- och filterkonfiguration
* Exempel: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Används för att tillämpa korrekta bearbetningsprofiler

### Målkolumn (kryssruta)

* Markera denna ruta för bilder som innehåller kalibreringsmål
* Påskyndar måldetekteringen avsevärt under bearbetningen
* Se [Välja målbilder](choosing-target-images.md) för mer information

***

## Hantera filer i ditt projekt

### Ta bort filer

Så här tar du bort oönskade bilder från ditt projekt:

1. Välj en eller flera bilder i tabellen Filbläddraren
2. Klicka på knappen **&quot;Ta bort valda&quot;**
3. Bekräfta borttagningen (filerna raderas inte från hårddisken, utan tas bara bort från projektet)

### Sortera och filtrera

* **Sortera efter kolumn**: Klicka på valfri kolumnrubrik för att sortera bilder
* **Sortera efter tidsstämpel**: Användbart för att organisera kronologiska bildsekvenser.
* **Filter för kameramodell**: Gruppera bilder efter kameratyp om du använder flera kameror.

***

## Bildförhandsgranskning

### Visa hela bilden

Klicka på valfri miniatyrbild i filbläddraren för att visa den i huvudförhandsgranskningsområdet:

1. Bilden visas i det centrala förhandsgranskningsfönstret.
2. Använd zoomkontrollerna för att granska bilddetaljer.
3. Navigera mellan bilderna med piltangenterna

### Snabbnavigering

* **Föregående bild**: Klicka på vänsterpilen eller tryck på ←-tangenten
* **Nästa bild**: Klicka på högerpilen eller tryck på →-tangenten
* **Zooma in/ut**: Använd mushjulet eller zoomknapparna
* **Panorera**: Klicka och dra på bilden när du har zoomat in

***

## Hantering av dubbletter

Chloros upptäcker och ignorerar automatiskt dubbla filer:

* Filer med identiska filnamn hoppas över.
* Förhindrar oavsiktlig dubbelbearbetning.
* Varningsmeddelande visas när dubbletter upptäcks.

{% hint style=&quot;warning&quot; %}
**Viktigt**: Byt inte namn på eller ändra dina originalbildfiler innan du importerar dem. Chloros förlitar sig på originalfilnamn och metadata för korrekt bearbetning.
{% endhint %}

***

## Blandade kameradataset

Om ditt projekt innehåller bilder från flera MAPIR-kameror:

1. Chloros detekterar automatiskt varje kameramodell
2. Varje kameratyp bearbetas med sin lämpliga kalibreringsprofil
3. Filbläddraren visar kameramodellen i kolumnen Kameramodell
4. Bearbetningen tillämpar korrekta inställningar för varje kameratyp

**Exempel på scenario**: Survey3W RGN + Survey3N OCN dubbla kameror

***

## Bästa praxis

### Organisera före import

* Spara kalibreringsmålbilderna i samma mapp som undersökningsbilderna
* Behåll den ursprungliga mappstrukturen från din kamera/SD-kort
* Blanda inte datamängder från olika sessioner i ett projekt

### Filnamngivning

* Behåll de ursprungliga kamerafilnamnen (IMG\_0001.RAW, etc.)
* Byt inte namn på filerna före import
* De ursprungliga namnen innehåller viktig metadata

### Kalibreringsmålbilder

* Inkludera alltid 1–2 kalibreringsmålbilder per session.
* Ta bilder av målen före och efter fotograferingssessionen.
* Placera målen i samma ljusförhållanden som fotograferingsområdet.
* Markera målbilder med kryssrutan Mål för att påskynda bearbetningen.

***

## Vanliga problem och lösningar

### Bilder visas inte efter import

**Möjliga orsaker:**

* Filformatet stöds inte (endast RAW+JPG och JPG från MAPIR-kameror)
* Bilderna kommer från kameror som inte är MAPIR (se [Kameror som stöds](../supported-cameras.md))
* Filskada eller ofullständig överföring från SD-kort

**Lösning**: Kontrollera filformatet och kameramodellens kompatibilitet.

### Kameramodell upptäcks inte

**Möjliga orsaker:**

* Modifierade EXIF-metadata.
* Bilder redigerade i extern programvara.
* Ofullständig filöverföring.

**Lösning**: Importera om originalfilerna, utan modifieringar, från kameran/SD-kortet.

### Saknade tidsstämplar

**Möjliga orsaker:**

* Kamerans klocka är inte korrekt inställd
* EXIF-data har tagits bort av extern programvara

**Lösning**: Kontrollera att kamerans tidsinställningar var korrekta under fotograferingen

***

## Nästa steg

När dina filer har importerats:

1. **Granska fillistan** – Kontrollera att alla bilder har laddats korrekt
2. **Kontrollera kameramodeller** – Kontrollera att kamerorna har identifierats korrekt
3. **Markera målbilder** – Se [Välja målbilder](choosing-target-images.md)
4. **Justera inställningarna** – Konfigurera bearbetningsalternativ i [Projektinställningar](adjusting-project-settings.md)
5. **Starta bearbetningen** – Se [Starta bearbetningen](starting-the-processing.md)

För detaljerad information om projektkonfiguration, se [Justera projektinställningar](adjusting-project-settings.md).
