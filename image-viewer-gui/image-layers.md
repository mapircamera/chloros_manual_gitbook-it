# Livelli dell&#x27;immagine

Il menu a tendina &quot;Livelli dell&#x27;immagine&quot; nel visualizzatore di immagini Chloros consente di passare rapidamente da una versione all&#x27;altra della stessa immagine: dalle acquisizioni originali alle immagini di riflettanza elaborate e alle immagini indice calcolate.

## Cosa sono i livelli immagine?

In Chloros, i **livelli** si riferiscono ai diversi output di immagine disponibili per una singola immagine sorgente. Quando si elaborano le immagini, Chloros crea più versioni:

* **Immagini originali** (file JPG e RAW dalla fotocamera)
* Output **con riflettanza calibrata** (se la calibrazione della riflettanza era abilitata)
* **Immagini target** (se l&#x27;immagine contiene target di calibrazione)
* **Immagini indice** (NDVI, NDRE, GNDVI, ecc. se gli indici erano stati configurati)

Il **menu a tendina Selettore livelli** in alto a destra nel Visualizzatore immagini consente di passare istantaneamente da una versione all&#x27;altra senza uscire dal visualizzatore.***

## Tipi di livelli disponibili

### JPG

* L&#x27;immagine di anteprima JPG originale dalla fotocamera
* Sempre disponibile per tutte le immagini
* Non elaborata, così come acquisita dalla fotocamera
* La più veloce da caricare e visualizzare

**Quando visualizzare:**

* Anteprima rapida dello scatto originale
* Controllo della composizione e dell&#x27;inquadratura dell&#x27;immagine
* Verifica della qualità dello scatto prima dell&#x27;elaborazione

### RAW (Originale)

* I dati RAW originali del sensore della fotocamera
* Debayering senza post-elaborazione applicata
* Profondità di bit superiore rispetto al JPG (tipicamente dati del sensore a 12 o 14 bit)

**Quando visualizzare:**

* Ispezione della qualità dei dati originali del sensore
* Verifica di eventuali problemi del sensore o artefatti
* Confronto dei risultati prima e dopo l&#x27;elaborazione

### RAW (Target)

* Appare solo per le immagini identificate come contenenti target di calibrazione
* Mostra l&#x27;immagine RAW originale con il target rilevato
* Utilizzato per verificare che il rilevamento del target abbia avuto esito positivo

**Quando visualizzare:**

* Per confermare che i target di calibrazione siano stati rilevati correttamente
* Per verificare la qualità dell&#x27;immagine del target
* Per risolvere problemi di calibrazione

{% hint style="info" %}
**Livello Target**: Questo livello appare nel menu a tendina solo per le immagini che contengono target di calibrazione. Le immagini di acquisizione standard non avranno questa opzione.
{% endhint %}

### RAW (Riflettanza)

* L&#x27;immagine di output con riflettanza calibrata
* Vignettatura corretta (se abilitata in elaborazione)
* Riflettanza calibrata utilizzando i dati dei target (se abilitata)
* Multibanda TIFF con tutti i canali della fotocamera
* I valori dei pixel rappresentano la percentuale di riflettanza (quando si utilizza la modalità percentuale)
* Pronta per essere manipolata con [Index/LUT Sandbox](index-lut-sandbox.md)

**Quando visualizzare:**

* Ispezione dei risultati calibrati
* Verifica della qualità della calibrazione
* Controllo dei valori dei pixel per l&#x27;accuratezza scientifica
* Confronto con l&#x27;originale per vedere gli effetti della calibrazione

{% hint style="success" %}
**Consigliato**: utilizzare il livello RAW (Riflettanza) quando si controllano i valori dei pixel per misurazioni e analisi scientifiche.
{% endhint %}

### RAW (Indice NDVI)... e simili

* Immagine dell&#x27;indice di vegetazione calcolato (NDVI in questo esempio)
* Il nome dell&#x27;indice varia in base all&#x27;indice configurato durante l&#x27;elaborazione
* Esempi: RAW (Indice NDVI), RAW (Indice NDRE), RAW (Indice GNDVI), ecc.
* Immagine in scala di grigi a banda singola che mostra i risultati del calcolo dell&#x27;indice
* Viene visualizzato un livello per ogni indice configurato nelle Impostazioni del progetto

**Possibili nomi degli indici:**

* RAW (Indice NDVI)
* RAW (Indice NDRE)
* RAW (Indice GNDVI)
* RAW (Indice OSAVI)
* RAW (Indice EVI)
* RAW (Indice SAVI)
* E molti altri... (vedi [Formule degli indici multispettrali](../project-settings/multispectral-index-formulas.md))

**Quando visualizzare:**

* Esame dei risultati del calcolo dell&#x27;indice
* Verifica degli intervalli dei valori dell&#x27;indice
* Identificazione delle aree di interesse
* Verifica delle immagini dell&#x27;indice prima dell&#x27;utilizzo in GIS o nell&#x27;analisi

***

## Utilizzo del selettore di livelli

### Apertura del menu a tendina

1. Aprire un&#x27;immagine in modalità a schermo intero (fare clic su una qualsiasi miniatura nel Visualizzatore di immagini)
2. Individuare il **menu a tendina dei livelli** nell&#x27;angolo in alto a destra del visualizzatore
3. Il menu a tendina mostra il livello attualmente selezionato (ad es. &quot;JPG&quot;)
4. Fare clic sul menu a tendina per visualizzare tutti i livelli disponibili

### Cambio di livello

1. Fare clic sul menu a tendina dei livelli per aprire l&#x27;elenco
2. Vengono visualizzati tutti i livelli disponibili per l&#x27;immagine corrente
3. Clicca sul nome di qualsiasi livello per passare a quella versione
4. L&#x27;immagine si aggiorna immediatamente per mostrare il livello selezionato

**Cambio rapido:**

* Il menu a tendina ricorda la tua ultima selezione
* Quando si passa all&#x27;immagine successiva, Chloros tenta di mostrare lo stesso tipo di livello
* Se quel livello non esiste sull&#x27;immagine successiva, il valore predefinito è JPG

### Disponibilità dei livelli

Non tutti i livelli sono disponibili per ogni immagine:

**Sempre disponibili:*** ✅ JPG (ogni immagine ha un&#x27;anteprima JPG)

**Disponibili in base alle condizioni:**

* ⚠️ RAW (Originale) - Solo se l&#x27;immagine è stata acquisita in modalità RAW o RAW+JPG
* ⚠️ RAW (Target) - Solo se l&#x27;immagine contiene target di calibrazione rilevati
* ⚠️ RAW (Riflettanza) - Solo dopo l&#x27;elaborazione con la calibrazione della riflettanza abilitata
* ⚠️ RAW (\[Indice] Indice) - Solo dopo l&#x27;elaborazione con gli indici configurati

***

## Persistenza dei livelli

### Navigazione tra le immagini

Quando si passa a un&#x27;immagine diversa (utilizzando i tasti freccia o cliccando sulle miniature):**La preferenza del livello viene mantenuta:**

* Se si sta visualizzando &quot;RAW (Riflettanza)&quot;, l&#x27;immagine successiva mostra &quot;RAW (Riflettanza)&quot; (se disponibile)
* Se si sta visualizzando &quot;RAW (NDVI Indice)&quot;, l&#x27;immagine successiva mostra &quot;RAW (NDVI Indice)&quot; (se disponibile)
* Se lo stesso livello non esiste, viene impostato di default il formato JPG

**Esempio di flusso di lavoro:**

1. Aprire l&#x27;immagine 1, passare a RAW (NDVI Index)
2. Premere → per visualizzare l&#x27;immagine 2
3. L&#x27;immagine 2 visualizza automaticamente il livello RAW (NDVI Index)
4. Continua a navigare: tutte le immagini mostrano il livello NDVI
5. Molto efficiente per esaminare i risultati dell&#x27;indice su molte immagini

***

## Flussi di lavoro comuni

### Flusso di lavoro 1: Confronto prima/dopo

**Obiettivo**: Confrontare l&#x27;immagine originale con quella calibrata

1. Apri l&#x27;immagine elaborata nel Visualizzatore immagini
2. Selezionare **RAW (Originale)** dal menu a tendina
3. Notare la vignettatura e i valori non calibrati
4. Passare a **RAW (Riflettanza)** dal menu a tendina
5. Confrontare: vignettatura rimossa, valori calibrati

### Flusso di lavoro 2: Revisione dell&#x27;indice

**Obiettivo**: Esaminare rapidamente i risultati NDVI su tutto il set di dati

1. Aprire la prima immagine elaborata
2. Selezionare **RAW (NDVI Indice)** dal menu a tendina
3. Utilizzare il tasto freccia → per passare all&#x27;immagine successiva
4. Il livello NDVI persiste automaticamente
5. Continuare attraverso tutte le immagini, controllando i modelli NDVI
6. Passare a **RAW (NDRE Index)** per confrontare

### Flusso di lavoro 3: Verifica dei target

**Obiettivo**: Verificare che tutte le immagini dei target siano state rilevate correttamente

1. Passare a un&#x27;immagine di target
2. Selezionare **RAW (Target)** dal menu a tendina
3. Verificare che i target di calibrazione siano chiaramente visibili e rilevati
4. Passare all&#x27;immagine del bersaglio successivo
5. Ripetere la verifica per tutti i bersagli

### Flusso di lavoro 4: Ispezione dei valori dei pixel

**Obiettivo**: Verificare i valori di riflettanza per l&#x27;accuratezza scientifica

1. Aprire l&#x27;immagine elaborata
2. Selezionare il livello **RAW (Riflettanza)**

3. Abilitare la modalità**Percentuale pixel** (pulsante nella barra degli strumenti in alto a destra)
4. Spostare il cursore sulle aree di vegetazione
5. Verificare che i valori dei pixel rientrino negli intervalli previsti (30-70% per NIR, 5-15% per Red)
6. Controllare che le aree di suolo e acqua presentino valori appropriati

***

## Comprensione dei valori dei pixel per livello

Livelli diversi mostrano intervalli di valori dei pixel diversi:

### Livello JPG

* **Intervallo**: 0-255 (8 bit)
* **Significato**: Valori di visualizzazione, corretti per la gamma
* **Uso**: Solo ispezione visiva, non per misurazioni scientifiche

### RAW (Originale)

* **Intervallo**: 0-65535 (16 bit)
* **Significato**: valori digitali grezzi del sensore
* **Uso**: verifica delle prestazioni del sensore, non calibrati

### RAW (riflettanza)

* **Intervallo**: 0-65.535 (16 bit TIFF) o 0,0-1,0 (32 bit percentuale)
* **Significato**: Percentuale di riflettanza calibrata
* **Uso**: Misurazioni scientifiche e analisi**Per TIFF a 16 bit:**Dividere per 65.535 per ottenere la percentuale di riflettanza**Per Percentuale a 32 bit:** I valori rappresentano direttamente la percentuale (0,5 = 50% di riflettanza)

### RAW (immagini indice)

* **Intervallo**: varia in base all&#x27;indice (tipicamente da -1,0 a +1,0 per gli indici normalizzati)
* **Significato**: risultato del calcolo dell&#x27;indice
* **Esempi**:
  * NDVI: da -1 a +1 (vegetazione tipicamente da 0,4 a 0,9)
  * NDRE: da -1 a +1 (rilevamento dello stress)
  * EVI: da 0 a 1 (vegetazione migliorata)

***

## Suggerimenti e migliori pratiche

### Cambio efficiente dei livelli

* **Scorciatoie da tastiera**: sebbene non esistano scorciatoie da tastiera per i livelli, le frecce di navigazione (←/→) funzionano su tutti i livelli
* **Flussi di lavoro coerenti**: scegliete un livello (ad es. NDVI) ed esaminate l&#x27;intero set di dati prima di passare a un altro
* **Confronto rapido**: Passare da Originale a Riflettanza per verificare la qualità dell&#x27;elaborazione

### Considerazioni sulle prestazioni

* **I file JPG si caricano più velocemente**: utilizzarli per una navigazione rapida tra molte immagini
* **I livelli RAW si caricano più lentamente**: risoluzione e profondità di bit più elevate
* **Livelli indice**: velocità simile a quella dei livelli di riflettanza
* **Il primo caricamento è il più lento**: le visualizzazioni successive dello stesso livello vengono memorizzate nella cache e sono più veloci

### Verifica della qualità

* **Controlla sempre il RAW (Originale)**: verifica la qualità dei dati di origine prima di fidarti dei risultati elaborati
* **Confronta i livelli**: usa il cambio di livello per verificare che l&#x27;elaborazione abbia funzionato correttamente
* **Controlla gli intervalli dell&#x27;indice**: usa la modalità Percentuale di pixel con i livelli indice per verificare che i valori siano ragionevoli***

## Risoluzione dei problemi

### Livello non disponibile

**Problema**: il livello previsto non compare nel menu a tendina**Possibili cause:**

* L&#x27;immagine non è stata elaborata (sono disponibili solo JPG e RAW (Originale))
* La calibrazione della riflettanza è stata disabilitata durante l&#x27;elaborazione
* Un indice specifico non è stato configurato nelle Impostazioni del progetto
* L&#x27;immagine è un&#x27;immagine solo target (non vengono generati indici per i target)

**Soluzioni:**

1. Verificare che l&#x27;immagine sia stata elaborata (controllare la cartella di output per i file elaborati)
2. Controllare le Impostazioni del progetto per confermare che gli indici siano stati configurati
3. Rielaborare con gli indici desiderati abilitati

### Livello errato visualizzato

**Problema**: L&#x27;immagine si apre in un livello inaspettato**Causa**: La preferenza di livello dell&#x27;immagine precedente è stata riportata, ma quel livello non esiste nell&#x27;immagine corrente**Soluzione**: Chloros ricade automaticamente su JPG quando il livello preferito non è disponibile - questo è un comportamento normale

### Impossibile vedere i target di calibrazione

**Problema**: il livello RAW (Target) non mostra il rilevamento dei target**Possibili cause:**

* I target non sono stati rilevati durante l&#x27;elaborazione
* L&#x27;immagine non contiene effettivamente dei target
* Impostazioni di rilevamento dei target troppo rigide

**Soluzioni:**

1. Controllare il registro di debug per i messaggi &quot;Target trovato&quot;
2. Verificare che l&#x27;immagine contenga effettivamente dei target di calibrazione visibili
3. Regolare le impostazioni di rilevamento dei target nelle Impostazioni del progetto
4. Vedere [Scelta delle immagini target](../processing-images-gui/choosing-target-images.md)

***

## Funzionalità correlate

### Strumenti del visualizzatore di immagini

Durante la visualizzazione di qualsiasi livello, è possibile utilizzare:

* **Controlli di zoom**: ingrandire per ispezionare i dettagli
* **Panoramica**: cliccare e trascinare per spostarsi nell&#x27;immagine ingrandita
* **Ispezione del valore dei pixel**: visualizzare i valori nella posizione del cursore
* **Frecce di navigazione**: spostarsi tra le immagini mantenendo il livello
* **Modalità Percentuale pixel**: passare dalla visualizzazione in DN a quella in percentuale

Vedere [Apertura di un&#x27;immagine a schermo intero](opening-an-image-full-screen.md) per la documentazione completa del Visualizzatore di immagini.

### Sandbox Indice/LUT

Per test e visualizzazione interattivi dell&#x27;indice:

* **Calcolo dell&#x27;indice in tempo reale**: Prova diverse formule di indice
* **Mappatura dei colori LUT**: Applica gradienti di colore agli indici in scala di grigi
* **Esporta visualizzazioni**: Salva immagini dell&#x27;indice colorate

Vedi [Sandbox Indice/LUT](index-lut-sandbox.md) per i dettagli.

***

## Passi successivi

Ora che hai compreso i livelli dell&#x27;immagine:

* [**Aprire un&#x27;immagine a schermo intero**](opening-an-image-full-screen.md) - Guida completa a Image Viewer
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Visualizzazione interattiva degli indici
* [**Formule degli indici multispettrali**](../project-settings/multispectral-index-formulas.md) - Riferimento agli indici disponibili
* [**Completamento dell&#x27;elaborazione**](../processing-images-gui/finishing-the-processing.md) - Comprensione dei risultati elaborati
