# Rete DAQ-E e sincronizzazione temporale

> La configurazione fisica della rete per il sensore — cablaggio, PoE, assegnazione dell&#x27;indirizzo IP e impostazioni di rete del dispositivo stesso — è descritta nel **[manuale utente DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Questa pagina tratta gli aspetti relativi al modello Chloros: connessione, sincronizzazione temporale e cosa fare quando la procedura di individuazione non porta a nessun risultato.

Il DAQ-E è il modello Ethernet della famiglia DAQ: alimentato tramite PoE, rilevabile tramite mDNS (servizio `_daq-e._tcp`) e indirizzabile tramite un nome host derivato dall’ID del sensore — `daq-e-<6 hex>.local`, ad esempio `daq-e-def330.local`. Questa pagina illustra come il dispositivo trasferisce i dati sulla rete e come partecipa alla sincronizzazione temporale PTP.

## Modalità di trasporto

| Modalità | Endpoint | Destinatari | Note |
| --- | --- | --- | --- |
| **Multicast** (predefinito) | UDP `239.10.10.10:5002` | Qualsiasi dispositivo sulla stessa LAN riceve lo stesso flusso | Ogni datagramma è convalidato tramite CRC-16/CCITT |
| **Raw** | Porta TCP `5000` | Esattamente un client (esclusivo) | Compatibile a livello di byte con DAQ-U |

Chloros utilizza il multicast per impostazione predefinita, il che consente alla GUI, a CLI e a SDK di monitorare tutti contemporaneamente un unico sensore.

## Requisiti di rete

* **Stesso dominio di broadcast.** La macchina su cui è in esecuzione Chloros deve trovarsi sullo stesso segmento di rete L2 del sensore — il rilevamento mDNS non attraversa i router.
* **Richiesta del firewall di Windows: accettarla.** La prima volta che Chloros esegue il binding dei socket multicast, Windows Defender richiede l’autorizzazione una volta. Accettando la richiesta, si consentono i dati DAQ-E (UDP 5002), mDNS (UDP 5353) e PTP (UDP 319/320). Su Linux questa operazione avviene in modo silenzioso.
* **Alimentazione PoE, nessun LED di stato.** Il DAQ-E non dispone di un proprio LED: verificare l’alimentazione tramite l’indicatore Link/PoE sullo switch o sulla porta dell’iniettore e attendere alcuni secondi dopo l’accensione affinché si avvii e si connetta alla rete.

## Connessione

**GUI:** Scheda Sensori di luce → Connetti sensore → Tipo di dispositivo &quot;DAQ-E (Ethernet)&quot;. La ricerca viene eseguita solo mentre la finestra di dialogo di connessione è visualizzata sullo schermo (ricerca mDNS più una scansione ARP su Windows), ripetendosi ogni 15 secondi; il pulsante Aggiorna avvia immediatamente una nuova scansione. I sensori rilevati compaiono nel menu a tendina; il primo sensore rilevato viene selezionato automaticamente.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend in esecuzione):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Host con più schede di rete e prima connessione dopo l’avvio

Su host con più di un&#x27;interfaccia di rete attiva, il **primo** `pool-connect --eth` dopo l’avvio può risultare vuoto anche quando il sensore è funzionante: la scansione di rilevamento potrebbe non individuare l’interfaccia su cui risiede il sensore mentre la cache ARP è ancora fredda. La soluzione più affidabile consiste nel saltare la fase di rilevamento e specificare l’indirizzo in modo esplicito:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` accetta il nome host mDNS o l’indirizzo IP, individua sempre il sensore corretto ed è la forma consigliata per gli script e le installazioni headless. Nell’interfaccia grafica, utilizzare il pulsante «Aggiorna» della finestra di dialogo di connessione e attendere un ciclo di nuova scansione.

## Impostazioni del dispositivo e firmware

Il sensore stesso contiene le impostazioni di rete — IP statico o DHCP + indirizzamento link-local, nome del dispositivo, avvio automatico dello streaming all’avvio, password OTA. Queste impostazioni a livello di dispositivo non sono disponibili come comandi nel modello CLI fornito; vengono gestite tramite l’interfaccia grafica Chloros, dove sono visibili, oppure con il supporto MAPIR.

