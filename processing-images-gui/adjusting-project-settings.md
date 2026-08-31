# Justera projektinställningarna

Innan du bearbetar dina bilder är det viktigt att konfigurera projektinställningarna så att de passar dina arbetsflödeskrav. Panelen ”Projektinställningar” (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">) ger dig fullständig kontroll över kalibrering, bearbetningsalternativ, multispektrala index och exportformat.

## Öppna projektinställningarna

1. Öppna ditt projekt i Chloros
2. Klicka på ikonen **Projektinställningar** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i vänster sidofält
3. Panelen Projektinställningar visar alla konfigurationsalternativ

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Panelen Projektinställningar – Visning, måldetektering och bearbetning</p></figcaption></figure>{% hint style="info" %}
**Inställningarna sparas automatiskt** tillsammans med ditt projekt. När du öppnar ett projekt igen återställs alla inställningar.
{% endhint %}

***

## Snabbkonfiguration för vanliga arbetsflöden

### Standardinställningar (rekommenderas för de flesta användare)

Standardinställningarna fungerar bra för typiska Survey3- och LATTICE-arbetsflöden:

* ✅ **Vignettkorrigering**: Aktiverad
* ✅ **Reflektanskalibrering / vitbalans**: Aktiverad (använder MAPIR-mål och/eller data från DAQ-ljussensorn)
* ✅ **Debayer-metod**: Standard (Snabb, medelhög kvalitet)
* ✅ **Exportformat**: TIFF (16-bitars)
* ✅ **Alla exportprodukter**: Aktiverat (LATTICE exporterar automatiskt till debayered, förhandsgranskning, radians och reflektans)

Importera bara dina bilder och börja bearbeta med dessa standardinställningar.

***

## Översikt över projektinställningar

Panelen Projektinställningar är indelad i avsnitten nedan. Två ytterligare avsnitt – **DAQ-ljussensor**och**Arrayjustering** – visas automatiskt när ditt projekt innehåller relevanta filer. För fullständig dokumentation, se [Projektinställningar](../project-settings/project-settings.md).

### Visning

* **Upplösning för bildminiatyrer**: Upplösningen för miniatyrerna i bildrutnätet. Alternativ:**Standard (512 px)**,**1024 px**,**2048 px**,**Full upplösning**. Endast för visning — påverkar aldrig bearbetningen. Högre värden ser skarpare ut när man zoomar in, men laddas långsammare.

### Måldetektering

Styr hur Chloros identifierar kalibreringsmål i dina bilder.

**Viktiga inställningar:*** **Minsta kalibreringsprovområde (px)**: Storleksgräns för måldetektering (standard:**25**, intervall 0–10 000)
* **Minsta målkluster (0–100)**: Likhetströskel för gruppering av målregioner (standard:**60**)**När ska du justera:**

* Öka provområdet om du får falska detekteringar
* Minska om mål inte detekteras
* Justera klustringen om mål delas upp i flera detekteringar

{% hint style="info" %}
Dessa inställningar är nedtonade när **reflektanskalibrering/vitbalans** är avstängd – när den är avstängd körs måligenkänningen aldrig alls.
{% endhint %}

### Bearbetning

Huvudsakliga alternativ för bildbearbetning och kalibrering.

**Viktiga inställningar:*** **Vignettkorrigering**: Kompenserar för mörkare kanter i bilden ✅ Rekommenderas
* **Reflektanskalibrering / vitbalans**: Kalibrerar bilder med hjälp av detekterade mål (Survey3) och/eller data från DAQ-ljussensorn (LATTICE) ✅ Rekommenderas
* **Debayer-metod**: Algoritm för konvertering av RAW till 3-kanals multispektral
* **Minsta omkalibreringsintervall**: Minsta tid i sekunder mellan användning av kalibreringsmål (standard:**0** = använd alla, intervall 0–3600)**Okalibrerade reservprodukter:**När en bildram inte kan reflekskalibreras (inget mål tillgängligt eller kalibrering inaktiverad) exporteras den som en av två reservprodukter —**exakt en av de två finns per körning**, vald av omkopplaren för vignettkorrigering:

* **Exportera sensorns respons**: skriver `Sensor_Response_Images` — används när vignettkorrigering är**av*** **Exportera vignettkorrigerad**: skriver `Vignette_Corrected_Images` – används när vignettkorrigering är**på**Den kryssruta som inte är aktiv är gråmarkerad. Om du avmarkerar den aktiva kryssrutan skrivs den filen inte alls.**LATTICE-exportprodukter** (visas för varje projekt; de gäller för LATTICE-inspelningar):

* **Exportera debayered**: den linjära debayered-bilden (`Debayered_Images`). Gäller RGB och multispektrala moduler.
* **Exportera förhandsgranskning**: visningsförhandsgranskningen (`Preview_Images`). RGB = vitbalans (DAQ-ljuskälla om tillgänglig, annars gråvärld) + gamma; multispektral = falskfärgsträckning.
* **Exportera strålningsintensitet**: float32 spektral strålningsintensitet (`Radiance_Images`, W/m²/sr/nm). Endast multispektrala moduler — gäller inte RGB-master.
* ****Exportera reflektans**: uint16-reflektans (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0) när en `.daq`-avläsning av nedåtriktat ljus eller ett mål inom bildrutan täcker bildrutan. Endast multispektrala moduler.

Alla fyra är **aktiverade som standard**— en importerad LATTICE-råram fördelas till alla aktiverade och tillämpliga produkter i ett enda bearbetningspass. Kryssrutan**Exportera reflektans** är gråmarkerad när reflektankalibrering är avstängd. Inställningar som inte kan användas på grund av en överordnad inställning är alltid gråmarkerade med ett verktygstips som anger vilken inställning som måste ändras.**Avancerade inställningar:*** **Tidszonsförskjutning för ljussensor**: Timmar från UTC för tidsanpassning av ljussensorn (standard: 0, intervall −12 till +12)
* **Tillämpa PPK-korrigeringar**: Använder GPS-/exponeringsstiftdata från `.daq`-filer (standard: av)
* **Exponeringsstift 1/2**: Tilldelar kameror till exponeringsstift för konfigurationer med två kameror

{% hint style="info" %}
**LATTICE-ingångsnivån är automatisk.** LATTICE-bildtagningar har sin bearbetningsnivå angiven i XMP-metadata, och bearbetningen påbörjas alltid vid råbilden – det finns inget att konfigurera i GUI:n. (Flaggan CLI `--input-level` finns som en inställning för avancerade användare för bilder där metadata har gått förlorat; se [CLI-referensen](../reference/cli-reference.md).)
{% endhint %}

### Debayer-metod

Vi erbjuder för närvarande två debayer-metoder i Chloros:

#### Standard (snabb, medelhög kvalitet)

Standard-debayern bearbetar snabbt men visar färgbrus från debayeringen, vilket resulterar i mindre exakta och mer brusiga bilder.

#### Texture Aware (långsam, högsta kvalitet) \[Endast Chloros+]

Texture Aware använder en högkvalitativ kantkänslig debayer i kombination med en AI/ML-brusreduceringsmodell som tar bort nästan allt brus från debayeringen. Modellen kräver GPU-minne (VRAM) för att köras: med **7 GB eller mer VRAM** kan den bearbeta flera bilder samtidigt; under 7 GB körs en bild i taget (märkbart långsammare). Se [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**LATTICE-bilderna använder alltid standarddemosaic.** Det finns ingen LATTICE-tränad Texture Aware-modell, så alternativet erbjuds inte för LATTICE-bilder — Survey3-bilder i samma projekt kan fortfarande använda den.
{% endhint %}

### Index (multispektrala index)

Konfigurera vilka vegetationsindex som ska beräknas och exporteras. Rullgardinsmenyn i användargränssnittet erbjuder **27 fördefinierade indexformler**.**Så här lägger du till index:**

1. Klicka på knappen**”Lägg till index”**

2. Välj ett index från rullgardinsmenyn (NDVI, NDRE, GNDVI, etc.)
3. Konfigurera visualiseringsinställningarna (LUT-färger, värdeintervall)
4. Lägg till flera index efter behov

**Populära index:*** **NDVI**: Allmän vegetationshälsa (vanligast)
* **NDRE**: Tidig stressdetektering tillsammans med RedEdge
* **GNDVI**: Känslig för klorofyllkoncentration
* **OSAVI**: Fungerar bra med synlig mark
* **EVI**: Områden med högt bladarealindex (LAI)**Anpassade formler:**

* Skapa anpassade multispektrala indexformler med bandberäkningar över alla bildkanaler
* Spara anpassade formler för återanvändning
* Anpassade formler är en Chloros+-funktion; tillgängligheten beror på din prenumerationsnivå

För alla tillgängliga index och formler – inklusive vilka namn som endast finns i GUI och vilka som även fungerar i CLI/SDK – se [Multispektrala indexformler](../project-settings/multispectral-index-formulas.md).

### Export

Styr utdatafilformatet.

**Tillgängliga format**(inställning:**Kalibrerat bildformat**, standard**TIFF (16-bit)**):

* **TIFF (16-bit)**: Rekommenderas för GIS och vetenskaplig analys
* **TIFF (32-bit, procent)**: Flyttalsvärden
* **PNG (8-bit)**: Förlustfri komprimering för visualisering
* **JPG (8-bit)**: Minsta filstorlek, förlustkomprimering

Utdata skrivs till projektmappen, grupperade efter kamera och format: `<project>/<camera>/<format>/<Product>_Images/`. Radiance skrivs **alltid** som float32 till mappen `tiff32` oavsett denna inställning. Exporterade filer behåller källfilnamnet — mappen identifierar produkten. Se [Slutföra bearbetningen](finishing-the-processing.md) för den fullständiga utdatastrukturen.

{% hint style="warning" %}
**Avläsning av reflektansvärden**: det DN-värde som innebär att ρ = 1,0 beror på källkameran – LATTICE använder 32768 (märkt som XMP `Chloros:PixelScale`), Survey3 använder 65535. Läs av taggen istället för att anta ett konstant värde. Se [Utgångsbildformat](../output-image-formats.md).
{% endhint %}

### DAQ-ljussensor

I detta avsnitt listas alla DAQ-filer för nedåtriktad strålning (`.daq` / `.csv`) i ditt projekt, en rad per fil, med uppgifter om sensormodell, filnamn och den **cap**-korrigering för diffusorn som gäller för den filen.

* **Överskrivning av gränsvärde (alla filer)**: en enda rullgardinsmeny som gäller för hela projektet.**Auto** (standard) använder det registrerade gränsvärdet för varje fil – solsken antas där inget har registrerats, eftersom alla MAPIR-DAQ:er levereras med solskenskorrigeraren. Att välja ett cap åsidosätter alla filer: råa registreringar korrigeras med det, och registreringar som redan har ett cap refereras om (den registrerade korrigeringen ångras och det valda capet tillämpas).
* Raderna visar en varning när ett registrerat takvärde var hubbens antagna standardvärde snarare än bekräftat av operatören, och när det valda takvärdet saknar profil för den enhetsmodellen (överskrivningen nekas för den filen).

DAQ-inspelningar som gjorts på fliken Ljussensorer läggs automatiskt till i det öppna projektet, och importerade `.daq`-/`.csv`-filer visas här så snart de har lagts till.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Nedre projektinställningar — Index, exportformat, avsnittet DAQ-ljussensorer samt kontrollerna för projektmall och mapp</p></figcaption></figure>### Arrayjustering

Detta avsnitt visas **endast** när minst en bild i projektet innehåller den modul-till-modul-justeringstransformation som LATTICE-arrayer stämplar vid inspelningen (`Chloros:Alignment*` XMP). Här visas hur många bilder som har taggar och vilken kamera som är referens, med följande kontroller:

* **Tillämpa arrayjustering** (standard: på): förvränger varje bearbetad produkt (debayered / förhandsgranskning / strålning / reflektans / index) till arrayens gemensamma referensgeometri. Av = exporterar i sensorns ursprungliga geometri.
* **Beskär till gemensam överlappning** (standard: på): beskär justerade exporter till det område som alla moduler delar, så att varje band har samma fotavtryck. Av behåller hela sensorns arbetsyta (svart fyllning utanför källan).
* **Omprovtagning**:**Bilineär (jämn, standard)**,**Närmaste (bevarar exakta värden)**— ingen blandning mellan pixlar, för strikt radiometrisk analys — eller**Kubisk (skarpast)**.***

## Spara och ladda inställningar

### Spara projektmall

Skapa återanvändbara mallar för enhetliga arbetsflöden:

1. Konfigurera alla önskade inställningar i panelen Projektinställningar
2. Bläddra till avsnittet **”Spara projektmall”** längst ned
3. Ange ett beskrivande namn på mallen (t.ex. ”Survey3N\_RGN\_Agriculture”)
4. Klicka på spara-ikonen

**Fördelar:**

* Använd identiska inställningar i flera projekt
* Dela konfigurationer med teammedlemmar
* Upprätthåll enhetlighet vid upprepade undersökningar

### Ladda mallen till ett nytt projekt

När du skapar ett nytt projekt:

1. Välj **”Nytt projekt”** från huvudmenyn
2. Välj en projektmall i det valfria mallvalsfönstret
3. Alla inställningar från mallen tillämpas automatiskt

### Arbetsmapp

Inställningen **&quot;Arbetsmapp&quot;** anger var nya projekt skapas som standard:

* **Standardplats**: `C:\Users\[Username]\Chloros Projects`
* **Ändra plats**: Klicka på redigeringsikonen och välj en ny mapp
* **Delas med CLI**: `chloros-cli` använder samma standardinställning för projektmappen
* **När man bör ändra**:
  * Nätverksenhet för teamsamarbete
  * Annan enhet med mer lagringsutrymme
  * Organiserad mappstruktur efter år/kund

***

## PPK-inställningar (Post-Processed Kinematic)

Om du använder MAPIR DAQ-inspelare med GPS för exakt geolokalisering:

### Förutsättningar

* MAPIR DAQ med GPS-modul (GNSS)
* .daq-loggfil med exponeringstapp-poster
* Kamera ansluten till DAQ:s exponeringstappar under inspelningssessionen

### Konfigurationssteg

1. Placera .daq-loggfilen i din projektmapp
2. I Projektinställningar, markera kryssrutan **&quot;Tillämpa PPK-korrigeringar&quot;**

3. Ställ in**&quot;Tidszonsförskjutning för ljussensor&quot;** vid behov (standard: 0 för UTC)
4. Tilldela kameror till exponeringsstift:
   * **En kamera**: Tilldelas automatiskt till stift 1
   * **Två kameror**: Tilldela varje kamera manuellt till rätt stift**Tilldelning av exponeringsstift:*** **Exponeringsstift 1**: Välj kameramodell från rullgardinsmenyn
* **Exponeringsstift 2**: Välj den andra kameran eller ”Använd inte”
* Samma kamera kan inte tilldelas båda stiften

{% hint style="warning" %}
**Viktigt**: Exponeringsstiften måste tilldelas rätt till respektive kamera. Felaktig tilldelning leder till felaktiga geolokaliseringsdata.
{% endhint %}

***

## Avancerade scenarier

### Projekt med flera kameror

Vid bearbetning av bilder från flera MAPIR-kameror i ett projekt:

1. Chloros identifierar automatiskt varje kameramodell (både Survey3 och LATTICE)
2. Varje kamera tilldelas lämpliga bearbetningsprofiler och får sin egen utmatningsmappstruktur
3. PPK: Tilldela varje Survey3-kamera rätt exponeringsstift manuellt
4. Alla kameror använder samma exportformat och index

**Exempel**: Survey3W RGN + Survey3N OCN-rigg med två kameror, eller en LATTICE-uppställning som kombinerar en RGB-huvudkamera med smalbandsmoduler

### Tidsförlopps- eller flerdagarsundersökningar

För upprepade undersökningar av samma område över tid:

1. Skapa en mall med dina standardinställningar
2. Använd samma inställningar för kalibreringsmålet vid varje session
3. Bearbeta varje datum som ett separat projekt
4. Använd identiska inställningar för jämförbara resultat
5. Exportera i samma format för tidsmässig analys

### Stora datamängder

För projekt med många bilder (500+):

* Överväg att dela upp i mindre projekt efter datum eller område
* Använd Chloros+ för parallellbearbetning för snabbare resultat
* Överväg CLI eller API för automatisering av batchbearbetning
* Justera det minsta omkalibreringsintervallet för att minska tiden för måldetektering

***

## Kontrollera dina inställningar

Innan du påbörjar bearbetningen, kontrollera följande viktiga inställningar:

* [ ] Kameramodellen har identifierats korrekt i filbläddraren
* [ ] Vignettkorrigering aktiverad
* [ ] Reflektanskalibrering aktiverad
* [ ] För Survey3: minst en kalibreringsmålbild har importerats och kontrollerats; för LATTICE: ett mål och/eller en `.daq`-nedåtriktad inspelning finns
* [ ] Önskade multispektrala index har lagts till
* [ ] Exportformat som passar ditt arbetsflöde
* [ ] PPK-inställningar har konfigurerats (om du använder .daq med exponeringshändelser)

***

## Nästa steg

När dina inställningar är konfigurerade:

1. **Markera kalibreringsmålbilder** – Se [Välja målbilder](choosing-target-images.md)
2. **Starta bearbetningen** – Se [Starta bearbetningen](starting-the-processing.md)
3. **Övervaka förloppet** – Se [Övervaka bearbetningen](monitoring-the-processing.md)

För fullständiga detaljer om alla tillgängliga inställningar, se referensdokumentationen [Projektinställningar](../project-settings/project-settings.md).
