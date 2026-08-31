# Bildrutnät

När du har importerat bilder till ett projekt visas de i ett rutnät i huvudområdet. I rutnätet väljer du **vilken version av varje bild du vill se** – knapparna ovanför växlar samtidigt alla miniatyrbilder mellan källfilerna och de bearbetade bilderna.

## Miniatyrstorlek

Använd zoomreglaget uppe till höger för att justera storleken på bildminiatyrerna. Reglaget går från **64 px till 1200 px**.

* **Ctrl + mushjulet** skalar också miniatyrerna.
* **Ctrl + `+`**/**Ctrl + `=`**och**Ctrl + `−`** ändrar storleken med 4 px per tryckning. Tangentbordsinställningarna slutar vid 64 px i den lägre änden och, i den högre änden, vid den storlek som rymmer exakt två miniatyrbilder per rad i det aktuella fönstret.
* Den storlek du väljer sparas med projektet (`UI → Grid thumbnail size` i `project.json`, standard `160`), så när du öppnar projektet igen återställs den.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>Miniatyrbildens *upplösning* är en separat inställning från miniatyrbildens *storlek*: se **Visning → Upplösning för miniatyrbilder** i [Projektinställningar](../project-settings/project-settings.md) (standard 512 px på den långa sidan). Storleken avgör hur stor rutan ritas; upplösningen avgör hur mycket detalj som hämtas för att fylla den.***

## Verktygsfältet för rutnätet

Knappraden ovanför rutnätet har upp till tre grupper, från vänster till höger:

1. **Per utlösare / Per kamera** — grupperingsläge. Visas endast för projekt som innehåller LATTICE-inspelningar.
2. **Kamerafilterknappar** — en per LATTICE-kamera. Visas endast i läget Per kamera.
3. **Knappar för export-/visningsläge** — vilken produkt varje miniatyrbild visar.

När fönstret är för smalt för att rymma alla knappar fälls grupperna ihop från höger till vänster till rullgardinsmenyer som visas vid muspekning: export-/visningsknapparna fälls ihop först, därefter kameraknapparna. Den ihopfällda gruppen lämnar kvar en enda knapp märkt med det för närvarande aktiva valet, och när man håller muspekaren över den skjuts hela uppsättningen fram. **”Per Trigger” och ”Per Camera” fälls aldrig ihop.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Knappar för export och visning

Dessa knappar växlar mellan olika bildtyper i rutnätet med miniatyrbilder. **En knapp visas så snart den produkt den namnger finns** — vilket för källfilerna innebär omedelbart vid import, inte efter bearbetning. Chloros skannar om projektets produkter medan en körning pågår, så knapparna visas under bearbetningen när varje produkt börjar sparas på disken.

### Basknappen

Den vänstra exportknappen är märkt med **vad du faktiskt importerade**:

| Vad du importerade | Knapptext |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| LATTICE-bildtagningar med en förhandsgranskning bredvid RAW-bilden | `PNG` eller `TIFF`, beroende på vilka förhandsgranskningarna är |
| LATTICE-bildtagningar där basfilen **är** RAW-bilden | *ingen knapp* — `RAW (Original)` visar redan den filen |

I ett blandat projekt följer etiketten den filändelse som används av flest bilder.

### Produktknappar

| Knapp | Visar | När den visas |
| --- | --- | --- |
| **Mål** | Bilder med ett upptäckt kalibreringsmål | Efter en körning där mål har upptäckts |
| **Reflektans** | Kalibrerade reflektansbilder | Endast Survey3-projekt — LATTICE-projekt använder istället `RAW (Reflectance)`, så rutnätet visar aldrig två reflektansknappar |
| **Vitbalanserad** | Den vitbalanserade produkten (RGB-kameror) | Efter bearbetning |
| **Vignettkorrigerad** | Den okalibrerade vignettkorrigerade reservinställningen | Efter en körning där reflektanskalibrering inte kunde tillämpas och *Vignettkorrigering* var aktiverad |
| **Sensorsvar** | Den okalibrerade reservlösningen med sensorsvar | Samma, men med *Vignettkorrigering* avstängd |
| **`RAW (<INDEX> Index)`** | En knapp per beräknat index | Efter en körning med konfigurerade index |
| **`<INDEX> LUT`** | En knapp per färgkartlagt index | Efter en körning med en konfigurerad LUT |
| **`<Index> <Index\|LUT> <NNN>`** | En knapp per exportkörning av [Index/LUT Sandbox](index-lut-sandbox.md) | I det ögonblick en sandbox-export avslutas |

### Knappar på LATTICE-nivå

Projekt som innehåller LATTICE-inspelningar lägger till dessa, märkta med nivånamnet istället för ett produktnamn:

| Knapp | Nivå |
| --- | --- |
| **RAW (Original)** | Den ursprungliga råbilden, så som den importerades |
| **RAW (Strålning)** | Spektral strålning i Float32, W/m²/sr/nm |
| **RAW (Reflektans)** | Reflektans i uint16, 32768 = ρ 1,0 |

`RAW (Original)` är tillgängligt direkt vid importen — det kräver ingen bearbetning. När en LATTICE-import inte har någon basknapp alls (varje bilds basfil är dess råbild), flyttar sig rutnätet automatiskt till den första tillgängliga nivåknappen så att markeringen i verktygsfältet stämmer överens med det du ser.

Två nivåer av Chloros-exporter får **ingen egen rutnätsknapp**:

* **Debayered** — `RAW (Original)`-vyn renderas redan debayered, så en andra knapp på en visuellt identisk bild skulle vara onödig. `RAW (Debayered)`-produkten skrivs fortfarande till disken och kan fortfarande väljas från rullgardinsmenyn för helskärmslager.
* **Förhandsgranskning** — på RGB-kameror registreras förhandsgranskningen som lagret `White Balanced`, som har en knapp. På multispektrala kameror registreras den som `RAW (Preview)` och kan nås från rullgardinsmenyn för helskärmslager.

{% hint style="info" %}
Dessa nivåknappar visas endast för projekt som faktiskt innehåller LATTICE-ramar. Survey3-projekt registrerar några av samma interna lagernamn, och knapparna filtreras bort för dem så att ett Survey3-rutnät behåller sin välbekanta `JPG / Targets / Reflectance`-uppsättning.
{% endhint %}

Om du klickar på en rutnätsminiatyr öppnas [Bildvisaren](opening-an-image-full-screen.md) i helskärmsläge för **samma produkt som rutnätet visar** — om rutnätet är inställt på `Targets` öppnar miniatyrbilden den exporterade målbilden.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Gruppering av ett LATTICE-projekt: Per utlösare vs. per kamera

Array-tagningar ger flera bilder av samma ögonblick från olika kameramoduler. Grupperingen avgör hur rutnätet staplar dem. Båda lägena visar hopfällbara rubrikfält i full bredd; **varje grupp är utfälld från början**, och Chloros kommer ihåg vilka du stänger. Hopfällningsstatus spåras separat per läge, så att stänga en grupp i Per kamera stänger inte något i Per utlösare.

### Per kamera (standard)

En grupp per kameramodul. Rubriken visar kameramodellen och serienumret (`LATT-M3M — <serial>`) samt antalet bilder. Bildrutorna inom en grupp ordnas kronologiskt efter när bilden togs.

I detta läge får verktygsfältet också en **kamerafilterknapp per kamera**, märkt `MODEL (SERIAL)`. Alla kameror är markerade från början; genom att klicka på en knapp avmarkeras den kameran och dess grupp tas bort från rutnätet. Detta är det snabbaste sättet att granska ett band över en hel flygning.

### Per utlösning

En grupp per fotograferingstillfälle – den uppsättning bildrutor som alla moduler tog vid samma utlösning. Rubriken visar tidpunkten för fotograferingen, antalet kameror som bidragit samt en ikon för varje kameramodell i gruppen. Rutorna inom en grupp är ordnade efter kamerans serienummer, så att samma band hamnar i samma kolumn för varje utlösning.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Icke-LATTICE-bilder i ett blandat projekt grupperas inte – de visas som vanliga rutor efter grupperna.

***

## Miniatyrerna i rutnätet följer GSD-blockstorleken

Om du har ställt in en **GSD (px)**-blockstorlek i sidofältet på fliken Bild, visas rutnätets miniatyrbilder med samma markupplösning – inte bara i helskärmsvyn. En blockstorlek på 8 innebär att varje visad pixel är medelvärdet av ett 8 × 8-block av källpixlar, överallt i appen där bilden visas.

Eftersom en ruta från början bara är några hundra pixlar bred, slutar grova blockstorlekar att göra någon synlig skillnad i rutnätet långt innan de gör det i helskärmsvyn: en ram på 4000 px som ritas in i en ruta på 160 px har redan ungefär 25 källpixlar per visad pixel. Se [Öppna en bild i helskärmsläge](opening-an-image-full-screen.md#gsd-block-size) för själva kontrollen.

***

## Relaterade sidor

* [**Öppna en bild i helskärmsläge**](opening-an-image-full-screen.md) — helskärmsvisaren, markörvärden och histogram
* [**Bildlager**](image-layers.md) — lagermenyn i helskärmsvisaren
* [**Index/LUT-sandlåda**](index-lut-sandbox.md) — skapa och exportera indexvisualiseringar
* [**Projektinställningar**](../project-settings/project-settings.md) — exportknapparna som avgör vilka produkter som överhuvudtaget finns
