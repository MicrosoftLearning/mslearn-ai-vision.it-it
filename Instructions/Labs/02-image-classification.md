---
lab:
  title: Classificare le immagini con un modello personalizzato di Visione di Azure AI
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Classificare le immagini con un modello personalizzato di Visione di Azure AI

Visione di Azure AI consente di eseguire il training di modelli personalizzati per classificare e rilevare oggetti con etichette specificate. In questo lab verrà creato un modello di classificazione delle immagini personalizzato per classificare le immagini di frutta.

## Effettuare il provisioning di una risorsa di Visione artificiale

Se non è già disponibile nella sottoscrizione, sarà necessario effettuare il provisioning di una risorsa di **Visione artificiale**.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Selezionare **Crea una risorsa**.
1. Nella barra di ricerca cercare *Visione artificiale*, selezionare **Visione artificiale** e creare la risorsa con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, si potrebbe non essere autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *scegliere tra Stati Uniti orientali, Stati Uniti occidentali, Francia centrale, Corea centrale, Europa settentrionale, Asia sud-orientale, Europa occidentale o Asia orientale\**
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: gratuito F0.

    \*I set di funzionalità completi di Visione di Azure AI 4.0 attualmente sono disponibili solamente in queste aree.

1. Selezionare le caselle di controllo necessarie e creare la risorsa.
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

È anche necessario un account di archiviazione per archiviare le immagini di training.

1. Nel portale di Azure cercare e selezionare **Account di archiviazione** e creare un nuovo account di archiviazione con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *Scegliere lo stesso gruppo di risorse creato nella risorsa di Visione personalizzata in*
    - **Nome account di archiviazione**: customclassifySUFFIX 
        - *Nota: sostituire il token `SUFFIX` con le proprie iniziali o un altro valore per assicurarsi che il nome della risorsa sia univoco a livello globale.*
    - **Area**: *Scegliere la stessa area usata per la risorsa del servizio di intelligenza artificiale di Azure*
    - **Servizio primario**: Archiviazione BLOB di Azure o Azure Data Lake Storage Gen 2
    - **Carico di lavoro primario**: altro
    - **Prestazioni**: standard
    - **Ridondanza**: Archiviazione con ridondanza locale

1. Quando la risorsa è stata distribuita, selezionare **Vai alla risorsa**.
1. Abilitare l'accesso pubblico nell'account di archiviazione. Nel riquadro sinistro passare a **Configurazione** nel gruppo **Impostazioni** e abilitare *Consenti l'accesso anonimo al BLOB*. Seleziona **Salva**
1. Nel riquadro a sinistra, in **Archiviazione dati**, selezionare **Contenitori** e creare un nuovo contenitore denominato `fruit` e impostare **Livello di accesso anonimo** su *Contenitore (accesso in lettura anonimo per contenitori e BLOB)*.

    > **Nota**: Se il **livello di accesso anonimo** è disabilitato, aggiornare la pagina del browser.
   
## Clonare il repository per questo corso

I file di immagine per il training del modello sono stati forniti in un repository GitHub. Si clonerà il repository e si caricheranno le immagini nell'account di archiviazione usando Cloud Shell dal portale di Azure. 

> **Suggerimento**: Se il repository **mslearn-ai-vision** è già stato clonato di recente, è possibile ignorare l'attività di clonazione. In caso contrario, seguire questa procedura per clonare il repository nell'ambiente di sviluppo.

1. Nel portale di Azure, usare il pulsante **[\>_]** a destra della barra di ricerca nella parte superiore della pagina per creare una nuova Cloud Shell nel portale di Azure, selezionando un ambiente ***PowerShell***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Nel riquadro PowerShell immettere i comandi seguenti per clonare il repository GitHub per questo esercizio:

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. Dopo aver clonato il repository, passare alla cartella contenente i file dell'esercizio:  

    ```
   cd mslearn-ai-vision/Labfiles/02-image-classification
    ```

1. Eseguire il comando `code replace.ps1` ed esaminare il codice. Si noterà che sostituisce il nome dell'account di archiviazione per il segnaposto in un file JSON (il file COCO) usato in un passaggio successivo.
1. Sostituire il segnaposto *solo nella prima riga* del file con il nome dell'account di archiviazione.
1. Dopo aver sostituito i segnaposto, nell'editor di codice usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.
1. Eseguire lo script con il comando seguente:

    ```powershell
    ./replace.ps1
    ```

