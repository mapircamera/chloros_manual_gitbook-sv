# Dynamisk anpassning av beräkningskapacitet

Chloros 1.3.0 använder hårdvaruavkänning och automatiskt val av bearbetningsstrategi. Bearbetningsmotorn anpassar sig till din hårdvara – från en Jetson Orin Nano till en arbetsstation med flera GPU:er – utan någon manuell konfiguration.

***

## Så här fungerar det

När Chloros startar profilerar det ditt system:

1. **Upptäcker operativsystemet** – Windows eller Linux
2. **Identifierar CPU-kärnor och totalt RAM-minne**

3.**Upptäcker förekomst av GPU** – NVIDIA CUDA-kapacitet, VRAM, modell
4. **Identifierar Jetson-modellen** (om tillämpligt) — via `/proc/device-tree/model`
5. **Kontrollerar temperatursensorer** (Jetson) — för temperaturanpassad bearbetning
6. **Väljer beräkningsstrategi** — baserat på all upptäckt hårdvara
7. **Konfigurerar antal arbetare, pipelinetyp och minnesallokering** automatiskt

Den upptäckta profilen cachelagras för sessionen i minnet och på disken, så att senare körningar startar snabbare:

| Plattform | Cachelagrad profil |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (prioriterar `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Ta bort den filen för att tvinga fram en ny detektering — användbart efter att du har lagt till en GPU eller mer RAM-minne. Chloros utför också automatiskt en ny detektering när cachen har skrivits av en inkompatibel äldre version.

***

## Beräkningsstrategier

Chloros väljer en av tre beräkningsstrategier baserat på din hårdvara:

| Strategi | Väljs när | Arbetare | Exekverare | Pipeline |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA-GPU som rapporterar**12 GB+ VRAM**(på Jetson unified memory, kräver även minst 12 GB delat RAM-minne totalt) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, minst 2 —**begränsat till 2 på Jetson** | Arbetsprocesser (separata CUDA-kontexter) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA-GPU med**2–12 GB VRAM**| 3 (I/O-överlappning; GPU-åtkomst serialiserad av en semafor).**1 (sekventiellt) på Jetson-enheter med mindre än 12 GB RAM** | Arbetsprocesser (separata CUDA-kontexter); sekventiellt inom samma process på Jetson-enheter med lågt RAM-minne | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Ingen CUDA-GPU eller mindre än 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Exempel på hur formeln för antalet arbetare (`GPU_PARALLEL`) fungerar: 12 GB VRAM → 2 arbetare, 16 GB → 3 arbetare, 20 GB+ → 4 arbetare. Varje arbetare tilldelas 3,5 GB, vilket motsvarar det uppmätta behovet för en texturmedveten uppgift, inom 90 % av det återstående VRAM-utrymmet efter en reserv på 0,5 GB.

{% hint style="info" %}
**GPU-strategierna kräver en PyTorch-version som kan använda CUDA.** En maskin vars PyTorch inte har någon användbar CUDA-enhet tilldelas `CPU_PARALLEL`, oavsett vilken GPU den har. Jetson-paketet 1.3.0 använder JetPack 6:s CUDA-version av PyTorch, så en Jetson uppfyller kraven så snart JetPacks CUDA- och cuDNN-bibliotek har installerats. En Jetson med mindre än 12 GB RAM bearbetar alltid en bild i taget, i processen (loggas som `GPU_SINGLE`).
{% endhint %}

GPU-strategier kör en pool av arbetsprocesser, var och en en separat OS-process med sitt eget CUDA-kontext, vilket Chloros startar genom att återigen köra sitt eget installerade backend-program (inte via Pythons `multiprocessing`, vars start av arbetsprocesser inte fungerar inuti den paketerade applikationen). CPU-strategin använder en trådpool (Pythons `ThreadPoolExecutor`). Chloros använder inget distribuerat ramverk från tredje part (såsom Ray).

### Pipelinityper

* **`fused_gpu`** — Fullständig GPU-bearbetningsväg. Debayer-, korrigerings- och indexeringsoperationer körs på GPU:n i ett enda sammansmält pass. Högsta genomströmning, kräver mest VRAM.
* **`tiled_gpu`** — Minneseffektiv GPU-väg. Bearbetar bilder i rutor för att passa inom det begränsade GPU-minnet. Lägre genomströmning men fungerar på enheter med begränsat minne.
* **`cpu_fallback`** — Bearbetning enbart på CPU med hjälp av multitrådad parallellitet. Används när ingen NVIDIA-GPU är tillgänglig, och som sista utväg när båda GPU-vägarna misslyckas.

Kedjan för reservlösningar vid körning är alltid `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Manuell överskrivning av strategi

Ställ in miljövariabeln `CHLOROS_STRATEGY` för att tvinga fram en specifik strategi — en expertlösning för de fall då den automatiska detekteringen väljer något som inte passar din situation (till exempel för att hålla GPU:n ledig för annat arbete):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Variabeln matchas utan hänsyn till versaler och gemener; allt som inte är ett av de tre namnen ignoreras och den automatiska detekteringen fortsätter som vanligt. Vid en överskrivning väljer Chloros fortfarande antalet arbetare åt dig:

| Överskrivning | Antal arbetare som används |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

En överskrivning av `GPU_SINGLE` eller `GPU_PARALLEL` på en maskin utan användbar CUDA-enhet avvisas med en rad med koden `[STRATEGY]` i loggen, och körningen använder `CPU_PARALLEL`.

Det är bättre att ställa in detta per kommando än permanent, så att normala körningar fortsätter att anpassa sig automatiskt.

***

## Plattformsspecifikt beteende

| Plattform | Strategi | Arbetare | Pipeline | Anmärkningar |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekventiell) | Minneseffektivt läge, en bild i taget |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekventiell) | Delat RAM-minne under 12 GB tvingar fram sekventiell bearbetning |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (parallell) | Rekommenderad edge-enhet – begränsad till 2 arbetare på Jetson |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (parallell) | Maximal prestanda i kanten (även här begränsat till 2 arbetare på Jetson) |
| **Stationär dator med 8 GB GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 arbetare överlappar I/O medan en semafor serialiserar GPU-åtkomsten |
| **Stationär dator med 12 GB+ GPU** | `GPU_PARALLEL` | 2–4 | `fused_gpu` (samtidigt) | Optimal stationär prestanda: 12 GB → 2 arbetare, 16 GB → 3, 20 GB+ → 4 |
| **System med endast CPU** | `CPU_PARALLEL` | fysiska kärnor − 1 (minst 2) | `cpu_fallback` | Ingen GPU krävs, använder en trådpool |

