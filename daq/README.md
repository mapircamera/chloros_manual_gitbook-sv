# DAQ-ljussensorer

> **Letar du efter hårdvaran?**Sensorerna i sig – modeller, montering, skyddskåpor, anslutningar, strömförsörjning och SCANNER-appen – beskrivs i**[DAQ-användarhandboken](https://mapir.gitbook.io/daq)**. Detta kapitel behandlar hur man använder dem från Chloros.

MAPIR:s **DAQ**-ljussensorer mäter omgivningsljuset som radiometriskt kalibrerade spektra. I Chloros har de två funktioner:

* **Ett fristående spektralinstrument** — realtidsspektrumdiagram, kolorimetriska data och `.daq`-inspelningar, allt från [fliken Ljussensorer](gui.md), [CLI](cli-quick-start.md) eller Python SDK.
* **En nedåtriktad strålningskälla för reflektans** — under bearbetningen interpolerar Chloros dina `.daq`-avläsningar till varje bildsexponeringstidsstämpel och använder det uppmätta nedåtriktade ljuset för att omvandla kamerans strålningsintensitet till reflektans (`--reflectance-source daq`); ingen panel i scenen krävs för kalibrerade band.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Tre modeller, ett dataformat

| Modell | Transport | Discovery |
| --- | --- | --- |
| **DAQ-U** | USB (seriell) | skanning via seriell port |
| **DAQ-M** | Bluetooth Low Energy | BLE-skanning efter namn |
| **DAQ-E** | Ethernet (IPv4, PoE-driven) | mDNS `_daq-e._tcp` (värdnamn `daq-e-<id>.local`) |

Alla tre använder samma nätverksprotokoll och levererar identiska data:

* Ett **135-punktsspektrum från 340 till 1010 nm i steg om 5 nm**, plus CIE XYZ-tristimulusvärden, i varje ram.
* **Radiometriskt kalibrerad spektral irradians i W/m²/nm** — varje enhets fabrikskalibreringspaket (samt dess aktiva kapselkorrigeringsprofil) tillämpas innan data når dig.
* Samma **`.daq`-inspelningsformat** (en SQLite-fil). Den efterföljande bearbetningen är identisk oavsett vilken överföringsmetod som genererade filen.

Transportstäckarna (USB-seriell, BLE, mDNS/zeroconf) ingår i Chloros-backend — du behöver inte installera någonting för att kommunicera med någon av de tre modellerna via GUI:n eller med CLI:s `pool-*`-kommandon.

***

## Kalibrerat intervall: 340–1010 nm rapporterat, ~374–974 nm kalibrerat

Sensorn rapporterar hela rutnätet 340–1010 nm, men den NIST-spårbara radiometriska förstärkningen sträcker sig över ungefär **374–974 nm**. Chloros avvisar delningen av absolut reflektans för alla kameraband där mindre än hälften av dess spektrala vikt ligger inom det kalibrerade intervallet; det utelämnade bandet rapporteras med utelämningsorsaken `dls-uncalibrated-band-<nm>`.

Bland de LATTICE-filter som finns i sortimentet är endast **F988** berört:

F988:s reflektans kalibreras med hjälp av en reflektanspanel i scenen: bandet ligger utanför DAQ-ljussensorns kalibrerade intervall, så Chloros använder din senaste panelavläsning och behåller den mellan panelavläsningarna.

Om en F988-insamling bearbetas med endast DAQ-data tillgängliga, avvisar Chloros DAQ-baserad reflektans för det bandet med hopporsaken `dls-uncalibrated-band-988` — [arbetsflödet för reflektanspanelen](../calibration-targets.md) är den stödda vägen för F988.

***

## Sensor-ID:n

Varje DAQ rapporterar ett stabilt sensor-ID. Dess format varierar beroende på modell:

| Modell | ID-format | Exempel |
| --- | --- | --- |
| DAQ-U | 5 oktetter med bindestreck | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 oktetter med bindestreck | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Sensor-ID:t är:

* inpräglat i varje `.daq`-fil som den registrerar,
* nyckeln som Chloros använder för att hämta enhetens fabrikskalibreringspaket,
* det värde som du skickar till `--sensor-id` i kommandona CLI och `pool-*`, samt
* för DAQ-E även dess mDNS-värdnamn (`daq-e-def330.local`) – det värde som `--eth-host` accepterar.

***

## Fabrikskalibrering och molnet

Varje DAQ-enhet är individuellt fabrikskalibrerad med en NIST-spårbar radiometrisk kedja, och Chloros laddar varje enhets kalibreringspaket baserat på dess sensor-ID. Kalibreringsrapporten (PDF) för varje enhet kan laddas ner från sensorns inställningar under [fliken Ljussensorer](gui.md).

{% hint style="warning" %}
**DAQ-U och DAQ-M kräver molntillgång för kalibrering.**Ingen av modellerna lagrar något lokalt: deras fabrikskalibreringspaket finns i MAPIR:s moln och hämtas via sensor-ID (för att sedan cachelagras lokalt). Chloros behöver en internetanslutning för att leverera kalibrerade W/m²/nm-data från en DAQ-U eller DAQ-M.**DAQ-E är undantaget** – den har sin kalibrering inbyggd i enheten.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Var inspelningarna sparas

| Yta | Standardmål för `.daq` |
| --- | --- |
| GUI — fliken Ljussensorer | `<project folder>/light_sensor/` (färdiga inspelningar läggs automatiskt till i projektet) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` på den maskin som kör backend |

Varje `.daq`-filnamn innehåller sensor-ID och en tidsstämpel.

***

## I detta kapitel

* [**Fliken DAQ i Chloros**](gui.md) — en fullständig genomgång av användargränssnittet: anslutning av varje modell, inställningar per sensor, spektrumdiagram, kolorimetriska realtidsdata, reflektans från dubbla sensorer och inspelning.
* [**CLI Snabbstart (pool-\*)**](cli-quick-start.md) — styrning av DAQ-sensorer från `chloros-cli daq pool-*`, den stödda kommandoradsvägen.
* [**Cap-profiler och kalibrerat spektralområde**](caps-and-range.md) — vilka cap-profiler som finns per modell, hur man deklarerar dem samt det kalibrerade spektralområdet i detalj.
* [**Inspelning och .daq-formatet**](recording.md) — SQLite-formatet `.daq` och arbetsflöden för inspelning.
* [**DAQ-E-nätverk och tidssynkronisering**](ethernet-ptp.md) — DAQ-E-transportlägen och PTP-tidssynkronisering.
* [**Arbetsflöden för reflektans**](reflectance.md) — användning av DAQ-data för nedåtriktad strålning för att beräkna reflektans.
* För fullständig dokumentation på flaggnivå, se [CLI-referensen](../reference/cli-reference.md) (avsnitt `chloros-cli daq`) och [SDK-referensen](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), båda skrivna för att kunna användas direkt av AI-assistenter.
