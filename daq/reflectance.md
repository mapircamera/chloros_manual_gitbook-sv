# Arbetsflöden för reflektans

En DAQ-ljussensor omvandlar radiometriska bilder till reflektans. Det finns två olika arbetsflöden:

1. **Ensensor** — en DAQ mäter nedåtriktad irradians medan en kamera tar bilder, och Chloros dividerar kamerans radians med den referensen.
2. **Dubbel sensor** — två DAQ-sensorer, en riktad mot himlen och en riktad mot ett objekt, genererar en spektral reflektanskurva i realtid utan att någon kamera används.

## Enkel sensor + kamera (nedåtriktad referens)

DAQ-enheten fungerar som en sensor för nedåtriktat ljus (DLS): kameran mäter uppåtriktad strålning **L**(W/m²/sr/nm), DAQ-enheten mäter nedåtriktad irradians**E** (W/m²/nm) och Chloros beräknar reflektansen per band enligt:

> ρ = π · L / E

DAQ-avläsningen är alltid **tidsstämplad i förhållande till exponeringen** — det är därför DAQ och kamerorna delar en PTP-synkroniserad klocka (se [DAQ-E-nätverk och tidssynkronisering](ethernet-ptp.md)). Anpassa Sunshine-kosinuskåpan för utomhusarbete och deklarera den korrekt; kåpdeklarationen skalar direkt E (se [Cap Profiles & Calibrated Range](caps-and-range.md)). För kvantitativt arbete, kom ihåg instrumentets egenskaper: kvantitativ irradians beräknas utifrån ett genomsnitt av avläsningar under minst 15 sekunder.

### Live-inspelning

Koppla DAQ:en till en kamera på fliken Kameror: varje kameras inställningspanel har en rullgardinsmeny för **Ljussensor** som listar alla anslutna DAQ:er (DAQ-U/M/E) från fliken Ljussensorer; för en synkroniserad matris sprids ett val av ljussensor för hela matrisen till varje enhet (enskilda kameror kan fortfarande åsidosätta detta). När kopplingen är gjord matas sensorns spektra in i kamerans DLS-plats och exportvärdena för reflektans divideras med det matchade mätvärdet. I en uppsättning använder en enhet med egen sensor sitt eget mätvärde – för sina band i den livekompositbilden, sina exportdata och sin automatiska exponering – medan de övriga enheterna använder uppsättningens.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Två beteenden som är värda att känna till:

* **Ingen DAQ-koppling → reflektans avvisas, inte simuleras.** Chloros avvisar reflektansprodukten och registrerar orsaken till att den hoppas över, istället för att tyst returnera en sämre produkt.
* **Det använda mätvärdet bevaras.** För varje reflektansram skrivs det DAQ-mätvärde som faktiskt tillämpats som en `.daq`-fil i inspelningarnas `daq/`-mapp, så att inspelningen kan bearbetas på nytt senare ([Inspelning och .daq-formatet](recording.md)). En `.daq.cameras.json`-fil bredvid den namnger de kameror den beskriver, och vid ombearbetning matchas den endast mot bilderna från just dessa kameror.

### Bearbetning av inspelade bilder

För bearbetning efter flygningen ska du spela in en `.daq` under sessionen och spara den tillsammans med bildmaterialet – bearbetningskedjan identifierar automatiskt den tidsstämplade nedåtriktade strålningen och hämtar eventuell saknad fabrikskalibrering från MAPIRs moln vid första användningen. GUI-inspelningar läggs automatiskt till i det öppna projektet när de avslutas.

Reflektansreferensen kan väljas vid bearbetningstillfället – `--reflectance-source` på `chloros-cli process`, eller inställningen för reflektanskälla i GUI:ns projektinställningar:

| Värde | Beteende |
| --- | --- |
| `auto` (standard) | Ett kalibreringsmål inom ramen som klarat kvalitetskontrollen (QA) är den absoluta referensen; DAQ-nedstrålning (ρ = π·L/E) används som reserv. Ett mål som klarar kvalitetskontrollen med en flagga (en panel borttagen eller ett band ur funktion) prioriteras inte automatiskt: kedjorna mål, mål+DAQ och endast DAQ testas i ordning efter deras uppmätta poäng på uteslutna paneler. |
| `daq` | DAQ-auktoritativt |
| `target` | Strikt mål inom ramen; ingen DAQ-ersättning |

