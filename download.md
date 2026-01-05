---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Ladda ner

Ladda ner den senaste versionen av Chloros för att komma igång med multispektral bildbehandling.

### Systemkrav

| Krav          | Minsta                         | Rekommenderat                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Operativsystem** | Windows 10 (64-bitars)             | Windows 11 (64-bitars)             |
| **Processor**        | Intel Core i5 eller motsvarande     | Intel Core i7 eller bättre         |
| **Minne (RAM)**     | 8 GB                             | 16 GB eller mer                    |
| **Grafikkort**    | DirectX 11-kompatibelt           | NVIDIA GPU med 4 GB+ VRAM       |
| **Lagring**          | 6 GB ledigt utrymme                  | SSD med 10 GB+ ledigt utrymme       |
| **Skärm**          | 1920x1080                       | 2560x1440 eller högre             |
| **Internet**         | Krävs för licensaktivering | Krävs för licensaktivering |

{% hint style=&quot;info&quot; %}
**GPU-acceleration**: Chloros+-användare med NVIDIA GPU:er (4 GB+ VRAM) kan använda CUDA-acceleration för betydligt snabbare bearbetning. Chloros+-användare får också multitrådad bearbetning för maximal hastighet.
{% endhint %}

***

## Ladda ner Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Ladda ner Chloros här</a>

### Senaste stabila versionen

**Chloros Installationsprogram för Windows*** **Version**: 1.0.4
* **Utgivningsdatum**: 5 januari 2026
* **Filstorlek (nedladdning)**: 1,8 GB
* **Filstorlek (installerad)**: 5,7 GB
* **Filtyp**: .exe (Windows-installationsprogram)

#### **Installationssteg:**

1. Ladda ner filen `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Dubbelklicka på installationsprogrammet för att starta installationen
3. Följ anvisningarna i installationsguiden
4. Välj installationskatalog (standard: `C:\Program Files\[USER]\Chloros\`)
5. Slutför installationen och starta Chloros, Chloros (webbläsare) eller Chloros CLI
6. Logga in med ditt [MAPIR Cloud Chloros+-konto](https://cloud.mapir.camera/pricing) (eller fortsätt med gratisversionen)

{% hint style=&quot;success&quot; %}
Installationsprogrammet lägger automatiskt till `chloros-cli` i systemets PATH för åtkomst via kommandoraden.
{% endhint %}

***

## Ytterligare resurser

### Python SDK

För utvecklare och automatiseringsarbetsflöden, installera Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentation**: [API: Python SDK](api-python-sdk.md)**Krav**: Chloros Desktop måste vara installerat, Chloros+ licensinloggning krävs.***

## Vad ingår

Installationen av Chloros innehåller:

* ✅ **Chloros** – Fullfjädrat grafiskt gränssnitt
* ✅ **Chloros (webbläsare)** – Webbaserat gränssnitt för system med lägre specifikationer
* ✅ **Chloros CLI** – Kommandoradsgränssnitt (kräver Chloros+ licens)
* ✅ **Chloros SDK** - Python API (kräver Chloros+ licens)
* ✅ **Kameraprofil** - Förkonfigurerade MAPIR kameramallar***

## Uppgradera till Chloros+

Lås upp avancerade funktioner med ett Chloros+-abonnemang:

* 🚀 **Multitrådad bearbetning** – Bearbeta bilder parallellt
* ⚡ **GPU (CUDA)-acceleration** – Utnyttja kraften i NVIDIA GPU
* 💻 **CLI-åtkomst** – Automatisera med kommandoradsverktyg
* 🐍 **Python SDK** – Programmerbar API-åtkomst
* 📱 **Flera enheter** – Använd på 2–10+ enheter (beroende på plan)
* 🧮 **Anpassade formler** – Skapa anpassade multispektrala index

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Visa Chloros+ Planer och priser</a></p>***

## Hjälp med installation

### Felsökning

**Installationen misslyckas med felmeddelandet:**

* Se till att du har administratörsrättigheter
* Inaktivera tillfälligt antivirusprogrammet
* Kontrollera att du uppfyller minimikraven för systemet

**Programmet startar inte:**

* Prova Chloros (webbläsare) version
* Kontrollera att Windows 10/11 (64-bitars) är installerat
* Uppdatera grafikdrivrutiner
* Kontrollera Windows Händelsevisare för felinformation
* Kontakta supporten med felloggarna

**Problem med licensaktivering:**

* Se till att internetanslutningen är aktiv
* Kontrollera inloggningsuppgifterna på [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontrollera att brandväggen inte blockerar Chloros
* Se [Chloros+ Login](chloros+-login.md) för detaljerade instruktioner

### Få support

Behöver du hjälp med installation eller konfiguration?

* 📧 **E-post**: info@mapir.camera
* 🌐 **Webbplats**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentation**: [Komma igång](./)
* ❓ **FAQ**: [Vanliga frågor](faq.md)***

## Ändringslogg

<details>

<summary>Version 1.0.4</summary>

#### **Utgivningsdatum**: 5 januari 2026**Nya funktioner*** **Växla mellan bild/metadata**: Växlingsknapp har lagts till i filbläddraren för att visa metadata för vald bild i en tabell istället för i bildrutnätet
* **Zoomreglage för bildrutnät**: Nytt reglage i användargränssnittet för att justera miniatyrstorlek (stöder även CTRL + mushjul)
* **Exportknappar för bildrutnät**: Knappar i den övre raden för att växla miniatyrbilder från JPG till bearbetade exporter (mål, reflektans, index, LUT)
* **Kartflik**: Ny interaktiv 2D-karta som visar bildens GPS-positionsmarkörer.
  * Stöder Google Maps och ESRI-kartplattor (väljer automatiskt den bästa plattservicen baserat på tillgänglig zoomnivå).
  * Förhandsgranska miniatyrbilder genom att hålla muspekaren över kartmarkörerna.

**Buggfixar*** Förbättrat stöd för installation av Chloros på datorer med andra språk än engelska.

</details>

<details>

<summary>Version 1.0.3</summary>

#### **Utgivningsdatum**: 20 december 2025**Nya funktioner*** Första lansering

**Förbättringar*** Första lansering

**Buggfixar*** Första lansering

**Kända problem*** Första lansering

</details>***

## Licensavtal**Proprietär programvara** – Copyright (c) 2025 MAPIR Inc.

Obehörig användning, distribution eller modifiering är förbjuden.

**Gratisversion**: Tillgänglig för personlig och kommersiell användning med funktionsbegränsningar.**Chloros+**: Abonnemangsbaserad licens för avancerade funktioner och kommersiell användning.
