# Rete DAQ-E e sincronizzazione dell'ora

> La configurazione fisica della rete per il sensore — cablaggio, PoE, assegnazione dell'indirizzo IP e impostazioni di rete del dispositivo stesso — è descritta nel **[manuale d'uso DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Questa pagina tratta gli aspetti relativi a Chloros: connessione, sincronizzazione temporale e cosa fare quando la ricerca dei dispositivi non dà risultati.

Il DAQ-E è il modello Ethernet della famiglia DAQ: alimentato tramite PoE, rilevabile tramite mDNS (servizio `_daq-e._tcp`) e indirizzabile tramite un nome host derivato dall’ID del sensore — `daq-e-<6 hex>.local`, ad es. `daq-e-def330.local`. Questa pagina illustra come il dispositivo trasferisce i dati sulla rete e come partecipa alla sincronizzazione temporale PTP.

## Modalità di trasporto

| Modalità | Endpoint | Destinatari | Note |
| --- | --- | --- | --- |
| **Multicast** (predefinito) | UDP `239.10.10.10:5002` | Qualsiasi dispositivo sulla stessa LAN riceve lo stesso flusso | Ogni datagramma è convalidato tramite CRC-16/CCITT |
| **Raw** | Porta TCP `5000` | Esattamente un client (esclusivo) | Compatibile a livello di byte con DAQ-U |

Chloros utilizza il multicast per impostazione predefinita, il che consente alla GUI, alla CLI e all’SDK di monitorare tutte contemporaneamente un unico sensore.

## Requisiti di rete

* **Stesso dominio di broadcast.** Il computer su cui è in esecuzione Chloros deve trovarsi sullo stesso segmento di rete L2 del sensore — il rilevamento mDNS non attraversa i router.
* **Firewall.** Su Windows, il programma di installazione aggiunge la regola firewall **Chloros Backend (UDP-In)**per il backend, su ogni profilo di rete, che copre i dati DAQ-E (UDP 5002), mDNS (UDP 5353) e PTP (UDP 319/320), oltre a una regola**"Chloros Backend (TCP-In)"**, limitata alla sottorete locale, per la connessione che un DAQ-E stabilisce con il computer durante un aggiornamento del firmware. Su Linux, `.deb` apre le porte UDP in `ufw` o `firewalld` quando quel firewall è attivo al momento dell’installazione. Un firewall di terze parti, o uno abilitato in un secondo momento, richiede che le stesse porte siano consentite.
* **Alimentazione PoE, nessun LED di stato.** Il DAQ-E non dispone di un proprio LED: verificare l’alimentazione tramite l’indicatore Link/PoE sullo switch o sulla porta dell’iniettore e attendere alcuni secondi dopo l’accensione affinché si avvii e si colleghi alla rete.

## Connessione

**GUI:** Scheda Sensori di luce → Connetti sensore → Tipo di dispositivo "DAQ-E (Ethernet)". La ricerca viene eseguita solo mentre la finestra di dialogo di connessione è visualizzata sullo schermo (scansione mDNS più uno sweep ARP su Windows), ripetendosi ogni 15 secondi; il pulsante Aggiorna esegue immediatamente una nuova scansione. I sensori rilevati compaiono nel menu a tendina; il primo sensore rilevato viene selezionato automaticamente.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend in esecuzione):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

Un DAQ-E già presente nel pool viene riconosciuto sia dal nome host che dall’indirizzo IP: collegandolo nuovamente con l’altro indirizzo viene restituita la voce esistente nel pool anziché aprire l’unità una seconda volta.

### Host con più schede di rete e primo collegamento dopo l’avvio

Su host con più di un'interfaccia di rete attiva, il **primo** `pool-connect --eth` dopo l’avvio può risultare vuoto anche quando il sensore è funzionante — la scansione di rilevamento può non individuare l’interfaccia su cui risiede il sensore mentre la cache ARP è ancora fredda. La soluzione affidabile consiste nel saltare la fase di rilevamento e passare l’indirizzo in modo esplicito:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` accetta il nome host mDNS o l’IP, individua sempre il sensore corretto ed è la forma consigliata per gli script e le installazioni headless. Nell’interfaccia grafica, utilizzare il pulsante “Aggiorna” della finestra di dialogo di connessione e attendere un ciclo di ricerca.

## Impostazioni del dispositivo e firmware

Il sensore stesso conserva le proprie impostazioni di rete: DHCP (l’impostazione di fabbrica, che ricorre a un indirizzo link-local 169.254.x.x quando nessun server DHCP risponde) o un indirizzo statico. Conserva inoltre l’impostazione relativa all’avvio dello streaming all’accensione. È possibile modificarle nell’interfaccia grafica (nell’icona a forma di ingranaggio delle impostazioni del sensore, **Rete**:**Modifica…**,**Trasmissione all’accensione**,**Riavvia sensore**; vedere [La scheda DAQ](gui.md)), oppure tramite CLI e SDK con `pool-set-network`, `pool-set-auto-stream` e `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; consultare il [Riferimento CLI](../reference/cli-reference.md#daq-e-device-settings)). Il sensore si riavvia per applicare una modifica di rete e Chloros lo ricollega. È necessario che sulla rete del computer su cui è in esecuzione Chloros sia presente un indirizzo statico: Chloros rifiuta quelli che non è riuscito a raggiungere dopo il riavvio. L’ID del sensore e il suo nome host `daq-e-<id>.local` non cambiano mai. Novità nella versione 1.3.0.

**Gli aggiornamenti del firmware sono integrati nell’interfaccia grafica (GUI).** Quando un DAQ-E connesso utilizza un firmware più vecchio rispetto all’immagine inclusa nella versione di Chloros in uso, la riga relativa al sensore mostra un’icona color ambra con la dicitura **Aggiornamento disponibile**, e la finestra delle impostazioni a forma di ingranaggio offre un <version>pulsante</version>

“Aggiorna a<version>”. L’aggiornamento viene scaricato in rete in circa 30 secondi; il sensore si riavvia e si riconnette automaticamente, e un trasferimento interrotto lascia intatto il firmware corrente. Dalla CLI o dall’SDK, e su Linux, che non dispone di interfaccia grafica, eseguire `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sincronizzazione temporale PTP

