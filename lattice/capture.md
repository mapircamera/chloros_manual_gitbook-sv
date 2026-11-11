# Inställningar och lägen för bildtagning

Bildtagningen på fliken ”Kameror” styrs av en röd **Capture All**-knapp och ett**Capture Settings**-fönster som avgör vad knappen ger för resultat: vilka kameror som ska användas, vilka exportformat varje kamera ska spara i, och om slutaren ska utlösas en gång, kontinuerligt eller med ett visst intervall. Denna sida beskriver hela arbetsflödet – konfigurationen, själva fotograferingen, var filerna sparas på hårddisken och hur man senare bearbetar dem till kalibrerade produkter. Inställningarna för kameror och kameramatriser finns på [Kamerainställningar](camera-settings.md).

{% hint style="info" %}
**Inspelningar kräver att ett projekt är öppet.** ”Capture All” och kugghjulet för bildinställningar är inaktiverade tills ett projekt är öppet (”Skapa eller öppna ett projekt för att spara bilder”). Varje bild sparas i projektmappen i `captures/`.
{% endhint %}

## Panelen ”Inspelningsinställningar”

Öppna den med **kugghjulet bredvid ”Spela in allt”**i kameralistan i sidofältet, eller med knappen**”Öppna inspelningsinställningar…”** längst ned i valfri inställningspanel för en enskild kamera. Rubriken lyder ”Inspelningsinställningar” med en ←-knapp för att gå tillbaka.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Dina val här – valda kameror, kryssrutor per typ och inspelningsläget – sparas **per projekt** och återställs när du öppnar projektet igen.

### Inspelningslägen

Tre lägesknappar högst upp i fönstret:

| Läge | Funktion | Underinställningar (standard) |
| --- | --- | --- |
| **Enstaka** *(standard)* | En inspelning från alla valda kameror. | — |
| **Kontinuerlig**| Inspelningar i följd tills ett stoppvillkor uppfylls. | Avbryt vid**antal bilder** (standard 1) *eller* **bildtagningstid** (standard 10 s; enheter: sekunder / minuter / timmar / dagar). |
| **Intervall**(timelapse) | Bildserier enligt en timer. |**Bilder per intervall**(standard 1) ·**Var**N enheter (standard 5 s) ·**I** N enheter (standard 1 m). |

