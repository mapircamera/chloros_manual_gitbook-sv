# CLI : Kommandorad

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** ger kraftfull kommandoradsåtkomst till bildbehandlingsmotorn Chloros, vilket möjliggör automatisering, skriptning och headless-drift för dina bildbehandlingsarbetsflöden.

### Viktiga funktioner

* 🚀 **Automatisering** – Skriptbaserad batchbearbetning av flera datamängder
* 🔗 **Integration** – Integrera i befintliga arbetsflöden och pipelines
* 💻 **Headless-drift** – Kör utan GUI
* 🌍 **Flerspråkig** – Stöd för 38 språk
* ⚡ **Parallell bearbetning** – [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) optimerar automatiskt för din hårdvara

### Krav

| Krav          | Detaljer                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operativsystem** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licens**          | Chloros+ ([betald plan krävs](https://cloud.mapir.camera/pricing)) |
| **Minne**           | Minst 8 GB RAM (16 GB rekommenderas)                                  |
| **Internet**         | Krävs för licensaktivering                                     |
| **Diskutrymme**       | Varierar beroende på projektstorlek                                              |

{% hint style="warning" %}
**Licenskrav**: CLI kräver ett betalt Chloros+-abonnemang. Standardabonnemang (gratis) har inte tillgång till CLI. Besök [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) för att uppgradera.
{% endhint %}

## Snabbstart

### Installation

#### Windows

CLI ingår automatiskt i installationsprogrammet för Chloros:

1. Ladda ner och kör **Chloros Installer.exe**

2. Följ installationsguiden
3. CLI installeras i: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Installationsprogrammet lägger automatiskt till `chloros-cli` i systemets PATH. Starta om terminalen efter installationen.
{% endhint %}

#### Linux

Installera paketet `.deb` för din arkitektur:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

För detaljerad information om Linux-installationen, se [Linux Installation](linux/linux-installation.md).

### Första installation

Innan du använder CLI måste du aktivera din Chloros+-licens:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Grundläggande användning

Bearbeta en mapp med standardinställningar:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Kommandoreferens

### Allmän syntax

```
chloros-cli [global-options] <command> [command-options]
```

***

## Kommandon

### `process` - Bearbeta bilder

Bearbeta bilder i en mapp med kalibrering.

**Syntax:**

```bash
chloros-cli process <input-folder> [options]
```

**Exempel:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Alternativ för kommandot Process

| Alternativ                | Typ    | Standard        | Beskrivning                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Sökväg    | _Obligatoriskt_     | Mapp som innehåller RAW/JPG-multispektralbilder                                         |
| `-o, --output`        | Sökväg    | Samma som ingång  | Utdatamapp för bearbetade bilder                                                     |
| `-n, --project-name`  | Sträng  | Autogenererad | Anpassat projektnamn                                                                    |
| `--vignette`          | Flagga    | Aktiverad        | Aktivera vignettkorrigering                                                             |
| `--no-vignette`       | Flagga    | -              | Inaktivera vignettkorrigering                                                            |
| `--reflectance`       | Flagga    | Aktiverad        | Aktivera reflektanskalibrering                                                         |
| `--no-reflectance`    | Flagga    | -              | Inaktivera reflektanskalibrering                                                        |
| `--ppk`               | Flagga    | Inaktiverad       | Tillämpa PPK-korrigeringar från .daq-ljussensordata                                      |
| `--format`            | Val  | TIFF (16-bit)  | Utdataformat: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Heltal | Auto           | Minsta målstorlek i pixlar för detektering av kalibreringspanelen                          |
| `--target-clustering` | Heltal | Auto           | Tröskelvärde för målkluster (0–100)                                                    |
| `--debayer`           | Val  | `standard`     | Debayer-metod: `standard` eller `texture-aware` (endast Chloros+)                          |
| `--target`, `--targets` | Flagga  | Inaktiverad       | Sök endast efter kalibreringsmål i en undermapp med namnet ”target” eller ”targets” (påskyndar bearbetningen) |
| `--indices`           | Lista    | Ingen           | Vegetationsindex att beräkna (t.ex. `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Sträng  | Ingen           | Lås exponering för kameramodell (Pin 1)                                                 |
| `--exposure-pin-2`    | Sträng  | Ingen           | Lås exponering för kameramodell (stift 2)                                                 |
| `--recal-interval`    | Heltal | Auto           | Omkalibreringsintervall i sekunder                                                      |
| `--timezone-offset`   | Heltal | 0              | Tidszonsavvikelse i timmar                                                               |

***

### `login` - Autentisera konto

Logga in med dina Chloros+-inloggningsuppgifter för att aktivera CLI-bearbetning.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

**Exempel:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Specialtecken**: Använd enkla citattecken runt lösenord som innehåller tecken som `$`, `!` eller mellanslag.
{% endhint %}

**Utdata:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Rensa inloggningsuppgifter

Rensa lagrade inloggningsuppgifter och logga ut från ditt konto.

**Syntax:**

```bash
chloros-cli logout
```

**Exempel:**

```bash
chloros-cli logout
```

**Utdata:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK-användare**: Python SDK tillhandahåller även en programmatisk `logout()`-metod för att rensa inloggningsuppgifter inom Python-skript. Se [Python SDK dokumentationen](api-python-sdk.md#logout) för mer information.
{% endhint %}

***

### `status` – Kontrollera licensstatus

Visar aktuell licens- och autentiseringsstatus.

**Syntax:**

```bash
chloros-cli status
```

**Exempel:**

```bash
chloros-cli status
```

**Utdata:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` – Kontrollera exportförloppet

Övervaka exportförloppet för tråd 4 under eller efter bearbetningen.

**Syntax:**

```bash
chloros-cli export-status
```

**Exempel:**

```bash
chloros-cli export-status
```

**Användningsfall:** Använd detta kommando medan bearbetningen pågår för att kontrollera exportförloppet.***

### `language` – Hantera gränssnittsspråk

Visa eller ändra gränssnittsspråket för CLI.

**Syntax:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Exempel:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Språk som stöds (totalt 38)

| Kod    | Språk              | Namn på originalspråket      |
| ------- | --------------------- | ---------------- |
| `en`    | Engelska               | English          |
| `es`    | Spanska               | Español          |
| `pt`    | Portugisiska            | Português        |
| `fr`    | Franska                | Français         |
| `de`    | Tyska                | Deutsch          |
| `it`    | Italienska              | Italiano         |
| `ja`    | Japanska              | 日本語              |
| `ko`    | Koreanska                | 한국어              |
| `zh`    | Kinesiska (förenklad)  | 简体中文             |
| `zh-TW` | Kinesiska (traditionell) | 繁體中文             |
| `ru`    | Ryska               | Русский          |
| `nl`    | Nederländska                | Nederlands       |
| `ar`    | Arabiska                | العربية          |
| `pl`    | Polska                | Polski           |
| `tr`    | Turkiska               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonesiska            | Bahasa Indonesia |
| `vi`    | Vietnamesiska            | Tiếng Việt       |
| `th`    | Thailändska                  | ไทย              |
| `sv`    | Svenska               | Svenska          |
| `da`    | Danska                | Dansk            |
| `no`    | Norska             | Norsk            |
| `fi`    | Finska               | Suomi            |
| `el`    | Grekiska                 | Ελληνικά         |
| `cs`    | Tjeckiska                | Čeština          |
| `hu`    | Ungerska             | Magyar           |
| `ro`    | Rumänska              | Română           |
| `uk`    | Ukrainska             | Українська       |
| `pt-BR` | Brasiliansk portugisiska  | Português Brasileiro |
| `zh-HK` | Kantonesiska             | 粵語             |
| `ms`    | Malajiska                 | Bahasa Melayu    |
| `sk`    | Slovakiska                | Slovenčina       |
| `bg`    | Bulgariska             | Български        |
| `hr`    | Kroatiska              | Hrvatski         |
| `lt`    | Litauiska            | Lietuvių         |
| `lv`    | Lettiska               | Latviešu         |
| `et`    | Estniska              | Eesti            |
| `sl`    | Slovensk             | Slovenščina      |

{% hint style="success" %}
**Automatisk lagring**: Ditt språkval sparas i `~/.chloros/cli_language.json` och behålls under alla sessioner.
{% endhint %}

***

### `set-project-folder` - Ställ in standardprojektmapp

Ändra platsen för standardprojektmappen (delas med GUI på Windows).

**Syntax:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Exempel:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Visa projektmapp

Visa den aktuella standardprojektmappens plats.

**Syntax:**

```bash
chloros-cli get-project-folder
```

**Exempel:**

```bash
chloros-cli get-project-folder
```

**Utdata:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` – Återställ till standard

Återställ projektmappen till standardplatsen.

**Syntax:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` – Kör systemdiagnostik

Kör 7 diagnostiska kontroller för att verifiera din systemkonfiguration.

**Syntax:**

```bash
chloros-cli selftest
```

**Utförda diagnostiska kontroller:**

1. Versionskontroll
2. Porttillgänglighet (5000)
3. Start av backend
4. API-anslutningstest
5. Systeminformation och GPU-detektering
6. Verifiering av brusreduceringsmodeller
7. Kontroll av CUDA-tillgänglighet

{% hint style="info" %}
**Användbart för felsökning**: Kör `selftest` efter installationen för att verifiera att ditt system är korrekt konfigurerat, särskilt på Linux/Jetson där GPU- och CUDA-inställningarna kan behöva verifieras.
{% endhint %}

***

### `update` – Sök efter uppdateringar (endast Linux)

Sök efter och installera CLI-uppdateringar på Linux-system.

**Syntax:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Alternativ    | Beskrivning                        |
| --------- | ---------------------------------- |
| `--check` | Sök endast efter uppdateringar, installera inte |

{% hint style="info" %}
Detta kommando är endast tillgängligt på Linux. På Windows levereras uppdateringar via installationsprogrammet.
{% endhint %}

***

## Globala alternativ

Dessa alternativ gäller för alla kommandon:

| Alternativ            | Typ    | Standard       | Beskrivning                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Sökväg    | Automatiskt detekterad | Sökväg till backend-körbar fil                       |
| `--port`          | Heltal | 5000          | Backend API portnummer                          |
| `--restart`       | Flagga    | -             | Tvinga omstart av backend (avslutar befintliga processer) |
| `--version`       | Flagga    | -             | Visa versionsinformation och avsluta                |
| `--help`          | Flagga    | -             | Visa hjälpinformation och avsluta                   |

{% hint style="info" %}
**Automatisk upptäckt av backend**: Sökvägen `--backend-exe` upptäcks automatiskt per plattform:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuell)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Exempel med globala alternativ:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Guide till bearbetningsinställningar

### Parallellbearbetning och dynamisk beräkningsanpassning

Chloros 1.1.0 inkluderar [Dynamisk beräkningsanpassning](processing-architecture/dynamic-compute-adaptation.md) — bearbetningsmotorn **upptäcker automatiskt din hårdvara** och väljer den optimala strategin:

| Plattform | Strategi | Arbetare | Pipeline | Anmärkningar |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Minneseffektiv, serialiserad |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Samtidig GPU-bearbetning |
| **Stationär dator med 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Bra prestanda för stationära datorer |
| **Stationär dator med 12 GB+ GPU** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimal stationär prestanda |
| **System med endast CPU** | `CPU_PARALLEL` | kärnor – 1 | `cpu_fallback` | Inget grafikkort krävs |

{% hint style="success" %}
**Ingen manuell konfiguration behövs!** Chloros upptäcker automatiskt din CPU, GPU, RAM och (på Jetson) temperatursensorer, och konfigurerar sedan den optimala bearbetningspipeline automatiskt.
{% endhint %}

### Debayer-metoder

| Metod | CLI-flagga | Kvalitet | Hastighet | Licens |
| --- | --- | --- | --- | --- |
| **Standard (Snabb, medelhög kvalitet)** | `--debayer standard` | Bra | Snabb | Gratis / Chloros+ |
| **Texturmedveten (Långsam, högsta kvalitet)** | `--debayer texture-aware` | Högsta | Långsam | Endast Chloros+ |

Standardmetoden för debayering är **Standard**. Metoden**Texturmedveten** använder en AI/ML-brusreduceringsmodell för utdata av högsta kvalitet, men kräver en Chloros+-licens och ett NVIDIA-grafikkort.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Vignettkorrigering

**Vad det gör:** Korrigerar ljusavfall vid bildkanterna (mörkare hörn som är vanliga i kamerabilder).

* **Aktiverat som standard** – De flesta användare bör ha detta aktiverat
* Använd `--no-vignette` för att inaktivera

{% hint style="success" %}
**Rekommendation**: Aktivera alltid vignettkorrigering för att säkerställa enhetlig ljusstyrka över hela bilden.
{% endhint %}

### Reflektanskalibrering

Konverterar råa sensorvärden till standardiserade reflektansprocenttal med hjälp av kalibreringspaneler.

* **Aktiverat som standard** – Nödvändigt för vegetationsanalys
* Kräver kalibreringspaneler i bilderna
* Använd `--no-reflectance` för att inaktivera

{% hint style="info" %}
**Krav**: Se till att kalibreringspanelerna är korrekt exponerade och synliga i dina bilder för korrekt reflektanskonvertering.
{% endhint %}

### PPK-korrigeringar

**Vad det gör:** Tillämpar post-processerade kinematiska korrigeringar med hjälp av DAQ-A-SD-loggdata för förbättrad GPS-noggrannhet.

* **Inaktiverat som standard**
* Använd `--ppk` för att aktivera
* Kräver .daq-filer i projektmappen från MAPIR DAQ-A-SD-ljussensor.

### Utdataformat

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bittjocklek</th><th width="116.5999755859375">Filstorlek</th><th>Bäst för</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>16-bitars heltal</td><td>Stor</td><td>GIS-analys, fotogrammetri (rekommenderas)</td></tr><tr><td><strong>TIFF (32-bitars, procent)</strong></td><td>32-bitars flyttal</td><td>Mycket stor</td><td>Vetenskaplig analys, forskning</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bitars heltal</td><td>Medel</td><td>Visuell inspektion, delning på webben</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-bitars heltal</td><td>Liten</td><td>Snabb förhandsgranskning, komprimerad utdata</td></tr></tbody></table>***

## Automatisering och skriptning

### PowerShell-batchbearbetning (Windows)

Bearbeta flera datasetmappar automatiskt på Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows Batchskript (Windows)

Enkel loop för batchbearbetning på Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Bash-batchbearbetning (Linux)

Bearbeta flera datasetmappar på Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Python automatiseringsskript (plattformsoberoende)

Avancerad automatisering med felhantering (fungerar på Windows och Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Bearbetningsflöde

### Standardflöde

1. **Indata**: Mapp som innehåller RAW/JPG-bildpar
2. **Upptäckt**: CLI söker automatiskt efter bildfiler som stöds
3. **Bearbetning**: Parallellt läge skalar efter dina CPU-kärnor (Chloros+)
4. **Utdata**: Skapar undermappar för kameramodeller med bearbetade bilder

### Exempel på utdatastruktur

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Uppskattad bearbetningstid

Typisk bearbetningstid för 100 bilder (12 MP vardera):

| Plattform | Läge | Uppskattad tid | Anmärkningar |
| --- | --- | --- | --- |
| **Stationär dator med 12 GB+ GPU** | `GPU_PARALLEL` | 5–10 min | Snabbaste alternativet |
| **Stationär dator med 8 GB GPU** | `GPU_SINGLE` | 10–15 min | Bra prestanda |
| **Jetson Orin 16 GB** | `GPU_PARALLEL` | 15–25 min | Edge-beräkning |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30–60 min | Begränsat minne |
| **Endast CPU** | `CPU_PARALLEL` | 20–40 min | Ingen GPU krävs |

{% hint style="info" %}
**Prestandatips**: Bearbetningstiden varierar beroende på antal bilder, upplösning, debayer-metod och hårdvara. Texture Aware-debayer tar betydligt längre tid än Standard. Se [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) för mer information.
{% endhint %}

***

## Felsökning

### CLI hittades inte

**Windows-fel:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Lösningar:**

1. Kontrollera installationsplatsen:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Använd fullständig sökväg om den inte finns i PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Lägg till i PATH manuellt:
   * Öppna Systemegenskaper → Miljövariabler
   * Redigera variabeln PATH
   * Lägg till: `C:\Program Files\Chloros\resources\cli`
   * Starta om terminalen

**Linux Fel:**

```
chloros-cli: command not found
```

**Linux Lösningar:**

1. Kontrollera installationen:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Ladda om ditt skal:

```bash
source ~/.bashrc
```

3. Kontrollera behörigheter:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Backend kunde inte startas**Fel:**

```

Backend failed to start within 30 seconds
```

**Lösningar:**

1. Kontrollera om backend redan körs (stäng den först)
2. Kontrollera att brandväggen inte blockerar (Windows) eller kontrollera porttillgänglighet (Linux: `lsof -i :5000`)
3. Prova en annan port:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Tvinga omstart av backend:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. På Linux, kontrollera att backend-programmet finns:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Problem med licens/autentisering**Fel:**

```

Chloros+ license required for CLI access
```

**Lösningar:**

1. Kontrollera att du har en aktiv Chloros+-prenumeration
2. Logga in med dina inloggningsuppgifter:

```bash
chloros-cli login user@example.com 'password'
```

3. Kontrollera licensstatus:

```bash
chloros-cli status
```

4. Kontakta supporten: info@mapir.camera

***

### Inga bilder hittades**Fel:**

```

No images found in the specified folder
```

**Lösningar:**

1. Kontrollera att mappen innehåller format som stöds (.RAW, .TIF, .JPG)
2. Kontrollera att mappvägen är korrekt (använd citattecken för sökvägar med mellanslag)
3. Se till att du har läsbehörighet för mappen
4. Kontrollera att filändelserna är korrekta

***

### Bearbetningen stannar upp eller hänger sig**Lösningar:**

1. Kontrollera tillgängligt diskutrymme (se till att det finns tillräckligt för utdata)
2. Stäng andra program för att frigöra minne
3. Minska antalet bilder (bearbeta i omgångar)

***

### Porten används redan**Fel:**

```

Port 5000 is already in use
```

**Lösningar:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Vanliga frågor

### F: Behöver jag en licens för CLI?

**S:**Ja! CLI kräver en betald**Chloros+-licens**.

* ❌ Standardplan (gratis): CLI inaktiverat
* ✅ Chloros+ (betald) plan: CLI fullt aktiverat

Prenumerera på: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### F: Kan jag använda CLI på en server utan GUI?**S:** Ja! CLI körs helt utan grafiskt gränssnitt. Detta är det primära användningsfallet på Linux.**Windows-server:**
* Windows Server 2016 eller senare
* Visual C++ Redistributable installerat

**Linux Server:**
* Ubuntu 20.04+ / Debian 11+ (amd64) eller JetPack 6 (arm64)
* Installera via `.deb`-paketet

**Båda plattformarna:**
* Minst 8 GB RAM (16 GB rekommenderas)
* Engångsaktivering av licens: `chloros-cli login user@example.com 'password'`

***

### F: Var sparas bearbetade bilder?**S:**Som standard sparas bearbetade bilder i**samma mapp som ingångsfilerna** i undermappar för kameramodeller (t.ex. `Survey3N_RGN/`).

Använd alternativet `-o` för att ange en annan utmatningsmapp:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### F: Kan jag bearbeta flera mappar samtidigt?**S:** Inte direkt med ett enda kommando, men du kan använda skript för att bearbeta mappar i tur och ordning. Se avsnittet [Automatisering och skript](CLI.md#automation--scripting).***

### F: Hur sparar jag CLI-utdata till en loggfil?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### F: Vad händer om jag trycker på Ctrl+C under bearbetningen?**S:** CLI kommer att:

1. Avsluta bearbetningen på ett korrekt sätt
2. Stänga av backend
3. Avsluta med kod 130

Delvis bearbetade bilder kan finnas kvar i utdatamappen.

***

### F: Kan jag automatisera CLI-bearbetningen?**Svar:** Absolut! CLI är utformat för automatisering. Se [Automatisering och skriptning](CLI.md#automation--scripting) för exempel på PowerShell (Windows), Batch (Windows), Bash (Linux) och Python (plattformsoberoende) exempel.***

### F: Hur kontrollerar jag versionen av CLI?**S:**

```bash
chloros-cli --version
```

**Utdata:**

```

Chloros CLI 1.1.0
```

***

## Få hjälp

### Hjälp via kommandoraden

Visa hjälpinformation direkt i CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Supportkanaler

* **E-post**: info@mapir.camera
* **Webbplats**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Priser**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Kompletta exempel

### Exempel 1: Grundläggande bearbetning

Bearbetning med standardinställningar (vignett, reflektans):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Exempel 2: Vetenskaplig utdata av hög kvalitet

32-bitars flyttal TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Exempel 3: Snabb förhandsgranskning

8-bitars PNG utan kalibrering för snabb granskning:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Exempel 4: PPK-korrigerad bearbetning

Tillämpa PPK-korrigeringar med reflektans:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Exempel 5: Anpassad utmatningsplats

Bearbeta till en annan plats med specifikt format:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Exempel 6: Autentiseringsflöde

Komplett autentiseringsflöde (samma på alla plattformar):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Exempel 7: Användning av flera språk

Ändra gränssnittsspråk (samma på alla plattformar):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
