# Aggiunta di file a un progetto

Una volta creato o aperto un progetto in Chloros, il passo successivo consiste nell&#x27;aggiungere le immagini multispettrali per avviare l&#x27;elaborazione. La scheda File Browser<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> semplifica l&#x27;importazione delle immagini e la gestione del set di dati.

## Accesso al File Browser

1. Aprire o creare un progetto in Chloros
2. Fare clic sull&#x27;icona **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> nella barra laterale sinistra
3. Il pannello File Browser visualizzerà l&#x27;elenco dei file del progetto

{% suggerimento style=&quot;info&quot; %}
**Tipi di file supportati**: Chloros supporta file immagine RAW+JPG e JPG provenienti da fotocamere MAPIR Survey3W e Survey3N. Si consiglia di utilizzare solo RAW+JPG.
{% endhint %}

***

## Aggiunta di immagini al progetto

Esistono due modi principali per aggiungere immagini al progetto:

### Metodo 1: aggiunta di file

Utilizzare questa opzione per importare singoli file immagine o una piccola selezione di file.

1. Fare clic sul pulsante **&quot;Aggiungi file&quot;** nella parte superiore del pannello File Browser.
2. Passare alla cartella contenente le immagini.
3. Selezionare uno o più file immagine (tenere premuto **Ctrl** per selezionare più file).
4. Fare clic su **&quot;Apri&quot;** per importare i file selezionati.

### Metodo 2: aggiunta di cartelle

Utilizzare questa opzione per importare tutte le immagini da una cartella contemporaneamente.

1. Fare clic sul pulsante **&quot;Aggiungi cartella&quot;** nella parte superiore del pannello File Browser.
2. Individuare e selezionare la cartella contenente le immagini della sessione di acquisizione.
3. Fare clic su **&quot;Seleziona cartella&quot;** per importare tutte le immagini supportate da quella cartella.

***

## Informazioni sulla tabella File Browser

Una volta importate, le immagini vengono visualizzate in una tabella con le seguenti colonne:

### Miniatura

* Piccola anteprima di ciascuna immagine
* Fare clic sulla miniatura per visualizzare l&#x27;immagine completa nell&#x27;area di anteprima principale

### Nome file

* Nome file originale della fotocamera
* Mantiene la convenzione di denominazione della fotocamera (ad esempio, IMG\_0001.RAW)

### Data e ora

* Data e ora di acquisizione dell&#x27;immagine
* Estratto dai metadati EXIF dell&#x27;immagine
* Utilizzato per la sincronizzazione PPK e il rilevamento del target di calibrazione

### Modello della fotocamera

* Configurazione della fotocamera e del filtro rilevata automaticamente
* Esempi: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Utilizzato per applicare i profili di elaborazione corretti

### Colonna bersaglio (casella di controllo)

* Selezionare questa casella per le immagini che contengono bersagli di calibrazione
* Velocizza notevolmente il rilevamento dei bersagli durante l&#x27;elaborazione
* Per ulteriori dettagli, consultare [Scelta delle immagini bersaglio](choosing-target-images.md)

***

## Gestione dei file nel progetto

### Rimozione dei file

Per rimuovere le immagini indesiderate dal progetto:

1. Selezionare una o più immagini nella tabella del browser dei file
2. Fare clic sul pulsante **&quot;Rimuovi selezionati&quot;**
3. Confermare la rimozione (i file non vengono eliminati dal disco, ma solo rimossi dal progetto)

### Ordinamento e filtraggio

* **Ordina per colonna**: fare clic su qualsiasi intestazione di colonna per ordinare le immagini
* **Ordinamento per data**: utile per organizzare sequenze di acquisizione cronologiche
* **Filtro modello fotocamera**: raggruppa le immagini per tipo di fotocamera se si utilizzano più fotocamere

***

## Anteprima immagine

### Visualizzazione dell&#x27;immagine completa

Fare clic su una miniatura qualsiasi nel File Browser per visualizzarla nell&#x27;area di anteprima principale:

1. L&#x27;immagine viene visualizzata nel pannello di anteprima centrale
2. Utilizzare i controlli di zoom per esaminare i dettagli dell&#x27;immagine
3. Passa da un&#x27;immagine all&#x27;altra utilizzando i tasti freccia

