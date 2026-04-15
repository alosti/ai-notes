# Parte 2 — Nel mondo reale

---

# 6. ML classico in produzione

## Dalla teoria al software che gira

Finora abbiamo parlato di concetti. Adesso vediamo cosa succede quando il ML esce dal notebook Jupyter e finisce dentro un sistema che deve funzionare 24 ore al giorno, 7 giorni su 7, con soldi veri o decisioni reali in gioco. Perché la distanza tra "il modello funziona sul mio portatile" e "il modello funziona in produzione" è enorme — e piena di cadaveri.

I tre esempi che seguono sono scelti perché ognuno illustra un pattern diverso, e tutti e tre hanno paralleli diretti con il trading.

## Fraud detection: il fratello gemello del trading

Il rilevamento delle frodi nelle transazioni bancarie è probabilmente il caso d'uso di ML più vicino al trading. Non è un caso che le banche usino questi sistemi.

Il problema è semplice da descrivere: ogni giorno passano milioni di transazioni, e devi identificare quelle fraudolente. Che sono una frazione minuscola — tipicamente lo 0,1-0,5% del totale. Ti ricorda qualcosa? È esattamente il problema dello class imbalance che abbiamo visto.

Le feature sono cose come: importo della transazione, ora del giorno, distanza geografica dall'ultima transazione, frequenza delle transazioni recenti, tipologia del merchant, deviazione dal pattern abituale del cliente. Nota come sono tutte numeriche o categoriche — dati tabulari. E infatti i modelli che funzionano meglio sono Random Forest, XGBoost e gradient boosting in generale, non reti neurali profonde.

Ma ecco la parte che interessa per il parallelo con il trading: il modello da solo non basta. Servono almeno tre cose in più.

**Una soglia calibrata con cura.** Se il modello blocca troppe transazioni legittime (falsi positivi), i clienti impazziscono e la banca perde soldi in assistenza e reputazione. Se ne lascia passare troppe fraudolente (falsi negativi), la banca perde soldi direttamente. Il tradeoff precision/recall qui è letteralmente un tradeoff economico, esattamente come nel trading.

**Monitoraggio continuo.** I truffatori si adattano. Quando il modello inizia a bloccare un certo tipo di frode, i truffatori cambiano strategia. Il modello che funzionava tre mesi fa può essere già obsoleto. Questo è lo stesso problema dei mercati: i pattern cambiano perché i partecipanti si adattano. In gergo si chiama **concept drift** — il concetto che il modello ha imparato si sposta sotto i suoi piedi.

**Un sistema di feedback.** Quando il modello blocca una transazione e il cliente chiama dicendo "ero io, lasciamela fare", quel dato torna nel sistema e serve per riaddestare il modello. Senza questo ciclo di feedback, il modello si degrada nel tempo. Nel trading il feedback è automatico — il mercato ti dice se avevi ragione — ma usarlo per migliorare il modello è tutt'altro che banale.

## Recommendation: quando il ML modella le preferenze

Quando Netflix suggerisce un film o Amazon propone un prodotto, dietro c'è un sistema di raccomandazione. È un uso del ML molto diverso dal fraud detection, e illustra un altro pattern importante.

L'approccio classico si chiama **collaborative filtering**: non cerca di capire *perché* piace un film, ma trova utenti simili e consiglia quello che è piaciuto a loro. "Chi ha comprato X ha comprato anche Y" è la versione più grezza di questo principio.

Il pattern rilevante per il trading è il seguente: i sistemi di raccomandazione funzionano bene perché operano in un contesto dove i gusti delle persone sono relativamente stabili. Se piacciono i thriller scandinavi, probabilmente piaceranno anche tra sei mesi. I mercati non hanno questa proprietà. Un pattern che "funziona" su un titolo in un certo regime di mercato può smettere di funzionare da un giorno all'altro. La stazionarietà — cioè il fatto che le regole del gioco non cambino — è un presupposto implicito di quasi tutto il ML, ed è un presupposto che i mercati violano sistematicamente.

