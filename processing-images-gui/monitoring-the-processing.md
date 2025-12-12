# Övervakning av bearbetningen

När bearbetningen har startat erbjuder Chloros flera sätt att övervaka framstegen, kontrollera om det finns problem och förstå vad som händer med din dataset. På denna sida förklaras hur du kan spåra din bearbetning och tolka den information som Chloros tillhandahåller.

## Översikt över förloppsindikatorn

Förloppsindikatorn i den övre rubriken visar bearbetningsstatus och färdigställandegrad i realtid.

### Förloppsindikator i fritt läge

För användare utan Chloros+-licens:

**Förloppsindikator i två steg:**

1. **Måldetektering** – Hitta kalibreringsmål i bilder
2. **Bearbetning** – Tillämpa korrigeringar och exportera

**Förloppsindikatorn visar:**

* Total procentuell färdigställningsgrad (0–100 %)
* Namn på aktuellt steg
* Enkel horisontell stapelvisualisering

### Chloros+ förloppsindikator

För användare med Chloros+-licens:

**4-stegs förloppsindikator:**

1. **Detektering** – Hitta kalibreringsmål
2. **Analys** – Granska bilder och förbereda pipeline
3. **Kalibrering** – Tillämpa vignett- och reflektanskorrigeringar
4. **Export** – Spara bearbetade filer

**Interaktiva funktioner:**

* **Håll muspekaren över** förloppsindikatorn för att se den utökade panelen med fyra steg
* **Klicka** på förloppsindikatorn för att frysa/fästa den utökade panelen
* **Klicka igen** för att låsa upp och dölja automatiskt när musen lämnar panelen
* Varje steg visar individuellt förlopp (0–100 %)

***

## Förstå varje bearbetningssteg

### Steg 1: Detektering (målidentifiering)

**Vad händer:**

* Chloros skannar bilder som är markerade med kryssrutan Mål
* Datorvisionsalgoritmer identifierar de fyra kalibreringspanelerna
* Reflektansvärden extraheras från varje panel
* Målets tidsstämplar registreras för korrekt kalibreringsschemaläggning

**Varaktighet:**

* Med markerade mål: 10–60 sekunder
* Utan markerade mål: 5–30+ minuter (skannar alla bilder)

**Förloppsindikator:**

* Detektering: 0 % → 100 %
* Antal skannade bilder
* Antal hittade mål

**Vad du ska titta efter:**

* Bör slutföras snabbt om målen är korrekt markerade
* Om det tar för lång tid kan det hända att målen inte är markerade
* Kontrollera felsökningsloggen för meddelanden om ”Mål hittat”

### Steg 2: Analysera

**Vad händer:**

* Läser EXIF-metadata för bilder (tidsstämplar, exponeringsinställningar)
* Bestämmer kalibreringsstrategi baserat på måltidsstämplar
* Organiserar bildbehandlingskön
* Förbereder parallella bearbetningsarbetare (endast Chloros+)

**Varaktighet:** 5–30 sekunder

**Förloppsindikator:**

* Analyserar: 0 % → 100 %
* Snabb etapp, slutförs vanligtvis snabbt

**Vad du ska vara uppmärksam på:**

* Bör fortskrida stadigt utan pauser
* Varningar om saknade metadata visas i felsökningsloggen

### Steg 3: Kalibrering

**Vad händer:**

* **Debayering**: Konvertera RAW-Bayer-mönster till 3 kanaler
* **Vignettkorrigering**: Ta bort mörkare kanter på linsen
* **Reflektionskalibrering**: Normalisera med målvärden
* **Indexberäkning**: Beräkna multispektrala index
* Bearbeta varje bild genom hela processen

**Varaktighet:** Största delen av den totala bearbetningstiden (60–80 %)

**Förloppsindikator:**

* Kalibrering: 0 % → 100 %
* Aktuell bild bearbetas
* Bilder färdiga / Totalt antal bilder

**Bearbetningsbeteende:**

