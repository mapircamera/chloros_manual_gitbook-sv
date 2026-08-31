---
metaLinks: {}
---

# Kom igång

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros är ett program från [MAPIR](https://www.mapir.camera) för att bearbeta multispektrala bilder, fjärrstyra MAPIR-hårdvara i realtid och registrera sensordata. Chloros 1.2.0 stöder hela MAPIR-produktfamiljen:

* **Survey3-kameror** — bearbetar RAW+JPG-bilder till kalibrerade reflektans- och vegetationsindexkartor. Se [Kompatibla kameror](supported-cameras.md).
* **LATTICE-kameror** — anslut GigE-modulerna för multispektralkameror i realtid, enskilt eller som synkroniserade system med flera kameror: förhandsgranska, ta bilder och bearbeta till kalibrerade strålnings- och reflektansprodukter. Se [avsnittet om LATTICE](lattice/README.md).
* **DAQ-ljussensorer** — DAQ-U (USB), DAQ-M (Bluetooth) och DAQ-E (Ethernet) spektralsensorer: kalibrerade spektra i realtid, `.daq`-inspelningar och nedåtriktad belysning för reflektansbearbetning. Se [avsnittet om DAQ](daq/README.md).

{% hint style="success" %}
**Nyheter i Chloros 1.2.0**: kontroll av LATTICE-kameror och -matriser i realtid, integration av DAQ-ljussensorer, inspelningslägen och inspelningsenheter, en komplett radiometrisk bearbetningspipeline för LATTICE, projektautomatisering från CLI/SDK och mycket mer. Se listan över nyheter nedan och [Ladda ner](download.md) för ändringsloggen.
{% endhint %}

{% hint style="info" %}
**Använder du Chloros med en AI-assistent?** Den här handboken är utformad för just det. Låt din assistent söka efter:

* `https://mapir.gitbook.io/chloros/llms.txt` — ett maskinläsbart index över varje sida.
* Vilken sida som helst som rå Markdown — lägg till `.md` till dess URL (t.ex. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-referensen](reference/cli-reference.md) och [SDK-referensen](reference/sdk-reference.md) — fullständiga referenssidor med exakta värden, skrivna för användning av stora språkmodeller (LLM).

Exempel på prompt: *&quot;Läs https://mapir.gitbook.io/chloros/reference/cli-reference.md, och skriv sedan ett skript som loggar in och bearbetar mappen ~/flights/flight_001 till GeoTIFF-filer med reflektans + NDVI.&quot;*

Fullständig guide: [Använda Chloros med AI-assistenter](ai-assistants.md).
{% endhint %}

***

## Nyheter i Chloros 1.2.0

* **Livekamerastyrning — ny flik Kameror.** Anslut LATTICE-kameror en i taget eller som synkroniserade flerkamerasystem (PTP-tidssynkronisering, hårdvaruaktiverad bildtagning), med överlagringar för liveförhandsgranskning, bandvisa histogram, smart automatisk exponering, en liveindexberäknare och uppdateringar av kamerans firmware direkt i appen.
* **Ljussensorer — ny flik för ljussensorer.** Anslut DAQ-U- (USB), DAQ-M- (Bluetooth) och DAQ-E- (Ethernet) sensorer; visa kalibrerade spektra i realtid (W/m²/nm), spara `.daq`-filer i ditt projekt, välj profiler för kap-korrigering och uppdatera DAQ-E-firmware via nätverket.
* **Inspelningslägen och inspelningsenheter.** Enstaka / Kontinuerlig / Intervallinspelning samt ett ”Fastest Capture”-läge endast för rådata; val per projekt av vilka kameror och exporttyper som ”Capture All” genererar; array-inspelare för indexvideo av övervakningskvalitet och rådata-burst av analyskvalitet med offline-videokompileringar.
* **LATTICE-bearbetningspipeline.** Importera LATTICE-inspelningsmappar och dela upp varje råbild till debayered-bilder, förhandsgranskning, float32-radiance (W/m²/sr/nm) och reflektansprodukter med produktvisa växlingsalternativ. Reflektansen kan härstamma från ett kalibreringsmål i bilden eller DAQ-nedstrålning; arrayjustering tillämpas på exportfiler; saknad fabrikskalibrering hämtas automatiskt utifrån kamerans serienummer.
* **Projekt minns hårdvaran.** Anslutna kameror och ljussensorer sparas med projektet (`cameras.json` / `sensors.json`) och återansluts med sina sparade inställningar när du öppnar projektet igen. Se [GUI: Projekt](projects.md).
* **Uppgraderingar av bildvisaren.** Avläsning av markörens pixel/index med korrekt skalning av reflektans per fil, lagerhistogram, ett GSD-binning-skjutreglage, rutnätslägena Per Trigger / Per Camera, LATTICE-produktvyer samt export av index/LUT-sandbox till disk.
* **CLI &amp; SDK, kraftigt utökade.** Nya kommandofamiljer `lattice`, `daq pool-*`, `project` och `time-sync`; nya `process`-alternativ (`--input-level`, produktspecifika växlingsalternativ, `--reflectance-source`, flaggor för matrisjustering); SDK smart-connect-handtag (`connect_camera` / `connect_array` / `connect_daq_sensor`) som automatiskt startar backend; `open_project()`-automatisering; SDK-hjulet ingår i installationsprogrammen och publiceras på PyPI som `chloros-sdk`.
* **Tydlig felhantering.** En `chloros-cli process`-körning som begärde produkter men inte skrev ut några misslyckas nu tydligt och avslutas med ett värde som inte är noll; lyckade körningar rapporterar hur många bildprodukter de skrev ut.
* **Ny utdatastruktur.** Produkterna placeras i `<project>/<camera>/<format>/<Product>_Images/`-mappar och behåller källfilnamnet – det är mappen, inte ett filnamnstillägg, som identifierar produkten. Se [Utdataformat för bilder](output-image-formats.md).
* **Fler ingångar, planer och språk.** Stöd för `.dng`-ingångar; alla 38 gränssnittsspråk är fullt implementerade; hårdvarubegränsningar per plan med gratis (utan inloggning) användning av upp till 4 kameror och 2 ljussensorer.
* **Tillförlitlighet.** Funktionen ”Stoppa bearbetning” avslutas korrekt med en tydlig sammanfattning av körningen, projekt med flera kameror exporterar data från varje kamera och uppgraderingar via installationsprogrammet loggar inte längre ut dig.***

Chloros finns i tre användargränssnitt:

## Chloros: GUI-program för skrivbordet

Fristående separat fönster med alla funktioner, inklusive flikarna för livekameror och ljussensorer. _Endast Windows._

## [Chloros CLI: Kommandoradsgränssnitt](CLI.md)

Batchbearbetning via kommandoraden samt livekommandon `lattice`, `daq pool-*`, `project` och `time-sync`. Perfekt för automatisering, skriptning och headless-drift. Tillgängligt på **Windows, Linux amd64 och Linux arm64 (NVIDIA Jetson)**. _CLI kräver en betald Chloros+-nivå för åtkomst._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmerbart Python-gränssnitt för automatisering och anpassade arbetsflöden: bearbetning av hela pipelinen, live-sessioner med kamera/matris, DAQ-sensorsessioner och automatisering av sparade projekt. Installeras med paketet desktop/CLI och publiceras även som `pip install chloros-sdk`. _API kräver en betald Chloros+-nivå för åtkomst._

***

## Plattformar som stöds

| Plattform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Ja | Ja | Ja |
| **Linux amd64 (x86_64)** | Nej | Ja | Ja |
| **Linux arm64 (NVIDIA Jetson)** | Nej | Ja | Ja |

För installationsanvisningar för Linux, se avsnittet [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Kom igång i tre steg

1. **Installera** — ladda ner och kör installationsprogrammet för din plattform. Se [Nedladdning](download.md).
2. **Logga in (valfritt för GUI)** — GUI-gränssnittet bearbetar bilder gratis utan konto. En [Chloros+-inloggning](chloros+-login.md) ger tillgång till parallellbearbetning, GPU-acceleration, högre enhetsgränser samt åtkomst till CLI/SDK.
3. **Skapa ditt första projekt** — öppna Chloros, skapa ett [nytt projekt](projects.md), [lägg till dina bilder](processing-images-gui/adding-files-to-a-project.md) och [börja bearbetningen](processing-images-gui/starting-the-processing.md). Om du istället vill styra hårdvara i realtid öppnar du fliken Kameror eller Ljussensorer — se [GUI: Navigering](navigation.md).***

## Chloros+

Även om Chloros är gratis att använda för de flesta uppgifter, kanske du upptäcker att du vill ha mer. Det är då en betald licens för Chloros+ kan vara till nytta för dig. Med en Chloros+-licens kan du låsa upp nya funktioner såsom:

* **Flertrådig bearbetning**: påskynda bildbearbetningen avsevärt för större projekt genom att bearbeta bilder samtidigt genom bearbetningskedjan.
* **GPU-acceleration (CUDA)**: dra nytta av dagens större GPU-minneskapacitet för att ytterligare snabba upp bildbearbetningsprocessen. Vi rekommenderar 4 GB eller mer VRAM för bästa resultat.
* **Chloros+**[**CLI**](CLI.md)**Åtkomst**: kör Chloros+ från kommandoraden för att automatisera och integrera i din egen programvara. Tillgängligt på alla betalda nivåer; tillämpas på serversidan.
* **Chloros+**[**API**](api-python-sdk.md)**Åtkomst:** kör Chloros+ från Python för programmatisk styrning, vilket möjliggör sömlös integration med dina forskningspipelines, arbetsflöden för dataanalys och anpassade applikationer. Tillgängligt på alla betalda nivåer; tillämpas på serversidan.
* **Högre hårdvarugränser**: anslut fler kameror och ljussensorer samtidigt. Utan inloggning ansluter GUI upp till 4 kameror och 2 DAQ-ljussensorer; betalda nivåer höjer båda gränserna:

| Plan | Kameror | DAQ-ljussensorer |
| --- | --- | --- |
| Iron (gratis, ingen inloggning) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Användning på flera enheter**: varje Chloros+-licens tillåter registrering av 2 eller fler enheter. Använd ditt MAPIR Cloud-konto för att hantera registrerade enheter. Lägg till stöd för fler enheter genom att uppgradera din Chloros+-licens.
* **Avancerad texturkänslig debayermetod:** en högkvalitativ kantkänslig debayer kombinerad med en AI/ML-brusreduceringsmodell som tar bort nästan allt debayeringbrus.
* **Anpassade multispektrala indexformler:** ange anpassade multispektrala index i Chloros-rasterberäknarna, både för bearbetning och i bildvisningssandlådan.
* **Linux &amp; Edge Computing:** kör Chloros på Linux x86_64- och ARM64-plattformar, inklusive NVIDIA Jetson, för fält- och kantbearbetning. Se [Linux – Översikt](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Priser och registrering</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
