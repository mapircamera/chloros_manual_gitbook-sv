# Index/LUT Sandbox

Index/LUT Sandbox är ett interaktivt arbetsområde i Chloros Image Viewer som gör det möjligt att experimentera med multispektrala indexberäkningar och färgvisualiseringar i realtid. Detta kraftfulla verktyg hjälper dig att testa olika index, förfina värdeintervall och skapa publiceringsklara visualiseringar utan att behöva bearbeta hela datasetet på nytt.

## Vad är Index/LUT Sandbox?

### Syfte

Sandbox erbjuder:

* **Indexberäkning i realtid** – Tillämpa valfritt vegetationsindex direkt
* **Interaktiv LUT-justering** – Finjustera färgövergångar och intervall
* **Optimering av arbetsflödet** – Bestäm de bästa inställningarna innan batchbearbetning

### Sandbox vs. projektbearbetning

**Index/LUT Sandbox (interaktiv):**

* En bild i taget
* Omedelbar feedback
* Experimentell och iterativ
* Inga permanenta ändringar av filer
* Perfekt för utforskning och testning

**Projektbearbetning (batch):**

* Hela datasetet på en gång
* Förkonfigurerade inställningar
* Permanenta utdatafiler
* Tidskrävande
* Bäst när inställningarna är slutgiltiga

{% hint style=&quot;success&quot; %}
**Bästa arbetsflöde**: Använd Sandbox för att experimentera och hitta optimala index- och LUT-inställningar, och tillämpa sedan dessa inställningar under projektbearbetning för hela din dataset.
{% endhint %}

***

## Arbeta med Index/LUT Sandbox

### Förstå förberäknade index

I Chloros kan index tillämpas under projektbearbetningen. För att avgöra vilka index- och LUT-inställningar du vill tillämpa på exporter är det enklast att använda bildvisaren sandbox.

Sandboxen låter dig:

* **Tillämpa nya index och färgövergångar (LUT)** för att visualisera data
* **Justera visualiseringsinställningar** interaktivt
* **Visa** redan beräknade indexbilder
* **Inspektera** pixelvärden på alla zoomnivåer

### Öppna sandlådan

Index/LUT-sandlådan nås i **bildvisaren** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Klicka på en bild i bildrutnätet i filbläddraren så öppnas den i fliken **Bildvisare** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Klicka på fliken **Bildvisaren** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> för att öppna den vänstra popup-sidpanelen om den inte redan är öppen

### Välja en bild att tillämpa ett index/LUT på

För att arbeta med ett index i bildvisaren <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sandlåda:

1. **Öppna en bild** från huvudbildrutnätet genom att klicka på den
2. Fliken **Bildvisare** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> öppnas
3. Klicka på **Lager-rullgardinsmenyn** (uppe till höger i visaren)
4. Välj lager från rullgardinsmenyn:
   * RAW (Reflektans)

### Tillämpa ett index på en bild

När bilden är i helskärmsläge och sidofältet **Bildvisare** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> öppen:

1. Markera rutan Index längst upp i sidofältet.
2. Välj kamerans filter från rullgardinsmenyn till vänster.
3. Välj önskad indexformel från rullgardinsmenyn till höger.
4. Dra filterkanalens färgcirklar till platserna i indexformeln nedan.
5. När formeln är giltig uppdateras bilden och visar indexvärdena.
6. Flytta muspekaren för att se värdena vid muspekarens position.
7. Zooma in för att se enskilda pixlar och deras tillhörande värden.

Varje index har ett specifikt värdeintervall och en specifik betydelse:

#### NDVI Exempel

