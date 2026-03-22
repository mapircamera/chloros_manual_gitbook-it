# Panoramica di Linux

Chloros 1.1.0 introduce il supporto nativo per **CLI**e**Python SDK**, consentendo l&#x27;elaborazione di immagini multispettrali headless su workstation, server e dispositivi edge NVIDIA Jetson.

{% hint style="info" %}
**Nessuna GUI su Linux.** La GUI desktop Chloros è disponibile solo su Windows. Gli utenti di Linux interagiscono con Chloros tramite [CLI](../CLI.md) e [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Matrice di supporto della piattaforma

| Funzionalità | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interfaccia grafica desktop** | Sì | N/A | No | No |
| **CLI** | Sì | Sì | Sì | Sì |
| **Python SDK** | Sì | Sì | Sì | Sì |
| **Accelerazione GPU (CUDA)** | Sì | Sì | Sì | Sì (JetPack 6) |
| **Debayer sensibile alle texture** | Sì (Chloros+) | Sì (Chloros+) | Sì (Chloros+) | Sì (Chloros+) |
| **Adattamento dinamico del calcolo** | Sì | Sì | Sì | Sì |***

## Architetture supportate

| Architettura | Descrizione | Metodo di installazione |
| --- | --- | --- |
| **amd64 (x86_64)** | Processori desktop/server standard (Intel, AMD) | Pacchetto `.deb` |
| **arm64 (aarch64)** | Processori basati su ARM, principalmente NVIDIA Jetson | Pacchetto `.deb` (JetPack 6) |

## Distribuzioni Linux supportate

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — piattaforme Jetson)***

## Cosa ottengono gli utenti di Linux

* **Chloros CLI** — Interfaccia a riga di comando completa per l&#x27;elaborazione in batch, l&#x27;automazione e la creazione di script
* **Chloros Python SDK** — Interfaccia Python programmatica (`pip install chloros-sdk`) per l&#x27;integrazione in pipeline di ricerca e strumenti personalizzati
* **Accelerazione GPU** — Elaborazione accelerata da CUDA su GPU NVIDIA (desktop e Jetson)
* **Adattamento dinamico del calcolo** — Rilevamento automatico dell&#x27;hardware e ottimizzazione della strategia di elaborazione
* **Tutte le funzionalità di elaborazione** — Stessa pipeline di elaborazione multispettrale di Windows (calibrazione, correzione della vignettatura, indici di vegetazione, tutti i formati di esportazione)
* **Funzionalità di Chloros+** — Elaborazione multithread, debayer Texture Aware, indici personalizzati (con licenza Chloros+)

## Cosa non ottengono gli utenti di Linux

* **Interfaccia grafica desktop** — Nessuna interfaccia grafica; tutte le interazioni avvengono tramite CLI o Python SDK
* **Visualizzatore di immagini** — Nessun visualizzatore di immagini interattivo, vista a griglia o indicatori sulla mappa
* **Gestione visiva dei progetti** — I progetti vengono gestiti tramite comandi CLI e chiamate SDK***

## Guida introduttiva a Linux

1. **Installare Chloros** — Vedere [Installazione di Linux](linux-installation.md) per l&#x27;installazione del pacchetto `.deb`
2. **Installare Python SDK** (opzionale) — `pip install chloros-sdk`
3. **Attivare la licenza** — `chloros-cli login your@email.com 'password'`
4. **Elabora il tuo primo set di dati** — `chloros-cli process ~/datasets/flight001`

Per gli utenti NVIDIA Jetson, consulta la [Guida NVIDIA Jetson](nvidia-jetson-guide.md) dedicata per la configurazione e l&#x27;ottimizzazione specifiche della piattaforma.

***

## Passi successivi

* [Installazione di Linux](linux-installation.md) — Istruzioni dettagliate per l&#x27;installazione su amd64 e arm64
* [Guida NVIDIA Jetson](nvidia-jetson-guide.md) — Configurazione specifica per Jetson, gestione termica e implementazione sul campo
* [CLI : Riga di comando](../CLI.md) — Riferimento completo a CLI
* [API : Python SDK](../api-python-sdk.md) — Riferimento completo a SDK
* [Adattamento dinamico del calcolo](../processing-architecture/dynamic-compute-adaptation.md) — Come Chloros si adatta al tuo hardware
