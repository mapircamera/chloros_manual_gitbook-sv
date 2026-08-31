# Arbetsflöden för reflektans

En DAQ-ljussensor omvandlar radiometriska bilder till reflektans. Det finns två olika arbetsflöden:

1. **Enkelsensor** — en DAQ-sensor mäter nedåtriktad irradians medan en kamera tar bilder, och Chloros dividerar kamerans strålningsintensitet med den referensen.
2. **Dubbel sensor** — två DAQ-sensorer, en riktad mot himlen och en riktad mot ett objekt, genererar en spektral reflektanskurva i realtid utan att någon kamera används.

## Enkel sensor + kamera (nedåtriktad referens)

DAQ-enheten fungerar som en sensor för nedåtriktat ljus (DLS): kameran mäter uppåtriktad strålning **L**(W/m²/sr/nm), DAQ mäter nedåtriktad irradians**E** (W/m²/nm), och Chloros beräknar reflektansen per band enligt:

> ρ = π · L / E

DAQ-avläsningen är alltid **tidsstämplad i förhållande till exponeringen** — det är därför DAQ och kamerorna delar en PTP-synkroniserad klocka (se [DAQ-E Nätverk &amp; Tidssynkronisering](ethernet-ptp.md)). Sätt på Sunshine-kosinusmössan för arbete utomhus och ange den korrekt; mössans inställning skalar direkt E (se [Mössprofiler och kalibrerat intervall](caps-and-range.md)). För kvantitativt arbete, kom ihåg instrumentets egenskaper: kvantitativ irradians beräknas utifrån ett genomsnitt av avläsningar under minst 15 sekunder.

### Live-inspelning

Koppla DAQ:en till en kamera på fliken Kameror: varje kameras inställningspanel har en rullgardinsmeny för **Ljussensor** som listar alla anslutna DAQ:er (DAQ-U/M/E) från fliken Ljussensorer; för en synkroniserad uppsättning sprids ett val av ljussensor för hela uppsättningen till varje enhet (enskilda kameror kan fortfarande åsidosätta detta). När kopplingen är gjord matas sensorns spektra in i kamerans DLS-plats och exportvärdena för reflektans divideras med det matchade mätvärdet.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Två saker som är värda att känna till:

* **Ingen DAQ kopplad → reflektans avvisas, inte simuleras.** Chloros avvisar reflektansprodukten och registrerar orsaken till att den hoppas över, istället för att tyst returnera en lägre produkt.
* **Det använda mätvärdet bevaras.** För varje reflektansram skrivs det DAQ-värde som faktiskt tillämpats som en `.daq`-sidecar bredvid bildmaterialet, så att inspelningen kan bearbetas på nytt senare ([Inspelning och .daq-formatet](recording.md)).

### Bearbetning av inspelade bilder

För bearbetning efter flygningen ska du spela in en `.daq` under sessionen och spara den tillsammans med bildmaterialet – bearbetningskedjan löser automatiskt den tidsstämplade nedåtriktade strålningen och hämtar eventuella saknade fabrikskalibreringar från molnet i MAPIR vid första användningen. GUI-inspelningar läggs automatiskt till i det öppna projektet när de avslutas.

Reflektansreferensen kan väljas vid bearbetningstillfället – `--reflectance-source` på `chloros-cli process`, eller inställningen för reflektanskälla i GUI:ns projektinställningar:

| Värde | Beteende |
| --- | --- |
| `auto` (standard) | Ett QA-godkänt kalibreringsmål inom ramen är den absoluta referensen; DAQ-nedåtriktad strålning (ρ = π·L/E) är reservvärdet |
| `daq` | DAQ-auktoritativt |
| `target` | Strikt mål inom bildramen; ingen DAQ-ersättning |

Se [Kalibreringsmål](../calibration-targets.md) för arbetsflöden för mål och [LATTICE-kapitlet](../lattice/README.md) samt [CLI-referensen](../reference/cli-reference.md) för hela bearbetningskedjan. När du läser in exporterade reflektanspixlar ska du använda den angivna skalan (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) – se [Utgångsbildformat](../output-image-formats.md).

