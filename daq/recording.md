# Registrazione e formato .daq

Un file `.daq` è il formato di registrazione del sensore di luce di Chloros

: un **database SQLite** contenente fotogrammi spettrali calibrati provenienti da un sensore DAQ. Ne basta registrare uno durante una sessione di acquisizione e la pipeline di riflettanza potrà in seguito dividere ogni immagine per l’irraggiamento discendente misurato in quel preciso momento.

## Cosa contiene un file .daq

| Proprietà | Valore |
| --- | --- |
| Contenitore | Database SQLite, un file per sensore per registrazione |
| Nome file | Include l’**ID del sensore**e un**timestamp**, ad es. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spettro per fotogramma | 135 punti, 340–1010 nm con incrementi di 5 nm, più tristimolo CIE XYZ |
| Unità | Irradianza spettrale calibrata, **W/m²/nm** (pacchetto di calibrazione di fabbrica + profilo del cappuccio applicato) |
| Metadati timbrati | ID del sensore (la chiave per recuperare la calibrazione di fabbrica di quell’unità) e il profilo del cappuccio in vigore — vedi [Profili dei cappucci e intervallo calibrato](caps-and-range.md) |

Il formato è identico per DAQ-U, DAQ-M, DAQ-E e DAQ-E-S, quindi l’elaborazione a valle non tiene conto di quale dispositivo di acquisizione abbia effettuato la registrazione. Una registrazione DAQ-E-S memorizza anche l’inclinazione, il rollio e il beccheggio dell’unità accanto a ogni spettro.

**Le registrazioni sopravvivono a un&#x27;interruzione di corrente (1.3.0).**Una registrazione in tempo reale viene scritta con un journal e una sincronizzazione completa del disco ad ogni commit, quindi un&#x27;interruzione di corrente o un processo terminato prematuramente comporta la perdita solo degli ultimi 3 secondi, non dell’intero file. Una registrazione interrotta durante la scrittura può lasciare un file `<name>.daq-journal` accanto al file `.daq`:**conservare e copiare entrambi i file insieme.** Chloros

riporta indietro il journal la prima volta che legge il file. Un disco occupato ritarda il file, mai le letture del sensore.

**Un DAQ-E che smette di trasmettere viene ricollegato (1.3.0).** Quando il flusso di un DAQ-E si interrompe (spegnimento e riaccensione dello switch, perdita di connessione), il backend lo ricollega con le impostazioni precedenti e la registrazione su di esso prosegue nello stesso file `.daq` con l’interruzione contrassegnata nel file.

La registrazione calibrata richiede il pacchetto di calibrazione di fabbrica del sensore. Per i DAQ-U e i DAQ-M, il backend recupera il pacchetto dal cloud di MAPIR

in base all’ID del sensore (la registrazione viene rifiutata se non è possibile); le unità DAQ-E sono esenti perché conservano la calibrazione sul dispositivo stesso.

## Registrazione dall’interfaccia grafica

La registrazione nell’interfaccia grafica richiede un **progetto aperto** (in caso contrario i pulsanti di registrazione sono disabilitati):

* **Registra tutto / Interrompi tutto** — nella parte superiore della barra laterale dei sensori di luce; avvia o interrompe contemporaneamente una registrazione `.daq` su tutti i sensori collegati.
* **Registra / Interrompi registrazione** — per singolo sensore, nella finestra modale delle impostazioni (icona a forma di ingranaggio). Durante la registrazione, nelle righe delle informazioni in tempo reale del sensore compare un indicatore rosso “REC”.

I file vengono scritti in `<project>/light_sensor/` e, quando una registrazione si interrompe — sia tramite Interrompi, Interrompi tutto o scollegando un sensore di registrazione — il file `.daq` completato viene **aggiunto automaticamente al progetto aperto**. Appare nell’elenco dei file del progetto senza necessità di aggiungerlo manualmente, già pronto per l’elaborazione della riflettanza.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Registrazione dalla CLI



La CLI

effettua la registrazione tramite il pool di sensori del backend (il backend deve essere in esecuzione — questi comandi sono client HTTP

leggeri):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Ottieni il valore `--sensor-id` da `chloros-cli daq pool-list`. Due impostazioni predefinite da tenere a mente:

| Opzione | Impostazione predefinita |
| --- | --- |
| `--duration` | `0` — registra fino a `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` sul filesystem del **backend**, non su quello della CLI

|

La distinzione relativa alla directory di output è importante quando la CLI

punta a un backend su un’altra macchina: il file viene salvato dove è in esecuzione il backend. Se il backend si trova sullo stesso computer, un percorso relativo `-o` (come `./out` sopra) viene risolto rispetto alla cartella in cui è stato eseguito il comando.

## Registrazione da Python

`DAQSensorSession` (restituito da `chloros_sdk.connect_daq_sensor()`) espone la stessa registrazione in pool: `record_start(output_dir=None, device_name=None)` restituisce il percorso del file, mentre `record_stop()` restituisce `{path, rows}`. Consultare la [Guida di riferimento dell’SDK

](../reference/sdk-reference.md) per l’API

completa della sessione. Le classi hardware dirette dell’SDK

(solo per installazioni desktop) scrivono le registrazioni su `~/Documents/DAQ/` per impostazione predefinita; per le build rilasciate, il percorso raggruppato sopra indicato è quello supportato.

## Utilizzo di un file .daq in fase di elaborazione

Per ricavare la riflettanza dalle immagini, Chloros

necessita dell’irraggiamento discendente abbinato a ciascuna esposizione:

