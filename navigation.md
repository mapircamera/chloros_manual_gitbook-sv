# GUI: Navigering

När du startar Chloros och Chloros (webbläsare) för första gången startas dess backend. När den är klar visas huvudmenyikonen uppe till vänster <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Från vänster till höger innehåller den övre rubriken:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Huvudmeny

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Från huvudmenyn kan du:

* **Nytt projekt** — skapa ett nytt projekt
* **Öppna projekt** — öppna ett befintligt projekt
* **Öppna projektmapp** — öppna projektmappen i din filutforskare
* **Lägg till filer** — lägg till enskilda bildfiler till det aktuella projektet _(synligt efter att ett projekt har öppnats)_
* **Lägg till mapp** — lägg till en mapp med bilder till det aktuella projektet _(synlig efter att ett projekt har öppnats)_
* **Starta bearbetning / Stoppa bearbetning** — starta eller stoppa bildbearbetningsprocessen _(aktiveras efter att filer har lagts till)_

{% hint style="info" %}
**Endast Windows**: Chloros Desktop GUI är tillgängligt på Windows. Linux-användare bör se [CLI](CLI.md) och [Python SDK](api-python-sdk.md) för headless-bearbetning.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Spela upp/Start-knapp

När den är aktiverad startar knappen för bearbetningsstart bildbearbetningspipeline.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Förloppsindikator <img src=".gitbook/assets/image (5).png" alt="" data-size="line">I det kostnadsfria läget Chloros, som bearbetar alla filer sekventiellt, visar förloppsindikatorn två steg: Målidentifiering och Bearbetning.

I det betalda läget Chloros+, som bearbetar alla filer samtidigt, visar förloppsindikatorn fyra steg: Identifiering, Analys, Kalibrering, Export. Om du håller muspekaren över Chloros+-förloppsindikatorn visas en utökad panel med fyra steg så att du kan följa processen. Om du klickar på den övre förloppsindikatorn fryses rullgardinsmenyn, och om du klickar igen frigörs den.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Sidomeny

Menyn i vänster sidfält innehåller olika ikoner som du kan interagera med:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projektinställningar](project-settings/project-settings.md)

På fliken Projektinställningar kan du justera globala inställningar och inställningar för projektbearbetning. Justera dessa innan du börjar bearbeta dina filer.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Filbläddraren

Lägg till filer/mappar och ta bort filer från projektet. Dubbletter ignoreras. Markera rutan i målkolumnen för varje målbild, så kommer bearbetningen endast att söka efter mål i de markerade bilderna, vilket avsevärt påskyndar bearbetningstiden. Använd växlingsknappen Bild/Metadata för att växla mellan att visa miniatyrrutnätet för den valda bilden och en detaljerad metadatatabell.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Bildvisare](image-viewer-gui/opening-an-image-full-screen.md)

När du klickar på en bild i huvudbildvisaren öppnas den i helskärmsläge på fliken Bildvisare.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Karta](image-viewer-gui/map-markers.md)

Visa dina bilder på en interaktiv 2D-karta baserad på deras GPS-koordinater. Stöder Google Maps och ESRI-kartleverantörer och väljer automatiskt den bästa tjänsten för din plats. Håll muspekaren över markörer för att se förhandsgranskningar av bildminiatyrer.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Felsökningslogg

Granska loggen för felsökningsutskrifter när problem uppstår. Kopiera/ladda ner loggen och skicka den till [MAPIR Support](https://www.mapir.camera/community/contact) för hjälp.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Användarinloggning](chloros+-login.md)

I sidomenyn för användarinloggning kan du logga in på ditt Chloros+-konto för att låsa upp avancerade funktioner. Du kan också se den aktuella applikationsversionen samt justera språket för den text som visas i Chloros GUI och CLI.
