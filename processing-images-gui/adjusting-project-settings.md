# Modifica delle impostazioni del progetto

Prima di elaborare le immagini, è importante configurare le impostazioni del progetto in modo che corrispondano alle esigenze del proprio flusso di lavoro. Il pannello “Impostazioni del progetto” <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> offre un controllo completo su calibrazione, opzioni di elaborazione, indici multispettrali e formati di esportazione.

## Come accedere alle impostazioni del progetto

1. Apri il tuo progetto in Chloros
2. Fai clic sull’icona **Impostazioni del progetto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> nella barra laterale sinistra
3. Il pannello “Impostazioni del progetto” mostra tutte le opzioni di configurazione

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Il pannello **Impostazioni progetto** — Visualizzazione, rilevamento dei target ed elaborazione</p></figcaption></figure>{% hint style="info" %}**Le impostazioni vengono salvate automaticamente** insieme al progetto. Quando si riapre un progetto, tutte le impostazioni vengono ripristinate.
{% endhint %}

***

## Configurazione rapida per flussi di lavoro comuni

### Impostazioni predefinite (consigliate per la maggior parte degli utenti)

Le impostazioni predefinite funzionano bene per i flussi di lavoro tipici di Survey3 e LATTICE:

* ✅ **Correzione della vignettatura**: Abilitata
* ✅ **Calibrazione della riflettanza / bilanciamento del bianco**: Abilitata (utilizza i target MAPIR e/o i dati del sensore di luce DAQ)
* ✅ **Metodo di debayering**: Standard (veloce, qualità media)
* ✅ **Formato di esportazione**: TIFF (16 bit)
* ✅ **Tutti i prodotti di esportazione**: Abilitato (LATTICE acquisisce automaticamente i file di output in formato debayered, anteprima, radianza e riflettanza)

Basta importare le immagini e avviare l’elaborazione con queste impostazioni predefinite.

***

## Panoramica delle impostazioni di progetto

Il pannello Impostazioni progetto è organizzato nelle sezioni riportate di seguito. Due sezioni aggiuntive — **Sensore di luce DAQ**e**Allineamento array** — vengono visualizzate automaticamente quando il progetto contiene i file pertinenti. Per la documentazione completa, consultare [Impostazioni progetto](../project-settings/project-settings.md).

### Visualizzazione

* **Risoluzione delle miniature delle immagini**: risoluzione delle miniature nella griglia delle immagini. Opzioni:**Predefinita (512 px)**,**1024 px**,**2048 px**,**Risoluzione completa**. Solo visualizzazione — non influisce mai sull’elaborazione. Valori più alti risultano più nitidi quando si ingrandisce l’immagine, ma il caricamento è più lento.

### Rilevamento dei target

Controlla il modo in cui Chloros identifica i target di calibrazione nelle immagini.

**Impostazioni chiave:*** **Area minima del campione di calibrazione (px)**: Soglia dimensionale per il rilevamento dei bersagli (predefinito:**25**, intervallo 0–10000)
* **Raggruppamento minimo dei bersagli (0-100)**: Soglia di somiglianza per il raggruppamento delle regioni bersaglio (impostazione predefinita:**60**)**Quando regolare:**

* Aumentare l’area di campionamento se si verificano rilevamenti errati
* Ridurla se i bersagli non vengono rilevati
* Regolare il raggruppamento se i bersagli vengono suddivisi in più rilevamenti

{% hint style="info" %}
Queste impostazioni sono disattivate quando **Calibrazione della riflettanza / bilanciamento del bianco** è disattivata — se disattivata, il rilevamento dei bersagli non viene mai eseguito.
{% endhint %}

### Elaborazione

Opzioni principali di elaborazione delle immagini e calibrazione.

