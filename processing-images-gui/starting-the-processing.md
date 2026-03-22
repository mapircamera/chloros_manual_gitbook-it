# Avvio dell&#x27;elaborazione

Una volta importate le immagini, contrassegnati i target di calibrazione e configurate le impostazioni del progetto, sei pronto per iniziare l&#x27;elaborazione. Questa pagina ti guida nell&#x27;avvio della pipeline di elaborazione Chloros.

## Lista di controllo pre-elaborazione

Prima di fare clic sul pulsante Avvia, verifica che tutto sia pronto:

* [ ] **File importati** - Tutte le immagini sono visibili nel Browser dei file
* [ ] **Immagini target contrassegnate** - La colonna Target è stata selezionata per le immagini di calibrazione
* [ ] **Modelli di fotocamera rilevati** - La colonna Modello fotocamera mostra le fotocamere corrette
* [ ] **Impostazioni configurate** - Impostazioni del progetto riviste e regolate
* [ ] **Indici selezionati** - Indici multispettrali desiderati aggiunti (se necessario)
* [ ] **Formato di esportazione scelto** - Formato di output appropriato per il proprio flusso di lavoro

{% hint style="info" %}
**Suggerimento**: Cliccare su alcune immagini nel File Browser per verificare che siano state caricate correttamente prima dell&#x27;elaborazione.
{% endhint %}

***

## Avvio dell&#x27;elaborazione

### Individuare il pulsante Avvia

Il pulsante Avvia/Riproduci si trova nella barra di intestazione superiore di Chloros:

* Posizione: in alto al centro della finestra
* Icona: **pulsante Riproduci/Avvia** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Stato: il pulsante è attivo (illuminato) quando è pronto per l&#x27;elaborazione

### Clicca per avviare

1. Clicca sul **pulsante Riproduci/Avvia** nell&#x27;intestazione superiore
2. L&#x27;elaborazione inizia immediatamente
3. Il pulsante diventa inattivo (disattivato) durante l&#x27;elaborazione
4. La barra di avanzamento si aggiorna, mostrando lo stato dell&#x27;elaborazione

{% hint style="success" %}
**Elaborazione avviata**: una volta cliccato, Chloros gestisce automaticamente tutte le fasi di elaborazione: rilevamento del target, debayering, calibrazione, calcolo dell&#x27;indice ed esportazione.
{% endhint %}

***

## Comprendere le modalità di elaborazione

Chloros opera in due diverse modalità di elaborazione a seconda della licenza:

### Modalità gratuita (elaborazione sequenziale)

**Disponibile per tutti gli utenti**

**Come funziona:**

* Elabora le immagini una alla volta, in modo sequenziale
* Funzionamento a thread singolo
* Minore utilizzo di memoria

**La barra di avanzamento mostra 2 fasi:**

1.**Rilevamento target** - Scansione dei target di calibrazione
2. **Elaborazione** - Applicazione della calibrazione ed esportazione delle immagini**Tempo di elaborazione:**

* Molto più lento rispetto alla modalità parallela di Chloros+
* Adatto a set di dati di piccole e medie dimensioni (&lt; 200 immagini)

### Modalità Chloros+ (Elaborazione parallela)

**Richiede la licenza Chloros+**

**Come funziona:**

* Elabora più immagini contemporaneamente utilizzando una [pipeline di elaborazione a 4 thread](../processing-architecture/processing-pipeline.md)
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) seleziona automaticamente la strategia ottimale per il tuo hardware
* Accelerazione GPU (CUDA) con schede grafiche NVIDIA (desktop e Jetson)
* Scalabile da un Jetson Nano (1 worker) a un desktop con GPU da 12 GB+ (3-4 worker)

**La barra di avanzamento mostra 4 fasi** (corrispondenti ai 4 thread della pipeline):

