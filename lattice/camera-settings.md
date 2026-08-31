# Impostazioni telecamera

La scheda **Telecamere**è la superficie di controllo in tempo reale di Chloros per le telecamere LATTICE: un&#x27;area di visualizzazione principale che mostra ogni telecamera collegata sotto forma di riquadro in tempo reale e una barra laterale che scorre tra tre pagine — l&#x27;**elenco delle telecamere**, un**pannello delle impostazioni**(impostazioni per singola telecamera, per array o di acquisizione — una alla volta) e il**Calcolatore dell’indice**. Questa pagina descrive ogni comando presente nell’elenco delle telecamere, nel pannello delle impostazioni per singola telecamera e nel pannello delle impostazioni dell’array. Le modalità di acquisizione, la selezione del tipo di esportazione e il flusso “Acquisisci tutto” sono descritti nella pagina correlata [Impostazioni e modalità di acquisizione](capture.md).

La scheda “Telecamere” appare nella barra laterale una volta che il backend Chloros è pronto. Tutti i controlli riportati di seguito comunicano con il backend locale tramite `127.0.0.1:5000`; le modifiche vengono applicate immediatamente alla telecamera in diretta, salvo diversa indicazione.

## Tipi di telecamera utilizzati in questa pagina

I controlli si mostrano o si nascondono in base al tipo di telecamera selezionata. Il manuale utilizza questi termini in tutto il testo:

| Termine | Significato | Canali del filtro |
| --- | --- | --- |
| **Telecamera RGB** | LATTICE M3C con filtro FRGB (il modello contiene `-FRGB`) | Red / Green / Blue |
| **multispettrale Bayer** | LATTICE M3C con FRGN, FOCN o FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — un filtro a banda stretta, una banda calibrata | Banda singola |
| **Membro dell’array** | Una telecamera collegata come parte di un array sincronizzato (visualizzazione combinata o separata) | Per il proprio filtro |

Le telecamere RGB vengono sottoposte a elaborazione fotometrica (bilanciamento del bianco, profili colore, gamma); le telecamere multispettrali e mono vengono sottoposte alla catena radiometrica e saltano i controlli fotometrici. I membri dell’array trasferiscono le impostazioni a livello di flusso (formato pixel, risoluzione, binning, trigger, frequenza dei fotogrammi) all’array: tali righe diventano di sola lettura nel riquadro specifico per ciascuna telecamera e vengono spostate nel riquadro delle impostazioni dell’array.

## L’area

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

principale del feed In assenza di telecamere collegate, l’area del feed mostra una schermata iniziale con il messaggio **&quot;Collega una telecamera per iniziare&quot;**e due pulsanti:**Collega telecamera**(verde, apre la finestra di dialogo per il collegamento di una singola telecamera) e**Collega array** (blu, apre la finestra di dialogo per il collegamento dell’array). Le finestre di dialogo di connessione sono documentate in [Connessione delle telecamere](connecting.md); i concetti relativi agli array (sincronizzazione, livelli, larghezza di banda) sono descritti in [Array multicamera](arrays.md). Quando si apre un progetto salvato che contiene telecamere, la schermata iniziale mostra invece un indicatore di caricamento con il messaggio &quot;Riapertura di N telecamere salvate…&quot;, mentre Chloros ripristina i flussi dall’ultima sessione.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Barra superiore

| Controllo | Funzione |
| --- | --- |
| **Alternanza modalità di visualizzazione**| Passa dalla**vista a griglia**(tutte le tessere come celle) alla**vista elenco** (matrici a larghezza intera in alto, UNA telecamera attiva sotto). Suggerimenti: &quot;Passa alla visualizzazione a griglia&quot; / &quot;Passa alla visualizzazione a elenco&quot;. |
| **Blocco griglia**(lucchetto) |**Bloccata** per impostazione predefinita — riquadri fissati in posizione. Sbloccare per trascinare e riordinare i riquadri in qualsiasi slot (gli spazi vuoti vengono mantenuti). La griglia si blocca automaticamente ogni volta che si connette una nuova telecamera. Suggerimenti: &quot;Sblocca griglia (abilita trascinamento riquadri)&quot; / &quot;Blocca griglia (fissa riquadri in posizione)&quot;. |
| Cursore **Zoom feed** | Dimensione delle tessere, da 60 px fino alla larghezza totale del contenitore. Le celle mantengono un rapporto di aspetto 4:3. Se la larghezza della cella è inferiore a 200 px, le sovrapposizioni del nome e degli fps vengono nascoste per mantenere la tessera pulita. |

### Riquadri del feed

