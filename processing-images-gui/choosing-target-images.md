# Välja målbilder

Att markera vilka bilder som innehåller kalibreringsmål är ett avgörande steg som avsevärt påskyndar Chloros:s bearbetningsflöde. Genom att i förväg välja ut målbilder slipper du att Chloros behöver söka igenom varje bild i din datamängd efter kalibreringsmål.

## Varför markera målbilder?

### Bearbetningshastighet

Utan att markera målbilder måste Chloros:

* Skanna varje enskild bild i ditt projekt
* Köra algoritmer för måldetektering på varje bild
* Kontrollera hundratals eller tusentals bilder i onödan

**Resultat**: Bearbetningen kan ta betydligt längre tid, särskilt för stora dataset.

### Med markerade målbilder

När du markerar kolumnen Mål för specifika bilder:

* Skannar Chloros endast de markerade bilderna efter mål
* Sluts måldetekteringen mycket snabbare
* Minskas den totala bearbetningstiden avsevärt

{% hint style="success" %}
**Hastighetsförbättring**: Att markera 2–3 målbilder i en dataset med 500 bilder kan minska måldetekteringstiden från över 30 minuter till under 1 minut.
{% endhint %}

***

## Hur man markerar målbilder

### Steg 1: Identifiera dina målbilder

Bläddra igenom dina importerade bilder i filbläddraren och identifiera vilka bilder som innehåller kalibreringsmål.

**Vanliga scenarier:*** **Mål före bildtagning**: Taget före sessionens start
* **Mål efter bildtagning**: Taget efter sessionens slut
* **Mål i fältet**: Mål placerade inom bildtagningsområdet
* **Flera mål**: 2–3 målbilder per session (rekommenderas)

### Steg 2: Kontrollera kolumnen Mål

För varje bild som innehåller ett kalibreringsmål:

1. Leta reda på bilden i filbläddrarens tabell
2. Hitta kolumnen **Mål** (kolumnen längst till höger)
3. Klicka i kryssrutan i kolumnen Mål för den bilden
4. Upprepa för alla bilder som innehåller mål

### Steg 3: Verifiera ditt val

Innan bearbetning, dubbelkolla:

* [ ] Alla bilder med kalibreringsmål är markerade
* [ ] Inga bilder som inte är mål är av misstag markerade
* [ ] Målen är tydligt synliga i de markerade bilderna

***

## Bästa praxis för målbilder

### Riktlinjer för målbildstagning

**Tidpunkt:**

* Ta målbilder omedelbart före och under hela din bildtagning
* Under samma ljusförhållanden som din DAQ-ljussensor
* För bästa resultat bör du helst ta målbilder så ofta som möjligt. Annars kommer ljussensorns data att användas för att justera kalibreringen över tid.

**Kameraposition:**

* Håll kameran ovanför målet så att det är centrerat och fyller cirka 40–60 % av bildens mitt.
* Håll kameran parallell/nadir mot målytan

**Belysning:**

* Samma omgivande belysning som din DAQ-ljussensor
* Undvik skuggor på målytorna
* Blockera inte ljuskällan med din kropp, ditt fordon eller vegetation
* Molnigt väder ger de mest konsekventa resultaten

**Målförhållanden:**

* Håll målpanelerna rena och torra
* Alla 4 paneler ska vara tydligt synliga och fria från hinder
* Målen ska om möjligt vara vinkelräta mot/i nadirläge i förhållande till ljuskällan

### Hur många målbilder?

**Minimum:**1 målbild per session.**Rekommenderat:** 3–5 målbilder per session.**Bästa praxis-schema:**

* 3–5 bilder tagna strax efter att ljussensorn börjat spela in
* Rotera kameran mellan tagningarna för bästa resultat
* Valfritt: regelbundet under sessionen om ljusförhållandena förändras konstant

***

## Arbeta med flera kameror

### Konfigurationer med två kameror

Om du använder två MAPIR-kameror samtidigt (t.ex. Survey3W RGN + Survey3N OCN):

