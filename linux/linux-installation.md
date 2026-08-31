# Installazione di Linux

Chloros viene distribuito per Linux sotto forma di pacchetti `.deb` che installano CLI e il server backend. Python SDK è un pacchetto pip separato (incluso anche all’interno di `.deb` come wheel con versione corrispondente).

I nomi dei file dei pacchetti riportano la versione e l’architettura: `chloros_1.2.0_amd64.deb` per x86_64 e `chloros_1.2.0_arm64_jp6.deb` per le build JetPack 6 Jetson. Sostituisci il file che hai effettivamente scaricato nei comandi riportati di seguito.

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
| **Python** | Python 3.7+ (per SDK) | Python 3.10+ |

> **Ubuntu 20.04 e Debian 11 non sono supportati.** L’elenco delle dipendenze di `.deb` è
> derivato da ciò a cui il backend Chloros effettua effettivamente il collegamento, e ciò include
> `libc6 (>= 2.34)`. Sia Focal che Bullseye includono glibc 2.31, quindi `apt` rifiuta
> l’installazione fin dall’inizio piuttosto che lasciarla fallire in un secondo momento durante l’esecuzione.

### Installazione

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` non risolve le dipendenze. Se segnala pacchetti mancanti, `sudo apt-get install -f` (o `sudo apt --fix-broken install`) completa l’installazione: si tratta del percorso normale, non di un errore.
{% endhint %}

Verifica l’installazione:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Requisiti di sistema

| Requisito | Minimo | Consigliato |
| --- | --- | --- |
| **Piattaforma** | NVIDIA Jetson con JetPack 6 | Jetson Orin NX da 16 GB o AGX Orin |
| **JetPack** | JetPack 6.x | Ultima versione di JetPack 6 |
| **Memoria (RAM)** | 8 GB (condivisa tra GPU e CPU) | 16 GB+ condivisa (12 GB+ è la soglia per i worker GPU paralleli) |
| **Spazio di archiviazione** | 2 GB di spazio libero | SSD NVMe con 10 GB+ di spazio libero |
| **Python** | Python 3.7+ (per SDK) | Python 3.10+ |

### Installazione

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Stesso layout dell’amd64 `.deb`, con una build CUDA ottimizzata per Jetson Orin / Orin NX / Orin Nano. Per informazioni sul comportamento di Jetson in termini di memoria, temperatura e implementazione sul campo, consultare la [Guida NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Installazione di Python e SDK (tutti i modelli Linux)

SDK è un client puramente Python HTTP per il backend, quindi lo stesso pacchetto funziona su amd64 e arm64. Due fonti:**Da PyPI** — la versione stabile pubblicata:

```bash
pip install chloros-sdk
```

**Dal file wheel in dotazione** — compatibilità garantita con il backend CLI appena installato (utilizzarlo quando la versione di `.deb` è più recente di quella su PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Le distribuzioni PEP 668** (Ubuntu 23.10+, Debian 12+) non consentono installazioni pip a livello di sistema. Utilizza `pip install --user …`, un ambiente virtuale oppure `sudo pip install --break-system-packages …`. Il programma di installazione del pacchetto non installa mai automaticamente SDK nel tuo sistema Python: questa scelta spetta a te.
{% endhint %}

Extra opzionali:

| Extra | Comando | Aggiunge |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` per lo streaming in tempo reale dello stato di avanzamento |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` per il trasporto BLE (DAQ-M) |

Verifica l’SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` installa Chloros, CLI e il backend. Python SDK comunica con quel backend tramite una rete locale HTTP API (`http://127.0.0.1:5000`) e lo avvia automaticamente quando necessario. Utilizzare sempre l’indirizzo IPv4 letterale anziché `localhost` — `localhost` può risolversi in `::1` e comportare un tempo di circa due secondi per ogni richiesta.
{% endhint %}

***

## Configurazione iniziale

### 1. Accedi

L&#x27;accesso a CLI e SDK richiede un piano a pagamento Chloros+ (**Copper** o superiore), applicato a livello di server: un utente disconnesso riceve `401 AUTH_REQUIRED`, mentre un utente con piano gratuito (Iron) riceve `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Le credenziali vengono memorizzate nella cache in `~/.chloros/user_session.json`.

