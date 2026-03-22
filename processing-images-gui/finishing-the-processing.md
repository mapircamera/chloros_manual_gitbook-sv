# Avsluta bearbetningen

När Chloros har slutfört bearbetningen är det dags att granska resultaten, kontrollera utskriftskvaliteten och förbereda de bearbetade bilderna för användning i ditt arbetsflöde. Denna sida guidar dig genom de sista stegen och nästa åtgärder.

## Indikation på att bearbetningen är klar

När bearbetningen har slutförts framgångsrikt visas flera indikatorer:

* ✅ **Förloppsindikator**: Når 100 % färdigställande
* ✅ **Felsökningslogg**: Visar meddelandet ”Bearbetning klar”
* ✅ **Startknapp**: Blir aktiverad igen (redo för nästa bearbetningskörning)
* ✅ **Utdatafiler**: Alla bearbetade bilder sparas i undermappen för kameramodellen***

## Hitta dina bearbetade bilder

### Öppna utdatamappen

1. Klicka på **huvudmenyn** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (uppe till vänster)
2. Välj **&quot;Öppna projektmappen&quot;**

3. Din filutforskare öppnas till projektkatalogen
4. Leta reda på ditt projekt efter namn

***

## Granska bearbetade bilder

### Snabb förhandsgranskning i filutforskaren

**Windows inbyggd förhandsgranskning:**

1. Navigera till undermappen för kameramodellen
2. Välj en bildfil
3. Förhandsgranskningen visas i Windows Explorer-förhandsgranskningsfönstret
4. Använd piltangenterna för att bläddra igenom bilderna

### Förhandsgranskning i externa bildvisare

**Rekommenderade bildvisare:*** **QGIS** – Gratis GIS-programvara (bäst för georefererad multispektral analys)
* **IrfanView** – Snabb, lättviktig bildvisare (stöder TIFF)
* **Adobe Photoshop** – Professionell redigering (stöd för TIFF)
* **GIMP** – Gratis alternativ till Photoshop
* **Windows Photos** – Grundläggande visning (stöder eventuellt inte 16-bitars TIFF)

### Förhandsgranskning i Chloros Bildvisare

Använd Chloros:s inbyggda bildvisare för avancerad visualisering:

1. Klicka på en bildminiatyr i filbläddraren
2. Bilden öppnas i huvudförhandsgranskningsområdet
3. Klicka på fliken **Bildvisare** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i vänster sidofält
4. Använd [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) för interaktiv analys

Se [Bildvisare](../image-viewer-gui/opening-an-image-full-screen.md) för detaljerade instruktioner.

***

## Granska felsökningsloggen

### Kontrollera om det finns varningar eller fel

1. Öppna fliken **Felsökningslogg** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> -fliken
2. Bläddra igenom meddelandena
3. Leta efter gula varningar eller röda fel
4. Granska eventuella problem som noterats
5. Kontakta MAPIR support för hjälp

### Spara loggen

För att spara en logg över bearbetningen eller för att skicka den till MAPIR Support:

1. Klicka på knappen **&quot;Kopiera&quot;**eller**&quot;Ladda ner&quot;**

2. Spara som textfil i projektmappen
3. Bifoga med projektdokumentationen
4. Skicka till MAPIR support om problem uppstår

***

## Vanliga utdataproblem och lösningar

### Problem: Saknade utdatafiler

**Möjliga orsaker:**

* Filerna uppfyllde inte bearbetningskriterierna
* Bilder endast för mål (uteslutna från export)
* Diskutrymmet tog slut under exporten
* Filskada under bearbetningen

**Lösningar:**

1. Kontrollera felsökningsloggen för meddelanden om hoppade steg/fel
2. Kontrollera att det fanns tillräckligt med diskutrymme
3. Räkna filerna: Bör stämma överens med (ursprungligt antal – målantal) × (index + 1)
4. Importera och bearbeta eventuella saknade filer på nytt

