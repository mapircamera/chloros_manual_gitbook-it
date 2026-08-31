# CLI Guida rapida (pool-*)

L&#x27;unità `chloros-cli` fornita in dotazione gestisce i sensori DAQ tramite la famiglia di comandi **`daq pool-*`** — client leggeri HTTP che gestiscono il sensore tramite il pool di sensori persistente del backend Chloros. Il backend gestisce il trasporto, quindi l’interfaccia grafica, lo script CLI e lo script SDK condividono tutti un unico handle attivo invece di contendersi la porta. Tutto ciò di cui un cliente ha bisogno è accessibile tramite `pool-*`: connettersi, trasmettere in streaming, registrare file `.daq` calibrati e scambiare profili di capacità.

`pool-*` è anche l’**unica** superficie DAQ nelle build rilasciate. `chloros-cli daq --help` elenca i sottocomandi di `pool-*`, e l’esecuzione di un sottocomando DAQ diretto all’hardware su una build distribuita termina con un errore esplicito che indica il pacchetto mancante e rimanda a `pool-*` — nulla fallisce in modo silenzioso. (I comandi hardware diretti funzionano solo da un checkout del codice sorgente MAPIR; nemmeno `pip install chloros-sdk` li fornisce.)

***

## Prerequisiti

* **Il backend Chloros deve essere in esecuzione** — i comandi `pool-*` sono client HTTP, non driver hardware. Su Windows, avviare l’applicazione desktop Chloros (che avvia il backend). Su Linux/Jetson senza interfaccia grafica, abilitare il servizio: `sudo systemctl enable --now chloros-backend.service`.
* **Accesso a Chloros+ (livello a pagamento)**: eseguire prima `chloros-cli login`. L&#x27;applicazione delle regole avviene lato server: senza accesso, i comandi falliscono con `401 AUTH_REQUIRED`; sul piano gratuito (Iron) falliscono con `403 PLAN_UPGRADE_REQUIRED`.
* Per impostazione predefinita, i comandi hanno come destinazione `http://127.0.0.1:5000`; la famiglia `daq pool-*` rispetta la variabile d’ambiente `CHLOROS_BACKEND_URL` se il backend è in esecuzione altrove.

***

