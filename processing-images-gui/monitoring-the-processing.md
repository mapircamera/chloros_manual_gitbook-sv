# Övervaka bearbetningen

När bearbetningen har startat erbjuder Chloros

flera sätt att övervaka framstegen, kontrollera om det finns problem och förstå vad som händer med din datamängd. På den här sidan förklaras hur du kan följa bearbetningen och tolka den information som Chloros

tillhandahåller.

## Översikt över förloppsindikatorn

Förloppsindikatorn i den övre rubriken visar bearbetningsstatus i realtid och hur stor andel som är klar. Förloppet strömmas live från backend via Server-Sent Events (SSE), så att indikatorn återspeglar vad bearbetningskedjan faktiskt gör.

### Förloppsindikator i gratisläget

För användare utan Chloros

+-licens:

**2-stegs förloppsvisning:**

1.**Måldetektering** – Att hitta kalibreringsmål i bilder
2. **Bearbetning** – Att tillämpa korrigeringar och exportera**Förloppsindikatorn visar:**

* Total procentuell färdigställandegrad (0–100 %)
* Namn på aktuellt steg
* Enkel horisontell stapelvisualisering

### Chloros

+-förloppsindikator

För användare med Chloros

+-licens:

**4-stegs förloppsvisning:**

1.**Detektering** – Söker efter kalibreringsmål
2. **Analys** – Granskar bilder och förbereder bearbetningskedjan
3. **Kalibrering** – Tillämpa korrigeringar för vinjettering och reflektans
4. **Export** – Spara bearbetade filer**Interaktiva funktioner:*** **Håll muspekaren över** förloppsindikatorn för att se den utökade panelen med fyra steg
* **Klicka på** förloppsindikatorn för att frysa/fästa den utökade panelen
* **Klicka igen** för att låsa upp och dölja automatiskt när muspekaren flyttas bort
* Varje steg visar individuell framstegsgrad (0–100 %)

{% hint style="info" %}
**Paritet med CLI**: under en `chloros-cli process`-körning rapporterar samma fyra trådar som ”Detecting”, ”Analyzing”, ”Processing” och ”Exporting”, och `chloros-cli export-status` visar exportförloppet för tråd 4 i realtid från en annan terminal. Se [CLI

-referensen](../reference/cli-reference.md).
{% endhint %}

***

## Förstå varje bearbetningssteg

{% hint style="info" %}
**Pipelinearkitektur**: Dessa fyra GUI-steg motsvarar [4-trådsbearbetningspipeline](../processing-architecture/processing-pipeline.md). På system med GPU-acceleration drar tråd 3 (Kalibrering) nytta av [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md), som optimerar bearbetningen för just din hårdvara.
{% endhint %}

### Steg 1: Detektering (Måldetektering)

**Vad som händer:**

* Chloros

skannar de bilder du markerat med kryssrutan ”Mål” (alla bilder om ingen är markerad)
* Datorvisionsalgoritmer identifierar kalibreringspanelerna
* Reflektansvärden extraheras från varje panel
* Tidsstämplar för målen registreras för korrekt kalibreringsplanering

**Varaktighet:**

* Med markerade mål: 10–60 sekunder
* Utan markerade mål: 5–30+ minuter (skannar alla bilder)

**Förloppsindikator:**

* Detektering: 0 % → 100 %
* Antal skannade bilder (räknar endast de bilder som faktiskt skannas)
* Antal hittade mål

**Vad du ska hålla utkik efter:**

* Bör slutföras snabbt om målen är korrekt markerade
* Om det tar för lång tid kan det hända att målen inte är markerade
* Kontrollera felsökningsloggen efter meddelanden om ”Mål hittat”

### Steg 2: Analys

**Vad som händer:**

* Läser EXIF-metadata från bilderna (tidsstämplar, exponeringsinställningar)
* Fastställer kalibreringsstrategi baserat på målets tidsstämplar och tillgängliga DAQ-data
* Organiserar bildbehandlingskön
* Förbereder parallella bearbetningsprocesser (Endast Chloros

+)

**Varaktighet:** 5–30 sekunder**Förloppsindikator:**

* Analyserar: 0 % → 100 %
* Snabbt steg, avslutas vanligtvis snabbt

**Vad man ska hålla utkik efter:**

