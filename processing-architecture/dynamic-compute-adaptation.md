# Adattamento dinamico delle risorse di calcolo

Chloros 1.3.0 utilizza il rilevamento dell’hardware e la selezione automatica della strategia di elaborazione. Il motore di elaborazione si adatta al tuo hardware — da un Jetson Orin Nano a una workstation multi-GPU — senza alcuna configurazione manuale.

***

## Come funziona

All’avvio, Chloros esegue un’analisi del sistema:

1. **Rileva il sistema operativo** — Windows o Linux
2. **Identifica i core della CPU e la RAM totale**

3.**Rileva la presenza della GPU** — compatibilità con NVIDIA CUDA, VRAM, modello
4. **Identifica il modello Jetson** (se applicabile) — tramite `/proc/device-tree/model`
5. **Controlla i sensori termici** (Jetson) — per un’elaborazione sensibile alla temperatura
6. **Seleziona la strategia di calcolo** — in base a tutto l’hardware rilevato
7. **Configura automaticamente il numero di worker, il tipo di pipeline e l’allocazione di memoria**

Il profilo rilevato viene memorizzato nella cache per la sessione sia in memoria che su disco, in modo che le esecuzioni successive avviino più rapidamente:

| Piattaforma | Profilo memorizzato nella cache |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (prende in considerazione `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Eliminare quel file per forzare un nuovo rilevamento — utile dopo aver aggiunto una GPU o più RAM. Chloros esegue inoltre automaticamente un nuovo rilevamento quando la cache è stata scritta da una versione precedente incompatibile.

***

## Strategie di calcolo

Chloros seleziona una delle tre strategie di calcolo in base all’hardware in uso:

| Strategia | Selezionata quando | Worker | Esecutore | Pipeline |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU CUDA con**12 GB+ di VRAM**(su memoria unificata Jetson, richiede anche almeno 12 GB di RAM condivisa totale) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, minimo 2 —**limite massimo di 2 su Jetson** | Processi worker (contesti CUDA separati) | `fused_gpu` |
| **`GPU_SINGLE`**| GPU CUDA con**2-12 GB di VRAM**| 3 (sovrapposizione I/O; accesso alla GPU serializzato tramite un semaforo).**1 (sequenziale) su Jetson con meno di 12 GB di RAM** | Processi di lavoro (contesti CUDA separati); sequenziale all’interno dello stesso processo su Jetson con poca RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Nessuna GPU CUDA o meno di 2 GB di VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Esempi pratici della formula per i worker `GPU_PARALLEL`: 12 GB di VRAM → 2 worker, 16 GB → 3 worker, 20 GB+ → 4 worker. A ciascun worker vengono assegnati 3,5 GB, il fabbisogno stimato per un’attività Texture Aware, entro il 90% della VRAM rimanente dopo una riserva di 0,5 GB.

{% hint style="info" %}
**Le strategie GPU richiedono una versione di PyTorch in grado di utilizzare CUDA.** Una macchina il cui PyTorch non dispone di un dispositivo CUDA utilizzabile riceve `CPU_PARALLEL`, indipendentemente dalla GPU in uso. Il pacchetto Jetson 1.3.0 utilizza la build CUDA di PyTorch di JetPack 6, quindi un Jetson è idoneo una volta installate le librerie CUDA e cuDNN di JetPack. Un Jetson con meno di 12 GB di RAM elabora sempre un’immagine alla volta, in-process (registrato come `GPU_SINGLE`).
{% endhint %}

Le strategie GPU eseguono un pool di processi di lavoro, ciascuno dei quali è un processo separato del sistema operativo con il proprio contesto CUDA; Chloros avvia questo pool rilanciando il proprio programma backend installato (non tramite `multiprocessing` di Python, poiché l’avvio dei processi di lavoro non funziona all’interno dell’applicazione inclusa nel pacchetto). La strategia CPU utilizza un pool di thread (`ThreadPoolExecutor` di Python). Chloros non utilizza alcun framework distribuito di terze parti (come Ray).

### Tipi di pipeline

* **`fused_gpu`** — Percorso di elaborazione interamente su GPU. Le operazioni di debayering, correzione e indicizzazione vengono eseguite sulla GPU in un unico passaggio fuso. Offre il throughput più elevato, ma richiede la maggior quantità di VRAM.
* **`tiled_gpu`** — Percorso GPU efficiente in termini di memoria. Elabora le immagini in tessere per adattarle alla memoria limitata della GPU. Throughput inferiore, ma funziona su dispositivi con memoria limitata.
* **`cpu_fallback`** — Elaborazione esclusivamente su CPU tramite parallelismo multithread. Utilizzato quando non è disponibile una GPU NVIDIA e come soluzione di ripiego di ultima istanza qualora entrambi i percorsi GPU falliscano.

La catena di fallback in fase di esecuzione è sempre `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Sovrascrittura manuale della strategia

Impostare la variabile d’ambiente `CHLOROS_STRATEGY` per forzare una strategia specifica — una via d’uscita per esperti da utilizzare quando il rilevamento automatico sceglie un’opzione non adatta alla propria situazione (ad esempio, per mantenere la GPU libera per altre attività):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

La variabile viene confrontata senza distinzione tra maiuscole e minuscole; qualsiasi valore che non corrisponda a uno dei tre nomi viene ignorato e il rilevamento automatico procede normalmente. Anche in caso di sovrascrittura, Chloros continua a scegliere il numero di worker per voi:

| Sovrascrittura | Numero di worker utilizzato |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Una sovrascrittura di tipo `GPU_SINGLE` o `GPU_PARALLEL` su una macchina priva di dispositivi CUDA utilizzabili viene rifiutata con una riga `[STRATEGY]` nel log, e l’esecuzione utilizza `CPU_PARALLEL`.

È preferibile impostarlo per singolo comando anziché in modo permanente, in modo che le esecuzioni normali continuino ad adattarsi automaticamente.

***

## Comportamento specifico della piattaforma

| Piattaforma | Strategia | Worker | Pipeline | Note |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sequenziale) | Modalità efficiente in termini di memoria, un'immagine alla volta |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sequenziale) | La RAM condivisa inferiore a 12 GB impone l’elaborazione sequenziale |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (concorrente) | Dispositivo edge consigliato — Limite Jetson a 2 worker |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (concorrente) | Massime prestazioni edge (anche in questo caso con limite di 2 worker per Jetson) |
| **Desktop con GPU da 8 GB** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 worker sovrappongono le operazioni di I/O mentre un semaforo serializza l’accesso alla GPU |
| **Desktop con GPU da 12 GB+** | `GPU_PARALLEL` | 2-4 | `fused_gpu` (concorrenti) | Prestazioni ottimali del desktop: 12 GB → 2 worker, 16 GB → 3, 20 GB+ → 4 |
| **Sistema solo CPU** | `CPU_PARALLEL` | core fisici − 1 (min. 2) | `cpu_fallback` | Nessuna GPU richiesta, utilizza un pool di thread |

