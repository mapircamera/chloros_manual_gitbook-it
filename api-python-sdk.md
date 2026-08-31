# API : Python SDK

{% hint style="info" %}
**Cerchi la documentazione completa su API?** Questa pagina è un tutorial pratico. Ogni classe pubblica, metodo, firma esatta ed esempio copiabile si trova nel [Riferimento SDK](reference/sdk-reference.md), ottimizzato per gli assistenti AI.**Lavori con un assistente AI?** Incolla questo URL nella chat in modo che disponga della versione completa e aggiornata di Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Ogni pagina di questo manuale è disponibile in formato Markdown grezzo all&#x27;indirizzo corrispondente al suo slug in minuscolo + `.md`, mentre l&#x27;intero manuale è indicizzato all&#x27;indirizzo `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

Il **Chloros Python SDK** (`chloros-sdk` su PyPI) gestisce tutte le funzionalità dell’applicazione desktop a partire da Python: elaborazione in batch delle immagini, controllo in tempo reale della telecamera LATTICE e dell’array, sessioni DAQ con sensori di luce e automazione dei progetti salvati. Si tratta di un sottile strato che si sovrappone allo stesso backend locale utilizzato dall’interfaccia grafica e da CLI (HTTP su `127.0.0.1:5000`), pertanto il comportamento è identico su tutte e tre le interfacce.

## Installazione

L’installazione avviene in due fasi: prima il pacchetto desktop Chloros (che fornisce il backend di elaborazione e i runtime hardware), poi il pacchetto Python.

