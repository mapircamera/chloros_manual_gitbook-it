# Registrazione e formato .daq

Un file `.daq` è il formato di registrazione del sensore di luce di Chloros: un **database SQLite** contenente frame spettrali calibrati provenienti da un sensore DAQ. Ne basta registrare uno durante una sessione di acquisizione e la pipeline di riflettanza potrà in seguito dividere ogni immagine per l’irraggiamento discendente misurato in quel preciso momento.

## Cosa contiene un file .daq

| Proprietà | Valore |
| --- | --- |
| Contenitore | Database SQLite, un file per sensore per registrazione |
| Nome file | Include l’**ID del sensore**e un**timestamp**, ad es. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spettro per fotogramma | 135 punti, 340–1010 nm con incrementi di 5 nm, più tristimolo CIE XYZ |
| Unità | Irradianza spettrale calibrata, **W/m²/nm** (pacchetto di calibrazione di fabbrica + profilo del cappuccio applicato) |
| Metadati timbrati | ID del sensore (la chiave per recuperare la calibrazione di fabbrica di quell’unità) e il profilo del cappuccio in vigore — vedi [Profili dei cappucci e intervallo calibrato](caps-and-range.md) |

Il formato è identico per DAQ-U, DAQ-M e DAQ-E, quindi l’elaborazione a valle non tiene conto di quale dispositivo di trasmissione abbia effettuato la registrazione.

La registrazione calibrata richiede il pacchetto di calibrazione di fabbrica del sensore. Per DAQ-U e DAQ-M il backend recupera il pacchetto dal cloud di MAPIR in base all’ID del sensore (se non ci riesce, la registrazione viene rifiutata); le unità DAQ-E sono esenti perché conservano la calibrazione sul dispositivo stesso.

## Registrazione dall’interfaccia grafica

La registrazione nell’interfaccia grafica richiede un **progetto aperto** (in caso contrario i pulsanti di registrazione sono disabilitati):

* **Registra tutto / Interrompi tutto** — nella parte superiore della barra laterale dei sensori di luce; avvia o interrompe contemporaneamente una registrazione `.daq` su tutti i sensori collegati.
* **Registra / Interrompi registrazione** — per singolo sensore, nella finestra modale delle impostazioni (icona a forma di ingranaggio). Durante la registrazione, nelle righe delle informazioni in tempo reale del sensore compare un indicatore rosso “REC”.

I file vengono salvati in `<project>/light_sensor/` e, quando una registrazione si interrompe — sia tramite «Interrompi», «Interrompi tutto» o scollegando un sensore di registrazione — il file `.daq` completato viene **aggiunto automaticamente al progetto aperto**. Appare nell’elenco dei file del progetto senza necessità di aggiungerlo manualmente, già pronto per l’elaborazione della riflettanza.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Registrazione da CLI

L’CLI effettua la registrazione tramite il pool di sensori del backend (il backend deve essere in esecuzione — questi comandi sono client HTTP leggeri):

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

Recupera il valore `--sensor-id` da `chloros-cli daq pool-list`. Due impostazioni predefinite da tenere a mente:

| Opzione | Predefinito |
| --- | --- |
| `--duration` | `0` — registra fino a `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` sul filesystem del **backend**, non su quello di CLI |

La distinzione relativa alla directory di output è importante quando CLI punta a un backend su un&#x27;altra macchina: il file viene salvato dove è in esecuzione il backend.

## Registrazione da Python

`DAQSensorSession` (restituito da `chloros_sdk.connect_daq_sensor()`) espone la stessa registrazione in pool: `record_start(output_dir=None, device_name=None)` restituisce il percorso del file, mentre `record_stop()` restituisce `{path, rows}`. Consultare la [Guida di riferimento di SDK](../reference/sdk-reference.md) per la sessione completa API. Le classi hardware dirette di SDK (solo per installazioni desktop) scrivono le registrazioni su `~/Documents/DAQ/` per impostazione predefinita; per le build rilasciate, il percorso condiviso sopra indicato è quello supportato.

## Utilizzo di un file .daq in fase di elaborazione

Per ricavare la riflettanza dalle immagini, Chloros necessita di un’irradianza discendente abbinata a ciascuna esposizione:

