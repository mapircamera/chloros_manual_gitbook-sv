# LATTICE-kameror

LATTICE är MAPIRs modulära multispektrala kamerasystem för bildtagning inom jordbruk och forskning. Varje LATTICE-kamera är baserad på Sony IMX265-sensorn med global slutare (**3,1 MP, 3,45 µm pixlar**) och ansluts via Ethernet som en**GigE Vision**-enhet.

Chloros 1.3.0 styr LATTICE-kamerorna i realtid – upptäckt, liveförhandsgranskning, bildtagning och synkroniserade multikamerauppställningar – från tre gränssnitt:

| Gränssnitt    | Var                                                          | Plattformar                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Fliken **Kameror** i Chloros-sidopanelen                         | Windows 10/11 x64                                        |
| CLI        | Kommandofamiljen `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Letar du efter hårdvaran?**Kameramoduler, objektiv, filter och band, ramar och fästen, kablar, PoE och utlösarkablar beskrivs i [**LATTICE-användarhandboken**](https://mapir.gitbook.io/lattice-camera). Detta kapitel behandlar styrning av kamerorna från Chloros.

LATTICE-bildfiler är standardfiler av typen `.tif`/`.tiff`, och Chloros bearbetar dem alltid utifrån rådata. Se [CLI-referensen](../reference/cli-reference.md) och [SDK-referensen](../reference/sdk-reference.md) för den fullständiga kommando- och API-ytan.

## Två sensorkonfigurationer

| Konfiguration | Sensor       | Filter                                | Vad en kamera levererar                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer-färg | trippelbandpassfilter                |**Tre kalibrerade band från en enda exponering**                 |
| **M3M**| Monokrom   | ett smalbandigt interferensfilter |**Ett kalibrerat band**; kombinera flera M3M-kameror för index |

Eftersom en M3M-kamera är monokrom bakom ett enda filter får varje band sin egen exponering. En M3C-kamera täcker alla tre banden med en enda sensorexponering.

## Modellsträngar och namngivning

Varje kamera lagrar sin identitet i GenICam `DeviceUserID` som en modellsträng:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros visar den med prefixet `LATT-` (till exempel `LATT-M3M-L87-F450`). Samma sträng `LATT-…` skrivs in i EXIF-taggen `Model` vid varje export och används som namn på kamerans utmatningsmapp i bearbetade projekt.

| Komponent | Värden                                                   | Betydelse                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Bayer-färg / monokrom                                                                          |
| Objektiv      | `L41` / `L87`                                            | Siffran anger **det horisontella synfältet i grader**: L41 = smalt (41°), L87 = brett (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) eller `F<nm>` (M3M) | Se [Filter och spektralband](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Modellsträngen styr allt nedströms: Chloroshämtar sensorprofilen, bandlayouten och fabrikskalibreringen från `DeviceUserID` + `DeviceSerialNumber`. Det finns inget att konfigurera per kamera — se [Ansluta kameror](connecting.md).

## Filter och band

Bandcentrum, FWHM-kanter och hela M3M-katalogen med 23 SKU:er är produktspecifikationer, så de finns i hårdvaruhandboken: [**Filter och spektralband**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Vad som är viktigt på mjukvarusidan: filterkoden i modellsträngen avgör vilka produkter Chloros kan skapa. RGB-filterkameror (`FRGB`) genererar endast debayered- och förhandsgranskningsprodukter — strålning och reflektans per band är inte meningsfulla för en bredbandssensor, så Chloros hoppar över dem och anger detta. Alla andra filter ger den fullständiga kedjan strålning → reflektans → index.

## Radiometrisk kalibrering i korthet

Varje LATTICE-kamera kalibreras individuellt på fabriken mot en NIST-spårbar kedja och levereras med ett certifikat för varje enskild kamera. Vad detta omfattar, hur det mäts och vilken noggrannhet du kan ange finns i hårdvaruhandboken: [**Fabriksradiometrisk kalibrering**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

När det gäller programvaran är det viktigt att Chloros fastställer rätt kalibrering när en kamera ansluts och låser de tillämpade koefficienterna vid varje export – se [Ansluta kameror](connecting.md).

## I detta kapitel

* [Ansluta kameror](connecting.md) – automatisk upptäckt, anslutningsdialogrutan i GUI, motsvarigheter i CLI och SDK, samt hur fabrikskalibreringen fastställs (kamerapaket vs. moln) när en kamera ansluts.
* [Kamerainställningar](camera-settings.md) — alla kontroller på fliken Kameror: kameralistan, inställningsrutan per kamera och inställningsrutan för matrisen (inklusive ROI-redigeraren och inriktning).
* [Inspelningsinställningar och lägen](capture.md) — Inspelning av allt, enkel-/kontinuerlig-/intervallinspelning, exporttyper och mappen för inspelningar.
* [Flerkameramatriser](arrays.md) — guiden för anslutning av matriser, synkroniseringskabel, bandbredd, PTP och livevyn för matrisen.
* [Monokameror och vegetationsindex](mono-indices.md) — M3M-kameror och arbetsflödet justera → stapla → indexera.

Den fullständiga kommandolistan finns i [CLI-referensen](../reference/cli-reference.md) och [SDK-referensen](../reference/sdk-reference.md).
