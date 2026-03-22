# Övervaka bearbetningen

När bearbetningen har startat erbjuder Chloros flera sätt att övervaka förloppet, kontrollera om det finns problem och förstå vad som händer med din dataset. På den här sidan förklaras hur du följer bearbetningen och tolkar den information som Chloros tillhandahåller.

## Översikt över förloppsindikatorn

Förloppsindikatorn i den övre rubriken visar bearbetningsstatus i realtid och procentuell färdigställandegrad.

### Förloppsindikator i fritt läge

För användare utan Chloros+-licens:

**2-stegs visning av förlopp:**

1.**Målidentifiering** – Att hitta kalibreringsmål i bilder
2. **Bearbetning** – Tillämpa korrigeringar och exportera**Förloppsindikatorn visar:**

* Total procentuell färdigställandegrad (0–100 %)
* Namn på aktuellt steg
* Enkel horisontell stapelvisualisering

### Chloros+ förloppsindikator

För användare med Chloros+-licens:

**4-stegs förloppsvisning:**

1.**Detektering** – Hitta kalibreringsmål
2. **Analys** – Granska bilder och förbereda pipeline
3. **Kalibrering** – Tillämpa korrigeringar för vinjettering och reflektans
4. **Export** – Spara bearbetade filer**Interaktiva funktioner:*** **Håll muspekaren över** förloppsindikatorn för att se den utökade panelen med 4 steg
* **Klicka på** förloppsindikatorn för att frysa/fästa den utökade panelen
* **Klicka igen** för att låsa upp och automatiskt dölja när muspekaren flyttas bort
* Varje steg visar individuell framsteg (0–100 %)

***

## Förstå varje bearbetningssteg

{% hint style="info" %}
**Pipeline-arkitektur**: Dessa fyra GUI-steg motsvarar [4-trådsbehandlingspipeline](../processing-architecture/processing-pipeline.md). På system med GPU-acceleration drar tråd 3 (Kalibrering) nytta av [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) som optimerar bearbetningen för din specifika hårdvara.
{% endhint %}

### Steg 1: Detektering (måldetektering)

**Vad händer:**

* Chloros skannar bilder som är markerade med kryssrutan Mål
* Datorvisionsalgoritmer identifierar de fyra kalibreringspanelerna
* Reflektansvärden extraheras från varje panel
* Målens tidsstämplar registreras för korrekt kalibreringsplanering

**Varaktighet:**

* Med markerade mål: 10–60 sekunder
* Utan markerade mål: 5–30+ minuter (skannar alla bilder)

**Framstegsindikator:**

* Detektering: 0 % → 100 %
* Antal skannade bilder
* Antal hittade mål

**Vad du ska hålla utkik efter:**

* Bör slutföras snabbt om målen är korrekt markerade
* Om det tar för lång tid kan det hända att målen inte är markerade
* Kontrollera felsökningsloggen för meddelanden om ”Mål hittat”

### Steg 2: Analys

**Vad som händer:**

* Läser bildens EXIF-metadata (tidsstämplar, exponeringsinställningar)
* Fastställer kalibreringsstrategi baserat på målets tidsstämplar
* Organiserar bildbehandlingskön
* Förbereder parallella bearbetningsprocesser (endast Chloros+)

**Varaktighet:** 5–30 sekunder**Förloppsindikator:**

* Analyserar: 0 % → 100 %
* Snabb fas, slutförs vanligtvis snabbt

**Vad du ska hålla utkik efter:**

* Bör fortskrida stadigt utan pauser
* Varningar om saknade metadata visas i felsökningsloggen

### Steg 3: Kalibrering

**Vad som händer:*** **Debayering**: Konvertering av RAW-Bayer-mönster till 3 kanaler
* **Vignettkorrigering**: Tar bort mörkare kanter på linsen
* **Reflektanskalibrering**: Normaliserar med målvärden
* **Indexberäkning**: Beräknar multispektrala index
* Bearbetar varje bild genom hela processen

**Varaktighet:** Största delen av den totala bearbetningstiden (60–80 %)**Framstegsindikator:**

* Kalibrering: 0 % → 100 %
* Bild som bearbetas just nu
* Färdigbehandlade bilder / Totalt antal bilder

**Bearbetningsbeteende:*** **Fritt läge**: Bearbetar en bild i taget i sekventiell ordning
* **Chloros+ läge**: Bearbetar upp till 16 bilder samtidigt
* **GPU-acceleration**: Påskyndar detta steg avsevärt**Vad du ska hålla utkik efter:**

