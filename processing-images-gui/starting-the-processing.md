# Avvio dell&#x27;elaborazione

Una volta importate le immagini, contrassegnati i target di calibrazione e configurate le impostazioni del progetto, sei pronto per iniziare l&#x27;elaborazione. Questa pagina ti guida nell&#x27;avvio della pipeline di elaborazione Chloros.

## Lista di controllo pre-elaborazione

Prima di fare clic sul pulsante Avvia, verifica che tutto sia pronto:

* [ ] **File importati** - Tutte le immagini sono visibili nel File Browser
* [ ] **Immagini di riferimento contrassegnate** - Colonna &quot;Riferimento&quot; spuntata per le immagini di calibrazione (oppure una registrazione `.daq` importata per LATTICE)
* [ ] **Modelli di telecamera rilevati** - La colonna “Modello telecamera” mostra le telecamere corrette
* [ ] **Impostazioni configurate** - Impostazioni del progetto verificate e regolate
* [ ] **Indici selezionati** - Indici multispettrali desiderati aggiunti (se necessario)
* [ ] **Formato di esportazione scelto** - Formato di output adeguato al proprio flusso di lavoro

{% hint style="info" %}
**Suggerimento**: fare clic su alcune immagini nel File Browser per verificare che siano state caricate correttamente prima dell’elaborazione.
{% endhint %}

***

## Avvio dell’elaborazione

### Individuare il pulsante Avvia

Il pulsante Avvia/Riproduci si trova nella barra di intestazione superiore di Chloros:

* Posizione: in alto al centro della finestra
* Icona: **Pulsante Riproduci/Avvia** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Stato: il pulsante è attivo (illuminato) quando è pronto per l’elaborazione

### Fare clic per avviare

1. Fare clic sul **pulsante Riproduci/Avvia** nella barra superiore
2. L’elaborazione inizia immediatamente
3. Durante l’elaborazione, il pulsante diventa un pulsante **Stop**

4. La barra di avanzamento si aggiorna, mostrando lo stato dell’elaborazione

{% hint style="success" %}
**Elaborazione avviata**: una volta cliccato, Chloros gestisce automaticamente tutte le fasi di elaborazione: rilevamento del bersaglio, debayering, calibrazione, calcolo dell’indice ed esportazione. Rileva automaticamente se il progetto è di tipo Survey3, LATTICE o misto, e applica la pipeline corretta a ciascuna telecamera.
{% endhint %}

***

## Comprensione delle modalità di elaborazione

Chloros opera in due diverse modalità di elaborazione a seconda della licenza:

### Modalità gratuita (elaborazione sequenziale)

**Disponibile per tutti gli utenti**

**Come funziona:**

* Elabora le immagini una alla volta, in sequenza
* Funzionamento a thread singolo
* Minore utilizzo di memoria

**La barra di avanzamento mostra 2 fasi:**

1.**Rilevamento dei target** - Ricerca dei target di calibrazione
2. **Elaborazione** - Applicazione della calibrazione ed esportazione delle immagini**Tempo di elaborazione:**

* Molto più lento rispetto alla modalità parallela di Chloros+
* Adatto a set di dati di piccole e medie dimensioni (&lt; 200 immagini)

### Modalità Chloros+ (elaborazione parallela)

**Richiede la licenza Chloros+**

**Come funziona:**

* Elabora più immagini contemporaneamente utilizzando una [pipeline di elaborazione a 4 thread](../processing-architecture/processing-pipeline.md)
* L’[adattamento dinamico del calcolo](../processing-architecture/dynamic-compute-adaptation.md) seleziona automaticamente la strategia ottimale per il proprio hardware all’avvio dell’esecuzione
* Accelerazione GPU (CUDA) con schede grafiche NVIDIA (desktop e Jetson)
* **Il numero di worker si adatta all’hardware**: le strategie GPU eseguono**da 1 a 4 worker simultanei** (in base alla VRAM — un Jetson con poca memoria ne esegue 1, una GPU desktop da 12 GB o più ne esegue fino a 4); i sistemi solo CPU eseguono un worker per ogni core fisico, meno uno**La barra di avanzamento mostra 4 fasi** (corrispondenti ai 4 thread della pipeline):

