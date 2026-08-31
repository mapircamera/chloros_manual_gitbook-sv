# Övervakning av bearbetningen

När bearbetningen har startat erbjuder Chloros flera sätt att övervaka framstegen, kontrollera om det finns problem och förstå vad som händer med din datamängd. På den här sidan förklaras hur du kan följa bearbetningen och tolka den information som Chloros tillhandahåller.

## Översikt över förloppsindikatorn

Förloppsindikatorn i den övre rubriken visar bearbetningsstatus i realtid och hur stor andel som är klar. Förloppet strömmas live från backend via Server-Sent Events (SSE), så indikatorn återspeglar vad bearbetningskedjan faktiskt gör.

### Förloppsindikator i gratisläget

För användare utan Chloros+-licens:

**Visning av framsteg i två steg:**

1.**Måldetektering** – Att hitta kalibreringsmål i bilder
2. **Bearbetning** – Att tillämpa korrigeringar och exportera**Framstegsindikatorn visar:**

* Total procentuell färdigställandegrad (0–100 %)
* Namn på aktuellt steg
* Enkel horisontell stapelvisualisering

### Chloros+-förloppsindikator

För användare med Chloros+-licens:

**4-stegs förloppsvisning:**

1.**Detektering** – Att hitta kalibreringsmål
2. **Analys** – Att granska bilder och förbereda bearbetningsflödet
3. **Kalibrering** – Att tillämpa korrigeringar för vinjettering och reflektans
4. **Export** – Att spara bearbetade filer**Interaktiva funktioner:*** **Håll muspekaren över** förloppsindikatorn för att se den utökade panelen med fyra steg
* **Klicka på** förloppsindikatorn för att frysa/fästa den utökade panelen
* **Klicka igen** för att låsa upp den och dölja den automatiskt när muspekaren flyttas bort
* Varje steg visar individuell framstegsprocent (0–100 %)

{% hint style="info" %}
**CLI-paritet**: under en `chloros-cli process`-körning rapporterar samma fyra trådar att de är i fas med ”Detecting”, ”Analyserar”, ”Bearbetar”, ”Exporterar”, och `chloros-cli export-status` visar realtidsförloppet för tråd 4:s export från en annan terminal. Se [CLI-referensen](../reference/cli-reference.md).
{% endhint %}

***

## Förstå varje bearbetningssteg

{% hint style="info" %}
**Pipelinearkitektur**: Dessa fyra GUI-steg motsvarar [4-tråds bearbetningspipeline](../processing-architecture/processing-pipeline.md). På system med GPU-acceleration drar tråd 3 (Kalibrering) nytta av [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md) som optimerar bearbetningen för just din hårdvara.
{% endhint %}

### Steg 1: Detektering (måldetektering)

**Vad som händer:**

* Chloros skannar de bilder du markerat med kryssrutan ”Mål” (alla bilder om ingen är markerad)
* Datorvisionsalgoritmer identifierar kalibreringspanelerna
* Reflektansvärden extraheras från varje panel
* Tidsstämplar för målen registreras för korrekt kalibreringsplanering

**Varaktighet:**

* Med markerade mål: 10–60 sekunder
* Utan markerade mål: 5–30+ minuter (skannar alla bilder)

**Framstegsindikator:**

* Detektering: 0 % → 100 %
* Antal skannade bilder (räknar endast de bilder som faktiskt skannas)
* Antal hittade mål

**Vad du bör hålla koll på:**

* Bör slutföras snabbt om målen är korrekt markerade
* Om det tar för lång tid kan det hända att målen inte är markerade
* Kontrollera felsökningsloggen för meddelanden om ”Mål hittat”

### Steg 2: Analys

**Vad som händer:**

* Läser bildens EXIF-metadata (tidsstämplar, exponeringsinställningar)
* Fastställer kalibreringsstrategi baserat på målets tidsstämplar och tillgängliga DAQ-data
* Organiserar bildbehandlingskön
* Förbereder parallella bearbetningsprocesser (endast Chloros+)

**Varaktighet:** 5–30 sekunder**Framstegsindikator:**

