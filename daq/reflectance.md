# Flussi di lavoro basati sulla riflettanza

Un sensore di luce DAQ converte le immagini radiometriche in dati di riflettanza. Esistono due flussi di lavoro distinti:

1. **Sensore singolo** — un sensore DAQ misura l’irraggiamento discendente mentre una fotocamera acquisisce l’immagine, e Chloros divide la radianza della fotocamera per tale valore di riferimento.
2. **A doppio sensore** — due sensori DAQ, uno rivolto verso il cielo e l’altro verso un oggetto, producono una curva di riflettanza spettrale in tempo reale senza l’utilizzo di una telecamera.

## Sensore singolo + telecamera (riferimento discendente)

Il DAQ funge da sensore di luce discendente (DLS): la fotocamera misura la radianza ascendente **L**(W/m²/sr/nm), il DAQ misura l’irraggiamento discendente**E** (W/m²/nm) e Chloros calcola la riflettanza per banda come:

> ρ = π · L / E

La lettura del DAQ è sempre **sincronizzata con l’esposizione tramite timestamp** — ecco perché il DAQ e le telecamere condividono un orologio regolato tramite PTP (vedi [Rete DAQ-E e sincronizzazione temporale](ethernet-ptp.md)). Adattare il cappuccio coseno Sunshine per il lavoro all’aperto e dichiararlo correttamente; la dichiarazione del cappuccio scala direttamente E (vedere [Profili dei cappucci e intervallo calibrato](caps-and-range.md)). Per i lavori quantitativi, tenere presente la caratteristica dello strumento: l’irraggiamento quantitativo deriva da una media di almeno 15 s di letture.

### Acquisizione in tempo reale

Associare il DAQ a una telecamera nella scheda Telecamere: il pannello delle impostazioni di ciascuna telecamera presenta un menu a tendina **Sensore di luce** che elenca tutti i DAQ collegati (DAQ-U/M/E) dalla scheda Sensori di luce; per un array sincronizzato, la selezione del sensore di luce a livello di array si propaga a ogni membro (le singole telecamere possono comunque sovrascrivere questa impostazione). Una volta associato, gli spettri del sensore alimentano lo slot DLS della telecamera e le esportazioni di riflettanza vengono divise per la lettura corrispondente. In un array, un membro dotato di un proprio sensore utilizza la propria lettura — per le sue bande nel composito in tempo reale, per i dati esportati e per l’esposizione automatica — mentre gli altri membri utilizzano quella dell’array.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Due comportamenti da tenere presenti:

* **Nessun DAQ associato → la riflettanza viene rifiutata, non simulata.** *Chloros
* rifiuta il prodotto di riflettanza e registra il motivo del salto, anziché restituire silenziosamente un prodotto di qualità inferiore.
* **La lettura utilizzata viene conservata.** Per ogni fotogramma di riflettanza, la lettura DAQ effettivamente applicata viene salvata come file `.daq` nella cartella `daq/` delle acquisizioni, in modo che l’acquisizione possa essere rielaborata in un secondo momento ([Registrazione e formato .daq](recording.md)). Un file `.daq.cameras.json` adiacente indica le telecamere a cui si riferisce, e la rielaborazione lo abbina esclusivamente alle immagini di quelle telecamere.

### Elaborazione delle immagini registrate

Per l’elaborazione post-volo, registrare un file `.daq` durante la sessione e conservarlo insieme alle immagini: la pipeline risolve automaticamente il flusso discendente abbinato al timestamp, recuperando eventuali calibrazioni di fabbrica mancanti dal cloud di MAPIR al primo utilizzo. Le registrazioni dell’interfaccia grafica vengono aggiunte automaticamente al progetto aperto al termine della registrazione.

Il riferimento di riflettanza è selezionabile in fase di elaborazione: `--reflectance-source` su `chloros-cli process`, oppure l’impostazione della sorgente di riflettanza nelle Impostazioni di progetto dell’interfaccia grafica:

| Valore | Comportamento |
| --- | --- |
| `auto` (impostazione predefinita) | Un bersaglio di calibrazione all’interno del fotogramma che ha superato il controllo di qualità (QA) costituisce il riferimento assoluto; la calibrazione DAQ in downwelling (ρ = π·L/E) funge da alternativa. Un bersaglio che supera il controllo di qualità con un flag (un pannello mancante o una banda non funzionante) non viene automaticamente preferito: le catene «bersaglio», «bersaglio+DAQ» e «solo DAQ» vengono provate in ordine di punteggio misurato sui pannelli tenuti da parte. |
| `daq` | DAQ autorevole |
| `target` | Target rigorosamente in-frame; nessuna sostituzione con DAQ |

