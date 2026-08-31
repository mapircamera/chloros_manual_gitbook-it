# La scheda DAQ in Chloros

La scheda DAQ — denominata **Sensori di luce** nella barra laterale di Chloros — è la superficie di controllo in tempo reale per i [sensori di luce DAQ-U, DAQ-M e DAQ-E](README.md): collega i sensori tramite qualsiasi protocollo di trasporto, osserva gli spettri calibrati in tempo reale, calcola la riflettanza in tempo reale da una coppia di sensori e registra i file `.daq` direttamente nel tuo progetto.

La scheda diventa disponibile una volta che il backend Chloros ha completato l’avvio. I grafici della scheda sono alimentati dal servizio DAQ di Chloros tramite una connessione in tempo reale che si riconnette automaticamente (tempo di attesa di 2–10 s) in caso di interruzione; mentre il servizio è irraggiungibile, la riga **Stato**di un sensore riporta**Nessun server**.

Il layout è costituito da una **barra laterale dei sensori**(una riga per ogni sensore connesso) più un’**area grafici** (un riquadro grafico per sensore o gruppo).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Connessione di un sensore

Fare clic su **Connetti sensore** nella parte superiore della barra laterale. La finestra di dialogo di connessione si apre nell’area principale (o come finestra sovrapposta quando si aggiunge un altro sensore — in tal caso compare un pulsante Annulla).

| Controllo | Comportamento |
| --- | --- |
| **Tipo di dispositivo** | `DAQ-U (USB)` (predefinito), `DAQ-M (Bluetooth)` o `DAQ-E (Ethernet)`. La selezione di un&#x27;altra opzione riavvia la scansione per il protocollo di trasporto appena selezionato. |
| **Porta / Dispositivo BLE / Nome host / IP** | Elenca i dispositivi rilevati come `device - description`; la prima voce riconosciuta come sensore viene selezionata automaticamente. Durante la scansione vengono visualizzati `Scanning...` (USB), `Scanning (N)...` con un conto alla rovescia di 8 secondi (BLE) o `Discovering ethernet sensors (N)...` con un conto alla rovescia di 5 secondi (Ethernet). I risultati vuoti vengono visualizzati come `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Aggiorna** | Esegue immediatamente una nuova scansione del protocollo di trasporto selezionato (disabilitato durante la scansione BLE/Ethernet). |
| **Connetti** | Si abilita una volta selezionato un dispositivo; l’etichetta diventa `Connecting...` mentre viene stabilita la connessione. |

La ricerca viene eseguita solo **mentre la finestra di dialogo di connessione è visualizzata sullo schermo** e si ripete ogni 15 secondi solo per il trasporto selezionato; la semplice apertura della scheda non avvia la scansione. In caso di errore, la finestra di dialogo mostra il messaggio: *&quot;Connessione fallita. Prova a scollegare e ricollegare il sensore, quindi fai nuovamente clic su Connetti.&quot;*

La barra laterale si apre automaticamente quando si connette il primo sensore.

{% hint style="info" %}
**Il DAQ-E non viene visualizzato?** Il DAQ-E non dispone di LED di stato: controllare l’indicatore PoE/link sullo switch o sulla porta dell’iniettore a cui è collegato e attendere alcuni secondi dopo l’accensione affinché si avvii. Il dispositivo Chloros deve trovarsi nello stesso dominio di broadcast (il mDNS non attraversa i router). Su Windows, accettare la richiesta del firewall Defender la prima volta che Chloros associa i propri socket multicast (mDNS UDP 5353, dati DAQ-E UDP 5002, PTP UDP 319/320). Due unità DAQ-E presenti sulla stessa LAN vengono rilevate separatamente, ciascuna con il proprio nome host `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Il tipo di dispositivo offre DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet)</figcaption></figure>***

## La barra laterale dei sensori

