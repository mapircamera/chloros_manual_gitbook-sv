# Kamerainställningar

Fliken **Kameror** är Chloros live-kontrollpanel för LATTICE-kameror: ett huvudområde som visar varje ansluten kamera som en live-ruta, samt en sidofält som växlar mellan tre sidor – **kameralistan**, ett**inställningsfönster**(inställningar per kamera, för kameragrupper eller för inspelning – en i taget) samt**Index Calculator**. Denna sida beskriver alla reglage i kameralistan, inställningspanelen per kamera och inställningspanelen för kameragrupper. Inspelningslägen, val av exporttyp och flödet ”Capture All” finns på den tillhörande sidan [Inspelningsinställningar och lägen](capture.md).

Fliken ”Kameror” visas i sidofältet när Chloros-backenden är klar. Alla kontroller nedan kommunicerar med den lokala backenden via `127.0.0.1:5000`; ändringar tillämpas omedelbart på livekameran om inte annat anges.

## Kameratyper som används på denna sida

Kontrollerna visas eller döljs beroende på vilken typ av kamera som är vald. I manualen används följande termer genomgående:

| Term | Betydelse | Filterkanaler |
| --- | --- | --- |
| **RGB-kamera** | LATTICE M3C med FRGB-filter (modellen innehåller `-FRGB`) | Red / Green / Blue |
| **Bayer-multispektral** | LATTICE M3C med FRGN, FOCN eller FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — ett smalbandsfilter, ett kalibrerat band | Enstaka band |
| **Arraymedlem** | En kamera ansluten som del av en synkroniserad array (kombinerad eller separat visning) | Enligt sitt filter |

 RGB-kameror genomgår fotometrisk bearbetning (vitbalans, färgprofiler, gamma); multispektrala och monokameror genomgår den radiometriska kedjan och hoppar över de fotometriska inställningarna. Arraymedlemmar överför inställningar på strömnivå (pixelformat, upplösning, binning, utlösare, bildhastighet) till arrayen — dessa rader blir skrivskyddade i rutan per kamera och flyttas istället till rutan för arrayinställningar.

## Huvudområdet för flödet

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->



Om inga kameror är anslutna visar flödesområdet en startskärm med texten **”Anslut en kamera för att komma igång”**och två knappar:**Anslut kamera**(grön, öppnar anslutningsdialogrutan för en enskild kamera) och**Anslut array** (blå, öppnar anslutningsdialogrutan för en array). Själva anslutningsdialogrutorna beskrivs i [Ansluta kameror](connecting.md); begrepp relaterade till arrayer (synkronisering, nivåer, bandbredd) i [Kameragrupper med flera kameror](arrays.md). När du öppnar ett sparat projekt som innehåller kameror visas istället en laddningsindikator med texten ”Öppnar N sparade kameror på nytt…” medan Chloros återställer strömmarna från den senaste sessionen.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Övre fält

| Kontroll | Vad den gör |
| --- | --- |
| **Växla mellan visningslägen**| Växlar mellan**rutnätsvy**(alla rutor som celler) och**listvy**(rader i full bredd: varje kombinerad uppsättning som en sammansatt rad, varje medlem i en**separat** uppsättning som en egen rad med sin egen ström och kalibreringsmåltabell, och sedan EN fristående kamera – den aktiva – nedan). Verktygstips: ”Byt till rutnätsvy” / ”Byt till listvy”. |
| **Rutnätslås**(hänglås) | Standardinställning**låst** – rutorna är fastlåsta på plats. Lås upp för att dra och ordna om rutorna till valfri plats (mellanrum bevaras). Rutnätet låses automatiskt igen varje gång en ny kamera ansluts. Verktygstips: ”Lås upp rutnätet (aktivera dragning av rutor)” / ”Lås rutnätet (frys rutorna på plats)”. |
| **Feed Zoom**-skjutreglage | Rutstorlek, från 60 px upp till behållarens fulla bredd. Cellerna behåller bildförhållandet 4:3. Vid en cellbredd under 200 px döljs namnet och fps-överlagringarna för att hålla rutan ren. |

### Feed-rutor

