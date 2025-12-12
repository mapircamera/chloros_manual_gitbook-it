# Apertura di un&#x27;immagine a schermo intero

Il visualizzatore di immagini Chloros offre un&#x27;interfaccia dedicata a schermo intero per la visualizzazione, l&#x27;analisi e la manipolazione delle immagini multispettrali. Sia che si tratti di immagini originali o di risultati elaborati, il visualizzatore di immagini offre potenti strumenti per l&#x27;ispezione e l&#x27;analisi.

## Accesso all&#x27;Image Viewer

### Dal File Browser

Il modo più comune per aprire un&#x27;immagine nell&#x27;Image Viewer:

1. Assicurarsi di essere nella scheda **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Fare clic su una qualsiasi delle **miniature delle immagini** nella griglia delle immagini
3. L&#x27;immagine si apre nell&#x27;**area di anteprima principale** (al centro dello schermo)
4. L&#x27;immagine è ora caricata e pronta per la visualizzazione a schermo intero

### Apertura della scheda Visualizzatore immagini

Una volta caricata un&#x27;immagine nell&#x27;area di anteprima:

1. Fare clic sull&#x27;icona **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> nella barra laterale sinistra
2. Si aprirà la scheda Visualizzatore immagini, che mostrerà l&#x27;immagine selezionata a schermo intero
3. Nella barra laterale sinistra saranno disponibili strumenti avanzati di visualizzazione e analisi

***

## Panoramica dell&#x27;interfaccia del Visualizzatore immagini

### Area di visualizzazione principale

La parte più grande dello schermo mostra l&#x27;immagine:

* **Risoluzione completa**: immagini visualizzate alla risoluzione nativa
* **Zoomabile**: utilizzare i controlli o la rotellina del mouse per ingrandire
* **Panoramica**: fare clic e trascinare per spostarsi quando si ingrandisce
* **Proporzioni mantenute**: le immagini vengono ridimensionate in modo proporzionale

***

## Opzioni di visualizzazione

### Navigazione di base delle immagini

#### Sfogliare le immagini

Navigare nella serie di immagini utilizzando le scorciatoie da tastiera o i pulsanti:

* **Immagine successiva**: clicca sul pulsante → o premi il tasto **→** (freccia destra)
* **Immagine precedente**: clicca sul pulsante ← o premi il tasto **←** (freccia sinistra)
* **Passa a un&#x27;immagine specifica**: torna al File Browser e clicca sulla miniatura desiderata

#### Controlli di zoom

Regola l&#x27;ingrandimento per esaminare i dettagli dell&#x27;immagine:

**Ingrandisci:**

* Fai clic sul pulsante **+** (più)
* Premi il tasto **+** o **=**
* Scorri la rotellina del mouse **verso l&#x27;alto**

**Riduci:**

* Fai clic sul pulsante **−** (meno)
* Premi il tasto **−** (meno)
* Scorri la rotellina del mouse **verso il basso**

**Adatta allo schermo:**

* Fare clic sul pulsante **↔** (Adatta)
* Premere il tasto **0** (Zero)
* Fare doppio clic sull&#x27;immagine

#### Panoramica durante lo zoom

Quando lo zoom supera le dimensioni dello schermo:

1. Spostare il cursore del mouse sull&#x27;immagine
2. Fare clic e **tenere premuto il tasto sinistro del mouse**
3. **Trascinare** per spostare l&#x27;immagine
4. Rilasciare per interrompere lo spostamento

**Alternativa**: utilizzare i tasti freccia per spostarsi con piccoli incrementi

***

## Ispezione del valore dei pixel

### Visualizzazione dei valori dei pixel al cursore

Quando si sposta il cursore del mouse sull&#x27;immagine, i valori dei pixel vengono visualizzati in tempo reale:

**Posizione di visualizzazione del valore:**

* **Numero fluttuante e linea rossa nella legenda del gradiente LUT dell&#x27;indice sul lato destro**
* **Quando si ingrandisce ulteriormente, valore fluttuante vicino al cursore e pixel evidenziato**
* Mostra i valori dei pixel **sotto il cursore o evidenziati**
* Si aggiorna mentre si sposta il mouse

***

## Tipi di immagini che è possibile visualizzare

### Immagini originali (pre-elaborazione)

**Immagini RAW + JPG dalla fotocamera:**

* Visualizza i dati RAW come anteprima
* Mostra i valori originali non corretti
* Utile per controllare la qualità dell&#x27;immagine prima dell&#x27;elaborazione

### Immagini di riflettanza calibrate

**Dopo l&#x27;elaborazione:**

* Vignettatura corretta
* Riflettanza calibrata
* Multibanda TIFF (Red, Green, NIR, ecc.)
* Dati scientifici pronti per l&#x27;analisi