* Jämn framsteg genom bildräkningen
* Kontrollera felsökningsloggen för meddelanden om slutförande per bild
* Varningar om bildkvalitet eller kalibreringsproblem

### Steg 4: Exportera

**Vad som händer:**

* Skriver kalibrerade bilder till disk i valt format
* Exporterar multispektrala indexbilder med LUT-färger
* Skapar undermappar för kameramodeller
* Bevarar ursprungliga filnamn med lämpliga suffix

**Varaktighet:** 10–20 % av den totala bearbetningstiden**Framstegsindikator:**

* Exportering: 0 % → 100 %
* Filer som skrivs
* Exportformat och destination

**Vad du ska hålla koll på:**

* Varningar om diskutrymme
* Fel vid filskrivning
* Slutförande av alla konfigurerade utdata

***

## Fliken Debug Log

Debug Log ger detaljerad information om bearbetningens framsteg och eventuella problem som uppstår.

### Öppna Debug Log

1. Klicka på **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> i vänster sidomeny
2. Loggpanelen öppnas och visar bearbetningsmeddelanden i realtid
3. Bläddrar automatiskt för att visa de senaste meddelandena

### Förstå loggmeddelanden

#### Informationsmeddelanden (vita/grå)

Normala bearbetningsuppdateringar:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Varningsmeddelanden (gula)

Icke-kritiska problem som inte stoppar bearbetningen:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Åtgärd:** Granska varningarna efter bearbetningen, men avbryt inte

#### Felmeddelanden (Red)

Kritiska problem som kan orsaka att bearbetningen misslyckas:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Åtgärd:** Avbryt bearbetningen, åtgärda felet, starta om

### Vanliga loggmeddelanden

| Meddelande                          | Betydelse                                | Åtgärd som krävs                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Mål upptäckt i \[filnamn]&quot; | Kalibreringsmål hittat  | Ingen – normalt                                         |
| &quot;Bearbetar bild X av Y&quot;        | Aktuell uppdatering av förloppet                | Ingen – normalt                                         |
| &quot;Inga mål hittade&quot;               | Inga kalibreringsmål upptäckta        | Markera målbilder eller inaktivera reflektanskalibrering |
| &quot;Otillräckligt diskutrymme&quot;        | Otillräckligt lagringsutrymme för utdata          | Frigör diskutrymme                                    |
| &quot;Hoppar över skadad fil&quot;        | Bildfilen är skadad                  | Kopiera om filen från SD-kortet                             |
| &quot;PPK-data tillämpad&quot;               | GPS-korrigeringar från .daq-fil tillämpade | Ingen - normalt                                         |

### Kopiera loggdata

Så här kopierar du loggen för felsökning eller support:

1. Öppna panelen Debug Log
2. Klicka på knappen **&quot;Copy Log&quot;** (eller högerklicka → Välj allt)
3. Klistra in i en textfil eller e-post
4. Skicka till MAPIR support vid behov

***

## Övervakning av systemresurser

### CPU-användning

**Fritt läge:**

* 1 CPU-kärna på ~100 %
* Övriga kärnor är inaktiva eller tillgängliga
* Systemet förblir responsivt

**Chloros+ Parallellt läge:**

* Flera kärnor på 80–100 % (upp till 16 kärnor)
* Hög total CPU-användning
* Systemet kan kännas mindre responsivt

**För att övervaka:**

* Windows Aktivitetshanteraren (Ctrl+Shift+Esc)
* Fliken Prestanda → Avsnittet CPU
* Leta efter processerna &quot;Chloros&quot; eller &quot;chloros-backend&quot;

### Minne (RAM)

**Typisk användning:**

* Små projekt (&lt; 100 bilder): 2–4 GB
* Medelstora projekt (100–500 bilder): 4–8 GB
* Stora projekt (500+ bilder): 8–16 GB
* Chloros+ parallellt läge använder mer RAM

**Om minnet är lågt:**

* Bearbeta mindre batcher
* Stäng andra program
* Uppgradera RAM-minnet om du regelbundet bearbetar stora datamängder

### GPU-användning (Chloros+ med CUDA)

När GPU-acceleration är aktiverad:

