# Multikamera-uppställningar

En LATTICE **uppställning**består av två eller flera LATTICE-kameror som är anslutna som en synkroniserad enhet. En kamera fungerar som**master**: den avger en GPIO-triggerpuls via en gemensam synkroniseringslinje (standard**Line2**), så att varje enhet exponerar samma ögonblick. Chloros

lägger till PTP-tidssynkronisering, en liveförhandsvisning (rutor per kamera eller en enda justerad multibandskomposit) och synkroniserad bildtagning — varje bildtagning ger en **bildgrupp** där alla kameror delar samma tidsstämpel och bild-ID (rapporteras som `fid:N` i inspelningsutdata).

Det är genom arrayer som monokameror (M3M) genererar vegetationsindex – en kamera bidrar med ett band, och arrayen justerar dem till en multibandsstapel. Se [Monokameror och vegetationsindex](mono-indices.md).

Det finns tre likvärdiga sätt att ansluta en matris, och alla kör samma ”smart-prep”-flöde:

| Yta | Ingångspunkt |
| --- | --- |
| GUI | Fliken Kameror → **Anslut matris** (blå knapp) |
| CLI

| `chloros-cli lattice array-connect --serials SN1,SN2,…` (första serienummer = master) |
| Python

-SDK

| `connect_array(serials=[…])` → `ArraySession` (första serienumret = master) |

