# Impostazioni del progetto

La barra laterale Impostazioni del progetto <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> nella barra laterale di Chloros consentono di configurare tutti gli aspetti dell&#x27;elaborazione delle immagini, il rilevamento dei target di calibrazione, i calcoli degli indici multispettrali e le opzioni di esportazione per il progetto. Queste impostazioni vengono salvate insieme al progetto e possono essere salvate come modelli da riutilizzare in più progetti.

## Accesso alle impostazioni del progetto

Per accedere alle impostazioni del progetto:

1. Aprire un progetto in Chloros
2. Fare clic sulla scheda **Impostazioni del progetto**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> nella barra laterale sinistra
3. Il pannello delle impostazioni mostrerà tutte le opzioni di configurazione disponibili organizzate per categoria

***

## Rilevamento dei target

Queste impostazioni controllano il modo in cui Chloros rileva ed elabora i target di calibrazione nelle immagini.

### Area minima del campione di calibrazione (px)

* **Tipo**: Numero
* **Intervallo**: da 0 a 10.000 pixel
* **Impostazione predefinita**: 25 pixel
* **Descrizione**: Imposta l&#x27;area minima (in pixel) richiesta affinché una regione rilevata sia considerata un campione valido per il target di calibrazione. Valori più piccoli rileveranno target più piccoli, ma potrebbero aumentare i falsi positivi. Valori più grandi richiedono regioni target più grandi e più chiare per il rilevamento.
* **Quando regolare**:
  * Aumentare se si ottengono falsi rilevamenti su piccoli artefatti dell&#x27;immagine.
  * Diminuire se i target di calibrazione appaiono piccoli nelle immagini e non vengono rilevati.

### Raggruppamento minimo dei target (0-100)

* **Tipo**: Numero
* **Intervallo**: da 0 a 100
* **Impostazione predefinita**: 60
* **Descrizione**: Controlla la soglia di raggruppamento per raggruppare aree di colore simile durante il rilevamento dei target di calibrazione. Valori più alti richiedono che colori più simili vengano raggruppati insieme, con conseguente rilevamento dei target più conservativo. Valori più bassi consentono una maggiore variazione di colore all&#x27;interno di un gruppo di target.
* **Quando regolare**:
  * Aumentare se i target di calibrazione vengono suddivisi in più rilevamenti.
  * Diminuire se i target di calibrazione con variazioni di colore non vengono rilevati completamente.

***

## Elaborazione

Queste impostazioni controllano il modo in cui Chloros elabora e calibra le immagini.

### Correzione vignettatura

* **Tipo**: Casella di controllo
* **Impostazione predefinita**: Abilitata (selezionata)
* **Descrizione**: Applica la correzione della vignettatura per compensare l&#x27;oscuramento dell&#x27;obiettivo ai bordi delle immagini. La vignettatura è un fenomeno ottico comune in cui gli angoli e i bordi di un&#x27;immagine appaiono più scuri rispetto al centro a causa delle caratteristiche dell&#x27;obiettivo.
* **Quando disattivare**: disattivare solo se la combinazione fotocamera/obiettivo ha già applicato la correzione della vignettatura o se si desidera correggere manualmente la vignettatura in post-elaborazione.

### Calibrazione della riflettanza / bilanciamento del bianco

* **Tipo**: casella di controllo
* **Impostazione predefinita**: Abilitato (selezionato)
* **Descrizione**: abilita la calibrazione automatica della riflettanza utilizzando i target di calibrazione rilevati nelle immagini. Questo normalizza i valori di riflettanza in tutto il set di dati e garantisce misurazioni coerenti indipendentemente dalle condizioni di illuminazione.
* **Quando disabilitare**: disabilitare solo se si desidera elaborare immagini raw non calibrate o se si utilizza un flusso di lavoro di calibrazione diverso.

### Metodo Debayer

* **Tipo**: selezione a discesa
* **Opzioni**:
  * Alta qualità (più veloce) - Attualmente l&#x27;unica opzione disponibile
* **Impostazione predefinita**: Alta qualità (più veloce)
* **Descrizione**: Seleziona l&#x27;algoritmo di demosaicizzazione utilizzato per convertire i dati grezzi del sensore con pattern Bayer in immagini a colori. Il metodo &quot;Alta qualità (più veloce)&quot; offre un equilibrio ottimale tra velocità di elaborazione e qualità dell&#x27;immagine.
* **Nota**: nelle versioni future di Chloros potrebbero essere aggiunti ulteriori metodi di debayer.

