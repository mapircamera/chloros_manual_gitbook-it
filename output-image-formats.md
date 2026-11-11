---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formati immagine di output

Chloros esporta i prodotti elaborati in quattro formati di file. Selezionare il formato nelle Impostazioni del progetto (GUI), con `--format` (CLI) o con `export_format` (SDK). La CLI e l&#x27;SDK accettano le stringhe esatte riportate di seguito.

| Stringa di formato | Estensione | Tipo di pixel | Intervallo di pixel | Note |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(predefinito)* | `.tif` | numero digitale uint16 | 0 – 65535 | Consigliato per fotogrammetria / GIS. |
| `TIFF (8-bit)` | `.tif` | numero digitale uint8 | 0 – 255 | Per tutte le acquisizioni a 8 bit, che offrono solo i formati a 8 bit: Chloros non aumenta la profondità di bit. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = riflettanza al 100%. Alcune applicazioni non sono in grado di leggere i file TIFF in virgola mobile; i file sono più grandi. |
| `PNG (8-bit)` | `.png` | numero digitale uint8 | 0 – 255 | Compressione senza perdita di dati, adatta alla visualizzazione sul web e alla rappresentazione grafica. |
| `JPG (8-bit)` | `.jpg` | numero digitale uint8 | 0 – 255 | Compressione con perdita di dati, file di dimensioni minime. |

## Dove vengono salvati i file di output

I file vengono salvati nella cartella del progetto, raggruppati per fotocamera e poi per formato di file:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

La cartella della telecamera è `LATT-<sensor>-<lens>-F<filter>` per LATTICE e `<model>_<filter>` (ad es. `Survey3N_RGN`) per Survey3. **Ogni prodotto esportato mantiene il nome del file sorgente: è la cartella a identificare il prodotto, non il suffisso del nome del file.** Per le regole complete, consultare [Dove vengono salvati i file di output](reference/cli-reference.md) nella Guida di riferimento CLI.

## Prodotti LATTICE (livelli di acquisizione ed esportazione)

Un fotogramma grezzo di LATTICE viene suddiviso in tutti i prodotti richiesti in un unico passaggio. Ogni tipo di prodotto ha il proprio interruttore (caselle di controllo nella GUI o comandi CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, tutti attivi per impostazione predefinita):

| Livello | Contenuto | Tipo di dati |
| --- | --- | --- |
| `raw` | Dati Bayer direttamente dal sensore (telecamere monocromatiche: la singola banda). L’elaborazione parte sempre dai dati grezzi. | Come acquisiti |
| `debayered` | Demosaicatura lineare — a 3 canali per M3C, in scala di grigi a 1 canale per M3M: viene rimosso l’offset di scuro del pacchetto di calibrazione, quindi viene eseguita una demosaicatura sensibile ai bordi, senza correzione di linearità, flat field o unmix. Un’esportazione `process` scrive lo stesso file salvato da un’acquisizione a questo livello, in ogni modalità di elaborazione. | DN lineare |
| `radiance` | Radianza spettrale assoluta dalla catena radiometrica completa, in **W/m²/sr/nm**. Viene sempre salvato come TIFF a 32 bit (`tiff32/Radiance_Images/`), indipendentemente dal formato di esportazione selezionato. | float32 |
| `reflectance` | Riflettanza ρ, dove **DN 32768 = ρ 1,0 (100%)** con margine fino a ρ 2,0. Compatibile con Pix4D. | uint16 |
| `preview` | Rendering pronto per la visualizzazione: RGB = bilanciamento del bianco + gamma; multispettrale = espansione in falsi colori. | display a 8 bit |

### Orientamento

I file `raw` e `debayered` mantengono sempre l’orientamento **nativo** del sensore; l’impostazione di capovolgimento/rotazione della fotocamera viene applicata solo agli altri prodotti (anteprima, radianza, riflettanza, indice). Le acquisizioni in array mantengono i file di ogni elemento nell’orientamento nativo, a meno che non si richieda l’orientamento (`--orient` nell’CLI, `orient=True` nell’SDK). Ogni file registra l’orientamento dei propri pixel in XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — mentre `ChlorosOrientationApplied` indica se i propri pixel lo contengono (`false` nei file raw e debayered). Chloros Image Viewer ruota i file raw e debayered in base a quel record solo ai fini della visualizzazione; i file stessi non vengono mai riscritti. Un file privo di tale record (creato prima della versione 1.3.0 o con un altro strumento) viene visualizzato così come è stato salvato.

## Maschere di clipping

I prodotti di radianza e riflettanza di una fotocamera multispettrale M3C eseguono una separazione delle bande 3×3. Laddove la separazione produce un valore negativo in una banda, tale valore viene troncato a zero (le visualizzazioni in tempo reale applicano il clipping esattamente allo stesso modo). Novità della versione 1.3.0: un prodotto in cui almeno un pixel è stato troncato riceve una **maschera di limitazione** associata che indica esattamente quali pixel e bande sono interessati:

* **Dove:**in una cartella `clamp_masks/`**accanto** alle cartelle dei prodotti, mai all’interno di una di esse: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif` e `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Cosa:** un file TIFF a banda singola `uint8` sulla griglia di pixel del prodotto stesso (compresi orientamento, ROI, binning e allineamento dell’array). Il bit 0 (`mask & 1`) viene impostato nel punto in cui la banda nel canale R del prodotto è stata ritagliata, il bit 1 (`& 2`) indica il canale G, il bit 2 (`& 4`) indica il canale B. La percentuale di pixel clippati per ciascuna banda è inoltre indicata sul prodotto come `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` e `Chloros:BandBUnmixClampFraction`.
* **Come indicato nel prodotto:** il tag XMP `Chloros:UnmixClampMask` contiene il percorso della maschera relativo alla cartella del prodotto (`../clamp_masks/<product folder>/<name>_clampmask.tif`, sempre separato da `/`), `none` quando nessun pixel è stato ritagliato (non viene scritta alcuna maschera), oppure `unavailable` quando non è stato possibile scrivere una maschera. Un prodotto senza tag non ha mai eseguito la separazione dei colori (telecamere RGB e monocromatiche, livelli raw e debayered).

Poiché nessuna cartella del prodotto contiene una maschera, è possibile fornire a Pix4D, Metashape o ad altri software di fotogrammetria una cartella del prodotto come `Reflectance_Calibrated_Images/` così com’è, sottocartelle incluse — fornire la cartella del prodotto, non la cartella del formato che la contiene. L’importazione di Chloros ignora le maschere di clamp. La CLI le conta separatamente: `Image products written: N (plus M clamp masks)`.

## Prodotti di indice

Ogni indice viene inserito nella propria cartella `<INDEX>_Index_Images/` (ad es. `NDVI_Index_Images/`).

* **I formati interi**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) memorizzano un**allungamento lineare per singola immagine**, non una mappatura −1…1: DN 0 è il valore di indice più basso di quel fotogramma e la scala completa è il più alto. Il file contiene `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` e `Chloros:IndexMax`, quindi anche `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Poiché ogni fotogramma ha i propri valori limite, confrontare i fotogrammi in base ai valori, non al DN.
* **`TIFF (32-bit, Percent)`** memorizza i valori di indice stessi come float32, senza riscalatura — la scelta migliore per analisi quantitative su un intero volo.

Novità della versione 1.3.0: un prodotto di indice descrive **se stesso** anziché ripetere i tag dei dati grezzi di origine. I suoi tag GPS e della fotocamera vengono copiati dal prodotto da cui è stato calcolato (il prodotto di riflettanza, se presente), e quindi:

| Tag | Valore |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (in modo che il file non venga mai reimportato come acquisizione) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / la formula utilizzata |
| `Chloros:IndexLevel` | `reflectance` oppure `radiance` — da cosa è stato calcolato l’indice (assente se non è presente nessuno dei due) |
| `Chloros:IlluminationInvariant` | `True` per un indice calcolato dalla riflettanza, `False` dalla radianza o dai valori del sensore |
| `Chloros:ChlorosPipelineVersion` | la versione di Chloros che ha generato l’indice |

`Chloros:PixelScale` è stato rimosso dai prodotti dell’indice — utilizzare `IndexMin` / `IndexMax` sopra indicati. La procedura completa di lettura è disponibile nella sezione &quot;Un raster con indice intero è un&#x27;estensione del contrasto per singola immagine&quot; nella [Guida di riferimento CLI](reference/cli-reference.md).

## Lettura dei valori di riflettanza dei pixel

La riflettanza è memorizzata come numero digitale intero (DN) e **il DN che corrisponde a ρ = 1,0 (riflettanza al 100%) dipende dalla fotocamera di origine**:

| Fotocamera di origine | ρ = 1,0 corrisponde a DN | Come determinarlo |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margine fino a ρ 2,0) | Il tag XMP `Chloros:PixelScale=32768` è impresso sul file. |
| Survey3 | `65535` (tagliato a ρ 1,0) | Nessun tag `Chloros:PixelScale` — tale assenza è l’indicatore. (I prodotti Survey3 riportano altri tag di elaborazione `Chloros:*`, come `Chloros:CalibrationApplied`, in ogni modalità di elaborazione a partire dalla versione 1.3.0.) |

**Leggere il tag XMP `Chloros:PixelScale` e dividere per il suo valore** anziché ipotizzare una costante. Il tag è definito nel dominio uint16, quindi rimane `32768` in tutti i formati di output che effettuano un ridimensionamento — normalizza prima il tipo di dati memorizzato riportandolo a uint16 (×257 da 8 bit, ×65535 da float32).

{% hint style="warning" %}
**Acquisizioni a 8 bit.** Le esportazioni di un’acquisizione a 8 bit sono a 8 bit. La sua riflettanza viene ridimensionata e contrassegnata con `Chloros:PixelScale=32768`, in modo che la scala rimanga invariata; i prodotti DN a 8 bit e i file di riflettanza scritti prima della versione 1.3.0 vengono limitati a 0–255 e non riportano alcun tag di scala. Se il tag è assente in un file di riflettanza LATTICE, non dare per scontata l’esistenza di una scala; riesportare invece a 16 bit o 32 bit.
{% endhint %}

Per le regole complete (compresi i tag compatibili con MicaSense), consultare **&quot;Lettura dei pixel di riflettanza&quot;** nella [Guida di riferimento CLI](reference/cli-reference.md).