L'altra lezione è sul **cold start problem**: quando un utente è nuovo, il sistema non sa nulla di lui e le raccomandazioni fanno schifo. Migliora man mano che accumula dati. Questo è analogo al problema di un modello di trading applicato a un mercato o un titolo su cui ha pochi dati storici — le previsioni iniziali sono poco affidabili e migliorano con il tempo, se il regime di mercato resta stabile. Grosso "se".

## Churn prediction: prevedere chi se ne va

Il terzo esempio è la previsione dell'abbandono dei clienti — il churn. Un'azienda di telecomunicazioni vuole sapere quali clienti stanno per passare alla concorrenza, per provare a trattenerli con un'offerta mirata.

Le feature sono cose come: da quanto tempo è cliente, quante volte ha chiamato l'assistenza, se ha cambiato piano di recente, quanti reclami ha aperto, la differenza tra il suo piano e quelli della concorrenza. Di nuovo, dati tabulari. Di nuovo, gradient boosting che domina.

Il pattern interessante qui è quello della **finestra temporale**. "Il cliente abbandonerà" — quando? Nei prossimi 7 giorni? 30? 90? La scelta della finestra cambia radicalmente il problema. Con una finestra corta, il modello è più preciso ma hai meno tempo per agire. Con una finestra lunga, hai più tempo ma il modello è meno affidabile.

Questo è identico al problema della scelta dell'orizzonte temporale nel trading. Prevedere la direzione del mercato domani è un problema diverso dal prevedere la direzione tra una settimana o tra un mese. Più l'orizzonte si allunga, più fattori entrano in gioco e più il rumore si accumula — ma anche più il segnale ciclico ha tempo di manifestarsi. La scelta dell'orizzonte non è un parametro neutro: definisce la natura stessa del problema.

## Il pattern comune

Guardando questi tre casi, emerge una struttura ricorrente che si applica a qualunque applicazione di ML in produzione, trading incluso:

**I dati sono il problema, non l'algoritmo.** In tutti e tre i casi, la parte più costosa e più critica è ottenere dati puliti, etichettati correttamente, e in quantità sufficiente. L'algoritmo è quasi secondario — la differenza tra un buon XGBoost e un ottimo XGBoost è molto più piccola della differenza tra feature scelte bene e feature scelte male.

**Il modello si degrada.** Nessun modello funziona per sempre. I truffatori cambiano tattica, i gusti degli utenti evolvono, i motivi di abbandono cambiano con il mercato. Serve un processo di monitoraggio e riaddestramento continuo. Nei mercati questo è ancora più vero perché il sistema che stai modellando — il mercato stesso — reagisce ai modelli di chi ci opera.

**La metrica del modello non è la metrica del business.** Il modello ottimizza accuracy, precision, recall. Il business vuole sapere quanti soldi risparmia, quanto fatturato genera, quanti clienti trattiene. Tradurre le metriche del modello in metriche economiche è un passaggio che richiede competenza di dominio — non di ML. Un data scientist che non capisce il business produrrà un modello tecnicamente corretto ma economicamente inutile. Un trader che non capisce il ML userà gli strumenti sbagliati. Servono entrambe le competenze, oppure un dialogo serio tra chi le ha.

---

# 7. Deep Learning in produzione

## Dove la profondità serve davvero

Nel capitolo 3 abbiamo detto che il DL batte il ML classico quando i dati hanno una struttura spaziale o sequenziale intrinseca e quando ce n'è tanto. Ora vediamo i tre campi dove questo vantaggio si è tradotto in risultati concreti, e per ciascuno ci chiediamo: cosa si porta a casa un trader da questa storia?

## Computer vision: il trionfo del DL

Il riconoscimento delle immagini è il campo dove il Deep Learning ha vinto in modo più netto e indiscutibile. Prima del 2012, i sistemi di visione artificiale funzionavano così: un ingegnere progettava a mano dei filtri per estrarre feature dalle immagini, poi un classificatore ML classico usava quelle feature per decidere cosa c'era nell'immagine. Funzionava, ma raggiungeva un tetto che sembrava invalicabile.

