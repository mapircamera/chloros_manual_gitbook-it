# Pipeline di elaborazione

Chloros1.2.0 utilizza una pipeline di elaborazione a 4 thread che funziona come una catena di montaggio a fasi. Ogni thread gestisce una fase distinta del flusso di lavoro, consentendo così che più immagini possano trovarsi contemporaneamente in elaborazione in fasi diverse.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Architettura della pipeline

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Ogni immagine passa in sequenza attraverso tutti e quattro i thread. Grazie all’elaborazione multithread di Chloros+, più immagini occupano contemporaneamente thread diversi: mentre il thread 3 elabora un’immagine, il thread 1 può occuparsi del rilevamento di quella successiva, il thread 2 della calibrazione di un’altra e il thread 4 della scrittura su disco di quella completata.

Lo stato di avanzamento viene riportato per ogni thread e trasmesso tramite Server-Sent Events (il backend li pubblica su `/api/events`). Nella visualizzazione in tempo reale dello stato di avanzamento dell’CLI, le quattro fasi sono etichettate come **Rilevamento, Analisi, Elaborazione, Esportazione**.***

## Dettagli dei thread

### Thread 1: Rilevamento

**Scopo**: Caricare le immagini e rilevare i bersagli di calibrazione.

* Legge i file immagine dal disco — coppie Survey3 `.raw`+`.jpg`, acquisizioni LATTICE `.tif`/`.tiff` e `.dng`
* Estrae i metadati EXIF (GPS, modello della fotocamera, timestamp, esposizione)
* Rileva i target di calibrazione: geometrie dei target contrassegnate con ArUco per le acquisizioni LATTICE e il classico pannello di rilevamento per le foto dei target di calibrazione Survey3
* Risultati: dati dell’immagine + metadati + risultati del rilevamento dei target

Thread principalmente legato all’I/O e alla CPU.

### Thread 2: Calibrazione

**Scopo**: Calcolare i parametri di calibrazione a partire dai bersagli rilevati.

* Calcola i coefficienti di calibrazione della riflettanza dalle immagini dei target
* Calcola i parametri di correzione della vignettatura
* Determina le curve di calibrazione per ciascuna banda
* Output: parametri di calibrazione per ogni immagine

Un thread di calcolo vincolato alla CPU. Il thread 3 rimane in attesa di questo thread quando è abilitata la calibrazione della riflettanza, in modo che i suoi coefficienti siano pronti prima che qualsiasi immagine venga elaborata.

### Thread 3: Elaborazione (GPU)

**Scopo**: applicare le correzioni e calcolare gli indici di vegetazione.**Questo è il thread che richiede la maggiore potenza di calcolo.*** **Debayering**: converte i dati RAW Bayer in immagini multicanale
  * Standard (veloce, qualità media) — impostazione predefinita, `--debayer standard`
  * Texture Aware (lento, massima qualità) — solo Chloros+, `--debayer texture-aware`, utilizza un modello di denoising basato su AI/ML
  * Le acquisizioni LATTICE mono (M3M) sono a banda singola: per queste vengono saltate le fasi di demosaico e bilanciamento del bianco (con un messaggio di log di una riga), mentre le immagini M3C/Bayer nella stessa sessione vengono comunque elaborate
* **Correzione della vignettatura**: applica la correzione della vignettatura dell’obiettivo su tutta l’immagine
* **Calibrazione della riflettanza**: applica i coefficienti di calibrazione per convertire i valori in valori di riflettanza
* **Calcolo degli indici**: calcola gli indici di vegetazione (NDVI, NDRE, GNDVI, …)
* Risultati: dati immagine elaborati pronti per l’esportazione

Questo thread trae il massimo vantaggio dall’accelerazione GPU ed è il thread ottimizzato dalla [Dynamic Compute Adaptation](dynamic-compute-adaptation.md).

### Thread 4: Esportazione

**Scopo**: scrivere le immagini elaborate su disco.

