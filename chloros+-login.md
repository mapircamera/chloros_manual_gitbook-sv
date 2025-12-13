# Chloros+ Inloggning

## Chloros och Chloros (webbläsare) Inloggning

Användarmenyn <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> sidomenyn kan du logga in på ditt Chloros+-konto och låsa upp ytterligare funktioner.

När du är inloggad visas dina kontouppgifter:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Inloggning

Logga in med dina Chloros+-inloggningsuppgifter för att aktivera CLI-bearbetning.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

**Exempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Specialtecken**: Använd enkla citattecken runt lösenord som innehåller tecken som `$`, `!` eller mellanslag.
{% endhint %}

**Utdata:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Planens giltighetstid

Planens giltighetstid i GUI visar när din licens upphör att gälla. För återkommande månadsabonnemang upphör giltighetstiden vid månadens slut. För årsabonnemang är det ett år efter att du startade abonnemanget. Licenskontrollen kräver en månatlig internetanslutning för verifiering, med en 30 dagars respitperiod.

### Enhetsbegränsning

Varje Chloros+-plan erbjuder ett olika antal registrerade enheter. Varje enhet som du loggar in på med ett Chloros+-konto räknas in i antalet registrerade enheter. Du kan byta namn på och ta bort en enhet på din MAPIR Cloud-kontosida.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+-plan</th><th align="center">KOPPAR</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Enheter som stöds</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
