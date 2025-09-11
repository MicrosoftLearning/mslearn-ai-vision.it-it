---
lab:
  title: Rilevare e analizzare i volti
  module: Module 4 - Detecting and Analyze Faces
---

# Rilevare e analizzare i volti

La possibilità di rilevare, analizzare e riconoscere i volti umani è una delle principali funzionalità di intelligenza artificiale. In questo esercizio si esamineranno due servizi di intelligenza artificiale di Azure che è possibile usare per lavorare con i visi nelle immagini: il servizio **Visione di Azure AI** e il servizio **Viso**.

> **Importante**: Questo lab può essere completato senza richiedere alcun accesso aggiuntivo alle funzionalità ristrette.

> **Nota**: Dal 21 giugno 2022 le funzionalità dei Servizi di Azure AI che restituiscono informazioni personali dell'utente finale sono limitate ai clienti a cui è stato concesso l'[accesso limitato](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Inoltre, le funzionalità che deducono lo stato emotivo non sono più disponibili. Per altre informazioni sulle modifiche apportate da Microsoft e sul relativo motivo, vedere [Misure di sicurezza e investimenti responsabili nell'intelligenza artificiale per il riconoscimento facciale](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

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
1. Attendere il completamento della distribuzione e quindi visualizzare i relativi dettagli.
1. Al termine della distribuzione, passare alla risorsa e visualizzare la rispettiva pagina **Chiavi ed endpoint**. Nella procedura successiva saranno necessari l'endpoint e una delle chiavi indicate in questa pagina.

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
   cd mslearn-ai-vision/Labfiles/04-face
    ```

## Prepararsi all'uso dell’SDK Visione di Azure AI

In questo esercizio si completerà un'applicazione client parzialmente implementata che usa l’SDK Visione di Azure AI per analizzare le immagini.

> **Nota**: è possibile scegliere di usare l'SDK per **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Passare alla cartella contenente i file di codice dell'applicazione per il linguaggio preferito:  

    **C#**

    ```
   cd C-Sharp/computer-vision
    ```
    
    **Python**

    ```
   cd Python/computer-vision
    ```

1. Installare il pacchetto dell'SDK di Visione di Azure AI e le dipendenze necessarie eseguendo i comandi appropriati per il linguaggio preferito:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0
    dotnet add package SkiaSharp --version 3.116.1
    dotnet add package SkiaSharp.NativeAssets.Linux --version 3.116.1
    ``` 

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0
    pip install dotenv
    pip install matplotlib
    ```

1. Usando il comando `ls`, è possibile visualizzare il contenuto della cartella **computer-vision**. Si noti che contiene un file per le impostazioni di configurazione:

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

1. Nel file di codice aggiornare i valori di configurazione contenuti in modo che corrispondano all'**endpoint** e a una **chiave** di autenticazione per la risorsa di Visione artificiale.
1. Dopo aver sostituito i segnaposto, usare il comando **CTRL+S** per salvare le modifiche e quindi usare il comando **CTRL+Q** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

## Visualizzare l'immagine che verrà analizzata

In questo esercizio si userà il servizio Visione di Azure AI per analizzare un'immagine di persone.

1. Nella barra degli strumenti di Cloud Shell selezionare **Carica/Scarica file** e quindi **Scarica**. Nella nuova finestra di dialogo immettere il percorso di file seguente e selezionare **Scarica**:

    ```
    mslearn-ai-vision/Labfiles/04-face/C-Sharp/computer-vision/images/people.jpg
    ```

1. Aprire l'immagine **people.jpg** per visualizzarla.

## Rilevare i visi in un'immagine

A questo punto è possibile usare l'SDK per chiamare il servizio Visione e rilevare i visi in un'immagine.

1. Si noti che la cartella **computer-vision** contiene un file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: detect-people.py

1. Immettere il comando seguente per aprire il file di codice dell'applicazione client:

    **C#**

    ```
   code Program.cs
    ```

    **Python**

    ```
   code image-analysis.py
    ```

1. Aggiungere il codice seguente specifico del linguaggio sotto il commento **Import namespaces** per importare gli spazi dei nomi necessari per usare l'SDK di Visione di Azure AI:

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    using SkiaSharp;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```
    
1. Nella funzione **Main** si noti che è stato fornito il codice per caricare le impostazioni di configurazione. Individuare quindi il commento **Autentica client Visione di Azure AI**. Sotto questo commento aggiungere il codice specifico del linguaggio seguente per creare e autenticare un oggetto client Visione di Azure AI:

    **C#**

    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient cvClient = new ImageAnalysisClient(
        new Uri(aiSvcEndpoint),
        new AzureKeyCredential(aiSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Azure AI Vision client
    cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key)
    )
    ```

1. Nella funzione **Main**, sotto il codice appena aggiunto, si noti che il codice specifica il percorso di un file di immagine e quindi lo passa a una funzione denominata **AnalyzeImage.** Questa funzione non è ancora completamente implementata.

1. Nella funzione **AnalyzeImage**, sotto il commento **Ottieni risultato con specifiche caratteristiche da restituire (PERSONE)**, aggiungere il codice seguente:

    **C#**

    ```C#
    // Get result with specified features to be retrieved (PEOPLE)
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        VisualFeatures.People);
    ```

    **Python**

    ```Python
    # Get result with specified features to be retrieved (PEOPLE)
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.PEOPLE],
    )
    ```

1. Nella funzione **AnalyzeImage**, sotto il commento **Disegna rettangoli di selezione intorno alle persone rilevate**, aggiungere il codice seguente:

    **C#**

    ```C
    // Draw bounding box around detected people
    foreach (DetectedPerson person in result.People.Values)
    {
        if (person.Confidence > 0.5) 
        {
            // Draw object bounding box
            var r = person.BoundingBox;
            SKRect rect = new SKRect(r.X, r.Y, r.X + r.Width, r.Y + r.Height);
            canvas.DrawRect(rect, paint);
        }

        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }
    ```

    **Python**
    
    ```Python
    # Draw bounding box around detected people
    for detected_people in result.people.list:
        if(detected_people.confidence > 0.5):
            # Draw object bounding box
            r = detected_people.bounding_box
            bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
            draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
    ```

1. Salvare le modifiche, chiudere l'editor di codice e immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. Osservare l'output, che dovrebbe indicare il numero di visi rilevati.
7. Scaricare il file **people.jpg** generato nella stessa cartella del file di codice per visualizzare i visi annotati. In questo caso, il codice ha usato gli attributi del viso per etichettare la località nella parte in alto a sinistra del riquadro e le coordinate del rettangolo delimitatore per disegnare un rettangolo intorno a ogni viso.

Se si vuole visualizzare il punteggio di attendibilità di tutte le persone rilevate dal servizio, è possibile rimuovere il commento dalla riga di codice sotto il commento `Return the confidence of the person detected` ed eseguire nuovamente il codice.

## Preparare l'uso di Face SDK

Mentre il servizio **Visione di Azure AI** offre funzionalità di rilevamento del viso di base (oltre a molte altre funzionalità di analisi delle immagini), il servizio **Viso** offre funzionalità più complete per l'analisi e il riconoscimento facciale.

1. Passare alla cartella **face-api** eseguendo il comando `cd ../face-api`. Installare quindi il pacchetto Face SDK eseguendo il comando appropriato per il linguaggio scelto:

    **C#**

    ```
    dotnet add package Azure.AI.Vision.Face -v 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-vision-face==1.0.0b2
    ```
    
1. Visualizzare il contenuto della cartella **face-api** e notare che include un file per le impostazioni di configurazione:
    - **C#**: appsettings.json
    - **Python**: .env

1. Aprire il file di configurazione e aggiornare i valori in modo che includano l'**endpoint** e una **chiave** di autenticazione per la risorsa di Servizi di Azure AI. Salva le modifiche.

1. Si noti che la cartella **face-api** contiene un file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: analyze-faces.py

1. Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare Vision SDK:

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Vision.Face;
    using SkiaSharp;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.ai.vision.face import FaceClient
    from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection03
    from azure.core.credentials import AzureKeyCredential
    ```

