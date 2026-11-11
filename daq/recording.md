# Inspelning och .daq-formatet

En `.daq`-fil är Chloros ljussensorns inspelningsformat: en **SQLite-databas** med kalibrerade spektralbilder från en DAQ-sensor. Spela in en sådan fil under en inspelningssession så kan reflektanspipeline senare dividera varje bild med den nedåtriktade strålningsintensiteten som mättes just i det ögonblicket.

## Vad en .daq-fil innehåller

| Egenskap | Värde |
| --- | --- |
| Behållare | SQLite-databas, en fil per sensor per inspelning |
| Filnamn | Innehåller **sensor-ID**och en**tidsstämpel**, t.ex. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum per bildruta | 135 punkter, 340–1010 nm i steg om 5 nm, plus CIE XYZ-tristimulus |
| Enheter | Kalibrerad spektral irradians, **W/m²/nm** (fabriks kalibreringspaket + tillämpad kapselprofil) |
| Inlagda metadata | Sensor-ID (nyckeln för att hämta enhetens fabrikskalibrering) och den gällande kap-profilen — se [Kap-profiler och kalibrerat område](caps-and-range.md) |

Formatet är identiskt för DAQ-U, DAQ-M, DAQ-E och DAQ-E-S, så vid vidare bearbetning spelar det ingen roll vilken enhet som registrerade data. En DAQ-E-S-inspelning lagrar även enhetens lutning, rullning och tippning bredvid varje spektrum.

**Inspelningar överlever ett strömavbrott (1.3.0).**En liveinspelning skrivs med en logg och en fullständig disksynkronisering vid varje bekräftelse, så ett strömavbrott eller en avbruten process kostar endast de senaste 3 sekunderna, inte hela filen. En inspelning som avbröts mitt i skrivningen kan lämna kvar en `<name>.daq-journal`-fil bredvid `.daq`:**behåll och kopiera de båda tillsammans.** Chloros rullar tillbaka journalen första gången den läser filen. En upptagen disk fördröjer filen, men aldrig sensorns avläsningar.

**En DAQ-E som slutar leverera data återansluts (1.3.0).** När en DAQ-E:s ström tystnar (om strömmen till dess switch har slagits av och på, eller om en länk har brutits), återansluter backend-systemet den med de inställningar den hade, och en inspelning på den fortsätter i samma `.daq`-fil med avbrottet markerat i filen.

Kalibrerad inspelning kräver sensorns fabrikskalibreringspaket. För DAQ-U och DAQ-M hämtar backend-systemet paketet från MAPIRs moln utifrån sensor-ID (inspelning nekas om detta inte går); DAQ-E-enheter undantas eftersom de lagrar sin kalibrering på enheten.

## Inspelning från GUI

Inspelning i GUI kräver ett **öppet projekt** (annars är inspelningsknapparna inaktiverade):

* **Spela in allt / Stoppa allt** — högst upp i sidofältet för ljussensorer; startar eller stoppar en `.daq`-inspelning på alla anslutna sensorer samtidigt.
* **Spela in / Stoppa inspelning** — per sensor, i inställningsfönstret med kugghjulsikonen. En röd ”REC”-indikator visas i sensorns realtidsinformationsrader under inspelningen.

Filerna skrivs till `<project>/light_sensor/`, och när en inspelning avslutas — oavsett om det sker via Stopp, Stoppa allt eller genom att koppla bort en inspelningssensor — läggs den färdiga `.daq`-filen **automatiskt till det öppna projektet**. Den visas i projektets fillista utan att behöva läggas till manuellt och är redan redo för reflektansbearbetning.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Inspelning från CLI



CLI spelar in via backendens sensorpool (backenden måste vara igång – dessa kommandon är tunna HTTP-klienter):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Hämta värdet `--sensor-id` från `chloros-cli daq pool-list`. Två standardvärden som är värda att känna till:

| Alternativ | Standard |
| --- | --- |
| `--duration` | `0` — spela in tills `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` på **backendens** filsystem, inte på CLI:s |

Skillnaden mellan utdatakatalogerna är viktig när CLI:n riktar sig mot en backend på en annan maskin: filen hamnar där backenden körs. Om backend finns på samma dator tolkas en relativ `-o` (t.ex. `./out` ovan) utifrån den mapp där du körde kommandot.

## Inspelning från Python

`DAQSensorSession` (returneras av `chloros_sdk.connect_daq_sensor()`) exponerar samma samlade inspelning: `record_start(output_dir=None, device_name=None)` returnerar filvägen, `record_stop()` returnerar `{path, rows}`. Se [SDK-referensen](../reference/sdk-reference.md) för det fullständiga sessions-API:et. SDK:s klasser för direkt hårdvaruhantering (endast för stationära installationer) skriver inspelningar till `~/Documents/DAQ/` som standard; för släppta versioner är den samlade sökvägen ovan den stödda vägen.

## Använda en .daq-fil vid bearbetning

För att beräkna reflektans från bilder behöver Chloros-värden för nedåtriktad strålningsintensitet som matchar varje exponering:

