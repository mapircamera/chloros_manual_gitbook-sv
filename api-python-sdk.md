# API : Python SDK

**Chloros Python SDK** ger programmatisk åtkomst till bildbehandlingsmotorn Chloros, vilket möjliggör automatisering, anpassade arbetsflöden och sömlös integration med dina Python-applikationer och forskningspipelines.

### Viktiga funktioner

* 🐍 **Native Python** - Ren, Pythonic API för bildbehandling
* 🔧 **Fullständig API-åtkomst** - Fullständig kontroll över Chloros-bearbetning
* 🚀 **Automatisering** - Skapa anpassade arbetsflöden för batchbearbetning
* 🔗 **Integration** – Bädda in Chloros i befintliga Python-applikationer
* 📊 **Forskningsklar** – Perfekt för vetenskapliga analyspipelines
* ⚡ **Parallell bearbetning** – Skalar till dina CPU-kärnor (Chloros+)

### Krav

| Krav          | Detaljer                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Måste installeras lokalt                                           |
| **Licens**          | Chloros+ ([betald plan krävs](https://cloud.mapir.camera/pricing)) |
| **Operativsystem** | Windows 10/11 (64-bitars)                                              |
| **Python**           | Python 3.7 eller högre                                                |
| **Minne**           | Minst 8 GB RAM (16 GB rekommenderas)                                  |
| **Internet**         | Krävs för aktivering av licensen                                     |

{% hint style=&quot;warning&quot; %}
**Licenskrav**: Python SDK kräver ett betalt Chloros+-abonnemang för åtkomst till API. Standardabonnemang (gratis) har inte tillgång till API/SDK. Besök [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) för att uppgradera.
{% endhint %}

## Snabbstart

### Installation

Installera via pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Första installationen**: Innan du använder SDK måste du aktivera din Chloros+-licens genom att öppna Chloros, Chloros (webbläsare) eller Chloros CLI och logga in med dina inloggningsuppgifter. Detta behöver endast göras en gång.
{% endhint %}

### Grundläggande användning

Bearbeta en mapp med bara några rader:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Full kontroll

För avancerade arbetsflöden:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Installationsguide

### Förutsättningar

Innan du installerar SDK, se till att du har:

1. **Chloros Desktop** installerat ([nedladdning](download.md))
2. **Python 3.7+** installerat ([python.org](https://www.python.org))
3. **Aktiv Chloros+-licens** ([uppgradering](https://cloud.mapir.camera/pricing))

### Installera via pip

**Standardinstallation:**

```bash
pip install chloros-sdk
```

**Med stöd för övervakning av förloppet:**

```bash
pip install chloros-sdk[progress]
```

**Utvecklingsinstallation:**

```bash
pip install chloros-sdk[dev]
```

### Kontrollera installationen

Testa att SDK är korrekt installerat:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Första installationen

### Licensaktivering

SDK använder samma licens som Chloros, Chloros (webbläsare) och Chloros CLI. Aktivera en gång via GUI eller CLI:

1. Öppna **Chloros eller Chloros (webbläsare)** och logga in på fliken Användare <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Eller öppna **CLI**.
2. Ange dina Chloros+-inloggningsuppgifter och logga in
3. Licensen cachelagras lokalt (kvarstår efter omstart)

{% hint style=&quot;success&quot; %}
**Engångsinstallation**: Efter inloggning via GUI eller CLI använder SDK automatiskt den cachade licensen. Ingen ytterligare autentisering behövs!
{% endhint %}

### Testa anslutningen

Kontrollera att SDK kan ansluta till Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API Referens

### ChlorosLocal-klass

Huvudklass för lokal Chloros-bildbehandling.

#### Konstruktör

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parametrar:**

| Parameter                 | Typ | Standard                   | Beskrivning                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL av lokal Chloros-backend          |
| `auto_start_backend`      | bool | `True`                    | Starta backend automatiskt vid behov |
| `backend_exe`             | str  | `None` (automatisk detektering)      | Sökväg till backend-körbar fil            |
| `timeout`                 | int  | `30`                      | Begär tidsgräns i sekunder            |
| `backend_startup_timeout` | int  | `60`                      | Tidsgräns för start av backend (sekunder) |

**Exempel:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Metoder

#### `create_project(project_name, camera=None)`

Skapa ett nytt Chloros-projekt.

**Parametrar:**

| Parameter      | Typ | Obligatorisk | Beskrivning                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Ja      | Namn på projektet                                     |
| `camera`       | str  | Nej       | Kameramall (t.ex. &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**Returnerar:** `dict` – Svar på projektskapande

**Exempel:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Importera bilder från en mapp.

**Parametrar:**

| Parameter     | Typ     | Obligatorisk | Beskrivning                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Ja      | Sökväg till mapp med bilder         |
| `recursive`   | bool     | Nej       | Sök i undermappar (standard: False) |

**Returnerar:** `dict` – Importerade resultat med filantal

**Exempel:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Konfigurera bearbetningsinställningar.

**Parametrar:**

| Parameter                 | Typ | Standard                 | Beskrivning                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Hög kvalitet (snabbare)&quot; | Debayer-metod                  |
| `vignette_correction`     | bool | `True`                  | Aktivera vignettkorrigering      |
| `reflectance_calibration` | bool | `True`                  | Aktivera reflektanskalibrering  |
| `indices`                 | lista | `None`                  | Vegetationsindex att beräkna |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | Utdataformat                   |
| `ppk`                     | bool | `False`                 | Aktivera PPK-korrigeringar          |
| `custom_settings`         | dict | `None`                  | Avancerade anpassade inställningar        |

**Exportformat:**

* `"TIFF (16-bit)"` – Rekommenderas för GIS/fotogrammetri
* `"TIFF (32-bit, Percent)"` – Vetenskaplig analys
* `"PNG (8-bit)"` – Visuell inspektion
* `"JPG (8-bit)"` – Komprimerad utdata

**Tillgängliga index:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 och fler.

**Exempel:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Bearbeta projektbilderna.

**Parametrar:**

| Parameter           | Typ     | Standard      | Beskrivning                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Bearbetningsläge: &quot;parallel&quot; eller &quot;serial&quot;   |
| `wait`              | bool     | `True`       | Vänta på slutförande                       |
| `progress_callback` | callable | `None`       | Återkopplingsfunktion för framsteg (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Pollningsintervall för framsteg (sekunder)   |

**Returnerar:** `dict` - Bearbetningsresultat

{% hint style=&quot;warning&quot; %}
**Parallellt läge**: Kräver Chloros+-licens. Skalar automatiskt till dina CPU-kärnor (upp till 16 arbetare).
{% endhint %}

**Exempel:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Hämta aktuell projektkonfiguration.

**Returnerar:** `dict` – Aktuell projektkonfiguration

**Exempel:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Hämta information om backend-status.

**Returnerar:** `dict` – Backend-status

**Exempel:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Stäng av backend (om den startats av SDK).

**Exempel:**

```python
chloros.shutdown_backend()
```

***

### Praktiska funktioner

#### `process_folder(folder_path, **options)`

Enradig bekvämlighetsfunktion för att bearbeta en mapp.

**Parametrar:**

| Parameter                 | Typ     | Standard         | Beskrivning                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Obligatoriskt        | Sökväg till mapp med bilder     |
| `project_name`            | str      | Autogenererad  | Projektnamn                   |
| `camera`                  | str      | `None`          | Kameramall                |
| `indices`                 | list     | `["NDVI"]`      | Index att beräkna           |
| `vignette_correction`     | bool     | `True`          | Aktivera vignettkorrigering     |
| `reflectance_calibration` | bool     | `True`          | Aktivera reflektanskalibrering |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Utmatningsformat                  |
| `mode`                    | str      | `"parallel"`    | Bearbetningsläge                |
| `progress_callback`       | callable | `None`          | Återkoppling av förlopp              |

**Returnerar:** `dict` – Bearbetningsresultat

**Exempel:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Stöd för kontextmanager

SDK stöder kontextmanagers för automatisk rensning:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Kompletta exempel

### Exempel 1: Grundläggande bearbetning

Bearbeta en mapp med standardinställningar:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Exempel 2: Anpassat arbetsflöde

Full kontroll över bearbetningspipeline:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Exempel 3: Batchbearbetning av flera mappar

Bearbeta flera flygdatauppsättningar:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Exempel 4: Integration av forskningspipeline

Integrera Chloros med dataanalys:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Exempel 5: Anpassad övervakning av framsteg

Avancerad spårning av framsteg med loggning:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Exempel 6: Felhantering

Robust felhantering för produktionsanvändning:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Exempel 7: Kommandoradsverktyg

Skapa ett anpassat CLI-verktyg med SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    
    args = parser.parse_args()
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Användning:**

```bash
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Undantagshantering

SDK tillhandahåller specifika undantagsklasser för olika feltyper:

### Undantagshierarki

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Exempel på undantag

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Avancerade ämnen

### Anpassad backend-konfiguration

Använd en anpassad backend-plats eller konfiguration:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Icke-blockerande bearbetning

Starta bearbetningen och fortsätt med andra uppgifter:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Minnehantering

För stora datamängder, bearbeta i batcher:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Felsökning

### Backend startar inte

**Problem:** SDK kan inte starta backend

**Lösningar:**

1. Kontrollera att Chloros Desktop är installerat:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Kontrollera att Windows brandvägg inte blockerar
3. Prova manuell backend-sökväg:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Licens upptäcks inte

**Problem:** SDK varnar om saknad licens

**Lösningar:**

1. Öppna Chloros, Chloros (webbläsare) eller Chloros CLI och logga in.
2. Kontrollera att licensen finns i cacheminnet:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Kontakta supporten: info@mapir.camera

***

### Importfel

**Problem:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Lösningar:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Bearbetningstidsgräns

**Problem:** Bearbetningstiden löper ut

**Lösningar:**

1. Öka tidsgränsen:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Bearbeta mindre batchar
3. Kontrollera tillgängligt diskutrymme
4. Övervaka systemresurser

***

### Porten används redan

**Problem:** Backend-port 5000 upptagen

**Lösningar:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Eller hitta och stäng den process som orsakar konflikten:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Tips för prestanda

### Optimera bearbetningshastigheten

1. **Använd parallellt läge** (kräver Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Minska utgångsupplösningen** (om det är acceptabelt)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Inaktivera onödiga index**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Bearbeta på SSD** (inte HDD)

***

### Minneoptimering

För stora datamängder:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Bakgrundsbehandling

Frigör Python för andra uppgifter:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Integrations exempel

### Django-integration

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Vanliga frågor

### F: Kräver SDK en internetanslutning?

**S:** Endast för initial licensaktivering. Efter inloggning via Chloros, Chloros (webbläsare) eller Chloros CLI cachelagras licensen lokalt och fungerar offline i 30 dagar.

***

### F: Kan jag använda SDK på en server utan GUI?

**S:** Ja! Krav:

* Windows Server 2016 eller senare
* Chloros installerat (engångsinstallation)
* Licens aktiverad på valfri maskin (cachelagrad licens kopierad till servern)

***

### F: Vad är skillnaden mellan Desktop, CLI och SDK?

| Funktion         | Desktop GUI | CLI Kommandorad | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Gränssnitt**   | Pek-och-klicka | Kommando          | Python API  |
| **Bäst för**    | Visuellt arbete | Skriptning        | Integration |
| **Automatisering**  | Begränsad     | Bra             | Utmärkt   |
| **Flexibilitet** | Grundläggande       | Bra             | Maximal     |
| **Licens**     | Chloros+    | Chloros+         | Chloros+    |

***

### F: Kan jag distribuera appar som skapats med SDK?

**S:** SDK-kod kan integreras i dina applikationer, men:

* Slutanvändarna måste ha Chloros installerat
* Slutanvändarna måste ha aktiva Chloros+-licenser
* Kommersiell distribution kräver OEM-licensiering.

Kontakta info@mapir.camera för OEM-förfrågningar.

***

### F: Hur uppdaterar jag SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### F: Var sparas bearbetade bilder?

Som standard i projektvägen:

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### F: Kan jag bearbeta bilder från Python-skript som körs enligt schema?

**S:** Ja! Använd Windows Task Scheduler med Python-skript:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Schemalägg via Task Scheduler för att köra dagligen.

***

### F: Stöder SDK async/await?

**S:** Den aktuella versionen är synkron. För asynkron funktion, använd `wait=False` eller kör i separat tråd:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Få hjälp

### Dokumentation

* **API Referens**: Denna sida

### Supportkanaler

* **E-post**: info@mapir.camera
* **Webbplats**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Priser**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Exempelkod

Alla exempel som listas här är testade och produktionsklara. Kopiera och anpassa dem för ditt användningsfall.

***

## Licens

**Proprietär programvara** – Copyright (c) 2025 MAPIR Inc.

SDK kräver ett aktivt Chloros+-abonnemang. Obehörig användning, distribution eller modifiering är förbjuden.
