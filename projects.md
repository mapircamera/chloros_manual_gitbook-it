# Interfaccia grafica: Progetti

Chloros consente di creare progetti che potranno essere riaperti in futuro. Un progetto è una semplice cartella (all’interno della cartella “Project Folder”) contenente:

* `project.json` — impostazioni del progetto, elenco dei file e preferenze di visualizzazione
* `cameras.json` — telecamere e array collegati mentre il progetto era aperto, con le relative impostazioni
* `sensors.json` — sensori di luce DAQ collegati mentre il progetto era aperto, oltre alle associazioni telecamera↔sensore
* le tue acquisizioni, le registrazioni `.daq` e le cartelle di output elaborate

Non esiste un formato proprietario per i file di progetto: la cartella e i suoi file JSON costituiscono il progetto, il che rende anche facile copiare, archiviare e gestire i progetti dalla [CLI](CLI.md) o dal [Python SDK](api-python-sdk.md).

A meno che non venga modificata nelle [Impostazioni del progetto](project-settings/project-settings.md), la cartella del progetto è `Documents\MAPIR\Chloros Projects` nella cartella Documenti (`~/Documents/MAPIR/Chloros Projects` su Linux).

{% hint style="warning" %}
**Windows: mantenere brevi le cartelle di progetto a meno che non siano abilitati i percorsi lunghi.**Windows viene fornito con i percorsi lunghi disabilitati; pertanto, nessun percorso di file può superare i 260 caratteri. I prodotti elaborati occupano fino a circa 130 caratteri al di sotto della cartella del progetto, quindi a partire dalla versione 1.3.0 Chloros rifiuta una cartella del progetto troppo profonda — al momento della creazione, dell’apertura o dell’avvio dell’elaborazione — invece di eseguire il programma senza scrivere nulla. Il messaggio indica la cartella del progetto più lunga che risulta compatibile. Utilizzare una cartella di progetto più breve oppure attivare i percorsi lunghi di Windows (l’opzione**Abilita percorsi lunghi di Windows** del programma di installazione, selezionata per impostazione predefinita) e riavviare Chloros. Vedere [Avvio dell’elaborazione](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Nuovo progetto

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Seleziona &quot;Nuovo progetto&quot; dal menu principale e inserisci un nome univoco per il tuo progetto.

Se hai salvato dei modelli di progetto, sotto il campo del nome apparirà un menu a tendina **Seleziona modello**: scegliendone uno, il nuovo progetto verrà avviato con le impostazioni di quel modello. I modelli vengono salvati da [Impostazioni progetto](project-settings/project-settings.md): inserisci un nome nel campo &quot;Salva nome modello di progetto&quot; e fai clic sull&#x27;icona di salvataggio.

## Apri progetto

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>La sezione &quot;Apri progetto&quot; elenca tutti i progetti presenti nella cartella dei progetti, con l’opzione <strong>Apri cartella dei progetti</strong> in fondo alla pagina</p></figcaption></figure>

Selezionare “Apri progetto” per visualizzare un elenco dei progetti esistenti nella cartella dei progetti. Se non sono presenti progetti, il menu laterale secondario non si aprirà. Nella foto sopra sono riportati alcuni progetti creati tramite l’interfaccia grafica (t1, t2, t3). I progetti DATE\_TIME sono stati creati dall’CLI utilizzando lo schema di denominazione predefinito. Facendo clic su qualsiasi nome di progetto, questo verrà aperto.

Facendo clic sul pulsante “Apri cartella del progetto” si aprirà Esplora file del computer al percorso del progetto. È possibile modificare il percorso del progetto nelle [Impostazioni del progetto](project-settings/project-settings.md).

Se uno qualsiasi dei file immagine sorgente del progetto è stato spostato o eliminato dall’ultima volta che è stato aperto, Chloros mostra una finestra di dialogo che elenca esattamente quali file mancano, invece di aprire una griglia vuota.

## Duplica progetto

Opzione disponibile una volta aperto il progetto. Selezionare &quot;Duplica progetto&quot; per copiare il progetto corrente con un nuovo nome — Chloros suggerisce il prossimo nome disponibile (ad es. &quot;Il mio progetto (2)&quot;) — e il duplicato viene aperto immediatamente.

## Aggiungi file

Dopo aver aperto un progetto, seleziona “Aggiungi file” dal menu principale per aggiungere singoli file immagine al progetto corrente. Questa funzione rispecchia quella di aggiunta del browser dei file, ma è accessibile direttamente dal menu principale per maggiore comodità.

## Aggiungi cartella

Una volta aperto un progetto, seleziona &quot;Aggiungi cartella&quot; dal menu principale per aggiungere cartelle di immagini al progetto corrente. È possibile selezionare più cartelle in un&#x27;unica operazione. I file duplicati vengono ignorati.

## Avvia / Interrompi elaborazione

Dopo aver aggiunto i file a un progetto, nel menu principale diventa disponibile l’opzione “Avvia elaborazione”. Si tratta della stessa azione che si ottiene facendo clic sul pulsante Riproduci/Avvia nell’intestazione superiore. Durante l’elaborazione, la voce di menu cambia in “Interrompi elaborazione” per consentire di arrestare la pipeline.

## Connetti alla telecamera / Connetti al sensore di luce

Nella parte inferiore del menu principale sono presenti due scorciatoie hardware, disponibili sia con che senza un progetto aperto:

* **Connetti alla telecamera** — apre la [scheda Telecamere](lattice/) per collegare una telecamera o un array LATTICE.
* **Connetti al sensore di luce** — apre la [scheda Sensori di luce](daq/) per collegare un sensore di luce DAQ.

Il collegamento dell’hardware mentre un progetto è aperto lo salva all’interno del progetto (vedi sotto). In assenza di un progetto, i collegamenti sono validi solo per la sessione corrente.

{% hint style="info" %}
Le voci di menu **Aggiungi file**,**Aggiungi cartella**e**Avvia/Interrompi elaborazione**sono visibili o abilitate solo quando è aperto un progetto e sono stati aggiunti dei file. Consentono un accesso rapido alle azioni disponibili anche tramite la barra laterale del**File Browser** e i pulsanti dell’intestazione.
{% endhint %}

## I progetti memorizzano l’hardware

Novità nella versione 1.2.0: un progetto conserva l’hardware collegato finché rimane aperto. Le telecamere e gli array (con le relative impostazioni per ciascuna telecamera, i nomi, i colori e la disposizione a griglia) vengono salvati in `cameras.json`, mentre i sensori di luce (con nomi, colori e associazioni alle telecamere) in `sensors.json` — automaticamente, mentre lavori.

Quando **riapri** un progetto, Chloros non interagisce immediatamente con l’hardware. Ciascuna sezione si ricollega la prima volta che accedi alla scheda a cui appartiene:

* Aprendo la scheda **Telecamere**, le telecamere e gli array salvati vengono ricollegati e le loro impostazioni salvate vengono riapplicate.
* L’apertura della scheda **Sensori di luce**ricollega i sensori DAQ salvati e riapplica le loro impostazioni salvate, copri-sensore incluso. Un sensore impostato su**Nessuno (sensore nudo)** nel menu Cappuccio rimane nudo. Un progetto salvato da una versione precedente di Chloros ha registrato un sensore nudo come opzione predefinita, quindi viene ripristinato con la correzione del coseno solare; impostalo nuovamente su Nessuno e salva il progetto.

In questo modo, l’apertura di un progetto solo per sfogliare o esportare immagini non attiva mai lo streaming delle telecamere. Se un dispositivo salvato non viene trovato all’apertura della relativa scheda, una finestra di dialogo indica quali dispositivi non sono disponibili, in modo da poterli ricollegare o rimuovere.

## Registrazioni DAQ e file .daq in un progetto

* Le registrazioni `.daq` effettuate mentre il progetto è aperto (dalla scheda Sensori di luce o durante le acquisizioni) vengono **aggiunte automaticamente al progetto**.
* I file `.daq` importati e tutte le registrazioni del progetto sono elencati nella sezione **Sensore di luce DAQ** delle [Impostazioni del progetto](project-settings/project-settings.md), ciascuno con il proprio profilo di correzione del cap.
* Durante l’elaborazione, i file `.daq` del progetto forniscono l’illuminazione discendente per i prodotti di riflettanza — vedere [Formati immagine di output](output-image-formats.md).

## Esecuzione di un progetto salvato in modalità headless

Un progetto salvato può essere eseguito senza l’interfaccia grafica:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` opera sul percorso della cartella del progetto — si veda la [Guida di riferimento CLI](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` restituisce un handle del progetto; `connect_all()` attiva tutte le telecamere, gli array e i sensori di luce salvati con le relative impostazioni (i sensori di luce tramite un backend Chloros in esecuzione) — consultare la [Guida di riferimento SDK](reference/sdk-reference.md).
