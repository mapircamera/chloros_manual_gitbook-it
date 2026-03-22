# Guida NVIDIA Jetson

Chloros su NVIDIA Jetson consente l&#x27;elaborazione di immagini multispettrali in locale — sul campo, su UAV e in installazioni remote. Chloros rileva automaticamente il modello Jetson in uso e ottimizza la strategia di elaborazione in base all&#x27;hardware.

***

## Modelli Jetson supportati

| Modello                | RAM            | Strategia di elaborazione                                   | Uso consigliato                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB condivisi | `GPU_PARALLEL` (4 worker)                            | Massime prestazioni, set di dati di grandi dimensioni                      |
| **Jetson Orin NX**   | 8-16 GB condivisa  | `GPU_PARALLEL` (3 worker, 16 GB) / `GPU_SINGLE` (8 GB) | Raccomandazione principale per implementazioni aeree e sul campo |
| **Jetson Orin Nano** | 8 GB condivisi     | `GPU_SINGLE` (1 worker)                               | Elaborazione edge entry-level                                 |
| **Jetson Nano**      | 4-8 GB condivisi   | `GPU_SINGLE` (1 worker)                               | Livello base, con memoria limitata                          |

{% hint style="info" %}
**I modelli Jetson legacy** (TX2, TX1, Xavier NX) potrebbero non essere supportati. Le prestazioni variano in base alla memoria GPU disponibile e alle funzionalità CUDA.
{% endhint %}

***

## Requisiti

