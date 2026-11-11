# Profili dei cappucci e campo calibrato

> I cappucci stessi — quali cappucci vengono forniti con quali sensori, come si montano e il loro comportamento ottico — sono documentati nel **[manuale d'uso DAQ](https://mapir.gitbook.io/daq)**. Questa pagina tratta della *dichiarazione* del cappuccio montato a Chloros, che è ciò che rende corretta la correzione.

La calibrazione radiometrica di fabbrica di ogni sensore di luce DAQ descrive il sensore *nudo*. Il cappuccio fisico montato sul diffusore modifica la luce raccolta dal sensore, pertanto Chloros applica un **profilo di correzione del cappuccio** misurato in fabbrica in aggiunta al pacchetto di calibrazione. La dichiarazione del cappuccio corretto è parte integrante dell’ottenimento di dati calibrati: questa pagina illustra quali cappucci sono disponibili per ciascun modello, come dichiararli e quale sia effettivamente l’intervallo spettrale calibrato del sensore.

## Disponibilità dei cappucci per modello

| Profilo del cappuccio (`cap_id`) | Cappuccio fisico | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Cappuccio correttore coseno per luce solare (**predefinito su ogni modello**) | Sì | Sì | Sì |
| `fov_15` / `fov_45` / `fov_90` | Coni di limitazione del campo visivo (15° / 45° / 90°) | Sì | — | Sì |
| `fov_30` / `fov_60` | Coni di limitazione del campo visivo (30° / 60°) | Sì | — | — |
| `none` | Senza cappuccio | — | — | Sì |

Note specifiche per il modello:

* **Il DAQ-M ha un unico profilo di cappuccio: `sunshine_cosine`.** "Bare-plus-Sunshine-cap" è la sua definizione di prodotto, e un DAQ-M "bare" non necessita di alcun profilo geometrico.
* **Un DAQ-U "bare" è realmente "bare"** — non necessita di alcun profilo geometrico, motivo per cui non esiste un profilo `none` per esso.
* **`none` su un DAQ-E NON è un’operazione nulla.** Il diffusore incassato e rivestito di vetro del DAQ-E presenta una propria correzione geometrica effettiva, pertanto «senza cappuccio» costituisce di per sé un profilo misurato su questo modello.
* Un **DAQ-E senza coperchio non è in grado di misurare la luce solare diretta a nessuna elevazione** — il coperchio Sunshine rappresenta la configurazione da campo. Non pianificare lavori all’aperto utilizzando un DAQ-E senza coperchio.

Nelle impostazioni per singolo sensore dell’interfaccia grafica (icona a forma di ingranaggio nella scheda “Sensori di luce”), il menu a tendina **Cappuccio** offre anche l’opzione “Nessuno (sensore nudo)” sui modelli DAQ-U e DAQ-M — su questi due modelli “nudo” significa semplicemente che non viene applicata alcuna correzione del cappuccio, come indicato nelle note precedenti. Selezionatela solo quando il cappuccio è stato fisicamente rimosso.

## Dichiarazione del cappuccio — e perché è importante

**Il codice `cap_id` dichiarato deve corrispondere al cappuccio fisicamente montato sul sensore.** Né il sensore né il software sono in grado di rilevare il cappuccio montato. La dichiarazione determina due aspetti:

1. La **correzione in tempo reale** applicata a ogni spettro.
2. Il **codice del cappuccio inserito in ogni registrazione `.daq`**, su cui si basa l’elaborazione della riflettanza a valle.

Il cappuccio Sunshine attenua la luce di circa **12 volte per progettazione**, quindi registrare con un cappuccio errato dichiarato comporta una scalatura errata degli spettri di circa quel fattore. Dichiarare immediatamente eventuali cambi di cappuccio.

### Impostazione del cappuccio

Interfaccia grafica: scheda Sensori di luce → icona a forma di ingranaggio sulla riga del sensore → menu a tendina **Cappuccio**. L’impostazione predefinita per ogni modello è `sunshine_cosine`, un’ipotesi (DAQ-U, DAQ-M e DAQ-E vengono forniti senza cappuccio, e una registrazione indica che il cappuccio è stato ipotizzato fino a quando non se ne sceglie uno), e la selezione rimane valida per tutto il progetto.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(il backend deve essere in esecuzione):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

La CLI accetta sintatticamente l’elenco completo `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); ogni profilo viene convalidato rispetto al modello del sensore al momento della connessione, quindi un ID di cappuccio non disponibile (ad esempio un ID solo E su un DAQ-U) genera un errore chiaro anziché una correzione errata. L’impostazione predefinita del backend quando non viene passato nulla è `sunshine_cosine`.

Python SDK: assegnare un nome al cap al momento della connessione con `connect_daq_sensor(cap_id="sunshine_cosine")`, oppure sostituirlo in fase di esecuzione con `DAQSensorSession.set_cap("fov_45")`; consultare il [Riferimento SDK](../reference/sdk-reference.md#daq-sensor-sessions).

Avanzato: i profili sono inclusi nell’installazione di Chloros all’indirizzo `daq/cap_profiles/<u|m|e>/<cap_id>.json` e possono essere sovrascritti per singolo utente all’indirizzo `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Indipendentemente dai limiti massimi, i sensori che non sono mai stati ricalibrati ricevono automaticamente una piccola correzione dell’offset di fondo derivata dalla flotta — senza alcun intervento da parte dell’utente.

## DAQ-E-S: diffusore fisso e inclinazione

Il **DAQ-E-S** è un DAQ-E con due differenze rilevanti per Chloros. Tutto il resto (rilevamento, connessione, PTP, registrazione, impostazioni del dispositivo) funziona esattamente come per un DAQ-E.

* **Il suo diffusore è permanente, quindi non richiede alcuna correzione tramite profilo.** Il diffusore era presente sull’unità al momento della misurazione della calibrazione di fabbrica, pertanto tale calibrazione indica già l’irraggiamento verso il basso effettivo. Chloros non applica alcun profilo di limite massimo a questo dispositivo, indipendentemente da quanto richiesto: un limite massimo specificato per esso viene ignorato con un messaggio, le sue registrazioni indicano che il limite massimo deriva dal modello e l’avviso relativo al limite massimo presunto non viene mai attivato per questo dispositivo. In Impostazioni progetto → Sensore di luce DAQ la sua riga non mostra alcun limite massimo, e l’override del limite massimo a livello di progetto non lo riguarda. Non applicare manualmente un limite massimo ai suoi dati: la correzione Sunshine risultante sarebbe circa 11 volte superiore al valore reale.
* **È dotato di un accelerometro a 3 assi.** La risposta coseno del diffusore è valida solo quando è in posizione orizzontale, quindi un sensore inclinato riporta un’irradianza errata senza altri sintomi. Ogni lettura DAQ-E-S contiene quindi i propri valori di inclinazione, rollio e beccheggio, e un `.daq` registrato da uno di essi li memorizza accanto a ogni spettro (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), consentendo di distinguere una nuvola da un supporto urtato.

Per ogni campo relativo all’assetto e per sapere come viene segnalato un errore nel flusso dei dati di assetto, consultare [DAQ-E-S nella Guida di riferimento CLI](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Prestazioni del Sunshine Cap (configurazione per esterni)

Dati su cui basare le procedure:

| Proprietà | Valore |
| --- | --- |
| Campo visivo | Emisferico a 180° |
| Errore di risposta cosinusoidale | ≤ ±4 % fino a 60° di incidenza; ≤ ±4,5 % fino a 70° |
| Limite per sole basso | Non raccomandato al di sotto di ~15° di elevazione solare |
| Attenuazione | ~12× (per progettazione) |
| Ripetibilità del rimontaggio del cappuccio | ≈ 1,5 % |
| Irradianza quantitativa | Media di **≥ 15 s** di letture (caratteristica dello strumento, non un difetto) |

Per qualsiasi valore di irradianza quantitativa — compresi i riferimenti di riflettanza — utilizzare una media di almeno 15 secondi di letture anziché un singolo fotogramma.

## Intervallo spettrale calibrato

| Proprietà | Valore |
| --- | --- |
| Campionamento spettrale | 340–1010 nm con incrementi di 5 nm (135 punti) |
| Intervallo calibrato radiometricamente | **~374–974 nm** (imposto dal software) |

Il sensore riporta l’intera griglia 340–1010 nm, ma il guadagno radiometrico tracciabile secondo gli standard NIST copre un intervallo di ~374–974 nm. Chloros **rifiuta la suddivisione in riflettanza assoluta** per qualsiasi banda della fotocamera con meno della metà del proprio peso spettrale all’interno di tale intervallo, segnalando il motivo di salto `dls-uncalibrated-band-<nm>` anziché generare un prodotto non calibrato. Tra gli SKU delle fotocamere in commercio, solo il filtro F988 non rientra in questo intervallo; in tal caso viene utilizzato il flusso di lavoro con pannello di riflettanza — vedere [Flussi di lavoro con riflettanza](reflectance.md).

Per i modelli di sensore, i trasporti e gli ID dei sensori, consultare la [panoramica sul DAQ](README.md). Per informazioni su come viene utilizzato il "cap stamp" durante l’elaborazione, consultare [Registrazione e formato .daq](recording.md).
