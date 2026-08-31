---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formule degli indici multispettrali

Le formule degli indici riportate di seguito utilizzano una combinazione degli intervalli di trasmissione media del filtro Survey3:

<table><thead><tr><th align="center">Colore del filtro Survey3</th><th width="196.199951171875" align="center">Survey3 Nome del filtro</th><th width="159.800048828125" align="center">Intervallo di trasmissione (FWHM)</th><th align="center">Trasmissione media</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN - Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>Quando si utilizzano queste formule, il nome può terminare con &quot;\_1&quot; o &quot;\_2&quot;, a seconda che sia stato utilizzato il filtro NIR, NIR1 o NIR2.

Per le fotocamere LATTICE M3C (triplo passa-banda Bayer), lo stesso motore di indicizzazione utilizza le bande del filtro M3C:

| Filtro M3C | Banda 1 (centro/FWHM) | Banda 2 (centro/FWHM) | Banda 3 (centro/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

Le telecamere LATTICE M3M sono monobanda (un filtro a banda stretta per telecamera), pertanto gli indici multibanda non vengono calcolati per una singola immagine M3M. Per calcolare gli indici con M3M, combinare due o più telecamere in uno stack multibanda allineato e utilizzare il motore di calcolo degli indici LATTICE (`chloros-cli lattice index` o il Calcolatore di indici in tempo reale dell’interfaccia grafica).

***

## Dove funziona ciascun nome di indice

Chloros dispone di **tre** superfici di indice e i relativi elenchi predefiniti non sono identici. Utilizzate questa sezione per verificare se un nome funzionerà nel contesto in cui intendete utilizzarlo.

| Dove vi trovate | Quale elenco si applica | Conteggio |
| --- | --- | --- |
| Impostazioni progetto → Indice → Aggiungi indice (interfaccia grafica) | Superficie 1 | 27 |
| Visualizzatore immagini [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (interfaccia grafica) | Superficie 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Superficie 2 | 22 |
| SDK `process_folder(indices=[...])` | Superficie 2 | 22 |
| `chloros-cli lattice index --preset` | Superficie 3 | 22 (un 22 diverso) |
| Scheda “Telecamere” – Calcolatore indice in tempo reale | Superficie 3 | 22 (un 22 diverso) |

Le superfici 1 e 2 elaborano **un&#x27;immagine alla volta proveniente da una singola telecamera**, utilizzando gli slot dei simboli `x`/`y`/`z`(/`a`) associati ai canali di filtro di quella telecamera. Surface 3 opera su uno**stack multibanda allineato** — diverse telecamere LATTICE co-registrate in un unico cubo — e fa riferimento ai canali tramite nomi in minuscolo.

### 1. Impostazioni del progetto nell’interfaccia grafica / Menu a tendina “Sandbox” del visualizzatore di immagini — 27 formule

Il menu a tendina le elenca in questo ordine (si tratta dell’ordine di inserimento, non alfabetico):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

Nell’interfaccia grafica si trascinano i canali dei filtri della propria telecamera sugli slot delle bande della formula, in modo che qualsiasi formula possa essere utilizzata con qualsiasi assegnazione di banda supportata dalla propria telecamera. Le formule personalizzate che sono state salvate vengono aggiunte sotto questo elenco.

Le **cinque formule disponibili solo nell’interfaccia grafica** — quelle che l’elenco CLI/SDK `--indices` non accetta — sono implementate come segue:

| Preimpostazione solo GUI | Formula (come implementata) | Slot |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (quattro slot) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

La mappatura prevista per ciascuno di essi è indicata in una sezione dedicata più avanti in questa pagina (ad esempio, GARI prevede x=Green, y=NIR, z=Blue, a=Red). GARI è l’unica formula in Chloros che utilizza un quarto slot.

### 2. CLI / SDK Espansione del nome `--indices` — 22 impostazioni predefinite

L’opzione `chloros-cli process --indices` (e il parametro SDK `indices`) accetta i seguenti nomi di impostazioni predefinite:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**I nomi di indice sconosciuti vengono ignorati senza avviso.** Un nome non presente in questo elenco (comprese le cinque formule disponibili solo nella GUI `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` e qualsiasi formula personalizzata salvata nell’interfaccia grafica) viene ignorato con una semplice segnalazione nel log: l’esecuzione prosegue senza quell’indice e viene comunque segnalata come riuscita. La segnalazione viene visualizzata come:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

La corrispondenza dei nomi non tiene conto delle maiuscole/minuscole dopo la rimozione degli spazi, quindi `ndvi`, `NDVI` e ` NDVI ` sono lo stesso preset. Un preset viene inoltre saltato se richiede una banda che il filtro della fotocamera non fornisce.
{% endhint %}

Le formule esatte così come implementate (i simboli `x`/`y`/`z` rappresentano gli slot di banda; la mappatura predefinita è indicata per ciascun preset):

| Preset | Formula (come implementata) | Filtro predefinito | Slot (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Come un nome predefinito viene tradotto in posizioni di banda

Quando si passa un nome semplice come `NDVI`, Chloros deve decidere quale canale di quale file legga ciascun simbolo. A tal fine utilizza questa tabella, che associa un codice filtro alla posizione nell&#x27;array di ciascun canale:

| Codice filtro | Canale → indice dell’array |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` è accettato come alias per Orange, anch’esso 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

Il **filtro predefinito** del preset (la colonna &quot;Filtro predefinito&quot; sopra) viene utilizzato quando il progetto contiene immagini con quel filtro. In caso contrario, Chloros esamina i filtri effettivamente presenti nel progetto nell’ordine `RGN, OCN, NGB, RGB, RE, NIR` e seleziona il primo in grado di fornire tutti i canali richiesti dal preset. Se nessuno di essi è in grado di farlo, il preset viene scartato per quella esecuzione. Questo è il motivo per cui `NDVI` richiesto su un set di dati contenente solo OCN produce comunque un risultato sensato: si associa alle posizioni Orange e alle posizioni di NIR.

Le stringhe del modello LATTICE M3C contengono il filtro con un prefisso `F` (`LATT-M3C-L41-FRGN`), ma il prefisso viene omesso quando il codice del filtro viene letto dall’immagine; pertanto, una telecamera FRGN effettua la risoluzione tramite la riga `RGN` sovrastante e non richiede alcuna gestione speciale.

### 3. Motore di indicizzazione LATTICE (`lattice index --preset`, calcolatore di indici in tempo reale) — 22 impostazioni predefinite

Il motore LATTICE opera su stack multibanda allineati (array in tempo reale o file TIFF multibanda esportati) e utilizza nomi di canali in minuscolo (`red`, `green`, `blue`, `red_edge`, `nir`). Il suo elenco di preimpostazioni differisce dai due precedenti:

| Preimpostazione | Formula | Canali |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | rosso, nir |
| GNDVI | `(nir - green) / (nir + green)` | verde, nir |
| BNDVI | `(nir - blue) / (nir + blue)` | blu, nir |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | rosso\_bordo, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | blu, verde, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | rosso, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | rosso, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | rosso, nir |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | blu, rosso, nir |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | rosso, NIR |
| CVI | `(nir / green) - (red / green)` | rosso, verde, NIR |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | rosso, nir |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | rosso, NIR |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | rosso, verde, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | rosso, verde, blu |
| NGRDI | `(green - red) / (green + red)` | rosso, verde |
| VARI | `(green - red) / (green + red - blue)` | rosso, verde, blu |
| TGI | `green - 0.39*red - 0.61*blue` | rosso, verde, blu |
| EXG | `2*green - red - blue` | rosso, verde, blu |
| CIRE | `(nir / red_edge) - 1` | rosso_bordo, infrarosso |
| CIGREEN | `(nir / green) - 1` | verde, nir |
| NDWI | `(green - nir) / (green + nir)` | verde, nir |

Eseguire `chloros-cli lattice index --list-presets` per stampare questa tabella dalla versione installata e `--list-gradients` per i gradienti di colore disponibili. I simboli dei canali distinguono tra maiuscole e minuscolee devono corrispondere ai nomi in minuscolo dei preset (ad es. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Come implementato nell’interfaccia grafica e nell’elenco dei preset CLI/SDK, CVI è la formula del rapporto dei rapporti:

$$
CVI = {(z / y) \over (x / y)}
$$

con la mappatura dei canali predefinita RGB x=Red, y=Green, z=Blue. Nell&#x27;interfaccia grafica è possibile trascinare uno qualsiasi dei canali della propria telecamera negli slot x/y/z. Si noti che il preset `CVI` del motore di indici LATTICE utilizza una formula diversa, `(NIR / Green) - (Red / Green)`: consultare le tabelle sopra riportate per la superficie che si sta utilizzando.

***

## ENDVI - Indice di vegetazione normalizzato migliorato

Questo indice utilizza il canale blu oltre a NIR e al verde, ed è molto diffuso con le fotocamere filtrate con NGB, dove la banda blu sostituisce quella rossa.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

L’implementazione è la formula simbolica `((x+y)-(2*z))/((x+y)+(2*z))` — assegnare i canali della propria fotocameraai canali x/y e quello di Blue a z (per una fotocamera NGB: x=NIR, y=Green, z=Blue).

***

## EVI - Indice di vegetazione potenziato

Questo indice è stato originariamente sviluppato per l’uso con i dati MODIS come miglioramento rispetto a NDVI, ottimizzando il segnale di vegetazione nelle aree con elevato indice di area fogliare (LAI). È particolarmente utile nelle regioni con valori elevati di LAI, dove l’NDVI potrebbe saturarsi. Utilizza la regione di riflettanza blu per correggere i segnali di fondo del suolo e per ridurre le influenze atmosferiche, compresa la dispersione degli aerosol.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

I valori di EVI dovrebbero variare da 0 a 1 per i pixel di vegetazione. Elementi luminosi come le nuvole e gli edifici bianchi, insieme a elementi scuri come l’acqua, possono causare valori anomali dei pixel in un’immagine EVI. Prima di creare un’immagine EVI, è necessario mascherare le nuvole e gli elementi luminosi dall’immagine di riflettanza e, facoltativamente, applicare una soglia ai valori dei pixel compresa tra 0 e 1.

_Riferimento: Huete, A., et al. &quot;Panoramica sulle prestazioni radiometriche e biofisiche degli indici di vegetazione MODIS.&quot; Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Indice di copertura forestale 1

_Solo GUI — non disponibile come preimpostazione CLI/SDK `--indices`._

Questo indice distingue le chiome forestali da altri tipi di vegetazione utilizzando immagini di riflettanza multispettrale che includono una banda “red edge”.

$$
FCI1 = Red * RedEdge
$$

Le aree boschive presenteranno valori FCI1 inferiori a causa della minore riflettanza degli alberi e della presenza di ombre all’interno della chioma.

_Riferimento: Becker, Sarah J., Craig S.T. Daughtry e Andrew L. Russ. &quot;Indici robusti di copertura forestale per immagini multispettrali.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Indice di copertura forestale 2

_Solo interfaccia grafica (GUI) — non disponibile come preimpostazione CLI/SDK `--indices`._

Questo indice distingue le chiome forestali da altri tipi di vegetazione utilizzando immagini di riflettanza multispettrale che non includono una banda “red edge”.

$$
FCI2 = Red * NIR
$$

Le aree boschive presenteranno valori FCI2 inferiori a causa della minore riflettanza degli alberi e della presenza di ombre all’interno della chioma.

_Riferimento: Becker, Sarah J., Craig S.T. Daughtry e Andrew L. Russ. &quot;Indici robusti di copertura forestale per immagini multispettrali&quot;. Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Indice di monitoraggio ambientale globale

_Solo GUI — non disponibile come preimpostazione CLI/SDK `--indices`._

Questo indice di vegetazione non lineare viene utilizzato per il monitoraggio ambientale globale tramite immagini satellitari e cerca di correggere gli effetti atmosferici. È simile a NDVI, ma è meno sensibile agli effetti atmosferici. È influenzato dal suolo nudo; pertanto, se ne sconsiglia l’uso in aree con vegetazione rada o moderatamente densa.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Dove:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Riferimento: Pinty, B. e M. Verstraete. GEMI: un indice non lineare per il monitoraggio della vegetazione globale tramite satellite. Vegetation 101 (1992): 15-20._

***

## GARI - Green Indice resistente alle condizioni atmosferiche

_Solo GUI — non disponibile come preimpostazione CLI/SDK `--indices`._

Questo indice è più sensibile a un ampio intervallo di concentrazioni di clorofilla e meno sensibile agli effetti atmosferici rispetto a NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

La costante gamma è una funzione di ponderazione che dipende dalle condizioni degli aerosol nell’atmosfera. ENVI utilizza un valore pari a 1,7, che è il valore raccomandato da Gitelson, Kaufman e Merzylak (1996, pagina 296).

_Riferimento: Gitelson, A., Y. Kaufman e M. Merzylak. «Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS». Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Indice di clorofilla

Questo indice viene utilizzato per stimare il contenuto di clorofilla fogliare in un’ampia gamma di specie vegetali.

$$
GCI = {NIR \over Green} - 1
$$

L’utilizzo di ampie lunghezze d’onda NIR e nel verde fornisce una migliore previsione del contenuto di clorofilla, garantendo al contempo una maggiore sensibilità e un rapporto segnale-rumore più elevato.

_Riferimento: Gitelson, A., Y. Gritz e M. Merzlyak. «Relazioni tra il contenuto di clorofilla fogliare e la riflettanza spettrale e algoritmi per la valutazione non distruttiva della clorofilla nelle foglie delle piante superiori». Journal of Plant Physiology 160 (2003): 271-282._

***

## Indice fogliare GLI - Green

Questo indice è stato originariamente progettato per l’uso con una fotocamera digitale RGB per misurare la copertura del grano, dove i numeri digitali (DN) per il rosso, il verde e il blu variano da 0 a 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

I valori di GLI variano da -1 a +1. I valori negativi rappresentano il suolo e gli elementi non viventi, mentre quelli positivi rappresentano foglie e steli verdi.

_Riferimento: Louhaichi, M., M. Borman e D. Johnson. «Piattaforma con localizzazione spaziale e fotografia aerea per la documentazione degli impatti del pascolo sul grano». Geocarto International 16, n. 1 (2001): 65-70._

***

## GNDVI - Green Indice di vegetazione normalizzato per differenza

Questo indice è simile a NDVI, tranne per il fatto che misura lo spettro verde da 540 a 570 nm anziché lo spettro rosso. Questo indice è più sensibile alla concentrazione di clorofilla rispetto a NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Riferimento: Gitelson, A. e M. Merzlyak. &quot;Telerilevamento della concentrazione di clorofilla nelle foglie delle piante superiori&quot;. Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Indice di vegetazione ottimizzato e corretto per il suolo

Questo indice è stato originariamente progettato con la fotografia a colori e all’infrarosso per prevedere il fabbisogno di azoto del mais. È simile all’OSAVI, ma sostituisce la banda verde con quella rossa.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Riferimento: Sripada, R., et al. “Determinazione del fabbisogno di azoto durante la stagione di crescita del mais mediante fotografia aerea a colori e all’infrarosso”. Tesi di dottorato, North Carolina State University, 2005._

***

## Indice di vegetazione basato sul rapporto GRVI - Green

Questo indice è sensibile ai tassi di fotosintesi nelle chiome forestali, poiché le riflettanze del verde e del rosso sono fortemente influenzate dalle variazioni dei pigmenti fogliari.

$$
GRVI = {NIR \over Green }
$$

_Riferimento: Sripada, R., et al. &quot;Fotografia aerea a colori e all&#x27;infrarosso per la determinazione del fabbisogno di azoto nelle prime fasi della stagione nel mais.&quot; Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Indice di vegetazione corretto per il suolo

Questo indice è stato originariamente progettato con la fotografia a colori e infrarossi per prevedere il fabbisogno di azoto del mais. È simile a SAVI, ma sostituisce la banda verde con quella rossa.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Riferimento: Sripada, R., et al. “Determinazione del fabbisogno di azoto durante la stagione di crescita del mais mediante fotografia aerea a colori e infrarossi”. Tesi di dottorato, North Carolina State University, 2005._

***

## LAI - Indice di area fogliare

Questo indice viene utilizzato per stimare la copertura fogliare e per prevedere la crescita e la resa delle colture. ENVI calcola l’LAI verde utilizzando la seguente formula empirica tratta da Boegh et al. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Dove EVI è:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

I valori elevati di LAI variano in genere da circa 0 a 3,5. Tuttavia, quando la scena contiene nuvole e altri elementi luminosi che producono pixel saturi, i valori di LAI possono superare 3,5. Idealmente, si dovrebbero mascherare le nuvole e gli elementi luminosi dalla scena prima di creare un&#x27;immagine LAI.

_Riferimento: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde e A. Thomsen. «Dati multispettrali aerei per la quantificazione dell’indice di area fogliare, della concentrazione di azoto e dell’efficienza fotosintetica in agricoltura». Remote Sensing of Environment 81, n. 2-3 (2002): 179-193._

***

## LCI - Indice di clorofilla fogliare

_Solo GUI — non disponibile come preimpostazione CLI/SDK `--indices`._

Questo indice viene utilizzato per stimare il contenuto di clorofilla nelle piante superiori, sensibili alle variazioni di riflettanza causate dall’assorbimento della clorofilla.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Riferimento: Datt, B. &quot;Remote Sensing of Water Content in Eucalyptus Leaves.&quot; Journal of Plant Physiology 154, n. 1 (1999): 30-36._

***

## MNLI - Indice non lineare modificato

Questo indice rappresenta un miglioramento dell’indice non lineare (NLI) che incorpora l’indice di vegetazione corretto per il suolo (SAVI) per tenere conto dello sfondo del suolo. ENVI utilizza un valore del fattore di correzione dello sfondo della chioma (_L_) pari a 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Riferimento: Yang, Z., P. Willis e R. Mueller. «Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy». Atti del Simposio Pecora 17 sul telerilevamento (2008), Denver, CO._

***

## MSAVI2 - Indice di vegetazione corretto per il suolo modificato 2

Questo indice è una versione semplificata dell’indice MSAVI proposto da Qi et al. (1994), che rappresenta un miglioramento rispetto all’indice di vegetazione corretto per il suolo (SAVI). Riduce il rumore del suolo e aumenta la gamma dinamica del segnale della vegetazione. L’MSAVI2 si basa su un metodo induttivo che non utilizza un valore costante di _L_ (come nel caso dell’SAVI) per evidenziare la vegetazione sana.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Riferimento: Qi, J., A. Chehbouni, A. Huete, Y. Kerr e S. Sorooshian. «A Modified Soil Adjusted Vegetation Index». Remote Sensing of Environment 48 (1994): 119-126._

***

## MSR - Rapporto semplice modificato

Questo indice è una modifica del rapporto semplice NIR/Red, progettato per linearizzarne la relazione con i parametri biofisici, ed è più sensibile dell’NDVI in presenza di densità di vegetazione più elevate.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Riferimento: Chen, J. &quot;Valutazione degli indici di vegetazione e di un rapporto semplice modificato per applicazioni boreali&quot;. Canadian Journal of Remote Sensing 22 (1996): 229-242._

***

## NDRE - Differenza normalizzata RedEdge

Questo indice è simile a NDVI, ma confronta il contrasto tra NIR e RedEdge anziché con Red, che spesso rileva lo stress della vegetazione in tempi più rapidi.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Indice di vegetazione a differenza normalizzata

Questo indice è una misura della vegetazione sana e verde. La combinazione della sua formulazione basata sulla differenza normalizzata e l’utilizzo delle regioni di massimo assorbimento e riflettanza della clorofilla lo rendono robusto in un’ampia gamma di condizioni. Può tuttavia saturarsi in condizioni di vegetazione fitta quando LAI assume valori elevati.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Il valore di questo indice varia da -1 a 1. L’intervallo tipico per la vegetazione verde va da 0,2 a 0,8.

_Riferimento: Rouse, J., R. Haas, J. Schell e D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Terzo Simposio ERTS, NASA (1973): 309-317._

***

## NLI - Indice non lineare

Questo indice parte dal presupposto che la relazione tra molti indici di vegetazione e i parametri biofisici di superficie sia non lineare. Esso linearizza le relazioni con i parametri di superficie che tendono ad essere non lineari.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Riferimento: Goel, N. e W. Qin. &quot;Influenze dell’architettura della chioma sulle relazioni tra vari indici di vegetazione e LAI e Fpar: una simulazione al computer.&quot; Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Indice di vegetazione ottimizzato e corretto in base al suolo

Questo indice si basa sull’Indice di vegetazione corretto in base al suolo (SAVI). Utilizza un valore standard di 0,16 per il fattore di correzione dello sfondo della chioma. Rondeaux (1996) ha stabilito che questo valore offre una maggiore variazione del suolo rispetto a SAVI in caso di bassa copertura vegetale, dimostrando al contempo una maggiore sensibilità a una copertura vegetale superiore al 50%. Questo indice trova la sua migliore applicazione in aree con vegetazione relativamente rada, dove il suolo è visibile attraverso la chioma.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Riferimento: Rondeaux, G., M. Steven e F. Baret. «Optimization of Soil-Adjusted Vegetation Indices». Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Indice di vegetazione differenziale rinormalizzato

Questo indice utilizza la differenza tra le lunghezze d’onda del vicino infrarosso e del rosso, insieme all’NDVI, per evidenziare la vegetazione sana. È insensibile agli effetti del suolo e alla geometria di osservazione del sole.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Riferimento: Roujean, J. e F. Breon. &quot;Stima della PAR assorbita dalla vegetazione a partire da misurazioni di riflettanza bidirezionale.&quot; Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Indice di vegetazione corretto per il suolo

Questo indice è simile a NDVI, ma sopprime gli effetti dei pixel del suolo. Utilizza un fattore di correzione dello sfondo della chioma, _L_, che è una funzione della densità della vegetazione e spesso richiede una conoscenza preliminare della quantità di vegetazione. Huete (1988) suggerisce un valore ottimale di _L_=0,5 per tenere conto delle variazioni di primo ordine dello sfondo del suolo. Questo indice trova la sua migliore applicazione in aree con vegetazione relativamente rada, dove il suolo è visibile attraverso la chioma.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Riferimento: Huete, A. «A Soil-Adjusted Vegetation Index (SAVI)». Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Indice di vegetazione della differenza trasformata

Questo indice è utile per il monitoraggio della copertura vegetale in ambienti urbani. Non presenta saturazione come NDVI e SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Riferimento: Bannari, A., H. Asalhi e P. Teillet. «Indice di differenza della vegetazione trasformato (TDVI) per la mappatura della copertura vegetale» in Atti del Simposio sulle geoscienze e il telerilevamento, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI - Indice visibile resistente agli effetti atmosferici

Questo indice si basa sull’ARVI e viene utilizzato per stimare la frazione di vegetazione in una scena con bassa sensibilità agli effetti atmosferici.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Riferimento: Gitelson, A., et al. &quot;Linee di vegetazione e suolo nello spazio spettrale visibile: un concetto e una tecnica per la stima remota della frazione di vegetazione. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - Indice di vegetazione ad ampia gamma dinamica

Questo indice è simile a NDVI, ma utilizza un coefficiente di ponderazione (_a_) per ridurre la disparità tra i contributi dei segnali nel vicino infrarosso e nel rosso all’NDVI. L’WDRVI è particolarmente efficace in scene che presentano una densità di vegetazione da moderata-elevata, quando l’NDVI supera 0,6. L’NDVI tende a stabilizzarsi all’aumentare della frazione di vegetazione e dell’indice di area fogliare (LAI) aumentano, mentre l’WDRVI è più sensibile a un intervallo più ampio di frazioni di vegetazione e alle variazioni di LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Il coefficiente di ponderazione (_a_) può variare da 0,1 a 0,2. Un valore di 0,2 è raccomandato da Henebry, Viña e Gitelson (2004).

_Riferimenti_

_Gitelson, A. «Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation». Journal of Plant Physiology 161, n. 2 (2004): 165-173._

_Henebry, G., A. Viña e A. Gitelson. “The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis.” Gap Analysis Bulletin 12: 50-56._
