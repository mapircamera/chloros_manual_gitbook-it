# Adattamento dinamico dell&#x27;elaborazione

Chloros 1.2.0 utilizza il rilevamento dell&#x27;hardware e la selezione automatica della strategia di elaborazione. Il motore di elaborazione si adatta al tuo hardware — da un Jetson Orin Nano a una workstation multi-GPU — senza alcuna configurazione manuale.

***

## Come funziona

All’avvio, Chloros esegue un’analisi del sistema:

1. **Rileva il sistema operativo** — Windows o Linux
2. **Identifica i core della CPU e la RAM totale**

3.**Rileva la presenza di GPU** — compatibilità NVIDIA CUDA, VRAM, modello
4. **Identifica il modello Jetson** (se applicabile) — tramite `/proc/device-tree/model`
5. **Verifica i sensori termici** (Jetson) — per un&#x27;elaborazione sensibile alla temperatura
6. **Seleziona la strategia di calcolo** — in base a tutto l’hardware rilevato
7. **Configura automaticamente il numero di worker, il tipo di pipeline e l’allocazione della memoria**

Il profilo rilevato viene memorizzato nella cache per la sessione, sia in memoria che su disco, in modo che le esecuzioni successive avvengano più rapidamente:

| Piattaforma | Profilo memorizzato nella cache |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (prende in considerazione `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Elimina quel file per forzare un nuovo rilevamento — utile dopo aver aggiunto una GPU o più RAM. Chloros esegue inoltre un nuovo rilevamento automatico quando la cache è stata scritta da una versione precedente incompatibile.

***

## Strategie di calcolo

Chloros seleziona una delle tre strategie di calcolo in base all’hardware in uso:

| Strategia | Selezionata quando | Worker | Esecutore | Pipeline |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU CUDA che segnala**12 GB+ di VRAM**(su memoria unificata Jetson, richiede anche almeno 12 GB+ di RAM condivisa totale) | `min(4, VRAM ÷ 4GB)`, minimo 2 —**limite massimo di 2 su Jetson** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| GPU CUDA con**2-12 GB di VRAM**| 3 (sovrapposizione I/O; accesso alla GPU serializzato da un semaforo).**1 (sequenziale) su Jetson con meno di 12 GB di RAM** | `ProcessPoolExecutor` (spawn); sequenziale in-process su Jetson con poca RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Nessuna GPU CUDA o con meno di 2 GB di VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Esempi pratici della formula per i worker `GPU_PARALLEL`: 12 GB di VRAM → 3 worker, 16 GB e oltre → 4 worker, qualsiasi Jetson → 2 worker.

Il parallelismo è implementato con lo standard `concurrent.futures` di Python: le strategie GPU utilizzano un `ProcessPoolExecutor` con il metodo **spawn** (ogni worker è un processo separato con il proprio contesto CUDA — `fork` copierebbe uno stato CUDA già inizializzato e danneggerebbe i processi figli), mentre la strategia CPU utilizza un `ThreadPoolExecutor`. Chloros non utilizza alcun framework distribuito di terze parti (come Ray).

### Tipi di pipeline

* **`fused_gpu`** — Percorso di elaborazione interamente su GPU. Le operazioni di debayering, correzione e indicizzazione vengono eseguite sulla GPU in un unico passaggio fuso. Offre il throughput più elevato, ma richiede la maggior quantità di VRAM.
* **`tiled_gpu`** — Percorso GPU efficiente in termini di memoria. Elabora le immagini in riquadri per adattarle alla memoria limitata della GPU. Throughput inferiore, ma funziona su dispositivi con memoria limitata.
* **`cpu_fallback`** — Elaborazione esclusivamente su CPU tramite parallelismo multithread. Utilizzato quando non è disponibile una GPU NVIDIA e come soluzione di ripiego di ultima istanza quando entrambi i percorsi GPU falliscono.

La catena di fallback in fase di esecuzione è sempre `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Sovrascrittura manuale della strategia

Impostare la variabile d&#x27;ambiente `CHLOROS_STRATEGY` per forzare una strategia specifica — una via d&#x27;uscita per esperti da utilizzare quando il rilevamento automatico sceglie un&#x27;opzione non adatta alla propria situazione (ad esempio, per mantenere la GPU libera per altre attività):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

La variabile viene riconosciuta senza distinzione tra maiuscole e minuscole; qualsiasi valore che non corrisponda a uno dei tre nomi viene ignorato e il rilevamento automatico procede normalmente. Anche in caso di sovrascrittura, Chloros continua a selezionare automaticamente il numero di worker:

| Sovrascrittura | Numero di worker utilizzato |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

È preferibile impostarlo per singolo comando piuttosto che in modo permanente, in modo che le esecuzioni normali continuino ad adattarsi automaticamente.

***

## Comportamento specifico della piattaforma

| Piattaforma | Strategia | Worker | Pipeline | Note |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sequenziale) | Modalità a basso consumo di memoria, un’immagine alla volta |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sequenziale) | La RAM condivisa inferiore a 12 GB impone l&#x27;elaborazione sequenziale |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (concorrente) | Dispositivo edge consigliato — Limite massimo di 2 worker su Jetson |
| **Jetson AGX Orin da 32-64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (concorrente) | Massime prestazioni edge (anche in questo caso con limite Jetson a 2 worker) |
| **Desktop con GPU da 8 GB** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 worker si sovrappongono nell’I/O mentre un semaforo serializza l’accesso alla GPU |
| **Desktop con GPU da 12 GB+** | `GPU_PARALLEL` | 3-4 | `fused_gpu` (concorrente) | Prestazioni ottimali del desktop: 12 GB → 3 worker, 16 GB+ → 4 |
| **Sistema solo CPU** | `CPU_PARALLEL` | core fisici − 1 (min. 2) | `cpu_fallback` | Nessuna GPU richiesta, utilizza un pool di thread |

