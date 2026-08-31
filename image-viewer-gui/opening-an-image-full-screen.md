# Aprire un&#x27;immagine a schermo intero

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Un&#x27;immagine aperta a schermo intero, con il selettore dei livelli in alto a destra</p></figcaption></figure>

Il visualizzatore di immagini Chloros è l&#x27;interfaccia a schermo intero per la visualizzazione, l&#x27;ispezione e la misurazione delle immagini. È qui che è possibile leggere i **valori reali dei pixel** — DN per canale, percentuale di riflettanza o radianza in W/m²/sr/nm — anziché l’anteprima allungata visualizzata sullo schermo.

## Come accedere al Visualizzatore di immagini

### Dal Browser dei file

1. Aprire la scheda **Browser dei file** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Fare clic su una qualsiasi **miniatura** nella [griglia delle immagini](image-grid.md)
3. L’immagine si apre a schermo intero nella scheda **Visualizzatore immagini**

L’immagine si apre sul prodotto che era visualizzato nella griglia. Se la griglia è impostata su `RAW (Reflectance)`, quello è il livello su cui si accede.

### Apertura della barra laterale del Visualizzatore immagini

Fai clic sull’icona **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> nella barra laterale sinistra per aprire il pannello di analisi. Esso contiene, dall’alto verso il basso:

* il nome dell’immagine e il modello della fotocamera
* il pulsante **Esporta/Salva immagini** (solo quando è attivo un indice o una LUT)
* le caselle di controllo **Indice**e**LUT** e il pannello di configurazione dell’indice — vedi [Index/LUT Sandbox](index-lut-sandbox.md)
* il pannello **Valori del cursore**: lettura per canale, istogramma del livello e controllo GSD***

## Navigazione e zoom

### Scorrere le immagini

* **Immagine successiva**: il pulsante → oppure il tasto**→** (freccia destra)
* **Immagine precedente**: il pulsante ← oppure il tasto**←** (freccia sinistra)
* **Vai a un’immagine specifica**: torna alla griglia e clicca sulla relativa miniatura

Lo zoom e la panoramica rimangono attivi mentre ti sposti tra le immagini, così puoi scorrere una serie rimanendo sulla stessa parte del fotogramma.

### Zoom

Lo zoom viene controllato dalla **rotellina del mouse**, con incrementi del 15%, e rimane ancorato al cursore: il punto sotto il puntatore rimane sotto il puntatore. L’intervallo è limitato dalle dimensioni dell’immagine e della finestra: non è possibile ridurre lo zoom oltre l’adattamento alla finestra, mentre il limite massimo è determinato dalla risoluzione nativa dell’immagine.

Non ci sono tasti dedicati allo zoom nel visualizzatore a schermo intero. (Nella griglia, **Ctrl + `+` / `−`** ridimensiona le miniature — si tratta di un controllo diverso.)

### Panoramica con lo zoom attivo

Fare clic e tenere premuto il tasto sinistro del mouse sull’immagine, quindi trascinare. La panoramica è limitata in modo che l’immagine non possa essere trascinata fuori dallo schermo.

### Ispezione pixel per pixel con ingrandimento elevato

Quando l’ingrandimento effettivo supera **60×**, Chloros disegna un riquadro evidenziato attorno al singolo pixel visualizzato sotto il cursore e mostra un valore mobile accanto ad esso.

L’ingrandimento “effettivo” tiene conto della dimensione del blocco GSD: con una dimensione del blocco pari a 8, l’evidenziazione appare a uno zoom di 7,5× anziché a 60×, poiché un pixel visualizzato corrisponde già a 8 × 8 pixel sorgente. Riducendo lo zoom al di sotto della soglia, l’evidenziazione scompare.

### Scorciatoie da tastiera

