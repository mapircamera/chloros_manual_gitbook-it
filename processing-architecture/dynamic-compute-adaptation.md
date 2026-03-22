# Adattamento dinamico dell&#x27;elaborazione

Chloros 1.1.0 introduce il rilevamento intelligente dell&#x27;hardware e la selezione automatica della strategia di elaborazione. Il motore di elaborazione si adatta al tuo hardware — da un Jetson Nano a una workstation multi-GPU — senza alcuna configurazione manuale.

***

## Come funziona

All&#x27;avvio, Chloros esegue automaticamente il profiling del sistema:

1. **Rileva il sistema operativo** — Windows o Linux
2. **Identifica i core della CPU e la RAM totale**

3.**Rileva la presenza della GPU** — funzionalità NVIDIA CUDA, VRAM, modello
4. **Identifica il modello Jetson** (se applicabile) — tramite `/proc/device-tree/model`
5. **Controlla i sensori termici** (Jetson) — per un&#x27;elaborazione sensibile alla temperatura
6. **Seleziona la strategia di calcolo ottimale** — in base a tutto l&#x27;hardware rilevato
7. **Configura automaticamente il numero di worker, il tipo di pipeline e l&#x27;allocazione della memoria**Il risultato viene memorizzato nella cache in modo che le esecuzioni successive inizino più velocemente. Se l&#x27;hardware cambia (ad esempio, viene aggiunta una GPU), Chloros esegue una nuova profilatura al successivo avvio.***

## Strategie di calcolo

Chloros seleziona una delle tre strategie di calcolo in base al vostro hardware:

| Strategia | GPU richiesta | Worker | Pipeline | Ideale per |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Sì (12 GB+ di VRAM o 16 GB+ condivisa) | 3-4 | `fused_gpu` | GPU desktop con 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Sì (&lt; 12 GB di VRAM) | 1-3 | `tiled_gpu` | GPU entry-level, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | No | core - 1 | `cpu_fallback` | Sistemi senza GPU NVIDIA |

### Tipi di pipeline

* **`fused_gpu`** — Percorso di elaborazione GPU completo. Tutte le operazioni di debayering, correzione e indicizzazione vengono eseguite sulla GPU in un unico passaggio fuso. Offre il throughput più elevato, ma richiede più VRAM.
* **`tiled_gpu`** — Percorso GPU efficiente in termini di memoria. Elabora le immagini in tessere per adattarle alla memoria GPU limitata. Offre un throughput inferiore, ma funziona su dispositivi con memoria limitata.
* **`cpu_fallback`** — Elaborazione solo su CPU utilizzando il parallelismo multithread. Utilizzato quando non è disponibile una GPU NVIDIA.***

## Comportamento specifico della piattaforma

| Piattaforma | Strategia | Worker | Pipeline | Note |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (serializzato) | Modalità efficiente in termini di memoria, elabora un&#x27;immagine alla volta |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (concorrente) | Dispositivo edge consigliato — elaborazione GPU realmente parallela |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (concorrente) | Massime prestazioni edge |
| **Desktop con GPU da 8 GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Buone prestazioni desktop con tile efficienti in termini di memoria |
| **Desktop con GPU da 12 GB+** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Prestazioni desktop ottimali |
| **Sistema solo CPU** | `CPU_PARALLEL` | core - 1 | `cpu_fallback` | Nessuna GPU richiesta, utilizza ThreadPool |

{% hint style="info" %}
**Memoria unificata Jetson**: i dispositivi Jetson condividono la memoria della GPU e della CPU. Un Jetson Orin NX da 16 GB riporta circa 15,3 GB di VRAM, ma si tratta della stessa RAM fisica utilizzata dal sistema operativo e dai processi della CPU. Chloros tiene conto di questo aspetto quando imposta le soglie di allocazione della memoria.
{% endhint %}

***

## Allocazione dinamica della memoria GPU

Chloros utilizza una [pipeline di elaborazione a 4 thread](processing-pipeline.md):

