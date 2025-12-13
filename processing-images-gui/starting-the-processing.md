# Starta bearbetningen

När du har importerat dina bilder, markerat dina kalibreringsmål och konfigurerat dina projektinställningar är du redo att börja bearbeta. Denna sida guidar dig genom att starta Chloros-bearbetningspipeline.

## Checklista före bearbetning

Innan du klickar på Start-knappen, kontrollera att allt är klart:

* [ ] **Filer importerade** – Alla bilder visas i filbläddraren
* [ ] **Målbilder markerade** – Målkolumnen är markerad för kalibreringsbilder
* [ ] **Kameramodeller upptäckta** – Kameramodellkolumnen visar rätt kameror
* [ ] **Inställningar konfigurerade** – Projektinställningarna granskade och justerade
* [ ] **Index valda** – Önskade multispektrala index tillagda (om nödvändigt)
* [ ] **Exportformat valt** – Utmatningsformat som passar ditt arbetsflöde

{% hint style=&quot;info&quot; %}
**Tips**: Klicka igenom några bilder i filbläddraren för att kontrollera att de har laddats korrekt innan bearbetningen.
{% endhint %}

***

## Starta bearbetningen

### Hitta startknappen

Start-/uppspelningsknappen finns i den övre rubrikraden i Chloros:

* Position: Övre mitten av fönstret
* Ikon: **Uppspelnings-/startknapp** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Status: Knappen är aktiverad (ljus) när den är redo för bearbetning

### Klicka för att starta

1. Klicka på **knappen Spela/Starta** i den övre rubriken
2. Bearbetningen startar omedelbart
3. Knappen blir inaktiverad (grå) under bearbetningen
4. Progressionsfältet uppdateras och visar bearbetningsstatus

{% hint style=&quot;success&quot; %}
**Bearbetning påbörjad**: När du har klickat hanterar Chloros automatiskt alla bearbetningssteg – måldetektering, debayering, kalibrering, indexberäkning och export.
{% endhint %}

***

## Förstå bearbetningslägen

Chloros fungerar i två olika bearbetningslägen beroende på din licens:

### Gratis läge (sekventiell bearbetning)

**Tillgängligt för alla användare**

**Så här fungerar det:**

* Bearbetar bilder en i taget, sekventiellt.
* Enkelsträngad drift.
* Lägre minnesanvändning.

**Förloppsindikatorn visar två steg:**

1. **Måldetektering** – Söker efter kalibreringsmål.
2. **Bearbetning** – Tillämpar kalibrering och exporterar bilder.

**Bearbetningstid:**

* Mycket långsammare än Chloros+ parallellt läge
* Lämpligt för små till medelstora datamängder (&lt; 200 bilder)

### Chloros+ läge (parallell bearbetning)

**Kräver Chloros+ licens**

**Så fungerar det:**

* Bearbetar flera bilder samtidigt
* Multitrådad drift (upp till 16 parallella arbetare)
* Utnyttjar flera CPU-kärnor
* Valfri GPU-acceleration (CUDA) med NVIDIA-grafikkort

**Förloppsindikatorn visar fyra steg:**

1. **Detektering** – Hitta kalibreringsmål
2. **Analys** – Undersöka bildmetadata och förbereda pipeline
3. **Kalibrering** – Tillämpar korrigeringar och kalibreringar
4. **Export** – Sparar bearbetade bilder och index

**Interaktion med förloppsindikatorn:**

* **Håll muspekaren** över indikatorn för att se en detaljerad rullgardinsmeny med fyra steg
* **Klicka** på förloppsindikatorn för att frysa rullgardinsmenyn på plats
* **Klicka igen** för att låsa upp och dölja panelen

**Bearbetningstid:**

* Betydligt snabbare än fritt läge
* Skalas med antalet CPU-kärnor
* GPU-acceleration förbättrar hastigheten ytterligare

{% hint style=&quot;info&quot; %}
**Chloros+ Hastighet**: Parallell bearbetning kan vara 5-10 gånger snabbare än sekventiellt läge för stora datamängder. Ett projekt med 500 bilder som tar 2 timmar i gratis läge kan slutföras på 15-20 minuter med Chloros+.
{% endhint %}

