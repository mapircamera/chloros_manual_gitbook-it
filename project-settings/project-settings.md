# Impostazioni del progetto

La barra laterale &quot;Impostazioni del progetto&quot; (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

) in Chloros consente di configurare tutti gli aspetti relativi all&#x27;elaborazione delle immagini, al rilevamento dei target di calibrazione, al calcolo degli indici multispettrali e alle opzioni di esportazione per il proprio progetto. Queste impostazioni vengono salvate insieme al progetto e possono essere salvate come modelli da riutilizzare in più progetti.

## Come accedere alle impostazioni del progetto

Per accedere alle impostazioni del progetto:

1. Aprire un progetto in Chloros
2. Fare clic sulla scheda **Impostazioni del progetto**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

nella barra laterale sinistra
3. Il pannello delle impostazioni mostrerà tutte le opzioni di configurazione disponibili organizzate per categoria

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Le impostazioni che dipendono da altre impostazioni sono disattivate (di colore grigio).** Quando un&#x27;opzione principale rende impossibile una determinata impostazione (ad esempio, deselezionando *Calibrazione della riflettanza / bilanciamento del bianco* non è più possibile *Esportare la riflettanza*), il controllo dipendente viene disabilitato e il suo tooltip indica l&#x27;opzione che deve essere modificata.
{% endhint %}

***

## Visualizzazione

### Risoluzione delle miniature delle immagini

* **Tipo**: Selezione da menu a tendina
* **Opzioni**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Impostazione predefinita**: Predefinita (512 px)
* **Descrizione**: Risoluzione (lato più lungo, in pixel) alla quale vengono visualizzate le miniature nella griglia delle immagini. Valori più elevati garantiscono una maggiore nitidezza quando si ingrandisce l’immagine, ma rallentano il caricamento e consumano più memoria. La risoluzione completa corrisponde alle dimensioni originali dell’immagine.
* **Nota**: Solo a scopo di visualizzazione — questa impostazione non influisce mai sull’elaborazione o sui file esportati.***

## Rilevamento dei target

Queste impostazioni controllano il modo in cui Chloros rileva ed elabora i target di calibrazione nelle immagini. Entrambe sono attive solo quando è abilitata la **calibrazione della riflettanza / bilanciamento del bianco** (in caso contrario sono disattivate, poiché il rilevamento dei target viene completamente ignorato).

### Area minima del campione di calibrazione (px)

* **Tipo**: Numero
* **Intervallo**: da 0 a 10.000 pixel
* **Impostazione predefinita**: 25 pixel
* **Descrizione**: Imposta l’area minima (in pixel) necessaria affinché una regione rilevata sia considerata un campione valido di bersaglio di calibrazione. Valori più bassi consentiranno di rilevare bersagli più piccoli, ma potrebbero aumentare i falsi positivi. Valori più alti richiedono regioni bersaglio più grandi e più nitide per il rilevamento.
* **Quando regolare**:
  * Aumentare se si ottengono rilevamenti errati su piccoli artefatti dell’immagine
  * Diminuire se i bersagli di calibrazione appaiono piccoli nelle immagini e non vengono rilevati

### Raggruppamento minimo dei bersagli (0-100)

* **Tipo**: Numero
* **Intervallo**: da 0 a 100
* **Impostazione predefinita**: 60
* **Descrizione**: Controlla la soglia di raggruppamento per unire regioni di colore simile durante il rilevamento dei bersagli di calibrazione. Valori più alti richiedono che colori più simili vengano raggruppati insieme, determinando un rilevamento dei bersagli più conservativo. Valori più bassi consentono una maggiore variazione di colore all’interno di un gruppo di bersagli.
* **Quando regolare**:
  * Aumentare se i bersagli di calibrazione vengono suddivisi in più rilevamenti
  * Diminuire se i bersagli di calibrazione con variazioni di colore non vengono rilevati completamente

***

## Elaborazione

Queste impostazioni controllano il modo in cui Chloros elabora e calibra le immagini.

### Correzione della vignettatura

* **Tipo**: Casella di selezione
* **Impostazione predefinita**: Abilitata (selezionata)
* **Descrizione**: applica la correzione della vignettatura per compensare l’oscuramento dell’obiettivo ai bordi delle immagini. La vignettatura è un fenomeno ottico comune in cui gli angoli e i bordi di un’immagine appaiono più scuri rispetto al centro a causa delle caratteristiche dell’obiettivo.
* **Effetto collaterale**: questa opzione seleziona anche quale *prodotto di ripiego non calibrato* viene generato da un&#x27;esecuzione (vedi sotto).

