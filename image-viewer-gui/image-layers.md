# Livelli immagine

Il menu a tendina Livelli immagine nel visualizzatore di immagini Chloros consente di passare rapidamente da una versione all&#x27;altra della stessa immagine, dalle acquisizioni originali alle immagini di riflettanza elaborate e alle immagini indice calcolate.

## Cosa sono i livelli immagine?

In Chloros, i **livelli** si riferiscono alle diverse uscite immagine disponibili per una singola immagine sorgente. Quando si elaborano le immagini, Chloros crea più versioni:

* **Immagini originali** (file JPG e RAW dalla fotocamera)
* Risultati **con riflettanza calibrata** (se la calibrazione della riflettanza era abilitata)
* **Immagini target** (se l&#x27;immagine contiene target di calibrazione)
* **Immagini indice** (NDVI, NDRE, GNDVI, ecc. se gli indici erano configurati)

Il **menu a tendina Layer Selector** in alto a destra nell&#x27;Image Viewer consente di passare istantaneamente da una versione all&#x27;altra senza uscire dal visualizzatore.

***

## Tipi di livelli disponibili

### JPG

* L&#x27;immagine di anteprima JPG originale dalla fotocamera
* Sempre disponibile per tutte le immagini
* Non elaborata, così come catturata dalla fotocamera
* La più veloce da caricare e visualizzare

**Quando visualizzarla:**

* Anteprima rapida della cattura originale
* Controllo della composizione e dell&#x27;inquadratura dell&#x27;immagine
* Verifica della qualità della cattura prima dell&#x27;elaborazione

### RAW (originale)

* Dati originali del sensore RAW dalla fotocamera
* Debayering senza post-elaborazione applicata
* Profondità di bit superiore rispetto al JPG (in genere dati del sensore a 12 o 14 bit)

**Quando visualizzare:**

* Ispezione della qualità dei dati originali del sensore
* Controllo di eventuali problemi o artefatti del sensore
* Confronto dei risultati prima/dopo l&#x27;elaborazione

### RAW (Target)

* Appare solo per le immagini identificate come contenenti target di calibrazione
* Mostra l&#x27;immagine RAW originale con il target rilevato
* Utilizzato per verificare che il rilevamento del target abbia avuto esito positivo

**Quando visualizzare:**

* Conferma che i target di calibrazione siano stati rilevati correttamente
* Verifica della qualità dell&#x27;immagine del target
* Risoluzione dei problemi di calibrazione

{% hint style=&quot;info&quot; %}
**Livello target**: questo livello appare solo nel menu a tendina per le immagini che contengono target di calibrazione. Le immagini acquisite normalmente non avranno questa opzione.
{% endhint %}

### RAW (riflettanza)

* Immagine di output con riflettanza calibrata
* Correzione vignettatura (se abilitata nell&#x27;elaborazione)
* Riflettanza calibrata utilizzando i dati target (se abilitati)
* Multibanda TIFF con tutti i canali della fotocamera
* I valori dei pixel rappresentano la riflettanza percentuale (quando si utilizza la modalità percentuale)
* Pronto per essere manipolato con [Index/LUT Sandbox](index-lut-sandbox.md)

**Quando visualizzare:**

* Ispezione dei risultati calibrati
* Verifica della qualità della calibrazione
* Controllo dei valori dei pixel per l&#x27;accuratezza scientifica
* Confronto con l&#x27;originale per vedere gli effetti della calibrazione

{% hint style=&quot;success&quot; %}
**Consigliato**: utilizzare il livello RAW (riflettanza) quando si controllano i valori dei pixel per misurazioni e analisi scientifiche.
{% endhint %}

### RAW (NDVI Indice)... e simili

* Immagine dell&#x27;indice di vegetazione calcolato (NDVI in questo esempio)
* Il nome dell&#x27;indice cambia in base all&#x27;indice configurato durante l&#x27;elaborazione
* Esempi: RAW (NDVI Indice), RAW (NDRE Indice), RAW (GNDVI Indice), ecc.
* Immagine in scala di grigi a banda singola che mostra i risultati del calcolo dell&#x27;indice
* Viene visualizzato un livello per ogni indice configurato nelle impostazioni del progetto

**Possibili nomi degli indici:**

* RAW (indice NDVI)
* RAW (indice NDRE)
* RAW (indice GNDVI)
* RAW (OSAVI Index)
* RAW (EVI Index)
* RAW (SAVI Index)
* E molti altri... (vedere [Formule indici multispettrali](../project-settings/multispectral-index-formulas.md))

**Quando visualizzare:**

* Esaminare i risultati del calcolo dell&#x27;indice
* Controllare gli intervalli di valori dell&#x27;indice
* Identificare le aree di interesse
* Verificare le immagini dell&#x27;indice prima di utilizzarle nel GIS o nell&#x27;analisi

***

## Utilizzo del selettore di livelli

### Apertura del menu a tendina

1. Aprire un&#x27;immagine in modalità a schermo intero (fare clic su una qualsiasi delle miniature nel visualizzatore di immagini)
2. Individuare il **menu a tendina dei livelli** nell&#x27;angolo in alto a destra del visualizzatore
3. Il menu a tendina mostra il livello attualmente selezionato (ad esempio, &quot;JPG&quot;)
4. Fare clic sul menu a tendina per visualizzare tutti i livelli disponibili

### Cambio dei livelli

1. Fare clic sul menu a tendina dei livelli per aprire l&#x27;elenco
2. Vengono visualizzati tutti i livelli disponibili per l&#x27;immagine corrente
3. Fare clic sul nome di un livello qualsiasi per passare a quella versione
4. L&#x27;immagine si aggiorna immediatamente per mostrare il livello selezionato

**Cambio rapido:**

* Il menu a tendina ricorda l&#x27;ultima selezione effettuata
* Quando si passa all&#x27;immagine successiva, Chloros tenta di mostrare lo stesso tipo di livello
* Se quel livello non esiste nell&#x27;immagine successiva, viene impostato di default JPG

### Disponibilità dei livelli

Non tutti i livelli sono disponibili per ogni immagine:

**Sempre disponibili:**

* ✅ JPG (ogni immagine ha un&#x27;anteprima JPG)

**Disponibili in modo condizionale:**

* ⚠️ RAW (originale) - Solo se l&#x27;immagine è stata acquisita in modalità RAW o RAW+JPG
* ⚠️ RAW (target) - Solo se l&#x27;immagine contiene target di calibrazione rilevati
* ⚠️ RAW (riflettanza) - Solo dopo l&#x27;elaborazione con la calibrazione della riflettanza abilitata
* ⚠️ RAW (\[Indice] Indice) - Solo dopo l&#x27;elaborazione con gli indici configurati

***

## Persistenza dei livelli

### Navigazione tra le immagini

Quando si naviga verso un&#x27;immagine diversa (utilizzando i tasti freccia o facendo clic sulle miniature):

**La preferenza del livello viene mantenuta:**

* Se si visualizza &quot;RAW (Riflettanza)&quot;, l&#x27;immagine successiva mostra &quot;RAW (Riflettanza)&quot; (se disponibile)
* Se si visualizza &quot;RAW (NDVI Indice)&quot;, l&#x27;immagine successiva mostra &quot;RAW (NDVI Indice)&quot; (se disponibile)
* Se lo stesso livello non esiste, viene impostato JPG come impostazione predefinita

**Esempio di flusso di lavoro:**

1. Aprire l&#x27;immagine 1, passare a RAW (NDVI Index)
2. Premere → per visualizzare l&#x27;immagine 2
3. L&#x27;immagine 2 visualizza automaticamente il livello RAW (NDVI Index)
4. Continua a navigare: tutte le immagini mostrano il livello NDVI
5. Molto efficiente per rivedere i risultati dell&#x27;indice su molte immagini

***

## Flussi di lavoro comuni

### Flusso di lavoro 1: confronto prima/dopo

**Obiettivo**: confrontare l&#x27;immagine originale con quella calibrata

1. Apri l&#x27;immagine elaborata in Image Viewer
2. Seleziona **RAW (originale)** dal menu a tendina
3. Notare la vignettatura e i valori non calibrati
4. Passare a **RAW (Riflettanza)** dal menu a tendina
5. Confrontare: vignettatura rimossa, valori calibrati

### Flusso di lavoro 2: Revisione dell&#x27;indice

**Obiettivo**: Rivedere rapidamente i risultati NDVI in tutto il set di dati

1. Aprire la prima immagine elaborata
2. Selezionare **RAW (NDVI Indice)** dal menu a tendina
3. Utilizzare il tasto freccia → per passare all&#x27;immagine successiva
4. Il livello NDVI persiste automaticamente
5. Continuare con tutte le immagini, controllando i modelli NDVI
6. Passare a **RAW (NDRE Index)** per confrontare

### Flusso di lavoro 3: Verifica del target

**Obiettivo**: Verificare che tutte le immagini target siano state rilevate correttamente

1. Passare a un&#x27;immagine target
2. Selezionare **RAW (Target)** dal menu a tendina
3. Verificare che i target di calibrazione siano chiaramente visibili e rilevati
4. Passare all&#x27;immagine target successiva
5. Ripetere la verifica per tutti i target

### Flusso di lavoro 4: Ispezione del valore dei pixel

**Obiettivo**: Controllare i valori di riflettanza per verificarne l&#x27;accuratezza scientifica

1. Aprire l&#x27;immagine elaborata
2. Selezionare il livello **RAW (Riflettanza)**
3. Abilitare la modalità **Pixel Percent** (pulsante nella barra degli strumenti in alto a destra)
4. Spostare il cursore sulle aree di vegetazione
5. Verificare che i valori dei pixel rientrino negli intervalli previsti (30-70% per NIR, 5-15% per Red)
6. Controllare che le aree di suolo e acqua abbiano valori appropriati

***

## Comprensione dei valori dei pixel per livello

Livelli diversi mostrano intervalli di valori dei pixel diversi:

### Livello JPG

* **Intervallo**: 0-255 (8 bit)
* **Significato**: valori visualizzati, con correzione gamma
* **Uso**: solo ispezione visiva, non per misurazioni scientifiche

### RAW (originale)

* **Intervallo**: 0-65535 (16 bit)
* **Significato**: numeri digitali grezzi del sensore
* **Uso**: verifica delle prestazioni del sensore, non calibrato

### RAW (riflettanza)

* **Intervallo**: 0-65.535 (16 bit TIFF) o 0,0-1,0 (32 bit percentuale)
* **Significato**: riflettanza percentuale calibrata
* **Uso**: misurazioni e analisi scientifiche

**Per 16 bit TIFF:** dividere per 65.535 per ottenere la riflettanza percentuale **Per 32 bit Percent:** i valori rappresentano direttamente la percentuale (0,5 = 50% di riflettanza)

### RAW (immagini indice)

* **Intervallo**: varia in base all&#x27;indice (in genere da -1,0 a +1,0 per gli indici normalizzati)
* **Significato**: risultato del calcolo dell&#x27;indice
* **Esempi**:
  * NDVI: da -1 a +1 (vegetazione in genere da 0,4 a 0,9)
  * NDRE: da -1 a +1 (rilevamento dello stress)
  * EVI: da 0 a 1 (vegetazione migliorata)

***

## Suggerimenti e best practice

### Cambio efficiente dei livelli

* **Scorciatoie da tastiera**: sebbene non esistano scorciatoie da tastiera per i livelli, le frecce di navigazione (←/→) funzionano su tutti i livelli
* **Flussi di lavoro coerenti**: selezionare un livello (ad esempio, NDVI) e rivedere l&#x27;intero set di dati prima di passare a un altro
* **Confronto rapido**: passare da Originale a Riflettanza per verificare la qualità dell&#x27;elaborazione

### Considerazioni sulle prestazioni

* **Il formato JPG è quello che si carica più velocemente**: utilizzarlo per una navigazione rapida tra molte immagini
* **I livelli RAW si caricano più lentamente**: risoluzione e profondità di bit più elevate
* **Livelli indice**: velocità simile a quella dei livelli Riflettanza
* **Il primo caricamento è il più lento**: le visualizzazioni successive dello stesso livello vengono memorizzate nella cache e sono più veloci

### Verifica della qualità

* **Controllare sempre RAW (originale)**: verificare la qualità dei dati di origine prima di fidarsi dei risultati elaborati
* **Confrontare i livelli**: utilizzare il cambio di livello per verificare che l&#x27;elaborazione abbia funzionato correttamente
* **Controllare gli intervalli dell&#x27;indice**: utilizzare la modalità Percentuale pixel con i livelli indice per verificare che i valori siano ragionevoli

***

## Risoluzione dei problemi

### Livello non disponibile

**Problema**: il livello previsto non viene visualizzato nel menu a discesa

**Possibili cause:**

* L&#x27;immagine non è stata elaborata (sono disponibili solo JPG e RAW (originale))
* La calibrazione della riflettanza è stata disabilitata durante l&#x27;elaborazione
* L&#x27;indice specifico non è stato configurato nelle impostazioni del progetto
* L&#x27;immagine è un&#x27;immagine solo target (non sono stati generati indici per i target)

**Soluzioni:**

1. Verificare che l&#x27;immagine sia stata elaborata (controllare la cartella di output per i file elaborati)
2. Controllare le impostazioni del progetto per confermare che gli indici siano stati configurati
3. Elaborare nuovamente con gli indici desiderati abilitati

### Livello errato visualizzato

**Problema**: l&#x27;immagine si apre in un livello inaspettato

**Causa**: le preferenze di livello dell&#x27;immagine precedente sono state riportate, ma quel livello non esiste nell&#x27;immagine corrente

**Soluzione**: Chloros torna automaticamente al formato JPG quando il livello preferito non è disponibile: si tratta di un comportamento normale.

### Impossibile visualizzare i target di calibrazione

**Problema**: il livello RAW (Target) non mostra il rilevamento dei target.

**Possibili cause:**

* I target non sono stati rilevati durante l&#x27;elaborazione
* L&#x27;immagine non contiene effettivamente i target
* Impostazioni di rilevamento dei target troppo rigide

**Soluzioni:**

1. Controllare il log di debug per i messaggi &quot;Target trovato&quot;
2. Verificare che l&#x27;immagine contenga effettivamente target di calibrazione visibili
3. Regolare le impostazioni di rilevamento dei target nelle impostazioni del progetto
4. Vedere [Scelta delle immagini target](../processing-images-gui/choosing-target-images.md)

***

## Funzionalità correlate

### Strumenti del visualizzatore di immagini

Durante la visualizzazione di qualsiasi livello, è possibile utilizzare:

* **Controlli di zoom**: ingrandire per esaminare i dettagli
* **Panoramica**: fare clic e trascinare per spostarsi nell&#x27;immagine ingrandita
* **Controllo del valore dei pixel**: visualizzare i valori nella posizione del cursore
* **Frecce di navigazione**: spostarsi tra le immagini mantenendo il livello
* **Modalità percentuale pixel**: passare dalla visualizzazione DN a quella percentuale

Vedere [Apertura di un&#x27;immagine a schermo intero](opening-an-image-full-screen.md) per la documentazione completa sul visualizzatore di immagini.

### Sandbox indice/LUT

Per test e visualizzazione interattivi dell&#x27;indice:

* **Calcolo dell&#x27;indice in tempo reale**: prova diverse formule dell&#x27;indice
* **Mappatura dei colori LUT**: applica gradienti di colore agli indici in scala di grigi
* **Esporta visualizzazioni**: salva immagini dell&#x27;indice colorate

Per ulteriori dettagli, consulta [Sandbox indice/LUT](index-lut-sandbox.md).

***

## Passaggi successivi

Ora che hai compreso i livelli delle immagini:

* [**Apertura di un&#x27;immagine a schermo intero**](opening-an-image-full-screen.md) - Guida completa all&#x27;Image Viewer
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Visualizzazione interattiva degli indici
* [**Formule dell&#x27;indice multispettrale**](../project-settings/multispectral-index-formulas.md) - Riferimento agli indici disponibili
* [**Completamento dell&#x27;elaborazione**](../processing-images-gui/finishing-the-processing.md) - Comprensione dei risultati elaborati
