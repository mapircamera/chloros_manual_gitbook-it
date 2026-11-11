---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Domande frequenti

<details>

<summary>È possibile elaborare con Chloros immagini provenienti da fotocamere che non sono del marchio MAPIR?</summary>

No, Chloros supporta solo l’elaborazione delle immagini delle telecamere MAPIR, ovvero le famiglie Survey3 e LATTICE. Per ulteriori informazioni, consulta l’elenco dei [modelli di telecamere supportati](supported-cameras.md). Offriamo la possibilità di elaborare immagini provenienti da altre telecamere su MAPIR Cloud; consulta l’elenco completo [qui](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Chloros supporta le fotocamere LATTICE?</summary>

Sì. Chloros 1.3.0 supporta i moduli fotografici LATTICE M3C e M3M end-to-end: **controllo in tempo reale**— individuazione, connessione, anteprima e acquisizione dalla scheda «Telecamere» dell’interfaccia grafica (`chloros-cli lattice`) o dall’SDK SDK, inclusi array multicamera sincronizzati con sincronizzazione temporale PTP — e**elaborazione radiometrica completa** delle acquisizioni (raw → debayering → radianza → riflettanza → indice). Vedi [Telecamere supportate](supported-cameras.md) e la [guida LATTICE](lattice/README.md).

</details>

<details>

<summary>Posso calibrare le mie immagini per la riflettanza senza un bersaglio di calibrazione?</summary>

**Survey3:** No. Senza un’immagine del bersaglio di calibrazione acquisita contemporaneamente alle immagini non relative al bersaglio, non sarà possibile correlare i valori dei pixel dell’immagine a una percentuale di riflettanza nota. Se inoltre non si include il registro di un sensore di luce MAPIR, lo spettro della luce ambientale non verrà misurato e i risultati di riflettanza non saranno accurati.

**LATTICE:** Sì. La riflettanza può essere riferita all’irraggianza discendente misurata da un sensore di luce DAQ anziché da un pannello (ρ = π·L/E). Quando *è* presente un bersaglio all’interno dell’inquadratura che ha superato il controllo di qualità (QA), esso diventa il riferimento assoluto per impostazione predefinita (`--reflectance-source auto`), a meno che non superi il controllo con un flag, nel qual caso il bersaglio e le catene DAQ competono sul punteggio misurato. Un’eccezione: «La riflettanza F988 viene calibrata utilizzando un pannello di riflettanza presente nella scena: la banda si trova al di fuori dell’intervallo calibrato del sensore di luce DAQ, quindi Chloros a applica l’ultima acquisizione del pannello e la mantiene tra un rilevamento e l’altro». Vedi [Bersagli di calibrazione](calibration-targets.md).

</details>

<details>

<summary>È necessario un sensore di luce DAQ?</summary>

Non per la radianza: le esportazioni di radianza LATTICE derivano dalla calibrazione radiometrica di fabbrica di ciascuna telecamera e non richiedono né un sensore DAQ né un target. Per la **riflettanza**è necessario un riferimento per la luce ambientale — ovvero una misurazione della luce in discesa da un sensore di luce DAQ o un target di calibrazione all’interno dell’inquadratura. Un sensore DAQ consente di ottenere valori di riflettanza calibrati**senza posizionare alcun pannello nella scena**. I file `.daq` registrati vengono abbinati automaticamente alle immagini in base al timestamp. Vedi [Target di calibrazione](calibration-targets.md) e la [Guida di riferimento CLI](reference/cli-reference.md).

</details>

<details>

<summary>Posso utilizzare Chloros con un assistente AI (Claude, ChatGPT, ecc.)?</summary>

Sì — questo manuale e la CLI/SDK sono stati progettati proprio per questo:

* L’indice completo del manuale è disponibile all’indirizzo `https://mapir.gitbook.io/chloros/llms.txt`, in modo che gli assistenti AI possano individuare ogni pagina.
* Il codice Markdown grezzo di ogni pagina è disponibile all’URL della pagina in minuscolo con l’aggiunta di `.md` (ad esempio `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* La [Guida di riferimento CLI](reference/cli-reference.md) e la [Guida di riferimento SDK](reference/sdk-reference.md) sono redatte per l’utilizzo da parte dei modelli di linguaggio (LLM): flag esatti, valori predefiniti, semantica di uscita e comandi copiabili e incollabili.

Consulta [Assistenti AI](ai-assistants.md) per sapere come configurare il tuo assistente affinché utilizzi Chloros.

</details>

<details>

<summary>Dove vengono salvati i miei file di output elaborati?</summary>

I prodotti vengono salvati nella cartella del progetto, raggruppati per fotocamera e poi per formato di file:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **cartella-fotocamera** — `LATT-<sensor>-<lens>-F<filter>` per LATTICE, `<model>_<filter>` (ad es. `Survey3N_RGN`) per Survey3
* **cartella-formato** — `tiff16`, `tiff8`, `png8`, `jpg8` o `tiff32`
* **cartelle dei prodotti** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (sempre sotto `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — Solo multispettrale LATTICE: maschere per pixel accanto alle cartelle dei prodotti, vedi sotto**I file esportati mantengono il nome del file di origine — è la cartella a identificare il prodotto, non un suffisso del nome del file.**Con l’CLI, la cartella del progetto viene creata all’interno della cartella predefinita dei progetti (`Documents/MAPIR/Chloros Projects`, a meno che non sia stata modificata), a meno che non si passi `-o`; un percorso relativo `-o` viene interpretato come relativo alla cartella da cui si esegue il comando. Si noti che un'esecuzione di `chloros-cli process` che ha richiesto prodotti ma non ne ha scritto nessuno stampa `Processing finished but wrote no image products.` e**termina con un valore diverso da zero**, in modo che gli script possano rilevarla. Vedere [Formati delle immagini di output](output-image-formats.md) e la [Guida di riferimento CLI](reference/cli-reference.md).

</details>

<details>

<summary>Cosa sono le cartelle clamp_masks e i file *_clampmask.tif?</summary>

Novità della versione 1.3.0. I prodotti di radianza e riflettanza di una telecamera multispettrale LATTICE M3C eseguono una separazione delle bande 3×3 e, laddove viene generato un valore negativo in una banda, Chlorose limita tale valore a zero. Un prodotto con almeno un pixel troncato riceve una maschera `uint8` a banda singola che indica quali pixel e bande sono stati troncati (bit 0 = canale R del prodotto, bit 1 = G, bit 2 = B). Le maschere vengono salvate in una cartella `clamp_masks/` **accanto** alle cartelle dei prodotti — ad esempio `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — e il tag XMP del prodotto `Chloros:UnmixClampMask` indica il nome della relativa maschera (oppure riporta `none`).

Non si tratta di immagini della scena: l’importazione di Chloros le ignora e, poiché non si trovano mai all’interno di una cartella del prodotto, è possibile trasferire una cartella del prodotto a Pix4D o Metashape così com’è. Vedi [Maschere di aggancio](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Perché Chloros segnala "La cartella del progetto è troppo profonda per questo computer"?</summary>

Questo vale solo per Windows. Windows viene fornito con i percorsi lunghi disattivati, quindi nessun percorso di file può superare i 260 caratteri. I prodotti Chloros si trovano fino a circa 130 caratteri al di sotto della cartella del progetto, quindi Chloros rifiuta una cartella del progetto più profonda prima dell’elaborazione — anziché avviarsi e non scrivere nulla, come poteva accadere prima della versione 1.3.0. Il messaggio indica la cartella del progetto più lunga che risulta valida.

Risolvi il problema in uno dei seguenti modi:

* Utilizza una cartella di progetto più breve (la **Directory di lavoro** nelle Impostazioni del progetto, oppure `-o` / `chloros-cli set-project-folder` per la CLI).
* Attiva i percorsi lunghi di Windows e riavvia Chloros. L’opzione **Abilita percorsi lunghi di Windows** del programma di installazione di Chloros (selezionata per impostazione predefinita) esegue questa operazione; riesegui il programma di installazione se l’hai deselezionata.

Vedi [Avvio dell’elaborazione](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Posso modificare le mie immagini prima dell’elaborazione in Chloros?</summary>

No. Chloros presuppone che i dati di input non siano stati modificati. Non modificare i nomi dei file.

</details>

<details>

<summary>Posso impostare le mie fotocamere MAPIR Survey3 sull'esposizione automatica ed elaborare le immagini in Chloros?</summary>

No. I set di dati delle immagini Survey3 devono avere un’esposizione fissa/bloccata, quindi non è consentita né la velocità dell’otturatore automatica né l’ISO automatico. Tutte le immagini dello stesso modello di fotocamera devono avere velocità dell’otturatore e ISO (esposizione) identici.

Le fotocamere LATTICE non presentano questa limitazione: Chloros ne gestisce l’esposizione in tempo reale (Smart AE) e ogni acquisizione registra l’esposizione e il guadagno effettivamente utilizzati, di cui tiene conto la pipeline radiometrica.

</details>

<details>

<summary>Chloros può elaborare o analizzare immagini ortomosaiche?</summary>

No. Sono supportate solo le singole immagini delle fotocamere MAPIR, non le immagini unite come una mappa ortomosaica.

</details>

<details>

<summary>Come posso velocizzare la fase di rilevamento dei target in Chloros?</summary>

Nella tabella del browser dei file, preselezionando le immagini di riferimento nella colonna di destra si indicherà a Chloros di cercare i bersagli di calibrazione solo in quelle immagini, velocizzando notevolmente l’elaborazione.

</details>

<details>

<summary>Se intendo caricare le mie immagini su <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> devo elaborarle in Chloros prima del caricamento?</summary>

Se prevedi di caricare le immagini sulla nostra piattaforma di elaborazione online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), non modificarle prima del caricamento. Il Cloud eseguirà tutte le stesse elaborazioni e molto altro ancora.

</details>

<details>

<summary>MAPIR supporterà mai la funzione X? Mi piacerebbe davvero che MAPIR offrisse la funzione X.</summary>

Siamo sempre interessati a ricevere feedback sui nostri prodotti. Se riscontri un problema con i nostri prodotti o hai un suggerimento su come possiamo migliorarli, ti preghiamo di [CONTATTARCI](https://www.mapir.camera/community/contact) per condividere le tue opinioni. Gran parte della nostra attività di ricerca e sviluppo è guidata dall’ascolto delle esigenze principali dei nostri clienti.

</details>

<details>

<summary>Chloros è disponibile per Linux?</summary>

Sì! Chloros 1.3.0 supporta Linux amd64 (x86_64) e arm64 (NVIDIA Jetson JetPack 6) tramite i pacchetti `.deb`. La CLI e l’SDK Linux sono pienamente supportate su Linux, compreso il controllo in tempo reale delle telecamere LATTICE e dei sensori DAQ. Non è disponibile una GUI per Linux: tutte le interazioni avvengono tramite la [CLI](CLI.md) o l’[SDK SDK](api-python-sdk.md). Per ulteriori dettagli, consultare la [Panoramica su Linux](linux/linux-overview.md).

</details>

<details>

<summary>È possibile eseguire Chloros su NVIDIA Jetson?</summary>

Sì! Chloros supporta la **famiglia Orin** di NVIDIA Jetson — Orin Nano, Orin NX e AGX Orin — con JetPack 6. I modelli precedenti (Jetson Nano, TX2, Xavier NX) non possono eseguire JetPack 6 e non sono supportati dal pacchetto attuale. Chloros rileva automaticamente il modello di Jetson in uso e ne ottimizza la strategia di elaborazione. Il pacchetto Jetson utilizza la build CUDA di PyTorch di JetPack 6, quindi l’elaborazione avviene sulla GPU del Jetson. Consulta la [Guida NVIDIA Jetson](linux/nvidia-jetson-guide.md) per le istruzioni di configurazione e implementazione.

</details>

<details>

<summary>Chloros ottimizza automaticamente il funzionamento in base al mio hardware?</summary>

Sì! Chloros include il [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) che rileva automaticamente CPU, GPU, RAM e (su Jetson) i sensori termici. Seleziona quindi la strategia di elaborazione ottimale — da `GPU_PARALLEL` sulle GPU con molta memoria a `GPU_SINGLE` su quelle più piccole fino a `CPU_PARALLEL` sui sistemi privi di una GPU CUDA utilizzabile da PyTorch. Non è necessaria alcuna configurazione manuale.

</details>

<details>

<summary>Che cos’è la pipeline di elaborazione a 4 thread?</summary>

Chloros utilizza un’architettura a pipeline a 4 thread per gli utenti di Chloros+: Il thread 1 (Rilevamento) carica le immagini e rileva i target di calibrazione, il thread 2 (Calibrazione) calcola la calibrazione della riflettanza, il thread 3 (Elaborazione) esegue il debayering accelerato dalla GPU e il calcolo dell’indice, mentre il thread 4 (Esportazione) scrive i file di output. È possibile elaborare più immagini contemporaneamente in thread diversi per ottenere la massima produttività. Per ulteriori dettagli, consultare [Pipeline di elaborazione](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Come posso eseguire la diagnostica sulla mia installazione di Chloros?</summary>

Utilizzare il comando `selftest` per eseguire uno "smoke test" in 7 fasi: versione, disponibilità delle porte, avvio del backend, connettività API (`/api/test`), informazioni di sistema (`/api/system-info` — GPU/CUDA/PyTorch), presenza del modello di denoising e prontezza di CUDA e del denoising:

```bash
chloros-cli selftest
```

Ciò è particolarmente utile sui sistemi Linux per verificare la configurazione della GPU e di CUDA. Su un Jetson, `CUDA: False` di solito indica che mancano le librerie CUDA e cuDNN di JetPack 6: installarle con `sudo apt install nvidia-jetpack`.

</details>
