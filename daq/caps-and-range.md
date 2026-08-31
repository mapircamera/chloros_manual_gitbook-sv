# Kåpeprofiler och kalibrerat mätområde

> Själva kåporna – vilka kåpor som levereras med vilka sensorer, hur de monteras och deras optiska egenskaper – beskrivs i **[DAQ-användarhandboken](https://mapir.gitbook.io/daq)**. Denna sida behandlar *angivandet* av det monterade locket till Chloros, vilket är det som gör korrigeringen korrekt.

Varje DAQ-ljussensors radiometriska fabrikskalibrering beskriver den *nakna* sensorn. Det fysiska locket som är monterat över diffusorn förändrar vilket ljus sensorn samlar in, så Chloros tillämpar en fabriksmätt **lockkorrigeringsprofil** ovanpå kalibreringspaketet. Att ange rätt kåpa är en del av att få kalibrerade data – den här sidan beskriver vilka kåpor som finns för varje modell, hur man anger dem och vad sensorns kalibrerade spektralområde faktiskt är.

## Tillgängliga kåpor per modell

| Kåpprofil (`cap_id`) | Fysisk kåpa | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Sunshine-kosinuskorrigeringslock (**standard på alla modeller**) | Ja | Ja | Ja |
| `fov_15` / `fov_45` / `fov_90` | Koner som begränsar synfältet (15° / 45° / 90°) | Ja | — | Ja |
| `fov_30` / `fov_60` | Koner som begränsar synfältet (30° / 60°) | Ja | — | — |
| `none` | Inget lock monterat | — | — | Ja |

Modellspecifika anmärkningar:

* **DAQ-M har en enda lockprofil: `sunshine_cosine`.** ”Bare-plus-Sunshine-cap” är dess produktdefinition, och en DAQ-M utan lock behöver ingen geometriprofil.
* **En DAQ-U utan lock är helt utan lock** — den behöver ingen geometriprofil alls, vilket är anledningen till att det inte finns någon `none`-profil för den.
* **`none` på en DAQ-E är INTE en no-op.** DAQ-E:s infällda, glasöverdragna diffusor har en egen faktisk geometrisk korrigering, så ”ingen kåpa” är i sig en uppmätt profil på denna modell.
* En **nakna DAQ-E kan inte mäta direkt solljus vid någon höjd** — Sunshine-kåpan är fältkonfigurationen. Planera inte utomhusarbete med en nakna DAQ-E.

I GUI:ns inställningar per sensor (kugghjulsikonen på fliken Ljussensorer) erbjuder rullgardinsmenyn **Cap** även alternativet ”None (bare sensor)” på DAQ-U och DAQ-M — på dessa två modeller betyder ”bare” helt enkelt att ingen kåpkorrigering tillämpas, enligt anmärkningarna ovan. Välj detta endast när locket fysiskt har tagits bort.

## Ange lock – och varför det är viktigt

**Det angivna `cap_id` måste stämma överens med det lock som fysiskt sitter på sensorn.** Varken sensorn eller programvaran kan upptäcka vilket lock som är monterat. Angivelsen styr två saker:

1. Den **realtidskorrigering** som tillämpas på varje spektrum.
2. Den **lockstämpel som skrivs in i varje `.daq`-inspelning**, som den efterföljande reflektansbearbetningen förlitar sig på.

Sunshine-kåpan dämpar enligt konstruktion med ungefär **12×**, så om man registrerar med fel angiven kåpa blir spektrumen felaktigt skalade med ungefär den faktorn. Ange omedelbart om kåpan byts ut.

### Ställa in kåpan

GUI: Fliken Ljussensorer → kugghjulsikonen i sensorraden → rullgardinsmenyn **Kåpa**. Standardvärdet för alla modeller är `sunshine_cosine` (alla DAQ-sensorer levereras med kosinuskorrigeraren installerad), och valet sparas med projektet.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (backend måste vara igång):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI accepterar syntaktiskt hela `cap_id`-listan (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); varje profil valideras mot sensorns modell vid anslutning, så ett otillgängligt kap-id (till exempel ett E-only-id på en DAQ-U) ger ett tydligt felmeddelande istället för att korrigera felaktigt. Standardvärdet för backend när inget anges är `sunshine_cosine`.

Python SDK Obs: `cap_id` är **inte** en SDK-reglage — `connect_daq_sensor()` / `DAQSensorSession` exponerar inga kap-parametrar. Välj cap via CLI-kommandona ovan eller rullgardinsmenyn i GUI; se [SDK-referensen](../reference/sdk-reference.md).

Avancerat: profiler levereras med Chloros-installationen på `daq/cap_profiles/<u|m|e>/<cap_id>.json` och kan åsidosättas per användare på `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Oberoende av gränsvärdena får sensorer som aldrig har kalibrerats om automatiskt en liten justering av mörkerförskjutningen baserad på flottans data — utan att användaren behöver göra något.

## Prestanda för solskyddsgränsvärden (utomhuskonfigurationen)

Siffror som du kan basera rutiner på:

| Egenskap | Värde |
| --- | --- |
| Synfält | 180° halvsfäriskt |
| Kosinusresponsfel | ≤ ±4 % upp till 60° infallsvinkel; ≤ ±4,5 % upp till 70° |
| Gräns för lågt stående sol | Rekommenderas inte under ~15° solhöjd |
| Dämpning | ~12× (enligt konstruktion) |
| Repeterbarhet vid återmontering av skyddet | ≈ 1,5 % |
| Kvantitativ irradians | Genomsnitt av **≥ 15 s** av avläsningar (instrumentets egenskap, inte ett fel) |

För alla kvantitativa irradiansvärden — inklusive reflektansreferenser — ska ett genomsnitt av minst 15 sekunders avläsningar användas istället för en enskild bildruta.

## Kalibrerat spektralområde

| Egenskap | Värde |
| --- | --- |
| Spektral samplingsfrekvens | 340–1010 nm i steg om 5 nm (135 punkter) |
| Radiometriskt kalibrerat intervall | **~374–974 nm** (tvingas av programvaran) |

Sensorn rapporterar hela rutnätet 340–1010 nm, men den NIST-spårbara radiometriska förstärkningen sträcker sig över ~374–974 nm. Chloros **avvisar delningen av absolut reflektans** för alla kameraband där mindre än hälften av det spektrala vikten ligger inom detta intervall, och rapporterar orsaken till utelämnandet `dls-uncalibrated-band-<nm>` istället för att generera ett okalibrerat resultat. Bland de kameramodeller som finns i sortimentet är det endast F988-filtret som faller utanför detta intervall; det använder istället arbetsflödet med reflektanspaneler — se [Arbetsflöden för reflektans](reflectance.md).

För sensormodeller, transporttyper och sensor-ID:n, se [DAQ-översikten](README.md). För information om hur cap-stämpeln används under bearbetningen, se [Inspelning och .daq-formatet](recording.md).