Le reti neurali convoluzionali — le famose **CNN** — hanno eliminato il collo di bottiglia del feature engineering manuale. Prendono i pixel grezzi e imparano da sole la gerarchia di feature: bordi nel primo layer, forme semplici nel secondo, parti di oggetti nel terzo, oggetti completi nei layer più profondi.

Le applicazioni in produzione oggi sono ovunque. Diagnostica medica: reti che leggono radiografie e TAC con una precisione che in alcuni casi specifici eguaglia o supera i radiologi. Guida autonoma: elaborazione in tempo reale delle immagini per riconoscere corsie, pedoni, ostacoli, segnali. Controllo qualità industriale: telecamere in fabbrica che ispezionano pezzi alla velocità della linea di produzione.

Il motivo per cui funziona così bene è che le immagini hanno una struttura perfetta per il DL: i pixel vicini sono correlati, la posizione relativa conta, e ci sono miliardi di immagini disponibili per l'addestramento. Tutte le condizioni ideali sono soddisfatte.

**Cosa c'entra con il trading?** Poco, direttamente. Ma c'è un'idea che periodicamente riemerge: dare in pasto alla rete i *grafici* dei prezzi come immagini, invece che i dati numerici. L'intuizione è che un trader esperto "vede" i pattern sul grafico e una CNN potrebbe imparare a vederli nello stesso modo.

L'idea è affascinante ma nella pratica non funziona granché, per un motivo semplice: stai buttando via informazione. I dati numerici originali sono più ricchi e più precisi della loro rappresentazione visiva. Quando un trader guarda un grafico, il suo cervello integra esperienza, contesto e intuizione oltre a quello che vede. La rete vede solo i pixel. E quei pixel sono una versione degradata dei numeri sottostanti. È come fotografare un foglio Excel e chiedere a qualcuno di fare i conti dalla foto invece che dai numeri. Si può fare, ma perché dovresti?

## NLP: capire e generare il linguaggio

Il Natural Language Processing è il campo che ha prodotto i cambiamenti più visibili negli ultimi anni. I modelli di linguaggio, i traduttori automatici, i chatbot, i sistemi di riassunto automatico — tutto questo è DL applicato al testo.

Il punto di svolta è stata l'architettura **Transformer**, introdotta nel 2017. Prima dei Transformer, i modelli di linguaggio processavano il testo una parola alla volta, in sequenza, e facevano fatica a mantenere il contesto su frasi lunghe. Il Transformer ha introdotto un meccanismo chiamato **attention** che permette al modello di guardare tutte le parole di un testo contemporaneamente e di decidere quali sono rilevanti per capire il significato di ciascuna. È come la differenza tra leggere un libro una parola alla volta senza poter tornare indietro e avere il libro intero aperto davanti, potendo saltare con lo sguardo dove serve.

**Il collegamento con il trading** qui è più diretto. Il NLP viene usato per il **sentiment analysis** — analizzare notizie, report degli analisti, comunicati aziendali, social media, e trasformare il testo in un segnale numerico che può diventare una feature per un modello di trading.

### Una nota importante sul sentiment analysis e il trading

C'è un detto nel trading: "buy the rumor, sell the news". Sui mercati liquidi, le notizie attese sono già nel prezzo prima che escano. Il sentiment analysis da news funziona — ma per chi? Per gli HFT che parsano il testo della Fed 50 millisecondi dopo il rilascio e piazzano ordini prima che chiunque abbia finito di leggere il titolo. Per uno swing trader che guarda cicli di diversi giorni, quella finestra temporale è irrilevante.

Esistono effetti secondari — una notizia su regolamentazione ambientale che impatta indirettamente un fornitore di materie prime tre livelli sotto nella supply chain. Ma qui si accumulano problemi su problemi: bisogna avere un modello che capisca la catena causale (già difficilissimo), distinguere il segnale reale dalla coincidenza (come insegna Taleb), e soprattutto farlo in modo **ripetibile e sistematico**, non una volta ogni tanto. Un sistema che funziona il 5% delle volte e il restante 95% genera rumore non è un sistema — è un generatore di costi di transazione.

