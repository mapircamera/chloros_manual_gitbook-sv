# CLI : Kommandorad

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** ger kraftfull kommandoradsåtkomst till bildbehandlingsmotorn Chloros, vilket möjliggör automatisering, skriptning och headless-drift för dina bildbehandlingsarbetsflöden.

### Viktiga funktioner

* 🚀 **Automatisering** – Skriptbaserad batchbearbetning av flera datamängder
* 🔗 **Integration** – Integrera i befintliga arbetsflöden och pipelines
* 💻 **Headless-drift** – Kör utan GUI
* 🌍 **Flera språk** – Stöd för 38 språk
* ⚡ **Parallell bearbetning** – Skalar dynamiskt till din CPU (upp till 16 parallella arbetare)

### Krav

| Krav          | Detaljer                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operativsystem** | Windows 10/11 (64-bit)                                              |
| **Licens**          | Chloros+ ([betald plan krävs](https://cloud.mapir.camera/pricing)) |
| **Minne**           | Minst 8 GB RAM (16 GB rekommenderas)                                  |
| **Internet**         | Krävs för licensaktivering                                     |
| **Diskutrymme**       | Varierar beroende på projektets storlek                                              |

{% hint style=&quot;warning&quot; %}
**Licenskrav**: CLI kräver ett betalt Chloros+-abonnemang. Standardplaner (gratis) har inte tillgång till CLI. Besök [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) för att uppgradera.
{% endhint %}

## Snabbstart

### Installation

CLI ingår automatiskt i installationsprogrammet Chloros:

1. Ladda ner och kör **Chloros Installer.exe**

2. Följ installationsguiden
3. CLI installeras till: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Installationsprogrammet lägger automatiskt till `chloros-cli` i systemets PATH. Starta om terminalen efter installationen.
{% endhint %}

### Första installationen

Innan du använder CLI måste du aktivera din Chloros+-licens:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Grundläggande användning

Bearbeta en mapp med standardinställningar:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Kommandoreferens

### Allmän syntax

```
chloros-cli [global-options] <command> [command-options]
```

***

## Kommandon

### `process` – Bearbeta bilder

Bearbeta bilder i en mapp med kalibrering.

**Syntax:**

```bash
chloros-cli process <input-folder> [options]
```

**Exempel:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Alternativ för bearbetningskommandon

| Alternativ                | Typ    | Standard        | Beskrivning                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Sökväg    | _Krävs_     | Mapp som innehåller multispektrala RAW/JPG-bilder                                         |
| `-o, --output`        | Sökväg    | Samma som indata  | Utdatamapp för bearbetade bilder                                                     |
| `-n, --project-name`  | Sträng  | Autogenererad | Anpassat projektnamn                                                                    |
| `--vignette`          | Flagga    | Aktiverad        | Aktivera vignettkorrigering                                                             |
| `--no-vignette`       | Flagga    | -              | Inaktivera vignettkorrigering                                                            |
| `--reflectance`       | Flagga    | Aktiverad        | Aktivera reflektanskalibrering                                                         |
| `--no-reflectance`    | Flagga    | -              | Inaktivera reflektanskalibrering                                                        |
| `--ppk`               | Flagga    | Inaktiverad       | Tillämpa PPK-korrigeringar från .daq-ljussensordata                                      |
| `--format`            | Val  | TIFF (16-bit)  | Utmatningsformat: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Heltal | Auto           | Minsta målstorlek i pixlar för kalibreringspanelsdetektering                          |
| `--target-clustering` | Heltal | Auto           | Tröskelvärde för målkluster (0-100)                                                    |
| `--exposure-pin-1`    | Sträng  | Ingen           | Lås exponering för kameramodell (stift 1)                                                 |
| `--exposure-pin-2`    | Sträng  | Ingen           | Lås exponering för kameramodell (stift 2)                                                 |
| `--recal-interval`    | Heltal | Auto           | Omkalibreringsintervall i sekunder                                                      |
| `--timezone-offset`   | Heltal | 0              | Tidszonsförskjutning i timmar                                                               |

***

### `login` – Autentisera konto

Logga in med dina Chloros+-inloggningsuppgifter för att aktivera CLI-bearbetning.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

**Exempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
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

```powershell
chloros-cli logout
```

**Utdata:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style=&quot;info&quot; %}
**SDK Användare**: Python SDK tillhandahåller också en programmatisk `logout()`-metod för att rensa autentiseringsuppgifter inom Python-skript. Se [Python SDK-dokumentationen](api-python-sdk.md#logout) för mer information.
{% endhint %}

***

### `status` – Kontrollera licensstatus

Visa aktuell licens- och autentiseringsstatus.

**Syntax:**

```bash
chloros-cli status
```

**Exempel:**

```powershell
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

```powershell
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

```powershell
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

| Kod    | Språk              | Ursprungligt namn      |
| ------- | --------------------- | ---------------- |
| `en`    | Engelska               | Engelska          |
| `es`    | Spanska               | Español          |
| `pt`    | Portugisiska            | Português        |
| `fr`    | Franska                | Français         |
| `de`    | Tyska                | Deutsch          |
| `it`    | Italienska               | Italiano         |
| `ja`    | Japanska              | 日本語              |
| `ko`    | Koreanska                | 한국어              |
| `zh`    | Kinesiska (förenklad)  | 简体中文             |
| `zh-TW` | Kinesiska (traditionell) | 繁體中文             |
| `ru`    | Ryska               | Русский          |
| `nl`    | Nederländska                 | Nederlands       |
| `ar`    | Arabiska                | العربية          |
| `pl`    | Polska                | Polski           |
| `tr`    | Turkiska               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonesiska            | Bahasa Indonesia |
| `vi`    | Vietnamesiska            | Tiếng Việt       |
| `th`    | Thailändska                  | ไทย              |
| `sv`    | Svenska               | Svenska          |
| `da`    | Danska                | Dansk            |
| `no`    | Norsk             | Norsk            |
| `fi`    | Finska               | Suomi            |
| `el`    | Grekiska                 | Ελληνικά         |
| `cs`    | Tjeckiska                 | Čeština          |
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
| `sl`    | Slovenska             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Automatisk lagring**: Dina språkinställningar sparas i `~/.chloros/cli_language.json` och lagras mellan alla sessioner.
{% endhint %}

***

### `set-project-folder` – Ställ in standardprojektmapp

Ändra standardprojektmappens plats (delad med GUI).

**Syntax:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Exempel:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` – Visa projektmapp

Visa den aktuella standardprojektmappens plats.

**Syntax:**

```bash
chloros-cli get-project-folder
```

**Exempel:**

```powershell
chloros-cli get-project-folder
```

**Utdata:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` – Återställ till standard

Återställ projektmappen till standardplatsen.

**Syntax:**

```bash
chloros-cli reset-project-folder
```

***

## Globala alternativ

Dessa alternativ gäller för alla kommandon:

| Alternativ          | Typ    | Standard       | Beskrivning                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Sökväg    | Automatiskt upptäckt | Sökväg till backend-körbar fil                       |
| `--port`        | Heltal | 5000          | Backend API portnummer                          |
| `--restart`     | Flagga    | -             | Tvinga omstart av backend (avslutar befintliga processer) |
| `--version`     | Flagga    | -             | Visa versionsinformation och avsluta                |
| `--help`        | Flagga    | -             | Visa hjälpinformation och avsluta                   |

**Exempel med globala alternativ:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Guide till bearbetningsinställningar

### Parallell bearbetning

Chloros+ CLI **skalar automatiskt**parallell bearbetning för att matcha din dators kapacitet:**Så fungerar det:**

* Detekterar dina CPU-kärnor och RAM-minne
* Tilldelar arbetare: **2× CPU-kärnor** (använder hyperthreading)
* **Maximalt: 16 parallella arbetare** (för stabilitet)**Systemnivåer:**

| Systemtyp   | CPU        | RAM      | Arbetare  | Prestanda     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **Högpresterande**  | 16+ kärnor  | 32+ GB   | Upp till 16 | Maximal hastighet   |
| **Mellanklass** | 8–15 kärnor | 16–31 GB | 8–16     | Utmärkt hastighet |
| **Lågprisklass**   | 4–7 kärnor  | 8–15 GB  | 4–8      | Bra hastighet      |

{% hint style=&quot;success&quot; %}
**Automatisk optimering**: CLI detekterar automatiskt ditt systems specifikationer och konfigurerar optimal parallellbearbetning. Ingen manuell konfiguration behövs!
{% endhint %}

### Debayer-metoder

CLI använder **Hög kvalitet (snabbare)** som standard och rekommenderad debayer-algoritm:

| Metod                      | Kvalitet | Hastighet | Beskrivning                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Hög kvalitet (snabbare)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Kantmedveten algoritm (standard, rekommenderad) |

### Vignettkorrigering

**Vad den gör:** Korrigerar ljusfallet vid bildkanterna (mörkare hörn som är vanliga i kamerabilder).

* **Aktiverat som standard** – De flesta användare bör ha detta aktiverat
* Använd `--no-vignette` för att inaktivera

{% hint style=&quot;success&quot; %}
**Rekommendation**: Aktivera alltid vignettkorrigering för att säkerställa enhetlig ljusstyrka över hela bilden.
{% endhint %}

### Reflektanskalibrering

Konverterar råa sensorvärden till standardiserade reflektansprocenttal med hjälp av kalibreringspaneler.

* **Aktiverat som standard** – Väsentligt för vegetationsanalys.
* Kräver kalibreringsmålpaneler i bilder.
* Använd `--no-reflectance` för att inaktivera.

{% hint style=&quot;info&quot; %}
**Krav**: Se till att kalibreringspanelerna är korrekt exponerade och synliga i dina bilder för korrekt reflektanskonvertering.
{% endhint %}

### PPK-korrigeringar

**Vad det gör:** Tillämpar efterbearbetade kinematiska korrigeringar med hjälp av DAQ-A-SD-loggdata för förbättrad GPS-noggrannhet.

* **Inaktiverat som standard**
* Använd `--ppk` för att aktivera
* Kräver .daq-filer i projektmappen från MAPIR DAQ-A-SD ljussensor.

### Utdataformat

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bitdjup</th><th width="116.5999755859375">Filstorlek</th><th>Bäst för</th></tr></thead><tbody><tr><td><strong>TIFF (16-bitars)</strong> ⭐</td><td>16-bitars heltal</td><td>Stor</td><td>GIS-analys, fotogrammetri (rekommenderas)</td></tr><tr><td><strong>TIFF (32-bitars, procent)</strong></td><td>32-bitars flyttal</td><td>Mycket stor</td><td>Vetenskaplig analys, forskning</td></tr><tr><td><strong>PNG (8-bitars)</strong></td><td>8-bitars heltal</td><td>Medel</td><td>Visuell inspektion, webbdelning</td></tr><tr><td><strong>JPG (8-bitars)</strong></td><td>8-bitars heltal</td><td>Liten</td><td>Snabb förhandsgranskning, komprimerad utdata</td></tr></tbody></table>***

## Automatisering och skript

### PowerShell-batchbearbetning

Bearbeta flera datamängdsmappar automatiskt:

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

### Windows-batchskript

Enkel slinga för batchbearbetning:

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

### Python Automatiseringsskript

Avancerad automatisering med felhantering:

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

1. **Inmatning**: Mapp som innehåller RAW/JPG-bildpar
2. **Upptäckt**: CLI skannar automatiskt efter bildfiler som stöds
3. **Bearbetning**: Parallellt läge skalar till dina CPU-kärnor (Chloros+)
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

| Läge              | Tid      | Hårdvara                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Parallellt läge** | 5–10 min  | i7/Ryzen 7, 16 GB RAM, SSD (upp till 16 arbetare) |
| **Parallellt läge** | 10–15 min | i5/Ryzen 5, 8 GB RAM, HDD (upp till 8 arbetare)   |

{% hint style=&quot;info&quot; %}
**Prestandatips**: Bearbetningstiden varierar beroende på antal bilder, upplösning och datorspecifikationer.
{% endhint %}

***

## Felsökning

### CLI hittades inte

**Fel:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Lösningar:**

1. Kontrollera installationsplatsen:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Använd fullständig sökväg om den inte finns i PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Lägg till manuellt i PATH:
   * Öppna Systemegenskaper → Miljövariabler
   * Redigera variabeln PATH
   * Lägg till: `C:\Program Files\Chloros\resources\cli`
   * Starta om terminalen

***

### Backend kunde inte startas**Fel:**

```

Backend failed to start within 30 seconds
```

**Lösningar:**

1. Kontrollera om backend redan körs (stäng den först)
2. Kontrollera att Windows brandväggen inte blockerar
3. Prova en annan port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Tvinga omstart av backend:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Licens-/autentiseringsproblem**Fel:**

```

Chloros+ license required for CLI access
```

**Lösningar:**

1. Kontrollera att du har ett aktivt Chloros+-abonnemang
2. Logga in med dina inloggningsuppgifter:

```powershell
chloros-cli login user@example.com 'password'
```

3. Kontrollera licensstatus:

```powershell
chloros-cli status
```

4. Kontakta supporten: info@mapir.camera

***

### Inga bilder hittades**Fel:**

```

No images found in the specified folder
```

**Lösningar:**

1. Kontrollera att mappen innehåller format som stöds (.RAW, .TIF, .JPG).
2. Kontrollera att mappens sökväg är korrekt (använd citattecken för sökvägar med mellanslag).
3. Se till att du har läsbehörighet för mappen.
4. Kontrollera att filändelserna är korrekta.

***

### Bearbetningen avbryts eller hänger sig**Lösningar:**

1. Kontrollera tillgängligt diskutrymme (se till att det finns tillräckligt för utdata).
2. Stäng andra program för att frigöra minne.
3. Minska antalet bilder (bearbeta i omgångar).

***

### Porten används redan**Fel:**

```

Port 5000 is already in use
```

**Lösning:**

Ange en annan port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Vanliga frågor

### F: Behöver jag en licens för CLI?

**S:**Ja! CLI kräver en betald**Chloros+-licens**.

* ❌ Standardplan (gratis): CLI inaktiverad
* ✅ Chloros+ (betald) plan: CLI fullt aktiverad

Prenumerera på: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### F: Kan jag använda CLI på en server utan GUI?**S:** Ja! CLI körs helt utan grafiskt gränssnitt. Krav:

* Windows Server 2016 eller senare
* Visual C++ Redistributable installerat
* Tillräckligt med RAM-minne (minst 8 GB, 16 GB rekommenderas)
* Engångsaktivering av GUI-licens på valfri maskin

***

### F: Var sparas bearbetade bilder?**S:**Som standard sparas bearbetade bilder i**samma mapp som ingången** i undermappar för kameramodeller (t.ex. `Survey3N_RGN/`).

Använd alternativet `-o` för att ange en annan utdatamapp:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
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

***

### F: Vad händer om jag trycker på Ctrl+C under bearbetningen?**S:** CLI kommer att:

1. Avsluta bearbetningen på ett korrekt sätt
2. Stänga av backend
3. Avsluta med kod 130

Delvis bearbetade bilder kan finnas kvar i utdatamappen.

***

### F: Kan jag automatisera CLI-bearbetningen?**S:** Absolut! CLI är utformat för automatisering. Se [Automatisering och skriptning](CLI.md#automation--scripting) för exempel på PowerShell, Batch och Python.***

### F: Hur kontrollerar jag versionen av CLI?**S:**

```powershell
chloros-cli --version
```

**Utdata:**

```

Chloros CLI 1.0.2
```

***

## Få hjälp

### Kommandoradshjälp

Visa hjälpinformation direkt i CLI:

```powershell
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

Bearbeta med standardinställningar (vignett, reflektans):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Exempel 2: Högkvalitativt vetenskapligt resultat

32-bitars flytande TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Exempel 3: Snabb förhandsgranskning

8-bitars PNG utan kalibrering för snabb granskning:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Exempel 4: PPK-korrigerad bearbetning

Tillämpa PPK-korrigeringar med reflektans:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Exempel 5: Anpassad utdataplats

Bearbeta till annan enhet med specifikt format:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Exempel 6: Autentiseringsarbetsflöde

Slutför autentiseringsflödet:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Exempel 7: Flerspråkig användning

Ändra gränssnittsspråk:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
