---
lab:
  title: Analizzare le immagini con Visione di Azure AI
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Analizzare le immagini con Visione di Azure AI

Visione di Azure AI è una funzionalità di intelligenza artificiale che consente ai sistemi software di interpretare l'input visivo mediante l'analisi di immagini. In Microsoft Azure il servizio **Visione** di Azure AI fornisce modelli predefiniti per attività comuni di visione artificiale, tra cui analisi di immagini per suggerire didascalia e tag, rilevamento di oggetti comuni e persone. È possibile usare il servizio Visione di Azure AI anche per rimuovere lo sfondo o opacizzare gli oggetti in primo piano nelle immagini.

## Clonare il repository per questo corso

Se il repository di codice **Visione di Azure AI** non è già stato clonato nell'ambiente in cui si sta lavorando a questo lab, seguire questa procedura per clonarlo. In caso contrario, aprire la cartella clonata in Visual Studio Code.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-ai-vision` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**. Se viene presentato il messaggio *È stato rilevato un progetto di Funzioni di Azure nella cartella*, è possibile chiudere il messaggio in modo sicuro.

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
5. Al termine della distribuzione, passare alla risorsa e visualizzare la rispettiva pagina **Chiavi ed endpoint**. Nella procedura successiva saranno necessari l'endpoint e una delle chiavi indicate in questa pagina.

## Prepararsi all'uso dell’SDK Visione di Azure AI

In questo esercizio si completerà un'applicazione client parzialmente implementata che usa l’SDK Visione di Azure AI per analizzare le immagini.

> **Nota**: è possibile scegliere di usare l'SDK per **C#** o **Python**. Nella procedura seguente eseguire le azioni appropriate per il linguaggio scelto.

1. Nel riquadro **Esplora risorse** di Visual Studio Code passare alla cartella **Labfiles/01-analyze-images** ed espandere la cartella **C-Sharp** o **Python** in base al linguaggio scelto.
2. Fare clic con il pulsante destro del mouse sulla cartella **image-analysis** e aprire un terminale integrato. Installare quindi il pacchetto SDK Visione di Azure AI eseguendo il comando appropriato per il linguaggio scelto:

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **Nota**: Se viene richiesto di installare le estensioni del kit di sviluppo, è possibile chiudere il messaggio in modo sicuro.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
    ```
    
3. Visualizzare il contenuto della cartella **image-analysis** e notare che include un file per le impostazioni di configurazione:
    - **C#**: appsettings.json
    - **Python**: .env

    Aprire il file di configurazione e aggiornare i valori in modo che includano l'**endpoint** e una **chiave** di autenticazione per la risorsa di Servizi di Azure AI. Salva le modifiche.
4. Si noti che la cartella **image-analysis** contiene un file di codice per l'applicazione client:

    - **C#**: Program.cs
    - **Python**: image-analysis.py

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
    
## Visualizzare le immagini che verranno analizzate

In questo esercizio si userà il servizio Visione di Azure AI per analizzare più immagini.

1. Nella Visual Studio Code espandere la cartella **image-analysis** e la cartella **images** al suo interno.
2. Seleziona ognuno dei file di immagine per visualizzarli in Visual Studio Code.

## Analizzare un'immagine per suggerire una didascalia

A questo punto è possibile usare l'SDK per chiamare il servizio Visione e analizzare un'immagine.

1. Nel file di codice dell'applicazione client (**Program.cs** o **image-analysis.py**), nella funzione **Main**, si noti che è stato fornito il codice per caricare le impostazioni di configurazione. Individuare quindi il commento **Autentica client Visione di Azure AI**. Sotto questo commento aggiungere il codice specifico del linguaggio seguente per creare e autenticare un oggetto client Visione di Azure AI:

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

2. Nella funzione **Main**, sotto il codice appena aggiunto, si noti che il codice specifica il percorso di un file di immagine e quindi lo passa ad altre due funzioni (**AnalyzeImage** e **BackgroundForeground**). Queste funzioni non sono ancora completamente implementate.

3. Nella funzione **AnalyzeImage**, sotto il commento **Ottieni risultato con specifiche caratteristiche da restituire **, aggiungere il codice seguente:

**C#**

```C#
// Get result with specified features to be retrieved
ImageAnalysisResult result = client.Analyze(
    BinaryData.FromStream(stream),
    VisualFeatures.Caption | 
    VisualFeatures.DenseCaptions |
    VisualFeatures.Objects |
    VisualFeatures.Tags |
    VisualFeatures.People);
```

**Python**

```Python
# Get result with specified features to be retrieved
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
    
4. Nella funzione **AnalyzeImage**, sotto il commento **Mostra risultati analisi**, aggiungere il codice seguente, inclusi i commenti che indicano dove verrà aggiunto altro codice in seguito:

**C#**

```C#
// Display analysis results
// Get image captions
if (result.Caption.Text != null)
{
    Console.WriteLine(" Caption:");
    Console.WriteLine($"   \"{result.Caption.Text}\", Confidence {result.Caption.Confidence:0.00}\n");
}

// Get image dense captions
Console.WriteLine(" Dense Captions:");
foreach (DenseCaption denseCaption in result.DenseCaptions.Values)
{
    Console.WriteLine($"   Caption: '{denseCaption.Text}', Confidence: {denseCaption.Confidence:0.00}");
}

// Get image tags


// Get objects in the image


// Get people in the image
```

**Python**

```Python
# Display analysis results
# Get image captions
if result.caption is not None:
    print("\nCaption:")
    print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))

# Get image dense captions
if result.dense_captions is not None:
    print("\nDense Captions:")
    for caption in result.dense_captions.list:
        print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get objects in the image


# Get people in the image

```
    
5. Salvare le modifiche e tornare nel terminale integrato per la cartella **image-analysis**, quindi immettere il comando seguente per eseguire il programma con l'argomento **images/street.jpg**:

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. Osservare l'output, che dovrebbe includere una didascalia suggerita per l'immagine **street.jpg**.
7. Eseguire di nuovo il programma, questa volta con l'argomento **images/building.jpg** per visualizzare la didascalia che viene generata per l'immagine **building.jpg**.
8. Ripetere il passaggio precedente per generare una didascalia per il file **images/person.jpg**.

## Ottenere tag suggeriti per un'immagine

A volte può risultare utile identificare i *tag* rilevanti che forniscono indicazioni sui contenuti di un'immagine.

1. Nella funzione **AnalyzeImage**, sotto il commento **Get image tags**, aggiungere il codice seguente:

**C#**

```C#
// Get image tags
if (result.Tags.Values.Count > 0)
{
    Console.WriteLine($"\n Tags:");
    foreach (DetectedTag tag in result.Tags.Values)
    {
        Console.WriteLine($"   '{tag.Name}', Confidence: {tag.Confidence:F2}");
    }
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags.list:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. Salvare le modifiche ed eseguire una volta il programma per ogni file di immagine nella cartella **images** osservando che oltre alla didascalia dell'immagine viene visualizzato un elenco di tag suggeriti.

## Rilevare e individuare oggetti in un'immagine

*Rilevamento oggetti* è una forma specifica di visione artificiale, che consente di identificare i singoli oggetti in un'immagine e la rispettiva posizione, indicata da un rettangolo delimitatore.

1. Nella funzione **AnalyzeImage**, sotto il commento **Get objects in the image**, aggiungere il codice seguente:

**C#**

```C#
// Get objects in the image
if (result.Objects.Values.Count > 0)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    stream.Close();
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedObject detectedObject in result.Objects.Values)
    {
        Console.WriteLine($"   \"{detectedObject.Tags[0].Name}\"");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Tags[0].Name,font,brush,(float)r.X, (float)r.Y);
    }

    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get objects in the image
if result.objects is not None:
    print("\nObjects in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects.list:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height)) 
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.tags[0].name,(r.x, r.y), backgroundcolor=color)

    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. Salvare le modifiche ed eseguire una volta il programma per ogni file di immagine nella cartella **images**, osservando eventuali oggetti identificati. Dopo ogni esecuzione, vedere il file **objects.jpg** generato nella stessa cartella del file di codice per visualizzare gli oggetti annotati.

## Rilevare e individuare le persone in un'immagine

*Rilevamento persone* è una forma specifica di visione artificiale, che consente di identificare le singole persone in un'immagine e la rispettiva posizione, indicata da un rettangolo delimitatore.

1. Nella funzione **AnalyzeImage**, sotto il commento **Ottieni persone nell’immagine**, aggiungere il codice seguente:

**C#**

