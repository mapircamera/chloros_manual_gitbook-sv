---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Vanliga frågor

<details>

<summary>Kan jag bearbeta bilder från kameror som inte är av märket MAPIR med Chloros?</summary>

Nej, Chloros stöder endast bearbetning av bilder från MAPIR-kameror. Se listan över [kameramodeller som stöds](supported-cameras.md) för mer information. Vi erbjuder bearbetning av bilder från andra kameror på MAPIR Cloud, se hela listan [här](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Kan jag kalibrera mina bilder för reflektans utan ett kalibreringsmål?</summary>

Nej. Utan en bild av kalibreringsmålet som tagits i närheten när bilderna utan målet tas kommer du inte att kunna relatera bildens pixelvärden till en känd reflektansprocent. Om du inte heller inkluderar loggen från en MAPIR-ljussensor kommer det omgivande ljusspektrumet inte att mätas, och reflektansresultaten kommer inte att vara korrekta.

</details>

<details>

<summary>Kan jag redigera mina bilder innan de bearbetas i Chloros?</summary>

Nej. Chloros utgår från att indata inte har modifierats. Ändra inte filnamnen.

</details>

<details>

<summary>Kan jag ställa in mina MAPIR Survey3-kameror på automatisk exponering och bearbeta bilderna i Chloros?</summary>

Nej. Bilddatauppsättningar i Survey3 måste ha en fast/låst exponering, så ingen automatisk slutartid eller automatisk ISO. Alla bilder från samma kameramodell måste ha identisk slutartid och ISO (exponering).

</details>

<details>

<summary>Kan Chloros bearbeta eller analysera ortomosaikbilder?</summary>

Nej. Endast enskilda MAPIR-kamerabilder stöds, inte sammanfogade bilder som en ortomosaikkarta.

</details>

<details>

<summary>Hur kan jag påskynda måldetekteringssteget i Chloros?</summary>

I filbläddrarens tabell kan du förvälja målbilderna i den högra kolumnen, vilket talar om för Chloros att endast söka efter kalibreringsmål i dessa bilder, vilket avsevärt påskyndar bearbetningen.

</details>

<details>

<summary>Om jag ska ladda upp mina bilder till <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> ska jag då bearbeta dem i Chloros innan jag laddar upp dem?</summary>

Om du planerar att ladda upp till vår onlinebehandlingsplattform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) ska du inte redigera bilderna innan du laddar upp dem. Cloud utför samma bearbetning och mer därtill.

</details>

<details>

<summary>Kommer MAPIR någonsin att stödja X-funktionen? Jag önskar verkligen att MAPIR erbjöd X.</summary>

Vi är alltid intresserade av att få feedback på våra produkter. Om du upptäcker ett problem med våra produkter eller har ett förslag på hur vi kan förbättra dem, vänligen [KONTAKTA OSS](https://www.mapir.camera/community/contact) för att dela med dig av dina tankar. Större delen av vår forskning och utveckling styrs av att lyssna på våra kunders största behov.

</details>

<details>

<summary>Finns Chloros tillgängligt för Linux?</summary>

Ja! Chloros 1.1.0 stöder Linux amd64 (x86_64) och arm64 (NVIDIA Jetson JetPack 6) via `.deb`-paket. CLI och Python SDK stöds fullt ut på Linux. Det finns inget grafiskt gränssnitt för Linux — all interaktion sker via [CLI](CLI.md) eller [Python SDK](api-python-sdk.md). Se [Linux Översikt](linux/linux-overview.md) för mer information.

</details>

<details>

<summary>Kan jag köra Chloros på NVIDIA Jetson?</summary>

Ja! Chloros 1.1.0 stöder NVIDIA Jetson-plattformar, inklusive Jetson Nano, Orin Nano, Orin NX och AGX Orin som kör JetPack 6. Chloros identifierar automatiskt din Jetson-modell och optimerar dess bearbetningsstrategi. Se [NVIDIA Jetson-guiden](linux/nvidia-jetson-guide.md) för instruktioner om installation och driftsättning.

</details>

<details>

<summary>Optimerar Chloros automatiskt för min hårdvara?</summary>

Ja! Chloros 1.1.0 inkluderar [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) som automatiskt upptäcker din CPU, GPU, RAM och (på Jetson) temperatursensorer. Därefter väljer den den optimala bearbetningsstrategin – från `GPU_PARALLEL` på system med stort minne till `GPU_SINGLE` på enheter med begränsningar och till `CPU_PARALLEL` på system utan NVIDIA-GPU. Ingen manuell konfiguration behövs.

</details>

<details>

<summary>Vad är 4-trådsbehandlingspipeline?</summary>

Chloros 1.1.0 använder en 4-trådig pipelined-arkitektur för Chloros+-användare: Tråd 1 (Detektering) laddar bilder och detekterar kalibreringsmål, Tråd 2 (Kalibrering) beräknar reflektanskalibrering, Tråd 3 (Bearbetning) utför GPU-accelererad debayering och indexberäkning, och Tråd 4 (Export) skriver utdatafiler. Flera bilder kan finnas i olika trådar samtidigt för maximal genomströmning. Se [Bearbetningspipeline](processing-architecture/processing-pipeline.md) för mer information.

</details>

<details>

<summary>Hur kör jag diagnostik på min Chloros-installation?</summary>

Använd kommandot `selftest` för att köra 7 systemdiagnostiker, inklusive versionskontroll, porttillgänglighet, backend-start, API-anslutning, systeminformation, brusreduceringsmodeller och CUDA-tillgänglighet:

```bash
chloros-cli selftest
```

Detta är särskilt användbart på Linux/Jetson-system för att verifiera GPU- och CUDA-konfigurationen.

</details>
