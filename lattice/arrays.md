# Kamerasystem med flera kameror

Ett LATTICE **system**består av två eller flera LATTICE-kameror som är anslutna som en synkroniserad enhet. En kamera är**master**: den avger en GPIO-triggerpuls via en gemensam synkroniseringslinje (standard**Line2**), så att alla kameror exponerar samma ögonblick. Chloros lägger till PTP-tidssynkronisering-synkronisering, en liveförhandsvisning (rutor per kamera eller en enda justerad multibandskomposit) och synkroniserad bildtagning — varje bildtagningsomgång producerar en**bildgrupp** där alla kameror delar samma tidsstämpel och bild-ID (rapporteras som `fid:N` i bildtagningsutdata).

Det är genom arrayer som monokameror (M3M) producerar vegetationsindex – en kamera bidrar med ett band, och arrayen justerar dem till en multibandsstapel. Se [Monokameror och vegetationsindex](mono-indices.md).

Det finns tre likvärdiga sätt att ansluta en array, och alla kör samma ”smart-prep”-flöde:

| Yta | Ingångspunkt |
| --- | --- |
| GUI | Fliken Kameror → **Anslut matris** (blå knapp) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (första serienumret = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (första serienumret = master) |

Smart-prep utför, i följande ordning: en nätverkskapacitetskontroll (ICMP DF-ping + GVSP-kontroll), val av synkroniseringsnivå, automatisk anpassning av bildstorleken till bandbredden, aktivering av PTP, automatisk val av pixelformat per kamera, automatisk inställning av exponering utifrån varje kameras sparade tillstånd samt konfiguration av GPIO-utlösare på Line2.

{% hint style="info" %}
Kamerorna måste vara nåbara via länken innan något av detta fungerar — se [Ansluta kameror](connecting.md) för upptäckt, adressering och nedladdning av kalibrering vid första anslutningen. För riggar med flera kameror är värd-NIC:ets inställningar för mottagningsringen lika viktiga som länkhastigheten; den fullständiga tabellen över symptom→lösningar finns i [CLI Referens § Inställning och finjustering av värd-NIC](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialogrutan ”Anslut array”

Fliken Kameror → **Anslut array**öppnar en guide i tre steg:**Välj → Visningsläge → Inställningar**.

### Steg 1 — Välj master och slavar

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

Dialogrutan skannar nätverket så snart den öppnas (”Scanning network...”) och undersöker sedan GPIO-utlösarens kabeldragning (”Probing GPIO wiring...”). Du behöver minst **2 kameror** för att bygga en array.

Anslutningskontrollen fyller i rollvalet automatiskt när det är möjligt och visar ett av tre meddelanden:

| Meddelande | Betydelse |
| --- | --- |
| ”GPIO-master upptäckt — val förifyllda” (grön) | Testet har hittat utlösartopologin; kryssrutorna för master- och slavkameror är redan markerade. |
| ”Ingen master upptäckt – kontrollera GPIO-kabeln” (orange) | Ingen kamera har upptäckt en utlösarpuls; kontrollera synkroniseringskablarna. Du kan fortfarande välja roller manuellt. |
| ”Ingen synkroniseringskabel: {serienummer}” (orange) | De listade kamerorna har ingen synkroniseringskabel ansluten. |

Kameratabellen har kolumnerna **Kamera / Serienummer / IP / Master (radio) / Slav (kryssruta)**:

* Välj exakt **en master**och**en eller flera slavar**. Om du klickar på den aktuella masterns radio igen avmarkeras den.
* En kamera markerad med **”Ingen synkroniseringskabel”** kan aldrig väljas som slav – en slav utan utlösarkablage skulle vänta på synkroniseringslinjen för evigt och leverera en död bildström. Anslut istället den kameran som en fristående kamera.
* Kameror som redan är anslutna som fristående kameror inaktiveras *inte*: anslutning till arrayen avslutar den fristående sessionen och öppnar kameran igen inom arrayen.

**Nästa: Visningsläge →**aktiveras när en master och minst en slav har valts.**Skanna om** kör om upptäckten och anslutningskontrollen.

{% hint style="warning" %}
**Avbryt** är inaktiverat medan en skanning eller sondkörning pågår — att avbryta mitt i en sondkörning kan krascha kameran SDK på LATTICE-kamerans firmware. Vänta tills spinnaren har slutat snurra.
{% endhint %}

### Steg 2 – Visningsläge

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Läge | Vad du får |
| --- | --- |
| **Separata kameror** | En live-ruta per kamera, alla utlöses samtidigt så att bildrutorna förblir synkroniserade. Varje kamera behåller sin egen färg och sina egna inställningar. |
| **Kombinerade kameror** *(standard)* | En enda ruta som återger den justerade multibands-kompositen NDVI/index. Kamerorna delar matrisens färg. |

Visningsläget påverkar endast presentationen av liveförhandsvisningen — inspelningsbeteendet är detsamma i båda lägena.

### Steg 3 — Arrayinställningar och det förväntade resultatet

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

När du går in i denna scen ber Chloros backend-systemet om en **rekommendation**och tillämpar automatiskt en kombination av ROI och binning som passar ditt NIC:s mottagningsring (programmet föredrar binning framför ROI-beskärning, eftersom binning bevarar hela synfältet). Varje ändring du gör kör om analysen i realtid och uppdaterar panelen**Förväntat resultat** till höger.

Vänstra kolumnen — inställningar:

| Kontroll | Alternativ | Standard | Anmärkningar |
| --- | --- | --- | --- |
| **ROI (synfält)** | Fullt (2048×1536) / Halvt (1024×768) / Kvart (512×384) | Full | Sensorbeskärning: Halv/kvartsklippning till ett mindre område med ursprunglig pixelavstånd. |
| **Binning** | 1× / 2× (summa 2×2) / 4× (summa 4×4) | 1× | Hårdvarubinning: 2×2 = fullt synfält till en fjärdedel av ledningskostnaden; 4×4 = fullt synfält till 1/16. Döljs om kamerorna inte stöder binning. |
| **Bild på ledningssidan** (avläsning) | — | — | Bredden × höjden efter binning som faktiskt skickas via ledningen, avrundad till multiplar av 16 (minst 64). |
| **Pin-upplösning**| kryssruta | av | Chloros aktiverar normalt binning automatiskt vid anslutning när den beräknade hastigheten sjunker under**1,5 fps**. Att låsa inställningen behåller den valda bildstorleken och accepterar den lägre hastigheten – och omvandlar en överbelastad konfiguration till ett definitivt avvisande av anslutningen istället för en automatisk nedskalning. |
| **Triggerfrekvens** | 0,5–60 fps, steg 0,1 | tom = auto | Masterenhetens utlösningsfrekvens. Lämna fältet tomt för att låta Chloros beräkna den. |
| **Wire Budget**| 20–2000 MB/s, steg 10 | tom = auto | Hur mycket värden faktiskt kan hantera, i MB/s —**det enda talet som hela arraytilldelningen hänger på.** Detekteras automatiskt från nätverkskortet. Sänk värdet om arrayen rapporterar korrupta ramar: det upptäckta värdet överskattar USB-kort och delade switchar. Om du ändrar det körs prognosen om i realtid. |

Högra kolumnen — **Prognoserat resultat**:

* **Synkroniseringsnivå** — ”Samtidig inspelning” (grön), ”Samtidig inspelning (FTD-förskjuten sändning)” (grön), ”Förskjuten inspelning (100 ms avvikelse)” (gul) eller ”Konfigurationen är för stor” (röd).
* **fps-prognos** — visas som ett intervall (”svagt → starkt”), eftersom hastigheten för en synkroniserad matris begränsas av den långsammaste kamerans exponeringstid.
* **NIC-rad** — länkhastighet och kontinuerlig genomströmning (”NIC {mbps} Mbps · kontinuerlig {N} MB/s”).
* **Kontroll av simultan sändningsburst** — kan värdens NIC-ring ta emot en samtidig burst från alla kameror (&quot;Simultan sändningsburst: X MB · Användbar NIC-ring: Y MB ✓/✗&quot;).
* **Kontroll av ledningsbudget** — sammanlagd efterfrågan i stabilt tillstånd jämfört med det kollisionssäkra taket för ledningen (&quot;Ledningsbudget: {efterfrågan} MB/s efterfrågad av {n} kameror · tak {tak} MB/s ✓/✗ övertecknad&quot;).
* **”Max antal kameror på denna kabel: {n} — fastställs av bandbreddsminimumet per kamera, så binning höjer inte detta.”** — visas när du är nära (eller överstiger) taket för antalet kameror.
* **&quot;BILDER KOMMER ATT FÖRSVINNA med dessa inställningar.&quot;**— röd varning med backendens motivering, plus en lista över hinder och blå**förslag på lösningar** (&quot;För att få plats med denna matris i nätverket&quot; / &quot;För att möjliggöra samtidig inspelning&quot;).**Apply &amp; Connect** är inaktiverat tills en prognos finns, och dess etikett anger varför det nekas:

| Knappetikett | Betydelse | Vad som faktiskt hjälper |
| --- | --- | --- |
| &quot;Analyserar...&quot; | Analysen pågår fortfarande. | Vänta. |
| **”För många kameror för detta nätverk”**| Arrayen överbelastar nätverket (sammanlagd kontroll misslyckades). | Färre kameror, jumbo-ramar från ändpunkt till ändpunkt eller ett snabbare nätverkskort.**Ett mindre ROI hjälper INTE** — se nedan. |
| **&quot;Minska ROI för att aktivera&quot;** | Bilder skulle tappas bort med dessa inställningar (burst-/ringkontroll misslyckades). | Minska ROI, öka binning eller åtgärda nätverkskortets mottagningsring. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Under anslutningen kan en grön **kalibreringsnedladdningspanel** visas med en förloppsindikator per seriell anslutning: första gången en kamera ansluts till en maskin hämtar Chloros sitt fabrikskalibreringspaket på cirka 3,8 MB från kameran via GigE (cirka 70 sekunder per kamera). Cachelagrade kameror visar aldrig denna panel. Se [Ansluta kameror](connecting.md).

## Bandbredd: hur många kameror som ryms

Hur många kameror en array kan hantera beror på nätverkskabeln, inte på Chloros, så planeringssiffrorna finns i hårdvaruhandboken: **[Planering av arraybandbredd](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Så här hanterar Chloros dessa uppgifter: anslutningsdialogen kör en nätverkssond, beräknar den uppnåeliga bildfrekvensen och väljer en nivå som passar. Om arrayen överbelastar kabeln vägrar den att ansluta istället för att tyst släppa paket – se panelen för beräknat resultat som beskrivs ovan.

## När ramar saknas

En kamera kan saknas i en publicerad grupp av två helt olika skäl,
och de kräver motsatta åtgärder. Chloros räknar dem separat istället för att rapportera ett
”ofullständigt” tal som inte specificerar något av skälen:

| Vad som hände | Vad det betyder | Var man ska leta |
| --- | --- | --- |
| **Korrupt**— ramen anlände men var strukturellt felaktig | GVSP-paketförlust på nätverksvägen |**Kabelbudgeten**, nätverkskortets mottagningsring, jumbo-ramar, switchen |
| **Anlände aldrig**— ingen ram kom alls | Kameran utlöste inte, eller inget lämnade den |**M8-synkroniseringskabeln**, synkroniseringslinjen, om alla enheter är aktiverade |

Uppdelningen omvärderas var 10:e sekund medan arrayen strömmar. Över 5 % loggas det
med båda siffrorna angivna, och varje skadad buffert rapporteras första gången det
inträffar per kamera, därefter sammanfattas det en gång per minut så att en lång session förblir läsbar.

**Skadade ramar med noll ”kom aldrig fram” innebär att utlösning och kabelsynkronisering är perfekta**och varje förlorad bildram beror på nätverksvägen. Lösningen är att sänka**Wire Budget** och
ansluta om.

{% hint style="warning" %}
**Att sänka utlösningsfrekvensen hjälper inte mot korrupta bildramar.** Kamerans paket
taktsättning skrivs in en gång, vid anslutningen. Att sänka utlösningsfrekvensen ändrar hur ofta en burst
inträffar, inte hur snabbt själva bursten skickas ut på nätverket. På en uppmätt rigg med 4 kameror
förändrade en 5× sänkning av utlösningsfrekvensen ingenting, medan en sänkning av wire budget från 240 till
200 MB/s minskade andelen korrupta bildrutor för samma rigg från 10,4 % till noll.
{% endhint %}

En igångsatt array kan inte omplanera sig själv – koppla bort och anslut på nytt så att anslutningstidsväljaren
kan anpassas efter den nya bandbreddsbudgeten.

### USB-nätverkskort har en övre gräns på 200 MB/s

Ett USB-Ethernet-kort anger sin *Ethernet*-länkhastighet, men vad det faktiskt
kan upprätthålla begränsas av USB-bussen och dess drivrutin. En USB-dongel för 10 GbE brukade tillskrivas
en genomströmning på ungefär 1000 MB/s – ett värde som ingen någonsin hade mätt – och när fyra kameror
anpassades efter detta fiktiva utrymme förstördes 6–18 % av bildrutorna medan arrayen
fortfarande rapporterade en normal målbildfrekvens. USB-anslutna adaptrar är nu begränsade till
**200 MB/s**. Begränsningen är ett absolut värde snarare än en procentandel, eftersom gränsen ligger i
bussen: en USB 1 GbE-adapter uppnår cirka 80 MB/s och påverkas inte.

Om din värddator verkligen är snabbare än gränsen, höj **Wire Budget** för att ange detta.

## PTP-tidssynkronisering

Bildruts *synkronisering* sker via hårdvarutriggern; **PTP** (IEEE 1588 PTPv2) tillhandahåller jämförbara *tidsstämplar* på alla enheter. Det är aktiverat som standard vid anslutning av arrayen:

* **Chloros-värdbackend kör PTP-grandmaster**. LATTICE-kameror och DAQ-E-ljussensorer fungerar som slavar till denna i domän 0, så att bildtidstämplar och DAQ-spektra hamnar på samma klocka (~1 ms).
* `--no-ptp` (CLI) inaktiverar den för laboratoriearbete – då är tidsstämplarna mellan kamerorna **inte** jämförbara.
* Kontrollera synkroniseringens status med CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Fliken Kameror har i sig ingen PTP-indikator; där visas synkroniseringsinformation per kamera i form av de skrivskyddade uppgifterna **Roll**(Master/Slave),**Synkroniseringslinje** och matrisens kapacitetsnivå. DAQ-E:s PTP-status visas i sensordetaljerna på fliken Ljussensorer.

## Livevyn

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

för matrisen Huvudvisningsområdet erbjuder två layouter (växla i den övre fältet): **rutnätsvy**(varje ruta är en cell; dra för att ordna om när rutnätets hänglås är upplåst) och**listvy**(matriser i full bredd överst, en aktiv kamera nedan). Skjutreglaget**Feed Zoom** justerar storleken på rutorna; vid en cellbredd under 200 px döljs namn- och fps-överlagringarna automatiskt.**Separat läge** visar en ruta per kamera. Varje ruta visar följande överlagringar:

* kamerans namn (uppe till vänster),
* en **fps-visning** (uppe till höger) — detta är kamerans *verkliga bildhämtningshastighet* som rapporteras av backend, inte förhandsgranskningsfrekvensen (liveförhandsgranskningen är begränsad till 30 fps oavsett bildhämtningshastighet),
* en statuspunkt — grön (strömning) / gul (laddar) / röd (fel),
* en **spinner för föråldrad bildruta** när ingen ny bildruta har anlänt på 2 sekunder — normalt i ~5 sekunder efter varje anslutning/frånkoppling medan backend-systemet omfördelar bandbreddsbudgeten mellan kamerorna.**Kombinerat läge**visar en enda sammansatt ruta: backend-systemet utför debayering, skalning, justering, brusreducering, konvertering till strålning per band (plus DLS-reflektans när en ljussensor är kopplad), utvärderar matrisens indexuttryck, tillämpar LUT och strömmar resultatet som MJPEG. Tills den första justerade bilden renderas visar rutan dess status: ”Förbereder matris…”, ”Kalibrerar justering…”, ”Väntar på första bilden…” eller – om tidsgränsen för automatiska justeringsförsök (~30 s) har överskridits – ”Justering krävs” med en knapp för**Kalibrera justering**.

Användbara fakta om kombinerat läge:

* Kompositbilden är registrerad mot **huvud**kamerans bildruta. AE-ROI-fokusering och punktmätning på kompositbilden är exakta för huvudkameran och ungefärliga för slavkamerorna; använd**Split View** (matrisinställningar → ”Visa medlemskameror”) för pixelexakta rutor per kamera utan att öppna extra kameraanslutningar.
* **Display Layers**(array-inställningar; avstängt som standard) låter dig välja ett förgrunds- och bakgrundsskikt — valfri medlemskamera eller**Index**. Med förgrund = Index visar pixlar utanför LUT:s min/max-värden bakgrundsskiktet.
* **Renderingsupplösning** (standard 720p) ställer in höjden på livestreamen *och* storleken på den sparade kompositbilden vid export. Bilder per kamera exporteras alltid i full upplösning.
* Justeringen beräknas per session och sparas aldrig – se avsnittet om justering i fönstret för arrayinställningar för RMS-avvikelser och knappen *Rekalibrera*.

## Inspelning: övervakning kontra analys

Arrayens inspelningsytor delas tydligt upp i **övervakningsklass**(spela in det du ser) och**analysklass** (spela in rådata, kalibrera senare):

| Arbetsflöde | Kvalitet | Vad som sparas | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Inspelning**(stillbilder) | Analys | En synkroniserad bildgrupp per pass; filer per kamera på varje vald exportnivå (rådata/avbayerad/strålning/reflektans/förhandsgranskning/index) + `.daq` sidokar |****Spela in allt**-knapp + Inspelningsinställningar | `lattice array-capture` |
| **Spela in indexvideo** | Övervakning | Den kombinerade live-indexkompositen som den visas — 8-bitars, förhandsgranskningsupplösning, inbakad LUT; kräver att livestreamen är öppen | ● Spela in indexvideo (kombinerade matriser) | `lattice array-record` |
| **Rå bildserie → skapa video**| Analys | Råa sensorbilder med full inspelningshastighet + manifest + `.daq`, därefter offline-rekonstruktion till kalibrerad strålnings-/reflektans-/indexvideo, tidsanpassad till DAQ-avläsningar | ⦿ Spela in rå bildserie →**Skapa video** | `lattice array-burst` → `lattice array-build-video` |

Tumregel: om pixlarna ska mata in *mätningar*, använd *Capture* eller *Burst* (analysklass); om du bara behöver *titta på eller visa* vad sensormatrisen såg, spela in indexvideon (övervakningsklass).

### Inställningar för inspelning (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

Kugghjulet bredvid **Spela in allt** öppnar fönstret Inställningar för inspelning (kräver ett öppet projekt – inspelningarna sparas i det):

* **Inspelningsläge**:**Enstaka**(ett genomgång) /**Kontinuerlig**(i följd; begränsat av ett antal inspelningar, standard 1, eller en varaktighet, standard 10 s) /**Intervall** (tidsförlopp: N inspelningar var X:e intervall för totalt Y, standardinställning 1 var 5:e sekund i 1 minut).
* **Exporttyper per kamera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — alla tillämpliga alternativ är aktiverade som standard. Radiance/Reflectance är dolda för kameror med RGB-filter;**Reflectance visas endast när kameran har en DAQ-ljussensor** (sin egen eller ärvd från matrisen); Index kräver ett konfigurerat indexuttryck.
* **Justerad**(per matris, standard**på**): anpassar exporten av enskilda bilder till matrisens justeringsprofil så att exporten blir pixelregistrerad. Raw förblir alltid oförändrad men bär med sig transformationen i metadata.
* **Snabbaste inspelning** (växla): endast rådata + den tilldelade DAQ-avläsningen + den fria sammansatta indexkompositionen, vilket hoppar över kalibreringsberäkningarna vid inspelningstidpunkten för maximal hastighet — återuppbygg strålning/reflektans/index senare från den sparade `.daq`.
* Valen bevaras med projektet. Dolda eller pausade kameror hoppas över.

Motsvarande CLI (samma backend-ändpunkt, samma semantik):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

TIFF-komprimering för inspelningar är `deflate` (förlustfri, standard) eller `none` — fullständiga flaggtabeller, mappen för inspelningar och-bearbetningsregler finns i [CLI-referensen](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Koppla ihop en DAQ-ljussensor

Förhandsgranskningar med reflekans- och belysningskorrigering kräver data om nedåtriktat ljus från en DAQ-sensor (ansluten under fliken **Ljussensorer**):

* I **matrisraden**i sidofältet visas en**&quot;DAQ · på/av&quot;-knapp** — *på* när en ljussensor på matrisnivå är inställd **eller** när någon av kamerorna i matrisen har en egen; i verktygstipset anges exakt vilken sensor som matar vilken kamera.
* Tilldela för hela arrayen i arrayinställningarna → **Omgivande ljussensor**→ rullgardinsmenyn**Ljussensor**. Valet sparas med projektet, sprids till varje enskild kamera i arrayen, och enskilda kameror kan fortfarande åsidosätta det med sin egen sensor.
* Statusraden nedanför visar det aktuella läget: **Av**→ ”Väntar på första spektrumet…” →**”Aktiv — alla kameror i arrayen är belysningskorrigerade”** → eller, om inget nytt spektrum har kommit in under de senaste 3 sekunderna, ett meddelande om inaktuella data — den senaste avläsningen fortsätter att användas (avläsningar förfaller aldrig i inspelningskedjan).

När en sensor har tilldelats: exporttypen Reflektans blir tillgänglig, liveförhandsvisningar är belysningskorrigerade, den prediktiva automatiska exponeringen kan använda spektrumet, och varje reflektansinspelning skriver den DAQ-avläsning som faktiskt användes som en **`.daq` sidecar** bredvid bilden så att inspelningen kan bearbetas på nytt senare.

## `array-connect` CLI-alternativ

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatisk upptäckt av alla LATTICE-kameror (kräver ≥2) | **Den första seriella anslutningen är MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkroniseringslinje. |
| `--target-fps F` | auto | Master-triggers avfyrningsfrekvens. |
| `--binning {1,2,4}` | auto | Hårdvarubinning. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Expertöverskrivning av synkroniseringsnivåväljaren. |
| `--wire-ceiling-mbps MB_PER_S` | automatisk detektering | Värdens trådbudget i MB/s — CLI-formen av fältet **Trådbudget**. Sänk den om arrayen rapporterar korrupta ramar. Sparas med projektet, så att en senare återanslutning återställer den. |
| `--no-recommend` | av | Hoppa över nätverksanalyssteget. |
| `--no-ptp` | av | Inaktivera PTP (tidsstämplar mellan kameror kan då inte jämföras). |

`lattice array-list`, `array-status` och `array-disconnect` hanterar den permanenta sessionen. Den fullständiga referensen för underkommandon, inklusive justering (`align-calibrate` / `align-apply`) och nätverksverktygen, finns i [CLI-referens § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); motsvarigheterna till SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) finns i [SDK-referensen](../reference/sdk-reference.md). Från Python är ledningsbudgeten `connect_array(..., wire_ceiling_mbps=120)`, och uppdelningen mellan aktiva, korrupta och aldrig ankomna finns i [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
