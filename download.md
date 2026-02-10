---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Scarica l&#x27;ultima versione di Chloros per iniziare a elaborare immagini multispettrali.

### Requisiti di sistema

| Requisito          | Minimo                                              | Consigliato                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistema operativo** | Windows 10 (64 bit)                                  | Windows 11 (64 bit)                                  |
| **Processore**        | Intel Core i5 o equivalente                          | Intel Core i7 o superiore                              |
| **Memoria (RAM)**     | 8 GB                                                  | 16 GB o superiore                                         |
| **Scheda grafica**    | Compatibile con DirectX 11                                | GPU NVIDIA con 4 GB+ di VRAM                            |
| **Spazio di archiviazione**          | 6 GB di spazio libero                                       | SSD con 10 GB+ di spazio libero                            |
| **Schermo**          | 1920x1080                                            | 2560x1440 o superiore                                  |
| **Internet**         | Richiesto per l&#x27;attivazione della licenza [opzionale] Chloros+ | Richiesto per l&#x27;attivazione della licenza [opzionale] Chloros+ |

{% hint style="info" %}
**Accelerazione GPU**: gli utenti di Chloros+ con GPU NVIDIA possono utilizzare l&#x27;accelerazione CUDA per un&#x27;elaborazione significativamente più veloce. Gli utenti di Chloros+ ottengono anche l&#x27;elaborazione multi-thread per la massima velocità.
{% endhint %}

***

## Scarica Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Scarica Chloros qui</a>

### Ultima versione stabile

**Chloros Programma di installazione per Windows*** **Versione**: 1.0.5
* **Data di rilascio**: 10 febbraio 2026
* **Dimensione del file (download)**: 1,6 GB
* **Dimensione del file (installato)**: 5,7 GB
* **Tipo di file**: .exe (programma di installazione Windows)

#### **Procedura di installazione:**