| Tasto                             | Dove       | Azione                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Schermo intero | Immagine successiva                          |
| **←**                           | Schermo intero | Immagine precedente                      |
| **Ctrl + R**                    | Schermo intero | Ripristina l’indice/la sandbox LUT         |
| **Ctrl + `+`**/**Ctrl + `=`** | Griglia        | Miniature più grandi (4 px per pressione)  |
| **Ctrl + `−`**                  | Griglia        | Miniature più piccole (4 px per ogni pressione) |***

## Valori del cursore

Spostando il cursore sull’immagine, il pannello **Valori del cursore** riporta il valore di ogni canale sottostante.

{% hint style="success" %}
**Questi sono i valori reali del file.** L&#x27;area di lavoro sullo schermo è un&#x27;anteprima allungata a 8 bit e non può fornirli, quindi Chloros campiona il file del prodotto effettivo per la lettura. Ecco perché un fotogramma raw a 12 bit riporta valori superiori a 255 e perché un livello di radianza float32 riporta unità fisiche.
{% endhint %}

### Significato delle colonne

Il pannello si adatta al livello che si sta visualizzando:

| Livello visualizzato              | Colonne mostrate    | Note                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Riflettanza                        | **DN**e**%** | La percentuale viene calcolata in base alla scala propria del file — vedi sotto                                      |
| Radianza                           | **W/m²/sr/nm**   | Valori fisici in formato float; nessuna colonna DN, poiché in questo caso il DN non ha significato                           |
| Grezzo / Debayered / anteprima / JPG    | **DN**           | Numeri digitali interi                                                                         |
| Esportazioni della percentuale di riflettanza a 32 bit | Solo **%**       | Il valore in virgola mobile memorizzato non è un DN, quindi arrotondarlo a un numero intero produrrebbe un risultato privo di significato come `0` o `1` |

Ogni riga è etichettata con il nome del canale corrispondente al filtro della fotocamera — `Red / Green / NIR` per RGN, `Orange / Cyan / NIR` per OCN, `NIR / Green / Blue` per NGB, `Red / Green / Blue` per RGB, e il nome della singola banda per le fotocamere RE, NIR e mono M3M. Ogni etichetta riporta un punto colorato che corrisponde ai cerchi dei canali utilizzati nell’editor delle formule di indice.

Le immagini **di indice e LUT** salvate rappresentano un caso speciale: contengono componenti della mappa dei colori anziché bande spettrali, quindi le loro righe sono etichettate come `Red / Green / Blue` (o `Index` per un file indice a canale singolo) anziché con i nomi dei filtri della fotocamera.

Quando un indice è attivo nell’area di prova, sotto i canali compare una riga aggiuntiva che mostra il **valore dell’indice** in corrispondenza del cursore, con il nome dell’indice e un punto bianco che corrisponde al suo indicatore sull’istogramma.

### La percentuale di riflettanza utilizza la scala propria di ciascun file

{% hint style="warning" %}
**Non dare per scontato che 65535 = 100%.** Chloros memorizza la riflettanza su scale diverse a seconda della fotocamera che l’ha prodotta, e il visualizzatore individua quella corretta per ogni singolo file.
{% endhint %}

| Fonte                  | DN corrispondente alla riflettanza 1,0 | Come viene identificato                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | Tag XMP `Chloros:PixelScale=32768` scritto in ogni esportazione di riflettanza LATTICE. Il margine di 2× consente al file di contenere valori di ρ superiori a 1,0 senza clipping |
| **Survey3**|**65535**                      | Nessun tag di scala XMP Chloros — La calibrazione Survey3 scrive ρ × dtype-max e clippisce a 1,0                                                               |

Il visualizzatore, la sandbox dell’indice/LUT e l’esportazione dell’indice risolvono tutti la scala tramite la stessa singola implementazione, quindi un valore letto in corrispondenza del cursore è lo stesso valore utilizzato dal calcolo dell’indice.

Due conseguenze da tenere presenti:

* Un **valore percentuale a 32 bit**TIFF memorizza DN/65535 come numero in virgola mobile, mentre un**valore a 8 bit** PNG/JPG memorizza DN × 255/65535 — il visualizzatore converte entrambi prima di visualizzare la percentuale.
* Un caso non può essere recuperato: un’**esportazione a 8 bit TIFF di un’acquisizione da sorgente a 8 bit** viene troncata a 0–255 anziché riscalata e, deliberatamente, non riporta alcun tag di scala. Per quei file il pannello stampa solo il valore DN, senza la colonna delle percentuali. Questa è la risposta corretta, non si tratta di un bug.***

## L’istogramma del livello

Sotto le righe del cursore è presente un istogramma in tempo reale del livello che si sta visualizzando, suddiviso in **256 intervalli**. Per impostazione predefinita viene tracciata un’unica curva combinata, ponderata**`(R + 2G + B) / 4`**— lo stesso spazio di misura utilizzato dagli istogrammi della fotocamera LATTICE. Attivando**RGB**, questa viene sostituita da curve per singolo canale nei colori dei canali, fuse in modo additivo in modo che le sovrapposizioni rimangano leggibili. I livelli monocromatici disegnano sempre la curva singola.

L’asse orizzontale è espresso nell’unità propria del livello:

| Livello       | Unità asse  | Massimo asse                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Riflettanza | percentuale    | 125% — il margine del prodotto consente un ρ superiore a 1,0           |
| Radianza    | W/m²/sr/nm | Il picco proprio del fotogramma, arrotondato per eccesso a due cifre significative |
| Dati a 8 bit  | DN         | 255                                                        |
| Dati a 12 bit | DN         | 4095                                                       |
| Dati a 16 bit | DN         | 65535                                                      |

Quando l’asse è in DN e si trova su uno di questi tre limiti massimi, Chloros riconosce anche la profondità di bit di ciò che si sta visualizzando.

Sopra l’istogramma sono presenti tre pulsanti:

| Pulsante     | Impostazione predefinita | Effetto                                                                                                                                                                                                                                                                                                           |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CURSORE** | Attivo      | Traccia delle linee di riferimento sull’istogramma ai valori esatti indicati nelle righe precedenti, in modo da poter vedere dove si trova il pixel sotto il cursore nella distribuzione dei valori del fotogramma. In modalità RGB è presente un indicatore per canale con un colore proprio; altrimenti un unico indicatore bianco al valore combinato |
| **INDICE**| Attivo      | Appare solo quando è attivo un indice. Commuta l’istogramma dalle bande sorgente alla**distribuzione dei valori dell’indice**, con le due soglie di clip tracciate come linee tratteggiate arancioni e il valore dell’indice del cursore come linea bianca                                                          |
| **RGB**| Disattivato     | Passa dalla curva combinata alle curve per singolo canale. Su un sensore mono questo pulsante riporta la dicitura**MONO** ed è disabilitato — c’è un solo canale da visualizzare                                                                                                                                  |

L’istogramma viene calcolato sui **blocchi visibili**, non sui pixel sorgente sottostanti: modificando la dimensione del blocco GSD, la distribuzione viene ricalcolata, in modo che l’istogramma, l’indicatore del cursore e l’immagine visualizzata siano sempre allineati.***

## Dimensione del blocco GSD

Nella parte inferiore del pannello si trova il controllo **GSD (px)**: una casella di immissione numerica, un cursore da**1 a 256**e un pulsante**RESET**.

Questo controllo riduce la risoluzione dell’immagine _visualizzata_ calcolando la media di un blocco N × N di pixel sorgente in un unico pixel visualizzato. `1` è la risoluzione nativa.

* Influisce **sulla visualizzazione a schermo intero, sulle miniature della griglia, sulla lettura del cursore e su entrambi gli istogrammi**: tutto ciò che mostra l’immagine utilizza la stessa risoluzione al suolo.
* Si tratta di un’impostazione **solo per la visualizzazione**. L’elaborazione e l’esportazione non vengono modificate. L’unica eccezione è intenzionale: un’esportazione tramite [Index/LUT Sandbox](index-lut-sandbox.md) salva ciò che si sta visualizzando, quindi mantiene la dimensione del blocco corrente, e il pannello di esportazione avvisa quando la dimensione del blocco supera 1.
* Il valore viene memorizzato **per progetto** come `viewer_display.gsd_bin` in `project.json`, quindi rimane invariato anche dopo la chiusura e la riapertura.
* L’indicazione del cursore riporta il valore del blocco, non del pixel sorgente, ogni volta che la dimensione del blocco è superiore a 1 — il valore visualizzato è la media del blocco sotto il cursore.

{% hint style="info" %}
**Perché “dimensione del blocco” e non centimetri per pixel?** Un’immagine in cm/px richiede un’altezza dal suolo. I dati EXIF di un singolo fotogramma riportano l’altitudine GPS rispetto al livello medio del mare, non rispetto al terreno verso cui era puntata la fotocamera; pertanto, Chloros non visualizzerà una distanza dal suolo che non è in grado di ricavare con precisione. La dimensione del blocco in pixel sorgente è la stessa soluzione alternativa utilizzata dagli strumenti per le nuvole di MAPIR quando la distanza di campionamento al suolo è sconosciuta.
{% endhint %}

***

## Tipi di immagini visualizzabili

Il menu a tendina dei livelli in alto a destra del visualizzatore elenca tutte le versioni dell’immagine corrente. Le voci visualizzate dipendono dalla fotocamera e da ciò che è stato elaborato — consulta [Livelli dell’immagine](image-layers.md) per l’elenco completo e per capire come funziona il menu a tendina.

### Survey3

* **JPG** — il file di anteprima della fotocamera stessa
* **RAW (Originale)** — il file sorgente `.RAW`, sottoposto a debayering per la visualizzazione, senza correzioni
* **RAW (Target)** — un fotogramma identificato come contenente un target di calibrazione
* **RAW (Riflettanza)** — il prodotto di riflettanza calibrato (65535 = ρ 1,0)
* **Corretto per vignettatura**/**Risposta del sensore** — il prodotto di riserva non calibrato
* **Bilanciamento del bianco** — il prodotto con bilanciamento del bianco
* **RAW (Indice `<INDEX>`)**e**LUT `<INDEX>`** — immagini indice calcolate

