---
metaLinks: {}
---

# Komma igång

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros är ett program från [MAPIR](https://www.mapir.camera) för att bearbeta bilder och annan sensordata.

***{% hint style="success" %}**Nyheter i Chloros 1.1.0**: Inbyggt stöd för Linux (amd64 och arm64), NVIDIA Jetson edge computing, Dynamic Compute Adaptation, 4-tråds bearbetningspipeline, nya CLI-kommandon och alternativ. Se [Nedladdning](download.md) för den fullständiga ändringsloggen.
{% endhint %}

Chloros finns i 3 applikationslägen:

## Chloros: Desktop-GUI-applikation

Fristående separat fönster med alla funktioner. _Endast Windows._

## [Chloros CLI: Kommandoradsgränssnitt](CLI.md)

Batchbearbetning via kommandoraden. Perfekt för automatisering, skriptning och headless-drift. Tillgängligt på **Windows, Linux amd64 och Linux arm64 (NVIDIA Jetson)**. _CLI kräver en Chloros+-licens för åtkomst._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmeringsgränssnitt för automatisering och anpassade arbetsflöden. Perfekt för forskningspipelines, integration med befintliga applikationer och utveckling av anpassade verktyg. Tillgängligt på **alla plattformar** via `pip install chloros-sdk`. _API kräver en Chloros+-licens för åtkomst._***

## Plattformar som stöds

| Plattform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Ja | Ja | Ja |
| **Linux amd64 (x86_64)** | Nej | Ja | Ja |
| **Linux arm64 (NVIDIA Jetson)** | Nej | Ja | Ja |

För installationsanvisningar för Linux, se avsnittet [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

Även om Chloros är gratis att använda för de flesta uppgifter, kanske du upptäcker att du vill ha mer. Det är då en betald licens för Chloros+ kan vara till nytta för dig. Med en Chloros+-licens kan du låsa upp nya funktioner såsom:

* **Multitrådad bearbetning**: påskynda bildbearbetningen avsevärt för större projekt genom att samtidigt bearbeta bilder genom pipelinen.
* **GPU (CUDA)-acceleration**: dra nytta av dagens högre GPU-minnesalternativ för att ytterligare påskynda bildbehandlingspipeline. Vi rekommenderar 4 GB eller mer VRAM för bästa resultat.
* **Chloros+**[**CLI**](CLI.md)**Åtkomst**: kör Chloros+ från kommandoraden för att automatisera och integrera i din egen programvara.
* **Chloros+**[**API**](api-python-sdk.md)**Åtkomst:** kör Chloros+ från Python för programstyrning, vilket möjliggör sömlös integration med dina forskningspipelines, arbetsflöden för dataanalys och anpassade applikationer.
* **Användning på flera enheter**: varje Chloros+-licens tillåter registrering av 2+ enheter. Använd ditt MAPIR Cloud-konto för att hantera registrerade enheter. Lägg till stöd för fler enheter genom att uppgradera din Chloros+-licens.
* **Avancerad texturmedveten debayer-metod:** en högkvalitativ kantmedveten debayer kombinerad med en AI/ML-brusreduceringsmodell som tar bort nästan allt debayeringsbrus. 
* **Anpassade multispektrala indexformler:** ange anpassade multispektrala index i Chloros-rasterkalkylatorerna, både för bearbetning och bildvisningssandlådan.
* **Linux &amp; Edge Computing:** kör Chloros på Linux x86\_64- och ARM64-plattformar, inklusive NVIDIA Jetson, för fält- och kantbearbetning. Se [Linux Översikt](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Priser och registrering</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
