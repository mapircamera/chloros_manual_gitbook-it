# Lingue supportate

Chloros offre un supporto completo dell&#x27;interfaccia in **38 lingue in tutto il mondo**, rendendolo accessibile agli utenti di ogni parte del globo. È possibile cambiare lingua istantaneamente sia nell&#x27;interfaccia grafica desktop che in CLI.

Chloros supporta le seguenti lingue:

| # | Lingua | Nome originale | Codice CLI |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Inglese | English | `en` |
| 2 | 🇪🇸 Spagnolo | Español | `es` |
| 3 | 🇵🇹 Portoghese | Português | `pt` |
| 4 | 🇫🇷 Francese | Français | `fr` |
| 5 | 🇩🇪 Tedesco | Deutsch | `de` |
| 6 | 🇮🇹 Italiano | Italiano | `it` |
| 7 | 🇯🇵 Giapponese | 日本語 | `ja` |
| 8 | 🇰🇷 Coreano | 한국어 | `ko` |
| 9 | 🇨🇳 Cinese (semplificato) | 简体中文 | `zh` |
| 10 | 🇹🇼 Cinese (tradizionale) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Russo | Русский | `ru` |
| 12 | 🇳🇱 Olandese | Nederlands | `nl` |
| 13 | 🇸🇦 Arabo | العربية | `ar` |
| 14 | 🇵🇱 Polacco | Polski | `pl` |
| 15 | 🇹🇷 Turco | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indonesiano | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnamita | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Tailandese | ไทย | `th` |
| 20 | 🇸🇪 Svedese | Svenska | `sv` |
| 21 | 🇩🇰 Danese | Dansk | `da` |
| 22 | 🇳🇴 Norvegese | Norsk | `no` |
| 23 | 🇫🇮 Finlandese | Suomi | `fi` |
| 24 | 🇬🇷 Greco | Ελληνικά | `el` |
| 25 | 🇨🇿 Ceco | Čeština | `cs` |
| 26 | 🇭🇺 Ungherese | Magyar | `hu` |
| 27 | 🇷🇴 Rumeno | Română | `ro` |
| 28 | 🇺🇦 Ucraino | Українська | `uk` |
| 29 | 🇧🇷 Portoghese brasiliano | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Cantonese | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malese | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovacco | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulgaro | Български | `bg` |
| 34 | 🇭🇷 Croato | Hrvatski | `hr` |
| 35 | 🇱🇹 Lituano | Lietuvių | `lt` |
| 36 | 🇱🇻 Lettone | Latviešu | `lv` |
| 37 | 🇪🇪 Estone | Eesti | `et` |
| 38 | 🇸🇮 Sloveno | Slovenščina | `sl` |

## Come cambiare la lingua

### In Chloros Desktop

1. Apri le impostazioni dell’applicazione
2. Accedi al menu di selezione della lingua
3. Scegli la lingua preferita dall’elenco
4. L’interfaccia si aggiornerà immediatamente

### In Chloros CLI

Utilizza il comando `language` per visualizzare o modificare la lingua dell&#x27;interfaccia CLI:

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

Per ulteriori dettagli, consulta la [documentazione di CLI](CLI.md).

## Copertura

Tutte le 38 lingue sono pienamente supportate in:

* **Chloros Desktop** - Traduzione completa dell&#x27;interfaccia grafica
* **Chloros CLI** - Interfaccia a riga di comando e messaggi di output

Python SDK API e la relativa [documentazione di riferimento](reference/sdk-reference.md) sono disponibili in inglese.

Il supporto linguistico garantisce che gli utenti di tutto il mondo possano lavorare in modo efficiente nella propria lingua madre senza ostacoli.
