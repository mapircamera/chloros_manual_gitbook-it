# Panoramica su Linux

Chloros 1.2.0 offre supporto nativo Linux per **CLI**e**Python SDK** — elaborazione di immagini multispettrali senza interfaccia grafica, oltre al controllo in tempo reale delle telecamere LATTICE e dei sensori di luce DAQ — su workstation, server e dispositivi edge NVIDIA Jetson.

{% hint style="info" %}
**Nessuna interfaccia grafica desktop su Linux.**L’interfaccia grafica desktop Chloros è disponibile solo su Windows. Gli utenti di Linux interagiscono con Chloros tramite [CLI](../CLI.md) e [Python SDK](../api-python-sdk.md). `.deb` aggiunge effettivamente una voce**Chloros CLI** al menu dell’applicazione — si limita semplicemente ad aprire un emulatore di terminale che esegue `chloros-cli`.
{% endhint %}

***

## Matrice di supporto delle piattaforme

| Funzionalità | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interfaccia grafica desktop** | Sì | N/A | No | No |
| **CLI** (`chloros-cli`) | Sì | Sì | Sì | Sì |
| **Python SDK** (`chloros-sdk`) | Sì | Sì | Sì | Sì |
| **Pipeline di elaborazione delle immagini** | Sì | Sì | Sì | Sì |
| **Controllo telecamere LATTICE (in tempo reale)** | Sì (scheda Telecamere) | Sì (`chloros-cli lattice`, SDK) | Sì | Sì |
| **Sensori di luce DAQ (in tempo reale)** | Sì (scheda Sensori di luce) | Sì (`chloros-cli daq pool-*`, SDK) | Sì | Sì |
| **Sincronizzazione temporale PTP (l’host è il grandmaster)** | Sì | Sì (`chloros-cli time-sync`) | Sì | Sì |
| **Accelerazione GPU (CUDA)** | Sì | Sì | Sì | Sì (JetPack 6) |
| **Debayer sensibile alle texture** | Sì (Chloros+) | Sì (Chloros+) | Sì (Chloros+) | Sì (Chloros+) |
| **Adattamento dinamico del calcolo** | Sì | Sì | Sì | Sì |
| **Backend come servizio di sistema** (`chloros-backend.service`) | No | No | Sì (abilitazione opzionale) | Sì (abilitazione opzionale) |
| **Programma di aggiornamento in loco** (`chloros-cli update`) | No (eseguire il programma di installazione) | No (eseguire il programma di installazione) | Sì | Sì |***

## Architetture supportate

