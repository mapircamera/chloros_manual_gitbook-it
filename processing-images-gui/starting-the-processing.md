# Avvio dell&#x27;elaborazione

Una volta importate le immagini, contrassegnati i target di calibrazione e configurate le impostazioni del progetto, è possibile avviare l&#x27;elaborazione. Questa pagina guida l&#x27;utente attraverso l&#x27;avvio della pipeline di elaborazione Chloros.

## Lista di controllo pre-elaborazione

Prima di fare clic sul pulsante Avvia, verificare che tutto sia pronto:

* [ ] **File importati** - Tutte le immagini vengono visualizzate nel File Browser
* [ ] **Immagini target contrassegnate** - Colonna Target controllata per le immagini di calibrazione
* [ ] **Modelli di fotocamera rilevati** - La colonna Modello fotocamera mostra le fotocamere corrette
* [ ] **Impostazioni configurate** - Impostazioni del progetto riviste e regolate
* [ ] **Indici selezionati** - Indici multispettrali desiderati aggiunti (se necessario)
* [ ] **Formato di esportazione scelto** - Formato di output appropriato per il tuo flusso di lavoro

{% hint style=&quot;info&quot; %}
**Suggerimento**: clicca su alcune immagini nel File Browser per verificare che siano state caricate correttamente prima dell&#x27;elaborazione.
{% endhint %}

***

## Avvio dell&#x27;elaborazione

### Individuare il pulsante Avvia

Il pulsante Avvia/Riproduci si trova nella barra di intestazione superiore di Chloros:

* Posizione: parte superiore centrale della finestra
* Icona: **pulsante Riproduci/Avvia** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Stato: il pulsante è abilitato (illuminato) quando è pronto per l&#x27;elaborazione

### Fare clic per avviare

1. Fare clic sul **pulsante Riproduci/Avvia** nell&#x27;intestazione superiore
2. L&#x27;elaborazione inizia immediatamente
3. Il pulsante viene disabilitato (grigio) durante l&#x27;elaborazione
4. La barra di avanzamento si aggiorna, mostrando lo stato dell&#x27;elaborazione

{% hint style=&quot;success&quot; %}
**Elaborazione avviata**: una volta cliccato, Chloros gestisce automaticamente tutte le fasi di elaborazione: rilevamento del target, debayering, calibrazione, calcolo dell&#x27;indice ed esportazione.
{% endhint %}

***

## Comprensione delle modalità di elaborazione

Chloros funziona in due diverse modalità di elaborazione a seconda della licenza:

### Modalità gratuita (elaborazione sequenziale)

**Disponibile per tutti gli utenti**

**Come funziona:**

* Elabora le immagini una alla volta, in modo sequenziale
* Funzionamento a thread singolo
* Minore utilizzo di memoria

**La barra di avanzamento mostra 2 fasi:**

1. **Rilevamento del target** - Scansione dei target di calibrazione
2. **Elaborazione** - Applicazione della calibrazione ed esportazione delle immagini

**Tempo di elaborazione:**

* Molto più lento rispetto alla modalità parallela Chloros+
* Adatto per set di dati di piccole e medie dimensioni (&lt; 200 immagini)

### Modalità Chloros+ (elaborazione parallela)

**Richiede la licenza Chloros+**

**Come funziona:**

* Elabora più immagini contemporaneamente
* Funzionamento multi-thread (fino a 16 worker paralleli)
* Utilizza più core CPU
* Accelerazione GPU (CUDA) opzionale con schede grafiche NVIDIA

**La barra di avanzamento mostra 4 fasi:**

1. **Rilevamento** - Ricerca dei target di calibrazione
2. **Analisi** - Esame dei metadati dell&#x27;immagine e preparazione della pipeline
3. **Calibrazione** - Applicazione delle correzioni e delle calibrazioni
4. **Esportazione** - Salvataggio delle immagini elaborate e degli indici

**Interazione con la barra di avanzamento:**

* **Passa il mouse** sulla barra per visualizzare il pannello a tendina dettagliato con le 4 fasi
* **Clicca** sulla barra di avanzamento per bloccare il pannello a tendina in posizione
* **Clicca di nuovo** per sbloccare e nascondere il pannello

**Tempo di elaborazione:**

* Significativamente più veloce rispetto alla modalità gratuita
* Scalabile in base al numero di core della CPU
* L&#x27;accelerazione GPU migliora ulteriormente la velocità

{% hint style=&quot;info&quot; %}
**Chloros+ Velocità**: l&#x27;elaborazione parallela può essere 5-10 volte più veloce della modalità sequenziale per set di dati di grandi dimensioni. Un progetto di 500 immagini che richiede 2 ore in modalità gratuita può essere completato in 15-20 minuti con Chloros+.
{% endhint %}

***

## Cosa succede durante l&#x27;elaborazione

### Fase 1: Rilevamento del target

**Cosa fa Chloros:**

* Scansiona le immagini target contrassegnate (o tutte le immagini se nessuna è contrassegnata)
* Identifica i 4 pannelli di calibrazione in ciascun target
* Estrae i valori di riflettanza dai pannelli target
* Registra i timestamp dei target per la pianificazione della calibrazione

