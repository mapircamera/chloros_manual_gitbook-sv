# Dynamisk anpassning av beräkningskapacitet

Chloros 1.1.0 introducerar intelligent hårdvaruidentifiering och automatiskt val av bearbetningsstrategi. Bearbetningsmotorn anpassar sig till din hårdvara – från en Jetson Nano till en arbetsstation med flera GPU:er – utan någon manuell konfiguration.

***

## Så fungerar det

När Chloros startar profilerar det automatiskt ditt system:

1. **Upptäcker operativsystemet** – Windows eller Linux
2. **Identifierar CPU-kärnor och totalt RAM-minne**

3.**Upptäcker förekomst av GPU** — NVIDIA CUDA-kapacitet, VRAM, modell
4. **Identifierar Jetson-modell** (om tillämpligt) — via `/proc/device-tree/model`
5. **Kontrollerar temperatursensorer** (Jetson) — för temperaturmedveten bearbetning
6. **Väljer den optimala beräkningsstrategin** — baserat på all upptäckt hårdvara
7. **Konfigurerar antal arbetare, pipelinetyp och minnesallokering** automatiskt

Resultatet cachelagras så att efterföljande körningar startar snabbare. Om hårdvaran ändras (t.ex. om en GPU läggs till) skapar Chloros en ny profil vid nästa start.

***

## Beräkningsstrategier

Chloros väljer en av tre beräkningsstrategier baserat på din hårdvara:

| Strategi | GPU krävs | Arbetare | Pipeline | Bäst för |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Ja (12 GB+ VRAM eller 16 GB+ delat) | 3–4 | `fused_gpu` | Stationära GPU:er med 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Ja (&lt; 12 GB VRAM) | 1–3 | `tiled_gpu` | GPU:er på instegsnivå, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Nej | kärnor – 1 | `cpu_fallback` | System utan NVIDIA-GPU |

### Pipelintyper

* **`fused_gpu`** — Fullständig GPU-bearbetningsväg. Alla debayer-, korrigerings- och indexeringsoperationer körs på GPU:n i ett enda sammanslaget pass. Högsta genomströmning men kräver mer VRAM.
* **`tiled_gpu`** — Minneseffektiv GPU-väg. Bearbetar bilder i rutor för att passa inom det begränsade GPU-minnet. Lägre genomströmning men fungerar på enheter med begränsat minne.
* **`cpu_fallback`** — Endast CPU-bearbetning med multitrådad parallellitet. Används när ingen NVIDIA-GPU är tillgänglig.***

## Plattformsspecifikt beteende

| Plattform | Strategi | Arbetare | Pipeline | Anmärkningar |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (serialiserad) | Minneseffektivt läge, bearbetar en bild i taget |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (parallell) | Rekommenderad edge-enhet – verklig parallell GPU-bearbetning |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (parallell) | Maximal kantprestanda |
| **Stationär dator med 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Bra prestanda för stationära datorer med minneseffektiva rutor |
| **Stationär dator med 12 GB+ GPU** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimal stationär prestanda |
| **System med endast CPU** | `CPU_PARALLEL` | kärnor – 1 | `cpu_fallback` | Ingen GPU krävs, använder ThreadPool |

{% hint style="info" %}
**Jetson unified memory**: Jetson-enheter delar GPU- och CPU-minne. En Jetson Orin NX 16 GB rapporterar ~15,3 GB VRAM, men detta är samma fysiska RAM som används av operativsystemet och CPU-processerna. Chloros tar hänsyn till detta vid inställning av tröskelvärden för minnesallokering.
{% endhint %}

***

## Dynamisk GPU-minnesallokering

Chloros använder en [4-trådig bearbetningspipeline](processing-pipeline.md):

* **Tråd 1** (Detektering) — Bildinläsning, EXIF-parsning, måldetektering
* **Tråd 2** (Kalibrering) — Beräkning av reflektanskalibrering
* **Tråd 3** (Bearbetning) — GPU-debayer, vignettkorrigering, indexberäkning
* **Tråd 4** (Export) — Filskrivning, inbäddning av metadata

När tidigare trådar i pipelinen har slutfört sitt arbete (t.ex. alla bilder har detekterats) frigörs deras GPU-minnesallokering och **omfördelas till de återstående aktiva trådarna**. Detta innebär att tråd 3 (det GPU-intensiva steget) får allt mer minne ju längre pipelinen fortskrider, vilket förbättrar genomströmningen för det mest beräkningsintensiva arbetet.

### Allokeringssteg

| Steg | Aktiva trådar | GPU-minnesfördelning |
| --- | --- | --- |
| **Tidigt** | 1, 2, 3, 4 | Fördelat över alla trådar |
| **Tidigt-mellan** | 2, 3, 4 | Tråd 1:s minne omfördelas |
| **Mellan-sent** | 3, 4 | Trådarna 1+2:s minne går till 3+4 |
| **Sent** | 3 eller 4 | Maximalt minne för återstående tråd |***

## Texture Aware-bearbetning

Texture Aware-debayer-metoden (endast Chloros+) använder betydligt mer GPU-minne än standardmetoden på grund av AI/ML-brusreduceringsmodellen:

* System med **&lt; 7 GB VRAM** tvingas in i en synkron bearbetningsslinga för Texture Aware-läget (en bild i taget)
* System med **7 GB+ VRAM** kan bearbeta texturmedveten bearbetning samtidigt, men med ett reducerat antal arbetare jämfört med standardläget***

## Värmehantering (Jetson)

Jetson-enheter har termiska begränsningar, särskilt i slutna eller flygburna installationer. Chloros övervakar GPU- och CPU-temperaturer och justerar bearbetningen automatiskt:

| Temperatur | Reaktion |
| --- | --- |
| **&lt; 70 °C** | Normal drift — full hastighet |
| **70 °C** (Varning) | Minska batchstorlek |
| **80 °C** (Kritiskt) | Aggressiv strypning — lägre samtidighet och antal arbetare |
| **90 °C** (Avstängning) | Stoppa GPU-bearbetningen helt |

Temperaturövervakning använder `tegrastats` på Jetson-plattformar. På stationära system med tillräcklig kylning utlöses termisk strypning sällan.

***

## Hantering av minnesbelastning

Chloros övervakar systemminnesbelastningen under bearbetningen:

* **Minnesgräns**: 85 % utnyttjande utlöser konservativt beteende
* **OOM-reduktion**: Om en minnesbrist inträffar minskas allokeringen med 25 % (0,75x multiplikator)
* **Pipeline-fallback**: Vid kraftig minnesbelastning faller pipelinen automatiskt tillbaka från `fused_gpu` till `tiled_gpu`
* **Rekommendationer för swap**: På Jetson varnar Chloros dig om swaputrymmet är otillräckligt för din datasetstorlek***

## Övervakning av beräkningsanpassning

### CLI-statusutdata

När bearbetningen startar visar CLI den upptäckta hårdvaruprofilen:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Systemdiagnostik

Kör `chloros-cli selftest` för att se en fullständig hårdvaruprofil och verifiera beräkningskapaciteten:

```bash
chloros-cli selftest
```

Detta kontrollerar CUDA-tillgänglighet, GPU-minne, brusreduceringsmodeller och backend-anslutning.

***

## Nästa steg

* [Bearbetningspipeline](processing-pipeline.md) — Förstå arkitekturen för 4-tråds-pipeline
* [NVIDIA Jetson-guide](../linux/nvidia-jetson-guide.md) — Jetson-specifik distribution och optimering
* [CLI : Kommandorad](../CLI.md) — Fullständig CLI-referens
