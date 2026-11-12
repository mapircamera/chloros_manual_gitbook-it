---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Scarica l'ultima versione di Chloros per iniziare a elaborare immagini multispettrali.

### Requisiti di sistema

#### Windows


| Requisito          | Minimo                                              | Consigliato                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistema operativo** | Windows 10 (64 bit)                                  | Windows 11 (64 bit)                                  |
| **Processore**        | Intel Core i5 o equivalente                          | Intel Core i7 o superiore                              |
| **Memoria (RAM)**     | 8 GB                                                  | 16 GB o più                                         |
| **Scheda grafica**    | Compatibile con DirectX 11                                | GPU NVIDIA con almeno 4 GB di VRAM                            |
| **Spazio su disco**          | 6 GB di spazio libero                                       | SSD con almeno 10 GB di spazio libero                            |
| **Schermo**          | 1920x1080                                            | 2560x1440 o superiore                                  |
| **Internet**         | Necessario per l’attivazione [opzionale] della licenza di Chloros+ | Necessario per l’attivazione [opzionale] della licenza di Chloros+ |

#### Linux amd64 (x86_64)

| Requisiti       | Minimi                    | Consigliati               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuzione**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processore**     | x86\_64 (Intel/AMD)        | Intel Core i7 o superiore   |
| **Memoria (RAM)**  | 8 GB                        | 16 GB o più              |
| **Scheda grafica** | Nessuna (elaborazione via CPU)      | GPU NVIDIA con 4 GB o più di VRAM |
| **Spazio su disco**       | 2 GB di spazio libero             | SSD con almeno 10 GB di spazio libero       |
| **Python**        | Python 3.7+ (per SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Requisiti      | Minimi                      | Consigliati                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Piattaforma**     | NVIDIA Jetson con JetPack 6 | Jetson Orin NX da 16 GB o AGX Orin |
| **Memoria (RAM)** | 8 GB (condivisa tra GPU e CPU)         | 16 GB+ condivisa                    |
| **Spazio di archiviazione**      | 2 GB di spazio libero               | SSD NVMe con almeno 10 GB di spazio libero        |
| **Python**       | Python 3.7+ (per l’SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Accelerazione GPU**: gli utenti di Chloros+ con GPU NVIDIA possono utilizzare l’accelerazione CUDA per un’elaborazione notevolmente più veloce. Questa funzionalità è compatibile con le GPU desktop NVIDIA su Windows e Linux amd64, nonché su NVIDIA Jetson con JetPack 6. Gli utenti di Chloros+ beneficiano inoltre dell’elaborazione multithread per la massima velocità.
{% endhint %}

***

## Scarica Chloros


### Ultima versione stabile: Versione 1.3.0 (11 novembre 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Scarica Chloros per Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Scarica Chloros per Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Scarica Chloros per Linux arm64 / Jetson (.deb)</a>


#### Programma di installazione per Windows (GUI + CLI + Backend)

* **Tipo di file**: .exe (Programma di installazione per Windows)

**Procedura di installazione:**

1. Scaricare il file .exe sopra indicato
2. Fare doppio clic sul programma di installazione per avviare l’installazione
3. Seguire le istruzioni della procedura guidata di installazione
4. Scegliere la directory di installazione (impostazione predefinita: `C:\Program Files\Chloros\`)
5. Nella pagina **Seleziona attività aggiuntive**, lasciare selezionate entrambe le opzioni**Impostazioni di sistema** (consigliato — vedi sotto)
6. Completare l’installazione e avviare Chloros o Chloros CLI
7. Accedere con il proprio [account MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (oppure continua con la versione gratuita)

**Opzioni delle impostazioni di sistema** (entrambe selezionate per impostazione predefinita):

| Opzione, come indicata nel programma di installazione | Cosa fa | Disattivazione dell’installazione silenziosa |
| --- | --- | --- |
| **Abilita percorsi lunghi di Windows** (consente di avere cartelle di progetto con nomi più lunghi di 260 caratteri; consigliato) | Imposta il parametro `LongPathsEnabled` di Windows a livello di sistema su 1. Windows viene fornito con questa opzione disattivata, per cui nessun percorso di file può superare i 260 caratteri; i prodotti Chloros si trovano fino a circa 130 caratteri al di sotto della cartella di progetto, quindi con i percorsi lunghi disattivati una cartella di progetto più profonda viene rifiutata. L’impostazione ha effetto sui programmi avviati dopo l’installazione. Rimane attiva anche dopo la disinstallazione, poiché altri programmi potrebbero farne uso. | `/MERGETASKS="!longpaths"` |
| **Escludi il backend di Chloros dalla scansione di Microsoft Defender** (acquisizione delle immagini molto più veloce; consigliato) | Aggiunge un'esclusione del processo di Microsoft Defender per `chloros-backend.exe` nella cartella di installazione, in modo che la protezione in tempo reale non esegua la scansione di ogni file scritto dal backend, indipendentemente dalla posizione in cui viene scritto. A frequenze di acquisizione elevate, la scansione incide per circa il 30% della frequenza di acquisizione (77,7 contro 52,7 acquisizioni al secondo in un’acquisizione continua con 9 telecamere). Nessun altro programma viene escluso e l’esclusione viene rimossa al momento della disinstallazione. Laddove Defender sia assente o bloccato da criteri di sicurezza, questo passaggio fallisce silenziosamente e l’installazione prosegue. | `/MERGETASKS="!defenderexclusion"` |

Un’installazione silenziosa (`/SILENT` o `/VERYSILENT`) applica entrambe le opzioni a meno che non si scelga di escluderle con il valore `/MERGETASKS` indicato.

{% hint style="success" %}
Il programma di installazione aggiunge la cartella CLI (`C:\Program Files\Chloros\cli\`) al sistema `PATH` una sola volta: la reinstallazione o l’aggiornamento non aggiungono più una voce duplicata e rimuovono i duplicati aggiunti da installazioni precedenti. Aprire un nuovo terminale dopo l’installazione per utilizzare `chloros-cli`. Il programma di installazione aggiunge inoltre due regole del Firewall di Windows per il backend, su ogni profilo di rete: **Backend Chloros (UDP-In)**, per i flussi delle telecamere, PTP, mDNS e il flusso di dati DAQ-E, e**Backend Chloros (TCP-In)**, limitato alla sottorete locale, per l’aggiornamento del firmware DAQ-E, in cui il sensore si riconnette a questo computer per acquisire l’immagine. Entrambe vengono rimosse al momento della disinstallazione.
{% endhint %}

#### Linux amd64 (pacchetto .deb — CLI + Backend)

* **Tipo di file**: .deb (pacchetto Debian/Ubuntu)
* **Architettura**: x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (pacchetto .deb — CLI + Backend)

* **Tipo di file**: .deb (JetPack 6)
* **Architettura**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Utilizza il nome del file che hai effettivamente scaricato e mantieni il nome `./` in modo che `apt` installi quel file anziché cercare un pacchetto in base al nome. Esegui l’installazione con `apt install`, non con `dpkg -i`: apt installa anche i prerequisiti dell'Python SDK, che il pacchetto raccomanda anziché richiedere, mentre `dpkg -i` li ignora. A partire dalla versione 1.3.0, il backend e l’CLI vengono installati già decompressi sotto `/usr/lib/chloros/`, quindi si avviano senza doversi prima decomprimere.

Il pacchetto Jetson utilizza la build CUDA di PyTorch di JetPack 6, quindi processing, compreso il debayer Texture Aware, viene eseguito sulla GPU di Jetson. Utilizza le librerie CUDA e cuDNN installate da JetPack 6 (`sudo apt install nvidia-jetpack` su un'immagine che ne è sprovvista).

Consultare [Installazione su Linux](linux/linux-installation.md) per istruzioni dettagliate sulla configurazione e la [Guida NVIDIA Jetson](linux/nvidia-jetson-guide.md) per indicazioni specifiche su Jetson.

#### Python SDK (Tutte le piattaforme)

Ogni programma di installazione include un wheel `chloros_sdk` corrispondente, in modo che la versione dell’SDK corrisponda sempre alla GUI/CLI/backend installati. Su Windows, il programma di installazione lo installa automaticamente nel Python di sistema. Su Linux, il file `.deb` installa l’SDK per il sistema `python3` stesso (all’interno di `/usr/lib/python3/dist-packages`), quindi `import chloros_sdk` funziona subito dopo l’installazione, senza bisogno di pip né di connessione di rete. Solo un virtualenv, che non ha accesso ai pacchetti di sistema, richiede l’installazione del wheel in dotazione al suo interno:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Per gli host che utilizzano solo pip (senza pacchetto Chloros installato; l’SDK gestisce quindi un backend su un’altra macchina), l’SDK è disponibile anche su PyPI:

```bash
pip install chloros-sdk
```

Per la documentazione, consultare [API: Python SDK](api-python-sdk.md) e il [Riferimento SDK](reference/sdk-reference.md).

{% hint style="info" %}
**Utenti Linux**: il pacchetto `.deb` installa la CLI, il backend e l’SDK per Python. Non esiste una GUI per Linux: tutte le interazioni avvengono tramite CLI o SDK.
{% endhint %}

***

## Risorse aggiuntive

### SDK Python per Chloros


Per gli sviluppatori e i flussi di lavoro di automazione, l’Chloros Python SDK per Chloros è incluso in ogni programma di installazione: il programma di installazione per Windows lo installa nel vostro Python, mentre quello per Linux `.deb` lo installa per il sistema `python3`. Su una macchina priva del pacchetto Chloros (che gestisce un backend su un’altra macchina), installarlo da PyPI:

```bash
pip install chloros-sdk
```

**Documentazione**: [API: Python SDK](api-python-sdk.md)

**Requisiti**: un backend Chloros da gestire (l’installatore per Windows o il pacchetto `.deb` per Linux ne installano uno) e un account Chloros+ su un piano a pagamento

***

## Cosa è incluso

### Installatore per Windows


* ✅ **Interfaccia grafica Chloros** - Interfaccia grafica completa
* ✅ **Chloros CLI** - Interfaccia a riga di comando (richiede licenza Chloros+)
* ✅ **Backend Chloros** - Motore di elaborazione
* ✅ **Python SDK** - installato nel proprio Python se presente (richiede licenza Chloros+)
* ✅ **Profili delle telecamere** - Modelli di telecamere MAPIR preconfigurati

### Pacchetto .deb per Linux


* ✅ **Chloros CLI** - Interfaccia a riga di comando (richiede licenza Chloros+)
* ✅ **Chloros Backend** - Motore di elaborazione
* ✅ **Python SDK** - Installato per il sistema `python3` (richiede licenza Chloros+)
* ✅ **Profili delle telecamere** - Modelli preconfigurati per telecamere MAPIR

* ❌ Nessuna interfaccia grafica (GUI) — Linux è solo CLI/SDK senza interfaccia grafica

### Python SDK (PyPI, per macchine senza pacchetto Chloros)

* ✅ **Chloros SDK** - Python API (richiede licenza Chloros+)

***

## Passa a Chloros+

Sblocca funzionalità avanzate con un abbonamento a Chloros+:

* 🚀 **Elaborazione multithread** - Elabora le immagini in parallelo
* ⚡ **Accelerazione GPU (CUDA)** - Sfrutta la potenza delle GPU NVIDIA
* 💻 **Accesso CLI** - Automatizza con strumenti da riga di comando
* 🐍 **Python SDK** - Accesso programmatico alle API

* 📱 **Dispositivi multipli** - Utilizzabile su 2-10+ dispositivi (a seconda del piano)
* **🐻 Metodo avanzato di debayering sensibile alle texture** - un debayering di alta qualità sensibile ai bordi, combinato con un modello di riduzione del rumore basato su IA/ML che elimina quasi tutto il rumore del debayering.
* 🧮 **Formule personalizzate** - Crea indici multispettrali personalizzati

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Visualizza i piani e i prezzi di Chloros+</a></p>

***

## Guida all’installazione

### Risoluzione dei problemi

**L’installazione non va a buon fine con il seguente messaggio di errore:**

* Assicurati di disporre dei diritti di amministratore
* Disattiva temporaneamente il software antivirus
* Verifica di soddisfare i requisiti minimi di sistema

**L’applicazione non si avvia (Windows):**

* Verifica che Windows 10/11 (64 bit) sia installato
* Aggiorna i driver della scheda grafica
* Controlla il Visualizzatore eventi di Windows per i dettagli dell’errore
* Contatta l’assistenza fornendo i log degli errori

**La CLI non si avvia (Linux):**

* Verifica che il pacchetto `.deb` sia installato correttamente: `dpkg -l | grep chloros`
* Controlla i permessi: `sudo chmod +x /usr/bin/chloros-cli`
* Esegui la diagnostica: `chloros-cli selftest`
* Verificare la presenza di librerie mancanti: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (a partire dalla versione 1.3.0, `/usr/lib/chloros/chloros-backend` è un piccolo script di avvio; il programma vero e proprio si trova in `/usr/lib/chloros/backend/`)

**Problemi di attivazione della licenza:**

* Assicurarsi che la connessione a Internet sia attiva
* Verificare le credenziali su [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Controlla che il firewall non blocchi Chloros

* Consulta [Chloros+ Login](chloros+-login.md) per istruzioni dettagliate

### Assistenza

Hai bisogno di aiuto con l’installazione o la configurazione?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Sito web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentazione**: [Guida introduttiva](./)
* ❓ **FAQ**: [Domande frequenti](faq.md)

***

## Aggiornamenti software

Chloros verifica la presenza di aggiornamenti, avvisa quando è disponibile una nuova versione e fornisce un link a questa pagina di download: per aggiornare è sufficiente eseguire il nuovo programma di installazione firmato. Le impostazioni, i progetti e i dati di accesso rimangono invariati dopo gli aggiornamenti. Su Linux e Jetson, `chloros-cli update` verifica la presenza di una versione più recente e propone di scaricare e installare la versione corrispondente `.deb` (questo comando è disponibile solo su Linux).

***

## Registro delle modifiche**Versione 1.3.0 (ultima, 11 novembre 2026)**— consulta**Novità di Chloros

1.3.0** nella pagina [Introduzione](./) per l’elenco completo.**Versione 1.2.0**— consultare la sezione**Novità di Chloros 1.2.0** nella pagina [Introduzione](./).

<details>

<summary>Versione 1.0.5</summary>

**Data di rilascio: 10 febbraio 2026**

**Nuove funzionalità**

* **Metodo di debayering "Texture Aware" \[Solo Chloros+] -** "Texture Aware" utilizza un debayering di alta qualità sensibile ai bordi, combinato con un modello di denoising basato su IA/ML che rimuove quasi tutto il rumore del debayering.
* **Supporto per i target di calibrazione T4P**
* **Elaborazione GPU più veloce in Chloros+, migliore gestione della memoria**

**Correzioni di bug**

* Interfaccia utente (GUI) completamente rinnovata, ora dovrebbe funzionare su tutti i computer Windows.

</details>

<details>

<summary>Versione 1.0.4</summary>

**Data di rilascio: 5 gennaio 2026**

**Nuove funzionalità**

* **Opzione "Immagine/Metadati"**: Aggiunta un'opzione nel File Browser per visualizzare i metadati dell'immagine selezionata in una tabella anziché nella griglia delle immagini
* **Cursore di zoom della griglia delle immagini**: Nuovo cursore nell'interfaccia utente per regolare le dimensioni delle miniature (supporta anche CTRL + rotellina del mouse)
* **Pulsanti di esportazione della griglia di immagini**: pulsanti nella riga superiore per passare dalle miniature in formato JPG alle esportazioni elaborate (Targets, Reflectance, Index, LUT)
* **Scheda Mappa**: nuova mappa 2D interattiva che mostra i marcatori di posizione GPS delle immagini
  * Supporta Google Maps e le tessere cartografiche ESRI (seleziona automaticamente il miglior servizio di tessere in base alla disponibilità del livello di zoom)
  * Anteprima delle miniature al passaggio del mouse sui marcatori della mappa

**Correzioni di bug**

* Migliorato il supporto per l’installazione di Chloros su computer con lingua diversa dall’inglese

</details>

<details>

<summary>Versione 1.0.3</summary>

**Data di rilascio: 20 dicembre 2025**

**Nuove funzionalità**

* Lancio iniziale

**Miglioramenti**

* Lancio iniziale

**Correzioni di bug**

* Lancio iniziale

**Problemi noti**

* Lancio iniziale

</details>

***

## Contratto di licenza**Software proprietario** - Copyright (c) 2026 MAPIR Inc.

È vietato l’uso, la distribuzione o la modifica non autorizzati.

**Versione gratuita**: disponibile per uso personale e commerciale con funzionalità limitate**Chloros+**: licenza in abbonamento per funzionalità avanzate e implementazioni commerciali