* **Conservare il file `.daq` insieme alle immagini.**In fase di elaborazione, la pipeline determina automaticamente l’**irraggiamento verso il basso con timestamp corrispondente** da un file `.daq` registrato (qualsiasi modello DAQ) — oppure da un file `.csv` nativo di DAQ-M — presente insieme alle immagini. Le registrazioni GUI soddisfano automaticamente questo requisito, poiché vengono aggiunte al progetto nel momento stesso in cui terminano.
* **La calibrazione viene recuperata su richiesta.** Se un pacchetto di calibrazione di fabbrica per singola telecamera o per singolo DAQ non è già memorizzato nella cache locale, Chloros

lo recupera automaticamente dal cloud di MAPIR

al primo utilizzo (è necessaria una connessione a Internet una sola volta; viene memorizzato nella cache come `~/.chloros/`).
* **Le acquisizioni in tempo reale scrivono il proprio file sidecar.** Per ogni fotogramma di riflettanza acquisito in tempo reale, il valore DAQ effettivamente utilizzato viene salvato come file `.daq` nella cartella `daq/` delle acquisizioni, in modo che l’acquisizione possa essere rielaborata in un secondo momento senza la registrazione originale.
* **Registrazioni dai registratori `chloros_scripts` (1.3.0).**Un file `.daq` generato dai registratori pubblici [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) viene indicato come modello DAQ che lo ha prodotto, e il suo limite massimo può essere impostato in**Impostazioni del progetto → Sensore di luce DAQ** come per qualsiasi altra registrazione. Questi registratori scrivono i conteggi grezzi del sensore; Chloros

recupera la calibrazione di fabbrica del sensore tramite la porta seriale e la applica.
* **A chi appartiene la lettura (1.3.0).**Accanto a ciascuna acquisizione `.daq`, un `.daq.cameras.json` indica la o le telecamere i cui fotogrammi sono descritti da quella lettura, e l’elaborazione la abbina**esclusivamente alle immagini provenienti da quelle telecamere**. Una telecamera autonoma priva di un proprio sensore di luce, quindi, non prende più in prestito la lettura di un’altra telecamera: è necessario associarle un sensore oppure registrarne uno. Una registrazione effettuata con lo stesso DAQ (GUI Record, `daq pool-record`) e qualsiasi `.daq` privo di quel file (comprese le acquisizioni precedenti) continua a servire tutte le telecamere.

## Recuperare l’irraggiamento

L’elaborazione di un progetto esporta anche tutte le registrazioni dei sensori di luce in esso contenute in una
`Light Sensor/` accanto ai prodotti immagine. Ciò **non** richiede immagini: una
registrazione del sensore di luce da sola costituisce un’acquisizione completa, e una cartella contenente solo file `.daq`
è un input valido. Un progetto che contiene solo registrazioni del sensore di luce le elenca e
le elabora sia dall’app che dalla CLI

, e rimangono elencate dopo l’esecuzione. L’esecuzione
riporta quanti prodotti del sensore di luce sono stati scritti.

| Prodotto | Che cos’è |
| --- | --- |
| `<name>_calibrated.daq` | Un archivio rielaborabile con lo stesso schema di una registrazione in tempo reale, che ora dichiara il pacchetto di calibrazione che lo ha generato. Reimportarlo **non** comporta una seconda calibrazione. |
| `<name>_calibrated.csv` | Irradianza spettrale in W/m²/nm sulla griglia di lunghezze d’onda propria del sensore, una riga per lettura, più colonne fotometriche: potenza totale, lux fotopici e scotopici, PPFD con la sua suddivisione in blu/verde/rosso e lunghezza d’onda di picco. |

Un DAQ-U o DAQ-M il cui pacchetto di calibrazione non può essere recuperato — perché si è offline o
perché quel sensore non ha alcuna calibrazione in archivio — viene **saltato con una motivazione**, senza mai essere salvato
come file “calibrato” contenente i conteggi grezzi. Connettersi a Internet ed eseguire nuovamente l’operazione. Un DAQ-E
dispone di una propria calibrazione, quindi ne ha bisogno solo quando l’unità non è collegata e
non c’è nulla memorizzato nella cache locale.

### DAQ-A: conteggi grezzi, e perché questa è la risposta giusta

Il **DAQ-A** è antecedente al sistema dei pacchetti di calibrazione per porta seriale e non ha alcun pacchetto da
recuperare. Non si tratta di una svista: un DAQ-A viene calibrato sul campo rispetto a un
bersaglio di riflettanza, e la calibrazione basata sul bersaglio richiede solo la risposta *relativa*
del sensore — che è esattamente ciò che rappresentano i suoi conteggi grezzi. Chloros

oggi effettua la calibrazione proprio con questi.

Quindi una registrazione DAQ-A viene esportata, ma con un nome diverso:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, non `_calibrated` — un nome file diverso anziché un flag all’interno del file,
perché l’indicazione deve rimanere valida anche quando il file viene inviato via e-mail come semplice nome. L’intestazione `.csv`
indica `raw spectral sensor counts (NOT irradiance)` e avverte che i valori sono
comparabili **all’interno** del file e non tra sensori diversi. Le colonne che hanno significato
solo per l’irraggiamento reale — potenza totale, lux, PPFD — vengono lasciate vuote anziché
essere calcolate dai conteggi.

Le registrazioni DAQ-A-SD più vecchie (schema v1.01 / v1.02) registrano solo l’ora di scrittura del file, non un
timestamp per ogni lettura. Chloros

non abbinerà le immagini a tali dati — associare un fotogramma a un
tempo di scrittura sarebbe errato, pur non sembrando tale — ma l’esportazione li legge correttamente e
il file CSV

indica su quale orologio si riferisce.

Per una panoramica completa sulla riflettanza — sensore singolo con fotocamera e doppio sensore ambiente/oggetto — consultare [Flussi di lavoro sulla riflettanza](reflectance.md).
