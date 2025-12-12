---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Target di calibrazione

MAPIR offre vari target di calibrazione per coprire una vasta gamma di applicazioni. Il compatto T4-R50 mostrato di seguito contiene 4 pannelli che sono stati misurati per la riflettanza della luce da 250 a 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

I target di riferimento diffusi T4 hanno le seguenti curve di riflettanza, [scarica i dati qui](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Riflettanza :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Riflettanza :: 400-1000 nm</p></figcaption></figure>

Osservando il grafico della riflettanza è possibile notare che i valori sono la lunghezza d&#x27;onda (asse x) rispetto alla percentuale di riflettanza (asse y). Quando acquisiamo un&#x27;immagine del target di calibrazione, creiamo una relazione tra il valore dei pixel e la percentuale di riflettanza, all&#x27;interno dello spettro a cui sono sensibili ciascuna delle bande del sensore della fotocamera.

Ciò significa che con ogni immagine acquisita con le nostre fotocamere, è possibile utilizzare una foto dei nostri target di riflettanza, come il [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) o il [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) per calibrare le immagini in base alla riflettanza. Una volta calibrato, ogni pixel dell&#x27;immagine è uguale alla percentuale di riflettanza.

Se si esportano le immagini calibrate in Chloros come JPG o TIFF tipici, la percentuale di riflettanza viene calcolata dividendo il valore del pixel per la profondità di bit del formato dell&#x27;immagine. Quindi, per JPG dividere per 255 e per TIFF dividere per 65.535. È anche possibile scegliere il formato di output PERCENT in Chloros, e quindi ogni pixel avrà un valore percentuale compreso tra 0,0 e 1,0 (riflettanza dallo 0% al 100%). È importante tenere presente che alcune applicazioni di immagini non accettano immagini in percentuale (virgola mobile) e che queste immagini occupano molto spazio di archiviazione.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
