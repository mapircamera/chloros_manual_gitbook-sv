# Välja målbilder

Att markera vilka bilder som innehåller kalibreringsmål är ett viktigt steg som avsevärt påskyndar Chloros-bearbetningsprocessen. Genom att förvälja målbilder slipper du att Chloros behöver skanna varje bild i din dataset efter kalibreringsmål.

## Varför markera målbilder?

### Bearbetningshastighet

Utan att markera målbilder måste Chloros:

* Skanna varje enskild bild i ditt projekt.
* Köra algoritmer för måldetektering på varje bild.
* Kontrollera hundratals eller tusentals bilder i onödan.

**Resultat**: Bearbetningen kan ta betydligt längre tid, särskilt för stora datamängder.

### Med markerade målbilder

När du markerar specifika bilder i kolumnen Mål:

* Chloros skannar endast de markerade bilderna efter mål
* Målidentifieringen slutförs mycket snabbare
* Den totala bearbetningstiden minskas avsevärt

{% hint style=&quot;success&quot; %}
**Hastighetsförbättring**: Genom att markera 2–3 målbilder i en datamängd med 500 bilder kan måldetekteringstiden minskas från över 30 minuter till under 1 minut.
{% endhint %}

***

## Hur man markerar målbilder

### Steg 1: Identifiera dina målbilder

Gå igenom dina importerade bilder i filbläddraren och identifiera vilka bilder som innehåller kalibreringsmål.

**Vanliga scenarier:**

* **Mål före inspelning**: Inspelat innan sessionen startade.
* **Mål efter inspelning**: Inspelat efter att sessionen avslutats.
* **Mål i fält**: Mål placerade inom inspelningsområdet.
* **Flera mål**: 2–3 målbilder per session (rekommenderas).

### Steg 2: Kontrollera målkolumnen

För varje bild som innehåller ett kalibreringsmål:

1. Leta reda på bilden i tabellen i filbläddraren.
2. Leta reda på kolumnen **Mål** (kolumnen längst till höger).
3. Klicka på kryssrutan i kolumnen Mål för den bilden.
4. Upprepa för alla bilder som innehåller mål.

### Steg 3: Kontrollera ditt val

Innan bearbetningen, dubbelkolla:

* [ ] Alla bilder med kalibreringsmål är markerade.
* [ ] Inga bilder utan mål är markerade av misstag.
* [ ] Målen är tydligt synliga i de markerade bilderna.

***

## Bästa praxis för målbilder

### Riktlinjer för målfångst

**Tidpunkt:**

* Ta målbilderna omedelbart före och under hela din fotograferingssession.
* Under samma ljusförhållanden som din DAQ-ljussensor.
* Ta helst målbilder så ofta som möjligt för bästa resultat. Annars kommer ljussensorns data att användas för att justera kalibreringen över tid.

**Kameraposition:**

* Håll kameran ovanför målet så att det är centrerat och fyller cirka 40–60 % av bildens centrum.
* Håll kameran parallell/nadir med målets yta

**Belysning:**

* Samma omgivande belysning som din DAQ-ljussensor.
* Undvik skuggor på målytorna.
* Blockera inte ljuskällan med din kropp, fordon eller vegetation.
* Molniga förhållanden ger mest konsekventa resultat.

**Målförhållanden:**

* Håll målpanelerna rena och torra.
* Alla fyra paneler ska vara tydligt synliga och fria från hinder.
* Målen ska vara vinkelräta/nadir mot ljuskällan om möjligt.

### Hur många målbilder?

**Minimum:** 1 målbild per session. **Rekommenderat:** 3–5 målbilder per session.

**Bästa praxis:**

* 3–5 bilder tagna strax efter att ljussensorn har börjat spela in.
* Rotera kameran mellan bilderna för bästa resultat.
* Valfritt: regelbundet mitt i sessionen om ljusförhållandena förändras hela tiden.

***

## Arbeta med flera kameror

### Konfiguration med två kameror

Om du använder två MAPIR-kameror samtidigt (t.ex. Survey3W RGN + Survey3N OCN):

