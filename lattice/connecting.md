# Ansluta kameror

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Fliken Kameror innan något är anslutet</p></figcaption></figure>

Chloros
upptäcker automatiskt LATTICE-kameror på länken — från fliken Kameror i GUI:n, från `chloros-cli lattice` eller från Python
-SDK
:et. Kamerans modellsträng styr allt nedströms: Chloros
hämtar sensorprofilen, bandlayouten och fabrikskalibreringen från kamerans `DeviceUserID` + `DeviceSerialNumber`, så **det finns inget att konfigurera per kamera**.

Innan anslutning, se till att värdnätverket är konfigurerat — link-local-adressering, jumbo-ramar och, för arrayer, inställningarna för nätverkskortets mottagningsbuffert. Det är inställningar på hårdvarusidan och finns beskrivna i LATTICE-manualen: [**Nätverksinställningar**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Ansluta från GUI

Öppna fliken **Kameror** i Chloros
-sidomenyn (hårdvaruflikarna visas när backend har startat klart), eller använd huvudmenyn → **Anslut till kamera**. Båda öppnar dialogrutan**Anslut kamera(er)**.

### Dialogrutan **Anslut kamera(er)**Dialogrutan skannar nätverket så fort den öppnas (”Skannar nätverket...”) och visar en lista över alla kameror som den hittar. Varje rad visar kamerans**modell**(t.ex. `LATT-M3M-L41-F550`),**serienummer**och**IP-adress**.

* **Klicka på en rad för att markera den**(markeras i grönt). Du kan markera**flera kameror** och ansluta dem på en gång – Chloros
ansluter dem i tur och ordning.
* Rader med märket **&quot;Ansluten&quot;** är redan anslutna och kan inte markeras på nytt.
* Rader med märket **&quot;I grupp&quot;** tillhör en kamera-grupp som för närvarande är ansluten. Koppla först bort gruppen för att kunna använda den kameran fristående.
* **Anslut** — ansluter den eller de valda kamerorna; knappen visar ett antal, t.ex. ”Anslut (3)”, när fler än en är markerade.
* **Sök igen** — kör sökningen på nytt.
* **Stäng** — stänger dialogrutan.
* Om sökningen avslutas utan resultat visar dialogrutan **”Inga kameror hittades i nätverket”** — se [Felsökning](connecting.md#troubleshooting) nedan.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Dialogrutan Anslut kamera(er) — visas här utan kameror i nätverket</p></figcaption></figure>

### Första anslutningen: nedladdning av kalibreringspaket

Den **första gången** en viss kamera ansluts till en dator hämtar Chloros
kamerans fabrikskalibreringspaket (\~3,8 MB) från själva kameran via GigE. Medan detta pågår visar dialogrutan en grön panel med texten **”Hämtar kalibreringsdata från kameran”**och en förloppsindikator för varje serienummer — räkna med ungefär**70 sekunder** per kamera. Vid fler än sex kameror rullar förloppsraderna inuti panelen. Paketet lagras i cache på värddatorn, så vid senare anslutningar av samma kamera hoppas nedladdningen över helt (och panelen visas aldrig).

### Analysera system

Dialogrutans knapp **Analysera system** undersöker värddatorn och nätverket (texten visar ”Analyserar...” medan det pågår) och genererar en diagnostikrapport:

* **Värddator** — CPU-kärnor och RAM; GPU-namn och minne, eller ”GPU: Ingen upptäckt”.
* **Nätverksgränssnitt** — varje nätverkskortets namn, länkhastighet, MTU (med en ”jumbo”-markering där den är aktiv), upp-/ned-status och om det sitter på en USB-buss.
* **Kameror**— serienummer, modell, IP-adress och**vilket nätverkskort varje kamera är ansluten till**.
* **Prestanda** — aktuell jämfört med idealisk bildfrekvens (fps) per kamera för pixelformatet, med en grön rad som lyder ”Potential: N× förbättring möjlig” när det ideala värdet överstiger det aktuella.
* **Varningar och numrerade rekommendationer** — eller ”Systemet ser bra ut för det aktuella antalet kameror.” när det inte finns något att åtgärda.

Kör verktyget när upptäckt eller strömning beter sig oväntat — det identifierar de flesta problem på nätverkskortssidan (felaktig MTU, kamera på fel gränssnitt, begränsningar för USB-adapter) utan att du behöver lämna dialogrutan.

### Ansluta en array

För att ansluta två eller flera kameror som en **synkroniserad array**, använd istället guiden för arrayanslutning (**Anslut kameraarray**): den guidar dig genom valet av master/slav (förifylld av en synkroniseringskabel – vilken kabelansluten kamera som helst kan vara master, och RGB
-kameran `-FRGB` är förvald när den är ansluten via kabel), val av visningsläge (separata eller kombinerade rutor) samt en scen med arrayinställningar med en live-projektion av möjliga bildhastigheter (fps) och kabelbandbredd innan du bekräftar. Guiden och arbetsflödena för uppsättningar beskrivs i avsnittet om uppsättningar med flera kameror i denna handbok; motsvarigheten i CLI
är ”LATTICE Camera First-Connect Workflow” i [CLI
-referensen](../reference/cli-reference.md).

## Anslutning via CLI
och SDK
Åtkomst via

CLI
och SDK
kräver ett betalt Chloros
+-abonnemang och att du är inloggad; detta kontrolleras på serversidan (`401 AUTH_REQUIRED` när du inte är inloggad, `403 PLAN_UPGRADE_REQUIRED` på gratiskontot).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Fullständiga signaturer, alternativ och arbetsflöden för insamling: [CLI
-referens](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK
-referens](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Hur kalibreringen hanteras vid anslutning

Varje LATTICE-kamera har sitt fabrikskalibreringspaket **inbyggt**, och Chloros
kontrollerar även MAPIR
molntjänst när kameran ansluts:

| Situation   | Vad Chloros
använder                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**| Den**senaste kalibreringen som publicerats för den serienumret** — molnkopian har företräde framför kopian på kameran. En kamera som har kalibrerats om eller uppdaterats av MAPIR
uppdateras därför automatiskt; ingen åtgärd från användaren krävs. |
| **Offline**|**Paketet i kameran**, i befintligt skick. Helt offline-baserade arbetsflöden fortsätter att fungera; de hämtar helt enkelt inte nyare kalibreringar förrän kameran har varit online en gång (eller har fått en fabriksåterställning).                                                  |

Vid fotograferingstillfället ”fryses” de koefficienter som faktiskt tillämpas **in i varje bilds XMP-metadata**. En senare kalibreringsuppdatering ändrar aldrig i det tysta bilder som du redan har tagit — vid ombearbetning av en gammal bild används de koefficienter som är inlagda i dess XMP, inte de som är senaste idag.

## Felsökning

* **”Inga kameror hittades i nätverket”**– kontrollera den länklokala konfigurationen i [Nätverksinställningar](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statisk värd-NIC `169.254.x.x/16`, kameror på samma länk, ingen DHCP/gateway förväntas. Använd sedan**Analysera system**i anslutningsdialogrutan för att kontrollera på vilket nätverkskort varje kamera är (eller inte är) synlig.**Skanna om** efter varje ändring av kabeldragning eller nätverkskort.
* **En rigg som tidigare fungerade vägrar att ansluta** (arraypanelen stängs av med `FRAMES WILL DROP` / `Reduce ROI to enable`) — en uppdatering av nätverkskortdrivrutinen har i det tysta återställt inställningarna för mottagningsringen. Återanvänd inställningarna eller kör `chloros-cli lattice network --fix` från en terminal med administratörsbehörighet; se [Nätverksinställningar](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **En kamera visar ”In Array”** — den tillhör en ansluten array-session. Koppla bort arrayen för att använda kameran fristående.