{% hint style="info" %}
**Jetsons enhetliga minne**: Jetson-enheter delar GPU- och CPU-minne. En Jetson Orin NX med 16 GB rapporterar ~15,3 GB VRAM, men det är samma fysiska RAM-minne som används av operativsystemet och CPU-processerna. Det är därför Jetson-enheter med 16 GB eller mer uppfyller kraven för `GPU_PARALLEL` på samma sätt som en stationär GPU med 12 GB eller mer, men ändå är begränsade till två arbetare – GPU:n, arbetarprocesserna och deras CUDA-kontexter per arbetare drar alla från samma delade pool.
{% endhint %}

### GPU-budget baserad på VRAM (diskreta GPU:er)

På x86_64-värdar med en diskret NVIDIA-GPU avgör det upptäckta VRAM-minnet också hur stor del av kortets kapacitet som får användas och hur stora batcher som kan bli:

| Upptäckt VRAM | Tak för GPU-budget | Multiplikator för batchstorlek |
| --- | --- | --- |
| **8 GB+** | 90 % | ×2,0 |
| **6–8 GB** | 85 % | ×1,75 |
| **3,5–6 GB** | 80 % | ×1,5 |
| **2–3,5 GB** | 75 % | ×1,25 |
| **Under 2 GB** | 70 % | ×1,0 |