* **Spara `.daq` tillsammans med bilderna.**Vid bearbetning beräknar bearbetningskedjan automatiskt den**tidsstämplade nedåtriktade strålningen** från en inspelad `.daq` (valfri DAQ-modell) — eller från en DAQ-M-specifik `.csv` — som finns tillsammans med bilderna. GUI-inspelningar uppfyller detta krav automatiskt, eftersom de läggs till i projektet så fort de avslutas.
* **Kalibreringen hämtas vid behov.** Om ett fabrikskalibreringspaket per kamera eller per DAQ inte redan finns cachelagrat lokalt hämtar Chloros det automatiskt från MAPIRs moln vid första användningen (internet krävs en gång; cachelagrad under `~/.chloros/`).
* **Liveinspelningar skriver sin egen sidokar.** För varje reflektansbild som fångas live sparas det DAQ-värde som faktiskt användes som en `.daq`-fil i inspelningens `daq/`-mapp, så att inspelningen kan bearbetas på nytt senare utan den ursprungliga inspelningen.
* **Inspelningar från `chloros_scripts`-inspelarna (1.3.0).**En `.daq`-fil som skrivits av de allmänna [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)-inspelarna anges som den DAQ-modell som skapade den, och dess gränsvärde kan ställas in i**Projektinställningar → DAQ-ljussensor** precis som för alla andra inspelningar. Dessa inspelningsenheter skriver ut råa sensorräkneställningar; Chloros hämtar sensorns fabrikskalibrering via seriell anslutning och tillämpar den.
* **Vems avläsning det är (1.3.0).**Bredvid varje inspelning `.daq` finns en `.daq.cameras.json` som anger vilken eller vilka kameror vars bildrutor avläsningen avser, och bearbetningen matchar den**endast mot bilder från dessa kameror**. En fristående kamera utan egen ljussensor lånar därför inte längre en annan kameras avläsning – koppla en sensor till den, eller spela in en. En inspelning gjord med själva DAQ:en (GUI Record, `daq pool-record`), och alla `.daq` utan den filen (även äldre inspelningar), fungerar fortfarande för alla kameror.

## Hämta tillbaka irradiansvärdena

När ett projekt bearbetas exporteras även alla ljussensorinspelningar som det innehåller till en
`Light Sensor/`-mapp bredvid bildprodukterna. Detta kräver **inte** bilddata: en
ljussensorinspelning i sig är en komplett inspelning, och en mapp som endast innehåller `.daq`-
filer är en giltig indata. Ett projekt som endast innehåller ljussensorinspelningar listar dem och
bearbetas både från appen och CLI, och de förblir listade efter körningen. Körningen
rapporterar hur många ljussensorprodukter den skrev.

| Produkt | Vad det är |
| --- | --- |
| `<name>_calibrated.daq` | Ett arkiv som kan bearbetas på nytt enligt samma schema som en liveinspelning, men som nu anger det kalibreringspaket som genererade det. Att importera det på nytt innebär **inte** att det kalibreras en andra gång. |
| `<name>_calibrated.csv` | Spektral irradians i W/m²/nm på sensorns eget våglängdsrutnät, en rad per avläsning, plus fotometriska kolumner: total effekt, fotopisk och skotopisk lux, PPFD med uppdelning i blått/grönt/rött samt toppvåglängd. |

En DAQ-U eller DAQ-M vars kalibreringspaket inte kan hämtas – du är offline, eller
sensorn saknar kalibrering i arkivet – **hoppas över med en motivering** och skrivs aldrig ut
som en ”kalibrerad” fil med råa mätvärden. Anslut till internet och kör om. En DAQ-E
har sin egen kalibrering, så den behöver detta endast när enheten inte är ansluten och
inget finns cachelagrat lokalt.

### DAQ-A: råa mätvärden, och varför det är rätt svar

**DAQ-A** är äldre än systemet med kalibreringspaket per seriell anslutning och har inget paket att
hämta. Det är inget förbiseende: en DAQ-A kalibreras ute på fältet mot ett
reflektansmål, och målbaserad kalibrering behöver endast sensorns *relativa*
respons – vilket är precis vad dess råa räkneställningar är. Chloros kalibrerar med dem idag.

Så en DAQ-A-inspelning kan exporteras, men under ett annat namn:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, inte `_calibrated` – ett annat filnamn istället för en flagga inuti filen,
eftersom uppgiften måste klara att filen skickas vidare via e-post som ett rent namn. Rubriken `.csv`
anger `raw spectral sensor counts (NOT irradiance)` och varnar för att värdena är
jämförbara **inom** filen och inte mellan olika sensorer. De kolumner som endast har
betydelse för verklig irradians — total effekt, lux, PPFD — lämnas tomma istället för att
beräknas utifrån räknevärden.

Äldre DAQ-A-SD-inspelningar (schema v1.01 / v1.02) registrerar endast filens skrivtid, inte en
tidsstämpel per avläsning. Chloros kommer inte att matcha bilder mot dessa – att koppla en bildruta till en
skrivtid skulle vara fel utan att det någonsin ser fel ut – men exporten läser dem utan problem och
CSV-filen anger vilken klocka det gäller.

För en fullständig beskrivning av reflektansmätningar – en sensor med kamera och två sensorer för omgivning/objekt – se [Arbetsflöden för reflektans](reflectance.md).
