# NVIDIA Jetson-guide

Chloros på NVIDIA Jetson möjliggör multispektral bildbehandling i fält – ute på fältet, på drönare, och i avlägsna installationer. Chloros 1.2.0 identifierar din Jetson-modell vid uppstart och optimerar bearbetningsstrategin för den hårdvara som upptäcks. **Ingen manuell inställning krävs.**

***

## Jetson-modeller som stöds

| Modell                | RAM            | Bearbetningsstrategi                                     | Rekommenderad användning                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB delat | `GPU_PARALLEL` (2 arbetsprocesser)                              | Maximal prestanda, stora datamängder                      |
| **Jetson Orin NX**   | 8–16 GB delat  | `GPU_PARALLEL` (2 arbetsprocesser, 16 GB) / `GPU_SINGLE` (8 GB)   | Förstahandsrekommendation för användning i flygplan och fält |
| **Jetson Orin Nano** | 8 GB delat     | `GPU_SINGLE` (1 arbetsenhet, sekventiell)                     | Edge-beräkning på instegsnivå                                 |

{% hint style="info" %}
arm64-paketet Linux kräver **JetPack 6**, som finns tillgängligt för Jetson Orin-familjen. Äldre modeller (Nano, TX2, Xavier NX) kan inte köra JetPack 6 och stöds inte av det aktuella paketet.
{% endhint %}

***

## Krav

* **JetPack 6.x** (senaste versionen rekommenderas)
* **NVIDIA CUDA** (ingår i JetPack)
* **Betald Chloros+-plan** — Copper-nivå eller högre (krävs för all åtkomst till CLI/SDK; tillämpas på serversidan)

## Installation

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

För allmän information om installation av Linux, filplatser och felsökning, se [Linux Installation](linux-installation.md).

{% hint style="info" %}
**Placera extraheringskatalogen på ett snabbt lagringsmedium.** De kompilerade binärfilerna packas upp till en tillfällig katalog vid varje start — vilket går extremt långsamt från ett SD-kort. Chloros använder `/mnt/ssd/tmp` automatiskt om den finns; annars ställer du in `TMPDIR` till en sökväg på din NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamisk beräkningsanpassning på Jetson

### Så här fungerar det

Vid uppstart profilerar Chloros ditt system:

1. **Identifierar Jetson-modellen** via `/proc/device-tree/model`
2. **Läser av tillgängligt delat GPU-/CPU-minne** (Jetson använder enhetligt minne)
3. **Väljer en bearbetningsstrategi** (`GPU_PARALLEL`, `GPU_SINGLE` eller `CPU_PARALLEL`)
4. **Ställer in antalet arbetare, pipelintyp och minnesallokering** automatiskt

Beslutet styrs av **det totala delade RAM-minnet**, inte av modellnamnet:

* **Under 12 GB totalt RAM-minne**(alla Jetson-enheter med 8 GB): `GPU_SINGLE` med**1 arbetsprocess – avsiktlig sekventiell bearbetning**. Minnet är för begränsat för samtidiga arbetare, så bilderna bearbetas en i taget. På Jetson-enheter med**8 GB eller mindre** hoppar tråd 3 helt över arbetarpoolen och kör sitt arbete per bild i processen.
* **12 GB eller mer**(Orin NX 16 GB, AGX Orin): det enhetliga minnet uppfyller kraven för `GPU_PARALLEL`, men antalet arbetare är**begränsat till 2 på Jetson** — GPU:n, arbetarprocessernas RAM-minne och deras CUDA-kontexter per arbetare drar alla från samma delade pool, vilket innebär att fler arbetare ökar risken för minnesbrist.

