# CLI Snabbstart (pool-*)

De medföljande `chloros-cli`-enheterna styr DAQ-sensorerna via **`daq pool-*`** kommandofamiljen — smala HTTP-klienter som styr sensorn via Chloros-backendens permanenta sensorpool. Backend-modulen hanterar transporten, så GUI:t, CLI och SDK-skripten delar alla ett aktivt handtag istället för att konkurrera om porten. Allt en kund behöver är tillgängligt via `pool-*`: anslutning, strömning, inspelning av kalibrerade `.daq`-filer och byte av kap-profiler.

`pool-*` är också den **enda** DAQ-ytan i släppta versioner. `chloros-cli daq --help` listar `pool-*`-underkommandona, och om du anropar ett DAQ-underkommando för direkt hårdvarustyrning i en släppt version avslutas programmet med ett explicit felmeddelande som anger vilket paket som saknas och hänvisar dig tillbaka till `pool-*` — ingenting misslyckas i tysthet. (Kommandona för direkt hårdvaruanslutning körs endast från en MAPIR-källkodsutcheckning; `pip install chloros-sdk` tillhandahåller dem inte heller.)

***

## Förutsättningar

* **Backend-programmet Chloros måste vara igång** — kommandona i `pool-*` är HTTP-klienter, inte hårdvarudrivrutiner. På Windows startar du Chloros-skrivbordsappen (den startar backend). På en headless Linux/Jetson aktiverar du tjänsten: `sudo systemctl enable --now chloros-backend.service`.
* **Inloggning på Chloros+ (betalnivå)**: kör `chloros-cli login` först. Kontrollen sker på serversidan – utan inloggning misslyckas kommandona med `401 AUTH_REQUIRED`; på den kostnadsfria (Iron) nivån misslyckas de med `403 PLAN_UPGRADE_REQUIRED`.
* Kommandona riktar sig som standard mot `http://127.0.0.1:5000`; `daq pool-*`-familjen respekterar miljövariabeln `CHLOROS_BACKEND_URL` om din backend körs någon annanstans.

***

## En fem minuters session

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — öppna en sensor i poolen

