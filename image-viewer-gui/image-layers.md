# Bildlager

Med **rullgardinsmenyn för lager** längst upp till höger i bildvisaren kan du växla mellan alla versioner av den bild du tittar på – från källbilden via varje bearbetad produkt till de beräknade indexbilderna – utan att behöva lämna visaren.

## Vad är bildlager?

Ett ”lager” i Chloros är en **produktfil**som är kopplad till en källbild. Importen ger dig källfilerna; bearbetningen lägger till ett lager för varje produkt som körningen genererade. Exporterade filer behåller källfilens namn – det är**mappen** som identifierar produkten, och lager namnet är Chloros:s etikett för den mappen.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Lagerlistan

### Alltid närvarande

| Lager | Vad det är |
| --- | --- |
| **JPG**(eller**PNG**/**TIFF**) | Basfilen som kom in med inspelningen. Survey3 importerar en `.JPG` bredvid varje `.RAW`; LATTICE-inspelningar medför en förhandsgranskning av typen PNG eller TIFF. Märkt enligt vad som faktiskt importerades |
| **RAW (Original)** | Den ursprungliga råbilden, avbayered för visning utan tillämpade korrigeringar. Tillgänglig från importögonblicket — kräver ingen bearbetning |

En LATTICE-inspelning vars basfil **är** dess råbild har ingen separat baspost: `RAW (Original)` täcker redan den.

### Survey3-bearbetningsprodukter

| Lager | Skrivs till | Finns när |
| --- | --- | --- |
| **RAW (Mål)** | — | Bilden identifierades som innehållande ett kalibreringsmål |
| **RAW (reflektans)** | `Reflectance_Calibrated_Images/` | Reflektanskalibreringen genomfördes framgångsrikt på denna bildruta |
| **Vignettkorrigerad**| `Vignette_Corrected_Images/` | Bilden kunde inte reflektanskalibreras**och** *vignettkorrigering* var aktiverad |
| **Sensorsvar**| `Sensor_Response_Images/` | Bilden kunde inte kalibreras för reflektans**och** *vignettkorrigering* var avstängd |
| **Vitbalanserad** | `White_Balanced_Images/` | En vitbalanserad produkt skrevs ut |

{% hint style="info" %}
**Vignettkorrigering och sensorns respons är alternativ, aldrig båda samtidigt.** Det finns exakt en okalibrerad reservprodukt per körning för varje kameramodell, och *Vignettkorrigering*-knappen väljer vilken. Se [Projektinställningar](../project-settings/project-settings.md).
{% endhint %}

### LATTICE-nivåer

LATTICE sammanställer dessa i ett enda bearbetningssteg. Vilka som finns beror på exportinställningarna per produkt i Projektinställningar och på vad som gäller för kameran.

| Lager | Skrivs till | Gäller för |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB och multispektralt |
| **RAW (förhandsgranskning)** | `Preview_Images/` | Multispektrala (falskfärgssträckning) |
| **Vitbalanserad** | `Preview_Images/` | RGB-huvudkameror — förhandsvisningen RGB är registrerad under detta namn så att den stämmer överens med lagret Survey3 med samma namn |
| **RAW (strålning)** | `Radiance_Images/` | Endast multispektralt |
| **RAW (reflektans)** | `Reflectance_Calibrated_Images/` | Endast multispektralt, och endast när en matchande `.daq`-nedåtriktad post eller ett QA-godkänt mål inom bilden täcker bilden |

RGB-huvudkameror har ingen radiometri per band, så strålning och reflektans hoppas över för dem som **ej tillämpligt** — loggen anger detta istället för att tyst misslyckas.

### Index-, LUT- och sandbox-lager

| Lagermönster | Exempel | Varifrån det kommer |
| --- | --- | --- |
| **RAW (`<INDEX>`-index)** | `RAW (NDVI Index)` | Ett per index konfigurerat i projektinställningarna, beräknat under bearbetningen |
| **`<INDEX>` LUT** | `NDVI LUT` | Den färgkartlagda versionen av ett index |
| **Sandlåda (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | En per exportkörning av [Index/LUT Sandbox](index-lut-sandbox.md) |

Om samma indexnamn konfigureras mer än en gång med olika inställningar får den andra och efterföljande versionerna ett nummer i namnet (`RAW (NDVI2 Index)`) så att lagren förblir urskiljbara.

***

## Använda lagerväljaren

1. Öppna en bild i helskärmsläge genom att klicka på en miniatyr i rutnätet
2. Klicka på **lagermenyn** längst upp till höger i visningsfönstret
3. Välj ett lager – bilden uppdateras omedelbart

I rullgardinsmenyn visas **JPG, RAW (Original), RAW (Target), RAW (Reflectance)** först, i den ordningen, och allt annat listas efter dem i den ordning produkterna registrerades.

### Lagerinställning när du navigerar

Genom att trycka på **←**/**→** går du till nästa bild och systemet försöker hålla dig kvar på samma lager:

1. **Exakt matchning först** – om nästa bild har ett lager med samma namn, får du det. Det är detta som gör att du stannar kvar på `RAW (NDVI Index)` när du bläddrar igenom en hel uppsättning
2. **Därefter en matchning efter typ** — ett indexlager söker efter vilket indexlager som helst, en LUT efter vilken LUT som helst, reflektans efter reflektans, mål efter mål, original efter original, bas efter bas
3. **Därefter, endast för exportlager** — namnet behålls även om lagerlistan inte hunnit ikapp ännu, eftersom filen redan finns på disken. Det är detta som gör att du kan granska produkter medan en körning fortfarande skriver ut dem
4. **I övriga fall** — det första tillgängliga lagret, vilket normalt är basbilden

Sidokarfilerna `.daq` och `.csv` i projektet hoppas över vid navigering med piltangenterna, så att man vid bläddring genom bilderna aldrig hamnar på en ljussensorinspelning.

Zoom och panorering överförs även mellan bilderna, vilket gör det enkelt att jämföra samma fältposition före och efter.

***

## Förstå pixelvärden per lager

Panelen [Cursor Values](opening-an-image-full-screen.md#cursor-values) visar det faktiska värdet per kanal under markören, i den enhet som lagret är lagrat i. Kolumnerna ändras beroende på lagret:

| Lager | Visad enhet | Anmärkningar |
| --- | --- | --- |
| Base (JPG / PNG / TIFF förhandsgranskning) | DN, 0–255 | Visningsvärden, gammakorrigerade i RGB. Endast för visuell granskning |
| RAW (Original) | DN | Råa digitala värden från sensorn. Histogramaxeln anger djupet: 255 (8-bit), 4095 (12-bit) eller 65535 (16-bit) |
| RAW (Debayered) | DN | Linjär, ingen bildsträckning |
| RAW (Förhandsgranskning) / Vitbalanserad | DN | Visningsresultat — sträckt eller gammakorrigerat. Inte avsett för mätning |
| RAW (Strålning) | **W/m²/sr/nm** | Float32 fysisk strålning. Ingen DN-kolumn |
| RAW (reflektans) | DN **och %** | Procent beräknat med filens egen skala — se nedan |
| Index / LUT / sandbox-exporter | Indexvärde, eller RGB-komponenter | En enkelkanalig indexfil rapporterar indexvärdet; en färgkartlagd LUT-fil rapporterar komponenterna Red/Green/Blue |

### Reflektans: skalan är per fil

{% hint style="warning" %}
**”Dela med 65 535” är endast korrekt för Survey3.** LATTICE-reflektansen lagras i en annan skala, och att blanda ihop de två divisorerna är det vanligaste sättet att få reflektansvärden som är exakt hälften av vad de borde vara.
{% endhint %}

| Källa | DN som motsvarar reflektans 1,0 | Identifieras av |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | XMP-taggen `Chloros:PixelScale=32768` som finns med i varje LATTICE-reflektans-export. Tack vare 2× headroom kan ρ över 1,0 återges istället för att klippas bort |
| **Survey3**|**65535** | Ingen Chloros XMP-skalatagg – kalibreringen Survey3 skriver ut ρ × dtype-max och beskär vid 1,0 |

För GIS och skript: läs in `Chloros:PixelScale` från filen och dividera med det. Om taggen saknas är filen i Survey3-skala (65535). Visningsprogrammet, index-/LUT-sandlådan och indexexporten beräknar alla skalan på samma sätt, så det tal du läser vid markören är det tal som indexberäkningen använde.

Formatspecifik lagring utöver den skalan:

* **TIFF (32-bitars, procent)** lagrar DN / 65535 som ett flyttal
* **PNG (8-bitars)**och**JPG (8-bitars)** lagrar DN × 255 / 65535
* En **8-bitars TIFF-export av en 8-bitars källinspelning** beskärs till 0–255 istället för att omskalas, och har medvetet ingen skalningsetikett. Panelen visar endast DN för dessa filer, utan procentkolumn

### Indexvärdesintervall

| Indexfamilj | Typiskt intervall | Avläsning |
| --- | --- | --- |
| Normaliserad skillnad (NDVI, GNDVI, NDRE, ENDVI…) | −1 till +1 | Frisk vegetation vanligtvis 0,4–0,9; bar mark nära 0; vatten negativt |
| Jordjusterad (SAVI, OSAVI, MSAVI2…) | ungefär −1 till +1,5 | Värde liknande NDVI med jordbakgrunden undertryckt |
| Förhållande (GRVI, GCI, MSR, CIRE…) | obegränsat uppåt | Förhållandena ökar obegränsat när nämnarens band går mot noll |
| EVI / LAI | 0 till ~1, 0 till ~3,5 | Moln och andra mättade pixlar pressar båda utanför intervallområdet – maskera dem först |

Se [Formler för multispektrala index](../project-settings/multispectral-index-formulas.md) för den exakta formeln bakom varje förinställning.

***

## Vanliga arbetsflöden

### Jämförelse före/efter

1. Välj **RAW (Original)** och notera vinjetteringen och de okalibrerade värdena
2. Byt till **RAW (Reflectance)**

3. Jämför — vinjetteringen är borttagen, värdena kalibrerade. Zoom och panorering förblir oförändrade, så att du tittar på samma markyta

### Granska ett index över en hel bildserie

1. Öppna den första bearbetade bilden och välj indexlagret
2. Tryck på **→** upprepade gånger – indexlagret följer med dig från bild till bild
3. Håll koll på histogrammet i sidofältet medan du går igenom bilderna: en bildruta där fördelningen hoppar är värd att titta närmare på

### Verifiera kalibreringsmål

1. Välj **RAW (Target)** på en målbildruta
2. Kontrollera att målet är tydligt synligt och detekterat
3. Gå vidare till nästa målbildruta — mållagret följer med

### Kontrollera reflektansvärdenas noggrannhet

1. Välj **RAW (Reflectance)**

2. Läs av kolumnen**%** i panelen Cursor Values — den är redan korrekt skalad för den filen
3. Kontrollera mot kända material i bilden: frisk vegetation har höga värden i NIR och låga värden i rött; ett kalibreringsmål bör visa värden som ligger nära dess publicerade reflektans

***

## Felsökning

### Ett lager som jag förväntade mig finns inte i rullgardinsmenyn

**Möjliga orsaker**

* Bilden har aldrig bearbetats — endast baslagret och `RAW (Original)` finns
* Produktens exportknapp är avmarkerad i projektinställningarna
* Produkten gäller inte för den kameran (radian och reflektans på en RGB-master; valfritt index på en enkelbands M3M-monokamera)
* Reflektanskalibreringen hade inget att utgå ifrån – ingen `.daq`-täckning i nedåtriktad riktning och inget QA-godkänt mål inom bilden – så bilden återgick till ”Vignette Corrected” eller ”Sensor Response”

**Åtgärder**

1. Kontrollera körloggen: Chloros anger när en begärd exportprodukt inte kunde genereras och varför
2. Kontrollera exportinställningarna per produkt i [Projektinställningar](../project-settings/project-settings.md)
3. Kontrollera att produktmappen finns i projektets utdataträd
4. Kör om med produkten aktiverad

### Lagerlistan ser ut att vara inaktuell

Chloros skannar om projektets produktmappar medan en körning pågår och åtgärdar saknade lagerregistreringar utifrån vad som faktiskt finns på disken, så ett lager som har exporterats normalt visas av sig själv vid en avläsning. Att växla bort från bilden och tillbaka tvingar fram en ny upplösning.

### Reflektansvärdena verkar vara hälften av vad de borde vara

Du delar med största sannolikhet en LATTICE-fil med 65535. Använd `Chloros:PixelScale` (32768), eller läs kolumnen **%**, där detta redan har tillämpats.

### Indexlagret finns men bilden är tom

Indexet kräver band som ditt lager inte har – till exempel ett index som läser en tredje kanal som tillämpas på en en- eller tvåkanalsfil. Byt till ett flerbandslager (reflektans eller debayerat), eller välj ett index som passar kamerans filter.

***

## Nästa steg

* [**Öppna en bild i helskärmsläge**](opening-an-image-full-screen.md) — marköravläsning, histogram och GSD-kontroll
* [**Index/LUT-sandlåda**](index-lut-sandbox.md) — interaktiv indexvisualisering och export
* [**Multispektrala indexformler**](../project-settings/multispectral-index-formulas.md) — indexreferensen
* [**Avsluta bearbetningen**](../processing-images-gui/finishing-the-processing.md) — utdatamappstrukturen som dessa lager pekar mot
