# Indicatori sulla mappa

La scheda &quot;Mappa&quot; traccia le immagini su una mappa 2D interattiva in base alle loro coordinate GPS. Offre una panoramica geografica di una sessione di acquisizione ed è il modo più rapido, subito dopo l&#x27;importazione, per eliminare le immagini che non si desidera elaborare.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Come accedere alla scheda &quot;Mappa&quot;

1. Apri o crea un progetto in Chloros
2. Importa le immagini che contengono metadati GPS
3. Fai clic sulla scheda **Mappa** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> nella barra laterale sinistra
4. La mappa visualizza un indicatore in corrispondenza della posizione GPS di ciascuna immagine

{% hint style="info" %}
**GPS richiesto**: sulla mappa compaiono solo le immagini con coordinate GPS nei metadati EXIF. Un’immagine priva di coordinate rimane comunque nel progetto e viene elaborata normalmente; semplicemente non presenta alcun indicatore.
{% endhint %}

***

## Modifica delle immagini dalla scheda &quot;Mappa&quot;

La scheda **Mappa**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> presenta gli stessi pulsanti per l’aggiunta <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> e la rimozione <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> dei file presenti nella scheda [**Esplora file**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Mostra lo stesso elenco di file di progetto, con colonne geografiche:

| Colonna        | Contenuto                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Nome**      | Il nome del file così come è stato salvato dalla fotocamera                             |
| **Latitudine**  | Gradi decimali, sei cifre decimali                                |
| **Longitudine** | Gradi decimali, sei cifre decimali                                |
| **Altitudine**  | Metri, una cifra decimale — `-` quando l’immagine non riporta l’altitudine |

{% hint style="info" %}
Clicca su qualsiasi intestazione di colonna per ordinare i dati in base a quella; clicca di nuovo per invertire l&#x27;ordine.
{% endhint %}

{% hint style="warning" %}
**L’altitudine è l’altezza sul livello del mare, non l’altezza dal suolo.** Il valore proviene dal tag EXIF `GPSAltitude` dell’immagine, che fa riferimento al livello medio del mare. Non si tratta dell&#x27;altezza di volo rispetto al terreno, e Chloros non ne ricaverà la distanza campionaria al suolo (GSD): su un campo a 300 m sul livello del mare, un drone a 100 m AGL registrerà qui circa 400 m. Utilizza questa colonna per individuare valori anomali e confermare la coerenza dell’altitudine di volo, non come misura AGL.
{% endhint %}

***

## Indicatori delle immagini

Ogni immagine con dati GPS riceve un indicatore alle proprie coordinate.

### Visualizzazione dei marcatori

* I marcatori si trovano alle coordinate esatte registrate per ogni acquisizione
* I marcatori vicini tra loro possono sovrapporsi visivamente quando si riduce lo zoom: ingrandisci l’immagine per distinguerli
* I marcatori selezionati ed evidenziati vengono visualizzati sopra gli altri

### Anteprima al passaggio del mouse

* **Passa il mouse** su un indicatore per visualizzare una miniatura dell’immagine corrispondente con il nome del file
* **Fai clic**su un indicatore per selezionare l’immagine e**fissare** la finestra a comparsa aperta: rimarrà visibile finché non farai clic altrove. Mentre la finestra a comparsa è fissata, passando con il mouse su altri indicatori non verrà nascosta
* Questo è il modo più veloce per trovare un fotogramma specifico in una sessione di grandi dimensioni senza uscire dalla mappa

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>La scheda «Mappa» riporta tutte le immagini geotaggate presenti nel progetto</p></figcaption></figure>### Super-zoom

{% hint style="success" %}
**SUPER-ZOOM**: quando raggiungi lo zoom massimo per cui il fornitore di tessere dispone di immagini, un ulteriore ingrandimento aumenta le dimensioni delle tessere invece di fermarsi, così puoi distinguere i marcatori che si trovano quasi uno sopra l’altro.
{% endhint %}

* Il super-zoom si attiva solo quando ci si trova **al** livello massimo di zoom del fornitore per quella posizione e il caricamento delle tessere è terminato. Al di sotto di tale livello, lo zoom funziona normalmente
* L’intervallo va da **1× a 32×** oltre il massimo del fornitore stesso
* Un indicatore nell’angolo mostra il super-zoom corrente in percentuale, mentre un pulsante **×** accanto ad esso riporta allo zoom normale con un solo clic
* Lo zoom indietro passa sempre attraverso la mappa stessa, quindi non è mai possibile rimanere bloccati nel super-zoom
* Lo zoom e la panoramica mentre si è in super-zoom trasferiscono lo spostamento risultante alla mappa, in modo che l’area decentrata in cui ci si è spostati continui a richiedere le tessere invece di rimanere vuota
* I marcatori sono disegnati come elementi vettoriali anziché rasterizzati, quindi rimangono nitidi a ogni livello di super-zoom

***

## Fornitori di tessere cartografiche

{% hint style="success" %}
**Selezione automatica**: Chloros sceglie il servizio di tessere che offre il miglior livello di zoom per la posizione delle immagini. È possibile cambiare manualmente in qualsiasi momento.
{% endhint %}

| Fornitore        | Note                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Ampia copertura mondiale; supporta tutti e quattro i tipi di tile                                                                                                            |
| **Esri ArcGIS**| Immagini aeree spesso a risoluzione più elevata in determinate regioni. Il tipo di tile**Terrain** non è disponibile per Esri e il relativo pulsante è disabilitato quando è selezionato Esri |***

## Tipi di tessere cartografiche

Scegli il tipo di livello cartografico utilizzando i pulsanti (da sinistra a destra):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Tipo                 | Mostra                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Terreno**          | Ombreggiatura altimetrica con dettagli cartografici (strade, etichette). Solo Google       |
| **Mappa**              | Tessere standard della mappa stradale — l’opzione con la larghezza di banda più bassa              |
| **Satellite**        | Immagini satellitari dettagliate, senza etichette — l’opzione che richiede la larghezza di banda più elevata |
| **Ibrido** (predefinito) | Immagini satellitari con strade ed etichette sovrapposte                |

La scheda Mappa si apre su **Ibrido**. La scelta effettuata si applica anche al cambio di provider, laddove quest’ultimo lo supporti.***

## Navigazione sulla mappa

* **Zoom**: rotellina del mouse o pulsanti di zoom sulla mappa
* **Panoramica**: clicca e trascina
* **Schermo intero**: il comando &quot;Schermo intero&quot; espande la mappa a tutta la finestra***

## Casi d&#x27;uso

### Revisione della traiettoria di volo

* Visualizza a colpo d’occhio l’area di copertura di una sessione con il drone
* Individua le lacune dove è stato saltato un passaggio
* Verifica che il volo abbia seguito il percorso pianificato

### Revisione del rilevamento a terra

* Visualizza la distribuzione delle acquisizioni da terra
* Individua i fotogrammi dei target di calibrazione rispetto all’area di rilevamento
* Decidi dove sono necessarie ulteriori acquisizioni

### Controllo qualità

* Individuare le immagini acquisite in punti inaspettati e rimuoverle prima dell’elaborazione
* Ordinare per altitudine per individuare un fotogramma acquisito all’altezza sbagliata o in cui la posizione GPS era imprecisa
* Confrontare le posizioni delle immagini con le note sul campo

***

## Risoluzione dei problemi

### Non vengono visualizzati i marcatori

**Possibili cause**

* Le immagini non contengono metadati GPS
* Il GPS era disattivato sulla fotocamera durante l’acquisizione
* I dati EXIF sono stati rimossi da un altro software prima dell’importazione

**Cosa fare**: verificare che il GPS sia abilitato sulla fotocamera e reimportare i file originali. È possibile verificare se un file specifico possiede coordinate cercandolo nella tabella dei file della scheda Mappa: un’immagine senza coordinate non avrà alcuna riga in quella tabella.

### I marcatori sono nella posizione sbagliata

**Possibili cause**: una cattiva acquisizione del segnale satellitare al momento dell’acquisizione o una deriva del GPS durante la sessione.**Cosa fare**: si tratta di un problema relativo al momento dell’acquisizione, non di qualcosa che Chloros possa correggere a posteriori. Per lavori di precisione, utilizzare un flusso di lavoro GPS PPK/RTK — consultare l’impostazione**Applica correzioni PPK** in [Impostazioni progetto](../project-settings/project-settings.md).

### La mappa è vuota o il caricamento delle tessere si interrompe

I fornitori di tessere sono servizi online. Se le tessere smettono di arrivare, controlla la connessione di rete del dispositivo, quindi prova a cambiare fornitore. Se avevi effettuato uno zoom estremo, premi il pulsante di ripristino **×** per tornare a un livello di zoom normale e consentire alla mappa di richiedere nuovamente le tessere.***

## Pagine correlate

* [**Griglia immagini**](image-grid.md) — lo stesso set di immagini utilizzato come miniature
* [**Aprire un’immagine a schermo intero**](opening-an-image-full-screen.md) — esaminare un’immagine in dettaglio
* [**Aggiunta di file a un progetto**](../processing-images-gui/adding-files-to-a-project.md) — i pulsanti di aggiunta/rimozione file presenti in questa scheda