Smart-prep utför: en nätverkskapacitetskontroll (ICMP DF-ping + GVSP-kontroll), val av synkroniseringsnivå, en prognos för bildfrekvens vid den begärda bildstorleken, beslut om packad 12-bitars transport (se [Packad 12-bitars transport](#packed-12-bit-transport)), aktivering av PTP, automatisk val av pixelformat per kamera, startvärden för automatisk exponering från varje kameras sparade tillstånd samt konfiguration av GPIO-utlösare på Line2.

**Hela sensorn som standard (nytt i 1.3.0).** Connect behåller den bildstorlek du begärde — hela sensorn på 2048×1536, utan binning och utan beskärning, såvida du inte väljer något annat — och låter bildfrekvensen sjunka till vad värden och nätverket klarar av. Programmet utför aldrig binning eller beskärning på egen hand (i version 1.2.0 aktiverades binning automatiskt vid under 1,5 fps). Om du vill använda binning eller ett mindre ROI är det upp till dig: i anslutningsdialogen väljer du ett alternativ och det beräknade resultatet visar den bildhastighet det ger; `array-connect` visar de snabbare alternativen med deras beräknade hastigheter.

{% hint style="info" %}
Kamerorna måste vara nåbara via länken innan något av detta fungerar — se [Ansluta kameror](connecting.md) för upptäckt, adressering och nedladdning av kalibrering vid första anslutningen. För riggar med flera kameror är inställningarna för mottagningsringen på värd-NIC:n lika viktiga som länkhastigheten; den fullständiga tabellen med symptom→lösningar finns i [CLI

-referens § Inställning och justering av värd-NIC](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialogrutan Array Connect

Fliken Kameror → **Connect Array**öppnar en guiden i tre steg:**Välj → Visningsläge → Inställningar**.

### Steg 1 – Välj master och slavar

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



Dialogrutan skannar nätverket så snart den öppnas (”Skannar nätverket...”) och undersöker sedan synkroniseringskablarna (”Undersöker synkroniseringskablar...”) för att hitta vilka kameror som är anslutna till M8-synkroniseringslinjen. Du behöver minst **2 kameror** för att bygga en array.

M8-synkroniseringskablarna sammanstrålar vid en kopplingspunkt, så **vilken ansluten kamera som helst kan vara master**. Testet förifyllda rollerna – RGB

-kameran (`-FRGB`) som master när den är ansluten, annars den första anslutna kameran, och de övriga anslutna kamerorna som slavar – och visar en av tre meddelanderader:

| Meddelande | Betydelse |
| --- | --- |
| ”Synkroniseringskabel hittad på {n} kameror. Vilken som helst av dem kan vara master; RGB

-kameran (annars den första) är förvald.” (grön) | Proben hittade kamerorna på synkroniseringslinjen; kryssrutorna för master- och slavradio är redan ifyllda. |
| ”Ingen huvudkamera upptäckt – kontrollera GPIO-kabeln” (orange) | Ingen kamera upptäckte en utlösningspuls; kontrollera synkroniseringskablarna. Rollerna är fortfarande förifyllda (RGB

-kameran, annars den första kameran i listan, som huvudkamera) och du kan ändra dem. |
| ”Ingen synkroniseringskabel: {serialnummer}” (orange) | De listade kamerorna har ingen synkroniseringskabel ansluten. |

Kameratabellen har kolumnerna **Kamera / Serienummer / IP / Master (radio) / Slave (kryssruta)**. Raderna listas i**kamerordning** — M3C (Bayer)-kameror före M3M (mono); bland M3C, FRGB, FRGN, FOCN och sedan FNGB; M3M-kameror efter filtervåglängd, med den lägsta först (F450 före F808). Kameralistan i sidofältet och rutnätets standardlayout använder samma ordning.

* Välj exakt **en master**och**en eller flera slavar**. Om du klickar på den aktuella masterns radio igen avmarkeras den. Att välja en annan master görs med ett klick: den nya mastern lämnar slavarna och den tidigare mastern blir en slav (såvida den inte saknar synkroniseringskabel).
* En kamera märkt med **”Ingen synkroniseringskabel”** kan aldrig väljas som slav — en slav utan utlösarkabel skulle vänta på synkroniseringslinjen i evighet och leverera en död bildström. Anslut istället den kameran som en fristående kamera.
* Kameror som redan är anslutna som fristående kameror inaktiveras *inte*: anslutning till en array avslutar den fristående sessionen och öppnar kameran på nytt inom arrayen.

**Nästa: Visningsläge →**aktiveras när en master och minst en slav har valts.**Skanna om** kör om upptäckten och anslutningskontrollen.

{% hint style="warning" %}
**Avbryt** är inaktiverat medan en skanning eller sondering pågår – att avbryta mitt i en sondering kan krascha kamera-SDK

:et på LATTICE-kamerans firmware. Vänta tills snurrikonen är klar.
{% endhint %}

### Steg 2 — Visningsläge

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->



| Läge | Vad du får |
| --- | --- |
| **Separata kameror** | En live-ruta per kamera, alla utlöses samtidigt så att bildrutorna förblir synkroniserade. Varje kamera behåller sin egen färg och sina egna inställningar. |
| **Kombinerade kameror** *(standard)* | En enda ruta som visar den justerade multibands-NDVI

-/indexkompositen. Kamerorna delar matrisens färg. |

Visningsläget påverkar endast hur liveförhandsvisningen presenteras — inspelningsbeteendet är detsamma i båda lägena.

### Steg 3 — Matrisinställningar och det förväntade resultatet

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



När du öppnar den här scenen ber Chloros

backend-systemet om en **rekommendation**och tillämpar den:**full sensor — ingen binning, ingen ROI-beskärning — så länge bildfrekvensen klarar det**, oavsett vilken hastighet värden och nätverket levererar. Endast när ingen hastighet klarar av full sensor minskar rekommendationen bildrutan, och då föredrar den binning framför ROI-beskärning, eftersom binning bevarar hela synfältet. Därifrån byter du upplösning mot bildfrekvens. Varje ändring du gör kör om analysen i realtid och uppdaterar panelen**Förväntat resultat** till höger.

Vänstra kolumnen — inställningar:

| Kontroll | Alternativ | Standard | Anmärkningar |
| --- | --- | --- | --- |
| **ROI (synfält)** | Full (2048×1536) / Halv (1024×768) / Kvart (512×384) | Full | Sensorbeskärning: Halv/kvart-beskärning till ett mindre område med ursprunglig pixelavstånd. |
| **Binning** | 1× / 2× (summa 2×2) / 4× (summa 4×4) | 1× | Hårdvarubinning: 2×2 = fullt synfält till en fjärdedel av överföringskostnaden; 4×4 = fullt synfält till 1/16. Döljs om kamerorna inte stöder binning. |
| **Bild på ledningssidan** (avläsning) | — | — | Bredd×höjd efter binning som faktiskt skickas via ledningen, avrundad till multiplar av 16 (minst 64). |
| **Triggerfrekvens** | 0,5–60 fps, steg 0,1 | tom = auto | Masterns triggerfrekvens. Lämna tomt för att låta Chloros

beräkna det. |
| **Nätverkskapacitet**| 20–2000 MB/s, steg 10 | tomt = auto | Hur mycket värden faktiskt kan hantera, i MB/s —**det enda talet som hela arraytilldelningen hänger på.** Detekteras automatiskt från nätverksadaptern. Sänk det om arrayen rapporterar korrupta ramar: det detekterade värdet överskattar USB-adaptrar och delade switchar. Om du ändrar det körs prognosen om i realtid. Fältet öppnas med den budget som den senaste arrayen var ansluten med (sparad med projektet), så en budget som du har korrigerat en gång behålls. |

Kryssrutan **Pin-upplösning** från version 1.2.0 har tagits bort: anslutningen behåller alltid den bildstorlek du väljer och sänker istället bildfrekvensen.

Höger kolumn — **Prognos**:

* **Synkroniseringsnivå** — ”Samtidig inspelning” (grön), ”Samtidig inspelning (FTD-förskjuten sändning)” (grön), ”Förskjuten inspelning (100 ms avvikelse)” (gul) eller ”Konfigurationen är för stor” (röd). Chloros

väljer nivå: samtidig inspelning när ledningen kan delas med en hastighet som kamerorna kan anpassas till (bildfrekvensen sjunker när varje kameras andel minskar), och förskjuten inspelning endast när detta inte är möjligt — så att en array ansluts till vilket nätverk som helst, med den bildfrekvens som nätverket tillåter.
* **fps-prognos** — den bildfrekvens som kameran förväntas köra med. Vid samtidig inspelning är det ett enda tal: hastigheten anges först, och automatisk exponering fyller den tid den ger, därefter höjs förstärkningen så att en svagt upplyst scen inte sänker den (se [Hur automatisk exponering fördelar exponeringstid och förstärkning](camera-settings.md)). Vid förskjuten inspelning visas det som ett intervall (”mörkt → ljust”), eftersom varje kameras exponering förlänger förskjutningen i en mörk scen.
* **NIC-linje** — länkhastighet och kontinuerlig genomströmning (”NIC {mbps} Mbps · kontinuerlig {N} MB/s”).
* **Sim-emit-burstkontroll** — kan värdens NIC-ring ta emot en samtidig burst från alla kameror (”Sim-emit-burst: X MB · användbar NIC-ring: Y MB ✓/✗”).
* **Kontroll av nätverkskapacitet** — den sammanlagda efterfrågan i stabilt tillstånd jämfört med det kollisionssäkra taket för nätverkskapaciteten (&quot;Nätverkskapacitet: {efterfrågan} MB/s efterfrågad av {n} kameror · tak {tak} MB/s ✓/✗ övertecknad&quot;).
* **”Max antal kameror på denna kabel: {n} — fastställs av bandbreddsgränsen per kamera, så sammanläggning höjer inte detta värde.”** — visas när du är nära (eller överskrider) gränsen för antalet kameror.
* **”BILDER KOMMER ATT FÖRLORAS med dessa inställningar.&quot;**— röd varning med backendens orsak, plus en lista över hinder och blå**förslag på lösningar** (&quot;För att få plats med denna uppsättning i nätverket&quot; / &quot;För att möjliggöra samtidig inspelning&quot;).**Apply &amp; Connect** är spärrad tills en prognos finns, och dess etikett anger varför den nekas:

| Knapptext | Betydelse | Vad som faktiskt hjälper |
| --- | --- | --- |
| ”Analyserar...” | Analysen pågår fortfarande. | Vänta. |
| **”För många kameror för detta nätverk&quot;**| Nätverket klarar inte av arrayen ens med en kamera i taget (sammanlagd kontroll misslyckades): betydligt fler kameror än vad det klarar av, eller andra kameror som strömmar på samma nätverk. | Färre kameror, jumbo-ramar från ändpunkt till ändpunkt eller ett snabbare nätverkskort.**Ett mindre ROI hjälper INTE** — se nedan. |
| **&quot;Minska ROI för att aktivera&quot;** | Bilder skulle tappas bort med dessa inställningar (burst-/ringkontroll misslyckades). | Minska ROI, öka binning eller åtgärda nätverkskortets mottagningsring. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Under anslutningen kan en grön **kalibreringsnedladdningspanel** visas med en förloppsindikator per seriell port: första gången en kamera ansluts till en maskin hämtar Chloros

sitt fabrikskalibreringspaket på cirka 3,8 MB från kameran via GigE (cirka 70 sekunder per kamera). Kameror som finns i cachen visar aldrig denna panel; vid fler än sex kameror rullar raderna. Se [Ansluta kameror](connecting.md).

### Packad 12-bitarsöverföring

12-bitarsformat (`BayerRG12`, `Mono12`) överförs normalt som 16-bitarsord. **Komprimerad**överföring skickar samma 12 bitar i 1,5 byte per pixel och packar upp dem på värddatorn, vilket gör att varje bildruta tar 25 % kortare tid att skicka – vid full sensorkapacitet innebär det ungefär en tredjedel fler bildrutor per sekund.**De sparade uppgifterna är identiska.** Array connect bestämmer detta automatiskt (`auto`): den packar endast när det är fördelaktigt och värddatorn har tillräcklig CPU-kapacitet för uppackningen. Det finns ingen inställning för detta i GUI; från CLI

använder du `array-connect --packed {auto,on,off}` (standardinställning `auto`). 8-bitars- och 16-bitarsformat packas aldrig.

## Bandbredd: hur många kameror som ryms

Hur många kameror en array kan hantera beror på nätverksförbindelsen, inte på Chloros

, så planeringssiffrorna finns i hårdvaruhandboken: **[Planering av arraybandbredd](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Vad Chloros

gör med dem: anslutningsdialogen kör en nätverkssond, beräknar den uppnåeliga bildfrekvensen och väljer en nivå som passar. En array ansluts till vilket nätverk som helst: på en långsam länk anpassas varje kamera till sin andel av bandbredden och bildfrekvensen sjunker, istället för att arrayen avvisas eller att dess bildrutor krymper. Endast en array som bandbredden inte klarar av att hantera ens en kamera åt gången avvisas (**För många kameror för detta nätverk**) istället för att paket tystas bort — se panelen för prognostiserat resultat som beskrivs ovan.

## När bildrutor saknas

En kamera kan saknas i en publicerad grupp av två helt olika skäl,
och de kräver motsatta åtgärder. Chloros

räknar dem separat istället för att rapportera ett
”ofullständigt” tal som inte specificerar något av dem:

| Vad hände | Vad det betyder | Var man ska leta |
| --- | --- | --- |
| **Korrupt**— ramen anlände men var strukturellt felaktig | GVSP-paketförlust på nätverksvägen |**Kabelbudgeten**, NIC:s mottagningsring, jumboramar, switchen |
| **Anlände aldrig**— ingen ram kom alls | Kameran utlöste inte, eller inget lämnade den |**M8-synkroniseringskabeln**, synkroniseringslinjen, om alla enheter är aktiverade |

Fördelningen omvärderas var 10:e sekund medan arrayen strömmar. Över 5 % loggas det
med båda siffrorna angivna, och varje skadad buffert rapporteras första gången det
inträffar per kamera, sedan sammanfattas det en gång per minut så att en lång session förblir läsbar.

**Korrupta bildrutor med noll ”aldrig ankomna” innebär att utlösning och kabelsynkronisering är perfekta**och att varje förlorad bildruta ligger på nätverksvägen. Lösningen är att sänka**Wire Budget** och
ansluta om.

{% hint style="warning" %}
**Att sänka utlösningsfrekvensen hjälper inte mot korrupta bildrutor.** Kamerans
paketrytm ställs in en gång, vid anslutningen. Att sänka utlösningsfrekvensen ändrar hur ofta en burst
inträffar, inte hur snabbt själva bursten skickas ut på kabeln. På en uppmätt rigg med 4 kameror gjorde en
5× sänkning av utlösningsfrekvensen ingen skillnad, medan en sänkning av wire budget från 240 till
200 MB/s fick samma rigg att gå från 10,4 % korrupta bildrutor till noll.
{% endhint %}

En igångvarande array kan inte omplanera sig själv – koppla bort och anslut på nytt så att anslutningstidsväljaren
kan arbeta utifrån den nya bandbreddsbudgeten.

### USB-nätverksadaptrar är begränsade

En USB-Ethernet-adapter anger sin *Ethernet*-länkhastighet, men vad den faktiskt
kan upprätthålla begränsas av USB-bussen och dess drivrutin. En USB 10GbE-dongel brukade tillskrivas
en genomströmning på ungefär 1000 MB/s — ett värde som ingen någonsin hade mätt — och att anpassa
fyra kameror efter det fiktiva utrymmet ledde till att 6–18 % av bildrutorna blev felaktiga, medan arrayen
fortfarande rapporterade en normal målbildfrekvens. USB-anslutna adaptrar är nu begränsade till
**200 MB/s** på Windows

och NVIDIA Jetson, och **400 MB/s** på Linux

x86_64. Gränsen är ett absolut värde snarare än en procentandel, eftersom begränsningen ligger i
bussen: en USB 1 GbE-adapter uppnår cirka 80 MB/s och påverkas inte.

Om din värd verkligen är snabbare än gränsen, höj **Wire Budget** för att ange detta.

## PTP-tidssynkronisering

Bildruts *synkronisering* sker via hårdvarutriggern; **PTP** (IEEE 1588 PTPv2) tillhandahåller jämförbara *tidsstämplar* för alla enheter. Den är aktiverad som standard vid anslutning till arrayen:

* **Chloros

-värdens backend kör PTP-grandmastern**. LATTICE-kameror och DAQ-E-ljussensorer är slavar till den i domän 0, så att bildtidsstämplar och DAQ-spektra hamnar på samma klocka (~1 ms).
* `--no-ptp` (CLI

) inaktiverar det för arbete på testbänken — tidsstämplar mellan kameror är då **inte** jämförbara.
* Kontrollera synkroniseringens status med CLI

:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Fliken ”Kameror” har i sig ingen PTP-indikator; där visas synkroniseringsinformation per kamera i form av de skrivskyddade fälten **Roll**(Master/Slave),**Synkroniseringslinje** och arrayens kapacitetsnivå. DAQ-E:s PTP-status visas i sensordetaljerna på fliken ”Ljussensorer”.

## Live-vyn

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

för matrisen Huvudområdet erbjuder två layouter (växla i den övre fältet): **rutnätsvy**(varje ruta är en cell; dra för att ändra ordning när rutnätets hänglås är upplåst) och**listvy**(rader i full bredd: en**kombinerad**matris utgör en rad – dess sammansatta ruta – medan en**separat**matris ger**varje medlem en egen rad**, med sin egen live-feed och sin egen tabell för kalibreringsmål; under matriserna finns en fristående kamera – den aktiva). Skjutreglaget**Feed Zoom** justerar rutorna; vid en cellbredd under 200 px döljs namn- och fps-överlagringarna automatiskt. Medlemskamerorna visas i sidofältet och i rutnätets standardlayout i kamerordning (M3C före M3M, FRGB först, monokromkameror efter våglängd); om du låser upp rutnätet och drar med musen åsidosätts ändå rutnätets ordning.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**Separat läge** visar en ruta per kamera. Varje ruta visar följande överlagringar:

* kamerans namn (uppe till vänster),
* en **fps-visning** (uppe till höger) – detta är kamerans *verkliga bildhämtningsfrekvens* som rapporteras av backend, inte förhandsgranskningsfrekvensen (liveförhandsgranskningen är begränsad till 30 fps oavsett bildhämtningsfrekvens),
* en statuspunkt — grön (strömning) / gul (laddar) / röd (fel),
* en **spinner för föråldrad bildruta** när ingen ny bildruta har anlänt på 2 sekunder — normalt i ~5 sekunder efter varje anslutning/frånkoppling medan backend-systemet omfördelar bandbredden mellan kamerorna.**Kombinerat läge**visar en enda sammansatt ruta: backend-systemet utför debayering, skalning, justering, brusreducering, konvertering till strålning per band (plus DLS-reflektans när en ljussensor är kopplad), utvärderar matrisens indexuttryck, tillämpar LUT:en och strömmar resultatet som MJPEG. Tills den första justerade bilden renderas visar rutan sitt tillstånd: ”Förbereder array…”, ”Kalibrerar justering…”, ”Väntar på första bilden…” eller – om rutan har slut på försök till automatisk justering (cirka en minut) – ”Justering krävs” med en**Kalibrera justering**-knapp.

Matrisrutan visar även strömningens status via fps-avläsningen:**”Tillämpar inställningar…”**medan en inställningsändring (format, binning, ROI) startar om kamerorna,**”Fastnat”**när matrisen är aktiv men har slutat leverera bildgrupper, och**”Stoppad”**när den har slutat strömma – återanslut matrisen i så fall. När den spektrala avblandningen var tvungen att begränsa ett band som visas i kompositbilden på en stor andel av dess pixlar (25 % eller mer), visar rutan**”Banddata med låg tillförlitlighet · N %&quot;**: indexvärden som baseras på det bandet är tveksamma (vanligtvis har ett band tagit emot för lite ljus).

Användbara fakta om kombinerat läge:

* Kompositbilden är registrerad mot **huvud**kamerans bildruta. AE-ROI-inriktning och punktmätning på kompositbilden är exakta för huvudkameran och ungefärliga för slavkamerorna; använd**Split View** (matrisinställningar → ”Visa medlemskameror”) för pixelexakta rutor per kamera utan att öppna extra kameraanslutningar.
* **Display Layers**(arrayinställningar; avstängt som standard) låter dig välja ett förgrunds- och bakgrundsskikt — valfri medlemskamera eller**Index**. När förgrunden är inställd på Index visar pixlar utanför LUT:s min-/maxvärden bakgrundsskiktet.
* **Renderingsupplösning** (standard 720p) ställer in höjden på livestreamen *och* storleken på den sparade kompositbilden vid export. Bilder per kamera exporteras alltid i full upplösning.
* Justeringen beräknas per session och sparas aldrig. Chloros

justerar **varje array automatiskt efter anslutning, i båda visningslägena**, så snart medlemmarnas automatiska exponering har stabiliserats (programmet fortsätter att försöka i upp till 5 minuter). I avsnittet**Justering**i inställningspanelen för matriser visas RMS-avvikelserna och knappen**Kalibrera om justeringen** – även för separata matriser som inte ingår i en sammansatt bild.

## Inspelning: övervakning vs analys

Arrayernas inspelningsytor delas tydligt upp i **övervakningsklass**(spela in det du ser) och**analysklass** (spela in rådata, kalibrera senare):

| Arbetsflöde | Klass | Vad som sparas | GUI | CLI

|
| --- | --- | --- | --- | --- |
| **Inspelning**(stillbilder) | Analys | En synkroniserad bildgrupp per pass; filer per kamera på varje vald exportnivå (rådata/debayered/radiance/reflektans/förhandsgranskning/index) + `.daq` sidecar |**Knappen **Capture All** + Inställningar för bildtagning | `lattice array-capture` |
| **Spela in indexvideo** | Övervakning | Den kombinerade live-indexkompositen som den visas — 8-bitars, förhandsgranskningsupplösning, inbakad LUT. Från CLI

:n utan öppet GUI-fönster renderar backend-modulen arrayens NDVI

(eller `--expr`) så snart arrayen är inriktad; en inspelning som inte skriver någon bildruta misslyckas | ● Spela in indexvideo (kombinerade matriser) | `lattice array-record` |
| **Rå burst → skapa video**| Analys | Råa sensorbilder med full inspelningshastighet + manifest + `.daq`, därefter offline-rekonstruktion till kalibrerad strålnings-/reflektans-/indexvideo, tidsanpassad till DAQ-avläsningar | ⦿ Spela in rå burst →**Skapa video** | `lattice array-burst` → `lattice array-build-video` |

Tumregel: om pixlarna ska användas för *mätningar*, använd inspelning eller burst (analyskvalitet); om du bara behöver *titta på eller visa* vad sensorn såg, spela in indexvideon (övervakningskvalitet).

### Inställningar för inspelning (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



Kugghjulet bredvid **Capture All** öppnar fönstret med inställningar för inspelning (kräver ett öppet projekt – inspelningarna sparas i det):

* **Inspelningsläge**:**Enstaka**(ett genomgång) /**Kontinuerlig**(i följd; begränsad av antalet inspelningar, standard 1, eller en varaktighet, standard 10 s) /**Intervall** (tidsförlopp: N bilder var X:e intervall för totalt Y, standardinställning 1 var 5:e sekund i 1 minut).
* **Exporttyper per kamera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — alla tillämpliga alternativ är aktiverade som standard. Radiance/Reflectance är dolda för kameror med RGB

-filter; **Reflectance visas endast när kameran har en DAQ-ljussensor** (egen eller ärvd från matrisen); Index kräver ett konfigurerat indexuttryck.
* **Justerad**(per matris, standard**på**): anpassar exportfilerna till matrisens justeringsprofil så att exporten blir pixelregistrerad. Raw-data förblir alltid oförvrängda men bär med sig transformationen i metadata. Matrisens medlemsfiler sparas i varje kameras ursprungliga orientering (eventuella inställningar för spegling/rotation sparas i filen, men tillämpas inte).
* **Snabbast möjliga inspelning** (växlingsknapp): endast raw + den tilldelade DAQ-avläsningen + den fria sammansatta indexkompositen, vilket hoppar över kalibreringsberäkningarna vid inspelningstillfället för maximal hastighet — återuppbygg strålning/reflektans/index senare från den sparade filen `.daq`.
* Valen sparas med projektet. Dolda eller pausade kameror hoppas över.
* **Kontinuerligt** på arrayer körs inuti Chloros

-backend (nytt i 1.3.0): när varje kamera i listan tillhör en array kör backend själva inspelningsslingan och skriver en grupp medan den väljer nästa, utan någon begäran per inspelning. En full hårddisk avbryter körningen (&quot;Stoppad: hårddisk full. Sparade N, F misslyckades&quot;). Se [Inställningar och lägen för inspelning](capture.md#capture-modes) för information från Microsoft Defender om höga inspelningshastigheter.

Motsvarande CLI

-kommando (samma backend-ändpunkt, samma semantik):

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

TIFF

-komprimering för inspelningar är `deflate` (förlustfri, standard) eller `none` — fullständiga flaggtabeller, mappen för inspelningar och regler för ombearbetning finns i [CLI

-referensen](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Koppla ihop en DAQ-ljussensor

För förhandsgranskningar med reflekans- och belysningskorrigering krävs data om nedåtriktat ljus från en DAQ-sensor (ansluten under fliken **Ljussensorer**):

* I **matrisraden**i sidofältet visas en**”DAQ · på/av”-knapp** — *på* när en ljussensor på matrisnivå är inställd **eller** när någon av kamerorna i matrisen har en egen; verktygstipset visar exakt vilken sensor som matar vilken kamera.
* Tilldela för hela arrayen i arrayinställningarna → **Omgivningsljussensor**→ rullgardinsmenyn**Ljussensor**. Valet sparas med projektet, överförs till varje enskild kamera i arrayen, och enskilda kameror kan fortfarande åsidosätta det med sin egen sensor.
* Statusraden nedanför visar det aktuella läget: **Av**→ ”Väntar på första spektrumet…” →**”Aktiv – alla kameror i arrayen är belysningskorrigerade”** → eller, om inget nytt spektrum har kommit in under de senaste 3 sekunderna, ett meddelande om föråldrad data – den senaste avläsningen fortsätter att användas (avläsningar förfaller aldrig i inspelningsvägen).

När en sensor har tilldelats: exporttypen Reflektans blir tillgänglig, liveförhandsvisningar är belysningskorrigerade, den prediktiva automatiska exponeringen kan använda spektrumet, och varje reflektansinspelning skriver den DAQ-avläsning som faktiskt användes som en **`.daq`-fil** i inspelningarnas `daq/`-mapp så att inspelningen kan bearbetas på nytt senare. Bredvid den finns en `.daq.cameras.json`-fil som namnger de kameror vars bildrutor avläsningen beskriver; vid ombearbetning matchar Chloros

a avläsningen endast med bilder från dessa kameror.

## `array-connect`-CLI

-alternativ

| Flagga | Standard | Beskrivning |
| --- | --- | --- |
| `--serials SN1,SN2,…` | upptäck automatiskt alla LATTICE-kameror (kräver ≥2) | **Den första serienumret är MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkroniseringslinje. |
| `--target-fps F` | auto | Master-triggers avfyrningshastighet. |
| `--binning {1,2,4}` | auto | Hårdvarubinning. Auto innebär full sensor (ingen binning) närhelst någon bildfrekvens stöder det; connect använder aldrig binning för att hålla uppe hastigheten. |
| `--packed {auto,on,off}` | `auto` | [Paketerad 12-bitars transport](#packed-12-bit-transport): `auto` packar endast när det är till hjälp och värden har kapacitet att packa upp. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Expertöverskrivning av valet av synkroniseringsnivå. |
| `--wire-ceiling-mbps MB_PER_S` | auto-detected | Värdens bandbreddsbudget i MB/s — CLI

-formen av fältet **Wire Budget**. Sänk den om arrayen rapporterar korrupta ramar. Sparas med projektet, så att en senare återanslutning återställer den. |
| `--no-recommend` | av | Hoppa över nätverksanalyssteget. |
| `--no-ptp` | av | Inaktivera PTP (tidsstämplar mellan kameror är då inte jämförbara). |

`lattice array-list`, `array-status` och `array-disconnect` hanterar den permanenta sessionen. Den fullständiga referensen för underkommandon, inklusive justering (`align-calibrate` / `align-apply`) och nätverksverktygen, finns i [CLI

-referens § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); motsvarande SDK

-kommandon (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) finns i [SDK

-referensen](../reference/sdk-reference.md). Från Python

är trådbudgeten `connect_array(..., wire_ceiling_mbps=120)`, och uppdelningen mellan live-korrupt och aldrig ankommet finns på [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