**Fase 1 — Installare Chloros.** Windows: eseguire il programma di installazione desktop (percorso predefinito `C:\Program Files\MAPIR\Chloros\`) dalla pagina [Download](download.md). Linux: installare il pacchetto `.deb` ([Installazione di Linux](linux/linux-installation.md)).**Passaggio 2 — Installare SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Potrebbe non essere nemmeno necessario utilizzare pip: ogni programma di installazione include una wheel SDK corrispondente. Il programma di installazione Windows lo installa automaticamente nel sistema Python; quello Linux `.deb` lo colloca in `/usr/lib/chloros/sdk/` e visualizza il comando esatto `pip install --user`. PyPI viene aggiornato con le build di rilascio, quindi `pip install chloros-sdk` corrisponde all’ultima versione stabile.

**Passaggio 3 — Effettuare l’accesso una volta per ogni macchina:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Le credenziali vengono memorizzate nella cache in `~/.chloros/` (su entrambe le piattaforme). Su Windows è possibile effettuare l’accesso in modo equivalente tramite la scheda “Utente” <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> dell’app desktop. SDK richiede un piano a pagamento Chloros+ — vedere [Requisiti di licenza](#license-requirement) di seguito.

| Requisito | Dettagli |
| --- | --- |
| **Chloros installato** | Windows: programma di installazione desktop; Linux: pacchetto `.deb` (fornisce il binario del backend) |
| **Python** | 3.7 o superiore (sviluppato/testato su 3.10) |
| **Sistema operativo** | Windows 10/11 a 64 bit, Ubuntu 22.04 LTS o versioni successive, oppure NVIDIA Jetson (JetPack 6) |
| **Licenza** | Accesso Chloros+ attivo, qualsiasi piano a pagamento (Copper o superiore) |

## Il successo in 60 secondi

Una sola chiamata crea un progetto, importa una cartella, configura l’elaborazione ed esegue la pipeline — avviando automaticamente il backend se non è già in esecuzione:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Su Linux, utilizzare i percorsi Linux: `/home/user/drone_images/flight001`. SDK funziona in modo identico su entrambe le piattaforme.)

Si sta elaborando una cartella di acquisizione LATTICE? Utilizzare il wrapper compatibile con LATTICE: applica le impostazioni predefinite corrette (nessun rilevamento del pannello di destinazione, debayer standard):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — controllo completo della pipeline

Per qualsiasi operazione che vada oltre una singola riga di comando, usa `ChlorosLocal`. Al primo utilizzo avvia il backend (`auto_start_backend=True`), crea e configura i progetti, monitora lo stato di avanzamento e restituisce un riepilogo al termine dell’esecuzione.

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

{% hint style="info" %}
Mantenere il valore predefinito `http://127.0.0.1:5000` anziché sostituirlo con `localhost` — su Windows, `localhost` viene risolto prima in `::1` e richiede circa 2 secondi per ogni richiesta sul backend solo IPv4.
{% endhint %}

Utilizzarlo come gestore di contesto per garantire la pulizia:

```python
import chloros_sdk

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

`configure()` accetta le seguenti parole chiave: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` e `custom_settings`. I valori principali:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Le manopole specifiche per LATTICE (`input_level`, `radiometric_output` e la famiglia `array_alignment*`) sono documentate con le relative tabelle complete dei valori nel [Riferimento SDK](reference/sdk-reference.md#supported-values).

### Monitoraggio dello stato di avanzamento

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Lettura del riepilogo post-esecuzione — e individuazione delle esecuzioni vuote

Al termine, `process()` allega il riepilogo di elaborazione del backend come `result["summary"]`. Ogni voce in `summary["hints"]` è una frase completa che spiega qualsiasi aspetto degno di nota — ad esempio, perché un&#x27;esecuzione non ha prodotto alcun risultato — e ogni suggerimento viene inoltre ritrasmesso come Python `UserWarning`, quindi le esecuzioni vuote si autodiagnosticano anche se non si ispeziona mai il dizionario:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` non viene generato quando un&#x27;esecuzione non produce immagini.** Questo è l&#x27;unico caso in cui SDK e CLI differiscono deliberatamente: `chloros-cli process` considera &quot;prodotti richiesti, nessuno scritto&quot; come un errore e termina con un codice di uscita diverso da zero, mentre SDK termina normalmente e segnala la condizione tramite `summary` / suggerimenti. Se la tua pipeline dovesse interrompersi in caso di esecuzione vuota, verifica tu stesso la situazione: controlla `summary` (o conta i file nella cartella del progetto) invece di affidarti a un&#x27;eccezione.
{% endhint %}

## Smart Connect — hardware attivo

Tre helper aprono sessioni persistenti nel pool hardware del backend — lo stesso pool utilizzato dall’interfaccia grafica, quindi gli script SDK coesistono con l’app desktop senza competere per le porte seriali o la larghezza di banda di rete. Tutti e tre avviano automaticamente un backend locale se non ce n’è uno in esecuzione.

### Singola telecamera LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Array sincronizzato — `connect_array`

`connect_array` è il punto di ingresso consigliato per le configurazioni multi-telecamera. Esegue lo stesso flusso di preparazione intelligente della GUI: analisi di rete, selezione automatica del livello di sincronizzazione, sincronizzazione temporale PTP, selezione del formato pixel per telecamera, inizializzazione dell’esposizione automatica (AE) e attivazione del trigger GPIO. La **prima porta seriale è quella master** (emette l’impulso di trigger hardware); le altre sono slave.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Aggiungere `smart=True` a qualsiasi acquisizione in array per attendere che l’esposizione automatica si stabilizzi su tutte le telecamere prima dell’attivazione. Per le modalità di acquisizione (Singola / Continua / A intervalli / Più veloce), i registratori, la modalità burst-to-video e l’allineamento dell’array, consultare il [Riferimento SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Sensore di luce DAQ — `connect_daq_sensor`

Senza argomenti, `connect_daq_sensor()` rileva automaticamente il protocollo di trasporto (ordine di priorità: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Ogni frame contiene il valore di 135 punti `spectrum` (W/m²/nm una volta calibrato), un flag `is_saturated` e i valori CIE `x`, `y`, `z`. Per specificare un sensore o un protocollo di trasporto particolare — la scelta più affidabile su host con più interfacce di rete, dove il rilevamento automatico via Ethernet potrebbe non individuare un DAQ-E funzionante al primo tentativo — è necessario passare un suggerimento esplicito:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Si noti che i profili di correzione del volume (`cap_id`) **non** sono un parametro di regolazione come SDK: selezionarli invece tramite `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Progetti salvati — `open_project`

Un progetto Chloros salvato mantiene le impostazioni dell&#x27;hardware collegato (`cameras.json` + `sensors.json` insieme a `project.json`), e `chloros_sdk.open_project(path)` è in grado di ricollegare tutto in una volta sola e di gestire le acquisizioni in base al nome del dispositivo. Vedere [Automazione dei progetti](reference/sdk-reference.md#project-automation--chlorosproject) nella documentazione di riferimento.

## Cosa offre un&#x27;installazione solo tramite pip

Verificare i flag di disponibilità a livello di modulo prima di utilizzare le superfici hardware:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Su un host con **solo** `pip install chloros-sdk` e senza il pacchetto desktop Chloros:

* `ChlorosLocal`, `process_folder` e `process_lattice_capture` **non** funzionano — richiedono il binario di backend incluso nell’installatore desktop.
* Gli helper smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) sono client HTTP puri, quindi funzionano con un backend su un’altra macchina — ma i backend forniti si legano solo al loopback, quindi è necessario inoltrare la porta manualmente (ad es. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) e passare `backend_url="http://127.0.0.1:5000"` con `auto_start_backend=False`. Vedi [Modalità backend remoto](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Le classi LATTICE a livello hardware diretto (`LatticeCamera`, `CameraPool`, …) possono essere importate, ma richiedono il runtime Arena SDK presente nel pacchetto desktop — senza di esso, `CAMERA_AVAILABLE` è `False`.
* `daq_sdk` (le classi DAQ dirette) è incluso nell’installazione desktop, ma non nel pacchetto PyPI, quindi `DAQ_AVAILABLE` corrisponde a `False` su un host che utilizza solo pip — pilotare i sensori DAQ tramite `connect_daq_sensor()` su un backend (tunnelizzato) invece.

## Requisiti di licenza

L&#x27;accesso a SDK richiede un account Chloros+ attivo su qualsiasi piano a pagamento — **Copper o superiore**(Copper / Bronze / Silver / Gold); il piano gratuito Iron non prevede l’accesso a SDK/CLI. L&#x27;applicazione avviene**lato server**: ogni richiesta SDK deve includere sia una sessione attiva che un piano a pagamento, altrimenti il backend restituisce `403` / `PLAN_UPGRADE_REQUIRED` (generato come `ChlorosLicenseError` da `ChlorosLocal` e come `ChlorosConnectError` dagli helper `connect_*`). Un chiamante disconnesso riceve invece `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — l’esecuzione di `chloros-cli login` risolve il primo caso ma non il secondo.

L&#x27;utilizzo offline funziona entro il periodo di tolleranza del piano: il livello viene letto dalla cache di convalida del server (5 minuti) o dalla cache delle licenze firmate e legate al dispositivo (30 giorni per i piani mensili; fino alla scadenza dell&#x27;abbonamento per quelli annuali). Allo scadere del periodo di grazia, il piano passa alla versione gratuita e l’accesso tramite SDK viene interrotto fino a quando il dispositivo non si connette al server almeno una volta. `chloros-cli status` rimane accessibile nel livello gratuito, quindi il motivo è sempre visibile. Vedi [Chloros+ Accesso](chloros+-login.md).

## Eccezioni

Intercettare la classe base per gestire &quot;qualsiasi problema Chloros&quot;:

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

Tutte le eccezioni della pipeline (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) derivano da `ChlorosError`. Un dettaglio da tenere presente: `ChlorosConnectError` — generato solo da `connect_camera` / `connect_array` / `connect_daq_sensor` — deriva dal semplice `Exception`, **non** da `ChlorosError`, quindi `except ChlorosError` non lo intercetterà. La gerarchia completa è disponibile nel [Riferimento SDK](reference/sdk-reference.md#exceptions).

## Vedi anche

* [Riferimento SDK](reference/sdk-reference.md) — la superficie completa API, ottimizzata per gli assistenti AI.
* [Riferimento CLI](reference/cli-reference.md) — ogni sottocomando CLI rispecchia una chiamata SDK.
* [Download](download.md) — programmi di installazione per Windows e Linux.
