---
lab:
  title: Sviluppare un'app di chat abilitata per la visione
  description: Usare Fonderia Azure AI per creare un'app di IA generativa che supporta input di immagini.
---

# Sviluppare un'app di chat abilitata per la visione

In questo esercizio, verrà usato il modello di IA generativa *Phi-4-multimodal-instruct* per generare risposte a richieste che includono immagini. Verrà sviluppata un'applicazione che fornisce assistenza AI per i prodotti freschi in un negozio di alimentari usando Fonderia Azure AI e il servizio dell'inferenza del modello di Azure per intelligenza artificiale.

> **Nota**: Questo esercizio è basato sulla versione non definitiva del software SDK, che può essere soggetta a modifiche. Se necessario, abbiamo usato versioni specifiche dei pacchetti; che potrebbero non riflettere le versioni disponibili più recenti. È possibile che si verifichino alcuni comportamenti, avvisi o errori imprevisti.

Anche se questo esercizio è basato su Python SDK per Fonderia Azure AI, è possibile sviluppare applicazioni di chat basate su intelligenza artificiale usando più SDK specifici del linguaggio; tra cui:

- [Progetti di Azure AI per Python](https://pypi.org/project/azure-ai-projects)
- [Progetti di Azure AI per Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Projects)
- [Progetti di Azure AI per JavaScript](https://www.npmjs.com/package/@azure/ai-projects)

Questo esercizio richiede circa **30** minuti.

## Aprire il portale di Azure AI Foundry

Per iniziare, accedere al Portale Fonderia Azure AI.

1. In un Web browser, aprire il [Portale Fonderia Azure AI](https://ai.azure.com) su `https://ai.azure.com` e accedere usando le credenziali di Azure. Chiudere tutti i riquadri dei suggerimenti o di avvio rapido che vengono aperti al primo accesso e, se necessario, usare il logo **Fonderia Azure AI** in alto a sinistra per passare alla home page, simile all'immagine seguente (chiudere il riquadro **Aiuto** nel caso sia aperto):

    ![Screenshot del portale di Azure AI Foundry.](./media/ai-foundry-home.png)

1. Esaminare le informazioni nella home page.

## Scegliere un modello per avviare un progetto

Un *progetto* Azure AI offre un'area di lavoro collaborativa per lo sviluppo dell'intelligenza artificiale. Per iniziare, scegliere un modello da usare e creare un progetto in cui usarlo.

> **Nota**: i progetti Fonderia AI possono essere basati su una risorsa *Fonderia Azure AI* , che fornisce l'accesso ai modelli di intelligenza artificiale (tra cui Azure OpenAI), a Servizi di Azure AI e ad altre risorse per lo sviluppo di agenti di intelligenza artificiale e soluzioni di chat. In alternativa, i progetti possono essere basati sulle risorse *Hub IA*, che includono connessioni alle risorse di Azure per l'archiviazione sicura, il calcolo e gli strumenti specializzati. I progetti basati su Fonderia Azure AI sono ideali per gli sviluppatori che vogliono gestire le risorse per lo sviluppo di agenti IA o di app di chat. I progetti basati su Hub IA sono più adatti per i team di sviluppo aziendale che lavorano su soluzioni di intelligenza artificiale complesse.

1. Nella home page, nella sezione **Esplora modelli e funzionalità**, cercare il modello `Phi-4-multimodal-instruct`, che verrà usato nel progetto.

1. Nei risultati della ricerca selezionare il modello **Phi-4-multimodal-instruct** per visualizzarne i dettagli e quindi nella parte superiore della pagina per il modello selezionare **Usa questo modello**.

1. Quando viene richiesto di creare un progetto, immettere un nome valido per il progetto ed espandere **Opzioni avanzate**.

1. Selezionare **Personalizza** e specificare le impostazioni seguenti per l'hub:
    - **Risorsa di Fonderia Azure AI**: *nome valido per la risorsa di Fonderia Azure AI*
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *creare o selezionare un gruppo di risorse*
    - **Area**: *selezionare una delle **opzioni consigliate di Fonderia AI***\*

    > \* Alcune risorse Azure AI sono limitate da quote di modelli regionali. In caso di superamento di un limite di quota più avanti nell'esercizio, potrebbe essere necessario creare un'altra risorsa in un'area diversa.

1. Selezionare **Crea** e attendere la creazione del progetto, inclusa la distribuzione del modello Phi-4-multimodal-instruct selezionato.

    > Nota: A seconda della selezione del modello, è possibile che si ricevano richieste aggiuntive durante il processo di creazione del progetto. Accettare le condizioni e finalizzare la distribuzione.

1. Al termine della creazione del progetto, il modello verrà visualizzato nella pagina **Modelli ed endpoint**:

    ![Screenshot della pagina di distribuzione del modello.](./media/ai-foundry-model-deployment.png)

## Testare il modello nel playground

È ora possibile testare la distribuzione del modello multimodale con una richiesta basata su immagini nel playground della chat.

1. Selezionare **Apri nel playground** nella pagina di distribuzione del modello.

1. In una nuova scheda del browser, scaricare [mango.jpeg](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg) da `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg` e salvarlo in una cartella nel file system locale.

1. Nella pagina playground della chat, nel riquadro **Configurazione**, assicurarsi che sia selezionato il modello di distribuzione **Phi-4-multimodal-instruct**.

1. Nel pannello principale della sessione di chat, sotto la casella di input della chat, usare il pulsante allega (**&#128206;**) per caricare il file di immagine *mango.jpeg*, quindi aggiungere il testo `What desserts could I make with this fruit?` e inviare la richiesta.

    ![Screenshot della pagina del playground della chat.](../media/chat-playground-image.png)

1. Rivedere la risposta, che dovrebbe fornire indicazioni pertinenti per i dessert che è possibile realizzare utilizzando un mango.

## Creare un'applicazione client

Dopo aver distribuito il modello, è possibile usare la distribuzione in un'applicazione client.

### Preparare la configurazione dell'applicazione

1. Nel Portale Fonderia Azure AI visualizzare la pagina **Panoramica** per il progetto.

1. Nell'area **Endpoint e chiavi** assicurarsi che sia selezionata la libreria di **Fonderia Azure AI** e notare l'**Endpoint del progetto di Fonderia Azure AI**. Questa stringa di connessione verrà usata per connettersi al progetto in un'applicazione client.

1. Aprire una nuova scheda del browser (mantenendo aperto il Portale Fonderia Azure AI nella scheda esistente). In una nuova scheda del browser, passare al [portale di Azure](https://portal.azure.com) su `https://portal.azure.com`, accedendo con le credenziali di Azure se richiesto.

    Chiudere eventuali notifiche di benvenuto per visualizzare la pagina iniziale del portale di Azure.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***PowerShell*** senza archiviazione nell'abbonamento.

    Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. È possibile ridimensionare o ingrandire questo riquadro per ottimizzare l'esperienza d'uso.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

    > **Nota**: Se il portale chiede di selezionare una risorsa di archiviazione per salvare in modo permanente i file, scegliere **Nessun account di archiviazione richiesto**, selezionare la sottoscrizione in uso e premere **Applica**.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

1. Nel riquadro Cloud Shell immettere i comandi seguenti per clonare il repository GitHub contenente i file di codice per questo esercizio (digitare il comando o copiarlo negli Appunti e quindi fare clic con il pulsante destro del mouse nella riga di comando e incollarlo come testo normale):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:  

    ```
   cd mslearn-ai-vision/Labfiles/gen-ai-vision/python
    ```

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per installare le librerie che si useranno, ovvero:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```

1. Immettere il comando seguente per modificare il file di configurazione fornito:

    ```
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Nel file di codice sostituire il segnaposto **your_project_endpoint** con l'endpoint del progetto di Fonderia (copiato dalla pagina **Panoramica** del progetto nel Portale Fonderia Azure AI) e il segnaposto **your_model_deployment** con il nome assegnato alla distribuzione del modello Phi-4-multimodal-instruct.

1. Dopo aver sostituito i segnaposto con l'editor di codice, usare il comando **CTRL+S** o **Fare clic con il pulsante destro del mouse > Salva** per salvare le modifiche e quindi usare il comando **CTRL+Q** o **Fare clic con il pulsante destro del mouse > Esci** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

### Scrivere il codice per connettersi al progetto e ottenere un client di chat per il modello

> **Suggerimento**: quando si aggiunge codice, assicurarsi di mantenere il rientro corretto.

1. Immettere il comando seguente per modificare il file di codice fornito:

    ```
   code chat-app.py
    ```

1. Nel file di codice prendere nota delle istruzioni esistenti aggiunte all'inizio del file per importare i namespaces (spazi dei nomi) SDK necessari. Quindi, trovare il commento **Aggiungi riferimenti** e aggiungere il codice seguente per fare riferimento ai namespaces (spazi dei nomi) nelle librerie installati in precedenza:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from openai import AzureOpenAI
    ```

1. Nella funzione **principale**, sotto il commento **Ottieni impostazioni di configurazione**, notare che il codice carica i valori della stringa di connessione del progetto e del nome della distribuzione modello definiti nel file di configurazione.
1. Nella funzione **principale**, sotto il commento **Ottieni impostazioni di configurazione**, notare che il codice carica i valori della stringa di connessione del progetto e del nome della distribuzione modello definiti nel file di configurazione.
1. Trovare il commento **Initialize the project client** e aggiungere il codice seguente per connettersi al progetto di Fonderia Azure AI:

    > **Suggerimento**: prestare attenzione a mantenere il livello di rientro corretto per il codice.

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,
                exclude_managed_identity_credential=True
            ),
            endpoint=project_endpoint,
        )
    ```

1. Trovare il commento **Ottieni un client di chat** e aggiungere il codice seguente per creare un oggetto client per la chat con un modello:

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

### Scrivere codice per inviare una richiesta di immagine basata sull'URL

1. Si noti che il codice include un ciclo per consentire a un utente di immettere un prompt fino a quando non immette "esci". Quindi, nella sezione del ciclo, trovare il commento **Ottieni una risposta all'input immagine**, aggiungere il seguente codice per inviare una richiesta che includa la seguente immagine:

    ![Una foto di un mango.](../media/orange.jpeg)

    ```python
   # Get a response to image input
   image_url = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/orange.jpeg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = openai_client.chat.completions.create(
        model=model_deployment,
        messages=[
            {"role": "system", "content": system_message},
            { "role": "user", "content": [  
                { "type": "text", "text": prompt},
                { "type": "image_url", "image_url": {"url": data_url}}
            ] } 
        ]
   )
   print(response.choices[0].message.content)
    ```

1. Usare il comando **CTRL+S** per salvare le modifiche al file di codice, senza chiuderlo.

## Accedere in Azure ed eseguire l'app

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per accedere ad Azure.

    ```
   az login
    ```

    **<font color="red">È necessario accedere ad Azure, anche se la sessione di Cloud Shell è già autenticata.</font>**

    > **Nota**: nella maggior parte degli scenari, il semplice uso di *az login* sarà sufficiente. Tuttavia, in caso di sottoscrizioni in più tenant, potrebbe essere necessario specificare il tenant usando il parametro *--tenant*. Per dettagli, visualizzare [Accedere ad Azure in modo interattivo usando l'interfaccia della riga di comando di Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Quando richiesto, seguire le istruzioni per aprire la pagina di accesso in una nuova scheda e immettere il codice di autenticazione fornito e le credenziali di Azure. Completare quindi il processo di accesso nella riga di comando, selezionando la sottoscrizione contenente l'hub di Fonderia Azure AI, se richiesto.

1. Dopo aver eseguito l'accesso, immettere il comando seguente per eseguire l'applicazione:

    ```
   python chat-app.py
    ```

1. Quando richiesto, immettere la richiesta seguente:

    ```
   Suggest some recipes that include this fruit
    ```

1. Esaminare la risposta. Immettere quindi `quit` per uscire dal programma.

### Modificare il codice per caricare un file di immagine locale

1. Nell'editor di codice del codice dell'app, nella sezione del ciclo, individuare il codice aggiunto in precedenza sotto il commento **Ottieni una risposta all'input immagine**. Quindi, modificare il codice come indicato di seguito per caricare il file di immagine locale:

    ![Una foto di un frutto del drago.](../media/mystery-fruit.jpeg)

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
   response = openai_client.chat.completions.create(
            model=model_deployment,
            messages=[
                {"role": "system", "content": system_message},
                { "role": "user", "content": [  
                    { "type": "text", "text": prompt},
                    { "type": "image_url", "image_url": {"url": data_url}}
                ] } 
            ]
   )
   print(response.choices[0].message.content)
    ```

1. Usare il comando **CTRL+S** per salvare le modifiche apportate al file di codice. È anche possibile chiudere l'editor di codice (**CTRL+Q**) se si desidera.

1. Nel riquadro della riga di comando di Cloud Shell, sotto l'editor di codice, inserire il seguente comando per eseguire l'applicazione:

    ```
   python chat-app.py
    ```

1. Quando richiesto, immettere la richiesta seguente:

    ```
   What is this fruit? What recipes could I use it in?
    ```

15. Esaminare la risposta. Immettere quindi `quit` per uscire dal programma.

    > **Nota**: in questa semplice app non è stata implementata la logica per conservare la cronologia delle conversazioni, quindi il modello considererà ogni prompt come una nuova richiesta, senza il contesto del prompt precedente.

## Eseguire la pulizia

Al termine dell'esplorazione del portale Azure AI Foundry, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari.

1. Aprire il [portale di Azure](https://portal.azure.com) e visualizzare il contenuto del gruppo di risorse in cui sono state distribuite le risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.
