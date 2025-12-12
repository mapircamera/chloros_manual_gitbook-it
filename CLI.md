# CLI : Riga di comando

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** offre un potente accesso da riga di comando al motore di elaborazione delle immagini Chloros, consentendo l&#x27;automazione, la creazione di script e il funzionamento senza monitor per i flussi di lavoro di imaging.

### Caratteristiche principali

* 🚀 **Automazione** - Elaborazione batch di script di più set di dati
* 🔗 **Integrazione** - Integrazione nei flussi di lavoro e nelle pipeline esistenti
* 💻 **Funzionamento headless** - Esecuzione senza GUI
* 🌍 **Multilingue** - Supporto per 38 lingue
* ⚡ **Elaborazione parallela** - Si adatta dinamicamente alla tua CPU (fino a 16 worker paralleli)

### Requisiti

| Requisito          | Dettagli                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Sistema operativo** | Windows 10/11 (64 bit)                                              |
| **Licenza**          | Chloros+ ([abbonamento a pagamento richiesto](https://cloud.mapir.camera/pricing)) |
| **Memoria**           | Minimo 8 GB di RAM (consigliati 16 GB)                                  |
| **Internet**         | Necessario per l&#x27;attivazione della licenza                                     |
| **Spazio su disco**       | Varia in base alle dimensioni del progetto                                              |

{% suggerimento style=&quot;warning&quot; %}
**Requisiti di licenza**: CLI richiede un abbonamento a pagamento a Chloros+. I piani standard (gratuiti) non consentono l&#x27;accesso a CLI. Visita [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) per effettuare l&#x27;aggiornamento.
{% endhint %}

## Avvio rapido

### Installazione

CLI è automaticamente incluso nell&#x27;installatore Chloros:

1. Scarica ed esegui **Chloros Installer.exe**
2. Completa la procedura guidata di installazione
3. CLI installato in: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Il programma di installazione aggiunge automaticamente `chloros-cli` al PATH del sistema. Riavviare il terminale dopo l&#x27;installazione.
{% endhint %}

### Configurazione iniziale

Prima di utilizzare CLI, attiva la licenza Chloros+:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Utilizzo di base

Elaborare una cartella con le impostazioni predefinite:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Riferimento ai comandi

### Sintassi generale

```
chloros-cli [global-options] <command> [command-options]
```

***

## Comandi

### `process` - Elaborare immagini

Elabora le immagini in una cartella con calibrazione.

**Sintassi:**

```bash
chloros-cli process <input-folder> [options]
```

**Esempio:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Opzioni del comando di elaborazione

| Opzione                | Tipo    | Impostazione predefinita        | Descrizione                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Percorso    | _Obbligatorio_     | Cartella contenente immagini multispettrali RAW/JPG                                         |
| `-o, --output`        | Percorso    | Uguale all&#x27;input  | Cartella di output per le immagini elaborate                                                     |
| `-n, --project-name`  | Stringa  | Generato automaticamente | Nome progetto personalizzato                                                                    |
| `--vignette`          | Flag    | Abilitato        | Abilita correzione vignettatura                                                             |
| `--no-vignette`       | Flag    | -              | Disabilita correzione vignettatura                                                            |
| `--reflectance`       | Flag    | Abilitato        | Abilita calibrazione riflettanza                                                         |
| `--no-reflectance`    | Flag    | -              | Disabilita calibrazione riflettanza                                                        |
| `--ppk`               | Flag    | Disabilitato       | Applica correzioni PPK dai dati del sensore di luce .daq                                      |
| `--format`            | Scelta  | TIFF (16 bit)  | Formato di output: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Intero | Auto           | Dimensione minima target in pixel per il rilevamento del pannello di calibrazione                          |
| `--target-clustering` | Intero | Auto           | Soglia di raggruppamento target (0-100)                                                    |
| `--exposure-pin-1`    | Stringa  | Nessuna           | Blocco esposizione per modello fotocamera (Pin 1)                                                 |
| `--exposure-pin-2`    | Stringa  | Nessuna           | Blocco esposizione per modello fotocamera (Pin 2)                                                 |
| `--recal-interval`    | Intero | Auto           | Intervallo di ricalibrazione in secondi                                                      |
| `--timezone-offset`   | Intero | 0              | Offset fuso orario in ore                                                               |

***

### `login` - Autenticazione account

Accedi con le tue credenziali Chloros+ per abilitare l&#x27;elaborazione CLI.

**Sintassi:**

```bash
chloros-cli login <email> <password>
```

**Esempio:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Caratteri speciali**: utilizzare virgolette singole attorno alle password contenenti caratteri come `$`, `!` o spazi.
{% endhint %}

**Output:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Cancella credenziali

Cancella le credenziali memorizzate ed esci dal tuo account.

**Sintassi:**

```bash
chloros-cli logout
```

**Esempio:**

```powershell
chloros-cli logout
```

**Output:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - Controlla lo stato della licenza

Visualizza la licenza corrente e lo stato di autenticazione.

**Sintassi:**

```bash
chloros-cli status
```

**Esempio:**

```powershell
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

### `export-status` - Controlla lo stato di avanzamento dell&#x27;esportazione

Monitora lo stato di avanzamento dell&#x27;esportazione del thread 4 durante o dopo l&#x27;elaborazione.

**Sintassi:**

```bash
chloros-cli export-status
```

**Esempio:**

```powershell
chloros-cli export-status
```

**Caso d&#x27;uso:** richiamare questo comando durante l&#x27;esecuzione dell&#x27;elaborazione per controllare lo stato di avanzamento dell&#x27;esportazione.

***

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

```powershell
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
| `th`    | Thailandese                  | ไทย              |
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
| `ms`    | Malese                 | Bahasa Melayu    |
| `sk`    | Slovacco                | Slovenčina       |
| `bg`    | Bulgaro             | Български        |
| `hr`    | Croato              | Hrvatski         |
| `lt`    | Lituano            | Lietuvių         |
| `lv`    | Lettone               | Latviešu         |
| `et`    | Estone              | Eesti            |
| `sl`    | Sloveno             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Persistenza automatica**: la lingua preferita viene salvata in `~/.chloros/cli_language.json` e rimane attiva per tutte le sessioni.
{% endhint %}

***

### `set-project-folder` - Imposta cartella progetto predefinita

Modifica la posizione predefinita della cartella del progetto (condivisa con GUI).

**Sintassi:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Esempio:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Mostra cartella di progetto

Visualizza la posizione corrente della cartella di progetto predefinita.

**Sintassi:**

```bash
chloros-cli get-project-folder
```

**Esempio:**

```powershell
chloros-cli get-project-folder
```

**Output:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Ripristina impostazioni predefinite

Ripristina la cartella del progetto nella posizione predefinita.

**Sintassi:**

```bash
chloros-cli reset-project-folder
```

***

## Opzioni globali

Queste opzioni si applicano a tutti i comandi:

| Opzione          | Tipo    | Predefinita       | Descrizione                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Percorso    | Rilevato automaticamente | Percorso dell&#x27;eseguibile backend                       |
| `--port`        | Intero | 5000          | Numero porta backend API                          |
| `--restart`     | Flag    | -             | Forza il riavvio del backend (termina i processi esistenti) |
| `--version`     | Flag    | -             | Mostra le informazioni sulla versione e chiude                |
| `--help`        | Flag    | -             | Mostra le informazioni di aiuto e chiude                   |

**Esempio con opzioni globali:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Guida alle impostazioni di elaborazione

### Elaborazione parallela

Chloros+ CLI **adatta automaticamente** l&#x27;elaborazione parallela alle capacità del computer:

**Come funziona:**

* Rileva i core della CPU e la RAM
* Assegna i worker: **2× core della CPU** (utilizza l&#x27;hyperthreading)
* **Massimo: 16 worker paralleli** (per garantire la stabilità)

**Livelli di sistema:**

| Tipo di sistema   | CPU        | RAM      | Worker  | Prestazioni     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **High-End**  | 16+ core  | 32+ GB   | Fino a 16 | Velocità massima   |
| **Fascia media** | 8-15 core | 16-31 GB | 8-16     | Velocità eccellente |
| **Fascia bassa**   | 4-7 core  | 8-15 GB  | 4-8      | Velocità buona      |

{% hint style=&quot;success&quot; %}
**Ottimizzazione automatica**: CLI rileva automaticamente le specifiche del sistema e configura l&#x27;elaborazione parallela ottimale. Non è necessaria alcuna configurazione manuale!
{% endhint %}

### Metodi Debayer

CLI utilizza **Alta qualità (più veloce)** come algoritmo Debayer predefinito e consigliato:

| Metodo                      | Qualità | Velocità | Descrizione                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Alta qualità (più veloce)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Algoritmo sensibile ai bordi (impostazione predefinita, consigliata) |

### Correzione vignettatura

**Cosa fa:** corregge la caduta di luce ai bordi dell&#x27;immagine (angoli più scuri comuni nelle immagini delle fotocamere).

* **Abilitata per impostazione predefinita** - La maggior parte degli utenti dovrebbe mantenerla abilitata
* Utilizzare `--no-vignette` per disabilitarla

{% hint style=&quot;success&quot; %}
**Raccomandazione**: abilitare sempre la correzione vignetta per garantire una luminosità uniforme in tutto il fotogramma.
{% endhint %}

### Calibrazione riflettanza

Converte i valori grezzi del sensore in percentuali di riflettanza standardizzate utilizzando pannelli di calibrazione.

* **Abilitata per impostazione predefinita** - Essenziale per l&#x27;analisi della vegetazione.
* Richiede pannelli di calibrazione nelle immagini.
* Utilizzare `--no-reflectance` per disabilitare.

{% hint style=&quot;info&quot; %}
**Requisiti**: assicurarsi che i pannelli di calibrazione siano correttamente esposti e visibili nelle immagini per una conversione accurata della riflettanza.
{% endhint %}

### Correzioni PPK

**Cosa fa:** applica correzioni cinematiche post-elaborate utilizzando i dati di registro DAQ-A-SD per una maggiore precisione GPS.

* **Disabilitato per impostazione predefinita**
* Utilizzare `--ppk` per abilitare
* Richiede file .daq nella cartella del progetto dal sensore di luce MAPIR DAQ-A-SD.

### Formati di output

<table><thead><tr><th width="197">Formato</th><th width="130.20001220703125">Profondità di bit</th><th width="116.5999755859375">Dimensione file</th><th>Ideale per</th></tr></thead><tbody><tr><td><strong>TIFF (16 bit)</strong> ⭐</td><td>Intero a 16 bit</td><td>Grande</td><td>Analisi GIS, fotogrammetria (consigliato)</td></tr><tr><td><strong>TIFF (32 bit, percentuale)</strong></td><td>32 bit in virgola mobile</td><td>Molto grande</td><td>Analisi scientifica, ricerca</td></tr><tr><td><strong>PNG (8 bit)</strong></td><td>Intero a 8 bit</td><td>Medio</td><td>Ispezione visiva, condivisione web</td></tr><tr><td><strong>JPG (8 bit)</strong></td><td>Intero a 8 bit</td><td>Piccolo</td><td>Anteprima rapida, output compresso</td></tr></tbody></table>***

## Automazione e scripting

### Elaborazione batch PowerShell

Elaborazione automatica di più cartelle di set di dati:

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

### Windows Script batch

Semplice ciclo per l&#x27;elaborazione batch:

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

### Python Script di automazione

Automazione avanzata con gestione degli errori:

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

1. **Input**: cartella contenente coppie di immagini RAW/JPG
2. **Rilevamento**: CLI esegue la scansione automatica dei file immagine supportati
3. **Elaborazione**: la modalità parallela si adatta ai core della CPU (Chloros+)
4. **Output**: crea sottocartelle per modello di fotocamera con le immagini elaborate

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

| Modalità              | Tempo      | Hardware                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Modalità parallela** | 5-10 min  | i7/Ryzen 7, 16 GB di RAM, SSD (fino a 16 lavoratori) |
| **Modalità parallela** | 10-15 min | i5/Ryzen 5, 8 GB di RAM, HDD (fino a 8 worker)   |

{% hint style=&quot;info&quot; %}
**Suggerimento sulle prestazioni**: il tempo di elaborazione varia in base al numero di immagini, alla risoluzione e alle specifiche del computer.
{% endhint %}

***

## Risoluzione dei problemi

### CLI non trovato

**Errore:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Soluzioni:**

1. Verificare la posizione di installazione:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Utilizzare il percorso completo se non presente in PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Aggiungere manualmente a PATH:
   * Aprire Proprietà del sistema → Variabili d&#x27;ambiente
   * Modificare la variabile PATH
   * Aggiungere: `C:\Program Files\Chloros\resources\cli`
   * Riavvia il terminale

***

### Impossibile avviare il backend

**Errore:**

```
Backend failed to start within 30 seconds
```

**Soluzioni:**

1. Verifica se il backend è già in esecuzione (chiudilo prima)
2. Verifica che Windows il firewall non lo stia bloccando
3. Prova una porta diversa:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Forza il riavvio del backend:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Problemi di licenza/autenticazione

**Errore:**

```
Chloros+ license required for CLI access
```

**Soluzioni:**

1. Verificare di disporre di un abbonamento Chloros+ attivo
2. Accedere con le proprie credenziali:

```powershell
chloros-cli login user@example.com 'password'
```

3. Controllare lo stato della licenza:

```powershell
chloros-cli status
```

4. Contattare l&#x27;assistenza: info@mapir.camera

***

### Nessuna immagine trovata

**Errore:**

```
No images found in the specified folder
```

**Soluzioni:**

1. Verificare che la cartella contenga formati supportati (.RAW, .TIF, .JPG)
2. Verificare che il percorso della cartella sia corretto (utilizzare le virgolette per i percorsi con spazi)
3. Assicurarsi di disporre dei permessi di lettura per la cartella
4. Verificare che le estensioni dei file siano corrette

***

### Elaborazione bloccata o in sospeso

**Soluzioni:**

1. Verificare lo spazio disponibile su disco (assicurarsi che sia sufficiente per l&#x27;output)
2. Chiudere le altre applicazioni per liberare memoria
3. Ridurre il numero di immagini (elaborare in batch)

***

### Porta già in uso

**Errore:**

```
Port 5000 is already in use
```

**Soluzione:**

Specificare una porta diversa:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Domande frequenti

### D: È necessaria una licenza per CLI?

**R:** Sì! CLI richiede una **licenza Chloros+ a pagamento**.

* ❌ Piano Standard (gratuito): CLI disabilitato
* ✅ Piani Chloros+ (a pagamento): CLI completamente abilitato

Abbonati su: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### D: Posso utilizzare CLI su un server senza GUI?

**R:** Sì! CLI funziona completamente senza interfaccia grafica. Requisiti:

* Windows Server 2016 o versioni successive
* Visual C++ Redistributable installato
* RAM sufficiente (minimo 8 GB, consigliati 16 GB)
* Attivazione della licenza GUI una tantum su qualsiasi macchina

***

### D: Dove vengono salvate le immagini elaborate?

**R:** Per impostazione predefinita, le immagini elaborate vengono salvate nella **stessa cartella dell&#x27;input** nelle sottocartelle del modello di fotocamera (ad esempio, `Survey3N_RGN/`).

Utilizzare l&#x27;opzione `-o` per specificare una cartella di output diversa:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### D: È possibile elaborare più cartelle contemporaneamente?

**A:** Non direttamente con un unico comando, ma è possibile utilizzare uno script per elaborare le cartelle in sequenza. Vedere la sezione [Automazione e scripting](CLI.md#automation--scripting).

***

### D: Come posso salvare l&#x27;output di CLI in un file di log?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### D: Cosa succede se premo Ctrl+C durante l&#x27;elaborazione?

**R:** CLI:

1. Interromperà l&#x27;elaborazione in modo corretto
2. Chiuderà il backend
3. Usciremo con il codice 130

Le immagini parzialmente elaborate potrebbero rimanere nella cartella di output.

***

### D: Posso automatizzare l&#x27;elaborazione di CLI?

**R:** Certamente! CLI è progettato per l&#x27;automazione. Vedere [Automazione e scripting](CLI.md#automation--scripting) per esempi di PowerShell, Batch e Python.

***

### D: Come posso verificare la versione di CLI?

**R:**

```powershell
chloros-cli --version
```

**Output:**

```
Chloros CLI 1.0.2
```

***

## Ottenere assistenza

### Guida della riga di comando

Visualizza le informazioni di aiuto direttamente in CLI:

```powershell
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
* **Prezzi**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Esempi completi

### Esempio 1: elaborazione di base

Elaborazione con impostazioni predefinite (vignettatura, riflettanza):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Esempio 2: output scientifico di alta qualità

32 bit float TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Esempio 3: elaborazione rapida dell&#x27;anteprima

8 bit PNG senza calibrazione per una rapida revisione:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Esempio 4: elaborazione con correzione PPK

Applicazione delle correzioni PPK con riflettanza:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Esempio 5: posizione di output personalizzata

Elaborazione su un&#x27;unità diversa con formato specifico:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Esempio 6: flusso di lavoro di autenticazione

Completa il flusso di autenticazione:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Esempio 7: utilizzo multilingue

Cambia la lingua dell&#x27;interfaccia:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