### LATTICE

Le acquisizioni LATTICE utilizzano lo stesso menu a tendina, con i nomi dei livelli della pipeline:

| Livello                 | Cosa contiene                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Originale)**    | Il fotogramma RAW di origine così come acquisito                                     |
| **RAW (Debayered)**   | L’immagine lineare debayered                                           |
| **RAW (Anteprima)**     | L’anteprima sullo schermo — estensione in falsi colori per telecamere multispettrali |
| **Bilanciamento del bianco**    | L’anteprima visualizzata per le telecamere master RGB (bilanciamento del bianco + gamma)   |
| **RAW (Radianza)**    | Radianza spettrale Float32 in W/m²/sr/nm                              |
| **RAW (Riflettanza)** | Riflettanza uint16, 32768 = ρ 1,0                                    |

La radianza e la riflettanza sono disponibili solo in modalità multispettrale: una telecamera master RGB non dispone di radiometria per banda, pertanto tali livelli non vengono generati per essa.

***

## Indice e applicazione delle LUT

Applicare indici multispettrali e tabelle di ricerca (LUT) dei colori dalla barra laterale:

1. Aprire il **Visualizzatore immagini** nella barra later<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Selezionare **Indice**

3. Scegliere il filtro della propria telecamera e una formula di indice, quindi trascinare i cerchi dei canali negli slot della formula
4. Aggiungi una LUT e seleziona un gradiente, le soglie e una modalità di ritaglio
5. Leggi i valori in corrispondenza del cursore e salva il risultato con **Esporta/Salva immagini**Consulta [Index/LUT Sandbox](index-lut-sandbox.md) per la guida completa.***

