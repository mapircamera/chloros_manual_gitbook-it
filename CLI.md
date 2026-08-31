# CLI : Riga di comando

> **Riferimento completo:**[CLI Riferimento](reference/cli-reference.md) documenta**ogni flag di ogni sottocomando** ed è ottimizzato per gli assistenti AI — incolla il suo URL nel tuo assistente e chiedi un comando funzionante: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Suggerimento per gli strumenti di IA:** qualsiasi pagina di questo manuale è disponibile in formato Markdown grezzo aggiungendo `.md` al suo URL (ad es. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), mentre `https://mapir.gitbook.io/chloros/llms.txt` indicizza l’intero manuale per l’utilizzo da parte dei modelli LLM.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Che cos’è l’CLI


`chloros-cli` è il front-end da riga di comando dello stesso motore di elaborazione utilizzato dall’app desktopChloros
. Si tratta di un client &quot;thin&quot;HTTP
che si appoggia al backendChloros
(un server locale su `127.0.0.1:5000`) — la maggior parte dei comandi avvia automaticamente il backend, quindi una singola chiamata a `chloros-cli process …` è tutto ciò di cui uno script ha bisogno.

Funziona su **Windows
10/11 (x64)**e**Linux
(x86_64 e NVIDIA Jetson arm64 su JetPack 6)**, in qualsiasi terminale, senza bisogno di un&#x27;interfaccia grafica. Verifica l&#x27;installazione con:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Le famiglie di comandi, in sintesi:

* **Elaborazione e account** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 lingue — vedi [Lingue supportate](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (soloLinux
/Jetson)
* **Hardware live** — `lattice` (controllo telecamera LATTICE, oltre 45 sottocomandi), `daq pool-*` (sensori di luce DAQ), `time-sync` (PTP)
* **Automazione** — `project` (esecuzione in modalità headless di un progetto salvato suChloros
, incluse le ricette di acquisizione YAML)

Opzioni globali da conoscere: `--port N` (porta del backend, predefinita `5000`), `-v/--verbose`, `--restart` (riavvia forzatamente il backend), `--backend-exe PATH`. Consulta la [CLI
Guida di riferimento](reference/cli-reference.md) per l&#x27;elenco completo.

***

## Installazione

L’CLI
**è incluso nel programma di installazione diChloros** su tutte le piattaforme — non è disponibile un download separato suCLI
. Scarica il programma di installazione dalla pagina [Download](download.md).

###Windows


Il programma di installazione inserisce l’CLI
in:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

e aggiunge quella cartella al sistema `PATH` — **aprire un nuovo terminale**dopo l’installazione in modo che venga rilevato il file aggiornato `PATH`. Il programma di installazione inserisce inoltre gli script di avvio (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) nella directory principale di installazione, oltre a un**collegamento nel menu StartChlorosCLI
** nel menu Start, ciascuno dei quali apre un terminale con `chloros-cli` pronto all’uso.

###Linux


Installa la versione `.deb` adatta alla tua architettura:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Questo installa `chloros-cli` su `/usr/bin/chloros-cli` (già su `PATH`) e il backend a `/usr/lib/chloros/chloros-backend`, insieme al runtime ArenaSDK
necessario per le telecamere LATTICE. Per ulteriori dettagli, consultare [Installazione diLinux
](linux/linux-installation.md).

### Verifica

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Accesso e licenze

CLI
(ePython
SDK
) l’accesso richiede un **piano a pagamentoChloros
+**— qualsiasi piano a pagamento lo include; il piano gratuito no. La limitazione viene applicata**lato server** dal backend, non dal binarioCLI
: una chiamata senza aver effettuato l’accesso viene rifiutata con codice `401 AUTH_REQUIRED`, mentre una richiesta effettuata con account attivo sul piano gratuito viene respinta con il codice `403 PLAN_UPGRADE_REQUIRED`, indipendentemente dal fatto che provenga da `chloros-cli`, dall’SDK
o da un clientHTTP
sviluppato autonomamente. Eseguire l’aggiornamento su [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Effettuare l’accesso **una volta per ogni macchina**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Password con caratteri speciali**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` viene alterata dalla shell; l’CLI
e lo rileva con un errore 401 e riprova automaticamente, ma le virgolette singole risolvono completamente il problema).
{% endhint %}

La sessione viene memorizzata nella cache in `~/.chloros/user_session.json` e continua a funzionare offline per il periodo di tolleranza del piano (30 giorni per i piani mensili, fino alla scadenza per quelli annuali). `chloros-cli status` funziona anche senza un piano a pagamento, quindi il motivo del rifiuto è sempre visibile.

{% hint style="danger" %}
**Stai pianificando un&#x27;operazione headless? Effettua prima l&#x27;accesso.**Un comando di avvio del backend (`process`, `status`, `export-status`, …) eseguito**senza una sessione memorizzata nella cache**non fallisce immediatamente, ma passa a un prompt interattivo `Email:` / `Password:` su stdin. Un’operazione cron o una fase di CI in modalità automatica**rimarrà quindi in attesa di input**. Eseguire `chloros-cli login EMAIL 'PASSWORD'` una volta sulla macchina prima di pianificare qualsiasi operazione.
{% endhint %}

***

## La tua prima esecuzione di elaborazione

Indica a `process` una cartella contenente le acquisizioni: rileva automaticamenteSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` o una combinazione di questi:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

I flussi di avanzamento vengono trasmessi in tempo reale per ogni thread della pipeline (Rilevamento, Analisi, Elaborazione, Esportazione) e un&#x27;esecuzione riuscita termina segnalando il numero di prodotti immagine scritti (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Dove vengono salvati i risultati

`process` salva i dati in una **cartella di progetto**, non nella cartella di input:

* In assenza di `-o`: il progetto viene creato nella cartella predefinita dei progetti (condivisa con l’interfaccia grafica; gestiscila con `get-project-folder` / `set-project-folder`, con `~/Chloros Projects` come opzione di riserva), denominata `-n/--project-name` o con un timestamp (`YYYYMMDD_HHMMSS`) se omessa.
* Con `-o PATH`: quella cartella **è** la cartella del progetto. Se contiene già un `project.json`, viene creata una cartella gemella con suffisso `_1`/`_2`… invece di sovrascriverla.

All’interno del progetto, i prodotti sono raggruppati **per fotocamera, poi per formato di file**:

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

La cartella della fotocamera è `LATT-<sensor>-<lens>-F<filter>` per LATTICE (corrispondente all’EXIF della ripresa `Model`) e `<model>_<filter>` (ad es. `Survey3N_RGN`) perSurvey3
. La cartella del formato segue `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` o `tiff32` per `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Ogni prodotto esportato mantiene il nome del file ORIGINALE.**Un’esportazione Radiance di `capture_..._raw.tif` si chiama comunque `capture_..._raw.tif` — si trova semplicemente nella cartella `tiff32/Radiance_Images/`.**È la cartella a identificare il prodotto, non il nome del file**, quindi usa un’espressione regolare per la directory, non per il suffisso `*radiance*`.
{% endhint %}

### Le opzioni che userete effettivamente

| Flag | Impostazione predefinita | Funzione |
| --- | --- | --- |
| `-o, --output PATH` | cartella di progetto predefinita | Percorso della cartella di progetto (vedi sopra). |
| `-n, --project-name NAME` | timestamp | Nome del progetto. |
| `--format FMT` | `TIFF (16-bit)` | Uno tra `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | nessuno | Indici di vegetazione da esportare (vedi [Indici di vegetazione](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = debayer neurale, più lento, massima qualità (Chloros
+, GPU NVIDIA). |
| `--vignette / --no-vignette` | attivo | Correzione della vignettatura. |
| `--reflectance / --no-reflectance` | attivo | Calibrazione della riflettanza; per LATTICE questo è anche l’interruttore per il prodotto di riflettanza. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forza il punto di ingresso della pipeline per i file TIFF di LATTICE. |

Per tutto il resto — regolazione del rilevamento del bersaglio, PPK, pin di esposizione, flag di allineamento dell’array — consultare la [sezione `process` della Guida di riferimento diCLI
](reference/cli-reference.md).

***

## Scelta di cosa esportare (prodotti LATTICE)

L&#x27;elaborazione LATTICE si ramifica in **ogni prodotto applicabile in un unico passaggio**. I quattro interruttori per prodotto sono tutti**attivi per impostazione predefinita**; utilizzare il modulo `--no-` per disattivarne uno:

| Opzione | Prodotto |
| --- | --- |
| `--debayered` | Demosaicatura lineare → `Debayered_Images/` |
| `--preview` | Anteprima su schermo (bilanciamento del bianco + gamma; espansione in falsi colori per immagini multispettrali) → `Preview_Images/` |
| `--radiance` | radianza float32, W/m²/sr/nm → `Radiance_Images/` (sempre `tiff32/`) |
| `--reflectance` | uint16 riflettanza, compatibile con Pix4D → `Reflectance_Calibrated_Images/` |

RGB
le telecamere master emettono sempre e solo dati debayered + anteprima — la radianza/riflettanza per banda non è significativa per un sensore a banda larga, quindi tali opzioni non hanno alcun effetto su di esse.Survey3
`.raw` ignora le opzioni e segue il percorso standard di riflettanza/target.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (impostazione predefinita `auto`) seleziona il riferimento di riflettanza: `auto` crea un [bersaglio di calibrazione](calibration-targets.md) all’interno dell’inquadratura che supera i controlli di qualità come riferimento assoluto e, in assenza di bersaglio, ricorre al rapporto di divisione della luce in discesa del sensore DAQ (ρ = π·L/E); `target` è rigoroso (nessuna sostituzione DAQ); `daq` si basa sui dati del DAQ. Le scansioni del bersaglio misurate per unità possono essere fornite con `--target-reflectance-dir`.

{% hint style="info" %}
**Lettura dei pixel di riflettanza:**il DN che indica ρ = 1,0 è**per sorgente** — I file LATTICE inseriscono il tag `Chloros:PixelScale=32768` nell’XMP; i fileSurvey3
utilizzano 65535 (e non contengono tag `Chloros:*`). Leggere il tag e dividere per quel valore, anziché ipotizzare una costante. I dettagli e l’unico caso limite deliberato senza scala sono riportati nel [CLI
Riferimento](reference/cli-reference.md).
{% endhint %}

**L’elaborazione parte sempre da `raw`.** I prodotti derivati (esportazioni debayered/radianza/riflettanza) non vengono mai reimmessi nella pipeline: reimportarli ed elaborarli comporterebbe un’applicazione doppia dei calcoli di calibrazione, pertantoChloros
li salta e lo segnala. `--input-level` è la via di fuga prevista per i casi in cui sia davvero necessario forzare un punto di ingresso.

***

## Quando un&#x27;esecuzione fallisce

A partire dalla versione 1.2.0, `process` segnala chiaramente l’errore invece di “avere esito positivo” senza mostrare alcun risultato:

* Un&#x27;esecuzione che **ha richiesto prodotti ma non ne ha scritto nessuno**— solo `project.json` e `calibration_data.json` — stampa `Processing finished but wrote no image products.` e**termina con un valore diverso da zero**, quindi gli script possono rilevarlo. Le cause più comuni: la cartella di input non è stata riconosciuta come acquisizione (controllare il layout e `--input-level`), oppure tutti i prodotti richiesti non erano applicabili a quelle telecamere (ad es. richiedendo radianza/riflettanza da telecamere soloRGB
).
* Un&#x27;**esecuzione deliberata solo con metadati** (tutti i prodotti disattivati, nessun `--indices`) è comunque considerata riuscita: in questo caso, un output di immagine vuoto è il risultato corretto.
* Eseguire nuovamente l’operazione con `--verbose` e controllare il log del backend alla ricerca delle righe relative a `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, che spiegano i salti per singola telecamera.

Codici di uscita: `0` successo · `1` errore generico · `2` errore di argomento · `130` interrotto da Ctrl+C.

***

## Indici di vegetazione

Eseguire `--indices` con uno o più nomi di preset; ogni indice viene salvato nella propria cartella `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

I 22 nomi predefiniti accettati da `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Esistono tre elenchi di indici: non confonderli.**Il menu a tendina «Impostazioni progetto» dell’interfaccia grafica contiene 27 formule (aggiunge `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — queste cinque sono disponibili solo nell’interfaccia grafica e**non** sono valide per `--indices`). Il comando live/offline `lattice index --preset` utilizza un proprio elenco separato di 22 preset. Le formule e i calcoli delle bande sono documentati in [Formule degli indici multispettrali](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Sensori di luce DAQ: una breve panoramica

La famiglia `daq pool-*` gestisce i sensori spettrali DAQMAPIR
(DAQ-U tramite USB, DAQ-M tramite BLE, DAQ-E tramite Ethernet) attraverso il pool persistente del backend: l’interfaccia grafica (GUI),CLI
eSDK
condividono tutti un unico handle attivo. **`pool-*` è il percorso DAQ supportato nell’CLI
fornito**; gli altri sottocomandi `daq` a cui potresti vedere fare riferimento sono una superficie interna diMAPIR
, solo di origine, e terminano con un errore esplicito che rimanda a `pool-*`.

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

`pool-record` senza `--duration` viene eseguito fino a `pool-record --stop`; la directory di output predefinita è `~/Documents/DAQ Live View/` **sulla macchina del backend**. Il profilo di correzione del condensatore viene scelto al momento della connessione (`--cap-id`, impostazione predefinita del backend `sunshine_cosine`) e può essere sostituito in tempo reale con `pool-set-cap` — i profili di limite e l’intervallo calibrato del sensore sono trattati nei capitoli dedicati al DAQ di questo manuale.

{% hint style="warning" %}
**DAQ-E su un host con più schede di rete:** il primo rilevamento automatico di `pool-connect --eth` dopo l’avvio potrebbe fallire anche con un sensore funzionante. `--eth-host <ip-or-hostname>` è la forma affidabile — utilizzarla ogni volta che il rilevamento non produce risultati.
{% endhint %}

***

## Telecamere LATTICE, PTP e automazione di progetto

La famiglia `lattice` (oltre 45 sottocomandi) copre il funzionamento delle telecamere LATTICE dall’inizio alla fine: rilevamento, acquisizioni singole, array sincronizzati persistenti con il flusso di connessione “smart-prep” della GUI, anteprima live nel browser, allineamento, calcoli di indice e diagnostica della scheda di rete dell’host. Un assaggio:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Insieme a questo: `chloros-cli time-sync` genera un report sul grandmaster PTP eseguito dall’hostChloros
(le telecamere LATTICE e i sensori DAQ-E sono slave di questo grandmaster per i timestamp tra dispositivi), mentre `chloros-cli project` apre un progetto salvato suChloros
e gestisce le sue telecamere, array e sensori in modalità headless — comprese le ricette di acquisizione YAML basate su script.

Queste tre famiglie (`lattice`, `project`, `daq pool-*`) sono anche le uniche che supportano `CHLOROS_BACKEND_URL` per il controllo di un backend **remoto**; i comandi principali sono sempre destinati alla macchina locale.

Le guide complete sono disponibili nei capitoli dedicati a LATTICE di questo manuale; ogni flag è riportato nella [CLI
Guida di riferimento](reference/cli-reference.md).

***

## Risoluzione dei problemi: le 5 principali

| Sintomo | Soluzione |
| --- | --- |
| `Login required`, oppure un processo pianificato si blocca al prompt `Email:` | Eseguire `chloros-cli login EMAIL 'PASSWORD'` una volta su questa macchina: i comandi senza una sessione memorizzata nella cache vengono eseguiti in modo interattivo anziché generare un errore immediato. |
| `backend unreachable` | Avviare l’applicazione desktopChloros
oppure eseguire direttamente il binario di backend (`chloros-backend`). Se si punta `lattice`/`project`/`daq pool-*` verso un backend remoto, controllare `CHLOROS_BACKEND_URL`. |
| Connessione array bloccata: `FRAMES WILL DROP` / `Reduce ROI to enable` | Anello di ricezione della scheda di rete dell’host ripristinato ai valori predefiniti — la causa principale per cui un sistema precedentemente funzionante si rifiuta di connettersi, in genere dopo un aggiornamento del driver della scheda di rete. Eseguire `chloros-cli lattice network --fix` da un terminale **con privilegi elevati** (oppure impostare `ReceiveBufferLen=256`, `PendingReceives=64`); consultare la sezione *Configurazione e ottimizzazione della scheda di rete dell’host* della documentazione di riferimento. |
| Il sottocomando `daq` restituisce il messaggio: «richiede il pacchetto DAQ completo…» | Previsto nelle build fornite: l’CLI
o compilato include solo la famiglia `daq pool-*`, che copre le operazioni di connessione, streaming, registrazione e selezione dei canali. Utilizzare `pool-*` (o `chloros_sdk.connect_daq_sensor()` daPython
). |
| Jetson visualizza un avviso relativo allo swap prima di elaborare cartelle di grandi dimensioni | Aggiungere lo swap basato su file — il fileCLI
riporta i comandi esatti `fallocate`/`swapon` da eseguire. |

***

## Ottenere assistenza

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Ogni flag, ogni sottocomando:** [CLI
Riferimento](reference/cli-reference.md)
* **EquivalentePython
:** [Python
SDK
](api-python-sdk.md) e il [SDK
Riferimento](reference/sdk-reference.md)
* **Supporto:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
