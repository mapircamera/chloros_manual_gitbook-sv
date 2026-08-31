---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreringsmål

MAPIR erbjuder olika kalibreringsmål för en rad olika tillämpningar. Den kompakta T4-R50 som visas nedan innehåller fyra paneler som har mätts med avseende på ljusreflektans från 250 till 2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>De diffusa referensmålen T4 har följande reflektanskurvor, [ladda ner data här](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflektans :: 250–2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4-reflektans :: 400–1 000 nm</p></figcaption></figure>De diffusa referensmålen T4P har följande reflektanskurvor, [ladda ner data här](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P-reflektans :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P-reflektans :: 400–1000 nm</p></figcaption></figure>Om man tittar på reflektansdiagrammet ser man att värdena visar våglängd (x-axeln) mot reflektans i procent (y-axeln). När vi tar en bild av kalibreringsmålet skapar vi sedan ett samband mellan pixelvärde och reflektans i procent, inom det spektrum som kamerans respektive sensorband är känsliga för.

Detta innebär att för varje bild du tar med våra kameror kan du använda en bild av våra reflektansmål, till exempel [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) eller [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), för att kalibrera bilderna med avseende på reflektans. Efter kalibreringen motsvarar varje pixel i bilden en procentuell reflektans.

För **Survey3** : Om du exporterar de kalibrerade bilderna i Chloros som vanliga JPG-filer eller TIFF beräknas reflektansprocenten genom att dividera pixelvärdet med bildformatets bitdjup. För JPG dividerar man alltså med 255 och för TIFF dividerar man med 65 535. Du kan också välja att exportera i PERCENT-format i Chloros, och då kommer varje pixel att ligga inom intervallet 0,0–1,0 procent (0–100 % reflektans). Tänk bara på att vissa bildprogram inte kan hantera bilder i procent (flyttalsvärden), och att de tar upp mycket lagringsutrymme.

{% hint style="info" %}
**LATTICE-reflektans använder en annan pixelskala.** LATTICE-reflektans lagras med DN 32768 = 100 % reflektans (inte 65535), och varje fil har en XMP-tagg (`Chloros:PixelScale`) som anger dess skala. Läs av taggen och dividera med den istället för att anta ett konstant värde – se [Utgångsbildformat](output-image-formats.md).
{% endhint %}

## Kalibreringsmål med LATTICE-kameror

Med LATTICE-kameror är ett kalibreringsmål **valfritt** för reflektans: Chloros kan istället referera reflektansen till den nedåtriktade irradians som mäts av en DAQ-ljussensor (ρ = π·L/E). Referensen väljs med inställningen för reflektanskälla (Projektinställningar i GUI; `--reflectance-source` i CLI; `reflectance_source` i SDK):

| Värde | Beteende |
| --- | --- |
| `auto` *(standard)* | Ett QA-godkänt mål inom ramen är den **absoluta referensen**; när inget mål finns eller QA misslyckas faller Chloros tillbaka till DAQ:s nedåtriktade delningsvärde. |
| `target` | Strikt endast mål — ingen DAQ-ersättning. |
| `daq` | DAQ-auktoritativt — nedåtriktad mätning är alltid referensen. |

Ytterligare målbeteende för LATTICE:

* **Målgeometrier** — ArUco-märkta paneler, paneler med fast ROI och remsmål stöds alla; geometrin hämtas från projektets målkonfiguration.
* **Mätdata per enhet** — `--target-reflectance-dir DIR` pekar på en katalog med mätdata för målets reflektans per enhet (`<serial>.csv`, som hämtas utifrån målenhetens serienummer/QR-kod). Vid miss faller Chloros tillbaka till de nominella T3/T4P-spektren.
* **Tidsankring** — ett detekterat mål kalibrerar bildrutorna runt sig och hålls kvar mellan målobservationerna.

Fullständig flaggsemantik och exempel finns i [CLI-referensen](reference/cli-reference.md) (se ”Exportväxlar per produkt”).

### F988

”F988-reflektansen kalibreras med hjälp av en reflektanspanel i scenen: bandet ligger utanför DAQ-ljussensorns kalibrerade område, så Chloros tillämpar din senaste panelinspelning och behåller den mellan panelobservationerna.”

Om F988 körs med enbart DAQ-kalibrering avvisar Chloros den DAQ-baserade reflektansen för det bandet och anger varför (hoppa över orsak `dls-uncalibrated-band-988`); arbetsflödet med panelen är den rekommenderade metoden.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
