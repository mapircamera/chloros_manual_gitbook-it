# Monitoraggio dell&#x27;elaborazione

Una volta avviata l&#x27;elaborazione, Chloros offre diversi modi per monitorare lo stato di avanzamento, verificare la presenza di eventuali problemi e comprendere cosa sta accadendo con il proprio set di dati. Questa pagina spiega come monitorare l&#x27;elaborazione e interpretare le informazioni fornite da Chloros.

## Panoramica della barra di avanzamento

La barra di avanzamento nell’intestazione superiore mostra lo stato di elaborazione in tempo reale e la percentuale di completamento. L’avanzamento viene trasmesso in tempo reale dal backend tramite Server-Sent Events (SSE), quindi la barra riflette ciò che la pipeline sta effettivamente facendo.

### Barra di avanzamento in modalità gratuita

Per gli utenti senza licenza Chloros+:

**Visualizzazione dell’avanzamento in 2 fasi:**

1.**Rilevamento dei target** - Individuazione dei target di calibrazione nelle immagini
2. **Elaborazione** - Applicazione delle correzioni ed esportazione**La barra di avanzamento mostra:**

* Percentuale complessiva di completamento (0-100%)
* Nome della fase corrente
* Semplice visualizzazione a barra orizzontale

### Barra di avanzamento Chloros+

Per gli utenti con licenza Chloros+:

**Visualizzazione dell’avanzamento in 4 fasi:**

1.**Rilevamento** - Individuazione dei target di calibrazione
2. **Analisi** - Esame delle immagini e preparazione della pipeline
3. **Calibrazione** - Applicazione delle correzioni di vignettatura e riflettanza
4. **Esportazione** - Salvataggio dei file elaborati**Funzionalità interattive:*** **Passare il mouse sopra** la barra di avanzamento per visualizzare il pannello espanso a 4 fasi
* **Fare clic** sulla barra di avanzamento per bloccare/fissare il pannello espanso
* **Fare nuovamente clic** per sbloccarlo e nasconderlo automaticamente all’allontanamento del mouse
* Ogni fase mostra lo stato di avanzamento individuale (0-100%)

{% hint style="info" %}
**Parità CLI**: durante l’esecuzione di `chloros-cli process`, gli stessi quattro thread riportano lo stato “Rilevamento”, “Analisi”, “Elaborazione” e &quot;Exporting&quot;, mentre `chloros-cli export-status` mostra in tempo reale lo stato di avanzamento dell’esportazione del thread 4 da un altro terminale. Consulta la [Guida di riferimento di CLI](../reference/cli-reference.md).
{% endhint %}

***

## Comprensione di ciascuna fase di elaborazione

{% hint style="info" %}
**Architettura a pipeline**: Queste 4 fasi dell’interfaccia grafica corrispondono alla [pipeline di elaborazione a 4 thread](../processing-architecture/processing-pipeline.md). Sui sistemi con accelerazione GPU, il thread 3 (Calibrazione) beneficia del [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md), che ottimizza l’elaborazione in base al proprio hardware specifico.
{% endhint %}

### Fase 1: Rilevamento (Rilevamento dei target)

**Cosa succede:**

* Chloros esegue la scansione delle immagini selezionate tramite la casella di controllo &quot;Target&quot; (tutte le immagini solo se nessuna è selezionata)
* Gli algoritmi di visione artificiale identificano i pannelli di calibrazione
* I valori di riflettanza vengono estratti da ciascun pannello
* Vengono registrati i timestamp dei bersagli per una corretta pianificazione della calibrazione

**Durata:**

* Con bersagli contrassegnati: 10-60 secondi
* Senza bersagli contrassegnati: 5-30+ minuti (scansiona tutte le immagini)

**Indicatore di avanzamento:**

* Rilevamento: 0% → 100%
* Numero di immagini scansionate (conta solo le immagini effettivamente scansionate)
* Numero di target individuati

**Cosa tenere d’occhio:**

* Dovrebbe completarsi rapidamente se i target sono contrassegnati correttamente
* Se l’operazione richiede troppo tempo, i target potrebbero non essere contrassegnati
* Controllare il registro di debug per i messaggi “Target trovato”

### Fase 2: Analisi

**Cosa sta succedendo:**

* Lettura dei metadati EXIF delle immagini (timestamp, impostazioni di esposizione)
* Determinazione della strategia di calibrazione in base ai timestamp dei bersagli e ai dati DAQ in discesa disponibili
* Organizzazione della coda di elaborazione delle immagini
* Preparazione dei worker di elaborazione parallela (solo Chloros+)

**Durata:** 5-30 secondi**Indicatore di avanzamento:**

