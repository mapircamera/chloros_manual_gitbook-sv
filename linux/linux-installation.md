# Installation av Linux

Chloros distribueras för Linux som `.deb`-paket som installerar CLI och backend. Python SDK installeras separat via pip.

***

## Linux amd64 (x86_64)

### Systemkrav

| Krav | Minsta | Rekommenderat |
| --- | --- | --- |
| **Distribution** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Processor** | x86_64 (Intel/AMD) | Intel Core i7 eller bättre |
| **Minne (RAM)** | 8 GB | 16 GB eller mer |
| **Grafikkort** | Inget (CPU-bearbetning) | NVIDIA GPU med 4 GB+ VRAM |
| **Lagringsutrymme** | 2 GB ledigt utrymme | SSD med 10 GB+ ledigt utrymme |
| **Python** | Python 3.7+ (för SDK) | Python 3.10+ |

### Installation

Ladda ner paketet `.deb` och installera:

```bash
sudo dpkg -i chloros-amd64.deb
```

Kontrollera installationen:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Systemkrav

| Krav | Minsta | Rekommenderat |
| --- | --- | --- |
| **Plattform** | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **JetPack** | JetPack 6.x | Senaste JetPack 6 |
| **Minne (RAM)** | 8 GB (delat GPU/CPU) | 16 GB+ delat |
| **Lagring** | 2 GB ledigt utrymme | NVMe SSD med 10 GB+ ledigt |
| **Python** | Python 3.7+ (för SDK) | Python 3.10+ |

### Installation

Ladda ner JetPack 6 `.deb`-paketet och installera:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Kontrollera installationen:

```bash
chloros-cli --version
```

För detaljerad information om Jetson-konfiguration, inklusive värmehantering och fältinstallation, se [NVIDIA Jetson Guide](nvidia-jetson-guide.md).

***

## Python SDK Installation (All Linux)

Python SDK installeras separat via pip och fungerar på både amd64 och arm64:

```bash
pip install chloros-sdk
```

För att inkludera valfritt stöd för strömning av förlopp:

```bash
pip install chloros-sdk[progress]
```

Verifiera SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Paketet `.deb` installerar Chloros CLI och backend. Python SDK är ett separat pip-paket som kommunicerar med backend via en lokal HTTP API.
{% endhint %}

***

## Konfigurationskataloger

Chloros på Linux följer [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Syfte | Linux Sökväg | Windows Motsvarighet |
| --- | --- | --- |
| **Konfiguration** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Data / Projekt** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / Inloggningsuppgifter** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Platser för backend-program

Paketet `.deb` installerar backend på en standardplats. CLI och SDK upptäcker automatiskt backend-sökvägen:

| Installationsmetod | Backend-sökväg |
| --- | --- |
| `.deb`-paketet | `/usr/lib/chloros/chloros-backend` |
| Manuell / anpassad | `/opt/mapir/chloros/backend/chloros-backend` |

Du kan åsidosätta backend-sökvägen med flaggan `--backend-exe` CLI eller konstruktionsparametern `backend_exe` SDK.

***

## Första installation

### 1. Aktivera din licens

En Chloros+-licens krävs för åtkomst till CLI och SDK:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Kontrollera din licensstatus

```bash
chloros-cli status
```

### 3. Bearbeta din första dataset

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Kör systemdiagnostik

Kontrollera att ditt system är korrekt konfigurerat:

```bash
chloros-cli selftest
```

Detta kör 7 diagnostiska kontroller, inklusive version, uppstart av backend, API-anslutning och CUDA/GPU-tillgänglighet.

***

## Exempel på Bash-skript

### Bearbeta flera dataset

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Bearbeta med anpassade inställningar

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Automatiserad bearbetning med Cron

Lägg till i din crontab (`crontab -e`) för att bearbeta nya datamängder automatiskt:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Exempel

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Felsökning

### CLI hittas inte efter installation

Om `chloros-cli` inte hittas efter installation av `.deb`-paketet:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Åtkomst nekad

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Backend kunde inte startas

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA upptäcktes inte

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Saknade delade bibliotek

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Uppdatering av Chloros på Linux

Använd det inbyggda uppdateringskommandot för att söka efter och installera uppdateringar:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Nästa steg

* [NVIDIA Jetson-guide](nvidia-jetson-guide.md) — Jetson-specifik optimering och distribution
* [CLI : Kommandorad](../CLI.md) — Fullständig CLI-kommandoreferens
* [API : Python SDK](../api-python-sdk.md) — Fullständig SDK-referens
* [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md) — Hur Chloros anpassar sig till din hårdvara
