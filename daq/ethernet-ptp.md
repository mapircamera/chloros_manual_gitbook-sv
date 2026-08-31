# DAQ-E Nätverk och tidssynkronisering

> Den fysiska nätverkskonfigurationen för sensorn – kabeldragning, PoE, IP-tilldelning och enhetens egna nätverksinställningar – beskrivs i **[DAQ-användarhandboken](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Denna sida behandlar Chloros-sidan: anslutning, tidssynkronisering och vad man ska göra om upptäckten inte ger något resultat.

DAQ-E är Ethernet-modellen i DAQ-familjen: den drivs via PoE, upptäcks via mDNS (tjänsten `_daq-e._tcp`) och kan adresseras med ett värdnamn som härleds från dess sensor-ID — `daq-e-<6 hex>.local`, t.ex. `daq-e-def330.local`. Denna sida behandlar hur den överför data i nätverket och hur den deltar i PTP-tidssynkronisering.

## Transportlägen

| Läge | Slutpunkt | Användare | Anmärkningar |
| --- | --- | --- | --- |
| **Multicast** (standard) | UDP `239.10.10.10:5002` | Valfritt antal enheter på samma LAN tar emot samma ström | Varje datagram valideras med CRC-16/CCITT |
| **Raw** | TCP-port `5000` | Exakt en klient (exklusivt) | Direkt kompatibel på byte-nivå med DAQ-U |

Chloros använder multicast som standard, vilket gör att GUI:t, CLI och SDK alla kan övervaka en sensor samtidigt.

## Nätverkskrav

* **Samma sändningsdomän.** Datorn som kör Chloros måste befinna sig i samma L2-nätverkssegment som sensorn — mDNS-upptäckt går inte genom routrar.
* **Brandväggsfråga för Windows: godkänn den.** Första gången Chloros binder multicast-socklarna frågar Windows Defender en gång. Om du tillåter detta omfattar det DAQ-E-data (UDP 5002), mDNS (UDP 5353) och PTP (UDP 319/320). På Linux sker detta utan meddelande.
* **PoE-strömförsörjning, ingen status-LED.** DAQ-E har ingen egen LED – kontrollera strömförsörjningen via länk-/PoE-indikatorn på switchen eller injektorporten, och vänta några sekunder efter uppstart tills enheten har startat upp och anslutit sig till nätverket.

## Anslutning

**GUI:** Fliken Ljussensorer → Anslut sensor → Enhetstyp ”DAQ-E (Ethernet)”. Sökningen pågår endast medan anslutningsdialogrutan visas på skärmen (mDNS-sökning plus en ARP-sökning på Windows), och upprepas var 15:e sekund; knappen Uppdatera startar en ny sökning omedelbart. Upptäckta sensorer visas i rullgardinsmenyn; den första upptäckta sensorn väljs automatiskt.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend körs):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Värddatorer med flera nätverkskort och den första anslutningen efter uppstart

