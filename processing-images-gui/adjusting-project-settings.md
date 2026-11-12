# Modifica delle impostazioni del progetto

Prima di elaborare le immagini, è importante configurare le impostazioni del progetto in modo che corrispondano alle esigenze del proprio flusso di lavoro. Il pannello Impostazioni progetto <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> offre un controllo completo su calibrazione, opzioni di elaborazione, indici multispettrali e formati di esportazione.

## Come accedere alle impostazioni del progetto

1. Apri il progetto in Chloros
2. Fai clic sull’icona **Impostazioni del progetto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> nella barra laterale sinistra
3. Il pannello "Impostazioni del progetto" mostra tutte le opzioni di configurazione

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Il pannello “Impostazioni del progetto” — Visualizzazione, rilevamento dei bersagli ed elaborazione</p></figcaption></figure>

{% hint style="info" %}
**Le impostazioni vengono salvate automaticamente** insieme al progetto. Quando si riapre un progetto, tutte le impostazioni vengono ripristinate.
{% endhint %}

***

## Configurazione rapida per flussi di lavoro comuni

### Impostazioni predefinite (consigliate per la maggior parte degli utenti)

Le impostazioni predefinite funzionano bene per i flussi di lavoro tipici di Survey3 e LATTICE:

* ✅ **Correzione della vignettatura**: Abilitata
* ✅ **Calibrazione della riflettanza / bilanciamento del bianco**: Abilitata (utilizza i target MAPIR e/o i dati del sensore di luce DAQ)
* ✅ **Metodo di debayering**: Standard (Veloce, Qualità media)
* ✅ **Formato di esportazione**: TIFF (16 bit) (TIFF (8 bit) per un progetto interamente a 8 bit)
* ✅ **Tutti i prodotti di esportazione**: Abilitato (LATTICE acquisisce automaticamente i dati debayered, di anteprima, di radianza e di riflettanza)

È sufficiente importare le immagini e avviare l’elaborazione con queste impostazioni predefinite.

***

## Panoramica delle impostazioni di progetto

Il pannello Impostazioni di progetto è organizzato nelle sezioni riportate di seguito. Due sezioni aggiuntive — **Sensore di luce DAQ**e**Allineamento array** — vengono visualizzate automaticamente quando il progetto contiene i file pertinenti. Per la documentazione completa, consultare [Impostazioni del progetto](../project-settings/project-settings.md).

### Visualizzazione

* **Risoluzione delle miniature delle immagini**: risoluzione delle miniature della griglia di immagini. Opzioni:**Predefinita (512 px)**,**1024 px**,**2048 px**,**Risoluzione completa**. Impostazione solo a scopo di visualizzazione — non influisce mai sull’elaborazione. Valori più alti garantiscono immagini più nitide quando si ingrandiscono, ma rallentano il caricamento.

### Rilevamento dei bersagli

Controlla il modo in cui Chloros identifica i bersagli di calibrazione nelle immagini.

**Impostazioni chiave:*** **Area minima del campione di calibrazione (px)**: Soglia dimensionale per il rilevamento dei bersagli (predefinito:**25**, intervallo 0–10000)
* **Raggruppamento minimo dei bersagli (0-100)**: soglia di somiglianza per il raggruppamento delle regioni dei bersagli (impostazione predefinita:**60**)**Quando regolare:**

* Aumentare l’area di campionamento se si ottengono rilevamenti errati
* Diminuirla se i bersagli non vengono rilevati
* Regolare il raggruppamento se i bersagli vengono suddivisi in più rilevamenti

{% hint style="info" %}
Queste impostazioni sono disattivate quando **Calibrazione della riflettanza / bilanciamento del bianco** è disattivata — se disattivata, il rilevamento dei bersagli non viene mai eseguito.
{% endhint %}

### Elaborazione

Opzioni principali di elaborazione delle immagini e calibrazione.

