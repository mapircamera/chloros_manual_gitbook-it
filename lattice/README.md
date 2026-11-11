# Telecamere LATTICE

LATTICE è il sistema modulare di telecamere multispettrali di MAPIR per l&#x27;imaging agricolo e scientifico. Ogni telecamera LATTICE è basata sul sensore global-shutter Sony IMX265 (**3,1 MP, pixel da 3,45 µm**) e si collega tramite Ethernet come dispositivo**GigE Vision**.

Chloros 1.3.0 controlla le telecamere LATTICE in tempo reale — individuazione, anteprima live, acquisizione e array multitelecamera sincronizzati — da tre interfacce:

| Interfaccia    | Dove                                                          | Piattaforme                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Scheda **Telecamere** nella barra laterale di Chloros                         | Windows 10/11 x64                                        |
| CLI        | Famiglia di comandi `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Cerchi l’hardware?**I moduli telecamera, gli obiettivi, i filtri e le bande, i telai e i sistemi di montaggio, i cavi, il PoE e il cablaggio di trigger sono documentati nel [**manuale utente LATTICE**](https://mapir.gitbook.io/lattice-camera). Questo capitolo tratta il controllo delle telecamere da Chloros.

Le acquisizioni LATTICE sono file standard `.tif`/`.tiff` e Chloros le elabora sempre partendo dall’acquisizione grezza. Consultare il [Riferimento CLI](../reference/cli-reference.md) e il [Riferimento SDK](../reference/sdk-reference.md) per l’elenco completo dei comandi e delle interfacce API.

## Due configurazioni dei sensori

| Configurazione | Sensore       | Filtro                                | Cosa fornisce una telecamera                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Colore Bayer | filtro a tripla banda passante                |**Tre bande calibrate da una singola esposizione**                 |
| **M3M**| Monocromatica   | filtro a interferenza a banda stretta singola |**Una banda calibrata**; combinare più telecamere M3M per ottenere indici |

Poiché una telecamera M3M è monocromatica dietro un unico filtro, ogni banda ottiene la propria esposizione. Una fotocamera M3C copre tutte e tre le sue bande con un’unica esposizione del sensore.

## Stringhe di modello e denominazione

Ogni fotocamera memorizza la propria identità in GenICam `DeviceUserID` come stringa di modello:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros la visualizza con il prefisso `LATT-` (ad esempio `LATT-M3M-L87-F450`). La stessa stringa `LATT-…` viene inserita nel tag EXIF `Model` di ogni esportazione e viene utilizzata come nome della cartella di output della fotocamera nei progetti elaborati.

| Componente | Valori                                                   | Significato                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensore    | `M3C` / `M3M`                                            | Colore Bayer / monocromatico                                                                          |
| Obiettivo | `L41` / `L87`                                            | Il numero indica il **campo visivo orizzontale in gradi**: L41 = stretto (41°), L87 = ampio (87°)    |
| Filtro    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) o `F<nm>` (M3M) | Vedi [Filtri e bande spettrali](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

La stringa del modello determina tutto ciò che segue: Chlorosa il profilo del sensore, la disposizione delle bande e la calibrazione di fabbrica da `DeviceUserID` + `DeviceSerialNumber`. Non c’è nulla da configurare per ciascuna telecamera — vedi [Collegamento delle telecamere](connecting.md).

## Filtri e bande

I centri delle bande, i bordi FWHM e l’intero catalogo M3M con 23 SKU sono specifiche del prodotto, pertanto sono riportati nel manuale dell’hardware: [**Filtri e bande spettrali**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Ciò che conta dal punto di vista del software: il codice del filtro nella stringa del modello determina quali prodotti Chloros può generare. Le telecamere con filtro RGB (`FRGB`) emettono solo prodotti debayered e di anteprima — la radianza e la riflettanza per banda non sono significative per un sensore a banda larga, quindi Chloros le omette e lo specifica. Ogni altro filtro produce la catena completa radianza → riflettanza → indice.

## Panoramica sulla calibrazione radiometrica

Ogni telecamera LATTICE viene calibrata singolarmente in fabbrica rispetto a una catena tracciabile secondo gli standard NIST e viene fornita con un certificato specifico per ogni telecamera. Cosa comprende tale calibrazione, come viene misurata e quale precisione è possibile dichiarare sono indicati nel manuale dell’hardware: [**Calibrazione radiometrica di fabbrica**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Dal punto di vista del software, ciò che conta è che Chloros determini la calibrazione corretta quando una telecamera si connette e fissi i coefficienti applicati in ogni esportazione — vedere [Connessione delle telecamere](connecting.md).

## In questo capitolo

* [Collegamento delle telecamere](connecting.md) — rilevamento automatico, finestra di dialogo di connessione dell’interfaccia grafica, equivalenti CLI/SDK e modalità di determinazione della calibrazione di fabbrica (pacchetto integrato nella telecamera vs. cloud) quando una telecamera si collega.
* [Impostazioni della telecamera](camera-settings.md) — tutti i controlli nella scheda “Telecamere”: l’elenco delle telecamere, il pannello delle impostazioni per singola telecamera e il pannello delle impostazioni dell’array (compresi l’editor ROI e l’allineamento).
* [Impostazioni e modalità di acquisizione](capture.md) — Acquisizione totale, acquisizione singola / continua / a intervalli, tipi di esportazione e struttura della cartella delle acquisizioni.
* [Array multicamera](arrays.md) — la procedura guidata di connessione dell’array, la sonda del cavo di sincronizzazione, la larghezza di banda, il PTP e la visualizzazione in tempo reale dell’array.
* [Telecamere mono e indici di vegetazione](mono-indices.md) — Telecamere M3M e flusso di lavoro allineamento → impilamento → indice.

L’interfaccia completa dei comandi è disponibile nella [Guida di riferimento CLI](../reference/cli-reference.md) e nella [Guida di riferimento SDK](../reference/sdk-reference.md).