| Architettura | Descrizione | Pacchetto |
| --- | --- | --- |
| **amd64 (x86_64)** | Processori standard per desktop/server (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Processori ARM — Famiglia NVIDIA Jetson Orin | `chloros_<version>_arm64_jp6.deb` (build JetPack 6) |

## Distribuzioni Linux supportate

* **Ubuntu 22.04 LTS o versioni successive** (amd64)
* **Debian 12 o versioni successive** (amd64)
* **NVIDIA JetPack 6** (arm64 — piattaforme Jetson Orin)***

## Cosa ottengono gli utenti di Linux

* **Chloros CLI** — l’interfaccia a riga di comando completa per l’elaborazione in batch, l’automazione e la creazione di script
* **Chloros Python SDK** — interfaccia Python programmabile per pipeline di ricerca e strumenti personalizzati (installabile da PyPI e inclusa anche all’interno di `.deb` come wheel con versione corrispondente)
* **Controllo delle telecamere LATTICE** — individuazione, connessione, configurazione e acquisizione da telecamere LATTICE e array multicamera sincronizzati tramite `chloros-cli lattice` e SDK; `.deb` include il runtime Arena SDK necessario per le telecamere
* ****Controllo dei sensori di luce DAQ** — collega i sensori DAQ-U/M/E, trasmetti in streaming spettri calibrati e registra file `.daq` tramite `chloros-cli daq pool-*` e SDK
* **Sincronizzazione temporale PTP** — il backend Chloros esegue il grandmaster PTP a cui le telecamere LATTICE e i sensori DAQ-E si sincronizzano come slave; controllalo con `chloros-cli time-sync`, e mantienilo in esecuzione in modalità headless con l’unità systemd `chloros-backend.service` (vedi [Installazione di Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automazione dei progetti** — esegui i progetti salvati in modalità headless con `chloros-cli project` e `open_project` di SDK
* **Accelerazione GPU** — elaborazione accelerata tramite CUDA su GPU NVIDIA (desktop e Jetson)
* **Adattamento dinamico del calcolo** — rilevamento automatico dell’hardware e selezione della strategia di elaborazione, con l’opzione di override `CHLOROS_STRATEGY` come via d’uscita per gli esperti
* **Tutte le funzionalità di elaborazione** — la stessa pipeline di Windows: calibrazione, correzione della vignettatura, indici di vegetazione e tutti i formati di esportazione
* **Funzionalità di Chloros+** — elaborazione multithread (in pipeline), debayer Texture Aware e indici personalizzati, con un piano a pagamento Chloros+

## Cosa non è disponibile per gli utenti di Linux

* **Interfaccia grafica desktop** — nessuna interfaccia grafica; tutte le interazioni avvengono tramite CLI o Python SDK
* **Visualizzatore di immagini** — nessun visualizzatore di immagini interattivo, vista a griglia o indicatori sulla mappa
* **Gestione visiva dei progetti** — i progetti vengono creati e gestiti tramite comandi CLI e chiamate SDK (l’hardware stesso — telecamere, sensori, acquisizione — rimane completamente controllabile dal terminale)***

## Requisiti di licenza

L’accesso a CLI e SDK richiede un **livello a pagamento Chloros+ — Copper o superiore**(Copper, Bronze, Silver, Gold). Il livello gratuito**Iron** non prevede l’accesso a CLI/SDK. Il limite minimo è imposto dal backend, non solo da CLI:

| Situazione | Risposta del backend |
| --- | --- |
| Non connesso | `401` con `error_code: AUTH_REQUIRED` |
| Accesso effettuato sul livello gratuito Iron | `403` con `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` funziona su qualsiasi piano — è l’unico percorso esente dal gate — quindi il motivo del rifiuto è sempre visibile.

***

## Guida introduttiva a Linux

1. **Installa Chloros** — consulta [Installazione di Linux](linux-installation.md) per l’installazione di `.deb`
2. **Verifica** — `chloros-cli --version` stampa `Chloros CLI 1.2.0`; `chloros-cli selftest` esegue la diagnostica in 7 fasi
3. **Installare Python e SDK** (facoltativo) — `pip install chloros-sdk`
4. **Accedere** — `chloros-cli login your@email.com 'your-password'` (una volta per ogni macchina e nuovamente dopo ogni aggiornamento del pacchetto)
5. **Elaborare il primo set di dati** — `chloros-cli process ~/datasets/flight001`

Per NVIDIA Jetson, consultare la [Guida dedicata a NVIDIA Jetson](nvidia-jetson-guide.md) per la configurazione specifica della piattaforma, il comportamento termico e l’implementazione sul campo.

***

## Passaggi successivi

* [Installazione di Linux](linux-installation.md) — installazione dettagliata, percorsi dei file e risoluzione dei problemi per amd64 e arm64
* [Guida NVIDIA Jetson](nvidia-jetson-guide.md) — configurazione specifica per Jetson, comportamento termico e della memoria, implementazione sul campo
* [CLI : Riga di comando](../CLI.md) — la guida CLI
* [API : Python SDK](../api-python-sdk.md) — la guida SDK
* [Riferimento CLI](../reference/cli-reference.md) e [Riferimento SDK](../reference/sdk-reference.md) — elenco esaustivo dei comandi/API per la versione 1.2.0
* [Adattamento dinamico delle risorse di calcolo](../processing-architecture/dynamic-compute-adaptation.md) — come Chloros si adatta al proprio hardware

{% hint style="info" %}
**Lettura di questo manuale tramite programmazione.** Ogni pagina è disponibile anche in formato Markdown grezzo al proprio URL più `.md` (ad esempio `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), mentre un indice dell’intero manuale è pubblicato all’indirizzo [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