1. Scaricare il file `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Fare doppio clic sul programma di installazione per avviare l&#x27;installazione
3. Seguire le istruzioni della procedura guidata di installazione
4. Scegliere la directory di installazione (impostazione predefinita: `C:\Program Files\[USER]\Chloros\`)
5. Completare l&#x27;installazione e avviare Chloros o Chloros CLI
6. Accedere con il proprio [account MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (o continuare con la versione gratuita)

{% hint style="success" %}
Il programma di installazione aggiunge automaticamente `chloros-cli` al PATH di sistema per l&#x27;accesso dalla riga di comando.
{% endhint %}

***

## Risorse aggiuntive

### Python SDK

Per gli sviluppatori e i flussi di lavoro di automazione, installare Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentazione**: [API: Python SDK](api-python-sdk.md)**Requisiti**: Chloros Desktop deve essere installato, Chloros+ è richiesto il login con licenza***

## Cosa è incluso

L&#x27;installazione di Chloros include:

* ✅ **Chloros** - Interfaccia grafica completa (GUI)
* ✅ **Chloros CLI** - Interfaccia a riga di comando (richiede licenza Chloros+)
* ✅ **Chloros SDK** - Python API (richiede licenza Chloros+)
* ✅ **Profili fotocamera** - Modelli fotocamera MAPIR preconfigurati***

## Passa a Chloros+

Sblocca funzionalità avanzate con un abbonamento Chloros+:

* 🚀 **Elaborazione multi-thread** - Elabora le immagini in parallelo
* ⚡ **Accelerazione GPU (CUDA)** - Sfrutta la potenza della GPU NVIDIA
* 💻 **CLI Access** - Automatizza con strumenti da riga di comando
* 🐍 **Python SDK** - Accesso programmatico a API
* 📱 **Dispositivi multipli** - Utilizzo su 2-10+ dispositivi (a seconda del piano)
* **🐻 Metodo avanzato di debayerizzazione sensibile alla texture** - un debayer di alta qualità sensibile ai bordi combinato con un modello di denoising AI/ML che rimuove quasi tutto il rumore di debayerizzazione. 
* 🧮 **Formule personalizzate** - Crea indici multispettrali personalizzati

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Visualizza Chloros+ Piani e prezzi</a></p>***

## Guida all&#x27;installazione

### Risoluzione dei problemi

**L&#x27;installazione non riesce con il messaggio di errore:**

* Assicurati di disporre dei diritti di amministratore
* Disattiva temporaneamente il software antivirus
* Verifica di soddisfare i requisiti minimi di sistema

**L&#x27;applicazione non si avvia:**

* Verificare che Windows 10/11 (64 bit) sia installato
* Aggiornare i driver grafici
* Controllare Windows Event Viewer per i dettagli dell&#x27;errore
* Contattare l&#x27;assistenza con i log degli errori

**Problemi di attivazione della licenza:**

* Assicurarsi che la connessione Internet sia attiva
* Verificare le credenziali su [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verificare che il firewall non stia bloccando Chloros
* Per istruzioni dettagliate, consultare [Chloros+ Login](chloros+-login.md)

### Assistenza

Hai bisogno di aiuto per l&#x27;installazione o la configurazione?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Sito web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentazione**: [Guida introduttiva](./)
* ❓ **FAQ**: [Domande frequenti](faq.md)***

## Registro delle modifiche

<details>

<summary>Versione 1.0.5</summary>

#### **Data di rilascio**: 10 febbraio 2026**Nuove funzionalità*** **Metodo Debayer sensibile alla texture \[Chloros+ Solo] -** Texture Aware utilizza un debayer sensibile ai bordi di alta qualità combinato con un modello di denoising AI/ML che rimuove quasi tutto il rumore di debayering.
* **Supporto per target di calibrazione T4P*** **Elaborazione GPU Chloros+ più veloce, migliore gestione della memoria**

**Correzioni di bug*** Frontend (GUI) completamente nuovo, ora dovrebbe funzionare su tutti i computer Windows.

</details>

<details>

<summary>Versione 1.0.4</summary>

#### **Data di rilascio**: 5 gennaio 2026**Nuove funzionalità*** **Attivazione/disattivazione immagini/metadati**: aggiunta l&#x27;attivazione/disattivazione nel File Browser per visualizzare i metadati dell&#x27;immagine selezionata in una tabella invece che nella griglia delle immagini
* **Cursore di zoom della griglia delle immagini**: nuovo cursore dell&#x27;interfaccia utente per regolare le dimensioni delle miniature (supporta anche CTRL + rotellina del mouse)
* **Pulsanti di esportazione griglia immagini**: pulsanti nella riga superiore per passare dalle miniature JPG alle esportazioni elaborate (destinazioni, riflettanza, indice, LUT)
* **Scheda mappa**: nuova mappa 2D interattiva che mostra i marcatori di posizione GPS delle immagini
  * Supporta Google Maps e le tessere di mappa ESRI (seleziona automaticamente il miglior servizio di tessere in base alla disponibilità del livello di zoom)
  * Anteprima delle miniature al passaggio del mouse sui marcatori della mappa

**Correzioni di bug*** Migliorato il supporto per l&#x27;installazione di Chloros su computer in lingue diverse dall&#x27;inglese

</details>

<details>

<summary>Versione 1.0.3</summary>

#### **Data di rilascio**: 20 dicembre 2025**Nuove funzionalità*** Lancio iniziale

**Miglioramenti*** Lancio iniziale

**Correzioni di bug*** Lancio iniziale

**Problemi noti*** Lancio iniziale

</details>***

## Contratto di licenza**Software proprietario** - Copyright (c) 2026 MAPIR Inc.

È vietato l&#x27;uso, la distribuzione o la modifica non autorizzati.

**Versione gratuita**: disponibile per uso personale e commerciale con limitazioni delle funzionalità.**Chloros+**: licenza basata su abbonamento per funzionalità avanzate e implementazioni commerciali.
