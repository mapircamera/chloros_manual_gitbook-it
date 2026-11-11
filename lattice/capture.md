# Impostazioni e modalità di acquisizione

L&#x27;acquisizione nella scheda “Telecamere” è gestita da un pulsante rosso **Acquisisci tutto**e da un pannello**Impostazioni di acquisizione** che determina il risultato di tale pulsante: quali telecamere partecipano, quali tipi di esportazione salva ciascuna telecamera e se l’otturatore scatta una volta, in modo continuo o a intervalli regolari. Questa pagina illustra l’intero flusso: la configurazione, l’acquisizione stessa, la posizione in cui i file vengono salvati sul disco e come rielaborarli in seguito per ottenere prodotti calibrati. I controlli relativi alle telecamere e agli array si trovano nella pagina [Impostazioni telecamera](camera-settings.md).

{% hint style="info" %}
**Per effettuare un&#x27;acquisizione è necessario che sia aperto un progetto.** Le opzioni “Acquisisci tutto” e l’icona a forma di ingranaggio delle “Impostazioni di acquisizione” sono disabilitate finché non viene aperto un progetto (“Crea o apri un progetto per salvare le acquisizioni”). Ogni acquisizione viene salvata nella cartella del progetto in `captures/`.
{% endhint %}

## Il pannello &quot;Impostazioni di acquisizione&quot;

Aprilo con l’**icona a forma di ingranaggio accanto a &quot;Acquisisci tutto&quot;**nell’elenco delle telecamere nella barra laterale, oppure con il pulsante**&quot;Apri impostazioni di acquisizione…&quot;** nella parte inferiore di qualsiasi pannello delle impostazioni specifiche per telecamera. L’intestazione recita &quot;Impostazioni di acquisizione&quot; con un pulsante ← per tornare indietro.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Le selezioni effettuate qui — telecamere incluse, caselle di controllo per tipo e modalità di acquisizione — vengono salvate **per progetto** e ripristinate quando lo si riapre.

### Modalità di acquisizione

Tre pulsanti di modalità nella parte superiore del pannello:

| Modalità | Funzione | Sottosettazioni (impostazioni predefinite) |
| --- | --- | --- |
| **Singola** *(impostazione predefinita)* | Una singola acquisizione su tutte le telecamere selezionate. | — |
| **Continuativa**| Acquisizioni consecutive fino a una condizione di arresto. | Interruzione in base al**numero di acquisizioni** (impostazione predefinita 1) *oppure* alla **durata dell’acquisizione** (impostazione predefinita 10 s; unità: secondi / minuti / ore / giorni). |
| **Intervallo**(timelapse) | Serie di acquisizioni a intervalli regolari. |**Acquisizioni / intervallo**(impostazione predefinita 1) ·**Ogni**N unità (impostazione predefinita 5 s) ·**Per** N unità (impostazione predefinita 1 m). |

In modalità Continua o Intervallo, il pulsante “Acquisisci tutto” diventa un pulsante **Interrompi (N)** durante l’esecuzione, contando le acquisizioni man mano che vengono effettuate. Un&#x27;unità piena interrompe l&#x27;esecuzione (vedi [Il flusso &quot;Cattura tutto&quot;](#the-capture-all-flow)).**La modalità Continua su array viene eseguita all&#x27;interno del backend (novità della versione 1.3.0).** Quando tutte le telecamere nell’elenco appartengono a un array (telecamere autonome nascoste o escluse), la modalità “Continuo” non invia più una richiesta per ogni acquisizione: il backend di Chloros

esegue autonomamente il ciclo di acquisizione, scrivendo un gruppo sincronizzato mentre seleziona quello successivo, e il pulsante interroga il relativo conteggio. Se nell’elenco sono presenti telecamere autonome, Chloros

esegue le acquisizioni passo dopo passo come in precedenza.

{% hint style="warning" %}
**Acquisizione veloce su Windows

: mantenere il backend Chloros

escluso da Microsoft Defender.** La scansione in tempo reale di ogni file scritto da un’acquisizione ad alta velocità costa circa il 30% della velocità di acquisizione nei test di MAPIR

