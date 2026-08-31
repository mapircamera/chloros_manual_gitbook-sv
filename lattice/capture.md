# Inställningar och lägen för bildtagning

Bildtagningen på fliken ”Kameror” styrs av en röd **Capture All**-knapp och ett**Capture Settings**-fönster som avgör vad knappen ger för resultat: vilka kameror som ska användas, vilka exportformat varje kamera sparar i, och om slutaren ska avfyras en gång, kontinuerligt eller med ett visst intervall. Denna sida beskriver hela arbetsflödet – konfigurationen, själva bildtagningen, var filerna sparas på disken och hur man senare kan bearbeta dem till kalibrerade produkter. Inställningarna för kameror och kameramatriser finns på [Kamerainställningar](camera-settings.md).

{% hint style="info" %}
**Inspelningar kräver att ett projekt är öppet.** Knappen ”Spela in allt” och kugghjulet för inspelningsinställningar är inaktiverade tills ett projekt är öppet (”Skapa eller öppna ett projekt för att spara inspelningar”). Varje bildsamling sparas i projektmappen i `captures/`.
{% endhint %}

## Panelen ”Inspelningsinställningar”

Öppna den med **kugghjulet bredvid ”Spela in allt”**i kameralistan i sidofältet, eller med knappen**”Öppna inspelningsinställningar…”** längst ner i vilken kamerainställningspanel som helst. Rubriken lyder ”Inspelningsinställningar” med en ←-knapp för att gå tillbaka.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Dina val här – vilka kameror som ska ingå, kryssrutor för olika typer och inspelningsläget – sparas **per projekt** och återställs när du öppnar projektet igen.

### Inspelningslägen

Tre lägesknappar högst upp i fönstret:

| Läge | Funktion | Underinställningar (standard) |
| --- | --- | --- |
| **Enstaka** *(standard)* | En inspelning från alla valda kameror. | — |
| **Kontinuerligt**| Upptagningar i följd tills ett stoppvillkor uppfylls. | Stopp vid**Antal upptagningar** (standard 1) *eller* **Upptagningsvaraktighet** (standard 10 s; enheter: sekunder / minuter / timmar / dagar). |
| **Intervall**(timelapse) | Bildserier enligt en timer. |**Antal bilder per intervall**(standard 1) ·**Var**N enheter (standard 5 s) ·**Under** N enheter (standard 1 m). |

I kontinuerligt läge eller intervalläge ändras knappen ”Ta alla” till en **Stopp (N)**-knapp under körning, och räknar bilderna allteftersom de tas.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Välja kameror och exporttyper

Hjälptexten i rutan sammanfattar det hela: välj vilka kameror och exporttyper som ”Capture All” ska generera — allt är aktiverat som standard, och valen sparas med detta projekt.

* Knapparna **Välj alla / Välj inga** växlar alla kamerors inkluderingsrutor samtidigt.
* **Väljarreglage för massexporttyper**(två rader med knappar):**All Raw / All Debayered / All Preview / All Radiance / All Reflectance / All Index**. Varje alternativ har tre färgskalor: grönt ✓ = aktiverat för alla kameror som stöder det, gult – = aktiverat för vissa, grått = inget. En växlingsknapp är inaktiverad om ingen ansluten kamera stöder den typen. Alla blir gråa när ”Fastest Capture” är aktiverat.
* **Rader per kamera**: en kryssruta för att inkludera, plus en utvidgbar (▸/▾) lista över den kamerans tillämpliga exporttyper med individuella kryssrutor. Raden visar ett antal aktiverade, t.ex. ”4/6”.

### Exporttyper och vilka kameror som stöder dem

Det finns sex exporttyper: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Endast de tillämpliga visas i varje kameras rad:

