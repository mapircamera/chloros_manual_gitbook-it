---
description: This page lists some multispectral indices that Chloros uses.
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---
# Formule dell&#x27;indice multispettrale

Le formule dell&#x27;indice riportate di seguito utilizzano una combinazione degli intervalli di trasmissione media del filtro Survey3:

<table><thead><tr><th align="center">Survey3 Colore del filtro</th><th width="196.199951171875" align="center">Survey3 Nome del filtro</th><th width="159.800048828125" align="center">Intervallo di trasmissione (FWHM)</th><th align="center">Trasmissione media</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Quando si utilizzano queste formule, il nome può terminare con &quot;\_1&quot; o &quot;\_2&quot;, che corrisponde al filtro NIR utilizzato, ovvero NIR1 o NIR2.

***

## EVI - Indice di vegetazione potenziato

Questo indice è stato originariamente sviluppato per essere utilizzato con i dati MODIS come miglioramento rispetto a NDVI, ottimizzando il segnale di vegetazione in aree con un elevato indice di area fogliare (LAI). È particolarmente utile nelle regioni con LAI elevato, dove NDVI potrebbe saturarsi. Utilizza la regione di riflettanza blu per correggere i segnali di fondo del suolo e ridurre le influenze atmosferiche, compresa la dispersione degli aerosol.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

I valori EVI dovrebbero essere compresi tra 0 e 1 per i pixel della vegetazione. Caratteristiche luminose come nuvole ed edifici bianchi, insieme a caratteristiche scure come l&#x27;acqua, possono causare valori anomali dei pixel in un&#x27;immagine EVI. Prima di creare un&#x27;immagine EVI, è necessario mascherare le nuvole e gli elementi luminosi dall&#x27;immagine di riflettanza e, facoltativamente, impostare una soglia per i valori dei pixel da 0 a 1.

_Riferimento: Huete, A., et al. &quot;Panoramica delle prestazioni radiometriche e biofisiche degli indici di vegetazione MODIS&quot;. Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Indice di copertura forestale 1

Questo indice distingue le chiome delle foreste da altri tipi di vegetazione utilizzando immagini di riflettanza multispettrale che includono una banda di bordo rosso.

$$
FCI1 = Red * RedEdge
$$

Le aree boschive avranno valori FCI1 inferiori a causa della minore riflettanza degli alberi e della presenza di ombre all&#x27;interno della chioma.

_Riferimento: Becker, Sarah J., Craig S.T. Daughtry e Andrew L. Russ. &quot;Indici robusti di copertura forestale per immagini multispettrali&quot;. Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Indice di copertura forestale 2

Questo indice distingue le chiome forestali da altri tipi di vegetazione utilizzando immagini di riflettanza multispettrale che non includono una banda rossa.

$$
FCI2 = Red * NIR
$$

Le aree boschive avranno valori FCI2 inferiori a causa della minore riflettanza degli alberi e della presenza di ombre all&#x27;interno della chioma.

_Riferimento: Becker, Sarah J., Craig S.T. Daughtry e Andrew L. Russ. &quot;Indici robusti di copertura forestale per immagini multispettrali&quot;. Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Indice di monitoraggio ambientale globale

Questo indice di vegetazione non lineare viene utilizzato per il monitoraggio ambientale globale dalle immagini satellitari e cerca di correggere gli effetti atmosferici. È simile a NDVI, ma è meno sensibile agli effetti atmosferici. È influenzato dal suolo nudo; pertanto, se ne sconsiglia l&#x27;uso in aree con vegetazione rada o moderatamente densa.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Dove:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Riferimento: Pinty, B. e M. Verstraete. GEMI: un indice non lineare per monitorare la vegetazione globale dai satelliti. Vegetation 101 (1992): 15-20._

***

## GARI - Green Indice resistente all&#x27;atmosfera