{% hint style="warning" %}
**È necessario effettuare nuovamente l&#x27;accesso dopo ogni installazione o aggiornamento.** Lo script `prerm` del pacchetto cancella intenzionalmente `~/.chloros/user_session.json` e la licenza memorizzata nella cache per ogni utente presente sul computer, in modo che una nuova build convalidi sempre la licenza invece di fare affidamento su una cache non aggiornata.
{% endhint %}

### 2. Verifica lo stato della licenza

```bash
chloros-cli status
```

`chloros-cli status` funziona su qualsiasi livello (compreso quello gratuito), quindi è sempre possibile capire perché l’accesso è o non è disponibile.

### 3. Eseguire la diagnostica di sistema

```bash
chloros-cli selftest
```

Vengono eseguiti sette controlli in sequenza e il comando restituisce un valore diverso da zero se uno qualsiasi di essi fallisce:

| # | Controllo | Cosa verifica |
| --- | --- | --- |
| 1 | **Versione** | CLI riporta la propria versione (`v1.2.0`). |
| 2 | **Porta disponibile** | La porta 5000 è libera, *oppure* ha già ricevuto risposta da un backend Chloros funzionante (il che conta come superamento del test). |
| 3 | **Avvio del backend** | Il binario del backend viene avviato. |
| 4 | **Test di API (`/api/test`)** | Il backend risponde `status: ok`. |
| 5 | **Informazioni di sistema** | Visualizza `GPU: <name>, CUDA: <bool>, PyTorch: <version>` da `/api/system-info`. |
| 6 | **Modelli Denoiser** | Trova i modelli `*.pth.enc` (su Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Denoiser**| La funzione &quot;Texture Aware&quot; è effettivamente utilizzabile — richiede CUDA**e** almeno un file di modello. |

L’esecuzione termina con `N/7 checks passed`, elencando eventuali errori per nome.

### 4. Elaborare il primo set di dati

```bash
chloros-cli process ~/datasets/flight001
```

***

## File e directory

### Per utente

Chloros conserva le proprie credenziali e la configurazione di CLI in un’unica directory multipiattaforma, **`~/.chloros/`** (su Windows, `%USERPROFILE%\.chloros\`). Due cache specifiche di Linux seguono invece le convenzioni XDG: queste rispettano `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` quando impostate.

| Percorso | Scopo |
| --- | --- |
| `~/.chloros/user_session.json` | Cache della sessione di accesso creata da `chloros-cli login` (cancellata ad ogni installazione/aggiornamento di un pacchetto) |
| `~/.chloros/working_directory.txt` | Sovrascrittura predefinita della cartella del progetto (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferenza linguistica di CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Impostazione della lingua condivisa con l&#x27;interfaccia grafica Windows — un valore `language` qui ha la priorità su `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache di un’ora per il controllo degli aggiornamenti all’avvio di Linux/Jetson |
| `~/.chloros/backend.log` | Log del backend quando il backend è stato avviato da CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Pacchetti di calibrazione LATTICE per singola telecamera memorizzati nella cache, identificati tramite numero di serie e hash del bundle |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Sovrascritture opzionali da parte dell’utente per i profili di correzione cap del DAQ |
| `~/.config/chloros/system_config.json` | Profilo hardware memorizzato nella cache proveniente dal Dynamic Compute Adaptation — eliminarlo per forzare un nuovo rilevamento dell&#x27;hardware |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Log del server backend, un file per ogni avvio |
| `~/Chloros Projects/` | Cartella predefinita del progetto quando non è impostata alcuna sovrascrittura |

### A livello di sistema

| Percorso | Scopo |
| --- | --- |
| `/usr/bin/chloros-cli` | Script wrapper: imposta `LD_LIBRARY_PATH` per le librerie native incluse nel pacchetto, quindi esegue il binario effettivo |
| `/usr/bin/chloros-backend` | Script wrapper — come sopra, più `CHLOROS_PRODUCTION=1` in modo che il gate di autenticazione del backend non possa mai disattivarsi silenziosamente |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | I binari compilati |
| `/usr/lib/chloros/arena_runtime/` | Runtime Arena SDK richiesto dalle telecamere LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Modelli di denoising crittografati utilizzati dal debayer Texture Aware |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK: wheel corrispondente a questa esatta build |
| `/usr/lib/chloros/exiftool` | exiftool integrato (collegato tramite collegamento simbolico a `/usr/local/bin/exiftool` solo se non esiste un exiftool di sistema) |
| `/etc/chloros/update.conf` | Configurazione del canale di aggiornamento letta da `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Imposta `net.ipv4.ip_unprivileged_port_start = 319` in modo che il backend possa associare le porte PTP senza privilegi di root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Indirizza il caricatore dinamico verso `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Concede all&#x27;utente connesso l&#x27;accesso al bridge seriale USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Attiva il servizio backend sempre attivo (installato, **non abilitato**) |
| `/usr/share/applications/chloros-cli.desktop` | Voce del menu dell’applicazione &quot;Chloros CLI&quot; che apre un terminale |

## Percorso dell’eseguibile del backend

CLI e SDK rilevano automaticamente il backend:

| Componente | Percorso |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

È possibile sovrascrivere il percorso del backend utilizzando il flag `--backend-exe` CLI o il parametro del costruttore `backend_exe` SDK, e la porta con `--port` (impostazione predefinita `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` punta alle famiglie di comandi **`lattice`**,**`project`**e**`daq pool-*`** in un backend remoto. I comandi principali (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) lo ignorano deliberatamente e puntano sempre a `http://127.0.0.1:<port>`.
{% endhint %}

***

## Telecamere LATTICE e sensori di luce DAQ su Linux

Tutte le famiglie di comandi live-hardware funzionano su Linux (amd64 e Jetson):

* **`chloros-cli lattice`** — rileva, si connette, configura ed acquisisce dati da telecamere LATTICE e array sincronizzati. `.deb` include il runtime Arena SDK necessario e lo registra con il caricatore dinamico.
* **`chloros-cli daq pool-*`** — si connette ai sensori di luce DAQ-U/M/E tramite il pool di backend, trasmetti in streaming spettri calibrati e registra file `.daq`. Il file compilato CLI include solo la famiglia `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
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

`--sensor-id` è richiesto da `pool-latest`, `pool-stream`, `pool-record` e `pool-set-cap`; `pool-list` mostra gli ID attualmente presenti nel pool.

{% hint style="info" %}
**Preferire `--eth-host` per la prima connessione DAQ-E su una macchina multi-homed.** Il rilevamento automatico esegue una scansione di mDNS e potrebbe non individuare l’interfaccia del sensore a causa di una cache ARP vuota; pertanto, il primo `pool-connect --eth` dopo l’avvio potrebbe fallire anche se il sensore è perfettamente funzionante. Passando l’IP o il nome host del sensore si salta completamente il rilevamento.
{% endhint %}

**I permessi seriali DAQ-U** sono gestiti dalla regola udev installata (`uaccess` + gruppo `dialout`). Se un sensore già collegato rimane inaccessibile, ricaricare le regole o ricollegarlo:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Consultare la [guida di riferimento di CLI](../CLI.md) per l’insieme completo dei comandi.

### PTP sempre attivo per host senza interfaccia grafica

Alla prima installazione, viene generata l’unità systemd `chloros-backend.service`, ma **non è abilitata**. Su un Jetson o un server senza monitor che deve mantenere attiva in modo continuo la sincronizzazione temporale PTP per i sensori DAQ-E e le telecamere LATTICE, abilitarla:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Senza di essa, il PTP funziona solo mentre è in esecuzione il backend Chloros — ovvero, durante una sessione attiva di CLI/SDK.

L’unità associa il backend a `127.0.0.1:5000` (impostazioni ambientali `CHLOROS_HOST` / `CHLOROS_PORT` all’interno dell’unità; sovrascrivere con `sudo systemctl edit chloros-backend.service`) e lo riavvia in caso di errore dopo 5 secondi.

**Come PTP ottiene le sue porte.** PTP utilizza le porte UDP 319/320, entrambe al di sotto della soglia normale di 1024 porte privilegiate. Il pacchetto `postinst` scrive `/etc/sysctl.d/60-chloros-ptp.conf` con `net.ipv4.ip_unprivileged_port_start = 319`, il che consente al backend di associarle mentre viene eseguito come utente. Inoltre, applica `setcap cap_net_bind_service,cap_net_raw=+ep` al binario del backend come misura di sicurezza aggiuntiva — ecco perché `libcap2-bin` è una dipendenza dichiarata del pacchetto.***

## Esempi di scripting Bash

{% hint style="info" %}
**Codici di uscita compatibili con gli script.**`chloros-cli process` restituisce `0` in caso di successo e**un valore diverso da zero in caso di errore — compresa un&#x27;esecuzione che ha richiesto prodotti immagine ma non ne ha generati** (visualizza `Processing finished but wrote no image products.` e indica la cartella del progetto e le cause più comuni). Le esecuzioni riuscite segnalano quanti prodotti immagine sono stati generati (`Image products written: N`). Codici di uscita: `0` successo, `1` errore, `2` errore di argomento, `130` interruzione.
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
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

I valori validi di `--format` sono esattamente quattro e contengono spazi: inserirli sempre tra virgolette:

| Valore `--format` | Cartella di output |
| --- | --- |
| `TIFF (16-bit)` *(predefinito)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` accetta `standard` (impostazione predefinita) o `texture-aware` (Chloros+).

### Elaborazione automatizzata con Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Esempio di Python e SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Risoluzione dei problemi

### CLI non trovato dopo l’installazione

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
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### &quot;setcap failed&quot; durante l’installazione

`.deb` applica `cap_net_bind_service` a `/usr/lib/chloros/chloros-backend` in modo che possa associare le porte PTP 319/320 senza privilegi di root. Se al momento dell’installazione mancava `libcap2-bin`, la chiamata viene saltata. Installarlo e reinstallare il pacchetto:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP non si avvia / Impossibile associare la porta 319

Verificare che la soglia delle porte senza privilegi sia stata abbassata e, in caso contrario, riapplicarla per l’avvio corrente:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Quindi controllare il grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### &quot;Driver della telecamera LATTICE non trovati&quot;

Il runtime Arena SDK non viene risolto. Verificare che la configurazione del caricatore scritta dal pacchetto sia presente e aggiornata:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
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

I log del backend relativi all’avvio non riuscito si trovano in `~/.cache/chloros/logs/`.

### CUDA non rilevato

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` riporta la stessa cosa in una riga: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Librerie condivise mancanti

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Avvio lento sui sistemi con scheda SD

I file binari compilati si estraggono automaticamente in una directory temporanea ad ogni avvio. Se `/mnt/ssd/tmp` esiste, Chloros lo utilizza automaticamente; in caso contrario, impostare `TMPDIR` su un filesystem veloce:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Aggiornamento di Chloros su Linux

Il comando `update` è disponibile solo su Linux/Jetson. Verifica la versione pubblicata nel canale di aggiornamento configurato su `/etc/chloros/update.conf` e propone di scaricare e installare la versione corrispondente `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Su Linux/Jetson, il comando CLI esegue inoltre un controllo degli aggiornamenti non bloccante ad ogni avvio (risultato memorizzato nella cache per un’ora in `~/.chloros/update_cache.json`) e visualizza `Update available: vX.Y.Z` quando è disponibile una versione più recente. Le impostazioni e i progetti rimangono intatti dopo l’aggiornamento; al termine sarà necessario effettuare nuovamente l’accesso.

## Disinstallazione

```bash
sudo apt remove chloros
```

La rimozione arresta `chloros-backend.service`, ripristina il limite minimo predefinito per le porte senza privilegi (1024), rimuove il collegamento simbolico di exiftool in bundle e la configurazione del caricatore Arena, e cancella le credenziali memorizzate nella cache. I progetti e i file di dati di `~/.chloros/` rimangono inalterati.

***

## Passi successivi

* [Guida NVIDIA Jetson](nvidia-jetson-guide.md) — Ottimizzazione e distribuzione specifiche per Jetson
* [CLI : Riga di comando](../CLI.md) — la guida CLI
* [API : Python SDK](../api-python-sdk.md) — la guida SDK
* [Riferimento CLI](../reference/cli-reference.md) e [Riferimento SDK](../reference/sdk-reference.md) — Elenco esaustivo dei comandi/API per la versione 1.2.0
* [Adattamento dinamico dell’elaborazione](../processing-architecture/dynamic-compute-adaptation.md) — come Chloros si adatta al tuo hardware