{% hint style="info" %}
**Memoria unificata Jetson**: i dispositivi Jetson condividono la memoria della GPU e della CPU. Un Jetson Orin NX da 16 GB riporta circa 15,3 GB di VRAM, ma si tratta della stessa RAM fisica utilizzata dal sistema operativo e dai processi della CPU. Ecco perché i dispositivi Jetson da 16 GB e oltre sono idonei per `GPU_PARALLEL` come una GPU desktop da 12 GB e oltre, ma sono limitati a 2 worker: la GPU, i processi worker e i relativi contesti CUDA per ciascun worker attingono tutti dallo stesso pool condiviso.
{% endhint %}

### Budget GPU in base alla VRAM (GPU discrete)

Su host x86_64 dotati di una GPU NVIDIA discreta, la VRAM rilevata determina anche la quantità di risorse di elaborazione che la scheda può richiedere e la dimensione massima dei batch:

| VRAM rilevata | Limite massimo del budget GPU | Moltiplicatore della dimensione del batch |
| --- | --- | --- |
| **8 GB+** | 90% | ×2,0 |
| **6-8 GB** | 85% | ×1,75 |
| **3,5-6 GB** | 80% | ×1,5 |
| **2-3,5 GB** | 75% | ×1,25 |
| **Meno di 2 GB** | 70% | ×1,0 |