Se [Kalibreringsmål](../calibration-targets.md) för arbetsflöden för mål och [LATTICE-kapitlet](../lattice/README.md) samt [CLI-referensen](../reference/cli-reference.md) för den fullständiga bearbetningskedjan. Vid läsning av exporterade reflektanspixlar ska du använda den stämplade skalan (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) – se [Utgångsbildformat](../output-image-formats.md).

### Band utanför DAQ:ns kalibrerade intervall

DAQ:s radiometriskt kalibrerade intervall är ~374–974 nm. Chloros-programmet avvisar DAQ-baserad reflektans för alla kameraband där mindre än hälften av dess spektrala vikt ligger inom detta intervall och rapporterar orsaken till utelämnandet `dls-uncalibrated-band-<nm>`. Bland de levererade SKU:erna påverkar detta endast F988: F988:s reflektans kalibreras med hjälp av en reflektanspanel i scenen; bandet ligger utanför DAQ-ljussensorns kalibrerade intervall, så Chloros använder din senaste panelavläsning och behåller den mellan panelavläsningarna. Om en F988-kamera körs enbart med DAQ avvisar Chloros DAQ-baserad reflektans för det bandet med hopporsaken `dls-uncalibrated-band-988` – arbetsflödet med panelen är den stödda metoden.

## Dubbel sensor (omgivningsljus + objekt)

Två DAQ-sensorer – valfritt par, oavsett transport – ger ett realtidsreflektansspektrum utan kamera: en sensor är riktad mot himlen (**Omgivande ljuskälla**), en mot motivet (**Objektskanner**), och Chloros beräknar per våglängd:

> R(λ) = objekt(λ) / omgivning(λ)

(noll där omgivningen ≤ 0).

### I användargränssnittet

När båda sensorerna är anslutna under fliken Ljussensorer öppnar du överlägget för att lägga till sensorer (knappen ”+” på en diagramruta i rutnätsvyn) och väljer **Kombinera omgivning + objekt**. Välj de två sensorerna i rullgardinsmenyerna**Omgivande ljuskälla**och**Objektskanner**och klicka på**Skapa**. Gruppen visas som ett eget diagram och som en rad i sidofältet med en grön**REF**-markering.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Under reflektansdiagrammet (listvy) beräknar en **vegetationsindex-tabell** i realtid index från kurvan med hjälp av bandcentrum vid blått 450 / grönt 550 / rött 670 / NIR 800 nm. Förhållandebaserade index som eliminerar den absoluta skalan (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) visas alltid; index som kräver absolut reflektans (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) visas endast när båda sensorerna är effektkalibrerade modeller.

### Skenbar kontra relativ — märkningsregeln

Chloros märker utdata från dubbelsensorer utifrån vad sensorparet faktiskt kan uppvisa:

| Sensorpar | Märkning |
| --- | --- |
| Båda sensorerna kalibrerade — fabrikskalibreringspaket laddat | **Skenbar reflektans** |
| En av sensorerna okalibrerad | **Relativ reflektans** |

Alla tre modellerna är radiometriska: så snart en sensors fabrikskalibreringspaket har laddats är dess spektra absoluta W/m²/nm, så ett par kalibrerade sensorer ger en absolut skenbar reflektans — transporten avgör inte detta. En sensor som fortfarande strömmar råa räknestillstånd (inget paket tillgängligt) nedgraderar resultatet till en relativ kurva (spektralformen är fortfarande giltig). Båda sensorerna bör ha korrekt deklarerade gränsvärden ([Gränsvärdesprofiler & kalibrerat intervall](caps-and-range.md)).

### Från Python


Det finns inget särskilt anrop för dubbla sensorer i den samlade SDK-gränssnittet: öppna två sessioner med `chloros_sdk.connect_daq_sensor()` och jämför deras `latest()`-spektra själv, genom att tillämpa samma märkningskonvention. (Ett verktyg för inspelning med dubbla sensorer finns även i MAPIRs interna gränssnitt för direkt hårdvaruhantering, vilket för fullständighetens skull anges i [CLI-referensen](../reference/cli-reference.md) – det ingår inte i den levererade CLI:n; arbetsflödet via GUI ovan är den stödda metoden.)