### Calibrazione della riflettanza / bilanciamento del bianco

* **Tipo**: Casella di selezione
* **Impostazione predefinita**: Abilitato (selezionato)
* **Descrizione**: Abilita la calibrazione della riflettanza — a partire dai bersagli di calibrazione rilevati nell’inquadratura e/o dai dati di luce incidente del sensore DAQ, a seconda della telecamera e di ciò che è disponibile. Ciò normalizza i valori di riflettanza in tutto il set di dati e garantisce misurazioni coerenti indipendentemente dalle condizioni di illuminazione.
* **Quando disabilitata**: il rilevamento dei target viene completamente saltato e**nessuna telecamera può generare alcun prodotto di riflettanza** — sia Survey3 basato sui target che LATTICE basato sul DAQ. Le impostazioni correlate (*Esporta riflettanza*, *Intervallo minimo di ricalibrazione* e le soglie di rilevamento dei bersagli) sono disattivate (in grigio).

### Prodotti di fallback non calibrati: Esporta risposta del sensore / Esporta con correzione della vignettatura

* **Tipo**: Due caselle di controllo
* **Impostazioni predefinite**: Entrambe abilitate (spuntate)
* **Descrizione**: Quando non è possibile calibrare la riflettanza di un fotogramma (non è stato trovato alcun bersaglio di calibrazione oppure la calibrazione della riflettanza è disattivata), questo viene salvato come *prodotto di riserva non calibrato*. **Per ogni esecuzione, per ogni modello di fotocamera, esiste esattamente uno dei due prodotti di riserva**, scelto dall’opzione *Correzione della vignettatura*:
  * Correzione della vignettatura **attiva**→ `Vignette_Corrected_Images/` (regolata da**Esporta con correzione della vignettatura**)
  * Correzione vignettatura **disattivata**→ `Sensor_Response_Images/` (regolata dall’opzione**Esporta risposta del sensore**)
* Il prodotto di riserva non attivo è disattivato (in grigio). Deselezionando quello attivo si impedisce del tutto la scrittura di quel file.

### Prodotti di esportazione LATTICE

Per i progetti contenenti acquisizioni LATTICE, ogni fotogramma LATTICE importato viene distribuito in tutti i prodotti abilitati **e applicabili**in un unico ciclo di elaborazione. Quattro caselle di controllo gestiscono la distribuzione (tutte**attive** per impostazione predefinita):

| Impostazione | Cartella di output | Cosa esporta |
| --- | --- | --- |
| **Esporta debayered** | `Debayered_Images/` | L’immagine lineare debayered. Si applica a RGB e alle telecamere multispettrali. |
| **Esporta anteprima** | `Preview_Images/` | L’anteprima visualizzata. RGB = bilanciamento del bianco (illuminante DAQ se disponibile, altrimenti mondo grigio) + gamma; multispettrale = estensione a falsi colori. |
| **Radianza di esportazione** | `Radiance_Images/` | Radianza spettrale in Float32 espressa in W/m²/sr/nm. Solo multispettrale (M3C/M3M) — non applicabile ai master RGB. Viene sempre scritto come TIFF a 32 bit indipendentemente dall’impostazione *Formato immagine calibrato*. |
| **Riflettanza di esportazione**| `Reflectance_Calibrated_Images/` | Riflettanza Uint16, scalata in modo che**32768 = riflettanza 1,0** (contrassegnata come XMP `Chloros:PixelScale`). Solo multispettrale, scritto quando un record discendente `.daq` corrispondente (o un target all’interno del fotogramma che ha superato il controllo qualità) copre il fotogramma. |

* Le telecamere master RGB emettono dati debayered + anteprima; per queste, la radianza e la riflettanza vengono ignorate in quanto non applicabili.
* La profondità di bit del debayering/anteprima segue l’impostazione *Formato immagine calibrato*; la radianza è sempre float32.
* L’elaborazione Survey3 non è influenzata da questi quattro selettori.

Gli stessi quattro interruttori esistono in modalità headless come `chloros-cli process --debayered / --preview / --radiance / --reflectance` e come parametri corrispondenti di SDK. Hanno sostituito il vecchio flag `--radiometric-output`, che non esiste più.

{% hint style="warning" %}
**Disattivando tutti i prodotti applicabili, l’esecuzione fallisce.** A partire dalla versione 1.2.0, un’esecuzione di elaborazione che ha richiesto prodotti ma non ha scritto alcun prodotto immagine segnala un errore e CLI termina con un valore diverso da zero, invece di segnalare un successo silenzioso. Il log indica il prodotto che non è stato possibile generare e il motivo. Un&#x27;esecuzione deliberatamente limitata ai soli metadati (senza alcuna richiesta) viene comunque considerata riuscita.
{% endhint %}

