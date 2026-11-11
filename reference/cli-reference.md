# Chloros-CLI-referens

**Version:**

1.3.0**Skapad:**2026-07-29 19:19 ·**Uppdaterad:** 2026-11-06**Målgrupp:** Optimerad för användning av stora språkmodeller (LLM); läsbar för människor.**Omfattning:** Alla användarvända underkommandon för `chloros-cli`, med alternativ och exempel som kan kopieras och klistras in.

Detta dokument är den fullständiga referensen för kommandoradsverktyget `chloros-cli` som medföljer MAPIR Chloros. Det är avsiktligt uttömmande så att en LLM (eller människa) kan sammanställa vilket stödt arbetsflöde som helst utifrån listorna nedan utan att behöva granska källkoden.

Om du bara behöver det viktigaste kan du hoppa till:
- [Snabbstart på fem minuter](#five-minute-quickstart)
- [Arbetsflöde för första anslutning av LATTICE-kamera](#lattice-camera-first-connect-workflow)
- [Arbetsflöde för första anslutning av DAQ-sensor](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Inspelningslägen, inspelningsenheter och offline-bearbetning](#capture-modes-recorders--offline-reprocess)

> **Använder du en AI-assistent för att styra Chloros?** Claude Desktop, Claude Code, Cursor och andra MCP-klienter kan köra Processing, LATTICE-kameror och -matriser samt DAQ-ljussensorer via `chloros-mcp`, MCP-servern som installeras tillsammans med Python-SDK:et (och med Linux-.deb-paketet, i `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Konfiguration, alternativ och verktyg finns i SDK-referensens [MCP-server (AI-assistenter)](sdk-reference.md#mcp-server-ai-assistants). En assistent med ett skal kan också köra alla kommandon på denna sida direkt.

> **Ändrat i 1.3.0** (vad ett befintligt 1.2.0-skript eller arbetsflöde märker).
>
> **Ändringar som ett 1.2.0-skript kan behöva hantera:**
>
> - **`--preset` gäller.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` och `align-stream` togs bort i version 1.2.0 och kördes med kamerans egna inställningar. Nu ställer den in förinställningens format, exponering, förstärkning och bildfrekvens: `--preset high_speed` spelar in 8-bitars vid 5 ms, 6 dB, 60 fps. `lattice cam-connect --preset` behåller också förinställningens format (i version 1.2.0 ställdes kameran in på 12-bit efter att förinställningen tillämpats). Se [Vanliga alternativ](#common-options-shared-by-most-subcommands).
> - **Ett Bayer-pixelformat på en M3M-kamera (mono) är dess djup.** Förinställningarna namnger Bayer-format, vilket en monosensor inte kan hantera: i 1.2.0 avvisade kameran formatet och behöll det den hade, så `--preset high_speed` var inte 8-bit där. Det är nu Mono-formatet med samma djup: `high_speed` och `--pixel-format BayerRG8` ger `Mono8`, `high_quality`, `triggered` och `BayerRG12` ger `Mono12`. I en blandad M3C + M3M-matris använder varje kamera det gemensamma formatet inom sin egen familj, och `network-analysis` / `analyze-array --pixel-format` projicerar varje kamera på det djupet. Detsamma gäller tvärtom: en M3C som efterfrågar ett Mono-format antar Bayer-formatet för det djupet, så en matris vars master är en M3M öppnar inte längre sina M3C-kameror i Mono. Se [Mono (M3M) enkelbandskameror](#mono-m3m-single-band-cameras).
> - **`lattice array-capture`-exporter är som standard justerade, precis som appens.** Varje medlems `debayered`, `display`, `radiance`, `reflectance` och indexfilerna förvrängs till huvudkamerans bildram och beskäras till medlemmarnas överlappning (som är mindre än sensorn med kamerornas förskjutningar, `XMP-mapir:AlignmentJSON` `"applied": true`), så en CLI-inspelning består av samma filer som Chloros-appens inspelning av samma scen; `raw` förblir i sin ursprungliga storlek. Tidigare sparades vid en inspelning utan `--aligned` varje fil i sin ursprungliga storlek, utan förvrängning. `--no-aligned` behåller detta. Samma standard gäller för `--smart`, `--continuous` och `--interval`, samt för SDK och `chloros-mcp`. Se [`array-capture`-lägen](#array-capture-modes).
> - **`lattice array-capture --compression` har som standardinställning `none`.** Filerna skrivs okomprimerade (cirka 6,3 MB per bildruta i full upplösning) för den snabbaste kontinuerliga överföringshastigheten; version 1.2.0 skrev `deflate` (cirka 4,1 MB). Ange `--compression deflate` för mindre filer. Båda är förlustfria.
> - **PNG och JPEG accepterar endast `display` / `preview` och indexprodukter.** En `--format png`- eller `jpg`-inspelning med en nivå som en 8-bitarsfil inte kan hantera avvisas innan någon inspelning påbörjas (HTTP-kod 400, avsluta `1`), liksom standardinställningen `--processing all` med `--format png`/`jpg`. Se [Exportnivåer för insamling](#capture-export-levels-the-all-default).
> - **`lattice dls` kräver `--mac`.**Det finns ingen standard-MAC längre (det var en enhets adress); `daq pool-discover --only ble` hittar en sensors.**`lattice dls --camera SERIAL`** är nytt.
> - **Fler fel avslutar `1`.** `process` när en körnings export inte kunde verifieras eller om den förlorade en bild, även om de andra produkterna skrivs; ett inspelningskommando med någon fil som den inte kunde skriva (de övriga filerna sparas ändå); `lattice continuous` och `multi-stream` vid en full disk; `array-burst` / `array-record` när inspelaren rapporterar ett fel; `array-build-video` när en begärd produkt hoppades över eller inte skapades enligt begäran; `project burst` / `stream` när en inspelning misslyckas halvvägs; `project sensor log` när inspelningen inte innehåller någon avläsning; `lattice exposure --set`, `gain --set` och `configure --set` med ett värde som kameran avvisar. `lattice network-analysis --json` avslutas på samma sätt som sammanfattningen (`1` för `needs_force_slip`). Omvänt: `daq pool-disconnect` för en sensor som inte längre ingår i poolen avslutas med `0`. Se [Avslutningskoder](#exit-codes).
> - **En långsam `lattice array-connect` misslyckas aldrig när arrayen är ansluten.** Den väntar i upp till 600 s (180 s i 1.2.0; `project connect` 90 s) och skriver ut backendens steg under tiden, och en anslutning som inte är klar vid den tidpunkten ångras av backenden — varje kamera återställs till det läge den befann sig i — så ett `Error:` (avslut `1`) innebär att ingen matris är ansluten. I 1.2.0 kunde en anslutning direkt efter att kamerorna startats överstiga väntetiden och ansluta matrisen efter att kommandot hade misslyckats. Ctrl+C avbryter nu anslutningen. Se [Persistenta anslutningar](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` behåller hela sensorn.** Utan `--width` / `--height` / `--binning` är varje kamera inställd på hela sensorn och bildfrekvensen sjunker till vad den här datorn och nätverket klarar av; anslutningsutmatningen listar snabbare alternativ (binning eller ett mindre ROI) med deras beräknade hastigheter. I version 1.2.0 krympte eller binnade man bildrutorna för att hålla uppe hastigheten. Se [`array-connect`-alternativ](#array-connect-options).
> - **Ett långsamt nätverk avvisar inte längre en array.** Varje kamera anpassas efter sin andel av bandbredden och bildfrekvensen sjunker i takt med detta. Om kamerorna inte kan anpassas så långsamt körs arrayen i seriell utsändning (`slip-emit-and-capture`): varje bildram anländer, men kamerorna exponeras inte samtidigt. `--force-tier sim-capture-sim-emit` kräver samtidig exponering. Se [Array fps & burst-modell](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` visar vad connect tillämpar**: utan `--pixel-format`, de format som connect ställer in (Mono12 / BayerRG12, tidigare BayerRG8) och beslutet om [packad 12-bitars](#packed-12-bit-transport---packed). Samma kommando kan därför ange en annan bildfrekvens än i 1.2.0.
> - **Relativa sökvägar hänvisar till den plats där du kör kommandot.** `-o` och de andra sökvägarna som backend öppnar (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), ett receptstegs `output`, samt standardvärdet `output/` för `lattice index --live` löses upp mot den mapp som du kör `chloros-cli` från. I version 1.2.0 löste backend-modulen dem i sin egen installationsmapp, vilket vid en installerad version resulterade i felmeddelandet ”Åtkomst nekad”. Se [Konventioner](#conventions).
> - **`lattice index` körs på backend i alla former**, precis som de andra `lattice`-kommandona, så starta backend först. Offlineindexet (`--input`) beräknas av backend, som läser TIFF-filen och skriver ut resultaten bredvid den eller vid `-o`, och `--list-presets` / `--list-gradients` skriver ut backendens tabeller. I Windows fungerar dessa nu: tidigare svarade Windows-CLI på alla former av `lattice index`, inklusive `--live`, med `Error: lattice_sdk not available`. På Linux brukade offlineindexet köras inuti CLI utan backend och utan inloggning; nu krävs båda, precis som för alla andra CLI-kommandon. Se [Index / Vegetationsberäkningar](#index--vegetation-maths).
> - **Nya filer bredvid produkterna.** Varje radiance-, reflektans- och indexprodukt där bandavskiljningen har klämt en pixel får en mask i en `clamp_masks/`-mapp bredvid produktens mapp, och utskrifterna innehåller en rad med `clamp mask:`. Se [Unmix-klämmask](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Kamerans orientering.** En inspelning med en enda kamera sparar `raw` och `debayered` så som sensorn läste in dem och registrerar kamerans vändning/rotation i XMP; `process` tillämpar detta på de produkter den skapar. Arraymedlemsfilerna förblir i sitt ursprungliga format såvida du inte skickar den nya `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Se [Kamerans orientering](#hur-en-bildmapp-ser-ut).
> - **Ett exportnivånamn som inte är ett av dessa avvisas.** `lattice capture --processing` och alla `--levels`-listor tar `raw`, `debayered`, `radiance`, `reflectance` eller `display` (eller ett alias som `preview`; `--processing` samt `all`). Alla andra namn avvisas innan kameran berörs (HTTP 400 `UNKNOWN_LEVEL`, avsluta `1`); 1.2.0 sparade `raw` för den och avslutades `0`.
> - **`daq pool-set-cap` behöver `--cap-id`.** Utan det avslutas kommandot `2`; 1.2.0 tog bort lockkorrigeringen, vilket lämnade sensorn oskyddad. `--cap-id none` begär den nakna sensorn.
> - **`lattice resolution --binning-mode` finns inte längre.** Binning är alltid Genomsnitt; flaggan accepteras fortfarande, visar en varning och gör ingenting. Se [Sensorkalibrering](#sensor-tuning).
> - **`process --debayer texture-aware` avvisas för LATTICE-inspelningar.** För en mapp som innehåller LATTICE-bilder avslutar `process` `1` efter importen och anger hur många det finns, innan någon bearbetning sker. LATTICE-bilder får alltid standarddemosaik, så version 1.2.0 accepterade alternativet men ignorerade det; appen Chloros erbjuder det inte heller för dessa bilder. Se [`chloros-cli process`](#chloros-cli-process).
> - **Kamerorna återställs till det läge de befann sig i — såvida inte `--keep-settings`.** `lattice array-disconnect`, varje tillfälligt kommando (`multi-capture` utan array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) samt `lattice continuous` återställer varje kamera till det tillstånd den befann sig i: dess utlösare och synkroniseringsledningar, dess PTP samt alla kamerainställningar som ändrades under sessionen (exponering och förstärkning, AE-mål, gränsvärden och dämpning, vitbalans, svartnivå, pixelformat, binning och ROI, speglingar, bildfrekvens). `--keep-settings` sparar istället varje kamerasinställningar i kamerans eget minne, så att den startar med dem; `--clear-settings` återställer det minnet till fabriksinställningarna. I båda fallen lämnas kamerorna i fritt läge. Ett projekt ställer in sina kameror utifrån sina egna sparade inställningar när det ansluter dem. Se [Kameraminne](#camera-memory----keep-settings----clear-settings).
> - **Alla lägen sparar ett projekt på samma sätt.** Varje ändring av en kamera, en matris eller en ljussensor som är ansluten till ett projekt – från appen, CLI, SDK eller MCP-servern – sparas i projektets `cameras.json` / `sensors.json`, och när projektet öppnas i vilket läge som helst ställs enheterna in utifrån det. Tidigare sparade endast appen projektet, och nästa sparning skrev över allt annat. Se [En projektpost, alla lägen](#one-project-record-every-mode).
> - **Filer som skapats från flera kameror namnger var och en av dem.** En multiband-stack (`align-apply`, `align-stream`, `index --live --save-multiband`), ett liveindex skapat från flera kamerors band samt en bild från en array-inspelning (`composite/`) innehåller varje källkameras serienummer, modell och kalibreringspaket som listor (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), och EXIF-listorna `SerialNumber` innehåller serienumren, separerade med kommatecken. Tidigare innehöll de antingen ingen kamera alls eller endast den första kamerans serienummer. Se [Filer skapade från flera kameror](#files-made-from-several-cameras).
> - **Endast en `raw`-fil innehåller kamerans kalibreringstaggar.** Den MicaSense-kompatibla uppsättningen (`BlackLevel`, `RadiometricCalibration`, den kanalvisa `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`-uppsättningarna) samt nyttolasten `CalibrationJSON` beskriver sensorns råvärden, så de finns endast i inspelningsfilen `raw`. Alla övriga filer – en inspelnings `debayered`-, `display`-, `radiance`-, `reflectance` och indexfiler, `lattice continuous` och `multi-stream`-ramar, samt varje `process`-produkt – innehåller kamerans identitet (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) och, vid en bildtagning, bildens egen post (`XMP-mapir:FrameJSON`: exponering, förstärkning, temperatur, pixelformat, tidpunkt för bildtagning). I version 1.2.0 innehöll de även rådatauppsättningen, som ett verktyg som tillämpar den (Pix4D, Metashape) tillämpade på pixlar som inte är råvärden. `process` hittar en sådan fils kalibrering utifrån dess serienummer och `CalBundleSha256`.
>
> **Övriga ändringar:**
>
> - **En matris automatiska bildfrekvens anpassas efter vad matrisen levererar.** En frekvens som förlorar mer än 1 % av sina utlösningar sänks, vilket bedöms utifrån flera långa mätningar snarare än en, och en frekvens som den sänkts från prövas på nytt då och då. En frekvens som du själv ställer in lämnas oförändrad.
> - **Den automatiska exponeringen är stabilare.** Den håller sig konstant vid jämnt ljus, stabiliseras inom några sekunder efter en förändring, ignorerar små högdagrar och återställer en kamera som slutar svara i starkt solsken. Se [Hur automatisk exponering fungerar](#how-auto-exposure-behaves).
> - **`process` räknar endast dina bilder och inspelningar.** Om du kör om på en mapp som innehåller ett projekts `camera_cal/` räknas inte längre dessa kalibreringsfiler med (”Hittade 96 bilder” på en uppsättning med 90 bilder), och ett projekt som redan har tagit in dem tar bort dem.
> - **`lattice multi-test` utgår från kamerornas tillstånd vid uppstart.** Det aktiverar kamerornas PTP för körningen, väntar på att deras klockor ska låsas och stabiliseras innan den första utlösningen — varje kamera låses i 120 s, cirka 2 minuter från uppstart (`--ptp-settle SECONDS` ställer in det), eftersom en klocka som just låst sig avviker med upp till ~200 us under cirka 2 minuter — och återställer varje kameras PTP till det läge den fann den i. En kamera som fortfarande är låst från ett tidigare test väntas inte på igen. Tidigare lämnade den PTP avstängt i ett nätverk med en enda switch där ingen DAQ-E svarade, och läste varje kameras fristående klocka som en PTP-klocka som inte skulle låsa sig (`PTP is not locked`, vid varje körning). Se [Flera kameror (övergående sessioner)](#multi-camera-transient-sessions).
> - **En `.daq`-logg anger om locket valdes eller antogs.** DAQ-U, DAQ-M och DAQ-E levereras utan lock, och Chloros kan inte se om ett sådant är monterat, så en sensor som ansluts utan namngivet lock får `sunshine_cosine` som standard, ett *antagande*. Varje Chloros-inspelning (schema 1.30) lagrar nu `cap_id_source` bredvid `cap_id`: `operator` om du har namngett ett lock (inklusive `none`), `auto_default` om det var standardvärdet. Vid bearbetning av en registrering där locket antogs, med projektets DAQ-lock inställt på Auto, ger ett `[daq-dls] WARNING` som namnger filen: en ren DAQ-U under solskyddskorrigeringen ger ett värde som är ungefär 30 gånger för högt. **Namnge den kap som du använt** när du ansluter, eller ställ in projektets DAQ-kap, så försvinner varningen.
> - **Inga fler pip-konsolfönster i Windows.** På en dator med Python installerat körde backend-programmet och var och en av dess bildbearbetare `pip` en gång vid uppstart, och var och en öppnade ett konsolfönster. Det gör de inte längre.
> - **Live-inspelningar med `.daq` överlever ett strömavbrott.**En inspelning skrivs med en logg och en fullständig disksynkronisering vid varje commit, bakom sensorn, så en långsam disk fördröjer filen men aldrig avläsningarna. En inspelning som avbröts mitt i en commit (strömavbrott, avstängning) kan lämna kvar `<name>.daq-journal` bredvid sig:**kopiera de två tillsammans.** Chloros återställer journalen första gången den läser filen, så inspelningen öppnas med allt som kommiterats före avbrottet (ungefär de sista 3 sekunderna går förlorade); i 1.2.0 kunde ett strömavbrott vid fel tillfälle göra hela filen oläsbar (”databasdiskavbildningen är felaktig”).
> - **En inspelning på en upptagen disk blockerar inte längre sensorn.** `daq pool-record` köar avläsningarna och skriver dem i en egen tråd; en avläsning som inte kunde skrivas räknas (`daq pool-list` visar `N write error(s)` med det senaste felet) istället för att frysa sensorn (en skrivning tog 8,4 sekunder samtidigt som fyra kameror sparade TIFF-filer).
> - **`lattice multi-capture` utan en array tappar inte längre de bildrutor som sparats när en utlösare går ut.** En kameras utlösare som går ut (`SC_ERR_TIMEOUT -1011`, observerat med fyra kameror på en 1 GbE-switch) avslutar körningen vid den bildgruppen: de redan sparade grupperna listas med `Saved:`, därefter felet, exit `1`. I version 1.2.0 var det ett HTTP-fel 500 utan någon lista, trots att filerna fanns på disken.
> - **Programvaruaktiverade kameror missar inte längre vartannat utlösningssignal.** En kamera i programvaruaktiverat läge köar nu en utlösningssignal som anländer under dess avläsning (eller så väntar Chloros på att den ska bli klar), precis som kabelsynkroniserade kameror redan gjorde.
> - **`daq pool-discover` namnger en DAQ-E-S `DAQ-E-S`** i kolumnen `MODEL` (och `--json`:s `model`). Den angav `DAQ-E` för varje Ethernet-sensor; en som inte svarar på identitetskontrollen inom cirka 2 sekunder finns fortfarande med i listan, som `DAQ-E`.
> - **Förbättrade LATTICE-indexprodukter** (NDVI och andra index) från `process`, i alla bearbetningslägen.
> - **OCN-kameror: förbättrade radian-, reflektans- och indexprodukter i solljus.** En flottkorrigering tillämpas efter bandsepareringen; RGN-kameror är oförändrade, och OCN-värdena skiljer sig från version 1.2.0. Detta gäller även en OCN-bild som tagits med en tidigare version, när den bearbetas på en dator som innehåller den kalibrering som användes vid bildtagningen (Chloros behåller kamerans kalibrering så snart kameran har anslutits).
> - **`lattice continuous` spelar in råbilden.** En körning av `tiff` skrev ut demosaicerade 3-plansbilder, vilket `process` inte kan utgå ifrån. Dess bilder är nu den `raw`-fil som en array-inspelning skriver: ett mosaikplan med sensorns egna värden, okomprimerat, med kalibreringspaketet. Ett skript som läser tre plan från dessa filer läser endast ett.
>
> - **Varje bildtagning sparar den DAQ-avläsning som användes, precis som GUI:n gör.** `lattice capture`, `lattice continuous` (en `.daq` per bildram), `array-capture` och `multi-capture` med en ansluten matris (och SDK) skriver `.daq` och dess `.match.json` varje gång en DAQ binds eller läggs i pool, inklusive en ren råinspelning; i 1.2.0 sparas en rå eller debayered-inspelning endast med `--force-daq`. `--no-daq` stänger av funktionen, `--daq-tolerance-ms MS` ställer in matchningstoleransen för den inspelningen, och den nya `daq pool-bind` binder en kamera eller en matris till en DAQ från CLI:n. Med `.daq` matchas en inspelning som sparas i `process` endast mot de kameror som anges. En inspelning väntar nu på DAQ:ns nästa avläsning så att den kan blanda avläsningarna på båda sidor om exponeringen. Se [DAQ-avläsningen som en inspelning sparar](#the-daq-reading-a-capture-saves) och [En DAQ-avläsning långt från exponeringen](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** lägger till indexet i projektet för varje kameras band, inklusive ett nytt projekt. I version 1.2.0 ignorerades ett namn som ännu inte fanns med i projektet med `Unknown index name(s) ignored`.
> - **`process` bredvid en live-matris:**det startar inte längre om en backend som innehåller kameror eller DAQ:er, och väntar upp till 15 sekunder på en körning som håller på att avslutas.**Ctrl+C** stoppar körningen i backenden, inte bara i CLI:n. Se [`process`-anteckningar](#notes).
> - **En projektmapp som ligger för djupt för Windows gräns på 260 tecken i sökvägen** avvisas direkt med den längsta mappen som får plats, istället för en körning som inte skriver något. Windows-installationsprogrammet erbjuder nu ”Aktivera långa sökvägar i Windows” och ett undantag för Microsoft Defender för backend-delen, båda markerade som standard. Se [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** kör backendens egen kontinuerliga loggning: en förloppslinje varannan sekund, en sammanfattning istället för en filvis lista, och ett avslutningsvärde som inte är noll vid varje oskriven fil eller en full disk. Se [`array-capture`-lägen](#array-capture-modes).
> - **Det första `array-capture` efter `array-connect`** väntar upp till 20 sekunder på matrisens första bildgrupp istället för att avbryta med HTTP-kod 503 ”Array still warming up”.
> - **Arrayexponering under ett ROI:** ett litet ROI begränsar inte längre en arraymedlems exponering till sensorns frilöpningsperiod (4,9 ms vid 768 × 304). Se [Exponering på en arraymedlem](#exposure-on-an-array-member).
> - **Automatisk exponering sänker inte längre en arrays automatiska bildfrekvens.** En kabelsynkroniserad array körs med den hastighet som nätverket och kamerornas utläsning tillåter; automatisk exponering fyller den period som hastigheten ger och höjer sedan förstärkningen. En manuell exponering som är längre än perioden (`lattice exposure --set`, `configure --set`) saktar ner utlösningen för att anpassa den istället för att avvisas, och hastigheten återställs när enheten återgår till automatiskt läge. Se [Exponering på en array-enhet](#exposure-on-an-array-member).
> - **Arrayer levererar färre ofullständiga bildgrupper** och färre 503-fel (”Ingen ny synkroniserad bildgrupp”): varje slavkameras utlösningssignal är nu fri från störningar.
> - **Komprimerad 12-bitarsöverföring** (`--packed`, standard `auto`) höjer bildhastigheten för arrayer med full sensor med ungefär en tredjedel. `array-connect --packet-size` tillämpas nu (1.2.0 accepterade det men ignorerade det). Se [Packad 12-bitars transport](#packed-12-bit-transport---packed).
> - **En matris vars kameror återhämtar sig av sig själva efter en strömavstängning.** Efter en strömavstängning av switcharna öppnar `array-connect`:s matris-öppnar kamerorna när de svarar och strömmar igen med PTP återaktiverat; i 1.2.0 stannade den av helt och behövde en ny `array-connect`. `array-status` rapporterar `ptp_enabled: False` (och `ptp_off:`) medan en kamera läser av PTP, där den tidigare rapporterade den inställning som arrayen var ansluten med. Se [Persistenta anslutningar](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` spelar in utan att något GUI är öppet.** När GUI:ets kombinerade vy är stängd renderar backend-delen vyn själv: arrayensNDVI eller det index du skickar med det nya `--expr`. En inspelning som inte skriver någon bildruta misslyckas nu (`no-frames: …`, avsluta `1`). Tidigare spelade den in endast medan GUI:ns kombinerade vy var öppen, och en inspelning som inte skrev något avslutades ändå med `0`. Se [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **En kamera som Chloros redan innehar öppnas inte en andra gång.** Ett kommando som skulle öppna den själv svarar istället med HTTP-kod 409 `CAMERA_HELD`, där det anges vad som innehar den och vilket kommando som frigör den. Detta gäller även en kamera som ett annat kommando använder just nu (ett `multi-test` eller `multi-capture` som öppnar sina kameror, ett `lattice capture`): det andra kommandot avvisas tills det första är klart. Att öppna den igen kan stoppa innehavarens ström eller krascha backend-systemet. Se [Persistenta anslutningar](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Tillfälliga kommandon för flera kameror levererar alla kameror eller misslyckas.** `lattice multi-capture` utan ansluten matris, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` och `index --live` avslutas med att `1` namnger en kamera som inte levererade någon bildruta. Strömmarna hoppar över en bildgrupp där en kamera saknas och avbryts efter tre i rad; `multi-test` skriver ut en rad per utlösare, `(no frame)` där en kamera missade; `align-stream` avslutar också `1` vid ett jobbfel. Varje körning återställer sedan varje kamera till det tillstånd den fann den i: dess trigg- och synkroniseringsledningsanslutningar, dess PTP (`PtpEnable`) och alla kamerainställningar som körningen ändrade — exponering och förstärkning, AE-mål, gränser och dämpning, vitbalans, svartnivå, pixelformat, binning och ROI, speglingar, bildfrekvens — såvida inte `--keep-settings` / `--clear-settings` (se [Kameraminne](#camera-memory----keep-settings----clear-settings)). Tidigare behöll dessa kommandon alla bildrutor som kom in och avslutade `0` (`align-stream` visade ett jobbfel som `Note:`), och lämnade varje kamera kvar i `TriggerMode` `On`. De ställde också in varje kameras exponering utifrån scenen innan en bild sparades (högst några sekunder), på det mål där en ansluten matris startar varje kamera med: 40 multispektrala, 60 RGB (`--brightness N` ställer in det) och behåller den inställningen under hela körningen; detsamma gäller för en enskild `lattice capture` för en kamera som Chloros inte hanterar. Tidigare togs varje kameras första bildruta vid kamerans egen exponeringsgräns, cirka 100 ms, oavsett ljusförhållanden, och blev ofta klippt. Se [Flerkamerasystem (tillfälliga sessioner)](#multi-camera-transient-sessions).
> - **Tillfälliga flerkamerakommandon delar nätverket på samma sätt som `array-connect` gör, och öppnar endast `--serials`.** Varje kamera anpassas till sin andel av värdlänken, så fyra kameror körs på en 1 Gbps-länk; tidigare begärde de fyra sammanlagt 125 MB/s, vilket motsvarade hela 1 GbE, och varje bildruta förlorade paket. En uppsättning som nätverket inte klarar av avvisas innan någon kamera börjar strömma. `--serials` väljer nu ut kamerorna, där den första utlöser kommandot; tidigare öppnade varje kommando alla kameror i nätverket oavsett vad `--serials` angav. Ett fel som beror på paketförlust i GVSP anger nu detta istället för att peka på M8-synkroniseringskabeln. Se [Flera kameror (tillfälliga sessioner)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` låter kamerorna köras fritt, som de hittades.** `TriggerMode` `Off`, och allt annat som anslutningen fann – inställningarna för utlösare och synkroniseringslinje (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP samt de kamerainställningar som sessionen ändrade – så att ett endakamerakommando på en tidigare medlem hämtar bilder på en gång, och mastern lämnas inte längre kvar på `TriggerSource` `Software`. `--keep-settings` sparar sessionens inställningar i varje kameras minne, `--clear-settings` återställer det minnet (se [Kameraminne](#camera-memory----keep-settings----clear-settings)). `lattice continuous` ställer dessutom in en kamera som den upptäcker är aktiverad till friläge för körningen (och återställer den efteråt), och en körning utan ramavslut lämnar `1`.
> - **`lattice configure --get` / `lattice exposure`** skriver ut ett `note:` med kameransverkliga exponering eller förstärkning medan dess automatiska läge är aktiverat. Ett avvisat `lattice exposure --set` / `configure --set` anger det intervall som kameran accepterar och återställer det automatiska läget istället för att lämna den automatiska exponeringen avstängd. Se [Sensorkalibrering](#sensor-tuning).
> - **Nya kamerakontroller:**`lattice highlights` (dämpa ljusa vita ytor i RGB-visningen) och `lattice exposure --brightness-auto`.**`lattice multi-detect`** hittar de kabelanslutna kamerorna under en sondrunda och ställer in masteren som standard till RGB-kameran.
> - **Nya DAQ-kommandon:** `daq pool-bind`; `pool-enable-ptp`; och för en DAQ-E:s egna inställningar `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` och `pool-update-firmware`. Se [DAQ-E-enhetsinställningar](#daq-e-device-settings).
> - **En DAQ-E som slutar leverera data återansluts av backend-systemet.** När strömmen från en DAQ-E i en pool tystnar (t.ex. på grund av att strömbrytaren som driver den slås av och på, en avbruten kameralänk), återansluter backend-enheten den med de inställningar den hade och gör nya försök med allt längre intervall under tiden den är borta. En inspelning på den fortsätter i samma `.daq` med avbrottet markerat i filen, och `pool-record --stop` listar varje avbrott. I version 1.2.0 förblev sensorn tyst fram till `pool-disconnect` + `pool-connect`, vilket också avslutade inspelningen. Se [En DAQ-E som slutar leverera](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** rapporterar om PTP-grandmasterns paket når nätverket (`health`), och visar ett `WARNING:` på stderr när de inte gör det. Se [`time-sync`](#chloros-cli-time-sync).
> - **.deb-paket för Linux:** det utökar kärnans mottagningsbuffertar för socklar som kameraströmmarna behöver (vid kärnans standardinställning förloras bildrutor och bildfrekvens), lägger till en backend-tjänst som körs som din användare, och `lattice network` kontrollerar buffertarna. CLI:n avbryter aldrig en backend som körs av en tjänst, och vid en uppgradering startas de aktiverade backend-tjänsterna om. Se [Köra backenden som en tjänst](#running-the-backend-as-a-service-headless) och [Linux-värdar — socket-mottagningsbuffertar](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` anger när det inte gick att justera:**varje orsak till att en inspelning sparades ojusterad (ingen justeringsprofil, medlemmar med olika upplösningar, en array som inte är kabelsynkroniserad, en ram som profilen inte kan mappas till) skrivs ut som en `Alignment:`-rad.**`chloros-cli daq help`** är `daq --help` (det avslutades med `2`), och på Linux listar hjälpen de direkt-hårdvarubaserade `daq`-kommandon som ingår i kompileringen.
> - **Jetson:** paketet är `chloros_1.3.0_arm64_jp6.deb`, för JetPack 6, och dess GPU-bearbetning körs på Orin med JetPacks CUDA PyTorch. Se [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Konventioner

- Alla kommandon har prefixet `chloros-cli`. I Windows heter binärfilen `chloros-cli.exe`; i Linux/Jetson är den `chloros-cli`.
- Valfria argument visas som `--flag`. Obligatoriska positionsargument visas utan parenteser.
- Om ett standardvärde anges används det värdet om flaggan utelämnas.
- CLI:n är en enkel HTTP-klient via Chloros-backend (servern på `127.0.0.1:5000` som även desktop-appen använder). **Vilka kommandon startar den:**
  - De centrala kommandona — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` och `selftest` — använder en igångvarande backend, eller startar en om ingen är igång och stänger av den igen när de är klara.
  - **Kommandona `lattice`, `project` och `daq` startar aldrig någon.** De kräver en backend som redan körs: Windows-skrivbordsappens, Linux-[backend-tjänsten](#running-the-backend-as-a-service-headless) eller en som du startar själv (se [Starta backenden](#start-the-backend)). Utan en sådan visar de meddelandet `Error: Chloros backend not reachable at http://127.0.0.1:5000`, tillsammans med kommandot som startar den, och avslutas med `1`.
- `CHLOROS_BACKEND_URL=<url>` pekar på **`lattice`**,**`project`**och**`daq pool-*`** till en annan backend (`lattice network-analysis` pekar istället på `--backend-url`). Kärnkommandona låser medvetet fast `http://127.0.0.1:<port>` och ignorerar det (IPv4-literalen undviker Windows `localhost`→`::1`-straff på cirka 2 sekunder per begäran). Se [Miljövariabler](#environment-variables).
- **Relativa sökvägar tolkas utifrån den mapp där du kör `chloros-cli`.** Backenden är den process som öppnar ut- och ingångsmappar, så innan en sökväg skickas omvandlar CLI:et den till en absolut sökväg: `-o` / `--output` (inklusive dess standardvärde, `output`), `--input`, `--burst-dir`, `--profile`, `--save` och `--mask`, en [recept](#recipe-dsl)-stegets `output` (standard `output`), samt standardmappen `output` för `lattice index --live`. `-o captures/` skriver därför till `<current folder>/captures/`, och de utskrivna sökvägarna är absoluta. `~` expanderas. Vid en fjärrbackend (`CHLOROS_BACKEND_URL` som pekar på en annan maskin) skickas sökvägarna precis som de skrivits in och löses upp på den maskinen.
- Inloggning med ett Chloros+-konto krävs för alla SDK/CLI anrop (kör `chloros-cli login` en gång per maskin; cachas i `~/.chloros/`).
- Exemplen använder Linux-sökvägar; i Windows ersätter du `/home/user/...` med `C:/Users/.../...`. En DAQ-U:sserieport har formatet `COM3` i Windows och `/dev/ttyUSB0` / `/dev/ttyACM0` i Linux.
- Synopserna skriver ömsesidigt uteslutande flaggor som `{--a | --b}` och valfria flaggor som `[...]`.

---

## Synopsis på högsta nivå

```
chloros-cli [global options] COMMAND [command options]
```

### Globala alternativ

| Flagga | Beskrivning |
| --- | --- |
| `--backend-exe PATH` | Åsidosätt den automatiskt upptäckta backend-körfilen. |
| `--port N` | Backend-HTTP-port (standard: `5000`). Används endast av kärnkommandona (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` och `daq` använder `CHLOROS_BACKEND_URL` (standard `http://127.0.0.1:5000`). |
| `-v, --verbose` | Aktivera detaljerad utmatning. |
| `--restart` | Tvinga omstart av backend (avslutar alla körande `backend_server.py`). En backend som körs av en systemd-tjänst (Linux .deb) stoppas aldrig: kommandot skriver istället ut `sudo systemctl restart <unit>` och avslutas `1`. |
| `--version` | Visa version (`Chloros CLI 1.3.0`). |
| `--help` | Visa hjälp på högsta nivå. |

### Kommandoindex

| Kommando | Syfte |
| --- | --- |
| [`process`](#chloros-cli-process) | Bearbeta en mapp med Survey3- eller LATTICE-inspelningar från början till slut. |
| [`login`](#chloros-cli-login) | Autentisera den här datorn med ett Chloros+-konto. |
| [`logout`](#chloros-cli-logout) | Rensa cachade inloggningsuppgifter. |
| [`status`](#chloros-cli-status) | Visa aktuell licens-/autentiseringsstatus. |
| [`export-status`](#chloros-cli-export-status) | Visar exportförloppet för Live Thread-4 i realtid under en `process`-körning. |
| [`language`](#chloros-cli-language) | Ställ in eller visa CLI-visningsspråk (38 stöds). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Standardprojektmapp (delad med GUI). |
| [`update`](#chloros-cli-update) | Sök efter och installera Chloros-uppdateringar (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Systemdiagnostik + rökprov. |
| [`time-sync`](#chloros-cli-time-sync) | PTP-grandmaster-status / styrning. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-kamerastyrning och bildtagning (över 45 underkommandon). |
| [`daq`](#chloros-cli-daq) | Styrning av DAQ-spektralsensorer (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Öppna och kör ett sparat Chloros-projekt (kameror + DAQ:er). |

---

## Installation

`chloros-cli` ingår i Chloros-installationsprogrammet för skrivbordet på alla plattformar som stöds — det finns ingen separat nedladdning för CLI. När du installerar plattformspaketet läggs `chloros-cli` till i din `PATH` tillsammans med desktop-appen och den bakomliggande binärfilen som den styr.

Senaste nedladdningar: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Installationsprogrammet innehåller även praktiska startskript (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) som öppnar ett färdigt CLI-skal.

### Windows (.exe)

1. Ladda ner Windows-installationsprogrammet från nedladdningssidan.
2. Kör `Chloros Installer 1.3.0.exe` och följ guiden. Standardinstallationsvägen är `C:\Program Files\Chloros\` (CLI-skalet hamnar i `C:\Program Files\Chloros\cli\`, som installationsprogrammet lägger till i PATH).
3. Öppna en ny terminal (`cmd.exe`, PowerShell eller Windows-terminalen) så att den uppdaterade versionen `PATH` hämtas.

```powershell
chloros-cli --version
```

Installationsprogrammet lägger automatiskt till `chloros-cli.exe` i ditt systems `PATH` och inkluderar LATTICE-kameradrivrutinerna. Backend-mappen är `C:\Program Files\Chloros\backend\chloros-backend.exe` (mappen `backend` i din installationsmapp), som inte finns på `PATH`.

**Systeminställningar.** På installationsprogrammets sida *Välj ytterligare uppgifter* finns två alternativ under **Systeminställningar**, båda**markerade som standard**:

| Alternativ (enligt formuleringen i installationsprogrammet) | Vad det gör | Avaktivera tyst installation |
| --- | --- | --- |
| **Aktivera långa sökvägar i Windows** (gör det möjligt för projektmappar att ligga djupare än 260 tecken; rekommenderas) | Ställer in Windows systemomfattande inställning `LongPathsEnabled` till 1. Windows levereras med denna inställning avstängd, och då får ingen sökväg överskrida 260 tecken; Chloros-produkter ligger upp till cirka 130 tecken under projektmappen, så en djupare projektmapp avvisas (se [Felsökning](#felsöknings-tips)). Träder i kraft för program som startas efter installationen. Lämnas aktiverat vid avinstallation, eftersom andra program kan vara beroende av det. | `/MERGETASKS="!longpaths"` |
| **Uteslut Chloros-backend från Microsoft Defender-skanning**(mycket snabbare kamerafångst; rekommenderas) | Lägger till ett undantag för Defender-**processen** för `chloros-backend.exe` i installationsmappen, så att realtidsskyddet inte skannar varje fil som backend skriver, oavsett var den skrivs. Inget annat program undantas; undantaget tas bort vid avinstallation. Om Defender saknas eller är låst av policy misslyckas steget utan meddelande och installationen fortsätter. | `/MERGETASKS="!defenderexclusion"` |**Varför Defender-alternativet är viktigt:** vid höga inspelningshastigheter kostar realtidsskanningen av inspelningsfilerna cirka 30 % av inspelningshastigheten (uppmätt vid kontinuerlig inspelning med 9 kameror: 77,7 inspelningar/s exkluderade, 52,7 skannade). `lattice array-capture --continuous` visar ett meddelande när det upptäcker att Defender använder en eller flera CPU-kärnor vid inspelningen; kör installationsprogrammet igen med alternativet markerat.

En tyst installation (`/SILENT` eller `/VERYSILENT`) tillämpar båda om inte annat väljs, t.ex. `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Windows-brandväggen.** Installationsprogrammet lägger alltid till två inkommande regler för `chloros-backend.exe` i varje nätverksprofil, och avinstallationsprogrammet tar bort dem: `Chloros Backend (UDP-In)`, för kameraströmmar, PTP, mDNS och DAQ-E-dataströmmen, samt `Chloros Backend (TCP-In)`, begränsad till det lokala subnätet, för en DAQ-E-firmwareuppdatering, där sensorn ansluter tillbaka till den här datorn för att hämta bilden (se [DAQ-E-enhetsinställningar](#daq-e-device-settings)). Backendens HTTP-API förblir bunden till `127.0.0.1`.

### Linux amd64 (.deb)

För Ubuntu 22.04 LTS eller nyare / Debian-baserade x86_64-arbetsstationer.

> **Ubuntu 20.04 stöds inte.** Paketets beroendelista härleds från
> vad backend faktiskt länkar mot, och det inkluderar `libc6 (>= 2.35)`;
> focal levereras med glibc 2.31. `apt` avbryter installationen istället för att låta den misslyckas vid
> körning. Listan nämner även `libgl1`, som det medföljande OpenCV laddar och som kan saknas i en minimal
> serverbild; `apt install` tillför det.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

.deb-paketet installerar:
- `chloros-cli` till `/usr/bin/chloros-cli` och `chloros-backend` (startar backend) till `/usr/bin/chloros-backend`
- Den kompilerade backend-modulen och CLI:n, uppackade, i `/usr/lib/chloros/backend/` och `/usr/lib/chloros/cli/`
- LATTICE-kameradrivrutinerna
- Modeller för brusreducering, kalibreringspaket och konfiguration av uppdateringskanalen
- Python-SDK:et för systemet `python3` (`import chloros_sdk`). `apt install` medför även sina egna krav, `python3-numpy` och `python3-requests` (rekommenderas, men krävs inte: ovanstående program är fristående; `dpkg -i` hoppar över dem, och installationsutmatningen anger vad som saknas). I en virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
– `/etc/sysctl.d/60-chloros-gige.conf`, vilket ökar kärnans mottagningsbuffertar för kameraströmmar strömmar till 32 MB och tillämpar dem omedelbart (ett värde som redan är inställt på ett högre värde lämnas oförändrat). Om paketet avinstalleras tas filen bort. Se [Linux-värdar — socket-mottagningsbuffertar](#linux-hosts--socket-receive-buffers).
- Två valfria systemd-tjänster för backend, ingen av dem aktiverad (nästa avsnitt)

#### Köra backend som en tjänst (headless)

Utan en tjänst körs backend (och därmed PTP-grandmastern) endast så länge något har startat den: `chloros-backend` i en terminal, ett SDK-skript eller ett core-kommando `chloros-cli` under hela dess körtid. För en backend som alltid är igång på en headless-maskin, aktivera tjänsten **som din användare**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` kör backend-delen som den användaren. De data som den skriver tillhör den användaren, dess detaljerade logg finns i den användarens `~/.cache/chloros/logs`, och den använder deras `chloros-cli login` (`~/.chloros`), så logga in en gång som den användaren först. PTP behöver inte root. Liksom alla installerade backend-instanser lyssnar den endast på `127.0.0.1:5000`.

`chloros-backend.service` kör backend-programmet som **root**. Den är fortfarande installerad så att en maskin som har aktiverat den fortsätter att köras, men allt som en root-backend skriver tillhör root, så en användare kan inte radera sina egna inspelningar. För att byta till din användare:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Den detaljerade loggen för root-tjänsten finns i `/var/log/chloros` och kan läsas av medlemmar i gruppen `adm` (tjänsten ställer in `CHLOROS_LOG_DIR`; se [Miljövariabler](#environment-variables)). När paketet installeras visas kommandot `enable` med ditt användarnamn ifyllt, samt växlingskommandot när root-tjänsten är aktiverad.

**Aktivera en backend-enhet.** Endast en backend-enhet kan köras åt gången. En andra aktiverad enhet (`chloros-backend.service` vid sidan av `chloros-backend@<user>.service`, eller två användares `@`-enheter) vägrar att starta och förblir `failed` med avslutningsstatus 98; systemd startar inte om den. Dess `systemctl status` anger orsaken: `Another instance is already running. Exiting.`, sedan vad som håller kvar backend-enheten (port 5000, eller låsfilen och den användare den tillhör), sedan `systemd will not restart <unit> (exit status 98)`. Efter en omstart kan vilken som helst av enheterna vara den som körs. För att byta, inaktivera den du inte använder, enligt ovan.**CLI:n stoppar aldrig en tjänsts backend.** `chloros-cli` använder den som den är, precis som GUI:n och SDK:n gör: `process` körs på den och laddar sitt eget projekt istället för det senaste. Där CLI:et annars skulle starta om backend-delen (`--restart`, eller en `process` vars import inte hittade något), namnger den istället enheten, skriver ut `sudo systemctl restart <unit>` och avslutar `1`, eftersom en backend som stoppas på det sättet skulle förbli avstängd. `GET /api/status` namnger enheten som `service` (`null` för en backend där ingen tjänst körs).**Uppgradering och borttagning.** När paketet uppgraderas stoppas alla backend-tjänster, varefter varje aktiverad tjänst startas om, med enheterna `@<user>` först (en som startats manuellt men aldrig aktiverats förblir avstängd, precis som efter en omstart). Om fler än en är aktiverade körs endast en och de övriga förblir i felstatus enligt ovan; uppgraderingen visar meddelandena `Chloros: N backend units are enabled: …` och `Keep one enabled: sudo systemctl disable --now <unit>`. Avinstallation stoppar och inaktiverar dem. Om den rensas bort raderas även `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Samma uppbyggnad som amd64 .deb, inklusive mottagningsbuffertar och [backend-tjänster](#running-the-backend-as-a-service-headless), för Jetson Orin / Orin NX / Orin Nano på **JetPack 6** (L4T R36). Dess PyTorch är JetPack 6:s CUDA-version för Orin, så GPU-bearbetning, inklusive den texturmedvetna debayern, körs på Jetsons GPU. Paketet innehåller inte CUDA i sig: det använder de CUDA- och cuDNN-bibliotek som JetPack 6 installerar (`sudo apt install nvidia-jetpack` på en image som saknar dem). Om ingen CUDA-GPU kan användas körs bearbetningen på CPU:n och texturhanteringen använder standard-debayern (se [`process`](#chloros-cli-process)).

### Autentisera en gång per maskin

Varje plattform kräver en engångsinloggning på Chloros+ innan SDK- och CLI-anrop fungerar:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Inloggningsuppgifterna cachelagras i `~/.chloros/user_session.json`.

### Starta backend

Kommandona `lattice`, `project` och `daq` kommunicerar med en backend som redan körs; de startar aldrig en (se [Konventioner](#konventioner)). Se till att en är igång medan du använder dem:

| Plattform | Hur |
| --- | --- |
| Windows | Öppna Chloros-skrivbordsappen (dess backend betjänar även CLI), eller kör `"C:\Program Files\Chloros\backend\chloros-backend.exe"` i en egen terminal (mappen `backend` i din installationsmapp) och låt den vara öppen. |
| Linux/Jetson | Kör `chloros-backend` i en egen terminal, eller låt en alltid vara igång med [backend-tjänsten](#running-the-backend-as-a-service-headless). |

Endast en backend körs åt gången; alla klienter (GUI, CLI, SDK-skript) delar på den. Kärnkommandona (`process`, `login`, `status`, …) använder den också och startar en när ingen körs.

**En backend fortsätter att köras mellan kommandon**, så att nästa inte behöver starta om den igen. Den stängs av när du loggar ut (`chloros-cli logout`, såvida inte Chloros-appen är öppen eller en körning eller en offline-byggning pågår), när du kör `chloros-cli backend stop`, eller automatiskt efter 30 minuter om inget använder den: inget kommando, skript eller app, ingen körning eller offline-byggning, ingen ansluten kamera eller ljussensor. `backend stop` avvisar medan en körning eller en offline-byggning pågår och anger vilken; `--force` stoppar den omedelbart och den körningen eller byggningen går förlorad (en burst sparas, och `array-build-video` bygger den igen). Ställ in `CHLOROS_BACKEND_IDLE_MIN` innan backend-tjänsten startar för att ändra de 30 minuterna (`0`: aldrig); [backend-tjänsten](#running-the-backend-as-a-service-headless) stannar aldrig av sig själv. `chloros-cli backend status` visar om någon körs, vad som använder den och eventuella offline-byggningar som körs. `process` öppnar sitt projekt i den körande backenden, såvida inte Chloros-appen är öppen med ett projekt som den öppnat: då nekar den, och du måste antingen hantera det från appen eller stänga det projektet först. Ett projekt som en tidigare `process` eller ett SDK-skript lämnat öppet ersätts helt enkelt.

### Verifiera installationen

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` avslutas `0` först när alla sju kontroller har godkänts. Den sista kräver en CUDA-GPU (en NVIDIA-GPU, eller en Jetson Orin) samt brusreduceringsmodellerna, så på en dator utan en sådan visas `Failed: CUDA + Denoiser` och programmet avslutas vid `1`; allt utom den texturmedvetna debayern, som endast kräver GPU, fungerar fortfarande där.

> **Chloros+-abonnemang krävs.**CLI kräver ett aktivt Chloros+-abonnemang.**Copper**är instegsnivån i Chloros+ — alla betalda Chloros+-nivåer har tillgång till CLI/SDK; endast den kostnadsfria nivån**Iron** har inte det. (Abonnemangs-ID-mappning: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Uppgradera på [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Denna gräns tillämpas av backend-systemet, inte bara av CLI: en begäran med SDK/CLI-flagga utan betald plan avvisas med `403 PLAN_UPGRADE_REQUIRED`, oavsett om den kommer från `chloros-cli`, Python-SDK:et eller en egenutvecklad HTTP-klient. En utloggad användare får istället felkoden `401 AUTH_REQUIRED`. Åtkomst fungerar offline under abonnemangets respitperiod (30 dagar per månad, fram till utgångsdatumet för årsabonnemang) och upphör när den löper ut; `chloros-cli status` fortsätter att fungera så att orsaken syns (det är den enda SDK-/CLI-vägen som är undantagen från nivåbegränsningen — `GET /api/license-status`).

---

## Snabbstart på fem minuter

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (steg 2) startar en backend för sin egen körning när ingen är igång och stoppar den när den är klar, så starta en för steg 4 och framåt. Se [Starta backend](#start-the-backend).

---

## `chloros-cli process`

Bearbeta en mapp med bilder genom hela Chloros-pipeline (måldetektering → kalibrering → vignett → reflektans → indexexport).

### Översikt

```
chloros-cli process INPUT [OPTIONS]
```

### Positionsargument

| Argument | Beskrivning |
| --- | --- |
| `INPUT` | Sökväg till inmatningsmappen som innehåller filer med namnen `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) eller `.dng`. |

### Allmänna alternativ

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `-o, --output PATH` | en ny mapp med namnet `-n` i standardprojektmappen | Projektmappen. Projektet namnges efter den, och en mapp som redan innehåller en `project.json` får en `_1` / `_2` istället för att skrivas över. Standardprojektmappen är den som `set-project-folder` (eller GUI:n) valt, annars `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | auto (tidsstämpel) | Projektnamn, används endast **utan** `-o` (med `-o` får projektet mappens namn). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` använder en Chloros+ neural debayer för Survey3-bilder; långsammare men med högre kvalitet. Det är endast avsett för Survey3: LATTICE-bilder använder alltid standarddemosaik, så för en mapp som innehåller LATTICE-bilder avvisar `process` `texture-aware` efter importen, med ett felmeddelande som anger hur många det finns, och avslutar `1` innan något bearbetas (Chloros-appen erbjuder inte heller Texture Aware för dessa). Det krävs en CUDA-GPU (en NVIDIA-GPU på Windows/Linux x64, Orins GPU på en Jetson): utan en sådan använder körningen `standard`, och backend-loggen visar en rad med `[DEBAYER] … needs a CUDA GPU …` som förklarar varför (visas med `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Vignettkorrigering. |
| `--reflectance / --no-reflectance` | `--reflectance` | Reflektanskalibrering (använder panelmål om sådant hittas, NIST-kalibrering per serie för LATTICE). För LATTICE-multispektral fungerar detta även som reflektans **produkt** — se [Exportalternativ per produkt](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | av | Tillämpa PPK-GNSS-korrigeringar från sidecar-filer. |
| `--exposure-pin-1 CAMERA_MODEL` | ingen | Kameramodellen ansluten till exponeringsstift 1, för PPK-tidssynkronisering. |
| `--exposure-pin-2 CAMERA_MODEL` | ingen | Kameramodellen som är ansluten till exponeringsstift 2, för PPK-tidssynkronisering. |
| `--recal-interval SECONDS` | `0` | Minsta omkalibreringsintervall, i sekunder (projektets *Minsta omkalibreringsintervall*). |
| `--timezone-offset HOURS` | `0` | Ljussensorns tidszonsavvikelse från UTC, i timmar (projektets *Ljussensorns tidszonsavvikelse*). |
| `--format FORMAT` | `TIFF (16-bit)` | En av `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros ökar aldrig en bilds bitdjup: för bilder som helt och hållet består av 8 bitar skrivs ett format med större bredd som `TIFF (8-bit)` och detta anges en gång i körningen. I ett projekt som blandar 8-bitars- och 12/16-bitarsinspelningar exporteras varje inspelning med sitt eget bitdjup eller lägre. Radiance är alltid 32-bitars flyttal. |
| `--indices NAME [NAME ...]` | ingen | Vegetationsindex efter förinställningsnamn (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), skiftlägesokänsligt. Varje index läggs till i projektet för kamerans band, inklusive ett nytt projekt. Ett namn som inte är en förinställning, eller som kamerans band inte kan skapa (`NDRE` kräver RE och NIR, vilket ingen enskild kamera tillhandahåller), anges i en varning och får ingen indexbild. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Tvinga startpunkten för bearbetningskedjan för LATTICE TIFF-filer (Survey3.raw påverkas inte). Dessutom en nödutväg som gör att en inspelning utan **raw** kan bearbetas överhuvudtaget — se [Hur en inspelningsmapp ser ut](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | på | Skicka ut den linjära debayered-produkten (`Debayered_Images`). Se [Exportalternativ per produkt](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | på | Skickar ut förhandsgranskningen (`Preview_Images`): RGB = vitbalans (DAQ-ljuskälla om tillgänglig, annars gråskala) + gamma; multispektral = falskfärgsträckning. |
| `--radiance / --no-radiance` | på | Skicka ut strålningsvärde i float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referens för LATTICE-reflektansprodukten: `auto` = QA-godkänt mål inom ramen är den absoluta referensen (ett mål med en QA-flagga konkurrerar med DAQ om det uppmätta resultatet), DAQ-nedåtriktad (ρ = π·L/E) reserv; `target` = strikt (ingen DAQ-ersättning); `daq` = DAQ-auktoritativ. Se [Exportväxlar per produkt](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | ingen | Katalog över **uppmätta** målreflektansskanningar per enhet (`<serial>.csv`); faller tillbaka till de nominella T3/T4P-spektren vid avbrott. |
| `--array-alignment / --no-array-alignment` | på | LATTICE-matriser: tillämpa den modul-till-modul-justering som är inpräntad i varje bilds `Chloros:Alignment*` XMP på varje bearbetad produkt (debayering / förhandsgranskning / strålningsintensitet / reflektans / index). Ingen åtgärd för bilder utan taggarna. |
| `--array-alignment-crop / --no-array-alignment-crop` | beskärning | Beskär de justerade exportfilerna till matrisens gemensamma överlappningsområde så att alla moduler delar samma fotavtryck; `--no-…` behåller hela sensorns yta (svart fyllning utanför källan). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Omprovtagning för justeringsförvrängningen. `nearest` bevarar exakta käll-DN-värden (ingen blandning av radiometriska värden mellan pixlar). |

### Alternativ för måldetektering

| Flagga | Beskrivning |
| --- | --- |
| `--min-target-size PIXELS` | Minsta storlek i pixlar för kalibreringsmål för detektorn. |
| `--target-clustering 0-100` | Procentandel för målkluster (standard `60`). |
| `--target / --targets` | Sök endast efter kalibreringsmål i en `target`- eller `targets`-underkatalog till ingångsdata (snabbare). |

### Exempel

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Exportalternativ per produkt (LATTICE multispektral)

LATTICE-bearbetningen fördelas till **alla tillämpliga produkter i ett enda steg**. De fyra alternativknapparna per typ — `--debayered`, `--preview`, `--radiance`, `--reflectance` — är alla**aktiverade som standard**; använd formuläret `--no-<type>` för att inaktivera en. RGB-huvudkameror sänder endast debayered + förhandsgranskning (ingen strålning/reflektans per band), så `--radiance`/`--reflectance` har ingen effekt för dem. Växlarna ignoreras för Survey3 `.raw` (som följer standardvägen för reflektans/mål). *(Den gamla `--radiometric-output {reflectance,radiance,sensor-response}`-flaggan **togs bort** och ersattes av dessa växlingsalternativ; det finns ingen `sensor-response`-nivå längre.)*

| Produkt | Utdata | Behövs DAQ-nedåtriktad mätning? |
| --- | --- | --- |
| `--debayered` | Linjär demosaik (`Debayered_Images`): samma fil som en inspelning sparar på sin `debayered`-nivå — kalibreringspaketets mörkförskjutning (DSNU) från rådata, därefter OpenCV:s kantmedveten demosaik; ingen linearitets-LUT, flat field eller unmix. | Nej. |
| `--preview` | Förhandsgranskning på skärmen (`Preview_Images`): RGB = vitbalans + gamma; multispektral = falskfärgssträckning. | Nej. |
| `--radiance` | float32 W/m²/sr/nm från den fullständiga radiometriska kedjan (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 reflektans ρ (`32768` = 1,0), Pix4D-kompatibel. | **Ja**, såvida inte ett QA-godkänt mål inom bildramen förankrar den (se nedan). |

`--reflectance-source` väljer reflektansreferensen:**`auto`**(standard) gör ett QA-godkänt mål inom bilden till den**absoluta referensen**— de målförankrade empiriska linjekedjorna korsjämförs på uteslutna paneler och den uppmätta vinnaren tillämpas, band för band: ett band som DAQ inte kan kalibrera, eller vars DAQ-lutningsanpassning misslyckas, tar målets egen linje (`ReflectanceElBandMethods` anger vilken) — ett mål som klarar kvalitetskontrollen med en flagga (en borttagen panel, ett band som inte fungerar) konkurrerar också med DAQ-delningen om det uppmätta resultatet, och en ram som det förlorar mot märks med `ReflectanceChain=daq` — faller tillbaka till DAQ:s nedåtriktade gräns (ρ = π·L/E) när inget mål är närvarande eller QA misslyckas;**`target`**är strikt (ingen DAQ-ersättning);**`daq`**väljer bort det DAQ-auktoritativa beteendet. Målgeometri (ArUco / fast ROI / remsa) hämtas från projektets målkonfiguration; `--target-reflectance-dir DIR` lagrar**uppmätta** skanningar per enhet (`<serial>.csv`) som hämtas utifrån målenhetens serienummer/QR-kod, med de nominella T3/T4P-spektren som reserv.

DAQ-reflektansvägen löser automatiskt ut den **tidsstämpelmatchade nedåtriktade strålningen**från en inspelad**`.daq`**(DAQ-U/M/E)**eller en DAQ-M-inbyggd `.csv`**som finns tillsammans med bildmaterialet. Om ett kalibreringspaket per kamera eller DAQ inte finns i den lokala cachen,**hämtar pipelinen den automatiskt från AWS** vid första användningen (kräver internetanslutning en gång; cachas under `~/.chloros/`).**Vems avläsning är det?**En `.daq` som en inspelning sparar är avläsningen från den eller de kameror den beskriver — namngiven i en `<file>.daq.cameras.json`-sidokarfil bredvid den (`lattice capture`, `multi-capture`, `array-capture` och i GUI:n; en rå bildserie namnger varje medlems avläsning i sitt manifest) — och `process` matchar den**endast med bilder från dessa kameror**. En fristående kamera utan egna avläsningar lånar därför inte längre en annan kameras DAQ; koppla en DAQ till den, eller spela in en. En DAQ:s egen inspelning (`daq pool-record`, eller en inspelning som startats i GUI:n), eller vilken `.daq` som helst utan den sidokaraktären (även äldre inspelningar), tjänar fortfarande alla kameror.

#### Avläsning av reflektanspixlar (Pix4D / Metashape / dina egna skript)

Reflektansen lagras som ett heltal (DN), och **det DN-värde som motsvarar ρ = 1,0 beror på källkameran**:

| Källa | ρ = 1,0 är | Hur man avgör |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (utrymme upp till ρ 2,0) | XMP-taggen `Chloros:PixelScale=32768` är inlagd i filen. |
| Survey3 | `65535` (klippt vid ρ 1,0) | Inga `Chloros:*` XMP-taggar — den avsaknaden *är* tecknet. |

**Läs av `Chloros:PixelScale` och dividera med det** istället för att anta ett konstant värde. Taggen är definierad i uint16-domänen, så den förblir `32768` i utdataformat som skalar om — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` och `TIFF (32-bit, Percent)` är alla självbeskrivande (normalisera först den lagrade datatypen tillbaka till uint16: ×257 från 8-bit, ×65535 från float).

> **Filer från äldre versioner kan sakna skalning.** En 8-bitars reflektans-TIFF-fil skalas om och märks `Chloros:PixelScale` precis som de andra. Versioner före 1.3.0 *klippte* istället av den till 0..255, så varje värde över ρ≈0,008 jämnades ut till 255, och de utelämnade både `Chloros:PixelScale` och tuplen `MicaSense:RadiometricCalibration`. **Om taggen saknas i en LATTICE-reflektansfil, antag inte någon skala — bearbeta den på nytt** istället för att dela pixlar som aldrig var delbara.

#### Läsning av indexpixlar (NDVI och övriga vegetationsindex)

**En heltalindexraster är en kontraststräckning per bild, inte en −1…1-karta.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` och `JPG (8-bit)` – inklusive den 8-bitars TIFF-fil som en 8-bitars inspelning skrivs som – lagrar

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

där `IndexMin` / `IndexMax` är **denna bildrams egna lägsta och högsta ändliga indexvärden**. DN 0 är inte −1, och full skala är inte +1.

Chloros stämplar båda ändpunkterna så att mappningen förblir inverterbar:

| Tagg | Betydelse |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konventionen nedan |
| `Chloros:IndexMin` | det indexvärde som lagras som DN 0 |
| `Chloros:IndexMax` | det indexvärde som lagras som full skala |

Läs in dem och invertera, normalisera det lagrade DN-värdet till 0…1 utifrån det lagrade datatypens maximivärde först, så att ett uttryck täcker alla heltalformat:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Inläsning kräver inget `-config pix4d.config` — exiftool läser dessa XMP-egenskaper utan hjälp. Konfigurationen krävs endast för att *skriva* dem.

> **`IndexMin == IndexMax`** betyder att varje pixel i den bilden hade samma indexvärde. Rastern är fylld till mitten av skalan och det finns inget att dividera med — ta ändpunkten.

> **Inga `Chloros:Index*`-taggar innebär att ingen mappning anges.** En LUT-export är en färglagd rendering vars pixlar består av gradientfärger som är kopplade till ditt LUT-tröskelfönster, och en bildruta utan pixlar med ändligt index har inget intervall att ange. Antag inte att det finns någon mappning i något av fallen – samma regel gäller som vid en saknad `PixelScale`.**Eftersom sträckningen sker per bild är två bilder från samma flygning inte jämförbara som DN**— varje bild har sina egna ändpunkter, så samma NDVI hamnar på olika DN i olika bilder. För kvantitativt arbete över en flygning (zonstatistik, fasta tröskelvärden, tidsserier), exportera**`TIFF (32-bit, Percent)`**: indexraster skrivs där som float32**indexvärdena själva**, utan omskalning och utan att taggar behövs. Trots formatets namn är indexprodukter inte ×100 – den skalningen gäller endast reflektans med heltalstyp.

#### Unmix-klämmask (vilka pixlar ett band klämdes på)

Den spektrala 3×3-unmixen kan göra att ett band blir negativt – där ljuset knappt innehåller det bandet, eller där scenen faller utanför vad kamerans unmix-matris beskriver. Det är inte ett verkligt värde, så exporten sätter det till 0 – och livevisningen visar samma 0. Ett band kan klämmas fast på några få pixlar eller på nästan alla; masken registrerar båda fallen. `Chloros:Band{R,G,B}UnmixClampFraction` anger hur stor del av en bildruta detta gällde; klämmasken anger **vilka pixlar**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— i en `clamp_masks/`-mapp**bredvid** produktens mapp, aldrig inuti den (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; en export från `process`:s `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — för varje strålningsvärde, reflektans och indexprodukt där minst en pixel har klämts fast: både inspelningar och `process`-exporter (samt TIFF-filerna per bildruta för `array-build-video --save-tiffs`). Enbands-`uint8`, på produktens eget pixelrutnät (inklusive orientering, ROI, binning och matrisjustering). Varje mask namnger produktens kamera (`Make` / `Model` / serienummer), inspelningstid och objektivtaggar, samt listar de band som dess bitar står för (`XMP-Camera:BandName`, bit 0 först); dess `ImageDescription` och `Software` markerar den som en klämmask, och den innehåller ingen kalibrering.
- **Bitar**: bit 0 = bandet i produktens R-kanal (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). En aktiverad bit innebär att pixelvärdet i det bandet har klämts fast till 0 — betrakta den, och alla index som bygger på den, som låg tillförlitlighet.
- **`Chloros:UnmixClampMask`** stämplas på varje produkt där avblandningen har körts: maskens sökväg relativt produktens egen mapp (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, alltid separerad med `/` – lägg till den till bildens mapp för att öppna masken), eller `none` när ingen pixel har begränsats. Ingen tagg innebär att produkten aldrig har genomgått unmix-processen (RGB- eller monokameror, rådata/debayering).
- `lattice capture` / `array-capture` skriver ut maskens sökväg under den sparade filen (`clamp mask: …`), och i listan över inspelningsresultat anges den som bildrutan `clamp_mask`.
- `lattice index --live --save-multiband` skriver en mask per källkamera för multibandstacken — se [`lattice index` — multibandstackmasker](#lattice-index--multiband-stack-masks). En video har ingen mask: `array-build-video` rapporterar istället varje bildruts klämningsfraktioner (se [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Eftersom ingen produktmapp innehåller en mask, oavsett nivå, kan en produktmapp (`radiance/`, `reflectance/`, en `process`-exports `*_Images/`-mapp) överlämnas till Pix4D, Metashape eller annan fotogrammetriprogramvara precis som den är — inklusive undermappar — och programmet tar endast emot bilder. Överför produktmappen, inte mappen ovanför den (den innehåller även `clamp_masks/` och alla andra exporttyper). Chloros egen import hoppar över maskerna. Tolkning av en mask: `(mask & 1) != 0` betyder ”R-kanalbandet klämdes fast här”, `& 2` G, `& 4` B.

#### EXIF-data överförs till exporten

`process` kopierar källbildens **GPS-block och dess ExifIFD** till varje produkt, så en
export innehåller `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` och
`CameraSerialNumber` tillsammans med georefereringen.

**`FocalLength` är inte valfritt för fotogrammetri.** Pix4D beräknar markprovsavståndet utifrån
brännvidd plus höjd; om taggen saknas faller programmet tillbaka till en helt felaktig skala. Vid en
flygning över en apelsinlund med 49 bilder förvandlade den saknade taggen ett område på 411 m × 160 m till en rekonstruerad
47,8 km × 13 km – en ortofoto på 455 MP bestående mestadels av nodata, vilket sedan tolkades som ett problem med bildrutor och
ett BigTIFF-problem innan någon kontrollerade GSD. Om din ortofoto får en osannolik
skala, kör först `exiftool -FocalLength` på den exporterade produkten.

Kopian är medvetet **inte** `-all:all`: IFD0:ss strukturtaggar stör LATTICE-utdata när de
kopieras, och `ExifImageWidth` / `ExifImageHeight` utesluts eftersom de beskriver
*käll*inspelningen – en export som någonsin har ändrats i storlek skulle annars ha dimensioner
som strider mot dess egen raster. XMP skrivs direkt istället för att kopieras, eftersom ExifTool
kasserar XMP-taggar från samma anrop när XMP-blocket kopieras (vilket skulle ta bort MAPIR-
kalibreringstaggarna).

### Var utdata sparas

Produkterna skrivs **i projektmappen, grupperade efter kamera och sedan efter filformat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kameramappen är `LATT-<sensor>-<lens>-F<filter>` för LATTICE (vilket stämmer överens med bildens EXIF
`Model`) och `<model>_<filter>` för Survey3 — två kameror som delar sensor och filter men skiljer sig
åt i objektiv behåller separata träd, eftersom vinjettering, synfält och distorsion skiljer sig åt. Formatmappen
följer `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, eller `tiff32` för
`TIFF (32-bit, Percent)`.

> **Varje exporterad produkt behåller namnet på KÄLLFILEN.** En radiance-export av
> `capture_…_raw.tif` heter fortfarande `capture_…_raw.tif` — den finns bara i
> `tiff32/Radiance_Images/`. **Mappen identifierar produkten, inte filnamnet**, så om du söker
> efter `*radiance*.tif` hittar du ingenting; sök istället efter mappen.

### Inspelningar från ljussensorn — kalibrerade `.daq` + `.csv`

`process` hanterar även `.daq`-inspelningarna i din inmatningsmapp, och det **behöver inte**
behöver några bilder för att göra det: en DAQ-U / DAQ-M / DAQ-E som flygs på egen hand är en fullständig
insamling, och en mapp som endast innehåller `.daq`-filer är en giltig indata.

En DAQ kan spelas in **utan** kalibrering – det är vad de allmänna
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) inspelare
(`record_daq.py`) gör som standard: de skriver ut råa sensorräkneställningar och stämplar filen så att
Chloros hämtar den sensorns fabrikskalibrering **via seriell** (först från den lokala cachen,
sedan från MAPIR-molnet) och tillämpar den. `process` skriver ut resultatet igen:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` innehåller en rad per avläsning: UTC-tidsstämpel, integrationstid, total effekt,
fotopisk/skotopisk lux, PPFD (och dess uppdelning i blått/grönt/rött), toppvåglängd, sedan
hela spektrumet på sensorns eget våglängdsraster. `.daq` importeras på nytt utan att
kalibreras en andra gång.

`<name>` är inspelningens eget filnamn. Två inspelningar i olika mappar kan dela
en – mätvärden per bildtagning (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) som sparats i
samma millisekund av två kameror – och var och en får ändå sina egna produkter: den första behåller
`<name>`, den nästa lägger till de mappar som skiljer den åt **efter** namnet
(`<name>__<capture>_daq_calibrated.daq`), så att inspelningens datum och tid fortfarande står först.

Vid lyckat resultat rapporterar körningen `Light-sensor products written: N (calibrated .daq + .csv)`.
Texten inom parentes beskriver vad som faktiskt skrevs, så det lyder
`(RAW COUNTS — this sensor has no calibration bundle)` för en sensor utan paket och
`(N calibrated, M raw counts)` för en mapp som innehåller båda. Backendens egna
rubriker `[DAQ-EXPORT]` och `[RUN-SUMMARY]` härleder sin formulering på samma sätt — ingen av
de tre kan kalla en rå export kalibrerad.

En DAQ-U-/DAQ-M-/DAQ-E-inspelning vars kalibreringspaket inte kan hämtas – du är
offline, eller så finns ingen kalibrering registrerad för den sensorn – **hoppas över med en orsak** på en
`[DAQ-EXPORT]`-rad och skrivs aldrig ut som en ”kalibrerad” fil som innehåller råa räknevärden.
Anslut till internet och kör om. Orsaken är den som läsaren faktiskt
fastställt för den filen (oläsbart schema, inget paket, ett skrivfel), och körningssammanfattningen
listar **distinkta** orsaker – tjugo filer som hoppats över av en orsak läses som en
orsak, inte tjugo upprepningar av den. CLI:n namnger varje utelämnad inspelning i slutet av
körningen, tillsammans med orsaken:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

En mapp med inspelningar där **ingen** exporteras misslyckas: se
[En körning som inte skriver ut några bilder misslyckas](#a-run-that-writes-no-images-fails).

En inspelning vars namn slutar på `_calibrated` eller `_raw` (`field_raw.daq`) **importeras inte**:
importeras**: det är de namn som `process` ger sina egna produkter, och en produkt som läses in
som en inspelning skulle räknas två gånger. CLI-gränssnittet anger detta och namnger filen —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— istället för att rapportera en tom mapp. Byt namn på en sådan inspelning innan du bearbetar den.

#### DAQ-A-inspelningar exporteras som råa räkneställningar

**DAQ-A** -familjen är äldre än systemet med serienummerbundlar och har ingen kalibreringsbundel
att hämta – den kalibreras istället ute på fältet mot ett reflektansmål, vilket är
anledningen till att den aldrig behövt någon. Att avvisa dessa inspelningar gjorde att det inte fanns något sätt att få ut deras
värden alls, så de exporteras under ett **annat namn**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Ett annat filnamn istället för en flagga inuti filen, eftersom uppgiften måste klara av att
skickas vidare via e-post som ett rent namn. Rubriken `.csv` anger
`raw spectral sensor counts (NOT irradiance)` och varnar för att värdena är jämförbara
**inom** filen – vilket är precis vad målbaserad kalibrering använder dem till – och
inte mellan olika sensorer. De effektberoende fotometriska kolumnerna (total effekt, fotopisk och
skotopisk lux, PPFD) skrivs ut som **NULL** istället för att integreras från räknevärden, och körningssammanfattningen
anger `RAW COUNTS`, så ”exporterat” i en logg kan inte tolkas som irradians.

Äldre **v1.01 / v1.02**-inspelningar (en DAQ-A-SD skriver dessa) har ingen epok per avläsning,
utan endast filens skrivtid. Bild↔nedåtriktad matchare avvisar dem fortfarande — att matcha en
bildram mot en skrivtid skulle ge osynliga fel — men exportören läser dem, och
CSV-filen skriver ut `clock=daq_created_on` så produkten anger vilken klocka den är inställd på.

### Anmärkningar

- `process` upptäcker automatiskt om din mapp är av typen Survey3, LATTICE eller blandad.
- Förloppet strömmas via Server-Sent Events; CLI:n visar realtidsförlopp per tråd (Detekterar, Analyserar, Bearbetar, Exporterar).
- För Linux/Jetson kontrollerar CLI-verktyget swap-utrymmet och kan visa en varning innan stora mappar bearbetas. Den texturmedvetna debayern tillämpar också automatiskt en GPU-frekvensbegränsning på Jetson-enheter med låg effekt (Nano, Orin Nano).
- Vid lyckat resultat rapporterar körningen hur många bildprodukter som skrevs (`Image products written: N`, samt `(plus M clamp masks)` när avblandningen begränsade någon pixel).
- **`process` delar en körande backend på ett säkert sätt.** Den startar om en redan körande backend automatiskt endast när den backenden är inaktiv med ett tidigare projekt laddat och inte hanterar några kameror, matriser eller DAQ:er. En backend som hanterar något av dessa (GUI:n som strömmar en array, ett SDK-skript, en DAQ-inspelning) används som den är och startas aldrig om, så `process` kan köras parallellt med en aktiv array. Om backenden rapporterar att en körning pågår väntar `process` väntar upp till 15 sekunder på att den ska avslutas (en körning som just har avslutats tar ungefär en sekund att rensa), och avvisar sedan med `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. Den globala flaggan `--restart` tvingar fortfarande fram en ny backend och avslutar alla kamera-, array- eller DAQ-sessioner på den. En backend som körs av en Linux-systemd-tjänst startas aldrig om: `process` använder den, och `--restart` skriver ut `sudo systemctl restart <unit>` istället (se [Köra backend som en tjänst](#running-the-backend-as-a-service-headless)). Det gäller inte heller Chloros-appens egen backend: när appen är öppen använder `process` den och laddar sitt projekt istället för appens (en backend som startas i dess ställe skulle låsa ut den öppna appen).
- **En pågående körning behåller sitt projekt.** Medan en bearbetningskörning pågår — startad av appen, `process`, ett SDK-skript eller `chloros-mcp` — vägrar backend-miljön att skapa, öppna, importera till, lägga till eller ta bort filer från, flytta, radera eller ändra bearbetningsinställningarna för ett projekt, med HTTP-kod 409 `PROCESSING_IN_PROGRESS` och `A processing run is in progress on project '<name>', so ... now is refused`; en `process` som lyckas med detta avslutar `1`. Visningsinställningarna (miniatyrstorlek) sparas fortfarande. Bearbetningsläget och projektets inställningar för målreflektans avvisas på samma sätt (en körning behåller det läge den startade i). En inspelning som sparas i projektet mitt i en körning (från appen, CLI, SDK eller `chloros-mcp`) sparas som vanligt, men läggs till i projektets fillista först när körningen avslutas; vilket framgår av svaret (`project_add`), liksom en ljussensorinspelning som avbryts och läggs till i projektet. Tidigare kunde en andra klient byta det öppna projektet mitt i en körning, och den pågående körningen exporterades till det andra projektet.
- **Ctrl+C avbryter körningen i backend, inte bara i CLI.** CLI:n ber backend-miljön att avbryta bearbetningen, visar meddelandet `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` och väntar i upp till 120 sekunder på `Processing stopped.`, så att `chloros-cli process … && next` inte startar `next` på en backend som fortfarande är upptagen med körningen. Ett andra Ctrl+C avslutar väntan. `process` avslutar då `130`, precis som alla avbrutna kommandon gör.

#### En körning som inte skriver ut några bilder misslyckas

Om du begärde produkter och körningen skrev **inga** — endast `project.json` och
`calibration_data.json` — behandlar `process` detta som ett misslyckande: det skriver ut
`Processing finished but wrote no image products.` och **avslutas med ett värde som inte är noll**, så att ett skript kan
upptäcka det. Meddelandet anger projektmappen och de vanligaste orsakerna:

- inmatningsmappen kändes inte igen som en inspelning (kontrollera layouten och `--input-level`), eller
- alla begärda produkter hoppades över eftersom de inte var tillämpliga för dessa kameror (t.ex. vid begäran om
  radian/reflektans från kameror som endast har RGB).

Kör om med `--verbose` och kontrollera backend-loggen för raderna `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
vilka förklarar utelämnanden per kamera som annars inte visas i CLI-utmatningen.

En avsiktlig körning endast med metadata — alla produktalternativ avstängda och utan `--indices` — är fortfarande en
**framgång**, eftersom en tom bildutmatning är det korrekta resultatet i det fallet.

#### En körning vars exporter inte kunde verifieras misslyckas

Efter en körning kontrollerar `process` de exporterade filernas taggar med ExifTool. Om ExifTool inte kunde köras,
skriver det ut `Exportverifiering MISSLYCKADES: ExifTool kunde inte köras, så taggarna för N av M exporterade filer
inte kontrollerades (se [EXIFTOOL]-raderna i backend-loggen)` and `Produkterna skrivs ut och finns kvar
på sin plats; kör om för att verifiera dem.`, does not print `Bearbetningen är klar!`, och **avslutas med ett värde som inte är noll**.
Produkterna behålls.

#### Vilka bilder en körning tar med sig

Bilder som färdigställdes innan `process` kördes inkluderas. Bilder som sparades under körningen inkluderas inte; `process`
varnar om hur många som utelämnades, och om man kör det igen inkluderas de.

#### En körning där en bild gått förlorad misslyckas

Om en bild misslyckades under en körning, skriver `process` ut `N image(s) failed and were not exported` med den
vanligaste orsaken, skriver inte ut `Processing complete!` och **avslutas med ett värde som inte är noll**. De övriga produkterna
skrivs ut och förblir på plats, och backend-loggen innehåller varje fel.

Detsamma gäller en **körning med endast ljussensor**: en mapp med `.daq`-inspelningar har per definition inga bilder att exportera,
och körningen bedöms istället utifrån de kalibrerade `.daq` / `.csv` som den skrev.
Om den inte skrev **något** — alla inspelningar hoppades över — skriver den ut
`Processing finished but wrote no light-sensor products.`, namnger varje inspelning med
anledningen till att den hoppades över, och **avslutas med ett värde som inte är noll**.

---

## `chloros-cli login`

Autentisera den här maskinen med ett Chloros+-molnkonto. Inloggningsuppgifterna cachelagras säkert i `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Exempel

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (genom att ta bort eller duplicera delar av lösenordet). Vid ett 401-fel försöker CLI:n automatiskt igen med `$$`-tillagt, sedan med en avduplicerad halva av lösenordet; om ett nytt försök lyckas loggar det in dig och visar den korrekta syntaxen med enkla citattecken som ska användas nästa gång.

> **Användning utan grafiskt gränssnitt/med skript: ingen cachad session innebär en inloggningsprompt.** Ett kärnkommando (`process`, `status`, `export-status`, `time-sync`, …) som körs utan en cachad session begär `Email:` / `Password:` via stdin innan den fortsätter. Om stdin är stängt (cron, en systemd-enhet) misslyckas det omedelbart med `login cancelled`; om stdin är en öppen terminal eller pipe väntar det på indata. Kör `chloros-cli login EMAIL PASSWORD` en gång per maskin innan du schemalägger headless-arbete.

---

## `chloros-cli logout`

Rensar den cachade sessionen och tvingar fram en ny inloggning vid nästa anrop.

Att logga ut medan en bearbetningskörning pågår stoppar inte den: körningen avslutas i det bearbetningsläge den startade i, och nästa körning använder standardbearbetning. `logout` anger detta (`The processing run in progress finishes in <mode> mode; the next run uses standard`), och detsamma gör appen när du loggar ut där.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Visar aktuell licensnivå (Iron/Copper/Bronze/Silver/Gold), autentiserad användare och antal enhetsbindningar.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Hämta aktuell status för Thread-4-exporten. Kan säkert anropas **under** en `process`-körning från ett annat skal.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Ställ in CLI:s visningsspråk (38 språk stöds, inklusive CJK, RTL och indiska språk). Faller smidigt tillbaka till engelska på äldre konsoler som inte kan återge skriptet.

```
chloros-cli language [LANG_CODE] [--list]
```

### Exempel

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Kommandon för projektmapp

Dessa hanterar standardplatsen för projektmappen (delad med GUI).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` visar den mapp dit nya projekt sparas. Om den konfigurerade mappen inte längre finns (den har raderats, bytt namn eller ligger på en enhet som inte är ansluten), faller CLI:et tillbaka till standardmappen, precis som GUI:t gör: standardmappen skrivs ut på stdout, en varning som anger den saknade mappen skickas till stderr och avslutningskoden är `0`.

---

## `chloros-cli update`

Endast Linux/Jetson. Kontrollerar `version_url` mot `/etc/chloros/update.conf` och, om en nyare version finns, laddas den motsvarande versionen ner och installeras omedelbart (ingen bekräftelsefråga; du kan bli ombedd att ange ditt lösenord). På en Jetson erbjuds uppdateringen endast till system med JetPack 6. I Windows gör kommandot ingenting och rapporterar `up to date`: uppdatera med ett nytt installationsprogram.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

På Linux/Jetson utför CLI:n också en **automatisk uppdateringskontroll vid varje uppstart** (blockerar inte, fördröjer aldrig kommandot): den läser `/etc/chloros/update.conf`, cachelagrar resultatet i 1 timme i `~/.chloros/update_cache.json` och visar `Update available: vX.Y.Z / Run: chloros-cli update` när en nyare version finns. Hoppas tyst över vid eventuella fel och på Windows; `CHLOROS_DISABLE_UPDATE_CHECK=1` stänger av funktionen. På grund av cachen på 1 timme kan en version som publicerats inom den timmen fortfarande läsas som uppdaterad.

---

## `chloros-cli selftest`

Kör ett 7-stegs smottest: version, porttillgänglighet, uppstart av backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), förekomst av brusreduceringsmodell, beredskap för CUDA+brusreducerare. Det använder en igångvarande backend, eller startar en för testet och stänger av den efteråt. Det visar `N/7 checks passed` och de misslyckade kontrollerna, och avslutas `0` först när alla sju har godkänts; kontrollen av CUDA+brusreduceraren misslyckas på en dator utan en CUDA-GPU (se [Verifiera installationen](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP-grandmaster-status och styrning. Chloros-värden kör PTP-grandmastern; LATTICE-kameror och DAQ-E-enheter fungerar som slavar till den för tidsstämplar mellan enheter (cirka 1 ms mellan enheterna). PTP synkroniserar **tidsstämplar**: kamerorna i en matris exponerar samtidigt eftersom M8-hårdvarutriggaren utlöser dem (mindre än 100 µs mellan modulerna), inte på grund av PTP. `chloros-cli time-sync` utan underkommando kör `status`.

| Underkommando | Beskrivning |
| --- | --- |
| `status` | Visa grandmaster-status, BMCA-prioriteringar, klockidentitet och om dess paket når nätverket: `health` är `OK` (`STOPPED` när grandmastern inte körs), eller `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` tillsammans med en `health_detail`, och sedan skrivs en rad med `WARNING:` ut till stderr (stdout förblir JSON). `sync_sent` / `announce_sent` räknar endast meddelanden som skickades; `*_send_errors` och `last_send_error` räknar de som inte skickades. När en kameralänk bryts och återupprättas återuppbygger grandmastern sina socklar på egen hand (`rebinds`, `last_rebind`). |
| `peers` | Lista över slavar som upptäckts via Delay_Req (kameror + DAQ-E-sensorer). |
| `cameras` | PTP-status per kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Starta om grandmaster-processen. |
| `set-priority --priority1 N --priority2 N` | Åsidosätt BMCA-prioriteringar. |

Grandmastern sänder ut meddelanden på kamerornas nätverkskort (vilket nätverkskort som helst med en `169.254.x.x`-adress inkluderad). Om inget är uppkopplat faller den tillbaka till nätverkskortet för standardvägen endast när det är anslutet via kabel, aldrig via Wi-Fi eller ett virtuellt nätverkskort (Tailscale, WSL), eftersom ingen kamera eller DAQ-E som sitter bakom ett sådant. Om det inte finns något att tillkännage körs den utan socklar (`health` `NO_SOCKETS`), och nästa kamera eller DAQ-anslutning återuppbygger den på det nätverkskort som då är aktivt; `time-sync restart` gör det direkt.

Grandmaster körs i en egen process, vid sidan av backend (`own_process` / `process_pid` i `time-sync status`), med hög prioritet i Windows (`process_priority`), så att en upptagen backend (en bearbetningskörning, en export, flera kommandon samtidigt) inte kan fördröja de tidsstämplar den sätter; tidigare kunde en sådan fördröjning flytta alla kamerors klockor samtidigt. Den processen avslutas med backend; om den inte kan startas körs grandmastern inuti backend som tidigare (`own_process` false, och backend-loggen anger detta).

### Exempel

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE-kamerastyrning. Varje underkommando dirigeras via Chloros-backend; backend äger kamerapoolen så att efterföljande CLI-anrop återanvänder samma öppna handtag.

### Vanliga alternativ (gemensamma för de flesta underkommandon)

| Flagga | Beskrivning |
| --- | --- |
| `-d, --device N` | Kameraindex (standard: 0). |
| `-s, --serial SN` | Specifikt serienummer; åsidosätter `--device`. |
| `--serials SN1,SN2,…` | Serienummer separerade med kommatecken för drift med flera kameror. De [tillfälliga kommandona för flera kameror](#multi-camera-transient-sessions) öppnar endast dessa kameror, varav den första utlöser avtryckaren. |
| `--all` | Används på alla upptäckta kameror. |
| `--exposure US` | Exponeringstid i mikrosekunder. |
| `--gain DB` | Förstärkning i dB. |
| `--pixel-format FMT` | t.ex. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` på en M3M. Namnet anger bitdjupet inom kamerans egen familj: på en M3M anger ett Bayer-namn monoförmatet för dess djup (`BayerRG8` anger `Mono8`), på en M3C anger ett mononamn Bayer-namnet (`Mono12` ställer in `BayerRG12`). |
| `--width N` / `--height N` | Bilddimensioner. |
| `--preset {default,high_quality,high_speed,triggered}` | Tillämpa en förinställd inställning: dess pixelformat, exponering, förstärkning och bildfrekvens (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` listar dem). På en M3M är formatet Mono med samma djup: `high_speed` spelar in `Mono8`, `high_quality` och `triggered` `Mono12`. En flagga som medföljer (`--exposure`, `--gain`, `--pixel-format`) har företräde framför förinställningen, och en inställning som inte nämns i förinställningen behåller kamerans egen inställning. Alla fria-körning förutom `triggered`, som aktiverar kameran för en hårdvarukant på linje 2 — om inget driver den linjen kommer den att vänta för evigt istället för att ta en bild. |
| `-o, --output DIR` | Utmatningskatalog (standard: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-paketstorlek. `auto` kör ICMP+GVSP-sonder; `jumbo` = 9000; `standard` = 1500. |

### Arbetsflöde för första anslutning av LATTICE-kamera

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referens för underkommandon

#### Upptäckt och information

| Underkommando | Syfte |
| --- | --- |
| `lattice info` | Visa en lista över anslutna kameror (tillverkare, modell, serienummer, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analysera värdsystemet för optimal kamerakonfiguration. `--no-discover` hoppar över kameraupptäckten (snabbare, analys endast av nätverkskort). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Kontrollera/korrigera nätverkskortets inställningar (på Linux även kärnans mottagningsbuffertar för socklar — se [Linux-värdar](#linux-hosts--socket-receive-buffers)); uppskatta bandbredd/FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Stable-schema-backend, nätverks-kapacitet + rekommendation av array (returnerar `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` används som standard för hela sensorn, `2048` × `1536`. Den avslutas med `0` för `ok`, `auto_capped_fps` och `auto_shrunk`, samt `1` för `needs_force_slip` och `error`, med eller utan `--json`. Den når backend vid `--backend-url` (standard `http://127.0.0.1:5000`), inte `CHLOROS_BACKEND_URL`. `auto_capped_fps` behåller den begärda upplösningen men begränsar mål-fps – läs `recommended.recommended_target_fps` och skicka det som anslutningsmål; betrakta det som en framgång, inte som ett fel. `auto_shrunk` visas endast när den begärda bilden inte kan levereras med någon bildfrekvens; connect minskar aldrig bildstorleken på egen hand. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Vad-händer-om-analys utan att öppna kameror (`--width` / `--height` standard `2048` × `1536`). Både detta och `network-analysis` visar vad connect kommer att strömma: utan `--pixel-format`, gäller de format per kamera som Connect tillämpar (Mono12 / BayerRG12), och med `--packed auto` (standard) det [packade 12-bitars](#packed-12-bit-transport---packed) som Connect skulle fatta på den här datorn — så den angivna bildfrekvensen (fps) är den hastighet som matrisen körs med. Båda visar packningsvalet. **`--n-active` är det totala antalet kameror på nätverket, inte bara den här matrisens**— öka det när fristående kameror strömmar samtidigt, eller om nätverkskapaciteten beräknas utifrån en efterfrågan som underskattar antalet (standard: `len(--models)`). Visar alltid de sammanlagda raderna `Wire budget:` (krävd MB/s jämfört med kollisionssäkert tak) och `Max cameras:`, samt markerar `** OVER-SUBSCRIBED**` när arrayen överbelastar nätverket — se [Array fps & burst-modell](#array-fps--burst-model). |
| `lattice gpu` | Visa GPU-status. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Kontrollera eller uppdatera kamerans firmware. Det lokala valet av `.fwa` är låst: filen i `firmware/<MODEL_PREFIX>/` som matchar byggets `MIN_FIRMWARE_VERSION` flashas när den finns (endast den högsta versionen som reserv), så en nyare leverantörsbild som lagrats på disken är inaktiv tills den fästningen ändras — avsiktligt når nyare utgåvor enheterna via det signerade AWS-manifestet, vilket föredras när det är nyare. |
| `lattice presets [--apply NAME]` | Visa eller tillämpa kameraförinställningar. `--apply` anger vad förinställningarna heter (deras pixelformat, exponering, förstärkning, bildfrekvens, utlösare) och visar dessa inställningar så som kameran läser av dem efter: på en M3M, förinställningens djup i Mono-format; den bildhastighet den körs med, vilken kan vara lägre än vad förinställningen kräver (`high_speed` kräver 60 fps; en fullformatskamera på en 1 Gbps-länk körs med cirka 35). Alla övriga inställningar som dess öppna fil skrev återförs så som de hittades. `--apply default` återställer kameran till fabriksinställningarna: dess fabriksinställningar med fritt löpande automatisk exponering på Line2, länkens genomströmningsgräns och felkorrigering avstängda, en RGB-kameras kalibrerade vitbalans bevarad (enhet på alla andra). Ingenting sparas i kamerans minne, så vid nästa uppstart startar den från sitt eget minne som tidigare. Som `--preset` vid en inspelning eller anslutning finns det ingen förinställning för `default`. |
| `lattice status` | Kamerans status i realtid. |

#### Inspelning

| Underkommando | Syfte |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Enstaka bildruta. **Sparar alla exporttyper som standard**(`--processing all`); se [Exportnivåer för inspelning](#capture-export-levels-the-all-default). `--processing` tar emot `all`, `raw`, `debayered`, `radiance`, `reflectance` eller `display` (`preview` accepteras för `display`); alla andra namn avvisas innan kameran vidrörs (avsluta `1`). `--levels` sparar en explicit delmängd (åsidosätter `--processing`). En kamera som Chloros inte innehar öppnas för inspelningen, och dess exponering ställs in utifrån scenen först (högst några sekunder), precis som [tillfälliga kommandon](#multi-camera-transient-sessions) gör; när inspelningen avslutas, oavsett hur den avslutas, återställs den kameran till det tillstånd den befann sig i (format, exponering, färgpipeline, alla inställningar som ändrades under inspelningen — listan under [Flerkamerasystem (tillfälliga sessioner)](#multi-camera-transient-sessions)). Den DAQ som läser in den använda inspelningen sparas som en `.daq`-sidecar, inklusive en rådata-utdrag; `--no-daq` / `--daq-tolerance-ms` justerar den ([DAQ:en som läser in en inspelning sparar](#the-daq-reading-a-capture-saves)). `--force-daq` accepteras fortfarande. `--jpeg-quality` = JPEG-kvalitet 1–100 (standard 95).**Filer som sparas i det öppna projektet läggs till i det.** Precis som i appen läggs en inspelning vars filer hamnar i mappen för det projekt som är öppet i Chloros till i det projektets fillista; filer som sparas någon annanstans lämnas orörda. Kommandot visar en rad som anger detta (`Project: ...`), eller, om en bearbetningsomgång pågår, att de hålls tillbaka och läggs till när omgången avslutas. Detsamma gäller `multi-capture`, `array-capture` (`--smart` också) och `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Strömma till disk tills Ctrl+C. En körning av `tiff`består av den `raw`-fil som en array-inspelning skriver från den kameran: sensorns mosaik, ett plan med sina egna värden, okomprimerat, med kalibreringspaketet (en 2048 x 1536 12-bitars bildruta är 6,0 MiB plus paketet, oavsett motiv), så `process` utgår från dessa. `png` och `jpg` innehåller 8 bitar, så dessa serier är demosaicerade bilder för visning, vilket `process` inte utgår från. Med en DAQ-bunden (`daq pool-bind`) eller i en pool får varje bildruta sina egna `.daq` och `.match.json` i seriens `daq/`-mapp, eftersom `lattice capture` sparar en; `--no-daq` stänger av den funktionen, och vid stopp visas hur många. Varje bildruta skrivs ut en gång, med fullständig metadata; vid Ctrl+C skrivs de redan hämtade bildrutorna ut innan kameran släpps. Avbryter sig själv när enheten fylls, skriver ut `Failed:` / `Disk full:` och avslutas med `1`; varje bildruta som inte kunde skrivas ut avslutas också med `1`. En kamera som lämnats aktiverad för en hårdvarutrigger (`TriggerMode` `On`) ställs in på friläge för körningen. När körningen avslutas återställs kameran till det tillstånd den befann sig i – dess utlösarinställningar och alla inställningar som ändrades under körningen (exponering, förstärkning, format, …) — såvida inte `--keep-settings`, som sparar körningens inställningar i kamerans eget minne (fritt löpande), eller `--clear-settings`, som återställer det minnet till fabriksinställningarna. En körning som inte spelar in någon bildruta är ett misslyckande: om ingen bildruta registreras under 10 sekunder avbryter den sig själv, skriver ut `No frames:` med orsaken och avslutas med `1`. Bilder som skrivarna inte kunde hinna med räknas som borttappade, och avbrottet skriver ut `Dropped:` med orsaken (kamerans bildfrekvens, skrivarnas och kön); en lägre bildfrekvens eller ett 8-bitars pixelformat behåller dem alla. Kommandot rapporterar in till backend varannan sekund; om det avbryts utan att inspelningen stoppas (`kill -9`, minne fullt), avbryter backend den 30 sekunder efter den sista inrapporteringen och återställer kameran (`stop_reason` `client_gone`, samt en rad i backend-loggen). `multi-stream` och `align-stream` gör på samma sätt. En start där det tar längre tid för kameran att öppnas än vad kommandot väntar (60 sekunder; 180 sekunder för `multi-stream` / `align-stream`) — en kamera som fortfarande startar upp — ångras av backend: kameran återställs och stängs och ingen inspelning lämnas igång (504 HTTP-fel `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Webbläsarbaserad liveförhandsvisning i MJPEG: ansluter kameran, visar strömens URL och öppnar den i din webbläsare. `--ae-damping` ställer in dämpning av automatisk exponering (0,4–100). |

##### Livestreamar (MJPEG; inget RTSP)

Chloros levererar livevideo som **MJPEG över HTTP**(`multipart/x-mixed-replace`, gräns `frame`) från backend.**RTSP stöds inte.** Öppna en URL-adress i en webbläsare, eller läs den med valfri MJPEG-klient (OpenCV:s `cv2.VideoCapture(url)`, ffmpeg, VLC). Kameran måste först anslutas i backend (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` eller via GUI).

| URL (`http://127.0.0.1:5000` + …) | Ström |
| --- | --- |
| `/api/camera/stream/<serial>` | Förhandsgranskning av bild, upp till cirka 30 fps. För en arraymedlem, ställer `?max_h=` (standard `720`) och `?jpeg_q=` (standard `80`) in höjden och JPEG-kvaliteten. Detta är den URL som `lattice viewer` öppnar. |
| `/api/camera/<serial>/stream-annotated` | Samma förhandsgranskning med GUI:ns överlägg ritade in: `zebra=1`, `crosshair=1`, `grid=3x3` (eller `9x9`), `histogram=1`, `peaking=1`, `spot=1`; utan förfrågan använder projektet de sparade överlagringsinställningarna för den kameran. |
| `/api/camera/<serial>/stream-reflectance` | 8-bitars reflektans per band: ρ = pixel ÷ den del`X-Chloros-Pixel-Scale`-huvud (255 om inte `refl_max` är aktiverat). `fps` (standard `10`), `jpeg_q` (standard `95`), `max_h` (standard `0` = native), `band=<name>`, `align=1`. JPEG, alltså med förlust. Avvisas med HTTP-kod 409 när kameran saknar reflektans (en RGB-kamera, inget kalibreringspaket, ingen DAQ-avläsning). |

- **Matchning av bildrutor mellan kameror.**Varje del som en**array-medlem** skickar, på alla tre strömmarna, bär `X-Chloros-Group-Id` (arrayens synkroniserade gruppräknare), `X-Chloros-Frame-Id` och `X-Chloros-Timestamp-Ns`. Delar från olika kameror med samma `X-Chloros-Group-Id` kommer från samma utlösare. En fristående kamera har ingen grupp och skickar inget grupp-ID. Webbläsare ignorerar rubriker per del; en klient som själv analyserar den flerdelade strömmen läser dem.
- **Pausad medan arrayvyn är öppen.** `/api/camera/stream/<serial>` för en arraymedlem skickar en statisk ”pausad”-ram medan arrayens kombinerade livestream är öppen (till exempel GUI:ns arrayvy) och återupptas när den stängs.

#### Sensorkalibrering

| Underkommando | Syfte |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Läser/skriver till valfri GenICam-nod. `--set ExposureTime=…` (`Gain=…`) utan dess `ExposureAuto` (`GainAuto`)-switchar som först stänger av autoläget, som `lattice exposure --set` gör; ett värde som kameran vägrar att acceptera skriver ut `Failed to set <node>: …` på stderr och avslutar `1`. `--get` och den standardgrupperade vyn skriver ut ett `note:` under `ExposureTime` / `Gain` medan det automatiska läget körs — se [Avläsning av exponering och förstärkning i automatiskt läge](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Exponering & AE. En `--set`-exponering eller ett `--brightness`-mål behålls som ditt (Chloros automatiska exponeringskontroll lämnar det ifred); förstärkningen förblir oförändrad, så i autoläge fortsätter den att justeras mot målet tills du ställer in `lattice gain`. `--auto` återlämnar exponeringen, `--brightness-auto` återlämnar målet till Chloros automatiska mål. Utan flaggor läser den av exponeringsnoderna, med samma `note:` som `configure --get`. Om kameran vägrar ett `--set`-värde skrivs `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` ut på stderr — i automatläget återställs skrivfunktionen när den stängts av, istället för att förbli avstängd — skriver inte ut någon `Exposure: … us (auto OFF)`-rad och avslutar `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Förstärkning och automatisk förstärkning. Ett `--set`-värde som kameran avvisar skriver ut `Failed to set Gain: …` på stderr, ingen `Gain: … dB (auto OFF)`-rad, och avslutar `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Sensor-ROI och binning. Binning är alltid **Genomsnitt** — se nedan. |
| `lattice format [--set FMT] [--list]` | Pixelformat. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hårdvaru-/mjukvarutrigger. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (inga flaggor = engångsvitbalans) | Vitbalansoperationer. Endast RGB-/Bayer-kameror; en no-op (hoppas över) på mono M3M. Engångsläget behåller den vitbalans som det beräknar (`BalanceWhiteAuto` och förstärkningarna); alla andra inställningar som dess öppna skrivning avser återställs till ursprungliga värden, och alla återställs när engångsläget inte kan köras. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Färgbehandlingskedja för RGB-visning. `natural` (standard) är den billiga live-bearbetningen; `enhanced` lägger till defringe + vibrance + CLAHE lokal kontrast för full hub-parity-look till ungefär dubbla bearbetningskostnaden per bildruta, vilket ger en lägre **live**-bildfrekvens — sparade bilder får alltid full bearbetning oavsett. Endast RGB-/Bayer-kameror; hoppas över på mono M3M. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Mättnad/kontrast i liveförhandsgranskning (kameror med RGB-filter); `100` är neutralt, `--reset` ställer in båda på 100. Sparade bilder påverkas inte. Hoppas över på mono-M3M. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (inga flaggor = läs) | Visa **Högdagrar** (kameror med RGB-filter): dra ner de ljusaste, vitaste pixlarna — t.ex. en ljusvit yta som återges som en platt vit klump. `0` = av, `-100` = starkast. Liveförhandsvisningen och bilder på skärmnivå följer detta; strålning/reflektans gör det aldrig, och det går inte att återställa sensordata som har klippts (sänk exponeringen för det). Hoppas över på mono M3M. |
| `lattice filter [--list]` | Lista de filtermodeller som Chloros känner till (`RGN-IMX265`, `OCN`, `NGB`, …). En kameras filter läses av från dess modellsträng och kan inte ställas in: `--set NAME` söker endast upp en modell och avslutar sedan `1` med `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Undersök strömförsörjnings- och termiska noder; växla till viloläge med låg strömförbrukning. `--sleep` lämnar kameran i viloläge (dess gräns för länkgenomströmning aktiverad (`DeviceLinkThroughputLimitMode` `On`) på sin lägsta nivå, energisparläget aktiverat där kameran har ett sådant) med alla övriga inställningar återställs dess öppna skrivning till det ursprungliga läget; en viloläge som misslyckas återställs i sin helhet. Uppvaknandet återställer länken till det tillstånd som fanns vid övergången till viloläge (gränsen, dess läge och paketfördröjningen `GevSCPD`, som kameran håller på 0 medan gränsen är aktiverad; uppgifterna sparas på den här datorn, så att en uppvakning efter att backend-servern startats om också hittar den), eller, för en kamera som denna dator inte har satt i viloläge men som verkar vara i viloläge (dess gräns är aktiverad vid lägsta nivå), med gränsen avstängd och en `GevSCPD` kvar på 0 vid 80 (fabriksinställningarna); en gräns som är inställd över lägsta gränsvärdet lämnas som den är. Sonden och uppvakningen återställer det som deras öppning skrev. |

> **Binning är alltid Genomsnitt.** `lattice resolution --binning N` ställer in Genomsnittlig binning och erbjuder
> inte någon lägesomkoppling. Sum-binning multiplicerar varje DN med N², vilket kostar ungefär två bländarsteg av
> högdagrarsutrymme per binningssteg; Chloros delar ut den faktorn igen före den
> radiometriska kedjan, men pixlar som Sum drivit upp till sensorns tak klipps bort och kan inte
> återställas. Det ger inga fördelar som ”Average” inte ger – ett index är skalinvariant och strålningsvärdena
> är kalibrerade enligt ”Average”-skalan. Om en kamera redan är inställd på ”Sum” (efter att
> ha ställts in av ett annat verktyg eller genom en fabriksåterställning), upptäcker Chloros detta genom en nodavläsning och
> kompenserar; genom att köra `lattice resolution --binning N` återställs den till Average. I version 1.2.0
> `--binning-mode`-flaggan accepteras fortfarande så att äldre skript kan köras, men det ger endast en varning:
> binning förblir på ”Average”.

> **Ändringar av binning, pixelformat och utlösare på en kamera har ingen effekt.** På en kamera där
> backend hålls öppen och strömmar (`cam-connect`, GUI:n), träder `lattice resolution --binning`
> träder inte i kraft (det visar att binning fortfarande gäller, `Binning: 1x1`), och
> `lattice format --set` och `lattice trigger --mode` avvisas av kameran
> (`SC_ERR_ERROR -1001`). Släpp den först (`lattice cam-disconnect -s <SN>`), eller ställ in
> dem vid anslutning (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Avläsning av exponering och förstärkning i automatiskt läge

Medan en kameras `ExposureAuto` (`GainAuto`) körs behåller dess `ExposureTime` (`Gain`) kvarhåller det **senast skrivna värdet**, inte vad kameran gör just nu. För en kamera som backend håller öppen (`cam-connect`, `array-connect`, GUI), visar `lattice configure --get`, standardvyn `lattice configure` och `lattice exposure` (inga flaggor) därför nodvärdet och, under det, en anteckning med kamerans senaste bildruta:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Läs bildvärdet från anteckningen; nodvärdet är endast den senaste skrivningen. Via HTTP behåller samma `POST /api/camera/<serial>/read-nodes`-svar `nodes` oförändrat och lägger till `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, den senaste bildrutan) och `notes` (meningen ovan). Båda finns endast när autoläget inte är `Off` och en bildruta är känd; en kamera som öppnats enbart för läsning har ingen bildruta och får ingen av dem. `lattice gain` (inga flaggor) skriver endast ut noden — använd `lattice configure --get Gain` för anteckningen.

##### Exponering på en arraymedlem

Medan en array strömmar körs varje medlem med **`AcquisitionFrameRateEnable`**, med en bildfrekvens som ligger något över matrisens utlösningsfrekvens: dess period är utlösningsperioden minus 15 % (minst 3 ms), eller kamerans egen maximala frekvens när utlösningen redan ligger nära den. `configure --get` på en medlem avläses därför som `AcquisitionFrameRateEnable` vid den inställningen; aktiveringen stängs av igen när matrisen stannar.**Skriv inte dessa två noder på en arraymedlem** — arrayen håller dem synkroniserade med sin utlösare och skriver om dem.

Den inställningen är också medlemmens exponeringsgräns: cirka 85 ms vid en utlösare på 10 fps (100 ms period), vid **vilken ROI eller binning som helst**. Före version 1.3.0 begränsade en liten ROI istället en medlems exponering till sensorns frilöpningsperiod (4,9 ms vid 768 × 304), oavsett hur långsamt matrisen triggades.**En manuell `ExposureTime` som är längre än gränsvärdet saktar ner utlösningen för att anpassa den** när den skrivs med `lattice exposure --set` eller `configure --set` (eller från GUI). Matrisen öppnar först den medlemmens övre gräns för värdet och låter den långsammare hastigheten köras under några utlösningsperioder, så att skrivningen kan ta några sekunder längre; utlösaren avfyras sedan inte snabbare än vad exponeringen tillåter. På en matris med automatisk hastighet återgår den tidigare hastigheten så snart varje medlem vars manuella exponering saktade ner den är tillbaka på automatisk exponering. Ett värde som kameran själv inte kan ta emot avvisas fortfarande, med det intervall den accepterar angivet i felmeddelandet, och medlemmens automatiska exponering återställs om den endast hade stängts av för just den skrivningen. En exponering enligt receptet `apply` saktar inte ner utlösaren: den hålls vid takvärdet. Chloros förkortar aldrig en manuell exponering för att passa en snabbare utlösare; en fastställd hastighet (`array-connect --target-fps F`) som är snabbare än vad en exponering tillåter ger den medlemmen ett tak som är tillräckligt långt för den, och den medlemmen tar inte emot utlösare snabbare än vad dess exponering tillåter.

#### Kalibrering & sensorer

| Underkommando | Syfte |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrera utifrån ett reflektansmål. Upp till `--attempts` ramar (standard `30`) genomsöks efter det; en ofullständig ram (GVSP-paketförlust) kostar ett försök, inte hela körningen. Det misslyckas med `no target detected in N attempts`, eller vid `no complete frame in N attempts: <last error>` när ingen ram anlände i sin helhet. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | En DAQ-M:s spektrum och bandamplituder, avlästa via Bluetooth (BLE) genom backend. `--mac` är sensorns egen BLE-MAC-adress och krävs för någon av de tre åtgärderna: det finns inget standardvärde, eftersom varje enhet har sin egen (`daq pool-discover --only ble` listar dem); utan den avslutas kommandot `1`. Om ingen åtgärd anges visas hjälpinformationen. `--camera SERIAL` integrerar de utskrivna bandamplituderna genom kamerans egen uppmätta spektrala respons istället för filtermodellens fleet-kurva — RGB-kameror, med kamerans kalibreringspaket laddat (anslutet) eller cachat på denna dator. Rubriken anger vems kurva som användes: `Band magnitudes (camera <SERIAL>):`, eller `Band magnitudes (model <filter>):` när kameran saknar sådan (inte en RGB-kamera, eller inget paket laddat eller cachat) och standardkurvan användes istället. Via HTTP tar `POST /api/dls/cli-read` emot `camera_serial` och returnerar `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Tillämpa vignettkorrigering på befintliga bilder, var och en med sitt eget objektivs flatfält (från dess EXIF-modell). `--lens-model` tillämpas på bilder vars modell inte anger någon kamera; en bild vars modell strider mot detta hoppas över. |

#### Flera kameror (tillfälliga sessioner)

Ett tillfälligt kommando öppnar kamerorna för en körning och stänger dem när den avslutas: `multi-capture` utan någon permanent matris ansluten, `multi-stream` och `multi-test` här, och `align-calibrate`, `align-apply`, `align-stream` och `index --live` nedan. Dessa regler gäller för alla:

- **`--serials SN1,SN2,…` väljer kamerorna.** Endast dessa öppnas, och den första utlöser (för `align-calibrate` och `--reference` när de anges). En serienummer som inte finns i nätverket gör att körningen misslyckas innan någon kamera öppnas. Utan `--serials` öppnas alla kameror i nätverket. En kamera som Chloros redan har öppnad (via GUI, `cam-connect`, `array-connect`) avvisar körningen oavsett om `--serials` namnger den eller inte: den delar det nätverk som körningen är avsedd för. (`index --live` utan `--serials` öppnar istället de aktiverade kamerorna i sin profil: se [Index / Vegetationsmatematik](#index--vegetation-maths).)
- **Kamerorna delar nätverket på samma sätt som `array-connect` delar det.** Varje kamera anpassas till sin andel av vad värdlänken överför (en fördröjning mellan paket, inte kamerans genomströmningsgräns, vars lägsta gräns på 31,25 MB/s redan överskrids av fyra kameror på 1 GbE), så fyra kameror körs på en 1 Gbps-länk. När kamerorna inte alla kan sända samtidigt i den långsammaste takt de når, aktiveras de en efter en, var och en med länken till sig själv, och kommandot visar `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. När inte ens en kamera åt gången får plats avvisas körningen innan någon kamera hinner strömma: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP-kod 409, avslutas med `1`.
- **Alla kameror måste leverera, annars misslyckas körningen.** En kamera som inte levererar någon bildruta är ett fel: kommandot visar `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, där `<why>` är den kamerans hämtningsfel, följt av vad orsaken tyder på, och avslutas med `1`. En bildram som anlände med GVSP-paketförlust (`IncompleteBufferError: GVSP packet loss …`) innebär att nätverket inte klarar av kamerornas sammanlagda bandbredd (`Every missing frame arrived with GVSP packet loss: …`): kontrollera länkhastigheten för värden och för switchens upplänk (`lattice network`), använd jumbo-ramar, eller öppna färre kameror med `--serials`. En kamera som inte levererade något får vanligtvis inte triggern från mastern: kontrollera dess M8-synkroniseringskabel och kör sedan `lattice multi-detect`. Vad varje kommando gör med en ramgrupp där en kamera saknas framgår av dess rad.
- **Varje kamera återställs till det tillstånd den befann sig i.** När körningen avslutas återfår varje kamera det tillstånd den hade när körningen startade: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, samt `LineMode` / `LineSource` på `Line2` och `Line3` (och deglitch-signalen från trigger-ingången i en matris ger en slavlinje, `LineFilterSelector` / `LineFilterWidth`); dess `PtpEnable` (och uppsättningen `PtpSlaveOnly` / `PtpDomainNumber` som hör till den); `ExposureAuto` / `GainAuto` (samt `ExposureTime` / `Gain` där det var inställt på ett manuellt värde); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (samt varje `BalanceRatio` där vitbalansen var manuell); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (och `Gamma` där gamma var aktiverat), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; paketstorleken (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); takten, `DeviceLinkThroughputLimitMode` (och `DeviceLinkThroughputLimit` där läget var `On`), fördröjningen mellanfördröjningen mellan paketen `GevSCPD` och startförskjutningen `GevSCFTD` som en matris ger varje kamera; blockdata som varje ström aktiverar (`ChunkModeActive`, varje `ChunkEnable` under sin `ChunkSelector`); `AcquisitionFrameRateEnable` (och `AcquisitionFrameRate` där aktiveringen var på — när den är avstängd är hastigheten skrivskyddad och läser ut så mycket som takten tillåter, så det är ingen inställning). Endast det som skiljer sig åt skrivs, kameran stannade först, takten och bildfrekvensen före en exponering som kräver sin bildperiod (en kamera begränsar `ExposureTime` till en bildperiod). Med `--keep-settings` kopplas utlösaren och synkroniseringsledningen bort, kameran körs fritt och resten sparas i kamerans eget minne enligt inställningarna för körningen; `--clear-settings` återställer fabriksinställningarna där.
- **Exponeringen ställs in utifrån scenen innan den första bilden sparas.** Körningen tar synkroniserade bildgrupper och kasserar dem samtidigt som den ställer in varje kameras exponering och förstärkning utifrån dessa bilder. Målet är en medelvärdesljusstyrka på kamerans mål utan att mer än 1 % av bilden klipps bort; en scen med starka högdagrar når inte upp till målet. Detta tar vanligtvis 2–6 bildgrupper och högst 5 s.
  - Varje kamera riktas mot det mål som en ansluten matris startar den med, enligt dess filter: TargetBrightness **40**på en multispektral kamera (RGN, OCN, NGB) och**60** på en RGB-kamera. Det är hälften av det exponeringsmål som 80 ger (tre fjärdedelar på RGB), så det är mindre sannolikt att en ljus reflektanspanel klipps. `--brightness N` (0–255) ställer istället in varje kameras mål (`multi-test` har inget `--brightness`: dess automatiska exponering är avstängd). Backendens logg namnger varje kameras mål: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Exponeringen kommer först, upp till kamerans egen gräns för automatisk exponering. Förstärkningen täcker endast det som återstår.
  - Den fastställda exponeringen och förstärkningen behålls under hela körningen, så varje bildruta i en `multi-stream` tas med samma exponering. En bildtagning som måste följa förändrade ljusförhållanden hör hemma i en ansluten matris (`array-connect`), vars automatiska exponering kontinuerligt justeras.
  - `--exposure` och `--gain` behålls exakt som angivna. Om båda anges kasseras ingen bildruta.
  - När körningen avslutas slås varje kameras automatiska exponering på igen.
  - Backendens logg rapporterar resultatet: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …`, eller `exposure NOT solved after …` med orsaken och `(not settled)` för varje kamera som inte var med. Bilderna tas fortfarande i det fallet.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` ändrar gränsen på 5 sekunder; `0` stänger av denna funktion (se [Miljövariabler](#environment-variables)).

**Så länge Chloroshåller någon kamera** (GUI:n, `cam-connect` eller en ansluten matris), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` och `index --live` vägrar att köras, eftersom de öppnar alla kameror i nätverket och en andra öppning av en upptagen kamera kan krascha backend-systemet. (`index --live` avbryter när en kamera som den skulle öppna är upptagen: dess `--serials` eller kamerorna i dess profil, samt alla kameror för en profil som inte anger någon.) Backend-systemet svarar med HTTP-kod 409 `CAMERA_HELD` och anger varje upptagen kamera samt frigörandekommandona (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), varefter kommandot avslutas med `1`. Om en matris är ansluten spelar `array-capture --continuous` in den i stället för `multi-stream`, och `array-capture` (justerad som standard) sparar en justerad uppsättning i stället för `align-apply`; `multi-capture` och `align-calibrate` använder en ansluten matris som innehåller deras kameror istället för att öppna dem.

| Underkommando | Syfte |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Visar alla kameror med synkroniseringsroller; med `--serials` endast dessa, den första som `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | En synkroniserad bildruta från varje kamera. Sparar **alla exporttyper som standard**när en permanent matris är ansluten, samt den DAQ-avläsning som användes ([Den DAQ-avläsning som en inspelning sparar](#the-daq-reading-a-capture-saves)); den tillfälliga reservlösningen utan matris är**endast debayered** (kör `array-connect` först för resten). `--brightness` gäller endast fallback-läget: en ansluten matris behåller sin egen automatiska exponering. Fallback-läget sparar inte `.daq` (`--no-daq` / `--daq-tolerance-ms` har ingen effekt där), och `multi-capture` tar inte `--force-daq`. I fallback-läget misslyckas en kamera som inte levererar någon bildruta med inspelningen (`Error: multi-capture failed: no frame from …`, avsluta `1`) och ingen fil från den bildrutegruppen sparas. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Strömma synkroniserade bildrutor (tillfälligt), `--fps` standard `3`, tills Ctrl+C eller `--count`-grupper. En bildgrupp där en kamera inte levererade något skrivs inte ut, och tre i rad avslutar strömmen. Om någon grupp hoppades över avslutas körningen genom att skriva ut `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, plus `Error: the stream stopped: …` med namnet på kameran när den avslutade strömmen, och avslutar `1`X. Avbryter sig själv när enheten blir full, skriver ut `Failed:` / `Disk full:` och avslutas med `1`; alla filer som den inte kunde skriva till avslutar också `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | Test av GPIO-synkroniseringstid (övergående), `--count` standardutlösare `10`. Visar hur nätverket delas (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, och raden `Serial emit:` när kamerorna utlöses en efter en, vars tidsförskjutning då ingår i tidsstämplarna), en `Sync group: master SN:<serial> -> SN:<serial>, …`-rad per synkroniseringsgrupp (`(no slaves: not synced)` för en kamera utan synkroniseringskabel, som utlöses på egen hand), sedan en rad per utlösning: `[  0] SN:<serial>=<timestamp> \| …`, med `SN:<serial>=(no frame)` för en kamera som inte levererade något för den utlösningen. Sedan en rad `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` och en rad `SN:<serial>: mean <±x> us, jitter <y> us` per kamera, varje kamera i förhållande till mastern inom en klockcykel: en utlösning vars tidsstämplar blandar kamerornas enhetsklockor och PTP (PTP avstängd eller fortfarande låser) utelämnas, med en `Note:` som räknar dem. Varje utebliven bildruta innebär att testet misslyckas (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, avsluta `1`); tre ofullständiga triggare i rad avslutar det i förtid. Det misslyckas också (avsluta `1`) när färre än två triggare har alla kameror på samma klocka (`… the clocks cannot be compared`), eller när klockorna aldrig stämmer överens: PTP-klockor som avviker mer än 1 ms (plus eventuell förskjutning på seriell nivå) från mastern (`… PTP is not locked`), eller enhetsklockor vars avvikelse förändras mer än så mellan utlösningarna. Den startar PTP-grandmastern om den inte är igång, aktiverar PTP för varje kamera (precis som `array-connect` gör, oavsett hur nätverket ser ut) och väntar, innan den första triggern, på att varje kameras PTP-klocka ska låsa sig till grandmastern och stabilisera sig: `PtpStatus` Slave på en grandmaster, inom 100 us från den och stabil i 3 s, och varje kamera låst på detta sätt, utan avbrott, i 120 s — cirka 2 minuter från ström. En klocka som just har låst sig avviker med upp till ~200 us i ungefär 2 minuter efter att den läser Slave, och kamerans egen `PtpOffsetFromMaster` visar inte detta; 120 s bringar kamerorna inom ~10 us från en stabiliserad klocka. En kamera är fortfarande synkroniserad sedan ett tidigare test där PTP lämnades påslaget, enligt vad denna backend uppfattade, räknar från den låsningen, så ett test direkt efter ett annat väntar inte igen; en kamera som testet slår på, eller en som hittas med PTP aktiverat men som denna backend inte har sett låsa sig (som slagits på någon annanstans, eller före en omstart av backenden), väntar sina 120 s, och i en uppsättning av båda kamerorna väntar de som just låst sig fortfarande på sin tur. En kamera som lämnar slavläget, överskrider 100 us eller följer en annan grandmaster under 3 avläsningar i rad (1,5 s) under väntetiden börjar räkna om; en eller två sådana avläsningar loggas (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) och räkningen fortsätter, och ett grandmaster-ID som inte kan läsas betraktas som okänt, inte som en förändring. Om alla kamerors räkning startar om samtidigt (inom en sekund) beror det på att grandmasterns tid går vidare, inte kamerornas: backendens logg visar `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` listar det i `grandmaster_steps`, och en väntetid som sedan löper ut inleder sitt fel med `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Gränsen förlängs inte för detta. `--ptp-settle SECONDS` ställer in avväntningstiden för körningen (0–600; `0` för ingen avväntning). Väntetiden är begränsad till stabiliseringstiden plus 90 s (210 s som standard, minst 90 s), och kommandot väntar på backend i 240 s plus stabiliseringstiden (360 s som standard). Det skriver ut `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, och backendens logg innehåller en rad var 5:e sekund med varje kameras status och offset, hur länge den har varit låst och hur lång tid som återstår: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Varje kamera startas med PTP avstängt; därefter låser sig klockorna inom cirka 10 s. Därefter återställs varje kamera till det tillstånd som testet fann den i – dess PTP, den automatiska exponeringen och förstärkningen som testet håller avstängda under körningen, samt resten (om inte `--keep-settings`) — så en kamera som lämnats med PTP på (`lattice configure -s <serial> --set PtpEnable=True`, eller ett test med `--keep-settings`) förblir synkroniserad, och när denna backend har konstaterat att den varit synkroniserad under stabiliseringstiden väntar inte ett senare test på den. Om klockorna inte stabiliseras under väntetiden misslyckas den innan den utlöses (`PTP: NOT locked …`, avsluta `1`) och anger vad som ska göras: en kamera som aldrig låstes (`PtpStatus Listening`) innebär att grandmastern inte når den (`time-sync status`); en som hela tiden tappade anslutningen namnges efter hur länge den hade varit låst (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); klockor som fortfarande konvergerar: kontrollera `time-sync status`, och för att få dem låsta före testet, aktivera PTP och låt det vara på (`lattice configure -s <serial> --set PtpEnable=True` för varje kamera, eller kör testet med `--keep-settings`; testet återställer det som det hittade, så det förblir påslaget), kör sedan testet. |
| `lattice multi-detect [--line LINE] [--json]` | Hitta kamerorna på M8-synkroniseringslinjen och en standardmaster: RGB-kameran (modell som slutar på `-FRGB`) om den är ansluten via kabel, annars den första anslutna kameran i serieordning. M8-kablarna sammanstrålar vid en kopplingspunkt, så **vilken ansluten kamera som helst kan vara master** — den tryckta beteckningen `MASTER` är ett standardvärde, inte en faktisk kopplingsuppgift. Varje kamera driver linjen i tur och ordning och de andra kontrolleras för dess puls; en kamera som redan hittats i den anslutna gruppen undersöks inte igen, så en fullständigt ansluten uppsättning tar en omgång. Utan `--line` söker den efter `Line2`, sedan `Line3`. Skriver ut `MASTER: SN:…`, ett `SLAVE: SN:…` per övrig ansluten kamera och `(probed on <line>)`; `--json` lägger till det fullständiga resultatet (`master`, `slaves`, `no_cable` — kameror som pulsen inte nådde — `line` samt `results` per kamera). När processen är klar återställs varje kamera till de inställningar för utlösare och synkroniseringslinje som den hade tidigare (samma noder som ovan). |

#### Justering

| Underkommando | Syfte |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — plus reglage för detektor/matchare `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-reglage `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombination av flera bildrutor `[--averaging mean\|median\|inlier_weighted]`, geometriska begränsningar `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, rumslig begränsning `[--roi X0,Y0,X1,Y1] [--mask PATH]` ochslav-överskrivningar `[--per-cam-override SN:KEY=VALUE]` (upprepningsbar) | Beräkna inriktningsprofil från livekameror. Standardvärden: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (i den mapp du kör det från), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` har sina egna standardinställningar; se [dess tabell](#project-align-calibrate-options).) `--prefilter` har som standardinställning `gradient` (kantkarta; stämmer överens med GUI/array-inriktaren — kanterna bevaras över spektralbanden). `--matcher flann` ger bästa resultat vid över ~5000 drag; `--averaging median` är robust mot en felaktig avbildning, `inlier_weighted` viktar efter antal matchningar; `--lock-scale` projicerar till närmaste rotation (ingen skala), `--lock-axis` nollställer en translationskomponent; `--mask` gäller för alla kameror (använd `--per-cam-override` för inställningar per kamera, t.ex. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` vägrar att spara en kalibrering vars RMS-värde för omprojiceringen överskrider gränsvärdet. `--method manual` hämtar sina punkter från `--points PATH`: ett JSON-objekt `{"<serial>": [[x, y], ...], ...}` som anger varje kameras pixelkoordinater för samma objekt, inklusive referenskameran (minst 2 per kamera); var och en av `--frames` löses utifrån dessa punkter. När en ansluten array innehåller kamerorna använder kalibreringen den arrayens livebilder, och profilen blir också arrayens inriktning för dess bildtagningar. I annat fall öppnas en [tillfällig array](#multi-camera-transient-sessions): en kamera som inte levererar någon bildruta i någon av `--frames` misslyckas med kalibreringen (avsluta `1` och namnge den), och ingen profil sparas. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Ta en justerad multibandsbild. `--bit-depth` matchar som standard kameran; `--no-crop` behåller hela bilden (fyller med svart); `--interpolation` (standard `linear`) och `--border-mode`/`--border-value` (standard `constant`/0) styr CPU-förvrängningen — GPU-vägen är oavsett detta bilineär. Öppnar en [transient array](#multi-camera-transient-sessions): en kamera som inte levererar någon bildram misslyckas med inspelningen (avslutar `1` och namnger den), och ingenting skrivs. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Strömjusterade multibandsramar (samma warp-reglage som `align-apply`), `--fps` standard `3`. Öppnar en [transient array](#multi-camera-transient-sessions): en bildgrupp som en kamera inte levererade något till sparas inte, och tre i rad avslutar strömmen. Om någon grupp hoppades över avslutas körningen genom att `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` och `Error: …` skrivs ut, och avslutas med `1`. En ström som avslutas på grund av något annat fel skriver också ut `Error: …` och avslutas med `1`. |
| `lattice align-info --profile PATH [--json]` | Visa profilinformation. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Ändra lagerordning. |

#### Index / Vegetationsberäkningar

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` mappar en förinställnings symboler till band** (NDVI: `red`, `nir`), en `--channel SYMBOL=BAND` vardera.

- **`--live`: endast för en symbol som stapeln inte kan hantera.** Precis som med `--input` tar en symbol i en förinställning som du utelämnar det enda bandet i stapeln (banden för profilens aktiverade kameror) vars namn börjar med den. Detta kontrolleras innan någon kamera öppnas: en symbol som inget band, eller flera band, svarar på avvisas då, och genom att namnge dem avslutas `1`. En anpassad `--formula` kräver en `--channel` för varje symbol.
- **`--live`-produkter anger vem de tillhör.** `index.tif` och `index_LUT.png` innehåller stapelns kamera (märke, modell, serienummer), dess tagningstid (referenskamerans exponeringstid, UTC) samt objektiv, och båda anger indexet som sitt enda band (`XMP-Camera:BandName` `NDVI`; färg-PNG-filen anger indexet, inte dess färgkanaler), precis som en `--input`-produkt gör. Ett index från flera kamerors band namnger istället vart och ett av dem, och dess `Model` är `LATTICE` — se [Filer skapade från flera kameror](#files-made-from-several-cameras).
- **`--input`: endast för en symbol som filen inte kan svara på.** En symbol som du utelämnar tar det enda bandet i filen vars namn börjar med den: `red` tar `Red_660` eller `Red_660_RGN`, `nir` tar `NIR_850`. Namnen hämtas från själva filen: stackens egen bandlista, eller, vid en Chloros-radian- eller reflektansinspelning eller en `process`-produkt, dess `Chloros:Band{R,G,B}Name`- och `XMP-Camera:BandName`-taggar. Så `lattice index --input <reflectance capture> --preset NDVI` behöver ingen `--channel`.
- **Två band som svarar på en symbol gissas inte in** (en stapel med 9 kameror har två NIR-band): namnge det. En symbol som inget motsvarar avvisas med `channel_map missing entries`, avsluta med `1`.**Bandnamn i en stapel** bär kamerans filter: en M3C-kameras band är `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` på en RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` på en OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` på en RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` på en NGB — och en M3M-kameraenda band är `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` listar banden i en profil.

Fullständig flagguppsättning: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (upprepningsbar), `--capture-level raw|debayered|radiance|reflectance|unknown` (åsidosätter den inspelningsnivå som registrerats i källTIFF-filen; standard: läses från TIFF-metadata, med fallback till en Chloros-fils `Chloros:ProcessingLevel`, så en reflektansinspelning rapporterar `reflectance`), `--output PATH` / `-o` (offline: en sökväg; med en bildfiländelse namnges filerna utifrån dess stam — `-o ndvi_FRGN.png` skriver `ndvi_FRGN.tif` och `ndvi_FRGN_LUT.png` — och utan en sådan är det ett prefix, `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; standard vid sidan av indata, som `<input name>_<preset>`; med `--live` och utan `-o`, en mapp med namnet `output/` under den mapp där du kör kommandot), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Med `--live` gäller även justeringsreglagen: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (kamerornas mål för automatisk exponering, se reglerna för övergående sessioner), `--serials SN1,SN2,…`. `--live` öppnar en [övergående array](#multi-camera-transient-sessions): en kamera som inte levererar någon bildram misslyckas med körningen (avsluta `1` och namnge den), och ingen stack eller index skrivs. Den öppnar `--serials` (den första utlöser triggern), annars profilens kameror vars `layer_order`-post är aktiverad, med dess referens först — referensen även om dess band är inaktiverade, eftersom de andra är anpassade efter den och den utlöser triggern. En kamera vars band är inaktiverade kalibreras aldrig, så en RGB-kamera som är inaktiverad i profilen misslyckas inte längre med ett reflektansindex (`no-absolute-radiance-scale`). En profil som inte namnger några kameror öppnar alla kameror i nätverket, precis som tidigare.

Varje formulär körs på backend-systemet, så det måste vara igång: starta den från Chloros-GUI:n, eller med det kommando som CLI:n visar när den inte kan nå den. Offline läser backend `--input` och skriver själv float-TIFF-filen och den färglagda PNG-filen, så en relativ `--input` eller `-o` hänvisar till den mapp du kör kommandot från (se [Konventioner](#conventions)). Båda offlineprodukterna innehåller kamerainformationen från indata, precis som en `process`-indexexport gör: Märke / Modell / serienummer, GPS, objektivtaggarna, tagningstiden (indatats egen; när EXIF-informationen inte anger någon, det exponeringsögonblick som registrerades eller filnamnets tidsstämpel, i UTC) samt MAPIR-kalibreringsidentiteten — aldrig raw-DN-kalibreringen, som inte gäller för ett index — plus `Chloros:ProcessingLevel=index` och indexuttrycket; float-värdet TIFF namnger ett band, förinställningen. `--list-presets` och `--list-gradients` skriver ut backendens förinställnings- och gradienttabeller. En formel som backend avvisar, eller ett `--channel`-band som TIFF-filen inte har, avslutar `1` med backends motivering.

> **`--channel`-symboler är skiftlägeskänsliga.** Symbolsidan måste exakt matcha förinställningens kanalnamn (förinställningar använder gemener, t.ex. NDVI = `red`,`nir` — kontrollera `--list-presets`), och bandsidan måste stämma överens med ett bandnamn i den justerade stapeln (eller vara ett 0-baserat bandindex i offline-läge). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` fungerar; `--channel RED=660` misslyckas med ett `channel_map missing entries` .

#### `lattice index` — maskering av flerbandsstackar

Vid `--capture-level radiance` eller `reflectance` kan varje kameras 3×3-avblandning klämma fast ett band till 0 (se [Unmix-klämmask](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). En multiband-stack innehåller band från flera kameror, och varje kamera begränsar sitt eget band, så `--live --save-multiband` skriver **en mask per källkamera** bredvid stacken:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Varje mask är en enkelbandsmask `uint8` på **stapelns** pixelrutnät (genom samma justeringsförvrängning och beskärning), med den kamerans egna bitar: bit 0 = dess R-kanals band, bit 1 = G, bit 2 = B. Ett stapelband hämtat från kanal *c* hos en kamera begränsas där den kamerans bit *c* (R = 0, G = 1, B = 2) är inställd.
- Varje bands inbäddade härkomst anger kamerans mask (`unmix_clamp_mask`, relativt stackens mapp, eller `none`) och vilken bit som ska läsas (`unmix_clamp_mask_bit`).
- En kamera vars unmix-bearbetning inte begränsade något får ingen fil; en mono- eller RGB-kamera kör ingen unmix-bearbetning och får heller ingen fil. CLI:n skriver ut varje mask under stapeln (`clamp mask SN…: …`); svaret listar dem som `clamp_masks`.
- Index-TIFF-/PNG-filen från samma körning beräknas utifrån samma stapel, på samma rutnät, så maskerna beskriver även den. `--capture-level raw` kör ingen unmix och skriver ingen mask.
- Stapeln och indexet finns direkt i `--output`, så maskerna hamnar i `--output`:s egen `clamp_masks/` – aldrig ovanför den mapp du namngav. Den mappen innehåller olika produkttyper (stack, index, färglagd rendering): välj ut dess filer, överlämna inte hela mappen till fotogrammetriprogrammet.

#### Persistenta anslutningar (Smart-Prep, GUI-motsvarande flöde)

Dessa kommandon håller kamerorna öppna i backend-poolen över flera CLI-anrop.

| Underkommando | Syfte |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Lägg till en kamera i poolen (enstaka kamera, ingen matris). De [gemensamma alternativen](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` och `--packet-size` (`standard` = 1500, `jumbo` = 9000, eller byte; `auto`-sonder) tillämpas vid anslutning (en `--exposure`- eller `--gain`-kommando som stänger av autoläget). Det återgår när kamerans ström har startat (väntar cirka 15 s på det, sedan 10 s på dess första fullständiga buffert) och skriver ut `stream: streaming` eller `stream: no_frames -- connected, no frames yet: …` när strömmen inte levererade något — för en kamera som använder jumbo-paket, med rekommendationen att återansluta med `--packet-size standard`. Medan en kamera ansluter (en första anslutning laddar ner dess kalibrering, cirka 70 sekunder), väntar ett annat kommando som behöver en kamera på den i högst 5 sekunder, därefter misslyckas det med HTTP-felkod 503 `CAMERA_POOL_BUSY` där den anslutande kameran anges; kör det igen om en stund. `lattice status` och kameralistan väntar aldrig: en kamera som är mitt i anslutningen visas som anslutande, med sin fas. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Släpp. Kameran återställs till det tillstånd den befann sig i vid anslutningen (utlösarkoppling, PTP, inställningar); `--keep-settings` sparar istället sina inställningar i sitt eget minne, `--clear-settings` återställer det minnet till fabriksinställningarna (se [Kameraminne](#camera-memory----keep-settings----clear-settings)). En kamera som ingår i en array frigör hela arrayen. |
| `lattice cam-list` | Lista kameror i poolen. |
| **`lattice array-connect`**|**Anslut en permanent synkroniserad array (DEN rekommenderade startpunkten).** Kör hela GUI-flödet för smart-prep. Det tar ungefär en minut för 9 kameror; direkt efter att kamerorna har startats kan det ta flera. Medan den väntar skriver den ut varje steg som backend når, som `    <s> s: <step>` (till exempel `    41 s: open SN:214000535 (4 of 9)`), och backendens logg har en tidsrad per steg (`[array-connect <id>] <step>: <s> s …`) samt en sista rad med de långsammaste stegen. Den väntar i upp till 600 s. En anslutning som inte har genomförts vid den tidpunkten avbryts av backend: varje kamera som den öppnade återställs till det tillstånd den befann sig i när den hittades och stängs, kommandot skriver ut `Error:` med det steg som hade nåtts och avslutas med `1`, och ingen matris är ansluten (HTTP-kod 504 `CLIENT_DEADLINE_PASSED`). Ctrl+C avbryter anslutningen på samma sätt (`Interrupted: …`); om arrayen redan hade anslutits anges detta i raden och namnet på det kommando som frigör den anges (`array-disconnect`). Ett kommando som avbryts medan det väntar (`kill -9`) gör att anslutningen ångras när backend inte har hört något från det på 30 sekunder. En array som är igång men som ännu inte har någon synkroniserad ramgrupp rapporteras fortfarande som ansluten, med sin ström som `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Frigör en array. Varje kamera lämnas i fritt läge (`TriggerMode` `Off`), med allt annat som det var vid anslutningen — dess inställningar för utlösare och synkroniseringslinje (master-s `TriggerSource` återställs till vad den var, dess synkroniseringslinje återställs till en ingång), dess PTP och de kamerainställningar som sessionen ändrade — så att ett kommando för en enskild kamera på en medlem fungerar omedelbart. `--keep-settings` sparar varje kamerasinställningar som de var när sessionen avslutades i kamerans eget minne; `--clear-settings` återställer det minnet till fabriksinställningarna. Se [Kameraminne](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Lista anslutna kameragrupper. |
| `lattice array-status [--array-id ID]` | Live-fps, PTP, senaste felet. `ptp_enabled` är endast `True` när alla enheter har PTP aktiverat; `ptp_off:` anger en enhet som inte har det. `state:`-raden läser `restarting (applying settings)` under en omstart av inställningarna, `recovering (…)` medan en deltagare som förlorat ström återansluts (se anmärkningen nedan), `stalled (…)` medan hämtningsslingan körs men inte publicerar någon ramgrupp, och `stopped (…)` när hämtningsslingan har gett upp (anslut arrayen igen). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | En synkroniserad inspelning från live-matrisen — Enstaka / Kontinuerlig / Intervall / Snabbaste. **Standardinställning är `all`**(en fil per tillämplig exporttyp per kamera),**justerad** enligt appens inspelning (`--no-aligned` för ursprungliga filer). Kameror som hoppats över (t.ex. RGB-kameror uteslutna från strålningsintensitet/reflektans) rapporteras med `Skipped: SN:<serial> (<reason>)`; den DAQ som läser av den använda inspelningen sparas tillsammans med den, en inspelning som endast innehåller rådata inkluderas och rapporteras med `DAQ: <path>` ([DAQ:n som läser av en inspelning sparar](#the-daq-reading-a-capture-saves)). Se [Inspelningslägen, inspelningsenheter och offline-bearbetning](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Spela in den kombinerade indexvyn i realtid som video/GIF (övervakningskvalitet; utan öppet GUI-fönster renderar backend-modulen matrisens NDVI, eller `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Raw-Bayer-seriebildtagning med hög bildfrekvens (analyskvalitet; ombearbetas offline). Liksom `array-record` och `array-capture --continuous` rapporterar den till backend vid varje avfrågning: avslutas utan att stoppas (`kill -9`), backend avbryter den 30 sekunder efter den sista incheckningen (`stop_reason` `client_gone`; seriens manifest slutförs och skapandet av det kombinerade indexet påbörjas, precis som vid ett avbrott) och arrayen förblir ansluten. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Bearbeta om en sparad rå burst till kalibrerad video. |

> **En kamera som förlorar ström eller sin anslutning återansluts av sig själv.** När en enhet slutar svara (en strömbrytare startar om alla kameror den försörjer), väntar arrayen på den, öppnar anslutningen igen när den svarar, och återställer de inställningar som `array-connect` angav: PTP, trigger- och synkroniseringslinjen, nätverkshastighet, pixelformat och bildstorlek samt automatisk exponering. Därefter strömmar den igen utan någon ny `array-connect`. Under tiden visar `array-status` `state: recovering (...)` och `last_error` anger vilken kamera den väntar på; därefter skriver den ut `re-opened:`. En kamera som har startats om återkommer med PTP avstängt, så tills den öppnas på nytt är `ptp_enabled` lika med `False` och `ptp_off:` namnger den.

> **En kamera som hålls öppen öppnas aldrig en andra gång.** En andra öppning stoppar innehavarens ström och skriver om dess inställningar, vilket kan krascha backend-systemet. Så på en kamera som poolen håller (GUI:n eller `cam-connect`), en arraymedlem eller en kamera som ett bakgrundsjobb håller (en körande `lattice continuous`, `multi-stream` eller `align-stream`, sin egen eller en annan klients), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, en-shot `white-balance` och `presets --apply` avvisas med HTTP-kod 409 `CAMERA_HELD`, med angivande av innehavaren och frigivningskommandot (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>`, eller för ett jobb dess id och vem som startade det), och avsluta `1`. Detsamma gäller en kamera som ett annat kommando använder just nu (ett `multi-test` eller `multi-capture` som öppnar sina kameror, ytterligare en engångsbild), för varje kommando här: felmeddelandet säger att man ska vänta tills det kommandot är klart. `lattice power` utan `--sleep` läser igenom innehavarens eget handtag där det finns ett (poolen, en array, ett jobs öppna kamera) och avvisas om en kamera används av ett annat kommando. Varje `CAMERA_HELD`-svar innehåller koden i både `code` och `error_code`. För kommandon som öppnar alla kameror, se [Flera kameror (tillfälliga sessioner)](#multi-camera-transient-sessions).

##### `array-connect`-alternativ

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatisk upptäckt av alla LATTICE-kameror (kräver ≥2) | Den första i serien är MASTER. Om flaggan utelämnas filtreras upptäckten till LATTICE-modeller (M3C / M3M) och alla ansluts. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkroniseringslinje. |
| `--target-fps F` | auto | Master-triggers avfyrningshastighet. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Åsidosätt nivåväljaren. |
| `--wire-ceiling-mbps MB_PER_S` | auto-detected | **Värdens kontinuerliga bandbredd, i MB/s — det värde som hela arraytilldelningen baseras på.** Sänk den när arrayen rapporterar GVSP-korrupta ramar: det automatiska värdet härleds från nätverkskortets angivna länkhastighet, vilket överskattar USB-adaptrar, smala PCIe-banor och upptagna delade nätverk. Sparas i projektets array-insamlingsblock, så en återöppning eller återanslutning via CLI/SDK återställer det. Se [Arrayhälsa](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | ingen (full sensor) | Hårdvarubinning (alltid genomsnitt). |
| `--width N` / `--height N` / `--pixel-format FMT` | hel sensor; format per kamera | En explicit bildstorlek eller ett pixelformat för varje kamera, istället för valet mellan hel sensor och format per kamera (Mono12 / BayerRG12). |
| `--no-recommend` | av | Hoppa över nätverksanalyssteget (varje kamera behåller då sin nuvarande storlek istället för att ställas in på hela sensorn). |
| `--no-ptp` | av | Inaktivera PTP (tidsstämplar mellan kameror är då **inte** jämförbara). |
| `--packed {auto,on,off}` | `auto` | Strömma 12-bitarsramar packade och packa upp dem på den här datorn — fler ramar per sekund vid full sensor, identiska data. `auto` packar endast när det är till hjälp och den här datorn klarar det; `off` packar aldrig. Se [Packad 12-bitars transport](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | GVSP-paketstorlek för varje kamera i matrisen. `auto` låter varje kamera sondera nätverket; `jumbo` = 9000, `standard` = 1500 eller en storlek i byte låser fast den. Före version 1.3.0 accepterade `array-connect` denna flagga men ignorerade den. |

> **Full sensor som standard.** Utan `--width` / `--height` / `--binning` /
> `--pixel-format`, ställer `array-connect` in varje kamera på full sensor — ingen
> binning, ingen ROI-beskärning — och låter bildfrekvensen sjunka till vad den här datorn och
> nätverket klarar av, istället för att krympa bilderna för att hålla uppe frekvensen. Den
> nätverksanalysraden visar den beräknade frekvensen, och nästa rad listar
> snabbare alternativ (binning eller en mindre ROI) med deras beräknade frekvenser; hoppa
> `--binning` eller `--width`/`--height` för att välja ett alternativ. Connect utför aldrig binning eller beskärning
> på egen hand.

##### Packad 12-bitarsöverföring (`--packed`)

Vid full sensorkapacitet bestäms en matris bildhastighet av hur snabbt varje kamera kan skicka
sina bilder. 12-bitarsformat (`BayerRG12`, `Mono12`) överförs normalt som 16-bitars
ord; **packad** överföring skickar samma 12 bitar i 1,5 byte per pixel och
packar upp dem på den här datorn, så att varje bildruta tar 25 % kortare tid att skicka —
cirka **33 % fler bildrutor per sekund vid full sensoranvändning** (uppmätt: 10,7 → 14,0 fps
på en uppsättning med 9 kameror). **De sparade uppgifterna är identiska**: samma värden, samma
16-bitarsfiler, samma metadata.

- `auto` (standard) packar endast när det är fördelaktigt **och** den här datorn har kapacitet att
  hantera uppackningen. Vid anslutning mäter backend-enheten sin egen uppackningshastighet, antal CPU-kärnor,
  aktuell CPU-belastning och nätverkskapacitet, så en stationär dator med lediga kärnor komprimerar
  vid full sensor, medan en upptagen eller strömsnål dator strömmar okomprimerat. Små ROI:er och sammanslagna bildrutor ger
  lite.
- `off` komprimerar aldrig — använd det för att utesluta komprimering vid felsökning, eller på en
  dator som måste behålla all sin CPU-kapacitet för annat arbete.
- `on` packar alltid 12-bitarsformat.

Connect-utdata visar beslutet och varför (`packad 12-bitars: på (auto:
prognos 11,0 -> 14,4 fps, …)`); `/api/camera/array/status` rapporterar
`packed_transport` och `packed_decision`. För att ställa in en maskinomfattande standardinställning
istället för att ange flaggan varje gång, ställ in `CHLOROS_ARRAY_PACKED12=off` (eller
`on`) i backendens miljö – en explicit `--packed on|off` har fortfarande företräde.
8-bitars- och 16-bitarsformat packas aldrig.

### En projektpost, alla lägen

En projektmapp innehåller inställningarna för varje enhet: `cameras.json` — varje kameras (`cameras[].settings`, för en enskild kamera och för varje medlem i en array) samt de inställningar som gäller för hela arrayen (`arrays[].capture`: format, ROI, binning, utlösningsfrekvens, wire budget) — och `sensors.json`, för varje ljussensor (integrationstid, bildgenomsnitt, cap). Appen, CLI, SDK och MCP-servern läser och skriver alla samma filer på samma sätt:

- **En enhet som är ansluten till ett projekt sparas i det och registrerar varje ändring där.** ”Under ett projekt” innebär att ett projekt är öppet i backend (appens), eller — från SDK:et — via en `ChlorosProject`, som namnger sin egen mapp. En kamera, matris eller ljussensor som projektet ännu inte innehåller läggs till i dess filer när den ansluts (`lattice cam-connect`, `array-connect`, `daq pool-connect`, appen, SDK, `chloros-mcp`), tillsammans med dess inställningar. En ändring från vilket läge som helst (`lattice exposure`, `gain`, `configure --set`, `resolution`, en matris’inställningar och utlösningsfrekvens, en ljussensors gränsvärde, appens kontroller) sparas i projektets filer en stund senare. En exponering, förstärkning eller AE-mål som ställts in på en kamera sparas som inställd, så att den behålls när projektet öppnas igen.
- **När ett projekt öppnas ställs enheterna in enligt projektet**, i alla lägen: en kamera, matris eller ljussensor som är ansluten utan egna inställningar får projektets sparade inställningar (`lattice cam-connect` / `array-connect` med ett öppet projekt, `chloros-cli project connect`, `open_project().connect_all()`, när appen öppnas på nytt). En matris som ansluts under ett nytt ID uppdaterar den matris som projektet sparade med samma kameror; en matris med andra kameror läggs till bredvid den.
- **Appens egen sparning skriver aldrig över en enhets aktuella inställningar** med sin äldre kopia av dem, och behåller de överskrivningar per medlem som en CLI- eller SDK-anropare har sparat.
- **Det pixelformat du ställer in är det som projektet behåller.** Ett format som ställs in i Array Settings, via `--pixel-format` eller via `connect_array(pixel_format=)`, inklusive 8-bitarsformat, sparas och återansluts exakt. Om inget är inställt ansluts varje kamera med sitt 12-bitars standardvärde (`BayerRG12`, `Mono12` på en monosensor). Ett formatnamn anger dess bitdjup; familjen är sensorns, så en färgkamera ställs aldrig in på ett monofärgformat eller en monokamera på ett Bayer-format, och raden `[PixelFormat <serial>] ... written` anger detta när ett format har mappats. En 8-bitars inspelning exporteras som 8-bitars, inklusive reflektans: omskalad och märkt med dess `PixelScale`.
- En enhet som är ansluten utan att vara kopplad till något projekt spelar inte in någonting.
- **En inspelning följer ett projekt som skapats efter den; när den stängs kvarstår projektet.**När ett projekt skapas — ett SDK-projekt `create_project()`, `chloros-mcp`:s `create_project` eller appens**Nytt projekt**— eller dupliceras (appens**Duplicera**, som öppnar kopian) medan en inspelning fortfarande skriver till det projekt som var öppet tidigare, fortsätter inspelningen och det som skrivs hamnar i det nya projektet, på samma plats inuti det (nedan). Att öppna ett befintligt projekt — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, appens**Öppna projekt** — gör att en inspelning fortfarande pågår i det projekt den spelar in i: den avslutas där, och dess filer läggs till i det projektets fillista (appen avbryter varje inspelning innan den öppnar ett projekt).
  - `array-record` och `array-burst`: videon, eller seriemappen med varje bildruta, flyttas dit i sin helhet när inspelningen avslutas (dess `--duration`, Ctrl+C). Den `Burst dir` som skrivs ut är den nya, och `--build` samt `array-build-video` utgår från den. Om en flyttning avvisas av enheten lämnas inspelningen kvar där den skrevs; loggen islutloggen anger varför, liksom stoppets HTTP-svar (`relocation_error`). En avbruten bursts råbilder och `.daq` läggs till i fillistan för det projekt den hamnar i; en video (en `array-record`, en sammanställd video från en sekvens) läggs inte till i ett projekts fillista.
  - `array-capture --continuous`: varje inspelning efter ändringen sparas i det nya projektet; de som sparades tidigare stannar kvar där de är.
  - `daq pool-record` och `project sensor log` (samt appens egen inspelning med ljussensor): `.daq` flyttas till det nya projektets`light_sensor`-mappen när inspelningen avslutas och läggs till i det projektets fillista.

  En inspelning i en mapp utanför det öppna projektet (en `-o` någon annanstans) förblir där den är, och `lattice continuous` och `multi-stream` behåller den mapp de tilldelades.
- **I appen hamnar inspelningarna efter ett nytt projekt i det.** När du skapar eller duplicerar ett projekt fortsätter alla pågående inspelningskörningar och inspelningar, och varje inspelning som görs därefter (Capture All, Start Capturing) sparas i det nya projektet och läggs till i dess fillista. När du öppnar ett befintligt projekt i appen stoppas först alla dessa.
- **Ett projekt flyttas inte medan det spelar in.**Att flytta det öppna projektet till en annan arbetskatalog (appens**Inställningar → Arbetskatalog**; inget CLI-kommando flyttar ett projekt, däremot gör en HTTP-klient som skickar till `/api/move-project` det) avvisas med HTTP-kod 409 `PROJECT_RECORDING` medan en matrisinspelning eller burst, en kontinuerlig matrisinspelning, ett `lattice continuous`-/`multi-stream`-jobb eller en ljussensorinspelning skriver till den: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Appen avvisar på samma sätt, innan den ber om en mapp, medan dess egen ”Capture All”, ”Start Capturing” eller ljussensorinspelning körs.

Detta är skilt från hur kamerorna är inställda vid frånkoppling (se [Kameraminne](#camera-memory----keep-settings----clear-settings)): ett projekt ställer in sina kameror själv när det öppnas.

### Kameraminne — `--keep-settings` / `--clear-settings`

När en matris är ansluten övertar dess kameror matrisens inställningar, och du kan sedan ändra dem. När Chlorossläpper kamerorna — `array-disconnect`, slutet på ett övergående kommando (`multi-capture` utan array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), stoppet för `lattice continuous`, slutet på en `lattice capture` för en kamera som den inte hanterar — den återställer varje enhet till det tillstånd den befann sig i före anslutningen: dess utlösare och synkroniseringsledningar, PTP- och kamerainställningar (den fullständiga listan finns under [Flera kameror (tillfälliga sessioner)](#multi-camera-transient-sessions)).

För att använda kamerorna utanför ett projekt skriver två flaggor på något av dessa kommandon till kamerans **eget minne** — det som den startar upp med, oavsett programvara:

- **`--keep-settings`** sparar varje kameras inställningar i dess minne precis som de var när sessionen avslutades (UserSet2, som utgör startinställningarna). Kamerorna lämnas fortfarande i fritt läge (`TriggerMode` `Off`), med synkroniseringsledningarna som de är. En kamera utan UserSet2 sparar till UserSet1; den här datorn kommer ihåg detta, så Chloros-matrisens självkorrigering — vilket återställer en UserSet1 som har avvikit från Chloros baslinje — lämnar den ifred.
- **`--clear-settings`** återställer varje kameras minne till fabriksinställningarna: Chloros baslinje, den som dess matris-självkorrigering inbakar — fabriksinställningar med `TriggerMode` `Off`, `TriggerSource` `Line2`, automatisk exponering, en RGB-kameras kalibrerade vitbalans bevaras — i UserSet1, som utgör startuppsättningen.
- De två kan inte kombineras. Varje kamera får en rad: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` eller `SN:…: camera memory reset to the factory settings.`; svaret `camera_memory` har samma värde per kamera.

Appen erbjuder inte dessa flaggor.

**Varje utgång återställer kamerorna.**Samma återställning som vid upptäckt körs på varje utgångsväg, inte bara för kommandona ovan: `cam-disconnect` (och appens kamera**X**); appens**X** på en array eller på vilken rad som helst hos en arraymedlem (en array kan inte köras vidare utan en medlem, så hela arrayen frigörs); stängning av appen (den ger backend upp till 20 sekunder på sig att återställa varje kamera, array och körande `lattice continuous` innan den avbryter den; ett kommando som fortfarande körs då — ett övergående kommando som öppnar sina kameror eller väntar på deras PTP-klockor, ett engångskommando `lattice capture`, `power`, `white-balance` eller `presets --apply` – avbryts vid nästa säkra punkt, återställer först sina kameror, i upp till 8 sekunder, och svarar med HTTP-kod 503 `SHUTTING_DOWN`; en anslutning som fortfarande är igång – `lattice array-connect`, appens anslutning – stannar före nästa kamera och stänger de kameror den har öppnat, i upp till 10 sekunder, och svarar med HTTP-kod 503 `SHUTTING_DOWN`; och en som anländer under avstängningen öppnar ingenting: HTTP-kod 503 `SHUTTING_DOWN`); och ett CLI-kommando som avbryts eller vars terminal stängs (SIGTERM / SIGHUP avbryter dess uppdrag — och stänger av sina kameror — innan den avslutas `143` / `129`, precis som Ctrl+C). En `lattice continuous`, `multi-stream` eller `align-stream` som avslutas omedelbart (SIGKILL), vilket inte kan stoppa någonting, får sitt jobb avbrutet och sina kameror återställda av backend-systemet 30 sekunder efter dess senaste incheckning. En backend som kraschar eller tvingas stängas ner kan inte återställa någonting; en omstart återställer varje kamera till dess minne, och nästa Chloros-anslutning kopplar om en kamera som den finner aktiverad.**Ett projekt ställer in sina kameror själv.** När en array ansluts med ett öppet projekt får varje medlem de inställningar som projektet sparat för den — en exponering, förstärkning eller ett AE-mål som hålls fast för den (behandlas som ditt eget, så automatisk exponering lämnar den ifred) och dess vitbalansläge (korrigering från Chloros; kamerans egen vitbalans förblir avstängd) — oavsett vad kamerans minne innehåller.

### Smart-AE / Smart-Capture

LATTICE-uppsättningar kör kontinuerlig AE i bakgrunden så snart de är anslutna, men det tar en stund för en nyinsatt scen att konvergera. `array-capture --smart` är den **praktiska lösningen**: den väntar tills AE har stabiliserats på alla kameror i matrisen och utlöser sedan fotograferingen. Använd den när du byter motiv mitt i en session.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Standardinställningarna för stabiliseringsprocessen är konservativa: 5 sekunders timeout, 1,5 sekunders stabilitetsfönster, ±5 % tolerans för exponeringsspridning. Justera via SDK:et (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) om du behöver ett annat beteende från automatiseringen.

#### Hur automatisk exponering fungerar

- **Exponering först, förstärkning sist.** När förstärkningen är inställd på Auto förblir den på 0 dB tills exponeringen har nått det längsta tillåtna värdet.
- **En manuell exponering lämnar förstärkningen på Auto** tills du ändrar förstärkningen eller ställer in den på manuell. När den återgår till Auto är förstärkningens gräns densamma som tidigare.
- **Små högdagrar gör inte scenen mörkare.** En spegling på upp till 1 % av bilden ignoreras, och i panelerna på ett känt kalibreringsmål ignoreras upp till 0,5 % av en panel.
- **DAQ-exponeringstaket gäller endast så länge ett kalibreringsmål syns i bild.** Det förhindrar att en vit panel klipps; när ingen panel syns exponerar kameran för scenen.
- **Konstant ljus förblir oförändrat, och förändrat ljus stabiliseras inom några sekunder.** Vid konstant ljus förblir exponeringen och dess gräns oförändrade, och efter en stor ljusförändring stabiliseras bilden inom några sekunder. En klippning begränsar inte längre exponeringen permanent, och en förändring av ROI återfinner exponeringen.
- **Ljusstyrkemålet hålls inom räckhåll.** Det stiger inte medan kameran befinner sig vid sin exponeringsgräns och dess förstärkning inte kan öka, så att ett ljus som återkommer inte orsakar en överexponering i bilden.
- **En kamera vars automatiska exponering slutar svara hålls på den nivå som motsvarar det ljus den befinner sig i.** I starkt solsken kan en kamera sjunka till sin kortaste exponeringstid och stanna där med en mörk bild, med alla inställningar som visar normala värden. Chloros övervakar bilden, håller kameran på den nivå den hade för det ljuset och släpper den när ljuset förändras. En mörk eller täckt ljussensor stör inte detta.

### Nivåer för bildtagning och export (standardinställningen för `all`)

Från och med denna version, `lattice capture`, `lattice multi-capture` och `lattice array-capture` **standardinställningen är `--processing all`** — en sparad fil per exporttyp som gäller för varje kamera, vilket motsvarar beteendet för ”Capture All” i användargränssnittet. Nivåerna är:

| Nivå | Utdata | Gäller för |
| --- | --- | --- |
| `raw` | Enkanalig Bayer (monokameror: det enda bandet) direkt från sensorn. | Alla kameror. |
| `debayered` | 3-kanals färg, sparad i **RGB**-kanalordning (en TIFF-fil med RGB-fotometrisk tolkning; PNG/JPEG öppnas likaså som RGB): kalibreringspaketets mörkförskjutning (DSNU) avlägsnas från rådata, därefter utförs OpenCV:s kantdemosaik – ingen linearitets-LUT, flat field eller unmix (monokromkameror: 1-kanals gråskala). De debayered-ramar i minnet som Python-SDK:et returnerar (`grab()`, `frame_stream()`) är**BGR**, OpenCV:s ordning; endast den sparade filen är i RGB-format. | Alla kameror. |
| `radiance` | float32 W/m²/sr/nm via den fullständiga radiometriska kedjan. | Endast multispektrala (M3C/M3M) — **utesluts för RGB-filterkameror**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-kompatibel. | Endast multispektralt, och **endast när en DAQ är kopplad + kameran är kalibrerad**; annars hoppas över. |
| `display` | Fullständig GUI-förhandsgranskningskedja (CCM + WB + gamma enligt kamerans profil). Sparas i en `preview/`-mapp. `lattice capture` accepterar även namnet `preview` för den; `array-capture` / `multi-capture` accepterar endast `display`. | Alla kameror. |

Ange en enda nivå för att spara just den (`--processing debayered`). När du begär `all` hoppas nivåer som inte gäller för en viss kam över (och rapporteras), men ger inte fel – en ansluten eller okalibrerad kamera får fortfarande `raw` / `debayered` / `preview`.

#### Den DAQ-avläsning som en inspelning sparar

När en DAQ är kopplad till kameran eller matrisen, eller ingår i en pool, skrivs den nedåtriktade avläsningen som användes vid inspelningen till en **`.daq`**-fil i undermappen `daq/` i mappen captures, tillsammans med dess `.match.json` (hur långt avläsningen låg från exponeringen) och dess `.cameras.json` (vems avläsning det är), och rapporteras på en `DAQ:`-rad. Detta gäller för varje nivå, inklusive råa eller debayrade bilder, så att vilken bild som helst senare kan bearbetas om till strålningsvärde, reflektans eller ett index. Det är samma fil, med samma namn och sidecars, oavsett om bildtagningen kom från GUI, CLI eller SDK. Utan någon DAQ-avläsning alls finns det ingen `.daq`, och en `multi-capture` utan ansluten matris (den tillfälliga reservlösningen) sparar ingenting.

| Flagga | Effekt |
| --- | --- |
| `--no-daq` | Spara inte avläsningen för en inspelning vars nivåer inte använder den (`raw`, `debayered`). En `radiance`-, `reflectance`- eller `display`-nivå skriver fortfarande ut den, eftersom återimport kräver det mätvärde som produkten skapades med. |
| `--daq-tolerance-ms MS` | Hur långt från exponeringen avläsningen får ligga för att fortfarande räknas som inom tolerans (standard 500). Endast bedömningen ändras (`.match.json`:s `in_tolerance` och `tolerance_ms`, reflektansens `DlsInTolerance`), aldrig det använda mätvärdet. Se [Ett DAQ-mätvärde långt från exponeringen](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Det äldre sättet att begära `.daq` vid en inspelning som endast innehåller rådata. Det är nu standard, och `lattice capture` och `array-capture` accepterar fortfarande flaggan (`multi-capture` gör det inte). `--no-daq` har företräde framför den. |

Vilken DAQ: den som är kopplad till kameran eller matrisen (valet av ljussensor i GUI:n, en projekts sparad koppling eller `chloros-cli daq pool-bind`), annars den senaste som backend har. En rå bildserie (`array-burst`) behåller sin egen `.daq` per enskild avläsning i burst-mappen som tidigare.

**PNG och JPEG har 8 bitar per kanal, så de använder endast `display` / `preview` och indexprodukter.** `--format png` eller `jpg` med en nivå som de inte kan hantera avvisas innan något fångas in, varvid nivån och TIFF-formatet som kan hantera den anges:

- `radiance` (float32) och `reflectance` (uint16, `32768` = 1,0) — alltid, så `--processing all` (standard) med `--format png`/`jpg` avvisas också: använd `--format tiff` eller `--levels display`.
- `raw` och `debayered` från en kamera som strömmar mer än 8 bitar (de 12-bitars M3C-/Mono12 M3M-ramarna) — CLI:n kan inte se kamerans bitdjup, så backend-systemet avvisar dessa (HTTP 400). Den enskildaramen `lattice capture --processing debayered` hämtar en 8-bitarsram för PNG/JPEG och avvisas inte, och inte heller en `multi-capture` utan en beständig array (8-bitars debayered).

Varje inspelningsväg svarar på samma sätt (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst`, samt HTTP-rutterna bakom dem.

### Så här ser en inspelningsmapp ut

Varje exporttyp placeras i sin **egen undermapp**under `-o`, vilket innebär att olika typer aldrig blandas i en inspelning med flera nivåer. Skissen visar den**standardmässiga `all`-inspelningen från en kamera av `lattice capture`** — flera nivåer per kamera, så varje fil har suffixet `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Hur filnamnen varierar:

- **Kamerafält – serienummer, sedan filter:** varje inspelningsfil namnges efter den kamera som tog bilden som `SN<serial>-<filter>`, där `<filter>` är filterdelen av kamerans modell: `SN261902142-F450` för en `M3M-L87-F450`, `SN213800234-FRGN` för en `M3C-L87-FRGN`, `SN214701292-FRGB` för en RGB-kamera. Filtret kopplas samman med serienumret till `-`, inte med `_`, så ett namn har samma `_`-separerade fält som tidigare (prefix, datum, tid, millisekunder, kamera, nivå). En kamera vars modell inte kan läsas (ett raderat DeviceUserID) namnges enbart `SN<serial>`. Inspelningar från tidigare versioner har inget filter i namnet; Chloros läser båda.
- **Prefix:** `lattice capture` namnger filerna `capture_…`; `array-capture`, `multi-capture` med en ansluten matris, och `project capture` / `burst` / `stream` av en matris namnger dem `sync_<ts>_SN<serial>-<filter>…`.
- **Nivåsuffix endast vid flera nivåer.**En kamera som sparar**flera**nivåer i en inspelning (`all` eller `--levels a,b`) lägger till `_<level>`. En kamera som sparar**en** nivå behåller det vanliga namnet utan suffix – `--processing reflectance` skriver `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` – så att skript som är skrivna för inspelningar med en enda nivå fortsätter att fungera. Nivån anges fortfarande av mappen.
- **Klämmasker följer produktens namn:** `<product name>_clampmask.tif`. En reflektansinspelning på en nivå skriver `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; en `all`-inspelning skriver `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Tillägget** följer efter `--format`: standardvärdet `tiff` ger `.tiff`, `--format tif` ger `.tif` (klämmasker är alltid `.tif`).
- **Ett undantag:** `lattice capture --processing debayered` (eller `--levels debayered`) tar en snabb enskild-ramsväg och skriver `capture_<ts>_SN<serial>-<filter>.tiff` direkt till `-o` — ingen undermapp.
- **Strömmar och bursts** har samma fält: `lattice continuous` skriver `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` och en rå burst `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` är tidsstämpeln för inspelningen och `<serial>` kamerans serienummer, så en synkroniserad grupp delar en
tidsstämpel mellan kamerorna. **Observera den enda asymmetrin:** nivån `display` lagras i en mapp
med namnet `preview/`, medan filerna på flera nivåer behåller `_display` i namnet — mappen och suffixet skiljer sig åt
endast för den nivån. Okända nivåer faller tillbaka till en mapp med sitt eget namn, och om undermappen
inte kan skapas skrivs filen till utdatarotkatalogen istället för att gå förlorad.

**Ombearbetning av en mapp med inspelningar:**peka `chloros-cli process` mot**rotkatalogen för captures**
(`output/`). `process` importerar normalt endast den mapp du anger, men när den mappen inte innehåller några
bilder men har undermappar går programmet automatiskt igenom dem – så att undermapparna på alla nivåer och
undermappen `daq/` hämtas på en gång. Varje nivå i en inspelning importeras som en enda bild, där
de andra nivåerna är tillgängliga som lägen, istället för en bild per nivå.

Det fungerar också att namnge en **nivåundermapp** direkt (t.ex. `output/raw/`). Om du gör det lämnas `daq/`
kvar, så kopiera eller peka in DAQ-avläsningen samtidigt när duhärleder en radiometrisk
produkt från `raw/` – annars har tidsstämpeln inget att jämföra mot.

**Bearbetningen börjar alltid från `raw`.** Inom varje inspelning är den obearbetade bilden källan till bearbetningskedjan;
`debayered`, `radiance`, `reflectance` och `preview` ingår som visningslägen men matas aldrig
tillbaka genom bearbetningskedjan. Om en härledd produkt bearbetas på nytt skulle vignettering, CCM och
strålningsberäkningar som redan är inbakade i dess pixlar tillämpas på nytt, så Chloros-metoden avstår hellre än att
utföra dubbelbearbetning. Två konsekvenser som är värda att känna till:

- Renderingarna `index/` och `composite/` **bearbetas aldrig** bearbetas. De är utdata, inte inspelningar —
  en NDVI-LUT-rendering har ingen meningsfull tolkning av strålningen.
- En mapp med inspelningar som exporterats **utan** `raw` (t.ex. `array-capture --processing reflectance`) har
  ingen giltig pipelinekälla. Dessa inspelningar importeras och visas normalt, men `process` hoppar över
  dem och meddelar detta:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Om du verkligen behöver skicka vidare en härledd produkt — en hubbsession som spelats in med
  `demosaic` aktiverat, eller en äldre mapp – tvingar `--input-level {raw,debayered,processed}` in
  ingångspunkten och åsidosätter hoppet. Den flaggan är en avsiktlig nödutväg; `auto` (standardinställningen)
  bearbetar aldrig en inspelning som saknar rådata.

**Kalibreringstaggar: endast på `raw`.** `raw`-filen för en inspelning innehåller kamerans kalibreringspaket: `CalibrationJSON`-nyttolasten och de MicaSense-kompatibla taggarna (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), som beskriver sensorns råvärden. Varannan nivås fil (`debayered`, `display`, `radiance`, `reflectance`, renderingen `index/`) samt varje `lattice continuous` / `multi-stream`-ram innehåller kamerans identitet (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) samt bildens egen post, `XMP-mapir:FrameJSON` (exponering, förstärkning, temperatur, pixelformat, inspelningstid, DAQ-matchning), eftersom varje `process`-produkt bär denna identitet. `process` hittar en sådan fils kalibrering utifrån dess serienummer och `CalBundleSha256` i Chloros kalibreringscache, och laddar ner paketet en gång om datorn saknar det.

#### Filer skapade från flera kameror

En fil skapad från band från mer än en kamera – en multibandstack (`align-apply`, `align-stream`, `index --live --save-multiband`), ett liveindex skapat från flera kameror (`index.tif`, `index_LUT.png`), en bild från en array-inspelning (`composite/`) – namnger varje kamera som ingick i den:

- **Listor per kamera, i bandordning** (kompositbilden: bakgrundslagret, sedan förgrunden; ett indexlager med de kameror som banden härstammar från): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (det kalibreringspaket som varje kameras band har passerat genom, annars — en stapel med `align-apply` / `align-stream`, vars band inte har genomgått någon kalibrering — det paket som kameran Chloros har cachelagrat, det som dess inspelningsfiler namnges efter), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — parallella listor, där post *n* i varje lista beskriver samma kamera; `-` där en kamera saknar värde (inget kalibreringspaket, ett okänt objektiv). Varje kamera är **en** post: om dess paket har flera värden för ett fält — till exempel ett pakets halogenlampcertifikat — sammanfogas de med `;` (`F-1479-1;F-1480-1;F-1481-1`), aldrig med ett kommatecken, så att listan visar en post per kamera oavsett hur den skrivs ut.
- **EXIF `SerialNumber`** innehåller samma serienummer, separerade med kommatecken (`214000535,214000551`).
- Den innehåller **ingen** enskild kameras identitet (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): dessa namnger en kamera och dess paket, och `process` skulle bestämma filens kalibrering utifrån dem. Ett index beräknat från flera kamerors band (`index.tif`, `index_LUT.png`) har EXIF-värdena `Model` och `LATTICE` (riggen), inte en enskild kameras `LATT-…`-modell.

En fil med **en** kameras band — en TIFF-fil (`--per-band`), en `align-apply --per-camera`-bildruta, en stapel eller ett liveindex för en enskild kamera — innehåller den kamerans egna identitetsetiketter, precis som alla andra bildfiler gör. `align-apply`:s stapel, bandvisa och kameravisa filer, samt varje `align-stream`-stapel och dess bandvisa filer, innehåller gruppens inspelningstid (referenskamerans exponering, UTC), och en monokameras `--per-camera`-ram är ett plan, precis som stapeln i `--bit-depth`.**Kamerans orientering (spegling / rotation).** En kameras orienteringsinställning (GUI:ns Mirror H /
Mirror V / rotation) avgör vad som sparas vid en bildtagning:

- **`raw` och `debayered` sparas exakt så som sensorn läste in dem**, oavsett inställning. En
  Bayer-mosaik som vrids eller speglas skulle utgöra ett annat färgfiltermönster, och kalibreringskartorna
  (mörkförskjutning, flat field) ligger på sensorns eget rutnät.
- **`preview`/`display`, `radiance`, `reflectance` och renderingen `index/` är orienterade** — vridna
  en gång, efter all kalibreringsberäkning, så att de öppnas på samma sätt som kameran var inriktad. Deras
  `clamp_masks/…_clampmask.tif`-filer ligger på samma rutnät.
- **Varje bildfil registrerar inställningen** i XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, medurs, efter
  speglingarna) samt `ChlorosOrientationApplied` — `false` för rådata och debayered (ursprungliga pixlar),
  `true` på de orienterade produkterna. `ChlorosProcessing` listar de operationer som tillämpats på den filens
  pixlar (`none` när de är ursprungliga). Detta inkluderar varje bildruta som `lattice continuous` strömmar (
  kamerans egna inställningar, ej tillämpade) och varje bildruta i `lattice multi-stream` (identiteten: dess
  ingående bildrutor är ursprungliga, precis som vid en array-inspelning). `lattice align-stream` / `align-apply`
  -stackar registrerar huvudkamerans inställning, utan tillämpning: deras band förvrängs in i huvudkamerans
  bildram men vrids inte.
- **`process` tillämpar den registrerade orienteringen** när den bearbetar en råbild: mörkförskjutning, flat
  field, demosaik och avblandning i sensorkoordinater först, därefter arrayjusteringens förvrängning och
  orienteringen sist — så att förhandsgranskningen, strålnings-, reflektans- och indexprodukterna blir exakt som
  vid liveinspelningen, och registrerar `ChlorosOrientationApplied=true`. De okalibrerade reservprodukterna
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) är orienterade på samma sätt, efter
  flatfältsbilden. Den debayrade produkten förblir i sitt ursprungliga läge.
- **Chloros-bildvisaren visar en rå eller debayrad fil vänd enligt dess inspelningsriktning** (endast visning —
  filen ändras inte), så att den ser ut som dess produkter; miniatyrbilden, fullstorleksvyn,
  marköravläsningen och Index/LUT-sandlådan använder alla samma vridning. En fil utan post, och
  en produkt (redan orienterad), visas som den är lagrad. En sandlådeexport registrerar den orientering som dess
  pixlar har (`ChlorosOrientationApplied=true`; identiteten för ett lager utan registrering).
- **Bildtagningar utan posten behandlas som ursprungliga** (oförändrat beteende). Före version 1.3.0 vände en
  bildtagning med en enda kamera, där en spegelvändning eller rotation var inställd, själv sina råfiler och debayered-filer;
  Chloros kan inte avgöra om en sådan redan-vänd råfil från en native och gör inga gissningar, så den
  bearbetar den som den är lagrad. Kalibreringskartorna ligger på sensorns eget rutnät, så för en sådan
  bildtagning bör man använda en som tagits med 1.3.0 när kalibreringen är viktig.
- **Arrays roterar inga medlemsfiler som standard** (justeringsförvrängningen och XMP-filen `Alignment*` finns i
  kamerornas egna, oroterade bildrutor) och registrerar identiteten. `array-capture --orient` roterar varje
  medlems produkter efter justeringsförvrängningen: enligt **huvudbildens** orientering när de
  förvrängdes in i dess bildram (så att medlemmarna förblir samregistrerade), annars enligt sina egna kamerors. Varje
  fil registrerar orienteringen för den bildram där dess egna pixlar befinner sig — en råfil, med den egna kamerans orientering —
  och en råfil som innehåller justeringstransformationen registrerar även `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: masterbildens, för en produkt som förvrängts
  till huvudbildens ram. `process` vrider varje produkt med den vinkel som hör till dess pixlar:
  den justerade när den förvränger RAW-filen (`--array-alignment`, standardinställningen – produkterna från den livejusterade
  inspelningens produkter), kamerans egen när den inte gör det (`--no-array-alignment` — den live
  ojusterade inspelningens). Samma alternativ finns för `array-burst`, `project capture` / `burst` /
  `stream`, recept och SDK.

### Hoppade bilder i arrayer med blandade filter

När du blandar RGB- och multispektrala kameror i en matris sparar `array-capture --processing radiance` (eller `reflectance`) de multispektrala bildrutorna och **hoppar över** RGB-kamerorna — strålningsvärdet per Bayer-pixel är inte meningsfullt för en bredbandssensor. CLI:n visar varje sparad fil (med dess exportnivå), varje skriven `.daq` och varje hopp uttryckligen, så filantalet är inte överraskande:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Token för hopporsaker följer mönstret `<level>-not-applicable-to-rgb-cam`. En kamera vars kalibreringspaket inte är laddat hoppar över `radiance` / `reflectance` med `<level>-needs-calibration (connect the camera so its cal pack loads)`, och vid avsaknad av användbar DAQ-avläsning av reflektans hoppas över med `reflectance-skipped-no-dls`. Reflektans kan också hoppas över med `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, och med `dls-uncalibrated-band-<nm>` när bandet till största delen ligger utanför DAQ-ljussensorns radiometriskt kalibrerade intervall (~374–974 nm) — bland de levererade SKU:erna är det endast F988, vars stödda arbetsflöde är reflektanspanelen.

Använd `--processing debayered` (eller `display`) för att inkludera alla kameror oavsett filtertyp, eller standardvärdet `all` för att få alla tillämpliga nivåer per kamera i ett enda steg.

### En DAQ-avläsning långt från exponeringen

Varje `.daq` som en bildtagning sparar är den DAQ-avläsning som motsvarar den exponeringen, och `.match.json` bredvid den registrerar hur långt ifrån varandra de två låg (`gap_ms`) och om det ligger inom toleransgränsen (`in_tolerance`, 500 ms om inte annorlunda angivits i inställningarna för bildtagningen `--daq-tolerance-ms`). Ett värde utanför toleransgränsen används ändå, så ingen bild går förlorad, och det märks upp: reflektansfilens XMP-data innehåller `DlsInTolerance=False` och `DlsMatchGapS`. `lattice capture`, `multi-capture` och `array-capture` skriver också ut den under den `.daq` som den avser, med DAQ:sströmfel när man förklarar det:

**Hur en avläsning kopplas till en exponering.** En inspelning väntar på DAQ:ns första avläsning efter exponeringen, så att den kan sammanfoga avläsningarna på båda sidor om den (`method` `interpolated`). En DAQ-E skickar en avläsning var 0,4 s i dagsljus och var 1–2 s inomhus, så väntetiden motsvarar ungefär ett och ett halvt av dessa intervall för kommandot, inte för exponeringen. En DAQ som har slutat sända väntas inte på. En DAQ-E-avläsning är det ljus som är medelvärdet över intervallet sedan dess föregående avläsning, så den placeras i mitten av det intervallet: `gap_ms` mäts från exponeringen till den mitten, och `bracket_stamps_ns` anger avläsningarnas egna tidsstämplar. `array-capture --continuous` väntar utan att bromsa inspelningen: dess råfiler skrivs direkt och deras `.daq` så snart avläsningen anländer. Med `--interval` väntar varje omgång som en enskild avläsning.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

När ingen reflektansfil skrevs ut från den avläsningen avslutas den första raden istället med `saved anyway; its .match.json says in_tolerance: false`.

Inläsningen avslutas ändå med `0`: alla filer har skrivits och anger vad de innehåller. En DAQ vars ström har fastnat fortsätter att leverera sin senaste avläsning, så kontrollera den med `chloros-cli daq pool-list` innan du litar på den inläsningens reflektans. Backend återansluter en poolad DAQ-E som har slutat leverera (se [En DAQ-E som slutar leverera](#a-daq-e-that-stops-delivering)). Ett mätvärde utanför toleransgränsen skriver endast ut sin `DAQ:`-rad, såvida det inte är ett bevarat mätvärde (nedan).

**Ett mätvärde som behålls efter att dess DAQ försvunnit.** När ingen DAQ kan tillhandahålla ett mätvärde för en exponering (den bundna DAQ:en är otillgänglig, eller ingen poolad DAQ har något, t.ex. efter `daq pool-disconnect`), återanvänder avläsningen den senaste avläsningen som Chloros hade, och dess `.match.json` beskriver den avläsningen mot denna exponering: `method` `retained`, `gap_ms` från denna exponering till de prover som avläsningen gjordes från, `in_tolerance` från det mellanrummet, och `retained_from_epoch_ns`, den exponering som avläsningen först matchades mot. En registrering som görs långt efter att DAQ:n stängdes av skriver därför ut raderna ovanför, och under dem ytterligare en, som skrivs ut även när avläsningen fortfarande ligger inom toleransgränsen:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Dess `.daq` anger fortfarande vilken DAQ som tog avläsningen, tillsammans med kalibreringen och locket som avläsningen togs under, så att en återimport vet vilken DAQ, kalibrering och vilket lock som producerade den.

### Filer som inte kan skrivas

En kamera vars fil inte kan skrivas – en full hårddisk, ett behörighetsfel – hindrar inte de andra kamerornas filer från att sparas. Varje inspelningskommando (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; samt receptets `capture`, `burst` och `stream`) skriver ut varje fil som misslyckades till stderr, och **avslutar sedan `1`** trots att resten sparades:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Alla filer som en inspelning ska generera täcks, inklusive den endaram med debayering, `lattice capture --levels debayered`, ingår. En `.daq`-avläsning vars `.match.json`-sidokar med härkomstinformation inte kunde skrivas listas som `[match]` (själva `.daq` sparades).

På en full disk listas en upprepad inspelning (`array-capture --continuous` / `--interval`, `project burst` / `stream`, ett recept `burst` / `stream`) **avbryts vid det passet** istället för att inte skriva något till samma enhet under resten av körningen. Alla andra fel rapporteras vid varje pass de inträffar, och körningen fortsätter till slutet, varefter den avslutas med `1`. `array-capture --continuous` (backendens kontinuerliga insamling) listar inga filer, inklusive de som misslyckats: den räknar dem i sina framstegs- och sammanfattningsrader (`N failed`, `N file(s) failed to write.`) och avslutas med `1` — se [Kontinuerligt läge](#continuous-mode---continuous).

Inspelarna — `array-burst` och `array-record` — rapporterar sina egna fel. `array-burst` stänger av sig själv när enheten blir full, eller strax innan den blir det. Båda fortsätter trots eventuella andra misslyckade skrivförsök. När någon av dem rapporterar ett fel skrivs det ut efter räknarna, och kommandot **avslutar `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` istället för `Disk full:` innebär att inspelningen avbröts innan hårddisken. De ramar som spelades in före avbrottet behålls. `array-burst --build` hoppar över kompileringen när utrymmet tog slut, eftersom videorna skulle sparas på samma hårddisk. Frigör utrymme först, kör sedan `array-build-video --burst-dir …`. Efter vilket annat fel som helst fortsätter kompileringen ändå.

Ett `array-record` som inte skrev någon bildruta alls är också en misslyckad inspelning (`Recorder error: no-frames: the combined view published no frame to record …`, avsluta `1`) — se [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Inspelningslägen, inspelningsprogram och offline-bearbetning

Alla dessa fungerar på en **persistent array** (kör `array-connect` först). De speglar GUI-panelen för inspelning.

### `array-capture`-lägen

`array-capture` är ett enda kommando med fyra slutarlägen samt en uppsättning exportalternativ:

| Läge | Flagga | Beteende |
| --- | --- | --- |
| **Enstaka** *(standard)* | (ingen) | En synkroniserad inspelningsgrupp, avsluta sedan. |
| **Kontinuerligt** | `--continuous` | Sammanhängande inspelningar tills `Ctrl+C`, `--count N` har spelat in, `--duration S` eller en fullständig enhet. Kör backendens egen kontinuerliga insamling (GUI:ns kontinuerliga läge) — se [Kontinuerligt läge](#continuous-mode---continuous) nedan. |
| **Intervall** | `--interval S` | Ett genomlopp var `S` sekunder (mätt från början av varje genomlopp), samma gränser. |
| **Snabbaste** | `--fastest` | Endast rådata-endast + det tilldelade DAQ-värdet + det kombinerade indexet; hoppar över beräkningarna för strålningsintensitet/reflektans/visning så att bilden visas snabbt. Innebär `--processing raw --force-daq`. Bearbeta senare de sparade `.daq` till kalibrerade produkter. |

Exportalternativ (kan kombineras med valfritt läge; alla delar samma GUI/SDK-ändpunkt):

| Flagga | Effekt |
| --- | --- |
| `--array-id ID` | Den matris som ska läsas in från (utesluts när endast en är ansluten). |
| `-o, --output DIR` | Utmatningsmapp (standard `output`, under den mapp där du kör kommandot). |
| `--format {tiff,tif,png,jpg,jpeg}` | Filformat (standard `tiff`). PNG/JPEG-format tar endast `display` och indexrenderingen; se [Exportnivåer för inspelning](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Enkel exportnivå (`raw`, `debayered`, `radiance`, `reflectance`, `display`), eller `all` (standard). |
| `--levels L1,L2,…` | Explicit delmängd av exporttyper (t.ex. `raw,radiance,reflectance`); **åsidosätter `--processing`**. |
| `--aligned` / `--no-aligned` | **Standard: justerad, precis som Chloros-appens avläsning av samma array är**(dess inställning för**Justerad** avläsning per array, aktiverad om du inte stänger av den). Varje medlems icke-råa export (`debayered`, `display`, `radiance`, `reflectance`, indexrenderingen) förvrängs till huvudkamerans bildram och beskäras till medlemmarnas gemensamma överlappning, så att filerna är pixelregistrerade och får samma storlek, beskärning och `Alignment*`-taggar som appens – mindre än sensorn (eller ROI) med medlemmarnas förskjutningar; under en ROI ombaseras transformationen till fönstret. `XMP-mapir:AlignmentJSON` anger `"applied": true` och ger `crop_roi`. Raw-filer förblir oförvrängda i sin ursprungliga storlek men bär med sig transformationen i metadata (`"applied": false`). `--no-aligned` sparar varje fil i sin ursprungliga storlek, oförvrängd. `--aligned` är standardinställningen. Oavsett om förvrängningen tillämpas eller inte registrerar varje medlemsfil sin kameras `Alignment*`-transformation så snart matrisen har en justeringsprofil — med undantag för en kamera som profilen inte kan justera (utesluten eller tvingad till identitet på grund av en misslyckad lösning) och referenskameran när ingen annan kamera är justerad: dessa har inga `Alignment*`-taggar. Om det inte finns någon profil för matrisen sparas inspelningen ojusterad, och CLI:n visar orsaken på en `Alignment:`-rad, precis som i alla andra fall där förvrängningen inte kunde tillämpas (medlemmar med olika upplösningar, en matris som inte är kabelsynkroniserad, en bildruta som profilen inte kan mappas på). Matrisens automatiska justering skapar profilen så snart exponeringen har stabiliserats; `align-calibrate` skapar en på begäran. |
| `--orient` | Vänd varje medlems **produkter**— `display`, `radiance`, `reflectance`, `index/`-renderingen och `composite/` — enligt kamerans orientering, efter justeringsförvrängningen: en produkt som förvrängs in i huvudbildens ram (standard) enligt**masterbildens**orientering, så att delarna förblir samregistrerade; en icke-justerad (`--no-aligned`, eller en inspelning där justeringen inte kunde tillämpas) enligt sin egen kameras orientering.**Standard: av – varje delfil förblir i sitt ursprungliga format**, precis som array-inspelningar alltid har gjort. `raw` och `debayered` förblir i ursprunglig ram oavsett. Varje fil registrerar den orientering som dess bilder har (se [Kamerans orientering](#hur-en-bildmapp-ser-ut)); XMP-filen `Alignment*` behåller den ursprungliga bildramstransformationen, som tillämpas före orienteringen, så att `process` (förvrängning, sedan orientering) återger den faktiska produkten. |
| `--index` / `--no-index` | Spara / hoppa över överlägget med vegetationsindex per kamera där ett sådant är konfigurerat. Standard: rendera det. |
| `--no-daq` | Hoppa över sidokar-filen `.daq` när ingen vald nivå använder DAQ (en ren rådata-inspelning). Som standard sparas avläsningen, så att bildrutorna kan bearbetas om till reflektans/index offline. `--force-daq` accepteras fortfarande (standard). |
| `--daq-tolerance-ms MS` | DAQ-matchningstoleransen för dessa inspelningar (standard 500 ms). |
| `--smart` | Vänta tills AE har stabiliserats på alla kameror innan utlösning (se [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF-pixelkomprimering. `none` (standard sedan 1.3.0; 1.2.0 skrev `deflate`) = okomprimerat — sensorns pixlar exakt som de läses av, ~6,3 MB per bildruta i full upplösning, och den snabbaste kontinuerliga överföringshastigheten; `deflate` = förlustfri zlib L1 + horisontell prediktor, ~4,1 MB per bildruta, långsammare. Båda är förlustfria och läses identiskt vid import. |

> **TIFF med engångsinskrivning + modellen med kontinuerlig överföringshastighet.**Varje fil skrivs**en gång**: pixlar, XMP-paketet, IFD0 Make/Model/CameraSerialNumber och EXIF-taggarna `ExposureTime` / `SerialNumber` i ett enda steg, utan att något skriver över det efteråt. `--continuous`X fångar upp på varandra följande synkroniserade grupper — varje inspelning tar gruppen efter den föregående, inklusive en som publicerades medan den inspelningen fortfarande sparades — så det körs i arrayens hastighet. DEFLATE håller Pythons GIL, så komprimerade skrivningar parallelliseras**inte**över skrivtrådarna per kamera; kontinuerlig inspelning i full upplösning med sensorns hastighet kräver okomprimerade skrivningar (standard)**och** en disk av NVMe-klass (~600 MB/s kontinuerligt för en array med 9 kameror vid ~10 fps). Samma inställning finns tillgänglig som `compression` på `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Kontinuerligt läge (`--continuous`)

`array-capture --continuous` (utan `--interval` eller `--smart`) överlämnar körningen till **backendens egen kontinuerliga inspelning** — den session där GUI:ns kontinuerliga läge körs. Backenden sparar varje synkroniserad grupp själv, med två inspelningar på gång och ingen begäran från CLI:n mellan grupperna, så den håller jämna steg med en snabb array där en begäran per inspelning hamnar efter (uppmätt till 73–77 inspelningar/s på en array med 9 kameror vid ett ROI på 512 pixlar). Varje exportinställning ovan gäller för varje inspelning.

- **Gränser:** `--count N` avslutas efter N inspelningar (en inspelning = en synkroniserad grupp: varje medlems filer); `--duration S` efter S sekunder. Utan någon av dessa körs programmet tills `Ctrl+C`, en full hårddisk, ett fel eller att uppsättningen stängs av. `Ctrl+C` avslutar backendens session och sedan skriver ut sammanfattningen.
- **Framsteg** var 2:a sekund, med `, N failed` och `, N groups skipped` tillagda när värdet är större än noll (en hoppad grupp är en som arrayen publicerade men som ingen insamling tog):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Ingen lista per fil.** Körningen avslutas med en sammanfattningsrad — insamlingar, hastighet, filer och utdatamappen — följd av `N file(s) failed to write.` / `N group(s) skipped.` när värdet är större än noll:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Avbryts** vid full hårddisk med `Continuous capture stopped: the drive is full.`, och vid ett fel eller om arrayen försvinner med `Continuous capture stopped: <reason>.` (båda på stderr).
- **Avslutningsstatus:** `0` när inspelningarna har sparats och alla filer har skrivits, inklusive `Ctrl+C`; `1` om någon fil inte kunde skrivas, vid full disk, ett fel, om arrayen försvinner eller om det inte finns någon inspelning alls.
- **Microsoft Defender:** när Defender använder en CPU-kärna eller mer för att skanna de filer som inspelningen skriver, skrivs följande ut en gång på stderr: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Se installationsprogrammets [Systeminställningar](#windows-exe).
- En kontinuerlig inspelning per array åt gången; en andra avvisas med `a continuous capture is already running for this array`. Över HTTP är sessionen `POST /api/camera/array/continuous/start` (`/api/camera/array/capture`-kroppen plus `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` och `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, med eller utan `--continuous`) och**`--continuous --smart`** behåller slingan per begäran: en insamlingsbegäran per genomgång — vid varje schemalagt tillfälle för Intervall, efter att AE har stabiliserats för `--smart` — med varje passets filer listade som i Single-läge, en `Done — N capture pass(es).`-rad i slutet och `--count N` som räknar pass.

### `array-record` — kombinerad indexvideo/GIF (övervakningskvalitet)

Spelar in matrisens **live-vy med kombinerat index** till en `.avi` (och valfritt en `.gif`). När GUI:ns kombinerade vy är öppen spelar den in den vyn precis som den visas. Om ingen vy är öppen renderar backend-modulen vyn själv: det index du skickar med `--expr`, eller som standard matrisens NDVI, så snart matrisen är justerad. Standard-NDVI använder de röda och NIR-banden närmast 660 nm och 850 nm, precis som livevyn gör. En GUI-vy som öppnas under inspelningen spelas in så länge den förblir öppen. Den avläser förloppet varannan sekund och avbryts vid `--duration`, `Ctrl+C`, eller när inspelaren avslutar sig själv.

En inspelning som inte skriver någon bildram misslyckas: inspelaren rapporterar `no-frames: …` (matrisen kan fortfarande vara i färd med att justeras eller stabilisera sin exponering) och kommandot **avslutas med `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `--array-id ID` | endast matris | Målmatris (uteslut om endast en är ansluten). |
| `-o, --output DIR` | `output` | Utmatningskatalog. En relativ sökväg anges under den mapp från vilken kör kommandot från. |
| `--fps F` | `10` | Bildfrekvens vid inspelning. |
| `--duration S` | tills Ctrl+C | Stoppa automatiskt efter `S` sekunder. |
| `--gif` | av | Skriv även en animerad GIF. |
| `--gif-only` | av | Skriv endast en GIF (ingen `.avi`). |
| `--expr EXPR` | matrisens NDVI | Indexuttryck som ska registreras när ingen GUI-vy är öppen, i matrisens bandnamn (t.ex. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; se [bandnamn](#index--vegetation-maths)). Ett kort färgnamn (`NIR`, `Red`) fungerar endast när ett band i matrisen har det. En öppen GUI-vy registreras istället så som den visas. |

### `array-burst` — rå-Bayer-seriebildtagning med hög bildfrekvens (analysklass)

Läser direkt från synkroniserad gruppbuffert i inspelningsslingan — **ingen kalibreringskedja, inget exiftool, ingen live-visning behövs** — så den körs med kamerans fulla bildhastighet. Skriver ut råbilder + ett manifest per bild + en `.daq` per distinkt DLS-avläsning under `<output>/bursts/<base>/`. Bearbeta offline (nästa kommando), eller ange `--build` för att göra det omedelbart vid stopp.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `--array-id ID` | endast array | Målarray. |
| `-o, --output DIR` | `output` | Utdatakatalog (utdata hamnar i `<DIR>/bursts/<base>/`). |
| `--duration S` | tills Ctrl+C | Automatisk stopp efter `S` sekunder. |
| `--max-frames N` | obegränsad | Automatiskstopp efter `N` råa bildrutor. |
| `--orient` | av | De produkter som en byggprocess skapar från denna bildserie roteras utifrån kamerans orientering — produkter per kamera roteras utifrån sin egen kameras, det kombinerade indexet enligt huvudkamerans — precis som `array-capture --orient` gör. Råbilderna förblir i sitt ursprungliga format; seriens `manifest.json` registrerar varje bilds orientering och inställningen. |
| `--build` | av | Efter stopp, bearbeta serien omedelbart på nytt (samma som `array-build-video`). |
| `--products …` | `combined:index` | Med `--build`: vilka videor som ska skapas (se nedan). |
| `--fps F` | `10` | Med `--build`: bildfrekvens för utgångsvideo. |
| `--save-tiffs` | av | Med `--build`: spara även kalibrerade TIFF-filer per bildruta (var och en med sin [unmix-klämmask](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on) när en pixel klämts; byggresultatet listar dem som `clamp_masks`). |
| `--gif` | av | Med `--build`: skriv även animerade GIF-filer. |

### `array-build-video` — ombearbeta en sparad bildserie offline

Synkroniserar varje råbild med närmaste sparade `.daq`-avläsning och kör den genom **samma strålnings-/reflektans-/indexkedja som importpipeline**, vilket resulterar i en eller flera videor.

`--products` är en kommaseparerad lista med `kind:level`-objekt, där `kind` ∈ `per_cam` | `combined` och `level` ∈ `radiance` | `reflectance` | `index`. Ett enskilt `level` (utan `kind:`) har som standardvärde `per_cam`. Standardvärdet är `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `--burst-dir DIR` | (obligatoriskt) | Sökväg till burst-mappen (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level`-lista, enligt ovan. |
| `--fps F` | `10` | Utgående videofps. |
| `--save-tiffs` | av | Spara även kalibrerade TIFF-filer per bildruta tillsammans med videon/videorna. |
| `--gif` | av | Skriv även animerade GIF-filer. |

**En video innehåller ingen klämmask** (en bildström, 8-bitars). Istället innehåller byggresultatet `clamp_fractions`: per videoprodukt, varje bildruts andel av de pixlar som avklämningen blandat ut inom det band som dess R/G/B-kanal innehåller — det `Chloros:Band{R,G,B}UnmixClampFraction` som den bildrutans produkt av strålning och reflektans skulle märkas med (för det kombinerade indexet, för varje enskild kamera). Körningen skriver ut den sämsta bilden per produkt på en `Clamp:`-rad. `--save-tiffs` lägger till maskerna per pixel för TIFF-filerna, i `clamp_masks/<level>/` bredvid TIFF-filernas `<level>/`-mapp.**En avbruten kompilering lämnar inga ofullständiga videor.** Varje video skrivs som `<name>.partial.avi` och får sitt slutgiltiga namn först när den är färdig, så en äldre video med det namnet finns kvar tills dess ersättare är komplett. Kör byggprocessen igen för att ersätta en `.partial.avi` som blivit kvar.

Körningen skriver ut varje video som den har skapat, därefter eventuella `Skipped:`-produkter (en begärd kamera/nivå – eller det kombinerade indexet – som inte genererade någon video, med orsaken angiven – t.ex. inget kalibreringspaket, ingen DLS-avläsning för reflektans eller inget rött/NIR-bandpar för det kombinerade indexet) samt eventuella `Note:`. En RGB-kamera som begärts för strålningsintensitet eller reflektans får istället sin visningsvideo, vitbalanserad när kamerans kalibreringspaket är tillgängligt; en kamera som renderas utan detta namnges i en `Note:`. Den **avslutas med ett värde som inte är noll** när en begärd produkt hoppades över eller inte byggdes enligt begäran (ett konfigurerat indexuttryck utvärderas inte offline); de videor som byggdes skrivs fortfarande ut och listas.

> **Välj rätt inspelningsverktyg.** `array-record` är av *övervakningskvalitet* — den spelar in den renderade livekompositen: GUI:ns kombinerade vy, eller, om inget GUI är öppet, backendens egen NDVI-vy (eller `--expr`). `array-burst` → `array-build-video` är av *analyskvalitet* — den sparar råa sensordata med full hastighet och rekonstruerar därefter kalibrerade strålnings-/reflektans-/indexvideor, utan att någon livevy krävs.

### Mono (M3M) enkelbandskameror

**M3M**-serien är den monokroma motsvarigheten till Bayer**M3C**: ett smalbandigt interferensfilter per kamera (`M3M-<lens>-F<wavelength>`, t.ex. `M3M-L87-F685`), så sensorn levererar ett**enda gråskaleband** utan Bayer-mosaik. Det finns inget att demosaikera, ingen kanalöverskridande störning att separera och ingen vitbalans att ställa in – hela färgprocessen för RGB-visning gäller helt enkelt inte.

Vad det innebär i CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**upptäcker en monokamera och**hoppar över med ett enradigt meddelande** istället för att skicka meningslösa inställningar. De fungerar fortfarande normalt med en RGB-/Bayer-M3C-kamera i samma session.
- **Ett Bayer-pixelformat anger dess djup.** En M3M erbjuder endast monofärgformat, så ett Bayer-format från `--pixel-format` eller `--preset` ställer in monofärgformatet med samma djup: `BayerRG8` och `--preset high_speed` ger `Mono8`; `BayerRG12`, `high_quality` och `triggered` ger `Mono12`; ett packat namn förblir packat (`BayerRG12p` ger `Mono12p`). Filerna och `lattice format` benämns Mono-formatet. En matris som blandar M3C- och M3M-kameror antar ett format på samma sätt: Bayer på varje M3C, dess monodjup på varje M3M. Ett namn är dess bitdjup: detsamma gäller tvärtom, så en M3C placeras aldrig i ett Mono-format (`--pixel-format Mono12` är `BayerRG12` på en M3C, oavsett vilken kamera som är matrisens huvudkamera), och `lattice format --set` mappas på samma sätt och skriver ut det format som ställts in.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** fungerar fortfarande — strålnings- och reflektansvärdena är radiometriska kartor *per band* och är perfekt definierade för ett band. Monokrombildrutor har en **identitets**-sensorresponsmatris (ingen 3×3-avblandning), så planet passerar genom kalibreringsberäkningarna oförändrat.
- **En enskild monokamera kan inte generera ett vegetationsindex.**NDVI/NDRE/etc. kräver minst två band (t.ex. Red + NIR). För att få fram ett index från monohårdvara ska man rikta**flera** M3M-kameror mot olika våglängder, sammanfoga dem till en multibandstack och beräkna indexet *utifrån den*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` öppnar kamerorna själv (en [tillfällig session](#multi-camera-transient-sessions)), så anslut ingenting först: `--serials`, annars aktiveras profilens kameror (se [Index / Vegetationsberäkningar](#index--vegetation-maths)). Så länge Chloros har kontroll över någon kamera (GUI:n, `cam-connect`, `array-connect`) avvisas den med HTTP-kod 409 `CAMERA_HELD` och avslutas `1`; frigör dem med `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

`--channel`-symbolerna måste **exakt** stämma överens med förinställningens kanalnamn (skiljer på versaler och gemener; NDVI-kanalerna skrivs med gemener `red`,`nir` — se `--list-presets`), och bandnamnen ska ange ett band i den anpassade stapeln (offline-läget accepterar även 0-baserade bandindex, t.ex. `--channel red=0 --channel nir=1`).

Det som skiljer banden åt i hela stapeln är tokenet `M3M` i modellsträngen (det förekommer aldrig i en `M3C`-sträng), vilket visas i GUI/SDK som `is_mono`.

---

## Konfiguration och inställning av värd-NIC (LATTICE-arrayer)

LATTICE-kameror strömmar GVSP via värdens Ethernet-adapter, så för arrayer med flera kameror är adapterns **drivrutin**och**mottagningsringens storlek** lika viktiga som länkhastigheten. Felaktiga inställningar visas som en `FRAMES WILL DROP` / `Reduce ROI to enable`-varning i panelen Arrayinställningar (samt i `lattice network-analysis` / SDK:s `analyze_array_network()`), även när kamerorna i sig fungerar som de ska.

### USB 10GbE-adaptrar — Realtek RTL8157 (“Realtek USB 10GbE Family Controller”)

Inställningarna nedan gäller Windows-drivrutinens egenskaper. I Linux har en USB-adapter också en begränsning (400 MB/s på x86_64, 200 MB/s på NVIDIA Jetson) (se [Array fps & burst-modell](#array-fps--burst-model)), och kärnans socket-mottagningsbuffertar spelar roll för varje adapter — se [Linux-värdar](#linux-hosts--socket-receive-buffers).

| Post | Nödvändigt värde | Varför det är viktigt |
| --- | --- | --- |
| **Drivrutinsversion**|**≥ v10.67 (jan 2026)**, INF `rtump64x64sta.inf` | Den äldre**2016**-drivrutinen (v10.65, `rtump64x64.inf`) hanterar avstängning och felkontroller felaktigt med**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**vid avstängning/omstart/viloläge. Övergången hänger sig (~5 minuters timeout), användaren tvingar fram en avstängning, och upprepade oriktiga avstängningar**skadar WMI-repositoriet**(PowerShell/verktyg börjar misslyckas med `Invalid class`) och**blockerar USB-stacken** vid nästa uppstart (nätverkskortet aktiveras inte; USB-enheter slutar räknas upp). Uppdatera från realtek.com (eller dongelleverantören) innan du förlitar dig på korrekta omstarter. |
| **Mottagningsbuffertar**— nyckelord `ReceiveBufferLen` |**256**(drivrutinens maxvärde) | Nätverkskortets RX-ring. Drivrutinens standardvärde på**32**lämnar endast ~0,26 MB användbar ring – alldeles för liten för en multikamera-burst – så arraypanelen rapporterar `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` och blockerar anslutningar. Vid**256**är ringen stor (**~13,5 MB uppmätt på labbets 10GbE-värd**), vilket ger RX-pipeline verkligt utrymme för GVSP-burst med flera kameror. (Huruvida en given konfiguration faktiskt *ansluts* avgörs av två kontroller — den **drain-aware**tillträdeskontrollen och kontrollen av**aggregerad överteckning** — inte en ren jämförelse mellan burst och ring; se [Array fps & burst-modell](#array-fps--burst-model).) |
| **Mottagande URB:er**— nyckelord `PendingReceives` |**64** (max) | USB-begäranblock under överföring; öka tillsammans med mottagningsbuffertar för att absorbera burstar. |
| **Jumbo Frame** — nyckelord `*JumboPacket` | **9014** | Behövs för 9000-byte GVSP-paket (6× färre paket/ram än 1500). |

> ⚠️ **En uppdatering av nätverkskortdrivrutinen ÅTERSTÄLLER dessa avancerade egenskaper till standardvärdena.**Efter uppdatering eller byte av drivrutinen måste du**tillämpa** `ReceiveBufferLen=256` och `PendingReceives=64`, annars kommer arraypanelen att stängas av igen även om ”inget har ändrats i hårdvaran”. Detta är den främsta orsaken till att en rigg som tidigare fungerade plötsligt vägrar att ansluta.

Tillämpa från en **administratörs** PowerShell (ersätt med ditt nätverkskortnamn, t.ex. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` gäller USB 10GbE-kort.** Nu detekteras korttypen och rätt nyckelord för mottagningsringen ställs in: `*ReceiveBuffers`→2048 för PCIe-nätverkskort (Intel I219, m.fl.), eller `ReceiveBufferLen`→256 + `PendingReceives`→64 för Realteks **USB** 10GbE-kontroller (som inte exponerar `*ReceiveBuffers`). Värdena begränsas till det maximivärde som respektive drivrutin rapporterar (`NumericParameterMaxValue`), så att det aldrig skrivs in ett värde utanför intervallet. Kör det från en **administratörsterminal**; precis som alla registerbaserade justeringar träder ändringen i kraft efter att adaptern startats om eller efter en omstart av datorn. De manuella `Set-NetAdapterAdvancedProperty`-kommandona ovan är fortfarande ett bra alternativ — de tillämpas direkt (omkopplar adaptern) utan omstart.

### Linux-värddatorer — mottagningsbuffertar för socket

I Linux, inklusive NVIDIA Jetson, är kärnans mottagningsbuffertar för socket lika viktiga som nätverkskortets mottagningsring. Med kärnans standardinställning (`net.core.rmem_max` / `net.core.rmem_default` = 212 992 byte) överflödar en kameras burst av GVSP-paket ström-socketen innan Chloros hinner läsa den. Paketen tappas bort i kärnan medan nätverkskortet i sig inte rapporterar något förlust, och en array förlorar bildgrupper och sänker sin bildhastighet. Mätt på en array med fyra kameror anslutna via en USB 10 GbE-adapter ledde en höjning av båda värdena till 32 MB till att arrayens bildhastighet ökade från 4,96 till 7,52 fps efter anslutning, och en kontinuerlig råinspelning ökade från 4,7 till 7,6 inspelningar/s, utan att några paket tappades bort i kärnan.

- **.deb-paketet ställer in dem.** När paketet installeras skrivs `/etc/sysctl.d/60-chloros-gige.conf` (båda vid `33554432`) in och tillämpas omedelbart. Ett värde som redan är inställt på ett högre värde lämnas oförändrat.
- **`lattice network` kontrollerar dem.** När något av värdena är lägre listas `socket receive buffers too small for GigE Vision (…)` under *Issues found*, och backendens logg visar samma varning vid den första anslutningen av en kamera eller ett array.
- **För att höja dem manuellt** (fram till nästa omstart; paketets fil tillämpar dem vid varje uppstart):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` höjer och bibehåller dem också, men endast när backend körs som root. En backend som körs som din användare (en som startats med `chloros-backend`, eller [`chloros-backend@<user>`](#running-the-backend-as-a-service-headless)) kan inte ändra kärninställningarna och skriver istället ut raden `sudo sysctl`.

**Den automatiska adapterjusteringen påverkar endast anslutna adaptrar.** Vid den första kameraanslutningen justerar en backend som körs som root de trådbundna adaptrarna vars mottagningsring är liten. Nu justeras endast adaptrar med en aktiv länk, så en frånkopplad inbyggd port behåller sin MTU, och mottagningsbuffertarna lämnas orörda. `lattice network` listar inte längre en frånkopplad adaptersMTU som ett problem. `lattice network`, `--fix`, `lattice probe` och adapterjusteringen beaktar endast trådbundna Ethernet-portar, inte en Jetsons CAN-buss (`can0`) eller länkarna i USB-enhetsläge (`usb0`, `usb1`) som den erbjuder en värddator; en USB-Ethernet-adapter för kameror räknas fortfarande.

### Nätverksgrunder (alla LATTICE-länkar)

- **Adressering:** länklokal `169.254.0.0/16` (GigE Vision LLA). Värddatorn använder en statisk adress `169.254.x.x/16`; kameror och DAQ-E tilldelar sig själva adresser inom samma intervall. Ingen DHCP/gateway krävs.
- **Paketstorlek:**föredra jumbo (9000), men låt den automatiska sonden hitta den – den ser förbi kamerans ICMP-begränsning på 1500 byte via en GVSP-sond, så den hamnar på jumbo där kabeln faktiskt klarar det.**Varje kamera mäts en gång per backend-session och igen när dess länk ändras (en annan adapter, en annan länkhastighet eller en länk som har brutits, som vid byte av switch); vid återanslutning återanvänds resultatet. Efter en ändring bakom en switch som har varit igång, starta om backenden för att mäta igen.**En första anslutning kan ibland ta några minuter vid denna kontroll, och loggen anger detta när det inträffar. Använd `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` endast när du vet bättre än sonden, och föredra per kommando framför permanent: en sådan inställning hoppar över sonden, så om vägen faktiskt inte klarar 9000 kommer**varje** avlyssning att ta timeout med `SC_ERR_TIMEOUT -1011` (se [Miljövariabler](#environment-variables)).
- **RX-ringen skalar med `ReceiveBufferLen`:**vid standardvärdet `32` är den användbara ringen ~0,26 MB (för liten för vilken multikamera-burst som helst); vid maxvärdet `256` är den stor (~13,5 MB uppmätt på laboratoriets 10GbE-värd), vilket ger verkligt utrymme. Huruvida en konfiguration ansluts avgörs sedan av den dräneringsmedvetna tillträdeskontrollen**och** den aggregerade överteckningskontrollen nedan — inte en ren jämförelse mellan burst och ring.

### Array-fps och burst-modell

Hur man läser panelen Array-inställningar (och `lattice analyze-array` / SDK:s `analyze_array_network`):

- **Burst summeras per kamera enligt varje kameras faktiska pixelformat.**Mono**M3M**-kameror strömmar**Mono12 (2 B/px)**, eller med `--pixel-format` det Mono-formatet för dess djup (`BayerRG8` är `Mono8`, 1 B/px);**M3C**-Bayer-kameror strömmar 8- eller 12-bitars (M3C-firmware sänder automatiskt BayerRG12 även när BayerRG8 begärs). Så en bildruta i full upplösning från 4 kameror är**~12,6 MB om alla är 8-bitars, men ~25 MB med tre 12-bitars monokameror**. Projektionen avgör varje kameras format utifrån dess modell (identitetscache), så att dataströmmen stämmer överens med vad kabeln faktiskt överför – inte en generisk antagande om BayerRG8.
- **En USB-Ethernet-adapter har en övre gräns på 200 MB/s oavsett vad som står på typskylten.** Effektivitetstabellen som omvandlar en länkhastighet till ett kontinuerligt värde är härledd från PCIe; ett USB-nätverkskort anger sin *Ethernet*-länkhastighet men begränsas av USB-bussen och dess drivrutin. En USB 10GbE-dongel brukade uppnå ~1063 MB/s ”kontinuerligt” – ett värde som aldrig testades – och den resulterande hastighetsbegränsningen förstörde 6–18 % av ramarna samtidigt som den fortfarande rapporterade ett normalt mål-fps. USB-anslutna nätverkskort har nu en absolut begränsning: **200 MB/s** på Windows och på NVIDIA Jetson, **400 MB/s** på Linux x86_64 (på Linux tilldelas ett nätverkskort i en USB 2.0-port **30 MB/s**, och enligt specifikationerna ska det anslutas till en USB 3-port) (begränsningen ligger i värddatorns USB-buss och drivrutin, så den skalar inte med den angivna kapaciteten; ett USB 1 GbE-nätverkskort uppnår ~80 MB/s och påverkas inte). På Linux identifieras adaptern via sysfs (med fallback till `ethtool -i`:s `bus-info`). Varje siffra är uppmätt: i Windows förstörde samma uppsättning med fyra kameror 6–10 % av bildrutorna vid hastigheter över 200 MB/s; i Linux x86_64 fungerade samma adapter felfritt vid alla hastigheter upp till 480 MB/s (273 MB/s levererades, vilket är kamerornas egen gräns). På NVIDIA Jetson fungerade samma adapter felfritt endast vid 200 MB/s och tappade bildrutor sporadiskt vid högre hastigheter. `wire_ceiling_source` i kapacitetsloggen anger detta i text, och `nic_is_usb` markerar det. Åsidosätt detta på något av sätten med `--wire-ceiling-mbps`.
- **Admittansen är dräneringsmedveten, inte hel-burst-vs-ring.** En samtidig burst behöver endast passa in i den *tillfälliga backloggen* = `max(0, Σ per-cam arrival − host drain) × emit_window`, inte hela bursten. I en struktur med snabb värd och långsam kamera (en **PCIe**10G-värd + 4× 1 GbE-cams: ankomst ≈ 320 MB/s, tömning ≈ 1063 MB/s) töms värden snabbare än kamerorna fylls, backlog ≈ 0, så full-res sim-emit**släpper igenom**även om bursten på 25 MB överskrider ringen på 13,5 MB. Placera samma fyra kameror bakom en**USB** 10GbE-adapter på Windows, så blir utmatningen 200 MB/s, inte 1063 — ankomsten överstiger den, och förlusten visar sig som korrupta ramar snarare än som en lägre bildfrekvens. På en 1 GbE-värd gör kamerornas DLThr-golv på 31,25 MB/s att ankomsten överstiger avloppet → den **blockerar** korrekt (för *denna* klass av blockeringar, minska ROI eller använd binning ≥ 2). Admittans är en av **två** anslutningsgrindar – den andra är den sammanlagda kontrollen av överteckning nedan.
- **Beräknad fps är den långsammaste kamerans cykel.**Vid samtidig sändning skickar varje kamera sin bildruta samtidigt, så matrisen cyklar med den längsta av dess långsammaste medlems sensoravläsningstid och sändningstid (bildrutans byte ÷ dess sändningshastighet: kamerans**åtkomstlänk**, 1 GbE ≈ 80 MB/s, eller dess andel av värdens bandbreddstak när detta är lägre). Varje exponering överlappar den föregående bildramens avläsning, så exponeringen ingår inte i cykeln: automatisk exponering fyller den period som sändningshastigheten ger, höjer sedan förstärkningen, och en mörk scen sänker inte längre sändningshastigheten (i 1.2.0 gjorde den det). Endast en manuell exponering som är längre än perioden saktar ner utlösningen (se [Exponering på en arraymedlem](#exposure-on-an-array-member)). Att lägga till kameror saktar inte ner arrayen förrän deras trådandelar krymper. Vid seriell utsändning (`slip-emit-and-capture`) summeras istället kamerornas utsändningstider, och varje medlems exponering vägs in i förskjutningen. Uppmätt vid full sensor, 12-bitars, på en array med 9 kameror med en PCIe 10 GbE-värd: 10,7 fps, eller 14,0 fps med [packad överföring](#packed-12-bit-transport---packed).
- **Efter en kontinuerlig inspelning visar projektionen vad den höll.** En inspelning på 30 sekunder eller mer med automatisk hastighet registrerar den hastighet som den stabiliserades på vid den arraystorleken på den här datorn; hastighetsregulatorns korta försök med en snabbare hastighet utelämnas. Från och med då anger raden för `array-connect` och panelen Arrayinställningar den hastigheten när den ligger under modellens, tillsammans med vad som begränsade den: `clock-bound` (matrisen kördes med den hastighet den stabiliserades på, grabb-slingan väntade på sin triggerklocka), `loop-bound` (grabb-slingan hade ingen tid kvar), `capture-bound` (inspelningen hamnade efter och hoppade över grupper) eller `measured` (inget anger vilket). Modellens siffra finns i `wire_fps_max`. Arrayen stiger fortfarande till det som nätverket klarar av.
- **En matris fungerar i vilket nätverk som helst – det är bildfrekvensen som avgör.**Varje kamera får en jämn andel av den**kollisionssäkra nätverksgränsen**(`sustained × sim_emit_factor`), synkroniserad med GevSCPD, och alla exponerar fortfarande vid samma utlösare. På en långsam kabel är andelen liten och bildfrekvensen sjunker i takt med den: 8 kameror med full upplösning och 12 bitar på 1 GbE utan jumbo-ramar får 52 / 8 = 6,5 MB/s vardera, cirka**1 fps**(cirka 1,3 fps packat), medan en 10G-värd kör dem med 11–14. Om kamerorna inte kan synkroniseras så långsamt – läs av varje kameras GevSCPD-intervall för att ta reda på detta – växlar arrayen till**seriell**sändning (`slip-emit-and-capture`): kamerorna turas om på kabeln med full länkhastighet, utlöses en efter en, så varje ram anländer fortfarande men medlemmarna**exponeras inte samtidigt**. `analyze-array` och utdata från connect anger läget; sätt `--force-tier sim-capture-sim-emit` till för att avvisa fallback. Bildstorlek påverkar inte denna delning, så**binning och ett mindre ROI ökar bildfrekvensen men påverkar inte om en matris passar**. Endast en kabel som inte kan hantera matrisen ens seriellt är**överbelastad**— t.ex. betydligt fler kameror än vad en kabel klarar av, eller många andra kameror som strömmar på den — och där är symptomet GVSP-paketförlust, inte en lägre hastighet: `analyze-array` nollställer siffrorna för uppnåelig bildfrekvens och visar `** OVER-SUBSCRIBED**`, anslutningen fortsätter med en tydlig varning, och en HTTP-anslutning med `pin_resolution: true` (strikt läge) **avvisas** istället. Lösningar är färre kameror, jumbo-ramar från ände till ände eller ett snabbare nätverkskort. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` nedgraderar avvisningen till en tydlig varning för testarbete — se [Miljövariabler](#environment-variables).

### Arrayhälsa – vilket delsystem tappar ramar

En ansluten arrays `GET /api/camera/array/<array_id>/capability` innehåller ett live-
`health`-block, som omvärderas i ett rullande **10-sekunders** fönster. Den delar upp ramförlusten
i de två orsakerna som kräver motsatta åtgärder, istället för att rapportera en ”ofullständig”
frekvens som inte specificerar någon av dem:

| Fält | Vad det betyder | Vilket delsystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriell port) | Ramen **anlände men var strukturellt felaktig**— GVSP-paketförlust. |**Nätverk**: bandbredd, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (per serienummer) | Ramen **kom aldrig alls**— kameran utlöste inte, eller så skickades inget alls. |**Utlösare/synkronisering**: M8-kabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Den sämsta kamerans andel för respektive orsak. | — |
| `per_cam_rate_pct` | Sammanlagd andel ofullständiga bilder per kamera (båda orsakerna tillsammans). | — |
| `stable_for_seconds` | Hur länge varje kamera har legat under 0,01 %. | — |

Över 5 % loggar backend-systemet en rad med koden `[array-health <id>] WARN` som anger uppdelningen — vid
det första avbrottet, vid en förändring av allvarlighetsgraden, en gång per minut så länge det kvarstår och en gång när
det åtgärdas. Den felaktiga halvan skriver `[gvsp-corrupt <SN>]` till backend-loggfilen vid den
första träffen per kamera och orsak, därefter en sammanfattning var 60:e sekund; appens loggpanel visar istället raden
raden `[array-health <id>]` istället. Varje utvärdering hamnar fortfarande i backend-loggfilen;
räknarna rör sig vid varje buffert oavsett vad som skrivs ut.

Samma post rapporterar det nummer som hela allokeringen hänger på:

| Fält | Vad det betyder |
| --- | --- |
| `wire_ceiling_mbps` | Värdens gällande kontinuerliga bandbreddsbudget, MB/s. |
| `wire_ceiling_source` | Varifrån det talet kommer, i ord – t.ex. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (på Linux x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) eller `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` när `--wire-ceiling-mbps` (eller fältet **Wire Budget** i GUI:n) ställer in det. |
| `nic_is_usb` | `true` för en USB-Ethernet-adapter — se USB-begränsningen ovan. |

**Tolkning:** ett värde på `gvsp_corrupt_rate_pct` som inte är noll och `never_arrived_rate_pct` på 0
betyder att triggning och kabelsynkronisering är perfekta och att 100 % av förlusten ligger på nätverksvägen
— sänk `--wire-ceiling-mbps` och anslut på nytt. Det omvända mönstret pekar istället på
synkroniseringskabeln eller triggerledningen.

> **`--target-fps` är inte avgörande för korrupta ramar.** GevSCPD-takten skrivs
> en gång vid anslutning, så att sänka utlösningsfrekvensen ändrar arbetscykeln och inte
> burstfrekvensen för samtidig sändning. En uppmätt sänkning av efterfrågan med 5× gav ingen förbättring;
> att sänka trådens tak från 240 till 200 MB/s tog samma rigg från 10,4 %
> korrupta ramar till 0,00 %.

> **En igångvarande array kan inte omplanera sig själv på LATTICE (M3C / M3M)-firmware.** Den
> kan inte åtgärda detta själv; koppla bort och anslut på nytt så att anslutningstidsväljaren kan
> omplanera med den nya gränsen.

### Symptom → åtgärd

| Symptom (Arrayinställningar / anslutning / `analyze_array_network`) | Orsak | Åtgärd |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` återställs till 32 (vanligtvis efter en drivrutinsuppdatering) | Ställ in `ReceiveBufferLen`→256, `PendingReceives`→64; öppna panelen igen (starta om backend om den har cachat den gamla ringstorleken) |
| Omstart/avstängning hänger sig; senare `Invalid class` WMI-fel, nätverkskortet aktiveras inte, USB-enheter saknas | Gammal Realtek USB 10GbE-drivrutin från 2016 → BSOD `0x9F` → tvingade avstängningar | Uppdatera nätverkskortets drivrutin till ≥ v10.67 (2026), och tillämpa sedan inställningarna för mottagningsringen ovan på nytt |
| Arrayen ansluter med full sensor men låg bildfrekvens | Auto behåller full sensor och låter frekvensen sjunka till vad kabeln klarar | Välj ett av de snabbare alternativen i anslutningsutgångslistan (`--binning 2` / `4`, eller en lägre `--width`/`--height`), aktivera jumbo-ramar eller uppgradera länken |
| Arrayen rapporterar en korrekt mål-fps men levererar bara en bråkdel av den; `health.gvsp_corrupt_rate_pct` ≠ 0, `never_arrived_rate_pct` 0 | Värdens beräknade bandbredd överskattar vad den faktiskt klarar av (vanligt vid en USB-Ethernet-adapter, en smal PCIe-lane eller en delad nätverksstruktur) | Anslut på nytt med ett lägre `--wire-ceiling-mbps`-värde och kontrollera hälsoblocket igen. **Inte** `--target-fps` — GevSCPD-pacing är fastställd vid anslutning |
| Linux: en array tappar ramgrupper och sänker sin ramhastighet medan nätverkskortet inte rapporterar några bortfall; `lattice network` listar `socket receive buffers too small` | Kärnans mottagningsbuffertar för socklar är inställda på standardvärdet, vilket gör att kamerapaket tappas bort i kärnan | Höj `net.core.rmem_max` / `rmem_default` till 32 MB — se [Linux-värdar](#linux-hosts--socket-receive-buffers). .deb-paketet gör detta vid installationen |
| Kameror saknas i publicerade grupper; `health.never_arrived_rate_pct` är större än noll, `gvsp_corrupt_rate_pct` 0 | Utlösare/synkroniseringsväg — kamerorna utlöses inte, det är inte ett nätverksproblem | Kontrollera M8-synkroniseringskabeln och `--line`; bekräfta att alla enheter är aktiverade (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` överskridet i `analyze-array`, eller anslutningsvägran i strikt läge (`pin_resolution`; `array over-subscribes the wire`) | Kabeln klarar inte av att överföra data vid någon av de hastigheter som kamerorna når, vare sig samtidigt eller i tur och ordning — betydligt fler kameror än vad kabeln kan hantera, eller andra kameror som strömmar över den | Färre kameror, jumbo-ramar från ände till ände, eller ett snabbare nätverkskort. **ROI/binning hjälper INTE** (delningen är oberoende av ramstorlek). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` åsidosätter på testbänken (accepterar paketförlusten) |
| Arrayen ansluts i `slip-emit-and-capture` (seriell) på ett långsamt nätverk | Kamerorna kan inte synkroniseras tillräckligt långsamt för att dela kabeln samtidigt, så de turas om; bildramarna exponeras inte i samma ögonblick | Jumbo-ramar eller ett snabbare nätverkskort återställer samtidig inspelning; `--force-tier sim-capture-sim-emit` insisterar på det (och förlorar istället paket) |

---

## `chloros-cli daq`

Kommandon för spektralsensorer. `pool-*`-kommandona är tunna HTTP-klienter som styr sensorn via backendens persistenta pool, så de behöver en igångvarande backend (se [Starta backenden](#start-the-backend)). Backenden äger transporten, så CLI- och SDK-skripten delar ett aktivt handtag istället för att slåss om den seriella porten. `pool-discover` är det enda sättet att ta reda på en DAQ-M:s BLE MAC.

> **`chloros-cli daq --help`** (eller `-h`, eller `daq help`) listar `pool-*`-underkommandona (på Linux tillsammans med de direkt-hårdvarukommandon som ingår i kompileringen), och `chloros-cli daq <subcommand> --help` (eller `daq help <subcommand>`) anger alternativen för varje underkommando. De omfattar anslutning, strömning, inspelning av kalibrerade `.daq`-filer, byte av kapacitetsprofiler och koppling av en DAQ till en kamera, samt för en DAQ-E dess nätverksinställningar, PTP, omstart och firmwareuppdateringar. DAQ-enheten kan även styras från Python med `chloros_sdk.connect_daq_sensor()`, som använder samma gemensamma sökväg.
>
> **Använd `pool-*`-kommandona på alla plattformar.** De fungerar på samma sätt i Windows, Linux och Jetson. I Windows är är `daq discover` och `daq list` alias för `pool-discover`, och alla andra `daq`-kommandon som inte är `pool-*` avslutar `1` med `DAQ direct-hardware commands require the full daq package`. På Linux och Jetson är `daq discover`, `daq list` och de övriga kommandona som inte är av typen `pool-*` och `daq` DAQ-verktygets direktahårdvarukommandon: de öppnar sensorn i CLI:s egen process, utanför backendens pool (alltså inte på en sensor som backenden, GUI:n eller ett skript hanterar), och där tar `daq discover` emot `--only` och `--timeout` men inte `--json`. Denna referens dokumenterar endast kommandona `pool-*`.

### Arbetsflöde för första anslutning av DAQ-sensor

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referens för `pool-*`

| Underkommando | Syfte |
| --- | --- |
| `daq pool-connect` (smart-detect) | Öppna en sensor i backend-poolen. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U på en specifik seriell port. |
| `daq pool-connect --ble` | DAQ-M via BLE, automatisk skanning av MAC-adress. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M via BLE med en känd MAC-adress (förutsätter `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E via Ethernet med ett känt värdnamn eller IP-adress (förutsätter `--eth`). |
| `daq pool-connect --eth` | DAQ-E via Ethernet, värd upptäckt automatiskt (mDNS + ARP som reserv; fungerar även med en tom ARP-cache i Windows och Linux). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Justera integrationsfönster / AE-status. |
| `daq pool-connect --no-stream` | Anslut men starta inte strömningen ännu (återuppta med `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil för kapacitetskorrigering. Standardinställningen på backend är `sunshine_cosine`; `none` är en ren sensor. En [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) ignorerar den (fast diffusor). **Sensorerna levereras utan kåpa, så namnge den kåpa du monterat:** standardvärdet registreras som antaget (`cap_id_source=auto_default`) och bearbetningen varnar för detta, eftersom en oskyddad DAQ-U under solskyddskorrigeringen ger ett värde som är ~30 gånger för högt. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Sök igenom varje transport efter sensorer som du kan ansluta till, utan att ansluta. **Så här hittar du en DAQ-M:s BLE-MAC.** `--timeout` gäller per transport (standard 5 s, 1–20). Sensorer som redan är öppna i poolen listas inte – en ansluten DAQ-M slutar sända annonser – så använd `pool-list` för dessa. `usb` listar endast USB-serieenheter: datorns egna serieportar (`COM1`, Linuxs `/dev/ttyS*`) kan inte vara sensorer och utelämnas. |
| `daq pool-list` | Visa alla sensorer i backend-poolen. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Koppla en kamera eller en matris till en DAQ, precis som GUI:ns val av ljussensor gör: dess bildtagningar använder, matchar och sparar den DAQ:ns avläsningar. `SENSOR` är ett sensor-ID från `pool-list` eller den adress där DAQ:n nås. `--clear` (ingen `SENSOR`) avbryter kopplingen, och registreringarna återgår till den senaste DAQ:en. Kopplingen varar tills backend-modulen startas om, eller tills den kameran, matrisen eller DAQ:en kopplas bort: en frånkoppling (här, från SDK:et eller i appen) bryter alla kopplingar till den hårdvara som kopplas bort, oavsett vem som har ställt in dem, och matrisens medlemmar kopplas bort tillsammans med matrisen. Kameran måste vara ansluten, i det öppna projektet eller ha öppnats på den här datorn tidigare, och matrisen måste vara ansluten eller finnas i det öppna projektet; alla andra serienummer eller ID:n avvisas på grund av namnet och ingenting binds. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Frigör – oavsett vem som öppnade sensorn, så frigörs även en sensor som ett kraschat skript lämnat kvar i poolen (eventuella inspelningar på den avslutas först). En sensor som inte finns i poolen har inget att frigöra: den skriver ut `ID: already released (not in the pool)` och avslutas med `0`; en frigivning som backend vägrade eller inte bekräftade avslutas med `1`. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De senaste N spektrumramarna (standard 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Återuppta/pausa strömning. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Starta/stoppa en .daq-inspelning. `-o` är som standard inställt på `~/Documents/DAQ Live View` på backend-datorn (en relativ sökväg under den mapp där du kör kommandot); `--duration` (`-d`, standard `0` = tills `--stop`). `--confirm-after S` varnar när inget mätvärde har kommit in S sekunder efter start (standard 20; 0 stänger av funktionen); med `--duration` `--heartbeat S` skriver ut antalet avläsningar var S:e sekund och varnar när antalet slutar öka (standard 30; 0 stänger av funktionen). En inspelning på en DAQ-E som tystnar fortsätter i samma fil så snart backend har återanslutit den, och `--stop` listar luckorna ([En DAQ-E som slutar leverera](#a-daq-e-that-stops-delivering)). Ctrl+C avslutar en `--duration`-inspelning i förtid och rapporterar detta; utan `--duration` fortsätter inspelningen att köras efter Ctrl+C (avsluta `130`) tills `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Byt kap-korrigeringsprofil under körning. `--cap-id` krävs; `--cap-id none` tar bort kap-korrigeringen (en sensor utan lock). En DAQ-E-S behåller inställningen för `as_recorded` oavsett vad som är inställt. |
| `daq pool-enable-ptp --sensor-id ID` | Slå på PTP igen för en DAQ-E vars PTP är avstängd. Inställningen sparas på enheten, så en enhet som lämnats avstängd förblir avstängd — dess tidsstämplar är inte PTP-synkroniserade — tills detta kommando körs. `pool-list` markerar en sådan enhet (`!! PTP is switched OFF`) och skriver ut detta kommando för den. Chloros slår aldrig på en enhet igen av sig själv, eftersom avstängning kan vara avsiktlig. |
| `daq pool-info --sensor-id ID [--json]` | En DAQ-E:s egna inställningar: dess firmware (och om denna Chloros-version innehåller en nyare version), dess adress och hur den erhölls (DHCP / link-local / statisk), den sparade nätverksinställningen tills sensorn kör den (en som sparats med `--no-reboot` inkluderad), ström vid uppstart och PTP. På en sensor vars ström inte når denna dator läser den av sensorn vid en ny anslutning och meddelar detta. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Ställ in en DAQ-E på DHCP (`--auto`) eller en statisk adress (`--mask` standard `255.255.255.0`). Sensorn startas om för att tillämpa inställningen och återansluts med de inställningar den hade tidigare, och väntar upp till `--timeout` sekunder (standard 45) på detta; `--no-reboot` sparar endast inställningen. Se [DAQ-E-enhetsinställningar](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Om en DAQ-E ska börja strömma vid uppstart utan ansluten programvara (sparas på sensorn). Chloros startar strömmen själv oavsett. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Starta om en DAQ-E och anslut den igen med tidigare inställningar (väntar upp till `--timeout`, standard 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Flasha DAQ-E-firmware som ingår i denna Chloros-utgåva till en sensor som kör en äldre version, och anslut den sedan igen (tar ungefär en minut; stäng inte av strömmen; `--timeout` standard 45 s för att den ska återgå). En sensor som redan är uppdaterad lämnas ifred. |

### En DAQ-E som slutar leverera data

En DAQ-E i poolen som inte levererar något spektrum under 20 sekunder återansluts av backend-systemet. Detta sker genom att strömbrytaren som försörjer sensorn slås av och på: sensorn startar om och börjar inte strömma data igen förrän den får instruktioner om det (såvida inte `pool-set-auto-stream --on`).

- **Den kontrollerar först att det är samma sensor.** Den frågar på den adress sensorn hade, utan att ändra någonting, efter sensorns namn, MAC-adress och serienummer. Den frågar också under sensorns `daq-e-<id>.local`-namn, men litar endast på det svaret när det kommer från sensorns eget nätverkskort (MAC). Inget annat som svarar återansluts eller skrivs till.
- **Den återansluter med sessionens inställningar:** integreringstid, bildmedelvärde, automatisk exponering och cap-profil som de var, inte sensorns standardinställningar vid uppstart. En sensor som hade PTP aktiverat och återkom med det avstängt slås på igen. Därefter startar den om strömmen och datorns mottagare för den.
- **Den gör nya försök med allt längre intervall:** omedelbart, sedan efter 5, 10, 20 och 40 sekunder, därefter var 60:e sekund så länge sensorn är borta. Vid varje försök frågar man bara sensorn vilken den är. En sensor som svarar men fortfarande inte skickar något efter tre återanslutningsförsök lämnas ifred och rapporteras (`auto-reconnect GAVE UP`). `pool-stream --start` försöker igen.
- **`pool-list` visar var den befinner sig.** Medan sensorn är tyst anger raden `STREAM FAULT` vad återanslutningen gör. Raden rensas med det första spektrumet, och sensorn visar då `reconnected automatically N time(s)`.
- **En inspelning fortsätter i samma `.daq`.** Filen markerar varje avbrott med två rader: `event_type` 5 vid det sista spektrumet före avbrottet och 6 vid det första efter det, båda endast med en tidsstämpel. `pool-record --duration` skriver ut `!! STREAM LOST …` medan sensorn är borta och `stream resumed after …` när den är tillbaka. `pool-record --stop` listar varje avbrott och varnar, precis som den gör för en tom fil (en inspelning som inte innehåller några avläsningar avslutas med `1`).
- **En inspelning behåller en kalibrering.** Om sensorn återkommer med en annan kalibrering medan en inspelning pågår, behåller resten av inspelningen den kalibrering den startade med, och stoppvarningen anger detta. `pool-disconnect` + `pool-connect` tar över den nya.
- **Dina kommandon har företräde.** En sensor som du avbryter med `pool-disconnect` eller pausar med `pool-stream --stop` återansluts aldrig, inte ens om en återanslutning redan pågår.
- **`pool-info` på en tyst sensor** läser av sensorn vid en ny anslutning och lägger till `!! stream NOT reaching this computer`. En sensor som inte svarar rapporteras omedelbart. `pool-list` visar inte längre ett PTP-tillstånd för en sensor som har slutat svara.
- DAQ-U och DAQ-M omfattas inte. En DAQ-M återansluter automatiskt via Bluetooth, och en DAQ-U som kopplats bort från USB behöver `pool-connect` igen.

### DAQ-E-S — fast diffusor och lutningssensor

**DAQ-E-S** är en DAQ-E med två skillnader som är viktiga för programvaran. Allt annat — upptäckt, anslutning, PTP, inspelning, enhetsinställningarna nedan — fungerar exakt som för en DAQ-E.**Dess diffusor är fast, så den kräver ingen korrigering av kåpan.** Diffusorn i Sunshine-stil är inbyggd och fanns på enheten när dess fabrikskalibrering mättes, så den kalibreringen visar redan den verkliga nedåtriktade strålningsintensiteten. Chloros ställer in sin kåpa till `as_recorded` (ingen profil) oavsett vad som begärs: en cap som är namngiven för den (`sunshine_cosine`, `fov_*`, `none`) ignoreras med ett meddelande, och dess registreringar lagras som `cap_id_source=model`, vilket innebär att varningen för antagen begränsning aldrig utlöses för den. Tillämpa inte heller någon begränsning på dess data manuellt: `sunshine_cosine` i toppen skulle visa ett värde som är ungefär 11 gånger för högt.

**Den är utrustad med en 3-axlig accelerometer.** Solskyddsdiffusorns kosinusrespons gäller endast när den är vågrät, så en lutad sensor rapporterar felaktig irradians – och reflektansen ärver felet – utan några andra symptom. Varje DAQ-E-S-avläsning bär därför sin egen lägesinformation:

| Fält | Betydelse |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Vinklar i grader. `None` när firmware markerar vinkeln som odefinierad (fritt fall, en död axel). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Acceleration i milli-g. Om `mag_mg` ligger långt från ~1000 betyder det att enheten var i rörelse och att vinklarna inte är meningsfulla för det provet. |
| `cal_applied` | Inställt: axlarna och vinklarna är i diffusorns referensram (enhetens monteringskalibrering tillämpas). Rensat: råa kretskortsramvärden, vilket kan visa upp till ~2,3° lutning som inte är verklig. |
| `sample_age_ms` | Åldern på accelerometerns mätvärde när spektrumet skickades. |
| `temp_raw` | Okalibrerad **trend** för chipets temperatur — endast skillnader, aldrig ett termometeravläsningsvärde. |

`daq pool-latest --json` returnerar dem under varje rams `imu`-nyckel (`null` på en DAQ-U, DAQ-M eller vanlig DAQ-E). En `.daq` som registrerats från en DAQ-E-S lagrar samma värden i sina `imu_*`-kolumner (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) bredvid varje spektrum, så att läsaren kan skilja ett moln från ett stativ som har välts. På alla andra modeller är dessa kolumner tomma.

Lägesströmmen rapporteras separat: `daq pool-list` skriver ut `!! ATTITUDE STREAM FAULT (spectra unaffected)` medan den är avstängd, samt hur många fel den återhämtade sig från därefter. Ett fel där medför aldrig ett avbrott i spektralregistreringen. `daq pool-discover` listar enheten som `DAQ-E-S` (som `DAQ-E` om den inte svarar på identitetskontrollen inom cirka 2 sekunder).

### DAQ-E-enhetsinställningar

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` utan `--gateway` sparar ingen gateway** (alla tidigare inställda gateways tas bort), vilket är rätt för ett kameranätverk utan router.
- **En statisk adress måste finnas i den här datorns nätverk.** Chloros-programmet avvisar en adress vars subnät inte inkluderar den här datorns adress i sensorns nätverk: efter omstarten kan ingenting nå sensorn för att återställa inställningen. Använd endast `--force` när den här datorn kommer att omadresseras eller dirigeras till det nya subnätet.
- **En omstart nekas medan sensorn spelar in.** Avsluta först inspelningen (`pool-record --stop`). En inspelning som påbörjades strax före en omstart avslutas av denna, och `pool-set-network`, `pool-reboot` och `pool-update-firmware` avslutas med en `Warning:`-rad som anger inspelningsfilens namn (även om sensorn då inte återkommer).
- **Efter en omstart återansluter Chloros endast själva sensorn.** Programmet väntar på den plats där sensorn befann sig – och efter `--auto` även under dess `daq-e-<id>.local`-namn – på ett svar med sensorns namn, värdnamn och modell, från det nätverkskort (MAC) den hade, och återansluter den till den adress som svarade. Ett svar under namnet accepteras endast med just den MAC-adressen, så från en dator som når sensorn via en router, eller från WSL2 i dess standard-NAT-nätverk, återansluts inte en sensor som DHCP har flyttat till en ny adress: kommandot anger detta och visar kommandot `pool-connect --eth-host <its address> …` som återansluter den med de inställningar för bländare och exponering som den hade.
- **`pool-update-firmware` avvisar en sensor vars firmwareversion den inte kan läsa.** En sensor som tar längre tid än väntetiden på att återansluta efter blixten har inte gått sönder: så snart den svarar återansluts den med den nya firmwareversionen inställd, och kommandot varnar för att den var långsam.
- **En firmwareuppdatering kräver en inkommande TCP-anslutning.** Sensorn ansluter tillbaka till den här datorn för att hämta bilden. Chloros lyssnar på det nätverkskort som vidarebefordrar till sensorn (på alla nätverkskort endast när det inte går att avgöra vilket) och betjänar endast sensorns egen adress; alla andra anslutningar stängs utan att betjänas. I Windows släpper installationsprogrammets brandväggsregel `Chloros Backend (TCP-In)` in den (se [Windows (.exe)](#windows-exe)); en brandvägg som blockerar den gör att uppdateringen misslyckas med felkoden `… device never connected back on TCP <port> …`.
- **Under WSL2 ska du använda speglat nätverk** (`networkingMode=mirrored` i `%UserProfile%\.wslconfig`). I standardläget för NAT är den här datorns adress i sensorns nätverk WSL:s egen, så alla statiska adresser avvisas (förutom `--force`), livestreamen når inte WSL och en firmwareuppdatering kan inte slutföras, eftersom sensorn ansluter tillbaka till datorn för att ta emot bilden.
- **Sensorns ID och värdnamn ändras inte.** Oavsett vilken adress den har är en DAQ-E fortfarande `daq-e-<id>` och `daq-e-<id>.local`, så `--eth-host daq-e-<id>.local` fortsätter att hitta den.
- **GUI:n har samma inställningar**: fliken Ljussensorer, sensorns inställningar (kugghjulet), Nätverk. GUI:n behåller sin egen anslutning till en sensor, så om du ändrar en sensor som GUI:n har öppen från GUI:n: en omstart från CLI:n avslutar GUI:ns session med den.
- Motsvarigheterna i SDK:n är `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` och `update_firmware()`.

---

## `chloros-cli project`

Öppna, anslut till och kör ett sparat Chloros-projekt (en mapp med `cameras.json` + `sensors.json` + `project.json`). Allt går via backend, så att GUI och CLI ger identiskt hårdvarutillstånd.

### Referens för underkommandon

| Underkommando | Syfte |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Skriver ut projektets enhetsmanifest (kameror, arrayer, sensorer), läser från dess filer och kontrollerar dess källfiler (se [Ett flyttat projekt](#a-moved-project---relink)). Det öppnar inte projektet i backend: projektet som är öppet där, liksom dess enheter och inspelningar, förblir oförändrade. Avslutar `1` om ingen av projektets källfiler kan hittas. |
| `project devices PATH [--reconnect]` | Lista eller kör om upptäckten. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Anslut varje sparad kamera / array / sensor. Varje array ansluts på samma sätt som `lattice array-connect` gör: upp till 600 s, varvid stegen skrivs ut till stderr (`[array <id>]    <s> s: <step>`; stdout är JSON-rapporten), och en array som inte anslutits inom den tiden ångras — dess kameror återställs till det tillstånd de befann sig i — och rapporteras under `errors`, de lämnas aldrig anslutna. `--packed` ställer in varje arrays packade 12-bitars transport (standard `auto`; ett projekt sparar inga inställningar per array; se [Packad 12-bitars transport](#packed-12-bit-transport---packed)). Varje ljussensor öppnas i backendens DAQ-pool på sin sparade adress med sin sparade integrationstid, bildrutsmedelvärde, automatisk exponering och gränsvärde: en sensor utan vald cap får `sunshine_cosine`, en som sparats utan inställningar (`none`) förblir utan inställningar, och en DAQ-E-S tar aldrig en cap. GUI:ns DAQ-panel ingår inte i poolen: stäng först en sensor där. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Enstaka bild från en namngiven kamera eller matris (`--prefix` som standard, `capture`). `--orient` (matriser): matrisen sparar rådata, som förblir i ursprungligt format — detta registrerar att de produkter som `process` skapar utifrån dessa ska orienteras, som `array-capture --orient`. Standard: ursprungligt format. `project burst` / `stream` och ett recepts `capture` / `burst` / `stream` (`orient: true`) har samma inställning. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | N-bilders serie från en namngiven kamera eller matris (`-n/--count` standardvärde 5; `-i/--interval` sekunder mellan bilder, standardvärde 0). Serieburstar avduplicerar upprepade synkroniserade grupper (föråldringsvakt) så att en delvis cyklisk serien inte kan lämna tillbaka N kopior av en bildruta; skriver ut resultat per iteration. En inspelning som misslyckas på vägen avbryter serien och avslutar `1`, med information om hur många som sparades innan det hände. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Inspelning till disk i en loop: `--fps F` spelar in en sekund (standard 3), en bildruta per inspelning från en kamera, en synkroniserad bildrutegrupp från en matris (med samma föråldringsvakt som `project burst`), tills `-n N` spelar in (standard `0` = tills Ctrl+C trycks ned, vilket visar hur många som sparades och avslutar `0`, eller `1` om ingen sparades eller en fil misslyckades; om Ctrl+C trycks in innan `-n N` fångar upp avslutas `130`). En inspelning som misslyckas på halvvägs avbryter strömmen och avslutar `1`, samtidigt som den anger hur många bildrutor som sparades innan det hände. `--poll-interval` är föråldrat och ignoreras: den accepteras fortfarande så att äldre skript kan köras, med en anmärkning på stderr. |
| `project sensor read PATH NAME [--json]` | Senaste spektrumramen. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Spela in en .daq-fil för `--seconds` (standard 10). En inspelning som inte innehåller några avläsningar rapporteras på stderr och avslutar `1`. |
| `project run PATH RECIPE.yaml` | Kör ett YAML-/JSON-inspelningsrecept. `--dry-run` validerar utan att köras. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Beräkna inriktning för en matris — se [flaggtabellen nedan](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Skriv ut aktuell inriktningsprofil. |
| `project align clear PATH NAME` | Ta bort den cachade profilen. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Justera en slavs transformation (standardinställningar `0`, `0`, `0`, `1.0`: ingen ändring). |
| `project align export PATH NAME --to FILE` | Spara profilen som JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Ladda en sparad profil. |

#### `project align calibrate` Alternativ

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Justeringsmetod. **Dessa stavningar skiljer sig från `lattice align-calibrate`**, som accepterar kortformerna `orb` / `akaze` / `phase`; de två kommandona är inte utbytbara för denna flagga. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformera modellen så att den passar. |
| `--frames N` | `1` | Synkroniserade bildrutor till medelvärdet. |
| `--reference SN` | huvudkameran | Referenskamerans serienummer; alla övriga medlemmar förvrängs till denna. |
| `--max-features N` | `5000` | Övre gräns för antal ORB-egenskaper. |
| `--ratio-threshold F` | `0.75` | Lowes förhållandestest. |
| `--ransac-threshold-px F` | `3.0` | Tröskelvärde för RANSAC-inliers. |
| `--min-matches N` | `15` | **Kvalitetsgräns** — avvisa lösningen om antalet inlier-matchningar understiger detta värde. |
| `--max-reproj-err-px F` | `4.0` | **Kvalitetsgräns** — avvisa lösningen om RMS-felet vid omprojektion överstiger detta värde. |
| `--checkerboard RxC` | — | Brädgeometri för `--method checkerboard`, t.ex. `9x6`. |
| `--name PROFILE` | tom | Profilnamn inbäddat i den sparade JSON-filen. **Inte arraynamnet** — det är det positionella `NAME`. |

De två kvalitetskontrollerna är anledningen till att en kalibrering kan lyckas med lösningen men ändå
avvisa sparandet: en profil som misslyckas med någon av dem skulle i tysthet felregistrera varje
senare inspelning, så den avvisas istället för att sparas.

### Ett flyttat projekt (`--relink`)

Ett projekts `project.json` namnger varje källfil — varje bild och ljussensorinspelning
— med dess fullständiga sökväg. Flytta projektmappen, eller hela projektmappen,
till en annan enhet, så pekar dessa sökvägar fortfarande på den gamla platsen.

När ett projekt öppnas åtgärdas det som går att åtgärda automatiskt: en fil som tidigare fanns i projektets
mapp söks i den mapp där projektet nu finns, och varje sökväg som hittas där
omdirigeras och sparas (`project.json` säkerhetskopieras först som
`project.json.before-relink-<date>-<time>`). Detta sker vid varje öppning – av appen,
`chloros-cli project open`, `chloros-cli process` i ett befintligt projekt, SDK:s
och `chloros-mcp`:s. En sökväg omdirigeras endast till en fil som finns där.

Filer som befann sig utanför projektmappen – en hubbsession, en import via ”Lägg till mapp” –
kan inte hittas på det sättet. Ange namnet på den mapp de nu finns i:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` tar den mapp som filerna hamnade i, eller vilken mapp som helst ovanför den: varje saknad
fils mapp söks under den utifrån dess avslutande mappnamn
(`Hub Projects/<session>/raw/<serial>`, sedan `<session>/raw/<serial>`, ...), och varje
mappens filer hämtas från den plats där flest av dem hittas. `project open` visar
vad som har omdirigerats, hur många källfiler som hittades och i vilka mappar resten fanns. Vissa saknas fortfarande: en varning, avsluta `0` – projektet öppnas med resten. Inga
hittades: ett felmeddelande som anger dessa mappar och `--relink`, avsluta `1`; backend vägrar likaså
vägrar att öppna ett sådant projekt för `process` eller `project devices --reconnect` (HTTP 409 `PROJECT_SOURCES_MISSING`).

### Exempel

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Recept-DSL

`project run RECIPE.yaml` accepterar en YAML- eller JSON-fil som beskriver en sekvens av åtgärder:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Åtgärder som stöds: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` hämtar nycklarna `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (de äldre namnen `exposure_time`, `exposure_auto`, `gain`, `gain_auto` accepteras som alias). En exponering, förstärkning eller ett mål som ställs in på detta sätt behålls som ditt: Chloros automatiska exponeringskontroll (inklusive den DAQ-mätta exponeringen) lämnar det ifred. En manuell exponering lämnar förstärkningen som den var: i auto-läge fortsätter kameran att justera förstärkningen (över hela sitt intervall) mot målens ljusstyrka tills du ställer in `gain_db` eller `auto_gain: "Off"`. Ett senare `auto_exposure: "Continuous"` återför exponeringen till automatiken, och `target_brightness_auto: true` återför ett låst mål till Chloros automatiska mål. Åtgärden `burst` kräver `name` (obligatoriskt), `count` (standardvärde 5), `interval` (sekunder, standardvärde 0), `output`, `format`, och `settings` (samma inställningsform per kamera som `apply`); array-serier använder samma nysynkroniserade grupp-watchdog som `project burst`.

En `capture`, `burst` eller `stream`-steget har som standardinställning `output`, som är `output`. En relativ `output` — som ingår som standard — löses mot den mapp du kör `chloros-cli project run` från, inte receptfilens mapp eller projektmappen (i 1.2.0 hamnade den i backendens installationsmapp).

Kör det:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Varje körning kontrolleras först mot projektet.**Varje åtgärd måste ha exakt ett av ovanstående verb som nyckel — `- capture: {name: FrontLeft}`, inte `- action: capture` eller `- type: capture` — med ett `name` har projektets `cameras.json` / `sensors.json` en kamera eller matris för `apply` / `capture` / `burst` / `stream` samt en ljussensor för `sensor`; en `stream` kräver att `count` > 0 och en `wait` kräver en varaktighet. Ett recept med något sådant fel utför**ingen** åtgärd (`[recipe] error: …`, `[recipe] not run: …`, avsluta `1`), inte ens med `stop_on_error: false`. Nycklar som ett steg inte använder är en `[recipe] warning:`, liksom varje array eller ljussensor som receptet styr men som inte är ansluten just nu. Ett recept ansluter aldrig enheter: kör `chloros-cli project connect PATH` först. `--dry-run` skriver ut receptet, därefter samma fel och varningar samt `[recipe] valid: …` / `NOT valid: …`, kör ingenting och avslutar `1` när receptet har ett fel.

---

## Miljövariabler

| Variabel | Effekt |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Åsidosätter backend-URL:en (standard `http://127.0.0.1:5000`) — **respekteras endast av kommandofamiljerna `lattice`, `project` och `daq pool-*`** (`lattice network-analysis` använder istället `--backend-url`). Kärnkommandona (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) kopplar `http://127.0.0.1:<port>` och ignorerar denna variabel (IPv4-värdet kringgår Windows-straffet på cirka 2 sekunder per begäran för `localhost`→`::1`), så att de alltid riktas mot den lokala datorn. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Ställs in i backendens miljö. `1` nedgraderar avvisningen vid överteckning — en array-anslutning med `pin_resolution`, eller ett [tillfälligt flerkamerakommando](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — till en tydlig varning, vilket accepterar GVSP-paketförlust. Endast för testning — se [Array fps & burst-modell](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Ställs in i backendens miljö: `on` eller `off` ställer in standardvärdet [packad 12-bitars transport](#packed-12-bit-transport---packed) för varje array (standard `auto`). En explicit inställning med `--packed` har fortfarande företräde. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (eller `true` / `yes`) inaktiverar Linux-CLI:s [automatiska uppdateringskontroll](#chloros-cli-update) vid uppstart. |
| `CHLOROS_CLI_MODE` | Ställs in av CLI:n själv; talar om för backend att aktivera parallellbearbetning. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` hoppar över GVSP-fallback-sonden (endast ICMP-resultat). **Detta stänger av jumbo, det dämpar inte bara loggen** — kameran svarar endast på DF-pingar upp till 1500 på varje väg, så denna sond är det enda som kan upptäcka jumbo. Sparar ~1 s per kamera per anslutning; kostar ~1,45× bandbreddsgränsen om nätverket *skulle* ha kunnat hantera jumbo. SDK:t varnar när du ställer in det. |
| `CHLOROS_GVSP_SESSION_REUSE` | Ställs in i backendens miljö: `0` mäter paketstorleken vid varje anslutning istället för en gång per kamera per backend-session. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Lås GVSP-paketstorleken till N byte; hoppar över sonderingen helt. Använd helst per kommando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) istället för att ställa in det permanent: en fastställd storlek anpassar sig inte längre till nätverket framför den, och att fastställa 9000 på en väg som inte klarar jumbo-paket gör att **varje** insamling går över tiden med `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Ställs in i backendens miljö: det maximala antalet sekunder för ett [tillfälligt flerkamerakommando](#multi-camera-transient-sessions), eller en `lattice capture` för en kamera som inget håller kvar, får på sig för att ställa in varje kameras exponering från scenen innan den sparar en bildruta (standard `5`). `0` stänger av funktionen: varje kameras egen automatiska exponering körs då från den första bilden, med utgångspunkt från dess exponeringsgräns. |
| `CHLOROS_LOG_DIR` | Ställs in i backendens miljö: mappen dit dess detaljerade logg sparas. Standardinställning `%LOCALAPPDATA%\Chloros\logs` på Windows, `~/.cache/chloros/logs` på Linux (`$XDG_CACHE_HOME/chloros/logs` när det är inställt). .deb-filens rottjänst ställer in `/var/log/chloros`, som är läsbar för gruppen `adm` — se [Köra backend som en tjänst](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Var tillfälliga filer sparas. CLI använder automatiskt `/mnt/ssd/tmp` om den finns. |

---

## Avslutningskoder

| Kod | Betydelse |
| --- | --- |
| `0` | Framgång. |
| `1` | Generellt fel (de flesta underkommandofel). Ett inspelningskommando avslutas också med `1` om någon fil som det ombads att skriva inte kunde skrivas, även om de andra sparades, och med `array-burst` / `array-record` avslutar `1` när inspelaren rapporterar ett fel — se [Filer som inte går att skriva](#files-that-fail-to-write). `process` avslutas också `1` när en körnings exporter inte kunde verifieras, eller när en bild gick förlorad, även om de övriga produkterna har skrivits. |
| `2` | Argumentfel (en okänd eller saknad flagga; även `time-sync set-priority` utan någon av dessa). |
| `130` | Avbruten av Ctrl+C: alla kommandon som avbryts — `lattice`, `project`, `daq` och `process` (`process` efter att backend har avbrutit körningen, se [`process` Anmärkningar](#notes)). Ett kommando vars normala avbrytning är Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` och `project stream` med `-n 0` – avslutas istället beroende på vad som hände: `0` när allt hade skrivits, `1` när något misslyckades. |
| `143` / `129` | Avslutades av SIGTERM / SIGHUP (128 + signalen), efter att kommandots uppstädning körts: ett pågående jobb avbröts och dess kameror återställdes, en bearbetningskörning avbröts i backend. En `lattice continuous` eller annan inspelning som normalt avslutas med Ctrl+C avslutar även `143`, inte `0`. |

`lattice network-analysis` avslutar `1` för bedömningarna `needs_force_slip` och `error`, och `selftest` avslutar `1` om inte alla sju kontroller godkänns.

---

## Tips för felsökning

- **”Inloggning krävs”** → Kör `chloros-cli login EMAIL PASSWORD` en gång på den här datorn.
- **"Projektmappen är för djup för den här datorn"** → Långa sökvägar i Windows är inaktiverade på den här datorn (Windows-standard), så ingen sökväg får överstiga 260 tecken, och produkterna ligger upp till cirka 130 tecken under projektmappen. Ingenting har bearbetats. Meddelandet anger den längsta projektmappen som passar: använd en kortare (`-o` eller `chloros-cli set-project-folder`), eller aktivera långa sökvägar i Windows och starta om Chloros. Windows-installationsprogrammet erbjuder detta som ”Aktivera långa sökvägar i Windows”, markerat som standard; annars använder du grupprincip: *Datorkonfiguration > Administrativa mallar > System > Filsystem > Aktivera långa Win32-sökvägar*.
- **”backend unreachable”** → Starta Chloros-skrivbordsappen (Windows), eller starta backend själv: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` på Windows (mappen `backend` i din installationsmapp; den finns inte i `PATH`), `chloros-backend` på Linux. Eller kontrollera `CHLOROS_BACKEND_URL` om det är fjärranslutet.
- **`lattice`-kommandon misslyckas med felmeddelandet ”LATTICE-kameradrivrutiner hittades inte”** → Backend-programmet kunde inte ladda den LATTICE-kamerakörmiljö som det är installerat med. Varje `lattice`-kommando körs på backend; CLI:n i sig innehåller inga kameradrivrutiner. Installera om Chloros: Windows-installationsprogrammet och .deb-paketet installerar runtime-miljön bredvid backend.
- **Array connect / Array Settings visar ”FRAMES WILL DROP” eller ”Reduce ROI to enable”** → Värd-NIC:s mottagningsring är för liten (återställs vanligtvis till 32 efter en uppdatering av NIC-drivrutinen). Se [Konfiguration och inställning av värd-NIC](#host-nic-setup--tuning-lattice-arrays) — ställ in `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Datorn hänger sig vid omstart/avstängning, därefter WMI `Invalid class` / nätverkskortet aktiveras inte / USB-enheter saknas** → Föråldrad drivrutin för USB 10GbE-adapter orsakar `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Uppdatera adapterdrivrutinen — se [Konfiguration och finjustering av värd-NIC](#host-nic-setup--tuning-lattice-arrays).
- **Linux: inspelningarna tillhör root, så du kan inte ta bort dem, och backendens logg finns inte i din `~/.cache`** → Backenden körs som root via `chloros-backend.service`. Byt till `chloros-backend@<user>.service`, där den körs som du – se [Kör backend som en tjänst](#running-the-backend-as-a-service-headless). Fram till dess finns root-tjänstens logg i `/var/log/chloros` (läsbar av gruppen `adm`).
- **Linux: en backend-enhet är `failed` med meddelandet ”En annan instans körs redan” (avslutningsstatus 98)** → En annan backend har redan porten upptagen: endast en kan köras åt gången. Om två backend-enheter är aktiverade, behåll en: `sudo systemctl disable --now <unit>` för den du inte använder — se [Köra backend som en tjänst](#running-the-backend-as-a-service-headless). Om endast denna enhet är aktiverad, är det en backend som startats utanför systemd (med `chloros-backend`, `chloros-cli` eller ett SDK-skript) som upptar porten: stoppa den backenden och kör sedan `sudo systemctl start <unit>`.
- **Jetson-swapvarning** → Lägg till filbaserad swap; CLI:n visar de exakta kommandona `fallocate` / `swapon`.
- **"backend not reachable" från ett `lattice`-, `project`- eller `daq`-kommando** → ingen backend körs: dessa kommandon startar aldrig någon. Kör `chloros-cli backend status` och se [Starta backend](#start-the-backend).
- **"`daq net ...` är inte tillgängligt i Chloros"** (Linux och Jetson) → Vissa av DAQ-E:s egna inställningar är reserverade för MAPIR-tjänsten. Nätverksinställningar, omstarter och firmwareuppdateringar hanteras av `pool-set-network`, `pool-reboot` och `pool-update-firmware`; kontakta MAPIR-supporten för övriga frågor.
- **"Direkta hårdvarukommandon för DAQ kräver det fullständiga daq-paketet"** (Windows) → Använd `pool-*`, som styr samma sensor via backend, eller `chloros_sdk.connect_daq_sensor()` från Python.

---

## Se även

- [Python SDK-referens](sdk-reference.md) — programmeringsmässig motsvarighet till varje CLI-kommando.
- [DAQ-ljussensorer](https://mapir.gitbook.io/chloros/daq-light-sensors) i Chloros-handboken — sensorerna, deras gränsvärden och kalibrerade mätområde, inspelning, DAQ-E-nätverk.
- [LATTICE-kameror](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) i Chloros-handboken — anslutning, kamerainställningar, bildtagningslägen, matriser.
- Online-dokumentation: `https://mapir.gitbook.io/chloros/cli`
