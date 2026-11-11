---
metaLinks: {}
---

# Komma igång

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros
är ett program från [MAPIR
](https://www.mapir.camera) för bearbetning av multispektrala bilder, fjärrstyrning av MAPIR
-hårdvara och registrering av sensordata. Chloros
1.3.0 stöder hela MAPIR
-produktfamiljen:

* **Survey3
-kameror** — bearbetar RAW+JPG-bilder till kalibrerade reflektans- och vegetationsindexkartor. Se [Kameror som stöds](supported-cameras.md).
* **LATTICE-kameror** — anslut GigE-multispektralkameramodulerna i realtid, enskilt eller som synkroniserade multikamerasystem: förhandsgranska, ta bilder och bearbeta till kalibrerade strålnings- och reflektansprodukter. Se [avsnittet om LATTICE](lattice/README.md).
* **DAQ-ljussensorer** — DAQ-U (USB), DAQ-M (Bluetooth) samt DAQ-E och DAQ-E-S (Ethernet) spektralsensorer: kalibrerade spektra i realtid, `.daq`-inspelningar samt nedåtriktad belysning för reflektansbearbetning. Se [DAQ-avsnittet](daq/README.md).

{% hint style="success" %}
**Nyheter i Chloros
1.3.0**: AI-assistenter kan styra Chloros
via `chloros-mcp`, och CLI
, SDK
och `chloros-mcp` erbjuder samma funktioner. LATTICE-indexprodukterna har förbättrats, fullsensormatriser är snabbare och fungerar i alla nätverk, och backend-systemet fortsätter att köras mellan kommandon. Se listan över nyheter nedan och [ladda ner](download.md) installationsprogrammen.
{% endhint %}

{% hint style="info" %}
**Använder du Chloros
med en AI-assistent?** Den här manualen är utformad för just det. Be din assistent att gå till:

* `https://mapir.gitbook.io/chloros/llms.txt` — maskinläsbart index över alla sidor.
* Vilken sida som helst som rå Markdown — lägg till `.md` till dess URL
(t.ex. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI
-referensen](reference/cli-reference.md) och [SDK
-referensen](reference/sdk-reference.md) — fullständiga referenssidor med exakta värden, skrivna för användning av stora språkmodeller (LLM).

Exempel på prompt: *”Läs https://mapir.gitbook.io/chloros/reference/cli-reference.md, och skriv sedan ett skript som loggar in och bearbetar mappen ~/flights/flight_001 till GeoTIFF-filer med reflektans och NDVI
.”*

Eller låt den köra Chloros
direkt: `chloros-mcp`, en MCP-server som installeras tillsammans med SDK
, fungerar med Claude Desktop, Claude Code, Cursor och andra MCP-klienter.

Fullständig guide: [Använda Chloros
med AI-assistenter](ai-assistants.md).
{% endhint %}

***

## Nyheter i Chloros
1.3.0

* **AI-assistenter kan styra Chloros
.** `chloros-mcp`, en MCP-server som installeras tillsammans med Python
-SDK
:et, gör det möjligt för Claude Desktop, Claude Code, Cursor och andra MCP-klienter att bearbeta bilder och styra LATTICE-kameror, arrayer och DAQ-ljussensorer. Se [Använda Chloros
med AI-assistenter](ai-assistants.md).
* **CLI
, SDK
och `chloros-mcp` erbjuder samma funktioner.** Av säkerhetsskäl finns endast inloggning, DAQ-E-konfigurering och de direkta DAQ-servrarna kvar i CLI
. Vissa 1.2.0-skript behöver ändras: se **Ändringar i 1.3.0** högst upp i [CLI
-referensen](reference/cli-reference.md) och [SDK
-referensen](reference/sdk-reference.md).
* **Förbättrade LATTICE-indexprodukter.** NDVI
och de andra indexprodukterna som exporteras från LATTICE-inspelningar har förbättrats i alla bearbetningslägen.
* **Alternativ för Windows
-installationsprogrammet.**Under**Systeminställningar**erbjuder installationsprogrammet alternativen**Aktivera långa sökvägar i Windows
**och**Uteslut Chloros
-backend från Microsoft Defender-skanning**, vilka båda är markerade som standard. Det första alternativet gör att projektmappar kan ha namn som är längre än 260 tecken; det andra förhindrar att realtidsskanning saktar ner kamerainspelningen (cirka 30 % av inspelningshastigheten vid höga hastigheter). Installationsprogrammet lägger nu också till CLI
-mappen till `PATH` endast en gång och tar bort de dubbla poster som tidigare installationsprogram lämnat kvar. Se [Nedladdning](download.md).
* **Djupa projektmappar avvisas direkt.** När Windows
-långa sökvägar är avstängda (Windows
-standardinställningen) avvisas en projektmapp som är för djup för gränsen på 260 tecken innan något körs, och den längsta projektmappen som passar samt information om hur man aktiverar långa sökvägar visas. Tidigare skapades inga produkter vid en sådan körning. När långa sökvägar är aktiverade behåller produkter i mycket djupa mappar nu sina metadata (GPS- och XMP-taggar).
* **Klippmasker.** När bandavskiljningen av en LATTICE-radiance- eller reflektansprodukt ger ett negativt bandvärde, klipps det värdet till noll, och en mask per pixel i en `clamp_masks/`-mapp bredvid produktmapparna registrerar vilka pixlar och band som påverkades. Produkten namnger sin mask i taggen `Chloros:UnmixClampMask`. Live-vyer beskärs nu på exakt samma sätt som vid export. Se [Utgångsbildformat](output-image-formats.md).
* **Produkterna beskriver sig själva.** Indexprodukter har sina egna bearbetningstaggar (`Chloros:ProcessingLevel=index`, indexuttrycket och om de beräknats utifrån strålningsintensitet eller reflektans) istället för källrawfilernas. Survey3
-produkter har Chloros
-bearbetningstaggar i alla bearbetningslägen. LATTICE-råfiler och debayered-filer behåller sensorns ursprungliga orientering och registrerar sin orientering i XMP; bildvisaren roterar dem endast för visning.
* **Linux
-paketen levereras i okomprimerat format.** Backend och CLI
installeras som mappar under `/usr/lib/chloros/`, så de packas inte längre upp automatiskt vid varje start. På x86_64 tar det inte längre cirka 100 sekunder för backend att starta, och `.deb` är ungefär hälften så stor som tidigare. Det medföljande ExifTool körs nu på en ren Ubuntu-installation. `.deb` installerar nu även Python
-SDK
för systemet `python3` (installera det med `sudo apt install ./chloros_1.3.0_<arch>.deb`), ökar storleken på de socket-mottagningsbuffertar i kärnan som kameraströmmarna behöver och lägger till en backend-tjänst som körs som din användare. Jetson-paketet, `chloros_1.3.0_arm64_jp6.deb`, är byggt för JetPack 6 och bearbetar data på Jetsons GPU med JetPacks CUDA-version av PyTorch. På Linux
når varje exporterad bild nu disken en gång (skrivs och märks i minnet, sedan kopieras den till rätt plats): på en Jetson, där disken bestämmer takten, tar en LATTICE-körning med 288 bilder 14 minuter istället för 24. Se [Linux
-installation](linux/linux-installation.md).
* **LATTICE OCN
-kameror avläser mer exakt i solljus.** Radiance, reflektans och index får en snabb korrigering efter bandsepareringen, så att ett grått mål avläses nära neutralt NDVI
i solen. RGN
-kameror är oförändrade.
* **Felrapporter direkt i appen.** En felikon högst upp på fliken Debug Log skickar en beskrivning, upp till 10 bilder, loggarna och systeminformation till MAPIR
. Se [GUI: Navigering](navigation.md).
* **LATTICE-matriser.**Matriser behåller hela sensorn som standard, och packad 12-bitarsöverföring (`--packed`, standard `auto`) höjer bildhastigheten för hela sensorn med ungefär en tredjedel; en array fungerar på vilket nätverk som helst med den bildfrekvens som nätverket tillåter (en långsam länk sänker bildfrekvensen istället för att avvisa arrayen eller minska dess bildrutor); en array vars kameror startas om återhämtar sig av sig själv, med PTP återaktiverat; automatisk exponering sänker inte längre den frekvensen i svagt ljus (den fyller ut perioden och höjer sedan förstärkningen); Kontinuerlig inspelning håller jämna steg med snabba arrayer; en**Highlights**-kontroll för RGB
-kameror; och SDK
:s `ArraySession` får `grab()`, `frame_stream()` och `continuous()`. Se [LATTICE-avsnittet](lattice/README.md).
* **Stabilare automatisk exponering.** Exponeringen hålls konstant vid oförändrat ljus och stabiliseras inom några sekunder efter en förändring; små höjdpunkter gör inte längre scenen mörkare; vid manuell exponering förblir förstärkningen inställd på Auto; och en kamera som slutar svara i starkt solsken hålls på den nivå som motsvarar ljuset, avläst av ljussensorn eller, om ingen är ansluten, av kamerans egna bildrutor. Se [Kamerainställningar](lattice/camera-settings.md).
* **Ljussensorer.** **Stöd för DAQ-E-S:** dess fasta diffusor kräver ingen kapselkorrigering, och varje avläsning innehåller enhetens lutning, rullning och tippning. En inspelning anger om locket valdes eller antogs, och bearbetningen varnar vid ett antaget lock: DAQ-U, DAQ-M och DAQ-E levereras utan lock, så namnge det som är monterat. Live-inspelningar av typen `.daq` överlever ett strömavbrott (spara en `.daq-journal`-fil tillsammans med dess `.daq`), och en DAQ-E som slutar leverera data återansluts med avbrottet markerat i dess inspelning. Ett projekt som endast innehåller inspelningar från ljussensorer listar och bearbetar dem från appen och CLI
:n, och en inspelning gjord med `chloros_scripts` listas som den DAQ som gjorde den, med en gräns som du kan ställa in. Varje inspelning sparar den DAQ-avläsning som användes. Se [DAQ-avsnittet](daq/README.md).
* **Bearbetning.**LATTICE-ramar påbörjar bearbetningen under måldetektering. Referenser för mål och DAQ bestäms per band. Ett flyttat projekt länkar om sina filer. Exporter från en 8-bitarsinspelning är 8-bitars, med ett nytt**TIFF
-format (8-bitars)**. En Survey3
-körning utan kalibreringsmål sparas som dess reservprodukt (`Vignette_Corrected_Images` eller `Sensor_Response_Images`), inte som reflektans. En reflektansbilds DLS-taggar per band är alltid dess egen bildrams. Texture Aware avvisas för LATTICE-inspelningar, som alltid får standard-debayer.
* **CLI
&amp; SDK
.** Relativa utdatavägar hamnar nu i den mapp där du kör kommandot eller skriptet (en installerad backend brukade lösa dem i sin egen installationsmapp och misslyckades). Backend-modulen fortsätter att köras mellan kommandon och skript och stänger av sig själv efter 30 minuters inaktivitet; `chloros-cli backend status` och `backend stop` hanterar detta. Inläsningskommandon namnger varje fil som de inte kunde skriva och avslutas med kod 1, och upprepade inläsningar avbryts när hårddisken är full. En lyckad körning av `process` rapporterar `Image products written: N (plus M clamp masks)`. I Windows
hittar SDK
:s `ChlorosLocal` / `process_folder` hittar den installerade backenden (och använder en som redan körs), och `read_image_audit_tags` fungerar från en vanlig `pip install`.

***

## Nyheter i Chloros
1.2.0

* **Livekamerastyrning — ny flik Kameror.** Anslut LATTICE-kameror en i taget eller som synkroniserade multikamerauppsättningar (PTP-tidssynkronisering, hårdvarutriggad bildtagning), med överlägg för liveförhandsgranskning, bandvisa histogram, smart automatisk exponering, en liveindexberäknare och uppdateringar av kamerans firmware direkt i appen.
* **Ljussensorer — ny flik för ljussensorer.** Anslut DAQ-U- (USB), DAQ-M- (Bluetooth) och DAQ-E- (Ethernet) sensorer; visa kalibrerade spektra i realtid (W/m²/nm), spara `.daq`-filer i ditt projekt, välj profiler för kap-korrigering och uppdatera DAQ-E-firmware via nätverket.
* **Inspelningslägen och inspelningsenheter.** Enstaka / Kontinuerlig / Intervallinspelning samt ett ”Fastest Capture”-läge endast för rådata; val per projekt av vilka kameror och exporttyper som ”Capture All” genererar; array-inspelare för indexvideo av övervakningskvalitet och rådata-bursts av analyskvalitet med offline-videokompileringar.
* **LATTICE-bearbetningspipeline.** Importera LATTICE-inspelningsmappar och dela upp varje råbild i debayered-bilder, förhandsgranskning, float32-radiance (W/m²/sr/nm) och reflektansprodukter med produktvisa växlingsalternativ. Reflektansen kan härröra från ett kalibreringsmål i bilden eller DAQ-nedstrålning; arrayjustering tillämpas på exportfiler; saknad fabrikskalibrering laddas ned automatiskt utifrån kamerans serienummer.
* **Projekt minns hårdvaran.** Anslutna kameror och ljussensorer sparas med projektet (`cameras.json` / `sensors.json`) och återansluts med sina sparade inställningar när du öppnar projektet igen. Se [GUI: Projekt](projects.md).
* **Uppgraderingar av bildvisaren.** Avläsning av markörens pixel/index med korrekt skalning av reflektans per fil, lagerhistogram, ett GSD-binning-skjutreglage, rutnätslägena Per Trigger / Per Camera, LATTICE-produktvyer samt export av Index/LUT-sandbox till disk.
* **CLI
och SDK
har utökats avsevärt.** Nya kommandofamiljerna `lattice`, `daq pool-*`, `project` och `time-sync`; nya `process`-alternativ (`--input-level`, produktvisa växlingsalternativ, `--reflectance-source`, flaggor för matrisjustering); SDK
-handtag för smart anslutning (`connect_camera` / `connect_array` / `connect_daq_sensor`) som automatiskt startar backend; `open_project()`-automatisering; SDK
-hjulet ingår i installationsprogrammen och publiceras på PyPI som `chloros-sdk`.
* **Tydlig felhantering.** En `chloros-cli process`-körning som begärde produkter men inte skrev ut några misslyckas nu tydligt och avslutas med ett värde större än noll; lyckade körningar rapporterar hur många bildprodukter de skrev ut.
* **Ny utdatastruktur.** Produkterna placeras i `<project>/<camera>/<format>/<Product>_Images/`-mappar och behåller källfilnamnet — det är mappen, inte ett filnamnstillägg, som identifierar produkten. Se [Utdataformat för bilder](output-image-formats.md).
* **Fler ingångar, planer och språk.** Stöd för `.dng`-ingångar; alla 38 gränssnittsspråk är fullständigt implementerade; hårdvarubegränsningar per plan med gratis (utan inloggning) användning av upp till 4 kameror och 2 ljussensorer.
* **Tillförlitlighet.** ”Stop Processing” avslutas korrekt med en tydlig sammanfattning av körningen, projekt med flera kameror exporterar data från varje kamera och installationsprogrammet loggar inte längre ut dig vid uppgraderingar.***

Chloros
finns tillgängligt i tre användargränssnitt:

## Chloros
: GUI-program för skrivbordet

Fristående separat fönster med alla funktioner, inklusive flikarna för livekameror och ljussensorer. _Endast Windows._

## [Chloros
CLI
: Kommandoradsgränssnitt](CLI.md)

