# Impostazioni e modalità di acquisizione

L’acquisizione nella scheda “Telecamere” è gestita da un pulsante rosso **Acquisisci tutto**e da un pannello**Impostazioni di acquisizione** che determina il risultato di tale pulsante: quali telecamere partecipano, quali tipi di esportazione salva ciascuna telecamera e se l’otturatore scatta una volta, in modo continuo o a intervalli regolari. Questa pagina illustra l’intero flusso di lavoro: la configurazione, l’acquisizione vera e propria, la posizione in cui i file vengono salvati sul disco e come rielaborarli in seguito per ottenere prodotti calibrati. I controlli relativi alle telecamere e agli array si trovano nella pagina [Impostazioni telecamera](camera-settings.md).

{% hint style="info" %}
**Le acquisizioni richiedono un progetto aperto.** L’opzione “Acquisisci tutto” e l’icona a forma di ingranaggio delle “Impostazioni di acquisizione” rimangono disabilitate finché non viene aperto un progetto (“Crea o apri un progetto per salvare le acquisizioni”). Ogni acquisizione viene salvata nella cartella del progetto in `captures/`.
{% endhint %}

## Il pannello &quot;Impostazioni di acquisizione&quot;

Aprilo tramite l’**icona a forma di ingranaggio accanto a &quot;Acquisisci tutto&quot;**nell’elenco delle telecamere della barra laterale, oppure tramite il pulsante**&quot;Apri impostazioni di acquisizione…&quot;** nella parte inferiore di qualsiasi pannello delle impostazioni specifiche per telecamera. L’intestazione recita “Impostazioni di acquisizione” con un pulsante ← Indietro.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Le selezioni effettuate qui — telecamere incluse, caselle di controllo per tipo e modalità di acquisizione — vengono salvate **per progetto** e ripristinate quando lo si riapre.

### Modalità di acquisizione

Tre pulsanti di modalità nella parte superiore del pannello:

| Modalità | Funzione | Sottoselzioni (impostazioni predefinite) |
| --- | --- | --- |
| **Singola** *(impostazione predefinita)* | Una singola acquisizione su tutte le telecamere selezionate. | — |
| **Continuo**| Acquisizioni consecutive fino a una condizione di arresto. | Arresto in base al**Numero di acquisizioni** (impostazione predefinita 1) *oppure* alla **Durata dell’acquisizione** (impostazione predefinita 10 s; unità: secondi / minuti / ore / giorni). |
| **Intervallo**(timelapse) | Scatti a intervalli regolari. |**Scatti / intervallo**(impostazione predefinita 1) ·**Ogni**N unità (impostazione predefinita 5 s) ·**Per** N unità (impostazione predefinita 1 m). |

In modalità Continua o a intervalli, il pulsante «Acquisisci tutto» diventa un pulsante **«Stop (N)»** durante l’esecuzione, contando le acquisizioni man mano che vengono effettuate.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Selezione delle telecamere e dei tipi di esportazione

Il testo di aiuto del pannello riassume il concetto: scegli quali fotocamere e tipi di esportazione vengono prodotti da “Acquisisci tutto” — per impostazione predefinita tutto è attivato e le scelte vengono salvate con questo progetto.

* I pulsanti **Seleziona tutto / Deseleziona tutto** attivano o disattivano contemporaneamente le caselle di controllo di inclusione di ogni fotocamera.
* **Interruttori per i tipi di esportazione in blocco**(due file di pulsanti):**Tutto in formato Raw / Tutto debayered / Tutto in anteprima / Tutto in radianza / Tutto in riflettanza / Tutto in indice**. Ciascuna opzione presenta una colorazione a tre stati: verde ✓ = attiva per tutte le fotocamere che la supportano, ambra – = attiva per alcune, grigio = nessuna. Un&#x27;opzione è disabilitata quando nessuna fotocamera collegata supporta quel tipo. Tutte le opzioni appaiono in grigio quando è attiva l’opzione “Acquisizione più veloce”.
* **Righe per singola fotocamera**: una casella di controllo &quot;Includi&quot;, più un elenco espandibile (▸/▾) dei tipi di esportazione applicabili a quella fotocamera con singole caselle di controllo. La riga mostra un conteggio come &quot;4/6&quot;.