Il firmware DAQ-E v1.2.0+ partecipa allo standard IEEE 1588 PTPv2 come orologio ordinario (solo slave). **Il backend dell’host Chloros è il grandmaster PTP** — ogni DAQ-E e ogni telecamera LATTICE sulla LAN si sincronizza con esso nel dominio 0, mantenendo tutti i timestamp dei dispositivi entro una tolleranza di circa 1 ms. È proprio quell’orologio condiviso a rendere le letture del DAQ sincronizzabili con le esposizioni delle telecamere (vedi [Registrazione e formato .daq](recording.md)).

Verifica la sincronizzazione dalla CLI:

| Comando | Mostra |
| --- | --- |
| `chloros-cli time-sync status` | Stato del grandmaster host, priorità BMCA, identità dell’orologio e `health`: `OK`, oppure `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` con un `health_detail` che spiega perché i suoi pacchetti non raggiungono la rete |
| `chloros-cli time-sync peers` | Tutti gli slave rilevati (sensori DAQ-E + telecamere LATTICE) |
| `chloros-cli time-sync cameras` | Stato di integrità PTP per ciascuna telecamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Riavvia il processo grandmaster |

Nell’interfaccia grafica, la finestra modale delle impostazioni DAQ-E mostra una riga **PTP Sync** in tempo reale con lo stato PTP attuale del sensore.**Un DAQ-E con il PTP disattivato rimane disattivato.** L’unità conserva l’impostazione di attivazione/disattivazione del PTP nella propria memoria, quindi un’unità il cui PTP era disattivato rimane disattivata dopo una riconnessione o un riavvio, e i suoi timestamp non sono sincronizzati tramite PTP. `chloros-cli daq pool-list` segnala tale unità (`!! PTP is switched OFF on this unit …`) e stampa il comando che risolve il problema:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros non riattiva mai un’unità autonomamente, poiché lo stato disattivato può essere intenzionale.

Dettagli per gli utenti che richiedono un allineamento rigoroso:

* Ogni datagramma trasmesso in streaming contiene un campo "flags"; **il bit 2 è impostato sui frame il cui timestamp è sincronizzato con PTP**. Le pipeline che richiedono un allineamento rigoroso tra telecamera e DAQ devono basare il proprio funzionamento su quel bit.
* Prima di un’acquisizione sincronizzata, verificare che il sensore compaia in `chloros-cli time-sync peers` e che `chloros-cli daq pool-list` non segnali il suo PTP come disattivato. (Gli strumenti hardware diretti interni di MAPIR possono anche attivare la registrazione in base al blocco PTP tramite un flag `--wait-ptp` che attende fino a 15 s affinché il sensore raggiunga lo stato SLAVE; tali strumenti non fanno parte della CLI fornita.)
* Mentre il PTP è attivamente in modalità slave, il sensore rifiuta gli invii manuali di clock (“PTP sta fornendo il clock”). Ciò è previsto dal design: fidatevi del PTP.

## Note su Linux

* **Porte PTP senza privilegi di root.** Il PTP si associa alle porte UDP 319/320. `.deb` scrive `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), quindi il backend si associa a queste porte funzionando come utente. A partire dalla versione 1.3.0, il pacchetto non imposta alcuna capacità di file (`setcap`) sul backend e non richiede `libcap2-bin`; non aggiungere manualmente alcuna capacità, poiché in tal caso il backend decompresso non si avvierà. Se PTP non riesce a associarsi alla porta 319, consultare [Installazione su Linux](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Jetson senza monitor / host Linux:** il pacchetto installa i servizi del backend ma non ne abilita nessuno. Per garantire che PTP (e la disponibilità del DAQ) rimangano sempre attivi senza che nessuno esegua un comando, abilitare il servizio come utente corrente:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  In assenza di questa operazione, PTP funziona solo finché qualcosa ha avviato il backend — un comando `chloros-cli` o uno script SDK. Consultare [Installazione su Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) per informazioni sull’utente root `chloros-backend.service` e su come passare da esso.

## Risoluzione dei problemi: "Nessun dispositivo DAQ-E trovato"

| Controllo | Dettagli |
| --- | --- |
| Alimentazione | Nessun LED acceso sul sensore — controllare gli indicatori PoE e di collegamento della porta dello switch/iniettore; attendere alcuni secondi dopo l’accensione |
| Dominio di broadcast | Host e sensore sullo stesso segmento L2; mDNS non instrada |
| Firewall | Windows: la regola **Chloros Backend (UDP-In)** del programma di installazione consente l’accesso allo streaming DAQ-E (UDP 5002), a mDNS (5353) e a PTP (319/320); un firewall di terze parti deve configurare le stesse impostazioni. Linux: `.deb` apre queste porte in `ufw` / `firewalld` solo se il firewall era attivo al momento dell’installazione — consentirle manualmente se abilitato in un secondo momento |
| Host con più schede di rete | La funzione di rilevamento esegue la scansione di mDNS con un fallback ARP; `--eth-host <ip-or-hostname>` salta la fase di rilevamento e si connette direttamente |
| Nuova scansione dall’interfaccia grafica | La funzione di rilevamento viene eseguita solo mentre la finestra di dialogo di connessione è aperta; utilizzare “Aggiorna” |</version>
