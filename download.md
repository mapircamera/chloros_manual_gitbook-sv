---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Ladda ner

Ladda ner den senaste versionen av Chloros för att komma igång med bearbetning av multispektrala bilder.

### Systemkrav

#### Windows


| Krav          | Minsta krav                                              | Rekommenderat                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operativsystem** | Windows 10 (64-bitars)                                  | Windows 11 (64-bitars)                                  |
| **Processor**        | Intel Core i5 eller motsvarande                          | Intel Core i7 eller bättre                              |
| **Minne (RAM)**     | 8 GB                                                  | 16 GB eller mer                                         |
| **Grafikkort**    | DirectX 11-kompatibelt                                | NVIDIA-grafikkort med minst 4 GB VRAM                            |
| **Lagringsutrymme**          | 6 GB ledigt utrymme                                       | SSD med minst 10 GB ledigt utrymme                            |
| **Skärm**          | 1920x1080                                            | 2560x1440 eller högre                                  |
| **Internet**         | Krävs för \[valfri] aktivering av Chloros+-licens | Krävs för \[valfri] aktivering av Chloros+-licens |

#### Linux amd64 (x86\_64)

| Krav       | Minsta                    | Rekommenderat               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribution**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processor**     | x86\_64 (Intel/AMD)        | Intel Core i7 eller bättre   |
| **Minne (RAM)**  | 8 GB                        | 16 GB eller mer              |
| **Grafikkort** | Inget (bearbetas av CPU)      | NVIDIA-grafikkort med minst 4 GB VRAM |
| **Lagringsutrymme**       | 2 GB ledigt utrymme             | SSD med minst 10 GB ledigt       |
| **Python**        | Python 3.7+ (för SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Krav      | Minsta                      | Rekommenderat                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plattform**     | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **Minne (RAM)** | 8 GB (delat mellan GPU och CPU)         | 16 GB+ delat                    |
| **Lagringsutrymme**      | 2 GB ledigt utrymme               | NVMe SSD med minst 10 GB ledigt        |
| **Python**       | Python 3.7+ (för SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-acceleration**: Chloros+-användare med NVIDIA-grafikprocessorer kan använda CUDA-acceleration för betydligt snabbare bearbetning. Detta fungerar med NVIDIA-grafikprocessorer för stationära datorer på Windows och Linux amd64, samt på NVIDIA Jetson med JetPack 6. Chloros+-användare får dessutom tillgång till multitrådad bearbetning för maximal hastighet.
{% endhint %}

***

## Ladda ner Chloros


### Senaste stabila versionen: Version 1.3.0 (11 november 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Ladda ner Chloros för Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Ladda ner Chloros för Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Ladda ner Chloros för Linux arm64 / Jetson (.deb)</a>


#### Windows-installationsprogram (GUI + CLI + Backend)

* **Filtyp**: .exe (Windows-installationsprogram)

**Installationssteg:**

1. Ladda ner filen .exe ovan
2. Dubbelklicka på installationsprogrammet för att starta installationen
3. Följ anvisningarna i installationsguiden
4. Välj installationskatalog (standard: `C:\Program Files\Chloros\`)
5. På sidan **Välj ytterligare uppgifter**ska du se till att båda alternativen under**Systeminställningar** är markerade (rekommenderas – se nedan)
6. Slutför installationen och starta Chloros eller Chloros CLI
7. Logga in med ditt [MAPIR Cloud Chloros+-konto](https://cloud.mapir.camera/pricing) (eller fortsätt med gratisversionen)

**Alternativ för systeminställningar** (båda markerade som standard):

| Alternativ, enligt formuleringen i installationsprogrammet | Vad det gör | Avaktivera tyst installation |
| --- | --- | --- |
| **Aktivera långa sökvägar i Windows** (gör det möjligt att placera projektmappar djupare än 260 tecken; rekommenderas) | Ställer in Windows-inställningen `LongPathsEnabled` för hela datorn till 1. Windows levereras med denna inställning avstängd, vilket innebär att ingen filväg får vara längre än 260 tecken; Chloros-produkter placeras upp till cirka 130 tecken under projektmappen, så om inställningen för långa sökvägar är avstängd nekas en djupare projektmapp. Träder i kraft för program som startas efter installationen. Inställningen förblir aktiverad även efter avinstallation, eftersom andra program kan vara beroende av den. | `/MERGETASKS="!longpaths"` |
| **Undantag Chloros-backend från Microsoft Defender-skanning** (mycket snabbare kameraupptagning; rekommenderas) | Lägger till ett undantag för Microsoft Defender-processen för `chloros-backend.exe` i installationsmappen, så att realtidsskyddet inte skannar varje fil som backend skriver, oavsett var den skrivs. Vid höga inspelningshastigheter utgör skanningen cirka 30 % av inspelningshastigheten (77,7 mot 52,7 inspelningar per sekund vid kontinuerlig inspelning med 9 kameror). Inget annat program undantas, och undantaget tas bort vid avinstallation. Om Defender saknas eller är låst av policy misslyckas detta steg utan meddelande och installationen fortsätter. | `/MERGETASKS="!defenderexclusion"` |

En tyst installation (`/SILENT` eller `/VERYSILENT`) tillämpar båda alternativen om du inte väljer bort dem med det visade värdet `/MERGETASKS`.

{% hint style="success" %}
Installationsprogrammet lägger till mappen CLI (`C:\Program Files\Chloros\cli\`) på ditt system `PATH` en gång — vid ominstallation eller uppgradering läggs inte längre en dubblett till, och dubbletter som tidigare installationsprogram har lagt till tas bort. Öppna en ny terminal efter installationen för att använda `chloros-cli`. Installationsprogrammet lägger också till två regler i Windows-brandväggen för backend, i varje nätverksprofil: **Chloros Backend (UDP-In)**, för kameraströmmar, PTP, mDNS och DAQ-E-dataströmmen, samt**Chloros Backend (TCP-In)**, begränsad till det lokala subnätet, för en DAQ-E-firmwareuppdatering, där sensorn ansluter tillbaka till den här datorn för att hämta bilden. Båda tas bort vid avinstallation.
{% endhint %}

#### Linux amd64 (.deb-paket — CLI + backend)

* **Filtyp**: .deb (Debian/Ubuntu-paket)
* **Arkitektur**: x86\_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (.deb-paket — CLI + backend)

* **Filtyp**: .deb (JetPack 6)
* **Arkitektur**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Använd det filnamn du faktiskt laddade ner och behåll `./` så att `apt` installerar den filen istället för att söka efter ett paket efter namn. Installera med `apt install`, inte med `dpkg -i`: apt installerar även Python-SDK:s krav, som paketet rekommenderar snarare än kräver, och `dpkg -i` hoppar över dem. Sedan version 1.3.0 installeras backend och CLI-verktyget uppackade under `/usr/lib/chloros/`, så att de startar utan att först behöva packa upp sig själva.

Jetson-paketet använder JetPack 6:s CUDA-version av PyTorch, så bearbetningen, inklusive den texturmedvetna debayern, körs på Jetsons GPU. Det använder CUDA- och cuDNN-biblioteken som JetPack 6 installerar (`sudo apt install nvidia-jetpack` på en image som saknar dem).

Se [Linux-installation](linux/linux-installation.md) för detaljerade installationsanvisningar och [NVIDIA Jetson-guide](linux/nvidia-jetson-guide.md) för Jetson-specifik vägledning.

#### Python SDK (alla plattformar)

Varje installationsprogram innehåller ett matchande `chloros_sdk`-wheel, så att SDK-versionen alltid stämmer överens med den installerade GUI:n/CLI:n/backenden. I Windows installerar installationsprogrammet det automatiskt i ditt system-Python. I Linux installerar `.deb` SDK:et för själva systemet `python3` (i `/usr/lib/python3/dist-packages`), så att `import chloros_sdk` fungerar direkt efter installationen, utan pip och utan nätverk. Endast en virtualenv, som inte ser systemets paket, behöver det medföljande hjulet installerat i sig:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

För värdar som endast använder pip (inget Chloros-paket installerat; SDK driver då en backend på en annan maskin) finns SDK även på PyPI:

```bash
pip install chloros-sdk
```

Se [API: Python SDK](api-python-sdk.md) och [SDK-referensen](reference/sdk-reference.md) för dokumentation.

{% hint style="info" %}
**Linux-användare**: Paketet `.deb` installerar CLI, backend-delen och Python-SDK:et. Det finns inget grafiskt gränssnitt för Linux – all interaktion sker via CLI eller SDK.
{% endhint %}

***

## Ytterligare resurser

### Python-SDK


För utvecklare och automatiseringsflöden ingår Chloros-Python-SDK i varje installationsprogram: Windows-installationsprogrammet installerar det i din Python, och Linux-paketet `.deb` installerar det för systemet `python3`. På en dator utan Chloros-paket (en som styr en backend på en annan dator) installerar du det från PyPI:

```bash
pip install chloros-sdk
```

**Dokumentation**: [API: Python SDK](api-python-sdk.md)

**Krav**: en Chloros-backend att styra (Windows-installationsprogrammet eller Linux-paketet `.deb` installerar en sådan) samt ett Chloros+-konto på en betald nivå

***

## Vad ingår

### Windows-installationsprogram

* ✅ **Chloros GUI** – fullt utrustat grafiskt gränssnitt
* ✅ **Chloros CLI** – kommandoradsgränssnitt (kräver Chloros+-licens)
* ✅ **Chloros-backend** – bearbetningsmotor
* ✅ **Python SDK** – installeras i din Python om en sådan finns (kräver Chloros+-licens)
* ✅ **Kameraprofiler** – Förkonfigurerade MAPIR-kameramallar

### Linux-.deb-paket

* ✅ **Chloros CLI** – Kommandoradsgränssnitt (kräver Chloros+-licens)
* ✅ **Chloros Backend** – Bearbetningsmotor
* ✅ **Python SDK** – installeras för systemet `python3` (kräver Chloros+-licens)
* ✅ **Kameraprofiler** – Förkonfigurerade MAPIR-kameramallar
* ❌ Inget grafiskt gränssnitt — Linux är endast CLI/SDK utan grafiskt gränssnitt

### Python-SDK (PyPI, för maskiner utan Chloros-paket)

* ✅ **Chloros-SDK** – Python-API (kräver Chloros+-licens)

***

## Uppgradera till Chloros+

Få tillgång till avancerade funktioner med ett Chloros+-abonnemang:

* 🚀 **Flertrådig bearbetning** – Bearbeta bilder parallellt
* ⚡ **GPU (CUDA)-acceleration** – Utnyttja kraften i NVIDIA-GPU:er
* 💻 **Tillgång till CLI** – Automatisera med kommandoradsverktyg
* 🐍 **Python-SDK** – Programmerbar API-åtkomst
* 📱 **Flera enheter** – Använd på 2–10+ enheter (beroende på abonnemang)
* **🐻 Avancerad texturmedveten debayer-metod** – en högkvalitativ kantmedveten debayer kombinerad med en AI/ML-brusreduceringsmodell som tar bort nästan allt debayeringsbrus.
* 🧮 **Anpassade formler** – Skapa anpassade multispektrala index

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Se Chloros+ abonnemang och priser</a></p>

***

## Hjälp med installation

### Felsökning

**Installationen misslyckas med följande felmeddelande:**

* Se till att du har administratörsrättigheter
* Inaktivera antivirusprogrammet tillfälligt
* Kontrollera att du uppfyller minimikraven för systemet

**Programmet startar inte (Windows):**

* Kontrollera att Windows 10/11 (64-bitars) är installerat
* Uppdatera grafikkortsdrivrutinerna
* Kontrollera Windows Händelsevisare för felinformation
* Kontakta supporten med felloggarna

**CLI startar inte (Linux):**

* Kontrollera att paketet `.deb` är korrekt installerat: `dpkg -l | grep chloros`
* Kontrollera behörigheter: `sudo chmod +x /usr/bin/chloros-cli`
* Kör diagnostik: `chloros-cli selftest`
* Kontrollera om det saknas bibliotek: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (sedan version 1.3.0 är `/usr/lib/chloros/chloros-backend` ett litet startskript; själva programmet finns i `/usr/lib/chloros/backend/`)

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
* ❓ **Vanliga frågor**: [Vanliga frågor](faq.md)

***

## Programuppdateringar

Chloros söker efter uppdateringar, meddelar när en ny version finns tillgänglig och länkar till denna nedladdningssida – du uppdaterar genom att köra det nya signerade installationsprogrammet. Dina inställningar, projekt och inloggningsuppgifter bevaras vid uppdateringar. På Linux och Jetson söker `chloros-cli update` efter en nyare version och erbjuder att ladda ner och installera motsvarande `.deb` (detta kommando finns endast för Linux).

***

## Ändringslogg**Version 1.3.0 (Senaste, 11 november 2026)**— se**Vad är nytt i Chloros

1.3.0** på sidan [Kom igång](./) för en fullständig lista.**Version 1.2.0**— se**Vad är nytt i Chloros 1.2.0** på sidan [Komma igång](./).

<details>

<summary>Version 1.0.5</summary>

**Utgivningsdatum: 10 februari 2026**

**Nya funktioner*** **Texturmedveten debayer-metod \[Endast Chloros+] –** Texturmedveten använder en högkvalitativ kantmedveten debayer i kombination med en AI/ML-brusreduceringsmodell som tar bort nästan allt debayeringsbrus.
* **Stöd för T4P-kalibreringsmål*** **Snabbare Chloros+ GPU-bearbetning, bättre minneshantering**

**Buggfixar*** Helt nytt gränssnitt (GUI), bör nu fungera på alla Windows-datorer.

</details>

<details>

<summary>Version 1.0.4</summary>

**Utgivningsdatum: 5 januari 2026**

**Nya funktioner*** **Växla mellan bild och metadata**: En knapp har lagts till i filbläddraren för att visa den valda bildens metadata i en tabell istället för i bildrutnätet
* **Zoomreglage för bildrutnät**: Nytt reglage i användargränssnittet för att justera miniatyrbildernas storlek (stöder även CTRL + mushjulet)
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

</details>

***

## Licensavtal**Upphovsrättsskyddad programvara** – Copyright (c) 2026 MAPIR Inc.

Obehörig användning, distribution eller modifiering är förbjuden.

**Gratisversion**: Tillgänglig för privat och kommersiellt bruk med begränsade funktioner**Chloros+**: Prenumerationsbaserad licens för avancerade funktioner och kommersiell användning
