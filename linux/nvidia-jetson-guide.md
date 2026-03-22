# NVIDIA Jetson-guide

Chloros på NVIDIA Jetson möjliggör multispektral bildbehandling i fält – ute på fältet, på drönare och i avlägsna installationer. Chloros identifierar automatiskt din Jetson-modell och optimerar bearbetningsstrategin för din hårdvara.

***

## Jetson-modeller som stöds

| Modell                | RAM            | Bearbetningsstrategi                                   | Rekommenderad användning                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB delat | `GPU_PARALLEL` (4 arbetare)                            | Maximal prestanda, stora datamängder                      |
| **Jetson Orin NX**   | 8–16 GB delat  | `GPU_PARALLEL` (3 arbetare, 16 GB) / `GPU_SINGLE` (8 GB) | Förstahandsrekommendation för användning i luften och på fältet |
| **Jetson Orin Nano** | 8 GB delat     | `GPU_SINGLE` (1 arbetsenhet)                               | Edge-beräkning på instegsnivå                                 |
| **Jetson Nano**      | 4–8 GB delat   | `GPU_SINGLE` (1 arbetare)                               | Instegsnivå, begränsat minne                          |

{% hint style="info" %}
**Äldre Jetson-modeller** (TX2, TX1, Xavier NX) stöds eventuellt inte. Prestandan varierar beroende på tillgängligt GPU-minne och CUDA-kapacitet.
{% endhint %}

***

## Krav

* **JetPack 6.x** (senaste versionen rekommenderas)
* **NVIDIA CUDA** (ingår i JetPack)
* **Chloros+-licens** (krävs för åtkomst till CLI/SDK)

## Installation

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

För allmän information om installation av Linux, se [Linux Installation](linux-installation.md).

***

## Dynamisk beräkningsanpassning på Jetson

Chloros identifierar automatiskt din Jetson-modell och väljer den optimala bearbetningsstrategin. **Ingen manuell inställning krävs.**

### Hur det fungerar

Vid uppstart profilerar Chloros ditt system:

1. **Identifierar Jetson-modellen** via `/proc/device-tree/model`
2. **Läser tillgängligt GPU/delat minne**

3.**Väljer en bearbetningsstrategi** (`GPU_PARALLEL`, `GPU_SINGLE` eller `CPU_PARALLEL`)
4. **Ställer in antal arbetare, pipelintyp och minnesallokering** automatiskt

### Beteende per modell

| Jetson-modell                | Strategi       | Arbetare | Pipeline                       | Parallellitet |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (minneseffektiv) | Seriell  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Seriell  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serialiserad  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (full GPU-väg)    | Samtidig  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Samtidig  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** är det perfekta valet för edge-distribution – den använder strategin `GPU_PARALLEL` med 3 samtidiga arbetare och levererar verklig parallell GPU-bearbetning i ett kompakt format.
{% endhint %}

Den viktigaste skillnaden mellan plattformarna är **minnet**. En Jetson Nano med 8 GB delat minne måste bearbeta bilder en i taget med hjälp av en minneseffektiv tiled-metod, medan en Orin NX med 16 GB kan köra 3 bilder genom GPU:n samtidigt med hjälp av den fuserade pipelinen med högre genomströmning.

För den fullständiga referensen för beräkningsanpassning, se [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md).

***

## Värmehantering

Jetson-enheter har begränsat värmeutrymme, särskilt i slutna eller luftburna installationer. Chloros inkluderar automatisk värmeövervakning och strypning:

| Temperatur         | Åtgärd                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normal drift — full bearbetningshastighet          |
| **70 °C** (Varning)  | Minska batchstorleken automatiskt                   |
| **80 °C** (Kritiskt) | Aggressiv begränsning — lägre samtidighet         |
| **90 °C** (Avstängning) | Stoppa GPU-bearbetningen helt — nedkylning krävs |

{% hint style="warning" %}
**Säkerställ tillräcklig ventilation och kylning** för kontinuerlig bearbetning, särskilt i slutna fältkapslingar eller luftburna system. Termisk strypning minskar bearbetningskapaciteten för att skydda hårdvaran.
{% endhint %}

***

## Minnehantering

Jetson-enheter använder **enhetligt minne** — GPU och CPU delar samma fysiska RAM-minne. Detta innebär att det rapporterade VRAM-minnet (t.ex. 15,3 GB på Orin NX 16 GB) inte är dedikerat GPU-minne; det delas med operativsystemet och andra processer.

### Rekommendationer för swap

För stora datamängder eller Texture Aware-debayer-bearbetning kan Chloros rekommendera att man skapar swap-utrymme:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Uppskattat minnesbehov per bild:**

* Standard-debayer: ~10 MB per bild
* Texture Aware-debayer: ~15 MB per bild

Chloros beräknar automatiskt det nödvändiga minnet baserat på storleken på din datamängd och varnar dig om swap rekommenderas.

### OOM (Out of Memory) Fallback

Om ett minnesbristtillstånd upptäcks under bearbetningen:

1. Chloros minskar automatiskt antalet GPU-arbetare
2. Fallback från `fused_gpu` till `tiled_gpu`-pipeline (mer minneseffektiv)
3. Fortsätter bearbetningen med reducerad genomströmning istället för att krascha

***

## Fältinstallation

### Strömförsörjning

| Jetson-modell     | Typisk strömförbrukning | Anmärkningar                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5–10 W              | USB-C eller cylindrisk kontakt    |
| Jetson Orin Nano | 7–15 W              | DC-cylindrisk kontakt          |
| Jetson Orin NX   | 10–25 W             | DC-cylindrisk kontakt          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD eller cylindrisk kontakt |

Planera din strömförbrukning för kontinuerlig bearbetning — den högsta strömförbrukningen inträffar under den GPU-intensiva tråd 3 (Bearbetning).

### Rekommendationer för lagring

* **NVMe SSD** rekommenderas starkt för arm64-implementeringar
* SD-kort är för långsamma för bearbetning — använd endast som startmedia
* Planera för 2–3 gånger storleken på dina råa bilddata för bearbetad utdata

### Headless-drift via SSH

Chloros CLI är idealisk för headless Jetson-installationer:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

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
    --format tiff-32 \
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
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Rekommenderade Jetson-system för fältanvändning

För fält- och luftburna installationer bör du överväga dessa Jetson Orin NX 16 GB-bärkort:

* **Luftburet/drönare**: System med vibrationsklassning (MIL-STD), låg vikt (under 300 g) och passiv kylning
* **Robust fältanvändning**: Vattentäta höljen enligt IP67/IP69K med PoE GigE-kameranslutning
* **Minimal/budget**: Utvecklingskit med tillbehörshöljen

Kontakta [MAPIR Support](https://www.mapir.camera/community/contact) för specifika hårdvarurekommendationer för ditt användningsscenario.

***

## Nästa steg

* [Linux Installation](linux-installation.md) — Allmänna detaljer om Linux-installation
* [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md) — Fullständig referens för beräkningsstrategier
* [Processing Pipeline](../processing-architecture/processing-pipeline.md) — Förstå 4-trådspipeline
* [CLI : Command Line](../CLI.md) — Fullständig CLI-referens
* [API : Python SDK](../api-python-sdk.md) — Fullständig referens för SDK
