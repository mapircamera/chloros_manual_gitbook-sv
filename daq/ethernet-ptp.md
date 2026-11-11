# DAQ-E Nätverk och tidssynkronisering

> Den fysiska nätverkskonfigurationen för sensorn – kabeldragning, PoE, IP-tilldelning och enhetens egna nätverksinställningar – beskrivs i **[DAQ-användarhandboken](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Denna sida behandlar Chloros

-sidan: anslutning, tidssynkronisering och vad man ska göra om upptäckten inte ger något resultat.

DAQ-E är Ethernet-medlemmen i DAQ-familjen: den drivs via PoE, upptäcks via mDNS (tjänst `_daq-e._tcp`) och kan adresseras med ett värdnamn som härleds från dess sensor-ID — `daq-e-<6 hex>.local`, t.ex. `daq-e-def330.local`. Denna sida behandlar hur den överför data i nätverket och hur den deltar i PTP-tidssynkronisering.

## Transportlägen

| Läge | Slutpunkt | Användare | Anmärkningar |
| --- | --- | --- | --- |
| **Multicast** (standard) | UDP `239.10.10.10:5002` | Valfritt antal enheter på samma LAN tar emot samma ström | Varje datagram valideras med CRC-16/CCITT |
| **Raw** | TCP-port `5000` | Exakt en klient (exklusivt) | Direktkompatibel på byte-nivå med DAQ-U |

Chloros

använder multicast som standard, vilket gör att GUI, CLI

och SDK

alla kan övervaka en sensor samtidigt.

## Nätverkskrav

* **Samma sändningsdomän.** Datorn som kör Chloros

måste befinna sig i samma L2-nätverkssegment som sensorn — mDNS-upptäckt passerar inte genom routrar.
* **Brandvägg.** I Windows

lägger installationsprogrammet till brandväggsregeln **Chloros

Backend (UDP-In)**för backend i varje nätverksprofil, vilket täcker DAQ-E-data (UDP 5002), mDNS (UDP 5353) och PTP (UDP 319/320), samt en regel för**Chloros

-backend (TCP-In)**, begränsad till det lokala subnätet, för den anslutning som en DAQ-E upprättar tillbaka till datorn under en firmwareuppdatering. På Linux

öppnar `.deb` UDP-portarna i `ufw` eller `firewalld` när den brandväggen är aktiv vid installationen. En brandvägg från tredje part, eller en som aktiveras senare, kräver att samma portar tillåts.
* **PoE-strömförsörjning, ingen status-LED.** DAQ-E har ingen egen LED – kontrollera strömförsörjningen via länk-/PoE-indikatorn på switchen eller injektorporten, och vänta några sekunder efter uppstart tills enheten har startat upp och anslutit sig till nätverket.

## Anslutning

**GUI:** Fliken Ljussensorer → Anslut sensor → Enhetstyp ”DAQ-E (Ethernet)”. Sökningen körs endast medan anslutningsdialogrutan visas på skärmen (mDNS-sökning plus en ARP-sökning i Windows

) och upprepas var 15:e sekund; knappen Uppdatera startar en ny sökning omedelbart. Upptäckta sensorer visas i rullgardinsmenyn; den första sensorn som upptäcks väljs automatiskt.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend körs):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

En DAQ-E som redan finns i poolen känns igen både på sitt värdnamn och sin IP-adress: om man ansluter den igen under den andra adressen returneras den befintliga poolposten istället för att enheten öppnas en andra gång.

### Värdar med flera nätverkskort och den första anslutningen efter uppstart