1. È possibile esaminare il file COCO per assicurarsi che il nome dell'account di archiviazione sia presente. Eseguire `code training-images/training_labels.json` e visualizzare le prime voci. Nel campo *absolute_url* dovrebbe essere visualizzato qualcosa di simile a *"https://myStorage.blob.core.windows.net/fruit/...*. Se non viene visualizzata la modifica prevista, assicurarsi di aggiornare solo il primo segnaposto nello script di PowerShell.
1. Chiudere l'editor del codice.
1. Eseguire il comando seguente, sostituendo `<your-storage-account>` con il nome dell'account di archiviazione, per caricare il contenuto della cartella **training-images** nel contenitore `fruit` creato in precedenza.

    ```powershell
    az storage blob upload-batch --account-name <your-storage-account> -d fruit -s ./training-images/
    ```

1. Aprire il contenitore `fruit` e verificare che i file siano stati caricati correttamente.

## Creare un progetto di training del modello personalizzato

Si creerà quindi un nuovo progetto di training per la classificazione di immagini personalizzate in Vision Studio.

1. Nel Web browser passare a `https://portal.vision.cognitive.azure.com/` e accedere con l'account Microsoft in cui è stata creata la risorsa Azure per intelligenza artificiale.
1. Selezionare il riquadro **Personalizza modelli con immagini** (è disponibile nella scheda **Analisi di immagini**, se non viene mostrato nella visualizzazione predefinita).
1. Selezionare l'account Servizi di Azure AI creato.
1. Nel progetto selezionare **Aggiungi nuovo set di dati** nella parte superiore. Configurare  con le seguenti impostazioni:
    - **Nome del set di dati**: training_images
    - **Tipo di modello**: Classificazione immagini
    - **Selezionare un contenitore di archiviazione BLOB di Azure**: Selezionare **Seleziona contenitore**
        - **Sottoscrizione**: *la sottoscrizione di Azure usata*
        - **Account di archiviazione**: *Account di archiviazione creato*
        - **Contenitore BLOB**: frutta
    - Selezionare la casella "Consenti a Vision Studio di leggere e scrivere nell'archivio BLOB"
1. Selezionare il set di dati **training_images**.

A questo punto della creazione del progetto, in genere si seleziona **Crea progetto di etichettatura dati di Azure ML** e si etichettano le immagini, generando un file COCO. Si consiglia di provarci se si ha tempo, ma ai fini di questo lab abbiamo già etichettato le immagini e fornito il file COCO risultante.

1. Selezionare **Aggiungi file COCO**
1. Nell'elenco a discesa selezionare **Importa file COCO da un contenitore BLOB**
1. Poiché è già stato connesso il contenitore denominato `fruit`, Vision Studio cerca un file COCO. Selezionare **training_labels.json** nell'elenco a discesa e aggiungere il file COCO.
1. Passare a **Modelli personalizzati** a sinistra e selezionare **Eseguire il training di un nuovo modello**. Usare le seguenti impostazioni:
    - **Nome del modello**: classifyfruit
    - **Tipo di modello**: Classificazione immagini
    - **Scegliere il set di dati di training**: training_images
    - Lasciare il resto come predefinito e selezionare **Esegui training modello**

Il training può richiedere del tempo: il budget predefinito è fino a un'ora, ma per questo piccolo set di dati è di solito molto più rapido. Selezionare il pulsante **Aggiorna** ogni paio di minuti fino a quando lo stato del processo è *Completato*. Seleziona il modello.

Qui è possibile visualizzare le prestazioni del processo di training. Esaminare la precisione e l'accuratezza del modello sottoposto a training.

## Testare il modello personalizzato

Il training del modello è stato eseguito ed è pronto per il test.

1. Nella parte superiore della pagina per il modello personalizzato selezionare **Prova**.
1. Selezionare il modello **classifyfruit** dall'elenco a discesa specificando il modello da usare e passare alla cartella **02-image-classification\test-images**.
1. Selezionare ogni immagine e visualizzare i risultati. Selezionare la scheda **JSON** nella casella dei risultati per esaminare la risposta JSON completa.

<!-- Option coding example to run-->
## Pulire le risorse

Se non si usano le risorse di Azure create in questo lab per altri moduli di training, è possibile eliminarle per evitare addebiti aggiuntivi.

1. Aprire il portale di Azure in `https://portal.azure.com` e nella barra di ricerca superiore cercare le risorse create in questo lab.

2. Nella pagina della risorsa selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa. In alternativa, è possibile eliminare l'intero gruppo di risorse per pulire tutte le risorse contemporaneamente.