```

Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

För fullständig dokumentation om indexformler, se [Multispektrala indexformler](../project-settings/multispectral-index-formulas.md).

***

## Arbeta med LUT:er (uppslagstabeller)

### Vad är en LUT?

En **uppslagstabell (LUT)** mappar numeriska indexvärden till färger för visualisering:

* **Inmatning**: Indexpixelvärde (t.ex. NDVI 0,65)
* **Utgång**: RGB-färg (t.ex. ljusgrön)
* **Syfte**: Göra mönster lättare att se och tolka**Gråskala vs. färg-LUT:**

* Gråskala: Vetenskaplig och neutral, visar rådata
* Färg-LUT: Intuitiv och effektfull, lyfter fram mönster och skillnader

{% hint style=&quot;success&quot; %}
**Visualiseringskraft**: Genom att tillämpa en färg-LUT på en gråskalig indexbild blir det betydligt enklare att identifiera mönster, avvikelser och intressanta områden med ett ögonkast.
{% endhint %}

### Tillämpa en LUT på en indexbild

När du har en indexbild som visar

1. Klicka på <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> &quot;+Lägg till LUT&quot;
2. Välj färgövergången
3. Justera klippningens min-/maxändpunkter
4. Justera klippningsläget
5. Markera rutan Index i **Bildvisaren** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i sidofältet för att tillämpa LUT.

### Välja en färgövergång

**Välja en övergång:**

1. I LUT-panelen, leta reda på den**färgade övergångsstapeln**.
2. Håll muspekaren över den för att se tillgängliga förinställda övergångar.
3. Välj önskad övergång.
4. Bilden **uppdateras omedelbart** med nya färger när rutan Index är markerad.

{% hint style=&quot;success&quot; %}
**Bästa praxis**: För vegetationsindex som NDVI är gradienten Red-Yellow-Green mest intuitiv eftersom den stämmer överens med naturliga färgassociationer (grön = frisk, gul = måttlig, röd = stressad)..
{% endhint %}

### Justera färgklasser

**Klasskontrollen**avgör hur många diskreta färgsteg som visas i din gradient:**Alternativ för antal klasser:*** **2–5 klasser**: Mycket breda kategorier, distinkta zoner
* **6–10 klasser**: Balanserade, bra för klassificering
* **11–20 klasser**: Jämna gradienter, kontinuerligt utseende
* **20+ klasser**: Nästan kontinuerliga, maximal jämnhet**Så här justerar du:**

1. I LUT-panelen hittar du**färgproverna under gradientfältet**

2. Justera antalet klasser genom att lägga till med +-knappen
3. Ta bort antalet klasser genom att dubbelklicka på en färgpröv
4. Gradienten uppdateras **i realtid** på bilden**Effekt på visualiseringen:*** **Färre klasser** (3-5): Skapar distinkta zoner, förenklad klassificering, lättare att skilja mellan kategorier
* **Medelklasser** (6-10): Balanserad approach, bra för de flesta tillämpningar
* **Fler klasser** (15-20): Smidiga övergångar, detaljerad variation, fotografiskt utseende**När ska man använda:*** **Få klasser (3-5)**: Presentationsbilder, klassificeringskartor, enkla rapporter
* **Medelklasser (6-10)**: Allmän analys, balanserade detaljer, standardrapporter
* **Många klasser (15–20)**: Vetenskaplig analys, detaljerad inspektion, utdata av publikationskvalitet

### Finjustering av värdeintervall

**Värdeintervallskontrollerna**avgör vilka indexvärden som mappar till vilka färger i din gradient:**Intervallkontroller i LUT-panelen:*** **Minimivärde**: Nedre gräns för färgskalan
* **Maxvärde**: Övre gräns för färgskalan
* **Mellanliggande värden**: Fördelas automatiskt mellan min och max (baserat på antal klasser)

#### Justera min-/maxvärden

**Så här justerar du värdeintervall:**

1. I LUT-panelen letar du upp inmatningsfälten**Minvärde**och**Maxvärde**

2. Klicka på fältet**Minsta värde**

3. Ange önskat minsta värde (t.ex. `0.2`)
4. Tryck på **Enter** eller klicka utanför fältet
5. Upprepa för fältet **Maximalt värde** (t.ex. `0.9`)
6. Visualiseringen **uppdateras omedelbart**{% hint style=&quot;info&quot; %}**Automatisk skalning**: När du först tillämpar en LUT ställer Chloros automatiskt in min/max till det faktiska dataområdet i bilden. Du kan sedan begränsa detta område för att fokusera på specifika värdeintervall av intresse.
{% endhint %}

**Exempel på NDVI intervalljusteringar:*** **Hela intervallet**: `-1.0` till `1.0` (visa alla möjliga värden)
* **Vegetationsfokuserat**: `0.2` till `0.9` (uteslut bar mark och vatten)
* **Endast frisk vegetation**: `0.5` till `0.9` (markera endast livskraftiga växter)
* **Stressdetektering**: `0.2` till `0.5` (betona problemområden)
* **Anpassat intervall**: Justera utifrån dina observerade pixelvärden**Varför justera intervall?*** **Öka kontrasten** i ditt intresseområde
* **Uteslut irrelevanta värden** (t.ex. vattendrag, bar mark)
* **Standardisera visualiseringen** över flera bilder eller datum
* **Markera subtila skillnader** inom ett snävt värdeintervall

### Klippa bort värden utanför intervallet

När pixelvärdena ligger utanför ditt definierade min-/maxintervall kan du styra hur de visas med hjälp av **klippningslägen**.

#### **Tillgängliga alternativ för klippningsläge:**

#### 1. Minsta och största

* Pixlar **under minsta**→ visas med den**första färgen** i gradienten (t.ex. röd)
* Pixlar **över maximum**→ visas med den**sista färgen** i gradienten (t.ex. grön)
* **Användningsfall**: Betona extremvärden, visa hela dataområdet med mättade färger vid gränserna
* **Exempel**: NDVI-värden under 0,2 visas alla i rött, värden över 0,9 visas alla i grönt

#### 2. Transparent bakgrund

* Pixlar **utanför intervallet**blir**helt transparenta*** Endast pixlar **inom intervallet** visar färgövergång
* **Användningsfall**: GIS-överlägg, isolering av specifika värdeintervall, markering av endast områden av intresse
* **Exempel**: Visa endast NDVI 0,4-0,7 i färg, allt annat transparent

{% hint style=&quot;warning&quot; %}
**Begränsning av transparens**: Transparenta pixlar visas som bakgrundsfärg i visningsprogrammet. Vid export under bearbetning bevaras transparensen i PNG-format men inte i JPG.
{% endhint %}

#### 3. Indexbakgrund

* Pixlar **utanför intervallet**visas i**gråskala** (visar råa indexvärden)
* Pixlar **inom intervallet**visar**färgövergång*** **Användningsfall**: Subtil markering, bibehåller sammanhanget samtidigt som intressanta områden framhävs
* **Exempel**: Markera stressad vegetation med färg (NDVI 0,3–0,5) och visa friska områden i grått

#### 4. Originalbakgrund

* Pixlar **utanför intervallet**visas som**originalbilden i multispektralformat*** Pixlar **inom intervallet**visar**färgövergång*** **Användningsfall**: Mest intuitivt – kombinerar naturligt bildkontext med analytisk färgöverlagring
* **Exempel**: Se det faktiska utseendet på fältet/grödan med färgkodade stressade områden överlagrade

### Välja rätt klippningsläge

| Klippningsläge              | Bäst för                                   | Visualiseringsstil          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum och maximum**    | Fullständig datavisning, vetenskaplig analys     | Alla pixlar färgade           |
| **Transparent bakgrund** | GIS-överlägg, isolering av specifika intervall    | Färg inom intervallet, tomt utanför |
| **Indexbakgrund**       | Subtil betoning, bibehållande av datakontext  | Färg inom intervallet, grått utanför  |
| **Originalbakgrund**    | Rapporter, presentationer, intuitiv analys | Färg inom intervallet, foto utanför |

### Skapa anpassade LUT-färger

För full kontroll över din visualisering kan du skapa **anpassade färgövergångar** genom att redigera enskilda färgstopp.**Så här skapar du en anpassad övergång:**

1. I LUT-panelen hittar du**förhandsgranskningsfältet för övergångar**

2. Leta efter**färgprover** under övergången
3. **Klicka på ett färgstopp** för att välja det
4. En **färgväljare** öppnas
5. Välj en ny färg med hjälp av:
   * **Färghjul**: Visuell färgval
   * **RGB/HSV-reglage**: Exakt färgkontroll
   * **Hex-kodinmatning**: Exakt färgspecifikation (t.ex. `#FF0000` för rött)
