# Sandbox Indici/LUT

Il Sandbox Indici/LUT è un&#x27;area di lavoro interattiva all&#x27;interno del visualizzatore di immagini Chloros che consente di sperimentare in tempo reale il calcolo di indici multispettrali e la visualizzazione a colori. Questo potente strumento aiuta a testare diversi indici, perfezionare gli intervalli di valori e creare visualizzazioni pronte per la pubblicazione senza dover rielaborare l&#x27;intero set di dati.

## Che cos&#x27;è l&#x27;Index/LUT Sandbox?

### Scopo

L&#x27;Index/LUT Sandbox offre:

* **Calcolo degli indici in tempo reale** - Applicazione istantanea di qualsiasi indice di vegetazione
* **Regolazione interattiva della LUT** - Messa a punto dei gradienti e degli intervalli di colore
* **Ottimizzazione del flusso di lavoro** - Determinazione delle impostazioni ottimali prima dell&#x27;elaborazione in batch

### Sandbox vs. Elaborazione del progetto

**Sandbox Indici/LUT (Interattiva):**

* Una singola immagine alla volta
* Feedback immediato
* Sperimentale e iterativa
* Nessuna modifica permanente ai file
* Perfetta per esplorare e testare

**Elaborazione del progetto (Batch):**

* Intero set di dati in una volta sola
* Impostazioni preconfigurate
* File di output permanenti
* Richiede molto tempo
* Ideale quando le impostazioni sono definitive

{% hint style="success" %}
**Flusso di lavoro ottimale**: usa la Sandbox per sperimentare e trovare le impostazioni ottimali di indice e LUT, quindi applica tali impostazioni durante l&#x27;elaborazione del progetto per l&#x27;intero set di dati.
{% endhint %}

***

## Utilizzo della Sandbox Indice/LUT

### Comprendere gli indici precalcolati

In Chloros, gli indici possono essere applicati durante l&#x27;elaborazione del progetto. Per determinare quali impostazioni di indice e LUT si desidera applicare alle esportazioni, è più semplice utilizzare la sandbox del visualizzatore di immagini.

La sandbox consente di:

* **Applicare nuovi indici e gradienti di colore (LUT)** per visualizzare i dati
* **Regolare le impostazioni di visualizzazione** in modo interattivo
* **Visualizzare** immagini indice già calcolate
* **Ispezionare** i valori dei pixel a tutti i livelli di zoom

### Apertura della Sandbox

È possibile accedere alla Sandbox Indice/LUT dalla scheda **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Fare clic su un&#x27;immagine nella griglia delle immagini del browser dei file; questa si aprirà nella scheda **Visualizzatore di immagini**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> 2. Fare clic sulla scheda**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> per aprire la barra laterale a comparsa a sinistra, se non è già aperta

### Selezione di un&#x27;immagine a cui applicare un indice/LUT

Per lavorare con un indice nella sandbox <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Apri un&#x27;immagine** dalla griglia delle immagini principale cliccandoci sopra
2. Si aprirà quindi la scheda **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> si aprirà
3. Fare clic sul **menu a tendina Livelli** (in alto a destra nel visualizzatore)
4. Selezionare il livello dal menu a tendina:
   * RAW (Riflettanza)

### Applicazione di un indice a un&#x27;immagine

Una volta che l&#x27;immagine è a schermo intero e la barra laterale della scheda **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> è aperta:

1. Spuntare la casella Indice nella parte superiore della barra laterale
2. Scegliere il filtro della fotocamera dal menu a tendina a sinistra
3. Scegliere la formula dell&#x27;indice desiderata dal menu a tendina a destra
4. Trascinare i cerchi colorati dei canali del filtro nelle posizioni corrispondenti nella formula dell&#x27;indice sottostante
5. Una volta che la formula è valida, l&#x27;immagine si aggiornerà e mostrerà i valori dell&#x27;indice
6. Spostare il cursore del mouse per visualizzare i valori nella posizione del cursore
7. Ingrandisci l&#x27;immagine per vedere i singoli pixel e i valori associati

Ogni indice ha un intervallo di valori e un significato specifici:

#### Esempio NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Per la documentazione completa sulle formule degli indici, consulta [Formule degli indici multispettrali](../project-settings/multispectral-index-formulas.md).

***

## Utilizzo delle LUT (tabelle di consultazione)

### Che cos&#x27;è una LUT?

Una **tabella di consultazione (LUT)** mappa i valori numerici degli indici ai colori per la visualizzazione:

* **Input**: valore del pixel dell&#x27;indice (ad es., NDVI 0,65)
* **Output**: colore (ad es. verde brillante)
* **Scopo**: rendere i modelli più facili da vedere e interpretare**LUT in scala di grigi vs. LUT a colori:**

* Scala di grigi: scientifica e neutra, mostra i dati grezzi
* LUT a colori: intuitiva e d&#x27;impatto, evidenzia modelli e differenze

{% hint style="success" %}
**Potenza di visualizzazione**: l&#x27;applicazione di una LUT a colori a un&#x27;immagine indice in scala di grigi rende notevolmente più facile identificare modelli, anomalie e aree di interesse a colpo d&#x27;occhio.
{% endhint %}

### Applicazione di una LUT a un&#x27;immagine indice

Una volta ottenuta un&#x27;immagine indice che mostra

1. Fare clic sul <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> pulsante &quot;+Aggiungi LUT&quot;
2. Selezionare la sfumatura di colore
3. Regolare i punti finali min/max del clipping
4. Regolare la modalità di clipping
5. Selezionare la casella Indice nella barra laterale della scheda **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> per applicare la LUT

### Scelta di una sfumatura di colore

**Selezione di una sfumatura:**

1. Nel pannello LUT, individuare la**barra della sfumatura colorata**

2. Passare il mouse su di essa per visualizzare le impostazioni predefinite disponibili per la sfumatura
3. Selezionare la sfumatura desiderata
4. L&#x27;immagine **si aggiorna immediatamente** con i nuovi colori quando la casella Indice è selezionata

{% hint style="success" %}
**Best practice**: per indici di vegetazione come NDVI, il gradiente Red-Giallo-Green è il più intuitivo perché si allinea alle associazioni cromatiche naturali (verde=sano, giallo=moderato, rosso=stressato).
{% endhint %}

### Regolazione delle classi di colore

Il **controllo Classi**determina il numero di gradini di colore distinti che compaiono nel gradiente:**Opzioni per il numero di classi:*** **2-5 classi**: categorie molto ampie, zone distinte
* **6-10 classi**: equilibrato, ottimo per la classificazione
* **11-20 classi**: gradienti uniformi, aspetto continuo
* **20+ classi**: quasi continuo, massima uniformità**Come regolare:**

1. Nel pannello LUT, individua i**quadratini dei campioni di colore sotto la barra del gradiente**

2. Regola il numero di classi aggiungendone di nuove con il pulsante +
3. Rimuovi le classi facendo doppio clic su un campione di colore
4. Il gradiente si aggiorna **in tempo reale** sull&#x27;immagine**Effetto sulla visualizzazione:*** **Meno classi** (3-5): Crea zone distinte, classificazione semplificata, categorie più facili da distinguere
* **Numero medio di classi** (6-10): Approccio equilibrato, adatto alla maggior parte delle applicazioni
* **Maggior numero di classi** (15-20): Transizioni fluide, variazioni dettagliate, aspetto fotografico**Quando utilizzarlo:*** **Poche classi (3-5)**: Diapositive di presentazione, mappe di classificazione, report semplici
* **Classi medie (6-10)**: analisi generali, dettagli equilibrati, rapporti standard
* **Molte classi (15-20)**: analisi scientifiche, ispezioni dettagliate, risultati di qualità pubblicabile

### Regolazione fine degli intervalli di valori

I **controlli dell&#x27;intervallo di valori**determinano quali valori dell&#x27;indice vengono mappati su quali colori nel gradiente:**Controlli dell&#x27;intervallo nel pannello LUT:*** **Valore minimo**: limite inferiore della scala cromatica
* **Valore massimo**: limite superiore della scala cromatica
* **Valori intermedi**: distribuiti automaticamente tra il minimo e il massimo (in base al numero di classi)

#### Regolazione dei valori minimo/massimo

**Per regolare gli intervalli di valori:**

1. Nel pannello LUT, individuare i campi di immissione**Valore minimo**e**Valore massimo**

2. Fare clic sul campo**Valore minimo**

3. Digitare il valore minimo desiderato (ad es., `0.2`)
4. Premere **Invio** o fare clic fuori dal campo
5. Ripetere l&#x27;operazione per il campo **Valore massimo** (ad es., `0.9`)
6. La visualizzazione **si aggiorna immediatamente**{% hint style="info" %}**Ridimensionamento automatico**: quando si applica una LUT per la prima volta, Chloros imposta automaticamente il minimo e il massimo in base all&#x27;intervallo di dati effettivo nell&#x27;immagine. È quindi possibile restringere questo intervallo per concentrarsi su specifici intervalli di valori di interesse.
{% endhint %}

