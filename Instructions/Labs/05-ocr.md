---
lab:
  title: Leggere il testo nelle immagini
  module: Module 11 - Reading Text in Images and Documents
---

# Leggere il testo nelle immagini

Il riconoscimento ottico dei caratteri (OCR) è un subset di Visione artificiale che riguarda la lettura di testo in immagini e documenti. Il servizio **Visione di Azure AI** fornisce un'API per la lettura del testo, che verrà esaminata in questo esercizio.

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
   cd mslearn-ai-vision/Labfiles/05-ocr
    ```

## Prepararsi all'uso dell'SDK di Visione di Azure AI

In questo esercizio, verrà completata un'applicazione client parzialmente implementata che usa l'SDK di Visione di Azure AI per leggere il testo.

> **Nota**: è possibile scegliere di usare l'SDK per **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Passare alla cartella contenente i file di codice dell'applicazione per il linguaggio preferito:  

    **C#**

    ```
   cd C-Sharp/read-text
    ```
    
    **Python**

    ```
   cd Python/read-text
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
1. Dopo aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

## Usare l'SDK di Visione di Azure AI per leggere il testo da un'immagine

Una delle funzionalità dell'**SDK di Visione di Azure AI** consiste nel leggere il testo da un'immagine. In questo esercizio verrà completata un'applicazione client parzialmente implementata che usa l'SDK di Visione di Azure AI per leggere il testo da un'immagine.

1. La cartella **read-text** contiene un file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: read-text.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare l'SDK di Visione di Azure AI:

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

1. Nel file di codice dell'applicazione client, nella funzione **Main**, è stato fornito il codice per caricare le impostazioni di configurazione. Individuare quindi il commento **Autenticare il client di Visione di Azure AI**. Sotto questo commento aggiungere il codice specifico del linguaggio seguente per creare e autenticare un oggetto client di Visione di Azure AI:

    **C#**
    
    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient client = new ImageAnalysisClient(
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

1. Nella funzione **Main**, sotto il codice appena aggiunto, si noti che il codice specifica il percorso di un file di immagine e quindi lo passa alla funzione **GetTextRead.** Questa funzione non è ancora completamente implementata.

1. Aggiungere codice al corpo della funzione **GetTextRead**. Individuare il commento **Usare la funzione Analizza immagine per leggere il testo nell'immagine**. In questo commento aggiungere quindi il codice specifico del linguaggio seguente, notando che le funzionalità di visualizzazione vengono specificate quando si chiama la funzione `Analyze`:

    **C#**

    ```C#
    // Use Analyze image function to read text in image
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        // Specify the features to be retrieved
        VisualFeatures.Read);
    
    stream.Close();
    
    // Display analysis results
    if (result.Read != null)
    {
        Console.WriteLine($"Text:");
    
        // Load the image using SkiaSharp
        using SKBitmap bitmap = SKBitmap.Decode(imageFile);
        // Create canvas to draw on the bitmap
        using SKCanvas canvas = new SKCanvas(bitmap);

        // Create paint for drawing polygons (bounding boxes)
        SKPaint paint = new SKPaint
        {
            Color = SKColors.Cyan,
            StrokeWidth = 3,
            Style = SKPaintStyle.Stroke,
            IsAntialias = true
        };

        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save the annotated image using SkiaSharp
        using (SKFileWStream output = new SKFileWStream("text.jpg"))
        {
            // Encode the bitmap into JPEG format with full quality (100)
            bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
        }

        Console.WriteLine("\nResults saved in text.jpg\n");
    }
    ```
    
    **Python**
    
    ```Python
    # Use Analyze image function to read text in image
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ]
    )

    # Display the image and overlay it with the extracted text
    if result.read is not None:
        print("\nText:")

        # Prepare image for drawing
        image = Image.open(image_file)
        fig = plt.figure(figsize=(image.width/100, image.height/100))
        plt.axis('off')
        draw = ImageDraw.Draw(image)
        color = 'cyan'

        for line in result.read.blocks[0].lines:
            # Return the text detected in the image

            
        # Save image
        plt.imshow(image)
        plt.tight_layout(pad=0)
        outputfile = 'text.jpg'
        fig.savefig(outputfile)
        print('\n  Results saved in', outputfile)
    ```

1. Nel codice appena aggiunto alla funzione **GetTextRead** e al commento **Restituire il testo rilevato nell'immagine**, aggiungere il codice seguente (questo codice stampa il testo dell'immagine nella console e genera l'immagine **text.jpg** che evidenzia il testo dell'immagine):

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    bool drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

    DrawPolygon(canvas, polygonPoints, paint);
    }
    ```
    
    **Python**
    
    ```Python
    # Return the text detected in the image
    print(f"  {line.text}")    
    
    drawLinePolygon = True
    
    r = line.bounding_polygon
    bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
    
    # Return the position bounding box around each line
    
    
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    # Draw line bounding polygon
    if drawLinePolygon:
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

