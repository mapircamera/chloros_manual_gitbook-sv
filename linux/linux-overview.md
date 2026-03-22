# Linux – Översikt

Chloros 1.1.0 erbjuder inbyggt stöd för **CLI**och**Python SDK**, vilket möjliggör headless multispektral bildbehandling på Linux-arbetsstationer, servrar och NVIDIA Jetson-edge-enheter.

{% hint style="info" %}
**Inget GUI på Linux.** Chloros Desktop GUI är endast tillgängligt på Windows. Linux-användare interagerar med Chloros via [CLI](../CLI.md) och [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Plattformssupportmatris

| Funktion | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Skrivbordsgränssnitt** | Ja | Ej tillämpligt | Nej | Nej |
| **CLI** | Ja | Ja | Ja | Ja |
| **Python SDK** | Ja | Ja | Ja | Ja |
| **GPU-acceleration (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Texturmedveten debayer** | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamisk beräkningsanpassning** | Ja | Ja | Ja | Ja |***

## Arkitekturer som stöds

| Arkitektur | Beskrivning | Installationsmetod |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardprocessorer för stationära datorer/servrar (Intel, AMD) | `.deb`-paket |
| **arm64 (aarch64)** | ARM-baserade processorer, främst NVIDIA Jetson | `.deb`-paket (JetPack 6) |

## Linux-distributioner som stöds

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson-plattformar)***

## Vad Linux-användare får

* **Chloros CLI** — Fullständigt kommandoradsgränssnitt för batchbearbetning, automatisering och skriptning
* **Chloros Python SDK** — Programmeringsgränssnitt (Python) för integration i forskningspipelines och anpassade verktyg
* **GPU-acceleration** — CUDA-accelererad bearbetning på NVIDIA-GPU:er (stationära och Jetson)
* **Dynamisk beräkningsanpassning** — Automatisk hårdvarudetektering och optimering av bearbetningsstrategi
* **Alla bearbetningsfunktioner** — Samma multispektrala bearbetningspipeline som Windows (kalibrering, vignettkorrigering, vegetationsindex, alla exportformat)
* **Chloros+-funktioner** — Multitrådad bearbetning, texturmedveten debayer, anpassade index (med Chloros+-licens)

## Vad Linux-användare inte får

* **GUI för skrivbordet** — Inget grafiskt gränssnitt; all interaktion sker via CLI eller Python SDK
* **Bildvisare** — Ingen interaktiv bildvisare, rutnätsvy eller kartmarkörer
* **Visuell projektledning** — Projekt hanteras via CLI-kommandon och SDK-anrop***

## Komma igång med Linux

1. **Installera Chloros** — Se [Linux Installation](linux-installation.md) för installation av `.deb`-paketet
2. **Installera Python SDK** (valfritt) — `pip install chloros-sdk`
3. **Aktivera din licens** — `chloros-cli login your@email.com 'password'`
4. **Bearbeta din första dataset** — `chloros-cli process ~/datasets/flight001`

För NVIDIA Jetson-användare, se den särskilda [NVIDIA Jetson-guiden](nvidia-jetson-guide.md) för plattformsspecifik konfiguration och optimering.

***

## Nästa steg

* [Linux Installation](linux-installation.md) — Detaljerade installationsinstruktioner för amd64 och arm64
* [NVIDIA Jetson-guide](nvidia-jetson-guide.md) — Jetson-specifik konfiguration, värmehantering och fältinstallation
* [CLI : Kommandorad](../CLI.md) — Fullständig CLI-referens
* [API : Python SDK](../api-python-sdk.md) — Fullständig SDK-referens
* [Dynamisk beräkningsanpassning](../processing-architecture/dynamic-compute-adaptation.md) — Hur Chloros anpassar sig till din hårdvara
