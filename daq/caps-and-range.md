# Profili dei cappucci e intervallo calibrato

> I cappucci stessi — quali cappucci vengono forniti con quali sensori, come si montano e il loro comportamento ottico — sono documentati nel **[manuale d&#x27;uso del DAQ](https://mapir.gitbook.io/daq)**. Questa pagina illustra come *dichiarare* il cappuccio montato a Chloros, operazione che garantisce la correttezza della correzione.

La calibrazione radiometrica di fabbrica di ogni sensore di luce DAQ descrive il sensore *nudo*. Il cappuccio fisico montato sul diffusore modifica la luce che il sensore raccoglie, pertanto Chloros applica un **profilo di correzione del cappuccio** misurato in fabbrica in aggiunta al pacchetto di calibrazione. La dichiarazione del cappuccio corretto è parte integrante dell’ottenimento di dati calibrati: questa pagina illustra quali cappucci sono disponibili per ciascun modello, come dichiararli e quale sia effettivamente l’intervallo spettrale calibrato del sensore.

## Disponibilità dei cappucci per modello

| Profilo del cappuccio (`cap_id`) | Cappuccio fisico | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Cappuccio correttore coseno Sunshine (**predefinito su ogni modello**) | Sì | Sì | Sì |
| `fov_15` / `fov_45` / `fov_90` | Coni di limitazione del campo visivo (15° / 45° / 90°) | Sì | — | Sì |
| `fov_30` / `fov_60` | Coni di limitazione del campo visivo (30° / 60°) | Sì | — | — |
| `none` | Senza cappuccio | — | — | Sì |

Note specifiche per il modello:

* **Il DAQ-M ha un unico profilo di cappuccio: `sunshine_cosine`.** &quot;Bare-plus-Sunshine-cap&quot; è la sua definizione di prodotto, e un DAQ-M &quot;bare&quot; non necessita di alcun profilo geometrico.
* **Un DAQ-U &quot;bare&quot; è &quot;true-bare&quot;** — non necessita di alcun profilo geometrico, motivo per cui non esiste un profilo `none` per esso.
* **`none` su un DAQ-E NON è un’operazione nulla.** Il diffusore incassato e rivestito in vetro del DAQ-E presenta una propria correzione geometrica effettiva, pertanto “senza calotta” è di per sé un profilo misurato su questo modello.
* Un **DAQ-E &quot;nudo&quot; non è in grado di misurare la luce solare diretta a nessuna elevazione** — il cappuccio Sunshine rappresenta la configurazione da campo. Non pianificare lavori all’aperto utilizzando un DAQ-E &quot;nudo&quot;.

Nelle impostazioni per singolo sensore dell’interfaccia grafica (icona a forma di ingranaggio nella scheda “Sensori di luce”), il menu a tendina **Cap** offre anche l’opzione “None (sensore nudo)” sui modelli DAQ-U e DAQ-M — su questi due modelli “nudo” significa semplicemente che non viene applicata alcuna correzione del cappuccio, come indicato nelle note precedenti. Selezionatela solo quando il cappuccio è stato fisicamente rimosso.

## Dichiarazione del cappuccio — e perché è importante

**Il codice dichiarato `cap_id` deve corrispondere al cappuccio fisicamente montato sul sensore.** Né il sensore né il software sono in grado di rilevare il cappuccio montato. La dichiarazione determina due aspetti:

1. La **correzione in tempo reale** applicata a ogni spettro.
2. Il **codice del cappuccio registrato in ogni registrazione `.daq`**, su cui si basa l’elaborazione della riflettanza a valle.

Il cappuccio Sunshine attenua di circa **12 volte per progettazione**, quindi la registrazione con un cappuccio errato dichiarato causa una scalatura errata degli spettri di circa quel fattore. Dichiarare immediatamente eventuali cambiamenti di cappuccio.

### Impostazione del cappuccio

Interfaccia grafica: scheda Sensori di luce → icona a forma di ingranaggio sulla riga del sensore → menu a tendina **Cappuccio**. L’impostazione predefinita per ogni modello è `sunshine_cosine` (tutti i sensori DAQ vengono forniti con il correttore coseno installato) e la selezione rimane valida per il progetto.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (il backend deve essere in esecuzione):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

Il modello CLI accetta sintatticamente l’intero elenco `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); ogni profilo viene convalidato rispetto al modello del sensore al momento della connessione, quindi un ID del cappuccio non disponibile (ad esempio un ID solo E su un DAQ-U) genera un errore chiaro anziché una correzione errata. Il valore predefinito del backend quando non viene passato nulla è `sunshine_cosine`.

Python SDK nota: `cap_id` **non** è una manopola SDK — `connect_daq_sensor()` / `DAQSensorSession` non espongono alcun parametro cap. Selezionare il limite tramite i comandi CLI sopra indicati o dal menu a tendina dell’interfaccia grafica; consultare la [Guida di riferimento SDK](../reference/sdk-reference.md).

Avanzato: i profili sono inclusi nell’installazione di Chloros in `daq/cap_profiles/<u|m|e>/<cap_id>.json` e possono essere sovrascritti per singolo utente in `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Indipendentemente dai limiti massimi, i sensori che non sono mai stati ricalibrati ricevono automaticamente una piccola correzione dell’offset scuro derivata dalla flotta — senza alcun intervento da parte dell’utente.

## Prestazioni del limite massimo di luce solare (configurazione per esterni)

Dati su cui basare le procedure:

| Proprietà | Valore |
| --- | --- |
| Campo visivo | Emisferico a 180° |
| Errore di risposta coseno | ≤ ±4 % fino a 60° di incidenza; ≤ ±4,5 % fino a 70° |
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

Il sensore riporta l’intera griglia da 340 a 1010 nm, ma il guadagno radiometrico tracciabile secondo gli standard NIST copre un intervallo di ~374–974 nm. Chloros **rifiuta la divisione in base alla riflettanza assoluta** per qualsiasi banda della fotocamera con meno della metà del proprio peso spettrale all’interno di tale intervallo, segnalando il motivo di salto `dls-uncalibrated-band-<nm>` anziché produrre un risultato non calibrato. Tra gli SKU delle telecamere attualmente in commercio, solo il filtro F988 non rientra in questo intervallo; in questo caso viene utilizzato il flusso di lavoro con pannello di riflettanza — si veda [Flussi di lavoro con riflettanza](reflectance.md).

Per i modelli di sensore, i trasporti e gli ID dei sensori, consultare la [panoramica sul DAQ](README.md). Per informazioni su come viene consumato il cap stamp durante l’elaborazione, consultare [Registrazione e formato .daq](recording.md).
