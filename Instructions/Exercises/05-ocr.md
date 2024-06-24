---
lab:
  title: Leggere il testo nelle immagini
  module: Module 11 - Reading Text in Images and Documents
---

# Leggere il testo nelle immagini

Il riconoscimento ottico dei caratteri (OCR) è un subset di Visione artificiale che riguarda la lettura di testo in immagini e documenti. Il servizio **Visione di Azure AI** fornisce un'API per la lettura del testo, che verrà esaminata in questo esercizio.

## Clonare il repository per questo corso

Se non è già stato fatto, è necessario clonare il repository del codice per questo corso:

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-ai-vision` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**. Se viene visualizzato il messaggio *È stato rilevato un progetto di Funzioni di Azure nella cartella*, è possibile chiuderlo senza problemi.

## Effettuare il provisioning di una risorsa di Servizi di Azure AI

Se non è già disponibile nella sottoscrizione, sarà necessario effettuare il provisioning di una risorsa di **Servizi di Azure AI**.

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Nella barra di ricerca superiore cercare *Servizi di Azure AI*, selezionare **Servizi di Azure AI** e creare una risorsa account multiservizio di Servizi di Azure AI con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, si potrebbe non essere autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *Scegliere tra Stati Uniti orientali, Francia centrale, Corea centrale, Europa settentrionale, Asia sud-orientale, Europa occidentale, Stati Uniti occidentali e Asia orientale\**
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S0.

    \*Le funzionalità di Visione di Azure AI 4.0 sono attualmente disponibili solo in queste aree.

3. Selezionare le caselle di controllo necessarie e creare la risorsa.
4. Attendere il completamento della distribuzione e quindi visualizzare i relativi dettagli.
5. Al termine della distribuzione, passare alla risorsa e visualizzare la rispettiva pagina **Chiavi ed endpoint**. Nella procedura successiva, saranno necessari l'endpoint e una delle chiavi indicate in questa pagina.

## Prepararsi all'uso dell'SDK di Visione di Azure AI

In questo esercizio, verrà completata un'applicazione client parzialmente implementata che usa l'SDK di Visione di Azure AI per leggere il testo.

> **Nota**: è possibile scegliere di usare l'SDK per **C#** o **Python**. Nei passaggi seguenti, eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Esplora risorse** di Visual Studio Code, esplorare la cartella **Labfiles\05-ocr** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella **read-text** e aprire un terminale integrato. Installare quindi il pacchetto SDK di Visione di Azure AI eseguendo il comando appropriato per il linguaggio scelto:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **Nota**: Se viene richiesto di installare le estensioni del kit di sviluppo, è possibile chiudere il messaggio in modo sicuro.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
    ```

3. Visualizzare il contenuto della cartella **read-text** e notare che include un file per le impostazioni di configurazione:

    - **C#**: appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori in modo che includano l'**endpoint** e una **chiave** di autenticazione per la risorsa di Servizi di Azure AI. Salva le modifiche.


## Usare l'SDK di Visione di Azure AI per leggere il testo da un'immagine

una delle funzionalità dell'**SDK di Visione di Azure AI**consiste nel leggere il testo da un'immagine. In questo esercizio verrà completata un'applicazione client parzialmente implementata che usa l'SDK di Visione di Azure AI per leggere il testo da un'immagine.

1. La cartella **read-text** contiene un file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: read-text.py

    Aprire il file di codice e nella parte superiore, sotto i riferimenti agli spazi dei nomi esistenti, individuare il commento **Import namespaces**. In questo commento aggiungere quindi il codice seguente specifico del linguaggio per importare gli spazi dei nomi necessari per usare l'SDK di Visione di Azure AI:

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

2. Nel file di codice dell'applicazione client, nella funzione **Main**, è stato fornito il codice per caricare le impostazioni di configurazione. Individuare quindi il commento **Autenticare il client di Visione di Azure AI**. Sotto questo commento aggiungere il codice specifico del linguaggio seguente per creare e autenticare un oggetto client di Visione di Azure AI:

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

3. Nella funzione **Main**, sotto il codice appena aggiunto, si noti che il codice specifica il percorso di un file di immagine e quindi lo passa alla funzione **GetTextRead.** Questa funzione non è ancora completamente implementata.

