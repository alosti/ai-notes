# 3. Il metodo

## Cinque fasi, sempre le stesse

Analizzando le quaranta sessioni di lavoro con Claude Code — diciannove sul backend e ventuno sul frontend — emerge uno schema ricorrente. Non l'ho progettato a tavolino prima di cominciare, è cresciuto da solo, sessione dopo sessione, fino a diventare il modo naturale in cui affrontavo ogni problema. A posteriori, posso descriverlo come una sequenza di cinque fasi.

**Contesto.** Ogni sessione comincia con un'inquadratura precisa del problema. Non un generico "devo fare questa cosa", ma un'indicazione specifica dei file coinvolti, della situazione attuale, di cosa funziona e cosa no. Più il contesto è preciso, meno margine di interpretazione resta allo strumento, e meno probabilità ci sono che parta nella direzione sbagliata.

**Vincolo.** Subito dopo il contesto, arriva l'istruzione su cosa non fare. "Non toccare il codice." "Senza modificare." "Analizza ma non implementare." Questo è il passaggio che separa un uso consapevole dello strumento da un uso passivo. Il vincolo impedisce a Claude Code di agire prima che io abbia capito cosa sta succedendo. Sembra controintuitivo — hai uno strumento potente e la prima cosa che fai è impedirgli di fare qualcosa — ma è esattamente questa la mossa che fa la differenza.

**Diagnosi.** Con il contesto dato e l'azione bloccata, Claude Code è costretto a pensare. Se nel prompt c'è anche un "Think Hard", il ragionamento esteso si attiva e l'analisi che produce è più approfondita. Questa è la fase in cui lo strumento mi restituisce la sua comprensione del problema — che io poi valuto, sfido, correggo. È un dialogo, non un oracolo.

**Implementazione.** Solo quando la diagnosi è condivisa — quando entrambi, io e lo strumento, abbiamo una comprensione allineata del problema e della soluzione — arriva il momento di scrivere codice. A questo punto l'implementazione è quasi meccanica, perché le decisioni importanti sono già state prese nella fase precedente.

**Verifica.** L'implementazione non chiude la sessione. Dopo ogni modifica, verifico il risultato. A volte la verifica è un test eseguito, a volte è una rilettura del codice, a volte è un "ricontrolla tutto" esplicito. Se la verifica fallisce, si torna alla diagnosi — non si rattoppa alla cieca.

Questa sequenza non è rigida. In sessioni semplici, la fase di diagnosi si riduce a poche righe. In sessioni complesse, l'iterazione tra diagnosi e vincolo può durare più a lungo dell'implementazione stessa. Ma lo schema di fondo è sempre lo stesso: capire prima, agire poi, verificare sempre.

## "Non toccare il codice"

Di tutte le frasi che ho usato nelle sessioni, "non toccare" è la più frequente — sedici volte nel backend, diciassette nel frontend. Non è un caso. È la frase che attiva la fase più importante dell'intero metodo: la separazione tra analisi e azione.

Quando lavori con un LLM che ha accesso diretto al codice, la tentazione naturale dello strumento è agire subito. Gli descrivi un problema e lui parte a modificare file. Se il problema è semplice e ben definito, questo funziona. Ma se il problema è complesso, se coinvolge più file, se ci sono vincoli non ovvi o dipendenze nascoste, l'azione immediata è quasi sempre prematura. Lo strumento produce una soluzione che risolve il problema come lo ha capito lui — che non è necessariamente come lo hai capito tu.

"Non toccare il codice" interrompe questo automatismo. Forza lo strumento a restare nella modalità analitica, a esaminare il problema senza la fretta di risolverlo. E forza me a leggere l'analisi, a valutarla, a decidere se corrisponde alla mia comprensione della situazione.

Un esempio concreto viene dalla sessione sul frontend in cui il backend aveva cambiato la struttura dei dati per la tabella dei cicli. Il prompt iniziale era dettagliato: includeva la nuova struttura JSON campo per campo, una tabella con le differenze rispetto alla versione precedente, e tre richieste precise — analizzare i nuovi dati, valutare l'impatto, aggiornare i mock. Ma la frase che chiudeva il prompt era: "Non toccare il codice. Think Hard!"

