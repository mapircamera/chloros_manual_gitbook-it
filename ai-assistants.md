# Utilizzo di Chloros con gli assistenti AI

Questo manuale è rivolto a due tipi di pubblico: gli esseri umani e gli assistenti IA con cui gli esseri umani interagiscono sempre più spesso. Ogni pagina riporta valori esatti, impostazioni predefinite e comandi copiabili e incollabili, in modo che un assistente (Claude, ChatGPT, Copilot, un agente di programmazione, …) possa scrivere un'automazione Chloros funzionante al primo tentativo.

Versione di Chloros: **

1.3.0**. Piattaforme CLI/SDK: Windows 10/11 x64 e Linux (x86\_64 / Jetson aarch64).

## Cosa fornire al proprio assistente

| Risorsa | URL | A cosa serve |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Indice leggibile da macchina di ogni pagina di questo manuale. |
| **Riferimento CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | La superficie di comando completa di `chloros-cli`: ogni comando, flag, impostazione predefinita, codice di uscita e regola relativa alla cartella di output. Scritto per l’utilizzo da parte di LLM. |
| **Riferimento SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | L’API Python completa di `chloros_sdk` in Python: classi, firme, eccezioni ed esempi pratici. Scritta per l’utilizzo da parte di LLM. |
| **Qualsiasi pagina in formato Markdown grezzo** | aggiungere `.md` all’URL della pagina | ad es. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` restituisce la pagina in formato Markdown grezzo — ideale per incollarla in una finestra di contesto o per recuperarla da un agente. |

Collegamenti interni al manuale: [Riferimento CLI](reference/cli-reference.md) · [Riferimento SDK](reference/sdk-reference.md).

{% hint style="info" %}
Le due pagine di riferimento sono autonome: un assistente che ne abbia letta una non ha bisogno del resto del manuale per scrivere uno script corretto.
{% endhint %}

## Lascia che il tuo assistente esegua direttamente Chloros: `chloros-mcp`

Tutto quanto descritto sopra prevede che il tuo assistente *scriva* uno script o un comando. Con `chloros-mcp`, un assistente che supporta il [Model Context Protocol](https://modelcontextprotocol.io) (MCP), come Claude Desktop, Claude Code o Cursor, gestisce direttamente Chloros. Basta chiedere in linguaggio naturale e l’assistente richiamerà gli strumenti del server:

* **Elaborazione delle immagini:** crea un progetto, importa una cartella di immagini acquisite, scegli le impostazioni, esegui l’elaborazione e genera un report sui risultati esportati dall’esecuzione.
* **Gestire l’hardware:** collegare telecamere LATTICE, array e sensori di luce DAQ, visualizzare l’anteprima, impostare esposizione e colore, acquisire, registrare e aggiornare il firmware.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Gli strumenti del server coprono le funzionalità dell’[Python SDK](api-python-sdk.md). Si installa insieme all’SDK, quindi di solito non c’è nient’altro da installare: su Windows il programma di installazione di Chloros inserisce `chloros-mcp.exe` nella cartella `Scripts` di Python, mentre quello per Linux, `.deb`, inserisce `chloros-mcp` in `/usr/bin`.

**Codice Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**aggiungi questo a `claude_desktop_config.json` (si apre da**Impostazioni > Sviluppatore > Modifica configurazione**), quindi riavvia Claude Desktop.**Cursore** richiede la stessa voce in `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Se il client segnala che il server non è riuscito ad avviarsi, probabilmente `chloros-mcp` non è presente nel `PATH` che vede: inserisci invece il percorso completo (`where chloros-mcp` su Windows, `which chloros-mcp` su Linux).

È necessario che Chloros sia installato sullo stesso computer e che vi sia un accesso effettuato su quel computer, nell’app o tramite `chloros-cli login`, su un piano a pagamento Chloros+ (Copper o superiore). Il server non può effettuare l’accesso al posto tuo.

{% hint style="info" %}
Configurazione client per ogni app, opzioni, ogni strumento e comportamento di un'esecuzione: [MCP Server (Assistenti AI)](reference/sdk-reference.md#mcp-server-ai-assistants) nella Documentazione di riferimento dell'SDK.
{% endhint %}

## Ricette per i prompt

Copia, compila il campo `<placeholders>` e incolla nel tuo assistente.