* Analyserar: 0 % → 100 %
* Snabbt steg, avslutas vanligtvis snabbt

**Vad man ska hålla utkik efter:**

* Bör fortskrida stadigt utan avbrott
* Varningar om saknade metadata visas i felsökningsloggen

### Steg 3: Kalibrering

**Vad som händer:*** **Debayering**: Konvertering av RAW-Bayer-mönster till 3 kanaler (hoppas över för LATTICE-monomoduler, med en anmärkning)
* **Vignettkorrigering**: Borttagning av mörkare kanter vid objektivets ytterkanter
* **Reflektanskalibrering**: Normalisering med målvärden och/eller DAQ-nedviktning
* **Indexberäkning**: Beräkning av multispektrala index
* Bearbetning av varje bild genom hela bearbetningskedjan

**Varaktighet:** Merparten av den totala bearbetningstiden (60–80 %)**Framstegsindikator:**

* Kalibrering: 0 % → 100 %
* Bild som bearbetas just nu
* Bearbetade bilder / Totalt antal bilder

**Bearbetningsbeteende:*** **Fritt läge**: Bearbetar en bild i taget sekventiellt
* **Chloros+-läge**: Kör en hårdvaruanpassad arbetspool – 1–4 samtidiga arbetsprocesser på GPU-system (beroende på VRAM), en arbetsprocess per fysisk kärna (minus en) på system med enbart CPU. Se [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-acceleration**: Påskyndar detta steg avsevärt**Vad du ska hålla utkik efter:**

* Jämn framsteg genom bildantalet
* Kontrollera felsökningsloggen för meddelanden om slutförda bilder
* Varningar om bildkvalitet eller kalibreringsproblem

### Steg 4: Export

**Vad som händer:**

* Skriver bearbetade bilder till disk i det valda formatet allteftersom de blir färdiga
* **LATTICE**: varje bildruta fördelas till alla aktiverade produkter (debayering / förhandsgranskning / strålning / reflektans)
* Exportering av multispektrala indexbilder med LUT-färger
* Skapande av utdataträdet `<project>/<camera>/<format>/<Product>_Images/` — exporterade filer behåller källfilnamnet; mappen identifierar produkten

**Varaktighet:** 10–20 % av den totala bearbetningstiden**Förloppsindikator:**

* Exporterar: 0 % → 100 %
* Filer skrivs
* Exportformat och målplats

**Vad du bör hålla utkik efter:**

* Varningar om diskutrymme
* Fel vid filskrivning
* Att alla konfigurerade utdata är färdiga

***

## Fliken Debug Log

Debug Log ger detaljerad information om bearbetningens framsteg och eventuella problem som uppstått. Startmeddelanden från backend återges också i loggkonsolen, så loggen ger en fullständig bild även om du öppnar den senare.

### Så här öppnar du felsökningsloggen

1. Klicka på ikonen **Felsökningslogg**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

i vänster sidofält
2. Loggpanelen öppnas och visar bearbetningsmeddelanden i realtid
3. Rullar automatiskt för att visa de senaste meddelandena

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Förstå loggmeddelanden

Chloros-loggrader inleds med taggar inom parentes som anger delsystemet – till exempel `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Det viktigaste att känna till är **körningssammanfattningen**, som visas i slutet av varje körning (inklusive avbrutna körningar):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Extra `[RUN-SUMMARY]`-rader med förklaringar följer när något behöver förklaras — till exempel en körning som inte gav något resultat, eller en kamera vars begärda produkt hoppades över eftersom den inte var tillämplig. `[EXPORT-CHECK]`-rader förklarar utelämnanden per kamera (t.ex. varför en RGB-kamera inte fick någon strålningsprodukt).

De allmänna allvarlighetsgraderna för meddelanden (exemplen nedan är illustrativa, inte ordagrant återgivna):

#### Informationsmeddelanden (vita/grå)

Normala uppdateringar om bearbetningen: bearbetning påbörjad, mål upptäckta (med antal paneler), kalibreringsförlopp per bild, filer exporterade, bearbetning slutförd.

#### Varningsmeddelanden (gult)

Icke-kritiska problem som inte stoppar bearbetningen – t.ex. saknade GPS-data i en bildram, ett stort tidsmässigt glapp mellan målbilderna eller låg kontrast i en kalibreringspanel.

**Åtgärd:** Granska varningarna efter bearbetningen, men avbryt inte processen

#### Felmeddelanden (Red)

Kritiska problem som kan leda till att bearbetningen misslyckas – t.ex. full disk, en skadad bildfil eller inga mål upptäckta när reflektanskalibrering begärdes.

**Åtgärd:** Avbryt bearbetningen, åtgärda felet och starta om

### Vanliga loggsituationer

| Situation                             | Betydelse                                       | Nödvändig åtgärd                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Mål upptäckt i \[filnamn]        | Kalibreringsmål hittat utan problem         | Ingen – normalt                                         |
| Förloppslinjer per bild              | Aktuell uppdatering av förloppet                       | Ingen – normalt                                         |
| Inga mål hittade                      | Inga kalibreringsmål upptäckta               | Markera målbilder eller inaktivera reflektanskalibrering |
| Otillräckligt diskutrymme               | Inte tillräckligt med lagringsutrymme för utdata                 | Frigör diskutrymme                                    |
| Hoppar över skadad fil               | Bildfilen är skadad                         | Kopiera om filen från SD-kortet                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | En bildtagning utan en råbild kan inte bearbetas | Ta en ny bild med råbild, eller använd CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Körningen genererade inga bildprodukter — rapporterades som ett fel med tips | Läs tipsen; kontrollera vad som hoppades över och varför |

### Kopiera loggdata

Så här kopierar du loggen för felsökning eller support:

1. Öppna panelen **Felsökningslogg**

2. Klicka på knappen**&quot;Kopiera logg&quot;** (eller högerklicka → Markera allt)
3. Klistra in i en textfil eller i ett e-postmeddelande
4. Skicka till MAPIR-supporten vid behov

***

## Övervakning av systemresurser

### CPU-användning

**Fritt läge:**

* 1 CPU-kärna på ~100 %
* Övriga kärnor är inaktiva eller tillgängliga
* Systemet förblir responsivt

**Chloros+ Parallellläge:**

* Flera kärnor med hög utnyttjandegrad — antalet beror på den strategi som valts av [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Systemet kan upplevas som mindre responsivt

**För övervakning:**

* Windows Aktivitetshanteraren (Ctrl+Shift+Esc)
* Fliken Prestanda → avsnittet CPU
* Leta efter processerna ”Chloros” eller ”chloros-backend”

### Minnesanvändning (RAM)

**Typisk användning:**

* Små projekt (&lt; 100 bilder): 2–4 GB
* Medelstora projekt (100–500 bilder): 4–8 GB
* Stora projekt (500+ bilder): 8–16 GB
* Chloros+ i parallellläge använder mer RAM

**Om minnet är lågt:**

* Bearbeta mindre batcher
* Stäng andra program
* Uppgradera RAM-minnet om du regelbundet bearbetar stora datamängder

### GPU-användning (Chloros+ med CUDA)

När GPU-acceleration är aktiverad:

* NVIDIA-GPU:n visar hög utnyttjandegrad (60–90 %)
* VRAM-användningen ökar (kräver minst 4 GB VRAM; minst 7 GB för samtidig Texture Aware-debayering)
* Kalibreringsfasen går betydligt snabbare

**För att övervaka:**

* NVIDIA-ikonen i systemfältet
* Aktivitetshanteraren → Prestanda → GPU
* GPU-Z eller liknande övervakningsverktyg

### Disk-I/O

**Vad du kan förvänta dig:**

* Hög diskavläsning under analysfasen
* Hög diskskrivning under exportfasen
* SSD är betydligt snabbare än HDD

**Prestandatips:**

* Använd SSD för projektmappen när det är möjligt
* Undvik nätverksenheter för stora datamängder
* Se till att disken inte är nära full (påverkar skrivhastigheten)

***

## Upptäcka problem under bearbetningen

### Varningssignaler

**Framstegen avstannar (ingen förändring på 5+ minuter):**

* Kontrollera felsökningsloggen för eventuella fel
* Kontrollera att det finns ledigt diskutrymme
* Kontrollera Aktivitetshanteraren för att säkerställa att Chloros körs

**Felmeddelanden visas ofta:**

* Avbryt bearbetningen och granska felen
* Vanliga orsaker: diskutrymme, skadade filer, minnesproblem
* Se avsnittet Felsökning nedan

**Systemet svarar inte:**

* Chloros+ i parallellt läge använder för mycket resurser
* Överväg att minska antalet samtidiga uppgifter eller uppgradera hårdvaran
* Fritt läge är mindre resurskrävande

### När du ska avbryta bearbetningen

Avbryt bearbetningen om du ser:

* ❌ Felmeddelanden som ”Disken full” eller ”Kan inte skriva fil”
* ❌ Upprepade fel på grund av skadade bildfiler
* ❌ Systemet har fryst helt (svarar inte)
* ❌ Upptäckt att felaktiga inställningar har konfigurerats
* ❌ Felaktiga bilder har importerats

**Så här avbryter du:**

1. Klicka på**knappen Stopp** (ersätter Start-knappen) – en gång räcker
2. Indikatorstapeln visar ”Avslutar...” medan den pågående bilden bearbetas klart, därefter avslutas körningen i ett avbrutet tillstånd
3. Produkter som redan har exporterats finns kvar på disken; loggen visar en detaljerad `[RUN-SUMMARY]`-rapport över vad som har slutförts
4. Åtgärda problemen och starta om – körningen börjar från början

***

## Felsökning under bearbetning

### Bearbetningen går mycket långsamt

**Möjliga orsaker:**

* Omärkta målbilder (alla bilder skannas)
* Lagring på HDD istället för SSD
* Otillräckliga systemresurser
* Många index konfigurerade
* Åtkomst via nätverksenhet

**Lösningar:**

1. Om processen just har startat och befinner sig i detekteringsfasen: Avbryt, markera mål, starta om
2. För framtiden: Använd SSD, minska antalet index, uppgradera hårdvaran
3. Överväg CLI för batchbearbetning av stora datamängder

### Varningar om ”diskutrymme”

**Lösningar:**

1. Frigör diskutrymme omedelbart
2. Flytta projektet till en enhet med mer utrymme
3. Minska antalet index som ska exporteras
4. Inaktivera LATTICE-exportprodukter som du inte behöver (Projektinställningar → Bearbetning)
5. Använd JPG-format istället för TIFF (mindre filer)

### Återkommande meddelanden om ”skadade filer”

**Lösningar:**

1. Kopiera om bilderna från SD-kortet för att säkerställa att de är oskadda
2. Kontrollera SD-kortet för fel
3. Ta bort skadade filer från projektet
4. Fortsätt bearbeta återstående bilder

### Systemöverhettning / prestandabegränsning

**Lösningar:**

1. Se till att ventilationen är tillräcklig
2. Rengör datorns ventilationsöppningar från damm
3. Minska bearbetningsbelastningen (använd Free-läget istället för Chloros+)
4. Bearbeta under svalare tider på dygnet

***

## Meddelande om avslutad bearbetning

När bearbetningen är klar:

* Visas förloppsindikatorn 100 %
* Visas raderna `[RUN-SUMMARY]` i felsökningsloggen med slutliga siffror
* Aktiveras Start-knappen igen
* Finns alla utdatafiler i projektets utdatastruktur per kamera: `<project>/<camera>/<format>/<Product>_Images/`

***

## Nästa steg

När bearbetningen är klar:

1. **Granska resultaten** – Se [Avsluta bearbetningen](finishing-the-processing.md)
2. **Kontrollera utdatamappen** – Kontrollera att alla filer har exporterats korrekt
3. **Granska felsökningsloggen** – Kontrollera om det finns några varningar eller fel
4. **Förhandsgranska de bearbetade bilderna** – Använd Bildvisaren eller extern programvara

För information om hur du granskar och använder dina bearbetade resultat, se [Avsluta bearbetningen](finishing-the-processing.md).
