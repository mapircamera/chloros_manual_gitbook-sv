# Att använda Chloros tillsammans med AI-assistenter

Denna handbok är skriven för två målgrupper: människor och de AI-assistenter som människor i allt högre grad arbetar genom. Varje sida innehåller exakta värden, standardinställningar och kommandon som kan kopieras och klistras in, så att en assistent (Claude, ChatGPT, Copilot, en kodningsagent, …) kan skriva fungerande Chloros-automatisering redan vid första försöket.

Chloros-version: **

1.3.0**. CLI-/SDK-plattformar: Windows 10/11 x64 och Linux (x86\_64 / Jetson aarch64).

## Vad du ska ge din assistent

| Resurs | URL | Vad den är till för |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Maskinläsbart index över varje sida i denna handbok. |
| **Referens för CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Den fullständiga kommandoytan för `chloros-cli`: alla kommandon, flaggor, standardvärden, avslutningskoder och regler för utdatamappar. Skriven för användning av LLM. |
| **SDK-referens** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Det fullständiga `chloros_sdk` Python-API: klasser, signaturer, undantag och praktiska exempel. Skrivet för användning av stora språkmodeller (LLM). |
| **Vilken sida som helst som rå Markdown** | lägg till `.md` till sidans URL | t.ex. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` returnerar sidan som rå Markdown — perfekt för att klistra in i ett kontextfönster eller hämta från en agent. |

Länkar i manualen: [CLI-referens](reference/cli-reference.md) · [SDK-referens](reference/sdk-reference.md).

{% hint style="info" %}
De två referenssidorna är fristående: en assistent som har läst en av dem behöver inte resten av manualen för att skriva ett korrekt skript.
{% endhint %}

## Låt din assistent köra Chloros direkt: `chloros-mcp`

I allt ovanstående får din assistent *skriva* ett skript eller ett kommando. Med `chloros-mcp` styr en assistent som talar [Model Context Protocol](https://modelcontextprotocol.io) (MCP), såsom Claude Desktop, Claude Code eller Cursor, Chloros-programmet själv. Du ställer frågor i vanligt språk och den anropar serverns verktyg:

* **Bearbeta bilder:** skapa ett projekt, importera en mapp med bilder, välj inställningar, bearbeta och rapportera vad körningen exporterade.
* **Kör hårdvara:** anslut LATTICE-kameror, arrayer och DAQ-ljussensorer, förhandsgranska, ställ in exponering och färg, ta bilder, spela in och uppdatera firmware.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Serverns verktyg täcker vad [Python SDK](api-python-sdk.md) gör. De installeras tillsammans med SDK:et, så det finns vanligtvis inget annat att installera: i Windows placerar Chloros-installationsprogrammet `chloros-mcp.exe` i din Pythons `Scripts`-mapp, och i Linux-versionen av `.deb` placeras `chloros-mcp` i `/usr/bin`.

**Claude-kod:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**lägg till detta i `claude_desktop_config.json` (**Inställningar &gt; Utvecklare &gt; Redigera konfiguration**öppnar den), och starta sedan om Claude Desktop.**Cursor** använder samma post i `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Om klienten meddelar att servern inte kunde startas, finns `chloros-mcp` troligen inte i den `PATH` som den ser: ange istället den fullständiga sökvägen (`where chloros-mcp` i Windows, `which chloros-mcp` i Linux).

Du måste ha Chloros installerat på samma dator och vara inloggad där, i appen eller via `chloros-cli login`, på valfri betald Chloros+-nivå (Copper eller högre). Servern kan inte logga in åt dig.

