# CLI : Riga di comando

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** offre un potente accesso da riga di comando al motore di elaborazione delle immagini Chloros, consentendo l&#x27;automazione, la creazione di script e il funzionamento headless per i vostri flussi di lavoro di imaging.

### Caratteristiche principali

* 🚀 **Automazione** - Elaborazione batch tramite script di più set di dati
* 🔗 **Integrazione** - Incorporabile in flussi di lavoro e pipeline esistenti
* 💻 **Funzionamento senza interfaccia grafica** - Esecuzione senza GUI
* 🌍 **Multilingue** - Supporto per 38 lingue
* ⚡ **Elaborazione parallela** - [Adattamento dinamico della potenza di calcolo](processing-architecture/dynamic-compute-adaptation.md) ottimizza automaticamente in base al vostro hardware

### Requisiti

| Requisito          | Dettagli                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Sistema operativo** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licenza**          | Chloros+ ([è richiesto un piano a pagamento](https://cloud.mapir.camera/pricing)) |
| **Memoria**           | Minimo 8 GB di RAM (consigliati 16 GB)                                  |
| **Internet**         | Necessario per l&#x27;attivazione della licenza                                     |
| **Spazio su disco**       | Varia in base alle dimensioni del progetto                                              |

{% hint style="warning" %}
**Requisiti di licenza**: CLI richiede un abbonamento a pagamento Chloros+. I piani Standard (gratuiti) non consentono l&#x27;accesso a CLI. Visita [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) per effettuare l&#x27;aggiornamento.
{% endhint %}

## Guida rapida

### Installazione

#### Windows

CLI è incluso automaticamente nel programma di installazione di Chloros:

1. Scaricare ed eseguire **Chloros Installer.exe**

2. Completare la procedura guidata di installazione
3. CLI installato in: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Il programma di installazione aggiunge automaticamente `chloros-cli` al PATH di sistema. Riavvia il terminale dopo l&#x27;installazione.
{% endhint %}

#### Linux

Installa il pacchetto `.deb` per la tua architettura:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Per una configurazione dettagliata di Linux, consultare [Installazione di Linux](linux/linux-installation.md).

### Configurazione iniziale

Prima di utilizzare CLI, attivare la licenza Chloros+:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Utilizzo di base

Elaborazione di una cartella con le impostazioni predefinite:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Riferimento ai comandi

### Sintassi generale

```
chloros-cli [global-options] <command> [command-options]
```

***

## Comandi

### `process` - Elaborazione immagini

Elabora le immagini in una cartella con calibrazione.

**Sintassi:**

```bash
chloros-cli process <input-folder> [options]
```

**Esempi:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Opzioni del comando di elaborazione

| Opzione                | Tipo    | Predefinito        | Descrizione                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Percorso    | _Obbligatorio_     | Cartella contenente immagini multispettrali RAW/JPG                                         |
| `-o, --output`        | Percorso    | Uguale all&#x27;input  | Cartella di output per le immagini elaborate                                                     |
| `-n, --project-name`  | Stringa  | Generato automaticamente | Nome progetto personalizzato                                                                    |
| `--vignette`          | Flag    | Abilitato        | Abilita correzione vignettatura                                                             |
| `--no-vignette`       | Flag    | -              | Disabilita correzione vignettatura                                                            |
| `--reflectance`       | Flag    | Abilitato        | Abilita calibrazione della riflettanza                                                         |
| `--no-reflectance`    | Flag    | -              | Disabilita calibrazione della riflettanza                                                        |
| `--ppk`               | Flag    | Disabilitato       | Applica correzioni PPK dai dati del sensore di luce .daq                                      |
| `--format`            | Scelta  | TIFF (16 bit)  | Formato di output: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Intero | Auto           | Dimensione minima del bersaglio in pixel per il rilevamento del pannello di calibrazione                          |
| `--target-clustering` | Intero | Auto           | Soglia di raggruppamento dei bersagli (0-100)                                                    |
| `--debayer`           | Scelta  | `standard`     | Metodo di debayering: `standard` o `texture-aware` (solo Chloros+)                          |
| `--target`, `--targets` | Flag  | Disabilitato       | Cerca solo i target di calibrazione in una sottocartella &quot;target&quot; o &quot;targets&quot; (accelera l&#x27;elaborazione) |
| `--indices`           | Elenco    | Nessuno           | Indici di vegetazione da calcolare (ad es. `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Stringa  | Nessuno           | Blocca l&#x27;esposizione per il modello di fotocamera (Pin 1)                                                 |
| `--exposure-pin-2`    | Stringa  | Nessuno           | Blocco dell&#x27;esposizione per il modello di fotocamera (Pin 2)                                                 |
| `--recal-interval`    | Intero | Auto           | Intervallo di ricalibrazione in secondi                                                      |
| `--timezone-offset`   | Intero | 0              | Offset del fuso orario in ore                                                               |

***

### `login` - Autenticazione account

Accedi con le tue credenziali Chloros+ per abilitare l&#x27;elaborazione CLI.

**Sintassi:**

```bash
chloros-cli login <email> <password>
```

**Esempio:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caratteri speciali**: utilizzare le virgolette singole intorno alle password che contengono caratteri come `$`, `!` o spazi.
{% endhint %}

**Output:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Cancella credenziali

Cancella le credenziali memorizzate ed esci dal tuo account.

**Sintassi:**

```bash
chloros-cli logout
```

**Esempio:**

```bash
chloros-cli logout
```

**Output:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**Utenti SDK**: Python SDK fornisce anche un metodo `logout()` programmatico per cancellare le credenziali all&#x27;interno degli script Python. Per ulteriori dettagli, consultare la [documentazione di Python SDK](api-python-sdk.md#logout) per i dettagli.
{% endhint %}

***

### `status` - Verifica dello stato della licenza

Visualizza lo stato attuale della licenza e dell&#x27;autenticazione.

**Sintassi:**

```bash
chloros-cli status
```

**Esempio:**

```bash
chloros-cli status
```

**Output:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Verifica dello stato di avanzamento dell&#x27;esportazione

Monitora lo stato di avanzamento dell&#x27;esportazione del thread 4 durante o dopo l&#x27;elaborazione.

**Sintassi:**

```bash
chloros-cli export-status
```

**Esempio:**

```bash
chloros-cli export-status
```

**Caso d&#x27;uso:** Richiamare questo comando mentre l&#x27;elaborazione è in corso per verificare lo stato di avanzamento dell&#x27;esportazione.***

### `language` - Gestione della lingua dell&#x27;interfaccia

Visualizza o modifica la lingua dell&#x27;interfaccia CLI.

**Sintassi:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Esempi:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Lingue supportate (38 in totale)

| Codice    | Lingua              | Nome nativo      |
| ------- | --------------------- | ---------------- |
| `en`    | Inglese               | English          |
| `es`    | Spagnolo               | Español          |
| `pt`    | Portoghese            | Português        |
| `fr`    | Francese                | Français         |
| `de`    | Tedesco                | Deutsch          |
| `it`    | Italiano               | Italiano         |
| `ja`    | Giapponese              | 日本語              |
| `ko`    | Coreano                | 한국어              |
| `zh`    | Cinese (semplificato)  | 简体中文             |
| `zh-TW` | Cinese (tradizionale) | 繁體中文             |
| `ru`    | Russo               | Русский          |
| `nl`    | Olandese                 | Nederlands       |
| `ar`    | Arabo                | العربية          |
| `pl`    | Polacco                | Polski           |
| `tr`    | Turco               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonesiano            | Bahasa Indonesia |
| `vi`    | Vietnamita            | Tiếng Việt       |
| `th`    | Tailandese                  | ไทย              |
| `sv`    | Svedese               | Svenska          |
| `da`    | Danese                | Dansk            |
| `no`    | Norvegese             | Norsk            |
| `fi`    | Finlandese               | Suomi            |
| `el`    | Greco                 | Ελληνικά         |
| `cs`    | Ceco                 | Čeština          |
| `hu`    | Ungherese             | Magyar           |
| `ro`    | Rumeno              | Română           |
| `uk`    | Ucraino             | Українська       |
| `pt-BR` | Portoghese brasiliano  | Português Brasileiro |
| `zh-HK` | Cantonese             | 粵語             |
| `ms`    | Malese                | Bahasa Melayu    |
| `sk`    | Slovacco                | Slovenčina       |
| `bg`    | Bulgaro             | Български        |
| `hr`    | Croato              | Hrvatski         |
| `lt`    | Lituano            | Lietuvių         |
| `lv`    | Lettone               | Latviešu         |
| `et`    | Estone              | Eesti            |
| `sl`    | Sloveno             | Slovenščina      |

{% hint style="success" %}
**Persistenza automatica**: La preferenza di lingua viene salvata in `~/.chloros/cli_language.json` e rimane attiva in tutte le sessioni.
{% endhint %}

***

### `set-project-folder` - Imposta cartella progetto predefinita

Modifica la posizione della cartella progetto predefinita (condivisa con la GUI su Windows).

**Sintassi:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Esempi:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Mostra cartella del progetto

Visualizza il percorso corrente della cartella del progetto predefinita.

**Sintassi:**

```bash
chloros-cli get-project-folder
```

**Esempio:**

```bash
chloros-cli get-project-folder
```

**Output:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Ripristina impostazioni predefinite

Ripristina la cartella del progetto nella posizione predefinita.

**Sintassi:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Esegui diagnostica di sistema

Esegue 7 controlli diagnostici per verificare la configurazione del sistema.

**Sintassi:**

```bash
chloros-cli selftest
```

**Diagnostiche eseguite:**

1. Controllo della versione
2. Disponibilità della porta (5000)
3. Avvio del backend
4. Test di connettività API
5. Informazioni di sistema e rilevamento della GPU
6. Verifica dei modelli di denoiser
7. Controllo della disponibilità di CUDA

{% hint style="info" %}
**Utile per la risoluzione dei problemi**: Eseguire `selftest` dopo l&#x27;installazione per verificare che il sistema sia configurato correttamente, in particolare su Linux/Jetson dove potrebbe essere necessaria la verifica della configurazione della GPU e di CUDA.
{% endhint %}

***

### `update` - Verifica aggiornamenti (solo Linux)

Verifica e installa gli aggiornamenti CLI sui sistemi Linux.

**Sintassi:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Opzione    | Descrizione                        |
| --------- | ---------------------------------- |
| `--check` | Verifica solo gli aggiornamenti, non installarli |

{% hint style="info" %}
Questo comando è disponibile solo su Linux. Su Windows, gli aggiornamenti vengono forniti tramite il programma di installazione.
{% endhint %}

***

## Opzioni globali

Queste opzioni si applicano a tutti i comandi:

| Opzione            | Tipo    | Predefinito       | Descrizione                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Percorso    | Rilevato automaticamente | Percorso dell&#x27;eseguibile del backend                       |
| `--port`          | Intero | 5000          | Numero di porta del backend API                          |
| `--restart`       | Flag    | -             | Forza il riavvio del backend (termina i processi esistenti) |
| `--version`       | Flag    | -             | Mostra le informazioni sulla versione ed esci                |
| `--help`          | Flag    | -             | Mostra le informazioni di aiuto ed esci                   |

{% hint style="info" %}
**Rilevamento automatico del backend**: Il percorso `--backend-exe` viene rilevato automaticamente in base alla piattaforma:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuale)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Esempio con opzioni globali:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Guida alle impostazioni di elaborazione

### Elaborazione parallela e adattamento dinamico del calcolo

Chloros 1.1.0 include [Adattamento dinamico del calcolo](processing-architecture/dynamic-compute-adaptation.md): il motore di elaborazione **rileva automaticamente l&#x27;hardware** e seleziona la strategia ottimale:

| Piattaforma | Strategia | Worker | Pipeline | Note |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Efficiente in termini di memoria, serializzata |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Elaborazione GPU simultanea |
| **Desktop con GPU da 8 GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Buone prestazioni desktop |
| **Desktop con GPU da 12 GB+** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Prestazioni desktop ottimali |
| **Sistema solo CPU** | `CPU_PARALLEL` | core - 1 | `cpu_fallback` | Nessuna GPU richiesta |

{% hint style="success" %}
**Non è necessaria alcuna configurazione manuale!** Chloros rileva automaticamente CPU, GPU, RAM e (su Jetson) sensori termici, quindi configura automaticamente la pipeline di elaborazione ottimale.
{% endhint %}

### Metodi di debayering

| Metodo | CLI Flag | Qualità | Velocità | Licenza |
| --- | --- | --- | --- | --- |
| **Standard (Veloce, Qualità media)** | `--debayer standard` | Buona | Veloce | Gratuito / Chloros+ |
| **Sensibile alla texture (Lento, Massima qualità)** | `--debayer texture-aware` | Massima | Lento | Solo Chloros+ |

Il metodo di debayering predefinito è **Standard**. Il metodo**Sensibile alla texture** utilizza un modello di denoising AI/ML per ottenere un risultato di massima qualità, ma richiede una licenza Chloros+ e una GPU NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Correzione della vignettatura

**Cosa fa:** corregge la caduta di luce ai bordi dell&#x27;immagine (angoli più scuri comuni nelle immagini delle fotocamere).

* **Abilitato di default** - La maggior parte degli utenti dovrebbe mantenerlo abilitato
* Utilizzare `--no-vignette` per disabilitarlo

{% hint style="success" %}
**Raccomandazione**: Abilitare sempre la correzione della vignettatura per garantire una luminosità uniforme su tutto il fotogramma.
{% endhint %}

### Calibrazione della riflettanza

Converte i valori grezzi del sensore in percentuali di riflettanza standardizzate utilizzando pannelli di calibrazione.

* **Abilitato di default** - Essenziale per l&#x27;analisi della vegetazione
* Richiede pannelli di calibrazione nelle immagini
* Utilizzare `--no-reflectance` per disabilitare

{% hint style="info" %}
**Requisiti**: Assicurarsi che i pannelli di calibrazione siano correttamente esposti e visibili nelle immagini per una conversione accurata della riflettanza.
{% endhint %}

### Correzioni PPK

**Cosa fa:** Applica correzioni cinematiche post-elaborate utilizzando i dati di registro DAQ-A-SD per una maggiore precisione GPS.

* **Disabilitato per impostazione predefinita**
* Utilizzare `--ppk` per abilitare
* Richiede file .daq nella cartella del progetto provenienti dal sensore di luce DAQ-A-SD MAPIR.

### Formati di output

<table><thead><tr><th width="197">Formato</th><th width="130.20001220703125">Profondità di bit</th><th width="116.5999755859375">Dimensione file</th><th>Ideale per</th></tr></thead><tbody><tr><td><strong>TIFF (16 bit)</strong> ⭐</td><td>Intero a 16 bit</td><td>Grande</td><td>Analisi GIS, fotogrammetria (consigliato)</td></tr><tr><td><strong>TIFF (32 bit, percentuale)</strong></td><td>32 bit in virgola mobile</td><td>Molto grande</td><td>Analisi scientifica, ricerca</td></tr><tr><td><strong>PNG (8 bit)</strong></td><td>Intero a 8 bit</td><td>Medio</td><td>Ispezione visiva, condivisione web</td></tr><tr><td><strong>JPG (8 bit)</strong></td><td>Intero a 8 bit</td><td>Piccola</td><td>Anteprima rapida, output compresso</td></tr></tbody></table>***

## Automazione e scripting

### Elaborazione batch con PowerShell (Windows)

Elaborazione automatica di più cartelle di set di dati su Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Script batch Windows (Windows)

Ciclo semplice per l&#x27;elaborazione batch su Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Elaborazione batch Bash (Linux)

Elaborazione di più cartelle di set di dati su Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Script di automazione Python (multipiattaforma)

Automazione avanzata con gestione degli errori (funziona su Windows e Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Flusso di lavoro di elaborazione

### Flusso di lavoro standard

1. **Input**: Cartella contenente coppie di immagini RAW/JPG
2. **Rilevamento**: CLI esegue la scansione automatica dei file immagine supportati
3. **Elaborazione**: La modalità parallela si adatta ai core della CPU (Chloros+)
4. **Output**: Crea sottocartelle per modello di fotocamera con le immagini elaborate

### Esempio di struttura dell&#x27;output

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Stime dei tempi di elaborazione

Tempi di elaborazione tipici per 100 immagini (12 MP ciascuna):

| Piattaforma | Modalità | Tempo stimato | Note |
| --- | --- | --- | --- |
| **Desktop con GPU da 12 GB+** | `GPU_PARALLEL` | 5-10 min | Opzione più veloce |
| **Desktop con GPU da 8 GB** | `GPU_SINGLE` | 10-15 min | Buone prestazioni |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15-25 min | Elaborazione edge |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30-60 min | Memoria limitata |
| **Solo CPU** | `CPU_PARALLEL` | 20-40 min | Nessuna GPU richiesta |

{% hint style="info" %}
**Suggerimento sulle prestazioni**: Il tempo di elaborazione varia in base al numero di immagini, alla risoluzione, al metodo di debayering e all&#x27;hardware. Il debayering Texture Aware richiede molto più tempo rispetto a quello Standard. Per ulteriori dettagli, consultare [Adattamento dinamico dell&#x27;elaborazione](processing-architecture/dynamic-compute-adaptation.md).
{% endhint %}

***

## Risoluzione dei problemi

### CLI non trovato

**Errore Windows:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Soluzioni per Windows:**

1. Verificare la posizione di installazione:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Utilizzare il percorso completo se non è presente in PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Aggiungere manualmente a PATH:
   * Aprire Proprietà del sistema → Variabili d&#x27;ambiente
   * Modificare la variabile PATH
   * Aggiungere: `C:\Program Files\Chloros\resources\cli`
   * Riavviare il terminale

**Errore Linux:**

```
chloros-cli: command not found
```

**Soluzioni per Linux:**

1. Verifica l&#x27;installazione:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Ricarica la shell:

```bash
source ~/.bashrc
```

3. Controlla i permessi:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Impossibile avviare il backend**Errore:**

```

Backend failed to start within 30 seconds
```

**Soluzioni:**

1. Verificare se il backend è già in esecuzione (chiuderlo prima)
2. Verifica che il firewall non lo stia bloccando (Windows) o controlla la disponibilità della porta (Linux: `lsof -i :5000`)
3. Prova una porta diversa:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Forzare il riavvio del backend:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Su Linux, verificare che l&#x27;eseguibile del backend esista:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Problemi relativi alla licenza / autenticazione**Errore:**

```

Chloros+ license required for CLI access
```

**Soluzioni:**

1. Verificare di disporre di un abbonamento Chloros+ attivo
2. Effettuare l&#x27;accesso con le proprie credenziali:

```bash
chloros-cli login user@example.com 'password'
```

3. Controlla lo stato della licenza:

```bash
chloros-cli status
```

4. Contatta l&#x27;assistenza: info@mapir.camera

***

### Nessuna immagine trovata**Errore:**

```

No images found in the specified folder
```

**Soluzioni:**

1. Verifica che la cartella contenga formati supportati (.RAW, .TIF, .JPG)
2. Controlla che il percorso della cartella sia corretto (usa le virgolette per i percorsi con spazi)
3. Assicurati di avere i permessi di lettura per la cartella
4. Controlla che le estensioni dei file siano corrette

***

### Elaborazione in stallo o bloccata**Soluzioni:**

1. Controllare lo spazio disponibile su disco (assicurarsi che sia sufficiente per l&#x27;output)
2. Chiudere le altre applicazioni per liberare memoria
3. Ridurre il numero di immagini (elaborare in batch)

***

### Porta già in uso**Errore:**

```

Port 5000 is already in use
```

**Soluzioni:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Domande frequenti

### D: È necessaria una licenza per CLI?

**R:**Sì! CLI richiede una**licenza Chloros+** a pagamento.

* ❌ Piano Standard (gratuito): CLI disabilitato
* ✅ Piani Chloros+ (a pagamento): CLI completamente abilitato

Abbonati su: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### D: Posso utilizzare CLI su un server senza GUI?**R:** Sì! CLI funziona completamente in modalità headless. Questo è il caso d&#x27;uso principale su Linux.**Server Windows:**
* Server Windows 2016 o versioni successive
* Visual C++ Redistributable installato

**Server Linux:**
* Ubuntu 20.04+ / Debian 11+ (amd64) o JetPack 6 (arm64)
* Installazione tramite pacchetto `.deb`

**Entrambe le piattaforme:**
* Minimo 8 GB di RAM (consigliati 16 GB)
* Attivazione della licenza una tantum: `chloros-cli login user@example.com 'password'`

***

### D: Dove vengono salvate le immagini elaborate?**R:**Per impostazione predefinita, le immagini elaborate vengono salvate nella**stessa cartella di input** in sottocartelle relative al modello di fotocamera (ad es. `Survey3N_RGN/`).

Utilizzare l&#x27;opzione `-o` per specificare una cartella di output diversa:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### D: È possibile elaborare più cartelle contemporaneamente?**R:** Non direttamente con un unico comando, ma è possibile utilizzare gli script per elaborare le cartelle in sequenza. Vedere la sezione [Automazione e scripting](CLI.md#automation--scripting).***

### D: Come posso salvare l&#x27;output di CLI in un file di log?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### D: Cosa succede se premo Ctrl+C durante l&#x27;elaborazione?**R:** CLI:

1. Interromperà l&#x27;elaborazione in modo corretto
2. Chiuderà il backend
3. Uscirà con codice 130

Le immagini elaborate parzialmente potrebbero rimanere nella cartella di output.

***

### D: Posso automatizzare l&#x27;elaborazione di CLI?**R:** Certamente! CLI è progettato per l&#x27;automazione. Vedi [Automazione e scripting](CLI.md#automation--scripting) per gli esempi relativi a PowerShell (Windows), Batch (Windows), Bash (Linux) e Python (multipiattaforma).***

### D: Come posso verificare la versione di CLI?**R:**

```bash
chloros-cli --version
```

**Output:**

```

Chloros CLI 1.1.0
```

***

## Ottenere assistenza

### Guida dalla riga di comando

Visualizza le informazioni di aiuto direttamente in CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Canali di assistenza

* **E-mail**: info@mapir.camera
* **Sito web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Prezzi**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Esempi completi

### Esempio 1: Elaborazione di base

Elaborazione con impostazioni predefinite (vignettatura, riflettanza):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Esempio 2: Output scientifico di alta qualità

32 bit in virgola mobile TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Esempio 3: Elaborazione rapida dell&#x27;anteprima

PNG a 8 bit senza calibrazione per una rapida revisione:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Esempio 4: Elaborazione con correzione PPK

Applicare correzioni PPK con riflettanza:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Esempio 5: Posizione di output personalizzata

Elaborazione in una posizione diversa con formato specifico:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Esempio 6: Flusso di autenticazione

Flusso di autenticazione completo (uguale su tutte le piattaforme):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Esempio 7: Utilizzo multilingue

Modifica della lingua dell&#x27;interfaccia (uguale su tutte le piattaforme):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