## Risoluzione dei problemi

### L’immagine non si apre

**Possibili cause**: il file è stato spostato o eliminato dopo l’importazione; il prodotto non è mai stato salvato; memoria insufficiente per un’immagine di grandi dimensioni.**Cosa fare**:

1. Verificare che il file del livello esista ancora nella struttura di output del progetto
2. Aprire il file in un visualizzatore esterno per confermare che sia intatto
3. Chiudere le altre applicazioni per liberare memoria

### L’immagine è nera, bianca o presenta colori stravolti

**Possibili cause**: lo stretching dello schermo non ha dati su cui lavorare (un fotogramma quasi costante); un livello float32 con valori anomali; un indice che non ha prodotto dati validi.**Cosa fare**:

1. Controlla i valori del cursore: se ogni canale è pari a zero o vicino allo zero, il problema risiede nei dati, non nella visualizzazione
2. Controlla l’istogramma: un singolo picco a un’estremità indica che il fotogramma è troncato o vuoto
3. Controlla il log di elaborazione relativo all’esecuzione che ha generato il livello

### I valori sembrano errati

**Possibili cause**: vi trovate su un livello diverso da quello che pensate; state confrontando una percentuale con un DN grezzo; state confrontando un file LATTICE con un file Survey3 utilizzando lo stesso divisore.**Cosa fare**:

1. Verifica il livello selezionato nel menu a tendina: le unità del pannello seguono il livello
2. Per la riflettanza, utilizza la colonna **%** anziché dividere tu stesso il DN; se devi dividere, usa il valore `Chloros:PixelScale` di quel file (32768 per LATTICE; se assente, significa 65535 per Survey3)
3. Impostare nuovamente la dimensione del blocco GSD su 1 — se superiore a 1, si sta leggendo una media del blocco, non un singolo pixel
4. Verifica che la calibrazione della riflettanza sia stata effettivamente eseguita per quel fotogramma; un prodotto di ripiego non calibrato (Sensor Response / Vignette Corrected) non rappresenta la riflettanza

***

## Passi successivi

* [**Livelli dell’immagine**](image-layers.md) — il nome di ogni livello, se presente, e il significato dei relativi valori
* [**Sandbox indici/LUT**](index-lut-sandbox.md) — crea, ottimizza ed esporta visualizzazioni degli indici
* [**Indicatori sulla mappa**](map-markers.md) — lo stesso set di immagini su una mappa
* [**Formule degli indici multispettrali**](../project-settings/multispectral-index-formulas.md) — il riferimento agli indici

Per il flusso di lavoro di elaborazione, consultare [Elaborazione delle immagini (GUI)](../processing-images-gui/adding-files-to-a-project.md).