1. Ta bilder av motivet med **båda kamerorna** samtidigt.
2. Använd **samma fysiska motiv** för båda kamerorna.
3. Markera bilderna för **båda kameratyperna** i filbläddraren.
4. Chloros använder lämpliga motiv för kalibrering av varje kamera.

### Kolumn för kameramodell

Kolumnen **Kameramodell** hjälper dig att identifiera vilka bilder som kommer från vilken kamera:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* etc.

Använd denna kolumn för att kontrollera att du har markerat mål för varje kameratyp i ditt projekt.

***

## Inställningar för måldetektering

### Justera detekteringskänsligheten

Om Chloros inte detekterar dina mål korrekt, justera dessa inställningar i [Projektinställningar](adjusting-project-settings.md):

**Minsta kalibreringsprovområde:**

* **Standard**: 25 pixlar
* **Öka** om du får falska detekteringar på små artefakter
* **Minska** om målen inte detekteras

**Minsta målkluster:**

* **Standard**: 60
* **Öka** om målen delas upp i flera detekteringar
* **Minska** om mål med färgvariationer inte detekteras fullständigt

***

## Vanliga problem med målbilder

### Problem: Inga mål detekteras

**Möjliga orsaker:**

* Målbilderna är inte markerade i filbläddraren
* Målet är för litet i ramen (&lt; 30 % av bilden)
* Dålig belysning (skuggor, bländning)
* För strikta inställningar för måldetektering

**Lösningar:**

1. Kontrollera att kolumnen Mål är markerad för korrekta bilder.
2. Granska målbildens kvalitet i förhandsgranskningen.
3. Ta om målen om kvaliteten är dålig.
4. Justera inställningarna för måldetektering om det behövs.

### Problem: Falska måldetekteringar

**Möjliga orsaker:**

* Vita byggnader, fordon eller markbeläggning som misstas för mål
* Ljusa fläckar i vegetationen
* För låg detektionskänslighet

**Lösningar:**

1. Markera endast faktiska målbilder för att begränsa detektionsområdet
2. Öka minimalt kalibreringsprovområde
3. Öka minimivärdet för målkluster
4. Se till att målbilderna endast visar målet (minimalt med bakgrundsstörningar)

***

## Kontrollista för verifiering

Innan du påbörjar bearbetningen, verifiera ditt val av målbilder:

* [ ] Minst 1 målbild markerad per session
* [ ] Kontrollrutorna för målkolumnen är markerade för alla målbilder
* [ ] Målbilderna är tagna inom samma tidsram som undersökningen
* [ ] Målen är tydligt synliga i förhandsgranskningen när man klickar på dem
* [ ] Alla 4 kalibreringspaneler är synliga i varje målbild
* [ ] Inga skuggor eller hinder på målen
* [ ] För dubbla kameror: Mål markerade för båda kameratyperna

***

## Målfri bearbetning

### Bearbetning utan kalibreringsmål

Även om det inte rekommenderas för vetenskapligt arbete kan du bearbeta utan mål:

1. Lämna alla kryssrutor i målkolumnen omarkerade
2. **Inaktivera** &quot;Reflektanskalibrering&quot; i projektinställningarna
3. Vignettkorrigering kommer fortfarande att tillämpas
4. Utmatningen kommer inte att kalibreras för absolut reflektans

{% hint style=&quot;warning&quot; %}
**Rekommenderas inte**: Utan reflektanskalibrering representerar pixelvärden endast relativ ljusstyrka, inte vetenskapliga reflektansmätningar. Använd kalibreringsmål för exakta, repeterbara resultat.
{% endhint %}

***

## Nästa steg

När du har markerat dina målbilder:

1. **Granska dina inställningar** – Se [Justera projektinställningar](adjusting-project-settings.md)
2. **Starta bearbetningen** - Se [Starta bearbetningen](starting-the-processing.md)
3. **Övervaka framstegen** - Se [Övervaka bearbetningen](monitoring-the-processing.md)

För mer information om kalibreringsmål, se [Kalibreringsmål](../calibration-targets.md).
