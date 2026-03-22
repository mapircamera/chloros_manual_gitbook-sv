# Starta bearbetningen

När du har importerat dina bilder, markerat dina kalibreringsmål och konfigurerat dina projektinställningar är du redo att påbörja bearbetningen. Denna sida guidar dig genom hur du startar bearbetningsflödet Chloros.

## Checklista inför bearbetning

Innan du klickar på Start-knappen, kontrollera att allt är klart:

* [ ] **Filer importerade** - Alla bilder visas i filbläddraren
* [ ] **Målbilder markerade** - Målkolumnen är markerad för kalibreringsbilder
* [ ] **Kameramodeller upptäckta** - Kameramodellkolumnen visar rätt kameror
* [ ] **Inställningar konfigurerade** - Projektinställningarna har granskats och justerats
* [ ] **Index valda** – Önskade multispektrala index har lagts till (om nödvändigt)
* [ ] **Exportformat valt** – Utdataformat som passar ditt arbetsflöde

{% hint style="info" %}
**Tips**: Klicka igenom några bilder i filbläddraren för att kontrollera att de har laddats korrekt innan bearbetningen påbörjas.
{% endhint %}

***

## Starta bearbetningen

### Hitta startknappen

Start-/uppspelningsknappen finns i den övre rubrikraden i Chloros:

* Plats: Övre mitten av fönstret
* Ikon: **Uppspelnings-/startknapp** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Status: Knappen är aktiverad (ljus) när den är redo för bearbetning

### Klicka för att starta

1. Klicka på **Spela upp/Start-knappen** i den övre rubrikraden
2. Bearbetningen påbörjas omedelbart
3. Knappen inaktiveras (gråmarkeras) under bearbetningen
4. Förloppsindikatorn uppdateras och visar bearbetningsstatus

{% hint style="success" %}
**Bearbetning påbörjad**: När du klickar på knappen hanterar Chloros automatiskt alla bearbetningssteg – måldetektering, debayering, kalibrering, indexberäkning och export.
{% endhint %}

***

## Förstå bearbetningslägen

Chloros fungerar i två olika bearbetningslägen beroende på din licens:

### Gratis läge (sekventiell bearbetning)

**Tillgängligt för alla användare**

**Så här fungerar det:**

* Bearbetar bilder en i taget, sekventiellt
* Enkeltrådig drift
* Lägre minnesanvändning

**Förloppsindikatorn visar två steg:**

1.**Måligenkänning** – Söker efter kalibreringsmål
2. **Bearbetning** – Tillämpar kalibrering och exporterar bilder**Bearbetningstid:**

* Mycket långsammare än Chloros+ parallellt läge
* Lämpligt för små till medelstora datamängder (&lt; 200 bilder)

### Chloros+-läge (parallell bearbetning)

**Kräver Chloros+-licens**

**Så här fungerar det:**

* Bearbetar flera bilder samtidigt med hjälp av en [4-trådig bearbetningspipeline](../processing-architecture/processing-pipeline.md)
* [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md) väljer automatiskt den optimala strategin för din hårdvara
* GPU-acceleration (CUDA) med NVIDIA-grafikkort (stationära datorer och Jetson)
* Skalar från en Jetson Nano (1 arbetare) till en stationär dator med 12 GB+ GPU (3–4 arbetare)

**Förloppsindikatorn visar 4 steg** (motsvarande de 4 pipeline-trådarna):

1. **Detektering** (Tråd 1) – Hittar kalibreringsmål
2. **Analys** (Tråd 2) – Granskar bildmetadata och beräknar kalibrering
3. **Kalibrering** (Tråd 3) – GPU-debayering, vignettkorrigering, indexberäkning
4. **Exportera** (Tråd 4) – Spara bearbetade bilder och index**Interaktion med förloppsindikatorn:*** **Håll muspekaren** över indikatorn för att se en detaljerad rullgardinsmeny med 4 steg
* **Klicka** på förloppsindikatorn för att frysa rullgardinsmenyn på plats
* **Klicka igen** för att låsa upp och dölja menyn**Bearbetningstid:**

* Betydligt snabbare än gratisläget
* Skalar med antalet CPU-kärnor
* GPU-acceleration förbättrar hastigheten ytterligare

{% hint style="info" %}
**Chloros+ Hastighet**: Parallellbearbetning kan vara 5–10 gånger snabbare än sekventiellt läge för stora datamängder. Ett projekt med 500 bilder som tar 2 timmar i gratisläget kan slutföras på 15–20 minuter med Chloros+.
{% endhint %}

***

## Vad händer under bearbetningen

