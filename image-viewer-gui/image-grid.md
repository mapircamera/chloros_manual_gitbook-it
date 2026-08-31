# Griglia delle immagini

Dopo aver importato le immagini in un progetto, le vedrai disposte in una griglia nell’area principale. La griglia è il luogo in cui puoi scegliere **quale versione di ogni immagine visualizzare**: i pulsanti situati sopra di essa consentono di passare contemporaneamente da una miniatura all’altra, passando dai file sorgente a ciascun prodotto elaborato.

## Dimensione delle miniature

Utilizza il cursore di zoom in alto a destra per regolare la dimensione delle miniature delle immagini. Il cursore va da **64 px a 1200 px**.

* Anche **Ctrl + rotellina del mouse** consente di ridimensionare le miniature.
* **Ctrl + `+`**/**Ctrl + `=`**e**Ctrl + `−`** aumentano o diminuiscono la dimensione di 4 px per ogni pressione. L&#x27;intervallo di regolazione tramite tastiera termina a 64 px nella dimensione minima e, nella dimensione massima, a qualsiasi dimensione che permetta di inserire esattamente due miniature per riga nella finestra corrente.
* La dimensione scelta viene salvata insieme al progetto (`UI → Grid thumbnail size` in `project.json`, valore predefinito `160`), quindi riaprendo il progetto verrà ripristinata.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>La *risoluzione* delle miniature è un’impostazione distinta dalla *dimensione* delle miniature: vedi **Visualizza → Risoluzione miniature immagini** in [Impostazioni progetto](../project-settings/project-settings.md) (impostazione predefinita 512 px sul lato più lungo). La dimensione indica quanto è grande il riquadro disegnato; la risoluzione indica la quantità di dettagli recuperati per riempirlo.***

## La barra degli strumenti della griglia

La fila di pulsanti sopra la griglia presenta fino a tre gruppi, da sinistra a destra:

1. **Per trigger / Per telecamera** — modalità di raggruppamento. Appare solo per i progetti contenenti acquisizioni LATTICE.
2. **Pulsanti filtro telecamera** — uno per ogni telecamera LATTICE. Appare solo in modalità Per telecamera.
3. **Pulsanti modalità esportazione/visualizzazione** — quale prodotto viene mostrato da ciascuna miniatura.

Quando la finestra è troppo stretta per contenerli tutti, i gruppi si comprimono da destra a sinistra in menu a tendina che si aprono al passaggio del mouse: prima si comprimono i pulsanti di esportazione/visualizzazione, poi quelli delle telecamere. Il gruppo compresso lascia visibile un unico pulsante contrassegnato dall’opzione attualmente attiva; passandoci sopra con il mouse, si fa scorrere verso il basso l’intera serie. **I pulsanti &quot;Per trigger&quot; e &quot;Per telecamera&quot; non si comprimono mai.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Pulsanti di esportazione e visualizzazione

Questi pulsanti consentono di passare da un tipo di immagine all’altro nella griglia delle miniature. **Un pulsante appare non appena esiste il prodotto a cui fa riferimento** — il che, per i file sorgente, significa immediatamente al momento dell’importazione, non dopo l’elaborazione. Chloros esegue una nuova scansione dei prodotti del progetto mentre un’elaborazione è in corso, quindi i pulsanti compaiono durante l’elaborazione man mano che ogni prodotto viene salvato sul disco.

### Il pulsante di base

Il pulsante di esportazione più a sinistra riporta **ciò che è stato effettivamente importato**:

| Cosa hai importato | Etichetta del pulsante |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| Acquisizioni LATTICE con anteprima sullo schermo accanto al fotogramma RAW | `PNG` o `TIFF`, a seconda di quali siano le anteprime |
| Acquisizioni LATTICE in cui il file di base **è** il fotogramma raw | *nessun pulsante* — `RAW (Original)` mostra già quel file |

In un progetto misto, l’etichetta segue l’estensione utilizzata dalla maggior parte delle immagini.

### Pulsanti del prodotto

| Pulsante | Mostra | Quando appare |
| --- | --- | --- |
| **Obiettivi** | Immagini con un obiettivo di calibrazione rilevato | Dopo un&#x27;esecuzione che ha rilevato degli obiettivi |
| **Riflettanza** | Immagini di riflettanza calibrate | Solo nei progetti Survey3 — I progetti LATTICE utilizzano invece `RAW (Reflectance)`, quindi la griglia non mostra mai due pulsanti di riflettanza |
| **Bilanciamento del bianco** | Il prodotto con bilanciamento del bianco (telecamere RGB) | Dopo l’elaborazione |
| **Correzione della vignettatura** | Il valore di ripiego non calibrato con correzione della vignettatura | Dopo un&#x27;esecuzione in cui non è stato possibile applicare la calibrazione della riflettanza e la *correzione della vignettatura* era attiva |
| **Risposta del sensore** | L&#x27;opzione di riserva non calibrata con risposta del sensore | Lo stesso, ma con la *correzione della vignettatura* disattivata |
| **`RAW (<INDEX> Index)`** | Un pulsante per ogni indice calcolato | Dopo un ciclo con indici configurati |
| **`<INDEX> LUT`** | Un pulsante per ogni indice con mappatura cromatica | Dopo un&#x27;esecuzione con una LUT configurata |
| **`<Index> <Index\|LUT> <NNN>`** | Un pulsante per ogni esecuzione di esportazione [Index/LUT Sandbox](index-lut-sandbox.md) | Nel momento in cui termina un’esportazione sandbox |

### Pulsanti di livello LATTICE

I progetti contenenti acquisizioni LATTICE aggiungono questi pulsanti, etichettati con il nome del livello anziché con il nome del prodotto:

| Pulsante | Livello |
| --- | --- |
| **RAW (Originale)** | Il fotogramma raw di origine, così come è stato importato |
| **RAW (Radianza)** | Radianza spettrale in Float32, W/m²/sr/nm |
| **RAW (Riflettanza)** | Riflettanza in uint16, 32768 = ρ 1,0 |

`RAW (Original)` è disponibile fin dal momento dell’importazione — non richiede alcuna elaborazione. Quando un’importazione LATTICE non presenta alcun pulsante di base (il file di base di ogni acquisizione è il suo fotogramma grezzo), la griglia si sposta automaticamente sul primo pulsante di livello disponibile in modo che l’evidenziazione sulla barra degli strumenti corrisponda a ciò che si sta visualizzando.

Le esportazioni a due livelli Chloros **non dispongono di un proprio pulsante di griglia**:

* **Debayered** — la vista `RAW (Original)` viene già visualizzata senza debayering, quindi un secondo pulsante su un’immagine visivamente identica sarebbe superfluo. Il prodotto `RAW (Debayered)` viene comunque salvato su disco ed è ancora selezionabile dal menu a tendina dei livelli a schermo intero.
* **Anteprima** — sulle telecamere RGB l&#x27;anteprima è registrata come livello `White Balanced`, che dispone invece di un pulsante. Sulle telecamere multispettrali è registrato come `RAW (Preview)` ed è accessibile dal menu a tendina dei livelli a schermo intero.

{% hint style="info" %}
Questi pulsanti di livello vengono visualizzati solo per i progetti che contengono effettivamente fotogrammi LATTICE. I progetti Survey3 registrano alcuni degli stessi nomi di livelli interni, e i pulsanti vengono filtrati per essi, in modo che una griglia Survey3 mantenga il suo familiare set `JPG / Targets / Reflectance`.
{% endhint %}

Facendo clic su una miniatura della griglia si apre il [Visualizzatore immagini](opening-an-image-full-screen.md) a schermo intero su **lo stesso prodotto mostrato dalla griglia** — se la griglia è impostata su `Targets`, la miniatura apre l’immagine di destinazione esportata.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Raggruppamento di un progetto LATTICE: Per Trigger vs Per Telecamera

Le acquisizioni in array producono diverse immagini dello stesso istante da diversi moduli telecamera. Il raggruppamento determina come la griglia le impila. Entrambe le modalità visualizzano barre di intestazione comprimibili a larghezza intera; **ogni gruppo si apre per impostazione predefinita** e Chloros ricorda quelle che chiudi. Lo stato di compressione viene tracciato separatamente per ciascuna modalità, quindi chiudere un gruppo in «Per telecamera» non chiude nulla in «Per trigger».

### Per telecamera (impostazione predefinita)

Un gruppo per ogni modulo telecamera. L’intestazione mostra il modello e il numero di serie della telecamera (`LATT-M3M — <serial>`) e il numero di foto. Le immagini all’interno di un gruppo sono ordinate cronologicamente in base all’evento di acquisizione.

In questa modalità la barra degli strumenti presenta anche un **pulsante di filtro per telecamera** per ogni telecamera, contrassegnato con `MODEL (SERIAL)`. Tutte le telecamere sono inizialmente selezionate; cliccando su un pulsante si deseleziona quella telecamera e si rimuove il relativo gruppo dalla griglia. Questo è il modo più rapido per esaminare una singola banda lungo l’intero volo.

### Per trigger

Un gruppo per ogni evento di acquisizione: l’insieme dei fotogrammi ripresi da tutti i moduli con lo stesso trigger. L’intestazione mostra l’ora di acquisizione, il numero di telecamere coinvolte e un’icona per ogni modello di telecamera presente nel gruppo. Le tessere all’interno di un gruppo sono ordinate in base al numero di serie della fotocamera, quindi la stessa banda si trova nella stessa colonna per ogni trigger.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Le immagini non LATTICE in un progetto misto non vengono raggruppate — vengono visualizzate come semplici tessere dopo i gruppi.

***

## Le miniature della griglia seguono la dimensione del blocco GSD

Se hai impostato una dimensione del blocco **GSD (px)** nella barra laterale della scheda &quot;Immagine&quot;, le miniature della griglia vengono visualizzate con quella stessa risoluzione al suolo — non solo nella vista a schermo intero. Una dimensione del blocco pari a 8 significa che ogni pixel visualizzato è la media di un blocco di 8 × 8 pixel sorgente, in ogni punto dell’app in cui viene mostrata l’immagine.

Poiché una tessera ha una larghezza iniziale di solo un paio di centinaia di pixel, le dimensioni dei blocchi grossolane smettono di fare una differenza visibile sulla griglia ben prima che ciò avvenga nella visualizzazione a schermo intero: un fotogramma da 4000 px disegnato in una tessera da 160 px ha già circa 25 pixel sorgente per ogni pixel visualizzato. Vedi [Apertura di un’immagine a schermo intero](opening-an-image-full-screen.md#gsd-block-size) per il controllo stesso.

***

## Pagine correlate

* [**Aprire un’immagine a schermo intero**](opening-an-image-full-screen.md) — il visualizzatore a schermo intero, i valori del cursore e l’istogramma
* [**Livelli dell&#x27;immagine**](image-layers.md) — il menu a tendina dei livelli all’interno del visualizzatore a schermo intero
* [**Sandbox Indice/LUT**](index-lut-sandbox.md) — creazione ed esportazione di visualizzazioni dell’indice
* [**Impostazioni del progetto**](../project-settings/project-settings.md) — i pulsanti di esportazione che determinano quali prodotti sono effettivamente disponibili