Batchbearbetning via kommandoraden samt live-kommandon `lattice`, `daq pool-*`, `project` och `time-sync`. Perfekt för automatisering, skriptning och headless-drift. Tillgängligt på **Windows
, Linux
amd64 och Linux
arm64 (NVIDIA Jetson)**. _CLI
kräver en betald Chloros
+-nivå för åtkomst._

## [Chloros
API
: Python
SDK
](api-python-sdk.md)

Programmeringsgränssnitt i Python
för automatisering och anpassade arbetsflöden: bearbetning av hela pipelinen, live-sessioner med kamera/matris, DAQ-sensorsessioner och automatisering av sparade projekt. Installeras tillsammans med desktop-/CLI-paketet och publiceras även som `pip install chloros-sdk`. _API kräver ett betalt Chloros
+-abonnemang för åtkomst._

***

## Plattformar som stöds

| Plattform | GUI | CLI
| Python
SDK
|
| --- | --- | --- | --- |
| **Windows
10/11 (x64)** | Ja | Ja | Ja |
| **Linux
amd64 (x86_64)** | Nej | Ja | Ja |
| **Linux
arm64 (NVIDIA Jetson)** | Nej | Ja | Ja |

För installationsanvisningar för Linux
, se avsnittet [Linux
&amp; Edge Computing](linux/linux-overview.md).