* Analisi in corso: 0% → 100%
* Fase veloce, solitamente si completa rapidamente

**Cosa tenere d&#x27;occhio:**

* L&#x27;avanzamento dovrebbe essere costante senza pause
* Nel registro di debug appariranno avvisi relativi a metadati mancanti

### Fase 3: Calibrazione

**Cosa sta succedendo:*** **Debayering**: conversione del pattern RAW Bayer in 3 canali (saltata per i moduli mono LATTICE, con una nota)
* **Correzione della vignettatura**: rimozione dell’oscuramento ai bordi dell’obiettivo
* **Calibrazione della riflettanza**: normalizzazione rispetto ai valori target e/o al downwelling del DAQ
* **Calcolo degli indici**: calcolo degli indici multispettrali
* Elaborazione di ciascuna immagine attraverso l’intera pipeline

**Durata:** la maggior parte del tempo totale di elaborazione (60-80%)**Indicatore di avanzamento:**

* Calibrazione in corso: 0% → 100%
* Immagine attualmente in elaborazione
* Immagini completate / Totale immagini

**Comportamento di elaborazione:*** **Modalità libera**: Elabora un’immagine alla volta in modo sequenziale
* **Modalità Chloros+**: Esegue un pool di worker adattivo all’hardware — da 1 a 4 worker simultanei su sistemi GPU (in base alla VRAM), un worker per ogni core fisico (meno uno) su sistemi solo CPU. Vedi [Adattamento dinamico del calcolo](../processing-architecture/dynamic-compute-adaptation.md)
* **Accelerazione GPU**: accelera significativamente questa fase**Cosa tenere d’occhio:**

* Progresso costante nel conteggio delle immagini
* Controllare il log di debug per i messaggi di completamento per ogni singola immagine
* Avvisi relativi alla qualità delle immagini o a problemi di calibrazione

### Fase 4: Esportazione

**Cosa sta succedendo:**

* Scrittura sul disco delle immagini elaborate nel formato selezionato, man mano che vengono completate
* **LATTICE**: ogni fotogramma viene distribuito in tutti i prodotti abilitati (debayering / anteprima / radianza / riflettanza)
* Esportazione di immagini indice multispettrali con colori LUT
* Creazione dell’albero di output `<project>/<camera>/<format>/<Product>_Images/` — i file esportati mantengono il nome del file sorgente; la cartella identifica il prodotto

**Durata:** 10-20% del tempo totale di elaborazione**Indicatore di avanzamento:**

* Esportazione: 0% → 100%
* File in fase di scrittura
* Formato di esportazione e destinazione

**Cosa tenere d’occhio:**

* Avvisi relativi allo spazio su disco
* Errori di scrittura dei file
* Completamento di tutti gli output configurati

***

## Scheda «Log di debug»

Il log di debug fornisce informazioni dettagliate sullo stato di avanzamento dell’elaborazione e su eventuali problemi riscontrati. Anche i messaggi di avvio del backend vengono riprodotti nella console di log, quindi il log riporta l’intera cronologia anche se lo si apre in un secondo momento.

### Come accedere al log di debug

1. Fare clic sull’icona **Registro di debug**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

nella barra laterale sinistra
2. Si apre il pannello del registro che mostra i messaggi di elaborazione in tempo reale
3. Lo scorrimento automatico mostra i messaggi più recenti

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Comprendere i messaggi di log

