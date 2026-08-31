# Area di prova Index/LUT

La Sandbox Indice/LUT è l&#x27;area di lavoro interattiva presente nella barra laterale del visualizzatore di immagini Chloros. È possibile selezionare una formula, associarvi i canali della fotocamera, colorarla con una sfumatura e regolare l&#x27;intervallo dei valori: l&#x27;immagine si aggiorna in tempo reale man mano che si effettuano le modifiche. A partire dalla versione 1.2.0 è anche possibile **salvare ciò che si è creato**, per una singola immagine o per l’intero progetto, senza bisogno di rielaborare i dati.

## A cosa serve la Sandbox

| Sandbox Indice/LUT (interattiva)        | Elaborazione del progetto (in batch)       |
| -------------------------------------- | -------------------------------- |
| Una singola immagine alla volta, feedback immediato  | L’intero set di dati in un unico ciclo     |
| Sperimentale e iterativo             | Impostazioni preconfigurate          |
| Renderizza in tempo reale; salva solo quando richiesto  | Scrive sempre i file di output      |
| Perfetto per trovare le impostazioni giuste | Ideale una volta che le impostazioni sono definitive |

{% hint style="success" %}
**Il flusso di lavoro standard**: regola le impostazioni nella Sandbox finché la visualizzazione non corrisponde a ciò che desideri, quindi esporta direttamente dalla Sandbox oppure copia le stesse impostazioni di indice e LUT nelle [Impostazioni del progetto](../project-settings/project-settings.md) in modo che la successiva elaborazione le applichi a ogni immagine.
{% endhint %}

***

## Apertura della Sandbox

1. Clicca su un’immagine nella griglia: si aprirà a schermo intero nella scheda **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Fare clic sull’icona **Visualizzatore immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> per far scorrere fuori la barra laterale sinistra, se non è già aperta
3. Scegliere un livello multibanda dal menu a tendina dei livelli in alto a destra — **RAW (Riflettanza)** è la scelta più comune, poiché i valori dell’indice calcolati sulla riflettanza calibrata sono comparabili tra le immagini

La barra laterale mostra, dall’alto verso il basso:

* il nome dell’immagine e il modello della fotocamera
* il pulsante **Esporta/Salva immagini** — appare una volta selezionata l’opzione Indice o LUT
* le caselle di controllo **Indice**e**LUT**
* il pannello di configurazione dell’indice
* il pannello **Valori del cursore** con la lettura, l’istogramma e il controllo GSD

{% hint style="warning" %}
**Non disponibile per le fotocamere monocromatiche.** Su un’immagine LATTICE M3M a banda singola entrambe le caselle di controllo sono disabilitate, con il suggerimento _&quot;Non disponibile per sensori monocromatici (M3M)&quot;_ — un indice multibanda non è definito su una singola banda. Per calcolare gli indici dalle telecamere M3M, combinare due o più immagini in uno stack multibanda allineato e utilizzare il motore di indici LATTICE.
{% endhint %}

***

## Applicazione di un indice

