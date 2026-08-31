# Completamento dell&#x27;elaborazione

Una volta che Chloros ha completato l&#x27;elaborazione, è il momento di esaminare i risultati, verificare la qualità dell&#x27;output e preparare le immagini elaborate per l&#x27;utilizzo nel flusso di lavoro. Questa pagina ti guida attraverso i passaggi finali e le azioni successive.

## Indicatori di completamento dell’elaborazione

Quando l’elaborazione termina con successo, vedrai diversi indicatori:

* ✅ **Barra di avanzamento**: raggiunge il 100% di completamento
* ✅ **Log di debug**: mostra le righe finali di `[RUN-SUMMARY]` con i conteggi (immagini, gruppi di telecamere, target, immagini calibrate, file scritti)
* ✅ **Pulsante Avvia**: torna attivo (pronto per la prossima esecuzione dell’elaborazione)
* ✅ **File di output**: tutte le immagini elaborate vengono salvate nella struttura di output del progetto (di seguito)

{% hint style="warning" %}
**Un ciclo che non scrive alcuna immagine è considerato un errore.** Se sono stati richiesti prodotti immagine e l’esecuzione non ne ha salvata alcuna, Chloros la segnala come un errore — `[RUN-SUMMARY]` suggerisce nel nome del log la causa probabile (nulla importato, nessun bersaglio rilevato o tutti i prodotti richiesti saltati in quanto non applicabili). L’equivalente CLI termina con un codice di uscita diverso da zero. Un’esecuzione deliberata solo con metadati (tutti i prodotti di esportazione disattivati, nessun indice) è comunque considerata un successo. Vedi [il riferimento a CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Individuazione delle immagini elaborate

### Apertura della cartella di output

1. Fare clic sull’icona **Menu principale** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (in alto a sinistra)
2. Selezionare **&quot;Apri cartella del progetto&quot;**

3. Si aprirà Esplora file nella directory del progetto
4. Individuare il progetto in base al nome

### L’albero di output

I prodotti vengono salvati **nella cartella del progetto, raggruppati per fotocamera e poi per formato di file**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Cartella della fotocamera**: `LATT-<sensor>-<lens>-F<filter>` per LATTICE (corrispondente all’EXIF dell’acquisizione `Model`), `<model>_<filter>` per Survey3 (ad es. `Survey3N_RGN`). Due fotocamere che condividono sensore e filtro ma hanno obiettivi diversi mantengono alberi separati: vignettatura, campo visivo e distorsione differiscono.
* **Cartella del formato**: segue l’impostazione del formato di esportazione — `tiff16`, `tiff8`, `png8`, `jpg8`, oppure `tiff32` per TIFF (32 bit, percentuale). Radiance è sempre in formato float32 e viene sempre salvato in `tiff32`.
* **Cartelle dei prodotti**:
  * `Reflectance_Calibrated_Images/` — riflettanza calibrata
  * `Debayered_Images/` — debayering lineare (LATTICE)
  * `Preview_Images/` — anteprima su schermo (LATTICE)
  * `Radiance_Images/` — radianza spettrale in float32, W/m²/sr/nm (LATTICE multispettrale)
  * `Vignette_Corrected_Images/` **oppure** `Sensor_Response_Images/` — valore di ripiego non calibrato per i fotogrammi privi di riferimento di riflettanza; per ogni esecuzione ne esiste esattamente uno dei due, scelto in base all’impostazione di correzione della vignettatura
  * `<INDEX>_Index_Images/` — una cartella per ogni indice selezionato (ad es. `NDVI_Index_Images`)

{% hint style="info" %}
**Ogni prodotto esportato mantiene il nome del file ORIGINALE.**Un’esportazione di radianza di `capture_..._raw.tif` si chiama comunque `capture_..._raw.tif` — semplicemente si trova in `tiff32/Radiance_Images/`.**È la cartella a identificare il prodotto, non il nome del file**, quindi cercando `*radiance*.tif` non si trova nulla; cercare invece in base alla directory.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Quanti file dovrebbero esserci?

Non basarsi su una formula: il numero di file di output dipende dai prodotti abilitati e da quelli applicabili a ciascuna telecamera (ad esempio, le telecamere RGB non generano dati di radianza/riflettanza). Il conteggio ufficiale si trova nel log: l’ultima riga `[RUN-SUMMARY]` riporta esattamente quanti file sono stati scritti, mentre le righe di suggerimento spiegano tutto ciò che è stato saltato.

***

## Revisione delle immagini elaborate

### Anteprima rapida in Esplora file

**Anteprima integrata in Windows:**

1. Accedere a una cartella del prodotto (ad es. `tiff16/Reflectance_Calibrated_Images/`)
2. Selezionare un file immagine
3. L’anteprima appare nel riquadro di anteprima di Windows Explorer
4. Utilizzare i tasti freccia per scorrere le immagini

### Anteprima in visualizzatori di immagini esterni

**Visualizzatori consigliati:*** **QGIS** - Software GIS gratuito (ideale per l’analisi multispettrale georeferenziata)
* **IrfanView** - Visualizzatore di immagini veloce e leggero (supporta TIFF)
* **Adobe Photoshop** - Editing professionale (supporta TIFF)
* **GIMP** - Alternativa gratuita a Photoshop
* **Windows Photos** - Visualizzazione di base (potrebbe non supportare TIFF a 16 bit)

### Anteprima nel visualizzatore di immagini di Chloros

Utilizza il Visualizzatore di immagini integrato in Chloros per una visualizzazione avanzata:

1. Fai clic su una miniatura dell’immagine nel Browser dei file
2. L’immagine si apre nell’area di anteprima principale
3. Fai clic sulla scheda **Visualizzatore di immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> nella barra laterale sinistra
4. Utilizza [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) per l’analisi interattiva

Consulta [Visualizzatore di immagini](../image-viewer-gui/opening-an-image-full-screen.md) per istruzioni dettagliate.

***

## Lettura dei valori di riflettanza dei pixel (GIS / Pix4D / Script)

La riflettanza è memorizzata come valore intero DN, e **il valore DN che corrisponde a ρ = 1,0 dipende dalla fotocamera di origine**:

| Origine          | ρ = 1,0 corrisponde a | Come individuarlo                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (margine fino a ρ 2,0) | Il tag XMP `Chloros:PixelScale=32768` è inserito nel file |
| Survey3         | **65535** (limitato a ρ 1,0)     | Nessun tag XMP `Chloros:*` — tale assenza è il segnale |

**Leggere il tag `Chloros:PixelScale` e dividere per quel valore** anziché ipotizzare un valore generico pari a 65535 — dividere la riflettanza LATTICE per 65535 dimezza silenziosamente ogni valore. Un caso limite non presenta alcuna scala per impostazione predefinita: un’acquisizione da sorgente a 8 bit salvata come output a 8 bit viene troncata, non riscalata, e non riceve deliberatamente alcun tag di scala — riesportare a 16 bit o 32 bit invece di dividerla. Vedi [Formati immagine di output](../output-image-formats.md) per la descrizione completa.***

## Metadati trasferiti nelle esportazioni

Ogni prodotto conserva il **blocco GPS**dell’acquisizione sorgente e il suo**sotto-IFD EXIF**, quindi un’
esportazione include `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` e
`CameraSerialNumber`, oltre al georeferenziazione.

{% hint style="warning" %}
**Se un ortomosaico viene visualizzato con una scala assurda, controllare prima `FocalLength`.**
Pix4D calcola la distanza campionaria da terra (GSD) in base alla lunghezza focale e all’altitudine. Senza il tag,
si ricade su una scala completamente errata: in un volo misurato di 49 acquisizioni, un aranceto di 411 m × 160 m
è stato ricostruito come 47,8 km × 13 km, producendo un ortomosaico da 455 megapixel costituito prevalentemente da
spazio vuoto. Il tiling lento e un file inaspettatamente enorme sono sintomi di questo problema, non
problemi separati.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Non vengono copiati *tutti* i tag. I tag strutturali di IFD0 vengono deliberatamente tralasciati (copiarli
corrompe l’output di LATTICE), mentre `ExifImageWidth` / `ExifImageHeight` vengono esclusi
perché descrivono l’acquisizione originale — un’esportazione ridimensionata risulterebbe altrimenti
con dimensioni in contraddizione con il proprio raster.

***

## Revisione del registro di debug

### Verifica di avvisi o errori

1. Aprire la scheda **Registro di debug** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Scorrere i messaggi
3. Cercare avvisi gialli o errori rossi
4. Leggere le righe `[RUN-SUMMARY]` e eventuali suggerimenti
5. Contattare l’assistenza MAPIR per ricevere aiuto

### Salvataggio del log

Per conservare una traccia dell’elaborazione o per inviarla all’assistenza MAPIR:

1. Fare clic sul pulsante **&quot;Copia&quot;**o**&quot;Scarica&quot;**

2. Salvare come file di testo nella cartella del progetto
3. Allegarlo alla documentazione del progetto
4. Inviarlo all’assistenza MAPIR in caso di problemi riscontrati

***

## Problemi comuni relativi all’output e relative soluzioni

### Problema: file di output mancanti

**Possibili cause:**

* Il prodotto non è compatibile con quella telecamera (ad es. radianza/riflettanza per le telecamere RGB — come indicato nel log)
* Mancava un riferimento obbligatorio (ad es. riflettanza senza bersaglio e senza radiazione discendente `.daq`)
* La casella di controllo per l’esportazione del prodotto era disabilitata nelle Impostazioni del progetto
* Lo spazio su disco si è esaurito durante l’esportazione

**Soluzioni:**

1. Controllare i suggerimenti `[RUN-SUMMARY]` e le righe `[EXPORT-CHECK]` nel registro di debug: spiegano i salti per singola telecamera
2. Verificare le caselle di controllo dei prodotti di esportazione in [Impostazioni del progetto](adjusting-project-settings.md)
3. Verificare che lo spazio su disco fosse sufficiente
4. Eseguire nuovamente l’elaborazione dopo aver risolto la causa

### Problema: Bordi scuri o chiari (vignettatura ancora visibile)

**Possibili cause:**

* Correzione della vignettatura disabilitata
* Telecamera/obiettivo non presenti nel database dei profili Chloros
* Vignettatura estrema che supera la capacità di correzione

**Soluzioni:**

1. Verificare che la correzione della vignettatura sia stata abilitata in “Impostazioni progetto”
2. Verificare che il modello della telecamera sia stato rilevato correttamente
3. Contattare l’assistenza MAPIR se la vignettatura persiste

### Problema: Colori o valori errati

**Possibili cause:**

* Nessun target di calibrazione rilevato
* Modello di target di calibrazione selezionato errato
* Calibrazione della riflettanza disabilitata
* Immagini dei target di scarsa qualità

**Soluzioni:**

1. Verificare che la calibrazione della riflettanza sia stata abilitata
2. Controllare i messaggi &quot;Target trovato&quot; nel registro di debug
3. Verificare la qualità delle immagini dei target
4. Eseguire nuovamente l’elaborazione contrassegnando i target corretti

### Problema: i valori NDVI sembrano errati

**Intervalli NDVI previsti:*** **Acqua, rocce, suolo**: da -0,1 a 0,2
* **Vegetazione rada/malata**: da 0,2 a 0,4
* **Vegetazione moderata**: da 0,4 a 0,6
* **Vegetazione sana e fitta**: da 0,6 a 0,9**Se i valori esulano da questi intervalli:**

1. Verificare che sia stata applicata la calibrazione della riflettanza
2. Verificare che sia stato incluso il log del sensore di luce
3. Controllare che siano stati rilevati i target di calibrazione
4. Assicurarsi che sia stato rilevato il modello corretto di fotocamera
5. Verificare i tempi e le condizioni di acquisizione delle immagini dei target
6. Se si calcolano autonomamente gli indici dai file di riflettanza, confermare di aver diviso per il valore `Chloros:PixelScale` del file (vedi sopra)

***

## Utilizzo delle immagini elaborate

### Per la fotogrammetria / la creazione di ortomosaici

**Flusso di lavoro consigliato:**

1.**Importare le immagini di riflettanza calibrate** nel software di fotogrammetria:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Conservare i metadati EXIF**: assicurarsi che i dati GPS vengano mantenuti per la geolocalizzazione
3. **Flussi di lavoro calibrati**: utilizzare immagini di riflettanza per garantire l’accuratezza scientifica — la riflettanza LATTICE contiene i tag di calibrazione XMP che Pix4D legge
4. **Elaborare i mosaici indice**: Creare ortomosaici NDVI da singole immagini indice
5. **Esportare GeoTIFF georeferenziato**: Per l’utilizzo in applicazioni GIS

### Per l’analisi GIS

**Flusso di lavoro consigliato:**

1.**Caricare in QGIS, ArcGIS o simili**

2.**Utilizzare immagini di riflettanza a 16 bit TIFF** per l’analisi multibanda (dividere per il valore `Chloros:PixelScale` del file)
3. **Utilizzare le immagini indice** (NDVI, NDRE) come livelli di vegetazione pronti all’uso
4. **Calcolatore raster**: combinare le bande per analisi personalizzate
5. **Esportazione**: creare mappe di classificazione, rilevamento delle variazioni, mappe dello stato di salute della vegetazione

### Per analisi dirette / reportistica

**Flusso di lavoro consigliato:**

1.**Utilizzare immagini indice con colori LUT** per report visivi
2. **Estrazione delle statistiche**: media di NDVI per campo/parcella
3. **Serie temporali**: confronto degli indici tra più sessioni
4. **Generazione di report**: includere mappe, statistiche e visualizzazioni***

## Archiviazione e backup

### Strategia di backup consigliata

**Cosa salvare:*** ✅ **Immagini RAW/JPG originali o acquisizioni raw LATTICE** - Archiviare su un’unità separata o sul cloud; i file raw costituiscono la fonte della pipeline e tutto il resto può essere rigenerato a partire da essi
* ✅ **File dei sensori di luce `.daq` / `.csv`** - Necessari per ricavare nuovamente la riflettanza in un secondo momento
* ✅ **Risultati elaborati** - Conservare le immagini calibrate e gli indici
* ✅ **Cartella del progetto** (`project.json` e file correlati) - Contiene tutte le impostazioni per un&#x27;eventuale rielaborazione
* ✅ **Log di debug** - Documenta i dettagli dell’elaborazione
* ✅ **Immagini di riferimento per la calibrazione** - Per la verifica e la rielaborazione**Raccomandazioni per l’archiviazione:*** **Backup immediato**: disco rigido esterno
* **Archiviazione a lungo termine**: archiviazione su cloud (Google Drive, Dropbox, ecc.)
* **Dati critici**: conservare 2-3 copie in luoghi diversi***

## Prossime elaborazioni

### Riutilizzo delle impostazioni del progetto

Se in futuro si elaboreranno set di dati simili:

1. **Salvare il modello di progetto** (se non è già stato fatto)
2. **Creare un nuovo progetto** utilizzando il modello salvato
3. **Importare nuove immagini**

4.**Elaborare**con impostazioni identiche per garantire la coerenza

### Elaborazione in batch di più sessioni

Per più sessioni/set di dati:**Opzione 1: Interfaccia grafica (GUI) - Progetti multipli**

* Creare un progetto separato per ogni sessione
* Utilizzare impostazioni del modello coerenti
* Elaborare una alla volta

**Opzione 2: Chloros CLI (solo Chloros+)**

* Automatizzare l’elaborazione in batch
* Elaborare più cartelle con script
* Consultare la [Documentazione di CLI](../CLI.md) e il [Riferimento di CLI](../reference/cli-reference.md)

**Opzione 3: Python SDK (solo Chloros+)**

* Controllo programmatico
* Integrazione con le pipeline di analisi
* Vedi [Documentazione API](../api-python-sdk.md) e la [Riferenza SDK](../reference/sdk-reference.md)

***

## Risoluzione dei problemi nella post-elaborazione

### Rielaborazione con impostazioni diverse

Se i risultati non sono soddisfacenti:

1. Conservare le immagini originali (non cancellarle mai)
2. Aprire lo stesso progetto in Chloros
3. Modificare le impostazioni nel pannello “Impostazioni progetto”
4. Eseguire nuovamente l’elaborazione: i risultati vengono salvati nelle stesse cartelle di prodotto, quindi i file con lo stesso nome dell’esecuzione precedente vengono sostituiti

### Elaborazione di un sottoinsieme di immagini

Per rielaborare solo immagini specifiche:

1. Creare un nuovo progetto
2. Importare solo le immagini che necessitano di rielaborazione
3. Utilizzare lo stesso modello di impostazioni
4. Eseguire l’elaborazione su un set di dati più piccolo

### Come ottenere assistenza

In caso di problemi:

* 📧 **E-mail**: info@mapir.camera (allegare il log di debug)
* 🌐 **Assistenza**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Domande frequenti](../faq.md)
* 📖 **Documentazione**: [Manuale di Chloros](../)***