### Navigazione rapida

* **Immagine precedente**: clicca sulla freccia sinistra o premi il tasto ←
* **Immagine successiva**: clicca sulla freccia destra o premi il tasto →
* **Zoom avanti/indietro**: utilizza la rotellina del mouse o i pulsanti di zoom
* **Panoramica**: clicca e trascina sull&#x27;immagine quando è ingrandita

***

## Gestione dei file duplicati

Chloros rileva e ignora automaticamente i file duplicati:

* I file con nomi identici vengono ignorati
* Impedisce la doppia elaborazione accidentale
* Viene visualizzato un messaggio di avviso quando vengono rilevati duplicati

{% hint style=&quot;warning&quot; %}
**Importante**: non rinominare o modificare i file immagine originali prima dell&#x27;importazione. Chloros si basa sui nomi dei file originali e sui metadati per una corretta elaborazione.
{% endhint %}

***

## Set di dati misti delle telecamere

Se il progetto contiene immagini provenienti da più telecamere MAPIR:

1. Chloros rileva automaticamente ogni modello di telecamera
2. Ogni tipo di telecamera viene elaborato con il profilo di calibrazione appropriato
3. Il File Browser visualizza il modello della telecamera nella colonna Camera Model (Modello telecamera)
4. L&#x27;elaborazione applica le impostazioni corrette per ogni tipo di telecamera

**Esempio di scenario**: Survey3W RGN + Survey3N OCN configurazione a doppia fotocamera

***

## Best practice

### Organizza prima dell&#x27;importazione

* Conservare le immagini dei target di calibrazione nella stessa cartella delle immagini del rilievo
* Mantenere la struttura originale delle cartelle della fotocamera/scheda SD
* Non mescolare set di dati provenienti da sessioni diverse in un unico progetto

### Denominazione dei file

* Conservare i nomi originali dei file della fotocamera (IMG\_0001.RAW, ecc.)
* Non rinominare i file prima dell&#x27;importazione
* I nomi originali contengono metadati importanti

### Immagini target di calibrazione

* Includere sempre 1-2 immagini target di calibrazione per sessione
* Acquisire i target prima e dopo la sessione di acquisizione
* Posizionare i target nelle stesse condizioni di illuminazione dell&#x27;area di acquisizione
* Contrassegnare le immagini target utilizzando la casella di controllo Target per velocizzare l&#x27;elaborazione

***

## Problemi comuni e soluzioni

### Immagini non visualizzate dopo l&#x27;importazione

**Possibili cause:**

* Formato file non supportato (solo RAW+JPG e JPG dalle fotocamere MAPIR)
* Le immagini provengono da fotocamere non MAPIR (vedere [Fotocamere supportate](../supported-cameras.md))
* File danneggiato o trasferimento incompleto dalla scheda SD

**Soluzione**: verificare la compatibilità del formato file e del modello di fotocamera

### Modello di fotocamera non rilevato

**Possibili cause:**

* Metadati EXIF modificati
* Immagini modificate con software esterno
* Trasferimento file incompleto

**Soluzione**: reimportare i file originali non modificati dalla fotocamera/scheda SD

### Timestamp mancanti

**Possibili cause:**

* Orologio della fotocamera non impostato correttamente
* Dati EXIF rimossi da software esterno

**Soluzione**: verificare che le impostazioni dell&#x27;ora della fotocamera fossero corrette durante l&#x27;acquisizione

***

## Passaggi successivi

Una volta importati i file:

1. **Rivedere l&#x27;elenco dei file** - Assicurarsi che tutte le immagini siano state caricate correttamente
2. **Controllare i modelli di fotocamera** - Verificare il corretto rilevamento della fotocamera
3. **Contrassegnare le immagini di destinazione** - Vedere [Scelta delle immagini di destinazione](choosing-target-images.md)
4. **Regolare le impostazioni** - Configurare le opzioni di elaborazione in [Impostazioni progetto](adjusting-project-settings.md)
5. **Avvia l&#x27;elaborazione** - Vedi [Avvio dell&#x27;elaborazione](starting-the-processing.md)

Per informazioni dettagliate sulla configurazione del progetto, vedi [Modifica delle impostazioni del progetto](adjusting-project-settings.md).
