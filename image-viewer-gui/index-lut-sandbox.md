# Sandbox Indice/LUT

La Sandbox Indice/LUT è uno spazio di lavoro interattivo all&#x27;interno del visualizzatore di immagini Chloros che consente di sperimentare in tempo reale calcoli di indici multispettrali e visualizzazioni a colori. Questo potente strumento aiuta a testare diversi indici, perfezionare gli intervalli di valori e creare visualizzazioni pronte per la pubblicazione senza dover rielaborare l&#x27;intero set di dati.

## Che cos&#x27;è la Sandbox Indice/LUT?

### Scopo

La Sandbox offre:

* **Calcolo dell&#x27;indice in tempo reale**: applica istantaneamente qualsiasi indice di vegetazione
* **Regolazione LUT interattiva**: regola con precisione le sfumature e gli intervalli di colore
* **Ottimizzazione del flusso di lavoro**: determina le impostazioni migliori prima dell&#x27;elaborazione in batch

### Sandbox vs. Elaborazione del progetto

**Index/LUT Sandbox (interattivo):**

* Una sola immagine alla volta
* Feedback immediato
* Sperimentale e iterativo
* Nessuna modifica permanente ai file
* Perfetto per esplorare e testare

**Elaborazione del progetto (batch):**

* Intero set di dati in una volta sola
* Impostazioni preconfigurate
* File di output permanenti
* Richiede molto tempo
* Ideale quando le impostazioni sono definitive

{% suggerimento style=&quot;success&quot; %}
**Flusso di lavoro ottimale**: utilizzare Sandbox per sperimentare e trovare le impostazioni ottimali per l&#x27;indice e la LUT, quindi applicare tali impostazioni durante l&#x27;elaborazione del progetto all&#x27;intero set di dati.
{% endhint %}

***

## Utilizzo di Index/LUT Sandbox

### Comprensione degli indici precalcolati

In Chloros, gli indici possono essere applicati durante l&#x27;elaborazione del progetto. Per determinare quali impostazioni di indice e LUT si desidera applicare alle esportazioni, è più semplice utilizzare il visualizzatore di immagini sandbox.

Il sandbox consente di:

* **Applicare nuovi indici e gradienti di colore (LUT)** per visualizzare i dati
* **Regolare le impostazioni di visualizzazione** in modo interattivo
* **Visualizzare** immagini con indici già calcolati
* **Ispezionare** i valori dei pixel a tutti i livelli di zoom

### Apertura della sandbox

È possibile accedere alla sandbox Indice/LUT nella scheda **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Fare clic su un&#x27;immagine nella griglia delle immagini del browser dei file per aprirla nella scheda **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Fare clic sulla scheda **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> per aprire la barra laterale a comparsa a sinistra, se non è già aperta

### Selezionare un&#x27;immagine a cui applicare un indice/LUT

Per lavorare con un indice nella barra laterale <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Aprire un&#x27;immagine** dalla griglia immagini principale cliccandoci sopra
2. Si aprirà la scheda **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> si aprirà
3. Fare clic sul **menu a tendina Layer** (in alto a destra del visualizzatore)
4. Selezionare il livello dal menu a tendina:
   * RAW (Riflettanza)

### Applicazione di un indice a un&#x27;immagine

Una volta che l&#x27;immagine è a schermo intero e la barra laterale **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> è aperta:

1. Selezionare la casella Indice nella parte superiore della barra laterale
2. Scegliere il filtro della fotocamera dal menu a tendina a sinistra
3. Scegliere la formula dell&#x27;indice desiderata dal menu a tendina a destra
4. Trascinare i cerchi colorati del canale del filtro nelle posizioni indicate nella formula dell&#x27;indice sottostante
5. Una volta che la formula è valida, l&#x27;immagine si aggiornerà e mostrerà i valori dell&#x27;indice
6. Spostare il cursore del mouse per vedere i valori nella posizione del cursore
7. Ingrandisci per vedere i singoli pixel e i valori associati.

Ogni indice ha un intervallo di valori e un significato specifici:

#### NDVI Esempio

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

Per la documentazione completa sulla formula dell&#x27;indice, consulta [Formule dell&#x27;indice multispettrale](../project-settings/multispectral-index-formulas.md).

***

## Utilizzo delle LUT (tabelle di ricerca)

### Che cos&#x27;è una LUT?

Una **tabella di ricerca (LUT)** associa i valori numerici dell&#x27;indice ai colori per la visualizzazione:

