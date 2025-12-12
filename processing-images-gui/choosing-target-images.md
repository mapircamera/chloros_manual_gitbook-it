# Selezione delle immagini target

Contrassegnare le immagini che contengono target di calibrazione è un passaggio fondamentale che velocizza notevolmente il processo di elaborazione di Chloros. Preselezionando le immagini target, si elimina la necessità che Chloros esegua la scansione di ogni immagine nel set di dati alla ricerca di target di calibrazione.

## Perché contrassegnare le immagini target?

### Velocità di elaborazione

Senza contrassegnare le immagini target, Chloros deve:

* Scansionare ogni singola immagine nel progetto
* Eseguire algoritmi di rilevamento dei target su ogni immagine
* Controllare centinaia o migliaia di immagini inutilmente

**Risultato**: l&#x27;elaborazione può richiedere molto più tempo, specialmente per set di dati di grandi dimensioni.

### Con immagini target contrassegnate

Quando si seleziona la colonna Target per immagini specifiche:

* Chloros esegue la scansione solo delle immagini selezionate alla ricerca di target
* Il rilevamento dei target viene completato molto più rapidamente
* Il tempo di elaborazione complessivo viene notevolmente ridotto

{% suggerimento style=&quot;success&quot; %}
**Miglioramento della velocità**: contrassegnando 2-3 immagini target in un set di dati di 500 immagini è possibile ridurre il tempo di rilevamento dei target da oltre 30 minuti a meno di 1 minuto.
{% endhint %}

***

## Come contrassegnare le immagini target

### Passaggio 1: identificare le immagini target

Esaminare le immagini importate nel File Browser e identificare quali immagini contengono target di calibrazione.

**Scenari comuni:**

* **Obiettivo pre-acquisizione**: acquisito prima dell&#x27;inizio della sessione
* **Obiettivo post-acquisizione**: acquisito dopo il completamento della sessione
* **Obiettivi sul campo**: obiettivi posizionati all&#x27;interno dell&#x27;area di acquisizione
* **Obiettivi multipli**: 2-3 immagini target per sessione (consigliato)

### Passaggio 2: controllare la colonna Target

Per ogni immagine contenente un target di calibrazione:

