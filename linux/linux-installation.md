# Linux-installation

Chloros distribueras för Linux som `.deb`-paket som installerar CLI och backend-servern. Python SDK är ett separat pip-paket (ingår även i `.deb` som ett versionsanpassat wheel).

Paketfilernas namn anger version och arkitektur: `chloros_1.2.0_amd64.deb` för x86_64 och `chloros_1.2.0_arm64_jp6.deb` för JetPack 6 Jetson-byggnader. Ersätt med den fil du faktiskt har laddat ner i kommandona nedan.

***

## Linux amd64 (x86_64)

### Systemkrav

| Krav | Minsta | Rekommenderat |
| --- | --- | --- |
| **Distribution** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processor** | x86_64 (Intel/AMD) | Intel Core i7 eller bättre |
| **Minne (RAM)** | 8 GB | 16 GB eller mer |
| **Grafikkort** | Inget (bearbetas av CPU) | NVIDIA-grafikkort med minst 4 GB VRAM (12 GB eller mer låser upp `GPU_PARALLEL`, 7 GB eller mer håller Texture Aware borta från enkelbildsvägen) |
| **Lagringsutrymme** | 2 GB ledigt utrymme | SSD med minst 10 GB ledigt utrymme |
| **Python** | Python 3.7+ (för SDK) | Python 3.10+ |

> **Ubuntu 20.04 och Debian 11 stöds inte.** Beroendelistan för `.deb` är
> härledd från vad Chloros-backenden faktiskt länkar mot, och det inkluderar
> `libc6 (>= 2.34)`. Både Focal och Bullseye levereras med glibc 2.31, så `apt` avvisar
> installationen direkt istället för att låta den misslyckas senare vid körning.

### Installation

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` löser inte beroenden. Om det rapporterar saknade paket slutför `sudo apt-get install -f` (eller `sudo apt --fix-broken install`) installationen — detta är den normala vägen, inte ett fel.
{% endhint %}

Kontrollera installationen:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Systemkrav

| Krav | Minsta | Rekommenderat |
| --- | --- | --- |
| **Plattform** | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **JetPack** | JetPack 6.x | Senaste JetPack 6 |
| **Minne (RAM)** | 8 GB (delat mellan GPU och CPU) | 16 GB+ delat (12 GB+ är tröskelvärdet för parallella GPU-arbetare) |
| **Lagringsutrymme** | 2 GB ledigt utrymme | NVMe SSD med minst 10 GB ledigt utrymme |
| **Python** | Python 3.7+ (för SDK) | Python 3.10+ |

### Installation

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Samma uppbyggnad som amd64-versionen `.deb`, med en CUDA-version anpassad för Jetson Orin / Orin NX / Orin Nano. För information om Jetsons minne, värmehantering och beteende vid fältinstallation, se [NVIDIA Jetson-guiden](nvidia-jetson-guide.md).

***

## Python SDK Installation (Alla Linux)

SDK är en ren Python-klient för backend, så samma paket fungerar på både amd64 och arm64. Två källor:**Från PyPI** — den publicerade stabila versionen:

```bash
pip install chloros-sdk
```

**Från det medföljande wheel-paketet** — garanterat kompatibelt med den CLI/backend du just har installerat (använd detta om din `.deb` är nyare än den på PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**PEP 668-distributioner** (Ubuntu 23.10+, Debian 12+) tillåter inte systemomfattande pip-installationer. Använd `pip install --user …`, en virtuell miljö eller `sudo pip install --break-system-packages …`. Paketinstallatören installerar aldrig automatiskt SDK i ditt systems Python — det valet lämnas åt dig.
{% endhint %}

Valfria tillägg:

| Extra | Kommando | Lägger till |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` för strömning av förloppet i realtid |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` för BLE (DAQ-M)-överföring |

Kontrollera SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` installerar Chloros och backend. Python och SDK kommunicerar med den backenden via ett lokalt HTTP och API (`http://127.0.0.1:5000`) och startar den automatiskt vid behov. Använd alltid den bokstavliga IPv4-adressen istället för `localhost` — `localhost` kan omvandlas till `::1` och tar ungefär två sekunder per förfrågan.
{% endhint %}

***

## Första inställningen

### 1. Logga in