### Sorgente di riflettanza (impostazione del progetto, configurata tramite CLI/SDK)

Il progetto memorizza anche quale **riferimento di riflettanza** utilizza il prodotto di riflettanza LATTICE. Non esiste un controllo dedicato nel pannello delle impostazioni; il valore è memorizzato nella configurazione del progetto come `Processing → "Target reflectance source"` e viene impostato tramite `chloros-cli process --reflectance-source {auto,target,daq}` o il parametro SDK&#x27;s `reflectance_source`:

* **`auto`** (impostazione predefinita): un bersaglio di calibrazione all’interno dell’inquadratura che supera il controllo di qualità (QA) diventa il riferimento assoluto; in assenza di bersaglio o in caso di fallimento del controllo di qualità, si ricorre al rapporto di divisione del flusso discendente del DAQ (ρ = πL/E).
* **`target`**: riflettanza rigorosamente basata sul bersaglio — nessuna sostituzione da parte del DAQ.
* **`daq`**: riflettanza determinata dal DAQ; i target presenti nell’inquadratura non vengono utilizzati come riferimento.

Il valore memorizzato viene confrontato senza distinzione tra maiuscole e minuscole e alcune varianti ortografiche sono accettate come alias: `target`, `target_image`, `empirical` e `empirical_line` indicano tutti **target**; `daq`, `dls`, `light_sensor` e `sensor` indicano tutti**daq**. Qualsiasi altra voce — compresa l’assenza di una chiave — viene risolta come**auto**.

Le scansioni dei target **misurati** per unità vengono cercate in base al numero di serie/QR dell&#x27;unità di destinazione, come `<serial>.csv`, in tre posizioni: la directory indicata con `--target-reflectance-dir` (memorizzata come `Processing → "Target reflectance dir"`), nella cartella `target_reflectance/` del progetto stesso e nel percorso specificato dalla variabile d’ambiente `CHLOROS_TARGET_REFLECTANCE_DIR`. Se non esiste alcuna scansione misurata per quell’unità, viene utilizzata invece la curva nominale pubblicata per il modello di riferimento.

### Metodo di demosaicing

* **Tipo**: Selezione da menu a tendina
* **Opzioni**:
  * Standard (Veloce, Qualità media)
  * Texture Aware (Lento, Massima qualità) \[Chloros+]
* **Impostazione predefinita**: Standard (Veloce, Qualità media)
* **Descrizione**: Seleziona l’algoritmo di demosaicing utilizzato per convertire i dati grezzi del sensore con pattern Bayer in immagini a colori. Il metodo &quot;Standard (Veloce, Qualità media)&quot; offre un equilibrio ottimale tra velocità di elaborazione e qualità dell’immagine. Il metodo &quot;Texture Aware (Lento, Massima qualità)&quot; \[Chloros+] utilizza un algoritmo di demosaicing di alta qualità sensibile ai contorni, combinato con un modello di denoising basato su AI/ML che rimuove quasi tutto il rumore del demosaicing. Il modello Texture Aware richiede memoria GPU (VRAM) per funzionare. Si consiglia di utilizzarlo quando si dispone di &gt;4 GB di VRAM per un&#x27;elaborazione più veloce.
* **Solo quando la riga è un menu a tendina**: il menu a tendina a due opzioni appare solo quando**entrambe**le condizioni sono vere — si è effettuato l’accesso con un abbonamento Chloros+ idoneo,**e** il progetto non contiene acquisizioni LATTICE. In caso contrario, la riga viene visualizzata come testo semplice con la dicitura `Standard (Fast, Medium Quality)` senza alcuna opzione selezionabile.
* **Nota su LATTICE**: non esiste un modello &quot;Texture Aware&quot; addestrato per LATTICE e la pipeline impone il demosaic standard per i fotogrammi LATTICE indipendentemente dal valore memorizzato. Se si aggiunge una cartella LATTICE a un progetto in cui era già selezionato &quot;Texture Aware&quot;, Chloros riporta l&#x27;impostazione su &quot;Standard&quot; anziché lasciare un valore obsoleto in `project.json`.

### Intervallo minimo di ricalibrazione