Le righe di log Chloros sono precedute da tag tra parentesi che indicano il sottosistema — ad esempio `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. La riga più importante da conoscere è il **riepilogo dell’esecuzione**, visualizzato alla fine di ogni esecuzione (comprese quelle interrotte):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Seguono ulteriori righe di suggerimenti con il codice `[RUN-SUMMARY]` ogni volta che è necessaria una spiegazione — ad esempio, un&#x27;esecuzione che non ha prodotto alcun risultato o una telecamera il cui prodotto richiesto è stato saltato in quanto non applicabile. Le righe `[EXPORT-CHECK]` spiegano i salti per singola telecamera (ad es. perché una telecamera RGB non ha ottenuto alcun prodotto di radianza).

I livelli di gravità generali dei messaggi (gli esempi riportati di seguito sono illustrativi, non letterali):

#### Messaggi informativi (bianco/grigio)

Aggiornamenti normali sull’elaborazione: elaborazione avviata, bersagli rilevati (con conteggio dei pannelli), avanzamento della calibrazione per immagine, file esportati, elaborazione completata.

#### Messaggi di avviso (giallo)

Problemi non critici che non interrompono l’elaborazione — ad es. dati GPS mancanti in un fotogramma, un ampio intervallo temporale tra le immagini dei bersagli o basso contrasto in un pannello di calibrazione.

**Azione:** Esaminare gli avvisi dopo l’elaborazione, ma non interromperla

#### Messaggi di errore (Red)

Problemi critici che potrebbero causare il fallimento dell’elaborazione — ad es. disco pieno, un file immagine danneggiato o nessun bersaglio rilevato mentre era richiesta la calibrazione della riflettanza.

**Azione:** Interrompere l’elaborazione, risolvere l’errore, riavviare

### Situazioni comuni nel log

| Situazione                             | Significato                                       | Azione richiesta                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Target rilevato in \[nomefile]        | Target di calibrazione trovato con successo         | Nessuna - normale                                         |
| Barre di avanzamento per immagine              | Aggiornamento dell’avanzamento corrente                       | Nessuna - normale                                         |
| Nessun bersaglio trovato                      | Nessun bersaglio di calibrazione rilevato               | Contrassegnare le immagini dei bersagli o disabilitare la calibrazione della riflettanza |
| Spazio su disco insufficiente               | Spazio di archiviazione insufficiente per l&#x27;output                 | Liberare spazio su disco                                    |
| Salto del file danneggiato               | Il file immagine è danneggiato                         | Ricopiare il file dalla scheda SD                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Non è possibile elaborare un’acquisizione senza un fotogramma raw | Ripetere l’acquisizione con il fotogramma raw oppure utilizzare CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | L&#x27;esecuzione non ha prodotto alcun risultato — segnalata come errore con suggerimenti | Leggere le righe di suggerimento; verificare cosa è stato saltato e perché |

### Copia dei dati di log

Per copiare il log a scopo di risoluzione dei problemi o assistenza:

1. Aprire il pannello del log di debug
2. Fare clic sul pulsante **&quot;Copia log&quot;** (oppure clic con il tasto destro → Seleziona tutto)
3. Incollare in un file di testo o in un’e-mail
4. Inviare al supporto MAPIR se necessario

***

## Monitoraggio delle risorse di sistema

### Utilizzo della CPU

**Modalità libera:**

* 1 core della CPU a circa il 100%
* Gli altri core sono inattivi o disponibili
* Il sistema rimane reattivo

**Modalità parallela Chloros+:**

* Più core ad alto utilizzo — il numero dipende dalla strategia scelta da [Adattamento dinamico del calcolo](../processing-architecture/dynamic-compute-adaptation.md)
* Il sistema potrebbe sembrare meno reattivo

**Per monitorare:**

* Windows Task Manager (Ctrl+Maiusc+Esc)
* Scheda Prestazioni → Sezione CPU
* Cercare i processi &quot;Chloros&quot; o &quot;chloros-backend&quot;

### Utilizzo della memoria (RAM)

**Utilizzo tipico:**

* Progetti di piccole dimensioni (&lt; 100 immagini): 2-4 GB
* Progetti di medie dimensioni (100-500 immagini): 4-8 GB
* Progetti di grandi dimensioni (oltre 500 immagini): 8-16 GB
* La modalità parallela di Chloros+ richiede più RAM

**Se la memoria è insufficiente:**

* Elaborare batch più piccoli
* Chiudere le altre applicazioni
* Aumentare la RAM se si elaborano regolarmente set di dati di grandi dimensioni

### Utilizzo della GPU (Chloros+ con CUDA)

Quando l’accelerazione GPU è abilitata:

* La GPU NVIDIA mostra un elevato utilizzo (60-90%)
* L’utilizzo della VRAM aumenta (richiede almeno 4 GB di VRAM; almeno 7 GB per il debayering Texture Aware in parallelo)
* La fase di calibrazione è notevolmente più veloce

**Per monitorare:**

* Icona NVIDIA nella barra delle applicazioni
* Gestione attività → Prestazioni → GPU
* GPU-Z o uno strumento di monitoraggio simile

### I/O del disco

**Cosa aspettarsi:**

* Elevata lettura dal disco durante la fase di analisi
* Elevata scrittura sul disco durante la fase di esportazione
* Gli SSD sono significativamente più veloci degli HDD

**Suggerimento per le prestazioni:**

* Utilizzare un SSD per la cartella del progetto, quando possibile
* Evitare le unità di rete per set di dati di grandi dimensioni
* Assicurarsi che lo spazio libero sul disco non sia quasi esaurito (influisce sulla velocità di scrittura)

***

## Rilevamento dei problemi durante l’elaborazione

### Segnali di allarme

**Blocco dell’avanzamento (nessun cambiamento per più di 5 minuti):**

* Controllare il registro di debug per eventuali errori
* Verificare lo spazio disponibile su disco
* Controllare il Gestore attività per assicurarsi che Chloros sia in esecuzione

**I messaggi di errore compaiono frequentemente:**

* Interrompere l’elaborazione e esaminare gli errori
* Cause comuni: spazio su disco, file danneggiati, problemi di memoria
* Consultare la sezione Risoluzione dei problemi qui sotto

**Il sistema non risponde:**

* La modalità parallela di Chloros+ utilizza troppe risorse
* Valutare la possibilità di ridurre le attività simultanee o di aggiornare l’hardware
* La modalità libera richiede meno risorse

### Quando interrompere l’elaborazione

Interrompere l’elaborazione se si riscontrano:

* ❌ Errori &quot;Disco pieno&quot; o &quot;Impossibile scrivere il file&quot;
* ❌ Errori ripetuti di danneggiamento dei file immagine
* ❌ Sistema completamente bloccato (non risponde)
* ❌ Ci si è resi conto che sono state configurate impostazioni errate
* ❌ Immagini importate errate

**Come interrompere l’elaborazione:**

1. Fare clic sul**pulsante Interrompi** (che sostituisce il pulsante Avvia) — è sufficiente una volta
2. La barra mostra “Interruzione in corso...” mentre l’immagine in elaborazione viene completata, quindi l’esecuzione termina in stato di interruzione
3. I prodotti già esportati rimangono sul disco; il log riporta un codice `[RUN-SUMMARY]` che indica chiaramente cosa è stato completato
4. Risolvere i problemi e riavviare: l’elaborazione riparte dall’inizio

***

## Risoluzione dei problemi durante l’elaborazione

### L’elaborazione è molto lenta

**Possibili cause:**

* Immagini di destinazione non contrassegnate (viene eseguita la scansione di tutte le immagini)
* Utilizzo di un disco rigido (HDD) anziché di un SSD
* Risorse di sistema insufficienti
* Numero elevato di indici configurati
* Accesso a un&#x27;unità di rete

**Soluzioni:**

1. Se l’elaborazione è appena iniziata e si trova nella fase di rilevamento: interrompere, contrassegnare le immagini di destinazione, riavviare
2. Per il futuro: utilizzare un SSD, ridurre il numero di indici, aggiornare l’hardware
3. Valutare l’uso di CLI per l’elaborazione in batch di set di dati di grandi dimensioni

### Avvisi relativi allo “spazio su disco”

**Soluzioni:**

1. Liberare immediatamente spazio su disco
2. Spostare il progetto su un&#x27;unità con più spazio
3. Ridurre il numero di indici da esportare
4. Disabilitare i prodotti di esportazione LATTICE non necessari (Impostazioni progetto → Elaborazione)
5. Utilizzare il formato JPG anziché TIFF (file più piccoli)

### Messaggi frequenti relativi a “file danneggiati”

**Soluzioni:**

1. Ricopiare le immagini dalla scheda SD per garantirne l’integrità
2. Verificare la presenza di errori sulla scheda SD
3. Rimuovere i file danneggiati dal progetto
4. Continuare l’elaborazione delle immagini rimanenti

### Surriscaldamento del sistema / Riduzione delle prestazioni

**Soluzioni:**

1. Assicurarsi che la ventilazione sia adeguata
2. Rimuovere la polvere dalle prese d’aria del computer
3. Ridurre il carico di elaborazione (utilizzare la modalità Free invece di Chloros+)
4. Eseguire l’elaborazione nelle ore più fresche della giornata

***

## Notifica di completamento dell’elaborazione

Al termine dell’elaborazione:

* La barra di avanzamento raggiunge il 100%
* Le righe `[RUN-SUMMARY]` compaiono nel registro di debug con i conteggi finali
* Il pulsante «Avvia» torna ad essere attivo
* Tutti i file di output si trovano nella struttura di output del progetto relativa a ciascuna telecamera: `<project>/<camera>/<format>/<Product>_Images/`

***

## Passi successivi

Una volta completata l’elaborazione:

1. **Esamina i risultati** - Vedi [Conclusione dell’elaborazione](finishing-the-processing.md)
2. **Controlla la cartella di output** - Verifica che tutti i file siano stati esportati correttamente
3. **Esamina il log di debug** - Verifica la presenza di eventuali avvisi o errori
4. **Visualizza in anteprima le immagini elaborate** - Utilizza Image Viewer o un software esterno

Per informazioni su come esaminare e utilizzare i risultati elaborati, consulta [Completamento dell’elaborazione](finishing-the-processing.md).
