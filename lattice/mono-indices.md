# Telecamere monocromatiche e indici di vegetazione

## Una telecamera = una banda

Una telecamera **M3M**è la versione monocromatica della**M3C** con sensore Bayer: un sensore monocromatico IMX265 dotato di un unico filtro di interferenza a banda stretta. La stringa del modello indica la banda — `M3M-<lens>-F<wavelength>`, ad esempio `M3M-L87-F685` (visualizzata in Chloros come `LATT-M3M-L87-F685`). Il sensore fornisce una **singola banda in scala di grigi** senza mosaico Bayer: non c’è nulla da demosaicare, nessun crosstalk tra i canali da separare e nessun bilanciamento del bianco da impostare.

Conseguenze da tenere presenti prima di progettare un sistema monocromatico:

* **La radianza e la riflettanza sono completamente definite per ciascuna banda.**Si tratta di mappe radiometriche per banda, quindi una fotocamera M3M produce radianza calibrata in float32 (W/m²/sr/nm) e riflettanza in uint16 (`32768` = ρ 1,0) esattamente come fa una banda M3C. I fotogrammi mono contengono una matrice di risposta del sensore**identica** — non è necessario né viene applicato alcun processo di separazione 3×3.
* **Una singola telecamera mono non può produrre un indice di vegetazione.** NDVI, NDRE e simili richiedono almeno due bande. Per calcolare gli indici da hardware mono è necessario combinare diverse telecamere M3M — vedi sotto.
* Le telecamere M3M trasmettono in streaming **Mono12** (12 bit: 2 byte/pixel in trasmissione, oppure 1,5 byte/pixel quando un array utilizza il [trasporto a 12 bit compresso](arrays.md#packed-12-bit-transport)), il che è rilevante per la [gestione della larghezza di banda dell’array](arrays.md#bandwidth-how-many-cameras-fit).

## Cosa salta Chloros per il mono — e come te lo comunica

Le fasi della pipeline del colore semplicemente non si applicano a un sensore a banda singola. Chloros **le salta con un messaggio di una riga** anziché generare un errore, e continua a eseguirle normalmente per qualsiasi fotocamera M3C (Bayer) nella stessa sessione:

| Fase | Comportamento mono (M3M) | Comportamento M3C |
| --- | --- | --- |
| Demosaic / debayer | Saltata — il livello di esportazione `debayered` è un’immagine in scala di grigi a 1 canale. | Demosaic a 3 canali. |
| Bilanciamento del bianco (`lattice white-balance`) | Saltato con un messaggio di una riga. | Funziona normalmente. |
| Profilo colore (`lattice color-profile`) | Saltato con un messaggio di una riga. | Funziona normalmente. |
| Saturazione/contrasto (`lattice color`) | Saltato con un messaggio di una riga. | Funziona normalmente. |
| Separazione del crosstalk spettrale | Identità (nessuna matrice 3×3). | Matrice 3×3 applicata per ciascuna telecamera. |
| Radianza / riflettanza | **Funziona** — per banda, completamente calibrata. | Funziona per banda. |

L’interfaccia grafica applica lo stesso gating: per una telecamera monocromatica, il pannello delle impostazioni per telecamera nasconde le righe relative solo all’RGB (bilanciamento del bianco, gamma, profilo colore, saturazione, contrasto, suddivisione dei canali) e l’istogramma in tempo reale è bloccato su una singola traccia **MONO**. Il discriminante in tutto lo stack è il token `M3M` nella stringa del modello, visualizzato nell’interfaccia grafica/SDK come `is_mono`.

## Gli indici richiedono ≥ 2 bande: allineamento → stack → indice

Il flusso di lavoro dell’indice mono prevede sempre le stesse tre fasi:

1. **Allineamento** — puntare diverse telecamere M3M a diverse lunghezze d’onda (ad es. una F650 “Red” e una F850 “NIR”), collegarle come [array multicamera](arrays.md) e lasciare che Chloros calcoli la trasformazione di co-registrazione tra le telecamere.
2. **Accatastamento** — i fotogrammi allineati diventano un’unica immagine multibanda (ogni telecamera contribuisce con una banda denominata).
3. **Indice** — si valuta una formula di indice sulle bande dello stack, con la possibilità di renderizzarla tramite una LUT.

Nell’interfaccia grafica (GUI) l’intera catena corrisponde alla modalità di visualizzazione **Telecamere combinate**: il composito in tempo reale è già allineato e il Calcolatore di indice dell’array (di seguito) definisce la formula da renderizzare. Le esportazioni acquisite possono essere deformate per ottenere lo stesso allineamento con l’opzione di acquisizione**Aligned**.

## Il Calcolatore dell’indice

Il Calcolatore dell’indice definisce l’espressione dell’indice utilizzata dalla visualizzazione in tempo reale e dalle esportazioni dell’indice per singola telecamera. Si tratta di un’unica interfaccia condivisa, accessibile da due punti nella barra laterale della scheda *Telecamere*:

* **Per singola telecamera**— Anteprima in tempo reale → icona a forma di ingranaggio**Indice** (solo telecamere Bayer RGN/OCN/NGB; una singola telecamera mono non dispone di controllo dell’indice poiché una sola banda non può generare un indice).
* **Per array**— Impostazioni array → Anteprima in tempo reale → icona a forma di ingranaggio**Indice**. Questo è il percorso per le telecamere mono: l’elenco delle bande comprende**tutte le telecamere dell’array**, quindi una coppia mono contribuisce qui con le sue due bande.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

I suoi controlli, dall’alto verso il basso:

* **Chip delle bande** (“Bande — clicca per aggiungere all’espressione”) — un pulsante per ogni banda disponibile, etichettato con il nome del colore + lunghezza d’onda in nm (i nomi dei colori duplicati vengono disambiguati, ad esempio come “Colore 850”). Cliccando si inserisce il token della banda in corrispondenza del cursore. Le bande provenienti da fotocamere che non sono in grado di produrre radianza per singola banda (RGB/FRGB) vengono filtrate.
* **Pulsanti operatore e funzione** — `+ - * / ( ) ^ ,` più `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Area di testo dell’espressione** — formula digitabile liberamente; il segnaposto mostra la forma classica dell’NDVI `(NIR - Red) / (NIR + Red)`. Un’anteprima tokenizzata di sola lettura sopra di essa visualizza i chip di banda, i numeri e i flag come token sconosciuti.
* **Banner di validità**— grigio «Vuoto — non verrà applicato alcun indice»; verde «Espressione valida»; rosso con l’errore di analisi specifico (banda sconosciuta, banda ambigua rilevata da più telecamere, parentesi mancante, …); oppure giallo quando l’espressione è valida ma**la costante** (ad es. `X/X`, oppure un denominatore NDVI digitato con `−` invece di `+`) — una costante mappa l’intero fotogramma su un unico colore.
* Appare un avviso separato di colore ambra se l’espressione applicata è corretta ma il **fotogramma in tempo reale è uniforme** (scena piatta o saturata) — il collasso dell’istogramma viene rilevato automaticamente.
* **Applica LUT**(attivato per impostazione predefinita; disattivato = espansione in scala di grigi),**Livello**2/3/5/7 stop (impostazione predefinita 7 stop) e gli input**Min / Max**che affiancano la barra del gradiente. Il valore predefinito di Min è**0,2**— ingrandisce la rampa di colori nell’intervallo rilevante per la vegetazione, mentre i valori inferiori vengono visualizzati in scala di grigi; impostare Min su −1 per l’intero intervallo dell’indice (il pulsante**Reset** ripristina il range da −1 a +1). Il valore predefinito di Max è 1.
* **Istogramma in tempo reale** della distribuzione dell’indice — barre con scala radice quadrata, linee dei percentili p2/p98 in ambra, una linea mediana bianca e indicazioni delle code fuori intervallo ("◀ N% < lo" / "hi < N% ▶") che diventano ambra al di sopra dell’1% per indicare la necessità di ampliare la finestra Min/Max.
* **Applica** applica l’espressione al flusso in tempo reale; le modifiche alla LUT vengono applicate in tempo reale senza premere Applica. Se è aperto un progetto, l’espressione e le impostazioni della LUT vengono salvate con il progetto e ripristinate alla sua riapertura; in assenza di progetto, durano solo per la sessione.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## Il percorso CLI



La stessa catena allineamento → stack → indice, programmabile end-to-end:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` utilizza un array collegato che contiene le proprie telecamere, oppure le apre autonomamente per l’esecuzione singola. `index --live` apre sempre le telecamere autonomamente (un [array transitorio](../reference/cli-reference.md#multi-camera-transient-sessions)) e non dispone di `--serials`: apre tutte le telecamere presenti sulla rete. Quindi, rilasciare tutte le telecamere che Chloros detiene prima di esso — la GUI, `cam-connect`, `array-connect` (`chloros-cli lattice array-disconnect --all`): l’apertura di una telecamera già occupata una seconda volta può interrompere lo streaming di chi la detiene o causare il crash del backend. Una telecamera che non invia alcun fotogramma fa fallire l’esecuzione e non viene scritto né lo stack né l’indice.

`--channel` mappa i simboli di un preset ai nomi delle bande dello stack. Due regole consentono di evitare un'esecuzione fallita:

* **I simboli distinguono tra maiuscole e minuscole** e devono corrispondere esattamente ai nomi dei canali del preset — i preset utilizzano lettere minuscole (quelli NDVI sono `red`,`nir`; controlla `--list-presets`). `--channel red=Red_660` funziona; `--channel RED=660` fallisce con un errore `channel_map missing entries`.
* Il parametro «band side» deve specificare il nome di una banda presente nello stack allineato (`lattice align-info --profile align.json` ne elenca l’elenco). La modalità offline accetta anche indici di banda a partire da 0, ad esempio `--channel red=0 --channel nir=1`.

`lattice index` funziona anche in modalità offline su un file TIFF multibanda allineato salvato. Ogni modulo viene eseguito sul backend, pertanto è necessario che questo sia in esecuzione e che si sia effettuato l’accesso a un piano a pagamento di Chloros+; un `--input` o `--output` relativo viene risolto in base alla cartella da cui si esegue il comando:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Preimpostazioni dell’indice

`lattice index --preset` (e la [sandbox Indice/LUT](../image-viewer-gui/index-lut-sandbox.md) nella scheda Immagine, che utilizza lo stesso motore) include queste **22 preimpostazioni**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Eseguire `chloros-cli lattice index --list-presets` per visualizzare la formula e i simboli dei canali di ciascun preset, e `--list-gradients` per i gradienti di colore disponibili. Le formule personalizzate utilizzano `--formula EXPR` con la stessa sintassi del Calcolatore di indici. Si noti che questo elenco di preimpostazioni è specifico per il motore di indici LATTICE: il menu a tendina "Elaborazione" nelle Impostazioni del progetto per le immagini importate presenta un elenco diverso (vedere [Formule degli indici multispettrali](../project-settings/multispectral-index-formulas.md)).

Il set completo di flag (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, le manopole di allineamento e deformazione per `--live` e altro) è documentato nella [Guida di riferimento CLI § Indici / Calcoli sulla vegetazione](../reference/cli-reference.md#index--vegetation-maths); gli equivalenti SDK si trovano nel [Riferimento SDK](../reference/sdk-reference.md).

## Acquisizione di prodotti di indice da un array mono

Con un array collegato e un’espressione di indice applicata, `array-capture` (o l’opzione della GUI **Acquisisci tutto**) salva i livelli di esportazione per ciascuna telecamera *e* il rendering dell’indice — `--index`/`--no-index` lo attiva/disattiva sulla CLI e, per impostazione predefinita, acquisisce tutti i livelli applicabili. Il contributo di una telecamera mono a ciascun gruppo di acquisizione è costituito dalla sua singola banda ai livelli raw/debayered (scala di grigi)/radianza/riflettanza, oltre al composito a indice combinato condiviso quando l’array funziona in modalità combinata. Vedi [Array multicamera § Acquisizione](arrays.md#capturing-monitoring-vs-analysis).