Le GPU discrete riservano solo 0,5 GB al sistema, poiché non condividono la RAM di sistema. I profili Jetson riservano una quantità molto maggiore e hanno un limite inferiore — consultare la [Guida NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Allocazione dinamica della memoria GPU

Chloros utilizza una [pipeline di elaborazione a 4 thread](processing-pipeline.md):

* **Thread 1** (Rilevamento) — Caricamento dell’immagine, analisi dei dati EXIF, rilevamento del bersaglio
* **Thread 2** (Calibrazione) — Calcolo della calibrazione della riflettanza
* **Thread 3** (Elaborazione) — Debayer GPU, correzione della vignettatura, calcolo dell’indice
* **Thread 4** (Esportazione) — Scrittura del file, incorporamento dei metadati

I thread 1, 2 e 4 consumano poca GPU; il thread 3 è quello che ne consuma di più. Man mano che i thread precedenti della pipeline terminano, la loro quota di GPU viene **ridistribuita ai thread attivi rimanenti**, quindi il thread 3 ottiene progressivamente più memoria man mano che l’esecuzione procede.

### Fasi di allocazione

| Fase | Thread attivi | Distribuzione della memoria GPU |
| --- | --- | --- |
| **Iniziale** | 1, 2, 3, 4 | Distribuita tra tutti i thread, per la maggior parte al Thread 3 |
| **Fase iniziale-intermedia** | 2, 3, 4 | La quota del thread 1 viene ridistribuita |
| **Fase intermedia-finale** | 3, 4 | Le quote dei thread 1 e 2 vanno ai thread 3 e 4 |
| **Tarda** | 3 o 4 | L’ultimo thread attivo riceve la sua allocazione massima |

Due regole determinano questi valori:

* A un thread che è l’**unico** attivo viene concessa l’allocazione massima prevista dal suo profilo.
* Quando sono attivi più task *pesanti* della GPU, l’allocazione di base di ciascun task pesante viene suddivisa tra di essi (senza mai scendere al di sotto del minimo configurato).

Il valore effettivamente utilizzato in fase di esecuzione è il **minore** tra l’allocazione del profilo della piattaforma e la raccomandazione in tempo reale fornita dal monitor della memoria della GPU, quindi una scheda molto occupata ha sempre la precedenza su un profilo ottimistico.***

## Elaborazione sensibile alle texture

Il debayer Texture Aware (**solo Chloros+** — `--debayer texture-aware`) esegue un modello di denoising basato su AI/ML che richiede circa 1,75 GB di VRAM in FP16 per copia, quindi utilizza molta più memoria della GPU rispetto al metodo standard:

* I sistemi con **meno di 7 GB di VRAM**elaborano Texture Aware in un**ciclo sincrono, un’immagine alla volta** — non è possibile gestire più copie del modello contemporaneamente e un pool di worker non farebbe altro che aumentare la contesa
* I sistemi con **7 GB o più di VRAM** possono elaborare Texture Aware in modo concorrente, sebbene con un numero ridotto di worker rispetto al metodo Standard
* Su **Jetson**, Texture Aware è sempre assegnato a un singolo worker e, sui modelli a basso consumo (Nano, Orin Nano), applica automaticamente anche un limite alla frequenza della GPU — consultare la [Guida NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Gestione termica (Jetson)

I dispositivi Jetson presentano vincoli termici, specialmente in configurazioni chiuse o montate su velivoli. Chloros monitora i sensori di temperatura integrati nel Jetson e ridimensiona automaticamente le dimensioni dei batch:

| Temperatura | Risposta |
| --- | --- |
| **&lt; 70 °C** | Funzionamento normale — velocità massima |
| **70 °C** (Avviso) | La dimensione dei batch viene ridotta progressivamente (dal 100% al 50% tra 70 °C e 80 °C) |
| **80 °C** (Critico) | Throttling aggressivo (dal 50% allo 0% tra 80 °C e 90 °C) |
| **90 °C** (Spegnimento) | Arresto completo dell’elaborazione della GPU |

Sui sistemi desktop dotati di un raffreddamento adeguato, il throttling termico viene attivato raramente.

***

## Gestione della pressione sulla memoria

Chloros monitora continuamente la memoria della GPU durante l’elaborazione e reagisce a tre livelli.

**Dimensione dei batch.** Un batch inizia con 8 immagini moltiplicate per il moltiplicatore della piattaforma indicato nelle tabelle sopra riportate. Chloros verifica quindi la VRAM libera, ne riserva il 20% per l’overhead proprio di PyTorch e ipotizza circa 100 MB di memoria GPU per ogni immagine da 12 MP: il batch corrisponde al valore minore tra il limite derivato dalla memoria e il valore di base della piattaforma. Non scende mai al di sotto di 1.**Riduzione preventiva.**Al di sopra dell’**85% di utilizzo della VRAM**, le dimensioni dei batch vengono ridotte prima che si verifichi qualsiasi errore.**Riduzione dell’allocazione per thread.** Man mano che l’utilizzo in tempo reale aumenta, il budget GPU di ciascun thread viene ridimensionato: ×0,75 al di sopra dell’80% di utilizzo, ×0,5 al di sopra del 90%. Le soglie di monitoraggio sono il 70% (conservativo), l’85% (limite operativo normale) e il 95% (rischio di OOM).**Backoff e ripristino in caso di OOM.** Se si verifica comunque un evento di esaurimento della memoria (OOM):

* la dimensione del batch viene **dimezzata** e dimezzata nuovamente a ogni evento OOM consecutivo — ogni batch successivo completato con successo riduce tale penalità di un livello
* le allocazioni dei thread attivi vengono ridotte al 70% del loro valore attuale e l’allocatore passa alla strategia conservativa, allentandola nuovamente dopo una serie di allocazioni riuscite
* in condizioni di forte pressione, la pipeline passa da `fused_gpu` a `tiled_gpu` e, come ultima risorsa, a `cpu_fallback`

**RAM dell’host (Jetson).** Prima dell’elaborazione, CLI stima il picco di memoria dell’host in base al numero di immagini e alla modalità di debayering e avvisa se la RAM più lo swap supportato da file rischia di essere insufficiente, stampando i comandi esatti per aggiungere lo swap — consultare la [Guida NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

## Monitoraggio dell’adattamento computazionale

### Diagnostica di sistema

`chloros-cli selftest` è il modo più rapido per verificare ciò che rileva il livello computazionale:

```bash
chloros-cli selftest
```

I suoi 7 controlli riguardano la versione, la disponibilità delle porte, l’avvio del backend, `/api/test`, le informazioni di sistema, la presenza del modello denoiser e la prontezza di CUDA e del denoiser. Il controllo 5 stampa direttamente la riga relativa all’hardware:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Il controllo 7 stampa `CUDA: <bool>, Denoiser: <bool>` — entrambi devono essere veri affinché Texture Aware sia utilizzabile.

### Log del backend

La strategia e il numero di worker vengono scelti all’interno del backend all’inizio di ogni esecuzione — non esiste un banner CLI che li annunci. Quando qualcosa si comporta in modo inaspettato (un percorso GPU che ricade su un’alternativa, un OOM, un denoiser che non si carica), il log del backend relativo a quella sessione ne riporta la traccia:

| Piattaforma | Posizione del log |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (un file per ogni avvio) |
| **Linux, CLI-backend avviato** | anche `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Stato di avanzamento in tempo reale

Durante un&#x27;esecuzione, CLI mostra lo stato di avanzamento in tempo reale per ogni thread (Rilevamento, Analisi, Elaborazione, Esportazione) trasmesso tramite Server-Sent Events: un indicatore pratico per capire se il thread 3 rappresenta il collo di bottiglia. Vedi [Pipeline di elaborazione](processing-pipeline.md).

***

## Passi successivi

* [Pipeline di elaborazione](processing-pipeline.md) — Comprensione dell’architettura della pipeline a 4 thread
* [Guida NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Implementazione e ottimizzazione specifiche per Jetson
* [CLI : Riga di comando](../CLI.md) — La guida CLI
* [Riferimento CLI](../reference/cli-reference.md) — Elenco completo dei comandi per la versione 1.2.0
