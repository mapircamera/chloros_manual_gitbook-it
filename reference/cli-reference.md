# Guida di riferimento Chloros CLI

**Versione:**

1.3.0**Generata:**29/07/2026 19:19 ·**Aggiornata:** 06/11/2026**Destinatari:** Ottimizzato per l’utilizzo da parte di modelli di linguaggio di grandi dimensioni (LLM); leggibile da esseri umani.**Ambito:** Tutti i sottocomandi di `chloros-cli` destinati agli utenti, con opzioni ed esempi copiabili e incollabili.

Questo documento costituisce il riferimento completo per lo strumento da riga di comando `chloros-cli` fornito con MAPIR Chloros. È volutamente esaustivo in modo che un LLM (o un utente umano) possa comporre qualsiasi flusso di lavoro supportato dagli elenchi riportati di seguito senza dover esaminare il codice sorgente.

Se ti servono solo i punti salienti, vai a:
- [Guida rapida in cinque minuti](#five-minute-quickstart)
- [Flusso di lavoro per il primo collegamento della telecamera LATTICE](#lattice-camera-first-connect-workflow)
- [Flusso di lavoro per il primo collegamento del sensore DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Modalità di acquisizione, registratori e rielaborazione offline](#capture-modes-recorders--offline-reprocess)

> **Vuoi gestire Chloros da un assistente AI?** Claude Desktop, Claude Code, Cursor e altri client MCP possono gestire l’elaborazione, le telecamere e gli array LATTICE, nonché i sensori di luce DAQ tramite `chloros-mcp`, il server MCP che viene installato con l’Python SDK (e con il file .deb per Linux, in `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. La configurazione, le opzioni e gli strumenti sono disponibili nella sezione [Server MCP (assistenti AI)](sdk-reference.md#mcp-server-ai-assistants). Un assistente dotato di shell può inoltre eseguire direttamente ogni comando presente in questa pagina.

> **Modifiche nella versione 1.3.0** (come rilevato da uno script o un flusso di lavoro esistente della versione 1.2.0).
>
> **Modifiche che uno script della versione 1.2.0 potrebbe dover gestire:**
>
> - **Si applica `--preset`.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` e `align-stream` sono stati eliminati nella versione 1.2.0 e funzionava con le impostazioni predefinite della telecamera. Ora imposta il formato, l’esposizione, il guadagno e la frequenza dei fotogrammi del preset: `--preset high_speed` acquisisce a 8 bit a 5 ms, 6 dB, 60 fps. Anche `lattice cam-connect --preset` mantiene il formato del preset (la versione 1.2.0 impostava la fotocamera a 12 bit dopo l’applicazione del preset). Vedi [Opzioni comuni](#common-options-shared-by-most-subcommands).
> - **Il formato pixel Bayer su una fotocamera M3M (mono) corrisponde alla sua profondità.** I preset indicano formati Bayer che un sensore mono non può acquisire: nella versione 1.2.0 la fotocamera rifiutava il formato e manteneva quello che aveva, quindi `--preset high_speed` non era a 8 bit in quel caso. Ora è il formato Mono della stessa profondità: `high_speed` e `--pixel-format BayerRG8` danno `Mono8`, `high_quality`, `triggered` e `BayerRG12` danno `Mono12`. In un array misto M3C + M3M ogni telecamera adotta il formato condiviso nella propria famiglia, e `network-analysis` / `analyze-array --pixel-format` assegnano a ciascuna telecamera quella profondità. Lo stesso vale al contrario: un’M3C a cui viene richiesto un formato Mono adotta il formato Bayer di quella profondità, quindi un array il cui master è un’M3M non riapre più le proprie telecamere M3C in modalità Mono. Vedi [Telecamere Mono (M3M) a banda singola](#mono-m3m-single-band-cameras).
> - **Le esportazioni `lattice array-capture` sono allineate per impostazione predefinita, così come quelle dell’app.** I file `debayered`, `display`, `radiance`, `reflectance` e i file di indice di ogni membro vengono allineati al fotogramma della telecamera principale e ritagliati in base alla sovrapposizione dei membri (più piccola del sensore di un valore pari agli offset delle telecamere, `XMP-mapir:AlignmentJSON` `"applied": true`), quindi un’acquisizione CLI corrisponde agli stessi file dell’acquisizione della stessa scena effettuata dall’app Chloros; `raw` rimane in formato nativo. In precedenza, un’acquisizione senza `--aligned` salvava ogni file nelle dimensioni native, senza correzione prospettica. `--no-aligned` mantiene questa impostazione. Lo stesso comportamento predefinito si applica a `--smart`, `--continuous` e `--interval`, nonché all’SDK e a `chloros-mcp`. Vedi [modalità `array-capture`](#array-capture-modes).
> - **`lattice array-capture --compression` utilizza come impostazione predefinita `none`.** I file vengono scritti senza compressione (circa 6,3 MB per fotogramma a piena risoluzione) per ottenere la velocità di registrazione continua più elevata; la versione 1.2.0 scriveva in formato `deflate` (circa 4,1 MB). Specificare `--compression deflate` per ottenere file più piccoli. Entrambe le modalità sono senza perdita di dati.
> - **I formati PNG e JPEG accettano solo i prodotti `display` / `preview` e quelli con indice.** Una cattura `--format png` o `jpg` con un livello che un file a 8 bit non può contenere viene rifiutata prima che venga catturato alcunché (400 HTTP, uscita `1`), così come l’impostazione predefinita `--processing all` con `--format png`/`jpg`. Vedi [Livelli di esportazione dell’acquisizione](#capture-export-levels-the-all-default).
> - **`lattice dls` richiede `--mac`.**Non esiste più un MAC predefinito (era l’indirizzo di un’unità); `daq pool-discover --only ble` individua quello di un sensore.**`lattice dls --camera SERIAL`** è nuovo.
> - **Altri errori fanno uscire `1`.** `process` quando non è stato possibile verificare le esportazioni di un'esecuzione o se ha perso un'immagine, sebbene gli altri prodotti siano stati scritti; un comando di acquisizione con qualsiasi file che non è riuscito a scrivere (gli altri file vengono comunque salvati); `lattice continuous` e `multi-stream` in caso di disco pieno; `array-burst` / `array-record` quando il registratore segnala un errore; `array-build-video` quando un prodotto richiesto è stato saltato o non è stato creato come richiesto; `project burst` / `stream` quando un’acquisizione fallisce a metà; `project sensor log` quando la registrazione non contiene alcuna lettura; `lattice exposure --set`, `gain --set` e `configure --set` con un valore rifiutato dalla telecamera. `lattice network-analysis --json` termina come il riepilogo (`1` per `needs_force_slip`). Al contrario: `daq pool-disconnect` di un sensore che non è più nel pool termina con `0`. Vedi [Codici di uscita](#exit-codes).
> - **Un `lattice array-connect` lento non fallisce mai con l'array collegato.** Attende fino a 600 s (180 s nella versione 1.2.0; `project connect` 90 s), stampando la fase del backend man mano che procede, e una connessione non completata entro quel tempo viene annullata dal backend — ogni telecamera viene riportata allo stato in cui è stata trovata — quindi un `Error:` (uscita `1`) indica che nessun array è collegato. Nella versione 1.2.0 una connessione avviata subito dopo l’accensione delle telecamere poteva protrarsi oltre il tempo di attesa e collegare l’array dopo che il comando era fallito. Ora Ctrl+C annulla la connessione. Vedi [Connessioni persistenti](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` mantiene il sensore completo.** In assenza di `--width` / `--height` / `--binning`, ogni telecamera è impostata sul sensore completo e la frequenza dei fotogrammi scende al livello consentito da questo computer e dalla rete; l’output di connessione elenca opzioni più veloci (binning o un ROI più piccolo) con le rispettive frequenze previste. La versione 1.2.0 ha ridotto o raggruppato i fotogrammi per mantenere alta la frequenza. Vedi [Opzioni `array-connect`](#array-connect-options).
> - **Una rete lenta non impedisce più il funzionamento di un array.** Ogni telecamera viene sincronizzata in base alla propria larghezza di banda disponibile e la frequenza dei fotogrammi diminuisce di conseguenza. Laddove non sia possibile sincronizzare le telecamere a una velocità così bassa, l’array funziona in modalità di emissione seriale (`slip-emit-and-capture`): ogni fotogramma arriva, ma le telecamere non vengono esposte contemporaneamente. `--force-tier sim-capture-sim-emit` richiede l’esposizione simultanea. Vedi [FPS dell’array e modello a raffica](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` indicano a quale connect si applica**: in assenza di `--pixel-format`, i formati sono quelli impostati da connect (Mono12 / BayerRG12, prima era BayerRG8) e la scelta [12 bit compressi](#packed-12-bit-transport---packed). Lo stesso comando può quindi indicare una frequenza dei fotogrammi diversa rispetto alla versione 1.2.0.
> - **I percorsi relativi si riferiscono alla directory in cui si esegue il comando.** `-o` e gli altri percorsi aperti dal backend (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), l’`output` di un passaggio della ricetta, e il percorso predefinito `output/` di `lattice index --live` vengono risolti in base alla cartella da cui si esegue `chloros-cli`. Nella versione 1.2.0 il backend li risolveva nella propria cartella di installazione, il che, su una build installata, causava un errore con il messaggio "Accesso negato". Vedi [Convenzioni](#conventions).
> - **`lattice index` viene eseguito sul backend in ogni sua forma**, come gli altri comandi `lattice`, quindi avviare prima il backend. L’indice offline (`--input`) viene calcolato dal backend, che legge il file TIFF e scrive i risultati accanto ad esso o in `-o`, mentre `--list-presets` e `--list-gradients` stampano le tabelle del backend. Su Windows, ora funzionano: in precedenza, la CLI di Windows rispondeva a ogni forma di `lattice index`, `--live` inclusa, con `Error: lattice_sdk not available`. Su Linux, l’indice offline veniva eseguito all’interno della CLI senza backend e senza login; ora richiede entrambi, come ogni altro comando CLI. Vedi [Indice / Calcoli sulla vegetazione](#index--vegetation-maths).
> - **Nuovi file accanto ai prodotti.** Ogni prodotto di radianza, riflettanza e indice in cui la separazione delle bande ha limitato un pixel riceve una maschera nella cartella `clamp_masks/` accanto alla cartella del prodotto, e la stampa include una riga `clamp mask:`. Vedi [Maschera di clamping della separazione delle bande](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Orientamento della fotocamera.** Un'acquisizione con una singola fotocamera salva i file `raw` e `debayered` così come sono stati letti dal sensore e registra il capovolgimento/la rotazione della fotocamera nel file XMP; `process` applica tali informazioni ai prodotti che crea. I file membri dell’array rimangono in formato nativo a meno che non si passi il nuovo `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Vedi [Orientamento della fotocamera](#come-è-strutturata-una-cartella-di-acquisizioni).
> - **Un nome di livello di esportazione diverso da "uno" viene rifiutato.** `lattice capture --processing` e ogni elenco `--levels` accettano `raw`, `debayered`, `radiance`, `reflectance` o `display` (o un alias come `preview`; `--processing` e anche `all`). Qualsiasi altro nome viene rifiutato prima che la telecamera venga toccata (400 HTTP `UNKNOWN_LEVEL`, uscita `1`); 1.2.0 ha salvato `raw` per esso e si è chiuso con `0`.
> - **`daq pool-set-cap` richiede `--cap-id`.** Senza di esso il comando termina con `2`; 1.2.0 ha rimosso la correzione del tappo, lasciando il sensore nudo. `--cap-id none` richiede il sensore "nudo".
> - **`lattice resolution --binning-mode` non è più presente.** Il binning è sempre "Average"; il flag viene comunque accettato, ma viene visualizzato un avviso e non viene eseguita alcuna operazione. Vedi [Ottimizzazione del sensore](#sensor-tuning).
> - **`process --debayer texture-aware` viene rifiutato per le acquisizioni LATTICE.** Per una cartella contenente acquisizioni LATTICE, `process` termina con `1` dopo l’importazione, indicando quante ce ne sono, prima di elaborare qualsiasi cosa. Le acquisizioni LATTICE vengono sempre sottoposte al demosaic standard, quindi la versione 1.2.0 accettava l’opzione ma la ignorava; nemmeno l’app Chloros la offre per queste acquisizioni. Vedi [`chloros-cli process`](#chloros-cli-process).
> - **Le fotocamere tornano allo stato in cui sono state trovate — a meno che non si utilizzi `--keep-settings`.** `lattice array-disconnect`, ogni comando transitorio (`multi-capture` senza array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) e `lattice continuous` riportano ogni telecamera allo stato in cui l’hanno trovata: il cablaggio del trigger e della linea di sincronizzazione, il PTP e tutte le impostazioni della fotocamera modificate durante la sessione (esposizione e guadagno, obiettivo AE, limiti e smorzamento, bilanciamento del bianco, livello del nero, formato pixel, binning e ROI, capovolgimenti, frequenza dei fotogrammi). `--keep-settings`, invece, salva le impostazioni di ciascuna fotocameranella memoria della telecamera stessa, in modo che si accenda con quelle; `--clear-settings` ripristina quella memoria alle impostazioni di fabbrica. In entrambi i casi le telecamere vengono lasciate in funzionamento libero. Un progetto imposta le proprie telecamere a partire dalle proprie impostazioni salvate quando le collega. Vedi [Memoria della telecamera](#camera-memory----keep-settings----clear-settings).
> - **Ogni modalità salva un progetto allo stesso modo.** Qualsiasi modifica apportata a una telecamera, a un array o a un sensore di luce collegati a un progetto — dall’app, dalla CLI, dall’SDK o dal server MCP — viene salvata nel file `cameras.json` / `sensors.json`, e l’apertura del progetto in qualsiasi modalità configura i dispositivi in base a esso. In precedenza, solo l’app salvava il progetto e il salvataggio successivo sovrascriveva qualsiasi altra cosa. Vedi [Un unico record di progetto, in ogni modalità](#one-project-record-every-mode).
> - **I file generati da più telecamere indicano il nome di ciascuna di esse.** Uno stack multibanda (`align-apply`, `align-stream`, `index --live --save-multiband`), un indice in tempo reale creato dalle bande di diverse fotocamere e l’immagine `composite/` di un’acquisizione in array contengono il numero di serie, il modello e il pacchetto di calibrazione di ciascuna fotocamera di origine sotto forma di elenchi (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), mentre i dati EXIF `SerialNumber` elencano i numeri di serie separati da virgole. In precedenza non contenevano alcuna telecamera, oppure riportavano solo il numero di serie della prima telecamera. Vedi [File creati da più telecamere](#files-made-from-several-cameras).
> - **Solo un file `raw` contiene i tag di calibrazione della telecamera.** Il set compatibile con MicaSense (`BlackLevel`, `RadiometricCalibration`, il file per canale `Band{R,G,B}RadiometricCalibration`, i set `MicaSenseVignetting*`) e il payload `CalibrationJSON` descrivono i valori grezzi del sensore, pertanto sono contenuti esclusivamente nel file di acquisizione `raw`. Tutti gli altri file — i file di acquisizione `debayered`, `display`, `radiance`, `reflectance` e i file di indice, i frame `lattice continuous` e `multi-stream`, e ogni prodotto `process` — riporta l’identificativo della telecamera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) e, al momento dell’acquisizione, i dati specifici del fotogramma (`XMP-mapir:FrameJSON`: esposizione, guadagno, temperatura, formato dei pixel, ora di acquisizione). Nella versione 1.2.0 includevano anche il set di dati grezzi, che uno strumento che lo applica (Pix4D, Metashape) applicava ai pixel che non sono valori grezzi. `process` individua la calibrazione di un file di questo tipo tramite il suo numero di serie e `CalBundleSha256`.
>
> **Altre modifiche:**
>
> - **La frequenza dei fotogrammi automatica di un array si adatta a quella fornita dall’array stesso.** Una frequenza che perde più dell’1% dei propri trigger viene ridotta, in base a una valutazione effettuata su diverse osservazioni prolungate anziché su una singola, e la frequenza da cui è stata ridotta viene riprovata di tanto in tanto. Una frequenza impostata manualmente rimane invariata.
> - **L’esposizione automatica è più stabile.** Si mantiene costante in condizioni di luce costante, si stabilizza entro pochi secondi da un cambiamento, ignora le piccole aree sovraesposte e ripristina il funzionamento di una fotocamera che smette di rispondere in pieno sole. Vedi [Come funziona l’esposizione automatica](#how-auto-exposure-behaves).
> - **`process` conta solo le tue immagini e registrazioni.** Se si esegue nuovamente l’operazione su una cartella che contiene l’`camera_cal/` di un progetto, i file di calibrazione non vengono più conteggiati («96 immagini trovate» su un set di 90 immagini), e un progetto che li aveva già acquisiti li elimina.
> - **`lattice multi-test` parte dallo stato di accensione delle telecamere.** Attiva il PTP delle telecamere per l’esecuzione, attende che i loro orologi si sincronizzino e si stabilizzino prima del primo trigger — ogni telecamera rimane sincronizzata per 120 s, circa 2 minuti dall’accensione (`--ptp-settle SECONDS` lo imposta), poiché un orologio appena sincronizzato presenta uno scostamento fino a ~200 us per circa 2 minuti — e riporta il PTP di ciascuna telecamera allo stato in cui lo ha trovato. Non si attende nuovamente una telecamera ancora sincronizzata da un test precedente. In precedenza, su una rete con un unico switch in cui nessun DAQ-E rispondeva, il PTP veniva disattivato e l’orologio a funzionamento libero di ciascuna telecamera veniva interpretato come un orologio PTP che non si sarebbe sincronizzato (`PTP is not locked`, ad ogni esecuzione). Vedi [Multi-telecamera (sessioni transitorie)](#multi-camera-transient-sessions).
> - **Una registrazione `.daq` indica se il cappuccio è stato scelto o presunto.** I modelli DAQ-U, DAQ-M e DAQ-E vengono forniti senza cappuccio e Chloros non è in grado di rilevare se ne sia stato montato uno; pertanto, a un sensore collegato senza un cappuccio specificato viene assegnato per impostazione predefinita il codice `sunshine_cosine`, una *presunzione*. Ogni registrazione Chloros (schema 1.30) ora memorizza `cap_id_source` accanto a `cap_id`: `operator` se hai assegnato un nome al cappuccio (incluso `none`), `auto_default` se era quello predefinito. Elaborando una registrazione il cui cappuccio è stato ipotizzato, con l’impostazione del cappuccio DAQ del progetto lasciata su Auto, viene visualizzato un messaggio `[daq-dls] WARNING` che indica il nome del file: un DAQ-U senza correzione "sunshine" registra valori circa 30 volte superiori. **Assegnate un nome al filtro che avete applicato** al momento della connessione, oppure impostate il filtro DAQ del progetto, e l'avviso scomparirà.
> - **Niente più finestre di console "pip" su Windows.** Su un PC con Python installato, il backend e ciascuno dei suoi image worker eseguivano `pip` una volta all’avvio, e ciascuno apriva una finestra della console. Ora non lo fanno più.
> - **Le registrazioni `.daq` in tempo reale sopravvivono a un'interruzione di corrente.**Una registrazione viene scritta con un journal e una sincronizzazione completa del disco ad ogni commit, a livello del sensore, quindi un disco lento rallenta il file, ma mai le letture. Una registrazione interrotta a metà di un commit (interruzione di corrente, terminazione forzata) può lasciare accanto a sé `<name>.daq-journal`:**copiare i due file insieme.**Chloros ripristina il journal alla prima lettura del file, quindi la registrazione si apre con tutto ciò che è stato commesso prima dell’interruzione (vengono persi circa gli ultimi 3 s); nella versione 1.2.0 un’interruzione di corrente nel momento sbagliato poteva rendere l’intero file illeggibile (“l’immagine del disco del database è danneggiata”).
> - **Una registrazione su un disco occupato non blocca più il sensore.** `daq pool-record` mette in coda le letture e le scrive su un thread dedicato; una lettura che non è stato possibile scrivere viene conteggiata (`daq pool-list` mostra `N write error(s)` con l’ultimo errore) invece di bloccare il sensore (una scrittura ha richiesto 8,4 s accanto a quattro telecamere che salvavano file TIFF).
> - **`lattice multi-capture` senza array non perde più i fotogrammi salvati quando un trigger va in timeout.** Il timeout del trigger di una telecamera (`SC_ERR_TIMEOUT -1011`, osservato con quattro telecamere su uno switch 1 GbE) termina l’esecuzione a quel gruppo di fotogrammi: i gruppi già salvati vengono elencati con `Saved:`, poi l’errore, uscita `1`. Nella versione 1.2.0 si verificava un errore HTTP500 senza alcun elenco, sebbene i file fossero presenti sul disco.
> - **Le telecamere con trigger software non tralasciano più un trigger su due.** Una telecamera in modalità trigger software ora mette in coda un trigger che arriva durante la sua lettura (oppure Chloros attende che sia pronto), come già facevano le telecamere sincronizzate via cavo.
> - **`daq pool-discover` assegna un nome a un DAQ-E-S `DAQ-E-S`** nella colonna `MODEL` (e quella di `--json` nella colonna `model`). Per ogni sensore Ethernet veniva indicato `DAQ-E`; quello che non risponde al controllo di identità entro circa 2 s rimane comunque elencato come `DAQ-E`.
> - **Prodotti relativi agli indici LATTICE migliorati** (NDVI e altri indici) a partire da `process`, in ogni modalità di elaborazione.
> - **Telecamere OCN: prodotti relativi a radianza, riflettanza e indici migliorati in condizioni di luce solare.** Dopo la separazione delle bande viene applicata una correzione della flotta; le telecamere RGN rimangono invariate, mentre i valori OCN differiscono da quelli della versione 1.2.0. Ciò vale anche per un'acquisizione OCN effettuata con una versione precedente, quando viene elaborata su un computer che conserva la calibrazione utilizzata al momento dell'acquisizione (Chloros conserva la calibrazione di una fotocamera una volta che questa è stata collegata).
> - **`lattice continuous` registra il fotogramma grezzo.** Un'esecuzione di `tiff` ha scritto fotogrammi demosaicati a 3piani, da cui `process` non può partire. I suoi fotogrammi sono ora il file `raw` che una cattura ad array scrive: un piano mosaico ai valori propri del sensore, non compresso, con il pacchetto di calibrazione. Uno script che legge tre piani da questi file ne legge uno solo.
>
> - **Ogni acquisizione salva la lettura DAQ utilizzata, proprio come fa la GUI.** `lattice capture`, `lattice continuous` (un `.daq` per fotogramma), `array-capture` e `multi-capture` con un array collegato (e l’SDK) scrivono il file `.daq` e il relativo `.match.json` ogni volta che un DAQ viene associato o inserito in un pool, inclusa un’acquisizione solo raw; nella versione 1.2.0 una cattura raw o debayered ne salvava solo una con `--force-daq`. `--no-daq` la disattiva, `--daq-tolerance-ms MS` imposta la tolleranza di corrispondenza per quella cattura, e il nuovo `daq pool-bind` associa una telecamera o un array a un DAQ dalla CLI. Con `.daq`, un'acquisizione salvata viene abbinata in `process` solo alle telecamere specificate. Una cattura ora attende la lettura successiva del DAQ in modo da poter fondere le letture su entrambi i lati dell’esposizione. Vedi [La lettura del DAQ salvata da una cattura](#the-daq-reading-a-capture-saves) e [Una lettura del DAQ lontana dall’esposizione](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** aggiunge l’indice al progetto per le bande di ciascuna telecamera, includendo un nuovo progetto. Nella versione 1.2.0, un nome non ancora presente nell’elenco del progetto veniva ignorato con `Unknown index name(s) ignored`.
> - **`process` accanto a un array attivo:**non riavvia più un backend che gestisce telecamere o DAQ e attende fino a 15 s per un'esecuzione che sta terminando.**Ctrl+C** interrompe l'esecuzione nel backend, non solo nella CLI. Vedi [Note su `process`](#notes).
> - **Una cartella di progetto troppo profonda per il limite di 260 caratteri del percorso di Windows** viene rifiutata immediatamente, sostituita dalla cartella più lunga che rientra nel limite, invece di eseguire un'esecuzione che non scrive nulla. Il programma di installazione di Windows ora offre l’opzione «Abilita percorsi lunghi di Windows» e un’esclusione di Microsoft Defender per il backend, entrambe selezionate per impostazione predefinita. Vedi [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** esegue la cattura continua propria del backend: una barra di avanzamento ogni 2 secondi, un riepilogo anziché un elenco per singolo file e un'uscita con codice diverso da zero in caso di file non scritti o di disco pieno. Vedi [modalità `array-capture`](#array-capture-modes).
> - **Il primo `array-capture` dopo `array-connect`** attende fino a 20 s per il primo gruppo di fotogrammi dell’array invece di restituire un errore HTTP503 “Array still warming up”.
> - **Esposizione dell’array in un’area di interesse (ROI):** una ROI di piccole dimensioni non limita più l’esposizione di un elemento dell’array al periodo di funzionamento libero del sensore (4,9 ms a 768 × 304). Vedi [Esposizione su un elemento dell’array](#exposure-on-an-array-member).
> - **L’esposizione automatica non riduce più la frequenza dei fotogrammi automatica di un array.** Un array sincronizzato via cavo funziona alla frequenza consentita dalla rete e dalla velocità di lettura delle telecamere; l’esposizione automatica riempie il periodo corrispondente a tale frequenza, quindi aumenta il guadagno. Un’esposizione manuale più lunga del periodo (`lattice exposure --set`, `configure --set`) rallenta il trigger per adattarsi ad essa invece di essere rifiutata, e la frequenza torna normale quando l’elemento torna in modalità automatica. Vedi [Esposizione su un elemento dell’array](#exposure-on-an-array-member).
> - **Gli array producono meno gruppi di fotogrammi incompleti** e meno errori di acquisizione 503 (“Nessun nuovo gruppo di fotogrammi sincronizzato”): l’ingresso di trigger di ciascuna telecamera slave è ora privo di glitch.
> - **Trasporto compresso a 12 bit** (`--packed`, impostazione predefinita `auto`) aumenta la frequenza dei fotogrammi dell’array a sensore intero di circa un terzo. Ora viene applicato `array-connect --packet-size` (la versione 1.2.0 lo accettava ma lo ignorava). Vedi [Trasporto compresso a 12 bit](#packed-12-bit-transport---packed).
> - **Un array le cui telecamere si ripristinano automaticamente dopo un ciclo di alimentazione.** Dopo un ciclo di alimentazione dello switch, l’array di `array-connect`-riattiva le telecamere quando queste rispondono e riprende lo streaming con il PTP riattivato; nella versione 1.2.0 si arrestava definitivamente e richiedeva un nuovo `array-connect`. `array-status` segnala `ptp_enabled: False` (e `ptp_off:`) mentre una telecamera rileva il PTP disattivato, mentre in precedenza riportava l’impostazione con cui l’array era connesso. Vedi [Connessioni persistenti](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` registra senza che la GUI sia aperta.** Con la vista combinata della GUI chiusa, il backend genera la vista stessa: l’NDVI dell’array, oppure l’indice che si passa con il nuovo `--expr`. Una registrazione che non scrive alcun fotogramma ora fallisce (`no-frames: …`, uscita `1`). In precedenza, la registrazione avveniva solo mentre la vista combinata dell’interfaccia grafica era aperta, e una registrazione che non scriveva nulla usciva comunque da `0`. Vedi [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Una telecamera già detenuta da Chloros non viene aperta una seconda volta.** Un comando che tentasse di aprirla restituisce invece un errore HTTP 409 `CAMERA_HELD`, indicando chi la detiene e il comando che la libera. Ciò include una telecamera che un altro comando sta utilizzando in quel momento (un `multi-test` o un `multi-capture` che sta aprendo le proprie telecamere, un `lattice capture`): il secondo comando viene rifiutato finché il primo non termina. Riaprirla potrebbe interrompere lo streaming del dispositivo che la detiene o causare un crash del backend. Vedi [Connessioni persistenti](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **I comandi transitori per più telecamere trasmettono tutte le telecamere o falliscono.** `lattice multi-capture` senza array collegato, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` e `index --live` terminano con `1` indicando una telecamera che non ha trasmesso alcun fotogramma. I flussi saltano un gruppo di fotogrammi in cui manca una telecamera e si interrompono dopo tre di fila; `multi-test` stampa una riga per ogni trigger, `(no frame)` dove una telecamera ha saltato il trigger; `align-stream` esce anche da `1` a causa di un errore di elaborazione. Ogni esecuzione riporta quindi ogni telecamera alle condizioni in cui l’ha trovata: il cablaggio del trigger e della linea di sincronizzazione, il suo PTP (`PtpEnable`) e tutte le impostazioni della telecamera modificate durante l’esecuzione — esposizione e guadagno, target AE, limiti e smorzamento, bilanciamento del bianco, livello del nero, formato pixel, binning e ROI, capovolgimenti, frame rate — a meno che `--keep-settings` / `--clear-settings` (vedi [Memoria della telecamera](#camera-memory----keep-settings----clear-settings)). In precedenza, questi comandi conservavano tutti i fotogrammi in arrivo e uscivano da `0` (`align-stream` generava un errore di lavoro come `Note:`), e lasciavano ogni telecamera in `TriggerMode` `On`. Inoltre, impostavano l’esposizione di ciascuna telecamera in base alla scena precedente prima di conservare un fotogramma (al massimo pochi secondi), sul target un array collegato avvia ciascuna telecamera a: 40 multispettrali, 60 RGB (impostato da `--brightness N`), mantenendo tali valori per tutta la sessione; lo stesso vale per un singolo comando `lattice capture` di una telecamera che Chloros non gestisce. In precedenza, il primo fotogramma di ciascuna telecamera veniva acquisito al limite di esposizione della telecamera stessa, circa 100 ms, indipendentemente dalla luce, e spesso risultava tagliato. Vedi [Multi-Camera (Sessioni transitorie)](#multi-camera-transient-sessions).
> - **I comandi multi-camera transitori condividono la rete come fa `array-connect` e aprono solo `--serials`.** Ogni telecamera viene sincronizzata in base alla propria quota del collegamento host, quindi quattro telecamere funzionano su un collegamento da 1 Gbps; in precedenza, le quattro richiedevano complessivamente 125 MB/s, utilizzando tutta la larghezza di banda da 1 GbE, e ogni fotogramma subiva la perdita di pacchetti. Un array che la rete non è in grado di supportare viene rifiutato prima che qualsiasi telecamera inizi lo streaming. `--serials` ora seleziona le telecamere, a partire da quella che attiva il trigger; in precedenza, ogni comando apriva tutte le telecamere presenti sulla rete, indipendentemente da quanto specificato da `--serials`. Un errore dovuto alla perdita di pacchetti GVSP ora viene segnalato come tale, invece di indicare il cavo di sincronizzazione M8. Vedi [Multi-telecamera (sessioni transitorie)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` lascia le telecamere in funzionamento libero, così come sono state rilevate.** `TriggerMode`, `Off` e tutto il resto così come rilevato dalla connessione — le impostazioni del trigger e della linea di sincronizzazione (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP e le impostazioni della telecamera modificate durante la sessione — in modo che un singolocomando "-camera" su un ex membro acquisisce i fotogrammi in una volta sola, e il master non rimane più su `TriggerSource` `Software`. `--keep-settings` salva le impostazioni della sessione nella memoria di ciascuna telecamera, mentre `--clear-settings` ripristina tale memoria (vedi [Memoria della telecamera](#camera-memory----keep-settings----clear-settings)). `lattice continuous` imposta inoltre una telecamera che rileva come «armata» in modalità «free-run» per la corsa (e la riporta allo stato precedente al termine della stessa), mentre una corsa senza frame di uscita attiva `1`.
> - **`lattice configure --get` / `lattice exposure`** stampano un `note:` con l’reale della telecamera o il guadagno mentre è attiva la modalità automatica. Un `lattice exposure --set` / `configure --set` rifiutato indica l’intervallo accettato dalla telecamera e ripristina la modalità automatica invece di disattivare l’esposizione automatica. Vedi [Ottimizzazione del sensore](#sensor-tuning).
> - **Nuovi comandi per la telecamera:**`lattice highlights` (riduce la luminosità delle superfici bianche nella rappresentazione RGB) e `lattice exposure --brightness-auto`.**`lattice multi-detect`** individua le telecamere cablate in un ciclo di sondaggio e imposta come predefinita la telecamera RGB.
> - **Nuovi comandi DAQ:** `daq pool-bind`; `pool-enable-ptp`; e per le impostazioni specifiche di un DAQ-E: `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` e `pool-update-firmware`. Vedi [Impostazioni del dispositivo DAQ-E](#daq-e-device-settings).
> - **Un DAQ-E che smette di trasmettere viene ricollegato dal backend.** Quando lo stream di un DAQ-E in pool smette di trasmettere (a causa di un riavvio dello switch che lo alimenta, un'interruzione del collegamento con la telecamera), il backend lo ricollega con le impostazioni precedenti, riprovando a intervalli crescenti finché rimane inattivo. Una registrazione in corso su quel dispositivo prosegue nello stesso `.daq` con l’interruzione contrassegnata nel file, mentre `pool-record --stop` elenca ogni interruzione. Nella versione 1.2.0 il sensore rimaneva inattivo fino a `pool-disconnect` + `pool-connect`, il che comportava anche la fine della registrazione. Vedi [Un DAQ-E che smette di trasmettere](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** segnala se i pacchetti del grandmaster PTP raggiungono la rete (`health`); in caso contrario, viene visualizzato un messaggio `WARNING:` su stderr. Vedi [`time-sync`](#chloros-cli-time-sync).
> - **Pacchetto .deb per Linux:** aumenta la dimensione dei buffer di ricezione dei socket del kernel necessari per i flussi delle telecamere (con le impostazioni predefinite del kernel, si verificano perdite di fotogrammi e cali nella frequenza dei fotogrammi), aggiunge un servizio backend che viene eseguito con i privilegi dell’utente e `lattice network` verifica i buffer. La CLI non interrompe mai un backend in esecuzione come servizio, e un aggiornamento riavvia i servizi backend abilitati. Vedi [Esecuzione del backend come servizio](#running-the-backend-as-a-service-headless) e [Host Linux — buffer di ricezione dei socket](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` indica quando non è stato possibile eseguire l’allineamento:**ogni motivo per cui una cattura è stata salvata non allineata (assenza di profilo di allineamento, membri con risoluzioni diverse, un array non sincronizzato via cavo, un frame su cui il profilo non può essere mappato) viene visualizzato come una riga `Alignment:`.**`chloros-cli daq help`** è `daq --help` (è uscito da `2`) e su Linux la guida elenca i comandi hardware diretti `daq` inclusi nella build.
> - **Jetson:** il pacchetto è `chloros_1.3.0_arm64_jp6.deb`, per JetPack 6, e la sua elaborazione GPU funziona su Orin con CUDA PyTorch di JetPack. Vedi [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Convenzioni

- Tutti i comandi sono preceduti dal prefisso `chloros-cli`. Su Windows il binario è `chloros-cli.exe`; su Linux/Jetson è `chloros-cli`.
- Gli argomenti opzionali sono indicati come `--flag`. Gli argomenti posizionali obbligatori sono indicati senza parentesi.
- Laddove è specificato un valore predefinito, omettendo il flag si utilizza tale valore.
- La CLI è un client HTTP leggero che utilizza il backend Chloros (il server su `127.0.0.1:5000` utilizzato anche dall’app desktop). **Quali comandi la avviano:**
  - I comandi principali — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` e `selftest` — utilizzano un backend già in esecuzione, oppure ne avviano uno se non ce n’è nessuno in esecuzione e lo arrestano nuovamente al termine dell’operazione.
  - **I comandi `lattice`, `project` e `daq` non ne avviano mai uno.** Richiedono un backend già in esecuzione: quello dell’app desktop per Windows, il [servizio backend](#running-the-backend-as-a-service-headless) per Linux, oppure uno avviato dall’utente (vedere [Avvia il backend](#avviare-il-backend)). In assenza di un backend, visualizzano il messaggio `Error: Chloros backend not reachable at http://127.0.0.1:5000`, seguito dal comando che lo avvia, e terminano con il messaggio `1`.
- `CHLOROS_BACKEND_URL=<url>` indirizza le famiglie di comandi **`lattice`**,**`project`**e**`daq pool-*`** verso un altro backend (`lattice network-analysis` punta invece a `--backend-url`). I comandi principali fissano deliberatamente `http://127.0.0.1:<port>` e lo ignorano (il letterale IPv4 evita la penalità di Windows di circa 2 secondi per richiesta associata a `localhost`→`::1`). Vedi [Variabili d’ambiente](#environment-variables).
- **I percorsi relativi vengono risolti rispetto alla cartella da cui si esegue `chloros-cli`.** Il backend è il processo che apre le cartelle di output e input, quindi prima di inviare un percorso la CLI lo rende assoluto: `-o` / `--output` (compreso il suo valore predefinito, `output`), `--input`, `--burst-dir`, `--profile`, `--save` e `--mask`, un `output` (impostazione predefinita `output`) di un passo [ricetta](#recipe-dsl), e la cartella predefinita `output` di `lattice index --live`. `-o captures/` scrive quindi in `<current folder>/captures/`, e i percorsi stampati sono assoluti. `~` viene espanso. In presenza di un backend remoto (`CHLOROS_BACKEND_URL` che indica un'altra macchina), i percorsi vengono inviati così come digitati e risolti su quella macchina.
- È richiesto l'accesso con un account Chloros+ per tutte le chiamate SDK/CLI (eseguire `chloros-cli login` una volta per ogni macchina; memorizzato nella cache in `~/.chloros/`).
- Gli esempi utilizzano percorsi Linux; su Windows sostituire `/home/user/...` con `C:/Users/.../...`. La porta seriale di un DAQ-U'è in formato `COM3` su Windows e `/dev/ttyUSB0` / `/dev/ttyACM0` su Linux.
- Le sinossi riportano i flag mutuamente esclusivi come `{--a | --b}`, mentre quelli opzionali in `[...]`.

---

## Sinossi di primo livello

```
chloros-cli [global options] COMMAND [command options]
```

### Opzioni globali

| Flag | Descrizione |
| --- | --- |
| `--backend-exe PATH` | Ignora l’eseguibile del backend rilevato automaticamente. |
| `--port N` | Porta HTTP del backend (impostazione predefinita: `5000`). Utilizzata solo dai comandi principali (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` e `daq` utilizzano `CHLOROS_BACKEND_URL` (`http://127.0.0.1:5000` predefinito). |
| `-v, --verbose` | Abilita l'output dettagliato. |
| `--restart` | Riavvia forzatamente il backend (termina qualsiasi istanza di `backend_server.py` in esecuzione). Un backend eseguito da un servizio systemd (.deb Linux) non viene mai arrestato: il comando stampa invece `sudo systemctl restart <unit>` e termina `1`. |
| `--version` | Visualizza la versione (`Chloros CLI 1.3.0`). |
| `--help` | Mostra la guida di primo livello. |

### Indice dei comandi

| Comando | Scopo |
| --- | --- |
| [`process`](#chloros-cli-process) | Elabora end-to-end una cartella contenente acquisizioni Survey3 o LATTICE. |
| [`login`](#chloros-cli-login) | Autenticare questo computer con un account Chloros+. |
| [`logout`](#chloros-cli-logout) | Cancellare le credenziali memorizzate nella cache. |
| [`status`](#chloros-cli-status) | Mostra lo stato attuale della licenza e dell'autenticazione. |
| [`export-status`](#chloros-cli-export-status) | Visualizza in tempo reale lo stato di avanzamento dell’esportazione di Thread-4 durante l’esecuzione di `process`. |
| [`language`](#chloros-cli-language) | Imposta o elenca la lingua di visualizzazione della CLI (38 lingue supportate). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Cartella di progetto predefinita (condivisa con la GUI). |
| [`update`](#chloros-cli-update) | Verifica e installazione degli aggiornamenti di Chloros (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnostica di sistema + test di funzionamento. |
| [`time-sync`](#chloros-cli-time-sync) | Stato/controllo del grandmaster PTP. |
| [`lattice`](#chloros-cli-lattice) | Controllo e acquisizione della telecamera LATTICE (oltre 45 sottocomandi). |
| [`daq`](#chloros-cli-daq) | Controllo dei sensori spettrali DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Apertura ed esecuzione di un progetto Chloros salvato (telecamere + DAQ). |

---

## Installazione

`chloros-cli` è incluso nel programma di installazione desktop di Chloros su tutte le piattaforme supportate — non è disponibile un download separato per la CLI. L’installazione del pacchetto della piattaforma aggiunge `chloros-cli` al tuo `PATH` insieme all’app desktop e al binario di backend che gestisce.

Download più recenti: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Il programma di installazione include anche comodi script di avvio (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) che aprono una shell CLI pronta all’uso.

### Windows (.exe)

1. Scaricare il programma di installazione per Windows dalla pagina di download.
2. Eseguire `Chloros Installer 1.3.0.exe` e seguire la procedura guidata. Il percorso di installazione predefinito è `C:\Program Files\Chloros\` (la CLI si trova in `C:\Program Files\Chloros\cli\`, che il programma di installazione aggiunge al PATH).
3. Apri un nuovo terminale (`cmd.exe`, PowerShell o Terminale di Windows) in modo che venga rilevato il file aggiornato `PATH`.

```powershell
chloros-cli --version
```

Il programma di installazione aggiunge automaticamente `chloros-cli.exe` al PATH di sistema e include i driver della telecamera LATTICE. Il backend è `C:\Program Files\Chloros\backend\chloros-backend.exe` (la cartella `backend` nella cartella di installazione), che non è presente su `PATH`.

**Impostazioni di sistema.** La pagina *Seleziona attività aggiuntive* del programma di installazione presenta due opzioni sotto **Impostazioni di sistema**, entrambe**selezionate per impostazione predefinita**:

| Opzione (come indicata nel programma di installazione) | Cosa fa | Disattivazione dell’installazione silenziosa |
| --- | --- | --- |
| **Abilita percorsi lunghi di Windows** (consente alle cartelle di progetto di trovarsi a un livello di profondità superiore a 260 caratteri; consigliato) | Imposta il valore `LongPathsEnabled` di Windows a livello di sistema su 1. Windows viene fornito con questa opzione disattivata, e in tal caso nessun percorso può superare i 260 caratteri; i prodotti Chloros si trovano fino a circa 130 caratteri al di sotto della cartella di progetto, quindi una cartella di progetto più profonda viene rifiutata (vedere [Risoluzione dei problemi](#troubleshooting-pointers)). Ha effetto sui programmi avviati dopo l’installazione. Rimane attivo anche dopo la disinstallazione, poiché altri programmi potrebbero farne uso. | `/MERGETASKS="!longpaths"` |
| **Escludi il backend di Chloros dalla scansione di Microsoft Defender**(acquisizione delle immagini molto più veloce; consigliato) | Aggiunge un’esclusione del**processo** di Defender per `chloros-backend.exe` nella cartella di installazione, in modo che la protezione in tempo reale non esegua la scansione di ogni file scritto dal backend, indipendentemente dalla posizione in cui viene scritto. Nessun altro programma viene escluso; l’esclusione viene rimossa al momento della disinstallazione. Se Defender è assente o bloccato da criteri di sicurezza, il passaggio fallisce silenziosamente e l’installazione prosegue. | `/MERGETASKS="!defenderexclusion"` |**Perché l’opzione Defender è importante:** a velocità di acquisizione elevate, la scansione in tempo reale dei file acquisiti costa circa il 30% della velocità di acquisizione (misurato su un’acquisizione continua con 9 telecamere: 77,7 acquisizioni/s escluse, 52,7 scansionate). `lattice array-capture --continuous` visualizza un messaggio quando rileva che Defender utilizza uno o più core della CPU durante l’acquisizione; rieseguire il programma di installazione con l’opzione selezionata.

Un’installazione silenziosa (`/SILENT` o `/VERYSILENT`) applica entrambe le opzioni a meno che non si scelga di escluderle, ad esempio con `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Firewall di Windows.** Il programma di installazione aggiunge sempre due regole in entrata per `chloros-backend.exe`, su ogni profilo di rete, e il programma di disinstallazione le rimuove: `Chloros Backend (UDP-In)`, per i flussi delle telecamere, PTP, mDNS e il flusso di dati DAQ-E, e `Chloros Backend (TCP-In)`, limitata alla sottorete locale, per un aggiornamento del firmware DAQ-E, in cui il sensore si ricollega a questo computer per acquisire l’immagine (vedere [Impostazioni dispositivo DAQ-E](#daq-e-device-settings)). L’API HTTP del backend rimane associata a `127.0.0.1`.

### Linux amd64 (.deb)

Per Ubuntu 22.04 LTS o versioni successive / workstation x86_64 basate su Debian.

> **Ubuntu 20.04 non è supportato.** L'elenco delle dipendenze del pacchetto deriva da
> ciò a cui il backend effettua effettivamente il collegamento, e ciò include `libc6 (>= 2.35)`;
> Focal distribuisce glibc 2.31. `apt` impedisce l’installazione piuttosto che lasciarla fallire in
> fase di esecuzione. L’elenco menziona anche `libgl1`, che viene caricato dall’OpenCV in dotazione e che potrebbe mancare in un’immagine
> server minimale; `apt install` lo include.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

Il pacchetto .deb installa:
- da `chloros-cli` a `/usr/bin/chloros-cli` e `chloros-backend` (avvia il backend) fino a `/usr/bin/chloros-backend`
- Il backend e la CLI compilati, decompressi, in `/usr/lib/chloros/backend/` e `/usr/lib/chloros/cli/`
- I driver delle telecamere LATTICE
- Modelli di denoising, pacchetti di calibrazione e configurazione del canale di aggiornamento
- L’SDK Python per il sistema `python3` (`import chloros_sdk`). `apt install` include anche i propri prerequisiti, `python3-numpy` e `python3-requests` (consigliati, ma non obbligatori: i programmi sopra indicati sono autonomi; `dpkg -i` li ignora, e l’output dell’installazione indica cosa manca). In un virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`, che aumenta i buffer di ricezione dei socket del kernel per i flussi della telecamera a 32 MB e li applica immediatamente (un valore già impostato su un valore superiore viene lasciato invariato). La rimozione del pacchetto comporta la rimozione del file. Vedi [Host Linux — buffer di ricezione dei socket](#linux-hosts--socket-receive-buffers).
- Due servizi systemd opzionali per il backend, entrambi non abilitati (sezione successiva)

#### Esecuzione del backend come servizio (in modalità headless)

Senza un servizio, il backend (e con esso il grandmaster PTP) funziona solo finché è in esecuzione un processo che lo ha avviato: `chloros-backend` in un terminale, uno script SDK o un comando `chloros-cli` per tutta la durata della sua esecuzione. Per un backend sempre attivo su una macchina headless, abilitare il servizio **come utente**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` esegue il backend come quell’utente. Le acquisizioni che scrive appartengono a quell’utente, il suo log dettagliato si trova nel file `~/.cache/chloros/logs`, e utilizza il suo `chloros-cli login` (`~/.chloros`), quindi è necessario prima effettuare l’accesso una volta come quell’utente. PTP non richiede i privilegi di root. Come ogni backend installato, è in ascolto solo su `127.0.0.1:5000`.

`chloros-backend.service` esegue il backend come **root**. È comunque installato in modo che una macchina su cui è abilitato continui a funzionare, ma tutto ciò che un backend root scrive appartiene a root, quindi un utente non può eliminare le proprie acquisizioni. Per passare al proprio utente:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Il log dettagliato del servizio root si trova in `/var/log/chloros` ed è leggibile dai membri del gruppo `adm` (il servizio imposta `CHLOROS_LOG_DIR`; vedi [Variabili d’ambiente](#environment-variables)). L’installazione del pacchetto visualizza il comando `enable` con il proprio nome utente già inserito, nonché l’opzione di commutazione quando il servizio root è abilitato.

**Abilitare un’unità backend.** È possibile eseguire una sola unità backend alla volta. Una seconda unità abilitata (`chloros-backend.service` oltre a `chloros-backend@<user>.service`, oppure due unità `@` di utenti diversi) si rifiuta di avviarsi e rimane su `failed` con stato di uscita 98; systemd non la riavvia. Il suo `systemctl status` ne spiega il motivo: `Another instance is already running. Exiting.`, quindi ciò che trattiene il backend (la porta 5000, oppure il file di blocco e l’utente a cui appartiene), poi `systemd will not restart <unit> (exit status 98)`. Dopo un riavvio, potrebbe essere l’una o l’altra unità a funzionare. Per passare da una all’altra, disabilita quella che non usi, come indicato sopra.**L'CLI non arresta mai il backend di un servizio.** `chloros-cli` lo utilizza così com'è, proprio come fanno la GUI e l'SDK: `process` viene eseguito su di esso, caricando il proprio progetto al posto dell'ultimo. Laddove la CLI, altrimenti, riavvierebbe il backend (`--restart`, o un `process` la cui importazione non ha trovato nulla), assegna un nome all’unità, visualizza «`sudo systemctl restart <unit>`» e termina `1`, poiché un backend arrestato in questo modo rimarrebbe inattivo. `GET /api/status` assegna all’unità il nome `service` (`null` per un backend su cui non è in esecuzione alcun servizio).**Aggiornamento e rimozione.** L’aggiornamento del pacchetto arresta tutti i servizi backend, quindi riavvia ciascuno di quelli abilitati, iniziando dalle unità `@<user>` (una avviata manualmente ma mai abilitata rimane arrestata, come dopo un riavvio). Se ne sono abilitati più di uno, ne viene eseguito solo uno e gli altri rimangono in stato di errore come sopra; l’aggiornamento visualizza i codici `Chloros: N backend units are enabled: …` e `Keep one enabled: sudo systemctl disable --now <unit>`. La rimozione li arresta e li disabilita. La purga elimina anche `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Stesso layout del .deb per amd64, con buffer di ricezione e [servizi backend](#running-the-backend-as-a-service-headless) inclusi, per Jetson Orin / Orin NX / Orin Nano su **JetPack 6** (L4T R36). Il suo PyTorch è la build CUDA di JetPack 6 per Orin, quindi l’elaborazione GPU, compreso il debayer sensibile alle texture, viene eseguita sulla GPU del Jetson. Il pacchetto non include CUDA in sé: utilizza le librerie CUDA e cuDNN installate da JetPack 6 (`sudo apt install nvidia-jetpack` su un'immagine che ne è sprovvista). Laddove non sia disponibile una GPU CUDA, l’elaborazione viene eseguita sulla CPU e la modalità «texture-aware» utilizza il debayer standard (vedere [`process`](#chloros-cli-process)).

### Autenticazione una sola volta per macchina

Ogni piattaforma richiede un accesso una tantum a Chloros+ prima che le chiamate SDK/CLI funzionino:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Le credenziali vengono memorizzate nella cache in `~/.chloros/user_session.json`.

### Avvio del backend

I comandi `lattice`, `project` e `daq` comunicano con un backend già in esecuzione; non ne avviano mai uno (vedere [Convenzioni](#conventions)). Mantenerne uno in esecuzione mentre li si utilizza:

| Piattaforma | Come |
| --- | --- |
| Windows | Aprire l’app desktop Chloros (il cui backend gestisce anche la CLI), oppure eseguire `"C:\Program Files\Chloros\backend\chloros-backend.exe"` in un terminale dedicato (nella cartella `backend` della cartella di installazione) e lasciarlo aperto. |
| Linux/Jetson | Esegui `chloros-backend` in un terminale dedicato, oppure mantienine uno sempre in esecuzione con il [servizio backend](#running-the-backend-as-a-service-headless). |

È possibile eseguire un solo backend alla volta; ogni client (l’interfaccia grafica, la CLI, gli script SDK) lo condivide. Anche i comandi principali (`process`, `login`, `status`, …) lo utilizzano e ne avviano uno quando non ce n’è nessuno in esecuzione.

**Un backend rimane in esecuzione tra un comando e l’altro**, quindi il comando successivo non ne avvia uno nuovo. Si arresta quando si esce dal sistema (`chloros-cli logout`, a meno che l’app Chloros non sia aperta o sia in corso un’esecuzione o una compilazione offline), quando si esegue `chloros-cli backend stop`, oppure automaticamente dopo 30 minuti di inattività: nessun comando, script o app, nessuna esecuzione o build offline, nessuna telecamera o sensore di luce collegato. `backend stop` rifiuta l’operazione mentre è in corso un’esecuzione o una build offline, indicandone il nome; `--force` lo arresta immediatamente e quell’esecuzione o build va persa (viene conservata una sequenza di immagini, e `array-build-video` la ricrea). Impostare `CHLOROS_BACKEND_IDLE_MIN` prima che il backend inizi a modificare i 30 minuti (`0`: mai); il [servizio backend](#running-the-backend-as-a-service-headless) non si arresta mai da solo. `chloros-cli backend status` mostra se ce n’è uno in esecuzione, cosa lo sta utilizzando ed eventuali build offline in corso. `process` apre il proprio progetto nel backend in esecuzione, a meno che l’app Chloros non sia aperta con un progetto che ha : in tal caso l’operazione viene rifiutata e occorre prima elaborare il progetto dall’app o chiuderlo. Un progetto lasciato aperto da un precedente `process` o da uno script SDK viene semplicemente sostituito.

### Verifica dell’installazione

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` termina `0` solo quando tutti e sette i controlli hanno esito positivo. L’ultimo richiede una GPU CUDA (una GPU NVIDIA, o di un Jetson Orin) e i modelli di denoising; pertanto, su un computer che ne è sprovvisto, viene visualizzato `Failed: CUDA + Denoiser` e il programma termina con `1`; in questo caso, tutto funziona tranne il debayer sensibile alle texture, che richiede esclusivamente la GPU.

> **È richiesto un abbonamento a Chloros+.**La CLI richiede un piano Chloros+ attivo.**Copper**è il livello base di Chloros+ — ogni livello a pagamento di Chloros+ include l’accesso a CLI/SDK; solo il livello gratuito**Iron** non lo include. (Mappatura ID piano: `0`=Iron/gratuito, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Esegui l'aggiornamento su [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Questo limite minimo è applicato dal backend, non solo dalla CLI: una richiesta contrassegnata da SDK/CLI senza un piano a pagamento viene rifiutata con codice `403 PLAN_UPGRADE_REQUIRED`, indipendentemente dal fatto che provenga da `chloros-cli`, dall’SDK Python o da un client HTTP sviluppato autonomamente. Un chiamante che ha effettuato il logout riceve invece il codice di errore `401 AUTH_REQUIRED`. L’accesso funziona in modalità offline per il periodo di grazia del piano (30 giorni al mese, fino alla scadenza per i piani annuali) e cessa al termine di tale periodo; `chloros-cli status` continua a funzionare in modo che il motivo sia visibile (è l’unica rotta SDK/CLI esente dal limite di livello — `GET /api/license-status`).

---

## Guida rapida in cinque minuti

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (passaggio 2) avvia un backend per la propria esecuzione quando non ce n’è nessuno in esecuzione e lo arresta al termine, quindi avviarne uno a partire dal passaggio 4. Vedi [Avviare il backend](#start-the-backend).

---

## `chloros-cli process`

Elabora una cartella di immagini attraverso l’intera pipeline di Chloros (rilevamento del bersaglio → calibrazione → vignetta → riflettanza → esportazione dell’indice).

### Sinossi

```
chloros-cli process INPUT [OPTIONS]
```

### Argomenti di posizione

| Argomento | Descrizione |
| --- | --- |
| `INPUT` | Percorso della cartella di input contenente i file `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) o `.dng`. |

### Opzioni comuni

| Flag | Impostazione predefinita | Descrizione |
| --- | --- | --- |
| `-o, --output PATH` | una nuova cartella denominata `-n` nella cartella predefinita del progetto | La cartella del progetto. Il progetto prende il nome da essa, e una cartella che contiene già un file `project.json` riceve un nome del tipo `_1` / `_2` al suo posto, invece di essere sovrascritta. La cartella predefinita del progetto è quella scelta da `set-project-folder` (o dall’interfaccia grafica), altrimenti è `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | auto (timestamp) | Nome del progetto, utilizzato solo **senza** `-o` (con `-o` il progetto assume il nome della cartella). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` utilizza un debayer neurale Chloros+ per le immagini Survey3; più lento ma di qualità superiore. È destinato esclusivamente a Survey3: Le acquisizioni LATTICE utilizzano sempre il demosaic standard, quindi per una cartella contenente acquisizioni LATTICE, `process` rifiuta `texture-aware` dopo l’importazione, segnalando un errore che indica il numero di file presenti, e chiude `1` prima di elaborare qualsiasi cosa (l’app Chloros non offre la funzione «Texture Aware» nemmeno per queste immagini). È necessaria una GPU CUDA (una GPU NVIDIA su Windows/Linux x64, la GPU dell’Orin su un Jetson): senza di essa l’esecuzione utilizza `standard` e il backend registra una riga `[DEBAYER] … needs a CUDA GPU …` che ne spiega il motivo (mostrata con `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Correzione della vignettatura. |
| `--reflectance / --no-reflectance` | `--reflectance` | Calibrazione della riflettanza (utilizza il target a pannello se trovato, calibrazione NIST per serie per LATTICE). Per LATTICE multispettrale, questo funge anche da **prodotto** della riflettanza — vedi [Opzioni di esportazione per prodotto](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | disattivato | Applica le correzioni GNSS PPK dai file sidecar. |
| `--exposure-pin-1 CAMERA_MODEL` | nessuno | Il modello di fotocamera collegato al pin di esposizione 1, per la sincronizzazione temporale PPK. |
| `--exposure-pin-2 CAMERA_MODEL` | nessuno | Il modello della fotocamera collegato al pin di esposizione 2, per la sincronizzazione temporale PPK. |
| `--recal-interval SECONDS` | `0` | Intervallo minimo di ricalibrazione, in secondi (l’*Intervallo minimo di ricalibrazione* del progetto). |
| `--timezone-offset HOURS` | `0` | Differenza di fuso orario del sensore di luce rispetto all’UTC, in ore (il *Differenza di fuso orario del sensore di luce* del progetto). |
| `--format FORMAT` | `TIFF (16-bit)` | Uno tra `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros non aumenta mai la profondità di bit di un'immagine: sulle immagini interamente a 8 bit, un formato con profondità maggiore viene salvato come `TIFF (8-bit)` e la procedura lo indica una volta. In un progetto che combina acquisizioni a 8 bit e a 12/16 bit, ciascuna acquisizione viene esportata con la propria profondità di bit o con una inferiore. La luminosità è sempre a 32 bit in virgola mobile. |
| `--indices NAME [NAME ...]` | nessuno | Indici di vegetazione per nome predefinito (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), senza distinzione tra maiuscole e minuscole. Ciascuno viene aggiunto al progetto in base alle bande della propria fotocamera, con l’inclusione di un nuovo progetto. Un nome che non corrisponde a un’impostazione predefinita, o che le bande della fotocamera non sono in grado di generare (ad esempio, «`NDRE`» richiede RE e NIR, che nessuna singola telecamera fornisce), viene segnalato in un avviso e non riceve alcuna immagine indice. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forza il punto di ingresso della pipeline per i file TIFF LATTICE (i file .raw di Survey3 non sono interessati). Inoltre, la via di fuga che consente di elaborare una cattura **senza file raw** — vedi [Come si presenta una cartella delle catture](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | on | Emette il prodotto debayered lineare (`Debayered_Images`). Vedi [Opzioni di esportazione per singolo prodotto](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | attivo | Emette l’anteprima di visualizzazione (`Preview_Images`): RGB = bilanciamento del bianco (illuminante DAQ se disponibile, altrimenti “gray-world”) + gamma; multispec = estensione a falsi colori. |
| `--radiance / --no-radiance` | attivo | Emette radianza in float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Riferimento per il prodotto di riflettanza LATTICE: `auto` = il target all’interno del fotogramma che ha superato il controllo di qualità (QA) è il riferimento assoluto (un target con flag QA compete con il DAQ sul punteggio misurato), fallback DAQ downwelling (ρ = π·L/E); `target` = rigoroso (nessuna sostituzione DAQ); `daq` = DAQ autorevole. Vedi [Opzioni di esportazione per singolo prodotto](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | nessuno | Directory delle scansioni di riflettanza target **misurate** per unità (`<serial>.csv`); in caso di mancata rilevazione, ricorre agli spettri nominali T3/T4P. |
| `--array-alignment / --no-array-alignment` | attivo | Matrici LATTICE: applica l’allineamento da modulo a modulo indicato nel file XMP `Chloros:Alignment*` di ciascuna acquisizione a ogni prodotto elaborato (debayering / anteprima / radianza / riflettanza / indice). Nessuna operazione per le immagini prive dei tag. |
| `--array-alignment-crop / --no-array-alignment-crop` | ritaglio | Ritaglia le esportazioni allineate alla regione di sovrapposizione comune dell’array in modo che tutti i moduli condividano un’unica impronta; `--no-…` mantiene l’intera area del sensore (riempimento nero al di fuori della sorgente). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Ricampionamento per la distorsione da allineamento. `nearest` conserva i valori DN esatti della sorgente (senza mescolanza inter-pixel dei valori radiometrici). |

### Opzioni di rilevamento del bersaglio

| Flag | Descrizione |
| --- | --- |
| `--min-target-size PIXELS` | Dimensione minima in pixel del bersaglio di calibrazione per il rilevatore. |
| `--target-clustering 0-100` | Percentuale di raggruppamento dei bersagli (impostazione predefinita `60`). |
| `--target / --targets` | Ricerca dei bersagli di calibrazione solo in una sottocartella `target` o `targets` del file di input (più veloce). |

### Esempi

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Opzioni di esportazione per singolo prodotto (LATTICE multispettrale)

L'elaborazione LATTICE si ramifica in **ogni prodotto applicabile in un unico passaggio**. Le quattro opzioni per tipo — `--debayered`, `--preview`, `--radiance`, `--reflectance` — sono tutti**ATTIVATI per impostazione predefinita**; utilizzare il modulo `--no-<type>` per disattivarne uno. Le telecamere master RGB emettono sempre e solo dati debayered + anteprima (nessuna radianza/riflettanza per banda), quindi `--radiance`/`--reflectance` non hanno alcun effetto su di esse. I comandi di attivazione/disattivazione vengono ignorati per `.raw` di Survey3 (che segue il percorso standard di riflettanza/target). *(Il vecchio flag `--radiometric-output {reflectance,radiance,sensor-response}` è stato **rimosso** e sostituito da questi comandi; non esiste più il livello `sensor-response`.)*

| Prodotto | Output | È necessario il downwelling DAQ? |
| --- | --- | --- |
| `--debayered` | Demosaicatura lineare (`Debayered_Images`): lo stesso file salvato da una cattura al proprio livello `debayered` — l’offset del dark (DSNU) del pacchetto di calibrazione applicato al raw, seguito dal demosaic sensibile ai bordi di OpenCV; nessuna LUT di linearità, flat field o unmix. | No. |
| `--preview` | Anteprima sullo schermo (`Preview_Images`): RGB = bilanciamento del bianco + gamma; multispettrale = espansione in falsi colori. | No. |
| `--radiance` | float32 W/m²/sr/nm dalla catena radiometrica completa (`Radiance_Images`). | N. |
| `--reflectance` | uint16 riflettanza ρ (`32768` = 1,0), Compatibile con Pix4D. | **Sì**, a meno che non sia ancorata da un target presente nell’inquadratura che abbia superato il controllo di qualità (vedi sotto). |

`--reflectance-source` seleziona il riferimento di riflettanza:**`auto`**(impostazione predefinita) rende un target presente nell’inquadratura e che ha superato il controllo di qualità (QA) il**riferimento assoluto**— le catene di linee empiriche ancorate al bersaglio vengono sottoposte a valutazione incrociata su pannelli tenuti da parte e il risultato vincente misurato viene applicato, banda per banda: una banda che il DAQ non può calibrare, o la cui adattamento della pendenza DAQ fallisce, assume la linea del bersaglio stesso (`ReflectanceElBandMethods` indica quale) — un target che supera il controllo di qualità con un flag (un pannello escluso, una banda fuori uso) compete anche con la divisione del DAQ su quel punteggio misurato, e un frame a cui perde viene contrassegnato con `ReflectanceChain=daq` — si ricorre alla soglia di downwelling del DAQ (ρ = π·L/E) quando non è presente alcun bersaglio o il controllo di qualità fallisce;**`target`**è rigoroso (nessuna sostituzione da parte del DAQ);**`daq`**opta per il comportamento autoritativo del DAQ. La geometria del bersaglio (ArUco / ROI fissa / striscia) deriva dalla configurazione del bersaglio del progetto; `--target-reflectance-dir DIR` conserva le scansioni**misurate** per unità (`<serial>.csv`) individuate tramite il numero di serie/QR dell’unità di destinazione, con gli spettri T3/T4P nominali come valore di riserva.

Il percorso di riflettanza del DAQ risolve automaticamente il **flusso discendente con timestamp corrispondente**da un**`.daq`**registrato (DAQ-U/M/E)**o da un file `.csv` nativo DAQ-M**presente insieme alle immagini. Se un pacchetto di calibrazione per singola telecamera o DAQ non è memorizzato nella cache locale, la pipeline**lo recupera automaticamente da AWS** al primo utilizzo (richiede una connessione a Internet una sola volta; viene memorizzato nella cache sotto `~/.chloros/`).**A chi appartiene la lettura.**Un `.daq` salvato da un’acquisizione rappresenta la lettura della o delle telecamere che descrive — indicate in un file sidecar `<file>.daq.cameras.json` accanto ad esso (`lattice capture`, `multi-capture`, `array-capture` e l’interfaccia grafica; una raffica grezza indica la lettura di ciascun membro nel proprio manifesto) — e `process` la abbina**solo alle immagini provenienti da quelle telecamere**. Una telecamera autonoma senza una propria lettura, quindi, non utilizza più il DAQ di un’altra telecamera; è necessario associarle un DAQ o registrarne uno. La registrazione propria di un DAQ (`daq pool-record`, o una registrazione avviata nell’interfaccia grafica), o qualsiasi `.daq` senza quel sidecar (comprese le acquisizioni più vecchie), serve comunque tutte le telecamere.

#### Lettura dei pixel di riflettanza (Pix4D / Metashape / script personalizzati)

La riflettanza è memorizzata come DN intero, e **il valore DN che corrisponde a ρ = 1,0 dipende dalla telecamera sorgente**:

| Sorgente | ρ = 1,0 corrisponde a | Come identificarlo |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margine fino a ρ 2,0) | Il file riporta il tag XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (limitato a ρ 1,0) | Nessun tag XMP `Chloros:*` — tale assenza *è* il segnale. |

**Leggi `Chloros:PixelScale` e dividi per quel valore** invece di ipotizzare una costante. Il tag è definito nel dominio uint16, quindi rimane `32768` in tutti i formati di output che effettuano un ridimensionamento — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` e `TIFF (32-bit, Percent)` sono tutti autodescrittivi (normalizzare prima il tipo di dati memorizzato riportandolo a uint16: ×257 da 8 bit, ×65535 da float).

> **I file provenienti da versioni precedenti potrebbero non contenere alcuna scala.** Un file TIFF di riflettanza a 8 bit viene riscalato e contrassegnato come `Chloros:PixelScale`, come gli altri. Le versioni precedenti alla 1.3.0 lo *ritagliavano* invece a 0..255, quindi ogni valore superiore a ρ≈0,008 veniva appiattito a 255, e omettevano sia la tupla `Chloros:PixelScale` che quella `MicaSense:RadiometricCalibration`. **Se il tag è assente in un file di riflettanza LATTICE, non dare per scontata l’esistenza di una scala — rielaboralo** piuttosto che dividere pixel che non sono mai stati divisibili.

#### Lettura dei pixel indice (NDVI e gli altri indici di vegetazione)

**Un raster indice intero rappresenta un'estensione del contrasto per immagine, non una mappa −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` e `JPG (8-bit)` — compreso il file TIFF a 8 bit in cui viene salvata un’acquisizione a 8 bit — memorizzano

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

dove `IndexMin` / `IndexMax` sono **i valori di indice finiti più basso e più alto propri di quel fotogramma**. DN 0 non è −1 e il fondo scala non è +1.

I timbri Chloros contrassegnano entrambi gli estremi in modo che la mappatura rimanga invertibile:

| Tag | Significato |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — la convenzione riportata di seguito |
| `Chloros:IndexMin` | il valore dell’indice memorizzato come DN 0 |
| `Chloros:IndexMax` | il valore dell’indice memorizzato come fondo scala |

Leggerli e invertirli, normalizzando il DN memorizzato a un valore compreso tra 0 e 1 in base al massimo del tipo di dati (dtype) memorizzato, in modo che un'unica espressione copra ogni formato intero:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

La lettura non richiede `-config pix4d.config` — exiftool legge queste proprietà XMP autonomamente. La configurazione è necessaria solo per *scriverle*.

> **`IndexMin == IndexMax`** significa che ogni pixel di quel fotogramma conteneva lo stesso valore di indice. Il raster è riempito a metà scala e non c’è nulla per cui dividere — si prende il valore estremo.

> **L’assenza di tag `Chloros:Index*` indica che non è stata specificata alcuna mappatura.** Un’esportazione LUT è un rendering colorato i cui pixel sono colori sfumati abbinati alla finestra di soglia della LUT, e un fotogramma senza pixel con indice finito non ha un intervallo da specificare. Non dare per scontata alcuna mappatura in entrambi i casi — vale la stessa regola di un `PixelScale` mancante.**Poiché l’allungamento è per singola immagine, due fotogrammi di uno stesso volo non sono comparabili in termini di DN**— ciascuno ha i propri punti estremi, quindi lo stesso NDVI corrisponde a valori DN diversi in immagini diverse. Per analisi quantitative su un intero volo (statistiche zonali, soglie fisse, serie temporali), esportare**`TIFF (32-bit, Percent)`**: i raster di indice vi vengono scritti come float32**i valori di indice stessi**, senza necessità di riscalatura né di tag. Nonostante il nome del formato, i prodotti di indice non sono ×100 — tale scalatura si applica solo alla riflettanza con tipo di dati intero.

#### Maschera di limitazione della separazione spettrale (su quali pixel è stata applicata la limitazione a una banda)

La separazione spettrale 3×3 può rendere negativa una banda — laddove la luce contenga a malapena quella banda, o laddove la scena ricada al di fuori di quanto descritto dalla matrice di separazione della fotocamera. Non si tratta di un valore reale, quindi l’esportazione lo fissa a 0 — e la visualizzazione in tempo reale mostra lo stesso 0. Una banda può essere limitata su pochi pixel o su quasi tutti; la maschera registra entrambe le situazioni. `Chloros:Band{R,G,B}UnmixClampFraction` indica la percentuale del fotogramma in cui ciò si è verificato; la maschera di clipping indica **quali pixel**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— in una cartella `clamp_masks/`**accanto** alla cartella del prodotto, mai al suo interno (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; un’esportazione da `process`: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — per ogni prodotto relativo a radianza, riflettanza e indice in cui almeno un pixel è stato clippato: sia le acquisizioni che le esportazioni (e i TIFF per fotogramma). `uint8` a banda singola, sulla griglia di pixel propria del prodotto (compresi orientamento, ROI, binning e allineamento dell’array). Ogni maschera indica la fotocamera del proprio prodotto (`Make` / `Model` / numero di serie), l’ora di acquisizione e i tag dell’obiettivo, ed elenca le bande a cui corrispondono i suoi bit (`XMP-Camera:BandName`, bit 0 per primo); i suoi `ImageDescription` e `Software` la identificano come maschera di limitazione e non contiene alcuna calibrazione.
- **Bit**: bit 0 = la banda nel canale R del prodotto (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Un bit impostato indica che il valore di quel pixel in quella banda è stato limitato a 0 — considerarlo, così come qualsiasi indice da esso derivato, come a bassa affidabilità.
- **`Chloros:UnmixClampMask`** è impresso su ogni prodotto su cui è stata eseguita la separazione: il percorso della maschera relativo alla cartella del prodotto stesso (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, sempre separato da `/` — unirlo alla cartella dell’immagine per aprire la maschera), oppure `none` quando nessun pixel è stato limitato. L’assenza di tag indica che il prodotto non è mai stato sottoposto all’unmix (telecamere RGB o monocromatiche, livelli raw/debayered).
- `lattice capture` / `array-capture` stampano il percorso della maschera sotto il file salvato (`clamp mask: …`), e l’elenco dei risultati di acquisizione lo riporta come `clamp_mask` del fotogramma.
- `lattice index --live --save-multiband` scrive una maschera per ogni telecamera sorgente per lo stack multibanda — vedi [`lattice index` — maschere dello stack multibanda](#lattice-index--multiband-stack-masks). Un video non ha maschere: `array-build-video` riporta invece le frazioni di clamp di ciascun fotogramma (vedere [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Poiché nessuna cartella del prodotto contiene una maschera, a qualsiasi livello di profondità, una cartella del prodotto (`radiance/`, `reflectance/`, una cartella `*_Images/` di un’esportazione `process`) può essere consegnata a Pix4D, Metashape o ad altri software di fotogrammetria così com’è — sottocartelle incluse — e accetta solo immagini. Consegnate la cartella del prodotto, non quella sovrastante (che contiene anche `clamp_masks/` e ogni altro tipo di esportazione). L’importazione di Chloros ignora le maschere. Lettura di una maschera: `(mask & 1) != 0` indica che «qui il canale R è stato limitato», `& 2` G, `& 4` B.

#### Dati EXIF trasferiti nell’esportazione

`process` copia il **blocco GPS e il relativo ExifIFD** dell’acquisizione di origine su ogni prodotto, quindi un’
esportazione include `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` e
`CameraSerialNumber` insieme al georeferenziazione.

**`FocalLength` non è facoltativo per la fotogrammetria.** Pix4D calcola la distanza campionaria da terra (GSD) in base alla
lunghezza focale più l’altitudine; in assenza del tag, ricorre a una scala estremamente errata. In un
volo su un aranceto composto da 49 acquisizioni, il tag mancante ha trasformato un sito di 411 m × 160 m in uno ricostruito
di 47,8 km × 13 km — un'ortofoto da 455 MP composta prevalentemente da dati mancanti, che è stata quindi interpretata come un problema di tiling e
un problema BigTIFF prima che qualcuno controllasse la GSD. Se la vostra ortofoto risulta in una scala non plausibile, eseguite prima `exiftool -FocalLength` sul prodotto esportato.

La copia **non** è deliberatamente `-all:all`: i tag strutturali di IFD0'i tag strutturali di IFD0 compromettono l’output di LATTICE quando
vengono copiati, mentre `ExifImageWidth` / `ExifImageHeight` sono esclusi perché descrivono l’
acquisizione *di origine* — un’esportazione che fosse stata ridimensionata in qualsiasi momento presenterebbe altrimenti dimensioni
in contraddizione con il proprio raster. L’XMP viene scritto direttamente anziché copiato, poiché ExifTool
elimina i tag XMP della stessa invocazione quando il blocco XMP viene copiato (il che comporterebbe la perdita dei tag di calibrazione MAPIR).

### Dove vengono salvati i risultati

I prodotti vengono salvati **nella cartella del progetto, raggruppati per fotocamera e poi per formato di file**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

La cartella della fotocamera è `LATT-<sensor>-<lens>-F<filter>` per LATTICE (corrispondente all’EXIF dell’acquisizione
`Model`) e `<model>_<filter>` per Survey3 — due fotocamere che condividono sensore e filtro ma differiscono
nell’obiettivo mantengono alberi separati, poiché vignettatura, campo visivo e distorsione differiscono. La cartella del formato
cartella segue questo schema: `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, oppure `tiff32` per
`TIFF (32-bit, Percent)`.

> **Ogni prodotto esportato mantiene il nome del file ORIGINALE.** Un’esportazione di radianza di
> `capture_…_raw.tif` si chiama comunque `capture_…_raw.tif` — si trova semplicemente in
> `tiff32/Radiance_Images/`. **È la cartella a identificare il prodotto, non il nome del file**, quindi la ricerca con il carattere jolly
> per `*radiance*.tif` non trova nulla; cercare invece la corrispondenza nella directory.

### Registrazioni del sensore di luce — calibrate `.daq` + `.csv`

`process` gestisce anche le registrazioni `.daq` presenti nella cartella di input e **non**
ha bisogno di alcuna immagine per farlo: un DAQ-U / DAQ-M / DAQ-E fatto volare da solo costituisce un’acquisizione completa
e una cartella contenente solo file `.daq` è un input valido.

È possibile registrare un DAQ **senza** la sua calibrazione — ed è proprio ciò che fanno per impostazione predefinita i registratori pubblici
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) per impostazione predefinita: scrivono i conteggi grezzi dei sensori e contrassegnano il file in modo che
Chloros recuperi la calibrazione di fabbrica di quel sensore **tramite la porta seriale** (prima dalla cache locale,
poi dal cloud MAPIR) e la applica. `process` riscrive il risultato:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

L’`.csv` contiene una riga per ogni lettura: timestamp UTC, tempo di integrazione, potenza totale,
lux fotopico/scotopico, PPFD (e la sua suddivisione in blu/verde/rosso), lunghezza d’onda di picco, quindi lo
spettro completo sulla griglia di lunghezze d’onda propria del sensore. Il file `.daq` viene reimportato senza essere
calibrato una seconda volta.

`<name>` è il nome del file della registrazione stessa. Due registrazioni in cartelle diverse possono condividere
un unico valore — le letture per singola acquisizione (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) salvate nello
stesso millisecondo da due telecamere — e ciascuna ottiene comunque i propri prodotti: la prima mantiene
`<name>`, la successiva aggiunge le cartelle che la distinguono **dopo** il nome
(`<name>__<capture>_daq_calibrated.daq`), quindi la data e l’ora della registrazione rimangono sempre all’inizio.

In caso di esito positivo, l’esecuzione riporta `Light-sensor products written: N (calibrated .daq + .csv)`.
La parte tra parentesi descrive ciò che è stato effettivamente scritto, quindi si legge
`(RAW COUNTS — this sensor has no calibration bundle)` per un sensore senza bundle e
`(N calibrated, M raw counts)` per una cartella che contiene entrambi. Le intestazioni proprie del backend
`[DAQ-EXPORT]` e `[RUN-SUMMARY]` derivano la loro formulazione allo stesso modo — nessuna delle
tre può definire "calibrata" un'esportazione grezza.

Una registrazione DAQ-U / DAQ-M / DAQ-E il cui pacchetto di calibrazione non può essere recuperato — perché si è
offline o perché quel sensore non ha alcuna calibrazione in archivio — viene **saltata con una motivazione** su una
riga `[DAQ-EXPORT]`, e non viene mai salvata come file “calibrato” contenente conteggi grezzi.
Connettiti a Internet ed esegui nuovamente l’operazione. Il motivo è quello effettivamente
stabilito dal lettore per quel file (schema illeggibile, assenza di pacchetto, errore di scrittura), e il riepilogo dell’esecuzione
elenca motivi **distinti** — venti file saltati per un’unica causa vengono riportati come un’unica
causa, non come venti ripetizioni della stessa. La CLI indica ogni registrazione saltata alla fine dell’
esecuzione, con il relativo motivo:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Una cartella di registrazioni che non ne esporta **nessuna** fallisce: vedi
[Un'esecuzione che non scrive immagini fallisce](#a-run-that-writes-no-images-fails).

Una registrazione il cui nome termina con `_calibrated` o `_raw` (`field_raw.daq`) **non
importata**: questi sono i nomi che `process` assegna ai propri prodotti, e un prodotto letto
come registrazione verrebbe conteggiato due volte. L’CLI lo segnala e indica il nome del file —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— invece di segnalare una cartella vuota. Rinominare tale registrazione prima di elaborarla.

#### Le registrazioni DAQ-A vengono esportate come conteggi grezzi

La famiglia **DAQ-A** è antecedente al sistema dei pacchetti per numero di serie e non ha alcun pacchetto di calibrazione
da recuperare — viene invece calibrata sul campo rispetto a un bersaglio di riflettanza, motivo per cui
non ne ha mai avuto bisogno. Rifiutare tali registrazioni avrebbe impedito loro di ottenere i propri
dati, quindi vengono esportate con un **nome diverso**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Un nome file diverso anziché un flag all’interno del file, poiché l’informazione deve rimanere intatta
anche se inviata via e-mail come semplice nome. L’intestazione `.csv` riporta
`raw spectral sensor counts (NOT irradiance)` e avverte che i valori sono comparabili
**all’interno** del file — che è esattamente lo scopo per cui vengono utilizzati nella calibrazione basata su target — e
non tra sensori diversi. Le colonne fotometriche dipendenti dalla potenza (potenza totale, lux fotopici e
scotopici, PPFD) vengono registrate come **NULL** anziché integrate dai conteggi, e il riepilogo dell’esecuzione
riporta `RAW COUNTS`, quindi "esportato" in un log non può essere interpretato come irradianza.

Le registrazioni legacy **v1.01 / v1.02** (scritte da un DAQ-A-SD) non riportano alcuna epoca per ogni lettura,
ma solo l’ora di scrittura del file. Il programma di corrispondenza immagine↔irraggiamento discendente continua a rifiutarle — abbinare un
fotogramma a un’ora di scrittura comporterebbe un errore non visibile — ma l’esportatore le legge e
il file CSV riporta `clock=daq_created_on`, quindi il prodotto indica su quale orologio si trova.

### Note

- `process` rileva automaticamente se la cartella è di tipo Survey3, LATTICE o mista.
- Lo stato di avanzamento viene trasmesso tramite Server-Sent Events; la CLI mostra in tempo reale lo stato di avanzamento per ogni thread (Rilevamento, Analisi, Elaborazione, Esportazione).
- Per Linux/Jetson, la CLI controlla lo swap e potrebbe visualizzare un avviso prima di elaborare cartelle di grandi dimensioni. Il debayer sensibile alle texture applica inoltre automaticamente un limite di frequenza della GPU sui Jetson a basso consumo (Nano, Orin Nano).
- In caso di esito positivo, l’esecuzione riporta il numero di prodotti immagine scritti (`Image products written: N`, con `(plus M clamp masks)` quando il processo di separazione dei colori ha limitato un pixel).
- **`process` condivide in modo sicuro un backend in esecuzione.** Riavvia automaticamente un backend già in esecuzione solo quando tale backend è inattivo con un progetto precedente caricato e non gestisce telecamere, array o DAQ. Un backend che ne contenga uno qualsiasi (la GUI che trasmette in streaming un array, uno script SDK, una registrazione DAQ) viene utilizzato così com’è, senza mai essere riavviato, quindi `process` può funzionare parallelamente a un array attivo. Se il backend segnala un’esecuzione in corso, `process` attende fino a 15 s che questa si concluda (un’esecuzione appena completata impiega circa un secondo per essere cancellata), quindi restituisce un errore con codice `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. Il flag globale `--restart` forza comunque l’avvio di un nuovo backend e termina qualsiasi sessione relativa a telecamere, array o DAQ su di esso. Un backend eseguito come servizio systemd di Linux non viene mai riavviato: `process` lo utilizza e `--restart` stampa invece `sudo systemctl restart <unit>` (vedi [Esecuzione del backend come servizio](#running-the-backend-as-a-service-headless)). Lo stesso vale per il backend proprio dell’app Chloros: con l’app aperta, `process` lo utilizza, caricando il proprio progetto al posto di quello dell’app (un backend avviato al suo posto bloccherebbe l’accesso all’app aperta).
- **Un'esecuzione in corso mantiene il proprio progetto.** Mentre è in corso un'esecuzione di elaborazione — avviata dall’app, da `process`, da uno script SDK o da `chloros-mcp` — il backend rifiuta di creare, aprire, importare, aggiungere o rimuovere file, spostare, eliminare o modificare le impostazioni di elaborazione di un progetto, con codice di errore HTTP 409 `PROCESSING_IN_PROGRESS` e `A processing run is in progress on project '<name>', so ... now is refused`; un `process` che lo riceve esce da `1`. Le impostazioni di visualizzazione (dimensione delle miniature) vengono comunque salvate. La modalità di elaborazione e le impostazioni di riflettanza target del progetto vengono rifiutate allo stesso modo (un'esecuzione mantiene la modalità con cui è stata avviata). Un'acquisizione salvata nel progetto durante l'esecuzione (dall'app, dalla CLI, dall'SDK o da `chloros-mcp`) viene salvata come di consueto, ma viene aggiunta all'elenco dei file del progetto solo al termine dell'esecuzione; come indicato nella relativa risposta (`project_add`), così come una registrazione del sensore di luce interrotta all’interno del progetto. In precedenza, un secondo client poteva passare al progetto aperto durante l’esecuzione, e l’esecuzione in corso veniva esportata nell’altro progetto.
- **Ctrl+C interrompe l’esecuzione nel backend, non solo nell’CLI.** La CLI chiede al backend di interrompere l’elaborazione, visualizza `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` e attende fino a 120 s per `Processing stopped.`, in modo che `chloros-cli process … && next` non avvii `next` su un backend ancora occupato con l’esecuzione. Un secondo Ctrl+C termina l’attesa. `process` quindi esce da `130`, come fa qualsiasi comando interrotto.

#### Un'esecuzione che non scrive immagini fallisce

Se sono stati richiesti dei prodotti e l’esecuzione non ne ha scritti **alcuni** — solo `project.json` e
`calibration_data.json` — `process` lo considera un errore: stampa
`Processing finished but wrote no image products.` e **termina con un valore diverso da zero**, in modo che uno script possa
rilevarlo. Il messaggio indica la cartella del progetto e le cause più comuni:

- la cartella di input non è stata riconosciuta come acquisizione (controllare il layout e `--input-level`), oppure
- tutti i prodotti richiesti sono stati ignorati in quanto non applicabili a quelle telecamere (ad es. richiesta di
  radianza/riflettanza da telecamere solo RGB).

Eseguire nuovamente con `--verbose` e controllare il log del backend per le righe `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
che spiegano le omissioni per singola telecamera che altrimenti non compaiono nell’output della CLI.

Un'esecuzione deliberata solo con metadati — tutti i prodotti disattivati e nessun `--indices` — è comunque un
**successo**, poiché in questo caso un'immagine di output vuota è il risultato corretto.

#### Un'esecuzione le cui esportazioni non è stato possibile verificare fallisce

Al termine di un'esecuzione, `process` controlla i tag dei file esportati con ExifTool. Se ExifTool non è riuscito a funzionare,
visualizza `Verifica esportazione FALLITA: ExifTool non è riuscito a funzionare, quindi i tag di N su M file esportati
non sono stati controllati (vedere le righe [EXIFTOOL] nel log del backend)` and `I prodotti vengono scritti e rimangono
al loro posto; rieseguire l’operazione per verificarli.`, does not print `Elaborazione completata!`, e **termina con un valore diverso da zero**. I
prodotti vengono conservati.

#### Quali acquisizioni vengono incluse in un'esecuzione

Le acquisizioni terminate prima dell’esecuzione di `process` vengono incluse. Quelle salvate durante l’esecuzione no; `process`
segnala quante sono state escluse e, eseguendolo nuovamente, le include.

#### Un'esecuzione in cui è andata persa un'immagine fallisce

Se un'immagine ha dato esito negativo durante un'esecuzione, `process` visualizza `N image(s) failed and were not exported` con la
causa più comune, non visualizza `Processing complete!` e **termina con un codice di uscita diverso da zero**. Gli altri prodotti vengono
scritti e rimangono al loro posto, e il log del backend riporta ogni errore.

Lo stesso vale per un **ciclo dedicato esclusivamente al sensore di luce**: una cartella di registrazioni `.daq` non contiene immagini da esportare
per definizione, e l’esecuzione viene valutata in base ai file calibrati `.daq` / `.csv` che ha invece scritto.
Se non ne ha scritto **nessuno** — tutte le registrazioni sono state saltate — visualizza
`Processing finished but wrote no light-sensor products.`, assegna a ciascuna registrazione un nome che indica il
motivo per cui è stata saltata e **termina con un codice di uscita diverso da zero**.

---

## `chloros-cli login`

Autentica questa macchina con un account cloud Chloros+. Le credenziali vengono memorizzate in modo sicuro nella cache in `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Esempi

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (eliminando o duplicando parti della password). In caso di errore 401, l’CLI effettua automaticamente un nuovo tentativo con `$$`-aggiunto, poi con metà della password senza duplicati; se il nuovo tentativo va a buon fine, effettua l’accesso e visualizza la sintassi corretta con le virgolette singole da utilizzare la prossima volta.

> **Utilizzo in modalità headless/tramite script: l’assenza di una sessione memorizzata nella cache comporta la visualizzazione di una richiesta di accesso.** Un comando principale (`process`, `status`, `export-status`, `time-sync`, …) eseguito senza una sessione memorizzata in cache richiede `Email:` / `Password:` su stdin prima di procedere. Se stdin è chiuso (cron, un’unità systemd), il processo fallisce immediatamente con l’errore `login cancelled`; se stdin è un terminale aperto o una pipe, il processo attende l’input. Eseguire `chloros-cli login EMAIL PASSWORD` una volta per ogni macchina prima di pianificare operazioni in modalità headless.

---

## `chloros-cli logout`

Cancella la sessione memorizzata nella cache e impone un nuovo accesso alla chiamata successiva.

La disconnessione mentre è in corso un'elaborazione non la interrompe: l'elaborazione termina nella modalità in cui è stata avviata, mentre quella successiva utilizza l'elaborazione standard. Lo indica `logout` (`The processing run in progress finishes in <mode> mode; the next run uses standard`), così come l'app quando ci si disconnette da lì.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Mostra il livello di licenza corrente (Iron/Copper/Bronze/Silver/Gold), l’utente autenticato e il numero di associazioni del dispositivo.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Interroga lo stato di avanzamento in tempo reale dell’esportazione Thread-4. È sicuro da chiamare **durante** l’esecuzione di `process` da un’altra shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Imposta la lingua di visualizzazione della CLI (38 lingue supportate, incluse CJK, RTL e Indic). Passa automaticamente all’inglese sulle console legacy che non sono in grado di visualizzare lo script.

```
chloros-cli language [LANG_CODE] [--list]
```

### Esempi

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Comandi relativi alla cartella del progetto

Questi comandi gestiscono il percorso predefinito della cartella del progetto (condiviso con l’interfaccia grafica).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` visualizza la cartella in cui vengono salvati i nuovi progetti. Se la cartella configurata non esiste più (è stata eliminata, rinominata o si trova su un'unità non collegata), la CLI, come l’interfaccia grafica, ricorre alla cartella predefinita: la cartella predefinita viene visualizzata su stdout, un avviso che indica la cartella mancante viene inviato a stderr e il codice di uscita è `0`.

---

## `chloros-cli update`

Solo Linux/Jetson. Verifica la presenza di `version_url` rispetto a `/etc/chloros/update.conf` e, se esiste una versione più recente, scarica la versione corrispondente `.deb` e la installa immediatamente con `sudo dpkg -i` (senza richiesta di conferma; `sudo` potrebbe richiedere la password). Su un Jetson l’aggiornamento viene offerto solo ai sistemi con JetPack 6. Su Windows il comando non esegue alcuna operazione e segnala `up to date`: aggiornare con un nuovo programma di installazione.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Su Linux/Jetson la CLI esegue anche un **controllo automatico degli aggiornamenti ad ogni avvio** (non bloccante, non ritarda mai l’esecuzione del comando): legge `/etc/chloros/update.conf`, memorizza il risultato nella cache per 1 ora in `~/.chloros/update_cache.json` e visualizza `Update available: vX.Y.Z / Run: chloros-cli update` quando è disponibile una versione più recente. Viene saltato silenziosamente in caso di errore e su Windows; `CHLOROS_DISABLE_UPDATE_CHECK=1` lo disattiva. A causa della cache di 1 ora, una versione pubblicata entro quell’ora può comunque risultare aggiornata.

---

## `chloros-cli selftest`

Esegue un test di funzionamento in 7 fasi: versione, disponibilità delle porte, avvio del backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), presenza del modello di denoising, prontezza di CUDA+denoiser. Utilizza un backend in esecuzione oppure ne avvia uno per il test e lo arresta in seguito. Visualizza `N/7 checks passed` e i controlli falliti, e termina `0` solo quando tutti e sette i controlli hanno esito positivo; il controllo CUDA+denoiser fallisce su un computer privo di una GPU CUDA (vedere [Verifica dell’installazione](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Stato e controllo del grandmaster PTP. L’host Chloros esegue il grandmaster PTP; le telecamere LATTICE e le unità DAQ-E sono subordinate ad esso per i timestamp tra dispositivi (circa 1 ms tra i dispositivi). Il PTP allinea i **timestamp**: le telecamere di un array scattano contemporaneamente perché vengono attivate dal trigger hardware M8 (con un intervallo inferiore a 100 µs tra i moduli), non grazie al PTP. `chloros-cli time-sync` senza sottocomandi esegue `status`.

| Sottocomando | Descrizione |
| --- | --- |
| `status` | Mostra lo stato del grandmaster, le priorità BMCA, l’identità dell’orologio e se i suoi pacchetti raggiungono la rete: `health` è `OK` (`STOPPED` quando il grandmaster non è in esecuzione), oppure `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` con un `health_detail`, e quindi viene stampata una riga `WARNING:` su stderr (stdout rimane in formato JSON). `sync_sent` / `announce_sent` contano solo i messaggi che sono stati inviati; `*_send_errors` e `last_send_error` contano quelli che non lo sono stati. Dopo che una connessione della telecamera si interrompe e viene ripristinata, il grandmaster ricostruisce autonomamente i propri socket (`rebinds`, `last_rebind`). |
| `peers` | Elenco degli slave rilevati tramite Delay_Req (telecamere + sensori DAQ-E). |
| `cameras` | Stato di integrità PTP per singola telecamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Riavvia il processo grandmaster. |
| `set-priority --priority1 N --priority2 N` | Ignora le priorità BMCA. |

Il grandmaster si annuncia sulle schede di rete delle telecamere (qualsiasi scheda con un indirizzo `169.254.x.x` incluso). Se nessuna è attiva, ricorre alla scheda della rotta predefinita solo se questa è cablata, mai al Wi-Fi o a una scheda virtuale (Tailscale, WSL), poiché nessuna telecamera o DAQ-E si trova dietro una di esse. Non avendo nulla da annunciare, funziona senza socket (`health` `NO_SOCKETS`), e la connessione della telecamera o del DAQ successivo lo ricostruisce sulla scheda di rete attiva in quel momento; `time-sync restart` lo fa immediatamente.

Il grandmaster gira in un processo a sé stante, accanto al backend (`own_process` / `process_pid` in `time-sync status`), con priorità elevata su Windows (`process_priority`), in modo che un backend occupato (un'elaborazione in corso, un'esportazione, diversi comandi contemporaneamente) non possa ritardare i tempi che registra; in precedenza, un ritardo di questo tipo poteva spostare contemporaneamente l’orologio di ogni telecamera. Tale processo termina con il backend; se non può essere avviato, il grandmaster viene eseguito all’interno del backend come in precedenza (`own_process` false, e il log del backend lo indica).

### Esempi

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Controllo delle telecamere LATTICE. Ogni sottocomando passa attraverso il backend Chloros; il backend gestisce il pool di telecamere, quindi le successive chiamate CLI riutilizzano lo stesso handle aperto.

### Opzioni comuni (condivise dalla maggior parte dei sottocomandi)

| Flag | Descrizione |
| --- | --- |
| `-d, --device N` | Indice della telecamera (predefinito: 0). |
| `-s, --serial SN` | Numero di serie specifico; sovrascrive `--device`. |
| `--serials SN1,SN2,…` | Numeri di serie separati da virgola per il funzionamento multi-telecamera. I [comandi transitori multi-telecamera](#multi-camera-transient-sessions) aprono solo queste telecamere, la prima delle quali attiva il trigger. |
| `--all` | Agisce su tutte le telecamere rilevate. |
| `--exposure US` | Tempo di esposizione in microsecondi. |
| `--gain DB` | Guadagno in dB. |
| `--pixel-format FMT` | ad es. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` su un M3M. Il nome definisce la profondità di bit all’interno della famiglia della fotocamera: su un M3M un nome Bayer imposta il formato Mono della sua profondità (`BayerRG8` imposta `Mono8`), su un M3C un nome Mono imposta quello Bayer (`Mono12` imposta `BayerRG12`). |
| `--width N` / `--height N` | Dimensioni dell'immagine. |
| `--preset {default,high_quality,high_speed,triggered}` | Applica un preset di impostazioni: formato pixel, esposizione, guadagno e frequenza dei fotogrammi (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` li elenca). Su un M3M il formato è quello Mono della stessa profondità: `high_speed` acquisisce `Mono8`, `high_quality` e `triggered` `Mono12`. Un flag fornito insieme ad esso (`--exposure`, `--gain`, `--pixel-format`) ha la precedenza sull’impostazione predefinita, mentre un’impostazione non specificata da quest’ultima mantiene quella della telecamera stessa. Tutte le esecuzioni libere, tranne `triggered`, che attiva la telecamera in base a un fronte di fronte hardware sulla linea 2 — se quella linea non viene attivata, la telecamera attenderà all’infinito anziché effettuare l’acquisizione. |
| `-o, --output DIR` | Directory di output (impostazione predefinita: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Dimensione del pacchetto GVSP. `auto` esegue sonde ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Flusso di lavoro per la prima connessione delle telecamere LATTICE

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Riferimento ai sottocomandi

#### Rilevamento e informazioni

| Sottocomando | Scopo |
| --- | --- |
| `lattice info` | Elenca le telecamere connesse (produttore, modello, numero di serie, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analizza il sistema host per una configurazione ottimale della telecamera. `--no-discover` salta il rilevamento delle telecamere (più veloce, analisi solo tramite scheda di rete). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Verifica/correzione delle impostazioni della scheda di rete (su Linux, anche i buffer di ricezione dei socket del kernel — vedi [host Linux](#linux-hosts--socket-receive-buffers)); stima della larghezza di banda/FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Raccomandazione relativa alla capacità di rete del backend con schema stabile+ raccomandazione sull'array (restituisce `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` utilizzano per impostazione predefinita il sensore completo, `2048` × `1536`. Esce da `0` per `ok`, `auto_capped_fps` e `auto_shrunk`, e da `1` a `needs_force_slip` e `error`, con o senza `--json`. Raggiunge il backend a `--backend-url` (impostazione predefinita `http://127.0.0.1:5000`), non a `CHLOROS_BACKEND_URL`. `auto_capped_fps` mantiene la risoluzione richiesta ma limita il numero di fotogrammi al secondo (fps) di destinazione — leggere `recommended.recommended_target_fps` e passarlo come destinazione di connessione; considerarlo un esito positivo, non come un errore. `auto_shrunk` compare solo quando il fotogramma richiesto non può essere fornito a nessuna frequenza di fotogrammi; connect non riduce mai i fotogrammi di propria iniziativa. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Analisi "what-if" senza aprire le telecamere (`--width` / `--height` predefinito `2048` × `1536`). Sia questo che `network-analysis` proiettano ciò che connect trasmetterà in streaming: senza `--pixel-format`, si applicano i formati specifici per telecamera di Connect (Mono12 / BayerRG12), mentre con `--packed auto` (impostazione predefinita) la decisione [trasporto a 12 bit compresso](#packed-12-bit-transport---packed) che Connect adotterebbe su questo computer — quindi gli fps indicati corrispondono alla velocità alla quale opera l’array. Entrambi visualizzano la scelta relativa al packing. **`--n-active` è il numero totale di telecamere sulla rete, non solo quelle di questo array**— aumentarlo quando le telecamere autonome trasmettono in streaming contemporaneamente, oppure se la capacità della linea viene calcolata in base a una domanda che ne sottostima il numero (impostazione predefinita: `len(--models)`). Visualizza sempre le righe aggregate `Wire budget:` (MB/s richiesti rispetto al limite massimo a prova di collisione) e `Max cameras:`, e segnala `** OVER-SUBSCRIBED**` quando l’array sovraccarica la linea — vedi [Modello fps e burst dell'array](#array-fps--burst-model). |
| `lattice gpu` | Mostra lo stato della GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Verifica o aggiorna il firmware della telecamera. La selezione locale di `.fwa` è bloccata: il file in `firmware/<MODEL_PREFIX>/` corrispondente all’`MIN_FIRMWARE_VERSION` della build viene flashato quando presente (solo la versione più recente come soluzione di ripiego), quindi un’immagine del fornitore più recente memorizzata sul disco rimane inattiva finché quel pin non viene modificato — le versioni più recenti raggiungono deliberatamente le unità tramite il manifesto AWS firmato, che è preferibile quando è più recente. |
| `lattice presets [--apply NAME]` | Elenca o applica i preset della telecamera. `--apply` definisce le caratteristiche dei preset (formato pixel, esposizione, guadagno, frequenza dei fotogrammi, trigger) e stampa tali impostazioni così come vengono lette dalla telecamera dopo: su un M3M, il formato Mono della profondità del preset; la frequenza dei fotogrammi a cui opera, che può essere inferiore a quella richiesta dal preset (`high_speed` richiede 60 fps; una telecamera full-frame su un collegamento a 1 Gbps funziona a circa 35). Tutte le altre impostazioni scritte da «open» vengono ripristinate così come rilevate. `--apply default` ripristina la telecamera alle impostazioni predefinite di fabbrica: le impostazioni di fabbrica con esposizione automatica a funzionamento libero su Line2, il limite di throughput del collegamento e la correzione dei difetti disattivati, il bilanciamento del bianco calibrato di una telecamera RGB mantenuto (unità su qualsiasi altra). Nulla viene salvato nella memoria della telecamera, quindi al successivo avvio riparte dalla propria memoria come in precedenza. Come `--preset` per l’acquisizione o la connessione, `default` non è un’impostazione predefinita. |
| `lattice status` | Stato della telecamera in tempo reale. |

#### Acquisizione

| Sottocomando | Scopo |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Singolo fotogramma. **Salva ogni tipo di esportazione per impostazione predefinita**(`--processing all`); vedere [Livelli di esportazione dell’acquisizione](#capture-export-levels-the-all-default). `--processing` accetta `all`, `raw`, `debayered`, `radiance`, `reflectance` o `display` (`preview` è accettato per `display`); qualsiasi altro nome viene rifiutato prima che la fotocamera venga toccata (uscita `1`). `--levels` salva un sottoinsieme esplicito (sovrascrive `--processing`). Viene aperta per l’acquisizione una telecamera che Chloros non possiede, e la sua esposizione viene impostata inizialmente in base alla scena (al massimo pochi secondi), come fanno i [comandi transitori](#multi-camera-transient-sessions); al termine dell’acquisizione, indipendentemente da come si concluda, quella telecamera viene riportata allo stato in cui è stata trovata (formato, esposizione, pipeline del colore, ogni impostazione modificata dall’acquisizione — l’elenco riportato in [Multi-Camera (Sessioni transitorie)](#multi-camera-transient-sessions)). Il DAQ che legge la cattura utilizzata viene salvato come sidecar `.daq`, che include un’acquisizione solo in formato grezzo; `--no-daq` / `--daq-tolerance-ms` lo modificano ([Il DAQ che legge una cattura salva](#the-daq-reading-a-capture-saves)). `--force-daq` è ancora accettato. `--jpeg-quality` = qualità JPEG 1–100 (impostazione predefinita 95).**I file salvati nel progetto aperto vengono aggiunti ad esso.** Come nell’app, una cattura i cui file vengono salvati nella cartella del progetto aperto in Chloros li aggiunge all’elenco dei file di quel progetto; i file salvati altrove vengono ignorati. Il comando visualizza una riga che lo indica (`Project: ...`) oppure, se è in corso un'elaborazione, che vengono messi in attesa e aggiunti al termine dell'elaborazione. Lo stesso vale per `multi-capture`, `array-capture` (anche `--smart`) e `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Trasferisce in streaming su disco fino a Ctrl+C. Un’elaborazione `tiff`i fotogrammi di un'esecuzione di `tiff` costituiscono il file `raw`, un array in cui la cattura scrive i dati di quella telecamera: il mosaico del sensore, un piano con i propri valori, non compresso, con il pacchetto di calibrazione (un fotogramma da 2048 x 1536 a 12 bit è di 6,0 MiB più il pacchetto, indipendentemente dalla scena), quindi `process` parte da questi. `png` e `jpg` contengono 8 bit, quindi quelle sequenze sono fotogrammi demosaicati per la visualizzazione, da cui `process` non ha origine. Con un DAQ associato (`daq pool-bind`) o raggruppata, ogni fotogramma ottiene i propri `.daq` e `.match.json` nella cartella `daq/` della serie, poiché `lattice capture` ne salva uno; `--no-daq` disattiva questa funzione e, all’arresto, viene visualizzato il numero di fotogrammi. Ogni fotogramma viene scritto una sola volta, con i metadati completi; premendo Ctrl+C, i fotogrammi già acquisiti vengono scritti prima che la fotocamera venga rilasciata. Si arresta automaticamente quando l’unità si riempie, stampa `Failed:` / `Disk full:` e termina con `1`; ogni fotogramma che non è stato possibile salvare genera anch’esso un messaggio di terminazione `1`. Una fotocamera lasciata pronta per un trigger hardware (`TriggerMode` `On`) viene impostata in modalità libera per la sessione. Al termine della sessione, la fotocamera viene riportata allo stato in cui è stata trovata — le impostazioni di scatto e ogni parametro modificato durante la sessione (esposizione, guadagno, formato, …) — a meno che non si verifichi l’errore `--keep-settings`, che salva le impostazioni della sessione nella memoria interna della fotocamera (funzionamento in modalità libera), oppure l’errore `--clear-settings`, che ripristina tale memoria alle impostazioni di fabbrica. Una sessione che non registra alcun fotogramma è considerata un errore: in assenza di fotogrammi per 10 s, si arresta automaticamente, stampa `No frames:` con il motivo e termina con `1`. I fotogrammi che gli scrittori non sono riusciti a gestire vengono conteggiati come persi, e l’arresto stampa `Dropped:` con il motivo (la frequenza dei fotogrammi della telecamera, quella degli scrittori e della coda); una frequenza dei fotogrammi inferiore o un formato pixel a 8 bit li conserva tutti. Il comando effettua un check-in con il backend ogni 2 s; se si arresta senza interrompere la registrazione (`kill -9`, memoria insufficiente), il backend la interrompe 30 s dopo l’ultimo check-in e riporta la telecamera (`stop_reason` `client_gone`, e una riga nel log del backend). `multi-stream` e `align-stream` fanno lo stesso. Un avvio in cui la telecamera impiega più tempo ad aprirsi rispetto al tempo di attesa del comando (60 s; 180 s per `multi-stream` / `align-stream`) — una telecamera ancora in fase di avvio — viene annullato dal backend: la telecamera viene rimessa in posizione e chiusa e nessuna registrazione rimane in esecuzione (504 HTTP `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Anteprima MJPEG in diretta basata su browser: si connette alla telecamera, visualizza l’URL dello stream e lo apre nel browser. `--ae-damping` imposta lo smorzamento dell’esposizione automatica (0,4–100). |

##### Flussi in diretta (MJPEG; nessun RTSP)

Chloros fornisce video in diretta come **MJPEG su HTTP**(`multipart/x-mixed-replace`, limite `frame`) dal backend.**RTSP non è supportato.** Aprire un URL in un browser oppure leggerlo con qualsiasi client MJPEG (`cv2.VideoCapture(url)` di OpenCV, ffmpeg, VLC). La telecamera deve essere prima collegata nel backend (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` o tramite l’interfaccia grafica).

| URL (`http://127.0.0.1:5000` + …) | Flusso |
| --- | --- |
| `/api/camera/stream/<serial>` | L’anteprima sullo schermo, fino a circa 30 fps. Per un elemento dell’array, `?max_h=` (impostazione predefinita `720`) e `?jpeg_q=` (impostazione predefinita `80`) impostano l’altezza e la qualità JPEG. Questo è l’URL che `lattice viewer` apre. |
| `/api/camera/<serial>/stream-annotated` | La stessa anteprima con le sovrapposizioni dell’interfaccia grafica disegnate in: `zebra=1`, `crosshair=1`, `grid=3x3` (o `9x9`), `histogram=1`, `peaking=1`, `spot=1`; senza alcuna richiesta, le impostazioni di sovrapposizione salvate nel progetto per quella telecamera. |
| `/api/camera/<serial>/stream-reflectance` | Riflettanza a 8 bit per banda: ρ = pixel ÷ la parte`X-Chloros-Pixel-Scale` (255 a meno che non sia impostato `refl_max`). `fps` (impostazione predefinita `10`), `jpeg_q` (impostazione predefinita `95`), `max_h` (impostazione predefinita `0` = nativo), `band=<name>`, `align=1`. Formato JPEG, quindi con perdita di dati. Rifiutato con codice di stato HTTP 409 quando la telecamera non presenta riflettanza (telecamera RGB, nessun pacchetto di calibrazione, nessuna lettura DAQ). |

- **Allineamento dei fotogrammi tra telecamere.**Ogni parte inviata da un**elemento dell’array**, su tutti e tre i flussi, contiene le intestazioni `X-Chloros-Group-Id` (il contatore del gruppo sincronizzato dell’array), le intestazioni `X-Chloros-Frame-Id` e `X-Chloros-Timestamp-Ns`. Le parti provenienti da telecamere diverse con lo stesso `X-Chloros-Group-Id` derivano dallo stesso trigger. Una telecamera autonoma non ha alcun gruppo e non invia alcun ID di gruppo. I browser ignorano le intestazioni relative alle singole parti; è il client che analizza il flusso multipart a leggerle.
- **In pausa mentre la vista array è aperta.** `/api/camera/stream/<serial>` di un membro dell’array invia un fotogramma statico “in pausa” mentre lo stream live combinato dell’array è aperto (ad esempio, la vista array della GUI) e riprende quando questo viene chiuso.

#### Ottimizzazione dei sensori

| Sottocomando | Scopo |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Leggi/scrivi qualsiasi nodo GenICam. `--set ExposureTime=…` (`Gain=…`) senza i suoi switch `ExposureAuto` (`GainAuto`) che disattivano prima la modalità automatica, come fa `lattice exposure --set`; un valore rifiutato dalla telecamera stampa `Failed to set <node>: …` su stderr e termina `1`. `--get` e la visualizzazione raggruppata predefinita visualizzano un `note:` sotto `ExposureTime` / `Gain` mentre la modalità automatica è attiva — vedi [Lettura dell’esposizione e del guadagno in modalità automatica](#reading-exposure-and-gain-in-modalità-automatica). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Esposizione e AE. Un valore di esposizione `--set` o un valore target `--brightness` viene mantenuto come impostazione predefinita (il controllo automatico dell’esposizione di Chloros non lo modifica); il guadagno rimane invariato, quindi in modalità automatica continua a regolarsi verso il valore di riferimento finché non imposti `lattice gain`. `--auto` restituisce il controllo dell’esposizione, mentre `--brightness-auto` restituisce il valore di riferimento al sistema automatico di Chloros. In assenza di flag, legge i nodi di esposizione, con lo stesso `note:` di `configure --get`. Se la fotocamera rifiuta un valore `--set`, stampa `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` su stderr — la modalità automatica, la cui scrittura era stata disattivata, viene ripristinata anziché rimanere disattivata — non stampa alcuna riga `Exposure: … us (auto OFF)` e termina con `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Guadagno e guadagno automatico. Se la telecamera rifiuta un valore `--set`, stampa `Failed to set Gain: …` su stderr, non viene visualizzata la riga `Gain: … dB (auto OFF)` e il programma termina con `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | ROI del sensore e binning. Il binning è sempre **Media** — vedi sotto. |
| `lattice format [--set FMT] [--list]` | Formato pixel. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Trigger hardware/software. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (nessun flag = bilanciamento del bianco one-shot) | Operazioni di bilanciamento del bianco. Solo per telecamere RGB/Bayer; operazione nulla (saltata) su M3M monocromatiche. L’one-shot mantiene il bilanciamento del bianco che calcola (`BalanceWhiteAuto` e i guadagni); ogni altra impostazione che scrive in modalità aperta viene ripristinata com’era, e tutto ciò avviene quando l’one-shot non può essere eseguito. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Pipeline dei colori per la visualizzazione RGB. `natural` (impostazione predefinita) è la finitura live più economica; `enhanced` aggiunge defringe + vivacità + contrasto locale CLAHE per un look completo con parità di hub a circa il doppio del costo di elaborazione per fotogramma, quindi con un framerate **in tempo reale** inferiore — le registrazioni salvate ottengono comunque l’elaborazione completa in entrambi i casi. Solo per telecamere RGB/Bayer; ignorato su M3M mono. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Saturazione/contrasto in anteprima in tempo reale (telecamere con filtro RGB); `100` è neutro, `--reset` imposta entrambi a 100. Le acquisizioni salvate non ne risentono. Ignorato sulle M3M mono. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (nessun flag = lettura) | Visualizza **Highlights** (telecamere con filtro RGB): abbassa i pixel più luminosi e più bianchi — ad esempio, una superficie bianca brillante che viene resa come una macchia bianca piatta. `0` = disattivato, `-100` = intensità massima. L’anteprima in tempo reale e le immagini acquisite a livello di visualizzazione seguono questa impostazione; la radianza/riflettanza non lo fanno mai, e non è possibile recuperare i dati del sensore che hanno subito clipping (ridurre l’esposizione per questo). Ignorato su M3M mono. |
| `lattice filter [--list]` | Elenca i modelli di filtro conosciuti dChloros (`RGN-IMX265`, `OCN`, `NGB`, …). Il filtro di una fotocamera viene letto dalla stringa del modello e non può essere impostato: `--set NAME` cerca solo un modello, quindi esce da `1` con `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Rileva i nodi di alimentazione/termici; attiva/disattiva la modalità di inattività a basso consumo. `--sleep` lascia la telecamera in modalità sleep (il limite di throughput del collegamento attivato (`DeviceLinkThroughputLimitMode` `On`) al suo valore minimo, la modalità di risparmio energetico attivata, se presente) mentre tutte le altre impostazioni vengono ripristinate così come erano all’inizio; un’entrata in modalità di sospensione che fallisce viene ripristinata integralmente. Il risveglio riporta il collegamento nelle condizioni in cui era al momento dell’entrata in modalità di sospensione (il limite, la sua modalità e il ritardo dei pacchetti `GevSCPD`, che la telecamera mantiene a 0 mentre il limite è attivo; il registro viene conservato su questo computer, così che anche un risveglio dopo il riavvio del backend lo trovi), oppure, per una telecamera che questo computer non ha messo in modalità di sospensione ma che sembra in tale stato (con il limite impostato al valore minimo), con il limite disattivato e un `GevSCPD` lasciato a 0 a 80 (impostazioni di fabbrica); un limite impostato al di sopra del valore minimo viene lasciato così com’è. La sonda e il risveglio ripristinano ciò che la loro operazione di apertura ha scritto. |

> **Il binning è sempre di tipo “Average”.** `lattice resolution --binning N` imposta il binning “Average” e
> non offre la possibilità di cambiare modalità. Il binning "Sum" moltiplica ogni DN per N², con un costo di circa due stop di
> margine per le alte luci per ogni passo di binning; Chloros divide nuovamente quel fattore prima della
> catena radiometrica, ma i pixel che il binning "Sum" ha spinto al limite massimo del sensore vengono tagliati e non possono essere
> recuperati. Non offre nulla che la modalità «Media» non offra già: un indice è invariante rispetto alla scala e gli scalari di radianza
> sono calibrati sulla scala «Media». Se una fotocamera viene ricevuta già impostata su «Somma» (lasciata
> in tale stato da un altro strumento o a seguito di un ripristino delle impostazioni di fabbrica), Chloros lo rileva da una lettura del nodo e
> compensa; l’esecuzione di `lattice resolution --binning N` la riporta su “Average”. Il flag 1.2.0
> `--binning-mode` è ancora accettato, quindi gli script più vecchi funzionano, ma viene visualizzato solo un avviso:
> il binning rimane su “Average”.

> **La modifica del binning, del formato dei pixel e del trigger su una telecamera non viene mantenuta.** Su una telecamera in cui il
> backend rimane aperto e trasmette in streaming (`cam-connect`, l’interfaccia grafica), `lattice resolution --binning`
> non ha effetto (visualizza il binning ancora in vigore, `Binning: 1x1`), e
> `lattice format --set` e `lattice trigger --mode` vengono rifiutati dalla telecamera
> (`SC_ERR_ERROR -1001`). Rilasciarlo prima (`lattice cam-disconnect -s <SN>`), oppure impostarli
> al momento della connessione (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Lettura dell’esposizione e del guadagno in modalità automatica

Mentre il nodo `ExposureAuto` (`GainAuto`) di una telecamera è in esecuzione, il suo nodo `ExposureTime` (`Gain`) mantiene l’**ultimo valore scritto**, non ciò che la telecamera sta facendo in quel momento. Per una telecamera il backend mantiene aperte le connessioni (`cam-connect`, `array-connect`, l’interfaccia grafica), `lattice configure --get`, la vista predefinita `lattice configure` e `lattice exposure` (senza flag) visualizzano quindi il valore del nodo e, sotto di esso, una nota con il fotogramma più recente della telecamera:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Leggere il valore del fotogramma dalla nota; il valore del nodo è solo l’ultima scrittura. Tramite HTTP, la stessa risposta `POST /api/camera/<serial>/read-nodes` mantiene invariato `nodes` e aggiunge `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, quello del fotogramma più recente) e `notes` (la frase sopra). Entrambi sono presenti solo se la modalità automatica non è `Off` ed è noto un fotogramma; una telecamera aperta solo per la lettura non ha alcun fotogramma e non riceve nessuno dei due. `lattice gain` (senza flag) stampa solo il nodo — utilizzare `lattice configure --get Gain` per la nota.

##### Esposizione su un elemento dell’array

Mentre un array trasmette in streaming, ogni elemento funziona con **`AcquisitionFrameRateEnable`**, a una frequenza di fotogrammi leggermente superiore alla frequenza di trigger dell’array: il suo periodo è pari al periodo di trigger meno il 15% (almeno 3 ms), oppure alla frequenza massima della telecamera stessa quando il trigger è già vicino a tale valore. `configure --get` su un elemento legge quindi `AcquisitionFrameRateEnable` come `True` e `AcquisitionFrameRate` con tale impostazione; l’abilitazione si disattiva quando l’array si arresta.**Non scrivere questi due nodi su un elemento dell’array** — l’array li mantiene sincronizzati con il proprio trigger e li riscrive.

Tale impostazione rappresenta anche il limite massimo di esposizione dell’elemento: circa 85 ms con un trigger a 10 fps (periodo di 100 ms), con **qualsiasi ROI o binning**. Prima della versione 1.3.0, una ROI di piccole dimensioni limitava invece l’esposizione di un membro al periodo di funzionamento libero del sensore (4,9 ms a 768 × 304), indipendentemente dalla lentezza con cui l’array si attivava.**Un comando manuale `ExposureTime` più lungo del limite massimo rallenta il trigger per adattarlo** quando viene scritto con `lattice exposure --set` o `configure --set` (o dall’interfaccia grafica). L’array apre innanzitutto il limite massimo di quel membro per il valore e lascia che la frequenza più lenta funzioni per alcuni periodi di attivazione, quindi la scrittura può richiedere qualche secondo in più; l’attivazione poi non avviene più velocemente di quanto consentito dall’esposizione. Su un array a frequenza automatica, la frequenza precedente viene ripristinata non appena ogni membro la cui esposizione manuale l’aveva rallentata torna all’esposizione automatica. Un valore che la fotocamera stessa non può acquisire viene comunque rifiutato, con l’intervallo accettabile indicato nell’errore, e l’esposizione automatica del membro viene ripristinata se era stata disattivata solo per quella scrittura. Un’esposizione con ricetta `apply` non rallenta il trigger: viene mantenuta al limite massimo. Chloros non accorcia mai un’esposizione manuale per adattarla a un trigger più veloce; una frequenza fissata (`array-connect --target-fps F`) più veloce di quanto l’esposizione consenta lascia a quel membro un limite massimo sufficientemente lungo per essa, e quel membro non accetta trigger più veloci di quanto la sua esposizione consenta.

#### Calibrazione e sensori

| Sottocomando | Scopo |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Calibra da un bersaglio di riflettanza. Vengono cercati fino a `--attempts` fotogrammi (impostazione predefinita `30`); un frame incompleto (perdita di pacchetti GVSP) costa un tentativo, non l’intera esecuzione. L’operazione fallisce con `no target detected in N attempts`, oppure con `no complete frame in N attempts: <last error>` se nessun frame è arrivato integro. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | Lo spettro e le ampiezze di banda di un DAQ-M, letti tramite Bluetooth (BLE) attraverso il backend. `--mac` è il MAC BLE proprio del sensore ed è richiesto per una qualsiasi delle tre azioni: non esiste un valore predefinito, poiché ogni unità ha il proprio (`daq pool-discover --only ble` li elenca); senza di esso il comando termina con `1`. In assenza di azioni, visualizza la guida. `--camera SERIAL` integra le intensità di banda visualizzate utilizzando la risposta spettrale misurata dalla fotocamera stessa anziché la curva di flotta del modello di filtro — telecamere RGB, con il pacchetto di calibrazione della telecamera caricato (collegato) o memorizzato nella cache di questo computer. L’intestazione indica quale curva è stata utilizzata: `Band magnitudes (camera <SERIAL>):`, oppure `Band magnitudes (model <filter>):` quando la telecamera non ne possiede alcuna (non è una telecamera RGB, o nessun pacchetto caricato o memorizzato nella cache) e al suo posto è stata utilizzata la curva predefinita. Tramite HTTP, `POST /api/dls/cli-read` accetta `camera_serial` e restituisce `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Applica la correzione della vignettatura alle immagini esistenti, ciascuna con il campo piatto proprio dell’obiettivo (dal proprio modello EXIF). `--lens-model` si applica alle immagini il cui modello non indica alcuna fotocamera; un’immagine il cui modello contraddice tale impostazione viene ignorata. |

#### Multicamera (sessioni transitorie)

Un comando transitorio apre le telecamere per un'esecuzione e le chiude al termine: `multi-capture` senza array persistente collegato, `multi-stream` e `multi-test` in questo caso, e `align-calibrate`, `align-apply`, `align-stream` e `index --live` di seguito. Per tutti valgono le seguenti regole:

- **`--serials SN1,SN2,…` seleziona le telecamere.** Vengono aperte solo quelle, e la prima fa scattare il trigger (per `align-calibrate` e `--reference`, se specificate). Una telecamera seriale non presente sulla rete causa il fallimento dell’esecuzione prima ancora che venga aperta qualsiasi telecamera. Senza `--serials`, vengono aperte tutte le telecamere presenti sulla rete. Una telecamera già in possesso di Chloros (GUI, `cam-connect`, `array-connect`) rifiuta l’esecuzione indipendentemente dal fatto che `--serials` la nomini o meno: essa condivide la rete per cui è impostata l’esecuzione. (`index --live` senza `--serials` apre invece le telecamere abilitate nel proprio profilo: vedi [Indice / Matematica della vegetazione](#index--vegetation-maths).)
- **Le telecamere condividono la rete nello stesso modo in cui la condivide `array-connect`.** Ogni telecamera è sincronizzata in base alla propria quota di ciò che il collegamento host trasporta (un ritardo tra i pacchetti, non il limite di throughput della telecamera, il cui limite minimo di 31,25 MB/s viene già superato da quattro telecamere su 1 GbE), quindi quattro telecamere funzionano su un collegamento da 1 Gbps. Quando le telecamere non riescono a trasmettere tutte contemporaneamente alla velocità minima che raggiungono, vengono attivate una dopo l’altra, ciascuna con il collegamento dedicato a sé, e il comando visualizza `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Quando anche una sola telecamera alla volta non rientra nei limiti, l’esecuzione viene rifiutata prima che qualsiasi telecamera inizi lo streaming: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, uscita `1`.
- **Ogni telecamera deve trasmettere, altrimenti l’esecuzione fallisce.** Una telecamera che non trasmette alcun fotogramma costituisce un errore: il comando visualizza `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, dove `<why>` indica l’errore di recupero di quella telecamera, seguito dall’indicazione della causa, e termina con `1`. Un fotogramma arrivato con perdita di pacchetti GVSP (`IncompleteBufferError: GVSP packet loss …`) indica che la rete non è in grado di supportare la larghezza di banda combinata delle telecamere (`Every missing frame arrived with GVSP packet loss: …`): controllare la velocità di collegamento dell’host e dell’uplink dello switch (`lattice network`), utilizzare i jumbo frame, oppure avviare un numero inferiore di telecamere con `--serials`. Una telecamera che non ha trasmesso nulla solitamente non riceve il segnale di trigger dalla telecamera master: controllare il cavo di sincronizzazione M8, quindi eseguire `lattice multi-detect`. La funzione di ciascun comando in presenza di un gruppo di frame in cui manca una telecamera è indicata nella rispettiva riga.
- **Ogni telecamera viene ripristinata così come è stata trovata.** Al termine dell’esecuzione, ogni telecamera riacquista lo stato in cui si trovava al momento dell’avvio dell’esecuzione: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, e `LineMode` / `LineSource` su `Line2` e `Line3` (e l’eliminazione del deglitch dell’ingresso di trigger da parte di un array fornisce la linea di uno slave, `LineFilterSelector` / `LineFilterWidth`); il suo `PtpEnable` (e l’insieme `PtpSlaveOnly` / `PtpDomainNumber` associato ad esso); `ExposureAuto` / `GainAuto` (e `ExposureTime` / `Gain` dove era impostato su un valore manuale); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (e ogni `BalanceRatio` in cui il bilanciamento del bianco era manuale); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (e `Gamma` in cui la regolazione della gamma era attiva), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; la dimensione del pacchetto (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); il ritmo di trasmissione, `DeviceLinkThroughputLimitMode` (e `DeviceLinkThroughputLimit` dove la modalità era `On`), il ritardotra i pacchetti `GevSCPD` e lo sfalsamento di inizio `GevSCFTD` che un array fornisce a ciascuna telecamera; i dati a blocchi che ogni flusso attiva (`ChunkModeActive`, ogni `ChunkEnable` sotto il proprio `ChunkSelector`); `AcquisitionFrameRateEnable` (e `AcquisitionFrameRate` dove l’abilitazione era attiva — con essa disattivata, la frequenza è di sola lettura e legge il massimo consentito dal ritmo, quindi non è un’impostazione). Viene scritto solo ciò che differisce; la fotocamera si è fermata prima, la cadenza e la frequenza dei fotogrammi prima di un'esposizione che richiede il proprio periodo di fotogramma (una fotocamera limita `ExposureTime` a un periodo di fotogramma). Con `--keep-settings` il cablaggio del trigger e della linea di sincronizzazione viene disattivato, la fotocamera funziona in modalità libera e il resto viene salvato nella memoria della fotocamera stessa come impostato dalla sessione di scatto; `--clear-settings` ripristina lì le impostazioni di fabbrica.
- **L’esposizione viene impostata in base alla scena prima che venga conservato il primo fotogramma.** La sequenza acquisisce gruppi di fotogrammi sincronizzati e li scarta mentre imposta l’esposizione e il guadagno di ciascuna telecamera in base a tali fotogrammi. L’obiettivo è ottenere una luminosità media sul bersaglio della telecamera con non più dell’1% del fotogramma in clipping; una scena con luci intense non raggiunge il bersaglio. Ciò richiede solitamente da 2 a 6 gruppi di fotogrammi e al massimo 5 s.
  - Ogni telecamera punta al bersaglio da cui parte un array collegato, in base al proprio filtro: TargetBrightness **40**su una telecamera multispettrale (RGN, OCN, NGB) e**60** su una RGB. Si tratta della metà del valore di esposizione target fornito da 80 (tre quarti su RGB), quindi è meno probabile che un pannello a riflettanza elevata subisca clipping. `--brightness N` (0-255) imposta invece il valore target di ogni telecamera (`multi-test` non ha `--brightness`: la sua esposizione automatica è disattivata). I log del backend indicano il valore target di ciascuna fotocamera: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - L’esposizione viene prima, fino al limite dell’esposizione automatica della fotocamera stessa. Il guadagno copre solo ciò che resta.
  - L’esposizione e il guadagno individuati vengono mantenuti per l’intera sessione, quindi ogni fotogramma di una `multi-stream` viene acquisito con la stessa esposizione. Un’acquisizione che deve seguire variazioni di luce deve avvenire su un array collegato (`array-connect`), la cui esposizione automatica continua a regolarsi.
  - `--exposure` e `--gain` vengono mantenuti esattamente come specificati. Se entrambi sono specificati, nessun fotogramma viene scartato.
  - Al termine dell’esecuzione, l’esposizione automatica di ciascuna telecamera viene riattivata.
  - Il log del backend riporta il risultato: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …`, oppure `exposure NOT solved after …` con la motivazione e `(not settled)` per ciascuna telecamera che non è stata inclusa. In tal caso, i fotogrammi vengono comunque acquisiti.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` modifica il limite di 5 s; `0` disattiva questa opzione (vedere [Variabili d’ambiente](#environment-variables)).

**Mentre Chloros tiene aperta una telecamera qualsiasi** (l’interfaccia grafica, `cam-connect` o un array collegato), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` e `index --live` si rifiutano di funzionare, poiché aprono tutte le telecamere presenti in rete e una seconda apertura di una telecamera già occupata può causare il crash del backend. (`index --live` si blocca quando una telecamera che dovrebbe aprire è già occupata: la sua `--serials` o le telecamere del suo profilo, e qualsiasi telecamera per un profilo che non ne indichi nessuna.) Il backend risponde con un codice HTTP 409 a `CAMERA_HELD`, indicando ogni telecamera bloccata e i comandi di sblocco (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), dopodiché il comando termina con `1`. Se è collegato un array, `array-capture --continuous` lo registra al posto di `multi-stream`, mentre `array-capture` (allineato per impostazione predefinita) salva un insieme allineato al posto di `align-apply`; `multi-capture` e `align-calibrate` utilizzano un array collegato che contiene le loro telecamere invece di aprirle.

| Sottocomando | Scopo |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Elenca tutte le telecamere con ruoli di sincronizzazione; con `--serials`, solo quelle, la prima come `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Un fotogramma sincronizzato da ciascuna telecamera. Salva **tutti i tipi di esportazione per impostazione predefinita**quando è collegato un array persistente e viene utilizzata la lettura DAQ ([La lettura DAQ che salva un'acquisizione](#the-daq-reading-a-capture-saves)); il fallback transitorio senza array viene**solo debayered** (eseguire prima `array-connect` per il resto). `--brightness` si applica solo al fallback: un array collegato mantiene la propria esposizione automatica. Il fallback non salva `.daq` (`--no-daq` / `--daq-tolerance-ms` non hanno alcun effetto in quel caso), e `multi-capture` non accetta `--force-daq`. Nella modalità di fallback, una telecamera che non fornisce alcun fotogramma fallisce l’acquisizione (`Error: multi-capture failed: no frame from …`, uscita `1`) e nessun file di quel gruppo di fotogrammi viene salvato. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Fotogrammi sincronizzati in streaming (transitori), `--fps` predefinito `3`, fino a Ctrl+C o `--count` gruppi. Un gruppo di fotogrammi per il quale una telecamera non ha fornito nulla non viene scritto, e tre di seguito terminano lo stream. Se è stato saltato un gruppo qualsiasi, l’esecuzione termina stampando `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, più `Error: the stream stopped: …` indicando la telecamera quando ha terminato lo stream, e esce da `1`X. Si arresta automaticamente quando l’unità si riempie, stampa `Failed:` / `Disk full:` e termina con `1`; qualsiasi file che non è riuscito a scrivere causa l’uscita da `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | Test di sincronizzazione GPIO (transitorio), `--count` trigger predefiniti `10`. Visualizza come è condivisa la rete (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, e la riga `Serial emit:` quando le telecamere vengono attivate una dopo l’altra, il cui sfalsamento viene poi incluso nei timestamp), una riga `Sync group: master SN:<serial> -> SN:<serial>, …` per ogni gruppo di sincronizzazione (`(no slaves: not synced)` per una telecamera senza cavo di sincronizzazione, che viene attivata autonomamente), poi una riga per ogni trigger: `[  0] SN:<serial>=<timestamp> \| …`, con `SN:<serial>=(no frame)` per una telecamera che non ha fornito alcun dato per quel trigger. Quindi una riga `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` e una riga `SN:<serial>: mean <±x> us, jitter <y> us` per ogni telecamera, ogni telecamera rispetto al master entro un ciclo di clock: un trigger i cui timestamp combinano i clock dei dispositivi delle telecamere e il PTP (PTP disattivato o ancora in fase di sincronizzazione) viene tralasciato, con un `Note:` che li conta. Qualsiasi fotogramma mancante fa fallire il test (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, uscita `1`); tre trigger incompleti di fila lo terminano anticipatamente. Il test fallisce (uscita `1`) anche quando meno di due trigger hanno tutte le telecamere sincronizzate su un unico clock (`… the clocks cannot be compared`), oppure quando i clock non sono mai allineati: gli orologi PTP presentano uno scostamento superiore a 1 ms (più eventuali sfasamenti a livello seriale) rispetto al master (`… PTP is not locked`), oppure gli orologi dei dispositivi presentano uno scostamento superiore a tale valore tra un trigger e l’altro. Avvia il grandmaster PTP se non è in esecuzione, attiva il PTP su ogni telecamera (come fa `array-connect`, indipendentemente dalla configurazione della rete) e, prima del primo trigger, attende che l’orologio PTP di ogni telecamera si sincronizzi con il grandmaster e si stabilizzi: `PtpStatus` Slave su un grandmaster, entro 100 us da esso e stabile per 3 s, e ogni telecamera sincronizzata in questo modo, senza interruzioni, per 120 s — circa 2 minuti dall’accensione. Un orologio che si è appena sincronizzato presenta uno scostamento fino a ~200 us per circa 2 minuti dopo che viene rilevato come Slave, e il valore `PtpOffsetFromMaster` della telecamera stessa non lo mostra; 120 s portano le telecamere entro ~10 us da un orologio stabilizzato. Una telecamera rimane sincronizzata da un test precedente che aveva lasciato il PTP attivo, secondo quanto rilevato da questo backend, conta a partire da quel sincronismo, quindi un test eseguito subito dopo un altro non attende nuovamente; una telecamera che il test accende, o una trovata con PTP attivo che questo backend non ha visto sincronizzarsi (accesa altrove, o prima di un riavvio del backend), attende i suoi 120 s, e in un array entrambe le telecamere appena sincronizzate attendono ancora il proprio tempo. Una telecamera che esce dallo stato Slave, supera i 100 us o segue un altro grandmaster per 3 letture di fila (1,5 s) durante l’attesa, ricomincia il proprio conteggio; una o due di queste letture vengono registrate (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) e il conteggio prosegue; un ID di grandmaster che non può essere letto viene considerato sconosciuto, non come una modifica. Il riavvio simultaneo del conteggio di tutte le telecamere (nell’arco di un secondo) è dovuto allo scorrere del tempo del grandmaster, non a quello delle telecamere: il log del backend riporta `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` lo elenca in `grandmaster_steps`, e un'attesa che poi scade inizia il proprio errore con `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Il limite non viene esteso per questo. `--ptp-settle SECONDS` imposta il tempo di assestamento per l’esecuzione (0-600; `0` per nessuna attesa di assestamento). L’attesa è limitata al tempo di assestamento più 90 s (210 s per impostazione predefinita, almeno 90 s), e il comando attende il backend per 240 s più il tempo di assestamento (360 s per impostazione predefinita). Visualizza `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)` e il log del backend riporta una riga ogni 5 s con lo stato e l’offset di ciascuna telecamera, da quanto tempo è bloccata e quanto tempo le resta: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Ogni telecamera si accende con il PTP disattivato; da quel momento gli orologi si sincronizzano in circa 10 s. Successivamente ogni telecamera viene riportata alle condizioni in cui il test l’ha trovata — il suo PTP, l’esposizione automatica e il guadagno che il test mantiene disattivati per la durata dell’esecuzione, e il resto (a meno che non si verifichi `--keep-settings`) — quindi una telecamera lasciata con il PTP attivo (`lattice configure -s <serial> --set PtpEnable=True`, o un ciclo con `--keep-settings`) rimane sincronizzata, e una volta che questo backend l’ha rilevata sincronizzata per il tempo di assestamento, un test successivo non la attende. Se i clock non si stabilizzano durante l’attesa, il test fallisce prima dell’attivazione (`PTP: NOT locked …`, uscita `1`) e indica cosa fare: una telecamera che non si è mai bloccata (`PtpStatus Listening`) significa che il grandmaster non la sta raggiungendo (`time-sync status`); una che continuava a disconnettersi viene identificata in base a quanto tempo è rimasta bloccata (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); se i clock stanno ancora convergendo: controllare `time-sync status`, e per far sì che siano sincronizzate prima del test, attivare il PTP e lasciarlo attivo (`lattice configure -s <serial> --set PtpEnable=True` per ogni telecamera, oppure eseguire il test con `--keep-settings`; il test ripristina le impostazioni rilevate, quindi rimane attivo), quindi eseguire il test. |
| `lattice multi-detect [--line LINE] [--json]` | Individua le telecamere sulla linea di sincronizzazione M8 e un master predefinito: la telecamera RGB (modello che termina con `-FRGB`) se è cablata, altrimenti la prima telecamera cablata in ordine seriale. I cavi M8 si incontrano in un unico punto di giunzione, quindi **qualsiasi telecamera collegata può fungere da master** — la dicitura stampata `MASTER` è un valore predefinito, non indica un cablaggio effettivo. Ogni telecamera, a turno, attiva la linea e le altre vengono controllate per rilevarne l’impulso; una telecamera già individuata nel gruppo cablato non viene rilevata nuovamente, quindi un array completamente cablato richiede un solo ciclo. In assenza di `--line`, viene verificato `Line2`, poi `Line3`. Stampa `MASTER: SN:…`, un `SLAVE: SN:…` per ogni altra telecamera cablata e `(probed on <line>)`; `--json` aggiunge il risultato completo (`master`, `slaves`, `no_cable` — telecamere non raggiunte dall’impulso — `line` e, per ogni telecamera, `results`). Al termine, ogni telecamera ripristina le impostazioni di trigger e linea di sincronizzazione che aveva in precedenza (gli stessi nodi di cui sopra). |

#### Allineamento

| Sottocomando | Scopo |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — manopole aggiuntive per rilevatore/corrispondenza `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, parametri RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, combinazione multi-frame `[--averaging mean\|median\|inlier_weighted]`, vincoli geometrici `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, restrizione spaziale `[--roi X0,Y0,X1,Y1] [--mask PATH]` e leslave `[--per-cam-override SN:KEY=VALUE]` (ripetibile) | Calcola il profilo di allineamento dalle telecamere in tempo reale. Impostazioni predefinite: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (nella cartella da cui si esegue il programma), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` ha le proprie impostazioni predefinite; consultare [la relativa tabella](#project-align-calibrate-options).) `--prefilter` utilizza come impostazione predefinita `gradient` (mappa dei bordi; corrisponde all’allineatore GUI/array — i bordi rimangono invariati tra le bande spettrali). `--matcher flann` dà risultati soddisfacenti con più di ~5000 caratteristiche; `--averaging median` è robusto nei confronti di una singola acquisizione errata, `inlier_weighted` pondera in base al numero di corrispondenze; `--lock-scale` proietta sulla rotazione più vicina (senza scala), `--lock-axis` azzera una componente di traslazione; `--mask` si applica a tutte le telecamere (utilizzare `--per-cam-override` per le impostazioni per singola telecamera, ad es. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` rifiuta di salvare una calibrazione il cui RMS di riproiezione supera il limite. `--method manual` prende i propri punti da `--points PATH`: un oggetto JSON `{"<serial>": [[x, y], ...], ...}` che fornisce le coordinate in pixel delle stesse caratteristiche per ciascuna telecamera, inclusa la telecamera di riferimento (almeno 2 per telecamera); ciascuno degli `--frames` viene risolto a partire da quei punti. Quando le telecamere sono contenute in un array collegato, la calibrazione utilizza i fotogrammi in tempo reale di quell’array e il profilo diventa anche l’allineamento dell’array per le sue acquisizioni. Altrimenti, viene aperto un [array transitorio](#multi-camera-transient-sessions): una telecamera che non fornisce alcun fotogramma in uno qualsiasi degli `--frames` fallisce la calibrazione (uscita da `1`, assegnandogli un nome), e nessun profilo viene salvato. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Acquisisce un fotogramma multibanda allineato. `--bit-depth`, per impostazione predefinita, si allinea alla telecamera; `--no-crop` mantiene il fotogramma completo (riempiendo con nero); `--interpolation` (impostazione predefinita `linear`) e `--border-mode`/`--border-value` (impostazione predefinita `constant`/0) controllano la distorsione della CPU — il percorso della GPU è comunque bilineare. Apre un [array transitorio](#multi-camera-transient-sessions): una telecamera che non fornisce alcun fotogramma fallisce l’acquisizione (uscita `1`, assegnandogli un nome), e non viene scritto nulla. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Flusso di fotogrammi multibanda allineati (stessi controlli di warp di `align-apply`), `--fps` predefinito `3`. Apre un [array transitorio](#multi-camera-transient-sessions): un gruppo di fotogrammi per il quale una telecamera non ha fornito nulla non viene salvato, e tre di fila terminano lo stream. Se è stato saltato un gruppo qualsiasi, l’esecuzione termina stampando `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` e `Error: …`, e termina con `1`. Anche uno streaming che termina a causa di qualsiasi altro errore stampa `Error: …` e termina con `1`. |
| `lattice align-info --profile PATH [--json]` | Visualizza i dettagli del profilo. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Modifica l'ordine dei livelli. |

#### Indice / Calcoli sulla vegetazione

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` associa i simboli di un preset alle bande** (NDVI: `red`, `nir`), uno `--channel SYMBOL=BAND` per ciascuno.

- **`--live`: solo per un simbolo a cui lo stack non può rispondere.** Come per `--input`, un simbolo di un preset che si omette assume l’unica banda dello stack (le bande delle telecamere abilitate del profilo) il cui nome inizia con esso. Questo viene verificato prima che qualsiasi telecamera si apra: un simbolo a cui nessuna banda, o diverse bande, rispondono viene quindi rifiutato, indicandone i nomi, e si esce da `1`. Un `--formula` personalizzato richiede un `--channel` per ogni simbolo.
- **I prodotti `--live` indicano a chi appartengono.** `index.tif` e `index_LUT.png` contengono i dati della fotocamera dello stack (marca, modello, numero di serie), l’ora di acquisizione (l’esposizione della fotocamera di riferimento, UTC) e l’obiettivo, ed entrambi indicano l’indice come la loro unica banda (`XMP-Camera:BandName` `NDVI`; il file PNG a colori indica l’indice, non i suoi canali di colore), proprio come fa un prodotto `--input`. Un indice composto da bande di diverse fotocamere indica invece ciascuna di esse, e il suo `Model` è `LATTICE` — vedi [File creati da più fotocamere](#files-made-from-several-cameras).
- **`--input`: solo per un simbolo a cui il file non può rispondere.** Un simbolo che si omette assume la banda del file il cui nome inizia con esso: `red` prende `Red_660` o `Red_660_RGN`, `nir` prende `NIR_850`. I nomi provengono dal file stesso: dall’elenco delle bande dello stack oppure, in una acquisizione di radianza/riflettanza Chloros o in un prodotto `process`, i suoi tag `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName`. Quindi `lattice index --input <reflectance capture> --preset NDVI` non necessita di `--channel`.
- **Due bande corrispondenti a un unico simbolo non vengono prese in considerazione** (uno stack a 9 telecamere ha due bande NIR): assegnare un nome a quella. Un simbolo a cui non corrisponde nulla viene rifiutato con `channel_map missing entries`, uscire da `1`.**I nomi delle bande in uno stack** riportano il filtro della telecamera: le bande di una telecamera M3C sono `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` su un’RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` su un OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` su un RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` su un NGB — e lala banda singola è `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` elenca le bande di un profilo.

Set completo di flag: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (ripetibile), `--capture-level raw|debayered|radiance|reflectance|unknown` (sovrascrive il livello di acquisizione registrato nel file TIFF di origine; impostazione predefinita: lettura dai metadati TIFF, con ripiego su un file Chloros `Chloros:ProcessingLevel`, quindi un'acquisizione di riflettanza riporta `reflectance`), `--output PATH` / `-o` (in modalità offline: un percorso; con un'estensione immagine i file vengono denominati a partire dal radice — `-o ndvi_FRGN.png` scrive `ndvi_FRGN.tif` e `ndvi_FRGN_LUT.png` — e in assenza di estensione si tratta di un prefisso, `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; valore predefinito accanto all’input, come `<input name>_<preset>`; con `--live` e senza `-o`, una cartella `output/` nella cartella da cui si esegue il comando), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Con `--live` si applicano anche le manopole di deformazione dell’allineamento: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (il valore di riferimento per l’esposizione automatica delle telecamere, vedi le regole relative ai transienti), `--serials SN1,SN2,…`. `--live` apre un [array dei transienti](#sessioni-transitorie-multi-telecamera): una telecamera che non fornisce alcun fotogramma fa fallire l’esecuzione (uscita da `1`, assegnandogli un nome), e non viene scritto né lo stack né l’indice. Apre `--serials` (la prima fa scattare il trigger), altrimenti le telecamere del profilo la cui voce `layer_order` è abilitata, a partire dalla sua di riferimento — la telecamera di riferimento anche se le sue bande sono disabilitate, poiché le altre sono allineate ad ad essa e fa scattare il trigger. Una telecamera le cui bande sono disabilitate non viene mai calibrata, quindi una telecamera RGB disabilitata nel profilo non fallisce più un indice di riflettanza (`no-absolute-radiance-scale`). Un profilo che non specifica alcuna telecamera apre tutte le telecamere presenti in rete, come in precedenza.

Ogni modulo viene eseguito sul backend, quindi quest’ultimo deve essere in esecuzione: avviarlo dall’interfaccia grafica di Chloros, oppure con il comando che la CLI visualizza quando non riesce a raggiungerlo. In modalità offline, il backend legge `--input` e scrive il file TIFF in formato float e il file PNG colorato autonomamente, quindi un `--input` relativo o un `-o` si risolve in base alla cartella da cui si esegue il comando (vedi [Convenzioni](#conventions)). Entrambi i prodotti offline contengono i dati di registrazione della fotocamera dell’input, proprio come avviene con un’esportazione dell’indice `process`: marca / modello / numero di serie, GPS, i tag dell’obiettivo, l’ora di acquisizione (quella dell’input stesso; quando i suoi dati EXIF non ne indicano alcuna, l’istante di esposizione registrato o la data indicata nel nome del file, in UTC) e l’identificativo di calibrazione MAPIR — mai la calibrazione raw-DN, che non si applica a un indice — oltre a `Chloros:ProcessingLevel=index` e all’espressione dell’indice; il TIFF in formato float indica una banda, il preset. `--list-presets` e `--list-gradients` stampano le tabelle delle preimpostazioni e dei gradienti del backend. Una formula rifiutata dal backend, o una banda `--channel` che il TIFF non possiede, esce da `1` con la motivazione fornita dal backend.

> **I simboli `--channel` distinguono tra maiuscole e minuscole.** Il lato del simbolo deve corrispondere esattamente ai nomi dei canali del preset (i preset utilizzano lettere minuscole, ad es. NDVI = `red`,`nir` — verificare `--list-presets`), mentre la parte relativa alla banda deve corrispondere al nome di una banda nello stack allineato (o essere un indice di banda basato su 0 in modalità offline). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` funziona; `--channel RED=660` genera un errore `channel_map missing entries` .

#### `lattice index` — Maschere per stack multibanda

In `--capture-level radiance` o `reflectance`, l’unmix 3×3 di ciascuna telecamera può limitare una banda a 0 (vedi [Maschera di limitazione dell’unmix](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Uno stack multibanda contiene le bande di diverse telecamere e ciascuna telecamera limita la propria, quindi `--live --save-multiband` scrive **una maschera per ogni telecamera sorgente** accanto allo stack:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Ogni maschera è a banda singola `uint8` sulla griglia di pixel **dello stack** (attraverso lo stesso allineamento, deformazione e ritaglio), con i bit propri di quella fotocamera: bit 0 = la banda del suo canale R, bit 1 = G, bit 2 = B. Una banda dello stack prelevata dal canale *c* di una telecamera viene limitata nel punto in cui è impostato il bit *c* di quella telecamera (R = 0, G = 1, B = 2).
- La provenienza incorporata di ciascuna banda indica il nome della maschera della relativa telecamera (`unmix_clamp_mask`, relativa alla cartella dello stack, oppure `none`) e il bit da leggere (`unmix_clamp_mask_bit`).
- Una fotocamera il cui processo di separazione dei colori non ha limitato alcun valore non ottiene alcun file; una fotocamera mono o RGB non esegue la separazione dei colori e non ne ottiene alcuno. La CLI visualizza ciascuna maschera sotto lo stack (`clamp mask SN…: …`); la risposta le elenca come `clamp_masks`.
- Il file TIFF/PNG con indice della stessa esecuzione viene calcolato dallo stesso stack, sulla stessa griglia, quindi anche le maschere lo descrivono. `--capture-level raw` non esegue alcuna operazione di separazione dei colori e non scrive alcuna maschera.
- Lo stack e l’indice si trovano direttamente in `--output`, quindi le maschere vanno inserite nella cartella `clamp_masks/` all’interno di `--output` — mai al di sopra della cartella che avete denominato. Quella cartella contiene diversi tipi di prodotti (stack, indice, rendering colorato): selezionate i file che vi servono, non passate l’intera cartella al software di fotogrammetria.

#### Connessioni persistenti (Smart-Prep, flusso equivalente alla GUI)

Questi comandi mantengono le telecamere aperte nel pool di backend tra una chiamata e l’altra dellCLI.

| Sottocomando | Scopo |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Aggiunge una telecamera al pool (singola telecamera, senza array). Le [opzioni comuni](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` e `--packet-size` (`standard` = 1500, `jumbo` = 9000, oppure in byte; sonde `auto`) vengono applicate al momento della connessione (un comando `--exposure` o `--gain` disattiva la modalità automatica). Restituisce un risultato una volta che lo streaming della telecamera è iniziato (attende circa 15 s per questo, poi 10 s per il suo primo buffer completo) e stampa `stream: streaming`, oppure `stream: no_frames -- connected, no frames yet: …` quando lo streaming non ha trasmesso nulla — nel caso di una telecamera che utilizza pacchetti jumbo, con il consiglio di riconnettersi tramite `--packet-size standard`. Mentre una telecamera si connette (una prima connessione scarica la sua calibrazione, circa 70 s), un altro comando che richiede una telecamera attende al massimo 5 s, poi fallisce con errore HTTP 503 `CAMERA_POOL_BUSY` indicando la telecamera in fase di connessione; riprovare dopo qualche istante. `lattice status` e l’elenco delle telecamere non attendono mai: una telecamera a metà connessione viene visualizzata come in fase di connessione, con la relativa fase. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Rilascio. La telecamera viene riportata allo stato in cui si trovava al momento della connessione (cablaggio del trigger, PTP, impostazioni); `--keep-settings` salva invece le sue impostazioni nella propria memoria, `--clear-settings` ripristina tale memoria alle impostazioni di fabbrica (vedi [Memoria della telecamera](#camera-memory----keep-settings----clear-settings)). Una telecamera che fa parte di un array libera l’intero array. |
| `lattice cam-list` | Elenca le telecamere nel pool. |
| **`lattice array-connect`**|**Collega un array sincronizzato persistente (il punto di ingresso consigliato).** Esegue l’intero flusso di preparazione intelligente tramite GUI. Circa un minuto per 9 telecamere; subito dopo l’accensione delle telecamere può richiederne diversi. Durante l’attesa, stampa ogni fase raggiunta dal backend, come `    <s> s: <step>` (ad esempio `    41 s: open SN:214000535 (4 of 9)`), e il log del backend presenta una riga di temporizzazione per ogni fase (`[array-connect <id>] <step>: <s> s …`) e un'ultima riga con le fasi più lente. L’attesa dura fino a 600 s. Una connessione non completata entro tale termine viene annullata dal backend: ogni telecamera aperta viene riportata allo stato in cui è stata trovata e chiusa; il comando stampa `Error:` indicando il passaggio raggiunto e termina con `1`, senza che alcun array sia connesso (504 HTTP `CLIENT_DEADLINE_PASSED`). Ctrl+C annulla la connessione allo stesso modo (`Interrupted: …`); se l’array si era già connesso, la riga lo indica e riporta il nome di `array-disconnect` che lo libera. Un comando terminato mentre è in attesa (`kill -9`) fa sì che la connessione venga annullata quando il backend non riceve segnalazioni da esso per 30 s. Un array attivo ma che non ha ancora un gruppo di frame sincronizzato viene comunque segnalato come connesso, con il proprio flusso indicato come `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Rilascia un array. Ogni telecamera viene lasciata in funzionamento libero (`TriggerMode` `Off`), con tutte le altre impostazioni così come erano al momento della connessione — le impostazioni di trigger e linea di sincronizzazione (il`TriggerSource` torna a com’era, la sua linea di sincronizzazione torna a essere un ingresso), il suo PTP e le impostazioni della telecamera modificate dalla sessione — in modo che un comando per una singola telecamera su un membro funzioni immediatamente. `--keep-settings` salva le impostazioni di ciascuna telecameracosì come erano state lasciate dalla sessione nella memoria della telecamera stessa; `--clear-settings` ripristina quella memoria alle impostazioni di fabbrica. Vedi [Memoria della telecamera](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Elenca gli array collegati. |
| `lattice array-status [--array-id ID]` | FPS in tempo reale, PTP, ultimo errore. `ptp_enabled` è uguale a `True` solo quando tutti i membri hanno PTP attivo; `ptp_off:` indica un membro che non lo ha. La riga `state:` legge `restarting (applying settings)` durante un riavvio delle impostazioni, `recovering (…)` mentre un membro che ha subito un'interruzione di alimentazione viene riaperto (vedere la nota sotto), `stalled (…)` mentre il ciclo di acquisizione è in esecuzione ma non pubblica alcun gruppo di frame, e `stopped (…)` una volta che il ciclo di acquisizione si è interrotto (ricollegare l’array). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Una acquisizione sincronizzata dall’array live — Singola / Continua / A intervalli / Più veloce. **Impostazione predefinita: `all`**(un file per ogni tipo di esportazione applicabile per ogni telecamera),**allineata** come l’acquisizione dell’app (`--no-aligned` per i file nativi). Le telecamere saltate (ad es. RGB escluse dalla radianza/riflettanza) vengono segnalate con `Skipped: SN:<serial> (<reason>)`; la lettura DAQ della cattura utilizzata viene salvata insieme ad essa, viene inclusa una cattura solo in formato raw e segnalata con `DAQ: <path>` ([La lettura DAQ di una cattura salva](#il-daq-che-legge-una-acquisizione-salva)). Vedi [Modalità di acquisizione, registratori e rielaborazione offline](#modalità-di-acquisizione-registratori--riprocessamento-offline). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Registra la vista combinata dell’indice in tempo reale in formato video/GIF (qualità di monitoraggio; senza alcuna vista GUI aperta, il backend rende l’NDVI dell’array, oppure `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Scatto in raffica Bayer grezzo ad alto fps (di livello analitico; rielaborare offline). Come `array-record` e `array-capture --continuous`, effettua il check-in con il backend ad ogni polling: se terminato senza interruzione (`kill -9`), il backend lo arresta 30 s dopo l’ultimo check-in (`stop_reason` `client_gone`; il manifesto della raffica viene finalizzato e viene avviata la creazione dell’indice combinato, come avviene in caso di arresto) e l’array rimane connesso. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Rielaborare una raffica grezza salvata in video calibrati. |

> **Una telecamera che perde alimentazione o il collegamento si ripristina automaticamente.** Quando un membro smette di rispondere (un ciclo di spegnimento e riaccensione dello switch riavvia ogni telecamera che alimenta), l’array lo attende, lo riapre quando risponde nuovamente, e le restituisce le impostazioni configurate da `array-connect`: PTP, linea di trigger e sincronizzazione, pacing di rete, formato pixel e dimensione fotogramma, nonché esposizione automatica. Successivamente riprende lo streaming senza nuove impostazioni `array-connect`. Nel frattempo, `array-status` mostra `state: recovering (...)` e `last_error` indica quale telecamera sta aspettando; successivamente stampa `re-opened:`. Una fotocamera che si è riavviata torna con il PTP disattivato, quindi finché non viene riaperta `ptp_enabled` è `False` e `ptp_off:` le assegna un nome.

> **Una telecamera in attesa non viene mai aperta una seconda volta.** Una seconda apertura interrompe il flusso di chi la detiene e ne riscrive le impostazioni, e può causare il crash del backend. Quindi, su una telecamera gestita dal pool (la GUI o `cam-connect`), un elemento dell’array o una telecamera gestita da un processo in background (un `lattice continuous` in esecuzione, `multi-stream` o `align-stream`, propria o di un altro client), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, un-shot `white-balance` e `presets --apply` rifiutano con codice HTTP 409 `CAMERA_HELD`, indicando il titolare e il comando di rilascio (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>`, oppure, nel caso di un processo, il suo ID e chi lo ha avviato), e l’uscita `1`. Lo stesso vale per una telecamera che un altro comando sta utilizzando in quel momento (un `multi-test` o `multi-capture` che apre le proprie telecamere, un altro comando one-shot), per ogni comando qui presente: l’errore indica di attendere che quel comando finisca. `lattice power` senza `--sleep` legge attraverso l’handle del titolare stesso, se presente (il pool, un array, una telecamera aperta di un processo), e viene rifiutato se la telecamera è in uso da un altro comando. Ogni risposta `CAMERA_HELD` contiene il codice sia in `code` che in `error_code`. Per i comandi che aprono tutte le telecamere, vedere [Multi-Camera (Sessioni transitorie)](#multi-camera-transient-sessions).

##### Opzioni `array-connect`

| Flag | Predefinito | Descrizione |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Rilevamento automatico di tutte le telecamere LATTICE (ne occorrono ≥2) | La prima seriale è il MASTER. Se omesso, il rilevamento filtra i modelli LATTICE (M3C / M3M) e li collega tutti. |
| `--line {Line0,Line2,Line3}` | `Line2` | Linea di sincronizzazione GPIO. |
| `--target-fps F` | auto | Frequenza di attivazione del trigger MASTER. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Ignora la selezione del livello. |
| `--wire-ceiling-mbps MB_PER_S` | rilevato automaticamente | **Il budget di banda sostenuto dall’host, in MB/s — il valore da cui dipende l’allocazione dell’intero array.** Ridurlo quando l’array segnala frame GVSP danneggiati: il valore automatico deriva dalla velocità di collegamento pubblicizzata dalla scheda di rete, che sovrastima gli adattatori USB, le linee PCIe sottodimensionate e i fabric condivisi molto trafficati. Viene memorizzato nel blocco di acquisizione dell’array del progetto, quindi una riconnessione tramite riapertura / CLI / SDK lo ripristina. Vedi [Stato di salute dell’array](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | nessuno (sensore completo) | Binning hardware (sempre «Media»). |
| `--width N` / `--height N` / `--pixel-format FMT` | sensore completo; formato per singola telecamera | Una dimensione del fotogramma o un formato pixel esplicito per ogni telecamera, invece del sensore completo e della scelta del formato per singola telecamera (Mono12 / BayerRG12). |
| `--no-recommend` | disattivato | Salta la fase di analisi della rete (ogni telecamera mantiene quindi le proprie dimensioni attuali invece di essere impostata sul sensore completo). |
| `--no-ptp` | disattivato | Disabilita PTP (i timestamp tra telecamere diverse **non** sono quindi comparabili). |
| `--packed {auto,on,off}` | `auto` | Trasmette i fotogrammi a 12 bit compressi e li decomprime su questo computer — più fotogrammi al secondo a sensore pieno, dati identici. `auto` comprime solo quando è utile e questo computer ne ha la capacità; `off` non comprime mai. Vedi [Trasporto a 12 bit compresso](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | Dimensione del pacchetto GVSP per ogni telecamera nell’array. `auto` consente a ciascuna telecamera di eseguire il sondaggio della rete; `jumbo` = 9000, `standard` = 1500, oppure una dimensione in byte la blocca. Prima della versione 1.3.0, `array-connect` accettava questo flag ma lo ignorava. |

> **Sensore completo per impostazione predefinita.** In assenza di `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect` imposta ogni telecamera sul sensore completo — senza
> binning, nessun ritaglio ROI — e lascia che la frequenza dei fotogrammi scenda a quella che questo computer e
> la rete sono in grado di fornire, anziché ridurre i fotogrammi per mantenere alta la frequenza. La
> riga di analisi della rete mostra la frequenza prevista, mentre la riga successiva elenca
> opzioni più veloci (binning o un ROI più piccolo) con le relative frequenze previste; si deve
> `--binning` oppure `--width`/`--height` per selezionarne una. Connect non esegue mai il binning né il ritaglio
> di propria iniziativa.

##### Trasporto compresso a 12 bit (`--packed`)

A pieno carico del sensore, la frequenza dei fotogrammi di un array è determinata dalla velocità con cui ciascuna telecamera può inviare
i propri fotogrammi. I formati a 12 bit (`BayerRG12`, `Mono12`) vengono normalmente trasmessi come parole a 16 bit;
il trasporto **compresso** invia gli stessi 12 bit in 1,5 byte per pixel e
li decomprime su questo computer, quindi l’invio di ogni fotogramma richiede il 25% di tempo in meno —
circa **il 33% in più di fotogrammi al secondo a piena risoluzione del sensore** (misurato: da 10,7 a 14,0 fps
su un array di 9 telecamere). **I dati salvati sono identici**: stessi valori, stessi
file a 16 bit, stessi metadati.

- `auto` (impostazione predefinita) esegue il packing solo quando è utile **e** questo computer è in grado di
  gestire il decomprimimento. All’accensione, il backend misura la propria velocità di decompressione, il numero di core della CPU,
  il carico attuale della CPU e la larghezza di banda di rete disponibile; pertanto, un computer desktop con core disponibili esegue la compressione
  a piena risoluzione del sensore, mentre un computer occupato o a bassa potenza trasmette i dati già decompressi. Le ROI di piccole dimensioni e i fotogrammi raggruppati in bin ottengono
  poco.
- `off` non comprime mai — utilizzarlo per escludere la compressione durante la diagnostica, oppure su un
  computer che deve riservare tutta la propria CPU ad altre attività.
- `on` comprime sempre i formati a 12 bit.

L’output di connect mostra la decisione e il motivo (`compresso a 12 bit: su (auto:
proiettato 11,0 -> 14,4 fps, …)`); `/api/camera/array/status` riporta
`packed_transport` e `packed_decision`. Per impostare un valore predefinito a livello di macchina
anziché passare il flag ogni volta, impostare `CHLOROS_ARRAY_PACKED12=off` (o
`on`) nell’ambiente del backend — un `--packed on|off` esplicito ha comunque la precedenza.
I formati a 8 bit e 16 bit non vengono mai compressi.

### Un unico record di progetto, per ogni modalità

Una cartella di progetto contiene le impostazioni di ogni dispositivo: `cameras.json` — quelle di ciascuna telecamera (`cameras[].settings`, per una singola telecamera e per ciascun membro dell’array) e quelle a livello di array (`arrays[].capture`: formato, ROI, binning, frequenza di trigger, wire budget) — e `sensors.json`, di ciascun sensore di luce (tempo di integrazione, media dei fotogrammi, limite massimo). L’app, la CLI, l’SDK e il server MCP leggono e scrivono tutti gli stessi file, allo stesso modo:

- **Un dispositivo collegato a un progetto viene salvato al suo interno e registra ogni modifica in esso.** «All’interno di un progetto» significa con un progetto aperto nel backend (quello dell’app) oppure — dall’SDK — tramite un `ChlorosProject`, che specifica la propria cartella. Una telecamera, un array o un sensore di luce che il progetto non contiene ancora viene aggiunto ai suoi file quando si connette (`lattice cam-connect`, `array-connect`, `daq pool-connect`, l’app, l’SDK, `chloros-mcp`), insieme alle relative impostazioni. Qualsiasi modifica apportata in qualsiasi modalità (`lattice exposure`, `gain`, `configure --set`, `resolution`, le impostazioni di un arraye la frequenza di scatto, il limite del sensore di luce, i controlli dell’app) viene salvata nei file di quel progetto un attimo dopo. Un’esposizione, un guadagno o un valore target AE impostati su una fotocamera vengono salvati come “mantenuti”, quindi riaprendo il progetto vengono ripristinati.
- **L’apertura di un progetto imposta i dispositivi in base a esso**, in ogni modalità: una fotocamera, un array o un sensore di luce collegati senza impostazioni proprie assumono quelle salvate nel progetto (`lattice cam-connect` / `array-connect` con un progetto aperto, `chloros-cli project connect`, `open_project().connect_all()`, riapertura dell’app). Un array collegato con un nuovo ID aggiorna l’array salvato dal progetto con le stesse telecamere; un array con altre telecamere viene aggiunto accanto ad esso.
- **Il salvataggio effettuato dall’app non sovrascrive mai le impostazioni in tempo reale di un dispositivo** con una loro copia precedente e mantiene le sostituzioni per singolo membro salvate da un chiamante CLI o SDK.
- **Il formato pixel impostato è quello che il progetto conserva.** Un formato impostato nelle Impostazioni dell’array, tramite `--pixel-format` o `connect_array(pixel_format=)`, 8 bit inclusi, viene salvato e ricollegato esattamente così com’è. Se non viene impostato nulla, ogni telecamera si connette con l’impostazione predefinita a 12 bit (`BayerRG12`, `Mono12` su un sensore monocromatico). Il nome di un formato indica la sua profondità di bit; la famiglia è quella del sensore, quindi una telecamera a colori non viene mai impostata su un formato mono né una telecamera mono su uno Bayer, e la riga `[PixelFormat <serial>] ... written` lo indica quando un formato è stato mappato. Le esportazioni delle acquisizioni a 8sono a 8 bit, riflettanza inclusa: ridimensionate e contrassegnate con il proprio `PixelScale`.
- Un dispositivo collegato senza alcun progetto non registra nulla.
- **Una registrazione segue un progetto creato successivamente; una sessione aperta la lascia con il proprio progetto.**Quando viene creato un progetto — un SDK `create_project()`, l’`chloros-mcp` di `create_project` o il**Nuovo progetto**dell’app — o duplicato (l’opzione**Duplica**dell’app, che apre la copia) mentre una registrazione sta ancora scrivendo nel progetto che era aperto in precedenza, la registrazione continua e ciò che viene scritto finisce nel nuovo progetto, nella stessa posizione al suo interno (sotto). L’apertura di un progetto esistente — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, tramite la funzione**Apri progetto** dell’app — lascia una registrazione ancora in corso nel progetto in cui viene registrata: essa termina lì e i relativi file si aggiungono all’elenco dei file di quel progetto (l’app interrompe ogni registrazione prima di aprire un progetto).
  - `array-record` e `array-burst`: il video, o la cartella “burst” contenente ogni fotogramma, viene spostato lì per intero quando la registrazione si interrompe (il suo `--duration`, Ctrl+C). Il file `Burst dir` generato è quello nuovo, e `--build` e `array-build-video` operano a partire da esso. Se l’unità rifiuta lo spostamento, la registrazione rimane dove è stata scritta; ildel log finale ne spiega il motivo, così come la risposta HTTP' dell'interruzione (`relocation_error`). I fotogrammi grezzi di una raffica interrotta e `.daq` vengono aggiunti all'elenco dei file del progetto in cui finiscono; un video (un `array-record`, ovvero il video generato da una sequenza) non viene aggiunto all’elenco dei file di un progetto.
  - `array-capture --continuous`: ogni acquisizione successiva alla modifica viene salvata nel nuovo progetto; quelle salvate in precedenza rimangono dove si trovano.
  - `daq pool-record` e `project sensor log` (e la registrazione tramite sensore di luce dell’app stessa): il file `.daq` viene spostato nella`light_sensor` quando la registrazione termina, e viene aggiunta all’elenco dei file di quel progetto.

  Una registrazione in una cartella al di fuori del progetto aperto (un `-o` da qualche altra parte) rimane dove si trova, mentre `lattice continuous` e `multi-stream` mantengono la cartella che è stata loro assegnata.
- **Nell’app, le acquisizioni effettuate dopo la creazione di un nuovo progetto vengono inserite in quest’ultimo.** La creazione o la duplicazione di un progetto mantiene attive tutte le sessioni di acquisizione e le registrazioni in corso, e ogni acquisizione effettuata successivamente (Acquisisci tutto, Avvia acquisizione) viene salvata nel nuovo progetto e aggiunta al relativo elenco di file. L’apertura di un progetto esistente nell’app interrompe prima tutte queste operazioni.
- **Un progetto non viene spostato mentre è in fase di registrazione.**Lo spostamento del progetto aperto in un’altra directory di lavoro (dalla sezione**Impostazioni → Directory di lavoro** dell’app; nessun comando CLI sposta un progetto, mentre un client HTTP che invia una richiesta a `/api/move-project` lo fa) viene rifiutato con codice HTTP 409 `PROJECT_RECORDING` mentre una registrazione array o una raffica, un’acquisizione continua array, un processo `lattice continuous` / `multi-stream` o una registrazione del sensore di luce vi scrivono: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` L’app rifiuta allo stesso modo, prima di richiedere una cartella, mentre sono in esecuzione le sue funzioni “Cattura tutto”, “Avvia acquisizione” o la registrazione tramite sensore di luce.

Questo è indipendente dalle impostazioni delle telecamere al momento della disconnessione (vedi [Memoria della telecamera](#camera-memory----keep-settings----clear-settings)): un progetto imposta autonomamente le proprie telecamere all’apertura.

### Memoria della telecamera — `--keep-settings` / `--clear-settings`

Quando un array è collegato, le sue telecamere assumono le impostazioni dell’array, che è poi possibile modificare. Quando Chloros disconnette le telecamere — `array-disconnect`, la fine di un comando transitorio (`multi-capture` senza array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), l’arresto di `lattice continuous`, la fine di una `lattice capture` di una telecamera che non gestisce — riporta ciascuna di esse allo stato in cui si trovava prima della connessione: il cablaggio del trigger e della linea di sincronizzazione, le impostazioni PTP e della telecamera (l'elenco completo si trova in [Multi-Camera (Sessioni transitorie)](#multi-camera-transient-sessions)).

Per utilizzare le telecamere al di fuori di un progetto, due flag su uno qualsiasi di questi comandi scrivono nella **memoria propria** della telecamera — ovvero le impostazioni con cui si avvia, indipendentemente dal software utilizzato:

- **`--keep-settings`** salva nella memoria di ciascuna telecamera le impostazioni così come erano al termine della sessione (UserSet2, che costituisce il set di avvio). Le telecamere continuano a funzionare in modalità libera (`TriggerMode` `Off`), con il cablaggio della linea di sincronizzazione così com’è. Una telecamera senza UserSet2 salva le impostazioni in UserSet1; questo computer le memorizza, quindi il sistema di autoriparazione dell’array di Chloros — che ripristina un UserSet1 che si è discostato dalla linea di base di Chloros — la lascia così com’è.
- **`--clear-settings`** ripristina la memoria di ciascuna telecamera alle impostazioni di fabbrica: la linea di base di Chloros, quella che l’autoguarigione dell’array consolida — impostazioni predefinite di fabbrica con `TriggerMode` `Off`, `TriggerSource` `Line2`, esposizione automatica, il bilanciamento del bianco calibrato di una telecamera RGB mantenuto — in UserSet1, che costituisce il set di avvio.
- I due non possono essere combinati. A ogni telecamera corrisponde una riga: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` o `SN:…: camera memory reset to the factory settings.`; la risposta `camera_memory` è la stessa per ogni telecamera.

L’app non offre questi flag.

**Ogni via d’uscita ripristina le telecamere.**Lo stesso ripristino “come trovato” viene eseguito su ogni percorso di uscita, non solo per i comandi sopra indicati: `cam-disconnect` (e la telecamera**X**dell’app); la**X** dell’app su un array o sulla riga di qualsiasi membro dell’array (un array non può continuare a funzionare senza un membro, quindi l’intero array viene rilasciato); la chiusura dell’app (che concede al backend fino a 20 s per ripristinare ogni telecamera, array e l’`lattice continuous` in esecuzione prima di interromperla; un comando ancora in esecuzione in quel momento — un comando transitorio che sta aprendo le proprie telecamere o in attesa dei loro orologi PTP, un `lattice capture` monouso, `power`, `white-balance` o `presets --apply` — si interrompe al primo punto di sicurezza, ripristina prima le proprie telecamere, per un massimo di 8 s, e risponde con HTTP 503 `SHUTTING_DOWN`; una connessione array ancora in esecuzione — `lattice array-connect`, la connessione dell’app — si interrompe prima della telecamera successiva e richiude le telecamere che aveva aperto, per un massimo di 10 s, e risponde con HTTP 503 `SHUTTING_DOWN`; e una che arriva durante lo spegnimento non apre nulla: HTTP 503 `SHUTTING_DOWN`); e un comando CLI che viene terminato o il cui terminale si chiude (SIGTERM / SIGHUP interrompono il suo lavoro — e riporta le telecamere allo stato iniziale — prima di uscire (`143` / `129`, come Ctrl+C). Un processo `lattice continuous`, `multi-stream` o `align-stream` terminato immediatamente (SIGKILL), che non può arrestare nulla, vede il proprio processo interrotto e le proprie telecamere riportate allo stato iniziale dal backend 30 s dopo il suo ultimo check-in. Un backend che va in crash o viene terminato forzatamente non può ripristinare nulla; un riavvio riporta ogni telecamera alla propria memoria, e la successiva connessione di Chloros ricollega una telecamera che trova attiva.**Un progetto imposta autonomamente le proprie telecamere.** Quando un array si connette con un progetto aperto, ogni membro riceve le impostazioni che il progetto ha salvato per esso — un valore di esposizione, guadagno o obiettivo AE mantenuto su di esso (considerato come proprio, quindi l’esposizione automatica lo lascia inalterato) e la sua modalità di bilanciamento del bianco (correzione di Chloros; il bilanciamento del bianco della fotocamera rimane disattivato) — qualunque cosa contenga la memoria della fotocamera.

### Smart-AE / Smart-Capture

Gli array LATTICE eseguono l’AE continuo in background non appena vengono collegati, ma una scena appena inquadrata richiede un attimo per convergere. `array-capture --smart` è la **soluzione pronta all’uso**: attende che l’AE si stabilizzi su tutte le telecamere dell’array, quindi avvia l’acquisizione. Utilizzatelo quando cambiate scena durante una sessione.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Per impostazione predefinita, la politica di stabilizzazione è conservativa: timeout di 5 s, finestra di stabilità di 1,5 s, tolleranza di variazione dell’esposizione di ±5 %. Regolate i parametri tramite l’SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) se avete bisogno di un comportamento diverso dall’automazione.

#### Come funziona l’esposizione automatica

- **Prima l’esposizione, poi il guadagno.** Con il guadagno su «Auto», il guadagno rimane a 0 dB finché l’esposizione non ha raggiunto il valore massimo consentito.
- **Un'esposizione manuale lascia il guadagno in modalità Auto** finché non si modifica il guadagno o lo si imposta su manuale. Tornando in modalità Auto, il limite del guadagno è quello precedente.
- **Le piccole luci non scuriscono la scena.** Una luce speculare fino all’1% dell’inquadratura viene ignorata, mentre nei pannelli di un bersaglio di calibrazione noto viene ignorata una porzione fino allo 0,5% di un pannello.
- **Il limite massimo di esposizione del DAQ si applica solo quando un bersaglio di calibrazione è visibile.** Impedisce il clipping di un pannello bianco; se nessun pannello è visibile, la fotocamera espone in base alla scena.
- **La luce costante rimane stabile, mentre quella variabile si stabilizza in pochi secondi.** In condizioni di luce costante, l’esposizione e il relativo limite rimangono invariati; dopo un’ampia variazione di luce, l’immagine si stabilizza nel giro di pochi secondi. Un clipping non limita più l’esposizione in modo permanente e, in caso di variazione dell’area di interesse (ROI), l’esposizione viene ripristinata.
- **Il bersaglio di luminosità rimane alla portata.** Non aumenta mentre la telecamera è al limite di esposizione e il suo guadagno non può salire, quindi una luce che ritorna non provoca un flare nell’inquadratura.
- **Una telecamera la cui esposizione automatica smette di rispondere viene mantenuta al livello corrispondente alla luce in cui si trova.** In pieno sole una telecamera potrebbe scendere alla sua esposizione minima e rimanere lì con un fotogramma scuro, con tutte le impostazioni che risultano normali. Chloros monitora l’inquadratura, mantiene la fotocamera al livello che aveva impostato per quella luce e la lascia andare quando la luce cambia. Un sensore di luce oscurato o coperto non interferisce con questo processo.

### Livelli di acquisizione ed esportazione (impostazione predefinita di `all`)

A partire da questa versione, `lattice capture`, `lattice multi-capture` e `lattice array-capture` **impostano come predefinito `--processing all`** — un file salvato per ogni tipo di esportazione applicabile a ciascuna telecamera, in linea con il comportamento dell’opzione “Acquisisci tutto” dell’interfaccia grafica. I livelli sono:

| Livello | Output | Si applica a |
| --- | --- | --- |
| `raw` | Bayer a canale singolo (telecamere monocromatiche: la singola banda) direttamente dal sensore. | Tutte le telecamere. |
| `debayered` | Colore a 3 canali, salvato nell’ordine dei canali **RGB**(un file TIFF con interpretazione fotometrica RGB; anche i file PNG e JPEG si aprono come RGB): l’offset scuro (DSNU) del pacchetto di calibrazione viene sottratto dal raw, quindi viene eseguito il demosaic con riconoscimento dei bordi di OpenCV— senza LUT di linearità, flat field o unmix (telecamere monocromatiche: scala di grigi a 1 canale). I fotogrammi debayered in memoria restituiti dall’SDK Python (`grab()`, `frame_stream()`) sono in formato**BGR**, secondo l’ordine di OpenCV; solo il file salvato è in formato RGB. | Tutte le telecamere. |
| `radiance` | float32 W/m²/sr/nm tramite la catena radiometrica completa. | Solo multispettrali (M3C/M3M) — **tralasciato per le telecamere con filtro RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), pronto per Pix4D. | Solo multispettrale, e **solo quando è associato un DAQ e la fotocamera è calibrata**; altrimenti viene saltato. |
| `display` | Catena completa di anteprima GUI (CCM + WB + gamma secondo il profilo della fotocamera). Salvata in una cartella `preview/`. `lattice capture` accetta anche il nome `preview`; `array-capture` / `multi-capture` accettano solo `display`. | Tutte le webcam. |

Specificare un singolo livello per salvarne solo quello (`--processing debayered`). Quando si richiede `all`, i livelli che non si applicano a una determinata camma vengono ignorati (e segnalati), ma non generano errori: una telecamera non collegata o non calibrata riceve comunque `raw` / `debayered` / `preview`.

#### Il DAQ che legge una cattura salva

Ogni volta che un DAQ è associato alla telecamera o all’array, oppure è in pool, la lettura downwelling utilizzata per una cattura viene scritta in un file **`.daq`** nella sottocartella `daq/` della cartella delle acquisizioni, con il suo `.match.json` (la distanza del valore rispetto all’esposizione) e il suo `.cameras.json` (a chi appartiene il valore), e riportato su una riga `DAQ:`. Questo vale per ogni livello, incluse le acquisizioni raw o debayered, quindi qualsiasi acquisizione può essere successivamente rielaborata in radianza, riflettanza o indice. Si tratta dello stesso file, con lo stesso nome e gli stessi sidecar, indipendentemente dal fatto che l’acquisizione provenga dalla GUI, dalla CLI o dall’SDK. In assenza totale di letture DAQ non esiste un `.daq`, e un `multi-capture` senza array collegato (il fallback transitorio) non salva nulla.

| Flag | Effetto |
| --- | --- |
| `--no-daq` | Non salvare la lettura su un'acquisizione i cui livelli non la utilizzano (`raw`, `debayered`). Un livello `radiance`, `reflectance` o `display` la scrive comunque, poiché la reimportazione richiede la lettura con cui è stato realizzato il prodotto. |
| `--daq-tolerance-ms MS` | Quanto può discostarsi la lettura dall’esposizione per essere ancora considerata entro tolleranza (valore predefinito 500). Cambia solo il verdetto (`.match.json`, `in_tolerance` e `tolerance_ms` per , `DlsInTolerance` per la riflettanza), mai il valore utilizzato. Vedi [Una lettura DAQ lontana dall’esposizione](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Il metodo precedente per richiedere l’`.daq` in un’acquisizione solo raw. Ora è l’impostazione predefinita, e sia l’`lattice capture` che l’`array-capture` accettano ancora il flag (`multi-capture` no). `--no-daq` ha la precedenza su di esso. |

Quale DAQ: quello associato alla fotocamera o all’array (la selezione del sensore di luce nella GUI, un’associazione salvata nel progetto o `chloros-cli daq pool-bind`), altrimenti il più recente disponibile nel backend. Una raffica raw (`array-burst`) mantiene il proprio `.daq` per ogni lettura distinta nella cartella della raffica, come in precedenza.

**I formati PNG e JPEG contengono 8 bit per canale, quindi utilizzano solo i prodotti `display` / `preview` e quelli indicizzati.** `--format png` o `jpg` con un livello che non possono contenere vengono rifiutati prima che venga acquisito qualsiasi dato, indicando il livello e il formato TIFF che lo supporta:

- `radiance` (float32) e `reflectance` (uint16, `32768` = 1,0) — sempre, quindi anche `--processing all` (il valore predefinito) con `--format png`/`jpg` viene rifiutato: utilizzare `--format tiff` oppure `--levels display`.
- `raw` e `debayered` provenienti da una telecamera che trasmette più di 8 bit (i frame M3C a 12 bit / Mono12 M3M) — la CLI non è in grado di rilevare la profondità di bit della telecamera, pertanto il backend li rifiuta (400 HTTP). Il singolo`lattice capture --processing debayered` a fotogramma singolo acquisisce un fotogramma a 8 bit per PNG/JPEG e non viene rifiutato, così come un `multi-capture` senza array persistente (debayering a 8 bit).

Ogni percorso di acquisizione risponde allo stesso modo (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst`, e le rotte HTTP che si trovano dietro di esse.

### Come si presenta una cartella delle acquisizioni

Ogni tipo di esportazione viene inserito nella propria **sottocartella**all’interno di `-o`, quindi un’acquisizione a più livelli non mescola mai i diversi tipi. Lo schizzo mostra l’**acquisizione predefinita `all` di una telecamera da parte di `lattice capture`** — diversi livelli per telecamera, quindi ogni file porta il suffisso `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Come variano i nomi dei file:

- **Campo telecamera — numero di serie, poi filtro:** ogni file di acquisizione riporta il nome della telecamera che lo ha registrato come `SN<serial>-<filter>`, dove `<filter>` è la parte relativa al filtro del modello della telecamera: `SN261902142-F450` per una `M3M-L87-F450`, `SN213800234-FRGN` per una `M3C-L87-FRGN`, `SN214701292-FRGB` per una telecamera RGB. Il filtro si unisce al numero di serie con `-`, non con `_`; pertanto, il nome presenta gli stessi campi separati da `_` di prima (prefisso, data, ora, millisecondi, telecamera, livello). Una telecamera il cui modello non può essere letto (a causa di un DeviceUserID cancellato) viene denominata semplicemente `SN<serial>`. Le acquisizioni provenienti da versioni precedenti non presentano alcun filtro nel nome; Chloros legge entrambe.
- **Prefisso:** `lattice capture` assegna ai file i nomi `capture_…`; `array-capture`, `multi-capture` con un array collegato, e `project capture` / `burst` / `stream` di una serie li nomina `sync_<ts>_SN<serial>-<filter>…`.
- **Suffisso di livello solo in presenza di più livelli.**Una telecamera che salva**più**livelli in un’unica acquisizione (`all` o `--levels a,b`) aggiunge `_<level>`. Una telecamera che salva**un** livello mantiene il nome semplice, senza suffisso — `--processing reflectance` scrive `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — in modo che gli script scritti per acquisizioni a livello singolo continuino a funzionare. Il livello è ancora indicato dalla cartella.
- **Le maschere di limitazione seguono il nome del prodotto:** `<product name>_clampmask.tif`. Un’acquisizione di riflettanza a livello singolo genera `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; un’acquisizione `all` genera `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **L’estensione** segue `--format`: il valore predefinito `tiff` genera `.tiff`, mentre `--format tif` genera `.tif` (le maschere di limitazione sono sempre `.tif`).
- **Un'eccezione:** `lattice capture --processing debayered` (o `--levels debayered`) accetta un singoloe scrive `capture_<ts>_SN<serial>-<filter>.tiff` direttamente in `-o` — senza sottocartella.
- **I flussi e i burst** contengono lo stesso campo: `lattice continuous` scrive `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` e una raffica non elaborata `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` è il timestamp di acquisizione e `<serial>` il numero di serie della telecamera, quindi un gruppo sincronizzato condivide un
timestamp tra le telecamere. **Si noti l’unica asimmetria:** il livello `display` viene memorizzato in una cartella
denominata `preview/`, mentre i file multilivello mantengono `_display` nel nome — la cartella e il suffisso differiscono
solo per quel livello. I livelli sconosciuti ricorrono a una cartella con il proprio nome e, se la sottocartella
non può essere creata, il file viene scritto nella radice di output anziché andare perso.

**Rielaborazione di una cartella di acquisizioni:**indirizzare `chloros-cli process` alla**radice delle acquisizioni**
(`output/`). `process` normalmente importa solo la cartella specificata, ma quando tale cartella non contiene
immagini ma contiene sottocartelle, scende automaticamente al loro interno — in modo che le sottocartelle di livello e la
sottocartella `daq/` vengano acquisite tutte in un’unica operazione. Ogni livello di una cattura viene importato come singola immagine, con
gli altri livelli disponibili come modalità, anziché come un’immagine per livello.

È possibile anche specificare direttamente il nome di una **sottocartella di livello** (ad es. `output/raw/`). In questo modo, `daq/`
viene tralasciata, quindi copiare o indicare la lettura DAQ in modo che venga inclusa quando siderivare un prodotto radiometrico
da `raw/` — altrimenti la corrispondenza del timestamp non avrà alcun riferimento a cui riferirsi.

**L’elaborazione parte sempre da `raw`.** All’interno di ciascuna acquisizione, il fotogramma grezzo è la sorgente della pipeline;
`debayered`, `radiance`, `reflectance` e `preview` sono disponibili come modalità di visualizzazione ma non vengono mai reimmessi
nella pipeline. La rielaborazione di un prodotto derivato comporterebbe la riapplicazione di vignettatura, CCM e
calcoli di radianza già integrati nei suoi pixel, pertanto Chloros, piuttosto che
effettuare una doppia elaborazione, rinuncia a tale operazione. Due conseguenze da tenere presenti:

- I rendering `index/` e `composite/` non vengono **mai** elaborati. Sono output, non acquisizioni —
  un rendering LUT NDVI non ha un’interpretazione significativa della radianza.
- Una cartella di acquisizioni esportata **senza** `raw` (ad es. `array-capture --processing reflectance`) non ha
  alcuna fonte valida per la pipeline. Tali acquisizioni vengono importate e visualizzate normalmente, ma `process` le salta
  e lo segnala:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Se si ha realmente bisogno di far passare un prodotto derivato — una sessione hub acquisita con
  `demosaic` attivato, o una cartella legacy — `--input-level {raw,debayered,processed}` forza il punto di ingresso
  e ignora il salto. Quel flag è una via di fuga intenzionale; `auto` (l’impostazione predefinita)
  non elabora mai un’acquisizione priva di dati grezzi.

**Tag di calibrazione: solo su `raw`.** Il file `raw` di un'acquisizione contiene il pacchetto di calibrazione della fotocamera: il payload `CalibrationJSON` e i tag compatibili con MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), che descrivono i valori raw del sensore. I file di ogni altro livello (`debayered`, `display`, `radiance`, `reflectance`, il rendering `index/`) e ogni frame `lattice continuous` / `multi-stream` contiene l’identità della telecamera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) e il record proprio del fotogramma, `XMP-mapir:FrameJSON` (esposizione, guadagno, temperatura, formato dei pixel, ora di acquisizione, corrispondenza DAQ), poiché ogni prodotto `process` ne porta l’identificativo. `process` individua la calibrazione di tale file tramite il suo numero di serie e `CalBundleSha256` nella cache di calibrazione di Chloros, scaricando il pacchetto una sola volta se il computer non ne dispone.

#### File creati da più telecamere

Un file creato dalle bande di più di una telecamera — uno stack multibanda (`align-apply`, `align-stream`, `index --live --save-multiband`), un indice in tempo reale creato da diverse telecamere (`index.tif`, `index_LUT.png`), un'immagine `composite/` di un'acquisizione in array — indica il nome di ogni telecamera che ha contribuito alla sua creazione:

- **Elenchi per singola telecamera, in ordine di banda** (l’immagine composita: livello di sfondo, poi primo piano; un livello indice, le telecamere da cui provengono le sue bande): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (il pacchetto di calibrazione attraverso cui sono passate le bande di ciascuna fotocamera, altrimenti — uno stack `align-apply` / `align-stream`, le cui bande non sono state sottoposte ad alcuna calibrazione — il pacchetto della telecamera che Chloros ha memorizzato nella cache, quello da cui prendono il nome i suoi file di acquisizione), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — elenchi paralleli, in cui l’elemento *n* di ciascuno descrive la stessa telecamera; `-` indica che una telecamera non ha alcun valore (nessun pacchetto di calibrazione, un obiettivo sconosciuto). Ogni fotocamera è **una** voce: se il relativo pacchetto presenta più valori per un campo — ad esempio i certificati relativi alle lampade alogene di un pacchetto — questi vengono uniti con `;` (`F-1479-1;F-1480-1;F-1481-1`), mai con una virgola, quindi un elenco riporta una voce per ogni fotocamera, indipendentemente da come viene stampato.
- **EXIF `SerialNumber`** contiene gli stessi numeri di serie, separati da virgole (`214000535,214000551`).
- Non contiene **alcuna** identità relativa a una singola fotocamera (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): questi identificano una singola fotocamera e il relativo pacchetto, e `process` determinerebbe la calibrazione del file sulla base di essi. Un indice calcolato dalle bande di diverse fotocamere (`index.tif`, `index_LUT.png`) presenta nei dati EXIF `Model` `LATTICE` (il rig), non il modello di una singola fotocamera `LATT-…`.

Un file contenente le bande di **una** singola fotocamera — un file TIFF per banda (`--per-band`), un fotogramma `align-apply --per-camera`, uno stack o un indice in tempo reale di una singola fotocamera — contiene i tag di identità propri di quella fotocamera, come ogni altro file di acquisizione. Lo stack di `align-apply`, i file per banda e per telecamera, nonché ogni stack di `align-stream` e i relativi file per banda, riportano l’ora di acquisizione del proprio gruppo (l’esposizione della telecamera di riferimento, UTC), e un fotogramma `--per-camera` di una telecamera mono è un unico piano, in `--bit-depth` come lo stack.**Orientamento della telecamera (capovolgimento / rotazione).** L’impostazione di orientamento di una telecamera (le opzioni Mirror H /
Mirror V / rotazione) determina ciò che viene salvato in un'acquisizione:

- **`raw` e `debayered` vengono salvati esattamente come li ha letti il sensore**, indipendentemente dall’impostazione. Un
  mosaico di Bayer ruotato o capovolto presenterebbe uno schema di filtri colorati diverso, e le mappe di calibrazione
  (offset scuro, campo piatto) si basano sulla griglia propria del sensore.
- **`preview`/`display`, `radiance`, `reflectance` e il rendering `index/` sono orientati** — ruotati
  una sola volta, dopo tutti i calcoli di calibrazione, in modo da aprirsi nella stessa direzione in cui era inquadrata la fotocamera. I loro
  `clamp_masks/…_clampmask.tif` si trovano sulla stessa griglia.
- **Ogni file di acquisizione registra l’impostazione** in XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, in senso orario, dopo
  i capovolgimenti) e da `ChlorosOrientationApplied` a `false` sui prodotti grezzi e debayered (pixel nativi),
  `true` sui prodotti orientati. `ChlorosProcessing` elenca le operazioni applicate ai
  pixel di quel file (`none` quando nativo). Ciò include ogni fotogramma trasmesso da `lattice continuous` (l’
impostazione propria della fotocamera, non applicata) e ogni fotogramma di `lattice multi-stream` (l’identità: i suoi
fotogrammi componenti sono nativi, come quelli di un’acquisizione ad array). Gli stack `lattice align-stream` / `align-apply`
  registrano l’impostazione della fotocamera principale, non applicata: le loro bande vengono deformate nel fotogramma della fotocamera principale
  ma non ruotate.
- **`process` applica l’orientamento registrato** quando elabora un file raw: offset scuro, campo piatto, demosaic e unmix nelle coordinate del sensore per primi, la deformazione di allineamento dell’array successivamente, e
  orientamento per ultimo — in modo che i prodotti di anteprima, radianza, riflettanza e indice risultino esattamente come
  quelli dell’acquisizione in tempo reale, e registri `ChlorosOrientationApplied=true`. I prodotti di riserva non calibrati
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) sono orientati allo stesso modo, dopo
  il campo piatto. Il prodotto debayered rimane in formato nativo.
- **Il visualizzatore di immagini Chloros mostra un file raw o debayered ruotato in base al suo record** (solo a scopo di visualizzazione —
  il file non viene modificato), in modo che appaia come i suoi prodotti; la miniatura, la vista a schermo intero, la
  lettura del cursore e l’area di prova Indice/LUT utilizzano tutti la stessa rotazione. Un file senza record e
  un prodotto (già orientato) vengono visualizzati così come sono memorizzati. Un’esportazione nell’area di prova registra l’orientamento
  contenuto nei suoi pixel (`ChlorosOrientationApplied=true`; l’identità per un livello senza orientamento).
- **Le acquisizioni prive di record vengono elaborate come native** (comportamento invariato). Prima della versione 1.3.0, un’
  acquisizione con una singola fotocamera in cui era impostato un capovolgimento o una rotazione ruotava autonomamente i propri file raw e debayered;
  Chloros non è in grado di distinguere un file raw già-ruotato da uno nativo e non effettua ipotesi, quindi lo
  elabora così come è stato salvato. Le mappe di calibrazione si trovano sulla griglia del sensore stesso, quindi per una
  cattura di questo tipo è consigliabile utilizzarne una realizzata con la versione 1.3.0 quando la calibrazione è rilevante.
- **Gli array non ruotano alcun file membro per impostazione predefinita** (la deformazione di allineamento e l’XMP `Alignment*` si trovano nei
  fotogrammi originali delle fotocamere, non ruotati) e registrano l’identità. `array-capture --orient` ruota i
  prodotti di ciascun membro, dopo la trasformazione di allineamento: in base all’orientamento del **master** quando sono stati
  trasformati nel suo frame (in modo che i membri rimangano co-registrati), altrimenti in base a quello della propria fotocamera. Ogni
  file registra l’orientamento del fotogramma in cui si trovano i propri pixel — un file raw, quello della propria telecamera —
  e un file raw che contiene la trasformazione di allineamento registra anche `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: quello del master, per un prodotto allineato
  nel fotogramma del master. `process` ruota ciascun prodotto in base a quello che appartiene ai suoi pixel:
  quello allineato quando deforma il file raw (`--array-alignment`, l’impostazione predefinita — i prodotti dell’acquisizione allineata in tempo reale), quello della fotocamera quando non lo fa (`--no-array-alignment` — quelli dell’acquisizione in tempo reale
  non allineata). La stessa opzione esiste per `array-burst`, `project capture` / `burst` /
  `stream`, le ricette e l’SDK.

### Acquisizioni saltate in array con filtri misti

Quando si combinano telecamere RGB e multispettrali in un unico array, `array-capture --processing radiance` (o `reflectance`) salva i fotogrammi multispettrali e **salta** quelle delle telecamere RGB: la radianza per pixel Bayer non è significativa per un sensore a banda larga. L’CLI visualizza esplicitamente ogni file salvato (con il relativo livello di esportazione), ogni `.daq` scritto e ogni salto, quindi il numero di file non è sorprendente:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

I token che indicano il motivo del salto seguono lo schema `<level>-not-applicable-to-rgb-cam`. Una telecamera il cui pacchetto di calibrazione non è caricato salta `radiance` / `reflectance` con `<level>-needs-calibration (connect the camera so its cal pack loads)`, mentre in assenza di una lettura DAQ utilizzabile della riflettanza salta con `reflectance-skipped-no-dls`. La riflettanza può anche essere saltata con `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, e con `dls-uncalibrated-band-<nm>` quando la banda si trova prevalentemente al di fuori dell’intervallo radiometricamente calibrato del sensore di luce del DAQ (~374–974 nm) — tra gli SKU disponibili solo F988, il cui percorso supportato è il flusso di lavoro con pannello di riflettanza.

Utilizzare `--processing debayered` (o `display`) per includere tutte le telecamere indipendentemente dal tipo di filtro, oppure l’impostazione predefinita `all` per ottenere tutti i livelli applicabili per ciascuna telecamera in un’unica operazione.

### Una lettura DAQ lontana dall’esposizione

Ogni valore `.daq` salvato da un’acquisizione rappresenta la lettura DAQ corrispondente a quell’esposizione, mentre il valore `.match.json` accanto ad esso registra la distanza tra i due valori (`gap_ms`) e se tale distanza rientra nella tolleranza (`in_tolerance`, 500 ms a meno che non sia stato impostato diversamente nella cattura `--daq-tolerance-ms`). Un valore al di fuori di tale intervallo viene comunque utilizzato, quindi non si perde alcun fotogramma, e viene etichettato: il file XMP del file di riflettanza contiene `DlsInTolerance=False` e `DlsMatchGapS`. `lattice capture`, `multi-capture` e `array-capture` lo riportano anche sotto il valore `.daq` a cui si riferiscono, con l’errore di flusso del DAQquando si spiega il concetto:

**Come una lettura viene abbinata a un’esposizione.** Una cattura attende la prima lettura del DAQ dopo l’esposizione, in modo da poter fondere le letture su entrambi i lati di essa (`method` `interpolated`). Un DAQ-E invia una lettura ogni 0,4 s alla luce del giorno e ogni 1-2 s in ambienti interni, quindi l’attesa si somma a circa un intervallo e mezzo di questi al comando, non all’esposizione. Non si attende un DAQ che ha smesso di inviare dati. Una lettura del DAQ-E rappresenta la media della luce nell’intervallo trascorso dalla lettura precedente, quindi viene posizionato al centro di tale intervallo: `gap_ms` viene misurato dall’esposizione a quel punto centrale, mentre `bracket_stamps_ns` fornisce i timestamp delle letture stesse. `array-capture --continuous` attende senza rallentare l’acquisizione: i suoi file grezzi vengono scritti immediatamente e i relativi `.daq` una volta che arriva la lettura. Con `--interval`, ogni passaggio attende come se fosse una singola acquisizione.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Quando da quella lettura non è stato scritto alcun file di riflettanza, la prima riga termina invece con `saved anyway; its .match.json says in_tolerance: false`.

L'acquisizione termina comunque con `0`: tutti i file sono stati scritti e indicano il loro contenuto. Un DAQ il cui flusso si è bloccato continua a fornire la sua ultima lettura, quindi controllarla con `chloros-cli daq pool-list` prima di fare affidamento sulla riflettanza di quell'acquisizione. Il backend ricollega un DAQ-E in pool che ha smesso di fornire dati (vedi [Un DAQ-E che smette di fornire dati](#a-daq-e-that-stops-delivering)). Una lettura fuori tolleranza stampa solo la sua riga `DAQ:`, a meno che non si tratti di una lettura conservata (vedi sotto).

**Una lettura conservata dopo che il relativo DAQ è venuto a mancare.** Quando nessun DAQ è in grado di fornire una lettura per un’esposizione (il DAQ associato non è disponibile, oppure nessun DAQ in pool ne possiede una, ad esempio dopo `daq pool-disconnect`), l’acquisizione riutilizza l’ultima lettura di cui disponeva Chloros, e il suo `.match.json` descrive tale lettura in relazione a questa esposizione: `method` `retained`, `gap_ms` da questa esposizione ai campioni da cui è stata effettuata la lettura, `in_tolerance` da quell’intervallo e `retained_from_epoch_ns`, l’esposizione a cui la lettura è stata inizialmente abbinata. Una acquisizione effettuata molto tempo dopo che il DAQ è stato disattivato stampa quindi le righe sopra indicate e, sotto un’altra ancora, che viene stampata anche se la lettura rientra ancora nei limiti di tolleranza:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Il suo `.daq` indica ancora il DAQ che ha effettuato la lettura, con la calibrazione e il cappuccio in cui è stata effettuata, in modo che una reimportazione sappia quale DAQ, calibrazione e cappuccio l’hanno prodotta.

### File che non vengono scritti

Una telecamera il cui file non può essere scritto — a causa di un disco pieno o di un errore di permessi — non impedisce il salvataggio dei file delle altre telecamere. Ogni comando di acquisizione (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; e, per una ricetta, `capture`, `burst` e `stream`) stampa su stderr ogni file che non è stato salvato, quindi **termina con `1`** anche se il resto è stato salvato:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Sono coperti tutti i file relativi a una cattura, compreso l’unico`lattice capture --levels debayered` con debayering a fotogramma singolo. Una lettura `.daq` il cui sidecar di provenienza `.match.json` non è stato possibile scrivere viene elencata come `[match]` (il file `.daq` stesso è stato salvato).

Su un disco pieno, una cattura ripetuta (`array-capture --continuous` / `--interval`, `project burst` / `stream`, una ricetta `burst` / `stream`) **si interrompe in quel passaggio** invece di non scrivere nulla sulla stessa unità per il resto dell’esecuzione. Qualsiasi altro errore viene segnalato in ogni passaggio in cui si verifica, e l’esecuzione prosegue fino alla fine, per poi terminare con `1`. `array-capture --continuous` (l’acquisizione continua del backend) non elenca alcun file, compresi quelli con errore: li conta nelle righe di avanzamento e di riepilogo (`N failed`, `N file(s) failed to write.`) e termina con `1` — vedi [Modalità continua](#continuous-mode---continuous).

I registratori — `array-burst` e `array-record` — segnalano i propri errori. `array-burst` si arresta automaticamente quando l’unità si riempie, o poco prima che ciò avvenga. Entrambi continuano a funzionare nonostante eventuali altri errori di scrittura. Quando uno dei due segnala un errore, questo viene visualizzato dopo i contatori e il comando **esce da `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` al posto di `Disk full:` significa che il burst si è interrotto prima di riempire l’ unità. I frame registrati prima dell’interruzione vengono conservati. `array-burst --build` salta la compilazione quando lo spazio si esaurisce, poiché i video verrebbero salvati sulla stessa unità. Liberare prima dello spazio, quindi eseguire `array-build-video --burst-dir …`. Dopo qualsiasi altro errore, la compilazione continua comunque.

Anche un `array-record` che non ha scritto alcun fotogramma rappresenta una registrazione fallita (`Recorder error: no-frames: the combined view published no frame to record …`, uscire da `1`) — vedi [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Modalità di acquisizione, registratori e rielaborazione offline

Tutti questi elementi operano su un **array persistente** (eseguire prima `array-connect`). Rispecchiano il pannello di acquisizione della GUI.

### Modalità di `array-capture`

`array-capture` è un singolo comando con quattro modalità di scatto più una serie di opzioni di esportazione:

| Modalità | Flag | Comportamento |
| --- | --- | --- |
| **Singola** *(predefinita)* | (nessuna) | Un gruppo di acquisizioni sincronizzate, poi uscita. |
| **Continuo** | `--continuous` | Acquisizioni consecutive fino a quando `Ctrl+C`, `--count N` acquisiscono, `--duration S` o fino al riempimento completo dell’unità. Esegue l’acquisizione continua propria del backend (la modalità Continua dell’interfaccia grafica) — vedi [Modalità Continua](#continuous-mode---continuous) di seguito. |
| **Intervallo** | `--interval S` | Un passaggio ogni `S` secondi (misurati dall’inizio di ciascun passaggio), stessi limiti. |
| **Più veloce** | `--fastest` | Solo dati grezzi+ la lettura DAQ assegnata + il composito dell’indice combinato; salta i calcoli relativi a radianza/riflettanza/visualizzazione in modo che il fotogramma venga elaborato rapidamente. Implica `--processing raw --force-daq`. Rielaborare successivamente l’`.daq` salvato in prodotti calibrati. |

Opzioni di esportazione (combinabili con qualsiasi modalità; tutte condividono l’endpoint GUI/SDK):

| Flag | Effetto |
| --- | --- |
| `--array-id ID` | L’array da cui acquisire (da omettere quando ne è collegato solo uno). |
| `-o, --output DIR` | Cartella di output (impostazione predefinita `output`, nella cartella da cui si esegue il comando). |
| `--format {tiff,tif,png,jpg,jpeg}` | Formato file (impostazione predefinita: `tiff`). PNG / JPEG accettano solo `display` e il rendering dell'indice; vedi [Livelli di esportazione dell'acquisizione](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Livello di esportazione singolo (`raw`, `debayered`, `radiance`, `reflectance`, `display`), oppure `all` (impostazione predefinita). |
| `--levels L1,L2,…` | Sottoinsieme esplicito di tipi di esportazione (ad es. `raw,radiance,reflectance`); **sovrascrive `--processing`**. |
| `--aligned` / `--no-aligned` | **Impostazione predefinita: allineato, come la cattura dello stesso array da parte dell’app Chloros**(la sua impostazione di cattura**Allineato** per ogni array, attiva a meno che non la si disattivi). L’esportazione non raw di ciascun membro (`debayered`, `display`, `radiance`, `reflectance`, il rendering dell’indice) viene distorta nel fotogramma della telecamera principale e ritagliata in base alla sovrapposizione comune dei membri, in modo che i file siano allineati a livello di pixel e presentino le stesse dimensioni, lo stesso ritaglio e gli stessi tag `Alignment*` dell’app — più piccoli del sensore (o della ROI) in base agli offset dei membri; in presenza di una ROI, la trasformazione viene ribasata sulla finestra. `XMP-mapir:AlignmentJSON` indica `"applied": true` e fornisce `crop_roi`. Il file grezzo rimane non deformato alle dimensioni native ma contiene la trasformazione nei metadati (`"applied": false`). `--no-aligned` salva ogni file alle dimensioni native, non deformato. `--aligned` è l’impostazione predefinita. Indipendentemente dal fatto che la correzione sia applicata o meno, ogni file membro registra la trasformazione `Alignment*` della propria telecamera una volta che l’array dispone di un profilo di allineamento — ad eccezione di una fotocamera che il profilo non allinea (esclusa o forzata all’identità a causa di una risoluzione fallita) e della fotocamera di riferimento quando nessun’altra fotocamera è allineata: queste non riportano tag `Alignment*`. In assenza di un profilo sull’array, l’acquisizione viene salvata non allineata e l’CLI visualizza il motivo su una riga `Alignment:`, come fa per ogni altro caso in cui non è stato possibile applicare la deformazione (elementi con risoluzioni diverse, un array non sincronizzato via cavo, un fotogramma su cui il profilo non può essere mappato). L’allineamento automatico dell’array crea il profilo una volta che l’esposizione si è stabilizzata; `align-calibrate` ne crea uno su richiesta. |
| `--orient` | Trasforma i **prodotti**di ciascun membro — `display`, `radiance`, `reflectance`, il rendering `index/` e `composite/` — in base all’orientamento della telecamera, dopo la deformazione di allineamento: un prodotto deformato nel fotogramma del master (impostazione predefinita) in base all’orientamento del**orientamento del master**, in modo che i membri rimangano co-registrati; uno non allineato (`--no-aligned`, o una cattura a cui non è stato possibile applicare la deformazione) in base a quello della propria telecamera.**Impostazione predefinita: disattivata — ogni file membro rimane nativo**, come sempre per le catture in array. `raw` e `debayered` rimangono nativi in entrambi i casi. Ogni file registra l’orientamento dei propri prodotti (vedi [Orientamento della fotocamera](#come-è-strutturata-una-cartella-di-acquisizioni)); l’XMP di `Alignment*` mantiene la trasformazione del fotogramma nativo, applicata prima dell’orientamento, quindi `process` (trasformazione, poi orientamento) riproduce il prodotto in tempo reale. |
| `--index` / `--no-index` | Salva / ignora la sovrapposizione dell’indice di vegetazione per singola telecamera, laddove configurata. Impostazione predefinita: renderizzarla. |
| `--no-daq` | Salta il sidecar `.daq` quando nessun livello selezionato utilizza il DAQ (un’acquisizione solo raw). Per impostazione predefinita la lettura viene salvata, in modo che i fotogrammi possano essere rielaborati offline in riflettanza/indice. `--force-daq` è ancora accettato (impostazione predefinita). |
| `--daq-tolerance-ms MS` | La tolleranza di corrispondenza DAQ per queste acquisizioni (impostazione predefinita 500 ms). |
| `--smart` | Attesa che l’AE si stabilizzi su tutte le telecamere prima dell’attivazione (vedere [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Compressione dei pixel TIFF. `none` (impostazione predefinita dalla versione 1.3.0; la versione 1.2.0 utilizzava `deflate`) = non compresso — i pixel del sensore esattamente come letti, ~6,3 MB per fotogramma a piena risoluzione e la velocità sostenuta più elevata; `deflate` = zlib L1 senza perdita di dati + predittore orizzontale, ~4,1 MB per fotogramma, più lento. Entrambi sono senza perdita di dati e vengono letti in modo identico durante l’importazione. |

> **TIFF a scrittura singola + il modello a velocità sostenuta.**Ogni file viene scritto**una sola volta**: pixel, il pacchetto XMP, IFD0 Marca/Modello/Numero di serie della fotocamera e i tag EXIF `ExposureTime` / `SerialNumber` in un unico passaggio, senza che nulla li riscriva in seguito. `--continuous` acquisisce gruppi consecutivi sincronizzati — ogni acquisizione prende il gruppo successivo a quello precedente, compreso quello pubblicato mentre l’acquisizione era ancora in fase di salvataggio — quindi funziona alla velocità dell’array. DEFLATE mantiene il GIL di Python, quindi le scritture compresse**non**vengono parallelizzate tra i thread di scrittura per ciascuna fotocamera; l’acquisizione continua a piena risoluzione alla velocità del sensore richiede scritture non compresse (impostazione predefinita)**e** un disco di classe NVMe (~600 MB/s sostenuti per un array di 9 telecamere a ~10 fps). La stessa opzione è disponibile come `compression` su `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Modalità continua (`--continuous`)

`array-capture --continuous` (senza `--interval` o `--smart`) passa il testimone alla **cattura continua propria del backend** — la sessione in cui viene eseguita la modalità Continua della GUI. Il backend salva autonomamente ogni gruppo sincronizzato, con due acquisizioni in corso e nessuna richiesta dalla CLI tra un gruppo e l’altro, in modo da stare al passo con un array veloce in cui una richiesta per acquisizione risulterebbe in ritardo (misurato a 73–77 acquisizioni/s su un array di 9 telecamere con un ROI di 512 pixel). Ogni opzione di esportazione sopra indicata si applica a ciascuna acquisizione.

- **Limiti:** `--count N` si interrompe dopo N acquisizioni (un’acquisizione = un gruppo sincronizzato: i file di ogni membro); `--duration S` dopo S secondi. In assenza di entrambi, il programma continua a funzionare fino a `Ctrl+C`, al riempimento dell’unità, a un errore o alla disattivazione dell’array. `Ctrl+C` interrompe la sessione del backend, quindi stampa il riepilogo.
- **Avanzamento** ogni 2 s, con `, N failed` e `, N groups skipped` aggiunti quando diversi da zero (un gruppo saltato è uno pubblicato dall’array per il quale non è stata effettuata alcuna acquisizione):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Nessun elenco per singolo file.** L'esecuzione termina con una riga di riepilogo — acquisizioni, frequenza, file e cartella di output — seguita da `N file(s) failed to write.` / `N group(s) skipped.` se diversi da zero:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Si interrompe** quando il disco è pieno con `Continuous capture stopped: the drive is full.`, e in caso di errore o di perdita dell’array con `Continuous capture stopped: <reason>.` (entrambi su stderr).
- **Stato di uscita:** `0` quando le acquisizioni sono state salvate e tutti i file sono stati scritti, compreso `Ctrl+C`; `1` in caso di file la cui scrittura non è andata a buon fine, un disco pieno, un errore, la rimozione dell’array o in assenza totale di acquisizioni.
- **Microsoft Defender:** quando Defender impiega un core della CPU o più per la scansione dei file scritti dall’acquisizione, l’esecuzione stampa una volta, su stderr: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.`. Consultare le [Impostazioni di sistema](#windows-exe).
- È consentita una sola acquisizione continua per array alla volta; una seconda viene rifiutata con il codice `a continuous capture is already running for this array`. Su HTTP la sessione è `POST /api/camera/array/continuous/start` (il corpo `/api/camera/array/capture` più `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` e `POST /api/camera/array/continuous/stop`.

**Intervallo**(`--interval S`, con o senza `--continuous`) e**`--continuous --smart`** mantengono il ciclo per richiesta: una richiesta di acquisizione per ogni passaggio — in ogni momento programmato per Intervallo, dopo che l’AE si è stabilizzato per `--smart` — con i file di ogni passaggio elencati come in modalità Singola, una riga `Done — N capture pass(es).` alla fine e `--count N` che conta i passaggi.

### `array-record` — video/GIF con indice combinato (di livello di monitoraggio)

Registra la **vista in tempo reale dell’indice combinato** dell’array in un file `.avi` (e, facoltativamente, in un file `.gif`). Con la vista combinata dell’interfaccia grafica aperta, registra quella vista così come viene visualizzata. Se nessuna è aperta, il backend genera la vista autonomamente: l’indice passato con `--expr` o, per impostazione predefinita, l’NDVI dell’array, una volta che l’array è allineato. L'NDVI predefinito utilizza le bande rossa e NIR più vicine a 660 nm e 850 nm, come fa la vista in tempo reale. Una vista dell'interfaccia grafica aperta durante la registrazione viene registrata finché rimane aperta. Verifica lo stato di avanzamento ogni 2 s e si interrompe su `--duration`, `Ctrl+C`, oppure quando il registratore termina automaticamente.

Una registrazione che non scrive alcun fotogramma fallisce: il registratore segnala `no-frames: …` (l’array potrebbe essere ancora in fase di allineamento o di stabilizzazione dell’esposizione) e il comando **termina con `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Flag | Predefinito | Descrizione |
| --- | --- | --- |
| `--array-id ID` | solo array | Array di destinazione (omettete se ne è collegato solo uno). |
| `-o, --output DIR` | `output` | Directory di output. Un percorso relativo si trova nella cartella da cui si esegue il comando. |
| `--fps F` | `10` | Frequenza dei fotogrammi di registrazione. |
| `--duration S` | fino a Ctrl+C | Arresto automatico dopo `S` secondi. |
| `--gif` | off | Scrivere anche una GIF animata. |
| `--gif-only` | disattivato | Salva solo una GIF (senza `.avi`). |
| `--expr EXPR` | l’NDVI dell’array | Espressione dell’indice da registrare quando non è aperta alcuna vista GUI, nei nomi delle bande dell’array (ad es. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; vedi [nomi delle bande](#index--vegetation-maths)). Un nome breve del colore (`NIR`, `Red`) funziona solo se presente in una banda dell’array. Una vista GUI aperta viene invece registrata così come visualizzata. |

### `array-burst` — raffica raw-Bayer ad alto fps (di livello analitico)

Legge direttamente il buffer del gruppo sincronizzato del ciclo di acquisizione — **non sono necessari né catena di calibrazione, né exiftool, né anteprima in tempo reale** — quindi funziona alla massima velocità di acquisizione della fotocamera. Scrive fotogrammi raw + un manifesto per fotogramma + un `.daq` per ogni lettura DLS distinta sotto `<output>/bursts/<base>/`. Rielaborare offline (comando successivo) oppure passare `--build` per eseguire immediatamente l’operazione all’arresto.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Flag | Predefinito | Descrizione |
| --- | --- | --- |
| `--array-id ID` | solo array | Array di destinazione. |
| `-o, --output DIR` | `output` | Directory di output (i burst vengono salvati in `<DIR>/bursts/<base>/`). |
| `--duration S` | fino a Ctrl+C | Arresto automatico dopo `S` secondi. |
| `--max-frames N` | illimitato | Arresto automaticodopo `N` fotogrammi grezzi. |
| `--orient` | disattivato | I prodotti generati da questa sequenza in base a questa build vengono ruotati in base all’orientamento della fotocamera — i prodotti per singola fotocamera in base a quello della propria fotocamera, quello combinato in base a quello della telecamera principale — come fa `array-capture --orient`. I fotogrammi grezzi rimangono nel formato nativo; il burst `manifest.json` registra l’orientamento di ogni membro e l’opzione. |
| `--build` | disattivato | Dopo l’arresto, rielaborare immediatamente il burst (come in `array-build-video`). |
| `--products …` | `combined:index` | Con `--build`: quali video creare (vedi sotto). |
| `--fps F` | `10` | Con `--build`: fps del video in uscita. |
| `--save-tiffs` | disattivato | Con `--build`: salva anche i file TIFF calibrati per ogni fotogramma (ciascuno con la propria [maschera di clamping unmix](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on) quando un pixel viene clippato; il risultato della compilazione li elenca come `clamp_masks`). |
| `--gif` | disattivato | Con `--build`: scrive anche GIF animate. |

### `array-build-video` — rielabora offline una raffica salvata

Allinea temporalmente ogni fotogramma grezzo alla lettura `.daq` salvata più vicina e lo fa passare attraverso la **stessa catena di radianza / riflettanza / indice della pipeline di importazione**, generando uno o più video.

`--products` è un elenco separato da virgole di elementi `kind:level`, dove `kind` ∈ `per_cam` | `combined` e `level` ∈ `radiance` | `reflectance` | `index`. Un `level` senza specificazioni (senza `kind:`) assume per impostazione predefinita il valore `per_cam`. Il valore predefinito è `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Flag | Impostazione predefinita | Descrizione |
| --- | --- | --- |
| `--burst-dir DIR` | (obbligatorio) | Percorso della cartella dei burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level` elenco, come sopra. |
| `--fps F` | `10` | FPS del video in uscita. |
| `--save-tiffs` | disattivato | Salva anche i file TIFF calibrati per ogni fotogramma insieme ai video. |
| `--gif` | off | Scrivere anche GIF animate. |

**Un video non contiene alcuna maschera di clipping** (un unico flusso di immagini, a 8 bit). Il risultato della compilazione contiene invece `clamp_fractions`: per ogni prodotto video, la quota di pixel di ogni fotogramma che l’unmix ha limitato nella banda contenuta nel suo canale R / G / B — l’`Chloros:Band{R,G,B}UnmixClampFraction` con cui verrebbe contrassegnato il prodotto radianza/riflettanza di quel fotogramma (per l’indice combinato, di ciascuna telecamera partecipante). L’esecuzione stampa il fotogramma peggiore per ogni prodotto su una riga `Clamp:`. `--save-tiffs` aggiunge le maschere per pixel per i file TIFF, in `clamp_masks/<level>/` accanto alla cartella `<level>/` dei TIFF.**Una compilazione interrotta non lascia alcun video danneggiato.** Ogni video viene salvato come `<name>.partial.avi` e assume il nome definitivo solo una volta completato; pertanto, un video precedente con lo stesso nome rimane fino a quando il suo sostituto non è completo. Eseguire nuovamente la compilazione per sostituire un file `.partial.avi` rimasto in sospeso.

L’esecuzione stampa ogni video che ha scritto, quindi qualsiasi prodotto `Skipped:` (una telecamera/livello richiesto — o l’indice combinato — che non ha prodotto alcun video, con il motivo — ad es. nessun pacchetto di calibrazione, nessuna lettura DLS per la riflettanza, o nessuna coppia di bande rossa/NIR per l’indice combinato) e qualsiasi `Note:`. Una telecamera RGB per cui è stata richiesta la radianza o la riflettanza riceve invece il proprio video di visualizzazione, con bilanciamento del bianco se è disponibile il pacchetto di calibrazione della telecamera; una telecamera che lo genera senza tale pacchetto viene indicata in un `Note:`. Restituisce **un valore diverso da zero** quando un prodotto richiesto è stato saltato o non è stato generato come richiesto (un’espressione di indice configurata non viene valutata offline); i video che sono stati generati vengono comunque scritti ed elencati.

> **Scegli il registratore giusto.** `array-record` è *di livello di monitoraggio* — registra il composito live renderizzato: la vista combinata della GUI, oppure, se non è aperta alcuna GUI, la vista NDVI (o `--expr`) del backend. `array-burst` → `array-build-video` è *di livello analitico* — salva i dati grezzi del sensore alla massima frequenza e ricostruisce successivamente video calibrati di radianza/riflettanza/indice, senza necessità di una visualizzazione in diretta.

### Telecamere monocromatiche (M3M) a banda singola

La linea **M3M**è la versione mono della**M3C**con filtro Bayer: un filtro di interferenza a banda stretta per ogni telecamera (`M3M-<lens>-F<wavelength>`, ad es. `M3M-L87-F685`), quindi il sensore fornisce una**singola banda in scala di grigi** senza mosaico Bayer. Non c’è nulla da demosaicare, nessun crosstalk tra i canali da separare e nessun bilanciamento del bianco da impostare — l’intera pipeline dei colori per la visualizzazione RGB semplicemente non si applica.

Cosa significa questo nella CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**rilevano una telecamera monocromatica e**passano oltre con un messaggio di una riga** invece di applicare impostazioni prive di significato. Continuano a funzionare normalmente con una telecamera M3C RGB/Bayer nella stessa sessione.
- **Un formato pixel Bayer indica la sua profondità.** Un M3M offre solo formati mono, quindi un formato Bayer da `--pixel-format` o `--preset` imposta il formato mono della stessa profondità: `BayerRG8` e `--preset high_speed` danno `Mono8`; `BayerRG12`, `high_quality` e `triggered` danno `Mono12`; un nome compresso rimane compresso (`BayerRG12p` dà `Mono12p`). I file e `lattice format` indicano il formato Mono. Un array che mescola telecamere M3C e M3M adotta un unico formato allo stesso modo: Bayer su ogni M3C, la sua profondità Mono su ogni M3M. Un nome corrisponde alla sua profondità di bit: lo stesso vale anche al contrario, quindi una M3C non viene mai assegnata a un formato Mono (`--pixel-format Mono12` è `BayerRG12` su una M3C, indipendentemente da quale telecamera sia la master dell’array), e `lattice format --set` viene mappato allo stesso modo e stampa il formato che ha impostato.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** funzionano ancora — la radianza e la riflettanza sono mappe radiometriche *per banda* e sono perfettamente ben definite per una singola banda. I fotogrammi monocromatici contengono una matrice di risposta del sensore **identitaria** (senza separazione 3×3), quindi il piano passa attraverso i calcoli di calibrazione senza subire modifiche.
- **Una singola fotocamera monocromatica non può produrre un indice di vegetazione.**NDVI, NDRE e simili richiedono almeno due bande (ad es. Red + NIR). Per ottenere un indice da hardware monocromatico, puntare**diverse** fotocamere M3M su diverse lunghezze d’onda, allinearle in un unico stack multibanda e calcolare l’indice *da quello*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` avvia autonomamente le telecamere (una [sessione transitoria](#multi-camera-transient-sessions)), quindi non collegare nulla in un primo momento: `--serials`, altrimenti le telecamere abilitate nel profilo (vedi [Indice / Calcoli sulla vegetazione](#index--vegetation-maths)). Finché Chloros detiene il controllo di una qualsiasi telecamera (l’interfaccia grafica, `cam-connect`, `array-connect`) la richiesta viene rifiutata con codice HTTP 409 `CAMERA_HELD` e il programma si chiude `1`; rilasciarle con `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

I simboli `--channel` devono corrispondere **esattamente** ai nomi dei canali del preset (si fa distinzione tra maiuscole e minuscole; quelli dell’NDVI sono in minuscolo `red`, `nir` — vedi `--list-presets`), e il nome della banda deve indicare una banda nello stack allineato (la modalità offline accetta anche indici di banda basati su 0, ad es. `--channel red=0 --channel nir=1`).

Il discriminante in tutto lo stack è il token `M3M` nella stringa del modello (non compare mai in una stringa `M3C`), visualizzato nell’interfaccia grafica (GUI) e nell’SDK come `is_mono`.

---

## Configurazione e ottimizzazione della scheda di rete dell’host (array LATTICE)

Le telecamere LATTICE trasmettono GVSP tramite la scheda Ethernet dell’host, pertanto negli array multicamera il **driver**e la**dimensione del ring di ricezione** sono importanti tanto quanto la velocità di collegamento. Impostazioni errate si manifestano come un gate `FRAMES WILL DROP` / `Reduce ROI to enable` nel pannello Impostazioni array (e in `lattice network-analysis` / `analyze_array_network()` dell’SDK), anche quando le telecamere stesse sono funzionanti.

### Adattatori USB 10GbE — Realtek RTL8157 ("Realtek USB 10GbE Family Controller")

Le impostazioni riportate di seguito sono le proprietà del driver di Windows. Su Linux anche un adattatore USB ha un limite massimo (400 MB/s su x86_64, 200 MB/s su NVIDIA Jetson) (vedi [Modello fps e burst dell’array](#array-fps--burst-model)), e i buffer di ricezione dei socket del kernel sono importanti per ogni adattatore — vedi [Host Linux](#linux-hosts--socket-receive-buffers).

| Voce | Valore richiesto | Perché è importante |
| --- | --- | --- |
| **Versione del driver**|**≥ v10.67 (gennaio 2026)**, INF `rtump64x64sta.inf` | Il driver legacy**del 2016**(v10.65, `rtump64x64.inf`) gestisce in modo errato lo spegnimento e i bugcheck con**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**durante lo spegnimento, il riavvio o la sospensione. La transizione si blocca (timeout di circa 5 minuti), l’utente esegue lo spegnimento forzato e ripetuti spegnimenti non corretti**danneggiano il repository WMI**(PowerShell e gli strumenti iniziano a dare errori con `Invalid class`) e**bloccano lo stack USB** al successivo avvio (la scheda di rete non si abilita; le unità USB smettono di essere enumerate). Effettuare l’aggiornamento da realtek.com (o dal fornitore del dongle) prima di affidarsi a riavvii puliti. |
| **Buffer di ricezione**— parola chiave `ReceiveBufferLen` |**256**(massimo del driver) | L’anello RX della scheda di rete. Il valore predefinito del driver di**32**lascia solo circa 0,26 MB di anello utilizzabile — decisamente troppo piccolo per un burst multi-camera — quindi il pannello dell’array segnala `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` e blocca la connessione. A**256**l’anello è ampio (**circa 13,5 MB misurati sull’host 10GbE del laboratorio**), fornendo alla pipeline RX un margine reale per i burst GVSP multicamera. (Se una data configurazione riesca effettivamente a *connettersi* è determinato da due controlli: il controllo di **ammissibilità drain-aware**e il controllo**dell’over-subscription aggregata** — non da un semplice confronto tra burst grezzo e anello; vedi [Modello fps e burst dell’array](#array-fps--burst-model).) |
| **URB in ricezione**— parola chiave `PendingReceives` |**64** (max) | Blocchi di richiesta USB in transito; aumentare insieme ai buffer di ricezione per l’assorbimento dei burst. |
| **Jumbo Frame** — parola chiave `*JumboPacket` | **9014** | Necessario per i pacchetti GVSP da 9000 byte (6 volte meno pacchetti per frame rispetto a quelli da 1500). |

> ⚠️ **Un aggiornamento del driver della scheda di rete RIPORTA queste proprietà avanzate ai valori predefiniti.**Dopo aver aggiornato o sostituito il driver della scheda,**riapplicare** `ReceiveBufferLen=256` e `PendingReceives=64`, altrimenti il pannello dell’array si bloccherà nuovamente anche se “non è cambiato nulla nell’hardware”. Questa è la causa principale per cui un sistema che prima funzionava improvvisamente si rifiuta di connettersi.

Applicare da una istanza di PowerShell con **privilegi elevati** (sostituire il nome della propria scheda, ad es. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` copre gli adattatori USB 10GbE.** Ora rileva il tipo di adattatore e imposta la parola chiave corretta per il ring di ricezione: `*ReceiveBuffers`→2048 per le schede di rete PCIe (Intel I219, ecc.), oppure `ReceiveBufferLen`→256 + `PendingReceives`→64 per il controller **USB** 10GbE Realtek (che non espone `*ReceiveBuffers`). I valori di destinazione sono limitati al massimo segnalato da ciascun driver (`NumericParameterMaxValue`), in modo che non venga mai scritto un valore fuori intervallo. Eseguirlo da un terminale **con privilegi elevati**; come per qualsiasi ottimizzazione basata sul registro, la modifica ha effetto dopo il riavvio della scheda o del sistema. I comandi manuali `Set-NetAdapterAdvancedProperty` sopra indicati rimangono una valida alternativa: si applicano in tempo reale (riconfigurando la scheda) senza necessità di riavvio.

### Host Linux — buffer di ricezione dei socket

Su Linux, compreso NVIDIA Jetson, i buffer di ricezione dei socket del kernel sono importanti tanto quanto l’anello di ricezione della scheda di rete. Con le impostazioni predefinite del kernel (`net.core.rmem_max` / `net.core.rmem_default` = 212.992 byte), una raffica di pacchetti GVSP provenienti dalla telecamera provoca un overflow del socket di stream prima che Chloros riesca a leggerli. I pacchetti vengono scartati nel kernel mentre la scheda di rete stessa non segnala alcuna perdita, e un array perde gruppi di fotogrammi e riduce la propria frequenza di fotogrammi. In base alle misurazioni effettuate su un array di 4 telecamere collegate a un adattatore USB 10 GbE, aumentando entrambi i valori a 32 MB, la velocità dell’array è passata da 4,96 a 7,52 fps dopo la connessione, mentre l’acquisizione raw continua è passata da 4,7 a 7,6 acquisizioni/s, senza che alcun pacchetto venisse scartato nel kernel.

- **Il file .deb li imposta.** L’installazione del pacchetto scrive `/etc/sysctl.d/60-chloros-gige.conf` (entrambi a `33554432`) e li applica immediatamente. Un valore già impostato su un valore più alto viene lasciato invariato.
- **`lattice network` li controlla.** Quando uno dei due valori è inferiore, `socket receive buffers too small for GigE Vision (…)` viene elencato sotto *Problemi rilevati* e il log del backend riporta lo stesso avviso al primo collegamento di una telecamera o di un array.
- **Per aumentarli manualmente** (fino al prossimo riavvio; il file del pacchetto li applica ad ogni avvio):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  Anche `lattice network --fix` li aumenta e li mantiene, ma solo quando il backend viene eseguito come root. Un backend eseguito come utente (avviato con `chloros-backend` o [`chloros-backend@<user>`](#running-the-backend-as-a-service-headless)) non può modificare le impostazioni del kernel e stampa invece la riga `sudo sysctl`.

**L’ottimizzazione automatica delle schede di rete riguarda solo le schede collegate.** Al primo collegamento della telecamera, un backend in esecuzione come root ottimizza le schede di rete cablate il cui anello di ricezione è piccolo. Ora ottimizza solo gli adattatori con un collegamento, quindi una porta integrata scollegata mantiene il proprio MTU e i buffer di ricezione non vengono modificati. `lattice network` non elenca più l’MTU di un adattatore scollegatocome un problema. `lattice network`, `--fix`, `lattice probe` e l’ottimizzazione degli adattatori prendono in considerazione solo le porte Ethernet cablate, non il bus CAN di un Jetson (`can0`) né i collegamenti in modalità dispositivo USB (`usb0`, `usb1`) che offre a un PC host; un adattatore Ethernet USB per telecamere è comunque considerato.

### Nozioni di base sulla rete (tutti i collegamenti LATTICE)

- **Indirizzamento:** link-local `169.254.0.0/16` (GigE Vision LLA). L’host utilizza un indirizzo statico `169.254.x.x/16`; le telecamere e il DAQ-E si autoassegnano indirizzi nello stesso intervallo. Non sono richiesti DHCP né gateway.
- **Dimensione del pacchetto:**preferire jumbo (9000), ma lasciare che sia l’auto-probe a individuarla — supera il limite ICMP di 1500 byte della telecamera tramite una sonda GVSP, quindi si imposta su jumbo ovunque la linea lo supporti effettivamente.**Ogni telecamera viene misurata una volta per ogni sessione del backend e nuovamente quando il suo collegamento cambia (un altro adattatore, un’altra velocità di collegamento o un collegamento interrotto, come nel caso della sostituzione di uno switch); le riconnessioni riutilizzano il risultato. Dopo una modifica a monte di uno switch rimasto attivo, riavviare il backend per eseguire nuovamente la misurazione.**Una prima connessione può occasionalmente richiedere alcuni minuti per questo controllo, e il log lo segnala quando ciò accade. Utilizzate il pin con `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` solo quando ne sapete più della sonda, e preferite l’impostazione per singolo comando a quella permanente: un pin salta la sonda, quindi se il percorso non è effettivamente in grado di trasportare 9000,**ogni** acquisizione va in timeout con `SC_ERR_TIMEOUT -1011` (vedi [Variabili d’ambiente](#environment-variables)).
- **L'anello RX varia in base a `ReceiveBufferLen`:**al valore predefinito `32` l’anello utilizzabile è di ~0,26 MB (troppo piccolo per qualsiasi burst multi-camera); al valore massimo `256` è ampio (~13,5 MB misurati sull’host da 10GbE del laboratorio), offrendo un margine reale. La possibilità che una configurazione si connetta viene quindi determinata dal controllo di ammissione sensibile al consumo**e** dal controllo di sovrasottoscrizione aggregata riportato di seguito — non da un semplice confronto tra burst grezzo e anello.

### Modello di fps e burst dell’array

Come interpretare il pannello Impostazioni array (e `lattice analyze-array` / `analyze_array_network` dell’SDK):

- **Il burst viene sommato per ogni telecamera in base al formato pixel effettivo di ciascuna telecamera.**Le telecamere mono**M3M**trasmettono in streaming**Mono12 (2 B/px)**, oppure, con `--pixel-format`, il formato mono della propria profondità (`BayerRG8` corrisponde a `Mono8`, 1 bit/pixel); le telecamere Bayer**M3C**trasmettono a 8 o 12 bit (il firmware M3C emette silenziosamente BayerRG12 anche quando viene richiesto BayerRG8). Quindi un fotogramma a piena risoluzione con 4 telecamere è**~12,6 MB se tutte a 8 bit, ma ~25 MB con tre telecamere mono a 12 bit**. La proiezione determina il formato di ciascuna telecamera in base al suo modello (cache di identità), quindi il burst corrisponde a ciò che il cavo trasporta effettivamente — non a un'ipotesi standardizzata di BayerRG8.
- **Un adattatore Ethernet USB ha un limite massimo di 200 MB/s indipendentemente dalle specifiche tecniche riportate sulla targhetta.** La tabella di efficienza che converte una velocità di collegamento in un valore sostenuto deriva dallo standard PCIe; una scheda di rete USB dichiara la propria velocità di collegamento *Ethernet*, ma è limitata dal bus USB e dal relativo driver. Una chiavetta USB 10GbE ha registrato circa 1063 MB/s "sostenuta" — un valore che non è mai stato verificato — e il conseguente rallentamento ha corrotto il 6–18 % dei frame pur continuando a riportare un numero di fps target corretto. Le schede di rete collegate via USB sono ora limitate in modo assoluto: **200 MB/s** su Windows e su NVIDIA Jetson, **400 MB/s** su Linux x86_64 (su Linux, a un adattatore collegato a una porta USB 2.0 vengono assegnati **30 MB/s**, e le specifiche tecniche consigliano di collegarlo a una porta USB 3) (il limite è dato dal bus USB e dal driver dell’host, quindi non è proporzionale alle specifiche tecniche; un adattatore USB 1 GbE raggiunge circa 80 MB/s e non ne risente). Su Linux, l’adattatore viene identificato tramite sysfs (ricorrendo a `ethtool -i` come `bus-info`). Ogni dato è stato misurato: su Windows lo stesso array di 4 telecamere ha corrotto il 6–10 % dei fotogrammi oltre i 200 MB/s; su Linux x86_64 lo stesso adattatore ha funzionato senza problemi a ogni soglia massima fino a 480 MB/s (273 MB/s effettivi, il limite delle telecamere stesse). Su NVIDIA Jetson lo stesso adattatore ha funzionato correttamente solo a 200 MB/s, perdendo fotogrammi in modo intermittente al di sopra di tale soglia. `wire_ceiling_source` nel registro delle capacità lo indica esplicitamente, mentre `nic_is_usb` lo segnala. È possibile sovrascrivere entrambe le impostazioni con `--wire-ceiling-mbps`.
- **L’ammissione tiene conto del drain, non del confronto tra intero burst e ring.** Un burst simultaneo deve solo rientrare nel *backlog transitorio* = `max(0, Σ per-cam arrival − host drain) × emit_window`, non nell’intero burst. Su un fabric con host veloce / cam lenta (un **PCIe**host 10G + 4× cam da 1 GbE: arrivo ≈ 320 MB/s, drenaggio ≈ 1063 MB/s), l’host scarica più velocemente di quanto le telecamere riescano a riempire, il backlog è ≈ 0, quindi l’emissione simulata a piena risoluzione**viene ammessa**anche se il burst da 25 MB supera l’anello da 13,5 MB. Se si collegano le stesse quattro telecamere a un adattatore**USB** da 10 GbE su Windows, lo scarico è di 200 MB/s, non di 1063 — l’arrivo lo supera, e la perdita si manifesta come frame danneggiati anziché come un frame rate più basso. Su un host da 1 GbE, il limite minimo di DLThr delle telecamere pari a 31,25 MB/s fa sì che l’arrivo superi lo scarico → il sistema **blocca** correttamente (per *questa* classe di blocco, ridurre il ROI o utilizzare un binning ≥ 2). L’ammissione è uno dei **due** gate di connessione — l’altro è il controllo di sovrasottoscrizione aggregata riportato di seguito.
- **Gli fps previsti corrispondono al ciclo della telecamera più lenta.**Nell’emissione simultanea ogni telecamera invia il proprio fotogramma contemporaneamente, quindi l’array opera al ritmo del tempo più lungo tra la lettura del sensore e il tempo di emissione del suo membro più lento (byte del fotogramma ÷ la sua frequenza di emissione: il**collegamento di accesso** della telecamera, 1 GbE ≈ 80 MB/s, oppure la sua quota della larghezza di banda massima dell’host quando questa è inferiore). Ogni esposizione si sovrappone alla lettura del fotogramma precedente, quindi l’esposizione non entra nel ciclo: l’esposizione automatica riempie il periodo determinato dalla frequenza, quindi aumenta il guadagno, e una scena poco illuminata non riduce più la frequenza (nella versione 1.2.0 lo faceva). Solo un’esposizione manuale più lunga del periodo rallenta l’attivazione (vedi [Esposizione su un membro dell’array](#exposure-on-an-array-member)). L’aggiunta di telecamere non rallenta l’array finché le loro quote di banda non si riducono. Nell’emissione seriale (`slip-emit-and-capture`) i tempi di emissione delle telecamere si sommano invece, e l’esposizione di ciascun membro viene considerata nello sfalsamento. Misurato a sensore pieno, 12 bit, su un array di 9 telecamere con un host PCIe 10 GbE: 10,7 fps, oppure 14,0 fps con [trasporto compresso](#packed-12-bit-transport---packed).
- **Dopo un’acquisizione continua, la proiezione indica il valore raggiunto.** Un’acquisizione di 30 s o più a una frequenza automatica registra la frequenza a cui si è stabilizzata per quella dimensione dell’array su questo computer; i brevi tentativi del regolatore di frequenza di raggiungere una velocità maggiore vengono ignorati. Da quel momento in poi, la riga di `array-connect` e il pannello Impostazioni array riportano tale velocità quando è inferiore a quella del modello, indicando il motivo della limitazione: `clock-bound` (l’array ha funzionato alla velocità stabilita, il ciclo di acquisizione in attesa del proprio clock di trigger), `loop-bound` (il ciclo di acquisizione non aveva più tempo a disposizione), `capture-bound` (l’acquisizione è rimasta indietro e ha saltato dei gruppi) oppure `measured` (non è specificato quale). Il valore del modello è riportato in `wire_fps_max`. L’array raggiunge comunque la velocità massima consentita dalla linea.
- **Un array funziona su qualsiasi rete — è il frame rate a determinare i limiti.**Ogni telecamera riceve una quota equa del**limite massimo della linea senza collisioni**(`sustained × sim_emit_factor`), sincronizzata con GevSCPD, e tutte continuano a scattare con lo stesso trigger. Su una connessione lenta la quota è ridotta e la frequenza dei fotogrammi diminuisce di conseguenza: 8 telecamere a 12 bit e piena risoluzione su una rete 1 GbE senza jumbo frame ottengono 52 / 8 = 6,5 MB/s ciascuna, circa**1 fps**(circa 1,3 fps compressi), mentre un host 10G le fa funzionare a 11-14. Se le telecamere non possono essere sincronizzate a una velocità così bassa — collegarsi e leggere l’intervallo GevSCPD di ciascuna telecamera per scoprirlo — l’array passa alla modalità di trasmissione**seriale**(`slip-emit-and-capture`): le telecamere si alternano sulla linea alla velocità massima del collegamento, attivate una dopo l’altra, quindi ogni frame arriva comunque ma i membri**non vengono esposti simultaneamente**. `analyze-array` e l’output di connect indicano la modalità; impostare `--force-tier sim-capture-sim-emit` su “pin” per rifiutare il fallback. La dimensione del fotogramma non influisce su questa condivisione, quindi**il binning e una ROI più piccola aumentano la frequenza dei fotogrammi ma non modificano la capacità dell’array di adattarsi**. Solo un cavo che non è in grado di trasportare l’array nemmeno in serie è**sovraccaricato**— ad esempio, se il numero di telecamere è di gran lunga superiore a quello che un cavo può gestire, o se su di esso stanno trasmettendo molte altre telecamere — e in tal caso il sintomo è la perdita di pacchetti GVSP, non una velocità inferiore: `analyze-array` azzera i valori di fps raggiungibili e stampa `** OVER-SUBSCRIBED**`; la connessione procede con un avviso sonoro e una connessione HTTP con `pin_resolution: true` (modalità rigorosa) **viene rifiutata**. Le soluzioni sono: un numero inferiore di telecamere, l’uso di jumbo frame end-to-end o una scheda di rete più veloce. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` riduce il rifiuto a un avviso sonoro per il lavoro di bench — vedi [Variabili d’ambiente](#environment-variables).

### Stato dell’array — quale sottosistema sta perdendo frame

L’`GET /api/camera/array/<array_id>/capability` di un array connesso contiene un blocco
`health` in tempo reale, rivalutato su una finestra mobile di **10 secondi**. Esso suddivide la perdita di frame
nelle due cause che richiedono soluzioni opposte, anziché riportare un unico tasso di “incompletezza”
che non specifica nessuna delle due:

| Campo | Cosa significa | Quale sottosistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriale) | Il frame **è arrivato ma era strutturalmente errato**— perdita di pacchetti GVSP. |**Rete**: budget di banda, pacing, anello RX della scheda di rete, MTU |
| `never_arrived_rate_pct` (per seriale) | Il frame **non è mai arrivato**— la telecamera non ha scattato, oppure non è stato trasmesso nulla. |**Trigger / sincronizzazione**: cavo M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | La peggiore frequenza di ciascuna telecamera. | — |
| `per_cam_rate_pct` | Frequenza complessiva di immagini incomplete per telecamera (entrambe le cause insieme). | — |
| `stable_for_seconds` | Per quanto tempo ogni telecamera è rimasta al di sotto dello 0,01%. | — |

Al di sopra del 5%, il backend registra una riga `[array-health <id>] WARN` che indica la suddivisione — al
primo superamento della soglia, al cambiamento della fascia di gravità, una volta al minuto finché persiste e una volta quando
la situazione si risolve. La metà corrotta scrive `[gvsp-corrupt <SN>]` nel file di log del backend al
primo evento per telecamera e motivo, poi un riepilogo ogni 60 s; il pannello di log dell’app mostra invece la
riga `[array-health <id>]`. Ogni valutazione viene comunque registrata nel file di log del backend;
i contatori avanzano ad ogni buffer indipendentemente da ciò che viene stampato.

Lo stesso record riporta il numero a cui è collegata l’intera allocazione:

| Campo | Cosa significa |
| --- | --- |
| `wire_ceiling_mbps` | Il budget di banda sostenuto dell’host attualmente in vigore, in MB/s. |
| `wire_ceiling_source` | Da dove proviene quel numero, in parole — ad es. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (su Linux x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) oppure `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` quando `--wire-ceiling-mbps` (o il campo **Wire Budget** dell’interfaccia grafica) lo imposta. |
| `nic_is_usb` | `true` per un adattatore Ethernet USB — si veda il limite USB sopra indicato. |

**Interpretazione:** un valore di `gvsp_corrupt_rate_pct` diverso da zero con `never_arrived_rate_pct` pari a 0
significa che il trigger e la sincronizzazione del cavo sono perfetti e che il 100% della perdita si verifica sul percorso di rete
— ridurre il valore di `--wire-ceiling-mbps` e ricollegare. Il modello inverso indica invece un problema
nel cavo di sincronizzazione o nella linea di trigger.

> **`--target-fps` non è il fattore determinante per i frame danneggiati.** Il pacing di GevSCPD viene impostato
> una sola volta alla connessione, quindi abbassare la frequenza di trigger modifica il ciclo di lavoro e non la
> velocità di trasmissione in burst simultanea. Una riduzione misurata della richiesta di 5× non ha prodotto alcun miglioramento;
> abbassare il limite massimo della linea da 240 a 200 MB/s ha portato lo stesso sistema dal 10,4 %
> di frame corrotti allo 0,00 %.

> **Un array in esecuzione non può ripianificarsi autonomamente sul firmware LATTICE (M3C / M3M).** Non
> è in grado di risolvere autonomamente il problema; scollegare e ricollegare in modo che il selettore del tempo di connessione possa
> ripianificare con la nuova velocità massima.

### Sintomo → soluzione

| Sintomo (Impostazioni array / connessione / `analyze_array_network`) | Causa | Soluzione |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` reimpostato a 32 (in genere dopo un aggiornamento del driver) | Impostare `ReceiveBufferLen`→256, `PendingReceives`→64; riaprire il pannello (riavviare il backend se ha memorizzato nella cache la vecchia dimensione dell’anello) |
| Il riavvio/spegnimento si blocca; in seguito errori WMI `Invalid class`, la scheda di rete non si abilita, le unità USB non vengono rilevate | Vecchio driver Realtek USB 10GbE del 2016 → BSOD `0x9F` → spegnimenti forzati | Aggiornare il driver della scheda a ≥ v10.67 (2026), quindi riapplicare le impostazioni dell’anello di ricezione sopra indicate |
| L’array si connette al sensore al massimo ma con un frame rate basso | La modalità automatica mantiene il sensore al massimo e lascia che il frame rate scenda al livello supportato dalla linea | Scegliere una delle opzioni più veloci elencate nell’output di connessione (`--binning 2` / `4`, oppure un valore `--width`/`--height`), abilitare i frame jumbo o aggiornare il collegamento |
| L’array segnala un fps target corretto ma ne fornisce solo una frazione; `health.gvsp_corrupt_rate_pct` diverso da zero, `never_arrived_rate_pct` 0 | Il budget di banda stimato per l’host sovrastima quella effettivamente sostenuta (tipico su un adattatore Ethernet USB, una corsia PCIe sottile o un fabric condiviso) | Riconnettersi con un valore `--wire-ceiling-mbps` inferiore e ricontrollare lo stato di integrità. **Non** `--target-fps` — Il pacing di GevSCPD è fissato al momento della connessione |
| Linux: un array perde gruppi di frame e riduce la propria frequenza di trasmissione mentre la scheda di rete non segnala alcuna perdita; `lattice network` elenca `socket receive buffers too small` | I buffer di ricezione dei socket del kernel sono impostati sui valori predefiniti, quindi i pacchetti della telecamera vengono persi nel kernel | Aumentare `net.core.rmem_max` / `rmem_default` a 32 MB — vedi [host Linux](#linux-hosts--socket-receive-buffers). Il pacchetto .deb esegue questa operazione al momento dell’installazione |
| Telecamere mancanti dai gruppi pubblicati; `health.never_arrived_rate_pct` diverso da zero, `gvsp_corrupt_rate_pct` 0 | Percorso di attivazione/sincronizzazione — le telecamere non si attivano, non è un problema di rete | Controllare il cavo di sincronizzazione M8 e `--line`; verificare che ogni membro sia armato (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` superato in `analyze-array`, oppure rifiuto di connessione in modalità rigorosa (`pin_resolution`; `array over-subscribes the wire`) | Il cavo non è in grado di trasportare l’array a nessuna velocità di trasmissione raggiunta dalle telecamere, né simultaneamente né a turno — il numero di telecamere è di gran lunga superiore a quello che il cavo può gestire, oppure altre telecamere stanno trasmettendo su di esso | Meno telecamere, frame jumbo end-to-end o una scheda di rete più veloce. **ROI/binning NON saranno d’aiuto** (la condivisione è indipendente dalla dimensione dei frame). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` si attiva in ambiente di test (accetta la perdita di pacchetti) |
| L’array si connette in `slip-emit-and-capture` (seriale) su una rete lenta | Le telecamere non possono essere sincronizzate a una velocità sufficientemente bassa da condividere la linea contemporaneamente, quindi si alternano; i frame non vengono esposti nello stesso istante | I jumbo frame o una scheda di rete più veloce ripristinano l’acquisizione simultanea; `--force-tier sim-capture-sim-emit` insiste su questo (e perde invece dei pacchetti) |

---

## `chloros-cli daq`

Comandi del sensore spettrale. I comandi `pool-*` sono client HTTP "sottili" che gestiscono il sensore tramite il pool persistente del backend, quindi necessitano di un backend in esecuzione (vedere [Avviare il backend](#avviare-il-backend)). Il backend gestisce il trasporto, quindi gli script CLI e SDK condividono un unico handle attivo invece di contendersi la porta seriale. `pool-discover` è l’unico modo per rilevare il MAC BLE di un DAQ-M.

> **`chloros-cli daq --help`** (oppure `-h` o `daq help`) elenca i sottocomandi di `pool-*` (su Linux, insieme ai comandi hardware diretti inclusi nella compilazione), mentre `chloros-cli daq <subcommand> --help` (o `daq help <subcommand>`) fornisce le opzioni relative a ciascuno di essi. Queste opzioni riguardano la connessione, stream, la registrazione di file `.daq` calibrati, lo scambio di profili CAP e l’associazione di un DAQ a una telecamera; per un DAQ-E, inoltre, le impostazioni di rete, il PTP, il riavvio e gli aggiornamenti del firmware. Il DAQ può anche essere controllato da Python con `chloros_sdk.connect_daq_sensor()`, che utilizza lo stesso percorso condiviso.
>
> **Utilizzare i comandi `pool-*` su ogni piattaforma.** Si comportano allo stesso modo su Windows, Linux e Jetson. Su Windows, `daq discover` e `daq list` sono alias di `pool-discover`, così come qualsiasi altro comando `daq` che non sia `pool-*`, termina `1` con `DAQ direct-hardware commands require the full daq package`. Su Linux e Jetson, `daq discover`, `daq list` e gli altri comandi `pool-*` e `daq` non corrispondenti a `daq discover` sono i comandi hardware diretti: aprono il sensore nel processo proprio dell’CLI, al di fuori del pool del backend (quindi non su un sensore gestito dal backend, dalla GUI o da uno script), e lì accettano `daq discover`, `--only` e `--timeout` ma non `--json`. Il presente documento di riferimento descrive esclusivamente i comandi `pool-*`.

### Flusso di lavoro per la prima connessione al sensore DAQ

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Riferimento `pool-*`

| Sottocomando | Scopo |
| --- | --- |
| `daq pool-connect` (smart-detect) | Apre un sensore nel pool di backend. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U su una porta seriale specifica. |
| `daq pool-connect --ble` | DAQ-M su BLE, MAC rilevato automaticamente. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M su BLE con MAC noto (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E su Ethernet con nome host o IP noto (implica `--eth`). |
| `daq pool-connect --eth` | DAQ-E su Ethernet, host rilevato automaticamente (mDNS + fallback ARP; funziona anche con cache ARP vuota su Windows e Linux). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Regola la finestra di integrazione / lo stato AE. |
| `daq pool-connect --no-stream` | Connettiti ma non avviare ancora lo streaming (riprendere con `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profilo di correzione Cap. Il valore predefinito nel backend è `sunshine_cosine`; `none` è un sensore nudo. Un [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) lo ignora (diffusore fisso). **I sensori vengono forniti senza cappuccio, quindi assegnate un nome al cappuccio che avete montato:** il valore predefinito viene registrato come (`cap_id_source=auto_default`) e l’elaborazione emette un avviso al riguardo, poiché un DAQ-U senza cappuccio sottoposto alla correzione per la luce solare registra valori circa 30 volte superiori. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Esegue una scansione di ogni trasporto alla ricerca di sensori a cui è possibile connettersi, senza effettuare alcuna connessione. **Ecco come trovare l’indirizzo MAC BLE di un DAQ-M.** `--timeout` è per trasporto (impostazione predefinita 5 s, 1-20). I sensori già aperti nel pool non vengono elencati — un DAQ-M connesso smette di trasmettersi — quindi usa `pool-list` per quelli. `usb` elenca solo i dispositivi seriali USB: le porte seriali del computer stesso (`COM1`, `/dev/ttyS*` su Linux) non possono essere sensori e vengono escluse. |
| `daq pool-list` | Mostra tutti i sensori nel pool del backend. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Associa una telecamera o un array a un DAQ, come fa la selezione del sensore di luce nell’interfaccia grafica: le sue acquisizioni utilizzano, abbinano e salvano le letture di quel DAQ. `SENSOR` è un ID sensore proveniente da `pool-list` o l'indirizzo su cui è raggiungibile il DAQ. `--clear` (non `SENSOR`) interrompe l’associazione, e le acquisizioni tornano al DAQ più recente. Il binding dura fino al riavvio del backend, oppure fino a quando quella telecamera, quell’array o quel DAQ non vengono disconnessi: una disconnessione (in questo caso, dall’SDK o dall’app) interrompe ogni binding dell’hardware che disconnette, indipendentemente da chi lo abbia impostato, compresi i membri di un array insieme all’array stesso. La telecamera deve essere collegata, presente nel progetto aperto o essere stata aperta in precedenza su questo computer, e l’array deve essere collegato o presente nel progetto aperto; qualsiasi altra seriale o ID viene rifiutato in base al nome e non viene effettuato alcun binding. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Rilascio — chiunque abbia aperto il sensore; in questo modo viene liberato anche un sensore lasciato nel pool da uno script che si è bloccato (eventuali registrazioni in corso su di esso vengono prima completate). Un sensore che non si trova nel pool non ha nulla da rilasciare: visualizza `ID: already released (not in the pool)` e termina con `0`; un rilascio rifiutato o non confermato dal backend termina con `1`. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | I N frame dello spettro più recenti (impostazione predefinita 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Riprendi / metti in pausa lo streaming. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Avvia / interrompi una registrazione .daq. `-o` è impostato di default su `~/Documents/DAQ Live View` sul computer del backend (il percorso relativo si trova nella cartella da cui si esegue il comando); `--duration` (da `-d`, valore predefinito `0` = fino a `--stop`). `--confirm-after S` avvisa quando non è arrivata alcuna lettura entro S secondi dall’avvio (impostazione predefinita 20; 0 disattiva l’avviso); con `--duration`, `--heartbeat S` stampa il conteggio delle letture ogni S secondi e avvisa quando smette di crescere (impostazione predefinita 30; 0 lo disattiva). Una registrazione su un DAQ-E che si interrompe prosegue nello stesso file una volta che il backend si è ricollegato, e `--stop` elenca le lacune ([Un DAQ-E che smette di trasmettere](#a-daq-e-that-stops-delivering)). Ctrl+C interrompe anticipatamente una registrazione `--duration` e lo segnala; senza `--duration` la registrazione continua a funzionare dopo Ctrl+C (esci da `130`) fino a `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Scambia il profilo di correzione del condensatore durante l’esecuzione. È richiesto `--cap-id`; `--cap-id none` rimuove la correzione del condensatore (sensore "nudo"). Un DAQ-E-S mantiene `as_recorded` qualunque sia l’impostazione. |
| `daq pool-enable-ptp --sensor-id ID` | Riattiva il PTP per un DAQ-E il cui PTP è disattivato. L’impostazione viene salvata sull’unità, quindi un’unità lasciata disattivata rimane disattivata — i suoi timestamp non sono sincronizzati via PTP — finché non viene eseguito questo comando. `pool-list` contrassegna tale unità (`!! PTP is switched OFF`) e stampa questo comando per essa. Chloros non riattiva mai un’unità di propria iniziativa, poiché lo stato disattivato può essere intenzionale. |
| `daq pool-info --sensor-id ID [--json]` | Impostazioni proprie di un DAQ-E: il suo firmware (e se questa versione di Chloros ne contiene uno più recente), il suo indirizzo e come lo ha ottenuto (DHCP / link-local / statico), l’impostazione di rete salvata fino a quando il sensore non la esegue (quella salvata con `--no-reboot` inclusa), lo stream all’accensione e il PTP. Su un sensore il cui stream non raggiunge questo computer, legge il sensore su una nuova connessione e lo segnala. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Impostare un DAQ-E su DHCP (`--auto`) o un indirizzo statico (`--mask`, predefinito `255.255.255.0`). Il sensore si riavvia per applicare l’impostazione e si riconnette con le impostazioni precedenti, attendendo fino a `--timeout` secondi (impostazione predefinita 45); `--no-reboot` si limita a salvarla. Vedi [Impostazioni del dispositivo DAQ-E](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Indica se un DAQ-E avvia lo streaming all’accensione senza alcun software collegato (impostazione salvata sul sensore). Chloros avvia lo streaming autonomamente in entrambi i casi. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Riavvia un DAQ-E e lo ricollega con le impostazioni precedenti (attende fino a `--timeout`, valore predefinito 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Esegue il flashing del firmware DAQ-E incluso in questa versione di Chloros su un sensore che esegue una versione precedente, quindi lo ricollega (circa un minuto; non spegnerlo; `--timeout` 45 s predefiniti affinché torni operativo). Un sensore già aggiornato non viene modificato. |

### Un DAQ-E che smette di trasmettere

Un DAQ-E nel pool che non trasmette alcuno spettro per 20 secondi viene ricollegato dal backend. Ciò avviene tramite un ciclo di spegnimento e riaccensione dello switch che alimenta il sensore: il sensore si riavvia con esso e non riprende a trasmettere finché non gli viene richiesto di farlo (a meno che non sia impostato `pool-set-auto-stream --on`).

- **Per prima cosa verifica che si tratti dello stesso sensore.** Interroga l’indirizzo che il sensore aveva in precedenza, senza modificare nulla, per ottenere il nome del sensore, l’indirizzo MAC e il numero di serie. Invia inoltre una richiesta utilizzando il nome `daq-e-<id>.local` del sensore, ma accetta tale risposta solo se proviene dalla scheda di rete (MAC) del sensore stesso. Nessun altro dispositivo che risponde viene ricollegato né gli vengono scritti dati.
- **Si riconnette utilizzando le impostazioni della sessione:** tempo di integrazione, media dei fotogrammi, esposizione automatica e profilo CAP così come erano, non i valori predefiniti all’accensione del sensore. Un sensore che aveva il PTP attivato e che è tornato con il PTP disattivato viene riattivato. Quindi riavvia lo streaming e il ricevitore del computer dedicato.
- **Riprova a intervalli crescenti:** immediatamente, poi dopo 5, 10, 20 e 40 secondi, quindi ogni 60 secondi finché il sensore risulta assente. Ogni tentativo si limita a chiedere al sensore di identificarsi. Un sensore che risponde ma continua a non inviare nulla dopo tre tentativi di riconnnessione viene lasciato stare e segnalato (`auto-reconnect GAVE UP`). `pool-stream --start` riprova.
- **`pool-list` mostra lo stato attuale.** Mentre il sensore rimane silenzioso, la riga `STREAM FAULT` indica lo stato del processo di riconnnessione. La riga scompare con il primo spettro, dopodiché il sensore riporta `reconnected automatically N time(s)`.
- **La registrazione prosegue con lo stesso `.daq`.** Il file contrassegna ogni interruzione con due righe: `event_type` 5 nell’ultimo spettro prima dell’interruzione e 6 nel primo dopo di essa, ciascuna contenente solo un timestamp. `pool-record --duration` stampa `!! STREAM LOST …` mentre il sensore è assente e `stream resumed after …` quando ritorna. `pool-record --stop` elenca ogni interruzione e avvisa, come fa per un file vuoto (una registrazione che non contiene alcuna lettura esce da `1`).
- **Una registrazione mantiene una sola calibrazione.** Se il sensore torna con una calibrazione diversa mentre una registrazione è in corso, il resto della registrazione mantiene quella con cui è iniziata, e l’avviso di arresto lo segnala. `pool-disconnect` + `pool-connect` adotta quella nuova.
- **I tuoi comandi hanno la precedenza.** Un sensore che hai disattivato con `pool-disconnect`, o messo in pausa con `pool-stream --stop`, non viene mai ricollegato, anche se una ricollegazione è già in corso.
- **`pool-info` su un sensore silenzioso** legge il sensore su una nuova connessione e aggiunge `!! stream NOT reaching this computer`. Un sensore che non risponde viene segnalato immediatamente. `pool-list` non mostra più lo stato PTP per un sensore che ha smesso di rispondere.
- DAQ-U e DAQ-M non sono contemplati. Un DAQ-M si riconnette autonomamente tramite Bluetooth, mentre un DAQ-U scollegato dall’USB richiede nuovamente `pool-connect`.

### DAQ-E-S — diffusore fisso e sensore di inclinazione

Il **DAQ-E-S** è un DAQ-E con due differenze rilevanti per il software. Tutto il resto — individuazione, connessione, PTP, registrazione, le impostazioni del dispositivo riportate di seguito — funziona esattamente come per un DAQ-E.**Il suo diffusore è permanente, quindi non richiede alcuna correzione del cappuccio.** Il diffusore in stile Sunshine è integrato ed era presente sull’unità al momento della misurazione della calibrazione di fabbrica, pertanto tale calibrazione indica già l’irraggiamento discendente effettivo. Chloros imposta il suo cappuccio su `as_recorded` (nessun profilo) indipendentemente da quanto richiesto: un limite specificato per esso (`sunshine_cosine`, `fov_*`, `none`) viene ignorato con un messaggio di avviso, e le sue registrazioni memorizzano `cap_id_source=model`, quindi l’avviso relativo al limite presunto non viene mai attivato per esso. Non applicare nemmeno manualmente un limite ai suoi dati: `sunshine_cosine` in posizione orizzontale darebbe una lettura circa 11 volte superiore.

**È dotato di un accelerometro a 3 assi.** La risposta coseno del diffusore solare è valida solo quando è in posizione orizzontale, quindi un sensore inclinato riporta un’irraggianza errata — e la riflettanza eredita l’errore — senza altri sintomi. Ogni lettura DAQ-E-S riporta quindi il proprio assetto:

| Campo | Significato |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Angoli in gradi. `None` quando il firmware contrassegna l’angolo come indefinito (caduta libera, un asse inattivo). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Accelerazione in milli-g. Un valore di `mag_mg` molto lontano da ~1000 indica che l’unità era in movimento e che gli angoli non sono significativi per quel campione. |
| `cal_applied` | Impostato: gli assi e gli angoli sono nel sistema di riferimento del diffusore (viene applicata la calibrazione di montaggio dell’unità). Deselezionato: sistema di riferimento grezzo della scheda, che può mostrare un’inclinazione fino a ~2,3° che non è reale. |
| `sample_age_ms` | Tempo trascorso dal campionamento dell’accelerometro al momento dell’invio dello spettro. |
| `temp_raw` | **Andamento** della temperatura del die non calibrato — solo differenze, mai una lettura del termometro. |

`daq pool-latest --json` li restituisce sotto la chiave `imu` di ciascun frame (`null` su un DAQ-U, DAQ-M o DAQ-E standard). Un `.daq` registrato da un DAQ-E-S memorizza gli stessi valori nelle sue colonne `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) accanto a ogni spettro, in modo che il lettore possa distinguere una nuvola da un treppiede urtato. Su tutti gli altri modelli quelle colonne sono vuote.

Il flusso di assetto viene riportato separatamente: `daq pool-list` stampa `!! ATTITUDE STREAM FAULT (spectra unaffected)` mentre è inattivo e indica da quanti guasti si è ripreso in seguito. Un guasto in quel punto non causa mai un’interruzione nella registrazione spettrale. `daq pool-discover` elenca l’unità come `DAQ-E-S` (come `DAQ-E` se non risponde al controllo di identità entro circa 2 s).

### Impostazioni del dispositivo DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` senza `--gateway` non salva alcun gateway** (qualsiasi gateway impostato in precedenza viene eliminato), il che è corretto per una rete di telecamere senza router.
- **È necessario che su questa rete sia presente un indirizzo statico.** Chloros rifiuta qualsiasi indirizzo la cui sottorete non includa l’indirizzo di questo computer sulla rete del sensore: dopo il riavvio, nulla potrebbe raggiungere il sensore per ripristinare l’impostazione precedente. Impostare `--force` solo quando a questo computer verrà assegnato un nuovo indirizzo o verrà instradato verso la nuova sottorete.
- **Il riavvio viene rifiutato mentre il sensore sta registrando.** Interrompere prima la registrazione (`pool-record --stop`). Una registrazione avviata immediatamente prima di un riavvio viene chiusa da quest’ultimo, e `pool-set-network`, `pool-reboot` e `pool-update-firmware` terminano con una riga `Warning:` che indica il nome del file di registrazione (anche quando il sensore poi non torna).
- **Dopo un riavvio, Chloros riconnette solo il sensore stesso.** Attende nella posizione in cui si trovava il sensore — e dopo `--auto` anche sotto il suo nome `daq-e-<id>.local` — una risposta contenente il nome del sensore, nome host e modello, dall’adattatore di rete (MAC) che aveva, e lo ricollega all’indirizzo che ha risposto. Una risposta con quel nome viene accettata solo con quel MAC; pertanto, da un computer che raggiunge il sensore tramite un router, o da WSL2 nella sua configurazione NAT predefinita, un sensore che il DHCP ha spostato a un nuovo indirizzo non viene ricollegato: il comando lo segnala e visualizza il comando `pool-connect --eth-host <its address> …` che lo riporta in linea con le impostazioni di limite massimo ed esposizione che aveva.
- **`pool-update-firmware` rifiuta un sensore di cui non riesce a leggere la versione del firmware.** Un sensore che impiega più tempo del previsto per rispondere dopo il flash non ha subito un errore: una volta che risponde, viene ricollegato con le impostazioni del nuovo firmware già inserite, e il comando avverte che la risposta è stata lenta.
- **Un aggiornamento del firmware richiede una connessione TCP in entrata.** Il sensore si riconnette a questo computer per raccogliere l’immagine. Chloros è in ascolto sull’adattatore che instrada verso il sensore (su ogni adattatore solo quando non è in grado di individuarne uno specifico) e serve solo l’indirizzo del sensore stesso; qualsiasi altra connessione viene chiusa senza essere servita. Su Windows, la regola firewall `Chloros Backend (TCP-In)` del programma di installazione ne consente l’accesso (vedi [Windows (.exe)](#windows-exe)); un firewall che la blocca fa fallire l’aggiornamento con codice di errore `… device never connected back on TCP <port> …`.
- **In WSL2, utilizzare la rete in mirroring** (`networkingMode=mirrored` in `%UserProfile%\.wslconfig`). Nella modalità NAT predefinita, l’indirizzo di questo computer sulla rete del sensore è quello di WSL stesso, quindi ogni indirizzo statico viene rifiutato (ad eccezione di `--force`), il flusso in tempo reale non raggiunge WSL e l’aggiornamento del firmware non può essere completato, poiché il sensore si riconnette al computer per ricevere l’immagine.
- **L’ID del sensore e il nome host non cambiano.** Qualunque sia il suo indirizzo, un DAQ-E rimane comunque `daq-e-<id>` e `daq-e-<id>.local`, quindi `--eth-host daq-e-<id>.local` continua a individuarlo.
- **L’interfaccia grafica presenta le stesse impostazioni**: scheda «Sensori di luce», impostazioni del sensore (icona a forma di ingranaggio), «Rete». L’interfaccia grafica mantiene la propria connessione con un sensore, quindi se si modifica un sensore con cui l’interfaccia grafica ha una connessione aperta direttamente dall’interfaccia grafica: un riavvio dalla CLI termina la sessione dell’interfaccia grafica con esso.
- Gli equivalenti nell’SDK sono `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` e `update_firmware()`.

---

## `chloros-cli project`

Aprire, connettersi e gestire un progetto Chloros salvato (una cartella contenente `cameras.json` + `sensors.json` + `project.json`). Tutto passa attraverso il backend, quindi l’interfaccia grafica (GUI) e l’CLI producono lo stesso stato hardware.

### Riferimento ai sottocomandi

| Sottocomando | Scopo |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Stampa il manifesto dei dispositivi del progetto (telecamere, array, sensori), lo legge dai relativi file e verifica i file sorgente (vedi [Un progetto spostato](#a-moved-project---relink)). Non apre il progetto nel backend: il progetto lì aperto, così come i suoi dispositivi e le registrazioni, rimangono invariati. Esce da `1` quando non è possibile trovare nessuno dei file sorgente del progetto. |
| `project devices PATH [--reconnect]` | Elenca o riesegue il rilevamento. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Connette ogni telecamera / array / sensore salvato. Ogni array si connette come fa `lattice array-connect`: fino a 600 s, con i passaggi stampati su stderr (`[array <id>]    <s> s: <step>`; stdout contiene il report in formato JSON), e un array non collegato entro tale tempo viene annullato — le sue telecamere vengono riportate allo stato in cui sono state trovate — e segnalato sotto `errors`, senza mai rimanere collegato. `--packed` imposta il trasporto compresso a 12 bit di ogni array (impostazione predefinita `auto`; un progetto non salva impostazioni specifiche per ogni array; vedi [Trasporto compresso a 12 bit](#packed-12-bit-transport---packed)). Ogni sensore di luce viene aperto nel pool DAQ del backend al proprio indirizzo salvato con il tempo di integrazione, la media dei fotogrammi, l’esposizione automatica e il limite salvati: un sensore per cui non è stato selezionato alcun limite massimo riceve `sunshine_cosine`, uno salvato "nudo" (`none`) rimane "nudo", mentre un DAQ-E-S non accetta mai un limite massimo. Il pannello DAQ della GUI non fa parte del pool: chiudere prima un sensore da lì. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Singola acquisizione da una telecamera o da un array con nome (`--prefix` predefinito `capture`). `--orient` (array): l’array salva i dati grezzi, che rimangono in formato nativo — ciò indica che i prodotti che `process` ne ricava devono essere orientati, come `array-capture --orient`. Impostazione predefinita: nativo. `project burst` / `stream` e le voci `capture` / `burst` / `stream` (`orient: true`) utilizzano la stessa opzione. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Burst di N fotogrammi da una telecamera o da un array specificato (`-n/--count` valore predefinito 5; `-i/--interval` secondi tra un fotogramma e l’altro, valore predefinito 0). Le raffiche da array eliminano i duplicati dei gruppi sincronizzati ripetuti (watchdog di obsolescenza), in modo che un array a ciclo parziale non possa restituire N copie di un singolo fotogramma; stampa i risultati per ogni iterazione. Una cattura che fallisce a metà interrompe la sequenza e termina `1`, indicando quanti fotogrammi sono stati salvati prima di tale interruzione. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Acquisizione su disco in ciclo: `--fps F` acquisisce un secondo (impostazione predefinita 3), un fotogramma per acquisizione da una telecamera, un gruppo di fotogrammi sincronizzati da un array (con lo stesso watchdog di obsolescenza di `project burst`), finché `-n N` non acquisisce (impostazione predefinita `0` = fino a quando non si preme Ctrl+C, il che visualizza quanti sono stati salvati e termina `0`, oppure `1` quando non ne è stato salvato nessuno o si è verificato un errore nel file; un Ctrl+C prima che `-n N` completi l’acquisizione fa uscire `130`). Una cattura che fallisce a metà interrompe il flusso e termina `1`, indicando quanti fotogrammi sono stati salvati prima dell'interruzione. `--poll-interval` è deprecato e ignorato: viene comunque accettato in modo che gli script più vecchi funzionino, con una nota su stderr. |
| `project sensor read PATH NAME [--json]` | Ultimo frame dello spettro. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Registra un file .daq per `--seconds` (impostazione predefinita 10). Una registrazione priva di letture viene segnalata su stderr e termina `1`. |
| `project run PATH RECIPE.yaml` | Esegue una ricetta di acquisizione YAML/JSON. `--dry-run` esegue la convalida senza eseguire l’operazione. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Calcola l’allineamento per un array — vedere [la tabella dei flag qui sotto](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Visualizza il profilo di allineamento corrente. |
| `project align clear PATH NAME` | Elimina il profilo memorizzato nella cache. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Modifica leggermente la trasformazione di uno slave (impostazioni predefinite: `0`, `0`, `0`, `1.0`: nessuna modifica). |
| `project align export PATH NAME --to FILE` | Salva il profilo in formato JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Carica un profilo salvato. |

#### Opzioni `project align calibrate`

| Flag | Predefinito | Descrizione |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metodo di allineamento. **Queste denominazioni differiscono da `lattice align-calibrate`**, che accetta le forme abbreviate `orb` / `akaze` / `phase`; i due comandi non sono intercambiabili con questo flag. |
| `--model {translation, rigid, affine, homography}` | `affine` | Trasforma il modello per adattarlo. |
| `--frames N` | `1` | I fotogrammi sincronizzati vengono allineati alla media. |
| `--reference SN` | la master | Numero di serie della telecamera di riferimento; tutti gli altri membri vengono distorti su di essa. |
| `--max-features N` | `5000` | Limite del numero di caratteristiche ORB. |
| `--ratio-threshold F` | `0.75` | Test del rapporto di Lowe. |
| `--ransac-threshold-px F` | `3.0` | Soglia degli inlier di RANSAC. |
| `--min-matches N` | `15` | **Soglia di qualità** — rifiuta la soluzione se il numero di corrispondenze valide è inferiore a questo valore. |
| `--max-reproj-err-px F` | `4.0` | **Soglia di qualità** — rifiuta la soluzione se l’errore di riproiezione RMS supera questo valore. |
| `--checkerboard RxC` | — | Geometria della scheda per `--method checkerboard`, ad es. `9x6`. |
| `--name PROFILE` | vuoto | Nome del profilo incorporato nel file JSON salvato. **Non è il nome dell’array** — quello è l’`NAME` posizionale. |

I due controlli di qualità sono il motivo per cui una calibrazione può riuscire nella risoluzione e tuttavia
rifiutarsi di salvare: un profilo che fallisce uno dei due controlli causerebbe silenziosamente un errato allineamento di ogni
acquisizione successiva, quindi viene rifiutato anziché salvato.

### Un progetto spostato (`--relink`)

Il file `project.json` di un progetto assegna a ciascun file sorgente — ogni immagine e registrazione del sensore di luce
— il proprio percorso completo. Se si sposta la cartella del progetto, o l’intera cartella dei progetti,
su un’altra unità, quei percorsi continueranno a indicare la vecchia posizione.

L’apertura di un progetto corregge autonomamente ciò che è possibile: un file che si trovava all’interno della cartella del progetto
viene cercato nella cartella in cui si trova ora il progetto, e ogni percorso trovato lì
viene reindirizzato e salvato (`project.json` viene prima salvato come
`project.json.before-relink-<date>-<time>`). Ciò avviene ad ogni apertura — dell’app,
`chloros-cli project open`, `chloros-cli process` in un progetto esistente, dell’SDK e di `chloros-mcp`. Un percorso viene reindirizzato solo verso un file che si trova effettivamente lì.

I file che si trovavano al di fuori della cartella del progetto — una sessione hub, un’importazione tramite “Aggiungi cartella” —
non possono essere individuati in questo modo. Indicare il nome della cartella in cui si trovano ora:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` prende come riferimento la cartella in cui sono stati spostati i file, o qualsiasi cartella di livello superiore: la cartella di ogni
file mancante viene cercata al suo interno in base ai nomi delle cartelle finali
(`Hub Projects/<session>/raw/<serial>`, poi `<session>/raw/<serial>`, ...), e i file di ciascuna
i file di ciascuna cartella vengono prelevati dalla cartella in cui se ne trovano di più. `project open` visualizza
quali percorsi sono stati modificati, quanti file sorgente sono stati trovati e le cartelle in cui si trovavano gli altri
file. Alcuni ancora mancanti: un avviso, uscita da `0` — il progetto si apre con il resto. Nessuno
trovato: un errore che indica tali cartelle e `--relink`, uscita da `1`; anche il backend
si rifiuta di aprire un progetto di questo tipo per `process` o `project devices --reconnect` (409 HTTP `PROJECT_SOURCES_MISSING`).

### Esempi

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL delle ricette

`project run RECIPE.yaml` accetta un file YAML o JSON che descrive una sequenza di azioni:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Azioni supportate: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` utilizza le chiavi `cameras.json`, `settings`, `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (i nomi precedenti `exposure_time`, `exposure_auto`, `gain`, `gain_auto` sono accettati come alias). Un’esposizione, un guadagno o un valore target impostati in questo modo vengono considerati come propri: il controllo automatico dell’esposizione di Chloros (compresa l’esposizione misurata dal DAQ) non li modifica. Un’esposizione manuale lascia il guadagno com’era: in modalità automatica, la fotocamera continua a regolare il guadagno (su tutta la sua gamma) verso la luminosità target finché non si imposta `gain_db` o `auto_gain: "Off"`. Un successivo comando `auto_exposure: "Continuous"` restituisce l’esposizione all’automazione, mentre `target_brightness_auto: true` restituisce un valore di riferimento mantenuto al sistema automatico di Chloros. L’azione `burst` richiede `name` (obbligatorio), `count` (impostazione predefinita 5), `interval` (secondi, valore predefinito 0), `output`, `format`, e `settings` (stessa configurazione delle impostazioni per telecamera di `apply`); le raffiche in array utilizzano lo stesso watchdog del gruppo appena sincronizzato di `project burst`.

A `capture`, `burst` o `stream`, il valore predefinito di `output` è `output`. Un `output` relativo — quella predefinita inclusa — viene risolto in base alla cartella da cui si esegue `chloros-cli project run`, non alla cartella del file di ricetta o alla cartella del progetto (nella versione 1.2.0 è stato inserito nella cartella di installazione del backend).

Eseguirlo:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Ogni esecuzione viene prima verificata rispetto al progetto.**Ogni azione deve avere come chiave esattamente uno dei verbi sopra indicati — `- capture: {name: FrontLeft}`, non `- action: capture` o `- type: capture` — con un `name`, il progetto `cameras.json` / `sensors.json` dispone di una telecamera o di un array per `apply` / `capture` / `burst` / `stream` e un sensore di luce per `sensor`; un `stream` richiede che `count` > 0 e un `wait` una durata. Una ricetta che presenta uno qualsiasi di questi errori**non** esegue alcuna azione (`[recipe] error: …`, `[recipe] not run: …`, uscita `1`), nemmeno con `stop_on_error: false`. I tasti che un passaggio non utilizza sono un `[recipe] warning:`, così come ogni array o sensore di luce controllato dalla ricetta che al momento non è collegato. Una ricetta non collega mai i dispositivi: eseguire prima `chloros-cli project connect PATH`. `--dry-run` stampa la ricetta, poi gli stessi errori e avvisi e `[recipe] valid: …` / `NOT valid: …`, non esegue nulla e termina con `1` quando la ricetta presenta un errore.

---

## Variabili d’ambiente

| Variabile | Effetto |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Sovrascrive l’URL del backend (predefinito `http://127.0.0.1:5000`) — **accettata solo dalle famiglie di comandi `lattice`, `project` e `daq pool-*`** (`lattice network-analysis` accetta invece `--backend-url`). I comandi principali (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) utilizzano `http://127.0.0.1:<port>` e ignorano questa variabile (il valore IPv4 letterale aggira la penalità di Windows `localhost`→`::1` di circa 2 secondi per richiesta), quindi puntano sempre alla macchina locale. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Impostato nell’ambiente del backend. `1` riduce la severità del rifiuto per sovrasottoscrizione — una connessione array con `pin_resolution`, o un [comando transitorio multi-telecamera](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — a un avviso forte, accettando la perdita di pacchetti GVSP. Solo a scopo di test — vedere [Modello fps e burst dell’array](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Impostazione nell’ambiente del backend: `on` o `off` imposta come predefinito il [trasporto compresso a 12 bit](#packed-12-bit-transport---packed) per ogni array (impostazione predefinita `auto`). Un'impostazione esplicita `--packed` ha comunque la precedenza. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (oppure `true` / `yes`) disattiva il [controllo automatico degli aggiornamenti](#chloros-cli-update) della CLI di Linux all’avvio. |
| `CHLOROS_CLI_MODE` | Impostato dalla stessa CLI; indica al backend di abilitare l’elaborazione parallela. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` salta il test di fallback GVSP (solo risultati ICMP). **Questo disattiva il jumbo, non si limita a silenziare il log** — la telecamera risponde ai ping DF solo fino a 1500 su ogni percorso, quindi questa sonda è l’unica in grado di rilevare il jumbo. Risparmia circa 1 s per telecamera per connessione; costa circa 1,45 volte il limite massimo della linea se la rete *avesse potuto* trasportare pacchetti jumbo. L’SDK avvisa quando lo si imposta. |
| `CHLOROS_GVSP_SESSION_REUSE` | Impostazione nell’ambiente del backend: `0` misura la dimensione del pacchetto ad ogni connessione invece che una volta per telecamera per sessione del backend. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fissa la dimensione dei pacchetti GVSP a N byte; salta completamente la fase di rilevamento. È preferibile utilizzare l’impostazione per singolo comando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) piuttosto che impostarla in modo permanente: una dimensione fissa impedisce l’adattamento alla rete a monte e impostare 9000 su un percorso che non supporta i pacchetti jumbo fa sì che **ogni** acquisizione vada in timeout con `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Impostare nell’ambiente del backend: il numero massimo di secondi per un [comando transitorio multi-telecamera](#multi-camera-transient-sessions), o un `lattice capture` di una telecamera che non mantiene nulla, impiega per impostare l’esposizione di ciascuna telecamera a partire dalla scena precedente prima di conservare un fotogramma (impostazione predefinita `5`). `0` disattiva questa funzione: l'esposizione automatica di ciascuna telecamera viene quindi applicata a partire dal primo fotogramma, a partire dal proprio limite di esposizione. |
| `CHLOROS_LOG_DIR` | Impostazione nell’ambiente del backend: la cartella in cui viene salvato il log dettagliato. Impostazione predefinita `%LOCALAPPDATA%\Chloros\logs` su Windows, `~/.cache/chloros/logs` su Linux (`$XDG_CACHE_HOME/chloros/logs` quando è impostato). Il servizio root del pacchetto .deb imposta `/var/log/chloros`, leggibile dal gruppo `adm` — vedi [Esecuzione del backend come servizio](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Dove vengono salvati i file temporanei. L’CLI utilizza automaticamente `/mnt/ssd/tmp` se presente. |

---

## Codici di uscita

| Codice | Significato |
| --- | --- |
| `0` | Operazione riuscita. |
| `1` | Errore generico (la maggior parte degli errori dei sottocomandi). Un comando di acquisizione termina inoltre con il codice `1` quando la scrittura di uno qualsiasi dei file richiesti non è andata a buon fine, anche se gli altri sono stati salvati, e con il codice `array-burst` / `array-record` termina con codice `1` quando il registratore segnala un errore — vedi [File la cui scrittura non va a buon fine](#files-that-fail-to-write). `process` termina inoltre l'esecuzione di `1` quando non è stato possibile verificare le esportazioni di un'esecuzione o quando ha perso un'immagine, sebbene gli altri prodotti siano stati scritti. |
| `2` | Errore di argomento (un flag sconosciuto o mancante; vale anche per `time-sync set-priority` senza alcuna priorità). |
| `130` | Interrotto da Ctrl+C: qualsiasi comando che interrompe l'esecuzione — `lattice`, `project`, `daq` e `process` (`process` dopo che il backend ha interrotto l'esecuzione, vedi [Note su `process`](#notes)). Un comando il cui arresto normale è Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` e `project stream` con `-n 0` — termina in base a ciò che ha effettivamente eseguito: `0` quando tutto è stato scritto, `1` quando qualcosa è andato storto. |
| `143` / `129` | Interrotto da SIGTERM / SIGHUP (128 + il segnale), dopo l’esecuzione della pulizia del comando: un processo in esecuzione è stato interrotto e le relative telecamere sono state ripristinate, un’elaborazione in background è stata interrotta. Anche una registrazione `lattice continuous` o di altro tipo, il cui arresto normale avviene con Ctrl+C, termina con `143`, non con `0`. |

`lattice network-analysis` termina `1` per i verdetti `needs_force_slip` e `error`, mentre `selftest` termina `1` a meno che tutti e sette i controlli non abbiano esito positivo.

---

## Suggerimenti per la risoluzione dei problemi

- **"Accesso richiesto"** → Eseguire `chloros-cli login EMAIL PASSWORD` una volta su questo computer.
- **"La cartella del progetto è troppo profonda per questo computer"** → I percorsi lunghi di Windows sono disattivati su questa macchina (impostazione predefinita di Windows), quindi nessun percorso può superare i 260 caratteri e i prodotti si trovano fino a circa 130 caratteri al di sotto della cartella del progetto. Non è stata elaborata alcuna operazione. Il messaggio indica la cartella del progetto più lunga che rientra nei limiti: utilizzarne una più breve (`-o` o `chloros-cli set-project-folder`), oppure abilitare i percorsi lunghi di Windows e riavviare Chloros. Il programma di installazione di Windows offre questa opzione come "Abilita percorsi lunghi di Windows", selezionata per impostazione predefinita; in alternativa, utilizzare i Criteri di gruppo: *Configurazione computer > Modelli amministrativi > Sistema > File system > Abilita percorsi lunghi Win32*.
- **"backend unreachable"** → Avvia l’app desktop Chloros (Windows) oppure avvia manualmente il backend: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` su Windows (nella cartella `backend` della cartella di installazione; non si trova in `PATH`), `chloros-backend` su Linux. Oppure controllare `CHLOROS_BACKEND_URL` se in remoto.
- **I comandi `lattice` falliscono con il messaggio "Driver della telecamera LATTICE non trovati"** → Il backend non è riuscito a caricare il runtime della telecamera LATTICE con cui è stato installato. Ogni comando `lattice` viene eseguito sul backend; la CLI stessa non contiene driver per le telecamere. Reinstallare Chloros: il programma di installazione per Windows e il file .deb installano il runtime accanto al backend.
- **"Array connect" / "Array Settings" mostra "FRAMES WILL DROP" o "Reduce ROI to enable"** → L'anello di ricezione della scheda di rete (NIC) dell'host è troppo piccolo (di solito viene reimpostato a 32 dopo un aggiornamento del driver della scheda di rete). Vedere [Configurazione e ottimizzazione della scheda di rete host](#host-nic-setup--tuning-lattice-arrays) — impostare `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Il sistema si blocca al riavvio/spegnimento, quindi WMI `Invalid class` / la scheda di rete non si abilita / le unità USB non vengono rilevate** → Driver obsoleto dell’adattatore USB 10GbE che causa l’errore `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Aggiornare il driver dell’adattatore — vedere [Configurazione e ottimizzazione della scheda di rete host](#host-nic-setup--tuning-lattice-arrays).
- **Linux: le acquisizioni appartengono a root, quindi non è possibile eliminarle e il log del backend non si trova nella tua `~/.cache`** → Il backend viene eseguito come root tramite `chloros-backend.service`. Passare a `chloros-backend@<user>.service`, che lo esegue come utente locale — vedere [Esecuzione del backend come servizio](#running-the-backend-as-a-service-headless). Fino a quel momento, il log del servizio root si trova in `/var/log/chloros` (leggibile dal gruppo `adm`).
- **Linux: un'unità backend è `failed` con il messaggio "Un'altra istanza è già in esecuzione" (codice di uscita 98)** → Un'altra unità backend occupa già la porta: ne può essere in esecuzione solo una alla volta. Se sono abilitate due unità backend, mantenerne una sola: `sudo systemctl disable --now <unit>` per quella che non si utilizza — vedere [Esecuzione del backend come servizio](#running-the-backend-as-a-service-headless). Se è abilitata solo questa unità, un backend avviato al di fuori di systemd (tramite `chloros-backend`, `chloros-cli` o uno script SDK) la occupa: arrestare quel backend, quindi eseguire `sudo systemctl start <unit>`.
- **Avviso swap Jetson** → Aggiungere uno swap su file; la CLI visualizza i comandi esatti `fallocate` / `swapon`.
- **"backend non raggiungibile" da un comando `lattice`, `project` o `daq`** → nessun backend è in esecuzione: quei comandi non ne avviano mai uno. Eseguire `chloros-cli backend status` e consultare [Avvio del backend](#start-the-backend).
- **"`daq net ...` non è disponibile in Chloros"** (Linux e Jetson) → Alcune impostazioni proprie del DAQ-E sono riservate al servizio MAPIR. Le impostazioni di rete, i riavvii e gli aggiornamenti del firmware sono gestiti dai comandi `pool-set-network`, `pool-reboot` e `pool-update-firmware`; per il resto, contattare l’assistenza MAPIR.
- **"I comandi hardware diretti del DAQ richiedono il pacchetto daq completo"** (Windows) → Utilizzare `pool-*`, che controlla lo stesso sensore tramite il backend, oppure `chloros_sdk.connect_daq_sensor()` da Python.

---

## Vedi anche

- [Riferimento SDK Python](sdk-reference.md) — equivalente programmatico di ogni comando CLI.
- [Sensori di luce DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) nel manuale di Chloros — i sensori, i loro limiti e l’intervallo calibrato, la registrazione, il collegamento in rete DAQ-E.
- [Telecamere LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) nel manuale di Chloros — connessione, impostazioni delle telecamere, modalità di acquisizione, array.
- Documentazione online: `https://mapir.gitbook.io/chloros/cli`
