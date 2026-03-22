# Installazione di Linux

Chloros viene distribuito per Linux sotto forma di pacchetti `.deb` che installano CLI e il backend. Python SDK viene installato separatamente tramite pip.

***

## Linux amd64 (x86_64)

### Requisiti di sistema

| Requisito | Minimo | Consigliato |
| --- | --- | --- |
| **Distribuzione** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Processore** | x86_64 (Intel/AMD) | Intel Core i7 o superiore |
| **Memoria (RAM)** | 8 GB | 16 GB o più |
| **Scheda grafica** | Nessuna (elaborazione CPU) | GPU NVIDIA con 4 GB+ di VRAM |
| **Spazio su disco** | 2 GB di spazio libero | SSD con 10 GB+ di spazio libero |
| **Python** | Python 3.7+ (per SDK) | Python 3.10+ |

### Installazione

Scaricare il pacchetto `.deb` e installarlo:

```bash
sudo dpkg -i chloros-amd64.deb
```

Verificare l&#x27;installazione:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Requisiti di sistema

| Requisito | Minimo | Consigliato |
| --- | --- | --- |
| **Piattaforma** | NVIDIA Jetson con JetPack 6 | Jetson Orin NX 16 GB o AGX Orin |
| **JetPack** | JetPack 6.x | JetPack 6 più recente |
| **Memoria (RAM)** | 8 GB (condivisa GPU/CPU) | 16 GB+ condivisa |
| **Spazio di archiviazione** | 2 GB di spazio libero | SSD NVMe con 10 GB+ di spazio libero |
| **Python** | Python 3.7+ (per SDK) | Python 3.10+ |

### Installazione

Scaricare il pacchetto JetPack 6 `.deb` e installarlo:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Verifica l&#x27;installazione:

```bash
chloros-cli --version
```

Per la configurazione dettagliata di Jetson, compresa la gestione termica e l&#x27;implementazione sul campo, consulta la [Guida NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Installazione di Python SDK (tutto Linux)

Python SDK viene installato separatamente tramite pip e funziona sia su amd64 che su arm64:

```bash
pip install chloros-sdk
```

Per includere il supporto opzionale per lo streaming di progressi:

```bash
pip install chloros-sdk[progress]
```

Verifica SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Il pacchetto `.deb` installa Chloros, CLI e il backend. Python SDK è un pacchetto pip separato che comunica con il backend tramite un HTTP API locale.
{% endhint %}

***

## Directory di configurazione

Chloros su Linux segue la [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Scopo | Linux Percorso | Windows Equivalente |
| --- | --- | --- |
| **Configurazione** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Dati / Progetti** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / Credenziali** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Posizioni dei file eseguibili del backend

Il pacchetto `.deb` installa il backend in una posizione standard. I pacchetti CLI e SDK rilevano automaticamente il percorso del backend:

| Metodo di installazione | Percorso del backend |
| --- | --- |
| Pacchetto `.deb` | `/usr/lib/chloros/chloros-backend` |
| Manuale / personalizzato | `/opt/mapir/chloros/backend/chloros-backend` |

È possibile sovrascrivere il percorso del backend con il flag `--backend-exe` CLI o il parametro del costruttore `backend_exe` SDK.

***

## Configurazione iniziale

### 1. Attivare la licenza

Per l&#x27;accesso a CLI e SDK è richiesta una licenza Chloros+:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Verifica lo stato della tua licenza

```bash
chloros-cli status
```

### 3. Elabora il tuo primo set di dati

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Esegui la diagnostica di sistema

Verifica che il tuo sistema sia configurato correttamente:

```bash
chloros-cli selftest
```

Questo comando esegue 7 controlli diagnostici, tra cui versione, avvio del backend, API connettività e disponibilità di CUDA/GPU.

***

## Esempi di script Bash

### Elaborazione di più set di dati

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Elaborazione con impostazioni personalizzate

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Elaborazione automatizzata con Cron

Aggiungi al tuo crontab (`crontab -e`) per elaborare automaticamente i nuovi set di dati:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Esempio Python SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Risoluzione dei problemi

### CLI non trovato dopo l&#x27;installazione

Se `chloros-cli` non viene trovato dopo l&#x27;installazione del pacchetto `.deb`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Autorizzazione negata

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Impossibile avviare il backend

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA non rilevato

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Librerie condivise mancanti

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Aggiornamento di Chloros su Linux

Utilizza il comando di aggiornamento integrato per verificare la presenza di aggiornamenti e installarli:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Passaggi successivi

* [Guida NVIDIA Jetson](nvidia-jetson-guide.md) — Ottimizzazione e distribuzione specifiche per Jetson
* [CLI : Riga di comando](../CLI.md) — Riferimento completo ai comandi CLI
* [API : Python SDK](../api-python-sdk.md) — Riferimento completo su SDK
* [Adattamento dinamico del calcolo](../processing-architecture/dynamic-compute-adaptation.md) — Come Chloros si adatta al tuo hardware