Du kan åsidosätta det automatiska valet med miljövariabeln `CHLOROS_STRATEGY` – se [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Beteende per modell

| Jetson-modell                | Strategi       | Arbetare | Exekvering                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Sekventiell slinga i processen (`tiled_gpu` vid minnesbrist) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Sekventiell slinga inom processen                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Parallella arbetsprocesser, `fused_gpu`-väg  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Parallella arbetsprocesser, `fused_gpu`-sökväg  |

Den viktigaste skillnaden mellan plattformarna är **minnet**. En Jetson med 8 GB måste bearbeta bilder en i taget med hjälp av en minneseffektiv, segmenterad metod när belastningen är hög, medan en Orin med 16 GB eller mer kan köra två bilder genom GPU:n samtidigt med hjälp av den sammanslagna pipelinen med högre genomströmning.

### GPU-budget per modell

Varje Jetson-modell har också en hårdvaruprofil som begränsar hur mycket av den delade minnespoolen som bearbetningen får ta i anspråk och som skalar batchstorlekarna:

| Modell | Tak för GPU-budget | Multiplikator för batchstorlek | Reserverat för system/skärm |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

Detekterat RAM-minne justerar profilen: en Jetson som rapporterar **16 GB eller mer** får sin batchmultiplikator höjd med ×1,2. Basbatchstorleken före multiplikatorer är 8 bilder.

För fullständig referens om beräkningsanpassning, se [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md).

***

## GPU-frekvensbegränsning för Texture Aware på Nano och Orin Nano

Texture Aware-debayern kör inferens i ett neuralt nätverk på GPU:n, vilket kan utlösa **varningar om överström**på Jetson-modeller med låg effekt (10–15 W-klassen) vid full GPU-klockfrekvens. Innan Texture Aware-bearbetning på en**Jetson Nano eller Orin Nano**, kontrollerar Chloros GPU:ns maximala frekvens och begränsar den till**510 MHz** (510000000) om den för närvarande är högre:

* Om CLI kan skriva till sysfs-noden för GPU-frekvensen, **tillämpas begränsningen automatiskt** och en bekräftelse visas.
* Om inte (kräver root-behörighet) skriver CLI ut det exakta `sudo`-kommandot för att tillämpa begränsningen manuellt, väntar en stund så att du hinner läsa det och fortsätter sedan – bearbetningen pågår fortfarande men kan visa varningar om överström.

Så här tillämpar du begränsningen själv före bearbetningen:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modeller med högre effekt (Orin NX 25W, AGX Orin 60W) körs med full GPU-hastighet; ingen begränsning tillämpas. Standard-debayern utlöser aldrig begränsningen på någon modell.

{% hint style="info" %}
**Texture Aware på Jetson kör alltid en bild i taget.** Varje arbetare skulle behöva sitt eget CUDA-sammanhang (~1 GB) plus sin egen kopia av brusreduceringsmodellen, vilket det enhetliga minnet inte klarar av – därför är Texture Aware-vägen på Jetson knuten till en enda arbetare med serialiserad GPU-åtkomst. Räkna med att Texture Aware är betydligt långsammare än Standard på alla Jetson-enheter.
{% endhint %}

***

## Värmehantering

Jetson-enheter har begränsat värmeutrymme, särskilt i slutna eller flygburna installationer. Chloros övervakar SoC-temperaturen och begränsar batchstorlekarna automatiskt:

| Temperatur         | Åtgärd                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normal drift — full bearbetningshastighet          |
| **70 °C** (Varning)  | Batchstorleken minskas gradvis (100 % → 50 % mellan 70 °C och 80 °C) |
| **80 °C** (Kritiskt) | Aggressiv begränsning (50 % → 0 % mellan 80 °C och 90 °C) |
| **90 °C** (Avstängning) | Stoppa GPU-bearbetningen helt — nedkylning krävs |

{% hint style="warning" %}
**Säkerställ tillräcklig ventilation och värmeavledning** för kontinuerlig bearbetning, särskilt i slutna fältkapslingar eller flygburna system. Termisk begränsning minskar bearbetningskapaciteten för att skydda hårdvaran.
{% endhint %}

***

## Minnehantering

Jetson-enheter använder **enhetligt minne** — GPU:n och CPU:n delar samma fysiska RAM-minne. Det angivna VRAM-minnet (t.ex. ~15,3 GB på en Orin NX 16 GB) är inte dedikerat GPU-minne; det är samma RAM-minne som operativsystemet och alla andra processer använder.

### Varning och rekommendationer angående swap

Innan bearbetningen på Jetson påbörjas räknar CLI antalet RAW-bilder i din inmatningsmapp (`.tif`, `.tiff`, `.raw`, `.dng` — JPG-förhandsvisningar räknas inte), uppskattar det maximala minnesbehovet för körningen och **varnar innan start** om RAM + swap sannolikt är otillräckligt. Varningen har rubriken `LOW MEMORY WARNING - Jetson Detected`, visar antalet bilder, RAM-minnet, aktuellt swap-utrymme och det uppskattade maximala behovet, och ger sedan de exakta kommandona `fallocate` / `chmod` / `mkswap` / `swapon` anpassade efter ditt projekt (aldrig mindre än 8 GB). Den pausar några sekunder så att meddelandet inte försvinner i rullningsloggen, därefter fortsätter bearbetningen.**Minneuppskattningar som används av varningen:**

| Debayer-läge | Bas | Per bild |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Texture Aware | ~2,5 GB (modell + Python-körningstid) | ~15 MB |

Varningen utlöses när den uppskattade toppvärdet överskrider RAM + swap minus en säkerhetsmarginal på 1 GB, och den räknar endast **filbaserad** swap — en konfiguration som endast använder zram kommer fortfarande att flaggas.

Så här lägger du till swap manuellt (exempel: 8 GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```### Hantering av OOM (Out of Memory)

Under bearbetningen övervakar Chloros GPU-minnet och trappar ner prestandan smidigt istället för att krascha:

1. När GPU-minnesutnyttjandet överstiger **85 %** minskas batchstorlekarna förebyggande
2. Om en minnesbrist ändå inträffar **halveras** batchstorleken, och halveras igen vid varje efterföljande OOM; varje efterföljande lyckad batch drar tillbaka den straffåtgärden ett steg
3. Vid ihållande belastning övergår pipelinen från `fused_gpu` till den minneseffektiva vägen `tiled_gpu` och, som en sista utväg, till CPU-bearbetning

***

## Driftsättning på fältet

### Överväganden kring strömförsörjning

| Jetson-modell     | Typisk strömförbrukning | Anmärkningar                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC-cylinderkontakt          |
| Jetson Orin NX   | 10–25 W             | DC-cylinderkontakt          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD eller cylinderkontakt |

Planera din strömförbrukning för kontinuerlig bearbetning — den högsta strömförbrukningen inträffar under den GPU-intensiva tråden 3 (Bearbetning).

### Rekommendationer för lagring

* **NVMe SSD** rekommenderas starkt för arm64-installationer
* SD-kort är för långsamma för bearbetning — använd dem endast som startmedia
* Räkna med att den bearbetade utdata volymen blir 2–3 gånger större än den råa bilddatans storlek

### Headless-drift via SSH

Chloros CLI är idealisk för headless Jetson-installationer:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Alltid påslagen backend för LATTICE / DAQ-E-tidssynkronisering

Om din Jetson styr LATTICE-kameror eller DAQ-E-ljussensorer utan skärm, aktivera backend-systemd-tjänsten så att PTP-grandmastern körs kontinuerligt (enheten är installerad men inte aktiverad som standard):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Se [Linux Installation](linux-installation.md#always-on-ptp-for-headless-hosts) för mer information, inklusive hur paketet gör PTP-portarna 319/320 bindbara utan root-behörighet.

### Automatiserad bearbetning med systemd

Skapa en systemd-tjänst för automatiserad bearbetning:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

`chloros-cli process` avslutas med ett värde som inte är noll när en körning som begärt produkter inte skriver några bilder, vilket gör att systemds felstatus är meningsfull för övervakning.

Kombinera med en systemd-timer för schemalagd bearbetning:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Exempel på arbetsflöden

### Grundläggande Jetson-bearbetning

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI
```

### Python SDK på Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Batchbearbetning av flera flygningar

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Rekommenderade Jetson-system för fältanvändning

För fält- och luftburna installationer bör du överväga följande alternativ för Jetson Orin NX 16 GB-bärarkort:

* **Flygande/drönare**: System med vibrationsklassning (MIL-STD), låg vikt (under 300 g) och passiv kylning
* **Robust fältanvändning**: Vattentäta höljen enligt IP67/IP69K med PoE GigE-kameranslutning
* **Minimal/budget**: Utvecklingskit med tilläggskapslingar

Kontakta [MAPIR Support](https://www.mapir.camera/community/contact) för specifika hårdvarurekommendationer för just ditt användningsscenario.

***

## Nästa steg

* [Linux Installation](linux-installation.md) — Allmän information om installation av Linux
* [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md) – Fullständig referens för beräkningsstrategier
* [Bearbetningspipeline](../processing-architecture/processing-pipeline.md) – Förstå den 4-trådiga pipelinen
* [CLI : Kommandorad](../CLI.md) — Guiden för CLI
* [API : Python SDK](../api-python-sdk.md) — Guiden för SDK
* [CLI-referens](../reference/cli-reference.md) och [SDK-referens](../reference/sdk-reference.md) — Utförliga kommando-/API-listor för 1.2.0