**Impostazioni chiave:*** **Correzione della vignettatura**: compensa l’oscuramento ai bordi causato dall’obiettivo ✅ Consigliata
* **Calibrazione della riflettanza / bilanciamento del bianco**: calibra le immagini utilizzando i bersagli rilevati (Survey3) e/o i dati del sensore di luce DAQ (LATTICE) ✅ Consigliato
* **Metodo di debayering**: Algoritmo per la conversione da RAW a multispettrale a 3 canali
* **Intervallo minimo di ricalibrazione**: Tempo minimo in secondi tra un utilizzo e l’altro dei target di calibrazione (impostazione predefinita:**0** = usa tutti, intervallo 0–3600)**Prodotti di riserva non calibrati:**Quando non è possibile eseguire la calibrazione della riflettanza di un fotogramma (nessun bersaglio disponibile o calibrazione disabilitata), questo viene esportato come uno dei due prodotti di riserva —**esiste esattamente uno dei due per ogni esecuzione**, scelto dall’opzione di correzione della vignettatura:

* **Esporta risposta del sensore**: scrive `Sensor_Response_Images` — utilizzato quando la correzione della vignettatura è**disattivata*** **Esporta con correzione della vignettatura**: scrive `Vignette_Corrected_Images` — utilizzato quando la correzione della vignettatura è**attiva**La casella di controllo non attiva è disattivata. Deselezionando quella attiva si impedisce del tutto la scrittura di quel file.**Prodotti di esportazione LATTICE** (visualizzati per ogni progetto; si applicano alle acquisizioni LATTICE):

* **Esporta debayered**: l’immagine debayered lineare (`Debayered_Images`). Si applica a RGB e ai moduli multispettrali.
* **Esporta anteprima**: l’anteprima visualizzata (`Preview_Images`). RGB = bilanciamento del bianco (illuminante DAQ se disponibile, altrimenti “gray-world”) + gamma; multispettrale = espansione a falsi colori.
* **Esportazione della radianza**: radianza spettrale float32 (`Radiance_Images`, W/m²/sr/nm). Solo moduli multispettrali — non applicabile ai master RGB.
* ****Esporta riflettanza**: riflettanza uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0) quando una lettura in direzione discendente `.daq` o un bersaglio all’interno del fotogramma copre il fotogramma. Solo moduli multispettrali.

Tutte e quattro sono **attivate per impostazione predefinita**: un singolo fotogramma grezzo LATTICE importato viene distribuito a tutti i prodotti abilitati e applicabili in un unico ciclo di elaborazione. La casella di controllo**Esporta riflettanza** è disattivata quando la calibrazione della riflettanza è disattivata. Le impostazioni il cui interruttore principale le rende impossibili sono sempre disattivate con un tooltip che indica l’interruttore da modificare.**Impostazioni avanzate:*** **Offset del fuso orario del sensore di luce**: Ore rispetto all’UTC per la sincronizzazione dell’ora del sensore di luce (impostazione predefinita: 0, intervallo da −12 a +12)
* **Applica correzioni PPK**: utilizza i dati GPS/pin di esposizione dai file `.daq` (impostazione predefinita: disattivata)
* **Pin di esposizione 1/2**: assegna le telecamere ai pin di esposizione per configurazioni a doppia telecamera

{% hint style="info" %}
**Il livello di input di LATTICE è automatico.** Le acquisizioni LATTICE riportano il proprio livello di elaborazione nei metadati XMP e l’elaborazione entra sempre nella pipeline a partire dal fotogramma raw — non c’è nulla da configurare nell’interfaccia grafica. (Il flag CLI `--input-level` esiste come opzione avanzata per gli utenti esperti in caso di acquisizioni con metadati persi; vedere la [Guida di riferimento](../reference/cli-reference.md).)
{% endhint %}

### Metodo di debayering

Attualmente offriamo 2 metodi di debayering in Chloros:

#### Standard (Veloce, Qualità media)

Il debayering standard è veloce ma presenta rumore cromatico, con il risultato di immagini meno accurate e più rumorose.

#### Texture Aware (Lento, massima qualità) \[Solo Chloros+]