## Una sessione di cinque minuti

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

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
| `daq pool-connect --ble` | DAQ-M su BLE, MAC rilevato automaticamente. |
| `daq pool-connect --mac MAC` | DAQ-M su un MAC BLE noto (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E con nome host o IP noto — **il percorso affidabile**. |
| `daq pool-connect --eth` | DAQ-E con rilevamento automatico (mDNS, con fallback ARP). Vedi l’avvertenza qui sotto. |

Flag di ottimizzazione, tutti opzionali:

| Flag | Significato |
| --- | --- |
| `--integration-time MS` / `-t MS` | Tempo di integrazione manuale in millisecondi. |
| `--frame-avg N` / `-f N` | Numero medio di fotogrammi per spettro riportato. |
| `--no-ae` | Disattiva l’esposizione automatica (AE è attiva per impostazione predefinita). |
| `--no-stream` | Connettersi senza avviare lo streaming (riprendere in un secondo momento con `pool-stream --start`). |
| `--cap-id CAP` | Profilo di correzione del limite massimo; l&#x27;impostazione predefinita del backend è `sunshine_cosine`. Vedi [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**Avvertenza sul rilevamento automatico di `--eth`.** Su un host multi-homed (con più di un’interfaccia di rete attiva), il *primo* `pool-connect --eth` dopo l’avvio potrebbe risultare vuoto anche se il sensore è integro: la scansione di rilevamento potrebbe non individuare l’interfaccia del sensore mentre la cache ARP è fredda. Se `--eth` non trova nulla, riprovare oppure saltare del tutto il rilevamento utilizzando `--eth-host <ip-or-hostname>`, che rappresenta il percorso affidabile su macchine multi-homed. Il nome host del DAQ-E è `daq-e-<id>.local` (ad es. `daq-e-def330.local`); funziona anche il suo indirizzo IP semplice.
{% endhint %}

## `pool-list` — visualizza i dispositivi collegati

Mostra tutti i sensori nel pool di backend, compreso il comando `sensor_id` necessario per tutti gli altri comandi:

| Modello | Formato `sensor_id` | Esempio |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 ottetti con trattini | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — lettura dei frame dello spettro

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Restituisce il frame più recente, oppure i frame più recenti `--recent N`; `--json` emette un output leggibile dal computer per l’automazione tramite script. I frame rappresentano l’irraggianza spettrale (W/m²/nm) calibrata radiometricamente su una griglia di 135 punti compresa tra 340 e 1010 nm, con il profilo di copertura del sensore già applicato. Per ottenere valori quantitativi di irraggiamento, calcolare la media di almeno 15 secondi di fotogrammi: si tratta di una caratteristica dello strumento, non di un difetto.

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
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Directory di output, risolta **sul computer su cui è in esecuzione il backend**. |
| `--device-name NAME` | — | Etichetta memorizzata con la registrazione. |
| `--stop` | — | Interrompe una registrazione in corso. |

{% hint style="info" %}
La registrazione avviene nel backend, quindi il file `.daq` viene salvato nel filesystem del **computer del backend** — per impostazione predefinita in `~/Documents/DAQ Live View/`, non necessariamente nella posizione in cui è stato eseguito CLI. I nomi dei file includono l’ID del sensore e un timestamp.
{% endhint %}

## `pool-set-cap` — dichiarare il cappuccio montato

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

L’ID del cappuccio seleziona il profilo di correzione misurato in fabbrica applicato a ogni spettro e **deve corrispondere al cappuccio fisicamente montato sul sensore** — né il sensore né il software possono rilevare il cappuccio autonomamente, e la selezione viene registrata in ogni file `.daq`. L’impostazione predefinita ovunque è `sunshine_cosine` (ogni DAQ viene fornito con il cappuccio correttore coseno Sunshine installato, con un’attenuazione di circa 12× prevista dal progetto — una sostituzione del cappuccio non dichiarata corregge erroneamente gli spettri approssimativamente di quel fattore).

| `--cap-id` | Disponibile su |
| --- | --- |
| `sunshine_cosine` (predefinito) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Solo DAQ-U |
| `none` | Solo DAQ-E — vedi nota |

Un ID del cappuccio non compreso nell’insieme previsto dal sensore viene rifiutato al momento della connessione con un errore evidente. `none` (DAQ-E) indica che il cappuccio è stato rimosso fisicamente — continua comunque ad applicare un profilo geometrico di fabbrica per il diffusore in vetro incassato del DAQ-E, quindi non è un’operazione nulla (no-op), e un DAQ-E senza cappuccio è una configurazione da banco, non una configurazione da campo supportata. (Un DAQ-U senza cappuccio è completamente nudo e non necessita di alcun profilo di correzione; il DAQ-M viene utilizzato con il proprio cappuccio Sunshine.)

## `pool-disconnect` — Rilascio dei sensori

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Riepilogo dei comandi

| Comando | Scopo |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Aprire un sensore nel pool di backend. |
| `daq pool-list` | Mostra tutti i sensori nel pool con il relativo `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Gli ultimi N frame di spettro calibrati. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Riprende / mette in pausa lo streaming. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Avvia / interrompe una registrazione `.daq` (lato backend). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Cambia il profilo di correzione del limite massimo durante l’esecuzione. |
| `daq pool-disconnect --sensor-id ID [--all]` | Rilascia un sensore o tutti i sensori. |

***

## Risoluzione dei problemi relativi alla prima connessione del DAQ-E

1. Il DAQ-E non dispone di LED di stato: verificare l’alimentazione tramite l’indicatore PoE/link sullo switch o sulla porta dell’iniettore e attendere alcuni secondi dopo l’accensione affinché si avvii e si connetta alla rete.
2. Il computer backend deve trovarsi nello **stesso dominio di broadcast** del sensore — l’mDNS non attraversa i router.
3. Su Windows, accettare la richiesta del firewall Defender al primo avvio (mDNS UDP 5353, dati DAQ-E UDP 5002, PTP UDP 319/320).
4. Ancora nessun segnale da `--eth`? Utilizzare `--eth-host` con il nome host dell’unità (`daq-e-<id>.local`) o l’indirizzo IP: questa è la soluzione più affidabile, specialmente su host multi-homed.

***{% hint style="info" %}**Suggerimento per gli assistenti AI.** Ogni pagina di questo manuale è fornita in formato Markdown grezzo: aggiungi `.md` allo slug in minuscolo di una pagina, URL (questa pagina: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); l’indice leggibile dalla macchina è `https://mapir.gitbook.io/chloros/llms.txt`. Per la documentazione completa a livello di flag di `chloros-cli daq` e di ogni altra famiglia di comandi, consulta il [Riferimento CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); il percorso di Python è `chloros_sdk.connect_daq_sensor()` nel [Riferimento SDK](../reference/sdk-reference.md).
{% endhint %}