* **Thread 1** (Rilevamento) — Caricamento dell&#x27;immagine, analisi EXIF, rilevamento del target
* **Thread 2** (Calibrazione) — Calcolo della calibrazione della riflettanza
* **Thread 3** (Elaborazione) — Debayer GPU, correzione della vignettatura, calcolo dell&#x27;indice
* **Thread 4** (Esportazione) — Scrittura dei file, incorporamento dei metadati

Man mano che i thread precedenti della pipeline completano il loro lavoro (ad esempio, tutte le immagini sono state rilevate), la loro allocazione di memoria GPU viene rilasciata e **ridistribuita ai thread attivi rimanenti**. Ciò significa che il Thread 3 (la fase ad alta intensità di GPU) ottiene progressivamente più memoria man mano che la pipeline avanza, migliorando la produttività per il lavoro più intensivo in termini di calcolo.

### Fasi di allocazione

| Fase | Thread attivi | Distribuzione della memoria GPU |
| --- | --- | --- |
| **Iniziale** | 1, 2, 3, 4 | Distribuita tra tutti i thread |
| **Fase intermedia-iniziale** | 2, 3, 4 | Memoria del thread 1 ridistribuita |
| **Fase intermedia-finale** | 3, 4 | La memoria dei thread 1+2 va ai thread 3+4 |
| **Fase finale** | 3 o 4 | Memoria massima per il thread rimanente |***

## Elaborazione Texture Aware

Il metodo di debayering Texture Aware (solo Chloros+) utilizza una quantità di memoria GPU significativamente maggiore rispetto al metodo Standard a causa del modello di denoising AI/ML:

* I sistemi con **&lt; 7 GB di VRAM** sono costretti a un ciclo di elaborazione sincrono per la modalità Texture Aware (un&#x27;immagine alla volta)
* I sistemi con **7 GB+ di VRAM** possono elaborare Texture Aware in modo concorrente, sebbene con un numero di worker ridotto rispetto alla modalità Standard***

## Gestione termica (Jetson)

I dispositivi Jetson presentano limitazioni termiche, specialmente in implementazioni chiuse o aeree. Chloros monitora le temperature della GPU e della CPU e regola automaticamente l&#x27;elaborazione:

| Temperatura | Risposta |
| --- | --- |
| **&lt; 70 °C** | Funzionamento normale — piena velocità |
| **70 °C** (Avviso) | Riduzione della dimensione del batch |
| **80 °C** (Critico) | Throttling aggressivo — riduzione della concorrenza e del numero di worker |
| **90 °C** (Spegnimento) | Arresto completo dell&#x27;elaborazione della GPU |

Il monitoraggio della temperatura utilizza `tegrastats` sulle piattaforme Jetson. Sui sistemi desktop con raffreddamento adeguato, il throttling termico viene attivato raramente.

***

## Gestione della pressione di memoria

Chloros monitora la pressione della memoria di sistema durante l&#x27;elaborazione:

* **Soglia di memoria**: un utilizzo dell&#x27;85% attiva un comportamento conservativo
* **Riduzione OOM**: se si verifica un evento di esaurimento della memoria, l&#x27;allocazione viene ridotta del 25% (moltiplicatore 0,75x)
* **Fallback della pipeline**: In caso di forte pressione sulla memoria, la pipeline passa automaticamente da `fused_gpu` a `tiled_gpu`
* **Raccomandazioni sullo swap**: su Jetson, Chloros avvisa se lo spazio di swap è insufficiente per le dimensioni del set di dati***

## Monitoraggio dell&#x27;adattamento di calcolo

### Output dello stato di CLI

All&#x27;avvio dell&#x27;elaborazione, CLI visualizza il profilo hardware rilevato:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Diagnostica di sistema

Eseguire `chloros-cli selftest` per visualizzare un profilo hardware completo e verificare le capacità di calcolo:

```bash
chloros-cli selftest
```

Questo controllo verifica la disponibilità di CUDA, la memoria della GPU, i modelli di denoising e la connettività del backend.

***

## Passaggi successivi

* [Pipeline di elaborazione](processing-pipeline.md) — Comprensione dell&#x27;architettura della pipeline a 4 thread
* [Guida NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Distribuzione e ottimizzazione specifiche per Jetson
* [CLI : Riga di comando](../CLI.md) — Riferimento completo CLI