### Tipi di esportazione e quali fotocamere li supportano

Esistono sei tipi di esportazione: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Nella riga di ciascuna fotocamera compaiono solo quelli applicabili:

| Tipo di esportazione | Contenuto | RGB (FRGB) | Multispettrale Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mosaico Bayer (mono: singola banda) direttamente dal sensore | ✓ | ✓ | ✓ |
| **Debayered** | Demosaicatura lineare (mono: scala di grigi a 1 canale) | ✓ | ✓ | ✓ |
| **Anteprima** | Catena di elaborazione completa (bilanciamento del bianco + gamma secondo il profilo della fotocamera; multispettrale: espansione in falsi colori) | ✓ | ✓ | ✓ |
| **Radianza** | float32 W/m²/sr/nm tramite la catena radiometrica completa | — (non disponibile) | ✓ | ✓ |
| **Riflettanza** | uint16 ρ (32768 = 1,0) | — (non disponibile) | ✓ — visualizzata solo quando la fotocamera dispone di un sensore di luce DAQ (proprio o ereditato dall’array) | come per il multispettrale |
| **Indice** | Rendering dell’indice di vegetazione (LUT) | — | ✓ — richiede un’espressione di indice abilitata e non vuota sulla fotocamera, e non è disponibile per i membri di un array combinato (l’array possiede un unico indice condiviso) | — (un indice richiede ≥2 bande; vedi [Telecamere mono e indici di vegetazione](mono-indices.md)) |

La radianza e la riflettanza non sono mai disponibili per le fotocamere RGB — la radianza per pixel Bayer non è significativa per un sensore fotometrico a banda larga.

### Acquisizione più veloce

Il pulsante **⚡ Acquisizione più veloce — solo raw**(arancione quando attivo) sovrascrive tutte le selezioni di esportazione impostandole su**solo raw** — oltre a un composito con indice combinato gratuito per gli array — in modo che il fotogramma venga salvato il più rapidamente possibile: i calcoli relativi a radianza/riflettanza/visualizzazione vengono completamente saltati al momento dell’acquisizione.