Questo indice è più sensibile a un&#x27;ampia gamma di concentrazioni di clorofilla e meno sensibile agli effetti atmosferici rispetto a NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

La costante gamma è una funzione di ponderazione che dipende dalle condizioni degli aerosol nell&#x27;atmosfera. ENVI utilizza un valore di 1,7, che è il valore raccomandato da Gitelson, Kaufman e Merzylak (1996, pagina 296).

_Riferimento: Gitelson, A., Y. Kaufman e M. Merzylak. &quot;Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Indice di clorofilla

Questo indice viene utilizzato per stimare il contenuto di clorofilla delle foglie in un&#x27;ampia gamma di specie vegetali.

$$
GCI = {NIR \over Green} - 1
$$

L&#x27;ampiezza delle lunghezze d&#x27;onda NIR e verde consente una migliore previsione del contenuto di clorofilla, garantendo al contempo una maggiore sensibilità e un rapporto segnale/rumore più elevato.

_Riferimento: Gitelson, A., Y. Gritz e M. Merzlyak. &quot;Relazioni tra il contenuto di clorofilla nelle foglie e la riflettanza spettrale e algoritmi per la valutazione non distruttiva della clorofilla nelle foglie delle piante superiori&quot;. Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green Indice fogliare

Questo indice è stato originariamente progettato per essere utilizzato con una fotocamera digitale RGB per misurare la copertura del grano, dove i numeri digitali (DN) rosso, verde e blu vanno da 0 a 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

I valori GLI variano da -1 a +1. I valori negativi rappresentano il suolo e le caratteristiche non viventi, mentre i valori positivi rappresentano le foglie e gli steli verdi.

_Riferimento: Louhaichi, M., M. Borman e D. Johnson. &quot;Piattaforma localizzata spazialmente e fotografia aerea per la documentazione degli impatti del pascolo sul grano&quot;. Geocarto International 16, n. 1 (2001): 65-70._

***

## GNDVI - Green Indice di vegetazione normalizzato

Questo indice è simile a NDVI, tranne per il fatto che misura lo spettro verde da 540 a 570 nm invece dello spettro rosso. Questo indice è più sensibile alla concentrazione di clorofilla rispetto a NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Riferimento: Gitelson, A. e M. Merzlyak. &quot;Telerilevamento della concentrazione di clorofilla nelle foglie delle piante superiori&quot;. Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Indice di vegetazione ottimizzato e corretto in base al suolo

Questo indice è stato originariamente progettato con la fotografia a infrarossi a colori per prevedere il fabbisogno di azoto del mais. È simile a OSAVI, ma sostituisce la banda verde con quella rossa.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Riferimento: Sripada, R., et al. &quot;Determinazione del fabbisogno stagionale di azoto per il mais utilizzando la fotografia aerea a infrarossi a colori&quot;. Tesi di dottorato, North Carolina State University, 2005._

***

## GRVI - Green Indice di vegetazione del rapporto

Questo indice è sensibile ai tassi di fotosintesi nelle chiome delle foreste, poiché le riflettanze del verde e del rosso sono fortemente influenzate dai cambiamenti nei pigmenti fogliari.

$$
GRVI = {NIR \over Green }
$$

_Riferimento: Sripada, R., et al. &quot;Aerial Color Infrared Photography for Determining Early In-season Nitrogen Requirements in Corn.&quot; Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Indice di vegetazione corretto per il suolo

Questo indice è stato originariamente progettato con la fotografia a infrarossi a colori per prevedere il fabbisogno di azoto del mais. È simile a SAVI, ma sostituisce la banda verde con quella rossa.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Riferimento: Sripada, R., et al. &quot;Determinazione del fabbisogno stagionale di azoto per il mais utilizzando la fotografia aerea a infrarossi a colori&quot;. Tesi di dottorato, North Carolina State University, 2005._

***

## LAI - Indice di area fogliare