* **Input**: valore pixel dell&#x27;indice (ad es. NDVI 0,65)
* **Output**: colore RGB (ad es. verde brillante)
* **Scopo**: rendere i modelli più facili da vedere e interpretare

**LUT in scala di grigi vs. LUT a colori:**

* Scala di grigi: scientifica e neutra, mostra i dati grezzi
* LUT a colori: intuitiva e di grande impatto, evidenzia modelli e differenze

{% hint style=&quot;success&quot; %}
**Potere di visualizzazione**: l&#x27;applicazione di una LUT a colori a un&#x27;immagine indice in scala di grigi rende notevolmente più facile identificare a colpo d&#x27;occhio modelli, anomalie e aree di interesse.
{% endhint %}

### Applicazione di una LUT a un&#x27;immagine indice

Una volta ottenuta un&#x27;immagine indice che mostra

1. Fare clic sul pulsante <img src="../.gitbook/assets/image.png" alt="" data-size="line"> pulsante &quot;+Aggiungi LUT&quot;
2. Selezionare la sfumatura di colore
3. Regolare i punti finali min/max di ritaglio
4. Regolare la modalità di ritaglio
5. Selezionare la casella Indice nella barra laterale **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> nella barra laterale per applicare la LUT.

### Scegliere una sfumatura di colore

**Selezionare una sfumatura:**

1. Nel pannello LUT, individuare la **barra della sfumatura colorata**
2. Passa il mouse su di essa per visualizzare le impostazioni predefinite disponibili
3. Seleziona la sfumatura desiderata
4. L&#x27;immagine **si aggiorna immediatamente** con i nuovi colori quando la casella Indice è selezionata

{% suggerimento style=&quot;success&quot; %}
**Best practice**: per gli indici di vegetazione come NDVI, il gradiente Red-Giallo-Green è il più intuitivo perché si allinea alle associazioni cromatiche naturali (verde=sano, giallo=moderato, rosso=stressato).
{% endhint %}

### Regolazione delle classi di colore

Il **controllo Classi** determina il numero di gradazioni di colore distinte che compaiono nel gradiente:

**Opzioni per il numero di classi:**

* **2-5 classi**: categorie molto ampie, zone distinte
* **6-10 classi**: equilibrato, ottimo per la classificazione
* **11-20 classi**: gradienti uniformi, aspetto continuo
* **20+ classi**: quasi continuo, massima uniformità

**Come regolare:**

1. Nel pannello LUT, individuare i **quadrati dei campioni di colore sotto la barra della sfumatura**
2. Regolare il numero di classi aggiungendo con il pulsante +
3. Rimuovere il numero di classi facendo doppio clic su un campione di colore
4. La sfumatura si aggiorna **in tempo reale** sull&#x27;immagine

**Effetto sulla visualizzazione:**

* **Poche classi** (3-5): crea zone distinte, classificazione semplificata, categorie più facili da distinguere
* **Classi medie** (6-10): approccio equilibrato, adatto alla maggior parte delle applicazioni
* **Più classi** (15-20): transizioni uniformi, variazioni dettagliate, aspetto fotografico

**Quando utilizzare:**

* **Poche classi (3-5)**: diapositive di presentazione, mappe di classificazione, report semplici
* **Classi medie (6-10)**: analisi generale, dettagli equilibrati, report standard
* **Molte classi (15-20)**: analisi scientifica, ispezione dettagliata, output di qualità pubblicabile

### Regolazione fine degli intervalli di valori

I **controlli dell&#x27;intervallo di valori** determinano quali valori dell&#x27;indice vengono mappati su quali colori nel gradiente:

**Controlli dell&#x27;intervallo nel pannello LUT:**

* **Valore minimo**: limite inferiore della scala dei colori
* **Valore massimo**: limite superiore della scala dei colori
* **Valori intermedi**: distribuiti automaticamente tra il minimo e il massimo (in base al numero di classi)

#### Regolazione dei valori minimi/massimi

**Per regolare gli intervalli di valori:**

1. Nel pannello LUT, individuare i campi di immissione **Valore minimo** e **Valore massimo**
2. Fare clic sul campo **Valore minimo**
3. Digitare il valore minimo desiderato (ad esempio, `0.2`)
4. Premere **Invio** o fare clic fuori dal campo
5. Ripetere l&#x27;operazione per il campo **Valore massimo** (ad esempio, `0.9`)
6. La visualizzazione **si aggiorna immediatamente**