***

## Kom igång i tre steg

1. **Installera** — ladda ner och kör installationsprogrammet för din plattform. Se [Nedladdning](download.md).
2. **Logga in (valfritt för GUI)** — GUI-gränssnittet bearbetar bilder gratis utan konto. En [Chloros
+-inloggning](chloros+-login.md) ger tillgång till parallellbearbetning, GPU-acceleration, högre enhetsgränser samt åtkomst till CLI
och SDK
.
3. **Skapa ditt första projekt** — öppna Chloros
, skapa ett [nytt projekt](projects.md), [lägg till dina bilder](processing-images-gui/adding-files-to-a-project.md) och [börja bearbeta](processing-images-gui/starting-the-processing.md). Om du istället vill styra hårdvara i realtid öppnar du fliken Kameror eller Ljussensorer — se [GUI: Navigering](navigation.md).

***

## Chloros
+

Chloros
är gratis att använda för de flesta uppgifter, men kanske vill du ha mer. Då kan en betald licens för Chloros
+ vara till nytta för dig. Med en Chloros
+-licens kan du låsa upp nya funktioner såsom:

* **Flertrådig bearbetning**: påskynda bildbearbetningen avsevärt för större projekt genom att bearbeta bilder samtidigt genom bearbetningskedjan.
* **GPU (CUDA)-acceleration**: dra nytta av dagens större GPU-minnesalternativ för att ytterligare påskynda bildbearbetningskedjan. Vi rekommenderar 4 GB eller mer VRAM för bästa resultat.
* **Chloros
+**[**CLI**](CLI.md) **Åtkomst**: kör Chloros
+ från kommandoraden för att automatisera och integrera det i din egen programvara. Tillgängligt på alla betalda nivåer; tillämpas på serversidan.
* **Chloros
+**[**API**](api-python-sdk.md) **Åtkomst:** kör Chloros
+ från Python
för programmatisk styrning, vilket möjliggör sömlös integration med dina forskningspipelines, arbetsflöden för dataanalys och anpassade applikationer. Tillgängligt på alla betalda nivåer; tillämpas på serversidan.
* **Högre hårdvarugränser**: anslut fler kameror och ljussensorer samtidigt. Utan inloggning ansluter GUI upp till 4 kameror och 2 DAQ-ljussensorer; betalda nivåer höjer båda gränserna:

| Plan | Kameror | DAQ-ljussensorer |
| --- | --- | --- |
| Iron (gratis, ingen inloggning) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Användning på flera enheter**: varje Chloros
+-licens tillåter registrering av 2 eller fler enheter. Använd ditt MAPIR
Cloud-konto för att hantera registrerade enheter. Lägg till stöd för fler enheter genom att uppgradera din Chloros
+-licens.
* **Avancerad texturmedveten debayer-metod:** en högkvalitativ kantmedveten debayer i kombination med en AI/ML-brusreduceringsmodell som tar bort nästan allt debayeringsbrus.
* **Anpassade multispektrala indexformler:** ange anpassade multispektrala index i Chloros
-rasterkalkylatorerna, både för bearbetning och i bildvisningssandlådan.
* **Linux
och edge-beräkning:** kör Chloros
på Linux
x86_64- och ARM64-plattformar, inklusive NVIDIA Jetson, för fält- och edge-bearbetning. Se [Linux
-översikt](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Priser och registrering för Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