Åtkomst till CLI och SDK kräver en betald Chloros+-nivå (**Copper** eller högre), vilket tillämpas på serversidan: en utloggad användare får `401 AUTH_REQUIRED`, och en användare på gratispaketet (Iron) får `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Inloggningsuppgifterna cachelagras i `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Du måste logga in igen efter varje installation eller uppgradering.** Paketets `prerm`-skript rensar avsiktligt `~/.chloros/user_session.json` och den cachade licensen för varje användare på datorn, så att en ny version alltid validerar licensen på nytt istället för att lita på en inaktuell cache.
{% endhint %}

### 2. Kontrollera din licensstatus

```bash
chloros-cli status
```

`chloros-cli status` fungerar på alla nivåer (inklusive gratis), så du kan alltid se varför åtkomst är eller inte är tillgänglig.

### 3. Kör systemdiagnostik

```bash
chloros-cli selftest
```

Sju kontroller körs i ordning, och kommandot avslutas med ett värde som inte är noll om någon av dem misslyckas:

| # | Kontroll | Vad den bevisar |
| --- | --- | --- |
| 1 | **Version** | CLI rapporterar sin version (`v1.2.0`). |
| 2 | **Port tillgänglig** | Port 5000 är ledig, *eller* har redan besvarats av en fungerande Chloros-backend (vilket räknas som godkänt). |
| 3 | **Start av backend** | Backend-programmet startas. |
| 4 | **API-test (`/api/test`)** | Backenden svarar `status: ok`. |
| 5 | **Systeminformation** | Skriver ut `GPU: <name>, CUDA: <bool>, PyTorch: <version>` från `/api/system-info`. |
| 6 | **Denoiser-modeller** | Hittar `*.pth.enc`-modeller (på Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + brusreducerare**| Texture Aware fungerar faktiskt — kräver CUDA**och** minst en modellfil. |

Körningen avslutas med `N/7 checks passed`, där eventuella fel listas med namn.

### 4. Bearbeta din första datamängd

```bash
chloros-cli process ~/datasets/flight001
```

***

## Filer och kataloger

### Per användare

Chloros lagrar sina inloggningsuppgifter och CLI-konfigurationen i en enda plattformsoberoende katalog, **`~/.chloros/`** (på Windows, `%USERPROFILE%\.chloros\`). Två Linux-specifika cacher följer istället XDG-konventionerna — dessa respekterar `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` när de är inställda.

| Sökväg | Syfte |
| --- | --- |
| `~/.chloros/user_session.json` | Cache för inloggningssessioner som skrivs av `chloros-cli login` (rensas vid varje paketinstallation/uppgradering) |
| `~/.chloros/working_directory.txt` | Överskrivning av standardprojektmapp (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Språkinställning för CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Språkinställning som delas med Windows-användargränssnittet — en `language` här har företräde framför `cli_language.json` |
| `~/.chloros/update_cache.json` | En timmes cache för uppdateringskontrollen vid uppstart av Linux/Jetson |
| `~/.chloros/backend.log` | Backend-logg när backenden startades av CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Cachelagrade LATTICE-kalibreringspaket per kamera, indexerade efter serienummer och bunt-hash |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Valfria användaröverskrivningar för DAQ-kapacitetskorrigeringsprofiler |
| `~/.config/chloros/system_config.json` | Cachelagrad hårdvaruprofil från Dynamic Compute Adaptation — ta bort den för att tvinga fram en ny hårdvarudetektering |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Backend-serverloggar, en fil per start |
| `~/Chloros Projects/` | Standardprojektmapp när ingen överskrivning är inställd |

### Systemomfattande

| Sökväg | Syfte |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper-skript — ställer in `LD_LIBRARY_PATH` för de medföljande inbyggda biblioteken och kör sedan den egentliga binärfilen |
| `/usr/bin/chloros-backend` | Wrapper-skript — samma sak, plus `CHLOROS_PRODUCTION=1` så att backendens autentiseringsgrind aldrig kan inaktivera sig själv utan att meddela det |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | De kompilerade binärfilerna |
| `/usr/lib/chloros/arena_runtime/` | Arena SDK-körmiljö som krävs av LATTICE-kameror |
| `/usr/lib/chloros/models/*.pth.enc` | Krypterade brusreduceringsmodeller som används av Texture Aware-debayern |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK-hjul som matchar just denna version |
| `/usr/lib/chloros/exiftool` | Medföljande exiftool (symlänkat till `/usr/local/bin/exiftool` endast om det inte finns något exiftool i systemet) |
| `/etc/chloros/update.conf` | Konfiguration för uppdateringskanalen som läses av `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Ställer in `net.ipv4.ip_unprivileged_port_start = 319` så att backend kan binda PTP-portarna utan root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Riktar den dynamiska laddaren mot `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Ger den inloggade användaren åtkomst till DAQ-U USB-seriell brygga (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Aktivera alltid-på-backend-tjänsten (installerad, **inte aktiverad**) |
| `/usr/share/applications/chloros-cli.desktop` | Applikationsmenypost ”Chloros CLI” som öppnar en terminal |

## Plats för den körbara backend-filen

CLI och SDK upptäcker backenden automatiskt:

| Komponent | Sökväg |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Åsidosätt backend-sökvägen med flaggan `--backend-exe` CLI eller konstruktorparametern `backend_exe` SDK, och porten med `--port` (standard `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` pekar på **`lattice`**,**`project`**och**`daq pool-*`**-kommandofamiljerna på en fjärrbackend. Kärnkommandona (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignorerar den medvetet och riktar sig alltid mot `http://127.0.0.1:<port>`.
{% endhint %}

***

## LATTICE-kameror och DAQ-ljussensorer på Linux

Alla kommandofamiljerna för live-hårdvara fungerar på Linux (amd64 och Jetson):

* **`chloros-cli lattice`** — upptäcka, ansluta, konfigurera och samla in data från LATTICE-kameror och synkroniserade matriser. `.deb` innehåller den Arena SDK-körmiljö som krävs och registrerar den hos den dynamiska laddaren.
* **`chloros-cli daq pool-*`** — anslut DAQ-U/M/E-ljussensorer via backend-poolen, strömma kalibrerade spektra och spela in `.daq`-filer. Den kompilerade CLI levereras endast med `pool-*`-familjen: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — kör ett sparat projekt (dess kameror, sensorer och bearbetningsinställningar) utan skärm.
* **`chloros-cli time-sync`** — granska PTP-grandmastern som Chloros-backend kör för LATTICE-kameror och DAQ-E-sensorer.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` krävs av `pool-latest`, `pool-stream`, `pool-record` och `pool-set-cap`; `pool-list` visar de ID:n som för närvarande finns i poolen.

{% hint style="info" %}
**Använd helst `--eth-host` för den första DAQ-E-anslutningen på en maskin med flera nätverksanslutningar.** Automatisk upptäckt söker igenom mDNS och kan missa sensorns gränssnitt på grund av en tom ARP-cache, så den första `pool-connect --eth` efter uppstart kan misslyckas även om sensorn är helt felfri. Genom att ange sensorns IP-adress eller värdnamn hoppas upptäckten över helt.
{% endhint %}

**Seriebaserade behörigheter för DAQ-U** hanteras av den installerade udev-regeln (`uaccess` + gruppen `dialout`). Om en sensor som redan var ansluten förblir otillgänglig, ladda om reglerna eller koppla in den på nytt:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Se [CLI-referensen](../CLI.md) för den fullständiga kommandosatsen.

### PTP som alltid är på för headless-värdar

Vid den första installationen genereras systemd-enheten `chloros-backend.service` men den är **inte aktiverad**. På en Jetson-enhet utan skärm eller en server som ska hålla PTP-tidssynkroniseringen igång kontinuerligt för DAQ-E-sensorer och LATTICE-kameror ska du aktivera den:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Utan den körs PTP endast medan backend-enheten Chloros är igång — det vill säga under en aktiv session med CLI/SDK.

Enheten kopplar backend till `127.0.0.1:5000` (miljöinställningarna `CHLOROS_HOST` / `CHLOROS_PORT` inuti enheten; överskriv med `sudo systemctl edit chloros-backend.service`) och startar om den vid fel efter 5 sekunder.

**Hur PTP tilldelas sina portar.** PTP använder UDP 319/320, båda under den normala gränsen på 1024 för privilegierade portar. Paketets `postinst` skriver `/etc/sysctl.d/60-chloros-ptp.conf` med `net.ipv4.ip_unprivileged_port_start = 319`, vilket låter backend-programmet binda dem medan det körs som din användare. Det tillämpar också `setcap cap_net_bind_service,cap_net_raw=+ep` på backend-binären som en extra säkerhetsåtgärd – det är därför `libcap2-bin` är ett deklarerat beroende för paketet.***

## Exempel på Bash-skript

{% hint style="info" %}
**Skriptvänliga avslutningskoder.**`chloros-cli process` avslutar med `0` vid framgång och**ett värde olika noll vid misslyckande – inklusive en körning som begärde bildprodukter men inte skrev ut några** (det skriver ut `Processing finished but wrote no image products.` och anger projektmappen samt de vanligaste orsakerna). Lyckade körningar rapporterar hur många bildprodukter som skrevs ut (`Image products written: N`). Avslutningskoder: `0` framgång, `1` misslyckande, `2` argumentfel, `130` avbruten.
{% endhint %}

### Bearbeta flera datamängder

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Bearbeta med anpassade inställningar

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

De giltiga `--format`-värdena är exakt fyra till antalet och innehåller mellanslag – ange dem alltid inom citationstecken:

| `--format`-värde | Utmatningsmapp |
| --- | --- |
| `TIFF (16-bit)` *(standard)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` accepterar `standard` (standard) eller `texture-aware` (Chloros+).

### Automatiserad bearbetning med Cron

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

### CLI hittas inte efter installationen

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Åtkomst nekad

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### ”setcap misslyckades” under installationen

`.deb` tillämpar `cap_net_bind_service` på `/usr/lib/chloros/chloros-backend` så att det kan binda PTP-portarna 319/320 utan root-behörighet. Om `libcap2-bin` saknades vid installationen hoppas anropet över. Installera det och installera om paketet:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP startar inte / kan inte binda port 319

Kontrollera att gränsen för icke-privilegierade portar har sänkts, och tillämpa den på nytt för den aktuella uppstarten om så inte var fallet:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Kontrollera sedan grandmastern:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### ”LATTICE-kameradrivrutiner hittades inte”

Arena SDK-körmiljön kan inte lösas. Kontrollera att den laddarkonfiguration som paketet skriver finns och är uppdaterad:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
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

Backend-loggarna för den misslyckade starten finns i `~/.cache/chloros/logs/`.

### CUDA upptäcktes inte

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` rapporterar samma sak på en rad: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Saknade delade bibliotek

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Långsam uppstart på system med SD-kort

De kompilerade binärfilerna extraheras till en tillfällig katalog vid varje start. Om `/mnt/ssd/tmp` finns, använder Chloros den automatiskt; annars ska du ställa in `TMPDIR` till ett snabbt filsystem:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Uppdatering av Chloros på Linux

Kommandot `update` gäller endast Linux/Jetson. Det kontrollerar versionen som publicerats i den uppdateringskanal som konfigurerats på `/etc/chloros/update.conf` och erbjuder att ladda ner och installera den matchande `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

På Linux/Jetson utför CLI dessutom en icke-blockerande uppdateringskontroll vid varje uppstart (resultatet cachelagras i en timme i `~/.chloros/update_cache.json`) och visar meddelandet `Update available: vX.Y.Z` när en nyare version finns. Dina inställningar och projekt bevaras vid en uppdatering; du måste logga in igen efteråt.

## Avinstallation

```bash
sudo apt remove chloros
```

Avinstallationen avslutar `chloros-backend.service`, återställer standardgränsen för icke-privilegierade portar (1024), tar bort symlänken till det medföljande exiftool och Arena-laddarens konfiguration samt rensar cachade inloggningsuppgifter. Dina projekt och `~/.chloros/`-datafiler lämnas orörda.

***

## Nästa steg

* [NVIDIA Jetson-guide](nvidia-jetson-guide.md) — Jetson-specifik optimering och driftsättning
* [CLI : Kommandorad](../CLI.md) — guiden för CLI
* [API : Python SDK](../api-python-sdk.md) — guiden SDK
* [CLI Referens](../reference/cli-reference.md) och [SDK Referens](../reference/sdk-reference.md) – uttömmande listor över kommandon/API för version 1.2.0
* [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md) — hur Chloros anpassar sig till din hårdvara