Il risultato è stato un'analisi di due pagine che mappava ogni campo vecchio sul campo nuovo, identificava sei punti di impatto nel componente Vue con i numeri di riga, segnalava i crash potenziali dovuti ai null guard mancanti, e proponeva scelte di design per i casi ambigui — come gestire la start_date su mobile, se separare o unire le colonne Trascorso e Stimato, come distinguere visivamente una stima puntuale da un range. Tutto senza toccare una riga di codice.

Se avessi chiesto direttamente "aggiorna il componente alla nuova struttura dati", avrei ottenuto un componente aggiornato. Probabilmente funzionante. Ma non avrei avuto quella mappa degli impatti, quelle domande di design che mi hanno fatto riflettere su scelte che non avevo considerato. Non avrei scoperto che il `.toFixed()` crashava su campi opzionali prima di vederlo crashare davvero. La fase diagnostica, forzata dal vincolo, ha prodotto valore che l'implementazione diretta non avrebbe prodotto.

## L'arte della specifica

C'è un livello ancora più profondo del vincolo, e l'ho usato soprattutto nel backend dove la mia competenza tecnica era maggiore. Invece di dare un problema e chiedere un'analisi, davo direttamente la soluzione — sotto forma di piano di implementazione completo.

La sessione più rappresentativa di questo approccio è quella in cui ho implementato l'analisi ciclica nel backend. Il prompt non era una richiesta — era un documento tecnico di due pagine. Conteneva le costanti da usare, le firme delle funzioni con i loro parametri, la logica di ogni funzione helper scritta in pseudocodice, la struttura del JSON di output, il flusso dati dall'engine al responder, una tabella delle incongruenze tra la specifica originale e il codice reale, e persino i casi limite da testare.

Claude Code ha ricevuto questo piano e l'ha implementato in due minuti. Due file modificati, logica corretta, una sola svista minore — un import inutilizzato. Il costo della sessione è stato inferiore a due dollari.

Ma quel piano non è apparso dal nulla. Scriverlo mi ha richiesto tempo — probabilmente più tempo di quanto ne avrebbe richiesto implementare il codice direttamente. La differenza è che quel tempo l'ho speso a pensare, non a scrivere codice. A decidere come doveva funzionare la stima della durata, a definire cosa succede quando i dati non bastano, a verificare che la struttura degli errori fosse compatibile con l'engine che li avrebbe consumati. Quando ho incollato il piano in Claude Code, tutte le decisioni erano già prese. Lo strumento doveva solo tradurle in Python.

Questo è l'approccio da architetto di cui parlavo nel primo capitolo. Non stavo scrivendo codice — stavo scrivendo una specifica così precisa che la traduzione in codice diventava quasi banale. E se la traduzione è banale, non ha senso farla io quando ho uno strumento che la fa più veloce di me.

## Sfidare le risposte

La diagnosi non è un atto di fede. Quando Claude Code produce un'analisi, quella analisi va verificata — e l'unico modo per verificarla è sfidarla con dati, domande, obiezioni.

Nella sessione sulla nomenclatura dei cicli, il dialogo è stato lungo e conflittuale. Claude Code ha proposto una prima soluzione — un sistema di bucketing con fasce fisse che mappava le durate su nomi predefiniti come "ciclo mensile" o "ciclo semestrale". La proposta aveva una logica, ma era fragile: cosa succede quando un ciclo sta sul bordo di una fascia? Oggi misura 58 giorni e si chiama "ciclo a due mesi", domani ne misura 62 e diventa "ciclo trimestrale". Per un utente che segue l'analisi nel tempo, un nome che cambia senza motivo apparente è confusione pura.

Ho contestato la proposta. Claude Code ha riformulato, ma non nella direzione giusta — continuava a ragionare in termini di fasce e soglie. La soluzione vera era molto più semplice: usare direttamente la durata misurata come nome. "Ciclo a 24 giorni", "Ciclo a 90 giorni". Nessuna fascia, nessuna ambiguità, nessun bordo.

Lo strumento, in questo caso, stava complicando il problema. Cercava una soluzione elegante dove serviva una soluzione onesta. Il mio contributo non era tecnico — la matematica la sapeva fare benissimo — era di buon senso applicato al dominio. Sapevo come ragiona un trader che legge quei dati, e sapevo che un nome che cambia da un giorno all'altro è peggio di un nome semplice che dice esattamente quello che misura.

