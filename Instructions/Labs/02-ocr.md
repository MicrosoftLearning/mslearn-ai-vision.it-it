---
lab:
  title: Leggi il testo nelle immagini
  description: Usare il riconoscimento ottico dei caratteri (OCR) nel servizio Analisi delle immagini di Visione di Azure AI per individuare ed estrarre testo nelle immagini.
---

# Leggi il testo nelle immagini

Il riconoscimento ottico dei caratteri (OCR) è un subset di Visione artificiale che riguarda la lettura di testo in immagini e documenti. Il servizio Analisi delle immagini di **Visione di Azure AI** fornisce un'API per la lettura del testo, che verrà esaminata in questo esercizio.

> **Nota**: Questo esercizio è basato sulla versione non definitiva del software SDK, che può essere soggetta a modifiche. Se necessario, abbiamo usato versioni specifiche dei pacchetti; che potrebbero non riflettere le versioni disponibili più recenti. È possibile che si verifichino alcuni comportamenti, avvisi o errori imprevisti.

Anche se questo esercizio è basato su Python SDK per Analisi di Visione di Azure, è possibile sviluppare applicazioni per la visione usando più SDK specifici del linguaggio, tra cui:

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

## Sviluppare un'app di estrazione di testo con l'SDK di Visione di Azure AI

In questo esercizio verrà completata un'applicazione client parzialmente implementata che usa l'SDK di Visione di Azure AI per estrarre testo da immagini.

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

1. Dopo aver clonato il repository, usare il comando seguente per passare ai file di codice dell'applicazione:

    ```
   cd mslearn-ai-vision/Labfiles/ocr/python/read-text
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

### Aggiungere codice per leggere testo da un'immagine

1. Nella riga di comando di Cloud Shell immettere il comando seguente per aprire il file di codice per l'applicazione client:

    ```
   code read-text.py
    ```

    > **Suggerimento**: È consigliabile ingrandire il riquadro di Cloud Shell e spostare la barra di divisione tra la console della riga di comando e l'editor di codice in modo da visualizzare più facilmente il codice.

1. Nel file di codice trovare il commento **Import namespaces** e aggiungere il codice seguente per importare gli spazi dei nomi necessari per usare l'SDK di Visione di Azure AI:

    ```python
   # import namespaces
   from azure.ai.vision.imageanalysis import ImageAnalysisClient
   from azure.ai.vision.imageanalysis.models import VisualFeatures
   from azure.core.credentials import AzureKeyCredential
    ```

1. Nella funzione **Main** è stato fornito il codice per caricare le impostazioni di configurazione e determinare quale file deve essere analizzato. Trovare quindi il commento **Authenticate Azure AI Vision client** e aggiungere il codice specifico del linguaggio seguente per creare e autenticare un oggetto client di Analisi delle immagini di Visione di Azure AI:

    ```python
   # Authenticate Azure AI Vision client
   cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key))
    ```

1. Nella funzione **Main** sotto il codice appena aggiunto trovare il commento **Read text in image** e aggiungere il codice seguente per usare il client di Analisi delle immagini per leggere il testo nell'immagine:

    ```python
   # Read text in image
   with open(image_file, "rb") as f:
        image_data = f.read()
   print (f"\nReading text in {image_file}")

   result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ])
    ```

1. Trovare il commento **Print the text** e aggiungere il codice seguente, incluso il commento finale, per stampare le righe di testo trovate e chiamare una funzione per annotarle nell'immagine, usando **bounding_polygon** restituito per ogni riga di testo:

    ```python
   # Print the text
   if result.read is not None:
        print("\nText:")
    
        for line in result.read.blocks[0].lines:
            print(f" {line.text}")        
        # Annotate the text in the image
        annotate_lines(image_file, result.read)

        # Find individual words in each line
        
    ```

1. Salvare le modifiche (*CTRL+S*) ma mantenere aperto l'editor di codice nel caso in cui sia necessario correggere eventuali errori di digitazione.

1. Ridimensionare i riquadri in modo da visualizzare una parte più ampia della console, quindi immettere il comando seguente per eseguire il programma:

    ```
   python read-text.py images/Lincoln.jpg
    ```

1. Il programma legge il testo nel file di immagine specificato (*images/Lincoln.jpg*), che sarà simile al seguente:

    ![Fotografia di una statua di Abraham Lincoln.](../media/Lincoln.jpg)

1. Nella cartella **read-text** è stata creata un'immagine **lines.jpg**. Usare il comando **download** (specifico di Azure Cloud Shell) per scaricarla:

    ```
   download lines.jpg
    ```

    Il comando download crea un collegamento popup in basso a destra del browser, che è possibile selezionare per scaricare e aprire il file. L'immagine dovrebbe essere simile alla seguente:

    ![Immagine con il testo evidenziato.](../media/text.jpg)

1. Eseguire di nuovo il programma, questa volta specificando il parametro *images/Business-card.jpg* per estrarre testo dall'immagine seguente:

    ![Immagine di un biglietto da visita di cui è stata eseguita la scansione.](../media/Business-card.jpg)

    ```
   python read-text.py images/Business-card.jpg
    ```

1. Scaricare e visualizzare il file **lines.jpg** risultante:

    ```
   download lines.jpg
    ```

1. Eseguire il programma ancora una volta, specificando il parametro *images/Note.jpg* per estrarre testo da questa immagine:

    ![Fotografia di una lista della spesa scritta a mano.](../media/Note.jpg)

    ```
   python read-text.py images/Note.jpg
    ```

1. Scaricare e visualizzare il file **lines.jpg** risultante:

    ```
   download lines.jpg
    ```

### Aggiungere codice per restituire la posizione delle singole parole

1. Ridimensionare i riquadri in modo da visualizzare una parte più ampia del file di codice. Trovare quindi il commento **Find individual words in each line** e aggiungere il codice seguente, facendo attenzione a mantenere il livello di rientro corretto:

    ```python
   # Find individual words in each line
   print ("\nIndividual words:")
   for line in result.read.blocks[0].lines:
        for word in line.words:
            print(f"  {word.text} (Confidence: {word.confidence:.2f}%)")
   # Annotate the words in the image
   annotate_words(image_file, result.read)
    ```

1. Salvare le modifiche (*CTRL+S*). Nel riquadro della riga di comando eseguire quindi di nuovo il programma per estrarre testo da *images/Lincoln.jpg*.
1. Osservare l'output, che dovrebbe includere ogni singola parola nell'immagine e la confidenza associata alla stima.
1. Nella cartella **read-text** è stata creata un'immagine **words.jpg**. Usare il comando **download** (specifico di Azure Cloud Shell) per scaricarla e visualizzarla:

    ```
   download words.jpg
    ```

1. Eseguire di nuovo il programma per *images/Business-card.jpg* e *images/Note.jpg*, visualizzando il file **words.jpg** generato per ogni immagine.

## Pulire le risorse

Al termine dell'esplorazione di Visione di Azure AI, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari:

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.

1. Nella barra di ricerca superiore cercare *Visione artificiale* e selezionare la risorsa di Visione artificiale creata in questo lab.

1. Nella pagina della risorsa, selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.
