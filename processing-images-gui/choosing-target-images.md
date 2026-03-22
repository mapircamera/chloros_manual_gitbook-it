# Selezione delle immagini target

Contrassegnare le immagini che contengono target di calibrazione è un passaggio fondamentale che accelera notevolmente la pipeline di elaborazione di Chloros. Preselezionando le immagini target, si evita che Chloros debba scansionare ogni singola immagine del set di dati alla ricerca di target di calibrazione.

## Perché contrassegnare le immagini target?

### Velocità di elaborazione

Senza contrassegnare le immagini target, Chloros deve:

* Scansionare ogni singola immagine del progetto
* Eseguire algoritmi di rilevamento dei target su ciascuna immagine
* Controllare inutilmente centinaia o migliaia di immagini

**Risultato**: l&#x27;elaborazione può richiedere molto più tempo, specialmente per set di dati di grandi dimensioni.

### Con immagini target contrassegnate

Quando si seleziona la colonna Target per immagini specifiche:

* Chloros scansiona solo le immagini selezionate alla ricerca di target
* Il rilevamento dei target viene completato molto più velocemente
* Il tempo di elaborazione complessivo si riduce notevolmente

{% hint style="success" %}
**Miglioramento della velocità**: contrassegnare 2-3 immagini target in un set di dati di 500 immagini può ridurre il tempo di rilevamento dei target da oltre 30 minuti a meno di 1 minuto.
{% endhint %}

***

## Come contrassegnare le immagini dei target

### Passaggio 1: Identificare le immagini dei target

Esaminare le immagini importate nel File Browser e identificare quali immagini contengono target di calibrazione.

**Scenari comuni:*** **Target pre-acquisizione**: acquisito prima di iniziare la sessione
* **Target post-acquisizione**: acquisito dopo aver completato la sessione
* **Target sul campo**: target posizionati all&#x27;interno dell&#x27;area di acquisizione
* **Target multipli**: 2-3 immagini target per sessione (consigliato)

### Passaggio 2: Controlla la colonna Target

Per ogni immagine contenente un target di calibrazione:

1. Individua l&#x27;immagine nella tabella del File Browser
2. Trova la colonna **Target** (colonna più a destra)
3. Fai clic sulla casella di controllo nella colonna Target per quell&#x27;immagine
4. Ripeti l&#x27;operazione per tutte le immagini contenenti target

### Passaggio 3: Verifica la tua selezione

Prima dell&#x27;elaborazione, ricontrolla:

* [ ] Tutte le immagini con target di calibrazione sono selezionate
* [ ] Nessuna immagine non target è stata selezionata accidentalmente
* [ ] I target sono chiaramente visibili nelle immagini selezionate

***

## Best practice per le immagini target

### Linee guida per l&#x27;acquisizione dei target

**Tempistica:**

* Acquisisci le immagini target immediatamente prima e durante la sessione di acquisizione
* Nelle stesse condizioni di illuminazione del sensore di luce DAQ
* Idealmente, acquisisci le immagini dei target il più spesso possibile per ottenere i migliori risultati. In caso contrario, i dati del sensore di luce verranno utilizzati per regolare la calibrazione nel tempo.

**Posizione della fotocamera:**

* Tieni la fotocamera sopra il target in modo che sia centrato e occupi circa il 40-60% del centro dell&#x27;immagine.
* Mantieni la fotocamera parallela/nadir alla superficie del target

**Illuminazione:**

* Stessa illuminazione ambientale del sensore di luce DAQ
* Evitare ombre sulle superfici del bersaglio
* Non ostruire la fonte di luce con il proprio corpo, veicoli o vegetazione
* Le condizioni di cielo coperto forniscono i risultati più costanti

**Condizioni del bersaglio:**

* Mantenere i pannelli del bersaglio puliti e asciutti
* Tutti e 4 i pannelli devono essere chiaramente visibili e senza ostacoli
* Se possibile, posizionare i bersagli perpendicolarmente/nadir alla fonte di luce

### Quante immagini del bersaglio?

**Minimo:**1 immagine del bersaglio per sessione.**Consigliato:** 3-5 immagini del bersaglio per sessione.**Programma ottimale:**

* 3-5 immagini acquisite poco dopo l&#x27;avvio della registrazione del sensore di luce
* Ruotare la fotocamera tra un&#x27;acquisizione e l&#x27;altra per ottenere i migliori risultati
* Opzionale: periodicamente a metà sessione se le condizioni di illuminazione cambiano costantemente

***

## Utilizzo di più fotocamere

### Configurazioni a doppia fotocamera

Se si utilizzano due fotocamere MAPIR contemporaneamente (ad es., Survey3W RGN + Survey3N OCN):