* Förloppet bör ske jämnt utan avbrott
* Varningar om saknade metadata visas i felsökningsloggen

### Steg 3: Kalibrering

**Vad som händer:*** **Debayering**: Konvertering av RAW-Bayer-mönster till 3 kanaler (hoppas över för LATTICE-monomoduler, med en anmärkning)
* **Vignettkorrigering**: Avlägsnar mörkningen vid objektivets kanter
* **Reflektanskalibrering**: Normalisering med målvärden och/eller DAQ-nedviktning
* **Indexberäkning**: Beräkning av multispektrala index
* Bearbetning av varje bild genom hela bearbetningskedjan

**Varaktighet:** Största delen av den totala bearbetningstiden (60–80 %)**Framstegsindikator:**

* Kalibrerar: 0 % → 100 %
* Bild som bearbetas just nu
* Bearbetade bilder / Totalt antal bilder

**Bearbetningsbeteende:*** **Fritt läge**: Bearbetar en bild i taget sekventiellt
* **Läge ”Chloros

+”**: Kör en hårdvaruanpassad arbetspool — 1–4 samtidiga arbetsprocesser på GPU-system (beroende på VRAM), en arbetsprocess per fysisk kärna (minus en) på system med enbart CPU. Se [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-acceleration**: Påskyndar detta steg avsevärt**Vad du bör hålla koll på:**

* Jämn framsteg genom bildräkningen
* Kontrollera felsökningsloggen för meddelanden om att varje bild är klar
* Varningar om bildkvalitet eller kalibreringsproblem

### Steg 4: Export

**Vad som händer:**

* Skriver bearbetade bilder till disk i det valda formatet allteftersom de blir färdiga
* **LATTICE**: varje bildruta fördelas till alla aktiverade produkter (debayered / förhandsgranskning / radiance / reflektans)
* Exportering av multispektrala indexbilder med LUT-färger
* Skapande av utdataträdet `<project>/<camera>/<format>/<Product>_Images/` — exporterade filer behåller källfilnamnet; mappen identifierar produkten

**Varaktighet:** 10–20 % av den totala bearbetningstiden**Framstegsindikator:**

* Exporterar: 0 % → 100 %
* Filer skrivs
* Exportformat och mål

**Vad du ska hålla utkik efter:**

* Varningar om diskutrymme
* Fel vid filskrivning
* När alla konfigurerade utdata är färdiga

***

## Fliken Debug Log

Debug Log ger detaljerad information om bearbetningens förlopp och eventuella problem som uppstått. Startmeddelanden från backend återges också i loggkonsolen, så loggen ger en fullständig bild även om du öppnar den i efterhand.

### Öppna felsökningsloggen

1. Klicka på ikonen **Felsökningslogg** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

i vänster sidofält
2. Loggpanelen öppnas och visar bearbetningsmeddelanden i realtid
3. Rullar automatiskt för att visa de senaste meddelandena

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Förstå loggmeddelanden Loggraderna i

Chloros

har ett prefix med taggar inom parentes som anger delsystemet – till exempel `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Det viktigaste att känna till är **körningssammanfattningen**, som visas i slutet av varje körning (inklusive avbrutna körningar):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Extra `[RUN-SUMMARY]`-rader med förklaringar följer när något behöver förklaras — till exempel en körning som inte gav något resultat, eller en kamera vars begärda produkt hoppades över eftersom den inte var tillämplig. `[EXPORT-CHECK]`-rader förklarar utelämnanden per kamera (t.ex. varför en RGB

-kamera inte fick någon strålningsprodukt).

De allmänna allvarlighetsgraderna för meddelanden (exemplen nedan är illustrativa, inte ordagrant återgivna):

#### Informationsmeddelanden (vita/grå)

Normala uppdateringar om bearbetningen: bearbetning påbörjad, mål upptäckta (med antal paneler), kalibreringsförlopp per bild, filer exporterade, bearbetning slutförd.

#### Varningsmeddelanden (gula)

Icke-kritiska problem som inte avbryter bearbetningen – t.ex. saknade GPS-data i en bildruta, ett stort tidsmässigt gap mellan målbilder eller låg kontrast i en kalibreringspanel.

**Åtgärd:** Granska varningarna efter bearbetningen, men avbryt inte

#### Felmeddelanden (rRed

)

Kritiska problem som kan leda till att bearbetningen misslyckas – t.ex. full disk, en skadad bildfil eller inga mål upptäckta när reflektanskalibrering begärdes.

**Åtgärd:** Avbryt bearbetningen, åtgärda felet, starta om

### Vanliga loggsituationer

| Situation                             | Betydelse                                       | Åtgärd som krävs                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Mål upptäckt i \[filnamn]        | Kalibreringsmål hittat utan problem         | Ingen – normalt                                         |
| Förloppslinjer per bild              | Aktuell förloppsuppdatering                       | Ingen – normalt                                         |
| Inga mål hittades                      | Inga kalibreringsmål upptäckta               | Markera målbilder eller inaktivera reflektanskalibrering |
| Otillräckligt diskutrymme               | Inte tillräckligt med lagringsutrymme för utdata                 | Frigör diskutrymme                                    |
| Hoppar över skadad fil               | Bildfilen är skadad                         | Kopiera om filen från SD-kortet                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | En bildtagning utan en råbild kan inte bearbetas | Ta om bilden med råbild, eller använd CLI

`--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Körningen gav inga bildprodukter — rapporterades som ett fel med tips | Läs tipsen; kontrollera vad som hoppades över och varför |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Windows

-långa sökvägar är avstängda och projektmappen ligger för djupt för gränsen på 260 tecken; ingenting bearbetades | Använd en kortare projektmapp (meddelandet anger den längsta som ryms), eller aktivera långa sökvägar i Windows

och starta om Chloros

— se [Starta bearbetningen](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Det gick inte att skriva en klämmask för en LATTICE-produkt; själva produkten är okej och visar `Chloros:UnmixClampMask=unavailable` | Kontrollera diskutrymme och mappbehörigheter, och kör sedan bearbetningen igen om du behöver masken |

### Kopiera loggdata

Så här kopierar du loggen för felsökning eller support:

1. Öppna panelen Debug Log
2. Klicka på knappen **Kopiera**(kopierar loggen till urklipp) eller**Ladda ner** för att spara den som en textfil
3. Klistra in i en textfil eller i ett e-postmeddelande
4. Skicka till MAPIR

-supporten vid behov

### Rapportera ett fel från fliken *Debug Log*

Nytt i 1.3.0: felikonen (**Rapportera ett fel**) högst upp på fliken *Debug Log* skickar en rapport direkt till MAPIR

.

1. Klicka på felikonen högst upp på fliken *Debug Log*
2. Beskriv problemet: vad du gjorde, vad du förväntade dig och vad som istället hände (obligatoriskt)
3. Markera eventuellt **Lägg till några bilder (upp till 10)**och välj bilder från det öppna projektet, eller använd**Bläddra...** för andra bildfiler
4. Klicka på **Skicka**

Rapporten innehåller din beskrivning, eventuella bilder du bifogar, Chloros

-loggarna och systeminformation (Chloros

-version, operativsystem, GPU och projektinställningar). Du måste vara inloggad på Chloros

(fliken ”Användare”) för att kunna skicka en rapport. Du kan stänga fönstret medan rapporten skickas; när överföringen är klar skrivs en referens, till exempel `Bug report sent. Reference: BR-…`, in i loggen – ange den när du kontaktar MAPIR

. Om rapporten inte kan laddas upp (till exempel på grund av avsaknad av internetanslutning) sparar Chloros

den på din dator och visar sökvägen så att du kan skicka den via e-post till support@mapir.camera.

***

## Övervakning av systemresurser

### CPU-användning

**Fritt läge:**

* 1 CPU-kärna på ~100 %
* Övriga kärnor är inaktiva eller tillgängliga
* Systemet förblir responsivt

**Parallellt läge i Chloros

+:**

* Flera kärnor med hög utnyttjandegrad – antalet beror på den strategi som valts av [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Systemet kan kännas mindre responsivt

**Så här övervakar du:**

* Windows

Aktivitetshanteraren (Ctrl+Shift+Esc)
* Fliken Prestanda → avsnittet CPU
* Leta efter processerna ”Chloros

” eller ”chloros-backend”

### Minnesanvändning (RAM)

**Typisk användning:**

* Små projekt (&lt; 100 bilder): 2–4 GB
* Medelstora projekt (100–500 bilder): 4–8 GB
* Stora projekt (500+ bilder): 8–16 GB
* Chloros

i parallellt läge använder mer RAM

**Om minnet är lågt:**

* Bearbeta mindre batcher
* Stäng andra program
* Uppgradera RAM-minnet om du regelbundet bearbetar stora datamängder

### GPU-användning (”Chloros

+” med CUDA)

När GPU-acceleration är aktiverad:

* NVIDIA-GPU:n visar hög utnyttjandegrad (60–90 %)
* VRAM-användningen ökar (kräver minst 4 GB VRAM; minst 7 GB för samtidig Texture Aware-debayering)
* Kalibreringsfasen går betydligt snabbare

**Att övervaka:**

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
* Kontrollera Aktivitetshanteraren för att se till att Chloros

körs

**Felmeddelanden visas ofta:**

* Avbryt bearbetningen och granska felen
* Vanliga orsaker: diskutrymme, skadade filer, minnesproblem
* Se avsnittet Felsökning nedan

**Systemet svarar inte:**

* Chloros

i parallellt läge använder för mycket resurser
* Överväg att minska antalet samtidiga uppgifter eller uppgradera hårdvaran
* Fritt läge är mindre resurskrävande

### När du ska avbryta bearbetningen

Avbryt bearbetningen om du ser:

* ❌ Felmeddelanden som ”Disk full” eller ”Kan inte skriva fil”
* ❌ Upprepade fel på grund av skadade bildfiler
* ❌ Systemet har fryst helt (svarar inte)
* ❌ Du har insett att felaktiga inställningar har konfigurerats
* ❌ Felaktiga bilder har importerats

**Så här avbryter du:**

1. Klicka på**Stopp-knappen** (ersätter Start-knappen) – en gång räcker
2. Indikatorstapeln visar ”Avbryter...” medan den pågående bilden avslutas, därefter avslutas körningen i ett avbrutet tillstånd
3. Produkter som redan har exporterats finns kvar på disken; loggen visar en tydlig `[RUN-SUMMARY]`-rapport över vad som har slutförts
4. Åtgärda problemen och starta om — körningen börjar från början

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

1. Om bearbetningen just har startat och befinner sig i detekteringsfasen: Avbryt, markera mål, starta om
2. För framtiden: Använd SSD, minska antalet index, uppgradera hårdvaran
3. Överväg att använda CLI

för batchbearbetning av stora datamängder

### Varningar om ”diskutrymme”

**Lösningar:**

1. Frigör diskutrymme omedelbart
2. Flytta projektet till en enhet med mer utrymme
3. Minska antalet index som ska exporteras
4. Inaktivera LATTICE-exportprodukter som du inte behöver (Projektinställningar → Bearbetning)
5. Använd JPG-format istället för TIFF

-format (mindre filer)

### Återkommande meddelanden om ”skadade filer”

**Lösningar:**

1. Kopiera om bilderna från SD-kortet för att säkerställa integriteten
2. Testa SD-kortet för fel
3. Ta bort skadade filer från projektet
4. Fortsätt bearbeta återstående bilder

### Systemöverhettning / prestandabegränsning

**Lösningar:**

1. Se till att ventilationen är tillräcklig
2. Rengör datorns ventilationsöppningar från damm
3. Minska bearbetningsbelastningen (använd Free-läget istället för Chloros

+)
4. Bearbeta under svalare tider på dygnet

***

## Meddelande om avslutad bearbetning

När bearbetningen är klar:

* Förloppsindikatorn når 100 %
* Raderna `[RUN-SUMMARY]` visas i felsökningsloggen med slutliga siffror
* Startknappen blir återigen aktiv
* Alla utdatafiler finns i projektets utdatastruktur per kamera: `<project>/<camera>/<format>/<Product>_Images/`

***

## Nästa steg

När bearbetningen är klar:

1. **Granska resultaten** – Se [Avsluta bearbetningen](finishing-the-processing.md)
2. **Kontrollera utdatamappen** – Kontrollera att alla filer har exporterats korrekt
3. **Granska felsökningsloggen** – Kontrollera om det finns några varningar eller fel
4. **Förhandsgranska de bearbetade bilderna** – Använd Bildvisaren eller extern programvara

För information om hur du granskar och använder dina bearbetade resultat, se [Avsluta bearbetningen](finishing-the-processing.md).
