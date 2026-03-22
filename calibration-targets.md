---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreringsmål

MAPIR erbjuder olika kalibreringsmål för en rad olika tillämpningar. Den kompakta T4-R50 som visas nedan innehåller fyra paneler som har mätts med avseende på ljusreflektans mellan 250 och 2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4-referensmålen för diffus reflektion har följande reflektionskurvor, [data kan laddas ned här](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflektans :: 250–2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4-reflektans :: 400–1 000 nm</p></figcaption></figure>T4P-referensmålen för diffus reflektion har följande reflektanskurvor, [ladda ner data här](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P-reflektans :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P-reflektans :: 400–1000 nm</p></figcaption></figure>Om du tittar på reflektansdiagrammet ser du att värdena är våglängd (x-axeln) mot reflektansprocent (y-axeln). När vi tar en bild av kalibreringsmålet skapar vi sedan ett samband mellan pixelvärde och reflektansprocent, inom det spektrum som varje av kamerans sensorband är känsligt för.

Detta innebär att för varje bild du tar med våra kameror kan du använda en bild av våra reflektansmål, såsom [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) eller [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), för att kalibrera bilderna för reflektans. När kalibreringen är klar motsvarar varje pixel i bilden en procentuell reflektans.

Om du exporterar de kalibrerade bilderna i Chloros som vanliga JPG- eller TIFF-filer beräknas reflektansprocenten genom att dividera pixelvärdet med bildformatets bitdjup. För JPG dividerar du alltså med 255, och för TIFF dividerar du med 65 535. Du kan också välja utdataformatet PERCENT i Chloros, och då kommer varje pixel att ligga i intervallet 0,0 till 1,0 procent (0 % till 100 % reflektans). Tänk bara på att vissa bildprogram inte kan hantera bilder i procent (flyttalsformat), och att de tar stor lagringsutrymme.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
