# Kamerainställningar

Fliken **Kameror**är Chloros:s kontrollpanel för LATTICE-kameror i realtid: ett huvudområde som visar alla anslutna kameror som live-rutor, samt en sidofält som växlar mellan tre sidor –**kameralistan**, ett**inställningsfönster**(inställningar per kamera, för kameragrupper eller för inspelning – en i taget) och**Index Calculator**. Denna sida beskriver alla reglage i kameralistan, inställningsfönstret per kamera och inställningsfönstret för kameragrupper. Inspelningslägen, val av exporttyp och flödet ”Capture All” finns på den tillhörande sidan [Inspelningsinställningar och lägen](capture.md).

Fliken ”Kameror” visas i sidofältet så snart Chloros-backenden är klar. Alla kontroller nedan kommunicerar med den lokala backenden via `127.0.0.1:5000`; ändringar tillämpas omedelbart på livekameran om inte annat anges.

## Kameratyper som används på denna sida

Kontrollerna visas eller döljs beroende på vilken typ av kamera som är vald. I manualen används följande termer genomgående:

| Term | Betydelse | Filterkanaler |
| --- | --- | --- |
| **RGB-kamera** | LATTICE M3C med FRGB-filter (modellen innehåller `-FRGB`) | Red / Green / Blue |
| **Bayer multispektral** | LATTICE M3C med FRGN, FOCN eller FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — ett smalbandsfilter, ett kalibrerat band | Enkelt band |
| **Array-medlem** | En kamera ansluten som del av en synkroniserad array (kombinerad eller separat visning) | Enligt sitt filter |

RGB-kameror genomgår fotometrisk bearbetning (vitbalans, färgprofiler, gamma); multispektrala och monokameror genomgår den radiometriska kedjan och hoppar över de fotometriska inställningarna. Arraymedlemmar överför inställningar på strömnivå (pixelformat, upplösning, binning, utlösare, bildfrekvens) till arrayen — dessa rader blir skrivskyddade i rutan per kamera och flyttas istället till rutan för arrayinställningar.

## Huvudområdet

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

för bildflödet Om inga kameror är anslutna visar bildflödesområdet en startskärm med texten **”Anslut en kamera för att komma igång”**och två knappar:**Anslut kamera**(grön, öppnar dialogrutan för anslutning av en enskild kamera) och**Anslut array** (blå, öppnar dialogrutan för anslutning av en array). Själva anslutningsdialogrutorna beskrivs i [Ansluta kameror](connecting.md); begrepp relaterade till uppsättningar (synkronisering, nivåer, bandbredd) beskrivs i [Uppsättningar med flera kameror](arrays.md). När du öppnar ett sparat projekt som innehåller kameror visar startskärmen istället en snurrande ikon med texten ”Öppnar N sparade kameror på nytt…” medan Chloros återställer strömmarna från den senaste sessionen.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Övre fält

| Kontroll | Vad den gör |
| --- | --- |
| **Växla visningsläge**| Växlar mellan**rutnätsvy**(alla rutor som celler) och**listvy** (arrayer i full bredd högst upp, EN aktiv kamera nedanför). Verktygstips: ”Byt till rutnät” / ”Byt till listvy”. |
| **Rutnätslås**(hänglås) | Standardinställning**låst** — rutorna är låsta på plats. Lås upp för att dra och ordna om rutorna till valfri plats (mellanrum bevaras). Rutnätet låses automatiskt igen varje gång en ny kamera ansluts. Verktygstips: &quot;Lås upp rutnätet (aktivera dragning av rutor)&quot; / &quot;Lås rutnätet (frys rutorna på plats)”. |
| **Feed Zoom**-reglage | Rutstorlek, från 60 px upp till behållarens fulla bredd. Rutorna behåller ett 4:3-bildförhållande. Vid en rutbredd under 200 px döljs namnet och fps-överlagringarna för att hålla rutan ren. |

### Feed-rutor

