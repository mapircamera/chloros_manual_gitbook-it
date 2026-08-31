---
metaLinks: {}
---

# Guida introduttiva

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros

è un&#x27;applicazione software di [MAPIR

](https://www.mapir.camera) per l&#x27;elaborazione di immagini multispettrali, il controllo in tempo reale dell&#x27;hardwareMAPIR

e la registrazione dei dati dei sensori.Chloros

1.2.0 supporta l&#x27;intera famiglia di prodottiMAPIR

:

* **TelecamereSurvey3** — elaborano le acquisizioni RAW+JPG trasformandole in mappe calibrate di riflettanza e indici di vegetazione. Vedi [Telecamere supportate](supported-cameras.md).
* **Telecamere LATTICE** — collegano in tempo reale i moduli delle telecamere multispettrali GigE, singolarmente o come array sincronizzati di più telecamere: consentono di visualizzare in anteprima, acquisire ed elaborare i dati trasformandoli in prodotti calibrati di radianza e riflettanza. Consulta la [sezione dedicata a LATTICE](lattice/README.md).
* **Sensori di luce DAQ** — sensori spettrali DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet): spettri calibrati in tempo reale, registrazioni `.daq` e illuminazione discendente per l’elaborazione della riflettanza. Vedi la [sezione DAQ](daq/README.md).

{% hint style="success" %}
**Novità diChloros

1.2.0**: controllo in tempo reale di telecamere e array LATTICE, integrazione dei sensori di luce DAQ, modalità di acquisizione e registratori, una pipeline completa di elaborazione radiometrica LATTICE, automazione dei progetti tramiteCLI

/SDK

e molto altro ancora. Consulta l’elenco delle novità qui sotto e [Scarica](download.md) per il log delle modifiche.
{% endhint %}

{% hint style="info" %}
**UtilizziChloros

con un assistente AI?** Questo manuale è pensato proprio per questo. Indica al tuo assistente:

* `https://mapir.gitbook.io/chloros/llms.txt` — indice leggibile da macchina di ogni pagina.
* Qualsiasi pagina in formato Markdown grezzo — aggiungi `.md` al suoURL

(ad es. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* La [RiferenzaCLI

](reference/cli-reference.md) e la [RiferenzaSDK

](reference/sdk-reference.md) — pagine di riferimento complete e con valori esatti scritte per l’utilizzo da parte dei modelli di linguaggio di grandi dimensioni (LLM).

Esempio di prompt: *&quot;Leggi https://mapir.gitbook.io/chloros/reference/cli-reference.md,, quindi scrivi uno script che effettui l&#x27;accesso ed elabori la cartella ~/flights/flight_001 in file GeoTIFF con valori di riflettanza +NDVI

.&quot;*

Guida completa: [Utilizzo diChloros

con gli assistenti AI](ai-assistants.md).
{% endhint %}

***

## Novità diChloros

1.2.0

* **Controllo in tempo reale delle telecamere — nuova scheda &quot;Telecamere&quot;.** Collega le telecamere LATTICE una alla volta o come array multicamera sincronizzati (sincronizzazione temporale PTP, acquisizione attivata dall’hardware), con sovrapposizioni di anteprima in tempo reale, istogrammi per banda, esposizione automatica intelligente, calcolatore di indice in tempo reale e aggiornamenti del firmware delle telecamere direttamente dall’app.
* **Sensori di luce — nuova scheda “Sensori di luce”.** Collega i sensori DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet); visualizza spettri calibrati in tempo reale (W/m²/nm), registra file `.daq` nel tuo progetto, seleziona profili di correzione cap e aggiorna il firmware DAQ-E tramite rete.
* **Modalità di acquisizione e registratori.** Acquisizione singola / continua / a intervalli, oltre a una modalità «Acquisizione più veloce» solo in formato raw; selezione per progetto delle telecamere e dei tipi di esportazione generati da «Acquisisci tutto»; registratori array per video indicizzati di livello di monitoraggio e raffiche raw di livello analitico con creazione di video offline.
* **Pipeline di elaborazione LATTICE.** Importa le cartelle di acquisizione LATTICE e scomponi ogni fotogramma grezzo in prodotti debayered, di anteprima, radianza float32 (W/m²/sr/nm) e riflettanza con opzioni attivabili per singolo prodotto. La riflettanza può provenire da un bersaglio di calibrazione all’interno del fotogramma o da un segnale in discesa DAQ; l’allineamento dell’array viene applicato alle esportazioni; la calibrazione di fabbrica mancante viene scaricata automaticamente in base al numero di serie della telecamera.
* **I progetti memorizzano le configurazioni hardware.** Le telecamere e i sensori di luce collegati vengono salvati con il progetto (`cameras.json` / `sensors.json`) e si ricollegano con le impostazioni salvate quando si riapre il progetto. Vedi [GUI: Progetti](projects.md).
* **Aggiornamenti del visualizzatore di immagini.** Lettura dei pixel/indici del cursore con corretto ridimensionamento della riflettanza per ogni file, istogrammi dei livelli, cursore di binning GSD, modalità di griglia “Per Trigger” / “Per Camera”, visualizzazioni dei prodotti LATTICE ed esportazioni su disco delle sandbox di indici/LUT.
* **Funzionalità &quot;CLI

