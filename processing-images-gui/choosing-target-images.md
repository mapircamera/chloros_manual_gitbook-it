# Selezione delle immagini con bersagli

Contrassegnando le immagini che contengono bersagli di calibrazione, si indica a Chloros esattamente dove cercarli. Quando nella colonna “Target” è selezionata almeno un’immagine, Chloros esegue la scansione **solo delle immagini selezionate**: in questo modo, contrassegnare i bersagli permette sia di velocizzare l’elaborazione sia di evitare che le immagini del rilevamento vengano scambiate per un bersaglio.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Perché contrassegnare le immagini bersaglio?

### Il contrassegno controlla la scansione

Quando si selezionano immagini specifiche nella colonna &quot;Target&quot;:

* Chloros esegue la scansione alla ricerca di bersagli solo sulle immagini selezionate
* Il rilevamento dei bersagli viene completato molto più rapidamente
* Le immagini di rilevamento non possono generare falsi positivi nel rilevamento dei bersagli

Se **nessuna** immagine è selezionata, Chloros ricorre alla scansione di tutte le immagini del progetto:

* Gli algoritmi di rilevamento dei bersagli vengono eseguiti su ogni immagine
* Centinaia o migliaia di immagini vengono controllate inutilmente
* L’elaborazione richiede molto più tempo, specialmente per set di dati di grandi dimensioni

{% hint style="success" %}
**Miglioramento della velocità**: contrassegnare 2-3 immagini di bersaglio in un set di dati da 500 immagini può ridurre il tempo di rilevamento dei bersagli da oltre 30 minuti a meno di 1 minuto.
{% endhint %}

***

## Come contrassegnare le immagini dei bersagli

### Passo 1: Identificare le immagini dei bersagli

Esaminare le immagini importate nel File Browser e identificare quali immagini contengono i bersagli di calibrazione.

**Scenari comuni:*** **Bersaglio pre-acquisizione**: Acquisito prima dell’inizio della sessione
* **Target post-acquisizione**: acquisito dopo il completamento della sessione
* **Target sul campo**: target posizionati all’interno dell’area di acquisizione
* **Target multipli**: 2-3 immagini di target per sessione (consigliato)

### Passaggio 2: Controllare la colonna **Target** <img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

Per ogni immagine contenente un bersaglio di calibrazione:

1. Individuare l’immagine nella tabella del File Browser
2. Individuare la colonna **Target** (colonna più a destra)
3. Fare clic sulla casella di controllo nella colonna **Target** relativa a quell’immagine
4. Ripetere l’operazione per tutte le immagini contenenti bersagli

### Passaggio 3: Verificare la selezione

Prima dell’elaborazione, ricontrollare che:

* [ ] Tutte le immagini con bersagli di calibrazione siano selezionate
* [ ] Nessuna immagine non contenente bersagli sia stata selezionata accidentalmente
* [ ] I bersagli siano chiaramente visibili nelle immagini selezionate

***

## LATTICE: i target sono opzionali quando un DAQ sta registrando

Per le telecamere multispettrali LATTICE, un target di calibrazione all’interno dell’inquadratura è **uno dei due** possibili riferimenti di riflettanza:

* **Target all’interno dell’inquadratura**: quando un’immagine di target contrassegnata supera i controlli di qualità (QA) di Chloros, il target diventa il**riferimento di riflettanza assoluto** per le immagini circostanti.
* **Irradianza discendente del DAQ**: quando non è presente alcun bersaglio (o il controllo di qualità non viene superato), Chloros calcola invece la riflettanza a partire dall’irradianza discendente del sensore di luce del DAQ (ρ = π·L/E). Se una registrazione `.daq` o DAQ-M `.csv` copre le vostre acquisizioni, otterrete una riflettanza calibrata**senza alcuna immagine di riferimento**.

