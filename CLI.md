# CLI: Riga di comando

> **Riferimento completo:**[Riferimento CLI](reference/cli-reference.md) documenta**ogni flag di ogni sottocomando** ed è ottimizzato per gli assistenti AI — incolla il suo URL nel tuo assistente e chiedi un comando funzionante: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Suggerimento per gli strumenti di IA:** qualsiasi pagina di questo manuale è disponibile in formato Markdown grezzo aggiungendo `.md` al relativo URL (ad es. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), mentre `https://mapir.gitbook.io/chloros/llms.txt` indicizza l’intero manuale per l’utilizzo da parte dei modelli LLM.

> **Vuoi che il tuo assistente esegua Chloros direttamente?** `chloros-mcp`, un server MCP installato con l’Python SDK, funziona con Claude Desktop, Claude Code, Cursor e altri client MCP: vedi [Utilizzo di Chloros con gli assistenti IA](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Che cos’è la CLI


`chloros-cli` è il front-end a riga di comando dello stesso motore di elaborazione utilizzato dall’app desktop Chloros. Si tratta di un client HTTP leggero che si appoggia al backend di Chloros (un server locale su `127.0.0.1:5000`): la maggior parte dei comandi avvia automaticamente il backend, quindi a uno script basta una singola chiamata a `chloros-cli process …`. Il backend rimane quindi in esecuzione in attesa del comando successivo: si arresta su `chloros-cli logout` (a meno che l’app non sia aperta o sia in corso un’esecuzione o una compilazione), su `chloros-cli backend stop`, oppure automaticamente dopo 30 minuti in assenza di client, esecuzioni o dispositivi connessi. `chloros-cli backend status` lo mostra.

Funziona su **Windows 10/11 (x64)**e**Linux (x86_64 e NVIDIA Jetson arm64 su JetPack 6)**, in qualsiasi terminale, senza bisogno di una GUI. Verifica l’installazione con:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Le famiglie di comandi, in sintesi:

* **Elaborazione e account** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 lingue — vedi [Lingue supportate](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (solo Linux/Jetson)
* **Hardware attivo** — `lattice` (controllo telecamera LATTICE, oltre 45 sottocomandi), `daq pool-*` (sensori di luce DAQ), `time-sync` (PTP)
* **Automazione** — `project` (esecuzione in modalità headless di un progetto Chloros salvato, comprese le ricette di acquisizione YAML)

Opzioni globali da conoscere: `--port N` (porta del backend, predefinita `5000`), `-v/--verbose`, `--restart` (riavvia forzatamente il backend), `--backend-exe PATH`. Consulta il [Riferimento CLI](reference/cli-reference.md) per l’elenco completo.

***

## Installazione

La CLI **è inclusa nel programma di installazione di Chloros** su tutte le piattaforme — non è disponibile un download separato della CLI. Scarica il programma di installazione dalla pagina [Download](download.md).

### Windows


Il programma di installazione colloca la CLI in:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

e aggiunge quella cartella al sistema `PATH` una sola volta (un aggiornamento rimuove anche le voci duplicate lasciate dai programmi di installazione precedenti) — **aprire un nuovo terminale**dopo l’installazione in modo che venga rilevato il file `PATH` aggiornato. Il programma di installazione inserisce inoltre gli script di avvio (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) nella directory principale di installazione, oltre a un**Chloros CLI** nel menu Start, ciascuna delle quali apre un terminale con `chloros-cli` pronto all’uso.

### Linux


Installare `.deb` per la propria architettura con `apt` (non `dpkg -i`, che ignora i requisiti raccomandati dall’Python SDK):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Questo installa `chloros-cli` fino a `/usr/bin/chloros-cli` (già presenti in `PATH`), il backend e i programmi CLI decompressi in `/usr/lib/chloros/backend/` e `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` è un programma di avvio che avvia il backend), il runtime dell’SDK di Arena necessario al backend per le telecamere LATTICE e l’Python SDK per il sistema `python3`. Per ulteriori dettagli, consultare [Installazione su Linux](linux/linux-installation.md).

### Verifica

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Accesso e licenze

L’accesso alla CLI (e all’Python SDK) richiede un **piano Chloros+ a pagamento**— qualsiasi livello a pagamento lo include; il livello gratuito no. Il limite minimo viene applicato**lato server** dal backend, non dal binario della CLI: una chiamata senza aver effettuato l’accesso viene rifiutata con `401 AUTH_REQUIRED`, mentre una chiamata effettuata con account connesso sul piano gratuito viene respinta con il codice `403 PLAN_UPGRADE_REQUIRED`, indipendentemente dal fatto che provenga da `chloros-cli`, dall’SDK o da un client HTTP sviluppato autonomamente. Esegui l’aggiornamento su [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Effettua l’accesso **una volta per macchina**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Password con caratteri speciali**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` viene alterata dalla shell (l’CLI lo rileva con un errore 401 e riprova automaticamente, ma le virgolette singole evitano del tutto il problema).
{% endhint %}

La sessione viene memorizzata nella cache in `~/.chloros/user_session.json` e continua a funzionare offline per il periodo di tolleranza del piano (30 giorni per i piani mensili, fino alla scadenza per quelli annuali). `chloros-cli status` funziona anche senza un piano a pagamento, quindi il motivo del rifiuto è sempre visibile.

{% hint style="danger" %}
**Stai pianificando un'operazione headless? Effettua prima l'accesso.**Un comando di avvio del backend (`process`, `status`, `export-status`, …) eseguito**senza sessione memorizzata nella cache**non genera un errore immediato, ma passa a un prompt interattivo `Email:` / `Password:` su stdin. Un’operazione cron o una fase di CI in modalità automatica**rimarrà quindi bloccata in attesa di input**. Eseguire `chloros-cli login EMAIL 'PASSWORD'` una volta sulla macchina prima di pianificare qualsiasi operazione.
{% endhint %}

***

## La tua prima esecuzione di elaborazione

Indica a `process` una cartella contenente le acquisizioni: rileverà automaticamente Survey3 (`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` o una combinazione di questi:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

I flussi di avanzamento vengono trasmessi in tempo reale per ogni thread della pipeline (Rilevamento, Analisi, Elaborazione, Esportazione) e un'esecuzione riuscita termina segnalando il numero di prodotti immagine scritti: `Image products written: N`, seguito da `(plus M clamp masks)` quando l'esecuzione ha scritto anche [maschere di clamp](output-image-formats.md#clamp-masks) (che non vengono conteggiate come prodotti).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Se un backend Chloros è già in esecuzione (ad esempio quello dell’app desktop), `process` lo utilizza e vi apre il relativo progetto. Se l’app è aperta su un progetto da essa stessa avviato, il processo viene rifiutato: è necessario eseguire l’operazione dall’app oppure chiudere quel progetto. Se un'esecuzione precedente sta ancora terminando, attende fino a 15 secondi prima di rifiutare. `--restart` forza l'avvio di un nuovo backend. Su Linux, un backend eseguito come servizio systemd non viene mai riavviato: `process` lo utilizza così com’è e, nei casi in cui altrimenti lo riavvierebbe (`--restart` o un’importazione che non ha trovato nulla), visualizza il messaggio `sudo systemctl restart <unit>` e termina `1`.

### Dove vengono salvati i risultati

`process` scrive in una **cartella del progetto**, non nella cartella di input:

* In assenza di `-o`: il progetto viene creato nella cartella predefinita dei progetti (condivisa con l’interfaccia grafica; gestiscila con `get-project-folder` / `set-project-folder`; se non è impostata alcuna cartella, `Documents/MAPIR/Chloros Projects` nella cartella utente), denominato con `-n/--project-name` o un timestamp (`YYYYMMDD_HHMMSS`) se omesso.
* Con `-o PATH`: quella cartella **è** la cartella del progetto. Se contiene già un `project.json`, viene creato un file gemello con suffisso `_1`/`_2`… con suffisso, invece di sovrascriverlo. Un percorso relativo `PATH` viene considerato relativo alla cartella da cui si esegue `chloros-cli`.

{% hint style="info" %}
**Percorsi relativi (modificati nella versione 1.3.0).** Ogni percorso aperto dal backend — `-o`/`--output` con qualsiasi comando (incluso il comando predefinito `output` dei comandi di acquisizione), `--input`, `--burst-dir`, `--profile`, `--save` e `--mask` — viene risolto in base alla cartella da cui si esegue `chloros-cli`, quindi `-o captures/` scrive in `<current folder>/captures/`. Nella versione 1.2.0 il backend li risolveva nella propria cartella di installazione, il che causava un errore con il messaggio "Accesso negato". Per un backend remoto (`CHLOROS_BACKEND_URL`) i percorsi vengono inviati così come digitati e risolti su quella macchina.
{% endhint %}

{% hint style="warning" %}
**Windows: mantenere breve la cartella del progetto a meno che non siano abilitati i percorsi lunghi.** Con i percorsi lunghi disabilitati in Windows (impostazione predefinita di Windows), nessun percorso di file può superare i 260 caratteri e i prodotti si trovano fino a circa 130 caratteri al di sotto della cartella del progetto. `process` verifica questo prima di avviarsi: una cartella del progetto troppo profonda viene rifiutata con il messaggio **"La cartella del progetto è troppo profonda per questo computer"**, viene indicata la cartella del progetto più lunga che si adatta e come abilitare i percorsi lunghi — non viene elaborato nulla. Utilizzare una versione più breve di `-o` (o `set-project-folder`), oppure abilitare i percorsi lunghi di Windows (l’opzione **Abilita percorsi lunghi di Windows** del programma di installazione, selezionata per impostazione predefinita) e riavviare Chloros.
{% endhint %}

All’interno del progetto, i prodotti sono raggruppati **per fotocamera, poi per formato file**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

La cartella della fotocamera è `LATT-<sensor>-<lens>-F<filter>` per LATTICE (corrispondente all’EXIF dell’acquisizione `Model`) e `<model>_<filter>` (ad es. `Survey3N_RGN`) per Survey3. La cartella del formato segue `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` o `tiff32` per `TIFF (32-bit, Percent)`.

Per una fotocamera multispettrale M3C, in una cartella di formato può comparire anche una cartella `clamp_masks/`, accanto alle cartelle dei prodotti (ad esempio `tiff16/clamp_masks/Reflectance_Calibrated_Images/` e `tiff32/clamp_masks/Radiance_Images/`). Essa contiene le maschere per pixel dei prodotti di radianza e riflettanza in cui la separazione delle bande ha limitato un valore negativo a zero — si veda [Maschere di limitazione](output-image-formats.md#clamp-masks). Nessuna cartella di prodotto contiene mai una maschera.

{% hint style="info" %}
**Ogni prodotto esportato mantiene il nome del file ORIGINALE.**Un'esportazione di radianza di `capture_..._raw.tif` si chiama comunque `capture_..._raw.tif` — si trova semplicemente in `tiff32/Radiance_Images/`.**È la cartella a identificare il prodotto, non il nome del file**, quindi usa un’espressione regolare per la directory, non per il suffisso `*radiance*`.
{% endhint %}

### Le opzioni che userai effettivamente

| Flag | Impostazione predefinita | Cosa fa |
| --- | --- | --- |
| `-o, --output PATH` | cartella di progetto predefinita | Percorso della cartella di progetto (vedi sopra). |
| `-n, --project-name NAME` | timestamp | Nome del progetto. |
| `--format FMT` | `TIFF (16-bit)` | Uno tra `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros non esegue l'upscaling: sulle immagini interamente a 8 bit, un formato con profondità maggiore viene scritto come `TIFF (8-bit)`, e lo indica una sola volta. |
| `--indices NAME [NAME ...]` | nessuno | Indici di vegetazione da esportare (vedi [Indici di vegetazione](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = debayer neurale, più lento, massima qualità (Chloros+, richiede una GPU CUDA: in assenza di questa, l’esecuzione utilizza `standard`; non applicabile alle acquisizioni LATTICE, che utilizzano sempre `standard`). |
| `--vignette / --no-vignette` | attivo | Correzione della vignettatura. |
| `--reflectance / --no-reflectance` | attivo | Calibrazione della riflettanza; per LATTICE questo è anche l’interruttore per il prodotto di riflettanza. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forza il punto di ingresso della pipeline per i file TIFF di LATTICE. |

Per tutto il resto — regolazione del rilevamento dei target, PPK, pin di esposizione, flag di allineamento dell’array — consultare la [sezione `process` della Guida di riferimento CLI](reference/cli-reference.md).

***

## Scelta di cosa esportare (prodotti LATTICE)

L'elaborazione LATTICE si ramifica in **ogni prodotto applicabile in un unico passaggio**. I quattro interruttori per prodotto sono tutti**attivi per impostazione predefinita**; utilizzare il modulo `--no-` per disattivarne uno:

| Interruttore | Prodotto |
| --- | --- |
| `--debayered` | Demosaicatura lineare → `Debayered_Images/`: rimozione dell’offset scuro del pacchetto di calibrazione, seguita da un demosaic sensibile ai bordi, senza correzione di linearità, flat field o unmix — lo stesso file salvato da una cattura al livello `debayered`. Mantenuto nell’orientamento nativo del sensore (registrato in XMP). |
| `--preview` | Anteprima di visualizzazione (bilanciamento del bianco + gamma; espansione in falsi colori per immagini multispettrali) → `Preview_Images/` |
| `--radiance` | radianza float32, W/m²/sr/nm → `Radiance_Images/` (sempre `tiff32/`) |
| `--reflectance` | uint16 riflettanza, compatibile con Pix4D → `Reflectance_Calibrated_Images/` |

Le telecamere master RGB emettono sempre e solo dati debayered + anteprima — la radianza/riflettanza per banda non è significativa per un sensore a banda larga, quindi tali opzioni sono inattive per esse. Survey3 `.raw` ignora le opzioni e segue il percorso standard di riflettanza/target.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (impostazione predefinita `auto`) seleziona il riferimento di riflettanza: `auto` imposta come riferimento assoluto un [bersaglio di calibrazione](calibration-targets.md) presente nell’inquadratura che ha superato il controllo qualità (QA) e, in assenza di bersaglio, ricorre alla divisione della luce in discesa del sensore ottico del DAQ (ρ = π·L/E) quando non è presente alcun bersaglio (un bersaglio che supera il controllo di qualità con un flag entra in competizione con il DAQ sul punteggio misurato); `target` è rigoroso (nessuna sostituzione da parte del DAQ); `daq` è autorevole per il DAQ. Le scansioni del bersaglio misurate per unità possono essere fornite con `--target-reflectance-dir`.

{% hint style="info" %}
**Lettura dei pixel di riflettanza:**il DN che indica ρ = 1,0 è**per sorgente** — i file LATTICE inseriscono il tag `Chloros:PixelScale=32768` nell’XMP; i file Survey3 utilizzano 65535 e non contengono il tag `Chloros:PixelScale` (contengono invece altri tag di elaborazione `Chloros:*`). Leggere il tag e dividere per il valore indicato, anziché ipotizzare una costante. I dettagli e l’unico caso limite deliberato senza scala sono riportati nel [Riferimento CLI](reference/cli-reference.md).
{% endhint %}

**L’elaborazione parte sempre da `raw`.** I prodotti derivati (esportazioni debayered/radianza/riflettanza) non vengono mai reimmessi nella pipeline: reimportarli ed elaborarli comporterebbe un’applicazione doppia dei calcoli di calibrazione, quindi Chloros li salta e lo segnala. `--input-level` è la via di fuga prevista per i casi in cui sia davvero necessario forzare un punto di ingresso.

***

## Quando un'esecuzione fallisce

A partire dalla versione 1.2.0, `process` genera un errore evidente invece di «avere esito positivo» senza mostrare alcun risultato:

* Un'esecuzione che **ha richiesto prodotti ma non ne ha scritto nessuno**— solo `project.json` e `calibration_data.json` — stampa `Processing finished but wrote no image products.` e**termina con un valore diverso da zero**, quindi gli script possono rilevarlo. Le cause più comuni: la cartella di input non è stata riconosciuta come acquisizione (controllare il layout e `--input-level`), oppure tutti i prodotti richiesti non erano applicabili a quelle telecamere (ad esempio, richiedere radianza/riflettanza da telecamere solo RGB).
* Un'**esecuzione deliberata solo con metadati** (tutti i prodotti disattivati, nessun `--indices`) è comunque considerata riuscita: in questo caso, un'immagine di output vuota è il risultato corretto.
* Eseguire nuovamente l’operazione con `--verbose` e controllare il log del backend alla ricerca delle righe `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, che spiegano i salti per singola telecamera.
* Su Windows, con i percorsi lunghi disattivati, una cartella di progetto troppo profonda rispetto al limite di 260 caratteri per il percorso viene rifiutata **prima** dell’avvio dell’elaborazione, con la cartella di progetto più lunga che rientra nel limite (vedi [Dove vengono salvati i file di output](#where-the-outputs-land)).
* I comandi di acquisizione (`lattice capture`, `lattice array-capture` e le altre famiglie di acquisizione) indicano ogni file che non sono riusciti a scrivere e terminano con il codice `1`; le acquisizioni ripetute si interrompono quando l’unità è piena.

Codici di uscita: `0` successo · `1` errore generico · `2` errore di argomento · `130` interrotto da Ctrl+C.

***

## Indici di vegetazione

Eseguire `--indices` con uno o più nomi predefiniti; ogni indice viene salvato nella propria cartella `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

I 22 nomi predefiniti accettati da `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Viene aggiunto al progetto un nome che non era ancora presente nell’elenco. Un nome che non è un preset, o che le bande delle telecamere non sono in grado di generare, viene ignorato con un avviso che ne indica il nome. Se l’CLI non riesce a leggere l’elenco degli indici corrente del progetto, lascia l’elenco invariato e visualizza un avviso anziché sostituirlo.

{% hint style="warning" %}
**Esistono tre elenchi di indici: non confonderli.**Il menu a tendina «Impostazioni progetto» dell’interfaccia grafica contiene 27 formule (aggiunge `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — queste cinque sono disponibili solo nell’interfaccia grafica e**non** sono valide per `--indices`). Il comando live/offline `lattice index --preset` utilizza un proprio elenco separato di 22 preset. Le formule e i calcoli delle bande sono documentati in [Formule degli indici multispettrali](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Sensori di luce DAQ: una rapida panoramica

La famiglia `daq pool-*` gestisce i sensori spettrali DAQ MAPIR (DAQ-U tramite USB, DAQ-M tramite BLE, DAQ-E tramite Ethernet) attraverso il pool persistente del backend: l’interfaccia grafica (GUI), l’interfaccia a riga di comando (CLI) e l’SDK condividono tutti un unico handle attivo. **`pool-*` è il percorso DAQ supportato nell’CLI fornita**; altri sottocomandi `daq` a cui potresti trovare riferimento sono un’interfaccia interna a MAPIR solo in lettura e terminano con un errore esplicito che rimanda a `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` senza `--duration` viene eseguito fino a `pool-record --stop`; la directory di output predefinita è `~/Documents/DAQ Live View/` **sulla macchina del backend**. Il profilo di correzione del condensatore viene scelto al momento della connessione (`--cap-id`, predefinito del backend `sunshine_cosine`) e può essere sostituito in tempo reale con `pool-set-cap` — i profili cap e l’intervallo calibrato del sensore sono trattati nei capitoli dedicati al DAQ del presente manuale.

{% hint style="warning" %}
**DAQ-E su un host con più schede di rete:** il primo rilevamento automatico di `pool-connect --eth` dopo l’avvio potrebbe fallire anche con un sensore funzionante. `--eth-host <ip-or-hostname>` è la versione affidabile — utilizzarla ogni volta che il rilevamento non produce risultati.
{% endhint %}

***

## Telecamere LATTICE, PTP e automazione dei progetti

La famiglia `lattice` (oltre 45 sottocomandi) copre l’intero processo di gestione delle telecamere LATTICE: rilevamento, acquisizioni singole, array sincronizzati persistenti con il flusso di connessione “smart-prep” dell’interfaccia grafica, anteprima live nel browser, allineamento, calcoli di indice e diagnostica delle schede di rete dell’host. Un assaggio:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Insieme a questa: `chloros-cli time-sync` fornisce un resoconto sul grandmaster PTP eseguito dall’host Chloros (le telecamere LATTICE e i sensori DAQ-E sono slave di quest’ultimo per i timestamp tra dispositivi), mentre `chloros-cli project` apre un progetto Chloros salvato e controlla le sue telecamere, gli array e i sensori in modalità headless — incluse le ricette di acquisizione YAML basate su script.

Queste tre famiglie (`lattice`, `project`, `daq pool-*`) sono anche le uniche che supportano `CHLOROS_BACKEND_URL` per il controllo di un backend **remoto**; i comandi principali sono sempre destinati alla macchina locale.

Le guide complete sono disponibili nei capitoli dedicati a LATTICE di questo manuale; ogni flag è riportato nella [Guida di riferimento alla CLI](reference/cli-reference.md).

***

## Risoluzione dei problemi: i 5 principali

| Sintomo | Soluzione |
| --- | --- |
| `Login required` o un processo pianificato si blocca al prompt `Email:` | Eseguire una volta `chloros-cli login EMAIL 'PASSWORD'` su questa macchina: i comandi senza una sessione memorizzata nella cache vengono eseguiti in modalità interattiva anziché generare un errore immediato. |
| `backend unreachable` | Avvia l’app desktop Chloros, oppure avvia manualmente il backend: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` su Windows (la cartella `backend` nella cartella di installazione; non si trova in `PATH`), `chloros-backend` su Linux. `chloros-cli backend status` indica se ne è in esecuzione uno; `lattice`/`project`/`daq pool-*` non ne avviano mai uno. Se li si indirizza verso un backend remoto, controllare `CHLOROS_BACKEND_URL`. |
| Connessione array bloccata: `FRAMES WILL DROP` / `Reduce ROI to enable` | Anello di ricezione della scheda di rete dell’host ripristinato ai valori predefiniti — la causa principale per cui un sistema precedentemente funzionante si rifiuta di connettersi, in genere dopo un aggiornamento del driver della scheda di rete. Esegui `chloros-cli lattice network --fix` da un terminale **con privilegi elevati** (oppure imposta `ReceiveBufferLen=256`, `PendingReceives=64`); consulta la sezione *Configurazione e ottimizzazione della scheda di rete dell’host* della documentazione di riferimento. |
| Il sottocomando `daq` termina con il messaggio: "richiede il pacchetto DAQ completo…" | Utilizzare i comandi `daq pool-*`, che gestiscono lo stesso sensore tramite il backend: connessione, streaming, registrazione, selezione dei cap e, per un DAQ-E, le impostazioni di rete, il riavvio e gli aggiornamenti del firmware (oppure `chloros_sdk.connect_daq_sensor()` da Python). |
| Jetson visualizza un avviso relativo allo swap prima di elaborare cartelle di grandi dimensioni | Aggiungere uno swap basato su file: l’CLI visualizza i comandi esatti `fallocate`/`swapon` da eseguire. |

***

## Ottenere assistenza

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Ogni flag, ogni sottocomando:** [Riferimento CLI](reference/cli-reference.md)
* **Equivalente Python:** [Python SDK](api-python-sdk.md) e il [Riferimento SDK](reference/sdk-reference.md)
* **Assistenza:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