### Intervallo minimo di ricalibrazione

* **Tipo**: Numero
* **Intervallo**: da 0 a 3.600 secondi
* **Impostazione predefinita**: 0 secondi
* **Descrizione**: Imposta l&#x27;intervallo di tempo minimo (in secondi) tra l&#x27;utilizzo dei target di calibrazione. Se impostato su 0, Chloros utilizzerà tutti i target di calibrazione rilevati. Se impostato su un valore più alto, Chloros utilizzerà solo i target di calibrazione separati da almeno questo numero di secondi, riducendo il tempo di elaborazione per i set di dati con acquisizioni frequenti dei target di calibrazione.
* **Quando regolare**:
  * Impostare su 0 per la massima precisione di calibrazione quando le condizioni di illuminazione variano.
  * Aumentare (ad esempio, a 60-300 secondi) per un&#x27;elaborazione più rapida quando l&#x27;illuminazione è costante e si dispone di immagini di target di calibrazione frequenti.

### Offset fuso orario sensore di luce

* **Tipo**: Numero
* **Intervallo**: da -12 a +12 ore
* **Impostazione predefinita**: 0 ore
* **Descrizione**: specifica l&#x27;offset del fuso orario (in ore rispetto all&#x27;UTC) per i timestamp dei dati del sensore di luce. Viene utilizzato durante l&#x27;elaborazione dei file di dati PPK (Post-Processed Kinematic) per garantire la corretta sincronizzazione temporale tra le acquisizioni delle immagini e i dati GPS.
* **Quando regolare**: impostare questo valore sull&#x27;offset del fuso orario locale se i dati PPK utilizzano l&#x27;ora locale invece dell&#x27;UTC. Ad esempio:
  * Ora del Pacifico: -8 o -7 (a seconda dell&#x27;ora legale)
  * Ora della costa orientale: -5 o -4 (a seconda dell&#x27;ora legale)
  * Ora dell&#x27;Europa centrale: +1 o +2 (a seconda dell&#x27;ora legale)

### Applica correzioni PPK

* **Tipo**: Casella di controllo
* **Impostazione predefinita**: Disabilitato (deselezionato)
* **Descrizione**: Abilita l&#x27;uso delle correzioni cinematiche post-elaborate (PPK) dai registratori DAQ MAPIR contenenti un GPS (GNSS). Quando è abilitato, Chloros utilizzerà tutti i file di log .daq contenenti dati sui pin di esposizione nella directory del progetto e applicherà correzioni di geolocalizzazione precise alle immagini.
* **Requisiti**: nella directory del progetto deve essere presente un file di log .daq con voci relative ai pin di esposizione
* **Quando abilitare**: si consiglia di abilitare sempre la correzione PPK se nel file di log .daq sono presenti voci relative al feedback di esposizione.

### Pin di esposizione 1

* **Tipo**: selezione a discesa
* **Visibilità**: visibile solo quando &quot;Applica correzioni PPK&quot; è abilitato E i dati di esposizione sono disponibili per il Pin 1
* **Opzioni**:
  * Nomi dei modelli di fotocamera rilevati nel progetto
  * &quot;Non utilizzare&quot; - Ignora questo pin di esposizione
* **Impostazione predefinita**: selezionata automaticamente in base alla configurazione del progetto
* **Descrizione**: assegna una fotocamera specifica al Pin di esposizione 1 per la sincronizzazione temporale PPK. Il pin di esposizione registra il momento esatto in cui viene attivato l&#x27;otturatore della fotocamera, fondamentale per una geolocalizzazione PPK accurata.
* **Comportamento di selezione automatica**:
  * Fotocamera singola + pin singolo: seleziona automaticamente la fotocamera
  * Telecamera singola + due pin: il pin 1 viene assegnato automaticamente alla telecamera
  * Telecamere multiple: è necessaria la selezione manuale

### Pin di esposizione 2

* **Tipo**: selezione a discesa
* **Visibilità**: visibile solo quando &quot;Applica correzioni PPK&quot; è abilitato E i dati di esposizione sono disponibili per il pin 2
* **Opzioni**:
  * Nomi dei modelli di telecamera rilevati nel progetto
  * &quot;Non utilizzare&quot;: ignora questo pin di esposizione
* **Impostazione predefinita**: selezionato automaticamente in base alla configurazione del progetto
* **Descrizione**: assegna una fotocamera specifica al pin di esposizione 2 per la sincronizzazione temporale PPK quando si utilizza una configurazione a doppia fotocamera.
* **Comportamento di selezione automatica**:
  * Fotocamera singola + pin singolo: il pin 2 viene automaticamente impostato su &quot;Non utilizzare&quot;
  * Telecamera singola + due pin: il pin 2 viene automaticamente impostato su &quot;Non utilizzare&quot;
  * Telecamere multiple: è necessaria la selezione manuale
* **Nota**: la stessa telecamera non può essere assegnata contemporaneamente al pin 1 e al pin 2.

***

## Indice

Queste impostazioni consentono di configurare gli indici multispettrali per l&#x27;analisi e la visualizzazione.

### Aggiungi indice

* **Tipo**: pannello di configurazione indici speciali
* **Descrizione**: apre un pannello interattivo in cui è possibile selezionare e configurare gli indici multispettrali della vegetazione (NDVI, NDRE, EVI, ecc.) da calcolare durante l&#x27;elaborazione delle immagini. È possibile aggiungere più indici, ciascuno con le proprie impostazioni di visualizzazione.
* **Indici disponibili**: il sistema include oltre 30 indici multispettrali predefiniti, tra cui:
  * NDVI (Indice di vegetazione normalizzato)
  * NDRE (Differenza normalizzata RedEdge)
  * EVI (Indice di vegetazione potenziato)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * E molti altri (vedere [Formule indice multispettrale](multispectral-index-formulas.md) per l&#x27;elenco completo)
* **Caratteristiche**:
  * Selezionare tra formule indice predefinite
  * Configurare gradienti di colore di visualizzazione (LUT - Look-Up Tables)
  * Impostare valori soglia per l&#x27;analisi
  * Creare formule indice personalizzate

### Formule personalizzate (Funzionalità Chloros+)

* **Tipo**: Matrice di definizioni di formule personalizzate
* **Descrizione**: Consente di creare e salvare formule di indici multispettrali personalizzate utilizzando la matematica delle bande. Le formule personalizzate vengono salvate con le impostazioni del progetto e possono essere utilizzate proprio come gli indici integrati.
* **Come creare**:
  1. Nel pannello di configurazione dell&#x27;indice, cercare l&#x27;opzione formula personalizzata
  2. Definire la formula utilizzando gli identificatori di banda (ad esempio, NIR, Red, Green, Blue)
  3. Salvare la formula con un nome descrittivo
* **Sintassi della formula**: sono supportate le operazioni matematiche standard, tra cui:
  * Aritmetica: `+`, `-`, `*`, `/`
  * Parentesi per l&#x27;ordine delle operazioni
  * Riferimenti di banda: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Esportazione

Queste impostazioni controllano il formato e la qualità delle immagini elaborate esportate.

### Formato immagine calibrato

* **Tipo**: selezione a tendina
* **Opzioni**:
  * **TIFF (16 bit)** - Formato TIFF a 16 bit non compresso
  * **TIFF (32 bit, percentuale)** - TIFF a 32 bit in virgola mobile con valori di riflettanza espressi in percentuale
  * **PNG (8 bit)** - Formato PNG compresso a 8 bit
  * **JPG (8 bit)** - Formato JPEG compresso a 8 bit
* **Predefinito**: TIFF (16 bit)
* **Descrizione**: seleziona il formato file per il salvataggio delle immagini elaborate e calibrate.
* **Formati consigliati**:
  * **TIFF (16 bit)**: consigliato per analisi scientifiche e flussi di lavoro professionali. Conserva la massima qualità dei dati senza artefatti di compressione. Ideale per analisi multispettrali e ulteriori elaborazioni in software GIS.
  * **TIFF (32 bit, percentuale)**: ideale per flussi di lavoro che richiedono valori di riflettanza in percentuale (0-100%). Offre la massima precisione per le misurazioni radiometriche.
  * **PNG (8 bit)**: ideale per la visualizzazione sul web e la visualizzazione generale. File di dimensioni inferiori con compressione senza perdita di dati, ma gamma dinamica ridotta.
  * **JPG (8 bit)**: file di dimensioni minime, ideale solo per anteprime e visualizzazione sul web. Utilizza una compressione con perdita di dati che non è adatta all&#x27;analisi scientifica.

***

## Salva modello di progetto

Questa funzione consente di salvare le impostazioni del progetto corrente come modello riutilizzabile.

* **Tipo**: Inserimento di testo + pulsante Salva
* **Descrizione**: Immettere un nome descrittivo per il modello di impostazioni e fare clic sull&#x27;icona Salva. Il modello memorizzerà tutte le impostazioni del progetto corrente (rilevamento del target, opzioni di elaborazione, indici e formato di esportazione) per un facile riutilizzo in progetti futuri.
* **Casi d&#x27;uso**:
  * Creare modelli per diversi sistemi di telecamere (RGB, multispettrale, NIR)
  * Salvare configurazioni standard per tipi di colture specifici o flussi di lavoro di analisi
  * Condividere impostazioni coerenti all&#x27;interno di un team
* **Come utilizzare**:
  1. Configurare tutte le impostazioni desiderate per il progetto
  2. Inserire un nome per il modello (ad esempio, &quot;RedEdge Survey3 NDVI Standard&quot;)
  3. Fare clic sull&#x27;icona di salvataggio
  4. Il modello può ora essere caricato durante la creazione di nuovi progetti

***

## Salva cartella progetto

Questa impostazione specifica dove vengono salvati i nuovi progetti per impostazione predefinita.

* **Tipo**: Visualizzazione del percorso della directory + pulsante Modifica
* **Predefinito**: `C:\Users\[Username]\Chloros Projects`
* **Descrizione**: Mostra la directory predefinita corrente in cui vengono creati i nuovi progetti Chloros. Fare clic sull&#x27;icona di modifica per selezionare una directory diversa.
* **Quando modificare**:
  * Impostare su un&#x27;unità di rete per la collaborazione in team
  * Passare a un&#x27;unità con più spazio di archiviazione per set di dati di grandi dimensioni
  * Organizzare i progetti per anno, cliente o tipo di progetto in cartelle diverse
* **Nota**: la modifica di questa impostazione ha effetto solo sui NUOVI progetti. I progetti esistenti rimangono nella loro posizione originale.

***

## Persistenza delle impostazioni

Tutte le impostazioni del progetto vengono salvate automaticamente con il file di progetto (formato di progetto `.mapir`). Quando riapri un progetto, tutte le impostazioni vengono ripristinate esattamente come le hai lasciate.

### Gerarchia delle impostazioni

Le impostazioni vengono applicate nel seguente ordine:

1. **Impostazioni predefinite di sistema** - Impostazioni predefinite integrate definite da Chloros
2. **Impostazioni del modello** - Se si carica un modello durante la creazione di un progetto
3. **Impostazioni del progetto salvate** - Impostazioni salvate con il file di progetto
4. **Regolazioni manuali** - Qualsiasi modifica apportata durante la sessione corrente

### Impostazioni ed elaborazione delle immagini

La maggior parte delle modifiche alle impostazioni (in particolare nelle categorie Elaborazione ed Esportazione) attiverà una rielaborazione delle immagini per riflettere le nuove impostazioni. Tuttavia, alcune impostazioni sono &quot;solo esportazione&quot; e non richiedono una rielaborazione immediata:

* Salva modello di progetto
* Directory di lavoro
* Formato immagine calibrato (si applica durante l&#x27;esportazione)

***

## Best practice

1. **Inizia con le impostazioni predefinite**: le impostazioni predefinite funzionano bene per la maggior parte dei sistemi di telecamere MAPIR e dei flussi di lavoro tipici.
2. **Crea modelli**: una volta ottimizzate le impostazioni per un flusso di lavoro o una telecamera specifici, salvali come modello per garantire la coerenza tra i progetti.
3. **Eseguire un test prima dell&#x27;elaborazione completa**: quando si sperimentano nuove impostazioni, eseguire un test su un piccolo sottoinsieme di immagini prima di elaborare l&#x27;intero set di dati.
4. **Documentare le impostazioni**: utilizzare nomi di modelli descrittivi che indichino il sistema di telecamere, il tipo di elaborazione e l&#x27;uso previsto (ad esempio, &quot;Survey3\_RGB\_NDVI\_Agricoltura&quot;).
5. **Selezione del formato di esportazione**: scegli il formato di esportazione in base all&#x27;uso finale:
   * Analisi scientifica → TIFF (16 bit o 32 bit)
   * Elaborazione GIS → TIFF (16 bit)
   * Visualizzazione rapida → PNG (8 bit)
   * Condivisione web → JPG (8 bit)

***

Per ulteriori informazioni sugli indici multispettrali in Chloros, consultare la pagina [Formule degli indici multispettrali](multispectral-index-formulas.md).