På värddatorer med mer än ett aktivt nätverksgränssnitt kan den **första** `pool-connect --eth` efter uppstart bli tom även om sensorn är i gott skick — upptäcktsgenomsökningen kan missa det gränssnitt som sensorn finns på medan ARP-cachen fortfarande är tom. Den säkra lösningen är att hoppa över upptäckten och ange adressen explicit:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` accepterar mDNS-värdnamnet eller IP-adressen, riktar sig alltid mot rätt sensor och är den rekommenderade formen för skript och headless-installationer. I GUI:n använder du knappen Uppdatera i anslutningsdialogrutan och låter en ny sökningscykel genomföras.

## Enhetsinställningar och firmware

Sensorn lagrar själv sina nätverksinställningar: DHCP (fabriksinställningen, som faller tillbaka till en 169.254.x.x-länklokal adress när ingen DHCP-server svarar) eller en statisk adress. Den lagrar också om den ska börja strömma vid uppstart. Ändra dessa inställningar i GUI:n (sensorns inställningsmeny, **Nätverk**:**Ändra…**,**Strömma vid uppstart**,**Starta om sensorn**; se [Fliken DAQ](gui.md)), eller från CLI

och SDK

med `pool-set-network`, `pool-set-auto-stream` och `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; se [CLI

-referensen](../reference/cli-reference.md#daq-e-device-settings)). Sensorn startas om för att tillämpa en nätverksändring, och Chloros

återansluter den. En statisk adress måste finnas i nätverket för den dator som kör Chloros

: Chloros

avvisar en adress som den inte kunde nå efter omstarten. Sensorns ID och dess värdnamn `daq-e-<id>.local` ändras aldrig. Nytt i 1.3.0.

**Firmwareuppdateringar är inbyggda i GUI:n.** När en ansluten DAQ-E kör firmware som är äldre än den bild som medföljer din Chloros

-version, visas en gul **Uppdatering tillgänglig**-ikon i sensorraden, och inställningsfönstret med kugghjulet erbjuder en <version>knapp</version>

för ”Uppdatera till<version>”. Uppdateringen överförs via nätverket på cirka 30 sekunder; sensorn startar om och ansluter automatiskt igen, och om överföringen avbryts förblir den aktuella firmwareversionen oförändrad. Från CLI eller SDK, samt på Linux där det inte finns något grafiskt gränssnitt, kör du `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-tidssynkronisering

DAQ-E-firmware v1.2.0+ deltar i IEEE 1588 PTPv2 som en vanlig (endast slav) klocka. **Chloros-värdens backend är PTP-grandmastern** — varje DAQ-E och varje LATTICE-kamera på LAN:et är slavar till den i domän 0, vilket håller alla enheters tidsstämplar inom en tolerans på ~1 ms. Det är den delade klockan som gör att DAQ-avläsningarna kan tidsstämmas mot kamerans exponeringar (se [Inspelning och .daq-formatet](recording.md)).

Kontrollera synkroniseringen från CLI:

| Kommando | Visar |
| --- | --- |
| `chloros-cli time-sync status` | Värd-grandmaster-status, BMCA-prioriteringar, klockidentitet samt `health`: `OK` eller `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` med en `health_detail` som anger varför dess paket inte når nätverket |
| `chloros-cli time-sync peers` | Alla slavar som upptäcks (DAQ-E-sensorer + LATTICE-kameror) |
| `chloros-cli time-sync cameras` | PTP-status per kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Starta om grandmaster-processen |

I GUI:n visar inställningsfönstret för DAQ-E en realtidsrad för **PTP-synkronisering** med sensorns aktuella PTP-status.**En DAQ-E med PTP avstängt förblir avstängt.** Enheten lagrar sin PTP-inställning (på/av) i sitt eget minne, så en enhet vars PTP har stängts av förblir avstängt efter en återanslutning eller strömavbrott, och dess tidsstämplar är inte PTP-synkroniserade. `chloros-cli daq pool-list` markerar en sådan enhet (`!! PTP is switched OFF on this unit …`) och visar kommandot som åtgärdar problemet:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloross slår aldrig på en enhet igen av sig själv, eftersom avstängning kan vara avsiktlig.

Detaljer för användare som kräver strikt synkronisering:

* Varje strömmat datagram innehåller ett flaggfält; **bit 2 är aktiverad på ramar vars tidsstämpel är PTP-synkroniserad**. Pipelines som kräver strikt synkronisering mellan kamera och datainsamling bör basera sin styrning på den biten.
* Innan en synkroniserad inspelning, kontrollera att sensorn visas i `chloros-cli time-sync peers` och att `chloros-cli daq pool-list` inte markerar dess PTP som avstängd. (MAPIRs interna verktyg för direkt hårdvaruhantering kan också styra inspelningen utifrån PTP-låsning med en `--wait-ptp`-flagga som väntar upp till 15 sekunder på att sensorn ska nå SLAVE-tillstånd; det verktyget ingår inte i den medföljande CLI:n.)
* Medan PTP aktivt fungerar som slav avvisar sensorn manuella klocksignaler (”PTP tillhandahåller klocka”). Detta är avsiktligt – lita på PTP.

## Anmärkningar för Linux

* **PTP-portar utan root-behörighet.** PTP binder till UDP 319/320. `.deb` skriver `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), så backend-programmet binder dem och körs som din användare. Från och med version 1.3.0 ställer paketet inte in någon filkapacitet (`setcap`) på backend-programmet och behöver ingen `libcap2-bin`; lägg inte till någon kapacitet själv, eftersom det uppackade backend-programmet då inte startar. Om PTP inte kan binda port 319, se [Linux-installation](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Headless Jetson / Linux-värd:** paketet installerar backend-tjänsterna men aktiverar ingen av dem. För att PTP (och DAQ-tillgängligheten) ska vara alltid igång utan att någon kör ett kommando, aktivera tjänsten som din användare:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Utan detta körs PTP endast så länge något har startat backend-tjänsten – ett `chloros-cli`-kommando eller ett SDK-skript. Se [Linux-installation](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) för root-användaren `chloros-backend.service` och hur man byter från den.

## Felsökning: ”Inga DAQ-E-enheter hittades”

| Kontroll | Detalj |
| --- | --- |
| Ström | Ingen LED lyser på sensorn – kontrollera PoE- och länkindikatorerna på switch-/injektorporten; vänta några sekunder efter uppstart |
| Sändningsdomän | Värd och sensor befinner sig i samma L2-segment; mDNS dirigeras inte |
| Brandvägg | Windows: installationsprogrammets regel **Chloros Backend (UDP-In)** släpper in DAQ-E-strömmen (UDP 5002), mDNS (5353) och PTP (319/320); en brandvägg från tredje part måste ha samma inställningar. Linux: `.deb` öppnar dessa portar i `ufw` / `firewalld` endast om brandväggen var aktiv vid installationstillfället — tillåt dem själv i en brandvägg som aktiveras senare |
| Värd med flera nätverkskort | Upptäckten söker igenom mDNS med ARP som reserv; `--eth-host <ip-or-hostname>` hoppar över upptäckten och ansluter direkt |
| Omsökning via GUI | Upptäckten körs endast medan anslutningsdialogrutan är öppen; använd Uppdatera |</version>
