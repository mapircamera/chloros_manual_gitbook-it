# Aprire un&#x27;immagine a schermo intero

Il visualizzatore di immagini Chloros offre un&#x27;interfaccia dedicata a schermo intero per la visualizzazione, l&#x27;analisi e la manipolazione delle immagini multispettrali. Sia che si tratti di immagini originali o di risultati elaborati, il visualizzatore di immagini offre potenti strumenti per l&#x27;ispezione e l&#x27;analisi.

## Accesso al visualizzatore di immagini

### Dal File Browser

Il modo più comune per aprire un&#x27;immagine nel visualizzatore di immagini:

1. Assicurarsi di trovarsi nella scheda **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Fare clic su una qualsiasi **miniatura** nella griglia delle immagini
3. L&#x27;immagine si apre nell&#x27;**area di anteprima principale** (al centro dello schermo)
4. L&#x27;immagine è ora caricata e pronta per la visualizzazione a schermo intero

### Apertura della scheda Image Viewer

Una volta caricata un&#x27;immagine nell&#x27;area di anteprima:

1. Fare clic sull&#x27;icona **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> nella barra laterale sinistra
2. Si apre la scheda Visualizzatore immagini, che mostra l&#x27;immagine selezionata a schermo intero
3. Nella barra laterale sinistra diventano disponibili strumenti avanzati di visualizzazione e analisi

***

## Panoramica dell&#x27;interfaccia del Visualizzatore immagini

### Area di visualizzazione principale

La parte più ampia dello schermo mostra l&#x27;immagine:

* **Risoluzione completa**: immagini visualizzate alla risoluzione nativa
* **Zoomabile**: utilizzare i controlli o la rotellina del mouse per lo zoom
* **Panoramica**: cliccare e trascinare per spostarsi quando si è in zoom
* **Proporzioni mantenute**: le immagini vengono ridimensionate in modo proporzionale***

## Opzioni di visualizzazione

### Navigazione di base delle immagini

#### Sfogliare le immagini

Navigare attraverso il set di immagini utilizzando le scorciatoie da tastiera o i pulsanti:

* **Immagine successiva**: clicca sul pulsante → o premi il tasto**→** (freccia destra)
* **Immagine precedente**: clicca sul pulsante ← o premi il tasto**←** (freccia sinistra)
* **Vai a un&#x27;immagine specifica**: torna al File Browser e clicca sulla miniatura desiderata

#### Controlli di zoom

Regola l&#x27;ingrandimento per esaminare i dettagli dell&#x27;immagine:

**Ingrandisci:*** Clicca sul pulsante **+** (più)
* Premi il tasto **+**o**=*** Scorri la rotellina del mouse **verso l&#x27;alto**

**Riduci:*** Clicca sul pulsante **−** (meno)
* Premi il tasto **−** (meno)
* Scorri la rotellina del mouse **verso il basso**

#### Panoramica con lo zoom

Quando lo zoom supera le dimensioni dello schermo:

1. Spostare il cursore del mouse sull&#x27;immagine
2. Fare clic e **tenere premuto il tasto sinistro del mouse**

3.**Trascinare** per spostare l&#x27;immagine
4. Rilasciare per interrompere la panoramica

**Alternativa**: utilizzare i tasti freccia per eseguire una panoramica a piccoli incrementi***

## Ispezione dei valori dei pixel

### Visualizzazione dei valori dei pixel sul cursore

Mentre si sposta il cursore del mouse sull&#x27;immagine, i valori dei pixel vengono visualizzati in tempo reale:**Posizione di visualizzazione dei valori:*** **Numero mobile e linea rossa nella legenda del gradiente LUT dell&#x27;indice sul lato destro*** **Quando si ingrandisce ulteriormente, valore mobile vicino al cursore e al pixel evidenziato*** Mostra i valori per il pixel **sotto il cursore o evidenziato*** Si aggiorna mentre si sposta il mouse

***

## Tipi di immagini visualizzabili

### JPG

**Immagini JPG dalla fotocamera:**

* Visualizza i dati JPG come in anteprima
* Mostra i valori originali, non corretti
* Utile per verificare la qualità dell&#x27;immagine prima dell&#x27;elaborazione

### RAW (Originale)

### RAW (Riflettanza)

**Dopo l&#x27;elaborazione:**

* Vignettatura corretta
* Riflettanza calibrata
* Multibanda TIFF (Red, Green, NIR, ecc.)
* Dati scientifici pronti per l&#x27;analisi

