# Installazione su Linux

Chloros è distribuito per Linux sotto forma di pacchetti `.deb` che installano l'interfaccia CLI, il server backend e l'SDK Python per Python (per il sistema `python3`, oltre a un file wheel compatibile con la versione per gli ambienti virtualenvs).

I nomi dei file dei pacchetti indicano la versione e l’architettura: `chloros_1.3.0_amd64.deb` per x86_64, e `chloros_1.3.0_arm64_jp6.deb` per arm64 (NVIDIA Jetson, JetPack 6). Sostituire il file effettivamente scaricato nei comandi riportati di seguito.

{% hint style="info" %}
**Novità nella versione 1.3.0: i programmi vengono forniti già decompressi.** Il backend e l’CLI vengono installati come cartelle, `/usr/lib/chloros/backend/` e `/usr/lib/chloros/cli/`, anziché come singoli file autoestraenti. Non si decomprimono più in una cartella temporanea ad ogni avvio, quindi si avviano molto più velocemente (su x86_64 il backend impiegava circa 100 secondi per la decompressione ad ogni avvio) e non richiedono più diversi GB di spazio libero in `/tmp`. Anche la versione amd64 `.deb` occupa circa la metà dello spazio precedente.
{% endhint %}

***

## Linux amd64 (x86_64)

### Requisiti di sistema

| Requisito | Minimo | Consigliato |
| --- | --- | --- |
| **Distribuzione** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processore** | x86_64 (Intel/AMD) | Intel Core i7 o superiore |
| **Memoria (RAM)** | 8 GB | 16 GB o più |
| **Scheda grafica** | Nessuna (elaborazione tramite CPU) | GPU NVIDIA con almeno 4 GB di VRAM (12 GB o più sbloccano `GPU_PARALLEL`, 7 GB o più mantengono Texture Aware disattivato nel percorso a immagine singola) |
| **Spazio su disco** | 2 GB di spazio libero | SSD con almeno 10 GB di spazio libero |
| **Python** | Python 3.7 o superiore (per l’SDK) | Python 3.10 o superiore |

> **Ubuntu 20.04 e Debian 11 non sono supportati.** L’elenco delle dipendenze di `.deb` è
> derivato da ciò a cui il backend Chloros si collega effettivamente, e ciò include una
> versione di `libc6` più recente rispetto alla glibc 2.31 fornita da Focal e Bullseye; pertanto, `apt` rifiuta
> l’installazione fin dall’inizio, anziché lasciare che fallisca in un secondo momento durante l’esecuzione.

### Installazione

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Utilizzare `apt install ./…`, non `dpkg -i`.** `apt` risolve le dipendenze del pacchetto e installa anche ciò che raccomanda: i requisiti dell’Python SDK, `python3-numpy` e `python3-requests`. `dpkg -i` non installa nessuno dei due. L’CLI e il backend sono autonomi e funzionano senza di essi, ma `import chloros_sdk` ne ha bisogno; l’output dell’installazione indica eventuali componenti mancanti. Mantenere `./`, in modo che `apt` installi il file anziché cercare un pacchetto con quel nome. Il pacchetto dichiara anche `libgl1`, che viene caricato dall’OpenCV in dotazione e potrebbe mancare in un’immagine server minimale.
{% endhint %}

Verificare l’installazione:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Requisiti di sistema

| Requisito | Minimo | Consigliato |
| --- | --- | --- |
| **Piattaforma** | NVIDIA Jetson con JetPack 6 | Jetson Orin NX 16 GB o AGX Orin |
| **JetPack** | JetPack 6.x | Ultima versione di JetPack 6 |
| **Memoria (RAM)** | 8 GB (condivisa tra GPU e CPU) | 16 GB+ condivisi (12 GB+ è la soglia per l’elaborazione parallela) |
| **Spazio di archiviazione** | 2 GB di spazio libero | SSD NVMe con almeno 10 GB di spazio libero |
| **Python** | Python 3.7+ (per l’SDK) | Python 3.10+ |

