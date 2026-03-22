---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Domande frequenti

<details>

<summary>È possibile elaborare immagini provenienti da fotocamere che non sono del marchio MAPIR con Chloros?</summary>

No, Chloros supporta solo l&#x27;elaborazione delle immagini delle fotocamere MAPIR. Per ulteriori informazioni, consulta l&#x27;elenco dei [modelli di fotocamera supportati](supported-cameras.md). Offriamo l&#x27;elaborazione di immagini da altre fotocamere su MAPIR Cloud; consulta l&#x27;elenco completo [qui](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Posso calibrare le mie immagini per la riflettanza senza un target di calibrazione?</summary>

No. Senza un&#x27;immagine del target di calibrazione acquisita in concomitanza con le immagini non target, non sarà possibile correlare i valori dei pixel dell&#x27;immagine a una percentuale di riflettanza nota. Se inoltre non si include il log di un sensore di luce MAPIR, lo spettro della luce ambientale non verrà misurato e i risultati di riflettanza non saranno accurati.

</details>

<details>

<summary>Posso modificare le mie immagini prima dell&#x27;elaborazione in Chloros?</summary>

No. Chloros presuppone che i dati di input non siano stati modificati. Non modificare i nomi dei file.

</details>

<details>

<summary>Posso impostare le mie fotocamere MAPIR e Survey3 sull&#x27;esposizione automatica ed elaborare le immagini in Chloros?</summary>

No. I set di dati delle immagini devono avere un&#x27;esposizione fissa/bloccata, quindi non è possibile utilizzare la velocità dell&#x27;otturatore automatica o l&#x27;ISO automatico. Tutte le immagini dello stesso modello di telecamera devono avere velocità dell&#x27;otturatore e ISO (esposizione) identici.

</details>

<details>

<summary>Chloros può elaborare o analizzare immagini ortomosaiche?</summary>

No. Sono supportate solo le singole immagini della fotocamera MAPIR, non immagini unite come una mappa ortomosaica.

</details>

<details>

<summary>Come posso velocizzare la fase di rilevamento dei target di Chloros?</summary>

Nella tabella del browser dei file, preselezionando le immagini di riferimento nella colonna di destra, si indicherà a Chloros di cercare i target di calibrazione solo in quelle immagini, velocizzando notevolmente l&#x27;elaborazione.

</details>

<details>

<summary>Se intendo caricare le mie immagini su <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> devo elaborarle in Chloros prima del caricamento?</summary>

Se hai intenzione di caricare le immagini sulla nostra piattaforma di elaborazione online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), non modificarle prima del caricamento. Cloud eseguirà tutte le stesse elaborazioni e molto altro ancora.

</details>

<details>

<summary>MAPIR supporterà mai la funzione X? Mi piacerebbe davvero che MAPIR offrisse X.</summary>

Siamo sempre interessati a ricevere feedback sui nostri prodotti. Se riscontri un problema con i nostri prodotti o hai un suggerimento su come possiamo migliorarli, ti preghiamo di [CONTATTARCI](https://www.mapir.camera/community/contact) per condividere le tue opinioni. La maggior parte della nostra attività di ricerca e sviluppo è guidata dall&#x27;ascolto delle esigenze principali dei nostri clienti.

</details>

<details>

<summary>Chloros è disponibile per Linux?</summary>

Sì! Chloros 1.1.0 supporta Linux amd64 (x86_64) e arm64 (NVIDIA Jetson JetPack 6) tramite i pacchetti `.deb`. CLI e Python SDK sono pienamente supportati su Linux. Non esiste una GUI per Linux — tutte le interazioni avvengono tramite [CLI](CLI.md) o [Python SDK](api-python-sdk.md). Per i dettagli, consultare la [Panoramica di Linux](linux/linux-overview.md).

</details>

<details>

<summary>Posso eseguire Chloros su NVIDIA Jetson?</summary>

Sì! Chloros 1.1.0 supporta le piattaforme NVIDIA Jetson, tra cui Jetson Nano, Orin Nano, Orin NX e AGX Orin con JetPack 6. Chloros rileva automaticamente il modello di Jetson in uso e ne ottimizza la strategia di elaborazione. Consulta la [Guida NVIDIA Jetson](linux/nvidia-jetson-guide.md) per le istruzioni di configurazione e distribuzione.

</details>

<details>

<summary>Chloros si ottimizza automaticamente per il mio hardware?</summary>

Sì! Chloros 1.1.0 include la [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) che rileva automaticamente CPU, GPU, RAM e (su Jetson) i sensori termici. Seleziona quindi la strategia di elaborazione ottimale: da `GPU_PARALLEL` su sistemi con molta memoria a `GPU_SINGLE` su dispositivi con risorse limitate fino a `CPU_PARALLEL` su sistemi senza GPU NVIDIA. Non è necessaria alcuna configurazione manuale.

</details>

<details>

<summary>Cos&#x27;è la pipeline di elaborazione a 4 thread?</summary>

Chloros 1.1.0 utilizza un&#x27;architettura a pipeline a 4 thread per gli utenti di Chloros+: Il thread 1 (Rilevamento) carica le immagini e rileva i target di calibrazione, il thread 2 (Calibrazione) calcola la calibrazione della riflettanza, il thread 3 (Elaborazione) esegue il debayering accelerato da GPU e il calcolo dell&#x27;indice, mentre il thread 4 (Esportazione) scrive i file di output. È possibile avere più immagini in thread diversi contemporaneamente per ottenere la massima produttività. Per i dettagli, consultare [Pipeline di elaborazione](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Come posso eseguire la diagnostica sulla mia installazione di Chloros?</summary>

Utilizzare il comando `selftest` per eseguire 7 diagnostiche di sistema, tra cui controllo della versione, disponibilità delle porte, avvio del backend, connettività API, informazioni di sistema, modelli di denoiser e disponibilità CUDA:

```bash
chloros-cli selftest
```

Ciò è particolarmente utile sui sistemi Linux/Jetson per verificare la configurazione della GPU e di CUDA.

</details>