* **Fritt läge**: Bearbetar en bild i taget i sekventiell ordning
* **Chloros+-läge**: Bearbetar upp till 16 bilder samtidigt
* **GPU-acceleration**: Påskyndar denna fas avsevärt

**Vad du ska vara uppmärksam på:**

* Stadig framsteg genom bildantalet
* Kontrollera felsökningsloggen för meddelanden om färdigställande per bild
* Varningar om bildkvalitet eller kalibreringsproblem

### Fas 4: Exportera

**Vad händer:**

* Skriver kalibrerade bilder till disk i valt format
* Exporterar multispektrala indexbilder med LUT-färger
* Skapar undermappar för kameramodeller
* Bevarar originalfilnamn med lämpliga suffix

**Varaktighet:** 10–20 % av den totala bearbetningstiden

**Framstegsindikator:**

* Exporterar: 0 % → 100 %
* Filer som skrivs
* Exportformat och destination

**Vad du ska hålla koll på:**

* Varningar om diskutrymme
* Fel vid filskrivning
* Slutförande av alla konfigurerade utdata

***

## Fliken Debug Log (Felsökningslogg)

Felsökningsloggen ger detaljerad information om bearbetningsförloppet och eventuella problem som uppstått.

### Öppna felsökningsloggen

1. Klicka på ikonen **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> i vänster sidofält
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

**Åtgärd:** Avbryt bearbetningen, åtgärda felet och starta om.

### Vanliga loggmeddelanden

| Meddelande                          | Betydelse                                | Åtgärd som krävs                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Mål upptäckt i \[filnamn]&quot; | Kalibreringsmål hittat  | Ingen åtgärd – normalt                                         |
| &quot;Bearbetar bild X av Y&quot;        | Aktuell uppdatering av framsteg                | Ingen åtgärd – normalt                                         |
| &quot;Inga mål hittade&quot;               | Inga kalibreringsmål upptäckta        | Markera målbilder eller inaktivera reflektionskalibrering |
| &quot;Otillräckligt diskutrymme&quot;        | Otillräckligt lagringsutrymme för utdata          | Frigör diskutrymme                                    |
| &quot;Hoppar över skadad fil&quot;        | Bildfilen är skadad                  | Kopiera om filen från SD-kortet                             |
| &quot;PPK-data tillämpad&quot;               | GPS-korrigeringar från .daq-fil tillämpad | Ingen åtgärd – normalt                                         |

### Kopiera loggdata

Så här kopierar du loggen för felsökning eller support:

1. Öppna panelen Debug Log (Felsökningslogg).
2. Klicka på knappen **&quot;Copy Log&quot;** (Kopiera logg) (eller högerklicka → Välj alla).
3. Klistra in i textfil eller e-post.
4. Skicka till MAPIR support om det behövs.

***

## Övervakning av systemresurser

### CPU-användning

**Fritt läge:**

* 1 CPU-kärna på \~100 %
* Övriga kärnor inaktiva eller tillgängliga
* Systemet förblir responsivt

**Chloros+ Parallellt läge:**

* Flera kärnor på 80–100 % (upp till 16 kärnor)
* Hög total CPU-användning
* Systemet kan kännas mindre responsivt

**För att övervaka:**

* Windows Aktivitetshanteraren (Ctrl+Skift+Esc)
* Fliken Prestanda → avsnittet CPU
* Leta efter processerna &quot;Chloros&quot; eller &quot;chloros-backend&quot;

### Minneanvändning (RAM)

**Typisk användning:**

* Små projekt (&lt; 100 bilder): 2–4 GB
* Medelstora projekt (100–500 bilder): 4–8 GB
* Stora projekt (500+ bilder): 8–16 GB
* Chloros+ parallellt läge använder mer RAM

**Om minnet är lågt:**

* Bearbeta mindre batcher
* Stäng andra applikationer
* Uppgradera RAM om du regelbundet bearbetar stora datamängder

