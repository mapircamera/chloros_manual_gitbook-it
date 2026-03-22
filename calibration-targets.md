---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Target di calibrazione

MAPIR offre diversi target di calibrazione per soddisfare una vasta gamma di applicazioni. Il modello compatto T4-R50, visibile qui sotto, contiene 4 pannelli la cui riflettanza luminosa è stata misurata nell&#x27;intervallo da 250 a 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>I target di riferimento diffusi T4 presentano le seguenti curve di riflettanza, [scarica i dati qui](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Riflettanza T4 :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Riflettanza T4 :: 400-1000 nm</p></figcaption></figure>I bersagli di riferimento diffusi T4P presentano le seguenti curve di riflettanza, [scarica i dati qui](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR Riflettanza T4P :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Riflettanza T4P :: 400-1000 nm</p></figcaption></figure>Osservando il grafico di riflettanza è possibile notare che i valori rappresentano la lunghezza d&#x27;onda (asse x) rispetto alla percentuale di riflettanza (asse y). Quando acquisiamo un&#x27;immagine del bersaglio di calibrazione, creiamo quindi una relazione tra il valore del pixel e la percentuale di riflettanza, all&#x27;interno dello spettro a cui ciascuna delle bande del sensore della fotocamera è sensibile.

Ciò significa che con ogni immagine acquisita con le nostre fotocamere, è possibile utilizzare una foto dei nostri target di riflettanza, come il [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) o il [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) per calibrare le immagini in base alla riflettanza. Una volta calibrata, ogni pixel dell&#x27;immagine corrisponde a una percentuale di riflettanza.

Se si esportano le immagini calibrate in Chloros come un tipico JPG o in TIFF, la percentuale di riflettanza viene calcolata dividendo il valore del pixel per la profondità di bit del formato dell&#x27;immagine. Quindi, per il JPG si divide per 255, mentre per TIFF si divide per 65.535. È anche possibile scegliere l&#x27;output in formato PERCENT in Chloros; in questo caso, ogni pixel avrà un valore percentuale compreso tra 0,0 e 1,0 (riflettanza da 0% a 100%). Si tenga presente, tuttavia, che alcune applicazioni di elaborazione delle immagini non supportano le immagini in formato percentuale (a virgola mobile) e che queste occupano molto spazio di archiviazione.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