(centinaia di file al secondo con un’area di interesse [ROI] ridotta). Il programma di installazione per Windows

offre l’opzione **&quot;Escludi il backend di Chloros

dalla scansione di Microsoft Defender (acquisizione da telecamera molto più veloce; consigliato)&quot;**, selezionata per impostazione predefinita, e rimuove l’esclusione al momento della disinstallazione. Se l’hai deselezionata, riesegui il programma di installazione con l’opzione selezionata oppure aggiungi un’esclusione del processo Defender per `chloros-backend.exe` (nella cartella `backend` della cartella di installazione di Chloros

). Quando Defender utilizza uno o più core della CPU durante l’acquisizione continua di un array, il backend di Chloros

lo registra una volta nel proprio log, indicando queste stesse due soluzioni.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Selezione delle telecamere e dei tipi di esportazione

Il testo di aiuto del riquadro riassume il concetto: scegli quali telecamere e quali tipi di esportazione vengono generati da “Acquisizione totale” — per impostazione predefinita tutto è attivato e le scelte vengono salvate con questo progetto.

* I pulsanti **Seleziona tutto / Deseleziona tutto** attivano o disattivano contemporaneamente le caselle di controllo di inclusione di ogni telecamera.
* **Opzioni di tipo di esportazione in blocco**(due file di pulsanti):**Tutto Raw / Tutto Debayered / Tutto Anteprima / Tutto Radianza / Tutto Riflettanza / Tutto Indice**. Ciascuno presenta una colorazione a tre stati: verde ✓ = attivo per ogni telecamera che lo supporta, ambra – = attivo per alcune, grigio = nessuno. Un pulsante è disabilitato quando nessuna telecamera collegata supporta quel tipo. Tutti i pulsanti appaiono in grigio quando è attiva l’opzione “Acquisizione più veloce”.
* **Righe per singola telecamera**: una casella di controllo “Includi”, più un elenco espandibile (▸/▾) dei tipi di esportazione applicabili a quella telecamera con caselle di controllo individuali. La riga mostra un conteggio di quelli attivi, ad esempio &quot;4/6&quot;.

### Tipi di esportazione e quali fotocamere li supportano

Esistono sei tipi di esportazione: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Nella riga di ciascuna fotocamera compaiono solo quelli applicabili:

| Tipo di esportazione | Contenuto | RGB

(FRGB) | Multispettrale Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mosaico Bayer (mono: la singola banda) direttamente dal sensore | ✓ | ✓ | ✓ |
| **Debayered** | Demosaicatura lineare: offset scuro (DSNU) del pacchetto di calibrazione rimosso dal raw, seguito da una demosaicatura sensibile ai bordi — senza correzione di linearità, flat field o unmix. Salvato come RGB

a 3 canali in formato TIFF

(mono: scala di grigi a 1 canale) | ✓ | ✓ | ✓ |
| **Anteprima** | Catena di visualizzazione completa (bilanciamento del bianco + gamma secondo il profilo della fotocamera, più l’impostazione “Highlights” della fotocamera RGB

; multispettrale: espansione in falsi colori) | ✓ | ✓ | ✓ |
| **Radianza** | float32 W/m²/sr/nm tramite la catena radiometrica completa | — (non disponibile) | ✓ | ✓ |
| **Riflettanza** | uint16 ρ (32768 = 1,0) | — (non disponibile) | ✓ — visualizzato solo quando la telecamera dispone di un sensore di luce DAQ (proprio o ereditato dall’array) | come per multispettrale |
| **Indice** | Rendering dell’indice di vegetazione (LUT) | — | ✓ — richiede un’espressione dell’indice abilitata e non vuota sulla fotocamera, e non è disponibile per i membri di un array combinato (l’array possiede un unico indice condiviso) | — (un indice richiede ≥2 bande; vedi [Fotocamere mono e indici di vegetazione](mono-indices.md)) |

La radianza e la riflettanza non sono mai disponibili per le telecamere RGB

: la radianza per pixel Bayer non è significativa per un sensore fotometrico a banda larga.

### Acquisizione più veloce

