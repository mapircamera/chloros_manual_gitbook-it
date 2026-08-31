---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Scarica l&#x27;ultima versione di Chloros per iniziare a elaborare immagini multispettrali.

### Requisiti di sistema

#### Windows

| Requisito          | Minimo                                              | Consigliato                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistema operativo** | Windows 10 (64 bit)                                  | Windows 11 (64 bit)                                  |
| **Processore**        | Intel Core i5 o equivalente                          | Intel Core i7 o superiore                              |
| **Memoria (RAM)**     | 8 GB                                                  | 16 GB o più                                         |
| **Scheda grafica**    | Compatibile con DirectX 11                                | GPU NVIDIA con 4 GB o più di VRAM                            |
| **Spazio su disco**          | 6 GB di spazio libero                                       | SSD con almeno 10 GB di spazio libero                            |
| **Schermo**          | 1920x1080                                            | 2560x1440 o superiore                                  |
| **Internet**         | Necessario per l’attivazione della licenza [opzionale] Chloros+ | Necessario per l’attivazione della licenza [opzionale] Chloros+ |

#### Linux amd64 (x86_64)

| Requisiti       | Minimi                    | Consigliati               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuzione**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processore**     | x86\_64 (Intel/AMD)        | Intel Core i7 o superiore   |
| **Memoria (RAM)**  | 8 GB                        | 16 GB o più              |
| **Scheda grafica** | Nessuna (elaborazione CPU)      | GPU NVIDIA con 4 GB o più di VRAM |
| **Spazio su disco**       | 2 GB di spazio libero             | SSD con almeno 10 GB di spazio libero       |
| **Python**        | Python 3.7+ (per SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Requisiti      | Minimi                      | Consigliati                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Piattaforma**     | NVIDIA Jetson con JetPack 6 | Jetson Orin NX da 16 GB o AGX Orin |
| **Memoria (RAM)** | 8 GB (condivisa tra GPU e CPU)         | 16 GB+ condivisa                    |
| **Spazio di archiviazione** | 2 GB di spazio libero               | SSD NVMe con 10 GB+ di spazio libero        |
| **Python**       | Python 3.7+ (per SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Accelerazione GPU**: gli utenti di Chloros+ con GPU NVIDIA possono utilizzare l’accelerazione CUDA per un’elaborazione notevolmente più veloce. Questa funzionalità è disponibile sia su Windows (GPU desktop) che su Linux (GPU desktop e NVIDIA Jetson). Gli utenti di Chloros+ beneficiano inoltre dell’elaborazione multithread per la massima velocità.
{% endhint %}

***

## Scarica Chloros

### Ultima versione stabile: versione 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Scarica Chloros per Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Scarica Chloros per Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Scarica Chloros per Linux arm64 / Jetson (.deb)</a>

#### Programma di installazione Windows (GUI + CLI + Backend)

* **Tipo di file**: .exe (Programma di installazione Windows)**Procedura di installazione:**

1. Scaricare il file .exe sopra indicato
2. Fare doppio clic sul programma di installazione per avviare l’installazione
3. Seguire le istruzioni della procedura guidata di installazione
4. Scegliere la directory di installazione (impostazione predefinita: `C:\Program Files\MAPIR\Chloros\`)
5. Completa l’installazione e avvia Chloros o Chloros CLI
6. Accedi con il tuo [account MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (oppure continua con la versione gratuita)

{% hint style="success" %}
Il programma di installazione aggiunge automaticamente `chloros-cli` al PATH di sistema per l&#x27;accesso dalla riga di comando.
{% endhint %}

#### Linux amd64 (pacchetto .deb — CLI + backend)

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

Consultare [Installazione di Linux](linux/linux-installation.md) per istruzioni dettagliate sulla configurazione e la [Guida NVIDIA Jetson](linux/nvidia-jetson-guide.md) per indicazioni specifiche su Jetson.

#### Python SDK (Tutte le piattaforme)

Ogni programma di installazione include un wheel `chloros_sdk` corrispondente, quindi la versione SDK corrisponde sempre alla GUI/CLI/backend installati. Su Windows il programma di installazione lo installa automaticamente nel sistema Python; su Linux il programma `.deb` colloca il wheel in `/usr/lib/chloros/sdk/` e visualizza il comando di installazione:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Per gli host che utilizzano solo pip (senza pacchetto Chloros installato), SDK è disponibile anche su PyPI:

```bash
pip install chloros-sdk
```

Vedi [API : Python SDK](api-python-sdk.md) e la [Guida di riferimento di SDK](reference/sdk-reference.md) per la documentazione.

{% hint style="info" %}
**Utenti di Linux**: Il pacchetto `.deb` installa CLI e il backend. Non esiste un&#x27;interfaccia grafica per Linux: tutte le interazioni avvengono tramite CLI o SDK.
{% endhint %}

***

## Risorse aggiuntive

### Python SDK

Per gli sviluppatori e i flussi di lavoro di automazione, installare Chloros, Python e SDK:

```bash
pip install chloros-sdk
```

**Documentazione**: [API: Python SDK](api-python-sdk.md)**Requisiti**: è necessario che Chloros sia installato (programma di installazione Windows o pacchetto Linux `.deb`), è richiesto l’accesso con licenza Chloros+***

## Cosa è incluso

### Programma di installazione Windows

* ✅ **Chloros GUI** - Interfaccia grafica completa
* ✅ **Chloros CLI** - Interfaccia a riga di comando (richiede una licenza Chloros+)
* ✅ **Backend Chloros** - Motore di elaborazione
* ✅ **Profili telecamera** - Modelli di telecamera MAPIR preconfigurati

### Pacchetto .deb Linux

* ✅ **Chloros CLI** - Interfaccia a riga di comando (richiede licenza Chloros+)
* ✅ **Backend Chloros** - Motore di elaborazione
* ✅ **Profili telecamera** - Modelli di telecamera MAPIR preconfigurati
* ❌ Nessuna interfaccia grafica (GUI) — Linux è solo in modalità headless CLI/SDK

### Python SDK (pip, tutte le piattaforme)

* ✅ **Chloros SDK** - Python API (richiede licenza Chloros+)***

## Passa a Chloros+

Sblocca funzionalità avanzate con un abbonamento Chloros+:

* 🚀 **Elaborazione multithread** - Elabora le immagini in parallelo
* ⚡ **Accelerazione GPU (CUDA)** - Sfrutta la potenza delle GPU NVIDIA
* 💻 **Accesso a CLI** - Automatizza le operazioni con strumenti da riga di comando
* 🐍 **Python SDK** - Accesso programmatico a API
* 📱 **Dispositivi multipli** - Utilizzabile su 2-10+ dispositivi (a seconda del piano)
* **🐻 Metodo avanzato di debayering sensibile alle texture** - un debayering di alta qualità sensibile ai contorni combinato con un modello di riduzione del rumore basato su IA/ML che elimina quasi tutto il rumore del debayering.
* 🧮 **Formule personalizzate** - Crea indici multispettrali personalizzati

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Visualizza i piani e i prezzi di Chloros+</a></p>***

## Guida all’installazione

### Risoluzione dei problemi

**L’installazione non va a buon fine con il seguente messaggio di errore:**

* Assicurati di disporre dei diritti di amministratore
* Disattiva temporaneamente il software antivirus
* Verifica di soddisfare i requisiti minimi di sistema

**L&#x27;applicazione non si avvia (Windows):**

* Verifica che Windows 10/11 (64 bit) sia installato
* Aggiorna i driver della scheda grafica
* Controlla il Visualizzatore eventi di Windows per i dettagli dell’errore
* Contatta l’assistenza fornendo i log degli errori

**CLI non si avvia (Linux):**

* Verificare che il pacchetto `.deb` sia stato installato correttamente: `dpkg -l | grep chloros`
* Controllare i permessi: `sudo chmod +x /usr/bin/chloros-cli`
* Eseguire la diagnostica: `chloros-cli selftest`
* Verificare la presenza di librerie mancanti: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemi di attivazione della licenza:**

* Assicurarsi che la connessione a Internet sia attiva
* Verificare le credenziali su [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verificare che il firewall non stia bloccando Chloros
* Consulta [Chloros+ Accesso](chloros+-login.md) per istruzioni dettagliate

### Assistenza

Hai bisogno di aiuto con l&#x27;installazione o la configurazione?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Sito web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentazione**: [Guida introduttiva](./)
* ❓ **FAQ**: [Domande frequenti](faq.md)***

## Aggiornamenti del software

Chloros verifica la disponibilità di aggiornamenti, avvisa quando è disponibile una nuova versione e rimanda a questa pagina di download: per eseguire l’aggiornamento è sufficiente eseguire il nuovo programma di installazione firmato. Le impostazioni e i progetti vengono mantenuti dopo gli aggiornamenti. Su Linux e Jetson, `chloros-cli update` verifica la presenza di una versione più recente e propone di scaricare e installare la versione corrispondente `.deb` (questo comando è disponibile solo su Linux).

***

## Registro delle modifiche**Versione 1.2.0 (ultima)**— consultare**Novità di Chloros 1.2.0** nella pagina [Introduzione](./) per l’elenco completo delle funzionalità.

<details>

<summary>Versione 1.0.5</summary>

**Data di rilascio: 10 febbraio 2026**

**Nuove funzionalità*** **Metodo di debayering “Texture Aware” \[Solo Chloros+] -** “Texture Aware” utilizza un debayering di alta qualità sensibile ai bordi, combinato con un modello di denoising basato su IA/ML che rimuove quasi tutto il rumore del debayering.
* **Supporto per i target di calibrazione T4P*** **Elaborazione GPU più veloce su Chloros+, migliore gestione della memoria**

**Correzioni di bug*** Interfaccia utente (GUI) completamente rinnovata, ora dovrebbe funzionare su tutti i computer Windows.

</details>

<details>

<summary>Versione 1.0.4</summary>

**Data di rilascio: 5 gennaio 2026**

**Nuove funzionalità*** **Opzione &quot;Immagine/Metadati&quot;**: Aggiunta un&#x27;opzione nel File Browser per visualizzare i metadati dell&#x27;immagine selezionata in una tabella anziché nella griglia delle immagini
* **Cursore di zoom della griglia delle immagini**: Nuovo cursore nell&#x27;interfaccia utente per regolare le dimensioni delle miniature (supporta anche CTRL + rotellina del mouse)
* **Pulsanti di esportazione della griglia di immagini**: pulsanti nella riga superiore per passare dalle miniature ai file esportati elaborati (Obiettivi, Riflettanza, Indice, LUT)
* **Scheda Mappa**: nuova mappa 2D interattiva che mostra i marcatori GPS delle immagini
  * Supporta Google Maps e le tessere cartografiche ESRI (seleziona automaticamente il miglior servizio di tessere in base alla disponibilità del livello di zoom)
  * Anteprima delle miniature al passaggio del mouse sui marcatori della mappa

**Correzioni di bug*** Migliorato il supporto per l’installazione di Chloros su computer con lingua diversa dall’inglese

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

È vietato l’uso, la distribuzione o la modifica non autorizzati.

**Versione gratuita**: disponibile per uso personale e commerciale con funzionalità limitate**Chloros+**: licenza in abbonamento per funzionalità avanzate e implementazioni commerciali
