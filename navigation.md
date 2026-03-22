# Interfaccia grafica: Navigazione

Al primo avvio di Chloros e Chloros (Browser), verrà avviato il backend. Una volta pronto, apparirà l&#x27;icona del menu principale in alto a sinistra <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Da sinistra a destra, l&#x27;intestazione superiore contiene:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu principale

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Dal menu principale è possibile:

* **Nuovo progetto** — creare un nuovo progetto
* **Apri progetto** — aprire un progetto esistente
* **Apri cartella del progetto** — aprire la cartella del progetto in Esplora file
* **Aggiungi file** — aggiungere singoli file immagine al progetto corrente _(visibile dopo l&#x27;apertura di un progetto)_
* **Aggiungi cartella** — aggiungere una cartella di immagini al progetto corrente _(visibile dopo l&#x27;apertura di un progetto)_
* **Avvia elaborazione / Interrompi elaborazione** — avviare o interrompere la pipeline di elaborazione delle immagini _(abilitata dopo l&#x27;aggiunta dei file)_

{% hint style="info" %}
**Solo Windows**: L&#x27;interfaccia grafica desktop Chloros è disponibile su Windows. Gli utenti di Linux dovrebbero vedere [CLI](CLI.md) e [Python SDK](api-python-sdk.md) per l&#x27;elaborazione headless.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Pulsante Riproduci/Avvia

Quando è abilitato, il pulsante di avvio dell&#x27;elaborazione avvia la pipeline di elaborazione delle immagini.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Barra di avanzamento <img src=".gitbook/assets/image (5).png" alt="" data-size="line">Nella modalità gratuita Chloros, che elabora tutti i file in sequenza, la barra di avanzamento mostrerà 2 fasi: Rilevamento del bersaglio ed Elaborazione.

Nella modalità a pagamento con licenza Chloros+, che elabora tutti i file contemporaneamente, la barra di avanzamento mostra 4 fasi: Rilevamento, Analisi, Calibrazione, Esportazione. Se si posiziona il cursore del mouse sulla barra di avanzamento Chloros+, si aprirà il pannello esteso con le 4 fasi della barra di avanzamento, in modo da poter seguire il processo. Cliccando sulla barra di avanzamento in alto si bloccherà il pannello a tendina, cliccando di nuovo lo si sbloccherà.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu laterale

Il menu della barra laterale sinistra contiene varie icone con cui interagire:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Impostazioni progetto](project-settings/project-settings.md)

La scheda Impostazioni progetto consente di regolare le impostazioni globali e di elaborazione del progetto. Regolarle prima di iniziare a elaborare i file.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Browser file

Aggiungi file/cartelle e rimuovi file dal progetto. I file duplicati vengono ignorati. Seleziona la casella della colonna di destinazione per qualsiasi immagine di destinazione e l&#x27;elaborazione prenderà in considerazione solo le immagini selezionate come destinazioni, velocizzando notevolmente i tempi di elaborazione. Usa il pulsante di alternanza Immagine/Metadati per passare dalla visualizzazione della griglia di miniature dell&#x27;immagine selezionata a una tabella dettagliata dei metadati.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visualizzatore immagini](image-viewer-gui/opening-an-image-full-screen.md)

Quando si fa clic su un&#x27;immagine nel visualizzatore immagini principale, questa viene aperta a schermo intero nella scheda Visualizzatore immagini.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Mappa](image-viewer-gui/map-markers.md)

Visualizza le tue immagini su una mappa 2D interattiva in base alle loro coordinate GPS. Supporta i provider di mappe Google Maps ed ESRI, selezionando automaticamente il servizio migliore per la tua posizione. Passa il mouse sui marcatori per vedere le anteprime delle miniature delle immagini.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Registro di debug

Controlla il registro per i messaggi di debug quando si verificano dei problemi. Copia/scarica il registro e invialo all&#x27;[Assistenza MAPIR](https://www.mapir.camera/community/contact) per ricevere assistenza.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Accesso utente](chloros+-login.md)

La barra laterale di accesso utente consente di accedere al proprio account Chloros+ per sbloccare funzionalità avanzate. È inoltre possibile visualizzare la versione corrente dell&#x27;applicazione, nonché impostare la lingua del testo visualizzato nell&#x27;interfaccia grafica Chloros e in CLI.
