---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Ladda ner

Ladda ner den senaste versionen av Chloros för att komma igång med multispektral bildbehandling.

### Systemkrav

#### Windows

| Krav          | Minsta                                              | Rekommenderat                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operativsystem** | Windows 10 (64-bitars)                                  | Windows 11 (64-bitars)                                  |
| **Processor**        | Intel Core i5 eller motsvarande                          | Intel Core i7 eller bättre                              |
| **Minne (RAM)**     | 8 GB                                                  | 16 GB eller mer                                         |
| **Grafikkort**    | Kompatibelt med DirectX 11                                | NVIDIA GPU med 4 GB+ VRAM                            |
| **Lagringsutrymme**          | 6 GB ledigt utrymme                                       | SSD med 10 GB+ ledigt utrymme                            |
| **Skärm**          | 1920x1080                                            | 2560x1440 eller högre                                  |
| **Internet**         | Krävs för \[valfri] Chloros+ licensaktivering | Krävs för \[valfri] Chloros+ licensaktivering |

#### Linux amd64 (x86\_64)

| Krav       | Minsta                    | Rekommenderat               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribution**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Processor**     | x86\_64 (Intel/AMD)        | Intel Core i7 eller bättre   |
| **Minne (RAM)**  | 8 GB                        | 16 GB eller mer              |
| **Grafikkort** | Inget (CPU-bearbetning)      | NVIDIA GPU med 4 GB+ VRAM |
| **Lagringsutrymme** | 2 GB ledigt utrymme             | SSD med 10 GB+ ledigt       |
| **Python**        | Python 3.7+ (för SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Krav      | Minsta                      | Rekommenderat                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plattform**     | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **Minne (RAM)** | 8 GB (delat GPU/CPU)         | 16 GB+ delat                    |
| **Lagring**      | 2 GB ledigt utrymme               | NVMe SSD med 10 GB+ ledigt        |
| **Python**       | Python 3.7+ (för SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-acceleration**: Användare av Chloros+ med NVIDIA-GPU:er kan använda CUDA-acceleration för betydligt snabbare bearbetning. Detta fungerar både på Windows (stationära GPU:er) och Linux (stationära GPU:er och NVIDIA Jetson). Chloros+-användare får dessutom flertrådig bearbetning för maximal hastighet.
{% endhint %}

***

## Ladda ner Chloros

### Senaste stabila versionen (23 mars 2026): Version 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Ladda ner Chloros för Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Ladda ner Chloros för Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Ladda ner Chloros för Linux arm64 / Jetson (.deb)</a>

#### Windows Installer (GUI + CLI + Backend)

* **Filtyp**: .exe (Windows-installationsprogram)**Installationssteg:**

1. Ladda ner ovanstående .exe-fil
2. Dubbelklicka på installationsprogrammet för att starta installationen
3. Följ anvisningarna i installationsguiden
4. Välj installationskatalog (standard: `C:\Program Files\[USER]\Chloros\`)
5. Slutför installationen och starta Chloros eller Chloros CLI
6. Logga in med ditt [MAPIR Cloud Chloros+-konto](https://cloud.mapir.camera/pricing) (eller fortsätt med gratisversionen)

{% hint style="success" %}
Installationsprogrammet lägger automatiskt till `chloros-cli` i systemets PATH för åtkomst via kommandoraden.
{% endhint %}

#### Linux amd64 (.deb-paket — CLI + Backend)

* **Filtyp**: .deb (Debian/Ubuntu-paket)
* **Arkitektur**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb-paket — CLI + Backend)

* **Filtyp**: .deb (JetPack 6)
* **Arkitektur**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Se [Linux Installation](linux/linux-installation.md) för detaljerade installationsinstruktioner och [NVIDIA Jetson Guide](linux/nvidia-jetson-guide.md) för Jetson-specifik vägledning.

#### Python SDK (Alla plattformar)

```bash
pip install chloros-sdk
```

Se [API : Python SDK](api-python-sdk.md) för dokumentation.

{% hint style="info" %}
**Linux-användare**: Paketet `.deb` installerar CLI och backend. Python SDK installeras separat via pip. Det finns inget grafiskt gränssnitt för Linux — all interaktion sker via CLI eller SDK.
{% endhint %}

***

## Ytterligare resurser

### Python SDK

För utvecklare och automatiseringsarbetsflöden, installera Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentation**: [API: Python SDK](api-python-sdk.md)**Krav**: Chloros måste vara installerat (Windows-installationsprogram eller Linux `.deb`-paket), Chloros+ licensinloggning krävs***

## Vad ingår

### Windows-installationsprogram

* ✅ **Chloros GUI** – Grafiskt gränssnitt med full funktionalitet
* ✅ **Chloros CLI** - Kommandoradsgränssnitt (kräver Chloros+ licens)
* ✅ **Chloros Backend** - Bearbetningsmotor
* ✅ **Kameraprofil** - Förkonfigurerade MAPIR kameramallar

### Linux .deb-paket

* ✅ **Chloros CLI** - Kommandoradsgränssnitt (kräver Chloros+-licens)
* ✅ **Chloros Backend** - Bearbetningsmotor
* ✅ **Kameraprofiler** - Förkonfigurerade MAPIR-kameramallar
* ❌ Inget GUI — Linux är endast headless CLI/SDK

### Python SDK (pip, alla plattformar)

* ✅ **Chloros SDK** - Python API (kräver Chloros+-licens)***

## Uppgradera till Chloros+

Lås upp avancerade funktioner med en Chloros+-prenumeration:

* 🚀 **Multitrådad bearbetning** – Bearbeta bilder parallellt
* ⚡ **GPU (CUDA)-acceleration** – Utnyttja kraften i NVIDIA-GPU:er
* 💻 **CLI-åtkomst** – Automatisera med kommandoradsverktyg
* 🐍 **Python SDK** – Programmerad API-åtkomst
* 📱 **Flera enheter** – Använd på 2–10+ enheter (beroende på plan)
* **🐻 Avancerad texturmedveten debayer-metod** – en högkvalitativ kantmedveten debayer kombinerad med en AI/ML-brusreduceringsmodell som tar bort nästan allt debayeringsbrus.
* 🧮 **Anpassade formler** – Skapa anpassade multispektrala index

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Visa Chloros+-abonnemang och priser</a></p>***

## Installationshjälp

### Felsökning

**Installationen misslyckas med felmeddelandet:**

* Se till att du har administratörsrättigheter
* Inaktivera antivirusprogrammet tillfälligt
* Kontrollera att du uppfyller minimikraven för systemet

**Programmet startar inte (Windows):**

* Kontrollera att Windows 10/11 (64-bit) är installerat
* Uppdatera grafikkortets drivrutiner
* Kontrollera Windows Händelseloggen för felinformation
* Kontakta supporten med felloggar

**CLI startar inte (Linux):**

* Kontrollera att `.deb`-paketet är korrekt installerat: `dpkg -l | grep chloros`
* Kontrollera behörigheter: `sudo chmod +x /usr/bin/chloros-cli`
* Kör diagnostik: `chloros-cli selftest`
* Kontrollera om bibliotek saknas: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problem med licensaktivering:**

* Se till att internetanslutningen är aktiv
* Kontrollera inloggningsuppgifterna på [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontrollera att brandväggen inte blockerar Chloros
* Se [Chloros+ Inloggning](chloros+-login.md) för detaljerade instruktioner

### Få support

Behöver du hjälp med installation eller konfiguration?

* 📧 **E-post**: info@mapir.camera
* 🌐 **Webbplats**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentation**: [Kom igång](./)
* ❓ **FAQ**: [Vanliga frågor](faq.md)***

## Ändringslogg

<details>

<summary>Version 1.1.0 (Senaste)</summary>

**Utgivningsdatum: mars 2026**

**Nya funktioner*** **Linux-stöd** — Inbyggt stöd för CLI och SDK för Linux amd64 (x86\_64) och arm64 (NVIDIA Jetson JetPack 6). Installera via `.deb`-paket.
* **Stöd för NVIDIA Jetson** — Optimerad bearbetning för Jetson Nano, Orin Nano, Orin NX och AGX Orin-kantdatorer.
* **Dynamisk beräkningsanpassning** — Automatisk hårdvarudetektering och optimering av bearbetningsstrategi. Chloros anpassar sig till din hårdvara, från en Jetson Nano till en arbetsstation med flera GPU:er.
* **4-tråds bearbetningspipeline** — Samtidiga trådar för detektering, kalibrering, bearbetning och export med dynamisk GPU-minnesallokering.
* **Nya CLI-kommandon** — `selftest` (systemdiagnostik) och `update` (Linux-uppdateringshantering).
* **Nya CLI-processflaggor** — `--debayer` (standard/texturmedveten), `--indices` (ange index), `--target` (sök först i målunderkatalogen för snabbare detektering).
* **Nya GUI-menyalternativ** — Lägg till filer, Lägg till mapp och Starta/stoppa bearbetning är nu tillgängliga från huvudmenyns rullgardinsmeny.**Förbättringar**

* Automatisk upptäckt av plattformsoberoende backend (Windows och Linux-sökvägar)
* Förbättrad SDK `get_status()` med spårning av framsteg per tråd
* Nya SDK-undantag: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Värmehantering och adaptiv strypning för NVIDIA Jetson
* Automatisk minneshantering med OOM-fallback till kaklad GPU-bearbetning

</details>

<details>

<summary>Version 1.0.5</summary>

**Utgivningsdatum: 10 februari 2026**

**Nya funktioner*** **Texture Aware Debayer-metod \[Endast Chloros+] -** Texture Aware använder en högkvalitativ kantmedveten debayer i kombination med en AI/ML-brusreduceringsmodell som tar bort nästan allt debayering-brus.
* **Stöd för T4P-kalibreringsmål*** **Snabbare Chloros+ GPU-bearbetning, bättre minneshantering**

**Buggfixar*** Helt nytt gränssnitt (GUI), bör nu fungera på alla Windows-datorer.

</details>

<details>

<summary>Version 1.0.4</summary>

**Utgivningsdatum: 5 januari 2026**

**Nya funktioner*** **Växla mellan bild och metadata**: Lagt till en växlingsknapp i filbläddraren för att visa den valda bildens metadata i en tabell istället för i bildrutnätet
* **Zoomreglage för bildrutnät**: Nytt UI-reglage för att justera miniatyrstorleken (stöder även CTRL + mushjul)
* **Exportknappar för bildrutnätet**: Knappar i den översta raden för att växla miniatyrer från JPG till bearbetade exporter (Mål, Reflektans, Index, LUT)
* **Kartflik**: Ny interaktiv 2D-karta som visar bildernas GPS-markörer
  * Stöder Google Maps och ESRI-kartrutor (väljer automatiskt bästa ruttjänst baserat på tillgänglighet för zoomnivå)
  * Förhandsgranskning av miniatyrbilder vid muspekning på kartmarkörer

**Buggfixar*** Förbättrat stöd för installation av Chloros på datorer med andra språk än engelska

</details>

<details>

<summary>Version 1.0.3</summary>

**Utgivningsdatum: 20 december 2025**

**Nya funktioner*** Första lansering

**Förbättringar*** Första lansering

**Buggfixar*** Första lansering

**Kända problem*** Första lansering

</details>***

## Licensavtal**Upphovsrättsskyddad programvara** – Copyright (c) 2026 MAPIR Inc.

Obehörig användning, distribution eller modifiering är förbjuden.

**Gratisversion**: Tillgänglig för privat och kommersiellt bruk med begränsade funktioner**Chloros+**: Prenumerationsbaserad licens för avancerade funktioner och kommersiell användning
