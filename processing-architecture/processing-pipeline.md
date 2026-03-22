# Pipeline di elaborazione

Chloros 1.1.0 utilizza una pipeline di elaborazione a 4 thread che funziona come una catena di montaggio a fasi. Ogni thread gestisce una fase distinta del flusso di lavoro di elaborazione, consentendo l&#x27;elaborazione simultanea di più immagini in fasi diverse.

***

## Architettura della pipeline

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Ogni immagine passa attraverso tutti e quattro i thread in ordine. Con l&#x27;elaborazione multithread di Chloros+, più immagini possono trovarsi contemporaneamente in thread diversi: mentre il thread 3 elabora un&#x27;immagine, il thread 1 può rilevarne la successiva, il thread 2 può calibrarne un&#x27;altra e il thread 4 può scrivere su disco un&#x27;immagine elaborata in precedenza.

***

## Dettagli sui thread

### Thread 1: Rilevamento

**Scopo**: Caricare immagini e rilevare i target di calibrazione.

* Legge i file immagine dal disco (RAW, JPG)
* Estrae i metadati EXIF (GPS, modello della fotocamera, timestamp, esposizione)
* Rileva i target di calibrazione ArUco nelle immagini dei target contrassegnati
* Output: dati immagine + metadati + risultati del rilevamento dei target

Si tratta principalmente di un thread legato all&#x27;I/O e alla CPU.

### Thread 2: Calibrazione

**Scopo**: Calcolare i parametri di calibrazione dai target rilevati.

* Calcola i coefficienti di calibrazione della riflettanza dalle immagini dei target
* Calcola i parametri di correzione della vignettatura
* Determina le curve di calibrazione per banda
* Output: parametri di calibrazione per ciascuna immagine

Questo è un thread di calcolo legato alla CPU.

### Thread 3: Elaborazione (GPU)

**Scopo**: Applicare le correzioni e calcolare gli indici di vegetazione.**Questo è il thread che richiede la maggiore potenza di calcolo.*** **Debayering**: Converte i dati RAW con pattern Bayer in immagini multicanale
  * Standard (Veloce, Qualità media) — impostazione predefinita
  * Texture Aware (Lento, Massima qualità) — solo Chloros+, utilizza la denoising AI/ML
* **Correzione della vignettatura**: Applica la correzione della vignettatura dell&#x27;obiettivo all&#x27;intera immagine
* **Calibrazione della riflettanza**: applica coefficienti di calibrazione per convertire i valori in valori di riflettanza
* **Calcolo degli indici**: calcola gli indici di vegetazione (NDVI, NDRE, GNDVI, ecc.)
* Output: dati dell&#x27;immagine elaborati pronti per l&#x27;esportazione

Questo thread trae il massimo vantaggio dall&#x27;accelerazione GPU. Il sistema [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) ottimizza principalmente il comportamento di questo thread.

### Thread 4: Esportazione

**Scopo**: Scrivere le immagini elaborate su disco.

* Scrive i file di output nel formato selezionato (TIFF 16-bit, TIFF 32-bit %, PNG, JPG)
* Incorpora i metadati EXIF nei file di output (GPS, timestamp, parametri di elaborazione)
* Organizza l&#x27;output in sottocartelle relative al modello di fotocamera
* Output: file finali su disco

Si tratta principalmente di un thread vincolato dall&#x27;I/O. L&#x27;utilizzo di un&#x27;unità SSD migliora significativamente le prestazioni del Thread 4.

***

## Elaborazione sequenziale vs. elaborazione in pipeline

### Modalità gratuita (sequenziale)

Nella versione gratuita di Chloros, le immagini vengono elaborate **una alla volta**, in modo sequenziale attraverso tutte e quattro le fasi:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

La barra di avanzamento della GUI mostra 2 fasi: Rilevamento del bersaglio ed Elaborazione.

### Modalità Chloros+ (in pipeline)

Con una licenza Chloros+, tutti e quattro i thread operano **in modo concorrente** su immagini diverse:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

La barra di avanzamento dell&#x27;interfaccia grafica mostra 4 fasi: Rilevamento, Analisi, Calibrazione, Esportazione. Passa il mouse sulla barra di avanzamento per visualizzare lo stato di avanzamento per ogni thread.

{% hint style="success" %}
**L&#x27;elaborazione in pipeline con Chloros+** può essere da 3 a 5 volte più veloce dell&#x27;elaborazione sequenziale, a seconda dell&#x27;hardware e delle dimensioni del set di dati. L&#x27;aumento di velocità è maggiore su sistemi con GPU veloci e SSD.
{% endhint %}

***

## Stato di avanzamento dell&#x27;esportazione del thread 4

In Chloros 1.1.0, il thread di esportazione (Thread 4) dispone di un proprio monitoraggio dedicato dello stato di avanzamento. È possibile monitorare separatamente lo stato di avanzamento dell&#x27;esportazione:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

L&#x27;elaborazione è completata quando il Thread 4 raggiunge il 100%.

***

## Relazione con l&#x27;adattamento dinamico del calcolo

Il sistema [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) influisce principalmente sul **Thread 3 (Elaborazione)**:

* Strategia **`GPU_PARALLEL`**: il Thread 3 elabora più immagini contemporaneamente tramite la GPU utilizzando la pipeline `fused_gpu`
* Strategia **`GPU_SINGLE`**: il Thread 3 elabora un&#x27;immagine alla volta utilizzando la pipeline `tiled_gpu`, efficiente in termini di memoria
* Strategia **`CPU_PARALLEL`**: il Thread 3 utilizza l&#x27;elaborazione basata su CPU con parallelismo multithread

L&#x27;allocazione della memoria GPU del thread 3 cambia anche dinamicamente man mano che i thread 1 e 2 completano l&#x27;elaborazione — vedi [Allocazione dinamica della memoria GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Passi successivi

* [Adattamento dinamico dell&#x27;elaborazione](dynamic-compute-adaptation.md) — Come Chloros seleziona la strategia ottimale per il tuo hardware
* [Guida NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Comportamento della pipeline specifico per la piattaforma su Jetson
* [Monitoraggio dell&#x27;elaborazione](../processing-images-gui/monitoring-the-processing.md) — Monitoraggio dell&#x27;avanzamento tramite GUI