1. Nella funzione **Main** si noti che è stato fornito il codice per caricare le impostazioni di configurazione. Individuare quindi il commento **Authenticate Face client**. Sotto questo commento aggiungere il codice specifico del linguaggio seguente per creare e autenticare un oggetto **FaceClient**:

    **C#**

    ```C#
    // Authenticate Face client
    faceClient = new FaceClient(
        new Uri(cogSvcEndpoint),
        new AzureKeyCredential(cogSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Face client
    face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key)
    )
    ```

1. Nella funzione **Main**, sotto il codice appena aggiunto, si noti che il codice visualizza un menu che consente di chiamare funzioni per esplorare le funzionalità del servizio Viso. Queste funzioni verranno implementate nella parte restante di questo esercizio.

## Rilevare e analizzare i volti

Una delle funzionalità più importanti del servizio Viso consente di rilevare i visi in un'immagine e determinarne gli attributi, ad esempio posa della testa, sfocatura, presenza di maschere e così via.

1. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione di menu **1**. Questo codice chiama la funzione **DetectFaces,** passando il percorso di un file di immagine.
1. Individuare la funzione **DetectFaces** nel file di codice e quindi, sotto il commento **Specify facial features to be retrieved**, aggiungere i codice seguente:

    **C#**

    ```C#
    // Specify facial features to be retrieved
    FaceAttributeType[] features = new FaceAttributeType[]
    {
        FaceAttributeType.Detection03.HeadPose,
        FaceAttributeType.Detection03.Blur,
        FaceAttributeType.Detection03.Mask
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeTypeDetection03.HEAD_POSE,
                FaceAttributeTypeDetection03.BLUR,
                FaceAttributeTypeDetection03.MASK]
    ```

