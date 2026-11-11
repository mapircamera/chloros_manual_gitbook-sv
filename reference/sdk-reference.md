# Referens för Chloros Python SDK

**Version:**

1.3.0**Skapad:**2026-07-29 19:19 ·**Uppdaterad:** 2026-11-06**Paket:** `chloros-sdk` (PyPI)**Målgrupp:** Optimerad för LLM-användning; läsbar för människor.**Omfattning:** Alla offentliga klasser, funktioner och hjälpfunktioner som exponeras av `import chloros_sdk`, med exempel som kan kopieras och klistras in och som täcker bildbehandling, styrning av enstaka kameror, synkroniserade arrayer, DAQ-sensorer och projektautomatisering, samt `chloros-mcp`-servern för AI-assistenter.

Om du bara vill ha det viktigaste, gå till:
- [Installation och snabbstart](#installation)
- [Smart-Connect för LATTICE-matriser](#smart-connect-for-lattice-cameras)
- [DAQ-sensorsessioner](#daq-sensor-sessions)
- [Kameraverktyg](#camera-utilities)
- [Transient-matriser, inriktningsprofiler och index](#transient-arrays-alignment-profiles--indices)
- [Projektmapp och recept](#project-folder--recipes)
- [Projektautomatisering](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Livebilder i din egen pipeline](#live-frames-in-your-own-pipeline)
- [MCP-server (AI-assistenter)](#mcp-server-ai-assistants)

> **Ändringar i 1.3.0** (vad ett befintligt 1.2.0-skript märker).
>
> **Ändringar som ett 1.2.0-skript kan behöva hantera:**
>
> - **En release som backend inte bekräftar ger nu upphov till ett fel.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` på en array som sessionen öppnat, `DAQSensorSession.disconnect()` / `force_disconnect()` på en sensor som den öppnat, och `ChlorosProject.disconnect_all()` genererar `ChlorosConnectError` när backend vägrar frigivningen eller inte bekräftar den i tid, eftersom enheten fortfarande kan vara upptagen. I 1.2.0 återlämnades de utan meddelande. Ett `with`-blocks avslutning utlöser fortfarande aldrig något.
> - **En lånad enhet avslutas aldrig.** `with attach_array(...)` och ett explicit `disconnect()` på den låter arrayen fortsätta köras (i 1.2.0 avslutades den när blocket lämnades, även när GUI ägde den); anropa `force_disconnect()` när du vill avsluta den. På samma sätt lämnar en `connect_daq_sensor()` som upptäckte att sensorn redan fanns i poolen, inklusive smart-detect med nollflaggan, den öppen på `disconnect()` och när dess `with`-block lämnas.
> - **PNG och JPEG tar endast `display` och indexprodukter.** `capture()` med `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) och en nivå `radiance` eller `reflectance`, eller `processing="all"`, genererar `ValueError` före varje begäran; en 12-bitars `raw` / `debayered` avvisas av backend. Se [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` öppnar den sensorn eller genererar** `ChlorosConnectError` med en lista över vad den hittade; den faller aldrig tillbaka till en annan sensor. Se [Fäst transport](#pinned-transport).
> - **`frame_stream()` genererar ett felmeddelande istället för att tystna:** `FrameFetchError` vid ett avslag skulle nästa tick upprepa (en reflektansavläsning utan någon DAQ alls inkluderad), efter 60 sekunder utan ram och efter 60 sekunder utan ny trigger. Se [En trigger per avläsning](#one-trigger-per-read--group_id).
> - **En avläsningstimeout eller en avbruten anslutning utlöser `ChlorosConnectError`** från varje smart-connect-anrop och `analyze_array_network()`, där 1.2.0 utlöste den underliggande timeouten eller anslutningsfelet.
> - **`connect_array()` väntar som standard i 600 s (`timeout`, 180 s i 1.2.0), och en anslutning som det ger upp på ångras.** Anropet skickar sitt `timeout` till backend, vilket återställer varje kamera till det tillstånd den befann sig i och inte ansluter någonting om anslutningen inte är klar i tid; anropet genererar `ChlorosConnectError` (HTTP-kod 504 `CLIENT_DEADLINE_PASSED`). I version 1.2.0 kunde en anslutning direkt efter att kamerorna startats pågå längre än anropet och ansluta matrisen efter att felet redan hade genererats. `connect_all()` väntar på varje sparad matris på samma sätt (120 s i 1.2.0). Se [`connect_array()`-signatur](#connect_array-signature).
> - **`connect_array()` behåller hela sensorn.** Utan `width` / `height` / `binning` är varje kamera inställd på full sensor och bildfrekvensen sjunker till vad värden och nätverket levererar; 1.2.0 har krympt eller sammanfogat bildrutorna för att hålla uppe bildfrekvensen. Se [Synkroniserad matris](#synchronized-array--arraysession-smart-prep).
> - **Ett Bayer-pixelformat på en M3M-kamera (mono) är dess djup.** En M3M erbjuder endast monoformat, och förinställningarna benämner dem som Bayer-format: i 1.2.0 vägrade kameran att använda formatet och behöll sitt eget, medan `settings.pixel_format` namngav det som ett Bayer-format. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, en `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` och en matriss gemensamma formatet ställer nu in Mono-formatet med samma djup på en M3M: `"BayerRG8"` och `high_speed` ger `Mono8`; `"BayerRG12"`, `high_quality` och `triggered` ger `Mono12`; `cam.settings.pixel_format` namnger det. Detsamma gäller tvärtom: en M3C placeras aldrig i ett Mono-format, så `"Mono12"` – från vilken som helst av dessa, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` eller en matris sparade format – ger `BayerRG12` på en M3C, oavsett vilken kamera som är matrisens huvudkamera. Ett `CameraSettings` som du skickar vidare behåller sitt namn, så samma objekt frågar fortfarande varje kamera om det djupet inom sin egen familj. `connect_camera(preset=...)` behåller förinställningens format (i version 1.2.0 ställs kameran in på 12-bitars efter att förinställningen tillämpats), och en kamera som öppnas med en förinställning skriver inte längre in det den tagit till `PRESETS`. Alla andra format som en kamera vägrar (ett som saknas i dess `PixelFormat`-lista, såsom `RGB8` på en M3M eller `BayerRG16` på firmware utan det) får inte heller ett eget namn: efter en anslutning till `LatticeCamera`-anslutning namnger `apply_settings()` eller `set_format()`, `cam.settings.pixel_format` och `packed_transport` anger vilket format kameran strömmar, och en utskriven rad anger vilket format den avvisade.
> - **`analyze_array_network()` visar vilken anslutning som gäller:** `pixel_format=None` (tidigare `"BayerRG8"`) anger de format som anslutningen stöder (Mono12 / BayerRG12), och `packed="auto"` anger valet av komprimerad överföring, vilket innebär att den angivna bildfrekvensen kan avvika från 1.2.0&#x27;. Ett `pixel_format` som du skickar vidare projicerar en M3M på det djupet. Se [Network-Analysis Helper](#network-analysis-helper).
> - **`CameraSession.capture()` returnerar en `CaptureResult`** (fortfarande en lista med ramdikt) med `.skipped`, `.failed`, `.daq_files` och `.daq_matches`. Ett `ArraySession.capture()`-resultat innehåller även `.alignment` (om justeringsförskjutningen tillämpades, och varför inte) och `.oriented`.
> - **Array-fångster justeras som standard, precis som appens.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` och `ArrayHandle.capture()` / `capture_smart()` förvränger varje medlems `debayered`, `display`, `radiance`, `reflectance` och indexfilerna in i huvudkamerans bildruta och beskär dem till medlemmarnas överlappning (mindre än sensorn med kamerornas förskjutningar, `XMP-mapir:AlignmentJSON` `"applied": true`), så att de är samma filer som Chloros-appens bildtagning av samma scen; `raw` förblir i originalformat. Tidigare sparades vid en inspelning utan `aligned=True` varje fil i sin ursprungliga storlek, utan förvrängning. `aligned=False` behåller detta. `chloros-mcp`s `capture_array` och `start_capture_run` använder samma standardinställning. Se [`ArraySession`-metoder](#arraysession-methods).
> - **`connect_all()` anger ett backend som avvisar SDK:t.** Ett bakgrundsprogram som är igång men som är utloggat eller har en plan under Copper rapporteras under `errors` med sin egen orsak (HTTP 401 / 403), inte som ”inget Chloros-backend körs”, och varje `connect_all()`-anrop kontrollerar backenden igen, så att anropa den efter inloggning fungerar på samma projektobjekt.
> - **Ett relativt `output_dir` fungerar mot en lokal backend.** Relativa sökvägar omvandlas till absoluta i förhållande till skriptets arbetsmapp innan de skickas. I version 1.2.0 tolkade backenden dem i sin egen installationsmapp, vilket resulterade i felmeddelandet ”Åtkomst nekad”.
> - **`numpy` är nu ett krav** i SDK:et (live-frame-anropen returnerar numpy-matriser).
> - **Ett exportnivånamn som inte är ”one” avvisas.** `processing=` och `levels=` tar emot `raw`, `debayered`, `radiance`, `reflectance` eller `display` (eller ett alias såsom `preview`; `processing=` samt `"all"`). Alla andra namn avvisas innan kameran vidrörs (HTTP 400 `UNKNOWN_LEVEL`: `capture()` genererar `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); 1.2.0 sparade eller returnerade `raw` för detta.
> - **Texture Aware avvisas för LATTICE-inspelningar.** I ett projekt som innehåller LATTICE-inspelningar genererar `configure(debayer="Texture Aware (Slow, Highest Quality)")` ett `ChlorosConfigurationError`, och detsamma gäller `process()` efter ett `configure()` som begärde om detta före importen. Ett Texture Aware-värde som projektet sparade tidigare ställs in på Standard, med ett `UserWarning`. LATTICE-bilder får alltid standarddemosaik, så 1.2.0 accepterade inställningen och ignorerade den; Chloros-appen erbjuder inte heller detta för dem. Se [Stödda värden](#supported-values).
> - **Kameror återställs till det läge de hittades i — såvida inte `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, slutet på varje övergångsfunktion (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) samt avslutningen av `start_camera_continuous()` återställer varje kamera till det tillstånd den befann sig i: dess utlösare och synkroniseringsledningar, dess PTP samt alla kamerainställningar som ändrades under sessionen (exponering och förstärkning, AE-mål, gränsvärden och dämpning, vitbalans, svartnivå, pixelformat, binning och ROI, speglingar, bildfrekvens). `keep_settings=True` sparar istället varje kameras inställningar i kamerans eget minne, så att den startar med dem; `clear_settings=True` återställer det minnet till fabriksinställningarna. I båda fallen lämnas kamerorna i friläge. Ett projekt ställer in sina kameror utifrån sina egna sparade inställningar när det ansluter dem. Se [Kameraminne](#camera-memory--keep_settings--clear_settings).
> - **Alla lägen sparar ett projekt på samma sätt.** Varje ändring av en kamera, en matris eller en ljussensor som är ansluten till ett projekt — oavsett om den görs via SDK, appen, CLI eller MCP-servern — sparas i projektets `cameras.json` / `sensors.json`, och när projektet öppnas i valfritt läge konfigureras enheterna utifrån det; en `ChlorosProject` namnger sin egen mapp till backend. Tidigare sparade endast appen projektet. Se [En projektpost, alla lägen](#one-project-record-every-mode).
> - **Filer som skapas från flera kameror namnger var och en av dem.** En multibandstack, ett liveindex skapat av flera kamerors band och en bild från en array-inspelning (`composite/`) innehåller varje källkameras serienummer, modell och kalibreringspaket som listor (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), och EXIF-listorna `SerialNumber` innehåller serienumren, separerade med kommatecken. Tidigare innehöll de ingen kamerainformation alls, eller endast den första kamerans serienummer. Se [Filer skapade från flera kameror](#files-made-from-several-cameras).
> - **Endast en `raw`-fil innehåller kamerans kalibreringstaggar.** Den MicaSense-kompatibla uppsättningen (`BlackLevel`, `RadiometricCalibration`, dekanal-`Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`-uppsättningarna) samt nyttolasten `CalibrationJSON` beskriver sensorns råvärden, så de ingår endast i `raw`-inspelningsfilen. Alla övriga filer — en inspelningsfil`debayered`, `display`, `radiance`, `reflectance` samt indexfiler, kontinuerligt inspelade och flerkamer– samt varje `process()`-produkt — bär kamerans identitet (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) och, vid en bildtagning, bildrutan egen inspelning (`XMP-mapir:FrameJSON`: exponering, förstärkning, temperatur, pixelformat, tidpunkt för bildtagning). I version 1.2.0 innehöll de även rådatauppsättningen, som ett verktyg (Pix4D, Metashape) använde på pixlar som inte var råvärden. Bearbetningen hittar kalibreringen för en sådan fil utifrån dess serienummer och `CalBundleSha256`.
>
> **Övriga ändringar:**
>
> - **En matris automatiska bildfrekvens anpassas efter vad matrisen levererar.** En frekvens som förlorar mer än 1 % av sina utlösare sänks, vilket bedöms utifrån flera långa mätningar snarare än en enda, och en frekvens som den sänkts från prövas på nytt då och då. En frekvens som du själv ställer in lämnas oförändrad.
> - **Autoexponeringen är stabilare.** Den håller sig konstant vid oförändrat ljus, stabiliseras inom några sekunder efter en förändring, ignorerar små högdagrar och återställer en kamera som slutar svara i starkt solsken. Se [Hur autoexponeringen fungerar](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` räknar endast dina bilder och inspelningar.** En mapp som innehåller ett projekts `camera_cal/` räknar inte längre dessa kalibreringsfiler som bilder och inspelningar, och ett projekt som redan har tagit in dem tar bort dem.
> - **`configure(radiometric_output=...)` gäller.** Den ställer nu in projektets `Export radiance` / `Export reflectance`-produkter; tidigare hade varje projekts egna produktinställningar företräde, så att `radiance`, `sensor-response` och `all` alla exporterade strålnings- och reflektansvärden.
> - **`test_array_sync()` startar från kamerornas påslagna tillstånd.** Den aktiverar kamerornas PTP för körningen, väntar på att deras klockor ska låsas och stabiliseras innan den första utlösningen (`ptp_lock`) — varje kamera låses i 120 s, cirka 2 minuter från uppstart (`ptp_settle_s=` ställer in det), eftersom en klocka som just låst sig har en avvikelse på upp till ~200 us i ungefär 2 minuter — och återställer varje kameras PTP till det läge den fann den i. En kamera som fortfarande är synkroniserad från ett tidigare test väntas inte på igen. Tidigare lämnade den PTP avstängt i ett nätverk med en enda switch där ingen DAQ-E svarade, och läste varje kameras fritt löpande klocka som en PTP-klocka som inte skulle synkroniseras (”PTP är inte låst”, vid varje körning).
> - **En `.daq`-inspelning anger om dess lock valdes eller antogs.** DAQ-U, DAQ-M och DAQ-E levereras utan lock, och Chloros kan inte se om ett är monterat, så en sensor som ansluts utan namngivet lock får som standard `sunshine_cosine`, ett *antagande*. Varje Chloros-inspelning (schema 1.30) lagrar nu `cap_id_source` bredvid `cap_id`: `operator` när du namngav ett lock (inklusive `none`), `auto_default` när det var standard. Vid bearbetning av en inspelning där kondensatorn antogs, med projektets DAQ-kondensator kvar på Auto, skrivs ett `[daq-dls] WARNING` ut som namnger filen: en ren DAQ-U utan solskyddskorrigering ger ett värde som är ungefär 30 gånger för högt. **Namnge den kondensator du monterat** när du ansluter (`connect_daq_sensor(cap_id=...)` eller `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, eller det som anges i GUI:ns rullgardinsmeny för kondensatorer), eller ställ in projektets DAQ-kondensator, så försvinner varningen.
> - **Inga fler pip-konsolfönster i Windows.** På en dator med Python installerat körde backend-programmet och var och en av dess bildbearbetare `pip` en gång vid uppstart, och var och en öppnade ett konsolfönster. Det gör de inte längre.
> - **Live-inspelningar med `.daq` överlever ett strömavbrott.**En inspelning skrivs med en logg och en fullständig disksynkronisering vid varje commit, bakom sensorn, så en långsam disk fördröjer filen men aldrig avläsningarna. En inspelning som avbröts mitt i en commit (strömavbrott, avstängning) kan lämna kvar `<name>.daq-journal` bredvid sig:**kopiera de två tillsammans.** Chloros återställer journalen första gången den läser filen, så inspelningen öppnas med allt som kommiterats före avbrottet (ungefär de sista 3 sekunderna går förlorade); i 1.2.0 kunde ett strömavbrott vid fel tillfälle göra hela filen oläsbar (&quot;databasens diskavbildning är felaktig&quot;).
> - **`DAQSensorSession.record_start()` på en upptagen disk blockerar inte längre sensorn**: avläsningarna köas och skrivs i en egen tråd, och en som inte kunde skrivas räknas in i `status()`:s `record_errors` (tillsammans med `record_last_error`).
> - **Direkt `LatticeCamera`:**en bildruta utan chunk-exponering/förstärkning rapporterar nu kamerans**verkliga** exponering och förstärkning (fallback-värdet läste ett cachelagrad värde som kunde förbli oförändrat under en hel automatisk exponeringsomgång), och `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` tillämpar samma 5 µs-ingångsdeglitch som Chloros kabelslavar får (ofiltrerat, en slav avfyrade bildrutor utan att någon utlösare skickades).
> - **Programvarutriggade kameror tappar inte längre vartannat trigger.** En kamera i programvarutriggerläge köar nu ett trigger som anländer under dess utläsning (eller så väntar Chloros på att den ska bli klar), precis som kabelsynkroniserade kameror redan gjorde.
> - **`discover_daq_sensors()` namnger en DAQ-E-S till `DAQ-E-S`.** Dess `model` var `DAQ-E` för varje Ethernet-sensor; en som inte svarar på identitetskontrollen inom cirka 2 sekunder listas fortfarande, som `DAQ-E`.
> - **Nytt: `chloros-mcp`, en MCP-server för AI-assistenter.** Claude Desktop, Claude Code, Cursor och andra MCP-klienter kan bearbeta bildmaterial (skapa ett projekt, importera bilder, välja inställningar, bearbeta, rapportera vad körningen exporterade) och styra LATTICE-kameror, arrayer och DAQ-ljussensorer: ansluta, förhandsgranska, ställa in exponering och färg, ta bilder, spela in. Dess verktyg täcker vad detta SDK gör. Det installeras tillsammans med SDK, och .deb-paketet för Linux placerar det i `/usr/bin`. Se [MCP-server (AI-assistenter)](#mcp-server-ai-assistants).
> - **Nytt: kameraverktyg**, kommandona i `chloros-cli lattice` som SDK-anrop: färgprofil, mättnad/kontrast, högdagrar, vitbalans (engångsinställning, auto, fasta förhållanden), förinställningar, filtermodeller, firmware-status och uppdatering, ström (avkänning, viloläge, väckning), målkalibrering, nätverkskort (kontroll, uppskattning, korrigering), vignetteringskorrigering på filer, en DLS-avläsning och värdavkänning. Se [Kameraverktyg](#camera-utilities).
> - **Nytt: resten av CLI som SDK-anrop**, så att CLI, SDK och `chloros-mcp` erbjuder samma funktioner (CLI behåller endast inloggning, konfigurering av en DAQ-E och de direkta DAQ-servrarna, av säkerhetsskäl): kameror som öppnas med ett enda kommando (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), justeringsprofilfiler (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), index (`index_catalog()`, `compute_index()`, `capture_live_index()`), PTP-grandmastern (`time_sync_status()` och liknande), projektmappen (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) samt recept (`run_project_recipe()`, CLI:s egen körare). För ljussensorer: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` och `light_sensor_reflectance()` (två sensorer: omgivnings- och objektreflektans samt index); för en kamera: `CameraSession.status()` och `capture_repeated()`; för en matris: `ArraySession.capability()` och `set_member_settings()` (en elements exponering, förstärkning, ...). Se [Transientmatriser, inriktningsprofiler och index](#transient-arrays-alignment-profiles--indices), [Projektmapp och recept](#project-folder--recipes) och [DAQ-sensorsessioner](#daq-sensor-sessions).
> - **Förbättrade LATTICE-indexprodukter** (NDVI och andra index) från `process()`, i alla bearbetningslägen.
> - **LATTICE OCN-kameror: förbättrade produkter för strålningsintensitet, reflektans och index i solljus.** En flottkorrigering tillämpas efter bandsepareringen – i realtid, vid bildtagning och i `process()`; RGN-kameror är oförändrade. Se [Radiometrisk utdata](#radiometric-output-lattice-multispectral-pipeline).
> - **Varje bildsparning sparar den DAQ-avläsning som användes, precis som GUI:n gör.** `CameraSession.capture()` och `ArraySession.capture()` / `continuous()` skriver `.daq` och dess `.match.json` varje gång en DAQ binds eller läggs i poolen, inklusive en inspelning som endast innehåller rådata; i 1.2.0 sparades en rå eller debayered-inspelning endast med `force_daq=True`. `daq=False` stänger av funktionen och `daq_tolerance_ms=` ställer in matchningstoleransen; båda resultaten anger vad som sparades i `.daq_files` och `.daq_matches`. En inspelning väntar nu på DAQ:ns nästa avläsning så att den kan blanda avläsningarna på båda sidor om exponeringen. Se [Den DAQ-avläsning som en inspelning sparar](#the-daq-reading-en-inspelning-sparar--daq_filer-och-daq_matchningar).
> - **`ArraySession` har live-bilder och en kontinuerlig inspelning på serversidan.** `grab()` och `frame_stream()` returnerar numpy-bilder från en utlösare (samma avläsning som projekt-API:s `ArrayHandle.grab()` / `frame_stream()`), och `continuous()` sparar varje synkroniserad grupp i backend och returnerar en [`ContinuousHandle`](#continuoushandle). Se [Live-ramar i din egen pipeline](#live-frames-in-your-own-pipeline).
> - **En array `grab()` är kopplad till en utlösare** (`ArrayHandle.grab()` och den nya `ArraySession.grab()`). I version 1.2.0 läser 12 av 20 fyrakameror `ArrayHandle.grab()` sträckte sig över två triggare. Varje bildruta bär nu `group_id`, och en array `frame_stream()` genererar varje trigger en gång. Se [En utlösare per avläsning](#one-trigger-per-read--group_id).
> - **Bandvisa och justerade bildrutor:** `grab()` / `frame_stream()` tar `band=`, `align=`, `bit_depth=` och `refl_max=` på kamera- och array-handtag. Se [Exportera ett enskilt band](#exporting-a-single-band--band).
> - **Nya `ArraySession`-metoder och argument:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (tillsammans med `CaptureResult.group_seq` och `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` kedjar samman på varandra följande grupper och avbryter vid fullt utrymme.
> - **`ArraySession.record()` registrerar utan öppet GUI.** När GUI:ets kombinerade vy är stängd renderar backend-delen vyn själv: arrayensNDVI eller det index du skickar som `expr=`. `RecorderHandle.stop()` utlöser `ChlorosConnectError` för en inspelning som misslyckades, en där ingen bildruta skrevs, och behåller stoppresultatet på `.result` (att lämna kvar ett `with`-block utlöser inte något undantag: kontrollera `rec.result`). Se [`RecorderHandle`](#recorderhandle).
> - **Nya `DAQSensorSession`-metoder:** `enable_ptp()`, `force_disconnect()` och för en DAQ-E:s egna inställningar `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` och `update_firmware()`. `record_stop()` returnerar `warning` och `gaps`. En omstart som avslutade en inspelning namnger den i sitt svar eller, när omstarten misslyckas, i `ChlorosConnectError` (`recording_closed`; se [DAQ-E-enhetsinställningar](#daq-e-device-settings)). Se [`DAQSensorSession`-metoder](#daqsensorsession-methods).
> - **En poolad DAQ-E som slutar leverera data återansluts av backend**, med de inställningar den hade, och en inspelning på den fortsätter i samma `.daq` med luckan markerad i filen; `record_stop()` returnerar avbrotten som `gaps`. I version 1.2.0 förblev sensorn tyst tills den kopplades bort och anslöts igen, vilket avslutade inspelningen. Se CLI-referensens [En DAQ-E som slutar leverera data](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` fungerar från ett installerat SDK.** Hjulet (installationsprogrammetoch PyPI:s) innehåller nu de projektladdare som behövs; tidigare genererade `open_project()` ett fel med namnet `ImportError` som hänvisade till `project_cameras` / `project_sensors` i alla pip-installerade SDK. `connect_all()` ansluter de sparade kamerorna, matriserna och ljussensorerna via en körande backend (ljussensorer via sin DAQ-pool), och en sparad enhet som den inte har någon möjlighet att söka efter listas under `errors` med en förklaring istället för att hoppas över utan förvarning. En sparad matris som redan är igång i backend-systemet (GUI:ernas, eller `chloros-cli project connect`:s) ansluts och lämnas igång, där den tidigare rapporterades som ett fel (”Kameran  <sn>finns redan i matrisen <id>”). Se [`ChlorosProject`-metoder](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` och `process_lattice_capture()` hittar Chloros på Windows.** De letar där installationsprogrammet placerar backend-modulen (`<install folder>\backend\`, `C:\Program Files\Chloros\backend\` som standard, en anpassad mapp från installationsprogrammets egen post) och ansluter till en backend som redan körs utan att söka efter någon alls. I version 1.2.0 genererade de felmeddelandet ”Chloros-backend hittades inte” på en installerad maskin om du inte angav `backend_exe=`.
> - **En backend som kraschar vid uppstart rapporteras omedelbart.** När `ChlorosLocal()` eller ett smart-connect-anrop startar en backend som avslutas innan den svarar, utlöses felet inom några sekunder: `ChlorosBackendError` från `ChlorosLocal` och `ChlorosConnectError` från smart--connect-anropen, tillsammans med avslutningskoden, kommandot som kördes och de sista raderna som backend-modulen skrev ut. 1.2.0 väntade ut starttimeouten på 60 sekunder och angav inte orsaken. Se [Automatisk start av backend](#backend-auto-start).
> - **Backenden fortsätter att köras mellan skripten.** När man lämnar ett `with ChlorosLocal(...)`-block, `process_folder()` / `process_lattice_capture()` återvänder och `chloros-mcp` avslutas stoppar inte längre en backend som SDK startade; nästa skript återanvänder den. Den avslutas vid `logout()` (såvida inte Chloros-appen är öppen eller en körning pågår), vid `shutdown_backend()`, eller automatiskt efter 30 minuter om inget använder den (`CHLOROS_BACKEND_IDLE_MIN` i backendens miljö ändrar detta; `0`: aldrig).
> - **En backend som SDK:t startat ombeds att stängas av, den avslutas inte med våld.** `shutdown_backend()` låter backend-instansen frigöra sina kameror och ljussensorer och avsluta eventuella inspelningar; den avslutas endast om den fortfarande körs 10 sekunder senare. Version 1.2.0 avslutade den omedelbart (på Windows, en hård avslutning), vilket kunde leda till att en DAQ-ljussensor ansluten till den backenden inte svarade. En backend som SDK hittade igång lämnas fortfarande ifred.
> - **På Linux kör automatisk start aldrig ett `backend_server.py` från den aktuella mappen.** Om ingen backend var installerad kördes ett skript som startades i en mapp som innehöll en sådan; nu startas endast den installerade backenden (eller, för `ChlorosLocal`, den som `backend_exe=` namnger).
> - **`configure(indices=...)`**lägger till varje förinställning i projektet för kamerans band, inklusive ett nytt projekt, och varnar (`UserWarning`) för ett namn som det inte kan lägga till.**`import_images()`** rapporterar `.daq`-inspelningar som `scan_count` och varnar inte längre för en LATTICE-inspelningsmapp.
> - **`read_image_audit_tags()` fungerar efter en pip-installation.** Det läser själv bildens metadata; i version 1.2.0 returnerade det `{}` utan de interna komponenterna från skrivbordsinstallationen i Python-sökvägen.
> - **`capture()` direkt efter `connect_array()`** väntar upp till 20 sekunder på arrayens första bildgrupp istället för att avbryta med HTTP-fel 503 ”Array still warming up”.
> - **`status()` rapporterar ett avbrott.** Det nya fältet `stalled` är True medan hämtningsslingan körs men publicerar ingenting, och `actual_fps` är då 0. I 1.2.0 behöll `actual_fps` sitt senaste värde under ett avbrott, så en avbruten array visade `actual_fps > 0` medan inspelningarna misslyckades med felkod 503 ”Ingen ny synkroniserad bildgrupp”.
> - **Färre ofullständiga bilduppsättningar och 503-fel vid inspelning:** varje slavkameras utlösningsingång är nu avglitchad.
> - **Komprimerad 12-bitarsöverföring** (`connect_array(packed="auto")`) höjer bildhastigheten för hela sensorarrayen med ungefär en tredjedel. Se [Komprimerad 12-bitarsöverföring](#packed-12-bit-transport-packed).
> - **Arrayexponering under ett ROI:** ett litet ROI begränsar inte längre en medlems exponering till sensorns fria löptid (4,9 ms vid 768 × 304). Se [Exponering på arraymedlemmar](#exposure-on-array-members).
> - **Automatisk exponering sänker inte längre arrayens automatiska bildfrekvens.** En kabelsynkroniserad matris körs med den hastighet som nätverket och kamerornas utläsning tillåter; automatisk exponering fyller den period som hastigheten ger och höjer sedan förstärkningen. En manuell exponering som är längre än perioden saktar ner utlösningen för att anpassa den istället för att avvisas. Se [Exponering på matriselement](#exposure-on-array-members).
> - **`read_nodes()`**lägger till `frame_values` och `notes` medan automatisk exponering pågår (se [`read_nodes()` under automatisk exponering](#read_nodes-under-auto-exposure--frame_values-and-notes)). När kameran avvisar en exponering eller förstärkning anger**`set_settings()`** det intervall som den accepterar och återställer det automatiska läget som den hade stängt av för det värdet; `set_settings(target_brightness_auto=True)` är nytt.
> - **`orient=`** på `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` och `ArrayHandle.capture()` / `capture_smart()` roterar produkterna efter kamerans orientering. Se [`ArraySession`-metoder](#arraysession-methods).
> - **`DLS()` kräver sensorns egen `mac_address`.** Det finns inte längre någon standard-MAC (det var en enhets adress); `DLS()` utan en sådan ger upphov till `DLSError`. Se [DAQ-M-bandens magnituder](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Windows-installationsprogrammet** erbjuder alternativen ”Aktivera långa sökvägar i Windows” och ”Uteslut Chloros-backend från Microsoft Defender-skanning”, vilka båda är markerade som standard. Se [Steg 1](#step-1--install-the-chloros-platform-package).
> - **Linux-.deb-paketet** installerar Python-SDK för systemet `python3` (tidigare placerades endast wheel-filen på disken), utökar kärnans mottagningsbuffertar för socklar som kameraströmmarna behöver och lägger till en backend-tjänst som körs som din användare. Jetson-paketet är `chloros_1.3.0_arm64_jp6.deb`, för JetPack 6, med JetPacks CUDA PyTorch. Se [Steg 1](#step-1--install-the-chloros-platform-package) och [Steg 2](#linux-deb).

---

## Arkitekturen på 60 sekunder

SDK:t är ett tunt Python-lager över Chloros-backenden (samma Flask-server som desktop-GUI:n och CLI:n använder). För automatisering importerar du `chloros_sdk` och anropar högnivåmetoder; bakom kulisserna blir varje anrop en HTTP-förfrågan till den lokala backend-servern på port 5000 — `http://127.0.0.1:5000/api/...` (medvetet inte `localhost`, som först omdirigeras till `::1` i Windows och tar cirka 2 sekunder per begäran mot en backend som endast stöder IPv4). Backenden äger hårdvarupoolen — kameror, DAQ-sensorer, justeringsprofiler, bildbuffertar — så att SDK-skript kan samexistera med GUI:t utan att behöva konkurrera om seriella portar eller nätverkskortets bandbredd.

Det finns tre gränssnitt som du kommer att använda:

1. **`ChlorosLocal` + fria funktioner** (`process_folder`, `process_lattice_capture`) – Bildbehandlingspipeline. Kör en hel mapp genom kalibrering / debayer / indexexport från ett enda Python-anrop.
2. **Smart-connect-hanterare** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Öppna en permanent backend-session för live-hårdvara. Samma ”smart-prep”-flöde som i GUI: nätverkssond, automatisk val av nivå, PTP, AE-seeding, GPIO-triggerkonfiguration.
3. **`ChlorosProject` / `open_project`** — Ladda ett sparat projekt (mapp med `cameras.json` + `sensors.json` + `project.json`), anslut dess kameror, arrayer och ljussensorer på en gång, och styr bildtagningarna med namngivna handtag.

Ytorna 1 och 2 **startar automatiskt en lokal backend** om ingen redan lyssnar (samma medföljande binärfil som GUI/CLI startar) — så ett enkelt skript fungerar från ett nytt skal utan att du behöver starta en backend först: `ChlorosLocal` (samt `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` och `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` och `analyze_array_network()` startar aldrig: de behöver en igångvarande backend. Ange `auto_start_backend=False` för att avstå (t.ex. när man pekar på en fjärrbackend som aldrig startas). Se [Automatisk start av backend](#backend-auto-start). Surface 3 beter sig annorlunda: `open_project()` tar inte emot någon `auto_start_backend`-parameter, och `connect_all()` startar aldrig en backend — den söker efter `http://127.0.0.1:5000` en gång och, om inget svarar, faller den tillbaka till direkt (backend-fri) `lattice_sdk`-enhetsstyrning. Det kräver Arena SDK-runtime och OpenCV (`cv2`) som kan importeras i din Python, vilket SDK-hjulet inte installerar; utan dem rapporteras varje sparad kamera och matris som ett fel, med orsaken angiven. Ett projekts ljussensorer har ingen backend-fri väg: utan en körande backend rapporteras varje sparad sensor som ett fel, med orsaken angiven (se [`SensorHandle`](#sensorhandle)). Endast `proj.process()` och `stream(..., overlays=True)` konstruerar en `ChlorosLocal()` (som startar automatiskt) vid behov.

Alla tre är auth-styrda: kör `chloros-cli login` en gång på maskinen, eller logga in via skrivbordets grafiska gränssnitt. Vad ett anrop utan en giltig session ger upphov till beror på gränssnittet (se [Undantag](#exceptions)): `ChlorosAuthenticationError` från `ChlorosLocal` (och `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` från smart-connect-anrop och deras sessioner, `status: "error"` med ett `error_code` från `analyze_array_network()`, samt en post under `errors` i `connect_all()`-rapporten.

Krav:
- Python 3.7+ (enligt paketets specifikation; utvecklat/testat på 3.10)
- Chloros Desktop installerat lokalt (backend-binären medföljer i installationsprogrammet)
- Aktivt Chloros+-inloggning på **valfri betald Chloros+-nivå (Copper eller högre)**; den kostnadsfria**Iron**-nivån har ingen SDK-/CLI-åtkomst. Detta tillämpas**på serversidan****:** varje begäran med SDK- eller CLI-flagga måste innehålla både en aktiv session och ett betalt abonnemang. Ett abonnemang under Copper ger felkoden `403` tillsammans med `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` från `ChlorosLocal`, `ChlorosConnectError` från smart-connect-anropen). En utloggad anropare får istället `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` från `ChlorosLocal`, `ChlorosConnectError` från smart-connect-samtalen) — de två skiljer sig åt eftersom en omkörning av `chloros-cli login` åtgärdar `401` men kan inte åtgärda `403`, vilket kräver en uppgradering av planen.
- Offlineanvändning stöds inom planens respitperiod: nivån läses från cacheminnet för servervalidering (5 minuter) eller från cacheminnet för signerade, maskinbundna licenser (30 dagar för månadsplaner, till prenumerationens utgångsdatum för årsplaner). När denna respitperiod löper ut övergår abonnemanget till gratisversionen och åtkomsten till SDK/CLI avbryts tills enheten kan ansluta till servern en gång. `chloros-cli status` (`GET /api/license-status`) förblir tillgängligt på gratisnivån så att orsaken syns.
- Windows 10/11 64-bitars, **Ubuntu 22.04 LTS eller nyare**, eller Jetson (JetPack 6). Ubuntu 20.04 stöds**inte**: beroendena för `.deb` härrör från vad backend-modulen länkar mot, inklusive `libc6 (>= 2.35)`, och Focal levereras med glibc 2.31.

---

## Installation

Python-SDK:et är ett tunt Python-lager över Chloros-backend. Om du inte kör en backend på en annan maskin (se [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel)), behöver du **Chloros-skrivbordspaketet installerat lokalt** (Windows-installationsprogram eller Linux `.deb`), och det inkluderar DAQ-ljussensorer. Paketet innehåller backend-binären, Arena SDK-runtime för LATTICE-kameror samt kalibreringspaketen.

Senaste nedladdningar: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Steg 1 — Installera Chloros-plattformspaketet

#### Windows (.exe)

1. Ladda ner `Chloros Installer 1.3.0.exe` från nedladdningssidan.
2. Kör installationsprogrammet och följ guiden. Standardinstallationsvägen är `C:\Program Files\Chloros\`.
3. Starta Chloros minst en gång och logga in med ditt Chloros+-konto.

Installationsprogrammet låter backend-programmet (`chloros-backend.exe`, inget annat) att passera genom Windows-brandväggen för varje nätverksprofil: UDP-ingående trafik, för kameraströmmar, PTP, mDNS och DAQ-E-strömmen, samt TCP-ingående trafik endast från det lokala subnätet, för den anslutning som en DAQ-E upprättar tillbaka till datorn under en firmwareuppdatering ([`update_firmware()`](#daq-e-device-settings)). Avinstallationsprogrammet tar bort båda reglerna. Backendens HTTP-API förblir på `127.0.0.1` oavsett vilket.

Guidens grupp **Systeminställningar** har två alternativ, båda markerade som standard:

| Alternativ | Vad det gör | Avaktivera tyst installation |
| --- | --- | --- |
| **Aktivera långa sökvägar i Windows** | Gör det möjligt för projektmappar att ligga djupare än Windows gräns på 260 tecken för sökvägar. Om långa sökvägar är avstängda (Windows standardinställning) avvisas en projektmapp som ligger för djupt för sina produkter innan något bearbetas — se [Felsökning](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Uteslut Chloros-backend från Microsoft Defender-skanning** | Stoppar Defenders realtidsskanning-tidsskanning av varje fil som en kamera skriver. Vid höga bildtagningstakter kostar skanningen cirka 30 % av hastigheten: uppmätt vid ROI 512 på en uppsättning med 9 kameror, kontinuerlig RAW-bildtagning kördes med 52,7 bilder/s med skanning och 77,7 bilder/s med backend undantagen. | `/MERGETASKS="!defenderexclusion"` |

En kontinuerlig inspelning rapporterar Defenders belastning som `defender_cores` (se [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Installera med `apt install ./…`, inte `dpkg -i`: apt installerar även kraven för Python-SDK, vilket paketet rekommenderar snarare än kräver (se [Steg 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

För Jetson Orin / Orin NX / Orin Nano på JetPack 6 (L4T R36). PyTorch är JetPack 6:s CUDA-version, så GPU-bearbetningen körs på Orins GPU; den använder CUDA- och cuDNN-biblioteken som JetPack 6 installerar (`sudo apt install nvidia-jetpack` på en image som saknar dem).

#### Linux: mottagningsbuffertar och backend-tjänsten

Båda .deb-paketen ökar storleken på kärnans socket-mottagningsbuffertar, som LATTICE-kameraströmmar behöver, till 32 MB (`/etc/sysctl.d/60-chloros-gige.conf`). Med kärnans standardinställning tappas kamerapaket bort i kärnan och en array förlorar bildrutor och bildhastighet; `chloros-cli lattice network` rapporterar när buffertarna börjar ta slut. Se CLI-referensens [Linux-värdar — socket-mottagningsbuffertar](cli-reference.md#linux-hosts--socket-receive-buffers).

På en maskin utan grafiskt gränssnitt kör du backend som en tjänst **under din användarkonto**, så att ett skript hittar en som körs (`connect_all()` startar aldrig en backend):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Backenden körs då som du: de inspelningar den skriver är dina, och den använder din `chloros-cli login`. Root-`chloros-backend.service` är fortfarande installerad för maskiner där den är aktiverad, men varje fil som en root-backend skriver tillhör root. Se avsnittet [Kör backend som en tjänst](cli-reference.md#running-the-backend-as-a-service-headless) i CLI-referensen.

En backend som tjänsten kör är delad och stoppas aldrig av en klient: SDK-skript och `chloros-cli` använder den som den är (`ChlorosLocal.shutdown_backend()` stoppar endast en backend som `ChlorosLocal` själv startade), och `GET /api/status` namnger enheten som `service`. Starta om den med `sudo systemctl restart chloros-backend@$USER.service`. När paketet uppgraderas stoppas backend-tjänsterna under installationen, så ett skript som körs förlorar sina kameror och sensorer och startar sedan om alla aktiverade enheter; när paketet avinstalleras stoppas de och de aktiverade inaktiveras. Endast en backend körs åt gången, så aktivera en enhet: en andra vägrar att starta och förblir `failed` (avslutningsstatus 98, startas inte om), och dess `systemctl status` förklarar varför. För att byta enhet, inaktivera den du inte använder.</id></sn>

### Steg 2 — Installera Python-SDK

**Chloros-installationsprogrammet levereras med ett passande SDK-wheel.** Varje Windows-installationsprogram och varje .deb-paket för Linux placerar en fil med namnet `chloros_sdk-X.Y.Z-py3-none-any.whl` på hårddisken som exakt matchar versionen av GUI, CLI och backend. Du behöver inte hålla koll på PyPI för att hålla dig uppdaterad.

#### Windows

Installationsprogrammet kör automatiskt `pip install` mot det medföljande wheel-paketet med hjälp av ditt systems Python (`py.exe`-startprogrammet föredras, faller tillbaka till `python -m pip`). Ingen åtgärd krävs – `import chloros_sdk` fungerar i din Python-miljö efter en lyckad installation. Om det inte finns någon Python på datorn hoppar installationsprogrammet tyst över detta steg och GUI + CLI fortsätter att fungera.

#### Linux (.deb)

Båda .deb-paketen (x86_64 och Jetson) installerar SDK:t för systemet `python3`, så `import chloros_sdk` fungerar efter installationen, utan pip och utan nätverk. Paketen (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` och projektladdarna) placeras i `/usr/lib/python3/dist-packages`, och SDK:s kommando `chloros-mcp` ([MCP-servern](#mcp-server-ai-assistants)) i `/usr/bin`, vilka alla tillhör paketet: en uppgradering ersätter dem och om paketet tas bort tas även de bort.

SDK:s egna krav, `python3-numpy` och `python3-requests`, är **rekommenderade, inte obligatoriska** (GUI, CLI och backend är fristående): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` på en Jetson) installerar dem, medan `dpkg -i` inte gör det. Installationsutmatningen visar vilken SDK-version `python3` importerar, eventuella krav som saknas, samt en `chloros_sdk` någon annanstans i `python3`:s sökväg som överskuggar paketets kopia (till exempel en som installerats tidigare med `pip install --user`).

I en virtualenv, som inte ser systemets paket, installerar du det medföljande wheel-paketet i den:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

För Jetson-installationer i air-gapped-miljöer sker detta helt offline — SDK:et och wheel-paketet finns redan på disken.

#### Offentlig PyPI

För värdar som endast använder pip (inget Chloros-skrivbordspaket installerat; SDK-et driver då en backend på en annan maskin, se [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI innehåller den senaste stabila versionen. Se till att SDK:et har samma version som den Chloros-backend som det styr (se [Versionering och kompatibilitet](#versioning--compatibility)). DAQ-M-hjälpprogrammet `DLS` behöver också `bleak`: `pip install "chloros-sdk[camera]"` (eller `pip install bleak`).

#### Verifiera

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+-abonnemang krävs.** Alla SDK-anrop kräver en aktiv Chloros+-inloggning. Kör `chloros-cli login user@example.com 'YourPassword'` en gång per maskin; inloggningsuppgifterna cachelagras i `~/.chloros/`.

### Behöver jag Desktop-paketet?

Enbart pip-paketet räcker **inte** för de flesta arbetsflöden. Här är vad varje SDK-gränssnitt behöver:

| SDK-gränssnitt | Behöver Desktop-paketet? | Varför |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ja**(för att starta en)**/ Nej** (en backend som redan är tillgänglig) | Använd en igångvarande backend (den från desktop-appen eller en annan igångsatt skript); annars starta den installerade automatiskt: `/usr/lib/chloros/chloros-backend` (Linux) eller `<install folder>\backend\chloros-backend.exe` (för Windows; `C:\Program Files\Chloros\backend\` som standard, en anpassad installationsmapp läses in från installationsprogrammets register). Ange `backend_exe=` för att namnge den själv. |
| `chloros-mcp` ([MCP-server](#mcp-server-ai-assistants)) | **Ja**(för att starta en)**/ Nej** (en backend är redan tillgänglig) | Byggt på `ChlorosLocal`, alltså samma som raden ovan. |
| `read_image_audit_tags` | **Nej** | Läser den bearbetade bildens egna metadata (endast Python-standardbiblioteket). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ja**(lokalt)**/ Nej**(fjärr) | Rena HTTP-klienter via backend. Lokal backend → skrivbordspaket krävs. Fjärrbackend → `backend_url=`**via en tunnel** (se Fjärrbackend-läge — medföljande backends binder endast till loopback). |
| `ChlorosProject` / `open_project` | **Ja** | Hanterar sparade projekt via backend. |
| Direkta LATTICE-klasser (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ja** | Kräver Arena SDK-runtime, som ingår i desktop-paketet, samt OpenCV (`cv2`) i din Python-miljö, vilket varken desktop-paketet eller SDK installerar (`pip install opencv-python`). `CAMERA_AVAILABLE` är annars `False` vid import. `DLS` behöver också `bleak`. |

### Fjärr-backend-läge (endast pip-värd, via tunnel)

> **Den installerade backenden är inte nåbar via LAN.** Den lyssnar endast på
> loopback (`127.0.0.1` / `::1`), så `backend_url="http://<lan-ip>:5000"`
> **kan inte nå en installerad Chloros**. För att driva en backend på en annan
> maskin måste du själv vidarebefordra dess loopback-port och peka SDK:t mot
> tunneln:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Headless-/CI-/robotikvärdar kan ha en maskin med fullständig skrivbordsinstallation som ”Chloros-server” och `pip install chloros-sdk` på alla andra ställen — men transporten mellan dem sker via den ovan nämnda, användararrangerade tunneln, inte via en direkt LAN-URL.

> **`ChlorosLocal` på en värd som endast använder pip.** `ChlorosLocal(backend_url=BACKEND)` ansluter till en backend som den kan nå utan en skrivbordsinstallation: den söker efter en lokal backend-binärfil endast när den måste STARTA en (inget svarar och `auto_start_backend=True`). Fram till version 1.2.0 sökte den först efter binärfilen och genererade felmeddelandet `ChlorosBackendError` (”Chloros-backend hittades inte…”) på en värd med endast pip, även om en backend var tillgänglig. De mappar du anger (`process_folder`, `import_images`) kontrolleras på den här datorn och läses av backend-servern, så vid en fjärrbackend måste de peka på samma mapp på båda, till exempel en delad enhet.

### Arbetsflöde enbart för DAQ (värd enbart med pip)

`pip install chloros-sdk` kan inte på egen hand driva en DAQ-sensor: en DAQ-sensor drivs alltid via en Chloros-backend med `connect_daq_sensor()`:

- **Sensor på denna maskin:** installera Chloros-skrivbordspaketet här. `connect_daq_sensor()` startar sin backend om ingen körs.
- **Sensor på en annan maskin:** den maskinen behöver desktop-paketet; nå dess backend via tunneln i [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel). Backenden öppnar sensorn på sin egen maskin, så en DAQ-U är ansluten till den datorn, en DAQ-M befinner sig inom Bluetooth-räckvidden för den och en DAQ-E finns i ett nätverk som den kan nå.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Oavsett vilket tillämpar backend-enheten Chloros+-planen: logga in på den maskin som kör backend-enheten (`chloros-cli login` eller skrivbordsappen) med valfri betald nivå (Copper eller högre). En utloggad backend och ett abonnemang under Copper genererar båda `ChlorosConnectError` med backendens orsak (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Snabbstart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## API-index på högsta nivå

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Bildbehandling — `ChlorosLocal`

Den övergripande pipelineklassen. Startar backend när den konstrueras (om ingen svarar och `auto_start_backend=True`), skapar/konfigurerar projekt, övervakar framsteg och returnerar sammanfattningar efter körning.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metoder

| Metod | Beskrivning |
| --- | --- |
| `create_project(project_name, camera=None)` | Skapa ett nytt projekt (valfritt med en kameramall som `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Öppnar ett sparat projekt (dess mapp eller dess namn i projektmappen) och gör det till det öppna projektet. Sökvägarna för ett flyttat projekt omdirigeras till den mapp där det nu finns; `relink` anger namnet på den mapp där de övriga saknade filerna nu finns. Returnerar backendens svar med `source_files`; utlöser `ChlorosProjectFilesMissingError` när inga kan hittas. Se **Ett flyttat projekt** nedan. |
| `import_images(folder_path, recursive=False)` | Importera RAW/TIF/JPG/DNG-bilder **samt `.daq` ljussensorinspelningar**. Returnerar `count` (bilder) och `scan_count` (inspelningar). Varnar endast om mappen inte innehåller något av dessa. |
| `export_light_sensor(daq=True, csv=True)` | Skriv kalibrerade `.daq` + `.csv` för varje ljussensorinspelning i projektet, till `<project>/Light Sensor/`. Se [Ljussensorinspelningar](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Ställ in bearbetningsreglagen. `indices` är förinställda namn (`NDVI`, `GNDVI`, `OSAVI`, …), som alla läggs till i projektet för respektive kameras band; ett namn som inte är en förinställning, eller som banden inte kan bilda (`NDRE` kräver RE och NIR, vilket ingen enskild kamera tillhandahåller), får namnet `UserWarning` och ingen indexbild. `daq_log_path` namnger en inspelning från en ljussensor (en `.daq`- eller `.csv`-fil, eller en mapp med sådana) som varje bildruta jämförs mot, både i LATTICE och i Survey3, istället för de inspelningar som importerats tillsammans med bilderna; om en sökväg saknas eller inte innehåller någon inspelning anges detta, och de importerade används. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Kör pipelinen. Returnerar `{"status": "complete", "async": False}`, samt en `summary`-nyckel om backend-modulen tillhandahåller en sådan — se [Sammanfattning och tips efter körning](#post-run-summary--hints) — `export_check_failure` när körningens export inte kunde verifieras, och `image_failure` när en bild gick förlorad. `wait=False` returnerar `{"status": "started", "async": True}` på en gång. |
| `get_config()` / `get_status()` / `status()` | Kontrollera backend-status. |
| `logout()` | Logga ut: rensar de cachade inloggningsuppgifterna och frigör den här maskinens enhetsplats (`device_deregistration` i resultatet). En pågående bearbetningskörning stoppas inte: den avslutas i det läge den startades i (`run_in_progress: true` och `run_mode` i resultatet), och nästa körning använder standardbearbetning. |
| `shutdown_backend()` | Stoppa backend-modulen om denna instans startade den; en backend-modul som den upptäckte var igång lämnas ifred. Att lämna kvar ett `with`-block eller att instansen genomgår skräpinsamling stoppar inte den: backenden fortsätter att köras för nästa skript och stänger av sig själv efter 30 minuter om inget använder den. Backenden ombeds att stängas av, så den frigör sina kameror och ljussensorer och avslutar först eventuella inspelningar; den avslutas endast om den fortfarande körs 10 sekunder senare. |
| `discover_cameras()` | Upptäck LATTICE-kameror **via denna instans backend** (`/api/camera/discover`). Returnerar en lista med ordböcker (`serial`, `model`, `ip`, …) — samma form som GUI/CLI ser. Tom lista om inga hittas eller om backend inte kan nås. |
| `camera_capture(output_dir, format="tiff", **settings)` | Fånga en enskild bildruta**via backend**(startas automatiskt av detta handtag) så att den får samma förberedelse som GUI/CLI (standard 12-bitars, återanvändning av pool, inbäddade kalibreringsmetadata). Bestäm målet med `serial=` eller `device_index=`; skicka `exposure`/`gain`/`pixel_format`/`preset` som `**settings`. Returnerar den äldre metadatadiktionen (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Genererar överlagrade sammansatta förhandsgranskningsbilder från en sammanslagen kamera — tunn MJPEG-klient via backendens `/api/camera/<serial>/stream-annotated`-väg (zebra / rutnät / hårkors / histogram / peaking / punkt ritad på serversidan). `decode=True` genererar BGR-matriser och kräver OpenCV (`cv2`), vilket SDK:t inte installerar; `False` genererar råa JPEG-byte. Kan även nås per projekt som `ChlorosProject.stream(overlays=True)`. |

**En pågående körning behåller sitt projekt.** Medan en bearbetningskörning pågår – oavsett om det är detta skripts, appens, `chloros-cli process`:s eller `chloros-mcp`:s – vägrar backend att skapa, öppna, importera till eller ändra bearbetningsinställningarna för ett projekt (HTTP 409 `PROCESSING_IN_PROGRESS`): `create_project()` och `import_images()` genererar `ChlorosProcessingError`, och `configure()` utlöser `ChlorosConfigurationError`, och `process()` med ett annat läge än det som är aktivt utlöser `ChlorosProcessingError` (en körning behåller det läge den startade i), var och en med backendens meddelande. En inspelning som sparats i projektet läggs sedan till när körningen avslutas (dess svar `project_add` anger detta). Vänta tills körningen är klar (`get_status()`) och fortsätt sedan.**Ett flyttat projekt.** Ett projekts `project.json` namnger varje källfil (varje bild och ljussensorinspelning) med dess fullständiga sökväg, så om projektmappen – eller hela projektmappen – flyttas till en annan enhet fortsätter de att hänvisa till den gamla platsen. Varje gång projektet öppnas åtgärdas det som går att åtgärda automatiskt: en fil som tidigare fanns i projektmappen söks i den mapp där projektet nu finns, och varje sökväg som hittas där omdirigeras och sparas (`project.json` säkerhetskopieras först som `project.json.before-relink-<date>-<time>`). En sökväg omdirigeras endast till en fil som faktiskt finns där. Filer som låg utanför projektmappen – en hubbsession, en import via ”Lägg till mapp” – hittas endast med `relink=<the folder they are in now>` (eller någon mapp ovanför den). `open_project()` returnerar `source_files`: `total`, `found`, `missing`, `missing_roots` (de mappar där de saknade filerna fanns), `missing_outside_project` och `relink` (vad som omdirigerades, från var till var). Ett projekt där vissa filer saknas öppnas med resten; ett projekt där inga filer hittas öppnas inte, och `open_project()` genererar `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). CLI:s `project open --relink` och `chloros-mcp`’`open_project` (`relink_folder`) gör detsamma.

Använd som kontextmanager för garanterad rensning:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Ljussensorinspelningar — kalibrerade `.daq` + `.csv`

En DAQ-U / DAQ-M / DAQ-E kan registreras **utan** sitt kalibreringspaket. Det är
vad de offentliga [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) gör som standard: de skriver ut råa sensorvärden och märker
filen så att Chloros hämtar sensorns fabrikskalibrering **först från serienumret** — lokal cache
först, sedan MAPIR-molnet — och tillämpar den vid import.

Chloros skriver ut resultatet som två produkter per inspelning, under
`<project>/Light Sensor/`:

| Produkt | Vad det är |
| --- | --- |
| `<name>_calibrated.daq` | Det återbearbetningsbara arkivet — samma schema som en liveinspelning, men nu med angivelse av det paket som genererade det. Att importera det på nytt innebär **inte** att det kalibreras en andra gång. |
| `<name>_calibrated.csv` | Spektral irradians i W/m²/nm på sensorns eget våglängdsraster, en rad per avläsning, plus fotometriska kolumner (total effekt, fotopisk/skotopisk lux, PPFD och dess uppdelning i blått/grönt/rött, toppvåglängd). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Endast sensorer utan paket (DAQ-A).** Råa spektrala sensorräkneställningar — *inte* irradians. Se nedan. |

`<name>` är inspelningens eget filnamn. Två inspelningar i olika mappar kan dela
en gemensam avläsning per bildtagning (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) som sparats i
samma millisekund av två kameror — och var och en får ändå sina egna produkter: den första behåller
`<name>`, den nästa lägger till de mappar som skiljer den åt **efter** namnet
(`<name>__<capture>_daq_calibrated.daq`), så att inspelningens datum och tid fortfarande står först.

`process()` utför denna export som ett av sina steg. Det kräver **inte** bilder:
en ljussensor som flygs på egen hand är ett förstklassigt arbetsflöde, och ett sådant projekt har noll
bilder enligt konstruktion.

**DAQ-A-inspelningar exporteras som råa räkneställningar.** DAQ-A-familjen är äldre än systemet med
serienummerbundlar och har ingen bundel att hämta – den kalibreras istället ute på fältet mot ett
reflektansmål, vilket är anledningen till att den aldrig behövt någon. Dessa inspelningar exporteras
med filnamnsstammen `_raw` istället för `_calibrated`: ett annorlunda filnamn istället för en flagga
inuti filen, eftersom uppgiften måste klara att skickas vidare via e-post som ett rent filnamn.
`.csv`-rubriken anger `raw spectral sensor counts (NOT irradiance)` och varnar för att
värdena är jämförbara **inom** filen – precis vad målbaserad kalibrering använder
dem till — och inte mellan olika sensorer. De effektberoende fotometriska kolumnerna (total effekt,
fotopisk/skotopisk lux, PPFD) returneras som **NULL** istället för att integreras utifrån räknevärden.

En DAQ-U / DAQ-M / DAQ-E vars paket helt enkelt inte kunde hämtas **hoppas fortfarande över**,
och skrivs inte i råformat: där finns paketet och ”anslut om och bearbeta om” är ett konkret råd.

Äldre **v1.01 / v1.02**-inspelningar (en DAQ-A-SD skriver dessa) har ingen epok per avläsning,
utan endast filens skrivtid. Bild↔nedåtriktad matchare avvisar dem fortfarande — att matcha en
ram mot en skrivtid skulle vara fel utan att det syns – men exportören läser dem, och
i CSV-filen skrivs `clock=daq_created_on` ut så att produkten anger vilken klocka den använder.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

En inspelning vars kalibreringspaket inte kan hämtas (offline, eller en sensor utan
kalibrering i filen) rapporteras under `skipped` **med orsaken**. Den skrivs aldrig
ut som en ”kalibrerad” fil som innehåller råa mätvärden – anslut till internet och
kör om, så slutförs exporten.

### Återanrop för förlopp

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Sammanfattning och tips efter körning

När processen är klar hämtar `process()` `GET /api/processing-summary` och bifogar innehållet som `result["summary"]`. Hämtningen sker efter bästa förmåga och blockerar aldrig en lyckad retur — om sammanfattningen inte är tillgänglig faller `process()` tillbaka till den vanliga `{"status": "complete", "async": False}`-formen. Varje post i `summary["hints"]` — fullständiga meningar med föreslagna åtgärder, t.ex. varför en körning gav nollutdata — återges också som en Python-`UserWarning`, så körningar med nollutdata är självdiagnostiserande även om du aldrig granskar dict:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` är den maskinläsbara delen:

| Nyckel | Vad den räknar |
| --- | --- |
| `models` | Kameragrupper i körningen. |
| `images_in_groups` | Källbilder i dessa grupper. |
| `targets_found` | Detekterade reflektansmål. |
| `images_calibrated` | Bilder som körningen skapade en reflektansprodukt för. |
| `images_failed` | Bilder som misslyckades och inte exporterades. |
| `exported_files` | **Bildproduktfiler som körningen skapade**: alla produkter (debayered, förhandsgranskning, strålningsintensitet, reflektans, index, målrenderingar), i alla format. |
| `reflectance_files` | Reflektansprodukterna bland dessa (`Reflectance_Calibrated_Images`), oavsett om de kalibrerades av ett mål eller av ljussensorns (DAQ) irradians. En körning utan mål som ändå skrev ut reflektans anger detta i ett tips; endast en körning som inte skrev ut något får meddelandet att dess reflektansutdata saknas. |
| `clamp_masks_written` | [Unmix-klämmaskerna](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) som står skrivna bredvid dem — komplement, inte produkter, så de finns inte i `exported_files` (CLI:n räknar dem separat på samma sätt). |
| `captures_left_out` | Råbilder i projektets `captures`-mapp som denna körning inte tog. En körning väntar vid start på vad som fångats in före den, så detta är 0 om inte en inspelning sparades under körningen eller väntetiden löpte ut. `summary["captures_left_out_note"]` anger vad som ska göras med dem. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Inspelningar från ljussensorer, som avsiktligt räknas separat — de kommer från ett annat steg och finns även för körningar utan bilder alls, så att slå ihop dem skulle få en körning som endast omfattar datainsamling att se ut som om den exporterade bilder. |

Vid sidan av dessa: `summary["output_dirs"]` (varje katalog som skrivits till),
`summary["light_sensor_export"]`, `summary["stopped"]` (sant när användaren avbröt
körningen, så att partiella räkningar inte tolkas som en avslutad körning som underpresterade), och
`summary["groups"]` (uppdelningen per grupp).

`exported_files` registreras av pipelinen **när den skriver**, inte genom att skanna av
projektets bildobjekt i efterhand. Parallell- och GPU-strategierna bygger sina egna bildobjekt
(i arbetarsubprocesser för GPU-vägarna), så den gamla skanningen rapporterade
`0 file(s) written` för varje sådan körning och skickade sedan ut tipset om noll-exporter – vid körningar
där allt hade fungerat. Om du skriver ett skript baserat på detta nummer rapporterar en felfri parallellkörning nu
ett värde som inte är noll.

Hoppade bildobjekt rapporterar den orsak som läsaren faktiskt fastställde för varje fil – ett
oläsbart schema, en saknad bunt, ett skrivfel – **deduplicerat**, så att tjugo filer
som hoppades över av en orsak räknas som en orsak istället för tjugo upprepningar av den.

> **`process()` genereras inte när en körning inte producerar några bilder.** Detta är en av de två punkter där SDK och
> CLI:n medvetet skiljer sig åt: `chloros-cli process` behandlar ”produkter begärdes, inga
> skrevs” som ett fel och avslutas med ett värde som inte är noll, medan SDK återgår normalt och rapporterar
> tillståndet via `summary` / hints. Om din pipeline ska avbrytas vid en tom körning, kontrollera den
> själv – granska `summary` (eller räkna filerna i projektmappen) istället för att förlita dig på
> avsaknaden av ett undantag. De vanligaste orsakerna är en ingångsmapp som inte kändes igen som en
> inspelning och produkter som hoppats över eftersom de inte är tillämpliga för de befintliga kamerorna (t.ex. strålning från kameror som endast använder RGB
> ).
>
> `process()` utlöses inte heller när en körnings exporterade data inte kunde verifieras: det varnar och sätter
> `result["export_check_failure"]`. Produkterna skrivs ut. För att avbryta vid detta, kontrollera den nyckeln. En körning som
> har förlorat en bild fungerar på samma sätt: den varnar och sätter `result["image_failure"]`, vilket gör att `chloros-cli process`
> avslutas med ett värde som inte är noll.

### Hjälpfunktioner

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Varje körning sker inom sin egen `with ChlorosLocal(...)`; en backend som den var tvungen att starta fortsätter att köras när den återvänder, inför nästa anrop, och avslutar sig själv efter 30 minuter om ingen använder den. Ingen av dem tar emot `auto_start_backend`, `radiometric_output` eller `daq_log_path`; använd istället `ChlorosLocal` och `configure()` för dessa. `process_lattice_capture()` accepterar inte `camera`, `debayer` eller `ppk`: den hoppar över paneldetektering och använder standard-debayer.

### Värden som stöds

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrisk utdata (LATTICE multispektral pipeline)

LATTICE-multispektral (M3C/M3M)-exportnivån för `process`-pipeline — `reflectance` (standard), `radiance`, `sensor-response` eller `all` (alla tillämpliga lägen per bild) – anger projektets **”Exportera strålning”**och**”Exportera reflektans”**: `radiance` exporterar endast radians, `reflectance` endast reflektans, `sensor-response` varken eller (endast de debayrade produkterna och förhandsgranskningsprodukterna), `all` båda. Produkterna för debayering och förhandsgranskning kan inte ändras på denna nivå. `configure()` har ett särskilt nyckelord för detta:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Den avancerade nödutvägen — att skriva in produkterna via `custom_settings` — fungerar fortfarande, men kom ihåg att den ersätter hela inställningsblocket (se varningen nedan). Skriv in nycklarna `"Export <product>"`: ett projekts egna inställningar för `"Export radiance"` / `"Export reflectance"`-inställningar har företräde framför den äldre nyckeln `"Radiometric output"`, så den nyckeln i sig ändrar ingenting.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (standard) delar kamerans strålningsintensitet med den **tidsstämplade DAQ-nedstrålningen**, som automatiskt beräknas utifrån en inspelad `.daq` (DAQ-U/M/E)**eller en DAQ-M-inbyggd `.csv`**som finns tillsammans med bildmaterialet; eventuella kalibreringspaket per kamera eller DAQ som saknas lokalt**hämtas automatiskt från AWS** vid första användningen. CLI:n exponerar detta enligttyp av produktväljare på `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Vems avläsning är det?**En `.daq` som en bild sparar är avläsningen från den eller de kameror den beskriver — namngivna i en `<file>.daq.cameras.json`-sidecar bredvid den (enstaka, synkroniserade och array-inspelningar; en rå bildserie namnger varje medlems avläsning i sitt manifest) — och matchas**endast med bilder från dessa kameror**. En fristående kamera utan egen avläsning lånar därför inte längre en annan kameras DAQ; koppla en DAQ till den, eller spela in en. En DAQ:s egen inspelning (`chloros-cli daq pool-record`, eller en inspelning som startats i GUI:n), eller vilken `.daq` som helst utan den sidokaraktären (även äldre inspelningar ingår), fungerar fortfarande för alla kameror.**OCN-kameror i solljus.** Varje OCN-kalibreringspaket är sammanställt under smalbandigt ljus, en LED per band, medan solljus blandar banden. Chloros 1.3.0 tillämpar en snabb korrigering baserad på solljus efter bandsepareringen, vilket förbättrar OCN-produkterna för strålning, reflektans och index i solljus. Den körs överallt där separeringen sker – `process()`, bilder, `grab()` / `frame_stream()` och livevisningen – för OCN-paket som stöder den; RGN-kameror förblir oförändrade. OCN-bilder som tagits med en tidigare version får också denna korrigering när de bearbetas på en dator som har den kalibrering som användes vid fotograferingen (Chloros behåller kamerans kalibrering när kameran väl har anslutits); i annat fall bearbetas de som tidigare. OCN-värdena för strålning, reflektans och index skiljer sig därför från dem i version 1.2.0.

> `custom_settings` **ersätter** hela blocket med beräknade inställningar (det kringgår `configure()`:s övriga nyckelord och validering enligt design). När du använder det ska du inkludera alla `Project Settings`-nycklar som är viktiga för dig, som i exemplet ovan.

#### Läsning av indexpixlar (NDVI och de andra vegetationsindexen)

**En heltalindexraster är en kontraststräckning per bild, inte en −1…1-karta.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` och `JPG (8-bit)` indexprodukter — inklusive den 8-bitars TIFF-fil som en 8-bitars bild sparas som — lagrar

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

där `IndexMin` / `IndexMax` är **denna bildrams egna lägsta och högsta ändliga indexvärden**. DN 0 är inte −1, och full skala är inte +1.

Chloros märker båda ändpunkterna så att mappningen förblir inverterbar:

| Tagg | Betydelse |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konventionen nedan |
| `Chloros:IndexMin` | indexvärdet lagrat som DN 0 |
| `Chloros:IndexMax` | indexvärdet lagrat som full skala |

Läs in dem och invertera, normalisera det lagrade DN-värdet till 0…1 utifrån det lagrade datatypens maxvärde först, så att ett uttryck täcker alla heltalformat:

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

> **`IndexMin == IndexMax`** betyder att varje pixel i den bilden hade samma indexvärde. Rastern är fylld till halva skalan och det finns inget att dividera med — ta ändpunkten.

> **Inga `Chloros:Index*`-taggar innebär att ingen mappning har angetts.** En LUT-export är en färglagd rendering vars pixlar består av gradientfärger som är kopplade till ditt LUT-tröskelfönster, och en bildruta utan pixlar med ett ändligt index har inget intervall att ange. Antag inte att det finns någon mappning i något av fallen — samma regel gäller som vid en saknad `PixelScale`.**Eftersom sträckningen sker per bild är två bilder från samma flygning inte jämförbara som DN**— varje bild har sina egna ändpunkter, så samma NDVI hamnar på olika DN i olika bilder. För kvantitativt arbete över en flygning (zonstatistik, fasta tröskelvärden, tidsserier), exportera**`TIFF (32-bit, Percent)`**: indexraster skrivs där som float32**indexvärdena själva**, utan omskalning och utan att taggar behövs. Trots formatets namn är indexprodukterna inte ×100 – den skalningen gäller endast reflektans med heltalstyp.

#### Unmix-klämmask – vilka pixlar ett band klämdes på

Den spektrala 3×3-unmixen kan göra att ett band blir negativt – där ljuset knappt innehåller det bandet, eller där scenen faller utanför vad kamerans unmix-matris beskriver. Det är inte ett verkligt värde, så exporten sätter det till 0 — och livevyn visar samma 0. Ett band kan klämmas fast på några få pixlar eller på nästan alla; masken registrerar båda fallen. `Chloros:Band{R,G,B}UnmixClampFraction` anger hur stor del av en bildruta detta gäller; klämmasken anger **vilka pixlar**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— i en `clamp_masks/`-mapp**bredvid** produktens mapp, aldrig inuti den (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — för varje produkt avstrålning, reflektans och index där minst en pixel har klämts fast. Enbands-`uint8`, på produktens eget pixelrutnät (orientering, ROI, binning och arrayjusteringsförvrängningen inkluderade). Varje mask namnger produktens kamera (`Make` / `Model` / serienummer), inspelningstid och objektivtaggar, samt listar de band som dess bitar representerar (`XMP-Camera:BandName`, bit 0 först); dess `ImageDescription` och `Software` markerar den som en klämmask, och den innehåller ingen kalibrering.
- **Bitar**: bit 0 = bandet i produktens R-kanal (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). En aktiverad bit innebär att pixelvärdet i det bandet har klämts fast till 0 — betrakta det, och alla index som byggs utifrån det, som låg tillförlitlighet.
- **`Chloros:UnmixClampMask`** stämplas på varje produkt som har genomgått unmix-processen: maskens sökväg relativt produktens egen mapp (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, alltid `/`-separerad — lägg till den till bildens mapp för att öppna masken), eller `none` när ingen pixel har begränsats. Ingen tagg innebär att produkten aldrig har genomgått unmix-processen (RGB- eller monokameror, rådata/debayered-nivåer).
- Capture-resultat innehåller den som bildrutan **`clamp_mask`** (maskens sökväg, eller `None` när inget har klämts) — i `CameraSession.capture()`-resultat, `ArraySession.capture()`:ss `CaptureResult`-ramar och HTTP-inspelningssvaren. Offline-byggnaden listar sina masker som `clamp_masks`.
- Ett live-index multibandstack (`/api/camera/cli-array-index` med `save_multiband`) får en mask per källkamera, i utdatamappens egen `clamp_masks/` — se CLI-referensen`lattice index`. En video har ingen mask: resultatet från offline-byggnaden rapporterar istället varje bildruts klämningsfraktion.

Eftersom ingen produktmapp innehåller en mask, oavsett nivå, kan en produktmapp överlämnas till fotogrammetriprogramvaran som den är – inklusive undermappar – och den tar endast emot bilder. Överlämna produktmappen, inte mappen ovanför den. Chloros egen import hoppar över maskerna.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect för LATTICE-kameror

Persistenta backend-sessioner för live-hårdvara. Samma slutpunkter som GUI:n använder, så beteendet är identiskt i SDK, CLI och GUI.

### Enstaka kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

Den exponering, förstärkning och det mål som ställs in här behålls som dina: Chloros automatiska exponeringskontroll (inklusive den DAQ-mätta exponeringen) lämnar dem ifred. `cam.set_settings(exposure_auto="Continuous")` (eller `gain_auto=`) överlämnar exponeringen (förstärkningen) tillbaka, och `cam.set_settings(target_brightness_auto=True)` återlämnar målvärdet till Chloros automatiska målvärdesinställning.

#### `connect_camera()` Signatur

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Den återvänder när strömmen har startat.** Backend väntar, inom vissa gränser (cirka 15 sekunder tills strömmen startar, sedan 10 sekunder tills den första kompletta bufferten är klar – inte tills den första förhandsvisningsramen, som följer cirka 1,5 sekunder senare), så ett `status()` direkt efter det står det `streaming`. Sessionens `connect_stream` är svaret `stream`-block: `state` `streaming`, `no_frames` (strömmen startade men levererade ingenting — det framgår av `note` och, för en kamera med jumbo-paket, att återansluta med en paketstorlek på 1500 byte), `starting` eller `start_failed`, med `waited_s`. `None` för en kamera som redan var ansluten. En förhandsgranskning (`get_camera_preview`, `/api/camera/jpeg/<serial>`) som begärs direkt efter väntetiden, begränsad till 4 sekunder, väntar på kamerans första förhandsgranskningsbild istället för att svara ”Ingen bild tillgänglig ännu”; en kamera vars ström inte har levererat något (`no_frames`, eller aktiverad för en hårdvarutrigger) svarar fortfarande omedelbart.
- **Paketstorleken bestämmer du själv.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` öppnar kameran med paket på 1 500 byte, utan att testa (`chloros-cli lattice cam-connect --packet-size standard` skickar samma).
- **En anslutning i taget upptar kamerapoolen**: öppningen, sonderingen av paketstorlek, en första kalibreringsnedladdning (cirka 70 s för en kamera som är ny på den här datorn), strömstarten. En begäran som behöver en kamera under tiden (`read_nodes`, `set_settings`, en inspelning, en andra anslutning) väntar på den i högst 5 s, misslyckas sedan med HTTP-kod 503 `CAMERA_POOL_BUSY`, där den anslutande kameran namnges (`ChlorosConnectError`); fråga om en stund. Listorna väntar aldrig: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` och `CameraSession.status()` svarar omedelbart. En kamera som är mitt i en anslutning listas under poolens svar `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — och `elapsed_s`), och dess `status()` anger `connecting: True` med dess `phase`; de övriga kamerorna avläses i realtid när poolen är ledig, annars återges deras senaste avläsning (`live: False`).

#### `CameraSession`-metoder

| Metod | Beskrivning |
| --- | --- |
| `status(timeout=15.0)` | Kamerans live-status, såsom den avläses av `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` eller `stopped`), `device` (modell, serienummer, firmware, temperatur, `is_mono`) och dess aktuella inställningar. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Läser GenICam-noder; returnerar `{nodes, errors, enums, device, was_in_pool, in_array_id}`, samt `frame_values` och `notes` medan automatisk exponering pågår (se nedan). |
| `set_settings(**kwargs)` | Skriv till noder med användarvänligt namn (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). En `exposure_time` (`gain`) är en manuell exponering (förstärkning): skrivs utan `exposure_auto` (`gain_auto`) skrivs, måste autoläget först stängas av, så att värdet inte avvisas av ett pågående autoläge. Resultatet `applied` innehåller det värde som kameran tog emot; en skrivning som kameran avvisar finns i `errors` och sparas inte. Felmeddelandet anger det intervall som kameran nu accepterar (ett `ExposureTime`-maximivärde följer kamerans bildfrekvens), och ett automatiskt läge som stängdes av endast för det avvisade värdet återställs; `restored` listar detta (t.ex. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Ta en **enskild** bild. Returnerar en `CaptureResult`: en lista med bildmetadata-diktionärer (en per sparad nivå), med `.skipped`, `.failed`, `.daq_files` och `.daq_matches`. (Seriefotografering/flerbildsinspelning har tagits bort — anropa `capture()` i en loop om du behöver en serie.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Back-to-back-tagningar (`interval_s=0`) eller intervalltagningar från den här kameran, för `count`-inspelningar och/eller `duration_s` (minst en, så att den avslutas), precis som `ArraySession.capture_repeated()` gör för en array. Returnerar listan över `CaptureResult`:er; den avbryts efter en bildtagning som rapporterar att en enhet är full. `on_capture(i, result)` körs efter varje; övriga nyckelord går till `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Frigör från poolen, kameran återställs till det tillstånd den befann sig i vid anslutningen (utlösarkoppling, PTP, inställningar); `keep_settings=True` sparar istället sina inställningar i sitt eget minne, `clear_settings=True` återställer det minnet (se [Kameraminne](#camera-memory--keep_settings--clear_settings)); returnerar `camera_memory` när någon av dessa begärs. En kamera som ingår i en array frigör hela arrayen. Ingen åtgärd om vi anslutit till en redan öppen session. Returneras så snart backend bekräftar frigörandet, eller när det inte finns något att frigöra (kameran är redan borta, eller så körs ingen backend); en frigöring som har gått ut eller som backend har avvisat genererar `ChlorosConnectError`, eftersom kameran fortfarande kan vara upptagen och nästa anslutning skulle kollidera med den. Avslutningen av `with`-blocket genererar aldrig något fel. |

Attribut: `serial`, `device` (kamerans upptäcktspost) och `backend_url`.

`capture()`-exportkontroller (samma modell som arrayen + GUI):

- `processing` / `levels` — `processing="all"` sparar alla tillämpliga exporttyper; `levels=["raw","radiance"]` sparar endast dessa (åsidosätter `processing`). Utelämna båda för backend-standardinställningen.
- `daq` — den DAQ-avläsning som använts för inspelningen sparas som standard som en `.daq`-sidokar, inklusive en ren rådata-inspelning, så att ramen senare kan ombearbetas till reflektans/index. `daq=False` hoppar över den när ingen nivå använder den. `force_daq=True` är den äldre stavningen av standardvärdet. Ingen DAQ, inget `.daq`.
- `daq_tolerance_ms` — hur långt från exponeringen avläsningen får ligga och ändå ligga inom toleransen (standard 500). Endast bedömningen ändras.

#### `read_nodes()` vid automatisk exponering — `frame_values` och `notes`

Medan `ExposureAuto` (`GainAuto`) körs, läser noden `ExposureTime` (`Gain`) tillbaka **det senast skrivna värdet**, inte vad kameran gör just nu. `nodes` behåller fortfarande det värdet. När du begär `ExposureTime` eller `Gain` från en ansluten kamera vars automatiska läge inte är `Off`, läggs följande till i svaret:

- `frame_values` — det värde som kamerans senaste bildram innehöll, under nodens namn (`ExposureTime` i µs, `Gain` i dB);
- `notes` — en mening per sådan nod som anger båda värdena.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Båda nycklarna saknas när autoläget är `Off` (noden är då kameransverkliga värde) och för en kamera som inte är ansluten till backend. För en matris ger `ArraySession.status()["member_ae"]` alla elementens bildvärden på en gång.

#### DAQ-mätt exponeringsgräns (HTTP)

När en DAQ-ljussensor är ansluten begränsar Chloros kamerans exponering × förstärkning utifrån det uppmätta dagsljuset så att en vit referenspanel inte klipps. Gränsvärdet gäller endast så länge ett kalibreringsmål befinner sig i kamerans synfält (synligt i dess bildruta under de senaste 10 sekunderna); om inget mål syns används istället kamerans vanliga automatiska exponering, med dess eget skydd mot överexponering, så att resten av scenen inte hålls mörkare än nödvändigt. Det finns inget SDK-wrapper; läs av eller ändra inställningen per kamera med `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (POST-body `{"enabled": false}` stänger av funktionen för den kameran). Svaret innehåller `enabled`, `cap_us` (det aktuella takvärdet, `None` när funktionen är inaktiv, även när inget kalibreringsmål syns), `k_exp_g` (den konstant som takvärdet beräknas med) och:

| Fält | Betydelse |
| --- | --- |
| `k_exp_g_source` | Vems konstant som används: `camera` — den här kamerans egen, beräknad utifrån labbpunkten i dess kalibreringspaket (RGB-kameror); `fleet` — flottans konstant, används när paketet saknar labbpunkt; `override` — en fältöverskrivning inställd på backend. |
| `k_exp_g_fleet` | Flottans konstant, för jämförelse. |

### Synkroniserad matris — `ArraySession` (Smart-Prep)

`connect_array` är **den rekommenderade utgångspunkten** för installationer med flera kameror. Den kör hela Smart-Prep-flödet via det grafiska användargränssnittet i bakgrunden:

1. **Nätverksanalys**(`/api/camera/array/recommend`) — ställer in**hela sensorn** (ingen binning, ingen ROI-beskärning) och låter bildfrekvensen sjunka till vad denna värd och detta nätverk levererar, beräknat utifrån de pixelformat och packningsmetoder som Connect kommer att använda. Bildrutorna krymps aldrig för att hålla uppe frekvensen: ange en lägre `width` / `height` eller `binning=2` / `4` för att byta upplösning mot bildhastighet (ändpunktenss `alternatives` listar dessa alternativ med deras beräknade fps).
2. **Automatisk val av nivå** — `sim-capture-sim-emit` (alla kameror exponerar vid samma utlösare, sändningstakten anpassas för att dela kabeln, bildfrekvensen sjunker när delningen minskar) på vilket nätverk som helst där kamerorna kan anpassas för att dela; `slip-emit-and-capture` (kamerorna turas om på bandbredden, exponeras inte samtidigt) endast när detta inte är möjligt.
3. **PTP aktiverat**som standard — tidsstämplar mellan kamerorna hamnar på en gemensam klocka med en avvikelse på**~1 ms**, backendens [PTP-grandmaster](#ptp-grandmaster--time_sync_status). Samtidig exponering sker via M8-hårdvarutriggern (**&lt; 100 µs** mellan modulerna), inte via PTP: PTP synkroniserar *tidsstämplar*, inte exponeringar.
4. **Automatisk val av pixelformat per kamera** — svartvitt → `Mono12`, färg → `BayerRG12`.
5. **AE-initialisering** — tar en ögonblicksbild av varje kameras aktuella AE-tillstånd så att anslutningen inte återställer exponeringen mitt i en serie.
6. **GPIO-triggerkonfiguration** — `connect_array` aktiverar varje kamera (`TriggerMode=On`, `TriggerSource=Line2`) så att masterkamerapuls styr slavarna via M8-kabeln. Detta är ett steg som endast gäller för en array: en enskild kamera som öppnas med `LatticeCamera` körs istället fritt. När arrayen frigörs (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) återställer alla kameror till fritt löpande läge (`TriggerMode=Off`), med alla övriga inställningar som de var vid anslutningen: inställningarna för utlösare och synkroniseringslinje (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; huvudkamerans synkroniseringslinje tillbaka till en ingång), PTP och de kamerainställningar som ändrades under sessionen – såvida inte `keep_settings=True` / `clear_settings=True` (se [Kameraminne](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Attribut: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` och `packed_reason` (de fyra sista är `None` på ett handtag från `attach_array`), `connect_stream` och `backend_url`. `connect_stream` är anslutningssvarets `stream`-block, formen för den enskilda anslutningen plus `members` (`{serial: {state, why?}}`): `streaming` så snart den första synkroniserade gruppen i arrayen anlände (backend väntar på den, begränsat till cirka 10 sekunder), eller `no_frames` med varje medlems senaste hämtningsfel som dess `why` och, för jumbopaket, fallbacken på 1500 byte i `note`. `None` på ett `attach_array`-handtag.

#### `connect_array()`-signatur

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` anger hur länge anropet väntar på anslutningen, och det skickas till
backend som dess tidsgräns. En anslutning som inte har genomförts vid den tidpunkten avbryts: varje kamera som den
öppnat återställs till det tillstånd den befann sig i när den hittades och stängdes, ingen array är ansluten, och
anropet genererar `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, vilket anger vilket
steg anslutningen hade nått). Ett anrop som genererar ett fel lämnar alltså aldrig en matris
ansluten efter sig. En array med 9 kameror ansluts på ungefär en minut; direkt efter att
kamerorna startats kan det ta flera minuter. En array som är igång men som ännu inte har någon
synkroniserad bildgrupp returneras med `connect_stream["state"]`
`no_frames`. Backendens logg har en tidsrad per anslutningssteg
(`[array-connect <id>] <step>: <s> s …`) och en sista rad som anger det långsammaste.

`force_tier`-värden:
- `"sim-capture-sim-emit"` — verkligt simultant (alla kameror avfyras vid samma klockkant).
- `"sim-capture-ftd-stagger"` — flexibel förskjutning i tidsdomänen (kamerorna avger signaler vid något förskjutna tidpunkter så att paketen serialiseras på ledningen).
- `"slip-emit-and-capture"` — sekventiell avbildning per kamera (ingen tidsmässig synkronisering). Väljs automatiskt endast när kamerorna inte kan köras tillräckligt långsamt för att dela ledningen samtidigt.

`wire_ceiling_mbps` åsidosätter **värdens kontinuerliga ledningsbudget** i MB/s — det enda
talet som hela arraytilldelningen hänger på. Lämna det på `None` för att använda det automatiskt upptäckta
värdet. Sänk det när arrayen rapporterar GVSP-korrupta ramar: det automatiska värdet härleds
från nätverkskortets angivna länkhastighet, vilket överskattar USB-adaptrar, smala PCIe-banor och
upptagna delade nätverk — och överskattningen visar sig som korrupta ramar snarare än som en
synligt långsam länk. Värdet sparas i projektets array-inspelningsblock, så en
återöppning eller ett senare `connect_array` återställer det precis som vilken annan array-inställning som helst.
Se [Array Health](#array-health--which-subsystem-is-losing-frames).

#### Packad 12-bitars transport (`packed=`)

Vid full sensorbelastning bestäms en arrays bildfrekvens av hur snabbt varje kamera kan skicka
sina bildrutor. 12-bitarsformat (`BayerRG12`, `Mono12`) överförs normalt som 16-bitars
ord; packad överföring skickar samma 12 bitar i 1,5 byte per pixel och
packar upp dem på värddatorn, vilket gör att varje bildruta tar 25 % kortare tid att skicka — cirka
**33 % fler bildrutor per sekund vid full sensorkapacitet** (uppmätt: 10,7 → 14,0 fps på en
uppsättning med 9 kameror). **De sparade uppgifterna är identiska**: samma värden, samma 16-bitars
filer, samma metadata.

- `packed="auto"` (standard) packar endast när det är fördelaktigt **och** värddatorn har
  resurser att hantera uppackningen: vid anslutning mäter backend-modulen sin egen uppackningshastighet,
  antalet CPU-kärnor, aktuell belastning och nätverkskapacitet. En stationär dator med lediga kärnor
  komprimerar vid full sensorbelastning; en upptagen eller strömsnål värd (enkortdatorer,
  inbyggda styrenheter) strömmar okomprimerat. Små ROI:er och sammanslagna bildrutor ger liten vinst.
- `packed="off"` (eller `False`) komprimerar aldrig — för att utesluta det vid felsökning,
  eller på en värd som måste behålla sin CPU för annat arbete.
- `packed="on"` (eller `True`) packar alltid 12-bitarsformat.
- Alla andra värden genererar `ValueError`. 8-bitars- och 16-bitarsformat packas aldrig
  .

`analyze_array_network(..., packed="auto")` gör samma bedömning,
så dess fps är den hastighet som arrayen kommer att köras med (svaret innehåller
`packed_transport` och `packed_reason`); dess `pixel_format` är som standard inställt på vad
connect tillämpar (Mono12 / BayerRG12) snarare än 8-bitars.

Beslutet och dess motivering finns i sessionen:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` tillämpar samma överskrivning på varje
array i ett projekt (standard `None`, vilket innebär `"auto"`: ett projekt sparar inga
packade inställningar per array).
För en maskinomfattande standardinställning, ställ in `CHLOROS_ARRAY_PACKED12=off` (eller `on`) i
backendens miljö; en explicit inställning av `packed="on"` / `"off"` har fortfarande företräde.
`/api/camera/array/status` rapporterar `packed_transport` och `packed_decision`.

#### Förväntade bildhastigheter

Uppmätta bildhastigheter för matrisen (hårdvarusynkroniserade, 12-bitars; full sensor = 2048 × 1536). Använd dem för att kontrollera att allt stämmer med `status()["actual_fps"]`; `analyze_array_network()` beräknar bildfrekvensen för din egen värd och ditt nätverk.

| Konfiguration | Bildfrekvens för array |
| --- | --- |
| Full sensor, PCIe 10 GbE-värd-NIC, packad överföring (`packed="auto"` på en stationär dator) | ~14 fps |
| Full sensor, PCIe 10 GbE-värdnätverkskort, okomprimerad (`packed="off"`) | ~10,5 fps |
| Full sensor, 8 kameror på en USB 10 GbE-adapter | ~3,8 fps |
| Full sensor, 1 GbE-värdanslutning | 1,2–1,7 fps |
| 2 × 2 binning (`binning=2`), 10 GbE-värd-NIC | ~30–33 fps |

- På en PCIe 10 GbE-länk är det **kamerorna** som sätter gränsen vid full sensor, inte nätverket: 4 och 8 kameror körs med samma hastighet.
- **`target_fps` är utlösningshastigheten, och arrayen körs aldrig snabbare.** `target_fps=1` kör matrisen med 1 fps; `None` (standardinställningen) kör den med den högsta hastighet som värddatorn och nätverket klarar. En bild som överstiger den hastigheten sänks till den (se [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` av `4294` i `status()` är inte en länkhastighet.** Det innebär att Windows inte kunde avläsa adapterns hastighet (drivrutinen rapporterade det största 32-bitarsvärdet), vilket är typiskt för USB-Ethernet-adaptrar. En USB-adapters användbara hastighet i Windows är oavsett begränsad till 200 MB/s (se [Array Health](#array-health--which-subsystem-is-losing-frames)); för att finjustera en sådan, se avsnittet ”USB 10GbE-adaptrar” i CLI-referensen under [Host NIC Setup &amp; Tuning](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **En USB-begränsning gäller även i Linux: 400 MB/s på x86_64, 200 MB/s på NVIDIA Jetson.** Adaptern identifieras via sysfs (med `ethtool -i` som fallback), så en USB 5 GbE- eller 10 GbE-adapter räknas enligt gränsen istället för dess nominella hastighet på 500 / ~1060 MB/s. Siffran för x86_64 mättes på Linux: samma adapter som förstör ramar över 200 MB/s på Windows körde en uppsättning med fyra kameror felfritt vid alla takvärden upp till 480 MB/s. På NVIDIA Jetson fungerade samma adapter felfritt endast vid 200 MB/s och tappade ramar sporadiskt vid högre hastigheter. Om din värddator klarar högre hastigheter utan att `gvsp_corrupt`-ramar öka den med `wire_ceiling_mbps`. På Linux har ett nätverkskort i en USB 2.0-port en gräns på 30 MB/s; anslut det till en USB 3-port.

#### Översubskribering (minimigränsen per kamera)

En matris fungerar på vilket nätverk som helst; bildfrekvensen avgör. Sim-emit-pacing tilldelar varje kamera en jämn andel av den kollisionssäkra bandbreddsbudgeten och alla exponeras fortfarande vid samma utlösare, så på en långsam anslutning blir andelen — och bildfrekvensen — liten: 8 full-upplösning 12-bitarskameror på 1 GbE utan jumbo-ramar får 6,5 MB/s vardera, cirka **1 fps**(cirka 1,3 fps packat). Andelen har en lägsta gräns på den långsammaste hastighet som kamerorna kan anpassas till (`per_cam_floor_bps`: läses av från varje kameras GevSCPD-intervall vid anslutning, och aldrig under 1 MB/s). Om nätverket kräver att de går långsammare än så faller anslutningen tillbaka till**seriell**sändning (`slip-emit-and-capture`): kamerorna turas om med full länkhastighet, så varje ram anländer men medlemmarna**exponeras inte samtidigt**— svarets nivå och blockerare (`SERIAL: ...`) anger detta; skicka `force_tier="sim-capture-sim-emit"` för att avvisa fallbacken. Bildstorlek påverkar inte delningen, så**binning och ROI höjer bildfrekvensen men påverkar inte om en matris får plats**. Endast en länk som inte kan hantera matrisen vid någon överföringstakt, vare sig samtidigt eller turvis,**överbelastas** — felmoden är GVSP-paketförlust, inte en lägre bildfrekvens. Åtgärder: färre kameror, jumbo-ramar från ändpunkt till ändpunkt eller ett snabbare nätverkskort.

- Svaren `analyze_array_network()` och `/api/camera/array/connect` innehåller `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` och `per_cam_floor_bps`. När `oversubscribed` är sant nollställer projektionen **fps-fälten** (`achievable_fps_max` / `fps_bright` / `fps_dark`) istället för att rapportera en missvisande hastighet som är långsam men fungerar.
- `POST /api/camera/array/connect` accepterar en `pin_resolution`-body-parameter (**endast HTTP — inte en SDK-kwarg**; `connect_array` exponerar den inte). Det är strikt läge: en överbelastad anslutning med `pin_resolution` inställt**avvisas kategoriskt** med ett felmeddelande som anger alla möjliga åtgärder. Utan detta fortsätter anslutningen med en varning om att arrayen kommer att förlora paket. Oavsett vilket behåller anslutningen den begärda ramstorleken — den grupperar eller beskär aldrig på egen hand.
- Endast för testning: ställ in `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` i backendens miljö för att nedgradera avvisningen till en tydlig varning – du ansluter ändå och accepterar paketförlusten.

#### Arrayhälsa – vilket delsystem förlorar ramar

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) bär ett aktivt `health`-block på en
ansluten array, omvärderat i ett rullande **10-sekunders** fönster. Den delar upp ramförlusten
i de två orsakerna som kräver motsatta åtgärder, istället för en enda ”ofullständig” frekvens som
inte specificerar någon av dem:

| Fält | Vad det betyder | Vilket delsystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per serienummer) | Bildrutan **anlände men var strukturellt felaktig**— GVSP-paketförlust. |**Nätverk**: kabelkapacitet, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (per serienummer) | Ramen **kom aldrig alls**— kameran utlöste inte, eller så skickades inget ut. |**Utlösare / synkronisering**: M8-kabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Sämsta kamerahastighet för var och en. | — |
| `per_cam_rate_pct` | Kombinerad ofullständighetsfrekvens per kamera (båda orsakerna tillsammans). | — |
| `stable_for_seconds` | Hur länge varje kamera har legat under 0,01 %. | — |

Tillsammans med `health` visar samma post hur mycket av den totala tilldelningen som är ledig:

| Fält | Vad det betyder |
| --- | --- |
| `wire_ceiling_mbps` | Värdens gällande kontinuerliga bandbreddsbudget, MB/s. |
| `wire_ceiling_source` | Varifrån det värdet kommer, i ord — t.ex. `USB-capped 200 MB/s (was theoretical 1062; …)` eller `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` när `wire_ceiling_mbps=` ställde in den. |
| `nic_is_usb` | `true` för en USB-Ethernet-adapter. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Tolkning:** ett värde på `gvsp_corrupt_rate_pct` som inte är noll och `never_arrived_rate_pct` på 0 betyder att
triggern och kabelsynkroniseringen är perfekta och att 100 % av förlusten ligger på nätverksvägen — sänk
`wire_ceiling_mbps` och anslut på nytt. Det omvända mönstret pekar istället på synkroniseringskabeln eller
triggerledningen.

> **`target_fps` är inte avgörande för korrupta ramar.** GevSCPD-takten skrivs in en gång vid
> anslutning, så att sänka triggerfrekvensen ändrar arbetscykeln men inte
> burstfrekvensen för samtidig sändning. En uppmätt 5×-begränsning gav ingen förbättring, medan
> en sänkning av trådens tak från 240 till 200 MB/s tog samma rigg från 10,4 % korrupta till
> 0,00 %.

> **En igångsatt array kan inte omplanera sig själv på LATTICE (M3C / M3M)-firmware.** Den kan inte
> åtgärda detta själv; koppla bort och anslut på nytt så att anslutningstidsväljaren omplanerar utifrån
> det nya taket.

En **USB-Ethernet-adapter begränsas till 200 MB/s** av sonden oavsett dess
typbeteckning: effektivitetstabellen som omvandlar en länkhastighet till ett kontinuerligt värde är
hämtad från PCIe, och ett USB-nätverkskort anger sin Ethernet-länkhastighet samtidigt som det begränsas av
USB-bussen och dess drivrutin. Begränsningen är absolut, inte en bråkdel — en USB 1 GbE-adapter
uppnår ~80 MB/s och påverkas inte. Detta gäller för Windows och Linux (inklusive Jetson);
se [Förväntade bildhastigheter](#expected-frame-rates) för var det mättes.

#### `ArraySession`-metoder

| Metod | Beskrivning |
| --- | --- |
| `status(timeout=10.0)` | Aktuellt tillstånd för den körande matrisen: `target_fps` / `target_fps_auto` (triggerfrekvensen och om frekvensregulatorn styr den), `target_fps_clamped_reason` (anledningen till att en fastställd frekvens hålls lägre, annars `None`), `actual_fps` (uppmätt; 0 när grab-slingan har gett upp eller fastnat), `recommended_fps`, `streaming` (False när hämtningsslingan har gett upp — kontrollera detta, inte bara `last_error`; förblir Sant vid en omstart av inställningarna, vilket `restarting` markerar), `stalled` (Sant medan hämtningsslingan körs men inte har publicerat någon bildgrupp under cirka tio bildperioder, 2–30 s; `last_error` anger detta först efter 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` endast när arrayen var ansluten till PTP **och** varje medlem senast läste `PtpEnable` på; inte detsamma som låst, vilket `ptp_epoch` mäter), `ptp_configured` (vad anslutningen begärde), `ptp_off_members` / `ptp_unread_members` (enheter som senast läste `PtpEnable` av / inte svarade), `member_recovery` (`None`, eller en medlem som förlorat ström eller sin länk: `state` `waiting` medan den inte svarar, `reopening`, sedan `recovered`, tillsammans med `members` och `why` — matrisen återansluter den när den svarar igen och återställer dess PTP-värden för anslutningstid, trigger- och synkroniseringsledningskoppling, taktsättning och kamerainställningar, och strömmar sedan igen utan nya `connect_array`) och `ptp_epoch` (uppmätt), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` betyder att Windows inte kunde läsa adapterns hastighet — se [Förväntade bildhastigheter](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (om strålningskedjan per band är användbar och vilken del som orsakade avbrottet) samt `band_clamp` — medan en livevy återger matrisen: andelen upplysta pixlar som avblandningen begränsat till 0, per komponent (`members`: `{serial: [band0, band1, band2]}` i kamerans R-, G-, B-bandordning) och per band som kompositen visar (`composite`: `{band name: share}`), med `warn`, den andel som GUI:n markerar som ”Banddata med låg tillförlitlighet” (mäts ungefär en gång per sekund; `None` när ingen livevisning är igång), och `member_ae` — per medlem, exponeringen (`exposure_us`) och förstärkningen (`gain_db`) som dess senast publicerade bildruta innehöll, hämtade från själva bildrutan (`read_nodes` av `ExposureTime` / `Gain` returnerar det senaste värdet som skrevs medan automatisk exponering körs, inte vad den gör just nu). |
| `set_member_settings(serial, **kwargs)` | Skriv in inställningarna för en medlemskamera, eftersom `CameraSession.set_settings()` hämtar dem (exponering, förstärkning, måljashet, ...); medlemmen förblir i arrayen. Utlöser `ChlorosConnectError` för en serie som inte är en medlem. Se [Exponering på arraymedlemmar](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Vad smart-prep fastställde (inspelningsnivån, bildstorlek, `pixel_format`), det aktiva `health`-blocket och värdens bandbreddsbudget. Se [Arrayens hälsa](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Ändra triggfrekvensen medan arrayen körs, utan att återansluta. Ett tal låser den (frekvensregulatorn lämnar den ifred); `None` återställer den till auto. Returnerar `{requested, target_fps, target_fps_auto, clamped}` — `clamped` är True när hastigheten sänktes till vad värden och nätverket klarar av. Se [Intervallfotografering för drönarundersökningar](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | En bildruta från varje kamera, **alla från en utlösare**, som numpy-matriser (inga filer skrivs). Returnerar en `GrabResult` (en `dict`): `{serial: array}`, eller `{serial: (array, metadata)}` med `with_metadata=True`. `.group_id` är den utlösare som bildrutorna delar; `.errors` kopplar varje utelämnad kamera till orsaken (en kamera som inte kan betjänas vid den utlösaren utelämnas och fylls aldrig i från en annan). `processing` är `raw` / `debayered` / `radiance` / `reflectance` / `display`. Samma läsning som projekt-API:s [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — en gemensam implementering — så alla nyckelord där (`band`, `align`, `bit_depth`, `refl_max`) fungerar här. Kräver `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterator som ger ett `grab()`-resultat per tick (endast nyckelordsargument), anpassad till `fps`, begränsad av `duration` (sekunder) och/eller `count` (`None` = obegränsad). **Varje returnerad uppsättning är en ny utlösare:** en tick vars senaste utlösare är den uppsättning den redan genererat väntar på nästa utlösare istället för att upprepa den, så strömmen körs med den lägre av `fps` och arrayens egen bildfrekvens. Genererar `FrameFetchError` istället för att tystna: omedelbart när alla kameror av någon anledning vägrar att upprepa nästa tick, efter 60 sekunder av tickar utan bildruta och efter 60 sekunder utan ny utlösare (”ingen ny utlösare på 60 sekunder”). Samma implementering som `ArrayHandle.frame_stream()`. Se [Livebilder i din egen pipeline](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Starta backendens egen kontinuerliga inspelning (GUI:ns kontinuerliga läge) → [`ContinuousHandle`](#continuoushandle). Sparar **varje** synkroniserad grupp som arrayen publicerar, på samma sätt som `capture()` skulle spara den (samma exportkontroller), tills den stoppas eller tills `max_captures` / `duration_s` (`None` = tills det stoppas). Ger upphov till `ChlorosConnectError` om ett redan körs för denna matris. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | En synkroniserad inspelningsgrupp. Returnerar en `CaptureResult` (lista över ramordböcker + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Skicka ett tidigare resultat`group_seq` som `after_seq` för att få nästa synkroniserade grupp efter den istället för att vänta på en ny. Exportkontroller nedan. |
| `capture(..., smart=True)` | **Smart inspelning** — väntar tills AE har stabiliserats på alla kameror, och utlöser sedan. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Snabbaste inspelning: endast rådata + det tilldelade DAQ-värdet (+ det fria kombinerade indexet). Speglar GUI-knappen ”Fastest Capture”. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Enstaka / Kontinuerlig / Intervall i en avgränsad slinga. Returnerar `list[CaptureResult]`.**Kräver `count` och/eller `duration_s`** för att avslutas (SDK:t har inget Ctrl+C). Avslutas även i förtid, efter omgången, när ett pass rapporterar en fullständig enhet (en `.failed`-post med `disk_full`); det passet är det sista resultatet. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Börja spela in den kombinerade indexvyn i realtid till video/GIF → `RecorderHandle`. Videon är en Motion-JPEG-fil `.avi` (inget ffmpeg behövs). Om ingen GUI-vy är öppen renderar backend-modulen den: `expr` (ett indexuttryck i matrisens bandnamn, t.ex. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`), eller matrisens NDVI som standard. En sammansatt inspelare per matris. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Starta en rå-Bayer-bildserie med hög bildfrekvens → `RecorderHandle`. Bearbeta om offline med `build_video()`. `orient=True`: de produkter som `build_video()` skapar roteras enligt kamerans orientering (produkter per kamera enligt respektive kameras orientering, det kombinerade indexet enligt huvudkamerans); råbilderna förblir i sitt ursprungliga format och seriemanifestet registrerar varje medlems orientering. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Bearbeta en sparad råbildserie offline till kalibrerade videoklipp. Blockerar tills processen är klar (`wait=True`) och returnerar byggresultatet: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — se [byggresultatet](#the-build_video-result) nedan. |
| `build_video_status(job_id, timeout=15.0)` | Avfråga ett offline-byggjobb: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Frigör hela arrayen — **om den här sessionen öppnade den** (`connect_array`): då blir det `force_disconnect()`, och genererar ett undantag `ChlorosConnectError` när frigörandet inte bekräftas (utgången från blocket `with` utlöser aldrig något). Vid ett handtag från `attach_array` låter den matrisen fortsätta köras och skriver ut en rad till stderr som anger detta: matrisen kan tillhöra GUI:n eller ett annat skript. Varje kamera lämnas i fritt läge och i övrigt i det tillstånd som anslutningen fann den; `keep_settings=True` sparar dess inställningar i kamerans eget minne, `clear_settings=True` återställer det minnet till fabriksinställningarna (se [Kameraminne](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Frigör hela arrayen oavsett vem som öppnade den. Använd det på ett anslutet handtag när du avser att avsluta arrayen (t.ex. för att återansluta den med andra inställningar). Återgår så snart backend bekräftar frigörandet, eller när det inte finns något att frigöra (matrisen är redan borta, eller ingen backend körs); en nedmontering som har gått ut eller som backend har avvisat genererar `ChlorosConnectError`. Med vilken flagga som helst returnerar den vad varje kameras minne fick (`camera_memory`), annars `None`. |

`capture()` exportkontroller (samma slutpunkt som GUI/CLI använder):

- `processing` / `levels` — `processing="all"` (eller `levels=["raw","radiance",…]`) sparar varje tillämplig exporttyp per kamera; ett enda `processing`-värde sparar endast den nivån.
- `aligned` — **aktiverat som standard (`None`), eftersom Chloros-appens inspelning av samma array är**(dess**justerade** inspelningsinställning per matris). Varje medlems icke-råa export filer anpassas till huvudkamerans bildram enligt matrisens [justeringsprofil](#array-alignment) och beskärs till medlemmarnas gemensamma överlappning, så att filerna är pixelregistrerade och får samma storlek, beskärning och `Alignment*`-taggar som appens (inom ett ROI ombaseras transformationen till fönstret); `XMP-mapir:AlignmentJSON` anger `"applied": true` med dess `crop_roi`. Raw-filerna förblir oreviderade i sin ursprungliga storlek men bär med sig transformationen i metadata. `aligned=False` sparar varje fil i sin ursprungliga storlek, oreviderad; `aligned=True` är standardinställningen. `continuous()`, `capture(smart=True)`, `capture_repeated()` och projektnivåinställningarna `ArrayHandle.capture()` / `capture_smart()` har samma standardinställning. Oavsett om förvrängningen tillämpas eller inte registrerar varje medlemsfil sin kameras `Alignment*`-transformation så snart matrisen har en inriktningsprofil — med undantag för en kamera som profilen inte inriktar (utlämnad eller tvingad till identitet på grund av en misslyckad lösning), samt referenskameran när ingen annan kamera är justerad: dessa har inga `Alignment*`-taggar. Om det inte finns någon profil för matrisen sparas inspelningen ojusterad, och resultatet visar detta: `result.alignment` är `{"requested": True, "applied": False, "warnings": [...]}`, en varning per anledning till att förvrängningen inte kunde tillämpas (ingen profil, medlemmar med olika upplösningar, en matris som inte kansynkroniserad, en bildruta som profilen inte kan mappas på). `applied` är `True` endast när varje export som begärde förvrängningen fick den. Matrisens automatiska inriktning skapar profilen så snart exponeringen har stabiliserats; `ArrayHandle.calibrate_alignment()` / CLI:s `align-calibrate` skapar en på begäran. `result.oriented` anger om `orient=True` har roterat produkterna.
- `render_index=False` — hoppa över överlägget med vegetationsindex per kamera; standardinställningen renderar det där det är konfigurerat.
- `daq` / `daq_tolerance_ms` — DAQ-avläsningen sparas som en `.daq`-sidecar som standard, även när ingen vald nivå behöver den; `daq=False` hoppar då över den, och `daq_tolerance_ms` ställer in matchningstoleransen (standard 500). `force_daq=True` är den äldre stavningen av standardvärdet. Se [DAQ-avläsningen som en inspelning sparar](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — aktiverar varje medlems **produkter**(`display`, `radiance`, `reflectance`, indexet per kamera och den sammansatta bilden) med kamerans orientering, efter justeringsförvrängningen: med**huvudkamerans**orientering när de försköts till dess bildram (standardinställningen `aligned` — medlemmarna förblir samregistrerade), annars med sina egna kamerors orientering.**Standard: varje medlemsfil i ursprungligt format**, vilket alltid är fallet för array-inspelningar; `raw` och `debayered` förblir i ursprungligt format oavsett. Varje fil registrerar orienteringen för den bildram som dess pixlar ingår i (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — en råfil, den egna kamerans), och XMP-filen `Alignment*` behåller den ursprungliga bildramstransformationen, tillämpad före orienteringen; en RAW-fil som innehåller den transformeringen registrerar även `ChlorosAligned*` (huvudramens), orienteringen för en produkt som har förvrängts in i huvudramen. Bearbetningen vrider varje produkt med den orientering som tillhör dess pixlar: den justerade när den förvränger RAW-filen (Array alignment på), kamerans egen när den inte gör det. `capture_repeated()` och `capture_fastest()` låter den passera; `ArrayHandle.capture()` / `capture_smart()` på projektnivå tar också med den.

**TIFF-komprimering (inställning endast för HTTP):** `ArraySession.capture()` skickar ingen `compression`-nyckel, så backendens standardinställning gäller — `POST /api/camera/array/capture` läser en `compression`-kroppsparameter, `"none"` som standard: okomprimerad, sensorns pixlar exakt som de läses in (~6,3 MB per bildruta i full upplösning). `"deflate"` (förlustfri zlib L1 + horisontell prediktor, ~4,1 MB/bild) skriver mindre filer långsammare — båda är förlustfria och läses identiskt vid import. SDK:et tillhandahåller inga kwarg för detta; en alternativ lösning är `chloros-cli lattice array-capture --compression deflate` eller rå HTTP. DEFLATE låser Pythons GIL, så komprimerade skrivningar kan inte parallelliseras över skrivtrådarna per kamera — kontinuerlig inspelning i full upplösning med sensorns hastighet kräver standardinställningen (okomprimerat). Detaljer: [CLI-referens → array-capture](cli-reference.md).**På varandra följande grupper (HTTP):** slutpunkten returnerar `group_seq`; en loop som skickar tillbaka den som `after_seq` vid nästa begäran hämtar nästa synkroniserade grupp – inklusive en som publicerades medan den föregående inspelningen sparades — istället för att vänta på en grupp som publicerats efter förfrågan, och `skipped_groups` rapporterar alla grupper som slingan missat. `capture_repeated`, CLI:s `--continuous` och GUI:s kontinuerliga läge gör alla detta; intervallpasseringar förblir aktuella vid varje schemalagt tillfälle.**Överskrivningar vid export per medlem (endast HTTP):**samma slutpunkt accepterar även `exclude_serials` (lista – ta bort medlemmar från den sparade uppsättningen; matrisen utlöses fortfarande som en synkroniserad grupp och uteslutna medlemmar returneras i `excluded`), `serial_levels` (`{serial: [level tokens]}` överskrivningar på kameranivå) och `serial_index` (`{serial: bool}` överskrivningar av indexöverlägg per kamera). Dessa är GUI-paritetsparametrar och**ännu inte SDK-kwargs**; medlemmar som saknas i kartorna faller tillbaka till de arrayomfattande `levels` / `render_index`.

##### Granska hoppade kameror — `CaptureResult.skipped`

`ArraySession.capture()` returnerar en `CaptureResult`, som är en underklass till `list`: iterera den, indexera den, `len()` — alla befintliga mönster fortsätter att fungera. Ny kod kan inspektera attributet `.skipped` för att se vilka kameror som uteslöts och varför. Det vanligaste fallet är RGB-kameror i en matris med blandade filter när du begär `processing="radiance"` eller `"reflectance"` — strålningsvärdet per Bayer-pixel är meningslöst för en bredbandssensor, så backend-modulen hoppar över dessa kameror istället för att generera meningslösa värden.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Orsaktoken följer mönstret `<level>-not-applicable-to-rgb-cam` (en post per hoppad nivå, var och en med `level`). En kamera vars kalibreringspaket inte är laddat hoppar över `radiance` / `reflectance` med `<level>-needs-calibration (connect the camera so its cal pack loads)`, och om det inte finns någon användbar DAQ-avläsning av reflektans hoppas den över med `reflectance-skipped-no-dls`. De övriga reflektansspecifika hoppen är `reflectance-skipped-no-fresh-dls` (ingen ny nedåtriktad avläsning tillgänglig), `reflectance-skipped-bound-daq-unavailable (…)` (den kopplade DAQ-enheten kunde inte nås) och `dls-uncalibrated-band-<nm>` — bandet ligger till största delen utanför DAQ-ljussensorns radiometriskt kalibrerade område (~374–974 nm), så den absoluta DAQ-baserade reflektansuppdelningen avvisas och ramen nedgraderas tydligt till sensorns respons. Bland de levererade SKU:erna är det endast F988 som utlöser detta; den kamerans stödda väg är arbetsflödet med reflektanspanelen.

##### Filer som inte kunde skrivas — `CaptureResult.failed`

En inspelning som endast kunde skriva vissa av sina filer returnerar fortfarande: listan innehåller alla filer som **skrevs**, och `.failed` listar resten, ett uppslagsverk per fil med `serial` (`None` för den kombinerade kompositfilen och en matris `.daq`-sidokar), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, eller `match` — `.daq`:s `.match.json`-härkomst, när `.daq` skrevs men dess sidecar inte gjorde det) och `error`. På en full disk innehåller posten även `disk_full: True`; varje senare inspelning till den disken skulle misslyckas på samma sätt, vilket är anledningen till att `capture_repeated()` slutar där. Inget undantag genereras för en partiell inspelning, och en fil i `.failed` lämnades inte kvar på disken halvskriven. `.failed` är tom vid en ren inspelning.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### Den DAQ som läser av en inspelning sparar — `.daq_files` och `.daq_matches`

När en DAQ är kopplad till kameran eller matrisen, eller ingår i en pool, sparar varje inspelning den nedåtriktade avläsning som den använde: en `.daq` i undermappen `daq/` i bildmappen, tillsammans med dess `.match.json` och dess `.cameras.json` (vars mätvärde det är). Detta gäller även för råa eller debayrade bilder, och det är samma fil oavsett om bilden togs via GUI, CLI eller SDK. `daq=False` hoppar över detta för en bild vars nivåer inte använder DAQ (en `radiance`-, `reflectance`- eller `display`-nivå skriver fortfarande ut den); `daq_tolerance_ms` flyttar toleransgränsen för den inspelningen och aldrig den använda avläsningen. Vilken DAQ: den som är kopplad till kameran eller matrisen (GUI:ns val av ljussensor, en projekts koppling eller `chloros-cli daq pool-bind`), annars den senaste som backend har. CLI-gränssnittet anger kontrollerna `--no-daq` och `--daq-tolerance-ms` ([CLI-referens](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

En inspelning sparar det ljussensorvärde som användes som en `.daq` tillsammans med en `<file>.daq.match.json` bredvid, som anger hur värdet kopplades till exponeringen: `method`, `gap_ms` (från exponeringen till närmaste DAQ-avläsning) och `in_tolerance` (inom 500 ms, eller bildtagningens `daq_tolerance_ms`). Reflektansfilens XMP-data innehåller samma bedömning som `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` och `Chloros:DlsInTolerance`.

En bild väntar på DAQ:ns första avläsning efter exponeringen, så att den kan blanda avläsningarna på båda sidor om den (`method` `interpolated`). En DAQ-E skickar ett mätvärde var 0,4 s i dagsljus och var 1–2 s inomhus, så `capture()` returneras upp till ungefär ett och ett halvt av dessa intervall senare; själva exponeringen fördröjs inte. Man väntar inte på en DAQ som har slutat sända. Ett DAQ-E-mätvärde är ljuset genomsnittligt över intervallet sedan föregående mätning, så det placeras i mitten av det intervallet: `gap_ms` mäts till den mitten, och `.match.json`&#x27;`bracket_stamps_ns` ger avläsningarnas egna tidsstämplar. `ArraySession.continuous()` väntar utan att bromsa insamlingen. `capture_repeated()`:s-till-bak-pass väntar inte, så de behåller sin hastighet och matchas med det närmaste mätvärdet före exponeringen.

Ett mätvärde förfaller aldrig, så ingen bildram går förlorad på grund av ett försenat mätvärde — det märks istället:

- **DAQ:ns ström har fastnat** — dess senaste avläsning används, och `gap_ms` anger hur gammal den är (`in_tolerance: false` för mer än 500 ms sedan).
- **Ingen DAQ att avläsa**— frånkopplad (`disconnect()`, `chloros-cli daq pool-disconnect`), eller så går det inte att nå den kopplade DAQ:en — kamerans senaste avläsning återanvänds och beskrivs mot**denna** exponering: `method`, `retained` och `gap_ms` från denna exponering, `in_tolerance` beräknas om, och `retained_from_epoch_ns` namnger den exponering den först matchades med. En kamera som aldrig har haft någon avläsning får fortfarande ingen reflektans, och resultatet anger varför (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, eller att bildtagningen avvisades).

Filer som en bildtagning sparar i mappen för det projekt som är öppet i Chloros läggs till i projektets fillista, precis som en bildtagning i appen gör (filer som sparats någon annanstans lämnas orörda): `CameraSession.capture()`, `ArraySession.capture()` (även `smart=True`) och `capture_cameras_transient()`. Resultatet framgår av `.project_add` (`capture_cameras_transient()`: ordlistan i `project_add`) -- `held`, `files` och backendens `message`; `held` är `True` medan en bearbetningskörning pågår, och filerna läggs till när den avslutas. `None` när ingen fil har lagts till i projektet.

`CameraSession.capture()` och `ArraySession.capture()` returnerar dem som resultatets `.daq_files` och `.daq_matches`, då HTTP-fångstsvaren (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) innehåller `daq_files` och `daq_matches`, ett per `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` och `stream_fault` — DAQ:s eget strömningsfel, som endast kontrolleras vid avläsningar utanför toleransgränserna. Innan du förlitar dig på reflektansen från en mätning, kontrollera `in_tolerance` i dess `.match.json` eller XMP, eller DAQ:s [`status()`](#daqsensorsession-methods).

`processing`-nivåer:

| Nivå | Utgång |
| --- | --- |
| `"raw"` | Enkanalig Bayer (monokameror: det enda bandet) direkt från sensorn. |
| `"debayered"` *(standard i SDK)* | 3-kanals färg: kalibreringspaketets mörka offset (DSNU) avläst från rådata, följt av OpenCV:s kantmedvetna demosaik — ingen linearitets-LUT, flat field eller unmix (monokromkameror: 1-kanals gråskala). Den sparade TIFF-filen har **RGB**-kanalordning (fotometrisk RGB, så att alla bildvisare visar sanna färger); de minnesmatriser som `grab()` / `frame_stream()` returnerar har OpenCV:s**BGR** — vänd på kanalerna (`frame[..., ::-1]`) för att jämföra med en sparad fil. `chloros-cli process` skriver samma resultat som `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm via den fullständiga radiometriska kedjan. Endast multispektralt — RGB-kameror hoppas över. |
| `"reflectance"` | uint16 0..32768 (Pix4D-kompatibel); kräver en aktiv DAQ-koppling för absolut referens. Endast multispektral. |
| `"display"` | Fullständig kedja som matchar förhandsvisningen i GUI (CCM + WB + gamma enligt kamerans profil). |
| `"all"` | **En fil per tillämplig nivå** för varje kamera (motsvarar GUI-inställningen ”Capture All” / CLI-standard). Den returnerade `CaptureResult` innehåller då ett bilddikt per `(cam, level)`, med nivån i varje dikt; nivåer som inte är tillämpliga visas i `.skipped`. Den DAQ som läste av den använda inspelningen sparas som en `.daq`-sidokontakt. |

> **Obs! — standardinställningen skiljer sig från den i CLI.** `ArraySession.capture()` har som standardvärde `processing="debayered"`; kommandot `chloros-cli lattice array-capture` har som standardvärde `processing="all"`. Ange `processing="all"` explicit från SDK för att spegla CLI:s/GUI:s flernivåsparning.

##### Filnamn för arrayinspelningar

Varje nivå sparas i en egen undermapp till `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` för `display`, `index/`, `composite/`). Varje fil namnges efter sin kamera enligt mönstret `SN<serial>-<filter>`: serienumret följt av filterdelen av kamerans modell — `SN214000533-FOCN` för en `M3C-L87-FOCN`, `SN261902142-F450` för en `M3M-L87-F450` — sammanfogat med `-`, så namnet har samma `_`-separerade fält som tidigare. En kamera vars modell inte kan läsas är enbart `SN<serial>`, och bilder från tidigare versioner har inget filter i namnet. Filnamnet beror på hur många nivåer en medlem ombads ange:

| Begärd medlemsnivå | Filnamn | Exempel (`reflectance`, standard `format="tiff"`) |
| --- | --- | --- |
| **En nivå** (`processing="reflectance"`, eller en `levels` med ett objekt) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — inget nivåsuffix, precis som i tidigare versioner, så befintliga skript fortsätter att fungera | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Flera nivåer** (`processing="all"`, eller `levels` med fler än en post) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[Klämmasken](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) följer efter produktnamnet, `clamp_masks/<product folder>/<product name>_clampmask.tif`: enreflektansavläsning på en nivå skriver `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, medan `processing="all"` skriver `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. Suffixregeln bestäms per medlem utifrån de nivåer som begärdes, så en utelämnad nivå (t.ex. reflektans på en RGB-kamera) påverkar inte den. Läs in sökvägarna från de returnerade bildrutorna istället för att bygga dem. En inspelning med en enda kamera (`CameraSession.capture()`) lägger alltid till nivån: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### PNG/JPEG: endast bilder och indexprodukter

En PNG- eller JPEG-fil rymmer 8 bitar per kanal. Om man begär en fil med en nivå som den inte kan rymma avvisas begäran innan något fångas in, med ett meddelande som anger nivån och TIFF som det format som rymmer den:

- **`radiance`**(float32) och**`reflectance`**(uint16, `32768` = 1,0) — och därmed även `processing="all"`, som inkluderar dem. SDK:t kontrollerar detta på argumenten: `capture()` genererar**`ValueError`** utan en begäran (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` från en kamera som strömmar mer än 8 bitar** (12-bitars M3C, Mono12 M3M) — backend-systemet avvisar dessa, eftersom det känner till varje kameras bitdjup: HTTP-kod 400 med `error_code` `FORMAT_CANNOT_HOLD_LEVEL`, genererat som `ChlorosConnectError` (`CameraSession` / `ArraySession`) eller `RuntimeError` (`CameraHandle` / `ArrayHandle`). En en-nivå-inspelning med `debayered` från en kamera tar en 8-bitars bild för PNG/JPEG och avvisas inte.

Spara radiometriska data som `format="tiff"` (`ext=".tiff"`); använd PNG/JPEG för `display` och indexrenderingar.

#### Hur automatisk exponering fungerar

- **Exponering först, förstärkning sist.** När förstärkningen är inställd på Auto förblir den på 0 dB tills exponeringen har nått det högsta tillåtna värdet.
- **En manuell exponering lämnar förstärkningen på Auto** tills du ändrar förstärkningen eller ställer in den på manuell. När du återgår till Auto är förstärkningens gräns densamma som tidigare.
- **Små högdagrar gör inte scenen mörkare.** En spegling på upp till 1 % av bilden ignoreras, och i panelerna på ett känt kalibreringsmål ignoreras upp till 0,5 % av en panel.
- **DAQ-exponeringens övre gräns gäller endast så länge ett kalibreringsmål är i bild.** Det förhindrar att en vit panel klipps; när ingen panel syns exponerar kameran för scenen.
- **Konstant ljus förblir oförändrat, och förändrat ljus stabiliseras inom några sekunder.** Vid konstant ljus förblir exponeringen och dess gräns oförändrade, och efter en stor ljusförändring stabiliseras bilden inom några sekunder. En överexponering begränsar inte längre exponeringen permanent, och vid en förändring av ROI återfinns exponeringen igen.
- **Ljusstyrkemålet hålls inom räckhåll.** Det stiger inte medan kameran befinner sig vid sin exponeringsgräns och dess förstärkning inte kan öka, så att ett ljus som återkommer inte orsakar bländning i bilden.
- **En kamera vars automatiska exponering slutar svara hålls kvar på den nivå som motsvarar det rådande ljuset.** I starkt solsken kan en kamera sjunka till sin kortaste exponeringstid och stanna där med en mörk bildruta, samtidigt som alla inställningar visar normala värden. Chloros bevakar bildrutan, håller kameran kvar på den nivå den hade för det ljuset och släpper den när ljuset förändras. En mörk eller täckt ljussensor påverkar inte detta.

#### Exponering på arraymedlemmar

En LATTICE-kamera begränsar `ExposureTime` till en period av dess bildfrekvens. Medan en array strömmar kör Chloros varje medlem med `AcquisitionFrameRateEnable` aktiverat, med en hastighet strax över utlösningshastigheten, så att en medlems exponeringsgräns ligger något under utlösningsperioden, **vid vilken ROI som helst**:

- En enhets bildperiod är utlösningsperioden minus en marginal på 15 % av perioden (minst 3 ms, högst halva perioden), och dess exponeringsgräns är den bildperioden minus cirka 0,1 ms. Exempel: en utlösare på 10 fps (100 ms) ger en medlemshastighet på cirka 11,8 fps och ett exponeringstak på cirka 85 ms.
- **En manuell `ExposureTime` som är längre än det taket saktar ner utlösaren för att anpassa den.** Chloros öppnar först medlemmens tak för det värdet och låter den långsammare hastigheten köras under några utlösningsperioder (skrivningen kan ta några sekunder längre), skriver sedan in det, och utlösaren avfyras inte snabbare än vad exponeringen tillåter. I en matris med automatisk hastighet återställs den tidigare hastigheten så snart varje medlem vars manuella exponering saktade ner den återgår till automatisk exponering. Ett värde som kameran själv inte kan ta emot avvisas fortfarande, med det intervall den nu accepterar angivet i felmeddelandet, och medlemmens automatiska exponering återställs om den stängdes av endast för den skrivningen. Före version 1.3.0 begränsade ett litet ROI varje medlem till sensorns egen frilöpningsperiod, oavsett utlösningsfrekvens: 4,9 ms vid 768 × 304, så ingen medlem kunde göras ljusare.
- Chloros förkortar aldrig en manuell exponering för att anpassas till en snabbare utlösningsfrekvens. En automatisk frekvens håller sig inom vad de manuella exponeringarna tillåter; en fastställd frekvens som är snabbare än vad en exponering tillåter ger den medlemmen ett tak som är tillräckligt långt för den, och den tar emot utlösningar som inte är snabbare än vad dess exponering tillåter. Sänk exponeringen eller den fastställda frekvensen när varje medlem måste hinna med varje utlösning.
- **Återläsning:** när man läser av en medlems noder visas `AcquisitionFrameRateEnable` True och `AcquisitionFrameRate` med medlemmens hastighet; aktiveringen stängs av igen när matrisen stannar.
- **Arrayen äger dessa två noder. Skriv inte `AcquisitionFrameRateEnable` eller `AcquisitionFrameRate` på en arraymedlem** — ställ istället in arrayenshastighet med `target_fps` / `set_target_fps()` istället.

Skriv in en medlems exponering med `arr.set_member_settings(serial, exposure_time=20000)` (nyckelorden i `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` gör samma sak). Medlemmen förblir i matrisen. Öppna inte en matrismedlem med `connect_camera()`: connect avvisar en kamera som redan finns i en matris, och det värde som `CameraSession` skulle returnera tar bort kameran från matrisen när den kopplas bort. `ArraySession.status()["member_ae"]` rapporterar varje medlemsexponering och förstärkning från den senaste bilden.

### En projektpost, alla lägen

En projektmapp innehåller inställningarna för varje enhet: `cameras.json` — varje kameras (`cameras[].settings`, för en enskild kamera och för varje medlem i matrisen) och varje matris gemensamma inställningar (`arrays[].capture`: format, ROI, binning, utlösningsfrekvens, kabelbudget) — samt `sensors.json`, varje ljussensors inställningar (integrationstid, bildgenomsnitt, tak). Appen, CLI, SDK och MCP-servern läser och skriver alla samma filer på samma sätt:

- **En enhet som är ansluten till ett projekt sparas i det och registrerar varje ändring där.** ”Under ett projekt” innebär att ett projekt är öppet i backend (appens), eller — via en `ChlorosProject` (`open_project()`), som anger sin egen mapp till backend (`project_path`). En kamera, en sensorarray eller en ljussensor som projektet ännu inte innehåller läggs till i dess filer när den ansluts (`connect_camera()`, `connect_array()`, ljussensorns anslutning, CLI, appen, `chloros-mcp`), tillsammans med dess inställningar. En ändring från vilket läge som helst (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, matrisinställningarna, `set_cap()`, CLI, appens kontroller) sparas i projektets filer en stund senare. En exponering, förstärkning eller ett AE-mål som ställts in på en kamera sparas som inställt, så att det behålls när projektet öppnas igen.
- **När ett projekt öppnas ställs dess enheter in utifrån projektet**, i alla lägen: en kamera, matris eller ljussensor som är ansluten utan egna inställningar får projektetssparade inställningar (`lattice cam-connect` / `array-connect` när ett projekt är öppet, `chloros-cli project connect`, `open_project().connect_all()`, appens återöppning). En matris som ansluts under ett nytt ID uppdaterar den matris som projektet sparade med samma kameror; en matris med andra kameror läggs till bredvid den.
- **Appens egen sparning skriver aldrig över en enhets aktuella inställningar** med sin äldre kopia av dem, och behåller de överskrivningar per medlem som en CLI- eller SDK-anropare har sparat.
- **Det pixelformat du ställer in är det som projektet behåller.** Ett format som ställts in i Array-inställningarna, via `--pixel-format` eller via `connect_array(pixel_format=)`, inklusive 8-bitars, sparas och återansluts exakt. Om inget är inställt ansluts varje kamera med sitt 12-bitars standardvärde (`BayerRG12`, `Mono12` på en monosensor). Ett formatnamn anger dess bitdjup; familjen är sensorns, så en färgkamera ställs aldrig in på ett monofärgformat och en monokamera aldrig på ett Bayer-format, och raden `[PixelFormat <serial>] ... written` anger detta när ett format har mappats. Exporter från en 8-bitarsinspelning är 8-bitars, inklusive reflektans: omskalade och märkta med dess `PixelScale`.
- En enhet som är ansluten utan något projekt registrerar ingenting.
- **En inspelning följer ett projekt som skapats efter den; när den öppnas behåller den sitt projekt.**När ett projekt skapas — `create_project()`, `chloros-mcp`:s `create_project` eller appens**Nytt projekt**— eller dupliceras (appens**Duplicera**, som öppnar kopian) medan en inspelning fortfarande skriver till det projekt som var öppet tidigare, fortsätter inspelningen, och det som skrivs hamnar i det nya projektet, på samma plats inuti det (nedan). Att öppna ett befintligt projekt — `ChlorosProject.process()` (vilket öppnar projektet i bakgrunden), `chloros-cli process`, appens**Öppna projekt** — lämnar en inspelning som fortfarande pågår i det projekt den spelar in i: den avslutas där, och dess filer läggs till i det projektets fillista (appen avbryter varje inspelning innan den öppnar ett projekt).
  - `ArraySession.record()` och `ArraySession.burst()`: videon, eller burst-mappen med varje bildruta, flyttas dit i sin helhet när inspelningen avslutas. Resultatet av avslutningen anger den nya sökvägen (`out_dir`, `video_path`), och en sammanställning av bildserien skapas utifrån den. Om en flyttning avvisas av enheten lämnas inspelningen kvar där den skrevs, och resultatet anger detta (`relocation_error`). Råbildrutorna från en avbruten bildserie och `.daq` läggs till i fillistan för det projekt som den hamnar i; en video läggs inte till i projektets fillista.
  - `ArraySession.continuous()`: varje inspelning efter ändringen sparas i det nya projektet; de som sparats tidigare stannar kvar där de är.
  - `DAQSensorSession.record_start()` (och appens egen ljussensorinspelning): `.daq` flyttas till det nya projektets `light_sensor`-mapp när inspelningen avslutas; `record_stop()` returnerar den nya sökvägen, och filen läggs till i det projektets fillista.

  En inspelning i en mapp utanför det öppna projektet stannar kvar där den är, och `start_camera_continuous()` / flerströmsjobb behåller den mapp de tilldelades.
- **I appen hamnar inspelningarna efter ett nytt projekt i det.** Att skapa eller duplicera ett projekt gör att alla pågående inspelningskörningar fortsätter, och varje inspelning som görs därefter (Capture All, Start Capturing) sparas i det nya projektet och läggs till i dess fillista. När ett befintligt projekt öppnas i appen stoppas först alla inspelningar.
- **Ett projekt flyttas inte medan det spelar in.**Att flytta det öppna projektet till en annan arbetskatalog (appens**Inställningar → Arbetskatalog**; inget SDK-anrop flyttar ett projekt, en HTTP-klient som skickar till `/api/move-project` gör det) avvisas med HTTP 409 `PROJECT_RECORDING` medan en arrayinspelning eller burst, en kontinuerlig arrayinspelning, ett kontinuerligt kamera-/multiströmningsjobb eller en ljussensorinspelning skriver till den: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Detta är skilt från hur kamerorna lämnas vid frånkoppling (se [Kameraminne](#camera-memory--keep_settings--clear_settings)): ett projekt ställer in sina kameror själv när det öppnas.

### Kameraminne — `keep_settings` / `clear_settings`

När en matris ansluts övertar dess kameror matrisens inställningar, och du kan sedan ändra dem. När Chloros kopplar bort kamerorna — `ArraySession.disconnect()` / `force_disconnect()`, i slutet av en övergående funktion, vid avslutningen av `start_camera_continuous()` — återställs varje kamera till det läge den befann sig i före anslutningen: dess utlösare och synkroniseringsledningsanslutningar, PTP- och kamerainställningar — `ExposureAuto` / `GainAuto` (samt en manuell `ExposureTime` / `Gain`), `TargetBrightness` samt AE-gränserna och dämpningen, vitbalansen, svartnivån, `PixelFormat`, binning och ROI, färgpipeline (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTP:s `PtpSlaveOnly` / `PtpDomainNumber`, paketstorleken (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), stimuleringsfrekvensen (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), blockdata (`ChunkModeActive`, varje `ChunkEnable` under sin `ChunkSelector`), deglitch för triggingången – en matris ger en slavs synkroniseringslinje (`LineFilterSelector` / `LineFilterWidth`) och bildhastigheten för datainsamlingen (`AcquisitionFrameRate` endast när dess aktivering var på: när den är av läser hastigheten det högsta som takten tillåter, inte en inställning). Endast det som skiljer sig åt skrivs ut: kamerans första stopp, takten och bildhastigheten före en exponering som kräver sin bildperiod.

För att använda kamerorna utanför ett projekt skriver två argument in kamerans **eget minne** — det den startar upp med, i vilken programvara som helst (på `disconnect()` / `force_disconnect()`, alla transienta funktioner, `start_camera_continuous()` och `ChlorosProject.disconnect_all()`; MCP: samma argument som för `disconnect_device` och de transienta verktygen; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** sparar varje kameras inställningar i minnet precis som de var vid sessionens slut (UserSet2, som utgör startinställningarna). Kamerorna lämnas fortfarande i fritt läge (`TriggerMode` `Off`), med synkroniseringsledningarna som de var. En kamera utan UserSet2 sparar till UserSet1; den här datorn kommer ihåg detta, så Chloros-matrisens självläkning — som återställer en UserSet1 som har avvikit från Chloros-baslinjen — lämnar den ifred.
- **`clear_settings=True`** återställer varje kameras minne till fabriksinställningarna: Chloros baslinje, den som dess matris-självkorrigering använder — fabriksinställningar med `TriggerMode` `Off`, `TriggerSource` `Line2`, automatisk exponering, en RGB-kameras kalibrerade vitbalans bevaras — i UserSet1, som utgör startuppsättningen.
- Inte båda (`ValueError`; backend svarar 400). Svaret innehåller `camera_memory`, per kamera `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` eller `{"cleared": bool, "errors": [...]}` — `force_disconnect()` returnerar det; för ett jobb medföljer det `stop()`.

Appen erbjuder inte dessa.

**Varje utgång återställer kamerorna.**Återställningen till ursprungligt läge sker på varje utgångsväg: `CameraSession.disconnect()` (appens kamera**X**), appens**X** i en array eller någon arraymedlem (hela arrayen frigörs), stängning av appen (backend får upp till 20 s på sig att återställa varje kamera, array och pågående kontinuerlig inspelning; en transient funktion eller ett engångsanrop som fortfarande körs stannar vid sin nästa säkerhetspunkt, återställer först sina kameror, i upp till 8 sekunder, och svarar med HTTP-kod 503 `SHUTTING_DOWN`; en `connect_array()` som fortfarande körs avbryts före sin nästa kamera och återställer de kameror den har öppnat, i upp till 10 sekunder, och genererar `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); och en som anländer under avstängningen öppnar ingenting: HTTP 503 `SHUTTING_DOWN`), samt ett `chloros-cli`-kommando som avslutas eller vars terminal stängs (dess uppdrag avbryts och dess kameror återställs först). En backend som kraschar eller tvingas stängas ner kan inte återställa någonting; en omstart återställer varje kamera till dess minne.**Ett projekt ställer in sina kameror själv.** När en array ansluter via ett projekt – appens öppna projekt eller ett `ChlorosProject`, som identifierar sig själv för backend-modulen (`project_path`) – får varje medlem de inställningar som projektet har sparat för den: en exponering, förstärkning eller ett AE-mål som hålls fast för den (hålls som ditt eget, så automatisk exponering påverkar den inte) och dess vitbalansläge (korrigering enligt Chloros; kamerans egen vitbalans förblir avstängd), oavsett vad kamerans minne innehåller.

### Fotograferingslägen och inspelningsenheter

Matrisens yta speglar GUI:ns fotograferingspanel: lägena Enstaka / Kontinuerlig / Intervall / Snabbaste slutartid, plus två inspelningsenheter (live-kompositvideo och råbildsserie → offline-bearbetning).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**är SDK:s kontinuerliga/intervall-loop. Eftersom det inte finns något `Ctrl+C` för att avbryta den från ett skript,**måste** du skicka `count` och/eller `duration_s` (den avbryts när någon av dem nås). `interval_s` mäts från början av varje omgång (vilket stämmer överens med GUI:n). Kontinuerliga genomgångar (`interval_s=0`) kedjar ihop `group_seq` → `after_seq`, så de är på varandra följande synkroniserade grupper i arrayenshastighet. Återstående kwargs skickas direkt vidare till `capture()`.
- **`continuous`**är backendens egen kontinuerliga inspelning — GUI:ns kontinuerliga läge.**Använd den istället för `capture_repeated()` när du vill att varje utlösare ska sparas på disken med hög bildfrekvens:** `capture_repeated()` skickar en HTTP-förfrågan per grupp från ditt skript, medan `continuous()` håller två inspelningar igång inuti backend utan tur-och-retur-kommunikation mellan grupperna. Uppmätt vid ROI 512 på en uppsättning med 9 kameror: 73–77 inspelningar/s, jämfört med en uppsättning som publicerar cirka 78. Behåll `capture_repeated()` för intervallinspelning (`interval_s`) och för pass som du granskar ett efter ett. I Windows ska du undanta backend-modulen från Microsoft Defender för att uppnå höga hastigheter (se [Steg 1](#step-1--install-the-chloros-platform-package)).
- **`record`** är av *övervakningsklass*: den spelar in den renderade kombinerade indexkompositen i realtid. När GUI:ns kombinerade vy är öppen registrerar den den vyn som den visas. Om ingen är öppen renderar backend-modulen vyn själv: `expr` om du skickar en, annars matrisens NDVI (de röda och NIR-banden närmast 660 nm och 850 nm, så som livevyn avläser dem), så snart matrisen är inriktad. Namnge banden i `expr` enligt matrisens benämningar: `<band>_<filter>` för en M3C-kamera (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` för en M3M (`Mono_850`); ett kort färgnamn (`NIR`, `Red`) fungerar endast om ett band i matrisen har det. En GUI-vy som öppnas under inspelningen spelas in så länge den förblir öppen. En inspelning som inte skriver någon bildram misslyckas med `error` och `"no-frames: …"` (matrisen kan fortfarande vara i färd med att justera eller stabilisera sin exponering), och `stop()` genererar ett undantag (se [`RecorderHandle`](#recorderhandle)). En sammansatt inspelare per matris (genererar ett undantag om en redan körs).
- **`burst` → `build_video`** är av *analyskvalitet*: `burst` skriver råa bildrutor + ett manifest per bildruta + en `.daq` per distinkt DLS-avläsning under `<output>/bursts/<base>/` med hämtningsslingans fulla hastighet (ingen kedja, inget exiftool, ingen live-visning). `build_video` tidsmatchar varje bildruta mot närmaste `.daq` och kör om importpipelinens radiance/reflektans/index-kedjan. `products` är en lista över `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (standard: det kombinerade indexet). `burst().stop()` startar också automatiskt en ”best-effort”-kombineradindexbyggnad, som returneras som `build_job` i stoppresultatet.

##### Resultatet `build_video`

| Nyckel | Betydelse |
| --- | --- |
| `success` | `True` endast när varje begärd produkt har skapats enligt begäran: minst en utdata, ingenting i `skipped`, och inte i `degraded`. |
| `outputs` | Sökvägar till de videor (och GIF-filer) som skrivits ut. |
| `errors` | Problem, som text. |
| `skipped` | En `{"serial", "level", "reason"}` per begärd produkt som inte genererade **någon** video — en kameras (inget kalibreringspaket för den kameran, ingen DLS-avläsning för reflektans, bearbetningskedjan misslyckades) eller det kombinerade indexet, som innehåller `"kind": "combined"` och `serial` `None` (inget rött/NIR-bandpar bland kamerorna, ingen grupp med strålningsintensitet för båda banden). Varje post genererar `success` `False`; de övriga videorna skrivs fortfarande ut och listas. |
| `substitutions` | Kameror som återges på en annan nivå än den begärda, och varför — en RGB-kamera som begär strålningsintensitet/reflektans får sin visningsvideo, vitbalanserad när kamerans kalibreringspaket är tillgängligt. Inget fel. |
| `notes` | Informationstext, inga problem — t.ex. har en kamera utan indexuttryck per kamera ingen indexvideo, eller så är en RGB-kameras visningsvideo inte vitbalanserad (ingen kalibreringspaket, eller ett utan fabriksvitbalans), där kamerans namn anges. Inget fel. |
| `degraded` | `True` när en produkt har skrivits men inte är den som begärts: det kombinerade offline-indexet är den inbyggda normaliserade skillnaden mellan rött och NIR, så ett konfigurerat indexuttryck utvärderas inte. Ger `success` `False`. |
| `combined` | Sammanfattning av det kombinerade indexet: de använda banden, LUT-intervallet, skapade bildrutor. |
| `clamp_masks` | Sökvägar till [unmix-klämmaskerna](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) för deram kalibrerade TIFF-filerna (`save_tiffs=True`), i `clamp_masks/<level>/` bredvid TIFF-filernas `<level>/`-mapp, en per TIFF-fil där en pixel klämts fast. En video innehåller inga. |
| `clamp_fractions` | Vad en video anger istället för en mask: en `{"kind", "serial", "level", "frames"}` per videoprodukt (`kind` `"combined"` för kamerorna som ingår i det kombinerade indexet), `frames` en lista över `[frame, R, G, B]` — den bildramens andel av pixlar som avblandningen begränsade till det band som dess R/G/B-kanal innehåller (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Returneras av `ArraySession.record()` och `ArraySession.burst()`. Använd den som en kontextmanager för att automatiskt avsluta vid utgång ur scope, eller styr den manuellt.

| Medlem | Beskrivning |
| --- | --- |
| `job_id` | Backend-jobb-id (str). |
| `kind` | `"composite"` (från `record`) eller `"raw"` (från `burst`). |
| `start_stats` | Den ordlista som returneras av anropet `start`. |
| `result` | `None` under körning; den slutliga stoppresultatdiktionen när inspelningen har stoppats. |
| `stats(timeout=10.0)` | Live-jobbstatistik (skrivna bildrutor, uppnådd bildhastighet, förfluten). |
| `stop(timeout=60.0)` | Stoppa inspelaren; returnerar och cachar det slutliga resultatet (`.result` förblir `None` om själva stoppbegäran misslyckades). För en `record()`-inspelning som misslyckades – en som inte skrev någon bildruta (`error` `"no-frames: …"`) — cachar den resultatet på `.result` och genererar sedan ett undantag på `ChlorosConnectError`. Idempotent (ett andra anrop returnerar det cachade resultatet utan att generera ett undantag). Om man lämnar ett `with`-block avbryts inspelningen utan att ett fel genereras, så kontrollera `rec.result.get("error")` efter det. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Returneras av `ArraySession.continuous()`. Använd det som en kontextmanager — om du lämnar `with`-blocket stoppas inspelningen om den fortfarande körs — eller styr den med `status()` / `wait()` / `stop()`. Endast en kontinuerlig insamling per array kan köras åt gången.

| Medlem | Beskrivning |
| --- | --- |
| `start_status` | Statusdiktet som returneras av startanropet. |
| `result` | `None` under körning; den slutliga statusdiktionen när insamlingen har avslutats (ställs in av `stop()`, eller av ett `status()` / `wait()` som upptäckte att den avslutades). |
| `status(timeout=10.0)` | Live-statusdiktat (fält nedan). |
| `wait(timeout=None, poll_s=0.5)` | Blockera tills insamlingen avslutas av sig själv — `max_captures` eller `duration_s` har uppnåtts, arrayen är borta, en fullständig enhet, ett fel — och returnera den slutliga statusen. Med `timeout` returneras den senaste statusen efter det antalet sekunder även om den fortfarande körs: kontrollera `running`. |
| `stop(timeout=30.0)` | Avbryt insamlingen och returnera dess slutstatus. Idempotent (ett andra anrop returnerar det cachade resultatet). |

Statusfält:

| Fält | Betydelse |
| --- | --- |
| `running` | Sant medan inspelningen pågår. |
| `captures` | Synkroniserade grupper sparade. |
| `files` | Filer skrivna. |
| `failed_files` | Filer som inte kunde skrivas (som i [`CaptureResult.failed`](#filer-som-inte-kunde-skrivas--insamlingsresultat-misslyckades)). |
| `failed_captures` | Insamlingsförsök som misslyckades i sin helhet, inklusive sådana där insamlingen gjordes om (ett HTTP-fel 503 medan arrayen inte hann skapa någon ny grupp i tid). |
| `skipped_groups` | Grupper som arrayen publicerade men som inte sparades (insamlingen hamnade efter). |
| `captures_per_s` | Genomsnittlig sparningsgrad sedan starten. |
| `elapsed_s` / `started_at` | Antal sekunder som gått hittills; starttid (Unix-epoksekunder). |
| `last_group_seq` | Den senaste synkroniserade gruppen som sparats. |
| `last_error` | Det senaste felet, eller `None`. |
| `stop_reason` | När det har avslutats: `count` (`max_captures` nått), `duration` (`duration_s` nått), `stopped` (`stop()` anropat), `array_gone` (matrisen kopplades bort), `error` (20 fel i rad, eller en begäran som backend avvisade), `disk_full` (en fil som enheten inte hade utrymme för). `None` under körning. |
| `max_captures` / `duration_s` | De gränser som den startades med (`0` = inga). |
| `defender_cores` | CPU-kärnor som Microsoft Defender använt under de senaste 10 sekunderna; `None` tills en sådan mäts, och alltid `None` på en värd utan Defender. |
| `defender_warned` | Sant när avläsningen har loggat att Defender saktar ner den (vid 1 kärna eller fler). Uteslut backend från Defenders skanning – se [Steg 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Live-ramar i din egen pipeline

För att mata in synkroniserade bildrutor i din egen kod — maskininlärning, en anpassad visare, din egen inspelare — använd `ArraySession.frame_stream()`. Varje tick genererar en `GrabResult`: `{serial: numpy array}` för varje kamera, **allt från en enda utlösare**, med den utlösarens ID på `.group_id`. Det är samma implementering som projekt-API:s `ArrayHandle.frame_stream()`, så allt under [En utlösare per läsning](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) och [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) gäller för båda.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Varje genererad uppsättning är en ny utlösare.** `frames.group_id` upprepas aldrig inom en ström; när du begär fler `fps` än vad arrayen kör, väntar strömmen på nästa utlösare, så den körs med den långsammare av de två.
- **Kameror som saknas namnges, de fylls inte i i efterhand.** En kamera som inte kunde betjänas vid den utlösaren saknas i ordlistan och listas i `frames.errors`.
- **Det avslutas med ett ljud.** `FrameFetchError` vid ett avslag skulle nästa tick upprepas, efter 60 sekunder utan bildruta eller efter 60 sekunder utan ny utlösare.**Vad det levererar.**`frame_stream()` överför hela numpy-ramar via den lokala HTTP-anslutningen och når en topphastighet på nära**170 MB/s**. Det räcker för livebearbetning av råa bildrutor, eller av en binned- eller ROI-matris, men inte för varje utlösare vid full upplösning — för att behålla**varje** utlösare, spara till disk med [`continuous()`](#continuoushandle). Uppmätt på en uppsättning med fyra kameror (FOCN, FRGN, FRGB, F450) med packad överföring; varje tick var en komplett uppsättning med en utlösning:

| | Full sensor (matris 14,2–14,4 fps) | 2 × 2 binning (matris ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 uppsättningar/s | 22,2 uppsättningar/s |
| `frame_stream(processing="debayered")` | 2,3 uppsättningar/s | 9,3 uppsättningar/s |
| `frame_stream(processing="display")` | 0,8 uppsättningar/s | 3,0 uppsättningar/s |
| MJPEG per kamera, 4 strömmar vid 720p | 13,5 fps vardera; varje bildruta matchas av en utlösare | ~25 fps vardera; ~45 % av utlösarna finns på alla 4 |

Använd `processing="raw"` för genomströmning och demosaikering på din sida; nivån `debayered` tar även bort kalibreringspaketets mörka offset, vilket en demosaik av `raw` inte gör.

#### MJPEG-strömmar per kamera (ingen RTSP)

**RTSP stöds inte.** Strömningsgränssnittet är MJPEG över HTTP (`multipart/x-mixed-replace`, en JPEG per del, varje del med en `Content-Length`), en URL per kamera:

| URL | Vad den levererar |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8-bitars förhandsgranskning, upp till ~30 fps. För en arraymedlem anger `max_h` (standard 720, 64–2160) och `jpeg_q` (standard 80) anger storlek och kvalitet. |
| `GET /api/camera/<serial>/stream-annotated` | Samma förhandsgranskning med GUI-överlägg ritade på bilden (zebra, rutnät, hårkors, histogram, peaking, spotmätare). `ChlorosLocal.camera_stream()` läser av den åt dig. |
| `GET /api/camera/<serial>/stream-reflectance` | 8-bitars reflektans per band — se [Live-reflektansström](#live-reflectance-stream-mjpeg). |

- **Trigger-rubriker.** För en arraymedlem innehåller varje del av alla tre strömmarna `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` och `X-Chloros-Timestamp-Ns`. Delar från olika kameror med samma `X-Chloros-Group-Id` utgör samma utlösare: samla delarna och gruppera dem efter den rubriken för att sätta ihop en uppsättning igen. En fristående kamera har ingen grupp: dess förhandsgranskningsdelar innehåller ingen av dessa rubriker (dess reflektansdelar har en tom `X-Chloros-Group-Id`).
- **Förhandsgranskningsströmmen tappar bildrutor vid hög belastning.** Den levererar den senaste bildrutan och hoppar över alla som den inte kunde skicka i tid, så vid höga matrishastigheter når inte varje utlösare varje kameras ström (se tabellen ovan). Använd `frame_stream()` eller `continuous()` för varje utlösare.
- **`/api/camera/stream/<serial>` pausar medan arrayens kombinerade livestream är öppen** (`/api/camera/array/stream/<array_id>` — GUI:ns arrayvy öppnar den). Varje enskild kameras ström skickar då en platshållarbild med texten ”Pausad (matrisström aktiv)”, utan `X-Chloros-*`-rubriker, och inget mer förrän den sammanslagna strömmen stängs; därefter återupptas den av sig själv. `stream-annotated` och `stream-reflectance` pausar inte. Stäng GUI:ns matrisvy när ett skript läser förhandsgranskningsströmmarna per kamera.
- **För siffror, inte bilder, använd `frame_stream()`.**JPEG är förlustrikt och 8-bitars; `frame_stream()` returnerar samma resultat utan någon kodek i sökvägen.

### Intervallinspelning för drönarundersökningar

En undersökningsflygning behöver en synkroniserad bildgrupp med några meters mellanrum, inte varje bildruta som matrisen kan producera. Två inställningar styr detta:**`target_fps`**anger hur ofta kamerorna tar en bildruta, och**`capture_repeated(interval_s=…)`** anger hur ofta en bildruta sparas.**Kamerorna fortsätter att strömma mellan sparningarna.** Vid varje intervall sparas den senaste synkroniserade gruppen; bilderna däremellan sparas inte. Den automatiska exponeringen justeras för varje bild, så exponeringen har redan stabiliserats när sparandet sker. Det finns inget läge där kamerorna stannar mellan bilderna och väcks för varje enskild bild: varje bild skulle då behöva vänta på att strömmen startar och att exponeringen stabiliseras.**Ställ in bildfrekvensen efter vad intervallet kräver.**Om den lämnas på auto körs en array med den högsta hastighet som värden och nätverket klarar av. Att ställa in en lägre hastighet minskar kamerans värmeutveckling, nätverksbelastningen och värdens CPU-belastning utan att det som sparas förändras. Välj en hastighet som ger**2–4 bilder per intervall**:

- Varje sparning väntar på nästa bildruta efter att den begärts, så en sparad grupp hamnar upp till en bildrutperiod efter det schemalagda ögonblicket. En högre bildfrekvens håller det gapet litet.
- Autoexponeringen justeras en gång per bildruta, så en mycket låg bildfrekvens saktar ner hur snabbt exponeringen följer ett förbiilande moln.

**Beräkna intervallet utifrån flygplanen:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Till exempel ger en fotavtrycksbredd på 60 m vid 80 % framåtriktad överlappning ett avstånd på 12 m; vid 10 m/s blir det `interval_s=1.2`, och ett fastställt `target_fps=3` ger mellan 3 och 4 bilder per intervall. Bildtagningen är tidsbaserad: SDK:t har inget avstånds- eller GPS-utlöst läge, så håll en jämn markhastighet längs kartläggningslinjerna.

**Spara rådata på flygplanet och kalibrera senare.** `interval_s` mäts från början av varje sparning, så en sparning som tar längre tid än intervallet gör att nästa startar för sent och avståndet glider. Genom att spara rådata tillsammans med ljussensorns avläsning (`processing="raw"`; avläsningen sparas som standard) hålls varje sparning kort och varje bildruta kan senare konverteras till reflektans.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**låser hastigheten från start;**`arr.set_target_fps(fps)`**ändrar den under flygningen utan att återansluta, och**`arr.set_target_fps(None)`** återställer den till automatisk. En hastighet som överstiger vad värden och nätverket klarar sänks till den gränsen — kontrollera `clamped` i resultatet — och den lägsta hastigheten är 0,1 fps.
- **Använd inte `burst()` för att gallra bort bildrutor.** Den skriver ut varje bildruta som arrayen producerar; den följer visserligen en fastställd `target_fps`, så den är endast rätt verktyg när varje bildruta önskas.
- **`target_fps` åtgärdar inte korrupta bildrutor.** Att sänka värdet minskar antalet ramar som produceras, inte hur snabbt varje ram skickas; för GVSP-skadade ramar ska du istället sänka `wire_ceiling_mbps` (se [Array Health](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Ansluta till en redan ansluten array — `attach_array`

Om arrayen redan är igång (antingen via GUI eller genom att en tidigare SDK-session anropade `connect_array`), använd istället `attach_array` för att hämta ett handtag till den istället för att ansluta på nytt. `connect_array` ger alltid felmeddelandet ”Kameran  <sn>finns redan i arrayen<id>” i den situationen, eftersom en POST-begäran med `/array/connect` för en medlem i poolen inte är idempotent; `attach_array` läser `/api/camera/array/list` och matchar antingen efter array_id eller serienummer.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Mönster: SDK-skript som delar miljö med skrivbords-GUI:n bör först försöka med `attach_array` och sedan använda `connect_array` om det ännu inte finns någon array i poolen.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **En ansluten array är lånad, inte ägd.** Ett handtag från `attach_array` avslutar aldrig arrayen: varken när `with`-blocket avslutas eller genom ett explicit `disconnect()`-anrop frigörs den (det explicita anropet skriver ut en rad till stderr som anger att lämnade arrayen igång). Samma regel gäller för `CameraSession` / `DAQSensorSession`. Det är alltså säkert att dela resurser med GUI:et, med eller utan `with`:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> När du verkligen vill avsluta det – för att återansluta med ett annat `wire_ceiling_mbps`, till exempel – anropa `arr.force_disconnect()`. Återanslutning efter ett vanligt `disconnect()` på ett anslutet handtag misslyckas med &quot;Kameran  finns<sn> redan i arrayen <id>&quot;, eftersom ingenting har frigjorts.

### Hjälpmedel för nätverksanalys

Användbart innan du öppnar arrayen – visar om dina föreslagna inställningar kommer att passa:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Det startar aldrig en backend: en som inte svarar genererar `ChlorosConnectError`, medan en avvisad begäran returneras som resultat. Ett felaktigt `packed` genererar `ValueError`.

`status` är ett av följande: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (annars `error` — inklusive en avvisad begäran: inte inloggad, `error_code: AUTH_REQUIRED`, eller en plan under SDK-gränsen, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` innebär att den begärda upplösningen endast passar RX-ringen vid en begränsad triggfrekvens — behåll upplösningen och gå vidare från `target_fps=result["recommended"]["recommended_target_fps"]` till `connect_array` (se [Exempel 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` visas endast när den begärda bilden inte kan levereras vid någon bildhastighet; connect krymper aldrig bilder på egen hand, så tillämpa `recommended` själv.

**Hur man tolkar projektionen** (samma modell som panelen Array-inställningar i GUI):

- **Burst (`frame_bytes_total`) summeras per kamera i varje kameras faktiska pixelformat.**Mono**M3M**-kameror strömmar Mono12 (2 B/px), eller med en `pixel_format` i Mono-formatet för dess djup (`"BayerRG8"` är `Mono8`, 1 B/px), så utan detta blir en bildruta i full upplösning från fyra kameror**~25 MB** med tre monokameror, inte de ~12,6 MB som antagandet om enbart 8-bitarsdata ger. Backend-systemet avgör varje kameras format utifrån dess modell.
- **Admittance (`burst_fits_nic_ring`) är dräneringsmedveten**, inte hela-burst-vs-ring: sim-emit passar när värden tömmer RX-ringen snabbare än kamerorna fyller den. En 10G-värd + 1 GbE-kameror**tillåter** full upplösning även när burstet överskrider ringen; en 1 GbE-värd blockerar (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` är ett konservativt tak för seriell hämtning** — `max(readout+emit, N×emit)` med utsändning per kamera begränsad till 1 GbE-kameralänken, oberoende av exponering. T.ex. ~2,8 fps för en 4-kameras 12-bitarsmatris i full upplösning (stämmer överens med runtime-värdet ~2,7–3,0). Fullständig modell: [CLI-referens → Modell för bildhastighet och burst-läge för bildserier](cli-reference.md#array-fps--burst-model).
- **Efter en kontinuerlig inspelning anger projektionen vad den uppnådde.** En inspelning på 30 s eller mer med automatisk hastighet registrerar den hastighet som stabiliserades vid den arraystorleken på denna dator; hastighetsregulatorns korta försök med en snabbare hastighet utelämnas. Från och med då anger `array-connect`:ss rad, `connect_array()`:s analys och panelen Arrayinställningar anger den hastigheten när den ligger under modellens, tillsammans med vad som begränsade den i `cycle_limiter`: `"clock"` (matrisen kördes med den hastighet den stabiliserades på, medan insamlingsslingan väntade på sin triggningsklocka), `"loop"` (insamlingsslingan hade ingen tid kvar), `"capture"` (inspelningen hamnade efter och hoppade över grupper) eller `"measured"` (inget anger vilket). Modellens siffra finns i `wire_fps_max`. Arrayen stiger fortfarande till det som kabeln klarar av.
- **Överteckning (`oversubscribed: true`) innebär att kabeln inte kan hantera arrayen vid någon av de bildhastigheter som kamerorna når** — fps-fälten (`achievable_fps_max` / `fps_bright` / `fps_dark`) visar 0, och binning kan inte åtgärda detta (det minskar antalet byte per bildruta, inte antalet synkroniserade byte per sekund). En långsam kabel i sig orsakar inte detta: istället sjunker bildfrekvensen, och om kamerorna inte kan synkroniseras tillräckligt långsamt blir nivån `slip-emit-and-capture` (seriell). Lösningarna är färre kameror, jumbo-ramar eller ett snabbare nätverkskort; `max_cams_collision_safe` anger gränsvärdet för din anslutning och dina kameror. Svaret innehåller även `aggregate_demand_bps`, `collision_safe_ceiling_bps` och `per_cam_floor_bps`. Se [Överteckning](#over-subscription-the-per-cam-floor).

### Upptäckt och listning

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Var och en tar endast nyckelord `backend_url=` och `timeout=30.0` och returnerar en lista med ordböcker. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) tar även `auto_start_backend=True`; anropen `list_*` startar aldrig en backend.

---

## Smart-AE / Smart-Capture

LATTICE-matriser kör kontinuerlig AE i bakgrunden så snart de är anslutna, men det tar en stund för en nyinsatt scen att konvergera. **Smart-capture** är en praktisk funktion: den avläser exponeringen för varje kamera, väntar tills arrayen är stabil över ett fönster och utlöser sedan bildtagningen. Det motsvarar GUI:n – skrivbordsappens ”smart”-knapp för bildtagning anropar samma backend-ändpunkt.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

När du styr via `ChlorosProject` (nästa avsnitt) får du fler inställningsmöjligheter:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Smart-AE-policyn är konservativ som standard. Skärp inställningen med `exposure_tolerance_pct` för noggrant radiometriskt arbete; lätta på inställningen för snabbt föränderliga scener där du bara vill ha ”tillräckligt nära”.

---

## DAQ-sensorsessioner

Persistent backend-pool för spektralsensorer (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Speglar kamerans yta: smart-detect, återanvändning av pool, idempotent anslutning.

### Smart-Detect (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioritet: Ethernet → BLE → USB. Ange valfri explicit ledtråd för att låsa transporten.

### Låst transport

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` namnger en sensor, och endast den sensorn öppnas någonsin. Den matchas mot de sensorer som redan är öppna, därefter frågas en DAQ-E:s värdnamn eller IP (en DAQ-E som nätverksskanningen endast ser via IP tillfrågas, skrivskyddat, om dess namn, MAC och serienummer, så att även dessa kan hitta den), en DAQ-M:s BLE-MAC samt det ID som en DAQ-U eller DAQ-M rapporterar när den öppnas (alla andra som öppnas för kontroll stängs igen). Om den inte hittas, utlöser `connect_daq_sensor` ett `ChlorosConnectError` som listar vad som hittades; tillsammans med en adress (`port=` / `mac=` / `eth_host=`) som innehåller en annan sensor, genereras ett felmeddelande även där. Den faller aldrig tillbaka till den sensor som är synlig — det är vad anropet med nollflaggan är till för.</id></sn></id></sn>

### `connect_daq_sensor()` Signatur

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

En sensor som redan finns i poolen returneras som den är (lånad): `integration_time`, `frame_avg` och `enable_ae` tillämpas inte på den. Koppla bort den först för att öppna den med dina inställningar. `cap_id` är undantaget: på en lånad sensor ställs den in med `set_cap()`. Om den utelämnas gäller backendens standardinställning (se anmärkningen under metoder-tabellen).

### `DAQSensorSession`-metoder

| Metod | Beskrivning |
| --- | --- |
| `status(timeout=10.0)` | Sammanfattning av poolinmatning (strömnings-/inspelningsstatus, våglängdsområde, kalibrerings-sha, integrationstid, frame_avg, AE-status). För en DAQ-E vars ström har tystnat: `reconnect` (där backendens automatiska återanslutning gäller: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` medan den strömmar), `stream_lost_s`, `reconnects` och `last_reconnect`, samt `record_gaps` för den pågående inspelningen; `rx_stream_fault` anger återanslutningsstatus tills det första spektrumet anländer. |
| `latest(*, n=1, timeout=10.0)` | Returnera upp till N senaste spektrumramar (var och en `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Återuppta / pausa strömning (handtaget förblir öppet). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Starta inspelning av en .daq-fil. Returnerar filvägen. Avvisas för DAQ-U/M utan ett AWS-kalibreringspaket (DAQ-E undantaget). |
| `record_stop()` | Avsluta inspelning. Returnerar `{path, rows, warning, gaps}`; `warning` är `None` för en felfri inspelning, annars backendens diagnos (till exempel en tom fil eller ett avbrott i strömmen), som även skrivs ut till stderr. `gaps` listar varje sträckning där strömmen inte levererade något: `start_utc`, `end_utc` (`None`: fortfarande förlorat vid stoppet), `duration_s`, `reconnected`, `marked_in_file`. Den är tom vid kontinuerlig inspelning. |
| `set_cap(cap_id, timeout=15.0)` | Ställ in den profil för kap-korrigering som ska tillämpas på avläsningarna från och med nästa: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"`, eller `"none"` (eller `None`) för en sensor utan hölje. Returnerar sensornssammanfattning, som läses tillbaka av `cap_id`. En DAQ-E-S behåller värdet för `as_recorded` oavsett vad som begärs (dess diffusor är inbyggd). Genererar värdet `ChlorosConnectError` för en profil som inte laddas. CLI-kommando: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Endast DAQ-E: slå på PTP igen för en enhet vars PTP är avstängd (`status()["rx_ptp_enabled"]` är `False`; inställningen sparas på enheten, så den förblir avstängd tills den slås på igen). Returnerar den uppdaterade statusen. Genererar `ChlorosConnectError` för en DAQ-U / DAQ-M, eller när enheten inte bekräftar. Chloros slår aldrig på en enhet igen av sig själv, eftersom avstängning kan vara avsiktlig. CLI-kommando: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Endast DAQ-E: enhetens egna inställningar. `firmware` (`running`, `bundled`, `update_available`), `ip` och `ip_mode` (`dhcp` / `linklocal` / `static`: det som körs just nu), `network` (den sparade inställningen: `mode`, plus `ip` / `mask` / `gateway` i viloläge), `network_change_pending` (gäller tills enheten kör den sparade inställningen, en som sparats med `set_network(..., reboot=False)` inkluderat), `auto_stream`, `ptp`. `firmware["update_available"]` är `None` när enhetensversion inte kunde läsas. Medan enhetens ström inte når den här datorn läses den vid en ny anslutning och ordlistan lägger till `stream_silent` (`True`), `stream_fault` och `reconnect`; en enhet som inte svarar ger upphov till fel. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E endast: DHCP (`"auto"`) eller en statisk adress. Enheten startas om för att tillämpa inställningen och ansluts på nytt; den här sessionen fortsätter att fungera och `address` följer enheten. `reboot=False` sparar endast inställningen. Se [DAQ-E-enhetsinställningar](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Endast DAQ-E: strömma vid uppstart utan ansluten programvara (sparas på enheten). Returnerar den bekräftade inställningen. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Endast DAQ-E: starta om enheten och anslut den igen med de inställningar den hade tidigare. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Endast DAQ-E: uppdatera firmware som ingår i denna Chloros-utgåva till en enhet som kör en äldre version, anslut den sedan igen (tar ungefär en minut; stäng inte av strömmen). Returnerar `updated`, `fw_before`, `fw_after`, `bundled` och en `warning` när enheten var långsam att återansluta eller återanslöt med en annan version; `updated` är `False` för en enhet som redan har den senaste versionen. Genererar `ChlorosConnectError` för en enhet vars version inte kan läsas. CLI-kommando: `pool-update-firmware`. |
| `disconnect()` | Frigör från poolen. Inget-op för anslutna men icke-ägda handtag. På en sensor som denna session har öppnat är det `force_disconnect()`, och genererar `ChlorosConnectError` när frigörandet inte bekräftas (utgången från `with`-blocket utlöser aldrig ett undantag). |
| `force_disconnect(timeout=30.0)` | Frigör sensorn oavsett vem som öppnade den — till exempel en sensor som ett kraschat skript lämnat kvar i poolen, vilken ett nytt skript endast når som ett anslutet handtag vars `disconnect()` inte gör någonting. Att avsluta en annan klients sensor avslutar all inspelning som den utför. Returnerar när backend bekräftar frigörandet, eller när det inte finns något att frigöra (sensorn finns inte i poolen, eller så körs ingen backend); en frigivning som har gått ut eller som backenden har avvisat utlöser `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Attribut: `sensor_id`, `model`, `transport`, `address` (följer en DAQ-E som `set_network()` omadresserade), `info` och `backend_url`.

> **Namnge locket.** DAQ-U, DAQ-M och DAQ-E levereras utan kondensator, och Chloros kan inte se om någon är monterad: en sensor som ansluts utan `cap_id` får värdet `sunshine_cosine`, vilket enligt dess mätningar antas vara (`cap_id_source=auto_default`). En oskyddad DAQ-U ger med den korrigeringen ett värde som är cirka 30 gånger för högt, så ange `cap_id="none"` för en oskyddad sensor (eller `set_cap("none")`), och profilen för det lock du monterat i övrigt.

### DAQ-E-S — fast diffusor och lutningssensor

**DAQ-E-S** är en DAQ-E med två skillnader som är viktiga för programvaran. Allt annat — upptäckt, anslutning, PTP, registrering, enhetsinställningarna nedan — fungerar exakt som för en DAQ-E.**Dess diffusor är fast, så den kräver ingen korrigering för lock.** Diffusorn av Sunshine-typ är inbyggd och fanns på enheten när fabrikskalibreringen mättes, vilket innebär att kalibreringen redan visar den verkliga nedåtriktade strålningsintensiteten. Chloros ställer in sitt takvärde till `as_recorded` (ingen profil) oavsett vad som begärs: ett takvärde uppkallat efter den (`sunshine_cosine`, `fov_*`, `none`) ignoreras med ett meddelande, och dess registreringar lagras som `cap_id_source=model`, så varningen om antagen gränsaktivering utlöses aldrig för den. Tillämpa inte heller en övre gräns på dess data manuellt: `sunshine_cosine` i toppen skulle visa ett värde som är ungefär 11 gånger för högt.

**Den har en 3-axlig accelerometer.** Solskyddsdiffusorns kosinusrespons gäller endast när den är vågrät, så en lutad sensor rapporterar felaktig irradians — och reflektansen ärver felet — utan några andra symptom. Varje DAQ-E-S-avläsning innehåller därför sin egen lägesinformation:

| Fält | Betydelse |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Vinklar i grader. `None` när firmware markerar vinkeln som odefinierad (fritt fall, en död axel). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Acceleration i milli-g. `mag_mg` långt från ~1000 innebär att enheten var i rörelse och att vinklarna inte är meningsfulla för det provet. |
| `cal_applied` | Inställt: axlarna och vinklarna är i diffusorns referensram (enhetens monteringskalibrering tillämpas). Rensat: rått kretskortsramverk, vilket kan visa upp till ~2,3° lutning som inte är verklig. |
| `sample_age_ms` | Åldern på accelerometerns mätvärde när spektrumet skickades. |
| `temp_raw` | Okalibrerad **trend** för chipstemperatur — endast skillnader, aldrig ett termometeravläsningsvärde. |

`DAQSensorSession.latest()` returnerar dem under varje rams `imu`-nyckel (`None` på en DAQ-U, DAQ-M eller vanlig DAQ-E). En `.daq` som registrerats från en DAQ-E-S lagrar samma värden i sina `imu_*`-kolumner (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) bredvid varje spektrum, så att läsaren kan skilja ett moln från ett stativ som har välts. På alla andra modeller är dessa kolumner tomma.

Attitydströmmen rapporterar om sina egna fält i `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), åtskilda från spektrumfälten så att ett avbrott i accelerometern aldrig tolkas som ett avbrott i registreringen. `discover_daq_sensors()` och sessionens `model` anger `DAQ-E-S`.

### DAQ-E-enhetsinställningar

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **En statisk adress måste finnas i den här datorns nätverk.** `set_network` genererar `ChlorosConnectError` för en enhet vars subnät inte inkluderar den här datornsadress ingår i sensorns nätverk: efter omstarten kan inget nå sensorn för att återställa inställningarna. Använd endast `force=True` när den här datorn ska omadresseras eller dirigeras till det nya subnätet.
- **`set_network`, `reboot` och `update_firmware` startar om sensorn** och genererar `ChlorosConnectError` medan den registrerar, och när den inte återkommer inom `comeback_timeout` sekunder. Sensor-ID:t och värdnamnet `daq-e-<id>.local` ändras aldrig.
- **En inspelning som påbörjades strax före omstarten avslutas av den och ges ett namn.** När sensorn återkommer innehåller det returnerade dict-objektet `recording_closed` (filens sökväg) och ett `warning` som anger detta. Om sensorn inte återkommer, eller om överföringen av firmware misslyckas, anger `ChlorosConnectError` detta istället: meddelandet avslutas med en varning, och `e.recording_closed` / `e.warning` aktiveras (båda `None` när ingen inspelning avslutades). Starta inspelningen igen när sensorn är tillbaka.
- **Endast sensorn själv återansluts.** Efter omstarten måste enheten svara där den befann sig -- och efter `set_network("auto")` även under sitt `.local`-namn -- med sitt namn, värdnamn och modell, från den nätverksadapter (MAC) den hade; `address` blir den IP-adress som svarade. Ett svar under namnet räknas endast med den MAC-adressen, så från en dator som når sensorn via en router, eller från WSL2:s standard-NAT-nätverk, återansluts inte en sensor som flyttats via DHCP (`ChlorosConnectError`; anslut den igen till dess nya adress).
- **En firmwareuppdatering kräver att sensorn ansluter igen.** `update_firmware()` anropar enheten, och enheten öppnar en TCP-anslutning tillbaka till den här datorn för att hämta bilden, på en port som operativsystemet väljer. Backend lyssnar på det nätverkskort som når enheten (på alla nätverkskort endast när det inte går att avgöra vilket) och betjänar endast den enheten: alla andra anslutningar avvisas (`[DAQ-FW] … refused a firmware connection from <ip>` i backendens utdata) medan den fortsätter att vänta. En brandvägg som blockerar anslutningen gör att uppdateringen misslyckas med `device never connected back on TCP <port>`. Windows-installationsprogrammet lägger till regeln för detta (se [Steg 1](#windows-exe)); på en annan brandvägg ska inkommande TCP-trafik till backend från sensorns subnät tillåtas.
- **Under WSL2 ska du använda speglat nätverk** (`networkingMode=mirrored` i `.wslconfig`): i standard-NAT-läget avvisas alla statiska adresser (WSL:s egen adress finns inte i sensorns nätverk), strömmen når inte WSL och en firmwareuppdatering kan inte slutföras.
- GUI:n upprätthåller sin egen anslutning till en sensor: en omstart från ett skript avslutar GUI:ns session med sensorn. GUI:n har samma inställningar (fliken Ljussensorer, sensorns inställningar, Nätverk).

### Upptäckt — hitta en adress att ansluta till

`discover_daq_sensors()` söker igenom USB / BLE / ETH efter sensorer som du *skulle kunna* öppna. Det är DAQ-motsvarigheten till `discover_lattice_cameras()` och det enda sättet att få fram **en DAQ-M:s BLE-MAC** — en DAQ-E har ett värdnamn och en DAQ-U en COM-port, men en MAC-adress är varken tryckt på enheten eller listad av operativsystemet.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Fält | Beskrivning |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-port / BLE MAC / värdnamn — vidarebefordras till `connect_daq_sensor` som `port=` / `mac=` / `eth_host=`. |
| `display` | Läsbar etikett. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, eller `None`. En Ethernet-sensor är `DAQ-E-S` eller `DAQ-E`, eftersom den rapporterar detta när den tillfrågas, skrivskyddad, under skanningen (`DAQ-E` när den inte svarar i tid). En seriell USB-port visar värdet `DAQ-U` när dess bryggchip är av en typ som används av DAQ-U (FTDI, CH340, CP210x, Prolific) – ett tips från portens beskrivning, inte en sond — och `None` för alla andra USB-enheter, som fortfarande finns med i listan så att du kan skicka den som `port=`. En seriell port utan USB-leverantörs-ID och utan ett sådant bryggchip (en UART på moderkortet, t.ex. Linuxs `/dev/ttyS*`, en seriell Bluetooth-länk, Intel AMT) kan inte vara en DAQ-U och finns inte med i listan. |
| `extra` | Detaljer per transport (BLE-annonserat namn; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Tomma värden utelämnas. |

| Parameter | Standardvärde | Beskrivning |
| --- | --- | --- |
| `transports` | alla tre | Sekvens (eller CSV-sträng) som begränsar skanningen. Värt att ange när du vet vad du vill ha — BLE är den långsamma delen. |
| `scan_timeout` | 5 | Sökfönster per transport i sekunder; backend begränsar värdet till 1–20. |
| `timeout` | 60,0 | HTTP-tak för hela anropet (precis som på andra ställen i SDK). |
| `auto_start_backend` | `True` | Starta en lokal backend om ingen körs. Startas aldrig för en fjärrstyrd `backend_url`. |

> **Sensorer som redan är öppna i poolen visas inte.** En ansluten BLE-periferienhet slutar sända annonser och en öppen COM-port kan inte undersökas, så upptäcktsfunktionen listar endast vad som är *tillgängligt för anslutning*. Ett tomt resultat direkt efter att du har anslutit något är förväntat — använd `list_daq_sensors()` för det du redan har. Transportprotokoll vars skanning inte kan köras (ingen bleak/zeroconf installerad) hoppas över istället för att generera ett fel, så en maskin utan Bluetooth får fortfarande sina USB- och ETH-svar.

### Lista

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Vilken DAQ matar vilken kamera — `daq_bindings()`

En bilds reflektans och den `.daq` som den sparar använder den DAQ som är kopplad till den kameran eller matrisen, annars den senaste DAQ som backend har. Valet av ljussensor i GUI:n och `chloros-cli daq pool-bind` ställer in den; liksom:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` returnerar `{"cameras": {serial: b}, "arrays": {array_id: b}}`: varje `b` innehåller `binding` (det inställda värdet), `source` (`project`, från projektets `sensors.json` eller `runtime`), `known` / `known_as` (om kamera- eller matrisnyckeln namnger något som Chloros känner till, och hur: `connected`, `project` — det öppna projektets `cameras.json` — eller, för en kamera, `seen`: har öppnats på den här datorn tidigare; `known: False` för en nyckel som inte namnger någonting) och `resolved` (den pool-sensor som den nu är kopplad till: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). En bindning vid körning har företräde framför projektetsför sessionen; projektfilen ändras inte. Om man kopplar bort en kamera, en array eller en DAQ avbryts alla bindningar till den hårdvaran, oavsett vem som har ställt in dem (en arrays medlemmar binds till själva arrayen).

`set_daq_bindings()` binder endast en kamera som är ansluten, finns i det öppna projektet eller tidigare har öppnats på denna dator, samt en matris som är ansluten eller finns i det öppna projektet (eller en nyckel som redan är bunden). Varje annan nyckel genererar `ChlorosConnectError` med namnet (`400`, `code`, `UNKNOWN_BINDING_TARGET`, med nycklarna `unknown`), och **ingenting** i det anropet binds. `None` rensar alla nycklar.

### Omgivning + objektreflektans — `light_sensor_reflectance()`

Två sammanslagna DAQ:er, en som riktas upp mot himlen (omgivning) och en ned mot målet (objekt), ger målets reflektans och vegetationsindex från sina senaste avläsningar: GUI:ns par ”Omgivning + Objekt” och `chloros-cli daq reflectance` utan att sensorerna behöver öppnas en andra gång. Båda måste vara anslutna och sända data.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Band.** Varje bands reflektans är objektets band-integrerade irradians över omgivningens. `daq_band_presets(filter_code=None)` listar banduppsättningarna: en per LATTICE M3C-filter (`RGN`, `OCN`, `NGB`, kamerans egen bandmodell, så en DAQ-NDVI och en kamera-NDVI är samma storhet) samt `GENERIC` bredband. Dess `suggested` är den förinställning som matchar `filter_code`, annars det första anslutna kamerans filter (`suggested_from` anger vilket).
- **Returnerar** `bands` (per band `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (de inbyggdainbyggda), `custom` (vardera med `value`, eller `valid: False` och `error`), `invalid_bands`, `reflectance_type`, `preset` och de två avläsningarnas `timestamps` (`ts_ns`). Ett intervall som omgivningssensorn inte kunde mäta (utanför dess mätområde eller vid dess brusnivå) är ogiltigt, med angivande av orsaken, och alla index som kräver detta värde utelämnas och beräknas aldrig mot noll.
- **Skarp eller relativ.** De skalberoende indexen (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) kräver ett absolut förhållande, så de genereras endast när båda sensorerna är effektkalibrerade (var och en har en `calibration_bundle_sha`): `"Apparent Reflectance"`. I annat fall blir resultatet `"Relative Reflectance"`, med endast de skalinvarianta indexen.
- **Två olika våglängdsrutnät** jämförs utifrån deras överlappning: omgivningssensorns punkter inom objektets räckvidd, objektets spektrum interpolerat till dessa.
- **Två sensorer, aldrig en.** Att namnge en sensor som båda (samma id, i vilken stavning som helst som poolen accepterar) ger upphov till `ChlorosConnectError`: en sensors avläsning över sig själv är 1,0 på varje band, vilket inte mäter någonting. Backend avvisar det också (`POST /api/daq/indices` svarar `400`, `code` `SAME_SENSOR`, när `ambient_sensor_id` och `object_sensor_id` som SDK:t skickar är en sammanslagen sensor, t.ex. dess ID och dess port).

### Delad användning med CLI och andra skript

Backendens DAQ-sensorpool delas av alla CLI- och SDK-klienter på maskinen. Om en sensor redan finns i poolen — öppnad av `chloros-cli daq pool-connect` eller av ett annat skript — returnerar ett anrop av `connect_daq_sensor(port="COM3")` från Python ett lånat handtag: dess `disconnect()` är en no-op, så ditt skript rycker inte bort sensorn från den klient som öppnade den när scope avslutas. Detsamma gäller för `sensor_id=` (skrivet med `:` eller `-`, båda fungerar), för en DAQ-E som namngavs med sin IP när den öppnades med värdnamn (eller tvärtom), och för smart-detect med nollflaggan när backend svarar med en sensor den redan innehade — ett `with`-block frigör endast en sensor som det själv har öppnat.

Chloros-GUI:n ingår **inte** i denna pool: dess DAQ-panel hanterar sin egen anslutning till sensorn, så `connect_daq_sensor` lånar aldrig en sensor som GUI:n har öppen. Använd inte samma sensor från GUI:et och från ett skript samtidigt. En DAQ-U eller DAQ-M som GUI:et håller öppen går normalt sett inte att öppna, men en DAQ-E accepterar en andra session: skriptets anslutning ändrar enhetens insamlingsinställningar i GUI:n, och dess `disconnect()` stoppar enhetens ström, vilket avslutar GUI:ns livevisning och eventuell inspelning som pågår. Stäng först sensorn i GUI:n.

### PTP Grandmaster — `time_sync_status()`

Backend-modulen kör den PTP-grandmaster som LATTICE-kameror och DAQ-E-enheter hämtar sina tidsstämplar från. Den startas på begäran: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` och `connect_all()` startar den om den inte redan körs — inklusive en anslutning som sedan misslyckas, till exempel när `connect_all()` försöker ansluta till en DAQ-E som inte finns — och detsamma gäller `test_array_sync()` och varje anrop som öppnar kameror för ett kommando (`capture_cameras_transient()`, `start_multi_camera_stream()`, justeringsanropen, `capture_live_index()`).

- **Var det meddelas.** På de trådbundna nätverksadaptrarna som är riktade mot kamerorna (inklusive en `169.254.x.x`-lokal länkadress), annars på adaptern för standardvägen om den är trådbunden. Aldrig på Wi-Fi eller en virtuell adapter (Tailscale, WSL): ingen kamera eller DAQ-E finns bakom en sådan. Om ingen sådan adapter är aktiv körs den utan att annonsera (`health` `NO_SOCKETS`) och binder till den adapter som aktiveras nästa gång en kamera eller DAQ ansluts.
- **Efter ett anslutningsavbrott.** När kameraadaptern förlorar sin anslutning och återfår den (en kabel dras ur, en switch byts ut, en USB-adapter omregistreras), återuppbygger grandmastern sina socklar på egen hand, utan omstart, och kontrollerar om alla anslutna DAQ-E-enheter på nytt, samtidigt som den återaktiverar den som fastnat i luckan. Den återuppbygger också när alla kameror och DAQ-E-enheter som den betjänade är inaktiva i 15 sekunder.
- **Når den kabeln?** Läs `time_sync_status()` (`GET /api/time-sync/status`). `state` anger endast att grandmaster-tråden körs; `health` anger om dess paket skickas iväg: `OK` (inget fel observerat), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (vissa adaptrar fungerar inte) eller `PEERS_SILENT` (kamerorna och DAQ-enheterna som den betjänade slutade svara), med en mening i `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` räknar endast meddelanden som skickats iväg; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` och `last_delay_req_age_s` beskriver de som inte skickades, och `rebinds` / `last_rebind` återuppbyggnaderna. `sync_sends_slow` räknar de Syncs vars sändning tog längre tid än 200 us och `sync_send_max_us` är den längsta; varje Follow_Ups ursprungstid räknar högst 200 us av dess sändning. Grandmastern körs i en egen process (`own_process`, `process_pid`), med hög prioritet i Windows (`process_priority`), så att en upptagen backend inte kan fördröja tidsstämplingarna och flytta varje kameras klocka samtidigt; om den processen inte kan startas körs den inuti backenden som tidigare (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funktion | Vad den gör |
| --- | --- |
| `time_sync_status()` | Grandmasterns tillstånd, hälsa, klockidentitet, prioriteringar och meddelanderäknare (ovan). `chloros-cli time-sync status`. |
| `time_sync_peers()` | De PTP-slavar den hör (kameror, DAQ-E:er): `peers`, var och en av `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Varje ansluten arraykameras `PtpStatus`, `PtpOffsetFromMaster` och `PtpMeanPathDelay` (en kamera vars firmware saknar PTP-noder har endast sitt serienummer). `time-sync cameras`. |
| `restart_time_sync()` | Stoppa och starta om grandmastern; varje sammanslagna DAQ-E:s PTP kontrolleras på nytt efter (`daq_ptp_rechecked`). Genererar `ChlorosConnectError` när den inte återkommer. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Dess PTP-prioriteringar (0–255, lägre värde vinner valet av bästa master), för ett nätverk med en annan PTP-master. Kräver att den är igång. `time-sync set-priority`. |

Var och en tar `backend_url=` och `timeout=` och startar en lokal backend som inte körs.

---

## Kameraverktyg

Vad `chloros-cli lattice` gör utöver att ansluta och spela in, som vanliga funktioner i `chloros_sdk` (modul `chloros_sdk.camera_utils`). Var och en anropar den backend-väg som CLI-kommandot använder, med CLI:s regler, så de två fungerar på samma sätt. De tar emot ett kamerans **serienummer**, inte en session, och samma `backend_url=` / `timeout=`-nyckelord som smart-connect-anropen; en backend som inte körs startas, precis som `connect_camera()` gör. Ett avslag genererar `ChlorosConnectError` med backendens orsak.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funktion | Vad den gör |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Skärmens färgprofil: `raw`, `linear`, `natural`, `enhanced` eller `custom_temp`, samt dess färgtemperatur (begränsad till 2000–10000 K; avläses när profilen är `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Skärmens mättnad och kontrast, 0–200 % vardera (100 = oförändrat); `reset=True` återställer båda till 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Dämpa de ljusaste, vitaste pixlarna i skärmbilden: -100..0 (avkortas mot noll, därefter begränsad; 0 = av). |
| `camera_white_balance_once(serial, settle_s=None)` | Engångsvittbalans: kameran strömmar kortvarigt, konvergerar och returnerar `red_ratio` / `blue_ratio` (`ratios_read` False när den inte kan rapportera dem). Den behåller den vitbalans den har beräknat; alla andra inställningar som dess öppna skrivning skickade återgår till de ursprungliga värdena. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (kontinuerlig) eller `"off"`; eller fasta `red` / `blue` balansförhållanden, som stänger av automatikfunktionen först. Ett förhållande som kameran avvisar namnges under `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Kamerans förinställningar, och när en tillämpas: kameran behåller inställningarna som förinställningens namn (svaret är `settings`, eftersom kameran läser av dem efteråt, t.ex. den bildfrekvens den kör med; `None` för ett fält som förinställningen inte namnger), och alla andra inställningar som dess öppna skrivning återgår till det ursprungliga läget. `"default"` återställer kameran till fabriksinställningarna (fabriksinställningarna med fritt löpande automatisk exponering, gränsen för länkgenomströmning och felkorrigering avstängda, en RGB-kameras kalibrerade vitbalans bevarad), som inte sparas i dess minne: svaret innehåller `factory_defaults: True` och alla fält i `settings`. I `camera_presets()` är `default` lika med `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Filtermodellerna och detaljerna för en modell (band, spektralt responskonditionsnummer). En kameras filtermodell läses in från dess modellsträng; den ställs inte in manuellt. |
| `camera_firmware_status(serial=None)` | Varje kamerasfirmware: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` och `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Flasha denna Chloros-versions firmware till en kamera: återställ till fabriksinställningar, flasha, starta om, Chloros-standardinställningarna skrivs om; kamerans kalibrering behålls. Tar flera minuter; kameran måste vara påslagen. `force=True` flashar om en befintlig kamera. |
| `camera_power_probe(serial)` | Kamerans ström- och temperaturnoder samt dess temperatur. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Kamerans strömsparläge (den förblir i viloläge tills den väcks: dess gräns för länkgenomströmning aktiverad (`DeviceLinkThroughputLimitMode` `On`) på lägsta nivå, energisparläget aktiverat där kameran har ett sådant), med temperaturen före/efter; uppvaknandet återställer länken till det tillstånd den befann sig i vid viloläget (gränsen, dess läge och paketfördröjningen `GevSCPD`, som kameran håller på 0 medan gränsen är aktiverad; uppgifterna sparas på den här datorn, så att även en uppvakning från en senare backend kan hitta dem), eller, för en kamera som denna dator inte har satt i viloläge men som verkar vara i viloläge (dess gräns är aktiverad vid lägsta nivån), med gränsen avstängd och en `GevSCPD` kvar på 0 vid 80 (fabriksinställningarna); en gräns som är inställd över lägsta nivån lämnas som den är. Endast strömtillståndet sparas; alla andra inställningar som deras öppna skrivning avser återställs till det ursprungliga värdet. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Fånga in bildrutor tills ett MAPIR-kalibreringsmål syns, och anpassa koefficienterna per kanal utifrån dess paneler: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Denna dators Ethernet-kort för GigE Vision (MTU, länkhastighet, jumbo-ramar) och eventuella `issues`; `estimate=True` lägger till den bildfrekvens och bandbredd som `cameras`-kameror skulle få. |
| `fix_network_settings()` | Tillämpa GigE Vision-adapterinställningarna på den här datornskameraadapter. Det ändrar adapterns inställningar och fungerar endast när Chloros körs som administratör; i annat fall avslutas programmet med en logg över vad det hittade. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Korrigera vignetteringen i bilderna i en mapp, var och en med sin egen objektivets flatfield-karta: `processed`, `lens_models` (det objektiv som varje fil korrigerades för), `skipped` (med motiveringar). Utan `lens_model` hämtas varje fils objektiv från dess EXIF-modell; en `lens_model` gäller för filer vars modell inte anger någon kamera, och en fil vars modell står i strid med detta hoppas över. Endast filer. |
| `read_dls(mac, filter=None, camera_serial=None)` | En avläsning från en DLS (Bluetooth-sensor för nedåtriktat ljus): spektrum, irradians per band och bandmagnituder genom kamerans egen respons när `camera_serial` anges. `discover_daq_sensors(transports=["ble"])` hittar MAC-adressen. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Värdens hårdvara för kameror och den konfiguration som rekommenderas (`report_text`), samt systemets CPU / GPU / PyTorch-körmiljö. |

- **Vissa behöver kameran för sig själva.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` och `update_camera_firmware()` öppnar kameran. Medan Chloros håller den upptagen (ansluten i appen, med `connect_camera()`, som en del av en matris, eller just nu i bruk av ett annat kommando, till exempel ett arraytest eller en bildtagning som öppnar sina kameror) avvisar backend-servern med HTTP-kod 409 `CAMERA_HELD` (både i `code` och `error_code`), där det anges hur den ska frigöras. Färginställningarna, högdagrarna, `camera_power_probe()` och avläsningarna fungerar på en ansluten kamera.
- **Färg, högdagrar och vitbalans gäller för RGB-kameror.** På en enkelbandsmono-kamera (M3M) ändrar inställningarna ingenting och returnerar `{"skipped": True, "reason": ...}`, precis som CLI. Det är visningsinställningar: livevisningen och bilder på skärmnivå, aldrig strålningsintensitet eller reflektans.

---

## Transienta matriser, justeringsprofiler och index

Vad `chloros-cli lattice` gör utan en permanent anslutning, liksom funktionerna i `chloros_sdk` (modul `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` och `index`. Var och en skickar den begäran som CLI-kommandot skickar, med CLI:s standardinställningar.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **De öppnar kamerorna själva.** Varje kommando öppnar sina kameror, utför sin uppgift och stänger dem, så det krävs kameror som Chloros inte hanterar (anslutna i appen, med `connect_camera()` eller som en arraymedlem); backend avvisar med HTTP-kod 409 `CAMERA_HELD` och anger hur man frigör en. En tillfällig öppning tar en stund (cirka 40 sekunder för några kameror). För upprepade bildtagningar är är `connect_array()` en gång den snabbaste vägen, och den sparar alla exporttyper där en tillfällig bildtagning sparar en debayered bild. `calibrate_alignment_profile()` är undantaget: på en ansluten matris av dessa kameror löser den utifrån matrisens egna bildrutor (`used_live_array`).
- **`serials`** är de kameror som ska öppnas, varav den första utlöser avtryckaren; `None` öppnar alla kameror i nätverket.
- **`settings`** är en ordlista med överskrivningar av kamerainställningar, de fält som CLI:s flaggor ställer in: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` och `preset` (en förinställd kamerainställning med namn). En angiven exponering eller förstärkning stänger av den automatiska kontrollen, precis som flaggorna gör.
- **`gpu`**: `True` tvingar GPU-warp, `False` tvingar CPU (`--gpu` / `--no-gpu`); `None` låter backend välja.
- **`keep_settings`**/**`clear_settings`** (alla funktioner som öppnar kameror, standard är `False`): i slutet återställs varje kamera till det tillstånd den befann sig i; `keep_settings=True` sparar istället körningens inställningar i kamerans eget minne (frikörning), `clear_settings=True` återställer det minnet till fabriksinställningarna — se [Kameraminne](#camera-memory--keep_settings--clear_settings). Svaret (för ett jobb, `stop()`:s) innehåller `camera_memory`.
- **Sökvägarna är backendens**: relativa sökvägar görs absoluta i förhållande till din arbetskatalog när backend-modulen finns på den här datorn. Funktionerna för justeringsprofilfiler (`alignment_profile_info()`, `reorder_alignment_profile()`) läser filen på den här datorn, precis som CLI-gränssnittet gör.

| Funktion | Vad den gör |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Spelar in från en kamera kontinuerligt tills den stoppas. Returnerar en `CliJob`. En `tiff`-körnings bildrutor utgörs av den `raw`-fil som en array-inspelning skriver för den kameran (ett mosaikplan vid sensornsegna värden, okomprimerade); `png` och `jpg` innehåller 8 bitar, så dessa körningar är demosaicerade bildrutor för visning. En bunden eller sammanslagen DAQ skriver en `.daq` per bildruta i körningens`daq/`-mapp (`daq_files` i dess statistik). Varje bildruta skrivs en gång med fullständig metadata (modell, inspelningstid, objektiv, band, kalibrering på en råbild); `stop()` returneras när varje redan insamlad bildruta har skrivits (upp till 25 s), och först då stängs kameran. En kamera som lämnats i läget ”armed” för en hårdvarutrigger (`TriggerMode` `On`) ställs in på friläge för körningen (svarsmeddelandet vid start, `trigger`, anger vad som hittades). Vid stopp återställs kameran till det tillstånd den befann sig i – dess utlösarinställningar och alla inställningar som ändrades under körningen – såvida inte `keep_settings=True`, som sparar körningens inställningar i kamerans eget minne (fritt löpande), eller `clear_settings=True`, som återställer det minnet. En körning som inte spelar in någon bild avslutas automatiskt efter 10 sekunder med `stop_reason` `no_frames` och orsaken anges i `last_error`, och dess `stop()` returnerar den orsaken som `error`. Bilder som tappats bort på grund av att skrivarna inte hann med räknas, och `drop_reason` anger orsaken. För en kamera som Chloros-håller, använd `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Strömma synkroniserade bildgrupper vid `fps`, för `count`-grupper eller tills det stoppas. Returnerar en `CliJob` (dess `start_reply` har `n_cams` och överföringen `pacing`); dess slutliga statistik räknar de grupper som hämtats, skrivits och tappats bort, de filer som skrivaren inte kunde skriva och de grupper som en kamera inte levererade något för. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | En synkroniserad inspelning, en debayered bild per kamera: `results` (`filepath`, `serial`, `frame_id`) och `pacing`. En grupp där en kamera saknas visas, med angivelse av hur många filer som sparades först. `lattice multi-capture` utan ansluten matris. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` synkroniserade bildtagningar med fast exponering: `sync_groups`, `master`, en rad `group_timestamps` per utlösning (en kamera som missade en bild saknas i sin rad) och `sync`: varannan kameras avvikelse från huvudkameran, utlösning för utlösning (`offsets_us`: per kamera `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), på EN klocka (`clock_domain`: `ptp`, eller `device` — PTP avstängt, varje kameras egen drifttidsklocka, så dess avvikelse uppstår vid uppstart och endast jitter mäter synkroniseringen). En utlösare vars tidsstämplar blandar enhetens klocka och PTP (PTP avstängd eller fortfarande låst) utelämnas och räknas (`groups_mixed_domain`), liksom en där en PTP-klocka fortfarande ligger mer än `agree_within_us` från mastern (`groups_clocks_disagree`); `notes` anger detta. `clock_offset_mean_us` / `clock_offset_jitter_us` är det största medelvärdet och jittervärdet per kamera. `success` är falskt, tillsammans med `error`, när en trigger missades, när färre än två triggare har alla kameror på samma klocka, eller när klockorna aldrig stämmer överens (`sync.problems`); raderna returneras oavsett vilket. Den startar PTP-grandmastern om den inte körs (`time_sync` anger `grandmaster_running` och `grandmaster_started_by_this_test`), aktiverar varje kameras PTP (precis som `connect_array()` gör) och väntar, innan den första triggern, på att varje kameras PTP-klocka ska låsa sig till grandmastern och stabilisera sig: `PtpStatus` Slave på en grandmaster, inom 100 us från den och stabil i 3 s, och varje kamera låst på det sättet, utan avbrott, i 120 s — cirka 2 minuter efter strömpåslag. En klocka som just har låst sig avviker med upp till ~200 us i cirka 2 minuter efter att den läser av som Slave, och kamerans egen `PtpOffsetFromMaster` visar inte detta; 120 s bringar kamerorna inom ~10 us från en stabil klocka. En kamera som fortfarande är låst sedan ett tidigare test där PTP lämnades på, enligt vad denna backend såg, räknar från den låsningen, så ett test direkt efter ett annat väntar inte igen; en kamera som testet slår på, eller en som hittas med PTP aktiverat men som denna backend inte har sett låsa sig (som slagits på någon annanstans eller före en omstart av backenden), väntar sina 120 s, och i en uppsättning av båda kamerorna väntar de som just låst sig fortfarande på sin tur. En kamera som lämnar slavläget, överskrider 100 us eller följer en annan grandmaster under 3 avläsningar i rad (1,5 s) under väntetiden börjar räkna om; en eller två sådana avläsningar loggas (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) och räkningen fortsätter, och ett grandmaster-ID som inte kan läsas betraktas som okänt, inte som en förändring. Om alla kamerors räkning startar om samtidigt (inom en sekund) beror det på att grandmasterns tid går framåt, inte kamerornas: backendenslogg visar `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` listas i `grandmaster_steps`, och en väntetid som sedan löper ut inleder sitt felmeddelande med `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Gränsen förlängs inte för detta. `ptp_settle_s` (0–600; `0` för väntan utan stabilisering) anger stabiliseringstiden för körningen; ett annat värde avvisas (HTTP-kod 400). Väntetiden är begränsad till avväntningstiden plus 90 s (210 s som standard, minst 90 s), och anropet väntar i backend-tiden `timeout` s: som standard 240 plus avväntningstiden (360 s), precis som `chloros-mcp`:s `test_array_sync` gör. `ptp_lock` har `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (hur länge varje kamera har varit låst och stabil tillsammans), `found_settled` (kamerorna var redan låsta under stabiliseringstiden när de upptäcktes) och, per kamera, `status`, `offsets_ns`, `slave_s` (hur länge den har varit låst; Ingen: inte låst) och `remaining_s` (Ingen när ingen kameras PTP skulle slås på). Backendens logg har en rad var 5:e sekund med varje kameras tillstånd, hur länge den har varit låst och hur lång tid som återstår. Varje kamera startas med PTP avstängt; därefter synkroniseras klockorna efter cirka 10 sekunder. Därefter återställs varje kamera till det läge som testet fann den i – dess PTP, den automatiska exponeringen och förstärkningen som testet håller avstängda under körningen (`ExposureAuto` / `GainAuto`, samt en manuell `ExposureTime` / `Gain`), och resten, såvida inte `keep_settings=True` – det vill säga en kamera som lämnats med PTP på (`PtpEnable=True`, eller en körning med `keep_settings=True`) förblir låst, och när denna backend har sett att den varit låst under stabiliseringstiden väntar inte ett senare test på den. Om klockorna inte stabiliseras är `success` falskt före någon utlösning, och `error` anger vad som ska göras: en kamera som aldrig har låsts (`not_slave`) innebär att grandmastern inte når den (`time_sync_status()`); en som hela tiden tappade anslutningen namnges efter hur länge den hade varit låst (`remaining_s`); klockor som fortfarande konvergerar (`far`): kontrollera `time_sync_status()`, och för att få dem låsta före testet, aktivera PTP och låt det vara aktiverat (`PtpEnable=True` på varje kamera, eller `keep_settings=True`; testet återställer det som det hittade, så det förblir påslaget), kör sedan testet. `lattice multi-test`. |
| `detect_array_master(line=None)` | Kamerorna på synkroniseringslinjen och en standardmaster: `master_serial`, `slave_serials`, de som `line` har undersökt. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Vilken uppsättning av dessa kameramodeller (produktnamn, `"LATT-M3C-L87-FOCN"`) som skulle dyka upp i den här datorns nätverk: `tier`, `achievable_fps_max`, kabelsatsen (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` och `fix_suggestions`. Som standard visas de upptäckta kamerornas modeller; ingen kamera är öppen. För kameror som du har, med de inställningar som en anslutning skulle använda, se [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Beräkna en inriktningsprofil utifrån livebilder och spara den. `method`: `orb`, `akaze`, `phase`, `checkerboard` (tillsammans med `checkerboard="9x6"`, de inre hörnen) eller `manual` (tillsammans med `control_points={serial: [[x, y], ...]}`, även referenskamerans). `model`: `translation`, `rigid`, `affine` eller `homography`. De mer detaljerade alternativen är CLI-flaggorna: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` är `{serial: {option: value}}`. `rms_threshold_px` vägrar att spara en sämre lösning. Returnerar `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (per kamera `reprojection_error`, `num_matches`) och `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | En justerad multibandsbild genom profilen: `files` (multibands-TIFF-filen med dess `bands` och storlek, plus en per band med `per_band` och en per kamera med `per_camera`) samt `vignette_corrected`. `bands` väljer ut och `order` sorterar banden efter namn (en lista eller en kommaseparerad sträng). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Strömjusterade flerbandsbilder (samma förvrängningsalternativ som `capture_aligned_multiband()`), för `count` eller tills processen avbryts. Returnerar en `CliJob`; dess slutliga statistik `frames_written` och eventuella `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | En profilfil som sammanfattas som `align-info` skriver ut följande: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (per kamera `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (per kamera `band_names`, `wavelengths_nm`, `enabled`); `profile` är hela filen. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Ändra profilfilen på plats: `order` är bandnamn (var och en flyttar upp sin kameras lager, i den ordningen), `enable` / `disable` är serienummer. Returnerar den nya `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Indexförinställningarna (`presets`: var och en `name`, `channels`, `formula`) och färg `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Ett index över en justerad flerbands-TIFF: en `preset` (`NDVI`, `NDRE`, ...) eller en `formula` över symbolerna `channel_map` mappar till band (`{"NIR": "NIR_850", "Red": 0}`, efter namn eller index). En förinställdsymbol `channel_map` utelämnar det enda bandet i filen vars namn börjar med den (`red` -&gt; `Red_660`, `nir` → `NIR_850_RGN`), namngivet av själva filen (dess bandlista eller en Chloros-inspelning/produkts `Chloros:Band{R,G,B}Name`), så NDVI-värden på en Chloros-reflektansbild behöver inget `channel_map`; två band som motsvarar en symbol avvisas, inte gissas. `capture_level` anger vad banden innehåller: `reflectance` eller `radiance` gör värdena kvantitativa; en okalibrerad stapel (`align-apply`:s) ger kvalitativa värden; `"unknown"` läser in det från filen, inklusive en Chloros-fils `Chloros:ProcessingLevel`. Returnerar `stats` (form, 1–99-percentilintervallet, LUT-intervallet, den använda `capture_level`) och den `files` som skrivits bredvid indata (`<input stem>_<preset>.tif`, `..._LUT.png`) eller vid `output`, en sökväg: med en bildfiländelse namnges filerna utifrån dess stam (`output="D:/out/ndvi_FRGN.png"` skriver `ndvi_FRGN.tif` och `ndvi_FRGN_LUT.png`), utan en bildfiländelse är det ett prefix (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); läs in sökvägarna från `files`. De innehåller kamerans registreringsuppgifter (märke/modell/serienummer, GPS, objektiv, tagningstid i UTC, kalibrerings-ID men aldrig dess raw-DN-kalibrering), `Chloros:ProcessingLevel=index`, samt indexet som sitt enda band (`XMP-Camera:BandName`, både TIFF- och LUT-PNG-filerna). Endast filer; ingen kamera. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Ta bild, kalibrera (`reflectance` eller `radiance`; `raw` hoppar över det), justera och beräkna ett index i ett steg; `save_multiband=True` behåller den kalibrerade stapeln, som `compute_index()` kan hämta senare. Returnerar `stats` (tillsammans med `serials` vars band den använde) och `files`. Den öppnar `serials` (den första avfyrar utlösaren), annars profilens kameror vars `layer_order`-post är aktiverad, först referenskameran (öppnas även om dess band är inaktiverade); en kamera vars band är inaktiverade kalibreras aldrig. `channel_map` behövs endast för en symbol som stapeln inte kan svara på, som i `compute_index()`: en förinställningssymbol tar det enda aktiverade bandet vars namn börjar med den, vilket kontrolleras innan någon kamera öppnas (om ingen eller flera svarar avvisas de, och namnges); en anpassad `formula` krävs för varje symbol. `index.tif` och LUT:en innehåller stackens kamera, inspelningstid (referenskamerans exponering, UTC) och objektiv, och båda anger indexet som sitt enda band (`XMP-Camera:BandName`, t.ex. `NDVI`: färg-PNG-filen anger indexet, inte dess färgkanaler). `lattice index --live`. |

### `CliJob`

Det handtag som de tre strömmarna returnerar. Jobbet körs i backend-miljön, som behåller kontrollen över sina kameror tills det avslutas.

| Medlem | Beskrivning |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (rutor sparade / skrivna, borttagna, `fps`, `elapsed`, skrivfel, `stop_reason`; ett kontinuerligt jobb även `frames_grabbed` och `frames_queued` – varje insamlad bildruta skrivs, köas, tas bort eller misslyckas – samt `drop_reason`, `writers` och `write_ms`, den genomsnittliga tiden det tog att skriva en bildruta) samt `output_dir`. `state` är `running`, `ended` (den har stannat av sig själv och har ännu inte stoppats), `stopping` (en stoppad instans skriver ut sin kö: live-statistik, samt `stopped_by`) eller `finished` (dess slutliga statistik, stoppets `error` och `stopped_by`). En post för ett avbrutet jobb sparas i 10 minuter (högst de senaste 32), för varje klient: ett annat skript eller `chloros-mcp` läser samma status och slutstatistik. Ett jobb som startades med `chloros-cli` (`lattice continuous`, `multi-stream`, `align-stream`) stoppas av backend när CLI:n slutar rapportera in i 30 sekunder — avslutat, datorn i viloläge — med `stop_reason`, `client_gone` och `stopped_by` i backend; att läsa dess statistik härifrån håller den inte igång. Ett jobb som startas från SDK:t körs tills det stoppas. Varje start skickar sitt `timeout` som backendens tidsgräns (`client_deadline_s`): en kamera som öppnas efter det (en som fortfarande startar upp) återställs och stängs och inget jobb startas, och anropet genererar `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). En kamera som ett jobb håller kvar nekas till alla andra öppna (`connect_camera()`, ytterligare en start, en tillfällig inspelning) med HTTP-kod 409 `CAMERA_HELD`, där jobbet och den som startade det anges. |
| `stop(timeout=30.0)` | Stoppa det och stäng dess kameror: `final_stats`, `output_dir` och `error` när avslutningen inte gick smidigt. Ett andra anrop returnerar samma svar, liksom ett stopp från en annan klient: ett som skickas medan den första fortfarande skriver ut sin kö väntar på den (upp till 25 s; efter det genereras ett fel, och de slutliga statistiska uppgifterna följer på `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Vänta tills det avslutas av sig själv (när `count` har nåtts, en full disk eller ett fel), och kör sedan `stop()` för att samla in de slutliga siffrorna. `timeout` utlöser `TimeoutError` medan det fortfarande körs; ett jobb utan `count` avslutas först när det stoppas. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Dess id och typ (`continuous`, `multi-stream`, `align-stream`), vart den skriver, svaret från startrutten och om `stop()` har körts. |

Använd den som en kontextmanager för att avsluta den när omfattningen avslutas.

---

## Projektmapp och recept

**Projektmappen** är den mapp där nya projekt skapas, gemensam för Chloros-appen, CLI och SDK (modul `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

En konfigurerad mapp som inte längre finns återgår till standardmappen (`Documents/MAPIR/Chloros Projects`), precis som appen gör. En relativ sökväg som anges för `set_project_folder()` hämtas från din arbetskatalog när backend-miljön finns på den här datorn.

**Ett recept** är den YAML-/JSON-åtgärdslista som `chloros-cli project run PATH RECIPE` kör mot ett sparat projekt. `run_project_recipe()` kör den med CLI:segen körare (installerad med SDK), så ett recept fungerar på samma sätt oavsett om det körs från:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** är en `.yaml` / `.yml` / `.json`-fil eller receptet självt: en ordlista med `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) samt valfritt `overrides` och `stop_on_error` (standardvärde true: körningen avbryts vid den första åtgärden som misslyckas). Åtgärderna är de som anges i CLI-referensen [Recipe DSL](cli-reference.md#recipe-dsl); en relativ `output`-sökväg omvandlas till skriptets arbetskatalog.
- **Returvärden** `ok` (alla åtgärder lyckades), `actions` (hur många), `log` (körarens utdata; `echo=True` skriver också ut den under körningen) och `validation`. En receptfil som saknas eller inte går att tolka ger upphov till `ChlorosConnectError`.
- **Kontrolleras innan den körs.**`validation` är `{"ok", "errors", "warnings", "steps"}`: varje åtgärd måste ha exakt ett känt verb som nyckel (`{"capture": {"name": ...}}`, inte `{"action": "capture"}` eller `{"type": "capture"}`) som namnger en kamera, matris eller sensor som projektet har, av rätt typ. Ett recept med ett fel kör**inga** åtgärder (`ok` false, felen i `log`), oavsett vad `stop_on_error` anger. Oanvända nycklar, samt varje matris eller ljussensor som receptet styr men som inte är ansluten just nu, är `warnings`. Ett recept ansluter inte enheter: anslut projektets första (`ChlorosProject.connect_all()`, `chloros-cli project connect`, eller MCP-verktyget `connect_project_devices`). `dry_run=True` kör ingenting och returnerar `{"recipe": ..., "validation": ...}`.
- Endast ett recept kan köras åt gången i en process. Ett YAML-recept kräver PyYAML.

---

## Projektautomatisering — `ChlorosProject`

Ett sparat Chloros-projekt är en mapp som innehåller `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` laddar manifestet (`relink`: mappen där de saknade källfilerna från ett flyttat projekt nu finns, används när backend laddar det — `process()` — se **Ett flyttat projekt** under `ChlorosLocal`), och `connect_all` kopplar upp alla sparade kameror, matriser och ljussensorer online med sina sparade inställningar — samma hårdvarutillstånd som GUI:n skulle åstadkomma. Ljussensorer ansluts via backendens [DAQ-sensorpool](#daq-sensor-sessions), så de behöver en igångsatt backend (se [`SensorHandle`](#sensorhandle)).

### Minimalt exempel

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Eller som en kontextmanager:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject`-metoder

| Metod | Beskrivning |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Upptäck och anslut alla sparade kameror, matriser och ljussensorer. Returnerar en anslutningsrapport per klass (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, var och en med `connected`, `missing` och `errors`; `arrays` innehåller även `alignment`). Med en backend som lyssnar på `127.0.0.1:5000` hittas och ansluts de sparade kamerorna och matriserna via den. Om ingen finns hittar och styr denna process dem själv via `lattice_sdk`, vilket kräver Arena SDK-runtime och OpenCV (`cv2`) – SDK-hjulet installerar inget av dessa; när det inte går listas varje sparad kamera och matris under `errors` med orsaken angiven, och `missing` förblir tomt. En backend som körs men avvisar SDK:et (utloggad: HTTP 401; en plan under Copper: 403) är inte ”ingen backend”: varje sparad kamera, matris och ljussensor listas under `errors` med backendens orsak, och ingenting öppnas i denna process. Varje anrop kontrollerar backend-servern igen, så `connect_all()` efter inloggning, eller efter att Chloros har startats, ansluter till samma projektobjekt. En sparad matris som redan är igång i backend med samma kameror (GUI:ns eller `chloros-cli project connect`:s) kopplas in som den är, utan att de sparade inställningarna tillämpas på nytt, och `disconnect_all()` låter den fortsätta köras. Ljussensorer går alltid via backend (se [`SensorHandle`](#sensorhandle)). Den startar aldrig en backend. |
| `disconnect_all()` | Avsluta allt. Varje enhet frigörs först; därefter, om backend inte bekräftade frigörandet av en kamera, matris eller ljussensor (avslutningen gick över tiden eller nekades), namnger `ChlorosConnectError` var och en. En enhet som redan är borta, eller där ingen backend körs, är tyst. En matris eller ljussensor som en annan klient hade öppen lämnas kvar att köras. Projektets `with`-avslutning utlöses aldrig. |
| `capture_all(output_dir=".")` | En bildruta från varje ansluten kamera + matris, och ett spektrum (`SensorHandle.read()`) från varje ansluten ljussensor, tagna i tur och ordning (kameror, sedan matriser, sedan sensorer), inte samtidigt. |
| `stream(camera, overlays=False, fps=10.0)` | Generator som genererar BGR `numpy`-ramar från en namngiven kamera (eller matris). `overlays=False` läser ramar med handtaget`grab()` på `debayered`-nivån: genom backend när projektet är anslutet via en sådan, annars en direkt `lattice_sdk`-hämtningsslinga (matriser genererar `{serial: frame}`-ordlistor). `overlays=True` dirigerar via `ChlorosLocal.camera_stream()` → backendens `/api/camera/<serial>/stream-annotated` MJPEG-flöde, där kamerans sparade `ui.overlay`-block skickas vidare som frågeparametrar. Kräver backend-läge, OpenCV (`cv2`) och en **fristående kamera**: en kamera i direktläge ger upphov till `RuntimeError` (backend kan inte hämta en kamera som denna process äger) och en array ger upphov till `NotImplementedError` (överlagrar komposit per kamera — strömmar en medlem efter namn). Motsvarighet för engångskörning: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Kör justering på varje array som för närvarande är ansluten. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Kör kalibrerings-/indexeringspipeline på projektets bilder (omsluter `ChlorosLocal.process`; dessa fyra är de **enda** accepterade kwargs — `indices=` etc. utlöser `TypeError`; ställer in index via `ChlorosLocal.configure()`). Konstruerar en `ChlorosLocal()` på ett lat sätt, vilket automatiskt startar en backend. |

Attribut:
- `proj.cameras` — `Dict[str, CameraHandle]` med nyckel baserad på namn OCH serienummer.
- `proj.arrays` — `Dict[str, ArrayHandle]` med nyckel baserad på namn OCH array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` med nyckel baserad på namn OCH slot_id.
- `proj.config` — `project.json["config"]` ordlista.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Bearbetningsnivåer.** `capture()`, `grab()` och `frame_stream()` använder alla samma `processing`
token, och kedjan är kumulativ — varje nivå kör allt ovanför sig:

| Nivå | Utgång | Anmärkningar |
| --- | --- | --- |
| `raw` | 1-kanals Bayer, sensornativ | Ingen demosaik. Överlagringar är inte tillgängliga på denna nivå. |
| `debayered` | 3-kanals färg (**standard** för `CameraHandle` och alla `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` och `stream()` har som standardinställning `raw`): BGR i minnet (`grab()` / `frame_stream()`), sparas som RGB av `capture()` | I backend-läge, som en inspelning: kalibreringspaketets mörkförskjutning (DSNU) från rådata, därefter OpenCV:s kantmedvetna demosaik — ingen linearitets-LUT, flat field eller unmix. Den enda nivån som fungerar utan backend-läge är kamerabibliotekets vanliga bilineära demosaik (ingen mörkförskjutning). Den sparade TIFF-filen är fotometrisk RGB, så vänd på kanalerna i en bildram i minnet (`frame[..., ::-1]`) för att jämföra den med en fil. |
| `radiance` | float32, W/m²/sr/nm, **ett plan per band** | Fullständig radiometrisk kedja: demosaik + 3×3-avblandning (multispektral) + DSNU + flat-field + NIST-skala, där exponering × förstärkning har dividerats bort så att värdena är absoluta. |
| `reflectance` | uint16, 32768 = 1,0 (eller uint8, 255 = 1,0 — se `bit_depth`), **ett plan per band** | Strålning dividerad med nedåtriktad irradians (ρ = π·L/E). Kräver en DLS/DAQ-avläsning — se anmärkningen nedan. |
| `display` | 8-bitars sRGB-liknande | GUI-motsvarande rendering: CCM + vitbalans + gamma via kamerans aktiva färgprofil. |

Endast i direktläge (utan backend) är `debayered` tillgängligt: `grab()` på en annan nivå aktiverar
`NotImplementedError`, `capture()` ignorerar `processing`, och `frame_stream()` höjer
`RuntimeError` på varje nivå. `reflectance` behöver ett användbart nedåtriktat mätvärde — ramens slutpunkt drar
automatiskt in den sammanslagna DAQ:en i kamerans DLS-plats, men utan någon bunden DAQ vägrar kedjan
reflektansutgången och markerar ärligt nedgraderingen i de returnerade metadata istället för att tyst
lämna tillbaka en sämre produkt.

> **Reflektans DN-skala — kod den inte in.** LATTICE-reflektans använder `32768` = ρ 1,0 och markerar
> XMP `Chloros:PixelScale=32768`; Survey3-reflektans använder `65535` = ρ 1,0 och har inga
> `Chloros:*`-taggar. Läs av taggen och dividera med den. Den är definierad i uint16-domänen, så den förblir
> `32768` för alla format som skalar om (16-bitars TIFF, 8-bitars TIFF, 8-bitars PNG/JPG, 32-bitars procent)
> — normalisera först den lagrade datatypen tillbaka till uint16 (×257 från 8-bitars, ×65535 från float). Det enda
> undantaget är en fil från en version före 1.3.0: en 8-bitars TIFF-fil *beskars* då, inte omskalades, så
> ingen skala beskriver den, och den innehåller ingen `PixelScale`- eller MicaSense-tupel. Behandla en saknad tagg i en
> LATTICE-reflektansfil som ”ingen giltig skala”, inte som ett standardvärde.

**Kalibreringstaggar: endast på `raw`.** `raw`-filen för en inspelning innehåller kamerans kalibreringspaket: nyttolasten `CalibrationJSON` och de MicaSense-kompatibla taggarna (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), som beskriver sensorns råvärden. Varje fil på övriga nivåer (`debayered`, `display`, `radiance`, `reflectance`, samt renderingen `index/`) och varje bildruta från kontinuerlig inspelning eller flera kameraströmmar innehåller kamerans identitet (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) samt bildrutan egen inspelningsinformation, `XMP-mapir:FrameJSON` (exponering, förstärkning, temperatur, pixelformat, inspelningstid, DAQ-matchning), eftersom varje `process()`-produkt bär denna identitet. Bearbetningen hittar kalibreringen för en sådan fil utifrån dess serienummer och `CalBundleSha256` i Chloros kalibreringscache och laddar ner paketet en gång om den här datorn saknar det.

#### Filer skapade från flera kameror

En fil skapad från mer än en kameras band – en multibandstack (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), ett liveindex skapat från flera kameror (`index.tif`, `index_LUT.png`), en bild från en array-inspelning (`composite/`) – namnger varje kamera som ingick i den:

- **Per-kameralistor, i bandordning** (kompositbilden: bakgrundsskikt, sedan förgrund; ett indexskikt, kamerorna som banden kommer från): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (det kalibreringspaket som varje kameras band har genomgått, annars — en stapel med `capture_aligned_multiband()` / `start_aligned_stream()`, vars band inte har genomgått något — det kamerapaket som Chloros har cachelagrat, det som dess bildfiler är namngivna efter), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — parallella listor, där post *n* i varje beskriver samma kamera; `-` där en kamera saknar värde (inget kalibreringspaket, ett okänt objektiv). Varje kamera är **en** post: om dess paket har flera värden för ett fält — ett pakets halogenlampcertifikat — de sammanfogas med `;` (`F-1479-1;F-1480-1;F-1481-1`), aldrig med kommatecken, så en lista visar en post per kamera oavsett hur den skrivs ut.
- **EXIF `SerialNumber`** innehåller samma serienummer, separerade med kommatecken (`214000535,214000551`).
- Den innehåller **ingen** enskild kameras identitet (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): dessa namnger en kamera och dess paket, och bearbetningen skulle bestämma filens kalibrering utifrån dem. Ett index beräknat från flera kamerors band (`index.tif`, `index_LUT.png`) har EXIF-värdena `Model` och `LATTICE` (riggen), inte en enskild kameras `LATT-…`-modell.

En fil med **en** kameras band – en TIFF-fil per band, en `capture_aligned_multiband(per_camera=True)`-bild, en stapel eller ett liveindex från en enskild kamera – innehåller den kamerans egna identitetsetiketter, precis som alla andra inspelningsfiler gör. `capture_aligned_multiband()`:s stapel, bandvisa och kameravisa filer, samt varje `start_aligned_stream()`-stapel och dess bandvisa filer, innehåller gruppens inspelningstid (referenskamerans exponering, UTC), och en monokameras `per_camera`-bildruta är ett plan, vid `bit_depth` precis som stapeln.**Kamerans orientering (spegelvändning/rotation).** Bilder från `grab()` / `frame_stream()` har alltid
kamerans ursprungliga orientering, på alla nivåer. Sparade bilder följer kamerans orienteringsinställning
(GUI:ns Mirror H / Mirror V / rotation):

- **`raw`- och `debayered`-filer är exakt vad sensorn avläser**, oavsett inställning – en Bayer-
  mosaik som vrids eller speglas skulle bli ett annat färgfiltermönster, och kalibreringskartorna är
  på sensorns eget rutnät.
- **`display`, `radiance`, `reflectance` och indexfilerna är orienterade** – vridna en gång, efter all
  kalibreringsberäkningarna. Deras klämmasker ligger på samma rutnät.
- **Varje bildfil registrerar inställningen** i XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, medurs, efter
  speglingarna) samt `ChlorosOrientationApplied` — `false` på rådata och debayered data, `true` på de orienterade
  produkterna. `ChlorosProcessing` listar de operationer som tillämpats på den filens pixlar (`none` när
  den är i ursprungligt format). Detta inkluderar varje bildruta i en kontinuerlig inspelning (kamerans egen inställning,
  utan tillämpning) och i en ström från flera kameror (identiteten: dess ingående bildrutor är i ursprungligt format, precis som vid en
  array-inspelning).
- **Vid bearbetning av ett projekt tillämpas den inspelade orienteringen** på de produkter som skapas från en rå
  (först kalibrering i sensorkoordinater, därefter matrisjustering och slutligen orientering),
  så att de stämmer överens med produkterna från liveinspelningen — inklusive de okalibrerade sensorrespons- och vignettkorrigerade
  reservprodukterna; dess debayered-produkt förblir i ursprungligt format.
- **Chloros-bildvisaren visar en rå- eller debayered-fil som har roterats enligt dess inspelningsinformation** — endast för visning,
  filen ändras inte; en fil utan inspelningsinformation visas som den är lagrad.
- **Inspelningar utan inspelningsinformation bearbetas som ursprungliga.** Före version 1.3.0 roterade en inspelning med en enda kamera
  själv sina RAW- och debayered-filer; Chloros kan inte skilja en sådan RAW-fil från en originalfil och
  gissar inte.
- **Array-inspelningar behåller varje medlemsfil i originalformat som standard** och registrerar identiteten;
  `ArraySession.capture(orient=True)` bearbetar deras produkter (se kontrollerna för array-upptagningar).

#### Utdata per band och blandade matriser

På nivåerna `radiance` och `reflectance` returnerar en kamera **ett plan per band för den
kameran** — tre för en M3C (dess tre oblandade band, BGR-kanalordning), ett för en M3M (dess
enda smalbandsfilter). Monokamerans replikerade plan kollapsas, så en **blandad
M3C/M3M-matris ger olika matrisformer per serie**. Läs av nyckeln `bands` från bildens
metadata för att ta reda på vad varje plan är; dra inga slutsatser utifrån antalet kanaler:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Kontrollera alltid `hdr["processing_level"] == "reflectance"` innan du använder värdena. En nedgradering
(ingen DAQ-avläsning, inget kalibreringspaket, okänd bandlayout) rapporteras i `processing_level` och
`radiometric_refused` istället för att uppgraderas – den matris du får tillbaka är fortfarande en giltig ram, bara
inte den produkt du bad om. En sparad inspelning som har nedgraderats har samma kod i sin XMP
som `Chloros:RadiometricRefused` (t.ex. `no-band-layout`, `no-dls-snapshot`).

#### En trigger per avläsning — `group_id`

**Arraymedlemmar hämtas från arrayens synkroniserade bildgrupp, inte från varje kameras egen
hämtningsbuffert.** Kamerorna i en hårdvarusynkroniserad array exponerar tillsammans, men N avläsningar per kamera
*samplar* inte tillsammans — avläsningen av varje kameras senaste buffrade bild sträcker sig över utlösarens
gränserna, och en enda utelämnad bildruta avsynkroniserar uppsättningen utan att någon märker det.

Att leverera från gruppen är nödvändigt men inte tillräckligt i sig: varje begäran tar fortfarande
en *snapshot* av *den senaste* gruppen vid sin egen tidpunkt, och hämtningsslingan kan publicera däremellan.
Mätt på en live-matris med 4 kameror sträckte sig **12 av 20 parallella läsningar över två grupper** — alltid
exakt intill varandra. Så läsningen är **fastlåst**:

1. Varje bildruta bär `group_id`, uppsättningens monotona publiceringsräknare.
2. `grab()` läser av mastern (eller, om mastern inte kan betjänas, nästa kamera som kan),
   och fäster sedan de återstående kamerorna till det grupp-ID den hamnade på (`?group_id=N` på HTTP-
   slutpunkten).
3. Backenden behåller de senaste grupperna i **minst en halv sekund**, så en grupp som redan
   har ersatts av senare publiceringar betjänas fortfarande av den utlösaren — tillräckligt länge för att en
   reflektansutmatning att hinna genomföra sin rundtur, även i en fullsensoruppsättning med 9 kameror. En koppling som backend inte
   längre håller kvar avvisas med `409` och `group_aged_out: true` istället för att tyst besvaras
   med en annan utlösare – `grab()` läser om uppsättningen.

Omätning med fastsättning: **20 av 20 avläsningar returnerade en trigger.** `grab()` sköter detta åt dig;
du behöver endast `group_id=` när du anropar HTTP-ändpunkten direkt.

- En kamera som inte kan betjänas i den fästa gruppen – saknas därifrån eller avvisar begäran
  (en `band` som den inte har, en nivå som den inte kan producera) – **utesluts istället för att fyllas i i efterhand**
  från en annan utlösare, och de övriga kamerorna returneras fortfarande.
- Dict:et som `grab()` returnerar är en `GrabResult` (en vanlig `dict`-underklass;
  `from chloros_sdk import GrabResult`): `.errors` mappar varje
  utelämnad serienummer till det undantag som dess läsning orsakade, och `.group_id` är den delade gruppen. En tom
  `.errors` innebär att varje kamera ingår i denna grupp.
- En fristående kamera har ingen grupp, rapporterar `group_id: None` och **avvisar** en `group_id=`
  pin istället för att ignorera den.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Samma räknare publiceras på **varje MJPEG-ström per kamera** hos en medlem i en array —
förhandsvisningen (`/api/camera/stream/<serial>`), den kommenterade förhandsvisningen (`/stream-annotated`) och
reflektansströmmen (`/stream-reflectance`) – som `X-Chloros-Group-Id` på varje del, förutom
`X-Chloros-Frame-Id` och `X-Chloros-Timestamp-Ns`. Strömmarna är kontinuerliga snarare än
punktvisa, så de matchas genom att delar samlas in och grupperas utifrån den rubriken — det finns ingen
pin att ställa in. En fristående kamera har ingen grupp. Se
[MJPEG-strömmar per kamera](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` blir aldrig tyst.** En tick som misslyckas av en orsak som kan åtgärdas av sig själv
hoppas över — en kameraström ger inget resultat för den, en array-ström ger de kameror som
svarade (möjligen inga). Dessa orsaker är: ingen bildruta ännu; kamerans hämtningstråd pausade i en
sekund eller två (`404 not streaming`, vilket backend svarar på när en annan kamera ansluter
eller kopplar bort sig, eller när en inställningsändring ombalanserar poolen); en omstart av backend eller `5xx`; justeringen
håller fortfarande på att stabiliseras (`409 … retry shortly`); och ett avvisande av reflektans eftersom DAQ:en ännu inte har någon avläsning
(`no-dls-snapshot`, en föråldrad eller tom avläsning). Ett avvisande vid nästa tick skulle bara upprepas — ett
okänt `band`, `bit_depth=8` på strålningsintensitet, ett kalibreringspaket utan strålningsskala, ett avvisande av reflektans
med **ingen DAQ alls** (ingen kopplad till kameran eller dess matris, ingen ansluten till
backend: avvisningens `dls_source` är `"none"`), vilken annan `4xx` som helst —
utlöser `chloros_sdk.FrameFetchError` (en `RuntimeError`, som bär `.status` och `.body`) på
en gång. Detsamma gäller **60 sekunder** av utelämnade tickar utan någon bildruta däremellan, oavsett vad `fps` är — tillräckligt
lång tid för att klara av justeringen och en omstart av backend, men tillräckligt kort för att en kamera som
verkligen är borta avslutar strömmen. Det här uppgivningsfelet är också ett `FrameFetchError`, både för en kamera och för
en matris, med den senaste avläsningens eget undantag på `.errors` (och, för en kamera, som `__cause__`). För
en matris betyder ”refused” att varje kamera har avvisats; undantaget `.errors` innehåller varje kameras
orsak.

**En matris `frame_stream()` ger varje utlösare en gång** (både `ArrayHandle` och `ArraySession`). En
tick vars senaste uppsättning är den utlösare som den redan har genererat väntar på nästa utlösare istället för att
generera samma uppsättning igen, så strömmen körs med den lägre av hastigheterna `fps` och matrisens egen bildhastighet
, och på varandra följande uppsättningar delar aldrig en `group_id`. En matris som slutar publicera genererar
`FrameFetchError` (”ingen ny utlösare på 60 sekunder”) istället för att upprepa sin senaste uppsättning. Utan detta skulle en
ström som begärde mer än vad matrisen publicerade – eller startade direkt efter anslutningen – dela ut
upprepningar: i ett mätat test innehöll 20 uppsättningar 11 olika grupp-ID:n.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Exportera ett enda band — `band`

`grab()` / `frame_stream()` använd `band="<name>"` för att exportera endast ett av kamerans band som en
enkanalig `(H, W, 1)`-matris — samma form som en monokamera producerar, så utdata från ett enda band
hanteras på samma sätt oavsett om det kommer från en M3M eller från en uppdelning av en M3C. Ett FRGN:s röda band:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Godkända stavningar är det fullständiga layoutnamnet (`Red_660`, `NIR_850`, `Green_550`) eller dess korta
namn (`red`, `nir`, `green`), oavsett versaler och gemener. Ett okänt namn avvisas och de tillgängliga
namnen listas.

> **Valet görs efter namn, inte efter kanalnummer — avsiktligt.** ”Kanal 1” är `Green_550` i
> matrisens BGR-kanalordning men *röd* i filterkodensbokstavsordning (`RGN`), och de två
> stämmer inte överens. Om man väljer fel enhet får man en fullt trovärdig ram från fel band, vilket
> inget nedströms kan upptäcka. Det finns inget numeriskt index av den anledningen.

#### Justering av strömmarna — `align`

`align=True` förvränger bilden till arrayens gemensamma referensram. Varje kamera förvrängs
genom **samma** profilupplösning och beskärnings-ROI, så att strömmar per kamera som öppnas separat
blir **ömsesidigt registrerade** — du behöver inte begära dem tillsammans:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Krav och beteende:

- Kameran måste ingå i en **hårdvarusynkroniserad** matris (M8-synkroniseringskabel) som har en justeringsprofil
  — samregistrering förutsätter att medlemmarna exponeras samtidigt, vilket en programvaru- eller
  frilöpande array inte tillhandahåller. Avvisas annars med `409`.
- **Avvisas, blir aldrig tyst ojusterad.** Inspelningsvägen kan spara ojusterat med en varning
  eftersom en fil behåller sina härkomsttaggar och kan omregistreras senare; en livestream har ingen
  sådan andra chans, så en saknad profil är ett fel snarare än en reservlösning.
- **Att begära justering startar processen.** Att begära `align` på en array utan profil sätter igång
  kalibreringen och returnerar `409` med vad den väntar på (t.ex. *&quot;väntar på att exponeringen ska
  stabilisera sig på 214000551 (1/3)&quot;*) — försök igen så lyckas det, vanligtvis inom några tiotal sekunder.
  Detta är viktigt vid headless-drift: den automatiska justeringen vid anslutning ger upp efter 30 sekunder, och varje försök
  inom det tidsfönstret hoppas över medan smart-AE fortfarande stabiliseras, så en matris vars exponering
  konvergerar långsamt brukade tidigare sluta med att det inte fanns någon profil för hela sessionen och inget sätt att få
  tag på en utan att öppna den kombinerade vyn på skrivbordet. `alignment_status` rapporterar också
  `ae_align_holdouts`, där det anges vilka kameror som fortfarande stabiliseras och deras framsteg.
- Ge varje ström i en matris **samma `max_h`**, annars hamnar deras beskärningar på olika pixel
  rutnät.
- Justeringen tillämpas *före* kodningen av bitdjupet, så att förvrängningen interpolerar värden med full precision
  .

#### `bit_depth` — byter precision mot bandbredd

`grab()` och `frame_stream()` accepterar `bit_depth=16` (standard) eller `bit_depth=8`:

| `bit_depth` | Reflektanskodning | Kostnad |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, utrymme upp till ρ 2,0 | Full precision. |
| `8` | uint8 som spänner över `0..refl_max` (standard ρ 1,0) | Hälften så många byte; ~0,004 ρ-kvantisering vid standardtaket. Värden över taket klipps av — se nedan. |

##### 8-bitars taket — `refl_max`

`refl_max` anger det ρ-intervall som 8-bitarsområdet spänner över, så att `pixel_scale = 255 / refl_max` och ρ fortfarande är
`pixel / pixel_scale`. `grab()` och `frame_stream()` behandlar det som ett nyckelord på både kamera- och
matrishandtag (`?refl_max=` på HTTP-ändpunkten). Höj det för att få mer utrymme för speglande
högdagrar, med proportionellt grovare steg:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | steg |
| --- | --- | --- |
| `1.0` (standard) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Automatisk exponering kan inte åtgärda en 8-bitars klippning, och att försöka använda den gör bara data sämre.**
> ρ = π·L/E med exponering och förstärkning uträknade, så ρ är en egenskap hos scenen, inte hos
> kamerainställningen — uppmätt på en live-matris ledde en halvering av exponeringen till att medelvärdet för ρ försköts med 1,3 %. Att höja
> exponeringen för att ”anpassa” den till omfånget driver istället pixlarna in i **sensorns** mättnad, vilket
> underskattar L och ger en **för låg** ρ-värde utan att något fel rapporteras (uppmätt: max ρ 1,203 → 0,985 vid
> 2× exponering, medelvärde −23 %). Välj omfånget med `refl_max`; att hålla sensorn borta från
> mättnad är vad AE redan är till för, och det skyddar ρ:s *giltighet* snarare än dess omfång.

**Läs `reflectance_clipped_fraction`, inte flaggan.** `reflectance_clipped_at_1_0` utlöses av en
enda pixel — i en verklig scen rapporterade den `True` medan fraktionen var `0.000001`, en pixel
av en miljon. Bråkdelen är det som avgör om taket ska höjas.

**`bit_depth=8` avser endast reflektans.** En begäran om detta för `radiance` avvisas istället för att
tillgodoses: reflektansen har ett fast intervall mellan 0 och 1 som klarar en 8-bitars kodning, medan strålningsintensiteten inte har något
fast intervall, så för att få plats i 8 bitar krävs en percentilstäckning per bildruta vars mappning
ändras för varje bildruta — en bild, inte en mätning.

#### Live-ström av reflektans (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` levererar 8-bitars reflektans per band som
`multipart/x-mixed-replace` — en M3C som 3-kanals JPEG, en M3M som gråskala. Förfrågningsparametrar:
`fps` (standard 10), `jpeg_q` (standard 95), `max_h` (0 = inbyggt), `band` (ett namngivet band, enligt
ovan), `align` (`1` för samregistrering – samma regler som SDK:s `align`), `refl_max` (det 8-bitars
kodningstaket, som ovan). Varje del innehåller
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` och `X-Chloros-Reflectance-Clipped`.
**Delar från olika kameror som bär samma `X-Chloros-Group-Id` utgör samma utlösare** —
det är så flera strömmar per kamera från en matris sätts ihop igen, och detta garanteras
på serversidan eftersom matrisens medlemmar hämtas från den synkroniserade bildgruppen (se
[En utlösare per läsning](#one-trigger-per-read--group_id)). Den returnerar `409` när reflektansen är
otillgänglig vid anslutningstillfället och avslutar strömmen om den blir otillgänglig under körning, så en avbruten
ström betyder ”ingen reflektans längre” snarare än ”fortfarande okej, bara statisk”.

> **JPEG är förlustrikt — dessa pixlar är en mätning.** Komprimeringsfel påverkar direkt
> reflektansvärdena och är värst vid kanterna, där materialgränserna finns. Kvaliteten
> är därför inställd på 95 som standard. Om du behöver siffrorna intakta, använd
> `grab(processing="reflectance")` / `frame_stream(...)`, som ger samma resultat utan
> någon kodek i sökvägen (och `bit_depth=8` om det är bandbreddskostnaden, inte kodeken, som är begränsningen).

> **EXIF-data överförs till exporten.** `process()` kopierar källbildens GPS-block
> **och dess ExifIFD** till varje produkt, så exporten innehåller `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` och `CameraSerialNumber` samt
> georefereringen. `FocalLength` är det som Pix4D använder för att beräkna markprovavståndet utifrån – utan det
> hamnar rekonstruktionen i en helt felaktig skala (i ett uppmätt fall förvandlades en plats på 411 m
> till en på 47,8 km). Kopian är medvetet inte `-all:all`: IFD0:s strukturella taggar bryter
> LATTICE-utdata, och `ExifImageWidth`/`Height` utesluts eftersom de beskriver källans
> insamling snarare än den exporterade rasterbilden.

Underflaggor för inspelningsstadiet (gäller de radiometriska nivåerna — `radiance`, `reflectance`, `display`):

| Flagga | Standard | Betydelse |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + NIST-radiometrisk skala. |
| `apply_white_balance` | `True` | WB LUT. DLS-medveten när en DAQ är kopplad till kameran. |
| `apply_index` | `False` | Utvärdering av vegetationsindex. |
| `index_expression` | `None` | Åsidosätt formel. Omtomt → aktiverar indexet automatiskt. |
| `annotated` | `False` | Överlagring av GUI-dekorationer (zebra/rutnät/peaking). Ej tillgängligt för `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Returvärdet är ett `chloros_sdk.CapturePaths`, av typen `CapturePathMap`, inte `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` är `Dict[str, Union[str, List[str]]]`: en en-nivå
> `processing` ger varje serienummer en sökväg, medan `processing="all"` ger den en
> **ordnad lista** över alla produkter som sparats för den kameran. `ArrayHandle.capture()` tar
> inga `levels=`, och den ignorerar nyckelordsargument som den inte känner igen (`levels`, `daq`,
> `render_index`, `force_daq`), så ange endast de som visas här (`aligned=False`
> sparar inbyggda filer, precis som på `ArraySession.capture()`); använd
> `ArraySession.capture()` för dessa kontroller. En live-kombinerad komposit, om en sådan strömmades, anländer under den extra
> `"combined"`-nyckeln snarare än under en serienummernyckel. Kod som utgår från `str` slutar fungera i
> listformen utan att någon typkontroll protesterar — anteckningen angav `Dict[str, str]`
> under en tid efter att listformen släpptes, vilket är anledningen till att aliaset finns. Normalisera
> när du vill ha den platta formen:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Vad som inte kunde skrivas: `.failed`.** Kartan som `capture()` returnerar är en
> `chloros_sdk.CapturePaths` — en vanlig `dict`-underklass, så allt ovan fungerar
> oförändrat — som även innehåller `.failed`: de filer som backend inte kunde skriva,
> samma poster som [`CaptureResult.failed`](#filer-som-inte-kunde-skrivas--captureresultfailed)
> (`serial`, `level`, `error` och `disk_full: True` på en full disk). En partiell
> avläsning returnerar fortfarande alla sökvägar som skrevs; på en full hårddisk är kartan
> tom och `.failed` förklarar varför. `.skipped` listar de utelämnade bilderna per kamera och
> `.group_seq` den synkroniserade gruppen, och `.alignment` / `.oriented` är
> backendens, precis som på `CaptureResult`. `capture_smart()` returnerar dem som
> `result["frames"].failed` och `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Arrayjustering

`ArrayHandle` exponerar hela justeringsytan. Profiler är som standard endast sessionsbundna — anropa `export_alignment()` explicit för att spara dem.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Justering vid anslutning

`connect_all(align=...)` kan automatiskt justera varje array vid anslutning:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Återgår till `project.json["config"]["auto_align_on_connect"]` om inget anges.

### `SensorHandle`

En sparad ljussensor (DAQ-U, DAQ-M eller DAQ-E), öppnad av `connect_all()` i backendens [DAQ-sensorpool](#daq-sensor-sessions) — samma pool som `connect_daq_sensor()` och `chloros-cli daq pool-connect` använder.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` väntar på sensorns **nästa** spektrum — ett som redan finns i bufferten när du anropar räknas inte — och returnerar `(spectrum, is_saturated, integration_time, x, y, z)`, eller `None` om inget anländer inom 5 s. En pausad ström återupptas. `spectrum` anges i W/m²/nm när sensorn är kalibrerad.
- `.sensor` är sensorns [`DAQSensorSession`](#daq-sensor-sessions), för allt annat: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` och `.settings` är den sparade posten från `sensors.json`.

**Vad `connect_all` återställer.** Varje sensor öppnas på sin sparade adress — dess COM-port, Bluetooth-adress eller DAQ-E-värd — och, om projektet har registrerat sensorns ID, måste det vara just den sensorn: en annan enhet på den adressen avvisas och ansluts inte. Dess sparade integreringstid, bildrutsmedelvärde och automatisk exponering tillämpas, liksom dess kapselkorrigering:

| Sparad kap-profil | Ansluten med |
| --- | --- |
| En kap-profil (t.ex. `fov_30`) | Den profilen. |
| Ingen vald | `sunshine_cosine`, standardinställningen vid anslutning, registrerad som antagen (`cap_id_source=auto_default`), vilket ger upphov till en varning vid bearbetningen. Sensorerna levereras utan lock: välj det lock som är monterat. |
| `none` — valt som **Inget (sensor utan lock)** i användargränssnittet | Ingen lockkorrigering. |

En DAQ-E-S tar inte emot någon lockprofil, oavsett vad som sparats: dess diffusor ingår i kalibreringen (se [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Ett projekt som sparats av en tidigare Chloros registrerade en bar sensor som inget val, så den ansluter om med `sunshine_cosine`; välj **Ingen (bar sensor)** i GUI:n ännu en gång och spara projektet.

En sensor som en annan klient redan har öppen i poolen (CLI, ett annat skript) ansluts som den är, utan att inställningarna tillämpas på nytt, och `disconnect_all()` låter den fortsätta köras; en sensor som `connect_all()` öppnade frigörs. Precis som med `connect_daq_sensor()` ingår Chloros-GUI:n **inte** i poolen: stäng en sensor i GUI:n innan du ansluter projektet från ett skript.**Krav och rapporten.** Ljussensorer kräver en igångvarande Chloros-backend som är inloggad på en betald Chloros+-nivå (Copper eller högre), och `connect_all()` startar aldrig någon sådan. Om ingen är igång listas varje sparad sensor under `report["sensors"]["errors"]`, med en angivelse om detta. En sensor som backend inte kan öppna listas också där, med backendens anledning; `report["sensors"]["missing"]` förblir tomt för ljussensorer.

---

## Direkt hårdvara (utan backend)

När du vill ha noll beroende av backend för LATTICE-kameror (CI, huvudlösa robotar, inbyggda), importera `lattice_sdk` direkt — den exporteras på nytt av `chloros_sdk`. Observera följande angående `CAMERA_AVAILABLE`: `lattice_sdk` finns i SDK-hjulet, men kräver att Arena SDK-runtime och OpenCV (`cv2`) finns installerade, och hjulet installerar inget av dessa. Ljussensorer går via backend — [`connect_daq_sensor()`](#daq-sensor-sessions) eller ett projekts `proj.sensors` — med ett undantag, DAQ-M-hjälpfunktionen för bandmagnitud [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), som också kräver `bleak`.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Förinställningar och triggern

Tre av de fyra förinställningarna är **free-run**: kameran exponerar kontinuerligt och en
`capture()` returnerar nästa bildruta. `triggered` är undantaget — den aktiverar
kameran för en hårdvarukant på linje 2, så den tar inga bilder förrän en sådan anländer.

| Förinställning | Utlösare | Använd den när |
| --- | --- | --- |
| `default` | fri körning | allmänt bruk |
| `high_speed` | fri körning | 8-bitars, max 60 fps, kort exponering |
| `high_quality` | frilöpande | 12-bitars, ingen fps-begränsning — det vanliga valet för stillbilder |
| `triggered` | **klar, linje 2** | kameran är ansluten till en M8-synkroniseringskabel och utlöses av något annat |

Om du väljer `triggered` (eller själv ställer in `trigger_mode="On"`) utan att något
styr Line 2, kommer varje `capture()` att gå ut — vilket är korrekt, eftersom du bad
kameran att vänta. SDK:t förklarar detta när det inträffar; se
[SC_ERR_TIMEOUT under inspelning](#direct-hardware-backend-free).

> **Observera — ”GVSP-probe” / `SC_ERR_TIMEOUT -1011`-meddelanden vid anslutning är inga fel.**
> Vid anslutning försöker SDK:t förhandla fram**jumbo-ramar** (9000-byte GVSP-paket) för högre genomströmning. På en direkt punkt-till-punkt-NIC-länk (t.ex. en länklokal `169.254.x.x`-adress) kan nätverket vanligtvis inte hantera jumbo-ramar, så denna sond går ut och loggar rader som:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Detta är den **avsedda reservlösningen**: SDK:n återgår automatiskt till standardpaket på 1 500 byte och kameran fortsätter att ansluta som vanligt (de följande `[chunk-enable …]`-raderna ingår i den normala anslutningssekvensen). Bildtagningen fungerar fortfarande.
>
> Du kan hoppa över denna sond, men **den är inte bara till för att dämpa loggningen – den stänger av jumbo-ramar.** Kameran svarar på ”Don&#x27;t-Fragment”-pingar endast upp till 1500 byte, oavsett hur bra ditt nätverk är, så pingtestet i sig kan aldrig upptäcka jumbo-ramar; denna sond är det enda som kan göra det. Inaktiverar du den kommer kameran att använda standardpaket på 1500 byte för alltid, på vilket nätverk som helst:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Det lönar sig endast på ett nätverk som du *vet* inte klarar av jumbo-ramar, där det sparar ungefär en sekund i anslutningstid per kamera. Eftersom det handlar om en verklig avvägning snarare än en kosmetisk förändring, anger SDK:t nu detta när du använder funktionen:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Låt den vara om du inte har en anledning.**Om funktionen är aktiverad mäts varje kamera en gång per backend-session och igen när dess länk ändras (en annan adapter, en annan länkhastighet eller en länk som har brutits, som vid byte av switch); vid återanslutning återanvänds resultatet.**Efter en ändring bakom en switch som har varit igång, starta om backenden för att mäta igen.** En första anslutning kan ibland ta några minuter vid denna kontroll, och loggen anger detta när det sker.
>
> Om du *vill* ha jumbo-genomströmning, aktivera jumbo end-to-end (NIC MTU 9000 + en switch som släpper igenom dem), eller lås storleken med `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` när du vet att länken stöder det — men föredra `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` per kommando framför att ställa in det permanent, eftersom en fastställd storlek hoppar över sonden och slutar anpassa sig till nätverket framför den. **Varje** enhet i vägen måste kunna vidarebefordra jumbo-paket – inklusive eventuella PoE-splitters eller -injektorer, vilket oftast är anledningen till att en annars jumbo-kompatibel installation inte kan hantera dem.

> **`SC_ERR_TIMEOUT -1011` under `capture()` / `grab*()` är ett annat problem — det där är ett riktigt fel.**
> Anmärkningen ovan gäller endast `-1011` som loggats av**connect-time probe**. Samma fel som uppstår vid en**capture** innebär att kameran anslöt utan problem men inte skickar några bilder:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Det som avslöjar problemet är en kamera vars *kontroll*kanal fungerar som den ska — upptäckten fungerar, inställningarna och `[chunk-enable …]`-skrivningarna lyckas alla — medan *varje* bildram går över tiden.
>
> **Den vanligaste orsaken är att kameran är inställd på en hårdvarutrigger.** Med `trigger_mode="On"` och `trigger_source="Line2"` sänder kameran ingenting alls förrän en elektrisk flank anländer på M8-synkroniseringskabeln. Om du inte har någon kabel som driver den linjen väntar varje bildtagning i evighet. Kameran är inte trasig och nätverket fungerar som det ska — den gör precis som den ska.
>
> `CameraSettings()` och `default` / `high_speed` / `high_quality` förinställer friläge, och en bildtagning som går ut medan den är aktiverad förklarar sig själv istället för att skriva ut ett naket `-1011`. `PRESETS["triggered"]` aktiverar Line2, enligt design.
>
> För att tvinga vilken kamera som helst till friläge:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Om det fortfarande blir timeout med `trigger_mode="Off"`, levererar kameran verkligen inte data — skicka oss loggen och `ip link show`.

#### Färgprofiler (RGB-förhandsgranskning i realtid) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` väljer skärmens färgprofil för **förhandsgranskningen i realtid** på RGB-kameror (multispektralkameror ignorerar inställningen):

| Profil | Betydelse |
| --- | --- |
| `raw` | Kringgå den radiometriska kedjan helt. |
| `linear` | DSNU + flat + WB, ingen CCM, ingen gamma. |
| `natural` | Linjär + uppmätt CCM + sRGB-gamma, endast med den enkla efterbehandlingen (kromatisk utjämning + desaturering av högdagrar) — den realistiska standardinställningen. |
| `enhanced` | `natural` plus den fullständiga Hub-Parity-finishen (defringe, vibrance, CLAHE lokal kontrast). Rikare utseende till ungefär **dubbla kostnaden per bildruta**, vilket ger en lägre LIVE-bildfrekvens. |
| `custom_temp` | `natural` men vitbalansen låst till `custom_cct_k` Kelvin (DLS ignoreras; begränsad till 2000–10000 K på backend-sidan). |

Profilen är en **endast för live-förhandsgranskning** avsedd reglage för hastighet/utseende: sparade bilder får alltid den fullständiga, rika finishen oavsett vilken profil som valts, så att välja `natural` för att vinna tillbaka bildruts-tid sänker inte kvaliteten på det som sparas på disken. En okänd profil aktiverar `ValueError`; när en chloros-backend är tillgänglig skickas ändringen även via POST till den så att nästa förhandsgranskningsbild återspeglar den (användare av direkt-SDK utan backend får fortfarande inställningsändringen).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Höjdpunkter (RGB-skärm) — `set_highlights`

`LatticeCamera.set_highlights(value)` hämtar de ljusaste, vitaste pixlarna från **skärmen** på RGB-kameror — en lösning på att en ljus vit yta renderas som en platt vit klump. `value` är `-100`..`0` (begränsad); `0` är avstängt, och renderingen blir då exakt som den är utan inställningen. Starkare inställningar sänker skärmens vitnivå (till cirka 60 % vid `-100`) och breddar tonkurvan, så att en yta som förhandsvisningens automatiska ljusstyrka har förvandlat till rent vitt återfår sina detaljer.

Det är en skärmkontroll, men till skillnad från mättnad och kontrast (som endast ändrar liveförhandsvisningen) ändrar den även det du sparar: liveförhandsvisningen och bilder på **skärmnivå** följer inställningen, så även dessa filer blir mörkare (de har inställningen `Chloros:DisplayHighlights`); export av strålnings- och reflektansvärden gör det aldrig, och den kan inte återställa sensordata som har klippts – sänk exponeringen för det. Multispec- och monokameror ignorerar den. Värdet lagras på `settings.highlights` och skickas via POST till en tillgänglig chloros-backend, precis som `set_color_profile`; motsvarigheten i CLI är `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Monokameror (M3M) och `Calibration`

En monokamera av typen **M3M** (`M3M-<lens>-F<wavelength>`) är enkelbandig: ett gråskalplan, ingen Bayer-mosaik, ingen 3×3 spektral-crosstalk-matris. `Calibration` känner igen den och visar en `is_mono`-flagga. Reflektansen gäller fortfarande som en radiometrisk karta per band (avblandningen är identitetsmatrisen), men multibandsberäkningar på en enda kamera ger ett meningsfullt resultat istället för nonsens:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

För att skapa ett vegetationsindex från monokromatisk hårdvara, kombinera flera M3M-kameror med olika våglängder till en justerad multibandsstapel (se [Arrayjustering](#array-alignment)) och beräkna indexet över hela stapeln istället för på en enskild kamera.

#### DAQ-M-bandmagnituder — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` ansluter till en DAQ-M via Bluetooth med sin egen BLE-MAC. Det är den enda ljussensorklassen som kommunicerar direkt med sensorn istället för via backend, så den behöver det som de direkta kameraklasserna behöver (`CAMERA_AVAILABLE`: Arena SDK-runtime och OpenCV) plus `bleak` för Bluetooth (`pip install "chloros-sdk[camera]"` eller `pip install bleak`; utan den genererar `DLS` ett `DLSError`). En DAQ-M som är ansluten någon annanstans (backendens pool, ett projekt, GUI:n) slutar sända ut information, så frigör den där först. **`mac_address` krävs**: det finns inget standardvärde, eftersom varje enhet har sitt eget, och `DLS()` utan ett sådant genererar `DLSError` innan någon anslutning försöks. Hitta MAC-adressen med `discover_daq_sensors(transports=["ble"])` (se [Upptäckt](#discovery--finding-an-address-to-connect-with)) eller `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integrerar ett DAQ-M-spektrum (det senaste mätvärdet när `spectrum` är `None`; `DLSError` om det ännu inte finns något) genom en kameras tre kanalsvar och returnerar `{"red": …, "green": …, "nir": …}` — nycklarna namnger kanalerna i den ordningen; med en RGB-kameras kurvor är den tredje (`nir`) dess blå kanal.

- Utan `spectral_response` använder den filtermodellens standardkurva (den som `filter_model` och `DLS` byggdes med), vilken mättes på en enhet.
- `spectral_response` använder istället kamerans **egna** uppmätta kurvor: en dict `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, som lagras i kamerans kalibreringspaket (`color_correction.spectral_response_smoothed` — RGB-kameror; multispektrala paket innehåller ingen sådan kurva). Om en dict inte kan användas faller systemet tillbaka till flottkurvan.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

CLI:s `chloros-cli lattice dls --camera SERIAL` gör samma sak via backend, genom att hitta kamerans kurva i dess laddade eller cachade kalibreringspaket, och anger vems kurva som användes. Över HTTP är det `POST /api/dls/cli-read` med `mac` (obligatoriskt: utan detta blir svaret HTTP-fel 400, `code` `MAC_REQUIRED`) och `camera_serial` i huvudtexten; svarets `band_magnitudes_source` blir `camera <serial>` när den kamerans egen kurva användes, eller `model <filter>` för flottans kurva (en kamera utan kurva: inte RGB, eller inget paket laddat eller cachat).

---

## Undantag

Fånga basklassen för att hantera ”allt som gick fel i Chloros”:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` och `ChlorosConfigurationError` är attribut till `chloros_sdk` (och för `chloros_sdk.exceptions`) men inte i `__all__`, så `from chloros_sdk import *` tar inte med dem: importera dem efter namn. `ChlorosError`-familjen härstammar från `ChlorosLocal` och de funktioner som bygger på den (`process_folder()`, `process_lattice_capture()`); Smart-Connect-ytan skapar `ChlorosConnectError`.

Hierarki:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Exempel från början till slut

### 1. Bearbeta en mapp med en anpassad förloppsindikator

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Live LATTICE-matris → Reflektans + DAQ-referens

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projektdriven insamlingskampanj

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Bildström från flera kameror → NumPy-pipeline

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skript för inspelning direkt på hårdvaran utan gränssnitt (utan backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Kapacitetskontroll före anslutning av en 4-kameramatris

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Motsvarande inspelningsrecept (ren Python)

`run_project_recipe()` kör ett CLI-recept som det är (se [Projektmapp &amp; recept](#project-folder--recipes)). Skrivet i Python istället är stegen desamma:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## MCP-server (AI-assistenter)

`chloros-mcp` låter en AI-assistent som använder [Model Context Protocol](https://modelcontextprotocol.io) (MCP), till exempel Claude Desktop, Claude Code eller Cursor, styra Chloros: bearbeta bilder (skapa ett projekt, importera en mapp med bilder, välja inställningar, bearbeta, rapportera vad som exporterades vid körningen) och styra LATTICE-kameror, matriser och DAQ-ljussensorer (ansluta, förhandsgranska, ställa in exponering och färg, ta bilder, spela in, uppdatera firmware). Dess verktyg täcker vad detta SDK gör. Du frågar på vanligt språk och assistenten anropar serverns verktyg:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Servern är byggd på detta SDK: bearbetningsverktygen anropar [`ChlorosLocal`](#image-processing--chloroslocal) och hårdvaruverktygen [smart-connect-anropen](#smart-connect-for-lattice-cameras), [DAQ-sessioner](#daq-sensor-sessions), `ChlorosProject` och [kameraverktygen](#camera-utilities), som alla kommunicerar med samma backend som Chloros-appen använder. Dess förfrågningar innehåller `X-Chloros-Client: sdk`, så backend-systemet tillämpar SDK:s inloggnings- och abonnemangskontroller, och SDK:s regler gäller (till exempel nekas Texture Aware för LATTICE-inspelningar).

### Installation

Servern levereras i paketet `chloros-sdk`; det finns inget annat att installera. SDK:t installerar kommandot `chloros-mcp`, vilket är detsamma som `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** Chloros-installationsprogrammet pip-installerar SDK i din Python, vilket placerar `chloros-mcp.exe` i den Python-instansens `Scripts`-mapp.
- **Linux (.deb):** paketet installerar SDK:et för systemet `python3` och placerar kommandot `chloros-mcp` i `/usr/bin`, så att det finns i varje användares `PATH`. I en virtualenv lägger `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` till `chloros-mcp` i virtualensens `bin`.

Servern använder endast standardbiblioteket ovanpå SDK (Python 3.7+). En MCP-klient startar den och kommunicerar med den via stdin/stdout; du kör den inte själv, förutom med `--help`.

### Klientkonfiguration

**Claude Desktop:**lägg till servern till `claude_desktop_config.json` (i Windows: `%APPDATA%\Claude\claude_desktop_config.json`;**Inställningar &gt; Utvecklare &gt; Redigera konfiguration** öppnar den), och starta sedan om Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude-kod:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursor:** samma `mcpServers`-post som för Claude Desktop, i `~/.cursor/mcp.json` (alla projekt; `%USERPROFILE%\.cursor\mcp.json` på Windows) eller `.cursor/mcp.json` i ett projekt.

Alternativen anges i `args` eller efter kommandot för Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). För att använda en viss Python-SDK istället för den som finns i `chloros-mcp` på `PATH`, kör modulen med den Python-versionen (dess fullständiga sökväg som `command` om den inte är den första `python3` på `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

En klient startar servern utan ditt skal, så om den rapporterar att servern inte kunde startas är `chloros-mcp` troligen inte på den `PATH` som klienten ser. Ange istället den fullständiga sökvägen (`where chloros-mcp` på Windows, `which chloros-mcp` på Linux, där .deb-filerna finns `/usr/bin/chloros-mcp`), eller kör modulen med den Python-version som har SDK.

### Alternativ

| Alternativ | Standard | Vad det gör |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Vilken Chloros-backend som ska användas. |
| `--no-auto-start` | av | Starta aldrig en backend: använd en som redan körs, till exempel Chloros-appens, Linux-backend-tjänsten eller en tunnel till en annan maskins (se [Fjärr-backend-läge](#remote-backend-mode-pip-only-host-via-tunnel); mappvägarna måste då peka på samma mapp på båda maskinerna). |
| `--backend-exe PATH` | där Chloros installerar den | Den backend som ska startas när ingen körs, för en Chloros-installation där SDK:et inte letar. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` eller `ERROR`. Loggar skickas till stderr, vilket de flesta klienter visar i sin MCP-logg; stdout innehåller endast protokollmeddelanden. |

### Verktyg

Varje verktygs argument är desamma som för det SDK-anrop som det omsluter; klienten listar vart och ett med sitt fullständiga schema. Sökvägarna är absoluta.

**Bearbetning**

| Verktyg | SDK-anrop | Vad det gör |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Skapar ett projekt och gör det till det öppna projektet, som bearbetningsverktygen arbetar med. Ett vanligt namn skapar mappen i standardprojektmappen; en absolut sökväg skapar den där. |
| `open_project` | `ChlorosLocal.open_project` | Öppnar ett sparat projekt och gör det till det öppna projektet. Sökvägarna för ett flyttat projekt omdirigeras till den mapp det nu befinner sig i; `relink_folder` (absolut) anger namnet på den mapp där de övriga saknade filerna nu finns. Returnerar `source_files` och `image_count`. Inget hittades: en felmeddelande som anger var de befann sig och namnger `relink_folder`; inget är öppet. |
| `import_images` | `import_images` | Importerar Survey3-paren `.RAW`+`.JPG`, LATTICE `.tif`-inspelningar och DAQ `.daq`-inspelningar; returnerar `count` (bilder) och `scan_count` (inspelningar). |
| `configure_processing` | `configure` | Ändrar de angivna inställningarna och inga andra (värden: [Stödda värden](#supported-values)); `custom_settings` skickar en hel konfiguration enligt angivna värden. |
| `get_project_settings` | `get_config` | Läser in de sparade inställningarna för bearbetning, export och indexering för det öppna projektet. |
| `start_processing` | `process(wait=False)` | Påbörjar bearbetningen av det öppna projektet och återvänder omedelbart. |
| `get_processing_status` | `get_status` | Rapporterar `state` (`idle`, `starting`, `processing` eller `finished`) och förlopp; när körningen är klar, `summary`, `export_check_failure` om exporten inte kunde verifieras och `image_failure` om bilder gick förlorade: de övriga produkterna skrivs ut; rapportera körningen som misslyckad. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Skapar ett projekt för en mapp, importerar det, konfigurerar och startar körningen i ett enda anrop (`lattice: true` för LATTICE-inspelningar). Medan en körning pågår avbryts den innan något skapas, och backend-systemet avvisar då även projektändringar från vilket verktyg som helst. |
| `export_light_sensor` | `export_light_sensor` | Skriver kalibrerade `.daq`- och `.csv`-filer för projektets ljussensor utan att bearbeta bilder, och hämtar fabrikskalibreringen från MAPIR-molnet när den inte finns i cachen (se [Ljussensorinspelningar](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Läser en bearbetad bilds Chloros-granskningsmetadata (källa, kalibrering, tillämpade steg). |

**Kameror och matriser**

| Verktyg | SDK-anrop | Vad det gör |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Vad som är anslutet just nu, i valfri klient, var och en med `connected_here`; plus serverns pågående jobb. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Kameror och ljussensorer som kan vara anslutna. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | En kameras, en matris (med dess kapacitet och hälsostatus) eller en ljussensors aktuella status. |
| `analyze_array_network` | `analyze_array_network` | Vad denna värd och detta nätverk kan hantera för en föreslagen array. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Anslut en kamera eller en synkroniserad matris (master först). |
| `connect_project_devices` | `open_project(...).connect_all` | Anslut alla enheter som ett sparat projekt minns, med deras inställningar. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Koppla bort en kamera, en matris eller en ljussensor (se Beteende). `keep_settings: true` på en matris sparar varje kameras inställningar i sitt eget minne, `clear_settings: true` återställer det minnet (dess resultat är `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Projektets enheter, tillsammans. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Läs eller skriv kamerainställningar, med hjälp av ett beskrivande namn eller ett GenICam-nodnamn. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Ställ in en arrayhastighet eller återställ den till automatisk hastighet. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | En bildtagning (`fastest` och `wait_for_exposure` är `capture_fastest` och bilden med fastställd exponering). Om `output_dir` saknas sparas den i det öppna projektets`captures`-mapp, och filerna läggs till i projektet (hålls tillbaka tills en bearbetningsomgång är avslutad); svaret `project_add` bekräftar detta. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Kontinuerlig eller intervallbaserad bildtagning, som ett jobb. |
| `start_array_recording` | `ArraySession.record` / `burst` | En video med kombinerat index eller en rå bildserie, som ett jobb. |
| `build_burst_video` | `ArraySession.build_video` | Bearbeta om en bildserie till kalibrerad video, som ett jobb. |
| `get_job_status` / `stop_job` | | Följ eller avbryt ett jobb. |
| `get_camera_preview` | | Varje kameras liveförhandsvisning, returnerad som en bild. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | En bildruta (eller en synkroniserad uppsättning) på en bearbetningsnivå, utan att spara: statistik per plan och en PNG-fil för varje. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | En matris justering. |

**Ljussensorer**

| Verktyg | SDK-anrop | Vad det gör |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Anslut en DAQ-U, DAQ-M eller DAQ-E (`cap_id` ställer in dess kapkorrigering). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | De senaste avläsningarna: tidsstämpel, mättnad, spektrumområde och toppvärde, samt spektrumet medelvärdesberäknat i våglängdsintervall (`full_spectrum` för varje punkt). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Starta eller stoppa strömmen. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Spela in en `.daq`-fil som ett jobb; `duration_s` avbryter den automatiskt. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Kapsylkorrigeringen (`none` för en bar sensor). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Vilken sensor matar reflektansen för respektive kamera och matris. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Banduppsättningarna samt två strömmande sensorer för omgivningens och objektets reflektans och index. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | En DAQ-E:s egna inställningar, sparade i enheten. |

**Kameraverktyg** ([Kameraverktyg](#camera-utilities))

| Verktyg | SDK-anrop | Vad det gör |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` och deras inställningsfunktioner | En RGB-kameras bildfärg: profil och färgtemperatur, mättnad/kontrast, högdagrar. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Engångsvittbalans, eller automatisk / av / fasta förhållanden. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Kamerans förinställningar. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Filtermodellerna, eller en enskild modell efter namn. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Firmwareversioner och en blixt (ett jobb som standard). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Ström- och värmetillstånd; viloläge eller aktivt läge. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Koefficienter från ett kalibreringsmål i vyn. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Kamerans nätverkskort, en uppskattning och korrigeringen (Endast administratör). |
| `apply_vignette_correction` | `apply_vignette_correction` | Korrigera vinjettering i en mapp med bilder. |
| `read_dls` | `read_dls` | En DLS-avläsning. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Värdens hårdvara och körmiljö. |

**Transientkameror, inriktningsprofiler och index** ([Transientmatriser, inriktningsprofiler och index](#transient-arrays-alignment-profiles--indices)). Varje verktyg här som öppnar kameror använder `keep_settings` / `clear_settings` (standardvärde false): se [Kameraminne](#camera-memory--keep_settings--clear_settings).

| Verktyg | SDK-anrop | Vad det gör |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Spela in en kamera eller strömma flera synkroniserat som ett jobb (`duration_s` avslutar det automatiskt). |
| `capture_cameras_transient` | `capture_cameras_transient` | En synkroniserad inspelning utan att ansluta en array. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Kontroll av utlösarsynkronisering och vilka kameror som ingår i synkroniseringskedjan. |
| `analyze_array_models` | `analyze_array_models` | Vad en uppsättning av dessa modeller skulle uppnå i detta nätverk. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Lös och spara en justeringsprofilfil (`options` innehåller de mer detaljerade lösningsalternativen). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | En justerad flerbandsbild, eller en ström av sådana som ett jobb. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Innehållet i en profilfil; dess bandordning och aktiverade kameror. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Indexförinställningar och gradienter; ett index över en flerbands-TIFF; inspelning till index i ett steg. |

**Tidssynkronisering, projektmapp och recept** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Projektmapp och recept](#project-folder--recipes))

| Verktyg | SDK-anrop | Vad det gör |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | PTP-grandmasterns hälsostatus, dess jämbördiga enheter och varje arraykameras PTP-status. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Starta om den; ställ in dess prioriteringar. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Mappen där nya projekt skapas. |
| `run_project_recipe` | `run_project_recipe` | Kör receptet för ett sparat projekt som ett jobb med dess logg. Kontrolleras först: ett recept med ett fel avvisas och inget körs; varningar visas för anslutna matriser/sensorer (de ansluts inte: `connect_project_devices` först). `dry_run` avslutar kontrollen och kör ingenting. |

Det finns inget verktyg för att öppna ett sparat projekt för bearbetning: varje bearbetningsjobb startar med `create_project` (eller `process_folder`).

### Krav

- **Chloros installerat på samma dator.** Chloros-backendet sköter bearbetningen. Servern kan istället använda ett backend som nås via en tunnel (`--backend-url` med `--no-auto-start`, se [Alternativ](#options)).
- **Inloggad på den datorn**, i Chloros-appen eller med `chloros-cli login`. Servern kan inte logga in åt dig: ett verktyg som anropas när du är utloggad svarar ”Inte inloggad på Chloros” med backendens orsak, och assistenten informerar dig om detta.
- **Vilken betald Chloros+-nivå som helst (Copper eller högre).** Servern har SDK-åtkomst, och den kostnadsfria Iron-nivån har ingen: ett verktyg som anropas på den kostnadsfria nivån svarar ”Inte tillgängligt på denna Chloros-plan” med backendens anledning.
- Python 3.7 eller nyare med SDK, samt en MCP-klient som startar lokala (stdio) servrar.

### Funktionssätt

- **Backenden.** Att starta servern eller visa en lista över dess verktyg startar ingenting; det första verktygsanropet upprättar anslutningen. Om Chloros-appen (eller Linux-backendtjänsten) körs använder servern dess backend och låter den fortsätta köra. Om ingen backend körs startar servern den installerade, utan fönster. När servern avslutas (du avslutar klienten eller den startar om servern) stoppar den de insamlingsjobb och ljussensorinspelningar som den startat och frigör de enheter den anslutit; backenden fortsätter att köras, så att en oavslutad bearbetningsomgång fortsätter till sitt slut, och den stänger av sig själv efter 30 minuter om ingen använder den. Med `--no-auto-start` startar servern aldrig en backend: ett verktyg som anropas när ingen körs ber dig att öppna Chloros-appen, och nästa anrop efter att du gjort det upprättar anslutningen.
- **Ett öppet projekt.** Backend-delen har ett öppet projekt, som delas med Chloros-appen när appen är öppen, så arbeta inte med ett annat projekt i appen medan assistenten är igång.
- **Endast de inställningar som skickas vidare ändras.** `configure_processing` lämnar alla inställningar som inte anges på sina sparade värden (SDK:s `configure()` återställer debayer, vignettkorrigering, reflektanskalibrering, PPK och exportformat till standardvärdena när de inte anges; servern fyller i dem från projektet). Om den inte kan läsa projektets sparade inställningar ändrar den ingenting och meddelar detta, såvida inte alla fem anges. `indices` ersätter projektets indexlista. Resultatet listar vad som skickades som `settings_sent`, och namnger under `warnings` alla index som kamerans band inte kan skapa.
- **Bearbetningen sker enligt principen ”start-then-poll”.** `start_processing` återvänder så snart körningen startar, vilket sedan tar från några minuter till flera timmar, och assistenten följer upp med `get_processing_status`. `state` är `idle` (ingen bearbetning), `starting` (backend har ännu inte rapporterat körningen, i upp till 20 sekunder), `processing` eller `finished`, med körningens [sammanfattning](#post-run-summary--hints): vad som exporterades, samt tips när mindre än förväntat exporterades (inga bilder, inget kalibreringsmål hittades). Produkterna sparas i projektmappen, precis som i appen. `start_processing` avvisas medan en körning pågår, och en backend som slutar svara rapporteras som ett fel, inte som en avslutad körning.
- **Vad assistenten ser.** SDK:s varningar visas i verktygsresultatet under `warnings`, och fel (utloggad, plan, felaktig sökväg) visas som feltext som den kan läsa och vidarebefordra till dig. En lista med fler än 20 poster beskärs till de första 20, med den fullständiga längden angiven som `<key>_total`. Sökvägar måste vara absoluta.
- **Enheterna delas.** Kameror, matriser och ljussensorer finns i backend, så assistenten kan styra en enhet som är ansluten till Chloros-appen, och de ändringar som görs visas i appen. `disconnect_device` visar vad servern har anslutit; en enhet som en annan klient har anslutit (appen, `chloros-cli`, ett skript, en annan `chloros-mcp`) frigörs endast med `force: true`, vilket är SDK:s `force_disconnect`, och avvisningsnamnen för den som anslöt den när backend-systemet känner till det; en enhet som backend-systemet inte har någon information om alls rapporteras som inte ansluten (`released: false`, `connected: false`, inget att frigöra). En inspelning eller ström som backenden kör leder till att dess `get_job_status`-vy åtföljs av `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (bildgrupper för en ström med flera kameror) — samt en rad med `summary` (&quot;220 bildrutor skrivna, 55 borttagna, 0 misslyckade: ...&quot;), medan den körs och, när den är klar, dess slutliga siffror; en annan klients lista över den (`other_clients_jobs`) innehåller samma uppgifter (`list_cameras()`, `list_arrays()` och `list_daq_sensors()` innehåller varje enhetss `connected_by`: `client`, `label`, `name`, `at`). Varje `chloros-mcp` identifierar sig själv för backend-systemet som `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, skickas som `X-Chloros-Client-Label`; vilket SDK-skript som helst kan ställa in det). Att stoppa en annan klients ljussensorinspelning kräver också `force`.
- **Långvarigt arbete är ett jobb.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, och vid varje begäran om insamling, lösning eller indexering av `background` returneras omedelbart ett `job_id`; `get_job_status` följer därefter och `stop_job` avslutar den. En körning fortsätter tills den stoppas, såvida du inte anger `max_captures`, `duration_s` (eller, för en burst, `max_frames`); `min_free_gb` vägrar att starta när enheten har för lite ledigt utrymme. Ingen gräns tillämpas om du inte anger en. En inspelning eller ström som backend kör (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) behåller backendens jobb-ID, så att en annan `chloros-mcp` — en som startas senare — hittar den via det ID:t: `get_job_status` följer efter den (och listar sådana jobb som `other_clients_jobs`, från `GET /api/camera/cli-jobs`, vart och ett med `started_by`), och `stop_job` avslutar det först med `force: true`. Alla andra jobb (en inspelningskörning, en matrisinspelning eller burst, en bakgrundsinspelning) körs i den `chloros-mcp` som startade det, och endast den servern känner till det: en annan som frågade efter det säger så. Ett stoppat jobb läser `stopping` medan det skriver ut sin kö, sedan `finished` med sina slutliga räknevärden, i 10 minuter och från vilken `chloros-mcp` som helst: den som startade det ser en annans tvingade stopp på samma sätt. `get_job_status` väntar aldrig på ett stopp: under ett (sitt eget `stop_job`, en annan klientseller stoppet som frigör kamerorna från ett jobb som avslutats av sig själv) svarar den omedelbart med `stopping` och de aktuella räknevärdena, och `stop_job` för ett jobb som redan håller på att avslutas meddelar detta. Ett avslutat jobb anger orsaken i `stop_reason`: sin egen orsak (`no_frames`, `disk_full`, `member_missing`, eller för en kontinuerlig körning av en array `count`, `duration`, `array_gone`), `limit reached` endast när dess räknevärden visar dess `count` (eller en bursts `max_frames`, en inspelningens`duration_s`) nåtts, `error`, `stopped` (med `stopped_by`) när en klient avbröt den, annars `finished`.
- **Vart inspelningarna hamnar.** Till det öppna projektets `captures`-mapp, där appen sparar dem, såvida inte `output_dir` anges.
- **Bilder.** `get_camera_preview` och `grab_frames` returnerar bilder som assistenten kan titta på, vid sidan av deras JSON-data. `grab_frames`-bilderna har kontrastjusterats för visning; dess statistik är de verkliga värdena.
- **När servern avslutas** stoppas de jobb den startat och en ljussensorinspelning som den startat stängs. Om servern startade backend-delen frigör den även de enheter den anslutit innan backend-delen stoppas; i appens backend-del förblir de anslutna.
- **Protokoll.** Endast stdio (ingen HTTP-ändpunkt), endast verktyg (inga resurser eller uppmaningar), resultat i form av text och bilder, MCP-protokollversionerna 2024-11-05 till 2025-11-25.**Vad som utelämnas.** In- och utloggning (kontot är ditt; servern kan inte logga in) samt att stoppa en backend som servern inte startade. Allt annat som detta SDK gör med bearbetning och hårdvara hanteras av ett verktyg. (En DAQ-E:s konfiguration – dess namn, autentiseringstoken, OTA-lösenord, signaturkrav, fabriksåterställning – och de direkta DAQ-servrarna tillhör enbart `chloros-cli`, även i SDK: ett felaktigt namn eller token låser ute varje Chloros-installation från enheten.)

---

## Automatisk start av backend

Smart-connect-ingångspunkterna – `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` och `discover_daq_sensors` — är smala HTTP-klienter som förutsätter att en backend lyssnar på `127.0.0.1:5000` (standard-URL för smart-connect-gränssnittet). När GUI:n eller CLI:n redan körs, finns en sådan. Från ett rent skript finns det kanske ingen — så dessa funktioner **startar automatiskt den medföljande backend-binären** (utan fönster, på samma sätt som `ChlorosLocal` gör) innan de anropas för första gången, och väntar sedan upp till 60 sekunder på att den ska starta. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` och `analyze_array_network()` startar aldrig någon.

Regler:

- **Endast en lokal URL-adress startas någonsin.** En `backend_url` som pekar på `localhost` / `127.0.0.1` / `[::1]` är tillåten; alla andra värdar antas vara någon annans maskin och startas aldrig.
- **En backend som avslutas innan den svarar avslutar väntetiden.** Anropet avbryts omedelbart med avslutningskoden, kommandot som kördes och de sista raderna som backenden skrev ut, istället för att vänta ut de 60 sekunderna. Avslutningskod 98 är undantaget: en annan backend har redan porten eller låsfilen (Chloros-appens, backend-tjänsten eller ett annat skript som fortfarande startar), så anropet fortsätter att vänta på att den ska svara. `ChlorosLocal` gör på samma sätt.
- **Backenden lämnas igång för återanvändning** (samma som i CLI:n) – det sker ingen implicit avstängning när ditt skript avslutas. Om du kör skriptet igen återanvänds den aktiva backend-instansen.
- **Välj bort detta med `auto_start_backend=False`** vid något av dessa anrop (t.ex. när du har pekat på en fjärrbackend, eller om du själv hanterar backendens livscykel).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Om den medföljande binärfilen inte kan hittas eller startas, genererar det efterföljande HTTP-anropet ett åtgärdbart, **plattformsanpassat** `ChlorosConnectError`-fel istället för ett vanligt spårningsmeddelande om avvisad anslutning — i Windows hänvisar den till skrivbordsappen eller ett `chloros-cli`-kommando; i Linux (utan GUI) hänvisar den till ett `chloros-cli`-kommando eller `.deb`.

---

## Miljö och rubriker

SDK-paketet märker varje HTTP-anrop till backend med `X-Chloros-Client: sdk`. Backend tillämpar SDK/CLI-licensregler (inloggning **och** ett betalt Chloros+-abonnemang krävs) istället för GUI:ns kostnadsfria nivå. Detta ställs in automatiskt vid import – du behöver inte göra någonting.

Huvudet läggs till i varje `requests`- / `urllib`-anrop vars URL innehåller `localhost` eller `127.0.0.1`, på vilken port som helst – inklusive dina egna lokala tjänster — och ersätter ett värde som du själv har ställt in. Anrop till andra värdar påverkas inte.

Åsidosätt backend-URL:en genom att skicka `backend_url=` (`ChlorosLocal` tar emot `backend_url=` eller `api_url=`; `process_folder()` och `process_lattice_capture()` accepterar endast `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(En installerad backend lyssnar endast på loopback, så en LAN-enhet med `backend_url` kan inte nå den; se [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel) för tunnelmönstret.)

---

## Versionering och kompatibilitet

- SDK-versionen exponeras som `chloros_sdk.__version__`.
- SDK:t är kopplat till den medföljande backend-versionen. Att kombinera ett äldre SDK med en nyare backend fungerar vanligtvis (framåtkompatibla slutpunkter), men att kombinera ett nyare SDK med en äldre backend kan orsaka `404`-fel på nya slutpunkter — uppgradera skrivbordsappen så att den stämmer överens.
- Smart-connect-gränssnittet (`connect_camera` / `connect_array` / `connect_daq_sensor`) och nätverksanalys-ändpunkten returnerar stabila JSON-scheman; nya fält läggs till.

---

## Tips för felsökning

- **`ChlorosAuthenticationError: Login required`** (från `ChlorosLocal`; Smart Connect-anropen ger upphov till `ChlorosConnectError: Login required. …` av samma orsak) → Kör `chloros-cli login EMAIL PASSWORD` en gång på den här datorn, eller logga in via Chloros-skrivbordsappen.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (eller `Failed to create project: …` från `create_project()`) → Långa sökvägar i Windows är inaktiverade på den här datorn (Windows standardinställning), så ingen sökväg får överstiga 260 tecken, och produkterna placeras upp till cirka 130 tecken under projektmappen. Ingenting har bearbetats. Meddelandet anger den längsta projektmappen som passar: använd en kortare, eller aktivera långa sökvägar i Windows och starta om Chloros. Windows-installationsprogrammet erbjuder detta som ”Aktivera långa sökvägar i Windows”, markerat som standard; annars använd grupprincip: *Datorkonfiguration &gt; Administrativa mallar &gt; System &gt; Filsystem &gt; Aktivera långa Win32-sökvägar*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-connect anropar automatisk start av en lokal backend, så detta visas endast när den medföljande binärfilen inte kan hittas/startas (t.ex. en-endast-värd utan skrivbordspaket). Meddelandet är plattformsberoende: i Windows öppnar du skrivbordsappen eller kör valfritt `chloros-cli`-kommando; på Linux kör du ett `chloros-cli`-kommando (det finns inget grafiskt gränssnitt) eller installerar `.deb`. För en fjärrbackend anger du `backend_url=` (och `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** vid import → `lattice_sdk` kunde inte laddas: Arena SDK-körmiljön eller OpenCV (`cv2`), kan inte importeras i denna Python-version. Den backend-drivna ytan (`connect_*`, projiceras via en backend, `ChlorosLocal`) fungerar fortfarande.
- **Array ansluter med full sensorkapacitet men låg bildfrekvens**→ Så är det utformat: auto behåller full sensorkapacitet och låter frekvensen sjunka till vad kabeln klarar. Använd `binning=2` / `4` eller en mindre `width` / `height` för fler bildrutor per sekund, eller aktivera jumbo-ramar / uppgradera länken. Binning och ROI täcker**inte** aggregerad överteckning (`oversubscribed: true`, fps-fält 0): för många kameror för kabeln kan inte åtgärdas med binning/ROI — minska antalet kameror, aktivera jumbo-ramar eller byt till ett snabbare nätverkskort (se [Överteckning](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` rapporterar att nätverkskortets mottagningsring är mycket liten (~0,26 MB) / anslutningsgränser med ”FRAMES WILL DROP”** → Värd-nätverkskortets mottagningsring är inställd på standardvärdet (återställs ofta till 32 efter en uppdatering av nätverkskortdrivrutinen). På ett Realtek USB 10GbE-kort ställer du in `ReceiveBufferLen=256` och `PendingReceives=64` (med utökade behörigheter) och startar sedan om backend så att den läser av ringen på nytt. Fullständig procedur: [CLI-referens → Konfiguration och inställning av värd-NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Värden hänger sig vid omstart/avstängning, senare WMI-fel `Invalid class` / NIC aktiveras inte** → Föråldrad USB 10GbE-drivrutin orsakar `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Uppdatera adapterdrivrutinen till en aktuell version (≥ 2026) och tillämpa inställningarna för mottagningsringen på nytt. Se [CLI-referens → Konfiguration och inställning av värd-NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Array-insamling misslyckas med HTTP-fel 503 ”Ingen ny synkroniserad ramgrupp”** → Arrayen publicerade ingen ny ramgrupp under väntetiden. Kontrollera `arr.status()`: `stalled: True` (eller `streaming: False`) där `actual_fps` 0 innebär att insamlingsslingan har slutat publicera — koppla bort och anslut arrayen på nytt. Om `stalled` är False och `actual_fps` är större än 0, försök igen. Direkt efter `connect_array()` väntar en inspelning redan upp till 20 sekunder på den första gruppen.
- **`status()` visar `link_speed_mbps: 4294`** → Windows kunde inte läsa ut adapterns hastighet; detta är typiskt för USB-Ethernet-adaptrar och utgör inget fel. Se [Förväntade bildhastigheter](#expected-frame-rates).
- **`update_firmware()` misslyckas med `device never connected back on TCP <port>`** → En brandvägg blockerade DAQ-E:s anslutning tillbaka till den här datorn. Windows-installationsprogrammet lägger till regeln (se [DAQ-E-enhetsinställningar](#daq-e-device-settings)); med en annan brandvägg måste du tillåta inkommande TCP-trafik till backend från sensorns subnät.
- **Reflektans avvisad** → En aktiv DAQ måste vara kopplad till kameran (eller sensormatrisen) för reflektans i absolut skala. Koppla antingen via GUI eller använd `processing="radiance"` (W/m²/sr/nm) som inte kräver en kopplad sensor.
- **`smart=True`-insamlingen tar längre tid än förväntat** → AE-konvergensen beror på scenens dynamik. Med `ArrayHandle.capture_smart()`, bredda `exposure_tolerance_pct`, förkorta `stability_window_s` eller sänk `settle_timeout_s` för en snabbare, mindre stabil utlösning.

---

## Se även

- [CLI-referens](cli-reference.md) — varje CLI-underkommando motsvarar ett SDK-anrop.
- [DAQ-ljussensorer](https://mapir.gitbook.io/chloros/daq-light-sensors) i Chloros-handboken — sensorerna, deras gränsvärden och kalibrerade mätområde, registrering, DAQ-E-nätverk.
- [LATTICE-kameror](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) i Chloros-handboken — anslutning, kamerainställningar, bildtagningslägen, matriser.
- Online-dokumentation: `https://mapir.gitbook.io/chloros/api-python-sdk`