Texture Aware utilizza un debayer di alta qualità sensibile ai bordi combinato con un modello di denoising basato su AI/ML che rimuove quasi tutto il rumore da debayering. Il modello richiede memoria GPU (VRAM) per funzionare: con **7 GB o più di VRAM** è in grado di elaborare più immagini contemporaneamente; con meno di 7 GB elabora un’immagine alla volta (in modo notevolmente più lento). Vedi [Adattamento dinamico del calcolo](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Le acquisizioni LATTICE utilizzano sempre il demosaic standard.** Non esiste un modello Texture Aware addestrato per LATTICE, quindi l’opzione non è disponibile per le immagini LATTICE — le immagini Survey3 nello stesso progetto possono comunque utilizzarlo.
{% endhint %}

### Indici (Indici multispettrali)

Configurare quali indici di vegetazione calcolare ed esportare. Il menu a tendina dell’interfaccia grafica offre **27 formule di indici predefinite**.**Come aggiungere indici:**

1. Fare clic sul pulsante**&quot;Aggiungi indice&quot;**

2. Selezionare un indice dal menu a tendina (NDVI, NDRE, GNDVI, ecc.)
3. Configurare le impostazioni di visualizzazione (colori LUT, intervalli di valori)
4. Aggiungere più indici secondo necessità

**Indici più diffusi:*** **NDVI**: Stato di salute generale della vegetazione (il più comune)
* **NDRE**: Rilevamento precoce dello stress con RedEdge
* **GNDVI**: Sensibile alla concentrazione di clorofilla
* **OSAVI**: Funziona bene con il suolo visibile
* **EVI**: Aree con elevato indice di superficie fogliare (LAI)**Formule personalizzate:**

* Creazione di formule personalizzate per indici multispettrali con operazioni matematiche sulle bande di tutti i canali dell’immagine
* Salvataggio delle formule personalizzate per il riutilizzo
* Le formule personalizzate sono una funzionalità di Chloros+; la disponibilità dipende dal livello del piano sottoscritto

Per tutti gli indici e le formule disponibili — compresi quelli i cui nomi sono disponibili solo nell’interfaccia grafica e quelli che funzionano anche in CLI/SDK — consulta [Formule degli indici multispettrali](../project-settings/multispectral-index-formulas.md).

### Esportazione

Controlla il formato del file di output.

**Formati disponibili**(impostazione:**Formato immagine calibrata**, predefinito**TIFF (16 bit)**):

* **TIFF (16 bit)**: consigliato per GIS e analisi scientifiche
* **TIFF (32 bit, percentuale)**: valori in virgola mobile
* **PNG (8 bit)**: compressione senza perdita di dati per la visualizzazione
* **JPG (8 bit)**: file di dimensioni minime, compressione con perdita di dati

I file di output vengono salvati nella cartella del progetto, raggruppati per fotocamera e formato: `<project>/<camera>/<format>/<Product>_Images/`. La radianza viene **sempre** salvata come float32 nella cartella `tiff32`, indipendentemente da questa impostazione. I file esportati mantengono il nome del file sorgente — la cartella identifica il prodotto. Vedere [Completamento dell’elaborazione](finishing-the-processing.md) per l’albero completo dei file di output.

{% hint style="warning" %}
**Lettura dei valori di riflettanza**: il valore DN che corrisponde a ρ = 1,0 dipende dalla fotocamera di origine: LATTICE utilizza 32768 (contrassegnato come XMP `Chloros:PixelScale`), mentre Survey3 utilizza 65535. Leggere il tag anziché ipotizzare un valore costante. Vedere [Formati delle immagini di output](../output-image-formats.md).
{% endhint %}

### Sensore di luce DAQ

Questa sezione elenca tutti i file DAQ relativi alla radiazione discendente (`.daq` / `.csv`) presenti nel progetto, con una riga per ogni file, indicando il modello del sensore, il nome del file e la correzione del **tappo** del diffusore applicata a quel file.

* **Sovrascrittura del limite (tutti i file)**: un unico menu a tendina valido per l’intero progetto. L’opzione**Auto** (impostazione predefinita) utilizza il limite registrato per ciascun file; se non è stato registrato nulla, si presume che si tratti di luce solare, poiché tutti i DAQ MAPIR sono forniti con il correttore per la luce solare. La selezione di un limite sovrascrive tutti i file: le registrazioni grezze vengono corrette in base a esso, mentre quelle che contengono già un limite vengono ricalibrate (la correzione registrata viene annullata e viene applicato il limite selezionato).
* Le righe segnalano un avviso quando un limite massimo registrato era il valore predefinito ipotizzato dall’hub anziché confermato dall’operatore, e quando il limite massimo selezionato non ha un profilo per quel modello di dispositivo (la sovrascrittura viene rifiutata per quel file).

Le registrazioni DAQ effettuate nella scheda “Sensori di luce” vengono aggiunte automaticamente al progetto aperto, e i file `.daq` / `.csv` importati compaiono qui non appena vengono aggiunti.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Impostazioni inferiori del progetto — Indice, formato di esportazione, sezione DAQ Sensori di luce e controlli relativi al modello/alla cartella del progetto</p></figcaption></figure>### Allineamento dell’array

Questa sezione appare **solo** quando almeno un’immagine nel progetto contiene la trasformazione di allineamento da modulo a modulo che gli array LATTICE applicano al momento dell’acquisizione (`Chloros:Alignment*` XMP). Mostra quante immagini contengono i tag e quale fotocamera è di riferimento, con i seguenti controlli:

* **Applica allineamento dell’array** (impostazione predefinita: attiva): deforma ogni prodotto elaborato (debayering / anteprima / radianza / riflettanza / indice) nella geometria di riferimento condivisa dall’array. Disattivata = esportazione nella geometria nativa del sensore.
* **Ritaglia alla sovrapposizione comune** (impostazione predefinita: attiva): ritaglia le esportazioni allineate alla regione condivisa da tutti i moduli, in modo che ogni banda abbia la stessa impronta. Se disattivata, viene mantenuta l’intera area del sensore (riempimento nero al di fuori della sorgente).
* **Ricampionamento**:**Bilineare (uniforme, predefinito)**,**Più vicino (conserva i valori esatti)**— nessuna miscelazione tra i pixel, per un&#x27;analisi radiometrica rigorosa — oppure**Cubico (maggiore nitidezza)**.***

## Salvataggio e caricamento delle impostazioni

### Salva modello di progetto

Crea modelli riutilizzabili per flussi di lavoro coerenti:

1. Configura tutte le impostazioni desiderate nel pannello Impostazioni progetto
2. Scorri fino alla sezione **&quot;Salva modello di progetto&quot;** in fondo alla pagina
3. Inserisci un nome descrittivo per il modello (ad es. “Survey3N\_RGN\_Agriculture”)
4. Fai clic sull’icona di salvataggio

**Vantaggi:**

* Applica impostazioni identiche su più progetti
* Condividere le configurazioni con i membri del team
* Mantenere la coerenza per sondaggi ripetuti

### Caricare un modello su un nuovo progetto

Quando si crea un nuovo progetto:

1. Selezionare **&quot;Nuovo progetto&quot;** dal menu principale
2. Scegliere un modello di progetto dal selettore di modelli opzionale
3. Tutte le impostazioni del modello vengono applicate automaticamente

### Cartella di lavoro

L’impostazione **&quot;Cartella di lavoro&quot;** specifica dove vengono creati per impostazione predefinita i nuovi progetti:

* **Percorso predefinito**: `C:\Users\[Username]\Chloros Projects`
* **Modifica percorso**: fare clic sull’icona di modifica e selezionare una nuova cartella
* **Condivisa con CLI**: `chloros-cli` utilizza la stessa impostazione predefinita per la cartella dei progetti
* **Quando modificare**:
  * Unità di rete per la collaborazione in team
  * Un’unità diversa con maggiore spazio di archiviazione
  * Struttura delle cartelle organizzata per anno/cliente

***

## Configurazione PPK (Post-Processed Kinematic)

Se si utilizzano registratori DAQ MAPIR con GPS per una geolocalizzazione precisa:

### Prerequisiti

* DAQ MAPIR con modulo GPS (GNSS)
* File di log .daq con voci relative ai pin di esposizione
* Fotocamera collegata ai pin di esposizione del DAQ durante la sessione di acquisizione

### Procedura di configurazione

1. Inserire il file di log .daq nella cartella del progetto
2. Nelle Impostazioni del progetto, selezionare la casella di controllo **&quot;Applica correzioni PPK&quot;**

3. Impostare**&quot;Offset fuso orario del sensore di luce&quot;** se necessario (impostazione predefinita: 0 per UTC)
4. Assegnare le fotocamere ai pin di esposizione:
   * **Fotocamera singola**: assegnata automaticamente al Pin 1
   * **Doppia fotocamera**: assegnare manualmente ciascuna fotocamera al pin corretto**Assegnazione dei pin di esposizione:*** **Pin di esposizione 1**: selezionare il modello di fotocamera dal menu a tendina
* **Pin di esposizione 2**: selezionare la seconda fotocamera o «Non utilizzare»
* Non è possibile assegnare la stessa fotocamera a entrambi i pin

{% hint style="warning" %}
**Importante**: i pin di esposizione devono essere assegnati correttamente alle rispettive telecamere. Un&#x27;assegnazione errata comporterà dati di geolocalizzazione errati.
{% endhint %}

***

## Scenari avanzati

### Progetti con più telecamere

Quando si elaborano immagini provenienti da più telecamere MAPIR in un unico progetto:

1. Chloros rileva automaticamente il modello di ciascuna telecamera (sia Survey3 che LATTICE)
2. A ciascuna telecamera vengono assegnati i profili di elaborazione appropriati e una propria struttura di cartelle di output
3. PPK: assegnare manualmente a ciascuna telecamera Survey3 il pin di esposizione corretto
4. Tutte le telecamere utilizzano lo stesso formato di esportazione e gli stessi indici

**Esempi**: Survey3W RGN + Survey3N OCN (configurazione a doppia telecamera), oppure un array LATTICE che combina una telecamera master RGB con moduli a banda stretta

### Rilevamenti time-lapse o su più date

Per rilevamenti ripetuti della stessa area nel corso del tempo:

1. Creare un modello con le impostazioni standard
2. Utilizzare una configurazione coerente dei bersagli di calibrazione in ogni sessione
3. Elaborare ogni data come progetto separato
4. Utilizzare impostazioni identiche per ottenere risultati comparabili
5. Esportare nello stesso formato per l’analisi temporale

### Set di dati di grandi dimensioni

Per progetti con molte immagini (oltre 500):

* Valutare la possibilità di suddividere il progetto in progetti più piccoli per data o area
* Utilizzare l’elaborazione parallela Chloros+ per ottenere risultati più rapidi
* Valutare l’uso di CLI o API per l’automazione in batch
* Regolare l’intervallo minimo di ricalibrazione per ridurre il tempo di rilevamento dei target

***

## Verifica delle impostazioni

Prima di avviare l’elaborazione, verificare queste impostazioni chiave:

* [ ] Modello della fotocamera correttamente rilevato nel File Browser
* [ ] Correzione della vignettatura abilitata
* [ ] Calibrazione della riflettanza abilitata
* [ ] Per Survey3: almeno un&#x27;immagine di bersaglio di calibrazione importata e verificata; per LATTICE: presenza di un bersaglio e/o di una registrazione downwelling `.daq`
* [ ] Indici multispettrali desiderati aggiunti
* [ ] Formato di esportazione adeguato al proprio flusso di lavoro
* [ ] Impostazioni PPK configurate (se si utilizzano file .daq con eventi di esposizione)

***

## Passi successivi

Una volta configurate le impostazioni:

1. **Selezionare le immagini di riferimento per la calibrazione** - Vedi [Scelta delle immagini di riferimento](choosing-target-images.md)
2. **Avviare l’elaborazione** - Vedi [Avvio dell’elaborazione](starting-the-processing.md)
3. **Monitorare lo stato di avanzamento** - Vedi [Monitoraggio dell’elaborazione](monitoring-the-processing.md)

Per i dettagli completi su tutte le impostazioni disponibili, consultare la documentazione di riferimento [Impostazioni del progetto](../project-settings/project-settings.md).