## Riepilogo: flusso di lavoro completo

Hai ora completato l’intero flusso di lavoro di elaborazione di Chloros:

1. ✅ **Progetto creato** - Vedi [Progetti](../projects.md)
2. ✅ **File aggiunti** - Vedi [Aggiunta di file](adding-files-to-a-project.md)
3. ✅ **Impostazioni modificate** - Vedi [Modifica delle impostazioni del progetto](adjusting-project-settings.md)
4. ✅ **Obiettivi contrassegnati** - Vedi [Scelta delle immagini di destinazione](choosing-target-images.md)
5. ✅ **Elaborazione avviata** - Vedi [Avvio dell’elaborazione](starting-the-processing.md)
6. ✅ **Monitoraggio dello stato di avanzamento** - Vedi [Monitoraggio dell’elaborazione](monitoring-the-processing.md)
7. ✅ **Risultati esaminati** - Questa pagina**Le tue immagini multispettrali calibrate e con correzione della riflettanza sono pronte per l’analisi!**

***

## Risorse aggiuntive

### Funzionalità avanzate

* [**Visualizzatore di immagini**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualizzazione e analisi interattive
* [**Indice/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) - Test personalizzati degli indici
* [**Formule degli indici multispettrali**](../project-settings/multispectral-index-formulas.md) - Riferimento completo sugli indici

### Automazione e integrazione

* [**Documentazione CLI**](../CLI.md) - Elaborazione batch da riga di comando
* [**Python SDK**](../api-python-sdk.md) - Automazione programmatica
* [**Funzionalità di Chloros+**](../#chloros) - Funzionalità di elaborazione avanzate

### Assistenza e formazione

* [**Domande frequenti**](../faq.md) - Risposte alle domande più frequenti
* [**Target di calibrazione**](../calibration-targets.md) - Informazioni sulla calibrazione della riflettanza
* [**Telecamere supportate**](../supported-cameras.md) - Hardware compatibile