1. Spuntare la casella **Indice** nella parte superiore della barra laterale
2. Scegliere il filtro della propria fotocamera dal menu a tendina a sinistra (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Scegliere una formula di indice dal menu a tendina a destra: 27 formule predefinite, più eventuali formule personalizzate salvate
4. La formula viene visualizzata come espressione matematica qui sotto, con un cerchio vuoto in corrispondenza di ogni slot di banda. **Trascinare un cerchio colorato del canale su uno slot** per associarlo
5. Una volta associati tutti gli slot utilizzati dalla formula, l’immagine si aggiorna e mostra i valori dell’indice
6. Passare il cursore sull’immagine per leggere i valori; il pannello **Valori del cursore** aggiunge una riga dell’indice con il valore sotto il cursore

Fai doppio clic su uno slot associato per cancellarlo. Una formula incompleta è un normale stato durante il trascinamento, non un errore: l’immagine semplicemente non si aggiorna finché la formula non è completa.

I cerchi dei canali sono contrassegnati da colori: rosso = Red, verde = Green, blu = Blue, arancione = Orange, ciano = Cyan, viola = NIR, magenta = RE. Gli stessi colori sono utilizzati per i punti dei canali e le curve dell’istogramma nel pannello “Valori del cursore”.

### Esempio di NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Per il riferimento completo alle formule — tutti e tre gli elenchi di preimpostazioni e quali nomi funzionano in quali contesti — consultare [Formule degli indici multispettrali](../project-settings/multispectral-index-formulas.md).

### Con l’opzione “Indice” selezionata ma senza LUT

L’immagine viene disegnata in **scala di grigi**, estesa tra i due valori di soglia. Si tratta di una scelta deliberata: l’immagine dell’indice è costituita da dati scalari e la scala di grigi ne rappresenta la resa più fedele. Aggiungere una LUT quando si desidera il colore.***

## Utilizzo delle LUT (tabelle di consultazione)

Una **tabella di consultazione** associa i valori dell’indice ai colori: in ingresso NDVI 0,65, in uscita un particolare verde. Non modifica i dati, ma cambia il modo in cui vengono interpretati.

### Aggiunta di una LUT

1. Clicca sul pulsante **&quot;+ Aggiungi LUT&quot;** <img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> sotto la formula
2. Scegli una sfumatura di colore
3. Imposta il minimo e il massimo di clipping
4. Scegli una modalità di clipping
5. Spunta la casella **LUT** nella barra laterale per renderizzarla

La casella di controllo LUT rimane disabilitata finché non viene effettivamente configurata una LUT sull’indice.

### Scegliere una sfumatura di colore

Passa il mouse sulla **barra della sfumatura**per aprire l’elenco delle impostazioni predefinite — Chloros include**sette** impostazioni predefinite per le sfumature:

| # | Sfumatura                            | Forma                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Giallo → Green (**predefinito**)  | Divergente — corrisponde alla consueta percezione della vegetazione: verde = sano |
| 2 | Viola → Giallo → Green             | Divergente, con una parte bassa ben distinta                                  |
| 3 | Marrone → Bianco → Blue                | Divergente attorno a un punto medio chiaro                                   |
| 4 | Nero → Viola → Rosa → Giallo pallido | Sequenziale, dal buio alla luce                                           |
| 5 | Red → Giallo → Blue                 | Divergente attorno a un punto medio chiaro                                   |
| 6 | Viola → Blue → Green → Giallo      | Sequenziale, dal più scuro al più chiaro                                           |
| 7 | Orange → Bianco → Viola             | Divergente attorno a un punto medio chiaro                                   |

Un gradiente **divergente**posiziona un colore neutro al centro della finestra, il che risulta efficace quando il punto medio ha un significato specifico (una soglia, una data di riferimento). Un gradiente**sequenziale** varia in modo monotono dal più scuro al più chiaro, il che risulta efficace per una quantità che presenta solo i valori &quot;più&quot; e &quot;meno&quot;.

Ogni preimpostazione ha sette punti di colore. Clicca su una preimpostazione e l’immagine si aggiorna immediatamente (quando la casella LUT è spuntata).

### Modifica dei punti di colore

Sotto la barra del gradiente c’è una fila di campioni di colore, uno per ogni punto:

* **Modifica un colore**: clicca su un campione per aprire il selettore di colore (ruota dei colori, cursori RGB/HSV o un codice esadecimale come `#FF0000`)
* **Aggiungere una tappa**: clicca sul pulsante**+** alla fine della riga — verrà aggiunta una tappa bianca
* **Rimuovere una tappa**:**fai doppio clic** sul campione
* **Conserva un gradiente modificato**: clicca sull’icona di salvataggio accanto alla barra del gradiente per aggiungere il gradiente modificato all’elenco dei preset, in modo da poterlo selezionare nuovamente

Il gradiente che hai configurato su un indice viene memorizzato insieme a quell’indice nelle impostazioni del progetto, quindi rimane disponibile anche dopo la chiusura e la riapertura del progetto.

**Un numero minore di punti**produce zone distinte che si interpretano come una classificazione;**un numero maggiore di punti** produce transizioni morbide, quasi fotografiche. Da tre a cinque punti sono adatti per diapositive di presentazione e mappe di classificazione; da sei a dieci sono adatti per analisi generali; quindici o più sono adatti per ispezioni dettagliate e figure di pubblicazione.

### Impostazione dell’intervallo di valori

Il controllo della soglia è un **cursore a doppia manopola**che va da −1 a +1, con una casella di testo modificabile a ciascuna estremità per i valori esatti e un pulsante**AUTO**.

* Trascinare una delle due manopole oppure digitare un numero nella casella corrispondente e premere Invio
* **AUTO**imposta l’intervallo sui**2° e 98° percentili** dei valori di indice validi dell’immagine — un buon punto di partenza che ignora i valori anomali. Chloros arrotonda il risultato in modo adattivo: a 4 cifre decimali per un intervallo molto ristretto, a 3 per uno ristretto e a 2 in tutti gli altri casi
* Qualsiasi regolazione manuale ha la precedenza su **AUTO**fino a quando non si preme nuovamente**AUTO**

Esempio di finestre NDVI:

| Obiettivo                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Mostra tutto                         | −1,0 | 1,0 |
| Solo vegetazione, esclude suolo e acqua | 0,2  | 0,9 |
| Solo vegetazione sana                 | 0,5  | 0,9 |
| Enfatizza lo stress                        | 0,2  | 0,5 |

Restringere la finestra aumenta il contrasto all’interno dell’area di interesse ed esclude tutto il resto dal range — dove la **Modalità di ritaglio** decide cosa ne sarà di esso.***

## Modalità di ritaglio

Quando il valore dell’indice di un pixel cade al di fuori della finestra min/max, la Modalità di ritaglio decide come verrà disegnato.

| Etichetta del menu a tendina                  | Valore memorizzato      | I pixel fuori intervallo vengono disegnati come                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimo e massimo** (impostazione predefinita) | `clip`            | Il colore più vicino all’estremità del gradiente — i valori inferiori al minimo assumono il primo colore, quelli superiori al massimo assumono l’ultimo |
| **Sfondo trasparente**      | `transparent`     | Completamente trasparente (alfa reale)                                                                                                  |
| **Sfondo indicizzato**| `indexColor`      | Scala di grigi, estesa su**tutto** l’intervallo di indice dell’immagine, in modo che le strutture fuori intervallo rimangano visibili in grigio                |
| **Sfondo originale**         | `backgroundColor` | L’immagine sottostante stessa, quindi la sovrapposizione di colore si trova sopra la scena reale                                                |

| Modalità                       | Ideale per                               | Aspetto                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimo e massimo**      | Visualizzazione completa dei dati, analisi scientifica | Ogni pixel colorato                      |
| **Sfondo trasparente** | Sovrapposizioni GIS, isolamento di una fascia di valori   | Colore all’interno della finestra, nulla all’esterno |
| **Sfondo indicizzato**       | Enfasi mantenendo il contesto dei dati    | Colore all’interno, grigio all’esterno               |
| **Sfondo originale**    | Rapporti e presentazioni              | Colore all’interno, fotografia all’esterno         |

{% hint style="info" %}
**I pixel privi di dati sono sempre trasparenti, in ogni modalità.** Un pixel il cui indice non è finito (una divisione per 0) oppure è esattamente −1,0 o +1,0 (sentinelle di saturazione, dovute al fatto che una banda registra zero mentre l’altra no) viene trattato come “nessun dato” anziché come un valore estremo. Ciò consente di escludere dalla scala cromatica le alte luci bruciate e le ombre morte, anziché rappresentarle come i valori più estremi presenti nell’inquadratura. La stessa regola definisce quali pixel alimentano le soglie AUTO e l’istogramma degli indici, in modo che tutti e tre siano coerenti.
{% endhint %}

La trasparenza viene preservata quando l’esportazione viene salvata come PNG. Non può essere rappresentata in JPG.

***

## Lettura dei valori durante la regolazione

Il pannello **Valori del cursore** sotto il pannello di configurazione funge da strumento di misurazione per la Sandbox:

* Spostare il cursore sull’immagine e leggere i valori sorgente per ciascun canale, oltre al valore dell’indice nella riga corrispondente
* Attiva il pulsante **INDICE** sopra l’istogramma per visualizzare la distribuzione dei valori di indice nel fotogramma, con le due soglie di ritaglio tracciate come linee tratteggiate arancioni e il valore del cursore come linea bianca: questo è il modo più veloce per scegliere una finestra che contenga effettivamente i tuoi dati
* Attiva **CURSOR** per visualizzare le linee di riferimento in corrispondenza dei valori sotto il puntatore
* Ingrandisci oltre 60× (meno se è impostata una dimensione del blocco GSD) per evidenziare i singoli pixel visualizzati con un valore fluttuante

Una procedura pratica:

1. Prendi nota dei valori relativi alla vegetazione sana, alla vegetazione stressata, al suolo nudo e all’acqua
2. Osserva dove si trovano quei gruppi sull’istogramma dell’indice
3. Imposta i valori min/max per racchiudere il gruppo che ti interessa
4. Scegli una modalità di ritaglio: _Sfondo originale_ mantiene visibile la scena circostante

***

## Esportazione dalla Sandbox

Tutto quanto sopra è un’anteprima in tempo reale finché non si salva. Il pulsante **Esporta/Salva immagini** nella parte superiore della barra laterale apre un pannello che scorre sopra la barra laterale (anziché coprire l’immagine, in modo da poter continuare a vedere ciò su cui si sta decidendo).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Opzioni

| Opzione                          | Effetto                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Applica all&#x27;immagine corrente**      | Salva esattamente l&#x27;immagine visualizzata, con queste impostazioni                                                                                                |
| **Applica a tutte le immagini del progetto** | Riesegue la stessa configurazione su ogni immagine del progetto. Le immagini prive delle bande richieste da questo indice vengono ignorate, non considerate come errori |
| **Barra di gradiente indice/LUT**      | Scrive anche un’immagine di legenda separata per ogni esportazione, con l’intervallo di valori etichettato                                                                     |
| **Istogramma indice**             | Scrive anche un’immagine di istogramma separata per ogni esportazione, mostrando i valori minimi/massimi dei dati e le soglie di clipping                                               |

Se la **dimensione del blocco GSD** nella scheda dell’immagine è superiore a 1, il pannello lo segnala prima di confermare: l’esportazione salva ciò che si sta visualizzando, media dei blocchi inclusa. Impostare nuovamente il controllo GSD su 1 se si desidera la risoluzione completa.

### Dove vengono salvati i file

Ogni clic su **Esporta**crea una**nuova cartella, che non verrà mai riutilizzata**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Esempi: `Sandbox_Exports/NDVI_LUT_001/`, poi `Sandbox_Exports/NDVI_LUT_002/` per l’esecuzione successiva. La numerazione viene generata analizzando ciò che è già presente sul disco, quindi rimane invariata anche dopo i riavvii e la cancellazione manuale delle cartelle. Nulla viene mai sovrascritto: lo scopo principale della Sandbox è confrontare un tentativo con l’ultimo.

All’interno della cartella, per ogni immagine:

| File                                                   | Contenuto                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | L’immagine renderizzata, pixel per pixel come visualizzata dal visualizzatore |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Il file sidecar della barra del gradiente, se richiesto                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Il file sidecar dell’istogramma degli indici, se richiesto                  |

I due file di accompagnamento vengono sempre scritti a **risoluzione piena**, anche quando l&#x27;immagine principale è mediata per blocchi: la dimensione di un blocco corrisponde alla risoluzione di visualizzazione ed entrambi i file di accompagnamento riportano i valori reali dell&#x27;indice per ogni singolo pixel. Inoltre, riportano più informazioni rispetto alle versioni visualizzate sullo schermo: entrambi indicano la finestra di allungamento _e_ i valori minimi e massimi reali dei dati, in modo che una legenda salvata sia ancora leggibile mesi dopo senza dover aprire il progetto.

### Avanzamento e risultati

L’esportazione dell’intero progetto richiede pochi minuti, quindi l’esecuzione invia aggiornamenti tramite un canale di avanzamento in tempo reale anziché bloccarsi:

* Una barra di avanzamento mostra `current / total` e il file in fase di scrittura
* Al termine, il riquadro riporta quante immagini sono state esportate, quante sono state saltate e il percorso della cartella di output
* Le immagini saltate sono elencate con il motivo (ne vengono mostrate fino a cinque, poi una riga &quot;+N altre&quot;). Il motivo più comune è un livello che non dispone dei canali necessari a questo indice
* Se **nessuna** immagine del progetto può utilizzare l’indice, l’esecuzione segnala un errore anziché lasciare una cartella vuota

È possibile eseguire una sola esportazione in sandbox alla volta. L’avvio di una seconda esportazione mentre una è in corso viene rifiutato con un messaggio chiaro, per evitare che due operazioni entrino in conflitto sullo stesso file di progetto.

### La griglia rileva l’esecuzione

Ogni esecuzione completata appare come un pulsante a sé stante nella [griglia delle immagini](image-grid.md) della barra degli strumenti, etichettato `<IndexName> <Index|LUT> <NNN>`. Ecco come si confrontano le esecuzioni: esportare due volte con gradienti o soglie diversi, quindi passare da un pulsante all’altro sulla griglia.

***

## Formule di indice personalizzate (Chloros+)

{% hint style="info" %}
**Dove crearle**: nella barra laterale della Sandbox o nelle**Impostazioni del progetto** prima dell’elaborazione. Entrambe scrivono nello stesso elenco a livello di progetto.
{% endhint %}

1. Apri il calcolatore delle formule personalizzate dal menu a tendina delle formule di indice (è necessario effettuare l’accesso con un abbonamento Chloros+ idoneo)
2. Scrivere la formula utilizzando i **simboli degli slot di banda** `x`, `y`, `z`, `a`, `b`, `c` — non i nomi delle bande
3. Operatori disponibili: `+`, `-`, `*`, `/`, `^` e `()` per il raggruppamento
4. Funzioni disponibili: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Assegnagli un nome e salvalo: apparirà in fondo al menu a tendina delle formule e potrai associare i suoi slot trascinando i cerchi dei canali, esattamente come per un preset integrato

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Le formule personalizzate sono disponibili solo nell’interfaccia grafica.** L’opzione CLI/SDK `--indices` espande i 22 nomi dei preset integrati e ignora silenziosamente tutto il resto, comprese le formule personalizzate. Per applicare in batch una formula personalizzata, configurarla nelle Impostazioni del progetto ed eseguire l’elaborazione, oppure utilizzare l’esportazione “Applica a tutte le immagini del progetto” della Sandbox.
{% endhint %}

***

## Risoluzione dei problemi

### “Questo livello non dispone dei canali richiesti da questo indice”

La formula richiede una posizione di canale che il livello corrente non possiede — ad esempio un indice a tre slot su un file a uno o due canali. Passa a un livello multibanda (riflettanza o debayering) oppure scegli un indice compatibile con il filtro della tua fotocamera.

### “Impossibile raggiungere il backend di elaborazione delle immagini”

Il backend non risponde. Controlla la scheda Log; se il backend si sta riavviando, Sandbox si ripristina automaticamente una volta che è di nuovo operativo.

### L’immagine non è cambiata quando ho trascinato un cerchio

La formula non è ancora completa. Una formula incompleta viene trattata come un normale stato di trascinamento in corso: non viene renderizzato nulla e non viene segnalato alcun errore. Compila ogni campo utilizzato dalla formula.

### L’intera immagine è di un unico colore

Probabilmente la finestra di ritaglio si trova ben al di fuori dei dati. Premi **AUTO**per allinearla al 2°/98° percentile, oppure attiva l’istogramma**INDICE** per vedere dove si trovano effettivamente i dati.

### I colori esportati non corrispondono a quelli che vedevo

Dovrebbero — il percorso di esportazione rispecchia fedelmente l’anteprima in tempo reale, compreso l’alfa in modalità di ritaglio, e la media a blocchi viene applicata _dopo_ la colorazione esattamente come fa il visualizzatore. Se differiscono, verifica che la dimensione del blocco GSD non sia cambiata tra la visualizzazione e l’esportazione.

***

## Passi successivi

* [**Livelli dell’immagine**](image-layers.md) — su quale livello eseguire un indice e cosa significano i suoi valori
* [**Apertura di un&#x27;immagine a schermo intero**](opening-an-image-full-screen.md) — la lettura del cursore, l’istogramma e il controllo del GSD in dettaglio
* [**Formule degli indici multispettrali**](../project-settings/multispectral-index-formulas.md) — ogni preimpostazione, su ogni superficie
* [**Impostazioni del progetto**](../project-settings/project-settings.md) — integrazione delle impostazioni individuate in una sessione di elaborazione
