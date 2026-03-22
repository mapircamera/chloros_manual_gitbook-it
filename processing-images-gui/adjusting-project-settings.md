# Modifica delle impostazioni del progetto

Prima di elaborare le immagini, è importante configurare le impostazioni del progetto in modo che corrispondano alle esigenze del proprio flusso di lavoro. Il pannello Impostazioni progetto <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> offrono un controllo completo su calibrazione, opzioni di elaborazione, indici multispettrali e formati di esportazione.

## Accesso alle impostazioni del progetto

1. Apri il tuo progetto in Chloros
2. Fai clic sull&#x27;icona **Impostazioni del progetto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> nella barra laterale sinistra
3. Il pannello Impostazioni progetto mostra tutte le opzioni di configurazione

{% hint style="info" %}
**Le impostazioni vengono salvate automaticamente** con il progetto. Quando si riapre un progetto, tutte le impostazioni vengono ripristinate.
{% endhint %}

***

## Configurazione rapida per flussi di lavoro comuni

### Impostazioni predefinite (consigliate per la maggior parte degli utenti)

Per i flussi di lavoro tipici delle fotocamere MAPIR e Survey3, le impostazioni predefinite funzionano bene:

* ✅ **Correzione della vignettatura**: Abilitata
* ✅ **Calibrazione della riflettanza**: Abilitata (richiede immagini dei target MAPIR)
* ✅ **Metodo Debayer**: Standard (Veloce, Qualità media)
* ✅ **Formato di esportazione**: TIFF (16 bit)

Basta importare le immagini e avviare l&#x27;elaborazione con queste impostazioni predefinite.

***

## Panoramica delle impostazioni di progetto

Il pannello delle impostazioni di progetto è organizzato in diverse categorie. Di seguito è riportato un riepilogo di ciascuna sezione. Per la documentazione completa, consultare [Impostazioni di progetto](../project-settings/project-settings.md).

### Rilevamento dei target

Controlla il modo in cui Chloros identifica i target di calibrazione nelle immagini.

**Impostazioni chiave:*** **Area minima del campione di calibrazione**: soglia di dimensione per il rilevamento dei target (impostazione predefinita: 25 pixel)
* **Clustering minimo dei target**: soglia di similarità per il raggruppamento delle regioni target (impostazione predefinita: 60)**Quando regolare:**

* Aumentare l&#x27;area del campione se si ottengono rilevamenti errati
* Ridurla se i target non vengono rilevati
* Regolare il raggruppamento se i target vengono suddivisi in più rilevamenti

### Elaborazione

Opzioni principali di elaborazione delle immagini e calibrazione.

**Impostazioni chiave:*** **Correzione della vignettatura**: Compensa l&#x27;oscuramento dell&#x27;obiettivo ai bordi ✅ Consigliato
* **Calibrazione della riflettanza**: Normalizza i valori utilizzando i target di calibrazione ✅ Consigliato
* **Metodo Debayer**: Algoritmo per convertire il formato RAW in multispettrale a 3 canali
* **Intervallo minimo di ricalibrazione**: Tempo tra un utilizzo e l&#x27;altro dei target di calibrazione (0 = usa tutti)**Impostazioni avanzate:*** **Offset fuso orario sensore di luce**: Per la sincronizzazione temporale PPK (impostazione predefinita: 0)
* **Applica correzioni PPK**: Utilizza i dati GPS/pin di esposizione dai file .daq
* **Pin di esposizione 1/2**: Assegna le fotocamere ai pin di esposizione per configurazioni a doppia fotocamera

### Metodo di debayering

Attualmente offriamo 2 metodi di debayering in Chloros:

#### Standard (Veloce, Qualità media)

Il debayering standard è veloce ma mostra rumore cromatico, con immagini meno accurate e più rumorose.

#### Texture Aware (Lento, Massima qualità) \[Solo Chloros+]

