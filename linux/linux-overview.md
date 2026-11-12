# Panoramica su Linux

Chloros 1.3.0 offre supporto nativo per Linux per la **CLI**e il**Python SDK** — elaborazione di immagini multispettrali in modalità headless, oltre al controllo in tempo reale delle telecamere LATTICE e dei sensori di luce DAQ — su workstation Linux, server e dispositivi edge NVIDIA Jetson.

{% hint style="info" %}
**Nessuna interfaccia grafica desktop su Linux.**L’interfaccia grafica desktop di Chloros è disponibile solo per Windows. Gli utenti Linux interagiscono con Chloros tramite la [CLI](../CLI.md) e il [Python SDK](../api-python-sdk.md). `.deb` aggiunge una voce**Chloros CLI** al menu delle applicazioni: semplicemente apre un emulatore di terminale che esegue `chloros-cli`.
{% endhint %}

***

## Matrice di supporto delle piattaforme

| Funzionalità | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interfaccia grafica desktop** | Sì | N/A | No | No |
| **Interfaccia CLI** (`chloros-cli`) | Sì | Sì | Sì | Sì |
| **Python SDK** (`chloros-sdk`) | Sì | Sì | Sì | Sì |
| **Pipeline di elaborazione delle immagini** | Sì | Sì | Sì | Sì |
| **Controllo della telecamera LATTICE (in tempo reale)** | Sì (scheda Telecamere) | Sì (`chloros-cli lattice`, SDK) | Sì | Sì |
| **Sensori di luce DAQ (in tempo reale)** | Sì (scheda Sensori di luce) | Sì (`chloros-cli daq pool-*`, SDK) | Sì | Sì |
| **Sincronizzazione temporale PTP (host come grandmaster)** | Sì | Sì (`chloros-cli time-sync`) | Sì | Sì |
| **Accelerazione GPU (CUDA)** | Sì | Sì | Sì | Sì (JetPack 6) |
| **Debayer Texture Aware** (richiede una GPU CUDA) | Sì (Chloros+) | Sì (Chloros+) | Sì (Chloros+) | Sì (Chloros+) |
| **Adattamento dinamico del calcolo** | Sì | Sì | Sì | Sì |
| **Backend come servizio di sistema** (`chloros-backend@<user>.service`) | No | No | Sì (attivabile) | Sì (attivabile) |
| **Programma di aggiornamento in loco** (`chloros-cli update`) | No (eseguire il programma di installazione) | No (eseguire il programma di installazione) | Sì | Sì |

***

## Architetture supportate

