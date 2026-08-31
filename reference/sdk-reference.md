# Chloros Python SDK Riferimento

**Versione:**

1.2.0**Generato:**29/07/2026 19:19 ·**Aggiornato:** 30/08/2026**Pacchetto:** `chloros-sdk` (PyPI)**Destinatari:** Ottimizzato per l&#x27;utilizzo da parte di modelli di linguaggio di grandi dimensioni (LLM); leggibile dall&#x27;uomo.**Ambito:** Tutte le classi pubbliche, le funzioni e gli helper esposti da `import chloros_sdk`, con esempi copiabili e incollabili che coprono l’elaborazione delle immagini, il controllo di una singola telecamera, gli array sincronizzati, i sensori DAQ e l’automazione dei progetti.

Se ti servono solo i punti salienti, vai a:
- [Installazione e guida rapida](#installazione)
- [Smart-Connect per array LATTICE](#smart-connect-for-lattice-cameras)
- [Sessioni sui sensori DAQ](#daq-sensor-sessions)
- [Automazione del progetto](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Architettura in 60 secondi

L’SDK è un sottile strato diPythone che si sovrappone al backend Chloros (lo stesso server Flask utilizzato dall’interfaccia grafica desktop e da CLI). Per l’automazione, si importa `chloros_sdk` e si chiamano metodi di alto livello; dietro le quinte, ogni chiamata si trasforma in una richiesta HTTP al backend locale sulla porta 5000 — `http://127.0.0.1:5000/api/...` (volutamente non `localhost`, che viene risolto prima come `::1` su Windows e costa circa2 s per richiesta su un backend solo IPv4). Il backend possiede il pool hardware — telecamere, sensori DAQ, profili di allineamento, frame buffer — quindi gli script SDK possono coesistere con la GUI senza competere per le porte seriali o la larghezza di banda della scheda di rete.

Ci sono tre interfacce che userete:

1. **`ChlorosLocal` + funzioni libere** (`process_folder`, `process_lattice_capture`) — Pipeline di elaborazione delle immagini. Elabora un&#x27;intera cartella attraverso la calibrazione / debayer / esportazione degli indici con un’unica chiamata a Python.
2. **Handle Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Apertura di una sessione backend persistente per l’hardware in tempo reale. Stesso flusso “smart-prep” dell’interfaccia grafica: sonda di rete, selezione automatica del livello, PTP, seeding AE, configurazione del trigger GPIO.
3. **`ChlorosProject` / `open_project`** — Carica un progetto salvato (cartella contenente `cameras.json` + `sensors.json` + `project.json`), connette tutto in una volta sola ed esegue le acquisizioni tramite handle denominati.

Le interfacce 1 e 2 **avviano automaticamente un backend locale** se non ce n’è già uno in ascolto (lo stesso binario in dotazione che la GUI/CLI ) — quindi uno script semplice funziona da una shell nuova senza che sia necessario avviare prima un backend. Passare `auto_start_backend=False` per disattivare questa funzione (ad esempio quando si punta a un backend remoto, che non viene mai avviato). Vedi [Avvio automatico del backend](#backend-auto-start). Surface 3 si comporta in modo diverso: `open_project()` non accetta alcun `auto_start_backend` parametro, e `connect_all()` non avvia mai un backend — effettua una sola prova su `http://127.0.0.1:5000` una volta e, se non riceve risposta, ricorre silenziosamente al controllo diretto (senza backend-free) controllo del dispositivo `lattice_sdk`. Solo `proj.process()` e `stream(..., overlays=True)` creano in modo differito un `ChlorosLocal()` (che esegue l’avvio automatico).

Tutti e tre sono soggetti a controllo di autenticazione: eseguire `chloros-cli login` una volta sulla macchina oppure effettuare l’accesso tramite l’interfaccia grafica desktop. Le chiamate a SDK senza una sessione valida generano l’errore `ChlorosAuthenticationError`.

Requisiti:
- Python 3.7+ (come dichiarato dal pacchetto; sviluppato/testato su 3.10)
- Chloros Desktop installato localmente (il binario del backend è incluso nel programma di installazione)
- Accesso attivo a Chloros+. Il livello minimo per SDK / CLI è **Copper**o superiore (Copper / Bronze / Silver / Gold); il livello gratuito**Iron**non consente l’accesso a SDK / CLI. Questa restrizione viene applicata**lato server**: ogni richiesta contrassegnata con SDK / CLI deve includere sia una sessione attiva che un piano a pagamento, altrimenti il backend restituisce `403` con `error_code: PLAN_UPGRADE_REQUIRED` (visualizzato come `ChlorosLicenseError` da `ChlorosLocal` e come `ChlorosConnectError` dagli helper `connect_*`). Un chiamante disconnesso riceve invece `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — i due sono distinti perché l’esecuzione di `chloros-cli login` risolve il primo problema ma non il secondo.
- L’utilizzo offline è supportato entro il periodo di tolleranza del piano: il livello viene letto dalla cache di convalida del server (5 min) o dalla cache delle licenze firmate e legate al dispositivo (30 giorni per i piani mensili, fino alla scadenza dell’abbonamento per quelli annuali). Allo scadere di tale periodo di tolleranza, il piano passa alla versione gratuita e l’accesso a SDK / CLI viene interrotto fino a quando il computer non riesce a connettersi al server almeno una volta. `chloros-cli status` (`GET /api/license-status`) rimane raggiungibile nel livello gratuito, in modo che il motivo sia visibile: è l’unica rotta SDK / CLI esente dal limiti del piano tariffario.
- Windows 10/11 a 64 bit, **Ubuntu 22.04 LTS o versioni successive**, oppure Jetson (JetPack 6). Ubuntu 20.04**non** è supportato: le dipendenze di `.deb` derivano da ciò a cui il backend si collega, incluso `libc6 (>= 2.34)`, e Focal distribuisce glibc 2.31.

---

## Installazione

L&#x27;Python SDK è un sottile livello Python che si sovrappone al backend Chloros. Per qualsiasi cosa che vada oltre alcuni flussi di lavoro limitati alla sola acquisizione dati (DAQ), è necessario **installare localmente il pacchetto desktop Chloros** (programma di installazione Windows o Linux `.deb`) — che fornisce il binario del backend, il runtime Arena SDK per le telecamere LATTICE e i pacchetti di calibrazione.

Ultimi download: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Passaggio 1 — Installare il pacchetto della piattaforma Chloros

#### Windows (.exe)

1. Scaricare `Chloros-Setup-x.y.z.exe` dalla pagina di download.
2. Eseguire il programma di installazione e seguire la procedura guidata. Il percorso di installazione predefinito è `C:\Program Files\MAPIR\Chloros\`.
3. Avviare Chloros almeno una volta ed effettuare l’accesso con il proprio account Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Passaggio 2 — Installa l’Python SDK

**Il programma di installazione Chloros include un pacchetto wheel SDK corrispondente.** Ogni programma di installazione Windows e ogni file .deb Linux installa sul disco un file `chloros_sdk-X.Y.Z-py3-none-any.whl` che corrisponde esattamente alla versione della GUI / CLI / backend. Non è necessario cercare su PyPI per mantenere la sincronizzazione.

#### Windows

Il programma di installazione esegue automaticamente `pip install` sul wheel in dotazione utilizzando l&#x27;Python del sistema (si preferisce il launcher `py.exe`, ma si ricorre a `python -m pip` in caso di errore). Non è richiesta alcuna azione: `import chloros_sdk` funziona nel vostro ambiente Python dopo un’installazione riuscita. Se sul sistema non è presente Python, il programma di installazione salta silenziosamentee l’interfaccia grafica e CLI continuano a funzionare.

#### Linux (.deb)

Il pacchetto .deb installa il wheel in `/usr/lib/chloros/sdk/`. Il comando `postinst` stampa il comando esatto — Le distribuzioni PEP 668 rifiutano per impostazione predefinita le scritture globali di pip, quindi non eseguiamo l’installazione automatica:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Per le implementazioni Jetson in modalità air-gapped, il processo avviene completamente offline — il file wheel è già presente sul disco.

#### PyPI pubblico

Per gli host che utilizzano esclusivamente pip (senza pacchetto desktop Chloros installato; flussi di lavoro basati esclusivamente su backend remoto o DAQ):

```bash
pip install chloros-sdk
```

PyPI viene aggiornato nelle build dell’installer relative alla versione di rilascio, quindi il wheel pubblicato corrisponde all’ultima versione stabile. Le build di sviluppo (ad es. `1.1.4.dev1`) vengono distribuite solo tramite il wheel dell’installer in dotazione.

#### Verifica

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **È richiesto un abbonamento a Chloros+.** Tutte le chiamate a SDK richiedono un accesso attivo a Chloros+. Eseguire `chloros-cli login user@example.com 'YourPassword'` una volta per ogni macchina; le credenziali vengono memorizzate nella cache in `~/.chloros/`.

### È necessario il pacchetto Desktop?

Il pacchetto pip da solo **non** è sufficiente per la maggior parte dei flussi di lavoro. Ecco cosa serve per ogni superficie SDK:

| Superficie SDK | È necessario il pacchetto Desktop? | Perché |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Sì** | Avvia automaticamente il binario del backend su `/usr/lib/chloros/chloros-backend` (Linux) o `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Sì**(locale)**/ No**(remoto) | Client Pure HTTP sul backend. Backend locale → è richiesto il pacchetto desktop. Backend remoto → `backend_url=`**tramite un tunnel** (vedi Modalità backend remoto — i backend forniti si legano solo al loopback). |
| `ChlorosProject` / `open_project` | **Sì** | Gestisce i progetti salvati tramite il backend. |
| Classi LATTICE dirette (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Sì** | Richiedono il runtime nativo Arena SDK incluso nel pacchetto desktop. `CAMERA_AVAILABLE` corrisponde altrimenti a `False` al momento dell’importazione. |
| Classi DAQ dirette (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **No** | Puro Python su pyserial/bleak/zeroconf. Un ambiente basato esclusivamente su pip è in grado di gestire i DAQ end-to-end. |

### Modalità backend remoto (host pip-only, tramite tunnel)

> **Il backend fornito non è raggiungibile tramite LAN.** Le build di produzione
> si legano solo al loopback (entrambe le famiglie di loopback) e rifiutano categoricamente l’
> unica modalità non loopback (`CHLOROS_CLOUD_MODE`), quindi
> `backend_url="http://<lan-ip>:5000"` **non può funzionare con un
> Chloros installato** — tale schema ha sempre funzionato solo con un backend source/dev
> . Per pilotare un backend su un’altra macchina, inoltra tu stesso la sua porta loopback
> e punta l’SDKe verso il tunnel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Gli host headless / CI / robotica possono mantenere una macchina con l’installazione desktop completa come “server Chloros” e `pip install chloros-sdk` ovunque altrove — ma il trasporto tra di essi è il tunnel configurato dall’utente sopra indicato, non un’URL LAN diretta.

> **Limitazione nota — `ChlorosLocal` non supporta esclusivamente pip.** `ChlorosLocal(backend_url=BACKEND)` attualmente risolve un binario backend locale nel proprio costruttore *prima* di sondare l’URL e genera l’errore `ChlorosBackendError` («Backend Chloros non trovato…») quando non è installato alcun pacchetto desktop — anche in presenza di un backend remoto raggiungibile. Solo l’interfaccia smart-connect sopra indicata (`connect_camera` / `connect_array` / `connect_daq_sensor`, oltre a `analyze_array_network` e agli helper `list_*` / `discover_*`) funzionano da un host solo pip.

### Flusso di lavoro solo DAQ (host solo pip)

Se avete bisogno solo dei sensori DAQ e non utilizzate le telecamere LATTICE né l’elaborazione delle immagini, il pacchetto pip è autonomo:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Nessun backend, nessun .deb, nessun accesso tramite Chloros+ richiesto per il lavoro DAQ diretto sull&#x27;hardware.

---

## Guida rapida

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Indice di primo livello dell&#x27;API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Elaborazione delle immagini — `ChlorosLocal`

La classe principale della pipeline. Avvia il backend al primo utilizzo, crea e configura i progetti, monitora lo stato di avanzamento e restituisce i riepiloghi al termine dell’esecuzione.

### Costruttore

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metodi

| Metodo | Descrizione |
| --- | --- |
| `create_project(project_name, camera=None)` | Crea un nuovo progetto (facoltativamente con un modello di fotocamera come `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importa immagini RAW/TIF/JPG/DNG **e registrazioni del sensore di luce `.daq`**. Restituisce `count` (immagini) e `scan_count` (registrazioni). Avvisa solo se la cartella non contiene né le une né le altre. |
| `export_light_sensor(daq=True, csv=True)` | Scrive i file calibrati `.daq` + `.csv` per ognidel progetto in `<project>/Light Sensor/`. Vedi [Registrazioni dei sensori di luce](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Imposta i parametri di elaborazione. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Eseguire la pipeline. Restituisce `{"status": "complete", "async": False}`, oltre a una chiave `summary` quando il backend ne fornisce una — vedi [Riepilogo post-esecuzione e suggerimenti](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Verifica lo stato del backend. |
| `logout()` | Cancella le credenziali memorizzate nella cache. |
| `shutdown_backend()` | Termina il backend (se avviato con `SDK -start`). |
| `discover_cameras()` | Individua le telecamere LATTICE **tramite il backend di questa istanza** (`/api/camera/discover`). Restituisce un elenco di dizionari (`serial`, `model`, `ip`, …) — con la stessa struttura visibile nell’ GUI/ CLI. Elenco vuoto se non ne viene trovata nessuna o se il backend è irraggiungibile. |
| `camera_capture(output_dir, format="tiff", **settings)` | Acquisisce un singolo fotogramma**tramite il backend**(avviato automaticamente da questo handle) in modo che riceva la stessa preparazione del GUI/ CLI (impostazione predefinita a 12bit per impostazione predefinita, riutilizzo del pool, metadati di calibrazione incorporati). Risolvere il target con `serial=` o `device_index=`; passare `exposure`/`gain`/`pixel_format`/`preset` come `**settings`. Restituisce il dizionario dei metadati legacy (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Genera fotogrammi di anteprima con sovrapposizione composita da una telecamera in pool — client MJPEG leggero tramite il percorso `/api/camera/<serial>/stream-annotated` del backend (zebra / griglia / mirino / istogramma / peaking / punto disegnati lato server). `decode=True` restituisce array BGR; `False` restituisce byte grezzi JPEG. Raggiungibile anche a livello di progetto come `ChlorosProject.stream(overlays=True)`. |

Utilizzare come gestore di contesto per garantire la pulizia:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Registrazioni dei sensori di luce — calibrati `.daq` + `.csv`

È possibile registrare un DAQ-U / DAQ-M / DAQ-E **senza** il relativo pacchetto di calibrazione. Questo è
ciò che fanno per impostazione predefinita i registratori pubblici [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): registrano i conteggi grezzi dei sensori e contrassegnano il
file in modo che Chloros recuperi la calibrazione di fabbrica di quel sensore **in base al numero di serie** — prima dalla cache locale,
poi dal cloud MAPIR — e la applichi al momento dell’importazione.

Chloros scrive il risultato come due prodotti per ogni registrazione, sotto
`<project>/Light Sensor/`:

| Prodotto | Che cos’è |
| --- | --- |
| `<name>_calibrated.daq` | L’archivio rielaborabile — stesso schema di una registrazione in tempo reale, ora con l’indicazione del pacchetto che l’ha prodotto. Reimportarlo **non** comporta una seconda calibrazione. |
| `<name>_calibrated.csv` | Irradianza spettrale in W/m²/nm sulla griglia di lunghezze d’onda del sensore stesso, una riga per lettura, più colonne fotometriche (potenza totale, lux fotopico/scotopico, PPFD e la sua suddivisione in blu/verde/rosso, lunghezza d’onda di picco). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Solo sensori senza bundle (DAQ-A).** Conteggi spettrali grezzi del sensore — *non* l’irradianza. Vedi sotto. |

`process()` esegue questa esportazione come una delle sue fasi. **Non** richiede immagini:
un sensore di luce utilizzato da solo costituisce un flusso di lavoro a tutti gli effetti, e un progetto di questo tipo non presenta
per definizione.

**Le registrazioni DAQ-A vengono esportate come conteggi grezzi.** La famiglia DAQ-A è antecedente al sistema di
bundle per numero di serie e non ha alcun bundle da recuperare — viene invece calibrata sul campo rispetto a un
target di riflettanza, motivo per cui non ne ha mai avuto bisogno. Tali registrazioni vengono esportate
con un prefisso `_raw` anziché `_calibrated`: un nome file diverso anziché un flag
all’interno del file, poiché l’identificativo deve sopravvivere all’invio via e-mail come semplice nome. L’
intestazione `.csv` riporta `raw spectral sensor counts (NOT irradiance)` e avverte che i
valori siano comparabili **all’interno** del file — esattamente lo scopo per cui la calibrazione basata sul bersaglio
li utilizza — e non tra sensori diversi. Le colonne fotometriche dipendenti dalla potenza (potenza totale,
lux fotopico/scotopico, PPFD) restituiscono **NULL** anziché essere integrate dai conteggi.

Un DAQ-U / DAQ-M / DAQ-E il cui bundle semplicemente non è stato possibile recuperare viene comunque **saltato**,
non scritto in formato grezzo: in quel caso il bundle esiste e &quot;riconnettere e rielaborare&quot; è un consiglio valido.

Le registrazioni legacy **v1.01 / v1.02** (un DAQ-A-SD le scrive) non riportano alcuna epoca per lettura,
ma solo l’ora di scrittura del file. Il sistema di corrispondenza immagine↔flusso discendente continua a rifiutarle — l’abbinamento di un
frame con un’ora di scrittura comporterebbe un errore non visibile — ma l’esportatore le legge e il
file CSV stampa `clock=daq_created_on`, in modo che il prodotto indichi su quale orologio si trova.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Una registrazione il cui pacchetto di calibrazione non può essere recuperato (offline o sensore senza
calibrazione in archivio) viene segnalata con il codice `skipped` **indicandone il motivo**. Non viene mai
scritta come file “calibrato” contenente conteggi grezzi: connettersi a Internet e
eseguire nuovamente l’operazione: l’esportazione verrà completata.

### Callback di avanzamento

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Riepilogo post-esecuzione e suggerimenti

Al termine, `process()` recupera `GET /api/processing-summary` e allega il corpo come `result["summary"]`. Il recupero avviene secondo il principio del «best effort» e non blocca mai un ritorno positivo — se il riepilogo non è disponibile, `process()` ricade nella forma semplice `{"status": "complete", "async": False}`. Ogni voce in `summary["hints"]` — frasi complete con la correzione suggerita, ad es. perché un&#x27;esecuzione ha prodotto un output pari a zero — viene inoltre riemessa come `UserWarning` Python, quindi le esecuzioni con output pari a zero sono autodiagnostiche anche se non si ispeziona mai il dizionario:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` è la parte leggibile dalla macchina:

| Chiave | Cosa conta |
| --- | --- |
| `models` | Gruppi di telecamere nell’esecuzione. |
| `images_in_groups` | Immagini sorgente in tali gruppi. |
| `targets_found` | Target di riflettanza rilevati. |
| `images_calibrated` | Immagini calibrate dall’esecuzione. |
| `exported_files` | **File dei prodotti immagine generati dall’esecuzione.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Registrazioni del sensore di luce, conteggiate separatamente di proposito — provengono da una fase diversa ed esistono anche per sessioni prive di immagini, quindi includerle farebbe sembrare che una sessione solo DAQ abbia esportato immagini. |

Accanto a questi: `summary["output_dirs"]` (ogni directory in cui è stato scritto),
`summary["light_sensor_export"]`, `summary["stopped"]` (vero quando l’utente ha interrotto l’
esecuzione, in modo che i conteggi parziali non vengano interpretati come un’esecuzione completata con produzione insufficiente) e
`summary["groups"]` (la suddivisione per gruppo).

`exported_files` viene registrato dalla pipeline **mentre scrive**, non viene estratto dagli
oggetti immagine del progetto in un secondo momento. Le strategie parallele e GPU creano i propri oggetti immagine
(nei sottoprocessi dei worker per i percorsi GPU), quindi la vecchia scansione riportava
`0 file(s) written` per ogni esecuzione di questo tipo ed emetteva poi l’indicazione “zero esportazioni” — su esecuzioni
in cui tutto aveva funzionato. Se si crea uno script basato su questo numero, un’esecuzione parallela corretta ora
riporta un conteggio diverso da zero.

I salti del sensore di luce riportano il motivo effettivamente rilevato dal lettore per ciascun file — uno
schema illeggibile, un bundle mancante, un errore di scrittura — **deduplicati**, quindi venti file
saltati per un’unica causa vengono interpretati come un’unica causa anziché come venti ripetizioni della stessa.

> **`process()` non viene generato quando un’esecuzione non produce immagini.** Questo è l’unico punto in cui l’SDK e
> l’CLI differiscono deliberatamente: `chloros-cli process` considera &quot;i prodotti sono stati richiesti, ma nessuno è stato
> scritto&quot; come un errore e termina con un codice non nullo, mentre l&#x27;SDK termina normalmente e segnala la
> condizione tramite `summary` / hints. Se la vostra pipeline dovesse interrompersi in caso di esecuzione vuota, controllalo
> personalmente: ispeziona `summary` (o conta i file nella cartella del progetto) anziché fare affidamento sull’
> assenza di un’eccezione. Le cause più comuni sono una cartella di input non riconosciuta come
> acquisizione e prodotti saltati in quanto non applicabili alle telecamere presenti (ad es. radiance da RGB -solo
> telecamere).

### Funzioni di utilità

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Valori supportati

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Output radiometrico (pipeline multispettrale LATTICE)

Il livello di esportazione multispettrale LATTICE della pipeline `process` (M3C/M3M) della pipeline LATTICE — `reflectance` (impostazione predefinita), `radiance`, `sensor-response` o `all` (ogni modalità applicabile per immagine) — corrisponde all’impostazione di elaborazione **&quot;Output radiometrico&quot;** del progetto. `configure()` dispone di una parola chiave dedicata:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

La soluzione alternativa avanzata — scrivere la chiave `"Radiometric output"` del progetto tramite `custom_settings` — funziona ancora, ma ricordate che sostituisce l’intero blocco di impostazioni (vedere l’avviso qui sotto):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (il valore predefinito) divide la radianza della telecamera per la **radiometria DAQ in direzione discendente abbinata al timestamp**, risolta automaticamente da un `.daq` registrato (DAQ-U/M/E)**o un `.csv` nativo DAQ-M**presente insieme alle immagini; eventuali pacchetti di calibrazione per singola telecamera o per DAQ mancanti localmente vengono**recuperati automaticamente da AWS** al primo utilizzo. L’CLIe espone questa funzionalità come opzioni di attivazione/disattivazione per tipo di prodotto su `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **sostituisce** l’intero blocco delle impostazioni calcolate (per come è progettato, ignora le altre parole chiave e la convalida di `configure()`). Quando lo si utilizza, includere ogni chiave `Project Settings` di interesse, come nell’esempio sopra riportato.

---

## Smart-Connect per le telecamere LATTICE

Sessioni di backend persistenti per l’hardware in tempo reale. Vengono utilizzati gli stessi endpoint della GUI, pertanto il comportamento è identico su SDK / CLI / GUI.

### Singola telecamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Firma `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` Metodi

| Metodo | Descrizione |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Leggi i nodi GenICam; restituisce `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Scrive i nodi in base al nome descrittivo (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Acquisisce un **singolo** fotogramma. Restituisce una lista di un elemento contenente dizionari di metadati del fotogramma. (L’acquisizione in raffica/multi-fotogramma è stata rimossa — chiamare `capture()` in un ciclo se è necessaria una serie.) |
| `disconnect()` | Rilascia dal pool. Operazione nulla se ci siamo collegati a una sessione già aperta. |

Controlli di esportazione `capture()` (stesso modello dell’array + GUI):

- `processing` / `levels` — `processing="all"` salva tutti i tipi di esportazione applicabili; `levels=["raw","radiance"]` salva solo quelli (sovrascrive `processing`). Omettete entrambi per utilizzare l’impostazione predefinita del backend.
- `force_daq=True` — salva la lettura DAQ/DLS assegnata come sidecar `.daq` anche in caso di acquisizione solo raw, in modo che il fotogramma possa essere rielaborato in riflettanza/indice in un secondo momento. Nessuna operazione se non è collegato alcun DAQ.

### Array sincronizzato — `ArraySession` (Smart-Prep)

`connect_array` è **il punto di ingresso consigliato** per le configurazioni multicamera. Esegue in background l’intero flusso Smart-Prep della GUI:

1. **Analisi di rete** (`/api/camera/array/recommend`) — individua la dimensione massima del frame che rientra nel livello di emissione simulata senza perdere frame.
2. **Selezione automatica del livello** — `sim-capture-sim-emit` se il cavo è in grado di gestirlo; altrimenti `sim-capture-ftd-stagger` o `slip-emit-and-capture`.
3. **Ridimensionamento automatico**— riduce silenziosamente la dimensione dei frame / aumenta il binning quando la linea non è in grado di sostenere la risoluzione richiesta.**Questa misura di sicurezza non copre l’oversubscription aggregata**: un numero eccessivo di telecamere per la linea non può essere risolto riducendo i fotogrammi — vedi [Sovrasottoscrizione](#over-subscription-the-per-cam-floor).
4. **PTP abilitato**per impostazione predefinita — i timestamp tra le telecamere vengono sincronizzati su un unico orologio condiviso con una precisione di**~1 ms**. L’esposizione simultanea deriva dal trigger hardware M8 (**&lt; 100 µs** tra i moduli), non dal PTP: il PTP allinea i *timestamp*, non le esposizioni.
5. **Selezione automatica del formato pixel per ogni telecamera** — telecamere RGB → `BayerRG8`, multispettrali → `BayerRG12`.
6. **Inizializzazione AE** — acquisisce lo stato AE corrente di ciascuna telecamera in modo che la connessione non reimposti l’esposizione durante il funzionamento.
7. **Configurazione trigger GPIO** — `connect_array` attiva tutte le telecamere (`TriggerMode=On`, `TriggerSource=Line2`) in modo che l’impulso del master comandi gli slave tramite il cavo M8. Questo è un passaggio valido solo per gli array: una singola telecamera aperta con `LatticeCamera` funziona invece in modalità libera.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### Firma `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Valori `force_tier`:
- `"sim-capture-sim-emit"` — vera simultaneità (tutte le telecamere scattano sullo stesso fronte di clock).
- `"sim-capture-ftd-stagger"` — sfalsamento flessibile nel dominio del tempo (le telecamere emettono in momenti leggermente sfalsati in modo che i pacchetti si serializzino sulla linea).
- `"slip-emit-and-capture"` — acquisizione sequenziale per singola cam (nessuna sincronizzazione temporale; unica opzione quando nessuna dimensione di frame è compatibile con la modalità simultanea).

`wire_ceiling_mbps` sovrascrive il **budget di banda sostenuto dall&#x27;host** in MB/s — l’unico
valore da cui dipende l’intera allocazione dell’array. Lasciarlo su `None` per utilizzare il valore rilevato automaticamente
. Ridurlo quando l’array segnala frame danneggiati da GVSP: il valore automatico è derivato
dalla velocità di collegamento dichiarata dalla scheda di rete, che sovrastima gli adattatori USB, le linee PCIe sottodimensionate e i
fabric condivisi molto trafficati — e tale sovrastima si manifesta come frame danneggiati anziché come un
collegamento visibilmente lento. Il valore viene memorizzato in modo permanente nel blocco di acquisizione dell’array del progetto, quindi una
la riapertura o un successivo `connect_array` lo ripristini come qualsiasi altra impostazione dell’array.
Vedi [Stato di salute dell’array](#array-health--which-subsystem-is-losing-frames).

#### Sovrasottoscrizione (il limite minimo per telecamera)

Il pacing Sim-emit assegna a ciascuna telecamera una quota del budget di banda a prova di collisione, con un limite minimo di **8 MB/s per telecamera**(`per_cam_floor_bps`). Una volta che `N × floor` supera il limite massimo di sicurezza contro le collisioni, l’array**sovrascrive la banda**— la modalità di errore è la perdita di pacchetti GVSP, non una frequenza dei fotogrammi inferiore — e non esiste alcuna soluzione per recuperare i fotogrammi:**il binning e la ROI riducono il numero di byte per fotogramma, non i byte regolati al secondo**che il controllo aggregato mette a confronto. Limiti pratici alla piena risoluzione su un host da 1 GbE:**6 telecamere a 1500 MTU, 9 con frame jumbo** (`max_cams_collision_safe` nella risposta all’analisi riporta il limite massimo per la vostra linea). Soluzioni: meno telecamere, frame jumbo end-to-end o una scheda di rete più veloce.

- Le risposte `analyze_array_network()` e `/api/camera/array/connect` contengono `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` e `per_cam_floor_bps`. Quando `oversubscribed` è vero, la proiezione **azzeri i campi fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) anziché segnalare una velocità fuorviante, bassa ma funzionante.
- `POST /api/camera/array/connect` accetta un parametro del corpo `pin_resolution` (**solo HTTP — non è un kwarg dell&#x27;SDK**; `connect_array` nonnon lo espone). Il pinning rimuove la rete di sicurezza del walk-down del binning, quindi una connessione sovrascritta con `pin_resolution` impostato viene**rigorosamente rifiutata** con un errore che indica ogni possibile soluzione. Senza il pinning, la connessione procede con il walk-down ma avvisa che la riduzione non può liberare l’aggregato.
- Scappatoia per il lavoro di bench: impostare `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` nell’ambiente del backend per declassare il rifiuto a un avviso forte — ci si connette comunque e si accetta la perdita del pacchetto .

#### Integrità dell’array — quale sottosistema sta perdendo frame

`GET /api/camera/array/<array_id>/capability` contiene un blocco `health` attivo su un
array connesso, rivalutato in una finestra mobile di **10 secondi**. Suddivide la perdita di frame
nelle due cause che richiedono soluzioni opposte, invece di un unico tasso di “incompletezza” che
non ne identifica nessuna:

| Campo | Cosa significa | Quale sottosistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriale) | Il frame **è arrivato ma era strutturalmente errato**— perdita di pacchetti GVSP. |**Rete**: larghezza di banda disponibile, pacing, anello RX della scheda di rete, MTU |
| `never_arrived_rate_pct` (per seriale) | Il frame **non è mai arrivato**— la telecamera non ha scattato, oppure non è stato inviato alcun segnale. |**Trigger / sincronizzazione**: cavo M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Tasso di errore massimo per ciascuna fotocamera. | — |
| `per_cam_rate_pct` | Tasso di incompletezza combinato per fotocamera (entrambe le cause insieme). | — |
| `stable_for_seconds` | Per quanto tempo ogni telecamera è rimasta al di sotto dello 0,01%. | — |

Insieme a `health`, lo stesso record riporta il numero relativo all’intera allocazione in sospeso:

| Campo | Significato |
| --- | --- |
| `wire_ceiling_mbps` | Il budget di banda sostenuto dall’host attualmente in vigore, in MB/s. |
| `wire_ceiling_source` | Da dove proviene quel valore, in parole — ad es. `USB-capped 200 MB/s (was theoretical 1062; …)` o `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` quando è stato impostato da `wire_ceiling_mbps=`. |
| `nic_is_usb` | `true` per un adattatore Ethernet USB. |

Non esiste un wrapper SDK per questo endpoint — leggerlo direttamente:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Leggendolo:** un valore `gvsp_corrupt_rate_pct` diverso da zero con `never_arrived_rate_pct` pari a 0 indica che
il trigger e la sincronizzazione via cavo sono perfetti e che il 100% della perdita si verifica sul percorso di rete — abbassare
`wire_ceiling_mbps` e riconnettersi. Il modello inverso indica invece un problema nel cavo di sincronizzazione o nella
linea di trigger.

> **`target_fps` non è la causa dei frame danneggiati.** Il pacing di GevSCPD viene impostato una volta alla
> connessione, quindi abbassare la frequenza di trigger modifica il ciclo di lavoro e non la
> velocità di trasmissione in burst simultanea. Una riduzione misurata della richiesta di 5× non ha prodotto alcun miglioramento, mentre
> abbassare il limite massimo della linea da 240 a 200 MB/s ha portato lo stesso sistema dal 10,4 % di frame danneggiati allo
> 0,00 %.

> **La riduzione automatica a metà flusso non è disponibile sul firmware TRI032S.** Un array in esecuzione non può
> risolvere autonomamente il problema; scollegare e ricollegare in modo che il selettore del tempo di connessione pianifichi nuovamente in base
> al nuovo limite massimo.

Un **adattatore Ethernet USB è limitato a 200 MB/s** dal sistema di monitoraggio indipendentemente dalle sue
specifiche tecniche: la tabella di efficienza che converte la velocità del collegamento in un valore sostenuto è
derivata dallo standard PCIe, mentre una scheda di rete USB pubblicizza la propria velocità di collegamento Ethernet pur essendo limitata dal
bus USB e dal relativo driver. Il limite è assoluto, non una frazione: un adattatore USB 1 GbE
deriva circa 80 MB/s e non ne risente.

#### `ArraySession`Metodi OTX

| Metodo | Descrizione |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Un gruppo di acquisizione sincronizzato. Restituisce un `CaptureResult` (elenco di dizionari di frame + `.skipped`). Controlli di esportazione riportati di seguito. |
| `capture(..., smart=True)` | **Acquisizione intelligente** — attende che l&#x27;AE si stabilizzi su tutte le telecamere, quindi attiva l&#x27;acquisizione. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Acquisizione più veloce: solo dati grezzi + la lettura DAQ assegnata (+ l’indice combinato libero). Rispecchia il pulsante “Acquisizione più veloce” dell’interfaccia grafica. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Singola / Continua / A intervalli in un unico ciclo. Restituisce `list[CaptureResult]`.**Richiede `count` e/o `duration_s`** per terminare l&#x27;esecuzione (l&#x27;SDKe non supporta Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Avvia la registrazione della vista combinata in tempo reale in formato video/GIF → `RecorderHandle`. Un registratore composito per ogni array. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Avvia una raffica raw Bayer ad alto fps → `RecorderHandle`. Rielabora offline con `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Rielaborare offline una raffica raw salvata in video calibrati. Rimane in attesa fino al completamento (`wait=True`) e restituisce `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Interroga un processo di compilazione offline: `{running, result, error, burst_dir}`. |
| `disconnect()` | Rilascia l’intero array. |

Controlli di esportazione `capture()` (stesso endpoint utilizzato dalla GUI/CLI):

- `processing` / `levels` — `processing="all"` (o `levels=["raw","radiance",…]`) salva ogni tipo di esportazione applicabile per ciascuna telecamera; un singolo valore `processing` salva solo quel livello.
- `aligned=True` — applica il warp all’esportazione non raw di ogni membro in base al [profilo di allineamento](#array-alignment) dell’array (co-registrato); i dati raw rimangono non allineati ma riportano la trasformazione nei metadati. Si ricorre all&#x27;allineamento non allineato (con un avviso visualizzato nel campo `alignment` del risultato) se l&#x27;array non dispone di un profilo.
- `render_index=False` — salta la sovrapposizione dell&#x27;indice di vegetazione per singola telecamera; per impostazione predefinita lo rende dove configurato.
- `force_daq=True` — salva la lettura DAQ/DLS assegnata come sidecar `.daq` anche quando nessun livello selezionato ne ha bisogno.

**Compressione TIFF (solo opzione HTTP):** `ArraySession.capture()` non invia alcun codice `compression`, quindi si applica l&#x27;impostazione predefinita del backend — `POST /api/camera/array/capture` legge un parametro del corpo `compression`, `"deflate"` per impostazione predefinita (zlib L1 senza perdita di dati + predittore orizzontale, ~4,1 MB per fotogramma a piena risoluzione). `"none"` scrive senza compressione (~6,3 MB/fotogramma) con una**velocità di scrittura circa 5 volte superiore** — entrambi sono senza perdita di dati e vengono letti in modo identico durante l’importazione. Il metodo `SDK` non espone alcun parametro `kwarg` per questo; la via d’uscita è ``chloros-cli lattice array-capture --compression none`` o `HTTP` in formato raw. Inoltre, DEFLATE detiene il GIL di `Python`, quindi le scritture compresse non vengono parallelizzate tra i thread di scrittura per ciascuna telecamera — per una cattura continua a piena risoluzione con 8 telecamere alla frequenza del sensore è necessario `compression: "none"`. Dettagli: [CLI Riferimento → array-capture](cli-reference.md).**Sovrascritture di esportazione per singolo membro (solo HTTP):**lo stesso endpoint accetta anche `exclude_serials` (elenco — esclude membri dal set salvato; l’array continua a scattare come un unico gruppo sincronizzato e i membri esclusi vengono restituiti in `excluded`), `serial_levels` (sovrascritture a livello di singola telecamera `{serial: [level tokens]}`) e `serial_index` (sovrascritture di sovrapposizione indice per telecamera `{serial: bool}`). Si tratta di parametri di corpo in parità con la GUI e**non sono ancora kwargs SDK**; i membri assenti dalle mappe ricorrono ai valori a livello di array `levels` / `render_index`.

##### Ispezione delle cam saltate — `CaptureResult.skipped`

`ArraySession.capture()` restituisce un `CaptureResult`, che è una sottoclasse di `list`: iterarlo, indicizzarlo, applicare `len()` — ogni modello esistente continua a funzionare. Il nuovo codice può ispezionare l’attributo `.skipped` per vedere quali camme sono state escluse e perché. Il caso più comune è quello delle telecamere RGB in un array con filtri misti quando si richiede `processing="radiance"` o `"reflectance"` — la radianza per pixel Bayer non ha senso per un sensore a banda larga, quindi il backend salta quelle cam piuttosto che produrre dati privi di significato.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

I token di motivazione seguono lo schema `<level>-not-applicable-to-rgb-cam` (una voce per ogni livello saltato, ciascuna contenente `level`). I salti specifici per la riflettanza sono `reflectance-skipped-no-fresh-dls` (nessuna nuova lettura in direzione discendente disponibile), `reflectance-skipped-bound-daq-unavailable (…)` (impossibilità di raggiungere il DAQ associato) e `dls-uncalibrated-band-<nm>` — la banda si trova prevalentemente al di fuori dell’intervallo radiometricamente calibrato del sensore di luce del DAQ (~374–974 nm), pertanto la divisione assoluta della riflettanza basata sul DAQ viene rifiutata e il fotogramma ricade nettamente sulla risposta del sensore. Tra gli SKU in vendita, solo l’F988 lo attiva; il percorso supportato da quella fotocamera è il flusso di lavoro con pannello di riflettanza.

Livelli `processing`:

| Livello | Uscita |
| --- | --- |
| `"raw"` | Bayer a canale singolo (telecamere monocromatiche: la singola banda) direttamente dal sensore. |
| `"debayered"` *(impostazione predefinita di SDK)* | BGR a 3 canali tramite demosaicizzazione bilineare (telecamere monocromatiche: scala di grigi a 1 canale). |
| `"radiance"` | float32 W/m²/sr/nm tramite la catena radiometrica completa. Solo multispettrale — le telecRGBi vengono ignorate. |
| `"reflectance"` | uint16 0..32768 (Pix4D-ready); richiede un abbinamento DAQ in tempo reale per il riferimento assoluto. Solo multispettrale. |
| `"display"` | Catena completa corrispondente all’anteprima dell’interfaccia grafica (CCM + WB + gamma secondo il profilo della fotocamera). |
| `"all"` | **Un file per ogni livello applicabile** per ciascuna telecamera (in linea con l’impostazione predefinita “Capture All”/CLI dell’interfaccia grafica). Il file `CaptureResult` restituito contiene quindi un dizionario per fotogramma per ogni `(cam, level)`, con il livello specificato in ciascun dizionario; i livelli non applicabili compaiono in `.skipped`. Il valore DAQ utilizzato per qualsiasi fotogramma di riflettanza viene salvato come sidecar `.daq`. |

> **Nota — l’impostazione predefinita differisce da quella dell’CLI.** `ArraySession.capture()` ha come impostazione predefinita `processing="debayered"`; il comando `chloros-cli lattice array-capture` ha come impostazione predefinita `processing="all"`. Passare esplicitamente `processing="all"` dall’SDK per rispecchiare il salvataggio multilivello dell’CLIe/GUI.

### Modalità di acquisizione e registratori

La superficie dell’array rispecchia il pannello di acquisizione della GUI: modalità otturatore Singolo / Continuo / Intervallo / Più veloce, più due registratori (video composito in diretta e raffica raw → rielaborazione offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**è il ciclo Continuo/a intervalli dell’SDK. Poiché non esiste `Ctrl+C` per interromperlo da uno script, è**necessario** passare `count` e/o `duration_s` (si interrompe quando viene raggiunto uno dei due). `interval_s` viene misurato dall’inizio di ogni passaggio (in linea con la GUI). I restanti kwarg vengono passati direttamente a `capture()`.
- **`record`** è *di livello di monitoraggio*: cattura il composito dell’indice combinato in tempo reale così come visualizzato, quindi lo stream combinato deve essere aperto affinché i fotogrammi possano essere acquisiti. Un registratore composito per ogni array (genera un’eccezione se ne è già in esecuzione uno).
- **`burst` → `build_video`** è *di livello analisi*: `burst` scrive i fotogrammi grezzi + un manifesto per fotogramma + un `.daq` per ogni lettura DLS distinta sotto `<output>/bursts/<base>/` alla velocità massima del ciclo di acquisizione*a piena velocità (senza catena, senza exiftool, senza anteprima). `build_video` allinea temporalmente ogni fotogramma al più vicino `.daq` e riesegue la pipeline di importazionecatena di radianza/riflettanza/indice. `products` è un elenco di `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (impostazione predefinita: l’indice combinato). `burst().stop()` avvia inoltre automaticamente una generazione dell’indice combinato «best-effort», restituita come `build_job` nel risultato finale.

#### `RecorderHandle`

Restituito da `ArraySession.record()` e `ArraySession.burst()`. Utilizzarlo come gestore di contesto per l&#x27;arresto automatico all&#x27;uscita dall&#x27;ambito, oppure gestirlo manualmente.

| Membro | Descrizione |
| --- | --- |
| `job_id` | ID del processo di backend (str). |
| `kind` | `"composite"` (da `record`) o `"raw"` (da `burst`). |
| `start_stats` | Il dizionario restituito dalla chiamata `start`. |
| `result` | `None` durante l’esecuzione; il dizionario finale dei risultati di arresto una volta terminata l’esecuzione. |
| `stats(timeout=10.0)` | Statistiche in tempo reale sul processo (fotogrammi scritti, fps effettivi, tempo trascorso). |
| `stop(timeout=60.0)` | Arresta il registratore; restituisce e memorizza nella cache il risultato finale. Idempotente (una seconda chiamata restituisce il risultato memorizzato nella cache). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Collegamento a un array già connesso — `attach_array`

Se l’array è già attivo (è stato aperto dall’interfaccia grafica o una precedente sessione di SDK ha chiamato `connect_array`), utilizzare `attach_array` per acquisirne un handle invece di riconnettersi. <sn><id>In tale situazione,</id></sn> `connect_array` genera sempre l’errore &quot;La telecamera è<sn> già presente nell’array<id>&quot;, poiché l’invio di una richiesta POST a `/array/connect` per un membro del pool non è idempotente; `attach_array` legge `/api/camera/array/list` ed effettua la corrispondenza in base all’array_id o ai numeri di serie.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Modello: gli script SDK che condividono l’ambiente con l’interfaccia grafica desktop dovrebbero provare prima `attach_array` e ricorrere a `connect_array` se nel pool non è ancora presente alcun array.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Importante — la chiusura del context manager provoca la disconnessione.**`ArraySession.disconnect()` invia sempre un POST a `/array/disconnect`; non esiste una protezione &quot;attaccato ma non di proprietà&quot; come nel caso di `CameraSession` / `DAQSensorSession`. Se staicon l’interfaccia grafica e non vuoi smantellare l’array all’uscita dall’ambito,**non usare il blocco `with`** — conserva l’handle in una variabile normale e salta l’`disconnect()` esplicito:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Strumento di analisi della rete

Utile prima di aprire l’array — verifica se le impostazioni proposte sono compatibili:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` è uno tra `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (altrimenti `error`). `auto_capped_fps` indica che la risoluzione richiesta si adatta all’anello RX solo con una frequenza di trigger limitata — mantenere la risoluzione e passare da `target_fps=result["recommended"]["recommended_target_fps"]` a `connect_array` (vedi [Esempio 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Come interpretare la proiezione** (stesso modello del pannello “Impostazioni array” della GUI):

- **Burst (`frame_bytes_total`) viene sommato per ogni singola telecamera in base al formato pixel reale di ciascuna telecamera.**Le telecamere mono**M3M**trasmettono in streaming Mono12 (2 B/px) indipendentemente dal valore `pixel_format` specificato, quindi un fotogramma a piena risoluzione con 4 telecamere è di**~25 MB** con tre telecamere mono, non i ~12,6 MB che si otterrebbero ipotizzando un formato interamente a 8 bit. Il backend determina il formato di ciascuna telecamera in base al suo modello.
- **L’admittanza (`burst_fits_nic_ring`) tiene conto della capacità di scarico**, non si basa sul confronto tra «intero burst» e «anello»: la modalità «sim-emit» si attiva quando l’host svuota l’anello RX più velocemente di quanto le cam lo riempiano. Un host da 10G + cam da 1 GbE**accetta** la piena risoluzione anche quando il burst supera la capacità dell’anello; un host da 1 GbE blocca (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` è un limite conservativo seriale**limite massimo** — `max(readout+emit, N×emit)` con l’emissione per singola telecamera limitata al collegamento 1 GbE, indipendentemente dall’esposizione. Ad es. ~2,8 fps per un array a 12 bit a piena risoluzione con 4 telecamere (corrisponde ai valori misurati in fase di esecuzionemisurato ~2,7–3,0). Modello completo: [CLI Riferimento → Modello fps e burst dell’array](cli-reference.md#array-fps--burst-model).
- **L’over-subscription (`oversubscribed: true`) indica che il limite minimo N × per telecamera supera il-**** — i campi fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) indicano 0, e il ridimensionamento automatico o il binning non possono risolvere il problema (riducono i byte per fotogramma, non i byte al secondo con regolazione del ritmo). Le soluzioni sono: un numero inferiore di telecamere, frame jumbo o una scheda di rete più veloce; `max_cams_collision_safe` segnala il limite massimo (6 telecamere a piena risoluzione su 1 GbE a 1500 MTU, 9 con jumbo). La risposta contiene anche `aggregate_demand_bps`, `collision_safe_ceiling_bps` e `per_cam_floor_bps` (8 MB/s). Vedi [Over-Subscription](#over-subscription-the-per-cam-floor).

### Rilevamento e Elenco

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Gli array LATTICE eseguono l’AE in modo continuo in background non appena vengono collegati, ma una scena appena inquadrata richiede un attimo per convergere. **Smart-capture** è la soluzione pratica integrata: interroga l’esposizione di ciascuna telecamera, attende che l’array sia stabile in una finestra, quindi avvia l’acquisizione. È equivalente all’interfaccia grafica: il pulsante di acquisizione “smart” dell’app desktop richiama lo stesso endpoint di backend.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Quando si utilizza `ChlorosProject` (sezione successiva) si hanno a disposizione più opzioni di regolazione:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

La politica di AE intelligente è conservativa per impostazione predefinita. Impostare `exposure_tolerance_pct` su un valore più stretto per lavori radiometrici esigenti ; allargarela per scene in rapido cambiamento in cui basta che il risultato sia “sufficientemente vicino”.

---

## Sessioni dei sensori DAQ

Pool di backend persistente per i sensori spettrali (DAQ-U su USB, DAQ-M su BLE, DAQ-E su Ethernet). Rispecchia la superficie della fotocamera: rilevamento intelligente, riutilizzo del pool, connessione idempotente.

### Rilevamento intelligente (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Priorità: Ethernet → BLE → USB. Passare un qualsiasi suggerimento esplicito per fissare il trasporto.

### Trasporto fissato

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Metodi `DAQSensorSession`

| Metodo | Descrizione |
| --- | --- |
| `status(timeout=10.0)` | Riepilogo voce pool (stato streaming/registrazione, intervallo di lunghezza d’onda, SHA di calibrazione, tempo di integrazione, frame_avg, stato AE). |
| `latest(n=1, timeout=10.0)` | Restituisce fino a N frame di spettro più recenti. |
| `stream_start()` / `stream_stop()` | Riprende / mette in pausa lo streaming (l’handle rimane aperto). |
| `record_start(output_dir=None, device_name=None)` | Avvia la registrazione di un file .daq. Restituisce il percorso del file. Non è consentito per DAQ-U/M senza un pacchetto di calibrazione AWS (DAQ-E è escluso). |
| `record_stop()` | Interrompe la registrazione. Restituisce `{path, rows}`. |
| `disconnect()` | Rilascia dal pool. Operazione nulla per handle collegati non. |

> **I profili di correzione del limite (`cap_id`) non sono un parametro di regolazione dell’SDK.** `connect_daq_sensor()` / `DAQSensorSession` non espongono alcun parametro `cap_id` né alcun metodo `set_cap`. Selezionare un profilo di correzione del limite della flottatramite l’CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) o i percorsi `/api/daq` HTTP del backend (`/api/daq/connect` e `/api/daq/<id>/cap-id` accettano `cap_id`).

### Discovery — ricerca di un indirizzo a cui connettersi

`discover_daq_sensors()` esegue la scansione delle porte USB / BLE / ETH alla ricerca di sensori che *potresti* aprire. È la controparte DAQ di `discover_lattice_cameras()` e l’unico modo per ottenere il **MAC BLE di un DAQ-M** — un DAQ-E ha un nome host e un DAQ-U una porta COM, ma il MAC non è né stampato sul dispositivo né elencato dal sistema operativo.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Campo | Descrizione |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Porta COM / MAC BLE / nome host — da passare a `connect_daq_sensor` come `port=` / `mac=` / `eth_host=`. |
| `display` | Etichetta leggibile dall&#x27;utente. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, oppure `None` per una porta che la scansione non riesce a identificare (gli adattatori seriali USB sono indistinguibili senza una sonda, quindi i valori sconosciuti vengono visualizzati anziché nascosti). |
| `extra` | Dettagli per trasporto (nome pubblicizzato BLE, produttore USB, IP/FW/… DAQ-E). I valori vuoti vengono omessi. |

| Parametro | Predefinito | Descrizione |
| --- | --- | --- |
| `transports` | tutti e tre | Sequenza (o stringa CSV) che limita la scansione. Vale la pena specificarla quando si sa esattamente cosa si desidera — il BLE è la parte più lenta. |
| `scan_timeout` | 5 | Finestra di scansione per trasporto in secondi; il backend la limita a un intervallo compreso tra 1 e 20. |
| `timeout` | 60,0 | Limite massimo per l’HTTPe dell’ l’intera chiamata (come altrove nell’SDK). |
| `auto_start_backend` | `True` | Avvia un backend locale se non ne è in esecuzione nessuno. Non viene mai avviato per un `backend_url` remoto. |

> **I sensori già aperti nel pool non vengono visualizzati.** Una periferica BLE connessa smette di trasmettere la propria pubblicità e una porta COM aperta non può essere rilevata, quindi la ricerca elenca ciò che è *disponibile per la connessione*. È normale ottenere un risultato vuoto subito dopo aver collegato un dispositivo: usa `list_daq_sensors()` per ciò che hai già in mano. I trasporti la cui scansione non può essere eseguita (bleak / zeroconf non installati) vengono saltati anziché generare un errore, quindi una macchina senza Bluetooth riceve comunque le risposte relative a USB ed ETH.

### Elenco

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Co-tenancy con GUI / CLI

Se la GUI ha già un sensore aperto, la chiamata a `connect_daq_sensor(port="COM3")` da Python restituisce un handle contrassegnato come `already_connected=True`. L’`disconnect()` della sessione è quindi un’operazione nulla, quindi lo script SDK non non rimuove il sensore dall’interfaccia grafica all’uscita dall’oscilloscopio.

### Classi hardware dirette (senza backend)

`daq_sdk` viene riesportato da `chloros_sdk`, quindi è anche possibile gestire i sensori end-to-end all’interno del processo senza il backend:

> **Disponibilità:**`daq_sdk` è incluso nell’installazione desktop di Chloros,**non** con il pacchetto PyPI — `pip install chloros-sdk` fornisce `lattice_sdk` ma esclude `chloros_sdk.DAQ_AVAILABLE == False`. Verificare tale indicatore prima di utilizzare queste classi; su un host pip, pilotate il sensore tramite [`connect_daq_sensor()`](#daq-sensor-sessions), che non richiede librerie di trasporto locali.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Preferite il percorso smart-connect (`connect_daq_sensor`) quando desiderate la proprietà condivisa con la GUI; utilizzate le classi dirette per gli script headless che possiedono il sensore in modo esclusivo.

---

## Automazione del progetto — `ChlorosProject`

Un progetto salvato su Chloros è una cartella contenente `cameras.json` + `sensors.json` + `project.json`. `open_project` carica il manifesto, mentre `connect_all` mette online tutti i dispositivi salvati con le impostazioni salvate — lo stesso stato hardware che si otterrebbe tramite l’interfaccia grafica.

### Esempio minimo

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Oppure come gestore di contesto:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metodi di `ChlorosProject`

| Metodo | Descrizione |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Individua e connette ogni dispositivo salvato. Restituisce un rapporto di connessione per ogni classe. Utilizza un backend in esecuzione quando ce n’è uno in ascolto su `127.0.0.1:5000`; altrimenti ricorre silenziosamente al controllo diretto (senza backend) `lattice_sdk` — non avvia mai un backend. |
| `disconnect_all()` | Chiude tutte le connessioni. |
| `capture_all(output_dir=".")` | Un fotogramma da ogni telecamera + array + spettro da ogni sensore. |
| `stream(camera, overlays=False, fps=10.0)` | Generatore che produce fotogrammi BGR `numpy` da una telecamera (o array) specificata. `overlays=False` è un ciclo di acquisizione diretta `lattice_sdk` (gli array generano dizionari `{serial: frame}`). `overlays=True` instrada attraverso `ChlorosLocal.camera_stream()` → il feed MJPEG `/api/camera/<serial>/stream-annotated` del backend, con il blocco `ui.overlay` salvato dalla telecamera trasmesso come parametri di query. Richiede la modalità backend e una **telecamera autonoma**: una telecamera in modalità diretta genera un&#x27;eccezione `RuntimeError` (il backend non può acquisire una telecamera di proprietà di questo processo) e un array genera `NotImplementedError` (sovrappone il composito per telecamera — trasmette un membro in base al nome). Equivalente one-shot: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Esegue l’allineamento su ogni array attualmente connesso. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Esegue la pipeline di calibrazione/indicizzazione sulle immagini del progetto (racchiude `ChlorosLocal.process`; questi quattro sono gli **unici** argomenti chiave accettati — `indices=` ecc. generano l&#x27;eccezione `TypeError`; imposta gli indici tramite `ChlorosLocal.configure()`). Costruisce in modo differito un `ChlorosLocal()`, che avvia automaticamente un backend. |

Attributi:
- `proj.cameras` — `Dict[str, CameraHandle]` indicizzato per nome E numero di serie.
- `proj.arrays` — `Dict[str, ArrayHandle]` con chiave costituita da nome E array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` con chiave composta da nome E slot_id.
- `proj.config` — Dizionario `project.json["config"]`.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Livelli di elaborazione.** `capture()`, `grab()` e `frame_stream()` utilizzano tutti lo stesso token `processing`
, e la catena è cumulativa: ogni livello esegue tutto ciò che si trova al di sopra di esso:

| Livello | Output | Note |
| --- | --- | --- |
| `raw` | Bayer a 1 canale, nativo del sensore | Nessun demosaic. A questo livello non sono disponibili sovrapposizioni. |
| `debayered` | BGR a 3 canali (**predefinito**) | Demosaic bilineare. L’unico livello che funziona senza la modalità backend. |
| `radiance` | float32, W/m²/sr/nm | Catena radiometrica completa: demosaico + separazione 3×3 (multispec) + DSNU + flat-field + scala NIST, con esposizione × guadagno eliminati in modo che i valori siano assoluti. |
| `reflectance` | uint16, 32768 = 1,0 | Radianza divisa per l’irraggiamento discendente (ρ = π·L/E). Richiede una lettura DLS/DAQ — vedi la nota qui sotto. |
| `display` | 8 bit sRGB-ish | Rendering equivalente alla GUI: CCM + bilanciamento del bianco + gamma tramite il profilo colore attivo della fotocamera. |

Qualsiasi valore diverso da `debayered` richiede la modalità backend; una fotocamera in modalità diretta genera
`NotImplementedError`. `reflectance` richiede una lettura utilizzabile dell’irraggiamento discendente — l’endpoint del fotogramma inserisce
automaticamente il DAQ raggruppato nello slot DLS della fotocamera, ma in assenza di un DAQ associato la catena rifiuta l’
uscita di riflettanza e contrassegna chiaramente la declassazione nei metadati restituiti, anziché restituire silenziosamente
un prodotto di qualità inferiore.

> **Scala DN di riflettanza — non codificarla in modo rigido.** La riflettanza LATTICE utilizza `32768` = ρ 1,0 e registra
> XMP `Chloros:PixelScale=32768`; la riflettanza Survey3 utilizza `65535` = ρ 1,0 e non contiene
> tag `Chloros:*`. Leggere il tag e dividere per esso. È definito nel dominio uint16, quindi rimane
> `32768` per ogni formato che effettua un ridimensionamento (TIFF a 16 bit, 8 bit PNG /JPG, 32 bit percentuale) — normalizza
> prima il tipo di dati memorizzato riportandolo a uint16 (×257 da 8 bit, ×65535 da float). L’unica eccezione:
> un’acquisizione da sorgente a 8 bit salvata come 8-bit TIFF viene *tagliata* (clipped), non ridimensionata, quindi nessuna scala la descrive
> — Chloros omette completamente `PixelScale` e la tupla MicaSense in quel caso. Considerare un
> tag mancante in un file di riflettanza LATTICE come “nessuna scala valida”, non come valore predefinito.

> **EXIF trasferito nell’esportazione.** `process()` copia il blocco GPS dell’acquisizione di origine
> **e il relativo ExifIFD** su ogni prodotto; pertanto, le esportazioni contengono `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` e `CameraSerialNumber`, oltre al
> georeferenziazione. `FocalLength` è il file da cui Pix4D ricava la distanza tra i campioni al suolo (GSD); senza di esso
> la ricostruzione ricade su una scala estremamente errata (in un caso misurato, un sito di 411 m
> è stato trasformato in uno di 47,8 km). La copia non è volutamente `-all:all`: i tag strutturali di IFD0 compromettono
> l’output di LATTICE, mentre `ExifImageWidth`/`Height` sono esclusi perché descrivono l’acquisizione
> della sorgente piuttosto che il raster esportato.

Sottomarcatori della fase di acquisizione (si applicano ai livelli radiometrici — `radiance`, `reflectance`, `display`):

| Flag | Predefinito | Significato |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + separazione 3x3 + scala radiometrica NIST. |
| `apply_white_balance` | `True` | LUT di bilanciamento del bianco. Compatibile con DLS quando un DAQ è associato alla fotocamera. |
| `apply_index` | `False` | Valutazione dell’indice di vegetazione. |
| `index_expression` | `None` | Formula di override. Se non vuota → abilita automaticamente l’indice. |
| `annotated` | `False` | Sovrapposizione decorazioni GUI (zebra/griglia/picchi). Non disponibile per `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Il tipo di ritorno è `CapturePathMap`, non `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` è `Dict[str, Union[str, List[str]]]`: a livello singolo
> `processing` assegna a ogni numero di serie un percorso, mentre uno a più livelli (`"all"`, o un
> elenco esplicito `levels`) fornisce l’**elenco ordinato** di tutti i prodotti salvati per quella
> telecamera. Un composito combinato in tempo reale, se fosse in streaming, arriverebbe sotto la chiave aggiuntiva
> `"combined"` anziché sotto un numero di serie. Il codice che presuppone `str` genera un errore nella
> forma a elenco senza che alcun verificatore di tipi lo segnali — l’annotazione indicava `Dict[str, str]`
> per un po’ dopo il rilascio del formato a elenco, ed è per questo che esiste l’alias. Normalizza
> quando desideri il formato piatto:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Allineamento degli array

`ArrayHandle` espone l’intera superficie di allineamento. I profili sono, per impostazione predefinita, validi solo per la sessione: chiamare esplicitamente `export_alignment()` per renderli persistenti.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Allineamento al momento della connessione

`connect_all(align=...)` è in grado di allineare automaticamente ogni array al momento della connessione:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Se non specificato, ricorre a `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Hardware diretto (senza backend)

Quando si desidera azzerare la dipendenza dal backend (CI, robot headless, sistemi embedded), importare direttamente `lattice_sdk` e `daq_sdk`: entrambi vengono riesportati da `chloros_sdk`. Verificare `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` è presente nel pacchetto PyPI (ma richiede la presenza del runtime Arena SDK), mentre `daq_sdk` è fornito solo con l’installazione desktop.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Preimpostazioni e trigger

Tre delle quattro preimpostazioni sono **free-run**: la telecamera espone in modo continuo e, non appena
`capture()` rileva il fotogramma successivo, restituisce il fotogramma stesso. `triggered` costituisce l’eccezione: arma la
telecamera in attesa di un fronte d’impulso hardware sulla Linea 2, quindi non cattura nulla finché non ne arriva uno.

| Preset | Trigger | Da usare quando |
| --- | --- | --- |
| `default` | free-run | uso generale |
| `high_speed` | free-run | 8 bit, limite di 60 fps, esposizione breve |
| `high_quality` | funzionamento libero | 12 bit, nessun limite di fps — la scelta abituale per le foto |
| `triggered` | **pronta, Linea 2** | la fotocamera è collegata a un cavo di sincronizzazione M8 e viene attivata da un altro dispositivo |

Se si seleziona `triggered` (o si imposta manualmente `trigger_mode="On"`) senza che nulla
attivi la Linea 2, ogni `capture()` andrà in timeout — correttamente, poiché hai chiesto
alla fotocamera di attendere. L’SDKa spiega questo comportamento quando si verifica; vedi
[SC_ERR_TIMEOUT durante l’acquisizione](#direct-hardware-backend-free).

> **Nota — I messaggi «GVSP probe» / `SC_ERR_TIMEOUT -1011` al momento della connessione non sono errori.**&gt; Al momento della connessione, l’SDKe cerca di negoziare**jumbo frame** (pacchetti GVSP da 9000 byte) per un throughput più elevato. Su un collegamento NIC diretto punto-punto (ad es. un indirizzo `169.254.x.x` link-local), la rete solitamente non è in grado di trasportare jumbo frame, quindi questa sonda va in timeout e registra righe del tipo:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Questo è il **fallback previsto**: l’SDKa automaticamente ai pacchetti standard da 1500 byte e la telecamera continua a connettersi normalmente (le righe `[chunk-enable …]` che seguono fanno parte della normale sequenza di connessione). L’acquisizione continua a funzionare.
>
> È possibile ignorare questa sonda, ma **non si tratta semplicemente di un silenziatore di log: disattiva i jumbo frame.** La telecamera risponde ai ping «Don&#x27;t-Fragment» solo fino a 1500 byte, indipendentemente dalla qualità della rete; pertanto, il test del ping da solo non può mai rilevare i jumbo frame; questa sonda è l&#x27;unica in grado di farlo. Disattivandola, la telecamera utilizzerà per sempre pacchetti standard da 1500 byte, su qualsiasi rete:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Ne vale la pena solo su una rete che *sai* non supportare i jumbo, dove consente di risparmiare circa un secondo di tempo di connessione per ogni telecamera. Poiché si tratta di una vera e propria scelta di compromesso piuttosto che di una modifica puramente estetica, l’SDKa ora lo segnala quando la si utilizza:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Non modificarlo a meno che tu non abbia un motivo valido.** Se lasciato abilitato, ogni connessione ricalibra la rete effettivamente disponibile: collegati a uno switch compatibile con i pacchetti jumbo e la connessione successiva rileverà automaticamente i pacchetti jumbo, senza bisogno di configurazioni né riavvii.
>
> Se *desideri* la velocità di trasmissione dei pacchetti jumbo, abilita jumbo end-to--end (MTU della scheda di rete 9000 + uno switch che li lasci passare), oppure fissalo con `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` quando sai che il collegamento lo supporta — anche se è preferibile usare `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` per singolo comando piuttosto che impostarlo in modo permanente, poiché una dimensione fissa salta la fase di rilevamento e impedisce l’adattamento alla rete a monte. **Ogni** dispositivo lungo il percorso deve supportare i pacchetti jumbo — compresi eventuali splitter o iniettori PoE, che sono la causa più comune per cui una configurazione altrimenti compatibile con i pacchetti jumbo non riesca a trasmetterli.

> **`SC_ERR_TIMEOUT -1011` durante `capture()` / `grab*()` è un problema diverso: in quel caso si tratta di un vero e proprio errore.**&gt; La nota sopra riportata riguarda esclusivamente l’errore `-1011` registrato dalla**sonda di tempo di connessione**. Lo stesso errore generato da una**cattura** indica che la telecamera si è collegata correttamente ma non sta inviando alcuna immagine:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Laè una telecamera il cui canale *di controllo* funziona correttamente — il rilevamento funziona, le impostazioni e le scritture `[chunk-enable …]` vanno a buon fine — mentre *ogni* fotogramma va in timeout.
>
> **La causa più comune è che la telecamera sia impostata per un trigger hardware.** Con `trigger_mode="On"` e `trigger_source="Line2"`, la telecamera non emette alcun segnale finché non arriva un fronte di segnale sul cavo di sincronizzazione M8. Se non c’è alcun cavo che pilotia quella linea, ogni acquisizione rimane in attesa all’infinito. La telecamera non è guasta e la rete funziona correttamente — sta semplicemente facendo esattamente ciò che le è stato comandato.
>
> `CameraSettings()` e i preset `default` / `high_speed` / `high_quality` sono a funzionamento libero, e un’acquisizione che va in timeout mentre è attiva fornisce una spiegazione invece di visualizzare semplicemente `-1011`. `PRESETS["triggered"]` attiva Line2, come previsto.
>
> Per forzare qualsiasi telecamera a funzionare in modalità libera:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Se continua a andare in timeout con `trigger_mode="Off"`, la telecamera non sta effettivamente trasmettendo dati — inviateci il log e `ip link show`.

#### Profili colore (anteprima live RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` seleziona il profilo colore dello schermo per l’**anteprima live** sulle telecamere RGB (le telecamere multispectrali ignorano l’impostazione):

| Profilo | Significato |
| --- | --- |
| `raw` | Ignora completamente la catena radiometrica. |
| `linear` | DSNU + flat + WB, senza CCM né gamma. |
| `natural` | Lineare + CCM misurato + gamma sRGB, con solo la finitura &quot;economica&quot; (smussamento della crominanza + desaturazione delle luci) — l&#x27;impostazione predefinita realistica. |
| `enhanced` | `natural` più la finitura completa con parità hub (rimozione frange, vivacità, contrasto locale CLAHE). Aspetto più ricco a circa **il doppio del costo di finitura per fotogramma**, quindi un framerate LIVE inferiore. |
| `custom_temp` | `natural` ma con il bilanciamento del bianco fissato al valore in Kelvin di `custom_cct_k` (DLS ignorato; limitato a 2000–10000 K lato backend). |

Il profilo è un controllo di velocità/aspetto **solo per l’anteprima in tempo reale**: le acquisizioni salvate ottengono sempre la finitura ricca completa indipendentemente dal profilo selezionato, quindi scegliere `natural` per recuperare tempo di frame non riduce la qualità di ciò che viene salvato su disco. Un profilo sconosciuto attiva `ValueError`; quando un backend chloros è raggiungibile, la modifica viene inviata anche a esso tramite POST, in modo che il fotogramma di anteprima successivo la rifletta (gli utenti di direct-SDK senza backend ottengono comunque la modifica delle impostazioni).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Telecamere mono (M3M) e `Calibration`

Una telecamera mono **M3M** (`M3M-<lens>-F<wavelength>`) è a banda singola: un piano in scala di grigi, nessun mosaico di Bayer, nessuna matrice di crosstalk spettrale 3×3. `Calibration` la riconosce e espone un flag `is_mono`. La riflettanza si applica comunque come mappa radiometrica per banda (la separazione dei colori è la matrice identità), ma i calcoli multibandasu una singola fotocamera genera risultati sensati anziché restituire dati privi di senso:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Per costruire un indice di vegetazione con hardware monocromatico, combinare diverse fotocamere M3M a diverse lunghezze d’onda in una pila multibanda allineata (vedi [Allineamento dell’array](#array-alignment)) e calcolare l’indice su quell’stack anziché su una singola fotocamera.

Modalità diretta DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **Chiavi accettate da `apply_sensor_settings`**— esattamente `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; deprecato a favore di `cap_id`), `filter_model` (DAQ-M) e `cap_id` (tutti i tipi DAQ; `None`/`""`/`"none"` = sensore nudo, senza correzione del condensatore). Le chiavi sconosciute vengono**vengono ignorate silenziosamente** — ad es. `{"integration_time": 64}` non fa nulla (deve essere `integration_time_ms`). Restituisce `{"applied": [...], "errors": {...}}` e non genera mai un&#x27;eccezione.

`chloros_sdk` riesporta solo la superficie principale utilizzata sopra. L’APIe pubblico completo di `daq_sdk` (22 nomi) aggiunge quanto segue — importarli direttamente da `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Eccezioni

Intercettare la classe base per gestire &quot;qualsiasi problema si verifichi Chloros&quot;:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` e `ChlorosConfigurationError` sono esportati al livello superiore insieme al resto; sono inoltre importabili da `chloros_sdk.exceptions` come mostrato.

Gerarchia:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Esempi end-to-end

### 1. Elaborazione di una cartella con una barra di avanzamento personalizzata

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Matrice LATTICE in tempo reale → Riflettanza + Riferimento DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Campagna di acquisizione guidata dal progetto

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Flusso di fotogrammi da più telecamere → Pipeline NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Script di acquisizione diretto su hardware (senza backend) senza interfaccia grafica

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Verifica delle funzionalità prima di collegare un array di 4 telecamere

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Equivalente della ricetta di acquisizione (Python puro)

Il DSL delle ricette di CLI ha un equivalente diretto in Python:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Avvio automatico del backend

I punti di ingresso smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor` e `discover_lattice_cameras` — sono client &quot;HTTP&quot; leggeri che presuppongono che un backsia in ascolto su `127.0.0.1:5000` (l’URL predefinito della superficie smart-connect). Quando l’interfaccia grafica o CLI è già in esecuzione, ne è già presente uno. Partendo da uno script nudo, potrebbe non essercene nessuno — quindi queste funzioni **avviano automaticamente il binario del backend in bundle** (senza finestra, proprio come fa `ChlorosLocal`) prima della loro prima chiamata, quindi attendono fino a `backend_startup_timeout` affinché si avvii.

Regole:

- **Viene avviato solo un URL locale.** È ammesso un `backend_url` che punti a `localhost` / `127.0.0.1` / `[::1]`; qualsiasi altro host viene considerato come appartenente a un’altra macchinae non viene mai generato.
- **Il backend rimane in esecuzione per essere riutilizzato** (come nel caso di CLI) —non avviene alcuno spegnimento implicito all’uscita dello script. Eseguendo nuovamente lo script, viene riutilizzato il backend attivo.
- **Disattivare l’opzione con `auto_start_backend=False`** in una qualsiasi di queste chiamate (ad esempio quando si è indicato un backend remoto o si gestisce autonomamente il ciclo di vita del backend).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Se il binario integrato non può essere individuato o avviato, la successiva chiamata HTTP genera un errore `ChlorosConnectError` **specifico per la piattaforma** e su cui è possibile intervenire, anziché una semplice traccia di connessione rifiutata — su Windows rimanda all’app desktop o a un comando `chloros-cli`; su Linux (senza GUI) rimanda a un comando `chloros-cli` o a `.deb`.

---

## Ambiente e intestazioni

L’SDKa contrassegna ogni chiamata al backend HTTP con `X-Chloros-Client: sdk`. Il backend applica le regole di licenza di SDK / CLI (è richiesto il login **e** un piano a pagamento Chloros+), anziché il percorso gratuito dell’interfaccia grafica. Questa impostazione viene configurata automaticamente al momento dell’importazione: non è necessario intervenire.

`http://localhost` e `http://127.0.0.1` vengono rilevati come backend locale. Le chiamate ad altri host (ad es. il proprio servizio di analisi) rimangono invariate.

È possibile sovrascrivere il backend URL passando `backend_url=` (o `api_url=` su `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Un `backend_url` non loopback raggiunge solo un backend source/dev — i backend forniti si legano solo al loopback; vedere Modalità backend remoto per lo schema del tunnel.)

---

## Versioni e compatibilità

- La versione SDK viene esposta come `chloros_sdk.__version__`.
- L’SDKa il comportamento ai pin della versione del backend in dotazione. La combinazione di un backend più vecchio SDK con un backend più recente di solito funziona (endpoint compatibili in avanti), ma la combinazione di un backend più recente SDK con un backend più vecchio può causare errori `404` sugli endpoint più recenti: aggiornare l’app desktop in modo che sia compatibile.
- L’interfaccia smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) e l’endpoint di analisi di rete restituiscono schemi JSON stabili; i nuovi campi sono aggiuntivi.

---

## Suggerimenti per la risoluzione dei problemi

- **`ChlorosAuthenticationError: Login required`** → Eseguire una volta `chloros-cli login EMAIL PASSWORD` su questo computer oppure accedere tramite l’app desktop Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Le chiamate Smart-Connect avviano automaticamente un backend locale, quindi questo messaggio compare solo quando il binario in dotazione non viene trovato o non può essere avviato (ad es. su un host con solo pip e senza pacchetto desktop). Il messaggio varia a seconda della piattaforma: su Windows aprire l’app desktop o eseguire un qualsiasi comando `chloros-cli`; su Linux eseguire un comando `chloros-cli` (non esiste una GUI) o installare `.deb`. Per un backend remoto, passare `backend_url=` (e `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** in fase di importazione → `lattice_sdk` non è stato caricato (in genere le DLL di runtime di Arena SDK non sono installate). La superficie non relativa alla telecamera funziona ancora.
- **Array connect restituisce una risoluzione inferiore a quella nativa**→ La funzione smart-prep del backend riduce automaticamente le dimensioni del fotogramma per adattarle alla linea di trasmissione. Utilizzare `analyze_array_network()` per capire il motivo, quindi aggiornare il collegamento, accettare la riduzione, oppure passare `force_tier="slip-emit-and-capture"` per l’acquisizione sequenziale. La misura di sicurezza della riduzione**non** copre l’over-subscription aggregata (`oversubscribed: true`, campi fps 0): un numero eccessivo di telecamere per la linea non può essere risolto tramite binning/ROI — ridurre il numero di telecamere, abilitare i jumbo frame o passare a una scheda di rete più veloce (vedi [Sovrasottoscrizione](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` segnala che l’anello di ricezione della scheda di rete è molto piccolo (~0,26 MB) / i gate di connessione riportano &quot;FRAMES WILL DROP&quot;** → L’anello di ricezione della scheda di rete dell’host è al suo valore predefinito (spesso reimpostato a 32 dopo un aggiornamento del driver della scheda di rete). Su una scheda Realtek USB 10GbE, impostare `ReceiveBufferLen=256` e `PendingReceives=64` (livello elevato), quindi riavviare il backend affinché rilegga l’anello. Procedura completa: [Riferimento CLI → Configurazione e ottimizzazione della scheda di rete host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **L&#x27;host si blocca al riavvio/spegnimento, con successivi errori WMI `Invalid class` / la scheda di rete non si abilita** → Driver USB 10GbE obsoleto che causa l&#x27;errore `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Aggiornare il driver della scheda alla versione più recente (≥ 2026) e riapplicare le impostazioni del receive-ring. Vedere [Riferimento CLI → Configurazione e ottimizzazione della scheda di rete host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Riflettanza rifiutata** → È necessario associare un DAQ attivo alla telecamera (o all’array) per ottenere la riflettanza in scala assoluta. Effettuare l’associazione tramite l’interfaccia grafica oppure utilizzare `processing="radiance"` (W/m²/sr/nm), che non richiede un sensore abbinato.
- **L’acquisizione con `smart=True` richiede più tempo del previsto** → La convergenza AE dipende dalla dinamica della scena; stringere `exposure_tolerance_pct` o accorciare `stability_window_s` se si desidera un trigger più veloce (ma meno stabile).

---

## Vedi anche

- [Riferimento CLI](cli-reference.md) — ogni sottocomando CLI corrisponde a una chiamata SDK.
- [Guida ai sensori DAQ](../daq/README.md) — cablaggio, calibrazione e regole di registrazione specifiche per ciascun sensore.
- Documentazione online: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