{% hint style="info" %}
**Viene comunque salvato un `.daq`.** Quando è assegnato un sensore di luce, l’opzione “Acquisizione più veloce” registra comunque il valore DAQ della luce in discesa accanto ai fotogrammi raw — in modo che i prodotti di radianza, riflettanza e indice possano essere tutti generati in un secondo momento tramite rielaborazione (vedi [Rielaborazione delle acquisizioni](#re-processing-captures-into-calibrated-products)). Inoltre, Fastest Capture non altera le selezioni effettuate tramite le caselle di controllo: disattivando la funzione, queste vengono ripristinate.
{% endhint %}

### Controlli per singolo array

Ogni array collegato dispone di una propria scheda di gruppo nel pannello:

* **Casella di controllo &quot;Includi&quot;** (a tre stati per i membri) e il nome dell’array con la relativa modalità di visualizzazione: &quot;(combinata | separata)&quot;.
* Casella di controllo **Allineato**(impostazione predefinita**attiva**): adatta le esportazioni dei membri al profilo di allineamento dell’array in modo che le esportazioni siano registrate pixel per pixel tra le telecamere. I dati grezzi rimangono non distorti ma riportano la trasformazione nei propri metadati. (Il profilo stesso viene calcolato nel [pannello delle impostazioni dell’array](camera-settings.md#alignment-co-registration-combined-only).)
* Le righe delle telecamere dei membri sono annidate all’interno della scheda.

La scheda dell’array ospita anche due registratori. Considerateli come **monitoraggio vs analisi**:

| Registratore | Livello | Cosa registra |
| --- | --- | --- |
| **● Registra video dell’indice / ■ Interrompe la registrazione** *(solo array combinati)* | **Monitoraggio** | Il composito dell’indice combinato in tempo reale in formato video a 10 fps — 8 bit, risoluzione di anteprima, LUT integrata. Richiede un progetto aperto e una visualizzazione live in streaming. Mostra i fotogrammi e il tempo trascorso durante la registrazione. |
| **⦿ Registra raffica raw / ■ Interrompi sequenza raw** *(qualsiasi matrice)* | **Analisi**| Fotogrammi Bayer raw alla frequenza di acquisizione live (senza elaborazione) più un manifesto per fotogramma e letture `.daq`, in formato `captures/bursts/`. Dopo una raffica, compare il pulsante**Crea video**: rielabora la raffica offline in un video calibrato — indice combinato e/o radianza / riflettanza / indice per singola telecamera — più TIFF opzionali. La creazione dell&#x27;indice combinato parte automaticamente quando si interrompe la raffica.

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

|## Il flusso

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

“Capture All” Premere **Capture All** nell’elenco delle telecamere nella barra laterale:

1. Ogni telecamera inclusa, visibile e non in pausa effettua l’acquisizione con i tipi di esportazione selezionati. **Gli array scattano come un unico trigger sincronizzato** (un unico gruppo sincronizzato tra tutti i membri — vedi [Array multicamera](arrays.md)); le telecamere autonome registrano individualmente.
2. Le telecamere nascoste (con l&#x27;icona a forma di occhio) o in pausa vengono ignorate. Un array viene completamente bloccato solo quando *tutti* i suoi membri sono nascosti o in pausa.
3. Ogni volta che viene assegnato un sensore di luce, la lettura DAQ corrispondente della radiazione discendente viene salvata come file `.daq` insieme alle immagini — anche per le acquisizioni solo in formato raw — in modo che i prodotti radiometrici possano sempre essere ricavati in un secondo momento.
4. Il pulsante mostra lo stato di avanzamento in tempo reale — «Acquisizione in corso… completata/totale» — e in modalità Continua/A intervalli diventa **Stop (N)**. Ogni elemento di acquisizione ha un timeout di 300 s.
5. Al termine del passaggio, un messaggio lampeggiante riporta **&quot;N file salvati&quot;**o**&quot;N salvati, F falliti&quot;**, oltre a &quot;(S nascosti/in pausa/saltati)&quot; quando alcune telecamere sono state saltate.

## Dove vengono salvate le acquisizioni

Le acquisizioni vengono salvate all’interno del progetto aperto in `<project>/captures/`. Ogni tipo di esportazione viene salvato nella propria **sottocartella**, quindi un’acquisizione a più livelli non mescola mai i tipi:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` è il timestamp dell’acquisizione e `<serial>` il numero di serie della telecamera. Le acquisizioni autonome sono denominate `capture_<ts>_SN<serial>_<level>`; le acquisizioni in array da un singolo trigger sincronizzato sono denominate `sync_<ts>_SN<serial>_<level>` e **condividono un unico timestamp tra tutte le telecamere del gruppo** (il suffisso del livello viene omesso quando una telecamera salva un solo livello).
* **Un&#x27;asimmetria da tenere presente:** il livello di visualizzazione viene memorizzato in una cartella denominata `preview/`, mentre i file mantengono `_display` nel nome — la cartella e il suffisso differiscono solo per quel livello.
* I livelli sconosciuti vengono inseriti in una cartella con il loro stesso nome; se non è possibile creare una sottocartella, il file viene salvato nella directory principale delle acquisizioni anziché andare perso.
* I file TIFF delle acquisizioni sono compressi senza perdita di dati (DEFLATE) per impostazione predefinita e contengono tutti i metadati di calibrazione ed elaborazione **all’interno dell’XMP del file** — le acquisizioni sono autodescrittive, senza file sidecar oltre al file di lettura `.daq`.

Si tratta dello stesso layout che `chloros-cli lattice capture` / `array-capture` scrivono nella propria directory `-o` — documentato nella [Riferimento CLI § Come si presenta una cartella delle acquisizioni](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Rielaborazione delle acquisizioni in prodotti calibrati

I fotogrammi grezzi acquisiti, insieme al file `.daq` salvato, sono tutto ciò di cui la pipeline di elaborazione ha bisogno: ecco perché la modalità “Fastest Capture” è sicura per il lavoro vero e proprio.

* **Interfaccia grafica**: aggiungi la cartella delle acquisizioni a un progetto ([Aggiunta di file a un progetto](../processing-images-gui/adding-files-to-a-project.md)) ed esegui l’elaborazione come di consueto.
* **CLI**: indirizzare `process` alla**radice delle acquisizioni**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` normalmente importa solo la cartella specificata, ma quando tale cartella non contiene immagini e presenta sottocartelle, scende automaticamente al loro interno — in questo modo le sottocartelle dei livelli e i file della radice `.daq` vengono acquisiti in un unico passaggio. Ogni acquisizione viene importata come **singola immagine** con gli altri livelli allegati come modalità visualizzabili, non come un&#x27;immagine per ogni livello.

È possibile anche specificare direttamente il nome di una sottocartella di livello (ad es. `…/captures/raw/`), ma in questo caso i file radice `.daq` vengono tralasciati: copiateli insieme quando ricavate nuovamente un prodotto radiometrico da `raw/`, altrimenti la corrispondenza del timestamp non avrà alcun riferimento a cui fare riferimento.

{% hint style="warning" %}
**L’elaborazione inizia sempre da `raw`.**All’interno di ciascuna acquisizione, il fotogramma grezzo costituisce la fonte della pipeline; `debayered`, `radiance`, `reflectance` e `preview` sono disponibili come modalità di visualizzazione ma non vengono mai reimmessi nella pipeline — la rielaborazione di un prodotto derivato riapplicherebbe la vignettatura, il colore e i calcoli di radianza già incorporati nei suoi pixel, pertanto Chloros viene scartato anziché essere sottoposto a doppia elaborazione. I rendering `index/` e `composite/` non vengono mai elaborati (sono output, non acquisizioni). Una cartella «captures» salvata**senza** importazioni raw viene visualizzata normalmente, ma `process` la salta e lo segnala; `--input-level {raw,debayered,processed}` è la via di fuga intenzionale che forza un punto di ingresso. Consultare il [Riferimento CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like) per i messaggi esatti relativi all’esclusione.
{% endhint %}

Altri due comportamenti da tenere presenti quando si scrivono script di rielaborazione:

* Un&#x27;esecuzione `chloros-cli process` che ha richiesto prodotti ma non ha scritto **alcun prodotto immagine fallisce in modo evidente e termina con un codice di uscita diverso da zero** — non si otterrà mai un&#x27;esecuzione vuota silenziosa. Le esecuzioni riuscite riportano il numero dei prodotti. (Un&#x27;esecuzione deliberata solo con metadati viene comunque considerata un successo.)
* Le esportazioni elaborate e reimportate non occupano mai lo slot dei dati grezzi di una cattura — i dati grezzi originali rimangono sempre la fonte della pipeline.

## Equivalenti di CLI

Tutto ciò che è descritto in questa pagina può essere eseguito in modalità headless. Le modalità di acquisizione della GUI corrispondono direttamente a `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Singola | `chloros-cli lattice array-capture` |
| Continua | `array-capture --continuous [--count N] [--duration S]` |
| A intervalli | `array-capture --interval S [--duration S]` |
| Acquisizione più veloce | `array-capture --fastest` |
| Casella di controllo allineata | `--aligned / --no-aligned` |
| Caselle di controllo relative al tipo di esportazione | `--processing LEVEL` o `--levels L1,L2,…` (impostazione predefinita `all`) |
| Registra video indice | `chloros-cli lattice array-record` |
| Registra raffica raw / Crea video | `chloros-cli lattice array-burst` / `array-build-video` |

Le tabelle dei flag complete, l’opzione di acquisizione stabilizzata con Smart-AE (`--smart`) e il modello a velocità costante sono descritti nella sezione [CLI Riferimento § Modalità di acquisizione, registratori e rielaborazione offline](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
