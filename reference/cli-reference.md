# Chloros CLI Riferimento

**Versione:**

1.2.0**Generato:**29/07/2026 19:19 ·**Revisionato:** 30/08/2026**Destinatari:** Ottimizzato per l’utilizzo da parte di modelli di linguaggio di grandi dimensioni (LLM); leggibile dall’uomo.**Ambito:** Ogni sottocomando di `chloros-cli` destinato agli utenti, con opzioni ed esempi copiabili e incollabili.

Questo documento costituisce il riferimento completo per lo strumento da riga di comando `chloros-cli` fornito con MAPIR Chloros. È volutamente esaustivo in modo che un LLM (o una persona) possa comporre qualsiasi flusso di lavoro supportato dagli elenchi riportati di seguito senza dover esaminare il codice sorgente.

Se ti servono solo i punti salienti, vai a:
- [Guida rapida in cinque minuti](#five-minute-quickstart)
- [Flusso di lavoro per la prima connessione della telecamera LATTICE](#lattice-camera-first-connect-workflow)
- [Flusso di lavoro per la prima connessione del sensore DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Modalità di acquisizione, registratori e rielaborazione offline](#capture-modes-recorders--offline-reprocess)

---

## Convenzioni

- Tutti i comandi sono preceduti dal prefisso `chloros-cli`. Su Windows il file binario è `chloros-cli.exe`; su Linux /Jetson è `chloros-cli`.
- Gli argomenti opzionali sono indicati come `--flag`. Gli argomenti posizionali obbligatori sono indicati senza parentesi.
- Laddove sia specificato un valore predefinito, omettendo il flag si utilizza tale valore.
- L’CLI è un client leggero HTTP che utilizza il backend Chloros (server Flask su `127.0.0.1:5000`). Il backend viene avviato automaticamente dalla maggior parte dei comandi. `CHLOROS_BACKEND_URL=<url>` indirizza le famiglie di comandi **`lattice`**,**`project`**e**`daq pool-*`** verso un backend remoto — i comandi principali (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) bloccano deliberatamente `http://127.0.0.1:<port>` e lo ignorano (il valore IPv4 letterale evita la penalità di circa 2 secondi per richiesta prevista da Windows&#x27; per `localhost`→`::1`). Vedi [Variabili d’ambiente](#environment-variables).
- È richiesto l’accesso con un account Chloros+ per tutte le chiamate a SDK / CLI (eseguire `chloros-cli login` una volta per macchina; memorizzato nella cache in `~/.chloros/`).
- Gli esempi utilizzano percorsi Linux; su Windows sostituire `/home/user/...` con `C:/Users/.../...`.

---

## Sinossi di primo livello

```
chloros-cli [global options] COMMAND [command options]
```

### Opzioni globali

| Flag | Descrizione |
| --- | --- |
| `--backend-exe PATH` | Ignora l&#x27;eseguibile del backend rilevato automaticamente. |
| `--port N` | Porta di HTTP del backend (impostazione predefinita: `5000`). |
| `-v, --verbose` | Abilita l&#x27;output dettagliato. |
| `--restart` | Riavvia forzatamente il backend (termina qualsiasi processo `backend_server.py` in esecuzione). |
| `--version` | Visualizza la versione (`Chloros CLI 1.2.0`). |
| `--help` | Mostra la guida di primo livello. |

### Indice dei comandi

| Comando | Scopo |
| --- | --- |
| [`process`](#chloros-cli-process) | Elabora una cartella contenente acquisizioni Survey3 o LATTICE dall’inizio-to-end. |
| [`login`](#chloros-cli-login) | Autentica questo computer con un account Chloros+. |
| [`logout`](#chloros-cli-logout) | Cancella le credenziali memorizzate nella cache. |
| [`status`](#chloros-cli-status) | Mostra lo stato attuale della licenza / dell’autenticazione. |
| [`export-status`](#chloros-cli-export-status) | Visualizza lo stato di avanzamento dell’esportazione di Live Thread-4 durante l’esecuzione di `process`. |
| [`language`](#chloros-cli-language) | Imposta o elenca la lingua di visualizzazione di CLI (38 lingue supportate). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Cartella predefinita del progetto (condivisa con l’interfaccia grafica). |
| [`update`](#chloros-cli-update) | Verifica e installazione degli aggiornamenti di CLI (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnostica di sistema + test di funzionamento. |
| [`time-sync`](#chloros-cli-time-sync) | Stato e controllo del grandmaster PTP. |
| [`lattice`](#chloros-cli-lattice) | Controllo e acquisizione della telecamera LATTICE (oltre 45 sottocomandi). |
| [`daq`](#chloros-cli-daq) | Controllo del sensore spettrale DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Apertura ed esecuzione di un progetto salvato in formato Chloros (telecamere + DAQ). |

---

## Installazione

`chloros-cli` è incluso nel programma di installazione desktop Chloros su tutte le piattaforme supportate — non è disponibile un download separato CLI. L’installazione del pacchetto della piattaforma aggiunge `chloros-cli` al tuo `PATH` insieme all’app desktop e al binario di backend che gestisce.

Download più recenti: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Il programma di installazione include anche script di avvio (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) che aprono una shell CLI pronta all’uso; questi sono descritti nella [Guida per l’utente di CLI](../CLI.md) e non vengono ripetuti in questa sede.

### Windows (.exe)

1. Scaricare il programma di installazione di Windows dalla pagina di download.
2. Eseguire `Chloros-Setup-x.y.z.exe` e seguire la procedura guidata. Il percorso di installazione predefinito è `C:\Program Files\Chloros\` (il file CLI viene salvato in `C:\Program Files\Chloros\cli\`, che il programma di installazione aggiunge al PATH).
3. Aprire un nuovo terminale (`cmd.exe`, PowerShell o il terminale di Windows) in modo che venga rilevato il file `PATH` aggiornato.

```powershell
chloros-cli --version
```

Il programma di installazione aggiunge automaticamente `chloros-cli.exe` al PATH di sistema `PATH` e include il runtime Arena SDK necessario per le telecamere LATTICE.

### Linux amd64 (.deb)

Per Ubuntu 22.04 LTS o versioni successive / workstation x86_64 basate su Debian.

> **Ubuntu 20.04 non è supportato.** L&#x27;elenco delle dipendenze del pacchetto deriva da
> ciò a cui il backend si collega effettivamente, e ciò include `libc6 (>= 2.34)`;
> Focal distribuisce glibc 2.31. `apt` rifiuta l’installazione piuttosto che lasciarla fallire in
> fase di esecuzione.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Il pacchetto .deb installa:
- da `chloros-cli` a `/usr/bin/chloros-cli`
- Il backend compilato in `/usr/lib/chloros/chloros-backend`
- Il runtime Arena SDK (per le telecamere LATTICE)
- Modelli di denoising, pacchetti di calibrazione e configurazione del canale di aggiornamento

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Stesso layout del file .deb per amd64, con una build CUDA ottimizzata per Jetson Orin / Orin NX / Orin Nano.

### Autenticazione una tantum per macchina

Ogni piattaforma richiede un accesso una tantum a Chloros+ affinché le chiamate a SDK / CLI funzionino:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Le credenziali vengono memorizzate nella cache in `~/.chloros/user_session.json`.

### Verifica dell’installazione

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **È richiesto un abbonamento a Chloros+.**L’CLI richiede un piano Chloros+ attivo.**Copper**è il livello base Chloros+ — ogni livello a pagamento Chloros+ include l’accesso a CLI / SDK; solo il livello gratuito**Iron** non lo include. (Mappatura piano-id: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Effettua l’aggiornamento all’indirizzo [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Questo limite minimo è imposto dal backend, non solo dall’CLI: una richiesta contrassegnata con SDK / CLI senza un piano a pagamento viene rifiutata con `403 PLAN_UPGRADE_REQUIRED`, indipendentemente dal fatto che provenga da `chloros-cli`, da Python SDK o da un client HTTP personalizzato. Un chiamante disconnesso riceve invece il codice di errore `401 AUTH_REQUIRED`. L’accesso funziona offline per il periodo di grazia del piano (30 giorni al mese per il piano mensile, fino alla scadenza per quello annuale) e cessa al termine di tale periodo; `chloros-cli status` continua a funzionare, quindi il motivo è visibile (è l’unico percorso SDK / CLI esente dal filtro a livelli — `GET /api/license-status`).

---

## Guida rapida in cinque minuti

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Elaborare una cartella di immagini attraverso l’intera pipeline Chloros (rilevamento del bersaglio → calibrazione → vignetta → riflettanza → esportazione dell’indice).

### Sinossi

```
chloros-cli process INPUT [OPTIONS]
```

### Argomenti di posizione

| Argomento | Descrizione |
| --- | --- |
| `INPUT` | Percorso della cartella di input contenente i file `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) o `.dng`. |

### Opzioni comuni

| Flag | Impostazione predefinita | Descrizione |
| --- | --- | --- |
| `-o, --output PATH` | una nuova cartella con data e ora nel percorso predefinito del progetto (`~/Chloros Projects` se non diversamente configurato) | Cartella del progetto da creare o riutilizzare. Se la cartella contiene già un file `project.json`, verrà creata una cartella `_1`/`_2` al posto di sovrascriverla. |
| `-n, --project-name NAME` | automatico (data e ora) | Nome del progetto. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` utilizza un debayer neurale Chloros+; più lento ma di qualità superiore. |
| `--vignette / --no-vignette` | `--vignette` | Correzione della vignettatura. |
| `--reflectance / --no-reflectance` | `--reflectance` | Calibrazione della riflettanza (utilizza il target del pannello se rilevato, calibrazione NIST per serie per LATTICE). Per LATTICE multispettrale, questa opzione funge anche da interruttore per il **prodotto** di riflettanza — vedi [Interruttori di esportazione per prodotto](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | disattivato | Applica le correzioni GNSS PPK dai file sidecar. |
| `--exposure-pin-1 MODEL` | disattivato | Fissa il modello “pin-1” di un rig a doppia fotocamera Survey3. |
| `--exposure-pin-2 MODEL` | off | Fissare il modello &quot;pin-2&quot;. |
| `--recal-interval SECONDS` | 0 | Forza la rielaborazione dei calcoli di calibrazione ogni N secondi di tempo di acquisizione. |
| `--timezone-offset HOURS` | locale | Ignora l&#x27;offset del fuso orario incorporato nei metadati di output. |
| `--format FORMAT` | `TIFF (16-bit)` | Uno tra `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | nessuno | Indici di vegetazione (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Imposta forzatamente il punto di ingresso della pipeline per i file TIFF LATTICE (il formato Survey3.raw non è interessato). Inoltre, la via di fuga che consente di elaborare una cattura **senza raw** — vedi [Come si presenta una cartella delle catture](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | on | Emette il prodotto debayered lineare (`Debayered_Images`). Vedi [Opzioni di esportazione per singolo prodotto](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | on | Emette l’anteprima di visualizzazione (`Preview_Images`): RGB = bilanciamento del bianco (illuminante DAQ se disponibile, altrimenti “gray-world”) + gamma; multispec = espansione a colori falsi. |
| `--radiance / --no-radiance` | attivo | Emette radianza in float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Riferimento per il prodotto di riflettanza LATTICE: `auto` = il bersaglio all’interno dell’inquadratura che supera il controllo di qualità (QA) è il riferimento assoluto, fallback DAQ-downwelling (ρ = π·L/E); `target` = rigoroso (nessuna sostituzione DAQ); `daq` = autorevole DAQ. Vedi [Opzioni di esportazione per singolo prodotto](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | nessuno | Directory delle scansioni della riflettanza **misurata** del bersaglio per unità (`<serial>.csv`); in caso di mancata acquisizione, si ricorre agli spettri nominali T3/T4P. |
| `--array-alignment / --no-array-alignment` | on | Array LATTICE: applica l’allineamento da modulo a modulo indicato nel file XMP `Chloros:Alignment*` di ciascuna acquisizione a ogni prodotto elaborato (debayering / anteprima / radianza / riflettanza / indice). Nessuna operazione per le immagini prive dei tag. |
| `--array-alignment-crop / --no-array-alignment-crop` | ritaglio | Ritaglia le esportazioni allineate alla regione di sovrapposizione comune dell’array in modo che tutti i moduli condividano un’unica impronta; `--no-…` mantiene l’intera area del sensore (riempimento nero al di fuori della sorgente). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Ricampionamento per la distorsione dovuta all’allineamento. `nearest` conserva i valori DN esatti della sorgente (senza mescolanza inter-pixel dei valori radiometrici). |

### Opzioni di rilevamento del bersaglio

| Flag | Descrizione |
| --- | --- |
| `--min-target-size PIXELS` | Dimensione minima del pannello(px) per il rilevatore. |
| `--target-clustering 0-100` | Sensibilità di raggruppamento. |
| `--target / --targets` | Considera la cartella di input come contenente solo pannelli di bersaglio (salta il rilevamento delle indagini). |

### Esempi

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Opzioni di esportazione per singolo prodotto (LATTICE multispettrale)

L&#x27;elaborazione LATTICE si ramifica in **ogni prodotto applicabile in un unico passaggio**. Le quattro opzioni per tipo — `--debayered`, `--preview`, `--radiance`, `--reflectance` — sono tutti**ATTIVATI per impostazione predefinita**; utilizzare il modulo `--no-<type>` per disattivarne uno. Le cam master RGB emettono esclusivamente immagini debayered + anteprima (senza radianza/riflettanza per banda), quindi `--radiance`/`--reflectance` non hanno alcun effetto su di esse. I comandi di attivazione/disattivazione vengono ignorati per Survey3 `.raw` (che segue lo standard riflettanza/percorso target). *(Il vecchio flag `--radiometric-output {reflectance,radiance,sensor-response}` è stato **rimosso** e sostituito da questi flag; non esiste più il livello `sensor-response`.)*

| Prodotto | Uscita | È necessario il downwelling DAQ? |
| --- | --- | --- |
| `--debayered` | Demosaicatura lineare (`Debayered_Images`). | No. |
| `--preview` | Anteprima di visualizzazione (`Preview_Images`): RGB = WB + gamma; multispec = estensione a falsi colori. | No. |
| `--radiance` | float32 W/m²/sr/nm dalla catena radiometrica completa (`Radiance_Images`). | No. |
| `--reflectance` | riflettanza uint16 ρ (`32768` = 1,0), pronto per Pix4D. | **Sì**, a meno che non sia ancorata da un target all’interno del fotogramma che abbia superato il controllo di qualità (vedi sotto). |

`--reflectance-source` seleziona il riferimento di riflettanza:**`auto`**(impostazione predefinita) rende un bersaglio presente nell’inquadratura e che ha superato il controllo di qualità (QA) il**riferimento assoluto**— le catene di linee empiriche ancorate al bersaglio vengono sottoposte a valutazione incrociata su pannelli tenuti da parte e viene applicato il risultato vincente misurato — ricorrendo alla divisione di downwelling del DAQ (ρ = π·L/E) quando non è presente alcun bersaglio o il controllo di qualità (QA) fallisce;**`target`**è rigoroso (nessuna sostituzione DAQ);**`daq`**opta per il comportamento autoritativo del DAQ. La geometria del bersaglio (ArUco / ROI fissa / striscia) deriva dalla configurazione del bersaglio del progetto; `--target-reflectance-dir DIR` conserva le scansioni**misurate** per unità (`<serial>.csv`) individuate tramite il numero di serie/QR dell’unità di destinazione, con gli spettri T3/T4P nominali come alternativa.

Il percorso di riflettanza DAQ risolve automaticamente il **flusso discendente con timestamp corrispondente**da un**`.daq`**(DAQ-U/M/E)**o da un `.csv` nativo di DAQ-M**presente insieme alle immagini. Se un pacchetto di calibrazione per singola telecamera o per DAQ non è memorizzato nella cache locale, la pipeline**lo recupera automaticamente da AWS** al primo utilizzo (richiede una connessione a Internet una sola volta; viene memorizzato nella cache come `~/.chloros/`).

#### Lettura dei pixel di riflettanza (Pix4D / Metashape / script personalizzati)

La riflettanza è memorizzata come DN intero, e **il valore DN che corrisponde a ρ = 1,0 dipende dalla telecamera di origine**:

| Origine | ρ = 1,0 corrisponde a | Come identificarlo |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margine fino a ρ 2,0) | Il file riporta l’indicazione XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (limitato a ρ 1,0) | Nessun tag XMP `Chloros:*` — tale assenza *è* il segnale. |

**Leggi `Chloros:PixelScale` e dividi per esso** anziché ipotizzare un valore costante. Il tag è definito nel dominio uint16, quindi rimane `32768` in tutti i formati di output che effettuano un ridimensionamento — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` e `TIFF (32-bit, Percent)` sono tutti autodescrittivi (normalizzare prima il tipo di dati memorizzato riportandolo a uint16: ×257 da 8 bit, ×65535 da float).

> **Un caso non presenta alcuna scala, per impostazione predefinita.** Quando un&#x27;acquisizione con sorgente a 8 bit (BayerRG8) viene scritta come TIFF a 8 bit, la pipeline *climita* a 0..255 invece di riscalare, quindi ogni valore superiore a ρ≈0,008 viene appiattito a 255 e nessuna scala descrive il file. Chloros omette deliberatamente sia `Chloros:PixelScale` che la tupla `MicaSense:RadiometricCalibration` presenti in quel punto, e ne registra il motivo. **Se il tag è assente in un file di riflettanza LATTICE, non dare per scontata la presenza di una scala: riesportare a 16 bit o 32 bit** anziché dividere pixel che non sono mai stati divisibili.

#### Dati EXIF trasferiti nell’esportazione

`process` copia il **blocco GPS e il relativo ExifIFD** dell’acquisizione di origine su ogni prodotto, quindi un’
esportazione contiene `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` e
`CameraSerialNumber` insieme ai dati di georeferenziazione.

**`FocalLength` non è facoltativo per la fotogrammetria.** Pix4D calcola la distanza campionaria al suolo (GSD) in base alla
lunghezza focale più l’altitudine; in assenza del tag, ricorre a una scala estremamente errata. In un
volo di 49di un volo di acquisizione su un aranceto, la mancanza del tag ha trasformato un sito di 411 m × 160 m in uno ricostruito
di 47,8 km × 13 km — un&#x27;ortofoto da 455 MP composta per lo più da &quot;nodata&quot;, che è stata poi interpretata come un problema di tiling e
un problema BigTIFF prima che qualcuno controllasse il GSD. Se la vostra ortofoto risulta con una scala inverosimile,
eseguite prima `exiftool -FocalLength` sul prodotto esportato.

La copia è volutamente **non** `-all:all`: i tag strutturali di IFD0&#x27;i tag strutturali di IFD0 compromettono l’output di LATTICE quando
vengono copiati, e `ExifImageWidth` / `ExifImageHeight` sono esclusi perché descrivono l’
acquisizione *di origine* — un’esportazione che fosse stata ridimensionata in precedenza conterrebbe altrimenti dimensioni
in contraddizione con il proprio raster. L’XMP viene scritto direttamente anziché copiato, poiché ExifTool
scarta i tag XMP della stessa invocazione quando viene copiato il blocco XMP viene copiato (il che comporterebbe la perdita dei tag di calibrazione MAPIR
).

### Dove vengono salvati i file di output

I prodotti vengono salvati **nella cartella del progetto, raggruppati per fotocamera e poi per formato di file**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

La cartella della fotocamera è `LATT-<sensor>-<lens>-F<filter>` per LATTICE (in corrispondenza con l’EXIF dell’acquisizione
`Model`) e `<model>_<filter>` per Survey3 — due fotocamere che condividono sensore e filtro ma differiscono
nell’obiettivo mantengono alberi separati, poiché vignettatura, campo visivo e distorsione differiscono. La cartella
del formato segue `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` o `tiff32` per
`TIFF (32-bit, Percent)`.

> **Ogni prodotto esportato mantiene il nome del file ORIGINALE.** Un’esportazione Radiance di
> `capture_…_raw.tif` si chiama comunque `capture_…_raw.tif` — si trova semplicemente in
> `tiff32/Radiance_Images/`. **È la cartella a identificare il prodotto, non il nome del file**, quindi la ricerca con il carattere jolly
> per `*radiance*.tif` non trova nulla; è necessario invece effettuare la corrispondenza sulla directory.

### Registrazioni del sensore di luce — calibrate `.daq` + `.csv`

`process` gestisce anche le registrazioni `.daq` presenti nella cartella di input e **non**
richiede alcuna immagine per farlo: un DAQ-U / DAQ-M / DAQ-E utilizzato da solo è sufficiente per un’acquisizione completa
e una cartella contenente solo file `.daq` è un input valido.

È possibile registrare un DAQ **senza** la sua calibrazione — è ciò che i registratori pubblici
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) fanno per impostazione predefinita: scrivono i conteggi grezzi del sensore e contrassegnano il file in modo che
Chloros recuperi la calibrazione di fabbrica di quel sensore **tramite la porta seriale** (prima dalla cache locale,
poi dal cloud MAPIR) e la applichi. `process` riscrive il risultato:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

L’`.csv` contiene una riga per ogni lettura: timestamp UTC, tempo di integrazione, potenza totale,
lux fotopico/scotopico, PPFD (e la sua suddivisione in blu/verde/rosso), lunghezza d’onda di picco, quindi lo
spettro completo sullagriglia di lunghezze d’onda del sensore stesso. `.daq` reimporta i dati senza essere
calibrato una seconda volta.

In caso di esito positivo, l’esecuzione riporta `Light-sensor products written: N (calibrated .daq + .csv)`.
La parte tra parentesi descrive ciò che è stato effettivamente scritto, quindi si legge
`(RAW COUNTS — this sensor has no calibration bundle)` per un sensore senza bundle e
`(N calibrated, M raw counts)` per una cartella che contiene entrambi. Le intestazioni proprie del backend
`[DAQ-EXPORT]` e `[RUN-SUMMARY]` derivano la loro formulazione allo stesso modo: nessuno dei
tre può definire &quot;calibrata&quot; un&#x27;esportazione grezza.

Una registrazione DAQ-U / DAQ-M / DAQ-E il cui pacchetto di calibrazione non può essere recuperato — perché si è
offline o perché quel sensore non ha alcuna calibrazione in archivio — viene **saltata con una motivazione** su una
riga `[DAQ-EXPORT]`, senza mai essere salvata come file “calibrato” contenente conteggi grezzi.
Connettiti a Internet ed esegui nuovamente l’operazione. Il motivo è quello effettivamente
rilevato dal lettore per quel file (schema illeggibile, assenza di pacchetto, errore di scrittura), e il riepilogo dell’esecuzione
elenca motivi **distinti** — venti file saltati per un’unica causa vengono considerati come un’unica
causa, non venti ripetizioni della stessa.

#### Le registrazioni DAQ-A vengono esportate come conteggi grezzi

La famiglia **DAQ-A** è antecedente al sistema dei bundle per serie e non ha alcun bundle di calibrazione
da recuperare — viene invece calibrata sul campo rispetto a un bersaglio di riflettanza, motivo per cui
non ne ha mai avuto bisogno. Rifiutare quelle registrazioni avrebbe impedito loro di estrarre i propri
dati, quindi vengono esportate con un **nome diverso**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Un nome file diverso anziché un flag all’interno del file, poiché l’informazione deve rimanere intatta
anche se inviata via e-mail come semplice nome. L’intestazione `.csv` riporta
`raw spectral sensor counts (NOT irradiance)` e avverte che i valori sono comparabili
**all’interno** del file — che è esattamente lo scopo per cui la calibrazione basata su target li utilizza — e
non tra sensori diversi. Le colonne fotometrichedipendenti dalla potenza (potenza totale, lux fotopico e
scotopico, PPFD) vengono registrate come **NULL** anziché essere integrate dai conteggi, e il riepilogo
dell’esecuzione riporta `RAW COUNTS`, quindi i valori “esportati” in un log non può essere interpretato come irradianza.

Le registrazioni legacy **v1.01 / v1.02** (scritte da un DAQ-A-SD) non riportano l’epoca per ogni lettura,
ma solo l’ora di scrittura del file. Il programma di corrispondenza immagine↔downwelling continua a rifiutarli — abbinare un
fotogramma a un’ora di scrittura comporterebbe un errore non visibile — ma l’esportatore li legge, e
l’CSVa stampa `clock=daq_created_on`, quindi il prodotto indica su quale orologio si trova.

### Note

- `process` rileva automaticamente se la cartella è di tipSurvey3, LATTICE o mista.
- Lo stato di avanzamento viene trasmesso tramite Server-Sent Events; l’CLI mostra in tempo reale lo stato di avanzamento per ogni thread (Rilevamento, Analisi, Elaborazione, Esportazione).
- Per Linux /Jetson, CLI controlla lo spazio di swap e potrebbe visualizzare un avviso prima dell’elaborazione di cartelle di grandi dimensioni. Il debayer sensibile alle texture applica inoltre automaticamente un limite di frequenza della GPU sui dispositivi Jetson a basso consumo (Nano, Orin Nano).
- In caso di esito positivo, l’esecuzione riporta il numero di prodotti immagine scritti (`Image products written: N`).

#### Un&#x27;esecuzione che non scrive immagini fallisce

Se sono stati richiesti dei prodotti e l&#x27;esecuzione non ne ha scritto **nessuno** — solo `project.json` e
`calibration_data.json` — `process` lo considera un errore: visualizza
`Processing finished but wrote no image products.` e **termina con un valore diverso da zero**, quindi uno script può
rilevarlo. Il messaggio indica la cartella del progetto e le cause più comuni:

- la cartella di input non è stata riconosciuta come acquisizione (controlla il layout e `--input-level`), oppure
- tutti i prodotti richiesti sono stati ignorati in quanto non applicabili a quelle telecamere (ad es. richiesta di
  radianza/riflettanza da telecamere solo con modalità «RGB»).

Eseguire nuovamente con `--verbose` e controllare il log del backend per le righe `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
che spiegano le omissioni per singola telecamera che altrimenti non verrebbero riportate nell’output dell’CLI.

Un&#x27;esecuzione deliberata solo con metadati — tutti i prodotti disattivati e nessun `--indices` — è comunque un
**successo**, poiché in questo caso un output di immagini vuoto è il risultato corretto.

Lo stesso vale per un&#x27;**esecuzione solo con sensore di luce**: una cartella di registrazioni `.daq` non contiene immagini da esportare
per definizione, e l’esecuzione viene valutata in base ai file `.daq` / `.csv` calibrati che ha invece generato.

---

## `chloros-cli login`

Autentica questo computer con un account cloud Chloros+. Le credenziali vengono memorizzate in modo sicuro nella cache in `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Esempi

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (rimuovendo o duplicando parti della password). In caso di errore 401, l’CLIe riprova automaticamente aggiungendo nuovamente `$$`, poi con metà della password senza duplicati; se il tentativo va a buon fine, effettua l’accesso e visualizza la sintassi corretta con le virgolette singole da utilizzare la prossima volta.

> **Utilizzo senza interfaccia grafica/tramite script: l’assenza di una sessione memorizzata in cache comporta un prompt interattivo, non un errore immediato.** Qualsiasi comando che avvia un backend (`process`, `status`, `export-status`, `time-sync`, …) eseguito senza una licenza o una sessione memorizzata nella cache, passa a un prompt interattivo `Email:` / `Password:` su stdin prima di procedere. Un lavoro in modalità automatica senza una sessione memorizzata nella cache rimarrà quindi in attesa di input: eseguire `chloros-cli login EMAIL PASSWORD` una volta per ogni macchina prima di pianificare un’elaborazione senza interfaccia grafica.

---

## `chloros-cli logout`

Cancella la sessione memorizzata nella cache e forza un nuovo accesso alla successiva chiamata.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Mostra il livello di licenza corrente (Iron/Copper/Bronze/Silver/Gold), l’utente autenticato e il numero di associazioni del dispositivo.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Interroga lo stato di avanzamento in tempo reale dell’esportazione di Thread-4. È sicuro da chiamare **durante** l’esecuzione di `process` da un’altra shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Imposta la lingua di visualizzazione dell’CLI (38 lingue supportate, incluse CJK, RTL e Indic). Passa automaticamente all’inglese sulle console legacy che non sono in grado di visualizzare lo script.

```
chloros-cli language [LANG_CODE] [--list]
```

### Esempi

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Comandi relativi alla cartella del progetto

Questi comandi gestiscono il percorso predefinito della cartella del progetto (condiviso con l’interfaccia grafica).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Solo per Jetson. Verifica `version_url` rispetto a `/etc/chloros/update.conf` e propone di scaricare e installare il file corrispondente `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Su Linux /Jetson, l&#x27;CLI esegue anche un **controllo automatico degli aggiornamenti ad ogni avvio** (non bloccante, non ritarda mai l’esecuzione del comando): legge `/etc/chloros/update.conf`, memorizza il risultato nella cache per 1 ora in `~/.chloros/update_cache.json` e visualizza `Update available: vX.Y.Z / Run: chloros-cli update` quando è disponibile una versione più recente. In caso di errore, l’operazione viene saltata in modo silenzioso e su Windows.

---

## `chloros-cli selftest`

Esegue un test di funzionamento in 7 fasi: versione, disponibilità delle porte, avvio del backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), presenza del modello di denoising, prontezza di CUDA+denoising.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Stato e controllo del grandmaster PTP. L’host Chloros esegue il grandmaster PTP; le telecamere LATTICE e le unità DAQ-E si sincronizzano con esso per i timestamp tra dispositivi.

| Sottocomando | Descrizione |
| --- | --- |
| `status` | Mostra lo stato del grandmaster, le priorità BMCA e l’identità dell’orologio. |
| `peers` | Elenca gli slave rilevati tramite Delay_Req (telecamere + sensori DAQ-E). |
| `cameras` | Stato di integrità PTP per singola telecamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Riavvia il processo grandmaster. |
| `set-priority --priority1 N --priority2 N` | Ignora le priorità BMCA. |

### Esempi

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Controllo della telecamera LATTICE. Ogni sottocomando passa attraverso il backend Chloros; il backend gestisce il pool di telecamere, quindi le successive chiamate a CLI riutilizzano lo stesso handle aperto.

### Opzioni comuni (condivise dalla maggior parte dei sottocomandi)

| Flag | Descrizione |
| --- | --- |
| `-d, --device N` | Indice della telecamera (predefinito: 0). |
| `-s, --serial SN` | Numero di serie specifico; sovrascrive `--device`. |
| `--serials SN1,SN2,…` | Numeri di serie separati da virgola per il funzionamento multi-telecamera. |
| `--all` | Opera su tutte le telecamere rilevate. |
| `--exposure US` | Tempo di esposizione in microsecondi. |
| `--gain DB` | Guadagno in dB. |
| `--pixel-format FMT` | ad es. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Dimensioni dell&#x27;immagine. |
| `--preset {default,high_quality,high_speed,triggered}` | Applica un&#x27;impostazione predefinita. Tutti funzionano in modalità libera tranne `triggered`, che attiva la telecamera in presenza di un segnale hardware sulla linea 2 — se quella linea non riceve alcun segnale, il sistema attenderà all’infinito anziché effettuare l’acquisizione. |
| `-o, --output DIR` | Directory di output (impostazione predefinita: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Dimensione del pacchetto GVSP. `auto` esegue sondaggi ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Flusso di lavoro per la prima connessione della telecamera LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Riferimento ai sottocomandi

#### Rilevamento e informazioni

| Sottocomando | Scopo |
| --- | --- |
| `lattice info` | Elenca le telecamere connesse (produttore, modello, numero di serie, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analizza il sistema host per una configurazione ottimale della telecamera. `--no-discover` salta il rilevamento delle telecamere (più veloce, analisi solo tramite scheda di rete). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Verifica/correzione delle impostazioni della scheda di rete; stima della larghezza di banda e degli FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Capacità di rete del backend con schema stabile + raccomandazione sull’array (restituisce `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` mantiene la risoluzione richiesta ma limita il numero di fotogrammi al secondo (fps) di destinazione — leggere `recommended.recommended_target_fps` e passarlo come destinazione di connessione; considerarlo un successo, non un errore. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Analisi &quot;what-if&quot; senza aprire le telecamere. **`--n-active` è il numero totale di telecamere sulla rete, non solo di questo array**— generarlo quando le telecamere autonome trasmettono in streaming contemporaneamente, oppure quando il budget della rete viene calcolato in base a una domanda che ne sottostima il numero (impostazione predefinita: `len(--models)`). Visualizza sempre il valore aggregato `Wire budget:` (MB/s richiesti rispetto al limite di sicurezza contro le collisioni) e `Max cameras:`, e segnala con il flag `** OVER-SUBSCRIBED**` quando l’array sovraccarica la banda — vedi [Modello fps e burst dell’array](#array-fps--burst-model). |
| `lattice gpu` | Mostra lo stato della GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Verifica o aggiorna il firmware della telecamera. La selezione locale `.fwa` è bloccata: il file in `firmware/<MODEL_PREFIX>/` corrispondente alla build`MIN_FIRMWARE_VERSION` viene installato se presente (la versione più alta viene utilizzata solo come soluzione di ripiego), quindi un’immagine del fornitore più recente memorizzata sul disco rimane inattiva finché quel pin non viene aggiornato — le versioni più recenti raggiungono le unità tramite il manifesto AWS firmato, che è preferibile quando più recente. |
| `lattice presets [--apply NAME]` | Elenca o applica le impostazioni predefinite della telecamera. |
| `lattice status` | Stato in tempo reale della telecamera. |

#### Acquisizione

| Sottocomando | Scopo |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Singolo fotogramma. **Salva ogni tipo di esportazione per impostazione predefinita** (`--processing all`); vedere [Livelli di esportazione dell’acquisizione](#capture-export-levels-the-all-default). `--levels` salva un sottoinsieme esplicito (sovrascrive `--processing`); `--force-daq` scrive la lettura DAQ assegnata come immagine secondaria `.daq`anche in caso di acquisizione solo raw. `--jpeg-quality` = JPEG qualità 1–100 (predefinita 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Trasmette in streaming su disco fino a Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Anteprima MJPEG in tempo reale basata su browser. `--ae-damping` imposta lo smorzamento dell’esposizione automatica (0,4–100). |

#### Regolazione del sensore

| Sottocomando | Scopo |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Leggi/scrivi qualsiasi nodo GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Esposizione e AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Guadagno e guadagno automatico. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | ROI del sensore e bin. |
| `lattice format [--set FMT] [--list]` | Formato pixel. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Trigger hardware/software. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (nessun flag = WB a scatto singolo) | Operazioni WB. Solo telecamereRGB/Bayer; un&#x27;operazione nulla (saltata) su M3M mono. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | Pipeline dei colori per la visualizzazione “RGB”. `natural` (impostazione predefinita) è la finitura live economica; `enhanced` aggiunge defringe + vivacità + contrasto locale CLAHE per ottenere il look completo da hub-parity a circa il doppio del costo di finitura per fotogramma, quindi un framerate **live** — le acquisizioni salvate ottengono comunque la finitura completa in entrambi i casi. Solo per telecamere RGB /Bayer; saltato su M3M mono. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Visualizza saturazione/contrasto (telecamere con filtro RGB). Saltato su M3M mono. |
| `lattice filter [--set NAME] [--list]` | Imposta il modello di filtro della telecamera (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Rileva i nodi di potenza/termici; attiva/disattiva la modalità di inattività a basso consumo. |

#### Calibrazione e sensori

| Sottocomando | Scopo |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Calibrazione tramite un bersaglio di riflettanza. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Comandi integrati per il sensore di luce discendente. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Applica la correzione della vignettatura alle immagini esistenti. |

#### Multicamera (sessioni transitorie)

| Sottocomando | Scopo |
| --- | --- |
| `lattice multi-info` | Elenca tutte le telecamere con ruoli di sincronizzazione. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Un fotogramma sincronizzato da ciascuna telecamera. Salva **tutti i tipi di esportazione per impostazione predefinita**quando è collegato un array persistente; il fallback transitorio senza array è**solo debayered** (eseguire prima `array-connect` per il resto). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Trasmissione di fotogrammi sincronizzati (transitoria). |
| `lattice multi-test [--count N]` | Test di temporizzazione della sincronizzazione GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Rilevamento automatico del cablaggio master/slave GPIO. |

#### Allineamento

| Sottocomando | Scopo |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — più manopole del rilevatore/corrispondenza `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, manopole RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, combinazione di-frame `[--averaging mean\|median\|inlier_weighted]`, vincoli geometrici `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, restrizione spaziale `[--roi X0,Y0,X1,Y1] [--mask PATH]` e sovrascritture per singolo slave `[--per-cam-override SN:KEY=VALUE]` () | Calcola il profilo di allineamento dalle telecamere in tempo reale. `--prefilter` utilizza come impostazione predefinita `gradient` (mappa dei bordi; corrisponde all’allineatore GUI/array — i bordi rimangono invariati tra le bande spettrali). `--matcher flann` dà risultati migliori con più di ~5000 caratteristiche; `--averaging median` è robusto nei confronti di una singola acquisizione errata, `inlier_weighted` pondera in base al numero di corrispondenze; `--lock-scale` proietta sulla rotazione più vicina (senza scala), `--lock-axis` azzera una componente di traslazione; `--mask` si applica a tutte le telecamere (utilizzare `--per-cam-override` per le impostazioni per singola telecamera, ad es. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` rifiuta di salvare una calibrazione il cui RMS di riproiezione superi il limite. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Acquisisce un fotogramma multi-banda allineatobanda allineato. `--bit-depth` imposta per impostazione predefinita l’adattamento alla fotocamera; `--no-crop` mantiene l’intero fotogramma (riempiendo con nero); `--interpolation` (impostazione predefinita `linear`) e `--border-mode`/`--border-value` (impostazione predefinita `constant`/0) controllano il warp della CPU — il percorso della GPU è comunque bilineare. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Fotogrammi multibanda allineati al flusso (stesse regolazioni di warp di `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Visualizza i dettagli del profilo. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Modifica l’ordine dei livelli. |

#### Indice / Calcoli sulla vegetazione

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Set completo di flag: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (ripetibile), `--capture-level raw|debayered|radiance|reflectance|unknown` (sovrascrive il livello di acquisizione registrato nell’TIFF di origine; impostazione predefinita: lettura dai metadati di TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Con `--live` si applicano anche le manopole di deformazione dell’allineamento: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **I simboli `--channel` distinguono tra maiuscole e minuscole.** Il lato del simbolo deve corrispondere esattamente ai nomi dei canali del preset (i preset utilizzano lettere minuscole, ad es. NDVI = `red`, `nir` — verificare `--list-presets`), mentre la parte relativa alla banda deve corrispondere a un nome di banda presente nello stack allineato (oppure essere unin modalità offline). `--channel red=Red_660 --channel nir=NIR_850` funziona; `--channel RED=660` genera un errore `channel_map missing entries`.

#### Connessioni persistenti (Smart-Prep, flusso equivalente alla GUI)

Questi comandi mantengono le telecamere aperte nel pool di backend tra una chiamata all&#x27;CLI.

| Sottocomando | Scopo |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Aggiunge una telecamera al pool (singola telecamera, senza array). |
| `lattice cam-disconnect [--serial SN] [--all]` | Rilascia. |
| `lattice cam-list` | Elenca le telecamere nel pool. |
| **`lattice array-connect`**|**Collega un array sincronizzato persistente (IL punto di ingresso consigliato).** Esegue l’intero flusso di preparazione intelligente tramite GUI. |
| `lattice array-disconnect [--array-id ID] [--all]` | Rilascia un array. |
| `lattice array-list` | Elenca gli array collegati. |
| `lattice array-status [--array-id ID]` | FPS in tempo reale, PTP, ultimo errore. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Una acquisizione sincronizzata dall’array in tempo reale — Singola / Continua / A intervalli / Più veloce. **Impostazione predefinita: `all`** (un file per ogni tipo di esportazione applicabile per ogni telecamera). Le telecamere saltate (ad es. RGB escluse dalla radianza/riflettanza) vengono segnalate con `Skipped: SN:<serial> (<reason>)`; la lettura DAQ utilizzata per la riflettanza viene salvata insieme e segnalata con `DAQ: <path>`. Vedi [Modalità di acquisizione, registratori e Rielaborazione offline](#modalità-di-acquisizione-registratori--rielaborazione-offline). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Registra la vista dell’indice combinato in tempo reale su video/GIF (livello di monitoraggio; richiede che il flusso combinato sia aperto). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Burst Bayer grezzo ad alto fps (di livello analitico; rielaborazione offline). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Rielabora un burst grezzo salvato in video calibrato. |

##### Opzioni `array-connect`

| Flag | Predefinito | Descrizione |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Rilevamento automatico di tutte le telecamere LATTICE (ne occorrono ≥2) | La prima in sequenza è la MASTER. Se omesso, il rilevamento filtra i modelli LATTICE (`TRI032*`) e li collega tutti. |
| `--line {Line0,Line2,Line3}` | `Line2` | Linea di sincronizzazione GPIO. |
| `--target-fps F` | auto | Frequenza di attivazione del trigger MASTER. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Ignora il selettore di livello. |
| `--wire-ceiling-mbps MB_PER_S` | rilevato automaticamente | **Il budget di banda sostenuto dall’host, in MB/s — il valore da cui dipende l’allocazione dell’intero array.** Ridurlo quando l’array segnala frame con GVSP danneggiato: il valore automatico deriva dalla velocità di collegamento pubblicizzata dalla scheda di rete, che sovrastima gli adattatori USB, le linee PCIe sottodimensionate e i fabric condivisi molto trafficati. Viene salvato nel blocco di acquisizione dell&#x27;array del progetto, quindi una riconnessione tramite «reopen», «CLI» o «SDK» lo ripristina. Vedi [Stato di salute dell’array](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Binning hardware. |
| `--no-recommend` | off | Salta la fase di analisi della rete. |
| `--no-ptp` | off | Disabilita il PTP (i timestamp tra le telecamere **non** sono quindi comparabili). |

### Smart-AE / Smart-Capture

Gli array LATTICE eseguono l’AE in modo continuo in background non appena vengono collegati, ma una scena appena inquadrata richiede un po’ di tempo per convergere. `array-capture --smart` è la **soluzione predefinita**: attende che l’AE si stabilizzi su tutte le telecamere dell’array, quindi avvia l’acquisizione. Utilizzarlo quando si cambia scena durante una sessione.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Per impostazione predefinita, la politica di stabilizzazione è conservativa: timeout di 5 s, finestra di stabilità di 1,5 s, tolleranza di variazione dell’esposizione di ±5 %. Regolate i parametri tramite l’SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) se avete bisogno di un comportamento diverso dall’automazione.

### Livelli di esportazione delle acquisizioni (impostazione predefinita `all`)

A partire da questa versione, `lattice capture`, `lattice multi-capture` e `lattice array-capture` **impostazione predefinita su `--processing all`** — un file salvato per ogni tipo di esportazione applicabile a ciascuna telecamera, in linea con il comportamento dell’opzione “Acquisisci tutto” dell’interfaccia grafica. I livelli sono:

| Livello | Output | Si applica a |
| --- | --- | --- |
| `raw` | Bayer a canale singolo (telecamere monocromatiche: la singola banda) direttamente dal sensore. | Tutte le telecamere. |
| `debayered` | Demosaicatura BGR a 3 canali (telecamere monocromatiche: scala di grigi a 1 canale). | Tutte le telecamere. |
| `radiance` | float32 W/m²/sr/nm tramite la catena radiometrica completa. | Solo multispettrali (M3C/M3M) — **omesso per le telecamere con filtro “RGB”**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Compatibile con Pix4D. | Solo multispettrale, e **solo se è associato un DAQ e la fotocamera è calibrata**; altrimenti saltato. |
| `preview` / `display` | Catena completa di anteprima GUI (CCM + WB + gamma secondo il profilo della fotocamera). `lattice capture` assegna questo nome a `preview`; `array-capture`/`multi-capture` utilizza `display`. | Tutte le telecamere. |

Specificare un singolo livello per salvare solo quello (`--processing debayered`). Quando si richiede `all`, i livelli che non si applicano a una determinata telecamera vengono saltati (e segnalati), ma non generano errori: una telecamera non collegata o non calibrata riceve comunque `raw` / `debayered` / `preview`.

Per qualsiasi fotogramma di riflettanza, la lettura DAQ in direzione discendente effettivamente utilizzata viene scritta in un file sidecar **`.daq`** accanto all’immagine (in modo che l’acquisizione possa essere rielaborata in seguito) e riportata su una riga `DAQ:`.

### Come si presenta una cartella di acquisizioni

Ogni tipo di esportazione viene collocato nella propria **sottocartella** all’interno di `-o`, in modo che un’acquisizione a più livelli non mescoli mai i tipi:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` è il timestamp dell’acquisizione e `<serial>` il numero di serie della telecamera, quindi un gruppo sincronizzato condivide un
timestamp tra tutte le telecamere. **Si noti l’unica asimmetria:** il livello `display` viene memorizzato in una cartella
denominata `preview/`, mentre i file stessi mantengono `_display` nel nome — la cartella e il suffisso differiscono
solo per quel livello. I livelli sconosciuti vengono inseriti in una cartella con il proprio nome e, se la sottocartella
non può essere creata, il file viene scritto nella radice di output anziché andare perso.

**Rielaborazione di una cartella di acquisizioni:**indicare `chloros-cli process` come**radice delle acquisizioni**
(`output/`). `process` normalmente importa solo la cartella specificata, ma quando tale cartella non contiene
immagini e presenta sottocartelle, esplora automaticamente il percorso a livello inferiore — in questo modo lee le
sottocartelle della radice `.daq` vengono tutte acquisite in un’unica operazione. Ogni livello di un’acquisizione viene importato come singola immagine, con
gli altri livelli disponibili come modalità, anziché come un’immagine per livello.

È possibile anche specificare direttamente il nome di una **sottocartella di livello** (ad es. `output/raw/`). In questo modo la radice
`.daq`, quindi copiare o far puntare la lettura DAQ parallelamente quando si ricava un prodotto radiometrico
da `raw/` — altrimenti la corrispondenza del timestamp non ha alcun riferimento a cui risolversi.

**L’elaborazione parte sempre da `raw`.** All’interno di ogni acquisizione, il fotogramma grezzo è la fonte della pipeline;
`debayered`, `radiance`, `reflectance` e `preview` sono presenti come modalità visualizzabili ma non vengono mai reimmessi
nella pipeline. La rielaborazione di un prodotto derivato comporterebbe la riapplicazione di vignettatura, CCM e
calcoli di radianza già integrati nei suoi pixel; pertanto, &quot;Chloros&quot; rifiuta l’operazione piuttosto che
effettuare una doppia elaborazione. Due conseguenze da tenere presenti:

- I rendering `index/` e `composite/` **non vengono mai** elaborati. Sono output, non acquisizioni —
  un rendering LUT NDVI non ha alcuna interpretazione significativa in termini di radianza.
- Una cartella di acquisizioni esportata **senza** `raw` (ad es. `array-capture --processing reflectance`) non ha
  alcuna fonte valida nella pipeline. Tali acquisizioni vengono importate e visualizzate normalmente, ma `process` le ignora
  e lo segnala:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Se si ha davvero bisogno di far passare un prodotto derivato — una sessione hub acquisita con
  `demosaic` attivo, o una cartella legacy — `--input-level {raw,debayered,processed}` forza l’inserimento
  e ignora il salto. Quel flag è una via di fuga intenzionale; `auto` (l’impostazione predefinita)
  non elabora mai un’acquisizione priva di dati grezzi.

### Acquisizioni saltate in array con filtri misti

Quando si combinano telecamere a RGBe e multispettrali in un unico array, `array-capture --processing radiance` (o `reflectance`) salva i fotogrammi multispettrali e **salta** le telecamere RGB — la radianza per pixel Bayer non è significativa per un sensore a banda larga. Il file CLI riporta esplicitamente ogni file salvato (con il relativo livello di esportazione), ogni `.daq` scritto e ogni salto, quindi il numero di file non è sorprendente:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

I token che indicano il motivo del salto seguono lo schema `<level>-not-applicable-to-rgb-cam`. La riflettanza può anche essere saltata con `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, e con `dls-uncalibrated-band-<nm>` quando la banda si trova prevalentemente al di fuori dell’intervallo radiometricamente calibrato del sensore di luce DAQ (~374–974 nm) — tra gli SKU disponibili solo l’F988, il cui percorso supportato è il flusso di lavoro con pannello di riflettanza.

Utilizzare `--processing debayered` (o `display`) per includere tutte le telecamere indipendentemente dal tipo di filtro, oppure l’impostazione predefinita `all` per ottenere tutti i livelli applicabili per ciascuna telecamera in un’unica operazione.

---

## Modalità di acquisizione, registratori e rielaborazione offline

Tutte queste funzioni operano su un **array persistente** (eseguire prima `array-connect`). Rispecchiano il pannello di acquisizione dell’interfaccia grafica.

### Modalità `array-capture`

`array-capture` è un singolo comando con quattro modalità di scatto più una serie di opzioni di esportazione:

| Modalità | Flag | Comportamento |
| --- | --- | --- |
| **Singola** *(predefinita)* | (nessuna) | Un gruppo di acquisizione sincronizzato, poi uscita. |
| **Continuo** | `--continuous` | Passaggi consecutivi fino a `Ctrl+C`, `--count N`, o `--duration S`. |
| **Intervallo** | `--interval S` | Un passaggio ogni `S` secondi (misurati dall’inizio di ogni passaggio), stessi limiti. |
| **Più veloce** | `--fastest` | Solo dati grezzi + la lettura DAQ assegnata + il composito a indice combinato; salta i calcoli di radianza/riflettanza/visualizzazione in modo che il fotogramma venga elaborato rapidamente. Implica `--processing raw --force-daq`. Rielaborare in un secondo momento i dati salvati `.daq` in prodotti calibrati. |

Opzioni di esportazione (combinabili con qualsiasi modalità; tutte condividono l’interfaccia grafica e l’endpoint SDK):

| Flag | Effetto |
| --- | --- |
| `--processing LEVEL` | Livello di esportazione singolo, oppure `all` (impostazione predefinita). |
| `--levels L1,L2,…` | Sottoinsieme esplicito di tipi di esportazione (ad es. `raw,radiance,reflectance`); **sostituisce `--processing`**. |
| `--aligned` / `--no-aligned` | Allinea l’esportazione non raw di ogni membro al [profilo di allineamento](#) (co-registrato). I dati raw rimangono non allineati ma riportano la trasformazione nei metadati. Si ricorre all’allineamento non specificato (con un avviso) se l’array non ha un profilo. |
| `--index` / `--no-index` | Salva / salta la sovrapposizione dell’indice di vegetazione per singola telecamera, laddove sia configurata. Impostazione predefinita: renderizzarla. |
| `--force-daq` | Salva la lettura DAQ/DLS assegnata come sidecar `.daq` anche quando nessun livello selezionato ne ha bisogno (ad es. un’acquisizione solo raw), in modo che i fotogrammi possano essere rielaborati in riflettanza/indice offline. |
| `--smart` | Attendere che l’AE si stabilizzi su tutte le telecamere prima dell’attivazione (vedere [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Compressione pixel TIFF. `deflate` (impostazione predefinita) = zlib L1 senza perdita di dati + predittore orizzontale, ~4,1 MB per fotogramma a piena risoluzione; `none` = non compresso, scrittura circa 5 volte più veloce a circa 6,3 MB per fotogramma — da utilizzare per la massima velocità sostenuta quando lo spazio su disco lo consente. Entrambe le opzioni sono senza perdita di dati e vengono lette in modo identico durante l’importazione. |

> **TIFFa a scrittura singola + il modello a velocità sostenuta.**Le acquisizioni vengono scritte in**un**unico passaggio del file TIFF contenente pixel + XMP + IFD0 Marca/Modello (misurato su Mono12 a piena risoluzione: 36 ms compresso / 6,5 ms non compresso, contro ~148 ms per il vecchio metodo “scrittura e poi riscrittura con ExifTool”); l’unico lavoro rimanente di ExifTool (rifinitura del sub-IFD EXIF) viene eseguito su un processo asincrono in background, e un fotogramma è completo e pronto per l’importazione anche se tale processo non viene mai eseguito. Si noti che la compressione DEFLATE mantiene il GIL di Python, quindi le scritture compresse**non**vengono parallelizzate tra i thread di scrittura di ciascuna fotocamera — acquisizione sostenuta a piena risoluzione con 8 fotocamere alla velocità del sensore (~10,4 fps) richiede `--compression none`**e** un disco di classe NVMe (~500 MB/s di scrittura sostenuta). La stessa opzione è disponibile come `compression` su `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — video/GIF con indice combinato (livello di monitoraggio)

Registra tutto ciò che viene visualizzato nella **vista combinata in tempo reale** su un `.avi` (e, facoltativamente, su un `.gif`). Poiché attinge al composito in tempo reale, il flusso combinato deve essere aperto (adad es. l’array è in anteprima nell’interfaccia grafica) affinché i fotogrammi vengano acquisiti. Verifica lo stato di avanzamento ogni 2 s e si interrompe su `--duration`, `Ctrl+C` o quando il registratore termina automaticamente.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Flag | Predefinito | Descrizione |
| --- | --- | --- |
| `--array-id ID` | solo array | Array di destinazione (omettere se ne è collegato solo uno). |
| `-o, --output DIR` | `output` | Directory di output (locale sul backend). |
| `--fps F` | `10` | Frequenza fotogrammi di registrazione. |
| `--duration S` | fino a Ctrl+C | Arresto automaticodopo `S` secondi. |
| `--gif` | disattivato | Scrivi anche una GIF animata. |
| `--gif-only` | disattivato | Scrivi solo una GIF (senza `.avi`). |

### `array-burst` — raffica raw-Bayer ad alto fps (di livello analitico)

Legge direttamente il buffer del gruppo sincronizzato del ciclo di acquisizione — **non sono necessari né la catena di calibrazione, né exiftool, né la visualizzazione in tempo reale** — quindi funziona alla massima velocità di acquisizione della fotocamera. Scrive fotogrammi raw + un manifesto per ogni fotogramma + un `.daq` per ogni lettura DLS distinta sotto `<output>/bursts/<base>/`. Rielaborare offline (comando successivo), oppure passare `--build` per eseguirla immediatamente all’arresto.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Flag | Predefinito | Descrizione |
| --- | --- | --- |
| `--array-id ID` | solo array | Array di destinazione. |
| `-o, --output DIR` | `output` | Directory di output (il burst viene salvato in `<DIR>/bursts/<base>/`). |
| `--duration S` | fino a Ctrl+C | Arresto automatico dopo `S` secondi. |
| `--max-frames N` | illimitato | Arresto automatico dopo `N` fotogrammi grezzi. |
| `--build` | disattivato | Dopo l’arresto, rielaborare immediatamente la raffica (come in `array-build-video`). |
| `--products …` | `combined:index` | Con `--build`: quali video creare (vedi sotto). |
| `--fps F` | `10` | Con `--build`: fps del video in uscita. |
| `--save-tiffs` | disattivato | Con `--build`: salva anche i file TIFF calibrati per ogni fotogramma. |
| `--gif` | disattivato | Con `--build`: scrivere anche GIF animate. |

### `array-build-video` — rielaborazione offline di una sequenza salvata

Allinea temporalmente ogni fotogramma grezzo alla lettura `.daq` salvata più vicina e lo fa passare attraverso la **stessa catena di radianza / riflettanza / indice della pipeline di importazione**, generando uno o più video.

`--products` è un elenco separato da virgole di elementi `kind:level`, dove `kind` ∈ `per_cam` | `combined` e `level` ∈ `radiance` | `reflectance` | `index`. Un `level` (senza `kind:`) assume per impostazione predefinita il valore di `per_cam`. Il valore predefinito è `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Flag | Impostazione predefinita | Descrizione |
| --- | --- | --- |
| `--burst-dir DIR` | (obbligatorio) | Percorso della cartella burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Elenco `kind:level`, come sopra. |
| `--fps F` | `10` | FPS del video in uscita. |
| `--save-tiffs` | off | Salva anche file TIFF calibrati per ogni fotogramma insieme ai video. |
| `--gif` | off | Scrive anche GIF animate. |

> **Scegli il registratore giusto.** `array-record` è di *livello monitoraggio* — acquisisce il composito in tempo reale così come viene visualizzato e richiede che lo stream sia aperto. `array-burst` → `array-build-video` è di *livello analisi* — salva i dati grezzi del sensore alla massima frequenza e ricostruisce successivamente video calibrati di radianza/riflettanza/indice, senza necessità di visualizzazione in tempo reale.

### Telecamere monocromatiche (M3M) a banda singola

La linea **M3M**è la versione monocromatica della**M3C**con filtro Bayer: un filtro di interferenza a banda stretta per telecamera (`M3M-<lens>-F<wavelength>`, ad es. `M3M-L87-F685`), quindi il sensore fornisce una**singola banda in scala di grigi** senza mosaico Bayer. Non c’è nulla da demosaicare, nessun crosstalk tra i canali da separare e nessun bilanciamento del bianco da impostare: l’intera pipeline di elaborazione del colore RGB -display semplicemente non si applica.

Cosa significa questo per le CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**rilevano una telecamera monocromatica e**passano oltre con un messaggio di una riga** invece di imporre impostazioni prive di senso. Continuano a funzionare normalmente con una videocamera RGB /Bayer M3C nella stessa sessione.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** funzionano ancora: la radianza e la riflettanza sono *per-banda* e sono perfettamente ben definite per una singola banda. I fotogrammi mono contengono una matrice di risposta del sensore **identitaria** (senza separazione 3×3), quindi il piano passa attraverso i calcoli di calibrazione senza subire modifiche.
- **Una singola telecamera mono non può produrre un indice di vegetazione.**NDVI / NDRE / ecc. richiedono almeno due bande (ad es. Red + NIR). Per ottenere un indice da hardware monocromatico, puntare**diverse** telecamere M3M su diverse lunghezze d’onda, allinearle in un unico stack multibanda e calcolare l’indice *da quello*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel`; i simboli devono corrispondere **esattamente** ai nomi dei canali del preset (si distingue tra maiuscole e minuscole; quelli di NDVI sono in minuscolo: `red`,`nir` — vedi `--list-presets`), e il nome della banda indica una banda nello stack allineato (la modalità offline accetta anche indici di banda basati su 0, ad es. `--channel red=0 --channel nir=1`).

Il discriminatore in tutto lo stack è il token `M3M` nella stringa del modello (non compare mai in una stringa `M3C`), visualizzato nell’interfaccia grafica/SDKe come `is_mono`.

---

## Configurazione e ottimizzazione della scheda di rete dell’host (array LATTICE)

Le telecamere LATTICE trasmettono GVSP tramite la scheda di rete Ethernet dell’host, quindi per gli array multicamera il **driver**della scheda e la**dimensione del ring di ricezione** sono importanti tanto quanto la velocità di collegamento. Impostazioni errate si manifestano come un gate `FRAMES WILL DROP` / `Reduce ROI to enable` nel pannello Impostazioni array (e in `lattice network-analysis` / l’SDK `analyze_array_network()`), anche quando le telecamere sono in perfetto stato.

### Adattatori USB 10GbE — Realtek RTL8157 (&quot;Realtek USB 10GbE Family Controller&quot;)

| Voce | Valore richiesto | Perché è importante |
| --- | --- | --- |
| **Versione del driver**|**≥ v10.67 (gennaio 2026)**, INF `rtump64x64sta.inf` | Il driver legacy**del 2016**(v10.65, `rtump64x64.inf`) gestisce in modo errato lo spegnimento e genera bugcheck con**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**durante lo spegnimento, il riavvio o la modalità di sospensione. La transizione si blocca (timeout di circa 5 minuti); se l’utente esegue lo spegnimento forzatoe gli spegnimenti non corretti ripetuti**corrompono il repository WMI**(PowerShell e gli strumenti iniziano a dare errori con `Invalid class`) e**bloccano lo stack USB** all’avvio successivo (la scheda di rete non si abilita; le unità USB smettono di essere riconosciute). Effettuare l’aggiornamento da realtek.com (o dal fornitore del dongle) prima di affidarsi a riavvii corretti. |
| **Buffer di ricezione**— parola chiave `ReceiveBufferLen` |**256**(massimo del driver) | L’anello RX della scheda di rete. L’impostazione predefinita del driver di**32**lascia solo circa 0,26 MB di anello utilizzabile — decisamente troppo piccolo per un burst multi-camera — quindi il pannello dell’array segnala `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` e blocca le connessioni. A**256**l’anello è ampio (**circa 13,5 MB misurati sull’host 10GbE del laboratorio**), fornendo alla pipeline RX un margine reale per i burst GVSP multicamera. (Il fatto che una determinata configurazione riesca effettivamente a *connettersi* è determinato da due controlli — il controllo di ammissione **drain-aware**e il controllo di**sovrasottoscrizione aggregata** — non da un confronto tra burst grezzorispetto all’anello; vedi [Modello fps e burst dell’array](#array-fps--burst-model).) |
| **URB in ricezione**— parola chiave `PendingReceives` |**64** (max) | Blocchi di richiesta USB in transito; aumentare insieme ai buffer di ricezione per l’assorbimento dei burst. |
| **Jumbo Frame** — parola chiave `*JumboPacket` | **9014** | Necessario per pacchetti GVSP da 9000 byte (6 volte meno pacchetti per frame rispetto a quelli da 1500). |

> ⚠️ **Un aggiornamento del driver della scheda di rete RIPORTA queste proprietà avanzate ai valori predefiniti.**Dopo aver aggiornato o sostituito il driver della scheda,**riapplicare** `ReceiveBufferLen=256` e `PendingReceives=64`, altrimenti il pannello dell’array si bloccherà nuovamente anche se “non è cambiato nulla nell’hardware”. Questa è la causa principale per cui un sistema che prima funzionava improvvisamente si rifiuta di connettersi.

Applicare da una sessione di PowerShell con **privilegi elevati** (sostituire il nome della scheda, ad es. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` copre gli adattatori USB 10GbE.** Ora rileva il tipo di scheda e imposta la parola chiave corretta per il receive-ring: `*ReceiveBuffers`→2048 per le schede di rete PCIe (Intel I219, ecc.), oppure `ReceiveBufferLen`→256 + `PendingReceives`→64 per il controller **USB** 10GbE Realtek (che non espone `*ReceiveBuffers`). I valori di destinazione sono limitati al massimo segnalato da ciascun driver (`NumericParameterMaxValue`), in modo che non venga mai scritto un valore fuori intervallo. Eseguirlo da un terminale **con privilegi elevati**; come per qualsiasi ottimizzazione basata sul registro, la modifica avrà effetto solo dopo il riavvio della scheda o del sistema. I comandi manuali `Set-NetAdapterAdvancedProperty` sopra indicati rimangono una valida alternativa: si applicano in tempo reale (riconfigurando la scheda) senza bisogno di riavvio.

### Nozioni di base sulla rete (tutti i collegamenti LATTICE)

- **Indirizzamento:** link-local `169.254.0.0/16` (GigE Vision LLA). L’host utilizza un indirizzo statico `169.254.x.x/16`; le telecamere e il DAQ-E si autoassegnano nella stessa gamma. Non sono necessari DHCP né gateway.
- **Dimensione del pacchetto:**preferibile jumbo (9000), ma lasciare che sia l’auto-probe a individuarla — la rimisura ad ogni connessione e supera già il limite ICMP di 1500 byte della telecamera tramite una sonda GVSP, quindi si imposta su jumbo ovunque la linea lo supporti effettivamente. Impostare un valore fisso con `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` solo se si è più sicuri della sonda, e preferire l’impostazione per singolo comando a quella permanente: un pin bypassa la sonda, quindi se il percorso non è effettivamente in grado di trasportare 9000,**ogni** acquisizione va in timeout con `SC_ERR_TIMEOUT -1011` (vedi [Variabili d’ambiente](#environment-variables)).
- **L’anello RX varia proporzionalmente a `ReceiveBufferLen`:**al valore predefinito `32` l’anello utilizzabile è di ~0,26 MB (troppo piccolo per qualsiasi burst multi-camera); al valore massimo `256` è ampio (~13,5 MB misurato sull’host da 10GbE del laboratorio), offrendo un margine reale. La possibilità che una configurazione si connetta viene quindi determinata dal controllo di ammissione sensibile al drenaggio**e** dal controllo di oversubscription aggregato riportato di seguito — non da un semplice confronto tra burst grezzo e anello.

### Modello di fps e burst dell’array

Come interpretare il pannello delle impostazioni dell’array (e `lattice analyze-array` / `analyze_array_network` dell’SDK):

- **Il burst viene sommato per ciascuna telecamera in base al formato pixel reale di ciascuna telecamera.**Le telecamere**M3M**in mono trasmettono**Mono12 (2 B/px)**; le telecamere**M3C**con sensore Bayer trasmettono a 8 o 12 bit (TRI032S emette silenziosamente BayerRG12 anche quando viene richiesto BayerRG8). Quindi un fotogramma a piena risoluzione con 4 telecamere è di**~12,6 MB se tutte a 8 bit, ma ~25 MB con tre telecamere mono a 12 bit**. La proiezione determina il formato di ciascuna telecamera in base al suo modello (cache di identità), quindi il burst corrisponde a ciò che il cavo trasporta effettivamente — non a un&#x27;ipotesi standardizzata di BayerRG8.
- **Un adattatore Ethernet USB ha un limite massimo di 200 MB/s indipendentemente dalle specifiche tecniche dichiarate.** La tabella di efficienza che converte la velocità di collegamento in un valore sostenuto deriva dallo standard PCIe; una scheda di rete USB dichiara la propria velocità di collegamento *Ethernet*, ma è limitata dal bus USB e dal relativo driver. Una chiavetta USB 10GbEha registrato circa 1063 MB/s “sostenuti” — un valore che non è mai stato verificato — e la regolazione della velocità risultante ha corrotto il 6–18 % dei frame pur continuando a riportare un fps target corretto. Le schede di rete collegate via USB sono ora limitate a **200 MB/s** in valore assoluto (il limite è il bus, quindi non è scalabile in base alle specifiche; un adattatore USB da 1 GbE raggiunge circa 80 MB/s e non ne risente). Il codice `wire_ceiling_source` nel record delle capacità lo indica esplicitamente, mentre `nic_is_usb` lo segnala. È possibile sovrascrivere entrambe le impostazioni con `--wire-ceiling-mbps`.
- **L’ammissione tiene conto del drain, non del confronto tra l’intero burst e l’anello.** Un burst simultaneo deve solo rientrare nel *backlog transitorio* = `max(0, Σ per-cam arrival − host drain) × emit_window`, non l’intero burst. Su un fabric con host veloce / cam lente (un host **PCIe**10G + 4 cam da 1 GbE: arrivo ≈ 320 MB/s, scarico ≈ 1063 MB/s) l’host scarica più velocemente di quanto le cam si riempiano, il backlog è ≈ 0, quindi l’emissione simulata a piena risoluzione**viene ammessa**anche se il burst da 25 MB supera l’anello da 13,5 MB. Mettendo le stesse quattro telecamere dietro un**adattatore USB**10GbE e lo scarico è di 200 MB/s, non 1063 — l’arrivo supera lo scarico, e la perdita si manifesta come fotogrammi danneggiati anziché come un frame rate inferiore. Su un host da 1 GbE, il limite minimo di DLThr delle telecamere pari a 31,25 MB/s fa sì che l’arrivo superi lo scarico → viene correttamente**blocca** (per *questa* classe di blocchi, ridurre il ROI o utilizzare un binning ≥ 2). L’ammissione è uno dei **due** criteri di connessione — l’altro è il controllo di sovrasottoscrizione aggregata riportato di seguito.
- **Gli fps previsti rappresentano un limite massimo prudenziale per il recupero seriale.**Il ciclo di acquisizione dell’host attualmente estrae il buffer di ciascuna telecamera**in modo seriale**(circa una finestra di emissione per telecamera ciascuna), quindi il ciclo è limitato da `max(readout+emit, N × emit)` con l’emissione per telecamera limitata al**collegamento di accesso**della telecamera (1 GbE ≈ 80 MB/s), non all’uplink dell’host. Per un array a 12 bit a pienaa 12 bit con**circa 2,8 fps**, in linea con i valori misurati di circa 2,7–3,0. Il valore in fps è volutamente**indipendente dall’esposizione**, quindi nelle scene poco illuminate il valore effettivo può scendere leggermente al di sotto del limite massimo man mano che l’esposizione si allunga. Il recupero seriale è il vero limitatore di fps; parallelizzarlo aumenterebbe il limite massimo verso la velocità di trasmissione singola.
- **L’oversubscription aggregata è un blocco di connessione inderogabile.**L’allocazione della larghezza di banda per telecamera ha un limite minimo di**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), quindi una volta raggiunto tale limite, la domanda aggregata (`per_cam × N`) può superare il**limite massimo di sicurezza contro le collisioni**(`sustained × sim_emit_factor`). Limiti massimi pratici a piena risoluzione su 1 GbE:**6 telecamere a 1500 MTU, 9 con jumbo**. Questo limite massimo dipende esclusivamente dalla linea e dal limite minimo — è**indipendente dalla dimensione del frame**, quindi**il binning e un ROI più piccolo NON aiutano** (riducono i byte per *frame*, non i byte al *secondo* regolati dal GevSCPD); gli unici rimedi sono un numero inferiore di telecamere, frame jumbo end-to-end o una scheda di rete più veloce. Il sintomo sarebbe la perdita di pacchetti GVSP, non una riduzione graduale degli fps, quindi `analyze-array` azzera i valori di fps raggiungibili e visualizza `**OVER-SUBSCRIBED**`, e `array-connect` con una risoluzione fissa **rifiuta la connessione** (il walk-down, altrimenti, raggruppa i frame in blocchi più piccoli, il che non risolve nemmeno questo tipo di blocco). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` ridimensiona il rifiuto a un forte avvertimento per il lavoro in laboratorio — vedi [Variabili d’ambiente](#environment-variables).

### Stato dell’array — quale sottosistema sta perdendo fotogrammi

L’`GET /api/camera/array/<array_id>/capability` di un array connesso contiene un blocco
`health` attivo, rivalutato su una finestra mobile di **10 secondi** . Suddivide la perdita di frame
nelle due cause che richiedono soluzioni opposte, anziché riportare un unico tasso “incompleto”
che non specifica nessuna delle due:

| Campo | Cosa significa | Quale sottosistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per numero di serie) | Il fotogramma **è arrivato ma era strutturalmente errato**— perdita di pacchetti GVSP. |**Rete**: budget di banda, pacing, anello RX della scheda di rete, MTU |
| `never_arrived_rate_pct` (per seriale) | Il frame **non è mai arrivato**— la telecamera non ha scattato, oppure non è stato trasmesso nulla. |**Trigger / sincronizzazione**: cavo M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | La frequenza peggiore di ciascuna telecamera. | — |
| `per_cam_rate_pct` | Tasso di incompletezza combinato per fotocamera (entrambe le cause insieme). | — |
| `stable_for_seconds` | Per quanto tempo ogni telecamera è rimasta al di sotto dello 0,01 %. | — |

Al di sopra del 5 %, il backend registra una riga `[array-health <id>] WARN` che indica la variazione — alla
prima violazione, in caso di variazione della fascia di gravità, una volta al minuto finché persiste e una volta quando
si risolve. La metà danneggiata stampa `[gvsp-corrupt <SN>]` al primo evento per telecamera e
motivo, poi un riepilogo ogni 60 s. Ogni valutazione viene comunque registrata nel file di log del backend;
i contatori avanzano su ogni buffer indipendentemente da ciò che viene stampato.

Lo stesso record riporta il numero a cui si aggancia l’intera allocazione :

| Campo | Significato |
| --- | --- |
| `wire_ceiling_mbps` | Il budget di banda effettivo dell’host, in MB/s. |
| `wire_ceiling_source` | Da dove proviene quel numero, in parole — ad es. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` o `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` quando `--wire-ceiling-mbps` (o il campo **Budget di banda** dell&#x27;interfaccia grafica) lo imposta. |
| `nic_is_usb` | `true` per un adattatore Ethernet USB — si veda il limite di 200 MB/s sopra indicato. |

**Interpretazione:** un valore di `gvsp_corrupt_rate_pct` diverso da zero con `never_arrived_rate_pct` a 0
indica che l’attivazione e la sincronizzazione del cavo sono perfette e che il 100% della perdita si verifica sul percorso di rete — ridurre il valore di `--wire-ceiling-mbps` e ricollegare. Il modello inverso indica invece un problema nel
cavo di sincronizzazione o nella linea di trigger.

> **`--target-fps` non è la leva per i frame corrotti.** Il pacing di GevSCPD viene scritto
> una sola volta alla connessione, quindi abbassare la frequenza di trigger modifica il ciclo di lavoro e non la
> velocità di trasmissione in burst simultanea. Una riduzione misurata della richiesta di 5× non ha prodotto alcun miglioramento;
> abbassare il limite massimo della linea da 240 a 200 MB/s ha portato lo stesso sistema dal 10,4 %
> allo 0,00%.

> **La riduzione automatica a metà flusso non è disponibile sul firmware TRI032S.** Un array in esecuzione
> non può risolvere autonomamente questo problema; disconnettersi e riconnettersi in modo che il selettore del tempo di connessione possa
> ripianificare con il nuovo limite massimo.

### Sintomo → soluzione

| Sintomo (Impostazioni array / connessione / `analyze_array_network`) | Causa | Soluzione |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` reimpostato a 32 (in genere dopo un aggiornamento del driver) | Impostare `ReceiveBufferLen`→256, `PendingReceives`→64; riaprire il pannello (riavviare il backend se ha memorizzato nella cache la vecchia dimensione dell&#x27;anello) |
| Il riavvio/spegnimento si blocca; in seguito si verificano errori WMI `Invalid class`, la scheda di rete non si abilita, le unità USB risultano mancanti | Vecchio driver Realtek USB 10GbE del 2016 → BSOD `0x9F` → spegnimento forzato-spegnimenti forzati | Aggiornare il driver della scheda a ≥ v10.67 (2026), quindi riapplicare le impostazioni del ring di ricezione sopra indicate |
| La connessione va a buon fine ma restituisce una risoluzione inferiore a quella nativa | Smart-prep riduce automaticamenteriduce automaticamente il frame per adattarlo alla larghezza di banda disponibile | Aggiornare il collegamento / accettare la riduzione / `--force-tier slip-emit-and-capture` |
| L’array riporta un valore di fps target corretto ma ne fornisce solo una frazione; `health.gvsp_corrupt_rate_pct` diverso da zero, `never_arrived_rate_pct` 0 | Il budget di banda del cavo dedotto dall’host sovrastima la capacità effettiva (tipico su un adattatore Ethernet USB, una corsia PCIe sottile o un fabric condiviso) | Riconnettersi con un valore `--wire-ceiling-mbps` più basso everificare nuovamente lo stato di integrità. **Non** `--target-fps` — Il pacing di GevSCPD è fissato al momento della connessione |
| Telecamere mancanti dai gruppi pubblicati; `health.never_arrived_rate_pct` diverso da zero, `gvsp_corrupt_rate_pct` 0 | Percorso di trigger/sincronizzazione — le telecamere non si attivano, non è un problema di rete | Controllare il cavo di sincronizzazione M8 e `--line`; verificare che ogni membro sia armato (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` superato in `analyze-array`, oppure rifiuto di connessione con risoluzione bloccata (`array over-subscribes the wire`) | La richiesta aggregata per telecamera (8 MB/s minimi × N telecamere) supera il limite massimo della linea a prova di collisione — 6 telecamere a piena risoluzione su 1 GbE @1500 MTU, 9 con jumbo | Meno telecamere, frame jumbo end-to-end o una scheda di rete più veloce. **Il ROI/binning NON sarà d’aiuto** (il limite massimo è indipendente dalla dimensione del frame). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` si sovrascrive sul banco di prova (accetta la perdita di pacchetti) |

---

## `chloros-cli daq`

Comandi del sensore spettrale. Due classi:
- **`pool-*`**— client “thin” HTTP che gestiscono il sensore tramite il pool persistente del backend.**Questo è il percorso supportato e l’unico presente nell’CLI fornito.** Il backend gestisce il trasporto, quindi l’interfaccia grafica, gli script CLI e SDK condividono tutti un unico handle attivo invece di contendersi la porta seriale.
- **Tutto il resto**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — accesso diretto all’hardware, documentato di seguito per completezza. Questi richiedono il pacchetto `daq` Python, che**non è incluso in nessun artefatto distribuito**: il file compilato CLI lo esclude (`scripts/Build-CLI.ps1` imposta `--nofollow-import-to=daq`, e i trasporti `pyserial` / `bleak` / `zeroconf` insieme ad esso), e nemmeno il pacchetto PyPI SDK lo contiene. Funzionano solo da un checkout del codice sorgente, quindi considerateli come un percorso di sviluppo interno a MAPIR piuttosto che come qualcosa a cui ricorrere.
- **`discover` / `list`** si collocano a metà strada tra i due: sono sono comandi hardware diretti provenienti da un checkout del codice sorgente, ma su una build distribuita ricorrono a `pool-discover` e il backend esegue la scansione. Pertanto la scansione funziona ovunque — il che è importante perché è l’unico modo per rilevare il MAC BLE di un DAQ-M.

> **`chloros-cli daq --help`** (e `-h` / `help`) elenca i sottocomandi `pool-*` — la guida è deliberatamente indirizzata al client del pool in modo da riflettere i comandi effettivamente eseguiti. Se si invoca un sottocomando direct-hardware su una build distribuita, il programma termina con un errore esplicito che indica il pacchetto mancante e rimanda a `pool-*`; nulla fallisce in modo silenzioso. (`discover` / `list` costituiscono l’eccezione: reindirizzano a `pool-discover` e funzionano semplicemente.)
>
> **Tutto ciò di cui un cliente ha bisogno è accessibile tramite `pool-*`** — connettersi, trasmettere in streaming, registrare file `.daq` calibrati e scambiare profili di condensatori. Il DAQ può anche essere controllato da Python con `chloros_sdk.connect_daq_sensor()`, che utilizza lo stesso percorso condiviso.

### Flusso di lavoro per la prima connessione del sensore DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Riferimento `pool-*`

| Sottocomando | Scopo |
| --- | --- |
| `daq pool-connect` (smart-detect) | Aprire un sensore nel pool di backend. |
| `daq pool-connect --port PORT` | DAQ-U su una porta seriale specifica. |
| `daq pool-connect --ble` | DAQ-M su BLE, MAC rilevato automaticamente. |
| `daq pool-connect --mac MAC` | DAQ-M su BLE con MAC noto (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E su Ethernet su un host noto. |
| `daq pool-connect --eth` | DAQ-E su Ethernet, host rilevato automaticamente (mDNS + fallback ARP; funziona con una cache ARP vuota su Windows e Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Ottimizzazione della finestra di integrazione / stato AE. |
| `daq pool-connect --no-stream` | Connettersi ma non avviare ancora lo streaming (riprendere con `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profilo di correzione del limite. Il valore predefinito sul backend è `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Esegue la scansione di ogni trasporto alla ricerca di sensori a cui è possibile connettersi, senza effettuare la connessione. **Ecco come trovare l’indirizzo MAC BLE di un DAQ-M.** `daq discover` / `daq list` vengono reindirizzati automaticamente qui nelle build distribuite. I sensori già aperti nel pool non vengono elencati — un DAQ-M connesso smette di trasmettere la propria pubblicità — quindi utilizza `pool-list` per quelli. |
| `daq pool-list` | Mostra tutti i sensori nel pool di backend. |
| `daq pool-disconnect --sensor-id ID [--all]` | Rilascia. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | I frame dello spettro N più recenti. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Riprendi / metti in pausa lo streaming. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Avvia / interrompi una registrazione .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Sostituisci il profilo di correzione cap durante l’esecuzione. |

### Sottocomandi hardware diretti (solo nel checkout del codice sorgente — non presenti nelle build distribuite)

> Elencati per completezza. Richiedono il pacchetto `daq` Python oltre a `pyserial` / `bleak` / `zeroconf`, nessuno dei quali è incluso nella versione compilata CLI o su PyPI SDK — funzionano solo da un checkout del codice sorgente MAPIR. **Se si utilizza una build rilasciata Chloros, utilizzare invece i comandi `pool-*` sopra indicati**; questi coprono le operazioni di connessione, streaming, registrazione e selezione dei cap.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Aprire, connettersi e gestire un progetto salvato su Chloros (una cartella contenente `cameras.json` + `sensors.json` + `project.json`). Tutto passa attraverso il backend, quindi l’interfaccia grafica e CLI producono lo stesso stato hardware.

### Riferimento ai sottocomandi

| Sottocomando | Scopo |
| --- | --- |
| `project open PATH` | Stampa il manifesto dei dispositivi del progetto (telecamere, array, sensori). |
| `project devices PATH [--reconnect]` | Elenca o riesegue la ricerca. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Connette ogni telecamera / array / sensore salvato. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Acquisizione singola da una telecamera o array specificati. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Scatto in raffica di N fotogrammi da una telecamera o da un array specificati (`-n/--count` valore predefinito 5; `-i/--interval` secondi tra un fotogramma e l’altro, valore predefinito 0). Le raffiche di array eliminano i gruppi sincronizzati ripetuti (watchdog di obsolescenza) in modo che un array a ciclo parziale non possa restituire N copie di un singolo fotogramma; stampa i risultati per ogni iterazione. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Trasferimento in streaming su disco tramite un processo di backend. `--poll-interval` = secondi tra i polling di `/stats` (valore predefinito 2,0). |
| `project sensor read PATH NAME [--json]` | Ultimo frame dello spettro. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Registra .daq. |
| `project run PATH RECIPE.yaml` | Esegue una ricetta di acquisizione YAML/JSON. `--dry-run` esegue la convalida senza eseguire l&#x27;operazione. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Calcola l&#x27;allineamento per un array — vedi [la tabella dei flag qui sotto](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Visualizza il profilo di allineamento corrente. |
| `project align clear PATH NAME` | Elimina il profilo memorizzato nella cache. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Sposta leggermente la trasformazione di uno slave. |
| `project align export PATH NAME --to FILE` | Salva il profilo in unJSONe. |
| `project align import PATH NAME --from FILE [--no-validate]` | Carica un profilo salvato. |

#### Opzioni `project align calibrate`

| Flag | Predefinito | Descrizione |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metodo di allineamento. **Queste grafie differiscono da `lattice align-calibrate`**, che accetta le forme abbreviate `orb` / `akaze` / `phase`; i due comandi non sono intercambiabili con questo flag. |
| `--model {translation, rigid, affine, homography}` | `affine` | Trasforma il modello per adattarlo. |
| `--frames N` | `1` | I fotogrammi sincronizzati vengono allineati alla media. |
| `--reference SN` | La | telecamera di riferimento principale; tutti gli altri membri vengono distorti su di essa. |
| `--max-features N` | `5000` | Limite del numero di caratteristiche ORB. |
| `--ratio-threshold F` | `0.75` | Test del rapporto di Lowe. |
| `--ransac-threshold-px F` | `3.0` | Soglia di inclusione soglia dei punti interni. |
| `--min-matches N` | `15` | **Controllo di qualità** — rifiuta la soluzione al di sotto di questo numero di corrispondenze inlier. |
| `--max-reproj-err-px F` | `4.0` | **Soglia di qualità** — rifiuta la soluzione al di sopra di questo errore di riproiezione RMS. |
| `--checkerboard RxC` | — | Geometria della scheda per `--method checkerboard`, ad es. `9x6`. |
| `--name PROFILE` | vuoto | Nome del profilo incorporato nel file JSON salvato. **Non è il nome dell&#x27;array** — ovvero l’`NAME` posizionale. |

I due controlli di qualità sono il motivo per cui una calibrazione può riuscire nella risoluzione e tuttavia
rifiutarsi di salvare: un profilo che non superi uno dei due controlli causerebbe silenziosamente un errato allineamento di ogni
acquisizione successiva, pertanto viene rifiutato anziché salvato.

### Esempi

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL della ricetta

`project run RECIPE.yaml` accetta un file YAML o JSON che descrive una sequenza di azioni:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Azioni supportate: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. L’azione `burst` richiede `name` (obbligatorio), `count` (impostazione predefinita 5), `interval` (in secondi, valore predefinito 0), `output`, `format` e `settings` (stessa configurazione delle impostazioni per singola telecamera di `apply`); i burst array utilizzano lo stesso watchdog &quot;fresh-synced-group&quot; di `project burst`.

Eseguirlo:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Variabili d’ambiente

| Variabile | Effetto |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Sovrascrive l’URL del backend (impostazione predefinita `http://127.0.0.1:5000`) — **rispettata solo dalle famiglie di comandi `lattice`, `project`, e `daq pool-*`.** I comandi principali (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) indirizzano `http://127.0.0.1:<port>` e ignorano questa variabile (ilv4 letterale aggira l’Windows `localhost`→`::1` penalità di circa 2 s per richiesta), quindi puntano sempre alla macchina locale. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` ridimensiona il rifiuto di connessione per sovrasottoscrizione dell&#x27;array (aggregato per-cam &gt; limite di sicurezza anti-collisione della linea con `pin_resolution`) a un &quot;avviso forte e prosegui&quot;, accettando la perdita di pacchetti GVSP. Solo per uso di benchmark — vedi [Modello fps e burst dell&#x27;array](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Impostato dall’CLIe stesso; indica al backend di abilitare l’elaborazione parallela. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` salta la sonda di fallback GVSP (solo risultati ICMP). **Questo disattiva i pacchetti jumbo, non si limita a ridurre il volume del log** — la telecamera risponde ai ping DF solo fino a 1500 su ogni percorso, quindi questa sonda è l’unica in grado di rilevare i pacchetti jumbo. Risparmia circa 1 s per telecamera per connessione; costa circa 1,45 volte il limite massimo della linea se la rete *potesse* trasportare pacchetti jumbo. L’SDKa un avviso quando si imposta questa opzione. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fissa la dimensione dei pacchetti GVSP a N byte; salta completamente il test. È preferibile utilizzare l’impostazione per singolo comando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) piuttosto che impostarla in modo permanente: una dimensione fissa impedisce l’adattamento alla rete a monte, e fissare 9000 su un percorso che non supporta i pacchetti jumbo fa sì che **ogni** acquisizione vada in timeout con `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Sovrascrive la directory di estrazione &quot;onefile&quot; di Nuitka. L’opzione CLI utilizza automaticamente `/mnt/ssd/tmp` se presente. |

---

## Codici di uscita

| Codice | Significato |
| --- | --- |
| `0` | Operazione riuscita. |
| `1` | Errore generico (la maggior parte degli errori dei sottocomandi). |
| `2` | Errore di argomento. |
| `130` | Interrotto da Ctrl+C. |

---

## Suggerimenti per la risoluzione dei problemi

- **&quot;Accesso richiesto&quot;** → Eseguire una volta `chloros-cli login EMAIL PASSWORD` su questo computer.
- **&quot;backend irraggiungibile&quot;** → Avviare l’applicazione desktop Chloros, oppure eseguire direttamente il binario del backend (`chloros-backend`), oppure verificare `CHLOROS_BACKEND_URL` se si tratta di un accesso remoto.
- **I comandi `lattice` falliscono con il messaggio &quot;Driver della telecamera LATTICE non trovati&quot;** → Il runtime Arena SDK non è installato; l’CLIa viene fornita con `win32api` incluso su Windows, ma il runtime C fa parte del programma di installazione della GUI.
- **&quot;Connetti array&quot; / &quot;Impostazioni array&quot; mostra &quot;FRAMES WILL DROP&quot; o &quot;Riduci ROI per abilitare&quot;** → L’anello di ricezione della scheda di rete dell’host è troppo piccolo (di solito viene reimpostato a 32 dopo un aggiornamento del driver della scheda di rete). Vedere [Configurazione e ottimizzazione della scheda di rete host](#host-nic-setup--tuning-lattice-arrays) — impostare `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Il sistema si blocca al riavvio/spegnimento, quindi WMI `Invalid class` / la scheda di rete non si abilita / le unità USB non vengono rilevate** → Driver obsoleto dell’adattatore USB 10GbE che causa l’errore `DRIVER_POWER_STATE_FAILURE` (schermata blu `0x9F`). Aggiornare il driver dell’adattatore — vedere [Configurazione e ottimizzazione della scheda di rete host](#host-nic-setup--tuning-lattice-arrays).
- **Avviso relativo allo swap su Jetson** → Aggiungere uno swap basato su file; il comando `CLI` riporta i comandi esatti `fallocate` / `swapon`.
- **Comandi diretti DAQ mancanti** → Come previsto: il pacchetto `chloros-cli` fornito esclude deliberatamente il pacchetto `daq`, quindi è presente solo `pool-*` (nemmeno l’SDKe PyPI lo include). Utilizzare `pool-*`, che controlla lo stesso sensore tramite il backend, oppure `chloros_sdk.connect_daq_sensor()` da Python.

---

## Vedi anche

- [Python SDK Riferimento](sdk-reference.md) — equivalente programmatico di ogni comando CLI.
- [Guida ai sensori DAQ](../daq/README.md) — cablaggio e calibrazione specifici per sensore.
- Documentazione online: `https://mapir.gitbook.io/chloros/cli`
