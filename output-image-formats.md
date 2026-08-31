---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formati immagine di output

Chloros esporta i prodotti elaborati in quattro formati di file. Selezionare il formato nelle Impostazioni del progetto (GUI), con `--format` (CLI) o con `export_format` (SDK). CLI e SDK accettano le stringhe esatte riportate di seguito.

| Stringa di formato | Estensione | Tipo di pixel | Intervallo di pixel | Note |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(predefinito)* | `.tif` | numero digitale uint16 | 0 – 65535 | Consigliato per fotogrammetria / GIS. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = riflettanza al 100%. Alcune applicazioni non sono in grado di leggere file TIFF in virgola mobile; i file sono più grandi. |
| `PNG (8-bit)` | `.png` | numero digitale uint8 | 0 – 255 | Compressione senza perdita di dati, adatta alla visualizzazione sul web e alla rappresentazione grafica. |
| `JPG (8-bit)` | `.jpg` | numero digitale uint8 | 0 – 255 | Compressione con perdita, file più piccoli. |

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

La cartella della telecamera è `LATT-<sensor>-<lens>-F<filter>` per LATTICE e `<model>_<filter>` (ad es. `Survey3N_RGN`) per Survey3. **Ogni prodotto esportato mantiene il nome del file di origine: è la cartella a identificare il prodotto, non un suffisso del nome del file.** Consultare [Dove vengono salvati i risultati](reference/cli-reference.md) nella Guida di riferimento di CLI per le regole complete.

## Prodotti LATTICE (livelli di acquisizione ed esportazione)

Un frame grezzo LATTICE viene suddiviso in tutti i prodotti richiesti in un unico passaggio. Ogni tipo di prodotto dispone di un proprio interruttore (caselle di controllo dell’interfaccia grafica o CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, tutti attivi per impostazione predefinita):

| Livello | Contenuto | Tipo di dati |
| --- | --- | --- |
| `raw` | Dati Bayer direttamente dal sensore (telecamere monocromatiche: la singola banda). L’elaborazione parte sempre dai dati grezzi. | Come acquisiti |
| `debayered` | Demosaicatura lineare — a 3 canali per M3C, in scala di grigi a 1 canale per M3M. | DN lineare |
| `radiance` | Radianza spettrale assoluta derivata dall’intera catena radiometrica, in **W/m²/sr/nm**. Sempre scritto come TIFF a 32 bit (`tiff32/Radiance_Images/`), indipendentemente dal formato di esportazione selezionato. | float32 |
| `reflectance` | Riflettanza ρ, dove **DN 32768 = ρ 1,0 (100%)** con margine fino a ρ 2,0. Pronto per Pix4D. | uint16 |
| `preview` | Rendering pronto per la visualizzazione: RGB = bilanciamento del bianco + gamma; multispettrale = estensione in falsi colori. | visualizzazione a 8 bit |

## Lettura dei valori dei pixel di riflettanza

La riflettanza è memorizzata come numero digitale intero e **il DN che corrisponde a ρ = 1,0 (riflettanza al 100%) dipende dalla fotocamera di origine**:

| Fotocamera di origine | ρ = 1,0 corrisponde a DN | Come determinarlo |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margine fino a ρ 2,0) | Il tag XMP `Chloros:PixelScale=32768` è impresso sul file. |
| Survey3 | `65535` (tagliato a ρ 1,0) | Nessun tag XMP `Chloros:*` — tale assenza è il segnale. |

**Leggi il tag XMP `Chloros:PixelScale` e dividi per esso** invece di ipotizzare un valore costante. Il tag è definito nel dominio uint16, quindi rimane invariato `32768` nei formati di output che ridimensionano — normalizza prima il tipo di dati memorizzato riportandolo a uint16 (×257 da 8 bit, ×65535 da float32).

{% hint style="warning" %}
**Un caso non prevede alcuna scala, per scelta progettuale.** Quando un&#x27;acquisizione da sorgente a 8 bit (BayerRG8) viene scritta come TIFF a 8 bit, la pipeline effettua un clipping nell’intervallo 0–255 invece di ridimensionare, quindi il file non presenta alcuna scala — Chloros omette deliberatamente `Chloros:PixelScale` in quel caso. Se il tag è assente in un file di riflettanza LATTICE, non dare per scontata l’esistenza di una scala; riesportare invece a 16 bit o 32 bit.
{% endhint %}

Per le regole complete (compresi i tag compatibili con MicaSense), consultare **&quot;Lettura dei pixel di riflettanza&quot;** nella [Riferenza](reference/cli-reference.md).