Varje kamera renderar en sammansatt live-ruta; en kamera kan dessutom visa tre gråskaliga **kanaluppdelade** rutor (se [Kanaluppdelningar](#display-overlays-drawn-over-the-live-feed)), och arrayer renderar en kombinerad ruta. Den aktiva rutan har en markeringsring i kamerans (eller arrayens) färg.

När du håller muspekaren över en ruta visas en **X**-stängningsknapp:

* Om du stänger en **sammansatt** ruta medan dess kanaluppdelningar fortfarande visas döljs endast den sammansatta rutan.
* Om du stänger den **sista synliga rutan för en fristående kamera** kopplas den kameran bort.
* **Uppdelade rutor för en del av en kombinerad matris kopplar aldrig bort** kameran – de döljs endast.

När rutnätet är olåst kan du dra vilken ruta som helst till valfri plats; layouten sparas tillsammans med projektet.

## Sidopanel – kameralista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Den första sidan i sidopanelen visar alla anslutna kameror och matriser:

* **Anslut kamera**(grön) /**Anslut array** (blå, visar ”Söker efter...” under skanning). Båda är inaktiverade medan en anslutningsdialog är öppen.
* **Spela in allt** (röd) — spelar in alla listade kameror med de exporttyper som valts i inspelningsinställningarna. Kräver ett öppet projekt. Fullständig dokumentation finns i [Inspelningsinställningar och lägen](capture.md).
* **Kugghjulet för inspelningsinställningar** (bredvid ”Spela in allt”) — öppnar [panelen Inspelningsinställningar](capture.md#the-capture-settings-pane). Inaktiverad utan projekt eller under inspelning.

### Kamerarader

Varje kamerarad visar en färgkodad kant (kamerans anpassade färg), en ”CAM”-etikett — med en blå **M**(master) eller grönt**S** (slav) som rollbokstav för arraymedlemmar — samt visningsnamnet. Standardnamnet är `LATTICE-MODEL (serial)`; döp om det från inställningspanelen för varje kamera. Radknappar:

| Knapp | Effekt |
| --- | --- |
| **Öga**| Växla synlighet. Dolda kameror försvinner ur rutnätet och**utesluts från Capture All**. |
| **Kugghjul** | Öppna inställningsfönstret för varje kamera (nästa avsnitt). |
| **Paus / Spela upp**| Fryser liveförhandsvisningen**endast på skärmsidan** — bakgrundsprocessen fortsätter att köra. Pausade kameror kan inte spela in. |
| **X** | Koppla från. Användargränssnittet uppdateras omedelbart (i bästa fall); det kan ta 10–30 sekunder för backend-avkopplingen att slutföras. |

### Arrayrader

En arrayrad visar en ”ARRAY”-etikett i arrayens färg, arrayens namn (kan bytas i arrayinställningarna) och en **DAQ · på/av**-knapp —**på** när ljussensorn på arraynivå är aktiverad *eller* om någon medlem har en kamera-specifik sensor; dess verktygstips visar exakt vilken sensor som matar vad. Medlemskamerorna listas indragna nedanför med egna rader. Knappar i matrisraden: **öga**(döljer/visar ALLA medlemmar samtidigt),**kugghjul**(inställningsfönster för matrisen),**X**(kopplar bort hela matrisen).

Ljussensorns (DLS) status som används i matrisraderna och inställningsfönstret för matrisen har fyra tillstånd:**av**,**väntar**(inget spektrum ännu),**aktiv**(ett spektrum har anlänt inom de senaste 3 sekunderna) och**föråldrad** — inget nytt spektrum på 3 sekunder, men den senaste avläsningen *används fortfarande* (DAQ-avläsningar förfaller aldrig i insamlingsvägen).

Tills du ändrar ordningen visar listan först matriser, sedan fristående kameror, med kameror (och varje matris medlemmar) i **kamerordning**: M3C (Bayer) före M3M (mono); bland M3C, FRGB, FRGN, FOCN och sedan FNGB; M3M efter filtervåglängd, med den lägsta först. Rutnätets standardlayout följer samma ordning. Du kan dra fristående kameror och hela arraygrupper förbi varandra i sidofältet för att ändra ordningen i listan; arraymedlemmar kan inte dras separat.

## Inställningsfönster per kamera

Öppnas med **kugghjulet** i en kamerarad. Fönstret glider över kameralistan.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Rubrik**: kamerans**färgprov**(klicka för att öppna ett inbyggt färgväljare — ställer in sidopanelens kant och färgen på valringen för rutorna),**namnet**med en penna och knappen**Byt namn**(om du sparar ett tomt namn återställs det till standardnamnet `MODEL (serial)`) samt**×** för att stänga.

### Live-histogram

Högst upp i fönstret finns ett live-luma-histogram som beräknas utifrån JPEG-förhandsvisningen med en frekvens på cirka 8 Hz. Medelvärdet är Bayer-vägt — (R+2G+B)/4 — för att matcha kamerans egen AE-mätning.

* **Den Orange a streckade linjen**= AE-målet.**Dra den horisontellt för att ändra målet** — ett kommando skickas när du släpper, och genom att dra växlar AE-målläget till Manuellt.
* **Den Green a heldragna linjen** = det faktiska medelvärdet för luma (det som AE för närvarande levererar).
* **RGB-knapp** (uppe till höger): växlar mellan överlagrade bandvisa histogram färgade enligt kamerans filter (t.ex. på FRGN: grått NIR, grönt, rött). På monokameror (M3M) står det ”MONO” på knappen och den är inaktiverad — mono visar alltid det bandvisa luminanshistogrammet.
* Etiketterna på X-axeln följer sensorns bitdjup för det aktuella pixelformatet: 0..255, 0..1023, 0..4095 eller 0..65535.

### Rader

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

med kamerainformation

| Rad | Beteende |
| --- | --- |
| **Modell** | Skrivskyddad (t.ex. `LATT-M3C-L87-FRGN`). |
| **Radiometrisk kalibrering** | Green**"Aktiv"**-märkning med en bildtext som visar kalibreringsnivå, hash, kalibreringsdatum och bandlista, hämtade från kamerans kalibreringspaket (se [Fabriksradiometrisk kalibrering](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Dolt för RGB-kameror** — de har en fotometrisk vitbalanskalibrering, inte strålningsintensitet per band. |
| **Kalibreringsrapport**|**Hämta**-knapp — öppnar kamerans NIST-kalibreringscertifikat (per serienummer) som PDF i ditt operativsystems visningsprogram. Om certifikatet ännu inte finns i cachen visar Chloros istället ett tips. |
| **Serienummer** | Skrivskyddat. |
| **Firmware**| Visar den aktuella versionen och hämtar sedan den tillgängliga versionen för denna modell (cachelagrad per modell — en grupp med N kameror kontrollerar servern en gång). Status: ”Kontrollerar…” →**”Uppdatera till X”**-knapp → ”Uppdaterar…” → ”Uppdaterad A → B” / ”Misslyckades: …” / ”Hoppades över: …” / grönt**”Uppdaterad”**. Verktygstips för uppdateringsknappen: ”Återställ till fabriksinställningar + uppdatering + omprogrammering av UserSet1. ~2–3 minuter; koppla inte bort.” |
| **IP** | Skrivskyddad. |
| **Temperatur** | Skrivskyddad, uppdateras var 3:e sekund. Blir orange vid ≥65 °C och röd med en ⚠ vid ≥75 °C. |
| **Kalibreringsmål** kryssruta | Aktiverar ArUco-reflektansmåldetektering med en NDVI-valideringstabell per panel under liveflödet (listvy) — se [Tabellen för kalibreringsmål](#the-calibration-target-table). Endast för sessionen — öppnas alltid avstängd. |
| **Ljussensor** rullgardinsmeny | Kopplar en DAQ-ljussensor (DAQ-E/M/U, från listan på fliken Ljussensorer) till denna kamera för belysningskorrigering av nedåtriktat ljus (DLS) och prediktiv automatisk exponering. "”Ingen” avmarkerar kopplingen. Om inga sensorer är anslutna visar rullgardinsmenyn ”(inga sensorer anslutna – öppna fliken DAQ)”. Kopplingen sparas med projektet. |

#### Tabellen Kalibreringsmål

Tabellen är endast avsedd för **validering** ("endast validering — ändrar inte kalibreringen"). Rubrikerna anger kamerans serienummer, filter, målnummer och markörstorlek i pixlar, och den har en rad per panel:

* **Panel**, följt av en kolumn per band med panelens värde — en Bayer-kamera visar sina tre band med namn och våglängd (t.ex. FRGN: Red 660 nm / Green 550 nm / NIR 850 nm); en **mono (M3M)-kamera visar sitt enda band** (t.ex. `F750`) och har inga NDVI-kolumner.
* **Sat** — andelen av panelens pixlar som är mättade (gult från 1 %, rött från 5 %).
* **NDVI Meas**,**NDVI Theo** (panelens referensreflektans-NDVI) och **Δ** — endast Bayer-multispektralkameror. En liten markering efter ”NDVI Meas” anger vad den uppmätta NDVI:n har beräknats utifrån: **ρ**(reflektans — ett ljussensorvärde finns tillgängligt),**L**(strålning — inget värde från ljussensorn) eller**DN**(råvärden från kameran). Endast ett**ρ**-NDVI-värde är jämförbart med NDVI Theo: för L och DN är Δ-kolumnen gråmarkerad och en anteckning anger detta (”Inget värde från ljussensorn: NDVI är baserat på strålningsintensitet, så Δ är ingen kalibreringskontroll” / ”NDVI är baserat på råa kameravärden, så Δ är ingen kalibreringskontroll”). Med ρ är Δ grönt under 0,02, gult under 0,05 och rött i övriga fall.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Exponering & förstärkning

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Alla numeriska inmatningar här använder spinnare som accelererar vid långt tryck: tryck = ±1, håll ned >1,5 s = ±10, håll ned >3 s = ±100. Värdet skickas till kameran när du släpper.

| Kontroll | Intervall / alternativ | Standard | Gäller för | Vad den gör |
| --- | --- | --- | --- | --- |
| **Exponering (us)**| Kamerans aktuella min/max | Auto | Alla | Exponeringstid i mikrosekunder, med en**Auto/Manuell**omkopplare. Auto = kontinuerlig automatisk exponering på kamerasidan. Om du ställer in manuell exponering förblir**Förstärkning** på Auto om den var det tidigare, vilket gör att du fritt kan använda kamerans hela förstärkningsområde tills du redigerar förstärkningen eller ställer in den på Manuell. |
| **Förstärkning (dB)**| Kamerans aktuella min/max (t.ex. upp till 48 dB) | Manuell (av) | Alla | Analog/digital förstärkning med egen**Auto/Manuell**-omkopplare. |
| **AE-målbelysning**| 0–255 | 80, läge**Auto**| Alla (redigerbar när AE eller automatisk förstärkning är på) | Den belysning som AE siktar på. I**Auto**(standard) väljer en histogrambaserad bakgrundskontroll själv målet och håller exponeringen på 60–75 % av sensorns maxvärde. Om du skriver in ett värde eller drar i histogrammets orange linje växlar det till**Manuell**. |
| **AE-utjämning** | 0,5–40, steg 0,1 | 8,0 | Alla | AE-dämpning. Verktygstips: "Lägre = AE reagerar snabbare (kan pulsera vid hög bildfrekvens). Högre = jämnare / långsammare.” Värden långt under standardvärdet kan få AE att pulsera och destabilisera strömningen vid höga bildfrekvenser; 8,0 är det stabila standardvärdet. |
| **AE-område av intresse**| Kryssruta för aktivering +**Aim**-knapp | Av | Alla | När den är aktiverad mäter AE endast det gröna streckade området istället för hela bilden.**Sikta** aktiverar ”klicka för att placera” i liveflödet: ett klick centrerar ett område vid 30 % av bilden; klicka och dra för att rita en anpassad rektangel (minst 5 % × 5 %). *Aim* inaktiveras automatiskt efter en placering. Området mappas tillbaka till kamerans egna koordinater enligt den rotation/spegling du har ställt in och sparas med projektet. |
| **AE-inställningshastighet** | 0,1–5, steg 0,1 | 1,0 | Endast för array-medlemmar | Hur snabbt det automatiska AE-målet spårar förändringar i scenens ljusstyrka; 1,0× kontrollerar på nytt var 2,5:e sekund. |
| **Highlight Protection** | Strikt (1 %) / Normal (5 %) / Lätt (15 %) | Strikt | Kameror som exponerar inställningen | Hur stor del av bilden som får klippas till vitt innan AE mörkar ner bilden. |

{% hint style="info" %}
**Ljuskrav för multispektrala Bayer-kameror (RGN / OCN / NGB):** scenen måste ha tillräckligt med ljus i alla tre kanalerna, annars fungerar inte kalibreringen korrekt — en enda sensorexponering täcker alla tre spektrum. Använd en DAQ-ljussensor för att mäta ljuset, eller använd helt mono (M3M) så att varje band får sin egen exponering. Om en bildtagning bryter mot detta upptäcker Chloros det och varnar dig (meddelandet om avklämning vid avblandning). I realtid visar en kanaluppdelad ruta **”Banddata med låg tillförlitlighet · N %”** när unmix har klämt fast det bandet på 25 % eller mer av dess pixlar.
{% endhint %}

{% hint style="info" %}
**Hur automatisk exponering fördelar exponeringstid och förstärkning (1.3.0).** När förstärkningen är inställd på Auto håller Chloros förstärkningen på 0 dB tills exponeringen har nått det längsta tillåtna värdet (AE:s övre gräns, en matris utlösningsperiod eller DAQ:s exponeringsgräns); först då ökar förstärkningen. I en matris har utlösningsfrekvensen företräde: automatisk exponering fyller den period som frekvensen anger, och höjer sedan förstärkningen, så en mörknande scen sänker inte längre en automatisk utlösningsfrekvens (i 1.2.0 gjorde den det). En **manuell**exponering på en arraymedlem som är längre än utlösningsperioden saktar ner arrayens utlösning för att anpassa den; en automatisk frekvens återkommer så snart varje enhet som saktade ner den återgår till automatisk exponering. När både**Exponering**och**AE-målbelysning** är inställda på Auto och en DAQ-ljussensor ger ett mätvärde, hålls varje kameras exponering oförändrad så snart den automatiska exponeringen har stabiliserats, så länge det uppmätta ljuset ligger inom cirka ±25 %; en större, ihållande ljusförändring, en omstart av inställningarna (till exempel en ROI-ändring) eller din egen exponeringsändring aktiverar den på nytt.
{% endhint %}

{% hint style="info" %}
**Hur automatisk exponering fungerar (1.3.0).**

* Vid manuell exponering förblir förstärkningen inställd på Auto tills du ändrar förstärkningen.
* Små högdagrar gör inte scenen mörkare: en spegling på upp till 1 % av bilden ignoreras, och i panelerna på ett känt kalibreringsmål ignoreras upp till 0,5 % av en panel.
* DAQ-exponeringstaket gäller endast så länge ett kalibreringsmål är i bild; om ingen panel syns exponerar kameran för scenen.
* Exponeringen hålls stabil vid konstant ljus och stabiliseras inom några sekunder efter en stor förändring. En överexponering begränsar inte längre exponeringen permanent, och vid en förändring av intresseområdet (ROI) återställs exponeringen.
* Ljusstyrkemålet överskrider inte kameransräckvidd, så ett återkommande ljus orsakar inte reflexer i bilden.
* En kamera vars automatiska exponering slutar svara i starkt solsken hålls på den nivå som motsvarar det ljus den befinner sig i, avläst av ljussensorn eller, om ingen är ansluten, av kamerans egna bilder, och släpper när ljuset förändras. En mörk eller täckt ljussensor stör inte detta.
{% endhint %}

### Pixelformat och upplösning

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Array-medlemmar** visar skrivskyddade rader för ”Aktuell” (format + BxH) och ”Binning” med anmärkningen ”Ställs in i array-inställningarna” — en omstart av strömmen på en medlem skulle bryta synkroniseringen, så dessa hanteras i [panelen för array-inställningar](#array-settings-pane).**Fristående kameror** har:

| Kontroll | Alternativ | Vad den gör |
| --- | --- | --- |
| **Pixelformat** | 8-bit / 10-bit / 12-bit / 16-bit | Sensorns bitdjup; formatfamiljen är sensorns. Standardvärdet är 12-bit, och ett projekt sparar den inställning du väljer, inklusive 8-bit, och återansluter med den. |
| **Upplösning** | Full / Halv / Kvart | Relativt den aktuella binningen: Full = 2048/N × 1536/N för N×N-binning. |
| **Binning** | 1x1 (ingen) / 2x2 / 4x4 | Hårdvarubinning N×N — högre värden ger lägre upplösning men förbättrar signal-brusförhållandet (SNR) och bildfrekvensen. Om du ändrar detta startas strömmen om och eventuella ROI återställs till det nya fulla synfältet. |
| **Nuvarande** | skrivskyddad | Den faktiska WxH och (x, y)-förskjutningen som gäller. |

### Liveförhandsgranskning

Allt i detta avsnitt gäller **endast visningssidan**— det ändrar vad du ser i liveflödet, medan sparade bilder förblir linjära och oförändrade — med ett undantag:**Vignett** är radiometrisk och påverkar även exporter (se nedan).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Kontroll | Intervall/alternativ | Standard | Gäller för | Vad den gör |
| --- | --- | --- | --- | --- |
| **Renderingsupplösning** | 360p (snabbaste) / 480p / 720p / 1080p / Sensorens ursprungliga upplösning (långsammaste) | 720p | Alla | Den höjd vid vilken backend kör den radiometriska förhandsgranskningskedjan. Lägre värden ger högre bildfrekvens utan att synfältet ändras. |
| **Index**| Kryssruta för aktivering + kugghjul | Av | Endast Bayer-multispektralt,**inte** kombinerade matriser | Förhandsgranskning av vegetationsindex i realtid. Kugghjulet öppnar den delade [Indexberäknaren](#index-calculator-pane) som är förladdad med kamerans filter-naturliga band (t.ex. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Det anpassade uttrycket plus LUT (på/av, standardnivå 7-stop, standardmin 0,2, standardmax 1) beräknas för varje förhandsgranskningsbild. Medlemmar i den kombinerade matrisen döljer denna rad — matrisen har ett gemensamt index. |
| **Vitbalans** | Av / En gång / Kontinuerligt + en knapp för ny mätning | Kontinuerligt | Endast RGB | Live-vitbalans. Uppdateringsknappen återinsamlar vitbalansen från det aktuella DLS-spektrumet (inaktiverad när läget är Av). |
| **Gamma** | På / Av | På | Endast RGB | Visar gamma (γ = 2,2 LUT) i liveförhandsvisningen. Sparade bilder förblir linjära. |
| **Brusreducering** | Kryssruta + styrka 0–100 | Av / 50 | Alla (per kamera, även inom matriser) | Bilateralt filter i liveförhandsvisningen. Högre värde = jämnare men mjukare detaljer. |
| **Skärpa** | Kryssruta + styrka 0–100 | Av / 30 | Alla | Oskärpmask i liveförhandsvisningen, tillämpas sist. Kan förstärka brus. Endast i förhandsvisningen. |
| **Vignett**| Kryssruta + styrka 0–100 | Av / 0 | Alla | Manuell korrigering av kvarvarande vignett (ljusar upp hörnen), läggs ovanpå bildseriens uppskattning av smart vignett.**Radiometrisk — påverkar både liveförhandsvisningen OCH exporten**, till skillnad från brusreducering/skärpa. |
| **Färgprofil** | Raw / Linjär / Naturlig / Förbättrad / Anpassad temperatur | Naturlig | Endast RGB | Se nedan. |
| **Färgtemperatur** | 2000–10000 K, steg 100 | 5500 K | Endast RGB, anpassad temperaturprofil | Låser vitbalansen till en fast korrelerad färgtemperatur (DLS-inmatning ignoreras). Det senast valda Kelvin-värdet sparas även vid profilbyten. |
| **Högdagrar** | −100 till 0, steg 1 | 0 (av) | Endast RGB | Nytt i 1.3.0. Mörkar ner de ljusaste, vitaste områdena i bilden (0 = oförändrat, −100 = starkast). Gäller för liveförhandsvisningen **och för bilder tagna på skärmnivå (Förhandsvisning)**, som sparar inställningen (`Chloros:DisplayHighlights`); export av strålnings- och reflektansvärden påverkas aldrig. Det går inte att återställa sensordata som har klippts – sänk exponeringen för att undvika detta. Zebraöverlägget följer den mörkare bilden. Inaktiveras med Raw-färgprofilen (”Tillämpas inte med Raw-färgprofilen.”). |
| **Mättnad** | 0–200 (100 = neutral) | 100 | Endast RGB | HSV-mättnad i liveförhandsvisningen. |
| **Kontrast** | 0–200 (100 = neutral) | 100 | Endast RGB | Linjär kontrast runt mellangrått i liveförhandsvisningen. |
| **Spegla H / Spegla V** | Kryssrutor | Av | Alla | Spegla förhandsvisningen horisontellt/vertikalt. |
| **Rotation**| 0° / 90° / 180° / 270° | 0° | Alla | Rotera förhandsvisningen. Orienteringen tillämpas i slutet av förhandsgranskningskedjan i backend.**Sparade bilder (1.3.0):**Förhandsgranskning, Radiance, Reflectance och Index-produkter från en fristående kamera vänds med Spegling/Rotation, medan**Raw och Debayered behåller kamerans ursprungliga orientering**; varje bildfil sparar inställningen i sin XMP, så att bearbetning av en raw-fil senare tillämpar den på samma sätt.**Filer i en array behåller sin ursprungliga orientering** (inställningen sparas, men tillämpas inte), och sammansatta vyer i en array ignorerar den. |

**Semantik för färgprofiler** (RGB-kameror):

* **Raw** — kringgår bearbetningskedjan helt.
* **Linjär** — mörksignal + flat-field + vitbalans; ingen färgmatris, ingen gamma.
* **Naturlig** *(standard)* — linjär plus den uppmätta färgkorrigeringsmatrisen och en scenanpassad tonkurva.
* **Förbättrad**— Naturlig plus livfullhet och lokal kontrast (CLAHE). Den extra kostnaden gäller**endast liveförhandsvisningen** — sparade bilder får alltid den fullständiga efterbehandlingen oavsett profil.
* **Anpassad temperatur** — Naturlig med vitbalansen fastställd till det Kelvin-värde du valt.

{% hint style="warning" %}
För Natural, Enhanced och Custom Temperature visas en tonanmärkning i fönstret: bildrutorna ljusas upp utifrån sin egen scen, så sparade *visnings*bilder är inte jämförbara bildruta för bildruta. **Exportera strålning eller reflektans för mätningar.**
{% endhint %}

### Visningsöverlägg (ritas över livebilden)

Dessa finns endast i gränssnittet — de ritas över videon och påverkar aldrig strömmen eller inspelningarna.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Överlägg | Kontroller | Standard | Vad det gör |
| --- | --- | --- | --- |
| **Zebra** | Kryssruta + tröskelvärde 200–255 | Av / 250 | Magenta diagonala ränder på beskärda pixlar. |
| **Siktkors** | Kryssruta | Av | Markering för bildruts mittpunkt. |
| **Rutnät** | Av / 3 × 3 / 9 × 9 | Av | Kompositionsrutnät. |
| **Histogram** | Kryssruta + bredd 0,10–0,90 av bilden | Av / 0,25 | En histogramremsa i strömmen. |
| **Fokuspeak** | Kryssruta + tröskelvärde 20–200 + färgprov | Av / 80 / `#ff5722` | Sobel-kantmarkering för fokusering. |
| **Kanaluppdelningar** | "Visa uppdelningar (Red / Green / NIR)" / Knappen ”Dölj uppdelningar” | Dold | Lägger till tre oberoende gråskaliga rutor per kanal bredvid den sammansatta bilden (knapptexten följer kamerans filterkanaler). Varje uppdelad ruta kan dras och har samma kantfärg som kameran. Ej tillgängligt på monokameror. Sparas med projektet. |

### Spotmätare

* Kryssrutan **Klicka för att ta prov**: klicka på livebilden för att ta ett prov på en enskild pixel (ett hårkors markerar den), eller klicka och dra över ett område för att få ett pixelgenomsnitt.**Rensa**tar bort provet och hårkorset. Kan inte användas samtidigt med AE-ROI-läget**Sikta**.
* **Välj**Visa**i rullgardinsmenyn:**Raw (bitdjup)**— ursprungliga digitala värden med sensorns bitdjup (t.ex. 12-bit → 0..4095) — eller**Display (8-bit)** (standard). När ett liveindex är aktivt visar Display istället det beräknade indexvärdet (t.ex. NDVI).
* Avläsningspanelen visar pixelkoordinater, bildstorlek, pixelformat, bitdjup och en kanaltabell (Chan / Värde / %) med bandbeteckningar och våglängder; Bayer-gröna par är medelvärdesberäknade; regionprover visar ”N px avg”.

Spotmätarens status gäller endast för den aktuella sessionen.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktiv automatisk exponering (DLS-styrd)

Detta avsnitt visas endast när **minst en DAQ-ljussensor är ansluten** — lösaren behöver ett realtidsspektrum av nedåtriktat ljus för att driva funktionen.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Kontroll | Intervall | Standard | Funktion |
| --- | --- | --- | --- |
| **Aktivera** | Kryssruta | På (fristående kameror) | En lösare med sluten form använder DLS-spektrumet plus skalärvärdena från kamerans kalibreringspaket för att få det ljusaste bandet nära mättnad samtidigt som det mörkaste bandet hålls över SNR-gränsen — en enda exponering per lösning, ingen stabiliseringsslinga. Utformad för solcellsdriven timelapse där varje bild måste vara korrekt exponerad. Backend faller tyst tillbaka till reaktiv AE när DLS-avläsningen är inaktuell/saknas eller kalibreringspaketet inte är laddat. |
| **Utjämning (α)** | 0,05–1,0, steg 0,05 | 0,3 | Utjämning av på varandra följande prediktiva lösningar (lägre värde = mjukare). |
| **Scenreflektans**|**Omkalibrera ρ**-knapp | — | Omberäknar den scenreflektansfaktor som lösaren använder. |

{% hint style="info" %}
**Array Connect stänger av den prediktiva AE som standard** — för arrayer hanteras exponeringen av Chloros smarta AE samt kamerans automatiska exponering (med mättnadskydd), och den prediktiva AE:s-reflektansuppskattning inte säker i blandade scener. Du kan återaktivera den per kamera här om du specifikt vill ha DLS-driven radiometrisk exponering.
{% endhint %}

**DAQ-driven exponeringsgräns och incident-pinned AE.** Oavsett kryssrutan ovan, när en DAQ-ljussensor tilldelas en RGB-kamera, beräknar Chloros — utifrån den uppmätta absoluta nedåtriktade irradians — den maximala exponeringen × förstärkningen vid vilken en yta med 100 % reflektans förblir under klippningsgränsen, och tillämpar den som ett **tak**för den automatiska exponeringen. Sedan version 1.3.0 gäller taket endast så länge ett kalibreringsmål befinner sig i kamerans synfält (synligt i bildrutan under de senaste 10 sekunderna); när inget mål syns används istället kamerans vanliga automatiska exponering, med sitt eget skydd mot överexponering, så att resten av scenen inte hålls mörkare än nödvändigt. Medan takvärdet är aktivt är kameran**fäst vid infallande ljus**: den körs i öppen slinga med den incidentmätna exponeringen och förstärkningen inställd på 0 dB — exponeringen följer det uppmätta ljuset, inte scenens innehåll. Eftersom takvärdet endast kan förkorta exponeringstiden kan det inte i sig orsaka klippning. Sedan version 1.3.0 beräknas takvärdet med kamerans egen exponeringskonstant från dess kalibreringspaket (ett flottgenomsnittsvärde endast när paketet inte innehåller något). Takvärdet kopplas ur automatiskt – och normal automatisk exponering återupptas – när inget kalibreringsmål syns, DAQ-avläsningen saknas, är föråldrad (>30 s), mörk eller om ≥15 % av bilden klipps vid den fastställda exponeringen (vilket innebär att sensorn och kameran ser olika belysning). Det finns ingen GUI-knapp; detta är standardbeteende när en RGB-kamera är kopplad till en DAQ.

### Insamling och utlösning

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Arraymedlemmar visar dessutom de skrivskyddade raderna **Roll**(Master i blått / Slave i grönt),**Synkroniseringslinje**och**Jämbördiga enheter**.

| Kontroll | Alternativ | Standard | Anmärkningar |
| --- | --- | --- | --- |
| **Triggerläge** | Av / På | På | Inaktiverat för arraymedlemmar (arrayen hanterar triggningen). |
| **Triggerkälla** | Programvara / Linje 0 (M8) / Linje 1 / Linje 2 | Linje 0 | Dold när Triggerläge är Av; inaktiverad för arraymedlemmar. Linje 0 är den optoisolerade externa triggeringången på M8. |
| **Bildfrekvens**| Auto / Manuell + värde | Auto |**Auto**: kamerans bildhastighetsbegränsning är avstängd — exponeringen avgör fps, och rutan visar den aktuella bildhastigheten i realtid.**Manuell**: du begränsar fps med ett skjutreglage (1 upp till det bandbreddsbegränsade maximivärdet), utgående från den aktuella faktiska frekvensen. Array-medlemmar ser ett skrivskyddat ”N fps (live)” med ”Ställ in i array-inställningarna”. |

### Nätverk / Transport

| Rad | Beteende |
| --- | --- |
| **Paketstorlek**| 1500 (Standard) / 9000 (Jumbo) — standard**Jumbo**. |
| **Genomströmning** | Skrivskyddad gräns för länkgenomströmning i MB/s. Backend omfördelar detta mellan alla anslutna kameror vid varje anslutning/frånkoppling. |
| **Bufferhantering** | Skrivskyddat läge för bufferhantering. |

### Inspelning

Fönstret avslutas med en knapp **”Öppna inspelningsinställningar…”** som tar dig till [fönstret Inspelningsinställningar](capture.md#the-capture-settings-pane) (inaktiverat tills ett projekt är öppet — ”Skapa eller öppna ett projekt för att spara inspelningar”). Om kameran är dold eller pausad påminner ett tips dig om att visa/återuppta innan du börjar spela in.

## Fönstret Array-inställningar

Öppnas med **kugghjulet**i en ARRAY-rad. Rubrik: arraynamn med en penna för att byta namn och**×** för att stänga. Avsnitten nedan som är markerade med *endast kombinerat* visas endast för arrayer som är anslutna i kombinerat visningsläge.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Synkronisering

Skrivskyddade rader för **Master**,**Slaves**och**Synkroniseringslinje**.

### Omgivningsljussensor

Visas för både kombinerade och separata arrayer:

* Rullgardinsmenyn **Ljussensor** — kopplar en DAQ till hela arrayen. Valet träder i kraft omedelbart, sprids till varje enskild kameras egen rullgardinsmeny för ljussensor (du kan fortfarande åsidosätta inställningen per kamera) och börjar vidarebefordra spektra till matrisen.
* **Status**-rad i realtid: Av · ”Väntar på första spektrumet…” · ”Aktiv — alla kameror i matrisen är belysningskorrigerade” · ”Inget nytt spektrum under de senaste 3 sekunderna — använder fortfarande den senaste avläsningen (ingen timeout för inaktuella data)…”.
* Anmärkning i fönstret: ”Radiometrisk korrigering för hela matrisen. Inställningar per kamera åsidosätter detta.”

### Kalibreringsmål

En egen grupp direkt efter omgivningsljussensorn, för både kombinerade och separata matriser:

* Kryssrutan **Aktivera** — ”Upptäck MAPIR ArUco-mål och validera NDVI mot panelens reflektans-LUT”; styr överlägg och valideringstabeller för det kombinerade rutnätets mål. På ett **separat** rutnät aktiverar den varje medlems egen [kalibreringsmåltabell](#the-calibration-target-table) (en per kamerarad i listvyn).

### Inspelning — enhetliga sensorinställningar

Visas för både kombinerade och separata matriser (grupprubriken lyder ”Inspelning (enhetligt för alla kameror)”). Dessa inställningar gäller enhetligt för varje medlem (ändringar per medlem skulle störa synkroniseringen). Ändringar lagras tillfälligt och tillämpas tillsammans med **Apply**— förutom växlingsknappen**ROI on / ROI off**, som tillämpas omedelbart.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Kontroll | Alternativ / intervall | Vad den gör |
| --- | --- | --- |
| **Pixelformat** | 8-bit / 10-bit / 12-bit / 16-bit | Enhetligt bitdjup för alla medlemmar; varje kamera använder formatet för det bitdjupet för sin egen sensor. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hårdvarubinning — behåller hela synfältet samtidigt som signal-brusförhållandet och bildfrekvensen förbättras. Om du ändrar detta återställs ROI-fälten till det nya fullständiga synfältet. |
| **Upplösning** (förinställning) | Full / Halv / Kvart | Relativt binning; fyller ROI-fälten med en centrerad beskärning. |
| **ROI-beskärning (px)**|**ROI på / ROI av**-väljare,**Välj**-knapp, fälten B / H / X / Y, karta över hela sensorn | Sensorbeskärning — se [ROI-redigeraren](#the-roi-editor). Bredd/höjd snäpper fast vid multiplar av 16 (minst 64); förskjutningar snäpper till multiplar av 4. Ett ”max WxH”-tips visar det högsta värdet och**Återställ** återställer fälten till hela synfältet. Under redigering följer den orangea beskärningsrutan varje fältändring på kartan och på matrisrutan. |
| **Triggerfrekvens**| Välj mellan Auto / Manuell + fps 0,5–10, steg 0,5 |**Auto**(standard): backend beräknar utlösningsfrekvensen utifrån upplösning och bandbredd — inmatningsfältet är inaktiverat och visar det beräknade värdet.**Manuell**: låser ditt värde när du klickar på Tillämpa. En låst frekvens som en ny bildstorlek inte klarar av hålls kvar på den snabbaste frekvensen som storleken tillåter, och återställs när bildstorleken tillåter det igen. |

Observera i fönstret: ”Ändringar av format/upplösning startar om alla kameror kortvarigt. Triggfrekvensen gäller direkt.” Knapparna **Apply / Cancel** finns längst ned i fönstret.

#### ROI-redigeraren

Nytt i 1.3.0. Raden för ROI-beskärning och kartan under den fungerar tillsammans:

* **ROI på / ROI av**— aktiverar eller inaktiverar sensorbeskärningen**omedelbart**(kamerorna startas om kortvarigt; matrisrutan visar ”Tillämpar inställningar…”).**Av**läser av hela sensorn och**kommer ihåg din beskärning**— fälten fortsätter att visa den;**På** tillämpar den igen. Om du aktiverar ROI utan att någon beskärning är inställd tillämpas en centrerad beskärning i halv storlek ("Ingen beskärning var inställd, så en centrerad beskärning i halv storlek tillämpades. Justera den med fälten.").
* **Välj**— ritar beskärningen på kartan: dra över den eller klicka på två motsatta hörn (Esc avbryter). Rektangeln placeras i fälten B/H/X/Y, justerad enligt stegen ovan; ingenting tillämpas förrän du trycker på**Apply**, så du kan finjustera den med fältpilarna först. En rektangel per tryck på Select.
* **Kartan**visar beskärningen på hela sensorn, så du kan bredda ut den igen: ”Full sensor, live” medan kamerorna läser av hela sensorn, eller ”Fullbildsöversikt från {tid}. Kamerorna läser endast den beskärda delen.” medan de är beskärda. Om matrisen endast någonsin har strömmat beskuret visar kartan ”Ingen fullbildsvy ännu: denna matris har endast strömmat beskuret.” med en**Visa fullbild**-knapp, som stänger av ROI. Beskärningens etikett `W×H @ X,Y` placeras utanför rutan (ovanför den, eller under när rutan når upp till kanten), på kartan och på live-rutan.
* **Apply**bekräftar inställningarna: en beskärning aktiverar ROI; helbilden (efter**Reset**) inaktiverar den med den tidigare beskärningen fortfarande lagrad; en ROI som är inaktiverad och vars fält fortfarande visar den lagrade beskärningen förblir inaktiverad när du tillämpar en annan ändring (format, binning, hastighet).
* **På en inriktad matris placeras varje medlems beskärningsfönster enligt inriktningen**, så att varje kamera ramar in samma del av scenen. En matris som beskärts innan den inriktats inriktas först på hela sensorn (beskärningen tas bort, inriktningen löses och beskärningen återställs med fönster per kamera).
* Efter en beskärningsändring anpassas den automatiska exponeringen på nytt efter den nya inramningen. Under en beskärning kan en medlems exponering fortfarande pågå fram till utlösningsperioden.
* ROI:ns på/av-status och den lagrade beskärningen sparas tillsammans med projektet.

### Justering (samregistrering)

Visas för **både kombinerade och separata matriser** (1.2.0 visade det endast för kombinerade matriser); grupprubriken lyder ”Justering”.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Residual**-rutan: ”RMS x.xx px” — grönt under 1 px, gult under 3 px, rött i övriga fall eller om någon kamera misslyckades; ”ingen profil” före den första lösningen.
* Sammanfattningsrad: ”✓ Alla kameror inriktade (N)” / ”⚠ p/N kameror inriktade —  <serial (filter)="">misslyckades” / ”Beskärning aktiv — Kalibrera om för att inrikta (använder hela sensorn)” / ”Väntar på att exponeringen ska stabiliseras…” / ”Tillämpar inställningar…” (kamerorna startas om) / ”Arrayen har slutat leverera bildrutor — anslut arrayen igen”.
* Tabell per kamera: kamera (de fyra sista siffrorna i serienumret + filter), omprojiceringsfel i px med antal matchningar ("ref" för huvudkameran) och poängen för normaliserad korskorrelation av överlappningen mot gränsvärdet 0,35 för godkänd passning.
* **Omkalibrera justeringen** knapp (visar ”Kalibrera inriktning” före den första profilen) — kör om samregistreringen på nya bildrutor.
* **Kryssrutan ”Auto-expose cameras for alignment”** (markerad som standard) — gör tillfälligt mörka eller platta kameror ljusare (först exponering, sedan förstärkning) så att de får textur att matcha, och återställer sedan AE.

Chloros justerar varje matris automatiskt efter anslutning, i båda visningslägena, så snart medlemmarnas automatiska exponering har stabiliserats (programmet fortsätter att försöka i upp till 5 minuter); den kombinerade förhandsvisningen justeras också vid öppning. En separat matris har ingen sammansatt ruta, så detta avsnitt avläser justeringsstatusen och kör ”Recalibrate” själv. Kalibrera om fokus eller scenens djup har ändrats. Justeringen är **avsedd endast för den aktuella sessionen** — den sparas aldrig i en profil, eftersom den beror på scenavståndet just då. Bildserier kan fortfarande exporteras med pixelregistrering (se [Justerade exporter](capture.md#per-array-controls)).

### Smart vignett

* Kryssrutan **Aktivera korrigering**— tillämpar den kameraspecifika vignettberäkningen på den radiometriska kedjan (i realtid**och** vid export).
* **Kalibrera från aktuell vy**— rikta först matrisen mot ett enhetligt mål (platt skärm, vägg eller himmel); varje kamera jämnas ut individuellt och statusen rapporterar ”n/N kameror · −x,x %” utjämningsvinst.**Rensa** tar bort uppskattningen.
* Status: ”Kalibrerad”, ”Ej kalibrerad” eller ”Delvis kalibrerad — kör om” när endast vissa kamerors uppskattningar fortfarande gäller vid deras aktuella bildutsnitt (till exempel efter att en omkalibrering har flyttat en kameras beskärningsfönster) — kör Kalibrera igen.
* Finjustera per kamera med skjutreglaget **Vignett** för varje kamera i [Liveförhandsvisning](#live-preview).

### Liveförhandsgranskning

* Kryssrutan **Histogram**+ breddslider (0,10–0,90 av bilden) — histogramremsan i liveflödet visas på varje medlems ruta samtidigt (inklusive rutor med delade kanaler), samt [Histogramöverlägg](#display-overlays-drawn-over-the-live-feed) som är inställt för hela matrisen. Det visas som aktiverat när alla medlemmar redan har det. På en**separat** matris är detta den enda raden i gruppen.
* *Endast kombinerat:* **Index**: kryssruta för aktivering + kugghjul — öppnar den delade [Indexkalkylatorn](#index-calculator-pane) med band ritade från**alla** medlemskameror. En förhandsgranskningsrad för uttryck nedanför visar det aktuella uttrycket ("Inget uttryck angivet — öppna kalkylatorn för att skapa ett"), som uppdateras varje sekund.
* *Endast kombinerat:* Rullgardinsmenyn **Renderingsupplösning**(samma förinställningar som per kamera, standard 720p): höjden på live-view-strömmen**och** den sparade storleken på den sammansatta exporten. Observera i fönstret: ”Förhandsgranskning + storlek på den sparade sammansatta bilden. Bilder per kamera exporteras alltid i full upplösning.”

### Visningslager *(endast kombinerat)** Kryssrutan **Aktivera** (standardinställning avstängd — huvudkameran visas direkt; på = lagerkomposit).
* **Rullgardinsmenyerna**Förgrund**/**Bakgrund**: varje medlemskamera (efter namn) eller**Index**. När Förgrund är Index visas bakgrundslagret för pixlar utanför LUT:s min-/maxvärden.

### Delad vy *(endast kombinerad)*

**”Visa medlemskameror”**— en knapp för**Dela / Dölj medlemskameror** som lägger till varje medlems egen livebild som separata rutnätbrickor bredvid den sammansatta bilden. Brickorna läser av arrayens befintliga bildbuffert (ingen extra kameraanslutning). Endast rutnätsvy; sparas per array tillsammans med projektet.

### Funktioner

En skrivskyddad panel som uppdateras var 5:e sekund:

* **Nivåbeteckning**: ”Samtidig inspelning” (grön) · ”Samtidig inspelning (FTD-förskjuten utsändning)” (grön) · ”Förskjuten inspelning (100 ms avvikelse)” (gul) · ”Konfigurationen är för stor” (röd).
* **Ramstatus**: ”x,xx % ofullständig” — grön under 1 %, gul under 5 %, röd vid 5 % eller mer.
* **Länklinje**: ”NIC {mbps} Mbps – kontinuerlig {MB/s} MB/s”.

Detta är arrayens aktuella bandbreddsbudget. För information om underliggande fps och nätverksmodell — samt vad som ska ändras när nivån blir gul eller röd — se [Arrayer med flera kameror](arrays.md) och [CLI-referensen](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Fönstret Index Calculator

Den tredje sidan i sidofältet, som delas av Index-inställningarna per kamera och de kombinerade array-inställningarna (en i taget – rubriken lyder ”Index Calculator — <camera name="">” eller ”Index Calculator — <array name="">”). Den tar emot bandlistan (kamerans naturliga filterbandnaturliga band, eller alla band över arrayens medlemmar), det aktuella uttrycket och LUT-konfigurationen (på/av, nivå – standard 7-steg, min – standard 0,2, max – standard 1), samt ett live-indexhistogram. **Apply** bekräftar uttrycket; LUT-ändringar tillämpas direkt i förhandsvisningen.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Inställningar per kamera kontra arrayhanterade inställningar

Snabböversikt över vad som finns var när en kamera ingår i en array:

| Arrayhanterat (skrivskyddat i kamerapanelen) | Fortfarande per kamera inom en array |
| --- | --- |
| Pixelformat, upplösning, binning | Autoexponering (exponering, förstärkning, mål, utjämning, ROI) |
| Utlösningsläge/källa, bildfrekvens | Brusreducering, skärpa, vignettering |
| | Orientering (spegling/rotation), överlagringar (matrisinställningar → Live Preview ställer in histogrammet på alla medlemmar samtidigt), punktmätning |
| | Index (matriser med separat visning), koppling till ljussensor |

Övriga övergripande funktioner:

* **Kombinerad vs separat visning** väljs vid anslutning till matrisen: kombinerad = en justerad sammansatt ruta (medlemmarna sänder endast via delad vy); separat = varje medlem renderar sin egen synkroniserade ruta. En kamera visar aldrig både en fristående sändning och en matrisruta.
* **Automatisk återanslutning**: när ett sparat projekt öppnas återställs dess kameror och matriser och alla sparade inställningar tillämpas på nytt på backend innan strömmarna återupptas.
* **Capture-gating**: dolda eller pausade kameror exkluderas från Capture All; en matris blockeras helt endast när ALLA medlemmar är dolda/pausade. Se [Inspelningsinställningar och lägen](capture.md).

## Hur inställningarna sparas

Kameraflikens tillstånd sparas **tillsammans med projektet**, inte i webbläsaren:

* Varje reaktiv förändring tar en ögonblicksbild av kamerorna och matriserna i projektets `cameras.json` (debounced 500 ms). Detta omfattar kameranamn och färger, inställningar för exponering/förstärkning/AE, pixelformat/upplösning/binning, utlösningsfrekvens, förhandsgranskningsinställningar (renderingsupplösning, brusreducering, skärpa, vinjettering, färgprofil, högdagrar, mättnad/kontrast), orientering, överlägg, kanaluppdelningar, indexkonfiguration, inställningar för prediktiv AE, AE-ROI, matrisnamn, visningsläge, inställningar för matrisinspelning (inklusive ROI-beskärning, om ROI är aktiverad och den sparade beskärningen när den är avstängd), matrisens Wire Budget samt rutnätsblocket (feedzoom, visningsläge, rutnätslås, manuell kakelordning, dolda kameror, stängda kakelrutor, aktiv kamera).
* Ljussensorbindningar sparas i projektets `sensors.json`.
* När projektet öppnas på nytt återansluts hårdvaran och alla inställningar tillämpas på nytt.
* **Inget projekt öppet = endast sessionsbaserat**: utan projekt bevaras ingenting när Chloros stängs.
* Endast sessionsbaserat oavsett projekt: pausläge, spotmätningsprover, kryssrutan för kalibreringsmål per kamera (öppnas alltid avstängd) och matrisjusteringsprofilen (beräknas om per session enligt design).
* Ett undantag: valen för export av **Inspelningsinställningar** och inspelningsläget sparas per projekt i appens lokala lagringsutrymme istället för i `cameras.json` — se [Inspelningsinställningar och lägen](capture.md).</array></camera></serial>