| Variant | Betydelse |
| --- | --- |
| `daq pool-connect` | Smart-detect: hitta valfri DAQ på den här maskinen. |
| `daq pool-connect --port PORT` | DAQ-U på en specifik seriell port (t.ex. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC-adress skannas automatiskt. |
| `daq pool-connect --mac MAC` | DAQ-M på en känd BLE-MAC (innebär `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E med känt värdnamn eller IP-adress — **den tillförlitliga vägen**. |
| `daq pool-connect --eth` | DAQ-E med automatisk upptäckt (mDNS, med ARP som reserv). Se anmärkningen nedan. |

Inställningsflaggor, alla valfria:

| Flagga | Betydelse |
| --- | --- |
| `--integration-time MS` / `-t MS` | Manuell integrationstid i millisekunder. |
| `--frame-avg N` / `-f N` | Antal ramar som medelvärdesberäknas per rapporterat spektrum. |
| `--no-ae` | Inaktivera automatisk exponering (AE är aktiverat som standard). |
| `--no-stream` | Anslut utan att starta strömmen (återuppta senare med `pool-stream --start`). |
| `--cap-id CAP` | Profil för cap-korrigering; standardinställningen för backend är `sunshine_cosine`. Se [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**`--eth` – anmärkning om automatisk upptäckt.** På en värd med flera nätverksgränssnitt (fler än ett aktivt nätverksgränssnitt) kan den *första* `pool-connect --eth` efter uppstart resultera i inga resultat även om sensorn är i gott skick — upptäcktsgenomsökningen kan missa sensorns gränssnitt medan ARP-cachen är tom. Om `--eth` inte hittar något, försök igen, eller hoppa över upptäckten helt med `--eth-host <ip-or-hostname>`, vilket är den tillförlitliga vägen på datorer med flera nätverksgränssnitt. DAQ-E:s värdnamn är `daq-e-<id>.local` (t.ex. `daq-e-def330.local`); dess vanliga IP-adress fungerar också.
{% endhint %}

## `pool-list` — se vad som är anslutet

Visar alla sensorer i backend-poolen, inklusive det `sensor_id` som alla andra kommandon behöver:

| Modell | `sensor_id`-format | Exempel |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktett med bindestreck | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — läser spektrumramar

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Returnerar den senaste ramen, eller de senaste `--recent N`-ramarna; `--json` genererar maskinläsbar utdata för skriptning. Ramarna är radiometriskt kalibrerad spektral irradians (W/m²/nm) på ett rutnät med 135 punkter i intervallet 340–1010 nm, där sensorns täckningsprofil redan har tillämpats. För kvantitativa strålningsvärden ska man beräkna medelvärdet av minst 15 sekunders bildrutor – detta är en egenskap hos instrumentet, inte ett fel.

## `pool-stream` — pausa eller återuppta strömningen

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — spela in en `.daq`-fil

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Flagga | Standard | Betydelse |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Inspelningslängd i sekunder; `0` innebär att körningen fortsätter tills du anger `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Utmatningskatalog, löses **på den maskin som kör backend**. |
| `--device-name NAME` | — | Etikett som lagras tillsammans med inspelningen. |
| `--stop` | — | Avbryt en pågående inspelning. |

{% hint style="info" %}
Inspelningen sker i backend, så filen `.daq` hamnar i **backend-maskinens** filsystem — som standard i `~/Documents/DAQ Live View/` där, inte nödvändigtvis där du körde CLI. Filnamnen innehåller sensor-ID och en tidsstämpel.
{% endhint %}

## `pool-set-cap` — ange det monterade locket

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Kåp-ID:t väljer den fabriksmätta korrigeringsprofilen som tillämpas på varje spektrum, och den **måste matcha den kåpa som fysiskt är monterad på sensorn** — varken sensorn eller programvaran kan på egen hand upptäcka kåpan, och valet stämplas in i varje `.daq`-fil. Standardvärdet överallt är `sunshine_cosine` (varje DAQ levereras med Sunshine-kosinuskorrigeringskåpan installerad, ~12× dämpning enligt konstruktion – ett odeklarerat byte av kåpa korrigerar spektra felaktigt med ungefär den faktorn).

| `--cap-id` | Tillgänglig på |
| --- | --- |
| `sunshine_cosine` (standard) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Endast DAQ-U |
| `none` | Endast DAQ-E – se anmärkning |

Ett lock-ID som inte ingår i sensorns uppsättning avvisas vid anslutning med ett tydligt felmeddelande. `none` (DAQ-E) innebär att locket är fysiskt borttaget — det tillämpar fortfarande en fabriksgeometriprofil för DAQ-E:s infällda glasdiffusor, så det är inte en no-op, och en oskyddad DAQ-E är en bänkkonfiguration, inte en stödd fältkonfiguration. (En DAQ-U utan kåpa är helt utan kåpa och behöver ingen korrigeringsprofil alls; DAQ-M används med sin Sunshine-kåpa.)

## `pool-disconnect` — frigör sensorer

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Kommandosammanfattning

| Kommando | Syfte |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Öppna en sensor i backend-poolen. |
| `daq pool-list` | Visa alla sensorer i poolen med deras `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De senaste N kalibrerade spektrumramarna. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Återuppta/pausa strömning. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Starta/stoppa en `.daq`-inspelning (på backend-sidan). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Byt ut cap-korrigeringsprofilen under körning. |
| `daq pool-disconnect --sensor-id ID [--all]` | Frigör en sensor eller alla. |

***

## Felsökning vid första anslutningen av DAQ-E

1. DAQ-E har ingen status-LED – kontrollera strömförsörjningen via PoE-/länkindikatorn på switchen eller injektorporten, och vänta några sekunder efter uppstart tills enheten har startat upp och anslutit sig till nätverket.
2. Backend-datorn måste befinna sig i **samma sändningsdomän** som sensorn – mDNS passerar inte routrar.
3. På Windows ska du godkänna Defender-brandväggens förfrågan vid första körningen (mDNS UDP 5353, DAQ-E-data UDP 5002, PTP UDP 319/320).
4. Fortfarande inget svar från `--eth`? Använd `--eth-host` med enhetens värdnamn (`daq-e-<id>.local`) eller IP-adress – det är den säkraste vägen, särskilt på värdar med flera anslutningar.

***{% hint style="info" %}**Tips för AI-assistenter.** Varje sida i denna handbok serveras som rå Markdown – lägg till `.md` till sidans slug i gemener (URL) (denna sida: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); då blir det maskinläsbara indexet `https://mapir.gitbook.io/chloros/llms.txt`. För fullständig dokumentation på flaggnivå om `chloros-cli daq` och alla andra kommandofamiljer, hämta [CLI-referensen](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); sökvägen till Python är `chloros_sdk.connect_daq_sensor()` i [SDK-referensen](../reference/sdk-reference.md).
{% endhint %}