```C#
// Get people in the image
if (result.People.Values.Count > 0)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedPerson person in result.People.Values)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        
        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }

    // Save annotated image
    String output_file = "persons.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get people in the image
if result.people is not None:
    print("\nPeople in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people.list:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
        draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
        
    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'people.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. (Facoltativo) Rimuovere il commento dal comando **Console.Writeline** nella sezione **Restituisci l'attendibilità della persona rilevata** per esaminare il livello di attendibilità della persona che è stata rilevata in una determinata posizione dell'immagine.

3. Salvare le modifiche ed eseguire una volta il programma per ogni file di immagine nella cartella **images**, osservando eventuali oggetti identificati. Dopo ogni esecuzione, vedere il file **objects.jpg** generato nella stessa cartella del file di codice per visualizzare gli oggetti annotati.

> **Nota**: nelle attività precedenti è stato usato un singolo metodo per analizzare l'immagine e quindi è stato aggiunto in modo incrementale codice per analizzare e visualizzare i risultati. L'SDK fornisce inoltre singoli metodi per suggerire didascalie, identificare tag, rilevare oggetti e così via, consentendo quindi di usare il metodo più appropriato per restituire solo le informazioni necessarie, riducendo le dimensioni del payload di dati da restituire. Per altri dettagli, vedere la [documentazione di .NET SDK](https://learn.microsoft.com/dotnet/api/overview/azure/cognitiveservices/computervision?view=azure-dotnet) o la [documentazione di Python SDK](https://learn.microsoft.com/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision).

## Rimuovere lo sfondo o opacizzare gli oggetti in primo piano di un'immagine

In alcuni casi, potrebbe essere necessario rimuovere lo sfondo di un'immagine o opacizzare gli oggetti in primo piano di tale immagine. Iniziamo con la rimozione dello sfondo.

1. Nel file di codice individuare la funzione **BackgroundForeground** e nel commento** Rimuovi lo sfondo dall'immagine o genera opacizzazione in primo piano**, aggiungere il codice seguente:

**C#**

```C#
// Remove the background from the image or generate a foreground matte
Console.WriteLine($" Background removal:");
// Define the API version and mode
string apiVersion = "2023-02-01-preview";
string mode = "backgroundRemoval"; // Can be "foregroundMatting" or "backgroundRemoval"

string url = $"computervision/imageanalysis:segment?api-version={apiVersion}&mode={mode}";

// Make the REST call
using (var client = new HttpClient())
{
    var contentType = new MediaTypeWithQualityHeaderValue("application/json");
    client.BaseAddress = new Uri(endpoint);
    client.DefaultRequestHeaders.Accept.Add(contentType);
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);

    var data = new
    {
        url = $"https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{imageFile}?raw=true"
    };

    var jsonData = JsonSerializer.Serialize(data);
    var contentData = new StringContent(jsonData, Encoding.UTF8, contentType);
    var response = await client.PostAsync(url, contentData);

    if (response.IsSuccessStatusCode) {
        File.WriteAllBytes("background.png", response.Content.ReadAsByteArrayAsync().Result);
        Console.WriteLine("  Results saved in background.png\n");
    }
    else
    {
        Console.WriteLine($"API error: {response.ReasonPhrase} - Check your body url, key, and endpoint.");
    }
}
```

**Python**

```Python
# Remove the background from the image or generate a foreground matte
print('\nRemoving background from image...')
    
url = "{}computervision/imageanalysis:segment?api-version={}&mode={}".format(endpoint, api_version, mode)

headers= {
    "Ocp-Apim-Subscription-Key": key, 
    "Content-Type": "application/json" 
}

image_url="https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{}?raw=true".format(image_file)  

body = {
    "url": image_url,
}
    
response = requests.post(url, headers=headers, json=body)

image=response.content
with open("background.png", "wb") as file:
    file.write(image)
print('  Results saved in background.png \n')
```
    
2. Salvare le modifiche ed eseguire una volta il programma per ogni file di immagine nella cartella **images**, aprendo il file **background.jpg** generato nella stessa cartella del file di codice per ogni immagine.  Si noterà che lo sfondo è stato rimosso da ognuna delle immagini.

Verrà ora generata un’opacizzazione in primo piano per le immagini.

3. Nel file di codice individuare la funzione **BackgroundForeground** e nel commento **Definisci la versione API e la modalità** modificare la variabile modalità in `foregroundMatting`.

4. Salvare le modifiche ed eseguire una volta il programma per ogni file di immagine nella cartella **images**, aprendo il file **background.jpg** generato nella stessa cartella del file di codice per ogni immagine.  Si noterà che nelle immagini è stata generata un’opacizzazione in primo piano.

## Pulire le risorse

Se non si usano le risorse di Azure create in questo lab per altri moduli di training, è possibile eliminarle per evitare addebiti aggiuntivi. In tal caso, eseguire la procedura seguente:

1. Aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.

2. Nella barra di ricerca superiore cercare *account multiservizio Servizi di Azure AI* e selezionare la risorsa dell'account multiservizio di Servizi di Azure AI creata in questo lab.

3. Nella pagina della risorsa selezionare **Elimina** e seguire le istruzioni per eliminare la risorsa.

## Ulteriori informazioni

In questo esercizio sono state esplorate alcune funzionalità di analisi e modifica delle immagini del servizio Visione di Azure AI. Il servizio include anche funzionalità per il rilevamento di oggetti e persone e altre attività di visione artificiale.

Per altre informazioni sull'uso del servizio **Visione di Azure AI**, vedere la [documentazione Visione di Azure AI](https://learn.microsoft.com/azure/ai-services/computer-vision/).