1. Nella funzione **DetectFaces**, sotto il codice appena aggiunto, individuare il commento **Get faces** e aggiungere il codice seguente:

    **C#**

    ```C#
    // Get faces
    using (var imageData = File.OpenRead(imageFile))
    {    
        var response = await faceClient.DetectAsync(
            BinaryData.FromStream(imageData),
            FaceDetectionModel.Detection03,
            FaceRecognitionModel.Recognition04,
            returnFaceId: false,
            returnFaceAttributes: features);
        IReadOnlyList<FaceDetectionResult> detected_faces = response.Value;

        if (detected_faces.Count() > 0)
        {
            Console.WriteLine($"{detected_faces.Count()} faces detected.");

            // Load the image using SkiaSharp
            using SKBitmap bitmap = SKBitmap.Decode(imageFile);
            using SKCanvas canvas = new SKCanvas(bitmap);

            // Set up paint styles for drawing
            SKPaint rectPaint = new SKPaint
            {
                Color = SKColors.LightGreen,
                StrokeWidth = 3,
                Style = SKPaintStyle.Stroke,
                IsAntialias = true
            };

            SKPaint textPaint = new SKPaint
            {
                Color = SKColors.White,
                TextSize = 16,
                IsAntialias = true
            };

            int faceCount=0;

            // Draw and annotate each face
            foreach (var face in detected_faces)
            {
                faceCount++;
                Console.WriteLine($"\nFace number {faceCount}");
            
                // Get face properties
                Console.WriteLine($" - Head Pose (Yaw): {face.FaceAttributes.HeadPose.Yaw}");
                Console.WriteLine($" - Head Pose (Pitch): {face.FaceAttributes.HeadPose.Pitch}");
                Console.WriteLine($" - Head Pose (Roll): {face.FaceAttributes.HeadPose.Roll}");
                Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
                Console.WriteLine($" - Mask: {face.FaceAttributes.Mask.Type}");

                // Draw and annotate face
                var r = face.FaceRectangle;

                // Create an SKRect from the face rectangle data
                SKRect rect = new SKRect(r.Left, r.Top, r.Left + r.Width, r.Top + r.Height);
                canvas.DrawRect(rect, rectPaint);

                string annotation = $"Face number {faceCount}";
                canvas.DrawText(annotation, r.Left, r.Top, textPaint);
            }

            // Save annotated image
            using (SKFileWStream output = new SKFileWStream("detected_faces.jpg"))
            {
                bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
            }

            Console.WriteLine(" Results saved in detected_faces.jpg");   
        }
    }
    ```

    **Python**

    ```Python
    # Get faces
    with open(image_file, mode="rb") as image_data:
        detected_faces = face_client.detect(
            image_content=image_data.read(),
            detection_model=FaceDetectionModel.DETECTION03,
            recognition_model=FaceRecognitionModel.RECOGNITION04,
            return_face_id=False,
            return_face_attributes=features,
        )

        if len(detected_faces) > 0:
            print(len(detected_faces), 'faces detected.')

            # Prepare image for drawing
            fig = plt.figure(figsize=(8, 6))
            plt.axis('off')
            image = Image.open(image_file)
            draw = ImageDraw.Draw(image)
            color = 'lightgreen'
            face_count = 0

            # Draw and annotate each face
            for face in detected_faces:
    
                # Get face properties
                face_count += 1
                print('\nFace number {}'.format(face_count))

                print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
                print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
                print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
                print(' - Blur: {}'.format(face.face_attributes.blur.blur_level))
                print(' - Mask: {}'.format(face.face_attributes.mask.type))

                # Draw and annotate face
                r = face.face_rectangle
                bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
                draw = ImageDraw.Draw(image)
                draw.rectangle(bounding_box, outline=color, width=5)
                annotation = 'Face number {}'.format(face_count)
                plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

            # Save annotated image
            plt.imshow(image)
            outputfile = 'detected_faces.jpg'
            fig.savefig(outputfile)

            print('\nResults saved in', outputfile)
    ```