1. Acquisire le immagini del bersaglio con **entrambe le telecamere** contemporaneamente
2. Utilizzare lo **stesso bersaglio fisico** per entrambe le telecamere
3. Contrassegnare le immagini del bersaglio per **entrambi i tipi di telecamera** nel File Browser
4. Chloros utilizzerà i bersagli appropriati per la calibrazione di ciascuna telecamera

### Colonna Modello di fotocamera

La colonna **Modello di fotocamera** aiuta a identificare quali immagini provengono da quale fotocamera:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* ecc.

Utilizza questa colonna per verificare di aver contrassegnato i target per ciascun tipo di telecamera nel tuo progetto.

***

## Impostazioni di rilevamento dei target

### Regolazione della sensibilità di rilevamento

Se Chloros non rileva correttamente i tuoi target, modifica queste impostazioni in [Impostazioni del progetto](adjusting-project-settings.md):**Area minima del campione di calibrazione:*** **Impostazione predefinita**: 25 pixel
* **Aumentare** se si ottengono rilevamenti errati su piccoli artefatti
* **Diminuire** se i target non vengono rilevati**Raggruppamento minimo dei target:*** **Impostazione predefinita**: 60
* **Aumentare** se i target vengono suddivisi in più rilevamenti
* **Ridurre** se i target con variazioni di colore non vengono rilevati completamente***

## Problemi comuni relativi alle immagini dei target

### Problema: Nessun target rilevato

**Possibili cause:**

* Immagini dei target non contrassegnate nel File Browser
* Target troppo piccolo nell&#x27;inquadratura (&lt; 30% dell&#x27;immagine)
* Illuminazione scadente (ombre, riflessi)
* Impostazioni di rilevamento dei target troppo rigide

**Soluzioni:**

1. Verificare che la colonna &quot;Target&quot; sia selezionata per le immagini corrette
2. Controllare la qualità dell&#x27;immagine del target nell&#x27;anteprima
3. Riprendere i target se la qualità è scarsa
4. Regolare le impostazioni di rilevamento dei target se necessario

### Problema: Rilevamenti di falsi target

**Possibili cause:**

* Edifici bianchi, veicoli o copertura del terreno scambiati per target
* Macchie luminose nella vegetazione
* Sensibilità di rilevamento troppo bassa

**Soluzioni:**

1. Contrassegnare solo le immagini dei bersagli effettivi per limitare l&#x27;ambito di rilevamento
2. Aumentare l&#x27;area minima del campione di calibrazione
3. Aumentare il valore minimo di raggruppamento dei bersagli
4. Assicurarsi che le immagini dei bersagli mostrino solo il bersaglio (minimo ingombro dello sfondo)

***

## Lista di controllo per la verifica

Prima di iniziare l&#x27;elaborazione, verificare la selezione delle immagini dei bersagli:

* [ ] Almeno 1 immagine di bersaglio contrassegnata per sessione
* [ ] Le caselle di controllo della colonna &quot;Target&quot; sono selezionate per tutte le immagini dei target
* [ ] Immagini dei target acquisite nello stesso periodo di tempo del rilevamento
* [ ] Target chiaramente visibili nell&#x27;anteprima quando cliccati
* [ ] Tutti e 4 i pannelli di calibrazione visibili in ciascuna immagine del target
* [ ] Nessuna ombra o ostruzione sui target
* [ ] Per doppia fotocamera: Target contrassegnati per entrambi i tipi di fotocamera

***

## Elaborazione senza target

### Elaborazione senza bersagli di calibrazione

Sebbene non sia raccomandato per lavori scientifici, è possibile eseguire l&#x27;elaborazione senza bersagli:

1. Lasciare deselezionate tutte le caselle di controllo della colonna &quot;Bersaglio&quot;
2. **Disattivare** &quot;Calibrazione della riflettanza&quot; nelle Impostazioni del progetto
3. La correzione della vignettatura verrà comunque applicata
4. L&#x27;output non sarà calibrato per la riflettanza assoluta

{% hint style="warning" %}
**Non raccomandato**: senza la calibrazione della riflettanza, i valori dei pixel rappresentano solo la luminosità relativa, non misurazioni scientifiche della riflettanza. Utilizzare i target di calibrazione per ottenere risultati accurati e ripetibili.
{% endhint %}

***

## Passaggi successivi

Una volta contrassegnate le immagini target:

1. **Controlla le impostazioni** - Vedi [Regolazione delle impostazioni del progetto](adjusting-project-settings.md)
2. **Avvia l&#x27;elaborazione** - Vedi [Avvio dell&#x27;elaborazione](starting-the-processing.md)
3. **Monitorare lo stato di avanzamento** - Vedi [Monitoraggio dell&#x27;elaborazione](monitoring-the-processing.md)

Per ulteriori informazioni sui target di calibrazione, vedi [Target di calibrazione](../calibration-targets.md).