Texture Aware usa un debayering di alta qualità sensibile ai bordi combinato con un modello di denoising AI/ML che rimuove quasi tutto il rumore. Il modello Texture Aware richiede memoria GPU (VRAM) per funzionare. Si consiglia di utilizzarlo quando si dispone di &gt;4 GB di VRAM per un&#x27;elaborazione più veloce.

### Indice (Indici multispettrali)

Configura quali indici di vegetazione calcolare ed esportare.

**Come aggiungere indici:**

1. Clicca sul pulsante**&quot;Aggiungi indice&quot;**

2. Selezionare un indice dal menu a tendina (NDVI, NDRE, GNDVI, ecc.)
3. Configurare le impostazioni di visualizzazione (colori LUT, intervalli di valori)
4. Aggiungere più indici secondo necessità

**Indici più diffusi:*** **NDVI**: Stato di salute generale della vegetazione (il più comune)
* **NDRE**: Rilevamento precoce dello stress con RedEdge
* **GNDVI**: Sensibile alla concentrazione di clorofilla
* **OSAVI**: Funziona bene con il suolo visibile
* **EVI**: Aree con indice di area fogliare elevato (LAI)**Formule personalizzate (solo Chloros+):**

* Crea formule personalizzate per indici multispettrali
* Usa la matematica delle bande con tutti i canali dell&#x27;immagine
* Salva le formule personalizzate per riutilizzarle

Per tutti gli indici e le formule disponibili, vedi [Formule degli indici multispettrali](../project-settings/multispectral-index-formulas.md).

### Esporta

Controlla il formato e la qualità del file di output.

**Formati disponibili:*** **TIFF (16 bit)**: Consigliato per GIS e analisi scientifiche (intervallo 0-65.535)
* **TIFF (32 bit, percentuale)**: Valori di riflettanza in virgola mobile (intervallo 0,0-1,0)
* **PNG (8 bit)**: compressione senza perdita di dati per la visualizzazione (intervallo 0-255)
* **JPG (8 bit)**: file più piccoli, compressione con perdita di dati (intervallo 0-255)***

## Salvataggio e caricamento delle impostazioni

### Salva modello di progetto

Crea modelli riutilizzabili per flussi di lavoro coerenti:

1. Configura tutte le impostazioni desiderate nel pannello Impostazioni progetto
2. Scorri fino alla sezione **&quot;Salva modello di progetto&quot;** in fondo alla pagina
3. Inserisci un nome descrittivo per il modello (ad es. &quot;Survey3N\_RGN\_Agricoltura&quot;)
4. Fai clic sull&#x27;icona di salvataggio

**Vantaggi:**

* Applicare impostazioni identiche su più progetti
* Condividere le configurazioni con i membri del team
* Mantenere la coerenza per sondaggi ripetuti

### Caricare il modello su un nuovo progetto

Quando si crea un nuovo progetto:

1. Selezionare **&quot;Nuovo progetto&quot;** dal menu principale
2. Scegliere l&#x27;opzione **&quot;Carica da modello&quot;**

3. Selezionare il modello salvato
4. Tutte le impostazioni vengono applicate automaticamente

### Directory di lavoro

L&#x27;impostazione **&quot;Salva cartella progetto&quot;** specifica dove vengono creati i nuovi progetti per impostazione predefinita:

* **Posizione predefinita**: `C:\Users\[Username]\Chloros Projects`
* **Cambia posizione**: clicca sull&#x27;icona di modifica e seleziona una nuova cartella
* **Quando modificare**:
  * Unità di rete per la collaborazione in team
  * Un&#x27;unità diversa con più spazio di archiviazione
  * Struttura di cartelle organizzata per anno/cliente

***

## Configurazione PPK (Post-Processed Kinematic)

Se si utilizzano registratori DAQ MAPIR con GPS per una geolocalizzazione precisa:

### Prerequisiti

* DAQ MAPIR con modulo GPS (GNSS)
* File di log .daq con voci relative ai pin di esposizione
* Fotocamera collegata ai pin di esposizione del DAQ durante la sessione di acquisizione