* **Tipo**: Numero
* **Intervallo**: da 0 a 3.600 secondi
* **Impostazione predefinita**: 0 secondi
* **Descrizione**: Imposta l’intervallo di tempo minimo (in secondi) tra l’utilizzo dei target di calibrazione. Se impostato su 0, Chloros utilizzerà ogni target di calibrazione rilevato. Se impostato su un valore più alto, Chloros utilizzerà solo i target di calibrazione separati da almeno questo numero di secondi, riducendo il tempo di elaborazione per i set di dati con acquisizioni frequenti dei target di calibrazione.
* **Quando regolare**:
  * Impostare su 0 per la massima precisione di calibrazione quando le condizioni di illuminazione variano
  * Aumentare (ad es. a 60-300 secondi) per un’elaborazione più veloce quando l’illuminazione è costante e si dispone di immagini frequenti dei bersagli di calibrazione

### Offset del fuso orario del sensore di luce

* **Tipo**: Numero
* **Intervallo**: da -12 a +12 ore
* **Impostazione predefinita**: 0 ore
* **Descrizione**: Specifica l’offset del fuso orario (in ore rispetto all’UTC) per i timestamp dei dati del sensore di luce, utilizzato per allineare i log del sensore di luce agli orari di acquisizione delle immagini. Le registrazioni più recenti con codice `.daq` riportano il proprio fuso orario di provenienza, quindi questa impostazione è necessaria soprattutto per i log più vecchi registrati in ora locale.

### Applica correzioni PPK

* **Tipo**: Casella di controllo
* **Impostazione predefinita**: Disabilitato (deselezionata)
* **Descrizione**: Abilita l’uso delle correzioni PPK (Post-Processed Kinematic) provenienti dai registratori DAQ MAPIR dotati di GPS (GNSS). Se abilitata, Chloros utilizzerà tutti i file di log .daq contenenti dati relativi ai pin di esposizione presenti nella directory del progetto e applicherà correzioni precise di geolocalizzazione alle immagini.
* **Requisiti**: nella directory del progetto deve essere presente un file di log .daq con voci relative al pin di esposizione
* **Quando abilitare**: si raccomanda di abilitare sempre la correzione PPK se nel file di log .daq sono presenti voci relative al feedback di esposizione.

### Pin di esposizione 1

* **Tipo**: Selezione da menu a tendina
* **Visibilità**: Visibile solo quando l’opzione “Applica correzioni PPK” è abilitata E sono disponibili dati di esposizione per il Pin 1
* **Opzioni**:
  * Nomi dei modelli di fotocamera rilevati nel progetto
  * “Non utilizzare” - Ignora questo pin di esposizione
* **Impostazione predefinita**: Selezionato automaticamente in base alla configurazione del progetto
* **Descrizione**: Assegna una fotocamera specifica al Pin di esposizione 1 per la sincronizzazione temporale PPK. Il pin di esposizione registra il momento esatto in cui viene azionato l’otturatore della fotocamera, il che è fondamentale per una geolocalizzazione PPK accurata.
* **Comportamento della selezione automatica**:
  * Singola fotocamera + singolo pin: seleziona automaticamente la fotocamera
  * Singola fotocamera + due pin: il pin 1 viene assegnato automaticamente alla fotocamera
  * Fotocamere multiple: è richiesta la selezione manuale

### Pin di esposizione 2

* **Tipo**: Selezione da menu a tendina
* **Visibilità**: Visibile solo quando è abilitata l’opzione “Applica correzioni PPK” E sono disponibili dati di esposizione per il Pin 2
* **Opzioni**:
  * Nomi dei modelli di telecamera rilevati nel progetto
  * &quot;Non utilizzare&quot; - Ignora questo pin di esposizione
* **Impostazione predefinita**: Selezione automatica in base alla configurazione del progetto
* **Descrizione**: Assegna una telecamera specifica al pin di esposizione 2 per la sincronizzazione temporale PPK quando si utilizza una configurazione a doppia telecamera.
* **Comportamento della selezione automatica**:
  * Singola fotocamera + singolo pin: il Pin 2 viene automaticamente impostato su &quot;Non utilizzare&quot;
  * Singola fotocamera + due pin: il Pin 2 viene automaticamente impostato su &quot;Non utilizzare&quot;
  * Fotocamere multiple: è richiesta la selezione manuale
* **Nota**: Non è possibile assegnare la stessa telecamera contemporaneamente sia al pin 1 che al pin 2.***

## Sensore di luce DAQ

Questa sezione è presente nelle Impostazioni del progetto ed elenca tutti i file DAQ di radiazione discendente presenti nel progetto — registrazioni `.daq` e registri DAQ-M `.csv`. Le registrazioni effettuate nella scheda Sensori di luce vengono aggiunte automaticamente al progetto aperto.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Ogni riga mostra il file, il modello del sensore e la correzione del cappuccio diffusore effettivamente in vigore per quel file. Sopra le righe è presente un unico controllo a livello di progetto:

### Sovrascrittura del cappuccio (tutti i file)

* **Tipo**: Selezione a tendina
* **Opzioni**: `Auto` più i profili di correzione del cappuccio validi per i tipi di sensori presenti nel progetto
* **Impostazione predefinita**: Auto
* **Memorizzato come**: `Processing → "DAQ cap id"` (impostazione predefinita `auto`)
* **Descrizione**: `Auto` utilizza il cap registrato in ciascun file (se non è stato registrato nulla, viene assunto il cap &quot;Sunshine&quot; — tutti i DAQ MAPIR vengono forniti con il correttore &quot;Sunshine&quot;). La selezione di un cap specifico sovrascrive**tutti** i file downwelling del progetto: le registrazioni grezze vengono corrette con esso, mentre quelle che presentano già un cappuccio vengono ricalibrate (la correzione registrata viene annullata e viene applicata quella selezionata).
* **Importante**: il cappuccio selezionato deve corrispondere a quello fisicamente montato durante la registrazione. Né il sensore né il software sono in grado di rilevare il cappuccio fisico — un ID del cappuccio non corrispondente corregge erroneamente gli spettri.

È presente volutamente **un unico** controllo a livello di progetto anziché menu a tendina per singolo file: l’impostazione si applica a ogni sorgente downwelling del progetto.***

## Allineamento dell’array

Questa sezione appare **solo** quando almeno un&#x27;immagine nel progetto contiene la trasformazione di allineamento da modulo a modulo che gli array LATTICE applicano al momento dell&#x27;acquisizione (tag XMP `Chloros:Alignment*`). Mostra quante immagini contengono tag di allineamento, quale fotocamera è quella di riferimento (badge `REF`) e una tabella con il conteggio delle immagini per ciascuna fotocamera.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Applica allineamento array

* **Tipo**: Casella di selezione
* **Impostazione predefinita**: Abilitato (selezionato)
* **Memorizzato come**: `Processing → "Array alignment"`
* **Descrizione**: Deforma ogni prodotto elaborato (debayering / anteprima / radianza / riflettanza / indice) nella geometria di riferimento condivisa dell’array utilizzando la trasformazione registrata al momento dell’acquisizione. Disattivato = esportazione nella geometria nativa per sensore.

### Ritaglia in base alla sovrapposizione comune

* **Tipo**: Casella di controllo (attiva solo se *Applica allineamento array* è attivo)
* **Impostazione predefinita**: Abilitato (selezionato)
* **Memorizzato come**: `Processing → "Array alignment crop"`
* **Descrizione**: Ritaglia le esportazioni allineate alla regione condivisa da tutti i moduli della fotocamera, in modo che ogni banda abbia la stessa impronta. Disattivata mantiene l’intera area del sensore (riempimento nero al di fuori della sorgente).

### Ricampionamento

* **Tipo**: Menu a tendina (attivo solo quando *Applica allineamento array* è attivato)
* **Opzioni**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Impostazione predefinita**: Bilineare
* **Memorizzato come**: `Processing → "Array alignment interpolation"`
* **Descrizione**: Interpolazione utilizzata dalla deformazione di allineamento. *Nearest* (Più vicino) conserva i valori esatti della sorgente (senza miscelazione tra pixel) per un’analisi radiometrica rigorosa; *Bilineare* è l’opzione migliore per la mappatura e l’uso visivo.

