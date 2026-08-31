# Aggiunta di file a un progetto

Una volta creato o aperto un progetto in Chloros, il passo successivo consiste nell&#x27;aggiungere le immagini multispettrali per avviare l&#x27;elaborazione. La scheda “File Browser” <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> semplifica l&#x27;importazione delle immagini e la gestione del set di dati.

## Accesso al File Browser

1. Aprire o creare un progetto in Chloros
2. Fare clic sull’icona **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> nella barra laterale sinistra
3. Il pannello File Browser mostrerà l’elenco dei file del progetto

{% hint style="info" %}
**Tipi di file supportati**:

* **Survey3W / Survey3N**: coppie RAW+JPG e immagini JPG (si consiglia RAW+JPG)
* **LATTICE**: acquisizioni `.tif` / `.tiff` — registrate tramite il controllo della telecamera Chloros o da un hub LATTICE
* **Dati del sensore di luce**: registrazioni `.daq` (DAQ-U/M/E) e registrazioni del flusso discendente DAQ-M `.csv` — importate insieme alle immagini per eseguire la calibrazione della riflettanza
{% endhint %}

***

## Aggiunta di immagini al progetto

Esistono due modi principali per aggiungere immagini al progetto:

### Metodo 1: Aggiungi file

Utilizzare questa opzione per importare singoli file immagine o una piccola selezione di file.

1. Fare clic sul pulsante **&quot;Aggiungi file&quot;** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> nella parte superiore del pannello Esplora file
2. Accedere alla cartella contenente le immagini
3. Selezionare uno o più file immagine (tenere premuto **Ctrl** per selezionare più file)
4. Fare clic su **&quot;Apri&quot;** per importare i file selezionati

### Metodo 2: Aggiungi cartella

Utilizza questa opzione per importare tutte le immagini da una cartella in una sola volta. È possibile selezionare **più cartelle** in un&#x27;unica finestra di dialogo.

1. Fare clic sul pulsante **&quot;Aggiungi cartella&quot;** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> nella parte superiore del pannello Esplora file
2. Accedere alle cartelle contenenti le immagini della sessione di acquisizione e selezionarle
3. Fare clic su **&quot;Seleziona cartella&quot;** per importare tutte le immagini supportate

{% hint style="info" %}
**I file che non riescono a caricarsi vengono segnalati.** Se una cartella contiene file che Chloros riconosce ma non riesce a caricare, un avviso lo segnala: le immagini non scompaiono silenziosamente dalla griglia.
{% endhint %}

***

## Importazione delle cartelle di acquisizione LATTICE

