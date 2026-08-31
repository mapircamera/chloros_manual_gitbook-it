# Matrici multicamera

Una **matrice**LATTICE è costituita da due o più telecamere LATTICE collegate come un’unica unità sincronizzata. Una telecamera funge da**master**: invia un impulso di trigger GPIO hardware su una linea di sincronizzazione condivisa (di default**Line2**), in modo che ogni telecamera riprenda lo stesso istante. Chloros aggiunge la sincronizzazione temporale PTP, un’anteprima in tempo reale (tessere per singola telecamera o un unico composito multibanda allineato) e l’acquisizione sincronizzata: ogni ciclo di acquisizione produce un**gruppo di fotogrammi** in cui tutte le telecamere condividono lo stesso timestamp e lo stesso ID fotogramma (riportato come `fid:N` nell’output di acquisizione).

Gli array sono il modo in cui le telecamere mono (M3M) producono indici di vegetazione: una telecamera contribuisce con una banda e l’array le allinea in uno stack multibanda. Vedi [Telecamere mono e indici di vegetazione](mono-indices.md).

Esistono tre modi equivalenti per collegare un array, e tutti eseguono lo stesso flusso “smart-prep”:

| Interfaccia | Punto di accesso |
| --- | --- |
| GUI | Scheda “Telecamere” → **Collega array** (pulsante blu) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (primo numero di serie = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (primo numero di serie = master) |

Smart-prep esegue, in ordine: un test di funzionalità di rete (ping ICMP DF + sonda GVSP), la selezione del livello di sincronizzazione, la riduzione automatica delle dimensioni del frame per adattarle alla linea, l’abilitazione del PTP, la selezione automatica del formato pixel per ciascuna telecamera, l’impostazione iniziale dell’esposizione automatica in base allo stato salvato di ciascuna telecamera e la configurazione del trigger GPIO sulla Linea 2.

{% hint style="info" %}
Le telecamere devono essere raggiungibili sul collegamento prima che qualsiasi di queste operazioni funzioni — consultare [Collegamento delle telecamere](connecting.md) per il rilevamento, l’indirizzamento e il download della calibrazione al primo collegamento. Per le configurazioni con più telecamere, le impostazioni del receive-ring della scheda di rete host sono importanti tanto quanto la velocità del collegamento; la tabella completa dei sintomi e delle soluzioni si trova nella sezione [Riferimento CLI § Configurazione e ottimizzazione della scheda di rete host](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## La finestra di dialogo «Connetti array»

La scheda «Telecamere» → **Connetti array**apre una procedura guidata in tre passaggi:**Seleziona → Modalità di visualizzazione → Impostazioni**.

### Passaggio 1 — Seleziona master e slave



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->La finestra di dialogo esegue una scansione della rete non appena viene aperta (“Scansione della rete…”), quindi verifica il cablaggio del trigger GPIO (“Verifica cablaggio GPIO…”). Sono necessarie almeno **2 telecamere** per creare un array.

La verifica del cablaggio precompila la selezione dei ruoli quando possibile e riporta uno dei tre banner seguenti:

| Messaggio | Significato |
| --- | --- |
| &quot;Master GPIO rilevato — selezioni precompilate&quot; (verde) | Il test ha individuato la topologia di trigger; le caselle di controllo relative al master e agli slave sono già selezionate. |
| &quot;Nessun master rilevato — controllare il cavo GPIO&quot; (arancione) | Nessuna telecamera ha rilevato un impulso di trigger; controllare il cablaggio di sincronizzazione. È comunque possibile selezionare manualmente i ruoli. |
| &quot;Nessun cavo di sincronizzazione: {seriali}&quot; (arancione) | Le telecamere elencate non hanno alcun cavo di sincronizzazione collegato. |

La tabella delle telecamere presenta le colonne **Telecamera / Seriale / IP / Master (radio) / Slave (casella di controllo)**:

* Selezionare esattamente **un master**e**uno o più slave**. Facendo nuovamente clic sulla radio del master corrente, questa viene deselezionata.
* Una telecamera contrassegnata con **&quot;Nessun cavo di sincronizzazione&quot;** non può mai essere selezionata come slave: uno slave senza cablaggio di trigger rimarrebbe in attesa sulla linea di sincronizzazione all’infinito e fornirebbe un segnale inattivo. Collegare invece quella telecamera come telecamera autonoma.
* Le telecamere già collegate in modalità autonoma *non* vengono disabilitate: il collegamento in array rilascia la sessione autonoma e riapre la telecamera all’interno dell’array.

**Avanti: Modalità di visualizzazione →**si abilita una volta scelti un master e almeno uno slave.**Riscansiona** riesegue il rilevamento e la verifica del cablaggio.

{% hint style="warning" %}
**Annulla** è disabilitato mentre è in corso una scansione o un test di cablaggio — l’annullamento a metà del test di cablaggio può causare il crash dell’SDKe della telecamera sul firmware LATTICE. Attendere il completamento dell’indicatore di caricamento.
{% endhint %}

### Passaggio 2 — Modalità di visualizzazione



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->| Modalità | Cosa si ottiene |
| --- | --- |
| **Telecamere separate** | Un riquadro live per ogni telecamera, tutti attivati contemporaneamente in modo che i fotogrammi rimangano sincronizzati. Ogni telecamera mantiene il proprio colore e le proprie impostazioni. |
| **Telecamere combinate** *(impostazione predefinita)* | Un unico riquadro che riproduce il composito multibanda allineato NDVI /index. Le telecamere condividono il colore dell’array. |

La modalità di visualizzazione modifica solo la presentazione dell’anteprima in tempo reale; il comportamento di acquisizione rimane lo stesso in entrambe le opzioni.

### Passaggio 3 — Impostazioni dell’array e risultato

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->previsto

All’accesso a questa scena, Chloros richiede al backend una **raccomandazione**e applica automaticamente una combinazione di ROI e binning che si adatta all’anello di ricezione della NIC (preferisce il binning al ritaglio ROI, poiché il binning preserva l’intero campo visivo). Ogni modifica apportata riavvia l’analisi in tempo reale e aggiorna il pannello**Risultato previsto** sulla destra.

Colonna di sinistra — impostazioni:

| Controllo | Opzioni | Predefinito | Note |
| --- | --- | --- | --- |
| **ROI (Campo visivo)** | Intero (2048×1536) / Metà (1024×768) / Un quarto (512×384) | Intero | Ritaglio del sensore: ritaglio a metà o a un quarto in una regione più piccola con pitch nativo dei pixel. |
| **Binning** | 1× / 2× (somma 2×2) / 4× (somma 4×4) | 1× | Binning hardware: 2×2 = campo visivo completo a un quarto del costo di elaborazione; 4×4 = campo visivo completo a 1/16. Nascosto se le telecamere non supportano il binning. |
| **Immagine sul cavo** (lettura) | — | — | Larghezza × altezza post-binning effettivamente inviate sul cavo, arrotondate ai multipli di 16 (minimo 64). |
| **Risoluzione dei pin**| casella di controllo | disattivata | Chloros normalmente attiva automaticamente il binning alla connessione quando la frequenza prevista scende al di sotto di**1,5 fps**. Il pinning mantiene la dimensione del fotogramma scelta e accetta la frequenza inferiore — trasformando una configurazione sovraccarica in un rifiuto categorico della connessione anziché in una riduzione automatica della frequenza. |
| **Frequenza di trigger** | 0,5–60 fps, passo 0,1 | vuoto = auto | La frequenza di attivazione del trigger del master. Lasciare vuoto per consentire a Chloros di derivarla. |
| **Wire Budget**| 20–2000 MB/s, passo 10 | vuoto = auto | La quantità che l’host può effettivamente assorbire, in MB/s —**l’unico valore da cui dipende l’intera allocazione dell’array.** Rilevato automaticamente dalla scheda di rete. Abbassarlo se l’array segnala frame danneggiati: il valore rilevato sovrastima le capacità delle schede USB e degli switch condivisi. Modificandolo, la proiezione viene rieseguita in tempo reale. |

Colonna di destra — **Risultato previsto**:

* **Livello di sincronizzazione** — &quot;Acquisizione simultanea&quot; (verde), &quot;Acquisizione simultanea (emissione sfalsata FTD)&quot; (verde), “Acquisizione sfalsata (deriva di 100 ms)” (ambra) o “Configurazione troppo grande” (rosso).
* **Proiezione fps** — mostrata come intervallo (“da scuro → a chiaro”), poiché la frequenza di un array sincronizzato è limitata dall’esposizione della telecamera più lenta.
* **Riga NIC** — velocità di collegamento e throughput sostenuto (“NIC {mbps} Mbps · sostenuto {N} MB/s”).
* **Verifica burst Sim-emit** — la NIC dell’host è in grado di assorbire un burst simultaneo proveniente da tutte le telecamere (&quot;Burst Sim-emit: X MB · Anello NIC utilizzabile: Y MB ✓/✗&quot;).
* **Controllo del budget di banda** — domanda aggregata in condizioni di regime rispetto al limite massimo di banda a prova di collisione (&quot;Budget di banda: {domanda} MB/s richiesti da {n} telecamere · limite massimo {limite} MB/s ✓/✗ sovrasottoscritto&quot;).
* **&quot;Numero massimo di telecamere su questa linea: {n} — determinato dalla larghezza di banda minima per telecamera, quindi il raggruppamento non lo aumenta.&quot;** — visualizzato quando ci si avvicina (o si supera) il limite massimo del numero di telecamere.
* **&quot;CON QUESTE IMPOSTAZIONI SI VERIFICHERANNO PERDITE DI FRAME.&quot;**— avviso rosso con la motivazione fornita dal backend, oltre a un elenco di ostacoli e**suggerimenti di risoluzione** in blu (&quot;Per far entrare questo array nella rete&quot; / &quot;Per sbloccare l’acquisizione simultanea&quot;).**Applica e connetti** rimane disabilitato finché non esiste una proiezione, e la sua etichetta indica il motivo del rifiuto:

| Etichetta del pulsante | Significato | Cosa aiuta effettivamente |
| --- | --- | --- |
| &quot;Analisi in corso...&quot; | Analisi ancora in corso. | Attendere. |
| **&quot;Troppe telecamere per questa rete&quot;**| L’array sovraccarica la linea (controllo dell’aggregato fallito). | Meno telecamere, jumbo frame end-to-end o una scheda di rete più veloce.**Un ROI più piccolo NON sarà d’aiuto** — vedi sotto. |
| **&quot;Ridurre l’area di interesse (ROI) per abilitare&quot;** | Con queste impostazioni i frame andrebbero persi (controllo burst/ring fallito). | Ridurre l’area di interesse (ROI), aumentare il binning o riparare l’anello di ricezione della scheda di rete (NIC). |



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->Durante la connessione, potrebbe apparire un **pannello di download della calibrazione** verde con una barra di avanzamento per ogni porta seriale: la prima volta che una telecamera viene collegata a un computer, Chloros scarica il pacchetto di calibrazione di fabbrica da circa 3,8 MB dalla telecamera tramite GigE (circa 70 secondi per telecamera). Le telecamere già memorizzate nella cache non mostrano mai questo pannello. Vedere [Collegamento delle telecamere](connecting.md).

## Larghezza di banda: quante telecamere è possibile collegare

La capacità di un array dipende dalla linea di trasmissione, non dChloros; pertanto, i dati di progettazione sono riportati nel manuale dell’hardware: **[Progettazione della larghezza di banda dell’array](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Cosa ne fa Chloros: la finestra di dialogo di connessione esegue un test di rete, calcola la frequenza dei fotogrammi raggiungibile e seleziona un livello adeguato. Se l’array sovraccarica la linea, rifiuta la connessione anziché scartare i pacchetti in modo silenzioso — si veda il pannello dei risultati previsti descritto sopra.

## Quando i frame vanno persi

Una telecamera può essere assente da un gruppo pubblicato per due motivi completamente diversi,
e questi richiedono soluzioni opposte. Chloros li conta separatamente anziché riportare un unico
numero “incompleto” che non specifica nessuno dei due:

| Cosa è successo | Cosa significa | Dove cercare |
| --- | --- | --- |
| **Corrotto**— il fotogramma è arrivato ma era strutturalmente danneggiato | Perdita di pacchetti GVSP sul percorso di rete | Il**wire budget**, l’anello di ricezione della scheda di rete, i jumbo frame, lo switch |
| **Mai arrivato**— non è arrivato alcun frame | La telecamera non si è attivata, oppure non è stato trasmesso nulla | Il**cavo di sincronizzazione M8**, la linea di sincronizzazione, se tutti i membri sono attivi |

La suddivisione viene rivalutata ogni 10 secondi mentre l’array trasmette in streaming. Se supera il 5%, viene
registrato indicando entrambi i valori, e ogni buffer danneggiato viene segnalato la prima volta che
si verifica per ciascuna telecamera, poi aggregato una volta al minuto in modo che una sessione lunga rimanga leggibile.

**I fotogrammi danneggiati con zero &quot;mai arrivati&quot; indicano che l’attivazione e la sincronizzazione via cavo sono perfette**e che ogni fotogramma perso si trova sul percorso di rete. La soluzione consiste nel ridurre il**Wire Budget** e
riconnettersi.

{% hint style="warning" %}
**Ridurre la frequenza di trigger non risolve il problema dei fotogrammi danneggiati.** Il
pacing dei pacchetti della telecamera viene impostato una sola volta, al momento della connessione. Ridurre la frequenza di trigger modifica la frequenza con cui si verifica un burst,
non la velocità con cui il burst stesso viene trasmesso sulla linea. Su un sistema testato con 4 telecamere, una
riduzione di 5 volte della frequenza di trigger non ha cambiato nulla, mentre abbassare il wire budget da 240 a
200 MB/s ha portato lo stesso sistema dal 10,4% di frame corrotti a zero.
{% endhint %}

Un array in esecuzione non può ripianificarsi autonomamente: è necessario disconnettersi e riconnettersi affinché il selettore del tempo di connessione
possa operare in base al nuovo budget.

### Gli adattatori di rete USB hanno un limite massimo di 200 MB/s

Un adattatore Ethernet USB dichiara la propria velocità di collegamento *Ethernet*, ma ciò che può effettivamente
sostenere è limitato dal bus USB e dal relativo driver. Un dongle USB 10GbE era considerato
di circa 1000 MB/s di throughput — un valore che nessuno aveva mai misurato — e il regolazione della velocità
di quattro telecamere in base a quel margine di riserva fittizio ha corrotto il 6–18 % dei fotogrammi, mentre l’array
continuava a segnalare una frequenza dei fotogrammi target corretta. Gli adattatori collegati tramite USB sono ora limitati a
**200 MB/s**. Il limite è un valore assoluto piuttosto che una percentuale, poiché il vincolo è rappresentato dal
bus: un adattatore USB da 1 GbE raggiunge circa 80 MB/s e non ne risente.

Se il vostro host è effettivamente più veloce del limite, aumentate il **Wire Budget** per indicarlo.

## Sincronizzazione temporale PTP

La *sincronizzazione* dei fotogrammi deriva dal trigger hardware; il **PTP** (IEEE 1588 PTPv2) fornisce *timestamp* comparabili su ogni dispositivo. È abilitato per impostazione predefinita al momento della connessione dell’array:

* Il **backend host Chloros esegue il grandmaster PTP**. Le telecamere LATTICE e i sensori di luce DAQ-E si sincronizzano con esso nel dominio 0, in modo che i timestamp delle immagini e gli spettri DAQ siano allineati su un unico clock (~1 ms).
* `--no-ptp` (CLI) lo disabilita per il lavoro da banco — i timestamp tra le telecamere **non** sono quindi comparabili.
* Verificare lo stato della sincronizzazione con l’CLI ():

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

La scheda “Telecamere” non presenta alcun indicatore PTP; le informazioni di sincronizzazione per ciascuna telecamera disponibili in questa scheda sono il **Ruolo**(Master/Slave) in sola lettura, la**Linea di sincronizzazione** e il livello di Capacità dell’array. Lo stato PTP del DAQ-E è visualizzato nei dettagli del sensore nella scheda “Sensori di luce”.

## La vista

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->in tempo reale dell’array

L’area principale del feed offre due layout (commutabili nella barra superiore): **vista a griglia**(ogni riquadro è una cella; trascinare per riordinare quando il lucchetto della griglia è sbloccato) e**vista a elenco**(array a tutta larghezza in alto, una telecamera attiva sotto). Il cursore**Zoom feed** regola le dimensioni delle tessere; se la larghezza della cella è inferiore a 200 px, le sovrapposizioni del nome e degli fps si nascondono automaticamente.

La **modalità separata** mostra una tessera per ogni telecamera. Ogni riquadro mostra:

* il nome della telecamera (in alto a sinistra),
* un **valore degli fps** (in alto a destra) — si tratta della *frequenza di acquisizione effettiva* della telecamera riportata dal backend, non della frequenza di campionamento dell’anteprima (l’anteprima live è limitata a 30 fps indipendentemente dalla frequenza di acquisizione),
* un puntino di stato — verde (in streaming) / ambra (in caricamento) / rosso (errore),
* un **indicatore di frame obsoleto** quando non arriva alcun frame aggiornato da 2 s — normale per circa 5 s dopo qualsiasi connessione/disconnessione, mentre il backend riequilibra la larghezza di banda tra le telecamere.

La **modalità combinata**mostra un unico riquadro composito: il backend esegue il debayering, il ridimensionamento, l’allineamento, la riduzione del rumore, la conversione in radianza per banda (più la riflettanza DLS quando è associato un sensore di luce), valuta l’espressione dell’indice dell’array, applica la LUT e trasmette il risultato in formato MJPEG. Fino a quando non viene renderizzato il primo fotogramma allineato, il riquadro ne spiega lo stato: «Preparazione dell’array…», «Calibrazione dell’allineamento…», «In attesa del primo fotogramma…» oppure — se il tempo a disposizione per i tentativi di allineamento automatico (~30 s) è esaurito — «Allineamento richiesto» con un pulsante**Calibra allineamento**.

Informazioni utili sulla modalità combinata:

* Il composito è allineato al fotogramma della telecamera **master**. Il puntamento AE-ROI e la misurazione spot sul composito sono esatti per la telecamera master e approssimativi per quelle slave; utilizzare**Split View** (impostazioni dell’array → “Mostra telecamere associate”) per ottenere riquadri per telecamera con precisione al pixel senza aprire connessioni aggiuntive con le telecamere.
* **Display Layers**(impostazioni array; disattivato di default) consente di scegliere un livello di primo piano e uno di sfondo — qualsiasi telecamera membro o**Indice**. Con primo piano = Indice, i pixel al di fuori dei valori Min/Max della LUT mostrano il livello di sfondo.
* **Risoluzione di rendering** (impostazione predefinita 720p) imposta l’altezza dello streaming live *e* le dimensioni di esportazione del composito salvato. Le immagini per singola telecamera vengono sempre esportate a piena risoluzione.
* L’allineamento viene calcolato per ogni sessione e non viene mai memorizzato — consulta la sezione dedicata all’allineamento nel pannello delle impostazioni dell’array per i residui RMS e il pulsante Ricalibra.

## Acquisizione: monitoraggio vs analisi

Le superfici di acquisizione dell’array si dividono nettamente in **livello di monitoraggio**(registra ciò che vedi) e**livello di analisi** (registra dati grezzi, calibra in seguito):

| Flusso di lavoro | Livello | Cosa salva | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Acquisizione**(immagini fisse) | Analisi | Un gruppo di fotogrammi sincronizzati per ogni passaggio; file per ogni telecamera a ogni livello di esportazione selezionato (grezzo/debayering/radianza/riflettanza/anteprima/indice) + `.daq` sidecar | Pulsante**Acquisisci tutto** + Impostazioni di acquisizione | `lattice array-capture` |
| **Registra video indice** | Monitoraggio | Il composito dell’indice combinato in tempo reale così come visualizzato — 8 bit, risoluzione di anteprima, LUT integrata; richiede che lo streaming live sia aperto | ● Registra video indice (array combinati) | `lattice array-record` |
| **Burst grezzo → crea video**| Analisi | Fotogrammi grezzi del sensore alla massima frequenza di acquisizione + manifesto + `.daq`, quindi ricostruzione offline in video calibrato di radianza / riflettanza / indice, sincronizzato temporalmente con le letture DAQ | ⦿ Registra burst grezzo →**Crea video** | `lattice array-burst` → `lattice array-build-video` |

Regola generale: se i pixel forniranno *misurazioni*, utilizzare la modalità “Acquisizione” o “Burst” (livello di analisi); se devi semplicemente *guardare o mostrare* ciò che l’array ha rilevato, registra il video dell’indice (di livello di monitoraggio).

### Impostazioni di acquisizione (GUI)



<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->L’icona a forma di ingranaggio accanto a **Acquisisci tutto** apre il pannello delle impostazioni di acquisizione (richiede un progetto aperto — le acquisizioni vengono salvate al suo interno):

* **Modalità di acquisizione**:**Singola**(un solo passaggio) /**Continuativa**(in sequenza; limitata da un numero di acquisizioni, valore predefinito 1, o da una durata, valore predefinito 10 s) /**A intervalli** (timelapse: N acquisizioni ogni X intervalli per un totale di Y, impostazione predefinita 1 ogni 5 s per 1 minuto).
* **Tipi di esportazione per telecamera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — tutte le opzioni applicabili sono attive per impostazione predefinita. Radiance/Reflectance sono nascoste per le telecamere con filtro &quot;RGB&quot;;**Reflectance appare solo quando la telecamera dispone di un sensore di luce DAQ** (proprio o ereditato dall’array); Index richiede un’espressione di indice configurata.
* **Allineato**(per array, predefinito**attivo**): deforma le esportazioni dei membri in base al profilo di allineamento dell’array, in modo che le esportazioni siano registrate a livello di pixel. Il formato Raw rimane sempre non deformato, ma porta la trasformazione nei metadati.
* **Acquisizione più veloce** (interruttore): solo raw + la lettura DAQ assegnata + il composito a indice combinato gratuito, saltando i calcoli di calibrazione al momento dell’acquisizione per ottenere la massima velocità — ricostruire successivamente radianza/riflettanza/indice dal file `.daq` salvato.
* Le selezioni rimangono attive con il progetto. Le telecamere nascoste o in pausa vengono ignorate.

L’equivalente CLI (stesso endpoint di backend, stessa semantica):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

La compressione TIFFe per le acquisizioni è `deflate` (senza perdita di dati, predefinita) o `none` — le tabelle complete dei flag, la struttura della cartella di acquisizione e le regole di rielaborazione sono riportate nel [Riferimento CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Accoppiamento di un sensore di luce DAQ

Le anteprime con correzione della riflettanza e dell’illuminazione richiedono i dati sulla luce in discesa provenienti da un sensore DAQ (collegato nella scheda **Sensori di luce**):

* La **riga dell’array**nella barra laterale mostra un**pulsante &quot;DAQ · on/off&quot;** — *on* quando è impostato un sensore di luce a livello di array **oppure** una qualsiasi telecamera membro ne possiede uno proprio; il relativo tooltip elenca esattamente quale sensore alimenta quale telecamera.
* Assegnare a livello di array nelle impostazioni dell’array → **Sensore di luce ambientale**→ menu a tendina**Sensore di luce**. La selezione rimane attiva con il progetto, si applica a tutte le telecamere del gruppo e le singole telecamere possono comunque sovrascriverla con il proprio sensore.
* La riga di stato sottostante riporta lo stato in tempo reale: **Off**→ &quot;In attesa del primo spettro…&quot; →**&quot;Attivo — tutte le telecamere dell’array sono sottoposte a correzione dell’illuminazione&quot;** → oppure, se non è arrivato alcun nuovo spettro negli ultimi 3 s, un avviso di dati non aggiornati — continua a essere utilizzata l’ultima lettura (le letture non scadono mai nel percorso di acquisizione).

Con un sensore assegnato: il tipo di esportazione «Riflettanza» diventa disponibile, le anteprime in tempo reale sono corrette per l’illuminazione, l’esposizione automatica predittiva può utilizzare lo spettro e ogni acquisizione di riflettanza scrive la lettura DAQ effettivamente utilizzata come **`.daq` sidecar** accanto all’immagine, in modo che l’acquisizione possa essere rielaborata in un secondo momento.

## `array-connect` Opzioni di CLI

| Flag | Predefinito | Descrizione |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Rilevamento automatico di tutte le telecamere LATTICE (ne occorrono ≥2) | **La prima porta seriale è il MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Linea di sincronizzazione GPIO. |
| `--target-fps F` | auto | Frequenza di attivazione del trigger master. |
| `--binning {1,2,4}` | auto | Binning hardware. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Sovrascrittura da parte di un esperto del selettore del livello di sincronizzazione. |
| `--wire-ceiling-mbps MB_PER_S` | rilevato automaticamente | Budget di banda host in MB/s — la forma CLI del campo **Wire Budget**. Ridurlo se l’array segnala frame danneggiati. Viene salvato con il progetto, quindi una successiva riconnessione lo ripristina. |
| `--no-recommend` | disattivato | Salta la fase di analisi della rete. |
| `--no-ptp` | disattivato | Disattiva il PTP (i timestamp tra le telecamere non saranno quindi comparabili). |

`lattice array-list`, `array-status` e `array-disconnect` gestiscono la sessione persistente. Il riferimento completo ai sottocomandi, compreso l’allineamento (`align-calibrate` / `align-apply`) e degli strumenti di rete, è disponibile nel [Riferimento a CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); gli equivalenti in SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) si trovano nel [SDK Reference](../reference/sdk-reference.md). Da Python il budget dei cavi è `connect_array(..., wire_ceiling_mbps=120)`, mentre la suddivisione tra cavi in funzione, danneggiati o mai arrivati si trova su [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