{% hint style="info" %}
**Memoria unificata Jetson**: i dispositivi Jetson condividono la memoria della GPU e della CPU. Un Jetson Orin NX da 16 GB riporta circa 15,3 GB di VRAM, ma si tratta della stessa RAM fisica utilizzata dal sistema operativo e dai processi della CPU. Ecco perché i Jetson da 16 GB e oltre sono idonei per `GPU_PARALLEL` come una GPU desktop da 12 GB e oltre, ma sono limitati a 2 worker: la GPU, i processi worker e i relativi contesti CUDA per ciascun worker attingono tutti allo stesso pool condiviso.
{% endhint %}

### Budget della GPU in base alla VRAM (GPU discrete)

Su host x86_64 con una GPU NVIDIA discreta, la VRAM rilevata determina anche quanta parte della capacità di elaborazione della scheda può essere utilizzata e quanto possono crescere i batch:

| VRAM rilevata | Limite massimo del budget GPU | Moltiplicatore della dimensione del batch |
| --- | --- | --- |
| **8 GB+** | 90% | ×2,0 |
| **6-8 GB** | 85% | ×1,75 |
| **3,5-6 GB** | 80% | ×1,5 |
| **2-3,5 GB** | 75% | ×1,25 |
| **Meno di 2 GB** | 70% | ×1,0 |

