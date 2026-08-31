# Interfaccia grafica: Navigazione

Quando si avvia Chloros per la prima volta, viene avviato il backend di elaborazione. Una volta che il backend è pronto, viene visualizzata l’icona del menu principale in alto a sinistra <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> e le schede “Telecamere” e “Sensori di luce” si sbloccano nella barra laterale sinistra (fino a quel momento sono disattivate).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Da sinistra a destra, l’intestazione superiore contiene:

### Menu principale di <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>Dal menu principale è possibile:

* **Nuovo progetto**— creare un nuovo progetto. Se sono stati salvati dei modelli di progetto, appare un menu a tendina**Seleziona modello** che consente di avviare il nuovo progetto utilizzando le impostazioni di un modello.
* **Apri progetto**— aprire un progetto esistente. L’elenco include un pulsante**Apri cartella del progetto** che apre la cartella dei progetti nel tuo Esplora file.
* **Duplica progetto** — copia il progetto attualmente aperto con un nuovo nome (viene suggerito un nome libero come &quot;Il mio progetto (2)&quot;) e apri la copia. _(visibile dopo l’apertura di un progetto)_
* **Aggiungi file** — aggiunge singoli file immagine al progetto corrente _(visibile dopo l’apertura di un progetto)_
* **Aggiungi cartella** — aggiunge una o più cartelle di immagini al progetto corrente _(visibile dopo l’apertura di un progetto)_
* **Avvia elaborazione / Interrompi elaborazione** — avvia o interrompi la pipeline di elaborazione delle immagini _(abilitato dopo l’aggiunta dei file)_
* **Connetti alla telecamera** — passa alla [scheda Telecamere](lattice/) per collegare una telecamera o un array LATTICE. Funziona anche senza un progetto aperto.
* **Connettiti al sensore di luce** — passa alla [scheda Sensori di luce](daq/) per collegare un sensore di luce DAQ. Funziona anche senza un progetto aperto.

{% hint style="info" %}
** Solo perWindows**: l’interfaccia grafica desktop diChloros

è disponibile suWindows

. Gli utenti diLinux

devono consultare la documentazione [CLI

](CLI.md) e [Python

SDK

](api-python-sdk.md) per l’elaborazione headless.
{% endhint %}

### Pulsante &quot;Riproduci/Avvia&quot; di<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">



Quando è abilitato, il pulsante &quot;Avvia elaborazione&quot; avvia la pipeline di elaborazione delle immagini.

### Barra di avanzamento di<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

<img src=".gitbook/assets/image (5).png" alt="" data-size="line">



Nella modalità gratuitaChloros

, che elabora tutti i file in sequenza, la barra di avanzamento mostrerà 2 fasi: Rilevamento del bersaglio ed Elaborazione.

Nella modalità a pagamentoChloros

+ con licenza, che elabora tutti i file contemporaneamente, la barra di avanzamento mostra 4 fasi: Rilevamento, Analisi, Calibrazione, Esportazione. Se si posiziona il cursore del mouse sulla barra di avanzamentoChloros

+, si aprirà il pannello esteso con le 4 fasi della barra di avanzamento, in modo da poter seguire l’avanzamento. Cliccando sulla barra di avanzamento superiore si blocca il pannello a tendina; cliccandoci di nuovo lo si sblocca.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu laterale

Il menu della barra laterale sinistra contiene varie icone con cui interagire, in questo ordine dall’alto verso il basso:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Impostazioni del progetto](project-settings/project-settings.md)

La scheda “Impostazioni del progetto” consente di regolare le impostazioni globali del progetto e quelle relative all’elaborazione. Modificale prima di iniziare l’elaborazione dei file.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Esploratore file

Aggiungi file/cartelle e rimuovi file dal progetto. I file duplicati vengono ignorati. Seleziona la casella della colonna &quot;destinazione&quot; per qualsiasi immagine di destinazione: l’elaborazione prenderà in considerazione solo le immagini selezionate come destinazioni, velocizzando notevolmente i tempi di elaborazione. Utilizza il pulsante di alternanza “Immagine/Metadati” per passare dalla visualizzazione della griglia di miniature dell’immagine selezionata a una tabella dettagliata dei metadati.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visualizzatore di immagini](image-viewer-gui/opening-an-image-full-screen.md)

Quando si fa clic su un&#x27;immagine nel visualizzatore principale, questa viene aperta a schermo intero nella scheda &quot;Visualizzatore immagini&quot;.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Visualizzatore mappe](image-viewer-gui/map-markers.md)

Visualizza le tue immagini su una mappa 2D interattiva in base alle loro coordinate GPS. Supporta i provider di tessere Google Maps ed ESRI, selezionando automaticamente il servizio più adatto alla propria posizione. Passare il mouse sui marcatori per visualizzare le anteprime delle miniature delle immagini.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Telecamere](lattice/)

Collegare e controllare le telecamere LATTICE in tempo reale — una alla volta o come array multicamera sincronizzati. La scheda mostra riquadri di anteprima in tempo reale con sovrapposizioni e istogrammi, impostazioni per singola telecamera e per sistema, nonché le impostazioni di acquisizione che determinano quali telecamere e tipi di esportazione vengono utilizzati dalla funzione “Acquisisci tutto”. Disponibile una volta che il backend è pronto; consulta la [sezione LATTICE](lattice/) per la guida completa.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Sensori di luce](daq/)

Collega i sensori di luce DAQ — DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet) — e visualizza i loro grafici spettrali calibrati in tempo reale in W/m²/nm. Da qui è possibile registrare i file `.daq` nel progetto aperto, rinominare i sensori, selezionare i profili di correzione del cappuccio e aggiornare il firmware del DAQ-E. Disponibile una volta che il backend è pronto; consultare la [sezione DAQ](daq/) per la guida completa.

#### Log di debug di <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">

Controlla il log per i messaggi di debug quando si verificano dei problemi. Copia/scarica il log e invialo al [Supporto MAPIR](https://www.mapir.camera/community/contact) per ricevere assistenza.

#### [Accesso utente](chloros+-login.md) <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">

La barra laterale di accesso utente consente di accedere al proprio account Chloros+ per sbloccare funzionalità avanzate. È inoltre possibile visualizzare la versione corrente dell’applicazione, nonché impostare la lingua del testo visualizzato nell’interfaccia grafica di Chloros e in CLI.
