# Utilizzo di Chloros con gli assistenti IA

Il presente manuale è rivolto a due tipi di destinatari: gli esseri umani e gli assistenti IA con cui gli esseri umani interagiscono sempre più spesso. Ogni pagina riporta valori esatti, impostazioni predefinite e comandi copiabili e incollabili, in modo che un assistente (Claude, ChatGPT, Copilot, un agente di programmazione, …) possa scrivere un’automazione Chloros funzionante al primo tentativo.

Versione di Chloros: **

1.2.0**. Piattaforme CLI/SDK: Windows 10/11 x64 e Linux (x86_64 / Jetson aarch64).

## Cosa fornire al proprio assistente

| Risorsa | URL | A cosa serve |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Indice leggibile dal computer di ogni pagina di questo manuale. |
| **Riferimento CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | La superficie di comando completa di `chloros-cli`: ogni comando, flag, impostazione predefinita, codice di uscita e regola relativa alla cartella di output. Scritto per l’utilizzo da parte dei modelli di linguaggio di grandi dimensioni (LLM). |
| **Riferimento SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | `chloros_sdk` Python API completo: classi, firme, eccezioni ed esempi pratici. Scritto per l’uso da parte degli studenti di LLM. |
| **Qualsiasi pagina in formato Markdown grezzo** | aggiungere `.md` alla pagina URL | ad es. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` restituisce la pagina in formato Markdown grezzo — ideale per incollarla in una finestra di contesto o recuperarla da un agente. |

Collegamenti all’interno del manuale: [Riferimento CLI](reference/cli-reference.md) · [Riferimento a SDK](reference/sdk-reference.md).

{% hint style="info" %}
Le due pagine di riferimento sono autonome: un assistente che ne abbia letta una non ha bisogno del resto del manuale per scrivere uno script corretto.
{% endhint %}

## Ricette pronte all’uso

Copia, compila il campo `<placeholders>` e incolla il tutto nel tuo assistente.

### 1. Elaborare una cartella di voli in NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Monitorare in batch una directory di acquisizioni

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Collegare un array LATTICE ed effettuare l’acquisizione

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Registrare gli spettri del sensore di luce DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
La creazione di script DAQ dalla riga di comando passa sempre attraverso la famiglia `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Altri sottocomandi `daq` che il vostro assistente potrebbe inventare non sono disponibili nelle build distribuite e generano un errore.
{% endhint %}

## Perché gli script scritti dall’IA funzionano bene con Chloros

Ciascuno di questi è un comportamento reale e verificato di Chloros 1.2.0: eliminano le classiche modalità di errore dell’automazione scritta da macchine:

* **Nessuna procedura di configurazione complicata.**Gli helper di connessione intelligente di SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) e i punti di ingresso di elaborazione (`ChlorosLocal`, `process_folder`)**avviano automaticamente il backend locale**. Uno script generato non richiede che la GUI sia aperta né che il server sia avviato manualmente: è sufficiente che sia installato il pacchetto desktop/CLI.
* **L’intera pipeline è costituita da un’unica chiamata.** `chloros_sdk.process_folder("path", indices=["NDVI"])` esegue in sequenza: importazione → calibrazione → riflettanza → esportazione dell’indice. Meno parti coinvolte, meno possibilità che uno script generato vada in errore.
* **Le esecuzioni senza output si autodiagnosticano.** Dopo `process()`, il riepilogo dell’esecuzione viene allegato al risultato e ogni indicazione relativa all’elaborazione (ad es. *perché* un&#x27;esecuzione non ha prodotto alcun output) viene riproposto come Python `UserWarning` — in modo che anche uno script che non ispeziona mai il dizionario dei risultati possa visualizzare la diagnosi.
* **Il codice CLI fallisce in modo evidente.**Un&#x27;esecuzione `chloros-cli process` che ha richiesto prodotti ma non ne ha scritto nessuno stampa `Processing finished but wrote no image products.` e**termina con un codice di uscita diverso da zero**, quindi gli script di shell e la CI lo rilevano con un semplice controllo del codice di uscita. Le esecuzioni riuscite riportano `Image products written: N`.

Un&#x27;asimmetria che un assistente dovrebbe conoscere: l&#x27;`process()` di SDK **non** genera deliberatamente un&#x27;eccezione in caso di esecuzione senza prodotti — lo segnala invece tramite il riepilogo/i suggerimenti. Se una pipeline Python deve arrestarsi in caso di esecuzione vuota, controllare il riepilogo (come fa la ricetta 2).

## Avvertenze

* **È richiesto l’accesso con Chloros+.**CLI e SDK richiedono un livello**a pagamento**Chloros+, applicato a livello di server: le richieste falliscono con codice `401 AUTH_REQUIRED` se non si è effettuato l’accesso e con codice `403 PLAN_UPGRADE_REQUIRED` se si utilizza il piano gratuito. Eseguire**`chloros-cli login`**una volta per macchina prima di eseguire gli script generati. Vedere [**Chloros+ Accesso**](chloros+-login.md).
* **I comandi di acquisizione controllano l&#x27;hardware reale.** I comandi `lattice` / `daq` / `project` e gli oggetti di sessione SDK si connettono, trasmettono in streaming e attivano telecamere e sensori fisici. Esaminare uno script generato prima della sua prima esecuzione ed eseguirlo in presenza dell’hardware.
* **Controllare a campione i risultati.** Verificare le cartelle dei prodotti e alcuni valori dei pixel prima di pubblicare i risultati. In particolare, i file TIFF di riflettanza vengono scalati in base alla sorgente — leggere il tag XMP di `Chloros:PixelScale` (LATTICE: 32768 = riflettanza 1,0; Survey3: 65535) invece di ipotizzare un divisore. Entrambe le pagine di riferimento documentano questo aspetto nella sezione «Lettura dei pixel di riflettanza».
* **Piccole insidie che causano errori nel codice generato:**`pool-record` scrive sul filesystem dell’**host di backend** (impostazione predefinita `~/Documents/DAQ Live View/`); su macchine con più interfacce di rete, è preferibile utilizzare `daq pool-connect --eth-host <ip-or-hostname>` anziché il rilevamento automatico; e utilizzare `http://127.0.0.1:5000` (mai `localhost`) ovunque compaia un backend URL.
