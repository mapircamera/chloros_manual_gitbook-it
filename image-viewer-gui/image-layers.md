# Livelli dell’immagine

Il **menu a tendina dei livelli** in alto a destra nel Visualizzatore di immagini consente di passare da una versione all’altra dell’immagine che si sta visualizzando — dall’acquisizione originale, passando per ogni prodotto elaborato, fino alle immagini indice calcolate — senza uscire dal visualizzatore.

## Cosa sono i livelli dell’immagine?

Un “livello” in Chloros è un **file di prodotto**associato a un’immagine originale. L’importazione fornisce i file originali; l’elaborazione aggiunge un livello per ciascun prodotto generato dall’esecuzione. I file esportati mantengono il nome del file sorgente: è la**cartella** a identificare il prodotto, mentre il nome del livello è l’etichetta assegnata da Chloros a quella cartella.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## L’elenco dei livelli

### Sempre presenti

| Livello | Descrizione |
| --- | --- |
| **JPG**(o**PNG**/**TIFF**) | Il file di base importato con l’acquisizione. Survey3 importa un `.JPG` accanto a ogni `.RAW`; Le acquisizioni LATTICE forniscono un’anteprima di visualizzazione PNG o TIFF. Etichettata in base a ciò che è stato effettivamente importato |
| **RAW (Originale)** | Il fotogramma raw di origine, sottoposto a debayering per la visualizzazione senza correzioni applicate. Disponibile dal momento dell’importazione — non necessita di elaborazione |

Una cattura LATTICE il cui file di base **è** il suo fotogramma raw non ha una voce di base separata: `RAW (Original)` la copre già.

### Prodotti di elaborazione Survey3

| Livello | Scritto su | Esiste quando |
| --- | --- | --- |
| **RAW (Target)** | — | Il fotogramma è stato identificato come contenente un target di calibrazione |
| **RAW (Riflettanza)** | `Reflectance_Calibrated_Images/` | La calibrazione della riflettanza è stata eseguita con successo su questo fotogramma |
| **Correzione vignettatura**| `Vignette_Corrected_Images/` | Non è stato possibile eseguire la calibrazione della riflettanza sul fotogramma**e** la *correzione della vignettatura* era attiva |
| **Risposta del sensore**| `Sensor_Response_Images/` | Non è stato possibile eseguire la calibrazione della riflettanza del fotogramma**e** la *correzione della vignettatura* era disattivata |
| **Bilanciamento del bianco** | `White_Balanced_Images/` | È stato generato un prodotto con bilanciamento del bianco |

{% hint style="info" %}
**La correzione della vignettatura e la risposta del sensore sono alternative, mai entrambe.** Per ogni modello di fotocamera esiste esattamente un prodotto di riserva non calibrato per ogni esecuzione, e l’opzione *Correzione della vignettatura* determina quale utilizzare. Vedi [Impostazioni del progetto](../project-settings/project-settings.md).
{% endhint %}

### Livelli LATTICE

LATTICE acquisisce il fan-out in questi livelli in un unico passaggio di elaborazione. Quali livelli siano presenti dipende dalle opzioni di esportazione per singolo prodotto nelle Impostazioni del progetto e da ciò che si applica alla fotocamera.

| Livello | Scritto su | Si applica a |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB e multispettrale |
| **RAW (anteprima)** | `Preview_Images/` | Multispettrali (stretching a falsi colori) |
| **Bilanciamento del bianco** | `Preview_Images/` | Telecamere master RGB — l’anteprima RGB è registrata con questo nome in modo da allinearsi con il livello Survey3 dallo stesso nome |
| **RAW (radianza)** | `Radiance_Images/` | Solo multispettrale |
| **RAW (Radianza)** | `Reflectance_Calibrated_Images/` | Solo multispettrale, e solo quando un record di radiazione discendente `.daq` corrispondente o un bersaglio all’interno del fotogramma che ha superato il controllo di qualità (QA) copre il fotogramma |

Le telecamere master RGB non dispongono di radiometria per banda, pertanto radianza e riflettanza vengono ignorate come **non applicabili** — il log lo segnala invece di generare un errore silenzioso.

### Livelli di indice, LUT e sandbox

| Schema del livello | Esempio | Da dove proviene |
| --- | --- | --- |
| **RAW (`<INDEX>` Indice)** | `RAW (NDVI Index)` | Uno per ogni indice configurato nelle Impostazioni del progetto, calcolato durante l’elaborazione |
| **`<INDEX>` LUT** | `NDVI LUT` | La versione con mappatura dei colori di un indice |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Uno per ogni ciclo di esportazione [Indice/LUT Sandbox](index-lut-sandbox.md) |

Se lo stesso nome di indice viene configurato più di una volta con impostazioni diverse, al secondo e a quelli successivi viene aggiunto un numero nel nome (`RAW (NDVI2 Index)`) in modo che i livelli rimangano distinguibili.

***

## Utilizzo del selettore di livelli

1. Aprire un’immagine a schermo intero facendo clic su una miniatura nella griglia
2. Fare clic sul **menu a tendina dei livelli** in alto a destra nel visualizzatore
3. Selezionare un livello: l’immagine si aggiorna immediatamente

Il menu a tendina mostra per primi **JPG, RAW (Originale), RAW (Destinazione), RAW (Riflettanza)**, in quest’ordine, e elenca tutto il resto dopo di essi nell’ordine in cui i prodotti sono stati registrati.

### Preferenza dei livelli durante la navigazione

Premendo **←**/**→** si passa all’immagine successiva cercando di mantenere lo stesso livello:

1. **Prima la corrispondenza esatta** — se l’immagine successiva ha un livello con lo stesso nome, viene visualizzato quello. È questo che ti mantiene su `RAW (NDVI Index)` mentre scorri l’intera serie
2. **Poi una corrispondenza per tipo** — un livello indice cerca qualsiasi livello indice, una LUT qualsiasi LUT, un livello di riflettanza un altro livello di riflettanza, un livello target un altro livello target, un livello originale un altro livello originale, un livello base un altro livello base
3. **Infine, solo per i livelli di esportazione** — il nome viene mantenuto anche se l’elenco dei livelli non è ancora aggiornato, poiché il file esiste già sul disco. È questo che consente di rivedere i prodotti mentre un’elaborazione li sta ancora scrivendo
4. **Altrimenti** — il primo livello disponibile, che normalmente è l’immagine di base

I file sidecar `.daq` e `.csv` presenti nel progetto vengono saltati durante la navigazione con i tasti freccia, quindi scorrendo le immagini non si arriva mai a una registrazione del sensore di luce.

Anche lo zoom e lo scorrimento si applicano alle immagini, il che semplifica il confronto prima/dopo della stessa posizione sul campo.

***

## Comprendere i valori dei pixel per livello

Il [pannello Valori del cursore](opening-an-image-full-screen.md#cursor-values) riporta il valore reale per canale sotto il cursore, nell’unità in cui è memorizzato il livello. Le sue colonne cambiano a seconda del livello:

| Livello | Unità riportata | Note |
| --- | --- | --- |
| Base (JPG / PNG / anteprima TIFF) | DN, 0–255 | Valori visualizzati, con correzione gamma su RGB. Solo ispezione visiva |
| RAW (Originale) | DN | Valori digitali grezzi del sensore. L’asse dell’istogramma indica la profondità: 255 (8 bit), 4095 (12 bit) o 65535 (16 bit) |
| RAW (Debayered) | DN | Lineare, senza allungamento della visualizzazione |
| RAW (Anteprima) / Bilanciamento del bianco | DN | Risultato visualizzato — esteso o con correzione gamma. Non destinato alla misurazione |
| RAW (Radianza) | **W/m²/sr/nm** | Radianza fisica in Float32. Nessuna colonna DN |
| RAW (Riflettanza) | DN **e %** | Percentuale calcolata in base alla scala propria del file — vedi sotto |
| Esportazioni indice / LUT / sandbox | Valore dell’indice, o componenti RGB | Un file indice a canale singolo riporta il valore dell’indice; un file LUT con mappatura cromatica riporta le componenti Red/Green/Blue |

### Riflettanza: la scala è specifica per ogni file

{% hint style="warning" %}
**&quot;Dividere per 65.535&quot; è corretto solo per Survey3.** La riflettanza LATTICE è memorizzata con una scala diversa, e mescolare i due divisori è il modo più comune per ottenere valori di riflettanza pari esattamente alla metà di quelli che dovrebbero essere.
{% endhint %}

| Origine | DN corrispondente a riflettanza 1,0 | Identificato da |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | Il tag XMP `Chloros:PixelScale=32768` inserito in ogni esportazione di riflettanza LATTICE. Il margine di 2× significa che valori di ρ superiori a 1,0 sono rappresentabili anziché troncati |
| **Survey3**|**65535** | In assenza del tag di scala XMP Chloros, la calibrazione Survey3 scrive ρ × dtype-max e troncano il valore a 1,0 |

Per GIS e scripting: leggere `Chloros:PixelScale` dal file e dividere per esso. Se il tag è assente, il file è in scala Survey3 (65535). Il visualizzatore, l’area di prova dell’indice/LUT e l’esportazione dell’indice risolvono tutti la scala allo stesso modo, quindi il numero che si legge in corrispondenza del cursore è lo stesso utilizzato dai calcoli dell’indice.

Memorizzazione specifica per formato in aggiunta a tale scala:

* **TIFF (32 bit, percentuale)** memorizza DN / 65535 come numero in virgola mobile
* **PNG (8 bit)**e**JPG (8 bit)** memorizzano DN × 255 / 65535
* Un’**esportazione a 8 bit TIFF di un’acquisizione da sorgente a 8 bit** viene limitata a 0–255 anziché riscalata e, deliberatamente, non riporta alcun tag di scala. Il pannello visualizza solo il DN per quei file, senza la colonna della percentuale

### Intervalli dei valori dell&#x27;indice

| Famiglia di indici | Intervallo tipico | Lettura |
| --- | --- | --- |
| Differenza normalizzata (NDVI, GNDVI, NDRE, ENDVI…) | da −1 a +1 | Vegetazione sana solitamente 0,4–0,9; suolo nudo vicino a 0; acqua: valore negativo |
| Corretto per il suolo (SAVI, OSAVI, MSAVI2…) | approssimativamente da −1 a +1,5 | Valore simile a NDVI con lo sfondo del suolo soppresso |
| Rapporto (GRVI, GCI, MSR, CIRE…) | illimitato verso l’alto | I rapporti crescono senza limiti man mano che la banda del denominatore tende a zero |
| EVI / LAI | da 0 a ~1, da 0 a ~3,5 | Le nuvole e altri pixel saturi fanno uscire entrambi dal range — mascherarli prima |

Vedi [Formule degli indici multispettrali](../project-settings/multispectral-index-formulas.md) per la formula esatta alla base di ogni preimpostazione.

***

## Flussi di lavoro comuni

### Confronto prima/dopo

1. Seleziona **RAW (Originale)** e prendi nota della vignettatura e dei valori non calibrati
2. Passa a **RAW (Riflettanza)**

3. Confronta: la vignettatura è stata rimossa, i valori sono stati calibrati. Lo zoom e la panoramica rimangono fissi, quindi stai osservando lo stesso punto del terreno

### Esamina un indice su un intero set

1. Apri la prima immagine elaborata e seleziona il livello indice
2. Premi ripetutamente **→**: il livello indice ti segue di immagine in immagine
3. Osserva l’istogramma nella barra laterale man mano che procedi: un fotogramma la cui distribuzione presenta picchi merita un’analisi più approfondita

### Verifica dei target di calibrazione

1. Seleziona **RAW (Target)** su un fotogramma di riferimento
2. Verifica che il bersaglio sia chiaramente visibile e rilevato
3. Passa al fotogramma di riferimento successivo: il livello del bersaglio ti segue

### Controlla l’accuratezza dei valori di riflettanza

1. Seleziona **RAW (Reflectance)**

2. Leggi la colonna**%** nel pannello Valori del cursore: è già scalata correttamente per quel file
3. Verifica la correttezza confrontando con materiali noti presenti nel fotogramma: la vegetazione sana presenta valori elevati di NIR e bassi di rosso; un bersaglio di calibrazione dovrebbe presentare valori vicini alla sua riflettanza pubblicata

***

## Risoluzione dei problemi

### Un livello che mi aspettavo non è presente nel menu a tendina

**Possibili cause**

* L’immagine non è mai stata elaborata — esistono solo il livello di base e `RAW (Original)`
* L’opzione di esportazione del prodotto non è selezionata nelle Impostazioni del progetto
* Il prodotto non è applicabile a quella telecamera (radianza e riflettanza su un master RGB; qualsiasi indice su una telecamera mono M3M a banda singola)
* La calibrazione della riflettanza non aveva dati su cui basarsi — nessuna copertura downwelling `.daq` e nessun target nell’immagine che avesse superato il controllo di qualità — quindi l’immagine è ricaduta su “Vignette Corrected” o “Sensor Response”

**Cosa fare**

1. Controllare il log dell’esecuzione: Chloros indica quando un prodotto di esportazione richiesto era impossibile da ottenere e il motivo
2. Controllare i pulsanti di attivazione/disattivazione dell’esportazione per ciascun prodotto in [Impostazioni del progetto](../project-settings/project-settings.md)
3. Verifica che la cartella del prodotto esista nell’albero di output del progetto
4. Eseguisci nuovamente l’elaborazione con il prodotto abilitato

### L’elenco dei livelli sembra non aggiornato

Chloros esegue una nuova scansione delle cartelle dei prodotti del progetto mentre un&#x27;esecuzione è in corso e corregge le registrazioni dei livelli mancanti in base a ciò che è effettivamente presente sul disco; pertanto, un livello la cui esportazione è stata completata correttamente appare automaticamente durante un sondaggio. Passare a un&#x27;altra schermata e poi tornare indietro forza un nuovo aggiornamento.

### I valori di riflettanza sembrano pari alla metà di quelli che dovrebbero essere

Quasi certamente si sta dividendo un file LATTICE per 65535. Utilizzare `Chloros:PixelScale` (32768), oppure leggere la colonna **%**, in cui tale valore è già stato applicato.

### Il livello indice esiste ma l’immagine è vuota

L’indice richiede bande che il tuo livello non possiede — ad esempio, un indice che legge un terzo canale applicato a un file a uno o due canali. Passa a un livello multibanda (riflettanza o debayering), oppure scegli un indice compatibile con il filtro della fotocamera.

***

## Passi successivi

* [**Apertura di un&#x27;immagine a schermo intero**](opening-an-image-full-screen.md) — lettura del cursore, istogramma e controllo GSD
* [**Area di prova per indici/LUT**](index-lut-sandbox.md) — visualizzazione interattiva degli indici ed esportazione
* [**Formule degli indici multispettrali**](../project-settings/multispectral-index-formulas.md) — il riferimento agli indici
* [**Completamento dell’elaborazione**](../processing-images-gui/finishing-the-processing.md) — l’albero della cartella di output a cui puntano questi livelli