### Installazione

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Stesso layout dell’amd64 `.deb` (Python SDK, buffer di ricezione e servizi backend inclusi), per Jetson Orin / Orin NX / Orin Nano su **JetPack 6** (L4T R36). Il suo PyTorch è la build CUDA di JetPack 6 per Orin, quindi l’elaborazione GPU, compreso il debayer Texture Aware, viene eseguita sulla GPU del Jetson. Il pacchetto non include CUDA in sé: utilizza le librerie CUDA e cuDNN installate da JetPack 6 (`sudo apt install nvidia-jetpack` su un'immagine che ne è sprovvista). Laddove non sia disponibile una GPU CUDA, l’elaborazione viene eseguita sulla CPU e `--debayer texture-aware` utilizza il debayer Standard (il backend registra una riga `[DEBAYER] … needs a CUDA GPU …` che ne spiega il motivo). Per informazioni sul comportamento di Jetson in termini di memoria, temperatura e implementazione sul campo, consultare la [Guida NVIDIA Jetson](nvidia-jetson-guide.md).***

## Installazione dell’Python SDK (tutte le distribuzioni Linux)**`.deb` installa automaticamente l’SDK.** Entrambi i pacchetti (amd64 e Jetson) lo installano per il sistema `python3`, quindi `import chloros_sdk` funziona subito dopo l’installazione, senza bisogno di pip né di connessione di rete. I suoi pacchetti (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` e i caricatori di progetto) vengono inseriti in `/usr/lib/python3/dist-packages`, di proprietà del pacchetto: un aggiornamento li sostituisce, mentre la rimozione del pacchetto li elimina. La versione corrisponde sempre alla CLI e al backend che avete installato.

I requisiti propri dell’SDK, `python3-numpy` e `python3-requests`, sono **consigliati, non obbligatori** dal pacchetto: `sudo apt install ./chloros_<version>_<arch>.deb` li installa, mentre `dpkg -i` no. L’output dell’installazione riporta la versione dell’SDK che `python3` importa, eventuali requisiti mancanti, e qualsiasi `chloros_sdk` presente altrove nel percorso di `python3` che sovrascriva la copia del pacchetto (come ad esempio una versione installata in precedenza con `pip install --user`).**In un virtualenv**, che non vede i pacchetti di sistema, installare al suo interno il file wheel in dotazione (completamente offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Da PyPI** — solo per una macchina su cui non è installato il pacchetto Chloros, il cui SDK gestisce un backend su un’altra macchina (vedi [Modalità backend remoto](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Componenti aggiuntivi opzionali, per un'installazione tramite pip (in un virtualenv o da PyPI):

| Componente | Comando | Aggiunge |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` per lo streaming in tempo reale dell’avanzamento |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` per il trasporto BLE (DAQ-M) |

Verifica l’SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
L’SDK Python comunica con il backend che `.deb` installa tramite un’HTTP API locale (`http://127.0.0.1:5000`). `ChlorosLocal` e gli helper di connessione lo avviano automaticamente quando necessario; l’`connect_all()` di un progetto salvato non lo fa mai, quindi su una macchina senza interfaccia grafica è necessario eseguire il backend come [servizio](#always-on-ptp-for-headless-hosts). Utilizzare sempre l’indirizzo IPv4 letterale anziché `localhost` — `localhost` può risolversi in `::1` e comportare un costo di circa due secondi per ogni richiesta.
{% endhint %}

***

## Configurazione iniziale

### 1. Accedi

L'accesso alla CLI e all'SDK richiede un piano a pagamento Chloros+ (**Copper** o superiore), applicato a livello di server: un chiamante disconnesso riceve `401 AUTH_REQUIRED`, mentre un chiamante con piano gratuito (Iron) riceve `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Le credenziali vengono memorizzate nella cache in `~/.chloros/user_session.json`.

{% hint style="info" %}
**Gli aggiornamenti mantengono i dati di accesso; la disinstallazione li cancella.** L’installazione di una versione più recente di `.deb` su un’installazione esistente (anche tramite `chloros-cli update`) mantiene `~/.chloros/user_session.json` e la licenza memorizzata nella cache, quindi i processi CLI/SDK senza interfaccia grafica continuano a funzionare. La rimozione del pacchetto (`sudo apt remove chloros`) cancella i dati di accesso salvati e la cache della licenza per ogni utente sul computer, pertanto un'installazione ex novo richiede nuovamente `chloros-cli login`.
{% endhint %}

### 2. Verifica lo stato della tua licenza

```bash
chloros-cli status
```

`chloros-cli status` funziona su qualsiasi livello (compreso quello gratuito), quindi puoi sempre capire perché l’accesso è o non è disponibile.

### 3. Eseguire la diagnostica di sistema

```bash
chloros-cli selftest
```

Vengono eseguiti sette controlli in sequenza e il comando restituisce un codice di uscita diverso da zero se uno qualsiasi di essi fallisce:

| # | Controllo | Cosa verifica |
| --- | --- | --- |
| 1 | **Versione** | L’CLI riporta la propria versione (`v1.3.0`). |
| 2 | **Porta disponibile** | La porta 5000 è libera, *oppure* ha già ricevuto risposta da un backend Chloros funzionante (il che conta come superamento del test). |
| 3 | **Avvio del backend** | Il binario del backend viene avviato. |
| 4 | **Test API (`/api/test`)** | Il backend risponde con `status: ok`. |
| 5 | **Informazioni di sistema** | Visualizza `GPU: <name>, CUDA: <bool>, PyTorch: <version>` da `/api/system-info`. |
| 6 | **Modelli del denoiser** | Trova i modelli `*.pth.enc` (su Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Denoiser**| Texture Aware è effettivamente utilizzabile — richiede CUDA**e** almeno un file di modello. |

L'esecuzione termina con `N/7 checks passed`, elencando eventuali errori per nome.

### 4. Elabora il tuo primo set di dati

```bash
chloros-cli process ~/datasets/flight001
```

***

## File e directory

### Per utente

Chloros conserva le proprie credenziali e la configurazione della CLI in un’unica directory multipiattaforma, **`~/.chloros/`** (su Windows, `%USERPROFILE%\.chloros\`). Due cache specifiche per Linux seguono invece le convenzioni XDG: queste rispettano `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` quando impostate.

| Percorso | Scopo |
| --- | --- |
| `~/.chloros/user_session.json` | Cache della sessione di accesso creata da `chloros-cli login` (mantenuta durante gli aggiornamenti del pacchetto; cancellata quando il pacchetto viene rimosso) |
| `~/.chloros/working_directory.txt` | Sovrascrittura della cartella di progetto predefinita (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferenza di lingua della CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Impostazione della lingua condivisa con l'interfaccia grafica di Windows: un valore `language` qui ha la priorità su `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache di un'ora per il controllo degli aggiornamenti all'avvio di Linux/Jetson |
| `~/.chloros/backend.log` | Log del backend quando quest'ultimo è stato avviato dalla CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Pacchetti di calibrazione LATTICE per singola telecamera memorizzati nella cache, identificati dal numero di serie e dall’hash del bundle |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Sovrascritture opzionali da parte dell’utente per i profili di correzione del cap DAQ |
| `~/.config/chloros/system_config.json` | Profilo hardware memorizzato nella cache da Dynamic Compute Adaptation — eliminarlo per forzare un nuovo rilevamento dell’hardware |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Log del server backend, un file per ogni avvio |
| `~/Documents/MAPIR/Chloros Projects/` | Cartella predefinita del progetto quando non è impostata alcuna sovrascrittura |

### A livello di sistema

| Percorso | Scopo |
| --- | --- |
| `/usr/bin/chloros-cli` | Script wrapper — imposta `LD_LIBRARY_PATH` per le librerie native incluse nel pacchetto (e da `TMPDIR` a `/mnt/ssd/tmp` quando tale cartella esiste), quindi esegue il binario effettivo |
| `/usr/bin/chloros-backend` | Script wrapper — lo stesso per il backend, che contrassegna come build di produzione installata |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | L’CLI e il backend compilati, installati senza decompressione (novità della versione 1.3.0); i programmi sono `cli/chloros-cli` e `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Script di avvio di due righe che lanciano i programmi effettivi sopra indicati — mantenuti in questi percorsi perché l’SDK, la CLI e gli script precedenti li cercano qui |
| `/usr/lib/chloros/arena_runtime/` | Runtime dell’SDK di Arena richiesto dalle telecamere LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Modelli di denoising crittografati utilizzati dal debayer Texture Aware |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (e `lattice_sdk`, `lattice_cam`, `arena_api`, i caricatori di progetto) | L'SDK Python, installato per il sistema `python3` e di proprietà del pacchetto |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Wheel dell'Python SDK corrispondente esattamente a questa build, per un virtualenv |
| `/usr/lib/chloros/exiftool` | Il programma di avvio ExifTool integrato (collegato tramite collegamento simbolico a `/usr/local/bin/exiftool` solo se non esiste un exiftool di sistema). A partire dalla versione 1.3.0, lo script ExifTool viene fornito con la propria libreria Perl in `/usr/lib/chloros/exiftool-dist/`, quindi funziona senza il pacchetto di sistema `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Configurazione del canale di aggiornamento letta da `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Imposta `net.ipv4.ip_unprivileged_port_start = 319` in modo che il backend possa associare le porte PTP senza privilegi di root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Aumenta i buffer di ricezione dei socket del kernel (`net.core.rmem_max`, `net.core.rmem_default`) a 32 MB per i flussi delle telecamere LATTICE e li applica al momento dell’installazione (un valore già impostato su un valore più alto viene mantenuto invariato). Con le impostazioni predefinite del kernel, i pacchetti delle telecamere vengono scartati nel kernel e un array perde fotogrammi e frequenza dei fotogrammi; `chloros-cli lattice network` segnala i buffer quando sono bassi. Vedi [Host Linux — buffer di ricezione dei socket](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Indica al caricatore dinamico `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Concede all’utente connesso l’accesso al bridge seriale USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Servizio backend sempre attivo (opt-in) che esegue il backend come utente specificato (installato, **non abilitato**) — quello da abilitare, vedi [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Lo stesso servizio che esegue il backend come **root**(installato,**non abilitato**), mantenuto affinché una macchina che lo ha abilitato continui a funzionare |
| `/var/log/chloros/` | Il log dettagliato del servizio root, leggibile dal gruppo `adm` (cancellato quando il pacchetto viene rimosso) |
| `/usr/share/applications/chloros-cli.desktop` | Voce del menu delle applicazioni «Chloros CLI» che apre un terminale |

**Firewall (novità della versione 1.3.0).** Se `ufw` o `firewalld` sono attivi al momento dell’installazione, il pacchetto apre le porte UDP `5002` (dati DAQ-E), `5353` (rilevamento mDNS), `319` e `320` (PTP), e rimuove tali regole quando il pacchetto viene disinstallato. Se è installato un firewall ma non è attivo, il pacchetto non aggiunge nulla e visualizza le porte da consentire nel caso in cui lo si abiliti in seguito. Non apre alcuna porta TCP: un aggiornamento del firmware DAQ-E, in cui il sensore si riconnette a questo computer tramite TCP per acquisire l’immagine, fallisce con l’errore `… device never connected back on TCP <port> …` se un firewall attivo blocca tale connessione.

## Posizione dell’eseguibile del backend

La CLI e l’SDK rilevano automaticamente il backend:

| Componente | Percorso |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (un programma di avvio per `/usr/lib/chloros/backend/chloros-backend`) |

È possibile sovrascrivere il percorso del backend utilizzando il flag CLI `--backend-exe` o il parametro del costruttore SDK `backend_exe`, e la porta con `--port` (impostazione predefinita `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` punta alle famiglie di comandi **`lattice`**,**`project`**e**`daq pool-*`** a un backend remoto. I comandi principali (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) lo ignorano deliberatamente e puntano sempre a `http://127.0.0.1:<port>`.
{% endhint %}

***

## Telecamere LATTICE e sensori di luce DAQ su Linux

Tutte le famiglie di comandi live-hardware funzionano su Linux (amd64 e Jetson):

* **`chloros-cli lattice`** — rileva, si connette, configura ed effettua l’acquisizione da telecamere LATTICE e array sincronizzati. `.deb` include il runtime dell’SDK di Arena necessario e lo registra con il caricatore dinamico.
* **`chloros-cli daq pool-*`** — consente di collegare i sensori di luce DAQ-U/M/E tramite il pool di backend, trasmettere in streaming spettri calibrati, registrare file `.daq` e scambiare profili CAP; per un DAQ-E, permette inoltre di modificarne le impostazioni di rete, riavviarlo e aggiornarne il firmware. `chloros-cli daq discover` esegue la ricerca dei sensori collegabili, mentre `chloros-cli daq --help` elenca i sottocomandi `pool-*`.
* **`chloros-cli project`** — esegue un progetto salvato (con le relative telecamere, sensori e impostazioni di elaborazione) in modalità headless.
* **`chloros-cli time-sync`** — ispeziona il grandmaster PTP su cui gira il backend Chloros per le telecamere LATTICE e i sensori DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` è un prerequisito per `pool-latest`, `pool-stream`, `pool-record` e `pool-set-cap`; `pool-list` mostra gli ID attualmente presenti nel pool.

{% hint style="info" %}
**`--eth-host` salta la fase di individuazione.** `pool-connect --eth` individua un DAQ-E tramite mDNS, con un fallback ARP che funziona da una cache ARP vuota; passando l’IP o il nome host del sensore con `--eth-host` ci si connette direttamente ad esso.
{% endhint %}

**I permessi seriali del DAQ-U** sono gestiti dalla regola udev installata (`uaccess` + gruppo `dialout`). Se un sensore già collegato rimane inaccessibile, ricaricare le regole o ricollegarlo:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Consultare il [riferimento CLI](../CLI.md) per l’insieme completo dei comandi.

### PTP sempre attivo per host senza monitor

Il pacchetto installa due servizi systemd per il backend, entrambi disabilitati. Su un Jetson senza monitor o su un server che deve mantenere attiva in modo continuo la sincronizzazione temporale PTP per i sensori DAQ-E e le telecamere LATTICE, oppure dove gli script SDK si collegano a progetti salvati (`connect_all()` non avvia mai un backend), abilitare il servizio **come utente**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` esegue il backend come quell’utente: le acquisizioni che scrive appartengono a quell’utente, il suo log dettagliato si trova nel file `~/.cache/chloros/logs` e utilizza il file `chloros-cli login` (`~/.chloros`), quindi accedi prima una volta con quell’utente. L’installazione del pacchetto visualizza questo comando con il tuo nome utente già inserito.

Senza un servizio, il backend (e con esso il grandmaster PTP) funziona solo finché qualcosa lo tiene in esecuzione — un comando `chloros-cli` o uno script SDK.

L’unità associa il backend alle impostazioni di ambiente `127.0.0.1:5000` (impostazioni ambientali `CHLOROS_HOST` / `CHLOROS_PORT` all’interno dell’unità; sovrascrivibili con `sudo systemctl edit chloros-backend@.service`) e lo riavvia in caso di errore dopo 5 secondi.

**Il servizio root.**`chloros-backend.service` esegue il backend come**root**. Rimane comunque installato in modo che una macchina su cui è stato abilitato continui a funzionare, ma tutto ciò che il backend root scrive appartiene a root, quindi un utente non può eliminare le proprie acquisizioni; il suo log si trova in `/var/log/chloros`, leggibile dal gruppo `adm`. Per passare al proprio utente:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Il backend di un servizio è condiviso e non viene mai arrestato da un client.** `chloros-cli` e gli script SDK lo utilizzano così come lo trovano: `process` viene eseguito su di esso, caricando il proprio progetto al posto dell’ultimo. Laddove la CLI, altrimenti, riavvierebbe il backend (`--restart`, o un `process` la cui importazione non ha trovato nulla), essa indica il nome dell’unità, visualizza `sudo systemctl restart <unit>` e termina `1`, poiché un backend arrestato in questo modo rimarrebbe inattivo. Riavviatelo manualmente con `sudo systemctl restart chloros-backend@$USER.service`. Il backend di un servizio non si arresta mai autonomamente in stato di inattività, e `chloros-cli backend stop` lo rifiuta, indicando il comando `systemctl stop`.**Come PTP ottiene le proprie porte.** PTP utilizza le porte UDP 319/320, entrambe al di sotto della soglia normale di 1024 porte privilegiate. Il pacchetto `postinst` scrive `/etc/sysctl.d/60-chloros-ptp.conf` con `net.ipv4.ip_unprivileged_port_start = 319`, il che consente al backend di associarle mentre viene eseguito come utente. A partire dalla versione 1.3.0 il pacchetto non imposta alcuna capacità di file (`setcap`) sul backend, e `libcap2-bin` non è più una dipendenza: sul backend decompresso una capacità di file ne impedirebbe l’avvio, quindi non aggiungerne una manualmente.***

## Esempi di script Bash

{% hint style="info" %}
**Codici di uscita compatibili con gli script.**`chloros-cli process` restituisce `0` in caso di successo e**un valore diverso da zero in caso di errore — inclusa un'esecuzione che ha richiesto prodotti immagine ma non ne ha generati** (visualizza `Processing finished but wrote no image products.` e indica il nome della cartella del progetto e le cause più comuni). Le esecuzioni riuscite segnalano quanti prodotti immagine sono stati generati (`Image products written: N`, più `(plus M clamp masks)` quando sono state generate delle maschere). Codici di uscita: `0` successo, `1` errore, `2` errore di argomento, `130` interruzione.
{% endhint %}

### Elaborazione di più set di dati

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Elaborazione con impostazioni personalizzate

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

I valori validi di `--format` sono esattamente quattro e contengono spazi: inserirli sempre tra virgolette:

| Valore `--format` | Cartella di output |
| --- | --- |
| `TIFF (16-bit)` *(impostazione predefinita)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` accetta `standard` (impostazione predefinita) o `texture-aware` (Chloros+).

### Elaborazione automatizzata con Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Esempio di SDK per Python

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Risoluzione dei problemi

### CLI non trovata dopo l’installazione

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Autorizzazione negata

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Il backend non si avvia dopo l’aggiunta delle funzionalità dei file

Non eseguire `setcap` sul backend. A partire dalla versione 1.3.0 il pacchetto non imposta alcuna funzionalità di file, e le porte 319/320 di PTP derivano dall’impostazione `sysctl` descritta in [Come PTP ottiene le sue porte](#always-on-ptp-for-headless-hosts). Una capacità di file mette il backend decompresso in una modalità di esecuzione sicura in cui non riesce a trovare le proprie librerie, quindi non si avvia. Se ne è stata aggiunta una, rimuoverla:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP non si avvia / Non riesce a legarsi alla porta 319

Verifica che il limite minimo delle porte non privilegiate sia stato abbassato e, se non lo è stato, riapplicalo per l’avvio corrente:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Quindi controlla il grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### I comandi `lattice` falliscono con il messaggio "Driver della telecamera LATTICE non trovati"

Il backend non è riuscito a caricare il runtime della telecamera LATTICE con cui è stato installato. Ogni comando `lattice` viene eseguito sul backend; la CLI stessa non contiene driver per telecamere. Reinstallare Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): il comando `.deb` installa il runtime accanto al backend, in `/usr/lib/chloros/arena_runtime`, e lo registra con il caricatore dinamico. Per verificare tale registrazione:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
```

### Avvio del backend non riuscito

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

I log del backend relativi all’avvio fallito si trovano in `~/.cache/chloros/logs/`. Se la porta 5000 appartiene a un servizio del backend (`systemctl status chloros-backend@$USER.service`), riavviare il servizio anziché terminarne il processo.

### CUDA non rilevato

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` riporta la stessa informazione in una sola riga: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Librerie condivise mancanti

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Avvio lento sui sistemi con scheda SD

A partire dalla versione 1.3.0, il backend e l’CLI vengono installati già decompressi, quindi non si estraggono più in una directory temporanea ad ogni avvio. I file temporanei vengono salvati automaticamente in `/mnt/ssd/tmp` se tale cartella esiste; in caso contrario, è possibile indirizzarli verso un filesystem veloce:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Aggiornamento di Chloros su Linux

Il comando `update` è disponibile solo su Linux/Jetson. Verifica la versione pubblicata nel canale di aggiornamento configurato in `/etc/chloros/update.conf` e propone di scaricare e installare la versione corrispondente in `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Su Linux/Jetson, la CLI esegue inoltre un controllo degli aggiornamenti non bloccante ad ogni avvio (il risultato viene memorizzato nella cache per un’ora in `~/.chloros/update_cache.json`) e visualizza `Update available: vX.Y.Z` quando è disponibile una versione più recente. Le impostazioni, i progetti e i dati di accesso rimangono inalterati dopo l’aggiornamento.

L’aggiornamento del pacchetto arresta tutti i servizi di backend durante l’installazione, quindi uno script in esecuzione perde le sue telecamere e i suoi sensori, per poi riavviare ciascuno di quelli abilitati (uno avviato manualmente ma mai abilitato rimane arrestato, come dopo un riavvio).

## Disinstallazione

```bash
sudo apt remove chloros
```

La rimozione arresta e disabilita tutti i servizi di backend, ripristina il limite predefinito delle porte non privilegiate (1024), rimuove il file di configurazione del buffer di ricezione (i valori correnti rimangono fino al riavvio), il collegamento simbolico a exiftool incluso nel pacchetto, la configurazione del caricatore Arena, le regole del firewall aggiunte dal pacchetto e l’SDK Python installato, e cancella le credenziali memorizzate nella cache. I vostri progetti e i file di dati `~/.chloros/` rimangono intatti. `sudo apt purge chloros` elimina anche `/var/log/chloros`.

***

## Passi successivi

* [Guida NVIDIA Jetson](nvidia-jetson-guide.md) — Ottimizzazione e distribuzione specifiche per Jetson
* [CLI: riga di comando](../CLI.md) — la guida alla CLI
* [API: Python SDK](../api-python-sdk.md) — la guida all’SDK
* [Riferimento CLI](../reference/cli-reference.md) e [Riferimento SDK](../reference/sdk-reference.md) — elenchi completi dei comandi e delle API per la versione 1.3.0
* [Adattamento dinamico delle risorse di calcolo](../processing-architecture/dynamic-compute-adaptation.md) — come Chloros. si adatta al tuo hardware