A ogni sensore collegato viene assegnata una riga (più una riga per ogni gruppo “Ambiente+Oggetto”). Le righe possono essere riordinate trascinandole, e il loro ordine determina anche la disposizione dei riquadri del grafico. Fare clic su una riga per rendere quel sensore/gruppo il grafico attivo nella vista elenco.

| Elemento | Significato |
| --- | --- |
| Bordo sinistro colorato | Il colore del grafico del sensore. |
| Icona di trasporto | `DAQ-U` / `DAQ-M` / `DAQ-E`, oppure un badge verde `REF` per un gruppo di riflettanza Ambient+Object. |
| Nome del dispositivo | Per impostazione predefinita corrisponde al numero di serie del sensore (la sua identità stabile per la calibrazione, i nomi dei file `.daq` e la corrispondenza durante l’importazione); i nomi personalizzati vengono mantenuti per ogni progetto. |
| Pillola **Calibrato** (verde) | Visualizzata quando è caricato il pacchetto di calibrazione di fabbrica del sensore, ovvero quando gli spettri sono espressi in W/m²/nm. |
| Icona **Aggiornamento disponibile** (ambra, solo DAQ-E) | Il firmware in esecuzione è più vecchio dell’immagine inclusa in questa build Chloros. Durante un aggiornamento mostra l’avanzamento in tempo reale (`Flashing… N%`, `Restarting sensor…`, poi `Updated X → Y` o `Failed`). |
| Occhio | Attiva/disattiva la visibilità di questo sensore sul grafico. |
| Ingranaggio | Apre la finestra modale delle impostazioni specifiche del sensore (sotto). |
| ✕ (rosso) | Disconnette il sensore o rimuove un gruppo Ambiente+Oggetto. |

Sopra le righe si trovano due pulsanti:

* **Connetti sensore** — apre la finestra di dialogo di connessione (viene rinominato in `Connecting...` mentre è occupato).
* **Registra tutto / Interrompi tutto**— avvia o interrompe una registrazione `.daq` su**tutti**i sensori collegati. Richiede almeno un sensore**e un progetto aperto** (testo di suggerimento: “Apri un progetto per registrare”); diventa rosso mentre è in corso una registrazione.

Quando è vuoto, visualizza il messaggio “Nessun sensore collegato”.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Impostazioni per singolo sensore (finestra a forma di ingranaggio)

Si apre facendo clic sull’icona a forma di ingranaggio nella riga di un sensore. Contenuti in ordine:

* **Righe informative** — Tipo di dispositivo (DAQ-U/M/E), Connessione (`Serial (USB)` / `Bluetooth` / `Ethernet`), porta (porta COM, indirizzo BLE o host) e numero di serie.
* **Rapporto di calibrazione: Scarica** — recupera il certificato di calibrazione tracciabile NIST (PDF) di questa unità e lo apre nel visualizzatore PDF. Disponibile una volta noto il numero di serie; il certificato viene memorizzato nella cache alla prima connessione.
* **Nome del dispositivo** — clicca sulla matita per rinominarlo; rimane valido per ogni progetto.
* **Colore linea grafico** — campioncino di colore; rimane impostato per ogni progetto.
* **Tempo di integrazione (ms)**— cursore + numero,**1–500 ms**, valore predefinito**32 ms**. Disabilitato quando l’AE è attivo.
* **Media fotogrammi**— cursore + numero,**1–50 fotogrammi**, valore predefinito**20**.
* **AE: ON/OFF**— interruttore per l’esposizione automatica;**predefinito su ON** alla connessione. Disattivarlo per impostare manualmente il tempo di integrazione.
* **Interrompi streaming / Avvia streaming** — mette in pausa o riprende lo streaming live.
* **Registra / Interrompi registrazione** — registrazione `.daq` per singolo sensore (richiede un progetto aperto).
* **Cap** — il profilo di correzione del cap (sezione successiva).
* **Righe di informazioni in tempo reale** — Tempo di integrazione (ms), FPS, Campioni, Registrazione (rosso `REC` o `Off`) e Stato (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Solo DAQ-E: righe relative a rete, firmware e PTP

* **Nome host / IP** — l’indirizzo corrente dell’unità.
* **Firmware** — versione del firmware in tempo reale, più una cella di azione: un<version\>

pulsante</version\>

**Aggiorna a \<version\>** appare quando questa build Chloros include un’immagine del firmware DAQ-E più recente. L’aggiornamento viene installato in rete in circa 30 secondi; il sensore si riavvia e si riconnette automaticamente, e un trasferimento interrotto lascia intatto il firmware attuale. L’avanzamento viene visualizzato in tempo reale (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), e la cella riporta il valore `Up to date` quando è aggiornato.
* **Sincronizzazione PTP** — lo stato PTP in tempo reale (ricade su `unknown`). Il firmware DAQ-E v1.2.0+ partecipa allo standard IEEE 1588 PTPv2 come clock esclusivamente slave; il backend dell’host Chloros è il grandmaster PTP, e ogni DAQ-E e ogni telecamera LATTICE sulla LAN si sincronizza con esso nel dominio 0, mantenendo i timestamp entro circa 1 ms.

Per un gruppo “Ambient+Object”, la finestra modale “Gear” mostra solo i sensori sorgente del gruppo, il nome del dispositivo e il colore della linea del grafico.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Selezione del cappuccio

Il menu a tendina **Cap** indica a Chloros quale coperchio fisico è montato sul diffusore del sensore e applica a ogni spettro il profilo di correzione misurato in fabbrica per quel coperchio. Le opzioni disponibili dipendono dal modello:

| Modello | Opzioni coperchio |
| --- | --- |
| DAQ-U | Nessuno (sensore nudo), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (correttore coseno) |
| DAQ-M | Nessuno (sensore nudo), Sunshine (correttore coseno) |
| DAQ-E | Nessuno (sensore nudo), FOV 15°, FOV 45°, FOV 90°, Sunshine (correttore coseno) |

**L’impostazione predefinita per ogni modello è Sunshine (correttore del coseno)** — MAPIR fornisce ogni DAQ con il cappuccio Sunshine installato, e questa è la configurazione standard per uso esterno: una visione emisferica a 180° con errore coseno ≤ ±4 % fino a 60° e ≤ ±4,5 % fino a 70° (sconsigliato con elevazione solare inferiore a ~15°), con attenuazione prevista dal design (~12×). La selezione effettuata rimane valida nel progetto.

{% hint style="warning" %}
**La selezione del cappuccio deve corrispondere al cappuccio fisico.**Né il sensore né il software sono in grado di rilevare quale cappuccio sia montato. La selezione determina sia la correzione in tempo reale sia il timbro scritto in ogni file `.daq` — con l’attenuazione di circa 12× del cappuccio Sunshine, una sostituzione del cappuccio non dichiarata corregge erroneamente gli spettri di circa quel fattore. (La rimozione e il rimontaggio dello stesso cappuccio comportano una ripetibilità di circa l’1,5%.) Selezionare**Nessuno (sensore nudo)** solo quando il cappuccio è fisicamente rimosso; su un DAQ-E, l’opzione “Nessuno” applica comunque un profilo geometrico di fabbrica per il suo diffusore in vetro incassato — non è un’operazione nulla — e un DAQ-E senza cappuccio rappresenta una configurazione da banco, non una configurazione da campo supportata.
{% endhint %}

{% hint style="info" %}
Aggiornamento da un manuale precedente: il pulsante di attivazione/disattivazione «Sunshine Diffuser Installed» (Diffusore Sunshine installato) presente nel browser a partire dalla versione 1.1.0 non è più disponibile. La gestione dei cappucci avviene ora tramite questo profilo &quot;Cap&quot; specifico per ciascun sensore, applicato lato server.
{% endhint %}

***

## L&#x27;area dei grafici

Una barra superiore fissa contiene un **pulsante per passare dalla visualizzazione a elenco a quella a griglia**e un cursore**Zoom grafico** (dimensione delle tessere 200–2000 px). La visualizzazione passa automaticamente alla griglia quando è presente più di un gruppo di grafici, e torna alla visualizzazione a elenco quando ce n’è uno solo o meno. La modalità di visualizzazione e le dimensioni del grafico vengono mantenute per ogni progetto.

Il **grafico dello spettro** per ciascun sensore mostra:

* **Asse X** — Lunghezza d’onda (nm). La griglia del sensore va da 340 a 1010 nm con passi di 5 nm (135 punti), interpolata a 1 nm per la visualizzazione.
* **Asse Y** — Potenza (W/m²), con un prefisso SI automatico (m/µ/n) scelto in base al picco. Gli spettri sono irradianti spettrali (W/m²/nm) calibrati radiometricamente su tutte e tre le modalità di trasporto.
* Un riempimento spettrale arcobaleno sotto una singola traccia; più sensori su un unico grafico si sovrappongono come linee colorate con riempimenti sfocati.
* **Passaggio del mouse**— un cursore verticale con la lunghezza d’onda e il valore per ciascun sensore;**trascinare** per ingrandire (durante l’ingrandimento appare un pulsante per rimpicciolire).
* Un pulsante **+** (solo nella vista a griglia) per aggiungere un sensore a questo grafico o creare un gruppo (vedi sotto).
* Il nome del dispositivo centrato in alto e un indicatore di caricamento fino all’arrivo del primo fotogramma.

La **saturazione** non è indicata sul grafico stesso: un sensore saturo mostra il testo di stato rosso `SATURATED` e una riga rossa `Saturated: Yes` nella tabella dei dati in tempo reale. Ridurre il tempo di integrazione o riattivare l’AE per eliminarlo.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Tabella dei dati in tempo reale (visualizzazione a elenco)

Sotto il grafico nella visualizzazione a elenco, aggiornata ogni 500 ms:

* **Tutti i modelli**: Campione di colore della luce (sRGB da CIE XYZ), Saturato (Sì/No), CIE 1931 X/Y/Z, Cromaticità x/y, CIE u′/v′, CCT (K), CRI (Ra), Lunghezza d’onda dominante (nm), lunghezza d’onda di picco (nm), purezza di eccitazione, Duv, CIE L\*/a\*/b\* e Munsell H/V/C.
* **Solo sensori calibrati**(qualsiasi modello tra DAQ-U / DAQ-M / DAQ-E una volta caricato il pacchetto di calibrazione di fabbrica — il badge verde**Calibrato** sulla riga del sensore ne è l’indicazione): Potenza totale (W/m²), lux fotopico (lx), lux scotopico (lx), rapporto S/P, PPFD più PPFD Red/Green/Blue (µmol/m²/s) e le irradianze opiche — cono S, melanopico, rodopico, cono M, cono L (tutte in W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Gruppi di riflettanza (Ambiente + Oggetto)

È possibile combinare due sensori collegati per ottenere una visualizzazione in tempo reale della riflettanza, senza l’utilizzo di una telecamera:

1. Nella vista a griglia, fare clic su **+**su un riquadro del grafico e selezionare**Combina Ambiente + Oggetto**.
2. Selezionare un sensore **Fonte di luce ambientale**e un sensore**Scanner oggetto**(due sensori distinti), quindi**Crea**.

Chloros calcola R(λ) = oggetto(λ) / ambiente(λ) per ciascuna lunghezza d’onda dai due flussi in tempo reale (0 quando ambiente ≤ 0). L’etichetta del gruppo dipende dalla classe di calibrazione dei sensori:

* Entrambi i sensori calibrati (pacchetto caricato) → **&quot;Riflettanza apparente&quot;**.
* Uno dei due sensori non calibrato → **&quot;Riflettanza relativa&quot;**.

Il gruppo appare come una riga verde `REF` nella barra laterale e con un proprio grafico (riempimento arcobaleno, valori visualizzati al passaggio del mouse con 4 decimali, zoom tramite trascinamento).

Il menu **+**offre anche l’opzione**Aggiungi nuovo sensore** con tre posizioni: *Combina nuovo sensore* (aggiungi a questo grafico), *Sposta sensore esistente qui* oppure *Visualizza nuovo sensore* (grafico dedicato).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Tabella degli indici di vegetazione

Nella vista elenco, sotto il grafico di un gruppo di riflettanza è presente una tabella degli indici di vegetazione, calcolata sulla base della riflettanza in tempo reale ai centri di banda **blu 450 / verde 550 / rosso 670 / NIR 800 nm** (valori con 4 cifre decimali, `---` quando non calcolabile; passare il mouse sul nome di un indice per visualizzarne il nome completo):

* **Sempre visualizzati** (invarianti rispetto alla scala, per qualsiasi combinazione di sensori): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Solo quando entrambi i sensori sono calibrati in base alla potenza** (entrambi i pacchetti caricati): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

.***

## Registrazione dei file `.daq`

* La registrazione richiede un **progetto aperto** — in caso contrario, sia l’opzione “Registra tutto” (nella barra laterale) che il pulsante “Registra” relativo a ciascun sensore risultano disabilitati.
* I file vengono salvati con estensione **`<project folder>/light_sensor/`**; i nomi dei file contengono l’ID del sensore e un timestamp, mentre il nome del dispositivo viene memorizzato insieme alla registrazione.
* Quando una registrazione si interrompe (Stop, Stop All o una disconnessione durante la registrazione), il file `.daq` completato viene **aggiunto automaticamente al progetto aperto** — e compare nell’elenco dei file del progetto senza necessità di aggiungerlo manualmente, pronto per essere utilizzato come dati di irraggiamento discendente per l’[elaborazione della riflettanza](README.md).
* Durante la registrazione, nelle righe in tempo reale della finestra modale delle impostazioni viene visualizzato un indicatore rosso `REC`.

Per ottenere valori quantitativi di irraggiamento, calcolare la media su almeno 15 secondi di dati — si tratta di una caratteristica dello strumento, non di un difetto.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Layout multisensore e persistenza del progetto

* È possibile combinare più sensori in un unico grafico (assi condivisi), mantenere grafici separati (disposizione automatica a griglia), spostare i sensori tra i grafici, riordinare le righe/i riquadri trascinandoli e nascondere i singoli sensori con l’icona a forma di occhio.
* Per ogni progetto, Chloros mantiene: nomi dei dispositivi, colori dei grafici, dimensioni del grafico, modalità di visualizzazione e impostazioni di ciascun sensore (tempo di integrazione, media dei fotogrammi, stato AE, selezione del limite massimo).
* **Riaprendo un progetto, i sensori si riconnettono automaticamente** tramite indirizzo — porta COM per DAQ-U, dispositivo BLE per DAQ-M, nome host mDNS per DAQ-E (risolto anche se l’IP dell’unità è cambiato) — e riapplica il profilo cap salvato, la media dei fotogrammi, lo stato AE e il tempo di integrazione manuale di ciascun sensore.***

## Accoppiamento della telecamera (DLS)

Non c’è nulla da accoppiare. A differenza dei flussi di lavoro DLS dei droni che associano in anticipo un sensore di luce a una fotocamera, Chloros abbina i dati DAQ alle immagini a valle: al momento dell’importazione/elaborazione, le letture di `.daq` vengono interpolate in base al timestamp di esposizione di ciascuna acquisizione. Effettua la registrazione con qualsiasi sensore collegato (l&#x27;`.daq` viene automaticamente inserito nel progetto) e l&#x27;elaborazione della riflettanza individua le letture corrette in base al tempo — consulta [Sensori di luce DAQ](README.md) per capire come vengono utilizzati i dati di irraggiamento verso il basso.</version\>
