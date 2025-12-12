---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreringsmål

MAPIR erbjuder olika kalibreringsmål för en rad olika tillämpningar. Den kompakta T4-R50 som visas nedan innehåller fyra paneler som har mätts för ljusreflektans från 250 till 2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>De diffusa referensmålen T4 har följande reflektionskurvor, [data nedladdning här](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflektans :: 250–2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflektans :: 400–1 000 nm</p></figcaption></figure>Om du tittar på reflektansdiagrammet kan du se att värdena är våglängd (x-axeln) mot reflektansprocent (y-axeln). När vi tar en bild av kalibreringsmålet skapar vi sedan ett samband mellan pixelvärde och reflektansprocent, inom det spektrum som kamerans sensorbands är känsliga för.

Det innebär att för varje bild du tar med våra kameror kan du använda ett foto av våra reflektansmål, till exempel [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) eller [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) för att kalibrera bilderna för reflektans. Efter kalibrering motsvarar varje pixel i bilden procentuell reflektans.

Om du exporterar de kalibrerade bilderna i Chloros som vanlig JPG eller TIFF beräknas reflektansprocenten genom att dividera pixelvärdet med bitdjupet i bildformatet. För JPG dividerar du alltså med 255 och för TIFF dividerar du med 65 535. Du kan också välja utmatningsformatet PERCENT i Chloros, och då kommer varje pixel att ha ett procentvärde mellan 0,0 och 1,0 (0 % till 100 % reflektans). Tänk bara på att vissa bildprogram inte kan hantera procentbilder (flyttal) och att de tar mycket lagringsutrymme.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
