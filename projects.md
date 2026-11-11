# GUI: Projekt Med

Chloros kan du skapa projekt som kan öppnas igen senare. Ett projekt är en vanlig mapp (i din projektmapp) som innehåller:

* `project.json` — projektinställningar, fillista och visningsinställningar
* `cameras.json` — kameror och sensormatriser som var anslutna medan projektet var öppet, med tillhörande inställningar
* `sensors.json` — DAQ-ljussensorer som var anslutna medan projektet var öppet, samt kopplingar mellan kameror och sensorer
* dina bilder, `.daq`-inspelningar och mappar med bearbetade resultat

Det finns inget proprietärt projektfilformat — mappen och dess JSON-filer utgör projektet, vilket också gör det enkelt att kopiera, arkivera och styra projekt från [CLI](CLI.md) eller [Python SDK](api-python-sdk.md).

Om du inte ändrar det i [Projektinställningar](project-settings/project-settings.md) är projektmappen `Documents\MAPIR\Chloros Projects` i mappen Dokument (`~/Documents/MAPIR/Chloros Projects` på Linux).

{% hint style="warning" %}
**Windows: håll projektmapparna korta såvida inte långa sökvägar är aktiverade.**Windows levereras med långa sökvägar inaktiverade, och då får ingen filväg vara längre än 260 tecken. Bearbetade produkter placeras upp till cirka 130 tecken under projektmappen, så från och med version 1.3.0 avvisar Chloros en projektmapp som ligger för djupt – när du skapar eller öppnar den, eller när du startar bearbetningen – istället för att köras utan att skriva ut något. Meddelandet anger den längsta projektmappen som passar. Använd en kortare projektmapp eller aktivera långa sökvägar i Windows (installationsprogrammets alternativ**Aktivera långa sökvägar i Windows**, markerat som standard) och starta om Chloros. Se [Starta bearbetningen](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Nytt projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Välj ”Nytt projekt” från huvudmenyn och ange ett unikt namn för ditt projekt.

Om du har sparat några projektmallar visas en rullgardinsmeny med **Välj mall** under namnfältet – om du väljer en mall startar det nya projektet med inställningarna från den mallen. Mallar sparas från [Projektinställningar](project-settings/project-settings.md): ange ett namn i fältet ”Spara projektmallens namn” och klicka på spara-ikonen.

## Öppna projekt

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>”Öppna projekt” visar alla projekt i din projektmapp, med <strong>”Öppna projektmapp”</strong> längst ned</p></figcaption></figure>

Välj ”Öppna projekt” för att se en lista över befintliga projekt i projektmappen. Om inga projekt finns öppnas inte den sekundära sidomenyn. Du kan se några projekt som skapats via GUI (t1, t2, t3) listade på bilden ovan. Projekten DATE\_TIME skapades av CLI:n med hjälp av standardnamngivningsschemat för projekt. Om du klickar på ett projektnamn öppnas projektet.

Om du klickar på knappen ”Öppna projektmapp” öppnas datorns filutforskare vid projektets sökväg. Du kan justera projektvägen i [Projektinställningar](project-settings/project-settings.md).

Om någon av projektets källbildsfiler har flyttats eller raderats sedan det senast öppnades visar Chloros en dialogruta som listar exakt vilka filer som saknas istället för att öppna ett tomt rutnät.

## Duplicera projekt

Funktionen är tillgänglig när ett projekt är öppet. Välj ”Duplicera projekt” för att kopiera det aktuella projektet under ett nytt namn – Chloros föreslår nästa lediga namn (t.ex. ”MittProjekt (2)”) – och kopian öppnas omedelbart.

## Lägg till filer

När ett projekt har öppnats väljer du ”Lägg till filer” i huvudmenyn för att lägga till enskilda bildfiler till det aktuella projektet. Detta fungerar på samma sätt som filutväljarens funktion för att lägga till filer, men är för enkelhetens skull tillgängligt direkt från huvudmenyn.

## Lägg till mapp

När ett projekt har öppnats väljer du ”Lägg till mapp” i huvudmenyn för att lägga till mappar med bilder till det aktuella projektet. Du kan välja flera mappar på en gång. Dubbletter av filer ignoreras.

## Starta/stoppa bearbetning

När filer har lagts till i ett projekt blir ”Starta bearbetning” tillgängligt i huvudmenyn. Detta motsvarar att klicka på knappen Spela upp/Starta i den övre rubriken. Under bearbetningen ändras menyalternativet till ”Stoppa bearbetning” så att du kan avbryta bearbetningsflödet.

## Anslut till kamera / Anslut till ljussensor

Längst ner i huvudmenyn finns två hårdvarugenvägar, tillgängliga oavsett om ett projekt är öppet eller inte:

* **Anslut till kamera** — öppnar [fliken Kameror](lattice/) för att ansluta en LATTICE-kamera eller -array.
* **Anslut till ljussensor** — öppnar [fliken Ljussensorer](daq/) för att ansluta en DAQ-ljussensor.

Om du ansluter hårdvara medan ett projekt är öppet sparas anslutningen i projektet (se nedan). Utan ett projekt gäller anslutningarna endast för den aktuella sessionen.

{% hint style="info" %}
Menyalternativen **Lägg till filer**,**Lägg till mapp**och**Starta/stoppa bearbetning**är endast synliga eller aktiverade när ett projekt är öppet och filer har lagts till. De ger snabb åtkomst till åtgärder som också är tillgängliga via sidopanelen**Filbläddraren** och knapparna i sidhuvudet.
{% endhint %}

## Projekt kommer ihåg din hårdvara

Nytt i 1.2.0: ett projekt behåller den hårdvara du ansluter så länge det är öppet. Kameror och matriser (med inställningar per kamera, namn, färger och rutnätslayout) sparas som en ögonblicksbild i `cameras.json`, och ljussensorer (med namn, färger och kamerakopplingar) i `sensors.json` — automatiskt, medan du arbetar.

När du **öppnar** ett projekt igen kommer Chloros inte omedelbart att ansluta till någon hårdvara. Varje del återansluts första gången du besöker den flik som den tillhör:

* När du öppnar fliken **Kameror** återansluts de sparade kamerorna och matriserna och deras sparade inställningar tillämpas på nytt.
* När du öppnar fliken **Ljussensorer**återansluts de sparade DAQ-sensorerna och deras sparade inställningar tillämpas på nytt, inklusive kåpa. En sensor som är inställd på**Ingen (bar sensor)** i sitt Cap-meny förblir bar. Ett projekt som sparats av en tidigare version av Chloros registrerade en bar sensor som inget val, så den återkommer med solskenskosinuskorrigering; ställ in den på Ingen igen och spara projektet.

På detta sätt aktiveras aldrig kamerornas strömning när du öppnar ett projekt enbart för att bläddra bland eller exportera bilder. Om en sparad enhet inte kan hittas när dess flik öppnas, visar en dialogruta vilka enheter som inte är tillgängliga så att du kan återansluta eller ta bort dem.

## DAQ-inspelningar och .daq-filer i ett projekt

* `.daq`-inspelningar som görs medan projektet är öppet (från fliken Ljussensorer eller under inspelningar) **läggs automatiskt till i projektet**.
* Importerade `.daq`-filer och alla projektinspelningar listas i avsnittet **DAQ-ljussensor** under [Projektinställningar](project-settings/project-settings.md), var och en med sin egen profil för kap-korrigering.
* Under bearbetningen tillhandahåller projektets `.daq`-filer nedåtriktad belysning för reflektansprodukter – se [Utgångsbildformat](output-image-formats.md).

## Körning av ett sparat projekt utan grafiskt gränssnitt

Ett sparat projekt kan köras utan det grafiska gränssnittet:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` arbetar med en projektmappväg – se [CLI-referens](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` returnerar ett projekthandtag; `connect_all()` aktiverar alla sparade kameror, matriser och ljussensorer med deras sparade inställningar (ljussensorer via en körande Chloros-backend) — se [SDK-referensen](reference/sdk-reference.md).
