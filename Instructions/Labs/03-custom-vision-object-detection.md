---
lab:
  title: Rilevare oggetti nelle immagini con Visione personalizzata di Azure AI
---

# Rilevare oggetti nelle immagini con Visione personalizzata di Azure AI

In questo esercizio si userà il servizio Visione personalizzata per eseguire il training di un modello di *rilevamento oggetti* in grado di rilevare e individuare tre classi di frutti (mela, banana e arancia) in un'immagine.

## Creare risorse di Visione personalizzata

Se nella sottoscrizione di Azure sono già disponibili risorse di **Visione personalizzata** per il training e la previsione, in questo esercizio è possibile usare tali risorse o un account multiservizio esistente. In caso contrario, usare le istruzioni seguenti per crearle.

> **Nota**: se si usa un account multiservizio, la chiave e l'endpoint saranno uguali sia per il training che per la previsione.

1. In una nuova scheda del browser aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Selezionare il pulsante **&#65291;Crea una risorsa**, cercare *visione personalizzata* e creare una risorsa di **Visione personalizzata** con le impostazioni seguenti:
    - **Opzioni di creazione**: Entrambi
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, si potrebbe non essere autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario per il training**: F0
    - **Piano tariffario previsione**: F0

    > **Nota**: se è già disponibile un servizio di visione personalizzata F0 nella sottoscrizione in uso, selezionare **S0**.

1. Attendere la creazione delle risorse e quindi visualizzare i dettagli di distribuzione e osservare che è stato effettuato il provisioning di due risorse di Visione personalizzata: una per il training e un'altra per la stima (suffisso **-Prediction**). È possibile visualizzarle accedendo al gruppo di risorse in cui sono state create.

