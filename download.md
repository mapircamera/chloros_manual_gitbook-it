---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Scarica

Scarica l&#x27;ultima versione di Chloros per iniziare a lavorare con l&#x27;elaborazione di immagini multispettrali.

### Requisiti di sistema

#### Windows

| Requisito          | Minimo                                              | Consigliato                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistema operativo** | Windows 10 (64 bit)                                  | Windows 11 (64 bit)                                  |
| **Processore**        | Intel Core i5 o equivalente                          | Intel Core i7 o superiore                              |
| **Memoria (RAM)**     | 8 GB                                                  | 16 GB o più                                         |
| **Scheda grafica**    | Compatibile con DirectX 11                                | GPU NVIDIA con 4 GB+ di VRAM                            |
| **Spazio su disco**          | 6 GB di spazio libero                                       | SSD con 10 GB+ di spazio libero                            |
| **Schermo**          | 1920x1080                                            | 2560x1440 o superiore                                  |
| **Internet**         | Richiesto per l&#x27;attivazione della licenza \[opzionale] Chloros+ | Richiesto per l&#x27;attivazione della licenza \[opzionale] Chloros+ |

#### Linux amd64 (x86\_64)

| Requisiti       | Minimi                    | Consigliati               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuzione**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Processore**     | x86\_64 (Intel/AMD)        | Intel Core i7 o superiore   |
| **Memoria (RAM)**  | 8 GB                        | 16 GB o più              |
| **Scheda grafica** | Nessuna (elaborazione CPU)      | GPU NVIDIA con 4 GB+ di VRAM |
| **Spazio su disco** | 2 GB di spazio libero             | SSD con 10 GB+ di spazio libero       |
| **Python**        | Python 3.7+ (per SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Requisiti      | Minimi                      | Consigliati                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Piattaforma**     | NVIDIA Jetson con JetPack 6 | Jetson Orin NX 16 GB o AGX Orin |
| **Memoria (RAM)** | 8 GB (condivisa GPU/CPU)         | 16 GB+ condivisa                    |
| **Spazio di archiviazione**      | 2 GB di spazio libero               | SSD NVMe con 10 GB+ di spazio libero        |
| **Python**       | Python 3.7+ (per SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Accelerazione GPU**: gli utenti di Chloros+ con GPU NVIDIA possono utilizzare l&#x27;accelerazione CUDA per un&#x27;elaborazione significativamente più veloce. Ciò funziona sia su Windows (GPU desktop) che su Linux (GPU desktop e NVIDIA Jetson). Gli utenti di Chloros+ beneficiano inoltre dell&#x27;elaborazione multithread per la massima velocità.
{% endhint %}

***

## Scarica Chloros

### Ultima versione stabile (23 marzo 2026): Versione 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Scarica Chloros per Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Scarica Chloros per Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Scarica Chloros per Linux arm64 / Jetson (.deb)</a>

#### Programma di installazione Windows (GUI + CLI + Backend)

* **Tipo di file**: .exe (Programma di installazione Windows)**Procedura di installazione:**

1. Scaricare il file .exe sopra indicato
2. Fare doppio clic sul programma di installazione per avviare l&#x27;installazione
3. Seguire le istruzioni della procedura guidata di installazione
4. Scegli la directory di installazione (impostazione predefinita: `C:\Program Files\[USER]\Chloros\`)
5. Completa l&#x27;installazione e avvia Chloros o Chloros CLI
6. Accedi con il tuo [account MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (oppure continua con la versione gratuita)

{% hint style="success" %}
Il programma di installazione aggiunge automaticamente `chloros-cli` al PATH di sistema per l&#x27;accesso da riga di comando.
{% endhint %}

#### Linux amd64 (pacchetto .deb — CLI + Backend)

* **Tipo di file**: .deb (pacchetto Debian/Ubuntu)
* **Architettura**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (pacchetto .deb — CLI + Backend)

* **Tipo di file**: .deb (JetPack 6)
* **Architettura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Vedere [Installazione di Linux](linux/linux-installation.md) per istruzioni dettagliate sulla configurazione e la [Guida NVIDIA Jetson](linux/nvidia-jetson-guide.md) per indicazioni specifiche su Jetson.

#### Python SDK (Tutte le piattaforme)

```bash
pip install chloros-sdk
```

Vedere [API : Python SDK](api-python-sdk.md) per la documentazione.

{% hint style="info" %}
**Utenti di Linux**: Il pacchetto `.deb` installa CLI e il backend. Python e SDK vengono installati separatamente tramite pip. Non esiste una GUI per Linux — tutte le interazioni avvengono tramite CLI o SDK.
{% endhint %}

***

## Risorse aggiuntive

### Python SDK

Per gli sviluppatori e i flussi di lavoro di automazione, installare Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentazione**: [API: Python SDK](api-python-sdk.md)**Requisiti**: Chloros deve essere installato (programma di installazione Windows o pacchetto Linux `.deb`), è richiesto l&#x27;accesso con licenza Chloros+***

## Cosa è incluso

### Programma di installazione Windows

* ✅ **Chloros GUI** - Interfaccia grafica completa
* ✅ **Chloros CLI** - Interfaccia a riga di comando (richiede licenza Chloros+)
* ✅ **Backend Chloros** - Motore di elaborazione
* ✅ **Profili fotocamera** - Modelli di fotocamera MAPIR preconfigurati

### Pacchetto .deb Linux

* ✅ **Chloros CLI** - Interfaccia a riga di comando (richiede licenza Chloros+)
* ✅ **Backend Chloros** - Motore di elaborazione
* ✅ **Profili della telecamera** - Modelli di telecamera MAPIR preconfigurati
* ❌ Nessuna GUI — Linux è solo CLI/SDK headless

### Python SDK (pip, tutte le piattaforme)

* ✅ **Chloros SDK** - Python API (richiede licenza Chloros+)***

## Passa a Chloros+

Sblocca funzionalità avanzate con un abbonamento Chloros+:

* 🚀 **Elaborazione multithread** - Elabora le immagini in parallelo
* ⚡ **Accelerazione GPU (CUDA)** - Sfrutta la potenza delle GPU NVIDIA
* 💻 **Accesso CLI** - Automatizza con strumenti da riga di comando
* 🐍 **Python SDK** - Accesso programmatico a API
* 📱 **Dispositivi multipli** - Utilizzabile su 2-10+ dispositivi (a seconda del piano)
* **🐻 Metodo avanzato di debayering sensibile alle texture** - un debayering di alta qualità sensibile ai bordi combinato con un modello di denoising AI/ML che rimuove quasi tutto il rumore del debayering.
* 🧮 **Formule personalizzate** - Crea indici multispettrali personalizzati

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Visualizza piani e prezzi Chloros+</a></p>***

## Guida all&#x27;installazione

### Risoluzione dei problemi

**L&#x27;installazione non va a buon fine con il seguente messaggio di errore:**

* Assicurati di disporre dei diritti di amministratore
* Disattiva temporaneamente il software antivirus
* Verifica di soddisfare i requisiti minimi di sistema

**L&#x27;applicazione non si avvia (Windows):**

* Verifica che Windows 10/11 (64 bit) sia installato
* Aggiorna i driver grafici
* Controlla Windows Visualizzatore eventi per i dettagli dell&#x27;errore
* Contatta l&#x27;assistenza con i log degli errori

**CLI non si avvia (Linux):**

* Verifica che il pacchetto `.deb` sia installato correttamente: `dpkg -l | grep chloros`
* Controlla i permessi: `sudo chmod +x /usr/bin/chloros-cli`
* Eseguire la diagnostica: `chloros-cli selftest`
* Verificare la presenza di librerie mancanti: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemi di attivazione della licenza:**

* Assicurarsi che la connessione a Internet sia attiva
* Verificare le credenziali su [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verificare che il firewall non stia bloccando Chloros
* Consultare [Chloros+ Login](chloros+-login.md) per istruzioni dettagliate

### Assistenza

Hai bisogno di aiuto con l&#x27;installazione o la configurazione?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Sito web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentazione**: [Guida introduttiva](./)
* ❓ **FAQ**: [Domande frequenti](faq.md)***

## Registro delle modifiche

<details>

<summary>Versione 1.1.0 (Ultima)</summary>

**Data di rilascio: marzo 2026**

**Nuove funzionalità*** **Supporto Linux** — CLI e SDK nativi per Linux amd64 (x86\_64) e arm64 (NVIDIA Jetson JetPack 6). Installazione tramite pacchetti `.deb`.
* **Supporto NVIDIA Jetson** — Elaborazione ottimizzata per dispositivi edge Jetson Nano, Orin Nano, Orin NX e AGX Orin.
* **Adattamento dinamico dell&#x27;elaborazione** — Rilevamento automatico dell&#x27;hardware e ottimizzazione della strategia di elaborazione. Chloros si adatta al tuo hardware, da un Jetson Nano a una workstation multi-GPU.
* **Pipeline di elaborazione a 4 thread** — Thread simultanei di rilevamento, calibrazione, elaborazione ed esportazione con allocazione dinamica della memoria GPU.
* **Nuovi comandi CLI** — `selftest` (diagnostica di sistema) e `update` (gestione degli aggiornamenti Linux).
* **Nuovi flag di processo CLI** — `--debayer` (standard/sensibile alle texture), `--indices` (specifica indici), `--target` (cerca prima nella sottocartella di destinazione per un rilevamento più veloce).
* **Nuove voci del menu GUI** — Aggiungi file, Aggiungi cartella e Avvia/Interrompi elaborazione ora accessibili dal menu a tendina principale.**Miglioramenti**

* Rilevamento automatico del backend multipiattaforma (percorsi Windows e Linux)
* Miglioramento di SDK `get_status()` con monitoraggio dell&#x27;avanzamento per thread
* Nuove eccezioni SDK: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Gestione termica e throttling adattivo per NVIDIA Jetson
* Gestione automatica della memoria con fallback OOM all&#x27;elaborazione GPU a tessere

</details>

<details>

<summary>Versione 1.0.5</summary>

**Data di rilascio: 10 febbraio 2026**

**Nuove funzionalità*** **Metodo di debayering Texture Aware \[Solo Chloros+] -** Texture Aware utilizza un debayering di alta qualità sensibile ai bordi combinato con un modello di denoising AI/ML che rimuove quasi tutto il rumore del debayering.
* **Supporto per i target di calibrazione T4P*** **Elaborazione GPU Chloros+ più veloce, migliore gestione della memoria**

**Correzioni di bug*** Interfaccia utente (GUI) completamente nuova, ora dovrebbe funzionare su tutti i computer Windows.

</details>

<details>

<summary>Versione 1.0.4</summary>

**Data di rilascio: 5 gennaio 2026**

**Nuove funzionalità*** **Alternanza immagine/metadati**: Aggiunta un&#x27;opzione nel File Browser per visualizzare i metadati dell&#x27;immagine selezionata in una tabella invece che nella griglia delle immagini
* **Cursore di zoom della griglia delle immagini**: Nuovo cursore nell&#x27;interfaccia utente per regolare le dimensioni delle miniature (supporta anche CTRL + rotellina del mouse)
* **Pulsanti di esportazione della griglia di immagini**: Pulsanti nella riga superiore per passare dalle miniature alle esportazioni elaborate (Obiettivi, Riflettanza, Indice, LUT)
* **Scheda Mappa**: Nuova mappa 2D interattiva che mostra i marcatori di posizione GPS delle immagini
  * Supporta Google Maps e le tessere cartografiche ESRI (seleziona automaticamente il miglior servizio di tessere in base alla disponibilità del livello di zoom)
  * Anteprima delle miniature al passaggio del mouse sui marcatori della mappa

**Correzioni di bug*** Migliorato il supporto per l&#x27;installazione di Chloros su computer con lingua diversa dall&#x27;inglese

</details>

<details>

<summary>Versione 1.0.3</summary>

**Data di rilascio: 20 dicembre 2025**

**Nuove funzionalità*** Lancio iniziale

**Miglioramenti*** Lancio iniziale

**Correzioni di bug*** Lancio iniziale

**Problemi noti*** Lancio iniziale

</details>***

## Contratto di licenza**Software proprietario** - Copyright (c) 2026 MAPIR Inc.

È vietato l&#x27;uso, la distribuzione o la modifica non autorizzati.

**Versione gratuita**: Disponibile per uso personale e commerciale con funzionalità limitate**Chloros+**: Licenza in abbonamento per funzionalità avanzate e implementazioni commerciali