***

## Vad händer under bearbetningen

### Steg 1: Målidentifiering

**Vad Chloros gör:**

* Skannar markerade målbilder (eller alla bilder om inga är markerade)
* Identifierar de fyra kalibreringspanelerna i varje mål
* Extraherar reflektansvärden från målpanelerna
* Registrerar måltidsstämplar för kalibreringsschemaläggning

**Varaktighet:** 1–30 sekunder (med markerade mål), 5–30+ minuter (omärkta)

### Steg 2: Debayering (RAW-konvertering)

**Vad Chloros gör:**

* Konverterar RAW-data i Bayer-mönster till fullständiga RGB-bilder
* Tillämpar högkvalitativ demosaicing-algoritm
* Bevarar maximal bildkvalitet och detaljrikedom

**Varaktighet:** Varierar beroende på antal bilder och CPU-hastighet

### Steg 3: Kalibrering

**Vad Chloros gör:**

* **Vignettkorrigering**: Tar bort linsmörkning i kanterna
* **Reflektanskalibrering**: Normaliserar med hjälp av målreflektansvärden
* Tillämpar korrigeringar över alla band/kanaler
* Använder lämpligt kalibreringsmål för varje bild baserat på tidsstämpel

**Varaktighet:** Större delen av bearbetningstiden

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
* Bevarar originalfilnamn med suffix

**Varaktighet:** Varierar beroende på exportformat och filstorlek

***

## Bearbetningsbeteende

### Automatisk bearbetningspipeline

När den har startats körs hela pipelinen automatiskt:

* Ingen användarinteraktion behövs
* Alla konfigurerade steg utförs i sekvens
* Uppdateringar av framsteg visas i realtid

### Datoranvändning under bearbetning

**Fritt läge:**

* Relativt låg CPU-användning (enkelsträngad)
* Datorn förblir responsiv för andra uppgifter
* Det är säkert att minimera Chloros och arbeta i andra applikationer

**Chloros+ Parallellt läge:**

* Hög CPU-användning (multitrådad, upp till 16 kärnor)
* Med GPU-acceleration: Hög GPU-användning
* Datorn kan vara mindre responsiv under bearbetningen
* Undvik att starta andra CPU-intensiva uppgifter

{% hint style=&quot;warning&quot; %}
**Prestandatips**: För bästa prestanda med Chloros+ stänger du andra program och låter Chloros använda alla systemresurser.
{% endhint %}

### Bearbetningen kan inte pausas

**Viktiga begränsningar:**

* När bearbetningen har startat kan den inte pausas.
* Du kan avbryta bearbetningen, men då går framstegen förlorade.
* Delresultat sparas inte.
* Om bearbetningen avbryts måste du börja om från början.

**Planeringstips:** För mycket stora projekt bör du överväga att bearbeta i omgångar eller använda CLI för bättre kontroll.

***

## Övervaka din bearbetning

Medan bearbetningen pågår kan du:

* **Se förloppsindikatorn** – Se den totala procentandelen som är klar.
* **Visa aktuellt steg** – Detektera, analysera, kalibrera eller exportera.
* **Kontrollera fliken Logg** – Se detaljerade bearbetningsmeddelanden och varningar.
* **Förhandsgranska färdiga bilder** – Vissa exportfiler kan visas under bearbetningen.

För detaljerad information om övervakning, se [Övervaka bearbetningen](monitoring-the-processing.md).

***

## Avbryta bearbetningen

Om du behöver avbryta bearbetningen:

### Så här avbryter du

1. Leta reda på **knappen Stopp/Avbryt** (ersätter knappen Start under bearbetningen)
2. Klicka på knappen Stopp
3. Bearbetningen avbryts omedelbart
4. Delresultat kasseras

### När ska man avbryta

**Giltiga skäl att avbryta:**

* Insett att felaktiga inställningar har använts
* Glömt att markera målbilder
* Felaktiga bilder importerade
* Systemet går för långsamt eller svarar inte

**Efter avbrytande:**

* Granska och åtgärda eventuella problem
* Justera inställningarna efter behov
* Starta om bearbetningen från början
* För bästa resultat, stäng Chloros helt och starta om.