**Impostazioni chiave:*** **Correzione della vignettatura**: compensa l’oscuramento dell’obiettivo ai bordi ✅ Consigliata
* **Calibrazione della riflettanza / bilanciamento del bianco**: calibra le immagini utilizzando i bersagli rilevati (Survey3) e/o i dati del sensore di luce DAQ (LATTICE) ✅ Consigliata
* **Metodo di debayering**: algoritmo per la conversione da RAW a multispettrale a 3 canali
* **Intervallo minimo di ricalibrazione**: tempo minimo in secondi tra un utilizzo e l’altro dei target di calibrazione (impostazione predefinita:**0** = usa tutti, intervallo 0–3600)**Prodotti di riserva non calibrati:**Quando non è possibile eseguire la calibrazione della riflettanza di un fotogramma (nessun bersaglio disponibile o calibrazione disabilitata), questo viene esportato come uno dei due prodotti di riserva —**esiste esattamente uno dei due per ogni esecuzione**, scelto dall’opzione di correzione della vignettatura:

* **Esporta risposta del sensore**: scrive `Sensor_Response_Images` — utilizzato quando la correzione della vignettatura è**disattivata*** **Esporta con correzione della vignettatura**: scrive `Vignette_Corrected_Images` — utilizzato quando la correzione della vignettatura è**attiva**La casella di controllo non attiva è disattivata. Deselezionando quella attiva si impedisce del tutto la scrittura di quel file.**Prodotti di esportazione LATTICE** (visualizzati per ogni progetto; si applicano alle acquisizioni LATTICE):

* **Esporta senza demosaicizzazione**: l’immagine lineare debayered (`Debayered_Images`): l’offset di scuro del pacchetto di calibrazione viene rimosso, quindi viene eseguito il demosaicing, senza flat field né unmix — lo stesso file che un’acquisizione salva al suo livello debayered — nell’orientamento nativo del sensore. Si applica ai moduli RGB e multispettrali.
* **Esporta anteprima**: l’anteprima visualizzata (`Preview_Images`). RGB = bilanciamento del bianco (illuminante DAQ se disponibile, altrimenti “gray-world”) + gamma; multispettrale = estensione a falsi colori.
* **Esportazione della radianza**: radianza spettrale float32 (`Radiance_Images`, W/m²/sr/nm). Solo moduli multispettrali — non applicabile ai master RGB.
* **Esporta riflettanza**: riflettanza uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0) quando una lettura in direzione discendente `.daq` o un bersaglio all’interno del fotogramma copre il fotogramma. Solo moduli multispettrali.

Tutte e quattro sono **attivate per impostazione predefinita**: un singolo fotogramma grezzo LATTICE importato viene distribuito a tutti i prodotti abilitati e applicabili in un unico ciclo di elaborazione. La casella di controllo**Esporta riflettanza** è disattivata quando la calibrazione della riflettanza è disattivata. Le impostazioni il cui interruttore principale ne impedisce l’attivazione sono sempre disattivate con un suggerimento che indica l’interruttore da modificare.**Impostazioni avanzate:*** **Offset del fuso orario del sensore di luce**: Ore rispetto all’UTC per la sincronizzazione dell’ora del sensore di luce (impostazione predefinita: 0, intervallo da −12 a +12)
* **Applica correzioni PPK**: utilizza i dati GPS/pin di esposizione dai file `.daq` (impostazione predefinita: disattivata)
* **Pin di esposizione 1/2**: assegna le telecamere ai pin di esposizione per configurazioni a doppia telecamera

{% hint style="info" %}
**Il livello di input di LATTICE è automatico.** Le acquisizioni LATTICE riportano il proprio livello di elaborazione nei metadati XMP e l’elaborazione entra sempre nella pipeline a partire dal fotogramma grezzo — non c’è nulla da configurare nell’interfaccia grafica. (Il flag CLI `--input-level` è disponibile come opzione avanzata per gli utenti esperti in caso di acquisizioni con metadati persi; consultare il [Riferimento CLI](../reference/cli-reference.md).)
{% endhint %}

### Metodo di debayering

Attualmente in Chloros offriamo 2 metodi di debayering:

#### Standard (veloce, qualità media)

Il debayer standard elabora rapidamente ma presenta rumore cromatico dovuto al debayering, con il risultato di immagini meno accurate e più rumorose.

#### Texture Aware (Lento, massima qualità) \[Solo Chloros+]