Le GPU discrete riservano solo 0,5 GB al sistema, poiché non condividono la RAM di sistema. I profili Jetson riservano una quantità di memoria molto maggiore e hanno un limite inferiore — consultare la [Guida NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Allocazione dinamica della memoria GPU

Chloros utilizza una [pipeline di elaborazione a 4 thread](processing-pipeline.md):

* **Thread 1** (Rilevamento) — Caricamento dell’immagine, analisi dei dati EXIF, rilevamento del bersaglio
* **Thread 2** (Calibrazione) — Calcolo della calibrazione della riflettanza
* **Thread 3** (Elaborazione) — Debayer GPU, correzione della vignettatura, calcolo dell’indice
* **Thread 4** (Esportazione) — Scrittura dei file, incorporamento dei metadati

I thread 1, 2 e 4 consumano poca GPU; il thread 3 è quello che ne consuma di più. Man mano che i thread precedenti della pipeline terminano, la loro quota di GPU viene **ridistribuita ai thread attivi rimanenti**, quindi il thread 3 ottiene progressivamente più memoria man mano che l’esecuzione procede.

### Fasi di allocazione

| Fase | Thread attivi | Distribuzione della memoria GPU |
| --- | --- | --- |
| **Iniziale** | 1, 2, 3, 4 | Distribuita tra tutti i thread, la maggior parte al thread 3 |
| **Inizio-metà** | 2, 3, 4 | La quota del thread 1 viene ridistribuita |
| **Fase intermedia-finale** | 3, 4 | Le quote dei thread 1 e 2 vengono assegnate ai thread 3 e 4 |
| **Fase finale** | 3 o 4 | L’ultimo thread attivo riceve la sua allocazione massima |

Due regole regolano questi valori:

* A un thread che è l’**unico** attivo viene concessa l’allocazione massima prevista dal proprio profilo.
* Quando sono attivi più di un’attività *pesante* sulla GPU, l’allocazione di base di ciascuna attività pesante viene suddivisa tra di esse (senza mai scendere al di sotto del minimo configurato).

Il valore effettivamente utilizzato in fase di esecuzione è il **minore** tra l’allocazione del profilo della piattaforma e la raccomandazione in tempo reale fornita dal monitor della memoria della GPU, quindi una scheda molto sollecitata ha sempre la precedenza su un profilo ottimistico.

***

## Elaborazione Texture Aware

Il debayer Texture Aware (**solo Chloros+** — `--debayer texture-aware`) esegue un modello di denoising AI/ML che richiede circa 1,75 GB di VRAM in FP16 per copia, quindi utilizza molta più memoria GPU rispetto al metodo standard:

* I sistemi con **meno di 7 GB di VRAM**elaborano il Texture Aware in un**ciclo sincrono, un’immagine alla volta** — non è possibile gestire più copie del modello contemporaneamente e un pool di worker non farebbe altro che aumentare la contesa
* I sistemi con **7 GB o più di VRAM** possono elaborare Texture Aware in modo concorrente, sebbene con un numero ridotto di worker rispetto al metodo Standard
* Su **Jetson**, Texture Aware è sempre assegnato a un singolo worker
* In assenza di **una GPU CUDA utilizzabile**, l’esecuzione utilizza il debayer Standard e il backend registra una riga `[DEBAYER] … needs a CUDA GPU …` che ne spiega il motivo
* Sui Jetson a basso consumo (Nano, Orin Nano), la CLI applica un limite di frequenza della GPU quando si richiede Texture Aware — consultare la [Guida NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Gestione termica (Jetson)

I dispositivi Jetson presentano vincoli termici, specialmente in configurazioni chiuse o a bordo di velivoli. Chloros monitora i sensori di temperatura integrati nel Jetson e ridimensiona automaticamente le dimensioni dei batch:

| Temperatura | Risposta |
| --- | --- |
| **< 70 °C** | Funzionamento normale — velocità massima |
| **70 °C** (Avviso) | La dimensione del batch viene ridotta progressivamente (dal 100% al 50% tra 70 °C e 80 °C) |
| **80 °C** (Critico) | Limitazione aggressiva (dal 50% allo 0% tra 80 °C e 90 °C) |
| **90 °C** (Spegnimento) | Arresto completo dell’elaborazione della GPU |

Sui sistemi desktop dotati di raffreddamento adeguato, il throttling termico viene attivato raramente.

***

## Gestione della pressione sulla memoria

Chloros monitora continuamente la memoria della GPU durante l’elaborazione e reagisce a tre livelli.

**Dimensione dei batch.** Un batch parte da 8 immagini moltiplicate per il fattore di moltiplicazione della piattaforma indicato nelle tabelle sopra riportate. Chloros verifica quindi la VRAM libera, ne riserva il 20% per l’overhead di PyTorch e ipotizza circa 100 MB di memoria GPU per ogni immagine da 12 MP: il batch corrisponde al valore minore tra il limite derivato dalla memoria e il valore di base della piattaforma. Non scende mai al di sotto di 1.

**Riduzione preventiva.**Al di sopra dell’**85% di utilizzo della VRAM**, le dimensioni dei batch vengono ridotte prima che si verifichino errori.**Riduzione dell’allocazione per thread.** Man mano che l’utilizzo in tempo reale aumenta, il budget GPU di ciascun thread viene ridimensionato: ×0,75 al di sopra dell’80% di utilizzo, ×0,5 al di sopra del 90%. Le soglie di monitoraggio sono il 70% (conservativo), l’85% (limite operativo normale) e il 95% (rischio di esaurimento della memoria).**Backoff e recupero in caso di esaurimento della memoria (OOM).** Se si verifica comunque un evento di esaurimento della memoria:

* la dimensione del batch viene **dimezzata** e dimezzata nuovamente a ogni evento OOM consecutivo — ogni batch successivo completato con successo riduce tale penalità di un livello
* le allocazioni dei thread attivi vengono ridotte al 70% del loro valore attuale e l’allocatore passa alla strategia conservativa, allentandola nuovamente dopo una serie di allocazioni riuscite
* in caso di forte pressione, la pipeline passa da `fused_gpu` a `tiled_gpu` e, come ultima risorsa, a `cpu_fallback`

**RAM dell’host (Jetson).** Prima dell’elaborazione, la CLI stima la memoria massima dell’host in base al numero di immagini e alla modalità di debayering e avvisa se la RAM più lo swap supportato da file potrebbero essere insufficienti, visualizzando i comandi esatti per aggiungere lo swap — consultare la [Guida NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Monitoraggio dell’adattamento computazionale

### Diagnostica di sistema

`chloros-cli selftest` è il modo più rapido per verificare cosa rileva il livello computazionale:

```bash
chloros-cli selftest
```

I suoi 7 controlli riguardano la versione, la disponibilità delle porte, l’avvio del backend, `/api/test`, le informazioni di sistema, la presenza del modello denoiser e la prontezza di CUDA e del denoiser. Il controllo 5 visualizza direttamente la riga relativa all’hardware:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Il controllo 7 visualizza `CUDA: <bool>, Denoiser: <bool>` — entrambi devono essere veri affinché Texture Aware sia utilizzabile.

### Log del backend

La strategia e il numero di worker vengono scelti all’interno del backend all’inizio di ogni esecuzione — non esiste un banner della CLI che li annunci. Quando si verifica un comportamento inaspettato (un percorso GPU che ricade su un’alternativa, un OOM, un denoiser che non si carica), è nel log del backend di quella sessione che viene segnalato:

| Piattaforma | Posizione del log |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (un file per ogni avvio) |
| **Linux, backend avviato tramite CLI** | anche `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Avanzamento in tempo reale

Durante un'esecuzione, la CLI mostra in tempo reale lo stato di avanzamento per ogni thread (Rilevamento, Analisi, Elaborazione, Esportazione) trasmesso tramite Server-Sent Events: un indicatore pratico per capire se il thread 3 rappresenta il collo di bottiglia. Vedi [Pipeline di elaborazione](processing-pipeline.md).

***

## Passi successivi

* [Pipeline di elaborazione](processing-pipeline.md) — Comprendere l’architettura della pipeline a 4 thread
* [Guida NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Implementazione e ottimizzazione specifiche per Jetson
* [CLI: riga di comando](../CLI.md) — Guida alla CLI

* [Riferimento CLI](../reference/cli-reference.md) — Elenco completo dei comandi per la versione 1.3.0
