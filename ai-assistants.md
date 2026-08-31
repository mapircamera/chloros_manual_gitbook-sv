# Användning av Chloros med AI-assistenter

Denna handbok riktar sig till två målgrupper: människor och de AI-assistenter som människor i allt högre grad arbetar via. Varje sida innehåller exakta värden, standardinställningar och kommandon som kan kopieras och klistras in, så att en assistent (Claude, ChatGPT, Copilot, en kodningsagent, …) kan skriva fungerande Chloros-automatisering redan vid första försöket.

Chloros-version: **

1.2.0**. CLI/SDK-plattformar: Windows 10/11 x64 och Linux (x86\_64 / Jetson aarch64).

## Vad du ska ge din assistent

| Resurs | URL | Vad den är till för |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Maskinläsbart index över varje sida i denna handbok. |
| **CLI Referens** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Den fullständiga `chloros-cli`-kommandoytan: alla kommandon, flaggor, standardvärden, avslutningskoder och regler för utdatamappar. Skriven för användning av LLM. |
| **SDK-referens** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Den fullständiga `chloros_sdk` Python API: klasser, signaturer, undantag och praktiska exempel. Skrivet för LLM-användning. |
| **Vilken sida som helst som rå Markdown** | lägg till `.md` till sidan URL | t.ex. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` returnerar sidan som ren Markdown — perfekt för att klistra in i ett kontextfönster eller hämta från en agent. |

Länkar i manualen: [CLI Referens](reference/cli-reference.md) · [SDK Referens](reference/sdk-reference.md).

{% hint style="info" %}
De två referenssidorna är fristående: en assistent som har läst en av dem behöver inte resten av manualen för att skriva ett korrekt skript.
{% endhint %}

## Färdiga recept

Kopiera, fyll i `<placeholders>` och klistra in i din assistent.

### 1. Bearbeta en flygmapp till NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Övervaka en katalog med inspelningar i batch

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Anslut en LATTICE-array och spela in

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Registrera spektra från DAQ-ljussensorer

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
DAQ-skript från kommandoraden går alltid via `daq pool-*`-familjen (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Andra `daq`-underkommandon som din assistent kan hitta på finns inte i de levererade versionerna och avslutas med ett felmeddelande.
{% endhint %}

## Varför AI-skrivna skript fungerar bra med Chloros

Var och en av dessa är ett verkligt, verifierat beteende hos Chloros 1.2.0 — de eliminerar de klassiska felkällorna hos maskinskrivna automatiseringar:

* **Inga krångliga inställningar.**SDK:s smart-connect-hjälpfunktioner (`connect_camera`, `connect_array`, `connect_daq_sensor`) och bearbetningsingångspunkterna (`ChlorosLocal`, `process_folder`)**startar den lokala backend-modulen automatiskt**. Ett genererat skript kräver varken att GUI är öppet eller att en server startas manuellt — det kräver endast att paketet desktop/CLI är installerat.
* **Hela pipelinen utförs med ett enda anrop.** `chloros_sdk.process_folder("path", indices=["NDVI"])` kör import → kalibrering → reflektans → indexexport från början till slut. Mindre yta innebär färre ställen där ett genererat skript kan gå fel.
* **Körningar utan utdata diagnostiserar sig själva.** Efter `process()` bifogas körningens sammanfattning till resultatet, och varje bearbetningstips (t.ex. *varför* en körning inte gav något resultat) skickas också ut på nytt som en Python `UserWarning` — så även ett skript som aldrig granskar resultatdictet visar diagnosen.
* **CLI misslyckas med ett tydligt felmeddelande.**En `chloros-cli process`-körning som begärde produkter men inte skrev ut några skriver ut `Processing finished but wrote no image products.` och**avslutas med ett värde som inte är noll**, så skript och CI upptäcker det med en enkel kontroll av avslutningskoden. Framgångsrika körningar rapporterar `Image products written: N`.

En asymmetri som en assistent bör känna till: SDK:s `process()` utlöser medvetet **inte** ett fel vid en körning utan resultat — det rapporteras istället via sammanfattningen/tipsen. Om en Python-pipeline måste avbrytas vid en tom körning, kontrollera sammanfattningen (recept 2 gör det).

## Förbehåll

* **Inloggning krävs för Chloros+.**CLI och SDK kräver en**betald** Chloros+-nivå, vilket kontrolleras på serversidan: förfrågningar misslyckas med `401 AUTH_REQUIRED` om du inte är inloggad och med `403 PLAN_UPGRADE_REQUIRED` på gratisnivån. Kör `chloros-cli login` en gång per maskin innan du kör genererade skript. Se [Chloros+ Inloggning](chloros+-login.md).
* **Capture-kommandon styr verklig hårdvara.** Kommandona `lattice` / `daq` / `project` och sessionsobjekten SDK ansluter, strömmar och aktiverar fysiska kameror och sensorer. Granska ett genererat skript innan det körs för första gången och kör det medan du övervakar hårdvaran.
* **Gör stickprovskontroller av utdata.** Verifiera produktmapparna och några pixelvärden innan du publicerar resultaten. I synnerhet skalas reflektans-TIFF-filer per källa — läs `Chloros:PixelScale` XMP-taggen (LATTICE: 32768 = 1,0 reflektans; Survey3: 65535) istället för att anta en divisor. Båda referenssidorna dokumenterar detta under ”Läsa reflektanspixlar”.
* **Små fallgropar som kan ställa till problem för genererad kod:**`pool-record` skriver till**backend-värdens** filsystem (standard `~/Documents/DAQ Live View/`); på maskiner med flera nätverksgränssnitt bör du föredra `daq pool-connect --eth-host <ip-or-hostname>` framför automatisk upptäckt; och använd `http://127.0.0.1:5000` (aldrig `localhost`) överallt där en backend-instans av URL förekommer.
