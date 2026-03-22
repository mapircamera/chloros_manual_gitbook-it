# Completamento dell&#x27;elaborazione

Una volta che Chloros ha completato l&#x27;elaborazione, è il momento di esaminare i risultati, verificare la qualità dell&#x27;output e preparare le immagini elaborate per l&#x27;utilizzo nel flusso di lavoro. Questa pagina ti guida attraverso i passaggi finali e le azioni successive.

## Indicazione di elaborazione completata

Quando l&#x27;elaborazione si conclude con successo, vedrai diversi indicatori:

* ✅ **Barra di avanzamento**: raggiunge il 100% di completamento
* ✅ **Log di debug**: mostra il messaggio &quot;Elaborazione completata&quot;
* ✅ **Pulsante Start**: torna abilitato (pronto per la prossima esecuzione dell&#x27;elaborazione)
* ✅ **File di output**: tutte le immagini elaborate vengono salvate nella sottocartella del modello di fotocamera***

## Individuazione delle immagini elaborate

### Apertura della cartella di output

1. Fare clic sull&#x27;**icona del menu principale** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (in alto a sinistra)
2. Selezionare **&quot;Apri cartella progetto&quot;**

3. Si aprirà Esplora file nella directory del progetto
4. Individuare il progetto in base al nome

***

## Visualizzazione delle immagini elaborate

### Anteprima rapida in Esplora file

**Anteprima integrata in Windows:**

1. Accedere alla sottocartella del modello di fotocamera
2. Selezionare un file immagine
3. L&#x27;anteprima appare nel pannello di anteprima di Windows Explorer
4. Utilizzare i tasti freccia per sfogliare le immagini

### Anteprima in visualizzatori di immagini esterni

**Visualizzatori consigliati:*** **QGIS** - Software GIS gratuito (ideale per l&#x27;analisi multispettrale georeferenziata)
* **IrfanView** - Visualizzatore di immagini veloce e leggero (supporta TIFF)
* **Adobe Photoshop** - Editing professionale (supporto TIFF)
* **GIMP** - Alternativa gratuita a Photoshop
* **Windows Photos** - Visualizzazione di base (potrebbe non supportare TIFF a 16 bit)

### Anteprima nel visualizzatore di immagini Chloros

Utilizza il visualizzatore di immagini integrato in Chloros per una visualizzazione avanzata:

1. Clicca su una miniatura dell&#x27;immagine nel File Browser
2. L&#x27;immagine si apre nell&#x27;area di anteprima principale
3. Clicca sulla scheda **Visualizzatore di immagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> nella barra laterale sinistra
4. Utilizza [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) per un&#x27;analisi interattiva

Consulta [Visualizzatore di immagini](../image-viewer-gui/opening-an-image-full-screen.md) per istruzioni dettagliate.

***

## Controllo del registro di debug

### Verifica la presenza di avvisi o errori

1. Apri la scheda **Registro di debug** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> 2. Scorrere i messaggi
3. Cercare avvisi gialli o errori rossi
4. Esaminare eventuali problemi rilevati
5. Contattare l&#x27;assistenza MAPIR per ricevere supporto

### Salvataggio del log

Per conservare una registrazione dell&#x27;elaborazione o per inviarla all&#x27;assistenza MAPIR:

1. Fare clic sul pulsante **&quot;Copia&quot;**o**&quot;Scarica&quot;**

2. Salvare come file di testo nella cartella del progetto
3. Includere nella documentazione del progetto
4. Inviare all&#x27;assistenza MAPIR se si riscontrano problemi

***

## Problemi comuni di output e soluzioni

### Problema: file di output mancanti

**Possibili cause:**

* I file non soddisfacevano i criteri di elaborazione
* Immagini solo di destinazione (escluse dall&#x27;esportazione)
* Spazio su disco esaurito durante l&#x27;esportazione
* File danneggiato durante l&#x27;elaborazione

**Soluzioni:**

1. Controllare il registro di debug per messaggi di salto/errore
2. Verificare che lo spazio su disco fosse sufficiente
3. Contare i file: il numero dovrebbe corrispondere a (numero originale - numero di destinazione) × (indici + 1)
4. Reimportare ed elaborare nuovamente eventuali file mancanti

### Problema: Bordi scuri o chiari (vignettatura ancora visibile)

**Possibili cause:**

* Correzione della vignettatura disabilitata
* Fotocamera/obiettivo non presenti nel database dei profili Chloros
* Vignettatura estrema oltre la capacità di correzione

**Soluzioni:**

1. Verificare che la correzione della vignettatura sia stata abilitata nelle Impostazioni del progetto
2. Verificare che il modello della fotocamera sia stato rilevato correttamente
3. Contattare l&#x27;assistenza MAPIR se la vignettatura persiste

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
4. Elaborare nuovamente con i target corretti contrassegnati

### Problema: I valori NDVI sembrano errati

**Intervalli NDVI previsti:*** **Acqua, rocce, suolo**: da -0,1 a 0,2
* **Vegetazione rada/non sana**: da 0,2 a 0,4
* **Vegetazione moderata**: da 0,4 a 0,6
* **Vegetazione sana e fitta**: da 0,6 a 0,9**Se i valori sono al di fuori di questi intervalli:**

1. Verificare che sia stata applicata la calibrazione della riflettanza
2. Verificare che sia stato incluso il log del sensore di luce
3. Controllare che siano stati rilevati i target di calibrazione
4. Assicurarsi che sia stato rilevato il modello corretto di fotocamera
5. Rivedere i tempi e le condizioni di acquisizione delle immagini dei target

***

## Utilizzo delle immagini elaborate

### Per la fotogrammetria / creazione di ortomosaici

**Flusso di lavoro consigliato:**

1.**Importare le immagini di riflettanza calibrate** nel software di fotogrammetria:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Mantenere i metadati EXIF**: assicurarsi che i dati GPS siano conservati per il geotagging
3. **Flussi di lavoro calibrati**: utilizzare immagini di riflettanza per garantire l&#x27;accuratezza scientifica
4. **Elaborare i mosaici indice**: Creare ortomosaici NDVI da singole immagini indice
5. **Esportare GeoTIFF georeferenziato**: Per l&#x27;uso in applicazioni GIS

### Per l&#x27;analisi GIS

**Flusso di lavoro consigliato:**

1.**Caricare in QGIS, ArcGIS o simili**

2.**Utilizzare immagini di riflettanza a 16 bit TIFF** per l&#x27;analisi multibanda
3. **Utilizzare immagini indice** (NDVI, NDRE) come livelli di vegetazione pronti all&#x27;uso
4. **Calcolatore raster**: combinare le bande per analisi personalizzate
5. **Esportazione**: creare mappe di classificazione, rilevamento delle modifiche, mappe dello stato di salute della vegetazione

### Per analisi diretta / reportistica

**Flusso di lavoro consigliato:**

1.**Utilizzare immagini indice con colori LUT** per report visivi
2. **Estrarre statistiche**: media NDVI per campo/appezzamento
3. **Serie temporali**: confronta gli indici tra più sessioni
4. **Genera report**: includi mappe, statistiche e visualizzazioni***

## Archiviazione e backup

### Strategia di backup consigliata

**Cosa salvare:*** ✅ **Immagini RAW/JPG originali** - Archivia su un&#x27;unità separata/cloud
* ✅ **Risultati elaborati** - Conservare immagini calibrate e indici
* ✅ **File di progetto** - Contiene tutte le impostazioni per la rielaborazione, se necessario
* ✅ **Log di debug** - Documenta i dettagli dell&#x27;elaborazione
* ✅ **Immagini di riferimento per la calibrazione** - Per la verifica e la rielaborazione**Raccomandazioni per l&#x27;archiviazione:*** **Backup immediato**: disco rigido esterno
* **Archiviazione a lungo termine**: cloud (Google Drive, Dropbox, ecc.)
* **Dati critici**: conservare 2-3 copie in posizioni diverse***

## Prossime elaborazioni

### Riutilizzo delle impostazioni di progetto

Se in futuro si elaboreranno set di dati simili:

1. **Salvare il modello di progetto** (se non è già stato fatto)
2. **Creare un nuovo progetto** utilizzando il modello salvato
3. **Importare nuove immagini**

4.**Elaborare**con impostazioni identiche per garantire la coerenza

### Elaborazione in batch di più sessioni

Per più sessioni/set di dati:**Opzione 1: GUI - Progetti multipli**

* Creare un progetto separato per ogni sessione
* Utilizzare impostazioni del modello coerenti
* Elaborare una alla volta

**Opzione 2: Chloros CLI (solo Chloros+)**

* Automatizzare l&#x27;elaborazione in batch
* Elaborare più cartelle con script
* Vedi [Documentazione di CLI](../CLI.md)

**Opzione 3: Python SDK (solo Chloros+)**

* Controllo programmatico
* Integrazione con pipeline di analisi
* Vedi [Documentazione API](../api-python-sdk.md)

***

## Risoluzione dei problemi nella post-elaborazione

### Rielaborazione con impostazioni diverse

Se i risultati non sono soddisfacenti:

1. Conserva le immagini originali (non cancellarle mai)
2. Aprire lo stesso progetto in Chloros
3. Modificare le impostazioni nel pannello Impostazioni progetto
4. Eseguire nuovamente l&#x27;elaborazione: i risultati sovrascriveranno quelli precedenti

### Elaborazione di un sottoinsieme di immagini

Per rielaborare solo immagini specifiche:

1. Creare un nuovo progetto
2. Importare solo le immagini che necessitano di rielaborazione
3. Utilizzare lo stesso modello di impostazioni
4. Elaborare un set di dati più piccolo

### Richiesta di assistenza

In caso di problemi:

* 📧 **E-mail**: info@mapir.camera (includere il log di debug)
* 🌐 **Assistenza**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Domande frequenti](../faq.md)
* 📖 **Documentazione**: [Manuale Chloros](../)***

## Riepilogo: flusso di lavoro completo

Hai ora completato l&#x27;intero flusso di lavoro di elaborazione Chloros:

1. ✅ **Progetto creato** - Vedi [Progetti](../projects.md)
2. ✅ **File aggiunti** - Vedi [Aggiunta di file](adding-files-to-a-project.md)
3. ✅ **Impostazioni regolate** - Vedi [Regolazione delle impostazioni del progetto](adjusting-project-settings.md)
4. ✅ **Obiettivi contrassegnati** - Vedi [Scelta delle immagini target](choosing-target-images.md)
5. ✅ **Elaborazione avviata** - Vedi [Avvio dell&#x27;elaborazione](starting-the-processing.md)
6. ✅ **Monitoraggio dello stato di avanzamento** - Vedi [Monitoraggio dell&#x27;elaborazione](monitoring-the-processing.md)
7. ✅ **Risultati esaminati** - Questa pagina**Le tue immagini multispettrali calibrate e con correzione della riflettanza sono pronte per l&#x27;analisi!**

***

## Risorse aggiuntive

### Funzionalità avanzate

* [**Visualizzatore di immagini**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualizzazione e analisi interattive
* [**Sandbox indici/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Test di indici personalizzati
* [**Formule degli indici multispettrali**](../project-settings/multispectral-index-formulas.md) - Riferimento completo sugli indici

### Automazione e integrazione

* [**Documentazione CLI**](../CLI.md) - Elaborazione batch da riga di comando
* [**Python SDK**](../api-python-sdk.md) - Automazione programmatica
* [**Funzionalità di Chloros+**](../#chloros) - Funzionalità di elaborazione avanzate

### Assistenza e formazione

* [**Domande frequenti**](../faq.md) - Risposte alle domande più comuni
* [**Target di calibrazione**](../calibration-targets.md) - Informazioni sulla calibrazione della riflettanza
* [**Telecamere supportate**](../supported-cameras.md) - Hardware compatibile