> **Importante**: per ogni risorsa sono disponibili un *endpoint* e *chiavi* specifiche, che vengono usati per gestire l'accesso dal codice. Per eseguire il training di un modello di classificazione immagini, il codice deve usare la risorsa di *training* (con l'endpoint e la chiave corrispondenti). Per usare il modello con training per prevedere le classi di immagini, il codice deve usare la risorsa di *previsione* (con l'endpoint e la chiave corrispondenti).

## Clonare il repository per questo corso

Verrà sviluppato il codice usando Cloud Shell dal portale di Azure. I file di codice per l'app sono stati forniti in un repository GitHub.

> **Suggerimento**: Se il repository **mslearn-ai-vision** è già stato clonato di recente, è possibile ignorare questa attività. In caso contrario, eseguire i passaggi seguenti per clonarlo nell'ambiente di sviluppo.

1. Nel portale di Azure, usare il pulsante **[\>_]** a destra della barra di ricerca nella parte superiore della pagina per creare una nuova Cloud Shell nel portale di Azure, selezionando un ambiente ***PowerShell***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Nel riquadro PowerShell immettere i comandi seguenti per clonare il repository GitHub per questo esercizio:

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:  

    ```
   cd mslearn-ai-vision/Labfiles/03-object-detection
    ```

## Creare un progetto di Visione personalizzata

Per eseguire il training di un modello di rilevamento oggetti, è necessario creare un progetto Visione personalizzata basato sulla risorsa di training. A questo scopo, si userà il portale di Visione personalizzata.

1. In una nuova scheda del browser aprire il portale di Visione personalizzata all'indirizzo `https://customvision.ai` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
1. Creare un nuovo progetto con le seguenti impostazioni:
    - **Nome**: Detect Fruit
    - **Descrizione**: Object detection for fruit.
    - **Risorsa**: *la risorsa di Visione personalizzata creata in precedenza*
    - **Tipi di progetto**: Rilevamento oggetti
    - **Domini**: Generale
1. Attendere che il progetto venga creato e aperto nel browser.

## Aggiungere e contrassegnare le immagini

Per eseguire il training di un modello di rilevamento oggetti, è necessario caricare immagini contenenti le classi che si desidera far identificare al modello e contrassegnarle per indicare i rettangoli delimitatori per ogni istanza dell'oggetto.

1. Scaricare le immagini di training da `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/03-object-detection/training-images.zip` ed estrarre la cartella ZIP per visualizzarne il contenuto. Questa cartella contiene immagini di frutta.
1. Nel portale Visione personalizzata, nel tuo progetto di rilevamento degli oggetti, seleziona **Aggiungi immagini** e carica tutte le immagini della cartella estratta.
1. Dopo aver caricato le immagini, selezionare la prima per aprirla.
1. Posizionare il mouse su qualsiasi oggetto nell'immagine fino a quando viene visualizzata un'area rilevata automaticamente come nell'immagine seguente. Quindi selezionare l'oggetto e, se necessario, ridimensionare l'area per circondarlo.

    ![L'area predefinita di un oggetto](../media/object-region.jpg)

    In alternativa, è possibile trascinare semplicemente il mouse intorno all'oggetto per creare un'area.

1. Quando l'area circonda l'oggetto, aggiungere un nuovo tag con il tipo di oggetto appropriato (*apple*, *banana* o *orange*) come mostrato di seguito:

    ![Un oggetto con tag in un'immagine](../media/object-tag.jpg)

1. Selezionare ogni altro oggetto nell'immagine e aggiungere i tag, ridimensionando le aree e aggiungendo nuovi tag come necessario.

    ![Due oggetti con tag in un'immagine](../media/object-tags.jpg)

1. Usare il collegamento **>** sulla destra per passare all'immagine successiva e aggiungere un tag ai relativi oggetti. Continuare così per tutta la raccolta di immagini, aggiungendo un tag a ogni mela, banana e arancia.

1. Al termine dell'assegnazione di tag all'ultima immagine, chiudere l'editor **Dettaglio immagine**. Nella pagina **Immagini di training**, in **Tag** selezionare **Contrassegnate** per visualizzare tutte le immagini con tag:

![Immagini con tag in un progetto](../media/tagged-images.jpg)

## Usare l'API Training per caricare le immagini

È possibile usare l'interfaccia utente nel portale di Visione personalizzata per assegnare un tag alle immagini, ma molti team di sviluppo per intelligenza artificiale usano altri strumenti che generano file contenenti informazioni sui tag e sulle aree oggetto nelle immagini. In scenari come questo è possibile usare l'API Training di Visione personalizzata per caricare immagini con tag nel progetto.

> **Nota**: in questo esercizio è possibile scegliere se usare l'API dall'SDK **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Fare clic sull'icona delle *impostazioni* (&#9881;) in alto a destra nella pagina **Immagini training** nel portale di Visione personalizzata per visualizzare le impostazioni del progetto.
1. In **Generale** (sulla sinistra) prendere nota del valore di **ID progetto** che identifica in modo univoco questo progetto.
1. A destra, in **Risorse** osservare la chiave e l'endpoint. Questi dati sono relativi alla risorsa di *training*. È anche possibile ottenere queste informazioni visualizzando la risorsa nel portale di Azure.
1. Nel portale di Azure eseguire il comando `cd C-Sharp/train-detector` o `cd Python/train-detector` in base alle preferenze per il linguaggio.
1. Installare il pacchetto Training di Visione personalizzata eseguendo il comando appropriato per il linguaggio scelto:

    **C#**

    ```
   dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training --version 2.0.0
    ```

    **Python**

    ```
   pip install azure-cognitiveservices-vision-customvision==3.1.1
    ```

1. Usando il comando `ls`, è possibile visualizzare il contenuto della cartella **train-detector**. Si noti che contiene un file per le impostazioni di configurazione:

    - **C#**: appsettings.json
    - **Python**: .env

1. Immettere il comando seguente per modificare il file di configurazione fornito:

    **C#**

    ```
   code appsettings.json
    ```

    **Python**

    ```
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Nel file di codice aggiornare i valori di configurazione in esso contenuti in modo che corrispondano all'**endpoint** e a una **chiave** di autenticazione per la risorsa *training* di Visione personalizzata, quindi all'ID progetto per il progetto di rilevamento oggetti creato in precedenza.
1. Dopo aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.
1. Eseguire `code tagged-images.json` per aprire il file ed esaminare il codice JSON in esso contenuto. Il codice JSON definisce un elenco di immagini, ognuna delle quali contiene una o più aree con tag. Ogni area con tag include un nome di tag e le coordinate top e left, nonché le dimensioni di larghezza e altezza del rettangolo delimitatore contenente l'oggetto con tag.

    > **Nota**: le coordinate e le dimensioni in questo file indicano punti relativi nell'immagine. Ad esempio, un valore *height* pari a 0.7 si riferisce a un rettangolo delimitatore che rappresenta il 70% dell'altezza dell'immagine. Con alcuni strumenti per l'assegnazione di tag vengono generati altri formati di file in cui i valori di coordinate e dimensioni rappresentano pixel, pollici o altre unità di misura.

1. Si noti che la cartella **train-detector** contiene una sottocartella in cui vengono archiviati i file di immagine a cui si fa riferimento nel file JSON.

1. Si noti che la cartella **train-detector** contiene un file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: train-detector.py

    Aprire il file di codice ed esaminare il codice in esso contenuto, notando i dettagli seguenti:
    - Gli spazi dei nomi del pacchetto installato vengono importati
    - La funzione **Main** recupera le impostazioni di configurazione e usa la chiave e l'endpoint per creare un oggetto **CustomVisionTrainingClient** autenticato, che viene quindi usato con l'ID progetto per creare un riferimento **Project** al progetto.
    - La funzione **Upload_Images** estrae le informazioni sull'area contrassegnata dal file JSON e le usa per creare un batch di immagini con aree, che viene quindi caricato nel progetto.

1. Eseguire il comando seguente per eseguire il programma:
    
    **C#**
    
    ```
   dotnet run
    ```
    
    **Python**
    
    ```
   python train-detector.py
    ```
    
1. Attendere il completamento del programma. Tornare quindi al browser e visualizzare la pagina **Immagini training** per il progetto nel portale di Visione personalizzata, se necessario aggiornando il browser.
1. Verificare che alcune nuove immagini con tag siano state aggiunte al progetto.

## Eseguire il training e testare un modello

Dopo aver contrassegnato le immagini nel progetto, è possibile eseguire il training di un modello.

1. Nel progetto di Visione personalizzata, fare clic su **Train** (Esegui training) per eseguire il training di un modello di rilevamento oggetti usando le immagini con tag. Selezionare l'opzione **Quick Training** (Training rapido).
1. Attendere il completamento del training (potrebbe richiedere circa dieci minuti), quindi esaminare le metriche di prestazione *Precision* (Precisione), *Recall* (Richiamo) e *AP*, che misurano l'accuratezza della previsione del modello di classificazione e dovrebbero essere tutte elevate.
1. In alto a destra nella pagina fare clic su **Test rapido** e quindi nella casella **URL immagine** immettere `https://aka.ms/apple-orange` e visualizzare la previsione generata. Quindi chiudere la finestra **Quick Test** (Test rapido).

## Pubblicare il modello di rilevamento oggetti

A questo punto è possibile pubblicare il modello con training per poterlo usare da un'applicazione client.

1. Nella pagina **Prestazioni** del portale di Visione personalizzata fare clic su **&#128504; Pubblica** per pubblicare il modello con training con le impostazioni seguenti:
    - **Nome del modello**: fruit-detector
    - **Risorsa stima**: *la risorsa di **stima** creata in precedenza, che termina con "-Prediction" (<u>non</u> la risorsa di training)*.
1. In alto a sinistra nella pagina **Impostazioni progetto** fare clic sull'icona *Projects Gallery* (Raccolta progetti) (&#128065;) per tornare alla pagina iniziale del portale di Visione personalizzata in cui è ora elencato il progetto appena creato.
1. In alto a destra nella pagina iniziale del portale di Visione personalizzata fare clic sull'icona delle *impostazioni* (&#9881;) per visualizzare le impostazioni del servizio Visione personalizzata. In **Risorse** individuare quindi la risorsa di *stima* che termina con "-Prediction" (<u>non</u> la risorsa di training) per determinare i relativi valori di **Chiave** ed **Endpoint**. È anche possibile ottenere queste informazioni visualizzando la risorsa nel portale di Azure.

## Usare il classificatore di immagini da un'applicazione client

A questo punto, dopo aver pubblicato il modello di classificazione immagini, è possibile usarlo da un'applicazione client. Anche questa volta, è possibile scegliere di usare **C#** o **Python**.

1. Nel portale di Azure passare alla cartella **test-detector** che eseguendo il comando `cd ../test-detector`.
1. Immettere il comando specifico dell'SDK seguente per installare il pacchetto Stima di Visione personalizzata:

    **C#**

    ```
   dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction --version 2.0.0
    ```

    **Python**

    ```
   pip install azure-cognitiveservices-vision-customvision==3.1.1
    ```

> **Nota**: il pacchetto Python SDK include sia pacchetti di training che di previsione e potrebbe essere già installato.

1. Aprire il file di configurazione per l'applicazione client (*appsettings.json* per C# o *.env* per Python) e aggiornare i valori di configurazione in esso contenuti modo che rispecchino l'endpoint e la chiave per la risorsa di *previsione* di Visione personalizzata, l'ID progetto per il progetto di rilevamento oggetti e il nome del modello pubblicato (che dovrebbe essere *fruit-detector*). Salvare le modifiche e chiudere il file.
1. Aprire il file di codice per l'applicazione client (*Program.cs* per C#, *test-detector.py* per Python) esaminare il codice in esso contenuto, notando i dettagli seguenti:
    - Gli spazi dei nomi del pacchetto installato vengono importati
    - La funzione **Main** recupera le impostazioni di configurazione e usa la chiave e l'endpoint per creare un oggetto **CustomVisionPredictionClient** autenticato.
    - L'oggetto client di previsione viene usato per ottenere le previsioni di rilevamento oggetti per l'immagine **produce.jpg**, specificando l'ID progetto e il nome del modello nella richiesta. Le aree con tag stimate vengono quindi disegnate nell'immagine e il risultato viene salvato come **output.jpg**.
1. Chiudere l'editor di codice e immettere il comando seguente per eseguire il programma:

    **C#**

    ```
   dotnet run
    ```

    **Python**

    ```
   python test-detector.py
    ```

1. Al termine del programma, nella barra degli strumenti di Cloud Shell selezionare **Carica/Scarica file** e quindi **Scarica**. Nella nuova finestra di dialogo immettere il percorso di file seguente e selezionare **Scarica**:

    **C#**
   
    ```
   mslearn-ai-vision/Labfiles/03-object-detection/C-Sharp/test-detector/output.jpg
    ```

    **Python**
   
    ```
   mslearn-ai-vision/Labfiles/03-object-detection/Python/test-detector/output.jpg
    ```

1. Visualizzare il file **output.jpg** risultante per vedere gli oggetti rilevati nell'immagine.

## Pulire le risorse

Se non si usano le risorse di Azure create in questo lab per altri moduli di training, è possibile eliminarle per evitare addebiti aggiuntivi.

1. Aprire il portale di Azure in `https://portal.azure.com` e nella barra di ricerca superiore cercare le risorse create in questo lab.

1. Nella pagina della risorsa selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa. In alternativa, è possibile eliminare l'intero gruppo di risorse per pulire tutte le risorse contemporaneamente.
   
## Ulteriori informazioni

Per altre informazioni sul rilevamento oggetti con il servizio Visione personalizzata, vedere la [documentazione di Visione personalizzata](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/).