6. Klicka utanför färgväljaren **för att tillämpa den nya färgen**

7. Gradienten**uppdateras omedelbart** på bilden**Lägga till eller ta bort färgstopp:*** **Lägg till ett stopp**: Klicka på +-ikonen för att lägga till en ny färgruta i slutet
* **Ta bort ett stopp**: Dubbelklicka på färgrutan för att ta bort färgprovet**Anpassningsstrategier:*** **Invertera gradient**: Vänd på färgordningen för att vända på betydelsen (t.ex. grönt = lågt, rött = högt)
* **Varumärkesfärger**: Anpassa rapportens färgpalett efter din organisations färgpalett
* **Färgblindvänligt**: Använd orange-blå eller lila-gula kombinationer
* **Utskriftsoptimering**: Välj färger som fungerar både vid färg- och gråskaleutskrift
* **Flera tröskelvärden**: Använd distinkta färger vid specifika värdetrösklar för klassificering

{% hint style=&quot;info&quot; %}
**Spara anpassade gradienter**: Anpassade gradienter kan sparas och återanvändas. Klicka på spara-ikonen i LUT-panelen för att spara dina anpassade färgscheman för framtida bruk.
{% endhint %}

***

## Interaktivt arbetsflöde

### Uppdateringar i realtid

Alla LUT-justeringar i sandlådan uppdaterar bilden **omedelbart och interaktivt**:

* **Byt lager** → Bilden ändras omedelbart
* **Välj gradient** → Färgerna uppdateras omedelbart
* **Justera värdeintervall** → Kontrasten ändras i realtid
* **Ändra klasser** → Gradientens jämnhet uppdateras omedelbart
* **Ändra beskärning** → Bakgrundsvisningen ändras omedelbart
* **Redigera färger** → Anpassad gradient tillämpas omedelbart**Ingen &quot;Tillämpa&quot;-knapp behövs** – alla ändringar är live och interaktiva!

{% hint style=&quot;success&quot; %}
**Live-feedback**: Den omedelbara visuella feedbacken gör att du snabbt kan experimentera med olika inställningar tills du hittar den optimala visualiseringen för dina analysbehov.
{% endhint %}

