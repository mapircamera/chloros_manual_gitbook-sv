# Starta bearbetningen

När du har importerat dina bilder, markerat dina kalibreringsmål och konfigurerat dina projektinställningar är du redo att påbörja bearbetningen. Denna sida guidar dig genom hur du startar bearbetningsflödet Chloros.

## Checklista inför bearbetningen

Innan du klickar på Start-knappen ska du kontrollera att allt är klart:

* [ ] **Filer importerade** – Alla bilder visas i filbläddraren
* [ ] **Målbilder markerade** – Kolumnen ”Mål” har kontrollerats för kalibreringsbilder (eller en `.daq`-inspelning har importerats för LATTICE)
* [ ] **Kameramodeller identifierade** – Kolumnen ”Kameramodell” visar rätt kameror
* [ ] **Inställningar konfigurerade** – Projektinställningarna har granskats och justerats
* [ ] **Index valda** – Önskade multispektrala index har lagts till (vid behov)
* [ ] **Exportformat valt** – Utdataformat som passar ditt arbetsflöde

{% hint style="info" %}
**Tips**: Klicka dig igenom några bilder i filbläddraren för att kontrollera att de har laddats korrekt innan bearbetning.
{% endhint %}

***

## Starta bearbetningen

### Hitta startknappen

Start-/uppspelningsknappen finns i den övre rubrikraden i Chloros:

* Plats: Överst i mitten av fönstret
* Ikon: **Uppspelnings-/startknapp** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Status: Knappen är aktiverad (lysande) när den är redo för bearbetning

### Klicka för att starta

1. Klicka på **uppspelnings-/startknappen** i den övre rubrikraden
2. Bearbetningen påbörjas omedelbart
3. Knappen förvandlas till en **stoppknapp** under bearbetningen
4. Förloppsindikatorn uppdateras och visar bearbetningsstatus

{% hint style="success" %}
**Bearbetning påbörjad**: När du klickar på knappen sköter Chloros automatiskt alla bearbetningssteg – måldetektering, debayering, kalibrering, indexberäkning och export. Programmet känner automatiskt av om ditt projekt är Survey3, LATTICE eller en blandning, och tillämpar rätt bearbetningsflöde för varje kamera.
{% endhint %}

***

## Förstå bearbetningslägen

Chloros fungerar i två olika bearbetningslägen beroende på din licens:

### Gratisläge (sekventiell bearbetning)

**Tillgängligt för alla användare**

**Så här fungerar det:**

* Bearbetar bilderna en i taget, sekventiellt
* Entrådig drift
* Lägre minnesanvändning

**Förloppsindikatorn visar två steg:**

1.**Måldetektering** – Söker efter kalibreringsmål
2. **Bearbetning** – Tillämpar kalibrering och exporterar bilder**Bearbetningstid:**

* Mycket långsammare än Chloros+ parallellt läge
* Lämpligt för små till medelstora datamängder (&lt; 200 bilder)

### Chloros+-läge (Parallell bearbetning)

**Kräver Chloros+-licens**

**Så här fungerar det:**

* Bearbetar flera bilder samtidigt med hjälp av en [bearbetningspipeline med 4 trådar](../processing-architecture/processing-pipeline.md)
* [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md) väljer automatiskt den optimala strategin för din hårdvara vid start
* GPU-acceleration (CUDA) med NVIDIA-grafikkort (stationära datorer och Jetson)
* **Antalet arbetare anpassas efter hårdvaran**: GPU-strategier kör**1–4 samtidiga arbetare** (skalade efter VRAM – en Jetson med lite minne kör 1, en stationär GPU med 12 GB eller mer kör upp till 4); system med enbart CPU kör en arbetare per fysisk kärna, minus en**Förloppsindikatorn visar 4 steg** (motsvarande de 4 trådarna i pipelinen):