Questo tipo di interazione è forse il cuore di tutto il metodo. Lo strumento è bravo a scrivere codice, a trovare bug, a produrre analisi strutturate. Ma le decisioni di design — quelle che riguardano cosa è giusto per l'utente finale, cosa ha senso nel dominio specifico, cosa è semplice e cosa è complicato senza motivo — quelle restano umane. E l'unico modo per farle emergere è non accettare la prima risposta, ma discuterla.

## Ancora un giro in più

C'è stato poi un altro passaggio in quella stessa sessione che vale la pena raccontare, perché mostra come la sfida può andare avanti diversi giri prima di convergere.

Dopo aver concordato di usare la durata misurata come nome, restava da decidere come esprimere le unità di tempo. Claude Code ha proposto di usare le barre — il che era corretto per i dati giornalieri, dove una barra corrisponde a un giorno, ma completamente sbagliato per qualsiasi altro timeframe. Su dati settimanali, 90 barre non sono 90 giorni, sono 630. Su dati intraday, 90 barre potrebbero essere 90 ore o 90 quarti d'ora a seconda del timeframe.

Ho fatto notare l'errore. Claude Code ha corretto il tiro e proposto di convertire le barre in giorni calendario usando gli indici datetime — approccio corretto in sé, ma nella versione proposta usava `.days` sulla timedelta, che perde le frazioni di giorno. Su dati intraday, turning point distanziati di poche ore avrebbero prodotto `.days = 0`, perdendo l'informazione.

Servono due osservazioni su questo giro di iterazioni. La prima: lo strumento ha sbagliato due volte di fila, in due modi diversi. La seconda: in entrambi i casi, l'errore non era di programmazione — era di comprensione del dominio. Claude Code non sapeva che i dati potevano essere intraday, e non sapeva che `.days` tronca le frazioni. Io lo sapevo, e il mio lavoro in quel momento era riportare quella conoscenza nel dialogo finché la soluzione non diventava corretta.

Alla fine siamo arrivati a usare `total_seconds() / 60` come unità interna, con una funzione di conversione che sceglie l'unità più naturale per ogni valore. Ma ci siamo arrivati dopo tre giri di obiezioni e correzioni — non dopo la prima risposta.

## Il piano come prompt

Tornando alla sessione dei cicli, vale la pena soffermarsi su cosa conteneva quel piano di implementazione, perché è un esempio concreto di come la competenza di dominio si traduce in un prompt efficace.

Il piano non si limitava a dire "implementa l'analisi ciclica". Conteneva:

Le **costanti** con i loro valori e il motivo di quei valori — il minimo di 200 barre non era un numero arbitrario, era calcolato sul warmup necessario per il canale a 153 periodi.

Le **firme delle funzioni** con i tipi dei parametri e i valori di ritorno — non "fai una funzione che stima la durata", ma `_estimate_duration(turning_points) -> Optional[float]` con la logica specifica per tre minimi, due minimi, meno di due minimi.

La **struttura degli errori** compatibile con il codice a valle — il dict di errore doveva avere le chiavi `error`, `instrument`, `message` perché l'engine faceva `block['instrument']` e `block['message']` alle righe 707-708.

Il **flusso dati completo** dall'engine al responder, con ogni trasformazione intermedia.

Le **incongruenze** tra la specifica originale e il codice reale, già risolte nel piano — così Claude Code non si sarebbe confuso trovando discrepanze tra documentazione e implementazione.

I **casi limite** da testare, scritti come condizione e risultato atteso.

Tutto questo non era lavoro dell'LLM — era lavoro mio. Conoscevo il codebase, sapevo dove le cose potevano rompersi, sapevo quali decisioni di design avevo preso e perché. Il piano era la traduzione di questa conoscenza in un formato che Claude Code poteva eseguire senza ambiguità.

Questo approccio funziona particolarmente bene quando la persona ha una competenza forte sul dominio. Non serve scrivere codice — serve scrivere una specifica così precisa che il codice ne diventa la conseguenza naturale. È un lavoro diverso dalla programmazione, ma non meno impegnativo. Richiede la capacità di pensare al problema nella sua interezza prima di scrivere una sola riga, di prevedere i punti di rottura, di specificare i casi limite.

