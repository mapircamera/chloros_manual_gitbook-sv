# Välja målbilder

Genom att markera vilka bilder som innehåller kalibreringsmål anger du för Chloros exakt var programmet ska leta efter dem. När minst en bild är markerad i kolumnen ”Mål” skannar Chloros **endast de markerade bilderna** – så genom att markera mål kan du både påskynda bearbetningen och förhindra att kartläggningsbilder misstas för mål.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Varför markera målbilder?

### Markeringen styr skanningen

När du markerar specifika bilder i kolumnen ”Mål”:

* Chloros skannar endast de markerade bilderna efter mål
* Målidentifieringen slutförs mycket snabbare
* Kartläggningsbilder kan inte orsaka falska målidentifieringar

Om **inga** bilder är markerade går Chloros tillbaka till att skanna varje bild i projektet:

* Algoritmerna för måldetektering körs på varje bild
* Hundratals eller tusentals bilder kontrolleras i onödan
* Bearbetningen tar betydligt längre tid, särskilt för stora datamängder

{% hint style="success" %}
**Hastighetsförbättring**: Genom att markera 2–3 målbilder i en datamängd med 500 bilder kan tiden för måldetektering minskas från över 30 minuter till under 1 minut.
{% endhint %}

***

## Så här markerar du målbilder

### Steg 1: Identifiera dina målbilder

Bläddra igenom dina importerade bilder i filbläddraren och identifiera vilka bilder som innehåller kalibreringsmål.

**Vanliga scenarier:*** **Mål före bildtagning**: Taget innan sessionen påbörjas
* **Mål efter bildtagning**: Tagna efter att sessionen avslutats
* **Mål i fält**: Mål placerade inom bildtagningsområdet
* **Flera mål**: 2–3 målbilder per session (rekommenderas)

### Steg 2: Markera rutan i kolumnen **Target** <img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

För varje bild som innehåller ett kalibreringsmål:

1. Leta reda på bilden i tabellen i filbläddraren
2. Leta reda på kolumnen **Target** (kolumnen längst till höger)
3. Markera rutan i kolumnen **Target** för den bilden
4. Upprepa för alla bilder som innehåller mål

### Steg 3: Kontrollera ditt val

Innan bearbetningen påbörjas, kontrollera noga att:

* [ ] Alla bilder med kalibreringsmål är markerade
* [ ] Inga bilder som inte är mål är av misstag markerade
* [ ] Målen är tydligt synliga i de markerade bilderna

***

## LATTICE: Mål är valfria när en DAQ spelar in

För LATTICE-multispektralkameror är ett kalibreringsmål i bildrutan **en av två** möjliga reflektansreferenser:

* **Målobjekt i bildrutan**: när en markerad målbild klarChloross kvalitetskontroll (QA) blir målobjektet den**absoluta reflektansreferensen** för bildmaterialet runt omkring.
* **DAQ-nedåtriktad strålning**: när inget mål finns (eller om QA-kontrollen misslyckas) beräknar Chloros istället reflektansen utifrån den nedåtriktade strålningsintensiteten från DAQ-ljussensorn (ρ = π·L/E). Om en `.daq`- eller DAQ-M `.csv`-inspelning täcker dina bilder får du kalibrerad reflektans**utan några målbilder alls**.

