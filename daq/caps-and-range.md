# Sensorkåpor och kalibrerat mätområde

> Själva kåporna — vilken kåpa som levereras med vilken sensor, hur de monteras och deras optiska beteende — dokumenteras i **[DAQ-användarhandboken](https://mapir.gitbook.io/daq)**. Denna sida behandlar *angivandet* av den monterade kåpan till Chloros, vilket är det som gör korrigeringen korrekt.

Varje DAQ-ljussensors radiometriska fabrikskalibrering beskriver den *nakna* sensorn. Det fysiska locket som är monterat över diffusorn förändrar vilket ljus sensorn samlar in, så Chloros tillämpar en fabriksmätt **lockkorrigeringsprofil** ovanpå kalibreringspaketet. Att ange rätt lock är en del av att få kalibrerade data – den här sidan beskriver vilka lock som finns för varje modell, hur man anger dem och vad sensorns kalibrerade spektralområde faktiskt är.

## Tillgängliga lock per modell

| Lockprofil (`cap_id`) | Fysiskt lock | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Sunshine-kosinuskorrigeringslock (**standard på alla modeller**) | Ja | Ja | Ja |
| `fov_15` / `fov_45` / `fov_90` | Koner som begränsar synfältet (15° / 45° / 90°) | Ja | — | Ja |
| `fov_30` / `fov_60` | Koner för begränsning av synfältet (30° / 60°) | Ja | — | — |
| `none` | Inget lock monterat | — | — | Ja |

Modellspecifika anmärkningar:

* **DAQ-M har en enda lockprofil: `sunshine_cosine`.** ”Bare-plus-Sunshine-cap” är dess produktdefinition, och en DAQ-M utan lock behöver ingen geometriprofil.
* **En DAQ-U utan lock är helt utan lock** — den behöver ingen geometriprofil alls, vilket är anledningen till att det inte finns någon `none`-profil för den.
* **`none` på en DAQ-E är INTE en no-op.** DAQ-E:s infällda, glasöverdragna diffusor har en egen verklig geometrisk korrigering, så ”ingen kåpa” är i sig en uppmätt profil på denna modell.
* En **nakna DAQ-E kan inte mäta direkt solljus vid någon höjd** — Sunshine-kåpan är fältkonfigurationen. Planera inte utomhusarbete med en nakna DAQ-E.

I GUI:ns inställningar per sensor (kugghjulsikonen på fliken Ljussensorer) erbjuder rullgardinsmenyn **Cap** även alternativet ”None (bare sensor)” på DAQ-U och DAQ-M — på dessa två modeller betyder ”bare” helt enkelt att ingen kåpkorrigering tillämpas, enligt anmärkningarna ovan. Välj detta endast när locket är fysiskt borttaget.

## Ange lock – och varför det är viktigt

**Det angivna `cap_id` måste stämma överens med det lock som fysiskt sitter på sensorn.** Varken sensorn eller programvaran kan upptäcka det monterade locket. Angivelsen styr två saker:

1. Den **realtidskorrigering** som tillämpas på varje spektrum.
2. Den **lockstämpel som skrivs in i varje `.daq`-inspelning**, som den efterföljande reflektansbearbetningen förlitar sig på.

Sunshine-locket dämpar med ungefär **12× enligt konstruktion**, så om man registrerar med fel angivet lock blir spektrumen felaktigt skalade med ungefär den faktorn. Ange omedelbart om locket byts ut.

### Ställa in locket

GUI: Fliken Ljussensorer → kugghjulsikonen på sensorraden → rullgardinsmenyn **Lock**. Standardvärdet för alla modeller är `sunshine_cosine`, vilket är ett antagande (DAQ-U, DAQ-M och DAQ-E levereras utan kap, och en inspelning anger att kap antogs tills du väljer ett), och valet behålls i projektet.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(backend måste köras):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI accepterar syntaktiskt hela `cap_id`-listan (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); varje profil valideras mot sensorns modell vid anslutning, så ett otillgängligt lock-ID (till exempel ett E-only-ID på en DAQ-U) ger ett tydligt felmeddelande istället för att korrigera felaktigt. Backendens standardvärde när inget anges är `sunshine_cosine`.

Python SDK: namnge cap vid anslutning med `connect_daq_sensor(cap_id="sunshine_cosine")`, eller byt ut den under körning med `DAQSensorSession.set_cap("fov_45")`; se [SDK-referensen](../reference/sdk-reference.md#daq-sensor-sessions).

Avancerat: profilerna levereras med Chloros-installationen vid `daq/cap_profiles/<u|m|e>/<cap_id>.json` och kan åsidosättas per användare vid `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Oberoende av gränsvärdena får sensorer som aldrig har kalibrerats om automatiskt en liten justering av mörkerförskjutningen baserad på flottan – utan att användaren behöver göra något.

## DAQ-E-S: fast diffusor och lutning

**DAQ-E-S** är en DAQ-E med två skillnader som är viktiga för Chloros. Allt annat (upptäckt, anslutning, PTP, inspelning, enhetsinställningar) fungerar exakt som för en DAQ-E.

* **Dess diffusor är permanent, så den kräver ingen kap-korrigering.** Diffusorn satt på enheten när fabrikskalibreringen mättes, så den kalibreringen visar redan den verkliga nedåtriktade irradians. Chloros tillämpar ingen cap-profil på den oavsett vad som begärs: en cap som namngivits för den ignoreras med ett meddelande, dess inspelningar anger att capen kom från modellen, och varningen om antagen cap aktiveras aldrig för den. I Projektinställningar → DAQ-ljussensor visar dess rad ingen cap, och den projektomfattande cap-överskrivningen påverkar den inte. Tillämpa inte en cap på dess data manuellt: Sunshine-korrigeringen ovanpå skulle då visa ett värde som är cirka 11 gånger för högt.
* **Den är utrustad med en 3-axlig accelerometer.** Diffusorns kosinusrespons gäller endast när den är vågrät, så en lutad sensor rapporterar felaktig irradians utan några andra symptom. Varje DAQ-E-S-avläsning innehåller därför dess lutning, rullning och tippning, och en `.daq` som spelats in från en sådan lagrar dessa uppgifter bredvid varje spektrum (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), så att du kan skilja ett moln från ett skakigt stativ.

För information om varje attitydfält och hur ett fel i attitydströmmen rapporteras, se [DAQ-E-S i CLI-referensen](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Sunshine Cap-prestanda (utomhuskonfigurationen)

Siffror som du kan basera dina rutiner på:

| Egenskap | Värde |
| --- | --- |
| Synfält | 180° halvsfäriskt |
| Kosinusresponsfel | ≤ ±4 % upp till 60° infallsvinkel; ≤ ±4,5 % upp till 70° |
| Gräns för låg solhöjd | Rekommenderas inte under ~15° solhöjd |
| Dämpning | ~12× (enligt konstruktion) |
| Repeterbarhet vid återmontering av kåpan | ≈ 1,5 % |
| Kvantitativ irradians | Genomsnitt av **≥ 15 s** mätvärden (instrumentets egenskap, inte ett fel) |

För alla kvantitativa irradiansvärden — inklusive reflektansreferenser — ska ett genomsnitt av minst 15 sekunders mätvärden användas istället för en enskild bildruta.

## Kalibrerat spektralområde

| Egenskap | Värde |
| --- | --- |
| Spektral upplösning | 340–1010 nm i steg om 5 nm (135 punkter) |
| Radiometriskt kalibrerat område | **~374–974 nm** (tvingas av programvaran) |

Sensorn rapporterar hela rutnätet 340–1010 nm, men den NIST-spårbara radiometriska förstärkningen sträcker sig över ~374–974 nm. Chloros **avvisar uppdelningen enligt absolut reflektans** för alla kameraband där mindre än hälften av dess spektrala vikt ligger inom detta intervall, och rapporterar hopporsaken `dls-uncalibrated-band-<nm>` istället för att generera ett okalibrerat resultat. Bland de kameramodeller som finns i sortimentet är det endast F988-filtret som ligger utanför detta intervall; det använder istället arbetsflödet för reflektanspaneler — se [Arbetsflöden för reflektans](reflectance.md).

För sensormodeller, transporttyper och sensor-ID:n, se [DAQ-översikten](README.md). För information om hur cap-stämpeln förbrukas under bearbetningen, se [Inspelning och .daq-formatet](recording.md).
