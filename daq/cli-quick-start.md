# Guida rapida alla CLI (pool-*)

Il driver `chloros-cli` in dotazione gestisce i sensori DAQ tramite la famiglia di comandi **`daq pool-*`** — client HTTP “sottili” che controllano il sensore attraverso il pool di sensori persistente del backend Chloros. Il backend gestisce il trasporto, quindi l’interfaccia grafica (GUI), la CLI e gli script SDK condividono tutti un unico handle attivo invece di contendersi la porta. Tutto ciò di cui un cliente ha bisogno è accessibile tramite `pool-*`: connettersi, trasmettere in streaming, registrare file `.daq` calibrati e scambiare profili CAP.

`pool-*` è l’interfaccia DAQ supportata nelle build rilasciate. `chloros-cli daq --help` elenca i sottocomandi di `pool-*`, mentre `daq discover` / `daq list` eseguono la stessa scansione di `pool-discover`. Laddove non sia disponibile un sottocomando DAQ per l’hardware diretto, il programma termina con un errore esplicito che indica il pacchetto mancante e rimanda a `pool-*` — nessun errore passa inosservato. (Neanche `pip install chloros-sdk` fornisce i comandi hardware diretti.)

***

## Prerequisiti

* **Il backend Chloros deve essere in esecuzione** — i comandi `pool-*` sono client HTTP, non driver hardware. Su Windows, avviare l’app desktop Chloros (che avvia il backend). Su Linux/Jetson senza interfaccia grafica, abilitare il servizio come utente corrente: `sudo systemctl enable --now chloros-backend@$USER.service` (vedere [Installazione su Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Accesso a Chloros+ (livello a pagamento)**: eseguire prima `chloros-cli login`. L&#x27;applicazione delle regole avviene lato server: senza accesso, i comandi falliscono con `401 AUTH_REQUIRED`; nel piano gratuito (Iron) falliscono con `403 PLAN_UPGRADE_REQUIRED`.
* I comandi hanno come destinazione predefinita `http://127.0.0.1:5000`; la famiglia `daq pool-*` rispetta la variabile d’ambiente `CHLOROS_BACKEND_URL` se il backend è in esecuzione altrove.

***

## Una sessione di cinque minuti

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — apri un sensore nel pool

| Variante | Significato |
| --- | --- |
| `daq pool-connect` | Rilevamento intelligente: individua qualsiasi DAQ su questa macchina. |
| `daq pool-connect --port PORT` | DAQ-U su una porta seriale specifica (ad es. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M tramite BLE, MAC rilevato automaticamente. |
| `daq pool-connect --mac MAC` | DAQ-M su un MAC BLE noto (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E con nome host o IP noto — **il percorso affidabile**. |
| `daq pool-connect --eth` | DAQ-E con rilevamento automatico (mDNS, con fallback ARP). Vedere l’avvertenza riportata di seguito. |

Flag di ottimizzazione, tutti opzionali:

| Flag | Significato |
| --- | --- |
| `--integration-time MS` / `-t MS` | Tempo di integrazione manuale in millisecondi. |
| `--frame-avg N` / `-f N` | Numero medio di fotogrammi per spettro riportato. |
| `--no-ae` | Disattiva l&#x27;esposizione automatica (AE è attiva per impostazione predefinita). |
| `--no-stream` | Connettiti senza avviare lo streaming (riprendi in seguito con `pool-stream --start`). |
| `--cap-id CAP` | Profilo di correzione Cap; l&#x27;impostazione predefinita del backend è `sunshine_cosine`. Vedi [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Il collegamento di un DAQ-E già presente nel pool con un altro dei suoi indirizzi — il suo nome host `daq-e-<id>.local` o il suo IP — restituisce la voce esistente nel pool invece di aprire l’unità una seconda volta.

{% hint style="warning" %}
**Avvertenza sul rilevamento automatico di `--eth`.** Su un host multi-homed (con più di un&#x27;interfaccia di rete attiva), il *primo* `pool-connect --eth` dopo l’avvio potrebbe risultare vuoto anche se il sensore è funzionante: la scansione di rilevamento potrebbe non individuare l’interfaccia del sensore mentre la cache ARP è fredda. Se `--eth` non trova nulla, riprovare oppure saltare completamente il rilevamento utilizzando `--eth-host <ip-or-hostname>`, che rappresenta il percorso affidabile su macchine multi-homed. Il nome host del DAQ-E è `daq-e-<id>.local` (ad es. `daq-e-def330.local`); funziona anche il suo indirizzo IP semplice.
{% endhint %}

## `pool-discover` — scansione senza connessione

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Esegue la scansione dei sensori a cui è possibile connettersi e mostra l’indirizzo a cui connettersi, senza effettuare la connessione. `--only` accetta un sottoinsieme separato da virgole di `usb,ble,eth`; `--timeout SEC` imposta il tempo di scansione per ogni trasporto (impostazione predefinita 5 s, compreso tra 1 e 20 s). I sensori già aperti nel pool non vengono elencati (un DAQ-M connesso smette di trasmettersi) — per quelli, utilizzare `pool-list`. In una build distribuita, `chloros-cli daq discover` e `daq list` eseguono questa stessa scansione.

## `pool-list` — visualizza cosa è connesso

Mostra tutti i sensori nel pool di backend, incluso l’`sensor_id` necessario per tutti gli altri comandi:

| Modello | Formato `sensor_id` | Esempio |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 ottetti con trattini | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Un DAQ-E il cui PTP è stato disattivato (l’impostazione viene salvata sull’unità) viene contrassegnato in `pool-list` con `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` e il comando che lo riattiva — vedi [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — lettura dei frame dello spettro

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Restituisce il frame più recente oppure i frame più recenti `--recent N`; `--json` emette un output leggibile dal computer per l’automazione tramite script. I frame rappresentano l’irraggianza spettrale (W/m²/nm) calibrata radiometricamente su una griglia di 135 punti compresa tra 340 e 1010 nm, con il profilo del cappuccio del sensore già applicato. Per ottenere valori quantitativi di irradianza, calcolare la media di almeno 15 secondi di fotogrammi: si tratta di una caratteristica dello strumento, non di un difetto.

## `pool-stream` — mettere in pausa o riprendere lo streaming

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — registrare un file `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Flag | Predefinito | Significato |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Durata della registrazione in secondi; `0` indica di continuare l&#x27;esecuzione fino all&#x27;emissione di `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Directory di output. Con il backend su questo computer, un percorso relativo viene risolto rispetto alla cartella in cui è stato eseguito il comando; con un backend remoto, il percorso viene risolto **sul computer su cui è in esecuzione il backend**. |
| `--device-name NAME` | — | Etichetta memorizzata con la registrazione. |
| `--stop` | — | Interrompe una registrazione in corso. |
| `--confirm-after SEC` | `20` | Dopo l’avvio, attendere fino a questo tempo affinché venga scritta la prima riga e avvisare con un messaggio in evidenza (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`) se non ne arriva nessuna. `0` disabilita il controllo. |
| `--heartbeat SEC` | `30` | Con `--duration`, visualizza il conteggio delle righe ogni SEC secondi e segnala un avviso se smette di aumentare. `0` lo disabilita. |

{% hint style="info" %}
La registrazione avviene nel backend, quindi il file `.daq` viene salvato sul filesystem della **macchina backend** — per impostazione predefinita in `~/Documents/DAQ Live View/` lì, non necessariamente dove è stata eseguita la CLI. I nomi dei file includono l’ID del sensore e un timestamp.
{% endhint %}

## `pool-set-cap` — dichiarare il cappuccio montato

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

L’ID del cappuccio seleziona il profilo di correzione misurato in fabbrica applicato a ogni spettro e **deve corrispondere al cappuccio fisicamente montato sul sensore** — né il sensore né il software possono rilevare il cappuccio autonomamente, e la selezione viene registrata in ogni file `.daq`. L’impostazione predefinita ovunque è `sunshine_cosine` (ogni DAQ viene fornito con il cappuccio correttore coseno Sunshine installato, con un’attenuazione di circa 12× prevista dal progetto — una sostituzione del cappuccio non dichiarata corregge erroneamente gli spettri di circa quel fattore).

| `--cap-id` | Disponibile su |
| --- | --- |
| `sunshine_cosine` (impostazione predefinita) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Solo DAQ-U |
| `none` | Solo DAQ-E — vedi nota |

Un ID del cappuccio non compreso nel set del sensore viene rifiutato al momento della connessione con un errore evidente. `none` (DAQ-E) indica che il coperchio è stato rimosso fisicamente — continua comunque ad applicare un profilo geometrico di fabbrica per il diffusore in vetro incassato del DAQ-E, quindi non è un’operazione nulla, e un DAQ-E senza coperchio rappresenta una configurazione da banco, non una configurazione da campo supportata. (Un DAQ-U senza cappuccio è completamente nudo e non necessita di alcun profilo di correzione; il DAQ-M viene utilizzato con il proprio cappuccio Sunshine.)

## `pool-disconnect` — Rilascio dei sensori

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` libera un sensore indipendentemente da chi lo abbia aperto, quindi libera anche un sensore lasciato nel pool da uno script che si è bloccato (a condizione che la registrazione su di esso sia terminata prima). Un sensore che non si trova nel pool non ha nulla da liberare: il comando stampa `<sensor-id>: already released (not in the pool)` e termina con `0`. Una liberazione rifiutata o non confermata dal backend termina con `1`.

## `pool-enable-ptp` — riattivare il PTP di un DAQ-E

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Un DAQ-E conserva l’impostazione di attivazione/disattivazione del PTP nella propria memoria, quindi un’unità il cui PTP è stato disattivato rimane disattivata — e i suoi timestamp non sono sincronizzati via PTP — finché non viene riattivata. `pool-list` contrassegna tale unità e visualizza questo comando per essa. Chloros non riattiva mai un’unità di propria iniziativa, poiché lo stato “off” può essere intenzionale. Novità della versione 1.3.0.

## Impostazioni del dispositivo DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` mostra il firmware del sensore (e se questa versione di Chloros ne contiene uno più recente), il suo indirizzo e come lo ha ottenuto, le impostazioni di rete salvate qualora differiscano da quelle correnti, lo streaming all’accensione e il PTP. `pool-set-network` riavvia il sensore per applicare la modifica e lo ricollega con le impostazioni precedenti; `--no-reboot` si limita a salvarle. L’indirizzo statico deve trovarsi sulla rete di questo computer, altrimenti Chloros lo rifiuta; `--force` ignora tale restrizione, ma solo per un computer a cui verrà riassegnato l’indirizzo o che verrà instradato verso di esso. I riavvii vengono rifiutati mentre il sensore sta registrando. `pool-update-firmware` corrisponde all’opzione **Aggiorna a \<version\>**</version\> dell’interfaccia grafica <version\>per la riga di comando ed è il modo per aggiornare un DAQ-E su Linux, che non dispone di interfaccia grafica. Novità della versione 1.3.0.***

## Riepilogo dei comandi

| Comando | Scopo |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Aprire un sensore nel pool di backend. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Esegue la scansione dei sensori senza connettersi (individua l’indirizzo MAC BLE di un DAQ-M). |
| `daq pool-list` | Visualizza tutti i sensori nel pool con il relativo `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Gli ultimi N frame di spettro calibrati. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Riprende/mette in pausa lo streaming. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Avvia/interrompe una registrazione `.daq` (lato backend). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Sostituisci il profilo di correzione del limite massimo durante l&#x27;esecuzione. |
| `daq pool-disconnect --sensor-id ID [--all]` | Rilascia un sensore o tutti i sensori. |
| `daq pool-enable-ptp --sensor-id ID` | Riattiva il PTP per un DAQ-E il cui PTP è disattivato. |
| `daq pool-info --sensor-id ID [--json]` | Firmware, indirizzo, impostazioni di rete salvate, streaming all&#x27;accensione e PTP di un DAQ-E. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP o indirizzo statico; riavvia e ricollega il sensore. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Streaming all’accensione. |
| `daq pool-reboot --sensor-id ID` | Riavvia un DAQ-E e riconnettilo. |
| `daq pool-update-firmware --sensor-id ID` | Installa il firmware DAQ-E di questa versione su un sensore meno recente. |

***

## Risoluzione dei problemi relativi alla prima connessione del DAQ-E

1. Il DAQ-E non dispone di LED di stato: verificare l’alimentazione tramite l’indicatore PoE/link sullo switch o sulla porta dell’iniettore e attendere alcuni secondi dopo l’accensione affinché il dispositivo si avvii e si connetta alla rete.
2. Il computer backend deve trovarsi nello **stesso dominio di broadcast** del sensore: l’mDNS non attraversa i router.
3. Il firewall deve consentire il passaggio di mDNS (UDP 5353), dei dati DAQ-E (UDP 5002) e del PTP (UDP 319/320). Il programma di installazione di Windows aggiunge una regola a questo scopo, mentre il programma di installazione di Linux `.deb` le apre in un file `ufw` / `firewalld` attivo al momento dell’installazione; un firewall di terze parti, o uno abilitato in un secondo momento, richiede che siano autorizzati.
4. Ancora nessun riscontro da `--eth`? Utilizza `--eth-host` con il nome host dell’unità (`daq-e-<id>.local`) o l’IP, in modo da saltare la fase di rilevamento.

***
{% hint style="info" %}
**Suggerimento per gli assistenti AI.** Ogni pagina di questo manuale è fornita in formato Markdown grezzo: aggiungi `.md` allo slug dell’URL in minuscolo di una pagina (questa pagina: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); l’indice leggibile dalla macchina è `https://mapir.gitbook.io/chloros/llms.txt`. Per la documentazione completa a livello di flag di `chloros-cli daq` e di ogni altra famiglia di comandi, consulta il [Riferimento CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); il percorso Python è `chloros_sdk.connect_daq_sensor()` nella [Guida di riferimento SDK](../reference/sdk-reference.md).
{% endhint %}
</version\>
