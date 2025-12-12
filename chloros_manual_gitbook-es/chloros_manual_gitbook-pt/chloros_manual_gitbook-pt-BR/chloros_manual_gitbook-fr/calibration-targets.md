---
description: Pannelli misurati in laboratorio utilizzati per calibrare i dati acquisiti in post-elaborazione
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Obiettivi di calibrazione

MAPIR offre vari target di calibrazione per coprire una gamma di applicazioni. Il compatto T4-R50 mostrato di seguito contiene 4 pannelli di cui è stata misurata la riflessione della luce da 250 a 2.500 nm.

__TAG_1<img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

Gli obiettivi di riferimento diffuso T4 hanno le seguenti curve di riflettanza, [download dei dati qui](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

__TAG_1<img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p> Riflettanza MAPIR T4: 250-2500 nm</p></figcaption></figure>

__TAG_1<img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p> Riflettanza MAPIR T4: 400-1000 nm</p></figcaption></figure>

Osservando il grafico della riflettanza puoi vedere che i valori sono la lunghezza d'onda (asse x) rispetto alla percentuale di riflettanza (asse y). Quando catturiamo un'immagine del target di calibrazione, creiamo quindi una relazione tra il valore dei pixel e la percentuale di riflettanza, all'interno dello spettro a cui è sensibile ciascuna banda del sensore della fotocamera.

Ciò significa che con ogni immagine acquisita con le nostre fotocamere, puoi utilizzare una foto dei nostri target di riflettanza, come [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) o [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) per calibrare le immagini per la riflettanza. Una volta calibrato, ogni pixel dell'immagine è pari alla riflettanza percentuale.

Se si generano le immagini calibrate in Chloros come il tipico JPG o TIFF, la percentuale di riflettanza viene calcolata dividendo il valore del pixel per la profondità di bit del formato dell'immagine. Quindi per JPG dividi per 255 e per TIFF dividi per 65.535. Puoi anche scegliere l'output del formato PERCENT in Chloros, quindi ciascun pixel varierà da un valore percentuale compreso tra 0,0 e 1,0 (riflettanza dallo 0% al 100%). Tieni presente che alcune applicazioni di immagini non possono accettare immagini in percentuale (virgola mobile) e hanno dimensioni di archiviazione di grandi dimensioni.

<div>TAG_4__<img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
