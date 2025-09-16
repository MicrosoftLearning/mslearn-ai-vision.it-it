---
lab:
  title: Analizzare immagini
  description: 'Usare Analisi delle immagini di Visione di Azure AI per analizzare le immagini, suggerire didascalie e tag e rilevare oggetti e persone.'
---

# Analizzare le immagini

Visione di Azure AI è una funzionalità di intelligenza artificiale che consente ai sistemi software di interpretare l'input visivo mediante l'analisi di immagini. In Microsoft Azure il servizio **Visione** di Azure AI fornisce modelli predefiniti per attività comuni di visione artificiale, tra cui analisi di immagini per suggerire didascalia e tag, rilevamento di oggetti comuni e persone. È possibile usare il servizio Visione di Azure AI anche per rimuovere lo sfondo o opacizzare gli oggetti in primo piano nelle immagini.

> **Nota**: Questo esercizio è basato sulla versione non definitiva del software SDK, che può essere soggetta a modifiche. Se necessario, abbiamo usato versioni specifiche dei pacchetti; che potrebbero non riflettere le versioni disponibili più recenti. È possibile che si verifichino alcuni comportamenti, avvisi o errori imprevisti.

Anche se questo esercizio è basato su Python SDK per Visione di Azure, è possibile sviluppare applicazioni per la visione usando più SDK specifici del linguaggio, tra cui:

* [Analisi di Visione di Azure AI per JavaScript](https://www.npmjs.com/package/@azure-rest/ai-vision-image-analysis)
* [Analisi di Visione di Azure AI per Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Vision.ImageAnalysis)
* [Analisi di Visione di Azure AI per Java](https://mvnrepository.com/artifact/com.azure/azure-ai-vision-imageanalysis)

Questo esercizio richiede circa **30** minuti.

## Effettuare il provisioning di una risorsa Visione di Azure AI

Se non è già disponibile nella sottoscrizione, sarà necessario effettuare il provisioning di una risorsa di Visione di Azure AI.

> **Nota**: In questo esercizio si userà una risorsa autonoma di **Visione artificiale**. È anche possibile usare i servizi di Visione di Azure AI in una risorsa multiservizio dei *Servizi di Azure AI*, direttamente o in un progetto di *Fonderia Azure AI*.

1. Aprire il [portale di Azure](https://portal.azure.com) all'indirizzo `https://portal.azure.com` e accedere usando le credenziali di Azure. Chiudere eventuali messaggi di benvenuto o suggerimenti visualizzati.
1. Selezionare **Crea una risorsa**.
1. Nella barra di ricerca cercare `Computer Vision`, selezionare **Visione artificiale** e creare la risorsa con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *creare o selezionare un gruppo di risorse*
    - **Area**: *Scegliere tra **Stati Uniti orientali**, **Stati Uniti occidentali**, **Francia centrale**, **Corea centrale**, **Europa settentrionale**, **Asia sud-orientale**, **Europa occidentale** o **Asia orientale**\**
    - **Nome**: *Un nome valido per la risorsa di Visione artificiale*
    - **Piano tariffario**: gratuito F0.

    \*I set di funzionalità completi di Visione di Azure AI 4.0 attualmente sono disponibili solamente in queste aree.

1. Selezionare le caselle di controllo necessarie e creare la risorsa.
1. Attendere il completamento della distribuzione e quindi visualizzare i relativi dettagli.
1. Quando la risorsa è stata distribuita, passare alla risorsa e nel nodo **Gestione risorse** nel riquadro di spostamento visualizzare la relativa pagina **Chiavi ed endpoint**. Nella procedura successiva saranno necessari l'endpoint e una delle chiavi indicate in questa pagina.

## Sviluppare un'app di analisi delle immagini con l'SDK di Visione di Azure AI

In questo esercizio si completerà un'applicazione client parzialmente implementata che usa l’SDK Visione di Azure AI per analizzare le immagini.

### Preparare la configurazione dell'applicazione

1. Nel portale di Azure usare il pulsante **[\>_]** a destra della barra di ricerca nella parte superiore della pagina per creare una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***PowerShell*** senza archiviazione nella sottoscrizione.

    Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

    > **Nota**: Se il portale chiede di selezionare una risorsa di archiviazione per salvare in modo permanente i file, scegliere **Nessun account di archiviazione richiesto**, selezionare la sottoscrizione in uso e premere **Applica**.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

1. Ridimensionare il riquadro di Cloud Shell per visualizzare comunque la pagina **Chiavi e endpoint** per la risorsa di Visione artificiale.

    > **Suggerimento**" È possibile ridimensionare il riquadro trascinando il bordo superiore. È anche possibile usare i pulsanti Riduci a icona e Ingrandisci per spostarsi tra Cloud Shell e l'interfaccia principale del portale.

1. Nel riquadro Cloud Shell immettere i comandi seguenti per clonare il repository GitHub contenente i file di codice per questo esercizio (digitare il comando o copiarlo negli Appunti e quindi fare clic con il pulsante destro del mouse nella riga di comando e incollarlo come testo normale):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Dopo aver clonato il repository, usare il comando seguente per passare e visualizzare la cartella contenente i file di codice dell'applicazione:   

    ```
   cd mslearn-ai-vision/Labfiles/analyze-images/python/image-analysis
   ls -a -l
    ```

    La cartella contiene i file di configurazione dell'applicazione e i file di codice per l'app. Contiene anche una sottocartella **/images** che contiene alcuni file che potranno essere analizzati dall'app.
    
1. Installare il pacchetto dell'SDK di Visione di Azure AI e altri pacchetti necessari eseguendo i comandi seguenti:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-imageanalysis==1.0.0
    ```

1. Immettere il comando seguente per modificare il file di configurazione per l'app:

    ```
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Nel file di codice aggiornare i valori di configurazione contenuti in modo che corrispondano all'**endpoint** e a una**chiave** di autenticazione per la risorsa di Visione artificiale, copiati dalla relativa pagina **Chiavi ed endpoint** nel portale di Azure.
1. Dopo aver sostituito i segnaposto, usare il comando **CTRL+S** per salvare le modifiche e quindi usare il comando **CTRL+Q** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

### Aggiungere codice per suggerire una didascalia

1. Nella riga di comando di Cloud Shell immettere il comando seguente per aprire il file di codice per l'applicazione client:

    ```
   code image-analysis.py
    ```

    > **Suggerimento**: È consigliabile ingrandire il riquadro di Cloud Shell e spostare la barra di divisione tra la console della riga di comando e l'editor di codice in modo da visualizzare più facilmente il codice.

1. Nel file di codice trovare il commento **Import namespaces** e aggiungere il codice seguente per importare gli spazi dei nomi necessari per usare l'SDK di Visione di Azure AI:

    ```python
   # import namespaces
   from azure.ai.vision.imageanalysis import ImageAnalysisClient
   from azure.ai.vision.imageanalysis.models import VisualFeatures
   from azure.core.credentials import AzureKeyCredential
    ```

1. Nella funzione **Main** si noti che è stato fornito il codice per caricare le impostazioni di configurazione e determinare quale file di immagine deve essere analizzato. Trovare quindi il commento **Authenticate Azure AI Vision client** e aggiungere il codice seguente per creare ed autenticare un oggetto client di Visione di Azure AI. Assicurarsi di mantenere i livelli di rientro corretti:

    ```python
   # Authenticate Azure AI Vision client
   cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key))
    ```

1. Nella funzione **Main** sotto il codice appena aggiunto trovare il commento **Analyze image** e aggiungere il codice seguente:

    ```python
   # Analyze image
   with open(image_file, "rb") as f:
        image_data = f.read()
   print(f'\nAnalyzing {image_file}\n')

   result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.CAPTION,
            VisualFeatures.DENSE_CAPTIONS,
            VisualFeatures.TAGS,
            VisualFeatures.OBJECTS,
            VisualFeatures.PEOPLE],
   )
    ```

1. Trovare il commento **Get image captions**, aggiungere il codice seguente per visualizzare le didascalie delle immagini e le didascalie compatte:

    ```python
   # Get image captions
   if result.caption is not None:
        print("\nCaption:")
        print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))
    
   if result.dense_captions is not None:
        print("\nDense Captions:")
        for caption in result.dense_captions.list:
            print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))
    ```

1. Salvare le modifiche (*CTRL+S*) e ridimensionare i riquadri in modo da poter visualizzare chiaramente la console della riga di comando mantenendo aperto l'editor di codice. Immettere quindi il comando seguente per eseguire il programma con l'argomento **images/street.jpg**:

    ```
   python image-analysis.py images/street.jpg
    ```

1. Osservare l'output, che dovrebbe includere una didascalia suggerita per l'immagine **street.jpg**, che sarà analoga alla seguente:

    ![Una foto di una strada trafficata.](../media/street.jpg)

1. Eseguire di nuovo il programma, questa volta con l'argomento **images/building.jpg**, per visualizzare la didascalia che viene generata per l'immagine **building.jpg**, che sarà analoga alla seguente:

    ![Immagine di un edificio.](../media/building.jpg)

1. Ripetere il passaggio precedente per generare una didascalia per il file **images/person.jpg**, che sarà analoga alla seguente:

    ![Immagine di una persona.](../media/person.jpg)

### Aggiungere codice per generare tag suggeriti

A volte può risultare utile identificare i *tag* rilevanti che forniscono indicazioni sui contenuti di un'immagine.

1. Nell'editor di codice trovare nella funzione **AnalyzeImage** il commento **Get image tags** e aggiungere il codice seguente:

    ```python
   # Get image tags
   if result.tags is not None:
        print("\nTags:")
        for tag in result.tags.list:
            print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
    ```

1. Salvare le modifiche (*CTRL+S*) ed eseguire il programma con l'argomento **images/street.jpg**, osservando che oltre alla didascalia dell'immagine viene visualizzato un elenco di tag suggeriti.
1. Eseguire di nuovo il programma per i file **images/building.jpg** e **images/person.jpg**.

### Aggiungere codice per rilevare e individuare oggetti

1. Nell'editor di codice trovare nella funzione **AnalyzeImage** il commento **Get objects in the image**, quindi aggiungere il codice seguente per elencare gli oggetti rilevati nell'immagine e chiamare la funzione fornita per annotare un'immagine con gli oggetti rilevati:

    ```python
   # Get objects in the image
   if result.objects is not None:
        print("\nObjects in image:")
        for detected_object in result.objects.list:
            # Print object tag and confidence
            print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        # Annotate objects in the image
        show_objects(image_file, result.objects.list)
    ```

1. Salvare le modifiche (*CTRL+S*) ed eseguire il programma con l'argomento **images/street.jpg**, osservando che oltre alla didascalia dell'immagine e ai tag suggeriti viene generato un file denominato **objects.jpg**.
1. Usare il comando **download** (specifico di Azure Cloud Shell) per scaricare il file **objects.jpg**:

    ```
   download objects.jpg
    ```

    Il comando download crea un collegamento popup in basso a destra del browser, che è possibile selezionare per scaricare e aprire il file. L'immagine dovrebbe essere simile alla seguente:

    ![Immagine con caselle di limite oggetto.](../media/objects.jpg)

1. Eseguire di nuovo il programma per i file **images/building.jpg** e **images/person.jpg**, scaricando il file objects.jpg generato dopo ogni esecuzione.

### Aggiungere codice per rilevare e individuare persone

1. Nell'editor di codice trovare nella funzione **AnalyzeImage** il commento **Get people in the image**, quindi aggiungere il codice seguente per elencare le persone rilevate con un livello di attendibilità pari o superiore al 20% e chiamare una funzione fornita per annotarle in un'immagine:

    ```Python
   # Get people in the image
   if result.people is not None:
        print("\nPeople in image:")

        for detected_person in result.people.list:
            if detected_person.confidence > 0.2:
                # Print location and confidence of each person detected
                print(" {} (confidence: {:.2f}%)".format(detected_person.bounding_box, detected_person.confidence * 100))
        # Annotate people in the image
        show_people(image_file, result.people.list)
    ```

1. Salvare le modifiche (*CTRL+S*) ed eseguire il programma con l'argomento **images/street.jpg**, osservando che oltre alla didascalia dell'immagine, ai tag suggeriti e al file objects.jpg, viene generato un file denominato **people.jpg** che include un elenco di posizioni di persone.

1. Usare il comando **download** (specifico di Azure Cloud Shell) per scaricare il file **objects.jpg**:

    ```
   download people.jpg
    ```

    Il comando download crea un collegamento popup in basso a destra del browser, che è possibile selezionare per scaricare e aprire il file. L'immagine dovrebbe essere simile alla seguente:

    ![Immagine con caselle di limite per le persone rilevate.](../media/people.jpg)

1. Eseguire di nuovo il programma per i file **images/building.jpg** e **images/person.jpg**, scaricando il file people.jpg generato dopo ogni esecuzione.

   > **Suggerimento:** Se vengono visualizzate caselle di delimitazione restituite dal modello che non hanno senso, controllare il punteggio di attendibilità JSON e provare ad aumentare il filtro del punteggio di attendibilità nell'app.

## Pulire le risorse

Al termine dell'esplorazione di Visione di Azure AI, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari:

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.

1. Nella barra di ricerca superiore cercare *Visione artificiale* e selezionare la risorsa di Visione artificiale creata in questo lab.

1. Nella pagina della risorsa, selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

