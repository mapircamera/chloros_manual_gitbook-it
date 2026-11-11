# API: SDK per Python

{% hint style="info" %}
**Cerchi l'API completa?** Questa pagina è un tutorial pratico. Tutte le classi pubbliche, i metodi, le firme esatte e gli esempi copiabili e incollabili si trovano nella [Guida di riferimento SDK](reference/sdk-reference.md), ottimizzata per gli assistenti AI.**Lavori con un assistente AI?** Incolla questo URL nella chat in modo che disponga dell’API completa e aggiornata di Chloros 1.3.0:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Ogni pagina di questo manuale è disponibile in formato Markdown grezzo all’indirizzo corrispondente al suo slug in minuscolo + `.md`, mentre l’intero manuale è indicizzato all’indirizzo `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

Il **Chloros Python SDK** (`chloros-sdk` su PyPI) gestisce tutte le funzionalità dell’app desktop da Python: elaborazione batch delle immagini, controllo in tempo reale della telecamera LATTICE e dell’array, sessioni DAQ con sensori di luce e automazione dei progetti salvati. Si tratta di un sottile strato sovrapposto allo stesso backend locale utilizzato dall’interfaccia grafica (GUI) e dalla riga di comando (CLI) (HTTP su `127.0.0.1:5000`), pertanto il comportamento è identico su tutte e tre le interfacce.

## Installazione

L’SDK è incluso nel pacchetto desktop Chloros, che fornisce anche il backend di elaborazione e i runtime hardware, e la cui versione corrisponde sempre a quella dell’SDK.

**Passaggio 1 — Installare Chloros.** Windows: eseguire il programma di installazione desktop (percorso predefinito `C:\Program Files\Chloros\`) dalla pagina [Download](download.md). Linux: installare il pacchetto `.deb` con `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Installazione su Linux](linux/linux-installation.md)).**Passaggio 2 — Verifica dell’SDK** (Python 3.7+). Di solito non è necessario installare nulla:

* **Windows:** il programma di installazione installa il wheel dell’SDK in dotazione nel Python di sistema (se Python non è installato, questo passaggio viene saltato).
* **Linux:** il pacchetto `.deb` installa l’SDK per il sistema `python3` in `/usr/lib/python3/dist-packages`. I suoi prerequisiti, `python3-numpy` e `python3-requests`, sono raccomandati dal pacchetto: `apt install` li include, mentre `dpkg -i` no.
* **Un virtualenv** non vede i pacchetti di sistema; installate al suo interno il file wheel in dotazione: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` su Linux, oppure `chloros_sdk-*.whl` nella cartella `sdk` della cartella di installazione di Windows (`C:\Program Files\Chloros\sdk\` per impostazione predefinita).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Su una macchina priva del pacchetto Chloros, il cui SDK gestisce un backend su un’altra macchina (vedi [Cosa si ottiene con un’installazione solo tramite pip](#what-a-pip-only-install-gets)), installarlo da PyPI: `pip install chloros-sdk`. PyPI viene aggiornato in occasione delle build di rilascio, in modo da corrispondere all’ultima versione stabile.

**Passaggio 3 — Effettua l’accesso una volta per ogni macchina:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Le credenziali vengono memorizzate nella cache in `~/.chloros/` (su entrambe le piattaforme). Su Windows, è possibile effettuare l’accesso in modo equivalente tramite la scheda Utente <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> dell’app desktop. L’SDK richiede un piano Chloros+ a pagamento — vedi [Requisiti di licenza](#license-requirement) di seguito.

| Requisito | Dettagli |
| --- | --- |
| **Chloros installato** | Windows: programma di installazione desktop; Linux: pacchetto `.deb` (fornisce il binario del backend e l’SDK) |
| **Python** | 3.7 o superiore (sviluppato/testato su 3.10) |
| **Sistema operativo** | Windows 10/11 a 64 bit, Ubuntu 22.04 LTS o versioni più recenti, oppure NVIDIA Jetson (JetPack 6) |
| **Licenza** | Accesso attivo a Chloros+, qualsiasi piano a pagamento (Copper o superiore) |

## Il successo in 60 secondi

Una sola chiamata crea un progetto, importa una cartella, configura l’elaborazione ed esegue la pipeline — avviando automaticamente il backend se non è già in esecuzione:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(Su Linux, utilizzare i percorsi Linux: `/home/user/drone_images/flight001`. L’SDK funziona in modo identico su entrambe le piattaforme.)

Si sta elaborando una cartella di acquisizione LATTICE? Utilizza il wrapper ottimizzato per LATTICE: applica le impostazioni predefinite corrette (nessun rilevamento del pannello di destinazione, debayer standard):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — controllo completo della pipeline

Per qualsiasi operazione che vada oltre un comando di una sola riga, usa `ChlorosLocal`. Utilizza un backend già in esecuzione (quello dell’app desktop) oppure avvia quello installato al primo utilizzo (`auto_start_backend=True`; su Windows dalla cartella `backend` della cartella di installazione di Chloros, `C:\Program Files\Chloros\backend\` per impostazione predefinita), crea e configura i progetti, monitora lo stato di avanzamento e restituisce un riepilogo al termine dell’esecuzione.

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
Mantenere il valore predefinito `http://127.0.0.1:5000` anziché sostituirlo con `localhost` — su Windows, `localhost` viene prima risolto in `::1` e richiede circa 2 secondi per ogni richiesta al backend solo IPv4.
{% endhint %}

