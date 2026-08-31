# Avsluta bearbetningen

När Chloros har slutfört bearbetningen är det dags att granska resultaten, kontrollera utskriftskvaliteten och förbereda de bearbetade bilderna för användning i ditt arbetsflöde. Den här sidan guidar dig genom de sista stegen och nästa åtgärder.

## Indikation på att bearbetningen är klar

När bearbetningen har slutförts framgångsrikt visas flera indikatorer:

* ✅ **Förloppsindikator**: Når 100 % färdigställande
* ✅ **Felsökningslogg**: Visar de sista raderna i `[RUN-SUMMARY]` med antal (bilder, kameragrupper, mål, kalibrerade bilder, skrivna filer)
* ✅ **Startknapp**: Blir aktiverad igen (redo för nästa bearbetningskörning)
* ✅ **Utdatafiler**: Alla bearbetade bilder sparas i projektets utdatastruktur (nedan)

{% hint style="warning" %}
**En körning som inte skriver ut några bilder är ett misslyckande.** Om du begärde bildprodukter och körningen inte skrev några, rapporterar Chloros det som ett misslyckande — `[RUN-SUMMARY]` antyder i loggnamnet den troliga orsaken (inget importerat, inget mål upptäckt eller alla begärda produkter hoppades över som icke tillämpliga). Motsvarigheten CLI avslutas med ett värde som inte är noll. En avsiktlig körning med endast metadata (alla exportprodukter avstängda, inga index) räknas fortfarande som lyckad. Se [referensen för CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Hitta dina bearbetade bilder

### Öppna utdatamappen

1. Klicka på **Huvudmenyn**-ikonen <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (uppe till vänster)
2. Välj **”Öppna projektmapp”**

3. Din filutforskare öppnas till projektkatalogen
4. Leta reda på ditt projekt efter namn

### Utmatningsträdet

Produkterna sparas **i projektmappen, grupperade efter kamera och sedan efter filformat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Kameramapp**: `LATT-<sensor>-<lens>-F<filter>` för LATTICE (motsvarar bildens EXIF-data `Model`), `<model>_<filter>` för Survey3 (t.ex. `Survey3N_RGN`). Två kameror som delar sensor och filter men har olika objektiv har separata träd — vinjettering, synfält och distorsion skiljer sig åt.
* **Formatmapp**: följer din inställning för exportformat – `tiff16`, `tiff8`, `png8`, `jpg8`, eller `tiff32` för TIFF (32-bitars, procent). Radiance är alltid float32 och placeras alltid under `tiff32`.
* **Produktmappar**:
  * `Reflectance_Calibrated_Images/` — kalibrerad reflektans
  * `Debayered_Images/` — linjär debayering (LATTICE)
  * `Preview_Images/` — förhandsgranskning på skärm (LATTICE)
  * `Radiance_Images/` — spektral radianstyp float32, W/m²/sr/nm (LATTICE multispektral)
  * `Vignette_Corrected_Images/` **eller** `Sensor_Response_Images/` — det okalibrerade alternativet för bilder utan reflektansreferens; exakt en av de två finns per körning, vald genom inställningen för vignettkorrigering
  * `<INDEX>_Index_Images/` — en mapp per valt index (t.ex. `NDVI_Index_Images`)

{% hint style="info" %}
**Varje exporterad produkt behåller namnet på KÄLLFILEN.**En radiance-export av `capture_..._raw.tif` heter fortfarande `capture_..._raw.tif` — den finns bara i `tiff32/Radiance_Images/`.**Det är mappen som identifierar produkten, inte filnamnet**, så om du söker efter `*radiance*.tif` hittar du ingenting; sök istället efter mappen.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Hur många filer ska det finnas?

Räkna inte ut det med en formel – antalet utdata beror på vilka produkter som har aktiverats och vilka som gäller för varje kamera (t.ex. får kameror med RGB ingen strålningsintensitet/reflektans). Det officiella antalet finns i loggen: den sista raden i `[RUN-SUMMARY]` anger exakt hur många filer som skrevs, och förklarande rader redogör för allt som hoppades över.

***

## Granska bearbetade bilder

### Snabb förhandsgranskning i File Explorer

**Windows inbyggd förhandsgranskning:**

1. Navigera till en produktmapp (t.ex. `tiff16/Reflectance_Calibrated_Images/`)
2. Välj en bildfil
3. Förhandsvisningen visas i förhandsgranskningsfönstret i Windows Explorer
4. Använd piltangenterna för att bläddra igenom bilderna

### Förhandsvisning i externa bildvisare

**Rekommenderade bildvisare:*** **QGIS** – Gratis GIS-programvara (bäst för georefererad multispektralanalys)
* **IrfanView** – Snabb, lättviktig bildvisare (stöder TIFF)
* **Adobe Photoshop** – Professionell bildredigering (stöd för TIFF)
* **GIMP** – Gratis alternativ till Photoshop
* **Windows Photos** - Grundläggande visning (stöder eventuellt inte 16-bitars TIFF)

### Förhandsgranskning i Chloros-bildvisaren

Använd Chloros:s inbyggda bildvisare för avancerad visning:

1. Klicka på en bildminiatyr i filbläddraren
2. Bilden öppnas i huvudförhandsgranskningsområdet
3. Klicka på fliken **Bildvisare** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i vänster sidofält
4. Använd [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) för interaktiv analys

Se [Bildvisaren](../image-viewer-gui/opening-an-image-full-screen.md) för detaljerade instruktioner.

***

## Avläsning av reflektansvärden för pixlar (GIS / Pix4D / Skript)

Reflektansen lagras som ett heltal (DN), och **det DN-värde som motsvarar ρ = 1,0 beror på källkameran**:

| Källa          | ρ = 1,0 är | Hur man avgör                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (utrymme upp till ρ 2,0) | XMP-taggen `Chloros:PixelScale=32768` är inlagd i filen |
| Survey3         | **65535** (avskuren vid ρ 1,0)     | Inga `Chloros:*` XMP-taggar — den avsaknaden är signalen |

**Läs av `Chloros:PixelScale`-taggen och dividera med den** istället för att utgå från ett generellt värde på 65535 — att dividera LATTICE-reflektansen med 65535 halverar tyst varje värde. Ett specialfall har ingen skalning enligt designen: en 8-bitars källinspelning som skrivs ut som 8-bitars utdata beskärs, inte omskalas, och får avsiktligt ingen skalningstagg — exportera om i 16-bitars eller 32-bitars format istället för att dela den. Se [Utgångsbildformat](../output-image-formats.md) för hela historien.***

## Metadata som överförs till exportfilerna

Varje produkt behåller källbildens **GPS-block**och dess**EXIF-sub-IFD**, så en
export överför `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` och
`CameraSerialNumber` samt georeferensen.

{% hint style="warning" %}
**Om en ortomosaik får en orimlig skala, kontrollera först `FocalLength`.**
Pix4D beräknar markprovavståndet utifrån brännvidd plus höjd. Utan taggen
faller programmet tillbaka till en helt felaktig skala – vid en uppmätt flygning med 49 bilder rekonstruerades en 411 m × 160 m
stor apelsinlund som 47,8 km × 13 km, vilket resulterade i en ortofoto på 455 megapixel bestående av mestadels
tomt utrymme. Långsam uppdelning i rutor och en oväntat stor fil är symptom på detta, inte separata
problem.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Inte *alla* taggar kopieras. IFD0:s strukturtaggar lämnas medvetet kvar (att kopiera
dem förstör LATTICE-utdata), och `ExifImageWidth` / `ExifImageHeight` utesluts
eftersom de beskriver den ursprungliga inspelningen – en export som har ändrats i storlek skulle annars
ange dimensioner som står i strid med dess egen raster.

***

## Granska felsökningsloggen

### Kontrollera om det finns varningar eller fel

1. Öppna fliken **Felsökningslogg**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Bläddra igenom meddelandena
3. Leta efter gula varningar eller röda fel
4. Läs raderna med `[RUN-SUMMARY]` och eventuella tips
5. Kontakta MAPIR-supporten för hjälp

### Spara loggen

För att spara en logg över bearbetningen eller skicka den till MAPIR-supporten:

1. Klicka på knappen **”Kopiera”**eller**”Ladda ner”**

2. Spara som textfil i projektmappen
3. Bifoga den med projektdokumentationen
4. Skicka till MAPIR-supporten om problem uppstår

***

## Vanliga problem med utdata och lösningar

### Problem: Saknade utdatafiler

**Möjliga orsaker:**

* Produkten gäller inte för den kameran (t.ex. strålningsintensitet/reflektans för RGB-kameror – detta framgår av loggen)
* En nödvändig referens saknades (t.ex. reflektans utan mål och utan `.daq` nedåtriktad strålning)
* Kryssrutan för export av produkten var inaktiverad i projektinställningarna
* Det tog slut på diskutrymme under exporten

**Lösningar:**

1. Kontrollera raderna `[RUN-SUMMARY]` och `[EXPORT-CHECK]` i felsökningsloggen – de förklarar utelämnanden per kamera
2. Kontrollera kryssrutorna för exportprodukter i [Projektinställningar](adjusting-project-settings.md)
3. Kontrollera att det fanns tillräckligt med diskutrymme
4. Kör om processen efter att orsaken har åtgärdats

### Problem: Mörka eller ljusa kanter (vignettering syns fortfarande)

**Möjliga orsaker:**

* Vignetteringskorrigering inaktiverad
* Kamera/objektiv finns inte i Chloros-profildatabasen
* Extrem vignettering som överstiger korrigeringskapaciteten

**Lösningar:**

1. Kontrollera att vignetteringskorrigeringen är aktiverad i projektinställningarna
2. Kontrollera att kameramodellen har identifierats korrekt
3. Kontakta MAPIR-supporten om vignetteringen kvarstår

### Problem: Felaktiga färger eller värden

**Möjliga orsaker:**

* Inga kalibreringsmål har identifierats
* Fel modell för kalibreringsmål har valts
* Reflektanskalibrering inaktiverad
* Målbilder av dålig kvalitet

**Lösningar:**

1. Kontrollera att reflektanskalibrering var aktiverad
2. Kontrollera meddelandena ”Mål hittat” i felsökningsloggen
3. Granska målbildernas kvalitet
4. Bearbeta om med rätt mål markerade

### Problem: NDVI-värdena verkar felaktiga

**Förväntade NDVI-intervall:*** **Vatten, stenar, jord**: -0,1 till 0,2
* **Gles/sjuklig vegetation**: 0,2 till 0,4
* **Måttlig vegetation**: 0,4 till 0,6
* **Frisk, tät vegetation**: 0,6 till 0,9**Om värdena ligger utanför dessa intervall:**

1. Kontrollera att reflektanskalibreringen har tillämpats
2. Kontrollera att ljussensorloggen har inkluderats
3. Kontrollera att kalibreringsmålen har detekterats
4. Se till att rätt kameramodell har detekterats
5. Granska tidpunkten och förhållandena för målbildens inspelning
6. Om du själv beräknar indexen utifrån reflektansfilerna, kontrollera att du har dividerat med filens `Chloros:PixelScale` (se ovan)

***

## Använda dina bearbetade bilder

### För fotogrammetri / skapande av ortomosaik

**Rekommenderat arbetsflöde:**

1.**Importera kalibrerade reflektansbilder** till fotogrammetriprogram:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Behåll EXIF-metadata**: Se till att GPS-data bevaras för geotaggning
3. **Kalibrerade arbetsflöden**: Använd reflektansbilder för vetenskaplig noggrannhet – LATTICE-reflektans innehåller de XMP-kalibreringstaggar som Pix4D läser
4. **Bearbeta indexmosaiker**: Skapa NDVI-ortomosaiker från enskilda indexbilder
5. **Exportera georefererade GeoTIFF**: För användning i GIS-applikationer

### För GIS-analys

**Rekommenderat arbetsflöde:**

1.**Ladda in i QGIS, ArcGIS eller liknande**

2.**Använd 16-bitars TIFF** reflektansbilder för multibandsanalys (dela med filens `Chloros:PixelScale`)
3. **Använd indexbilder** (NDVI, NDRE) som färdiga vegetationslager
4. **Rasterkalkylator**: Kombinera band för anpassad analys
5. **Exportera**: Skapa klassificeringskartor, förändringsdetektering, kartor över vegetationens hälsa

### För direkt analys/rapportering

**Rekommenderat arbetsflöde:**

1.**Använd indexbilder med LUT-färger** för visuella rapporter
2. **Extrahera statistik**: Medelvärde för NDVI per fält/parcell
3. **Tidsserier**: Jämför index mellan flera mätomgångar
4. **Skapa rapporter**: Inkludera kartor, statistik och visualiseringar***

## Arkivering och säkerhetskopiering

### Rekommenderad säkerhetskopieringsstrategi

**Vad som ska sparas:*** ✅ **Originalbilder i RAW/JPG eller LATTICE-råbilder** – Arkivera på separat enhet/i molnet; rådata är källan till bearbetningskedjan och allt annat kan återskapas utifrån den
* ✅ **`.daq` / `.csv`-filer från ljussensorer** – Behövs för att senare återberäkna reflektansen
* ✅ **Bearbetade resultat** – Spara kalibrerade bilder och index
* ✅ **Projektmapp** (`project.json` och tillhörande filer) – Innehåller alla inställningar för eventuell ombearbetning
* ✅ **Felsökningslogg** – Dokumenterar detaljerna i bearbetningen
* ✅ **Kalibreringsmålbilder** – För verifiering och ombearbetning**Rekommendationer för lagring:*** **Omedelbar säkerhetskopiering**: Extern hårddisk
* **Långsiktigt arkiv**: Molnlagring (Google Drive, Dropbox, etc.)
* **Kritiska data**: Spara 2–3 kopior på olika platser***

## Nästa bearbetningsomgångar

### Återanvända projektinställningar

Om du ska bearbeta liknande datamängder i framtiden:

1. **Spara projektmall** (om det inte redan är gjort)
2. **Skapa nytt projekt** med den sparade mallen
3. **Importera nya bilder**

4.**Bearbeta**med identiska inställningar för att säkerställa konsistens

### Batchbearbetning av flera sessioner

För flera sessioner/datauppsättningar:**Alternativ 1: GUI – Flera projekt**

* Skapa ett separat projekt för varje session
* Använd konsekventa mallinställningar
* Bearbeta en i taget

**Alternativ 2: Chloros CLI (endast Chloros+)**

* Automatisera batchbearbetning
* Bearbeta flera mappar med skript
* Se [CLI-dokumentationen](../CLI.md) och [CLI-referensen](../reference/cli-reference.md)

**Alternativ 3: Python SDK (endast Chloros+)**

* Programmerad styrning
* Integration med analyspipelines
* Se [API-dokumentationen](../api-python-sdk.md) och [SDK-referensen](../reference/sdk-reference.md)

***

## Felsökning vid efterbearbetning

### Ombearbetning med andra inställningar

Om resultaten inte är tillfredsställande:

1. Behåll originalbilderna (radera dem aldrig)
2. Öppna samma projekt i Chloros
3. Justera inställningarna i panelen Projektinställningar
4. Bearbeta igen – resultaten sparas i samma produktmappar, så filer med samma namn från föregående körning ersätts

### Bearbeta en delmängd av bilder

Så här bearbetar du endast specifika bilder på nytt:

1. Skapa ett nytt projekt
2. Importera endast de bilder som behöver bearbetas på nytt
3. Använd samma inställningsmall
4. Bearbeta den mindre datamängden

### Få hjälp

Om du stöter på problem:

* 📧 **E-post**: info@mapir.camera (bifoga felsökningsloggen)
* 🌐 **Support**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Vanliga frågor**: [Vanliga frågor](../faq.md)
* 📖 **Dokumentation**: [Chloros-handbok](../)***

## Sammanfattning: Hela arbetsflödet

Du har nu slutfört hela arbetsflödet för Chloros:

1. ✅ **Skapat projekt** – Se [Projekt](../projects.md)
2. ✅ **Lagt till filer** – Se [Lägga till filer](adding-files-to-a-project.md)
3. ✅ **Justerat inställningar** – Se [Justera projektinställningar](adjusting-project-settings.md)
4. ✅ **Mål markerade** – Se [Välja målbilder](choosing-target-images.md)
5. ✅ **Bearbetning påbörjad** – Se [Starta bearbetningen](starting-the-processing.md)
6. ✅ **Övervakade framsteg** – Se [Övervaka bearbetningen](monitoring-the-processing.md)
7. ✅ **Granskade resultat** – Denna sida**Dina kalibrerade, reflektanskorrigerade multispektrala bilder är klara för analys!**

***

## Ytterligare resurser

### Avancerade funktioner

* [**Bildvisare**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktiv visualisering och analys
* [**Index/LUT-sandlåda**](../image-viewer-gui/index-lut-sandbox.md) – Testning av anpassade index
* [**Multispektrala indexformler**](../project-settings/multispectral-index-formulas.md) – Fullständig indexreferens

### Automatisering och integration

* [**CLI-dokumentation**](../CLI.md) – Batchbearbetning via kommandoraden
* [**Python SDK**](../api-python-sdk.md) – Programmerbar automatisering
* [**Chloros+ Funktioner**](../#chloros) – Avancerade bearbetningsfunktioner

### Support och utbildning

* [**Vanliga frågor**](../faq.md) – Svar på vanliga frågor
* [**Kalibreringsmål**](../calibration-targets.md) – Förstå reflekanskalibrering
* [**Kameror som stöds**](../supported-cameras.md) – Kompatibel hårdvara