1. Ta bilder av målet med **båda kamerorna** samtidigt
2. Använd **samma fysiska mål** för båda kamerorna
3. Markera målbilderna för **båda kameratyperna** i filbläddraren
4. Chloros använder lämpliga mål för kalibrering av varje kamera

### Kolumnen Kameramodell

Kolumnen **Kameramodell** hjälper till att identifiera vilka bilder som kommer från vilken kamera:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* etc.

Använd denna kolumn för att kontrollera att du har markerat mål för varje kameratyp i ditt projekt.

***

## Inställningar för måldetektering

### Justera detekteringskänsligheten

Om Chloros inte detekterar dina mål korrekt, justera dessa inställningar i [Projektinställningar](adjusting-project-settings.md):**Minsta kalibreringsprovområde:*** **Standard**: 25 pixlar
* **Öka** om du får falska detekteringar på små artefakter
* **Minska** om mål inte detekteras**Minsta målkluster:*** **Standard**: 60
* **Öka** om mål delas upp i flera detekteringar
* **Minska** om mål med färgvariationer inte detekteras fullständigt***

## Vanliga problem med målbilder

### Problem: Inga mål detekterade

**Möjliga orsaker:**

* Målbilder är inte markerade i filbläddraren
* Målet är för litet i bilden (&lt; 30 % av bilden)
* Dålig belysning (skuggor, bländning)
* Inställningarna för måldetektering är för strikta

**Lösningar:**

1. Kontrollera att kolumnen Mål är markerad för rätt bilder
2. Granska målbildens kvalitet i förhandsgranskningen
3. Ta om bilder på målen om kvaliteten är dålig
4. Justera inställningarna för måldetektering vid behov

### Problem: Felaktiga måldetekteringar

**Möjliga orsaker:**

* Vita byggnader, fordon eller markvegetation som misstas för mål
* Ljusa fläckar i vegetationen
* För låg detekteringskänslighet

**Lösningar:**

1. Markera endast faktiska målbilder för att begränsa detekteringsomfånget
2. Öka det minsta kalibreringsprovområdet
3. Öka det minsta värdet för målkluster
4. Se till att målbilderna endast visar målet (minimalt med störande bakgrund)

***

## Kontrollista för verifiering

Innan bearbetningen påbörjas, kontrollera ditt urval av målbilder:

* [ ] Minst 1 målbild markerad per session
* [ ] Kryssrutorna i målkolumnen är markerade för alla målbilder
* [ ] Målbilder tagna inom samma tidsram som undersökningen
* [ ] Mål tydligt synliga i förhandsgranskningen när man klickar på dem
* [ ] Alla 4 kalibreringspaneler synliga i varje målbild
* [ ] Inga skuggor eller hinder på målen
* [ ] För dubbla kameror: Mål markerade för båda kameratyperna

***

## Bearbetning utan mål

### Bearbetning utan kalibreringsmål

Även om det inte rekommenderas för vetenskapligt arbete kan du bearbeta utan mål:

1. Lämna alla kryssrutor i kolumnen Mål avmarkerade
2. **Inaktivera** &quot;Reflektanskalibrering&quot; i Projektinställningar
3. Vignettkorrigering kommer fortfarande att tillämpas
4. Utdata kommer inte att kalibreras för absolut reflektans

{% hint style="warning" %}
**Rekommenderas inte**: Utan reflektanskalibrering representerar pixelvärdena endast relativ ljusstyrka, inte vetenskapliga reflektansmätningar. Använd kalibreringsmål för exakta, repeterbara resultat.
{% endhint %}

***

## Nästa steg

När du har markerat dina målbilder:

1. **Granska dina inställningar** – Se [Justera projektinställningar](adjusting-project-settings.md)
2. **Starta bearbetningen** – Se [Starta bearbetningen](starting-the-processing.md)
3. **Övervaka förloppet** – Se [Övervaka bearbetningen](monitoring-the-processing.md)

För mer information om kalibreringsmål, se [Kalibreringsmål](../calibration-targets.md).