Le stesse tre opzioni sono disponibili in formato headless come `chloros-cli process --array-alignment`, `--array-alignment-crop` e `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indice

Queste impostazioni consentono di configurare indici multispettrali per l’analisi e la visualizzazione.

### Aggiungi indice

* **Tipo**: Pannello di configurazione indici speciali
* **Descrizione**: Apre un pannello interattivo in cui è possibile selezionare e configurare indici multispettrali della vegetazione (NDVI, NDRE, EVI, ecc.) da calcolare durante l’elaborazione delle immagini. È possibile aggiungere più indici, ciascuno con le proprie impostazioni di visualizzazione.
* **Indici disponibili**: Il menu a tendina dell’interfaccia grafica include**27** formule di indici multispettrali predefinite (vedere [Formule degli indici multispettrali](multispectral-index-formulas.md) per l’elenco completo, compresi i nomi accettati anche dall’opzione CLI/SDK `--indices`).
* **Funzionalità**:
  * Selezione tra formule di indici predefinite
  * Trascinare i canali dei filtri della fotocamera negli slot delle bande della formula
  * Configurare i gradienti di colore per la visualizzazione (LUT - Look-Up Tables)
  * Impostare i valori di soglia e le modalità di clipping
  * Creare formule di indice personalizzate
* **Nota**: Gli indici non vengono calcolati per le fotocamere monocromatiche a banda singola LATTICE M3M — gli indici multibanda non sono definiti su una singola banda. Survey3 e LATTICE M3C non sono interessati da questa limitazione.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Ogni indice aggiunto viene visualizzato come formula matematica, con un cerchio colorato per ogni slot di banda: rosso = Red, verde = Green, blu = Blue, arancione = Orange, ciano = Cyan, viola = NIR, magenta = RE. Trascinare un cerchio dalla riga sopra la formula su uno slot per associarlo; fare doppio clic su uno slot associato per cancellarlo. L&#x27;indice viene calcolato solo una volta, quando ogni slot utilizzato dalla formula dispone di un canale.

### Formule personalizzate (Funzionalità Chloros+)

* **Tipo**: Matrice di definizioni di formule personalizzate
* **Disponibilità**: Richiede l’accesso con un abbonamento Chloros+ idoneo.
* **Descrizione**: Consente di creare e salvare formule personalizzate per indici multispettrali utilizzando operazioni matematiche sulle bande. Le formule personalizzate vengono salvate con le impostazioni del progetto e possono essere utilizzate proprio come gli indici predefiniti.
* **Come creare**:
  1. Nel pannello di configurazione dell’indice, aprire il calcolatore di formule personalizzate
  2. Scrivi la formula utilizzando i **simboli degli slot di banda**, non i nomi delle bande
  3. Salva la formula con un nome descrittivo: apparirà quindi nella parte inferiore del menu a tendina delle formule e potrai trascinare i cerchi dei canali della tua fotocamera sugli slot esattamente come per un preset integrato
* **Sintassi della formula**:
  * Slot delle bande: `x`, `y`, `z`, `a`, `b`, `c` — sei posizioni da associare ai canali reali tramite trascinamento
  * Operatori: `+`, `-`, `*`, `/`, `^` e `()` per il raggruppamento
  * Funzioni: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Perché simboli e non nomi di bande**: una formula scritta come `(y-x)/(y+x)` funziona su qualsiasi fotocamera, poiché la mappaturadecide se `y` corrisponde all&#x27;850 nm NIR di un filtro RGN o all&#x27;808 nm NIR di un filtro OCN. Anche i preset integrati sono memorizzati allo stesso modo — consultare [Formule degli indici multispettrali](multispectral-index-formulas.md) per la forma simbolica esatta di tutti e 27.
* **Dove funzionano**: le formule personalizzate vengono salvate con le impostazioni del progetto e possono essere utilizzate sia nell’[Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) che durante l’elaborazione.**Non** sono accettate dall’CLI/SDK `--indices`, che si limita a espandere i 22 nomi predefiniti integrati.***

## Esportazione

Queste impostazioni controllano il formato e la qualità delle immagini elaborate esportate.

### Formato immagine calibrata

* **Tipo**: Selezione da menu a tendina
* **Opzioni**:
  * **TIFF (16 bit)** - Formato TIFF a 16 bit non compresso
  * **TIFF (32 bit, percentuale)** - Formato TIFF a 32 bit in virgola mobile con valori di riflettanza espressi in percentuale
  * **PNG (8 bit)** - Formato PNG compresso a 8 bit
  * **JPG (8 bit)** - Formato JPEG compresso a 8 bit
* **Predefinito**: TIFF (16 bit)
* **Descrizione**: Seleziona il formato di file per il salvataggio delle immagini elaborate e calibrate. I file esportati vengono salvati in una sottocartella specifica per ciascun formato all&#x27;interno della cartella di ciascuna fotocamera (`tiff16`, `tiff32`, `png8`, `jpg8`), con una cartella `<Product>_Images/` per ogni prodotto. I file esportati mantengono il nome del file di origine: è la cartella, non il suffisso del nome del file, a identificare il prodotto.
* **Raccomandazioni sul formato**:
  * **TIFF (16 bit)**: Consigliato per analisi scientifiche e flussi di lavoro professionali. Preserva la massima qualità dei dati senza artefatti di compressione. Ideale per l’analisi multispettrale e l’ulteriore elaborazione con software GIS.
  * **TIFF (32 bit, percentuale)**: Ideale per flussi di lavoro che richiedono valori di riflettanza espressi in percentuale (0-100%). Offre la massima precisione per le misurazioni radiometriche.
  * **PNG (8 bit)**: Adatto alla visualizzazione sul web e alla visualizzazione generale. Dimensioni dei file più ridotte con compressione senza perdita di dati, ma con gamma dinamica ridotta.
  * **JPG (8 bit)**: Dimensioni dei file minime, ideale solo per anteprime e visualizzazione sul web. Utilizza una compressione con perdita di dati, non adatta all’analisi scientifica.
* **Nota**: la radianza LATTICE viene sempre esportata come TIFF a 32 bit in virgola mobile, indipendentemente da questa impostazione.***

## Salva modello di progetto

Questa funzione consente di salvare le impostazioni correnti del progetto come modello riutilizzabile.

* **Tipo**: Campo di testo + pulsante Salva
* **Descrizione**: Inserisci un nome descrittivo per il tuo modello di impostazioni e fai clic sull’icona di salvataggio. Il modello memorizzerà tutte le impostazioni correnti del progetto (rilevamento dei target, opzioni di elaborazione, indici e formato di esportazione) per un facile riutilizzo in progetti futuri. I modelli vengono salvati nella cartella `Project Templates/` all’interno della cartella di salvataggio del progetto e possono anche essere selezionati o esportati dal menu principale (*Seleziona modello* / *Salva modello* / *Esporta modello*).
* **Casi d’uso**:
  * Creare modelli per diversi sistemi di telecamere (RGB, multispettrale, NIR)
  * Salvare configurazioni standard per specifici tipi di colture o flussi di lavoro di analisi
  * Condividere impostazioni uniformi all’interno di un team
* **Come si usa**:
  1. Configurare tutte le impostazioni desiderate per il progetto
  2. Inserire un nome per il modello (ad es., &quot;RedEdge Survey3 NDVI Standard&quot;)
  3. Fare clic sull’icona di salvataggio
  4. Il modello può ora essere caricato durante la creazione di nuovi progetti

***

## Salva cartella progetto

Questa impostazione specifica dove vengono salvati per impostazione predefinita i nuovi progetti.

* **Tipo**: Visualizzazione del percorso della directory + pulsante Modifica
* **Predefinito (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Predefinito (Linux)**: `~/Chloros Projects`
* **Descrizione**: Mostra la directory predefinita corrente in cui vengono creati i nuovi progetti Chloros. Fare clic sull’icona di modifica per selezionare una directory diversa. La modifica viene memorizzata come una singola riga di testo nel file `~/.chloros/working_directory.txt` — all’interno di Windows, ovvero `C:\Users\<Username>\.chloros\working_directory.txt`. Se tale file manca o indica un percorso non più esistente, Chloros ricorre all’impostazione predefinita sopra indicata. Il file CLI legge e scrive lo stesso file, quindi `chloros-cli` e l’interfaccia grafica sono sempre allineati riguardo alla posizione dei progetti.
* **I modelli di progetto** si trovano in una sottocartella denominata `Project Templates/` all’interno di questa directory.
* **Quando modificare**:
  * Impostare un’unità di rete per la collaborazione in team
  * Passare a un’unità con maggiore spazio di archiviazione per set di dati di grandi dimensioni
  * Organizzare i progetti per anno, cliente o tipo di progetto in cartelle diverse
* **Nota**: la modifica di questa impostazione influisce solo sui NUOVI progetti. I progetti esistenti rimangono nelle loro posizioni originali.***

## Persistenza delle impostazioni

Un progetto Chloros è una **cartella**. Tutte le impostazioni del progetto vengono salvate all&#x27;interno di `project.json`; l&#x27;hardware collegato viene memorizzato insieme ad esse in `cameras.json` e `sensors.json`, quindi riaprendo un progetto si ricollegano automaticamente anche le telecamere e i sensori di luce. Quando si riapre un progetto, tutte le impostazioni vengono ripristinate esattamente come le avevi lasciate. I progetti salvati possono anche essere gestiti in modalità headless tramite `chloros-cli project` o `open_project` di SDK.

### Gerarchia delle impostazioni

Le impostazioni vengono applicate nel seguente ordine:

1. **Impostazioni predefinite di sistema** - Impostazioni predefinite integrate definite da Chloros
2. **Impostazioni del modello** - Se si carica un modello durante la creazione di un progetto
3. **Impostazioni del progetto salvate** - Impostazioni salvate con il file di progetto
4. **Modifiche manuali** - Qualsiasi modifica apportata durante la sessione corrente

### Impostazioni ed elaborazione delle immagini

Le impostazioni di elaborazione vengono lette all’avvio di un ciclo di elaborazione. La modifica di un’impostazione non altera retroattivamente i prodotti già presenti su disco: eseguire nuovamente l’elaborazione per applicare le nuove impostazioni. Alcune impostazioni non influiscono in alcun modo sull’elaborazione:

* Risoluzione delle miniature delle immagini (solo visualizzazione)
* Salva modello di progetto
* Salva cartella di progetto

***

## Riferimento alle chiavi di configurazione

Per l’automazione (CLI `--config`, SDK `configure`, o per la lettura diretta di `project.json`), queste sono le chiavi esatte presenti in `Project Settings`:

| Percorso chiave | Tipo | Predefinito |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | numero 0-10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | numero compreso tra 0 e 100 | `60` |
| `Processing → Vignette correction` | booleano | `true` |
| `Processing → Reflectance calibration / white balance` | booleano | `true` |
| `Processing → Export sensor response` | booleano | `true` |
| `Processing → Export vignette corrected` | booleano | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | numero 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | numero -12..12 | `0` |
| `Processing → Apply PPK corrections` | booleano | `false` |
| `Processing → DAQ cap id` | ID profilo limite oppure `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | elenco delle configurazioni degli indici | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Le chiavi `Array alignment` vengono scritte la prima volta che viene visualizzata la sezione &quot;Array Alignment&quot; oppure quando vengono impostate da una chiamata di automazione. In loro assenza, la pipeline utilizza gli stessi valori mostrati sopra (`true`, `true`, bilineare), pertanto un progetto.json privo di tali chiavi si comporta in modo identico a uno che le contiene.