&quot; e &quot;SDK

&quot;, notevolmente ampliate.** Nuove famiglie di comandi `lattice`, `daq pool-*`, `project` e `time-sync`; nuove opzioni `process` (`--input-level`, opzioni di attivazione/disattivazione per singolo prodotto, `--reflectance-source`, flag di allineamento degli array);SDK

handle smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) che avviano automaticamente il backend; automazione `open_project()`; il wheelSDK

è incluso nei programmi di installazione e pubblicato su PyPI come `chloros-sdk`.
* **Semantica di errore trasparente.** Un&#x27;esecuzione di `chloros-cli process` che ha richiesto prodotti ma non ne ha scritto alcuno ora fallisce in modo evidente e termina con un codice di uscita diverso da zero; le esecuzioni riuscite riportano il numero di prodotti immagine scritti.
* **Nuovo layout di output.** I prodotti vengono salvati in cartelle `<project>/<camera>/<format>/<Product>_Images/` e mantengono il nome del file sorgente: è la cartella, non un suffisso del nome del file, a identificare il prodotto. Vedi [Formati delle immagini di output](output-image-formats.md).
* **Più input, piani e lingue.** Supporto per input `.dng`; tutte le 38 lingue dell’interfaccia sono ora completamente disponibili; limiti hardware per piano con utilizzo gratuito (senza login) fino a 4 telecamere e 2 sensori di luce.
* **Affidabilità.** La funzione “Interrompi elaborazione” termina in modo corretto con un riepilogo dettagliato dell’esecuzione; i progetti multicamera esportano i dati di ogni singola telecamera e gli aggiornamenti del programma di installazione non causano più la disconnessione dell’utente.***

Chloros

è disponibile in 3 interfacce applicative:

##Chloros

: applicazione GUI desktop

Finestra autonoma separata con tutte le funzionalità, comprese le schede “Telecamere in diretta” e “Sensori di luce”. _Solo per Windows._

## [Chloros

CLI

: interfaccia a riga di comando](CLI.md)

Elaborazione batch da riga di comando con comandi in tempo reale `lattice`, `daq pool-*`, `project` e `time-sync`. Ideale per l’automazione, la creazione di script e il funzionamento senza interfaccia grafica. Disponibile su **Windows

,Linux

amd64 eLinux

arm64 (NVIDIA Jetson)**. _Per accedere alla CLI è necessario un piano a pagamentoChloros

+._

## [Chloros

API

:Python

SDK

](api-python-sdk.md)

Interfaccia programmaticaPython

per l&#x27;automazione e i flussi di lavoro personalizzati: elaborazione completa della pipeline, sessioni live con telecamere/array, sessioni con sensori DAQ e automazione dei progetti salvati. Installata con il pacchetto desktop/CLI

e pubblicata anche come `pip install chloros-sdk`. _L&#x27;API richiede un abbonamento a pagamentoChloros

+ per l&#x27;accesso._

***

## Piattaforme supportate

| Piattaforma | GUI |CLI

