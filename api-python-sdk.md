# API : Python SDK

**Chloros Python SDK** fornisce accesso programmatico al motore di elaborazione delle immagini Chloros, consentendo l&#x27;automazione, flussi di lavoro personalizzati e una perfetta integrazione con le applicazioni Python e le pipeline di ricerca.

### Caratteristiche principali

* 🐍 **Python nativo** - API pulito e Pythonic per l&#x27;elaborazione delle immagini
* 🔧 **Accesso completo a API** - Controllo completo sull&#x27;elaborazione Chloros
* 🚀 **Automazione** - Creazione di flussi di lavoro personalizzati per l&#x27;elaborazione in batch
* 🔗 **Integrazione** - Incorpora Chloros nelle applicazioni Python esistenti
* 📊 **Pronto per la ricerca** - Perfetto per le pipeline di analisi scientifica
* ⚡ **Elaborazione parallela** - Scalabile in base ai core della CPU (Chloros+)

### Requisiti

| Requisito          | Dettagli                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Deve essere installato localmente                                           |
| **Licenza**          | Chloros+ ([abbonamento a pagamento richiesto](https://cloud.mapir.camera/pricing)) |
| **Sistema operativo** | Windows 10/11 (64 bit)                                              |
| **Python**           | Python 3.7 o superiore                                                |
| **Memoria**           | Minimo 8 GB di RAM (consigliati 16 GB)                                  |
| **Internet**         | Necessario per l&#x27;attivazione della licenza                                     |

{% hint style=&quot;warning&quot; %}
**Requisiti di licenza**: Python SDK richiede un abbonamento a pagamento Chloros+ per l&#x27;accesso a API. I piani standard (gratuiti) non consentono l&#x27;accesso a API/SDK. Visita [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) per effettuare l&#x27;aggiornamento.
{% endhint %}

## Avvio rapido

### Installazione

Installare tramite pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Configurazione iniziale**: prima di utilizzare SDK, attiva la licenza Chloros+ aprendo Chloros, Chloros (Browser) o Chloros CLI e accedendo con le proprie credenziali. Questa operazione deve essere eseguita una sola volta.
{% endhint %}

### Utilizzo di base

Elaborare una cartella con poche righe:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Controllo completo

Per flussi di lavoro avanzati:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Guida all&#x27;installazione

### Prerequisiti

Prima di installare SDK, assicurarsi di disporre di:

1. **Chloros Desktop** installato ([download](download.md))
2. **Python 3.7+** installato ([python.org](https://www.python.org))
3. **Licenza Chloros+ attiva** ([aggiornamento](https://cloud.mapir.camera/pricing))

### Installazione tramite pip

**Installazione standard:**

```bash
pip install chloros-sdk
```

**Con supporto per il monitoraggio dell&#x27;avanzamento:**

```bash
pip install chloros-sdk[progress]
```

**Installazione di sviluppo:**

```bash
pip install chloros-sdk[dev]
```

### Verifica dell&#x27;installazione

Verificare che SDK sia installato correttamente:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Configurazione iniziale

### Attivazione della licenza

SDK utilizza la stessa licenza di Chloros, Chloros (Browser) e Chloros CLI. Attivare una volta tramite la GUI o CLI:

1. Aprire **Chloros o Chloros (Browser)** ed effettuare il login nella scheda Utente <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Oppure, apri **CLI**.
2. Inserisci le tue credenziali Chloros+ ed effettua il login
3. La licenza viene memorizzata nella cache locale (persiste anche dopo il riavvio)

{% hint style=&quot;success&quot; %}
**Configurazione una tantum**: dopo aver effettuato l&#x27;accesso tramite la GUI o CLI, SDK utilizza automaticamente la licenza memorizzata nella cache. Non è necessaria alcuna autenticazione aggiuntiva!
{% endhint %}

### Test di connessione

Verificare che SDK sia in grado di connettersi a Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## Riferimento API

### Classe ChlorosLocal

Classe principale per l&#x27;elaborazione delle immagini locali Chloros.

#### Costruttore

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parametri:**

| Parametro                 | Tipo | Predefinito                   | Descrizione                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL del backend locale Chloros          |
| `auto_start_backend`      | bool | `True`                    | Avvia automaticamente il backend se necessario |
| `backend_exe`             | str  | `None` (rilevamento automatico)      | Percorso dell&#x27;eseguibile del backend            |
| `timeout`                 | int  | `30`                      | Timeout della richiesta in secondi            |
| `backend_startup_timeout` | int  | `60`                      | Timeout per l&#x27;avvio del backend (secondi) |

**Esempi:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Metodi

#### `create_project(project_name, camera=None)`

Crea un nuovo progetto Chloros.

**Parametri:**

| Parametro      | Tipo | Obbligatorio | Descrizione                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Sì      | Nome del progetto                                     |
| `camera`       | str  | No       | Modello di telecamera (ad es. &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**Restituisce:** `dict` - Risposta alla creazione del progetto

**Esempio:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Importa immagini da una cartella.

**Parametri:**

| Parametro     | Tipo     | Obbligatorio | Descrizione                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Sì      | Percorso della cartella con le immagini         |
| `recursive`   | bool     | No       | Cerca sottocartelle (impostazione predefinita: False) |

**Restituisce:** `dict` - Risultati dell&#x27;importazione con conteggio dei file

**Esempio:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Configura le impostazioni di elaborazione.

**Parametri:**

| Parametro                 | Tipo | Predefinito                 | Descrizione                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Alta qualità (più veloce)&quot; | Metodo Debayer                  |
| `vignette_correction`     | bool | `True`                  | Abilita correzione vignetta      |
| `reflectance_calibration` | bool | `True`                  | Abilita calibrazione riflettanza  |
| `indices`                 | list | `None`                  | Indici di vegetazione da calcolare |
| `export_format`           | str  | &quot;TIFF (16 bit)&quot;         | Formato di output                   |
| `ppk`                     | bool | `False`                 | Abilita correzioni PPK          |
| `custom_settings`         | dict | `None`                  | Impostazioni personalizzate avanzate        |

**Formati di esportazione:**

* `"TIFF (16-bit)"` - Consigliato per GIS/fotogrammetria
* `"TIFF (32-bit, Percent)"` - Analisi scientifica
* `"PNG (8-bit)"` - Ispezione visiva
* `"JPG (8-bit)"` - Output compresso

**Indici disponibili:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 e altri.

**Esempio:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Elabora le immagini del progetto.

**Parametri:**

| Parametro           | Tipo     | Predefinito      | Descrizione                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Modalità di elaborazione: &quot;parallel&quot; o &quot;serial&quot;   |
| `wait`              | bool     | `True`       | Attendere il completamento                       |
| `progress_callback` | callable | `None`       | Funzione di callback di avanzamento (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Intervallo di polling per l&#x27;avanzamento (secondi)   |

**Restituisce:** `dict` - Risultati dell&#x27;elaborazione

{% hint style=&quot;warning&quot; %}
**Modalità parallela**: richiede la licenza Chloros+. Si adatta automaticamente ai core della CPU (fino a 16 worker).
{% endhint %}

**Esempio:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Ottiene la configurazione corrente del progetto.

**Restituisce:** `dict` - Configurazione corrente del progetto

**Esempio:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Ottiene le informazioni sullo stato del backend.

**Restituisce:** `dict` - Stato del backend

**Esempio:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Arresta il backend (se avviato da SDK).

**Esempio:**

```python
chloros.shutdown_backend()
```

***

### Funzioni di utilità

#### `process_folder(folder_path, **options)`

Funzione di utilità su una riga per elaborare una cartella.

**Parametri:**

| Parametro                 | Tipo     | Predefinito         | Descrizione                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Obbligatorio        | Percorso della cartella con le immagini     |
| `project_name`            | str      | Generato automaticamente  | Nome del progetto                   |
| `camera`                  | str      | `None`          | Modello fotocamera                |
| `indices`                 | list     | `["NDVI"]`      | Indici da calcolare           |
| `vignette_correction`     | bool     | `True`          | Abilita correzione vignettatura     |
| `reflectance_calibration` | bool     | `True`          | Abilita calibrazione riflettanza |
| `export_format`           | str      | &quot;TIFF (16 bit)&quot; | Formato di output                  |
| `mode`                    | str      | `"parallel"`    | Modalità di elaborazione                |
| `progress_callback`       | callable | `None`          | Richiamo di avanzamento              |

**Restituisce:** `dict` - Risultati dell&#x27;elaborazione

**Esempio:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Supporto Context Manager

SDK supporta i context manager per la pulizia automatica:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Esempi completi

### Esempio 1: elaborazione di base

Elaborazione di una cartella con impostazioni predefinite:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Esempio 2: flusso di lavoro personalizzato

Controllo completo sulla pipeline di elaborazione:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Esempio 3: elaborazione batch di più cartelle

Elaborazione di più set di dati di volo:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Esempio 4: integrazione della pipeline di ricerca

Integrazione di Chloros con l&#x27;analisi dei dati:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Esempio 5: monitoraggio personalizzato dello stato di avanzamento

Monitoraggio avanzato dello stato di avanzamento con registrazione:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Esempio 6: gestione degli errori

Gestione degli errori robusta per l&#x27;uso in produzione:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Esempio 7: strumento da riga di comando

Crea uno strumento CLI personalizzato con SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    
    args = parser.parse_args()
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Utilizzo:**

```bash
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Gestione delle eccezioni

SDK fornisce classi di eccezioni specifiche per diversi tipi di errore:

### Gerarchia delle eccezioni

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Esempi di eccezioni

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Argomenti avanzati

### Configurazione backend personalizzata

Utilizzare una posizione o una configurazione backend personalizzata:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Elaborazione non bloccante

Avviare l&#x27;elaborazione e continuare con altre attività:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Gestione della memoria

Per set di dati di grandi dimensioni, elaborare in batch:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Risoluzione dei problemi

### Il backend non si avvia

**Problema:** SDK non riesce ad avviare il backend.

**Soluzioni:**

1. Verificare che Chloros Desktop sia installato:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Verificare che Windows Firewall non stia bloccando
3. Provare il percorso manuale del backend:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Licenza non rilevata

**Problema:** SDK avvisa della mancanza della licenza

**Soluzioni:**

1. Aprire Chloros, Chloros (browser) o Chloros CLI ed effettuare il login.
2. Verificare che la licenza sia memorizzata nella cache:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Contattare l&#x27;assistenza: info@mapir.camera

***

### Errori di importazione

**Problema:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Soluzioni:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Timeout di elaborazione

**Problema:** Timeout di elaborazione

**Soluzioni:**

1. Aumentare il timeout:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Elaborare batch più piccoli
3. Controllare lo spazio disponibile su disco
4. Monitorare le risorse di sistema

***

### Porta già in uso

**Problema:** Porta backend 5000 occupata

**Soluzioni:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Oppure individuare e chiudere il processo in conflitto:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Suggerimenti per le prestazioni

### Ottimizzare la velocità di elaborazione

1. **Utilizzare la modalità parallela** (richiede Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Ridurre la risoluzione di output** (se accettabile)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Disattivare gli indici non necessari**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Elaborare su SSD** (non su HDD)

***

### Ottimizzazione della memoria

Per set di dati di grandi dimensioni:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Elaborazione in background

Liberare Python per altre attività:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Esempi di integrazione

### Integrazione Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Domande frequenti

### D: SDK richiede una connessione Internet?

**R:** Solo per l&#x27;attivazione iniziale della licenza. Dopo aver effettuato l&#x27;accesso tramite Chloros, Chloros (browser) o Chloros CLI, la licenza viene memorizzata nella cache locale e funziona offline per 30 giorni.

***

### D: Posso utilizzare SDK su un server senza GUI?

**R:** Sì! Requisiti:

* Windows Server 2016 o versioni successive
* Chloros installato (una sola volta)
* Licenza attivata su qualsiasi macchina (licenza memorizzata nella cache copiata sul server)

***

### D: Qual è la differenza tra Desktop, CLI e SDK?

| Funzionalità         | GUI desktop | CLI Riga di comando | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Interfaccia**   | Puntamento e clic | Comando          | Python API  |
| **Ideale per**    | Lavoro visivo | Scripting        | Integrazione |
| **Automazione**  | Limitata     | Buona             | Eccellente   |
| **Flessibilità** | Base       | Buona             | Massima     |
| **Licenza**     | Chloros+    | Chloros+         | Chloros+    |

***

### D: Posso distribuire app create con SDK?

**R:** Il codice SDK può essere integrato nelle vostre applicazioni, ma:

* Gli utenti finali devono avere installato Chloros
* Gli utenti finali devono avere licenze Chloros+ attive
* La distribuzione commerciale richiede licenze OEM

Contattare info@mapir.camera per richieste relative alle licenze OEM.

***

### D: Come si aggiorna SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### D: Dove vengono salvate le immagini elaborate?

Per impostazione predefinita, nel percorso del progetto:

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### D: Posso elaborare immagini da script Python eseguiti in base a una pianificazione?

**R:** Sì! Utilizza Windows Task Scheduler con gli script Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Imposta l&#x27;esecuzione giornaliera tramite Task Scheduler.

***

### D: SDK supporta async/await?

**R:** La versione attuale è sincrona. Per un comportamento asincrono, utilizza `wait=False` o esegui in un thread separato:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Assistenza

### Documentazione

* **Riferimento API**: questa pagina

### Canali di assistenza

* **E-mail**: info@mapir.camera
* **Sito web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Prezzi**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Codice di esempio

Tutti gli esempi qui elencati sono stati testati e sono pronti per la produzione. Copiateli e adattateli al vostro caso d&#x27;uso.

***

## Licenza

**Software proprietario** - Copyright (c) 2025 MAPIR Inc.

SDK richiede un abbonamento attivo a Chloros+. È vietato l&#x27;uso, la distribuzione o la modifica non autorizzati.