### 1. Elaborare una cartella di volo in NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Monitoraggio in batch di una directory di acquisizioni

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Collegamento di un array LATTICE e acquisizione

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Registrare gli spettri dei sensori di luce DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
La creazione di script DAQ dalla riga di comando avviene sempre tramite la famiglia `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Altri sottocomandi `daq` che il vostro assistente potrebbe inventare non sono disponibili nelle versioni distribuite e generano un errore.
{% endhint %}

## Perché gli script scritti dall’IA funzionano bene con Chloros

Ciascuno di questi è un comportamento reale e verificato di Chloros 1.3.0: eliminano le classiche modalità di errore dell’automazione scritta da macchine:

* **Nessuna procedura di configurazione complessa.**Gli helper smart-connect dell’SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) e i punti di ingresso di elaborazione (`ChlorosLocal`, `process_folder`)**avviano automaticamente il backend locale**. Uno script generato non richiede l’apertura dell’interfaccia grafica né l’avvio manuale di un server: è sufficiente che sia installato il pacchetto desktop o CLI.
* **L’intera pipeline è costituita da un’unica chiamata.** `chloros_sdk.process_folder("path", indices=["NDVI"])` esegue in sequenza: importazione → calibrazione → riflettanza → esportazione dell’indice. Meno aree di superficie, meno punti in cui uno script generato può andare in errore.
* **Le esecuzioni senza output si autodiagnosticano.** Dopo `process()`, il riepilogo dell’esecuzione viene allegato al risultato e ogni indicazione relativa all’elaborazione (ad es. *perché* un'esecuzione non ha prodotto alcun output) viene riemesso come Python `UserWarning` — in modo che anche uno script che non ispeziona mai il dizionario dei risultati mostri la diagnosi.
* **L’CLI fallisce in modo evidente.**Un’esecuzione `chloros-cli process` che ha richiesto prodotti ma non ne ha scritti alcuni stampa `Processing finished but wrote no image products.` e**termina con un codice di uscita diverso da zero**, quindi gli script della shell e la CI lo rilevano con un semplice controllo del codice di uscita. Le esecuzioni riuscite riportano il codice `Image products written: N`, oltre a `(plus M clamp masks)` quando sono state scritte le maschere. Allo stesso modo, i comandi di acquisizione indicano ogni file che non sono riusciti a scrivere e terminano con il codice `1`.
* **I percorsi relativi hanno il significato previsto dallo script.** Con un backend locale, un percorso di output relativo (`-o out/` nella CLI, `output_dir="out/"` nell’SDK) viene risolto rispetto alla cartella da cui viene eseguito il comando o lo script.

Un’asimmetria che un assistente dovrebbe conoscere: `process()` dell’SDK **non** genera deliberatamente un’eccezione in caso di esecuzione senza risultati — segnala invece l’evento tramite il riepilogo/i suggerimenti. Se una pipeline Python deve interrompersi in caso di esecuzione vuota, controllare il riepilogo (come fa la ricetta 2).

## Avvertenze

* **È richiesto l’accesso a Chloros+.**La CLI e l’SDK richiedono un piano**a pagamento** di Chloros+, con verifica lato server: le richieste falliscono con l’errore `401 AUTH_REQUIRED` se non si è effettuato l’accesso e con l’errore `403 PLAN_UPGRADE_REQUIRED` se si utilizza il piano gratuito. Eseguire `chloros-cli login` una volta per ogni macchina prima di eseguire gli script generati. Vedere [Accesso a Chloros+](chloros+-login.md).
* **I comandi di acquisizione controllano l’hardware fisico.** `lattice` / `daq` / `project`, gli oggetti di sessione dell’SDK e gli strumenti di `chloros-mcp` si connettono, trasmettono in streaming e attivano telecamere e sensori fisici. Esaminare uno script generato prima della sua prima esecuzione ed eseguirlo in presenza dell’hardware.
* **Controllare a campione i risultati.** Verificare le cartelle dei prodotti e alcuni valori dei pixel prima di pubblicare i risultati. In particolare, i file TIFF di riflettanza vengono ridimensionati in base alla sorgente: consultare il tag XMP di `Chloros:PixelScale` (i file LATTICE lo contengono: 32768 = 1,0 di riflettanza; i file Survey3 no, e utilizzano 65535 = 1,0) invece di ipotizzare un divisore. Entrambe le pagine di riferimento documentano questo aspetto nella sezione "Lettura dei pixel di riflettanza".
* **Piccole insidie che causano errori nel codice generato:**`pool-record` scrive sul filesystem dell’**host di backend** (impostazione predefinita `~/Documents/DAQ Live View/`); su macchine con più interfacce di rete, è preferibile utilizzare `daq pool-connect --eth-host <ip-or-hostname>` anziché il rilevamento automatico; utilizzare `http://127.0.0.1:5000` (mai `localhost`) ovunque compaia un URL del backend; uno script che percorre l’albero di output di un progetto dovrebbe saltare le cartelle `clamp_masks/` (i loro file `*_clampmask.tif` sono maschere, non immagini — vedi [Formati delle immagini di output](output-image-formats.md#clamp-masks)); e su Windows, con i percorsi lunghi disattivati, mantenere brevi le cartelle dei progetti — una cartella di progetto con un nome più lungo di circa 130 caratteri viene rifiutata prima dell’avvio dell’elaborazione.
