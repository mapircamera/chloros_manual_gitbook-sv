# Inspelning och .daq-formatet

En `.daq`-fil är Chloros:s inspelningsformat för ljussensorer: en **SQLite-databas** med kalibrerade spektralbilder från en DAQ-sensor. Spela in en sådan fil under en inspelningssession så kan reflektanspipeline senare dividera varje bild med den nedåtriktade strålningsintensiteten som mättes just i det ögonblicket.

## Vad en .daq-fil innehåller

| Egenskap | Värde |
| --- | --- |
| Behållare | SQLite-databas, en fil per sensor per inspelning |
| Filnamn | Innehåller **sensor-ID**och en**tidsstämpel**, t.ex. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum per bildruta | 135 punkter, 340–1010 nm i steg om 5 nm, plus CIE XYZ-tristimulus |
| Enheter | Kalibrerad spektral irradians, **W/m²/nm** (fabriks kalibreringspaket + tillämpad kapselprofil) |
| Inlagda metadata | Sensor-ID (nyckeln för att hämta enhetens fabrikskalibrering) och den gällande kapselprofilen — se [Kapselprofiler och kalibrerat område](caps-and-range.md) |

Formatet är identiskt för DAQ-U, DAQ-M och DAQ-E, så vid vidarebehandling spelar det ingen roll vilken transportenhet som registrerade data.

Kalibrerad registrering kräver sensorns fabrikskalibreringspaket. För DAQ-U och DAQ-M hämtar backend-systemet paketet från MAPIR:s moln utifrån sensor-ID (inspelning nekas om detta inte går); DAQ-E-enheter undantas eftersom de lagrar sin kalibrering på enheten.

## Inspelning från GUI

Inspelning i GUI kräver ett **öppet projekt** (annars är inspelningsknapparna inaktiverade):

* **Spela in allt / Stoppa allt** — högst upp i sidofältet för ljussensorer; startar eller stoppar en `.daq`-inspelning på alla anslutna sensorer samtidigt.
* **Spela in / Avsluta inspelning** — per sensor, i inställningsfönstret (kugghjulet). En röd ”REC”-indikator visas i sensorns realtidsinformationsrader under inspelningen.

Filerna sparas i `<project>/light_sensor/`, och när en inspelning avslutas – oavsett om det sker via Stopp, Stoppa allt eller genom att koppla bort en inspelningssensor – läggs den färdiga filen `.daq` **automatiskt till det öppna projektet**. Den visas i projektets fillista utan att behöva läggas till manuellt och är redan klar för reflektansbearbetning.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Inspelning från CLI

CLI spelar in via backendens sensorpool (backenden måste vara igång – dessa kommandon är smala HTTP-klienter):

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

Skillnaden mellan utdatakatalogen är viktig när CLI riktar sig mot en backend på en annan maskin: filen hamnar där backenden körs.

## Inspelning från Python

`DAQSensorSession` (returneras av `chloros_sdk.connect_daq_sensor()`) visar samma samlade inspelning: `record_start(output_dir=None, device_name=None)` returnerar filvägen, `record_stop()` returnerar `{path, rows}`. Se [SDK-referensen](../reference/sdk-reference.md) för den fullständiga sessionen API. SDK:s direkt-hårdvaruklasser (endast för installationer på stationära datorer) skriver inspelningar till `~/Documents/DAQ/` som standard; för släppta versioner är den samlade sökvägen ovan den stödda vägen.

## Använda en .daq-fil vid bearbetning

För att beräkna reflektans från bildmaterial behöver Chloros nedåtriktad irradians som matchar varje exponering:

* **Spara `.daq` tillsammans med bildmaterialet.**Vid bearbetningstidpunkten beräknar bearbetningskedjan automatiskt den**tidsstämplade nedåtriktade strålningsintensiteten** från en inspelad `.daq` (valfri DAQ-modell) — eller från en DAQ-M-specifik `.csv` — som finns tillsammans med bilderna. GUI-inspelningar uppfyller detta automatiskt, eftersom de läggs till i projektet så fort de avslutas.
* **Kalibreringen hämtas vid behov.** Om ett fabrikskalibreringspaket per kamera eller per DAQ inte redan finns cachelagrat lokalt, hämtar Chloros det automatiskt från MAPIR:s moln vid första användningen (internet krävs en gång; cachas under `~/.chloros/`).
* **Liveinspelningar skriver sin egen sidecar-fil.** För varje reflektansbild som spelas in live sparas den DAQ-avläsning som faktiskt användes som en `.daq`-sidecar-fil bredvid bildmaterialet, så att inspelningen kan bearbetas på nytt senare utan den ursprungliga inspelningen.

## Hämta tillbaka irradiansvärdet

När ett projekt bearbetas exporteras även alla ljussensorinspelningar som det innehåller till en
`Light Sensor/`-mapp bredvid bildprodukterna. Detta kräver **inte** bilder: en
ljussensor som flugits separat utgör en komplett inspelning, och en mapp som endast innehåller `.daq`
-filer är en giltig indata. Körningen rapporterar hur många ljussensorprodukter den har skrivit.

| Produkt | Vad det är |
| --- | --- |
| `<name>_calibrated.daq` | Ett arkiv som kan bearbetas på nytt enligt samma schema som en liveinspelning, nu med uppgift om det kalibreringspaket som genererade det. Att importera det på nytt innebär **inte** att det kalibreras en andra gång. |
| `<name>_calibrated.csv` | Spektral irradians i W/m²/nm på sensorns eget våglängdsrutnät, en rad per avläsning, plus fotometriska kolumner: total effekt, fotopisk och skotopisk lux, PPFD med uppdelning i blått/grönt/rött samt toppvåglängd. |

En DAQ-U eller DAQ-M vars kalibreringspaket inte kan hämtas – du är offline, eller
sensorn saknar kalibrering i filen – **hoppas över med en motivering** och skrivs aldrig ut
som en ”kalibrerad” fil innehållande råa räkneställningar. Anslut till internet och kör om. En DAQ-E
har sin egen kalibrering, så den behöver detta endast när enheten inte är ansluten och
inget finns cachelagrat lokalt.

### DAQ-A: råa mätvärden, och varför det är rätt svar

**DAQ-A** är äldre än systemet med kalibreringspaket per seriell anslutning och har inget paket att
hämta. Det är inget förbiseende: en DAQ-A kalibreras ute på fältet mot ett
reflektansmål, och målbaserad kalibrering behöver endast sensorns *relativa*
respons – vilket är precis vad dess råa räkneställningar är. Chloros kalibrerar med dem idag.

En DAQ-A-inspelning kan alltså exporteras, men under ett annat namn:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, inte `_calibrated` – ett annat filnamn istället för en flagga inuti filen,
eftersom uppgiften måste klara att filen skickas vidare via e-post som ett rent filnamn. Rubriken `.csv`
anger `raw spectral sensor counts (NOT irradiance)` och varnar för att värdena är
jämförbara **inom** filen och inte mellan olika sensorer. De kolumner som endast har
betydelse för faktisk irradians — total effekt, lux, PPFD — lämnas tomma istället för att
beräknas utifrån mätvärden.

Äldre DAQ-A-SD-inspelningar (schema v1.01 / v1.02) registrerar endast filens skrivtid, inte en
tidsstämpel per avläsning. Chloros kommer inte att matcha bilder mot dessa – att koppla en bildruta till en
skrivtid skulle vara fel utan att det någonsin ser fel ut – men exporten läser dem utan problem och
CSV anger vilken klocka den är inställd på.

För en fullständig beskrivning av reflektans – enkel sensor med kamera och dubbel sensor för omgivning/objekt – se [Reflektansarbetsflöden](reflectance.md).