**Durata:** 1-30 secondi (con target contrassegnati), 5-30+ minuti (non contrassegnati)

### Fase 2: Debayering (conversione RAW)

**Cosa fa Chloros:**

* Converte i dati RAW del pattern Bayer in immagini RGB complete
* Applica un algoritmo di demosaicing di alta qualità
* Preserva la massima qualità e il massimo dettaglio dell&#x27;immagine

**Durata:** varia in base al numero di immagini e alla velocità della CPU

### Fase 3: Calibrazione

**Cosa fa Chloros:**

* **Correzione vignettatura**: rimuove l&#x27;oscuramento dell&#x27;obiettivo ai bordi
* **Calibrazione riflettanza**: normalizza utilizzando i valori di riflettanza target
* Applica correzioni su tutte le bande/canali
* Utilizza un target di calibrazione appropriato per ogni immagine in base al timestamp

**Durata:** la maggior parte del tempo di elaborazione

### Fase 4: calcolo dell&#x27;indice

**Cosa fa Chloros:**

* Calcola gli indici multispettrali configurati (NDVI, NDRE, ecc.)
* Applica la matematica delle bande alle immagini calibrate
* Genera immagini indice per ciascun indice selezionato

**Durata:** pochi secondi per immagine

### Fase 5: esportazione

**Cosa fa Chloros:**

* Salva le immagini calibrate nel formato selezionato
* Esporta le immagini indice con i colori LUT configurati
* Scrive i file nelle sottocartelle del modello di fotocamera
* Conserva i nomi dei file originali con i suffissi

**Durata:** varia in base al formato di esportazione e alle dimensioni del file

***

## Comportamento di elaborazione

### Pipeline di elaborazione automatica

Una volta avviata, l&#x27;intera pipeline viene eseguita automaticamente:

* Non è necessaria alcuna interazione da parte dell&#x27;utente
* Tutti i passaggi configurati vengono eseguiti in sequenza
* Aggiornamenti sullo stato di avanzamento mostrati in tempo reale

### Utilizzo del computer durante l&#x27;elaborazione

**Modalità libera:**

* Utilizzo della CPU relativamente basso (single-threaded)
* Il computer rimane reattivo per altre attività
* È sicuro ridurre a icona Chloros e lavorare in altre applicazioni

**Chloros+ Modalità parallela:**

* Utilizzo elevato della CPU (multi-threaded, fino a 16 core)
* Con accelerazione GPU: utilizzo elevato della GPU
* Il computer potrebbe essere meno reattivo durante l&#x27;elaborazione
* Evitare di avviare altre attività che richiedono un uso intensivo della CPU

{% hint style=&quot;warning&quot; %}
**Suggerimento sulle prestazioni**: per ottenere le migliori prestazioni da Chloros+, chiudere le altre applicazioni e lasciare che Chloros utilizzi tutte le risorse di sistema.
{% endhint %}

### L&#x27;elaborazione non può essere messa in pausa

**Limitazioni importanti:**

* Una volta avviata, l&#x27;elaborazione non può essere messa in pausa.
* È possibile annullare l&#x27;elaborazione, ma i progressi andranno persi.
* I risultati parziali non vengono salvati.
* Se annullata, è necessario ricominciare dall&#x27;inizio.

**Suggerimento di pianificazione:** per progetti di grandi dimensioni, prendere in considerazione l&#x27;elaborazione in batch o l&#x27;utilizzo di CLI per un controllo migliore.

***

## Monitoraggio dell&#x27;elaborazione

Durante l&#x27;elaborazione, è possibile:

* **Visualizzare la barra di avanzamento**: vedere la percentuale di completamento complessiva
* **Visualizzare la fase corrente**: rilevamento, analisi, calibrazione o esportazione
* **Controllare la scheda del registro**: vedere i messaggi e gli avvisi dettagliati relativi all&#x27;elaborazione
* **Visualizzare in anteprima le immagini completate**: alcuni file di esportazione potrebbero essere visualizzati durante l&#x27;elaborazione

