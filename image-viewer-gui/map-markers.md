# Indicatori sulla mappa

La scheda Mappa visualizza le immagini su una mappa 2D interattiva in base alle loro coordinate GPS. Ciò fornisce una panoramica geografica della sessione di acquisizione e aiuta a visualizzare la copertura spaziale. È utile anche al momento della prima importazione delle immagini per eliminare rapidamente quelle che non è necessario elaborare.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Come accedere alla scheda Mappa

1. Apri o crea un progetto in Chloros
2. Importa le immagini che contengono metadati GPS
3. Clicca sulla scheda **Mappa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> nella barra laterale sinistra
4. La mappa mostrerà degli indicatori in corrispondenza della posizione GPS di ciascuna immagine

{% hint style="info" %}
**GPS richiesto**: sulla mappa appariranno solo le immagini con coordinate GPS incorporate nei metadati EXIF. Assicurati che la tua fotocamera abbia il GPS abilitato durante l&#x27;acquisizione.
{% endhint %}

***

## Regolazione delle immagini dalla scheda Mappa

La scheda **Mappa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> ha lo stesso pulsante di aggiunta  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  e rimuovi  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  come la scheda [**File Browser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Mostra inoltre lo stesso elenco di file di progetto, ma con intestazioni di colonna diverse:

### Nome file

* Nome file originale della fotocamera
* Mantiene la convenzione di denominazione della fotocamera (ad es. IMG\_0001.RAW)

### Latitudine

* Latitudine dell&#x27;immagine

### Longitudine

* Longitudine dell&#x27;immagine

### Altitudine

* L&#x27;altitudine dell&#x27;immagine

{% hint style="info" %}
Cliccando sulle intestazioni delle colonne della tabella si ordinano anche i dati delle righe
{% endhint %}

***

## Indicatori delle immagini

Ogni immagine con dati GPS è rappresentata da un indicatore sulla mappa:

### Visualizzazione degli indicatori

* I marcatori indicano le coordinate GPS esatte in cui è stata scattata ciascuna immagine
* I marcatori raggruppati possono unirsi quando si riduce lo zoom
* Ingrandisci per vedere le posizioni delle singole immagini

{% hint style="success" %}
SUPER-ZOOM: Quando raggiungi il livello massimo di zoom dal fornitore di tessere della mappa, la tessera viene ingrandita ulteriormente, permettendoti di vedere i marcatori che sono vicini tra loro.
{% endhint %}

### Anteprima al passaggio del mouse

* **Passa il mouse** su qualsiasi indicatore per visualizzare un&#x27;anteprima in miniatura di quell&#x27;immagine
* Ciò consente una rapida identificazione visiva senza uscire dalla vista della mappa
* Utile per individuare immagini specifiche all&#x27;interno di una sessione di acquisizione di grandi dimensioni

***

## Fornitori di tessere cartografiche

{% hint style="success" %}
**Selezione automatica**: Chloros sceglie automaticamente il servizio di tessere che fornisce il miglior livello di zoom per la posizione corrente sulla mappa. Se lo desideri, puoi passare manualmente da un fornitore all&#x27;altro.
{% endhint %}

La scheda Mappa supporta due fornitori di tessere per le immagini di sfondo della mappa:

### Google Maps

* Immagini satellitari e cartografiche standard di Google
* Ideale per una copertura globale generale

### ESRI

* Immagini satellitari e aeree di ESRI ArcGIS
* Spesso fornisce immagini a risoluzione più elevata in determinate regioni

***

## Tipi di tessere cartografiche

È possibile scegliere il tipo di livello cartografico (da sinistra a destra):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terreno

Mostra i profili altimetrici e le tessere della mappa con i dettagli (strade, ecc.)

### Mappa

Mostra tessere della mappa standard (a larghezza di banda ridotta) con i dettagli (strade, ecc.)

### Satellite

Mostra tessere della mappa satellitare dettagliate (a larghezza di banda maggiore)

### Ibrido

Mostra tessere della mappa satellitare con dettagli aggiuntivi (strade, ecc.)

***

## Navigazione sulla mappa

### Controlli di zoom

* **Zoom avanti/indietro**: utilizzare la rotellina del mouse o i pulsanti di zoom
* **Schermo intero**: visualizza la mappa a schermo intero

### Controlli di panoramica

* **Panoramica**: cliccare e trascinare per spostarsi sulla mappa***

## Casi d&#x27;uso

### Visualizzazione della traiettoria di volo

* Visualizza l&#x27;area di copertura delle sessioni di acquisizione con drone
* Identifica le lacune nella copertura delle immagini
* Verifica l&#x27;esecuzione della traiettoria di volo

### Revisione del rilevamento a terra

* Visualizza la distribuzione spaziale delle acquisizioni a terra
* Individua le immagini dei target di calibrazione rispetto all&#x27;area di rilevamento
* Pianifica ulteriori posizioni di acquisizione

### Controllo qualità

* Identifica rapidamente le immagini acquisite in posizioni inaspettate
* Verifica la precisione del GPS nell&#x27;intero set di dati
* Incrocia le posizioni delle immagini con le note sul campo

***

## Risoluzione dei problemi

### Nessun indicatore visualizzato

**Possibili cause:**

* Le immagini non contengono metadati GPS
* Il GPS era disabilitato sulla fotocamera durante l&#x27;acquisizione
* I dati EXIF sono stati rimossi da un software esterno

**Soluzione**: verificare che il GPS sia abilitato sulla fotocamera e reimportare i file originali

### Indicatori in posizione errata

**Possibili cause:**

* Il GPS della fotocamera aveva una scarsa ricezione satellitare
* Deriva del GPS durante l&#x27;acquisizione

**Soluzione**: si tratta in genere di un problema legato al momento dell&#x27;acquisizione; considerare l&#x27;utilizzo di GPS PPK/RTK per applicazioni di precisione
