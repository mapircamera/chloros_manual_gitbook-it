# Accesso a Chloros+

## Accesso tramite interfaccia grafica (GUI)

Il menu laterale dell'utente <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> consente di accedere al proprio account Chloros+ e sbloccare funzionalità aggiuntive.

**È sufficiente effettuare l'accesso una sola volta per ogni macchina.** L’interfaccia grafica (GUI), l’interfaccia a riga di comando (CLI) e l’SDK Python in SDK condividono la stessa sessione memorizzata nella cache: l’accesso tramite l’interfaccia grafica desktop attiva anche la CLI e l’SDK su quel computer (e viceversa tramite `chloros-cli login`).

Una volta effettuato l’accesso, verranno visualizzati i dettagli del tuo account:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Livelli di abbonamento

| Abbonamento | `plan_id` | Tipo |
| --- | --- | --- |
| Iron | `0` | Gratuito |
| Copper | `1` | A pagamento (Chloros+) |
| Bronze | `2` | A pagamento (Chloros+) |
| Silver | `3` | A pagamento (Chloros+) |
| Oro | `4` | A pagamento (Chloros+) |

Consulta [i piani e i prezzi](https://cloud.mapir.camera/pricing) per scoprire cosa include ciascun livello a pagamento.

### L'accesso a CLI/SDK richiede un livello a pagamento

L'accesso a CLI e Python SDK richiede **un piano a pagamento Chloros+ (Copper o superiore)**. Questa regola viene applicata**a livello di server**: ogni richiesta CLI/SDK deve includere sia una sessione attiva che un piano a pagamento:

| Stato HTTP | `error_code` | Significato | Soluzione |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Non si è effettuato l’accesso su questo computer | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Accesso effettuato, ma il livello del piano è troppo basso (livello gratuito Iron) | Passa a un piano a pagamento Chloros+ |

`chloros-cli status` rimane accessibile nel piano gratuito, quindi puoi sempre visualizzare il tuo piano attuale e il motivo per cui l’accesso è negato.

### Limiti relativi all’hardware collegato per piano

Ogni piano limita il numero di telecamere LATTICE e sensori di luce DAQ che possono essere collegati in tempo reale contemporaneamente:

| Piano | Telecamere LATTICE | Sensori di luce DAQ |
| --- | --- | --- |
| Iron (gratuito / senza aver effettuato l’accesso) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Accesso alla CLI



Accedi con le tue credenziali Chloros+ per abilitare l’elaborazione CLI. Su Linux (senza GUI), questo è l’unico modo per attivare la tua licenza.

**Sintassi:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Utenti SDK**: l’Python SDK fornisce anche un metodo programmatico `logout()` per cancellare le credenziali memorizzate nella cache. Per ulteriori dettagli, consulta la [Guida di riferimento SDK](reference/sdk-reference.md).
{% endhint %}

**Esempio:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caratteri speciali**: Utilizzare le virgolette singole per racchiudere le password che contengono caratteri come `$`, `!` o spazi.
{% endhint %}

**Output:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Archiviazione delle credenziali

Le credenziali e la configurazione memorizzate nella cache sono conservate nella cartella `.chloros` della directory home dell’utente su **tutte le piattaforme**:

| Piattaforma | Percorso della cache delle credenziali |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

L'installazione di una versione più recente su una esistente mantiene i dati di accesso, sia su Windows (il programma di installazione) che su Linux (la cartella `.deb`, inclusa `chloros-cli update`). La disinstallazione di Chloros — o la rimozione del pacchetto Linux — cancella i dati di accesso memorizzati nella cache, pertanto è necessario effettuare nuovamente l’accesso dopo una nuova installazione.

### Scadenza del piano e periodo di tolleranza offline

La scadenza del piano indicata nell’interfaccia grafica mostra quando la licenza non sarà più valida. Per gli abbonamenti mensili ricorrenti la scadenza è alla fine del mese; per gli abbonamenti annuali è un anno dopo l’inizio dell’abbonamento.

Chloros convalida la licenza online, ma il funzionamento offline è supportato entro un periodo di tolleranza:

* Le convalide riuscite sul server vengono memorizzate nella cache per **5 minuti**, quindi un utilizzo normale comporta pochissime richieste di licenza.
* Una cache delle licenze firmata e associata al dispositivo copre periodi offline più lunghi: **30 giorni per i piani mensili**e**fino alla data di scadenza dell’abbonamento (al massimo 365 giorni) per i piani annuali**.
* Allo scadere del periodo di tolleranza, il piano passa al livello gratuito “Iron” fino a quando il dispositivo non riesce a connettersi al server delle licenze; l’accesso riprende al successivo controllo andato a buon fine.

### Limite dei dispositivi

Ogni piano Chloros+ offre un numero diverso di dispositivi registrati. Ogni dispositivo su cui effettui l’accesso con un account Chloros+ viene conteggiato nel numero dei tuoi dispositivi registrati. Puoi rinominare e rimuovere un dispositivo dalla pagina del tuo account MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Piano Chloros</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Dispositivi supportati</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Il numero esatto di dispositivi consentiti dal tuo account è indicato nella pagina del tuo account MAPIR Cloud. Effettuando il logout da un dispositivo, lo slot corrispondente viene liberato in modo definitivo; inoltre, un dispositivo già registrato può sempre effettuare nuovamente l'accesso anche quando l'account ha raggiunto il limite massimo di dispositivi.
