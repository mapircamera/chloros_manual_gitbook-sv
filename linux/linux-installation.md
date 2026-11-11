# Installation på Linux

Chloros distribueras för Linux som `.deb`-paket som installerar CLI, backend-servern och Python-SDK (för systemet `python3`, plus ett versionsanpassat wheel för virtualenvs).

Paketfilernas namn anger version och arkitektur: `chloros_1.3.0_amd64.deb` för x86_64, och `chloros_1.3.0_arm64_jp6.deb` för arm64 (NVIDIA Jetson, JetPack 6). Ersätt med den fil du faktiskt har laddat ner i kommandona nedan.

{% hint style="info" %}
**Nytt i 1.3.0: programmen levereras i uppackat format.** Backend och CLI installeras som mappar, `/usr/lib/chloros/backend/` och `/usr/lib/chloros/cli/`, istället för som enskilda självuppackande filer. De packar inte längre upp sig själva i en tillfällig mapp vid varje start, vilket gör att de startar mycket snabbare (på x86_64 brukade backend-delen ta cirka 100 sekunder att packa upp vid varje start) och behöver inte längre flera GB ledigt utrymme i `/tmp`. Amd64-versionen `.deb` är också ungefär hälften så stor som tidigare.
{% endhint %}

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

> **Ubuntu 20.04 och Debian 11 stöds inte.** `.deb`:s beroendelista är
> härledd från vad Chloros-backend faktiskt länkar mot, och det inkluderar en
> `libc6` som är nyare än den glibc 2.31 som levereras med Focal och Bullseye, så `apt` avvisar
> installationen direkt istället för att låta den misslyckas senare vid körning.

### Installation

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Använd `apt install ./…`, inte `dpkg -i`.** `apt` löser paketets beroenden och installerar även det som rekommenderas: kraven för Python SDK, `python3-numpy` och `python3-requests`. `dpkg -i` installerar inget av dem. CLI:n och backend:en är fristående och körs utan dem, men `import chloros_sdk` behöver dem; installationsutmatningen anger vilka som saknas. Behåll `./`, så att `apt` installerar filen istället för att söka efter ett paket med det namnet. Paketet deklarerar även `libgl1`, som det medföljande OpenCV laddar och som kan saknas i en minimal serverbild.
{% endhint %}

Kontrollera installationen:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Systemkrav

| Krav | Minsta | Rekommenderat |
| --- | --- | --- |
| **Plattform** | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **JetPack** | JetPack 6.x | Senaste JetPack 6 |
| **Minne (RAM)** | 8 GB (delat mellan GPU och CPU) | 16 GB+ delat (12 GB+ är tröskelvärdet för parallellbearbetning) |
| **Lagringsutrymme** | 2 GB ledigt utrymme | NVMe SSD med minst 10 GB ledigt utrymme |
| **Python** | Python 3.7+ (för SDK) | Python 3.10+ |

### Installation

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Samma uppbyggnad som amd64 `.deb` (inklusive Python-SDK, mottagningsbuffertar och backend-tjänster), för Jetson Orin / Orin NX / Orin Nano på **JetPack 6** (L4T R36). PyTorch är JetPack 6:s CUDA-version för Orin, vilket innebär att GPU-bearbetning, inklusive Texture Aware-debayer, körs på Jetsons GPU. Paketet innehåller inte CUDA i sig: det använder de CUDA- och cuDNN-bibliotek som JetPack 6 installerar (`sudo apt install nvidia-jetpack` på en image som saknar dem). Om ingen CUDA-GPU kan användas körs bearbetningen på CPU:n och `--debayer texture-aware` använder Standard-debayern (backend-loggen visar en `[DEBAYER] … needs a CUDA GPU …`-rad som förklarar varför). För information om Jetsons minne, värmehantering och beteende vid fältinstallation, se [NVIDIA Jetson-guiden](nvidia-jetson-guide.md).***

