# Justera projektinställningar

Innan du bearbetar dina bilder är det viktigt att konfigurera projektinställningarna så att de passar dina arbetsflödeskrav. Projektinställningspanelen <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> ger omfattande kontroll över kalibrering, bearbetningsalternativ, multispektrala index och exportformat.

## Öppna projektinställningarna

1. Öppna ditt projekt i Chloros
2. Klicka på ikonen **Projektinställningar** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i vänster sidofält
3. Panelen Projektinställningar visar alla konfigurationsalternativ

{% hint style=&quot;info&quot; %}
**Inställningarna sparas automatiskt** med ditt projekt. När du öppnar ett projekt igen återställs alla inställningar.
{% endhint %}

***

## Snabbkonfiguration för vanliga arbetsflöden

### Standardinställningar (rekommenderas för de flesta användare)

För typiska MAPIR Survey3 kamerarbetsflöden fungerar standardinställningarna bra:

* ✅ **Vignettkorrigering**: Aktiverad
* ✅ **Reflektanskalibrering**: Aktiverad (kräver bilder av MAPIR-mål)
* ✅ **Debayer-metod**: Hög kvalitet (snabbare)
* ✅ **Exportformat**: TIFF (16-bitars)

Importera bara dina bilder och börja bearbeta med dessa standardinställningar.

***

## Översikt över projektinställningar

Panelen Projektinställningar är indelad i flera kategorier. Nedan följer en sammanfattning av varje avsnitt. För fullständig dokumentation, se [Projektinställningar](../project-settings/project-settings.md).

### Målidentifiering

Styr hur Chloros identifierar kalibreringsmål i dina bilder.

**Viktiga inställningar:**

* **Minsta kalibreringsprovområde**: Storleksgräns för måldetektering (standard: 25 pixlar)
* **Minsta målkluster**: Likhetsgräns för gruppering av målområden (standard: 60)

**När ska du justera:**

* Öka provområdet om du får falska detekteringar.
* Minska om målen inte detekteras.
* Justera klustringen om målen delas upp i flera detekteringar.

### Bearbetning

Huvudsakliga alternativ för bildbearbetning och kalibrering.

**Viktiga inställningar:**

* **Vignettkorrigering**: Kompenserar för linsens mörkare kanter ✅ Rekommenderas
* **Reflektanskalibrering**: Normaliserar värden med hjälp av kalibreringsmål ✅ Rekommenderas
* **Debayer-metod**: Algoritm för att konvertera RAW till 3-kanals multispektral
* **Minsta omkalibreringsintervall**: Tid mellan användning av kalibreringsmål (0 = använd alla)

**Avancerade inställningar:**

* **Ljussensorns tidszonsförskjutning**: För PPK-tidssynkronisering (standard: 0)
* **Tillämpa PPK-korrigeringar**: Använder GPS-/exponeringsstiftdata från .daq-filer
* **Exponeringsstift 1/2**: Tilldelar kameror till exponeringsstift för dubbla kamerakonfigurationer

### Index (multispektrala index)

Konfigurera vilka vegetationsindex som ska beräknas och exporteras.

**Så här lägger du till index:**

1. Klicka på knappen **”Lägg till index”**
2. Välj ett index från rullgardinsmenyn (NDVI, NDRE, GNDVI, etc.)
3. Konfigurera visualiseringsinställningar (LUT-färger, värdeintervall)
4. Lägg till flera index efter behov

**Populära index:**

* **NDVI**: Allmän vegetationhälsa (vanligast)
* **NDRE**: Tidig stressdetektering med RedEdge
* **GNDVI**: Känslig för klorofyllkoncentration
* **OSAVI**: Fungerar bra med synlig jord
* **EVI**: Regioner med högt bladarealindex (LAI)

**Anpassade formler (endast Chloros+):**

* Skapa anpassade multispektrala indexformler
* Använd bandmatematik med alla bildkanaler
* Spara anpassade formler för återanvändning

För alla tillgängliga index och formler, se [Multispektrala indexformler](../project-settings/multispectral-index-formulas.md).

### Exportera

Styr utdatafilformat och kvalitet.

**Tillgängliga format:**

* **TIFF (16-bitars)**: Rekommenderas för GIS och vetenskaplig analys (intervall 0–65 535)
* **TIFF (32-bitars, procent)**: Flytande reflektansvärden (intervall 0,0–1,0)
* **PNG (8-bitars)**: Förlustfri komprimering för visualisering (intervall 0–255)
* **JPG (8-bitars)**: Minsta filer, förlustrik komprimering (intervall 0–255)

***

## Spara och ladda inställningar

### Spara projektmall

Skapa återanvändbara mallar för enhetliga arbetsflöden:

1. Konfigurera alla önskade inställningar i panelen Projektinställningar.
2. Bläddra till avsnittet **”Spara projektmall”** längst ned.
3. Ange ett beskrivande mallnamn (t.ex. ”Survey3N\_RGN\_Agriculture”).
4. Klicka på ikonen Spara.

**Fördelar:**

* Tillämpa identiska inställningar i flera projekt.
* Dela konfigurationer med teammedlemmar.
* Upprätthåll konsistens för upprepade undersökningar.

### Ladda mall i nytt projekt

När du skapar ett nytt projekt:

1. Välj **&quot;Nytt projekt&quot;** i huvudmenyn.
2. Välj alternativet **&quot;Ladda från mall&quot;**.
3. Välj din sparade mall.
4. Alla inställningar tillämpas automatiskt.

### Arbetsmapp

Inställningen **&quot;Spara projektmapp&quot;** anger var nya projekt skapas som standard:

* **Standardplats**: `C:\Users\[Username]\Chloros Projects`
* **Ändra plats**: Klicka på redigeringsikonen och välj en ny mapp
* **När ska du ändra**:
  * Nätverksenhet för teamsamarbete
  * Annan enhet med mer lagringsutrymme
  * Organiserad mappstruktur efter år/kund

***

## PPK-inställningar (Post-Processed Kinematic)

Om du använder MAPIR DAQ-inspelare med GPS för exakt geolokalisering:

### Förutsättningar

* MAPIR DAQ med GPS-modul (GNSS)
* .daq-loggfil med exponeringstiftposter
* Kamera ansluten till DAQ-exponeringsstift under inspelningssessionen

### Konfigurationssteg

1. Placera .daq-loggfilen i din projektmapp.
2. I Projektinställningar aktiverar du kryssrutan **&quot;Tillämpa PPK-korrigeringar&quot;**.
3. Ställ in **&quot;Ljussensorns tidszonsförskjutning&quot;** om det behövs (standard: 0 för UTC).
4. Tilldela kameror till exponeringsstift:
   * **Enkel kamera**: Tilldelas automatiskt till stift 1
   * **Två kameror**: Tilldela varje kamera manuellt till rätt stift

**Tilldelning av exponeringsstift:**

* **Exponeringsstift 1**: Välj kameramodell från rullgardinsmenyn
* **Exponeringsstift 2**: Välj en andra kamera eller &quot;Använd inte&quot;
* Samma kamera kan inte tilldelas båda stiften

{% hint style=&quot;warning&quot; %}
**Viktigt**: Exponeringsstift måste tilldelas korrekt till respektive kamera. Felaktig tilldelning resulterar i felaktiga geolokaliseringsdata.
{% endhint %}

***

## Avancerade scenarier

### Projekt med flera kameror

När du bearbetar bilder från flera MAPIR-kameror i ett projekt:

1. Chloros identifierar automatiskt varje kameramodell
2. Varje kamera får en lämplig bearbetningsprofil
3. PPK: Tilldela varje kamera manuellt rätt exponeringsstift
4. Alla kameror använder samma exportformat och index

**Exempel**: Survey3W RGN + Survey3N OCN rigg med dubbla kameror

### Tidsfördröjda eller flerdagarsundersökningar

För upprepade undersökningar av samma område över tid:

1. Skapa en mall med dina standardinställningar
2. Använd en konsekvent kalibreringsmålkonfiguration för varje session
3. Bearbeta varje datum som ett separat projekt
4. Använd identiska inställningar för jämförbara resultat
5. Exportera i samma format för tidsanalys

### Stora datamängder

För projekt med många bilder (500+):

* Överväg att dela upp i mindre projekt efter datum eller område.
* Använd Chloros+ parallellbearbetning för snabbare resultat.
* Överväg CLI eller API för batchautomatisering.
* Justera minimalt omkalibreringsintervall för att minska måldetekteringstiden.

***

## Verifiera dina inställningar

Innan du börjar bearbeta, granska dessa viktiga inställningar:

* [ ] Kameramodell korrekt detekterad i filbläddraren
* [ ] Vignettkorrigering aktiverad
* [ ] Reflektanskalibrering aktiverad
* [ ] Minst en kalibreringsmålbild importerad
* [ ] Önskade multispektrala index tillagda
* [ ] Exportformat lämpligt för ditt arbetsflöde
* [ ] PPK-inställningar konfigurerade (om du använder .daq med exponeringshändelser)

***

## Nästa steg

När dina inställningar är konfigurerade:

1. **Markera kalibreringsmålbilder** – Se [Välja målbilder](choosing-target-images.md)
2. **Starta bearbetningen** – Se [Starta bearbetningen](starting-the-processing.md)
3. **Övervaka förloppet** – Se [Övervaka bearbetningen](monitoring-the-processing.md)

För fullständiga detaljer om alla tillgängliga inställningar, se referensdokumentationen [Projektinställningar](../project-settings/project-settings.md).