### Problem: Mörka eller ljusa kanter (vignettering fortfarande synlig)

**Möjliga orsaker:**

* Vignettkorrigering inaktiverad
* Kamera/objektiv finns inte i Chloros-profildatabasen
* Extrem vignettering som överstiger korrigeringskapaciteten

**Lösningar:**

1. Kontrollera att vignetteringskorrigering var aktiverad i projektinställningarna
2. Kontrollera att kameramodellen har identifierats korrekt
3. Kontakta MAPIR-support om vignetteringen kvarstår

### Problem: Felaktiga färger eller värden

**Möjliga orsaker:**

* Inga kalibreringsmål har identifierats
* Fel kalibreringsmålmodell har valts
* Reflektanskalibrering inaktiverad
* Målbilder av dålig kvalitet

**Lösningar:**

1. Kontrollera att reflektanskalibrering var aktiverad
2. Kontrollera meddelanden om &quot;Mål hittat&quot; i felsökningsloggen
3. Granska målbildens kvalitet
4. Bearbeta om med rätt mål markerade

### Problem: NDVI-värden verkar felaktiga

**Förväntade NDVI-intervall:*** **Vatten, stenar, jord**: -0,1 till 0,2
* **Gles/ohälsosam vegetation**: 0,2 till 0,4
* **Måttlig vegetation**: 0,4 till 0,6
* **Hälsosam, tät vegetation**: 0,6 till 0,9**Om värdena ligger utanför dessa intervall:**

1. Kontrollera att reflektanskalibrering har tillämpats
2. Kontrollera att ljussensorloggen har inkluderats
3. Kontrollera att kalibreringsmål har detekterats
4. Se till att rätt kameramodell har detekterats
5. Granska tidpunkten och förhållandena för bildtagningen

***

## Använda dina bearbetade bilder

### För fotogrammetri / skapande av ortomosaik

**Rekommenderat arbetsflöde:**

1.**Importera kalibrerade reflektansbilder** till fotogrammetriprogramvara:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Behåll EXIF-metadata**: Se till att GPS-data bevaras för geotaggning
3. **Kalibrerade arbetsflöden**: Använd reflektansbilder för vetenskaplig noggrannhet
4. **Bearbeta indexmosaiker**: Skapa NDVI-ortomosaiker från enskilda indexbilder
5. **Exportera georefererade GeoTIFF**: För användning i GIS-applikationer

### För GIS-analys

**Rekommenderat arbetsflöde:**

1.**Ladda in i QGIS, ArcGIS eller liknande**

2.**Använd 16-bitars TIFF** reflektansbilder för multibandsanalys
3. **Använd indexbilder** (NDVI, NDRE) som färdiga vegetationslager
4. **Rasterkalkylator**: Kombinera band för anpassad analys
5. **Exportera**: Skapa klassificeringskartor, förändringsdetektering, kartor över vegetationens hälsa

### För direkt analys/rapportering

**Rekommenderat arbetsflöde:**

1.**Använd indexbilder med LUT-färger** för visuella rapporter
2. **Extrahera statistik**: Medelvärde NDVI per fält/tomt
3. **Tidsserier**: Jämför index över flera sessioner
4. **Skapa rapporter**: Inkludera kartor, statistik och visualiseringar***

## Arkivering och säkerhetskopiering

### Rekommenderad säkerhetskopieringsstrategi

**Vad du ska spara:*** ✅ **Original RAW/JPG-bilder** – Arkivera på separat enhet/moln
* ✅ **Bearbetade resultat** – Spara kalibrerade bilder och index
* ✅ **Projektfil** – Innehåller alla inställningar för ombearbetning vid behov
* ✅ **Felsökningslogg** – Dokumenterar bearbetningsdetaljer
* ✅ **Kalibreringsmålbilder** – För verifiering och ombearbetning**Rekommendationer för lagring:*** **Omedelbar säkerhetskopiering**: Extern hårddisk
* **Långsiktigt arkiv**: Molnlagring (Google Drive, Dropbox, etc.)
* **Kritiska data**: Spara 2–3 kopior på olika platser***

