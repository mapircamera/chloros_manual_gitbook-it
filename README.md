---
metaLinks: {}
---

# Introduzione

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros è un&#x27;applicazione software di [MAPIR](https://www.mapir.camera) per l&#x27;elaborazione di immagini e altri dati dei sensori.

***{% hint style="success" %}**Novità di Chloros 1.1.0**: Supporto nativo per Linux (amd64 e arm64), edge computing NVIDIA Jetson, Dynamic Compute Adaptation, pipeline di elaborazione a 4 thread, nuovi comandi e opzioni CLI. Vedi [Download](download.md) per il changelog completo.
{% endhint %}

Chloros è disponibile in 3 modalità applicative:

## Chloros: Applicazione GUI desktop

Finestra separata autonoma con tutte le funzionalità. _Solo Windows._

## [Chloros CLI: Interfaccia a riga di comando](CLI.md)

Elaborazione batch da riga di comando. Perfetta per l&#x27;automazione, la creazione di script e il funzionamento senza interfaccia grafica. Disponibile su **Windows, Linux amd64 e Linux arm64 (NVIDIA Jetson)**. _La CLI richiede una licenza Chloros+ per l&#x27;accesso._

## [Chloros API: Python SDK](api-python-sdk.md)

Interfaccia Python programmatica per l&#x27;automazione e i flussi di lavoro personalizzati. Perfetta per pipeline di ricerca, integrazione con applicazioni Python esistenti e creazione di strumenti personalizzati. Disponibile su **tutte le piattaforme** tramite `pip install chloros-sdk`. _L&#x27;API richiede una licenza Chloros+ per l&#x27;accesso._***

## Piattaforme supportate

| Piattaforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Sì | Sì | Sì |
| **Linux amd64 (x86_64)** | No | Sì | Sì |
| **Linux arm64 (NVIDIA Jetson)** | No | Sì | Sì |

Per le istruzioni di installazione di Linux, consultare la sezione [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

Sebbene Chloros sia gratuito per la maggior parte delle attività, potresti scoprire di aver bisogno di qualcosa in più. È qui che una licenza a pagamento per Chloros+ può tornarti utile. Con una licenza Chloros+ puoi sbloccare nuove funzionalità quali:

* **Elaborazione multithread**: accelera notevolmente l&#x27;elaborazione delle immagini per progetti più grandi elaborando simultaneamente le immagini attraverso la pipeline.
* **Accelerazione GPU (CUDA)**: sfrutta le attuali opzioni di memoria GPU più elevate per velocizzare ulteriormente la pipeline di elaborazione delle immagini. Per ottenere i migliori risultati, consigliamo 4 GB o più di VRAM.
* **Chloros+**[**CLI**](CLI.md)**Accesso**: esegui Chloros+ dalla riga di comando per automatizzare e integrare nel tuo software.
* **Chloros+**[**API**](api-python-sdk.md)**Accesso:** esegui Chloros+ da Python per il controllo programmatico, consentendo una perfetta integrazione con le tue pipeline di ricerca, i flussi di lavoro di analisi dei dati e le applicazioni personalizzate.
* **Utilizzo su più dispositivi**: ogni licenza Chloros+ consente di registrare 2 o più dispositivi. Utilizza il tuo account MAPIR Cloud per gestire i dispositivi registrati. Aggiungi il supporto per più dispositivi aggiornando la tua licenza Chloros+.
* **Metodo avanzato di debayering sensibile alla texture:** un debayering di alta qualità sensibile ai bordi combinato con un modello di denoising AI/ML che rimuove quasi tutto il rumore del debayering. 
* **Formule personalizzate per indici multispettrali:** inserite indici multispettrali personalizzati nei calcolatori raster Chloros, sia per l&#x27;elaborazione che per l&#x27;area di prova di visualizzazione delle immagini.
* **Linux ed edge computing:** esegui Chloros su piattaforme Linux x86_64 e ARM64, incluso NVIDIA Jetson, per l&#x27;elaborazione sul campo e edge. Vedi [Panoramica di Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Prezzi e registrazione a Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