{% hint style="info" %}
Klientkonfiguration för varje app, alternativ, varje verktyg och hur en körning fungerar: [MCP-server (AI-assistenter)](reference/sdk-reference.md#mcp-server-ai-assistants) i SDK-referensen.
{% endhint %}

## Recept för kommandorader

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

### 2. Övervaka en katalog med bilder i batch

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Anslut en LATTICE-array och ta bilder

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
DAQ-skriptning från kommandoraden sker alltid via `daq pool-*`-familjen (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Andra `daq`-underkommandon som din assistent kan hitta på finns inte i de levererade versionerna och avslutas med ett felmeddelande.
{% endhint %}

## Varför AI-skrivna skript fungerar bra med Chloros

Var och en av dessa är ett verkligt, verifierat beteende hos Chloros 1.3.0 – de eliminerar de klassiska felkällorna hos maskinskrivna automatiseringar:

* **Inga krångliga inställningar.**SDK:s smart-connect-hjälpfunktioner (`connect_camera`, `connect_array`, `connect_daq_sensor`) och bearbetningsingångspunkterna (`ChlorosLocal`, `process_folder`)**startar den lokala backend-miljön automatiskt**. Ett genererat skript behöver varken ett öppet GUI eller en manuellt startad server — det krävs endast att desktop-/CLI-paketet är installerat.
* **Hela pipelinen utförs med ett enda anrop.** `chloros_sdk.process_folder("path", indices=["NDVI"])` kör import → kalibrering → reflektans → indexexport från början till slut. Mindre yta innebär färre ställen där ett genererat skript kan gå fel.
* **Körningar utan utdata diagnostiserar sig själva.** Efter `process()` bifogas körningens sammanfattning till resultatet, och varje bearbetningstips (t.ex. *varför* en körning inte gav något resultat) skickas också ut på nytt som en Python-kod (`UserWarning`) – så även ett skript som aldrig granskar resultatdiktet visar diagnosen.
* **CLI:n misslyckas tydligt.**En körning med `chloros-cli process` som begärde produkter men inte skrev ut några skriver ut `Processing finished but wrote no image products.` och**avslutas med ett värde som inte är noll**, så att skript och CI kan upptäcka det med en enkel kontroll av avslutningskoden. Framgångsrika körningar rapporterar `Image products written: N`, plus `(plus M clamp masks)` när masker skrevs. Capture-kommandon namnger på samma sätt varje fil som de inte kunde skriva och avslutas med `1`.
* **Relativa sökvägar betyder det som ett skript förväntar sig.** Med en lokal backend tolkas en relativ utdatasökväg (`-o out/` i CLI, `output_dir="out/"` i SDK) utifrån den mapp som kommandot eller skriptet körs från.

En asymmetri som en assistent bör känna till: SDK:s `process()` utlöser medvetet **inte** något undantag vid en körning utan resultat – istället rapporteras detta via sammanfattningen/tipsen. Om en Python-pipeline måste avbrytas vid en tom körning, kontrollera sammanfattningen (recept 2 gör det).

## Förbehåll

* **Inloggning på Chloros+ krävs.**CLI:n och SDK:n kräver en**betald** Chloros+-nivå, vilket kontrolleras på serversidan: förfrågningar misslyckas med felkoden `401 AUTH_REQUIRED` om man inte är inloggad och med `403 PLAN_UPGRADE_REQUIRED` på gratisnivån. Kör `chloros-cli login` en gång per maskin innan du kör de genererade skripten. Se [Chloros+ inloggning](chloros+-login.md).
* **Capture-kommandon styr fysisk hårdvara.** `lattice` / `daq` / `project`-kommandon, SDK-sessionsobjekten och `chloros-mcp`:s verktyg ansluter till, strömmar från och aktiverar fysiska kameror och sensorer. Granska ett genererat skript innan det körs för första gången och kör det medan du övervakar hårdvaran.
* **Gör stickprovskontroller av utdata.** Kontrollera produktmapparna och några pixelvärden innan du publicerar resultaten. I synnerhet skalas reflektans-TIFF-filer per källa – läs `Chloros:PixelScale` XMP-taggen (LATTICE-filer innehåller den: 32768 = 1,0 reflektans; Survey3-filer har den inte och använder 65535 = 1,0) istället för att anta en divisor. Båda referenssidorna dokumenterar detta under ”Läsa reflektanspixlar”.
* **Små fallgropar som kan ställa till problem för genererad kod:**`pool-record` skriver till**backend-värdens** filsystem (standard `~/Documents/DAQ Live View/`); på datorer med flera nätverksgränssnitt bör du föredra `daq pool-connect --eth-host <ip-or-hostname>` framför automatisk upptäckt; använd `http://127.0.0.1:5000` (aldrig `localhost`) överallt där en backend-URL förekommer; ett skript som går igenom ett projekts utdataträd bör hoppa över `clamp_masks/`-mappar (deras `*_clampmask.tif`-filer är masker, inte bilder — se [Utmatningsbildformat](output-image-formats.md#clamp-masks)); och på Windows där långa sökvägar är inaktiverade bör projektmapparna hållas korta — en projektmapp som är djupare än cirka 130 tecken avvisas innan bearbetningen påbörjas.