* Scrive i file di output nel formato selezionato — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Incorpora i metadati nei file di output (GPS, timestamp, parametri di elaborazione)
* Organizza i file di output nella cartella del progetto come `<camera>/<format>/<Product>_Images/` — ad esempio `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **I file esportati mantengono il nome del file sorgente; la cartella identifica il prodotto.**
* Per le acquisizioni LATTICE, un fotogramma sorgente può generare diversi prodotti (Debayered, Preview, Radiance, Reflectance, Index), ciascuno nella propria cartella di prodotto
* Risultati: file finali su disco

Si tratta principalmente di un thread vincolato dall’I/O — l’utilizzo di un’unità SSD ne migliora notevolmente le prestazioni.

***

## Dietro le quinte: gli esecutori

All’interno del Thread 3, il lavoro per singola immagine viene parallelizzato con lo standard `concurrent.futures` di Python:

* **Le strategie GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) utilizzano un `ProcessPoolExecutor` con il metodo**spawn** — ogni worker è un processo separato con il proprio contesto CUDA (`fork` erediterebbe lo stato CUDA inizializzato del genitore e corromperebbe i figli)
* **`CPU_PARALLEL`** utilizza un `ThreadPoolExecutor` — NumPy e OpenCV rilasciano il GIL, quindi i thread sono sufficienti
* I dispositivi Jetson con 8 GB o meno di RAM condivisa saltano completamente l’esecutore ed eseguono l’elaborazione in-process, in modo sequenziale
* Anche Texture Aware su una GPU con meno di 7 GB di VRAM viene eseguito in modo sequenziale — il modello di denoising non può essere adattato più di una volta

Chlorosnon utilizza alcun framework distribuito di terze parti (come Ray). Vedi [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) per capire come vengono scelti la strategia e il numero di worker.

***

## Elaborazione sequenziale vs. elaborazione in pipeline

### Modalità libera (sequenziale)

Nella versione gratuita di Chloros, le immagini vengono elaborate **una alla volta**, in modo sequenziale attraverso tutte e quattro le fasi:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

L’interfaccia grafica mostra una barra di avanzamento semplificata in modalità gratuita; le sue fasi seriali sono indicate come **Rilevamento del bersaglio**e poi**Elaborazione**.

### Modalità Chloros+ (in pipeline)

Con una licenza Chloros+, tutti e quattro i thread operano **in parallelo** su immagini diverse:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

La barra di avanzamento dell’interfaccia grafica mostra le quattro fasi; passandoci sopra con il mouse si può vedere lo stato di avanzamento per ogni thread. Nell’CLIe, le stesse quattro fasi vengono trasmesse in tempo reale come **Rilevamento, Analisi, Elaborazione, Esportazione**.

{% hint style="info" %}
**Un’etichetta, due nomi.** L’CLI chiama la fase 3 _Elaborazione_. Il feed di avanzamento in modalità premium del backend — quello visualizzato dalla barra di avanzamento dell’interfaccia grafica — etichetta la stessa fase come _Calibrazione_. Si tratta dello stesso thread che esegue lo stesso lavoro (Thread 3: debayer, correzioni, indici).
{% endhint %}

{% hint style="success" %}
**L&#x27;elaborazione in pipeline con Chloros+** può essere da 3 a 5 volte più veloce dell’elaborazione sequenziale, a seconda dell’hardware e delle dimensioni del set di dati. L’aumento di velocità è maggiore su sistemi dotati di GPU veloci e SSD.
{% endhint %}

***

## Stato di avanzamento del thread 4 (esportazione)

Il thread di esportazione dispone di un proprio sistema di monitoraggio dello stato di avanzamento, che è possibile interrogare separatamente:**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

L’elaborazione è completata quando il thread 4 raggiunge il 100%.

{% hint style="info" %}
**Un&#x27;esecuzione che non scrive alcuna immagine è considerata un errore.**In caso di esito positivo, `chloros-cli process` riporta il numero di prodotti immagine scritti (`Image products written: N`). Se sono stati richiesti prodotti e**nessuno**è stato scritto — solo `project.json` e `calibration_data.json` — l&#x27;CLIe stampa `Processing finished but wrote no image products.` e**termina con un valore diverso da zero**, indicando il nome della cartella del progetto e le cause più comuni (la cartella di input non è stata riconosciuta come acquisizione — controllare il layout e `--input-level` — oppure tutti i prodotti richiesti non erano applicabili a quelle telecamere). Gli script possono fare affidamento sul codice di uscita.
{% endhint %}

***

## Relazione con l’adattamento dinamico del calcolo

[L’adattamento dinamico del calcolo](dynamic-compute-adaptation.md) influisce principalmente sul **Thread 3 (Elaborazione)**:

* **`GPU_PARALLEL`**: il Thread 3 elabora più immagini contemporaneamente tramite la GPU utilizzando la pipeline `fused_gpu`
* **`GPU_SINGLE`**: il thread 3 serializza l’accesso alla GPU con un semaforo mentre i processi di lavoro sovrappongono le operazioni di I/O, utilizzando la pipeline `fused_gpu` o quella `tiled_gpu`, più efficiente in termini di memoria
* **`CPU_PARALLEL`**: il thread 3 utilizza l’elaborazione basata su CPU con parallelismo multithread

L’allocazione di memoria GPU del thread 3 aumenta anche man mano che i thread 1 e 2 terminano — vedi [Allocazione dinamica della memoria GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Passi successivi

* [Adattamento dinamico del calcolo](dynamic-compute-adaptation.md) — Come Chloros seleziona la strategia ottimale per il tuo hardware
* [Guida a NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Comportamento della pipeline specifico per la piattaforma su Jetson
* [Monitoraggio dell’elaborazione](../processing-images-gui/monitoring-the-processing.md) — Monitoraggio dello stato di avanzamento tramite interfaccia grafica
* [Riferimento su CLI](../reference/cli-reference.md) — `process`, `export-status`, codici di uscita e layout dell’output