### RAW (Indice)

**NDVI, NDRE, GNDVI, ecc. (file \_NDVI.tif):**

* Immagini in scala di grigi a banda singola
* I valori dei pixel rappresentano i risultati del calcolo dell&#x27;indice
* Intervallo tipicamente compreso tra -1 e +1 per gli indici normalizzati
* È possibile applicare LUT di colore per la visualizzazione

***

## Applicazione di indici e LUT

Applicare indici multispettrali e tabelle di ricerca (LUT) dei colori:

1. Individuare **Index/LUT Sandbox**in**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> barra laterale
2. Selezionare l&#x27;indice di vegetazione (NDVI, NDRE, ecc.)
3. Selezionare una formula multispettrale o crearne una personalizzata (solo Chloros+)
4. Applicare un gradiente LUT a colori per la visualizzazione
5. Regolare gli intervalli di valori e le soglie

Vedere [Index/LUT Sandbox](index-lut-sandbox.md) per istruzioni dettagliate.

***

## Scorciatoie da tastiera

### Navigazione

* **→** (Freccia destra): Immagine successiva
* **←** (Freccia sinistra): Immagine precedente
* **Home**: Prima immagine dell&#x27;elenco
* **Fine**: Ultima immagine nell&#x27;elenco

### Zoom

* **+**o**=**: Ingrandisci
* **−**: Riduci
* **Rotellina del mouse**: Ingrandisci/riduci***

### Verifica dei calcoli degli indici

Verifica che gli indici siano stati calcolati correttamente:

1. Aprire NDVI o un&#x27;altra immagine dell&#x27;indice
2. Controllare le aree di vegetazione:
   * **NDVI**: dovrebbe mostrare un valore compreso tra 0,4 e 0,9 per le piante sane
   * **NDRE**: valori più elevati per una crescita vigorosa
   * **GNDVI**: Simile a NDVI ma sensibile alla clorofilla
3. Controllare le aree non vegetate:
   * **Suolo**: Vicino a 0 o leggermente negativo
   * **Acqua**: Valori negativi (da -0,5 a 0)***

## Risoluzione dei problemi di visualizzazione

### L&#x27;immagine non si apre

**Possibili cause:**

* File danneggiato durante l&#x27;elaborazione
* Formato file non supportato
* Memoria insufficiente per immagini di grandi dimensioni

**Soluzioni:**

1. Provare ad aprire il file in un visualizzatore esterno per verificarne l&#x27;integrità
2. Verificare che il formato del file corrisponda al tipo previsto
3. Chiudere le altre applicazioni per liberare memoria
4. Provare con un&#x27;immagine più piccola o diversa

### Visualizzazione dell&#x27;immagine in bianco o nero

**Possibili cause:**

* Intervallo di valori al di fuori della capacità di visualizzazione
* Immagine a 32 bit in virgola mobile con valori insoliti
* Errore di calcolo dell&#x27;indice

**Soluzioni:**

1. Controlla i valori dei pixel: se sono tutti molto bassi o molto alti, regola l&#x27;intervallo di visualizzazione
2. Prova ad aprire il file in QGIS o in un programma simile con regolazione automatica dell&#x27;intervallo
3. Controlla il registro di debug dell&#x27;elaborazione per individuare eventuali errori

### I valori dei pixel sembrano errati

**Possibili cause:**

* Visualizzazione dell&#x27;immagine sbagliata (originale vs elaborata)
* La calibrazione non è stata applicata correttamente
* I dati del sensore di luce non sono stati inclusi nell&#x27;input
* La modalità percentuale è stata attivata in modo errato

**Soluzioni:**

1. Verifica di stare visualizzando l&#x27;output elaborato (controlla il suffisso del nome del file)
2. Controlla lo stato del pulsante della modalità percentuale
3. Confronta con immagini note e corrette dello stesso dataset

***

## Passi successivi

Ora che è possibile visualizzare le immagini a schermo intero:

* [**Livelli immagine**](image-layers.md) - Informazioni sulla visualizzazione multibanda
* [**Sandbox indici/LUT**](index-lut-sandbox.md) - Applicare indici personalizzati e mappatura dei colori
* [**Formule degli indici multispettrali**](../project-settings/multispectral-index-formulas.md) - Comprendi gli indici disponibili

Per il flusso di lavoro di elaborazione, consulta:

* [**Elaborazione delle immagini (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Guida completa all&#x27;elaborazione