1. **Detektering** (tråd 1) – Hittar kalibreringsmål
2. **Analys** (tråd 2) – Granskar bildens metadata och beräknar kalibrering
3. **Kalibrering** (tråd 3) – Debayering, vignettkorrigering, kalibrering, indexberäkning
4. **Export** (tråd 4) – Spara bearbetade bilder och index**Interaktion med förloppsindikatorn:*** **Håll muspekaren** över stapeln för att se en detaljerad rullgardinsmeny med fyra steg
* **Klicka** på förloppsindikatorn för att frysa rullgardinsmenyn på plats
* **Klicka igen** för att låsa upp och dölja menyn**Bearbetningstid:**

* Betydligt snabbare än i gratisläget
* GPU-acceleration förbättrar hastigheten ytterligare

{% hint style="info" %}
**Chloros+ Hastighet**: Parallellbearbetning kan vara 5–10 gånger snabbare än sekventiellt läge för stora datamängder. Ett projekt med 500 bilder som tar 2 timmar i gratisläget kan slutföras på 15–20 minuter med Chloros+.
{% endhint %}

***

## Vad händer under bearbetningen

### Steg 1: Målidentifiering

**Vad Chloros gör:**

* Skannar de bilder du markerat i kolumnen ”Mål” (alla bilder om inga är markerade)
* Identifierar kalibreringspanelerna i varje mål
* Extraherar reflektansvärden från målpanelerna
* Registrerar målets tidsstämplar för kalibreringsschemaläggning

**Varaktighet:** 1–30 sekunder (med markerade mål), 5–30+ minuter (omarkerade)

### Steg 2: Debayering (RAW-konvertering)

**Vad Chloros gör:**

* Konverterar RAW-data med Bayer-mönster till fullständiga 3-kanalsbilder (LATTICE-monomoduler förblir enkelbandsbilder — debayering hoppas över för dem med en anteckning i loggen)
* Tillämpar den valda demosaicing-algoritmen
* Bevarar maximal bildkvalitet och detaljrikedom

**Varaktighet:** Varierar beroende på antal bilder och CPU/GPU-hastighet

### Steg 3: Kalibrering

**Vad Chloros gör:*** **Vignettkorrigering**: Tar bort mörkningen vid bildkanterna
* **Reflektanskalibrering**: Normaliserar med hjälp av målreflektansvärden och/eller DAQ-data för nedåtriktad strålning
* Tillämpar korrigeringar på alla band/kanaler
* Använder lämplig kalibreringsreferens för varje bild baserat på tidsstämpel

**Varaktighet:** Största delen av bearbetningstiden

### Steg 4: Indexberäkning

**Vad Chloros gör:**

* Beräknar konfigurerade multispektrala index (NDVI, NDRE, etc.)
* Tillämpar bandberäkningar på kalibrerade bilder
* Genererar indexbilder för varje valt index

**Varaktighet:** Några sekunder per bild

### Steg 5: Export

**Vad Chloros gör:**

* Sparar bearbetade bilder i det valda formatet
* **LATTICE fan-out**: varje rå LATTICE-bildruta exporteras som alla aktiverade produkter i ett steg — debayered, förhandsgranskning, radiance (alltid float32), reflektans
* Skriver filer till projektets utdatastruktur: `<project>/<camera>/<format>/<Product>_Images/`
* **Behåller källfilnamnet** – mappen identifierar produkten, inget suffix läggs till**Varaktighet:** Varierar beroende på exportformat och filstorlek***

## Bearbetningsbeteende

### Automatisk bearbetningspipeline

När den har startats körs hela pipelinen automatiskt:

* Ingen användarinteraktion krävs
* Alla konfigurerade steg utförs i tur och ordning
* Förloppsuppdateringar visas i realtid
* Exporterade filer skrivs till disken allteftersom de blir färdiga — du kan öppna färdiga utdata medan körningen fortsätter

### Datoranvändning under bearbetningen

**Fritt läge:**

* Relativt låg CPU-användning (enkeltrådad)
* Datorn förblir responsiv för andra uppgifter
* Det är säkert att minimera Chloros och arbeta i andra program

**Chloros+ Parallellt läge:**