* **JetPack 6.x** (consigliata l&#x27;ultima versione)
* **NVIDIA CUDA** (inclusa in JetPack)
* **Licenza Chloros+** (richiesta per l&#x27;accesso a CLI/SDK)

## Installazione

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Per i dettagli generali sull&#x27;installazione di Linux, consultare [Installazione di Linux](linux-installation.md).

***

## Adattamento dinamico del calcolo su Jetson

Chloros rileva automaticamente il modello Jetson in uso e seleziona la strategia di elaborazione ottimale. **Non è richiesta alcuna regolazione manuale.**

### Come funziona

All&#x27;avvio, Chloros esegue il profiling del sistema:

1. **Rileva il modello Jetson** tramite `/proc/device-tree/model`
2. **Legge la memoria GPU/condivisa disponibile**

3.**Seleziona una strategia di elaborazione** (`GPU_PARALLEL`, `GPU_SINGLE` o `CPU_PARALLEL`)
4. **Imposta automaticamente il numero di worker, il tipo di pipeline e l&#x27;allocazione di memoria**

### Comportamento per modello

| Modello Jetson                | Strategia       | Worker | Pipeline                       | Concorrenza |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (efficiente in termini di memoria) | Serializzato  |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Serializzato  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serializzato  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (percorso GPU completo)    | Concorrente  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Concorrente  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** rappresenta il punto di equilibrio ideale per l&#x27;implementazione edge: adotta la strategia `GPU_PARALLEL` con 3 worker simultanei, garantendo un&#x27;elaborazione GPU realmente parallela in un fattore di forma compatto.
{% endhint %}

La differenza fondamentale tra le piattaforme è la **memoria**. Un Jetson Nano con 8 GB di memoria condivisa deve elaborare le immagini una alla volta utilizzando un approccio a tessere efficiente in termini di memoria, mentre un Orin NX con 16 GB può elaborare 3 immagini contemporaneamente tramite la GPU utilizzando la pipeline fusa a throughput più elevato.

Per il riferimento completo sull&#x27;adattamento computazionale, consultare [Adattamento computazionale dinamico](../processing-architecture/dynamic-compute-adaptation.md).

***

## Gestione termica

I dispositivi Jetson hanno un margine termico limitato, specialmente in implementazioni in ambienti chiusi o a bordo di velivoli. Chloros include il monitoraggio termico automatico e la limitazione della potenza:

| Temperatura         | Azione                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Funzionamento normale — velocità di elaborazione massima          |
| **70 °C** (Avviso)  | Riduzione automatica della dimensione del batch                   |
| **80 °C** (Critico) | Limitazione aggressiva — minore concorrenza         |
| **90°C** (Spegnimento) | Arresto completo dell&#x27;elaborazione della GPU — raffreddamento richiesto |

{% hint style="warning" %}
**Garantire una ventilazione e un dissipatore di calore adeguati** per un&#x27;elaborazione prolungata, specialmente in involucri da campo chiusi o sistemi aerei. La limitazione termica ridurrà la velocità di elaborazione per proteggere l&#x27;hardware.
{% endhint %}

***

## Gestione della memoria

I dispositivi Jetson utilizzano la **memoria unificata**: la GPU e la CPU condividono la stessa RAM fisica. Ciò significa che la VRAM riportata (ad es. 15,3 GB su Orin NX 16 GB) non è memoria dedicata alla GPU, ma è condivisa con il sistema operativo e altri processi.

### Raccomandazioni sullo swap

Per set di dati di grandi dimensioni o elaborazione debayer Texture Aware, Chloros potrebbe raccomandare la creazione di spazio di swap:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Stime di memoria per immagine:**

* Debayer standard: ~10 MB per immagine
* Debayer Texture Aware: ~15 MB per immagine

Chloros calcola automaticamente la memoria richiesta in base alle dimensioni del set di dati e avvisa se è consigliabile lo swap.

### Fallback OOM (Out of Memory)

Se durante l&#x27;elaborazione viene rilevata una condizione di memoria insufficiente:

1. Chloros riduce automaticamente il numero di worker della GPU
2. Passa dalla pipeline `fused_gpu` a quella `tiled_gpu` (più efficiente in termini di memoria)
3. Continua l&#x27;elaborazione a una velocità ridotta invece di arrestarsi

***

## Implementazione sul campo

### Considerazioni sull&#x27;alimentazione

| Modello Jetson     | Assorbimento tipico | Note                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10 W              | USB-C o jack cilindrico    |
| Jetson Orin Nano | 7-15 W              | Jack cilindrico CC          |
| Jetson Orin NX   | 10-25 W             | Jack cilindrico CC          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD o jack cilindrico |

Pianificate il vostro budget energetico per un&#x27;elaborazione prolungata: il picco di assorbimento energetico si verifica durante il Thread 3 (Elaborazione), che richiede un uso intensivo della GPU.

### Raccomandazioni relative allo storage

* **SSD NVMe** fortemente raccomandato per le implementazioni arm64
* Le schede SD sono troppo lente per l&#x27;elaborazione: utilizzatele solo come supporto di avvio
* Prevedete uno spazio pari a 2-3 volte la dimensione dei dati grezzi dell&#x27;immagine per l&#x27;output elaborato

### Funzionamento headless tramite SSH

Chloros CLI è ideale per le implementazioni Jetson headless:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Elaborazione automatizzata con systemd

Creare un servizio systemd per l&#x27;elaborazione automatizzata:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Abbinarlo a un timer systemd per l&#x27;elaborazione programmata:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Flussi di lavoro di esempio

### Elaborazione Jetson di base

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK su Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Elaborazione in batch di più voli

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Sistemi Jetson consigliati per l&#x27;uso sul campo

Per le implementazioni sul campo e aeree, prendete in considerazione queste opzioni di schede carrier Jetson Orin NX da 16 GB:

* **Aeree/droni**: sistemi con resistenza alle vibrazioni (MIL-STD), leggeri (meno di 300 g), raffreddamento passivo
* **Ambiente esterno difficile**: involucri impermeabili IP67/IP69K con connettività per telecamera GigE PoE
* **Minimo/economico**: kit di sviluppo con involucri aggiuntivi

Contattare [MAPIR Supporto](https://www.mapir.camera/community/contact) per consigli specifici sull&#x27;hardware in base al proprio scenario di implementazione.

***

## Passi successivi

* [Installazione Linux](linux-installation.md) — Dettagli generali sull&#x27;installazione di Linux
* [Adattamento dinamico della potenza di calcolo](../processing-architecture/dynamic-compute-adaptation.md) — Riferimento completo sulle strategie di calcolo
* [Pipeline di elaborazione](../processing-architecture/processing-pipeline.md) — Informazioni sulla pipeline a 4 thread
* [CLI : Riga di comando](../CLI.md) — Riferimento completo su CLI
* [API : Python SDK](../api-python-sdk.md) — Riferimento completo su SDK
