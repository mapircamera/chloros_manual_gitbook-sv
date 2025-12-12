---
description: Labbuppmätta paneler som används för att kalibrera infångade data i efterbehandling
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreringsmål

MAPIR erbjuder olika kalibreringsmål för att täcka en rad applikationer. Den kompakta T4-R50 som visas nedan innehåller 4 paneler som har uppmätts för ljusreflektans från 250 - 2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

De diffusa T4-referensmålen har följande reflektanskurvor, [ladda ner data här](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflection :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflection :: 400-1000nm</p></figcaption></figure>

Om du tittar på reflektansgrafen kan du se att värdena är våglängd (x-axel) kontra reflektansprocent (y-axel). När vi tar en bild av kalibreringsmålet skapar vi ett förhållande mellan pixelvärde och reflektansprocent, inom det spektrum som var och en av kamerans sensorband är känsliga för.

Det betyder att med varje bild du tar med våra kameror kan du använda ett foto av våra reflektionsmål, som [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) eller [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) för att kalibrera bilderna för reflektion. En gång kalibrerad är varje pixel i bilden lika med procentuell reflektans.

Om du matar ut de kalibrerade bilderna i Chloros som typisk JPG eller TIFF beräknas reflektansprocenten genom att dividera pixelvärdet med bildformatets bitdjup. Så för JPG dividera med 255 och för TIFF dividera med 65 535. Du kan också välja formatet PROCENT i Chloros, och då kommer varje pixel att variera från ett procentvärde på 0,0 till 1,0 (0 % till 100 % reflektans). Tänk bara på att vissa bildapplikationer inte kan acceptera procentuella (flytande komma) bilder, och de är stora i storlek lagringsmässigt.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption>__TAG <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