* NVIDIA GPU visar hög utnyttjandegrad (60–90 %)
* VRAM-användningen ökar (kräver 4 GB+ VRAM)
* Kalibreringsfasen går betydligt snabbare

**Att övervaka:**

* NVIDIA-ikonen i systemfältet
* Aktivitetshanteraren → Prestanda → GPU
* GPU-Z eller liknande övervakningsverktyg

### Disk-I/O

**Vad du kan förvänta dig:**

* Hög diskläsning under analysfasen
* Hög diskskrivning under exportfasen
* SSD är betydligt snabbare än HDD

**Prestandatips:**

* Använd SSD för projektmappen när det är möjligt
* Undvik nätverksenheter för stora datamängder
* Se till att disken inte är nära full (påverkar skrivhastigheten)

***

## Upptäcka problem under bearbetning

### Varningssignaler

**Framstegen stannar upp (ingen förändring på 5+ minuter):**

* Kontrollera felsökningsloggen för fel
* Kontrollera tillgängligt diskutrymme
* Kontrollera Aktivitetshanteraren för att säkerställa att Chloros körs

**Felmeddelanden visas ofta:**

* Avbryt bearbetningen och granska felen
* Vanliga orsaker: diskutrymme, skadade filer, minnesproblem
* Se avsnittet Felsökning nedan

**Systemet svarar inte:**

* Chloros+ parallellt läge använder för mycket resurser
* Överväg att minska antalet samtidiga uppgifter eller uppgradera hårdvaran
* Fritt läge är mindre resurskrävande

### När du ska avbryta bearbetningen

Avbryt bearbetningen om du ser:

* ❌ Felmeddelanden som ”Disk full” eller ”Kan inte skriva fil”
* ❌ Upprepade fel på bildfiler
* ❌ Systemet har fryst helt (svarar inte)
* ❌ Insåg att felaktiga inställningar hade konfigurerats
* ❌ Felaktiga bilder importerade

**Så här avbryter du:**

1. Klicka på**knappen Stopp/Avbryt** (ersätter Start-knappen)
2. Bearbetningen avbryts, framstegen går förlorade
3. Åtgärda problemen och starta om från början

***

## Felsökning under bearbetning

### Bearbetningen går mycket långsamt

**Möjliga orsaker:**

* Omärkta målbilder (skannar alla bilder)
* HDD istället för SSD-lagring
* Otillräckliga systemresurser
* Många index konfigurerade
* Åtkomst till nätverksenhet

**Lösningar:**

1. Om du just har startat och befinner dig i detekteringsfasen: Avbryt, markera mål, starta om
2. För framtiden: Använd SSD, minska antalet index, uppgradera hårdvaran
3. Överväg CLI för batchbearbetning av stora datamängder

### Varningar om ”diskutrymme”

**Lösningar:**

1. Frigör diskutrymme omedelbart
2. Flytta projektet till en enhet med mer utrymme
3. Minska antalet index som ska exporteras
4. Använd JPG-format istället för TIFF (mindre filer)

### Frekventa meddelanden om &quot;skadade filer&quot;

**Lösningar:**

1. Kopiera om bilderna från SD-kortet för att säkerställa integriteten
2. Testa SD-kortet för fel
3. Ta bort skadade filer från projektet
4. Fortsätt bearbeta återstående bilder

### Systemöverhettning / Throttling

**Lösningar:**

1. Se till att ventilationen är tillräcklig
2. Rengör datorns ventilationsöppningar från damm
3. Minska bearbetningsbelastningen (använd Free-läge istället för Chloros+)
4. Bearbeta under svalare tider på dygnet

***

## Meddelande om att bearbetningen är klar

När bearbetningen är klar:

* Visas en förloppsindikator som når 100 %
* Visas meddelandet **”Bearbetning klar”** i felsökningsloggen
* Blir startknappen aktiverad igen
* Finns alla utdatafiler i undermappen för kameramodellen

***

## Nästa steg

När bearbetningen är klar:

1. **Granska resultaten** – Se [Avsluta bearbetningen](finishing-the-processing.md)
2. **Kontrollera utdatamappen** – Kontrollera att alla filer har exporterats korrekt
3. **Granska felsökningsloggen** – Kontrollera om det finns några varningar eller fel
4. **Förhandsgranska bearbetade bilder** – Använd bildvisaren eller extern programvara

För information om hur du granskar och använder dina bearbetade resultat, se [Avsluta bearbetningen](finishing-the-processing.md).
