# Guida NVIDIA Jetson

Chloros

su NVIDIA Jetson consente l’elaborazione di immagini multispettrali in edge computing — sul campo, su UAV e in installazioni remote.Chloros

1.2.0 rileva il modello Jetson all’avvio e ottimizza la strategia di elaborazione in base all’hardware rilevato. **Non è necessaria alcuna regolazione manuale.**

***

## Modelli Jetson supportati

| Modello                | RAM            | Strategia di elaborazione                                     | Uso consigliato                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB condivisi | `GPU_PARALLEL` (2 worker)                              | Massime prestazioni, set di dati di grandi dimensioni                      |
| **Jetson Orin NX**   | 8-16 GB condivisa  | `GPU_PARALLEL` (2 worker, 16 GB) / `GPU_SINGLE` (8 GB)   | Raccomandazione principale per implementazioni aeree e sul campo |
| **Jetson Orin Nano** | 8 GB condivisi     | `GPU_SINGLE` (1 worker, sequenziale)                     | Elaborazione edge entry-level                                 |

{% hint style="info" %}
Il pacchetto arm64 diLinux

richiede **JetPack 6**, disponibile sulla famiglia Jetson Orin. I modelli precedenti (Nano, TX2, Xavier NX) non possono eseguire JetPack 6 e non sono supportati dal pacchetto attuale.
{% endhint %}

***

## Requisiti

* **JetPack 6.x** (si consiglia l’ultima versione)
* **NVIDIA CUDA** (inclusa in JetPack)
* **Piano a pagamentoChloros

+** — livello Copper o superiore (richiesto per l’accesso a tutti i serviziCLI

/SDK

; applicato a livello di server)

## Installazione

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Per informazioni generali sull’installazione diLinux

, sui percorsi dei file e sulla risoluzione dei problemi, consultare [Installazione diLinux

](linux-installation.md).

{% hint style="info" %}
**Posizionare la directory di estrazione su un supporto di archiviazione veloce.** I file binari compilati si decomprimono automaticamente in una directory temporanea ad ogni avvio — operazione estremamente lenta se eseguita da una scheda SD.Chloros

utilizza automaticamente `/mnt/ssd/tmp` se presente; in caso contrario, impostare `TMPDIR` su un percorso sul proprio NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Adattamento dinamico delle risorse di calcolo su Jetson

### Come funziona

All’avvio,Chloros

esegue la profilatura del sistema:

1. **Rileva il modello di Jetson** tramite `/proc/device-tree/model`
2. **Legge la memoria GPU/CPU condivisa disponibile** (Jetson utilizza la memoria unificata)
3. **Seleziona una strategia di elaborazione** (`GPU_PARALLEL`, `GPU_SINGLE` o `CPU_PARALLEL`)
4. **Imposta automaticamente il numero di worker, il tipo di pipeline e l’allocazione di memoria**La scelta dipende dalla**RAM condivisa totale**, non dal nome del modello:

* **Con meno di 12 GB di RAM totale**(tutti i Jetson da 8 GB): `GPU_SINGLE` con**1 worker — elaborazione sequenziale intenzionale**. La memoria è insufficiente per i worker concorrenti, quindi le immagini vengono elaborate una alla volta. Sui Jetson con**8 GB o meno**, Thread 3 ignora completamente il pool di worker ed esegue il proprio lavoro per ogni singola immagine all’interno del processo.
* **12 GB o più**(Orin NX da 16 GB, AGX Orin): la memoria unificata soddisfa i requisiti per `GPU_PARALLEL`, ma il numero di worker è**limitato a 2 su Jetson**: la GPU, la RAM dei processi worker e i contesti CUDA per ciascun worker attingono tutti dallo stesso pool condiviso, quindi un numero maggiore di worker comporta il rischio di errori di memoria insufficiente.