### Steg 1: Målidentifiering

**Vad Chloros gör:**

* Skannar markerade målbilder (eller alla bilder om inga är markerade)
* Identifierar de 4 kalibreringspanelerna i varje mål
* Extraherar reflektansvärden från målpanelerna
* Registrerar målets tidsstämplar för kalibreringsschemaläggning

**Varaktighet:** 1–30 sekunder (med markerade mål), 5–30+ minuter (omarkerade)

### Steg 2: Debayering (RAW-konvertering)

**Vad Chloros gör:**

* Konverterar RAW-data med Bayer-mönster till fullständiga RGB-bilder
* Använder en högkvalitativ demosaicing-algoritm
* Bevarar maximal bildkvalitet och detaljrikedom

**Varaktighet:** Varierar beroende på antal bilder och CPU-hastighet

### Steg 3: Kalibrering

**Vad Chloros gör:*** **Vignettkorrigering**: Tar bort mörkare kanter vid bildens ytterkanter
* **Reflektanskalibrering**: Normaliserar med hjälp av målreflektansvärden
* Tillämpar korrigeringar över alla band/kanaler
* Använder lämpligt kalibreringsmål för varje bild baserat på tidsstämpel

**Varaktighet:** Största delen av bearbetningstiden

### Steg 4: Indexberäkning

**Vad Chloros gör:**

* Beräknar konfigurerade multispektrala index (NDVI, NDRE, etc.)
* Tillämpar bandmatematik på kalibrerade bilder
* Genererar indexbilder för varje valt index

**Varaktighet:** Några sekunder per bild

### Steg 5: Export

**Vad Chloros gör:**

* Sparar kalibrerade bilder i valt format
* Exporterar indexbilder med konfigurerade LUT-färger
* Skriver filer till undermappar för kameramodeller
* Bevarar ursprungliga filnamn med suffix

**Varaktighet:** Varierar beroende på exportformat och filstorlek***

## Bearbetningsbeteende

### Automatisk bearbetningspipeline

När den har startats körs hela pipelinen automatiskt:

* Ingen användarinteraktion behövs
* Alla konfigurerade steg utförs i sekvens
* Uppdateringar om förloppet visas i realtid

### Datoranvändning under bearbetning

**Fritt läge:**

* Relativt låg CPU-användning (enkeltrådad)
* Datorn förblir responsiv för andra uppgifter
* Det är säkert att minimera Chloros och arbeta i andra program

**Chloros+ Parallellt läge:**

* Hög CPU-användning (flertrådad, upp till 16 kärnor)
* Med GPU-acceleration: Hög GPU-användning
* Datorn kan vara mindre responsiv under bearbetningen
* Undvik att starta andra CPU-krävande uppgifter

{% hint style="warning" %}
**Prestandatips**: För bästa Chloros+ prestanda, stäng andra program och låt Chloros använda fullständiga systemresurser.
{% endhint %}

### Bearbetningen kan inte pausas

**Viktiga begränsningar:**

* När bearbetningen har startat kan den inte pausas
* Du kan avbryta bearbetningen, men framstegen går förlorade
* Delresultat sparas inte
* Måste starta om från början om den avbryts

**Planeringstips:** För mycket stora projekt bör du överväga att bearbeta i omgångar eller använda CLI för bättre kontroll.***

## Övervaka bearbetningen

Medan bearbetningen pågår kan du:

* **Se förloppsindikatorn** – Se den totala procentuella färdigställandegraden
* **Visa aktuellt steg** – Detektera, analysera, kalibrera eller exportera
* **Kontrollera fliken Logg** – Se detaljerade bearbetningsmeddelanden och varningar
* **Förhandsgranska färdiga bilder** – Vissa exportfiler kan visas under bearbetningen

För detaljerad information om övervakning, se [Övervaka bearbetningen](monitoring-the-processing.md).

***

## Avbryta bearbetningen

Om du behöver stoppa bearbetningen:

### Så här avbryter du

1. Leta reda på **knappen Stopp/Avbryt** (ersätter Start-knappen under bearbetningen)
2. Klicka på Stopp-knappen
3. Bearbetningen avbryts omedelbart
4. Delresultat kasseras

### När ska du avbryta

**Giltiga skäl att avbryta:**

* Du har insett att felaktiga inställningar har använts
* Du har glömt att markera målbilder
* Felaktiga bilder har importerats
* Systemet går för långsamt eller svarar inte

**Efter avbrytning:**

* Granska och åtgärda eventuella problem
* Justera inställningarna efter behov
* Starta om bearbetningen från början
* För bästa resultat, stäng Chloros helt och starta om