### Band utanför DAQ:ns kalibrerade intervall

DAQ:ns radiometriskt kalibrerade intervall är ~374–974 nm. Chloros avvisar DAQ-baserad reflektans för alla kameraband där mindre än hälften av dess spektrala vikt ligger inom detta intervall och rapporterar orsaken till utelämnandet som `dls-uncalibrated-band-<nm>`. Bland de levererade SKU:erna påverkar detta endast F988: F988:s reflektans kalibreras med hjälp av en reflektanspanel i scenen; bandet ligger utanför DAQ-ljussensorns kalibrerade område, så Chloros använder din senaste panelavläsning och behåller den mellan panelavläsningarna. Om en F988-kamera körs enbart med DAQ avvisar Chloros DAQ-baserad reflektans för det bandet med hopporsaken `dls-uncalibrated-band-988` – arbetsflödet med panelen är den stödda metoden.

## Dubbel sensor (omgivande ljus + objekt)

Två DAQ-sensorer – valfritt par, oavsett transport – ger ett realtidsreflektansspektrum utan kamera: en sensor är riktad mot himlen (**Omgivande ljuskälla**), en mot motivet (**Objektskanner**), och Chloros beräknar per våglängd:

> R(λ) = objekt(λ) / omgivning(λ)

(noll där omgivningen ≤ 0).

### I användargränssnittet

När båda sensorerna är anslutna under fliken Ljussensorer öppnar du överlägget för att lägga till sensorer (knappen ”+” på en diagramruta i rutnätsvyn) och väljer **Kombinera omgivning + objekt**. Välj de två sensorerna i rullgardinsmenyerna ”Omgivande ljuskälla” och ”Objektskanner” och klicka på ”Skapa”. Gruppen visas som ett eget diagram och som en rad i sidofältet med en grön**REF**-markering.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Under reflektansdiagrammet (listvy) beräknar en **vegetationsindex-tabell** i realtid index från kurvan med hjälp av bandcentrum vid blått 450 / grönt 550 / rött 670 / NIR 800 nm. Förhållandebaserade index som eliminerar den absoluta skalan (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) visas alltid; index som kräver absolut reflektans (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) visas endast när båda sensorerna är strömkalibrerade modeller.

### Skenbar kontra relativ – märkningsregeln

Chloros märker utdata från dubbelsensorer enligt vad sensorparet faktiskt kan uppge:

| Sensorpar | Märkning |
| --- | --- |
| Båda sensorerna kalibrerade — fabrikskalibreringspaket laddat | **Synlig reflektans** |
| En av sensorerna okalibrerad | **Relativ reflektans** |

Alla tre modellerna är radiometriska: så snart en sensors fabrikskalibreringspaket har laddats är dess spektra absoluta W/m²/nm, så ett par kalibrerade sensorer ger en absolut skenbar reflektans — transporten avgör inte detta. En sensor som fortfarande strömmar råa räknestillstånd (inget paket tillgängligt) nedgraderar resultatet till en relativ kurva (spektralformen är fortfarande giltig). Båda sensorerna bör ha korrekt angivna gränsvärden ([Gränsvärdesprofiler &amp; kalibrerat område](caps-and-range.md)).

### Från Python

Det finns inget särskilt anrop för dubbla sensorer i den sammanslagna ytan SDK: öppna två sessioner med `chloros_sdk.connect_daq_sensor()` och jämför deras `latest()`-spektra själv, med samma märkningskonvention. (Ett verktyg för dubbelsensorinspelning finns även på MAPIR:s interna yta för direkt hårdvaruanslutning, listad i [CLI-referensen](../reference/cli-reference.md) för fullständighetens skull – det ingår inte i den levererade CLI; arbetsflödet i grafiskt gränssnitt ovan är den stödda metoden.)