L'altro problema: il training set per questo tipo di modello è minuscolo. Quante volte nella storia una specifica regolamentazione ha impattato uno specifico settore in un modo misurabile e attribuibile? Cinque? Dieci? Non puoi addestrare nulla di affidabile su dieci esempi.

C'è un uso più sottile che vale la pena menzionare. I modelli di linguaggio possono analizzare i **verbali della BCE o della Fed** e quantificare quanto il tono è cambiato rispetto alla riunione precedente. Non il contenuto — quello lo prezzano tutti — ma le sfumature linguistiche. Questo tipo di analisi è stato studiato in ambito accademico e ci sono evidenze che il tono dei comunicati delle banche centrali contenga informazione predittiva marginale sui tassi futuri. Marginale è la parola chiave — non è un segnale che cambia la vita, ma può essere una feature in più in un modello più ampio.

## Generative AI: creazione, non previsione

L'AI generativa — modelli che creano testo, immagini, codice, musica — è il fenomeno mediatico del momento. Ed è un risultato genuinamente impressionante del DL. Ma è fondamentale capire una distinzione: i modelli generativi sono progettati per **creare contenuto plausibile**, non per **fare previsioni accurate**.

Quando un modello di linguaggio genera testo, sta calcolando la parola più probabile dato il contesto precedente. Non sta "ragionando" nel senso umano del termine, e soprattutto non sta facendo previsioni su eventi futuri del mondo reale. La stessa architettura che produce un saggio convincente sull'economia globale non ha nessuna capacità intrinseca di prevedere dove andrà il mercato domani.

Questo è un punto che va sottolineato perché l'hype attorno all'AI generativa ha creato un'aspettativa pericolosa: "se l'AI può scrivere un articolo, sicuramente può prevedere il mercato". No. Sono compiti fondamentalmente diversi. Scrivere un articolo richiede coerenza linguistica e conoscenza pregressa. Prevedere il mercato richiede la capacità di estrarre segnale debole da dati rumorosi in un sistema non stazionario. L'AI generativa è straordinaria nel primo compito e non ha nessun vantaggio particolare nel secondo.

Detto questo, l'AI generativa **è** utile nel workflow del trading — ma come strumento di produttività, non di previsione. Riassumere report, estrarre dati da documenti non strutturati, generare codice per l'analisi, rispondere a domande sui propri dati. È un uso molto diverso dal "predici il mercato".

## La lezione trasversale

Guardando computer vision, NLP e generative AI insieme, emerge un filo conduttore.

Il Deep Learning ha prodotto risultati rivoluzionari in campi dove tre condizioni sono soddisfatte contemporaneamente: la struttura dei dati è ricca e naturale (pixel, testo, audio), i dati disponibili per l'addestramento sono enormi, e il problema è relativamente **stazionario** — un gatto oggi ha lo stesso aspetto di un gatto di cinque anni fa, e una frase grammaticalmente corretta oggi lo è anche domani.

I mercati finanziari violano almeno due di queste tre condizioni. I dati, pur essendo abbondanti in termini di tick, sono molto più limitati in termini di *pattern indipendenti* — migliaia di giorni di trading non equivalgono a milioni di immagini indipendenti, perché i giorni consecutivi sono correlati. E il problema è per definizione non stazionario: le regole cambiano, i partecipanti si adattano, i regimi si alternano.

Questo non significa che il DL sia inutile nel trading. Significa che le aspettative vanno calibrate su questa realtà, non sui successi in campi che hanno proprietà completamente diverse.

---

# 8. Da notebook a produzione: il ciclo di vita di un modello

## Il modello è il 5% del lavoro