| Architettura | Descrizione | Pacchetto |
| --- | --- | --- |
| **amd64 (x86_64)** | Processori standard per desktop/server (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Processori ARM — Famiglia NVIDIA Jetson Orin (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Distribuzioni Linux supportate

* **Ubuntu 22.04 LTS o versioni successive** (amd64)
* **Debian 12 o versioni successive** (amd64)
* **NVIDIA JetPack 6** (arm64 — piattaforme Jetson Orin)

***

## Cosa ottengono gli utenti Linux

* **Chloros CLI** — l’interfaccia a riga di comando completa per l’elaborazione in batch, l’automazione e la creazione di script
* **Chloros Python SDK** — interfaccia Python programmatica per pipeline di ricerca e strumenti personalizzati, installata dal pacchetto `.deb` per il sistema `python3` (una wheel con versione compatibile per virtualenvs è presente in `/usr/lib/chloros/sdk/`; PyPI è destinato alle macchine prive del pacchetto Chloros)
* **Controllo delle telecamere LATTICE** — individuazione, connessione, configurazione e acquisizione da telecamere LATTICE e array multicamera sincronizzati tramite `chloros-cli lattice` e l’SDK; `.deb` include il runtime dell’SDK Arena richiesto dalle telecamere
* **Controllo dei sensori di luce DAQ** — collega i sensori DAQ-U/M/E, trasmetti in streaming spettri calibrati e registra file `.daq` tramite `chloros-cli daq pool-*` e l’SDK
* **Sincronizzazione temporale PTP** — il backend Chloros esegue il grandmaster PTP a cui le telecamere LATTICE e i sensori DAQ-E si sincronizzano; ispezionarlo con `chloros-cli time-sync` e mantenerlo in esecuzione in modalità headless con l’unità systemd `chloros-backend@<user>.service` (vedere [Installazione su Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automazione dei progetti** — esegui i progetti salvati in modalità headless con `chloros-cli project` e l’SDK `open_project`
* **Accelerazione GPU** — elaborazione accelerata tramite CUDA su GPU NVIDIA (GPU desktop su amd64 e la GPU del Jetson con JetPack 6)
* **Adattamento dinamico dell’elaborazione** — rilevamento automatico dell’hardware e selezione della strategia di elaborazione, con l’opzione `CHLOROS_STRATEGY` come via d’uscita per gli esperti
* **Tutte le funzionalità di elaborazione** — la stessa pipeline di Windows: calibrazione, correzione della vignettatura, indici di vegetazione e tutti i formati di esportazione
* **Funzionalità Chloros+** — elaborazione multithread (in pipeline), debayer Texture Aware (su una GPU CUDA) e indici personalizzati, con un piano Chloros+ a pagamento

## Cosa non hanno a disposizione gli utenti Linux

* **Interfaccia grafica desktop** — nessuna interfaccia grafica; tutte le interazioni avvengono tramite CLI o Python SDK
* **Visualizzatore di immagini** — nessun visualizzatore di immagini interattivo, vista a griglia o indicatori sulla mappa
* **Gestione visiva dei progetti** — i progetti vengono creati e gestiti tramite comandi CLI e chiamate SDK (l’hardware stesso — telecamere, sensori, acquisizione — rimane completamente controllabile dal terminale)

***

## Requisiti di licenza

L’accesso alla CLI e all’SDK richiede un **livello Chloros+ a pagamento — Copper o superiore**(Copper, Bronze, Silver, Gold). Il livello gratuito**Iron** non prevede l’accesso alla CLI/SDK. Il limite minimo è imposto dal backend, non solo dalla CLI:

| Situazione | Risposta del backend |
| --- | --- |
| Non connesso | `401` con `error_code: AUTH_REQUIRED` |
| Accesso effettuato con il piano gratuito Iron | `403` con `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` funziona su qualsiasi piano — è l’unica rotta esente dal filtro — quindi il motivo del rifiuto è sempre visibile.

***

## Guida introduttiva su Linux

1. **Installa Chloros** — consulta [Installazione su Linux](linux-installation.md) per l’installazione di `.deb`
2. **Verifica** — `chloros-cli --version` visualizza `Chloros CLI 1.3.0`; `chloros-cli selftest` esegue la diagnostica in 7 passaggi
3. **Verifica l’Python SDK** — l’installazione di `.deb` lo installa: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (in un virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Effettua l’accesso** — `chloros-cli login your@email.com 'your-password'` (una volta per macchina; gli aggiornamenti del pacchetto mantengono i dati di accesso, mentre la rimozione del pacchetto li cancella)
5. **Elabora il tuo primo set di dati** — `chloros-cli process ~/datasets/flight001`

Per NVIDIA Jetson, consultare la [Guida dedicata a NVIDIA Jetson](nvidia-jetson-guide.md) per la configurazione specifica della piattaforma, il comportamento termico e l’implementazione sul campo.

***

## Passi successivi

* [Installazione su Linux](linux-installation.md) — installazione dettagliata, percorsi dei file e risoluzione dei problemi per amd64 e arm64
* [Guida NVIDIA Jetson](nvidia-jetson-guide.md) — configurazione specifica per Jetson, comportamento termico e della memoria, implementazione sul campo
* [CLI: riga di comando](../CLI.md) — la guida alla CLI
* [API: Python SDK](../api-python-sdk.md) — la guida all’SDK
* [Riferimento CLI](../reference/cli-reference.md) e [Riferimento SDK](../reference/sdk-reference.md) — elenchi completi dei comandi e delle API per la versione 1.3.0
* [Adattamento dinamico delle risorse di calcolo](../processing-architecture/dynamic-compute-adaptation.md) — come Chlorossi adatta al tuo hardware

{% hint style="info" %}
**Lettura di questo manuale a livello di programmazione.** Ogni pagina è disponibile anche in formato Markdown grezzo al proprio URL più `.md` (ad esempio `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), mentre l’indice dell’intero manuale è pubblicato all’indirizzo [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