Le acquisizioni LATTICE vengono salvate con **una sottocartella per ogni livello di esportazione** — ad esempio `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — con il file di downwelling corrispondente `.daq` nella directory principale:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Indicare la cartella radice delle acquisizioni** (`output/` sopra). Quando la cartella selezionata non contiene immagini ma presenta sottocartelle, Chloros vi accede automaticamente: le sottocartelle di livello e la cartella principale `.daq` vengono acquisite tutte in una volta sola.**Come avviene l’importazione delle acquisizioni:*** Ogni acquisizione viene importata come una **singola immagine**, raggruppata per acquisizione (non una voce per livello). Gli altri livelli della stessa acquisizione appaiono come modalità di visualizzazione di quell’unica immagine.
* **L’elaborazione parte sempre dal fotogramma grezzo.** Gli altri livelli sono visualizzabili, ma solo `raw` viene mai inserito nella pipeline: rielaborare un prodotto già elaborato comporterebbe un&#x27;applicazione doppia delle correzioni, quindi Chloros viene scartato. Un’esportazione reimportata non può mai occupare lo slot raw di una cattura.
* Una cartella di cattura salvata **senza** file raw viene importata e visualizzata normalmente, ma l’elaborazione la salta e lo segnala nel log. (Il flag CLI `--input-level` può forzare un punto di ingresso in questo caso — vedi [il Riferimento CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**Le sessioni dell’hub LATTICE** vengono importate allo stesso modo: selezionare “Aggiungi cartella” indicando la cartella della sessione copiata dall’hub (che contiene `raw/` e `previews/`), insieme a qualsiasi log di downwelling DAQ-M `.csv`. Se la calibrazione della fotocamera o del DAQ non è ancora memorizzata nella cache del computer, Chloros la recupera automaticamente in base al numero di serie al momento dell’importazione (richiede una connessione a Internet una sola volta).***

## Come interpretare la tabella del browser dei file

Una volta importate, le immagini vengono visualizzate in una tabella con le seguenti colonne:

### Nome file

* Nome file originale della fotocamera
* Mantiene la convenzione di denominazione della fotocamera (ad es., IMG\_0001.RAW o capture\_20260816\_101500\_SN213800234\_raw.tif)

### Data e ora

* Data e ora di acquisizione dell’immagine
* Estratte dai metadati EXIF dell’immagine
* Utilizzate per l’abbinamento dei sensori di luce, la sincronizzazione PPK e la pianificazione dei bersagli di calibrazione

### Modello della fotocamera

* Configurazione della fotocamera e del filtro rilevata automaticamente
* Esempi Survey3: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Esempi LATTICE: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Utilizzato per applicare i profili di elaborazione corretti

### Colonna &quot;Target&quot; (casella di selezione)

* Selezionare questa casella per le immagini che contengono bersagli di calibrazione
* Quando è selezionata almeno un&#x27;immagine, **solo le immagini selezionate vengono analizzate** alla ricerca di bersagli
* Per ulteriori dettagli, consultare [Scelta delle immagini bersaglio](choosing-target-images.md)

### Visualizzazione dei metadati delle immagini

Facendo clic sul pulsante di attivazione/disattivazione nell’angolo in alto a destra sopra la tabella, vengono visualizzati i metadati dell’immagine selezionata nell’area della griglia delle immagini.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## File dei sensori di luce nel progetto

* I file `.daq` e `.csv` compaiono nell’elenco del File Browser ma non sono immagini cliccabili: forniscono l’irraggianza discendente per la calibrazione della riflettanza.
* Ogni file `.daq`/`.csv` importato è elencato in **Impostazioni progetto → Sensore di luce DAQ**, dove è possibile verificare la correzione del cappuccio diffusore applicata a ciascun file. Vedi [Modifica delle impostazioni del progetto](adjusting-project-settings.md).
* Le registrazioni effettuate nella scheda **Sensori di luce** vengono aggiunte automaticamente al progetto aperto — non è necessaria alcuna importazione manuale.***

## Gestione dei file nel progetto

### Rimozione dei file

Per rimuovere immagini indesiderate dal progetto:

1. Selezionare una o più immagini nella tabella del File Browser
2. Fare clic sul pulsante **&quot;Rimuovi selezionati&quot;** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Confermare la rimozione (i file non vengono eliminati dal disco, ma solo rimossi dal progetto)

### Ordinamento e filtraggio

* **Ordina per colonna**: clicca su qualsiasi intestazione di colonna per ordinare le immagini
* **Ordinamento per data e ora**: utile per organizzare sequenze di acquisizione in ordine cronologico
* **Filtro per modello di fotocamera**: raggruppa le immagini in base al tipo di fotocamera se ne utilizzi più di una***

## Anteprima delle immagini

### Visualizzazione dell’immagine completa

Fare clic su una qualsiasi miniatura nell’Esplora file per visualizzarla nell’area di anteprima principale:

1. L’immagine appare nel pannello di anteprima centrale
2. Utilizzare i controlli di zoom per esaminare i dettagli dell’immagine
3. Passare da un’immagine all’altra utilizzando i tasti freccia

### Navigazione rapida

* **Immagine precedente**: fare clic sulla freccia sinistra o premere il tasto ←
* **Immagine successiva**: fare clic sulla freccia destra o premere il tasto →
* **Ingrandisci/Riduci**: utilizzare la rotellina del mouse o i pulsanti di zoom
* **Panoramica**: fare clic e trascinare sull’immagine quando è ingrandita***

## Gestione dei file duplicati

Chloros rileva automaticamente e ignora i file duplicati:

* I file con nomi identici vengono saltati
* Impedisce l’elaborazione accidentale di file duplicati
* Viene visualizzato un messaggio di avviso quando vengono rilevati dei duplicati

{% hint style="warning" %}
**Importante**: non rinominare né modificare i file immagine originali prima dell’importazione. Chloros si basa sui nomi file e sui metadati originali per un’elaborazione corretta.
{% endhint %}

***

## Set di dati con telecamere diverse

Se il progetto contiene immagini provenienti da più telecamere MAPIR:

1. Chloros rileva automaticamente il modello di ciascuna telecamera: Survey3, LATTICE o una combinazione di entrambi
2. Ogni tipo di telecamera viene elaborato con il proprio profilo di calibrazione appropriato
3. Il Browser dei file mostra il modello della telecamera nella colonna “Modello telecamera”
4. A ogni telecamera viene assegnata una propria struttura di cartelle di output una volta elaborata

**Esempi di scenari**: configurazione a doppia telecamera con Survey3W, RGN + Survey3N, OCN, oppure un array LATTICE con un RGB master e diversi moduli a banda stretta***

## Best practice

### Organizzazione prima dell’importazione

* Conservare le immagini dei target di calibrazione nella stessa cartella delle immagini di rilevamento
* Conservare i file dei sensori di luce `.daq` / `.csv` di ciascuna sessione di acquisizione insieme alle immagini di quella sessione
* Mantenere la struttura originale delle cartelle proveniente dalla fotocamera/scheda SD/hub
* Non mescolare set di dati provenienti da sessioni diverse all’interno di un unico progetto

### Denominazione dei file

* Conservare i nomi originali dei file della fotocamera (IMG\_0001.RAW, capture\_..., ecc.)
* Non rinominare i file prima dell’importazione
* I nomi originali contengono metadati importanti

### Immagini di riferimento per la calibrazione

* Includere sempre 1-2 immagini di riferimento per la calibrazione per ogni sessione (Survey3; per LATTICE è possibile sostituirle con una registrazione DAQ — vedere [Scelta delle immagini di riferimento](choosing-target-images.md))
* Acquisire i bersagli prima e dopo la sessione di acquisizione
* Posizionare i bersagli nelle stesse condizioni di illuminazione dell’area di acquisizione
* Contrassegnare le immagini bersaglio utilizzando la casella di controllo “Target”

***

## Problemi comuni e soluzioni

### Immagini che non vengono visualizzate dopo l’importazione

**Possibili cause:**

* Formato file non supportato (vedere l’elenco dei tipi supportati nella parte superiore di questa pagina)
* Le immagini provengono da fotocamere diverse da MAPIR (vedere [Fotocamere supportate](../supported-cameras.md))
* File danneggiato o trasferimento incompleto dalla scheda SD

**Soluzione**: Verificare la compatibilità tra il formato del file e il modello della fotocamera e controllare l’avviso di caricamento dei file per individuare esattamente quali file non sono stati importati

### Modello della fotocamera non rilevato

**Possibili cause:**

* Metadati EXIF modificati
* Immagini modificate con software esterno
* Trasferimento dei file incompleto

**Soluzione**: Reimportare i file originali e non modificati dalla fotocamera o dalla scheda SD

### Mancano i timestamp

**Possibili cause:**

* Orologio della fotocamera non impostato correttamente
* Dati EXIF rimossi da software esterno

**Soluzione**: Verificare che le impostazioni dell’ora della fotocamera fossero corrette durante l’acquisizione

### Il progetto riaperto segnala file mancanti

Se i file di origine sono stati spostati o eliminati dall’ultima volta che il progetto è stato aperto, il codice Chloros indica **quali** file mancano, invece di aprire una griglia vuota. Ripristina i file nei percorsi originali oppure rimuovi le voci mancanti e reimportali.***

## Passaggi successivi

Una volta importati i file:

1. **Esamina l’elenco dei file** - Assicurati che tutte le immagini siano state caricate correttamente
2. **Controlla i modelli delle fotocamere** - Verifica che il rilevamento delle fotocamere sia corretto
3. **Contrassegnare le immagini di destinazione** - Vedi [Scelta delle immagini di destinazione](choosing-target-images.md)
4. **Modificare le impostazioni** - Configurare le opzioni di elaborazione in [Impostazioni del progetto](adjusting-project-settings.md)
5. **Avvia l&#x27;elaborazione** - Vedi [Avvio dell&#x27;elaborazione](starting-the-processing.md)

Per informazioni dettagliate sulla configurazione del progetto, consulta [Modifica delle impostazioni del progetto](adjusting-project-settings.md).