**Esempi di regolazioni dell&#x27;intervallo NDVI:*** **Intervallo completo**: da `-1.0` a `1.0` (mostra tutti i valori possibili)
* **Focalizzato sulla vegetazione**: da `0.2` a `0.9` (esclude il suolo nudo e l&#x27;acqua)
* **Solo vegetazione sana**: da `0.5` a `0.9` (evidenzia solo le piante vigorose)
* **Rilevamento dello stress**: da `0.2` a `0.5` (enfatizza le aree problematiche)
* **Intervallo personalizzato**: regola in base ai valori dei pixel osservati**Perché regolare gli intervalli?*** **Aumenta il contrasto** nell&#x27;area di interesse
* **Escludere i valori irrilevanti** (ad es. specchi d&#x27;acqua, terreno nudo)
* **Standardizzare la visualizzazione** su più immagini o date
* **Evidenziare le differenze sottili** all&#x27;interno di un intervallo di valori ristretto

### Ritaglio dei valori fuori intervallo

Quando i valori dei pixel non rientrano nell&#x27;intervallo minimo/massimo definito, è possibile controllare la loro visualizzazione utilizzando le **modalità di ritaglio**.

#### **Opzioni disponibili per le modalità di clipping:**

#### 1. Minimo e Massimo

* Pixel **al di sotto del minimo**→ visualizzati utilizzando il**primo colore** della sfumatura (ad es. rosso)
* Pixel **al di sopra del massimo**→ visualizzati utilizzando l&#x27;**ultimo colore** della sfumatura (ad es. verde)
* **Caso d&#x27;uso**: enfatizzare gli estremi, mostrare l&#x27;intero intervallo di dati con colori saturi ai limiti
* **Esempio**: i valori NDVI inferiori a 0,2 appaiono tutti rossi, i valori superiori a 0,9 appaiono tutti verdi

#### 2. Sfondo trasparente

* I pixel **al di fuori dell&#x27;intervallo**diventano**completamente trasparenti*** Solo i pixel **all&#x27;interno dell&#x27;intervallo** mostrano la sfumatura di colore
* **Caso d&#x27;uso**: sovrapposizione GIS, isolamento di intervalli di valori specifici, evidenziazione solo delle aree di interesse
* **Esempio**: Mostra solo i valori NDVI compresi tra 0,4 e 0,7 a colori, tutto il resto trasparente

{% hint style="warning" %}
**Limitazione della trasparenza**: i pixel trasparenti appariranno come colore di sfondo nel visualizzatore. Quando esportati durante l&#x27;elaborazione, la trasparenza viene preservata nel formato PNG ma non in JPG.
{% endhint %}

#### 3. Sfondo indice

* I pixel **fuori intervallo**vengono visualizzati in**scala di grigi** (mostrando i valori grezzi dell&#x27;indice)
* I pixel **all&#x27;interno dell&#x27;intervallo**mostrano una**sfumatura di colore*** **Caso d&#x27;uso**: Evidenziazione sottile, mantenere il contesto mentre si enfatizzano le aree di interesse
* **Esempio**: Evidenziare a colori la vegetazione stressata (NDVI 0,3-0,5) mostrando le aree sane in grigio

#### 4. Sfondo originale

* I pixel **fuori intervallo**visualizzano l&#x27;**immagine multispettrale originale*** I pixel **all&#x27;interno dell&#x27;intervallo**mostrano un**gradiente di colore*** **Caso d&#x27;uso**: Il più intuitivo - combina il contesto naturale dell&#x27;immagine con una sovrapposizione cromatica analitica
* **Esempio**: Osservare l&#x27;aspetto reale del campo/della coltura con le aree stressate contrassegnate da un codice cromatico sovrapposto

### Scegliere la giusta modalità di ritaglio

| Modalità di ritaglio              | Ideale per                                   | Stile di visualizzazione          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimo e massimo**    | Visualizzazione completa dei dati, analisi scientifica     | Tutti i pixel colorati           |
| **Sfondo trasparente** | Sovrapposizioni GIS, isolamento di intervalli specifici    | Colore nell&#x27;intervallo, vuoto oltre |
| **Sfondo indicizzato**       | Enfasi sottile, mantenimento del contesto dei dati  | Colore sull&#x27;intervallo, grigio oltre  |
| **Sfondo originale**    | Rapporti, presentazioni, analisi intuitiva | Colore sull&#x27;intervallo, foto oltre |

### Creazione di colori LUT personalizzati

Per avere il pieno controllo sulla visualizzazione, è possibile creare **gradienti di colore personalizzati** modificando i singoli punti di colore.**Per creare una sfumatura personalizzata:**

1. Nel pannello LUT, individuare la**barra di anteprima della sfumatura**

2. Cercare i**quadratini dei campioni di colore** sotto la sfumatura
3. **Fare clic su un punto di colore** per selezionarlo
4. Si aprirà un **selettore di colore**

5. Scegli un nuovo colore utilizzando:
   * **Ruota dei colori**: selezione visiva del colore
   * **Cursori RGB/HSV**: controllo preciso del colore
   * **Inserimento codice esadecimale**: specifica esatta del colore (ad es., `#FF0000` per il rosso)
6. Clicca fuori dal selettore di colore **per applicare il nuovo colore**

7. Il gradiente**si aggiorna immediatamente** sull&#x27;immagine**Aggiunta o rimozione di punti di colore:*** **Aggiungi un punto**: Clicca sull&#x27;icona + per aggiungere un nuovo campione alla fine
* **Rimuovi un punto**: Fai doppio clic sul quadrato di colore per rimuovere il campione**Strategie di personalizzazione:*** **Invertire il gradiente**: capovolgere l&#x27;ordine dei colori per invertire il significato (ad es. verde=basso, rosso=alto)
* **Colori del marchio**: abbinare la tavolozza dei colori della propria organizzazione per i report
* **Adatto ai daltonici**: utilizzare combinazioni arancione-blu o viola-giallo
* **Ottimizzazione della stampa**: scegliere colori che funzionino sia nella stampa a colori che in scala di grigi
* **Soglia multipla**: usa colori distinti a soglie di valore specifiche per la classificazione

{% hint style="info" %}
**Salvataggio dei gradienti personalizzati**: i gradienti personalizzati possono essere salvati e riutilizzati. Clicca sull&#x27;icona di salvataggio nel pannello LUT per conservare le tue combinazioni di colori personalizzate per un uso futuro.
{% endhint %}

***

## Flusso di lavoro interattivo

### Aggiornamenti in tempo reale

Tutte le regolazioni LUT nell&#x27;area di prova aggiornano l&#x27;immagine **istantaneamente e in modo interattivo**:

* **Cambia livello** → L&#x27;immagine cambia immediatamente
* **Seleziona gradiente** → I colori si aggiornano istantaneamente
* **Regola intervallo di valori** → Il contrasto cambia in tempo reale
* **Cambia classi** → La fluidità del gradiente si aggiorna immediatamente
* **Modifica il clipping** → La visualizzazione dello sfondo cambia istantaneamente
* **Modifica i colori** → Il gradiente personalizzato viene applicato immediatamente**Non è necessario alcun pulsante &quot;Applica&quot;**: tutte le modifiche sono in tempo reale e interattive!

{% hint style="success" %}
**Feedback in tempo reale**: il feedback visivo istantaneo ti consente di sperimentare rapidamente diverse impostazioni fino a trovare la visualizzazione ottimale per le tue esigenze di analisi.
{% endhint %}

### Flusso di lavoro di perfezionamento iterativo

**Flusso di lavoro tipico per l&#x27;ottimizzazione della LUT:**

1.**Seleziona il livello indice** (ad es. RAW (Riflettanza))
2. **Applica indice** - Scegli il filtro della fotocamera e la formula dell&#x27;indice, trascina i cerchi colorati nella posizione appropriata nella formula dell&#x27;indice
3. **Applicare il gradiente LUT** - Iniziare con il preset Red-Yellow-Green
4. **Esaminare i valori dei pixel** - Spostare il cursore, annotare gli intervalli di valori
5. **Regolare min/max** - Restringere l&#x27;intervallo per concentrarsi sulla vegetazione (ad es. da 0,2 a 0,9)
6. **Scegliere il ritaglio** - Provare &quot;Sfondo originale&quot; per il contesto
7. **Perfezionare i colori** - Personalizzare il gradiente se necessario per un&#x27;enfasi specifica
8. **Finalizzare le impostazioni**- Documentare le impostazioni e copiarle nelle Impostazioni del progetto per l&#x27;elaborazione dell&#x27;esportazione

### Ispezione dei valori dei pixel

Comprendere i valori effettivi dei pixel è fondamentale per impostare intervalli LUT efficaci:**Come ispezionare i valori:**

1. I valori dei pixel vengono visualizzati quando nell&#x27;immagine è**selezionata** la casella &quot;Indice&quot; o entrambe le caselle &quot;Indice&quot; e &quot;LUT&quot;.
2. **Spostare il cursore** su diverse aree dell&#x27;immagine
3. **Osservare i valori dei pixel** visualizzati nella legenda mentre si passa il mouse
4. Ingrandire l&#x27;immagine per vedere i singoli pixel evidenziati con un valore fluttuante
5. **Prendere nota** degli intervalli di valori per le diverse caratteristiche:
   * **Vegetazione sana**: ad es., NDVI 0,55-0,85
   * **Vegetazione stressata**: ad es., NDVI 0,30-0,50
   * **Suolo nudo**: ad es., NDVI 0,05-0,25
   * **Acqua** (se presente): ad es., NDVI da -0,05 a 0,10**Utilizzo dei valori dei pixel per impostare gli intervalli LUT:**Dopo aver esaminato i valori dei pixel, regolare i valori min/max della LUT di conseguenza:**Scenario di esempio:*** **Osservazione**: Valori del suolo = 0,05-0,25, Stressato = 0,25-0,50, Sano = 0,50-0,85
* **Obiettivo**: Visualizzare solo lo stato di salute delle piante (escludere il suolo)
* **Impostazioni LUT**: Min = `0.25`, Max = `0.85`
* **Clipping**: &quot;Sfondo originale&quot; per vedere il suolo nel colore naturale
* **Risultato**: La sfumatura di colore si applica solo alla vegetazione, il suolo viene visualizzato come nell&#x27;immagine originale

{% hint style="info" %}
**Gamma dinamica**: colture, stagioni e fasi di crescita diverse avranno gamme di valori diverse. Controllare sempre i valori dei pixel nel proprio set di dati specifico prima di impostare le gamme LUT.
{% endhint %}

***

## Indici personalizzati (Chloros+)

### Creazione di formule di indici personalizzati

{% hint style="info" %}
**Dove creare**: Gli indici personalizzati possono essere configurati nelle**Impostazioni del progetto** prima dell&#x27;elaborazione, nonché nella barra laterale della sandbox del Visualizzatore di immagini.
{% endhint %}

**Per creare un indice personalizzato:**

1.**Aprire le Impostazioni del progetto** (prima dell&#x27;elaborazione) o la barra laterale della sandbox di Image Viewer
2. Accedere al **menu a tendina Formula indice**

3. Cercare l&#x27;opzione**&quot;Personalizzato&quot;** (è necessario aver effettuato l&#x27;accesso con una licenza Chloros+)
4. **Definire la formula** utilizzando le variabili di banda:
   * Nomi delle bande: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, ecc.
   * Operatori: `+`, `-`, `*`, `/`, `^` (esponente)
   * Funzioni: `sqrt()`, `abs()`, ecc. (se supportate)
   * Parentesi: `()` per l&#x27;ordine delle operazioni
5. **Assegnare un nome all&#x27;indice** (ad es. &quot;MyIndex&quot; o &quot;CustomNDVI&quot;)
6. **Salvare la configurazione**

**Esempi di formule personalizzate:**

```

Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style="warning" %}
**Convalida della formula**: Assicurati che la tua formula utilizzi le bande disponibili nella tua fotocamera. Ad esempio, RedEdge è disponibile solo su fotocamere con un filtro RedEdge.
{% endhint %}

***

## Passi successivi

Ora che hai compreso l&#x27;Index/LUT Sandbox:

* **Applica all&#x27;elaborazione**: Utilizza le impostazioni individuate in [Impostazioni del progetto](../project-settings/project-settings.md)
* **Elaborazione in batch**: applica gli indici ottimizzati a set di dati completi
* **Ulteriori informazioni**: leggi [Formule degli indici multispettrali](../project-settings/multispectral-index-formulas.md)

Documentazione correlata:

* [**Livelli immagine**](image-layers.md) - Gestione e visualizzazione dei livelli
* [**Apertura di un&#x27;immagine a schermo intero**](opening-an-image-full-screen.md) - Nozioni di base sul visualizzatore di immagini
* [**Elaborazione delle immagini (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Flusso di lavoro completo di elaborazione