### Immagini indice

**NDVI, NDRE, GNDVI, ecc. (file \_NDVI.tif):**

* Immagini in scala di grigi a banda singola
* I valori dei pixel rappresentano i risultati del calcolo dell&#x27;indice
* Intervallo tipicamente compreso tra -1 e +1 per gli indici normalizzati
* È possibile applicare LUT di colore per la visualizzazione

***

## Applicazione di indici e LUT

Applicare indici multispettrali e tabelle di ricerca dei colori:

1. Individuare **Index/LUT Sandbox** nella <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> barra laterale
2. Selezionare l&#x27;indice di vegetazione (NDVI, NDRE, ecc.)
3. Selezionare la formula multispettrale o crearne una personalizzata (solo Chloros+)
4. Applicare il gradiente LUT a colori per la visualizzazione
5. Regolare gli intervalli di valori e le soglie

Per istruzioni dettagliate, consultare [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Scorciatoie da tastiera

### Navigazione

* **→** (freccia destra): immagine successiva
* **←** (freccia sinistra): immagine precedente
* **Home**: prima immagine dell&#x27;elenco
* **Fine**: Ultima immagine nell&#x27;elenco

### Zoom

* **+** o **=**: Ingrandisci
* **−**: Riduci
* **0** (Zero): Adatta allo schermo
* **Rotellina del mouse**: Ingrandisci/Riduci

### Controlli di visualizzazione

* **P**: Attiva/disattiva la modalità percentuale dei pixel
* **L**: Attiva/disattiva il pannello dei livelli
* **Esc**: Chiude la modalità a schermo intero o torna al File Browser

### Altro

* **Ctrl+S**: Salva l&#x27;immagine corrente
* **F**: Modalità a schermo intero (se disponibile)

***

### Verifica dei calcoli dell&#x27;indice

Verificare che gli indici siano stati calcolati correttamente:

1. Aprire NDVI o un&#x27;altra immagine dell&#x27;indice
2. Controllare le aree di vegetazione:
   * **NDVI**: dovrebbe mostrare 0,4-0,9 per le piante sane
   * **NDRE**: valori più alti per una crescita vigorosa
   * **GNDVI**: simile a NDVI ma sensibile alla clorofilla
3. Controllare le aree non vegetate:
   * **Suolo**: vicino a 0 o leggermente negativo
   * **Acqua**: valori negativi (da -0,5 a 0)

***

## Risoluzione dei problemi di visualizzazione

### L&#x27;immagine non si apre

**Possibili cause:**

* File danneggiato durante l&#x27;elaborazione
* Formato file non supportato
* Memoria insufficiente per immagini di grandi dimensioni

**Soluzioni:**

1. Provare ad aprire il file in un visualizzatore esterno per verificare l&#x27;integrità del file
2. Verificare che il formato del file corrisponda al tipo previsto
3. Chiudere le altre applicazioni per liberare memoria
4. Provare con un&#x27;immagine più piccola/diversa

### Visualizzazione dell&#x27;immagine in bianco e nero

**Possibili cause:**

* Intervallo di valori al di fuori delle capacità di visualizzazione
* Immagine a 32 bit con valori insoliti
* Errore di calcolo dell&#x27;indice

**Soluzioni:**

1. Controllare i valori dei pixel: se sono tutti molto bassi o molto alti, regolare l&#x27;intervallo di visualizzazione
2. Provare ad aprire in QGIS o simili con regolazione automatica dell&#x27;intervallo
3. Controllare il log di debug dall&#x27;elaborazione per eventuali errori

### I valori dei pixel sembrano errati

**Possibili cause:**

* Visualizzazione di un&#x27;immagine errata (originale vs elaborata)
* La calibrazione non è stata applicata correttamente
* I dati del sensore di luce non sono stati inclusi nell&#x27;input
* La modalità percentuale è stata attivata in modo errato

**Soluzioni:**

1. Verificare di stare visualizzando l&#x27;output elaborato (controllare il suffisso del nome del file)
2. Controllare lo stato del pulsante della modalità percentuale
3. Confrontare con immagini note come corrette provenienti dallo stesso set di dati

***

## Passaggi successivi

Ora che è possibile visualizzare le immagini a schermo intero:

* [**Livelli immagine**](image-layers.md) - Informazioni sulla visualizzazione multibanda
* [**Sandbox indice/LUT**](index-lut-sandbox.md) - Applicazione di indici personalizzati e mappatura dei colori
* [**Formule indice multispettrale**](../project-settings/multispectral-index-formulas.md) - Comprendi gli indici disponibili

Per il flusso di lavoro di elaborazione, vedi:

* [**Elaborazione delle immagini (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Guida completa all&#x27;elaborazione