## Nästa bearbetningsomgångar

### Återanvända projektinställningar

Om du bearbetar liknande datamängder i framtiden:

1. **Spara projektmall** (om det inte redan är gjort)
2. **Skapa nytt projekt** med hjälp av den sparade mallen
3. **Importera nya bilder**

4.**Bearbeta**med identiska inställningar för att säkerställa konsistens

### Batchbearbetning av flera sessioner

För flera sessioner/datauppsättningar:**Alternativ 1: GUI – Flera projekt**

* Skapa ett separat projekt för varje session
* Använd konsekventa mallinställningar
* Bearbeta ett i taget

**Alternativ 2: Chloros CLI (endast Chloros+)**

* Automatisera batchbearbetning
* Bearbeta flera mappar med skript
* Se [CLI-dokumentation](../CLI.md)

**Alternativ 3: Python SDK (endast Chloros+)**

* Programstyrning
* Integration med analyspipelines
* Se [API Dokumentation](../api-python-sdk.md)

***

## Felsökning av efterbearbetning

### Ombearbetning med andra inställningar

Om resultaten inte är tillfredsställande:

1. Behåll originalbilderna (radera dem aldrig)
2. Öppna samma projekt i Chloros
3. Justera inställningarna i panelen Projektinställningar
4. Bearbeta igen – utdata kommer att skriva över tidigare resultat

### Bearbeta en delmängd av bilder

För att endast bearbeta specifika bilder:

1. Skapa ett nytt projekt
2. Importera endast de bilder som behöver bearbetas om
3. Använd samma inställningsmall
4. Bearbeta en mindre datamängd

### Få hjälp

Om du stöter på problem:

* 📧 **E-post**: info@mapir.camera (bifoga felsökningslogg)
* 🌐 **Support**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Vanliga frågor](../faq.md)
* 📖 **Dokumentation**: [Chloros Manual](../)***

## Sammanfattning: Komplett arbetsflöde

Du har nu slutfört hela Chloros-bearbetningsflödet:

1. ✅ **Skapat projekt** – Se [Projekt](../projects.md)
2. ✅ **Lagt till filer** – Se [Lägga till filer](adding-files-to-a-project.md)
3. ✅ **Justerat inställningar** – Se [Justera projektinställningar](adjusting-project-settings.md)
4. ✅ **Markerade mål** – Se [Välja målbilder](choosing-target-images.md)
5. ✅ **Startade bearbetningen** – Se [Starta bearbetningen](starting-the-processing.md)
6. ✅ **Övervakad framsteg** - Se [Övervaka bearbetningen](monitoring-the-processing.md)
7. ✅ **Granskade resultat** - Denna sida**Dina kalibrerade, reflektanskorrigerade multispektrala bilder är klara för analys!**

***

## Ytterligare resurser

### Avancerade funktioner

* [**Bildvisare**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktiv visualisering och analys
* [**Index/LUT-sandlåda**](../image-viewer-gui/index-lut-sandbox.md) – Testning av anpassade index
* [**Multispektrala indexformler**](../project-settings/multispectral-index-formulas.md) – Komplett indexreferens

### Automatisering och integration

* [**CLI Dokumentation**](../CLI.md) – Batchbearbetning via kommandoraden
* [**Python SDK**](../api-python-sdk.md) – Programmerbar automatisering
* [**Chloros+ Funktioner**](../#chloros) – Avancerade bearbetningsfunktioner

### Support &amp; utbildning

* [**Vanliga frågor**](../faq.md) – Svar på vanliga frågor
* [**Kalibreringsmål**](../calibration-targets.md) – Förstå reflektanskalibrering
* [**Kompatibla kameror**](../supported-cameras.md) – Kompatibel hårdvara