### GPU-användning (Chloros+ med CUDA)

När GPU-acceleration är aktiverad:

* NVIDIA GPU visar hög användning (60-90 %)
* VRAM-användningen ökar (kräver 4 GB+ VRAM)
* Kalibreringsfasen går betydligt snabbare

**För att övervaka:**

* NVIDIA-ikonen i systemfältet
* Aktivitetshanteraren → Prestanda → GPU
* GPU-Z eller liknande övervakningsverktyg

### Disk-I/O

**Vad du kan förvänta dig:**

* Hög diskavläsning under analysfasen
* Hög diskskrivning under exportfasen
* SSD betydligt snabbare än HDD

**Prestandatips:**

* Använd SSD för projektmappen när det är möjligt
* Undvik nätverksenheter för stora datamängder
* Se till att disken inte är nära sin kapacitet (påverkar skrivhastigheten)

***

## Upptäcka problem under bearbetningen

### Varningssignaler

**Processen avstannar (ingen förändring på mer än 5 minuter):**

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

### När ska bearbetningen avbrytas

Avbryt bearbetningen om du ser:

* ❌ Felmeddelanden om att disken är full eller att filen inte kan skrivas
* ❌ Upprepade fel på bildfiler
* ❌ Systemet är helt fryst (svarar inte)
* ❌ Felaktiga inställningar har konfigurerats
* ❌ Felaktiga bilder har importerats

**Så här stoppar du:**

1. Klicka på **knappen Stopp/Avbryt** (ersätter knappen Start)
2. Bearbetningen avbryts, framstegen går förlorade
3. Åtgärda problemen och starta om från början

***

## Felsökning under bearbetning

### Bearbetningen går mycket långsamt

**Möjliga orsaker:**

* Omärkta målbilder (skanning av alla bilder)
* HDD istället för SSD-lagring
* Otillräckliga systemresurser
* Många index konfigurerade
* Åtkomst till nätverksenhet

**Lösningar:**

1. Om du just har startat och befinner dig i detekteringsfasen: Avbryt, markera mål, starta om
2. För framtiden: Använd SSD, minska index, uppgradera hårdvara
3. Överväg CLI för batchbearbetning av stora datamängder

### Varningar om ”diskutrymme”

**Lösningar:**

1. Frigör diskutrymme omedelbart
2. Flytta projektet till en enhet med mer utrymme
3. Minska antalet index som ska exporteras.
4. Använd JPG-format istället för TIFF (mindre filer).

### Frekventa meddelanden om &quot;korrupta filer&quot;

**Lösningar:**

1. Kopiera om bilderna från SD-kortet för att säkerställa integriteten.
2. Testa SD-kortet för fel.
3. Ta bort korrupta filer från projektet.
4. Fortsätt bearbeta återstående bilder.

### Systemet överhettas/stryps

**Lösningar:**

1. Se till att ventilationen är tillräcklig.
2. Rengör datorns ventilationsöppningar från damm.
3. Minska bearbetningsbelastningen (använd Free-läget istället för Chloros+).
4. Bearbeta under svalare tider på dygnet.

***

## Meddelande om att bearbetningen är klar

När bearbetningen är klar:

* Förloppsindikatorn når 100 %
* Meddelandet **”Bearbetning klar”** visas i felsökningsloggen
* Startknappen aktiveras igen
* Alla utdatafiler finns i undermappen för kameramodellen

***

## Nästa steg

När bearbetningen är klar:

1. **Granska resultaten** – Se [Avsluta bearbetningen](finishing-the-processing.md)
2. **Kontrollera utdatamappen** – Kontrollera att alla filer har exporterats korrekt
3. **Granska felsökningsloggen** – Kontrollera om det finns några varningar eller fel
4. **Förhandsgranska bearbetade bilder** – Använd bildvisaren eller extern programvara

För information om hur du granskar och använder dina bearbetade resultat, se [Avsluta bearbetningen](finishing-the-processing.md).