1. **Rilevamento** (Thread 1) - Individuazione dei target di calibrazione
2. **Analisi** (Thread 2) - Esame dei metadati dell’immagine e calcolo della calibrazione
3. **Calibrazione** (Thread 3) - Debayering, correzione della vignettatura, calibrazione, calcolo dell’indice
4. **Esportazione** (Thread 4) - Salvataggio delle immagini elaborate e degli indici**Interazione con la barra di avanzamento:*** **Passare il mouse** sulla barra per visualizzare il pannello a tendina dettagliato delle 4 fasi
* **Cliccare** sulla barra di avanzamento per bloccare il pannello a tendina in posizione
* **Cliccare nuovamente** per sbloccare e nascondere il pannello**Tempo di elaborazione:**

* Significativamente più veloce rispetto alla modalità gratuita
* L’accelerazione GPU migliora ulteriormente la velocità

{% hint style="info" %}
**Chloros+ Velocità**: L’elaborazione parallela può essere da 5 a 10 volte più veloce rispetto alla modalità sequenziale per set di dati di grandi dimensioni. Un progetto di 500 immagini che richiede 2 ore in modalità gratuita può essere completato in 15-20 minuti con Chloros+.
{% endhint %}

***

## Cosa succede durante l’elaborazione

### Fase 1: Rilevamento dei target

**Cosa fa Chloros:**

* Esegue la scansione delle immagini selezionate nella colonna “Target” (tutte le immagini se nessuna è selezionata)
* Identifica i pannelli di calibrazione in ciascun bersaglio
* Estrazione dei valori di riflettanza dai pannelli dei bersagli
* Registrazione dei timestamp dei bersagli per la pianificazione della calibrazione

**Durata:** 1-30 secondi (con bersagli contrassegnati), 5-30+ minuti (non contrassegnati)

### Fase 2: Debayering (Conversione RAW)

**Cosa fa Chloros:**

* Converte i dati RAW con schema Bayer in immagini complete a 3 canali (i moduli mono LATTICE rimangono a banda singola — per questi il debayering viene saltato con una nota nel log)
* Applica l’algoritmo di demosaicing selezionato
* Preserva la massima qualità dell’immagine e il massimo livello di dettaglio

**Durata:** varia in base al numero di immagini e alla velocità della CPU/GPU

### Fase 3: Calibrazione

**Cosa fa Chloros:*** **Correzione della vignettatura**: rimuove l’oscuramento dei bordi causato dall’obiettivo
* **Calibrazione della riflettanza**: normalizza utilizzando i valori di riflettanza di riferimento e/o i dati di irraggiamento verso il basso (downwelling) del sistema di acquisizione dati (DAQ)
* Applica le correzioni su tutte le bande/canali
* Utilizza il riferimento di calibrazione appropriato per ciascuna immagine in base al timestamp

**Durata:** La maggior parte del tempo di elaborazione

### Fase 4: Calcolo degli indici

**Cosa fa Chloros:**

* Calcola gli indici multispettrali configurati (NDVI, NDRE, ecc.)
* Applica operazioni matematiche sulle bande alle immagini calibrate
* Genera immagini di indice per ciascun indice selezionato

**Durata:** Pochi secondi per immagine

### Fase 5: Esportazione

**Cosa fa Chloros:**

* Salva le immagini elaborate nel formato selezionato
* **Fan-out LATTICE**: ogni fotogramma LATTICE grezzo viene esportato come tutti i prodotti abilitati in un unico passaggio — debayering, anteprima, radianza (sempre float32), riflettanza
* Scrive i file nella struttura di output del progetto: `<project>/<camera>/<format>/<Product>_Images/`
* **Mantiene il nome del file sorgente** — la cartella identifica il prodotto, non viene aggiunto alcun suffisso**Durata:** Varia in base al formato di esportazione e alle dimensioni del file***

## Comportamento di elaborazione

### Pipeline di elaborazione automatica

Una volta avviata, l’intera pipeline viene eseguita automaticamente:

* Non è richiesta alcuna interazione da parte dell’utente
* Tutti i passaggi configurati vengono eseguiti in sequenza
* Gli aggiornamenti sullo stato di avanzamento vengono mostrati in tempo reale
* I file esportati vengono scritti su disco man mano che vengono completati — è possibile aprire i file di output finiti mentre l’elaborazione continua

### Utilizzo del computer durante l’elaborazione

**Modalità libera:**

* Utilizzo della CPU relativamente basso (single-threaded)
* Il computer rimane reattivo per altre attività
* È sicuro ridurre a icona Chloros e lavorare in altre applicazioni

**Modalità parallela Chloros+:**

