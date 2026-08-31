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
| **Grafikkort**    | Kompatibelt med DirectX 11                                | NVIDIA-grafikkort med minst 4 GB VRAM                            |
| **Lagringsutrymme**          | 6 GB ledigt utrymme                                       | SSD med minst 10 GB ledigt utrymme                            |
| **Skärm**          | 1920x1080                                            | 2560x1440 eller högre                                  |
| **Internet**         | Krävs för \[valfri] aktivering av Chloros+-licensen | Krävs för \[valfri] aktivering av Chloros+-licensen |

#### Linux amd64 (x86\_64)

| Krav       | Minsta                    | Rekommenderat               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribution**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processor**     | x86\_64 (Intel/AMD)        | Intel Core i7 eller bättre   |
| **Minne (RAM)**  | 8 GB                        | 16 GB eller mer              |
| **Grafikkort** | Inget (bearbetas av CPU)      | NVIDIA-grafikkort med 4 GB eller mer VRAM |
| **Lagringsutrymme**       | 2 GB ledigt utrymme             | SSD med minst 10 GB ledigt       |
| **Python**        | Python 3.7+ (för SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Krav      | Minsta                      | Rekommenderat                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plattform**     | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **Minne (RAM)** | 8 GB (delat mellan GPU och CPU)         | 16 GB+ delat                    |
| **Lagringsutrymme**      | 2 GB ledigt utrymme               | NVMe SSD med 10 GB+ ledigt        |
| **Python**       | Python 3.7+ (för SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-acceleration**: Användare av Chloros+ med NVIDIA-GPU:er kan använda CUDA-acceleration för betydligt snabbare bearbetning. Detta fungerar både på Windows (stationära GPU:er) och Linux (stationära GPU:er och NVIDIA Jetson). Användare av Chloros+ får dessutom tillgång till multitrådad bearbetning för maximal hastighet.
{% endhint %}

***

## Ladda ner Chloros

### Senaste stabila versionen: Version 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Ladda ner Chloros för Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Ladda ner Chloros för Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Ladda ner Chloros för Linux arm64 / Jetson (.deb)</a>

#### Windows-installationsprogram (GUI + CLI + Backend)

* **Filtyp**: .exe (Windows-installationsprogram)**Installationssteg:**

1. Ladda ner ovanstående .exe-fil
2. Dubbelklicka på installationsprogrammet för att påbörja installationen
3. Följ anvisningarna i installationsguiden
4. Välj installationskatalog (standard: `C:\Program Files\MAPIR\Chloros\`)
5. Slutför installationen och starta Chloros eller Chloros CLI
6. Logga in med ditt [MAPIR Cloud Chloros+-konto](https://cloud.mapir.camera/pricing) (eller fortsätt med gratisversionen)

{% hint style="success" %}
Installationsprogrammet lägger automatiskt till `chloros-cli` i systemets PATH för åtkomst via kommandoraden.
{% endhint %}

#### Linux amd64 (.deb-paket — CLI + backend)

* **Filtyp**: .deb (Debian/Ubuntu-paket)
* **Arkitektur**: x86_64 (amd64)

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

Se [Linux-installation](linux/linux-installation.md) för detaljerade installationsanvisningar och [NVIDIA Jetson-guide](linux/nvidia-jetson-guide.md) för Jetson-specifik vägledning.

#### Python SDK (alla plattformar)

Varje installationsprogram innehåller ett matchande `chloros_sdk`-hjul, så att SDK-versionen alltid matchar det installerade GUI/CLI/backend. På Windows installerar installationsprogrammet det automatiskt i ditt system Python; på Linux placerar `.deb` hjulet i `/usr/lib/chloros/sdk/` och visar installationskommandot:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

För värdar som endast använder pip (inget Chloros-paket installerat) finns SDK även på PyPI:

```bash
pip install chloros-sdk
```

Se [API : Python SDK](api-python-sdk.md) och [SDK-referensen](reference/sdk-reference.md) för dokumentation.

{% hint style="info" %}
**Linux-användare**: Paketet `.deb` installerar CLI och backend. Det finns inget grafiskt gränssnitt för Linux — all interaktion sker via CLI eller SDK.
{% endhint %}

***

## Ytterligare resurser

### Python SDK

För utvecklare och automatiseringsarbetsflöden, installera Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentation**: [API: Python SDK](api-python-sdk.md)**Krav**: Chloros måste vara installerat (Windows-installationsprogrammet eller Linux `.deb`-paketet), inloggning med Chloros+-licens krävs***

## Vad ingår

### Windows-installationsprogram

* ✅ **Chloros GUI** – Grafiskt gränssnitt med full funktionalitet
* ✅ **Chloros CLI** - Kommandoradsgränssnitt (kräver Chloros+-licens)
* ✅ **Chloros Backend** – Bearbetningsmotor
* ✅ **Kameraprofiler** – Förkonfigurerade MAPIR-kameramallar

### Linux .deb-paket

* ✅ **Chloros CLI** – Kommandoradsgränssnitt (kräver Chloros+-licens)
* ✅ **Chloros-backend** – Bearbetningsmotor
* ✅ **Kameraprofiler** – Förkonfigurerade MAPIR-kameramallar
* ❌ Inget grafiskt gränssnitt — Linux är endast en headless CLI/SDK

### Python SDK (pip, alla plattformar)

* ✅ **Chloros SDK** – Python API (kräver Chloros+-licens)***

## Uppgradera till Chloros+

Få tillgång till avancerade funktioner med ett Chloros+-abonnemang:

* 🚀 **Flertrådig bearbetning** – Bearbeta bilder parallellt
* ⚡ **GPU-acceleration (CUDA)** – Utnyttja kraften i NVIDIA-GPU:er
* 💻 **CLI-åtkomst** – Automatisera med kommandoradsverktyg
* 🐍 **Python SDK** – Programmerad åtkomst till API
* 📱 **Flera enheter** – Använd på 2–10+ enheter (beroende på abonnemang)
* **🐻 Avancerad texturmedveten debayermetod** – en högkvalitativ kantmedveten debayering kombinerad med en AI/ML-brusreduceringsmodell som tar bort nästan allt debayeringbrus.
* 🧮 **Anpassade formler** – Skapa anpassade multispektrala index

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Visa Chloros+-abonnemang och priser</a></p>***

## Installationshjälp

### Felsökning

**Installationen misslyckas med följande felmeddelande:**

* Se till att du har administratörsrättigheter
* Inaktivera antivirusprogrammet tillfälligt
* Kontrollera att du uppfyller minimikraven för systemet

**Programmet startar inte (Windows):**

* Kontrollera att Windows 10/11 (64-bitars) är installerat
* Uppdatera grafikkortdrivrutinerna
* Kontrollera händelsevisaren för Windows för information om felet
* Kontakta supporten med felloggarna

**CLI startar inte (Linux):**

* Kontrollera att paketet `.deb` är korrekt installerat: `dpkg -l | grep chloros`
* Kontrollera behörigheter: `sudo chmod +x /usr/bin/chloros-cli`
* Kör diagnostik: `chloros-cli selftest`
* Kontrollera om det saknas bibliotek: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problem med licensaktivering:**

* Se till att internetanslutningen är aktiv
* Verifiera inloggningsuppgifterna på [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontrollera att brandväggen inte blockerar Chloros
* Se [Chloros+ Inloggning](chloros+-login.md) för detaljerade instruktioner

### Få support

Behöver du hjälp med installation eller konfiguration?

* 📧 **E-post**: info@mapir.camera
* 🌐 **Webbplats**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentation**: [Kom igång](./)
* ❓ **FAQ**: [Vanliga frågor](faq.md)***

## Programuppdateringar

Chloros söker efter uppdateringar, meddelar när en ny version finns tillgänglig och länkar till denna nedladdningssida – du uppdaterar genom att köra det nya signerade installationsprogrammet. Dina inställningar och projekt bevaras vid uppdateringar. På Linux och Jetson söker `chloros-cli update` efter en nyare version och erbjuder att ladda ner och installera motsvarande `.deb` (detta kommando finns endast i Linux).

***

## Ändringslogg**Version 1.2.0 (senaste)**— se**Vad är nytt i Chloros 1.2.0** på sidan [Kom igång](./) för en fullständig lista över funktioner.

<details>

<summary>Version 1.0.5</summary>

**Utgivningsdatum: 10 februari 2026**

**Nya funktioner*** **Texturmedveten debayermetod \[Endast Chloros+] –** Texturmedveten använder en högkvalitativ kantmedveten debayer i kombination med en AI/ML-brusreduceringsmodell som tar bort nästan allt debayeringbrus.
* **Stöd för T4P-kalibreringsmål*** **Snabbare GPU-bearbetning i Chloros+, bättre minneshantering**

**Felkorrigeringar*** Helt nytt användargränssnitt (GUI), bör nu fungera på alla Windows-datorer.

</details>

<details>

<summary>Version 1.0.4</summary>

**Utgivningsdatum: 5 januari 2026**

**Nya funktioner*** **Växla mellan bild och metadata**: En ny växlingsknapp har lagts till i filbläddraren för att visa den valda bildens metadata i en tabell istället för i bildrutnätet
* **Zoomreglage för bildrutnät**: Nytt reglage i användargränssnittet för att justera miniatyrstorleken (stöder även CTRL + mushjulet)
* **Exportknappar för bildrutnätet**: Knappar i den översta raden för att växla mellan miniatyrer i JPG-format och bearbetade exportfiler (Targets, Reflectance, Index, LUT)
* **Kartfliken**: Ny interaktiv 2D-karta som visar GPS-markörer för bildernas positioner
  * Stöder Google Maps och ESRI-kartrutor (väljer automatiskt bästa karttjänst baserat på tillgänglighet vid aktuell zoomnivå)
  * Förhandsgranskning av miniatyrbilder vid muspekning på kartmarkörer

**Felkorrigeringar*** Förbättrat stöd för installation av Chloros på datorer med andra språk än engelska

</details>

<details>

<summary>Version 1.0.3</summary>

**Utgivningsdatum: 20 december 2025**

**Nya funktioner*** Första lanseringen

**Förbättringar*** Första lanseringen

**Buggfixar*** Första lanseringen

**Kända problem*** Första lanseringen

</details>***

## Licensavtal**Upphovsrättsskyddad programvara** – Copyright (c) 2026 MAPIR Inc.

Obehörig användning, distribution eller modifiering är förbjuden.

**Gratisversion**: Tillgänglig för privat och kommersiellt bruk med begränsade funktioner**Chloros+**: Prenumerationsbaserad licens för avancerade funktioner och kommersiell användning
