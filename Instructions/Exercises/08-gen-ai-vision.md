---
lab:
  title: Sviluppare un'app di chat abilitata per la visione
  description: Informazioni su come usare Fonderia Azure AI per creare un'app di IA generativa che supporta input di immagini.
---

# Sviluppare un'app di chat abilitata per la visione

In questo esercizio, verrà usato il modello di IA generativa *Phi-4-multimodal-instruct* per generare risposte a richieste che includono immagini. Verrà sviluppata un'applicazione che fornisce assistenza AI per i prodotti freschi in un negozio di alimentari usando Fonderia Azure AI e il servizio dell'inferenza del modello di Azure per intelligenza artificiale.

Questo esercizio richiede circa **30** minuti.

## Creare un progetto Fonderia Azure AI

Per iniziare, creare un progetto Fonderia Azure AI.

1. In un Web browser, aprire il [Portale Fonderia Azure AI](https://ai.azure.com) su `https://ai.azure.com` e accedere usando le credenziali di Azure. Chiudere i suggerimenti o i riquadri di avvio rapido aperti la prima volta che si accede e, se necessario, usare il logo **Fonderia Azure AI** in alto a sinistra per passare alla home page, simile all'immagine seguente:

    ![Screenshot del portale di Azure AI Foundry.](../media/ai-foundry-home.png)

2. Nella home page, selezionare **+ Crea progetto**.
3. Nella procedura guidata **Crea un progetto**, immettere un nome appropriato per il progetto. Se viene suggerito un hub esistente, selezionare l'opzione per crearne uno nuovo. Successivamente, esaminare le risorse Azure che verranno create automaticamente per supportare l'hub e il progetto.
4. Selezionare **Personalizza** e specificare le impostazioni seguenti per l'hub:
    - **Nome hub**: *un nome valido per l'hub*
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *creare o selezionare un gruppo di risorse*
    - **Posizione**: selezionare una delle aree seguenti\*:
        - Stati Uniti orientali
        - Stati Uniti orientali 2
        - Stati Uniti centro-settentrionali
        - Stati Uniti centro-meridionali
        - Svezia centrale
        - Stati Uniti occidentali
        - Stati Uniti occidentali 3
    - **Connettere Servizi di Azure AI o Azure OpenAI**: *Creare una nuova risorsa di Servizi di AI*
    - **Connettere Azure AI Search**: ignorare la connessione

    > \* Al momento della stesura del presente documento, il modello Microsoft *Phi-4-multimodal-instruct* che verrà usato in questo esercizio è disponibile in queste regioni. È possibile controllare la disponibilità a livello di area più recente per modelli specifici nella documentazione di [Fonderia Azure AI ](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability). In caso di raggiungimento di un limite di quota di area più avanti nell'esercizio, potrebbe essere necessario creare un'altra risorsa in un'area diversa.

5. Selezionare **Avanti** per esaminare la configurazione. Quindi selezionare **Crea** e attendere il completamento del processo.
6. Quando viene creato il progetto, chiudere tutti i suggerimenti visualizzati e rivedere la pagina del progetto nel portale Fonderia di Azure AI, che dovrebbe essere simile all'immagine seguente:

    ![Screenshot dei dettagli di un progetto di Azure AI nel portale Fonderia di Azure AI.](../media/ai-foundry-project.png)

## Distribuire un modello multimodale

A questo punto è possibile distribuire un modello multimodale in grado di supportare l'input basato su immagini. Esistono diversi modelli tra cui scegliere, compreso il modello OpenAI *gpt-4o*. In questo esercizio, verrà utilizzato un modello *Phi-4-multimodal-instruct* che supporta le richieste che includono immagini.

1. Nella barra degli strumenti nella parte superiore destra della pagina del progetto Fonderia Azure AI, usare l'icona **Funzionalità di anteprima** (**&#9215;**) per assicurarsi che la funzionalità **Distribuisci modelli nel servizio di inferenza del modello di Azure per intelligenza artificiale** sia abilitata. Questa funzionalità garantisce che la distribuzione del modello sia disponibile per il servizio di inferenza di Azure AI, che verrà usato nel codice dell'applicazione.
2. Nel riquadro a sinistra del progetto, nella sezione **Risorse personali** selezionare la pagina **Modelli + endpoint**.
3. Nella scheda **Distribuzioni del modello** della pagina **Modelli + endpoint**, nel menu **+ Distribuisci modello** selezionare **Distribuisci modello di base**.
4. Cercare il modello **Phi-4-multimodal-instruct** nell'elenco, quindi selezionarlo e confermarlo.
5. Accettare il contratto di licenza se richiesto e quindi distribuire il modello con le impostazioni seguenti selezionando **Personalizza** nei dettagli della distribuzione:
    - **Nome distribuzione**: *nome univoco per la distribuzione del modello*
    - **Tipo di distribuzione**: standard globale
    - **Dettagli della distribuzione**: *usare le impostazioni predefinite*
6. Attendere che lo stato di provisioning della distribuzione sia **completato**.

## Testare il modello nel playground

Dopo aver distribuito un modello multimodale, è possibile testarlo con una richiesta basata su immagini nel playground della chat.

1. Nel riquadro di spostamento a sinistra, selezionare la pagina **Playground** e aprire il playground **Chat**.
1. 1. In una nuova scheda del browser, scaricare [mango.jpeg](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/mango.jpeg) da `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/mango.jpeg` e salvarlo in una cartella nel file system locale.
1. Nella pagina playground della chat, nel riquadro **Configurazione**, assicurarsi che sia selezionato il modello di distribuzione **Phi-4-multimodal-instruct**.
1. Nel pannello principale della sessione di chat, sotto la casella di input della chat, usare il pulsante allega (**&#128206;**) per caricare il file di immagine *mango.jpeg*, quindi aggiungere il testo `What desserts could I make with this fruit?` e inviare la richiesta.

    ![Screenshot del playground della chat con una richiesta basata sull'immagine.](../media/chat-playground-image.png)

1. Rivedere la risposta, che dovrebbe fornire indicazioni pertinenti per i dessert che è possibile realizzare utilizzando un mango.

## Creare un'applicazione client

Dopo aver distribuito il modello, è possibile usare la distribuzione in un'applicazione client.

> **Suggerimento**: è possibile scegliere di sviluppare la propria soluzione usando Python o Microsoft C#  *(presto disponibile)*. Seguire le istruzioni nella sezione appropriata per la lingua scelta.

### Preparare la configurazione dell'applicazione

1. Nel Portale Fonderia Azure AI visualizzare la pagina **Panoramica** per il progetto.
2. Nell'area **Dettagli di progetto** prendere nota della **stringa di connessione del progetto**. Questa stringa di connessione verrà usata per connettersi al progetto in un'applicazione client.
3. Aprire una nuova scheda del browser (mantenendo aperto il Portale Fonderia Azure AI nella scheda esistente). In una nuova scheda del browser, passare al [portale di Azure](https://portal.azure.com) su `https://portal.azure.com`, accedendo con le credenziali di Azure se richiesto.

    Chiudere eventuali notifiche di benvenuto per visualizzare la pagina iniziale del portale di Azure.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***PowerShell*** senza archiviazione nell'abbonamento.

    Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. È possibile ridimensionare o ingrandire questo riquadro per ottimizzare l'esperienza d'uso.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

5. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

1. Nel riquadro Cloud Shell immettere i comandi seguenti per clonare il repository GitHub contenente i file di codice per questo esercizio (digitare il comando o copiarlo negli Appunti e quindi fare clic con il pulsante destro del mouse nella riga di comando e incollarlo come testo normale):


    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

7. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:  

    **Python**

    ```
   cd mslearn-ai-vision/Labfiles/08-gen-ai-vision/python
    ```

    **C#**

    ```
   cd mslearn-ai-vision/Labfiles/08-gen-ai-vision/c-sharp
    ```

8. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per installare le librerie che si useranno, ovvero:

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
    ```

9. Immettere il comando seguente per modificare il file di configurazione fornito:

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    Il file viene aperto in un editor di codice.

10. Nel file di codice, sostituire il segnaposto **your_project_connection_string** con la stringa di connessione per il progetto (copiata dalla pagina **Panoramica** del Portale Fonderia Azure AI) e il segnaposto **your_model_deployment** con il nome assegnato alla distribuzione modello Phi-4-multimodal-instruct.
11. Dopo aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

### Scrivere il codice per connettersi al progetto e ottenere un client di chat per il modello

> **Suggerimento**: quando si aggiunge codice, assicurarsi di mantenere il rientro corretto.

1. Immettere il comando seguente per modificare il file di codice fornito:

    **Python**

    ```
   code chat-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

2. Nel file di codice prendere nota delle istruzioni esistenti aggiunte all'inizio del file per importare i namespaces (spazi dei nomi) SDK necessari. Quindi, trovare il commento **Aggiungi riferimenti** e aggiungere il codice seguente per fare riferimento ai namespaces (spazi dei nomi) nelle librerie installati in precedenza:

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
        ImageContentItem,
        ImageUrl,
   )
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using Azure.AI.Inference;
    ```

3. Nella funzione **principale**, sotto il commento **Ottieni impostazioni di configurazione**, notare che il codice carica i valori della stringa di connessione del progetto e del nome della distribuzione modello definiti nel file di configurazione.
4. Trovare il commento **Inizializza il client del progetto**, aggiungere il codice seguente per connettersi al progetto Fonderia Azure AI usando le credenziali di Azure con cui si è attualmente eseguito l'accesso:

    **Python**

    ```python
   # Initialize the project client
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
   // Initialize the project client
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

5. Trovare il commento **Ottieni un client di chat**, aggiungere il seguente codice per creare un oggetto client per chattare con il modello:

    **Python**

    ```python
   # Get a chat client
   chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```

### Scrivere codice per inviare una richiesta di immagine basata sull'URL

1. Si noti che il codice include un ciclo per consentire a un utente di immettere un prompt fino a quando non immette "esci". Quindi, nella sezione del ciclo, trovare il commento **Ottieni una risposta all'input immagine**, aggiungere il seguente codice per inviare una richiesta che includa la seguente immagine:

    ![Una foto di un mango.](../media/orange.jpeg)

    **Python**

    ```python
   # Get a response to image input
   image_url = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/orange.jpeg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(content=[
                TextContentItem(text=prompt),
                ImageContentItem(image_url=ImageUrl(url=data_url))
            ]),
        ]
   )
   print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imageUrl = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/orange.jpeg";
   ChatCompletionsOptions requestOptions = new ChatCompletionsOptions()
   {
        Messages = {
           new ChatRequestSystemMessage(system_message),
           new ChatRequestUserMessage([
                new ChatMessageTextContentItem(prompt),
                new ChatMessageImageContentItem(new Uri(imageUrl))
            ]),
        },
        Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. Usare il comando **CTRL+S** per salvare le modifiche al file di codice, senza chiuderlo.

3. Nel riquadro della riga di comando di Cloud Shell, sotto l'editor di codice, inserire il seguente comando per eseguire l'applicazione:

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. Quando richiesto, immettere la richiesta seguente:

    ```
   Suggest some recipes that include this fruit
    ```

5. Rivedere la risposta. Immettere quindi `quit` per uscire dal programma.

### Modificare il codice per caricare un file di immagine locale

1. Nell'editor di codice del codice dell'app, nella sezione del ciclo, individuare il codice aggiunto in precedenza sotto il commento **Ottieni una risposta all'input immagine**. Quindi, modificare il codice come indicato di seguito per caricare il file di immagine locale:

    ![Una foto di un frutto del drago.](../media/mystery-fruit.jpeg)

    **Python**

    ```python
   # Get a response to image input
   script_dir = Path(__file__).parent  # Get the directory of the script
   image_path = script_dir / 'mystery-fruit.jpeg'
   mime_type = "image/jpeg"

    # Read and encode the image file
    with open(image_path, "rb") as image_file:
        base64_encoded_data = base64.b64encode(image_file.read()).decode('utf-8')

    # Include the image file data in the prompt
    data_url = f"data:{mime_type};base64,{base64_encoded_data}"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(content=[
                TextContentItem(text=prompt),
                ImageContentItem(image_url=ImageUrl(url=data_url))
            ]),
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imagePath = "mystery-fruit.jpeg";
   string mimeType = "image/jpeg";
    
   // Read and encode the image file
   byte[] imageBytes = File.ReadAllBytes(imagePath);
   var binaryImage = new BinaryData(imageBytes);
    
   // Include the image file data in the prompt
   ChatCompletionsOptions requestOptions = new ChatCompletionsOptions()
   {
        Messages = {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage([
                new ChatMessageTextContentItem(prompt),
                new ChatMessageImageContentItem(bytes: binaryImage, mimeType: mimeType) 
            ]),
        },
        Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. Usare il comando **CTRL+S** per salvare le modifiche apportate al file di codice. È anche possibile chiudere l'editor di codice (**CTRL+Q**) se si desidera.

3. Nel riquadro della riga di comando di Cloud Shell, sotto l'editor di codice, inserire il seguente comando per eseguire l'applicazione:

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. Quando richiesto, immettere la richiesta seguente:

    ```
   What is this fruit? What recipes could I use it in?
    ```

5. Rivedere la risposta. Immettere quindi `quit` per uscire dal programma.

    > **Nota**: in questa semplice app non è stata implementata la logica per conservare la cronologia delle conversazioni, quindi il modello considererà ogni prompt come una nuova richiesta, senza il contesto del prompt precedente.

## Esplorare ulteriormente: (se il tempo lo consente)

Si è appreso come usare l'SDK di inferenza di Azure AI e un modello multimodale per implementare un'app di IA generativa in grado di rispondere alle richieste basate su immagini. Se il tempo a disposizione lo consente, ecco alcune idee per un'ulteriore esplorazione.

### Usare un modello multimodale diverso

È stato usato un modello *Phi-4-multimodal-instruct* per generare una risposta a una richiesta basata su immagini. Provare ora un modello OpenAI *gpt-4o*.

1. In Fonderia Azure AI distribuire un modello **gpt-4o** in un endpoint di inferenza del modello Azure AI (potrebbe essere necessario creare una nuova risorsa in un'area diversa).
1. Aggiornare il file di configurazione del codice per l'app (*.env* per Python, *appsettings.json* per C#) per specificare il nome del modello gpt-4o.
1. Eseguire l'app come in precedenza, usando le stesse richieste (è possibile ripristinare il codice che usa un'immagine basata su URL, se lo si desidera).

### Usare le API OpenAI

Il codice usato in questo esercizio è basato sull'SDK inferenza di Azure AI, che funziona con qualsiasi modello distribuito in un endpoint di inferenza del modello Azure AI. Quando si usa un modello OpenAI, in alternativa è possibile usare SDK di OpenAI.

Le istruzioni seguenti presuppongono che questo esercizio e l'attività aggiuntiva precedente siano stati completati per distribuire e testare un modello **gpt-4o**.

1. Installare (o aggiornare) i pacchetti necessari per l'app:

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects openai
    ```
    
    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Azure.AI.OpenAI --prerelease
    ```

1. Aggiornare i namespaces nel file di codice (rimuovendo i riferimenti *azure.ai-inference*):

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   import openai
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using OpenAI.Chat;
   using Azure.AI.OpenAI;
    ```

1. Modificare il codice per ottenere un client di chat:

    **Python**

    ```python
   # Get a chat client
   chat_client = project_client.inference.get_azure_openai_client(api_version="2024-10-21")
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatClient chatClient = projectClient.GetAzureOpenAIChatClient(model_deployment);
    ```

1. Modificare il codice per ottenere un completamento basato su un file di immagine locale

    **Python**

    ```python
   # Get a response to image input
   script_dir = Path(__file__).parent  # Get the directory of the script
   image_path = script_dir / 'mystery-fruit.jpeg'
   mime_type = "image/jpeg"

   # Read and encode the image file
   with open(image_path, "rb") as image_file:
        base64_encoded_data = base64.b64encode(image_file.read()).decode('utf-8')

   # Include the image file data in the prompt
   data_url = f"data:{mime_type};base64,{base64_encoded_data}"
   response = chat_client.chat.completions.create(
        model=model_deployment,
        messages=[
            { "role": "system", "content": system_message },
            { "role": "user", "content": [  
                { 
                    "type": "text", 
                    "text": prompt 
                },
                { 
                    "type": "image_url",
                    "image_url": {
                        "url": data_url
                    }
                }
            ] } 
        ]
   )
   completion = response.choices[0].message.content
   print(completion)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imagePath = "mystery-fruit.jpeg";
   string mimeType = "image/jpeg";
    
   // Read and encode the image file
   byte[] imageBytes = File.ReadAllBytes(imagePath);
   var binaryImage = new BinaryData(imageBytes);

   // Include the image file data in the prompt
   List<ChatMessage> messages =
   [
        new SystemChatMessage(system_message),
        new UserChatMessage(
            ChatMessageContentPart.CreateTextPart(prompt),
            ChatMessageContentPart.CreateImagePart(binaryImage, mimeType)),
   ];

   ChatCompletion completion = chatClient.CompleteChat(messages);
   Console.WriteLine(completion.Content[0].Text);
    ```

1. Salvare le modifiche ed eseguire l'app per testarla con le stesse richieste usate in precedenza.

## Eseguire la pulizia

Al termine dell'esplorazione di Fonderia Azure AI, è importante eliminare le risorse create durante l'esercizio per evitare costi di Azure non necessari.

1. Tornare alla scheda del browser che contiene il portale di Azure (o riaprire il [portale di Azure](https://portal.azure.com) su `https://portal.azure.com` in una nuova scheda del browser) e visualizzare il contenuto del gruppo di risorse in cui sono state distribuite le risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.
