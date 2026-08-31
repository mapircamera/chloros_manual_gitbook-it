---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Domande frequenti

<details>

<summary>È possibile elaborare immagini provenienti da telecamere che non sono del marchio MAPIR con Chloros?</summary>

No, Chloros supporta solo l&#x27;elaborazione delle immagini delle telecamere MAPIR, ovvero le famiglie Survey3 e LATTICE. Per ulteriori informazioni, consulta l’elenco dei [modelli di telecamere supportati](supported-cameras.md). Offriamo la possibilità di elaborare le immagini di altre telecamere su MAPIR Cloud; consulta l’elenco completo [qui](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Chloros supporta le telecamere LATTICE?</summary>

Sì. Chloros 1.2.0 supporta i moduli telecamera LATTICE M3C e M3M end-to-end: **controllo in tempo reale**— individuazione, connessione, anteprima e acquisizione dalla scheda “Telecamere” dell’interfaccia grafica, `chloros-cli lattice` o Python SDK, compresi gli array multicamera sincronizzati con sincronizzazione temporale PTP — e**elaborazione radiometrica completa** delle acquisizioni (raw → debayering → radianza → riflettanza → indice). Vedi [Telecamere supportate](supported-cameras.md) e la [guida LATTICE](lattice/README.md).

</details>

<details>

<summary>Posso calibrare le mie immagini per la riflettanza senza un bersaglio di calibrazione?</summary>

**Survey3:** No. Senza un’immagine del bersaglio di calibrazione acquisita contestualmente alle immagini non relative al bersaglio, non sarà possibile correlare i valori dei pixel dell’immagine a una percentuale di riflettanza nota. Se inoltre non si include il registro di un sensore di luce MAPIR, lo spettro della luce ambientale non verrà misurato e i risultati di riflettanza non saranno accurati.**LATTICE:** Sì. La riflettanza può essere riferita all’irradianza discendente misurata da un sensore di luce DAQ anziché da un pannello (ρ = π·L/E). Quando è presente un bersaglio in-frame che ha superato il controllo di qualità, questo diventa per impostazione predefinita il riferimento assoluto (`--reflectance-source auto`). Un’eccezione: «La riflettanza F988 viene calibrata utilizzando un pannello di riflettanza presente nella scena: la banda si trova al di fuori dell’intervallo calibrato del sensore di luce DAQ, quindi Chloros applica l’ultima acquisizione del pannello ed è valida fino alla successiva rilevazione del pannello.» Vedi [Target di calibrazione](calibration-targets.md).

</details>

<details>

<summary>È necessario un sensore di luce DAQ?</summary>

Non per la radianza: le esportazioni di radianza di LATTICE derivano dalla calibrazione radiometrica di fabbrica di ciascuna telecamera e non richiedono né un sensore DAQ né un bersaglio. Per la **riflettanza**è necessario un riferimento per la luce ambientale — che sia una misurazione della luce discendente effettuata da un sensore di luce DAQ oppure un bersaglio di calibrazione all’interno dell’inquadratura. Un sensore DAQ consente di ottenere valori di riflettanza calibrati**senza posizionare alcun pannello nella scena**. I file `.daq` registrati vengono abbinati automaticamente alle immagini in base al timestamp. Vedi [Target di calibrazione](calibration-targets.md) e la [Guida di riferimento](reference/cli-reference.md).

</details>

<details>

<summary>Posso utilizzare Chloros con un assistente AI (Claude, ChatGPT, ecc.)?</summary>

Sì — questo manuale e i file CLI/SDK sono stati pensati proprio per questo:

* L’indice completo del manuale è disponibile all’indirizzo `https://mapir.gitbook.io/chloros/llms.txt`, in modo che gli assistenti AI possano individuare ogni pagina.
* Il codice Markdown grezzo di ogni pagina è disponibile all’indirizzo corrispondente in minuscolo, seguito dall’estensione `.md` (ad esempio `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* La [Guida di riferimento di CLI](reference/cli-reference.md) e [Riferimento SDK](reference/sdk-reference.md) sono stati redatti per l’utilizzo da parte dei modelli di linguaggio di grandi dimensioni (LLM): flag esatti, impostazioni predefinite, semantica di uscita e comandi copiabili e incollabili.

Vedi [Assistenti AI](ai-assistants.md) per sapere come indirizzare il tuo assistente verso Chloros.

</details>

<details>

<summary>Dove vengono salvati i miei file di output elaborati?</summary>

I prodotti vengono salvati nella cartella del progetto, raggruppati per fotocamera e poi per formato di file:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **cartella-fotocamera** — `LATT-<sensor>-<lens>-F<filter>` per LATTICE, `<model>_<filter>` (ad es. `Survey3N_RGN`) per Survey3
* **cartella-formato** — `tiff16`, `tiff8`, `png8`, `jpg8` o `tiff32`
* **cartelle dei prodotti** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (sempre all’interno di `tiff32`), `<INDEX>_Index_Images/`**I file esportati mantengono il nome del file di origine — la cartella identifica il prodotto, non un suffisso del nome del file.**Con CLI, la cartella del progetto viene creata accanto alla cartella di input, a meno che non si passi `-o`. Si noti che un&#x27;esecuzione di `chloros-cli process` che ha richiesto dei prodotti ma non ne ha scritto nessuno stampa `Processing finished but wrote no image products.` e**termina con un valore diverso da zero**, in modo che gli script possano rilevarla. Vedere [Formati immagine di output](output-image-formats.md) e il [Riferimento CLI](reference/cli-reference.md).

</details>

<details>

<summary>Posso modificare le mie immagini prima dell’elaborazione in Chloros?</summary>

No. Chloros presuppone che i dati di input non siano stati modificati. Non modificare i nomi dei file.

</details>

<details>

<summary>Posso impostare le mie fotocamere MAPIR e Survey3 sull’esposizione automatica ed elaborare le immagini in Chloros?</summary>

No. I set di dati delle immagini Survey3 devono avere un&#x27;esposizione fissa/bloccata, quindi non è consentita né la velocità dell&#x27;otturatore automatica né l&#x27;ISO automatico. Tutte le immagini dello stesso modello di telecamera devono avere velocità dell&#x27;otturatore e ISO (esposizione) identici.

Le telecamere LATTICE non presentano questa restrizione: Chloros gestisce l’esposizione in tempo reale (Smart AE) e ogni acquisizione registra l’esposizione e il guadagno effettivamente utilizzati, di cui tiene conto la pipeline radiometrica.

</details>

<details>

<summary>Chloros è in grado di elaborare o analizzare immagini ortomosaiche?</summary>

No. Sono supportate solo le singole immagini delle fotocamere MAPIR, non immagini unite come una mappa ortomosaica.

</details>

<details>

<summary>Come posso velocizzare la fase di rilevamento dei target in Chloros?</summary>

Nella tabella del browser dei file, preselezionando le immagini di riferimento nella colonna di destra si indicherà a Chloros di cercare i target di calibrazione solo in quelle immagini, accelerando notevolmente l’elaborazione.

</details>

<details>

<summary>Se intendo caricare le mie immagini su <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> devo elaborarle in Chloros prima del caricamento?</summary>

Se prevedi di caricare le immagini sulla nostra piattaforma di elaborazione online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), non modificare le immagini prima del caricamento. Il Cloud eseguirà tutte le stesse elaborazioni e molto altro ancora.

</details>

<details>

<summary>MAPIR supporterà mai la funzionalità X? Mi piacerebbe davvero che MAPIR offrisse X.</summary>

Siamo sempre interessati a ricevere feedback sui nostri prodotti. Se riscontrate un problema con i nostri prodotti o avete un suggerimento su come possiamo migliorarli, vi preghiamo di [CONTATTARCI](https://www.mapir.camera/community/contact) per condividere le vostre opinioni. Gran parte della nostra attività di ricerca e sviluppo è guidata dall’ascolto delle esigenze principali dei nostri clienti.

</details>

<details>

<summary>Chloros è disponibile per Linux?</summary>

Sì! Chloros 1.2.0 supporta Linux amd64 (x86_64) e arm64 (NVIDIA Jetson JetPack 6) tramite i pacchetti `.deb`. CLI e Python SDK sono pienamente supportati su Linux, compreso il controllo in tempo reale delle telecamere LATTICE e dei sensori DAQ. Non è disponibile alcuna interfaccia grafica (GUI) per Linux : tutte le interazioni avvengono tramite [CLI](CLI.md) o [Python SDK](api-python-sdk.md). Per ulteriori dettagli, consultare la [Panoramica di Linux](linux/linux-overview.md).

</details>

<details>

<summary>Posso eseguire Chloros su NVIDIA Jetson?</summary>

Sì! Chloros supporta le piattaforme NVIDIA Jetson, tra cui Jetson Nano, Orin Nano, Orin NX e AGX Orin con JetPack 6. Chloros rileva automaticamente il modello di Jetson in uso e ne ottimizza la strategia di elaborazione. Consulta la [Guida NVIDIA Jetson](linux/nvidia-jetson-guide.md) per le istruzioni di configurazione e implementazione.

</details>

<details>

<summary>Chloros si ottimizza automaticamente per il mio hardware?</summary>

Sì! Chloros include il [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) che rileva automaticamente la CPU, la GPU, la RAM e (su Jetson) i sensori termici. Seleziona quindi la strategia di elaborazione ottimale: da `GPU_PARALLEL` su sistemi con molta memoria a `GPU_SINGLE` su dispositivi con risorse limitate fino a `CPU_PARALLEL` su sistemi senza GPU NVIDIA. Non è necessaria alcuna configurazione manuale.

</details>

<details>

<summary>Che cos’è la pipeline di elaborazione a 4 thread?</summary>

Chloros utilizza un’architettura a pipeline a 4 thread per gli utenti di Chloros+: Il thread 1 (Rilevamento) carica le immagini e rileva i target di calibrazione, il thread 2 (Calibrazione) calcola la calibrazione della riflettanza, il thread 3 (Elaborazione) esegue il debayering accelerato dalla GPU e il calcolo dell’indice, mentre il thread 4 (Esportazione) scrive i file di output. È possibile elaborare più immagini contemporaneamente in thread diversi per ottenere la massima produttività. Per ulteriori dettagli, consultare [Pipeline di elaborazione](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Come posso eseguire la diagnostica sulla mia installazione di Chloros?</summary>

Utilizzare il comando `selftest` per eseguire uno &quot;smoke test&quot; in 7 fasi: versione, disponibilità delle porte, avvio del backend, connettività API (`/api/test`), informazioni di sistema (`/api/system-info` — GPU/CUDA/PyTorch), presenza del modello di denoising e prontezza di CUDA + denoising:

```bash
chloros-cli selftest
```

Ciò è particolarmente utile sui sistemi Linux/Jetson per verificare la configurazione della GPU e di CUDA.

</details>