{% hint style="warning" %}
**Inga delresultat**: Avbrytning raderar all framsteg. Chloros sparar inte delvis bearbetade bilder.
{% endhint %}

***

## Uppskattad bearbetningstid

Den faktiska bearbetningstiden varierar kraftigt beroende på:

* Antal bilder
* Bildupplösning
* Inmatningsformat (RAW eller JPG)
* Bearbetningsläge (Free vs Chloros+)
* CPU-hastighet och antal kärnor
* Tillgång till GPU (endast Chloros+)
* Antal index att beräkna
* Exportformatets komplexitet

### Grova uppskattningar (Chloros+, 12 MP-bilder, modern CPU)

| Antal bilder | Gratis läge | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 bilder   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 bilder  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 bilder  | 1–1,5 tim | 20–30 min      | 10–15 min      |
| 500 bilder  | 2–3 timmar   | 45–60 min      | 20–30 min      |
| 1000 bilder | 4–6 timmar   | 1,5–2 timmar      | 40–60 min      |

{% hint style="info" %}
**Första körningen**: Den initiala bearbetningen kan ta längre tid eftersom Chloros bygger upp cacheminnen och profiler. Efterföljande bearbetning av liknande datamängder kommer att gå snabbare.
{% endhint %}

***

## Vanliga problem vid start

### Startknappen är inaktiverad (gråmarkerad)

**Möjliga orsaker:**

* Inga bilder importerade
* Backend inte helt startad
* Tidigare bearbetning pågår fortfarande
* Projektet inte helt laddat

**Lösningar:**

1. Vänta tills backend har initialiserats helt (kontrollera ikonen i huvudmenyn)
2. Kontrollera att bilderna är importerade i filbläddraren
3. Starta om Chloros om knappen fortfarande är inaktiverad
4. Kontrollera felsökningsloggen för felmeddelanden

### Bearbetningen startar men misslyckas omedelbart

**Möjliga orsaker:**

* Inga giltiga bilder i projektet
* Skadade bildfiler
* Otillräckligt diskutrymme
* Otillräckligt minne (RAM)

**Lösningar:**

1. Kontrollera felsökningsloggen <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> efter felmeddelanden
2. Kontrollera tillgängligt diskutrymme
3. Försök bearbeta en mindre delmängd av bilderna
4. Kontrollera att bilderna inte är skadade

### Varningen ”Inga mål upptäckta”

**Möjliga orsaker:**

* Glömde att markera målbilder
* Målbilderna innehåller inga synliga mål
* Inställningarna för måldetektering är för strikta

**Lösningar:**

1. Läs igenom [Välja målbilder](choosing-target-images.md)
2. Markera lämpliga bilder i kolumnen Mål
3. Kontrollera att målen är synliga i de markerade bilderna
4. Justera inställningarna för måldetektering vid behov

***

## Tips för lyckad bearbetning

### Innan du börjar

1. **Testa först med en liten delmängd** – Bearbeta 10–20 bilder för att verifiera inställningarna
2. **Kontrollera tillgängligt diskutrymme** – Se till att det finns 2–3 gånger datasetets storlek ledigt
3. **Stäng onödiga program** – Frigör systemresurser
4. **Verifiera målbilderna** – Förhandsgranska markerade mål för att säkerställa kvaliteten
5. **Spara projektet** – Projektet sparas automatiskt, men det är bra att spara manuellt

### Under bearbetningen

1. **Undvik att systemet går i viloläge** – Inaktivera energisparlägen
2. **Håll Chloros i förgrunden** – Eller åtminstone synligt i aktivitetsfältet
3. **Övervaka framstegen då och då** – Kontrollera om det finns varningar eller fel
4. **Ladda inte andra resurskrävande program** – Särskilt inte med Chloros+ parallellläge

### Chloros+ GPU-acceleration

Om du använder NVIDIA GPU-acceleration:

1. Uppdatera NVIDIA-drivrutinerna till den senaste versionen
2. Se till att GPU:n har minst 4 GB VRAM
3. Stäng GPU-krävande program (spel, videoredigering)
4. Övervaka GPU-temperaturen (se till att kylningen är tillräcklig)

***

## Nästa steg

När bearbetningen har startat:

1. **Övervaka förloppet** – Se [Övervaka bearbetningen](monitoring-the-processing.md)
2. **Vänta tills processen är klar** – Bearbetningen körs automatiskt
3. **Granska resultaten** – Se [Avsluta bearbetningen](finishing-the-processing.md)

För information om vad du ska göra under bearbetningen, se [Övervaka bearbetningen](monitoring-the-processing.md).