### Chiavi memorizzate in `project.json` senza controllo nel pannello delle impostazioni

Queste si trovano nella stessa struttura ad albero di `Project Settings` e vengono lette dall’elaborazione, ma non troverete un widget dedicato nella barra laterale:

| Percorso chiave | Tipo | Predefinito | Impostato da |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Sovrascrive il modo in cui vengono interpretati i file TIFF di input di LATTICE; `auto` deduce le informazioni dal tag XMP di ciascun file `Chloros:ProcessingLevel` e dal numero di canali. Ignper le acquisizioni Survey3 `.raw`. Non è volutamente un&#x27;impostazione dell&#x27;interfaccia grafica: l’impostazione automatica è corretta in tutti i casi normali. |
| `Processing → Target reflectance dir` | stringa del percorso | `""` | `chloros-cli process --target-reflectance-dir`, oppure la destinazione del progetto API |
| `Processing → Target reflectance config` | dizionario con chiave del numero di serie della telecamera | `{}` | Registrazione di un obiettivo all&#x27;interno dell&#x27;inquadratura (modalità `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | stringa del percorso | `""` | SDK `process_folder(daq_log_path=…)`. Indica una registrazione `.daq` o una cartella che li contiene |
| `Target Detection → Minimum calibration target squares` | numero | `4` | Impostazione predefinita legacy; nessun controllo e nessun flag CLI |
| `UI → Grid thumbnail size` | numero | `160` | Il cursore di zoom delle miniature della griglia delle immagini |

