# Accesso a Chloros+

## Accesso tramite interfaccia grafica

Il menu laterale dell&#x27;<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">e utente consente di accedere al proprio account Chloros+ e sbloccare funzionalità aggiuntive.

**È sufficiente effettuare l&#x27;accesso una sola volta per ogni macchina.** L&#x27;interfaccia grafica (GUI), CLI, Python e SDK condividono la stessa sessione memorizzata nella cache : effettuare l’accesso tramite l’interfaccia grafica desktop attiva anche CLI e SDK su quel dispositivo (e viceversa tramite `chloros-cli login`).

Una volta effettuato l’accesso, verranno visualizzati i dettagli del tuo account:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Livelli dei piani

| Piano | `plan_id` | Tipo |
| --- | --- | --- |
| Iron | `0` | Gratuito |
| Copper | `1` | A pagamento (Chloros+) |
| Bronze | `2` | A pagamento (Chloros+) |
| Argento | `3` | A pagamento (Chloros+) |
| Oro | `4` | A pagamento (Chloros+) |

Consulta [i piani e i prezzi](https://cloud.mapir.camera/pricing) per scoprire cosa include ciascun livello a pagamento.

### L’accesso a CLI / SDK richiede un piano a pagamento

L&#x27;accesso a CLI, Python e SDK richiede **un piano a pagamento Chloros+ (Copper o superiore)**. Questa regola viene applicata**a livello di server**: ogni richiesta CLI/SDK deve includere sia una sessione attiva che un piano a pagamento:

| Stato HTTP | `error_code` | Significato | Soluzione |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Non effettuato l’accesso su questo dispositivo | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Accesso effettuato, ma il livello del piano è troppo basso (livello gratuito Iron) | Passare a un piano Chloros+ a pagamento |

`chloros-cli status` rimane accessibile nel piano gratuito, quindi puoi sempre visualizzare il tuo piano attuale e il motivo per cui l’accesso è stato negato.

### Limiti relativi all’hardware collegato per piano

Ogni piano limita il numero di telecamere LATTICE e sensori di luce DAQ che possono essere collegati in tempo reale contemporaneamente:

| Piano | Telecamere LATTICE | Sensori di luce DAQ |
| --- | --- | --- |
| Iron (gratuito / senza aver effettuato l’accesso) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Accesso a CLI

Accedi con le tue credenziali Chloros+ per abilitare l’elaborazione CLI. Su Linux (senza interfaccia grafica), questo è l&#x27;unico modo per attivare la licenza.

**Sintassi:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Utenti di SDK**: Python SDK fornisce anche un metodo a livello di programmazione `logout()` per cancellare le credenziali memorizzate nella cache. Per ulteriori dettagli, consultare la [Documentazione di riferimento di SDK](reference/sdk-reference.md).
{% endhint %}

**Esempio:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caratteri speciali**: utilizzare le virgolette singole per racchiudere le password contenenti caratteri come `$`, `!` o spazi.
{% endhint %}

**Output:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Archiviazione delle credenziali

Le credenziali e la configurazione memorizzate nella cache sono conservate nella cartella `.chloros` della directory home dell&#x27;utente su **tutte le piattaforme**:

| Piattaforma | Percorso della cache delle credenziali |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### Scadenza del piano e periodo di tolleranza offline

La scadenza del piano indicata nell’interfaccia grafica mostra quando la licenza non sarà più valida. Per gli abbonamenti mensili ricorrenti la scadenza è alla fine del mese; per gli abbonamenti annuali è un anno dopo l’inizio dell’abbonamento.

Chloros convalida la licenza online, ma il funzionamento offline è supportato entro un periodo di tolleranza:

* Le convalide riuscite sul server vengono memorizzate nella cache per **5 minuti**, quindi un utilizzo normale comporta pochissime richieste di licenza.
* Una cache di licenze firmate e vincolate al dispositivo copre periodi offline più lunghi: **30 giorni per i piani mensili**e**fino alla data di scadenza dell’abbonamento (al massimo 365 giorni) per i piani annuali**.
* Allo scadere del periodo di tolleranza, il piano passa al livello gratuito Iron fino a quando il dispositivo non riesce a connettersi al server delle licenze; l’accesso riprende al successivo controllo andato a buon fine.

### Limite dei dispositivi

Ogni piano Chloros+ offre un numero diverso di dispositivi registrati. Ogni dispositivo su cui effettui l’accesso con un account Chloros+ viene conteggiato nel numero di dispositivi registrati. Puoi rinominare e rimuovere un dispositivo dalla pagina del tuo account MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Piano Chloros+</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">ORO</th></tr></thead><tbody><tr><td align="right">Dispositivi supportati</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>Il numero esatto di dispositivi consentiti per il tuo account è indicato nella pagina del tuo account MAPIR Cloud. Effettuando il logout da un dispositivo, lo slot corrispondente viene liberato in modo definitivo; inoltre, un dispositivo già registrato può sempre effettuare nuovamente l&#x27;accesso anche quando l&#x27;account ha raggiunto il limite massimo di dispositivi consentiti.
