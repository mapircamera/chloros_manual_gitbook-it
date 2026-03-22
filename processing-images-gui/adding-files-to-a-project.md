# Aggiunta di file a un progetto

Una volta creato o aperto un progetto in Chloros, il passo successivo consiste nell&#x27;aggiungere le immagini multispettrali per avviare l&#x27;elaborazione. La scheda &quot;File Browser&quot;<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> semplifica l&#x27;importazione delle immagini e la gestione del set di dati.

## Accesso al File Browser

1. Aprire o creare un progetto in Chloros
2. Fare clic sull&#x27;icona **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> nella barra laterale sinistra
3. Il pannello File Browser visualizzerà l&#x27;elenco dei file del progetto

{% hint style="info" %}
**Tipi di file supportati**: Chloros supporta file immagine RAW+JPG e JPG provenienti dalle fotocamere MAPIR, Survey3W e Survey3N. Si consiglia di utilizzare solo file RAW+JPG.
{% endhint %}

***

## Aggiunta di immagini al progetto

Esistono due modi principali per aggiungere immagini al progetto:

### Metodo 1: Aggiungi file

Utilizza questa opzione per importare singoli file immagine o una piccola selezione di file.

1. Fai clic sul pulsante **&quot;Aggiungi file&quot;** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> nella parte superiore del pannello Esplora file
2. Accedere alla cartella contenente le immagini
3. Selezionare uno o più file immagine (tenere premuto **Ctrl** per selezionare più file)
4. Fare clic su **&quot;Apri&quot;** per importare i file selezionati

### Metodo 2: Aggiungi cartella

Utilizzare questa opzione per importare tutte le immagini da una cartella in una sola volta.

1. Fare clic sul pulsante **&quot;Aggiungi cartella&quot;** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> nella parte superiore del pannello Esplora file
2. Vai alla cartella contenente le immagini della sessione di acquisizione e selezionala
3. Fai clic su **&quot;Seleziona cartella&quot;** per importare tutte le immagini supportate da quella cartella***

## Comprendere la tabella di Esplora file

Una volta importate, le immagini vengono visualizzate in una tabella con le seguenti colonne:

### Nome file

* Nome file originale della fotocamera
* Mantiene la convenzione di denominazione della fotocamera (ad es. IMG\_0001.RAW)

### Data e ora

* Data e ora di acquisizione dell&#x27;immagine
* Estratte dai metadati EXIF dell&#x27;immagine
* Utilizzate per la sincronizzazione PPK e il rilevamento del bersaglio di calibrazione

### Modello della fotocamera

* Configurazione della fotocamera e del filtro rilevata automaticamente
* Esempi: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Utilizzata per applicare i profili di elaborazione corretti

### Colonna Target (casella di controllo)

* Selezionare questa casella per le immagini che contengono target di calibrazione
* Accelera notevolmente il rilevamento dei target durante l&#x27;elaborazione
* Per i dettagli, consultare [Scelta delle immagini target](choosing-target-images.md)

### Visualizzazione dei metadati delle immagini

Facendo clic sul pulsante di attivazione/disattivazione nell&#x27;angolo in alto a destra sopra la tabella, vengono visualizzati i metadati dell&#x27;immagine selezionata nell&#x27;area della griglia delle immagini.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Gestione dei file nel progetto

### Rimozione dei file

Per rimuovere immagini indesiderate dal progetto:

1. Selezionare una o più immagini nella tabella del File Browser
2. Fare clic sul pulsante **&quot;Rimuovi selezionati&quot;** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> 3. Confermare la rimozione (i file non vengono eliminati dal disco, ma solo rimossi dal progetto)

### Ordinamento e filtraggio

* **Ordina per colonna**: cliccare su qualsiasi intestazione di colonna per ordinare le immagini
* **Ordina per data e ora**: utile per organizzare sequenze di acquisizione cronologiche
* **Filtro per modello di fotocamera**: raggruppare le immagini per tipo di fotocamera se si utilizzano più fotocamere***

## Anteprima immagine

### Visualizzazione dell&#x27;immagine completa

Fare clic su qualsiasi miniatura dell&#x27;immagine nel Browser file per visualizzarla nell&#x27;area di anteprima principale:

1. L&#x27;immagine appare nel pannello di anteprima centrale
2. Utilizzare i controlli di zoom per esaminare i dettagli dell&#x27;immagine
3. Passare da un&#x27;immagine all&#x27;altra utilizzando i tasti freccia