1. Esaminare il codice aggiunto alla funzione **DetectFaces**. Il codice analizza un file di immagine e rileva i visi che contiene, inclusi gli attributi per la posizione della testa, la sfocatura e la presenza di maschere. Vengono visualizzati i dettagli di ogni viso, con un identificatore univoco assegnato a ogni viso. Inoltre, sull'immagine è indicata la posizione dei visi con un rettangolo delimitatore.
1. Salvare le modifiche e chiudere l'editor di codice, quindi immettere il comando seguente per eseguire il programma:

    **C#**

    ```
    dotnet run
    ```

    *L'output C# può visualizzare avvisi relativi a funzioni asincrone che non usano l'operatore **await**. È possibile ignorarli.*

    **Python**

    ```
    python analyze-faces.py
    ```

1. Quando richiesto, immettere **1** e osservare l'output, che dovrà includere l'ID e gli attributi di ogni viso rilevato.
1. Scaricare il file **detected_faces.jpg** generato nella stessa cartella del file di codice per visualizzare i visi annotati.

## Pulire le risorse

Se non si usano le risorse di Azure create in questo lab per altri moduli di training, è possibile eliminarle per evitare addebiti aggiuntivi.

1. Aprire il portale di Azure in `https://portal.azure.com` e nella barra di ricerca superiore cercare le risorse create in questo lab.

1. Nella pagina della risorsa selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa. In alternativa, è possibile eliminare l'intero gruppo di risorse per pulire tutte le risorse contemporaneamente.

## Ulteriori informazioni

Sono disponibili diverse funzionalità aggiuntive all'interno del servizio **Viso**, ma secondo lo [standard di IA responsabile](https://aka.ms/aah91ff) queste funzionalità sono soggette ai criteri di Accesso limitato. Queste funzionalità includono l'identificazione, la verifica e la creazione di modelli di riconoscimento facciale. Per altre informazioni e per richiedere l'accesso, vedere [Accesso limitato per i Servizi di Azure AI](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access).

Per altre informazioni sull'uso del servizio **Visione di Azure AI** per il rilevamento dei volti, vedere la [documentazione di Visione di Azure AI](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces).

Per altre informazioni sul servizio **Viso**, vedere la [documentazione di Viso](https://learn.microsoft.com/azure/ai-services/computer-vision/overview-identity).