È possibile sovrascrivere la scelta automatica con la variabile d’ambiente `CHLOROS_STRATEGY` — vedere [Adattamento dinamico del calcolo](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Comportamento per modello

| Modello Jetson                | Strategia       | Processi di lavoro | Esecuzione                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Ciclo sequenziale all’interno del processo (`tiled_gpu` in condizioni di pressione di memoria) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Ciclo sequenziale all’interno del processo                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Processi di lavoro concorrenti, percorso `fused_gpu`  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2       | Processi di lavoro concorrenti, percorso `fused_gpu`  |

La differenza fondamentale tra le piattaforme è la **memoria**. Un Jetson da 8 GB deve elaborare le immagini una alla volta utilizzando un approccio a tasselli efficiente in termini di memoria quando la carico è elevato, mentre un Orin da 16 GB o più può elaborare 2 immagini contemporaneamente tramite la GPU utilizzando la pipeline fusa a maggiore throughput.

### Budget GPU per modello

Ogni modello Jetson presenta inoltre un profilo hardware che limita la quantità di risorse che l’elaborazione del pool condiviso può richiedere e scala le dimensioni dei batch:

| Modello | Limite massimo del budget GPU | Moltiplicatore delle dimensioni del batch | Riservato al sistema/display |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

La RAM rilevata regola il profilo: un Jetson che segnala **16 GB o più** vede il proprio moltiplicatore di batch aumentato di ×1,2. La dimensione di base del batch prima dei moltiplicatori è di 8 immagini.

Per il riferimento completo sull’adattamento computazionale, consultare [Adattamento computazionale dinamico](../processing-architecture/dynamic-compute-adaptation.md).

***

## Limite di frequenza della GPU per Texture Aware su Nano e Orin Nano

Il debayer Texture Aware esegue l’inferenza della rete neurale sulla GPU, il che può attivare **avvisi di sovracorrente**sui modelli Jetson a basso consumo (classe 10-15 W) a piena velocità di clock della GPU. Prima dell’elaborazione con Texture Aware su un**Jetson Nano o Orin Nano**,Chloros

verifica la frequenza massima della GPU e la limita a **510 MHz** (510000000) qualora fosse attualmente superiore:

* Se l’CLI

e è in grado di scrivere nel nodo sysfs della frequenza della GPU, il limite viene **applicato automaticamente** e viene visualizzata una conferma.
* In caso contrario (è necessario il root), l’CLI

e visualizza il comando esatto `sudo` per applicare manualmente il limite, attende un attimo per consentirti di leggerlo, quindi prosegue — l’elaborazione continua ma potrebbero comparire avvisi di sovracorrente.

Per applicare il limite manualmente prima dell’elaborazione:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

I modelli a maggiore potenza (Orin NX 25W, AGX Orin 60W) funzionano alla massima velocità della GPU; non viene applicato alcun limite. Il debayer Standard non attiva mai il limite su nessun modello.

{% hint style="info" %}
**Il percorso &quot;Texture Aware&quot; su Jetson elabora sempre un&#x27;immagine alla volta.** Ogni worker avrebbe bisogno di un proprio contesto CUDA (~1 GB) oltre a una propria copia del modello di denoising, cosa che la memoria unificata non può sostenere; pertanto, su Jetson il percorso Texture Aware è fissato a un singolo worker con accesso serializzato alla GPU. È prevedibile che Texture Aware risulti notevolmente più lento rispetto allo Standard su qualsiasi dispositivo Jetson.
{% endhint %}

***

## Gestione termica

I dispositivi Jetson hanno un margine termico limitato, specialmente in configurazioni chiuse o aeree.Chloros

monitora la temperatura del SoC e limita automaticamente le dimensioni dei batch:

| Temperatura         | Azione                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Funzionamento normale — velocità di elaborazione massima          |
| **70 °C** (Avviso)  | La dimensione dei batch si riduce progressivamente (dal 100% al 50% tra 70 °C e 80 °C) |
| **80 °C** (Critico) | Limitazione aggressiva (dal 50% allo 0% tra 80 °C e 90 °C) |
| **90 °C** (Spegnimento) | Arresto completo dell’elaborazione della GPU — è necessario il raffreddamento |

{% hint style="warning" %}
**Garantire un&#x27;adeguata ventilazione e dissipazione del calore** per un&#x27;elaborazione prolungata, specialmente in involucri da campo chiusi o sistemi aerei. Il throttling termico ridurrà la velocità di elaborazione per proteggere l&#x27;hardware.
{% endhint %}

***

## Gestione della memoria

I dispositivi Jetson utilizzano la **memoria unificata**: la GPU e la CPU condividono la stessa RAM fisica. La VRAM indicata (ad es. ~15,3 GB su un Orin NX da 16 GB) non è memoria dedicata alla GPU, ma è la stessa RAM utilizzata dal sistema operativo e da ogni altro processo.

### Avviso relativo allo swap e raccomandazioni

Prima dell’elaborazione su Jetson, l’CLI

a il conteggio delle immagini RAW presenti nella cartella di input (`.tif`, `.tiff`, `.raw`, `.dng` — le anteprime JPG non vengono conteggiate), stima il picco di memoria richiesto dall’esecuzione e **avvisa prima dell’avvio** se la RAM + lo swap rischiano di essere insufficienti. L’avviso è intitolato `LOW MEMORY WARNING - Jetson Detected`, riporta il numero di immagini, la RAM, lo spazio di swap attuale e il picco stimato, quindi fornisce i comandi esatti `fallocate` / `chmod` / `mkswap` / `swapon`, dimensionati in base al progetto (mai inferiori a 8 GB). Il programma si mette in pausa per alcuni secondi in modo che il messaggio non vada perso nello scorrimento della schermata, quindi l’elaborazione prosegue.**Stime di memoria utilizzate dall’avviso:**

| Modalità Debayer | Base | Per immagine |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Texture Aware | ~2,5 GB (modello + runtime diPython

) | ~15 MB |

L’avviso viene generato quando il picco stimato supera la RAM + lo swap meno un margine di sicurezza di 1 GB, e tiene conto solo dello swap **basato su file**: una configurazione basata esclusivamente su zram verrà comunque segnalata.

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

### Gestione dell’OOM (Out of Memory)

Durante l’elaborazione,Chloros

monitora la memoria della GPU e riduce gradualmente le prestazioni invece di andare in crash:

1. Quando l’utilizzo della memoria della GPU supera l’**85%**, le dimensioni dei batch vengono ridotte in modo preventivo
2. Se si verifica comunque un evento di esaurimento della memoria, la dimensione del batch viene **dimezzata** e nuovamente dimezzata a ogni successivo evento OOM; ogni batch successivo completato con successo riduce tale penalità di un livello
3. In caso di pressione prolungata, la pipeline passa dal percorso `fused_gpu` a quello `tiled_gpu`, più efficiente in termini di memoria, e, come ultima risorsa, all’elaborazione tramite CPU

***

## Implementazione sul campo

### Considerazioni sul consumo energetico

| Modello Jetson     | Consumo energetico tipico | Note                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15 W              | Connettore cilindrico CC          |
| Jetson Orin NX   | 10-25 W             | Connettore cilindrico CC          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD o connettore cilindrico |

Pianificate il vostro budget energetico per un&#x27;elaborazione prolungata: il consumo energetico di picco si verifica durante il Thread 3 (Elaborazione), che richiede un uso intensivo della GPU.

### Raccomandazioni relative allo storage

* **SSD NVMe** fortemente raccomandato per le implementazioni arm64
* Le schede SD sono troppo lente per l’elaborazione — utilizzarle solo come supporto di avvio
* Prevedere uno spazio pari a 2-3 volte la dimensione dei dati grezzi delle immagini per l’output elaborato

### Funzionamento headless tramiteSSH



Chloros

CLI

è ideale per implementazioni Jetson headless:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Backend sempre attivo per la sincronizzazione temporale di LATTICE / DAQ-E

Se il vostro Jetson controlla telecamere LATTICE o sensori di luce DAQ-E in modalità headless, abilitate il servizio systemd di backend in modo che il grandmaster PTP funzioni in modo continuo (l’unità è installata ma non abilitata per impostazione predefinita):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Consulta [Installazione diLinux

](linux-installation.md#always-on-ptp-for-headless-hosts) per i dettagli, compreso il modo in cui il pacchetto rende le porte PTP 319/320 associabili senza privilegi di root.

### Elaborazione automatizzata con systemd

Creare un servizio systemd per l’elaborazione automatizzata:

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

`chloros-cli process` restituisce un valore diverso da zero quando un’esecuzione che ha richiesto dei prodotti non scrive alcuna immagine, quindi lo stato di errore di systemd è significativo ai fini del monitoraggio.

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
    --indices NDVI NDRE GNDVI
```

###Python

SDK

su Jetson

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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Sistemi Jetson consigliati per l’uso sul campo

Per le implementazioni sul campo e aeree, prendete in considerazione queste opzioni di schede carrier Jetson Orin NX da 16 GB:

* **Aeree/droni**: Sistemi con resistenza alle vibrazioni (MIL-STD), leggeri (meno di 300 g), raffreddamento passivo
* **Ambiente esterno con condizioni estreme**: Involucri impermeabili IP67/IP69K con connettività per telecamere GigE PoE
* **Configurazione minima/economica**: Kit di sviluppo con involucri aggiuntivi

Contatta l’[AssistenzaMAPIR

](https://www.mapir.camera/community/contact) per consigli specifici sull’hardware in base al tuo scenario di implementazione.

***

## Passi successivi

* [Installazione diLinux

](linux-installation.md) — Dettagli generali sull’installazione diLinux


* [Adattamento dinamico della potenza di calcolo](../processing-architecture/dynamic-compute-adaptation.md) — Riferimento completo sulle strategie di calcolo
* [Pipeline di elaborazione](../processing-architecture/processing-pipeline.md) — Informazioni sulla pipeline a 4 thread
* [CLI

: Riga di comando](../CLI.md) — Guida a “CLI

”
* [API

:Python

SDK

](../api-python-sdk.md) — Guida a “SDK

”
* [RiferimentoCLI

](../reference/cli-reference.md) e [RiferimentoSDK

](../reference/sdk-reference.md) — Elenchi esaustivi di comandi eAPI

per la versione 1.2.0
