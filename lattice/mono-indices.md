# Telecamere monocromatiche e indici di vegetazione

## Una telecamera = una banda

Una telecamera **M3M**è la versione monocromatica della**M3C**con sensore Bayer: un sensore monocromatico IMX265 dotato di un unico filtro di interferenza a banda stretta. La stringa del modello indica la banda — `M3M-<lens>-F<wavelength>`, ad esempio `M3M-L87-F685` (visualizzata in Chloros come `LATT-M3M-L87-F685`). Il sensore fornisce una**singola banda in scala di grigi** senza mosaico di Bayer: non c’è nulla da demosaicare, nessun crosstalk tra i canali da separare e nessun bilanciamento del bianco da impostare.

Conseguenze da conoscere prima di progettare un sistema monocromatico:

* **La radianza e la riflettanza sono completamente definite per ciascuna banda.**Si tratta di mappe radiometriche per banda, quindi una telecamera M3M produce radianza calibrata in float32 (W/m²/sr/nm) e riflettanza in uint16 (`32768` = ρ 1,0) esattamente come fa una banda M3C. I fotogrammi mono contengono una matrice di risposta del sensore**identica** — non è necessario né viene applicato alcun unmix 3×3.
* **Una singola telecamera mono non può produrre un indice di vegetazione.** NDVI, NDRE e simili richiedono almeno due bande. Per calcolare gli indici da hardware mono è necessario combinare diverse telecamere M3M — vedi sotto.
* Le telecamere M3M trasmettono **Mono12** (12 bit, 2 byte/pixel sulla linea), il che è importante per la [gestione della larghezza di banda dell’array](arrays.md#bandwidth-the-rules-of-thumb).

## Cosa salta Chloros per il mono — e come te lo comunica

Le fasi della pipeline del colore semplicemente non si applicano a un sensore a banda singola. Chloros **li salta con un messaggio di una riga** invece di generare un errore, e li esegue comunque normalmente per qualsiasi fotocamera M3C (Bayer) nella stessa sessione:

| Fase | Comportamento Mono (M3M) | Comportamento M3C |
| --- | --- | --- |
| Demosaic / debayer | Saltato — il livello di esportazione `debayered` è un&#x27;immagine in scala di grigi a 1 canale. | Demosaic a 3 canali. |
| Bilanciamento del bianco (`lattice white-balance`) | Saltato con un messaggio di una riga. | Funziona normalmente. |
| Profilo colore (`lattice color-profile`) | Saltato con un messaggio di una riga. | Funziona normalmente. |
| Saturazione/contrasto (`lattice color`) | Saltato con un messaggio di una riga. | Funziona normalmente. |
| Separazione del crosstalk spettrale | Identità (nessuna matrice 3×3). | Matrice 3×3 applicata per ogni telecamera. |
| Radianza / riflettanza | **Funziona** — per banda, completamente calibrato. | Funziona per banda. |

L&#x27;interfaccia grafica applica lo stesso filtro: per una telecamera mono, il pannello delle impostazioni per telecamera nasconde le righe relative esclusivamente a RGB (bilanciamento del bianco, gamma, profilo colore, saturazione, Contrasto, suddivisioni dei canali), e l’istogramma in tempo reale è bloccato su una singola traccia **MONO**. Il discriminante in tutto lo stack è il token `M3M` nella stringa del modello, visualizzato nell’interfaccia grafica come SDK.

## Gli indici richiedono ≥ 2 bande: allineamento → stacking → indicizzazione

Il flusso di lavoro per l’indicizzazione mono prevede sempre le stesse tre fasi:

1. **Allineamento** — puntare diverse telecamere M3M a diverse lunghezze d’onda (ad es. una F650 &quot;Red&quot; e una F850 &quot;NIR&quot;), collegarle come [array multicamera](arrays.md) e lasciare che Chloros calcoli la trasformazione di co-registrazione tra le telecamere.
2. **Stack** — i fotogrammi allineati diventano un’unica immagine multibanda (ogni telecamera contribuisce con una banda denominata).
3. **Indice** — si valuta una formula di indice sulle bande dello stack, opzionalmente renderizzandola tramite una LUT.

Nell’interfaccia grafica (GUI) l’intera catena corrisponde alla modalità di visualizzazione a matrice **Telecamere combinate**: il composito in tempo reale è già allineato e il Calcolatore di indice della matrice (di seguito) definisce la formula da renderizzare. Le esportazioni acquisite possono essere deformate per ottenere lo stesso allineamento con l’opzione di acquisizione**Allineato**.

## Il Calcolatore di indice

Il Calcolatore di indice definisce l’espressione dell’indice utilizzata dalla visualizzazione in tempo reale e dalle esportazioni dell’indice per singola telecamera. Si tratta di un’unica superficie condivisa, accessibile da due punti nella barra laterale della scheda *Telecamere*:

* **Per singola telecamera**— Anteprima in tempo reale → icona a forma di ingranaggio**Indice** (solo telecamere Bayer RGN/OCN/NGB; una singola telecamera mono non dispone di controllo dell’indice poiché una sola banda non è sufficiente per generarlo).
* **Per array**— Impostazioni array → Anteprima in tempo reale → icona a forma di ingranaggio**Indice**. Questo è il percorso per le telecamere mono: l’elenco delle bande comprende**tutte le telecamere dell’array**, quindi una coppia mono contribuisce qui con le sue due bande.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

I relativi controlli, dall’alto verso il basso:

* **Chip delle bande** (“Bande — clicca per aggiungere all’espressione”) — un pulsante per ogni banda disponibile, etichettato con il nome del colore + lunghezza d’onda in nm (i nomi dei colori duplicati vengono disambiguati, ad esempio come “Colore 850”). Cliccando si inserisce il token della banda in corrispondenza del cursore. Le bande provenienti da telecamere che non sono in grado di produrre radianza per banda (RGB/FRGB) vengono filtrate.
* **Pulsanti degli operatori e delle funzioni** — `+ - * / ( ) ^ ,` e `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Area di testo dell’espressione** — formula digitabile liberamente; il segnaposto mostra la forma classica NDVI `(NIR - Red) / (NIR + Red)`. Un’anteprima tokenizzata in sola lettura sopra di essa visualizza i chip di banda, i numeri e i flag come token sconosciuti.
* **Banner di validità**— grigio «Vuoto — non verrà applicato alcun indice»; verde “Espressione valida”; rosso con l’errore di analisi specifico (banda sconosciuta, banda ambigua rilevata da più telecamere, parentesi mancante, …); oppure giallo quando l’espressione è valida ma**costante** (ad es. `X/X`, oppure un denominatore NDVI digitato con `−` invece di `+`) — una costante mappa l’intero fotogramma su un unico colore.
* Viene visualizzato un avviso separato di colore ambra se l’espressione applicata è corretta ma il **fotogramma in tempo reale è uniforme** (scena piatta o saturata) — il collasso dell’istogramma viene rilevato automaticamente.
* **Applica LUT**(attivato per impostazione predefinita; disattivato = espansione in scala di grigi),**Livello**2/3/5/7 stop (impostazione predefinita 7 stop) e gli input**Min / Max**che affiancano la barra del gradiente. Il valore predefinito per Min è**0,2**— ingrandisce la rampa di colori nell’intervallo rilevante per la vegetazione, mentre i valori inferiori vengono visualizzati in scala di grigi; imposta Min su −1 per l’intero intervallo dell’indice (il pulsante**Reset** ripristina il range da −1 a +1). Il valore predefinito per Max è 1.
* **Istogramma in tempo reale** della distribuzione dell’indice — barre con scala radice quadrata, linee dei percentili p2/p98 in ambra, una linea mediana bianca e indicazioni delle code fuori intervallo (&quot;◀ N% &lt; lo&quot; / &quot;hi &lt; N% ▶&quot;) che diventano ambra al di sopra dell’1% come indicazione per ampliare la finestra Min/Max.
* **Applica**applica l’espressione al flusso in tempo reale; le modifiche alla LUT si applicano in tempo reale senza premere Applica. Le espressioni sono volutamente**limitate alla sessione** — non vengono conservate tra una sessione e l’altra.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## Il percorso CLI

La stessa catena allineamento → stack → indice, programmabile end-to-end:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` mappa i simboli di un preset ai nomi delle bande dello stack. Due regole ti evitano un’esecuzione fallita:

* **I simboli distinguono tra maiuscole e minuscole** e devono corrispondere esattamente ai nomi dei canali del preset — i preset utilizzano lettere minuscole (quelli di NDVI sono `red`,`nir`; controlla `--list-presets`). `--channel red=Red_660` funziona; `--channel RED=660` fallisce con un errore `channel_map missing entries`.
* Il lato della banda deve specificare il nome di una banda nello stack allineato (`lattice align-info --profile align.json` ne elenca l’elenco). La modalità offline accetta anche indici di banda con base 0, ad esempio `--channel red=0 --channel nir=1`.

`lattice index` funziona anche completamente offline su un TIFF multibanda allineato e salvato:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Preimpostazioni dell’indice

`lattice index --preset` (e la [sandbox Indice/LUT](../image-viewer-gui/index-lut-sandbox.md) della scheda Immagine, che utilizza lo stesso motore) include questi **22 preset**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Eseguire `chloros-cli lattice index --list-presets` per le formule e i simboli dei canali di ciascun preset, e `--list-gradients` per i gradienti di colore disponibili. Le formule personalizzate utilizzano `--formula EXPR` con la stessa sintassi del Calcolatore di indici. Si noti che questo elenco di preset è specifico per il motore di indici LATTICE: il menu a tendina &quot;Elaborazione&quot; nelle Impostazioni di progetto per le immagini importate presenta un elenco diverso (vedere [Formule degli indici multispettrali](../project-settings/multispectral-index-formulas.md)).

Il set completo di flag (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, manopole di allineamento e deformazione per `--live`, e altro) è documentato nella [Guida di riferimento di CLI § Indice / Matematica della vegetazione](../reference/cli-reference.md#index--vegetation-maths); gli equivalenti di SDK si trovano nel [Riferimento SDK](../reference/sdk-reference.md).

## Acquisizione dei prodotti dell’indice da un array mono

Con un array collegato e un’espressione di indice applicata, `array-capture` (o l’opzione **Acquisisci tutto** dell’interfaccia grafica) salva i livelli di esportazione per ciascuna telecamera *e* il rendering dell’indice — `--index`/`--no-index` attiva questa opzione su CLI, e l’acquisizione include per impostazione predefinita tutti i livelli applicabili. Il contributo di una telecamera mono a ciascun gruppo di acquisizione è costituito dalla sua singola banda ai livelli raw/debayered (scala di grigi)/radianza/riflettanza, più il composito a indice combinato condiviso quando l’array funziona in modalità combinata. Vedere [Array multicamera § Acquisizione](arrays.md#capturing-monitoring-vs-analysis).