* **Conservare il file `.daq` insieme alle immagini.**In fase di elaborazione, la pipeline risolve automaticamente l’**irraggiamento discendente con timestamp corrispondente** da un file `.daq` registrato (qualsiasi modello DAQ) — oppure da un file `.csv` nativo di DAQ-M — presente insieme alle immagini. Le registrazioni della GUI soddisfano automaticamente questo requisito, poiché vengono aggiunte al progetto nel momento stesso in cui terminano.
* **La calibrazione viene recuperata su richiesta.** Se un pacchetto di calibrazione di fabbrica per singola telecamera o per singolo DAQ non è già memorizzato nella cache locale, Chloros lo recupera automaticamente dal cloud di MAPIR al primo utilizzo (è richiesta una connessione a Internet una sola volta; viene memorizzata nella cache sotto `~/.chloros/`).
* **Le acquisizioni in tempo reale generano il proprio file sidecar.** Per ogni fotogramma di riflettanza acquisito in tempo reale, la lettura DAQ effettivamente utilizzata viene salvata come file sidecar `.daq` accanto alle immagini, in modo che l’acquisizione possa essere rielaborata in un secondo momento senza la registrazione originale.

## Recuperare l’irradianza

L’elaborazione di un progetto esporta anche tutte le registrazioni dei sensori di luce in esso contenute in una
cartella denominata `Light Sensor/` accanto ai prodotti immagine. Ciò **non** richiede la presenza di immagini: un
sensore di luce utilizzato da solo costituisce una registrazione completa, e una cartella contenente solo file `.daq`
è un input valido. L’esecuzione riporta il numero di prodotti del sensore di luce che ha generato.

| Prodotto | Descrizione |
| --- | --- |
| `<name>_calibrated.daq` | Un archivio rielaborabile con lo stesso schema di una registrazione in tempo reale, che ora dichiara il pacchetto di calibrazione che lo ha prodotto. Reimportarlo **non** comporta una seconda calibrazione. |
| `<name>_calibrated.csv` | Irradianza spettrale in W/m²/nm sulla griglia di lunghezze d’onda propria del sensore, una riga per lettura, più colonne fotometriche: potenza totale, lux fotopici e scotopici, PPFD con la sua suddivisione in blu/verde/rosso e lunghezza d’onda di picco. |

Un DAQ-U o DAQ-M il cui pacchetto di calibrazione non può essere recuperato — perché si è offline o
perché quel sensore non ha alcuna calibrazione in archivio — viene **saltato con una motivazione**, senza mai essere salvato
come file &quot;calibrato&quot; contenente i conteggi grezzi. Connettersi a Internet ed eseguire nuovamente l&#x27;operazione. Un DAQ-E
dispone della propria calibrazione, quindi ne ha bisogno solo quando l’unità non è connessa e
non c’è nulla in cache localmente.

### DAQ-A: conteggi grezzi, e perché questa è la risposta giusta

Il **DAQ-A** è antecedente al sistema dei pacchetti di calibrazione per porta seriale e non ha alcun pacchetto da
recuperare. Non si tratta di una svista: un DAQ-A viene calibrato sul campo rispetto a un
target di riflettanza, e la calibrazione basata sul target richiede solo la risposta *relativa*
del sensore — che è esattamente ciò che rappresentano i suoi conteggi grezzi. Chloros effettua la calibrazione con essi oggi.

Quindi una registrazione DAQ-A viene esportata, ma con un nome diverso:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, non `_calibrated` — un nome file diverso anziché un flag all’interno del file,
perché l’indicazione deve rimanere visibile anche quando il file viene inoltrato via e-mail come semplice nome. L’intestazione `.csv`
indica `raw spectral sensor counts (NOT irradiance)` e avverte che i valori sono
comparabili **all’interno** del file e non tra sensori diversi. Le colonne che hanno significato
solo per l’irradianza reale — potenza totale, lux, PPFD — vengono lasciate vuote anziché
essere calcolate dai conteggi.

Le registrazioni DAQ-A-SD più vecchie (schema v1.01 / v1.02) registrano solo l’ora di scrittura del file, non un
timestamp per ogni lettura. Chloros non abbinerà le immagini a quelle — associare un fotogramma a un
tempo di scrittura sarebbe errato senza che sembri mai sbagliato — ma l’esportazione le legge correttamente e
il file CSV indica su quale orologio si basa.

Per una panoramica completa sulla riflettanza — sensore singolo con fotocamera e doppio sensore ambiente/oggetto — consultare [Flussi di lavoro sulla riflettanza](reflectance.md).
