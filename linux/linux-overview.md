# Linux – Översikt

Chloros 1.2.0 erbjuder inbyggt stöd för **CLI**och**Python SDK** — multispektral bildbehandling utan skärm, samt styrning av LATTICE-kameror och DAQ-ljussensorer i realtid — på Linux-arbetsstationer, servrar och NVIDIA Jetson-enheter för kantberäkning.

{% hint style="info" %}
**Inget skrivbordsgränssnitt på Linux.**Skrivbordsgränssnittet för Chloros är endast tillgängligt på Windows. Användare av Linux interagerar med Chloros via [CLI](../CLI.md) och [Python SDK](../api-python-sdk.md). `.deb` lägger till en**Chloros CLI** till din applikationsmeny – den öppnar helt enkelt en terminalemulator som kör `chloros-cli`.
{% endhint %}

***

## Plattformsstödmatris

| Funktion | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **GUI för stationära datorer** | Ja | Ej tillämpligt | Nej | Nej |
| **CLI** (`chloros-cli`) | Ja | Ja | Ja | Ja |
| **Python SDK** (`chloros-sdk`) | Ja | Ja | Ja | Ja |
| **Bildbehandlingspipeline** | Ja | Ja | Ja | Ja |
| **LATTICE-kamerastyrning (live)** | Ja (fliken Kameror) | Ja (`chloros-cli lattice`, SDK) | Ja | Ja |
| **DAQ-ljussensorer (live)** | Ja (fliken Ljussensorer) | Ja (`chloros-cli daq pool-*`, SDK) | Ja | Ja |
| **PTP-tidssynkronisering (värden är grandmaster)** | Ja | Ja (`chloros-cli time-sync`) | Ja | Ja |
| **GPU-acceleration (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Texturmedveten debayer** | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamisk beräkningsanpassning** | Ja | Ja | Ja | Ja |
| **Backend som systemtjänst** (`chloros-backend.service`) | Nej | Nej | Ja (opt-in) | Ja (opt-in) |
| **Uppdatering på plats** (`chloros-cli update`) | Nej (kör installationsprogrammet) | Nej (kör installationsprogrammet) | Ja | Ja |***

## Arkitekturer som stöds

| Arkitektur | Beskrivning | Paket |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardprocessorer för stationära datorer/servrar (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM-processorer — NVIDIA Jetson Orin-familjen | `chloros_<version>_arm64_jp6.deb` (JetPack 6-version) |

## Distributioner som stöds av Linux

* **Ubuntu 22.04 LTS eller nyare** (amd64)
* **Debian 12 eller nyare** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin-plattformar)***

## Vad Linux-användare får

* **Chloros CLI** — det fullständiga kommandoradsgränssnittet för batchbearbetning, automatisering och skriptning
* **Chloros Python SDK** — ett programmeringsgränssnitt för Python avsett för forskningspipelines och anpassade verktyg (kan installeras från PyPI och ingår även i `.deb` som ett versionanpassat wheel)
* **LATTICE-kamerastyrning** — upptäck, anslut, konfigurera och ta bilder från LATTICE-kameror och synkroniserade flerkamerasystem via `chloros-cli lattice` och SDK; `.deb` innehåller den Arena SDK-runtime som kamerorna kräver
* **DAQ-ljussensorkontroll** — anslut DAQ-U/M/E-sensorer, strömma kalibrerade spektra och spela in `.daq`-filer via `chloros-cli daq pool-*` och SDK
* **PTP-tidssynkronisering** — Chloros-backend kör den PTP-grandmaster som LATTICE-kameror och DAQ-E-sensorer är slavar till; granska den med `chloros-cli time-sync`, och håll den igång i headless-läge med systemd-enheten `chloros-backend.service` (se [Linux-installation](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projektautomatisering** — kör sparade projekt i bakgrunden med `chloros-cli project` och SDK:s `open_project`
* **GPU-acceleration** — CUDA-accelererad bearbetning på NVIDIA-GPU:er (stationära och Jetson)
* **Dynamisk beräkningsanpassning** — automatisk hårdvarudetektering och val av bearbetningsstrategi, med `CHLOROS_STRATEGY` som en expertutväg
* **Alla bearbetningsfunktioner** — samma bearbetningsflöde som Windows: kalibrering, vignettkorrigering, vegetationsindex och alla exportformat
* **Chloros+-funktioner** — flertrådig (pipelinebaserad) bearbetning, Texture Aware-debayer och anpassade index, med ett betalt Chloros+-abonnemang

## Vad Linux-användare inte får

* **GUI för skrivbordet** — inget grafiskt gränssnitt; all interaktion sker via CLI eller Python SDK
* **Bildvisare** — ingen interaktiv bildvisare, rutnätsvy eller kartmarkörer
* **Visuell projektledning** — projekt skapas och drivs via CLI-kommandon och SDK-anrop (själva hårdvaran — kameror, sensorer, bildinsamling — förblir fullt styrbar från terminalen)***

## Licenskrav

Åtkomst till CLI och SDK kräver en **betald Chloros+-nivå — Copper eller högre**(Copper, Bronze, Silver, Gold). Den kostnadsfria nivån**Iron** har ingen åtkomst till CLI/SDK. Gränsen tillämpas av backend-systemet, inte bara av CLI:

| Situation | Svar från backend |
| --- | --- |
| Inte inloggad | `401` med `error_code: AUTH_REQUIRED` |
| Inloggad på den kostnadsfria Iron-nivån | `403` med `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` fungerar på alla nivåer – det är den enda rutten som är undantagen från gränskontrollen – så orsaken till ett avslag är alltid synlig.

***

## Komma igång med Linux

1. **Installera Chloros** – se [Linux Installation](linux-installation.md) för installationen av `.deb`
2. **Kontrollera** — `chloros-cli --version` skriver ut `Chloros CLI 1.2.0`; `chloros-cli selftest` kör den 7-stegsdiagnostiken
3. **Installera Python och SDK** (valfritt) — `pip install chloros-sdk`
4. **Logga in** — `chloros-cli login your@email.com 'your-password'` (en gång per maskin och igen efter varje paketuppgradering)
5. **Bearbeta din första dataset** — `chloros-cli process ~/datasets/flight001`

För NVIDIA Jetson, se den särskilda [NVIDIA Jetson-guiden](nvidia-jetson-guide.md) för plattformsspecifik konfiguration, termiskt beteende och fältinstallation.

***

## Nästa steg

* [Linux Installation](linux-installation.md) — detaljerad installation, filplatser och felsökning för amd64 och arm64
* [NVIDIA Jetson-guide](nvidia-jetson-guide.md) — Jetson-specifik konfiguration, minnes- och värmebeteende samt fältinstallation
* [CLI : Kommandorad](../CLI.md) — guiden för CLI
* [API : Python SDK](../api-python-sdk.md) — guiden SDK
* [CLI Referens](../reference/cli-reference.md) och [SDK Referens](../reference/sdk-reference.md) – uttömmande listor över kommandon/API för version 1.2.0
* [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md) — hur Chloros anpassar sig till din hårdvara

{% hint style="info" %}
**Att läsa denna handbok programmatiskt.** Varje sida tillhandahålls även som rå Markdown på sin egen URL samt `.md` (till exempel `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), och ett index över hela handboken publiceras på [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