|Python

SDK

|
| --- | --- | --- | --- |
| **Windows

10/11 (x64)** | Sì | Sì | Sì |
| **Linux

amd64 (x86_64)** | No | Sì | Sì |
| **Linux

arm64 (NVIDIA Jetson)** | No | Sì | Sì |

Per le istruzioni di installazione suLinux

, consultare la sezione [Linux

e Edge Computing](linux/linux-overview.md).

***

## Guida introduttiva in tre passaggi

1. **Installazione** — scarica ed esegui il programma di installazione per la tua piattaforma. Vedi [Download](download.md).
2. **Accedi (facoltativo per l’interfaccia grafica)** — l’interfaccia grafica elabora le immagini gratuitamente senza bisogno di un account. Un [Chloros

+ login](chloros+-login.md) sblocca l’elaborazione parallela, l’accelerazione GPU, limiti più elevati per i dispositivi e l’accesso aCLI

/SDK

.
3. **Crea il tuo primo progetto** — apriChloros

, crea un [Nuovo progetto](projects.md), [aggiungi le tue immagini](processing-images-gui/adding-files-to-a-project.md) e [avvia l’elaborazione](processing-images-gui/starting-the-processing.md). Per controllare invece hardware in tempo reale, apri la scheda “Telecamere” o “Sensori di luce” — vedi [GUI: Navigazione](navigation.md).

***

##Chloros

+

SebbeneChloros

sia gratuito per la maggior parte delle attività, potresti scoprire di aver bisogno di qualcosa in più. È qui che una licenza a pagamento perChloros

+ può tornarti utile. Con una licenzaChloros

+ puoi sbloccare nuove funzionalità quali:

* **Elaborazione multithread**: accelera notevolmente l’elaborazione delle immagini per progetti di grandi dimensioni elaborando simultaneamente le immagini attraverso la pipeline.
* **Accelerazione GPU (CUDA)**: sfrutta le attuali opzioni di memoria GPU più capienti per velocizzare ulteriormente la pipeline di elaborazione delle immagini. Per ottenere i migliori risultati, consigliamo 4 GB o più di VRAM.
* **Accesso aChloros

+**[**CLI**](CLI.md): eseguiChloros

+ dalla riga di comando per automatizzare e integrare il software nel tuo. Disponibile su qualsiasi piano a pagamento; applicato lato server.
* **Chloros

+**[**API**](api-python-sdk.md) **Accesso:** eseguireChloros

+ daPython

per il controllo programmatico, consentendo una perfetta integrazione con le vostre pipeline di ricerca, i flussi di lavoro di analisi dei dati e le applicazioni personalizzate. Disponibile su qualsiasi piano a pagamento; applicato lato server.
* **Limiti hardware più elevati**: collega più telecamere e sensori di luce contemporaneamente. Senza effettuare il login, l’interfaccia grafica (GUI) consente di collegare fino a 4 telecamere e 2 sensori di luce DAQ; i piani a pagamento aumentano entrambi i limiti:

| Piano | Telecamere | Sensori di luce DAQ |
| --- | --- | --- |
| Iron (gratuito, senza accesso) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Utilizzo su più dispositivi**: ogni licenzaChloros

+ consente di registrare 2 o più dispositivi. Utilizza il tuo account CloudMAPIR

per gestire i dispositivi registrati. Aggiungi il supporto per ulteriori dispositivi effettuando l’aggiornamento della tua licenzaChloros

+.
* **Metodo avanzato di debayering sensibile alla trama:** un debayering di alta qualità sensibile ai bordi combinato con un modello di denoising basato su AI/ML che rimuove quasi tutto il rumore del debayering.
* **Formule personalizzate per indici multispettrali:** inserisci indici multispettrali personalizzati nei calcolatori raster diChloros

, sia per l’elaborazione che per l’area di prova di visualizzazione delle immagini.
* **Linux

e Edge Computing:** eseguiChloros

sulle piattaformeLinux

x86_64 e ARM64, tra cui NVIDIA Jetson, per l’elaborazione sul campo e edge. Vedi [Panoramica diLinux

](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Prezzi e registrazione</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