1. Solo per il file di programma C#, è ancora necessaria una funzione helper per disegnare un poligono. Sotto il commento **Helper method to draw a polygon given an array of SKPoints** aggiungere il codice seguente:

    **C#**
   
    ```C#
    // Helper method to draw a polygon given an array of SKPoints
    static void DrawPolygon(SKCanvas canvas, SKPoint[] points, SKPaint paint)
    {
        if (points == null || points.Length == 0)
            return;

        using (var path = new SKPath())
        {
            path.MoveTo(points[0]);
            for (int i = 1; i < points.Length; i++)
            {
                path.LineTo(points[i]);
            }
            path.Close();
            canvas.DrawPath(path, paint);
        }
    }
    ```

1. Nella funzione **Main** esaminare il codice eseguito se l'utente seleziona l'opzione di menu **1**. Questo codice chiama la funzione **GetTextRead**, passando il percorso al file di immagine *Lincoln.jpg*.

1. Salvare le modifiche e chiudere l'editor di codice.

1. Nella barra degli strumenti di Cloud Shell selezionare **Carica/Scarica file** e quindi **Scarica**. Nella nuova finestra di dialogo immettere il percorso di file seguente e selezionare **Scarica**:

    **C#**
   
    ```
    mslearn-ai-vision/Labfiles/05-ocr/C-Sharp/read-text/images/Lincoln.jpg
    ```

    **Python**

    ```
    mslearn-ai-vision/Labfiles/05-ocr/Python/read-text/images/Lincoln.jpg
    ```
       
1. Aprire l'immagine **Lincoln.jpg** per visualizzarla.

1. Dopo aver visualizzato l'immagine elaborata dal codice, immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Quando richiesto, immettere **1** e osservare l'output, ovvero il testo estratto dall'immagine.

1. Nella cartella **read-text** è stata creata un'immagine **text.jpg**. È possibile scaricarla usando il percorso di file `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/text.jpg`. Si noti come sia presente un poligono intorno a ogni *riga* di testo.

1. Riaprire il file di codice e trovare il commento **Return the position bounding box around each line**. Sotto questo commento aggiungere il codice seguente:

    **C#**
    
    ```C#
    // Return the position bounding box around each line
    Console.WriteLine($"   Bounding Polygon: [{string.Join(" ", line.BoundingPolygon)}]");
    ```
    
    **Python**
    
    ```Python
    # Return the position bounding box around each line
    print("   Bounding Polygon: {}".format(bounding_polygon))
    ```

1. Salvare le modifiche e chiudere l'editor di codice, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Quando richiesto, immettere **1** e osservare l'output, che deve essere ogni riga di testo nell'immagine con la rispettiva posizione nell'immagine.

1. Aprire di nuovo il file di codice e trovare il commento **Return each word detected in the image and the position bounding box around each word with the confidence level of each word**. Sotto questo commento aggiungere il codice seguente:

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        // Convert the bounding polygon into an array of SKPoints    
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

        // Draw the word polygon on the canvas
        DrawPolygon(canvas, polygonPoints, paint);
    }
    ```
    
    **Python**
    
    ```Python
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    for word in line.words:
        r = word.bounding_polygon
        bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
        print(f"    Word: '{word.text}', Bounding Polygon: {bounding_polygon}, Confidence: {word.confidence:.4f}")
    
        # Draw word bounding polygon
        drawLinePolygon = False
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

1. Salvare le modifiche e chiudere l'editor di codice, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Quando richiesto, immettere **1** e osservare l'output, che deve essere ogni parola di testo nell'immagine con la rispettiva posizione nell'immagine. Si noti che viene restituito anche il livello di attendibilità di ogni parola.

1. Scaricare di nuovo l'immagine **text.jpg** e osservare come sia presente un poligono intorno a ogni *parola*.

## Usare l'SDK di Visione di Azure AI per leggere il testo scritto a mano da un'immagine

Nell'esercizio precedente, si legge il testo ben definito da un'immagine, ma talvolta si può leggere testo da note o documenti scritti a mano. La buona notizia è che l'**SDK di Visione di Azure AI** può anche leggere testo scritto a mano con lo stesso codice esatto usato per leggere testo ben definito. Verrà usato lo stesso codice dell'esercizio precedente, ma questa volta si userà un'immagine diversa.

1. Scaricare **Note.jpg** usando il percorso di file `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/images/Note.jpg` per visualizzare l'immagine successiva elaborata dal codice.

1. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione **2**. Questo codice chiama la funzione **GetTextRead**, passando il percorso al file di immagine *Note.jpg*.

1. Dal terminale immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Quando richiesto, immettere **2** e osservare l'output, ovvero il testo estratto dall'immagine della nota.

1. Scaricare di nuovo l'immagine **text.jpg** e osservare come sia presente un poligono intorno a ogni *parola* della nota.

## Pulire le risorse

Se non si usano le risorse di Azure create in questo lab per altri moduli di training, è possibile eliminarle per evitare addebiti aggiuntivi. In tal caso, eseguire la procedura seguente:

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.

1. Nella barra di ricerca superiore cercare *Visione artificiale* e selezionare la risorsa di Visione artificiale creata in questo lab.

1. Nella pagina della risorsa, selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

## Ulteriori informazioni

Per altre informazioni sull'uso del servizio **Visione di Azure AI** per leggere il testo, vedere la [documentazione di Visione di Azure AI](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr).
