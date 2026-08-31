# Språk som stöds

Chloros erbjuder fullständigt gränssnittsstöd på **38 språk världen över**, vilket gör det tillgängligt för användare över hela världen. Du kan växla språk direkt både i skrivbordsgränssnittet och i CLI.

Chloros stöder följande språk:

| # | Språk | Namn på originalspråket | CLI-kod |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Engelska | English | `en` |
| 2 | 🇪🇸 Spanska | Español | `es` |
| 3 | 🇵🇹 Portugisiska | Português | `pt` |
| 4 | 🇫🇷 Franska | Français | `fr` |
| 5 | 🇩🇪 Tyska | Deutsch | `de` |
| 6 | 🇮🇹 Italienska | Italiano | `it` |
| 7 | 🇯🇵 Japanska | 日本語 | `ja` |
| 8 | 🇰🇷 Koreanska | 한국어 | `ko` |
| 9 | 🇨🇳 Kinesiska (förenklad) | 简体中文 | `zh` |
| 10 | 🇹🇼 Kinesiska (traditionell) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Ryska | Русский | `ru` |
| 12 | 🇳🇱 Nederländska | Nederlands | `nl` |
| 13 | 🇸🇦 Arabiska | العربية | `ar` |
| 14 | 🇵🇱 Polska | Polski | `pl` |
| 15 | 🇹🇷 Turkiska | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indonesiska | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnamesiska | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Thailändska | ไทย | `th` |
| 20 | 🇸🇪 Svenska | Svenska | `sv` |
| 21 | 🇩🇰 Danska | Dansk | `da` |
| 22 | 🇳🇴 Norska | Norsk | `no` |
| 23 | 🇫🇮 Finska | Suomi | `fi` |
| 24 | 🇬🇷 Grekiska | Ελληνικά | `el` |
| 25 | 🇨🇿 Tjeckiska | Čeština | `cs` |
| 26 | 🇭🇺 Ungerska | Magyar | `hu` |
| 27 | 🇷🇴 Rumänska | Română | `ro` |
| 28 | 🇺🇦 Ukrainska | Українська | `uk` |
| 29 | 🇧🇷 Brasiliansk portugisiska | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Kantonesiska | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malajiska | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovakiska | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulgariska | Български | `bg` |
| 34 | 🇭🇷 Kroatiska | Hrvatski | `hr` |
| 35 | 🇱🇹 Litauiska | Lietuvių | `lt` |
| 36 | 🇱🇻 Lettiska | Latviešu | `lv` |
| 37 | 🇪🇪 Estniska | Eesti | `et` |
| 38 | 🇸🇮 Slovenska | Slovenščina | `sl` |

## Så här byter du språk

### I Chloros Desktop

1. Öppna programinställningarna
2. Gå till menyn för språkval
3. Välj önskat språk från listan
4. Gränssnittet uppdateras omedelbart

### I Chloros CLI

Använd kommandot `language` för att visa eller ändra gränssnittsspråket i CLI:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

Mer information finns i [CLI-dokumentationen](CLI.md).

## Täckning

Alla 38 språk stöds fullt ut i:

* **Chloros Desktop** – Fullständig översättning av grafiskt användargränssnitt
* **Chloros CLI** – Kommandoradsgränssnitt och utmatningsmeddelanden

Python SDK API och dess [referensdokumentation](reference/sdk-reference.md) tillhandahålls på engelska.

Språkstöd säkerställer att användare världen över kan arbeta effektivt på sitt modersmål utan hinder.
