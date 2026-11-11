# Översikt över Linux

Chloros 1.3.0 erbjuder inbyggt stöd för Linux för **CLI**och**Python SDK** – bearbetning av multispektrala bilder utan grafiskt gränssnitt samt styrning i realtid av LATTICE-kameror och DAQ-ljussensorer – på Linux-arbetsstationer, servrar och NVIDIA Jetson-enheter för kantberäkning.

{% hint style="info" %}
**Inget grafiskt användargränssnitt (GUI) på Linux.**Chloros Desktop GUI finns endast för Windows. Linux-användare interagerar med Chloros via [CLI](../CLI.md) och [Python SDK](../api-python-sdk.md). `.deb` lägger till en post för**Chloros CLI** i din programmeny – den öppnar helt enkelt en terminalemulator som kör `chloros-cli`.
{% endhint %}

***

## Plattformsstödmatris

| Funktion | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/ --- | --- | --- | --- |
| **GUI för skrivbordet** | Ja | Ej tillämpligt | Nej | Nej |
| **CLI** (`chloros-cli`) | Ja | Ja | Ja | Ja |
| **Python-SDK** (`chloros-sdk`) | Ja | Ja | Ja | Ja |
| **Bildbehandlingspipeline** | Ja | Ja | Ja | Ja |
| **LATTICE-kamerastyrning (live)** | Ja (fliken Kameror) | Ja (`chloros-cli lattice`, SDK) | Ja | Ja |
| **DAQ-ljussensorer (live)** | Ja (fliken Ljussensorer) | Ja (`chloros-cli daq pool-*`, SDK) | Ja | Ja |
| **PTP-tidssynkronisering (värden är grandmaster)** | Ja | Ja (`chloros-cli time-sync`) | Ja | Ja |
| **GPU-acceleration (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Texturmedveten debayer** (kräver en CUDA-GPU) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamisk beräkningsanpassning** | Ja | Ja | Ja | Ja |
| **Backend som systemtjänst** (`chloros-backend@<user>.service`) | Nej | Nej | Ja (opt-in) | Ja (opt-in) |
| **Uppdatering på plats** (`chloros-cli update`) | Nej (kör installationsprogrammet) | Nej (kör installationsprogrammet) | Ja | Ja |***

## Arkitekturer som stöds

| Arkitektur | Beskrivning | Paket |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardprocessorer för stationära datorer/servrar (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM-processorer — NVIDIA Jetson Orin-familjen (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Stödda Linux-distributioner

* **Ubuntu 22.04 LTS eller nyare** (amd64)
* **Debian 12 eller nyare** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin-plattformar)***

## Vad Linux-användare får

* **Chloros CLI** — det fullständiga kommandoradsgränssnittet för batchbearbetning, automatisering och skriptning
* **Chloros Python SDK** — ett programmeringsgränssnitt i Python för forskningspipelines och anpassade verktyg, som installeras av `.deb` för systemet `python3` (ett versionsanpassat wheel för virtualenvs finns i `/usr/lib/chloros/sdk/`; PyPI är avsett för maskiner utan ett Chloros-paket)
* **LATTICE-kamerastyrning** — upptäck, anslut, konfigurera och ta bilder från LATTICE-kameror och synkroniserade multikamerauppsättningar via `chloros-cli lattice` och SDK; `.deb` innehåller den Arena SDK-runtime som kamerorna kräver
* **DAQ-ljussensorstyrning** — anslut DAQ-U/M/E-sensorer, strömma kalibrerade spektra och spela in `.daq`-filer via `chloros-cli daq pool-*` och SDK
* **PTP-tidssynkronisering** — Chloros-backend kör den PTP-grandmaster som LATTICE-kameror och DAQ-E-sensorer är slavar till; kontrollera den med `chloros-cli time-sync` och håll den igång i headless-läge med systemd-enheten `chloros-backend@<user>.service` (se [Linux-installation](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projektautomatisering** — kör sparade projekt i bakgrunden med `chloros-cli project` och SDK:s `open_project`
* **GPU-acceleration** — CUDA-accelererad bearbetning på NVIDIA-GPU:er (stationära GPU:er på amd64 och Jetsons GPU med JetPack 6)
* **Dynamisk beräkningsanpassning** — automatisk hårdvarudetektering och val av bearbetningsstrategi, med `CHLOROS_STRATEGY`-överskrivningen som en expertutväg
* **Alla bearbetningsfunktioner** — samma pipeline som i Windows: kalibrering, vignettkorrigering, vegetationsindex och alla exportformat
* **Chloros+-funktioner** — flertrådig (pipelinebaserad) bearbetning, Texture Aware-debayer (på en CUDA-GPU) och anpassade index, med ett betalt Chloros+-abonnemang

## Vad Linux-användare inte får

* **GUI för skrivbordet** — inget grafiskt gränssnitt; all interaktion sker via CLI eller Python-SDK
* **Bildvisare** — ingen interaktiv bildvisare, rutnätsvy eller kartmarkörer
* **Visuell projektledning** — projekt skapas och drivs via CLI-kommandon och SDK-anrop (själva hårdvaran — kameror, sensorer, bildinsamling — förblir fullt styrbar från terminalen)***

## Licenskrav

Åtkomst till CLI och SDK kräver en **betald Chloros+-nivå — Copper eller högre**(Copper, Bronze, Silver, Gold). Den kostnadsfria**Iron**-nivån har ingen åtkomst till CLI/SDK. Begränsningen tillämpas av backend-systemet, inte bara av CLI:

| Situation | Backend-svar |
| --- | --- |
| Inte inloggad | `401` med `error_code: AUTH_REQUIRED` |
| Inloggad på den kostnadsfria nivån Iron | `403` med `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` fungerar på alla nivåer — det är den enda vägen som är undantagen från kontrollen — så orsaken till ett avslag är alltid synlig.

***

## Komma igång på Linux

1. **Installera Chloros** — se [Linux-installation](linux-installation.md) för installationen av `.deb`
2. **Kontrollera** — `chloros-cli --version` visar `Chloros CLI 1.3.0`; `chloros-cli selftest` kör den 7-stegsdiagnostiken
3. **Kontrollera Python-SDK** — `.deb` installerar det: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (i en virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Logga in** — `chloros-cli login your@email.com 'your-password'` (en gång per maskin; paketuppgraderingar behåller inloggningen, medan avinstallation av paketet raderar den)
5. **Bearbeta din första dataset** — `chloros-cli process ~/datasets/flight001`

För NVIDIA Jetson, se den särskilda [NVIDIA Jetson-guiden](nvidia-jetson-guide.md) för plattformsspecifik konfiguration, termiskt beteende och driftsättning.

***

## Nästa steg

* [Installation på Linux](linux-installation.md) — detaljerad installation, filplatser och felsökning för amd64 och arm64
* [NVIDIA Jetson-guide](nvidia-jetson-guide.md) — Jetson-specifik konfiguration, minnes- och värmebeteende samt fältinstallation
* [CLI: Kommandorad](../CLI.md) — guiden för CLI
* [API: Python SDK](../api-python-sdk.md) — guiden för SDK
* [CLI-referens](../reference/cli-reference.md) och [SDK-referens](../reference/sdk-reference.md) — uttömmande listor över kommandon/API för version 1.3.0
* [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md) — hur Chlorosanpassar sig till din hårdvara

{% hint style="info" %}
**Att läsa denna handbok programmatiskt.** Varje sida tillhandahålls även som rå Markdown på sin egen URL plus `.md` (till exempel `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), och ett index över hela handboken publiceras på [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