Questo indice viene utilizzato per stimare la copertura fogliare e per prevedere la crescita e la resa delle colture. ENVI calcola il verde LAI utilizzando la seguente formula empirica di Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Dove EVI è:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

I valori elevati di LAI variano in genere da circa 0 a 3,5. Tuttavia, quando la scena contiene nuvole e altri elementi luminosi che producono pixel saturi, i valori di LAI possono superare 3,5. Idealmente, è opportuno mascherare le nuvole e gli elementi luminosi dalla scena prima di creare un&#x27;immagine LAI.

_Riferimento: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde e A. Thomsen. &quot;Dati multispettrali aerei per quantificare l&#x27;indice di area fogliare, la concentrazione di azoto e l&#x27;efficienza fotosintetica in agricoltura&quot;. Remote Sensing of Environment 81, n. 2-3 (2002): 179-193._

***

## LCI - Indice di clorofilla fogliare

Questo indice viene utilizzato per stimare il contenuto di clorofilla nelle piante superiori, sensibili alle variazioni di riflettanza causate dall&#x27;assorbimento della clorofilla.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Riferimento: Datt, B. &quot;Telerilevamento del contenuto d&#x27;acqua nelle foglie di eucalipto&quot;. Journal of Plant Physiology 154, n. 1 (1999): 30-36._

***

## MNLI - Indice non lineare modificato

Questo indice è un miglioramento dell&#x27;indice non lineare (NLI) che incorpora l&#x27;indice di vegetazione corretto per il suolo (SAVI) per tenere conto dello sfondo del suolo. ENVI utilizza un valore del fattore di correzione dello sfondo della chioma (_L_) pari a 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Riferimento: Yang, Z., P. Willis e R. Mueller. &quot;Impatto dell&#x27;immagine AWIFS potenziata con rapporto di banda sulla precisione della classificazione delle colture&quot;. Atti del Simposio sul telerilevamento Pecora 17 (2008), Denver, CO._

***

## MSAVI2 - Indice di vegetazione modificato e corretto in base al suolo 2

Questo indice è una versione semplificata dell&#x27;indice MSAVI proposto da Qi, et al (1994), che migliora l&#x27;indice di vegetazione corretto in base al suolo (SAVI). Riduce il rumore del suolo e aumenta la gamma dinamica del segnale della vegetazione. MSAVI2 si basa su un metodo induttivo che non utilizza un valore _L_ costante (come nel caso di SAVI) per evidenziare la vegetazione sana.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Riferimento: Qi, J., A. Chehbouni, A. Huete, Y. Kerr e S. Sorooshian. &quot;A Modified Soil Adjusted Vegetation Index.&quot; Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE- Differenza normalizzata RedEdge

Questo indice è simile a NDVI, ma confronta il contrasto tra NIR e RedEdge invece che con Red, che spesso rileva prima lo stress della vegetazione.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Indice di vegetazione normalizzato differenziale

Questo indice è una misura della vegetazione sana e verde. La combinazione della sua formulazione normalizzata differenziale e l&#x27;uso delle regioni di assorbimento e riflettanza più elevate della clorofilla lo rendono robusto in un&#x27;ampia gamma di condizioni. Tuttavia, può saturarsi in condizioni di vegetazione densa quando LAI diventa elevato.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Il valore di questo indice varia da -1 a 1. L&#x27;intervallo comune per la vegetazione verde è compreso tra 0,2 e 0,8.

_Riferimento: Rouse, J., R. Haas, J. Schell e D. Deering. Monitoraggio dei sistemi di vegetazione nelle Grandi Pianure con ERTS. Terzo Simposio ERTS, NASA (1973): 309-317._

***

## NLI - Indice non lineare

Questo indice presuppone che la relazione tra molti indici di vegetazione e i parametri biofisici superficiali sia non lineare. Linearizza le relazioni con i parametri superficiali che tendono ad essere non lineari.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Riferimento: Goel, N. e W. Qin. &quot;Influenze dell&#x27;architettura della chioma sulle relazioni tra vari indici di vegetazione e LAI e Fpar: una simulazione al computer&quot;. Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Indice di vegetazione ottimizzato e corretto in base al suolo

