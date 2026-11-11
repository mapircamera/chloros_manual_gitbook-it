# Guida di riferimento all&#x27;SDK Chloros Python

**Versione:**

1.3.0**Generato:**29/07/2026 19:19 ·**Aggiornato:** 06/11/2026**Pacchetto:** `chloros-sdk` (PyPI)**Destinatari:** Ottimizzato per l’utilizzo con modelli di linguaggio di grandi dimensioni (LLM); leggibile dall’uomo.**Ambito:** Tutte le classi pubbliche, le funzioni e gli helper esposti da `import chloros_sdk`, con esempi copiabili e incollabili che coprono l’elaborazione delle immagini, il controllo di una singola telecamera, gli array sincronizzati, sensori DAQ e automazione dei progetti, oltre al server `chloros-mcp` per assistenti IA.

Se ti interessano solo i punti salienti, vai a:
- [Installazione e guida rapida](#installazione)
- [Smart-Connect per array LATTICE](#smart-connect-for-lattice-cameras)
- [Sessioni dei sensori DAQ](#daq-sensor-sessions)
- [Utilità per le telecamere](#utility-per-le-telecamere)
- [Array transitori, profili di allineamento e indici](#array-transitori-profili-di-allineamento--indici)
- [Cartella del progetto e ricette](#cartella-del-progetto--ricette)
- [Automazione del progetto](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Fotogrammi in tempo reale nella propria pipeline](#live-frames-in-your-own-pipeline)
- [Server MCP (assistenti AI)](#mcp-server-ai-assistants)

> **Modifiche nella versione 1.3.0** (cosa rileva uno script esistente della versione 1.2.0).
>
> **Modifiche che uno script 1.2.0 potrebbe dover gestire:**
>
> - **Ora viene generato un errore se il backend non conferma una versione.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` su un array aperto dalla sessione, `DAQSensorSession.disconnect()` / `force_disconnect()` su un sensore da essa aperto, mentre `ChlorosProject.disconnect_all()` genera l’errore `ChlorosConnectError` quando il backend rifiuta il rilascio o non lo conferma in tempo, poiché il dispositivo potrebbe essere ancora occupato. Nella versione 1.2.0 tornavano silenziosamente. L’uscita da un blocco `with` continua a non generare mai un evento.
> - **Un dispositivo preso in prestito non viene mai smantellato.** `with attach_array(...)` e un `disconnect()` esplicito su di esso lasciano l’array in esecuzione (nella versione 1.2.0 l’uscita dal blocco lo smantellava, anche quando era di proprietà della GUI); chiama `force_disconnect()` quando intendi terminarlo. Allo stesso modo, un `connect_daq_sensor()` che ha trovato il sensore già nel pool, incluso lo smart-detect con flag zero, lo lascia aperto su `disconnect()` e all’uscita dal proprio blocco `with`.
> - **I formati PNG e JPEG accettano solo i prodotti `display` e quelli indicizzati.** `capture()` con `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) e un livello `radiance` o `reflectance`, oppure `processing="all"`, genera un errore `ValueError` prima di qualsiasi richiesta; un `raw` / `debayered` a 12 bit viene rifiutato dal backend. Vedi [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` apre quel sensore o genera** un errore `ChlorosConnectError` elencando ciò che ha rilevato; non ricorre mai a un altro sensore. Vedi [Trasporto fissato](#pinned-transport).
> - **`frame_stream()` genera un&#x27;eccezione invece di rimanere in silenzio:** `FrameFetchError`, in caso di rifiuto, al tick successivo ripeterebbe (una lettura della riflettanza senza alcun DAQ incluso), dopo 60 s senza frame e dopo 60 s senza un nuovo trigger. Vedi [Un trigger per ogni lettura](#one-trigger-per-read--group_id).
> - **Un timeout di lettura o una connessione interrotta genera l’errore `ChlorosConnectError`** da ogni chiamata smart-connect e `analyze_array_network()`, dove la versione 1.2.0 generava l’errore di timeout o di connessione sottostante.
> - **`connect_array()` attende per 600 s per impostazione predefinita (`timeout`, 180 s nella versione 1.2.0), e una connessione che non riesce a stabilire viene annullata.** La chiamata invia il proprio `timeout` al backend, che riporta ogni telecamera allo stato in cui è stata rilevata e non effettua alcuna connessione se la connessione non viene completata in tempo; la chiamata genera l’errore `ChlorosConnectError` (504 HTTP `CLIENT_DEADLINE_PASSED`). Nella versione 1.2.0, una connessione avviata subito dopo l’accensione delle telecamere poteva protrarsi oltre la durata della chiamata e connettere l’array dopo che questa aveva generato l’eccezione. `connect_all()` attende ogni array salvato allo stesso modo (120 s nella versione 1.2.0). Vedi [`connect_array()` Signature](#connect_array-signature).
> - **`connect_array()` mantiene il sensore completo.** In assenza di `width` / `height` / `binning`, ogni telecamera è impostata sul sensore completo e la frequenza dei fotogrammi scende a quella fornita dall’host e dalla rete; nella versione 1.2.0 ha ridotto o raggruppato i fotogrammi per mantenere alta la frequenza. Vedi [Matrice sincronizzata](#synchronized-array--arraysession-smart-prep).
> - **Un formato pixel Bayer su una telecamera M3M (mono) corrisponde alla sua profondità.** Una M3M offre solo formati mono, mentre i nomi delle impostazioni predefinite indicano formati Bayer: nella versione 1.2.0 la fotocamera rifiutava il formato e manteneva il proprio, mentre `settings.pixel_format` indicava quello Bayer. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, un `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` e un array&#x27;formato condiviso ora impostano il formato Mono della stessa profondità su un M3M: `"BayerRG8"` e `high_speed` danno `Mono8`; `"BayerRG12"`, `high_quality` e `triggered` danno `Mono12`; `cam.settings.pixel_format` lo nomina. Lo stesso vale al contrario: un M3C non viene mai salvato in formato Mono, quindi `"Mono12"` — da uno qualsiasi di questi, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` o il formato salvato di un array — produce `BayerRG12` su un M3C, indipendentemente da quale telecamera sia il master dell’array. Un `CameraSettings` che si passa mantiene il proprio nome, quindi lo stesso oggetto continua a richiedere a ciascuna telecamera quella profondità all’interno della propria famiglia. `connect_camera(preset=...)` mantiene il formato del preset (la versione 1.2.0 imposta la telecamera a 12 bit dopo l’applicazione del preset), e una telecamera aperta su un preset non scrive più ciò che ha acquisito in `PRESETS`. Qualsiasi altro formato che una telecamera rifiuta (uno che manca dal proprio elenco `PixelFormat`, come `RGB8` su un M3M o `BayerRG16` su un firmware che non lo supporta) non viene nemmeno denominato come proprio: dopo la connessione di un `LatticeCamera`, `apply_settings()` o `set_format()`, `cam.settings.pixel_format` e `packed_transport` indicano il formato trasmesso dalla telecamera, mentre una riga stampata indica il formato rifiutato.
> - **`analyze_array_network()` indica a quale connessione si applica:** `pixel_format=None` (precedentemente `"BayerRG8"`) indica i formati impostati dalla connessione (Mono12 / BayerRG12), mentre `packed="auto"` indica la scelta del trasporto compresso, pertanto il valore di fps riportato potrebbe differire da 1.2.0&#x27;. Un `pixel_format` che si passa indica un M3M a quella profondità. Vedi [Network-Analysis Helper](#network-analysis-helper).
> - **`CameraSession.capture()` restituisce un `CaptureResult`** (ancora un elenco di dizionari di frame) con `.skipped`, `.failed`, `.daq_files` e `.daq_matches`. Un risultato `ArraySession.capture()` contiene anche `.alignment` (che indichi se è stata applicata la distorsione di allineamento e, in caso contrario, il motivo) e `.oriented`.
> - **Le acquisizioni degli array vengono allineate per impostazione predefinita, così come quelle dell’app.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` e `ArrayHandle.capture()` / `capture_smart()` applicano la distorsione ai file `debayered`, `display`, `radiance`, `reflectance` e i file di indice nel fotogramma della telecamera principale e li ritaglia in base alla sovrapposizione dei membri (più piccola del sensore di un valore pari agli offset delle telecamere, `XMP-mapir:AlignmentJSON` `"applied": true`), in modo che siano gli stessi file acquisiti dall&#x27;app Chloros per la stessa scena; `raw` rimane in formato nativo. In precedenza, un’acquisizione senza `aligned=True` salvava ogni file alle dimensioni native, senza correzione della distorsione. `aligned=False` mantiene questa impostazione. `chloros-mcp``capture_array` e `start_capture_run` adottano la stessa impostazione predefinita. Vedi [Metodi `ArraySession`](#arraysession-methods).
> - **`connect_all()` indica un backend che rifiuta l’SDK.** Un backend in esecuzione che è disconnesso o su un piano inferiore a Copper viene segnalato sotto `errors` con il proprio motivo (HTTP 401 / 403), non come &quot;nessun backend Chloros in esecuzione&quot;; inoltre, ogni chiamata `connect_all()` verifica nuovamente il backend, quindi richiamarla dopo aver effettuato l’accesso funziona sullo stesso oggetto progetto.
> - **Un `output_dir` relativo funziona con un backend locale.** I percorsi relativi vengono trasformati in assoluti rispetto alla directory di lavoro dello script prima di essere inviati. Nella versione 1.2.0 il backend li risolvava nella propria cartella di installazione, il che causava un errore con il messaggio &quot;Accesso negato&quot;.
> - **`numpy` è ora un requisito** dell’SDK (le chiamate live-frame restituiscono array numpy).
> - **Un nome a livello di esportazione diverso da &quot;one&quot; viene rifiutato.** `processing=` e `levels=` accettano `raw`, `debayered`, `radiance`, `reflectance` o `display` (o un alias come `preview`; `processing=` e anche `"all"`). Qualsiasi altro nome viene rifiutato prima che la fotocamera venga toccata (400 HTTP `UNKNOWN_LEVEL`: `capture()` genera `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); la versione 1.2.0 ha salvato o restituito `raw` per questo.
> - **La funzione &quot;Texture Aware&quot; non è supportata per le acquisizioni LATTICE.** In un progetto contenente acquisizioni LATTICE, `configure(debayer="Texture Aware (Slow, Highest Quality)")` genera un errore `ChlorosConfigurationError`, così come `process()` dopo un `configure()` che ne ha richiesto prima dell’importazione. Un valore di Texture Aware salvato in precedenza dal progetto viene impostato su Standard, con un errore `UserWarning`. Le acquisizioni LATTICE utilizzano sempre il demosaic standard, quindi la versione 1.2.0 ha accettato l’impostazione e l’ha ignorata; nemmeno l’app Chloros la offre per queste acquisizioni. Vedi [Valori supportati](#supported-values).
> - **Le fotocamere tornano allo stato in cui sono state trovate — a meno che non si verifichi un errore `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, alla fine di ogni funzione transitoria (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) e l’arresto di `start_camera_continuous()` riportano ciascuna telecamera allo stato in cui è stata trovata: il cablaggio del trigger e della linea di sincronizzazione, il PTP e tutte le impostazioni della telecamera modificate durante la sessione (esposizione e guadagno, il target AE, i limiti e lo smorzamento, il bilanciamento del bianco, il livello del nero, il formato dei pixel, il binning e la ROI, i capovolgimenti, la frequenza dei fotogrammi). `keep_settings=True`, invece, salva le impostazioni di ciascuna telecamera nella memoria della telecamera stessa, in modo che si avvii con quelle; `clear_settings=True` ripristina quella memoria alle impostazioni di fabbrica. In entrambi i casi le telecamere continuano a funzionare in modo autonomo. Un progetto imposta le proprie telecamere in base alle impostazioni salvate quando le collega. Vedi [Memoria della telecamera](#camera-memory--keep_settings--clear_settings).
> - **Ogni modalità salva un progetto allo stesso modo.** Qualsiasi modifica apportata a una telecamera, a un array o a un sensore di luce collegati a un progetto — dall’SDK, dall’app, dalla CLI o dal server MCP — viene salvata nel file `cameras.json` / `sensors.json`, e l’apertura del progetto in qualsiasi modalità configura i dispositivi in base a esso; un `ChlorosProject` assegna un nome alla propria cartella nel backend. In precedenza, solo l’app salvava il progetto. Vedi [Un unico record di progetto, in ogni modalità](#one-project-record-every-mode).
> - **I file generati da più telecamere riportano il nome di ciascuna di esse.** Uno stack multibanda, un indice in tempo reale creato dalle bande di diverse telecamere e un’immagine di acquisizione in array `composite/` contengono il numero di serie, il modello e il pacchetto di calibrazione di ciascuna telecamera sorgente sotto forma di elenchi (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), mentre i dati EXIF `SerialNumber` elencano i numeri di serie separati da virgole. In precedenza non riportavano alcuna fotocamera o riportavano solo il numero di serie della prima fotocamera. Vedi [File creati da più fotocamere](#files-made-from-several-cameras).
> - **Solo un file `raw` contiene i tag di calibrazione della fotocamera.** Il set compatibile con MicaSense (`BlackLevel`, `RadiometricCalibration`, i set`Band{R,G,B}RadiometricCalibration` per canale, i set `MicaSenseVignetting*`) e il payload `CalibrationJSON` descrivono i valori grezzi del sensore, quindi sono contenuti esclusivamente nel file di acquisizione `raw`. Tutti gli altri file — una cattura`debayered`, `display`, `radiance`, `reflectance` e i file di indice, i fotogrammi di registrazione continua e multi-telecamerae ogni file di prodotto `process()` — riporta l’identificativo della telecamera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) e, al momento dell’acquisizione, i dati specifici del fotogramma (`XMP-mapir:FrameJSON`: esposizione, guadagno, temperatura, formato dei pixel, ora di acquisizione). Nella versione 1.2.0 contenevano anche il set raw, che uno strumento di elaborazione (Pix4D, Metashape) applicava ai pixel che non presentavano valori raw. Processing individua la calibrazione di un file di questo tipo tramite il suo numero di serie e `CalBundleSha256`.
>
> **Altre modifiche:**
>
> - **La frequenza dei fotogrammi automatica di un array si adatta a quella fornita dall’array stesso.** Una frequenza che perde più dell’1% dei propri trigger viene ridotta, in base a una valutazione effettuata su diverse osservazioni prolungate anziché su una singola, e la frequenza da cui è stata ridotta viene riprovata di tanto in tanto. Una frequenza impostata manualmente rimane invariata.
> - **L’esposizione automatica è più stabile.** Si mantiene costante in condizioni di luce costante, si stabilizza entro pochi secondi da un cambiamento, ignora le piccole aree sovraesposte e ripristina il funzionamento di una fotocamera che smette di rispondere in pieno sole. Vedi [Come funziona l’esposizione automatica](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` conta solo le immagini e le registrazioni dell’utente.** Una cartella contenente il file `camera_cal/` di un progetto non vede più quei file di calibrazione conteggiati come immagini e registrazioni, e un progetto che li aveva già acquisiti li elimina.
> - **Si applica `configure(radiometric_output=...)`.** Ora imposta i prodotti `Export radiance` / `Export reflectance` del progetto; in precedenza, le impostazioni dei prodotti specifiche di ciascun progetto avevano la precedenza, quindi `radiance`, `sensor-response` e `all` esportavano tutti radianza e riflettanza.
> - **`test_array_sync()` parte dallo stato di accensione delle telecamere.** Attiva il PTP delle telecamere per l’esecuzione, attende che i loro orologi si sincronizzino e si stabilizzino prima del primo trigger (`ptp_lock`) — ogni telecamera rimane sincronizzata per 120 s, circa 2 minuti dall’accensione (`ptp_settle_s=` lo imposta), poiché un orologio appena sincronizzato presenta uno scostamento fino a ~200 us per circa 2 minuti — e riporta il PTP di ciascuna telecamera allo stato in cui lo ha trovato. Non si attende nuovamente una telecamera ancora sincronizzata da un test precedente. In precedenza, lasciava il PTP disattivato su una rete a switch singolo in cui nessun DAQ-E rispondeva, e leggeva l’orologio a funzionamento libero di ciascuna telecamera come un orologio PTP che non si sarebbe sincronizzato (&quot;PTP non sincronizzato&quot;, ad ogni esecuzione).
> - **Una registrazione `.daq` indica se il suo cappuccio è stato selezionato o presunto.** DAQ-U, DAQ-M e DAQ-E vengono forniti senza cappuccio, e Chloros non è in grado di rilevare se ne sia montato uno; pertanto, a un sensore collegato senza un cappuccio specificato viene assegnato per impostazione predefinita il nome `sunshine_cosine`, ovvero un’*ipotesi*. Ogni registrazione Chloros (schema 1.30) ora memorizza `cap_id_source` accanto a `cap_id`: `operator` quando si è assegnato un nome al cappuccio (`none` incluso), `auto_default` se era l’impostazione predefinita. L’elaborazione di una registrazione il cui condensatore è stato ipotizzato, con l’impostazione del condensatore DAQ del progetto lasciata su Auto, genera un `[daq-dls] WARNING` che indica il nome del file: un DAQ-U senza filtri, con la correzione &quot;sunshine&quot;, registra un valore circa 30 volte superiore. **Assegnate un nome al condensatore che avete installato** al momento del collegamento (`connect_daq_sensor(cap_id=...)` o `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, oppure l’opzione del menu a tendina “condensatore” della GUI), oppure imposta il condensatore DAQ del progetto, e l’avviso scomparirà.
> - **Niente più finestre di console pip su Windows.** Su un PC con Python installato, il backend e ciascuno dei suoi image worker eseguivano `pip` una volta all’avvio, e ciascuno apriva una finestra di console. Ora non lo fanno più.
> - **Le registrazioni `.daq` in tempo reale sopravvivono a un&#x27;interruzione di corrente.**Una registrazione viene scritta con un journal e una sincronizzazione completa del disco ad ogni commit, a livello del sensore, quindi un disco lento ritarda il file ma mai le letture. Una registrazione interrotta a metà di un commit (interruzione di corrente, terminazione forzata) può lasciare accanto a sé un file `<name>.daq-journal`:**copiare i due file insieme.**Chloros ripristina il journal alla prima lettura del file, quindi la registrazione si apre con tutto ciò che è stato salvato prima dell’interruzione (vengono persi circa gli ultimi 3 s); nella versione 1.2.0 un’interruzione di corrente nel momento sbagliato poteva rendere l’intero file illeggibile (“l’immagine del disco del database è danneggiata”).
> - **`DAQSensorSession.record_start()` su un disco occupato non blocca più il sensore**: le letture vengono messe in coda e scritte su un thread dedicato, e quelle che non è stato possibile scrivere vengono conteggiate in `status()` di `record_errors` (insieme a `record_last_error`).
> - **`LatticeCamera` diretto:**un fotogramma senza esposizione/guadagno del chunk ora riporta l’esposizione e il guadagno**reali** della fotocamera (il fallback leggeva un valore memorizzato nella cache che poteva rimanere bloccato per un’intera sessione di esposizione automatica), mentre `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` applicano lo stesso deglitch di ingresso da 5 µs di cui beneficiano gli slave via cavo di Chloros (senza filtro, uno slave generava fotogrammi senza che fosse stato inviato alcun trigger).
> - **Le fotocamere con trigger software non tralasciano più un trigger su due.** Una fotocamera in modalità trigger software ora mette in coda un trigger che arriva durante la sua lettura (oppure Chloros attende che sia pronta), come già facevano le fotocamere sincronizzate via cavo.
> - **`discover_daq_sensors()` assegna un nome a un DAQ-E-S `DAQ-E-S`.** Il suo `model` era `DAQ-E` per ogni sensore Ethernet; un sensore che non risponde al controllo di identità entro circa 2 s viene comunque elencato, come `DAQ-E`.
> - **Novità: `chloros-mcp`, un server MCP per assistenti AI.** Claude Desktop, Claude Code, Cursor e altri client MCP possono elaborare immagini (creare un progetto, importare acquisizioni, scegliere le impostazioni, elaborare, generare report sui risultati dell’elaborazione) e gestire telecamere LATTICE, array e sensori di luce DAQ: connettersi, visualizzare l’anteprima, impostare esposizione e colore, acquisire, registrare. I suoi strumenti coprono le funzionalità di questo SDK. Si installa insieme all’SDK e il pacchetto .deb per Linux lo colloca in `/usr/bin`. Vedi [Server MCP (Assistenti IA)](#mcp-server-ai-assistants).
> - **Novità: utilità per le telecamere**, i comandi `chloros-cli lattice` come chiamate dell’SDK: profilo colore, saturazione / contrasto, luci, bilanciamento del bianco (one-shot, automatico, rapporti fissi), preimpostazioni, modelli di filtro, stato e aggiornamento del firmware, alimentazione (rilevamento, sospensione, riattivazione), calibrazione del bersaglio, schede di rete (verifica, stima, correzione), correzione della vignettatura sui file, lettura DLS e rilevamento dell’host. Vedi [Utilità della fotocamera](#camera-utilities).
> - **Novità: il resto dell’CLI come chiamate SDK**, quindi l’CLI, l’SDK e `chloros-mcp` offrono le stesse funzionalità (l’CLI mantiene solo l’accesso, il provisioning di un DAQ-E e i server DAQ diretti, per motivi di sicurezza): telecamere aperte con un unico comando (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), file dei profili di allineamento (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indici (`index_catalog()`, `compute_index()`, `capture_live_index()`), il grandmaster PTP (`time_sync_status()` e simili), la cartella del progetto (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) e le ricette (`run_project_recipe()`, il runner proprio della CLI). Per i sensori di luce: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` e `light_sensor_reflectance()` (riflettanza e indici ambientali e degli oggetti di due sensori); per una telecamera: `CameraSession.status()` e `capture_repeated()`; per un array: `ArraySession.capability()` e `set_member_settings()` (esposizione, guadagno, ecc. di un elemento). Vedi [Array transitori, profili di allineamento e indici](#transient-arrays-alignment-profiles--indices), [Cartella di progetto e ricette](#project-folder--recipes) e [Sessioni dei sensori DAQ](#daq-sensor-sessions).
> - **Prodotti indice LATTICE migliorati** (NDVI e altri indici) da `process()`, in ogni modalità di elaborazione.
> - **Telecamere LATTICE OCN: prodotti di radianza, riflettanza e indici migliorati in condizioni di luce solare.** Viene applicata una correzione della flotta dopo la separazione delle bande — in tempo reale, nelle acquisizioni e in `process()`; le telecamere RGN rimangono invariate. Vedi [Output radiometrico](#radiometric-output-lattice-multispectral-pipeline).
> - **Ogni acquisizione salva la lettura DAQ utilizzata, proprio come fa l’interfaccia grafica.** `CameraSession.capture()` e `ArraySession.capture()` / `continuous()` scrivono il file `.daq` e il relativo `.match.json` ogni volta che un DAQ viene associato o inserito nel pool, inclusa una cattura solo raw; nella versione 1.2.0 una cattura raw o debayered ne salvava una sola con `force_daq=True`. `daq=False` la disattiva e `daq_tolerance_ms=` imposta la tolleranza di corrispondenza; entrambi i risultati indicano ciò che è stato salvato in `.daq_files` e `.daq_matches`. Ora un&#x27;acquisizione attende la lettura successiva del DAQ in modo da poter fondere le letture su entrambi i lati dell&#x27;esposizione. Vedi [La lettura del DAQ salvata da un&#x27;acquisizione](#the-daq-reading-una-acquisizione-salva--file-daq-e-corrispondenze-daq).
> - **`ArraySession` contiene fotogrammi in tempo reale e un’acquisizione continua lato server.** `grab()` e `frame_stream()` restituiscono fotogrammi numpy da un trigger (la stessa lettura delle API di progetto `ArrayHandle.grab()` / `frame_stream()`), mentre `continuous()` salva ogni gruppo sincronizzato nel backend e restituisce un [`ContinuousHandle`](#continuoushandle). Vedi [Frame in tempo reale nella propria pipeline](#live-frames-in-your-own-pipeline).
> - **Un array `grab()` è associato a un trigger** (`ArrayHandle.grab()` e il nuovo `ArraySession.grab()`). Nella versione 1.2.0, 12 delle 20`ArrayHandle.grab()` a quattro telecamere coprivano due trigger. Ora ogni fotogramma contiene `group_id`, e un array `frame_stream()` genera un risultato per ciascun trigger una sola volta. Vedi [Un trigger per lettura](#one-trigger-per-read--group_id).
> - **Frame per banda e allineati:** `grab()` / `frame_stream()` prendono `band=`, `align=`, `bit_depth=` e `refl_max=` sugli handle della telecamera e dell’array. Vedi [Esportazione di una singola banda](#exporting-a-single-band--band).
> - **Nuovi metodi e argomenti `ArraySession`:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (insieme a `CaptureResult.group_seq` e `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` concatena gruppi consecutivi e si interrompe quando l’unità è piena.
> - **`ArraySession.record()` registra senza alcuna GUI aperta.** Con la vista combinata della GUI chiusa, il backend rende la vista stessa: l’arrayNDVI dell’array, oppure l’indice che si passa come `expr=`. `RecorderHandle.stop()` genera un’eccezione `ChlorosConnectError` per una registrazione fallita, ovvero una registrazione che non ha scritto alcun fotogramma, e mantiene il risultato di interruzione su `.result` (lasciare un blocco `with` non genera un&#x27;eccezione: controllare `rec.result`). Vedi [`RecorderHandle`](#recorderhandle).
> - **Nuovi metodi `DAQSensorSession`:** `enable_ptp()`, `force_disconnect()` e, per le impostazioni specifiche del DAQ-E, `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` e `update_firmware()`. `record_stop()` restituisce `warning` e `gaps`. Un riavvio che ha chiuso una registrazione ne riporta il nome nella propria risposta oppure, quando il riavvio fallisce, su `ChlorosConnectError` (`recording_closed`; vedere [Impostazioni del dispositivo DAQ-E](#daq-e-device-settings)). Vedere [Metodi `DAQSensorSession`](#daqsensorsession-methods).
> - **Un DAQ-E in pool che smette di trasmettere viene ricollegato dal backend**, con le impostazioni che aveva, e una registrazione su di esso prosegue nello stesso `.daq` con l’intervallo contrassegnato nel file; `record_stop()` restituisce le interruzioni come `gaps`. Nella versione 1.2.0 il sensore rimaneva inattivo fino a quando non veniva disconnesso e ricollegato, il che terminava la registrazione. Vedere la sezione [Un DAQ-E che smette di trasmettere](cli-reference.md#a-daq-e-that-stops-delivering) del riferimento alla CLI. [Un DAQ-E che smette di trasmettere](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` funziona a partire da un SDK installato.** Il wheel (quello del programma di installazionee PyPI) ora contiene i caricatori di progetto necessari; in precedenza, `open_project()` generava `ImportError` che chiamava `project_cameras` / `project_sensors` in qualsiasi SDK installato tramite pip. `connect_all()` collega le telecamere, gli array e i sensori di luce salvati tramite un backend in esecuzione (i sensori di luce tramite il proprio pool DAQ), e un dispositivo salvato che non ha modo di individuare viene elencato sotto `errors` con l’indicazione del motivo, invece di essere ignorato senza alcuna spiegazione. Un array salvato che è già attivo nel backend (quello delle GUI, o di `chloros-cli project connect`) viene collegato e lasciato in esecuzione, mentre in precedenza veniva segnalato come errore (“La telecamera  <sn>è già presente nell’array<id>”). Vedi [Metodi di `ChlorosProject`](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` e `process_lattice_capture()` individuano Chloros su Windows.** Cercano dove il programma di installazione colloca il backend (`<install folder>\backend\`, `C:\Program Files\Chloros\backend\` per impostazione predefinita, una cartella personalizzata indicata dal programma di installazione stesso), e si connettono a un backend già in esecuzione senza cercarne uno nuovo. Nella versione 1.2.0 generavano l’errore “Backend Chloros non trovato” su una macchina installata, a meno che non si passasse `backend_exe=`.
> - **Un backend che si arresta all’avvio viene segnalato immediatamente.** Quando `ChlorosLocal()` o una chiamata smart-connect avvia un backend che si chiude prima di rispondere, la chiamata genera in pochi secondi l’errore `ChlorosBackendError` da `ChlorosLocal` e `ChlorosConnectError` dalle chiamate smart--connect, con il codice di uscita, il comando eseguito e le ultime righe stampate dal backend. La versione 1.2.0 attendeva il timeout di avvio di 60 s senza fornire alcuna spiegazione. Vedi [Avvio automatico del backend](#backend-auto-start).
> - **Il backend continua a funzionare tra uno script e l’altro.** Uscendo da un blocco `with ChlorosLocal(...)`, il ritorno di `process_folder()` / `process_lattice_capture()` e l’uscita di `chloros-mcp` non arrestano più un backend avviato dall’SDK; lo script successivo lo riutilizza. Si arresta su `logout()` (a meno che l’app Chloros non sia aperta o sia in corso un’esecuzione), su `shutdown_backend()`, oppure automaticamente dopo 30 minuti se non viene utilizzato da nulla (`CHLOROS_BACKEND_IDLE_MIN` nell’ambiente del backend modifica questa impostazione; `0`: mai).
> - **A un backend avviato dall’SDK viene richiesto di arrestarsi, non viene terminato forzatamente.** `shutdown_backend()` consente al backend di liberare le proprie telecamere e i sensori di luce e di completare eventuali registrazioni; viene terminato solo se è ancora in esecuzione 10 secondi dopo. La versione 1.2.0 lo terminava immediatamente (su Windows, un&#x27;interruzione forzata), il che poteva rendere non rispondente un sensore di luce DAQ collegato a quel backend. Un backend che l&#x27;SDK trovava in esecuzione veniva comunque lasciato in pace.
> - **Su Linux, l&#x27;avvio automatico non esegue mai un `backend_server.py` dalla cartella corrente.** In assenza di un backend installato, uno script avviato in una cartella che ne conteneva uno lo eseguiva; ora viene avviato solo il backend installato (o, per `ChlorosLocal`, quello specificato da `backend_exe=`).
> - **`configure(indices=...)`**aggiunge ogni preset al progetto per le bande della relativa telecamera, inclusa una nuova cartella &quot;project&quot;, e avvisa (`UserWarning`) in caso di un nome che non è in grado di aggiungere.**`import_images()`** riporta le registrazioni `.daq` come `scan_count` e non avvisa più in caso di una cartella di acquisizione LATTICE.
> - **`read_image_audit_tags()` funziona con un&#x27;installazione tramite pip.** Legge autonomamente i metadati dell&#x27;immagine; nella versione 1.2.0 restituiva `{}` senza i componenti interni dell&#x27;installazione desktop presenti nel percorso Python.
> - **`capture()`, subito dopo `connect_array()`**, attende fino a 20 secondi per il primo gruppo di fotogrammi dell’array invece di restituire un errore HTTP503 “Array still warming up”.
> - **`status()` segnala un blocco.** Il nuovo campo `stalled` è True mentre il ciclo di acquisizione è in esecuzione ma non pubblica nulla, e `actual_fps` è quindi 0. Nella versione 1.2.0, `actual_fps` manteneva il suo ultimo valore durante un blocco, quindi un array bloccato mostrava `actual_fps > 0` mentre le acquisizioni fallivano con errore 503 “Nessun gruppo di fotogrammi sincronizzato recente”.
> - **Meno set di fotogrammi incompleti ed errori 503 durante l’acquisizione:** l’ingresso di trigger di ciascuna telecamera slave è ora privo di glitch.
> - **Trasporto compresso a 12 bit** (`connect_array(packed="auto")`) aumenta la frequenza dei fotogrammi dell’array a sensore intero di circa un terzo. Vedi [Trasporto compresso a 12 bit](#packed-12-bit-transport-packed).
> - **Esposizione dell’array in un’area di interesse (ROI):** una ROI di piccole dimensioni non limita più l’esposizione di un elemento al periodo di funzionamento libero del sensore (4,9 ms a 768 × 304). Vedi [Esposizione sugli elementi dell’array](#exposure-on-array-members).
> - **L’esposizione automatica non riduce più la frequenza dei fotogrammi automatica di un array.** Un array sincronizzato via cavo funziona alla frequenza consentita dalla rete e dalla lettura delle telecamere; l’esposizione automatica riempie il periodo consentito da tale frequenza, quindi aumenta il guadagno. Un’esposizione manuale più lunga del periodo rallenta il trigger per adattarsi ad essa, invece di essere rifiutata. Vedi [Esposizione sugli elementi dell’array](#exposure-on-array-members).
> - **`read_nodes()`**aggiunge `frame_values` e `notes` mentre l’esposizione automatica è in esecuzione (vedere [`read_nodes()` in esposizione automatica](#read_nodes-under-auto-exposure--frame_values-and-notes)). Quando la fotocamera rifiuta un’esposizione o un guadagno,**`set_settings()`** indica l’intervallo che accetta e ripristina la modalità automatica che era stata disattivata per quel valore; `set_settings(target_brightness_auto=True)` è un nuovo codice.
> - **`orient=`** su `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` e `ArrayHandle.capture()` / `capture_smart()` ruota i prodotti in base all&#x27;orientamento della fotocamera. Vedi [Metodi `ArraySession`](#arraysession-methods).
> - **`DLS()` richiede l’`mac_address` proprio del sensore.** Non esiste più un MAC predefinito (era l’indirizzo di un’unità); l’uso di `DLS()` senza un MAC genera un errore `DLSError`. Vedi [magnitudini della banda DAQ-M](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Il programma di installazione per Windows** offre le opzioni «Abilita percorsi lunghi di Windows» ed «Escludi il backend Chloros dalla scansione di Microsoft Defender», entrambe selezionate per impostazione predefinita. Vedi [Passo 1](#step-1--install-the-chloros-platform-package).
> - **Il pacchetto .deb per Linux** installa l’Python SDK per il sistema `python3` (in precedenza, si limitava a copiare il file wheel sul disco), aumenta la dimensione dei buffer di ricezione dei socket del kernel necessari per i flussi delle telecamere e aggiunge un servizio backend che viene eseguito con i privilegi dell’utente. Il pacchetto Jetson è `chloros_1.3.0_arm64_jp6.deb`, per JetPack 6, con PyTorch CUDA di JetPack. Vedi [Passo 1](#step-1--install-the-chloros-platform-package) e [Passo 2](#linux-deb).

---

## L&#x27;architettura in 60 secondi

L’SDK è un sottile strato Python che si sovrappone al backend Chloros (lo stesso server Flask utilizzato dall’interfaccia grafica desktop e dalla CLI). Per l’automazione, si importa `chloros_sdk` e si chiamano metodi di alto livello; dietro le quinte, ogni chiamata diventa una richiesta HTTP al backend locale sulla porta 5000 — `http://127.0.0.1:5000/api/...` (deliberatamente non `localhost`, che su Windows viene prima risolto in `::1` e richiede circa 2 s per ogni richiesta verso un backend solo IPv4). Il backend gestisce il pool hardware — telecamere, sensori DAQ, profili di allineamento, frame buffer — in modo che gli script dell’SDK possano coesistere con la GUI senza competere per le porte seriali o la larghezza di banda della scheda di rete.

Ci sono tre interfacce che userete:

1. **`ChlorosLocal` + funzioni libere** (`process_folder`, `process_lattice_capture`) — Pipeline di elaborazione delle immagini. Esegui l’elaborazione completa di un’intera cartella (calibrazione / debayer / esportazione indice) tramite un’unica chiamata Python.
2. **Gestione Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Apri una sessione backend persistente per l’hardware in tempo reale. Stesso flusso “smart-prep” dell’interfaccia grafica: sonda di rete, selezione automatica del livello, PTP, seeding AE, configurazione del trigger GPIO.
3. **`ChlorosProject` / `open_project`** — Carica un progetto salvato (cartella contenente `cameras.json` + `sensors.json` + `project.json`), collega contemporaneamente le sue telecamere, gli array e i sensori di luce, e gestisci le acquisizioni tramite handle denominati.

Le superfici 1 e 2 **avviano automaticamente un backend locale** se non ce n’è già uno in ascolto (lo stesso binario in dotazione che la GUI e la CLI generano) — quindi uno script semplice funziona da una shell nuova senza che sia necessario avviare prima un backend: `ChlorosLocal` (e `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` e `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` e `analyze_array_network()` non si avviano mai: necessitano di un backend in esecuzione. Passare `auto_start_backend=False` per disattivare questa opzione (ad esempio quando si punta a un backend remoto, che non viene mai avviato). Vedi [Avvio automatico del backend](#backend-auto-start). Surface 3 si comporta in modo diverso: `open_project()` non accetta il parametro `auto_start_backend`, e `connect_all()` non avvia mai un backend : interroga `http://127.0.0.1:5000` una volta e, se non riceve risposta, ricorre al controllo diretto (senza backend) del dispositivo `lattice_sdk`. Ciò richiede il runtime dell’SDK di Arena e OpenCV (`cv2`) importabile nel vostro Python, che la wheel dell’SDK non installa; senza di essi, ogni telecamera e array salvati vengono segnalati come errore, con l’indicazione del motivo. I sensori di luce di un progetto non hanno un percorso senza backend: in assenza di un backend in esecuzione, ogni sensore salvato viene segnalato come errore, con l’indicazione del motivo (vedi [`SensorHandle`](#sensorhandle)). Solo `proj.process()` e `stream(..., overlays=True)` costruiscono in modo differito un `ChlorosLocal()` (che si avvia automaticamente).

Tutti e tre sono auth: eseguire `chloros-cli login` una volta sulla macchina oppure effettuare l’accesso tramite l’interfaccia grafica desktop. Ciò che genera una chiamata senza una sessione valida dipende dall’interfaccia (vedi [Eccezioni](#exceptions)): `ChlorosAuthenticationError` da `ChlorosLocal` (e `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` dalle chiamate smart-connect e delle relative sessioni, `status: "error"` con un `error_code` derivante da `analyze_array_network()`, e una voce sotto `errors` nel report `connect_all()`.

Requisiti:
- Python 3.7+ (come dichiarato dal pacchetto; sviluppato/testato su 3.10)
- Chloros Desktop installato localmente (il binario del backend è incluso nel programma di installazione)
- Accesso Chloros+ attivo su **qualsiasi piano a pagamento di Chloros+ (Copper o superiore)**; il piano gratuito**Iron**non prevede l’accesso a SDK o CLI. Questa restrizione è applicata**dal lato server****: ogni richiesta contrassegnata come SDK/CLI deve includere sia una sessione attiva che un piano a pagamento. Un piano inferiore a Copper genera l’errore `403` con `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` da `ChlorosLocal`, `ChlorosConnectError` dalle chiamate smart-connect). Un chiamante disconnesso riceve invece `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` da `ChlorosLocal`, `ChlorosConnectError` dalle chiamate smart-connect) — i due sono distinti perché l&#x27;esecuzione di `chloros-cli login` risolve l’errore `401` ma non può risolvere l’errore `403`, che richiede un upgrade del piano.
- L’utilizzo offline è supportato entro il periodo di tolleranza del piano: il livello viene letto dalla cache di convalida del server (5 min) o dalla cache delle licenze firmate e legate al dispositivo (30 giorni per i piani mensili, fino alla scadenza dell’abbonamento per quelli annuali). Allo scadere di tale periodo di tolleranza, il piano passa alla versione gratuita e l’accesso a SDK e CLI viene interrotto fino a quando il dispositivo non riesce a connettersi al server almeno una volta. L’errore `chloros-cli status` (`GET /api/license-status`) rimane visibile nel livello gratuito, in modo che il motivo sia chiaro.
- Windows 10/11 a 64 bit, **Ubuntu 22.04 LTS o versioni successive**, oppure Jetson (JetPack 6). Ubuntu 20.04**non** è supportato: le dipendenze di `.deb` derivano da ciò a cui il backend si collega, incluso `libc6 (>= 2.35)`, e Focal distribuisce glibc 2.31.

---

## Installazione

L’SDK Python per Python è un sottile livello Python che si sovrappone al backend Chloros. A meno che non si utilizzi un backend su un’altra macchina (vedere [Modalità backend remoto](#remote-backend-mode-pip-only-host-via-tunnel)), è necessario che il **pacchetto desktop Chloros sia installato localmente** (programma di installazione per Windows o Linux `.deb`), che include i sensori di luce DAQ. Il pacchetto fornisce il binario del backend, il runtime dell’SDK Arena per le telecamere LATTICE e i pacchetti di calibrazione.

Download più recenti: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Passaggio 1 — Installare il pacchetto della piattaforma Chloros

#### Windows (.exe)

1. Scaricare `Chloros Installer 1.3.0.exe` dalla pagina dei download.
2. Eseguire il programma di installazione e seguire la procedura guidata. Il percorso di installazione predefinito è `C:\Program Files\Chloros\`.
3. Avviare Chloros almeno una volta ed effettuare l’accesso con il proprio account Chloros+.

Il programma di installazione consente al backend (`chloros-backend.exe`, nient’altro) di attraversare il Firewall di Windows su ogni profilo di rete: UDP in entrata, per i flussi delle telecamere, PTP, mDNS e il flusso DAQ-E, e TCP in entrata solo dalla sottorete locale, per la connessione che un DAQ-E stabilisce con il computer durante un aggiornamento del firmware ([`update_firmware()`](#daq-e-device-settings)). Il programma di disinstallazione rimuove entrambe le regole. L’API HTTP HTTP del backend rimane su `127.0.0.1` in entrambi i casi.

Il gruppo **Impostazioni di sistema** della procedura guidata presenta due opzioni, entrambe selezionate per impostazione predefinita:

| Opzione | Funzione | Disattivazione installazione silenziosa |
| --- | --- | --- |
| **Abilita percorsi lunghi di Windows** | Consente alle cartelle di progetto di trovarsi a un livello di profondità superiore al limite di 260 caratteri dei percorsi di Windows. Se i percorsi lunghi sono disattivati (impostazione predefinita di Windows), una cartella di progetto troppo profonda per i propri prodotti viene rifiutata prima che venga elaborato qualsiasi dato — vedere [Risoluzione dei problemi](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Escludi il backend Chloros dalla scansione di Microsoft Defender** | Interrompe la scansione in tempo realedi ogni file scritto da un’acquisizione della telecamera. A frequenze di acquisizione elevate, la scansione incide per circa il 30% della frequenza: misurato con ROI 512 su un array di 9 telecamere, l’acquisizione raw continua ha raggiunto 52,7 acquisizioni/s con la scansione e 77,7 acquisizioni/s con il backend escluso. | `/MERGETASKS="!defenderexclusion"` |

Un&#x27;acquisizione continua segnala il carico di Defender come `defender_cores` (vedi [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Installare con `apt install ./…`, non con `dpkg -i`: apt installa anche i prerequisiti dell’Python SDK, che il pacchetto raccomanda anziché richiedere (vedere [Passaggio 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Per Jetson Orin / Orin NX / Orin Nano su JetPack 6 (L4T R36). Il suo PyTorch è la build CUDA di JetPack 6, quindi l’elaborazione GPU viene eseguita sulla GPU di Orin; utilizza le librerie CUDA e cuDNN installate da JetPack 6 (`sudo apt install nvidia-jetpack` su un’immagine che ne è sprovvista).

#### Linux: buffer di ricezione e servizio di backend

Entrambi i pacchetti .deb aumentano a 32 MB i buffer di ricezione dei socket del kernel necessari per gli streaming delle telecamere LATTICE (`/etc/sysctl.d/60-chloros-gige.conf`). Con le impostazioni predefinite del kernel, i pacchetti della telecamera vengono scartati nel kernel e un array perde fotogrammi e frequenza di fotogrammi; `chloros-cli lattice network` segnala quando i buffer sono insufficienti. Consultare la sezione [Host Linux — buffer di ricezione dei socket](cli-reference.md#linux-hosts--socket-receive-buffers) della guida di riferimento della CLI.

Su una macchina senza interfaccia grafica, esegui il backend come servizio **con il tuo account utente**, in modo che uno script ne trovi uno in esecuzione (`connect_all()` non avvia mai un backend):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Il backend verrà quindi eseguito come utente: le acquisizioni che scrive sono dell’utente e utilizza il file `chloros-cli login` dell’utente. Il file `chloros-backend.service` di root è ancora installato per le macchine che lo hanno abilitato, ma ogni file scritto da un backend come root appartiene a root. Vedere la sezione [Esecuzione del backend come servizio](cli-reference.md#running-the-backend-as-a-service-headless) della guida di riferimento della CLI.

Un backend eseguito dal servizio è condiviso e non viene mai arrestato da un client: gli script SDK e `chloros-cli` lo utilizzano così come lo trovano (`ChlorosLocal.shutdown_backend()` arresta solo un backend avviato da `ChlorosLocal` stesso), e `GET /api/status` assegna all’unità il nome `service`. Riavviarlo con `sudo systemctl restart chloros-backend@$USER.service`. L’aggiornamento del pacchetto arresta i servizi backend durante l’installazione, quindi uno script in esecuzione perde le sue telecamere e i suoi sensori, per poi riavviare tutti quelli abilitati; la rimozione del pacchetto li arresta e disabilita quelli abilitati. È possibile eseguire un solo backend alla volta, quindi abilitare una sola unità: una seconda unità non si avvia e rimane su `failed` (codice di uscita 98, non riavviato), e il suo `systemctl status` ne spiega il motivo. Per passare da un’unità all’altra, disabilitare quella che non si utilizza.</id></sn>

### Passaggio 2 — Installazione dell&#x27;Python SDK

**Il programma di installazione di Chloros include un wheel SDK corrispondente.** Ogni programma di installazione per Windows e ogni pacchetto .deb per Linux inserisce sul disco un file `chloros_sdk-X.Y.Z-py3-none-any.whl` che corrisponde esattamente alla versione della GUI, della CLI e del backend. Non è necessario controllare PyPI per mantenere la sincronizzazione.

#### Windows

Il programma di installazione esegue automaticamente `pip install` con il file wheel incluso, utilizzando il Python di sistema (si preferisce il launcher `py.exe`, ma in caso di problemi si ricorre a `python -m pip`). Non è richiesta alcuna azione: `import chloros_sdk` funziona nel proprio ambiente Python dopo un’installazione riuscita. Se sul sistema non è presente Python, il programma di installazione salta silenziosamente questo passaggio e l’interfaccia grafica (GUI) e la riga di comando (CLI) continuano a funzionare.

#### Linux (.deb)

Entrambi i pacchetti .deb (x86_64 e Jetson) installano l’SDK per il sistema `python3`, quindi `import chloros_sdk` funziona dopo l’installazione, senza bisogno di pip né di connessione di rete. I pacchetti (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` e i caricatori di progetto) vengono inseriti in `/usr/lib/python3/dist-packages`, mentre il comando `chloros-mcp` dell’SDK (il [server MCP](#mcp-server-ai-assistants)) va in `/usr/bin`; tutti questi elementi appartengono al pacchetto: un aggiornamento li sostituisce e la rimozione del pacchetto li elimina.

I requisiti propri dell’SDK, `python3-numpy` e `python3-requests`, sono **consigliati, non obbligatori** (la GUI, la CLI e il backend sono autonomi): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` su un Jetson) li installa, mentre `dpkg -i` no. L&#x27;output dell&#x27;installazione riporta la versione dell&#x27;SDK che `python3` importa, eventuali prerequisiti mancanti, e la presenza di un `chloros_sdk` in un&#x27;altra posizione nel percorso di `python3` che sovrascrive la copia del pacchetto (ad esempio uno installato in precedenza con `pip install --user`).

In un virtualenv, che non vede i pacchetti di sistema, installare al suo interno il wheel in dotazione:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Per le distribuzioni Jetson in modalità air-gapped, l’operazione avviene completamente offline: l’SDK e il wheel sono già presenti sul disco.

#### PyPI pubblico

Per host solo pip (senza pacchetto desktop Chloros installato; l’SDK gestisce quindi un backend su un’altra macchina, vedi [Modalità backend remoto](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI contiene l’ultima versione stabile. Assicurati che l’SDK sia della stessa versione del backend Chloros che gestisce (vedi [Versioni e compatibilità](#versioning--compatibility)). Anche l’helper DAQ-M `DLS` richiede `bleak`: `pip install "chloros-sdk[camera]"` (o `pip install bleak`).

#### Verifica

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **È richiesto un abbonamento a Chloros+.** Tutte le chiamate all’SDK richiedono un accesso attivo a Chloros+. Eseguire `chloros-cli login user@example.com 'YourPassword'` una volta per ogni macchina; le credenziali vengono memorizzate nella cache in `~/.chloros/`.

### È necessario il pacchetto Desktop?

Il pacchetto pip da solo **non** è sufficiente per la maggior parte dei flussi di lavoro. Ecco cosa richiede ciascuna interfaccia SDK:

| Interfaccia SDK | Richiede il pacchetto Desktop? | Perché |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Sì**(per avviarne uno)**/ No** (un backend già raggiungibile) | Utilizza un backend in esecuzione (quello dell’app desktop o un altro script avviato); altrimenti avvia automaticamente quello installato: `/usr/lib/chloros/chloros-backend` (Linux) o `<install folder>\backend\chloros-backend.exe` (per Windows; `C:\Program Files\Chloros\backend\` per impostazione predefinita, la cartella di installazione personalizzata viene letta dal record del programma di installazione). Passare `backend_exe=` per assegnargli un nome personalizzato. |
| `chloros-mcp` ([server MCP](#mcp-server-ai-assistants)) | **Sì**(per avviarne uno)**/ No** (un backend già raggiungibile) | Basato su `ChlorosLocal`, quindi uguale alla riga precedente. |
| `read_image_audit_tags` | **No** | Legge i metadati dell’immagine elaborata (solo libreria standard di Python). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Sì**(locale)**/ No**(remoto) | Client HTTP puri sul backend. Backend locale → è richiesto il pacchetto desktop. Backend remoto → `backend_url=`**tramite un tunnel** (vedi Modalità backend remoto — i backend forniti si legano solo al loopback). |
| `ChlorosProject` / `open_project` | **Sì** | Gestisce i progetti salvati tramite il backend. |
| Classi LATTICE dirette (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Sì** | Richiede il runtime dell’SDK di Arena, incluso nel pacchetto desktop, e OpenCV (`cv2`) nel proprio Python, che né il pacchetto desktop né l’SDK installano (`pip install opencv-python`). Altrimenti, `CAMERA_AVAILABLE` corrisponde a `False` al momento dell’importazione. `DLS` richiede anche `bleak`. |

### Modalità backend remoto (host solo pip, tramite tunnel)

> **Il backend installato non è raggiungibile tramite la LAN.** È in ascolto
> solo sulla porta di loopback (`127.0.0.1` / `::1`), quindi `backend_url="http://<lan-ip>:5000"`
> **non può raggiungere un Chloros installato**. Per gestire un backend su un’altra
> macchina, inoltrare manualmente la sua porta di loopback e indirizzare l’SDK verso il
> tunnel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Gli host headless / CI / robotica possono mantenere una macchina con l’installazione desktop completa come “server Chloros” e `pip install chloros-sdk` ovunque altrove — ma il trasporto tra di essi è il tunnel configurato dall’utente sopra indicato, non un URL LAN diretto.

> **`ChlorosLocal` su un host solo pip.** `ChlorosLocal(backend_url=BACKEND)` si connette a un backend raggiungibile senza installazione desktop: cerca un binario backend locale solo quando deve AVVIARNE uno (nessuna risposta e `auto_start_backend=True`). Fino alla versione 1.2.0 cercava prima il file binario e generava l’errore `ChlorosBackendError` (“Backend Chloros non trovato…”) su un host solo pip anche se il backend era raggiungibile. Le cartelle che si specificano (`process_folder`, `import_images`) vengono controllate su questa macchina e lette dal backend; pertanto, in presenza di un backend remoto, devono indicare la stessa cartella su entrambe le macchine, ad esempio un’unità condivisa.

### Flusso di lavoro solo DAQ (host solo pip)

`pip install chloros-sdk` da solo non può pilotare un sensore DAQ: un sensore DAQ viene sempre pilotato tramite un backend Chloros con `connect_daq_sensor()`:

- **Sensore su questa macchina:** installare qui il pacchetto desktop Chloros. `connect_daq_sensor()` avvia il proprio backend se non ne è in esecuzione nessuno.
- **Sensore su un&#x27;altra macchina:** quella macchina necessita del pacchetto desktop; raggiungi il suo backend tramite il tunnel in [Modalità backend remoto](#remote-backend-mode-pip-only-host-via-tunnel). Il backend apre il sensore sulla propria macchina, quindi un DAQ-U deve essere collegato a quella macchina, un DAQ-M deve trovarsi nel raggio d’azione del Bluetooth di quest’ultima e un DAQ-E deve trovarsi su una rete raggiungibile da essa.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

In entrambi i casi il backend applica il piano Chloros+: effettuare l’accesso sulla macchina che esegue il backend (`chloros-cli login` o l’app desktop) con un piano a pagamento qualsiasi (Copper o superiore). Un backend disconnesso e un piano inferiore a Copper generano entrambi un `ChlorosConnectError` contenente il motivo del backend (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Guida rapida

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Indice delle API di primo livello

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Elaborazione delle immagini — `ChlorosLocal`

La classe principale della pipeline. Avvia il backend al momento della sua creazione (se nessuno risponde e `auto_start_backend=True`), crea e configura i progetti, monitora lo stato di avanzamento e restituisce i riepiloghi al termine dell’esecuzione.

### Costruttore

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metodi

| Metodo | Descrizione |
| --- | --- |
| `create_project(project_name, camera=None)` | Crea un nuovo progetto (facoltativamente con un modello di telecamera come `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Apre un progetto salvato (la relativa cartella o il suo nome nella cartella dei progetti) e lo imposta come progetto aperto. I percorsi di un progetto spostato vengono reindirizzati alla cartella in cui si trova ora; `relink` indica il nome della cartella in cui si trovano ora gli altri file mancanti. Restituisce la risposta del backend con `source_files`; genera l’eccezione `ChlorosProjectFilesMissingError` quando non è possibile trovarne alcuno. Vedi **Un progetto spostato** di seguito. |
| `import_images(folder_path, recursive=False)` | Importa immagini RAW/TIF/JPG/DNG **e `.daq` registrazioni del sensore di luce**. Restituisce `count` (immagini) e `scan_count` (registrazioni). Emette un avviso solo se la cartella non contiene né l’uno né l’altro. |
| `export_light_sensor(daq=True, csv=True)` | Scrive i file calibrati `.daq` + `.csv` per ogni registrazione del sensore di luce del progetto, nella cartella `<project>/Light Sensor/`. Vedi [Registrazioni del sensore di luce](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Imposta i controlli di elaborazione. `indices` sono nomi di preset (`NDVI`, `GNDVI`, `OSAVI`, …), ciascuno aggiunto al progetto per le bande della propria fotocamera; un nome che non è un’impostazione predefinita, o che le bande non possono formare (`NDRE` richiede RE e NIR, che nessuna singola fotocamera fornisce), riceve un nome del tipo `UserWarning` e nessuna immagine indice. `daq_log_path` indica una registrazione del sensore di luce (un file `.daq` o `.csv`, oppure una cartella contenente tali file) con cui viene effettuato il confronto di ogni fotogramma, sia in LATTICE che in Survey3, anziché le registrazioni importate insieme alle immagini; se un percorso è mancante o non contiene alcuna registrazione, vengono utilizzate quelle importate. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Esegue la pipeline. Restituisce `{"status": "complete", "async": False}`, più una chiave `summary` quando il backend ne fornisce una — vedi [Riepilogo post-esecuzione e suggerimenti](#post-run-summary--hints) — `export_check_failure` quando non è stato possibile verificare le esportazioni dell’esecuzione e `image_failure` quando è stata persa un’immagine. `wait=False` restituisce `{"status": "started", "async": True}` contemporaneamente. |
| `get_config()` / `get_status()` / `status()` | Controlla lo stato del backend. |
| `logout()` | Disconnessione: cancella le credenziali memorizzate nella cache e libera lo slot del dispositivo di questa macchina (`device_deregistration` nel risultato). Un&#x27;esecuzione in corso non viene interrotta: termina nella modalità in cui è stata avviata (`run_in_progress: true` e `run_mode` nel risultato), e l&#x27;esecuzione successiva utilizza l&#x27;elaborazione standard. |
| `shutdown_backend()` | Arresta il backend se è stato avviato da questa istanza; un backend trovato in esecuzione viene lasciato in essere. Lasciare un blocco `with` o sottoporre l’istanza alla garbage collection non lo arresta: il backend continua a funzionare per lo script successivo e si arresta automaticamente dopo 30 minuti se non viene utilizzato da nessun altro processo. Al backend viene richiesto di spegnersi, quindi libera le telecamere e i sensori di luce e porta a termine eventuali registrazioni in corso; viene terminato solo se è ancora in esecuzione 10 secondi dopo. |
| `discover_cameras()` | Individua le telecamere LATTICE **tramite il backend di questa istanza** (`/api/camera/discover`). Restituisce un elenco di dizionari (`serial`, `model`, `ip`, …) — con la stessa struttura visualizzata dall’interfaccia grafica o dalla CLI. Elenco vuoto se non ne viene trovata nessuna o se il backend è irraggiungibile. |
| `camera_capture(output_dir, format="tiff", **settings)` | Acquisisce un singolo fotogramma**tramite il backend**(avviato automaticamente da questo handle) in modo che riceva la stessa preparazione della GUI o della CLI (impostazione predefinita a 12 bit, riutilizzo del pool, metadati di calibrazione incorporati). Risolvere il target con `serial=` o `device_index=`; passare `exposure`/`gain`/`pixel_format`/`preset` come `**settings`. Restituisce il dizionario dei metadati legacy (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Genera fotogrammi di anteprima composti tramite overlay da una telecamera in pool — client MJPEG leggero sul percorso `/api/camera/<serial>/stream-annotated` del backend (zebra / griglia / mirino / istogramma / peaking / punto disegnato lato server). `decode=True` restituisce array BGR e richiede OpenCV (`cv2`), che l’SDK non installa; `False` genera byte JPEG grezzi. Raggiungibile anche a livello di progetto come `ChlorosProject.stream(overlays=True)`. |

**Un’esecuzione in corso mantiene il proprio progetto.** Mentre è in corso un&#x27;esecuzione di elaborazione — di questo script, dell’app, di `chloros-cli process` o di `chloros-mcp` — il backend rifiuta di creare, aprire, importare o modificare le impostazioni di elaborazione di un progetto (409 HTTP `PROCESSING_IN_PROGRESS`): `create_project()` e `import_images()` generano l’eccezione `ChlorosProcessingError`, e `configure()` genera `ChlorosConfigurationError`, mentre `process()` con una modalità diversa da quella corrente genera `ChlorosProcessingError` (un&#x27;esecuzione mantiene la modalità con cui è stata avviata), ciascuna con il messaggio del backend. Una cattura salvata nel progetto viene quindi unita ad esso al termine dell&#x27;esecuzione (lo indica la risposta `project_add`). Attendere il completamento dell’esecuzione (`get_status()`), quindi proseguire.**Un progetto spostato.** L’`project.json` di un progetto assegna a ciascun file sorgente (ogni immagine e registrazione del sensore di luce) il proprio percorso completo; pertanto, spostando la cartella del progetto — o l’intera cartella dei progetti su un’altra unità — i file continuano a fare riferimento alla vecchia posizione. Ogni apertura corregge autonomamente ciò che può: un file che si trovava all’interno della cartella del progetto viene cercato nella cartella in cui si trova ora il progetto, e ogni percorso trovato lì viene reindirizzato e salvato (`project.json` viene prima salvato come copia di backup con il nome `project.json.before-relink-<date>-<time>`). Un percorso viene reindirizzato solo verso un file effettivamente presente in quella posizione. I file che si trovavano al di fuori della cartella del progetto — una sessione hub, un’importazione tramite “Aggiungi cartella” — vengono individuati solo con `relink=<the folder they are in now>` (o qualsiasi cartella a un livello superiore). `open_project()` restituisce `source_files`: `total`, `found`, `missing`, `missing_roots` (le cartelle in cui si trovavano i file mancanti), `missing_outside_project` e `relink` (cosa è stato reindirizzato, da dove a dove). Un progetto con alcuni file mancanti si apre con il resto; uno in cui non ne viene trovato nessuno non viene aperto, e `open_project()` genera l’errore `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). Anche gli errori `project open --relink` e `chloros-mcp` della CLI`open_project` (`relink_folder`) della CLI fanno lo stesso.

Utilizzare come gestore di contesto per garantire la pulizia:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Registrazioni del sensore di luce — calibrate `.daq` + `.csv`

È possibile registrare un DAQ-U / DAQ-M / DAQ-E **senza** il relativo pacchetto di calibrazione. Questo è
ciò che fanno per impostazione predefinita i registratori pubblici [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): scrivono i conteggi grezzi dei sensori e contrassegnano il
file in modo che Chloros recuperi la calibrazione di fabbrica di quel sensore **in base al numero di serie** — prima dalla cache locale
, poi dal cloud MAPIR — e la applica al momento dell’importazione.

Chloros riscrive il risultato come due prodotti per ogni registrazione, sotto
`<project>/Light Sensor/`:

| Prodotto | Che cos’è |
| --- | --- |
| `<name>_calibrated.daq` | L’archivio rielaborabile — stesso schema di una registrazione in tempo reale, ora con l’indicazione del bundle che lo ha generato. Reimportarlo **non** comporta una seconda calibrazione. |
| `<name>_calibrated.csv` | Irradianza spettrale in W/m²/nm sulla griglia di lunghezze d’onda propria del sensore, una riga per ogni lettura, più colonne fotometriche (potenza totale, lux fotopico/scotopico, PPFD e la sua suddivisione in blu/verde/rosso, lunghezza d’onda di picco). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Solo sensori senza bundle (DAQ-A).** Conteggi spettrali grezzi del sensore — *non* irradianza. Vedi sotto. |

`<name>` è il nome del file della registrazione stessa. Due registrazioni in cartelle diverse possono condividere
un unico valore di lettura per acquisizione (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) salvato nello
stesso millisecondo da due telecamere — e ciascuna ottiene comunque i propri prodotti: la prima mantiene
`<name>`, la successiva aggiunge le cartelle che la distinguono **dopo** il nome
(`<name>__<capture>_daq_calibrated.daq`), quindi la data e l’ora della registrazione rimangono comunque in primo piano.

`process()` esegue questa esportazione come una delle sue fasi. **Non** richiede immagini:
un sensore di luce utilizzato da solo costituisce un flusso di lavoro di prima classe, e un progetto di questo tipo non ha alcuna
immagine per definizione.

**Le registrazioni DAQ-A vengono esportate come conteggi grezzi.** La famiglia DAQ-A è antecedente al sistema di
bundle per serie e non ha alcun bundle da recuperare — viene invece calibrata sul campo rispetto a un
bersaglio di riflettanza, motivo per cui non ne ha mai avuto bisogno. Tali registrazioni vengono esportate
con un prefisso `_raw` anziché `_calibrated`: un nome file diverso anziché un flag
all’interno del file, poiché l’identificativo deve rimanere intatto anche se inviato via e-mail come semplice nome. L’intestazione
intestazione `.csv` riporta `raw spectral sensor counts (NOT irradiance)` e avverte che i
valori sono comparabili **all’interno** del file — esattamente ciò per cui serve la calibrazione basata su target
per utilizzarli — e non tra sensori diversi. Le colonne fotometriche dipendenti dalla potenza (potenza totale,
lux fotopico/scotopico, PPFD) restituiscono **NULL** anziché essere integrate dai conteggi.

Un DAQ-U / DAQ-M / DAQ-E il cui bundle semplicemente non è stato possibile recuperare viene comunque **saltato**,
non scritto in formato grezzo: in quel caso il bundle esiste e «riconnettersi e rielaborare» è un consiglio concreto.

Le registrazioni legacy **v1.01 / v1.02** (un DAQ-A-SD le scrive) non riportano un’epoca per ogni lettura,
ma solo l’ora di scrittura del file. Il matcher immagine↔luce in discesa continua a rifiutarle — abbinare un
frame con un’ora di scrittura comporterebbe un errore invisibile — ma l’esportatore le legge e il
file CSV stampa `clock=daq_created_on`, in modo che il prodotto indichi su quale orologio si trova.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Una registrazione il cui pacchetto di calibrazione non può essere recuperato (offline o sensore senza
calibrazione in archivio) viene segnalata con il codice `skipped` **indicandone il motivo**. Non viene mai
scritto come file “calibrato” contenente conteggi grezzi: connettersi a Internet ed
eseguire nuovamente l’operazione, e l’esportazione verrà completata.

### Callback di avanzamento

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Riepilogo post-esecuzione e suggerimenti

Al termine, `process()` recupera `GET /api/processing-summary` e allega il corpo come `result["summary"]`. Il recupero avviene secondo il principio del «best effort» e non blocca mai un ritorno con esito positivo — se il riepilogo non è disponibile, `process()` ricorre alla forma semplice `{"status": "complete", "async": False}`. Ogni voce in `summary["hints"]` — frasi complete con la correzione suggerita, ad es. perché un&#x27;esecuzione ha prodotto un output pari a zero — viene inoltre riemessa come `UserWarning` in Python, quindi le esecuzioni con output pari a zero sono autodiagnostiche anche se non si ispeziona mai il dizionario:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` è la parte leggibile dalla macchina:

| Chiave | Cosa conta |
| --- | --- |
| `models` | Gruppi di telecamere nell’esecuzione. |
| `images_in_groups` | Immagini sorgente in tali gruppi. |
| `targets_found` | Target di riflettanza rilevati. |
| `images_calibrated` | Immagini per le quali l’esecuzione ha generato un prodotto di riflettanza. |
| `images_failed` | Immagini che hanno dato esito negativo e non sono state esportate. |
| `exported_files` | **File dei prodotti immagine generati dall’esecuzione**: ogni prodotto (debayering, anteprima, radianza, riflettanza, indice, rendering dei target), in ogni formato. |
| `reflectance_files` | Tra questi, i prodotti di riflettanza (`Reflectance_Calibrated_Images`), indipendentemente dal fatto che siano state calibrate dal bersaglio o dall’irraggianza del sensore di luce (DAQ). Un’esecuzione senza bersaglio che ha comunque generato dati di riflettanza lo indica in un suggerimento; solo a un’esecuzione che non ne ha generati viene segnalata la mancanza degli output di riflettanza. |
| `clamp_masks_written` | Le [maschere di limitazione dell’unmix](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) riportate accanto ad esse — sono dati complementari, non prodotti, quindi non sono presenti in `exported_files` (la CLI, le conta separatamente allo stesso modo). |
| `captures_left_out` | Fotogrammi grezzi nella cartella `captures` del progetto che questa esecuzione non ha acquisito. Un&#x27;esecuzione, all&#x27;avvio, attende i dati acquisiti in precedenza; pertanto, questo valore è 0 a meno che non sia stata salvata un&#x27;acquisizione durante l&#x27;esecuzione o che il tempo di attesa sia scaduto. `summary["captures_left_out_note"]` indica come gestirli. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Registrazioni del sensore di luce, conteggiate separatamente di proposito — provengono da una fase diversa ed esistono per esecuzioni prive di immagini, quindi includerle farebbe sembrare che un’esecuzione solo DAQ abbia esportato immagini. |

Insieme a questi: `summary["output_dirs"]` (ogni directory in cui è stato scritto),
`summary["light_sensor_export"]`, `summary["stopped"]` (vero quando l’utente ha interrotto l’
esecuzione, in modo che i conteggi parziali non vengano interpretati come un&#x27;esecuzione completata con produzione insufficiente), e
`summary["groups"]` (la suddivisione per gruppo).

`exported_files` viene registrato dalla pipeline **mentre scrive**, non ricavato dagli
oggetti immagine del progetto in un secondo momento. Le strategie parallele e GPU creano i propri oggetti immagine
(nei sottoprocessi dei worker per i percorsi GPU), quindi la vecchia scansione riportava
`0 file(s) written` per ogni esecuzione di questo tipo ed emetteva quindi l’indicazione di esportazioni pari a zero — su esecuzioni
in cui tutto aveva funzionato. Se si crea uno script basato su questo numero, un’esecuzione parallela corretta ora
riporta un conteggio diverso da zero.

I saltati dal sensore di luce riportano il motivo effettivamente rilevato dal lettore per ciascun file — uno
schema illeggibile, un bundle mancante, un errore di scrittura — **deduplicato**, quindi venti file
saltati per un unico motivo vengono interpretati come un unico motivo anziché come venti ripetizioni dello stesso.

> **`process()` non viene generato quando un&#x27;esecuzione non produce immagini.** Questo è uno dei due punti in cui l’SDK e
> la CLI differiscono deliberatamente: `chloros-cli process` considera “i prodotti sono stati richiesti, ma nessuno è stato
> scritto” come un errore e termina con un valore diverso da zero, mentre l’SDK ritorna normalmente e segnala la
> condizione tramite `summary` / hints. Se la tua pipeline deve arrestarsi in caso di esecuzione vuota, controlla
> tu stesso — esamina `summary` (o conta i file nella cartella del progetto) anziché fare affidamento sull’
> assenza di un’eccezione. Le cause più comuni sono una cartella di input non riconosciuta come
> acquisizione e prodotti saltati in quanto non applicabili alle telecamere presenti (ad es. radianza da telecamere solo RGB
>).
>
> `process()` non viene generato nemmeno quando non è stato possibile verificare le esportazioni di un&#x27;esecuzione: emette un avviso e imposta
> `result["export_check_failure"]`. I prodotti vengono scritti. Per interrompere l&#x27;esecuzione in questo caso, controlla quella chiave. Lo stesso vale per un&#x27;esecuzione che
> ha perso un&#x27;immagine: emette un avviso e imposta `result["image_failure"]`, il quale fa sì che `chloros-cli process`
> restituisca un valore diverso da zero.

### Funzioni di utilità

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Ogni esecuzione avviene all’interno del proprio `with ChlorosLocal(...)`; un backend che doveva avviare continua a funzionare quando essa ritorna, per la chiamata successiva, e si arresta automaticamente dopo 30 minuti se non viene utilizzata da nessun altro processo. Nessuna delle due accetta `auto_start_backend`, `radiometric_output` o `daq_log_path`; per quelle utilizzare `ChlorosLocal` e `configure()` per quei casi. `process_lattice_capture()` non accetta `camera`, `debayer` o `ppk`: salta il rilevamento del pannello e utilizza il debayer standard.

### Valori supportati

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Uscita radiometrica (pipeline multispettrale LATTICE)

Il livello di esportazione multispettrale LATTICE (M3C/M3M) della pipeline `process` — `reflectance` (impostazione predefinita), `radiance`, `sensor-response` o `all` (ogni modalità applicabile per immagine) — imposta i prodotti **&quot;Esporta radianza&quot;**e**&quot;Esporta riflettanza&quot;****: `radiance` esporta solo la radianza, `reflectance` solo la riflettanza, `sensor-response` nessuna delle due (solo i prodotti debayered e di anteprima), mentre `all` esporta entrambi. I prodotti debayered e di anteprima non possono essere modificati a questo livello. `configure()` dispone di una parola chiave dedicata:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

La via d’uscita avanzata — la scrittura dei prodotti tramite `custom_settings` — funziona ancora, ma ricordate che sostituisce l’intero blocco delle impostazioni (vedere l’avviso qui sotto). Scrivere le chiavi `"Export <product>"`: le impostazioni specifiche di un progetto, ovvero `"Export radiance"` / `"Export reflectance"` di un progetto hanno la precedenza sulla vecchia chiave `"Radiometric output"`, quindi quella chiave da sola non modifica nulla.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (impostazione predefinita) divide la radianza della telecamera per il **flusso in discesa DAQ abbinato al timestamp**, risolto automaticamente da un `.daq` (DAQ-U/M/E) registrato**o da un `.csv` nativo DAQ-M**presente insieme alle immagini; qualsiasi pacchetto di calibrazione per singola telecamera o DAQ mancante localmente viene**recuperato automaticamente da AWS** al primo utilizzo. La CLI lo espone secondoopzioni di tipo “product” su `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Di chi è la lettura?**Una lettura salvata in un `.daq` corrisponde alla lettura della o delle telecamere che descrive — indicate in un sidecar `<file>.daq.cameras.json` adiacente ad essa (acquisizioni singole, sincronizzate e in array; una raffica raw indica la lettura di ciascun elemento nel proprio manifesto) — ed è abbinata**esclusivamente alle immagini provenienti da quelle telecamere**. Una telecamera autonoma priva di una propria lettura, pertanto, non utilizza più il DAQ di un’altra telecamera; è necessario associarle un DAQ oppure registrarne uno. La registrazione propria di un DAQ (`chloros-cli daq pool-record`, o una registrazione avviata nell’interfaccia grafica), o qualsiasi `.daq` privo di quel file sidecar (comprese le acquisizioni più vecchie), continua a servire tutte le telecamere.**Telecamere OCN alla luce del sole.** Ogni pacchetto di calibrazione OCN viene creato in condizioni di luce a banda stretta, con un LED per banda, mentre la luce solare mescola le bande. Chloros 1.3.0 applica una correzione di base per la luce solare dopo la separazione delle bande, il che migliora i prodotti OCN relativi a radianza, riflettanza e indice in condizioni di luce solare. Funziona ovunque avvenga la separazione delle bande — `process()`, acquisizioni, `grab()` / `frame_stream()` e la visualizzazione in tempo reale — per i pacchetti OCN che la supportano; le fotocamere RGN rimangono invariate. Anche le immagini OCN acquisite con una versione precedente ne beneficiano quando vengono elaborate su un computer che conserva la calibrazione con cui sono state scattate (Chloros mantiene la calibrazione di una fotocamera una volta che questa è stata collegata); in caso contrario, vengono elaborate come in precedenza. I valori di radianza, riflettanza e indice OCN differiscono quindi da quelli della versione 1.2.0.

> `custom_settings` **sostituisce** l’intero blocco delle impostazioni calcolate (per come è progettato, ignora le altre parole chiave e la convalida di `configure()`). Quando lo si utilizza, includere ogni chiave `Project Settings` di interesse, come nell’esempio sopra riportato.

#### Lettura dei pixel dell’indice (NDVI e gli altri indici di vegetazione)

**Un raster di indice intero rappresenta un&#x27;estensione del contrasto per singola immagine, non una mappa nell&#x27;intervallo da −1 a 1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` e `JPG (8-bit)` — compreso il file TIFF a 8 bit in cui viene salvata un’acquisizione a 8 bit — memorizzano

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

dove `IndexMin` / `IndexMax` sono **i valori di indice finiti più basso e più alto propri di quel fotogramma**. DN 0 non è −1 e il fondo scala non è +1.

Il Chloros appone entrambi i valori limite in modo che la mappatura rimanga invertibile:

| Tag | Significato |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — la convenzione riportata di seguito |
| `Chloros:IndexMin` | il valore dell’indice memorizzato come DN 0 |
| `Chloros:IndexMax` | il valore dell’indice memorizzato come scala completa |

Leggerli e invertirli, normalizzando il DN memorizzato a un valore compreso tra 0 e 1 in base al massimo del tipo di dati (dtype) memorizzato, in modo che un&#x27;unica espressione copra ogni formato intero:

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

> **`IndexMin == IndexMax`** significa che ogni pixel di quel fotogramma conteneva lo stesso valore di indice. Il raster è riempito a metà scala e non c’è nulla da dividere — si prenda il valore estremo.

> **L’assenza di tag `Chloros:Index*` indica che non è stata specificata alcuna mappatura.** Un’esportazione LUT è un rendering colorato i cui pixel presentano colori sfumati associati alla finestra di soglia della LUT; un fotogramma privo di pixel con indice finito non ha un intervallo da specificare. Non dare per scontata alcuna mappatura in entrambi i casi — vale la stessa regola di un `PixelScale` mancante.**Poiché l’allungamento è per singola immagine, due fotogrammi di uno stesso volo non sono comparabili in termini di DN**— ciascuno porta i propri punti estremi, quindi lo stesso NDVI ricade su DN diversi in immagini diverse. Per lavori quantitativi su un volo (statistiche zonali, soglie fisse, serie temporali), esportare**`TIFF (32-bit, Percent)`**: i raster di indice vi vengono scritti come float32**i valori di indice stessi**, senza necessità di riscalatura né di tag. Nonostante il nome del formato, i prodotti di indice non sono ×100 — tale scalatura si applica solo alla riflettanza con tipo di dati intero.

#### Maschera di limitazione della separazione spettrale — su quali pixel una banda è stata limitata

La separazione spettrale 3×3 può rendere negativa una banda — laddove la luce contiene a malapena quella banda, o dove la scena ricade al di fuori di ciò che descrive la matrice di separazione della fotocamera. Non si tratta di un valore reale, quindi l’esportazione lo fissa a 0 — e la visualizzazione in tempo reale mostra lo stesso 0. Una banda può essere limitata su pochi pixel o su quasi tutti; la maschera registra entrambe le situazioni. `Chloros:Band{R,G,B}UnmixClampFraction` indica su quale porzione del fotogramma ciò si è verificato; la maschera di limitazione indica **quali pixel**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— in una cartella `clamp_masks/`**accanto** alla cartella del prodotto, mai al suo interno (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — per ogni prodotto di radianza, riflettanza e indice in cui almeno un pixel è stato sottoposto a clamping. `uint8` a banda singola, sulla griglia di pixel del prodotto stesso (orientamento, ROI, binning e la distorsione di allineamento dell’array inclusi). Ogni maschera indica la fotocamera del proprio prodotto (`Make` / `Model` / numerazione seriale), l’ora di acquisizione e i tag dell’obiettivo, ed elenca le bande a cui corrispondono i suoi bit (`XMP-Camera:BandName`, bit 0 per primo); i suoi `ImageDescription` e `Software` la contrassegnano come maschera di clamping e non contiene alcuna calibrazione.
- **Bit**: bit 0 = la banda nel canale R del prodotto (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Un bit impostato indica che il valore di quel pixel in quella banda è stato limitato a 0 — considerarlo, così come qualsiasi indice da esso derivato, come a bassa affidabilità.
- **`Chloros:UnmixClampMask`** è apposto su ogni prodotto su cui è stata eseguita la separazione: il percorso della maschera relativo alla cartella del prodotto stesso (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, sempre separato da `/`-separato — unirlo alla cartella dell’immagine per aprire la maschera), oppure `none` quando nessun pixel è stato limitato. L’assenza di tag indica che il prodotto non ha mai eseguito l’unmix (telecamere RGB o monocromatiche, livelli raw/debayered).
- I risultati di acquisizione lo riportano come **`clamp_mask`** del fotogramma (il percorso della maschera, oppure `None` quando non c’è alcun pixel clippato) — nei risultati `CameraSession.capture()`, `ArraySession.capture()`&#x27;nei fotogrammi `CaptureResult` e nelle risposte di acquisizione HTTP. La build offline elenca le sue maschere come `clamp_masks`.
- Lo stack multibanda di un indice live (`/api/camera/cli-array-index` con `save_multiband`) ottiene una maschera per ogni telecamera sorgente, nella cartella di output `clamp_masks/` — si veda il riferimento CLI&#x27;`lattice index`. Un video non ha alcuna maschera: il risultato della build offline riporta invece la frazione di clamp di ciascun fotogramma.

Poiché nessuna cartella del prodotto contiene una maschera, a qualsiasi livello di profondità, una cartella del prodotto può essere consegnata al software di fotogrammetria così com’è — sottocartelle incluse — e questo accetta solo le immagini. Consegnategli la cartella del prodotto, non la cartella sovraordinata. La procedura di importazione di Chloros ignora le maschere.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect per telecamere LATTICE

Sessioni backend persistenti per hardware in tempo reale. Gli stessi endpoint utilizzati dall’interfaccia grafica, quindi il comportamento è identico tra SDK, CLI e GUI.

### Singola telecamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

L’esposizione, il guadagno e il bersaglio impostati qui vengono mantenuti come da te specificati: il controllo automatico dell’esposizione di Chloros (compresa l’esposizione misurata dal DAQ) non li modifica. `cam.set_settings(exposure_auto="Continuous")` (o `gain_auto=`) restituisce l’esposizione (guadagno), mentre `cam.set_settings(target_brightness_auto=True)` restituisce il target al sistema automatico di Chloros.

#### `connect_camera()` Firma

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Ritorna con lo stream avviato.** Il backend attende, entro un limite (circa 15 s per l’avvio dello stream, poi 10 s per il suo primo buffer completo — non per il primo fotogramma di anteprima, che segue circa 1,5 s dopo), quindi un `status()` subito dopo indica `streaming`. L’`connect_stream` della sessione è il blocco `stream` della risposta: `state` `streaming`, `no_frames` (il flusso è iniziato ma non ha trasmesso nulla — lo indica `note` e, per una telecamera con pacchetti jumbo, di riconnettersi con una dimensione del pacchetto di 1500 byte), `starting` o `start_failed`, con `waited_s`. `None` per una telecamera già connessa. Una (`get_camera_preview`, `/api/camera/jpeg/<serial>`) richiesta subito dopo un&#x27;attesa, limitata a 4 secondi, per il primo fotogramma di anteprima della telecamera, invece di rispondere &quot;Nessun fotogramma ancora disponibile&quot;; una telecamera il cui flusso non ha ancora trasmesso nulla (`no_frames`, o attivata da un trigger hardware) risponde comunque immediatamente.
- **La dimensione del pacchetto la decidi tu.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` apre la telecamera con pacchetti da 1500 byte, senza effettuare il test (`chloros-cli lattice cam-connect --packet-size standard` invia lo stesso).
- **Una sola connessione alla volta mantiene attivo il pool delle telecamere**: l’apertura, la verifica della dimensione del pacchetto, un primo download di calibrazione (circa 70 s per una telecamera nuova su questo computer), l’avvio del flusso. Una richiesta che necessita di una telecamera nel frattempo (`read_nodes`, `set_settings`, una cattura, una seconda connessione) la attende al massimo 5 s, poi fallisce con codice HTTP 503 `CAMERA_POOL_BUSY`, indicando la telecamera che si sta connettendo (`ChlorosConnectError`); riprova di nuovo tra un attimo. Gli elenchi non attendono mai: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` e `CameraSession.status()` rispondono immediatamente. Una telecamera in fase di connessione viene inserita nell’elenco sotto la risposta del pool `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — e `elapsed_s`), e il suo `status()` indica `connecting: True` con il suo `phase`; le altre telecamere vengono lette in tempo reale quando il pool è libero, altrimenti viene restituito il valore dell’ultima lettura (`live: False`).

#### Metodi `CameraSession`

| Metodo | Descrizione |
| --- | --- |
| `status(timeout=15.0)` | Lo stato in tempo reale della telecamera, così come viene letto da `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` o `stopped`), `device` (modello, numero di serie, firmware, temperatura, `is_mono`) e le sue impostazioni correnti. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Legge i nodi GenICam; restituisce `{nodes, errors, enums, device, was_in_pool, in_array_id}`, inoltre `frame_values` e `notes` mentre è in corso l’esposizione automatica (vedi sotto). |
| `set_settings(**kwargs)` | Scrive i nodi in base al nome descrittivo (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Un `exposure_time` (`gain`) indica un&#x27;esposizione (guadagno) manuale: se scritto senza `exposure_auto` (`gain_auto`), la modalità automatica viene prima disattivata, in modo che il valore non venga rifiutato da una modalità automatica in esecuzione. Il risultato, `applied`, contiene ciò che la fotocamera ha rilevato; una scrittura rifiutata dalla fotocamera si trova in `errors` e non viene conservata. L&#x27;errore indica l&#x27;intervallo attualmente accettato dalla fotocamera (un valore massimo in `ExposureTime` dipende dalla frequenza dei fotogrammi della fotocamera), e viene ripristinata una modalità automatica che era stata disattivata solo per il valore rifiutato; `restored` lo elenca (ad es. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Cattura un **singolo** fotogramma. Restituisce un `CaptureResult`: un elenco di dizionari di metadati dei fotogrammi (uno per ogni livello salvato), con `.skipped`, `.failed`, `.daq_files` e `.daq_matches`. (L&#x27;acquisizione in raffica/multi-fotogramma è stata rimossa — chiamare `capture()` in un ciclo se è necessaria una serie.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Acquisizioni consecutive (`interval_s=0`) o a intervalli da questa fotocamera, per acquisizioni `count` e/o `duration_s` (almeno una, in modo che termini), così come fa `ArraySession.capture_repeated()` per un array. Restituisce l’elenco degli `CaptureResult`; si interrompe dopo un’acquisizione che segnala un’unità piena. `on_capture(i, result)` viene eseguito dopo ciascuna di esse; le altre parole chiave vengono indirizzate a `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Rilascio dal pool; la telecamera viene riportata allo stato in cui si trovava al momento della connessione (cablaggio del trigger, PTP, impostazioni); `keep_settings=True` salva invece le proprie impostazioni nella propria memoria, mentre `clear_settings=True` resetta tale memoria (vedi [Memoria della telecamera](#camera-memory--keep_settings--clear_settings)); restituisce `camera_memory` quando viene richiesta una delle due operazioni. Una telecamera che è membro di un array libera l’intero array. Operazione nulla se ci si è collegati a una sessione già aperta. Restituisce il risultato una volta che il backend conferma il rilascio, oppure quando non c’è nulla da rilasciare (la telecamera è già stata rimossa o nessun backend è in esecuzione); un rilascio che ha superato il timeout o che è stato rifiutato dal backend genera l’errore `ChlorosConnectError`, poiché la telecamera potrebbe essere ancora occupata e la connessione successiva entrerebbe in conflitto con essa. L’uscita dal blocco `with` non genera mai un errore. |

Attributi: `serial`, `device` (il record di rilevamento della telecamera) e `backend_url`.

Controlli di esportazione `capture()` (stesso modello dell’array + GUI):

- `processing` / `levels` — `processing="all"` salva tutti i tipi di esportazione applicabili; `levels=["raw","radiance"]` salva solo quelli (sovrascrive `processing`). Omettete entrambi per utilizzare l’impostazione predefinita del backend.
- `daq` — la lettura DAQ della cattura utilizzata viene salvata per impostazione predefinita come sidecar `.daq`, con inclusa una cattura solo in formato raw, in modo che il fotogramma possa essere rielaborato in seguito in riflettanza/indice. `daq=False` lo ignora quando nessun livello lo utilizza. `force_daq=True` è la vecchia grafia dell’impostazione predefinita. Nessun DAQ, né `.daq`.
- `daq_tolerance_ms` — di quanto la lettura può discostarsi dall’esposizione pur rimanendo entro i limiti di tolleranza (valore predefinito 500). Cambia solo il verdetto.

#### `read_nodes()` con esposizione automatica — `frame_values` e `notes`

Mentre `ExposureAuto` (`GainAuto`) è in esecuzione, il nodo `ExposureTime` (`Gain`) rilegge l’**ultimo valore scritto**, non ciò che la fotocamera sta effettivamente facendo. `nodes` mantiene ancora quel valore del nodo. Quando si richiede `ExposureTime` o `Gain` su una telecamera collegata la cui modalità automatica non è `Off`, la risposta aggiunge:

- `frame_values` — il valore contenuto nel fotogramma più recente della telecamera, sotto il nome del nodo (`ExposureTime` in µs, `Gain` in dB);
- `notes` — una riga per ciascun nodo di questo tipo che riporta entrambi i valori.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Entrambe le chiavi sono assenti quando la modalità automatica è `Off` (il nodo rappresenta in tal caso il) e per una fotocamera non collegata al backend. Per un array, `ArraySession.status()["member_ae"]` fornisce contemporaneamente i valori dei fotogrammi di ogni elemento.

#### Limite massimo di esposizione misurato dal DAQ (HTTP)

Con un sensore di luce DAQ collegato, Chloros limita l’esposizione × guadagno di una telecamera in base alla luce diurna misurata, in modo che un pannello di riferimento bianco non vada in clipping. Il limite massimo si applica solo mentre un bersaglio di calibrazione si trova nel campo visivo di quella telecamera (rilevato nel suo fotogramma negli ultimi 10 s); in assenza di bersaglio nel campo visivo, viene invece attivata la normale esposizione automatica della telecamera, con la propria protezione dalle alte luci, in modo che il resto della scena non risulti più scuro del necessario. Non esiste un wrapper SDK; è possibile leggerlo o modificarlo per ogni singola telecamera tramite `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (il corpo del POST `{"enabled": false}` lo disattiva per quella telecamera). La risposta contiene `enabled`, `cap_us` (il limite massimo in tempo reale, `None` quando inattivo, anche quando non è visibile alcun bersaglio di calibrazione), `k_exp_g` (la costante con cui viene calcolato il limite massimo) e:

| Campo | Significato |
| --- | --- |
| `k_exp_g_source` | Quale costante è in uso: `camera` — quella propria di questa telecamera, calcolata dal punto di riferimento nel suo pacchetto di calibrazione (telecamere RGB); `fleet` — la costante della flotta, utilizzata quando il pacchetto non contiene alcun punto di riferimento; `override` — un’impostazione di sovrascrittura impostata sul backend. |
| `k_exp_g_fleet` | La costante della flotta, a titolo di confronto. |

### Array sincronizzato — `ArraySession` (Smart-Prep)

`connect_array` è **il punto di partenza consigliato** per le configurazioni multicamera. Esegue in background l’intero flusso Smart-Prep tramite interfaccia grafica:

1. **Analisi di rete**(`/api/camera/array/recommend`) — imposta il**sensore completo** (senza binning, senza ritaglio ROI) e lascia che la frequenza dei fotogrammi scenda al livello garantito da questo host e dalla rete, in base ai formati pixel e al packing che Connect utilizzerà. I fotogrammi non vengono mai ridotti per mantenere alta la frequenza: passare un valore più piccolo per `width` / `height` o `binning=2` / `4` per sacrificare la risoluzione a favore della frequenza dei fotogrammi (l’endpoint`alternatives` elenca tali opzioni con i relativi fps previsti).
2. **Selezione automatica del livello** — `sim-capture-sim-emit` (tutte le telecamere espongono allo stesso trigger, con emissione a intervalli regolari per condividere la linea, e la frequenza dei fotogrammi diminuisce man mano che la condivisione si riduce) su qualsiasi rete in cui le telecamere possano essere sincronizzate per condividere la linea; `slip-emit-and-capture` (le telecamere si alternano sulla linea, senza esposizioni simultanee) solo quando ciò non è possibile.
3. **PTP abilitato**per impostazione predefinita — i timestamp tra le telecamere si allineano su un orologio condiviso con una precisione di**~1 ms**, il [grandmaster PTP](#ptp-grandmaster--time_sync_status) del backend. L’esposizione simultanea deriva dal trigger hardware M8 (**&lt; 100 µs** tra i moduli), non dal PTP: il PTP allinea i *timestamp*, non le esposizioni.
4. **Selezione automatica del formato pixel per ogni telecamera** — mono → `Mono12`, colore → `BayerRG12`.
5. **Inizializzazione AE** — acquisisce lo stato AE corrente di ciascuna telecamera in modo che la connessione non resetti l’esposizione durante il funzionamento.
6. **Configurazione del trigger GPIO** — `connect_array` arma ogni telecamera (`TriggerMode=On`, `TriggerSource=Line2`) in modo che l’impulso del masterpulsazione del master comandi gli slave tramite il cavo M8. Questo è un passaggio valido solo per gli array: una singola telecamera avviata con `LatticeCamera` funziona invece in modalità libera. Rilasciando l’array (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) riporta tutte le telecamere in modalità di funzionamento autonomo (`TriggerMode=Off`), mantenendo tutte le altre impostazioni così come rilevate al momento della connessione: le impostazioni di trigger e della linea di sincronizzazione (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; la linea di sincronizzazione della fotocamera principale riportata su un ingresso), PTP e le impostazioni della fotocamera modificate durante la sessione — a meno che `keep_settings=True` / `clear_settings=True` (vedere [Memoria della telecamera](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Attributi: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` e `packed_reason` (gli ultimi quattro sono `None` su un handle proveniente da `attach_array`), `connect_stream` e `backend_url`. `connect_stream` è il blocco `stream` della risposta alla connessione, la forma della singola connessione più `members` (`{serial: {state, why?}}`): `streaming` una volta che è arrivato il primo gruppo sincronizzato dell’array (il backend lo attende, entro un limite di circa 10 s), oppure `no_frames` con l’ultimo errore di recupero di ciascun membro come `why` e, per i pacchetti jumbo, il fallback da 1500 byte nell’`note`. `None` su un handle `attach_array`.

#### Firma `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` indica per quanto tempo la chiamata attende la connessione e viene inviato al
backend come termine ultimo. Se la connessione non viene stabilita entro tale termine, l’operazione viene annullata: ogni telecamera
aperta viene riportata allo stato in cui si trovava al momento della chiusura, nessun array viene connesso e la
chiamata genera l’errore `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, indicando la
fase raggiunta dalla connessione). Pertanto, una chiamata che genera un&#x27;eccezione non lascia mai un array
connesso alle sue spalle. Un array di 9 telecamere si connette in circa un minuto; subito dopo
l’accensione delle telecamere possono volerci diversi minuti. Viene restituito un array che è attivo ma non ha
ancora un gruppo di fotogrammi sincronizzato, con `connect_stream["state"]`
`no_frames`. Il log del backend contiene una riga di temporizzazione per ogni fase di connessione
(`[array-connect <id>] <step>: <s> s …`) e un’ultima riga che indica la più lenta.

Valori `force_tier`:
- `"sim-capture-sim-emit"` — vera simultaneità (tutte le telecamere scattano sullo stesso fronte di clock).
- `"sim-capture-ftd-stagger"` — sfalsamento flessibile nel dominio del tempo (le camme trasmettono in momenti leggermente sfalsati, in modo che i pacchetti vengano serializzati sulla linea).
- `"slip-emit-and-capture"` — acquisizione sequenziale per singola camma (senza sincronizzazione temporale). Selezionato automaticamente solo quando non è possibile rallentare le telecamere abbastanza da consentire loro di condividere la linea contemporaneamente.

`wire_ceiling_mbps` sovrascrive il **budget di banda sostenuto dall’host** in MB/s — l’unico
valore da cui dipende l’allocazione dell’intero array. Lasciarlo su `None` per utilizzare il valore rilevato automaticamente
. Ridurlo quando l’array segnala frame danneggiati da GVSP: il valore automatico è derivato
dalla velocità di collegamento pubblicizzata dalla scheda di rete, che sovrastima gli adattatori USB, le linee PCIe sottodimensionate e
i fabric condivisi molto trafficati — e tale sovrastima si manifesta come frame danneggiati anziché come un
collegamento visibilmente lento. Il valore viene salvato nel blocco di acquisizione dell’array del progetto, quindi una
riapertura o un successivo `connect_array` lo ripristina come qualsiasi altra impostazione dell’array.
Vedi [Stato di salute dell’array](#array-health--which-subsystem-is-losing-frames).

#### Trasporto compresso a 12 bit (`packed=`)

A piena capacità dei sensori, la frequenza dei fotogrammi di un array è determinata dalla velocità con cui ciascuna telecamera può inviare
i propri fotogrammi. I formati a 12 bit (`BayerRG12`, `Mono12`) vengono normalmente trasmessi come parole a 16 bit;
il trasporto compresso invia gli stessi 12 bit in 1,5 byte per pixel e
li decomprime sull’host, quindi l’invio di ogni fotogramma richiede il 25% di tempo in meno — circa
**il 33% in più di fotogrammi al secondo a piena capacità del sensore** (misurato: da 10,7 a 14,0 fps su un
array di 9 telecamere). **I dati salvati sono identici**: stessi valori, stessi file a 16 bit
, stessi metadati.

- `packed="auto"` (impostazione predefinita) esegue la compressione solo quando è utile **e** l’host è in grado di
  gestire la decompressione: al momento della connessione, il backend misura la propria velocità di decompressione,
  il numero di CPU, il carico attuale e la larghezza di banda di rete disponibile. Un desktop con core disponibili
  comprime a piena capacità del sensore; un host occupato o a bassa potenza (computer a scheda singola,
  controllori embedded) trasmette in streaming senza compressione. Le piccole ROI e i fotogrammi raggruppati in bin ottengono pochi vantaggi.
- `packed="off"` (o `False`) non comprime mai — per escludere questa opzione durante la diagnostica,
  o su un host che deve riservare la propria CPU ad altre attività.
- `packed="on"` (o `True`) comprime sempre i formati a 12 bit.
- Qualsiasi altro valore genera `ValueError`. I formati a 8 e 16 bit non vengono mai
  compressi.

`analyze_array_network(..., packed="auto")` opera con la stessa logica,
quindi il suo fps corrisponde alla velocità alla quale l’array verrà eseguito (la risposta contiene
`packed_transport` e `packed_reason`); il suo valore predefinito `pixel_format` è quello
applicato da «connect» (Mono12 / BayerRG12) anziché 8 bit.

La decisione e la relativa motivazione si trovano nella sessione:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` applica la stessa sovrascrittura a ogni
array di un progetto (impostazione predefinita `None`, il che significa `"auto"`: un progetto non salva alcuna
impostazione di compressione per array).
Per un&#x27;impostazione predefinita a livello di macchina, impostare `CHLOROS_ARRAY_PACKED12=off` (o `on`) nell&#x27;
ambiente del backend; un valore esplicito `packed="on"` / `"off"` ha comunque la precedenza.
`/api/camera/array/status` riporta `packed_transport` e `packed_decision`.

#### Frequenze di fotogrammi previste

Frequenze di fotogrammi dell’array misurate (sincronizzate via hardware, 12 bit; sensore completo = 2048 × 1536). Utilizzateli per verificare la correttezza di `status()["actual_fps"]`; `analyze_array_network()` proietta la frequenza per il vostro host e la vostra rete.

| Configurazione | Frequenza dei fotogrammi dell&#x27;array |
| --- | --- |
| Sensore completo, scheda di rete host PCIe 10 GbE, trasporto compresso (`packed="auto"` su un computer desktop) | ~14 fps |
| Sensore completo, scheda di rete host PCIe 10 GbE, senza compressione (`packed="off"`) | ~10,5 fps |
| Sensore completo, 8 telecamere su un adattatore USB 10 GbE | ~3,8 fps |
| Sensore completo, collegamento host 1 GbE | 1,2–1,7 fps |
| Binning 2 × 2 (`binning=2`), scheda di rete host PCIe 10 GbE | ~30–33 fps |

- Su un collegamento PCIe 10 GbE, a sensore completo, sono le **telecamere** a costituire il limite, non la rete: 4 e 8 telecamere funzionano alla stessa velocità.
- **`target_fps` rappresenta la frequenza di trigger e l’array non funziona mai a una velocità superiore.** `target_fps=1` fa funzionare l’array a 1 fps; `None` (l’impostazione predefinita) lo fa funzionare alla frequenza massima supportata dall’host e dalla rete. Qualsiasi valore superiore a tale frequenza viene ridotto a essa (vedi [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` di `4294` in `status()` non è una velocità di collegamento.** Ciò significa che Windows non è riuscito a leggere la velocità dell’adattatore (il driver ha riportato il valore massimo a 32 bit), cosa tipica degli adattatori Ethernet USB. La velocità utilizzabile di un adattatore USB su Windows è comunque limitata a 200 MB/s (vedere [Stato dell’array](#array-health--which-subsystem-is-losing-frames)); per ottimizzarne uno, consultare la sezione &quot;Schede USB 10GbE&quot; della guida alla CLI alla voce [Configurazione e ottimizzazione della scheda di rete host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Un limite USB si applica anche su Linux: 400 MB/s su x86_64, 200 MB/s su NVIDIA Jetson.** L’adattatore viene identificato tramite sysfs (ricorrendo a `ethtool -i`), pertanto un adattatore USB 5 GbE o 10 GbE viene considerato entro il limite massimo piuttosto che in base alla velocità nominale di 500 / ~1060 MB/s. Il valore per x86_64 è stato misurato su Linux: lo stesso adattatore che su Windows corrompe i frame oltre i 200 MB/s ha gestito senza problemi un array di 4 telecamere a ogni limite massimo fino a 480 MB/s. Su NVIDIA Jetson lo stesso adattatore ha funzionato correttamente solo a 200 MB/s e ha perso frame in modo intermittente al di sopra di tale soglia. Se il vostro host gestisce una velocità maggiore senza perdita di frame con `gvsp_corrupt`, aumentalo con `wire_ceiling_mbps`. Su Linux, a un adattatore collegato a una porta USB 2.0 sono assegnati 30 MB/s; collegalo a una porta USB 3.

#### Sovrasottoscrizione (il limite minimo per telecamera)

Un array funziona su qualsiasi rete; è la frequenza dei fotogrammi a determinare il limite. Il pacing Sim-emit assegna a ciascuna telecamera una quota uniforme della larghezza di banda a prova di collisione e tutte continuano a trasmettere allo stesso trigger; pertanto, su una linea lenta, la quota — e la frequenza dei fotogrammi — è ridotta: 8 telecamere aa 12 bit su una rete 1 GbE senza jumbo frame ottengono 6,5 MB/s ciascuna, circa **1 fps**(circa 1,3 fps compressi). La quota ha un limite minimo pari alla velocità più bassa alla quale le telecamere possono essere sincronizzate (`per_cam_floor_bps`: si legge dall’intervallo GevSCPD di ciascuna telecamera al momento della connessione, e non scende mai al di sotto di 1 MB/s). Se la linea richiede una velocità inferiore, la connessione ricorre alla trasmissione**seriale**(`slip-emit-and-capture`): le telecamere si alternano alla velocità massima del collegamento, quindi ogni frame arriva ma i membri**non vengono esposti simultaneamente**— il livello di risposta e i blocchi (`SERIAL: ...`) lo indicano; passare `force_tier="sim-capture-sim-emit"` per rifiutare il ripiego. La dimensione del fotogramma non influisce sulla condivisione, quindi**il binning e la ROI aumentano la frequenza dei fotogrammi ma non modificano la capacità di un array di rientrare**. Solo una linea che non è in grado di trasportare l’array a nessuna velocità, né simultaneamente né a turno, va in**sovrasottoscrizione** — la modalità di errore è la perdita di pacchetti GVSP, non una frequenza dei fotogrammi inferiore. Rimedi: meno telecamere, fotogrammi jumbo end-to-end o una scheda di rete più veloce.

- Le risposte `analyze_array_network()` e `/api/camera/array/connect` trasportano `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` e `per_cam_floor_bps`. Quando `oversubscribed` è vero, la proiezione **azzeri i campi fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) anziché riportare una frequenza fuorviante, che indica un funzionamento lento ma comunque attivo.
- `POST /api/camera/array/connect` accetta un parametro body `pin_resolution` (**solo HTTP — non è un kwarg dell’SDK**; `connect_array` non lo espone). Si tratta di una modalità rigorosa: una connessione sovraccarica con `pin_resolution` impostato viene**rigorosamente rifiutata** con un errore che indica ogni possibile soluzione. In assenza di tale impostazione, la connessione procede con un avviso che indica che l’array perderà dei pacchetti. In entrambi i casi, la connessione mantiene la dimensione del frame richiesta — non raggruppa né ritaglia mai autonomamente.
- Solo a scopo di test: impostare `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` nell’ambiente del backend per ridurre il rifiuto a un avviso evidente — ci si connette comunque e si accetta la perdita di pacchetti.

#### Stato dell’array — quale sottosistema sta perdendo frame

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) gestisce un blocco `health` attivo su un
array connesso, rivalutato su una **finestra**mobile di**10 secondi** . Suddivide la perdita di frame
nelle due cause che richiedono soluzioni opposte, invece di un unico tasso di “incompletezza” che
non ne identifica nessuna:

| Campo | Cosa significa | Quale sottosistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriale) | Il fotogramma **è arrivato ma era strutturalmente errato**— perdita di pacchetti GVSP. |**Rete**: larghezza di banda disponibile, pacing, anello RX della scheda di rete, MTU |
| `never_arrived_rate_pct` (per seriale) | Il frame **non è mai arrivato**— la telecamera non si è attivata, oppure non è stato trasmesso nulla. |**Trigger / sincronizzazione**: cavo M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Tasso peggiore delle telecamere per ciascuna. | — |
| `per_cam_rate_pct` | Tasso combinato di incompletezza per fotocamera (entrambe le cause insieme). | — |
| `stable_for_seconds` | Per quanto tempo ogni fotocamera è rimasta al di sotto dello 0,01%. | — |

Insieme a `health`, lo stesso record riporta il numero relativo all’intera allocazione in sospeso:

| Campo | Significato |
| --- | --- |
| `wire_ceiling_mbps` | Il budget di banda effettivo dell’host, in MB/s. |
| `wire_ceiling_source` | Da dove proviene quel valore, in parole — ad es. `USB-capped 200 MB/s (was theoretical 1062; …)` o `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` quando è stato impostato da `wire_ceiling_mbps=`. |
| `nic_is_usb` | `true` per un adattatore Ethernet USB. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Interpretazione:** un valore diverso da zero per `gvsp_corrupt_rate_pct` con `never_arrived_rate_pct` pari a 0 indica che
il trigger e la sincronizzazione via cavo sono perfetti e che il 100% della perdita si verifica sul percorso di rete — abbassare
`wire_ceiling_mbps` e ricollegare. Il modello inverso indica invece il cavo di sincronizzazione o la
linea di trigger.

> **`target_fps` non è l’indicatore dei frame danneggiati.** Il pacing di GevSCPD viene impostato una sola volta al
> momento della connessione, quindi abbassare la frequenza di trigger modifica il ciclo di lavoro e non la
> velocità di trasmissione in burst simultanea. Una riduzione misurata della richiesta del 5× non ha prodotto alcun miglioramento, mentre
> l’abbassamento del limite massimo della linea da 240 a 200 MB/s ha portato lo stesso sistema dal 10,4 % di frame danneggiati allo
> 0,00 %.

> **Un array in esecuzione non può ripianificarsi autonomamente sul firmware LATTICE (M3C / M3M).** Non può
> risolvere autonomamente il problema; scollegare e ricollegare in modo che il selettore del tempo di connessione si ripianifichi in base
> al nuovo limite massimo.

Un **adattatore Ethernet USB è limitato a 200 MB/s** dalla sonda indipendentemente dalla sua
denominazione: la tabella di efficienza che converte la velocità di collegamento in un valore sostenuto è
derivata dallo standard PCIe, mentre una scheda di rete USB dichiara la propria velocità di collegamento Ethernet pur essendo limitata dal
bus USB e dal relativo driver. Il limite è assoluto, non una frazione: un adattatore USB 1 GbE
raggiunge circa 80 MB/s e non ne risente. Si applica su Windows e Linux (Jetson incluso);
vedere [Frequenze di frame previste](#expected-frame-rates) per sapere dove è stato misurato.

#### Metodi `ArraySession`

| Metodo | Descrizione |
| --- | --- |
| `status(timeout=10.0)` | Stato in tempo reale dell’array in esecuzione: `target_fps` / `target_fps_auto` (la frequenza di trigger e se è gestita dal controller di frequenza), `target_fps_clamped_reason` (motivo per cui una frequenza fissata viene mantenuta più bassa, altrimenti `None`), `actual_fps` (misurato; 0 una volta che il ciclo di acquisizione ha rinunciato o si è bloccato), `recommended_fps`, `streaming` (False una volta che il ciclo di acquisizione si è interrotto — controllare questo valore, non solo `last_error`; rimane vero anche dopo un riavvio delle impostazioni, come segnalato da `restarting`), `stalled` (vero mentre il ciclo di acquisizione è in esecuzione ma non ha pubblicato alcun gruppo di fotogrammi per circa dieci periodi di fotogrammi, 2–30 s; `last_error` lo segnala solo dopo 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` solo quando l’array era connesso con PTP **e** ogni membro ha letto per l’ultima volta `PtpEnable` su; non è la stessa cosa di “bloccato”, che `ptp_epoch` misura), `ptp_configured` (ciò che la connessione ha richiesto), `ptp_off_members` / `ptp_unread_members` (membri che hanno letto per l’ultima volta `PtpEnable` disattivato / non hanno risposto), `member_recovery` (`None`, ovvero un membro che ha perso alimentazione o il collegamento: `state` `waiting` pur non rispondendo, `reopening`, poi `recovered`, con `members` e `why` — l’array lo riapre quando risponde nuovamente e gli restituisce il PTP del momento della connessione, il cablaggio di trigger e linea di sincronizzazione, il pacing e le impostazioni della telecamera, quindi riprende lo streaming senza nuovi `connect_array`) e `ptp_epoch` (misurato), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` significa che Windows non è riuscito a leggere la velocità dell&#x27;adattatore — vedi [Frequenze di fotogrammi previste](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (se la catena di radianza per banda è utilizzabile e quale elemento l’ha interrotta) e `band_clamp` — mentre una visualizzazione in tempo reale riproduce l’array: la percentuale di pixel illuminati che l’unmix ha limitato a 0, per elemento (`members`: `{serial: [band0, band1, band2]}` nell’ordine delle bande R, G, B della fotocamera) e per ciascuna banda il composito mostra (`composite`: `{band name: share}`), con `warn`, la percentuale che l’interfaccia grafica contrassegna come «Dati di banda a bassa affidabilità» (misurati circa una volta al secondo; `None` quando non è in esecuzione alcuna visualizzazione in tempo reale), e `member_ae` — per ciascun membro, l’esposizione (`exposure_us`) e il guadagno (`gain_db`) contenuti nell’ultimo fotogramma pubblicato, ricavati dal fotogramma stesso (`read_nodes` di `ExposureTime` / `Gain` restituisce l’ultimo valore scritto mentre è in corso l’esposizione automatica, non il valore attuale). |
| `set_member_settings(serial, **kwargs)` | Scrive le impostazioni di una telecamera membro, così come `CameraSession.set_settings()` le rileva (esposizione, guadagno, luminosità target, ...); la telecamera rimane nell’array. Genera un errore `ChlorosConnectError` per una telecamera seriale che non fa parte dell’array. Vedi [Esposizione sui membri dell’array](#exposure-on-array-members). |
| `capability(timeout=10.0)` | I valori stabiliti dallo smart-prep (il livello di acquisizione, la dimensione del fotogramma, `pixel_format`), il blocco `health` in tempo reale e il budget di banda dell’host. Vedi [Stato di salute dell’array](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Modifica la frequenza di trigger mentre l’array è in funzione, senza ricollegarsi. Un numero la blocca (il controller di frequenza non la modifica); `None` lo riporta in modalità automatica. Restituisce `{requested, target_fps, target_fps_auto, clamped}` — `clamped` è True quando la frequenza è stata ridotta al valore che l’host e la rete sono in grado di supportare. Vedi [Acquisizione a intervalli per rilievi con droni](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Un fotogramma da ogni telecamera, **tutti da un unico trigger**, come array numpy (nessun file scritto). Restituisce un `GrabResult` (un `dict`): `{serial: array}`, oppure `{serial: (array, metadata)}` con `with_metadata=True`. `.group_id` è il trigger condiviso dai fotogrammi; `.errors` associa ogni telecamera esclusa al motivo (una telecamera che non può essere gestita a quel trigger viene omessa, mai sostituita da un’altra). `processing` è `raw` / `debayered` / `radiance` / `reflectance` / `display`. La stessa lettura dell’API del progetto [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — un’unica implementazione condivisa — quindi ogni parola chiave presente lì (`band`, `align`, `bit_depth`, `refl_max`) funziona qui. Richiede `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iteratore che restituisce un risultato `grab()` per tick (solo argomenti chiave), con cadenza `fps`, limitato da `duration` (secondi) e/o `count` (`None` = illimitato). **Ogni insieme restituito costituisce un nuovo trigger:** un tick il cui trigger più recente è l’insieme che ha già generato attende il trigger successivo invece di ripeterlo, quindi lo stream scorre alla velocità minore tra `fps` e la frequenza dei fotogrammi dell’array stesso. Genera un errore `FrameFetchError` anziché rimanere in silenzio: immediatamente quando tutte le telecamere rifiutano per un motivo per cui il tick successivo si ripeterebbe, dopo 60 s di tick senza fotogrammi e dopo 60 s senza nuovi trigger (“nessun nuovo trigger per 60 s”). Stessa implementazione di `ArrayHandle.frame_stream()`. Vedi [Fotogrammi in tempo reale nella propria pipeline](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Avvia l’acquisizione continua propria del backend (la modalità Continua della GUI) → [`ContinuousHandle`](#continuoushandle). Salva **ogni** gruppo sincronizzato pubblicato dall’array, così come lo salverebbe `capture()` (stessi controlli di esportazione), fino a quando non viene interrotto o fino a quando `max_captures` / `duration_s` (`None` = fino all’arresto). Genera un errore `ChlorosConnectError` se ne è già in esecuzione uno per questo array. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Un gruppo di acquisizione sincronizzato. Restituisce un `CaptureResult` (elenco di dizionari di frame + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Passare un risultato precedente`group_seq` come `after_seq` per ottenere il gruppo sincronizzato successivo a esso, anziché attendere uno nuovo. Controlli di esportazione di seguito. |
| `capture(..., smart=True)` | **Acquisizione intelligente** — attende che l’AE si stabilizzi su tutte le telecamere, quindi si attiva. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Acquisizione più veloce: solo dati grezzi + la lettura DAQ assegnata (+ l’indice combinato libero). Rispecchia il pulsante “Acquisizione più veloce” dell’interfaccia grafica. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Singola / Continua / A intervalli in un unico ciclo limitato. Restituisce `list[CaptureResult]`.**Richiede `count` e/o `duration_s`** per terminare (l’SDK non supporta Ctrl+C). Termina inoltre anticipatamente, dopo il passaggio, quando un passaggio segnala un’unità piena (una voce `.failed` con `disk_full`); quel passaggio è l’ultimo risultato. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Avvia la registrazione della vista combinata in tempo reale dell’indice in formato video/GIF → `RecorderHandle`. Il video è un Motion JPEG `.avi` (non è necessario ffmpeg). Se non è aperta alcuna vista GUI, il backend lo renderizza: `expr` (un’espressione di indice nei nomi delle bande dell’array, ad es. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`), oppure l’NDVI dell’array per impostazione predefinita. Un registratore composito per ogni array. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Avvia una raffica raw-Bayer ad alto fps → `RecorderHandle`. Rielabora offline con `build_video()`. `orient=True`: i prodotti generati da `build_video()` vengono ruotati in base all’orientamento della telecamera (i prodotti per singola telecamera in base alla propria telecamera, l’indice combinato in base a quella principale); i fotogrammi raw rimangono nativi e il manifesto della raffica registra l’orientamento di ogni elemento. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Elabora offline una sequenza raw salvata trasformandola in uno o più video calibrati. Blocca l’esecuzione fino al completamento (`wait=True`) e restituisce il risultato della compilazione: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — vedi [il risultato della compilazione](#the-build_video-result) di seguito. |
| `build_video_status(job_id, timeout=15.0)` | Interroga un processo di compilazione offline: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Rilascia l’intero array — **se questa sessione lo ha aperto** (`connect_array`): diventa quindi `force_disconnect()` e genera un&#x27;eccezione `ChlorosConnectError` quando il rilascio non viene confermato (l&#x27;uscita dal blocco `with` non genera mai un&#x27;eccezione). Su un handle proveniente da `attach_array` lascia l&#x27;array in esecuzione e stampa una riga su stderr che lo indica: l&#x27;array potrebbe appartenere alla GUI o a un altro script. Ogni telecamera viene lasciata in funzionamento libero e per il resto così come è stata trovata al momento della connessione; `keep_settings=True` salva le sue impostazioni nella memoria della telecamera stessa, mentre `clear_settings=True` ripristina tale memoria alle impostazioni di fabbrica (vedi [Memoria della telecamera](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Rilascia l’intero array indipendentemente da chi lo abbia aperto. Usalo su un handle collegato quando intendi terminare l’array (ad es. per ricollegarlo con impostazioni diverse). Restituisce il risultato una volta che il backend conferma il rilascio, oppure quando non c’è nulla da rilasciare (l’array è già stato rimosso o nessun backend è in esecuzione); uno smantellamento che ha superato il timeout o che è stato rifiutato dal backend genera l’errore `ChlorosConnectError`. Con entrambi i flag restituisce ciò che la memoria di ciascuna telecamera ha ottenuto (`camera_memory`), altrimenti `None`. |

`capture()` controlli di esportazione (stesso endpoint utilizzato dalla GUI e dalla CLI):

- `processing` / `levels` — `processing="all"` (o `levels=["raw","radiance",…]`) salva ogni tipo di esportazione applicabile per ciascuna telecamera; un singolo valore `processing` salva solo quel livello.
- `aligned` — **attivo per impostazione predefinita (`None`), poiché l’acquisizione dello stesso array da parte dell’app Chloros è**(la sua impostazione di acquisizione**Allineata** per array). L’esportazione non raw di ogni membro viene allineata al fotogramma della telecamera principale tramite il [profilo di allineamento](#array-alignment) e ritagliata alla sovrapposizione comune dei membri, in modo che i file siano allineati a livello di pixel e presentino le stesse dimensioni, lo stesso ritaglio e gli stessi tag `Alignment*` dell’app (all’interno di una ROI la trasformazione viene ricalibrata sulla finestra); `XMP-mapir:AlignmentJSON` indica `"applied": true` con il suo `crop_roi`. Il file Raw rimane non deformato alle dimensioni native ma porta la trasformazione nei metadati. `aligned=False` salva ogni file alle dimensioni native, non deformato; `aligned=True` è l’impostazione predefinita esplicita. `continuous()`, `capture(smart=True)`, `capture_repeated()` e le opzioni a livello di progetto `ArrayHandle.capture()` / `capture_smart()` adottano lo stesso valore predefinito. Indipendentemente dal fatto che la deformazione venga applicata o meno, ogni file membro registra la trasformazione `Alignment*` della propria telecamera una volta che l’array dispone di un profilo di allineamento — ad eccezione di una telecamera che il profilo non allinea (tralasciata o forzata all’identità a causa di una risoluzione fallita) e la telecamera di riferimento quando nessun’altra telecamera è allineata: queste non riportano tag `Alignment*`. In assenza di profilo sull’array, l’acquisizione viene salvata non allineata e il risultato lo indica: `result.alignment` è uguale a `{"requested": True, "applied": False, "warnings": [...]}`, con un avviso per ogni motivo per cui non è stato possibile applicare la deformazione (assenza di profilo, membri con risoluzioni diverse, un array non sincronizzabile, un fotogramma su cui il profilo non può essere mappato). `applied` è `True` solo quando ogni esportazione che ha richiesto la correzione l’ha ottenuta. L’allineamento automatico dell’array crea il profilo una volta che l’esposizione si è stabilizzata; `ArrayHandle.calibrate_alignment()` / `align-calibrate` della CLI ne creano uno su richiesta. `result.oriented` indica se `orient=True` ha elaborato i prodotti.
- `render_index=False` — salta la sovrapposizione dell’indice di vegetazione per singola telecamera; per impostazione predefinita, la rende dove configurata.
- `daq` / `daq_tolerance_ms` — la lettura DAQ viene salvata come sidecar `.daq` per impostazione predefinita, anche quando nessun livello selezionato ne ha bisogno; `daq=False` la salta in tal caso, mentre `daq_tolerance_ms` imposta la tolleranza di corrispondenza (predefinita 500). `force_daq=True` è la vecchia grafia del valore predefinito. Vedi [La lettura DAQ salvata da una cattura](#il-DAQ-che-legge-una-cattura-salva--file-DAQ-e-corrispondenze-DAQ).
- `orient=True` — attiva i **prodotti**di ciascun membro (`display`, `radiance`, `reflectance`, l’indice per telecamera e il composito) in base all’orientamento della telecamera, dopo la deformazione di allineamento: in base all’orientamento della**camera principale**al momento della traslazione nel suo sistema di riferimento (impostazione predefinita `aligned` — i membri rimangono co-registrati), altrimenti in base a quello della propria telecamera.**Impostazione predefinita: ogni file membro in formato nativo**, come avviene sempre per le acquisizioni in array; `raw` e `debayered` rimangono in formato nativo in entrambi i casi. Ogni file registra l’orientamento del fotogramma in cui si trovano i propri pixel (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — un file raw, quello della propria fotocamera), e l’XMP `Alignment*` mantiene la trasformazione del fotogramma nativo, applicata prima dell’orientamento; un file raw che contiene tale trasformazione registra anche `ChlorosAligned*` (quello del master), ovvero l’orientamento di un prodotto trasformato nel sistema di riferimento del master. L’elaborazione ruota ciascun prodotto in base a quello che appartiene ai suoi pixel: quello allineato quando trasforma il file raw (allineamento array attivato), quello della fotocamera stessa quando non lo fa. `capture_repeated()` e `capture_fastest()` lo lasciano passare; anche `ArrayHandle.capture()` / `capture_smart()` a livello di progetto lo accettano.

**Compressione TIFF (opzione HTTP):** `ArraySession.capture()` non invia alcuna chiave `compression`, quindi si applica l’impostazione predefinita del backend: `POST /api/camera/array/capture` legge un parametro del corpo della richiesta `compression`, `"none"` per impostazione predefinita: non compresso, i pixel del sensore esattamente come vengono letti (~6,3 MB per fotogramma a piena risoluzione). `"deflate"` (zlib L1 senza perdita di dati + predittore orizzontale, ~4,1 MB/fotogramma) scrive file più piccoli più lentamente — entrambi sono senza perdita di dati e vengono letti in modo identico durante l’importazione. L&#x27;SDK non espone alcun parametro per questa opzione; la via d&#x27;uscita è `chloros-cli lattice array-capture --compression deflate` o HTTP grezzo. DEFLATE blocca il GIL di Python, quindi le scritture compresse non vengono parallelizzate tra i thread di scrittura per ciascuna telecamera — l&#x27;acquisizione continua a piena risoluzione alla frequenza del sensore richiede l&#x27;impostazione predefinita non compressa. Dettagli: [Riferimento CLI → array-capture](cli-reference.md).**Gruppi consecutivi (protocollo HTTP):** l’endpoint restituisce `group_seq`; un ciclo che lo rinvia come `after_seq` nella richiesta successiva ottiene il gruppo sincronizzato successivo — compreso quello pubblicato mentre veniva salvata l’acquisizione precedente — invece di attendere un gruppo pubblicato dopo la richiesta, mentre `skipped_groups` segnala eventuali gruppi che il ciclo non ha rilevato. `capture_repeated`, sia `--continuous` della CLI che la modalità Continua della GUI funzionano in questo modo; i passaggi dell’intervallo rimangono aggiornati in ogni momento programmato.**Sovrascritture di esportazione per singolo membro (solo HTTP):**lo stesso endpoint accetta anche `exclude_serials` (elenco — esclude membri dall’insieme salvato; l’array continua a funzionare come un unico gruppo sincronizzato e i membri esclusi vengono restituiti in `excluded`), `serial_levels` (sovrascritture a livello di singola telecamera `{serial: [level tokens]}`) e `serial_index` (sovrascritture dell’overlay dell’indice per singola telecamera `{serial: bool}`). Si tratta di parametri del corpo in parità con la GUI e**non sono ancora argomenti chiave dell’SDK**; i membri assenti dalle mappe ricorrono ai valori a livello di array `levels` / `render_index`.

##### Ispezione delle cam saltate — `CaptureResult.skipped`

`ArraySession.capture()` restituisce un `CaptureResult`, che è una sottoclasse di `list`: iterarlo, indicizzarlo, `len()` — ogni modello esistente continua a funzionare. Il nuovo codice può ispezionare l’attributo `.skipped` per vedere quali cam sono state escluse e perché. Il caso più comune è quello delle cam RGB in un array a filtri misti quando si richiede `processing="radiance"` o `"reflectance"` — la radianza per pixel Bayer non ha senso per un sensore a banda larga, quindi il backend salta quelle telecamere piuttosto che produrre dati privi di significato.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

I token di motivazione seguono lo schema `<level>-not-applicable-to-rgb-cam` (una voce per ogni livello saltato, ciascuna contenente `level`). Una telecamera il cui pacchetto di calibrazione non è caricato salta `radiance` / `reflectance` con `<level>-needs-calibration (connect the camera so its cal pack loads)`, e in assenza di una lettura DAQ utilizzabile, salta i valori di riflettanza con `reflectance-skipped-no-dls`. Gli altri salti specifici per la riflettanza sono `reflectance-skipped-no-fresh-dls` (nessuna nuova lettura della radiazione discendente disponibile), `reflectance-skipped-bound-daq-unavailable (…)` (impossibile raggiungere il DAQ associato) e `dls-uncalibrated-band-<nm>` — la banda si trova prevalentemente al di fuori dell’intervallo radiometricamente calibrato del sensore di luce del DAQ (~374–974 nm), pertanto la divisione assoluta della riflettanza basata sul DAQ viene rifiutata e il fotogramma ricade nettamente sulla risposta del sensore. Tra gli SKU in commercio, solo l’F988 lo attiva; il percorso supportato da quella fotocamera è il flusso di lavoro con pannello di riflettanza.

##### File che non è stato possibile scrivere — `CaptureResult.failed`

Un’acquisizione che è riuscita a scrivere solo alcuni dei propri file restituisce comunque: l’elenco contiene tutti i file che **sono stati** scritti, mentre `.failed` elenca il resto, un dizionario per ogni file con `serial` (`None` per il composito combinato e il sidecar di un array `.daq`), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, oppure `match` — la provenienza di `.daq` e `.match.json`, quando `.daq` è stato scritto ma il suo sidecar no) e `error`. Su un’unità piena, la voce contiene anche `disk_full: True`; ogni successiva acquisizione su quell’unità fallirebbe allo stesso modo, motivo per cui `capture_repeated()` si ferma lì. Non viene generata alcuna eccezione per una cattura parziale, e un file in `.failed` non è stato lasciato sul disco scritto a metà. `.failed` è vuoto in caso di cattura pulita.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### Il DAQ che legge una acquisizione salva — `.daq_files` e `.daq_matches`

Ogni volta che un DAQ è associato alla telecamera o all’array, oppure è in pool, ogni acquisizione salva la lettura downwelling utilizzata: un file `.daq` nella sottocartella `daq/` della cartella dell’acquisizione, insieme ai file `.match.json` e `.cameras.json` (di cui proviene la lettura). Ciò vale anche per le acquisizioni in formato raw o debayered, e il file è lo stesso indipendentemente dal fatto che l’acquisizione provenga dall’interfaccia grafica (GUI), dalla riga di comando (CLI) o dall’SDK. `daq=False` lo omette per un’acquisizione i cui livelli non utilizzano il DAQ (un livello `radiance`, `reflectance` o `display` lo scrive comunque); `daq_tolerance_ms` sposta la linea di tolleranza per quella acquisizione e mai la lettura utilizzata. Quale DAQ: quello associato alla telecamera o all’array (la selezione del sensore di luce nella GUI, un’associazione del progetto o `chloros-cli daq pool-bind`), altrimenti il più recente disponibile nel backend. La CLI specifica i controlli `--no-daq` e `--daq-tolerance-ms` ([Riferimento CLI](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Un&#x27;acquisizione salva la lettura del sensore di luce utilizzata come `.daq` con un `<file>.daq.match.json` accanto, che indica come la lettura è stata collegata all&#x27;esposizione: `method`, `gap_ms` (dall’esposizione alla lettura DAQ più vicina) e `in_tolerance` (entro 500 ms, ovvero l’`daq_tolerance_ms` dell’acquisizione). L’XMP del file di riflettanza riporta lo stesso risultato di `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` e `Chloros:DlsInTolerance`.

Una cattura attende la prima lettura del DAQ dopo l’esposizione, in modo da poter fondere le letture su entrambi i lati di essa (`method` `interpolated`). Un DAQ-E invia una lettura ogni 0,4 s alla luce del giorno e ogni 1-2 s in ambienti interni, quindi `capture()` viene restituito fino a circa un intervallo e mezzo dopo; l’esposizione stessa non subisce ritardi. Non si attende un DAQ che ha smesso di inviare dati. Una lettura DAQ-E rappresenta la media della luce nell’intervallo trascorso dalla lettura precedente, quindi viene collocata al centro di tale intervallo: `gap_ms` viene misurato in corrispondenza di quel punto centrale, mentre `.match.json`&#x27;fornisce i timestamp propri delle letture. `ArraySession.continuous()` attende senza rallentare l’acquisizione. I passaggi di ritorno di `capture_repeated()`non attendono, quindi mantengono la loro frequenza e vengono abbinate alla lettura più vicina prima dell’esposizione.

Una lettura non scade mai, quindi nessun fotogramma viene perso a causa di una lettura in ritardo — viene invece etichettata:

- **Flusso del DAQ bloccato** — viene utilizzata la sua lettura più recente e `gap_ms` indica da quanto tempo risale (`in_tolerance: false` oltre i 500 ms).
- **Nessun DAQ da leggere**— disconnesso (`disconnect()`, `chloros-cli daq pool-disconnect`), oppure il DAQ associato non è raggiungibile — l’ultima lettura della fotocamera viene riutilizzata e associata a**questa** esposizione: `method`, `retained`, `gap_ms` da questa esposizione, `in_tolerance` ricalcolato, e `retained_from_epoch_ns` che indica l’esposizione a cui è stato inizialmente abbinato. Una telecamera che non ha mai effettuato una lettura continua a non avere alcuna riflettanza, e il risultato ne spiega il motivo (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, ovvero il rifiuto dell’acquisizione).

I file salvati da un’acquisizione all’interno della cartella del progetto aperto in Chloros si aggiungono all’elenco dei file di quel progetto, proprio come avviene per un’acquisizione effettuata nell’app (i file salvati altrove rimangono invariati): `CameraSession.capture()`, `ArraySession.capture()` (anche `smart=True`) e `capture_cameras_transient()`. Il risultato è riportato in `.project_add` (`capture_cameras_transient()`: `project_add` del dizionario) -- `held`, `files` e `message` del backend; `held` è `True` mentre è in corso un&#x27;elaborazione, e i file si uniscono una volta terminata. `None` quando nessun file è stato inserito nel progetto.

`CameraSession.capture()` e `ArraySession.capture()` restituiscono come risultato `.daq_files` e `.daq_matches`, mentre le risposte di acquisizione HTTP (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) contengono `daq_files` e `daq_matches`, uno per ogni `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` e `stream_fault` — errore di flusso proprio del DAQ, verificato solo in caso di lettura fuori tolleranza. Prima di fare affidamento sulla riflettanza ottenuta da un’acquisizione, controllare `in_tolerance` nel relativo `.match.json` o XMP, oppure i livelli [`status()`](#daqsensorsession-methods).

Livelli `processing`:

| Livello | Uscita |
| --- | --- |
| `"raw"` | Bayer monocanale (telecamere monocromatiche: la singola banda) direttamente dal sensore. |
| `"debayered"` *(impostazione predefinita SDK)* | Colore a 3 canali: offset scuro (DSNU) del pacchetto di calibrazione sottratto dal raw, seguito dal demosaic sensibile ai bordi di OpenCV — senza LUT di linearità, flat field o separazione dei colori (telecamere monocromatiche: scala di grigi a 1 canale). Il file TIFF salvato è nell’ordine dei canali **RGB**(RGB fotometrico, quindi qualsiasi visualizzatore di immagini mostra i colori reali); gli array in memoria restituiti da `grab()` / `frame_stream()` sono nell’ordine**BGR** di OpenCV — invertire i canali (`frame[..., ::-1]`) per confrontare i dati con un file salvato. `chloros-cli process` scrive lo stesso risultato di `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm tramite la catena radiometrica completa. Solo multispettrale — le telecamere RGB vengono ignorate. |
| `"reflectance"` | uint16 0..32768 (compatibile con Pix4D); richiede un abbinamento DAQ in tempo reale per il riferimento assoluto. Solo multispettrale. |
| `"display"` | Catena completa corrispondente all’anteprima dell’interfaccia grafica (CCM + WB + gamma secondo il profilo della fotocamera). |
| `"all"` | **Un file per ogni livello applicabile** per ciascuna fotocamera (in linea con l’impostazione predefinita “Capture All” dell’interfaccia grafica / CLI). Il file `CaptureResult` restituito contiene quindi un dizionario di fotogrammi per ogni `(cam, level)`, con il livello specificato in ciascun dizionario; i livelli non applicabili compaiono in `.skipped`. La lettura DAQ utilizzata per l’acquisizione viene salvata come sidecar `.daq`. |

> **Nota — l’impostazione predefinita differisce da quella della CLI.** `ArraySession.capture()` assume per impostazione predefinita il valore di `processing="debayered"`; il comando `chloros-cli lattice array-capture` assume per impostazione predefinita il valore di `processing="all"`. Passare esplicitamente `processing="all"` dall’SDK per rispecchiare il salvataggio multilivello della CLI/GUI.

##### Nomi dei file di acquisizione dell&#x27;array

Ogni livello viene salvato in una propria sottocartella di `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` per `display`, `index/`, `composite/`). Ogni file indica la propria telecamera come `SN<serial>-<filter>`: il numero di serie, seguito dalla parte relativa al filtro del modello della fotocamera — `SN214000533-FOCN` per una `M3C-L87-FOCN`, `SN261902142-F450` per una `M3M-L87-F450` — unito a `-`, quindi il nome presenta gli stessi campi separati da `_` di prima. Una telecamera il cui modello non può essere letto è indicata semplicemente come `SN<serial>`, mentre le acquisizioni delle versioni precedenti non presentano alcun filtro nel nome. Il nome del file dipende dal numero di livelli richiesti al membro:

| Livelli richiesti | Nome del file | Esempio (`reflectance`, predefinito `format="tiff"`) |
| --- | --- | --- |
| **Un livello** (`processing="reflectance"`, oppure un `levels` con un solo elemento) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — nessun suffisso di livello, come nelle versioni precedenti, in modo che gli script esistenti continuino a funzionare | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Più livelli** (`processing="all"`, oppure `levels` con più di un elemento) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

La [maschera di limitazione](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) segue il nome del prodotto, `clamp_masks/<product folder>/<product name>_clampmask.tif`: l’acquisizione della riflettanza aacquisizione della riflettanza a livello singolo scrive `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, mentre `processing="all"` scrive `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. La regola del suffisso viene determinata per ciascun membro in base ai livelli richiesti, quindi un livello saltato (ad es. la riflettanza su una telecamera RGB) non la modifica. Leggere i percorsi dai fotogrammi restituiti anziché costruirli. Un&#x27;acquisizione con una singola telecamera (`CameraSession.capture()`) aggiunge sempre il livello: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### Per i formati PNG e JPEG, vengono acquisiti solo i prodotti visualizzati e quelli dell&#x27;indice

Un file PNG o JPEG contiene 8 bit per canale. Se si richiede un livello che il formato non è in grado di supportare, la richiesta viene rifiutata prima ancora che venga acquisito alcun dato, con un messaggio che indica il livello e il formato TIFF come quello in grado di supportarlo:

- **`radiance`**(float32) e**`reflectance`**(uint16, `32768` = 1,0) — e quindi `processing="all"`, che li include. L’SDK verifica questo sugli argomenti: `capture()` genera un**`ValueError`** senza una richiesta (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` provenienti da una telecamera che trasmette più di 8 bit** (M3C a 12 bit, Mono12 M3M) — il backend li rifiuta, conoscendo la profondità di bit di ciascuna telecamera: 400 HTTP con `error_code` `FORMAT_CANNOT_HOLD_LEVEL`, generato come `ChlorosConnectError` (`CameraSession` / `ArraySession`) oppure `RuntimeError` (`CameraHandle` / `ArrayHandle`). L’acquisizione a livello singolo `debayered` di una telecamera acquisisce un fotogramma a 8 bit per PNG/JPEG e non viene rifiutata.

Salvare i dati radiometrici come `format="tiff"` (`ext=".tiff"`); utilizzare PNG/JPEG per `display` e i rendering indicizzati.

#### Come funziona l’esposizione automatica

- **Prima l’esposizione, poi il guadagno.** Con il guadagno su Auto, il guadagno rimane a 0 dB finché l’esposizione non ha raggiunto il valore massimo consentito.
- **Un’esposizione manuale lascia il guadagno in modalità automatica** finché non si modifica il guadagno o lo si imposta su manuale. Una volta tornati in modalità automatica, il limite del guadagno è quello precedente.
- **Le piccole luci non scuriscono la scena.** Viene ignorata una luce speculare fino all’1% dell’inquadratura e, nei pannelli di un bersaglio di calibrazione noto, fino allo 0,5% di un pannello.
- **Il limite massimo di esposizione del DAQ si applica solo mentre un bersaglio di calibrazione è visibile.** Impedisce il clipping di un pannello bianco; se non è visibile alcun pannello, la telecamera espone in base alla scena.
- **La luce costante rimane stabile, mentre quella variabile si stabilizza in pochi secondi.** In condizioni di luce costante, l’esposizione e il relativo limite rimangono invariati; dopo un’ampia variazione di luce, l’immagine si stabilizza nel giro di pochi secondi. Un clipping non limita più l’esposizione in modo permanente e, in caso di variazione dell’area di interesse (ROI), l’esposizione viene ripristinata.
- **Il target di luminosità rimane raggiungibile.** Non aumenta mentre la telecamera è al limite di esposizione e il suo guadagno non può salire, quindi una luce che ritorna non provoca un flare nell’inquadratura.
- **Una telecamera la cui esposizione automatica smette di rispondere viene mantenuta al livello corrispondente alla luce in cui si trova.** In pieno sole una telecamera potrebbe scendere alla sua esposizione minima e rimanere lì con un fotogramma scuro, pur con tutte le impostazioni che risultano normali. Chloros monitora il fotogramma, mantiene quella telecamera al livello che aveva impostato per quella luce e la lascia andare quando la luce cambia. Un sensore di luce oscurato o coperto non interferisce con questo processo.

#### Esposizione sui membri dell’array

Una telecamera LATTICE limita `ExposureTime` a un periodo della propria frequenza di fotogrammi. Mentre un array trasmette in streaming, Chloros esegue ogni membro con `AcquisitionFrameRateEnable` attivato, a una frequenza appena superiore a quella di trigger, in modo che il limite massimo di esposizione di un elemento si collochi leggermente al di sotto del periodo di trigger, **in qualsiasi ROI**:

- Il periodo di fotogramma di un elemento è pari al periodo di trigger meno un margine di 15% del periodo (almeno 3 ms, al massimo metà del periodo), e il suo limite massimo di esposizione è pari a quel periodo di fotogramma meno circa 0,1 ms. Esempio: un trigger a 10 fps (100 ms) fornisce una frequenza del membro di circa 11,8 fps e un limite massimo di esposizione di circa 85 ms.
- **Un `ExposureTime` manuale più lungo di tale limite massimo rallenta il trigger per adattarsi ad esso.** Chloros apre innanzitutto il limite massimo di quel membro per il valore e lascia che la frequenza più lenta funzioni per alcuni periodi di trigger (la scrittura può richiedere qualche secondo in più), quindi lo scrive, e l’attivazione non avviene più velocemente di quanto consentito dall’esposizione. Su un array a frequenza automatica, la frequenza precedente viene ripristinata non appena ogni membro la cui esposizione manuale l’aveva rallentata torna all’esposizione automatica. Un valore che la fotocamera stessa non può acquisire viene comunque rifiutato, con l’intervallo che ora accetta indicato nell’errore, e l’esposizione automatica del membro viene ripristinata se era stata disattivata solo per quella scrittura. Prima della versione 1.3.0, una piccola ROI limitava ogni membro al periodo di funzionamento libero del sensore, indipendentemente dalla frequenza di trigger: 4,9 ms a 768 × 304, quindi nessun elemento poteva essere schiarito.
- *Chloros* non accorcia mai un’esposizione manuale per adattarla a un trigger più veloce. Una frequenza automatica rimane entro i limiti consentiti dalle esposizioni manuali; una frequenza fissata più veloce di quanto consentito da un’esposizione lascia a quel membro un limite massimo sufficientemente lungo per essa, e accetta trigger non più veloci di quanto consentito dalla sua esposizione. Abbassare l’esposizione, o la frequenza fissata, quando ogni membro deve rispondere a ogni trigger.
- **Lettura:** la lettura dei nodi di un elemento mostra `AcquisitionFrameRateEnable` True e `AcquisitionFrameRate` alla frequenza dell’elemento; l’abilitazione si disattiva quando l’array si ferma.
- **L&#x27;array possiede quei due nodi. Non scrivere `AcquisitionFrameRateEnable` o `AcquisitionFrameRate` su un membro dell&#x27;array** — imposta invece lacon `target_fps` / `set_target_fps()`.

Scrivere l’esposizione di un membro con `arr.set_member_settings(serial, exposure_time=20000)` (le parole chiave di `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` ha la stessa funzione). Il membro rimane nell’array. Non aprire un membro dell’array con `connect_camera()`: connect rifiuta una telecamera già presente in un array, e il codice `CameraSession` che verrebbe restituito rimuove la telecamera dal proprio array al momento della disconnessione. `ArraySession.status()["member_ae"]` riporta l’esposizione di ogni membrodall’ultimo fotogramma.

### Un unico record di progetto, per ogni modalità

Una cartella di progetto contiene le impostazioni di ogni dispositivo: `cameras.json` — quelle di ciascuna telecamera (`cameras[].settings`, per una singola telecamera e per ciascun membro dell’array) e quelle a livello di array (`arrays[].capture`: formato, ROI, binning, frequenza di trigger, wire budget) — e `sensors.json`, relative a ciascun sensore di luce (tempo di integrazione, media dei fotogrammi, limite massimo). L’app, la CLI, l’SDK e il server MCP leggono e scrivono tutti gli stessi file, allo stesso modo:

- **Un dispositivo collegato a un progetto viene salvato al suo interno e registra ogni modifica in esso.** «All’interno di un progetto» significa con un progetto aperto nel backend (quello dell’app) oppure — tramite un `ChlorosProject` (`open_project()`), che indica la propria cartella al backend (`project_path`). Una telecamera, un array o un sensore di luce che il progetto non contiene ancora viene aggiunto ai suoi file al momento della connessione (`connect_camera()`, `connect_array()`, la connessione del sensore di luce, l’CLI, l’app, `chloros-mcp`), con le relative impostazioni. Una modifica da qualsiasi modalità (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, le impostazioni dell’array, `set_cap()`, l’CLI, i controlli dell’app) viene salvata nei file di quel progetto un attimo dopo. Un’esposizione, un guadagno o un valore target AE impostati su una telecamera vengono salvati come “mantenuti”, quindi riaprendo il progetto vengono mantenuti nuovamente.
- **L’apertura di un progetto ne imposta i dispositivi**, in ogni modalità: una fotocamera, un array o un sensore di luce collegati senza impostazioni proprie assumono quelle(`lattice cam-connect` / `array-connect` con un progetto aperto, `chloros-cli project connect`, `open_project().connect_all()`, riapertura dell’app). Un array collegato con un nuovo ID aggiorna l’array salvato dal progetto con le stesse fotocamere; un array con altre fotocamere viene aggiunto accanto ad esso.
- **Il salvataggio dell’app non sovrascrive mai le impostazioni in tempo reale di un dispositivo** con una loro copia precedente e mantiene le sostituzioni per singolo membro salvate da un chiamante CLI o SDK.
- **Il formato dei pixel impostato è quello che il progetto conserva.** Un formato impostato nelle Impostazioni dell’array, tramite `--pixel-format` o `connect_array(pixel_format=)`, 8 bit inclusi, viene salvato e ricollegato esattamente. Se non viene impostato alcun formato, ogni telecamera si connette con l’impostazione predefinita a 12 bit (`BayerRG12`, `Mono12` su un sensore monocromatico). Il nome del formato indica la profondità di bit; la famiglia è quella del sensore, quindi una telecamera a colori non viene mai impostata su un formato mono né una telecamera mono su uno Bayer, e la riga `[PixelFormat <serial>] ... written` lo indica quando un formato è stato mappato. Le esportazioni di un’acquisizione a 8 bit sono a 8 bit, riflettanza inclusa: ridimensionate e contrassegnate con il proprio `PixelScale`.
- Un dispositivo collegato senza alcun progetto non registra nulla.
- **Una registrazione segue un progetto creato successivamente; una registrazione aperta la lascia con il proprio progetto.**Quando viene creato un progetto — `create_project()`, `chloros-mcp`&#x27;s `create_project`, oppure il**Nuovo progetto**dell’app — o duplicato (la funzione**Duplica**dell’app, che apre la copia) mentre una registrazione sta ancora scrivendo nel progetto che era aperto in precedenza, la registrazione continua e ciò che viene scritto finisce nel nuovo progetto, nella stessa posizione al suo interno (sotto). Aprendo un progetto esistente — `ChlorosProject.process()` (che apre il proprio progetto in background), `chloros-cli process`, l’opzione**Apri progetto** dell’app — lascia una registrazione ancora in corso nel progetto in cui viene registrata: essa termina lì e i relativi file si aggiungono all’elenco dei file di quel progetto (l’app interrompe ogni registrazione prima di aprire un progetto).
  - `ArraySession.record()` e `ArraySession.burst()`: il video, o la cartella della raffica con tutti i fotogrammi, viene spostato lì per intero quando la registrazione si interrompe. Il risultato dell’interruzione riporta il nuovo percorso (`out_dir`, `video_path`), e da lì viene generata una versione della sequenza. Se l’unità rifiuta lo spostamento, la registrazione rimane dove è stata scritta e il risultato lo indica (`relocation_error`). I fotogrammi grezzi di una sequenza interrotta e `.daq` vengono aggiunti all’elenco dei file del progetto in cui finiscono; un video non viene aggiunto all’ elenco dei file di un progetto.
  - `ArraySession.continuous()`: ogni acquisizione successiva alla modifica viene salvata nel nuovo progetto; quelle salvate in precedenza rimangono dove sono.
  - `DAQSensorSession.record_start()` (e la registrazione del sensore di luce dell’app stessa): il file `.daq` viene spostato nella cartella `light_sensor` del nuovo progetto al termine della registrazione; `record_stop()` restituisce il nuovo percorso e il file viene aggiunto all’elenco dei file di quel progetto.

  Una registrazione in una cartella esterna al progetto aperto rimane dove si trova, mentre i lavori `start_camera_continuous()` / multi-stream mantengono la cartella loro assegnata.
- **Nell’app, le acquisizioni effettuate dopo la creazione di un nuovo progetto vengono inserite in quest’ultimo.** La creazione o la duplicazione di un progetto mantiene attive tutte le sessioni di acquisizione e le registrazioni in corso, e ogni acquisizione effettuata successivamente (Acquisisci tutto, Avvia acquisizione) viene salvata nel nuovo progetto e aggiunta al relativo elenco di file. L’apertura di un progetto esistente nell’app interrompe prima tutte le operazioni in corso.
- **Un progetto non viene spostato mentre è in fase di registrazione.**Lo spostamento del progetto aperto in un’altra directory di lavoro (in**Impostazioni → Directory di lavoro** dell’app; nessuna chiamata SDK sposta un progetto, mentre un client HTTP che invia dati a `/api/move-project` lo fa) viene rifiutata con codice HTTP 409 `PROJECT_RECORDING` mentre una registrazione array o in raffica, un’acquisizione continua array, un’attività continua/multistream della telecamera o una registrazione del sensore di luce sta scrivendo al suo interno: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Ciò è indipendente dalle impostazioni delle telecamere al momento della disconnessione (vedi [Memoria della telecamera](#camera-memory--keep_settings--clear_settings)): un progetto imposta autonomamente le proprie telecamere all’apertura.

### Memoria della telecamera — `keep_settings` / `clear_settings`

Quando un array è collegato, le sue telecamere assumono le impostazioni dell’array, che è poi possibile modificare. Quando Chloros disconnette le telecamere — `ArraySession.disconnect()` / `force_disconnect()`, al termine di una funzione transitoria o all’arresto di `start_camera_continuous()` — riporta ciascuna di esse allo stato in cui si trovava prima della connessione: il cablaggio del trigger e della linea di sincronizzazione, le impostazioni PTP e della telecamera — `ExposureAuto` / `GainAuto` (e un `ExposureTime` / `Gain`), `TargetBrightness`, nonché i limiti AE e lo smorzamento, il bilanciamento del bianco, il livello del nero, `PixelFormat`, binning e ROI, la pipeline del colore (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, `PtpSlaveOnly` / `PtpDomainNumber` di PTP, la dimensione del pacchetto (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), il ritmo (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), i dati del blocco (`ChunkModeActive`, ogni `ChunkEnable` sotto il proprio `ChunkSelector`), la rimozione dei disturbi dall’ingresso di trigger: un array fornisce la linea di sincronizzazione di uno slave (`LineFilterSelector` / `LineFilterWidth`) e la frequenza dei fotogrammi di acquisizione (`AcquisitionFrameRate` solo quando il suo abilitamento era attivo: se disattivato, la frequenza viene letta al massimo consentito dal ritmo di funzionamento, non come impostazione). Viene scritto solo ciò che differisce: l’arresto iniziale della telecamera, il ritmo di funzionamento e la frequenza dei fotogrammi prima di un’esposizione che richiede il proprio periodo di fotogramma.

Per l’utilizzo delle telecamere al di fuori di un progetto, due argomenti scrivono nella **memoria propria** della telecamera — ciò con cui si avvia, in qualsiasi software (su `disconnect()` / `force_disconnect()`, ogni funzione transitoria, `start_camera_continuous()` e `ChlorosProject.disconnect_all()`; MCP: gli stessi argomenti di `disconnect_device` e degli strumenti transitori; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** salva le impostazioni di ciascuna telecamera così come sono state lasciate nella memoria al termine della sessione (UserSet2, impostazione di avvio). Le telecamere rimangono comunque in funzionamento libero (`TriggerMode` `Off`), con il cablaggio della linea di sincronizzazione così come trovato. Una telecamera priva di UserSet2 salva le impostazioni in UserSet1; questo computer lo tiene presente, quindi l’autoguarigione dell’array di Chloros — che ripristina un UserSet1 che si è discostato dalla linea di base di Chloros — la lascia inalterata.
- **`clear_settings=True`** ripristina la memoria di ciascuna telecamera alle impostazioni di fabbrica: la linea di base di Chloros, quella su cui si basa l’autoguarigione dell’array — impostazioni predefinite di fabbrica con `TriggerMode`, `Off`, `TriggerSource`, `Line2`, esposizione automatica, bilanciamento del bianco calibrato di una telecamera RGB mantenuto — in UserSet1, definito come set di avvio.
- Non entrambi (`ValueError`; il backend risponde con 400). La risposta contiene `camera_memory`, per ciascuna telecamera `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` o `{"cleared": bool, "errors": [...]}` — `force_disconnect()` lo restituisce; per un lavoro viene fornito con `stop()`.

L’app non offre questi valori.

**Ogni via d’uscita ripristina le telecamere.**Il ripristino «così come trovato» viene eseguito su ogni percorso di uscita: `CameraSession.disconnect()` (la telecamera**X**dell’app), il**X** dell’app su un array o su qualsiasi elemento dell’array (l’intero array viene rilasciato), chiudendo l’app (il backend ha fino a 20 s per ripristinare ogni telecamera, array e registrazione continua in esecuzione; una funzione transitoria o una chiamata one-shot ancora in esecuzione si ferma al suo prossimo punto di sicurezza, ripristina prima le proprie telecamere, per un massimo di 8 s, e risponde con HTTP 503 `SHUTTING_DOWN`; un `connect_array()` ancora in esecuzione si interrompe prima della sua prossima telecamera e ripristina le telecamere che ha aperto, per un massimo di 10 s, e genera `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); e una che arriva durante lo spegnimento non apre nulla: HTTP 503 `SHUTTING_DOWN`), e un comando `chloros-cli` che viene terminato o il cui terminale si chiude (il suo processo viene interrotto e le telecamere vengono prima riportate allo stato iniziale). Un backend che va in crash o viene terminato forzatamente non può riportare nulla allo stato iniziale; un riavvio riporta ogni telecamera nella propria memoria.**Un progetto imposta autonomamente le proprie telecamere.** Quando un array si connette tramite un progetto — il progetto aperto dall’app o un `ChlorosProject`, che si identifica presso il backend (`project_path`) — ogni membro riceve le impostazioni che il progetto ha salvato per esso: un valore di esposizione, guadagno o obiettivo AE mantenuto su di esso (considerato come proprio, quindi l’esposizione automatica non lo modifica) e la sua modalità di bilanciamento del bianco (correzione di Chloros; il bilanciamento del bianco della fotocamera rimane disattivato), qualunque cosa sia memorizzata nella memoria della fotocamera.

### Modalità di acquisizione e registratori

La superficie dell’array rispecchia il pannello di acquisizione della GUI: modalità otturatore Singolo / Continuo / Intervallo / Più veloce, più due registratori (video composito in diretta e raffica raw → rielaborazione offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**è il ciclo Continuo/Intervallo dell’SDK. Poiché non esiste `Ctrl+C` per interromperlo da uno script, è**necessario** passare `count` e/o `duration_s` (si interrompe quando viene raggiunto uno dei due). `interval_s` viene misurato dall’inizio di ogni passaggio (in linea con l’interfaccia grafica). I passaggi continui (`interval_s=0`) concatenano `group_seq` → `after_seq`, quindi sono gruppi sincronizzati consecutivi alla velocità dell’array. I restanti kwarg vengono passati direttamente a `capture()`.
- **`continuous`**è la cattura continua propria del backend — la modalità Continua dell’interfaccia grafica.**Utilizzala al posto di `capture_repeated()` quando desideri che ogni trigger venga salvato su disco con un’elevata frequenza di fotogrammi:** `capture_repeated()` effettua una richiesta HTTP per gruppo dallo script, mentre `continuous()` mantiene due acquisizioni in corso all’interno del backend senza scambi tra i gruppi. Misurato con ROI 512 su un array di 9 telecamere: 73–77 acquisizioni/s, a fronte di un array che ne pubblica circa 78. Utilizzate `capture_repeated()` per l’acquisizione a intervalli (`interval_s`) e per i passaggi che si controllano uno per uno. Su Windows, escludere il backend da Microsoft Defender per ottenere velocità elevate (vedere [Passaggio 1](#step-1--install-the-chloros-platform-package)).
- **`record`** è di *livello di monitoraggio*: registra il composito dell’indice combinato in tempo reale così come viene visualizzato. Con la vista combinata della GUI aperta, registra quella vista così come viene visualizzata. Se non ce n’è nessuna aperta, il backend genera la vista autonomamente: `expr` se ne passi una, altrimenti l’NDVI dell’array (le bande rosse e NIR più vicine a 660 nm e 850 nm, come le rileva la vista in tempo reale), una volta allineato l’array. Denominare le bande in `expr` secondo la denominazione dell’array: `<band>_<filter>` per una telecamera M3C (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` per una M3M (`Mono_850`); un nome breve del colore (`NIR`, `Red`) funziona solo se presente in una banda dell’array. Una vista GUI aperta durante la registrazione viene registrata finché rimane aperta. Una registrazione che non scrive alcun fotogramma genera un errore con `error` e `"no-frames: …"` (l’array potrebbe essere ancora in fase di allineamento o di stabilizzazione dell’esposizione), e si genera l’errore `stop()` (vedere [`RecorderHandle`](#recorderhandle)). Un registratore composito per ogni array (genera un errore se ne è già in esecuzione uno).
- **`burst` → `build_video`** è di *livello analitico*: `burst` scrive i fotogrammi raw + un manifesto per ogni fotogramma + un `.daq` per ogni lettura DLS distinta sotto `<output>/bursts/<base>/` alla velocità massima del ciclo di acquisizione (senza catena, senza exiftool, senza anteprima). `build_video` allinea temporalmente ogni fotogramma al più vicino `.daq` e riesegue la pipeline di importazione per radianza/riflettanza/indice della pipeline di importazione. `products` è un elenco di `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (impostazione predefinita: l’indice combinato). `burst().stop()` avvia inoltre automaticamente la creazione di un indice combinato, restituita come `build_job` nel risultato di arresto.

##### Il risultato `build_video`

| Chiave | Significato |
| --- | --- |
| `success` | `True` solo quando ogni prodotto richiesto è stato generato come richiesto: almeno un output, nulla in `skipped`, e non in `degraded`. |
| `outputs` | Percorsi dei video (e delle GIF) scritti. |
| `errors` | Problemi, sotto forma di testo. |
| `skipped` | Un `{"serial", "level", "reason"}` per ogni prodotto richiesto che **non** ha prodotto alcun video — a causa di un problema della telecamera (mancanza del pacchetto di calibrazione per quella telecamera, assenza di lettura DLS per la riflettanza, errore nella catena di elaborazione) o dell’indice combinato, che contiene `"kind": "combined"`, `serial` e `None` (nessuna coppia di bande rosso/ NIR tra le telecamere, nessun gruppo con radianza per entrambe le bande). Qualsiasi voce genera `success` e `False`; gli altri video vengono comunque scritti ed elencati. |
| `substitutions` | Telecamere renderizzate a un livello diverso da quello richiesto, e perché — una telecamera RGB per cui è stata richiesta la radianza/riflettanza riceve il proprio video di visualizzazione, con bilanciamento del bianco se è disponibile il pacchetto di calibrazione della telecamera. Non si tratta di un errore. |
| `notes` | Testo informativo, non problemi — ad esempio, una telecamera senza espressione di indice specifica non ha un video di indice, oppure il video di visualizzazione di una telecamera RGB non è bilanciato in bianco (mancanza del pacchetto di calibrazione o presenza di uno senza bilanciamento del bianco di fabbrica), con indicazione del nome della telecamera. Non è un errore. |
| `degraded` | `True` quando è stato scritto un prodotto ma non è quello richiesto: l’indice combinato offline è la differenza normalizzata integrata tra rosso e NIR, quindi un’espressione di indice configurata non viene valutata. Genera `success` `False`. |
| `combined` | Riepilogo dell’indice combinato: le bande utilizzate, l’intervallo LUT, i fotogrammi generati. |
| `clamp_masks` | Percorsi delle [maschere di limitazione dell’unmix](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) deisingolo fotogramma (`save_tiffs=True`), in `clamp_masks/<level>/` accanto alla cartella `<level>/` dei TIFF, uno per ogni TIFF in cui è stato limitato un pixel. Un video non ne contiene alcuno. |
| `clamp_fractions` | Cosa indica un video al posto di una maschera: un `{"kind", "serial", "level", "frames"}` per ogni prodotto video (`kind` `"combined"` per le telecamere membri dell’indice combinato), `frames` un elenco di `[frame, R, G, B]` — la quota di pixel di quel fotogramma che l’unmix ha limitato nella banda contenuta dal suo canale R / G / B (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Restituito da `ArraySession.record()` e `ArraySession.burst()`. Utilizzarlo come gestore di contesto per l&#x27;arresto automatico all&#x27;uscita dallo scope, oppure gestirlo manualmente.

| Membro | Descrizione |
| --- | --- |
| `job_id` | ID del processo di backend (str). |
| `kind` | `"composite"` (da `record`) o `"raw"` (da `burst`). |
| `start_stats` | Il dizionario restituito dalla chiamata `start`. |
| `result` | `None` durante l’esecuzione; il dizionario finale del risultato di arresto una volta terminata l’operazione. |
| `stats(timeout=10.0)` | Statistiche in tempo reale del processo (fotogrammi scritti, fps effettivi, tempo trascorso). |
| `stop(timeout=60.0)` | Arresta il registratore; restituisce e memorizza nella cache il risultato finale (`.result` rimane `None` se la richiesta di arresto stessa ha fallito). Per una registrazione `record()` che ha dato esito negativo — ovvero una registrazione che non ha scritto alcun fotogramma (`error` `"no-frames: …"`) — il risultato viene memorizzato nella cache su `.result`, quindi genera un&#x27;eccezione su `ChlorosConnectError`. Idempotente (una seconda chiamata restituisce il risultato memorizzato nella cache, senza generare un&#x27;eccezione). L&#x27;uscita da un blocco `with` arresta il registratore senza generare un&#x27;eccezione, quindi controllare `rec.result.get("error")` dopo di esso. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Restituito da `ArraySession.continuous()`. Utilizzarlo come gestore di contesto — uscire dal blocco `with` interrompe l&#x27;acquisizione se è ancora in esecuzione — oppure gestirlo con `status()` / `wait()` / `stop()`. È possibile eseguire una sola acquisizione continua per array alla volta.

| Membro | Descrizione |
| --- | --- |
| `start_status` | Il dizionario di stato restituito dalla chiamata di avvio. |
| `result` | `None` durante l’esecuzione; il dizionario di stato finale una volta terminata l’acquisizione (impostato da `stop()`, oppure da un `status()` / `wait()` che ne ha rilevato la fine). |
| `status(timeout=10.0)` | Dizionario di stato in tempo reale (campi riportati di seguito). |
| `wait(timeout=None, poll_s=0.5)` | Blocca fino al termine automatico dell’acquisizione — raggiungimento di `max_captures` o `duration_s`, esaurimento dell’array, disco pieno, errore — e restituisca lo stato finale. Con `timeout`, restituisce lo stato più recente dopo quel numero di secondi anche se l’operazione è ancora in corso: verificare `running`. |
| `stop(timeout=30.0)` | Interrompe l’acquisizione e restituisce lo stato finale. Idempotente (una seconda chiamata restituisce il risultato memorizzato nella cache). |

Campi di stato:

| Campo | Significato |
| --- | --- |
| `running` | Vero mentre l’acquisizione è in esecuzione. |
| `captures` | Gruppi sincronizzati salvati. |
| `files` | File scritti. |
| `failed_files` | File che non è stato possibile scrivere (come in [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)). |
| `failed_captures` | Tentativi di acquisizione falliti nel loro complesso, compresi quelli in cui l’acquisizione è stata riprovata (un errore HTTP503 mentre l’array non aveva un nuovo gruppo in tempo). |
| `skipped_groups` | Gruppi pubblicati dall’array che non sono stati salvati (l’acquisizione è rimasta indietro). |
| `captures_per_s` | Tasso medio di salvataggio dall’inizio. |
| `elapsed_s` / `started_at` | Secondi trascorsi finora; ora di inizio (secondi dall’epoca Unix). |
| `last_group_seq` | L&#x27;ultimo gruppo sincronizzato salvato. |
| `last_error` | L&#x27;ultimo errore, oppure `None`. |
| `stop_reason` | Una volta terminato: `count` (raggiunto `max_captures`), `duration` (raggiunto `duration_s`), `stopped` (chiamato `stop()`), `array_gone` (l’array è stato disconnesso), `error` (20 errori consecutivi o una richiesta rifiutata dal backend), `disk_full` (un file per cui l’unità non aveva spazio sufficiente). `None` durante l’esecuzione. |
| `max_captures` / `duration_s` | I limiti con cui è stato avviato (`0` = nessuno). |
| `defender_cores` | Core della CPU utilizzati da Microsoft Defender nell’ultimo intervallo di 10 secondi; `None` fino a quando non ne viene misurato uno, e sempre `None` su un host senza Defender. |
| `defender_warned` | Vero una volta che la cattura ha registrato che Defender la sta rallentando (con 1 o più core). Escludere il backend dalla scansione di Defender — vedi [Passaggio 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Frame in tempo reale nella propria pipeline

Per inviare fotogrammi sincronizzati al proprio codice — machine learning, un visualizzatore personalizzato, il proprio registratore — utilizzare `ArraySession.frame_stream()`. Ogni tick genera un `GrabResult`: `{serial: numpy array}` per ogni telecamera, **tutto da un unico trigger**, con l’ID di quel trigger su `.group_id`. Si tratta della stessa implementazione dell’API del progetto `ArrayHandle.frame_stream()`, quindi tutto ciò che è descritto in [Un trigger per ogni lettura](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#allineamento-dei-flussi--allineamento) e [`bit_depth`](#profondità-di-bit--scambio-di-precisione-per-larghezza-di-banda) si applica a entrambi.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Ogni insieme generato costituisce un nuovo trigger.** `frames.group_id` non si ripete mai all’interno di uno stream; quando si richiede un numero di `fps` superiore a quello a cui opera l’array, lo stream attende il trigger successivo, quindi opera alla velocità più bassa tra le due.
- **Le telecamere mancanti vengono indicate per nome, non sostituite.** Una telecamera che non è stato possibile servire in quel trigger è assente dal dizionario ed elencata in `frames.errors`.
- **Si interrompe in modo evidente.** `FrameFetchError`, in caso di rifiuto, si ripeterebbe al tick successivo, dopo 60 secondi senza frame o dopo 60 secondi senza un nuovo trigger.**Cosa offre.**`frame_stream()` trasferisce interi frame numpy tramite la connessione HTTP locale e raggiunge una velocità massima di circa**170 MB/s**. Questo è sufficiente per l’elaborazione in tempo reale di frame grezzi, o di un array raggruppato o di una regione di interesse (ROI), ma non per ogni trigger alla massima risoluzione — per conservare**ogni** trigger, salvare su disco con [`continuous()`](#continuoushandle). Misurato su un array di quattro telecamere (FOCN, FRGN, FRGB, F450) con trasporto compresso; ogni tick corrispondeva a un set completo di un singolo trigger:

| | Sensore completo (array 14,2–14,4 fps) | Binning 2 × 2 (array ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 set/s | 22,2 set/s |
| `frame_stream(processing="debayered")` | 2,3 set/s | 9,3 set/s |
| `frame_stream(processing="display")` | 0,8 set/s | 3,0 set/s |
| MJPEG per telecamera, 4 flussi a 720p | 13,5 fps ciascuno; ogni fotogramma abbinato a un trigger | ~25 fps ciascuno; ~45 % dei trigger presenti su tutti e 4 |

Utilizza `processing="raw"` per la velocità di trasmissione e il demosaic da parte tua; il livello `debayered` rimuove anche l’offset scuro del pacchetto di calibrazione, cosa che invece non fa il demosaic di `raw`.

#### Flussi MJPEG per singola telecamera (senza RTSP)

**RTSP non è supportato.** L’interfaccia di streaming è MJPEG su HTTP (`multipart/x-mixed-replace`, un JPEG per parte, ogni parte con un `Content-Length`), un URL per telecamera:

| URL | Cosa fornisce |
| --- | --- |
| `GET /api/camera/stream/<serial>` | Anteprima a 8 bit, fino a ~30 fps. Per un membro dell’array, `max_h` (impostazione predefinita 720, 64–2160) e `jpeg_q` (impostazione predefinita 80) definiscono le dimensioni e la qualità. |
| `GET /api/camera/<serial>/stream-annotated` | La stessa anteprima con le sovrapposizioni dell’interfaccia grafica (zebra, griglia, mirino, istogramma, peaking, esposimetro spot). `ChlorosLocal.camera_stream()` la legge automaticamente. |
| `GET /api/camera/<serial>/stream-reflectance` | Riflettanza a 8 bit per banda — vedi [Flusso di riflettanza in tempo reale](#live-reflectance-stream-mjpeg). |

- **Intestazioni di trigger.** Per un elemento dell’array, ogni parte di tutti e tre i flussi contiene `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` e `X-Chloros-Timestamp-Ns`. Le parti provenienti da telecamere diverse con lo stesso `X-Chloros-Group-Id` appartengono allo stesso trigger: raccogli le parti e raggruppale in base a quell’intestazione per ricomporre un set. Una telecamera autonoma non ha alcun gruppo: le sue parti di anteprima non contengono nessuna di queste intestazioni (i suoi componenti di riflettanza riportano un `X-Chloros-Group-Id` vuoto).
- **Il flusso di anteprima perde fotogrammi sotto carico.** Fornisce il fotogramma più recente e salta quelli che non è riuscito a inviare in tempo, quindi a frequenze di array elevate non tutti i trigger raggiungono il flusso di ogni telecamera (vedi la tabella sopra). Per ogni trigger, utilizzare `frame_stream()` o `continuous()`.
- **`/api/camera/stream/<serial>` si mette in pausa mentre lo streaming live combinato dell’array è aperto** (`/api/camera/array/stream/<array_id>` — la vista array dell’interfaccia grafica lo apre). Lo streaming di ciascun membro invia quindi un fotogramma segnaposto «In pausa (stream dell’array attivo)», senza intestazioni `X-Chloros-*`, e nient’altro fino alla chiusura dello stream combinato; successivamente riprende automaticamente. `stream-annotated` e `stream-reflectance` non si mettono in pausa. Chiudere la vista array dell’interfaccia grafica quando uno script legge gli stream di anteprima per singola telecamera.
- **Per i numeri, non per le immagini, utilizzare `frame_stream()`.**Il formato JPEG è con perdita di dati e a 8 bit; `frame_stream()` restituisce gli stessi risultati senza alcun codec nel percorso.

### Acquisizione a intervalli per rilievi con droni

Un volo di rilevamento richiede un gruppo di immagini sincronizzate ogni pochi metri di percorrenza, non ogni fotogramma che l’array è in grado di produrre. Due impostazioni controllano questo aspetto:**`target_fps`**imposta la frequenza con cui le telecamere scattano un fotogramma, mentre**`capture_repeated(interval_s=…)`** imposta la frequenza con cui un fotogramma viene salvato.**Le telecamere continuano a trasmettere in streaming tra un salvataggio e l’altro.** Ad ogni intervallo viene salvato il gruppo sincronizzato più recente; i fotogrammi intermedi non vengono scritti. L’esposizione automatica si regola su ogni fotogramma, quindi l’esposizione si è già stabilizzata quando avviene il salvataggio. Non esiste una modalità in cui le telecamere si fermino tra uno scatto e l’altro e vengano riattivate per ciascuno di essi: ogni scatto dovrebbe infatti attendere l’avvio dello streaming e la stabilizzazione dell’esposizione.**Imposta la frequenza dei fotogrammi in base alle esigenze dell’intervallo.**Se lasciata su «auto», una matrice funziona alla frequenza massima supportata dall’host e dalla rete. Impostando una frequenza inferiore si riducono il surriscaldamento delle telecamere, il carico di rete e l’utilizzo della CPU dell’host senza modificare ciò che viene salvato. Scegli una frequenza che garantisca**2–4 fotogrammi per intervallo**:

- Ogni salvataggio attende il fotogramma successivo dopo che è stato richiesto, quindi un gruppo salvato viene acquisito fino a un periodo di un fotogramma dopo il momento previsto. Una frequenza più elevata riduce tale intervallo.
- L’esposizione automatica si regola una volta per fotogramma, quindi una frequenza molto bassa rallenta la velocità con cui l’esposizione segue una nuvola che passa.

**Calcola l’intervallo in base al piano di volo:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Ad esempio, un’area di copertura di 60 m con una sovrapposizione in avanti dell’80 % corrisponde a una distanza di 12 m; a 10 m/s si ottiene `interval_s=1.2`, e un valore fisso di `target_fps=3` corrisponde a un intervallo compreso tra 3 e 4 fotogrammi. L’acquisizione è basata sul tempo: l’SDK non dispone di una modalità attivata dalla distanza o dal GPS, quindi è necessario mantenere una velocità al suolo costante lungo le linee di rilevamento.

**Salvare i dati grezzi a bordo del velivolo e calibrare in un secondo momento.** `interval_s` viene misurato dall’inizio di ogni salvataggio, quindi un salvataggio che richiede più tempo dell’intervallo fa sì che il successivo inizi in ritardo e la distanza tra i fotogrammi subisca una deriva. Il salvataggio dei dati grezzi insieme alla lettura del sensore di luce (`processing="raw"`; la lettura viene salvata per impostazione predefinita) mantiene breve ogni salvataggio e garantisce che ogni fotogramma sia successivamente convertibile in riflettanza.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**fissa la frequenza sin dall’inizio;**`arr.set_target_fps(fps)`**la modifica durante il volo senza riconnettersi, mentre**`arr.set_target_fps(None)`** la riporta in modalità automatica. Una frequenza superiore a quella che l’host e la rete sono in grado di gestire viene ridotta a tale limite massimo — controllare `clamped` nel risultato — e la frequenza minima è 0,1 fps.
- **Non utilizzare `burst()` per ridurre il numero di fotogrammi.** Scrive ogni fotogramma prodotto dall’array; segue un `target_fps` fissato, quindi è lo strumento giusto solo quando si desidera ogni fotogramma.
- **`target_fps` non corregge i fotogrammi danneggiati.** Ridurne il valore diminuisce il numero di frame prodotti, non la velocità di invio di ciascuno; per i frame danneggiati da GVSP, ridurre invece `wire_ceiling_mbps` (vedere [Stato dell’array](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Collegamento a un array già connesso — `attach_array`

Se l’array è già attivo (è stato aperto dall’interfaccia grafica o una precedente sessione SDK ha chiamato `connect_array`), utilizzare `attach_array` per ottenere un handle ad esso invece di riconnettersi. `connect_array` genera sempre l’errore “La telecamera  <sn>è già presente nell’array<id>&quot;, poiché l’invio di un POST con `/array/connect` per un membro del pool non è idempotente; `attach_array` legge `/api/camera/array/list` ed effettua la corrispondenza in base all’array_id o ai numeri di serie.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Modello: gli script SDK, che condividono l’ambiente con la GUI desktop, dovrebbero provare prima con `attach_array` e ricorrere a `connect_array` se nel pool non è ancora presente alcun array.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Un array collegato viene preso in prestito, non posseduto.** Un handle proveniente da `attach_array` non smantella mai l’array: né l’uscita dal blocco `with` né un rilascio esplicito tramite `disconnect()` lo liberano (la chiamata esplicita stampa una riga su stderr indicando che ha lasciato l’array in esecuzione). Stessa regola di `CameraSession` / `DAQSensorSession`. Pertanto, la condivisione con la GUI è sicura con o senza `with`:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Quando si desidera terminarlo — ad esempio per riconnettersi con un `wire_ceiling_mbps` diverso — chiamare `arr.force_disconnect()`. La riconnessione dopo un semplice `disconnect()` su un handle collegato fallisce con il messaggio &quot;La telecamera è<sn> già nell’array<id>&quot;, poiché nulla è stato liberato.

### Strumento di analisi della rete

Utile prima di aprire l’array — verifica se le impostazioni proposte sono compatibili:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Non avvia mai un backend: se uno non risponde, genera l’errore `ChlorosConnectError`, mentre una richiesta rifiutata viene restituita come risultato. Un errore `packed` genera l’errore `ValueError`.

`status` è uno tra `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (altrimenti `error` — inclusa una richiesta rifiutata: accesso non effettuato, `error_code: AUTH_REQUIRED`, o un piano al di sotto della soglia minima dell’SDK, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` indica che la risoluzione richiesta si adatta all’anello RX solo a una frequenza di trigger limitata — mantenere la risoluzione e passare da `target_fps=result["recommended"]["recommended_target_fps"]` a `connect_array` (vedi [Esempio 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` compare solo quando il fotogramma richiesto non può essere trasmesso a nessuna frequenza di fotogrammi; connect non riduce mai i fotogrammi di propria iniziativa, quindi applicare `recommended` manualmente.

**Come interpretare la proiezione** (stesso modello del pannello «Impostazioni array» dell’interfaccia grafica):

- **Burst (`frame_bytes_total`) viene sommato per ciascuna telecamera nel formato pixel reale di ciascuna telecamera.**Le telecamere**M3M**in modalità mono trasmettono in Mono12 (2 bit/px) oppure, con un `pixel_format`, il formato mono della propria profondità (`"BayerRG8"` è `Mono8`, 1 B/px), quindi in assenza di `Mono8` un fotogramma a piena risoluzione con 4 telecamere è di**~25 MB** con tre telecamere mono, non i ~12,6 MB che si otterrebbero ipotizzando un formato interamente a 8 bit. Il backend determina il formato di ciascuna telecamera in base al suo modello.
- **L’ammissione (`burst_fits_nic_ring`) tiene conto del drenaggio**, non si basa sul confronto tra «intero burst» e «anello»: la modalità «sim-emit» si attiva quando l’host svuota l’anello RX più velocemente di quanto le cam lo riempiano. Un host da 10G + cam da 1 GbE**ammette** la piena risoluzione anche quando il burst supera la capacità dell’anello; un host da 1 GbE blocca (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` rappresenta un limite massimo conservativo per il recupero seriale** — `max(readout+emit, N×emit)` con l’emissione per singola cam limitata al Camera Link da 1 GbE, indipendentemente dall’esposizione. Ad es. ~2,8 fps per un array a 4 telecamere a piena risoluzione a 12 bit (corrisponde ai ~2,7–3,0 misurati dal runtime). Modello completo: [Riferimento CLI → Modello fps e burst dell’array](cli-reference.md#array-fps--burst-model).
- **Dopo un&#x27;acquisizione continua, la proiezione indica il valore raggiunto.** Un&#x27;acquisizione di 30 s o più a una frequenza automatica registra la frequenza a cui si è stabilizzata per quella dimensione dell’array su questo computer; i brevi tentativi del regolatore di frequenza di raggiungere una velocità maggiore vengono tralasciati. Da quel momento in poi, la riga di `array-connect`, l’analisi di `connect_array()` e il pannello Impostazioni array riportano tale frequenza quando è inferiore a quella del modello, indicando cosa l’ha limitata in `cycle_limiter`: `"clock"` (l’array funzionava alla frequenza a cui si era stabilizzato, mentre il ciclo di acquisizione era in attesa del proprio clock di trigger), `"loop"` (il ciclo di acquisizione non aveva più tempo a disposizione), `"capture"` (l’acquisizione è rimasta indietro e ha saltato dei gruppi) o `"measured"` (non è specificato quale dei due casi). Il valore del modello è riportato in `wire_fps_max`. L’array continua a crescere fino a raggiungere il valore trasportato dal cavo.
- **L’oversubscription (`oversubscribed: true`) significa che il cavo non è in grado di trasportare l’array a nessuna frequenza raggiunta dalle telecamere** — i campi fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) indicano 0, e il binning non può risolvere il problema (riduce i byte per fotogramma, non i byte sincronizzati al secondo). Una connessione lenta di per sé non causa questo problema: la frequenza dei fotogrammi diminuisce invece, e se le telecamere non possono essere sincronizzate a una velocità sufficientemente bassa, il livello diventa `slip-emit-and-capture` (seriale). Le soluzioni sono: un numero inferiore di telecamere, frame jumbo o una scheda di rete più veloce; `max_cams_collision_safe` indica il limite massimo della vostra connessione e delle telecamere. La risposta contiene anche `aggregate_demand_bps`, `collision_safe_ceiling_bps` e `per_cam_floor_bps`. Vedi [Over-Subscription](#over-subscription-the-per-cam-floor).

### Rilevamento ed elenco

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Ciascuna accetta solo parole chiave `backend_url=` e `timeout=30.0` e restituisce un elenco di dizionari. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) accetta anche `auto_start_backend=True`; le chiamate a `list_*` non avviano mai un backend.

---

## Smart-AE / Smart-Capture

Gli array LATTICE eseguono l’AE in modo continuo in background non appena vengono collegati, ma una scena appena inquadrata richiede un attimo per convergere. **Smart-capture** è la soluzione preconfigurata: interroga l’esposizione di ciascuna telecamera, attende che l’array sia stabile su un’intera finestra, quindi avvia l’acquisizione. È equivalente all’interfaccia grafica: il pulsante di acquisizione “smart” dell’app desktop richiama lo stesso endpoint del backend.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Quando si utilizza `ChlorosProject` (sezione successiva) sono disponibili ulteriori opzioni di regolazione:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

La politica smart-AE è conservativa per impostazione predefinita. Restringete `exposure_tolerance_pct` per lavori radiometrici meticolosi; allargatela per scene in rapida evoluzione in cui vi basta un risultato “sufficientemente preciso”.

---

## Sessioni dei sensori DAQ

Pool di backend persistente per sensori spettrali (DAQ-U su USB, DAQ-M su BLE, DAQ-E su Ethernet). Rispecchia la superficie della fotocamera: rilevamento intelligente, riutilizzo del pool, collegamento idempotente.

### Rilevamento intelligente (configurazione automatica)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Priorità: Ethernet → BLE → USB. Passare un qualsiasi suggerimento esplicito per fissare il trasporto.

### Trasporto fissato

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` specifica un sensore, e solo quel sensore viene mai aperto. Viene confrontato con i sensori già aperti, quindi viene richiesto il nome host o l’IP di un DAQ-E (a un DAQ-E che la scansione di rete rileva solo tramite IP vengono richiesti, in sola lettura, il nome, l’indirizzo MAC e il numero di serie, quindi anche uno qualsiasi di questi lo individua), l’indirizzo MAC BLE di un DAQ-M e l’ID che un DAQ-U o un DAQ-M riporta una volta aperto (qualsiasi altro dispositivo aperto per la verifica viene richiuso). Se non viene trovato, `connect_daq_sensor` genera un evento `ChlorosConnectError` elencando ciò che è stato trovato; includendo anche un indirizzo (`port=` / `mac=` / `eth_host=`) che contiene un sensore diverso, genera anch’esso un errore. Non ricade mai sul sensore visibile — ecco a cosa serve la chiamata con il flag zero.</id></sn></id></sn>

### Firma `connect_daq_sensor()`

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Un sensore già presente nel pool viene restituito così com&#x27;è (preso in prestito): `integration_time`, `frame_avg` e `enable_ae` non vengono applicati ad esso. Scollegarlo prima per aprirlo con le proprie impostazioni. `cap_id` costituisce l’eccezione: su un sensore preso in prestito viene impostato con `set_cap()`. Se omesso, vale l’impostazione predefinita del backend (vedere la nota sotto la tabella dei metodi).

### Metodi `DAQSensorSession`

| Metodo | Descrizione |
| --- | --- |
| `status(timeout=10.0)` | Riepilogo delle voci del pool (stato di streaming/registrazione, intervallo di lunghezze d’onda, sha di calibrazione, tempo di integrazione, frame_avg, stato AE). Per un DAQ-E il cui flusso si è interrotto: `reconnect` (dove è attiva la riconnessione automatica del backend: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` mentre trasmette), `stream_lost_s`, `reconnects` e `last_reconnect`, e `record_gaps` per la registrazione in corso; `rx_stream_fault` indica lo stato di riconnessione fino all’arrivo del primo spettro. |
| `latest(*, n=1, timeout=10.0)` | Restituisce fino a N frame di spettro più recenti (ciascuno `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Riprende / mette in pausa lo streaming (l&#x27;handle rimane aperto). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Avvia la registrazione di un file .daq. Restituisce il percorso del file. Non è supportato per DAQ-U/M senza un pacchetto di calibrazione AWS (DAQ-E è escluso). |
| `record_stop()` | Interrompe la registrazione. Restituisce `{path, rows, warning, gaps}`; `warning` è uguale a `None` in caso di registrazione corretta, altrimenti la diagnosi del backend (ad esempio un file vuoto o un&#x27;interruzione nel flusso), che viene anche stampata su stderr. `gaps` elenca ogni intervallo in cui il flusso non ha fornito nulla: `start_utc`, `end_utc` (`None`: ancora perso all&#x27;arresto), `duration_s`, `reconnected`, `marked_in_file`. È vuoto in caso di registrazione continua. |
| `set_cap(cap_id, timeout=15.0)` | Imposta il profilo di correzione del limite massimo applicato alle letture a partire dalla successiva: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"`, oppure `"none"` (o `None`) per un sensore nudo. Restituisce il, il cui valore `cap_id` lo riporta. Un DAQ-E-S mantiene il valore `as_recorded` indipendentemente da ciò che viene richiesto (il suo diffusore è integrato). Genera il valore `ChlorosConnectError` per un profilo che non si carica. CLIA: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Solo DAQ-E: riattiva il PTP per un&#x27;unità il cui PTP è disattivato (`status()["rx_ptp_enabled"]` corrisponde a `False`; l’impostazione viene salvata sull’unità, quindi rimane disattivato finché non viene riattivato). Restituisce lo stato aggiornato. Genera l’errore `ChlorosConnectError` per un DAQ-U / DAQ-M, oppure quando l’unità non conferma. Chloros non riattiva mai un’unità di propria iniziativa, poiché lo stato disattivato può essere intenzionale. CLI: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Solo DAQ-E: impostazioni proprie dell’unità. `firmware` (`running`, `bundled`, `update_available`), `ip` e `ip_mode` (`dhcp` / `linklocal` / `static`: quello attualmente in esecuzione), `network` (l’impostazione salvata: `mode`, più `ip` / `mask` / `gateway` in modalità statica), `network_change_pending` (vero fino a quando l’unità non esegue l’impostazione salvata, quella salvata con `set_network(..., reboot=False)` inclusa), `auto_stream`, `ptp`. `firmware["update_available"]` è `None` quando la versione dell’unitànon è stato possibile leggerne la versione. Finché il flusso dell’unità non raggiunge questo computer, viene letto su una nuova connessione e il dizionario aggiunge `stream_silent` (`True`), `stream_fault` e `reconnect`; un’unità che non risponde genera un errore. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E solo: DHCP (`"auto"`) o un indirizzo statico. Riavvia l’unità per applicare l’impostazione e la ricollega; questa sessione continua a funzionare e `address` segue l’unità. `reboot=False` si limita a salvarla. Vedere [Impostazioni del dispositivo DAQ-E](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Solo DAQ-E: trasmissione all’accensione senza software collegato (salvato sull’unità). Restituisce l’impostazione confermata. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Solo DAQ-E: riavvia l’unità e ricollegala con le impostazioni precedenti. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Solo DAQ-E: esegue il flashing del firmware incluso in questa versione di Chloros su un&#x27;unità che esegue una versione precedente, quindi la ricollega (circa un minuto; non spegnerla). Restituisce `updated`, `fw_before`, `fw_after`, `bundled` e un `warning` quando l’unità ha impiegato molto tempo a rispondere o ha risposto con una versione diversa; `updated` corrisponde a `False` per un’unità già aggiornata. Genera l’errore `ChlorosConnectError` per un’unità la cui versione non può essere letta. CLI: `pool-update-firmware`. |
| `disconnect()` | Rilascio dal pool. Nessunaper gli handle collegati ma non di proprietà. Su un sensore aperto da questa sessione è `force_disconnect()` e genera l&#x27;errore `ChlorosConnectError` quando il rilascio non viene confermato (l&#x27;uscita dal blocco `with` non genera mai un&#x27;eccezione). |
| `force_disconnect(timeout=30.0)` | Rilascia il sensore indipendentemente da chi lo abbia aperto — ad esempio un sensore lasciato nel pool da uno script che si è bloccato, che un nuovo script raggiunge solo come handle collegato il cui `disconnect()` non esegue alcuna operazione. La chiusura del sensore di un altro client interrompe qualsiasi registrazione in corso. Ritorna una volta che il backend conferma il rilascio, oppure quando non c’è nulla da rilasciare (il sensore non è nel pool o nessun backend è in esecuzione); un rilascio che ha superato il timeout o che è stato rifiutato dal backend genera l’errore `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Attributi: `sensor_id`, `model`, `transport`, `address` (segue un DAQ-E che `set_network()` ha reindirizzato), `info` e `backend_url`.

> **Indicare il nome del cappuccio.** I modelli DAQ-U, DAQ-M e DAQ-E vengono forniti senza condensatore e Chloros non è in grado di rilevare se ne sia montato uno: un sensore collegato senza `cap_id` riceve il valore `sunshine_cosine`, un valore che, secondo le registrazioni, si presume sia stato generato (`cap_id_source=auto_default`). Un DAQ-U senza cappuccio, con tale correzione, fornisce una lettura circa 30 volte superiore; pertanto, passare `cap_id="none"` per un sensore senza cappuccio (o `set_cap("none")`), mentre il profilo del cappuccio montato indica il contrario.

### DAQ-E-S — diffusore fisso e sensore di inclinazione

Il **DAQ-E-S** è un DAQ-E con due differenze rilevanti per il software. Tutto il resto — individuazione, connessione, PTP, registrazione, le impostazioni del dispositivo riportate di seguito — funziona esattamente come per un DAQ-E.**Il suo diffusore è fisso, quindi non richiede alcuna correzione del cappuccio.** Il diffusore di tipo Sunshine è integrato ed era presente sull’unità al momento della calibrazione di fabbrica, pertanto tale calibrazione indica già l’irraggiamento verso il basso effettivo. Chloros imposta il proprio limite a `as_recorded` (senza profilo) indipendentemente da quanto richiesto: un limite denominato in suo onore (`sunshine_cosine`, `fov_*`, `none`) viene ignorato con un messaggio, e le sue registrazioni memorizzano `cap_id_source=model`, quindi l’avviso relativo al limite presunto non viene mai attivato per questo dispositivo. Non applicare nemmeno manualmente un limite ai suoi dati: `sunshine_cosine` in posizione orizzontale darebbe una lettura circa 11 volte superiore.

**È dotato di un accelerometro a 3 assi.** La risposta cosinusoidale del diffusore solare è valida solo quando è in posizione orizzontale, quindi un sensore inclinato riporta un’irradianza errata — e la riflettanza eredita l’errore — senza altri sintomi. Ogni lettura del DAQ-E-S riporta quindi il proprio assetto:

| Campo | Significato |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Angoli in gradi. `None` quando il firmware contrassegna l’angolo come indefinito (caduta libera, asse inattivo). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Accelerazione in milli-g. `mag_mg` lontano da ~1000 indica che l’unità era in movimento e che gli angoli non sono significativi per quel campione. |
| `cal_applied` | Impostato: gli assi e gli angoli sono nel sistema di riferimento del diffusore (viene applicata la calibrazione di montaggio dell’unità). Disattivato: sistema di riferimento grezzo della scheda, che può mostrare fino a ~2,3° di inclinazione non reale. |
| `sample_age_ms` | Età del campione dell’accelerometro al momento dell’invio dello spettro. |
| `temp_raw` | **Andamento** della temperatura del die non calibrato — solo differenze, mai una lettura del termometro. |

`DAQSensorSession.latest()` li restituisce sotto la chiave `imu` di ciascun frame (`None` su un DAQ-U, DAQ-M o DAQ-E standard). Un `.daq` registrato da un DAQ-E-S memorizza gli stessi valori nelle sue colonne `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) accanto a ogni spettro, in modo che il lettore possa distinguere una nuvola da un treppiede urtato. Su tutti gli altri modelli quelle colonne sono vuote.

Il flusso di dati sull’assetto riporta i propri campi in `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), tenuti separati da quelli spettrali in modo che un’interruzione dell’accelerometro non venga mai interpretata come una lacuna nella registrazione. `discover_daq_sensors()` e l’`model` della sessione indicano `DAQ-E-S`.

### Impostazioni del dispositivo DAQ-E

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **È necessario un indirizzo statico sulla rete di questo computer.** `set_network` genera l’errore `ChlorosConnectError` per chi ha una sottorete che non include l’sulla rete del sensore: dopo il riavvio, nessun dispositivo potrebbe raggiungere il sensore per ripristinare l’impostazione precedente. Impostare `force=True` solo quando a questo computer verrà assegnato un nuovo indirizzo o verrà reindirizzato alla nuova sottorete.
- **`set_network`, `reboot` e `update_firmware` riavviano il sensore** e generano `ChlorosConnectError` mentre questo sta registrando, e quando non si ripristina entro `comeback_timeout` secondi. L’ID del sensore e il nome host `daq-e-<id>.local` non cambiano mai.
- **Una registrazione avviata poco prima del riavvio viene chiusa da quest’ultimo e le viene assegnato un nome.** Quando il sensore torna operativo, il dizionario restituito contiene `recording_closed` (il percorso del file) e un `warning` che lo indica. Se non si riconnette, o se il trasferimento del firmware fallisce, lo indica invece il `ChlorosConnectError`: il suo messaggio termina con l’avviso, e vengono impostati `e.recording_closed` / `e.warning` (entrambi `None` se non è stata chiusa alcuna registrazione). Avviare nuovamente la registrazione una volta che il sensore è tornato in linea.
- **Viene ricollegato solo il sensore stesso.** Dopo il riavvio, l’unità deve rispondere indicando dove si trovava -- e, dopo `set_network("auto")`, anche con il proprio nome `.local` -- indicando il proprio nome, nome host e modello, tramite l’interfaccia di rete (MAC) di cui disponeva; `address` diventa l’IP che ha risposto. Una risposta con quel nome è valida solo con quel MAC; pertanto, da un computer che raggiunge il sensore tramite un router o dalla rete NAT predefinita di WSL2, un sensore spostato dal DHCP non viene ricollegato (`ChlorosConnectError`; ricollegarlo al suo nuovo indirizzo).
- **Un aggiornamento del firmware richiede che il sensore si ricolleghi.** `update_firmware()` invia una richiesta all’unità, e l’unità apre una connessione TCP verso questo computer per ricevere l’immagine, su una porta scelta dal sistema operativo. Il backend ascolta sulla scheda di rete che raggiunge l’unità (su tutte le schede solo quando non è in grado di individuarla) e serve esclusivamente quell’unità: qualsiasi altra connessione viene rifiutata (`[DAQ-FW] … refused a firmware connection from <ip>` nell’output del backend) mentre rimane in attesa. Un firewall che blocca la connessione fa fallire l’aggiornamento con `device never connected back on TCP <port>`. Il programma di installazione di Windows aggiunge la regola necessaria (vedere [Passaggio 1](#windows-exe)); su un altro firewall, consentire il traffico TCP in entrata verso il backend dalla sottorete del sensore.
- **In WSL2, utilizzare la rete mirrorata** (`networkingMode=mirrored` in `.wslconfig`): nella modalità NAT predefinita ogni indirizzo statico viene rifiutato (l’indirizzo di WSL non si trova sulla rete del sensore), il flusso non raggiunge WSL e l’aggiornamento del firmware non può essere completato.
- L’interfaccia grafica mantiene la propria connessione con un sensore: un riavvio tramite script termina la sessione dell’interfaccia grafica con esso. L’interfaccia grafica presenta le stesse impostazioni (scheda «Sensori di luce», impostazioni del sensore, «Rete»).

### Rilevamento — individuazione di un indirizzo a cui connettersi

`discover_daq_sensors()` esegue una scansione su USB / BLE / ETH alla ricerca di sensori che *potresti* aprire. È la controparte DAQ di `discover_lattice_cameras()` e l’unico modo per ottenere il **MAC BLE di un DAQ-M** — un DAQ-E ha un nome host e un DAQ-U una porta COM, ma il MAC non è né stampato sul dispositivo né elencato dal sistema operativo.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Campo | Descrizione |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Porta COM / MAC BLE / nome host — passare a `connect_daq_sensor` come `port=` / `mac=` / `eth_host=`. |
| `display` | Etichetta leggibile dall’utente. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, oppure `None`. Un sensore Ethernet è `DAQ-E-S` o `DAQ-E`, poiché si identifica come tale quando richiesto, in sola lettura, durante la scansione (`DAQ-E` quando non risponde in tempo). Una porta seriale USB viene identificata come `DAQ-U` quando il suo chip bridge è uno di quelli utilizzati da DAQ-U (FTDI, CH340, CP210x, Prolific) — un indizio fornito dalla descrizione della porta, non una sonda — e `None` per qualsiasi altro dispositivo USB, che è comunque elencato, quindi è possibile passarlo come `port=`. Una porta seriale senza ID fornitore USB e senza un chip bridge di questo tipo (un UART della scheda madre come `/dev/ttyS*` di Linux, un collegamento seriale Bluetooth, Intel AMT) non può essere un DAQ-U e non è elencata. |
| `extra` | Dettagli per tipo di trasporto (nome pubblicizzato BLE; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). I valori vuoti vengono omessi. |

| Parametro | Predefinito | Descrizione |
| --- | --- | --- |
| `transports` | tutti e tre | Sequenza (o stringa CSV) che limita la scansione. È consigliabile specificarla quando si sa esattamente cosa si desidera — il BLE è la parte più lenta. |
| `scan_timeout` | 5 | Finestra di scansione per trasporto in secondi; il backend limita il valore a un intervallo compreso tra 1 e 20. |
| `timeout` | 60,0 | Limite massimo HTTP per l’intera chiamata (come altrove nell’SDK). |
| `auto_start_backend` | `True` | Avvia un backend locale se non ne è in esecuzione nessuno. Non avvia mai un backend per un `backend_url` remoto. |

> **I sensori già aperti nel pool non vengono visualizzati.** Una periferica BLE connessa smette di trasmettere la propria pubblicità e una porta COM aperta non può essere rilevata, quindi l’elenco dei dispositivi rilevati riporta solo ciò che è *disponibile per la connessione*. È normale ottenere un risultato vuoto subito dopo aver collegato qualcosa: usa `list_daq_sensors()` per ciò che hai già in mano. I trasporti la cui scansione non può essere eseguita (bleak / zeroconf non installati) vengono ignorati anziché generare un errore, quindi una macchina senza Bluetooth riceve comunque le risposte relative a USB ed ETH.

### Elenco

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Quale DAQ alimenta quale telecamera — `daq_bindings()`

La riflettanza di un&#x27;acquisizione e il file `.daq` in cui viene salvata utilizzano il DAQ associato a quella telecamera o a quell&#x27;array, altrimenti il DAQ più recente disponibile nel backend. La selezione del sensore di luce nell&#x27;interfaccia grafica e il file `chloros-cli daq pool-bind` lo impostano; così come:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` restituisce `{"cameras": {serial: b}, "arrays": {array_id: b}}`: ogni `b` contiene `binding` (il valore impostato), `source` (`project`, dall’`sensors.json` del progetto, oppure `runtime`), `known` / `known_as` (se la chiave della fotocamera o dell’array indica un nome noto a Chloros, e in che modo: `connected`, `project` — l’`cameras.json` del progetto aperto — oppure, per una telecamera, `seen`: già aperto su questo computer in precedenza; `known: False` per una chiave che non identifica nulla) e `resolved` (il sensore del pool a cui è ora associato: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Un binding in fase di esecuzione ha la precedenza su quello del progettoper la sessione; il file di progetto non viene modificato. La disconnessione di una telecamera, di un array o di un DAQ interrompe ogni associazione a quell’hardware, indipendentemente da chi l’abbia impostata (i membri di un array vengono associati all’array stesso).

`set_daq_bindings()` associa solo una telecamera che sia collegata, presente nel progetto aperto o precedentemente aperta su questo computer, e un array che sia collegato o presente nel progetto aperto (o un tasto già associato). Qualsiasi altro tasto genera `ChlorosConnectError` indicandolo (`400`, `code`, `UNKNOWN_BINDING_TARGET`, con i tasti `unknown`), e **nulla** in quella chiamata viene associato. `None` cancella qualsiasi chiave.

### Riflettanza ambientale + oggetto — `light_sensor_reflectance()`

Due DAQ in pool, uno rivolto verso il cielo (ambientale) e uno verso il bersaglio (oggetto), forniscono la riflettanza del bersaglio e gli indici di vegetazione dalle loro ultime letture: la coppia Ambientale + Oggetto della GUI e `chloros-cli daq reflectance` senza dover riattivare i sensori una seconda volta. Entrambi devono essere collegati e in streaming.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Bande.** La riflettanza di ciascuna banda è l’irraggianza integrata della bandasull’irraggianza ambientale. `daq_band_presets(filter_code=None)` elenca gli insiemi di bande: uno per ogni filtro LATTICE M3C (`RGN`, `OCN`, `NGB`, il modello di banda proprio della telecamera, quindi un NDVI DAQ e un NDVI della telecamera sono la stessa grandezza) e `GENERIC` a banda larga. Il suo `suggested` è l’impostazione predefinita corrispondente a `filter_code`, altrimenti è il filtro della prima telecamera collegata (`suggested_from` indica quale).
- **Restituisce** `bands` (per banda `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (iintegrati), `custom` (ciascuno con `value`, oppure `valid: False` e `error`), `invalid_bands`, `reflectance_type`, `preset` e le due letture `timestamps` (`ts_ns`). Una banda che il sensore ambientale non è riuscito a misurare (fuori dal suo intervallo, al livello di rumore di fondo) è considerata non valida, con l’indicazione del motivo, e ogni indice che ne ha bisogno viene omesso, senza mai essere calcolato rispetto allo zero.
- **Apparente o relativo.** Gli indici dipendenti dalla scala (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) richiedono un rapporto assoluto, quindi vengono calcolati solo quando entrambi i sensori sono stati sottoposti a calibrazione di potenza (ognuno ha un `calibration_bundle_sha`): `"Apparent Reflectance"`. Altrimenti il risultato è `"Relative Reflectance"`, con solo gli indici invarianti rispetto alla scala.
- **Due diverse griglie di lunghezze d’onda** vengono confrontate in base alla loro sovrapposizione: i punti del sensore ambientale all’interno del raggio d’azione dell’oggetto, lo spettro dell’oggetto interpolato su di essi.
- **Due sensori, mai uno solo.** Assegnare a un unico sensore entrambi i nomi (lo stesso ID, in qualsiasi forma ortografica accettata dal pool) genera l’errore `ChlorosConnectError`: la lettura di un sensore su se stesso è pari a 1,0 su ogni banda, il che non misura nulla. Anche il backend lo rifiuta (`POST /api/daq/indices` risponde a `400`, `code` a `SAME_SENSOR`, mentre `ambient_sensor_id` e `object_sensor_id` inviati dall’SDK corrispondono a un unico sensore in pool, ad esempio il suo ID e la sua porta).

### Condivisione con la CLI e altri script

Il pool di sensori DAQ del backend è condiviso da ogni client CLI e SDK presente sulla macchina. Se un sensore è già presente nel pool — aperto da `chloros-cli daq pool-connect` o da un altro script — la chiamata a `connect_daq_sensor(port="COM3")` da Python restituisce un handle preso in prestito: il suo `disconnect()` è un’operazione nulla, quindi lo script non sottrae il sensore al client che lo ha aperto all’uscita dall’ambito. Lo stesso vale per `sensor_id=` (digitato con `:` o `-`, entrambi validi), per un DAQ-E identificato tramite il proprio IP quando è stato aperto tramite nome host (o viceversa), e per il rilevamento intelligente con flag zero quando il backend risponde con un sensore già in suo possesso — un blocco `with` rilascia sempre e solo un sensore che ha aperto esso stesso.

La GUI di Chloros **non** fa parte di questo pool: il suo pannello DAQ gestisce la propria connessione al sensore, quindi `connect_daq_sensor` non prende mai in prestito un sensore che la GUI ha aperto. Non utilizzare lo stesso sensore contemporaneamente dalla GUI e da uno script. Un DAQ-U o DAQ-M gestito dalla GUI normalmente non si apre, mentre un DAQ-E accetta una seconda sessione: la connessione dello script modifica le impostazioni di acquisizione dell’unità gestite dalla GUI, e il suo `disconnect()` interrompe il flusso dell’unità, il che termina la visualizzazione in tempo reale della GUI e qualsiasi registrazione in corso. Chiudere prima il sensore nella GUI.

### PTP Grandmaster — `time_sync_status()`

Il backend esegue il PTP grandmaster da cui le telecamere LATTICE e le unità DAQ-E ricavano i propri timestamp. Si avvia su richiesta: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` e `connect_all()` lo avviano se non è in esecuzione — compresa una connessione che poi fallisce, come nel caso di `connect_all()` che tenta di connettersi a un DAQ-E non presente — e lo stesso vale per `test_array_sync()` e ogni chiamata che apre le telecamere per un comando (`capture_cameras_transient()`, `start_multi_camera_stream()`, le chiamate di allineamento, `capture_live_index()`).

- **Dove viene annunciato.** Sugli adattatori di rete cablati collegati alle telecamere (incluso un indirizzo link-local `169.254.x.x`), altrimenti sull’adattatore della route predefinita se quest’ultimo è cablato. Mai su Wi-Fi o su un adattatore virtuale (Tailscale, WSL): dietro di essi non c’è nessuna telecamera né nessun DAQ-E. In assenza di tali schede attive, il sistema continua a trasmettere l’annuncio su nessuna scheda (`health` `NO_SOCKETS`) e si associa alla scheda che si attiva al collegamento della telecamera o del DAQ successivo.
- **Dopo un’interruzione del collegamento.** Quando l’adattatore della telecamera perde il collegamento e lo riacquista (a seguito di uno sblocco del cavo, della sostituzione di uno switch o della rianumerazione di un adattatore USB), il grandmaster ricostruisce autonomamente i propri socket, senza bisogno di riavvio, e ricontrolla ogni DAQ-E connesso, riattivando quello rimasto inattivo. Inoltre, ricostruisce la configurazione quando tutte le telecamere e i DAQ-E che stava servendo rimangono inattivi per 15 secondi.
- **Raggiunge il cavo?** Leggi `time_sync_status()` (`GET /api/time-sync/status`). `state` indica solo che il thread del grandmaster è in esecuzione; `health` indica se i suoi pacchetti vengono inviati: `OK` (nessun errore rilevato), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (alcuni adattatori non funzionanti) oppure `PEERS_SILENT` (le telecamere e i DAQ-E a cui forniva servizio hanno smesso di rispondere), con una frase in `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` contano solo i messaggi in uscita; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` e `last_delay_req_age_s` descrivono quelli che non sono stati inviati, mentre `rebinds` / `last_rebind` indicano le ricostruzioni. `sync_sends_slow` conta le sincronizzazioni il cui invio ha superato i 200 us e `sync_send_max_us` è la più lunga; il tempo di origine di ogni Follow_Up conta al massimo 200 us del proprio invio. Il grandmaster viene eseguito in un processo a sé stante (`own_process`, `process_pid`), con priorità elevata su Windows (`process_priority`), quindi un backend occupato non può ritardare i tempi che registra e sincronizzare l’orologio di ogni telecamera contemporaneamente; se quel processo non può essere avviato, viene eseguito all’interno del backend come in precedenza (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funzione | Cosa fa |
| --- | --- |
| `time_sync_status()` | Stato, integrità, identità dell’orologio, priorità e contatori dei messaggi del grandmaster (sopra). `chloros-cli time-sync status`. |
| `time_sync_peers()` | Gli slave PTP che rileva (telecamere, DAQ-E): `peers`, ciascuno dei seguenti: `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | `PtpStatus`, `PtpOffsetFromMaster` e `PtpMeanPathDelay` (una telecamera il cui firmware non presenta nodi PTP dispone solo del numero di serie). `time-sync cameras`. |
| `restart_time_sync()` | Arresta e riavvia il grandmaster; il PTP di ogni DAQ-E in pool viene ricontrollato dopo (`daq_ptp_rechecked`). Genera un evento `ChlorosConnectError` quando non si ripristina. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Le sue priorità PTP (0-255; il valore più basso vince l’elezione del best-master), per una rete con un altro master PTP. Richiede che sia in esecuzione. `time-sync set-priority`. |

Ciascuno prende `backend_url=` e `timeout=` e avvia un backend locale che non è in esecuzione.

---

## Utilità per le telecamere

Cosa fa `chloros-cli lattice` oltre alla connessione e all’acquisizione, come semplici funzioni in `chloros_sdk` (modulo `chloros_sdk.camera_utils`). Ciascuna chiama il percorso del backend utilizzato dal comando CLI, con le regole della CLI, quindi le due si comportano allo stesso modo. Accettano un **numero seriale** della telecamera, non una sessione, e le stesse parole chiave `backend_url=` / `timeout=` delle chiamate smart-connect; se il backend non è in esecuzione, viene avviato, come fa `connect_camera()`. Un rifiuto genera un errore `ChlorosConnectError` con il motivo fornito dal backend.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funzione | Cosa fa |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Il profilo colore dello schermo: `raw`, `linear`, `natural`, `enhanced` o `custom_temp`, e la sua temperatura di colore (limitata a 2000-10000 K; letta quando il profilo è `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Saturazione e contrasto dello schermo, 0-200 % ciascuno (100 = invariato); `reset=True` riporta entrambi a 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Riduce la luminosità dei pixel più luminosi e più bianchi del rendering dello schermo: da -100 a 0 (troncato verso lo zero, poi limitato; 0 = spento). |
| `camera_white_balance_once(serial, settle_s=None)` | Bilanciamento del bianco in un unico passaggio: la telecamera trasmette brevemente, converge e restituisce `red_ratio` / `blue_ratio` (`ratios_read` False quando non è in grado di segnalarli). Mantiene il bilanciamento del bianco che ha calcolato; ogni altra impostazione scritta dal suo file di configurazione viene ripristinata così com’era. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (continuo) oppure `"off"`; oppure rapporti di bilanciamento fissi `red` / `blue`, che disattivano per primi la modalità automatica. Un rapporto rifiutato dalla fotocamera è indicato con il nome `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Le impostazioni predefinite della fotocamera; applicandone una, la fotocamera mantiene le impostazioni indicate dai nomi delle impostazioni predefinite (la risposta è `settings`, poiché la telecamera li legge in seguito, ad esempio la frequenza dei fotogrammi a cui opera; `None` per un campo che il preset non nomina), mentre ogni altra impostazione scritta dal file di apertura viene ripristinata così come trovata. `"default"` ripristina la telecamera alle impostazioni predefinite di fabbrica (impostazioni di fabbrica con esposizione automatica a funzionamento libero, limite di throughput del collegamento e correzione dei difetti disattivati, bilanciamento del bianco calibrato di una telecamera RGB mantenuto), non salvate nella sua memoria: la risposta contiene `factory_defaults: True` e tutti i campi di `settings`. In `camera_presets()`, `default` è `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | I modelli di filtro e i dettagli di un modello (bande, numero di condizione della risposta spettrale). Il modello di filtro di una telecamera viene letto dalla sua stringa di modello; non viene impostato manualmente. |
| `camera_firmware_status(serial=None)` | Il firmware di ciascuna fotocamera: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` e `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Installare il firmware di questa build di Chloros su una telecamera: ripristino delle impostazioni di fabbrica, installazione, riavvio; le impostazioni di base di Chloros vengono riscritte; la calibrazione della telecamera viene mantenuta. Ci vogliono alcuni minuti; la telecamera deve rimanere accesa. `force=True` esegue un nuovo flashing su una telecamera già in funzione. |
| `camera_power_probe(serial)` | I nodi di alimentazione e termici della telecamera e la sua temperatura. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Modalità di standby a basso consumo della telecamera (rimane in standby fino al risveglio: il limite di throughput del collegamento è attivato (`DeviceLinkThroughputLimitMode` `On`) al suo valore minimo, la modalità di risparmio energetico è attiva se la telecamera ne dispone), con la temperatura prima e dopo; il risveglio riporta il collegamento nelle condizioni in cui si trovava al momento della sospensione (il limite, la modalità e il ritardo dei pacchetti `GevSCPD`, che la telecamera mantiene a 0 mentre il limite è attivo; il registro viene conservato su questo computer, in modo che anche un risveglio da un backend successivo lo trovi), oppure, per una telecamera che questo computer non ha messo in modalità di sospensione ma che sembra in tale stato (con il limite impostato al valore minimo), con il limite disattivato e un valore `GevSCPD` lasciato a 0 a 80 (impostazioni di fabbrica); un limite impostato al di sopra del valore minimo viene lasciato così com’è. Viene mantenuto solo lo stato di alimentazione; ogni altra impostazione la cui scrittura è aperta viene riportata allo stato in cui è stata trovata. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Acquisire fotogrammi finché non è visibile un bersaglio di calibrazione MAPIR e adattare i coefficienti per canale dai suoi pannelli: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Le schede Ethernet di questo computer per GigE Vision (MTU, velocità di collegamento, frame jumbo) e qualsiasi `issues`; `estimate=True` aggiunge la frequenza dei fotogrammi e la larghezza di banda che le telecamere `cameras` otterrebbero. |
| `fix_network_settings()` | Applica le impostazioni dell&#x27;adattatore GigE Vision all&#x27;adattatore della telecamera di questo computeradattatore della telecamera. Modifica le impostazioni dell’adattatore e funziona solo quando Chloros è in esecuzione come Amministratore; in caso contrario, genera un messaggio di errore con il log di ciò che ha rilevato. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Esegue la correzione della vignettatura delle immagini in una cartella, ciascuna con la mappa di campo piatto del proprio obiettivo: `processed`, `lens_models` (l’obiettivo per cui ogni file è stato corretto), `skipped` (con le motivazioni). Senza `lens_model`, l’obiettivo di ogni file deriva dal campo “Modello” dei dati EXIF; un file `lens_model` si applica ai file il cui modello non specifica alcuna fotocamera, mentre un file il cui modello è in contraddizione con esso viene ignorato. Solo file. |
| `read_dls(mac, filter=None, camera_serial=None)` | Una lettura da un DLS (sensore Bluetooth di luce in discesa): spettro, irradianza per banda e intensità delle bande attraverso la risposta propria della fotocamera quando viene specificato `camera_serial`. `discover_daq_sensors(transports=["ble"])` individua il MAC. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | L&#x27;hardware dell&#x27;host per le telecamere e la configurazione consigliata (`report_text`), nonché la CPU / GPU / runtime PyTorch del sistema. |

- **Alcuni richiedono l&#x27;uso esclusivo della telecamera.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` e `update_camera_firmware()` aprono la telecamera. Mentre Chloros la detiene (connessa nell’app, con `connect_camera()`, come membro di un array, o attualmente in uso da un altro comando, come ad esempio un test dell’array o un’acquisizione che apre le proprie telecamere), il backend restituisce un errore HTTP 409 `CAMERA_HELD` (sia in `code` che in `error_code`), indicando come liberarlo. Le impostazioni del colore, le luci, `camera_power_probe()` e le letture funzionano su una telecamera collegata.
- **Colore, luci e bilanciamento del bianco sono per telecamere RGB.** Su una telecamera monocromatica a banda singola (M3M) i setter non modificano nulla e restituiscono `{"skipped": True, "reason": ...}`, proprio come fa l’CLI. Si tratta di impostazioni di visualizzazione: la vista in diretta e le acquisizioni a livello di display, mai radianza o riflettanza.

---

## Array transitori, profili di allineamento e indici

Cosa fa `chloros-cli lattice` in assenza di una connessione persistente, come le funzioni in `chloros_sdk` (modulo `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` e `index`. Ciascuno invia la richiesta generata dal comando CLI, con le impostazioni predefinite della CLI.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Aprono le telecamere autonomamente.** Ogni comando apre le proprie telecamere, esegue la propria operazione e le chiude; pertanto, necessita di telecamere che Chloros non sta gestendo (collegate nell’app, con `connect_camera()` o come elemento di un array); il backend restituisce un errore HTTP 409 `CAMERA_HELD` e indica come liberarne una. Un’apertura transitoria richiede un po’ di tempo (circa 40 s per alcune telecamere). Per acquisizioni ripetute, `connect_array()` una sola volta è il percorso più veloce e salva ogni tipo di esportazione, mentre un’acquisizione transitoria salva un’immagine debayered. `calibrate_alignment_profile()` è l’eccezione: su un array collegato di quelle telecamere, ricava i fotogrammi dall’array stesso (`used_live_array`).
- **`serials`** sono le telecamere da aprire, la prima delle quali attiva il trigger; `None` apre tutte le telecamere presenti in rete.
- **`settings`** è un dizionario di impostazioni di sovrascrittura delle telecamere, i campi impostati dai flag dell’CLI: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` e `preset` (un preset della telecamera con questo nome). Impostando un valore di esposizione o di guadagno, si disattiva il controllo automatico, proprio come avviene con i flag.
- **`gpu`**: `True` forza il warp sulla GPU, `False` sulla CPU (`--gpu` / `--no-gpu`); `None` lascia la scelta al backend.
- **`keep_settings`**/**`clear_settings`** (ogni funzione che apre le telecamere, impostazione predefinita `False`): al termine, ogni telecamera viene riportata allo stato in cui è stata trovata; `keep_settings=True` salva invece le impostazioni dell’esecuzione nella memoria della telecamera stessa (funzionamento autonomo), `clear_settings=True` ripristina quella memoria alle impostazioni di fabbrica — vedi [Memoria della telecamera](#camera-memory--keep_settings--clear_settings). La risposta (per un processo, quella di `stop()`) contiene `camera_memory`.
- **I percorsi sono quelli del backend**: quelli relativi vengono trasformati in assoluti rispetto alla directory di lavoro quando il backend si trova su questa macchina. Le funzioni relative al file del profilo di allineamento (`alignment_profile_info()`, `reorder_alignment_profile()`) leggono il file su questo computer, così come fa la CLI.

| Funzione | Cosa fa |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Registra in modo continuo da una telecamera fino all’interruzione. Restituisce un `CliJob`. I fotogrammi di un&#x27;esecuzione `tiff` costituiscono il file `raw`, un array in cui vengono scritti i dati acquisiti da quella telecamera (un piano del mosaico con i, non compressi); `png` e `jpg` contengono 8 bit, quindi quelle sessioni sono fotogrammi demosaicati destinati alla visualizzazione. Un DAQ associato o in pool scrive un file `.daq` per ogni fotogramma nellacarta `daq/` della serie (`daq_files` nelle relative statistiche). Ogni fotogramma viene scritto una volta con i metadati completi (modello, ora di acquisizione, obiettivo, bande, calibrazione su un fotogramma grezzo); `stop()` viene restituito una volta che tutti i fotogrammi già acquisiti sono stati scritti (fino a 25 s), e solo allora la fotocamera viene chiusa. Una fotocamera lasciata in modalità di attivazione per un trigger hardware (`TriggerMode` `On`) viene impostata in modalità free-run per la sessione (la risposta di avvio `trigger` indica cosa è stato rilevato). Allo stop la telecamera viene riportata allo stato in cui è stata trovata — le sue impostazioni di trigger e ogni impostazione modificata durante la sessione — a meno che non venga inviato `keep_settings=True`, che salva le impostazioni della sessione nella memoria interna della telecamera (funzionamento in modalità libera), oppure `clear_settings=True`, che resetta tale memoria. Una sessione che non registra alcun fotogramma termina automaticamente dopo 10 s con `stop_reason` `no_frames` e il motivo indicato in `last_error`, mentre il comando `stop()` restituisce tale motivo come `error`. I fotogrammi persi perché i dispositivi di scrittura non sono riusciti a stare al passo vengono conteggiati, con `drop_reason` che ne indica il motivo. Per una telecamera gestita da Chloros, utilizzare `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Gruppi di fotogrammi sincronizzati in streaming a `fps`, per gruppi `count` o fino all’interruzione. Restituisce un `CliJob` (il suo `start_reply` contiene `n_cams` e il wire `pacing`); le sue statistiche finali contano i gruppi acquisiti, scritti e scartati, i file che lo scrittore non è riuscito a scrivere e i gruppi per i quali una telecamera non ha fornito nulla. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Una acquisizione sincronizzata, un&#x27;immagine debayered per telecamera: `results` (`filepath`, `serial`, `frame_id`) e `pacing`. Viene segnalato un gruppo in cui manca una telecamera, indicando innanzitutto quanti file sono stati salvati. `lattice multi-capture` senza array collegato. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` acquisizioni sincronizzate con esposizione fissa: `sync_groups`, `master`, una riga `group_timestamps` per ogni attivazione (una telecamera che ne ha saltata una è assente dalla propria riga) e `sync`: lo scostamento di ogni altra fotocamera rispetto alla fotocamera principale, scatto per scatto (`offsets_us`: per fotocamera `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), su UN unico clock (`clock_domain`: `ptp`, oppure `device` — PTP disattivato, ogni telecamera utilizza il proprio orologio di funzionamento, quindi il suo offset corrisponde al momento dell’avvio e solo il jitter misura la sincronizzazione). Un trigger i cui timestamp combinano l’orologio del dispositivo e il PTP (PTP disattivato o ancora in fase di aggancio) viene escluso e conteggiato (`groups_mixed_domain`), così come uno in cui un orologio PTP è ancora a più di `agree_within_us` dal master (`groups_clocks_disagree`); `notes` lo confermano. `clock_offset_mean_us` / `clock_offset_jitter_us` rappresentano la media e il jitter massimi per ciascuna telecamera. `success` è falso, così come `error`, quando è stato perso un trigger, quando meno di due trigger hanno tutte le telecamere su un unico clock, o quando i clock non coincidono mai (`sync.problems`); le righe vengono restituite in entrambi i casi. Avvia il grandmaster PTP se non è in esecuzione (`time_sync` indica `grandmaster_running` e `grandmaster_started_by_this_test`), attiva il PTP di ogni telecamera (come fa `connect_array()`) e, prima del primo trigger, attende che l’orologio PTP di ogni telecamera si sincronizzi con il grandmaster e si stabilizzi: `PtpStatus` Slave su un grandmaster, entro 100 us da esso e stabile per 3 s, e ogni telecamera sincronizzata in questo modo, senza interruzioni, per 120 s — circa 2 minuti dall’accensione. Un clock che si è appena sincronizzato presenta uno scostamento fino a ~200 us per circa 2 minuti dopo aver letto “Slave”, e l’`PtpOffsetFromMaster` della telecamera stessa non lo mostra; 120 s portano le telecamere entro ~10 us da un orologio stabilizzato. Una telecamera ancora sincronizzata da un test precedente che ha lasciato il PTP attivo, secondo quanto rilevato da questo backend, conta a partire da quella sincronizzazione, quindi un test effettuato subito dopo un altro non attende nuovamente; una telecamera accesa dal test stesso, o una trovata con il PTP attivo che questo backend non ha visto sincronizzarsi (accesa altrove o prima di un riavvio del backend), attende i suoi 120 s, e in un array che include entrambe le telecamere appena sincronizzate, queste ultime attendono comunque il proprio tempo. Una telecamera che esce dallo stato Slave, supera i 100 us o segue un altro grandmaster per 3 letture consecutive (1,5 s) durante l’attesa, ricomincia il proprio conteggio; una o due di queste letture vengono registrate (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) e il conteggio prosegue, mentre un ID di grandmaster che non può essere letto viene considerato sconosciuto, non come una modifica. Il riavvio simultaneo del conteggio di tutte le telecamere (nell’arco di un secondo) è dovuto allo spostamento del grandmaster nel tempo, non a quello delle telecamere: il log del backendregistro del backend riporta `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` lo elenca in `grandmaster_steps`, e un&#x27;attesa che poi giunge al termine inizia il proprio errore con `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Il limite non viene esteso per questo caso. `ptp_settle_s` (0-600; `0` in assenza di attesa di stabilizzazione) imposta il tempo di stabilizzazione per l’esecuzione; un altro valore viene rifiutato (400 HTTP). L’attesa è limitata al tempo di stabilizzazione più 90 s (210 s per impostazione predefinita, almeno 90 s), e la chiamata attende il backend per `timeout` s: per impostazione predefinita 240 più il tempo di stabilizzazione (360 s), così come fa `test_array_sync` di `chloros-mcp`. `ptp_lock` ha `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (da quanto tempo ogni telecamera è rimasta bloccata e stabile), `found_settled` (le telecamere già bloccate per il tempo di assestamento al momento del rilevamento) e, per ogni telecamera, `status`, `offsets_ns`, `slave_s` (da quanto tempo è sincronizzata; Nessuna: non sincronizzata) e `remaining_s` (Nessuna quando il PTP di nessuna telecamera si accende). Il log del backend contiene una riga ogni 5 s con lo stato di ciascuna telecamera, da quanto tempo è sincronizzata e quanto tempo le resta. Ogni telecamera si accende con il PTP disattivato; da quel momento gli orologi si sincronizzano in circa 10 s. Successivamente, ogni telecamera viene riportata alle impostazioni rilevate durante il test — il suo PTP, l’esposizione automatica e il guadagno che il test mantiene disattivati per la durata dell’esecuzione (`ExposureAuto` / `GainAuto`, e una modalità manuale `ExposureTime` / `Gain`), mentre le altre, a meno che non si tratti di `keep_settings=True` — quindi una telecamera lasciata con il PTP attivo (`PtpEnable=True`, o un&#x27;esecuzione con `keep_settings=True`) rimane agganciata, e una volta che questo backend l’ha vista agganciata per il tempo di assestamento, un test successivo non la attende. Se gli orologi non si assestano, `success` è falso prima di qualsiasi trigger, e `error` indica cosa fare: una telecamera che non si è mai sincronizzata (`not_slave`) significa che il grandmaster non la sta raggiungendo (`time_sync_status()`); una che continuava a disconnettersi viene identificata in base a quanto tempo è rimasta sincronizzata (`remaining_s`); se i clock stanno ancora convergendo (`far`): controllare `time_sync_status()` e, per assicurarsi che siano sincronizzati prima del test, attivare il PTP e lasciarlo attivo (`PtpEnable=True` su ciascuna telecamera, oppure `keep_settings=True`; il test ripristina le impostazioni rilevate, quindi rimane attivo), quindi eseguirlo. `lattice multi-test`. |
| `detect_array_master(line=None)` | Le telecamere sulla linea di sincronizzazione e un master predefinito: `master_serial`, `slave_serials`, l’`line` rilevato. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Che vasta gamma di questi modelli di telecamere (nomi di prodotto, `"LATT-M3C-L87-FOCN"`) si collegherebbe alla rete di questo computer: `tier`, `achievable_fps_max`, il budget di cavi (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` e `fix_suggestions`. Per impostazione predefinita, i modelli delle telecamere rilevate; nessuna telecamera è aperta. Per le telecamere in tuo possesso, con le impostazioni che una connessione utilizzerebbe, consulta [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Risolvere un profilo di allineamento dai fotogrammi in tempo reale e salvarlo. `method`: `orb`, `akaze`, `phase`, `checkerboard` (con `checkerboard="9x6"`, gli angoli interni) oppure `manual` (con `control_points={serial: [[x, y], ...]}`, anche quelli della telecamera di riferimento). `model`: `translation`, `rigid`, `affine` o `homography`. Le opzioni più dettagliate sono i flag dell’CLI: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` è uguale a `{serial: {option: value}}`. `rms_threshold_px` rifiuta di salvare una soluzione peggiore. Restituisce `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (per ogni telecamera `reprojection_error`, `num_matches`) e `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Un’immagine multibanda allineata attraverso il profilo: `files` (il file TIFF multibanda con il suo `bands` e le relative dimensioni, più una per banda con `per_band` e una per fotocamera con `per_camera`) e `vignette_corrected`. `bands` seleziona e `order` ordina le bande in base al nome (un elenco o una stringa separata da virgole). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Immagini multibanda allineate in stream (stesse opzioni di warp di `capture_aligned_multiband()`), per `count` o fino all’interruzione. Restituisce un `CliJob`; le sue statistiche finali contano `frames_written` e qualsiasi `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Un file di profilo riassunto come `align-info` lo stampa: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (per telecamera `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (per telecamera `band_names`, `wavelengths_nm`, `enabled`); `profile` è il file completo. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Modifica il file di profilo in loco: `order` contiene i nomi delle bande (ognuna sposta in alto il livello della propria telecamera, in quell’ordine), `enable` / `disable` sono i numeri di serie. Restituisce il nuovo `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | I preset dell’indice (`presets`: ciascuno `name`, `channels`, `formula`) e colore `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Un indice su un file TIFF multibanda allineato: un `preset` (`NDVI`, `NDRE`, ...) oppure un `formula` sui simboli `channel_map` mappa alle bande (`{"NIR": "NIR_850", "Red": 0}`, per nome o per indice). Un&#x27;impostazione predefinita&#x27;simbolo di un preset `channel_map` omette la singola banda del file il cui nome inizia con esso (`red` -&gt; `Red_660`, `nir` → `NIR_850_RGN`), denominata dal file stesso (il suo elenco di bande, oppure l’`Chloros:Band{R,G,B}Name` di un’acquisizione o di un prodotto Chloros), quindi l’NDVIsu una cattura di riflettanza Chloros non necessita di `channel_map`; due bande corrispondenti a un unico simbolo vengono rifiutate, non ipotizzate. `capture_level` indica il contenuto delle bande: `reflectance` o `radiance` rendono i valori quantitativi; uno stack non calibrato (quello di `align-apply`) fornisce valori qualitativi; `"unknown"` lo legge dal file, compreso l’`Chloros:ProcessingLevel` di un file Chloros. Restituisce `stats` (forma, intervallo dal 1° al 99° percentile, intervallo LUT, `capture_level` utilizzato) e il valore `files` scritto accanto all’input (`<input stem>_<preset>.tif`, `..._LUT.png`) o in `output`, un percorso: se hanno un&#x27;estensione immagine, i file vengono denominati a partire dal loro nome base (`output="D:/out/ndvi_FRGN.png"` scrive `ndvi_FRGN.tif` e `ndvi_FRGN_LUT.png`), senza di essa si tratta di un prefisso (`D:/out/ndvi` -&gt; `ndvi.tif`, `ndvi_LUT.png`); leggere i percorsi a partire da `files`. Questi contengono i dati della fotocamera di origine (marca / modello / numero di serie, GPS, obiettivo, ora di acquisizione in UTC, l’identificativo di calibrazione ma mai la calibrazione raw-DN), `Chloros:ProcessingLevel=index` e l’indice come unica banda (`XMP-Camera:BandName`, sia il file TIFF che il PNG LUT). Solo file; nessuna telecamera. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Acquisizione, calibrazione (`reflectance` o `radiance`; `raw` salta questo passaggio), allineare e calcolare un indice in un unico passaggio; `save_multiband=True` conserva la pila calibrata, che `compute_index()` potrà utilizzare in seguito. Restituisce `stats` (insieme a `serials`, le cui bande sono state utilizzate) e `files`. Apre `serials` (il primo fa scattare il trigger), altrimenti le telecamere del profilo la cui voce `layer_order` è abilitata, la di riferimento per prima (aperta anche se le sue bande sono disabilitate); una telecamera le cui bande sono disabilitate non viene mai calibrata. `channel_map` è necessario solo per un simbolo a cui lo stack non può rispondere, come in `compute_index()`: il simbolo di un preset prende l’unica banda abilitata il cui nome inizia con esso, verificata prima che qualsiasi telecamera si apra (se nessuna o più rispondono, la richiesta viene rifiutata, indicandone i nomi); un `formula` personalizzato richiede ogni simbolo. `index.tif` e la LUT contengono la telecamera dello stack, l’ora di acquisizione (l’esposizione della telecamera di riferimento, UTC) e l’obiettivo, ed entrambi indicano l’indice come la loro unica banda (`XMP-Camera:BandName`, ad es. `NDVI`: il file PNG a colori indica l’indice, non i suoi canali di colore). `lattice index --live`. |

### `CliJob`

L&#x27;handle restituito dai tre flussi. Il processo viene eseguito sul backend, che mantiene il controllo delle proprie telecamere fino a quando non viene interrotto.

| Membro | Descrizione |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (fotogrammi salvati / scritti, scartati, `fps`, `elapsed`, errori di scrittura, `stop_reason`; in un processo continuo anche `frames_grabbed` e `frames_queued` — ogni fotogramma acquisito viene scritto, messo in coda, scartato o non viene registrato — e `drop_reason`, `writers` e `write_ms`, il tempo medio impiegato per la scrittura di un fotogramma) e `output_dir`. `state` è `running`, `ended` (si è arrestato autonomamente e non è stato ancora arrestato), `stopping` (un processo in fase di arresto sta scrivendo la propria coda: statistiche in tempo reale) e `stopped_by`) oppure `finished` (le sue statistiche finali, l’arresto di `error` e `stopped_by`). Il record di un lavoro interrotto viene conservato per 10 minuti (al massimo gli ultimi 32), per ogni client: un altro script o `chloros-mcp` legge lo stesso stato e le statistiche finali. Un lavoro `chloros-cli` avviato (`lattice continuous`, `multi-stream`, `align-stream`) viene interrotto dal backend quando la CLI smette di effettuare il check-in per 30 s — terminato, la sua macchina in modalità sleep — con `stop_reason`, `client_gone` e `stopped_by` sul backend; la lettura delle sue statistiche da qui non lo mantiene attivo. Un processo avviato dall’SDK viene eseguito fino a quando non viene interrotto. Ogni avvio invia il proprio `timeout` come scadenza del backend (`client_deadline_s`): una telecamera che si apre dopo di esso (una ancora in fase di avvio) viene rimessa in posizione e chiusa e non viene avviato alcun processo, e la chiamata genera `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Una telecamera occupata da un processo viene rifiutata a ogni altra apertura (`connect_camera()`, un altro avvio, una cattura transitoria) con codice HTTP 409 `CAMERA_HELD`, indicando il processo e chi lo ha avviato. |
| `stop(timeout=30.0)` | Interrompilo e chiudi le sue telecamere: `final_stats`, `output_dir` e `error` quando l’arresto non è andato a buon fine. Una seconda chiamata restituisce la stessa risposta, così come un comando di arresto da un altro client: se ne invia uno mentre il primo sta ancora scrivendo la propria coda, lo si attende (fino a 25 s; superato questo limite, viene generato un errore e le statistiche finali seguono su `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Attendere che termini autonomamente (raggiungimento di `count`, disco pieno, errore), quindi `stop()` per raccogliere i conteggi finali. `timeout` genera un&#x27;eccezione `TimeoutError` mentre è ancora in esecuzione; un processo senza `count` termina solo quando viene interrotto. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Il suo ID e il tipo (`continuous`, `multi-stream`, `align-stream`), dove scrive, la risposta del percorso iniziale e se `stop()` è stato eseguito. |

Utilizzarlo come gestore di contesto per interromperlo all’uscita dall’ambito.

---

## Cartella del progetto e ricette

**La cartella del progetto** è la cartella in cui vengono creati i nuovi progetti, condivisa dall’app Chloros, dalla CLI e dall’SDK (modulo `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Se una cartella configurata non esiste più, si ricorre a quella predefinita (`Documents/MAPIR/Chloros Projects`), così come fa l’app. Un percorso relativo specificato per `set_project_folder()` viene ricavato dalla directory di lavoro quando il backend si trova su questa macchina.

**Una ricetta** è l’elenco di azioni in formato YAML/JSON che `chloros-cli project run PATH RECIPE` esegue su un progetto salvato. `run_project_recipe()` la esegue con il runner della CLI(installato con l’SDK), quindi una ricetta si comporta allo stesso modo in entrambi i casi:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** è un file `.yaml` / `.yml` / `.json` oppure la ricetta stessa: un dizionario contenente `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) e, facoltativamente, `overrides` e `stop_on_error` (impostazione predefinita true: l&#x27;esecuzione si interrompe alla prima azione che fallisce). Le azioni sono quelle descritte nella [Recipe DSL](cli-reference.md#recipe-dsl) della documentazione di riferimento della CLI; un `output` relativo viene risolto in base alla directory di lavoro dello script.
- **Restituisce** `ok` (tutte le azioni sono andate a buon fine), `actions` (quante), `log` (l&#x27;output del runner; anche `echo=True` lo stampa durante l&#x27;esecuzione) e `validation`. Un file di ricetta mancante o non analizzabile genera l&#x27;errore `ChlorosConnectError`.
- **Verificato prima dell’esecuzione.**`validation` è `{"ok", "errors", "warnings", "steps"}`: ogni azione deve avere come chiave esattamente un verbo noto (`{"capture": {"name": ...}}`, non `{"action": "capture"}` o `{"type": "capture"}`) che identifichi una telecamera, un array o un sensore presente nel progetto, del tipo corretto. Una ricetta con un errore**non** esegue alcuna azione (`ok` false, gli errori in `log`), indipendentemente da ciò che dice `stop_on_error`. Le chiavi inutilizzate, e ogni array o sensore di luce controllato dalla ricetta che non sia attualmente collegato, sono `warnings`. Una ricetta non collega i dispositivi: collegare il primo del progetto (`ChlorosProject.connect_all()`, `chloros-cli project connect`, oppure lo strumento MCP `connect_project_devices`). `dry_run=True` non esegue nulla e restituisce `{"recipe": ..., "validation": ...}`.
- In un processo viene eseguita una sola ricetta alla volta. Una ricetta YAML richiede PyYAML.

---

## Automazione del progetto — `ChlorosProject`

Un progetto Chloros salvato è una cartella contenente `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` carica il manifesto (`relink`: la cartella in cui si trovano ora i file sorgente mancanti di un progetto spostato, utilizzata quando il backend lo carica — `process()` — vedi **Un progetto spostato** sotto `ChlorosLocal`), mentre `connect_all` attiva tutte le telecamere, gli array e i sensori di luce salvati con le relative impostazioni — lo stesso stato hardware che si otterrebbe tramite l’interfaccia grafica. I sensori di luce si connettono tramite il [pool di sensori DAQ](#daq-sensor-sessions), pertanto richiedono un backend in esecuzione (vedere [`SensorHandle`](#sensorhandle)).

### Esempio minimale

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Oppure come gestore di contesto:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metodi `ChlorosProject`

| Metodo | Descrizione |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Individua e connette ogni telecamera, array e sensore di luce salvati. Restituisce un rapporto di connessione per ciascuna classe (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, ciascuno con `connected`, `missing` e `errors`; `arrays` include anche `alignment`). Con un backend in ascolto su `127.0.0.1:5000`, le telecamere e gli array salvati vengono individuati e collegati tramite esso. In assenza di un backend, questo processo li individua e li gestisce autonomamente tramite `lattice_sdk`, che richiede il runtime dell’SDK di Arena e OpenCV (`cv2`) — il pacchetto wheel dell’SDK non installa nessuno dei due; quando ciò non è possibile, ogni telecamera e array salvati vengono elencati sotto `errors` con l’indicazione del motivo, mentre `missing` rimane vuoto. Un backend in esecuzione ma che rifiuta l’SDK (disconnesso: HTTP 401; un piano al di sotto di Copper: 403) non è considerato “assenza di backend”: ogni telecamera, array e sensore di luce salvati vengono elencati sotto `errors` con il motivo fornito dal backend, e nulla viene aperto in questo processo. Ogni chiamata verifica nuovamente il backend, quindi `connect_all()`, dopo l’accesso o dopo l’avvio di Chloros, si connette allo stesso oggetto di progetto. Un array salvato già attivo nel backend con le stesse telecamere (quelle dell’interfaccia grafica o di `chloros-cli project connect`) viene collegato così com’è, senza riapplicarne le impostazioni salvate, e `disconnect_all()` lo lascia in esecuzione. I sensori di luce passano sempre attraverso il backend (vedi [`SensorHandle`](#sensorhandle)). Non avvia mai un backend. |
| `disconnect_all()` | Smantella tutto. Ogni dispositivo viene prima rilasciato; poi, se il backend non ha confermato il rilascio di una telecamera, di un array o di un sensore di luce (il tempo di smantellamento è scaduto o è stato rifiutato), `ChlorosConnectError` assegna un nome a ciascuno di essi. Un dispositivo già rimosso, o per cui non è in esecuzione alcun backend, rimane silenzioso. Un array o un sensore di luce che un altro client aveva aperto viene lasciato in esecuzione. L’uscita del progetto `with` non viene mai generata. |
| `capture_all(output_dir=".")` | Un fotogramma da ogni telecamera e array connessi, e uno spettro (`SensorHandle.read()`) da ogni sensore di luce connesso, acquisiti in sequenza (telecamere, poi array, poi sensori), non in un unico istante. |
| `stream(camera, overlays=False, fps=10.0)` | Generatore che produce fotogrammi BGR `numpy` da una telecamera (o array) specificata. `overlays=False` legge i fotogrammi con l&#x27;handle`grab()` a livello di `debayered`: attraverso il backend quando il progetto è connesso tramite uno di essi, altrimenti tramite un ciclo di acquisizione diretto `lattice_sdk` (gli array generano dizionari `{serial: frame}`). `overlays=True` instrada attraverso `ChlorosLocal.camera_stream()` → il feed MJPEG `/api/camera/<serial>/stream-annotated` del backend, con il blocco salvato dalla videocamera `ui.overlay` trasmesso come parametri di query. Richiede la modalità backend, OpenCV (`cv2`) e una **videocamera autonoma**: una telecamera in modalità diretta genera l&#x27;errore `RuntimeError` (il backend non può acquisire una telecamera di proprietà di questo processo) e un array genera l&#x27;errore `NotImplementedError` (sovrappone il composito per telecamera — trasmette un membro in base al nome). Equivalente one-shot: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Esegue l’allineamento su ogni array attualmente connesso. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Esegue la pipeline di calibrazione/indicizzazione sulle immagini del progetto (racchiude `ChlorosLocal.process`; questi quattro sono gli **unici** kwarg accettati — `indices=` ecc. genera un&#x27;eccezione `TypeError`; imposta gli indici tramite `ChlorosLocal.configure()`). Costruisce in modo differito un `ChlorosLocal()`, che avvia automaticamente un backend. |

Attributi:
- `proj.cameras` — `Dict[str, CameraHandle]` con chiave composta da nome E numero di serie.
- `proj.arrays` — `Dict[str, ArrayHandle]` con chiave composta da nome E array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` con chiave composta da nome E slot_id.
- `proj.config` — Dizionario `project.json["config"]`.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Livelli di elaborazione.** `capture()`, `grab()` e `frame_stream()` utilizzano tutti lo stesso token `processing`
, e la catena è cumulativa: ogni livello esegue tutto ciò che si trova al di sopra di esso:

| Livello | Uscita | Note |
| --- | --- | --- |
| `raw` | Bayer a 1 canale, nativo del sensore | Nessun demosaic. A questo livello non sono disponibili sovrapposizioni. |
| `debayered` | Colore a 3 canali (**impostazione predefinita** per `CameraHandle` e ogni `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` e `stream()` hanno come impostazione predefinita `raw`): BGR in memoria (`grab()` / `frame_stream()`), salvato come RGB da `capture()` | In modalità backend, come acquisizione: l’offset del dark (DSNU) del pacchetto di calibrazione applicato al raw, seguito dal demosaic sensibile ai bordi di OpenCV — senza LUT di linearità, flat field o unmix. L’unico livello che funziona senza la modalità backend è il semplice demosaic bilineare della libreria della fotocamera (senza offset scuro). Il file TIFF salvato è in formato RGB fotometrico, quindi inverti i canali di un fotogramma in memoria (`frame[..., ::-1]`) per confrontarlo con un file. |
| `radiance` | float32, W/m²/sr/nm, **un piano per banda** | Catena radiometrica completa: demosaic + separazione 3×3 (multispec) + DSNU + flat-field + scala NIST, con esposizione × guadagno eliminati in modo che i valori siano assoluti. |
| `reflectance` | uint16, 32768 = 1,0 (oppure uint8, 255 = 1,0 — vedi `bit_depth`), **un piano per banda** | Radianza divisa per l’irraggiamento discendente (ρ = π·L/E). Richiede una lettura DLS/DAQ — vedi la nota qui sotto. |
| `display` | 8 bit, simile a sRGB | Rendering equivalente a quello della GUI: CCM + bilanciamento del bianco + gamma tramite il profilo colore attivo della fotocamera. |

In modalità diretta (senza backend) è disponibile solo `debayered`: `grab()` a un altro livello ignora
`NotImplementedError`, `capture()` ignora `processing`, mentre `frame_stream()` genera
`RuntimeError` a ogni livello. `reflectance` necessita di una lettura downwelling utilizzabile — l’endpoint del frame inserisce
automaticamente il DAQ raggruppato nello slot DLS della fotocamera, ma senza un DAQ associato la catena rifiuta l’
uscita di riflettanza e contrassegna chiaramente la retrocessione nei metadati restituiti, anziché
restituendo un prodotto di qualità inferiore.

> **Scala DN della riflettanza — non codificarla in modo rigido.** La riflettanza LATTICE utilizza `32768` = ρ 1,0 e registra
> XMP `Chloros:PixelScale=32768`; la riflettanza Survey3 utilizza `65535` = ρ 1,0 e non contiene alcun
> tag `Chloros:*`. Leggi il tag e dividi per esso. È definito nel dominio uint16, quindi rimane
> `32768` per ogni formato che effettua il ridimensionamento (TIFF a 16 bit, TIFF a 8 bit, PNG/JPG a 8 bit, percentuale a 32 bit)
> — normalizza prima il tipo di dati memorizzato riportandolo a uint16 (×257 da 8 bit, ×65535 da float). L’unica
> eccezione è un file proveniente da una build precedente alla 1.3.0: all’epoca un TIFF a 8 bit veniva *tagliato*, non ridimensionato, quindi
> non è descritto da alcuna scala e non contiene alcuna tupla `PixelScale` o MicaSense. Considerare un tag mancante su un
> file di riflettanza LATTICE come “scala non valida”, non come impostazione predefinita.

**Tag di calibrazione: solo su `raw`.** Il file `raw` di un’acquisizione contiene il pacchetto di calibrazione della fotocamera: il payload `CalibrationJSON` e i tag compatibili con MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), che descrivono i valori grezzi del sensore. Ogni altro file di livello (`debayered`, `display`, `radiance`, `reflectance`, il rendering `index/`) e ogni fotogramma di registrazione continua o di flusso multicamera riporta l’identificativo della telecamera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) e i dati di registrazione propri del fotogramma, `XMP-mapir:FrameJSON` (esposizione, guadagno, temperatura, formato pixel, ora di acquisizione, corrispondenza DAQ), poiché ogni prodotto `process()` porta con sé l’identità. L’elaborazione individua la calibrazione di tale file tramite il suo numero di serie e `CalBundleSha256` nella cache di calibrazione di Chloros, scaricando il pacchetto una sola volta se questo computer ne è sprovvisto.

#### File creati da più telecamere

Un file creato dalle bande di più telecamere — uno stack multibanda (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), un indice in tempo reale creato da più telecamere (`index.tif`, `index_LUT.png`), un&#x27;immagine `composite/` di un&#x27;acquisizione in array — indica il nome di ogni telecamera che ha contribuito alla sua creazione:

- **Per-elenchi per telecamera, in ordine di banda** (il composito: livello di sfondo, poi primo piano; un livello indice, le telecamere da cui provengono le rispettive bande): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (il pacchetto di calibrazione attraverso cui sono passate le bande di ciascuna telecamera, altrimenti — uno stack `capture_aligned_multiband()` / `start_aligned_stream()`, le cui bande non passano attraverso nessuno — il pacchetto della telecamera che Chloros ha memorizzato nella cache, quello che dà il nome ai suoi file di acquisizione), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — elenchi paralleli, in cui l’elemento *n* di ciascuno descrive la stessa telecamera; `-` dove una telecamera non ha alcun valore (nessun pacchetto di calibrazione, un obiettivo sconosciuto). Ogni telecamera è **un** elemento: se il suo pacchetto presenta più valori per un campo — i certificati delle lampade alogene di un pacchetto — sono uniti con `;` (`F-1479-1;F-1480-1;F-1481-1`), mai con una virgola, quindi un elenco presenta una voce per ogni fotocamera indipendentemente da come viene stampato.
- **EXIF `SerialNumber`** contiene gli stessi numeri di serie, separati da virgola (`214000535,214000551`).
- **Non** contiene l’identificativo di una singola telecamera (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): questi identificano una telecamera e il relativo pacchetto, e l’elaborazione determinerebbe la calibrazione del file sulla base di essi. Un indice calcolato dalle bande di diverse telecamere (`index.tif`, `index_LUT.png`) presenta i dati EXIF `Model` `LATTICE` (il rig), non il modello `LATT-…` di una singola fotocamera.

Un file contenente le bande di **una sola** telecamera — un TIFF per banda, un fotogramma `capture_aligned_multiband(per_camera=True)`, uno stack o un indice live di una singola telecamera — riporta i tag di identità propri di quella telecamera, proprio come ogni altro file di acquisizione. Lo stack di `capture_aligned_multiband()`, i file per banda e per telecamera, nonché ogni stack di `start_aligned_stream()` e i relativi file per banda, riportano l’ora di acquisizione del proprio gruppo (l’esposizione della telecamera di riferimento, UTC), mentre un fotogramma `per_camera` di una telecamera mono è un unico piano, in `bit_depth` come lo stack.**Orientamento della telecamera (capovolgimento / rotazione).** I fotogrammi provenienti da `grab()` / `frame_stream()` sono sempre nell’
orientamento nativo della fotocamera, a ogni livello. Le acquisizioni salvate seguono l’impostazione di orientamento
della fotocamera (le opzioni Mirror H / Mirror V / rotazione dell’interfaccia grafica):

- **I file `raw` e `debayered` riproducono esattamente ciò che il sensore ha rilevato**, indipendentemente dall’impostazione: un mosaico Bayer
  ruotato o capovolto costituirebbe uno schema di filtri colore diverso, e le mappe di calibrazione si basano
  sulla griglia del sensore stesso.
- **I file `display`, `radiance`, `reflectance` e i file indice sono orientati** — ruotati una volta, dopo tutte le
  calcoli di calibrazione. Le loro maschere di clamp sono sulla stessa griglia.
- **Ogni file di acquisizione registra l’impostazione** in XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, in senso orario, dopo
  i capovolgimenti) e `ChlorosOrientationApplied` — `false` sui file grezzi e debayered, `true` sui
  prodotti orientati. `ChlorosProcessing` elenca le operazioni applicate ai pixel di quel file (`none` quando
  è nativo). Ciò include ogni fotogramma di una registrazione continua (l’impostazione predefinita della fotocamera,
  non applicata) e di uno streaming multicamera (l’identità: i suoi fotogrammi componenti sono nativi, come quelli di un’
  acquisizione array).
- **L’elaborazione di un progetto applica l’orientamento registrato** ai prodotti che genera da un file raw
  (prima la calibrazione nelle coordinate del sensore, poi la distorsione di allineamento dell’array, infine l’orientamento),
  in modo che corrispondano ai prodotti dell’acquisizione in tempo reale — compresi i prodotti di riserva con risposta del sensore non calibrata e correzione della vignettatura;
  il suo prodotto debayered rimane nativo.
- **Il visualizzatore di immagini Chloros mostra un file raw o debayered ruotato in base al suo record** — solo a scopo di visualizzazione,
  il file non viene modificato; un file senza il record viene mostrato così come è stato salvato.
- **Le acquisizioni senza il record vengono elaborate come native.** Prima della versione 1.3.0, un’acquisizione con una singola fotocamera
  convertiva autonomamente i propri file raw e debayered; Chloros non è in grado di distinguere un file raw di questo tipo da uno nativo e
  non effettua alcuna ipotesi.
- **Le acquisizioni in array mantengono ogni file membro in formato nativo per impostazione predefinita** e registrano l’identità;
  `ArraySession.capture(orient=True)` converte i propri prodotti (vedere i controlli di acquisizione dell’array).

#### Uscita per banda e array misti

Ai livelli `radiance` e `reflectance` una fotocamera restituisce **un piano per ogni banda di quella
fotocamera** — tre per un M3C (le sue tre bande non miste, ordine dei canali BGR), uno per un’M3M (il suo
unico filtro a banda stretta). I piani replicati della telecamera mono vengono compressi, quindi un **array misto
M3C/M3M produce forme di array diverse per ogni serie**. Leggere la chiave `bands` dai
metadati del fotogramma per capire a cosa corrisponde ciascun piano; non dedurlo dal numero di canali:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Controllare sempre `hdr["processing_level"] == "reflectance"` prima di utilizzare i valori. Un declassamento
(nessuna lettura DAQ, nessun pacchetto di calibrazione, layout di banda sconosciuto) viene segnalato in `processing_level` e
`radiometric_refused` anziché come un aumento di livello — l’array che si ottiene è comunque un frame valido, semplicemente
non è il prodotto richiesto. Una cattura salvata che è stata declassata riporta lo stesso codice nel proprio XMP
di `Chloros:RadiometricRefused` (ad es. `no-band-layout`, `no-dls-snapshot`).

#### Un trigger per ogni lettura — `group_id`

**I membri dell’array vengono serviti dal gruppo di fotogrammi sincronizzati dell’array, non dal proprio
buffer di acquisizione di ciascuna telecamera.** Le telecamere di un array sincronizzato via hardware espongono i dati insieme, ma le N letture per telecamera
non *campionano* insieme — la lettura dell’ultimo fotogramma bufferizzato di ciascuna telecamera si estende oltre i
e un singolo fotogramma perso desincronizza l’insieme senza che nessuno se ne accorga.

L’erogazione dal gruppo è necessaria ma non sufficiente di per sé: ogni richiesta continua a
cattura *l’ultimo* gruppo nel proprio istante, e il ciclo di acquisizione può pubblicare nel frattempo.
Misurato su un array live a 4 telecamere, **12 delle 20 letture parallele si sono estese su due gruppi** — sempre
esattamente adiacenti. Quindi la lettura viene **fissata**:

1. Ogni fotogramma contiene `group_id`, il contatore di pubblicazione monotono dell’array.
2. `grab()` legge la telecamera principale (o, se questa non è disponibile, la telecamera successiva disponibile),
   quindi fissa le telecamere rimanenti all’ID del gruppo su cui è atterrato (`?group_id=N` sull’endpoint HTTP
  ).
3. Il backend conserva i gruppi recenti per **almeno mezzo secondo**, quindi un gruppo affine già
   superato da pubblicazioni successive viene comunque servito da quel trigger — un tempo sufficiente per il round trip dell’esportazione della
   , anche su un array a sensore pieno di 9 telecamere. Un&#x27;associazione che il backend non
   mantiene più viene rifiutata con `409` e `group_aged_out: true` anziché ricevere una risposta silenziosa
   con un trigger diverso — `grab()` rilegge l’insieme.

Rimisurato con il pinning: **20 letture su 20 hanno restituito un unico trigger.** `grab()` lo fa per te;
hai bisogno di `group_id=` solo quando accedi direttamente all’endpoint HTTP.

- Una telecamera che non può essere servita nel gruppo fissato — perché non presente in esso o perché rifiuta la richiesta
  (un `band` che non possiede, un livello che non può produrre) — viene **omessa anziché essere inserita a posteriori**
  da un trigger diverso, mentre le altre telecamere vengono comunque restituite.
- Il dizionario restituito da `grab()` è un `GrabResult` (una semplice sottoclasse di `dict`;
  `from chloros_sdk import GrabResult`): `.errors` associa ogni
  seriale omesso all’eccezione generata dalla sua lettura, mentre `.group_id` rappresenta il gruppo condiviso. Un
  `.errors` vuoto indica che tutte le telecamere fanno parte di questo gruppo.
- Una telecamera autonoma non appartiene a nessun gruppo, segnala `group_id: None` e **rifiuta** un `group_id=`
  pin anziché ignorarlo.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Lo stesso contatore viene pubblicato su **ogni flusso MJPEG per singola telecamera** di un membro dell’array — l’
anteprima (`/api/camera/stream/<serial>`), l’anteprima annotata (`/stream-annotated`) e il
flusso di riflettanza (`/stream-reflectance`) — come `X-Chloros-Group-Id` su ciascuna parte, oltre a
`X-Chloros-Frame-Id` e `X-Chloros-Timestamp-Ns`. Gli stream sono continui anziché
puntuali, quindi vengono abbinati raccogliendo le parti e raggruppandole in base a quell’intestazione — non c’è alcun
pin da impostare. Una telecamera autonoma non ha alcun gruppo. Vedi
[Flussi MJPEG per singola telecamera](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` non rimane mai inattivo.** Un tick che fallisce per un motivo risolvibile autonomamente
viene saltato: uno stream da telecamera non restituisce nulla per esso, uno stream da array restituisce le telecamere che hanno
risposto (possibilmente nessuna). Tali motivi sono: nessun fotogramma ancora disponibile; il thread di acquisizione della telecamera si è messo in pausa per un
secondo o due (`404 not streaming`, a cui il backend risponde ogni volta che un’altra telecamera si connette
o si disconnette, oppure quando una modifica delle impostazioni riequilibra il pool); un riavvio del backend o `5xx`; l’allineamento
è ancora in fase di stabilizzazione (`409 … retry shortly`); e un rifiuto di riflettanza perché il DAQ non ha ancora una lettura
(`no-dls-snapshot`, una lettura obsoleta o vuota). Un rifiuto al tick successivo non farebbe altro che ripetersi — un
`band` sconosciuto, `bit_depth=8` sulla radianza, un pacchetto di calibrazione senza scala di radianza, un rifiuto di riflettanza
con **nessun DAQ** (nessuno associato alla fotocamera o al suo array, nessuno collegato al
backend: l’`dls_source` del rifiuto è `"none"`), qualsiasi altro `4xx` —
genera `chloros_sdk.FrameFetchError` (un `RuntimeError`, che trasporta `.status` e `.body`) in
una sola volta. Lo stesso vale per **60 secondi** di tick saltati senza alcun frame in mezzo, qualunque sia l’`fps` — abbastanza
lungo da superare la stabilizzazione dell’allineamento e un riavvio del backend, abbastanza breve da far sì che una telecamera che sia
è effettivamente fuori uso interrompa lo streaming. Anche quell’errore di rinuncia è un `FrameFetchError`, sia per una singola telecamera che per
un array, con l’eccezione dell’ultima lettura su `.errors` (e, per una telecamera, come `__cause__`). Per
un array, “rifiutato” significa che ogni telecamera ha rifiutato; l’eccezione `.errors` contiene il
motivo di ciascuna telecamera.

**Un array `frame_stream()` genera ogni trigger una sola volta** (così come `ArrayHandle` e `ArraySession`). Un
tick il cui set più recente è il trigger che ha già restituito attende il trigger successivo invece di
restituire nuovamente lo stesso set, quindi lo stream scorre alla velocità minore tra `fps` e la frequenza dei frame dell’array stesso
, e gli insiemi consecutivi non condividono mai un `group_id`. Un array che smette di pubblicare genera
`FrameFetchError` («nessun nuovo trigger per 60 s») anziché ripetere il proprio ultimo set. Senza questa misura, uno
stream che richiedesse più di quanto l’array pubblicasse — o che iniziasse subito dopo la connessione — distribuiva
ripetizioni: in un&#x27;esecuzione misurata, 20 set contenevano 11 ID di gruppo distinti.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Esportazione di una singola banda — `band`

`grab()` / `frame_stream()` prendono `band="<name>"` per esportare solo una delle bande di una telecamera come
array `(H, W, 1)` a canale singolo — la stessa forma prodotta da una telecamera monocromatica, quindi l’uscita a banda singola viene
gestita in modo identico sia che provenga da un M3M sia che derivi dalla suddivisione di un M3C. La banda rossa di un FRGN:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Le convenzioni di denominazione accettate sono il nome completo del layout (`Red_660`, `NIR_850`, `Green_550`) o la sua forma abbreviata
(`red`, `nir`, `green`), indipendentemente dalle maiuscole (`red`, `nir`, `green`), indipendentemente dalle maiuscole e minuscole. Un nome sconosciuto viene rifiutato con l’elenco dei
nomi disponibili.

> **La selezione avviene in base al nome, non al numero di canale — deliberatamente.** «Canale 1» è `Green_550` nell’
> ordine dei canali BGR dell’array, ma *rosso* nell’ordine delle lettere del codice del filtro(`RGN`), e i due
> non coincidono. Scegliere quello sbagliato produce un frame perfettamente plausibile della banda sbagliata, che
> nulla a valle è in grado di rilevare. Per questo motivo non esiste un indice numerico.

#### Allineamento dei flussi — `align`

`align=True` trasforma il fotogramma nel fotogramma di riferimento comune del proprio array. Ogni telecamera viene trasformata
attraverso la **stessa** risoluzione del profilo e la stessa area di interesse (ROI) di ritaglio, quindi i flussi per singola telecamera aperti indipendentemente
risultano **reciprocamente allineati** — non è necessario richiederli insieme:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Requisiti e comportamento:

- La telecamera deve trovarsi in un array **sincronizzato a livello hardware** (cavo di sincronizzazione M8) dotato di un profilo di allineamento
  — la co-registrazione presuppone che i membri siano esposti simultaneamente, cosa che un array in
  array in modalità free-run non garantisce. In caso contrario, viene rifiutato con codice `409`.
- **Rifiutato, mai disallineato in modo silenzioso.** Il percorso di acquisizione potrebbe salvare i dati disallineati con un avviso
  poiché un file conserva i propri tag di provenienza e può essere riallineato in un secondo momento; uno streaming live non ha
  questa seconda possibilità, quindi la mancanza di un profilo costituisce un errore piuttosto che una soluzione di ripiego.
- **La richiesta di allineamento ne avvia l’esecuzione.** Richiedere `align` su un array senza profilo avvia la
  calibrazione e restituisce `409` con ciò che sta aspettando (ad es. *&quot;in attesa che l’esposizione si
  stabilizzi su 214000551 (1/3)&quot;*) — riprovando l’operazione va a buon fine, in genere entro poche decine di secondi.
  Ciò è importante in modalità headless: l’allineamento automatico al momento della connessione si interrompe dopo 30 s, e ogni tentativo
  effettuato in quel lasso di tempo viene saltato mentre lo smart-AE è ancora in fase di stabilizzazione; di conseguenza, un array la cui esposizione
  converge lentamente finiva per non avere alcun profilo per l’intera sessione e non c’era modo di ottenerne
  uno senza aprire la vista combinata sul desktop. `alignment_status` riporta inoltre
  `ae_align_holdouts`, indicando le telecamere ancora in fase di stabilizzazione e il loro stato di avanzamento.
- Assegnare a ogni flusso di un array lo **stesso `max_h`**, altrimenti i loro ritagli finiranno su griglie di pixel
  diverse.
- L’allineamento viene applicato *prima* della codifica della profondità di bit, quindi la deformazione interpola valori a piena precisione
  .

#### `bit_depth` — compromesso tra precisione e larghezza di banda

`grab()` e `frame_stream()` accettano `bit_depth=16` (impostazione predefinita) o `bit_depth=8`:

| `bit_depth` | Codifica della riflettanza | Costo |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, margine fino a ρ 2,0 | Precisione completa. |
| `8` | uint8 che copre `0..refl_max` (ρ predefinito 1,0) | Metà dei byte; quantizzazione di ~0,004 ρ al limite massimo predefinito. I valori superiori al limite massimo vengono troncati — vedi sotto. |

##### Il limite massimo a 8 bit — `refl_max`

`refl_max` imposta il valore di ρ che l’intervallo a 8 bit copre, quindi con `pixel_scale = 255 / refl_max` e ρ rimane
`pixel / pixel_scale`. `grab()` e `frame_stream()` lo considerano come parola chiave sia per la telecamera che per
(`?refl_max=` sull’endpoint HTTP). Aumentalo per ottenere margine per i riflessi speculari,
con incrementi proporzionalmente più grossolani:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | passo |
| --- | --- | --- |
| `1.0` (predefinito) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **L&#x27;esposizione automatica non può correggere un clipping a 8 bit, e tentare di farlo peggiora la qualità dei dati.**
> ρ = π·L/E, una volta eliminati l’esposizione e il guadagno; pertanto, ρ è una proprietà della scena, non dell’
> impostazione della fotocamera — misurato su un array in tempo reale, dimezzare l’esposizione ha spostato il valore medio di ρ dell’1,3%. Aumentare
> l’esposizione per “adattarla” alla gamma porta invece i pixel alla saturazione del **sensore**, il che
> sottostima L e distorce ρ **verso il basso** senza segnalare alcun errore (misurato: ρ massimo 1,203 → 0,985 con
> esposizione raddoppiata, media −23%). Scegliere la gamma con `refl_max`; mantenere il sensore fuori dalla
> saturazione è già compito dell’AE, e ciò protegge la *validità* di ρ piuttosto che la sua gamma.

**Leggere `reflectance_clipped_fraction`, non il flag.** `reflectance_clipped_at_1_0` si attiva per un
singolo pixel — in una scena reale ha segnalato `True` mentre la frazione era `0.000001`, un pixel
su un milione. È la frazione che indica se è necessario aumentare il limite massimo.

**`bit_depth=8` riguarda solo la riflettanza.** Richiederlo per `radiance` viene respinto anziché
soddisfatto: la riflettanza ha un intervallo fisso compreso tra 0 e 1 che resiste a una codifica a 8 bit, mentre la radianza non ha
un intervallo fisso, quindi inserirla in 8 bit richiede un allungamento percentile per fotogramma la cui mappatura
cambia ad ogni fotogramma — un’immagine, non una misurazione.

#### Flusso di riflettanza in tempo reale (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` fornisce la riflettanza a 8 bit per banda come
`multipart/x-mixed-replace` — un M3C come JPEG a 3 canali, un M3M in scala di grigi. Parametri di interrogazione:
`fps` (impostazione predefinita 10), `jpeg_q` (impostazione predefinita 95), `max_h` (0 = nativo), `band` (una banda denominata, come
sopra), `align` (da `1` a co-registrazione — stesse regole dell’`align` dell’SDK), `refl_max` (il limite massimo di codifica a 8 bit
, come sopra). Ogni parte contiene
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` e `X-Chloros-Reflectance-Clipped`.
**Le parti provenienti da telecamere diverse che riportano lo stesso `X-Chloros-Group-Id` corrispondono allo stesso trigger** —
è così che vengono riassemblati i diversi flussi per telecamera di un array, e ciò è garantito
lato server poiché i membri dell’array vengono forniti dal gruppo di fotogrammi sincronizzato (vedi
[Un trigger per lettura](#one-trigger-per-read--group_id)). Restituisce `409` quando la riflettanza non è
disponibile al momento della connessione e termina lo stream se diventa indisponibile durante l’esecuzione; pertanto, uno stream in stallo
significa “riflettanza non più disponibile” piuttosto che “tutto a posto, solo statico”.

> **Il formato JPEG è con perdita di dati — questi pixel rappresentano una misurazione.** Gli errori di compressione si riflettono direttamente nei
> valori di riflettanza e sono più evidenti lungo i bordi, dove si trovano i confini dei materiali. Per questo motivo, la qualità
> è impostata di default a 95. Se avete bisogno che i numeri rimangano intatti, utilizzate
> `grab(processing="reflectance")` / `frame_stream(...)`, che restituiscono lo stesso risultato senza
> codec nel percorso (e `bit_depth=8` se il vincolo è il costo della connessione, non il codec).

> **I dati EXIF vengono trasferiti nell’esportazione.** `process()` copia il blocco GPS dell’acquisizione di origine
> **e il suo ExifIFD** su ogni prodotto, quindi le esportazioni contengono `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` e `CameraSerialNumber`, oltre al
> georeferenziazione. `FocalLength` è il codice da cui Pix4D ricava la distanza campionaria al suolo — senza di esso
> la ricostruzione ricade su una scala estremamente errata (in un caso misurato, un sito di 411 m
> è stato trasformato in uno di 47,8 km). Il file non è volutamente `-all:all`: i tag strutturali di IFD0 compromettono
> l’output di LATTICE, mentre `ExifImageWidth`/`Height` sono esclusi perché descrivono l’acquisizione
> di origine piuttosto che il raster esportato.

Sottomarcatori della fase di acquisizione (si applicano ai livelli radiometrici — `radiance`, `reflectance`, `display`):

| Flag | Predefinito | Significato |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + separazione 3x3 + scala radiometrica NIST. |
| `apply_white_balance` | `True` | LUT di bilanciamento del bianco. Compatibile con DLS quando un DAQ è associato alla fotocamera. |
| `apply_index` | `False` | Valutazione dell’indice di vegetazione. |
| `index_expression` | `None` | Formula di override. Se nonvuoto → abilita automaticamente l’indice. |
| `annotated` | `False` | Sovrapposizione decorazioni GUI (zebra/griglia/peaking). Non disponibile per `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Il valore restituito è un `chloros_sdk.CapturePaths`, di tipo `CapturePathMap`, non `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` è `Dict[str, Union[str, List[str]]]`: un percorso a livello singolo
> `processing` assegna a ciascuna seriale un percorso, mentre `processing="all"` le assegna l’
> **elenco ordinato** di tutti i prodotti salvati per quella fotocamera. `ArrayHandle.capture()` non accetta
> nessun `levels=` e ignora gli argomenti chiave che non riconosce (`levels`, `daq`,
> `render_index`, `force_daq`), quindi passare solo quelli qui indicati (`aligned=False`
> salva i file nativi, come su `ArraySession.capture()`); usa
> `ArraySession.capture()` per quei controlli. Un composito combinato in tempo reale, se fosse in streaming, arriva sotto la chiave aggiuntiva
> `"combined"` anziché sotto un numero seriale. Il codice che presuppone `str` genera un errore nella
> forma a elenco senza che alcun verificatore di tipi lo segnali — l’annotazione indicava `Dict[str, str]`
> per un certo periodo dopo il rilascio della forma a elenco, motivo per cui esiste l’alias. Normalizza
> quando desideri la forma piatta:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Cosa non si poteva scrivere: `.failed`.** La mappa restituita da `capture()` è una
> `chloros_sdk.CapturePaths` — una semplice sottoclasse di `dict`, quindi tutto quanto sopra funziona
> senza modifiche — che contiene anche `.failed`: i file che il backend non è riuscito a scrivere,
> le stesse voci di [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> (`serial`, `level`, `error` e `disk_full: True` su un disco pieno). Una
> cattura parziale restituisce comunque tutti i percorsi in cui è stata effettuata la scrittura; su un disco pieno la mappa è
> vuota e `.failed` ne spiega il motivo. `.skipped` elenca i salti per singola telecamera e
> `.group_seq` il gruppo sincronizzato, mentre `.alignment` / `.oriented` sono quelli
> del backend, come in `CaptureResult`. `capture_smart()` li restituisce come
> `result["frames"].failed` e `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Allineamento degli array

`ArrayHandle` espone l’intera superficie di allineamento. Per impostazione predefinita, i profili sono validi solo per la sessione: chiamare esplicitamente `export_alignment()` per renderli persistenti.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Allineamento al momento della connessione

`connect_all(align=...)` può allineare automaticamente ogni array al momento della connessione:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Se non specificato, ricorre a `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

Un sensore di luce salvato (DAQ-U, DAQ-M o DAQ-E), aperto da `connect_all()` nel [pool di sensori DAQ](#daq-sensor-sessions) — lo stesso pool utilizzato da `connect_daq_sensor()` e `chloros-cli daq pool-connect`.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` attende lo **spettro successivo** del sensore — quello già memorizzato nel buffer al momento della chiamata non viene conteggiato — e restituisce `(spectrum, is_saturated, integration_time, x, y, z)`, oppure `None` se non ne arriva nessuno entro 5 s. Uno stream messo in pausa viene ripreso. `spectrum` è espresso in W/m²/nm quando il sensore è calibrato.
- `.sensor` è il [`DAQSensorSession`](#sessioni-sensore-daq), per tutto il resto: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` e `.settings` sono le voci salvate da `sensors.json`.

**Cosa ripristina `connect_all`.** Ogni sensore viene aperto al proprio indirizzo salvato — la sua porta COM, l’indirizzo Bluetooth o l’host DAQ-E — e, se il progetto ha registrato l’ID del sensore, deve trattarsi proprio di quel sensore: un’unità diversa a quell’indirizzo viene rifiutata, in quanto non collegata. Vengono applicati il tempo di integrazione salvato, la media dei fotogrammi e l’esposizione automatica, nonché la correzione del cappuccio:

| Profilo salvato | Collegato con |
| --- | --- |
| Un profilo (ad es. `fov_30`) | Quel profilo. |
| Nessuno selezionato | `sunshine_cosine`, l’impostazione predefinita di connessione, registrata come previsto (`cap_id_source=auto_default`), quindi l’elaborazione genera un avviso. I sensori vengono forniti senza coperchio: scegliere il coperchio montato. |
| `none` — selezionato come **Nessuno (sensore nudo)** nell’interfaccia grafica | Nessuna correzione del cappuccio. |

Un DAQ-E-S non accetta alcun profilo del cappuccio, indipendentemente da quanto sia stato salvato: il suo diffusore fa parte della sua calibrazione (vedi [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Un progetto salvato da una versione precedente di Chloros registrava un sensore nudo come opzione predefinita, quindi si ricollega con `sunshine_cosine`; selezionare nuovamente **Nessuno (sensore nudo)** nell’interfaccia grafica e salvare il progetto.

Un sensore che un altro client ha già aperto nel pool (CLI, un altro script) viene collegato così com’è, senza riapplicare le impostazioni, e `disconnect_all()` lo lascia in esecuzione; un sensore aperto da `connect_all()` viene rilasciato. Come nel caso di `connect_daq_sensor()`, l’interfaccia grafica di Chloros **non** fa parte del pool: chiudere un sensore nella GUI prima di collegare il progetto da uno script.**Requisiti e rapporto.** I sensori di luce richiedono un backend Chloros in esecuzione con accesso a un piano a pagamento Chloros+ (Copper o superiore), e `connect_all()` non ne avvia mai uno. Se non ce n’è uno in esecuzione, ogni sensore salvato viene elencato sotto `report["sensors"]["errors"]`, con un’indicazione in tal senso. Anche un sensore che il backend non riesce ad aprire viene elencato lì, con il motivo indicato dal backend; `report["sensors"]["missing"]` rimane vuoto per i sensori di luce.

---

## Hardware diretto (senza backend)

Quando si desidera azzerare la dipendenza dal backend per le telecamere LATTICE (CI, robot senza testa, sistemi embedded), importare direttamente `lattice_sdk` — esso viene riesportato da `chloros_sdk`. Avviso su `CAMERA_AVAILABLE`: `lattice_sdk` è presente nel wheel dell’SDK, ma richiede la presenza del runtime dell’Arena SDK e di OpenCV (`cv2`), e il wheel non installa nessuno dei due. I sensori di luce passano attraverso il backend — [`connect_daq_sensor()`](#daq-sensor-sessions) o l’`proj.sensors` di un progetto — con un’unica eccezione, l’helper DAQ-M per l’ampiezza di banda [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), che richiede anche `bleak`.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Preimpostazioni e trigger

Tre delle quattro preimpostazioni sono in modalità **free-run**: la fotocamera espone in modo continuo e un
`capture()` restituisce il fotogramma successivo. `triggered` è l’eccezione : prepara la
fotocamera per un fronte d’onda hardware sulla Linea 2, quindi non cattura nulla finché non ne arriva uno.

| Preset | Trigger | Da usare quando |
| --- | --- | --- |
| `default` | free-run | uso generale |
| `high_speed` | free-run | 8 bit, limite a 60 fps, esposizione breve |
| `high_quality` | funzionamento libero | 12 bit, nessun limite di fps — la scelta più comune per le foto |
| `triggered` | **pronta, Linea 2** | la fotocamera è collegata a un cavo di sincronizzazione M8 e viene attivata da un altro dispositivo |

Se si seleziona `triggered` (o si imposta manualmente `trigger_mode="On"`) senza che nulla
comandi la Linea 2, ogni `capture()` andrà in timeout — correttamente, poiché hai chiesto
alla telecamera di attendere. L’SDK lo spiega quando ciò accade; vedi
[SC_ERR_TIMEOUT durante l’acquisizione](#direct-hardware-backend-free).

> **Nota — I messaggi &quot;GVSP probe&quot; / `SC_ERR_TIMEOUT -1011` al momento della connessione non sono errori.**
> Al momento della connessione, l’SDK cerca di negoziare i**jumbo frame** (pacchetti GVSP da 9000 byte) per ottenere una maggiore velocità di trasmissione. Su un collegamento NIC diretto punto-punto (ad es. un indirizzo `169.254.x.x` link-local) la rete solitamente non è in grado di trasportare jumbo frame, quindi questa sonda va in timeout e registra righe come:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Questa è la **soluzione di ripiego prevista**: l’SDK torna automaticamente ai pacchetti standard da 1500 byte e la telecamera continua a connettersi normalmente (le righe `[chunk-enable …]` che seguono fanno parte della normale sequenza di connessione). L’acquisizione continua a funzionare.
>
> È possibile ignorare questa sonda, ma **non si tratta semplicemente di un silenziatore di log: disattiva i frame jumbo.** La telecamera risponde ai ping “Don&#x27;t-Fragment” solo fino a 1500 byte, indipendentemente dalla qualità della rete; pertanto, il test del ping da solo non può mai rilevare i jumbo; questa sonda è l’unica in grado di farlo. Se la si disabilita, la telecamera utilizzerà per sempre pacchetti standard da 1500 byte, su qualsiasi rete:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Ne vale la pena solo su una rete che *sai* non supportare i jumbo, dove consente di risparmiare circa un secondo di tempo di connessione per ogni telecamera. Poiché si tratta di una vera e propria scelta di compromesso piuttosto che di una modifica puramente estetica, l’SDK ora lo segnala quando lo si utilizza:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Non modificarlo a meno che non si abbia un motivo valido.**Se lasciato abilitato, ogni telecamera viene misurata una volta per ogni sessione del backend e nuovamente quando il suo collegamento cambia (un altro adattatore, un’altra velocità di collegamento o un collegamento interrotto, come nel caso della sostituzione di uno switch); le riconnessioni riutilizzano il risultato.**Dopo una modifica dietro uno switch rimasto attivo, riavviare il backend per effettuare una nuova misurazione.** Una prima connessione può occasionalmente richiedere alcuni minuti per questo controllo, e il log lo segnala quando ciò accade.
>
> Se *desideri* il throughput jumbo, abilita il jumbo end-to-end (MTU della scheda di rete a 9000 + uno switch che li lasci passare), oppure fissalo con `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` quando sai che il collegamento lo supporta — anche se è preferibile utilizzare `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` per singolo comando piuttosto che impostarlo in modo permanente, poiché una dimensione fissa salta la fase di sondaggio e impedisce l’adattamento alla rete a monte. **Ogni** dispositivo lungo il percorso deve supportare i pacchetti jumbo — compresi eventuali splitter o iniettori PoE, che sono la causa più comune per cui una configurazione altrimenti compatibile con i pacchetti jumbo non riesce a trasmetterli.

> **`SC_ERR_TIMEOUT -1011` durante `capture()` / `grab*()` è un problema diverso — in quel caso si tratta di un vero e proprio errore.**
> La nota sopra riportata riguarda esclusivamente l’errore `-1011` registrato dal**sensore di tempo di connessione**. Lo stesso errore generato da una**cattura** indica che la telecamera si è connessa correttamente ma non sta inviando alcuna immagine:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> L’indizio rivelatore è una telecamera il cui canale *di controllo* è integro — il rilevamento funziona, le impostazioni e le scritture `[chunk-enable …]` vanno a buon fine — mentre *ogni* fotogramma va in timeout.
>
> **La causa più comune è che la telecamera sia impostata per un trigger hardware.** Con i codici `trigger_mode="On"` e `trigger_source="Line2"`, la telecamera non trasmette nulla finché non arriva un fronte di segnale sul cavo di sincronizzazione M8. Se non c’è alcun cavo che pilotia quella linea, ogni acquisizione rimane in attesa all’infinito. La telecamera non è guasta e la rete funziona correttamente — sta semplicemente facendo esattamente ciò che le è stato comandato.
>
> `CameraSettings()` e i preset `default` / `high_speed` / `high_quality` impostano il funzionamento in modalità libera, e un&#x27;acquisizione che va in timeout mentre è armata fornisce una spiegazione invece di visualizzare semplicemente un `-1011`. `PRESETS["triggered"]` arma la Linea 2, come previsto.
>
> Per forzare qualsiasi telecamera in modalità free-run:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Se il timeout persiste con `trigger_mode="Off"`, significa che la telecamera non sta effettivamente trasmettendo dati: inviateci il log e `ip link show`.

#### Profili colore (anteprima in tempo reale RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` seleziona il profilo colore dello schermo per l’**anteprima in tempo reale** sulle telecamere RGB (le telecamere multispectrali ignorano questa impostazione):

| Profilo | Significato |
| --- | --- |
| `raw` | Ignora completamente la catena radiometrica. |
| `linear` | DSNU + flat + WB, senza CCM, senza gamma. |
| `natural` | Lineare + CCM misurato + gamma sRGB, solo con la finitura “economica” (smussamento della crominanza + desaturazione delle alte luci) — l’impostazione predefinita realistica. |
| `enhanced` | `natural` più la finitura completa con parità di hub (rimozione delle frange, vivacità, contrasto locale CLAHE). Aspetto più ricco a circa **il doppio del costo di finitura per fotogramma**, quindi un framerate LIVE inferiore. |
| `custom_temp` | `natural` ma con il bilanciamento del bianco fissato a `custom_cct_k` Kelvin (DLS ignorato; limitato a 2000–10000 K lato backend). |

Il profilo è una regolazione di velocità/aspetto **solo per l’anteprima in tempo reale**: le acquisizioni salvate ottengono sempre la finitura completa e ricca indipendentemente dal profilo selezionato, quindi scegliere `natural` per recuperare tempo di frame non riduce la qualità di ciò che viene salvato su disco. Un profilo sconosciuto attiva `ValueError`; quando un backend chloros è raggiungibile, la modifica viene inviata anche a esso tramite POST, in modo che il fotogramma di anteprima successivo la rifletta (gli utenti SDK diretto senza backend ricevono comunque la modifica delle impostazioni).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Punti salienti (display RGB) — `set_highlights`

`LatticeCamera.set_highlights(value)` riduce i pixel più luminosi e più bianchi del **display** sulle telecamere RGB — la correzione per una superficie bianca brillante che viene renderizzata come una macchia bianca piatta. `value` è `-100`..`0` (limitato); `0` è disattivato e il rendering risulta esattamente come senza l’impostazione. Impostazioni più elevate abbassano il bianco del display (fino a circa il 60% con `-100`) e ampliano la curva di tono, quindi una superficie che l’anteprima automatica aveva reso di bianco puro riacquista i propri dettagli.

Si tratta di un controllo dello schermo, ma a differenza della saturazione e del contrasto (che modificano solo l’anteprima in tempo reale) modifica anche ciò che si salva: l’anteprima in tempo reale e le acquisizioni **a livello di schermo** ne seguono l’impostazione, quindi anche quei file risultano più scuri (riportano il valore `Chloros:DisplayHighlights`); le esportazioni di radianza e riflettanza non lo fanno mai, e non è in grado di recuperare i dati del sensore che hanno subito clipping — per questo è necessario ridurre l’esposizione. Le telecamere multispectrali e monocromatiche lo ignorano. Il valore viene memorizzato su `settings.highlights` e inviato via POST a un backend chloros raggiungibile, esattamente come `set_color_profile`; l’equivalente in CLI è `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Telecamere mono (M3M) e `Calibration`

Una telecamera mono **M3M** (`M3M-<lens>-F<wavelength>`) è a banda singola: un unico piano in scala di grigi, nessun mosaico di Bayer, nessuna matrice di diafonia spettrale 3×3. `Calibration` la riconosce ed espone un flag `is_mono`. La riflettanza si applica comunque come mappa radiometrica per banda (la decomposizione è la matrice identità), ma i calcoli multibanda su una singola telecamera generano risultati validi anziché restituire valori privi di senso:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Per costruire un indice di vegetazione con hardware monocromatico, combinare diverse telecamere M3M a diverse lunghezze d’onda in uno stack multibanda allineato (vedi [Allineamento dell’array](#array-alignment)) e calcolare l’indice su quell’stack anziché su una singola telecamera.

#### Magnitudini delle bande DAQ-M — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` si connette a un DAQ-M tramite Bluetooth utilizzando il proprio MAC BLE. È l’unica classe di sensori di luce che comunica direttamente con il sensore anziché tramite il backend, pertanto richiede ciò di cui hanno bisogno le classi delle telecamere dirette (`CAMERA_AVAILABLE`: il runtime dell’SDK di Arena e OpenCV) più `bleak` per il Bluetooth (`pip install "chloros-sdk[camera]"` o `pip install bleak`; senza di esso, `DLS` genera l’errore `DLSError`). Un DAQ-M collegato altrove (il pool del backend, un progetto, la GUI) smette di trasmettersi, quindi è necessario liberarlo prima in quel contesto. **`mac_address` è obbligatorio**: non esiste un valore predefinito, poiché ogni unità ha il proprio, e `DLS()`, in assenza di tale valore, genera un errore `DLSError` prima che venga tentata qualsiasi connessione. Trovare l’indirizzo MAC con `discover_daq_sensors(transports=["ble"])` (vedi [Rilevamento](#discovery--finding-an-address-to-connect-with)) o `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integra uno spettro DAQ-M (l’ultima lettura quando `spectrum` è `None`; `DLSError` se non ce n’è ancora uno) attraverso le risposte dei tre canali di una telecamera e restituisce `{"red": …, "green": …, "nir": …}` — le chiavi indicano i canali in quell’ordine; con le curve di una telecamera RGB, il terzo (`nir`) è il canale blu.

- Senza `spectral_response` utilizza la curva fleet del modello di filtro (quella con cui è stato costruito `filter_model` e `DLS`), che è stata misurata su una singola unità.
- `spectral_response` utilizza invece le curve misurate **dalla stessa** telecamera: un dizionario `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, come memorizzato nel pacchetto di calibrazione di quella telecamera (`color_correction.spectral_response_smoothed` — telecamere RGB; i pacchetti multispettrali non contengono tale curva). Se il dizionario non è utilizzabile, si ricorre alla curva della flotta.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

Il comando `chloros-cli lattice dls --camera SERIAL` dell’CLI esegue la stessa operazione tramite il backend, individuando la curva della telecamera nel pacchetto di calibrazione caricato o memorizzato nella cache, e indica di chi è la curva utilizzata. Su HTTP si tratta di `POST /api/dls/cli-read` con `mac` (obbligatorio: in sua assenza la risposta è un errore HTTP400, `code` `MAC_REQUIRED`) e `camera_serial` nel corpo del messaggio; il valore `band_magnitudes_source` della risposta diventa `camera <serial>` quando viene utilizzata la curva propria di quella telecamera, oppure `model <filter>` per la curva della flotta (una telecamera senza curva: non RGB, o nessun pacchetto caricato o memorizzato nella cache).

---

## Eccezioni

Intercettare la classe base per gestire &quot;qualsiasi cosa sia andata storta in Chloros&quot;:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` e `ChlorosConfigurationError` sono attributi di `chloros_sdk` (e di `chloros_sdk.exceptions`) ma non in `__all__`, quindi `from chloros_sdk import *` non li include: importarli per nome. La famiglia `ChlorosError` è derivata da `ChlorosLocal` e dalle funzioni basate su di essa (`process_folder()`, `process_lattice_capture()`); la superficie smart-connect erede di `ChlorosConnectError`.

Gerarchia:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Esempi end-to-end

### 1. Elaborazione di una cartella con una barra di avanzamento personalizzata

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Matrice LATTICE in tempo reale → Riflettanza + Riferimento DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Campagna di acquisizione basata su progetto

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Flusso di fotogrammi da più telecamere → Pipeline NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Script di acquisizione headless direttamente su hardware (senza backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Verifica delle funzionalità prima di collegare un array a 4 telecamere

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Ricetta di acquisizione equivalente (Python puro)

`run_project_recipe()` esegue una ricetta CLI così com’è (vedi [Cartella del progetto e ricette](#project-folder--recipes)). Se invece scritti in Python, gli stessi passaggi sono:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Server MCP (assistenti AI)

`chloros-mcp` consente a un assistente AI che utilizza il [Model Context Protocol](https://modelcontextprotocol.io) (MCP), come Claude Desktop, Claude Code o Cursor, di gestire Chloros: elaborare immagini (creare un progetto, importare una cartella di acquisizioni, scegliere le impostazioni, elaborare i dati, generare report sui risultati dell’elaborazione) ed eseguire telecamere LATTICE, array e sensori di luce DAQ (connessione, anteprima, impostazione di esposizione e colore, acquisizione, registrazione, aggiornamento del firmware). I suoi strumenti coprono le funzionalità di questo SDK. Basta dare istruzioni in linguaggio naturale e l’assistente richiamerà gli strumenti del server:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Il server è basato su questo SDK: gli strumenti di elaborazione richiamano [`ChlorosLocal`](#image-processing--chloroslocal) e gli strumenti hardware le [chiamate smart-connect](#smart-connect-for-lattice-cameras), [sessioni DAQ](#daq-sensor-sessions), `ChlorosProject` e le [utilità della fotocamera](#camera-utilities), tutte collegate allo stesso backend utilizzato dall’app Chloros. Le sue richieste contengono `X-Chloros-Client: sdk`, quindi il backend applica i controlli di accesso e di piano previsti dall’SDK, e con essi vengono applicate anche le regole dell’SDK (ad esempio, Texture Aware viene rifiutato per le acquisizioni LATTICE).

### Installazione

Il server è incluso nel pacchetto `chloros-sdk`; non c’è nient’altro da installare. L’SDK installa un comando `chloros-mcp`, che è identico a `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** il programma di installazione di Chloros installa l’SDK tramite pip nel vostro Python, inserendo `chloros-mcp.exe` nella cartella `Scripts` di quel Python.
- **Linux (.deb):** il pacchetto installa l’SDK per il sistema `python3` e inserisce il comando `chloros-mcp` in `/usr/bin`, in modo che sia presente nell’`PATH`. In un virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` aggiunge `chloros-mcp` al `bin` del virtualenv.

Il server utilizza solo la libreria standard integrata nell’SDK (Python 3.7+). Un client MCP lo avvia e comunica con esso tramite stdin/stdout; non è necessario eseguirlo manualmente, tranne che con `--help`.

### Configurazione del client

**Claude Desktop:**aggiungi il server a `claude_desktop_config.json` (su Windows, `%APPDATA%\Claude\claude_desktop_config.json`; si apre da**Impostazioni &gt; Sviluppatore &gt; Modifica configurazione**), quindi riavvia Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursore:** la stessa voce `mcpServers` di Claude Desktop, in `~/.cursor/mcp.json` (in ogni progetto; `%USERPROFILE%\.cursor\mcp.json` su Windows) o `.cursor/mcp.json` in un progetto.

Le opzioni vanno inserite in `args`, oppure dopo il comando per Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Per utilizzare un particolare SDK di Python al posto di `chloros-mcp` su `PATH`, eseguire il modulo con quel Python (il suo percorso completo come `command` se non è il primo `python3` su `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Un client avvia il server senza la tua shell, quindi se segnala che il server non è riuscito ad avviarsi, probabilmente `chloros-mcp` non si trova sul `PATH` visibile al client. Specificare invece il percorso completo (`where chloros-mcp` su Windows, `which chloros-mcp` su Linux, dove il file .deb si trova in `/usr/bin/chloros-mcp`), oppure eseguire il modulo con la versione di Python che include l’SDK.

### Opzioni

| Opzione | Predefinito | Funzione |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Il backend Chloros da utilizzare. |
| `--no-auto-start` | off | Non avviare mai un backend: utilizzare uno già in esecuzione, come quello dell’app Chloros, il servizio backend Linux o un tunnel verso la (vedi [Modalità backend remoto](#remote-backend-mode-pip-only-host-via-tunnel); in tal caso, i percorsi delle cartelle devono indicare la stessa cartella su entrambe le macchine). |
| `--backend-exe PATH` | dove Chloros lo installa | Il backend da avviare quando non ce n’è nessuno in esecuzione, per un Chloros installato in una posizione non rilevata dall’SDK. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` o `ERROR`. I log vengono inviati a stderr, che la maggior parte dei client mostra nel proprio log MCP; stdout trasporta solo i messaggi di protocollo. |

### Strumenti

Gli argomenti di ogni strumento sono quelli della chiamata SDK che esso incapsula; il client elenca ciascuno di essi con il proprio schema completo. I percorsi sono assoluti.

**Elaborazione**

| Strumento | Chiamata SDK | Cosa fa |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Crea un progetto e lo imposta come progetto aperto, su cui agiscono gli strumenti di elaborazione. Un nome semplice crea la cartella nella cartella predefinita del progetto; un percorso assoluto la crea in quella posizione. |
| `open_project` | `ChlorosLocal.open_project` | Apre un progetto salvato e lo imposta come progetto aperto. I percorsi di un progetto spostato vengono reindirizzati alla cartella in cui si trova ora; `relink_folder` (assoluto) indica la cartella in cui si trovano ora gli altri file mancanti. Restituisce `source_files` e `image_count`. Nessuno trovato: un errore che elenca dove si trovavano e indica `relink_folder`; non c’è nulla di aperto. |
| `import_images` | `import_images` | Importa le coppie `.RAW`+`.JPG` di Survey3, `.tif` acquisizioni e DAQ `.daq` registrazioni; restituisce `count` (immagini) e `scan_count` (registrazioni). |
| `configure_processing` | `configure` | Modifica solo le impostazioni passate e nessun&#x27;altra (valori: [Valori supportati](#supported-values)); `custom_settings` invia l&#x27;intera configurazione così come specificata. |
| `get_project_settings` | `get_config` | Legge le impostazioni di elaborazione, esportazione e indice salvate del progetto aperto. |
| `start_processing` | `process(wait=False)` | Avvia l’elaborazione del progetto aperto e torna immediatamente. |
| `get_processing_status` | `get_status` | Segnala `state` (`idle`, `starting`, `processing` o `finished`) e lo stato di avanzamento; una volta terminata, l’esecuzione genera `summary`, `export_check_failure` quando non è stato possibile verificare le esportazioni e `image_failure` quando le immagini sono andate perse: gli altri prodotti vengono salvati; segnalare l’esecuzione come fallita. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Crea un progetto per una cartella, lo importa, lo configura e avvia l’esecuzione in un’unica chiamata (`lattice: true` per le acquisizioni LATTICE). Mentre un&#x27;esecuzione è in corso, il sistema la interrompe prima di creare qualsiasi cosa, e anche il backend rifiuta in quel momento qualsiasi modifica al progetto da qualsiasi strumento. |
| `export_light_sensor` | `export_light_sensor` | Scrive i file calibrati `.daq` e `.csv` per le registrazioni del sensore di luce del progetto senza elaborare le immagini, recuperando la calibrazione di fabbrica da MAPIR Cloud quando non è memorizzata nella cache (vedi [Registrazioni del sensore di luce](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Legge i metadati di audit di Chloros relativi a un&#x27;immagine elaborata (fonte, calibrazione, fasi applicate). |

**Telecamere e array**

| Strumento | Chiamata SDK | Cosa fa |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Cosa è attualmente connesso, in qualsiasi client, ciascuno con `connected_here`; oltre ai lavori in esecuzione sul server. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Telecamere e sensori di luce che potrebbero essere collegati. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Stato in tempo reale di una telecamera, di un array (con le relative capacità e condizioni operative) o di un sensore di luce. |
| `analyze_array_network` | `analyze_array_network` | Capacità di questo host e di questa rete per un array proposto. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Collegare una telecamera o un array sincronizzato (prima la telecamera master). |
| `connect_project_devices` | `open_project(...).connect_all` | Collegare tutti i dispositivi memorizzati in un progetto salvato, con le relative impostazioni. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Rimuovere una telecamera, un array o un sensore di luce (vedere Comportamento). `keep_settings: true` su un array salva le impostazioni di ciascuna telecamera nella propria memoria, `clear_settings: true` ripristina quella memoria (il cui risultato comporta `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | I dispositivi del progetto, tutti insieme. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Leggere o scrivere le impostazioni della fotocamera, tramite nome descrittivo o nome del nodo GenIC. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Imposta la frequenza di un array o ripristina la frequenza automatica. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Una singola acquisizione (`fastest` e `wait_for_exposure` sono `capture_fastest` e l’acquisizione con esposizione stabilizzata). In assenza di `output_dir`, il file viene salvato nella cartella aperta del progetto`captures` del progetto aperto, e i file vengono aggiunti al progetto (trattenuti fino al termine di un ciclo di elaborazione); la risposta `project_add` lo conferma. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Acquisizioni continue o a intervalli, come processo. |
| `start_array_recording` | `ArraySession.record` / `burst` | Un video con indice combinato o una raffica di dati grezzi, come processo. |
| `build_burst_video` | `ArraySession.build_video` | Rielaborare una sequenza in un video calibrato, come processo. |
| `get_job_status` / `stop_job` | | Seguire o interrompere un&#x27;attività. |
| `get_camera_preview` | | Anteprima in tempo reale di ciascuna telecamera, restituita come immagine. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Un fotogramma (o una serie sincronizzata) a un livello di elaborazione, senza salvataggio: statistiche per piano e un file PNG per ciascuno. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Allineamento di un array. |

**Sensori di luce**

| Strumento | Chiamata SDK | Funzione |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Collega un DAQ-U, DAQ-M o DAQ-E (`cap_id` imposta la correzione del limite massimo). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Le letture più recenti: timestamp, saturazione, intervallo e picco dello spettro, nonché lo spettro mediato in intervalli di lunghezza d’onda (`full_spectrum` per ogni punto). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Avvia o arresta il flusso. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Registra un file `.daq` come processo; `duration_s` lo interrompe automaticamente. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | La correzione del limite (`none` per un sensore nudo). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Quale sensore fornisce i dati di riflettanza a ciascuna telecamera e a ciascun array. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Le bande e la riflettanza e gli indici &quot;Ambiente + Oggetto&quot; dei due sensori in streaming. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Impostazioni specifiche di un DAQ-E, salvate nell’unità. |

**Utilità della fotocamera** ([Utilità della fotocamera](#camera-utilities))

| Strumento | Chiamata SDK | Funzione |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` e i relativi setter | Colore di visualizzazione di una telecamera RGB: profilo e temperatura di colore, saturazione / contrasto, luci. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Bilanciamento del bianco one-shot, oppure automatico / disattivato / rapporti fissi. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Preimpostazioni della telecamera. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | I modelli di filtro, oppure uno per nome. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Versioni del firmware e un flash (un&#x27;operazione predefinita). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Stato di alimentazione e termico; in sospensione o attivo. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Coefficienti provenienti da un bersaglio di calibrazione in vista. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Gli adattatori di rete della telecamera, una stima e la correzione (Solo amministratore). |
| `apply_vignette_correction` | `apply_vignette_correction` | Correzione della vignettatura di una cartella di immagini. |
| `read_dls` | `read_dls` | Una lettura DLS. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Hardware e runtime dell&#x27;host. |

**Telecamere transitorie, profili di allineamento e indici** ([Matrici transitorie, profili di allineamento e indici](#transient-arrays-alignment-profiles--indices)). Ogni strumento qui presente che apre le telecamere utilizza `keep_settings` / `clear_settings` (impostazione predefinita: false): vedere [Memoria della telecamera](#camera-memory--keep_settings--clear_settings).

| Strumento | Chiamata SDK | Cosa fa |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Registra una telecamera o trasmette in streaming più telecamere in sincronia, come un’attività (`duration_s` lo interrompe automaticamente). |
| `capture_cameras_transient` | `capture_cameras_transient` | Una singola acquisizione sincronizzata senza collegare un array. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Verifica della sincronizzazione del trigger e delle telecamere presenti sulla linea di sincronizzazione. |
| `analyze_array_models` | `analyze_array_models` | Cosa otterrebbe un array di questi modelli su questa rete. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Risolvere e salvare un file di profilo di allineamento (`options` contiene le opzioni di risoluzione più dettagliate). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Un&#x27;immagine multibanda allineata o una sequenza di immagini di questo tipo come processo. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Il contenuto di un file di profilo; l&#x27;ordine delle bande e le telecamere abilitate. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Preimpostazioni e gradienti dell’indice; un indice relativo a un file TIFF multibanda; acquisizione e creazione dell’indice in un unico passaggio. |

**Sincronizzazione temporale, cartella di progetto e ricette** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Cartella di progetto e ricette](#project-folder--recipes))

| Strumento | Chiamata SDK | Cosa fa |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | Lo stato di integrità del grandmaster PTP, dei suoi peer e lo stato PTP di ciascuna telecamera array. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Riavviarlo; impostare le sue priorità. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | La cartella in cui vengono creati i nuovi progetti. |
| `run_project_recipe` | `run_project_recipe` | Esegue la ricetta di un progetto salvato come processo con il relativo log. Verifica iniziale: una ricetta che presenta un errore viene rifiutata e non viene eseguito nulla; vengono segnalati gli array/sensori non collegati (non li collega: prima viene eseguito `connect_project_devices`). `dry_run` restituisce il risultato della verifica e non esegue nulla. |

Non esiste uno strumento per aprire un progetto salvato per l’elaborazione: ogni processo di elaborazione inizia con `create_project` (o `process_folder`).

### Requisiti

- **Chloros installato sullo stesso computer.** Il backend di Chloros esegue l’elaborazione. Il server può invece utilizzare un backend raggiungibile tramite un tunnel (`--backend-url` con `--no-auto-start`, vedi [Opzioni](#options)).
- **Essere effettuato l’accesso su quel computer**, nell’app Chloros o tramite `chloros-cli login`. Il server non può effettuare l’accesso al posto tuo: uno strumento chiamato mentre non sei connesso risponde «Non sei connesso a Chloros» indicando il motivo fornito dal backend, e l’assistente te lo comunica.
- **Qualsiasi piano a pagamento Chloros+ (Copper o superiore).** Il server è accessibile tramite SDK, mentre il piano gratuito Iron non ne dispone: uno strumento richiamato sul piano gratuito risponde &quot;Non disponibile su questo piano Chloros&quot; indicando il motivo fornito dal backend.
- Python 3.7 o versioni successive con l’SDK e un client MCP in grado di avviare server locali (stdio).

### Funzionamento

- **Il backend.** L&#x27;avvio del server o l&#x27;elenco dei suoi strumenti non avvia nulla; la prima chiamata a uno strumento stabilisce la connessione. Se l&#x27;app Chloros (o il servizio backend Linux) è in esecuzione, il server utilizza il suo backend e lo lascia in esecuzione. Se nessun backend è in esecuzione, il server avvia quello installato, in modalità senza finestra. Quando il server termina (si chiude il client o si riavvia il server), interrompe i processi di acquisizione e le registrazioni del sensore di luce che aveva avviato e rilascia i dispositivi a cui si era connesso; il backend continua a funzionare, in modo che un&#x27;elaborazione non completata prosegua fino al termine, per poi arrestarsi automaticamente dopo 30 minuti di inattività. Con `--no-auto-start` il server non avvia mai un backend: se si chiama uno strumento senza che ce ne sia uno in esecuzione, viene richiesto di aprire l’app Chloros, e la chiamata successiva, una volta fatto ciò, stabilisce la connessione.
- **Un solo progetto aperto.** Il backend ha un solo progetto aperto, condiviso con l’app Chloros quando questa è aperta; pertanto, non lavorare su un altro progetto nell’app mentre l’assistente è in funzione.
- **Cambiano solo le impostazioni specificate.** `configure_processing` lascia ogni impostazione non specificata al valore salvato (l’SDK `configure()` ripristina le impostazioni predefinite per debayer, correzione della vignettatura, calibrazione della riflettanza, PPK e formato di esportazione quando non vengono specificate; il server le ricava dal progetto). Se non riesce a leggere le impostazioni salvate del progetto, non modifica nulla e lo segnala, a meno che non vengano specificate tutte e cinque. `indices` sostituisce l’elenco degli indici del progetto. Il risultato elenca ciò che è stato inviato come `settings_sent`, mentre sotto `warnings` vengono elencati gli indici che le bande della fotocamera non sono in grado di generare.
- **L&#x27;elaborazione avviene secondo il modello &quot;start-then-poll&quot;.** `start_processing` restituisce un risultato non appena inizia l&#x27;esecuzione, che richiede da pochi minuti a diverse ore, e l’assistente lo segue con `get_processing_status`. `state` corrisponde a `idle` (nessuna elaborazione in corso), `starting` (il backend non ha ancora segnalato l’esecuzione, per un massimo di 20 s), `processing` o `finished`, con il [sintesi](#post-run-summary--hints): cosa è stato esportato e indicazioni qualora sia stato esportato meno del previsto (nessuna immagine, nessun bersaglio di calibrazione trovato). I prodotti vengono salvati nella cartella del progetto, come dall’app. `start_processing` viene rifiutato mentre un&#x27;esecuzione è in corso, e un backend che smette di rispondere viene segnalato come errore, non come esecuzione completata.
- **Cosa vede l’assistente.** Gli avvisi dell’SDK vengono riportati nel risultato dello strumento sotto `warnings`, mentre gli errori (disconnessione, piano, percorso errato) vengono visualizzati come testo di errore che l’assistente può leggere e trasmettervi. Un elenco con più di 20 voci viene ridotto alle prime 20, con la lunghezza completa indicata come `<key>_total`. I percorsi devono essere assoluti.
- **I dispositivi sono condivisi.** Telecamere, array e sensori di luce risiedono nel backend, quindi l’assistente può controllarne uno tramite l’app Chloros, e le modifiche apportate vengono visualizzate nell’app. `disconnect_device` riporta ciò a cui è connesso il server; un dispositivo a cui è connesso un altro client (l’app, `chloros-cli`, uno script, un altro `chloros-mcp`) viene rilasciato solo tramite `force: true`, che è l’`force_disconnect` dell’SDK, e il rifiuto indica chi lo ha collegato quando il backend ne è a conoscenza; un dispositivo che il backend non gestisce affatto viene segnalato come non connesso (`released: false`, `connected: false`, nulla da rilasciare). Una registrazione o uno streaming eseguito dal backend porta con sé la vista `get_job_status` con `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (gruppi di fotogrammi per uno streaming multicamera) — e una riga `summary` (&quot;220 fotogrammi scritti, 55 persi, 0 falliti: ...&quot;) durante l’esecuzione e, al termine, i dati finali; l’elenco fornito da un altro client (`other_clients_jobs`) riporta gli stessi dati (`list_cameras()`, `list_arrays()` e `list_daq_sensors()` riportano i dati di ciascun dispositivo`connected_by`: `client`, `label`, `name`, `at`). Ogni `chloros-mcp` si identifica presso il backend come `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, inviato come `X-Chloros-Client-Label`; qualsiasi script SDK può impostarlo). Anche l’interruzione della registrazione del sensore di luce di un altro client richiede `force`.
- **Un’attività di lunga durata è un lavoro.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, e qualsiasi richiesta di acquisizione, risoluzione o indicizzazione di `background` restituisce immediatamente un `job_id`; `get_job_status` lo segue e `stop_job` lo conclude . Un&#x27;esecuzione continua fino a quando non viene interrotta, a meno che non si specifichino `max_captures`, `duration_s` (o, per un burst, `max_frames`); `min_free_gb` rifiuta di avviarsi quando l’unità ha poco spazio libero. Non viene applicato alcun limite a meno che non ne venga impostato uno. Una registrazione o uno streaming eseguito dal backend (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) conserva l’ID del processo del backend, quindi un altro `chloros-mcp` — avviato successivamente — lo individua tramite quell’ID: `get_job_status` lo segue (ed elenca tali processi come `other_clients_jobs`, da `GET /api/camera/cli-jobs`, ciascuno con `started_by`), e `stop_job` lo interrompe solo con `force: true`. Qualsiasi altro processo (un&#x27;acquisizione, una registrazione in array o un burst, un&#x27;acquisizione in background) viene eseguito nel processo `chloros-mcp` che lo ha avviato, e solo quel server ne è a conoscenza: come conferma un altro che lo ha richiesto. Un processo interrotto legge `stopping` mentre scrive la propria coda, poi `finished` con i conteggi finali, per 10 minuti e da qualsiasi `chloros-mcp`: quello che l’ha avviato vede l’arresto forzato di un altro allo stesso modo. `get_job_status` non attende mai un arresto: durante uno (il proprio `stop_job`, quello di un altro cliento l’arresto che libera le telecamere da un lavoro terminato autonomamente) risponde immediatamente con `stopping` e i conteggi in tempo reale, mentre `stop_job` di un lavoro già in fase di arresto lo segnala. Un lavoro terminato ne indica il motivo in `stop_reason`: il proprio motivo (`no_frames`, `disk_full`, `member_missing`, oppure, per un&#x27;esecuzione continua in array, `count`, `duration`, `array_gone`), `limit reached` solo quando i suoi conteggi indicano il suo `count` (o l’`max_frames` di un burst, l’`duration_s` di una registrazione)`duration_s` di una registrazione) è stato raggiunto, `error`, `stopped` (con `stopped_by`) quando un client lo ha interrotto, altrimenti `finished`.
- **Dove vengono salvate le acquisizioni.** Nella cartella `captures` del progetto aperto, dove l’app le salva, a meno che non venga specificato `output_dir`.
- **Immagini.** `get_camera_preview` e `grab_frames` restituiscono immagini che l’assistente può visualizzare, oltre al relativo JSON. Le immagini `grab_frames` sono sottoposte a regolazione del contrasto per la visualizzazione; le relative statistiche rappresentano i valori reali.
- **Quando il server si chiude**, i processi che aveva avviato vengono interrotti e la registrazione del sensore di luce avviata viene chiusa. Se il server aveva avviato il backend, rilascia anche i dispositivi a cui si era connesso prima di arrestare il backend; nel backend dell’app rimangono connessi.
- **Protocollo.** Solo stdio (nessun endpoint HTTP), solo strumenti (nessuna risorsa o richiesta di conferma), risultati testuali e immagine, versioni del protocollo MCP dal 05/11/2024 al 25/11/2025.**Cosa non è incluso.** Accesso o disconnessione (l’account è tuo; il server non può effettuare l’accesso) e l’arresto di un backend che il server non ha avviato. Per tutto il resto che questo SDK fa con l’elaborazione e l’hardware esiste uno strumento. (Il provisioning di un DAQ-E — il suo nome, il token di autenticazione, password OTA, requisito di firma, ripristino delle impostazioni di fabbrica — e i server DAQ diretti sono di esclusiva competenza di `chloros-cli`, presenti anche nell’SDK: un nome o un token errato blocca ogni installazione di Chloros impedendone l’accesso all’unità.)

---

## Avvio automatico del backend

I punti di accesso smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` e `discover_daq_sensors` — sono client HTTP leggeri che presuppongono che un backend sia in ascolto su `127.0.0.1:5000` (l’URL predefinito dell’interfaccia smart-connect). Quando l’interfaccia grafica o la CLI sono già in esecuzione, ce n’è già uno attivo. Partendo da uno script nudo e crudo, potrebbe non esserci; pertanto queste funzioni **avviano automaticamente il binario del backend in dotazione** (in modalità senza finestra, proprio come fa `ChlorosLocal`) prima della loro prima chiamata, quindi attendono fino a 60 s affinché si avvii. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` e `analyze_array_network()` non ne avviano mai uno.

Regole:

- **Viene generato solo un URL locale.** Un `backend_url` che punta a `localhost` / `127.0.0.1` / `[::1]` è ammesso; qualsiasi altro host viene considerato come il computer di qualcun altro e non viene mai avviato.
- **Un backend che termina prima di rispondere interrompe l’attesa.** La chiamata restituisce immediatamente il codice di uscita, il comando eseguito e le ultime righe stampate dal backend, invece di attendere i 60 s. Il codice di uscita 98 costituisce l’eccezione: un altro backend detiene già la porta o il file di blocco (dell’app Chloros, del servizio backend o di un altro script ancora in fase di avvio), quindi la chiamata continua ad attendere che quello risponda. `ChlorosLocal` si comporta allo stesso modo.
- **Il backend viene lasciato in esecuzione per essere riutilizzato** (come nella CLI) — non c’è uno spegnimento implicito quando lo script termina. Rieseguendo lo script si riutilizza il backend attivo.
- **È possibile disattivare questa opzione con `auto_start_backend=False`** su una qualsiasi di queste chiamate (ad esempio quando si fa riferimento a un backend remoto o si gestisce autonomamente il ciclo di vita del backend).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Se il binario integrato non può essere individuato o avviato, la successiva chiamata HTTP genera un errore `ChlorosConnectError` **specifico per la piattaforma** e su cui è possibile intervenire, anziché una semplice traccia di connessione rifiutata — su Windows indirizza all’app desktop o a un comando `chloros-cli`; su Linux (senza GUI) indirizza a un comando `chloros-cli` o a `.deb`.

---

## Ambiente e intestazioni

L’SDK contrassegna ogni chiamata HTTP del backend con `X-Chloros-Client: sdk`. Il backend applica le regole di licenza SDK/CLI (è richiesto il login **e** un piano a pagamento Chloros+) anziché il percorso gratuito della GUI. Questa impostazione viene configurata automaticamente al momento dell’importazione: non è necessario intervenire.

L’intestazione viene aggiunta a ogni chiamata `requests` / `urllib` il cui URL contiene `localhost` o `127.0.0.1`, su qualsiasi porta — compresi i propri servizi locali — e sostituisce un valore impostato dall’utente. Le chiamate ad altri host rimangono invariate.

È possibile sovrascrivere l’URL del backend passando `backend_url=` (`ChlorosLocal` accetta `backend_url=` oppure `api_url=`; `process_folder()` e `process_lattice_capture()` accettano solo `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Un backend installato è in ascolto solo sul loopback, quindi un `backend_url` in LAN non può raggiungerlo; vedere [Modalità backend remoto](#remote-backend-mode-pip-only-host-via-tunnel) per lo schema del tunnel.)

---

## Versioni e compatibilità

- La versione dell’SDK è indicata come `chloros_sdk.__version__`.
- L’SDK vincola il comportamento alla versione del backend in dotazione. L’utilizzo combinato di un SDK più vecchio con un backend più recente di solito funziona (endpoint compatibili in avanti), ma l’utilizzo combinato di un SDK più recente con un backend più vecchio potrebbe causare errori `404` sui nuovi endpoint: aggiornare l’app desktop in modo che corrisponda.
- L’interfaccia smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) e l’endpoint di analisi di rete restituiscono schemi JSON stabili; i nuovi campi sono aggiuntivi.

---

## Suggerimenti per la risoluzione dei problemi

- **`ChlorosAuthenticationError: Login required`** (da `ChlorosLocal`; le chiamate Smart-Connect generano l’errore `ChlorosConnectError: Login required. …` per la stessa causa) → Eseguire una volta `chloros-cli login EMAIL PASSWORD` su questa macchina oppure accedere tramite l’app desktop Chloros.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (o `Failed to create project: …` derivante da `create_project()`) → I percorsi lunghi di Windows sono disattivati su questa macchina (impostazione predefinita di Windows), quindi nessun percorso può superare i 260 caratteri e i prodotti si trovano fino a circa 130 caratteri al di sotto della cartella del progetto. Non è stata elaborata alcuna operazione. Il messaggio indica la cartella del progetto più lunga che rientra nei limiti: utilizzarne una più breve oppure abilitare i percorsi lunghi di Windows e riavviare Chloros. Il programma di installazione di Windows offre questa opzione come «Abilita percorsi lunghi di Windows», selezionata per impostazione predefinita; in alternativa, utilizzare i Criteri di gruppo: *Configurazione computer &gt; Modelli amministrativi &gt; Sistema &gt; File system &gt; Abilita percorsi lunghi Win32*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Lo smart-connect avvia automaticamente un backend locale, quindi questo errore compare solo quando il binario integrato non viene trovato o non può essere avviato (ad es. un host *solo pipsenza pacchetto desktop). Il messaggio varia a seconda della piattaforma: su Windows, apri l’app desktop o esegui un qualsiasi comando `chloros-cli`; su Linux, eseguire un comando `chloros-cli` (non esiste una GUI) o installare `.deb`. Per un backend remoto, passare `backend_url=` (e `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** all’importazione → `lattice_sdk`: impossibile caricare il runtime dell’SDK di Arena o OpenCV (`cv2`), non è importabile in questa versione di Python. La superficie gestita dal backend (`connect_*`, proiettata tramite un backend, `ChlorosLocal`) funziona ancora.
- **L&#x27;array si connette a sensore completo ma con un frame rate basso**→ Per come è progettato: la modalità automatica mantiene il sensore completo e lascia che il frame rate scenda al livello supportato dalla connessione. Passare a `binning=2` / `4` oppure un valore inferiore come `width` / `height` per ottenere più fotogrammi al secondo, oppure abilitare i jumbo frame / potenziare il collegamento. Il binning e la ROI**non** risolvono il problema dell’oversubscription aggregata (`oversubscribed: true`, campi fps 0): il numero eccessivo di telecamere per la linea non può essere risolto tramite binning/ROI — ridurre il numero di telecamere, abilitare i jumbo frame o passare a una scheda di rete più veloce (vedere [Over-Subscription](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` segnala che l’anello di ricezione della scheda di rete è molto piccolo (~0,26 MB) / i gate di connessione riportano &quot;FRAMES WILL DROP&quot;** → L’anello di ricezione della scheda di rete host è impostato sul valore predefinito (spesso reimpostato a 32 dopo un aggiornamento del driver della scheda di rete). Su un adattatore Realtek USB 10GbE, impostare `ReceiveBufferLen=256` e `PendingReceives=64` (con privilegi elevati), quindi riavviare il backend affinché rilegga l’anello. Procedura completa: [Riferimento CLI → Configurazione e ottimizzazione della scheda di rete host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **L&#x27;host si blocca al riavvio/spegnimento, in seguito si verificano errori WMI `Invalid class` / la scheda di rete non si abilita** → Driver USB 10GbE obsoleto che causa l’errore `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Aggiornare il driver della scheda alla versione più recente (≥ 2026) e riapplicare le impostazioni del ring di ricezione. Vedere [Riferimento CLI → Configurazione e ottimizzazione della scheda di rete host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **L&#x27;acquisizione dell&#x27;array fallisce con errore HTTP503 &quot;Nessun nuovo gruppo di frame sincronizzato&quot;** → L&#x27;array non ha pubblicato alcun nuovo gruppo di frame durante il periodo di attesa. Verificare `arr.status()`: `stalled: True` (o `streaming: False`) con `actual_fps`: se `actual_fps` è pari a 0, significa che il ciclo di acquisizione ha smesso di pubblicare — disconnettere e ricollegare l’array. Se `stalled` è False e `actual_fps` è superiore a 0, riprovare. Subito dopo `connect_array()`, una cattura è già in attesa da un massimo di 20 s per il primo gruppo.
- **`status()` mostra `link_speed_mbps: 4294`** → Windows non è riuscito a leggere la velocità dell’adattatore; si tratta di un comportamento tipico degli adattatori Ethernet USB e non costituisce un errore. Vedi [Frequenze di frame previste](#expected-frame-rates).
- **`update_firmware()` fallisce con `device never connected back on TCP <port>`** → Un firewall ha bloccato la connessione del DAQ-E a questo computer. Il programma di installazione di Windows aggiunge la regola (vedere [Impostazioni del dispositivo DAQ-E](#daq-e-device-settings)); se si utilizza un altro firewall, consentire il traffico TCP in entrata verso il backend dalla sottorete del sensore.
- **Riflettanza rifiutata** → Per ottenere la riflettanza in scala assoluta, è necessario associare un DAQ attivo alla telecamera (o all’array). Effettuare l’associazione tramite l’interfaccia grafica oppure utilizzare `processing="radiance"` (W/m²/sr/nm), che non richiede un sensore abbinato.
- **L&#x27;acquisizione con `smart=True` richiede più tempo del previsto** → La convergenza AE dipende dalla dinamica della scena. Con `ArrayHandle.capture_smart()`, allargare `exposure_tolerance_pct`, ridurre `stability_window_s` o diminuire `settle_timeout_s` per ottenere un trigger più veloce e meno stabile.

---

## Vedi anche

- [Guida di riferimento CLI](cli-reference.md) — ogni sottocomando CLI corrisponde a una chiamata SDK.
- [Sensori di luce DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) nel manuale di Chloros — i sensori, i loro limiti e l&#x27;intervallo calibrato, la registrazione, il collegamento in rete DAQ-E.
- [Telecamere LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) nel manuale di Chloros — collegamento, impostazioni delle telecamere, modalità di acquisizione, array.
- Documentazione online: `https://mapir.gitbook.io/chloros/api-python-sdk`