Per informazioni dettagliate sul monitoraggio, vedere [Monitoraggio dell&#x27;elaborazione](monitoring-the-processing.md).

***

## Annullamento dell&#x27;elaborazione

Se è necessario interrompere l&#x27;elaborazione:

### Come annullare

1. Individuare il **pulsante Interrompi/Annulla** (sostituisce il pulsante Avvia durante l&#x27;elaborazione)
2. Fare clic sul pulsante Interrompi
3. L&#x27;elaborazione si interrompe immediatamente
4. I risultati parziali vengono eliminati

### Quando annullare

**Motivi validi per annullare:**

* Si è resi conto di aver utilizzato impostazioni errate
* Si è dimenticato di contrassegnare le immagini di destinazione
* Sono state importate immagini errate
* Il sistema è troppo lento o non risponde

**Dopo l&#x27;annullamento:**

* Rivedere e correggere eventuali problemi
* Regolare le impostazioni secondo necessità
* Riavviare l&#x27;elaborazione dall&#x27;inizio
* Per un&#x27;esperienza ottimale, chiudere completamente Chloros e riavviare

{% hint style=&quot;warning&quot; %}
**Nessun risultato parziale**: l&#x27;annullamento elimina tutti i progressi. Chloros non salva le immagini elaborate parzialmente.
{% endhint %}

***

## Stime dei tempi di elaborazione

Il tempo di elaborazione effettivo varia notevolmente in base a:

* Numero di immagini
* Risoluzione delle immagini
* Formato di input RAW o JPG
* Modalità di elaborazione (Free o Chloros+)
* Velocità della CPU e numero di core
* Disponibilità della GPU (solo Chloros+)
* Numero di indici da calcolare
* Complessità del formato di esportazione

### Stime approssimative (Chloros+, immagini da 12 MP, CPU moderna)

| Numero di immagini | Modalità gratuita | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 immagini   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 immagini  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 immagini  | 1-1,5 ore | 20-30 min      | 10-15 min      |
| 500 immagini  | 2-3 ore   | 45-60 min      | 20-30 min      |
| 1000 immagini | 4-6 ore   | 1,5-2 ore      | 40-60 min      |

{% hint style=&quot;info&quot; %}
**Prima esecuzione**: l&#x27;elaborazione iniziale potrebbe richiedere più tempo poiché Chloros crea cache e profili. L&#x27;elaborazione successiva di set di dati simili sarà più veloce.
{% endhint %}

***

## Problemi comuni all&#x27;avvio

### Pulsante di avvio disabilitato (grigio)

**Possibili cause:**

* Nessuna immagine importata
* Backend non completamente avviato
* Elaborazione precedente ancora in esecuzione
* Progetto non completamente caricato

**Soluzioni:**

1. Attendere che il backend sia completamente inizializzato (controllare l&#x27;icona del menu principale)
2. Verificare che le immagini siano state importate nel File Browser
3. Riavviare Chloros se il pulsante rimane disabilitato
4. Controllare il log di debug per eventuali messaggi di errore

### L&#x27;elaborazione si avvia ma fallisce immediatamente

**Possibili cause:**

* Nessuna immagine valida nel progetto
* File immagine danneggiati
* Spazio su disco insufficiente
* Memoria insufficiente (RAM)

**Soluzioni:**

1. Controllare il log di debug <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> per eventuali messaggi di errore
2. Verificare lo spazio disponibile sul disco
3. Provare a elaborare un sottoinsieme più piccolo di immagini
4. Verificare che le immagini non siano danneggiate

### Avviso &quot;Nessun target rilevato&quot;

**Possibili cause:**

* Dimenticato di contrassegnare le immagini target
* Le immagini target non contengono target visibili
* Impostazioni di rilevamento dei target troppo rigide

**Soluzioni:**

1. Rivedere [Scelta delle immagini target](choosing-target-images.md)
2. Contrassegnare le immagini appropriate nella colonna Target
3. Verificare che i target siano visibili nelle immagini contrassegnate
4. Regolare le impostazioni di rilevamento dei target, se necessario

***

## Suggerimenti per un&#x27;elaborazione corretta

### Prima di iniziare

1. **Eseguire prima un test con un piccolo sottoinsieme** - Elaborare 10-20 immagini per verificare le impostazioni
2. **Controllare lo spazio disponibile sul disco** - Assicurarsi di avere uno spazio libero pari a 2-3 volte la dimensione del set di dati
3. **Chiudere le applicazioni non necessarie** - Liberare risorse di sistema
4. **Verificare le immagini target** - Visualizzare in anteprima i target contrassegnati per assicurarne la qualità
5. **Salvare il progetto** - Il progetto viene salvato automaticamente, ma è buona norma salvarlo manualmente.

### Durante l&#x27;elaborazione

1. **Evitare la sospensione del sistema** - Disattivare le modalità di risparmio energetico.
2. **Mantenere Chloros in primo piano** - O almeno visibile nella barra delle applicazioni.
3. **Controllare occasionalmente lo stato di avanzamento** - Verificare la presenza di avvisi o errori.
4. **Non caricare altre applicazioni pesanti** - Soprattutto con la modalità parallela Chloros+

### Chloros+ Accelerazione GPU

Se si utilizza l&#x27;accelerazione GPU NVIDIA:

1. Aggiornare i driver NVIDIA all&#x27;ultima versione
2. Assicurarsi che la GPU abbia 4 GB+ di VRAM
3. Chiudere le applicazioni che richiedono un uso intensivo della GPU (giochi, editing video)
4. Monitorare la temperatura della GPU (assicurarsi che il raffreddamento sia adeguato)

***

## Passaggi successivi

Una volta avviata l&#x27;elaborazione:

1. **Monitorare lo stato di avanzamento** - Vedere [Monitoraggio dell&#x27;elaborazione](monitoring-the-processing.md)
2. **Attendere il completamento** - L&#x27;elaborazione viene eseguita automaticamente
3. **Esaminare i risultati** - Vedere [Completamento dell&#x27;elaborazione](finishing-the-processing.md)

Per informazioni su cosa fare durante l&#x27;elaborazione, vedere [Monitoraggio dell&#x27;elaborazione](monitoring-the-processing.md).
