# Guida NVIDIA Jetson

Chloros
su NVIDIA Jetson consente l&#x27;elaborazione di immagini multispettrali in edge computing — sul campo, su UAV e in installazioni remote. Chloros
1.3.0 rileva il modello Jetson all&#x27;avvio e ottimizza la strategia di elaborazione in base all&#x27;hardware rilevato. **Non è necessaria alcuna regolazione manuale.**
{% hint style="info" %}
**Il pacchetto Jetson 1.3.0 esegue l’elaborazione sulla GPU di Jetson.** `chloros_1.3.0_arm64_jp6.deb` è progettato per JetPack 6 e utilizza la versione CUDA di PyTorch inclusa in JetPack 6, pertanto l’elaborazione, compreso il debayer Texture Aware, viene eseguita sulla GPU di Orin. Il pacchetto non include CUDA in sé: utilizza le librerie CUDA e cuDNN installate da JetPack 6 (`sudo apt install nvidia-jetpack` su un&#x27;immagine che ne è sprovvista). Laddove non sia disponibile una GPU CUDA, l’elaborazione viene eseguita sulla CPU e Texture Aware utilizza il debayer standard.
{% endhint %}

***

## Modelli Jetson supportati

| Modello                | RAM            | Strategia di elaborazione                                     | Uso consigliato                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB condivisi | `GPU_PARALLEL` (2 worker)                              | Massime prestazioni, set di dati di grandi dimensioni                      |
| **Jetson Orin NX**   | 8-16 GB condivisi  | `GPU_PARALLEL` (2 worker, 16 GB) / `GPU_SINGLE` (8 GB)   | Raccomandazione principale per l’implementazione sul campo              |
| **Jetson Orin Nano** | 8 GB condivisi     | `GPU_SINGLE` (1 worker, sequenziale)                     | Elaborazione edge entry-level                                 |

{% hint style="info" %}
Il pacchetto Linux
arm64 richiede **JetPack 6**, disponibile sulla famiglia Jetson Orin. I modelli precedenti (Nano, TX2, Xavier NX) non possono eseguire JetPack 6 e non sono supportati dal pacchetto attuale.
{% endhint %}

***

## Requisiti

* **JetPack 6.x** (si consiglia l’ultima versione)
* **NVIDIA CUDA e cuDNN** da JetPack 6 (`sudo apt install nvidia-jetpack` se la vostra immagine non li include)
* **Piano a pagamento Chloros
+** — livello Copper o superiore (richiesto per tutti gli accessi CLI
/SDK
; applicato lato server)

## Installazione

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

Il file `.deb` installa l’SDK
Python
per il sistema `python3`, corrispondente a questa build. Solo un virtualenv richiede l’installazione del wheel in dotazione al suo interno: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (completamente offline). Per i dettagli generali sull’installazione su Linux
, i percorsi dei file e la risoluzione dei problemi, consultare [Installazione su Linux
](linux-installation.md).

{% hint style="info" %}
**Non è più necessario decomprimere i file all’avvio.** A partire dalla versione 1.3.0, il backend e l’CLI
vengono installati già decompressi in `/usr/lib/chloros/`, quindi non si estraggono più in una directory temporanea ogni volta che vengono avviati. I file temporanei vengono comunque salvati automaticamente in `/mnt/ssd/tmp` se tale cartella esiste; in caso contrario, è possibile impostare `TMPDIR` su un percorso sul proprio NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Adattamento dinamico delle risorse di calcolo su Jetson

### Come funziona

All’avvio, Chloros
crea un profilo del sistema:

1. **Rileva il modello di Jetson** tramite `/proc/device-tree/model`
2. **Legge la memoria condivisa disponibile tra GPU e CPU** (Jetson utilizza la memoria unificata)
3. **Chiede a PyTorch se può utilizzare CUDA** — con CUDA e cuDNN di JetPack 6 installati, è possibile
4. **Seleziona automaticamente una strategia di elaborazione** (`GPU_PARALLEL`, `GPU_SINGLE` o `CPU_PARALLEL`) e il numero di worker

La scelta dipende dalla **RAM condivisa totale**, non dal nome del modello:

* **Con meno di 12 GB di RAM totale**(tutti i Jetson da 8 GB):**1 worker — elaborazione sequenziale intenzionale**. La memoria è insufficiente per i worker concorrenti, quindi le immagini vengono elaborate una alla volta. Sui Jetson con**8 GB o meno**, il Thread 3 ignora completamente il pool di worker ed esegue il proprio lavoro per ogni singola immagine all’interno del processo stesso. Il log del backend denomina questa strategia `GPU_SINGLE`.
* **12 GB o più**(Orin NX da 16 GB, AGX Orin): la memoria unificata soddisfa i requisiti per `GPU_PARALLEL`, ma il numero di worker è**limitato a 2 su Jetson** — la GPU, la RAM dei processi worker e i contesti CUDA per ciascun worker attingono tutti allo stesso pool condiviso, quindi un numero maggiore di worker comporta il rischio di errori dovuti all’esaurimento della memoria.
* **CUDA non utilizzabile** (librerie CUDA di JetPack mancanti): `CPU_PARALLEL` con 12 GB o più, un&#x27;immagine alla volta di seguito.

È possibile sovrascrivere la scelta automatica con la variabile d’ambiente `CHLOROS_STRATEGY` — vedere [Adattamento dinamico del calcolo](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Comportamento per modello

| Modello Jetson                | Strategia       | Worker | Esecuzione                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Ciclo sequenziale in-process (`tiled_gpu` in caso di pressione sulla memoria) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Ciclo sequenziale all&#x27;interno del processo                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Processi di lavoro concorrenti, percorso `fused_gpu`  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2       | Processi di lavoro concorrenti, percorso `fused_gpu`  |

La differenza fondamentale tra le piattaforme è la **memoria**. Un Jetson da 8 GB deve elaborare le immagini una alla volta utilizzando un approccio a tessere efficiente in termini di memoria quando la pressione è elevata, mentre un Orin da 16 GB o più può elaborare 2 immagini contemporaneamente tramite la GPU utilizzando la pipeline fusa a maggiore throughput.

### Budget GPU per modello

Ogni modello Jetson presenta inoltre un profilo hardware che limita la quantità di risorse che l’elaborazione del pool condiviso può richiedere e scala le dimensioni dei batch:

| Modello | Limite massimo del budget GPU | Moltiplicatore delle dimensioni del batch | Riservato al sistema/display |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

La RAM rilevata regola il profilo: se un dispositivo Jetson segnala **16 GB o più**, il moltiplicatore del batch viene aumentato di ×1,2. La dimensione di base del batch prima dell’applicazione dei moltiplicatori è di 8 immagini.

Per il riferimento completo sull’adattamento computazionale, consultare [Adattamento computazionale dinamico](../processing-architecture/dynamic-compute-adaptation.md).

***

## Limite di frequenza della GPU per Texture Aware su Nano e Orin Nano

Il debayer Texture Aware esegue l’inferenza della rete neurale sulla GPU, il che può attivare **avvisi di sovracorrente**sui modelli Jetson a basso consumo (classe 10-15 W) quando la GPU funziona alla massima velocità di clock. Quando si passa `--debayer texture-aware` su un**Jetson Nano o Orin Nano**, la CLI
verifica la frequenza massima della GPU e la limita a **510 MHz** (510000000) se è attualmente superiore:

* Se la CLI
è in grado di scrivere nel nodo sysfs della frequenza della GPU, il limite viene **applicato automaticamente** e viene visualizzato un messaggio di conferma.
* In caso contrario (è necessario il privilegio di root), la CLI
visualizza il comando esatto `sudo` per applicare manualmente il limite, attende un attimo per consentirne la lettura, quindi prosegue — l’elaborazione continua ma potrebbero comparire avvisi di sovracorrente.

Per applicare il limite manualmente prima dell’elaborazione:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

I modelli a maggiore potenza (Orin NX 25W, AGX Orin 60W) funzionano alla massima velocità della GPU; non viene applicato alcun limite. Il debayer Standard non attiva mai il limite su nessun modello.

{% hint style="info" %}
**Il percorso &quot;Texture Aware&quot; su Jetson elabora sempre un’immagine alla volta.** Ogni worker avrebbe bisogno di un proprio contesto CUDA (~1 GB) oltre a una propria copia del modello di denoising, cosa che la memoria unificata non può sostenere; pertanto, su Jetson il percorso Texture Aware è associato a un singolo worker con accesso serializzato alla GPU. È prevedibile che Texture Aware risulti notevolmente più lento rispetto a Standard su qualsiasi Jetson. Texture Aware si applica alle immagini di Survey3
: le acquisizioni LATTICE utilizzano sempre il debayer Standard. In assenza di una GPU CUDA utilizzabile, l’esecuzione utilizza la modalità Standard e il backend registra una riga `[DEBAYER] … needs a CUDA GPU …` che ne spiega il motivo (mostrata con `chloros-cli --verbose`).
{% endhint %}

***

## Gestione termica

I dispositivi Jetson hanno un margine termico limitato, specialmente in configurazioni chiuse o aeree. Chloros
monitora la temperatura del SoC e limita automaticamente le dimensioni dei batch:

| Temperatura         | Azione                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Funzionamento normale — velocità di elaborazione massima          |
| **70 °C** (Avviso)  | La dimensione dei batch viene ridotta progressivamente (dal 100% al 50% tra 70 °C e 80 °C) |
| **80 °C** (Critico) | Riduzione aggressiva della velocità (dal 50% allo 0% tra 80 °C e 90 °C) |
| **90 °C** (Spegnimento) | Arresto completo dell’elaborazione della GPU — necessario raffreddamento |

{% hint style="warning" %}
**Garantire un’adeguata ventilazione e dissipazione del calore** per un’elaborazione prolungata, specialmente in involucri da campo chiusi o sistemi aerei. Il throttling termico ridurrà la velocità di elaborazione per proteggere l’hardware.
{% endhint %}

***

## Gestione della memoria

I dispositivi Jetson utilizzano la **memoria unificata**: la GPU e la CPU condividono la stessa RAM fisica. La VRAM indicata (ad es. ~15,3 GB su un Orin NX da 16 GB) non è memoria dedicata alla GPU; si tratta della stessa RAM utilizzata dal sistema operativo e da ogni altro processo.

### Avviso e raccomandazioni sullo swap

Prima dell’elaborazione su Jetson, la CLI
conta le immagini RAW presenti nella cartella di input (`.tif`, `.tiff`, `.raw`, `.dng` — le anteprime JPG non vengono conteggiate), stima il picco di memoria necessario per l’esecuzione e **avverte prima dell’avvio** se la RAM + lo swap rischiano di essere insufficienti. L’avviso ha come oggetto `LOW MEMORY WARNING - Jetson Detected`, riporta il numero di immagini, la RAM, lo swap attuale e il picco stimato, quindi fornisce i comandi esatti `fallocate` / `chmod` / `mkswap` / `swapon` dimensionati per progetto (mai inferiori a 8 GB). Si mette in pausa per alcuni secondi in modo che il messaggio non vada perso nello scorrimento dello schermo, quindi l’elaborazione prosegue.**Stime di memoria utilizzate dall’avviso:**

| Modalità Debayer | Base | Per immagine |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Texture Aware | ~2,5 GB (modello + runtime Python) | ~15 MB |

L’avviso viene generato quando il picco stimato supera la RAM + lo swap meno un margine di sicurezza di 1 GB, e tiene conto solo dello swap **basato su file** — una configurazione basata esclusivamente su zram verrà comunque segnalata.

Per aggiungere manualmente lo swap (esempio: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### Gestione OOM (Out of Memory)

Durante l’elaborazione, Chloros
monitora la memoria della GPU e riduce gradualmente le prestazioni invece di andare in crash:

1. Quando l’utilizzo della memoria della GPU supera l’**85%**, le dimensioni dei batch vengono ridotte in modo preventivo
2. Se si verifica comunque un evento di esaurimento della memoria, la dimensione del batch viene **dimezzata** e dimezzata nuovamente a ogni successivo evento OOM; ogni batch successivo completato con successo riduce tale penalità di un livello
3. In caso di pressione prolungata, la pipeline passa dal percorso `fused_gpu` a quello `tiled_gpu`, più efficiente in termini di memoria, e, come ultima risorsa, all’elaborazione tramite CPU

Poiché la GPU condivide la RAM del Jetson, la RAM più lo swap continuano a limitare l’esecuzione: l’avviso relativo allo swap riportato sopra è il controllo a cui prestare attenzione.

***

## Implementazione sul campo

### Considerazioni sull’alimentazione

| Modello Jetson     | Assorbimento tipico | Note                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15 W              | Jack cilindrico CC          |
| Jetson Orin NX   | 10-25 W             | Jack cilindrico CC          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD o jack cilindrico |

Pianificate il vostro budget energetico per un&#x27;elaborazione prolungata: il consumo energetico di picco si verifica durante il Thread 3 (Elaborazione), che richiede un uso intensivo della GPU.

### Raccomandazioni relative allo storage

* **SSD NVMe** fortemente raccomandato per le implementazioni arm64
* Le schede SD sono troppo lente per l&#x27;elaborazione: utilizzatele solo come supporto di avvio
* Prevedere uno spazio pari a 2-3 volte la dimensione dei dati grezzi delle immagini per l’output elaborato

### Funzionamento senza monitor tramite SSH
La Chloros di

Chloros
è ideale per le implementazioni Jetson senza monitor:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Backend sempre attivo per la sincronizzazione temporale di LATTICE / DAQ-E

Se il vostro Jetson controlla telecamere LATTICE o sensori di luce DAQ-E in modalità headless, abilitate il servizio systemd del backend **come utente** in modo che il grandmaster PTP funzioni in modo continuo e gli script dell’SDK
trovino un backend in esecuzione (l’unità è installata ma non abilitata di default):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

Il backend verrà quindi eseguito come utente: le acquisizioni che scrive sono le tue e utilizza il tuo `chloros-cli login`. Vedi [Installazione su Linux
](linux-installation.md#always-on-ptp-for-headless-hosts) per i dettagli, compreso il file `chloros-backend.service` di root e come il pacchetto renda le porte PTP 319/320 associabili senza privilegi di root.

### Elaborazione automatizzata con systemd

Crea un servizio systemd per l’elaborazione automatizzata:

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

`chloros-cli process` termina con un codice di uscita diverso da zero quando un’esecuzione che ha richiesto dei prodotti non scrive alcuna immagine, quindi lo stato di errore di systemd è significativo ai fini del monitoraggio.

Abbinarlo a un timer di systemd per l’elaborazione pianificata:

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

### Elaborazione di base su Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI
```

### SDK
Python
su Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Sistemi Jetson consigliati per l’uso sul campo

Per le implementazioni sul campo e a bordo di velivoli, prendete in considerazione queste opzioni di schede carrier Jetson Orin NX da 16 GB:

* **A bordo di velivoli/droni**: sistemi con resistenza alle vibrazioni (MIL-STD), leggeri (meno di 300 g), raffreddamento passivo
* **Ambienti difficili**: involucri impermeabili IP67/IP69K con connettività per telecamere GigE PoE
* **Configurazione minimale/economica**: kit di sviluppo con involucri aggiuntivi

Contatta l’[Assistenza MAPIR
](https://www.mapir.camera/community/contact) per consigli specifici sull’hardware in base al tuo scenario di implementazione.

***

## Passi successivi

* [Installazione di Linux
](linux-installation.md) — Dettagli generali sull’installazione di Linux

* [Adattamento dinamico della potenza di calcolo](../processing-architecture/dynamic-compute-adaptation.md) — Riferimento completo sulle strategie di calcolo
* [Pipeline di elaborazione](../processing-architecture/processing-pipeline.md) — Approfondimento sulla pipeline a 4 thread
* [CLI
: riga di comando](../CLI.md) — Guida alla CLI

* [API
: Python
SDK
](../api-python-sdk.md) — Guida all’SDK

* [Riferimento CLI
](../reference/cli-reference.md) e [Riferimento SDK
](../reference/sdk-reference.md) — Elenchi completi dei comandi e delle API
per la versione 1.3.0
