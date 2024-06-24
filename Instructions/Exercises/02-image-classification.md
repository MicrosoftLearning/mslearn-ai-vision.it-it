---
lab:
  title: Classificare le immagini con un modello personalizzato di Visione di Azure AI
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Classificare le immagini con un modello personalizzato di Visione di Azure AI

Visione di Azure AI consente di eseguire il training di modelli personalizzati per classificare e rilevare oggetti con etichette specificate. In questo lab verrà creato un modello di classificazione delle immagini personalizzato per classificare le immagini di frutta.

## Clonare il repository per questo corso

Se il repository di codice **Visione di Azure AI** non è già stato clonato nell'ambiente in cui si sta lavorando a questo lab, seguire questa procedura per clonarlo. In caso contrario, aprire la cartella clonata in Visual Studio Code.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-ai-vision` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**. Se viene presentato il messaggio *È stato rilevato un progetto di Funzioni di Azure nella cartella*, è possibile chiudere il messaggio in modo sicuro.

## Effettuare il provisioning delle risorse di Azure

Se non è già disponibile nella sottoscrizione, sarà necessario effettuare il provisioning di una risorsa di **Servizi di Azure AI**.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Nella barra di ricerca superiore cercare *Servizi di Azure AI*, selezionare **Servizi di Azure AI** e creare una risorsa account multiservizio di Servizi di Azure AI con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, si potrebbe non essere autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *Scegliere tra Stati Uniti orientali, Europa occidentale, Stati Uniti occidentali 2\**
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S0.

    \*I tag del modello personalizzato di Visione artificiale di Azure 4.0 sono attualmente disponibili solo in queste aree.

3. Selezionare le caselle di controllo necessarie e creare la risorsa.
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

È anche necessario un account di archiviazione per archiviare le immagini di training.

1. Nel portale di Azure cercare e selezionare **Account di archiviazione** e creare un nuovo account di archiviazione con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *Scegliere lo stesso gruppo di risorse creato nella risorsa del servizio di Azure per intelligenza artificiale*
    - **Nome account di archiviazione**: customclassifySUFFIX 
        - *nota: sostituire il token `SUFFIX` con le iniziali o un altro valore per assicurarsi che il nome della risorsa sia univoco a livello globale.*
    - **Area**: *Scegliere la stessa area usata per la risorsa del servizio di intelligenza artificiale di Azure*
    - **Prestazioni**: standard
    - **Ridondanza**: Archiviazione con ridondanza locale
1. Durante la creazione dell'account di archiviazione, passare a Visual Studio Code ed espandere la cartella **Labfiles/02-image-classification**.
1. In tale cartella selezionare **replace.ps1** ed esaminare il codice. Si noterà che sostituisce il nome dell'account di archiviazione per il segnaposto in un file JSON (il file COCO) usato in un passaggio successivo. Sostituire il segnaposto *solo nella prima riga* del file con il nome dell'account di archiviazione. Salvare il file.
1. Fare clic con il pulsante destro del mouse sulla cartella **02-image-classification** e aprire un terminale integrato. Esegui il comando seguente:

    ```powershell
    ./replace.ps1
    ```

1. È possibile esaminare il file COCO per assicurarsi che il nome dell'account di archiviazione sia presente. Selezionare **training-images/training_labels.json** e visualizzare le prime voci. Nel campo *absolute_url* dovrebbe essere visualizzato qualcosa di simile a *"https://myStorage.blob.core.windows.net/fruit/...*. Se non viene visualizzata la modifica prevista, assicurarsi di aggiornare solo il primo segnaposto nello script di PowerShell.
1. Chiudere sia il file JSON che il file PowerShell e tornare alla finestra del browser.
1. L'account di archiviazione deve essere completato. Passare all'account di archiviazione.
1. Abilitare l'accesso pubblico nell'account di archiviazione. Nel riquadro sinistro passare a **Configurazione** nel gruppo **Impostazioni** e abilitare *Consenti l'accesso anonimo al BLOB*. Seleziona **Salva**
1. Nel riquadro sinistro, selezionare **Contenitori** e creare un nuovo contenitore denominato `fruit`, e impostare **Livello di accesso anonimo** su *Contenitore (accesso in lettura anonimo per contenitori e BLOB)*.

    > **Nota**: Se il **livello di accesso anonimo** è disabilitato, aggiornare la pagina del browser.

1. Passare a `fruit` e caricare le immagini (e il file JSON) in **Labfiles/02-image-classification/training-images** in tale contenitore.

## Creare un progetto di training del modello personalizzato

Si creerà quindi un nuovo progetto di training per la classificazione di immagini personalizzate in Vision Studio.

1. Nel Web browser passare a `https://portal.vision.cognitive.azure.com/` e accedere con l'account Microsoft in cui è stata creata la risorsa Azure per intelligenza artificiale.
1. Selezionare il riquadro **Personalizza modelli con immagini** (è disponibile nella scheda **Analisi immagini** se non viene visualizzato nella visualizzazione predefinita) e, se richiesto, selezionare la risorsa di Azure per intelligenza artificiale creata.
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