Due preferenze del visualizzatore sono memorizzate **al livello superiore in `project.json`**, completamente al di fuori di `Project Settings`, poiché riguardano lo stato di visualizzazione piuttosto che le impostazioni di elaborazione:

| Percorso chiave | Tipo | Predefinito | Impostato da |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | numero intero 1–256 | `1` | Controllo GSD (px) della scheda immagine — vedi [Aprire un&#x27;immagine a schermo intero](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Best practice

1. **Iniziare con le impostazioni predefinite**: le impostazioni predefinite funzionano bene per la maggior parte dei sistemi di telecamere MAPIR e per i flussi di lavoro tipici.
2. **Creare modelli**: una volta ottimizzate le impostazioni per un flusso di lavoro o una telecamera specifici, salvarle come modello per garantire la coerenza tra i vari progetti.
3. **Effettuare dei test prima dell’elaborazione completa**: quando si sperimentano nuove impostazioni, testarle su un piccolo sottoinsieme di immagini prima di elaborare l’intero set di dati.
4. **Documentate le vostre impostazioni**: utilizzate nomi descrittivi per i modelli che indichino il sistema di telecamere, il tipo di elaborazione e l’uso previsto (ad es., “Survey3\_RGB\_NDVI\_Agricoltura”).
5. **Selezione del formato di esportazione**: Scegli il formato di esportazione in base all’uso finale:
   * Analisi scientifica → TIFF (16 bit o 32 bit)
   * Elaborazione GIS → TIFF (16 bit)
   * Visualizzazione rapida → PNG (8 bit)
   * Condivisione sul web → JPG (8 bit)

***

Per ulteriori informazioni sugli indici multispettrali in Chloros, consultare la pagina [Formule degli indici multispettrali](multispectral-index-formulas.md).
