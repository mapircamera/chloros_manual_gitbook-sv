# Fliken DAQ i Chloros

Fliken DAQ – som heter **Ljussensorer** i Chloros-sidopanelen – är kontrollpanelen för realtidshantering av [ljussensorerna DAQ-U, DAQ-M och DAQ-E](README.md): anslut sensorer via valfritt transportprotokoll, bevaka kalibrerade spektra i realtid, beräkna reflektans i realtid från ett sensorpar och spara `.daq`-filer direkt i ditt projekt.

Fliken blir tillgänglig så snart Chloros-backend har startat klart. Flikens diagram matas av Chloros DAQ-tjänst via en live-anslutning som återansluts automatiskt (2–10 sekunders väntetid) om den avbryts; så länge tjänsten är otillgänglig står det **Ingen server** i sensorns statusrad.

Layouten består av en **sensorsidopanel**(en rad per ansluten sensor) samt ett**diagramområde** (en diagramruta per sensor eller grupp).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Ansluta en sensor

Klicka på **Anslut sensor** högst upp i sidofältet. Anslutningsdialogrutan öppnas i huvudområdet (eller som ett överlägg när du lägger till ytterligare en sensor – i så fall visas en Avbryt-knapp).

| Kontroll | Beteende |
| --- | --- |
| **Enhetstyp** | `DAQ-U (USB)` (standard), `DAQ-M (Bluetooth)` eller `DAQ-E (Ethernet)`. Om du byter startar skanningen om för den nyligen valda transporten. |
| **Port / BLE-enhet / Värdnamn / IP** | Visar en lista över upptäckta enheter som `device - description`; den första posten som identifieras som en sensor väljs automatiskt. Under sökningen visas `Scanning...` (USB), `Scanning (N)...` med en nedräkning på 8 sekunder (BLE) eller `Discovering ethernet sensors (N)...` med en nedräkning på 5 sekunder (Ethernet). Tomma resultat visas som `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Uppdatera** | Skannar om den valda transporten omedelbart (inaktiverad mitt i en BLE-/Ethernet-skanning). |
| **Anslut sensor** | Aktiveras när en enhet har valts; ommärks till `Connecting...` medan anslutningen upprättas. |

Sökningen körs endast **medan anslutningsdialogrutan visas på skärmen** och upprepas var 15:e sekund endast för den valda transporten – enbart att öppna fliken utlöser inte någon sökning. Vid ett fel visas följande meddelande i dialogrutan: *”Anslutningen misslyckades. Prova att koppla ur och koppla in sensorn igen, och klicka sedan på Anslut igen.”*

Sidopanelen öppnas automatiskt när din första sensor ansluts.

{% hint style="info" %}
**Visas inte DAQ-E?**DAQ-E har ingen status-LED — kontrollera PoE-/länkindikatorn på den switch eller injektorport som den är ansluten till, och vänta några sekunder efter uppstart tills den har startat upp. Chloros-maskinen måste befinna sig i samma sändningsdomän (mDNS passerar inte genom routrar). Installationsprogrammets**brandväggsregel för Chloros-backend (UDP-In)** släpper igenom mDNS (UDP 5353), DAQ-E-data (UDP 5002) och PTP (UDP 319/320); en brandvägg från tredje part måste ha samma portar öppna. Två DAQ-E-enheter på ett LAN upptäcks separat, var och en under sitt eget `daq-e-<id>.local`-värdnamn.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Enhetstyp erbjuder DAQ-U (USB), DAQ-M (Bluetooth) och DAQ-E (Ethernet)</figcaption></figure>

***

## Sensorns sidofält

Varje ansluten sensor får en rad (plus en rad per Ambient+Object-grupp). Raderna kan ordnas om genom att dra, och deras ordning påverkar även ordningen på diagramrutorna. Klicka på en rad för att göra den sensorn/gruppen till det aktiva diagrammet i listvyn.

| Element | Betydelse |
| --- | --- |
| Färgad vänsterkant | Sensorns diagramfärg. |
| Transportmärke | `DAQ-U` / `DAQ-M` / `DAQ-E`, eller en grön `REF`-märkning för en Ambient+Object-reflektansgrupp. |
| Enhetsnamn | Standardvärdet är sensorns serienummer (dess fasta identitet för kalibrering, filnamn av typen `.daq` och importmatchning); anpassade namn sparas per projekt. |
| **Kalibrerad**-ikon (grön) | Visas när sensorns fabrikskalibreringspaket är laddat, dvs. spektra är uttryckta i W/m²/nm. |
| **Uppdatering tillgänglig**-ikon (gul, endast DAQ-E) | Den körda firmwareversionen är äldre än den som medföljer denna Chloros

-version. Under en uppdatering visas förloppet i realtid (`Flashing… N%`, `Restarting sensor…`, sedan `Updated X → Y` eller `Failed`). |
| Öga | Växlar mellan att visa och dölja sensorn i diagrammet. |
| Kugghjul | Öppnar inställningsfönstret för den enskilda sensorn (nedan). |
| ✕ (röd) | Kopplar bort sensorn eller tar bort en Ambient+Object-grupp. |

Ovanför raderna finns två knappar:

* **Anslut sensor** — öppnar anslutningsdialogen (ändrar namn till `Connecting...` när den är upptagen).
* **Spela in allt / Stoppa allt**— startar eller stoppar en `.daq`-inspelning på**alla**anslutna sensorer. Kräver minst en sensor**och ett öppet projekt** (verktygstips: ”Öppna ett projekt för att spela in”); knappen blir röd medan en inspelning pågår.

I tomt läge står det ”Inga sensorer anslutna”.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Inställningar per sensor (kugghjulsfönstret)

Öppnas med kugghjulsikonen i en sensorrad. Innehåll i ordning:

* **Informationsrader** — Enhetstyp (DAQ-U/M/E), Anslutning (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (COM-port, BLE-adress eller värd) och seriell anslutning.
* **Kalibreringsrapport: Ladda ner** — hämtar enhetens NIST-spårbara kalibreringscertifikat (PDF) och öppnar det i din PDF-läsare. Tillgängligt så snart serienumret är känt; certifikatet sparas i cache vid första anslutningen.
* **Enhetsnamn** — klicka på pennikonen för att byta namn; sparas per projekt.
* **Graflinjens färg** — färgprov; sparas per projekt.
* **Integrationstid (ms)**— skjutreglage + siffra,**1–500 ms**, standard**32 ms**. Inaktiverad när AE är PÅ.
* **Ramgenomsnitt**— skjutreglage + siffra,**1–50 ramar**, standard**20**.
* **AE: PÅ/AV**— växla mellan automatisk exponering;**standardinställning PÅ** vid anslutning. Stäng av den för att ställa in exponeringstiden manuellt.
* **Avbryt strömning / Starta strömning** — pausa eller återuppta livestreamen.
* **Spela in / Avsluta inspelning** — `.daq`-inspelning per sensor (kräver ett öppet projekt).
* **Cap** — profilen för cap-korrigering (nästa avsnitt).
* **Rader med liveinformation** — Integreringstid (ms), FPS, Provtagningar, Inspelning (röd `REC` eller `Off`) och Status (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Endast DAQ-E: raderna för nätverk, firmware och PTP

* **Värdnamn / IP** — enhetens aktuella adress.
* **Firmware**— aktuell firmwareversion, plus en åtgärdsruta: en**Uppdatera till \<version\>

**-knapp visas när denna Chloros

-version innehåller en nyare DAQ-E-firmwarebild. Uppdateringen överförs via nätverket på cirka 30 sekunder; sensorn startar om och ansluter automatiskt igen, och en avbruten överföring lämnar den aktuella firmwaren intakt. Förloppet visas i realtid (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), och cellen visar `Up to date` när den är aktuell.
* **PTP-synkronisering** — det aktuella PTP-tillståndet (faller tillbaka till `unknown`). DAQ-E-firmware v1.2.0+ deltar i IEEE 1588 PTPv2 som en ren slavklocka; Chloros

-värdens backend är PTP-grandmaster, och varje DAQ-E- och LATTICE-kamera i LAN:et är slavar till den i domän 0, vilket håller tidsstämplarna inom ungefär 1 ms.
* **Nätverk**— den adress enheten kör på och hur den fick den (DHCP, länklokal eller statisk), samt en sparad inställning som gäller efter omstart.**Ändra…**öppnar ett formulär:**Automatisk (DHCP)**eller**Statisk IP**med en IP-adress, en subnätmask och en valfri gateway.**Tillämpa och starta om** sparar inställningarna, startar om sensorn och ansluter den på nytt. En statisk adress utanför den här datorns nätverk avvisas, eftersom Chloros

inte kunde nå sensorn efter omstarten; en ruta för att tillåta det ändå visas då, för en dator som kommer att omadresseras eller dirigeras till det nätverket. **Strömma vid uppstart**anger om sensorn börjar strömma så snart den slås på, innan någon programvara ansluter.**Starta om sensorn** startar om den och återansluter den med de aktuella inställningarna. En omstart nekas medan sensorn registrerar data. Nytt i 1.3.0.

För en Ambient+Object-grupp visar kugghjulsmenyn endast gruppens källsensorer, enhetsnamn och grafens linjefärg.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Val av kåpa

I rullgardinsmenyn **Cap** anger du för Chloros

vilken fysisk kåpa som är monterad över sensorns diffusor, och tillämpar den kåpans fabriksmätta korrigeringsprofil på varje spektrum. Alternativen beror på modellen:

| Modell | Kåpalternativ |
| --- | --- |
| DAQ-U | Ingen (bar sensor), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (kosinuskorrigerare) |
| DAQ-M | Inget (bar sensor), Sunshine (kosinuskorrigerare) |
| DAQ-E | Inget (bar sensor), FOV 15°, FOV 45°, FOV 90°, Sunshine (kosinuskorrigerare) |

**Standardinställningen för alla modeller är Sunshine (kosinuskorrigerare)** — MAPIR

levererar alla DAQ-enheter med Sunshine-kåpan monterad, och detta är standardkonfigurationen för utomhusbruk: en 180° halvsfärisk vy med kosinusfel ≤ ±4 % upp till 60° och ≤ ±4,5 % upp till 70° (rekommenderas inte vid solhöjd under ~15°), med inbyggd dämpning (~12×). Ditt val sparas i projektet.

{% hint style="warning" %}
**Valet av kåpa måste stämma överens med den fysiska kåpan.**Varken sensorn eller programvaran kan avgöra vilken kåpa som är monterad. Valet styr både den realtidsbaserade korrigeringen och den stämpel som skrivs in i varje `.daq`-fil — med Sunshine-kåpans dämpning på ~12× leder ett odeklarerat lockbyte till felkorrigering av spektra med ungefär den faktorn. (Att ta bort och sätta tillbaka samma lock ger en avvikelse på cirka 1,5 %.) Välj**Inget (bar sensor)** endast när locket är fysiskt borttaget; på en DAQ-E tillämpar ”None” fortfarande en fabriksinställd geometriprofil för dess infällda glasdiffusor – det är inte en tomoperation – och en bar DAQ-E är en bänkkonfiguration, inte en stödd fältkonfiguration.
{% endhint %}

{% hint style="info" %}
Uppgradering från en tidigare manual: växlingsknappen ”Sunshine Diffuser Installed” på webbläsarsidan från version 1.1.0 finns inte längre. Hanteringen av locket sker nu via denna lockprofil per sensor, som tillämpas på serversidan.
{% endhint %}

***

## Diagramområdet

En fast översta fält innehåller en **växlingsknapp mellan list- och rutnätsvy**samt ett**zoomreglage för diagram** (rutstorlek 200–2000 px). Vyn växlar automatiskt till rutnätsvy när det finns fler än en diagramgrupp, och tillbaka till listvy när det finns en eller färre. Visningsläge och diagramstorlek sparas per projekt.**Spektrumdiagrammet** för varje sensor visar:

* **X-axeln** — Våglängd (nm). Sensorns rutnät är 340–1010 nm med 5 nm-steg (135 punkter), interpolerat till 1 nm för visning.
* **Y-axeln**— för en kalibrerad sensor (grön**Kalibrerad**-ruta),**Effekt (W/m²/nm)**— radiometriskt kalibrerad spektral irradians på alla tre transportformerna — med ett automatiskt SI-prefix (m/µ/n) valt utifrån toppen. En sensor vars kalibreringspaket inte är laddat plottar sina råa mätvärden som**Intensitet (räknevärden)**, oskalade. När ett diagram överlagrar båda typerna visas effekten på den vänstra axeln och antalet på den högra axeln.
* En regnbågsfärgad spektral bakgrund under en enskild kurva; flera sensorer på ett diagram överlagras som färgade linjer med nedtonade bakgrunder.
* **Håll muspekaren över**– en vertikal markör med våglängd och värde per sensor (W/m²/nm för en kalibrerad sensor, hela räknestal i övriga fall);**dra** för att zooma (en knapp för att zooma ut visas när du har zoomat in).
* En **+**-knapp (endast i rutnätsvyn) för att lägga till en sensor till detta diagram eller skapa en grupp (nedan).
* Enhetsnamnet centrerat högst upp, och en spinner tills den första ramen anländer.

**Mättnad** markeras inte på själva diagrammet: en mättad sensor visar röd `SATURATED`-statustext och en röd `Saturated: Yes`-rad i tabellen med realtidsdata. Sänk integrationstiden eller aktivera AE igen för att rensa den.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Tabell med realtidsdata (listvy)

Under diagrammet i listvy, uppdateras var 500 ms:

* **Alla modeller**: Ljusfärgprov (sRGB från CIE XYZ), Mättad (Ja/Nej), CIE 1931 X/Y/Z, Kromaticitet x/y, CIE u′/v′, CCT (K), CRI (Ra), dominerande våglängd (nm), toppvåglängd (nm), exciteringsrenhet, Duv, CIE L\*/a\*/b\* och Munsell H/V/C.
* **Endast kalibrerade sensorer**(valfri av DAQ-U / DAQ-M / DAQ-E så snart dess fabrikskalibreringspaket har laddats – den gröna**Kalibrerad**-markeringen i sensorraden är kännetecknet): Total effekt (W/m²), fotopisk lux (lx), skotopisk lux (lx), S/P-förhållande, PPFD samt PPFD Red

/Green

/Blue

(µmol/m²/s) och de opiska irradianserna – S-kon, Melanopic, Rhodopic, M-kon, L-kon (alla W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Reflektansgrupper (Omgivning + Objekt)

Två anslutna sensorer kan kombineras till en realtidsvisning av reflektans – utan kamera:

1. I rutnätsvyn klickar du på **+**på en diagramruta och väljer**Kombinera omgivning + objekt**.
2. Välj en **Omgivande ljuskälla**-sensor och en**Objektskanner**-sensor (två olika sensorer) och klicka sedan på**Skapa**.

Chloros

beräknar R(λ) = objekt(λ) / omgivning(λ) per våglängd utifrån de två realtidsströmmarna (0 där omgivningen ≤ 0). Gruppens etikett följer sensornas kalibreringsklass:

* Båda sensorerna kalibrerade (paket laddat) → **”Synlig reflektans”**.
* Om någon av sensorerna är okalibrerad → **”Relativ reflektans”**.

Gruppen visas som en grön `REF`-rad i sidofältet och i ett eget diagram (regnbågsfyllning, värden visas vid muspekning med 4 decimaler, dra för att zooma).

Menyn **+**erbjuder även**Lägg till ny sensor** med tre placeringsalternativ: *Kombinera ny sensor* (lägg till i detta diagram), *Flytta befintlig sensor hit* eller *Visa ny sensor* (i eget diagram).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Tabell över vegetationsindex

I listvyn finns en tabell över vegetationsindex under en reflektansgrupps diagram, beräknad utifrån den aktuella reflektansen vid bandcentrumen **blått 450 / grönt 550 / rött 670 / NIR

800 nm** (värden med 4 decimaler, `---` när de inte kan beräknas; håll muspekaren över ett indexnamn för att se dess fullständiga namn):

* **Visas alltid** (skalinvariant, valfri sensorkombination): NDVI

, GNDVI

, ENDVI, WDRVI

, GRVI

, CVI

, GCI

, MSR.
* **Endast när båda sensorerna är effektkalibrerade** (båda sensorpaketen laddade): EVI

, SAVI

, OSAVI

, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI

, NLI, MNLI, FCI, GEMI

.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Inspelning av `.daq`-filer

* Inspelning kräver ett **öppet projekt** — annars är både ”Spela in allt” (i sidofältet) och inspelningsknappen för varje sensor inaktiverade.
* Filerna sparas i **`<project folder>/light_sensor/`**; filnamnen innehåller sensor-ID och en tidsstämpel, och enhetens namn sparas tillsammans med inspelningen.
* När en inspelning avslutas (Stopp, Stopp alla eller en avbrott mitt i inspelningen) **läggs den färdiga `.daq`-filen automatiskt till i det öppna projektet** — den visas i projektets fillista utan att behöva läggas till manuellt, redo att användas som nedåtriktade data för [reflektansbearbetning](README.md).
* En röd `REC`-indikator visas i inställningsfönstrets realtidsrader under inspelningen.

För kvantitativa strålningsvärden ska du beräkna medelvärdet av minst 15 sekunders data — detta är en egenskap hos instrumentet, inte ett fel.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Layouter för flera sensorer och projektpersistens

* Kombinera flera sensorer i ett diagram (delade axlar), behåll separata diagram (automatisk rutnätslayout), flytta sensorer mellan diagram, dra och ordna om rader/rutor och dölj enskilda sensorer med ögonikonen.
* Chloros

sparar följande per projekt: enhetsnamn, diagramfärger, diagramstorlek, visningsläge och varje sensors inställningar (integreringstid, bildmedelvärde, AE-status, val av kåpa).
* **När ett projekt öppnas på nytt ansluts sensorerna automatiskt** via adress – COM-port för DAQ-U, BLE-enhet för DAQ-M, mDNS-värdnamn för DAQ-E (upplöses även om enhetens IP-adress har ändrats) – och tillämpar återigen varje sensors sparade kapselprofil, bildrutsmedelvärde, AE-status och manuell integrationstid.***

## Kameraparning (DLS)

Det finns inget att para ihop. Till skillnad från DLS-arbetsflöden för drönare som kopplar en ljussensor till en kamera i förväg, matchar Chloros

DAQ-data med bildmaterialet i efterhand: vid import/bearbetning interpoleras `.daq`-avläsningarna till varje bilds exponeringstidstämpel. Spela in med valfri ansluten sensor (`.daq` läggs automatiskt till i projektet), och reflektansbearbetningen hittar rätt mätvärden utifrån tid – se [DAQ-ljussensorer](README.md) för information om hur data om nedåtriktat ljus används. För **live**-inspelningar med LATTICE väljer du istället sensorn i rullgardinsmenyn**Ljussensor** på fliken Kameror; varje inspelning sparar då det mätvärde som användes, matchat endast mot den inspelningens kameror (se [Reflektansarbetsflöden](reflectance.md)).</version\>