### Navigazione rapida

* **Immagine precedente**: fare clic sulla freccia sinistra o premere il tasto ←
* **Immagine successiva**: clicca sulla freccia destra o premi il tasto →
* **Zoom avanti/indietro**: usa la rotellina del mouse o i pulsanti di zoom
* **Panoramica**: clicca e trascina sull&#x27;immagine quando è ingrandita***

## Gestione dei file duplicati

Chloros rileva e ignora automaticamente i file duplicati:

* I file con nomi identici vengono saltati
* Impedisce l&#x27;elaborazione doppia accidentale
* Viene visualizzato un messaggio di avviso quando vengono rilevati dei duplicati

{% hint style="warning" %}
**Importante**: Non rinominare o modificare i file immagine originali prima dell&#x27;importazione. Chloros si basa sui nomi dei file originali e sui metadati per un&#x27;elaborazione corretta.
{% endhint %}

***

## Set di dati di telecamere miste

Se il progetto contiene immagini provenienti da più telecamere MAPIR:

1. Chloros rileva automaticamente il modello di ciascuna telecamera
2. Ogni tipo di telecamera viene elaborato con il proprio profilo di calibrazione appropriato
3. Il File Browser visualizza il modello di fotocamera nella colonna &quot;Modello fotocamera&quot;
4. L&#x27;elaborazione applica le impostazioni corrette per ciascun tipo di fotocamera

**Scenario di esempio**: configurazione a doppia fotocamera Survey3W + RGN + Survey3N + OCN***

## Best practice

### Organizzazione prima dell&#x27;importazione

* Conservare le immagini dei target di calibrazione nella stessa cartella delle immagini di rilevamento
* Mantenere la struttura originale delle cartelle dalla fotocamera/scheda SD
* Non mescolare set di dati di sessioni diverse in un unico progetto

### Denominazione dei file

* Conservare i nomi originali dei file della fotocamera (IMG\_0001.RAW, ecc.)
* Non rinominare i file prima dell&#x27;importazione
* I nomi originali contengono metadati importanti

### Immagini dei target di calibrazione

* Includere sempre 1-2 immagini dei target di calibrazione per sessione
* Acquisire i target prima e dopo la sessione di acquisizione
* Posizionare i target nelle stesse condizioni di illuminazione dell&#x27;area di acquisizione
* Contrassegnare le immagini dei target utilizzando la casella di controllo Target per velocizzare l&#x27;elaborazione

***

## Problemi comuni e soluzioni

### Immagini non visualizzate dopo l&#x27;importazione

**Possibili cause:**

* Formato file non supportato (solo RAW+JPG e JPG da fotocamere MAPIR)
* Le immagini provengono da fotocamere non MAPIR (vedere [Fotocamere supportate](../supported-cameras.md))
* File danneggiato o trasferimento incompleto dalla scheda SD

**Soluzione**: Verificare la compatibilità del formato file e del modello di fotocamera

### Modello di fotocamera non rilevato

**Possibili cause:**

* Metadati EXIF modificati
* Immagini modificate con software esterno
* Trasferimento file incompleto

**Soluzione**: Reimportare i file originali e non modificati dalla fotocamera/scheda SD

### Mancano i timestamp

**Possibili cause:**

* Orologio della fotocamera non impostato correttamente
* Dati EXIF rimossi da software esterno

**Soluzione**: Verificare che le impostazioni dell&#x27;ora della fotocamera fossero corrette durante l&#x27;acquisizione***

## Passi successivi

Una volta importati i file:

1. **Controllare l&#x27;elenco dei file** - Assicurarsi che tutte le immagini siano state caricate correttamente
2. **Controlla i modelli di fotocamera** - Verifica il corretto rilevamento della fotocamera
3. **Seleziona le immagini di destinazione** - Vedi [Scelta delle immagini di destinazione](choosing-target-images.md)
4. **Modifica le impostazioni** - Configura le opzioni di elaborazione in [Impostazioni del progetto](adjusting-project-settings.md)
5. **Avvia l&#x27;elaborazione** - Vedi [Avvio dell&#x27;elaborazione](starting-the-processing.md)

Per informazioni dettagliate sulla configurazione del progetto, vedi [Modifica delle impostazioni del progetto](adjusting-project-settings.md).