Il pulsante **⚡ Acquisizione più veloce — solo raw**(di colore arancione quando attivo) sovrascrive tutte le selezioni di esportazione impostandole su**solo raw** — oltre a un composito con indice combinato gratuito per gli array — in modo che il fotogramma venga salvato il più rapidamente possibile: i calcoli relativi a radianza/riflettanza/visualizzazione vengono completamente saltati al momento dell’acquisizione.

{% hint style="info" %}
**Viene comunque salvato un file `.daq`.** Quando è assegnato un sensore di luce, l’opzione “Acquisizione più veloce” continua a scrivere la lettura DAQ in direzione discendente (nella cartella `daq/` delle acquisizioni) — in modo che i prodotti di radianza, riflettanza e indice possano essere tutti generati in un secondo momento tramite rielaborazione (vedi [Rielaborazione delle acquisizioni](#re-processing-captures-into-calibrated-products)). Inoltre, Fastest Capture non altera le selezioni effettuate tramite le caselle di controllo: disattivandolo, queste tornano alle impostazioni precedenti.
{% endhint %}

### Controlli per singolo array

Ogni array collegato dispone di una propria scheda di gruppo nel pannello:

* **Casella di controllo &quot;Includi&quot;** (a tre stati per i membri) e il nome dell’array con la relativa modalità di visualizzazione: «(combinata | separata)».
* Casella di controllo **Allineato**(impostazione predefinita**attiva**): applica una trasformazione ai file dei membri in base al profilo di allineamento dell’array, in modo che le esportazioni siano registrate pixel per pixel tra le diverse telecamere. I dati grezzi rimangono non distorti ma riportano la trasformazione nei propri metadati. (Il profilo stesso viene calcolato nel [pannello delle impostazioni dell’array](camera-settings.md#alignment-co-registration).) I file dei membri dell’array vengono salvati nell’orientamento nativo di ciascuna telecamera, con eventuali impostazioni di specchiatura/rotazione registrate nel file anziché applicate.
* Le righe delle telecamere che compongono l’array sono annidate all’interno della scheda.

La scheda dell’array ospita anche due registratori. Considerateli come **monitoraggio vs analisi**:

| Registratore | Livello | Cosa registra |
| --- | --- | --- |
| **● Registra video indice combinato / ■ Interrompe la registrazione** *(solo array combinati)* | **Monitoraggio** | Il composito in tempo reale dell’indice combinato in formato video a 10 fps — 8 bit, risoluzione di anteprima, LUT incorporata. Richiede un progetto aperto e una visualizzazione in streaming dal vivo. Mostra i fotogrammi e il tempo trascorso durante la registrazione. |
| **⦿ Registra raffica raw / ■ Interrompi sequenza raw** *(qualsiasi array)* | **Analisi**| Fotogrammi Bayer raw alla frequenza di acquisizione in diretta (senza elaborazione) più un manifesto per fotogramma e letture `.daq`, in formato `captures/bursts/`. Dopo una raffica, appare il pulsante**Crea video**: rielabora la raffica offline in un video calibrato — indice combinato e/o radianza / riflettanza / indice per singola telecamera — più TIFF opzionali. La creazione dell’indice combinato si avvia automaticamente quando si interrompe la raffica. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Il flusso

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

“Acquisisci tutto”

Premere **Acquisisci tutto** nell’elenco delle telecamere nella barra laterale:

1. Ogni telecamera inclusa, visibile e non in pausa acquisisce con i tipi di esportazione selezionati. **Gli array scattano come un unico trigger sincronizzato** (un unico gruppo sincronizzato tra tutti i membri — vedi [Array multicamera](arrays.md)); le telecamere autonome acquisiscono individualmente.
2. Le telecamere nascoste (con l’icona a forma di occhio) o in pausa vengono ignorate. Un array viene completamente bloccato solo quando *tutti* i suoi membri sono nascosti o in pausa.
3. Ogni volta che viene assegnato un sensore di luce, la lettura DAQ corrispondente della radiazione discendente viene salvata come file `.daq` nella cartella `daq/` delle acquisizioni — anche per le acquisizioni solo in formato raw — in modo che i prodotti radiometrici possano sempre essere ricavati in un secondo momento.
4. Il pulsante mostra lo stato di avanzamento in tempo reale — «Acquisizione in corso… completata/totale» — e in modalità Continua/Intervallo diventa **Stop (N)**. Ogni elemento di acquisizione ha un timeout di 300 s.
5. Al termine del passaggio, un messaggio lampeggiante riporta **&quot;N file salvati&quot;**o**&quot;N salvati, F non riusciti&quot;**, oltre a &quot;(S nascosti/in pausa/saltati)&quot; quando alcune telecamere sono state saltate. Se l’unità si riempie, l’esecuzione si interrompe e viene segnalato**&quot;Interrotto: unità piena. Salvati N, F falliti&quot;** — ogni file che non è stato possibile scrivere viene conteggiato come fallito.

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
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` è il timestamp dell’acquisizione e `<serial>` il numero di serie della telecamera. Le acquisizioni autonome sono denominate `capture_<ts>_SN<serial>_<level>`; le acquisizioni in array da un singolo trigger sincronizzato sono denominate `sync_<ts>_SN<serial>_<level>` e **condividono un unico timestamp tra tutte le telecamere del gruppo** (il suffisso del livello viene omesso quando una telecamera salva un solo livello).
* **Un&#x27;asimmetria da tenere presente:** il livello di visualizzazione viene memorizzato in una cartella denominata `preview/`, mentre i file mantengono `_display` nel nome — la cartella e il suffisso differiscono solo per quel livello.
* I livelli sconosciuti vengono salvati in una cartella con il proprio nome; se non è possibile creare una sottocartella, il file viene scritto nella directory principale delle acquisizioni anziché andare perso.
* I file TIFF delle acquisizioni sono compressi senza perdita di dati (DEFLATE) per impostazione predefinita e contengono tutti i metadati di calibrazione ed elaborazione **all’interno dell’XMP del file** — le acquisizioni sono autodescrittive, senza file sidecar oltre alla lettura `.daq` (e alla sua `.daq.cameras.json`).
* **Maschere di clipping (novità nella versione 1.3.0).**Laddove la separazione spettrale 3×3 doveva azzerare una banda su almeno un pixel, i prodotti di radianza, riflettanza e indice ricevono una maschera per pixel, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = la banda nel canale R del prodotto, bit 1 = G, bit 2 = B); il tag XMP del prodotto, `Chloros:UnmixClampMask`, punta a tale maschera. Le maschere si trovano**accanto** alle cartelle dei prodotti, mai al loro interno, quindi una cartella di prodotti come `reflectance/` può essere consegnata al software di fotogrammetria così com’è. L’importazione di Chloros

ignora le maschere.
* **Orientamento (1.3.0).** I prodotti Preview, Radiance, Reflectance e Index di una fotocamera autonoma vengono ruotati in base all’impostazione Mirror/Rotation; i file Raw e Debayered rimangono così come sono stati letti dal sensore. Tutti i file membri dell’array rimangono nel formato nativo. Ogni file registra l’impostazione nel proprio XMP, quindi la rielaborazione la applica allo stesso modo.
* **A quale fotocamera appartiene un `.daq` (1.3.0).** L’`.daq` di un’acquisizione viene abbinato, al momento dell’elaborazione, solo alle immagini provenienti dalle fotocamere indicate dal suo `.daq.cameras.json`. Una telecamera autonoma priva di un proprio sensore di luce non utilizza quindi più la lettura di un’altra telecamera: è necessario associarle un sensore oppure registrarne uno nella scheda “Sensori di luce”. Una registrazione del sensore di luce (e qualsiasi `.daq` privo di quel file, comprese le acquisizioni precedenti) continua a servire tutte le telecamere.

Si tratta dello stesso layout che i file `chloros-cli lattice capture` / `array-capture` scrivono nella loro directory `-o` — documentato nel [Riferimento CLI

§ Come è strutturata una cartella delle acquisizioni](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Rielaborazione delle acquisizioni in prodotti calibrati

I fotogrammi grezzi acquisiti, insieme al file `.daq` salvato, sono tutto ciò di cui la pipeline di elaborazione ha bisogno: ecco perché Fastest Capture è affidabile per il lavoro vero e proprio.

* **GUI**: aggiungere la cartella delle acquisizioni a un progetto ([Aggiunta di file a un progetto](../processing-images-gui/adding-files-to-a-project.md)) ed eseguire l’elaborazione come di consueto.
* **CLI**: indicare a `process` la **radice delle acquisizioni**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` normalmente importa solo la cartella specificata, ma quando tale cartella non contiene immagini e presenta sottocartelle, scende automaticamente nei livelli inferiori — in questo modo le sottocartelle di livello e la cartella `daq/` vengono acquisite in un unico passaggio. Ogni acquisizione viene importata come **singola immagine** con gli altri livelli allegati come modalità visualizzabili, non come un’immagine per ogni livello.

È possibile anche specificare direttamente il nome di una sottocartella di livello (ad es. `…/captures/raw/`), ma in questo caso la cartella `daq/` viene tralasciata: copiatela insieme alle altre quandoderivare un prodotto radiometrico da `raw/`, altrimenti la corrispondenza del timestamp non avrà alcun riferimento a cui fare riferimento.

{% hint style="warning" %}
**L’elaborazione parte sempre da `raw`.** All’interno di ciascuna acquisizione, il fotogramma grezzo è la sorgente della pipeline; `debayered`, `radiance`, `reflectance`, e `preview` sono presenti come modalità visualizzabili ma non vengono mai reimmessi nella pipeline — rielaborare un prodotto derivato significherebbe riapplicare la vignettatura, il colore e i calcoli di radianza già integrati nei suoi pixel, quindi Chloros

preferisce rifiutarli piuttosto che eseguire una doppia elaborazione. I rendering `index/` e `composite/` non vengono mai elaborati (sono output, non acquisizioni). Una cartella «captures» salvata **senza** importazioni raw viene visualizzata normalmente, ma `process` la salta e lo segnala; `--input-level {raw,debayered,processed}` è la via di fuga intenzionale che forza un punto di ingresso. Consultare il [Riferimento CLI

](../reference/cli-reference.md#what-a-captures-folder-looks-like) per i messaggi di salto esatti.
{% endhint %}

Altri due comportamenti da conoscere quando si scrivono script per la rielaborazione:

* Un&#x27;esecuzione di `chloros-cli process` che ha richiesto prodotti ma non ha scritto **alcun prodotto immagine fallisce in modo evidente e termina con un codice di uscita diverso da zero** — non si otterrà mai un&#x27;esecuzione vuota silenziosa. Le esecuzioni riuscite riportano il numero dei prodotti. (Un&#x27;esecuzione deliberata solo con metadati viene comunque considerata un successo.)
* Le esportazioni elaborate e reimportate non occupano mai lo slot raw di una cattura — il raw originale rimane sempre la fonte della pipeline.

## Equivalenti CLI



Tutto ciò che è descritto in questa pagina può essere eseguito in modalità headless. Le modalità di cattura della GUI corrispondono direttamente a `chloros-cli lattice array-capture`:

| GUI | CLI

|
| --- | --- |
| Singola | `chloros-cli lattice array-capture` |
| Continua | `array-capture --continuous [--count N] [--duration S]` |
| A intervalli | `array-capture --interval S [--duration S]` |
| Acquisizione più veloce | `array-capture --fastest` |
| Casella di controllo “Allineato” | `--aligned / --no-aligned` |
| Caselle di controllo relative al tipo di esportazione | `--processing LEVEL` oppure `--levels L1,L2,…` (impostazione predefinita `all`) |
| Registra video indice | `chloros-cli lattice array-record` |
| Registra raffica raw / Crea video | `chloros-cli lattice array-burst` / `array-build-video` |

Le tabelle complete dei flag, l’opzione di acquisizione stabilizzata con AE intelligente (`--smart`) e il modello a velocità costante sono descritti nella sezione [Riferimento CLI

§ Modalità di acquisizione, registratori e rielaborazione offline](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