På värddatorer med fler än ett aktivt nätverksgränssnitt kan den **första** `pool-connect --eth` efter uppstart visa tomt resultat även om sensorn är funktionsduglig — upptäcktsgenomsökningen kan missa det gränssnitt som sensorn finns på medan ARP-cachen fortfarande är tom. Den säkra lösningen är att hoppa över upptäckten och ange adressen explicit:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` accepterar mDNS-värdnamnet eller IP-adressen, riktar sig alltid mot rätt sensor och är den rekommenderade formen för skript och headless-installationer. I GUI:n använder du knappen Uppdatera i anslutningsdialogrutan och låter en ny sökningscykel genomföras.

## Enhetsinställningar och firmware

Sensorn själv lagrar nätverksinställningar – statisk IP kontra DHCP + länklokal adressering, enhetsnamn, automatisk strömning vid uppstart, OTA-lösenord. Dessa inställningar på enhetssidan är inte tillgängliga som kommandon i den levererade CLI; de hanteras via Chloros-grafiska gränssnittet där de visas, eller med hjälp av MAPIR-support.

**Firmwareuppdateringar är inbyggda i GUI:n.**När en ansluten DAQ-E kör en firmwareversion som är äldre än den som medföljer din Chloros-version, visas en gul**Uppdatering tillgänglig**-ikon i sensorraden, och i inställningsfönstret med kugghjulsikonen finns en<version>

knapp</version> för ”Uppdatera till<version>

”. Uppdateringen överförs via nätverket på cirka 30 sekunder; sensorn startar om och ansluter automatiskt på nytt, och om överföringen avbryts förblir den aktuella firmware-versionen oförändrad.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-tidssynkronisering

DAQ-E-firmware v1.2.0+ deltar i IEEE 1588 PTPv2 som en vanlig (endast slav) klocka. **Backend för Chloros-värden är PTP-grandmastern** — varje DAQ-E och varje LATTICE-kamera i LAN:et är slavar till den i domän 0, vilket håller alla enheters tidsstämplar inom en tolerans på ~1 ms. Det är den delade klockan som gör att DAQ-avläsningarna kan tidsstämmas mot kamerans exponeringar (se [Inspelning och .daq-formatet](recording.md)).

Kontrollera synkroniseringen från CLI:

| Kommando | Visar |
| --- | --- |
| `chloros-cli time-sync status` | Värdens grandmaster-status, BMCA-prioriteringar, klockidentitet |
| `chloros-cli time-sync peers` | Alla upptäckta slavar (DAQ-E-sensorer + LATTICE-kameror) |
| `chloros-cli time-sync cameras` | PTP-status per kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Starta om grandmaster-processen |

I GUI:n visar inställningsfönstret för DAQ-E en realtidsrad för **PTP-synkronisering** med sensorns aktuella PTP-status.

Detaljer för användare som kräver strikt synkronisering:

* Varje strömmat datagram innehåller ett flaggfält; **bit 2 är aktiverad på ramar vars tidsstämpel är PTP-synkroniserad**. Pipelines som kräver strikt synkronisering mellan kamera och DAQ bör basera sin styrning på den biten.
* Innan en synkroniserad inspelning påbörjas, kontrollera att sensorn visas i `chloros-cli time-sync peers`. (MAPIR:s interna verktyg för direkt hårdvaruhantering kan också styra inspelningen vid PTP-låsning med en `--wait-ptp`-flagga som väntar upp till 15 sekunder på att sensorn ska nå SLAVE-tillstånd; denna funktionalitet ingår inte i den levererade versionen av CLI.)
* Medan PTP är aktivt i slavläge avvisar sensorn manuella klocksignaler (”PTP tillhandahåller klocka”). Detta är avsiktligt – lita på PTP.

## Anmärkningar om Linux

* **PTP kräver `libcap2-bin` vid installationen.** `.deb` postinst beviljar `cap_net_bind_service=+ep` på `/usr/lib/chloros/chloros-backend` så att det kan binda PTP-portarna 319/320 utan root-behörighet. Om `libcap2-bin` saknas hoppas det steget över och PTP kommer inte att kunna startas. Lösning:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Headless Jetson / Raspberry Pi:** vid första installationen genereras systemd-enheten `chloros-backend.service` men aktiveras inte. För PTP som alltid är på (och DAQ-tillgänglighet) utan GUI:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Utan den körs PTP endast medan Chloros-GUI:n är öppen.

## Felsökning: ”Inga DAQ-E-enheter hittades”

| Kontroll | Detalj |
| --- | --- |
| Ström | Ingen LED lyser på sensorn – kontrollera PoE- och länkindikatorerna på switchens/injektorns port; vänta några sekunder efter uppstart |
| Sändningsdomän | Värd och sensor befinner sig i samma L2-segment; mDNS dirigerar inte |
| Windows-brandvägg | Acceptera Defender-frågan vid första körningen (UDP 5002, 5353, 319/320) |
| Värd med flera nätverkskort | Den första upptäckten efter uppstart kan missa sensorn – anslut med `--eth-host <ip-or-hostname>` |
| Omsökning via GUI | Upptäckten körs endast medan anslutningsdialogrutan är öppen; använd Uppdatera |</version>