* Hög CPU-användning i strategins arbetsgrupp
* Med GPU-acceleration: Hög GPU-användning
* Datorn kan vara mindre responsiv under bearbetningen
* Undvik att starta andra CPU-krävande uppgifter

{% hint style="warning" %}
**Prestandatips**: För bästa Chloros+-prestanda, stäng andra program och låt Chloros använda alla systemresurser.
{% endhint %}

### Bearbetningen kan inte pausas (men det går att avsluta den på ett korrekt sätt)

* När bearbetningen har startat kan den inte pausas och återupptas senare
* Om du klickar på **Stopp** avslutas körningen korrekt redan vid det första klicket
* Produkter som redan har exporterats innan stoppet finns kvar på disken
* En avbruten körning rapporterar korrekt vad som har slutförts (se raderna `[RUN-SUMMARY]` i loggen)
* En ny körning startar pipelinen från början

**Planeringstips:** För mycket stora projekt bör du överväga att bearbeta i omgångar eller använda CLI för bättre kontroll.***

## Övervaka bearbetningen

Medan bearbetningen pågår kan du:

* **Följa förloppsindikatorn** – Se den totala procentuella färdigställandegraden
* **Visa aktuellt steg** – Detektering, analys, kalibrering eller export
* **Kontrollera fliken Logg** – Se detaljerade bearbetningsmeddelanden och varningar
* **Förhandsgranska färdiga bilder** – Exporterade filer visas på disken under bearbetningen

För detaljerad information om övervakning, se [Övervaka bearbetningen](monitoring-the-processing.md).

***

## Avbryta bearbetningen

Om du behöver avbryta bearbetningen:

### Så här avbryter du

1. Leta reda på **Stopp-knappen** (ersätter Start-knappen under bearbetningen)
2. Klicka på den en gång – stapeln visar **”Avbryter...”** medan den pågående bilden slutförs
3. Körningen avslutas i ett definitivt stoppat tillstånd och loggen visar en detaljerad rapport (`[RUN-SUMMARY]`) över vad som har slutförts

### När ska man avbryta

**Giltiga skäl att avbryta:**

* Man har insett att felaktiga inställningar har använts
* Man har glömt att markera målbilder
* Felaktiga bilder har importerats
* Systemet går för långsamt eller svarar inte

**Efter avbrytning:**

* Produkter som exporterades före avbrytningen finns kvar på disken
* Granska och åtgärda eventuella problem, justera inställningarna efter behov
* Starta om bearbetningen – körningen börjar från början

***

## Uppskattad bearbetningstid

Den faktiska bearbetningstiden varierar kraftigt beroende på:

* Antal bilder
* Bildupplösning
* Inmatningsformat (RAW eller JPG)
* Bearbetningsläge (Free eller Chloros+)
* CPU-hastighet och antal kärnor
* Tillgänglighet av GPU (endast Chloros+)
* Antal index att beräkna
* Antal aktiverade exportprodukter (LATTICE)

### Grova uppskattningar (Chloros+, 12 MP-bilder, modern CPU)

| Antal bilder | Gratisläge | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 bilder   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 bilder  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 bilder  | 1–1,5 tim | 20–30 min      | 10–15 min      |
| 500 bilder  | 2–3 timmar   | 45–60 min      | 20–30 min      |
| 1 000 bilder | 4–6 timmar   | 1,5–2 timmar      | 40–60 min      |

{% hint style="info" %}
**Första körningen**: Den inledande bearbetningen kan ta längre tid eftersom Chloros bygger upp cacheminnen och profiler. Efterföljande bearbetning av liknande datamängder kommer att gå snabbare.
{% endhint %}

***

## Vanliga problem vid start

### Startknappen är inaktiverad (gråmarkerad)

**Möjliga orsaker:**

* Inga bilder har importerats
* Backend har inte startat helt
* Tidigare bearbetning pågår fortfarande
* Projektet har inte laddats helt

**Lösningar:**

