# Monitoraggio dell&#x27;elaborazione

Una volta avviata l&#x27;elaborazione, Chloros offre diversi modi per monitorare lo stato di avanzamento, verificare la presenza di eventuali problemi e comprendere cosa sta succedendo con il proprio set di dati. Questa pagina spiega come monitorare l&#x27;elaborazione e interpretare le informazioni fornite da Chloros.

## Panoramica della barra di avanzamento

La barra di avanzamento nell&#x27;intestazione superiore mostra lo stato di elaborazione in tempo reale e la percentuale di completamento.

### Barra di avanzamento in modalità gratuita

Per gli utenti senza licenza Chloros+:

**Visualizzazione dell&#x27;avanzamento in 2 fasi:**

1. **Rilevamento del target** - Ricerca dei target di calibrazione nelle immagini
2. **Elaborazione** - Applicazione delle correzioni ed esportazione

**La barra di avanzamento mostra:**

* Percentuale di completamento complessiva (0-100%)
* Nome della fase corrente
* Semplice visualizzazione con barra orizzontale

### Barra di avanzamento Chloros+

Per gli utenti con licenza Chloros+:

**Visualizzazione dell&#x27;avanzamento in 4 fasi:**

1. **Rilevamento** - Ricerca dei target di calibrazione
2. **Analisi** - Esame delle immagini e preparazione della pipeline
3. **Calibrazione** - Applicazione delle correzioni di vignettatura e riflettanza
4. **Esportazione** - Salvataggio dei file elaborati

**Funzionalità interattive:**

* **Passa con il mouse** sulla barra di avanzamento per visualizzare il pannello espanso a 4 fasi
* **Fare clic** sulla barra di avanzamento per bloccare/fissare il pannello espanso
* **Fare nuovamente clic** per sbloccare e nascondere automaticamente all&#x27;uscita del mouse
* Ogni fase mostra l&#x27;avanzamento individuale (0-100%)

***

## Comprensione di ogni fase di elaborazione

### Fase 1: Rilevamento (rilevamento del target)

**Cosa succede:**

* Chloros esegue la scansione delle immagini contrassegnate con la casella di controllo Target
* Gli algoritmi di visione artificiale identificano i 4 pannelli di calibrazione
* Valori di riflettanza estratti da ciascun pannello
* Registrazione dei timestamp dei target per una corretta pianificazione della calibrazione

**Durata:**

* Con target contrassegnati: 10-60 secondi
* Senza target contrassegnati: 5-30+ minuti (scansiona tutte le immagini)

**Indicatore di avanzamento:**

* Rilevamento: 0% → 100%
* Numero di immagini scansionate
* Conteggio dei target trovati

**Cosa tenere d&#x27;occhio:**

* Dovrebbe completarsi rapidamente se i target sono contrassegnati correttamente
* Se richiede troppo tempo, i target potrebbero non essere contrassegnati
* Controllare il log di debug per i messaggi &quot;Target trovato&quot;

### Fase 2: Analisi

**Cosa sta succedendo:**

* Lettura dei metadati EXIF delle immagini (timestamp, impostazioni di esposizione)
* Determinazione della strategia di calibrazione in base ai timestamp degli obiettivi
* Organizzazione della coda di elaborazione delle immagini
* Preparazione dei worker di elaborazione parallela (solo Chloros+)

**Durata:** 5-30 secondi

**Indicatore di avanzamento:**

* Analisi: 0% → 100%
* Fase veloce, di solito completa rapidamente

**Cosa tenere d&#x27;occhio:**

* Dovrebbe procedere in modo costante senza pause
* Gli avvisi relativi ai metadati mancanti appariranno nel registro di debug

### Fase 3: Calibrazione

**Cosa sta succedendo:**

* **Debayering**: Conversione del pattern Bayer RAW in 3 canali
* **Correzione vignettatura**: rimozione dell&#x27;oscuramento dei bordi dell&#x27;obiettivo
* **Calibrazione riflettanza**: normalizzazione con valori target
* **Calcolo indice**: calcolo indici multispettrali
* Elaborazione di ciascuna immagine attraverso l&#x27;intera pipeline

**Durata:** la maggior parte del tempo di elaborazione totale (60-80%)

**Indicatore di avanzamento:**

* Calibrazione: 0% → 100%
* Immagine corrente in elaborazione
* Immagini completate / Immagini totali

**Comportamento dell&#x27;elaborazione:**

* **Modalità libera**: elabora un&#x27;immagine alla volta in sequenza
* **Modalità Chloros+**: elabora fino a 16 immagini contemporaneamente
* **Accelerazione GPU**: accelera notevolmente questa fase

**Cosa tenere d&#x27;occhio:**

* Avanzamento costante attraverso il conteggio delle immagini
* Controllare il registro di debug per i messaggi di completamento per ogni immagine
* Avvisi relativi alla qualità dell&#x27;immagine o a problemi di calibrazione

