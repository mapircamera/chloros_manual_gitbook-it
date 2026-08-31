# Sensori di luce DAQ

> **Cerchi informazioni sull&#x27;hardware?**I sensori stessi — modelli, montaggio, cappucci, porte, alimentazione e l&#x27;app SCANNER — sono descritti nel**[manuale utente DAQ](https://mapir.gitbook.io/daq)**. Questo capitolo ne illustra l’utilizzo a partire da Chloros.

I sensori di luce **DAQ** di MAPIR misurano la luce ambientale sotto forma di spettri calibrati radiometricamente. In Chloros svolgono due ruoli:

* **Uno strumento spettrale autonomo** — grafici spettrali in tempo reale, dati colorimetrici e registrazioni `.daq`, il tutto dalla [scheda Sensori di luce](gui.md), [CLI](cli-quick-start.md) o l’Python SDK.
* **Una sorgente di irraggiamento discendente per la riflettanza** — durante l’elaborazione, Chloros interpola le letture di `.daq` in corrispondenza di ciascun timestamp di esposizione dell’acquisizionetimestamp di esposizione di ciascuna acquisizione e utilizza la luce discendente misurata per convertire la radianza della fotocamera in riflettanza (`--reflectance-source daq`); non è richiesto alcun pannello in scena per le bande calibrate.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Tre modelli, un unico formato dati

| Modello | Trasporto | Rilevamento |
| --- | --- | --- |
| **DAQ-U** | USB (seriale) | scansione porta seriale |
| **DAQ-M** | Bluetooth Low Energy | scansione BLE per nome |
| **DAQ-E** | Ethernet (IPv4, alimentazione PoE) | mDNS `_daq-e._tcp` (nome host `daq-e-<id>.local`) |

Tutti e tre utilizzano lo stesso protocollo di comunicazione e forniscono dati identici:

* Uno **spettro di 135 punti da 340 a 1010 nm con incrementi di 5 nm**, oltre ai valori tristimolo CIE XYZ, in ogni frame.
* **Irradianza spettrale calibrata radiometricamente in W/m²/nm** — il pacchetto di calibrazione di fabbrica di ciascuna unità (insieme al relativo profilo di correzione del cappuccio attivo) viene applicato prima che i dati vi raggiungano.
* Lo stesso **formato di registrazione `.daq`** (un file SQLite). L’elaborazione a valle è identica indipendentemente dal protocollo di trasporto utilizzato per generare il file.

Gli stack di trasporto (seriale USB, BLE, mDNS/zeroconf) sono integrati nel backend Chloros — non è necessario installare nulla per comunicare con uno qualsiasi dei tre modelli tramite l’interfaccia grafica o i comandi `pool-*` di CLI.

***

## Intervallo calibrato: 340–1010 nm riportati, ~374–974 nm calibrati

Il sensore riporta l’intera griglia 340–1010 nm, ma il guadagno radiometrico tracciabile secondo gli standard NIST copre approssimativamente **374–974 nm**. Chloros rifiuta la divisione per riflettanza assoluta per qualsiasi banda della fotocamera con meno della metà del proprio peso spettrale all’interno di tale intervallo calibrato; la banda saltata viene segnalata con il motivo di salto `dls-uncalibrated-band-<nm>`.

Tra gli SKU dei filtri LATTICE disponibili, solo **F988** è interessato:

La riflettanza dell’F988 viene calibrata utilizzando un pannello di riflettanza in scena: poiché la banda si trova al di fuori dell’intervallo calibrato del sensore di luce DAQ, Chloros applica l’ultima acquisizione del pannello e la mantiene tra un rilevamento e l’altro.

Se un&#x27;acquisizione di F988 viene elaborata con a disposizione solo i dati DAQ, Chloros rifiuta la riflettanza basata su DAQ per quella banda con motivo di esclusione `dls-uncalibrated-band-988` — il [flusso di lavoro del pannello di riflettanza](../calibration-targets.md) è il percorso supportato per F988.

***

## ID dei sensori

Ogni DAQ riporta un ID sensore stabile. La sua struttura varia a seconda del modello:

| Modello | Struttura ID | Esempio |
| --- | --- | --- |
| DAQ-U | 5 ottetti con trattini | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 ottetti con trattini | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

L’ID del sensore è:

* impresso in ogni file `.daq` che registra,
* la chiave che Chloros utilizza per recuperare il pacchetto di calibrazione di fabbrica di quell’unità,
* il valore che si passa a `--sensor-id` nei comandi CLI `pool-*`, e
* per il DAQ-E, anche il suo nome host mDNS (`daq-e-def330.local`) — il valore accettato da `--eth-host`.

***

## Calibrazione di fabbrica e cloud

Ogni unità DAQ viene calibrata singolarmente in fabbrica con una catena radiometrica tracciabile al NIST, e Chloros carica il pacchetto di calibrazione di ciascuna unità in base all’ID del sensore. Il rapporto di calibrazione per singola unità (PDF) è scaricabile dalle impostazioni del sensore nella [scheda Sensori di luce](gui.md).

{% hint style="warning" %}
**DAQ-U e DAQ-M richiedono l’accesso al cloud per la calibrazione.**Nessuno dei due modelli memorizza nulla a bordo: i loro pacchetti di calibrazione di fabbrica risiedono nel cloud di MAPIR e vengono recuperati tramite l’ID del sensore (per poi essere memorizzati nella cache locale). Chloros necessita di una connessione a Internet per fornire dati calibrati in W/m²/nm da un DAQ-U o DAQ-M.**Il DAQ-E costituisce l’eccezione**: conserva la propria calibrazione sul dispositivo stesso.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Dove vengono salvate le registrazioni

| Superficie | Destinazione predefinita di `.daq` |
| --- | --- |
| Interfaccia grafica — Scheda “Sensori di luce” | `<project folder>/light_sensor/` (le registrazioni completate vengono aggiunte automaticamente al progetto) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` sul computer che esegue il backend |

Ogni nome di file `.daq` include l’ID del sensore e un timestamp.

***

## In questo capitolo

* [**La scheda DAQ in Chloros**](gui.md) — guida completa all’interfaccia grafica: collegamento di ciascun modello, impostazioni per singolo sensore, grafici dello spettro, dati colorimetrici in tempo reale, riflettanza a doppio sensore e registrazione.
* [**Guida rapida a CLI (pool-\*)**](cli-quick-start.md) — gestione dei sensori DAQ da `chloros-cli daq pool-*`, il percorso da riga di comando supportato.
* [**Profili di limite massimo e intervallo calibrato**](caps-and-range.md) — quali limiti massimi esistono per ciascun modello, come dichiararli e l’intervallo spettrale calibrato in dettaglio.
* [**Registrazione e formato .daq**](recording.md) — il formato SQLite `.daq` e i flussi di lavoro di registrazione.
* [**Rete DAQ-E e sincronizzazione temporale**](ethernet-ptp.md) — modalità di trasporto DAQ-E e sincronizzazione temporale PTP.
* [**Flussi di lavoro sulla riflettanza**](reflectance.md) — utilizzo dei dati DAQ in direzione discendente per calcolare la riflettanza.
* Per la documentazione completa a livello di flag, consultare il [CLI Riferimento](../reference/cli-reference.md) (sezione `chloros-cli daq`) e il [Riferimento SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), entrambi redatti per essere direttamente fruibili dagli assistenti di intelligenza artificiale.
