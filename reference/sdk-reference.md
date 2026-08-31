# Chloros Python SDK Riferimento

**Versione:**

1.2.0**Generato:**29/07/2026 19:19 ·**Aggiornato:** 30/08/2026**Pacchetto:** `chloros-sdk` (PyPI)**Destinatari:** Ottimizzato per l’utilizzo da parte di modelli di linguaggio di grandi dimensioni (LLM); leggibile dall’uomo.**Ambito:** Tutte le classi pubbliche, le funzioni e gli helper esposti da `import chloros_sdk`, con esempi copiabili e incollabili che coprono l’elaborazione delle immagini, il controllo di una singola telecamera, gli array sincronizzati, i sensori DAQ e l’automazione dei progetti.

Se ti interessano solo i punti salienti, vai a:
- [Installazione e guida rapida](#installation)
- [Smart-Connect per array LATTICE](#smart-connect-for-lattice-cameras)
- [Sessioni con sensori DAQ](#daq-sensor-sessions)
- [Automazione del progetto](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Architettura in 60 secondi

SDK è un sottile strato Python che si sovrappone al backend Chloros (lo stesso server Flask utilizzato dall’interfaccia grafica desktop e da CLI). Per l’automazione si importa `chloros_sdk` e si chiamano metodi di alto livello; sotto il cofano, ogni chiamata diventa una richiesta HTTP al backend locale sulla porta 5000 — `http://127.0.0.1:5000/api/...` (deliberatamente non `localhost`, che viene risolto prima come `::1` su Windows e costa circa 2 s per richiesta contro un backend solo IPv4). Il backend gestisce il pool hardware — telecamere, sensori DAQ, profili di allineamento, frame buffer — quindi gli script SDK possono coesistere con la GUI senza contendere le porte seriali o la larghezza di banda della scheda di rete.

Ci sono tre interfacce che userete:

1. **`ChlorosLocal` + funzioni libere** (`process_folder`, `process_lattice_capture`) — Pipeline di elaborazione delle immagini. Esegui l’elaborazione di un’intera cartella attraverso le fasi di calibrazione / debayer / esportazione dell’indice da un’unica chiamata Python.
2. **Handle Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Apri una sessione backend persistente per l’hardware in tempo reale. Stesso flusso “smart-prep” dell’interfaccia grafica: sonda di rete, selezione automatica del livello, PTP, inizializzazione AE, configurazione del trigger GPIO.
3. **`ChlorosProject` / `open_project`** — Carica un progetto salvato (cartella contenente `cameras.json` + `sensors.json` + `project.json`), collega tutto in una volta e gestisci le acquisizioni tramite handle denominati.

Le superfici 1 e 2 **avviano automaticamente un backend locale** se non ce n’è già uno in ascolto (lo stesso binario incluso nel pacchetto che la GUI/CLI avvia) — quindi uno script semplice funziona da una shell nuova senza che sia necessario avviare prima un backend. Passare `auto_start_backend=False` per disattivare questa funzione (ad es. quando si punta a un backend remoto, che non viene mai avviato). Vedi [Avvio automatico del backend](#backend-auto-start). Surface 3 si comporta in modo diverso: `open_project()` non accetta alcun parametro `auto_start_backend`, e `connect_all()` non avvia mai un backend — effettua una sola richiesta a `http://127.0.0.1:5000` una volta e, se non riceve risposta, passa silenziosamente al controllo diretto (senza backend) del dispositivo `lattice_sdk`. Solo `proj.process()` e `stream(..., overlays=True)` creano in modo differito un `ChlorosLocal()` (che si avvia automaticamente).

Tutti e tre sono soggetti a controllo di autenticazione: eseguire `chloros-cli login` una volta sulla macchina oppure effettuare l’accesso tramite l’interfaccia grafica del desktop. Le chiamate a SDK senza una sessione valida generano l’errore `ChlorosAuthenticationError`.

Requisiti:
- Python 3.7+ (come dichiarato dal pacchetto; sviluppato/testato su 3.10)
- Chloros Desktop installato localmente (il binario del backend è incluso nel programma di installazione)
- Accesso attivo a Chloros+. Il livello minimo richiesto per SDK/CLI è **Copper**o superiore (Copper / Bronze / Silver / Gold); il livello gratuito**Iron**non ha accesso a SDK/CLI. Questo viene applicato**lato server**: ogni richiesta contrassegnata con SDK/CLI deve includere sia una sessione attiva che un piano a pagamento, altrimenti il backend restituisce `403` con `error_code: PLAN_UPGRADE_REQUIRED` (visualizzato come `ChlorosLicenseError` da `ChlorosLocal` e come `ChlorosConnectError` dagli helper `connect_*`). Un chiamante disconnesso riceve invece `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — i due sono distinti perché- l’esecuzione di `chloros-cli login` risolve il primo problema ma non può risolvere il secondo.
- L’utilizzo offline è supportato entro il periodo di tolleranza del piano: il livello viene letto dalla cache di convalida del server (5 min) o dalla cache delle licenze firmate e legate al dispositivo (30 giorni per i piani mensili, fino alla scadenza dell’abbonamento per quelli annuali). Una volta scaduto tale periodo di tolleranza, il piano passa alla versione gratuita e l’accesso a SDK/CLI viene interrotto fino a quando il computer non riesce a connettersi al server almeno una volta. `chloros-cli status` (`GET /api/license-status`) rimane raggiungibile nel livello gratuito, quindi il motivo è evidente: è l’unico percorso SDK/CLI esente dal filtro del livello.
- Windows 10/11 a 64 bit, **Ubuntu 22.04 LTS o versioni successive**, oppure Jetson (JetPack 6). Ubuntu 20.04**non** è supportato: le dipendenze di `.deb` derivano da ciò a cui il backend si collega, incluso `libc6 (>= 2.34)`, e Focal distribuisce glibc 2.31.

---

## Installazione

Python SDK è un sottile strato Python che si sovrappone al backend Chloros. Per tutto ciò che va oltre alcuni flussi di lavoro DAQ, è necessario che il **pacchetto desktop Chloros sia installato localmente** (programma di installazione Windows o Linux `.deb`) — che fornisce il binario del backend, il runtime Arena SDK per le telecamere LATTICE e i pacchetti di calibrazione.

Download più recenti: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Passaggio 1 — Installare il pacchetto della piattaforma Chloros

#### Windows (.exe)

1. Scaricare `Chloros-Setup-x.y.z.exe` dalla pagina dei download.
2. Eseguire il programma di installazione e seguire la procedura guidata. Il percorso di installazione predefinito è `C:\Program Files\MAPIR\Chloros\`.
3. Avviare Chloros almeno una volta ed effettuare l&#x27;accesso con il proprio account Chloros+.

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

### Passaggio 2 — Installare Python SDK

**Il programma di installazione Chloros include un file wheel SDK corrispondente.** Ogni programma di installazione Windows e ogni file .deb Linux inserisce sul disco un file `chloros_sdk-X.Y.Z-py3-none-any.whl` sul disco che corrisponde esattamente alla versione della GUI / CLI / backend. Non è necessario consultare PyPI per rimanere sincronizzati.

#### Windows

Il programma di installazione esegue automaticamente `pip install` sul wheel in dotazione utilizzando il launcher di sistema Python (si preferisce il launcher di `py.exe`, in caso contrario ricorre a `python -m pip`). Non è richiesta alcuna azione: `import chloros_sdk` funziona nel proprio ambiente Python dopo un&#x27;installazione riuscita. Se sul sistema non è presente Python, il programma di installazione salta silenziosamente questo passaggio e l’interfaccia grafica + CLI continuano a funzionare.

#### Linux (.deb)

Il file .deb colloca il wheel in `/usr/lib/chloros/sdk/`. Il comando `postinst` riporta il comando esatto: le distribuzioni conformi alla PEP 668 rifiutano per impostazione predefinita le scritture globali tramite pip, pertanto non procediamo all’installazione automatica:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Per le implementazioni Jetson in modalità air-gapped, il processo è completamente offline: la wheel è già presente sul disco.

#### PyPI pubblico

Per gli host che utilizzano solo pip (senza pacchetto desktop Chloros installato; flussi di lavoro con backend remoto o solo DAQ):

```bash
pip install chloros-sdk
```

PyPI viene aggiornato sulle build dell’installer della versione di rilascio, quindi il wheel pubblicato corrisponde all’ultima versione stabile. Le build di sviluppo (ad es. `1.1.4.dev1`) vengono distribuite solo tramite il wheel dell’installer in bundle.

#### Verifica

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **È richiesto l’abbonamento a Chloros+.** Tutte le chiamate a SDK richiedono un accesso attivo a Chloros+. Eseguire `chloros-cli login user@example.com 'YourPassword'` una volta per macchina; le credenziali vengono memorizzate nella cache in `~/.chloros/`.

### È necessario il pacchetto Desktop?

Il pacchetto pip da solo **non** è sufficiente per la maggior parte dei flussi di lavoro. Ecco cosa serve per ciascuna superficie SDK:

| Superficie SDK | È necessario il pacchetto Desktop? | Perché |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Sì** | Avvia automaticamente il binario di backend su `/usr/lib/chloros/chloros-backend` (Linux) o `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Sì**(locale)**/ No**(remoto) | Client HTTP puri sul backend. Backend locale → è richiesto il pacchetto desktop. Backend remoto → `backend_url=`**tramite un tunnel** (vedi Modalità backend remoto — i backend forniti si legano solo al loopback). |
| `ChlorosProject` / `open_project` | **Sì** | Gestisce i progetti salvati tramite il backend. |
| Classi LATTICE dirette (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Sì** | Richiede il runtime nativo Arena SDK incluso nel pacchetto desktop. In caso contrario, `CAMERA_AVAILABLE` corrisponde a `False` al momento dell’importazione. |
| Classi DAQ dirette (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **No** | Python puro su pyserial/bleak/zeroconf. Un ambientepuò gestire i DAQ end-to-end. |

### Modalità backend remoto (host solo pip, tramite tunnel)

> **Il backend fornito non è raggiungibile tramite LAN.** Le build
> di produzione si legano solo al loopback (entrambe le famiglie di loopback) e rifiutano categoricamente l’
> unica modalità non loopback (`CHLOROS_CLOUD_MODE`), quindi
> `backend_url="http://<lan-ip>:5000"` **non può funzionare con un
> Chloros** installato — tale schema ha sempre funzionato solo con un backend source/dev
> backend. Per pilotare un backend su un’altra macchina, inoltra tu stesso la sua porta loopback
> e punta SDK verso il tunnel:

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

Gli host headless / CI / robotica possono mantenere una macchina con l’installazione desktop completa come &quot;Chloros&quot; e `pip install chloros-sdk` ovunque altrove — ma il trasporto tra di essi avviene tramite il tunnel configurato dall’utente sopra indicato, non tramite una connessione LAN diretta URL.

> **Limitazione nota — `ChlorosLocal` non supporta esclusivamente pip.** `ChlorosLocal(backend_url=BACKEND)` attualmente risolve un binario backend locale nel proprio costruttore *prima* effettuare il rilevamento di URL e genera l’errore `ChlorosBackendError` (“Backend Chloros non trovato…”) quando non è installato alcun pacchetto desktop — anche se è presente un backend remoto raggiungibile. Solo l’interfaccia smart-connect sopra indicata (`connect_camera` / `connect_array` / `connect_daq_sensor`, oltre a `analyze_array_network` e gli helper `list_*` / `discover_*`) funzionano da un host solo pip.

### Flusso di lavoro solo DAQ (host solo pip)

Se avete bisogno solo di sensori DAQ e non utilizzate le telecamere LATTICE né l’elaborazione delle immagini, il pacchetto pip è autonomo:

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

Nessun backend, nessun .deb, non è richiesto alcun login Chloros+ per il lavoro di acquisizione dati (DAQ) direttamente sull’hardware.

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

## Indice di primo livello di API

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
| `import_images(folder_path, recursive=False)` | Importa immagini RAW/TIF/JPG/DNG **e registrazioni del sensore di luce `.daq`**. Restituisce `count` (immagini) e `scan_count` (registrazioni). Emette un avviso solo se la cartella non contiene né le une né le altre. |
| `export_light_sensor(daq=True, csv=True)` | Scrive i file calibrati `.daq` + `.csv` per ogni registrazione del sensore di luce del progetto, in `<project>/Light Sensor/`. Vedi [Registrazioni del sensore di luce](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Impostare i parametri di elaborazione. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Eseguire la pipeline. Restituisce `{"status": "complete", "async": False}`, oltre a una chiave `summary` quando il backend ne fornisce uno — vedi [Riepilogo post-esecuzione e suggerimenti](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Verifica lo stato del backend. |
| `logout()` | Cancella le credenziali memorizzate nella cache. |
| `shutdown_backend()` | Termina il backend (se avviato da SDK). |
| `discover_cameras()` | Individua le telecamere LATTICE **tramite il backend di questa istanza** (`/api/camera/discover`). Restituisce un elenco di dizionari (`serial`, `model`, `ip`, …) — della stessa struttura visualizzata dalla GUI/CLI. Elenco vuoto se non ne viene trovata nessuna o se il backend è irraggiungibile. |
| `camera_capture(output_dir, format="tiff", **settings)` | Acquisisce un singolo fotogramma**tramite il backend**(avviato automaticamente da questo handle) in modo che riceva la stessa preparazione della GUI/CLI (impostazione predefinita a 12bit per impostazione predefinita, riutilizzo del pool, metadati di calibrazione incorporati). Risolvere il bersaglio con `serial=` o `device_index=`; passare `exposure`/`gain`/`pixel_format`/`preset` come `**settings`. Restituisce il dizionario dei metadati legacy (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Genera fotogrammi di anteprima compositi sovrapposti da una telecamera in pool — client MJPEG leggero sulla rotta `/api/camera/<serial>/stream-annotated` del backend (zebra / griglia / mirino / istogramma / peaking / punto disegnato dal lato server). `decode=True` restituisce array BGR; `False` restituisce byte JPEG grezzi. Raggiungibile anche a livello di progetto come `ChlorosProject.stream(overlays=True)`. |

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

### Registrazioni del sensore di luce — calibrati `.daq` + `.csv`

È possibile registrare un DAQ-U / DAQ-M / DAQ-E **senza** il relativo pacchetto di calibrazione. Questo è
ciò che i registratori pubblici [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): scrivono i conteggi grezzi del sensore e contrassegnano il
file in modo che Chloros recuperi la calibrazione di fabbrica di quel sensore **tramite la cache seriale** — dalla cache locale
, poi da MAPIR Cloud — e la applica al momento dell’importazione.

Chloros riscrive il risultato come due prodotti per ogni registrazione, sotto
`<project>/Light Sensor/`:

| Prodotto | Che cos’è |
| --- | --- |
| `<name>_calibrated.daq` | L’archivio rielaborabile — stesso schema di una registrazione in tempo reale, ora con l’indicazione del bundle che lo ha generato. Reimportarlo **non** comporta una seconda calibrazione. |
| `<name>_calibrated.csv` | Irradianza spettrale in W/m²/nm sulla griglia di lunghezze d’onda propria del sensore, una riga per ogni lettura, più colonne fotometriche (potenza totale, lux fotopico/scotopico, PPFD e la sua suddivisione in blu/verde/rosso, lunghezza d’onda di picco). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Solo sensori senza bundle (DAQ-A).** Conteggi spettrali grezzi del sensore — *non* irradianza. Vedi sotto. |

`process()` esegue questa esportazione come una delle sue fasi. **Non** richiede immagini:
un sensore di luce utilizzato da solo costituisce un flusso di lavoro a tutti gli effetti, e un progetto di questo tipo non presenta alcuna
immagine per definizione.

**Le registrazioni DAQ-A vengono esportate come conteggi grezzi.** La famiglia DAQ-A è antecedente al sistema di bundle per serie
e non ha alcun bundle da recuperare — viene invece calibrato sul campo rispetto a un
bersaglio di riflettanza, motivo per cui non ne ha mai avuto bisogno. Tali registrazioni vengono esportate
con un prefisso `_raw` anziché `_calibrated`: un nome file diverso anziché un flag
all’interno del file, poiché l’identificativo deve rimanere intatto anche se inviato via e-mail come semplice nome. L’
intestazione `.csv` indica `raw spectral sensor counts (NOT irradiance)` e avverte che i
valori sono comparabili **all’interno** del file — esattamente lo scopo per cui la calibrazione basata su target
li utilizza — e non tra sensori diversi. Le colonne fotometriche dipendenti dalla potenza (potenza totale,
lux fotopico/scotopico, PPFD) restituiscono **NULL** anziché essere integrate dai conteggi.

Un DAQ-U / DAQ-M / DAQ-E il cui pacchetto semplicemente non è stato possibile recuperare viene comunque **saltato**,
non scritto in formato grezzo: in quel caso il pacchetto esiste e &quot;riconnettersi e rielaborare&quot; è un consiglio valido.

Le registrazioni legacy **v1.01 / v1.02** (un DAQ-A-SD le scrive) non riportano un&#x27;epoca per ogni lettura,
ma solo l’ora di scrittura del file. Il matcher immagine↔downwelling continua a rifiutarle — abbinare un
frame a un’ora di scrittura sarebbe un errore non visibile — ma l’esportatore le legge, e il
CSV stampa `clock=daq_created_on` in modo che il prodotto indichi su quale orologio si trova.

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
calibrazione in archivio) viene segnalata sotto `skipped` **con la motivazione**. Non viene mai
scritta come file &quot;calibrato&quot; contenente conteggi grezzi — connettersi a Internet ed
eseguire nuovamente l’operazione, dopodiché l’esportazione verrà completata.

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

Al termine, `process()` recupera `GET /api/processing-summary` e allega il corpo come `result["summary"]`. Il recupero avviene secondo il principio del &quot;best effort&quot; e non blocca mai un risposta positiva — se il riepilogo non è disponibile, `process()` ricade nella forma semplice `{"status": "complete", "async": False}`. Ogni voce in `summary["hints"]` — frasi complete con la correzione suggerita, ad esempio il motivo per cui un&#x27;esecuzione ha prodotto un output pari a zero — viene anch’essa riemessa come Python `UserWarning`, quindi le esecuzioni con output pari a zero sono autodiagnostiche anche se non si ispeziona mai il dizionario:

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
| `images_in_groups` | Immagini sorgente in quei gruppi. |
| `targets_found` | Target di riflettanza rilevati. |
| `images_calibrated` | Immagini calibrate durante l’esecuzione. |
| `exported_files` | **File dei prodotti immagine generati dalla sessione.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Registrazioni del sensore di luce, conteggiate separatamente di proposito — provengono da una fase diversa ed esistono per le sessioni prive di immagini, quindi includerle farebbe sembrare che una sessione di sola acquisizione dati (DAQ) abbia esportato immagini. |

Accanto a esse: `summary["output_dirs"]` (ogni directory in cui è stata effettuata la scrittura),
`summary["light_sensor_export"]`, `summary["stopped"]` (vero quando l’utente ha interrotto l’
esecuzione, in modo che i conteggi parziali non vengano interpretati come un’esecuzione completata che ha prodotto meno del previsto) e
`summary["groups"]` (la suddivisione per gruppo).

`exported_files` viene registrato dalla pipeline **mentre scrive**, non scansionato dagli
oggetti immagine del progetto in un secondo momento. Le strategie parallele e GPU creano i propri oggetti immagine
(nei sottoprocessi dei worker per i percorsi GPU), quindi la vecchia scansione riportava
`0 file(s) written` per ogni esecuzione di questo tipo ed emetteva poi l’indicazione di esportazioni pari a zero — su esecuzioni
in cui tutto aveva funzionato. Se si crea uno script basato su questo numero, un’esecuzione parallela corretta ora
riporta un conteggio diverso da zero.

I salti del sensore di luce riportano il motivo effettivamente rilevato dal lettore per ciascun file — uno
schema illeggibile, un bundle mancante, un errore di scrittura — **deduplicati**, quindi venti file
saltati per un’unica causa vengono considerati come un’unica causa anziché come venti ripetizioni della stessa.

> **`process()` non viene generato quando un’esecuzione non produce immagini.** Questo è l’unico punto in cui SDK e
> CLI differiscono deliberatamente: `chloros-cli process` considera «i prodotti sono stati richiesti, nessuno è stato
> scritto&quot; come un errore e termina con un codice di uscita diverso da zero, mentre SDK termina normalmente e segnala la
> condizione tramite `summary` / suggerimenti. Se la tua pipeline dovesse interrompersi in caso di esecuzione vuota, controllala
> tu stesso — verifica `summary` (o conta i file nella cartella del progetto) anziché fare affidamento sull’
> assenza di un’eccezione. Le cause più comuni sono una cartella di input non riconosciuta come
> acquisizione e prodotti saltati in quanto non applicabili alle telecamere presenti (ad es. radiance proveniente esclusivamente da telecamere RGB
>).

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

Il livello di esportazione multispettrale LATTICE (M3C/M3M) della pipeline `process` — `reflectance` (impostazione predefinita), `radiance`, `sensor-response` o `all` (ogni modalità applicabile per immagine) — corrisponde all’impostazione di elaborazione **&quot;Uscita radiometrica&quot;** del progetto. `configure()` dispone di una parola chiave dedicata:

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

La soluzione alternativa avanzata — scrivere la chiave `"Radiometric output"` del progetto tramite `custom_settings` — funziona ancora, ma ricordate che sostituisce l’intero blocco delle impostazioni (vedere l’avviso qui sotto):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (l’impostazione predefinita) divide la radianza della telecamera per il **flusso discendente DAQ con timestamp corrispondente**, risolto automaticamente da un `.daq` (DAQ-U/M/E) registrato**o da un `.csv` nativo DAQ-M**presente insieme alle immagini; qualsiasi pacchetto di calibrazione per singola telecamera o DAQ mancante a livello locale viene**recuperato automaticamente da AWS** al primo utilizzo. L’CLI espone questa funzionalità come opzioni di attivazione/disattivazione per tipo di prodotto su `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **sostituisce** l’intero blocco delle impostazioni calcolate (per come è progettato, ignora le altre parole chiave e la convalida di `configure()`). Quando lo si utilizza, includere tutte le chiavi `Project Settings` di interesse, come nell’esempio sopra riportato.

---

## Smart-Connect per telecamere LATTICE

Sessioni backend persistenti per hardware in tempo reale. Gli stessi endpoint utilizzati dall’interfaccia grafica, quindi il comportamento è identico su SDK / CLI / GUI.

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
| `read_nodes(names, enum_names=(), timeout=30.0)` | Legge i nodi GenICam; restituisce `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Scrive i nodi con il nome descrittivo (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Acquisisce un **singolo** fotogramma. Restituisce una lista composta da un solo elemento contenente i dizionari dei metadati del fotogramma. (L’acquisizione in raffica/multi-fotogramma è stata rimossa — chiamare `capture()` in un ciclo se è necessaria una serie.) |
| `disconnect()` | Rilascia dal pool. Operazione nulla se ci si è collegati a una sessione già aperta. |

`capture()` controlli di esportazione (stesso modello dell’array + GUI):

- `processing` / `levels` — `processing="all"` salva tutti i tipi di esportazione applicabili; `levels=["raw","radiance"]` salva solo quelli (sovrascrive `processing`). Omettete entrambi per utilizzare l’impostazione predefinita del backend.
- `force_daq=True` — salva la lettura DAQ/DLS assegnata come sidecar `.daq` anche in caso di acquisizione solo raw, in modo che il fotogramma possa essere rielaborato in riflettanza/indice in un secondo momento. Non ha alcun effetto se non è collegato alcun DAQ.

### Matrice sincronizzata — `ArraySession` (Smart-Prep)

`connect_array` è **il punto di ingresso consigliato** per le configurazioni multicamera. Esegue in background l’intero flusso smart-prep della GUI:

1. **Analisi di rete** (`/api/camera/array/recommend`) — individua la dimensione massima del fotogramma che rientra nel livello sim-emit senza perdere fotogrammi.
2. **Selezione automatica del livello** — `sim-capture-sim-emit` se la linea di trasmissione è in grado di gestirlo; altrimenti `sim-capture-ftd-stagger` o `slip-emit-and-capture`.
3. **Ridimensionamento automaticoriduzione**— riduce silenziosamente la dimensione dei fotogrammi / aumenta il binning quando la linea non è in grado di sostenere la risoluzione richiesta.**Questa misura di sicurezza non copre la sovrasottoscrizione aggregata**: un numero eccessivo di telecamere per la linea non può essere risolto riducendo i fotogrammi — vedi [Sovrasottoscrizione](#over-subscription-the-per-cam-floor).
4. **PTP abilitato** per impostazione predefinita — i timestamp tra le telecamere sono comparabili con una precisione nell’ordine dei microsecondi.
5. **Selezione automatica del formato pixel per ogni telecamera** — telecamere RGB → `BayerRG8`, multispec → `BayerRG12`.
6. **Inizializzazione AE** — acquisisce lo stato AE corrente di ciascuna telecamera in modo che la connessione non resetti l’esposizione durante il volo.
7. **Configurazione trigger GPIO** — `connect_array` attiva tutte le telecamere (`TriggerMode=On`, `TriggerSource=Line2`) in modo che l’impulso del master comandi gli slave tramite il cavo M8. Questo passaggio è valido solo per gli array: una singola telecamera aperta con `LatticeCamera` funziona invece in modalità autonoma.

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
- `"sim-capture-sim-emit"` — simultaneità vera (tutte le telecamere scattano sullo stesso fronte di clock).
- `"sim-capture-ftd-stagger"` — sfalsamento flessibile nel dominio del tempo (le cam emettono in momenti leggermente sfalsati in modo che i pacchetti vengano serializzati sulla linea).
- `"slip-emit-and-capture"` — acquisizione sequenziale per singola cam (nessuna sincronizzazione temporale; unica opzione quando nessuna dimensione di frame è compatibile con la modalità simultanea).

`wire_ceiling_mbps` sovrascrive il **budget di banda sostenuto dall’host** in MB/s — l’unico
valore da cui dipende l’intera allocazione dell’array. Lasciarlo su `None` per utilizzare il valore rilevato automaticamente
. Ridurlo quando l’array segnala frame corrotti da GVSP: il valore automatico è derivato
dalla velocità di collegamento dichiarata dalla scheda di rete, che sovrastima gli adattatori USB, le linee PCIe sottodimensionate e
i fabric condivisi molto trafficati — e tale sovrastima si manifesta sotto forma di frame danneggiati anziché come un
collegamento visibilmente lento. Il valore viene salvato nel blocco di acquisizione dell’array del progetto, quindi una
riapertura o un successivo `connect_array` lo ripristina come qualsiasi altra impostazione dell’array.
Vedi [Stato di salute dell’array](#array-health--which-subsystem-is-losing-frames).

#### Sovrasottoscrizione (il limite minimo per telecamera)

Il pacing Sim-emit assegna a ciascuna telecamera una quota del budget di banda a prova di collisione, con un limite minimo di **8 MB/s per telecamera**(`per_cam_floor_bps`). Una volta che `N × floor` supera il limite massimo di sicurezza contro le collisioni, l’array**sovrascrive la banda**— la modalità di errore è la perdita di pacchetti GVSP, non una frequenza dei fotogrammi inferiore — e non esiste alcuna soluzione relativa alla dimensione dei fotogrammi:**il binning e la ROI riducono i byte per fotogramma, non i byte regolarizzati al secondo**che il controllo aggregato mette a confronto. Limiti pratici a piena risoluzione su un host da 1 GbE:**6 telecamere a 1500 MTU, 9 con frame jumbo** (`max_cams_collision_safe` nella risposta all’analisi riporta il limite massimo per la vostra linea). Soluzioni: meno telecamere, frame jumbo end-to-end, oppure una scheda di rete più veloce.

- Le risposte `analyze_array_network()` e `/api/camera/array/connect` contengono `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` e `per_cam_floor_bps`. Quando `oversubscribed` è vero, la proiezione **azzeri i campi fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) anziché segnalare una velocità di elaborazione apparentemente bassa ma-funzionante.
- `POST /api/camera/array/connect` accetta un parametro del corpo `pin_resolution` (**solo HTTP — non un kwarg SDK**; `connect_array` non lo espone). Il pinning rimuove la rete di sicurezza del binning walk-down, quindi una connessione sovrascritta con `pin_resolution` impostato viene**rifiutata categoricamente** con un errore che indica ogni possibile soluzione. Senza il pinning, la connessione procede con il walk-down ma avvisa che la riduzione non può azzerare l’aggregato.
- Scappatoia da banco: imposta `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` nell’ambiente del backend per declassare il rifiuto a un avviso evidente — ci si connette comunque e si accetta la perdita di pacchetti.

#### Integrità dell’array — quale sottosistema sta perdendo frame

`GET /api/camera/array/<array_id>/capability` contiene un blocco `health` attivo su un
array connesso, rivalutato in una finestra **10 secondi**. Suddivide la perdita di frame
nelle due cause che richiedono soluzioni opposte, invece di un unico tasso di &quot;incompletezza&quot; che
non ne identifica nessuna:

| Campo | Cosa significa | Quale sottosistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriale) | Il fotogramma **è arrivato ma era strutturalmente errato**— perdita di pacchetti GVSP. |**Rete**: larghezza di banda disponibile, pacing, anello RX della scheda di rete, MTU |
| `never_arrived_rate_pct` (per seriale) | Il frame **non è mai arrivato**— la telecamera non ha scattato, oppure non è stato trasmesso nulla. |**Trigger / sincronizzazione**: cavo M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Tasso di errore tasso di ogni telecamera. | — |
| `per_cam_rate_pct` | Tasso combinato di incompletezza per telecamera (entrambe le cause insieme). | — |
| `stable_for_seconds` | Per quanto tempo ogni telecamera è rimasta al di sotto dello 0,01%. | — |

Insieme a `health`, lo stesso record riporta il numero da cui dipende l’intera allocazione:

| Campo | Significato |
| --- | --- |
| `wire_ceiling_mbps` | Il budget di banda effettivo dell’host, in MB/s. |
| `wire_ceiling_source` | Da dove proviene quel numero, in parole — ad es. `USB-capped 200 MB/s (was theoretical 1062; …)` o `user override 120 MB/s (auto said 200)`. |
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

**Lettura:** valore diverso da zero per `gvsp_corrupt_rate_pct` con `never_arrived_rate_pct` a 0 significa che
l&#x27;attivazione e la sincronizzazione del cavo sono perfette e che il 100 % della perdita è da attribuire al percorso di rete — ridurre
`wire_ceiling_mbps` e ricollegarsi. Il modello inverso indica invece un problema con il cavo di sincronizzazione o la
linea di trigger.

> **`target_fps` non è il fattore determinante per i frame corrotti.** Il pacing di GevSCPD viene impostato una sola volta al
> momento della connessione, quindi abbassare la frequenza di trigger modifica il ciclo di lavoro e non la
> velocità di trasmissione in burst simultanea. Una riduzione misurata della richiesta del 5× non ha prodotto alcun miglioramento, mentre
> l’abbassamento del limite massimo della linea da 240 a 200 MB/s ha portato lo stesso sistema dal 10,4 % di frame danneggiati allo
> 0,00 %.

> **La riduzione automatica a metà flusso non è disponibile sul firmware TRI032S.** Un array in esecuzione non può
> risolvere questo problema autonomamente; scollegarlo e ricollegarlo in modo che il selettore del tempo di connessione ripiani in base
> al nuovo limite massimo.

Un **adattatore Ethernet USB è limitato a 200 MB/s** dalla sonda indipendentemente dalla sua
denominazione: la tabella di efficienza che trasforma una velocità di collegamento in un valore sostenuto è
derivata dallo standard PCIe, mentre una scheda di rete USB comunica la propria velocità di collegamento Ethernet pur essendo limitata dal
bus USB e dal relativo driver. Il limite è assoluto, non una frazione: un adattatore USB da 1 GbE
raggiunge circa 80 MB/s e non ne risente.

#### `ArraySession` Metodi

| Metodo | Descrizione |
| --- | --- |
| `status(timeout=10.0)` | `{fps, ptp, frame_count, last_error, …}` in tempo reale. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Un gruppo di acquisizione sincronizzato. Restituisce un `CaptureResult` (elenco di dizionari di frame + `.skipped`). Controlli di esportazione di seguito. |
| `capture(..., smart=True)` | **Acquisizione intelligente** — attende che l’AE si stabilizzi su tutte le telecamere, quindi attiva l’acquisizione. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Acquisizione più veloce: solo dati grezzi + la lettura DAQ assegnata (+ l’indice combinato libero). Rispecchia il pulsante “Acquisizione più veloce” dell’interfaccia grafica. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Singola / Continua / A intervalli in un unico ciclo limitato. Restituisce `list[CaptureResult]`.**Richiede `count` e/o `duration_s`** affinché si interrompa (SDK non supporta Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Avvia la registrazione della vista in tempo reale dell&#x27;indice combinato su video/GIF → `RecorderHandle`. Un registratore composito per array. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Avvia una raffica raw Bayer ad alto fps → `RecorderHandle`. Rielabora offline con `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Rielaborare offline una sequenza raw salvata in video calibrati. Rimane in attesa fino al completamento (`wait=True`) e restituisce `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Interroga un processo di compilazione offline: `{running, result, error, burst_dir}`. |
| `disconnect()` | Rilascia l’intero array. |

`capture()` controlli di esportazione (stesso endpoint utilizzato dalla GUI/CLI):

- `processing` / `levels` — `processing="all"` (oppure `levels=["raw","radiance",…]`) salva ogni tipo di esportazione applicabile per ciascuna telecamera; un singolo valore `processing` salva solo quel livello.
- `aligned=True` — applica il warp a ogni elementoesportazione non raw a [profilo di allineamento](#array-alignment) (co-registrato); i dati grezzi rimangono non allineati ma riportano la trasformazione nei metadati. Si ricade sull’allineamento non allineato (con un avviso visualizzato nell’`alignment` del risultato) se l’array non ha un profilo.
- `render_index=False` — salta la sovrapposizione dell’indice di vegetazione per singola telecamera; per impostazione predefinita la rende dove configurata.
- `force_daq=True` — salva la lettura DAQ/DLS assegnata come sidecar `.daq` anche quando nessun livello selezionato ne ha bisogno.

**Compressione TIFF (regolatore disponibile solo in HTTP):**`ArraySession.capture()` non invia alcun `compression` key, quindi si applica l’impostazione predefinita del backend — `POST /api/camera/array/capture` legge un parametro del corpo `compression`, `"deflate"` per impostazione predefinita (zlib L1 senza perdita di dati + predittore orizzontale, ~4,1 MB per fotogramma a piena risoluzione). `"none"` scrive in formato non compresso (~6,3 MB/fotogramma) con una**velocità di scrittura ~5 volte superiore** — entrambe sono senza perdita di dati e vengono lette in modo identico in fase di importazione. SDK non espone alcun kwarg per questa opzione; la via d’uscita è `chloros-cli lattice array-capture --compression none` o HTTP in formato raw. Anche DEFLATE detiene il GIL di Python, quindi le scritture compresse non vengono parallelizzate tra i thread di scrittura per ciascuna telecamera — l’acquisizione sostenuta a piena risoluzione con 8 telecamere alla frequenza del sensore richiede `compression: "none"`. Dettagli: [Riferimento CLI → acquisizione array](cli-reference.md).**Sovrascritture di esportazione per singolo membro (solo HTTP):**lo stesso endpoint accetta anche `exclude_serials` (elenco — rimuove i membri dall’insieme salvato; l’array si attiva comunque come un unico gruppo sincronizzato e i membri esclusi vengono restituiti in `excluded`), `serial_levels` (sovrascritture a livello di singola telecamera `{serial: [level tokens]}`) e `serial_index` (sovrascritture dell’overlay dell’indice per singola telecamera `{serial: bool}`). Si tratta di parametri del corpo in parità con l’interfaccia grafica e**non sono ancora kwargs SDK**; i membri assenti dalle mappe ricorrono ai valori `levels` / `render_index` validi per l’intero array.

##### Ispezione delle cam saltate — `CaptureResult.skipped`

`ArraySession.capture()` restituisce un `CaptureResult`, che è una sottoclasse di `list`: iterarlo, indicizzarlo, `len()`arlo — ogni modello esistente continua a funzionare. Il nuovo codice può ispezionare l’attributo `.skipped` per vedere quali cam sono state escluse e perché. Il caso più comune è quello delle cam RGB in un array a filtri misti quando si richiede `processing="radiance"` o `"reflectance"` — la radianza per pixel Bayer non ha senso per un sensore a banda larga, quindi il backend salta quelle telecamere piuttosto che produrre dati privi di significato.

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

I token di motivazione seguono lo schema `<level>-not-applicable-to-rgb-cam` (una voce per ogni livello saltato, ciascuna contenente `level`). I salti specifici per la riflettanza sono `reflectance-skipped-no-fresh-dls` (nessuna nuova lettura della radiazione discendente disponibile), `reflectance-skipped-bound-daq-unavailable (…)` (impossibile raggiungere il DAQ associato) e `dls-uncalibrated-band-<nm>` — la banda si trova in gran parte al di fuori dell’intervallo radiometricamente calibrato del sensore di luce del DAQ (~374–974 nm), quindi la divisione assoluta della riflettanza basata sul DAQ viene rifiutata e il fotogramma ricade nettamente sulla risposta del sensore. Tra gli SKU in commercio, solo l’F988 lo attiva; il percorso supportato da quella fotocamera è il flusso di lavoro con pannello di riflettanza.

Livelli `processing`:

| Livello | Uscita |
| --- | --- |
| `"raw"` | Bayer monocanale (telecamere monocromatiche: la singola banda) direttamente dal sensore. |
| `"debayered"` *(impostazione predefinita SDK)* | BGR a 3 canali tramite demosaico bilineare (telecamere monocromatiche: scala di grigi a 1 canale). |
| `"radiance"` | float32 W/m²/sr/nm tramite la catena radiometrica completa. Solo multispettrale — le telecamere RGB vengono ignorate. |
| `"reflectance"` | uint16 0..32768 (compatibile con Pix4D); richiede un abbinamento DAQ in tempo reale per il riferimento assoluto. Solo multispettrale. |
| `"display"` | Catena completa corrispondente all’anteprima dell’interfaccia grafica (CCM + WB + gamma secondo il profilo della telecamera). |
| `"all"` | **Un file per ogni livello applicabile** per ciascuna telecamera (corrispondente all’impostazione predefinita “Cattura tutto” dell’interfaccia grafica / CLI). Il file `CaptureResult` restituito contiene quindi un dizionario di fotogrammi per ogni `(cam, level)`, con il livello indicato in ciascun dizionario; i livelli non applicabili compaiono in `.skipped`. La lettura DAQ utilizzata per qualsiasi fotogramma di riflettanza viene salvata come sidecar `.daq`. |

> **Nota — l’impostazione predefinita differisce da quella di CLI.** `ArraySession.capture()` assume come valore predefinito `processing="debayered"`; il comando `chloros-cli lattice array-capture` ha come impostazione predefinita `processing="all"`. Passare esplicitamente `processing="all"` da SDK per rispecchiare il salvataggio multilivello di CLI/GUI.

### Modalità di acquisizione e registratori

La superficie dell’array rispecchia il pannello di acquisizione della GUI: modalità otturatore Singolo / Continuo / Intervallo / Più veloce, oltre a due registratori (video composito in diretta e raffica raw → rielaborazione offline).

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

- **`capture_repeated`**è il ciclo Continuo/a intervalli di SDK. Poiché non esiste un `Ctrl+C` per interromperlo da uno script, è**devi** passare `count` e/o `duration_s` (si interrompe quando viene raggiunto uno dei due). `interval_s` viene misurato dall’inizio di ogni passaggio (in linea con la GUI). I restanti kwargs vengono passati direttamente a `capture()`.
- **`record`** è *di livello di monitoraggio*: acquisisce il composito dell’indice combinato in tempo reale così come viene visualizzato, quindi il flusso combinato deve essere aperto affinché i fotogrammi possano essere acquisiti. È consentito un solo registratore di composito per array (se ne è già uno in esecuzione, viene generato un errore).
- **`burst` → `build_video`** è *di livello analitico*: `burst` scrive i fotogrammi grezzi + un manifesto per fotogramma + un `.daq` per ogni lettura DLS distinta sotto `<output>/bursts/<base>/` alla velocità massima del ciclo di acquisizione (senza concatenamento, senza strumento EXIF, nessuna visualizzazione in tempo reale). `build_video` allinea temporalmente ogni fotogramma al più vicino `.daq` e riesegue la catena di radianza/riflettanza/indice della pipeline di importazione. `products` è un elenco di `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (impostazione predefinita: l’indice combinato). `burst().stop()` avvia inoltre automaticamente un, restituita come `build_job` nel risultato di arresto.

#### `RecorderHandle`

Restituito da `ArraySession.record()` e `ArraySession.burst()`. Utilizzarlo come gestore di contesto per l’arresto automatico all’uscita dall’ambito, oppure gestirlo manualmente.

| Membro | Descrizione |
| --- | --- |
| `job_id` | ID del processo di backend (str). |
| `kind` | `"composite"` (da `record`) oppure `"raw"` (da `burst`). |
| `start_stats` | Il dizionario restituito dalla chiamata a `start`. |
| `result` | `None` durante l&#x27;esecuzione; il dizionario finale dei risultati di arresto una volta terminata l&#x27;esecuzione. |
| `stats(timeout=10.0)` | Statistiche in tempo reale del processo (frame scritti, fps effettivi, tempo trascorso). |
| `stop(timeout=60.0)` | Arresta il registratore; restituisce e memorizza nella cache il risultato finale. Idempotente (una seconda chiamata restituisce il risultato memorizzato nella cache). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Collegamento a un array già connesso — `attach_array`

Se l’array è già attivo (è stato aperto dall’interfaccia grafica o una precedente sessione SDK ha chiamato `connect_array`), utilizzare `attach_array` per ottenere un handle ad esso invece di riconnettersi. `connect_array` genera sempre un errore del tipo &quot;La telecamera  è<sn> già nell’array <id>&quot; in quella situazione, poiché l’invio di un POST a `/array/connect` per un membrodel pool non è idempotente; `attach_array` legge `/api/camera/array/list` ed effettua la corrispondenza in base all’array_id o ai seriali.

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

Modello: gli script SDK che condividono l’ambiente con la GUI desktop dovrebbero provare prima `attach_array` e ricorrere a `connect_array` se non c’è ancora alcun array nel pool.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Importante — l’uscita da context-manager provoca la disconnessione.**`ArraySession.disconnect()` invia sempre un POST a `/array/disconnect`; non esiste una protezione &quot;attaccato ma non di proprietà&quot; come nel caso di `CameraSession` / `DAQSensorSession`. Se si sta condividendo l&#x27;ambiente con la GUI e non si desidera smantellare l&#x27;array all&#x27;uscita dall&#x27;ambito,**non utilizzare il blocco `with`** — conservate l’handle in una variabile normale e saltate l’uso esplicito di `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Strumento di analisi della rete

Utile prima di aprire l’array — verifica se le impostazioni proposte sono adeguate:

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

`status` è uno tra `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (altrimenti `error`). `auto_capped_fps` indica che la risoluzione richiesta si adatta all’anello RX solo con una frequenza di trigger limitata — mantenere la risoluzione e passare da `target_fps=result["recommended"]["recommended_target_fps"]` a `connect_array` (vedere [Esempio 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Come interpretare la proiezione** (stesso modello del pannello «Impostazioni array» dell’interfaccia grafica):

- **Il burst (`frame_bytes_total`) viene sommato per ciascuna telecamera nel formato pixel reale di ciascuna telecamera.**Le telecamere mono**M3M**trasmettono Mono12 (2 B/px) indipendentemente dal valore `pixel_format` specificato; pertanto, un fotogramma a piena risoluzione con 4 telecamere occupa**circa 25 MB** con tre telecamere mono, non i circa 12,6 MB che si otterrebbero ipotizzando un formato interamente a 8 bit. Il backend determina il formato di ciascuna telecamera in base al suo modello.
- **L’ammissione (`burst_fits_nic_ring`) tiene conto dello svuotamento**, non della differenza tra burst completo e anello: sim-emit è adatto quando l’host svuota l’anello RX più velocemente di quanto le telecamere lo riempiano. Un host a 10G + telecamere a 1 GbE**ammettela** piena risoluzione anche quando il burst supera l’anello; un host da 1 GbE va in blocco (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` rappresenta un limite massimo conservativo per il recupero seriale** — `max(readout+emit, N×emit)` con emissione per singola cam limitata al collegamento 1 GbE, indipendentemente dall’esposizione. Ad es. ~2,8 fps per un array a 12 bit a piena risoluzione di 4- telecamere a piena risoluzione con array a 12 bit (corrisponde ai ~2,7–3,0 misurati in fase di esecuzione). Modello completo: [CLI Riferimento → Modello fps e burst dell’array](cli-reference.md#array-fps--burst-model).
- **L’over-subscription (`oversubscribed: true`) indica che il limite minimo N × per telecamera supera il limite massimo di sicurezza contro le collisioni** — i campi fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) riportano 0, e il ridimensionamento automatico o il binning non possono risolvere il problema (riducono i byte per frame, non i byte regolarizzati al secondo). Le soluzioni sono: un numero inferiore di telecamere, frame jumbo o una scheda di rete più veloce; `max_cams_collision_safe` segnala il limite massimo (6 telecamere a piena risoluzione su 1 GbE a 1500 MTU, 9 con jumbo). La risposta contiene anche `aggregate_demand_bps`, `collision_safe_ceiling_bps` e `per_cam_floor_bps` (8 MB/s). Vedi [Over-Subscription](#over-subscription-the-per-cam-floor).

### Rilevamento e Elenco

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Gli array LATTICE eseguono l’AE in modo continuo in background non appena vengono collegati, ma una scena appena inquadrata richiede un attimo per convergere. **Smart-capture** è la soluzione pronta all’uso: interroga l’esposizione di ciascuna telecamera, attende che l’array sia stabile su un’intera finestra, quindi avvia l’acquisizione. È equivalente all’interfaccia grafica: il pulsante di acquisizione “smart” dell’app desktop richiama lo stesso endpoint di backend.

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

Quando si utilizza `ChlorosProject` (sezione successiva), sono disponibili ulteriori opzioni di regolazione:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

La politica smart-AE è conservativa per impostazione predefinita. Restringete `exposure_tolerance_pct` per lavori radiometrici meticolosi; allargatela per scene in rapida evoluzione in cui vi basta un risultato “sufficientemente preciso”.

---

## Sessioni dei sensori DAQ

Pool di backend persistente per sensori spettrali (DAQ-U su USB, DAQ-M su BLE, DAQ-E su Ethernet). Rispecchia la superficie della fotocamera: rilevamento intelligente, riutilizzo del pool, collegamento idempotente.

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
| `status(timeout=10.0)` | Riepilogo delle voci del pool (stato di streaming/registrazione, intervallo di lunghezza d’onda, sha di calibrazione, tempo di integrazione, frame_avg, stato AE). |
| `latest(n=1, timeout=10.0)` | Restituisce fino a N frame dello spettro più recenti. |
| `stream_start()` / `stream_stop()` | Riprende / mette in pausa lo streaming (l&#x27;handle rimane aperto). |
| `record_start(output_dir=None, device_name=None)` | Avvia la registrazione di un file .daq. Restituisce il percorso del file. Non è supportato per DAQ-U/M senza un pacchetto di calibrazione AWS (DAQ-E è escluso). |
| `record_stop()` | Interrompe la registrazione. Restituisce `{path, rows}`. |
| `disconnect()` | Rilascia dal pool. Operazione nulla per handle collegati ma non di proprietà. |

> **I profili di correzione del livello massimo (`cap_id`) non sono un controllo SDK.** `connect_daq_sensor()` / `DAQSensorSession` non espongono alcun parametro `cap_id` né alcun metodo `set_cap`. Selezionare un profilo di correzione del limite della flotta tramite CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) oppure tramite i percorsi `/api/daq` e HTTP del backend (`/api/daq/connect` e `/api/daq/<id>/cap-id` accettano `cap_id`).

### Rilevamento — ricerca di un indirizzo a cui connettersi

`discover_daq_sensors()` esegue una scansione su USB / BLE / ETH alla ricerca di sensori che *potresti* aprire. È la controparte DAQ di `discover_lattice_cameras()`, e l’unico modo per ottenere il **MAC BLE del DAQ-M** — un DAQ-E ha un nome host e un DAQ-U una porta COM, ma il MAC non è né stampato sul dispositivo né elencato dal sistema operativo.

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
| `display` | Etichetta leggibile dall’utente. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, oppure `None` per una porta che la scansione non riesce a identificare (gli adattatori seriali USB sono indistinguibili senza una sonda, quindi gli elementi sconosciuti vengono visualizzati anziché nascosti). |
| `extra` | Dettagli per tipo di trasporto (nome pubblicizzato BLE, produttore USB, IP/FW/… DAQ-E). I valori vuoti vengono omessi. |

| Parametro | Predefinito | Descrizione |
| --- | --- | --- |
| `transports` | tutti e tre | Sequenza (o stringa CSV) che limita la scansione. Vale la pena specificarla quando si sa cosa si vuole — il BLE è la parte più lenta. |
| `scan_timeout` | 5 | Finestra di scansione per ogni trasporto in secondi; il backend limita il valore a un intervallo compreso tra 1 e 20. |
| `timeout` | 60,0 | Limite massimo per l’intera chiamata (come altrove in HTTP). |
| `auto_start_backend` | `True` | Avvia un backend locale se non ce n’è uno in esecuzione. Non viene mai avviato per un `backend_url` remoto. |

> **I sensori già aperti nel pool non vengono visualizzati.** Una periferica BLE connessa smette di trasmettere e una porta COM aperta non può essere rilevata, quindi la ricerca elenca ciò che è *disponibile per la connessione*. È normale ottenere un risultato vuoto subito dopo aver collegato qualcosa: usa `list_daq_sensors()` per ciò che hai già in mano. I trasporti la cui scansione non può essere eseguita (bleak / zeroconf non installati) vengono ignorati anziché generare un errore, quindi una macchina senza Bluetooth riceve comunque le risposte relative a USB ed ETH.

### Elenco

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Co-Tenancy con GUI / CLI

Se la GUI ha già un sensore aperto, la chiamata a `connect_daq_sensor(port="COM3")` da Python restituisce un handle contrassegnato come `already_connected=True`. L’`disconnect()` della sessione diventa quindi un’operazione nulla, in modo che lo script SDK non rimuova il sensore dalla GUI all’uscita dall’oscilloscopio.

### Classi hardware dirette (senza backend)

`daq_sdk` viene riesportato da `chloros_sdk`, quindi è possibile pilotare i sensori end-to-end all’interno del processo senza il backend:

> **Disponibilità:**`daq_sdk` è incluso nell’installazione desktop di Chloros,**non** con il pacchetto PyPI — `pip install chloros-sdk` fornisce `lattice_sdk` ma esclude `chloros_sdk.DAQ_AVAILABLE == False`. Verificate questo flag prima di utilizzare queste classi; su un host che utilizza solo pip, utilizzate invece il sensore tramite [`connect_daq_sensor()`](#daq-sensor-sessions), che non richiede librerie di trasporto locali.

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

Preferire il percorso smart-connect (`connect_daq_sensor`) quando si desidera la proprietà condivisa con la GUI; utilizzare le classi dirette per gli script headless che possiedono il sensore in modo esclusivo.

---

## Automazione del progetto — `ChlorosProject`

Un progetto Chloros salvato è una cartella contenente `cameras.json` + `sensors.json` + `project.json`. `open_project` carica il manifesto, mentre `connect_all` mette online tutti i dispositivi salvati con le relative impostazioni salvate — lo stesso stato hardware che si otterrebbe con l’interfaccia grafica.

### Esempio minimale

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
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Individua e connette ogni dispositivo salvato. Restituisce un rapporto di connessione per ogni classe. Utilizza un backend in esecuzione se ne è presente uno in ascolto su `127.0.0.1:5000`; in caso contrario, ricorre silenziosamente al controllo diretto (senza backend) controllo dei dispositivi `lattice_sdk` — non avvia mai un backend. |
| `disconnect_all()` | Chiude tutte le connessioni. |
| `capture_all(output_dir=".")` | Un fotogramma da ogni telecamera + array + spettro da ogni sensore. |
| `stream(camera, overlays=False, fps=10.0)` | Generatore che produce fotogrammi BGR `numpy` da una telecamera (o array) specificata. `overlays=False` è un ciclo di acquisizione diretto `lattice_sdk` (gli array producono dizionari `{serial: frame}`). `overlays=True` passa attraverso `ChlorosLocal.camera_stream()` → il feed Mfeed JPEG del backend, con il blocco `ui.overlay` salvato dalla webcam trasmesso come parametri di query. Richiede la modalità backend e una **fotocamera autonoma**: una fotocamera in modalità diretta genera l’errore `RuntimeError` (il backend non può acquisire una fotocamera di proprietà di questo processo) e un array genera l’errore `NotImplementedError` (sovrappone il composito per fotocamera — trasmette un membro in base al nome). Equivalente one-shot: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Esegue l’allineamento su ogni array attualmente connesso. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Esegue la pipeline di calibrazione/indicizzazione sulle immagini del progetto (racchiude `ChlorosLocal.process`; questi quattro sono gli **unici** kwargs accettati — `indices=` ecc. generano un&#x27;eccezione `TypeError`; imposta gli indici tramite `ChlorosLocal.configure()`). Costruisce in modo differito un `ChlorosLocal()`, che avvia automaticamente un backend. |

Attributi:
- `proj.cameras` — `Dict[str, CameraHandle]` con chiave basata su nome E numero di serie.
- `proj.arrays` — `Dict[str, ArrayHandle]` con chiavein base al nome E all’array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` con chiave basata sul nome E sullo slot_id.
- `proj.config` — `project.json["config"]`.

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
| `raw` | Bayer a 1 canale, nativo del sensore | Nessun demosaic. Le sovrapposizioni non sono disponibili a questo livello. |
| `debayered` | BGR a 3 canali (**predefinito**) | Demosaic bilineare. L’unico livello che funziona senza la modalità backend. |
| `radiance` | float32, W/m²/sr/nm | Catena radiometrica completa: demosaic + separazione 3×3 (multispec) + DSNU + flat-field + scala NIST, con esposizione × guadagno eliminati in modo che i valori siano assoluti. |
| `reflectance` | uint16, 32768 = 1,0 | Radianza divisa per l’irraggiamento discendente (ρ = π·L/E). Richiede una lettura DLS/DAQ — vedi la nota qui sotto. |
| `display` | 8 bit, simile a sRGB | Rendering equivalente alla GUI: CCM + bilanciamento del bianco + gamma tramite il profilo colore attivo della fotocamera. |

Qualsiasi valore diverso da `debayered` richiede la modalità backend; una fotocamera in modalità diretta genera
`NotImplementedError`. `reflectance` necessita di una lettura downwelling utilizzabile — l’endpoint del frame inserisce
automaticamente il DAQ raggruppato nello slot DLS della fotocamera, ma senza un DAQ associato la catena rifiuta l’
uscita di riflettanza e contrassegna chiaramente la demotivazione nei metadati restituiti, anziché restituire silenziosamente
un prodotto di qualità inferiore.

> **Scala DN della riflettanza — non codificarla in modo rigido.** La riflettanza LATTICE utilizza `32768` = ρ 1,0 e registra
> XMP `Chloros:PixelScale=32768`; la riflettanza Survey3 utilizza `65535` = ρ 1,0 e non contiene
> tag `Chloros:*`. Leggere il tag e dividere per esso. È definito nel dominio uint16, quindi rimane
> `32768` per ogni formato che ridimensiona (16bit TIFF, 8 bit PNG/JPG, percentuale a 32 bit) — normalizza
> prima il tipo di dati memorizzato riportandolo a uint16 (×257 da 8 bit, ×65535 da float). L’unica eccezione:
> un’acquisizione con sorgente a 8 bit salvata come TIFF a 8 bit viene *tagliata*, non ridimensionata, quindi non è presente alcuna scala che la descriva
> — Chloros omette completamente `PixelScale` e la tupla MicaSense in quel caso. Considerare un
> tag mancante in un file di riflettanza LATTICE come “scala non valida”, non come valore predefinito.

> **EXIF trasferito nell’esportazione.** `process()` copia il blocco GPS dell’acquisizione di origine
> **e il suo ExifIFD** su ogni prodotto, quindi le esportazioni contengono `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` e `CameraSerialNumber`, oltre al
> georeferenziazione. `FocalLength` è il valore da cui Pix4D calcola la distanza campionaria al suolo — senza di esso
> la ricostruzione ricade su una scala estremamente errata (in un caso misurato, un sito di 411 m
> è stato trasformato in uno di 47,8 km). Il file di copia non è volutamente `-all:all`: i tag strutturali di IFD0 compromettono
> l’output di LATTICE, mentre `ExifImageWidth`/`Height` sono esclusi perché descrivono l’acquisizione
> della sorgente piuttosto che il raster esportato.

Acquisizione-sottoflag della fase di acquisizione (si applicano ai livelli radiometrici — `radiance`, `reflectance`, `display`):

| Flag | Predefinito | Significato |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + separazione 3x3 + scala radiometrica NIST. |
| `apply_white_balance` | `True` | LUT di bilanciamento del bianco (WB). Compatibile con DLS quando un DAQ è associato alla fotocamera. |
| `apply_index` | `False` | Valutazione dell’indice di vegetazione. |
| `index_expression` | `None` | Formula di override. Se non vuota → abilita automaticamente l’indice. |
| `annotated` | `False` | Sovrapposizione delle decorazioni dell’interfaccia grafica (zebra/griglia/picchi). Non disponibile per `raw`. |

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
> `chloros_sdk.CapturePathMap` è `Dict[str, Union[str, List[str]]]`: un livello singolo
> `processing` assegna a ogni seriale un unico percorso, mentre uno a più livelli (`"all"`, o un
> elenco esplicito `levels`) fornisce l’**elenco ordinato** di tutti i prodotti salvati per quella
> fotocamera. Un composito combinato in tempo reale, se ne fosse in streaming, arriva sotto la chiave aggiuntiva
> `"combined"` piuttosto che sotto un numero di serie. Il codice che presuppone `str` va in errore con la
> forma a elenco senza che alcun controllore di tipi sollevi obiezioni — l’annotazione indicava `Dict[str, str]`
> per un certo periodo dopo il rilascio della forma a elenco, motivo per cui esiste l’alias. Normalizza
> quando desideri la forma piatta:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Allineamento degli array

`ArrayHandle` espone l’intera superficie di allineamento. I profili sono, per impostazione predefinita, validi solo per la sessione — chiamare esplicitamente `export_alignment()` per renderli persistenti.

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

`connect_all(align=...)` può allineare automaticamente ogni array al momento della connessione:

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

Quando si desidera azzerare la dipendenza dal backend (CI, robot headless, embedded), importare direttamente `lattice_sdk` e `daq_sdk` — entrambi vengono riesportati da `chloros_sdk`. Attenzione a `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` è presente nel pacchetto PyPI (ma richiede la presenza del runtime Arena SDK), mentre `daq_sdk` è fornito solo con l’installazione desktop.

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

Tre delle quattro impostazioni predefinite sono in modalità **free-run**: la telecamera espone in modo continuo e
`capture()` restituisce il fotogramma successivo. `triggered` costituisce l’eccezione: arma la
telecamera in attesa di un fronte d’impulso hardware sulla linea 2, quindi non acquisisce nulla finché non ne arriva uno.

| Preset | Trigger | Da usare quando |
| --- | --- | --- |
| `default` | free-run | uso generale |
| `high_speed` | funzionamento libero | 8 bit, limite a 60 fps, esposizione breve |
| `high_quality` | funzionamento libero | 12 bit, nessun limite di fps — la scelta abituale per le foto |
| `triggered` | **pronta, Linea 2** | la telecamera è collegata a un cavo di sincronizzazione M8 e viene attivata da un altro dispositivo |

Se si seleziona `triggered` (o si imposta manualmente `trigger_mode="On"`) senza che nulla
comandi la Linea 2, ogni `capture()` andrà in timeout — correttamente, poiché si è chiesto
alla fotocamera di attendere. Il messaggio SDK spiega questo comportamento quando si verifica; vedi
[SC_ERR_TIMEOUT durante l’acquisizione](#direct-hardware-backend-free).

> **Nota — I messaggi «GVSP probe» / `SC_ERR_TIMEOUT -1011` alla connessione non sono errori.**&gt; Al momento della connessione, SDK tenta di negoziare**jumbo frame** (pacchetti GVSP da 9000 byte) per ottenere una maggiore velocità di trasmissione. Su un collegamento NIC diretto punto-punto (ad es. un indirizzo `169.254.x.x` link-local) la rete solitamente non è in grado di trasportare jumbo frame, pertanto questa sonda va in timeout e registra righe del tipo:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Questa è la **soluzione di ripiego prevista**: l’SDK torna automaticamente ai pacchetti standard da 1500 byte e la telecamera continua a connettersi normalmente (le righe `[chunk-enable …]` che seguono fanno parte della normale sequenza di connessione). L’acquisizione continua a funzionare.
>
> È possibile saltare questa sonda, ma **non si tratta semplicemente di un silenziatore di log: disattiva i jumbo frame.** La telecamera risponde ai ping «Don&#x27;t-Fragment» solo fino a 1500 byte, indipendentemente dalla qualità della rete; pertanto, il test del ping da solo non è mai in grado di rilevare i jumbo; questa sonda è l’unica in grado di farlo. Se la si disabilita, la telecamera utilizzerà per sempre pacchetti standard da 1500 byte, su qualsiasi rete:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Ne vale la pena solo su una rete che *sai* non supportare i jumbo, dove si risparmia circa un secondo di tempo di connessione per ogni telecamera. Poiché si tratta di un compromesso concreto e non solo estetico, l’opzione SDK ora lo specifica quando la si utilizza:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Non modificarlo a meno che non ci sia un motivo valido.** Se lasciato abilitato, ogni connessione ricalibra la rete effettivamente disponibile: collegati a uno switch compatibile con i pacchetti jumbo e la connessione successiva rileverà automaticamente i pacchetti jumbo, senza bisogno di configurazioni né riavvii.
>
> Se *desideri* la velocità di trasmissione dei pacchetti jumbo, abilita i pacchetti jumbo end-to-end (MTU della scheda di rete 9000 + uno switch che li trasmette), oppure fissalo con `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` quando sai che il collegamento lo supporta — anche se è preferibile usare `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` per singolo comando piuttosto che impostarlo in modo permanente, poiché una dimensione fissa salta la sonda e impedisce l’adattamento alla rete a monte. **Ogni** dispositivo lungo il percorso deve supportare i pacchetti jumbo — compresi eventuali splitter o iniettori PoE, che sono la causa più comune per cui una configurazione altrimenti compatibile con i pacchetti jumbo non riesca a trasmetterli.

> **`SC_ERR_TIMEOUT -1011` durante `capture()` / `grab*()` è un problema diverso: in quel caso si tratta di un vero e proprio errore.**&gt; La nota sopra si riferisce esclusivamente a `-1011` registrato dalla**sonda connect-time**. Lo stesso errore generato da una**cattura** significa che la telecamera si è collegata correttamente ma non sta inviando alcuna immagine:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> L’indizio rivelatore è una telecamera il cui canale di *controllo* è integro — il rilevamento funziona, le impostazioni e le scritture `[chunk-enable …]` vanno tutte a buon fine — mentre *ogni* fotogramma va in timeout.
>
> **La causa più comune è che la telecamera è impostata per un trigger hardware.** Con `trigger_mode="On"` e `trigger_source="Line2"`, la telecamera non emette alcun segnale finché non arriva un fronte di segnale sul cavo di sincronizzazione M8. Se non c’è alcun cavo che pilotia quella linea, ogni acquisizione rimane in attesa all’infinito. La telecamera non è guasta e la rete funziona correttamente — sta semplicemente facendo esattamente ciò che le è stato comandato.
>
> `CameraSettings()` e le preimpostazioni `default` / `high_speed` / `high_quality` consentono il funzionamento in modalità free-run, e un’acquisizione che va in timeout mentre è armata fornisce una spiegazione invece di visualizzare semplicemente `-1011`. `PRESETS["triggered"]` arma la Linea 2, come previsto.
>
> Per forzare qualsiasi telecamera al funzionamento in modalità free-run:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Se si verifica ancora un timeout con `trigger_mode="Off"`, la telecamera non sta effettivamente trasmettendo dati: inviateci il log e `ip link show`.

#### Profili colore (anteprima live RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` seleziona il profilo colore di visualizzazione per l’**anteprima live** sulle telecamere RGB (le telecamere multispectrali ignorano l’impostazione):

| Profilo | Significato |
| --- | --- |
| `raw` | Bypassa completamente la catena radiometrica. |
| `linear` | DSNU + flat + WB, senza CCM, senza gamma. |
| `natural` | Lineare + CCM misurato + gamma sRGB, solo con la finitura &quot;economica&quot; (smussamento della crominanza + desaturazione delle alte luci) — l&#x27;impostazione predefinita realistica. |
| `enhanced` | `natural` più la finitura completa con parità di hub (rimozione delle frange, vivacità, contrasto locale CLAHE). Aspetto più ricco a circa **il doppio del costo di elaborazione per fotogramma**, quindi un framerate LIVE inferiore. |
| `custom_temp` | `natural` ma con il bilanciamento del bianco fissato a `custom_cct_k` Kelvin (DLS ignorato; limitato a 2000–10000 K lato backend). |

Il profilo è un controllo di velocità/aspetto **solo per l’anteprima in tempo reale**: le acquisizioni salvate ottengono sempre la finitura completa e ricca indipendentemente dal profilo selezionato, quindi scegliere `natural` per recuperare tempo di frame non riduce la qualità di ciò che viene salvato su disco. Un profilo sconosciuto aumenta `ValueError`; quando un backend chloros è raggiungibile, la modifica viene inviata anche a esso tramite POST, così che il frame di anteprima successivo la rifletta (gli utenti direct-SDK senza backend ottengono comunque la modifica delle impostazioni).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Telecamere mono (M3M) e `Calibration`

Una telecamera mono **M3M** (`M3M-<lens>-F<wavelength>`) è a banda singola: un piano in scala di grigi, nessun mosaico di Bayer, nessuna matrice di crosstalk spettrale 3×3. `Calibration` la riconosce ed espone un flag `is_mono`. La riflettanza si applica comunque come mappa radiometrica per banda (la separazione dei canali è la matrice identità), ma i calcoli multibanda su una singola fotocamera generano risultati validi anziché restituire dati privi di senso:

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

Per costruire un indice di vegetazione a partire da hardware monocromatico, combinare diverse fotocamere M3M a diverse lunghezze d’onda in uno stack multibanda allineato (vedi [Allineamento dell’arrayignment](#array-alignment)) e calcolare l’indice su tale stack anziché su una singola telecamera.

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

> **Chiavi accettate per `apply_sensor_settings`**— esattamente `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; deprecato a favore di `cap_id`), `filter_model` (DAQ-M) e `cap_id` (tutti i tipi DAQ; `None`/`""`/`"none"` = sensore nudo, senza correzione del condensatore). Le chiavi sconosciute vengono**ignorate silenziosamente** — ad es.ad es. `{"integration_time": 64}` non ha alcun effetto (deve essere `integration_time_ms`). Restituisce `{"applied": [...], "errors": {...}}` e non genera mai un&#x27;eccezione.

`chloros_sdk`-esporta solo la superficie principale utilizzata sopra. L’interfaccia pubblica completa `daq_sdk` (22 nomi) aggiunge quanto segue — importarli direttamente da `daq_sdk`:

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

Intercettare la classe base per gestire &quot;qualsiasi errore Chloros&quot;:

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

> `ChlorosAuthenticationError` e `ChlorosConfigurationError` vengono esportati al livello superiore insieme al resto; sono inoltre importabili da `chloros_sdk.exceptions`X, come mostrato.

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

### 3. Campagna di acquisizione basata su progetto

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

### 5. Script di acquisizione diretto sull’hardware senza interfaccia grafica (senza backend)

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

### 6. Verifica delle funzionalità prima di collegare un array a 4 telecamere

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

### 7. Equivalente della ricetta di acquisizione (puro Python)

Il DSL della ricetta di CLI ha un equivalente diretto in Python:

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

I punti di ingresso smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor` e `discover_lattice_cameras` — sono client HTTP &quot;sottili&quot; che presuppongono che un backend sia in ascolto su `127.0.0.1:5000` (l’impostazione predefinita della superficie smart-connect URL). Quando la GUI o CLI sono già in esecuzione, uno di essi è attivo. Partendo da uno script nudo e crudo, potrebbe non essercene uno — quindi queste funzioni **avviano automaticamente il binario del backend in dotazione** (senza finestra, proprio come fa `ChlorosLocal`) prima della loro prima chiamata, quindi attendono fino a `backend_startup_timeout` affinché si avvii.

Regole:

- **Viene avviato solo un URL locale.** È ammesso un `backend_url` che punta a `localhost` / `127.0.0.1` / `[::1]` è idoneo; qualsiasi altro host viene considerato come il computer di qualcun altro e non viene mai generato.
- **Il backend viene lasciato in esecuzione per essere riutilizzato** (come nel caso di CLI) — non avviene alcuno spegnimento implicito all’uscita dello script. L’esecuzione dello script riutilizza il backend attivo.
- **È possibile disattivare questa opzione con `auto_start_backend=False`** in una qualsiasi di queste chiamate (ad esempio quando si è indicato un backend remoto o si gestisce autonomamente il ciclo di vita del backend).

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

Se il binario integrato non può essere individuato o avviato, la successiva chiamata HTTP genera una traccia **specifica per la piattaforma** su cui è possibile intervenire `ChlorosConnectError` anziché una semplice traccia di connessione rifiutata — su Windows ti indirizza all’app desktop o a un comando `chloros-cli`; su Linux (senza interfaccia grafica) rimanda a un comando `chloros-cli` o a `.deb`.

---

## Ambiente e intestazioni

Il codice SDK contrassegna ogni chiamata al backend HTTP con `X-Chloros-Client: sdk`. Il backend applica le regole di licenza SDK/CLI (sono richiesti il login **e** un piano Chloros+ a pagamento) anziché il percorso gratuito della GUI. Questa impostazione viene configurata automaticamente al momento dell’importazione: non è necessario intervenire.

`http://localhost` e `http://127.0.0.1` vengono rilevati come backend locale. Le chiamate ad altri host (ades. il proprio servizio di analisi) non vengono modificate.

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

- La versione SDK è esposta come `chloros_sdk.__version__`.
- SDK vincola il comportamento alla versione del backend in dotazione. La combinazione di un SDK con un backend più recente di solito funziona (endpoint compatibili in avanti), ma l’abbinamento di un SDK più recente con un backend più vecchio potrebbe causare errori `404` sugli endpoint nuovi — aggiornare l’app desktop in modo che sia compatibile.
- L’interfaccia Smart Connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) e l’endpoint di analisi di rete restituiscono schemi JSON stabili; i nuovi campi sono aggiuntivi.

---

## Suggerimenti per la risoluzione dei problemi

- **`ChlorosAuthenticationError: Login required`** → Eseguire una volta `chloros-cli login EMAIL PASSWORD` su questa macchina oppure effettuare l’accesso tramite l’app desktop Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Le chiamate Smart-Connect avviano automaticamente un backend locale, quindi questo messaggio compare solo quando il binario in dotazione non viene trovato o non può essere avviato (ad es. un host solo pip senza pacchetto desktop). Il messaggio varia a seconda della piattaforma: su Windows aprire l’app desktop o eseguire un qualsiasi comando `chloros-cli`; su Linux eseguire un comando `chloros-cli` (non esiste una GUI) oppure installare `.deb`. Per un backend remoto, passare `backend_url=` (e `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** in importazione → `lattice_sdk` non è stato caricato (in genere le DLL di runtime di Arena SDK non sono installate). La superficie non relativa alla telecamera funziona ancora.
- **La connessione array restituisce una risoluzione inferiore a quella nativa**→ La funzione smart-prep del backend riduce automaticamente le dimensioni del fotogramma per adattarle al cavo. Utilizzare `analyze_array_network()` per individuarne il motivo, quindi aggiornare il collegamento, accettare la riduzione o passare a `force_tier="slip-emit-and-capture"` per l’acquisizione sequenziale. La rete di sicurezza della riduzione**non** coprire la sovrasottoscrizione aggregata (`oversubscribed: true`, campi fps 0): un numero eccessivo di telecamere per la linea non può essere risolto tramite binning/ROI — ridurre il numero di telecamere, abilitare i frame jumbo o passare a una scheda di rete più veloce (vedere [Sovrasottoscrizione](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` segnala che l’anello di ricezione della scheda di rete è molto piccolo (~0,26 MB) / i gate di connessione con &quot;FRAMES WILL DROP&quot;** → L’anello di ricezione della scheda di rete host è al valore predefinito (spesso reimpostato a 32 dopo un aggiornamento del driver della scheda di rete). Su un adattatore Realtek USB 10GbE, impostare `ReceiveBufferLen=256` e `PendingReceives=64` (livello elevato), quindi riavviare il backend affinché rilegga l’anello. Procedura completa: [Riferimento CLI → Configurazione e ottimizzazione della scheda di rete host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **L&#x27;host si blocca al riavvio/spegnimento; successivamente si verificano errori WMI `Invalid class` / la scheda di rete non si abilita** → Driver USB 10GbE obsoleto che causa `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Aggiornare il driver della scheda di rete a una versione recente (≥ 2026) e riapplicare le impostazioni del receive-ring. Vedere [Riferimento → Configurazione e ottimizzazione della scheda di rete host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Riflettanza rifiutata** → Per ottenere la riflettanza in scala assoluta, è necessario associare un sistema DAQ attivo alla telecamera (o all’array). Effettuare l’associazione tramite l’interfaccia grafica oppure utilizzare `processing="radiance"` (W/m²/sr/nm), che non richiede un sensore abbinato.
- **L’acquisizione con `smart=True` richiede più tempo del previsto** → La convergenza AE dipende dalla dinamica della scena; restringere `exposure_tolerance_pct` o accorciare `stability_window_s` se si desidera un trigger più veloce (ma meno stabile).

---

## Vedi anche

- [Riferimento CLI](cli-reference.md) — ogni sottocomando CLI corrisponde a una chiamata SDK.
- [Guida ai sensori DAQ](../daq/README.md) — cablaggio, calibrazione e regole di registrazione specifiche per ciascun sensore.
- Documentazione online: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
