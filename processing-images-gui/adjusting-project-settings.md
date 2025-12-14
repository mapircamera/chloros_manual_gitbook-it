# Regolazione delle impostazioni del progetto

Prima di elaborare le immagini, è importante configurare le impostazioni del progetto in modo che corrispondano ai requisiti del flusso di lavoro. Il pannello Impostazioni progetto <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> offre un controllo completo su calibrazione, opzioni di elaborazione, indici multispettrali e formati di esportazione.

## Accesso alle impostazioni del progetto

1. Aprire il progetto in Chloros
2. Fare clic sull&#x27;icona **Impostazioni del progetto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> nella barra laterale sinistra
3. Il pannello Impostazioni del progetto mostra tutte le opzioni di configurazione

{% hint style=&quot;info&quot; %}
**Le impostazioni vengono salvate automaticamente** con il progetto. Quando si riapre un progetto, tutte le impostazioni vengono ripristinate.
{% endhint %}

***

## Configurazione rapida per flussi di lavoro comuni

### Impostazioni predefinite (consigliate per la maggior parte degli utenti)

Per i flussi di lavoro tipici della fotocamera MAPIR Survey3, le impostazioni predefinite funzionano bene:

* ✅ **Correzione vignettatura**: Abilitata
* ✅ **Calibrazione riflettanza**: Abilitata (richiede immagini di bersagli MAPIR)
* ✅ **Metodo Debayer**: Alta qualità (più veloce)
* ✅ **Formato di esportazione**: TIFF (16 bit)

È sufficiente importare le immagini e avviare l&#x27;elaborazione con queste impostazioni predefinite.

***

## Panoramica delle impostazioni di progetto

Il pannello Impostazioni progetto è organizzato in diverse categorie. Di seguito è riportato un riepilogo di ciascuna sezione. Per la documentazione completa, vedere [Impostazioni progetto](../project-settings/project-settings.md).

### Rilevamento target

Controlla il modo in cui Chloros identifica i target di calibrazione nelle immagini.

**Impostazioni chiave:**

* **Area minima del campione di calibrazione**: soglia di dimensione per il rilevamento dei target (impostazione predefinita: 25 pixel)
* **Clustering minimo dei target**: soglia di similarità per il raggruppamento delle regioni target (impostazione predefinita: 60)

**Quando regolare:**

* Aumentare l&#x27;area del campione se si ottengono rilevamenti falsi
* Diminuire se i target non vengono rilevati
* Regolare il clustering se i target vengono suddivisi in più rilevamenti

### Elaborazione

Opzioni principali di elaborazione delle immagini e calibrazione.

**Impostazioni chiave:**

* **Correzione vignettatura**: compensa l&#x27;oscuramento dell&#x27;obiettivo ai bordi ✅ Consigliato
* **Calibrazione riflettanza**: normalizza i valori utilizzando i target di calibrazione ✅ Consigliato
* **Metodo Debayer**: algoritmo per la conversione da RAW a multispettrale a 3 canali
* **Intervallo minimo di ricalibrazione**: tempo tra l&#x27;utilizzo dei target di calibrazione (0 = usa tutti)

**Impostazioni avanzate:**

* **Offset fuso orario sensore di luce**: per la sincronizzazione temporale PPK (impostazione predefinita: 0)
* **Applica correzioni PPK**: utilizza i dati GPS/pin di esposizione dai file .daq
* **Pin di esposizione 1/2**: assegna le fotocamere ai pin di esposizione per configurazioni a doppia fotocamera

### Indice (indici multispettrali)

Configura quali indici di vegetazione calcolare ed esportare.

**Come aggiungere indici:**

1. Fai clic sul pulsante **&quot;Aggiungi indice&quot;**
2. Selezionare un indice dal menu a discesa (NDVI, NDRE, GNDVI, ecc.)
3. Configurare le impostazioni di visualizzazione (colori LUT, intervalli di valori)
4. Aggiungere più indici secondo necessità

**Indici popolari:**

* **NDVI**: Stato di salute generale della vegetazione (il più comune)
* **NDRE**: Rilevamento precoce dello stress con RedEdge
* **GNDVI**: Sensibile alla concentrazione di clorofilla
* **OSAVI**: Funziona bene con il suolo visibile
* **EVI**: Regioni con indice di area fogliare elevato (LAI)

**Formule personalizzate (solo Chloros+):**

* Creazione di formule personalizzate per indici multispettrali
* Utilizzo di calcoli matematici su tutte le bande dell&#x27;immagine
* Salvataggio di formule personalizzate per il riutilizzo

Per tutti gli indici e le formule disponibili, vedere [Formule per indici multispettrali](../project-settings/multispectral-index-formulas.md).

### Esportazione

Controlla il formato e la qualità del file di output.

**Formati disponibili:**

* **TIFF (16 bit)**: consigliato per GIS e analisi scientifiche (intervallo 0-65.535)
* **TIFF (32 bit, percentuale)**: valori di riflettanza in virgola mobile (intervallo 0,0-1,0)
* **PNG (8 bit)**: compressione senza perdita di dati per la visualizzazione (intervallo 0-255)
* **JPG (8 bit)**: file più piccoli, compressione con perdita di dati (intervallo 0-255)

***

## Salvataggio e caricamento delle impostazioni

### Salvataggio del modello di progetto

Creare modelli riutilizzabili per flussi di lavoro coerenti:

1. Configurare tutte le impostazioni desiderate nel pannello Impostazioni progetto
2. Scorrere fino alla sezione **&quot;Salva modello di progetto&quot;** nella parte inferiore
3. Immettere un nome descrittivo per il modello (ad esempio, &quot;Survey3N\_RGN\_Agricoltura&quot;)
4. Fare clic sull&#x27;icona di salvataggio

**Vantaggi:**

* Applicare impostazioni identiche a più progetti
* Condividere le configurazioni con i membri del team
* Mantenere la coerenza per i sondaggi ripetuti

### Caricare il modello su un nuovo progetto

Quando si crea un nuovo progetto:

1. Selezionare **&quot;Nuovo progetto&quot;** dal menu principale
2. Scegliere l&#x27;opzione **&quot;Carica da modello&quot;**
3. Selezionare il modello salvato
4. Tutte le impostazioni vengono applicate automaticamente

### Directory di lavoro

L&#x27;impostazione **&quot;Salva cartella progetto&quot;** specifica dove vengono creati i nuovi progetti per impostazione predefinita:

* **Posizione predefinita**: `C:\Users\[Username]\Chloros Projects`
* **Modifica posizione**: fare clic sull&#x27;icona Modifica e selezionare una nuova cartella
* **Quando modificare**:
  * Unità di rete per la collaborazione in team
  * Unità diversa con più spazio di archiviazione
  * Struttura di cartelle organizzata per anno/cliente

***

## Configurazione PPK (Post-Processed Kinematic)

Se si utilizzano registratori DAQ MAPIR con GPS per una geolocalizzazione precisa:

### Prerequisiti

* DAQ MAPIR con modulo GPS (GNSS)
* File di log .daq con voci dei pin di esposizione
* Fotocamera collegata ai pin di esposizione DAQ durante la sessione di acquisizione

### Passaggi di configurazione

1. Posizionare il file di log .daq nella cartella del progetto
2. In Impostazioni progetto, abilitare la casella di controllo **&quot;Applica correzioni PPK&quot;**
3. Impostare **&quot;Offset fuso orario sensore di luce&quot;** se necessario (impostazione predefinita: 0 per UTC)
4. Assegnare le fotocamere ai pin di esposizione:
   * **Fotocamera singola**: Assegnata automaticamente al pin 1
   * **Doppia fotocamera**: assegnare manualmente ciascuna fotocamera al pin corretto

**Assegnazione dei pin di esposizione:**

* **Pin di esposizione 1**: selezionare il modello di fotocamera dal menu a tendina
* **Pin di esposizione 2**: selezionare la seconda fotocamera o &quot;Non utilizzare&quot;
* La stessa fotocamera non può essere assegnata a entrambi i pin

{% suggerimento style=&quot;warning&quot; %}
**Importante**: i pin di esposizione devono essere assegnati correttamente alle rispettive telecamere. Un&#x27;assegnazione errata comporterà dati di geolocalizzazione errati.
{% fine suggerimento %}

***

## Scenari avanzati

### Progetti multi-camera

Quando si elaborano immagini provenienti da più fotocamere MAPIR in un unico progetto:

1. Chloros rileva automaticamente il modello di ciascuna fotocamera
2. A ciascuna fotocamera viene assegnato il profilo di elaborazione appropriato
3. PPK: assegnare manualmente a ciascuna fotocamera il pin di esposizione corretto
4. Tutte le fotocamere utilizzano lo stesso formato di esportazione e gli stessi indici

**Esempio**: Survey3W RGN + Survey3N OCN doppio supporto per fotocamera

### Rilevamenti time-lapse o multi-data

Per rilevamenti ripetuti della stessa area nel tempo:

1. Creare un modello con le impostazioni standard
2. Utilizzare una configurazione di calibrazione coerente per ogni sessione
3. Elaborare ogni data come un progetto separato
4. Utilizzare impostazioni identiche per ottenere risultati comparabili
5. Esportare nello stesso formato per l&#x27;analisi temporale

### Set di dati di grandi dimensioni

Per progetti con molte immagini (oltre 500):

* Valutare la possibilità di suddividere il progetto in progetti più piccoli per data o area
* Utilizzare l&#x27;elaborazione parallela Chloros+ per ottenere risultati più rapidi
* Valutare l&#x27;utilizzo di CLI o API per l&#x27;automazione in batch
* Regolare l&#x27;intervallo minimo di ricalibrazione per ridurre il tempo di rilevamento del target

***

## Verifica delle impostazioni

Prima di iniziare l&#x27;elaborazione, controlla queste impostazioni chiave:

* [ ] Modello di fotocamera rilevato correttamente nel File Browser
* [ ] Correzione vignetta abilitata
* [ ] Calibrazione riflettanza abilitata
* [ ] Importata almeno un&#x27;immagine target di calibrazione
* [ ] Indici multispettrali desiderati aggiunti
* [ ] Formato di esportazione appropriato per il tuo flusso di lavoro
* [ ] Impostazioni PPK configurate (se si utilizza .daq con eventi di esposizione)

***

## Passaggi successivi

Una volta configurate le impostazioni:

1. **Contrassegnare le immagini target di calibrazione** - Vedere [Scelta delle immagini target](choosing-target-images.md)
2. **Avviare l&#x27;elaborazione** - Vedere [Avvio dell&#x27;elaborazione](starting-the-processing.md)
3. **Monitorare lo stato di avanzamento** - Vedere [Monitoraggio dell&#x27;elaborazione](monitoring-the-processing.md)

Per informazioni complete su tutte le impostazioni disponibili, consultare la documentazione di riferimento [Impostazioni di progetto](../project-settings/project-settings.md).