### Iterativ förfiningsarbetsflöde

**Typiskt LUT-optimeringsarbetsflöde:**

1.**Välj indexlager** (t.ex. RAW (Reflektans))
2. **Tillämpa index** – Välj kamerafilter och indexformel, dra färgade cirklar till lämplig plats i indexformeln
3. **Tillämpa LUT-gradient** – Börja med Red-Yellow-Green förinställning
4. **Kontrollera pixelvärden** – Flytta markören och notera värdeintervallen
5. **Justera min/max** – Begränsa för att fokusera på vegetation (t.ex. 0,2 till 0,9)
6. **Välj beskärning** – Prova ”Original Background” för sammanhang
7. **Förfina färger** – Anpassa gradienten vid behov för specifik betoning
8. **Slutför inställningarna**– Dokumentera inställningarna och kopiera till Projektinställningar för exportbearbetning

### Inspektion av pixelvärden

Det är viktigt att förstå de faktiska pixelvärdena för att kunna ställa in effektiva LUT-intervall:**Så här inspekterar du värden:**

1. Pixelvärden visas när bilden har antingen Index eller både Index och LUT**markerade**.
2. **Flytta markören** över olika områden i bilden
3. **Observera pixelvärdena** som visas i legenden när du håller muspekaren över dem
4. Zooma in för att se enskilda pixlar markerade med ett flytande värde
5. **Anteckna** värdeintervall för olika funktioner:
   * **Frisk vegetation**: t.ex. NDVI 0,55–0,85
   * **Stressad vegetation**: t.ex. NDVI 0,30–0,50
   * **Bar mark**: t.ex. NDVI 0,05–0,25
   * **Vatten** (om sådant förekommer): t.ex. NDVI -0,05 till 0,10**Använda pixelvärden för att ställa in LUT-intervall:**Efter att ha kontrollerat pixelvärdena justerar du LUT min/max därefter:**Exempel:*** **Observation**: Jordvärden = 0,05–0,25, stressad = 0,25–0,50, frisk = 0,50–0,85
* **Mål**: Visualisera endast växthälsa (exkludera jord)
* **LUT-inställningar**: Min = `0.25`, Max = `0.85`
* **Klippning**: ”Originalbakgrund” för att se jorden i naturlig färg
* **Resultat**: Färgövergången gäller endast vegetation, jorden visas som originalbild

{% hint style=&quot;info&quot; %}
**Dynamiskt intervall**: Olika grödor, årstider och tillväxtstadier har olika värdeintervall. Kontrollera alltid pixelvärdena i din specifika dataset innan du ställer in LUT-intervall.
{% endhint %}

***

## Anpassade index (Chloros+)

### Skapa anpassade indexformler

{% hint style=&quot;info&quot; %}
**Var du skapar dem**: Anpassade index kan konfigureras i**Projektinställningar** före bearbetning, samt i sidofältet i Image Viewer-sandboxen.
{% endhint %}

**Så här skapar du ett anpassat index:**

1.**Öppna Projektinställningar** (före bearbetning) eller bildvisarens sandlåda-sidfält
2. Navigera till **rullgardinsmenyn Indexformel**

3. Leta efter alternativet**&quot;Anpassad&quot;** (du måste vara inloggad med Chloros+-licens)
4. **Definiera din formel** med hjälp av bandvariabler:
   * Bandnamn: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, etc.
   * Operatörer: `+`, `-`, `*`, `/`, `^` (exponent)
   * Funktioner: `sqrt()`, `abs()`, etc. (om det stöds)
   * Parenteser: `()` för operationsordning
5. **Namnge ditt index** (t.ex. &quot;MyIndex&quot; eller &quot;CustomNDVI&quot;)
6. **Spara konfigurationen**

**Exempel på anpassade formler:**

```

Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style=&quot;warning&quot; %}
**Formelvalidering**: Se till att din formel använder band som är tillgängliga i din kamera. Till exempel är RedEdge endast tillgängligt på kameror med ett RedEdge-filter.
{% endhint %}

***

## Nästa steg

Nu när du förstår Index/LUT Sandbox:

* **Tillämpa på bearbetning**: Använd upptäckta inställningar i [Projektinställningar](../project-settings/project-settings.md)
* **Batchbearbetning**: Tillämpa optimerade index på hela datamängder
* **Läs mer**: Läs [Multispektrala indexformler](../project-settings/multispectral-index-formulas.md)

Relaterad dokumentation:

* [**Bildlager**](image-layers.md) – Lagerhantering och visualisering
* [**Öppna en bild i helskärmsläge**](opening-an-image-full-screen.md) – Grunderna i bildvisaren
* [**Bearbeta bilder (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Hela bearbetningsflödet
