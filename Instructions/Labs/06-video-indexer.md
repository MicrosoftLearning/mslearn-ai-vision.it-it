---
lab:
  title: Analizzare video
  description: Usare Video Indexer di Azure AI per analizzare un video.
---

# Analizzare video

Gran parte dei dati creati e utilizzati oggi è in formato video. **Video Indexer di Azure AI** è un servizio basato su intelligenza artificiale che è possibile usare per indicizzare i video ed estrarre informazioni dettagliate.

> **Nota**: Dal 21 giugno 2022 le funzionalità di Servizi di Azure AI che restituiscono informazioni personali dell'utente finale sono limitate ai clienti a cui è stato concesso l'[accesso limitato](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Senza previa approvazione dell'accesso limitato, il riconoscimento di persone e di celebrità con Video Indexer di Azure in questo lab non è disponibile. Per altre informazioni sulle modifiche apportate da Microsoft e sul relativo motivo, vedere [Misure di sicurezza e investimenti responsabili nell'intelligenza artificiale per il riconoscimento facciale](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/).

## Caricare un video in Video Indexer

Prima di tutto, è necessario accedere al portale di Video Indexer e caricare un video.

1. Nel browser aprire il [portale di Video Indexer](https://www.videoindexer.ai) all'indirizzo `https://www.videoindexer.ai`.
1. Se si dispone già di un account esistente di Video Indexer, accedere. In caso contrario, iscriversi per ricevere un account gratuito e accedere usando l'account Microsoft (o qualsiasi altro tipo di account valido). In caso di difficoltà con l'accesso, provare ad aprire una sessione privata del browser.

    > Nota: Se è la prima volta che si accede, potrebbe essere visualizzato una maschera popup che chiede di verificare come si userà il servizio. 

1. In una nuova scheda, scaricare il video sull'intelligenza artificiale responsabile visitando `https://aka.ms/responsible-ai-video`. Salvare il file.
1. In Video Indexer, selezionare l'opzione **Carica**. Selezionare quindi l'opzione per **Esplorare i file**, selezionare il video scaricato e fare clic su **Aggiungi**. Modificare il testo nel campo **Nome file** in **Intelligenza artificiale responsabile**. Selezionare **Rivedi e carica**, esaminare la panoramica di riepilogo, selezionare la casella di controllo per verificare la conformità ai criteri di Microsoft per il riconoscimento facciale e caricare il file.
1. Dopo aver caricato il file, attendere alcuni minuti mentre viene indicizzato automaticamente da Video Indexer.

> **Nota**: Questo video viene usato nell'esercizio per esplorare le funzionalità di Video Indexer, ma è consigliabile guardarlo per intero al termine dell'esercizio perché contiene informazioni e istruzioni utili per lo sviluppo responsabile di applicazioni basate su intelligenza artificiale. 

## Esaminare le informazioni dettagliate estratte dal video

Il processo di indicizzazione estrae informazioni dettagliate dal video che è possibile visualizzare nel portale.

1. Dopo l'indicizzazione del video, selezionarlo nel portale di Video Indexer per visualizzarlo. Il lettore video verrà visualizzato insieme a un riquadro che mostra le informazioni dettagliate estratte dal video.

    > **Nota**: A causa dei criteri di accesso limitato per proteggere le identità individuali, è possibile che i nomi non vengano visualizzati quando si indicizza il video.

    ![Video Indexer con un lettore video e un riquadro delle informazioni dettagliate](../media/video-indexer-insights.png)

1. Durante la riproduzione del video, selezionare la scheda **Sequenza temporale** per visualizzare una trascrizione dell'audio.

    ![Video Indexer con un lettore video e un riquadro Sequenza temporale che mostra la trascrizione del video.](../media/video-indexer-transcript.png)

1. In alto a destra nel portale selezionare il simbolo **Visualizza** (simile a &#128455;), quindi nell'elenco di informazioni dettagliate, oltre a **Trascrizione**, selezionare **OCR** e **Parlanti**.

    ![Menu di visualizzazione di Video Indexer con le opzioni Trascrizione, OCR e Altoparlanti selezionate](../media/video-indexer-view-menu.png)

1. Notare che ora il riquadro **Sequenza temporale** include quanto segue:
    - Trascrizione della narrazione audio.
    - Testo visibile nel video.
    - Indicazioni dei parlanti che compaiono nel video. Alcune persone note vengono riconosciute automaticamente per nome, mentre altre sono indicate dal numero (ad esempio *Speaker #1*).
1. Tornare nel riquadro **Informazioni dettagliate** e visualizzare le informazioni dettagliate che contiene. che includono:
    - Singole persone che compaiono nel video.
    - Argomenti trattati nel video.
    - Etichette per gli oggetti visualizzati nel video.
    - Entità denominate, ad esempio persone e marchi visualizzati nel video.
    - Scene chiave.
1. Con il riquadro **Informazioni dettagliate** visualizzato, selezionare di nuovo il simbolo **Visualizza**, quindi nell'elenco di informazioni dettagliate aggiungere **Parole chiave** e **Sentiment** al riquadro.

    Le informazioni dettagliate consentono di determinare i temi principali del video. Ad esempio, gli **argomenti** di questo video mostrano chiaramente che si tratta di tecnologia, responsabilità sociale ed etica.

## Cercare informazioni dettagliate

È possibile usare Video Indexer per cercare informazioni dettagliate nel video.

1. Nella casella **Cerca** del riquadro **Informazioni dettagliate** immettere *Bee* (Ape). Può essere necessario scorrere verso il basso nel riquadro per visualizzare i risultati per tutti i tipi di informazioni dettagliate.
1. Si noti che viene trovata un'*etichetta* corrispondente, con la relativa posizione nel video indicata al di sotto.
1. Selezionare l'inizio della sezione in cui è indicata la presenza di un'ape e visualizzare il video in corrispondenza di quel punto (può essere necessario mettere in pausa il video e selezionare con attenzione: l'ape viene visualizzata solo brevemente).

    ![Risultati della ricerca di Video Indexer per Bee](../media/video-indexer-search.png)

1. Cancellare la casella **Cerca** per visualizzare tutte le informazioni dettagliate per il video.

## Usare l'API REST di Video Indexer

Video Indexer include un'API REST che è possibile usare per caricare e gestire i video nell'account.

1. In una nuova scheda del browser aprire il [portale di Azure](https://portal.azure.com) all'indirizzo `https://portal.azure.com` e accedere usando le credenziali di Azure. Mantenere aperta la scheda esistente con il portale di Video Indexer.
1. Nel portale di Azure usare il pulsante **[\>_]** a destra della barra di ricerca nella parte superiore della pagina per creare una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***PowerShell*** senza archiviazione nella sottoscrizione.

    Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

    > **Nota**: Se il portale chiede di selezionare una risorsa di archiviazione per salvare in modo permanente i file, scegliere **Nessun account di archiviazione richiesto**, selezionare la sottoscrizione in uso e premere **Applica**.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    **<font color="red">Verificare di passare alla versione classica di Cloud Shell prima di continuare.</font>**

1. Ridimensionare il riquadro di Cloud Shell in modo da visualizzarne una parte più ampia.

    > **Suggerimento**" È possibile ridimensionare il riquadro trascinando il bordo superiore. È anche possibile usare i pulsanti Riduci a icona e Ingrandisci per spostarsi tra Cloud Shell e l'interfaccia principale del portale.

1. Nel riquadro Cloud Shell immettere i comandi seguenti per clonare il repository GitHub contenente i file di codice per questo esercizio (digitare il comando o copiarlo negli Appunti e quindi fare clic con il pulsante destro del mouse nella riga di comando e incollarlo come testo normale):

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Dopo aver clonato il repository, passare alla cartella contenente il file di codice dell'applicazione per questo esercizio:  

    ```
   cd mslearn-ai-vision/Labfiles/video-indexer
    ```

### Ottenere i dettagli dell'API

Per usare l'API di Video Indexer, sono necessarie alcune informazioni per autenticare le richieste:

1. Nel portale di Video Indexer espandere il riquadro sinistro e scegliere la pagina **Impostazioni account**.
1. Prendere nota dell'**ID account** in questa pagina, perché sarà necessario più avanti.
1. Aprire una nuova scheda del browser e passare al [portale per sviluppatori di Video Indexer](https://api-portal.videoindexer.ai) all'indirizzo https://api-portal.videoindexer.ai, accedendo con le credenziali di Azure.
1. Nella pagina **Profilo** visualizzare le **Sottoscrizioni** associate al profilo.
1. Nella pagina con le proprie sottoscrizioni si noti che sono state assegnate due chiavi (primaria e secondaria) per ognuna. Selezionare quindi **Mostra** per visualizzare una delle chiavi. Questa chiave sarà necessaria tra poco.

### Usare l'API REST

Dopo aver ottenuto l'ID account e una chiave API, è possibile usare l'API REST per lavorare con i video nell'account. In questa procedura si userà uno script di PowerShell per effettuare chiamate REST, ma gli stessi principi si applicano alle utilità HTTP, ad esempio cURL o Postman, o a qualsiasi linguaggio di programmazione in grado di inviare e ricevere codice JSON su HTTP.

Tutte le interazioni con l'API REST di Video Indexer seguono lo stesso modello:

- Viene effettuata una richiesta iniziale al metodo **AccessToken** con la chiave API nell'intestazione per ottenere un token di accesso.
- Le richieste successive usano il token di accesso per l'autenticazione quando si chiamano i metodi REST per lavorare con i video.

1. In Cloud Shell usare il comando seguente per aprire lo script di PowerShell:

    ```
   code get-videos.ps1
    ```
    
1. Nello script di PowerShell sostituire i segnaposto **YOUR_ACCOUNT_ID** e **YOUR_API_KEY** con i valori dell'ID account e della chiave API identificati in precedenza.
1. Si noti che il valore di *location* per un account gratuito è "trial". Se è stato creato un account di Video Indexer senza restrizioni (con una risorsa di Azure associata), è possibile cambiare questa località impostando l'area di Azure in cui è stato effettuato il provisioning della risorsa di Azure, ad esempio "eastus".
1. Esaminare il codice dello script, notando che richiama due metodi REST: uno per ottenere un token di accesso e un altro per elencare i video presenti nell'account.
1. Salvare le modifiche (premere *CTRL+S*), chiudere l'editor di codice (premere *CTRL+Q*) e quindi eseguire il comando seguente per eseguire lo script:

    ```
   ./get-videos.ps1
    ```
    
1. Visualizzare la risposta JSON dal servizio REST, che dovrà contenere i dettagli del video **Responsible AI** indicizzato in precedenza.

## Usare i widget di Video Indexer

Il portale di Video Indexer è un'interfaccia utile per gestire i progetti di indicizzazione video. È tuttavia possibile che in alcuni casi si voglia rendere il video e le relative informazioni dettagliate disponibili per persone che non hanno accesso all'account di Video Indexer. Video Indexer fornisce widget che è possibile incorporare in una pagina Web per tale scopo.

1. Usare il comando `ls` per visualizzare il contenuto della cartella **video-indexer**. Si noti che contiene un file **analyze-video.html**. Si tratta di una pagina HTML di base a cui si aggiungeranno i widget **Player** e**Informazioni dettagliate** di Video Indexer.
1. Immettere il comando seguente per modificare il file:

    ```
   code analyze-video.html
    ```

    Il file viene aperto in un editor di codice.
   
1. Si noti il riferimento allo script **vb.widgets.mediator.js** nell'intestazione: questo script consente l'interazione di più widget di Video Indexer nella pagina.
1. Nel portale di Video Indexer tornare nella pagina **File multimediali** e aprire il video **Intelligenza artificiale responsabile**.
1. Sotto il lettore video selezionare **&lt;/&gt; Incorpora** per visualizzare il codice iframe HTML per incorporare i widget.
1. Nella finestra **Incorpora e condividi** selezionare il widget **Player**, impostare le dimensioni del video su 560 x 315 e quindi copiare il codice di incorporamento negli Appunti.
1. Nell'editor di codice per il file **analyze-video.html** nel riquadro Cloud Shell del portale di Azure incollare il codice copiato sotto il commento **&lt;-- Player widget goes here -- &gt;**.
1. Tornare al portale di Video Indexer e nella finestra di dialogo **Condividi e incorpora** selezionare il widget **Informazioni dettagliate**, quindi copiare il codice di incorporamento negli Appunti. Chiudere quindi la finestra di dialogo **Incorpora e condividi**, tornare al portale di Azure e incollare il codice copiato sotto il commento **&lt;-- Insights widget goes here -- &gt;**.
1. Dopo aver modificato il file, nell'editor di codice salvare le modifiche (*CTRL+S*) e quindi chiudere l'editor di codice (*CTRL+Q*) mantenendo aperta la riga di comando di Cloud Shell.
1. Nella barra degli strumenti di Cloud Shell immettere il comando seguente (specifico di Cloud Shell) per scaricare il file HTML modificato:

    ```
    download analyze-video.html
    ```

    Il comando download crea un collegamento popup in basso a destra del browser, che è possibile selezionare per scaricare e aprire il file, che dovrebbe essere simile al seguente:

    ![Widget di Video Indexer in una pagina Web](../media/video-indexer-widgets.png)

1. Sperimentare con i widget, usando il widget **Insights** per cercare informazioni dettagliate e individuarle nel video.