### Fase 4: Esportazione

**Cosa sta succedendo:**

* Scrittura delle immagini calibrate su disco nel formato selezionato
* Esportazione delle immagini dell&#x27;indice multispettrale con colori LUT
* Creazione di sottocartelle del modello di fotocamera
* Conservazione dei nomi dei file originali con suffissi appropriati

**Durata:** 10-20% del tempo totale di elaborazione

**Indicatore di avanzamento:**

* Esportazione: 0% → 100%
* File in fase di scrittura
* Formato di esportazione e destinazione

**Cosa tenere d&#x27;occhio:**

* Avvisi relativi allo spazio su disco
* Errori di scrittura dei file
* Completamento di tutti gli output configurati

***

## Scheda Registro di debug

Il Registro di debug fornisce informazioni dettagliate sullo stato di avanzamento dell&#x27;elaborazione e su eventuali problemi riscontrati.

### Accesso al Registro di debug

1. Fare clic sull&#x27;icona **Registro di debug** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> nella barra laterale sinistra
2. Si apre il pannello del log che mostra i messaggi di elaborazione in tempo reale
3. Scorre automaticamente per mostrare i messaggi più recenti

### Comprensione dei messaggi di log

#### Messaggi informativi (bianchi/grigi)

Aggiornamenti di elaborazione normali:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Messaggi di avviso (gialli)

Problemi non critici che non interrompono l&#x27;elaborazione:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Azione:** rivedere gli avvisi dopo l&#x27;elaborazione, ma non interrompere

#### Messaggi di errore (Red)

Problemi critici che possono causare il fallimento dell&#x27;elaborazione:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Azione:** Interrompere l&#x27;elaborazione, risolvere l&#x27;errore, riavviare.

### Messaggi di log comuni

| Messaggio                          | Significato                                | Azione richiesta                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Target rilevato in \[nome file]&quot; | Target di calibrazione trovato con successo  | Nessuna - normale                                         |
| &quot;Elaborazione immagine X di Y&quot;        | Aggiornamento sullo stato di avanzamento                | Nessuna - normale                                         |
| &quot;Nessun target trovato&quot;               | Nessun target di calibrazione rilevato        | Contrassegnare le immagini target o disabilitare la calibrazione della riflettanza |
| &quot;Spazio su disco insufficiente&quot;        | Spazio di archiviazione insufficiente per l&#x27;output          | Liberare spazio su disco                                    |
| &quot;Salto del file danneggiato&quot;        | Il file immagine è danneggiato                  | Copiare nuovamente il file dalla scheda SD                             |
| &quot;Dati PPK applicati&quot;               | Correzioni GPS dal file .daq applicate | Nessuna - normale                                         |

### Copia dei dati di registro

Per copiare il registro per la risoluzione dei problemi o l&#x27;assistenza:

1. Aprire il pannello Registro di debug
2. Fare clic sul pulsante **&quot;Copia registro&quot;** (o fare clic con il pulsante destro del mouse → Seleziona tutto)
3. Incollare in un file di testo o in un&#x27;e-mail
4. Se necessario, inviare all&#x27;assistenza MAPIR

***

## Monitoraggio delle risorse di sistema

### Utilizzo della CPU

**Modalità libera:**

* 1 core della CPU a \~100%
* Altri core inattivi o disponibili
* Il sistema rimane reattivo

**Chloros+ Modalità parallela:**

* Più core all&#x27;80-100% (fino a 16 core)
* Elevato utilizzo complessivo della CPU
* Il sistema potrebbe risultare meno reattivo

**Per monitorare:**

* Windows Task Manager (Ctrl+Shift+Esc)
* Scheda Prestazioni → Sezione CPU
* Cercare i processi &quot;Chloros&quot; o &quot;chloros-backend&quot;

### Utilizzo della memoria (RAM)

**Utilizzo tipico:**

* Progetti di piccole dimensioni (&lt; 100 immagini): 2-4 GB
* Progetti di medie dimensioni (100-500 immagini): 4-8 GB
* Progetti di grandi dimensioni (oltre 500 immagini): 8-16 GB
* La modalità parallela Chloros+ utilizza più RAM

**Se la memoria è insufficiente:**

* Elaborare batch più piccoli
* Chiudere altre applicazioni
* Aumentare la RAM se si elaborano regolarmente grandi set di dati

### Utilizzo della GPU (Chloros+ con CUDA)

Quando l&#x27;accelerazione GPU è abilitata:

* La GPU NVIDIA mostra un utilizzo elevato (60-90%)
* L&#x27;utilizzo della VRAM aumenta (richiede 4 GB+ di VRAM)
* La fase di calibrazione è significativamente più veloce

**Per monitorare:**

* Icona NVIDIA nella barra delle applicazioni
* Task Manager → Prestazioni → GPU
* GPU-Z o strumento di monitoraggio simile

