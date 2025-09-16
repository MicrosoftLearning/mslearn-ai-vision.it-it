---
lab:
  title: Generare immagini con l'intelligenza artificiale
  description: Usare un modello DALL-E di OpenAI in Fonderia Azure AI per generare immagini.
---

# Generare immagini con l'intelligenza artificiale

In questo esercizio si usa il modello di IA generativa OpenAI DALL-E per generare immagini. Si usa anche OpenAI Python SDK per creare una semplice app per generare immagini in base alle richieste.

> **Nota**: Questo esercizio è basato sulla versione non definitiva del software SDK, che può essere soggetta a modifiche. Se necessario, abbiamo usato versioni specifiche dei pacchetti; che potrebbero non riflettere le versioni disponibili più recenti. È possibile che si verifichino alcuni comportamenti, avvisi o errori imprevisti.

Anche se questo esercizio è basato su OpenAI Python SDK, è possibile sviluppare applicazioni di chat basate su intelligenza artificiale usando più SDK specifici del linguaggio; tra cui:

* [Progetti OpenAI per Microsoft .NET](https://www.nuget.org/packages/OpenAI)
* [Progetti OpenAI per JavaScript](https://www.npmjs.com/package/openai)

Questo esercizio richiede circa **30** minuti.

## Aprire il portale di Azure AI Foundry

Per iniziare, accedere al Portale Fonderia Azure AI.

1. In un Web browser, aprire il [Portale Fonderia Azure AI](https://ai.azure.com) su `https://ai.azure.com` e accedere usando le credenziali di Azure. Chiudere tutti i riquadri dei suggerimenti o di avvio rapido che vengono aperti al primo accesso e, se necessario, usare il logo **Fonderia Azure AI** in alto a sinistra per passare alla home page, simile all'immagine seguente (chiudere il riquadro **Aiuto** nel caso sia aperto):

    ![Screenshot del portale di Azure AI Foundry.](./media/ai-foundry-home.png)

1. Esaminare le informazioni nella home page.

## Scegliere un modello per avviare un progetto

Un *progetto* Azure AI offre un'area di lavoro collaborativa per lo sviluppo dell'intelligenza artificiale. Per iniziare, scegliere un modello da usare e creare un progetto in cui usarlo.

> **Nota**: i progetti Fonderia AI possono essere basati su una risorsa *Fonderia Azure AI* , che fornisce l'accesso ai modelli di intelligenza artificiale (tra cui Azure OpenAI), a Servizi di Azure AI e ad altre risorse per lo sviluppo di agenti di intelligenza artificiale e soluzioni di chat. In alternativa, i progetti possono essere basati sulle risorse *Hub IA*, che includono connessioni alle risorse di Azure per l'archiviazione sicura, il calcolo e gli strumenti specializzati. I progetti basati su Fonderia Azure AI sono ideali per gli sviluppatori che vogliono gestire le risorse per lo sviluppo di agenti IA o di app di chat. I progetti basati su Hub IA sono più adatti per i team di sviluppo aziendale che lavorano su soluzioni di intelligenza artificiale complesse.

1. Nella home page, nella sezione **Esplora modelli e funzionalità**, cercare il modello `dall-e-3`, che verrà usato nel progetto.

1. Nei risultati della ricerca selezionare il modello **dall-e-3** e quindi nella parte superiore della pagina per il modello selezionare **Usa questo modello**.

1. Quando viene richiesto di creare un progetto, immettere un nome valido per il progetto ed espandere **Opzioni avanzate**.

1. Selezionare **Personalizza** e specificare le impostazioni seguenti per l'hub:
    - **Risorsa di Fonderia Azure AI**: *nome valido per la risorsa di Fonderia Azure AI*
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *creare o selezionare un gruppo di risorse*
    - **Area**: *selezionare una delle **opzioni consigliate di Fonderia AI***\*

    > \* Alcune risorse Azure AI sono limitate da quote di modelli regionali. In caso di superamento di un limite di quota più avanti nell'esercizio, potrebbe essere necessario creare un'altra risorsa in un'area diversa.

1. Selezionare **Crea** e attendere la creazione del progetto, inclusa la distribuzione del modello dall-e-3 selezionato.

    > Nota: A seconda della selezione del modello, è possibile che si ricevano richieste aggiuntive durante il processo di creazione del progetto. Accettare le condizioni e finalizzare la distribuzione.

1. Al termine della creazione del progetto, il modello verrà visualizzato nella pagina **Modelli ed endpoint**.

## Testare il modello nel playground

Prima di creare un'applicazione client, testare il modello DALL-E nel playground.

1. Selezionare **Playground**e quindi **Playground immagini**.

1. Assicurarsi che sia selezionata la distribuzione modello DALL-E. Nella casella nella parte inferiore della pagina immettere quindi una richiesta, ad esempio `Create an image of an robot eating spaghetti`, e selezionare **Genera**.

1. Esaminare l'immagine risultante nel playground:

    ![Screenshot del playground delle immagini con un'immagine generata.](../media/images-playground.png)

1. Immettere una richiesta di completamento, ad esempio `Show the robot in a restaurant` e rivedere l'immagine risultante.

1. Continuare a eseguire il test con nuove richieste per affinare l'immagine fino a quando non è soddisfacente. 

1. Selezionare il pulsante **\</\> Visualizza codice** e assicurarsi di trovarsi nella scheda **Autenticazione di Entra ID**. Registrare quindi le informazioni seguenti per usarle più avanti nell'esercizio. Si noti che i valori sono esempi. Assicurarsi di registrare le informazioni dalla distribuzione.

    * Endpoint OpenAI: *https://dall-e-aus-resource.cognitiveservices.azure.com/*
    * Versione dell'API OpenAI: *2024-04-01-preview*
    * Nome distribuzione (nome modello): *dall-e-3*

## Creare un'applicazione client

Il modello sembra funzionare nel playground. A questo punto è possibile usare l'SDK di OpenAI per usarlo in un'applicazione client.

### Preparare la configurazione dell'applicazione

1. Aprire una nuova scheda del browser (mantenendo aperto il Portale Fonderia Azure AI nella scheda esistente). In una nuova scheda del browser, passare al [portale di Azure](https://portal.azure.com) su `https://portal.azure.com`, accedendo con le credenziali di Azure se richiesto.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***PowerShell***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

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
   cd mslearn-ai-vision/Labfiles/dalle-client/python
    ```

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per installare le librerie che si useranno, ovvero:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity openai requests
    ```

1. Immettere il comando seguente per modificare il file di configurazione fornito:

    ```
   code .env
    ```

    Il file viene aperto in un editor di codice.

1. Sostituire i segnaposto **your_endpoint**, **your_model_deployment** e **your_api_version** con i valori registrati dal **Playground immagini**.

1. Dopo aver sostituito i segnaposto, usare il comando **CTRL+S** per salvare le modifiche e quindi usare il comando **CTRL+Q** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

### Scrivere codice per connettersi al progetto e chattare con il modello

> **Suggerimento**: quando si aggiunge codice, assicurarsi di mantenere il rientro corretto.

1. Immettere il comando seguente per modificare il file di codice fornito:

    ```
   code dalle-client.py
    ```

1. Nel file di codice prendere nota delle istruzioni esistenti aggiunte all'inizio del file per importare i namespaces (spazi dei nomi) SDK necessari. Quindi, sotto il commento **Aggiungi riferimenti**, aggiungere il codice seguente per fare riferimento ai namespaces (spazi dei nomi) installati in precedenza:

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential, get_bearer_token_provider
   from openai import AzureOpenAI
   import requests
    ```

1. Nella funzione **main** sotto il commento **Get configuration settings** si noti che il codice carica i valori dell'endpoint, della versione dell'API e del nome della distribuzione modello definiti nel file di configurazione.

1. Sotto il commento **Initialize the client** aggiungere il codice seguente per connettersi al modello usando le credenziali di Azure usate attualmente per l'accesso:

    ```python
   # Initialize the client
   token_provider = get_bearer_token_provider(
       DefaultAzureCredential(exclude_environment_credential=True,
           exclude_managed_identity_credential=True), 
       "https://cognitiveservices.azure.com/.default"
   )
    
   client = AzureOpenAI(
       api_version=api_version,
       azure_endpoint=endpoint,
       azure_ad_token_provider=token_provider
   )
    ```

1. Si noti che il codice include un ciclo per consentire a un utente di immettere un prompt fino a quando non immette "esci". Quindi, nella sezione del ciclo, sotto il commento **Genera un'immagine**, aggiungere il codice seguente per inviare il prompt e recuperare l'URL per l'immagine generata dal modello:

    **Python**

    ```python
   # Generate an image
   result = client.images.generate(
        model=model_deployment,
        prompt=input_text,
        n=1
    )

   json_response = json.loads(result.model_dump_json())
   image_url = json_response["data"][0]["url"] 
    ```

1. Si noti che il codice nel resto della funzione **principale** passa l'URL dell'immagine e un nome di file a una funzione fornita, che scarica l'immagine generata e la salva come file PNG.

1. Usare il comando **CTRL+S** per salvare le modifiche apportate al file di codice e quindi usare il comando **CTRL+Q** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.

### Eseguire l'applicazione client

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per accedere ad Azure.

    ```
   az login
    ```

    **<font color="red">È necessario accedere ad Azure, anche se la sessione di Cloud Shell è già autenticata.</font>**

    > **Nota**: nella maggior parte degli scenari, il semplice uso di *az login* sarà sufficiente. Tuttavia, in caso di sottoscrizioni in più tenant, potrebbe essere necessario specificare il tenant usando il parametro *--tenant*. Per dettagli, visualizzare [Accedere ad Azure in modo interattivo usando l'interfaccia della riga di comando di Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).

1. Quando richiesto, seguire le istruzioni per aprire la pagina di accesso in una nuova scheda e immettere il codice di autenticazione fornito e le credenziali di Azure. Completare quindi il processo di accesso nella riga di comando, selezionando la sottoscrizione contenente l'hub di Fonderia Azure AI, se richiesto.

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per eseguire l'app:

    ```
   python dalle-client.py
    ```

1. Quando richiesto, immettere una richiesta per un'immagine, ad esempio `Create an image of a robot eating pizza`. Dopo pochi minuti, l'app deve confermare che l'immagine è stata salvata.

1. Provare a immettere altre richieste. Al termine, immettere `quit` per uscire dal programma.

    > **Nota**: in questa semplice app non è stata implementata la logica per conservare la cronologia delle conversazioni, quindi il modello considererà ogni prompt come una nuova richiesta, senza il contesto del prompt precedente.

1. Per scaricare e visualizzare le immagini generate dall'app, usare il comando di **download** del Cloud Shell, specificando il file .png generato:

    ```
   download ./images/image_1.png
    ```

    Il comando download crea un collegamento popup in basso a destra del browser, che è possibile selezionare per scaricare e aprire il file.

## Riepilogo

In questo esercizio è stato usato Fonderia Azure AI e l'SDK di Azure OpenAI per creare un'applicazione client che usa un modello DALL-E per generare immagini.

## Eseguire la pulizia

Al termine dell'esplorazione di DALL-E, è necessario eliminare le risorse create in questo esercizio per evitare di incorrere in costi di Azure non necessari.

1. Tornare alla scheda del browser che contiene il portale di Azure (o riaprire il [portale di Azure](https://portal.azure.com) su `https://portal.azure.com` in una nuova scheda del browser) e visualizzare il contenuto del gruppo di risorse in cui sono state distribuite le risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.
