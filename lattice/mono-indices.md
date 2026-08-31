# Monokromkameror och vegetationsindex

## En kamera = ett band

En **M3M**-kamera är den monokroma motsvarigheten till Bayer**M3C**: en monokrom IMX265-sensor bakom ett enda smalbandigt interferensfilter. Modellsträngen anger bandet – `M3M-<lens>-F<wavelength>`, t.ex. `M3M-L87-F685` (visas i Chloros som `LATT-M3M-L87-F685`). Sensorn levererar ett**enda gråskaleband** utan Bayer-mosaik: det finns inget att demosaikera, ingen kanalöverskridande överhörning att separera och ingen vitbalans att ställa in.

Konsekvenser som är värda att känna till innan du planerar ett monosystem:

* **Strålning och reflektans är fullständigt definierade per band.**Det är radiometriska kartor per band, så en M3M-kamera producerar kalibrerad radianse i float32 (W/m²/sr/nm) och reflektans i uint16 (`32768` = ρ 1,0) precis som ett M3C-band gör. Monobilder har en**identitets**sensorresponsmatris – ingen 3×3-separering behövs eller tillämpas.
* **En enskild monokamera kan inte generera ett vegetationsindex.** NDVI, NDRE och liknande behöver minst två band. För att beräkna index från monohårdvara kombinerar man flera M3M-kameror – se nedan.
* M3M-kameror strömmar **Mono12** (12-bitars, 2 byte/pixel över kabeln), vilket är viktigt för [budgetering av arraybandbredd](arrays.md#bandwidth-the-rules-of-thumb).

## Vad Chloros hoppar över för mono – och hur det meddelar dig

Stegen i färgpipeline gäller helt enkelt inte för en sensor med ett enda band. Chloros **hoppar över dem med ett enradigt meddelande** istället för att rapportera ett fel, och kör dem fortfarande normalt för alla M3C-kameror (Bayer) i samma session:

| Steg | Beteende för mono (M3M) | Beteende för M3C |
| --- | --- | --- |
| Demosaic / debayer | Hoppas över — exportnivån för `debayered` är en 1-kanals gråskalebild. | 3-kanals demosaic. |
| Vitbalans (`lattice white-balance`) | Hoppas över med ett enradigt meddelande. | Körs normalt. |
| Färgprofil (`lattice color-profile`) | Hoppas över med ett enradigt meddelande. | Körs normalt. |
| Mättnad/kontrast (`lattice color`) | Hoppas över med ett meddelande på en rad. | Körs normalt. |
| Spektral crosstalk-avskiljning | Identitet (ingen 3×3-matris). | 3×3-matris tillämpad per kamera. |
| Strålningsintensitet/reflektans | **Körs** — per band, fullt kalibrerad. | Körs per band. |

GUI:n tillämpar samma gating: för en monokamera döljer inställningspanelen per kamera de rader som endast gäller RGB (vitbalans, gamma, färgprofil, mättnad, kontrast, kanaluppdelning), och livehistogrammet är låst till en enda **MONO**-kurva. Diskriminatorn i hela stacken är `M3M`-tokenet i modellsträngen, som visas i GUI/SDK som `is_mono`.

## Index kräver ≥ 2 band: justera → stapla → indexera

Arbetsflödet för monoindexering består alltid av samma tre steg:

1. **Justera** — rikta flera M3M-kameror mot olika våglängder (t.ex. en F650 ”Red” och en F850 &quot;NIR&quot;), koppla ihop dem som en [multikamera-array](arrays.md) och låt Chloros beräkna korrigeringsförskjutningen mellan kamerorna.
2. **Stack** — de inriktade bildrutorna blir en multibandsbild (varje kamera bidrar med ett namngivet band).
3. **Index** — utvärdera en indexformel över stackens band och rendera den valfritt genom en LUT.

I GUI:n utgör hela denna kedja visningsläget **Combined Cameras**: den sammansatta livebilden är redan justerad, och arrayens Index Calculator (nedan) definierar den formel som den renderar. Inspelade exporter kan justeras till samma inriktning med inspelningsalternativet**Aligned**.

## Indexkalkylatorn

Indexkalkylatorn skapar det indexuttryck som används av livevisningen och indexexporterna per kamera. Det är en gemensam yta som öppnas från två ställen i sidofältet på fliken **Kameror**:

* **Per kamera**— Liveförhandsgranskning →**Index**-kugghjulet (endast RGN/OCN/NGB Bayer-kameror; en enskild monokamera har ingen indexkontroll eftersom ett band inte kan bilda ett index).
* **Per array**— arrayinställningar → Liveförhandsvisning →**Index**-kugghjulet. Detta är monovägen: bandlistan omfattar**alla kameror i gruppen**, så ett monopar bidrar med sina två band här.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Kontrollerna, uppifrån och ner:

* **Bandchips** (”Bands — klicka för att lägga till i uttrycket”) — en knapp per tillgängligt band, märkt med färgnamn + våglängd i nm (duplicerade färgnamn skiljs åt, t.ex. ”Color 850”). Genom att klicka infogas bandtoken vid markören. Band från kameror som inte kan producera strålning per band (RGB/FRGB) filtreras bort.
* **Operator- och funktionsknappar** — `+ - * / ( ) ^ ,` samt `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Textfält för uttryck** — fritt inmatad formel; platshållaren visar den klassiska NDVI-formen `(NIR - Red) / (NIR + Red)`. En skrivskyddad, tokeniserad förhandsgranskning ovanför visar bandchips, siffror och flaggor som okända token.
* **Giltighetsbanner**— grått ”Tomt — inget index kommer att tillämpas”; grönt ”Giltigt uttryck”; rött med det specifika tolkningsfelet (okänt band, tvetydigt band som exponeras av flera kameror, saknad parentes, …); eller gult när uttrycket är giltigt men**konstant** (t.ex. `X/X`, eller en NDVI-nämnare som skrivits med `−` istället för `+`) — en konstant mappar hela bilden till en enda färg.
* En separat gul varning visas om det tillämpade uttrycket är korrekt men **live-bilden är enhetlig** (platt eller mättad scen) — histogramkollapsen upptäcks automatiskt.
* **Apply LUT**(standard: på; av = gråskalesträckning),**Level**2/3/5/7-stop (standard 7-stop) och**Min / Max**-inmatningar på båda sidor om gradientfältet. Min är som standard inställt på**0,2**— det zoomar in färgskalan till det vegetationsrelevanta området medan värden under detta passerar som gråskala; ställ in Min på −1 för hela indexområdet (knappen**Återställ** återställer −1…+1). Max är som standard inställt på 1.
* **Live-histogram** över indexfördelningen — kvadratrotskalade staplar, gulbruna p2/p98-percentillinjer, en vit medianlinje och avläsningar för värden utanför intervallet (&quot;◀ N% &lt; lo&quot; / &quot;hi &lt; N% ▶&quot;) som blir bärnstensfärgade över 1 % som en signal om att bredda Min/Max-fönstret.
* **Apply**tillämpar uttrycket på liveströmmen; LUT-justeringar tillämpas direkt utan att man behöver trycka på Apply. Uttryck är avsiktligt**endast giltiga för den aktuella sessionen** — de sparas inte mellan sessioner.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## CLI-kedjan

Samma kedja av justering → stapel → index, skriptbar från början till slut:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` mappar en förinställnings symboler till stapelns bandnamn. Två regler hjälper dig att undvika misslyckade körningar:

* **Symboler är skiftlägeskänsliga** och måste stämma exakt med förinställningens kanalnamn — förinställningarna använder gemener (NDVI:s är `red`,`nir`; kontrollera `--list-presets`). `--channel red=Red_660` fungerar; `--channel RED=660` misslyckas med ett `channel_map missing entries`-fel.
* Bandsidan måste ange ett band i den justerade stapeln (`lattice align-info --profile align.json` listar dem). Offline-läget accepterar även 0-baserade bandindex, t.ex. `--channel red=0 --channel nir=1`.

`lattice index` körs också helt offline mot en sparad, justerad multibands-TIFF:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Indexförinställningar

`lattice index --preset` (och fliken Bilds [Index/LUT-sandlåda](../image-viewer-gui/index-lut-sandbox.md), som använder samma motor) levereras med dessa **22 förinställningar**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Kör `chloros-cli lattice index --list-presets` för varje förinställnings formel och kanalsymboler, och `--list-gradients` för de tillgängliga färggradienterna. Anpassade formler använder `--formula EXPR` med samma syntax som Index Calculator. Observera att denna lista med förinställningar är specifik för LATTICE-indexmotorn – rullgardinsmenyn för bearbetning i projektinställningarna för importerade bilder innehåller en annan lista (se [Multispektrala indexformler](../project-settings/multispectral-index-formulas.md)).

Den fullständiga uppsättningen flaggor (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, justeringsreglagen för `--live`, och mer) dokumenteras i [CLI-referensen § Index / Vegetationsmatematik](../reference/cli-reference.md#index--vegetation-maths); motsvarigheter till SDK finns i [SDK-referensen](../reference/sdk-reference.md).

## Hämta indexprodukter från en mono-array

När en array är ansluten och ett indexuttryck tillämpas sparar `array-capture` (eller GUI-funktionen **Capture All**) exportnivåerna per kamera *och* indexrenderingen — `--index`/`--no-index` växlar mellan detta på CLI, och standardinställningen för insamlingen är att inkludera alla tillämpliga nivåer. En monokameras bidrag till varje inspelningsgrupp är dess enda band på rå-/debayered- (gråskala-)/radiance-/reflektansnivåer, plus den delade sammansatta indexkompositionen när matrisen körs i kombinerat läge. Se [Flerkameramatriser § Insamling](arrays.md#capturing-monitoring-vs-analysis).
