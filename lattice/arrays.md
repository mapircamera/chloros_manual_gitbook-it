# Matrici multicamera

Una **matrice**LATTICE è costituita da due o più telecamere LATTICE collegate come un’unica unità sincronizzata. Una telecamera funge da**master**: invia un impulso di trigger GPIO hardware su una linea di sincronizzazione condivisa (per impostazione predefinita**Line2**), in modo che ogni componente scatti nello stesso istante. Chloros aggiunge la sincronizzazione temporale PTP, un’anteprima in tempo reale (tessere per singola telecamera o un unico composito multibanda allineato) e l’acquisizione sincronizzata: ogni ciclo di acquisizione produce un **gruppo di fotogrammi** in cui tutte le telecamere condividono lo stesso timestamp e lo stesso ID fotogramma (riportato come `fid:N` nell’output di acquisizione).

Gli array sono il modo in cui le telecamere mono (M3M) producono indici di vegetazione: una telecamera contribuisce con una banda e l’array le allinea in uno stack multibanda. Vedi [Telecamere mono e indici di vegetazione](mono-indices.md).

Esistono tre modi equivalenti per collegare un array, e tutti eseguono lo stesso flusso “smart-prep”:

| Interfaccia | Punto di accesso |
| --- | --- |
| GUI | Scheda Telecamere → **Collega array** (pulsante blu) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (primo numero di serie = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (primo numero di serie = master) |

Smart-prep esegue: un test delle capacità di rete (ping ICMP DF + sonda GVSP), la selezione del livello di sincronizzazione, una proiezione della frequenza dei fotogrammi alla dimensione richiesta, la decisione relativa al trasporto compresso a 12 bit (vedi [Trasporto compresso a 12 bit](#packed-12-bit-transport)), l’abilitazione del PTP, la selezione automatica del formato pixel per ciascuna telecamera, l’impostazione iniziale dell’esposizione automatica in base allo stato salvato di ciascuna telecamera e la configurazione del trigger GPIO sulla Linea 2.

**Sensore completo per impostazione predefinita (novità nella versione 1.3.0).** Connect mantiene la dimensione del fotogramma richiesta — l’intero sensore a 2048×1536, senza binning né ritaglio, a meno che non si scelga diversamente — e lascia che la frequenza dei fotogrammi scenda al livello consentito dall’host e dalla rete. Non esegue mai il binning né il ritaglio di propria iniziativa (la versione 1.2.0 attivava automaticamente il binning al di sotto di 1,5 fps). Il binning o un’area di interesse (ROI) più piccola sono a tua discrezione: nella finestra di dialogo di connessione, scegline una e la sezione “Risultato previsto” mostra la frequenza dei fotogrammi che ne deriva; `array-connect` stampa le opzioni più veloci con le relative frequenze previste.

{% hint style="info" %}
Le telecamere devono essere raggiungibili sul collegamento prima che tutto ciò funzioni — vedi [Connessione delle telecamere](connecting.md) per il rilevamento, l’indirizzamento e il download della calibrazione al primo collegamento. Per le configurazioni con più telecamere, le impostazioni del receive-ring della scheda di rete host sono importanti tanto quanto la velocità del collegamento; la tabella completa dei sintomi e delle soluzioni si trova nella sezione [Riferimento CLI § Configurazione e ottimizzazione della scheda di rete host](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## La finestra di dialogo "Connetti array"

La scheda Telecamere → **Connetti array**apre una procedura guidata in tre passaggi:**Seleziona → Modalità di visualizzazione → Impostazioni**.

### Fase 1 — Selezione della telecamera master e delle telecamere slave

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



La finestra di dialogo esegue una scansione della rete non appena viene aperta (“Scansione della rete...”), quindi rileva i cavi di sincronizzazione (“Rilevamento dei cavi di sincronizzazione...”) per individuare quali telecamere sono collegate alla linea di sincronizzazione M8. Sono necessarie almeno **2 telecamere** per creare un array.

I cavi di sincronizzazione M8 convergono in un unico punto di giunzione, quindi **qualsiasi telecamera collegata può fungere da master**. Il test preassegna i ruoli — la telecamera RGB (`-FRGB`) come master quando è collegata, altrimenti la prima telecamera collegata, e le altre telecamere collegate come slave — e visualizza uno dei tre banner seguenti:

| Banner | Significato |
| --- | --- |
| "Cavo di sincronizzazione rilevato su {n} telecamere. Qualsiasi di esse può fungere da master; la telecamera RGB (altrimenti la prima) è preselezionata." (verde) | La sonda ha rilevato le telecamere sulla linea di sincronizzazione; le caselle di controllo relative a master e slave sono già selezionate. |
| "Nessun master rilevato - controllare il cavo GPIO" (arancione) | Nessuna telecamera ha rilevato un impulso di trigger; controllare il cablaggio di sincronizzazione. I ruoli sono comunque precompilati (la telecamera RGB, altrimenti la prima telecamera nell’elenco, come master) ed è possibile modificarli. |
| "Nessun cavo di sincronizzazione: {seriali}" (arancione) | Le telecamere elencate non hanno alcun cavo di sincronizzazione collegato. |

La tabella delle telecamere presenta le colonne **Telecamera / Seriale / IP / Master (radio) / Slave (casella di controllo)**. Le righe sono elencate in**ordine di telecamera** — telecamere M3C (Bayer) prima delle M3M (mono); tra le M3C, FRGB, FRGN, FOCN, poi FNGB; le M3M in base alla lunghezza d’onda del filtro, dalla più bassa alla più alta (F450 prima di F808). L’elenco delle telecamere nella barra laterale e il layout predefinito della griglia utilizzano lo stesso ordine.

* Scegliere esattamente **un master**e**uno o più slave**. Facendo nuovamente clic sul pulsante di selezione del master corrente, la selezione viene deselezionata. Per scegliere un master diverso basta un clic: il nuovo master abbandona gli slave e il master precedente diventa uno slave (a meno che non sia privo di cavo di sincronizzazione).
* Una telecamera contrassegnata come **"Senza cavo di sincronizzazione"** non può mai essere selezionata come slave: uno slave senza cavo di trigger rimarrebbe in attesa sulla linea di sincronizzazione all’infinito e fornirebbe un segnale inattivo. Collegare invece quella telecamera come telecamera autonoma.
* Le telecamere già collegate in modalità autonoma *non* vengono disabilitate: il collegamento all’array interrompe la sessione autonoma e riapre la telecamera all’interno dell’array.

**Successivo: Modalità di visualizzazione →**si abilita una volta scelti un master e almeno uno slave.**Riscansiona** riesegue il rilevamento e la verifica del cablaggio.

{% hint style="warning" %}
**Annulla** è disabilitato mentre è in corso una scansione o un test di collegamento — l’annullamento a metà del test può causare il crash dell’SDK della telecamera sul firmware delle telecamere LATTICE. Attendere il completamento dell’indicatore di caricamento.
{% endhint %}

### Passaggio 2 — Modalità di visualizzazione



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Modalità | Cosa si ottiene |
| --- | --- |
| **Telecamere separate** | Un riquadro live per ogni telecamera, tutti attivati contemporaneamente in modo che i fotogrammi rimangano sincronizzati. Ogni telecamera mantiene il proprio colore e le proprie impostazioni. |
| **Telecamere combinate** *(impostazione predefinita)* | Un unico riquadro che riproduce il composito multibanda NDVI/indice allineato. Le telecamere condividono il colore dell’array. |

La modalità di visualizzazione modifica solo la presentazione dell’anteprima in tempo reale — il comportamento di acquisizione è lo stesso in entrambe.

### Passaggio 3 — Impostazioni dell’array e risultato

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

previsto

All’accesso a questa scena, Chloros richiede al backend una **raccomandazione**e la applica: il**sensore completo — senza binning né ritaglio ROI — ogni volta che la frequenza dei fotogrammi lo consente**, a qualsiasi velocità fornita dall’host e dalla connessione. Solo quando nessuna frequenza è in grado di supportare il sensore completo, la raccomandazione riduce l’inquadratura, e in tal caso preferisce il binning al ritaglio ROI, poiché il binning preserva l’intero campo visivo. Da quel momento in poi si scambia la risoluzione con la frequenza dei fotogrammi. Ogni modifica apportata riavvia l’analisi in tempo reale e aggiorna il pannello**Risultato previsto** sulla destra.

Colonna di sinistra — impostazioni:

| Controllo | Opzioni | Predefinito | Note |
| --- | --- | --- | --- |
| **ROI (Campo visivo)** | Intero (2048×1536) / Metà (1024×768) / Un quarto (512×384) | Intero | Ritaglio del sensore: ritaglio a metà o a un quarto in una regione più piccola con passo dei pixel nativo. |
| **Binning** | 1× / 2× (somma 2×2) / 4× (somma 4×4) | 1× | Binning hardware: 2×2 = campo visivo completo a un quarto del costo di trasmissione; 4×4 = campo visivo completo a 1/16. Nascosto se le telecamere non supportano il binning. |
| **Immagine sul cavo** (lettura) | — | — | Larghezza×altezza post-binning effettivamente inviate sul cavo, arrotondate ai multipli di 16 (minimo 64). |
| **Frequenza di trigger** | 0,5–60 fps, passo 0,1 | vuoto = auto | La frequenza di attivazione del trigger del master. Lasciare vuoto per consentire a Chloros di calcolarlo automaticamente. |
| **Budget di banda**| 20–2000 MB/s, passo 10 | vuoto = auto | La quantità di dati che l’host può effettivamente assorbire, in MB/s —**l’unico valore da cui dipende l’intera allocazione dell’array.** Rilevato automaticamente dalla scheda di rete. Abbassarlo se l’array segnala frame danneggiati: il valore rilevato sovrastima le capacità degli adattatori USB e degli switch condivisi. Modificandolo si riesegue la proiezione in tempo reale. Il campo si apre con il budget con cui era collegato l’ultimo array (salvato con il progetto), quindi un budget corretto una volta viene mantenuto. |

La casella di controllo **Risoluzione dei pin** presente dalla versione 1.2.0 è stata rimossa: la connessione mantiene sempre la dimensione del frame scelta e riduce invece la frequenza dei frame.

Colonna di destra — **Risultato previsto**:

* **Livello di sincronizzazione** — "Acquisizione simultanea" (verde), "Acquisizione simultanea (emissione sfalsata FTD)" (verde), "Acquisizione sfalsata (deriva di 100 ms)" (ambra) o "Configurazione troppo grande" (rosso). Chloros sceglie il livello: acquisizione simultanea ogni volta che la linea può essere condivisa a una velocità alla quale le telecamere possono essere sincronizzate (la frequenza dei fotogrammi diminuisce man mano che la quota di ciascuna telecamera si riduce), e acquisizione sfalsata solo quando ciò non è possibile — in questo modo un array si connette su qualsiasi rete, alla frequenza dei fotogrammi consentita da quella rete.
* **Proiezione fps** — la frequenza dei fotogrammi alla quale si prevede che l’array funzioni. Con l’acquisizione simultanea si tratta di un unico valore: la frequenza viene prima, e l’esposizione automatica riempie l’intervallo che essa definisce, quindi aumenta il guadagno, in modo che una scena poco illuminata non lo riduca (vedi [Come l’esposizione automatica gestisce esposizione e guadagno](camera-settings.md)). Con l’acquisizione scaglionata viene indicato come un intervallo (“scuro → luminoso”), poiché l’esposizione di ciascuna telecamera allunga lo scaglionamento in una scena buia.
* **Linea NIC** — velocità di collegamento e throughput sostenuto (“NIC {mbps} Mbps · sostenuto {N} MB/s”).
* **Verifica burst Sim-emit** — la scheda di rete (NIC) dell’host è in grado di assorbire un burst simultaneo proveniente da tutte le telecamere (“Burst Sim-emit: X MB · Anello NIC utilizzabile: Y MB ✓/✗”).
* **Controllo del budget di banda** — domanda aggregata in condizioni di regime rispetto al limite massimo di banda a prova di collisione ("Budget di banda: {domanda} MB/s richiesti da {n} telecamere · limite massimo {limite} MB/s ✓/✗ sovrasottoscritto").
* **"Numero massimo di telecamere su questa linea: {n} — determinato dalla larghezza di banda minima per telecamera, quindi il raggruppamento non lo aumenta."** — visualizzato quando ci si avvicina (o si supera) il limite massimo del numero di telecamere.
* **"CON QUESTE IMPOSTAZIONI I FRAME VERRANNO PERDUTI."**— avviso rosso con la motivazione fornita dal backend, oltre a un elenco di ostacoli e**suggerimenti per la risoluzione** in blu ("Per far rientrare questo array nella rete" / "Per sbloccare l’acquisizione simultanea").**Applica e connetti** rimane disabilitato finché non esiste una proiezione, e la sua etichetta indica il motivo del rifiuto:

| Etichetta del pulsante | Significato | Cosa aiuta effettivamente |
| --- | --- | --- |
| "Analisi in corso..." | Analisi ancora in corso. | Attendere. |
| **"Troppe telecamere per questa rete"**| La linea non è in grado di gestire l’array nemmeno con una telecamera alla volta (controllo dell’aggregato fallito): il numero di telecamere è di gran lunga superiore a quello che la linea può gestire, oppure altre telecamere stanno trasmettendo sulla stessa linea. | Meno telecamere, frame jumbo end-to-end o una scheda di rete più veloce.**Un ROI più piccolo NON sarà d’aiuto** — vedi sotto. |
| **"Ridurre l’ROI per abilitare"** | Con queste impostazioni si verificherebbe la perdita di fotogrammi (controllo burst/ring fallito). | Ridurre l’ROI, aumentare il binning o correggere il ring di ricezione della scheda di rete. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Durante la connessione, potrebbe apparire un **pannello di download della calibrazione** verde con una barra di avanzamento per ogni porta seriale: la prima volta che una telecamera viene collegata a un computer, Chloros scarica il pacchetto di calibrazione di fabbrica da circa 3,8 MB dalla telecamera tramite GigE (circa 70 secondi per telecamera). Le telecamere già memorizzate nella cache non mostrano mai questo pannello; con più di sei telecamere le righe scorrono. Vedi [Collegamento delle telecamere](connecting.md).

### Trasporto compresso a 12 bit

I formati a 12 bit (`BayerRG12`, `Mono12`) vengono normalmente trasmessi come parole a 16 bit. **Il trasporto**compresso**invia gli stessi 12 bit in 1,5 byte per pixel e li decomprime sull’host, quindi l’invio di ogni fotogramma richiede il 25% di tempo in meno — a sensore pieno, circa un terzo di fotogrammi in più al secondo.**I dati salvati sono identici.** La connessione array lo decide automaticamente (`auto`): esegue il packing solo quando è vantaggioso e l’host dispone di CPU disponibile per il decomprimimento. L’interfaccia grafica non prevede un’opzione per questa impostazione; dalla CLI, utilizzare `array-connect --packed {auto,on,off}` (impostazione predefinita `auto`). I formati a 8 bit e a 16 bit non vengono mai compressi.

## Larghezza di banda: quante telecamere è possibile collegare

La capacità di un array dipende dalla linea di trasmissione, non da Chloros, quindi i dati di progettazione si trovano nel manuale dell’hardware: **[Pianificazione della larghezza di banda dell’array](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Cosa fa Chloros con questi dati: la finestra di dialogo di connessione esegue un test della rete, proietta la frequenza dei fotogrammi raggiungibile e sceglie un livello adeguato. Un array si connette su qualsiasi rete: su un collegamento lento, ogni telecamera viene regolata in base alla propria quota di banda e la frequenza dei fotogrammi diminuisce, anziché che l’array venga rifiutato o i suoi fotogrammi vengano ridotti. Solo un array che la linea non è in grado di supportare nemmeno una telecamera alla volta viene rifiutato (**Troppe telecamere per questa rete**) anziché perdere pacchetti in modo silenzioso — si veda il pannello dei risultati previsti descritto sopra.

## Quando i fotogrammi vanno persi

Una telecamera può essere assente da un gruppo pubblicato per due motivi completamente diversi,
e questi richiedono soluzioni opposte. Chloros li conta separatamente anziché riportare un unico
numero “incompleto” che non specifica nessuno dei due:

| Cosa è successo | Cosa significa | Dove cercare |
| --- | --- | --- |
| **Corrotto**— il fotogramma è arrivato ma era strutturalmente danneggiato | Perdita di pacchetti GVSP sul percorso di rete | Il**wire budget**, l’anello di ricezione della scheda di rete, i jumbo frame, lo switch |
| **Mai arrivato**— non è arrivato alcun frame | La telecamera non si è attivata, oppure non è stato trasmesso nulla | Il**cavo di sincronizzazione M8**, la linea di sincronizzazione, se tutti i membri sono attivi |

La ripartizione viene rivalutata ogni 10 secondi mentre l’array trasmette in streaming. Se supera il 5%, viene
registrata indicando entrambi i valori, e ogni buffer danneggiato viene segnalato la prima volta che
si verifica per ciascuna telecamera, poi aggregato una volta al minuto in modo che una sessione lunga rimanga leggibile.

**I fotogrammi corrotti con zero "mai arrivati" indicano che l’attivazione e la sincronizzazione via cavo sono perfette**e che ogni fotogramma perso si trova sul percorso di rete. La soluzione consiste nell’abbassare il**Wire Budget** e
riconnettersi.

{% hint style="warning" %}
**Ridurre la frequenza di trigger non risolve il problema dei fotogrammi danneggiati.** Il
pacing dei pacchetti della telecamera viene impostato una sola volta, al momento della connessione. Ridurre la frequenza di trigger modifica la frequenza con cui si verifica un burst,
non la velocità con cui il burst stesso viene trasmesso sulla linea. Su un sistema con 4 telecamere sottoposto a test, una
riduzione di 5 volte della frequenza di trigger non ha cambiato nulla, mentre abbassare il wire budget da 240 a
200 MB/s ha portato lo stesso sistema da un tasso di frame danneggiati del 10,4% a zero.
{% endhint %}

Un array in esecuzione non può ripianificarsi autonomamente: è necessario disconnettersi e riconnettersi affinché il selettore del tempo di connessione
possa operare in base al nuovo budget.

### Gli adattatori di rete USB sono limitati

Un adattatore Ethernet USB dichiara la propria velocità di collegamento *Ethernet*, ma ciò che può effettivamente
sostenere è limitato dal bus USB e dal relativo driver. A un dongle USB 10GbE venivano attribuiti
circa 1000 MB/s di throughput — un valore che nessuno aveva mai misurato — e il regolarne la velocità
di quattro telecamere in base a quel margine di riserva fantasma ha causato il danneggiamento del 6–18 % dei fotogrammi, mentre l’array
continuava a segnalare una frequenza di fotogrammi target corretta. Gli adattatori collegati via USB sono ora limitati a
**200 MB/s** su Windows e NVIDIA Jetson, e a **400 MB/s** su Linux x86_64. Il limite è un valore assoluto piuttosto che una percentuale, poiché il vincolo è rappresentato dal
bus: un adattatore USB da 1 GbE raggiunge circa 80 MB/s e non ne risente.

Se il vostro host è effettivamente più veloce del limite, aumentate il **Wire Budget** per indicarlo.

## Sincronizzazione temporale PTP

La *sincronizzazione* dei fotogrammi deriva dal trigger hardware; il **PTP** (IEEE 1588 PTPv2) fornisce *timestamp* comparabili su ogni dispositivo. È abilitato per impostazione predefinita alla connessione dell’array:

* Il **backend host Chloros esegue il grandmaster PTP**. Le telecamere LATTICE e i sensori di luce DAQ-E si sincronizzano con esso nel dominio 0, in modo che i timestamp delle immagini e gli spettri DAQ siano allineati su un unico clock (~1 ms).
* `--no-ptp` (CLI) lo disabilita per il lavoro da banco — in tal caso i timestamp tra le telecamere **non** sono comparabili.
* Verificare lo stato della sincronizzazione tramite la CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

La scheda “Telecamere” non presenta un indicatore PTP; le informazioni di sincronizzazione per ciascuna telecamera disponibili in questa scheda sono il **Ruolo**(Master/Slave) in sola lettura, la**Linea di sincronizzazione** e il livello di Capacità dell’array. Lo stato PTP del DAQ-E è visualizzato nei dettagli del sensore nella scheda “Sensori di luce”.

## La vista

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

in tempo reale dell’array L’area principale del feed offre due layout (commutabili nella barra superiore): **vista a griglia**(ogni riquadro è una cella; trascinare per riordinare quando il lucchetto della griglia è sbloccato) e**vista a elenco**(righe a larghezza intera: un array**combinato**occupa una riga — il suo riquadro composito — mentre un array**separato**assegna a**ogni membro una propria riga**, con il proprio feed in tempo reale e la propria tabella dei bersagli di calibrazione; sotto gli array, una telecamera autonoma — quella attiva). Il cursore**Zoom feed** regola le dimensioni delle tessere; se la larghezza della cella è inferiore a 200 px, le sovrapposizioni di nome/fps si nascondono automaticamente. Le telecamere componenti appaiono nella barra laterale e nel layout predefinito della griglia nell’ordine delle telecamere (M3C prima di M3M, prima le FRGB, poi le monocromatiche in ordine di lunghezza d’onda); sbloccando la griglia e trascinando si sovrascrive comunque l’ordine della griglia.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



La **modalità separata** mostra un riquadro per ogni telecamera. Ogni riquadro presenta in sovrimpressione:

* il nome della telecamera (in alto a sinistra),
* un **indicatore di fps** (in alto a destra) — si tratta della *frequenza di acquisizione effettiva* della telecamera segnalata dal backend, non della frequenza di campionamento dell’anteprima (l’anteprima in tempo reale è limitata a 30 fps indipendentemente dalla frequenza di acquisizione),
* un puntino di stato — verde (in streaming) / ambra (in caricamento) / rosso (errore),
* un **indicatore di frame obsoleto** quando non arriva alcun frame aggiornato da 2 s — normale per circa 5 s dopo qualsiasi connessione/disconnessione, mentre il backend riequilibra la larghezza di banda tra le telecamere.

La **modalità combinata**mostra un unico riquadro composito: il backend esegue il debayering, il ridimensionamento, l’allineamento, la riduzione del rumore, la conversione in radianza per banda (più la riflettanza DLS quando è associato un sensore di luce), valuta l’espressione dell’indice dell’array, applica la LUT e trasmette il risultato in formato MJPEG. Fino a quando non viene renderizzato il primo fotogramma allineato, il riquadro indica il proprio stato: «Preparazione dell’array…», "Calibrazione allineamento…", "In attesa del primo fotogramma…", oppure — se i tentativi di allineamento automatico del riquadro (circa un minuto) si esauriscono — "Allineamento richiesto" con un pulsante**Calibra allineamento**.

Anche la lettura degli fps del riquadro dell’array segnala lo stato dello streaming:**"Applicazione delle impostazioni…"**mentre una modifica delle impostazioni (formato, binning, ROI) riavvia le telecamere,**"Bloccato"**quando l’array è attivo ma ha smesso di fornire gruppi di fotogrammi e**"Interrotto"**quando ha smesso di trasmettere — in tal caso, ricollegare l’array. Quando la separazione spettrale ha dovuto limitare una banda visualizzata nell’immagine composita su una quota consistente dei suoi pixel (25 % o più), il riquadro mostra**"Dati della banda a bassa affidabilità · N%"**: i valori dell’indice basati su quella banda sono sospetti (di solito una banda ha ricevuto troppo poca luce).

Informazioni utili sulla modalità combinata:

* L’immagine composita è allineata al fotogramma della telecamera **master**. Il puntamento AE-ROI e la misurazione spot sull’immagine composita sono precisi per la telecamera master e approssimativi per quelle slave; usa**Split View** (impostazioni dell’array → “Mostra telecamere associate”) per ottenere riquadri per telecamera con precisione al pixel senza aprire connessioni aggiuntive alle telecamere.
* ****Display Layers**(impostazioni array; disattivato per impostazione predefinita) consente di scegliere un livello di primo piano e uno di sfondo — qualsiasi telecamera membro o**Indice**. Con primo piano = Indice, i pixel al di fuori dei valori Min/Max della LUT mostrano il livello di sfondo.
* **Risoluzione di rendering** (impostazione predefinita 720p) imposta l’altezza dello streaming live *e* le dimensioni di esportazione del composito salvato. Le immagini per singola telecamera vengono sempre esportate alla massima risoluzione.
* L’allineamento viene calcolato per ogni sessione e non viene mai salvato. Chloros allinea **ogni array automaticamente dopo la connessione, in entrambe le modalità di visualizzazione**, una volta che l’esposizione automatica delle telecamere associate si è stabilizzata (continua a provarci per un massimo di 5 minuti). La sezione**Allineamento**del pannello delle impostazioni dell’array mostra i residui RMS e il pulsante**Ricalibra allineamento** — anche per gli array separati, che non hanno una tessera composita.

## Acquisizione: monitoraggio vs analisi

Le superfici di acquisizione dell’array si dividono nettamente in **livello di monitoraggio**(registra ciò che vedi) e**livello di analisi** (registra i dati grezzi, calibra in seguito):

| Flusso di lavoro | Livello | Cosa salva | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Acquisizione**(immagini fisse) | Analisi | Un gruppo di fotogrammi sincronizzati per ogni passaggio; file per ogni telecamera a ogni livello di esportazione selezionato (grezzo/debayered/radianza/riflettanza/anteprima/indice) + sidecar `.daq` |**Pulsante **Acquisisci tutto** + Impostazioni di acquisizione | `lattice array-capture` |
| **Registra video indice** | Monitoraggio | Il composito dell’indice combinato in tempo reale così come visualizzato — 8 bit, risoluzione di anteprima, LUT incorporata. Dalla CLI, senza alcuna vista GUI aperta, il backend esegue il rendering dell’NDVI (o `--expr`) dell’array una volta che questo è allineato; una registrazione che non scrive alcun fotogramma fallisce | ● Registrazione video indice (array combinati) | `lattice array-record` |
| **Sequenza grezza → creazione video**| Analisi | Fotogrammi grezzi del sensore alla massima frequenza di acquisizione + manifesto + `.daq`, quindi ricostruzione offline in video calibrato di radianza / riflettanza / indice, sincronizzato temporalmente con le letture DAQ | ⦿ Registrazione sequenza grezza →**Creazione video** | `lattice array-burst` → `lattice array-build-video` |

Regola generale: se i pixel forniranno *misurazioni*, utilizzare la modalità di acquisizione o burst (livello di analisi); se occorre semplicemente *osservare o mostrare* ciò che l’array ha rilevato, registrare il video indicizzato (livello di monitoraggio).

### Impostazioni di acquisizione (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



L’icona a forma di ingranaggio accanto a **Acquisisci tutto** apre il pannello delle impostazioni di acquisizione (richiede un progetto aperto — le acquisizioni vengono salvate al suo interno):

* **Modalità di acquisizione**:**Singola**(un solo passaggio) /**Continuativa**(una dopo l’altra; limitata da un numero di acquisizioni, predefinito 1, o da una durata, predefinita 10 s) /**A intervalli** (timelapse: N acquisizioni ogni X intervalli per un totale di Y, impostazione predefinita 1 ogni 5 s per 1 minuto).
* **Tipi di esportazione per telecamera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — tutte le opzioni applicabili sono attive per impostazione predefinita. Radiance/Reflectance sono nascoste per le telecamere con filtro RGB; **Reflectance appare solo quando la telecamera dispone di un sensore di luce DAQ** (proprio o ereditato dall’array); Index richiede un’espressione di indice configurata.
* **Allineato**(per array, impostazione predefinita**attiva**): deforma le esportazioni dei membri in base al profilo di allineamento dell’array, in modo che le esportazioni siano registrate a livello di pixel. I dati grezzi rimangono sempre non distorti ma riportano la trasformazione nei metadati. I file dei membri dell’array vengono salvati nell’orientamento nativo di ciascuna telecamera (qualsiasi impostazione di Specchio/Rotazione viene registrata nel file, ma non applicata).
* **Acquisizione più veloce** (interruttore): solo raw + la lettura DAQ assegnata + il composito a indice combinato gratuito, saltando i calcoli di calibrazione al momento dell’acquisizione per ottenere la massima velocità — ricostruire successivamente radianza/riflettanza/indice dal file `.daq` salvato.
* Le selezioni rimangono associate al progetto. Le telecamere nascoste o in pausa vengono saltate.
* **Continuo** sugli array viene eseguito all’interno del backend Chloros (novità nella versione 1.3.0): quando ogni telecamera nell’elenco appartiene a un array, il backend esegue autonomamente il ciclo di acquisizione, scrivendo un gruppo mentre seleziona quello successivo, senza alcuna richiesta per ogni acquisizione. Un disco pieno interrompe l’esecuzione (“Interrotto: disco pieno. Salvati N, F fallito”). Vedere [Impostazioni e modalità di acquisizione](capture.md#capture-modes) per la nota di Microsoft Defender sulle elevate frequenze di acquisizione.

L'equivalente in CLI (stesso endpoint del backend, stessa semantica):

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

La compressione TIFF per le acquisizioni è `deflate` (senza perdita di dati, predefinita) o `none` — le tabelle complete dei flag, la struttura della cartella di acquisizione e le regole di rielaborazione sono riportate nel [Riferimento CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Accoppiamento di un sensore di luce DAQ

Le anteprime con correzione della riflettanza e dell’illuminazione richiedono dati sulla luce in discesa provenienti da un sensore DAQ (collegato nella scheda **Sensori di luce**):

* La **riga dell’array**nella barra laterale mostra un**pulsante "DAQ · on/off"** — *on* quando è impostato un sensore di luce a livello di array **oppure** quando una qualsiasi telecamera membro ne possiede uno proprio; il suo tooltip elenca esattamente quale sensore alimenta quale telecamera.
* Assegnare a livello di array nelle impostazioni dell’array → **Sensore di luce ambientale**→ menu a tendina**Sensore di luce**. La selezione rimane valida per il progetto, si applica a tutte le telecamere dell’array e le singole telecamere possono comunque sovrascriverla con il proprio sensore.
* La riga di stato sottostante riporta lo stato in tempo reale: **Disattivato**→ "In attesa del primo spettro…" →**"Attivo — tutte le telecamere dell’array sono sottoposte a correzione dell’illuminazione"** → oppure, se non è arrivato alcun nuovo spettro negli ultimi 3 secondi, un avviso di dati non aggiornati — continua a essere utilizzata l’ultima lettura (le letture non scadono mai nel percorso di acquisizione).

Con un sensore assegnato: il tipo di esportazione «Riflettanza» diventa disponibile, le anteprime in tempo reale sono corrette per l’illuminazione, l’esposizione automatica predittiva può utilizzare lo spettro e ogni acquisizione di riflettanza scrive la lettura DAQ effettivamente utilizzata come **file `.daq`** nella cartella `daq/`, in modo che l’acquisizione possa essere rielaborata in un secondo momento. Accanto ad esso, un file `.daq.cameras.json` indica le telecamere i cui fotogrammi sono descritti da quella lettura; durante la rielaborazione, Chloros abbina la lettura solo alle immagini provenienti da quelle telecamere.

## Opzioni CLI di `array-connect`

| Flag | Predefinito | Descrizione |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Rileva automaticamente tutte le telecamere LATTICE (ne occorrono ≥2) | **Il primo numero di serie è quello del MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Linea di sincronizzazione GPIO. |
| `--target-fps F` | auto | Frequenza di scatto del trigger MASTER. |
| `--binning {1,2,4}` | auto | Binning hardware. L’impostazione “auto” corrisponde al sensore completo (senza binning) ogni volta che una frequenza dei fotogrammi lo supporta; l’impostazione “connect never” non applica mai il binning, solo per mantenere alta la frequenza. |
| `--packed {auto,on,off}` | `auto` | [Trasporto compresso a 12 bit](#packed-12-bit-transport): `auto` esegue il packing solo quando è utile e l’host è in grado di gestire il decomprimimento. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Sovrascrittura avanzata del selettore del livello di sincronizzazione. |
| `--wire-ceiling-mbps MB_PER_S` | rilevato automaticamente | Budget di banda dell’host in MB/s — la forma CLI del campo **Wire Budget**. Ridurlo se l’array segnala frame danneggiati. Viene salvato con il progetto, quindi una successiva riconnessione lo ripristina. |
| `--no-recommend` | disattivato | Salta la fase di analisi della rete. |
| `--no-ptp` | disattivato | Disattiva PTP (i timestamp tra telecamere diverse non saranno quindi comparabili). |

`lattice array-list`, `array-status` e `array-disconnect` gestiscono la sessione persistente. Il riferimento completo ai sottocomandi, compresi l’allineamento (`align-calibrate` / `align-apply`) e gli strumenti di rete, si trova nella sezione [Riferimento CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); gli equivalenti nell’SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) si trovano nel [Riferimento SDK](../reference/sdk-reference.md). In Python, il wire budget è `connect_array(..., wire_ceiling_mbps=120)`, mentre la suddivisione tra dati corrotti in tempo reale e dati mai arrivati si trova su [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