Questo indice si basa sull&#x27;indice di vegetazione corretto in base al suolo (SAVI). Utilizza un valore standard di 0,16 per il fattore di correzione dello sfondo della chioma. Rondeaux (1996) ha stabilito che questo valore fornisce una maggiore variazione del suolo rispetto a SAVI per una copertura vegetale bassa, dimostrando al contempo una maggiore sensibilità alla copertura vegetale superiore al 50%. Questo indice è particolarmente indicato per aree con vegetazione relativamente rada, dove il suolo è visibile attraverso la copertura vegetale.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Riferimento: Rondeaux, G., M. Steven e F. Baret. &quot;Ottimizzazione degli indici di vegetazione corretti per il suolo&quot;. Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Indice di vegetazione differenziale rinormalizzato

Questo indice utilizza la differenza tra le lunghezze d&#x27;onda del vicino infrarosso e del rosso, insieme all&#x27;NDVI, per evidenziare la vegetazione sana. È insensibile agli effetti del suolo e alla geometria di osservazione del sole.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Riferimento: Roujean, J. e F. Breon. &quot;Stima della PAR assorbita dalla vegetazione da misurazioni di riflettanza bidirezionale&quot;. Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Indice di vegetazione corretto per il suolo

Questo indice è simile a NDVI, ma sopprime gli effetti dei pixel del suolo. Utilizza un fattore di aggiustamento dello sfondo della chioma, _L_, che è una funzione della densità della vegetazione e spesso richiede una conoscenza preliminare della quantità di vegetazione. Huete (1988) suggerisce un valore ottimale di _L_=0,5 per tenere conto delle variazioni di primo ordine dello sfondo del suolo. Questo indice è più adatto alle aree con vegetazione relativamente rada, dove il suolo è visibile attraverso la copertura vegetale.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Riferimento: Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Indice di vegetazione trasformato

Questo indice è utile per monitorare la copertura vegetale in ambienti urbani. Non si satura come NDVI e SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Riferimento: Bannari, A., H. Asalhi e P. Teillet. &quot;Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping&quot; In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI - Indice visibile resistente all&#x27;atmosfera

Questo indice si basa sull&#x27;ARVI e viene utilizzato per stimare la frazione di vegetazione in una scena con bassa sensibilità agli effetti atmosferici.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Riferimento: Gitelson, A., et al. &quot;Linee di vegetazione e suolo nello spazio spettrale visibile: un concetto e una tecnica per la stima remota della frazione di vegetazione. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - Indice di vegetazione ad ampio intervallo dinamico

Questo indice è simile a NDVI, ma utilizza un coefficiente di ponderazione (_a_) per ridurre la disparità tra i contributi dei segnali nel vicino infrarosso e nel rosso a NDVI. L&#x27;WDRVI è particolarmente efficace in scene con densità di vegetazione da moderata ad alta quando l&#x27;NDVI supera 0,6. NDVI tende a stabilizzarsi quando la frazione di vegetazione e l&#x27;indice di area fogliare (LAI) aumentano, mentre WDRVI è più sensibile a una gamma più ampia di frazioni di vegetazione e alle variazioni di LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Il coefficiente di ponderazione (_a_) può variare da 0,1 a 0,2. Henebry, Viña e Gitelson (2004) raccomandano un valore di 0,2.

_Riferimenti_

_Gitelson, A. &quot;Indice di vegetazione ad ampio intervallo dinamico per la quantificazione remota delle caratteristiche biofisiche della vegetazione&quot;. Journal of Plant Physiology 161, n. 2 (2004): 165-173._

_Henebry, G., A. Viña e A. Gitelson. &quot;The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis.&quot; Gap Analysis Bulletin 12: 50-56._