* Elevato utilizzo della CPU nell’intero pool di worker della strategia
* Con accelerazione GPU: elevato utilizzo della GPU
* Il computer potrebbe essere meno reattivo durante l’elaborazione
* Evitare di avviare altre attività che richiedono un uso intensivo della CPU

{% hint style="warning" %}
**Suggerimento per le prestazioni**: Per ottenere le migliori prestazioni con Chloros+, chiudere le altre applicazioni e consentire a Chloros di utilizzare tutte le risorse di sistema.
{% endhint %}

### L’elaborazione non può essere messa in pausa (ma l’arresto è pulito)

* Una volta avviata, l’elaborazione non può essere messa in pausa e ripresa in un secondo momento
* Facendo clic su **Stop**, l’esecuzione viene interrotta in modo pulito al primo clic
* I prodotti già esportati prima dell’arresto rimangono sul disco
* Un&#x27;esecuzione interrotta riporta in modo accurato ciò che è stato completato (vedere le righe `[RUN-SUMMARY]` nel log)
* Una nuova esecuzione avvia la pipeline dall’inizio

**Suggerimento per la pianificazione:** Per progetti di grandi dimensioni, valutare l’elaborazione in batch o l’utilizzo di CLI per un migliore controllo.***

## Monitoraggio dell’elaborazione

Durante l’esecuzione dell’elaborazione, è possibile:

* **Osservare la barra di avanzamento** - Visualizzare la percentuale complessiva di completamento
* **Visualizzare la fase corrente**: Rilevamento, Analisi, Calibrazione o Esportazione
* **Controllare la scheda del log**: visualizzare messaggi e avvisi dettagliati sull’elaborazione
* **Visualizzare in anteprima le immagini completate**: i file di esportazione vengono salvati su disco durante l’elaborazione

Per informazioni dettagliate sul monitoraggio, consultare [Monitoraggio dell’elaborazione](monitoring-the-processing.md).

***

## Interruzione dell’elaborazione

Se è necessario interrompere l’elaborazione:

### Come interrompere

1. Individuare il **pulsante Stop** (sostituisce il pulsante Avvia durante l’elaborazione)
2. Fare clic una volta: la barra mostra **&quot;Arresto in corso...&quot;** mentre l’immagine in elaborazione viene completata
3. L’esecuzione termina in uno stato di arresto definitivo e il registro riporta un resoconto dettagliato (`[RUN-SUMMARY]`) di ciò che è stato completato

### Quando interrompere

**Motivi validi per interrompere:**

* Ci si è resi conto di aver utilizzato impostazioni errate
* Si è dimenticato di contrassegnare le immagini di destinazione
* Sono state importate immagini sbagliate
* Il sistema è troppo lento o non risponde

**Dopo l’interruzione:**

* I prodotti esportati prima dell’interruzione rimangono sul disco
* Esaminare e risolvere eventuali problemi, regolare le impostazioni secondo necessità
* Riavviare l’elaborazione: l’esecuzione riparte dall’inizio

***

## Stime sui tempi di elaborazione

Il tempo di elaborazione effettivo varia notevolmente in base a:

* Numero di immagini
* Risoluzione delle immagini
* Formato di input RAW o JPG
* Modalità di elaborazione (Free o Chloros+)
* Velocità della CPU e numero di core
* Disponibilità della GPU (solo Chloros+)
* Numero di indici da calcolare
* Numero di prodotti di esportazione abilitati (LATTICE)

### Stime approssimative (Chloros+, immagini da 12 MP, CPU moderna)

| Numero di immagini | Modalità gratuita | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 immagini   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 immagini  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 immagini  | 1-1,5 ore | 20-30 min      | 10-15 min      |
| 500 immagini  | 2-3 ore   | 45-60 min      | 20-30 min      |
| 1.000 immagini | 4-6 ore   | 1,5-2 ore      | 40-60 min      |

{% hint style="info" %}

****Prima esecuzione**: l’elaborazione iniziale potrebbe richiedere più tempo poiché Chloros crea cache e profili. L’elaborazione successiva di set di dati simili sarà più veloce.
{% endhint %}

***

## Problemi comuni all’avvio

### Pulsante di avvio disabilitato (in grigio)

**Possibili cause:**

* Nessuna immagine importata
* Backend non completamente avviato
* Elaborazione precedente ancora in corso
* Progetto non completamente caricato

**Soluzioni:**