Detta automatiska beteende är standardinställningen. I CLI / SDK motsvarar det `--reflectance-source auto`; du kan också tvinga fram `target` (strikt — ingen DAQ-ersättning) eller `daq` (DAQ-auktoritativt). Se [CLI-referensen](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**LATTICE-målgeometrier**: förutom den klassiska paneldetekteringen som används för Survey3 stöder LATTICE-bearbetningen**ArUco-märkta mål**,**mål med fast ROI**och**remsmål**, konfigurerade per projekt.**Uppmätta** skanningar av målets reflektans per enhet kan tillhandahållas via serienummer (CLI: `--target-reflectance-dir`, en `<serial>.csv` per målenhet), med de nominella T3/T4P-spektren som reserv.

{% hint style="info" %}
**F988-modul**: F988-reflektansen kalibreras med hjälp av en reflektanspanel i scenen: bandet ligger utanför DAQ-ljussensorns kalibrerade område, så Chloros använder din senaste panelavläsning och behåller den mellan panelavläsningarna. Om en F988-modul bearbetas enbart med DAQ avvisar Chloros DAQ-baserad reflektans för det bandet (hoppningsorsak `dls-uncalibrated-band-988`) – arbetsflödet med paneler är den stödda metoden.
{% endhint %}

***

## Bästa praxis för målbilder

### Riktlinjer för målbildsinsamling

**Tidpunkt:**

* Ta målbilder omedelbart före och under hela din insamlingssession
* Under samma ljusförhållanden som din DAQ-ljussensor
* Ta helst målbilder så ofta som möjligt för bästa resultat. I annat fall kommer ljussensordata att användas för att justera kalibreringen över tid.

**Kameraposition:**

* Håll kameran ovanför målet så att det är centrerat och fyller cirka 40–60 % av bildens mitt.
* Håll kameran parallell med/vinkelrätt mot målytan

**Belysning:**

* Samma omgivande belysning som din DAQ-ljussensor
* Undvik skuggor på målytorna
* Blockera inte ljuskällan med din kropp, ditt fordon eller vegetation
* Molnigt väder ger de mest jämna resultaten

**Målets skick:**

* Håll målpanelerna rena och torra
* Alla paneler på ditt mål (t.ex. alla 4 på en T4) ska vara tydligt synliga och fria från hinder
* Placera målen vinkelrätt mot/rakt under ljuskällan om möjligt

### Hur många målbilder?

**Minimum:**1 målbild per session.**Rekommenderat:** 3–5 målbilder per session.**Rekommenderat schema:**

* Ta 3–5 bilder strax efter att ljussensorn börjat registrera
* Rotera kameran mellan bildtagningarna för bästa resultat
* Valfritt: med jämna mellanrum under sessionen om ljusförhållandena förändras kontinuerligt

***

## Arbeta med flera kameror

### Konfigurationer med två kameror

Om du använder två MAPIR-kameror samtidigt (t.ex. Survey3W RGN + Survey3N OCN):

1. Ta målbilder med **båda kamerorna** samtidigt
2. Använd **samma fysiska mål** för båda kamerorna
3. Markera målbilderna för **båda kameratyperna** i filbläddraren
4. Chloros kommer att använda lämpliga mål för kalibreringen av respektive kamera

### Kolumnen Kameramodell

Kolumnen **Kameramodell** hjälper till att identifiera vilka bilder som kommer från vilken kamera:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* osv.

Använd denna kolumn för att kontrollera att du har markerat mål för varje kameratyp i ditt projekt.

***

## Inställningar för måldetektering

### Justera detekteringskänsligheten

Om Chloros inte detekterar dina mål korrekt, justera dessa inställningar i [Projektinställningar](adjusting-project-settings.md):**Minsta kalibreringsprovområde (px):*** **Standard**: 25 pixlar
* **Öka** om du får falska detekteringar på små artefakter
* **Minska** om målen inte detekteras**Minsta målkluster (0–100):*** **Standard**: 60
* **Öka** om målen delas upp i flera detekteringar
* **Minska** om mål med färgvariationer inte detekteras fullständigt

{% hint style="info" %}
**Tips från CLI**: `chloros-cli process` accepterar samma reglage (`--min-target-size`, `--target-clustering`), och dess flagga `--target`/`--targets` markerar en hel inmatningsmapp som endast för målpanelen. Se [CLI-referensen](../reference/cli-reference.md).
{% endhint %}

***

## Vanliga problem med målbilder

### Problem: Inga mål upptäckta

**Möjliga orsaker:**

* Målbilderna är inte markerade i filbläddraren
* Målet är för litet i bildrutan (&lt; 30 % av bilden)
* Dålig belysning (skuggor, bländning)
* Inställningarna för måldetektering är för strikta

**Lösningar:**

1. Kontrollera att kolumnen ”Mål” är markerad för rätt bilder
2. Kontrollera målbildernas kvalitet i förhandsgranskningen
3. Ta om bilderna på målen om kvaliteten är dålig
4. Justera inställningarna för måldetektering vid behov

### Problem: Felaktiga måldetekteringar

**Möjliga orsaker:**

* Vita byggnader, fordon eller markvegetation som misstas för mål
* Ljusa fläckar i vegetationen
* För låg detekteringskänslighet

**Lösningar:**

1. Markera endast faktiska målbilder – endast markerade bilder skannas
2. Öka det minsta kalibreringsprovområdet
3. Öka det minsta värdet för målkluster
4. Se till att målbilderna endast visar målet (minimalt med störande bakgrund)

***

## Kontrollista för verifiering

Innan bearbetningen påbörjas, kontrollera ditt urval av målbilder:

* [ ] Minst 1 målbild markerad per session (eller, för LATTICE, en `.daq`/`.csv`-inspelning som täcker sessionen)
* [ ] Kryssrutorna i målkolumnen är markerade för alla målbilder
* [ ] Målbilder tagna inom samma tidsram som undersökningen
* [ ] Målen syns tydligt i förhandsgranskningen när man klickar på dem
* [ ] Alla kalibreringspaneler syns i varje målbild
* [ ] Inga skuggor eller hinder på målen
* [ ] Vid användning av dubbla kameror: Målmarkeringar finns för båda kameratyperna

***

## Bearbetning utan målmarkeringar

### LATTICE: Med en DAQ-inspelning

Om en DAQ-ljussensor har registrerat nedåtriktad strålningsintensitet under dina LATTICE-inspelningar behövs inga målmarkeringar:

1. Importera filen `.daq` (eller DAQ-M `.csv`) med bildmaterialet
2. Lämna kolumnen ”Mål” avmarkerad
3. Reflektansen beräknas automatiskt utifrån DAQ:s nedåtriktade referens
4. Strålningsintensiteten behöver aldrig något mål eller någon DAQ – den härleds enbart från kamerans fabriksinställda radiometriska kalibrering

### Bearbetning utan någon referens

Du kan även bearbeta utan mål och utan en DAQ:

1. Lämna alla kryssrutor i kolumnen ”Mål” avmarkerade
2. **Inaktivera** ”Reflektanskalibrering / vitbalans” i projektinställningarna – då hoppas måldetekteringen över helt
3. Vignettkorrigering tillämpas fortfarande
4. Utdata kalibreras inte för absolut reflektans (LATTICE multispektral exporterar fortfarande debayered-, förhandsgransknings- och radiance-produkter)

{% hint style="warning" %}
**Rekommenderas inte för vetenskapligt arbete med Survey3**: Utan reflektanskalibrering representerar pixelvärdena i Survey3 endast relativ ljusstyrka, inte vetenskapliga reflektansmätningar. Använd kalibreringsmål (eller, för LATTICE, en DAQ-ljussensor) för exakta, repeterbara resultat.
{% endhint %}

***

## Nästa steg

När du har markerat dina målbilder:

1. **Granska dina inställningar** – Se [Justera projektinställningar](adjusting-project-settings.md)
2. **Starta bearbetningen** – Se [Starta bearbetningen](starting-the-processing.md)
3. **Övervaka förloppet** – Se [Övervaka bearbetningen](monitoring-the-processing.md)

För mer information om själva kalibreringsmålen, se [Kalibreringsmål](../calibration-targets.md).
