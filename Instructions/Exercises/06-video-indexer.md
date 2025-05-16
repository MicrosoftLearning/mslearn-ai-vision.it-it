---
lab:
  title: Analizzare video con Video Indexer di Azure
  module: Module 8 - Getting Started with Azure AI Vision
---

# Analizzare video con Video Indexer di Azure

Gran parte dei dati creati e utilizzati oggi è in formato video. **Video Indexer di Azure AI** è un servizio basato su intelligenza artificiale che è possibile usare per indicizzare i video ed estrarre informazioni dettagliate.

> **Nota**: Dal 21 giugno 2022 le funzionalità di Servizi di Azure AI che restituiscono informazioni personali dell'utente finale sono limitate ai clienti a cui è stato concesso l'[accesso limitato](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Senza previa approvazione dell'accesso limitato, il riconoscimento di persone e di celebrità con Video Indexer di Azure in questo lab non è disponibile. Per altre informazioni sulle modifiche apportate da Microsoft e sul relativo motivo, vedere [Misure di sicurezza e investimenti responsabili nell'intelligenza artificiale per il riconoscimento facciale](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

## Clonare il repository per questo corso

Se di recente è stato clonato il repository di codice **mslearn-ai-vision** nell'ambiente in cui si sta lavorando a questo lab, aprirlo in Visual Studio Code. In caso contrario, seguire questa procedura per clonarlo ora.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-ai-vision` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Caricare un video in Video Indexer

Prima di tutto, è necessario accedere al portale di Video Indexer e caricare un video.

> **Suggerimento**: Se il caricamento della pagina di Video Indexer è lento nell'ambiente lab ospitato, usare il browser installato in locale. È possibile tornare alla macchina virtuale ospitata per le attività successive.

1. Nel browser aprire il portale di Video Indexer all'indirizzo `https://www.videoindexer.ai`.
2. Se si dispone già di un account esistente di Video Indexer, accedere. In caso contrario, iscriversi per ricevere un account gratuito e accedere usando l'account Microsoft (o qualsiasi altro tipo di account valido). In caso di difficoltà con l'accesso, provare ad aprire una sessione privata del browser.
3. In una nuova scheda, scaricare il video sull'intelligenza artificiale responsabile visitando `https://aka.ms/responsible-ai-video`. Salvare il file.
4. In Video Indexer, selezionare l'opzione **Carica**. Selezionare quindi l'opzione per **Esplorare i file**, selezionare il video scaricato e fare clic su **Aggiungi**. Cambiare il nome predefinito in **Responsible AI**, esaminare le impostazioni predefinite, selezionare la casella di controllo per verificare la conformità ai criteri di Microsoft per il riconoscimento facciale, quindi caricare il file.
5. Dopo aver caricato il file, attendere alcuni minuti mentre viene indicizzato automaticamente da Video Indexer.

> **Nota**: Questo video viene usato nell'esercizio per esplorare le funzionalità di Video Indexer, ma è consigliabile guardarlo per intero al termine dell'esercizio perché contiene informazioni e istruzioni utili per lo sviluppo responsabile di applicazioni basate su intelligenza artificiale. 

## Esaminare le informazioni dettagliate estratte dal video

Il processo di indicizzazione estrae informazioni dettagliate dal video che è possibile visualizzare nel portale.

1. Dopo l'indicizzazione del video, selezionarlo nel portale di Video Indexer per visualizzarlo. Il lettore video verrà visualizzato insieme a un riquadro che mostra le informazioni dettagliate estratte dal video.

    > **Nota**: A causa dei criteri di accesso limitato per proteggere le identità individuali, è possibile che i nomi non vengano visualizzati quando si indicizza il video.

![Video Indexer con un lettore video e un riquadro delle informazioni dettagliate](../media/video-indexer-insights.png)

2. Durante la riproduzione del video, selezionare la scheda **Sequenza temporale** per visualizzare una trascrizione dell'audio.

![Video Indexer con un lettore video e un riquadro Sequenza temporale che mostra la trascrizione del video.](../media/video-indexer-transcript.png)

3. In alto a destra nel portale selezionare il simbolo **Visualizza** (simile a &#128455;), quindi nell'elenco di informazioni dettagliate, oltre a **Trascrizione**, selezionare **OCR** e **Parlanti**.

![Menu di visualizzazione di Video Indexer con le opzioni Trascrizione, OCR e Altoparlanti selezionate](../media/video-indexer-view-menu.png)

4. Notare che ora il riquadro **Sequenza temporale** include quanto segue:
    - Trascrizione della narrazione audio.
    - Testo visibile nel video.
    - Indicazioni dei parlanti che compaiono nel video. Alcune persone note vengono riconosciute automaticamente per nome, mentre altre sono indicate dal numero (ad esempio *Speaker #1*).
5. Tornare nel riquadro **Informazioni dettagliate** e visualizzare le informazioni dettagliate che contiene. che includono:
    - Singole persone che compaiono nel video.
    - Argomenti trattati nel video.
    - Etichette per gli oggetti visualizzati nel video.
    - Entità denominate, ad esempio persone e marchi visualizzati nel video.
    - Scene chiave.
6. Con il riquadro **Informazioni dettagliate** visualizzato, selezionare di nuovo il simbolo **Visualizza**, quindi nell'elenco di informazioni dettagliate aggiungere **Parole chiave** e **Sentiment** al riquadro.

    Le informazioni dettagliate consentono di determinare i temi principali del video. Ad esempio, gli **argomenti** di questo video mostrano chiaramente che si tratta di tecnologia, responsabilità sociale ed etica.

## Cercare informazioni dettagliate

È possibile usare Video Indexer per cercare informazioni dettagliate nel video.

1. Nella casella **Cerca** del riquadro **Informazioni dettagliate** immettere *Bee* (Ape). Può essere necessario scorrere verso il basso nel riquadro per visualizzare i risultati per tutti i tipi di informazioni dettagliate.
2. Si noti che viene trovata un'*etichetta* corrispondente, con la relativa posizione nel video indicata al di sotto.
3. Selezionare l'inizio della sezione in cui è indicata la presenza di un'ape e visualizzare il video in corrispondenza di quel punto (può essere necessario mettere in pausa il video e selezionare con attenzione: l'ape viene visualizzata solo brevemente).
4. Cancellare la casella **Cerca** per visualizzare tutte le informazioni dettagliate per il video.

![Risultati della ricerca di Video Indexer per Bee](../media/video-indexer-search.png)

## Usare i widget di Video Indexer

Il portale di Video Indexer è un'interfaccia utile per gestire i progetti di indicizzazione video. È tuttavia possibile che in alcuni casi si voglia rendere il video e le relative informazioni dettagliate disponibili per persone che non hanno accesso all'account di Video Indexer. Video Indexer fornisce widget che è possibile incorporare in una pagina Web per tale scopo.

1. In Visual Studio Code, nella cartella **Labfiles/06-video-indexer**, aprire **analyze-video.html**. Si tratta di una pagina HTML di base a cui si aggiungeranno i widget **Player** e**Informazioni dettagliate** di Video Indexer. Si noti il riferimento allo script **vb.widgets.mediator.js** nell'intestazione: questo script consente l'interazione di più widget di Video Indexer nella pagina.
2. Nel portale di Video Indexer tornare nella pagina **File multimediali** e aprire il video **Intelligenza artificiale responsabile**.
3. Sotto il lettore video selezionare **&lt;/&gt; Incorpora** per visualizzare il codice iframe HTML per incorporare i widget.
4. Nella finestra **Incorpora e condividi** selezionare il widget **Player**, impostare le dimensioni del video su 560 x 315 e quindi copiare il codice di incorporamento negli Appunti.
5. Nel Visual Studio Code incollare il codice copiato nel file **analyze-video.html** sotto il commento **&lt;-- Player widget goes here -- &gt;**.
6. Tornare nella finestra di dialogo **Incorpora e condividi** e selezionare il widget **Insights**,quindi copiare il codice di incorporamento negli Appunti. Chiudere quindi la finestra di dialogo **Incorpora e condividi**, tornare in Visual Studio Code e incollare il codice copiato sotto il commento **&lt;-- Insights widget goes here -- &gt;**.
7. Salvare il file. Nel riquadro **Esplora risorse** fare clic con il pulsante destro del mouse su **analyze-video.html** e scegliere **Visualizza in Esplora file**.
8. In Esplora file aprire **analyze-video.html** nel browser per visualizzare la pagina Web.
9. Sperimentare con i widget, usando il widget **Insights** per cercare informazioni dettagliate e individuarle nel video.

![Widget di Video Indexer in una pagina Web](../media/video-indexer-widgets.png)

## Usare l'API REST di Video Indexer

Video Indexer include un'API REST che è possibile usare per caricare e gestire i video nell'account.

### Ottenere i dettagli dell'API

Per usare l'API di Video Indexer, sono necessarie alcune informazioni per autenticare le richieste:

1. Nel portale di Video Indexer espandere il riquadro sinistro e scegliere la pagina **Impostazioni account**.
2. Prendere nota dell'**ID account** in questa pagina, perché sarà necessario più avanti.
3. Aprire una nuova scheda del browser e passare al portale per sviluppatori di Video Indexer all'indirizzo `https://api-portal.videoindexer.ai`, accedendo con le credenziali dell'account di Video Indexer.
4. Nella pagina **Profilo** visualizzare le **Sottoscrizioni** associate al profilo.
5. Nella pagina con le proprie sottoscrizioni si noti che sono state assegnate due chiavi (primaria e secondaria) per ognuna. Selezionare quindi **Mostra** per visualizzare una delle chiavi. Questa chiave sarà necessaria tra poco.

### Usare l'API REST

Dopo aver ottenuto l'ID account e una chiave API, è possibile usare l'API REST per lavorare con i video nell'account. In questa procedura si userà uno script di PowerShell per effettuare chiamate REST, ma gli stessi principi si applicano alle utilità HTTP, ad esempio cURL o Postman, o a qualsiasi linguaggio di programmazione in grado di inviare e ricevere codice JSON su HTTP.

Tutte le interazioni con l'API REST di Video Indexer seguono lo stesso modello:

- Viene effettuata una richiesta iniziale al metodo **AccessToken** con la chiave API nell'intestazione per ottenere un token di accesso.
- Le richieste successive usano il token di accesso per l'autenticazione quando si chiamano i metodi REST per lavorare con i video.

1. In Visual Studio Code, nella cartella **Labfiles/06-video-indexer**, aprire **get-videos.ps1**.
2. Nello script di PowerShell sostituire i segnaposto **YOUR_ACCOUNT_ID** e **YOUR_API_KEY** con i valori dell'ID account e della chiave API identificati in precedenza.
3. Si noti che il valore di *location* per un account gratuito è "trial". Se è stato creato un account di Video Indexer senza restrizioni (con una risorsa di Azure associata), è possibile cambiare questa località impostando l'area di Azure in cui è stato effettuato il provisioning della risorsa di Azure, ad esempio "eastus".
4. Esaminare il codice dello script, notando che richiama due metodi REST: uno per ottenere un token di accesso e un altro per elencare i video presenti nell'account.
5. Salvare le modifiche e quindi in alto a destra nel riquadro di script usare il pulsante **&#9655;** per eseguire lo script.
6. Visualizzare la risposta JSON dal servizio REST, che dovrà contenere i dettagli del video **Responsible AI** indicizzato in precedenza.

## Ulteriori informazioni

Il riconoscimento di persone e di celebrità è ancora disponibile, ma come da [Standard di IA responsabile](https://aka.ms/aah91ff), queste funzionalità sono soggette ai criteri di Accesso limitato. Queste funzionalità includono l'identificazione facciale e il riconoscimento delle celebrità. Per altre informazioni e per richiedere l'accesso, vedere [Accesso limitato per Servizi di Azure AI](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access).

Per altre informazioni su **Video Indexer**, vedere la [documentazione di Video Indexer](https://learn.microsoft.com/azure/azure-video-indexer/).