1. Vänta tills backend har initialiserats helt (kontrollera ikonen i huvudmenyn)
2. Kontrollera att bilderna har importerats i filbläddraren
3. Starta om Chloros om knappen fortfarande är inaktiverad
4. Kontrollera felsökningsloggen för felmeddelanden

### Bearbetningen startar men avbryts omedelbart

**Möjliga orsaker:**

* Inga giltiga bilder i projektet
* Skadade bildfiler
* Otillräckligt diskutrymme
* Otillräckligt minne (RAM)

**Lösningar:**

1. Kontrollera felsökningsloggen <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> för felmeddelanden
2. Kontrollera tillgängligt diskutrymme
3. Försök bearbeta en mindre delmängd av bilderna
4. Kontrollera att bilderna inte är skadade

### Körningen avslutas men inga bilder skrivs ut

En körning som begärde bildprodukter men inte skrev ut några behandlas som ett **misslyckande, inte en framgång** — Chloros rapporterar detta tydligt:

* GUI-loggen visar `[RUN-SUMMARY]`-meddelanden som anger den troliga orsaken — inga bilder importerade, inget mål detekterat, eller att alla begärda produkter hoppats över som icke tillämpliga (t.ex. begäran om radians/reflektans från kameror som endast stöder RGB)
* Motsvarigheten till CLI (`chloros-cli process`) skriver ut `Processing finished but wrote no image products.` och **avslutas med ett värde som inte är noll**, så att skript kan upptäcka det
* En avsiktlig körning med endast metadata (alla exportprodukter inaktiverade, inga index) räknas fortfarande som lyckad

Se [referensen för CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails) för fullständig beskrivning.

### Varningen ”Inga mål upptäckta”

**Möjliga orsaker:**

* Glömde att markera målbilder
* Målbilderna innehåller inga synliga mål
* Inställningarna för måldetektering är för strikta

**Lösningar:**

1. Läs igenom [Att välja målbilder](choosing-target-images.md)
2. Markera lämpliga bilder i kolumnen ”Mål”
3. Kontrollera att målen är synliga i de markerade bilderna
4. Justera inställningarna för måldetektering vid behov

***

## Tips för lyckad bearbetning

### Innan du börjar

1. **Testa först med en liten delmängd** – Bearbeta 10–20 bilder för att verifiera inställningarna
2. **Kontrollera tillgängligt diskutrymme** – Se till att det finns 2–3 gånger datamängdens storlek ledigt (mer om alla LATTICE-produkter är aktiverade)
3. **Stäng onödiga program** – Frigör systemresurser
4. **Kontrollera målbilderna** – Förhandsgranska markerade mål för att säkerställa kvaliteten
5. **Spara projektet** – Projektet sparas automatiskt, men det är bra att spara manuellt

### Under bearbetningen

1. **Undvik att systemet går i viloläge** – Inaktivera energisparlägen
2. **Håll Chloros i förgrunden** – Eller åtminstone synligt i aktivitetsfältet
3. **Övervaka framstegen då och då** – Kontrollera om det finns varningar eller fel
4. **Starta inte andra resurskrävande program** – Särskilt inte när Chloros+ körs i parallellt läge

### Chloros+ GPU-acceleration

Om du använder NVIDIA GPU-acceleration:

1. Uppdatera NVIDIA-drivrutinerna till den senaste versionen
2. Se till att GPU:n har minst 4 GB VRAM (minst 7 GB för samtidig texturmedveten debayering)
3. Stäng GPU-krävande program (spel, videoredigering)
4. Övervaka GPU-temperaturen (se till att kylningen är tillräcklig)

***

## Nästa steg

När bearbetningen har startat:

1. **Övervaka förloppet** – Se [Övervaka bearbetningen](monitoring-the-processing.md)
2. **Vänta tills bearbetningen är klar** – Bearbetningen sker automatiskt
3. **Granska resultaten** – Se [Avsluta bearbetningen](finishing-the-processing.md)

För information om vad du ska göra under bearbetningen, se [Övervaka bearbetningen](monitoring-the-processing.md).
