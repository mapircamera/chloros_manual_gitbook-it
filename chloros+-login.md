# Chloros+ Login

## Chloros e Chloros (Browser) Login

Il menu <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> consente di accedere al proprio account Chloros+ e sbloccare funzionalità aggiuntive.

Una volta effettuato l&#x27;accesso, verranno visualizzati i dettagli dell&#x27;account:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Login

Accedi con le tue credenziali Chloros+ per abilitare l&#x27;elaborazione CLI.

**Sintassi:**

```bash
chloros-cli login <email> <password>
```

{% hint style=&quot;info&quot; %}
**Utenti SDK**: Python SDK fornisce anche un metodo programmatico `logout()` per cancellare le credenziali memorizzate nella cache. Per ulteriori dettagli, consultare la [documentazione Python SDK](api-python-sdk.md#logout).
{% endhint %}

**Esempio:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Caratteri speciali**: utilizzare le virgolette singole per le password che contengono caratteri come `$`, `!` o spazi.
{% endhint %}

**Output:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Scadenza del piano

La scadenza del piano nella GUI indica quando la licenza non sarà più valida. Per gli abbonamenti mensili ricorrenti, la scadenza è alla fine del mese. Per gli abbonamenti annuali, è un anno dopo l&#x27;inizio dell&#x27;abbonamento. Il controllo della licenza richiede una connessione Internet mensile per la verifica, con un periodo di tolleranza di 30 giorni.

### Limite dei dispositivi

Ogni piano Chloros+ offre un numero diverso di dispositivi registrati. Ogni dispositivo a cui si accede con un account Chloros+ verrà conteggiato nel numero di dispositivi registrati. È possibile rinominare e rimuovere un dispositivo dalla pagina dell&#x27;account MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Piano Chloros</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">ARGENTO</th><th align="center">ORO</th></tr></thead><tbody><tr><td align="right">Dispositivi supportati</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
