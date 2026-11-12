# Snabbstart för CLI (pool-*)

De medföljande `chloros-cli`-enheterna styr DAQ-sensorerna via kommandofamiljen **`daq pool-*`** – enkla HTTP-klienter som styr sensorn via Chloros-backendens permanenta sensorpool. Backend-systemet hanterar transporten, vilket innebär att GUI, CLI och SDK-skript alla delar ett aktivt handtag istället för att konkurrera om porten. Allt en kund behöver är tillgängligt via `pool-*`: anslutning, strömning, inspelning av kalibrerade `.daq`-filer och byte av kap-profiler.

`pool-*` är den DAQ-gränssnittsyta som stöds i släppta versioner. `chloros-cli daq --help` listar `pool-*`-underkommandona, och `daq discover` / `daq list` kör samma skanning som `pool-discover`. Om ett DAQ-underkommando för direkt hårdvarustyrning inte är tillgängligt avslutas programmet med ett uttryckligt felmeddelande som anger vilket paket som saknas och hänvisar tillbaka till `pool-*` — ingenting misslyckas tyst. (`pip install chloros-sdk` tillhandahåller inte heller kommandon för direkt hårdvaruanslutning.)

***

## Förutsättningar

* **Chloros-backend måste vara igång** — kommandona `pool-*` är HTTP-klienter, inte hårdvarudrivrutiner. I Windows startar du Chloros-skrivbordsappen (den startar backend). På headless Linux/Jetson aktiverar du tjänsten som din användare: `sudo systemctl enable --now chloros-backend@$USER.service` (se [Linux-installation](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Inloggning på Chloros+ (betaltjänst)**: kör först `chloros-cli login`. Kontrollen sker på serversidan – utan inloggning misslyckas kommandon med `401 AUTH_REQUIRED`; på den kostnadsfria (Iron) nivån misslyckas de med `403 PLAN_UPGRADE_REQUIRED`.
* Kommandona riktar sig som standard mot `http://127.0.0.1:5000`; `daq pool-*`-familjen respekterar miljövariabeln `CHLOROS_BACKEND_URL` om din backend körs någon annanstans.

***

## En fem minuters session

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

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
| `daq pool-connect --mac MAC` | DAQ-M på en känd BLE-MAC-adress (innebär `--ble`). |
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

Om du ansluter en DAQ-E som redan finns i poolen under en annan av dess adresser – dess `daq-e-<id>.local`-värdnamn eller dess IP-adress – returneras den befintliga poolposten istället för att enheten öppnas en andra gång.

{% hint style="warning" %}
**`--eth` – varning gällande automatisk upptäckt.** På en värd med flera nätverksgränssnitt (fler än ett aktivt nätverksgränssnitt) kan den *första* `pool-connect --eth` efter uppstart resultera i inget resultat även om sensorn är i gott skick – upptäcktsgenomsökningen kan missa sensorns gränssnitt medan ARP-cachen är tom. Om `--eth` inte hittar något, försök igen eller hoppa över upptäckten helt med `--eth-host <ip-or-hostname>`, vilket är den tillförlitliga vägen på datorer med flera nätverksgränssnitt. DAQ-E:s värdnamn är `daq-e-<id>.local` (t.ex. `daq-e-def330.local`); dess vanliga IP-adress fungerar också.
{% endhint %}

## `pool-discover` — skanna utan att ansluta

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Söker efter sensorer som du kan ansluta till och visar adressen att ansluta till, utan att ansluta. `--only` tar en kommaseparerad delmängd av `usb,ble,eth`; `--timeout SEC` ställer in söktiden per transport (standard 5 s, begränsad till 1–20 s). Sensorer som redan är öppna i poolen listas inte (en ansluten DAQ-M slutar sända ut information) — använd `pool-list` för dessa. I en levererad version kör `chloros-cli daq discover` och `daq list` samma skanning.

## `pool-list` — visa vad som är anslutet

Visar alla sensorer i backend-poolen, inklusive den `sensor_id` som alla andra kommandon behöver:

| Modell | `sensor_id`-format | Exempel |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktett med bindestreck | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

En DAQ-E vars PTP har stängts av (inställningen sparas på enheten) markeras i `pool-list` med `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` och kommandot som slår på den igen — se [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — läs spektrumramar

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Returnerar den senaste ramen, eller de senaste `--recent N`-ramarna; `--json` genererar maskinläsbar utdata för skriptning. Ramarna är radiometriskt kalibrerad spektral irradians (W/m²/nm) på ett rutnät med 135 punkter, 340–1010 nm, där sensorns täckningsprofil redan har tillämpats. För kvantitativa strålningsvärden ska man beräkna medelvärdet av minst 15 sekunders bildrutor – detta är en egenskap hos instrumentet, inte ett fel.

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
| `--duration SEC` / `-d SEC` | `0` | Inspelningslängd i sekunder; `0` innebär att programmet körs tills du anger `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Utmatningskatalog. Med backend på den här datorn tolkas en relativ sökväg utifrån den mapp där du körde kommandot; med en fjärrbackend tolkas sökvägen **på den maskin som kör backend**. |
| `--device-name NAME` | — | Etikett som lagras tillsammans med inspelningen. |
| `--stop` | — | Avbryt en pågående inspelning. |
| `--confirm-after SEC` | `20` | Efter start, vänta upp till denna tid på att den första raden ska skrivas, och varna med ett högt ljud (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`) om ingen anländer. `0` inaktiverar kontrollen. |
| `--heartbeat SEC` | `30` | Med `--duration` skrivs radantalet ut var SEC:e sekund och en varning ges om det slutar öka. `0` inaktiverar den. |

{% hint style="info" %}
Inspelningen sker i backend, så filen `.daq` hamnar på **backend-maskinens** filsystem — som standard i `~/Documents/DAQ Live View/` där, inte nödvändigtvis där du körde CLI-kommandot. Filnamnen innehåller sensor-ID och en tidsstämpel.
{% endhint %}

## `pool-set-cap` — ange vilket lock som är monterat

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Kåp-ID:t väljer den fabriksmätta korrigeringsprofilen som tillämpas på varje spektrum, och den **måste matcha den kåpa som fysiskt är monterad på sensorn** — varken sensorn eller programvaran kan på egen hand upptäcka kåpan, och valet stämplas in i varje `.daq`-fil. Standardvärdet överallt är `sunshine_cosine` (varje DAQ levereras med Sunshine-kosinuskorrigeringskåpan installerad, ~12× dämpning enligt konstruktion — ett odeklarerat byte av kåpa korrigerar spektra felaktigt med ungefär den faktorn).

| `--cap-id` | Tillgänglig på |
| --- | --- |
| `sunshine_cosine` (standard) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Endast DAQ-U |
| `none` | Endast DAQ-E – se anmärkning |

Ett lock-ID som inte ingår i sensorns uppsättning avvisas vid anslutning med ett tydligt felmeddelande. `none` (DAQ-E) innebär att kåpan är fysiskt borttagen — den tillämpar fortfarande en fabriksgeometriprofil för DAQ-E:s infällda glasdiffusor, så det är inte en no-op, och en bar DAQ-E är en bänkkonfiguration, inte en stödd fältkonfiguration. (En DAQ-U utan lock är helt utan lock och behöver ingen korrigeringsprofil alls; DAQ-M används med sitt Sunshine-lock.)

## `pool-disconnect` — frigör sensorer

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` frigör en sensor oavsett vem som öppnade den, så den frigör även en sensor som ett kraschat skript lämnat kvar i poolen (en inspelning på den måste först avslutas). En sensor som inte finns i poolen har inget att frigöra: kommandot skriver ut `<sensor-id>: already released (not in the pool)` och avslutas med `0`. En frigöring som backend vägrade eller inte bekräftade avslutas med `1`.

## `pool-enable-ptp` — slå på PTP på en DAQ-E igen

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

En DAQ-E lagrar sin PTP-inställning (på/av) i sitt eget minne, så en enhet vars PTP har stängts av förblir avstängd — och dess tidsstämplar är inte PTP-synkroniserade — tills den slås på igen. `pool-list` markerar en sådan enhet och skriver ut detta kommando för den. Chloross slår aldrig på en enhet igen av sig själv, eftersom avstängning kan vara avsiktlig. Nytt i 1.3.0.

## DAQ-E-enhetsinställningar

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` visar sensorns firmware (och om denna Chloros-version innehåller en nyare version), dess adress och hur den erhölls, den sparade nätverksinställningen om den skiljer sig från den aktuella, strömning vid uppstart och PTP. `pool-set-network` startar om sensorn för att tillämpa ändringen och återansluter den med de inställningar den hade; `--no-reboot` sparar endast inställningarna. En statisk adress måste finnas i den här datorns nätverk, annars avvisar Chloros den; `--force` åsidosätter detta, endast för en dator som kommer att omadresseras eller dirigeras till den. Omstart avvisas medan sensorn registrerar data. `pool-update-firmware` motsvarar GUI:ns **Uppdatera till \<version\>** för kommandoraden och är sättet att uppdatera en DAQ-E på Linux, som saknar GUI. Nytt i 1.3.0.

***

## Kommandosammanfattning

| Kommando | Syfte |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Öppna en sensor i backend-poolen. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Söker efter sensorer utan att ansluta (hittar en DAQ-M:s BLE-MAC). |
| `daq pool-list` | Visar alla sensorer i poolen med deras `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De senaste N kalibrerade spektrumramarna. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Återuppta/pausa strömning. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Starta/stoppa en `.daq`-inspelning (på backend-sidan). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Byt cap-korrigeringsprofil under körning. |
| `daq pool-disconnect --sensor-id ID [--all]` | Frigör en sensor eller alla. |
| `daq pool-enable-ptp --sensor-id ID` | Slå på PTP igen för en DAQ-E vars PTP är avstängd. |
| `daq pool-info --sensor-id ID [--json]` | En DAQ-E:s firmware, adress, sparade nätverksinställningar, strömning vid uppstart och PTP. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP eller en statisk adress; startar om och återansluter sensorn. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Strömma vid uppstart. |
| `daq pool-reboot --sensor-id ID` | Starta om en DAQ-E och anslut den på nytt. |
| `daq pool-update-firmware --sensor-id ID` | Flasha den här versionens DAQ-E-firmware till en äldre sensor. |

***

## Felsökning vid första anslutningen av en DAQ-E

1. DAQ-E har ingen status-LED – kontrollera strömförsörjningen via PoE-/länkindikatorn på switchen eller injektorporten, och vänta några sekunder efter uppstart tills den har startat upp och anslutit sig till nätverket.
2. Backend-datorn måste befinna sig i **samma sändningsdomän** som sensorn – mDNS passerar inte genom routrar.
3. En brandvägg måste släppa igenom mDNS (UDP 5353), DAQ-E-data (UDP 5002) och PTP (UDP 319/320). Windows-installationsprogrammet lägger till en regel för detta, och Linux-programmet `.deb` öppnar dem i en `ufw` / `firewalld` som var aktiv vid installationstillfället; en brandvägg från tredje part, eller en som aktiverats senare, måste tillåta dem.
4. Fortfarande inget från `--eth`? Använd `--eth-host` med enhetens värdnamn (`daq-e-<id>.local`) eller IP-adress, vilket hoppar över upptäckten.

***
{% hint style="info" %}
**Tips för AI-assistenter.** Varje sida i denna handbok serveras som rå Markdown – lägg till `.md` till sidans URL-slug i gemener (denna sida: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); det maskinläsbara indexet är `https://mapir.gitbook.io/chloros/llms.txt`. För fullständig dokumentation på flaggnivå om `chloros-cli daq` och alla andra kommandofamiljer, hämta [CLI-referensen](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); Python-sökvägen är `chloros_sdk.connect_daq_sensor()` i [SDK-referensen](../reference/sdk-reference.md).
{% endhint %}
