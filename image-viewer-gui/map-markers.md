# Indicatori sulla mappa

La scheda Mappa visualizza le immagini su una mappa 2D interattiva in base alle loro coordinate GPS. Ciò fornisce una panoramica geografica della sessione di acquisizione e aiuta a visualizzare la copertura spaziale. È utile anche quando si importano le immagini per la prima volta, per rimuovere rapidamente quelle che non è necessario elaborare.

## Accesso alla scheda Mappa

1. Aprire o creare un progetto in Chloros.
2. Importare le immagini che contengono metadati GPS.
3. Fare clic sulla scheda **Mappa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> nella barra laterale sinistra
4. La mappa mostrerà dei marcatori in corrispondenza della posizione GPS di ciascuna immagine

{% suggerimento style=&quot;info&quot; %}
**GPS richiesto**: solo le immagini con coordinate GPS incorporate nei metadati EXIF appariranno sulla mappa. Assicurarsi che la fotocamera abbia il GPS abilitato durante la cattura.
{% endhint %}

***

## Regolazione delle immagini dalla scheda Mappa

La scheda **Mappa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> ha le stesse funzioni di aggiunta  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  e rimuovi  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  file della scheda [**File Browser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Mostra anche lo stesso elenco di file di progetto, ma con intestazioni di colonna diverse:

### Nome file

* Nome file originale dalla fotocamera
* Mantiene la convenzione di denominazione della fotocamera (ad es. IMG\_0001.RAW)

### Latitudine

* Latitudine dell&#x27;immagine

### Longitudine

* Longitudine dell&#x27;immagine

### Altitudine

* Altitudine dell&#x27;immagine

{% hint style=&quot;info&quot; %}
Cliccando sulle intestazioni delle colonne della tabella si ordinano anche i dati delle righe
{% endhint %}

***

## Indicatori delle immagini

Ogni immagine con dati GPS è rappresentata da un indicatore sulla mappa:

### Visualizzazione degli indicatori

* Gli indicatori indicano le coordinate GPS esatte in cui è stata scattata ciascuna immagine
* Gli indicatori raggruppati possono essere raggruppati quando si riduce lo zoom
* Ingrandisci per vedere le posizioni delle singole immagini

{% suggerimento style=&quot;success&quot; %}
SUPER-ZOOM: quando si raggiunge il livello di zoom massimo dal fornitore di mappe, la mappa viene ingrandita ulteriormente, consentendo di vedere i marcatori vicini tra loro.
{% endhint %}

### Anteprima al passaggio del mouse

* **Passa il mouse** su qualsiasi indicatore per visualizzare un&#x27;anteprima in miniatura dell&#x27;immagine
* Ciò consente una rapida identificazione visiva senza uscire dalla visualizzazione della mappa
* Utile per individuare immagini specifiche all&#x27;interno di una sessione di acquisizione di grandi dimensioni

***

## Fornitori di mappe

{% hint style=&quot;success&quot; %}
**Selezione automatica**: Chloros sceglie automaticamente il servizio di tessere che fornisce il miglior livello di zoom per la posizione corrente sulla mappa. Se lo desideri, puoi passare manualmente da un fornitore all&#x27;altro.
{% endhint %}

La scheda Mappa supporta due fornitori di tessere per le immagini di sfondo della mappa:

### Google Maps

* Immagini satellitari e cartografiche standard di Google.
* Ideale per una copertura generale a livello mondiale.

### ESRI

* Immagini satellitari e aeree di ESRI ArcGIS.
* Spesso fornisce immagini ad alta risoluzione in determinate regioni.

***

## Tipi di tessere della mappa

È possibile scegliere il tipo di livello della mappa (da sinistra a destra):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terreno

Mostra i profili altimetrici e le tessere della mappa con dettagli (strade, ecc.)

### Mappa

Mostra tessere della mappa standard (banda larga inferiore) con dettagli (strade, ecc.)

### Satellite

Mostra tessere della mappa satellitare dettagliate (banda larga superiore)

### Ibrido

Mostra tessere della mappa satellitare con dettagli aggiuntivi (strade, ecc.)

***

## Navigazione della mappa

### Controlli di zoom

* **Zoom avanti/indietro**: utilizzare la rotellina del mouse o i pulsanti di zoom
* **Schermo intero**: visualizza la mappa a schermo intero

### Controlli di panoramica

* **Panoramica**: fare clic e trascinare per spostarsi sulla mappa***

## Casi d&#x27;uso

### Visualizzazione della traiettoria di volo

* Visualizza l&#x27;area di copertura delle sessioni di acquisizione con drone
* Identifica le lacune nella copertura delle immagini
* Verifica l&#x27;esecuzione della traiettoria di volo

### Revisione del rilevamento a terra

* Visualizza la distribuzione spaziale delle acquisizioni a terra
* Individua le immagini di calibrazione relative all&#x27;area di rilevamento
* Pianifica ulteriori posizioni di acquisizione

### Controllo qualità

* Identifica rapidamente le immagini acquisite in posizioni inaspettate
* Verifica la precisione del GPS in tutto il set di dati
* Incrocia le posizioni delle immagini con le note sul campo

***

## Risoluzione dei problemi

### Nessun indicatore visualizzato

**Possibili cause:**

* Le immagini non contengono metadati GPS
* Il GPS era disattivato sulla fotocamera durante l&#x27;acquisizione
* I dati EXIF sono stati rimossi da un software esterno

**Soluzione**: verificare che il GPS sia abilitato sulla fotocamera e reimportare i file originali

### Indicatori in posizione errata

**Possibili cause:**

* Il GPS della fotocamera aveva una scarsa ricezione satellitare
* Deriva del GPS durante l&#x27;acquisizione

**Soluzione**: si tratta in genere di un problema legato al momento dell&#x27;acquisizione; valutare l&#x27;utilizzo del GPS PPK/RTK per applicazioni di precisione