4. Aggiungere codice al corpo della funzione **GetTextRead**. Individuare il commento **Usare la funzione Analizza immagine per leggere il testo nell'immagine**. In questo commento aggiungere quindi il codice specifico del linguaggio seguente, notando che le funzionalità di visualizzazione vengono specificate quando si chiama la funzione `Analyze`:

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
    
        // Prepare image for drawing
        System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.Cyan, 3);
        
        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save image
        String output_file = "text.jpg";
        image.Save(output_file);
        Console.WriteLine("\nResults saved in " + output_file + "\n");   
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

5. Nel codice appena aggiunto alla funzione **GetTextRead** e al commento **Restituire il testo rilevato nell'immagine**, aggiungere il codice seguente (questo codice stampa il testo dell'immagine nella console e genera l'immagine **text.jpg** che evidenzia il testo dell'immagine):

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    var drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
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

6. Nella cartella **read-text/images**, selezionare **Lincoln.jpg** per visualizzare il file che viene elaborato dal codice.

7. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione di menu **1**. Questo codice chiama la funzione **GetTextRead**, passando il percorso al file di immagine *Lincoln.jpg*.

8. Salvare le modifiche e tornare nel terminale integrato per la cartella **read-text**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

9. Quando richiesto, immettere **1** e osservare l'output, ovvero il testo estratto dall'immagine.

10. Nella cartella **read-text**, selezionare l'immagine **text.jpg** e notare come sia presente un poligono intorno a ogni *riga* di testo.

11. Tornare al file di codice in Visual Studio Code e individuare il commento **Restituire il rettangolo di selezione della posizione intorno a ogni riga**. Sotto questo commento aggiungere il codice seguente:

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

12. Salvare le modifiche e tornare nel terminale integrato per la cartella **read-text**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

13. Quando richiesto, immettere **1** e osservare l'output, che deve essere ogni riga di testo nell'immagine con la rispettiva posizione nell'immagine.


14. Tornare al file di codice in Visual Studio Code e individuare il commento **Restituire ogni parola rilevata nell'immagine e il rettangolo di selezione della posizione intorno a ogni parola con il livello di attendibilità di ogni parola**. Sotto questo commento aggiungere il codice seguente:

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
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

15. Salvare le modifiche e tornare nel terminale integrato per la cartella **read-text**, quindi immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

16. Quando richiesto, immettere **1** e osservare l'output, che deve essere ogni parola di testo nell'immagine con la rispettiva posizione nell'immagine. Si noti che viene restituito anche il livello di attendibilità di ogni parola.

17. Nella cartella **read-text**, selezionare l'immagine **text.jpg** e notare come sia presente un poligono intorno a ogni *parola*.

## Usare l'SDK di Visione di Azure AI per leggere il testo scritto a mano da un'immagine

Nell'esercizio precedente, si legge il testo ben definito da un'immagine, ma talvolta si può leggere testo da note o documenti scritti a mano. La buona notizia è che l'**SDK di Visione di Azure AI** può anche leggere testo scritto a mano con lo stesso codice esatto usato per leggere testo ben definito. Verrà usato lo stesso codice dell'esercizio precedente, ma questa volta si userà un'immagine diversa.

1. Nella cartella **read-text/images**, selezionare **Note.jpg** per visualizzare il file che viene elaborato dal codice.

2. Nel file di codice dell'applicazione, nella funzione **Main**, esaminare il codice che viene eseguito se l'utente seleziona l'opzione **2**. Questo codice chiama la funzione **GetTextRead**, passando il percorso al file di immagine *Note.jpg*.

3. Nel terminale integrato per la cartella **read-text**, immettere il comando seguente per eseguire il programma:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

4. Quando richiesto, immettere **2** e osservare l'output, ovvero il testo estratto dall'immagine della nota.

5. Nella cartella **read-text**, selezionare l'immagine **text.jpg** e notare come sia presente un poligono intorno a ogni *parola* della nota.

## Pulire le risorse

Se non si usano le risorse di Azure create in questo lab per altri moduli di training, è possibile eliminarle per evitare addebiti aggiuntivi. In tal caso, eseguire la procedura seguente:

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.

2. Nella barra di ricerca superiore, cercare *account multiservizio di Servizi di Azure AI* e selezionare la risorsa dell'account multiservizio di Servizi di Azure AI creata in questo lab

3. Nella pagina della risorsa, selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

## Ulteriori informazioni

Per altre informazioni sull'uso del servizio **Visione di Azure AI** per leggere il testo, vedere la [documentazione di Visione di Azure AI](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr).