**Gli aggiornamenti del firmware sono integrati nell’interfaccia grafica.**Quando un DAQ-E connesso esegue un firmware più vecchio dell’immagine inclusa nella build Chloros, la riga del sensore mostra un’icona color ambra con la scritta**Aggiornamento disponibile**, e la finestra delle impostazioni a forma di ingranaggio offre un<version>

pulsante</version> “Aggiorna a<version>

”. L’aggiornamento viene scaricato in rete in circa 30 secondi; il DAQ-E si riavvia e si riconnette automaticamente, mentre un trasferimento interrotto lascia intatto il firmware attuale.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sincronizzazione temporale PTP

Il firmware v1.2.0+ del DAQ-E partecipa allo standard IEEE 1588 PTPv2 come orologio ordinario (solo slave). **Il backend dell’host Chloros è il grandmaster PTP** — ogni DAQ-E e ogni telecamera LATTICE sulla LAN si sincronizzano con esso nel dominio 0, mantenendo tutti i timestamp dei dispositivi entro una tolleranza di circa 1 ms. È proprio quell’orologio condiviso a rendere le letture del DAQ allineabili in termini di timestamp con le esposizioni delle telecamere (vedere [Registrazione e formato .daq](recording.md)).

Verifica la sincronizzazione da CLI:

| Comando | Visualizza |
| --- | --- |
| `chloros-cli time-sync status` | Stato del grandmaster host, priorità BMCA, identità dell’orologio |
| `chloros-cli time-sync peers` | Tutti gli slave rilevati (sensori DAQ-E + telecamere LATTICE) |
| `chloros-cli time-sync cameras` | Stato di integrità PTP per singola telecamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Riavvia il processo grandmaster |

Nell’interfaccia grafica, la finestra modale delle impostazioni DAQ-E mostra una riga **PTP Sync** in tempo reale con lo stato PTP attuale del sensore.

Dettagli per i consumatori che richiedono un allineamento rigoroso:

* Ogni datagramma trasmesso contiene un campo di flag; **il bit 2 è impostato sui frame il cui timestamp è sincronizzato con il PTP**. Le pipeline che richiedono un allineamento rigoroso tra telecamera e DAQ devono basare il proprio funzionamento su quel bit.
* Prima di un’acquisizione sincronizzata, verificare che il sensore compaia in `chloros-cli time-sync peers`. (Gli strumenti hardware diretti interni di MAPIR possono anche attivare la registrazione al raggiungimento del lock PTP tramite un flag di `--wait-ptp` che attende fino a 15 s affinché il sensore raggiunga lo stato SLAVE; tale funzionalità non fa parte della versione CLI fornita.)
* Mentre il PTP è attivamente in modalità slave, il sensore rifiuta gli invii manuali di clock (&quot;PTP sta fornendo il clock&quot;). Ciò è previsto dal design: fidatevi del PTP.

## Note su Linux

* **PTP richiede `libcap2-bin` al momento dell’installazione.** Il comando postinst di `.deb` concede i permessi a `cap_net_bind_service=+ep` su `/usr/lib/chloros/chloros-backend` in modo che possa associare le porte PTP 319/320 senza privilegi di root. Se `libcap2-bin` non è presente, tale passaggio viene saltato e PTP non si avvierà. Soluzione:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Jetson / Raspberry Pi senza interfaccia grafica:** alla prima installazione, l&#x27;unità systemd `chloros-backend.service` viene generata ma non abilitata. Per garantire che PTP (e la disponibilità del DAQ) rimangano sempre attivi senza l&#x27;interfaccia grafica:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Senza di essa, il PTP funziona solo mentre è aperta l’interfaccia grafica Chloros.

## Risoluzione dei problemi: “Nessun dispositivo DAQ-E trovato”

| Controllo | Dettagli |
| --- | --- |
| Alimentazione | Nessun LED acceso sul sensore — controllare gli indicatori PoE e di collegamento della porta dello switch/iniettore; attendere alcuni secondi dopo l’accensione |
| Dominio di broadcast | Host e sensore sullo stesso segmento L2; mDNS non instrada |
| Firewall Windows | Accettare la richiesta di Defender al primo avvio (UDP 5002, 5353, 319/320) |
| Host con più schede di rete | Il primo rilevamento dopo l’avvio potrebbe non individuare il sensore — connettersi con `--eth-host <ip-or-hostname>` |
| Nuova scansione dalla GUI | Il rilevamento viene eseguito solo mentre la finestra di dialogo di connessione è aperta; utilizzare Aggiorna |</version>