1. **Rilevamento** (Thread 1) - Individuazione dei target di calibrazione
2. **Analisi** (Thread 2) - Esame dei metadati dell&#x27;immagine e calcolo della calibrazione
3. **Calibrazione** (Thread 3) - Debayering GPU, correzione della vignettatura, calcolo dell&#x27;indice
4. **Esportazione** (Thread 4) - Salvataggio delle immagini elaborate e degli indici**Interazione con la barra di avanzamento:*** **Passare il mouse** sulla barra per visualizzare il pannello a tendina dettagliato delle 4 fasi
* **Cliccare** sulla barra di avanzamento per bloccare il pannello a tendina in posizione
* **Cliccare nuovamente** per sbloccare e nascondere il pannello**Tempo di elaborazione:**

* Significativamente più veloce rispetto alla modalità gratuita
* Scalabile in base al numero di core della CPU
* L&#x27;accelerazione GPU migliora ulteriormente la velocità

{% hint style="info" %}
**Velocità di Chloros+**: L&#x27;elaborazione parallela può essere da 5 a 10 volte più veloce rispetto alla modalità sequenziale per set di dati di grandi dimensioni. Un progetto di 500 immagini che richiede 2 ore in modalità gratuita può essere completato in 15-20 minuti con Chloros+.
{% endhint %}

***

## Cosa succede durante l&#x27;elaborazione

### Fase 1: Rilevamento dei target

**Cosa fa Chloros:**

* Esegue la scansione delle immagini dei target contrassegnati (o di tutte le immagini se nessuna è contrassegnata)
* Identifica i 4 pannelli di calibrazione in ciascun bersaglio
* Estrae i valori di riflettanza dai pannelli dei bersagli
* Registra i timestamp dei bersagli per la pianificazione della calibrazione

**Durata:** 1-30 secondi (con bersagli contrassegnati), 5-30+ minuti (non contrassegnati)

### Fase 2: Debayering (conversione RAW)

**Cosa fa Chloros:**

* Converte i dati RAW con pattern Bayer in immagini RGB complete
* Applica un algoritmo di demosaicing di alta qualità
* Preserva la massima qualità dell&#x27;immagine e il massimo livello di dettaglio

**Durata:** Varia in base al numero di immagini e alla velocità della CPU

### Fase 3: Calibrazione

**Cosa fa Chloros:*** **Correzione della vignettatura**: Rimuove l&#x27;oscuramento dell&#x27;obiettivo ai bordi
* **Calibrazione della riflettanza**: Normalizza utilizzando i valori di riflettanza target
* Applica correzioni su tutte le bande/canali
* Utilizza il target di calibrazione appropriato per ciascuna immagine in base al timestamp

**Durata:** La maggior parte del tempo di elaborazione

### Fase 4: Calcolo degli indici

**Cosa fa Chloros:**

* Calcola gli indici multispettrali configurati (NDVI, NDRE, ecc.)
* Applica operazioni matematiche sulle bande alle immagini calibrate
* Genera immagini indice per ciascun indice selezionato

**Durata:** Pochi secondi per immagine

### Fase 5: Esportazione

**Cosa fa Chloros:**

* Salva le immagini calibrate nel formato selezionato
* Esporta le immagini indice con i colori LUT configurati
* Scrive i file nelle sottocartelle relative al modello di fotocamera
* Mantiene i nomi dei file originali con i suffissi

**Durata:** Varia in base al formato di esportazione e alle dimensioni del file***

## Comportamento di elaborazione

### Pipeline di elaborazione automatica

Una volta avviata, l&#x27;intera pipeline viene eseguita automaticamente:

* Non è richiesta alcuna interazione da parte dell&#x27;utente
* Tutti i passaggi configurati vengono eseguiti in sequenza
* Aggiornamenti sullo stato di avanzamento visualizzati in tempo reale

### Utilizzo del computer durante l&#x27;elaborazione

**Modalità libera:**

* Utilizzo della CPU relativamente basso (single-threaded)
* Il computer rimane reattivo per altre attività
* È sicuro ridurre a icona Chloros e lavorare in altre applicazioni

**Chloros+ Modalità parallela:**