Texture Aware utilizza un debayer di alta qualità sensibile ai bordi combinato con un modello di denoising basato su AI/ML che rimuove quasi tutto il rumore da debayering. Il modello richiede memoria GPU (VRAM) per funzionare: con **7 GB o più di VRAM** è in grado di elaborare più immagini contemporaneamente; con meno di 7 GB elabora una immagine alla volta (in modo notevolmente più lento). Vedi [Adattamento dinamico della potenza di calcolo](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Le acquisizioni LATTICE utilizzano sempre il demosaic standard.** Non esiste un modello Texture Aware addestrato per LATTICE, quindi l’opzione non è disponibile per le immagini LATTICE — le immagini Survey3 nello stesso progetto possono comunque utilizzarlo.
{% endhint %}

### Indici (indici multispettrali)

Configurare quali indici di vegetazione calcolare ed esportare. Il menu a tendina dell’interfaccia grafica offre **27 formule di indici predefinite**.**Come aggiungere indici:**

1. Fare clic sul pulsante**"Aggiungi indice"**

2. Selezionare un indice dal menu a tendina (NDVI, NDRE, GNDVI, ecc.)
3. Configurare le impostazioni di visualizzazione (colori LUT, intervalli di valori)
4. Aggiungere più indici secondo necessità

**Indici più diffusi:*** **NDVI**: Stato di salute generale della vegetazione (il più comune)
* **NDRE**: Rilevamento precoce dello stress con RedEdge
* **GNDVI**: Sensibile alla concentrazione di clorofilla
* **OSAVI**: Funziona bene con il suolo visibile
* **EVI**: Regioni con elevato indice di area fogliare (LAI)

**Formule personalizzate:**

* Crea formule personalizzate per indici multispettrali utilizzando operazioni matematiche tra le bande su tutti i canali dell’immagine
* Salva le formule personalizzate per riutilizzarle
* Le formule personalizzate sono una funzionalità di Chloros+; la disponibilità dipende dal livello del tuo piano

Per tutti gli indici e le formule disponibili — compresi quelli che sono disponibili solo nell’interfaccia grafica (GUI) e quelli che funzionano anche nella CLI o nell’SDK — consulta [Formule degli indici multispettrali](../project-settings/multispectral-index-formulas.md).

### Esportazione

Controlla il formato del file di output.

**Formati disponibili**(impostazione:**Formato immagine calibrata**, predefinito**TIFF (16 bit)**):

* **TIFF (16 bit)**: consigliato per GIS e analisi scientifiche
* **TIFF (8 bit)**: TIFF a 8 bit* **TIFF (32 bit, Percentuale)**: valori in virgola mobile
* **PNG (8 bit)**: compressione senza perdita di dati per la visualizzazione
* **JPG (8 bit)**: file più piccoli, compressione con perdita di dati

L'elenco mostra quali formati possono contenere le immagini del progetto: Chloros non aumenta la profondità di bit. Le immagini interamente a 8 bit offrono solo TIFF (8 bit), PNG (8 bit) e JPG (8 bit), e il formato predefinito è TIFF (8 bit). In un progetto misto, ogni acquisizione viene esportata con la propria profondità di bit o con una inferiore. La radianza rimane a 32 bit in virgola mobile.

I file di output vengono salvati nella cartella del progetto, raggruppati per fotocamera e formato: `<project>/<camera>/<format>/<Product>_Images/`. Radiance viene **sempre** salvato come float32 nella cartella `tiff32`, indipendentemente da questa impostazione. I file esportati mantengono il nome del file sorgente — la cartella identifica il prodotto. Le esportazioni multispettrali LATTICE possono anche scrivere maschere di limitazione per pixel in una cartella `clamp_masks/` accanto alle cartelle dei prodotti. Vedere [Completamento dell’elaborazione](finishing-the-processing.md) per l’albero completo dei file di output.

{% hint style="warning" %}
**Lettura dei valori di riflettanza**: il DN che indica ρ = 1,0 dipende dalla fotocamera di origine — LATTICE utilizza 32768 (indicato come XMP `Chloros:PixelScale`), mentre Survey3 utilizza 65535. Leggere il tag anziché ipotizzare un valore costante. Vedere [Formati delle immagini di output](../output-image-formats.md).
{% endhint %}

### Sensore di luce DAQ

Questa sezione elenca tutti i file DAQ downwelling (`.daq` / `.csv`) presenti nel progetto, una riga per ogni file, indicando il modello del sensore, il nome del file e la **correzione del cappuccio** del diffusore in vigore per quel file.

* **Sovrascrittura del cappuccio (tutti i file)**: un unico menu a tendina valido per l’intero progetto.**Auto** (impostazione predefinita) utilizza il limite massimo registrato in ciascun file — se non è stato registrato nulla, si presume la luce solare; tuttavia, DAQ-U, DAQ-M e DAQ-E vengono forniti senza limite massimo e Chloros non è in grado di rilevare quale sia stato applicato, pertanto l’elaborazione genera un avviso relativo a un limite massimo ipotizzato: selezionare il limite massimo effettivamente applicato. La selezione di un limite sovrascrive tutti i file tranne quelli di un DAQ-E-S, il cui diffusore fa parte della sua calibrazione: le registrazioni grezze vengono corrette con esso, mentre quelle che contengono già un limite vengono ricalibrate (la correzione registrata viene annullata e viene applicato il limite selezionato).
* Le righe segnalano un avviso quando un cappuccio registrato era il valore predefinito ipotizzato dall’hub anziché quello confermato dall’operatore, e quando il cappuccio selezionato non ha un profilo per quel modello di dispositivo (la sostituzione viene rifiutata per quel file).

Le registrazioni DAQ effettuate nella scheda «Sensori di luce» vengono aggiunte automaticamente al progetto aperto, e i file `.daq` / `.csv` importati compaiono qui non appena vengono aggiunti.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Impostazioni inferiori del progetto — Indice, formato di esportazione, sezione DAQ Sensori di luce e controlli relativi al modello/alla cartella del progetto</p></figcaption></figure>### Allineamento dell’array

Questa sezione appare **solo** quando almeno un’immagine nel progetto contiene la trasformazione di allineamento da modulo a modulo che gli array LATTICE applicano al momento dell’acquisizione (`Chloros:Alignment*` XMP). Mostra quante immagini contengono i tag e quale fotocamera funge da riferimento, con i seguenti controlli:

* **Applica allineamento dell’array** (impostazione predefinita: attiva): deforma ogni prodotto elaborato (debayering / anteprima / radianza / riflettanza / indice) nella geometria di riferimento condivisa dall’array. Disattivata = esportazione nella geometria nativa del sensore.
* **Ritaglia alla sovrapposizione comune** (impostazione predefinita: attiva): ritaglia le esportazioni allineate alla regione condivisa da tutti i moduli, in modo che ogni banda abbia la stessa impronta. Se disattivata, viene mantenuta l’intera area del sensore (riempimento nero al di fuori della sorgente).
* **Ricampionamento**:**Bilineare (morbido, predefinito)**,**Più vicino (mantiene i valori esatti)**— nessuna miscelazione tra pixel, per un’analisi radiometrica rigorosa — oppure**Cubico (più nitido)**.

***

## Salvataggio e caricamento delle impostazioni

### Salva modello di progetto

Crea modelli riutilizzabili per flussi di lavoro coerenti:

1. Configura tutte le impostazioni desiderate nel pannello Impostazioni progetto
2. Scorri fino alla sezione **"Salva modello di progetto"** in fondo alla pagina
3. Inserisci un nome descrittivo per il modello (ad es., “Survey3N\_RGN\_Agriculture”)
4. Fai clic sull’icona di salvataggio

**Vantaggi:**

* Applica impostazioni identiche a più progetti
* Condividi le configurazioni con i membri del team
* Mantieni la coerenza per rilievi ripetuti

### Caricare un modello su un nuovo progetto

Quando si crea un nuovo progetto:

1. Selezionare **"Nuovo progetto"** dal menu principale
2. Scegliere un modello di progetto dal selettore di modelli opzionale
3. Tutte le impostazioni del modello vengono applicate automaticamente

### Directory di lavoro

L’impostazione **"Directory di lavoro"** specifica dove vengono creati per impostazione predefinita i nuovi progetti:

* **Percorso predefinito**: `Documents\MAPIR\Chloros Projects` nella cartella Documenti di Windows (di solito `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; segue la cartella Documenti se OneDrive o un profilo mobile la spostano)
* **Modifica posizione**: fare clic sull’icona di modifica e selezionare una nuova cartella
* **Condivisa con l’CLI**: `chloros-cli` utilizza la stessa impostazione predefinita della cartella di progetto
* **Nomi brevi su Windows**: a meno che non siano abilitati i percorsi lunghi di Windows, una cartella di progetto con un nome più lungo di circa 130 caratteri viene rifiutata prima dell’elaborazione — vedi [Avvio dell’elaborazione](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
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
* Telecamera collegata ai pin di esposizione del DAQ durante la sessione di acquisizione

### Passaggi di configurazione

1. Inserire il file di log .daq nella cartella del progetto
2. Nelle Impostazioni del progetto, selezionare la casella di controllo **"Applica correzioni PPK"**

3. Impostare**"Offset fuso orario sensore di luce"** se necessario (impostazione predefinita: 0 per UTC)
4. Assegnare le fotocamere ai pin di esposizione:
   * **Fotocamera singola**: assegnata automaticamente al Pin 1
   * **Doppia fotocamera**: assegnare manualmente ciascuna fotocamera al pin corretto**Assegnazione dei pin di esposizione:*** **Pin di esposizione 1**: selezionare il modello di telecamera dal menu a tendina
* **Pin di esposizione 2**: selezionare la seconda telecamera o “Non utilizzare”
* Non è possibile assegnare la stessa telecamera a entrambi i pin

{% hint style="warning" %}
**Importante**: i pin di esposizione devono essere correttamente assegnati alle rispettive telecamere. Un’assegnazione errata comporterà dati di geolocalizzazione errati.
{% endhint %}

***

## Scenari avanzati

### Progetti con più fotocamere

Quando si elaborano immagini provenienti da più fotocamere MAPIR in un unico progetto:

1. Chloros rileva automaticamente il modello di ciascuna fotocamera (sia Survey3 che LATTICE)
2. A ciascuna telecamera vengono assegnati i profili di elaborazione appropriati e una propria struttura di cartelle di output
3. PPK: assegnare manualmente a ciascuna telecamera Survey3 il pin di esposizione corretto
4. Tutte le telecamere utilizzano lo stesso formato di esportazione e gli stessi indici

**Esempi**: configurazione a doppia telecamera Survey3W RGN + Survey3N OCN, oppure un array LATTICE che combina una telecamera master RGB con moduli a banda stretta

### Rilevamenti time-lapse o in date diverse

Per rilevamenti ripetuti della stessa area nel corso del tempo:

1. Creare un modello con le impostazioni standard
2. Utilizzare una configurazione coerente dei target di calibrazione in ogni sessione
3. Elaborare ogni data come progetto separato
4. Utilizzare impostazioni identiche per ottenere risultati comparabili
5. Esportare nello stesso formato per l’analisi temporale

### Set di dati di grandi dimensioni

Per progetti con molte immagini (oltre 500):

* Valutare la possibilità di suddividere il progetto in progetti più piccoli per data o area
* Utilizza l’elaborazione parallela di Chloros+ per ottenere risultati più rapidi
* Valuta l’uso di CLI o API per l’automazione in batch
* Regola l’intervallo minimo di ricalibrazione per ridurre il tempo di rilevamento dei target

***

## Verifica delle impostazioni

Prima di iniziare l’elaborazione, controlla queste impostazioni chiave:

* [ ] Modello della fotocamera rilevato correttamente nel File Browser
* [ ] Correzione della vignettatura abilitata
* [ ] Calibrazione della riflettanza abilitata
* [ ] Per Survey3: almeno un’immagine di target di calibrazione importata e verificata; per LATTICE: presenza di un target e/o di una registrazione downwelling `.daq`
* [ ] Indici multispettrali desiderati aggiunti
* [ ] Formato di esportazione adeguato al proprio flusso di lavoro
* [ ] Impostazioni PPK configurate (se si utilizzano file .daq con eventi di esposizione)

***

## Passi successivi

Una volta configurate le impostazioni:

1. **Contrassegnare le immagini di riferimento per la calibrazione** - Vedi [Scelta delle immagini di riferimento](choosing-target-images.md)
2. **Avviare l’elaborazione** - Vedi [Avvio dell’elaborazione](starting-the-processing.md)
3. **Monitorare lo stato di avanzamento** - Vedi [Monitoraggio dell'elaborazione](monitoring-the-processing.md)

Per informazioni complete su tutte le impostazioni disponibili, consultare la documentazione di riferimento [Impostazioni del progetto](../project-settings/project-settings.md).
