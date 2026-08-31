# Collegamento delle telecamere

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>La scheda &quot;Telecamere&quot; prima che venga collegato qualsiasi dispositivo</p></figcaption></figure>Chloros rileva automaticamente le telecamere LATTICE presenti sulla connessione — dalla scheda &quot;Telecamere&quot; dell&#x27;interfaccia grafica, da `chloros-cli lattice`, oppure tramite Python SDK. La stringa del modello della telecamera determina tutte le operazioni successive: Chloros determina il profilo del sensore, la configurazione delle bande e la calibrazione di fabbrica in base ai valori `DeviceUserID` + `DeviceSerialNumber` della telecamera, quindi **non c’è nulla da configurare per ogni singola telecamera**.

Prima di effettuare la connessione, assicurarsi che la rete host sia configurata correttamente: indirizzamento link-local, jumbo frame e, per gli array, le impostazioni del buffer di ricezione della scheda di rete. Si tratta della configurazione a livello hardware, descritta nel manuale LATTICE: [**Configurazione di rete**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Connessione dall’interfaccia grafica

Aprire la scheda **Telecamere**nella barra laterale di Chloros (le schede relative all’hardware compaiono una volta che il backend ha completato l’avvio), oppure utilizzare il menu principale →**Connetti alla telecamera**. Entrambe le opzioni aprono la finestra di dialogo**Connetti telecamera(e)**.

### La finestra di dialogo **Connetti telecamere**La finestra di dialogo esegue una scansione della rete non appena viene aperta (&quot;Scansione della rete...&quot;), ed elenca tutte le telecamere rilevate. Ogni riga mostra il**modello**della telecamera (ad es. `LATT-M3M-L41-F550`), il**numero di serie**e l’**indirizzo IP**.

* **Fare clic su una riga per selezionarla**(evidenziazione in verde). È possibile selezionare**più telecamere** e collegarle in un’unica operazione; Chloros le collega in sequenza.
* Le righe contrassegnate con **&quot;Connesso&quot;** sono già collegate e non possono essere riselezionate.
* Le righe contrassegnate con **&quot;In array&quot;** appartengono a un array di telecamere attualmente connesso. Scollegare prima l&#x27;array per utilizzare quella telecamera in modalità autonoma.
* **Connetti** — collega le telecamere selezionate; il pulsante mostra un conteggio, ad esempio &quot;Connetti (3)&quot;, quando ne è selezionata più di una.
* **Ricerca** — esegue nuovamente la ricerca.
* **Chiudi** — chiude la finestra di dialogo.
* Se la scansione termina senza risultati, la finestra di dialogo mostra **&quot;Nessuna telecamera trovata sulla rete&quot;** — vedere [Risoluzione dei problemi](connecting.md#troubleshooting) di seguito.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>La finestra di dialogo «Connetti telecamere» — qui mostrata senza telecamere presenti in rete</p></figcaption></figure>### Prima connessione: download del pacchetto di calibrazione

La **prima volta**che una determinata telecamera viene collegata a un computer, Chloros scarica il pacchetto di calibrazione di fabbrica della telecamera (\~3,8 MB) direttamente dalla telecamera stessa tramite GigE. Durante questa operazione, la finestra di dialogo mostra un pannello verde**&quot;Download dei dati di calibrazione dalla telecamera&quot;**con una barra di avanzamento per ogni numero di serie — l’operazione richiede circa**70 secondi** per telecamera. Il pacchetto viene memorizzato nella cache dell’host, quindi i collegamenti successivi della stessa telecamera saltano completamente il download (e il pannello non viene mai visualizzato).

### Analizza sistema

Il pulsante **Analizza sistema** della finestra di dialogo esegue un&#x27;analisi dell’host e della rete (durante l’esecuzione viene visualizzato il messaggio “Analisi in corso...”) e genera un rapporto diagnostico:

* **Host** — Core della CPU e RAM; nome e memoria della GPU, oppure “GPU: Nessuna rilevata”.
* **Interfacce di rete** — nome di ciascuna scheda di rete, velocità di collegamento, MTU (con l’indicazione “jumbo” se attiva), stato di up/down e se si trova su un bus USB.
* **Telecamere**— numero di serie, modello, IP e**su quale scheda di rete si trova ciascuna telecamera**.
* **Prestazioni** — fps attuali rispetto a quelli ideali per telecamera in base al formato dei pixel, con una riga verde &quot;Potenziale: possibile miglioramento di N×&quot; quando il valore ideale supera quello attuale.
* **Avvisi e raccomandazioni numerate** — oppure il messaggio “Il sistema sembra funzionare correttamente per il numero attuale di telecamere” quando non c’è nulla da correggere.

Eseguilo ogni volta che il rilevamento o lo streaming si comportano in modo inaspettato: identifica la maggior parte dei problemi legati alle schede di rete (MTU errata, telecamera collegata all’interfaccia sbagliata, limiti dell’adattatore USB) senza uscire dalla finestra di dialogo.

### Collegamento di un array

Per collegare due o più telecamere come **array sincronizzato**, utilizzare invece la procedura guidata di collegamento dell’array (**Connetti array di telecamere**): la procedura guida l’utente attraverso la selezione master/slave (precompilata da una sonda di cablaggio GPIO), la scelta della modalità di visualizzazione (tessere separate o combinate) e una schermata delle impostazioni dell’array con una proiezione in tempo reale dei fps raggiungibili e della larghezza di banda del cavo prima di confermare. La procedura guidata e i flussi di lavoro relativi all’array sono descritti nella sezione dedicata agli array multicamera del presente manuale; l&#x27;equivalente per CLI è il “Flusso di lavoro per il primo collegamento delle telecamere LATTICE” nel [Riferimento CLI](../reference/cli-reference.md).

## Connessione da CLI e SDK

L’accesso a CLI e SDK richiede un piano a pagamento Chloros+ e l’avere effettuato l’accesso; questo requisito viene applicato a livello di server (`401 AUTH_REQUIRED` se non si è effettuato l’accesso, `403 PLAN_UPGRADE_REQUIRED` con il piano gratuito).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Firme complete, opzioni e flussi di lavoro di acquisizione: [CLI Riferimento](../reference/cli-reference.md) § `chloros-cli lattice`, [Riferimento SDK](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Come viene gestita la calibrazione al momento della connessione

Ogni telecamera LATTICE dispone del proprio pacchetto di calibrazione di fabbrica **integrato nella telecamera**, e Chloros verifica anche il cloud di MAPIR quando la telecamera si connette:

| Situazione   | Cosa utilizza Chloros                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**| La**calibrazione più recente pubblicata per quel numero di serie** — la copia nel cloud ha la precedenza su quella presente sulla fotocamera. Una fotocamera che è stata ricalibrata o aggiornata da MAPIR si aggiorna quindi automaticamente; non è necessaria alcuna azione da parte dell’utente. |
| **Offline**| Il**pacchetto presente sulla fotocamera**, così com&#x27;è. I flussi di lavoro completamente offline continuano a funzionare; semplicemente non acquisiscono le calibrazioni più recenti finché la fotocamera non viene collegata a Internet almeno una volta (o non viene ripristinata alle impostazioni di fabbrica).                                                  |

Al momento dell’acquisizione, i coefficienti effettivamente applicati vengono **fissati nei metadati XMP di ciascuna immagine**. Un successivo aggiornamento della calibrazione non modifica mai in modo invisibile le immagini già acquisite: la rielaborazione di una vecchia acquisizione utilizza i coefficienti registrati nei suoi metadati XMP, non quelli più recenti disponibili oggi.

## Risoluzione dei problemi

* **&quot;Nessuna telecamera trovata sulla rete&quot;**— verificare la configurazione link-local in [Configurazione di rete](https://mapir.gitbook.io/lattice-camera/setup/network-setup): scheda di rete host statica `169.254.x.x/16`, telecamere sullo stesso collegamento, non è previsto l’uso di DHCP/gateway. Quindi utilizzare**Analizza sistema**nella finestra di dialogo di connessione per verificare su quale scheda di rete ciascuna telecamera è (o non è) visibile. Eseguire una**nuova scansione** dopo qualsiasi modifica al cablaggio o alla scheda di rete.
* **Un sistema che prima funzionava non si connette più** (i pannelli dell’array si bloccano con `FRAMES WILL DROP` / `Reduce ROI to enable`) — un aggiornamento del driver della scheda di rete ha reimpostato in modo invisibile le impostazioni del receive-ring. Riapplicarle oppure eseguire `chloros-cli lattice network --fix` da un terminale con privilegi elevati; consultare [Configurazione di rete](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Una telecamera mostra &quot;In Array&quot;** — appartiene a una sessione array connessa. Scollegare l&#x27;array per utilizzare la telecamera in modalità autonoma.