* Elevato utilizzo della CPU (multi-threaded, fino a 16 core)
* Con accelerazione GPU: elevato utilizzo della GPU
* Il computer potrebbe essere meno reattivo durante l&#x27;elaborazione
* Evitare di avviare altre attività che richiedono un uso intensivo della CPU

{% hint style="warning" %}
**Suggerimento per le prestazioni**: per ottenere le migliori prestazioni, chiudere le altre applicazioni e consentire a Chloros+ di utilizzare tutte le risorse di sistema.
{% endhint %}

### L&#x27;elaborazione non può essere messa in pausa

**Limitazioni importanti:**

* Una volta avviata, l&#x27;elaborazione non può essere messa in pausa
* È possibile annullare l&#x27;elaborazione, ma i progressi andranno persi
* I risultati parziali non vengono salvati
* Se annullata, è necessario ricominciare dall&#x27;inizio

**Suggerimento per la pianificazione:** per progetti di grandi dimensioni, valutare l&#x27;elaborazione in batch o l&#x27;utilizzo di CLI per un controllo migliore.***

## Monitoraggio dell&#x27;elaborazione

Durante l&#x27;esecuzione dell&#x27;elaborazione, è possibile:

* **Osservare la barra di avanzamento** - Visualizzare la percentuale complessiva di completamento
* **Visualizzare la fase corrente** - Rilevamento, Analisi, Calibrazione o Esportazione
* **Controllare la scheda del registro** - Visualizzare messaggi e avvisi dettagliati sull&#x27;elaborazione
* **Visualizzare in anteprima le immagini completate** - Alcuni file di esportazione potrebbero apparire durante l&#x27;elaborazione