C'è una slide famosa di Google, pubblicata in un paper del 2015, che mostra l'architettura di un sistema di ML in produzione. Il box "ML Code" — il modello vero e proprio — è un rettangolino minuscolo al centro. Tutto il resto è infrastruttura: raccolta dati, validazione dati, feature engineering, monitoraggio, servizio, configurazione, testing. Il modello è la ciliegina. Tutto il resto è la torta.

Questo è un punto che chi viene dal mondo accademico o dai corsi online fatica ad accettare. Sul notebook Jupyter, il modello è il protagonista. In produzione, il modello è un componente in mezzo a decine di altri, e spesso non è nemmeno quello che si rompe più frequentemente.

## Il ciclo completo

Un sistema di ML in produzione segue un ciclo che si ripete continuamente. Non è un progetto con inizio e fine — è un processo vivente.

**Raccolta e validazione dei dati.** Tutto parte dai dati, e i dati in produzione sono sporchi. Arrivano in ritardo, con buchi, con formati che cambiano senza preavviso, con valori anomali che nessuno ti ha documentato. Un feed di dati di mercato che un giorno manda i prezzi con 4 decimali e il giorno dopo con 5 è un classico. Un fornitore che cambia il timezone dei timestamp senza avvisare è un altro classico. Se non hai un sistema che valida i dati in ingresso e ti avvisa quando qualcosa non quadra, scoprirai il problema quando il modello inizia a dare risultati insensati — e a quel punto hai già fatto danni.

La validazione non è sofisticata: controlla che i valori siano nei range attesi, che non ci siano buchi temporali, che le distribuzioni non siano cambiate drasticamente rispetto alla settimana precedente. È noioso, non è sexy, e ti salva la vita.

**Feature engineering e pipeline.** Le feature che il modello usa devono essere calcolate in modo identico in addestramento e in produzione. Sembra ovvio, ma è una fonte inesauribile di bug. Se in fase di sviluppo hai calcolato la media mobile a 20 giorni includendo il giorno corrente, e in produzione la calcoli escludendo il giorno corrente perché il dato non è ancora completo, hai un disallineamento che il modello non ti segnala — semplicemente fa previsioni peggiori e tu non capisci perché.

Questo problema si chiama **training-serving skew** ed è uno dei motivi più comuni per cui un modello che funzionava in sviluppo smette di funzionare in produzione. La soluzione è avere un'unica pipeline di calcolo delle feature condivisa tra addestramento e produzione. Una sola base di codice, non due.

**Il modello come servizio.** Una volta addestrato, il modello deve rispondere a richieste in tempo reale (o quasi). Questo significa impacchettarlo in un servizio — un'API, un microservizio, un processo che gira su un server. Deve essere veloce (la latenza conta), affidabile (se cade alle 9:01 di un lunedì di scadenze, hai un problema), e versionato (devi sapere quale versione del modello è in produzione e poter tornare alla precedente se qualcosa va storto).

Nel trading automatico, la latenza del modello si somma alla latenza di esecuzione. Se il modello impiega 200 millisecondi a rispondere, per l'HFT è un'eternità. Per lo swing trading su cicli di giorni è trascurabile — e questo è un altro motivo per cui lo swing trading è più compatibile con il ML classico: modelli semplici, inferenza veloce, nessun bisogno di GPU in produzione.

**Monitoraggio.** Questo è il pezzo che distingue un sistema serio da un giocattolo. Devi monitorare almeno tre cose.

Le performance del modello nel tempo — non l'accuracy sul test set originale, quella è storia. Le performance su dati nuovi, settimana dopo settimana. Se il modello prevedeva correttamente il 58% delle direzioni e dopo tre mesi è sceso al 51%, qualcosa è cambiato.

### Come si misura la performance nel tempo

Quel 58% citato nell'esempio deve essere una media su un periodo, non un dato puntuale. Un dato puntuale non dice niente — è rumore puro. Nella pratica si fa in due modi complementari.

