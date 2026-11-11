# Impostazioni delle telecamere

La scheda **Telecamere** è la superficie di controllo in tempo reale di Chloros

per le telecamere LATTICE: un&#x27;area principale che mostra ogni telecamera collegata sotto forma di riquadro live e una barra laterale che scorre tra tre pagine — l’**elenco delle telecamere**, un**pannello delle impostazioni**(impostazioni per singola telecamera, per array o di acquisizione — una alla volta) e il**Calcolatore dell’indice**. Questa pagina descrive ogni comando presente nell’elenco delle telecamere, nel pannello delle impostazioni per singola telecamera e nel pannello delle impostazioni per array. Le modalità di acquisizione, la selezione del tipo di esportazione e il flusso “Acquisizione totale” sono descritti nella pagina correlata [Impostazioni e modalità di acquisizione](capture.md).

La scheda “Telecamere” appare nella barra laterale una volta che il backend di Chloros

è pronto. Tutti i controlli riportati di seguito comunicano con il backend locale tramite `127.0.0.1:5000`; le modifiche vengono applicate immediatamente alla telecamera in diretta, salvo diversa indicazione.

## Tipi di telecamere utilizzati in questa pagina

I controlli si mostrano o si nascondono a seconda del tipo di telecamera selezionata. Il manuale utilizza questi termini in tutto il testo:

| Termine | Significato | Canali del filtro |
| --- | --- | --- |
| **Telecamera RGB** | LATTICE M3C con filtro FRGB (il modello contiene `-FRGB`) | Red

/ Green

/ Blue

|
| **Multispettrale Bayer** | LATTICE M3C con FRGN, FOCN o FNGB | FRGN: Red

/ Green

/ NIR

· FOCN: Orange

/ Cyan

/ NIR

· FNGB: NIR

/ Green

/ Blue

|
| **Mono (M3M)** | LATTICE M3M — un filtro a banda stretta, una banda calibrata | Banda singola |
| ****Membro dell’array** | Una telecamera collegata come parte di un array sincronizzato (visualizzazione combinata o separata) | In base al proprio filtro |

Le telecamere RGB

sottoposte a elaborazione fotometrica (bilanciamento del bianco, profili colore, gamma); le telecamere multispettrali e mono vengono sottoposte alla catena radiometrica e saltano i controlli fotometrici. I membri dell’array gestiscono le impostazioni a livello di flusso (formato pixel, risoluzione, binning, trigger, frequenza dei fotogrammi) all’array — tali righe diventano di sola lettura nel pannello specifico per ciascuna telecamera e vengono spostate nel pannello delle impostazioni dell’array.

## L’area principale del feed



<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

In assenza di telecamere collegate, l’area del feed mostra una schermata iniziale con il messaggio **“Collega una telecamera per iniziare”**e due pulsanti:**Collega telecamera**(verde, apre la finestra di dialogo di connessione per una singola telecamera) e**Connetti array** (blu, apre la finestra di dialogo di connessione per l’array). Le finestre di dialogo di connessione sono documentate in [Connessione delle telecamere](connecting.md); i concetti relativi agli array (sincronizzazione, livelli, larghezza di banda) in [Array multicamera](arrays.md). Quando si apre un progetto salvato che contiene telecamere, la schermata iniziale mostra invece un indicatore di caricamento con il messaggio &quot;Riapertura di N telecamere salvate…&quot; mentre Chloros

ripristina i flussi dall’ultima sessione.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Barra superiore

| Controllo | Funzione |
| --- | --- |
| **Alternanza modalità di visualizzazione**| Passa dalla**vista a griglia**(tutte le tessere come celle) alla**vista a elenco**(righe a larghezza intera: ogni array combinato come una riga composita, ogni membro di un array**separato** come riga a sé stante con il proprio feed e la propria tabella dei target di calibrazione, quindi UNA telecamera autonoma — quella attiva — sotto). Suggerimenti: &quot;Passa alla visualizzazione a griglia&quot; / &quot;Passa alla visualizzazione a elenco&quot;. |
| **Blocco griglia**(lucchetto) |**Bloccato** per impostazione predefinita — riquadri fissati in posizione. Sbloccare per trascinare e riordinare i riquadri in qualsiasi posizione (gli spazi vuoti vengono mantenuti). La griglia si blocca automaticamente ogni volta che si connette una nuova telecamera. Suggerimenti: &quot;Sblocca griglia (abilita trascinamento delle tessere)&quot; / &quot;Blocca griglia (fissa le tessere in posizione)&quot;. |
| Cursore **Zoom feed** | Dimensione delle tessere, da 60 px fino alla larghezza totale del contenitore. Le celle mantengono un rapporto di aspetto 4:3. Se la larghezza delle celle è inferiore a 200 px, le sovrapposizioni del nome e degli fps vengono nascoste per mantenere pulito il riquadro. |

### Riquadri del feed