{% suggerimento style=&quot;info&quot; %}
**Ridimensionamento automatico**: quando si applica una LUT per la prima volta, Chloros imposta automaticamente il minimo/massimo sull&#x27;intervallo di dati effettivo nell&#x27;immagine. È quindi possibile restringere questo intervallo per concentrarsi su intervalli di valori specifici di interesse.
{% endhint %}

**Esempio di regolazioni dell&#x27;intervallo NDVI:**

* **Intervallo completo**: da `-1.0` a `1.0` (mostra tutti i valori possibili)
* **Focus sulla vegetazione**: da `0.2` a `0.9` (esclude il suolo nudo e l&#x27;acqua)
* **Solo vegetazione sana**: da `0.5` a `0.9` (evidenzia solo le piante vigorose)
* **Rilevamento dello stress**: da `0.2` a `0.5` (enfatizza le aree problematiche)
* **Intervallo personalizzato**: regola in base ai valori dei pixel osservati

**Perché regolare gli intervalli?**

* **Aumenta il contrasto** nell&#x27;area di interesse
* **Esclude i valori irrilevanti** (ad esempio, specchi d&#x27;acqua, terreno nudo)
* **Standardizza la visualizzazione** su più immagini o date
* **Enfatizza le differenze sottili** all&#x27;interno di un intervallo di valori ristretto

### Ritaglio dei valori fuori intervallo

Quando i valori dei pixel non rientrano nell&#x27;intervallo minimo/massimo definito, è possibile controllare la loro visualizzazione utilizzando le **modalità di ritaglio**.

#### **Opzioni disponibili per la modalità di ritaglio:**

#### 1. Minimo e massimo

* Pixel **al di sotto del minimo** → visualizzazione utilizzando il **primo colore** della gradazione (ad es. rosso)
* Pixel **superiori al massimo** → visualizzazione utilizzando l&#x27;**ultimo colore** nella gradazione (ad es. verde)
* **Caso d&#x27;uso**: enfatizzare gli estremi, mostrare l&#x27;intervallo completo dei dati con colori saturi ai limiti
* **Esempio**: i valori NDVI inferiori a 0,2 appaiono tutti rossi, i valori superiori a 0,9 appaiono tutti verdi

#### 2. Sfondo trasparente

* I pixel **al di fuori dell&#x27;intervallo** diventano **completamente trasparenti**
* Solo i pixel **all&#x27;interno dell&#x27;intervallo** mostrano la sfumatura di colore
* **Caso d&#x27;uso**: sovrapposizione GIS, isolamento di intervalli di valori specifici, evidenziazione solo delle aree di interesse
* **Esempio**: mostrare solo NDVI 0,4-0,7 a colori, tutto il resto trasparente

{% hint style=&quot;warning&quot; %}
**Limitazione della trasparenza**: i pixel trasparenti appariranno come colore di sfondo nel visualizzatore. Quando esportati durante l&#x27;elaborazione, la trasparenza viene mantenuta nel formato PNG ma non in JPG.
{% endhint %}

#### 3. Sfondo dell&#x27;indice

* I pixel **fuori intervallo** vengono visualizzati in **scala di grigi** (mostrando i valori dell&#x27;indice grezzi)
* I pixel **nell&#x27;intervallo** mostrano una **sfumatura di colore**
* **Caso d&#x27;uso**: evidenziazione sottile, mantenimento del contesto con enfasi sulle aree di interesse
* **Esempio**: evidenziazione a colori della vegetazione stressata (NDVI 0,3-0,5) con visualizzazione delle aree sane in grigio

#### 4. Sfondo originale

* I pixel **fuori intervallo** visualizzano l&#x27;**immagine multispettrale originale**
* I pixel **all&#x27;interno dell&#x27;intervallo** mostrano **una sfumatura di colore**
* **Caso d&#x27;uso**: il più intuitivo - combina il contesto dell&#x27;immagine naturale con una sovrapposizione di colori analitici
* **Esempio**: vedere l&#x27;aspetto reale del campo/coltura con le aree di stress sovrapposte e codificate a colori

### Scegliere la modalità di ritaglio giusta

| Modalità di ritaglio              | Ideale per                                   | Stile di visualizzazione          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimo e massimo**    | Visualizzazione completa dei dati, analisi scientifica     | Tutti i pixel colorati           |
| **Sfondo trasparente** | Sovrapposizioni GIS, isolamento di intervalli specifici    | Colore sull&#x27;intervallo, vuoto oltre |
| **Sfondo indice**       | Enfasi sottile, mantenimento del contesto dei dati  | Colore sull&#x27;intervallo, grigio oltre  |
| **Sfondo originale**    | Rapporti, presentazioni, analisi intuitive | Colore sull&#x27;intervallo, foto oltre |

### Creazione di colori LUT personalizzati

Per un controllo completo sulla visualizzazione, è possibile creare **gradienti di colore personalizzati** modificando i singoli stop di colore.

**Per creare una sfumatura personalizzata:**

1. Nel pannello LUT, individuare la **barra di anteprima della sfumatura**
2. Cercare i **quadrati dei campioni di colore** sotto la sfumatura
3. **Fare clic su un punto di interruzione del colore** per selezionarlo
4. Si apre un **selettore di colori**
5. Scegliere un nuovo colore utilizzando:
   * **Ruota dei colori**: selezione visiva del colore
   * **Cursori RGB/HSV**: controllo preciso del colore
   * **Inserimento codice esadecimale**: specifica esatta del colore (ad es. `#FF0000` per il rosso)
6. Fare clic fuori dal selettore di colori **per applicare il nuovo colore**
7. La sfumatura **viene aggiornata immediatamente** sull&#x27;immagine

**Aggiunta o rimozione di interruzioni di colore:**

* **Aggiungere un&#x27;interruzione**: fare clic sull&#x27;icona + per aggiungere un nuovo campione alla fine
* **Rimuovere un&#x27;interruzione**: fare doppio clic sul quadrato colorato per rimuovere il campione

**Strategie di personalizzazione:**

* **Invertire la sfumatura**: invertire l&#x27;ordine dei colori per invertire il significato (ad esempio, verde=basso, rosso=alto)
* **Colori del marchio**: abbinare la tavolozza dei colori della propria organizzazione per i report
* **Adatto ai daltonici**: utilizzare combinazioni arancione-blu o viola-giallo
* **Ottimizzazione della stampa**: scegliere colori che funzionano sia nella stampa a colori che in scala di grigi
* **Soglie multiple**: utilizzare colori distinti a soglie di valore specifiche per la classificazione

{% hint style=&quot;info&quot; %}
**Salvataggio di gradienti personalizzati**: i gradienti personalizzati possono essere salvati e riutilizzati. Fare clic sull&#x27;icona di salvataggio nel pannello LUT per conservare le combinazioni di colori personalizzate per un utilizzo futuro.
{% endhint %}

***

## Flusso di lavoro interattivo

### Aggiornamenti in tempo reale

Tutte le regolazioni LUT nella sandbox aggiornano l&#x27;immagine **istantaneamente e in modo interattivo**:

* **Cambia livello** → L&#x27;immagine cambia immediatamente
* **Seleziona gradiente** → I colori si aggiornano istantaneamente
* **Regola intervallo di valori** → Il contrasto cambia in tempo reale
* **Cambia classi** → La fluidità del gradiente si aggiorna immediatamente
* **Modifica ritaglio** → La visualizzazione dello sfondo cambia istantaneamente
* **Modifica colori** → Il gradiente personalizzato viene applicato immediatamente

**Non è necessario alcun pulsante &quot;Applica&quot;**: tutte le modifiche sono live e interattive!

{% hint style=&quot;success&quot; %}
**Feedback in tempo reale**: il feedback visivo immediato consente di sperimentare rapidamente diverse impostazioni fino a trovare la visualizzazione ottimale per le proprie esigenze di analisi.
{% endhint %}

### Flusso di lavoro di perfezionamento iterativo

**Flusso di lavoro tipico di ottimizzazione LUT:**

1. **Selezionare il livello dell&#x27;indice** (ad es. RAW (riflettanza))
2. **Applicare l&#x27;indice** - Scegliere il filtro della fotocamera e la formula dell&#x27;indice, trascinare i cerchi colorati nella posizione appropriata nella formula dell&#x27;indice
3. **Applicare il gradiente LUT** - Iniziare con il preset Red-Yellow-Green
4. **Controllare i valori dei pixel** - Spostare il cursore, annotare gli intervalli di valori
5. **Regolare min/max** - Restringere per concentrarsi sulla vegetazione (ad es. da 0,2 a 0,9)
6. **Scegliere il ritaglio** - Provare &quot;Sfondo originale&quot; per il contesto
7. **Perfezionare i colori** - Personalizzare il gradiente se necessario per enfatizzare aspetti specifici
8. **Finalizzare le impostazioni** - Documentare le impostazioni e copiarle nelle Impostazioni di progetto per l&#x27;elaborazione dell&#x27;esportazione

### Controllo dei valori dei pixel

Comprendere i valori effettivi dei pixel è fondamentale per impostare intervalli LUT efficaci:

**Come controllare i valori:**

1. I valori dei pixel vengono visualizzati quando l&#x27;immagine ha la casella Indice o entrambe le caselle Indice e LUT **selezionate**.
2. **Spostare il cursore** su diverse aree dell&#x27;immagine
3. **Osservare i valori dei pixel** visualizzati nella legenda mentre si passa il cursore
4. Ingrandire per vedere i singoli pixel evidenziati con un valore fluttuante
5. **Prendere nota** degli intervalli di valori per le diverse caratteristiche:
   * **Vegetazione sana**: ad esempio, NDVI 0,55-0,85
   * **Vegetazione stressata**: ad esempio, NDVI 0,30-0,50
   * **Suolo nudo**: ad esempio, NDVI 0,05-0,25
   * **Acqua** (se presente): ad esempio, NDVI da -0,05 a 0,10

**Utilizzo dei valori dei pixel per impostare gli intervalli LUT:**

Dopo aver controllato i valori dei pixel, regola il minimo/massimo della LUT di conseguenza:

**Esempio:**

* **Osservazione**: valori del suolo = 0,05-0,25, stressato = 0,25-0,50, sano = 0,50-0,85
* **Obiettivo**: visualizzare solo la salute delle piante (escludere il suolo)
* **Impostazioni LUT**: Min = `0.25`, Max = `0.85`
* **Ritaglio**: &quot;Sfondo originale&quot; per vedere il suolo nel suo colore naturale
* **Risultato**: la sfumatura di colore si applica solo alla vegetazione, il suolo viene visualizzato come nell&#x27;immagine originale

{% hint style=&quot;info&quot; %}
**Intervallo dinamico**: colture, stagioni e fasi di crescita diverse avranno intervalli di valori diversi. Controllare sempre i valori dei pixel nel proprio set di dati specifico prima di impostare gli intervalli LUT.
{% endhint %}

***

## Indici personalizzati (Chloros+)

### Creazione di formule di indici personalizzati

{% hint style=&quot;info&quot; %}
**Dove creare**: gli indici personalizzati possono essere configurati nelle **Impostazioni di progetto** prima dell&#x27;elaborazione, nonché nella barra laterale della sandbox dell&#x27;Image Viewer.
{% endhint %}

**Per creare un indice personalizzato:**

1. **Aprire le Impostazioni progetto** (prima dell&#x27;elaborazione) o la barra laterale della sandbox del Visualizzatore immagini.
2. Passare al **menu a tendina Formula indice**.
3. Cercare l&#x27;opzione **&quot;Personalizzato&quot;** (è necessario aver effettuato l&#x27;accesso con la licenza Chloros+).
4. **Definire la formula** utilizzando le variabili di banda:
   * Nomi delle bande: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, ecc.
   * Operatori: `+`, `-`, `*`, `/`, `^` (esponente)
   * Funzioni: `sqrt()`, `abs()`, ecc. (se supportate)
   * Parentesi: `()` per l&#x27;ordine delle operazioni
5. **Assegnare un nome all&#x27;indice** (ad esempio, &quot;MyIndex&quot; o &quot;CustomNDVI&quot;)
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

{% hint style=&quot;warning&quot; %}
**Convalida della formula**: assicurati che la formula utilizzi le bande disponibili nella tua fotocamera. Ad esempio, RedEdge è disponibile solo su fotocamere con filtro RedEdge.
{% endhint %}

***

## Passaggi successivi

Ora che hai compreso il funzionamento dell&#x27;Index/LUT Sandbox:

* **Applica all&#x27;elaborazione**: utilizza le impostazioni individuate in [Impostazioni progetto](../project-settings/project-settings.md)
* **Elaborazione in batch**: applica gli indici ottimizzati a set di dati completi
* **Ulteriori informazioni**: leggi [Formule indice multispettrale](../project-settings/multispectral-index-formulas.md)

Documentazione correlata:

* [**Livelli immagine**](image-layers.md) - Gestione e visualizzazione dei livelli
* [**Apertura di un&#x27;immagine a schermo intero**](opening-an-image-full-screen.md) - Nozioni di base sul visualizzatore di immagini
* [**Elaborazione delle immagini (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Flusso di lavoro completo dell&#x27;elaborazione