### Passaggi di configurazione

1. Inserire il file di log .daq nella cartella del progetto
2. In Impostazioni progetto, abilitare la casella di controllo **&quot;Applica correzioni PPK&quot;**

3. Impostare**&quot;Offset fuso orario sensore di luce&quot;** se necessario (impostazione predefinita: 0 per UTC)
4. Assegnare le fotocamere ai pin di esposizione:
   * **Fotocamera singola**: assegnata automaticamente al Pin 1
   * **Fotocamere doppie**: assegnare manualmente ciascuna fotocamera al pin corretto**Assegnazione dei pin di esposizione:*** **Pin di esposizione 1**: selezionare il modello di fotocamera dal menu a tendina
* **Pin di esposizione 2**: selezionare la seconda fotocamera o &quot;Non utilizzare&quot;
* Non è possibile assegnare la stessa fotocamera a entrambi i pin

{% hint style="warning" %}
**Importante**: i pin di esposizione devono essere correttamente assegnati alle rispettive fotocamere. Un&#x27;assegnazione errata comporterà dati di geolocalizzazione errati.
{% endhint %}

***

## Scenari avanzati

### Progetti multi-telecamera

Quando si elaborano immagini provenienti da più telecamere MAPIR in un unico progetto:

1. Chloros rileva automaticamente il modello di ciascuna telecamera
2. A ciascuna telecamera viene assegnato il profilo di elaborazione appropriato
3. PPK: assegnare manualmente ogni fotocamera al pin di esposizione corretto
4. Tutte le fotocamere utilizzano lo stesso formato di esportazione e gli stessi indici

**Esempio**: Survey3W RGN + Survey3N OCN rig a doppia fotocamera

### Rilevamenti time-lapse o su più date

Per rilevamenti ripetuti della stessa area nel corso del tempo:

1. Creare un modello con le impostazioni standard
2. Utilizzare una configurazione del target di calibrazione coerente in ogni sessione
3. Elaborare ogni data come un progetto separato
4. Utilizzare impostazioni identiche per ottenere risultati comparabili
5. Esportare nello stesso formato per l&#x27;analisi temporale

### Set di dati di grandi dimensioni

Per progetti con molte immagini (oltre 500):

* Valutare la possibilità di suddividere il lavoro in progetti più piccoli per data o area
* Utilizzare l&#x27;elaborazione parallela Chloros+ per ottenere risultati più rapidi
* Valutare l&#x27;utilizzo di CLI o API per l&#x27;automazione in batch
* Regolare l&#x27;intervallo minimo di ricalibrazione per ridurre il tempo di rilevamento dei target

***

## Verifica delle impostazioni

Prima di iniziare l&#x27;elaborazione, controlla queste impostazioni chiave:

* [ ] Modello di fotocamera rilevato correttamente nel File Browser
* [ ] Correzione della vignettatura abilitata
* [ ] Calibrazione della riflettanza abilitata
* [ ] Almeno un&#x27;immagine del target di calibrazione importata
* [ ] Indici multispettrali desiderati aggiunti
* [ ] Formato di esportazione appropriato per il tuo flusso di lavoro
* [ ] Impostazioni PPK configurate (se si utilizza .daq con eventi di esposizione)

***

## Passaggi successivi

Una volta configurate le impostazioni:

1. **Contrassegnare le immagini target di calibrazione** - Vedere [Scelta delle immagini target](choosing-target-images.md)
2. **Avviare l&#x27;elaborazione** - Vedere [Avvio dell&#x27;elaborazione](starting-the-processing.md)
3. **Monitorare lo stato di avanzamento** - Vedi [Monitoraggio dell&#x27;elaborazione](monitoring-the-processing.md)

Per i dettagli completi su tutte le impostazioni disponibili, consultare la documentazione di riferimento [Impostazioni del progetto](../project-settings/project-settings.md).