Ogni telecamera genera un riquadro composito in tempo reale; una telecamera può inoltre mostrare tre riquadri **divisi per canale** in scala di grigi (vedi [Divisioni dei canali](#display-overlays-drawn-over-the-live-feed)), mentre gli array generano un riquadro combinato. Il riquadro attivo presenta un anello di selezione nel colore della telecamera (o dell’array).

Passando il mouse su un riquadro viene visualizzato un pulsante di chiusura **X**:

* Chiudere un riquadro **composito** mentre le sue divisioni per canale rimangono visibili nasconde solo il riquadro composito.
* Chiudendo l’**ultimo riquadro visibile di una telecamera autonoma**, si disconnette quella telecamera.
* **I riquadri suddivisi appartenenti a un array combinato non disconnettono mai** la telecamera: la nascondono soltanto.

Con la griglia sbloccata, trascinare qualsiasi riquadro in qualsiasi slot; il layout viene salvato con il progetto.

## Barra laterale — elenco

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

delle telecamere La prima pagina della barra laterale elenca tutte le telecamere e gli array collegati:

* **Collega telecamera**(verde) /**Collega array** (blu, mostra «Rilevamento in corso...» durante la scansione). Entrambi disabilitati mentre è aperta una finestra di dialogo di collegamento.
* **Acquisisci tutto** (rosso) — acquisisce tutte le telecamere elencate con i tipi di esportazione scelti nelle Impostazioni di acquisizione. Richiede un progetto aperto. Documentato in modo completo in [Impostazioni e modalità di acquisizione](capture.md).
* **Ingranaggio delle impostazioni di acquisizione**(accanto a**Acquisisci tutto**) — apre il [pannello delle impostazioni di acquisizione](capture.md#the-capture-settings-pane). Disabilitato in assenza di un progetto o durante l’acquisizione.

### Righe delle telecamere

Ogni riga di telecamera mostra un bordo con codice colore (il colore personalizzato della telecamera), un&#x27;etichetta “CAM” — con una lettera blu **M**(master) o verde**S** (slave) che indica il ruolo dei membri dell’array — e il nome visualizzato. Il nome predefinito è `LATTICE-MODEL (serial)`; è possibile rinominarlo dal pannello delle impostazioni specifiche per ciascuna telecamera. Pulsanti della riga:

| Pulsante | Effetto |
| --- | --- |
| **Occhio**| Attiva/disattiva la visibilità. Le telecamere nascoste escono dalla griglia e vengono**escluse da &quot;Cattura tutto&quot;**. |
| **Ingranaggio** | Apri il pannello delle impostazioni per singola telecamera (sezione successiva). |
| **Pausa / Riproduci**| Blocca l’anteprima live**solo sul lato display** — l’acquisizione sul backend continua a funzionare. Le telecamere in pausa non possono acquisire. |
| **X** | Disconnetti. L’interfaccia utente si aggiorna immediatamente (in teoria); la disconnessione effettiva da parte del backend può richiedere dai 10 ai 30 s per completarsi. |

### Righe dell’array

Una riga dell’array mostra un badge “ARRAY” nel colore dell’array, il nome dell’array (rinominabile nelle impostazioni dell’array) e un **DAQ · on/off**—**on** quando il sensore di luce a livello di array è attivato *oppure* un qualsiasi membro dispone di un sensore per singola telecamera; il suo tooltip elenca esattamente quale sensore alimenta cosa. Le telecamere dei membri sono elencate indentate sotto con le loro righe dedicate. Pulsanti della riga dell’array: **occhio**(nasconde/mostra TUTTI i membri insieme),**ingranaggio**(pannello delle impostazioni dell’array),**X**(disconnette l’intero array).

Lo stato del sensore di luce (DLS) utilizzato nelle righe dell’array e nel pannello delle impostazioni dell’array presenta quattro stati:**off**,**in attesa**(ancora nessun spettro),**attivo**(è arrivato uno spettro negli ultimi 3 s) e**non aggiornato** — nessun nuovo spettro negli ultimi 3 s, ma l’ultima lettura viene *ancora utilizzata* (le letture DAQ non scadono mai nel percorso di acquisizione).

È possibile trascinare le telecamere singole e i gruppi di array interi l’uno sopra l’altro nella barra laterale per riordinare l’elenco; i membri dell’array non sono trascinabili in modo indipendente.

## Pannello delle impostazioni per singola telecamera

Si apre cliccando sull’icona **a forma di ingranaggio** su una riga della telecamera. Il pannello si sovrappone all’elenco delle telecamere.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Intestazione**: il**campione di colore**della telecamera (fare clic per aprire un selettore di colori nativo — imposta il colore del bordo della barra laterale e dell’anello di selezione delle tessere), il**nome**con un pulsante a forma di matita**Rinomina**(salvando un nome vuoto si ripristina il valore predefinito `MODEL (serial)`) e**×** per chiudere.

### Istogramma in tempo reale

Nella parte superiore del pannello è presente un istogramma di luminanza in tempo reale calcolato dall’anteprima JPEG a circa 8 Hz. La media è ponderata secondo lo schema Bayer — (R+2G+B)/4 — per corrispondere alla misurazione AE della telecamera stessa.

* **Linea tratteggiata Orange**= il target AE.**Trascinarla orizzontalmente per ridefinire il target** — al rilascio viene inviato un comando, e il trascinamento commuta la modalità del target AE su Manuale.
* **Linea continua Green** = la luminanza media effettiva (quella che l’AE sta attualmente fornendo).
* **Pulsante RGB** (in alto a destra): attiva/disattiva gli istogrammi sovrapposti per banda colorati in base al filtro della fotocamera (ad es. su FRGN: grigio NIR, verde, rosso). Sulle telecamere mono (M3M) il pulsante riporta la scritta “MONO” ed è disabilitato — la modalità mono mostra sempre l’istogramma di luminanza a banda singola.
* Le etichette dell’asse X seguono la profondità di bit del sensore del formato pixel corrente: 0..255, 0..1023, 0..4095 o 0..65535.

### Righe

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

informative sulla fotocamera | Riga | Comportamento |
| --- | --- |
| **Modello** | Solo lettura (ad es. `LATT-M3C-L87-FRGN`). |
| **Calibrazione radiometrica**| Green Icona**&quot;Attiva&quot;**con una didascalia che mostra il livello di calibrazione, l’hash, la data di calibrazione e l’elenco delle bande, caricati dal pacchetto di calibrazione della fotocamera (vedi [Calibrazione radiometrica di fabbrica](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Nascosto per le telecamere RGB** — queste dispongono di una calibrazione fotometrica del bilanciamento del bianco, non della radianza per banda. |
| **Rapporto di calibrazione**| Pulsante**Scarica** — apre il certificato di calibrazione NIST specifico per ogni numero di serie della telecamera in formato PDF nel visualizzatore del sistema operativo. Se il certificato non è ancora stato memorizzato nella cache, Chloros mostra invece un suggerimento. |
| **Numero di serie** | Solo lettura. |
| **Firmware**| Mostra la versione corrente, quindi individua la versione disponibile per questo modello (memorizzata nella cache per modello — un array di N telecamere esegue una verifica sul server una sola volta). Stati: &quot;Verifica in corso…&quot; → pulsante**&quot;Aggiorna a X&quot;**→ &quot;Aggiornamento in corso…&quot; → &quot;Aggiornato da A a B&quot; / &quot;Fallito: …&quot; / &quot;Saltato: …&quot; / verde**&quot;Aggiornato&quot;**. Descrizione del pulsante di aggiornamento: &quot;Ripristino impostazioni di fabbrica + flash + riprogrammazione UserSet1. ~2–3 minuti; non scollegare.&quot; |
| **IP** | Solo lettura. |
| **Temperatura** | Solo lettura, aggiornata ogni 3 s. Diventa arancione a ≥65 °C e rossa con un ⚠ a ≥75 °C. |
| Casella di controllo **Obiettivo di calibrazione** | Abilita il rilevamento dell’obiettivo di riflettanza ArUco con una tabella di validazione NDVI per pannello sotto il feed in tempo reale (vista elenco). Solo per la sessione — si apre sempre disattivata. |
| Menu a tendina **Sensore di luce** | Associa un sensore di luce DAQ (DAQ-E/M/U, dall’elenco della scheda Sensori di luce) a questa telecamera per la correzione dell’illuminazione con luce discendente (DLS) e l’esposizione automatica predittiva. “Nessuno” cancella l’associazione. Se non sono collegati sensori, il menu a tendina mostra &quot;(nessun sensore collegato — aprire la scheda DAQ)&quot;. L&#x27;associazione viene salvata con il progetto. |

### Esposizione e guadagno

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Tutti i campi numerici qui presenti utilizzano rotelle di selezione con accelerazione al mantenimento: tocco = ±1, mantenimento &gt;1,5 s = ±10, mantenimento &gt;3 s = ±100. Il valore viene inviato alla fotocamera quando si rilascia.

| Controllo | Intervallo / opzioni | Predefinito | Si applica a | Cosa fa |
| --- | --- | --- | --- | --- |
| **Esposizione (us)**| Min/max in tempo reale della fotocamera | Auto | Tutti | Tempo di esposizione in microsecondi, con un selettore**Auto/Manuale**. Auto = esposizione automatica continua da parte della fotocamera. |
| **Guadagno (dB)**| Valori minimi/massimi in tempo reale della fotocamera (ad es. fino a 48 dB) | Manuale (disattivato) | Tutti | Guadagno analogico/digitale con un proprio selettore**Auto/Manuale**. |
| **Luminosità target AE**| 0–255 | 80, modalità**Auto**| Tutti (modificabile quando AE o il guadagno automatico sono attivi) | La luminosità a cui punta l’AE. In modalità**Auto**(impostazione predefinita) un controller interno basato sull’istogramma seleziona autonomamente il valore target, mantenendo l’esposizione al 60–75 % del massimo del sensore. Digitando un valore o trascinando la linea arancione dell’istogramma si passa alla modalità**Manuale**. |
| **Smussamento AE** | 0,5–40, passo 0,1 | 8,0 | Tutti | Smorzamento AE. Suggerimento: &quot;Valori più bassi = l’AE reagisce più velocemente (può pulsare a fps elevati). Valori più alti = più fluido / più lento.&quot; Valori molto inferiori a quelli predefiniti possono causare pulsazioni dell’AE e destabilizzare lo streaming a frame rate elevati; 8,0 è l’impostazione predefinita stabile. |
| **Area di interesse AE**| Casella di controllo “Abilita” + pulsante**Punta**| Disattivato | Tutti | Se attivata, AE misura solo l’area verde tratteggiata anziché l’intero fotogramma.**Mira**attiva la funzione &quot;clicca per posizionare&quot; sul feed live: un clic centra un&#x27;area al 30% del fotogramma; clicca e trascina per tracciare un rettangolo personalizzato (minimo 5% × 5%). La funzione**Aim** si disattiva automaticamente dopo un posizionamento. L’area viene rimappata sulle coordinate native della telecamera in base a qualsiasi rotazione o riflessione impostata dall’utente e viene salvata con il progetto. |
| **Velocità regolazione AE** | 0,1–5, incrementi di 0,1 | 1,0 | Solo per membri dell’array | Velocità con cui il target AE automatico tiene traccia delle variazioni di luminosità della scena; 1,0× ricalcola ogni 2,5 s. |
| **Protezione delle alte luci** | Rigida (1 %) / Normale (5 %) / Allentata (15 %) | Rigida | Solo per le telecamere che espongono questa impostazione | Quanta parte del fotogramma può essere clippata in bianco prima che l’AE scurisca l’immagine. |

{% hint style="info" %}
**Requisiti di illuminazione per le fotocamere multispettrali Bayer (RGN / OCN / NGB):** la scena deve avere luce sufficiente in tutti e tre i canali, altrimenti la calibrazione non funziona correttamente — un’unica esposizione del sensore copre tutti e tre gli spettri. Utilizza un sensore di luce DAQ per misurare l’illuminazione, oppure passa alla modalità interamente mono (M3M) in modo che ogni banda abbia la propria esposizione. Se un’acquisizione non rispetta questo requisito, Chloros la rileva e avvisa l’utente (notifica “unmix-clamp”).
{% endhint %}

### Formato pixel e risoluzione**I membri

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

dell’array** mostrano le righe di sola lettura «Current» (formato + LxH) e «Binning» con la nota «Impostato nelle impostazioni dell’array»: un riavvio dello stream su un membro interromperebbe la sincronizzazione, quindi queste impostazioni sono gestite nel [pannello delle impostazioni dell’array](#array-settings-pane).**Le telecamere autonome** dispongono di:

| Controllo | Opzioni | Funzione |
| --- | --- | --- |
| **Formato pixel** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Formato pixel del sensore (profondità di bit). |
| **Risoluzione** | Intera / Metà / Un quarto | Relativa al binning corrente: Intera = 2048/N × 1536/N per binning N×N. |
| **Binning** | 1x1 (nessuno) / 2x2 / 4x4 | Binning hardware N×N — valori più alti riducono la risoluzione ma aumentano il rapporto segnale/rumore (SNR) e la frequenza dei fotogrammi. Modificandolo si riavvia lo streaming e si reimpostano tutte le ROI sul nuovo campo visivo completo. |
| **Corrente** | di sola lettura | Le dimensioni effettive LxA e l’offset (x, y) attualmente in vigore. |

### Anteprima in tempo reale

Tutto ciò che è riportato in questa sezione riguarda **esclusivamente la visualizzazione**— modifica ciò che si vede nel feed in tempo reale, mentre le acquisizioni salvate rimangono lineari e inalterate — con un’unica eccezione:**Vignetta** è radiometrica e influisce anche sulle esportazioni (come indicato di seguito).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Controllo | Intervallo / opzioni | Predefinito | Si applica a | Cosa fa |
| --- | --- | --- | --- | --- |
| **Risoluzione di rendering** | 360p (più veloce) / 480p / 720p / 1080p / Risoluzione nativa del sensore (più lenta) | 720p | Tutti | L’altezza alla quale il backend esegue la catena di anteprima radiometrica. Un valore più basso garantisce una maggiore frequenza dei fotogrammi senza modificare il campo visivo. |
| **Indice**| Casella di selezione “Abilita” + ingranaggio | Disattivato | Solo multispettrale Bayer,**non** elementi dell’array combinato | Anteprima in tempo reale dell’indice di vegetazione. L’ingranaggio apre il [Calcolatore di indici](#index-calculator) precaricato con le bande naturali del filtro della fotocamera (ad es. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). L’espressione personalizzata più la LUT (on/off, livello predefinito 3, min predefinito 0,2, max predefinito 1) viene calcolata su ogni fotogramma di anteprima. I membri dell&#x27;array combinato nascondono questa riga — l&#x27;array possiede un unico indice condiviso. |
| **Bilanciamento del bianco** | Off / Una volta / Continuo + un pulsante di nuova acquisizione | Continuo | Solo RGB | Bilanciamento del bianco in tempo reale. Il pulsante di aggiornamentoacquisisce nuovamente il bilanciamento del bianco dallo spettro DLS corrente (disabilitato quando la modalità è disattivata). |
| **Gamma** | On / Off | On | Solo RGB | Visualizza la gamma (γ = 2,2 LUT) nell’anteprima in tempo reale. Le acquisizioni salvate rimangono lineari. |
| **Riduzione rumore** | Casella di controllo + intensità 0–100 | Off / 50 | Tutti (per singola telecamera, anche all’interno di array) | Filtro bilaterale sull’anteprima in tempo reale. Valori più alti = immagini più uniformi ma con dettagli meno nitidi. |
| **Nitidezza** | Casella di selezione + intensità 0–100 | Disattivato / 30 | Tutti | Maschera di contrasto nell’anteprima in tempo reale, applicata per ultima. Può amplificare il rumore. Solo in anteprima. |
| **Vignettatura**| Casella di selezione + intensità 0–100 | Disattivato / 0 | Tutti | Rimozione manuale della vignettatura residua (schiarisce gli angoli), sovrapposta alla stima della vignettatura intelligente dell’array.**Radiometrico — influisce sull’anteprima in tempo reale E sulle esportazioni**, a differenza di Denoise/Nitidezza. |
| **Profilo colore** | Raw / Lineare / Naturale / Migliorato / Temperatura personalizzata | Naturale | Solo RGB | Vedi sotto. |
| **Temperatura di colore** | 2000–10000 K, passo 100 | 5500 K | Solo RGB, solo profilo Temperatura personalizzata | Fissa il bilanciamento del bianco su una temperatura di colore correlata fissa (l’input DLS viene ignorato). L’ultimo valore in Kelvin selezionato viene memorizzato anche quando si cambia profilo. |
| **Saturazione** | 0–200 (100 = neutro) | 100 | Solo RGB | Saturazione HSV nell’anteprima in tempo reale. |
| **Contrasto** | 0–200 (100 = neutro) | 100 | Solo RGB | Contrasto lineare intorno al grigio medio nell’anteprima in tempo reale. |
| **Rifletti H / Rifletti V** | Caselle di controllo | Disattivato | Tutti | Capovolge l’anteprima orizzontalmente / verticalmente. |
| **Rotazione**| 0° / 90° / 180° / 270° | 0° | Tutti | Ruota l’anteprima. L’orientamento viene applicato alla fine della catena di anteprima del backend —**le acquisizioni salvate mantengono l’orientamento nativo della fotocamera** e le viste composite dell’array lo ignorano. |**Semantica dei profili colore** (telecamere RGB):

* **Raw** — bypassa completamente la catena di elaborazione.
* **Lineare** — segnale scuro + flat field + bilanciamento del bianco; nessuna matrice di colore, nessuna gamma.
* **Naturale** *(impostazione predefinita)* — lineare più la matrice di correzione del colore misurata e una curva di tono adattiva alla scena.
* **Migliorata**— Naturale più vivacità e contrasto locale CLAHE. Il costo aggiuntivo si applica**solo all’anteprima in tempo reale** — le immagini salvate ottengono sempre la finitura completa indipendentemente dal profilo.
* **Temperatura personalizzata** — *Natural* con il bilanciamento del bianco fissato al valore in Kelvin scelto.

{% hint style="warning" %}
Per le opzioni *Naturale*, *Migliorata* e *Temperatura personalizzata*, il pannello mostra una nota relativa alla tonalità: i fotogrammi vengono schiariti in base alla propria scena, quindi le immagini *visualizzate* salvate non sono comparabili tra un fotogramma e l’altro. **Esporta radianza o riflettanza per le misurazioni.**
{% endhint %}

### Sovrapposizioni sullo schermo (disegnate sul feed in tempo reale)

Sono disponibili solo nell’interfaccia utente — vengono sovrapposte al video, senza mai interferire con lo streaming o le immagini acquisite.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Sovrapposizione | Controlli | Impostazione predefinita | Funzione |
| --- | --- | --- | --- |
| **Zebra** | Casella di controllo + soglia 200–255 | Disattivato / 250 | Strisce diagonali magenta sui pixel tagliati. |
| **Mirino** | Casella di controllo | Disattivato | Segno al centro del fotogramma. |
| **Griglia** | Disattivato / 3 × 3 / 9 × 9 | Disattivato | Griglia di composizione. |
| **Istogramma** | Casella di selezione + larghezza 0,10–0,90 del fotogramma | Disattivato / 0,25 | Una striscia di istogramma sul feed. |
| **Focus Peak** | Casella di controllo + soglia 20–200 + campione di colore | Disattivato / 80 / `#ff5722` | Evidenziazione dei bordi con algoritmo Sobel per la messa a fuoco. |
| **Suddivisioni dei canali** | «Mostra suddivisioni (Red / Green / NIR)» / Pulsante «Nascondi divisioni&quot; | Nascosto | Aggiunge tre riquadri indipendenti in scala di grigi per canale accanto all&#x27;immagine composita (l&#x27;etichetta del pulsante segue i canali del filtro della fotocamera). Ciascun riquadro diviso è trascinabile e condivide il colore del bordo della fotocamera. Non disponibile sulle fotocamere monocromatiche. Viene salvato con il progetto. |

### Misuratore spot

* Casella di controllo **Clicca per campionare**: clicca sul feed in tempo reale per campionare un singolo pixel (contrassegnato da un reticolo a croce), oppure clicca e trascina su un’area per ottenere una media dei pixel.**Cancella**elimina il campione e il reticolo. Si esclude a vicenda con la modalità**Aim** dell’AE-ROI.
* Menu a tendina **Mostra**:**Raw (profondità di bit)**— valori digitali nativi alla profondità di bit del sensore (ad es. 12 bit → 0..4095) — oppure**Display (8 bit)** (impostazione predefinita). Quando è attivo un indice in tempo reale, la voce “Display” mostra invece il valore dell’indice calcolato (ad es. NDVI).
* Il pannello di lettura elenca le coordinate dei pixel, le dimensioni del fotogramma, il formato dei pixel, la profondità di bit e una tabella dei canali (Chan / Valore / %) con le etichette delle bande e le lunghezze d’onda; le coppie verdi Bayer sono mediate; i campioni di regione mostrano &quot;Media N px&quot;.

Lo stato dell’esposimetro spot è valido solo per la sessione.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Esposizione automatica predittiva (basata su DLS)

Questa sezione appare solo quando **è collegato almeno un sensore di luce DAQ** — il risolutore necessita di uno spettro in discesa in tempo reale per funzionare.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Controllo | Intervallo | Predefinito | Funzione |
| --- | --- | --- | --- |
| **Abilita** | Casella di selezione | On (telecamere autonome) | Un risolutore in forma chiusa utilizza lo spettro DLS insieme agli scalari del pacchetto di calibrazione della telecamera per portare la banda più luminosa vicino alla saturazione, mantenendo al contempo la banda più scura al di sopra della soglia minima di SNR — una sola scrittura di esposizione per ogni risoluzione, senza ciclo di stabilizzazione. Progettato per time-lapse alimentati a energia solare in cui ogni scatto deve essere correttamente esposto. Il backend ricorre silenziosamente all’esposizione automatica reattiva ogni volta che la lettura DLS è obsoleta/mancante o il pacchetto di calibrazione non è caricato. |
| **Smussamento (α)** | 0,05–1,0, passo 0,05 | 0,3 | Smussamento delle soluzioni predittive successive (valore più basso = smussamento maggiore). |
| **Riflettanza della scena**| Pulsante**Ricalibra ρ** | — | Ricalcola il fattore di riflettanza della scena utilizzato dal risolutore. |

{% hint style="info" %}
**La connessione in array disattiva l’AE predittivo per impostazione predefinita** — per gli array, l’AE intelligente di Chloros, insieme all’esposizione automatica lato fotocamera, gestisce l’esposizione (con protezione dalla saturazione) e la stima singola della riflettanza della scena fornita dall’AE predittivo non è affidabile in presenza di scene miste. È possibile riattivarla per singola telecamera da qui se si desidera specificatamente un’esposizione radiometrica guidata dal DLS.
{% endhint %}

**Limite massimo di esposizione basato sul DAQ e AE ancorata all’incidente.**Indipendentemente dalla casella di controllo sopra indicata, quando un sensore di luce DAQ è assegnato a una fotocamera RGB, Chloros calcola — a partire dall’irradianza assoluta discendente misurata — l’esposizione massima × guadagno alla quale una superficie con riflettanza del 100% rimane al di sotto del clipping, e la applica come**limite massimo**all’esposizione automatica. Mentre il limite massimo è attivo, la fotocamera è**fissata sulla luce incidente**: funziona in anello aperto all’esposizione misurata in base alla luce incidente con guadagno a 0 dB — l’esposizione segue la luce misurata, non il contenuto della scena. Poiché il limite massimo può solo ridurre l’esposizione, non può di per sé causare il clipping. Il limite massimo si disattiva automaticamente — e riprende la normale esposizione automatica (AE) della scena — ogni volta che la lettura DAQ è assente, obsoleta (&gt;30 s), o scuro, oppure se ≥15 % del fotogramma presenta clipping all’esposizione fissata (il che significa che il sensore e la fotocamera rilevano un’illuminazione diversa). Non è presente alcun interruttore nell’interfaccia grafica; si tratta di un comportamento standard ogni volta che una fotocamera RGB ha un DAQ associato.

### I membri dell’array di acquisizione e trigger

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

mostrano inoltre le righe di sola lettura **Ruolo**(Master in blu / Slave in verde),**Linea di sincronizzazione**e**Pari**.

| Controllo | Opzioni | Predefinito | Note |
| --- | --- | --- | --- |
| **Modalità di trigger** | Disattivata / Attivata | Attivata | Disabilitata per i membri dell’array (l’array gestisce il trigger). |
| **Sorgente di trigger** | Software / Linea 0 (M8) / Linea 1 / Linea 2 | Linea 0 | Nascosta quando la modalità di trigger è disattivata; disabilitata per i membri dell’array. Line0 è l’ingresso di trigger esterno optoisolato dell’M8. |
| **Frequenza fotogrammi**| Auto / Manuale + valore | Auto |**Auto**: il limite di frequenza fotogrammi della telecamera è disattivato — l’esposizione determina gli fps e la casella mostra la frequenza effettiva in tempo reale.**Manuale**: è possibile limitare gli fps tramite un cursore (da 1 fino al massimo consentito dalla larghezza di banda), basato sulla frequenza effettiva corrente. I membri dell’array vedono un valore di sola lettura «N fps (in tempo reale)» con l’indicazione «Impostato nelle impostazioni dell’array». |

### Rete / Trasporto

| Riga | Comportamento |
| --- | --- |
| **Dimensione pacchetto**| 1500 (Standard) / 9000 (Jumbo) — impostazione predefinita**Jumbo**. |
| **Throughput** | Limite di throughput del collegamento in sola lettura in MB/s. Il backend ribilancia questo valore tra tutte le telecamere connesse ad ogni connessione/disconnessione. |
| **Gestione del buffer** | Modalità di gestione del buffer di sola lettura. |

### Acquisizione

Il pannello termina con un pulsante **&quot;Apri impostazioni di acquisizione…&quot;** che porta al [pannello Impostazioni di acquisizione](capture.md#the-capture-settings-pane) (disabilitato finché non viene aperto un progetto — «Crea o apri un progetto per salvare le acquisizioni»). Se la telecamera è nascosta o in pausa, un suggerimento ricorda di mostrarla o riprenderne il funzionamento prima di procedere all’acquisizione.

## Pannello delle impostazioni dell’array

Si apre tramite l’**ingranaggio**su una riga dell’ARRAY. Intestazione: nome dell’array con una matita per rinominarlo e**×** per chiudere. Le sezioni sottostanti contrassegnate con *solo combinato* appaiono solo per gli array collegati in modalità di visualizzazione combinata.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Sincronizzazione

Righe **Master**,**Slave**e**Linea di sincronizzazione** di sola lettura.

### Sensore di luce ambientale

Visualizzato sia per gli array combinati che per quelli separati:

* Casella di controllo **Target di calibrazione** — &quot;Rileva il bersaglio ArUco MAPIR e convalida la LUT di riflettanza NDVI rispetto al pannello&quot;; gestisce la sovrapposizione del bersaglio e la tabella di convalida del riquadro combinato.
* Menu a tendina **Sensore di luce** — associa un DAQ all’intero array. La selezione ha effetto immediato, si propaga al menu a tendina “Sensore di luce” di ciascuna telecamera dell’array (è comunque possibile sovrascrivere l’impostazione per singola telecamera) e avvia l’inoltro degli spettri all’array.
* Riga **Stato** in tempo reale: Spento · “In attesa del primo spettro…&quot; · &quot;Attivo — tutte le telecamere dell’array sono sottoposte a correzione dell’illuminazione&quot; · &quot;Nessuno spettro nuovo negli ultimi 3 s — si sta ancora utilizzando l’ultima lettura (nessun timeout per dati obsoleti)…&quot;.
* Nota nel riquadro: &quot;Correzione radiometrica a livello di array. Le impostazioni per singola telecamera hanno la precedenza.&quot;

### Acquisizione — impostazioni uniformi del sensore *(solo combinate)*

Queste impostazioni si applicano in modo uniforme a ogni membro (le modifiche per singolo membro comprometterebbero la sincronizzazione). Le modifiche vengono preparate e applicate insieme.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Controllo | Opzioni / intervallo | Funzione |
| --- | --- | --- |
| **Formato pixel** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Formato del sensore uniforme per tutti i membri. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning hardware — mantiene l’intero campo visivo aumentando al contempo il rapporto segnale-rumore (SNR) e la frequenza dei fotogrammi. Modificandolo, i campi ROI vengono reimpostati sul nuovo campo visivo completo. |
| **Risoluzione** preimpostata | Intera / Metà / Un quarto | Relativa al binning; riempie i campi ROI con un ritaglio centrato. |
| **Ritaglio ROI (px)**| Campi numerici L / A / X / Y | Ritaglio del sensore. Larghezza/altezza si allineano a multipli di 16 (minimo 64); gli offset si allineano a multipli di 4. Un&#x27;indicazione &quot;max LxA&quot; mostra il limite massimo e**Reimposta** riporta al campo visivo completo. Durante la modifica, viene disegnato un riquadro arancione di anteprima del ritaglio in tempo reale sulla tessera dell’array (incluso uno schema del sensore completo quando si espande il ritaglio verso l’esterno). |
| **Frequenza di trigger**| Selettore Auto / Manuale + fps 0,5–10, passo 0,5 |**Auto**(impostazione predefinita): il backend ricava la frequenza di trigger dalla risoluzione e dalla larghezza di banda — l’input è disabilitato e mostra il valore ricavato.**Manuale**: fissa il valore al clic su Applica. |

Nota nel pannello: «Le modifiche al formato/alla risoluzione riavviano brevemente tutte le telecamere. La frequenza di trigger viene applicata in tempo reale». I pulsanti **Applica / Annulla** si trovano nella parte inferiore del pannello.

### Allineamento (co-registrazione) *(solo combinato)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Casella **Residuo**: &quot;RMS x,xx px&quot; — verde se inferiore a 1 px, giallo se inferiore a 3 px, rosso in caso contrario o se una qualsiasi telecamera non ha superato il test; &quot;nessun profilo&quot; prima della prima risoluzione.
* Riga di riepilogo: &quot;✓ Tutte le telecamere allineate (N)&quot; / &quot;⚠ p/N telecamere allineate —  <serial (filter)="">fallimento&quot; / &quot;Ritaglio attivo — Ricalibrare per allineare (utilizza l’intero sensore)&quot; / &quot;In attesa che l’esposizione si stabilizzi…&quot;.
* Tabella per singola telecamera: telecamera (ultime 4 cifre del numero di serie + filtro), errore di riproiezione in px con conteggio delle corrispondenze (“ref” per la telecamera master) e punteggio di correlazione incrociata normalizzata della sovrapposizione rispetto alla soglia minima di superamento di 0,35.
* Pulsante **Ricalibra allineamento** (con la dicitura &quot;Calibra allineamento&quot; prima del primo profilo) — riesegue la co-registrazione su fotogrammi nuovi.
* Casella di controllo **&quot;Esposizione automatica delle telecamere per l’allineamento&quot;** (selezionata per impostazione predefinita) — schiarisce temporaneamente le telecamere scure o piatte (prima l’esposizione, poi il guadagno) in modo che presentino una texture da allineare, quindi ripristina l’esposizione automatica (AE).

L’anteprima combinata si allinea automaticamente all’apertura; ricalibrare se la messa a fuoco o la profondità della scena sono cambiate. L’allineamento è **specifico della sessione**solo alla sessione** — non viene mai salvata in un profilo, poiché dipende dalla distanza della scena in quel momento. Le acquisizioni possono comunque essere esportate con registrazione pixel per pixel (vedi [Esportazioni allineate](capture.md#per-array-controls)).

### Vignettatura intelligente

* Casella di controllo **Abilita correzione**— applica la stima della vignettatura per ciascuna telecamera alla catena radiometrica (in tempo reale**e** nelle esportazioni).
* **Calibra dalla vista corrente**— punta prima l’array verso un bersaglio uniforme (schermo piatto, parete o cielo); ogni telecamera viene livellata individualmente e lo stato riporta un guadagno di planarità &quot;n/N telecamere · −x,x %&quot;.**Cancella** rimuove la stima.
* Regola con precisione per ogni telecamera utilizzando il cursore **Vignettatura** specifico per ciascuna telecamera in [Anteprima in tempo reale](#live-preview).

### Anteprima dal vivo *(solo combinata)** **Indice**: seleziona la casella di controllo + icona a forma di ingranaggio — apre il [Calcolatore dell’indice](#index-calculator-pane) condiviso con le bande tracciate da**tutte** le telecamere del gruppo. Una riga di anteprima dell’espressione sottostante mostra l’espressione corrente (&quot;Nessuna espressione impostata — apri il calcolatore per crearne una&quot;), aggiornata ogni secondo.
* **Menu a tendina**Risoluzione di rendering**(stesse impostazioni predefinite di quelle per singola telecamera, 720p predefinito): l’altezza del flusso di visualizzazione in tempo reale**e** la dimensione di esportazione del composito salvato. Nota nel pannello: &quot;Anteprima + dimensione del composito salvato. Le immagini per singola telecamera vengono sempre esportate a piena risoluzione.&quot;

### Livelli di visualizzazione *(solo combinati)** Casella di controllo **Abilita** (disattivata per impostazione predefinita — la telecamera principale viene mostrata direttamente; attivata = composito a livelli).
* Menu a tendina **Primo piano**/**Sfondo**: ciascuna telecamera membro (per nome) o**Indice**. Quando il Primo piano è impostato su Indice, i pixel al di fuori dei valori Min/Max della LUT mostrano il livello Sfondo.

### Vista divisa *(solo combinata)*

**&quot;Mostra telecamere associate&quot;**— un pulsante**Dividi / Nascondi telecamere associate** che aggiunge il feed live di ciascuna telecamera associata come riquadri separati della griglia accanto al composito. Le tessere leggono il frame buffer esistente dell’array (senza connessione aggiuntiva alla telecamera). Solo in vista a griglia; salvata per ogni array insieme al progetto.

### Funzionalità

Un pannello di sola lettura aggiornato ogni 5 s:

* **Etichetta del livello**: “Acquisizione simultanea” (verde) · &quot;Acquisizione simultanea (emissione FTD sfalsata)&quot; (verde) · &quot;Acquisizione sfalsata (deriva di 100 ms)&quot; (giallo) · &quot;Configurazione troppo grande&quot; (rosso).
* **Stato del frame**: «x,xx % incompleto» — verde se inferiore all’1 %, giallo se inferiore al 5 %, rosso al 5 % o più.
* **Linea di collegamento**: «NIC {mbps} Mbps - sostenuta {MB/s} MB/s».

Si tratta del budget di larghezza di banda in tempo reale dell’array. Per i valori fps sottostanti e il modello di rete — e cosa modificare quando il livello diventa giallo o rosso — consultare [Array multicamera](arrays.md) e il [Riferimento CLI](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Pannello &quot;Calcolatore dell’indice&quot;

La terza pagina della barra laterale, condivisa dall’ingranaggio “Indice” per singola telecamera e dall’ingranaggio “Indice” per array combinato (uno alla volta — l’intestazione recita “Calcolatore dell’indice — <camera name="">” o “Calcolatore dell’indice —<array name="">

”). Riceve l’elenco delle bande (le bande naturali del filtro della telecamera o tutte le bande dei membri dell’array), l’espressione corrente e la configurazione della LUT (on/off, livello — predefinito 3, min — predefinito 0,2, max — predefinito 1), oltre a un istogramma dell’indice in tempo reale. **Applica** conferma l’espressione; le modifiche alla LUT si applicano in tempo reale all’anteprima.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Impostazioni per singola telecamera vs. gestite dall’array

Riferimento rapido su cosa si trova dove quando una telecamera è membro di un array:

| Gestito dall’array (sola lettura nel pannello della telecamera) | Ancora per singola telecamera all’interno di un array |
| --- | --- |
| Formato pixel, risoluzione, binning | Esposizione automatica (esposizione, guadagno, target, smussamento, ROI) |
| Modalità/fonte di trigger, frequenza fotogrammi | Riduzione del rumore, nitidezza, vignettatura |
| | Orientamento (specchio/rotazione), sovrapposizioni di visualizzazione, esposimetro spot |
| | Indice (array con visualizzazione separata), associazione del sensore di luce |

Altri comportamenti trasversali:

* **Visualizzazione combinata vs separata** viene scelta al momento della connessione dell’array: combinata = un’unica tessera composita allineata (i feed dei membri solo tramite Split View); separata = ogni membro rende la propria tessera sincronizzata. Una telecamera non mostra mai contemporaneamente un feed autonomo e una tessera dell’array.
* **Riconnessione automatica**: l’apertura di un progetto salvato ripristina le telecamere e le matrici e riapplica tutte le impostazioni salvate al backend prima che i flussi riprendano.
* **Controllo dell’acquisizione**: le telecamere nascoste o in pausa sono escluse dalla funzione “Capture All”; un array viene completamente bloccato solo quando TUTTI i membri sono nascosti o in pausa. Vedi [Impostazioni e modalità di acquisizione](capture.md).

## Come vengono conservate le impostazioni

Lo stato della scheda &quot;Telecamere&quot; viene salvato **insieme al progetto**, non nel browser:

* Ogni modifica reattiva crea un&#x27;istantanea delle telecamere e degli array nel file `cameras.json` del progetto (con un tempo di ritardo di 500 ms). Ciò include i nomi e i colori delle telecamere, le impostazioni di esposizione/guadagno/AE, il formato pixel/risoluzione/binning, la frequenza di trigger, le impostazioni di anteprima (risoluzione di rendering, riduzione del rumore, nitidezza, vignettatura, profilo colore, saturazione/contrasto), orientamento, sovrapposizioni, divisioni dei canali, configurazione dell’indice, impostazioni AE predittive, ROI AE, nomi degli array, modalità di visualizzazione, impostazioni di acquisizione degli array (compresa la posizione di ritaglio della ROI) e il blocco della griglia (zoom del feed, modalità di visualizzazione, blocco griglia, ordine manuale delle tessere, telecamere nascoste, tessere chiuse, telecamera attiva).
* Le associazioni dei sensori di luce vengono salvate nel file `sensors.json` del progetto.
* La riapertura del progetto ricollega l’hardware e ne riapplica tutte le impostazioni.
* **Nessun progetto aperto = solo sessione**: in assenza di un progetto, nulla viene conservato alla chiusura di Chloros.
* Solo sessione indipendentemente dal progetto: stato di pausa, campioni dello spotmeter, casella di controllo “Calibration Target” per ciascuna telecamera (sempre disattivata all’apertura) e il profilo di allineamento dell’array (ricalcolato per ogni sessione per impostazione predefinita).
* Un&#x27;eccezione: le selezioni di esportazione delle **Impostazioni di acquisizione** e la modalità di acquisizione permangono per ogni progetto nella memoria locale dell&#x27;app anziché in `cameras.json` — vedi [Impostazioni e modalità di acquisizione](capture.md).</array></camera></serial>
