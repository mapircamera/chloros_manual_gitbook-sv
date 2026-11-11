# GUI: Navigering

När du startar Chloros för första gången startar programmet sin bakgrundsprocess. Så snart bakgrundsprocessen är klar visas huvudmenyikonen uppe till vänster <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> och flikarna ”Kameror” och ”Ljussensorer” blir tillgängliga i vänster sidofält (de är inaktiverade fram till dess).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Från vänster till höger innehåller den övre rubriken:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Huvudmeny

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>Från huvudmenyn kan du:

* **Nytt projekt**— skapa ett nytt projekt. Om du har sparat projektmallar visas en rullgardinsmeny med alternativet**Välj mall**, så att det nya projektet startar med inställningarna från en mall.
* **Öppna projekt**— öppna ett befintligt projekt. Listan innehåller en knapp**Öppna projektmapp** som öppnar projektmappen i din filutforskare.
* **Duplicera projekt** — kopiera det för närvarande öppna projektet under ett nytt namn (ett ledigt namn som ”MittProjekt (2)” föreslås) och öppna kopian. _(syns när ett projekt har öppnats)_
* **Lägg till filer** — lägg till enskilda bildfiler till det aktuella projektet _(syns när ett projekt har öppnats)_
* **Lägg till mapp** — lägg till en eller flera mappar med bilder till det aktuella projektet _(syns när ett projekt har öppnats)_
* **Starta bearbetning / Avsluta bearbetning** — starta eller avsluta bildbearbetningskedjan _(aktiveras när filer har lagts till)_
* **Anslut till kamera** — gå till [fliken Kameror](lattice/) för att ansluta en LATTICE-kamera eller -array. Fungerar även utan att ett projekt är öppet.
* **Anslut till ljussensor** — gå till [fliken Ljussensorer](daq/) för att ansluta en DAQ-ljussensor. Fungerar även utan att ett projekt är öppet.

{% hint style="info" %}
**Endast Windows**: Chloros

Desktop GUI är tillgängligt på Windows

. Linux

-användare bör se dokumentationen för [CLI

](CLI.md) och [Python

SDK

](api-python-sdk.md) för bearbetning utan grafiskt gränssnitt.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Knappen Spela upp/Starta

När den är aktiverad startar knappen för bearbetning bildbearbetningspipeline.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Förloppsindikator <img src=".gitbook/assets/image (5).png" alt="" data-size="line">



I det kostnadsfria Chloros

-läget, där alla filer bearbetas sekventiellt, visar förloppsindikatorn två steg: Måldetektering och Bearbetning.

I det licensierade Chloros

+-läget, där alla filer bearbetas samtidigt, visar förloppsindikatorn fyra steg: Detektering, Analys, Kalibrering, Export. Om du håller muspekaren över Chloros

+-förloppsindikatorn öppnas en utökad panel med fyra förloppsindikatorer så att du kan följa processen. Om du klickar på den översta förloppsindikatorn fryses rullgardinsmenyn, och om du klickar igen frigörs den.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Sidomeny

Menyn i vänster sidopanel innehåller olika ikoner att interagera med, i följande ordning uppifrån och ner:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projektinställningar](project-settings/project-settings.md)

På fliken Projektinställningar kan du justera globala projektinställningar och inställningar för projektbearbetning. Justera dessa innan du börjar bearbeta dina filer.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Filbläddrare

Lägg till filer/mappar och ta bort filer från projektet. Dubbletter ignoreras. Markera rutan i målkolumnen för varje målbild, så kommer bearbetningen endast att söka efter mål bland de markerade bilderna, vilket avsevärt påskyndar bearbetningstiden. Använd växlingsknappen Bild/Metadata för att växla mellan att visa miniatyrrutnätet för den valda bilden och en detaljerad metadatatabell.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Bildvisare](image-viewer-gui/opening-an-image-full-screen.md)

När du klickar på en bild i huvudbildvisaren öppnas den i helskärmsläge på fliken Bildvisare.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Kartvisare](image-viewer-gui/map-markers.md)

Visa dina bilder på en interaktiv 2D-karta baserad på deras GPS-koordinater. Stöder Google Maps och ESRI-kartleverantörer och väljer automatiskt den bästa tjänsten för din plats. Håll muspekaren över markörerna för att se förhandsgranskningar av bildminiatyrer.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kameror](lattice/)

Anslut och styr LATTICE-kameror i realtid – en i taget eller som synkroniserade multikamerauppsättningar. Fliken visar liveförhandsgranskningsrutor med överlägg och histogram, inställningar per kamera och per uppsättning samt inspelningsinställningar som avgör vilka kameror och exporttyper som ”Capture All” genererar. Tillgängligt när backend-systemet är klart; se [LATTICE-avsnittet](lattice/) för en fullständig genomgång.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Ljussensorer](daq/)

Anslut DAQ-ljussensorer — DAQ-U (USB), DAQ-M (Bluetooth) och DAQ-E (Ethernet) — och visa deras kalibrerade spektrumdiagram i realtid i W/m²/nm. Härifrån kan du spara `.daq`-filer i det öppna projektet, byta namn på sensorer, välja profiler för kapacitanskorrigering och uppdatera DAQ-E-firmware. Tillgängligt när backend-systemet är klart; se [DAQ-avsnittet](daq/) för en fullständig genomgång.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Felsökningslogg

Granska loggen för felsökningsutskrifter när problem uppstår. Kopiera/ladda ner loggen och skicka den till [MAPIR-supporten](https://www.mapir.camera/community/contact) för hjälp.

Nytt i 1.3.0: buggikonen högst upp på fliken Logg (**Rapportera en bugg**) skickar en rapport till MAPIR med din beskrivning, upp till 10 bilder, Chloros-loggarna och systeminformation. Du måste vara inloggad för att kunna skicka en rapport. Se [Rapportera ett fel från loggfliken](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Användarinloggning](chloros+-login.md)

I sidopanelen för användarinloggning kan du logga in på ditt Chloros+-konto för att låsa upp avancerade funktioner. Du kan också se den aktuella programversionen samt ställa in språket för den text som visas i Chloros GUI och CLI.