{% hint style=&quot;warning&quot; %}
**Inga partiella resultat**: Avbrytande raderar all framsteg. Chloros sparar inte partiellt bearbetade bilder.
{% endhint %}

***

## Uppskattad bearbetningstid

Den faktiska bearbetningstiden varierar kraftigt beroende på:

* Antal bilder
* Bildupplösning
* RAW- eller JPG-inmatningsformat
* Bearbetningsläge (Free eller Chloros+)
* CPU-hastighet och antal kärnor
* GPU-tillgänglighet (endast Chloros+)
* Antal index att beräkna
* Exportformatets komplexitet

### Grova uppskattningar (Chloros+, 12 MP-bilder, modern CPU)

| Antal bilder | Gratis läge | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 bilder   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 bilder  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 bilder  | 1–1,5 tim | 20–30 min      | 10–15 min      |
| 500 bilder  | 2-3 timmar   | 45-60 min      | 20-30 min      |
| 1000 bilder | 4-6 timmar   | 1,5-2 timmar      | 40-60 min      |

{% hint style=&quot;info&quot; %}
**Första körningen**: Den initiala bearbetningen kan ta längre tid eftersom Chloros skapar cacheminnen och profiler. Efterföljande bearbetning av liknande datamängder går snabbare.
{% endhint %}

***

## Vanliga problem vid start

### Startknappen inaktiverad (gråmarkerad)

**Möjliga orsaker:**

* Inga bilder importerade
* Backend inte helt startad
* Tidigare bearbetning fortfarande igång
* Projektet inte helt laddat

**Lösningar:**

1. Vänta tills backend är helt initialiserad (kontrollera huvudmenyikonen)
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

1. Kontrollera felsökningsloggen <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> för felmeddelanden
2. Kontrollera att det finns tillräckligt med diskutrymme
3. Försök att bearbeta en mindre delmängd av bilderna
4. Kontrollera att bilderna inte är skadade

### Varningen &quot;Inga mål upptäckta&quot;

**Möjliga orsaker:**

* Glömt att markera målbilder
* Målbilderna innehåller inga synliga mål
* Inställningarna för måldetektering är för strikta

**Lösningar:**

1. Granska [Välja målbilder](choosing-target-images.md)
2. Markera lämpliga bilder i kolumnen Mål
3. Kontrollera att målen är synliga i de markerade bilderna
4. Justera inställningarna för måldetektering vid behov

***

## Tips för framgångsrik bearbetning

### Innan du börjar

1. **Testa först med en liten delmängd** – Bearbeta 10–20 bilder för att verifiera inställningarna.
2. **Kontrollera tillgängligt diskutrymme** – Se till att det finns 2–3 gånger datamängdens storlek ledigt.
3. **Stäng onödiga program** – Frigör systemresurser.
4. **Verifiera målbilder** – Förhandsgranska markerade mål för att säkerställa kvaliteten.
5. **Spara projektet** – Projektet sparas automatiskt, men det är bra att spara manuellt.

### Under bearbetningen

1. **Undvik att systemet går i viloläge** – Inaktivera energisparlägen.
2. **Håll Chloros i förgrunden** – Eller åtminstone synligt i aktivitetsfältet.
3. **Kontrollera framstegen då och då** – Kontrollera om det finns varningar eller fel.
4. **Ladda inte andra tunga applikationer** – Särskilt med Chloros+ parallellt läge

### Chloros+ GPU-acceleration

Om du använder NVIDIA GPU-acceleration:

1. Uppdatera NVIDIA-drivrutinerna till den senaste versionen
2. Se till att GPU har 4 GB+ VRAM
3. Stäng GPU-intensiva program (spel, videoredigering)
4. Övervaka GPU-temperaturen (se till att kylningen är tillräcklig)

***

## Nästa steg

När bearbetningen har startat:

1. **Övervaka förloppet** – Se [Övervaka bearbetningen](monitoring-the-processing.md)
2. **Vänta tills bearbetningen är klar** – Bearbetningen körs automatiskt.
3. **Granska resultaten** – Se [Avsluta bearbetningen](finishing-the-processing.md).

För information om vad du ska göra under bearbetningen, se [Övervaka bearbetningen](monitoring-the-processing.md).
