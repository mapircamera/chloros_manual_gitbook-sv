# Chloros+ Inloggning

## Inloggning via grafiskt gränssnitt

Via sidomenyn i användar<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">en kan du logga in på ditt Chloros+-konto och få tillgång till ytterligare funktioner.

**Du behöver bara logga in en gång per dator.** GUI:n, CLI och Python SDK delar samma cachade session — när du loggar in via skrivbordsgränssnittet aktiveras även CLI och SDK på den datorn (och vice versa via `chloros-cli login`).

När du är inloggad visas dina kontouppgifter:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Abonnemangsnivåer

| Abonnemang | `plan_id` | Typ |
| --- | --- | --- |
| Iron | `0` | Gratis |
| Copper | `1` | Betald (Chloros+) |
| Bronze | `2` | Betald (Chloros+) |
| Silver | `3` | Betald (Chloros+) |
| Guld | `4` | Betald (Chloros+) |

Se [abonnemang och priser](https://cloud.mapir.camera/pricing) för information om vad varje betald nivå innehåller.

### Åtkomst till CLI / SDK kräver en betald nivå

CLI och Python SDK kräver **valfri betald nivå Chloros+ (Copper eller högre)**. Detta tillämpas**på serversidan** — varje CLI/SDK-förfrågan måste innehålla både en aktiv session och ett betalt abonnemang:

| HTTP-status | `error_code` | Betydelse | Lösning |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Inte inloggad på den här datorn | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Inloggad, men abonnemangsnivån är för låg (gratis Iron-nivå) | Uppgradera till valfritt betalt Chloros+-abonnemang |

`chloros-cli status` förblir tillgängligt på gratisnivån, så du kan alltid se ditt aktuella abonnemang och varför åtkomst nekas.

### Begränsningar för ansluten hårdvara per abonnemang

Varje abonnemang har en övre gräns för hur många LATTICE-kameror och DAQ-ljussensorer som kan vara anslutna samtidigt:

| Plan | LATTICE-kameror | DAQ-ljussensorer |
| --- | --- | --- |
| Iron (gratis / inte inloggad) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI-inloggning

Logga in med dina Chloros+-inloggningsuppgifter för att aktivera CLI-bearbetning. På Linux (utan grafiskt gränssnitt) är detta det enda sättet att aktivera din licens.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK-användare**: Python SDK tillhandahåller även en programmeringsmetod för att rensa cachade inloggningsuppgifter. Se [SDK-referensen](reference/sdk-reference.md) för mer information.
{% endhint %}

**Exempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Specialtecken**: Använd enkla citattecken runt lösenord som innehåller tecken som `$`, `!` eller mellanslag.
{% endhint %}

**Utdata:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Lagring av inloggningsuppgifter

Cachelagrade inloggningsuppgifter och konfiguration lagras i mappen `.chloros` i din användarkatalog på **alla plattformar**:

| Plattform | Sökväg till inloggningsuppgiftscache |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### Planens giltighetstid och offline-övergångsperiod

Planens giltighetstid i användargränssnittet visar när din licens upphör att gälla. För återkommande månadsabonnemang upphör giltigheten vid månadens slut; för årsabonnemang är det ett år efter att du startade abonnemanget.

Chloros validerar din licens online, men det går att arbeta offline under en övergångsperiod:

* Lyckade servervalideringar cachas i **5 minuter**, så vid normal användning görs väldigt få licensförfrågningar.
* En signerad, maskinbunden licenscache täcker längre perioder offline: **30 dagar för månadsabonnemang**och**fram till abonnemangets utgångsdatum (högst 365 dagar) för årsabonnemang**.
* När övergångsperioden löper ut övergår abonnemanget till den kostnadsfria Iron-nivån tills enheten kan ansluta till licensservern en gång; åtkomsten återupptas vid nästa lyckade kontroll.

### Enhetsbegränsning

Varje Chloros+-abonnemang erbjuder ett olika antal registrerade enheter. Varje enhet som du loggar in på med ett Chloros+-konto räknas in i antalet registrerade enheter. Du kan byta namn på och ta bort en enhet på din MAPIR Cloud-kontosida.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+-abonnemang</th><th align="center">KOPPAR</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Enheter som stöds</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>Det exakta antalet enheter som ditt konto tillåter visas på din MAPIR Cloud-kontosida. När du loggar ut från en enhet frigörs dess plats, och en enhet som redan är registrerad kan alltid logga in igen, även om kontot har nått sin gräns för antalet enheter.
