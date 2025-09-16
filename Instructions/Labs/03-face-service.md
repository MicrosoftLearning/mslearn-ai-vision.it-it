---
lab:
  title: Rilevare e analizzare i volti
  description: Usare il servizio Viso di Visione di Azure AI per implementare soluzioni di rilevamento e analisi dei volti.
---

# Rilevare e analizzare i volti

La possibilità di rilevare, analizzare e riconoscere i volti umani è una delle principali funzionalità di intelligenza artificiale. In questo esercizio si esaminerà il servizio **Viso** per lavorare con i visi.

> **Nota**: Questo esercizio è basato sulla versione non definitiva del software SDK, che può essere soggetta a modifiche. Se necessario, abbiamo usato versioni specifiche dei pacchetti; che potrebbero non riflettere le versioni disponibili più recenti. È possibile che si verifichino alcuni comportamenti, avvisi o errori imprevisti.

Anche se questo esercizio è basato su Python SDK per Viso di Visione di Azure, è possibile sviluppare applicazioni per la visione usando più SDK specifici del linguaggio, tra cui:

* [Viso di Visione di Azure AI per JavaScript](https://www.npmjs.com/package/@azure-rest/ai-vision-face)
* [Viso di Visione di Azure AI per Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Vision.Face)
* [Viso di Visione di Azure AI per Java](https://central.sonatype.com/artifact/com.azure/azure-ai-vision-face)

Questo esercizio richiede circa **30** minuti.

> **Nota**: Le funzionalità di Servizi di Azure AI che restituiscono informazioni personali dell'utente finale sono limitate ai clienti a cui è stato concesso l'[accesso limitato](https://learn.microsoft.com/legal/cognitive-services/computer-vision/limited-access-identity). Questo esercizio non include le attività di riconoscimento facciale e può essere completato senza richiedere alcun accesso aggiuntivo alle funzionalità con restrizioni.

## Effettuare il provisioning di una risorsa dell'API Viso di Azure AI

Se non è già disponibile nella sottoscrizione, sarà necessario effettuare il provisioning di una risorsa dell'API Viso di Azure AI.

> **Nota**: In questo esercizio si userà una risorsa di **Viso** autonoma. È anche possibile usare i servizi di Viso di Azure AI in una risorsa multiservizio dei *Servizi di Azure AI*, direttamente o in un progetto di *Fonderia Azure AI*.

1. Aprire il [portale di Azure](https://portal.azure.com) all'indirizzo `https://portal.azure.com` e accedere usando le credenziali di Azure. Chiudere eventuali messaggi di benvenuto o suggerimenti visualizzati.
1. Selezionare **Crea una risorsa**.
1. Nella barra di ricerca cercare `Face`, selezionare **Viso** e creare la risorsa con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *creare o selezionare un gruppo di risorse*
    - **Area**: *scegliere una qualsiasi area disponibile*
    - **Nome**: *Un nome valido per la risorsa di Viso*
    - **Piano tariffario**: gratuito F0.

1. Creare la risorsa e attendere il completamento della distribuzione, quindi visualizzare i relativi dettagli.
1. Quando la risorsa è stata distribuita, passare alla risorsa e nel nodo **Gestione risorse** nel riquadro di spostamento visualizzare la relativa pagina **Chiavi ed endpoint**. Nella procedura successiva saranno necessari l'endpoint e una delle chiavi indicate in questa pagina.

## Sviluppare un'app di analisi facciale con l'SDK di Viso

In questo esercizio si completerà un'applicazione client parzialmente implementata che usa l'SDK di Viso di Azure per rilevare e analizzare i visi nelle immagini.

### Preparare la configurazione dell'applicazione

1. Nel portale di Azure usare il pulsante **[\>_]** a destra della barra di ricerca nella parte superiore della pagina per creare una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***PowerShell*** senza archiviazione nella sottoscrizione.

    Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

    > **Nota**: Se il portale chiede di selezionare una risorsa di archiviazione per salvare in modo permanente i file, scegliere **Nessun account di archiviazione richiesto**, selezionare la sottoscrizione in uso e premere **Applica**.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

1. Ridimensionare il riquadro di Cloud Shell per visualizzare comunque la pagina **Chiavi e endpoint** per la risorsa di Viso.

    > **Suggerimento**" È possibile ridimensionare il riquadro trascinando il bordo superiore. È anche possibile usare i pulsanti Riduci a icona e Ingrandisci per spostarsi tra Cloud Shell e l'interfaccia principale del portale.

1. Nel riquadro Cloud Shell immettere i comandi seguenti per clonare il repository GitHub contenente i file di codice per questo esercizio (digitare il comando o copiarlo negli Appunti e quindi fare clic con il pulsante destro del mouse nella riga di comando e incollarlo come testo normale):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Dopo aver clonato il repository, usare il comando seguente per passare ai file di codice dell'applicazione:

    ```
   cd mslearn-ai-vision/Labfiles/face/python/face-api
   ls -a -l
    ```

    La cartella contiene i file di configurazione dell'applicazione e i file di codice per l'app. Contiene anche una sottocartella **/images** che contiene alcuni file che potranno essere analizzati dall'app.

1. Installare il pacchetto dell'SDK di Visione di Azure AI e altri pacchetti necessari eseguendo i comandi seguenti:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-face==1.0.0b2
    ```

1. Immettere il comando seguente per modificare il file di configurazione per l'app:

    ```
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Nel file di codice aggiornare i valori di configurazione contenuti in modo che corrispondano all'**endpoint** e a una**chiave** di autenticazione per la risorsa di Viso, copiati dalla relativa pagina **Chiavi ed endpoint** nel portale di Azure.
1. Dopo aver sostituito i segnaposto, usare il comando **CTRL+S** per salvare le modifiche e quindi usare il comando **CTRL+Q** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

### Aggiungere codice per creare un client dell'API Viso

1. Nella riga di comando di Cloud Shell immettere il comando seguente per aprire il file di codice per l'applicazione client:

    ```
   code analyze-faces.py
    ```

    > **Suggerimento**: È consigliabile ingrandire il riquadro di Cloud Shell e spostare la barra di divisione tra la console della riga di comando e l'editor di codice in modo da visualizzare più facilmente il codice.

1. Nel file di codice trovare il commento **Import namespaces** e aggiungere il codice seguente per importare gli spazi dei nomi necessari per usare l'SDK di Visione di Azure AI:

    ```python
   # Import namespaces
   from azure.ai.vision.face import FaceClient
   from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection01
   from azure.core.credentials import AzureKeyCredential
    ```

1. Nella funzione **Main** si noti che è stato fornito il codice per caricare le impostazioni di configurazione e determinare quale immagine deve essere analizzato. Trovare quindi il commento **Authenticate Face client** e aggiungere il codice seguente per creare ed autenticare un oggetto **FaceClient**:

    ```python
   # Authenticate Face client
   face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key))
    ```

### Aggiungere codice per rilevare e analizzare i visi

1. Nella funzione **Main** nel file di codice per l'applicazione trovare il commento **Specify facial features to be retrieved** e aggiungere il codice seguente:

    ```python
   # Specify facial features to be retrieved
   features = [FaceAttributeTypeDetection01.HEAD_POSE,
                FaceAttributeTypeDetection01.OCCLUSION,
                FaceAttributeTypeDetection01.ACCESSORIES]
    ```

1. Nella funzione **Main** sotto il codice appena aggiunto trovare il commento **Get faces** e aggiungere il codice seguente per stampare le informazioni sulle caratteristiche del viso e chiamare una funzione che annota l'immagine con il rettangolo di selezione per ogni viso rilevato, in base alla proprietà **face_rectangle** di ogni viso:

    ```Python
   # Get faces
   with open(image_file, mode="rb") as image_data:
        detected_faces = face_client.detect(
            image_content=image_data.read(),
            detection_model=FaceDetectionModel.DETECTION01,
            recognition_model=FaceRecognitionModel.RECOGNITION01,
            return_face_id=False,
            return_face_attributes=features,
        )

   face_count = 0
   if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')
        for face in detected_faces:
    
            # Get face properties
            face_count += 1
            print('\nFace number {}'.format(face_count))
            print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
            print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
            print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
            print(' - Forehead occluded?: {}'.format(face.face_attributes.occlusion["foreheadOccluded"]))
            print(' - Eye occluded?: {}'.format(face.face_attributes.occlusion["eyeOccluded"]))
            print(' - Mouth occluded?: {}'.format(face.face_attributes.occlusion["mouthOccluded"]))
            print(' - Accessories:')
            for accessory in face.face_attributes.accessories:
                print('   - {}'.format(accessory.type))
            # Annotate faces in the image
            annotate_faces(image_file, detected_faces)
    ```

1. Esaminare il codice aggiunto alla funzione **Main**. Analizza un file di immagine e rileva i visi contenuti in tale file, inclusi gli attributi per la posizione della testa, l'occlusione e la presenza di accessori come gli occhiali. Viene inoltre chiamata una funzione per annotare l'immagine originale con un rettangolo delimitatore per ogni viso rilevato.
1. Salvare le modifiche (*CTRL+S*) ma mantenere aperto l'editor di codice nel caso in cui sia necessario correggere eventuali errori di digitazione.

1. Ridimensionare i riquadri in modo da visualizzare una parte più ampia della console, quindi immettere il comando seguente per eseguire il programma con l'argomento *images/face1.jpg*:

    ```
   python analyze-faces.py images/face1.jpg
    ```

    L'app viene eseguita e analizzata l'immagine seguente:

    ![Fotografia di una statua di una persona.](../media/face1.jpg)

1. Osservare l'output, che dovrebbe includere l'ID e gli attributi di ogni viso rilevato. 
1. Si noti che viene generato anche un file di immagine denominato **detected_faces.jpg**. Usare il comando **download** (specifico di Azure Cloud Shell) per scaricarlo:

    ```
   download detected_faces.jpg
    ```

    Il comando download crea un collegamento popup in basso a destra del browser, che è possibile selezionare per scaricare e aprire il file. L'immagine dovrebbe essere simile alla seguente:

    ![Immagine con il viso evidenziato.](../media/detected_faces1.jpg)

1. Eseguire di nuovo il programma, questa volta specificando il parametro *images/face2.jpg* per estrarre testo dall'immagine seguente:

    ![Immagine di un'altra persona.](../media/face2.jpg)

    ```
   python analyze-faces.py images/face2.jpg
    ```

1. Scaricare e visualizzare il file **detected_faces.jpg** risultante:

    ```
   download detected_faces.jpg
    ```

    L'immagine risultante dovrebbe essere simile alla seguente:

    ![Un'altra immagine con un viso evidenziato.](../media/detected_faces2.jpg)

1. Eseguire il programma ancora una volta, specificando il parametro *images/faces.jpg* per estrarre testo da questa immagine:

    ![Foto di entrambe le persone.](../media/faces.jpg)

    ```
   python analyze-faces.py images/faces.jpg
    ```

1. Scaricare e visualizzare il file **detected_faces.jpg** risultante:

    ```
   download detected_faces.jpg
    ```

    L'immagine risultante dovrebbe essere simile alla seguente:

    ![Un'altra immagine con un viso evidenziato.](../media/detected_faces3.jpg)

## Pulire le risorse

Al termine dell'esplorazione di Visione di Azure AI, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari:

1. Aprire il portale di Azure in `https://portal.azure.com` e nella barra di ricerca superiore cercare le risorse create in questo lab.

1. Nella pagina della risorsa selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa. In alternativa, è possibile eliminare l'intero gruppo di risorse per pulire tutte le risorse contemporaneamente.