## Installation av Python-SDK (alla Linux-versioner)**`.deb` installerar SDK åt dig.** Båda paketen (amd64 och Jetson) installerar det för systemet `python3`, så `import chloros_sdk` fungerar direkt efter installationen, utan pip och utan nätverk. Dess paket (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` och projektladdarna) placeras i `/usr/lib/python3/dist-packages`, som tillhör paketet: en uppgradering ersätter dem, och när paketet avinstalleras tas de bort. Versionen stämmer alltid överens med den CLI och det backend som du har installerat.

SDK:s egna krav, `python3-numpy` och `python3-requests`, är **rekommenderade, inte obligatoriska** av paketet: `sudo apt install ./chloros_<version>_<arch>.deb` installerar dem, medan `dpkg -i` inte gör det. Installationsutmatningen rapporterar vilken SDK-version `python3` importerar, eventuella krav som saknas, samt alla `chloros_sdk` på andra platser i `python3`:s sökväg som överskuggar paketets kopia (till exempel en som installerats tidigare med `pip install --user`).**I en virtualenv**, som inte ser systemets paket, installerar du det medföljande wheel-paketet i den (helt offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Från PyPI** — endast för en maskin utan något Chloros-paket installerat, vars SDK-miljö driver en backend på en annan maskin (se [Remote-Backend Mode](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Valfria tillägg, för en pip-installation (en virtualenv eller PyPI):

| Tillägg | Kommando | Lägger till |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` för strömning av realtidsförlopp |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` för BLE (DAQ-M)-överföring |

Verifiera SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Python-SDK:et kommunicerar med den backend som `.deb` installerar via ett lokalt HTTP-API (`http://127.0.0.1:5000`). `ChlorosLocal` och anslutningshjälpprogrammen startar den automatiskt vid behov; ett sparat projekts `connect_all()` gör det aldrig, så på en maskin utan grafiskt gränssnitt ska du köra backend-modulen som en [tjänst](#always-on-ptp-for-headless-hosts). Använd alltid den bokstavliga IPv4-adressen istället för `localhost` — `localhost` kan omdirigeras till `::1` och ta ungefär två sekunder per förfrågan.
{% endhint %}

***

## Första installation

### 1. Logga in

Åtkomst till CLI och SDK kräver en betald Chloros+-nivå (**Copper** eller högre), vilket kontrolleras på serversidan: en utloggad användare får `401 AUTH_REQUIRED`, och en användare på gratisnivån (Iron) får `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Inloggningsuppgifterna cachelagras i `~/.chloros/user_session.json`.

{% hint style="info" %}
**Vid uppgradering behålls din inloggning; vid borttagning raderas den.** Om du installerar en nyare version av `.deb` över en befintlig installation (även via `chloros-cli update`) behålls `~/.chloros/user_session.json` och den cachade licensen, så att CLI- och SDK-jobb utan grafiskt gränssnitt fortsätter att fungera. Om du avinstallerar paketet (`sudo apt remove chloros`) raderas den sparade inloggningen och licenscachen för alla användare på datorn, vilket innebär att en ny installation kräver `chloros-cli login` igen.
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

Sju kontroller körs i tur och ordning, och kommandot avslutas med ett värde som inte är noll om någon av dem misslyckas:

| # | Kontroll | Vad den bevisar |
| --- | --- | --- |
| 1 | **Version** | CLI:n rapporterar sin version (`v1.3.0`). |
| 2 | **Port tillgänglig** | Port 5000 är ledig, *eller* har redan besvarats av en fungerande Chloros-backend (vilket räknas som godkänt). |
| 3 | **Start av backend** | Backend-programmet startas. |
| 4 | **Test av API (`/api/test`)** | Backenden svarar med `status: ok`. |
| 5 | **Systeminformation** | Skriver ut `GPU: <name>, CUDA: <bool>, PyTorch: <version>` från `/api/system-info`. |
| 6 | **Denoiser-modeller** | Hittar `*.pth.enc`-modeller (på Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + brusreducerare**| Texture Aware är faktiskt användbart — kräver CUDA**och** minst en modellfil. |

Körningen avslutas med `N/7 checks passed`, där eventuella fel listas med namn.

### 4. Bearbeta din första datamängd

```bash
chloros-cli process ~/datasets/flight001
```

***

## Filer och kataloger

### Per användare

Chloros lagrar sina inloggningsuppgifter och CLI-konfiguration i en enda plattformsoberoende katalog, **`~/.chloros/`** (i Windows, `%USERPROFILE%\.chloros\`). Två Linux-specifika cacher följer istället XDG-konventionerna — dessa använder `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` när de är inställda.

| Sökväg | Syfte |
| --- | --- |
| `~/.chloros/user_session.json` | Cache för inloggningssessioner som skrivs av `chloros-cli login` (bevaras vid paketuppgraderingar; rensas när paketet avinstalleras) |
| `~/.chloros/working_directory.txt` | Överskrivning av standardprojektmapp (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Språkinställning för CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Språkinställning som delas med Windows-GUI:n — ett `language` här har företräde framför `cli_language.json` |
| `~/.chloros/update_cache.json` | En timmes cache för uppdateringskontrollen vid uppstart av Linux/Jetson |
| `~/.chloros/backend.log` | Backend-logg när backend startades av CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Cachelagrade LATTICE-kalibreringspaket per kamera, indexerade efter serienummer och bunt-hash |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Valfria användaröverskrivningar för DAQ-kapacitetskorrigeringsprofiler |
| `~/.config/chloros/system_config.json` | Cachelagrad hårdvaruprofil från Dynamic Compute Adaptation – ta bort den för att tvinga fram en ny hårdvarudetektering |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Backend-serverloggar, en fil per start |
| `~/Documents/MAPIR/Chloros Projects/` | Standardprojektmapp när ingen överskrivning är inställd |

### Systemomfattande

| Sökväg | Syfte |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper-skript — ställer in `LD_LIBRARY_PATH` för de medföljande inbyggda biblioteken (och `TMPDIR` till `/mnt/ssd/tmp` när den mappen finns), och kör sedan den egentliga binärfilen |
| `/usr/bin/chloros-backend` | Wrapper-skript — samma sak för backend, som det markerar som en installerad produktionsversion |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Den kompilerade CLI:n och backend:en, installerade i uppackat skick (nytt i 1.3.0); programmen är `cli/chloros-cli` och `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Tvåradiga startskript som startar de egentliga programmen ovan — sparas på dessa sökvägar eftersom SDK, CLI och äldre skript letar efter dem här |
| `/usr/lib/chloros/arena_runtime/` | Arena SDK-körmiljö som krävs av LATTICE-kameror |
| `/usr/lib/chloros/models/*.pth.enc` | Krypterade brusreduceringsmodeller som används av Texture Aware-debayern |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (samt `lattice_sdk`, `lattice_cam`, `arena_api`, projektladdarna) | Python-SDK, installerat för systemet `python3` och tillhörande paketet |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python-SDK-wheel som exakt matchar denna build, för en virtualenv |
| `/usr/lib/chloros/exiftool` | Medföljande ExifTool-startprogram (symlänkat till `/usr/local/bin/exiftool` endast om det inte finns något system-exiftool). Från och med version 1.3.0 levereras ExifTool-skriptet med sitt eget Perl-bibliotek i `/usr/lib/chloros/exiftool-dist/`, så det körs utan systempaketet `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Konfiguration för uppdateringskanalen som läses av `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Ställer in `net.ipv4.ip_unprivileged_port_start = 319` så att backend kan binda PTP-portarna utan root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Ökar kärnans mottagningsbuffertar för socklar (`net.core.rmem_max`, `net.core.rmem_default`) till 32 MB för LATTICE-kameraströmmar och tillämpar dem vid installationen (ett värde som redan är högre lämnas oförändrat). Med kärnans standardinställning tappas kamerapaket bort i kärnan och en array förlorar bildrutor och bildhastighet; `chloros-cli lattice network` rapporterar buffertarna när de är låga. Se [Linux-värdar — mottagningsbuffertar för socklar](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Riktar den dynamiska laddaren mot `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Ger den inloggade användaren åtkomst till DAQ-U USB-seriell brygga (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Opt-in-tjänst för alltid-på-backend som kör backenden som den användare du anger (installerad, **inte aktiverad**) — den som ska aktiveras, se [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Samma tjänst som kör backend som **root**(installerad,**inte aktiverad**), behålls så att en maskin som har aktiverat den fortsätter att köras |
| `/var/log/chloros/` | Den detaljerade loggen för root-tjänsten, läsbar av gruppen `adm` (raderas när paketet rensas) |
| `/usr/share/applications/chloros-cli.desktop` | Applikationsmenypost för ”Chloros CLI” som öppnar en terminal |

**Brandvägg (nytt i 1.3.0).** Om `ufw` eller `firewalld` är aktiva vid installationstillfället öppnar paketet UDP-portarna `5002` (DAQ-E-data), `5353` (mDNS-upptäckt), `319` och `320` (PTP), och tar bort dessa regler när paketet avinstalleras. Om en brandvägg är installerad men inte aktiv lägger den inte till något och visar vilka portar som ska tillåtas om du aktiverar den senare. Den öppnar inga TCP-portar: en DAQ-E-firmwareuppdatering, där sensorn ansluter tillbaka till den här datorn via TCP för att hämta bilden, misslyckas med `… device never connected back on TCP <port> …` om en aktiv brandvägg blockerar den anslutningen.

## Plats för backend-programmet

CLI och SDK upptäcker backenden automatiskt:

| Komponent | Sökväg |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (ett startprogram för `/usr/lib/chloros/backend/chloros-backend`) |

Åsidosätt sökvägen till backend med CLI-flaggan `--backend-exe` eller SDK-konstruktorparametern `backend_exe`, och porten med `--port` (standard `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` pekar på kommandofamiljerna **`lattice`**,**`project`**och**`daq pool-*`**-kommandofamiljerna på en fjärrbackend. Kärnkommandona (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignorerar den medvetet och riktar sig alltid mot `http://127.0.0.1:<port>`.
{% endhint %}

***

## LATTICE-kameror och DAQ-ljussensorer på Linux

Alla kommandofamiljerna för live-hardware fungerar på Linux (amd64 och Jetson):

* **`chloros-cli lattice`** — upptäcka, ansluta, konfigurera och samla in data från LATTICE-kameror och synkroniserade matriser. `.deb` buntar ihop den Arena-SDK-runtime som krävs och registrerar den hos den dynamiska laddaren.
* **`chloros-cli daq pool-*`** — anslut DAQ-U/M/E-ljussensorer via backend-poolen, strömma kalibrerade spektra, spela in `.daq`-filer och byt cap-profiler, samt ändra nätverksinställningarna för en DAQ-E, starta om den och uppdatera dess firmware. `chloros-cli daq discover` söker efter sensorer som du kan ansluta, och `chloros-cli daq --help` listar `pool-*`-underkommandona.
* **`chloros-cli project`** — kör ett sparat projekt (dess kameror, sensorer och bearbetningsinställningar) utan skärm.
* **`chloros-cli time-sync`** — inspektera PTP-grandmastern som Chloros-backend kör för LATTICE-kameror och DAQ-E-sensorer.

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
**`--eth-host` hoppar över upptäckten.** `pool-connect --eth` hittar en DAQ-E via mDNS, med en ARP-reservlösning som fungerar från en tom ARP-cache; genom att skicka sensorns IP-adress eller värdnamn med `--eth-host` ansluts man direkt till den.
{% endhint %}

**Seriella behörigheter för DAQ-U** hanteras av den installerade udev-regeln (`uaccess` + gruppen `dialout`). Om en sensor som redan var ansluten förblir otillgänglig, ladda om reglerna eller anslut den på nytt:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Se [CLI-referensen](../CLI.md) för den fullständiga kommandosatsen.

### PTP som alltid är på för hostar utan skärm

Paketet installerar två systemd-tjänster för backend, men ingen av dem är aktiverad. På en Jetson utan skärm eller en server som ska hålla PTP-tidssynkroniseringen igång kontinuerligt för DAQ-E-sensorer och LATTICE-kameror, eller där SDK-skript ansluter sparade projekt (`connect_all()` startar aldrig en backend), aktiverar du tjänsten **som din användare**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` kör backend-delen som den användaren: de datainsamlingar den skriver tillhör den användaren, dess detaljerade logg finns i den användarens `~/.cache/chloros/logs` och den använder den användarens `chloros-cli login` (`~/.chloros`), så logga in en gång som den användaren först. När paketet installeras skrivs detta kommando ut med ditt användarnamn ifyllt.

Utan en tjänst körs backend-delen (och därmed PTP-grandmastern) endast så länge något har startat den – ett `chloros-cli`-kommando eller ett SDK-skript.

Enheten kopplar backend-delen till `127.0.0.1:5000` (miljöinställningarna `CHLOROS_HOST` / `CHLOROS_PORT` inuti enheten; åsidosätt med `sudo systemctl edit chloros-backend@.service`) och startar om den vid fel efter 5 sekunder.

**Root-tjänsten.**`chloros-backend.service` kör backend-modulen som**root**. Den är fortfarande installerad så att en maskin där den har aktiverats fortsätter att köras, men allt som en root-backend skriver tillhör root, så en användare kan inte radera sina egna inspelningar; dess logg finns i `/var/log/chloros` och kan läsas av gruppen `adm`. För att byta till din användare:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**En tjänsts backend är delad och stoppas aldrig av en klient.** `chloros-cli` och SDK-skript använder den som den är: `process` körs på den och laddar sitt eget projekt istället för det senaste. Där CLI:et annars skulle starta om backend-delen (`--restart`, eller en `process` vars import inte hittade något), namnger den istället enheten, skriver ut `sudo systemctl restart <unit>` och avslutar `1`, eftersom en backend som stoppas på det sättet skulle förbli avstängd. Starta om den själv med `sudo systemctl restart chloros-backend@$USER.service`. En tjänsts backend stängs aldrig av av sig själv vid inaktivitet, och `chloros-cli backend stop` avvisar detta och anger kommandot `systemctl stop`.**Hur PTP tilldelas sina portar.** PTP använder UDP 319/320, båda under den normala gränsen på 1024 för privilegierade portar. Paketets `postinst` skriver `/etc/sysctl.d/60-chloros-ptp.conf` med `net.ipv4.ip_unprivileged_port_start = 319`, vilket låter backend-delen binda dem medan den körs som din användare. Från och med version 1.3.0 ställer paketet inte in några filbehörigheter (`setcap`) på backend-programmet, och `libcap2-bin` är inte längre ett beroende: på det uppackade backend-programmet skulle en filbehörighet hindra det från att starta, så lägg inte till någon själv.***

## Exempel på Bash-skript

{% hint style="info" %}
**Skriptvänliga avslutningskoder.**`chloros-cli process` avslutar `0` vid framgång och**ger ett värde större än noll vid misslyckande — inklusive en körning som begärde bildprodukter men inte skrev ut några** (det visar `Processing finished but wrote no image products.` och anger projektmappen samt de vanligaste orsakerna). Lyckade körningar rapporterar hur många bildprodukter som skrevs (`Image products written: N`, plus `(plus M clamp masks)` när masker skrevs). Avslutningskoder: `0` framgång, `1` misslyckande, `2` argumentfel, `130` avbruten.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

De giltiga `--format`-värdena är exakt fyra till antalet och innehåller mellanslag – ange dem alltid inom citattecken:

| `--format`-värde | Utmatningsmapp |
| --- | --- |
| `TIFF (16-bit)` *(standard)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` accepterar `standard` (standard) eller `texture-aware` (Chloros+).

### Automatiserad bearbetning med Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Exempel på Python-SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Felsökning

### CLI-verktyget hittas inte efter installationen

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Backend startar inte efter att filbehörigheter har lagts till

Kör inte `setcap` på backend. Från och med version 1.3.0 ställer paketet inte in några filfunktioner, och PTP:s portar 319/320 kommer från inställningen `sysctl` som beskrivs under [Hur PTP får sina portar](#always-on-ptp-for-headless-hosts). En filkapacitet sätter den uppackade backend-modulen i ett säkert körningsläge där den inte kan hitta sina egna bibliotek, vilket gör att den inte startar. Om du har lagt till en sådan, ta bort den:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP startar inte / kan inte binda port 319

Kontrollera att gränsen för icke-privilegierade portar har sänkts, och tillämpa den på nytt för den aktuella uppstarten om så inte redan skett:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Kontrollera sedan grandmastern:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice`-kommandon misslyckas med meddelandet ”LATTICE-kameradrivrutiner hittades inte”

Backenden kunde inte ladda den LATTICE-kameraruntime som den är installerad med. Varje `lattice`-kommando körs på backenden; CLI:en i sig innehåller inga kameradrivrutiner. Installera om Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): `.deb` installerar runtime-miljön bredvid backend-modulen, i `/usr/lib/chloros/arena_runtime`, och registrerar den hos den dynamiska laddaren. För att kontrollera registreringen:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

Backend-loggarna för den misslyckade starten finns i `~/.cache/chloros/logs/`. Om port 5000 tillhör en backend-tjänst (`systemctl status chloros-backend@$USER.service`), starta om tjänsten istället för att avsluta dess process.

### CUDA har inte upptäckts

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` rapporterar samma sak på en rad: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Saknade delade bibliotek

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Långsam uppstart på system med SD-kort

Från och med version 1.3.0 installeras backend-tjänsten och CLI:n i okomprimerat format, så de packas inte längre upp till en tillfällig katalog vid varje start. Tillfälliga filer placeras automatiskt i `/mnt/ssd/tmp` om den mappen finns; annars kan du peka dem mot ett snabbt filsystem:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Uppdatera Chloros på Linux

Kommandot `update` fungerar endast på Linux/Jetson. Det kontrollerar versionen som publicerats i den uppdateringskanal som konfigurerats i `/etc/chloros/update.conf` och erbjuder att ladda ner och installera motsvarande `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

På Linux/Jetson utför CLI:n även en icke-blockerande uppdateringskontroll vid varje uppstart (resultatet cachas i en timme i `~/.chloros/update_cache.json`) och visar `Update available: vX.Y.Z` när en nyare version finns. Dina inställningar, projekt och inloggning bevaras vid en uppdatering.

När paketet uppgraderas stoppas alla backend-tjänster under installationen, vilket innebär att ett skript som körs förlorar sina kameror och sensorer och sedan startar om var och en av de som är aktiverade (en tjänst som startats manuellt men aldrig aktiverats förblir avstängd, precis som efter en omstart).

## Avinstallation

```bash
sudo apt remove chloros
```

Avinstallationen stoppar och inaktiverar alla backend-tjänster, återställer standardgränsen för icke-privilegierade portar (1024), tar bort inställningsfilen för mottagningsbuffert (de aktuella värdena kvarstår tills omstart sker), den medföljande symlänken till exiftool, Arena-laddarens konfiguration, de brandväggsregler som paketet lagt till och det Python-SDK som det installerat, samt rensar cachade inloggningsuppgifter. Dina projekt och `~/.chloros/`-datafiler lämnas orörda. `sudo apt purge chloros` raderar även `/var/log/chloros`.

***

## Nästa steg

* [NVIDIA Jetson-guide](nvidia-jetson-guide.md) — Jetson-specifik optimering och driftsättning
* [CLI: Kommandorad](../CLI.md) — guiden för CLI
* [API: Python SDK](../api-python-sdk.md) — guiden för SDK
* [Referens för CLI](../reference/cli-reference.md) och [Referens för SDK](../reference/sdk-reference.md) — uttömmande listor över kommandon/API för version 1.3.0
* [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md) — hur Chlorosanpassar sig till din hårdvara