Varje kamera renderar en sammansatt live-ruta; en kamera kan dessutom visa tre gråskaliga **kanaluppdelade** rutor (se [Kanaluppdelningar](#display-overlays-drawn-over-the-live-feed)), och matriser renderar en kombinerad ruta. Den aktiva rutan har en markeringsring i kamerans (eller matrisens) färg.

Om du håller muspekaren över en ruta visas en **X**-knapp för att stänga:

* Om du stänger en **sammansatt** ruta medan dess kanaluppdelningar fortfarande visas döljs endast den sammansatta rutan.
* Om du stänger den **sista synliga rutan för en fristående kamera** kopplas den kameran bort.
* **Delade rutor som ingår i en kombinerad matris kopplar aldrig bort** kameran — de döljs bara.

När rutnätet är olåst kan du dra vilken ruta som helst till valfri plats; layouten sparas med projektet.

## Sidopanel – kameralista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Den första sidan i sidopanelen visar alla anslutna kameror och arrayer:

* **Anslut kamera**(grön) /**Anslut array** (blå, visar ”Söker...” under skanning). Båda är inaktiverade medan en anslutningsdialogruta är öppen.
* **Spela in allt** (röd) — spelar in alla listade kameror med de exporttyper som valts i inspelningsinställningarna. Kräver ett öppet projekt. Fullständigt dokumenterat i [Inspelningsinställningar och lägen](capture.md).
* **Kugghjulet för inspelningsinställningar**(bredvid**Spela in allt**) — öppnar [fönstret Inspelningsinställningar](capture.md#the-capture-settings-pane). Inaktiverad utan projekt eller under inspelning.

### Kamerarader

Varje kamerarad visar en färgkodad ram (kamerans anpassade färg), en &quot;CAM&quot;-etikett — med en blå **M**(master) eller grönt**S** (slav) som rollbokstav för arraymedlemmar — samt visningsnamnet. Standardnamnet är `LATTICE-MODEL (serial)`; byt namn i inställningspanelen för varje kamera. Radknappar:

| Knapp | Effekt |
| --- | --- |
| **Öga**| Växla synlighet. Dolda kameror försvinner ur rutnätet och**utesluts från Capture All**. |
| **Kugghjul** | Öppna inställningsfönstret för varje kamera (nästa avsnitt). |
| **Paus / Spela**| Fryser liveförhandsvisningen**endast på skärmen** — inspelningen i bakgrunden fortsätter. Pausade kameror kan inte spela in. |
| **X** | Koppla från. Användargränssnittet uppdateras omedelbart (i bästa fall); det kan ta 10–30 sekunder för anslutningen att brytas i bakgrunden. |

### Arrayrader

En arrayrad visar en ”ARRAY”-etikett i arrayens färg, arrayens namn (kan bytas i arrayinställningarna) och en **DAQ · på/av**-knapp —**på** när ljussensorn på arraynivå är aktiverad *eller* om någon medlem har en kamera-specifik sensor; dess verktygstips visar exakt vilken sensor som matar vad. Medlemskamerorna listas indragna under med egna rader. Knappar i arrayraden: **öga**(döljer/visar ALLA medlemmar samtidigt),**kugghjul**(inställningsfönster för arrayen),**X**(kopplar bort hela arrayen).

Ljussensorns (DLS) status som används i arrayraderna och i inställningsfönstret för arrayen har fyra tillstånd:**av**,**väntar**(inget spektrum ännu),**aktiv**(ett spektrum har kommit in under de senaste 3 s), och**föråldrat** — inget nytt spektrum på 3 sekunder, men den senaste avläsningen *används fortfarande* (DAQ-avläsningar förfaller aldrig i insamlingsvägen).

Du kan dra fristående kameror och hela arraygrupper förbi varandra i sidofältet för att ändra ordningen i listan; arraymedlemmar kan inte dras separat.

## Inställningsfönster per kamera

Öppna med **kugghjulet** i en kamerarad. Panelen glider över kameralistan.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Rubrik**: kamerans**färgprov**(klicka för att öppna en inbyggd färgväljare – ställer in sidopanelens kant och färgen på valringen för rutorna),**namnet**med en penna och knappen**Byt namn**(om du sparar ett tomt namn återställs det till standardnamnet `MODEL (serial)`) samt**×** för att stänga.

### Live-histogram

Högst upp i fönstret finns ett live-luma-histogram som beräknas utifrån JPEG-förhandsvisningen med en frekvens på cirka 8 Hz. Medelvärdet är Bayer-vägt — (R+2G+B)/4 — för att matcha kamerans egen AE-mätning.

* **Orange streckad linje**= AE-målet.**Dra den horisontellt för att ändra målet** — ett kommando skickas när du släpper, och genom att dra växlar AE-målläget till Manuellt.
* **Green heldragen linje** = den faktiska medelvärdesluminansen (det som AE för närvarande levererar).
* **RGB-knappen** (uppe till höger): växlar mellan överlagrade bandvisa histogram färgade enligt kamerans filter (t.ex. på FRGN: grå NIR, grön, röd). På monokameror (M3M) står det ”MONO” på knappen och den är inaktiverad — mono visar alltid luma-histogrammet för ett enda band.
* Etiketterna på X-axeln följer sensorns bitdjup för det aktuella pixelformatet: 0..255, 0..1023, 0..4095 eller 0..65535.

### Rader

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

med kamerainformation | Rad | Beteende |
| --- | --- |
| **Modell** | Skrivskyddad (t.ex. `LATT-M3C-L87-FRGN`). |
| **Radiometrisk kalibrering**| Green**&quot;Aktiv&quot;**-märket med en bildtext som visar kalibreringsnivå, hash, kalibreringsdatum och bandlista, laddad från kamerans kalibreringspaket (se [Fabriksradiometrisk kalibrering](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Dold för RGB-kameror** — dessa har en fotometrisk vitbalanskalibrering, inte strålningsintensitet per band. |
| **Kalibreringsrapport**|**Hämta**-knapp — öppnar kamerans NIST-kalibreringscertifikat (per serienummer) som PDF-fil i ditt operativsystems visningsprogram. Om certifikatet ännu inte finns i cachen visar Chloros istället ett tips. |
| **Serienummer** | Skrivskyddat. |
| **Firmware**| Visar den aktuella versionen och hämtar sedan den tillgängliga versionen för denna modell (cachelagrad per modell — en grupp med N kameror kontrollerar servern en gång). Status: ”Kontrollerar…” →**”Uppdatera till X”**-knapp → ”Uppdaterar…” → ”Uppdaterad A → B” / ”Misslyckades: …” / ”Hoppades över: …” / grönt**”Uppdaterad”**. Verktygstips för uppdateringsknappen: ”Återställ till fabriksinställningar + flashning + omprogrammering av UserSet1. ~2–3 minuter; koppla inte bort.” |
| **IP** | Skrivskyddad. |
| **Temperatur** | Skrivskyddad, uppdateras var 3:e sekund. Blir orange vid ≥65 °C och röd med en ⚠ vid ≥75 °C. |
| Kryssrutan **Kalibreringsmål** | Aktiverar ArUco-detektering av reflektansmål med en valideringstabell per panel (NDVI) under liveflödet (listvy). Endast för sessionen – öppnas alltid avstängd. |
| **Ljussensor**-rullgardinsmeny | Kopplar en DAQ-ljussensor (DAQ-E/M/U, från listan på fliken Ljussensorer) till denna kamera för belysningskorrigering av nedåtriktat ljus (DLS) och prediktiv automatisk exponering. ”Ingen” avaktiverar kopplingen. Om inga sensorer är anslutna visar rullgardinsmenyn ”(inga sensorer anslutna – öppna fliken DAQ)”. Kopplingen sparas med projektet. |

### Exponering och förstärkning

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Alla numeriska inmatningar här använder rullgardinsmenyer som accelererar när man håller ned: tryck = ±1, håll ned &gt;1,5 s = ±10, håll ned &gt;3 s = ±100. Värdet skickas till kameran när du släpper.

| Kontroll | Intervall/alternativ | Standard | Gäller för | Vad den gör |
| --- | --- | --- | --- | --- |
| **Exponering (us)**| Kamerans min./max. i realtid | Auto | Alla | Exponeringstid i mikrosekunder, med en**Auto/Manuell**-omkopplare. Auto = kontinuerlig automatisk exponering på kamerasidan. |
| **Förstärkning (dB)**| Kamerans aktuella min/max (t.ex. upp till 48 dB) | Manuell (av) | Alla | Analog/digital förstärkning med egen**Auto/Manuell**-omkopplare. |
| **AE-måljashet**| 0–255 | 80, läge**Auto**| Alla (redigerbart när AE eller automatisk förstärkning är aktiverad) | Den jashet som AE siktar på. I**Auto**(standard) väljer en histogrambaserad bakgrundsregulator själv målet och håller exponeringen på 60–75 % av sensorns maximala värde. Om du skriver in ett värde eller drar i histogrammets orange linje växlar det till**Manuellt**. |
| **AE-utjämning** | 0,5–40, steg 0,1 | 8,0 | Alla | AE-dämpning. Verktygstips: ”Lägre = AE reagerar snabbare (kan pulsera vid höga bildfrekvenser). Högre = jämnare / långsammare.” Värden långt under standardvärdet kan få AE att pulsera och destabilisera strömningen vid höga bildhastigheter; 8,0 är det stabila standardvärdet. |
| **AE Region of Interest**| Kryssruta för aktivering +**Aim**-knapp | Av | Alla | När denna funktion är aktiverad mäter AE endast det streckadegröna området istället för hela bilden.**Sikta** aktiverar ”klicka för att placera” i liveflödet: ett klick centrerar ett område vid 30 % av bilden; klicka och dra för att rita en anpassad rektangel (minst 5 % × 5 %). *Aim* inaktiveras automatiskt efter en placering. Området mappas tillbaka till kamerans egna koordinater enligt den rotation/spegling du har ställt in och sparas med projektet. |
| **AE-inställning för hastighet** | 0,1–5, steg 0,1 | 1,0 | Endast för array-medlemmar | Hur snabbt det automatiska AE-målet spårar förändringar i scenens ljusstyrka; 1,0× kontrollerar på nytt var 2,5 s. |
| **Högdagsskydd** | Strikt (1 %) / Normal (5 %) / Lätt (15 %) | Strikt | Kameror som exponerar inställningen | Hur stor del av bilden som får klippas till vitt innan AE mörkar ner bilden. |

{% hint style="info" %}
**Ljuskrav för multispektrala Bayer-kameror (RGN / OCN / NGB):** scenen måste ha tillräckligt med ljus i alla tre kanalerna, annars fungerar inte kalibreringen korrekt — en enda sensorexponering täcker alla tre spektrum. Använd en DAQ-ljussensor för att mäta ljuset, eller använd helt mono (M3M) så att varje band får sin egen exponering. Om en bildtagning bryter mot detta upptäcker Chloros det och varnar dig (meddelandet om unmix-clamp).
{% endhint %}

### Pixelformat och

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

upplösning**Array-medlemmar** visar skrivskyddade rader för ”Current” (format + WxH) och ”Binning” med anmärkningen ”Ställs in i array-inställningarna” – en omstart av strömmen på en medlem skulle bryta synkroniseringen, så dessa hanteras i [panelen för array-inställningar](#array-settings-pane).**Fristående kameror** har följande:

| Kontroll | Alternativ | Vad det gör |
| --- | --- | --- |
| **Pixelformat** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Sensorns pixelformat (bitdjup). |
| **Upplösning** | Full / Halv / Kvart | Relativt den aktuella binningen: Full = 2048/N × 1536/N för N×N-binning. |
| **Binning** | 1x1 (ingen) / 2x2 / 4x4 | Hårdvarubinning N×N — högre värden ger lägre upplösning men förbättrar signal-brusförhållandet (SNR) och bildfrekvensen. Om du ändrar detta startas strömmen om och eventuella ROI återställs till det nya fullständiga synfältet. |
| **Aktuell** | skrivskyddad | Den faktiska bredd × höjd och (x, y)-förskjutningen som gäller. |

### Liveförhandsvisning

Allt i detta avsnitt gäller **endast visningssidan**— det ändrar vad du ser i liveflödet, medan sparade bilder förblir linjära och oförändrade — med ett undantag:**Vignett** är radiometrisk och påverkar även exporter (se nedan).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Kontroll | Intervall/alternativ | Standard | Gäller för | Vad den gör |
| --- | --- | --- | --- | --- |
| **Renderingsupplösning** | 360p (snabbaste) / 480p / 720p / 1080p / Sensorens ursprungliga upplösning (långsammast) | 720p | Alla | Den höjd vid vilken backend kör den radiometriska förhandsgranskningskedjan. Lägre värden ger högre bildfrekvens utan att synfältet ändras. |
| **Index**| Kryssruta för aktivering + kugghjul | Av | Endast Bayer-multispektrala,**inte** kombinerade matriser | Förhandsgranskning av vegetationsindex i realtid. Kugghjulet öppnar den delade [Indexberäknaren](#index-calculator-pane) som är förladdad med kamerans filter-naturliga band (t.ex. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Det anpassade uttrycket plus LUT (på/av, standardnivå 3, standardmin 0,2, standardmax 1) beräknas för varje förhandsgranskningsbildruta. Medlemmar i kombinerade matriser döljer denna rad — matrisen har ett gemensamt index. |
| **Vitbalans** | Av / En gång / Kontinuerlig + en knapp för ny mätning | Kontinuerlig | Endast RGB | Live-vitbalans. Uppdateringsknappen tar om vitbalansen från det aktuella DLS-spektrumet (inaktiverad när läget är Av). |
| **Gamma** | På / Av | På | Endast RGB | Visa gamma (γ = 2,2 LUT) i liveförhandsvisningen. Sparade bilder förblir linjära. |
| **Brusreducering** | Kryssruta + styrka 0–100 | Av / 50 | Alla (per kamera, även inom matriser) | Bilateralt filter i liveförhandsvisningen. Högre värde = jämnare men mjukare detaljer. |
| **Skärpa** | Kryssruta + styrka 0–100 | Av / 30 | Alla | Oskärpmask i liveförhandsvisningen, tillämpas sist. Kan förstärka brus. Endast förhandsvisning. |
| **Vignett**| Kryssruta + styrka 0–100 | Av / 0 | Alla | Manuell korrigering av kvarvarande vignettering (ljusar upp hörnen), läggs ovanpå bildseriens uppskattning av Smart Vignette.**Radiometrisk — påverkar både liveförhandsvisningen OCH exporten**, till skillnad från Denoise/Skärpa. |
| **Färgprofil** | Raw / Linjär / Naturlig / Förbättrad / Anpassad temperatur | Naturlig | Endast RGB | Se nedan. |
| **Färgtemperatur** | 2000–10000 K, steg 100 | 5500 K | Endast RGB, anpassad temperaturprofil | Låser vitbalansen till en fast korrelerad färgtemperatur (DLS-ingång ignoreras). Det senast valda Kelvin-värdet sparas även vid profilbyte. |
| **Mättnad** | 0–200 (100 = neutralt) | 100 | Endast RGB | HSV-mättnad i liveförhandsvisningen. |
| **Kontrast** | 0–200 (100 = neutral) | 100 | Endast RGB | Linjär kontrast runt mellangrått i liveförhandsvisningen. |
| **Spegla H / Spegla V** | Kryssrutor | Av | Alla | Spegla förhandsvisningen horisontellt / vertikalt. |
| **Rotation**| 0° / 90° / 180° / 270° | 0° | Alla | Rotera förhandsvisningen. Orienteringen tillämpas i slutet av förhandsvisningskedjan i backend —**sparade bilder behåller kamerans ursprungliga orientering**, och sammansatta vyer av bildmatriser ignorerar den. |**Färgprofilers betydelse** (RGB-kameror):

* **Raw** — kringgå bearbetningskedjan helt.
* **Linjär** — mörksignal + flat-field + vitbalans; ingen färgmatris, ingen gamma.
* **Natural** *(standard)* — linjär plus den uppmätta färgkorrigeringsmatrisen och en scenanpassad tonkurva.
* **Enhanced**— Natural plus vibrance och CLAHE-lokal kontrast. Den extra kostnaden gäller**endast liveförhandsgranskningen** — sparade bilder får alltid den fullständiga efterbehandlingen oavsett profil.
* **Anpassad temperatur** — Naturlig med vitbalansen fastställd till det Kelvin-värde du valt.

{% hint style="warning" %}
För Naturlig, Förbättrad och Anpassad temperatur visas en tonanmärkning i fönstret: bilderna ljusas upp utifrån sin egen scen, så sparade *visnings*bilder är inte jämförbara bild för bild. **Exportera strålningsintensitet eller reflektans för mätningar.**
{% endhint %}

### Överlagringar på skärmen (ritas över live-flödet)

Dessa finns endast i användargränssnittet — de läggs över videon och påverkar aldrig strömmen eller bilderna.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Överlagring | Kontroller | Standard | Funktion |
| --- | --- | --- | --- |
| **Zebra** | Kryssruta + tröskelvärde 200–255 | Av / 250 | Magenta diagonala ränder på klippta pixlar. |
| **Siktkors** | Kryssruta | Av | Markering för bildrutaens mitt. |
| **Rutnät** | Av / 3 × 3 / 9 × 9 | Av | Kompositionsrutnät. |
| **Histogram** | Kryssruta + bredd 0,10–0,90 av bildrutan | Av / 0,25 | En histogramremsa som visas under matningen. |
| **Fokusmarkering** | Kryssruta + tröskelvärde 20–200 + färgprov | Av / 80 / `#ff5722` | Sobel-kantmarkering för fokusering. |
| **Kanaluppdelningar** | &quot;Visa uppdelningar (Red / Green / NIR)&quot; / Knappen ”Dölj uppdelningar” | Dold | Lägger till tre oberoende gråskaliga rutor per kanal bredvid den sammansatta bilden (knapptexten följer kamerans filterkanaler). Varje uppdelad ruta kan dras och har samma kantfärg som kameran. Finns inte på monokameror. Sparas med projektet. |

### Punktmätare

* Kryssrutan **Klicka för att ta prov**: klicka på livebilden för att ta ett prov på en enskild pixel (ett hårkors markerar den), eller klicka och dra över ett område för att få ett pixelgenomsnitt.**Rensa**tar bort provet och hårkorset. Kan inte användas samtidigt med AE-ROI-läget**Sikta**.
* **Visa**-rullgardinsmenyn:**Raw (bitdjup)**— ursprungliga digitala värden med sensorns bitdjup (t.ex. 12-bit → 0..4095) — eller**Display (8-bit)** (standard). När ett liveindex är aktivt visar Display istället det beräknade indexvärdet (t.ex. NDVI).
* Avläsningspanelen visar pixelkoordinater, bildstorlek, pixelformat, bitdjup och en kanaltabell (Chan / Värde / %) med bandbeteckningar och våglängder; Bayer-gröna par är medelvärdesberäknade; regionprover visar ”N px avg”.

Spotmätarens status gäller endast för den aktuella sessionen.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktiv automatisk exponering (DLS-styrd)

Detta avsnitt visas endast när **minst en DAQ-ljussensor är ansluten** — lösaren behöver ett realtidsspektrum av nedåtriktat ljus för att driva funktionen.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Kontroll | Intervall | Standard | Funktion |
| --- | --- | --- | --- |
| **Aktivera** | Kryssruta | På (fristående kameror) | En lösare med slutna formler använder DLS-spektrumet plus skalärvärdena i kamerans kalibreringspaket för att få det ljusaste bandet nära mättnad samtidigt som det mörkaste bandet hålls över SNR-gränsen — en enda exponeringsskrivning per lösning, ingen stabiliseringsslinga. Utformad för solcellsdriven timelapse där varje bild måste vara korrekt exponerad. Backend faller tyst tillbaka till reaktiv AE när DLS-avläsningen är inaktuell/saknas eller kalibreringspaketet inte är laddat. |
| **Utjämning (α)** | 0,05–1,0, steg 0,05 | 0,3 | Utjämning av på varandra följande prediktiva lösningar (lägre värde = mjukare). |
| **Scenreflektans**|**Omkalibrera ρ**-knapp | — | Omberäknar den scenreflektansfaktor som lösaren använder. |

{% hint style="info" %}
**Array Connect stänger av den prediktiva AE:n som standard** — för arrayer hanterar Chloros:s smarta AE tillsammans med kamerans automatiska exponering (med mättnadskydd) och den prediktiva AE:ns enda uppskattning av scenreflektansen är inte tillförlitlig i blandade scener. Du kan återaktivera den per kamera här om du specifikt vill ha DLS-styrd radiometrisk exponering.
{% endhint %}

**DAQ-styrd exponeringsgräns och incidentljusbaserad AE.**Oberoende av kryssrutan ovan, när en DAQ-ljussensor tilldelas en RGB-kamera, beräknar Chloros — utifrån den uppmätta absoluta nedåtriktade irradians — den maximala exponering×förstärkning vid vilken en yta med 100 % reflektans förblir under klippgränsen, och tillämpar detta som ett**tak**för den automatiska exponeringen. Så länge takvärdet är aktivt är kameran**fastlåst på infallande ljus**: den körs i öppen slinga med den infallande ljusmätningen som exponering och förstärkningen på 0 dB — exponeringen följer det uppmätta ljuset, inte motivets innehåll. Eftersom takvärdet endast kan förkorta exponeringstiden kan det i sig inte orsaka klippning. Takvärdet inaktiveras automatiskt – och normal automatisk exponering för motivet återupptas – när DAQ-avläsningen saknas, är föråldrad (&gt;30 s) eller mörk, eller om ≥15 % av bilden klipps vid den låsta exponeringen (vilket innebär att sensorn och kameran uppfattar olika belysning). Det finns ingen GUI-knapp för detta; detta är standardbeteendet när en RGB-kamera har en DAQ-anslutning.

### Medlemmar i insamlings- och triggermatrisen

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

visar dessutom de skrivskyddade raderna **Roll**(Master i blått / Slave i grönt),**Synkroniseringslinje**och**Jämlikar**.

| Kontroll | Alternativ | Standard | Anmärkningar |
| --- | --- | --- | --- |
| **Triggerläge** | Av / På | På | Inaktiverat för arraymedlemmar (arrayen hanterar utlösningen). |
| **Triggerkälla** | Programvara / Linje 0 (M8) / Linje 1 / Linje 2 | Linje 0 | Dold när Triggerläge är Av; inaktiverad för arraymedlemmar. Linje 0 är den optoisolerade externa triggeringången på M8. |
| **Bildhastighet**| Auto / Manuell + värde | Auto |**Auto**: kamerans begränsning av bildhastigheten är avstängd — exponeringen avgör fps, och rutan visar den aktuella hastigheten i realtid.**Manuell**: du begränsar bildfrekvensen med ett skjutreglage (från 1 upp till det bandbreddsbegränsade maximivärdet), utifrån den aktuella faktiska frekvensen. Array-medlemmar ser ett skrivskyddat ”N fps (live)” med ”Ställ in i array-inställningarna”. |

### Nätverk / Transport

| Rad | Beteende |
| --- | --- |
| **Paketstorlek**| 1500 (Standard) / 9000 (Jumbo) — standardinställning**Jumbo**. |
| **Genomströmning** | Skrivskyddad gräns för länkgenomströmning i MB/s. Backend omfördelar detta mellan alla anslutna kameror vid varje anslutning/frånkoppling. |
| **Bufferhantering** | Skrivskyddat läge för bufferhantering. |

### Inspelning

Panelen avslutas med en **&quot;Öppna inspelningsinställningar…&quot;** som tar dig till [panelen Inspelningsinställningar](capture.md#the-capture-settings-pane) (inaktiverad tills ett projekt är öppet — ”Skapa eller öppna ett projekt för att spara inspelningar”). Om kameran är dold eller pausad påminner ett tips dig om att visa/återuppta innan du spelar in.

## Panelen Arrayinställningar

Öppnas med **kugghjulet**på en ARRAY-rad. Rubrik: arraynamn med en penna för att byta namn och**×** för att stänga. Avsnitt nedan märkta med *endast kombinerad* visas endast för arrayer som är anslutna i kombinerat visningsläge.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Synkronisering

Skrivskyddade rader för **Master**,**Slaves**och**Synkroniseringslinje**.

### Omgivningsljussensor

Visas för både kombinerade och separata matriser:

* Kryssrutan **Kalibreringsmål** — ”Upptäck MAPIR ArUco-mål och validera NDVI mot panelreflektans-LUT”; styr målöverlägget och valideringstabellen för den kombinerade rutan.
* Rullgardinsmenyn **Ljussensor** — kopplar en DAQ till hela matrisen. Valet träder i kraft omedelbart, sprids till varje enskild kameras egen rullgardinsmeny för ljussensor (du kan fortfarande åsidosätta inställningen per kamera) och börjar vidarebefordra spektra till matrisen.
* Rad för **status** i realtid: Av · &quot;Väntar på första spektrumet…&quot; · &quot;Aktiv — alla kameror i matrisen är belysningskorrigerade&quot; · &quot;Inget nytt spektrum under de senaste 3 sekunderna — använder fortfarande den senaste avläsningen (ingen timeout för inaktuella data)…&quot;.
* Anmärkning i fönstret: &quot;Radiometrisk korrigering för hela matrisen. Inställningar per kamera åsidosätter detta.&quot;

### Bildtagning — enhetliga sensorinställningar *(endast kombinerade)*

Dessa inställningar gäller enhetligt för alla enheter (ändringar per enhet skulle störa synkroniseringen). Ändringar lagras tillfälligt och tillämpas tillsammans.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Kontroll | Alternativ / intervall | Vad det gör |
| --- | --- | --- |
| **Pixelformat** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Enhetligt sensorformat för alla delar. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hårdvarubinning – behåller hela synfältet samtidigt som signal-brusförhållandet och bildfrekvensen förbättras. Om du ändrar detta återställs ROI-fälten till det nya fullständiga synfältet. |
| **Upplösning** (förinställning) | Full / Halv / Kvart | Relativt binning; fyller ROI-fälten med en centrerad beskärning. |
| **ROI-beskärning (px)**| W / H / X / Y numeriska fält | Sensorbeskärning. Bredd/höjd snäpper fast vid multiplar av 16 (minst 64); förskjutningarna snäpper fast vid multiplar av 4. Ett ”max BxH”-tips visar det övre gränsvärdet och**Återställ** återgår till fullt synfält. Under redigering ritas en orange förhandsgranskningsruta för beskärningen i realtid på matrisrutan (inklusive en schematisk bild av hela sensorn när beskärningen utvidgas utåt). |
| **Utlösningsfrekvens**| Välj mellan Auto / Manuell + fps 0,5–10, steg 0,5 |**Auto**(standard): backend beräknar utlösningsfrekvensen utifrån upplösning och bandbredd — inmatningsfältet är inaktiverat och visar det beräknade värdet.**Manuell**: låser värdet när du klickar på Tillämpa. |

Observera i fönstret: ”Ändringar av format/upplösning startar om alla kameror kortvarigt. Triggfrekvensen tillämpas direkt.” Knapparna **Apply / Cancel** finns längst ned i fönstret.

### Inriktning (samregistrering) *(endast kombinerad)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Restvärde**-rutan: ”RMS x,xx px” — grönt under 1 px, gult under 3 px, rött i övriga fall eller om någon kamera misslyckades; ”ingen profil” före den första lösningen.
* Sammanfattningsrad: ”✓ Alla kameror inriktade (N)” / ”⚠ p/N kameror inriktade — <serial (filter)="">misslyckades” / ”Beskärning aktiv — Kalibrera om för att inrikta (använder hela sensorn)” / ”Väntar på att exponeringen ska stabiliseras…”.
* Tabell per kamera: kamera (de sista 4 siffrorna i serienumret + filter), omprojiceringsfel i px med antal matchningar (&quot;ref&quot; för huvudkameran) och poängen för normaliserad korskorrelation av överlappningen mot gränsvärdet 0,35 för godkännande.
* **Omkalibrera inriktning**-knapp (visar ”Kalibrera inriktning” före den första profilen) — kör om samregistreringen på nya bildrutor.
* Kryssrutan **&quot;Autoexponera kameror för inriktning&quot;** (markerad som standard) — gör tillfälligt mörka eller platta kameror ljusare (först exponering, sedan förstärkning) så att de får textur att matcha, och återställer sedan AE.

Den kombinerade förhandsvisningen inriktas automatiskt vid öppning; kalibrera om fokus eller scenens djup har ändrats. Justeringen är **avsedd endast för den aktuella sessionen** – den sparas aldrig i en profil, eftersom den beror på scenavståndet just då. Bildserier kan fortfarande exporteras med pixelregistrering (se [Justerade exportfiler](capture.md#per-array-controls)).

### Smart vignett

* Kryssrutan **Aktivera korrigering**— tillämpar den kameraspecifika vignettberäkningen på den radiometriska kedjan (både i realtid**och** vid export).
* **Kalibrera från aktuell vy**— rikta först kameramatrisen mot ett enhetligt mål (platt skärm, vägg eller himmel); varje kamera jämnas ut individuellt och statusen rapporterar ”n/N kameror · −x,x %” utjämningsvinst.**Rensa** tar bort uppskattningen.
* Finjustera per kamera med skjutreglaget **Vignett** per kamera i [Liveförhandsgranskning](#live-preview).

### Liveförhandsgranskning *(endast kombinerad)** **Index**: aktivera kryssrutan + kugghjulet — öppnar den delade [Indexkalkylatorn](#index-calculator-pane) med band ritade från**alla** medlemskameror. En förhandsgranskningsrad för uttryck nedanför visar det aktuella uttrycket (&quot;Inget uttryck angivet — öppna kalkylatorn för att skapa ett&quot;), som uppdateras varje sekund.
* **Renderingsupplösning**i rullgardinsmenyn (samma förinställningar som per kamera, standard 720p): höjden på live-strömmen**och** storleken på den sparade sammansatta exporten. Observera i fönstret: ”Förhandsgranskning + storlek på sparad sammansättning. Bilder per kamera exporteras alltid i full upplösning.”

### Visningslager *(endast kombinerat)** Kryssrutan **Aktivera** (standardinställning avstängd — huvudkameran visas direkt; på = sammansatt i lager).
* Rullgardinsmenyerna **Förgrund**/**Bakgrund**: varje medlemskamera (efter namn) eller**Index**. När Förgrund är Index visar pixlar utanför LUT:s min-/maxvärden bakgrundslagret.

### Delad vy *(endast kombinerad)*

**”Visa medlemskameror”**— en knapp för**Dela / Dölj medlemskameror** som lägger till varje medlems egen live-feed som separata rutnätbrickor bredvid den sammansatta bilden. Rutorna läser av arrayens befintliga bildbuffert (ingen extra kameraanslutning). Endast rutnätsvy; sparas per array tillsammans med projektet.

### Funktioner

En skrivskyddad panel som uppdateras var 5:e sekund:

* **Nivåetikett**: ”Samtidig inspelning” (grön) · ”Samtidig inspelning (FTD-förskjuten utsändning)” (grön) · ”Förskjuten inspelning (100 ms avvikelse)” (gul) · ”Konfigurationen är för stor” (röd).
* **Bildramens status**: ”x,xx % ofullständig” — grön under 1 %, gul under 5 %, röd vid 5 % eller mer.
* **Länklinje**: ”NIC {mbps} Mbps – kontinuerlig {MB/s} MB/s”.

Detta är arrayens aktuella bandbreddsbudget. För den underliggande bildfrekvensen (fps) och nätverksmodellen – samt vad som ska ändras när nivån blir gul eller röd – se [Multikamera-arrayer](arrays.md) och [CLI-referensen](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Panelen ”Index Calculator”

Den tredje sidan i sidofältet, som delas av indexverktyget per kamera och indexverktyget för den kombinerade matrisen (ett i taget – rubriken lyder ”Index Calculator — <camera name="">” eller ”Index Calculator —<array name="">

&quot;). Här visas bandlistan (kamerans naturliga filterband eller alla band från alla medlemmar i matrisen), det aktuella uttrycket och LUT-konfigurationen (på/av, nivå — standard 3, min — standard 0,2, max — standard 1), samt ett live-indexhistogram. **Apply** bekräftar uttrycket; LUT-ändringar tillämpas direkt i förhandsgranskningen.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Inställningar per kamera kontra arrayhanterade inställningar

Snabböversikt över vad som finns var när en kamera ingår i en array:

| Arrayhanterat (skrivskyddat i kamerapanelen) | Fortfarande per kamera inom en array |
| --- | --- |
| Pixelformat, upplösning, binning | Autoexponering (exponering, förstärkning, mål, utjämning, ROI) |
| Utlösningsläge/källa, bildfrekvens | Brusreducering, skärpa, vinjettering |
| | Orientering (spegling/rotation), överlägg på skärmen, spotmätare |
| | Index (matriser med separat visning), koppling till ljussensor |

Övriga övergripande funktioner:

* **Kombinerad vs separat visning** väljs vid anslutning till matrisen: kombinerad = en justerad sammansatt ruta (medlemmarnas flöden visas endast via Split View); separat = varje medlem renderar sin egen synkroniserade ruta. En kamera visar aldrig både ett fristående flöde och en matrisruta.
* ****Automatisk återanslutning**: när ett sparat projekt öppnas återställs dess kameror och matriser, och alla sparade inställningar tillämpas på nytt på backend innan strömmarna återupptas.
* **Inspelningsbegränsning**: dolda eller pausade kameror exkluderas från ”Capture All”; en array blockeras helt endast när ALLA medlemmar är dolda/pausade. Se [Inspelningsinställningar och lägen](capture.md).

## Hur inställningarna bevaras

Kameraflikens tillstånd sparas **tillsammans med projektet**, inte i webbläsaren:

* Varje reaktiv förändring tar en ögonblicksbild av kamerorna och arrayerna i projektets `cameras.json` (efter en fördröjning på 500 ms). Detta omfattar kameranamn och färger, inställningar för exponering/förstärkning/AE, pixelformat/upplösning/binning, utlösningsfrekvens, förhandsgranskningsinställningar (renderingsupplösning, brusreducering, skärpa, vinjettering, färgprofil, mättnad/kontrast), orientering, överlägg, kanaluppdelningar, indexkonfiguration, inställningar för prediktiv AE, AE-ROI, matrisnamn, visningsläge, inställningar för matrisinspelning (inklusive ROI-beskärningsposition) samt rutnätsblocket (feed-zoom, visningsläge, rutnätslås, manuell kakelordning, dolda kameror, stängda kakelrutor, aktiv kamera).
* Ljussensorbindningar sparas i projektets `sensors.json`.
* När projektet öppnas på nytt återansluts hårdvaran och alla inställningar tillämpas på nytt.
* **Inget projekt öppet = endast sessionsbaserat**: utan projekt bevaras ingenting när Chloros stängs.
* Endast sessionsbaserat oavsett projekt: pausläge, spotmätningsprover, kryssrutan för kalibreringsmål per kamera (är alltid avmarkerad vid öppning) och matrisjusteringsprofilen (beräknas om per session enligt design).
* Ett undantag: **Inställningar för bildtagning**: val av export och bildtagningsläge sparas per projekt i appens lokala lagring istället för i `cameras.json` — se [Inställningar och lägen för bildtagning](capture.md).</array></camera></serial>