**Finestra mobile sui trade.** Prendi gli ultimi N segnali del modello e calcoli la percentuale di corretti. La dimensione della finestra dipende dalla frequenza operativa. Per uno swing trader che fa 3-5 operazioni al mese, una finestra di 30-50 trade dà un campione statisticamente decente — abbastanza grande da smorzare il rumore, abbastanza corto da reagire se il modello si degrada. Con meno di 20 trade sei nel territorio dell'aneddotica. Con più di 100 rischi di reagire troppo lentamente.

**Finestra temporale fissa.** Calcoli la performance su finestre di 3 mesi rolling, confrontando ogni trimestre con il precedente e con la media storica. Questo è più utile per decisioni strutturali — "devo riaddestrare?" — mentre la finestra sui trade è più utile per il monitoraggio giorno per giorno.

Il consiglio pratico è: usa entrambe. La finestra sui trade come allarme operativo — se scende sotto una soglia, metti il modello in pausa e indaghi. La finestra trimestrale come revisione strategica.

Un dettaglio importante: non guardare solo la media. Guarda anche la **stabilità** della performance nella finestra. Un modello che fa 58% con oscillazioni minime è molto diverso da uno che fa 58% perché alterna periodi al 70% e periodi al 45%. Il secondo è probabilmente sensibile al regime di mercato e dovresti capire a cosa.

Oltre alla performance, devi monitorare la distribuzione dei dati in ingresso. Se le feature che arrivano al modello in produzione hanno una distribuzione diversa da quelle su cui è stato addestrato, il modello è fuori dal suo terreno. Questo si chiama **data drift** ed è il cugino del concept drift. Nel trading, il data drift è la norma: la volatilità cambia regime, i volumi cambiano con le stagioni, le correlazioni tra asset si rompono e si riformano.

E infine, lo stato del sistema. CPU, memoria, latenza, errori. Le cose banali dell'infrastruttura che però, quando si rompono, rendono irrilevante quanto è bravo il modello.

**Riaddestramento.** Quando il monitoraggio ti dice che il modello si sta degradando, devi riaddestrarlo. Due filosofie.

Riaddestramento **schedulato**: ogni settimana, ogni mese, ogni trimestre, a prescindere. Semplice da gestire ma spreca risorse se il modello funziona ancora bene e reagisce in ritardo se si è degradato improvvisamente.

Riaddestramento **trigger-based**: riaddestri solo quando le metriche scendono sotto una soglia. Più efficiente ma richiede un sistema di monitoraggio robusto e la capacità di riaddestare velocemente.

In entrambi i casi, il riaddestramento non è premere un bottone. Devi ricalcolare le feature su dati aggiornati, riaddestare il modello, validarlo sul periodo più recente, confrontare le performance con il modello in produzione, e fare lo switch solo se il nuovo modello è effettivamente migliore. Se automatizzi male questo processo, puoi mettere in produzione un modello peggiore del precedente senza accorgertene.

## La realtà che nessuno ti racconta

Nella pratica quotidiana, un team che gestisce un sistema di ML in produzione passa la maggior parte del tempo su queste attività operative — non a inventare nuovi modelli o a sperimentare architetture innovative. Il lavoro è più simile a quello di un meccanico che tiene in funzione un motore che a quello di un ingegnere che ne progetta uno nuovo.

Questo è importante perché molti si avvicinano al ML pensando che la parte interessante — scegliere il modello, fare esperimenti, vedere i risultati — sia il lavoro. È il 20% del lavoro. L'80% è ingegneria, infrastruttura, e manutenzione.

Per un trader indipendente che gestisce il proprio sistema, questo ha un'implicazione pratica molto diretta: ogni componente di ML che aggiungi al sistema è un componente che devi mantenere. Non solo oggi, ma tra sei mesi e tra due anni. Se aggiungi un modello di regime detection, da quel momento hai una pipeline di feature da mantenere, un modello da monitorare, un processo di riaddestramento da gestire, e un punto di failure in più nel sistema. Il beneficio deve giustificare questo costo operativo, altrimenti hai aggiunto complessità senza aggiungere valore.

---