Diskreta GPU:er reserverar endast 0,5 GB för systemet, eftersom de inte delar systemets RAM-minne. Jetson-profiler reserverar betydligt mer och har lägre tak — se [NVIDIA Jetson-guiden](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dynamisk tilldelning av GPU-minne

Chloros använder en [bearbetningspipeline med 4 trådar](processing-pipeline.md):

* **Tråd 1** (Detektering) — Bildinläsning, EXIF-analys, måldetektering
* **Tråd 2** (Kalibrering) — Beräkning av reflektanskalibrering
* **Tråd 3** (Bearbetning) — GPU-debayer, vignettkorrigering, indexberäkning
* **Tråd 4** (Export) — Filskrivning, inbäddning av metadata

Trådarna 1, 2 och 4 är lätta GPU-användare; Tråd 3 är den som kräver mest resurser. När tidigare trådar i pipelinen avslutas **omfördelas deras GPU-resurser till de återstående aktiva trådarna**, vilket innebär att tråd 3 får allt mer minne ju längre körningen fortskrider.

### Allokeringssteg

| Steg | Aktiva trådar | GPU-minnesfördelning |
| --- | --- | --- |
| **Tidigt** | 1, 2, 3, 4 | Fördelas över alla trådar, varav det mesta till tråd 3 |
| **Tidigt-mitten** | 2, 3, 4 | Tråd 1:s andel omfördelas |
| **Mitten–slutet** | 3, 4 | Trådarnas 1+2 andelar går till 3+4 |
| **Slutet** | 3 eller 4 | Den sista aktiva tråden får sin maximala tilldelning |

Två regler styr siffrorna:

* En tråd som är den **enda** aktiva tilldelas sin profils maximala tilldelning.
* När mer än en *tung* GPU-uppgift är aktiv delas varje tung uppgifts bastilldelning mellan dem (aldrig under det konfigurerade minimumet).

Det värde som faktiskt används vid körning är det **lägsta** av plattformsprofilens tilldelning och den aktuella rekommendationen från GPU-minnesmonitorn, så ett upptaget kort har alltid företräde framför en optimistisk profil.***

## Texturmedveten bearbetning

Den texturmedvetna debayern (**endast Chloros+** — `--debayer texture-aware`) kör en AI/ML-brusreduceringsmodell som behöver ungefär 1,75 GB VRAM i FP16 per kopia, så den använder betydligt mer GPU-minne än standardmetoden:

* System med **mindre än 7 GB VRAM**bearbetar texturmedveten bearbetning i en**synkron slinga, en bild i taget** – flera modellkopior får inte plats, och en arbetspool skulle bara öka konkurrensen
* System med **7 GB eller mer VRAM** kan bearbeta Texture Aware samtidigt, dock med ett lägre antal arbetare jämfört med standardmetoden
* På **Jetson** är Texture Aware alltid knuten till en enda arbetare
* Om det **inte finns någon användbar CUDA-GPU** använder körningen Standard-debayern, och backend-loggen visar en rad med koden `[DEBAYER] … needs a CUDA GPU …` som förklarar varför
* På Jetson-enheter med låg strömförbrukning (Nano, Orin Nano) tillämpar CLI:n en GPU-frekvensbegränsning när du begär Texture Aware — se [NVIDIA Jetson-guiden](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Värmehantering (Jetson)

Jetson-enheter har termiska begränsningar, särskilt vid installationer i slutna utrymmen eller i flygplan. Chloros övervakar Jetsons inbyggda temperatursensorer och skalar batchstorlekarna automatiskt:

| Temperatur | Reaktion |
| --- | --- |
| **< 70 °C** | Normal drift – full hastighet |
| **70 °C** (Varning) | Batchstorleken minskas gradvis (100 % → 50 % mellan 70 °C och 80 °C) |
| **80 °C** (Kritiskt) | Aggressiv avreglering (50 % → 0 % mellan 80 °C och 90 °C) |
| **90 °C** (Avstängning) | Stoppa GPU-bearbetningen helt |

På stationära system med tillräcklig kylning utlöses termisk strypning sällan.

***

## Hantering av minnesbelastning

Chloros övervakar GPU-minnet kontinuerligt under bearbetningen och reagerar på tre nivåer.

**Batchstorlek.** En batch börjar med 8 bilder gånger plattformsmultiplikatorn från tabellerna ovan. Chloros kontrollerar sedan ledigt VRAM, reserverar 20 % av det för PyTorchs egen overhead och antar ungefär 100 MB GPU-minne per 12 MP-bild — batchen är den som är minst av de två, antingen den minnesbaserade gränsen eller plattformsbasen. Den sjunker aldrig under 1.

**Förebyggande minskning.**Vid**över 85 % VRAM-utnyttjande** minskas batchstorlekarna innan något går fel.**Begränsning av tilldelningen per tråd.** När den aktuella utnyttjandegraden stiger skalas varje tråds GPU-budget ned: ×0,75 vid över 80 % utnyttjande, ×0,5 vid över 90 %. Övervakningsintervallen är 70 % (konservativt), 85 % (normal driftsgräns) och 95 % (risk för OOM).**OOM-backoff och återställning.** Om en minnesbrist ändå inträffar:

* halveras batchstorleken, och halveras ytterligare vid varje efterföljande OOM-händelse — varje efterföljande lyckad batch minskar denna straffnivå med ett steg
* aktiva trådstilldelningar minskas till 70 % av sitt aktuella värde och tilldelningsmodulen övergår till sin konservativa strategi, för att sedan återgå till normal nivå efter en serie lyckade tilldelningar
* vid kraftig belastning övergår pipelinen från `fused_gpu` till `tiled_gpu`, och till `cpu_fallback` som en sista utväg

**Värd-RAM (Jetson).** Innan bearbetningen påbörjas uppskattar CLI-verktyget värdminnets toppvärde utifrån antalet bilder och debayer-läget och varnar om RAM-minnet plus filbaserad swap sannolikt är otillräckligt, samtidigt som det visar de exakta kommandona för att lägga till swap — se [NVIDIA Jetson-guiden](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Övervakning av beräkningsanpassning

### Systemdiagnostik

`chloros-cli selftest` är det snabbaste sättet att bekräfta vad beräkningslagret ser:

```bash
chloros-cli selftest
```

Dess 7 kontroller omfattar version, porttillgänglighet, uppstart av backend, `/api/test`, systeminformation, förekomst av brusreduceringsmodell samt beredskap för CUDA och brusreducering. Kontroll 5 visar hårdvaruraden direkt:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Kontroll 7 visar `CUDA: <bool>, Denoiser: <bool>` — båda måste vara sanna för att Texture Aware överhuvudtaget ska kunna användas.

### Backend-loggar

Strategin och antalet arbetare väljs inuti backend vid starten av varje körning — det finns ingen CLI-banner som visar dessa. När något beter sig oväntat (en GPU-väg som faller tillbaka, ett OOM, en brusreducerare som inte går att ladda) är det i backend-loggen för den sessionen som det syns:

| Plattform | Loggplats |
| --- | --- |
| **Linux/Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (en fil per start) |
| **Linux, backend startad via CLI** | även `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Förlopp i realtid

Under en körning visar CLI:n realtidsförlopp per tråd (Detektering, Analys, Bearbetning, Export) som strömmas via Server-Sent Events — det praktiska sättet att avgöra om tråd 3 är flaskhalsen. Se [Bearbetningspipeline](processing-pipeline.md).

***

## Nästa steg

* [Bearbetningspipeline](processing-pipeline.md) — Förstå arkitekturen för den 4-trådiga pipelinen
* [NVIDIA Jetson-guide](../linux/nvidia-jetson-guide.md) — Jetson-specifik driftsättning och optimering
* [CLI: Kommandorad](../CLI.md) — Guiden för CLI

* [CLI-referens](../reference/cli-reference.md) — Utförlig kommandolista för version 1.3.0
