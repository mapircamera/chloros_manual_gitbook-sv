# API : Python SDK

**Chloros Python SDK** ger programmatisk åtkomst till bildbehandlingsmotorn Chloros, vilket möjliggör automatisering, anpassade arbetsflöden och sömlös integration med dina Python-applikationer och forskningspipelines.

### Viktiga funktioner

* 🐍 **Native Python** – Ren, Pythonic API för bildbehandling
* 🔧 **Fullständig API-åtkomst** – Fullständig kontroll över Chloros-bearbetning
* 🚀 **Automatisering** – Skapa anpassade arbetsflöden för batchbearbetning
* 🔗 **Integration** – Bädda in Chloros i befintliga Python-applikationer
* 📊 **Forskningsklar** – Perfekt för vetenskapliga analyspipelines
* ⚡ **Parallellbearbetning** – Skalar efter dina CPU-kärnor (Chloros+)

### Krav

| Krav          | Detaljer                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros installerat** | Windows: Installationsprogram för stationära datorer; Linux: `.deb`-paket                  |
| **Licens**          | Chloros+ ([betalplan krävs](https://cloud.mapir.camera/pricing)) |
| **Operativsystem** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 eller högre                                                |
| **Minne**           | Minst 8 GB RAM (16 GB rekommenderas)                                  |
| **Internet**         | Krävs för licensaktivering                                     |

{% hint style="warning" %}
**Licenskrav**: Python SDK kräver ett betalt Chloros+-abonnemang för åtkomst till API. Standardplaner (gratis) har inte tillgång till API/SDK. Besök [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) för att uppgradera.
{% endhint %}

## Snabbstart

### Installation

Installera via pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Första inställningen**: Innan du använder SDK, aktivera din Chloros+-licens genom att öppna Chloros, Chloros (webbläsare) eller Chloros CLI och logga in med dina inloggningsuppgifter. Detta behöver endast göras en gång. På Linux (utan GUI), använd: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Grundläggande användning

Bearbeta en mapp med bara några rader:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Plattformsoberoende sökvägar**: Kodexemplen på denna sida använder sökvägar i stil med Windows (t.ex. `C:\\DroneImages\\Flight001`). På Linux använder du istället Linux-sökvägar (t.ex. `/home/user/drone_images/flight001` eller `~/drone_images/flight001`). SDK fungerar på samma sätt på båda plattformarna.
{% endhint %}

### Full kontroll

För avancerade arbetsflöden:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

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

1. **Chloros installerat** — Windows: Installationsprogram för skrivbordet ([nedladdning](download.md)); Linux: `.deb`-paket ([Linux Installation](linux/linux-installation.md))
2. **Python 3.7+** installerat ([python.org](https://www.python.org))
3. **Aktiv Chloros+ licens** ([uppgradering](https://cloud.mapir.camera/pricing))

### Installera via pip

**Standardinstallation:**

```bash
pip install chloros-sdk
```

**Med stöd för övervakning av installationsförloppet:**

```bash
pip install chloros-sdk[progress]
```

**Utvecklingsinstallation:**

```bash
pip install chloros-sdk[dev]
```

### Verifiera installationen

Testa att SDK är korrekt installerat:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Första inställningen

### Licensaktivering

SDK använder samma licens som Chloros, Chloros (webbläsare) och Chloros CLI. Aktivera en gång via GUI eller CLI:**Windows:**Öppna**Chloros eller Chloros (webbläsare)** och logga in på fliken Användare <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> eller använd CLI.**Linux:** Använd CLI (inget GUI tillgängligt):

```bash
chloros-cli login user@example.com 'your_password'
```

Licensen cachelagras lokalt och kvarstår vid omstart.

{% hint style="success" %}
**Engångsinstallation**: Efter inloggning via GUI eller CLI använder SDK automatiskt den cachade licensen. Ingen ytterligare autentisering behövs!
{% endhint %}

{% hint style="info" %}
**Utloggning**: SDK-användare kan programmatiskt rensa cachade inloggningsuppgifter med hjälp av metoden `logout()`. Se [logout()-metoden](#logout) i API-referensen.
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

## API-referens

### Klassen ChlorosLocal

Huvudklass för lokal bildbehandling med Chloros.

#### Konstruktor

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

| Parameter                 | Typ | Standardvärde                   | Beskrivning                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL för lokal Chloros-backend          |
| `auto_start_backend`      | bool | `True`                    | Starta backend automatiskt vid behov |
| `backend_exe`             | str  | `None` (auto-detect)      | Sökväg till backend-körbar fil            |
| `timeout`                 | int  | `30`                      | Timeout för begäran i sekunder            |
| `backend_startup_timeout` | int  | `60`                      | Timeout för start av backend (sekunder) |

**Exempel:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Plattformsoberoende automatisk detektering**: SDK försöker automatiskt hitta rätt backend-sökväg för din plattform:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuell)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Metoder

#### `create_project(project_name, camera=None)`

Skapa ett nytt Chloros-projekt.

**Parametrar:**

| Parameter      | Typ | Obligatorisk | Beskrivning                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Ja      | Namn på projektet                                     |
| `camera`       | str  | Nej       | Kameramall (t.ex. &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**Returnerar:** `dict` - Svar på projektskapande**Exempel:**

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

**Returnerar:** `dict` - Importresultat med antal filer**Exempel:**

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
| `debayer`                 | str  | &quot;Standard (Snabb, medelhög kvalitet)&quot; | Debayer-metod            |
| `vignette_correction`     | bool | `True`                  | Aktivera vignettkorrigering      |
| `reflectance_calibration` | bool | `True`                  | Aktivera reflektanskalibrering  |
| `indices`                 | list | `None`                  | Vegetationsindex att beräkna |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | Utdataformat                   |
| `ppk`                     | bool | `False`                 | Aktivera PPK-korrigeringar          |
| `custom_settings`         | dict | `None`                  | Avancerade anpassade inställningar        |

**Exportformat:**

* `"TIFF (16-bit)"` - Rekommenderas för GIS/fotogrammetri
* `"TIFF (32-bit, Percent)"` - Vetenskaplig analys
* `"PNG (8-bit)"` - Visuell inspektion
* `"JPG (8-bit)"` - Komprimerad utdata

**Tillgängliga index:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 och fler.**Exempel:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
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

| Parameter           | Typ     | Standardvärde      | Beskrivning                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Bearbetningsläge: &quot;parallel&quot; eller &quot;serial&quot;   |
| `wait`              | bool     | `True`       | Vänta på slutförande                       |
| `progress_callback` | callable | `None`       | Återkopplingsfunktion för framsteg (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Pollingintervall för framsteg (sekunder)   |

**Returnerar:** `dict` - Bearbetningsresultat

{% hint style="warning" %}
**Parallellt läge**: Kräver Chloros+ licens. Skalar automatiskt till dina CPU-kärnor (upp till 16 arbetare).
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

**Returnerar:** `dict` - Aktuell projektkonfiguration**Exempel:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Hämta statusinformation för backend, inklusive bearbetningsförlopp per tråd.

**Returnerar:** `dict` - Backend-status med följande struktur:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Exempel:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Stäng av backend (om den startats av SDK).

**Exempel:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Rensa cachade autentiseringsuppgifter från det lokala systemet.

**Beskrivning:**

Loggar ut programmatiskt genom att ta bort cachade autentiseringsuppgifter. Detta är användbart för:
* Att växla mellan olika Chloros+-konton
* Att rensa autentiseringsuppgifter i automatiserade miljöer
* Säkerhetsändamål (t.ex. att ta bort autentiseringsuppgifter före avinstallation)

**Returvärde:** `dict` – Resultat av utloggningsoperationen**Exempel:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Omautentisering krävs**: Efter att ha anropat `logout()` måste du logga in igen via Chloros, Chloros (webbläsare) eller Chloros CLI innan du använder SDK.
{% endhint %}

***

### Bekvämlighetsfunktioner

#### `process_folder(folder_path, **options)`

Enradig bekvämlighetsfunktion för att bearbeta en mapp.

**Parametrar:**

| Parameter                 | Typ     | Standard         | Beskrivning                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Obligatorisk        | Sökväg till mapp med bilder     |
| `project_name`            | str      | Autogenererad  | Projektnamn                   |
| `camera`                  | str      | `None`          | Kameramall                |
| `indices`                 | lista     | `["NDVI"]`      | Index att beräkna           |
| `vignette_correction`     | bool     | `True`          | Aktivera vignettkorrigering     |
| `reflectance_calibration` | bool     | `True`          | Aktivera reflektanskalibrering |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Utdataformat                  |
| `mode`                    | str      | `"parallel"`    | Bearbetningsläge                |
| `progress_callback`       | callable | `None`          | Återkoppling om framsteg              |

**Returnerar:** `dict` - Bearbetningsresultat**Exempel:**

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

{% hint style="info" %}
**Linux-användare**: Alla exempel nedan använder Windows-sökvägar. Ersätt `C:\\...`-sökvägar med dina Linux-sökvägar (t.ex. `/home/user/...` eller `~/...`). Alla SDK-funktioner är identiska på alla plattformar.
{% endhint %}

### Exempel 1: Grundläggande bearbetning

Bearbeta en mapp med standardinställningar:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Exempel 2: Anpassad arbetsflöde

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
    debayer="Standard (Fast, Medium Quality)",
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

### Exempel 4: Integration i forskningspipeline

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

### Exempel 5: Anpassad förloppsövervakning

Avancerad förloppsövervakning med loggning:

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
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
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

### Exempel 7: Kontohantering och utloggning

Hantera inloggningsuppgifter programmatiskt:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Exempel 8: Kommandoradsverktyg

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
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
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
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Hantering av undantag

SDK tillhandahåller specifika undantagsklasser för olika felkategorier:

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
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

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

### Minneshantering

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

**Problem:** SDK kan inte starta backend**Lösningar:**

1. Kontrollera att Chloros är installerat:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Kontrollera brandväggen (Windows) eller porttillgängligheten (Linux: `lsof -i :5000`)
3. Prova manuell backend-sökväg:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Licens upptäcktes inte**Problem:** SDK varnar om saknad licens**Lösningar:**

1. Öppna Chloros, Chloros (webbläsare) eller Chloros CLI och logga in.
2. Kontrollera att licensen finns i cacheminnet:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Om du har problem med inloggningsuppgifterna, rensa cacheminnet för inloggningsuppgifter och logga in på nytt:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Kontakta supporten: info@mapir.camera

***

### Importfel**Problem:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Lösningar:**

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

### Tidsgräns för bearbetning**Problem:** Tidsgränsen för bearbetning har löpt ut**Lösningar:**

1. Öka tidsgränsen:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Bearbeta mindre batcher
3. Kontrollera tillgängligt diskutrymme
4. Övervaka systemresurser

***

### Porten används redan**Problem:** Backend-port 5000 upptagen**Lösningar:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Eller hitta och stäng den process som orsakar konflikten:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## Prestandatips

### Optimera bearbetningshastigheten

1. **Använd parallellt läge** (kräver Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Minska utskriftsupplösningen** (om det är acceptabelt)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Inaktivera onödiga index**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Bearbeta på SSD** (inte HDD)***

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

**S:** Endast för den första licensaktiveringen. Efter inloggning via Chloros, Chloros (webbläsare) eller Chloros CLI sparas licensen lokalt och fungerar offline i 30 dagar.***

### F: Kan jag använda SDK på en server utan GUI?**S:** Ja! SDK fungerar utan grafiskt gränssnitt på både Windows- och Linux-servrar.**Linux (rekommenderas för headless):**
* Installera via `.deb`-paketet
* Aktivera licens: `chloros-cli login user@example.com 'password'`

**Windows-server:**
* Windows Server 2016 eller senare
* Chloros installerat (engångsinstallation)
* Licens aktiverad via CLI eller på valfri dator

***

### F: Vad är skillnaden mellan Desktop, CLI och SDK?

| Funktion         | Desktop GUI | CLI Kommandorad | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Gränssnitt**   | Pek-och-klicka | Kommando          | Python API  |
| **Bäst för**    | Visuellt arbete | Skriptning        | Integration |
| **Automatisering**  | Begränsad     | Bra             | Utmärkt   |
| **Flexibilitet** | Grundläggande       | Bra             | Maximal     |
| **Licens**     | Chloros+    | Chloros+         | Chloros+    |***

### F: Kan jag distribuera appar som är byggda med SDK?**S:** SDK-kod kan integreras i dina applikationer, men:

* Slutanvändare måste ha Chloros installerat
* Slutanvändare måste ha aktiva Chloros+-licenser
* Kommersiell distribution kräver OEM-licensiering

Kontakta info@mapir.camera för frågor om OEM.

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

### F: Kan jag bearbeta bilder från Python-skript som körs enligt schema?**S:** Ja! Använd ditt operativsystems schemaläggare med Python-skript:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Schemalägg via Task Scheduler för att köras dagligen.**Linux:** Schemalägg via cron:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### F: Stöder SDK async/await?**S:** Den nuvarande versionen är synkron. För asynkron funktion, använd `wait=False` eller kör i separat tråd:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### F: Hur växlar jag mellan olika Chloros+-konton?**S:** Använd metoden `logout()` för att rensa cachade inloggningsuppgifter och logga sedan in igen med det nya kontot:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Efter utloggning, autentisera med det nya kontot via GUI, webbläsare eller CLI innan du använder SDK igen.

***

## Få hjälp

### Dokumentation

* **API-referens**: Denna sida

### Supportkanaler

* **E-post**: info@mapir.camera
* **Webbplats**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Priser**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Exempelkod

Alla exempel som listas här är testade och redo för produktion. Kopiera och anpassa dem efter ditt användningsfall.

***

## Licens**Proprietär programvara** – Copyright (c) 2025 MAPIR Inc.

SDK kräver ett aktivt Chloros+-abonnemang. Obehörig användning, distribution eller modifiering är förbjuden.