1. Individuare l&#x27;immagine nella tabella del File Browser
2. Trovare la colonna **Target** (colonna all&#x27;estrema destra)
3. Fare clic sulla casella di controllo nella colonna Target per quell&#x27;immagine
4. Ripetere l&#x27;operazione per tutte le immagini contenenti target

### Passaggio 3: verificare la selezione

Prima dell&#x27;elaborazione, ricontrollare:

* [ ] Tutte le immagini con target di calibrazione sono selezionate
* [ ] Nessuna immagine non target è stata selezionata accidentalmente
* [ ] I target sono chiaramente visibili nelle immagini selezionate

***

## Best practice per le immagini target

### Linee guida per l&#x27;acquisizione dei target

**Tempistica:**

* Acquisire le immagini target immediatamente prima e durante la sessione di acquisizione
* Nelle stesse condizioni di illuminazione del sensore di luce DAQ
* Idealmente, acquisire le immagini target il più spesso possibile per ottenere i migliori risultati. In caso contrario, i dati del sensore di luce verranno utilizzati per regolare la calibrazione nel tempo.

**Posizione della fotocamera:**

* Tenere la fotocamera sopra il target in modo che sia centrata e riempia circa il 40-60% del centro dell&#x27;immagine.
* Mantenere la fotocamera parallela/nadir rispetto alla superficie del target

**Illuminazione:**

* Stessa illuminazione ambientale del sensore di luce DAQ
* Evitare ombre sulle superfici del bersaglio
* Non ostruire la fonte di luce con il proprio corpo, veicoli o vegetazione
* Le condizioni di cielo coperto forniscono i risultati più coerenti

**Condizioni del bersaglio:**

* Mantenere i pannelli del bersaglio puliti e asciutti
* Tutti e 4 i pannelli devono essere chiaramente visibili e senza ostacoli
* Se possibile, i bersagli devono essere perpendicolari/nadir rispetto alla fonte di luce

### Quante immagini del bersaglio?

**Minimo:** 1 immagine target per sessione. **Consigliato:** 3-5 immagini target per sessione.

**Programma di best practice:**

* 3-5 immagini acquisite poco dopo l&#x27;inizio della registrazione del sensore di luce
* Ruotare la telecamera tra un&#x27;acquisizione e l&#x27;altra per ottenere i migliori risultati
* Opzionale: periodicamente a metà sessione se le condizioni di illuminazione cambiano costantemente

***

## Utilizzo di più telecamere

### Configurazioni a doppia fotocamera

Se si utilizzano due fotocamere MAPIR contemporaneamente (ad esempio, Survey3W RGN + Survey3N OCN):

1. Acquisire le immagini target con **entrambe le telecamere** contemporaneamente
2. Utilizzare lo **stesso target fisico** per entrambe le telecamere
3. Contrassegnare le immagini target per **entrambi i tipi di telecamera** nel File Browser
4. Chloros utilizzerà i target appropriati per la calibrazione di ciascuna telecamera

### Colonna Modello fotocamera

La colonna **Modello fotocamera** aiuta a identificare quali immagini provengono da quale fotocamera:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* ecc.

Utilizzare questa colonna per verificare di aver contrassegnato i target per ciascun tipo di telecamera nel progetto.

***

## Impostazioni di rilevamento dei target

### Regolazione della sensibilità di rilevamento

Se Chloros non rileva correttamente i target, regolare queste impostazioni in [Impostazioni progetto](adjusting-project-settings.md):

**Area minima di campionamento della calibrazione:**

* **Impostazione predefinita**: 25 pixel
* **Aumentare** se si ottengono rilevamenti errati su piccoli artefatti
* **Diminuire** se i target non vengono rilevati

**Raggruppamento minimo dei target:**

* **Impostazione predefinita**: 60
* **Aumentare** se i target vengono suddivisi in più rilevamenti
* **Diminuire** se i target con variazioni di colore non vengono rilevati completamente

***

## Problemi comuni relativi alle immagini dei target

### Problema: nessun target rilevato

**Possibili cause:**

* Immagini dei target non contrassegnate nel File Browser
* Target troppo piccolo nell&#x27;inquadratura (&lt; 30% dell&#x27;immagine)
* Illuminazione scadente (ombre, riflessi)
* Impostazioni di rilevamento dei target troppo rigide

**Soluzioni:**

1. Verificare che la colonna Target sia selezionata per le immagini corrette
2. Controllare la qualità dell&#x27;immagine del target nell&#x27;anteprima
3. Catturare nuovamente i target se la qualità è scadente
4. Regolare le impostazioni di rilevamento dei target, se necessario

### Problema: rilevamenti di target falsi

**Possibili cause:**

* Edifici bianchi, veicoli o copertura del suolo scambiati per bersagli
* Macchie luminose nella vegetazione
* Sensibilità di rilevamento troppo bassa

**Soluzioni:**

1. Contrassegnare solo le immagini dei bersagli effettivi per limitare l&#x27;ambito di rilevamento
2. Aumentare l&#x27;area minima del campione di calibrazione
3. Aumentare il valore minimo di raggruppamento dei bersagli
4. Assicurarsi che le immagini dei bersagli mostrino solo il bersaglio (disturbo minimo dello sfondo)

***

## Lista di controllo per la verifica

Prima di avviare l&#x27;elaborazione, verificare la selezione delle immagini dei bersagli:

* [ ] Almeno 1 immagine del bersaglio contrassegnata per sessione
* [ ] Le caselle di controllo della colonna Bersaglio sono selezionate per tutte le immagini dei bersagli
* [ ] Immagini dei bersagli acquisite nello stesso intervallo di tempo del rilevamento
* [ ] Bersagli chiaramente visibili nell&#x27;anteprima quando si fa clic
* [ ] Tutti e 4 i pannelli di calibrazione visibili in ciascuna immagine del bersaglio
* [ ] Nessuna ombra o ostruzione sui target
* [ ] Per doppia fotocamera: target contrassegnati per entrambi i tipi di fotocamera

***

## Elaborazione senza target

### Elaborazione senza target di calibrazione

Sebbene non sia consigliabile per lavori scientifici, è possibile eseguire l&#x27;elaborazione senza target:

1. Lasciare deselezionate tutte le caselle di controllo della colonna Target
2. **Disattivare** &quot;Calibrazione della riflettanza&quot; nelle impostazioni del progetto
3. La correzione della vignettatura verrà comunque applicata
4. L&#x27;output non verrà calibrato per la riflettanza assoluta

{% hint style=&quot;warning&quot; %}
**Non raccomandato**: senza la calibrazione della riflettanza, i valori dei pixel rappresentano solo la luminosità relativa, non misurazioni scientifiche della riflettanza. Utilizzare i target di calibrazione per ottenere risultati accurati e ripetibili.
{% endhint %}

***

## Passaggi successivi

Dopo aver contrassegnato le immagini target:

1. **Rivedere le impostazioni** - Vedere [Regolazione delle impostazioni del progetto](adjusting-project-settings.md)
2. **Avviare l&#x27;elaborazione** - Vedere [Avvio dell&#x27;elaborazione](starting-the-processing.md)
3. **Monitorare lo stato di avanzamento** - Vedere [Monitoraggio dell&#x27;elaborazione](monitoring-the-processing.md)

Per ulteriori informazioni sui target di calibrazione, vedere [Target di calibrazione](../calibration-targets.md).
