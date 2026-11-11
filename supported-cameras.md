---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Telecamere supportate

Chloros elabora le immagini provenienti da due famiglie di telecamere MAPIR su **tutte le piattaforme** (Windows, Linux amd64 e Linux arm64/Jetson):

* **Survey3** — Telecamere Survey3W (grandangolari) e Survey3N (a campo stretto). Input: `RAW+JPG`.
* **LATTICE**— Moduli di telecamere multispettrali M3C e M3M. Input: acquisizioni `.tif`/`.tiff`. Le telecamere LATTICE possono anche essere**controllate in tempo reale** da Chloros — tramite la scheda “Telecamere” dell’interfaccia grafica (Windows) o `chloros-cli lattice` / l’Python SDK (Windows e Linux) — compresi gli array multicamera sincronizzati. Consultare la [guida LATTICE](lattice/).

La pipeline di elaborazione accetta anche file di input `.dng`.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Produttore</th><th width="250">Modello della telecamera</th><th width="138">Modello del filtro</th><th width="187">Tipo di immagine</th></tr></thead><tbody><tr><td><strong>Produttore</strong></td><td><strong>Modello della fotocamera</strong></td><td><strong>Modello del filtro</strong></td><td><strong>Tipo di immagine</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

La linea LATTICE è un sistema modulare di telecamere multispettrali basato sul sensore Sony IMX265 con otturatore globale (3,1 MP, pixel da 3,45 µm). Ogni telecamera memorizza la propria identità sotto forma di stringa di modello:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros la visualizza con il prefisso `LATT-` (ad esempio `LATT-M3M-L41-F550`), e la stringa del modello determina tutto a valle: il profilo del sensore, la disposizione delle bande e la calibrazione vengono risolti automaticamente; non c’è nulla da configurare per ogni singola telecamera. Il numero dell’obiettivo corrisponde al **campo visivo orizzontale in gradi**: `L41` = stretto 41°, `L87` = ampio 87°.

Esistono due configurazioni del sensore:

| Configurazione | Sensore      | Tipo di filtro                           | Bande per telecamera                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Colore Bayer | Triplo filtro passa-banda                       | 3 bande spettrali da una singola esposizione                                 |
| **M3M**       | Monocromatico  | Singolo filtro interferenziale a banda stretta | 1 banda calibrata — combinare più telecamere M3M per gli indici di vegetazione |

### Opzioni filtro M3C (Bayer)

| Filtro | Bande (nome @ centro nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 618,4/46 · Cyan 494,1/40 · NIR 821,6/54 (misurato — vedi sotto) |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

Per `FOCN` i valori corrispondono alla banda a metà massimo **misurata** del filtro (Chloros 1.3.0 e versioni successive; le versioni precedenti utilizzavano i valori riportati nella scheda tecnica 615/21 · 490/38 · 808/14). I nomi delle bande rimangono `Orange_615`, `Cyan_490` e `NIR_808`, mentre il centro e la larghezza misurati sono quelli che Chloros inserisce in ogni esportazione calibrata.

### Catalogo dei filtri M3M (mono) — 23 SKU

Il numero F è l’etichetta SKU; la banda misurata (stampata su ogni prodotto calibrato destinato all’esportazione) è la scansione del filtro per lotto:

| SKU    | Centro (nm, misurato) | Bordi FWHM (nm) | Larghezza (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominale)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominale)           | 867–893         | 26         |
| F905   | — (nominale)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_"I bordi delle bande sono misurati come valori di larghezza a metà altezza (FWHM) ricavati dalle scansioni dei filtri per lotto di MAPIR — gli stessi valori che Chloros inserisce in ogni esportazione calibrata."_ "— (nominale)" = nessuna scansione del lotto ancora disponibile; per tali SKU, il centro indicato è il numero SKU e la larghezza è il dato fornito dal produttore.

† "La riflettanza di F988 viene calibrata utilizzando un pannello di riflettanza in scena: la banda si trova oltre l’intervallo calibrato del sensore di luce DAQ, quindi Chloros applica la vostra acquisizione più recente del pannello e la mantiene tra un rilevamento e l’altro." Vedi [Obiettivi di calibrazione](calibration-targets.md).

Per il controllo in tempo reale delle telecamere, gli array, la configurazione di rete e la catena di elaborazione radiometrica, consulta la [guida LATTICE](lattice/).