| Exporttyp | Innehåll | RGB (FRGB) | Bayer-multispektral (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mosaik (mono: det enda bandet) direkt från sensorn | ✓ | ✓ | ✓ |
| **Debayered** | Linjär demosaikering (mono: 1-kanals gråskala) | ✓ | ✓ | ✓ |
| **Förhandsgranskning** | Fullständig visningskedja (vitbalans + gamma enligt kamerans profil; multispektral: falskfärgssträckning) | ✓ | ✓ | ✓ |
| **Strålning** | float32 W/m²/sr/nm via hela den radiometriska kedjan | — (erbjuds inte) | ✓ | ✓ |
| **Reflektans** | uint16 ρ (32768 = 1,0) | — (erbjuds inte) | ✓ — visas endast när kameran har en DAQ-ljussensor (sin egen eller ärvd från sin matris) | samma som multispektral |
| **Index** | Vegetationsindex (LUT) | — | ✓ — kräver ett aktiverat, icke-tomt indexuttryck på kameran och erbjuds inte till medlemmar i kombinerade arrayer (arrayen har ett gemensamt index) | — (ett index kräver ≥2 band; se [Monokameror och vegetationsindex](mono-indices.md)) |

Strålning och reflektans erbjuds aldrig för RGB-kameror — strålning per Bayer-matris är inte meningsfull för en bredbandsfotometrisk sensor.

### Snabbaste bildtagning

Väljaren **⚡ Snabbaste bildtagning — endast RAW**(orange när den är aktiverad) åsidosätter alla exportval till**endast RAW** — plus en gratis sammansatt bild med kombinerat index för bildserier — så att bilden sparas så snabbt som möjligt: beräkningarna av strålning, reflektans och visning hoppas över helt vid bildtagningen.

{% hint style="info" %}
**En `.daq` sparas fortfarande.** När en ljussensor är tilldelad skriver ”Snabbast möjliga inspelning” fortfarande ner DAQ:s nedåtriktade avläsning bredvid råbilderna — så att strålnings-, reflektans- och indexprodukter alla kan skapas senare genom ombearbetning (se [Ombearbetning av inspelningar](#re-processing-captures-into-calibrated-products)). Fastest Capture påverkar inte heller dina val i kryssrutorna: stäng av funktionen så återställs de.
{% endhint %}

### Kontroller per matris

Varje ansluten matris får ett eget gruppkort i fönstret:

* **Kryssrutan Inkludera** (tre lägen för medlemmarna) och matrisens namn med dess visningsläge: ”(kombinerad | separat)”.
* Kryssrutan **Justerad**(standardinställning**på**): anpassar exporten av medlemmar till matrisens justeringsprofil så att exporten är pixelregistrerad mellan kamerorna. Rådata förblir oförvrängda men bär med sig transformationen i sina metadata. (Profilen i sig beräknas i [panelen för arrayinställningar](camera-settings.md#alignment-co-registration-combined-only).)
* Kameraraderna för medlemmarna är inbäddade i kortet.

Arraykortet innehåller också två inspelare. Tänk på dem som **övervakning kontra analys**:

| Inspelare | Grad | Vad den spelar in |
| --- | --- | --- |
| **● Spela in indexvideo / ■ Avsluta inspelning** *(endast kombinerade arrayer)* | **Övervakning** | Den kombinerade indexkompositen i realtid till video med 10 fps — 8-bitars, förhandsgranskningsupplösning, inbakad LUT. Kräver ett öppet projekt och en strömmande livevy. Visar bildrutor + förfluten tid under inspelning. |
| **⦿ Spela in rå burst / ■ Avsluta rå bildserie** *(valfri matris)* | **Analys**| Råa Bayer-bilder med live-fångstfrekvens (ingen bearbetning) plus en bildförteckning per bildruta och `.daq`-mätvärden, till `captures/bursts/`. Efter en serie bilder visas en**Skapa video**-knapp: den bearbetar serien offline till kalibrerad video — kombinerat index och/eller radianse/reflektans/index per kamera — plus valfria TIFF-filer. Skapandet av det kombinerade indexet startar automatiskt när du avbryter serien. |##

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

Flödet

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

”Capture All” Tryck på **Capture All** i kameralistan i sidofältet:

1. Varje inkluderad, synlig och icke-pausad kamera tar bilder med sina valda exporttyper. **Arrayer utlöses som en synkroniserad utlösare** (en enda synkroniserad grupp över alla medlemmar — se [Multi-Camera Arrays](arrays.md)); fristående kameror tar bilder individuellt.
2. Dolda (ögon) eller pausade kameror hoppas över. En array blockeras endast helt när *alla* dess medlemmar är dolda eller pausade.
3. När en ljussensor tilldelas sparas den motsvarande DAQ-mätningen av nedåtriktat ljus som en `.daq`-fil tillsammans med bildmaterialet – även vid inspelningar som endast består av rådata – så att radiometriska produkter alltid kan härledas senare.
4. Knappen visar förloppet i realtid – ”Spelar in… klart/totalt” – och blir **Stopp (N)** i kontinuerligt/intervalläge. Varje inspelningsobjekt har en timeout på 300 s.
5. När passet är klart visar ett resultatmeddelande **”N filer sparade”**eller**”N sparade, F misslyckades”**, samt ”(S dold/pausad/hoppad över)” när kameror har hoppats över.

## Var inspelningarna sparas

Inspelningarna sparas under det öppna projektet i `<project>/captures/`. Varje exporttyp hamnar i sin **egen undermapp**, så en inspelning med flera nivåer blandar aldrig ihop typerna:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` är tidsstämpeln för inspelningen och `<serial>` är kamerans serienummer. Fristående inspelningar namnges `capture_<ts>_SN<serial>_<level>`; matrisinspelningar från en synkroniserad utlösare namnges `sync_<ts>_SN<serial>_<level>` och **delar en tidsstämpel mellan alla kameror i gruppen** (nivåsuffixet utelämnas när en kamera endast sparar en enda nivå).
* **En avvikelse att känna till:** visningsnivån lagras i en mapp med namnet `preview/` medan filerna behåller `_display` i namnet — mapp och suffix skiljer sig åt endast för den nivån.
* Okända nivåer placeras i en mapp med sitt eget namn; om en undermapp inte kan skapas sparas filen i rotkatalogen för bildtagningar istället för att gå förlorad.
* TIFF-filer från bildtagningar komprimeras som standard förlustfritt (DEFLATE) och innehåller alla metadata om kalibrering och bearbetning **inuti filens XMP** – bildtagningarna är självbeskrivande och har inga sidfiler utöver filen med namnet `.daq`.

Detta är samma layout som `chloros-cli lattice capture` / `array-capture` skriver till sin `-o`-katalog — dokumenterat i [CLI-referensen § Hur en inspelningsmapp ser ut](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Ombearbetning av inspelningar till kalibrerade produkter

Inspelade råbilder plus den sparade `.daq` är allt som bearbetningskedjan behöver — det är därför Fastest Capture är säkert att använda för verkligt arbete.

* **GUI**: lägg till mappen med bilder i ett projekt ([Lägga till filer i ett projekt](../processing-images-gui/adding-files-to-a-project.md)) och bearbeta som vanligt.
* **CLI**: peka `process` mot**rotmappen för bilderna**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` importerar normalt endast den mapp du anger, men när den mappen inte innehåller några bilder och har undermappar går programmet automatiskt vidare till dessa — så att undermapparna på olika nivåer och filerna i roten `.daq` hämtas i ett enda steg. Varje insamling importeras som en **enskild bild** med de övriga nivåerna bifogade som visningslägen, inte som en bild per nivå.

Det går också att namnge en undermapp på en nivå direkt (t.ex. `…/captures/raw/`), men då utelämnas rotfilerna `.daq` – kopiera dem samtidigt när du återberäknar en radiometrisk produkt från `raw/`, annars har tidsstämpeln inget att matchas mot.

{% hint style="warning" %}
**Bearbetningen börjar alltid från `raw`.**Inom varje inspelning är råbilden källan till bearbetningskedjan; `debayered`, `radiance`, `reflectance` och `preview` finns som visningslägen men matas aldrig tillbaka genom bearbetningskedjan — om en härledd produkt bearbetas på nytt skulle vignettering, färg och strålningsberäkningar som redan är inbakade i dess pixlar tillämpas på nytt, så Chloros avvisas istället för att bearbetas två gånger. Renderingarna `index/` och `composite/` bearbetas aldrig alls (de är utdata, inte inspelningar). En ”captures”-mapp som sparats**utan** råimport visas normalt, men `process` hoppar över den och meddelar detta; `--input-level {raw,debayered,processed}` är den avsiktliga nödutgången som tvingar fram en ingångspunkt. Se [CLI-referensen](../reference/cli-reference.md#what-a-captures-folder-looks-like) för de exakta meddelandena vid hopp.
{% endhint %}

Ytterligare två beteenden som är värda att känna till vid skriptning av ombearbetning:

* En `chloros-cli process`-körning som begärde produkter men inte skrev ut **några bildprodukter misslyckas med ett tydligt felmeddelande och avslutas med ett värde som inte är noll** — du kommer aldrig att få en tyst, tom körning. Framgångsrika körningar rapporterar antalet produkter. (En avsiktlig körning som endast omfattar metadata räknas fortfarande som en framgång.)
* Återimporterade bearbetade exporter tar aldrig en rårut i en inspelning – den ursprungliga rådata förblir alltid källan i pipelinen.

## CLI-motsvarigheter

Allt på denna sida kan köras i headless-läge. GUI-insamlingslägena motsvarar direkt `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Enstaka | `chloros-cli lattice array-capture` |
| Kontinuerlig | `array-capture --continuous [--count N] [--duration S]` |
| Intervall | `array-capture --interval S [--duration S]` |
| Snabbaste inspelning | `array-capture --fastest` |
| Justerad kryssruta | `--aligned / --no-aligned` |
| Kryssrutor för exporttyp | `--processing LEVEL` eller `--levels L1,L2,…` (standard `all`) |
| Spela in indexvideo | `chloros-cli lattice array-record` |
| Spela in rå bildserie / Skapa video | `chloros-cli lattice array-burst` / `array-build-video` |

Fullständiga flaggtabeller, alternativet smart-AE-stabiliserad inspelning (`--smart`) och modellen med konstant hastighet finns i [CLI Referens § Inspelningslägen, inspelningsenheter och offlinebearbetning](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