Per informazioni dettagliate sul monitoraggio, consultare [Monitoraggio dell&#x27;elaborazione](monitoring-the-processing.md).

***

## Annullamento dell&#x27;elaborazione

Se è necessario interrompere l&#x27;elaborazione:

### Come annullare

1. Individuare il **pulsante Stop/Annulla** (sostituisce il pulsante Avvia durante l&#x27;elaborazione)
2. Fare clic sul pulsante Stop
3. L&#x27;elaborazione si interrompe immediatamente
4. I risultati parziali vengono scartati

### Quando annullare

**Motivi validi per annullare:**

* Ci si è resi conto che sono state utilizzate impostazioni errate
* Si è dimenticato di contrassegnare le immagini di destinazione
* Sono state importate immagini errate
* Il sistema è troppo lento o non risponde

**Dopo l&#x27;annullamento:**

* Controlla e risolvi eventuali problemi
* Modifica le impostazioni secondo necessità
* Riavvia l&#x27;elaborazione dall&#x27;inizio
* Per un&#x27;esperienza ottimale, chiudi completamente Chloros e riavvia

{% hint style="warning" %}
**Nessun risultato parziale**: l&#x27;annullamento cancella tutti i progressi. Chloros non salva le immagini elaborate parzialmente.
{% endhint %}

***

## Stime dei tempi di elaborazione

Il tempo di elaborazione effettivo varia notevolmente in base a:

* Numero di immagini
* Risoluzione delle immagini
* Formato di input RAW o JPG
* Modalità di elaborazione (Versione gratuita vs Chloros+)
* Velocità della CPU e numero di core
* Disponibilità della GPU (solo Chloros+)
* Numero di indici da calcolare
* Complessità del formato di esportazione

### Stime approssimative (Chloros+, immagini da 12 MP, CPU moderna)

| Numero di immagini | Modalità Free | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 immagini   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 immagini  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 immagini  | 1-1,5 ore | 20-30 min      | 10-15 min      |
| 500 immagini  | 2-3 ore   | 45-60 min      | 20-30 min      |
| 1000 immagini | 4-6 ore   | 1,5-2 ore      | 40-60 min      |

{% hint style="info" %}
**Prima esecuzione**: l&#x27;elaborazione iniziale potrebbe richiedere più tempo poiché Chloros crea cache e profili. L&#x27;elaborazione successiva di set di dati simili sarà più veloce.
{% endhint %}

***

## Problemi comuni all&#x27;avvio

### Pulsante di avvio disabilitato (disattivato)

**Possibili cause:**

* Nessuna immagine importata
* Backend non completamente avviato
* Elaborazione precedente ancora in corso
* Progetto non completamente caricato

**Soluzioni:**

1. Attendere che il backend si inizializzi completamente (controllare l&#x27;icona nel menu principale)
2. Verificare che le immagini siano importate nel File Browser
3. Riavviare Chloros se il pulsante rimane disabilitato
4. Controllare il registro di debug per eventuali messaggi di errore

### L&#x27;elaborazione si avvia ma fallisce immediatamente

**Possibili cause:**

* Nessuna immagine valida nel progetto
* File immagine danneggiati
* Spazio su disco insufficiente
* Memoria insufficiente (RAM)

**Soluzioni:**

1. Controllare il registro di debug <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> per verificare la presenza di messaggi di errore
2. Verificare lo spazio disponibile su disco
3. Provare a elaborare un sottoinsieme più piccolo di immagini
4. Verificare che le immagini non siano danneggiate

### Avviso &quot;Nessun bersaglio rilevato&quot;

**Possibili cause:**

* Dimenticanza di contrassegnare le immagini bersaglio
* Le immagini bersaglio non contengono bersagli visibili
* Impostazioni di rilevamento dei bersagli troppo rigide

**Soluzioni:**

1. Consultare [Scelta delle immagini target](choosing-target-images.md)
2. Contrassegnare le immagini appropriate nella colonna Target
3. Verificare che i target siano visibili nelle immagini contrassegnate
4. Regolare le impostazioni di rilevamento dei target se necessario

***

## Suggerimenti per un&#x27;elaborazione di successo

### Prima di iniziare

1. **Eseguire prima un test con un piccolo sottoinsieme** - Elaborare 10-20 immagini per verificare le impostazioni
2. **Controllare lo spazio disponibile su disco** - Assicurarsi di avere a disposizione uno spazio libero pari a 2-3 volte la dimensione del set di dati
3. **Chiudere le applicazioni non necessarie** - Liberare risorse di sistema
4. **Verificare le immagini target** - Visualizzare in anteprima i target contrassegnati per assicurarne la qualità
5. **Salvare il progetto** - Il progetto viene salvato automaticamente, ma è buona norma salvarlo manualmente

### Durante l&#x27;elaborazione

1. **Evitare la sospensione del sistema** - Disattivare le modalità di risparmio energetico
2. **Mantenere Chloros in primo piano** - O almeno visibile nella barra delle applicazioni
3. **Monitorare occasionalmente lo stato di avanzamento** - Verificare la presenza di avvisi o errori
4. **Non caricare altre applicazioni pesanti** - Soprattutto con la modalità parallela di Chloros+

### Accelerazione GPU di Chloros+

Se si utilizza l&#x27;accelerazione GPU NVIDIA:

1. Aggiornare i driver NVIDIA all&#x27;ultima versione
2. Assicurarsi che la GPU disponga di almeno 4 GB di VRAM
3. Chiudere le applicazioni che richiedono un uso intensivo della GPU (giochi, editing video)
4. Monitorare la temperatura della GPU (assicurarsi che il raffreddamento sia adeguato)

***

## Passi successivi

Una volta avviata l&#x27;elaborazione:

1. **Monitorare lo stato di avanzamento** - Vedere [Monitoraggio dell&#x27;elaborazione](monitoring-the-processing.md)
2. **Attendere il completamento** - L&#x27;elaborazione viene eseguita automaticamente
3. **Esaminare i risultati** - Vedi [Completamento dell&#x27;elaborazione](finishing-the-processing.md)

Per informazioni su cosa fare durante l&#x27;elaborazione, vedi [Monitoraggio dell&#x27;elaborazione](monitoring-the-processing.md).