I läget Kontinuerligt eller Intervall förvandlas knappen Ta alla bilder till en **Stopp (N)**-knapp under körning, och räknar bilderna allteftersom de tas. En fullständig enhet avbryter körningen (se [Flödet för ”Capture All”](#the-capture-all-flow)).**Kontinuerligt läge på arrayer körs i backend (nytt i 1.3.0).** När alla kameror i listan tillhör en array (fristående kameror är dolda eller exkluderade) skickar kontinuerligt läge inte längre en begäran per inspelning: Chloros-backend kör själva inspelningsslingan, skriver en synkroniserad grupp medan den väljer nästa, och knappen avläser antalet. Med fristående kameror i listan gör Chloros inspelningar pass för pass som tidigare.

{% hint style="warning" %}
**Snabb inspelning på Windows: se till att Chloros-backend hålls undantagen från Microsoft Defender.** Realtidsskanning av varje fil som en inspelning med hög hastighet skriver kostar cirka 30 % av inspelningshastigheten i MAPIRs tester (hundratals filer per sekund vid ett litet ROI). Windows-installationsprogrammet erbjuder alternativet **”Uteslut Chloros-backend från Microsoft Defender-skanning (mycket snabbare kamerafångst; rekommenderas)”**, som är markerat som standard, och tar bort undantaget vid avinstallation. Om du har avmarkerat det, kör installationsprogrammet igen med alternativet markerat, eller lägg till ett undantag för Defender-processen för `chloros-backend.exe` (i mappen `backend` i Chloros-installationsmappen). När Defender använder en eller flera CPU-kärnor under en matris kontinuerlig inspelning noterar Chloros-backend detta en gång i sin logg, med samma två åtgärder.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Välja kameror och exporttyper

Hjälptexten i rutan sammanfattar det: välj vilka kameror och exporttyper som ”Capture All” ska generera – allt är aktiverat som standard, och valen sparas med detta projekt.

* **Knapparna ”Välj alla” / ”Välj inga”** växlar alla kamerors inkluderingsrutor samtidigt.
* **Väljare för massexporttyper**(två rader med knappar):**Alla Raw / Alla Debayered / Alla Förhandsgranskning / Alla Radiance / Alla Reflektans / Alla Index**. Varje alternativ har tre olika färger: grönt ✓ = aktiverat för alla kameror som stöder det, gult – = aktiverat för vissa, grått = inga. En knapp är inaktiverad om ingen ansluten kamera stöder den typen. Alla blir gråa när ”Fastest Capture” är aktiverat.
* **Rader per kamera**: en kryssruta för att inkludera, plus en utvidgbar (▸/▾) lista över den kamerans tillämpliga exporttyper med individuella kryssrutor. Raden visar ett antal aktiverade som ”4/6”.

### Exporttyper och vilka kameror som stöder dem

Det finns sex exporttyper: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Endast de tillämpliga visas i varje kameras rad:

| Exporttyp | Innehåll | RGB (FRGB) | Bayer-multispektral (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mosaik (mono: det enda bandet) direkt från sensorn | ✓ | ✓ | ✓ |
| **Debayered** | Linjär demosaik: kalibreringspaketets mörkförskjutning (DSNU) avlägsnas från rådata, därefter utförs en kantmedveten demosaik — ingen linearitetskorrigering, flat field eller unmix. Sparas som en 3-kanals RGB-TIFF-fil (mono: 1-kanals gråskala) | ✓ | ✓ | ✓ |
| **Förhandsgranskning** | Fullständig visningskedja (vitbalans + gamma enligt kamerans profil, plus RGB-kamerans inställning för höjdpunkter; multispektral: sträckning med falska färger) | ✓ | ✓ | ✓ |
| **Strålningsintensitet** | float32 W/m²/sr/nm via den fullständiga radiometriska kedjan | — (erbjuds inte) | ✓ | ✓ |
| **Reflektans** | uint16 ρ (32768 = 1,0) | — (erbjuds inte) | ✓ — visas endast när kameran har en DAQ-ljussensor (egen eller ärvd från sin matris) | samma som multispektral |
| **Index** | Vegetationsindex (LUT) -rendering | — | ✓ — kräver ett aktiverat, icke-tomt indexuttryck på kameran och erbjuds inte till medlemmar i kombinerade matriser (matrisen har ett gemensamt index) | — (ett index kräver ≥2 band; se [Monokameror och vegetationsindex](mono-indices.md)) |

Strålning och reflektans erbjuds aldrig för RGB-kameror — strålning per Bayer-pixel är inte meningsfull för en bredbandsfotometrisk sensor.

### Snabbaste inspelning

Väljaren **⚡ Snabbaste inspelning — endast RAW**(orange när den är aktiverad) åsidosätter alla exportval till**endast RAW** — plus en kostnadsfri sammansatt bild med kombinerat index för matriser — så att bilden sparas så snabbt som möjligt: beräkningarna för strålning/reflektans/visning hoppas över helt vid inspelningstillfället.

{% hint style="info" %}
**En `.daq` sparas fortfarande.** När en ljussensor är tilldelad skriver ”Snabbast möjliga insamling” fortfarande ner DAQ:s nedåtriktade avläsning (i insamlingarnas `daq/`-mapp) — så att strålnings-, reflektans- och indexprodukter alla kan skapas senare genom ombearbetning (se [Ombearbetning av mätningar](#re-processing-captures-into-calibrated-products)). Fastest Capture påverkar inte dina val i kryssrutorna: stäng av funktionen så återställs de.
{% endhint %}

### Kontroller per array

Varje ansluten array får ett eget gruppkort i fönstret:

* **Kryssrutan Inkludera** (tre lägen för medlemmarna) och arrayens namn med visningsläge: ”(kombinerad | separat)”.
* Kryssrutan **Justerad**(standardinställning**på**): anpassar exporten av medlemmarna till arrayens justeringsprofil så att exporten är pixelregistrerad mellan kamerorna. Raw-data förblir oförvrängda men bär med sig transformationen i sina metadata. (Profilen i sig beräknas i [fönstret för arrayinställningar](camera-settings.md#alignment-co-registration).) Filerna för arraymedlemmarna sparas i varje kameras ursprungliga orientering, där eventuella inställningar för spegling/rotation registreras i filen istället för att tillämpas.
* Kameraraderna i arrayen är inbäddade i kortet.

Arraykortet innehåller också två inspelare. Tänk på dem som **övervakning kontra analys**:

| Inspelare | Grad | Vad den spelar in |
| --- | --- | --- |
| **● Spela in indexvideo / ■ Avsluta inspelning** *(endast kombinerade arrayer)* | **Övervakning** | Den kombinerade indexkompositen i realtid som video med 10 fps — 8-bitars, förhandsgranskningsupplösning, inbakad LUT. Kräver ett öppet projekt och en strömmande livevy. Visar bildrutor + förfluten tid under inspelning. |
| **⦿ Spela in rå bildserie / ■ Avbryt rå bildserie** *(valfri matris)* | **Analys**| Råa Bayer-bilder med live-inspelningens hastighet (ingen bearbetning) plus ett manifest per bildruta och `.daq`-värden, till `captures/bursts/`. Efter en serie visas knappen**Skapa video**: den bearbetar serien offline till kalibrerad video — kombinerat index och/eller strålning/reflektans/index per kamera — plus valfria TIFF-filer. Skapandet av den kombinerade indexvideon startar automatiskt när du stoppar serien. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Flödet

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

”Capture All”

Tryck på **Capture All** i kameralistan i sidofältet:

1. Varje inkluderad, synlig och icke-pausad kamera tar bilder med sina valda exporttyper. **Arrayer utlöses som en synkroniserad utlösare** (en enda synkroniserad grupp över alla medlemmar — se [Multi-Camera Arrays](arrays.md)); fristående kameror tar bilder individuellt.
2. Dolda (ögon) eller pausade kameror hoppas över. En array blockeras endast helt när *alla* dess medlemmar är dolda eller pausade.
3. När en ljussensor tilldelas sparas motsvarande DAQ-värde för nedåtriktat ljus som en `.daq`-fil i inspelningarnas `daq/`-mapp – även för inspelningar som endast innehåller rådata – så att radiometriska produkter alltid kan härledas senare.
4. Knappen visar förloppet i realtid – ”Capturing… done/total” – och blir **Stop (N)** i kontinuerligt/intervalläge. Varje inspelningsobjekt har en timeout på 300 s.
5. När omgången är klar visar ett resultatmeddelande **”N filer sparade”**eller**”N sparade, F misslyckades”**, samt ”(S dold/pausad/hoppad över)” när kameror har hoppats över. Om hårddisken blir full avbryts körningen och ett meddelande visas:**”Avbruten: hårddisken full. Sparade N, F misslyckades”** – varje fil som inte kunde skrivas räknas som misslyckad.

## Var inspelningarna sparas

Inspelningarna sparas under det öppna projektet i mappen `<project>/captures/`. Varje exporttyp sparas i sin **egen undermapp**, så en inspelning med flera nivåer blandar aldrig ihop olika typer:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` är inspelningens tidsstämpel och `<serial>` är kamerans serienummer. Fristående bildtagningar namnges `capture_<ts>_SN<serial>_<level>`; bildserier från en synkroniserad utlösare namnges `sync_<ts>_SN<serial>_<level>` och **delar ett gemensamt tidsstämpel för alla kameror i gruppen** (nivåsuffixet utelämnas när en kamera endast sparar en enda nivå).
* **En avvikelse att känna till:** visningsnivån lagras i en mapp med namnet `preview/` medan filerna behåller `_display` i namnet — mapp och suffix skiljer sig åt endast för den nivån.
* Okända nivåer placeras i en mapp med sitt eget namn; om en undermapp inte kan skapas skrivs filen till rotkatalogen för inspelningarna istället för att gå förlorad.
* Inspelnings-TIFF-filer komprimeras som standard förlustfritt (DEFLATE) och innehåller alla metadata om kalibrering och bearbetning **inuti filens XMP** — bilderna är självbeskrivande och har inga sidfiler utöver filen `.daq` (och dess `.daq.cameras.json`).
* **Klämmasker (nytt i 1.3.0).**Där den spektrala 3×3-separeringen var tvungen att klämma fast ett band till 0 på minst en pixel, får produkterna för strålningsintensitet, reflektans och index en mask per pixel, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = bandet i produktens R-kanal, bit 1 = G, bit 2 = B); produktens `Chloros:UnmixClampMask` XMP-tagg pekar på den. Maskerna finns**bredvid** produktmapparna, aldrig inuti någon, så en produktmapp som `reflectance/` kan överlämnas till fotogrammetriprogramvaran som den är. Chloros egen import hoppar över maskerna.
* **Orientering (1.3.0).** En fristående kameras Preview-, Radiance-, Reflectance- och Index-produkter roteras enligt inställningen för spegling/rotation; Raw- och Debayered-filerna förblir som sensorn läste in dem. Alla filer i arrayen förblir i sitt ursprungliga format. Varje fil lagrar inställningen i sin XMP, så vid ombearbetning tillämpas den på samma sätt.
* **Vilken kamera en `.daq` tillhör (1.3.0).** En bilds `.daq` matchas, vid bearbetningstillfället, endast med bilder från de kameror som dess `.daq.cameras.json` namnger. En fristående kamera utan egen ljussensor lånar därför inte längre avläsningar från en annan kamera – koppla en sensor till den eller registrera en i fliken Ljussensorer. En ljussensorregistrering (och alla `.daq` utan den filen, inklusive äldre inspelningar) fungerar fortfarande för alla kameror.

Detta är samma struktur som `chloros-cli lattice capture` / `array-capture` skriver till sin `-o`-katalog — dokumenterad i [CLI-referens § Hur en inspelningsmapp ser ut](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Ombearbetning av bildserier till kalibrerade produkter

Inspelade råbilder plus den sparade filen `.daq` är allt som bearbetningspipeline behöver – det är därför Fastest Capture är säkert att använda för verkligt arbete.

* **GUI**: lägg till mappen med bilder i ett projekt ([Lägga till filer i ett projekt](../processing-images-gui/adding-files-to-a-project.md)) och bearbeta som vanligt.
* **CLI**: peka `process` mot **rotkatalogen för bilder**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` importerar normalt endast den mapp du anger, men när den mappen inte innehåller några bilder och har undermappar går programmet automatiskt ned i hierarkin — så att undermapparna på den nivån och mappen `daq/` hämtas i ett svep. Varje import sker som en **enskild bild** med de övriga nivåerna bifogade som visningslägen, inte som en bild per nivå.

Att namnge en undermapp på en nivå direkt (t.ex. `…/captures/raw/`) fungerar också, men lämnar mappen `daq/` kvar – kopiera den vid sidan av när du-härleder en radiometrisk produkt från `raw/`, annars har tidsstämpeln inget att jämföra mot.

{% hint style="warning" %}
**Bearbetningen börjar alltid från `raw`.** Inom varje inspelning är den obearbetade bilden källan till bearbetningskedjan; `debayered`, `radiance`, `reflectance`, och `preview` förekommer som visningslägen men matas aldrig tillbaka genom pipelinen – att återbearbeta en härledd produkt skulle innebära att vignettering, färg och strålningsberäkningar som redan är inbakade i dess pixlar tillämpas på nytt, så Chloros-algoritmen avstår hellre än att utföra dubbelbearbetning. Renderingarna `index/` och `composite/` bearbetas aldrig alls (de är utdata, inte inspelningar). En mapp med inspelningar som sparats **utan** råimport visas normalt, men `process` hoppar över den och meddelar detta; `--input-level {raw,debayered,processed}` är den avsiktliga nödutgången som tvingar fram en ingångspunkt. Se [CLI-referensen](../reference/cli-reference.md#what-a-captures-folder-looks-like) för de exakta meddelandena om hopp.
{% endhint %}

Ytterligare två beteenden som är värda att känna till vid skriptning av ombearbetning:

* En `chloros-cli process`-körning som begärde produkter men inte skrev ut **några bildprodukter misslyckas med ett tydligt felmeddelande och avslutas med ett värde som inte är noll** — du kommer aldrig att få en tyst, tom körning. Framgångsrika körningar rapporterar antalet produkter. (En avsiktlig körning som endast genererar metadata räknas fortfarande som en framgång.)
* Återimporterade bearbetade exportfiler tar aldrig en råfilplats från en inspelning — den ursprungliga råfilen förblir alltid källan i pipelinen.

## Motsvarigheter i CLI



Allt på denna sida kan köras utan grafiskt gränssnitt. GUI-inspelningslägena motsvarar direkt `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Enstaka | `chloros-cli lattice array-capture` |
| Kontinuerlig | `array-capture --continuous [--count N] [--duration S]` |
| Intervall | `array-capture --interval S [--duration S]` |
| Snabbaste inspelning | `array-capture --fastest` |
| Kryssruta för justering | `--aligned / --no-aligned` |
| Kryssrutor för exporttyp | `--processing LEVEL` eller `--levels L1,L2,…` (standard `all`) |
| Spela in indexvideo | `chloros-cli lattice array-record` |
| Spela in rå bildserie / Skapa video | `chloros-cli lattice array-burst` / `array-build-video` |

Fullständiga flaggtabeller, alternativet för smart-AE-stabiliserad inspelning (`--smart`) och modellen för kontinuerlig hastighet finns i [CLI-referens § Inspelningslägen, inspelningsenheter och offline-bearbetning](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