### I/O del disco

**Cosa aspettarsi:**

* Elevata lettura del disco durante la fase di analisi
* Elevata scrittura del disco durante la fase di esportazione
* SSD significativamente più veloce dell&#x27;HDD

**Suggerimento sulle prestazioni:**

* Utilizzare SSD per la cartella del progetto, quando possibile
* Evitare unità di rete per set di dati di grandi dimensioni
* Assicurarsi che il disco non sia quasi pieno (influisce sulla velocità di scrittura)

***

## Rilevamento di problemi durante l&#x27;elaborazione

### Segnali di avvertimento

**Progresso bloccato (nessuna modifica per più di 5 minuti):**

* Controllare il log di debug per eventuali errori
* Verificare lo spazio disponibile sul disco
* Controllare Task Manager per assicurarsi che Chloros sia in esecuzione

**Messaggi di errore frequenti:**

* Interrompere l&#x27;elaborazione e controllare gli errori
* Cause comuni: spazio su disco, file danneggiati, problemi di memoria
* Vedere la sezione Risoluzione dei problemi di seguito

**Il sistema non risponde:**

* Chloros+ in modalità parallela utilizza troppe risorse
* Valutare la possibilità di ridurre le attività simultanee o aggiornare l&#x27;hardware
* La modalità libera richiede meno risorse

### Quando interrompere l&#x27;elaborazione

Interrompere l&#x27;elaborazione se si verificano:

* ❌ Errori &quot;Disco pieno&quot; o &quot;Impossibile scrivere il file&quot;
* ❌ Errori ripetuti di danneggiamento dei file immagine
* ❌ Sistema completamente bloccato (non risponde)
* ❌ Configurazione errata delle impostazioni
* ❌ Importazione di immagini errate

**Come interrompere:**

1. Fare clic sul pulsante **Stop/Cancel** (sostituisce il pulsante Start)
2. L&#x27;elaborazione si interrompe, i progressi vengono persi
3. Risolvere i problemi e ricominciare dall&#x27;inizio

***

## Risoluzione dei problemi durante l&#x27;elaborazione

### L&#x27;elaborazione è molto lenta

**Possibili cause:**

* Immagini di destinazione non contrassegnate (scansione di tutte le immagini)
* Archiviazione su HDD invece che su SSD
* Risorse di sistema insufficienti
* Molti indici configurati
* Accesso all&#x27;unità di rete

**Soluzioni:**

1. Se appena avviato e in fase di rilevamento: annullare, contrassegnare gli obiettivi, riavviare
2. Per il futuro: utilizzare SSD, ridurre gli indici, aggiornare l&#x27;hardware
3. Considerare CLI per l&#x27;elaborazione in batch di grandi set di dati

### Avvisi &quot;Spazio su disco&quot;

**Soluzioni:**

1. Liberare immediatamente spazio su disco
2. Spostare il progetto su un&#x27;unità con più spazio
3. Ridurre il numero di indici da esportare
4. Utilizzare il formato JPG invece di TIFF (file più piccoli)

### Messaggi frequenti di &quot;file danneggiato&quot;

**Soluzioni:**

1. Copiare nuovamente le immagini dalla scheda SD per garantirne l&#x27;integrità
2. Verificare la presenza di errori nella scheda SD
3. Rimuovere i file danneggiati dal progetto
4. Continuare l&#x27;elaborazione delle immagini rimanenti

### Surriscaldamento/riduzione della velocità del sistema

**Soluzioni:**

1. Assicurarsi che la ventilazione sia adeguata
2. Pulire la polvere dalle prese d&#x27;aria del computer
3. Ridurre il carico di elaborazione (utilizzare la modalità Free invece di Chloros+)
4. Eseguire l&#x27;elaborazione nelle ore più fresche della giornata

***

## Notifica di completamento dell&#x27;elaborazione

Al termine dell&#x27;elaborazione:

* La barra di avanzamento raggiunge il 100%
* Il messaggio **&quot;Elaborazione completata&quot;** viene visualizzato nel registro di debug
* Il pulsante Avvia viene nuovamente abilitato
* Tutti i file di output si trovano nella sottocartella del modello di fotocamera

***

## Passaggi successivi

Una volta completata l&#x27;elaborazione:

1. **Esaminare i risultati** - Vedere [Completamento dell&#x27;elaborazione](finishing-the-processing.md)
2. **Controllare la cartella di output** - Verificare che tutti i file siano stati esportati correttamente
3. **Esaminare il registro di debug** - Verificare la presenza di eventuali avvisi o errori
4. **Visualizzare in anteprima le immagini elaborate** - Utilizzare Image Viewer o un software esterno

Per informazioni sulla revisione e l&#x27;utilizzo dei risultati elaborati, vedere [Completamento dell&#x27;elaborazione](finishing-the-processing.md).