Ogni telecamera genera un riquadro composito in tempo reale; una telecamera può inoltre mostrare tre riquadri in scala di grigi **divisi per canale** (vedi [Divisioni dei canali](#display-overlays-drawn-over-the-live-feed)), mentre gli array visualizzano una tessera combinata. La tessera attiva presenta un anello di selezione nel colore della telecamera (o dell’array).

Passando il mouse su una tessera viene visualizzato un pulsante di chiusura **X**:

* Chiudere un riquadro **composito** mentre le sue divisioni per canale rimangono visibili nasconde semplicemente il riquadro composito.
* Chiudere l’**ultimo riquadro visibile di una telecamera autonoma** disconnette quella telecamera.
* **I riquadri di divisione dei membri di un array combinato non disconnettono mai** la telecamera: la nascondono soltanto.

Con la griglia sbloccata, trascinare qualsiasi riquadro in qualsiasi slot; il layout viene salvato con il progetto.

## Barra laterale — elenco

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

delle telecamere La prima pagina della barra laterale elenca tutte le telecamere e gli array collegati:

* **Connetti telecamera**(verde) /**Connetti array** (blu, mostra &quot;Rilevamento in corso...&quot; durante la scansione). Entrambe disabilitate mentre è aperta una finestra di dialogo di connessione.
* **Acquisisci tutto** (rosso) — acquisisce tutte le telecamere elencate con i tipi di esportazione scelti nelle Impostazioni di acquisizione. Richiede un progetto aperto. Documentato in modo completo in [Impostazioni e modalità di acquisizione](capture.md).
* **Ingranaggio delle impostazioni di acquisizione** (accanto a Acquisisci tutto) — apre il [pannello delle impostazioni di acquisizione](capture.md#the-capture-settings-pane). Disabilitato in assenza di un progetto o durante l’acquisizione.

### Righe delle telecamere

Ogni riga di telecamera mostra un bordo con codice colore (il colore personalizzato della telecamera), un’etichetta “CAM” — con una **M**blu (master) o una**S** verde (slave) per i membri dell’array — e il nome visualizzato. Il nome predefinito è `LATTICE-MODEL (serial)`; è possibile rinominarlo dal pannello delle impostazioni specifiche per ciascuna telecamera. Pulsanti della riga:

| Pulsante | Effetto |
| --- | --- |
| **Occhio**| Attiva/disattiva la visibilità. Le telecamere nascoste escono dalla griglia e vengono**escluse dalla funzione &quot;Cattura tutto&quot;**. |
| **Ingranaggio** | Apri il pannello delle impostazioni specifiche per ciascuna telecamera (sezione successiva). |
| **Pausa / Riproduci**| Blocca l’anteprima in tempo reale**solo sul lato di visualizzazione** — l’acquisizione sul backend continua a funzionare. Le telecamere in pausa non possono acquisire. |
| **X** | Disconnetti. L’interfaccia utente si aggiorna immediatamente (in condizioni ottimali); la disconnessione effettiva da parte del backend può richiedere dai 10 ai 30 s per completarsi. |

### Righe dell’array

Una riga dell’array mostra un badge “ARRAY” nel colore dell’array, il nome dell’array (rinominabile nelle impostazioni dell’array) e un pulsante **DAQ · on/off**—**on** quando il sensore di luce a livello di array è impostato *oppure* qualsiasi membro dispone di un sensore per singola telecamera; il suo tooltip elenca esattamente quale sensore alimenta cosa. Le telecamere dei membri sono elencate sotto, indentate, con le loro righe dedicate. Pulsanti della riga dell’array: **occhio**(nasconde/mostra TUTTI i membri contemporaneamente),**ingranaggio**(pannello delle impostazioni dell’array),**X**(disconnette l’intero array).

Lo stato del sensore di luce (DLS) utilizzato nelle righe dell’array e nel pannello delle impostazioni dell’array presenta quattro stati:**off**,**in attesa**(ancora nessun spettro),**attivo**(è arrivato uno spettro negli ultimi 3 s) e**obsoleto** — nessun nuovo spettro negli ultimi 3 s, ma l’ultima lettura viene *ancora utilizzata* (le letture DAQ non scadono mai nel percorso di acquisizione).

Finché non si modifica l’ordine, l’elenco mostra prima gli array, poi le telecamere autonome, con le telecamere (e i membri di ciascun array) in **ordine di telecamera**: M3C (Bayer) prima di M3M (mono); tra M3C, FRGB, FRGN, FOCN, poi FNGB; M3M in base alla lunghezza d’onda del filtro, dalla più bassa in poi. Il layout predefinito della griglia segue lo stesso ordine. È possibile trascinare le telecamere autonome e interi gruppi di array l’uno sopra l’altro nella barra laterale per riordinare l’elenco; i membri degli array non sono trascinabili in modo indipendente.

## Pannello delle impostazioni per singola telecamera

Si apre cliccando sull’**ingranaggio** in una riga della telecamera. Il pannello scorre sopra l’elenco delle telecamere.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Intestazione**: il**campione di colore**della telecamera (fare clic per aprire un selettore di colore nativo — imposta il colore del bordo della barra laterale e dell’anello di selezione delle tessere), il**nome**con un pulsante a forma di matita**Rinomina**(salvando un nome vuoto si ripristina il valore predefinito `MODEL (serial)`) e**×** per chiudere.

### Istogramma in tempo reale

Nella parte superiore del riquadro è presente un istogramma di luminanza in tempo reale calcolato dall’anteprima JPEG

a circa 8 Hz. La media è ponderata secondo lo schema Bayer — (R+2G+B)/4 — per corrispondere alla misurazione AE propria della fotocamera.

* **Linea tratteggiata Orange**= il target AE. **Trascinarla orizzontalmente per ridefinire il target** — al rilascio viene inviato un comando, e il trascinamento commuta la modalità del target AE su Manuale.
* **Linea continua Green** = la luminanza media effettiva (ciò che l’AE sta attualmente fornendo).
* **Pulsante RGB** (in alto a destra): attiva/disattiva gli istogrammi sovrapposti per banda, colorati in base al filtro della fotocamera (ad es. su FRGN: grigio NIR

, verde, rosso). Sulle fotocamere mono (M3M) il pulsante riporta la scritta &quot;MONO&quot; ed è disabilitato — la modalità mono mostra sempre l’istogramma di luminanza a banda singola.
* Le etichette dell’asse X seguono la profondità di bit del sensore del formato pixel corrente: 0..255, 0..1023, 0..4095 o 0..65535.

### Righe

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

con informazioni sulla fotocamera

| Riga | Comportamento |
| --- | --- |
| **Modello** | Solo lettura (ad es. `LATT-M3C-L87-FRGN`). |
| **Calibrazione radiometrica** | Badge Green**&quot;Attivo&quot;**con una didascalia che mostra il livello di calibrazione, l’hash, la data di calibrazione e l’elenco delle bande, caricati dal pacchetto di calibrazione della fotocamera (vedi [Calibrazione radiometrica di fabbrica](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Nascoste per le telecamere RGB

** — queste dispongono di una calibrazione fotometrica del bilanciamento del bianco, non della radianza per banda. |
| **Rapporto di calibrazione**| Pulsante**Scarica** — apre il certificato di calibrazione NIST specifico per il numero di serie della telecamera in formato PDF nel visualizzatore del sistema operativo. Se il certificato non è ancora memorizzato nella cache, Chloros

mostra invece un suggerimento. |
| **Numero di serie** | Solo lettura. |
| **Firmware**| Mostra la versione corrente, quindi individua la versione disponibile per questo modello (memorizzata nella cache per modello — un array di N telecamere effettua una verifica sul server una sola volta). Stati: &quot;Verifica in corso…&quot; → pulsante**&quot;Aggiorna a X&quot;**→ &quot;Aggiornamento in corso…&quot; → &quot;Aggiornato da A a B&quot; / &quot;Operazione fallita: …&quot; / &quot;Saltato: …&quot; / verde**&quot;Aggiornato&quot;**. Descrizione del pulsante di aggiornamento: &quot;Ripristino delle impostazioni di fabbrica + flash + riprogrammazione di UserSet1. ~2–3 minuti; non scollegare.&quot; |
| **IP** | Solo lettura. |
| **Temperatura** | Solo lettura, aggiornata ogni 3 s. Diventa arancione a ≥65 °C e rossa con un ⚠ a ≥75 °C. |
| **Target di calibrazione** casella di controllo | Abilita il rilevamento del bersaglio di riflettanza ArUco con una tabella di convalida NDVI

per pannello sotto il feed in tempo reale (vista elenco) — vedi [La tabella del bersaglio di calibrazione](#the-calibration-target-table). Solo per la sessione — si apre sempre disattivato. |
| Menu a tendina **Sensore di luce** | Associa un sensore di luce DAQ (DAQ-E/M/U, dall’elenco della scheda Sensori di luce) a questa telecamera per la correzione dell’illuminazione della luce discendente (DLS) e l’esposizione automatica predittiva. &quot;&quot;Nessuno&quot; cancella l’associazione. Se non sono collegati sensori, il menu a tendina mostra &quot;(nessun sensore collegato — aprire la scheda DAQ)&quot;. L’associazione viene salvata con il progetto. |

#### La tabella del bersaglio di calibrazione

La tabella è **solo a scopo di lettura di convalida** (“solo convalida — non modifica la calibrazione”). L’intestazione riporta il numero di serie della fotocamera, il filtro, il numero del bersaglio e la dimensione del marcatore in pixel; presenta una riga per ogni pannello:

* **Pannello**, seguito da una colonna per ciascuna banda con il valore del pannello — una telecamera Bayer mostra le sue tre bande per nome e lunghezza d’onda (ad es. FRGN: Red

o 660 nm / Green

e 550 nm / NIR

850 nm); una **telecamera mono (M3M) mostra la sua singola banda** (ad es. `F750`) e non presenta colonne NDVI

.
* **Sat** — la percentuale di pixel del pannello che risultano saturi (ambra a partire dall’1%, rosso a partire dal 5%).
* **NDVI

Meas**,**NDVI

Theo** (NDVI

di riflettanza di riferimento del pannello) e **Δ**— solo per telecamere multispettrali Bayer. Un piccolo simbolo dopo “NDVI

Meas” indica da cosa è stato calcolato l’NDVI

misurato:**ρ**(riflettanza — è disponibile una lettura del sensore di luce),**L**(radianza — nessuna lettura del sensore di luce) o**DN** (valori grezzi della fotocamera). Solo un NDVI**ρ** è comparabile con l’NDVI

Theo: per L e DN la colonna Δ è disattivata e una nota lo specifica (&quot;Nessuna lettura del sensore di luce: l’NDVI

deriva dalla radianza, quindi Δ non è un controllo di calibrazione&quot; / &quot;L’NDVI

deriva dai valori grezzi della fotocamera, quindi Δ non è un controllo di calibrazione&quot;). Con ρ, Δ è verde se inferiore a 0,02, giallo se inferiore a 0,05, rosso in tutti gli altri casi.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Esposizione e guadagno

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Tutti gli input numerici qui utilizzano rotelle di regolazione con accelerazione al mantenimento: tocco = ±1, mantenimento &gt;1,5 s = ±10, mantenimento &gt;3 s = ±100. Il valore viene inviato alla fotocamera quando si rilascia.

| Controllo | Intervallo / opzioni | Predefinito | Si applica a | Cosa fa |
| --- | --- | --- | --- | --- |
| **Esposizione (us)**| Min/max in tempo reale della fotocamera | Auto | Tutto | Tempo di esposizione in microsecondi, con un . Auto = esposizione automatica continua da parte della telecamera. Impostando un’esposizione manuale, il**Guadagno** rimane su Auto (se lo era), consentendo di utilizzare l’intera gamma di guadagno della telecamera, finché non si modifica il guadagno o lo si imposta su Manuale. |
| **Guadagno (dB)**| Min/max in tempo reale della telecamera (ad es. fino a 48 dB) | Manuale (disattivato) | Tutti | Guadagno analogico/digitale con il proprio selettore**Auto/Manuale**. |
| **Luminosità target AE**| 0–255 | 80, modalità**Auto**| Tutti (modificabile quando AE o guadagno automatico sono attivi) | La luminosità a cui punta l’AE. In**Auto**(impostazione predefinita) un controller interno basato sull’istogramma seleziona autonomamente il valore target, mantenendo l’esposizione al 60–75 % del massimo del sensore. Digitando un valore o trascinando la linea arancione dell’istogramma si passa alla modalità**Manuale**. |
| **Smoothing AE** | 0,5–40, incrementi di 0,1 | 8,0 | Tutti | Smorzamento AE. Descrizione: &quot;Più basso = l’AE reagisce più velocemente (può pulsare ad alti fps). Più alto = più fluido / più lento.” Valori molto inferiori a quelli predefiniti possono causare pulsazioni dell’AE e destabilizzare lo streaming ad alti frame rate; 8,0 è il valore predefinito stabile. |
| **Area di interesse AE**| Casella di controllo “Abilita” + pulsante**Punta**| Disattivato | Tutti | Quando è attivata, AE misura solo l’area verde tratteggiata anziché l’intero fotogramma.**Mira** attiva la funzione “clicca per posizionare” sul feed in diretta: un clic centra un’area al 30% del fotogramma; clicca e trascina per tracciare un rettangolo personalizzato (minimo 5% × 5%). La funzione *Aim* si disattiva automaticamente dopo un posizionamento. L’area viene mappata sulle coordinate native della telecamera in base a qualsiasi rotazione o specchiatura impostata e viene salvata con il progetto. |
| **Velocità di regolazione AE** | 0,1–5, passo 0,1 | 1,0 | Solo per membri dell’array | La velocità con cui il bersaglio AE automatico tiene traccia delle variazioni di luminosità della scena; 1,0× effettua un nuovo controllo ogni 2,5 s. |
| **Protezione delle alte luci** | Rigorosa (1 %) / Normale (5 %) / Approssimativa (15 %) | Rigorosa | Solo per le fotocamere che espongono l’impostazione | Quanta parte del fotogramma può essere clippata in bianco prima che l’AE scurisca l’immagine. |

{% hint style="info" %}
**Requisiti di illuminazione per le fotocamere multispettrali Bayer (RGN

/ OCN

/ NGB

):** la scena deve avere luce sufficiente in tutti e tre i canali, altrimenti la calibrazione non funziona correttamente — un’unica esposizione del sensore copre tutti e tre gli spettri. Utilizza un sensore di luce DAQ per misurare la luminosità, oppure passa alla modalità interamente mono (M3M) in modo che ogni banda abbia la propria esposizione. Se un’acquisizione viola questo requisito, Chloros

lo rileva e ti avvisa (notifica di limitazione della separazione dei colori). In tempo reale, una casella di suddivisione dei canali mostra **&quot;Dati della banda a bassa affidabilità · N%&quot;** quando l’unmix ha limitato quella banda sul 25% o più dei suoi pixel.
{% endhint %}

{% hint style="info" %}
**Come l’esposizione automatica gestisce l’esposizione e il guadagno (1.3.0).** Con il guadagno in modalità automatica, Chloros

mantiene il guadagno a 0 dB finché l’esposizione non raggiunge il valore massimo consentito (il limite superiore dell’esposizione automatica, il periodo di trigger di un array o il limite massimo di esposizione del DAQ); solo allora il guadagno aumenta. In un array, la frequenza di trigger ha la priorità: l’esposizione automatica riempie il periodo stabilito dalla frequenza, quindi aumenta il guadagno; di conseguenza, una scena che si oscura non riduce più la frequenza di trigger in modalità automatica (nella versione 1.2.0 invece lo faceva). Un’esposizione **manuale**su un elemento dell’array di durata superiore al periodo di trigger rallenta il trigger dell’array per adattarlo; la frequenza automatica viene ripristinata non appena tutti gli elementi che l’avevano rallentata tornano all’esposizione automatica. Con**Esposizione**e**Luminosità target AE** entrambe su Auto e un sensore di luce DAQ che fornisce una lettura, l’esposizione di ciascuna telecamera viene mantenuta una volta che l’esposizione automatica si è stabilizzata, purché la luce misurata rimanga entro circa ±25 %; una variazione di luce più ampia e prolungata, un riavvio delle impostazioni (ad esempio una modifica della ROI) o una modifica dell’esposizione da parte dell’utente la riattiva.
{% endhint %}

{% hint style="info" %}
**Come funziona l’esposizione automatica (1.3.0).**

* Un’esposizione manuale lascia il guadagno su “Auto” finché non si modifica il guadagno.
* Le piccole luci non scuriscono la scena: viene ignorata una luce speculare fino all’1% dell’inquadratura e, nei pannelli di un bersaglio di calibrazione noto, fino allo 0,5% di un pannello.
* Il limite massimo di esposizione del DAQ si applica solo quando un bersaglio di calibrazione è visibile; se non si vede alcun pannello, la fotocamera espone in base alla scena.
* L’esposizione rimane stabile in condizioni di luce costante e si assesta entro pochi secondi da un cambiamento significativo. Un clipping non limita più l’esposizione in modo permanente e una modifica dell’area di interesse (ROI) ripristina l’esposizione.
* Il target di luminosità non supera il, quindi un ritorno di luce non provoca riflessi nell’inquadratura.
* Una telecamera la cui esposizione automatica smette di rispondere in pieno sole viene mantenuta al livello corrispondente alla luce in cui si trova, rilevata dal sensore di luce o, in assenza di sensore collegato, dai fotogrammi della telecamera stessa, e viene rilasciata quando la luce cambia. Un sensore di luce oscurato o coperto non interferisce con questo processo.
{% endhint %}

### Formato pixel e risoluzione

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**I membri dell’array** mostrano le righe di sola lettura “Corrente” (formato + LxH) e “Binning” con la nota “Impostato nelle impostazioni dell’array”: un riavvio dello stream su un membro interromperebbe la sincronizzazione, quindi queste impostazioni sono gestite nel [pannello delle impostazioni dell’array](#array-settings-pane).

Le **telecamere autonome** presentano:

| Controllo | Opzioni | Funzione |
| --- | --- | --- |
| **Formato pixel** | 8 bit / 10 bit / 12 bit / 16 bit | Profondità di bit del sensore; la famiglia di formati è quella del sensore. L’impostazione predefinita è 12 bit; un progetto salva l’impostazione scelta, compresi gli 8 bit, e si riconnette con tale impostazione. |
| **Risoluzione** | Intera / Metà / Un quarto | Relativa al binning corrente: Piena = 2048/N × 1536/N per il binning N×N. |
| **Binning** | 1x1 (nessuno) / 2x2 / 4x4 | Binning hardware N×N — valori più alti riducono la risoluzione ma aumentano il rapporto segnale-rumore (SNR) e la frequenza dei fotogrammi. Modificandolo si riavvia lo streaming e si reimposta qualsiasi ROI al nuovo campo visivo completo. |
| **Attuale** | di sola lettura | Le dimensioni effettive LxA e l’offset (x, y) attualmente in vigore. |

### Anteprima in tempo reale

Tutto in questa sezione riguarda **solo la visualizzazione**— modifica ciò che si vede nel feed in diretta, mentre le acquisizioni salvate rimangono lineari e inalterate — con un’eccezione:**Vignetta** è radiometrica e influisce anche sulle esportazioni (come indicato di seguito).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Controllo | Intervallo / opzioni | Predefinito | Si applica a | Cosa fa |
| --- | --- | --- | --- | --- |
| **Risoluzione di rendering** | 360p (più veloce) / 480p / 720p / 1080p / Risoluzione nativa del sensore (più lenta) | 720p | Tutti | L’altezza alla quale il backend esegue la catena di anteprima radiometrica. Un valore più basso garantisce una maggiore frequenza dei fotogrammi senza modificare il campo visivo. |
| **Indice**| Casella di controllo “Abilita” + ingranaggio | Disattivato | Solo multispettrale Bayer,**non** per i membri con array combinato | Anteprima in tempo reale dell’indice di vegetazione. L’icona a forma di ingranaggio apre il [Calcolatore di indici](#index-calculator-pane) condiviso, precaricato con le bande naturali del filtro della fotocamera (ad es. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). L’espressione personalizzata più la LUT (on/off, livello predefinito 7 stop, min predefinito 0,2, max predefinito 1) viene calcolata su ogni fotogramma di anteprima. I membri dell’array combinato nascondono questa riga — l’array possiede un unico indice condiviso. |
| **Bilanciamento del bianco** | Off / Una volta / Continuo + un pulsante di nuova acquisizione | Continuo | Solo RGB

| Bilanciamento del bianco in tempo reale. Il pulsante di aggiornamento acquisisce nuovamente il bilanciamento del bianco dallo spettro DLS corrente (disabilitato quando la modalità è disattivata). |
| **Gamma** | On / Off | On | Solo RGB

| Visualizza la gamma (LUT γ = 2,2) sull’anteprima in tempo reale. Le acquisizioni salvate rimangono lineari. |
| **Riduzione del rumore** | Casella di controllo + intensità 0–100 | Disattivato / 50 | Tutti (per ogni telecamera, anche all’interno degli array) | Filtro bilaterale sull’anteprima in tempo reale. Valori più alti = immagini più uniformi ma con dettagli meno nitidi. |
| **Nitidezza** | Casella di controllo + intensità 0–100 | Disattivato / 30 | Tutti | Maschera di contrasto nell’anteprima in tempo reale, applicata per ultima. Può amplificare il rumore. Solo in anteprima. |
| **Vignettatura**| Casella di selezione + intensità 0–100 | Disattivato / 0 | Tutti | Rimozione manuale della vignettatura residua (schiarisce gli angoli), sovrapposta alla stima della vignettatura intelligente dell’array.**Radiometrico — influisce sull’anteprima in tempo reale E sulle esportazioni**, a differenza di Denoise/Nitidezza. |
| **Profilo colore** | Raw / Lineare / Naturale / Migliorato / Temperatura personalizzata | Naturale | Solo RGB

| Vedi sotto. |
| **Temperatura di colore** | 2000–10000 K, incrementi di 100 | 5500 K | Solo RGB

, solo profilo Temperatura personalizzata | Fissa il bilanciamento del bianco su una temperatura di colore correlata fissa (l’input DLS viene ignorato). L’ultimo valore in Kelvin selezionato viene memorizzato anche quando si cambia profilo. |
| **Alte luci** | Da −100 a 0, con incrementi di 1 | 0 (disattivato) | Solo RGB

| Novità della versione 1.3.0. Scurisce le aree più luminose e più bianche dell’immagine (0 = invariato, −100 = massimo). Si applica all’anteprima in tempo reale **e alle acquisizioni a livello di schermo (Anteprima)**, che registrano l’impostazione (`Chloros:DisplayHighlights`); le esportazioni di radianza e riflettanza non ne sono mai influenzate. Non è in grado di recuperare i dati del sensore che hanno subito clipping — per farlo, ridurre l’esposizione. La sovrapposizione zebra segue l’immagine scurita. Disabilitato con il profilo colore Raw (&quot;Non applicato con il profilo colore Raw.&quot;). |
| **Saturazione** | 0–200 (100 = neutro) | 100 | Solo RGB

| Saturazione HSV nell’anteprima in tempo reale. |
| **Contrasto** | 0–200 (100 = neutro) | 100 | Solo RGB

| Contrasto lineare intorno al grigio medio nell’anteprima in tempo reale. |
| **Rifletti H / Rifletti V** | Caselle di controllo | Disattivato | Tutti | Capovolge l’anteprima orizzontalmente / verticalmente. |
| **Rotazione**| 0° / 90° / 180° / 270° | 0° | Tutti | Ruota l’anteprima. L’orientamento viene applicato alla fine della catena di anteprima del backend.**Acquisizioni salvate (1.3.0):**i prodotti Anteprima, Radianza, Riflettanza e Indice di una fotocamera autonoma vengono ruotati tramite Specchio/Rotazione, mentre**i file Raw e Debayered mantengono l’orientamento nativo della fotocamera**; ogni file di acquisizione registra l’impostazione nel proprio XMP, quindi l’elaborazione successiva di un file raw la applica allo stesso modo.**I file membri dell’array mantengono l’orientamento nativo** (l’impostazione viene registrata, non applicata) e le viste composite dell’array la ignorano. |**Semantica dei profili colore** (telecamere RGB

):

* **Raw** — bypassa completamente la catena di elaborazione.
* **Lineare** — segnale scuro + flat field + bilanciamento del bianco; nessuna matrice colore, nessuna gamma.
* **Naturale** *(impostazione predefinita)* — lineare più la matrice di correzione del colore misurata e una curva di tono adattiva alla scena.
* **Migliorata**— Naturale più vivacità e contrasto locale CLAHE. Il costo aggiuntivo si applica**solo all’anteprima in tempo reale** — le acquisizioni salvate ottengono sempre la finitura completa indipendentemente dal profilo.
* **Temperatura personalizzata** — “Naturale” con il bilanciamento del bianco fissato al valore in Kelvin scelto.

{% hint style="warning" %}
Per le modalità *Naturale*, *Migliorata* e *Temperatura personalizzata*, il pannello mostra una nota sulla tonalità: i fotogrammi vengono schiariti in base alla propria scena, quindi le immagini *di visualizzazione* salvate non sono comparabili tra un fotogramma e l’altro. **Esporta la radianza o la riflettanza per le misurazioni.**
{% endhint %}

### Sovrapposizioni di visualizzazione (disegnate sul feed in diretta)

Queste sono solo a livello di front-end — vengono disegnate sul video, senza mai interferire con lo streaming o le acquisizioni.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Sovrapposizione | Controlli | Predefinito | Funzione |
| --- | --- | --- | --- |
| **Zebra** | Casella di controllo + soglia 200–255 | Disattivato / 250 | Strisce diagonali magenta sui pixel tagliati. |
| **Mirino** | Casella di controllo | Disattivato | Segno al centro del fotogramma. |
| **Griglia** | Disattivato / 3 × 3 / 9 × 9 | Disattivato | Griglia di composizione. |
| **Istogramma** | Casella di selezione + larghezza 0,10–0,90 del fotogramma | Disattivato / 0,25 | Una striscia di istogramma sul feed. |
| **Focus Peak** | Casella di controllo + soglia 20–200 + campione di colore | Disattivato / 80 / `#ff5722` | Evidenziazione dei bordi con filtro di Sobel per la messa a fuoco. |
| **Suddivisioni dei canali** | &quot;Mostra suddivisioni (Red

/ Green

/ NIR

)&quot; / Pulsante &quot;Nascondi divisioni&quot; | Nascosto | Aggiunge tre riquadri indipendenti in scala di grigi per ciascun canale accanto all’immagine composita (l’etichetta del pulsante segue i canali del filtro della fotocamera). Ogni riquadro diviso è trascinabile e condivide il colore del bordo della fotocamera. Non disponibile sulle fotocamere monocromatiche. Salvato con il progetto. |

### Misuratore spot

* Casella di controllo **Clicca per campionare**: clicca sul feed in tempo reale per campionare un singolo pixel (contrassegnato da un reticolo a croce), oppure clicca e trascina su un’area per ottenere una media dei pixel.**Cancella**elimina il campione e il reticolo. Si esclude a vicenda con la modalità**Aim** di AE-ROI.
* **Menu a tendina**Mostra**:**Raw (profondità di bit)**— valori digitali nativi alla profondità di bit del sensore (ad es. 12 bit → 0..4095) — oppure**Display (8 bit)**(impostazione predefinita). Quando è attivo un indice in tempo reale, l’opzione**Display** mostra invece il valore dell’indice calcolato (ad es. NDVI

).
* Il pannello di lettura elenca le coordinate dei pixel, le dimensioni del fotogramma, il formato dei pixel, la profondità di bit e una tabella dei canali (Chan / Valore / %) con le etichette delle bande e le lunghezze d’onda; le coppie di verde Bayer vengono mediate; i campioni di regione mostrano &quot;Media N px&quot;.

Lo stato dell’esposimetro spot è valido solo per la sessione.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Esposizione automatica predittiva (basata su DLS)

Questa sezione appare solo quando **è collegato almeno un sensore di luce DAQ** — il solutore necessita di uno spettro discendente in tempo reale per funzionare.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Controllo | Intervallo | Predefinito | Funzione |
| --- | --- | --- | --- |
| **Abilita** | Casella di controllo | On (telecamere autonome) | Un risolutore in forma chiusa utilizza lo spettro DLS insieme agli scalari del pacchetto di calibrazione della fotocamera per portare la banda più luminosa vicino alla saturazione, mantenendo al contempo la banda più scura al di sopra della soglia minima del rapporto segnale-rumore (SNR) — una singola scrittura dell’esposizione per ogni soluzione, senza ciclo di stabilizzazione. Progettato per time-lapse alimentati a energia solare in cui ogni scatto deve essere correttamente esposto. Il backend ricorre silenziosamente all’esposizione automatica reattiva ogni volta che la lettura DLS è obsoleta/mancante o il pacchetto di calibrazione non è caricato. |
| **Smussamento (α)** | 0,05–1,0, passo 0,05 | 0,3 | Smussamento delle soluzioni predittive successive (valore più basso = smussamento maggiore). |
| **Riflettanza della scena**| Pulsante**Ricalibra ρ** | — | Ricalcola il fattore di riflettanza della scena utilizzato dal risolutore. |

{% hint style="info" %}
**La connessione in array disattiva l’AE predittivo per impostazione predefinita** — per gli array, l’AE intelligente di Chloros

e l’esposizione automatica lato fotocamera gestiscono l’esposizione (con protezione dalla saturazione) e la stima della riflettanza di una singola scenanon è affidabile in scene miste. È possibile riattivarla per singola telecamera da qui se si desidera specificatamente un’esposizione radiometrica basata sul DLS.
{% endhint %}

**Limite massimo di esposizione basato sul DAQ ed AE con fissaggio sull’incidente.** Indipendentemente dalla casella di controllo sopra indicata, quando un sensore di luce DAQ è assegnato a una telecamera RGB

, Chloros

calcola — a partire dall’irradianza assoluta discendente misurata — l’esposizione massima × guadagno al quale una superficie con riflettanza del 100% rimane al di sotto del clipping, e la applica come **limite massimo**per l’esposizione automatica. A partire dalla versione 1.3.0 il limite massimo si applica solo mentre un bersaglio di calibrazione si trova nel campo visivo di quella telecamera (rilevato nel suo fotogramma negli ultimi 10 s); in assenza di bersaglio nel campo visivo, entra invece in funzione la normale esposizione automatica della telecamera, con la propria protezione dalle alte luci, in modo che il resto della scena non risulti più scuro del necessario. Mentre il limite massimo è attivo, la telecamera è**fissata alla luce incidente**: funziona in anello aperto all’esposizione misurata in base alla luce incidente con guadagno a 0 dB — l’esposizione segue la luce misurata, non il contenuto della scena. Poiché il limite massimo può solo ridurre l’esposizione, non può di per sé causare clipping. A partire dalla versione 1.3.0 il limite massimo viene calcolato utilizzando la costante di esposizione propria della telecamera, ricavata dal suo pacchetto di calibrazione (un valore medio della flotta solo quando il pacchetto non ne contiene uno). Il limite massimo si disattiva automaticamente — e riprende la normale esposizione automatica (AE) della scena — ogni volta che non è visibile alcun bersaglio di calibrazione, la lettura DAQ è mancante, obsoleta (&gt;30 s), o scura, oppure se ≥15 % del fotogramma va in clipping all’esposizione fissata (il che significa che il sensore e la telecamera rilevano un’illuminazione diversa). Non è presente alcun interruttore nell’interfaccia grafica; si tratta di un comportamento standard ogni volta che una telecamera RGB

ha un DAQ associato.

### Acquisizione e trigger

I membri<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

dell’array mostrano inoltre le righe di sola lettura **Ruolo**(Master in blu / Slave in verde),**Linea di sincronizzazione**e**Pari**.

| Controllo | Opzioni | Predefinito | Note |
| --- | --- | --- | --- |
| **Modalità di trigger** | Off / On | On | Disabilitata per gli elementi dell’array (l’array gestisce il trigger). |
| ****Sorgente di trigger** | Software / Linea 0 (M8) / Linea 1 / Linea 2 | Linea 0 | Nascosta quando la modalità di trigger è disattivata; disabilitata per i membri dell’array. La Linea 0 è l’ingresso di trigger esterno optoisolato M8. |
| **Frequenza fotogrammi**| Auto / Manuale + valore | Auto |**Auto**: il limite di frame rate della telecamera è disattivato — l’esposizione determina gli fps e il riquadro mostra la frequenza effettiva in tempo reale.**Manuale**: si limita il numero di fps tramite un cursore (da 1 fino al massimo consentito dalla larghezza di banda), basato sulla frequenza effettiva corrente. I membri dell’array vedono un valore di sola lettura &quot;N fps (in tempo reale)&quot; con la nota &quot;Impostato nelle impostazioni dell’array&quot;. |

### Rete / Trasporto

| Riga | Comportamento |
| --- | --- |
| **Dimensione pacchetto**| 1500 (Standard) / 9000 (Jumbo) — impostazione predefinita**Jumbo**. |
| **Throughput** | Limite di throughput del collegamento in sola lettura, in MB/s. Il backend ridistribuisce questo valore tra tutte le telecamere connesse ad ogni connessione/disconnessione. |
| **Gestione del buffer** | Modalità di gestione del buffer in sola lettura. |

### Acquisizione

Il pannello termina con un pulsante **&quot;Apri impostazioni di acquisizione…&quot;** che porta al [pannello Impostazioni di acquisizione](capture.md#the-capture-settings-pane) (disabilitato finché non viene aperto un progetto — &quot;Crea o apri un progetto per salvare le acquisizioni&quot;). Se la telecamera è nascosta o in pausa, un suggerimento ricorda di mostrarla/riprenderla prima di procedere all’acquisizione.

## Pannello delle impostazioni dell’array

Si apre facendo clic sull’**ingranaggio**su una riga dell’ARRAY. Intestazione: nome dell’array con una matita per rinominarlo e**×** per chiudere. Le sezioni sottostanti contrassegnate con *solo combinato* appaiono solo per gli array collegati in modalità di visualizzazione combinata.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Sincronizzazione

Righe **Master**,**Slaves**e**Linea di sincronizzazione** di sola lettura.

### Sensore di luce ambientale

Visualizzato sia per gli array combinati che per quelli separati:

* Menu a tendina **Sensore di luce**— associa un DAQ all’intero array. La selezione ha effetto immediato, si propaga al menu a tendina**Sensore di luce** di ogni singola telecamera (è comunque possibile sovrascrivere l’impostazione per ogni telecamera) e avvia l’invio degli spettri all’array.
* Riga **Stato** in tempo reale: Spento · &quot;In attesa del primo spettro…&quot; · &quot;Attivo — tutte le telecamere dell’array sono sottoposte a correzione dell’illuminazione&quot; · &quot;Nessuno spettro nuovo negli ultimi 3 s — si sta ancora utilizzando l’ultima lettura (nessun timeout per dati obsoleti)…&quot;.
* Nota nel riquadro: &quot;Correzione radiometrica a livello di array. Le impostazioni per singola telecamera hanno la precedenza.&quot;

### Target di calibrazione

Ha un proprio gruppo subito dopo il Sensore di luce ambientale, sia per gli array combinati che per quelli separati:

* Casella di controllo **Abilita** — &quot;Rileva il target ArUco di MAPIR

e convalida la LUT NDVI

rispetto alla riflettanza del pannello&quot;; determina la sovrapposizione del bersaglio e le tabelle di convalida del riquadro combinato. Su un array **separato** attiva la [tabella del bersaglio di calibrazione](#the-calibration-target-table) (una per ogni riga di telecamera nella vista elenco).

### Acquisizione — impostazioni uniformi dei sensori

Visualizzato sia per gli array combinati che per quelli separati (l’intestazione del gruppo recita &quot;Acquisizione (uniforme su tutte le telecamere)&quot;). Queste impostazioni si applicano in modo uniforme a ogni membro (le modifiche per singolo membro comprometterebbero la sincronizzazione). Le modifiche vengono accumulate e applicate insieme con **Applica**— tranne l’opzione**ROI on / ROI off**, che si applica immediatamente.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Controllo | Opzioni / intervallo | Funzione |
| --- | --- | --- |
| **Formato pixel** | 8 bit / 10 bit / 12 bit / 16 bit | Profondità di bit uniforme per tutti i membri; ogni telecamera adotta il formato di quella profondità per il proprio sensore. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning hardware — mantiene l’intero campo visivo aumentando al contempo il rapporto segnale/rumore (SNR) e la frequenza dei fotogrammi. Modificandolo, i campi ROI vengono reimpostati sul nuovo campo visivo completo. |
| **Risoluzione** preimpostata | Intera / Metà / Un quarto | Relativa al binning; riempie i campi ROI con un ritaglio centrato. |
| **Ritaglio ROI (px)**| Interruttore**ROI on / ROI off**, pulsante**Seleziona**, campi L / A / X / Y, mappa del sensore completo | Ritaglio del sensore — vedi [L’editor ROI](#the-roi-editor). Larghezza/altezza si allineano a multipli di 16 (minimo 64); gli offset si allineano a multipli di 4. Un suggerimento “max WxH” indica il limite massimo e**Reimposta** riporta i campi al campo visivo completo. Durante la modifica, il riquadro di ritaglio arancione tiene traccia di ogni modifica dei campi sulla mappa e sulla tessera dell’array. |
| **Frequenza di trigger**| Interruttore Auto / Manuale + fps 0,5–10, passo 0,5 |**Auto**(impostazione predefinita): il backend ricava la frequenza di attivazione dalla risoluzione e dalla larghezza di banda — l’input è disabilitato e mostra il valore ricavato.**Manuale**: fissa il valore al momento di Applicare. Una frequenza fissata che una nuova dimensione del fotogramma non è in grado di supportare viene mantenuta alla frequenza massima consentita da quella dimensione, e torna alla frequenza precedente quando la dimensione del fotogramma lo consente nuovamente. |

Nota nel pannello: «Le modifiche al formato/alla risoluzione riavviano brevemente tutte le telecamere. La frequenza di trigger viene applicata in tempo reale». I pulsanti **Applica / Annulla** si trovano nella parte inferiore del pannello.

#### L’editor ROI

Novità nella versione 1.3.0. La riga di ritaglio ROI e la mappa sottostante funzionano insieme:

* **ROI attivo / ROI disattivo**— attiva o disattiva**immediatamente**il ritaglio del sensore (le telecamere si riavviano brevemente; il riquadro dell’array mostra &quot;Applicazione delle impostazioni…&quot;).**Disattivato**legge l’intero sensore e**ricorda il ritaglio**— i campi continuano a mostrarlo;**Attivato** lo applica nuovamente. Attivando la ROI senza alcun ritaglio impostato, viene applicato un ritaglio centrato a metà dimensione (“Non è stato impostato alcun ritaglio, quindi è stato applicato un ritaglio centrato a metà dimensione. Regolalo tramite i campi.”).
* **Seleziona**— disegna il ritaglio sulla mappa: trascina il cursore su di esso oppure fai clic su due angoli opposti (Esc annulla). Il rettangolo viene inserito nei campi L/H/X/Y, allineato in base ai passaggi precedenti; non viene applicato nulla finché non si preme**Applica**, quindi è possibile perfezionarlo prima con le frecce dei campi. Un rettangolo per ogni pressione di Seleziona.
* **La mappa**mostra il ritaglio sull’intero sensore, quindi è possibile allargarlo nuovamente: “Sensore completo, in tempo reale” mentre le telecamere leggono il sensore completo, oppure &quot;Istantanea a pieno formato delle {ora}. Le telecamere leggono solo l’area ritagliata&quot; mentre sono in modalità ritagliata. Se l’array ha sempre trasmesso solo immagini ritagliate, la mappa riporta &quot;Nessuna vista a pieno formato ancora: questo array ha trasmesso solo immagini ritagliate.&quot; con un pulsante**Mostra fotogramma completo**, che disattiva la ROI. L’etichetta del ritaglio `W×H @ X,Y` si trova all’esterno del riquadro (sopra di esso, o sotto quando il riquadro tocca la parte superiore), sulla mappa e sul riquadro in tempo reale.
* **Applica**conferma le impostazioni: un ritaglio attiva la ROI; l’inquadratura completa (dopo**Reimposta**) la disattiva, pur mantenendo in memoria il ritaglio precedente; una ROI disattivata, i cui campi mostrano ancora il ritaglio memorizzato, rimane disattivata quando si applica un’altra modifica (formato, binning, frequenza).
* **Su un array allineato, la finestra di ritaglio di ciascun membro viene posizionata in base all’allineamento**, in modo che ogni fotocamera inquadri la stessa porzione della scena. Un array ritagliato prima di essere mai stato allineato viene prima allineato sull’intero sensore (il ritaglio viene rimosso, l’allineamento risolto e il ritaglio ripristinato con finestre specifiche per ciascuna fotocamera).
* Dopo una modifica del ritaglio, l’esposizione automatica si riconfigura in base alla nuova inquadratura. Durante un ritaglio, l’esposizione di un membro può comunque protrarsi fino al termine del periodo di trigger.
* Lo stato di attivazione/disattivazione della ROI e il ritaglio memorizzato vengono salvati con il progetto.

### Allineamento (co-registrazione)

Visualizzato sia per **gli array combinati che per quelli separati** (la versione 1.2.0 lo mostrava solo per gli array combinati); l’intestazione del gruppo recita &quot;Allineamento&quot;.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Casella **Residuo**: &quot;RMS x.xx px&quot; — verde se inferiore a 1 px, giallo se inferiore a 3 px, rosso in caso contrario o se una qualsiasi telecamera ha dato esito negativo; &quot;nessun profilo&quot; prima della prima risoluzione.
* Riga di riepilogo: &quot;✓ Tutte le telecamere allineate (N)&quot; / &quot;⚠ p/N telecamere allineate —  <serial (filter)="">errore&quot; / &quot;Ritaglio attivo — Ricalibrare per allineare (utilizza l’intero sensore)&quot; / «In attesa che l’esposizione si stabilizzi…» / «Applicazione delle impostazioni…» (le telecamere si stanno riavviando) / «L’array ha smesso di fornire fotogrammi — ricollegare l’array».
* Tabella per singola telecamera: telecamera (ultime 4 cifre del numero di serie + filtro), errore di riproiezione in px con conteggio delle corrispondenze (&quot;ref&quot; per la telecamera master) e punteggio di correlazione incrociata normalizzata della sovrapposizione rispetto alla soglia minima di superamento di 0,35.
* **Ricalibra allineamento** (con la dicitura &quot;Calibra allineamento&quot; prima del primo profilo) — riesegue la co-registrazione su fotogrammi nuovi.
* Casella di controllo **&quot;Esposizione automatica delle telecamere per l’allineamento&quot;** (selezionata per impostazione predefinita) — schiarisce temporaneamente le telecamere scure o piatte (prima l’esposizione, poi il guadagno) in modo che presentino una texture da abbinare, quindi ripristina l’esposizione automatica.

Chloros allinea automaticamente ogni array dopo la connessione, in entrambe le modalità di visualizzazione, una volta che l’esposizione automatica dei membri si è stabilizzata (continua a provarci per un massimo di 5 minuti); anche l’anteprima combinata si allinea all’apertura. Un array separato non ha una tessera composita, quindi questa sezione verifica lo stato di allineamento ed esegue autonomamente la ricalibrazione. Ricalibrare se la messa a fuoco o la profondità della scena sono cambiate. L’allineamento è **progettato per durare solo per la sessione** — non viene mai salvato in un profilo, poiché dipende dalla distanza della scena in quel momento. Le acquisizioni possono comunque essere esportate con registrazione pixel per pixel (vedi [Esportazioni allineate](capture.md#per-array-controls)).

### Vignettatura intelligente

* Casella di controllo **Abilita correzione**— applica la stima della vignettatura per ciascuna fotocamera alla catena radiometrica (in tempo reale**e** nelle esportazioni).
* **Calibra dalla vista corrente**— puntare prima l’array verso un bersaglio uniforme (schermo piatto, parete o cielo); ogni telecamera viene appiattita individualmente e lo stato riporta un guadagno di appiattimento di &quot;n/N telecamere · −x,x %&quot;.**Cancella** rimuove la stima.
* Stato: &quot;Calibrato&quot;, &quot;Non calibrato&quot; o &quot;Parzialmente calibrato — rieseguire&quot; quando solo le stime di alcuni membri sono ancora valide nell’inquadratura corrente (ad esempio dopo che un’operazione di Ricalibrazione ha spostato la finestra di ritaglio di un membro) — eseguire nuovamente Calibra.
* Regola con precisione per ogni telecamera utilizzando il cursore **Vignetta** specifico per ciascuna telecamera in [Anteprima dal vivo](#live-preview).

### Anteprima in tempo reale

* Casella di controllo **Istogramma**+ cursore di larghezza (0,10–0,90 del fotogramma) — la striscia dell’istogramma sul feed su tutte le tessere di ogni membro contemporaneamente (tessere a canali separati incluse), la [Sovrapposizione dell’istogramma](#display-overlays-drawn-over-the-live-feed) impostata per l’intero array. Appare come attiva quando ogni membro la possiede già. Su un array**separato** questa è l’unica riga del gruppo.
* *Solo in modalità combinata:* **Indice**: casella di selezione &quot;Abilita&quot; + icona a forma di ingranaggio — apre il [Calcolatore dell’indice](#index-calculator-pane) condiviso con le bande tracciate da**tutte** le telecamere dei membri. Una riga di anteprima dell’espressione sottostante mostra l’espressione corrente (&quot;Nessuna espressione impostata — apri il calcolatore per crearne una&quot;), aggiornata ogni secondo.
* *Solo in modalità combinata:* menu a tendina **Risoluzione di rendering**(stesse impostazioni predefinite delle singole telecamere, predefinita 720p): l’altezza dello streaming in diretta**e** la dimensione di esportazione del composito salvato. Nota nel pannello: &quot;Anteprima + dimensione del composito salvato. Le immagini delle singole telecamere vengono sempre esportate a piena risoluzione.&quot;

### Livelli di visualizzazione *(solo in modalità combinata)** Casella di controllo **Abilita** (disattivata per impostazione predefinita — la telecamera principale viene mostrata direttamente; attivata = composito a livelli).
* Menu a tendina **Menu a tendina**Primo piano**/**Sfondo**: ciascuna telecamera membro (per nome) o**Indice**. Quando Primo piano è impostato su Indice, i pixel al di fuori dei valori Min/Max della LUT mostrano il livello Sfondo.

### Vista divisa *(solo combinata)*

**&quot;Mostra telecamere membro&quot;**— un pulsante**Dividi / Nascondi telecamere associate** che aggiunge il feed live di ciascuna telecamera associata come riquadri separati della griglia accanto al composito. I riquadri leggono il frame buffer esistente dell’array (senza connessione aggiuntiva alla telecamera). Solo vista a griglia; salvata per array con il progetto.

### Funzionalità

Un pannello di sola lettura aggiornato ogni 5 s:

* **Etichetta livello**: &quot;Acquisizione simultanea&quot; (verde) · &quot;Acquisizione simultanea (emissione sfalsata FTD)&quot; (verde) · &quot;Acquisizione sfalsata (deriva di 100 ms)&quot; (ambra) · &quot;Configurazione troppo grande&quot; (rosso).
* **Stato del frame**: &quot;x,xx % incompleto&quot; — verde se inferiore all’1 %, giallo se inferiore al 5 %, rosso al 5 % o più.
* ****Linea di collegamento**: &quot;NIC {mbps} Mbps - sostenuta {MB/s} MB/s&quot;.

Questo rappresenta il budget di larghezza di banda in tempo reale dell’array. Per i valori fps sottostanti e il modello di rete — e cosa modificare quando il livello diventa giallo o rosso — consultare [Array multicamera](arrays.md) e il [Riferimento CLI](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Pannello «Calcolatore dell’indice»

La terza pagina della barra laterale, condivisa dall’ingranaggio «Indice» per singola telecamera e dall’ingranaggio «Indice» per array combinato (uno alla volta — l’intestazione recita «Calcolatore dell’indice — <camera name="">» o «Calcolatore dell’indice — <array name="">»). Riceve l’elenco delle bande (le bande naturali del filtro-bande naturali della fotocamera, oppure tutte le bande dei membri dell’array), l’espressione corrente e la configurazione della LUT (on/off, livello — predefinito 7 stop, min — predefinito 0,2, max — predefinito 1), oltre a un istogramma dell’indice in tempo reale. **Applica** conferma l’espressione; le modifiche alla LUT si applicano in tempo reale all’anteprima.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Impostazioni per singola telecamera vs. gestite dall’array

Riferimento rapido su cosa si trova dove quando una telecamera è membro di un array:

| Gestito dall’array (sola lettura nel pannello della telecamera) | Ancora per singola telecamera all’interno di un array |
| --- | --- |
| Formato pixel, risoluzione, binning | Esposizione automatica (esposizione, guadagno, target, smussamento, ROI) |
| Modalità/fonte di scatto, frequenza dei fotogrammi | Riduzione del rumore, nitidezza, vignettatura |
| | Orientamento (specchio/rotazione), sovrapposizioni di visualizzazione (impostazioni dell’array → L’anteprima live imposta l’istogramma su tutti i membri contemporaneamente), esposimetro spot |
| | Indice (array con visualizzazione separata), associazione del sensore di luce |

Altri comportamenti trasversali:

* **Visualizzazione combinata vs separata** viene scelta al momento della connessione dell’array: combinata = un’unica tessera composita allineata (i feed dei singoli elementi sono visibili solo tramite Split View); separata = ogni elemento rende la propria tessera sincronizzata. Una telecamera non mostra mai contemporaneamente un feed autonomo e una tessera dell’array.
* ****Riconnessione automatica**: l’apertura di un progetto salvato ripristina le telecamere e gli array e riapplica tutte le impostazioni salvate al backend prima che i flussi riprendano.
* **Gating di acquisizione**: le telecamere nascoste o in pausa sono escluse da “Acquisisci tutto”; un array viene completamente bloccato solo quando TUTTI i membri sono nascosti/in pausa. Vedi [Impostazioni e modalità di acquisizione](capture.md).

## Come vengono mantenute le impostazioni

Lo stato della scheda “Telecamera” viene salvato **con il progetto**, non nel browser:

* Ogni modifica reattiva crea un’istantanea delle telecamere e degli array nel file `cameras.json` del progetto (con un tempo di ritardo di 500 ms). Ciò include nomi e colori delle telecamere, impostazioni di esposizione/guadagno/AE, formato pixel/risoluzione/binning, frequenza di trigger, impostazioni di anteprima (risoluzione di rendering, riduzione del rumore, nitidezza, vignettatura, profilo colore, luci, saturazione/contrasto), orientamento, sovrapposizioni, divisioni dei canali, configurazione dell’indice, impostazioni AE predittivo, ROI AE, nomi degli array, modalità di visualizzazione, impostazioni di acquisizione degli array (inclusi il ritaglio ROI, se il ROI è attivo e il ritaglio memorizzato quando è disattivato), il Wire Budget dell’array e il blocco griglia (zoom del feed, modalità di visualizzazione, blocco griglia, ordine manuale delle tessere, telecamere nascoste, tessere chiuse, telecamera attiva).
* Le associazioni dei sensori di luce vengono salvate nel file `sensors.json` del progetto.
* La riapertura del progetto ricollega l’hardware e riapplica tutte le impostazioni.
* **Nessun progetto aperto = solo sessione**: in assenza di un progetto, nulla viene conservato alla chiusura di Chloros.
* Solo sessione indipendentemente dal progetto: stato di pausa, campioni dell’esposimetro spot, casella di controllo “Calibration Target” per ciascuna telecamera (sempre disattivata all’apertura) e profilo di allineamento dell’array (ricalcolato per ogni sessione per impostazione predefinita).
* Un&#x27;eccezione: le selezioni di esportazione delle **Impostazioni di acquisizione** e la modalità di acquisizione permangono per ogni progetto nella memoria locale dell&#x27;app anziché in `cameras.json` — vedi [Impostazioni e modalità di acquisizione](capture.md).</array></camera></serial>