Richiede, in altre parole, seniority.

## "Reverta TUTTO"

Non tutte le sessioni vanno bene. La capacità di riconoscere quando un percorso è sbagliato e tornare indietro è parte integrante del metodo.

In una sessione sul backend, dopo diversi giri di modifiche a due file che gestivano il download dei ticker, ho valutato che la direzione presa non era quella giusta. Le modifiche si stavano accumulando, la complessità cresceva, e la soluzione stava diventando più complicata del problema che doveva risolvere. A quel punto la decisione è stata netta: "reverta TUTTO, riporta il codice all'inizio della chat".

Claude Code ha eseguito. Ha letto lo stato dei file, ha identificato tutte le modifiche fatte durante la sessione, e ha riportato entrambi i file al punto di partenza. Operazione pulita, nessun residuo.

Questo è un vantaggio poco raccontato del vibe coding. Quando scrivi codice a mano e ti accorgi dopo un'ora che la direzione è sbagliata, disfare il lavoro è doloroso — psicologicamente, perché "butti via" lavoro fatto, e praticamente, perché devi ricordare cosa hai cambiato e come era prima. Con Claude Code, il revert è un comando. Non c'è attaccamento emotivo al codice scritto, perché non l'hai scritto tu. E non c'è il problema di ricordare lo stato precedente, perché lo strumento lo sa.

La lezione è semplice: se la direzione è sbagliata, fermati e torna indietro. Non cercare di salvare il lavoro fatto adattandolo, non aggiungere complessità per compensare una scelta iniziale sbagliata. Il costo di ricominciare è basso — molto più basso del costo di trascinare avanti una soluzione che non convince.

## Il ponte tra due progetti

C'è una tecnica che ho sviluppato lavorando su ARIA che merita un capitolo a sé, perché ha risolto un problema che chiunque lavori su un sistema composto da più progetti conosce bene: come trasferire conoscenza tra il backend e il frontend quando i due vivono in repository separati, e le due istanze di Claude Code non si parlano tra loro.

Il problema è concreto. Quando implemento una nuova funzionalità nel backend — un nuovo endpoint, un cambio nella struttura dei dati, un protocollo di comunicazione — il frontend deve sapere esattamente cosa è cambiato per adeguarsi. In un team tradizionale, questa informazione passa attraverso riunioni, messaggi, documentazione scritta a mano, e spesso si perde qualche pezzo per strada. Con due istanze di Claude Code su due progetti diversi, il rischio è lo stesso: l'istanza del frontend non sa cosa ha fatto l'istanza del backend.

La soluzione che ho trovato è semplice e funziona molto bene: faccio scrivere la documentazione a Claude Code stesso.

Quando ho implementato il servizio di Speech-to-Text nel backend, una volta che il codice funzionava, ho chiesto a Claude Code del backend: "scrivi un documento di specifiche su come funziona il servizio di STT; il documento verrà utilizzato dal front-end per integrare i servizi ed implementare la funzionalità STT in Aria". Il documento è stato salvato nella cartella `docs/` del progetto.

Il risultato è stato un file di specifiche completo — requisiti audio con codice di conversione pronto all'uso, protocollo WebSocket con tutti i tipi di messaggio client-server e server-client, codici di chiusura, quote per tier, diagrammi di sequenza per i casi d'uso principali, snippet JavaScript, note implementative sui punti non ovvi. Claude Code del backend ha scritto questa documentazione con una precisione che io non avrei raggiunto a mano, perché lui aveva appena scritto quel codice e ne conosceva ogni dettaglio.

Poi il documento è stato copiato nella cartella `docs/` del progetto frontend. A quel punto, aprendo Claude Code sul frontend, il prompt diventava: "Lato backend ho implementato la funzionalità di speech to text. Leggi la documentazione e fammi una proposta di come integreresti questa nuova funzionalità. Fino a quando non abbiamo le idee chiare non toccare il codice. Think Hard!" — con il riferimento al file `@docs/STT_API.md`.