Questo comportamento automatico è l’impostazione predefinita. In CLI / SDK corrisponde a `--reflectance-source auto`; è anche possibile forzare `target` (rigoroso — nessuna sostituzione DAQ) o `daq` (autorevole DAQ). Consultare il [CLI Riferimento](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**Geometrie dei target LATTICE**: oltre al classico rilevamento a pannello utilizzato per Survey3, l’elaborazione LATTICE supporta**target contrassegnati con ArUco**,**target con ROI fissa**e**target a striscia**, configurati per ogni progetto. Le scansioni della riflettanza dei target**misurate** per singola unità possono essere fornite tramite numero di serie (CLI: `--target-reflectance-dir`, uno `<serial>.csv` per ogni unità target), con gli spettri nominali T3/T4P come alternativa.

{% hint style="info" %}
**Modulo F988**: la riflettanza dell’F988 viene calibrata utilizzando un pannello di riflettanza in scena: poiché la banda si trova al di fuori dell’intervallo calibrato del sensore di luce DAQ, Chloros applica l’ultima acquisizione del pannello effettuata e la mantiene tra una rilevazione e l’altra. Se un modulo F988 viene elaborato solo con il DAQ, Chloros rifiuta la riflettanza basata sul DAQ per quella banda (motivo di esclusione `dls-uncalibrated-band-988`): il flusso di lavoro con il pannello è l’approccio supportato.
{% endhint %}

***

## Best practice per le immagini del bersaglio

### Linee guida per l’acquisizione del bersaglio

**Tempistica:**

* Acquisire le immagini del bersaglio immediatamente prima e durante tutta la sessione di acquisizione
* Nelle stesse condizioni di illuminazione del sensore di luce DAQ
* Idealmente, acquisire le immagini del bersaglio il più spesso possibile per ottenere i migliori risultati. In caso contrario, i dati del sensore di luce del DAQ verranno utilizzati per regolare la calibrazione nel tempo.

**Posizione della fotocamera:**

* Tenere la fotocamera sopra il bersaglio in modo che sia centrato e occupi circa il 40-60% del centro dell’immagine.
* Mantenere la fotocamera parallela o in posizione nadir rispetto alla superficie del bersaglio

**Illuminazione:**

* Stessa illuminazione ambientale del sensore di luce del DAQ
* Evitare ombre sulle superfici del bersaglio
* Non ostruire la fonte di luce con il proprio corpo, un veicolo o la vegetazione
* Le condizioni di cielo coperto garantiscono risultati più costanti

**Condizioni del bersaglio:**

* Mantenere i pannelli del bersaglio puliti e asciutti
* Tutti i pannelli del bersaglio (ad es. tutti e 4 su un T4) devono essere chiaramente visibili e liberi da ostacoli
* Se possibile, posizionare i bersagli perpendicolarmente o in posizione nadir rispetto alla fonte di luce

### Quante immagini del bersaglio?

**Minimo:**1 immagine del bersaglio per sessione.**Consigliato:** 3-5 immagini del bersaglio per sessione.**Programma consigliato:**

* 3-5 immagini acquisite poco dopo l’avvio della registrazione del sensore di luce
* Ruotare la fotocamera tra un’acquisizione e l’altra per ottenere i migliori risultati
* Facoltativo: periodicamente a metà sessione se le condizioni di illuminazione cambiano costantemente

***

## Utilizzo di più telecamere

### Configurazioni a doppia telecamera

Se si utilizzano contemporaneamente due telecamere MAPIR (ad es., Survey3W RGN + Survey3N OCN):

1. Acquisire le immagini del bersaglio con **entrambe le telecamere** contemporaneamente
2. Utilizzare lo **stesso bersaglio fisico** per entrambe le telecamere
3. Contrassegnare le immagini dei bersagli per **entrambi i tipi di telecamera** nel File Browser
4. Chloros utilizzerà i bersagli appropriati per la calibrazione di ciascuna telecamera

### Colonna “Modello della telecamera”

La colonna **“Modello della telecamera”** aiuta a identificare quali immagini provengono da quale telecamera:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* ecc.

Utilizza questa colonna per verificare di aver contrassegnato i bersagli per ciascun tipo di telecamera nel tuo progetto.

***

## Impostazioni di rilevamento dei bersagli

### Regolazione della sensibilità di rilevamento

Se Chloros non rileva correttamente i bersagli, regolare queste impostazioni in [Impostazioni del progetto](adjusting-project-settings.md):**Area minima del campione di calibrazione (px):*** **Impostazione predefinita**: 25 pixel
* **Aumentare** se si verificano falsi rilevamenti su piccoli artefatti
* **Diminuire** se i bersagli non vengono rilevati**Raggruppamento minimo dei bersagli (0-100):*** **Impostazione predefinita**: 60
* **Aumenta** se i target vengono suddivisi in più rilevamenti
* **Riduci** se i target con variazioni di colore non vengono rilevati completamente

{% hint style="info" %}
**Suggerimento per CLI**: `chloros-cli process` accetta gli stessi parametri (`--min-target-size`, `--target-clustering`), e il suo flag `--target`/`--targets` contrassegna un&#x27;intera cartella di input come &quot;solo pannello dei target&quot;. Vedi la [Guida di riferimento di CLI](../reference/cli-reference.md).
{% endhint %}

***

## Problemi comuni relativi alle immagini bersaglio

### Problema: Nessun bersaglio rilevato

**Possibili cause:**

* Immagini di riferimento non contrassegnate nel File Browser
* Obiettivo troppo piccolo nell’inquadratura (&lt; 30% dell’immagine)
* Illuminazione inadeguata (ombre, riflessi)
* Impostazioni di rilevamento degli obiettivi troppo rigide

**Soluzioni:**

1. Verificare che la colonna “Obiettivo” sia selezionata per le immagini corrette
2. Verificare la qualità delle immagini dei target nell’anteprima
3. Riprendere i target se la qualità è scarsa
4. Regolare le impostazioni di rilevamento dei target, se necessario

### Problema: Rilevamenti errati dei target

**Possibili cause:**

* Edifici bianchi, veicoli o copertura del terreno scambiati per target
* Macchie luminose nella vegetazione
* Sensibilità di rilevamento troppo bassa

**Soluzioni:**

1. Contrassegnare solo le immagini dei bersagli effettivi: verranno scansionate solo le immagini selezionate
2. Aumentare l’area minima del campione di calibrazione
3. Aumentare il valore minimo di raggruppamento dei bersagli
4. Assicurarsi che le immagini dei bersagli mostrino solo il bersaglio (con il minimo disturbo di sfondo)

***

## Lista di controllo per la verifica

Prima di avviare l’elaborazione, verificare la selezione delle immagini dei bersagli:

* [ ] Almeno 1 immagine di bersaglio contrassegnata per sessione (oppure, per LATTICE, una registrazione `.daq`/`.csv` che copra la sessione)
* [ ] Le caselle di controllo della colonna «bersaglio» sono spuntate per tutte le immagini dei bersagli
* [ ] Le immagini dei bersagli sono state acquisite nello stesso arco di tempo dell’indagine
* [ ] I bersagli sono chiaramente visibili nell’anteprima quando cliccati
* [ ] Tutti i pannelli di calibrazione sono visibili in ciascuna immagine del bersaglio
* [ ] Non sono presenti ombre o ostacoli sui bersagli
* [ ] Per configurazione a doppia fotocamera: i bersagli sono contrassegnati per entrambi i tipi di fotocamera

***

## Elaborazione senza bersagli

### LATTICE: con una registrazione DAQ

Se un sensore di luce DAQ ha registrato l’irraggiamento discendente durante le acquisizioni LATTICE, non è necessario alcun bersaglio:

1. Importare il file `.daq` (o DAQ-M `.csv`) contenente le immagini
2. Lasciare deselezionata la colonna “Target”
3. La riflettanza viene calcolata automaticamente dal riferimento di irraggiamento discendente del DAQ
4. La radianza non richiede mai un target né un DAQ: deriva esclusivamente dalla calibrazione radiometrica di fabbrica della fotocamera

### Elaborazione senza alcun riferimento

È anche possibile eseguire l’elaborazione senza target e senza un DAQ:

1. Lasciare deselezionate tutte le caselle di controllo della colonna &quot;Target&quot;
2. **Disattivare** «Calibrazione della riflettanza / bilanciamento del bianco» nelle Impostazioni del progetto: il rilevamento dei target verrà così completamente saltato
3. La correzione della vignettatura verrà comunque applicata
4. L’output non sarà calibrato per la riflettanza assoluta (LATTICE multispettrale esporta comunque prodotti debayered, di anteprima e di radianza)

{% hint style="warning" %}
**Non raccomandato per attività scientifiche Survey3**: senza calibrazione della riflettanza, i valori dei pixel dell’Survey3e rappresentano solo la luminosità relativa, non misurazioni scientifiche della riflettanza. Utilizzate i target di calibrazione (o, per LATTICE, un sensore di luce DAQ) per ottenere risultati accurati e ripetibili.
{% endhint %}

***

## Passi successivi

Una volta contrassegnate le immagini di riferimento:

1. **Verifica le impostazioni** - Vedi [Regolazione delle impostazioni del progetto](adjusting-project-settings.md)
2. **Avvia l’elaborazione** - Vedi [Avvio dell’elaborazione](starting-the-processing.md)
3. **Monitorare lo stato di avanzamento** - Vedi [Monitoraggio dell’elaborazione](monitoring-the-processing.md)

Per ulteriori informazioni sui target di calibrazione stessi, vedi [Target di calibrazione](../calibration-targets.md).