Utilizzarlo come gestore di contesto. Uscendo dal blocco si interrompe l’utilizzo del backend da parte di questa istanza, non il backend stesso: quest’ultimo rimane in esecuzione per lo script successivo e si arresta automaticamente dopo 30 minuti di inattività. `shutdown_backend()` interrompe l’istanza una volta avviata.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` accetta le seguenti parole chiave: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` e `custom_settings`. I valori principali:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

I parametri specifici di LATTICE (`input_level`, `radiometric_output`, la famiglia `array_alignment*`) sono documentati con le relative tabelle complete dei valori nella [Guida di riferimento SDK](reference/sdk-reference.md#supported-values).

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

Al termine, `process()` allega il riepilogo di elaborazione del backend come `result["summary"]`. Ogni voce in `summary["hints"]` è una frase completa che spiega qualsiasi aspetto degno di nota — ad esempio, perché un'esecuzione non ha prodotto alcun risultato — e ogni suggerimento viene inoltre riproposto come `UserWarning` in Python, quindi le esecuzioni vuote si autodiagnosticano anche se non si ispeziona mai il dizionario:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` non genera un'eccezione quando un'esecuzione non produce immagini.** Questo è l'unico punto in cui l'SDK e la CLI differiscono deliberatamente: `chloros-cli process` considera "i prodotti sono stati richiesti, ma nessuno è stato scritto" come un errore e termina con un codice di uscita diverso da zero, mentre l’SDK torna normalmente e segnala la condizione tramite `summary` / suggerimenti. Se la tua pipeline deve interrompersi in caso di esecuzione vuota, verifica tu stesso la situazione — controlla `summary` (o conta i file nella cartella del progetto) anziché fare affidamento su un'eccezione.
{% endhint %}

## Smart Connect — hardware attivo

Tre helper aprono sessioni persistenti nel pool hardware del backend — lo stesso pool utilizzato dalla CLI, da altri script e dalle schede delle telecamere della GUI — in modo che gli script SDK possano coesistere con essi senza competere per le porte seriali o la larghezza di banda di rete. Tutti e tre avviano automaticamente un backend locale se non ce n’è uno in esecuzione. Il pannello DAQ dell’interfaccia grafica non fa parte del pool: chiudere un sensore di luce nell’interfaccia grafica prima di aprirlo da uno script.

### Singola telecamera LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**I percorsi relativi puntano alla directory in cui viene eseguito lo script (modificato nella versione 1.3.0).** Con un backend locale, un percorso relativo `output_dir` (incluso quello predefinito `"output"`) — e gli altri percorsi che il backend apre (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — vengono risolti in base alla directory di lavoro dello script, quindi `cam.capture("output/")` scrive in `<current folder>/output/`. Nella versione 1.2.0 un backend installato li risolvava nella propria cartella di installazione e generava un errore con il messaggio "Accesso negato". Con un backend remoto, i percorsi vengono inviati così come digitati.
{% endhint %}

### Array sincronizzato — `connect_array`

`connect_array` è il punto di ingresso consigliato per le configurazioni multicamera. Esegue lo stesso flusso di preparazione intelligente (smart-prep) dell’interfaccia grafica: analisi di rete, selezione automatica del livello di sincronizzazione, sincronizzazione temporale PTP, selezione del formato pixel per ciascuna telecamera, inizializzazione dell’esposizione automatica (AE) e attivazione del trigger GPIO. La **prima porta seriale è quella master** (emette l’impulso di trigger hardware); le altre sono slave.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Aggiungere `smart=True` a qualsiasi acquisizione in array per attendere che l’esposizione automatica si stabilizzi su tutte le telecamere prima dell’attivazione.

Novità della versione 1.3.0: `ArraySession` fornisce anche direttamente i fotogrammi sincronizzati, senza scrivere file:

* `arr.grab()` restituisce un fotogramma per ogni telecamera, tutti provenienti da un unico trigger, sotto forma di array numpy (`{serial: array}`); `.group_id` è il trigger condiviso e `.errors` indica eventuali telecamere escluse.
* `arr.frame_stream(fps=5.0)` restituisce un set di questo tipo per ogni tick, per l’elaborazione in tempo reale nella propria pipeline.
* `arr.continuous("output/")` avvia l’acquisizione continua propria del backend (la modalità “Continuous” della GUI) e restituisce un `ContinuousHandle` con `status()`, `wait()` e `stop()` — utilizzatelo al posto di un ciclo di acquisizione quando è necessario salvare ogni trigger a una frequenza elevata.

Per le modalità di acquisizione (Singola / Continua / A intervalli / Più veloce), i registratori, il burst-to-video e l’allineamento degli array, consultare il [Riferimento SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Sensore di luce DAQ — `connect_daq_sensor`

Senza argomenti, `connect_daq_sensor()` rileva automaticamente il protocollo di trasporto (ordine di priorità: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Ogni frame contiene il valore di 135 punti `spectrum` (W/m²/nm una volta calibrato), un flag `is_saturated` e i valori CIE `x`, `y`, `z`. Per specificare un sensore o un protocollo di trasporto particolare, passare un’indicazione esplicita:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Si noti che i profili di correzione del limite massimo (`cap_id`) **non** sono un parametro dell’SDK: selezionarli invece tramite `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Progetti salvati — `open_project`

Un progetto Chloros salvato mantiene l’hardware collegato (`cameras.json` + `sensors.json` insieme a `project.json`), e `chloros_sdk.open_project(path)` può ricollegare contemporaneamente le proprie telecamere, array e sensori di luce ed eseguire acquisizioni in base al nome del dispositivo. I sensori di luce si riconnettono tramite un backend Chloros in esecuzione, con le impostazioni e la capacità salvate. Vedi [Automazione del progetto](reference/sdk-reference.md#project-automation--chlorosproject) nella documentazione di riferimento.

## Cosa si ottiene con un’installazione solo pip

Verificare i flag di disponibilità a livello di modulo prima di utilizzare le superfici hardware:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Su un host con **solo** `pip install chloros-sdk` e senza il pacchetto desktop Chloros:

* `ChlorosLocal`, `process_folder` e `process_lattice_capture` **non** funzionano: richiedono il binario di backend fornito all’interno del programma di installazione desktop.
* Gli helper smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) sono client HTTP puri, quindi funzionano con un backend su un’altra macchina — ma i backend forniti si legano solo al loopback, quindi è necessario inoltrare la porta manualmente (ad es. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) e passare `backend_url="http://127.0.0.1:5000"` insieme a `auto_start_backend=False`. Vedi [Modalità backend remoto](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Le classi LATTICE a livello hardware diretto (`LatticeCamera`, `CameraPool`, …) vengono importate, ma richiedono il runtime dell’SDK di Arena dal pacchetto desktop; senza di esso, `CAMERA_AVAILABLE` è `False`.
* I sensori DAQ vengono gestiti solo tramite un backend (tunnelizzato) — `connect_daq_sensor()`, oppure l’`proj.sensors` di un progetto; il backend apre il sensore sulla propria macchina.

## Assistenti AI — `chloros-mcp`

L’SDK include `chloros-mcp`, un server MCP che consente a Claude Desktop, Claude Code, Cursor e altri client MCP di gestire Chloros: elaborare immagini e far funzionare telecamere LATTICE, array e sensori di luce DAQ. I suoi strumenti coprono le funzionalità di questo SDK. Si installa insieme all’SDK, quindi per Claude Code la configurazione richiede una sola riga:

```bash
claude mcp add chloros -- chloros-mcp
```

Altri client: [Utilizzo di Chloros con gli assistenti AI](ai-assistants.md). Opzioni e tutti gli strumenti: [Server MCP (assistenti AI)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Requisiti di licenza

L’accesso all’SDK richiede un account Chloros+ attivo su qualsiasi piano a pagamento — **Copper o superiore**(Copper / Bronze / Silver / Gold); il piano gratuito Iron non prevede l’accesso all’SDK o alla CLI. La verifica avviene**lato server**: ogni richiesta SDK deve includere sia una sessione attiva che un piano a pagamento, altrimenti il backend restituisce `403` / `PLAN_UPGRADE_REQUIRED` (generati come `ChlorosLicenseError` da `ChlorosLocal` e come `ChlorosConnectError` dagli helper `connect_*`). Un chiamante che ha effettuato il logout riceve invece `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — l’esecuzione di `chloros-cli login` risolve il primo caso ma non il secondo.

L’utilizzo offline funziona entro il periodo di tolleranza del piano: il livello viene letto dalla cache di convalida del server (5 minuti) o dalla cache delle licenze firmate e legate al dispositivo (30 giorni per i piani mensili; fino alla scadenza dell’abbonamento per quelli annuali). Allo scadere del periodo di tolleranza, il piano passa alla versione gratuita e l’accesso all’SDK viene interrotto fino a quando il dispositivo non si connette al server almeno una volta. `chloros-cli status` rimane accessibile nel livello gratuito, quindi il motivo è sempre visibile. Vedi [Accesso a Chloros+](chloros+-login.md).

## Eccezioni

Intercetta la classe base per gestire "qualsiasi problema con Chloros":

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

Tutte le eccezioni della pipeline (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) derivano da `ChlorosError`. Un dettaglio da tenere presente: `ChlorosConnectError` — generato solo da `connect_camera` / `connect_array` / `connect_daq_sensor` — deriva dal semplice `Exception`, **non** da `ChlorosError`, quindi `except ChlorosError` non lo intercetterà. La gerarchia completa è disponibile nella [Guida di riferimento SDK](reference/sdk-reference.md#exceptions).

## Vedi anche

* [Guida di riferimento SDK](reference/sdk-reference.md) — l’interfaccia API completa, ottimizzata per gli assistenti AI.
* [Riferimento CLI](reference/cli-reference.md) — ogni sottocomando CLI rispecchia una chiamata SDK.
* [Utilizzo di Chloros con gli assistenti AI](ai-assistants.md) — configurazione di `chloros-mcp` e ricette per i prompt.
* [Download](download.md) — programmi di installazione per Windows e Linux.