Claude Code del frontend ha letto il documento, ha analizzato il codice esistente, e ha prodotto una proposta di integrazione dettagliata — posizione del pulsante, stati dell'interfaccia, gestione del testo trascritto, architettura del composable, gestione degli errori. Tutto basato sulla specifica scritta dall'altra istanza.

La cosa più potente di questa tecnica è che il documento vive e si aggiorna. Quando nel backend ho aggiunto i campi per le quote STT nei servizi di admin, ho chiesto a Claude Code di aggiornare lo stesso documento: "aggiungi al documento `@docs/STT_API.md` anche la parte sui servizi di admin, in modo che il front-end possa gestire i nuovi campi". Il documento è stato aggiornato, copiato nel frontend, e lì il prompt era: "Le variazioni ai servizi con i nomi dei campi sono scritte in `@docs/STT_API.md`" — e Claude Code del frontend sapeva esattamente cosa era cambiato e dove intervenire.

In pratica, ho creato un protocollo di comunicazione tra due istanze dello strumento che non possono parlarsi direttamente. Il documento fa da contratto tra i due lati del sistema: il backend lo scrive, il frontend lo legge, e io faccio da postino. Il bello è che il documento è scritto da chi conosce il codice nel modo più intimo possibile — l'istanza che l'ha appena implementato — e viene letto da chi ha bisogno di quella conoscenza per fare il proprio lavoro.

Non è una tecnica che ho letto da qualche parte. È nata dalla necessità pratica di tenere allineati due progetti che evolvono insieme, e si è rivelata uno degli strumenti più efficaci dell'intero processo.

Una nota pratica per chi volesse adottare questo approccio: nel mio caso, il trasferimento dei documenti tra i due progetti era un banale `cp` da una cartella all'altra. Entrambi i progetti hanno una cartella `docs/` nel repository git, e i file di specifica vivono lì insieme al codice. Questa scelta è deliberata. Tenere la documentazione dentro il repository dà storia e corpo al progetto — i documenti evolvono insieme al codice, sono versionati con lo stesso strumento, e chiunque cloni il repository ha tutto quello che serve per capire come funziona il sistema.

In un contesto aziendale con più team, la documentazione potrebbe vivere anche su piattaforme documentali come SharePoint o Teams — che hanno il vantaggio di un versioning più accessibile per chi non mastica git e di una disponibilità immediata senza bisogno di clonare un repository. Ma il mio consiglio resta quello di copiare i documenti anche nella cartella del progetto. La ragione è semplice: quando Claude Code apre una sessione, i file nel repository sono il suo contesto naturale. Un documento nella cartella `docs/` è a un `@docs/STT_API.md` di distanza. Un documento su SharePoint richiede un passaggio in più — scaricarlo, incollarlo, o descriverne il contenuto a mano — e ogni passaggio in più è attrito, e l'attrito è esattamente quello che questo metodo di lavoro cerca di eliminare.

## Il metodo non è una formula

Voglio essere chiaro su un punto: la sequenza che ho descritto — contesto, vincolo, diagnosi, implementazione, verifica — non è una formula da applicare meccanicamente. Non ho mai aperto una sessione pensando "adesso faccio la fase uno, poi la fase due". Ho lavorato, e il metodo è emerso dal lavoro.

La ragione per cui lo descrivo come un metodo è che analizzando le sessioni a posteriori, lo schema è evidente. Le fasi ci sono, la sequenza si ripete, i pattern sono misurabili — sedici "non toccare", tredici "Think Hard", undici "implementa", otto "controlla". I numeri raccontano una storia coerente: prima freno, poi analizzo, poi agisco, poi verifico.

Ma i numeri raccontano anche un'altra cosa. Su quaranta sessioni, ci sono state sessioni in cui il vincolo non serviva perché il problema era semplice e ben definito. Sessioni in cui la diagnosi si è ridotta a una riga perché la causa era ovvia. Sessioni in cui l'implementazione è stata immediata e la verifica è stata un test che passava al primo colpo. Il metodo si adatta al problema, non il contrario.

Quello che resta costante è l'atteggiamento: non fidarsi ciecamente, non delegare le decisioni, non accettare la prima risposta solo perché suona ragionevole. Lo strumento è potente, ma il cervello è il mio.