Vedere [Bersagli di calibrazione](../calibration-targets.md) per i flussi di lavoro relativi ai bersagli e il [capitolo LATTICE](../lattice/README.md) e la [Guida di riferimento CLI](../reference/cli-reference.md) per la pipeline di elaborazione completa. Durante la lettura dei pixel di riflettanza esportati, utilizzare la scala indicata (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — si veda [Formati immagine di output](../output-image-formats.md).

### Bande al di fuori dell’intervallo calibrato del DAQ

L'intervallo calibrato radiometricamente del DAQ è di circa 374–974 nm. Chloros rifiuta la riflettanza basata sul DAQ per qualsiasi banda della fotocamera con meno della metà del proprio peso spettrale all'interno di tale intervallo, segnalando il motivo di esclusione `dls-uncalibrated-band-<nm>`. Tra gli SKU disponibili, ciò riguarda solo l’F988: la riflettanza dell’F988 viene calibrata utilizzando un pannello di riflettanza in scena; poiché la banda si trova al di fuori dell’intervallo calibrato del sensore di luce del DAQ, Chloros applica l’ultima acquisizione del pannello e la mantiene tra un rilevamento e l’altro. Se una telecamera F988 viene utilizzata in modalità solo DAQ, Chloros a rifiuta la riflettanza basata sul DAQ per quella banda con il motivo di esclusione `dls-uncalibrated-band-988`: il flusso di lavoro con il pannello è l’approccio supportato.

## Doppio sensore (ambientale + oggetto)

Due sensori DAQ — qualsiasi coppia, su qualsiasi sistema di trasporto — forniscono uno spettro di riflettanza in tempo reale senza bisogno di una telecamera: un sensore è rivolto verso il cielo (**Sorgente di luce ambientale**), l’altro verso il soggetto (**Scanner dell’oggetto**), e Chloros a calcola per ciascuna lunghezza d’onda:

> R(λ) = oggetto(λ) / ambiente(λ)

(zero quando ambiente ≤ 0).

### Nell’interfaccia grafica

Con entrambi i sensori collegati nella scheda Sensori di luce, apri il pannello di sovrapposizione per l’aggiunta di sensori (il pulsante “+” su una casella del grafico nella vista a griglia) e seleziona **Combina luce ambientale + oggetto**. Selezionare i due sensori dai menu a tendina**Sorgente di luce ambientale**e**Scanner oggetto**, quindi fare clic su**Crea**. Il gruppo appare come grafico a sé stante e come riga nella barra laterale con un badge verde**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Sotto il grafico di riflettanza (vista elenco), una **tabella degli indici di vegetazione** in tempo reale calcola gli indici dalla curva utilizzando i centri di banda a 450 nm (blu), 550 nm (verde), 670 nm (rosso) e 800 nm (NIR). Gli indici basati sul rapporto che annullano la scala assoluta (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) vengono sempre visualizzati; gli indici che richiedono la riflettanza assoluta (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) compaiono solo quando entrambi i sensori sono modelli calibrati in potenza.

### Apparente vs. Relativa — la regola di etichettatura

Chloros etichetta l’output a doppio sensore in base a ciò che la coppia di sensori può effettivamente dichiarare:

| Coppia di sensori | Etichetta |
| --- | --- |
| Entrambi i sensori calibrati — pacchetto di fabbrica caricato | **Riflettanza apparente** |
| Uno dei due sensori non calibrato | **Riflettanza relativa** |

Tutti e tre i modelli sono radiometrici: una volta caricato il pacchetto di calibrazione di fabbrica di un sensore, i suoi spettri sono espressi in W/m²/nm assoluti, quindi una coppia di sensori calibrati fornisce una riflettanza apparente assoluta — il trasporto non la determina. Un sensore che continua a trasmettere conteggi grezzi (pacchetto non raggiungibile) riduce il risultato a una curva relativa (la forma spettrale rimane comunque valida). Entrambi i sensori dovrebbero avere limiti massimi correttamente dichiarati ([Profili dei limiti massimi e intervallo calibrato](caps-and-range.md)).

### Da Python


Non esiste una chiamata dedicata ai sensori duali nell’interfaccia SDK aggregata: aprite due sessioni con `chloros_sdk.connect_daq_sensor()` e calcolate voi stessi il rapporto tra i loro spettri `latest()`, applicando la stessa convenzione di etichettatura. (Esiste anche uno strumento di registrazione a doppio sensore nell’interfaccia hardware diretta interna di MAPIR, elencato nella [Guida di riferimento CLI](../reference/cli-reference.md) per completezza — non fa parte della CLI fornita; il flusso di lavoro GUI sopra descritto è il percorso attivo supportato.)
