# Bildlager

Med **lagermenyn** längst upp till höger i bildvisaren kan du växla mellan alla versioner av den bild du tittar på – från källbilden via varje bearbetad produkt till de beräknade indexbilderna – utan att behöva lämna visaren.

## Vad är bildlager?

Ett ”lager” i Chloros är en **produktfil**som är kopplad till en källbild. Importen ger dig källfilerna; bearbetningen lägger till ett lager för varje produkt som körningen genererade. Exporterade filer behåller källfilens namn – det är**mappen** som identifierar produkten, och lagernamnet är Chloros etikett för den mappen.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Lagerlistan

### Alltid närvarande

| Lager | Vad det är |
| --- | --- |
| **JPG**(eller**PNG**/**TIFF**) | Basfilen som kom in med inspelningen. Survey3-importen skapar en `.JPG` bredvid varje `.RAW`; LATTICE-inspelningar ger en PNG- eller TIFF-förhandsvisning. Märkt efter vad som faktiskt importerades |
| **RAW (Original)** | Den ursprungliga råbilden, avbayered för visning utan tillämpade korrigeringar. Tillgänglig från importögonblicket — den behöver ingen bearbetning |

En LATTICE-inspelning vars basfil **är** dess råbild har ingen separat baspost: `RAW (Original)` täcker redan den.

### Survey3-bearbetningsprodukter

| Lager | Skrivs till | Finns när |
| --- | --- | --- |
| **RAW (Mål)** | — | Bilden identifierades som innehållande ett kalibreringsmål |
| **RAW (Reflektans)** | `Reflectance_Calibrated_Images/` | Reflektanskalibrering genomfördes framgångsrikt på denna bild |
| **Vignettkorrigerad**| `Vignette_Corrected_Images/` | Bilden kunde inte reflektanskallibreras**och** *vignettkorrigering* var aktiverad |
| **Sensorsvar**| `Sensor_Response_Images/` | Bilden kunde inte kalibreras med reflektans**och** *vignettkorrigering* var avstängd |
| **Vitbalanserad** | `White_Balanced_Images/` | En vitbalanserad produkt skrevs ut |

{% hint style="info" %}
**Vignettkorrigering och sensorns respons är alternativ, aldrig båda samtidigt.** Det finns exakt en okalibrerad reservprodukt per körning, för varje kameramodell, och *Vignettkorrigering*-knappen väljer vilken. Se [Projektinställningar](../project-settings/project-settings.md).
{% endhint %}

### LATTICE-nivåer

LATTICE sammanställer dessa i ett enda bearbetningssteg. Vilka som finns beror på exportinställningarna per produkt i Projektinställningar och på vad som gäller för kameran.

| Lager | Skrivs till | Gäller för |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB och multispektralt |
| **RAW (förhandsgranskning)** | `Preview_Images/` | Multispektralt (falskfärgssträckning) |
| **Vitbalanserad** | `Preview_Images/` | RGB-huvudkameror — RGB-förhandsvisningen registreras under detta namn så att den stämmer överens med Survey3-lagret med samma namn |
| **RAW (strålning)** | `Radiance_Images/` | Endast multispektralt |
| **RAW (reflektans)** | `Reflectance_Calibrated_Images/` | Endast multispektralt, och endast när en matchande `.daq`-nedåtriktad post eller ett QA-godkänt mål inom bildrutan täcker bildrutan |

RGB-huvudkameror har ingen radiometri per band, så strålning och reflektans hoppas över för dem som **ej tillämpligt** — loggen anger detta istället för att tyst misslyckas.

{% hint style="info" %}
**Rå- och debayered-lager stäms av mot produkterna (nytt i 1.3.0).**LATTICE-råfiler och debayered-filer sparas i sensorns ursprungliga orientering och registrerar i XMP den orientering som hör till dem; de andra produkterna sparas med den orienteringen redan tillämpad. Visningsprogrammet och bildrutnätet växlar mellan**RAW (Original)**och**RAW (Debayered)** enligt den informationen för visning, så att bilden inte längre vänds när du växlar mellan ett produktlager och ett råformatlager. Detta gäller endast visningen — filerna skrivs aldrig om. Filer utan orienteringsinformation (tagna före version 1.3.0 eller skapade av annan programvara) visas som de är lagrade. Se [Utgående bildformat](../output-image-formats.md#orientation).
{% endhint %}

Klämmasker (`*_clampmask.tif` i en `clamp_masks/`-mapp) är inte lager: Chloros hoppar över dem vid import. Se [Utgångsbildformat](../output-image-formats.md#clamp-masks).

### Index-, LUT- och sandbox-lager

| Lagermönster | Exempel | Varifrån det kommer |
| --- | --- | --- |
| **RAW (`<INDEX>`-index)** | `RAW (NDVI Index)` | Ett per index som konfigurerats i projektinställningarna, beräknat under bearbetningen |
| **`<INDEX>` LUT** | `NDVI LUT` | Den färgkartlagda versionen av ett index |
| **Sandlåda (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | En per [Index/LUT Sandbox](index-lut-sandbox.md) exportkörning |

Om samma indexnamn konfigureras mer än en gång med olika inställningar får det andra och efterföljande namnen ett nummer i namnet (`RAW (NDVI2 Index)`) så att lagren förblir urskiljbara.

***

## Använda lagerväljaren

1. Öppna en bild i helskärmsläge genom att klicka på en miniatyr i rutnätet
2. Klicka på **lagermenyn** längst upp till höger i visningsfönstret
3. Välj ett lager – bilden uppdateras omedelbart

I rullgardinsmenyn visas **JPG, RAW (Original), RAW (Target), RAW (Reflectance)** först, i den ordningen, och allt annat listas efter dem i den ordning produkterna registrerades.

### Lagerinställning när du navigerar

Genom att trycka på **←**/**→** går du till nästa bild och systemet försöker hålla dig kvar på samma lager:

1. **Exakt matchning först** – om nästa bild har ett lager med samma namn får du det. Det är detta som gör att du stannar kvar på `RAW (NDVI Index)` när du går igenom en hel uppsättning
2. **Därefter matchning efter typ** — ett indexlager söker efter vilket indexlager som helst, en LUT efter vilken LUT som helst, reflektans efter reflektans, mål efter mål, original efter original, bas efter bas
3. **Därefter, endast för exportlager** — namnet behålls även om lagerlistan ännu inte hunnit ikapp, eftersom filen redan finns på disken. Det är detta som gör att du kan granska produkter medan en körning fortfarande skriver ut dem
4. **I övriga fall** — det första tillgängliga lagret, vilket normalt är basbilden

Sidokarfilerna `.daq` och `.csv` i projektet hoppas över vid navigering med piltangenterna, så att man vid bläddring genom bilderna aldrig hamnar på en ljussensorinspelning.

Zoom och panorering överförs även mellan bilderna, vilket gör det enkelt att jämföra före- och efterbilder av samma fältposition.

***

## Förstå pixelvärden per lager

Panelen [Cursor Values](opening-an-image-full-screen.md#cursor-values) visar det faktiska värdet per kanal under markören, i den enhet som lagret är lagrat i. Kolumnerna ändras beroende på lagret:

| Lager | Visad enhet | Anmärkningar |
| --- | --- | --- |
| Bas (JPG/PNG/TIFF-förhandsgranskning) | DN, 0–255 | Visningsvärden, gammakorrigerade för RGB. Endast för visuell granskning |
| RAW (Original) | DN | Råa digitala sensordata. Histogramaxeln anger djupet: 255 (8-bit), 4095 (12-bit) eller 65535 (16-bit) |
| RAW (Debayered) | DN | Linjär, ingen bildsträckning |
| RAW (Förhandsgranskning) / Vitbalanserad | DN | Visningsresultat — sträckt eller gammakorrigerat. Inte avsett för mätning |
| RAW (Strålning) | **W/m²/sr/nm** | Float32 fysisk strålning. Ingen DN-kolumn |
| RAW (Reflektans) | DN **och %** | Procent beräknad med filens egen skala — se nedan |
| Index / LUT / sandbox-exporter | Indexvärde eller RGB-komponenter | En enkelkanalig indexfil anger indexvärdet; en färgkartlagd LUT-fil anger komponenterna Red /Green /Blue |

### Reflektans: skalan är filspecifik

{% hint style="warning" %}
**”Dela med 65 535” är endast korrekt för Survey3.** LATTICE-reflektans lagras i en annan skala, och att blanda ihop de två divisorerna är det vanligaste sättet att få reflektansvärden som är exakt hälften av vad de borde vara.
{% endhint %}

| Källa | DN som motsvarar reflektans 1,0 | Identifieras av |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | XMP-taggen `Chloros:PixelScale=32768` som finns i varje LATTICE-reflektansimport. Det dubbla utrymmet innebär att ρ över 1,0 kan återges istället för att beskäras |
| **Survey3**| **65535** | Ingen Chloros XMP-skalningstagg — Survey3-kalibreringen skriver ρ × dtype-max och beskär vid 1,0 |

För GIS och skript: läs ut `Chloros:PixelScale` från filen och dividera med det. Om taggen saknas är filen i Survey3-skala (65535). Visningsprogrammet, index-/LUT-sandlådan och indexexporten tolkar alla skalan på samma sätt, så det tal du ser vid markören är det tal som indexberäkningen använde.

Formatspecifik lagring utöver den skalan:

* **TIFF (32-bitars, procent)** lagrar DN / 65535 som ett flyttal
* **PNG (8-bitars)**och**JPG (8-bitars)** lagrar DN × 255 / 65535
* Ett **8-bitars DN-värde från en 8-bitars bildtagning**, eller en reflektansfil som skapades före version 1.3.0, beskärs till 0–255 och har ingen skalmärkning. Panelen visar endast DN-värdet för dessa filer, utan procentkolumn

### Indexvärdesintervall

| Indexfamilj | Typiskt intervall | Värde |
| --- | --- | --- |
| Normaliserad skillnad (NDVI, GNDVI, NDRE, ENDVI…) | −1 till +1 | Frisk vegetation vanligtvis 0,4–0,9; bar mark nära 0; vatten negativt |
| Jordjusterat (SAVI, OSAVI, MSAVI2…) | ungefär −1 till +1,5 | Liknande värde som NDVI men med jordbakgrunden undertryckt |
| Förhållande (GRVI, GCI, MSR, CIRE…) | obegränsat uppåt | Förhållandena växer obegränsat när nämnarbandet går mot noll |
| EVI / LAI | 0 till ~1, 0 till ~3,5 | Moln och andra mättade pixlar gör att båda värdena hamnar utanför intervallet – maskera dem först |

Se [Formler för multispektrala index](../project-settings/multispectral-index-formulas.md) för den exakta formeln bakom varje förinställning.

***

## Vanliga arbetsflöden

### Före/efter-jämförelse

1. Välj **RAW (Original)** och notera vinjetteringen och de okalibrerade värdena
2. Byt till **RAW (reflektans)**

3. Jämför — vinjetteringen är borttagen, värdena kalibrerade. Zoom och panorering förblir oförändrade, så att du tittar på samma mark

### Granska ett index över en hel bildserie

1. Öppna den första bearbetade bilden och välj indexlagret
2. Tryck på **→** upprepade gånger — indexlagret följer dig från bild till bild
3. Observera histogrammet i sidofältet medan du går igenom: en bildruta där fördelningen hoppar är värd en närmare titt

### Verifiera kalibreringsmål

1. Välj **RAW (Mål)** på en målbildruta
2. Kontrollera att målet är tydligt synligt och detekterat
3. Gå vidare till nästa målbildruta — mållagret följer med

### Kontrollera reflektansvärdenas noggrannhet

1. Välj **RAW (Reflektans)**

2. Läs av kolumnen**%** i panelen Cursor Values – den är redan korrekt skalad för den filen
3. Kontrollera mot kända material i bilden: frisk vegetation har höga värden i NIR-området och låga i det röda; ett kalibreringsmål bör visa värden nära dess publicerade reflektans

***

## Felsökning

### Ett lager som jag förväntade mig finns inte i rullgardinsmenyn

**Möjliga orsaker**

* Bilden har aldrig bearbetats — endast basen och `RAW (Original)` finns
* Produktens exportknapp är avmarkerad i projektinställningarna
* Produkten gäller inte för den kameran (radian och reflektans på en RGB-master; valfritt index på en enkelbands M3M-monokamera)
* Reflektanskalibreringen hade inget att utgå ifrån — ingen `.daq`-täckning i nedåtgående riktning och inget QA-godkänt mål inom bilden — så bilden föll tillbaka till Vignette Corrected eller Sensor Response

**Åtgärder**

1. Kontrollera körloggen: Chloros anger när en begärd exportprodukt inte kunde genereras och varför
2. Kontrollera exportmarkeringarna för varje produkt i [Projektinställningar](../project-settings/project-settings.md)
3. Kontrollera att produktmappen finns i projektets utdatastruktur
4. Kör om med produkten aktiverad

### Lagerlistan verkar vara inaktuell

Chloros skannar om projektets produktmappar medan en körning pågår och korrigerar saknade lagerregistreringar utifrån vad som faktiskt finns på disken, så ett lager som har exporterats färdigt visas normalt av sig själv i en avläsning. Om du byter bort från bilden och tillbaka tvingas en ny upplösning fram.

### Reflektansvärdena verkar vara hälften av vad de borde vara

Du delar nästan säkert en LATTICE-fil med 65535. Använd `Chloros:PixelScale` (32768), eller läs kolumnen **%**, där detta redan har tillämpats.

### Indexlagret finns men bilden är tom

Indexet kräver band som ditt lager inte har – till exempel ett index som läser av en tredje kanal som tillämpas på en en- eller tvåkanalsfil. Byt till ett flerbandslager (reflektans eller debayered), eller välj ett index som passar kamerans filter.

***

## Nästa steg

* [**Öppna en bild i helskärmsläge**](opening-an-image-full-screen.md) – marköravläsning, histogram och GSD-kontroll
* [**Index/LUT-sandlåda**](index-lut-sandbox.md) – interaktiv indexvisualisering och export
* [**Formler för multispektrala index**](../project-settings/multispectral-index-formulas.md) — indexreferensen
* [**Avsluta bearbetningen**](../processing-images-gui/finishing-the-processing.md) — utdatamappstrukturen som dessa lager pekar mot