1. Attendere che il backend si inizializzi completamente (controllare l’icona nel menu principale)
2. Verificare che le immagini siano state importate nel File Browser
3. Riavviare Chloros se il pulsante rimane disabilitato
4. Controllare il log di debug per eventuali messaggi di errore

### L’elaborazione si avvia ma fallisce immediatamente

**Possibili cause:**

* Nessuna immagine valida nel progetto
* File immagine danneggiati
* Spazio su disco insufficiente
* Memoria insufficiente (RAM)

**Soluzioni:**

1. Controllare il registro di debug <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> per eventuali messaggi di errore
2. Verificare lo spazio disponibile su disco
3. Provare a elaborare un sottoinsieme più piccolo di immagini
4. Verificare che le immagini non siano danneggiate

### L&#x27;esecuzione termina ma non vengono scritte immagini

Un&#x27;esecuzione che ha richiesto prodotti immagine ma non ne ha scritta nessuna viene considerata un **fallimento, non un successo** — Chloros lo segnala chiaramente:

* Il log dell’interfaccia grafica riporta il codice `[RUN-SUMMARY]`, indicando la causa probabile: nessuna immagine importata, nessun bersaglio rilevato oppure tutti i prodotti richiesti ignorati in quanto non applicabili (ad es. richiesta di radianza/riflettanza da telecamere che supportano solo RGB)
* L’equivalente di CLI (`chloros-cli process`) visualizza `Processing finished but wrote no image products.` e **termina con un valore diverso da zero**, in modo che gli script possano rilevarlo
* Un&#x27;esecuzione deliberata solo con metadati (tutti i prodotti di esportazione disabilitati, nessun indice) viene comunque considerata riuscita

Consultare [il riferimento CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails) per la semantica completa.

### Avviso «Nessun bersaglio rilevato»

**Possibili cause:**

* Si è dimenticato di contrassegnare le immagini bersaglio
* Le immagini bersaglio non contengono bersagli visibili
* Impostazioni di rilevamento dei bersagli troppo rigide

**Soluzioni:**

1. Consultare [Scelta delle immagini bersaglio](choosing-target-images.md)
2. Contrassegnare le immagini appropriate nella colonna &quot;Target&quot;
3. Verificare che i target siano visibili nelle immagini contrassegnate
4. Regolare le impostazioni di rilevamento dei target, se necessario

***

## Suggerimenti per un&#x27;elaborazione efficace

### Prima di iniziare

1. **Eseguire prima un test con un piccolo sottoinsieme** - Elaborare 10-20 immagini per verificare le impostazioni
2. **Controllare lo spazio disponibile su disco** - Assicurarsi di avere spazio libero pari a 2-3 volte la dimensione del set di dati (di più se sono abilitati tutti i prodotti LATTICE)
3. **Chiudere le applicazioni non necessarie** - Liberare risorse di sistema
4. **Verifica le immagini dei target** - Visualizza in anteprima i target contrassegnati per assicurarti della qualità
5. **Salva il progetto** - Il progetto viene salvato automaticamente, ma è buona norma salvarlo manualmente

### Durante l’elaborazione

1. **Evitare la sospensione del sistema** - Disattivare le modalità di risparmio energetico
2. **Mantenere Chloros in primo piano** - O almeno visibile nella barra delle applicazioni
3. **Monitorare occasionalmente lo stato di avanzamento** - Verificare la presenza di avvisi o errori
4. **Non caricare altre applicazioni pesanti** - Soprattutto con Chloros+ in modalità parallela

### Accelerazione GPU di Chloros+

Se si utilizza l’accelerazione GPU NVIDIA:

1. Aggiornare i driver NVIDIA all’ultima versione
2. Assicurarsi che la GPU disponga di almeno 4 GB di VRAM (7 GB o più per il debayering Texture Aware in parallelo)
3. Chiudere le applicazioni che richiedono un uso intensivo della GPU (giochi, editing video)
4. Monitorare la temperatura della GPU (assicurarsi che il raffreddamento sia adeguato)

***

## Passaggi successivi

Una volta avviata l’elaborazione:

1. **Monitorare lo stato di avanzamento** - Vedi [Monitoraggio dell’elaborazione](monitoring-the-processing.md)
2. **Attendere il completamento** - L’elaborazione viene eseguita automaticamente
3. **Verificare i risultati** - Vedi [Conclusione dell&#x27;elaborazione](finishing-the-processing.md)

Per informazioni su cosa fare durante l&#x27;elaborazione, consultare [Monitoraggio dell&#x27;elaborazione](monitoring-the-processing.md).
