# Parte 3 — Trading e ML/DL

---

# 9. Cosa funziona davvero e cosa è marketing puro

## La premessa scomoda

Prima di entrare nel tecnico, mettiamo sul tavolo una verità che l'industria del "fintech AI" non ha interesse a dirti.

I mercati finanziari sono il peggior campo di applicazione possibile per il Machine Learning. Non il migliore, non uno tra tanti — il peggiore. E il motivo è che violano simultaneamente quasi tutte le condizioni che rendono il ML efficace negli altri campi.

Nella fraud detection, un pattern fraudolento oggi è simile a un pattern fraudolento di sei mesi fa. Nel riconoscimento immagini, un gatto oggi è uguale a un gatto del 2015. In un sistema di raccomandazione, i gusti cambiano lentamente. Nei mercati, le regole del gioco cambiano continuamente perché i giocatori si adattano. Un pattern che funziona viene scoperto, sfruttato, e arbitraggiato via — a volte in mesi, a volte in settimane. Il sistema che stai cercando di modellare **reagisce** ai modelli che ci costruisci sopra. È come giocare a scacchi contro un avversario che cambia le regole ogni volta che trovi una strategia vincente.

Questo non significa che il ML sia inutile nel trading. Significa che chi ti promette risultati facili o sta mentendo o non ha capito il problema.

## La mappa dell'hype

Facciamo ordine su cosa esiste là fuori, separando tre categorie.

**Quello che funziona ed è reale** — ma non è accessibile al trader retail. I desk quantitativi delle grandi firme (Renaissance Technologies, Two Sigma, DE Shaw, Citadel) usano ML in modo massiccio e profittevole. Ma operano con vantaggi che non puoi replicare: dati proprietari che costano milioni di dollari all'anno, infrastrutture di esecuzione co-locate nei data center delle borse, team di 50-100 PhD in fisica e matematica, e una capacità di diversificazione su migliaia di strumenti simultanei che diluisce il rischio del singolo modello. Il loro edge non è l'algoritmo — è l'ecosistema. Puoi avere lo stesso XGBoost di Renaissance, ma senza i loro dati, la loro esecuzione e il loro risk management, quel modello non vale niente.

**Quello che ha fondamento scientifico ma risultati modesti.** Esiste una letteratura accademica seria sull'applicazione del ML ai mercati. I risultati sono coerenti: si possono estrarre segnali predittivi deboli, nell'ordine del 51-55% di accuratezza direzionale, che in combinazione con una buona gestione del rischio e costi di transazione bassi possono generare rendimenti positivi. Nota le parole chiave: segnali *deboli*, accuratezza *modesta*, gestione del rischio *buona*, costi *bassi*. Nessuno di questi paper promette di raddoppiare il capitale. Promettono un edge piccolo e fragile che richiede disciplina e infrastruttura per essere sfruttato.

**Quello che è marketing puro.** I corsi online che ti insegnano a "usare l'AI per fare trading", i bot che promettono rendimenti del 30% al mese, i sistemi "completamente automatici" che richiedono solo un deposito iniziale. Questi si riconoscono facilmente: mostrano solo backtest (mai risultati live verificabili), promettono rendimenti fissi o prevedibili (il mercato non funziona così), non parlano mai di drawdown o periodi di perdita (che esistono sempre), e spesso il vero prodotto sei tu — paghi il corso, paghi l'abbonamento al segnale, paghi lo spread maggiorato del broker convenzionato. Se il sistema funzionasse davvero con quei rendimenti, il venditore non avrebbe nessun incentivo economico a venderlo — lo userebbe.

### Un esempio concreto: la confidenza al 100%

Un segnale di allarme perfetto è un sistema che dà una confidenza del 100% su un segnale di trading. Un sistema serio non dà mai 100% su nulla nei mercati. Un modello che ti dice "sono sicuro al 100%" ti sta dicendo una di due cose: o non ha capito l'incertezza intrinseca del mercato, o la sta nascondendo deliberatamente per venderti qualcosa. In entrambi i casi, scappa. Quando persino il trader che sta facendo la demo di un prodotto alza il sopracciglio davanti al 100% di confidenza del suo stesso strumento, il prodotto si è smontato da solo.

## I cinque problemi specifici del ML nei mercati

Vediamo nel dettaglio perché i mercati sono così ostili al ML, punto per punto.

**Problema 1: non stazionarietà.** Un modello di ML assume, implicitamente, che i pattern nei dati di addestramento saranno presenti anche nei dati futuri. Nei mercati questo è vero solo a tratti. I regimi cambiano: periodi di trend si alternano a periodi laterali, la volatilità si comprime e si espande, le correlazioni tra asset si rompono durante le crisi. Un modello addestrato sul 2017-2019 (bassa volatilità, trend rialzista) si trova spaesato nel 2020 (shock esogeno, crollo e rimbalzo violento) e di nuovo nel 2022 (rialzo dei tassi, bear market ordinato). Ogni volta che il regime cambia, il modello è fuori dominio.

**Problema 2: rapporto segnale/rumore bassissimo.** Nei dati di mercato, la componente casuale domina. I movimenti giornalieri dei prezzi sono in larga parte rumore. Il segnale vero — se esiste — è sepolto sotto questo rumore ed è piccolo. Un modello che cerca pattern in dati dominati dal rumore troverà tantissimi pattern fasulli e pochissimi reali, e distinguere gli uni dagli altri è enormemente difficile.

**Problema 3: i dati non sono indipendenti.** Le tecniche standard di validazione nel ML assumono che le osservazioni siano indipendenti. Il prezzo di oggi non lo è rispetto a quello di ieri. I rendimenti sono autocorrelati, la volatilità è clusterizzata, e gli eventi di mercato hanno code lunghe. Se fai cross-validation mescolando i dati a caso, stai usando il futuro per predire il passato e i risultati saranno gonfiati in modo drammatico. Tutto deve essere rigorosamente walk-forward.

**Problema 4: il mercato è un sistema adattivo.** A differenza di un gatto che non cambia aspetto perché qualcuno lo ha fotografato, il mercato cambia *perché* i partecipanti lo osservano e agiscono. Se un pattern predittivo diventa noto, viene sfruttato, e lo sfruttamento stesso lo elimina. Anche un modello che trova un edge reale ha una data di scadenza implicita.

**Problema 5: i costi di transazione mangiano l'edge.** Un modello che ha un edge teorico del 2% per trade sembra interessante. Ma se le commissioni, lo spread bid-ask e lo slippage costano l'1.5% per trade, l'edge reale è dello 0.5% — e basta un leggero peggioramento della performance per andare in negativo. Questo favorisce lo swing trading rispetto all'alta frequenza per il trader indipendente: meno operazioni, meno costi, più tempo perché l'edge si manifesti.

## Allora, serve o non serve?

Dopo questa doccia fredda, la risposta onesta è: sì, serve, ma non nel modo in cui viene venduto.

Il ML non è una sfera di cristallo che predice il mercato. È uno strumento per **sistematizzare e testare rigorosamente** le intuizioni del trader. La differenza è enorme.

Se un trader, dopo vent'anni di esperienza, ha l'intuizione che certi setup all'interno dei canali funzionano meglio quando la volatilità è in contrazione e i volumi confermano, il ML permette di prendere questa intuizione, formalizzarla in feature, testarla su dati storici in modo rigoroso (non con il senno di poi), e quantificare *quanto* funziona e *quando* smette di funzionare. Non dice niente di nuovo sui mercati — dice se quello che credi di sapere è statisticamente fondato o è un'illusione confermata dalla memoria selettiva.

Questo è un uso molto più modesto di quello che promettono i venditori di hype, ma è molto più reale e molto più utile.

---

# 10. ML classico nel trading: dove ha senso

## Il terreno giusto

Dopo la doccia fredda del capitolo precedente, restringiamo il campo a quello che ha basi solide. Il ML classico applicato al trading funziona quando lo usi per tre cose: classificare il regime di mercato, filtrare i segnali di un sistema esistente, e costruire feature composite che sintetizzano informazioni che un umano fatica a tenere insieme simultaneamente.

Nota cosa manca dalla lista: *prevedere la direzione del mercato*. Non perché sia teoricamente impossibile, ma perché quando lo fai direttamente, finisci quasi sempre nell'overfitting.

## Regime detection: l'applicazione naturale

Se si dovesse scegliere una sola applicazione di ML per uno swing trader, sarebbe questa. Il concetto è semplice: il mercato non si comporta allo stesso modo in tutti i periodi. Ci sono fasi di trend, fasi laterali, fasi di alta volatilità, fasi di compressione. Una strategia che funziona in un regime spesso perde soldi in un altro.

Chi fa trading sui cicli lo sa per esperienza diretta — i cicli hanno caratteristiche diverse a seconda del contesto di mercato in cui si sviluppano. Un ciclo di 4-8 giorni in un mercato in trend forte è una cosa. Lo stesso ciclo in un mercato laterale e volatile è un'altra.

Il ML può formalizzare questa distinzione. L'approccio base funziona così: prendi un set di feature che descrivono lo "stato" del mercato — volatilità realizzata su diverse finestre (5, 10, 20 giorni), direzionalità (la pendenza di una media mobile, l'ADX), ampiezza dei canali, distribuzione dei volumi, rapporto tra volatilità intraday e volatilità interday — e usi un algoritmo di clustering non supervisionato per raggruppare i periodi storici in regimi.

L'algoritmo non sa cosa è un "trend" o un "laterale". Trova raggruppamenti nei dati basandosi sulla similarità delle feature. Poi tocca al trader, con la sua esperienza, guardare quei raggruppamenti e interpretarli. Spesso corrispondono a regimi che si riconoscono immediatamente.

Una volta che hai i regimi, puoi fare una cosa potentissima: valutare la strategia **separatamente per ogni regime**. Potresti scoprire che la strategia sui cicli funziona benissimo nei regimi 1 e 3, perde soldi nel regime 2, e va in pari nel regime 4. A quel punto hai un filtro: operi solo nei regimi favorevoli e stai fermo negli altri. Non hai cambiato strategia — hai imparato quando applicarla e quando no.

Gli algoritmi più usati per questo sono K-Means (il più semplice — decidi tu quanti regimi cercare), Gaussian Mixture Models (più flessibile — ogni punto può appartenere parzialmente a più regimi, il che riflette meglio la realtà dei mercati dove le transizioni sono graduali), e Hidden Markov Models (che modellano esplicitamente le transizioni tra regimi nel tempo).

Un avvertimento importante: il numero di regimi non è un dato scientifico, è una scelta di design. Nella pratica, 3-5 regimi sono il range in cui la maggior parte delle applicazioni trova il bilancio giusto. E soprattutto, i regimi devono avere senso dal punto di vista del mercato — se l'algoritmo propone un raggruppamento che non si riesce a interpretare, probabilmente sta trovando rumore.

## Signal filtering: migliorare un sistema esistente

Questo è l'uso più pragmatico del ML per chi ha già un sistema operativo.

L'idea è semplice: il sistema genera dei segnali di trading basati sulla sua logica (cicli, canali, volumi). Non tutti quei segnali sono uguali — alcuni sono più puliti, altri sono ambigui, altri escono in contesti sfavorevoli. Un modello di ML può imparare a distinguere i segnali buoni da quelli cattivi, non sostituendo la logica originale ma facendo da filtro a valle.

In pratica: prendi tutti i segnali storici del sistema, per ciascuno calcoli un set di feature che descrivono il contesto in cui il segnale è apparso (regime di mercato, forza del segnale, posizione nel canale, confluenza con altri indicatori, comportamento dei volumi), e etichetti ciascun segnale con il suo risultato reale. Poi addestri un classificatore — un Random Forest o un XGBoost — per prevedere la qualità del segnale.

Il risultato non è un nuovo sistema di trading. È un punteggio di confidenza che si affianca ai segnali originali: "questo segnale esce in un contesto che storicamente ha funzionato il 65% delle volte" oppure "questo segnale esce in un contesto che storicamente ha funzionato il 43% delle volte — forse meglio lasciar perdere".

Il vantaggio di questo approccio è che rispetta la logica di trading originale. Non si chiede al ML di inventare una strategia — gli si chiede di dire quando la strategia ha le probabilità migliori. E siccome è il trader a definire le feature, può inserire la propria conoscenza del mercato nel modello. La posizione nel canale, la fase del ciclo, la conferma del volume — queste sono feature che un data scientist puro non penserebbe mai di usare, ma che un trader esperto sa essere rilevanti.

L'altra cosa importante: un filtro che elimina il 30% dei segnali peggiori può migliorare drammaticamente il profit factor di un sistema anche senza toccare nient'altro. A volte non fare le operazioni sbagliate vale più che aggiungere operazioni giuste.

## Feature composite: vedere tutto insieme

Il terzo uso è più sottile. Un trader esperto, guardando un grafico, integra simultaneamente decine di informazioni: il prezzo rispetto al canale, il volume, la volatilità, il contesto macro, il comportamento degli ultimi giorni. Lo fa intuitivamente, senza pensarci. Ma questa sintesi intuitiva ha un limite: il cervello umano fatica a pesare correttamente più di 5-7 variabili contemporaneamente.

Un modello di ML può combinare 20 o 30 feature e trovare interazioni tra di esse che un umano non noterebbe. Non perché è più intelligente — perché è più sistematico. Può scoprire che la combinazione "prezzo nella parte bassa del canale + volume in calo da 3 giorni + volatilità intraday in aumento + ADX sotto 20" ha un valore predittivo che nessuna di queste feature ha da sola.

La chiave è che il trader sceglie le feature — la sua esperienza decide cosa è potenzialmente rilevante. Il modello trova le combinazioni e i pesi. È una divisione del lavoro che sfrutta il meglio di entrambi: la conoscenza di dominio dell'umano e la capacità computazionale della macchina.

## Quali algoritmi, in pratica

Per tutti e tre questi usi, gli algoritmi che funzionano meglio sono:

**Random Forest** — robusto, difficile da far esplodere, interpretabile. Buon punto di partenza per qualsiasi problema. Il suo limite è che non è il più preciso in assoluto, ma la sua stabilità nel trading vale più del mezzo punto percentuale di accuracy in più.

**XGBoost e LightGBM** — gradient boosting, il gradino sopra in termini di performance predittiva. Sono lo stato dell'arte per dati tabulari e dominano le competizioni Kaggle. Richiedono un po' più di attenzione nel tuning degli iperparametri e sono più inclini all'overfitting se non li tieni a bada — ma con regolarizzazione adeguata sono strumenti formidabili.

**Support Vector Machines** — meno usati oggi per la classificazione, ma ancora validi per il regime detection con kernel non lineari. Funzionano bene quando il dataset è piccolo, che nel trading è spesso il caso.

Quello che *non* si consiglia per lo swing trading: reti neurali profonde, per tutti i motivi discussi nel capitolo 3. Troppi parametri, troppo pochi dati, troppo facile overfittare, troppo difficile interpretare.

---

# 11. Deep Learning nel trading: LSTM, Transformer e le promesse infrante

## Perché tutti ci provano

Il fascino del DL applicato al trading è comprensibile. Se una rete neurale profonda può riconoscere un volto tra milioni, tradurre una frase in tempo reale, battere il campione mondiale di Go — perché non dovrebbe poter prevedere il mercato? L'intuizione è seducente. E sbagliata, per i motivi che abbiamo già visto. Ma vale la pena capire cosa si è tentato e perché i risultati sono quelli che sono.

## LSTM: la memoria che non basta

Le reti **LSTM** — Long Short-Term Memory — sono una variante delle reti neurali ricorrenti, progettate specificamente per le sequenze temporali. L'idea fondante è che hanno una "memoria" interna: possono ricordare informazioni rilevanti viste molti passi indietro nella sequenza e dimenticare quelle irrilevanti.

L'analogia: immagina di leggere un romanzo. Per capire il capitolo 20, devi ricordare che nel capitolo 3 il protagonista ha tradito un amico — un'informazione distante ma cruciale. Le reti neurali ricorrenti classiche faticano a mantenere queste dipendenze a lungo raggio. Le LSTM hanno un meccanismo esplicito — delle "porte" che decidono cosa ricordare, cosa dimenticare, e cosa trasmettere — che risolve questo problema.

Applicata ai dati di mercato, l'idea è: dai alla rete la sequenza degli ultimi N giorni e le chiedi di prevedere il giorno N+1. La LSTM dovrebbe imparare le dipendenze temporali.

Nella pratica accademica, le LSTM producono risultati che sembrano buoni nei paper ma si ridimensionano molto nella realtà. Il numero di parametri è alto rispetto ai dati disponibili. Molti paper mostrano performance eccellenti sul test set ma non pubblicano risultati out-of-sample su periodi successivi. La rete tratta la sequenza temporale come se avesse una struttura stabile — ma la "grammatica" del mercato cambia nel tempo. E l'interpretabilità è quasi zero: la rete dice "compra" ma non puoi sapere perché.

## Transformer sulle serie temporali: l'ultimo hype

Dopo il successo esplosivo dei Transformer nel linguaggio naturale, la comunità ha iniziato ad applicarli alle serie temporali finanziarie. L'idea è che il meccanismo di attention possa catturare pattern temporali meglio delle LSTM.

I risultati sono misti e il dibattito è aperto. Alcuni paper mostrano che i Transformer funzionano bene su serie temporali lunghe e complesse. Altri hanno mostrato che modelli lineari semplicissimi battono i Transformer su molti benchmark di previsione di serie temporali.

Il punto fondamentale è sempre lo stesso. I Transformer eccellono nel linguaggio perché il linguaggio ha una struttura ricchissima e stabile, e perché sono stati addestrati su quantità di testo che non hanno equivalenti in finanza. Un modello di linguaggio vede centinaia di miliardi di token. Un modello finanziario vede, nella migliore delle ipotesi, qualche milione di osservazioni giornaliere — e queste non sono indipendenti, non hanno una grammatica stabile, e il rapporto segnale/rumore è incomparabilmente più basso.

## I pochi casi in cui il DL aggiunge valore nel trading

Detto tutto questo, ci sono nicchie dove il DL ha un vantaggio legittimo.

**Dati ad altissima frequenza.** Quando si opera sui tick — migliaia di osservazioni al secondo — ci sono finalmente abbastanza dati per addestrare reti profonde senza overfittare immediatamente. Ma questo è il mondo dell'HFT istituzionale, non dello swing trading.

**Dati multimodali.** Se si combinano dati numerici di mercato con testo e magari immagini, il DL è l'unico approccio che può integrare queste fonti diverse in un unico modello. Ma la complessità esplode e il rischio di overfitting su interazioni spurie è altissimo.

**Generazione di feature tramite autoencoder.** Invece di usare il DL per prevedere direttamente, si può usare per comprimere i dati in una rappresentazione più compatta — un autoencoder che prende 50 feature e le riduce a 5 feature "latenti". Poi si usano quelle 5 feature come input per un modello ML classico. È un uso ibrido interessante, ma la domanda è sempre: aggiunge abbastanza valore da giustificare la complessità?

## La regola pratica

Per uno swing trader su cicli di 4-8 giorni, con dati giornalieri, su un paio di indici, la risposta è quasi sempre: no, il DL non aggiunge valore rispetto al ML classico. Non perché sia sbagliato in teoria, ma perché nella pratica ci sono troppo pochi dati, troppo rumore, e un problema troppo non stazionario perché la complessità aggiuntiva del DL si ripaghi.

Il ML classico — XGBoost, Random Forest — con feature ben scelte sulla base dell'esperienza di trading è lo strumento giusto. Non è la risposta che fa titoli su YouTube, ma è quella che funziona.

---

# 12. Analisi dei cicli e ML: dove si incontrano e dove divergono

## Due filosofie che partono da premesse diverse

L'analisi ciclica e il Machine Learning partono da ipotesi fondamentalmente diverse sul mercato, e capire questa differenza è essenziale prima di provare a combinarli.

L'analisi ciclica parte da una premessa strutturale: i mercati si muovono in cicli di durata relativamente prevedibile, e questi cicli si possono isolare dal rumore tramite strumenti come i canali. La premessa è che esiste un meccanismo sottostante — psicologico, legato ai flussi, legato ai cicli economici — che genera queste ricorrenze. Non serve sapere esattamente quale sia il meccanismo: basta che il fenomeno sia sufficientemente regolare da poterci costruire sopra un'operatività.

Il Machine Learning non ha premesse sul mercato. Non sa cosa sia un ciclo, non conosce metodologie, non ha una teoria. Cerca pattern nei dati, qualunque pattern, senza chiedersi se abbiano un senso economico o strutturale. Se trova che il mercato sale ogni terzo martedì del mese quando piove a Milano, lo usa — a meno che qualcuno non lo fermi.

Questa differenza è sia il punto di forza che il punto di rischio della combinazione. Il punto di forza: il ML può trovare conferme o smentite della teoria che la sola osservazione umana non può dare. Il punto di rischio: senza una teoria a guidarlo, il ML troverà pattern fasulli e li tratterà come oro.

## Dove il ML rafforza l'analisi ciclica

Ci sono almeno quattro aree dove il ML può aggiungere qualcosa di concreto.

**Quantificare la regolarità dei cicli.** L'esperienza dice "i cicli su questo indice tendono a durare 4-8 giorni". Il ML può prendere 20 anni di dati, identificare sistematicamente tutti i cicli, misurarne la durata, l'ampiezza, la simmetria, e dire esattamente quanto sono regolari e in quali condizioni lo sono di più o di meno. Si potrebbe scoprire che i cicli sono molto più regolari quando la volatilità implicita è sotto un certo livello. Questa è informazione operativa diretta: dice quando fidarsi di più dell'analisi ciclica e quando essere più cauti.

**Classificare la "qualità" di un ciclo in formazione.** Quando un nuovo ciclo inizia a formarsi, non si sa ancora se sarà un ciclo pulito e tradabile o un ciclo sporco e caotico. Il ML può guardare le caratteristiche delle prime fasi del ciclo e confrontarle con le prime fasi dei cicli storici, classificati per qualità. Il risultato è un punteggio di probabilità: "i cicli che sono iniziati così in passato sono stati puliti nel 70% dei casi".

Non è una certezza — è un'informazione in più che si aggiunge al giudizio del trader. E siccome è il trader a definire cosa significa "ciclo pulito" e "ciclo sporco" attraverso il labeling, il modello sta imparando *il suo* framework, non uno generico.

**Ottimizzare il timing all'interno del ciclo.** Entrare nella parte bassa del canale e uscire nella parte alta è il principio. Ma "parte bassa" non è un punto preciso — è una zona. Il ML può aiutare a restringere quella zona guardando le feature del momento specifico — velocità di avvicinamento al bordo del canale, volume relativo, comportamento degli ultimi bar rispetto alla media del ciclo.

**Rilevare quando i cicli si stanno rompendo.** Questo è forse l'uso più prezioso. I cicli non funzionano sempre — ci sono periodi in cui il mercato smette di comportarsi ciclicamente. Un modello di regime detection può avvisare che le condizioni stanno cambiando prima che sia evidente dal grafico. È il ML che fa da vedetta, non da comandante.

## Dove il ML deve farsi da parte

Ci sono aspetti dell'operatività dove il ML non solo non aiuta, ma può fare danni.

**La teoria sottostante.** L'idea che i mercati abbiano una componente ciclica è un'ipotesi di lavoro che viene dall'esperienza e dalla letteratura. Il ML non può confermare o negare questa ipotesi — può solo dire se i pattern ciclici nei dati sono statisticamente significativi. Ma la significatività statistica su dati finanziari è un terreno minato.

**La gestione del rischio in tempo reale.** Quando si è dentro un trade e il mercato va contro, la decisione di tenere, ridurre o chiudere è un mix di regole predefinite e giudizio situazionale. Il ML non sa che c'è una riunione BCE tra due ore, non sa che il future ha appena fatto un gap, non sa che il drawdown cumulativo è già al limite della tolleranza psicologica.

**Il position sizing.** Quanti soldi mettere su un trade è una funzione del rischio totale del portafoglio, della correlazione con le altre posizioni, del capitale, della soglia di dolore. Un modello ML può stimare la probabilità di successo di un segnale, ma tradurre quella probabilità in una size è un atto di risk management che dipende dalla situazione complessiva.

**L'ultima parola.** Anche se si implementano tutti gli usi positivi descritti, il modello deve essere un input al processo decisionale, non il decisore. Quando esperienza e modello sono d'accordo, bene. Quando sono in disaccordo, è il momento di capire perché — e nella maggior parte dei casi, l'esperienza ha ragione e il modello sta reagendo a rumore.

## Come si combinano nella pratica

Se si dovesse disegnare un'architettura concreta per integrare ML e analisi ciclica, sarebbe questa:

Il sistema genera i segnali come ha sempre fatto — cicli, canali, volumi, la logica di sempre. Questo è il motore primario e non si tocca.

In parallelo, un modello di regime detection classifica lo stato corrente del mercato. Se il regime è favorevole, i segnali passano. Se non lo è, vengono filtrati o ridotti in size.

Per i segnali che passano il filtro di regime, un modello di signal quality stima la probabilità storica di successo di quel tipo di segnale in quel contesto. I segnali ad alta qualità vengono operati normalmente. Quelli a bassa qualità vengono scartati o operati con size ridotta.

Il tutto viene monitorato nel tempo. Se la performance del filtro ML peggiora, si riaddestra. Se peggiora in modo persistente, si mette in pausa e si torna al sistema base senza filtro.

In nessun punto il ML genera segnali autonomi. In nessun punto il ML ha il potere di aprire un trade che la logica originale non avrebbe aperto. Il ML toglie — filtra, pesa, sconsiglia. Non aggiunge. Questa asimmetria è deliberata e fondamentale: un filtro che sbaglia fa perdere qualche opportunità, un generatore di segnali che sbaglia fa perdere soldi.

---

# 13. La doccia fredda: overfitting, backtest bugiardi e le bugie che ci raccontiamo

## Perché questo capitolo è il più importante del libro

Tutto quello che abbiamo costruito nei capitoli precedenti — la teoria, gli algoritmi, le applicazioni — non serve a niente se non si sa riconoscere quando un risultato è falso. E nei mercati finanziari, i risultati falsi sono la norma, non l'eccezione. Non perché la gente sia disonesta — anche se molti lo sono — ma perché il cervello umano e gli strumenti statistici cospirano insieme per far vedere segnali dove non ce ne sono.

## Overfitting sui mercati: perché è 10 volte peggio

Abbiamo parlato di overfitting nel capitolo 3. Nei mercati è un problema di ordine di grandezza diverso, e il motivo è la combinazione di tre fattori che negli altri campi non si presentano insieme.

**Il numero di ipotesi testate è enorme e nascosto.** Quando si sviluppa un sistema di trading, si fanno centinaia di scelte: quale indicatore usare, con quale parametro, su quale timeframe, con quale filtro, con quale regola di uscita, con quale stop. Ogni combinazione è un'ipotesi. Se si provano 1.000 combinazioni e si sceglie la migliore, si sono fatti 1.000 test statistici — e per puro caso, alcune combinazioni avranno funzionato benissimo sui dati storici senza avere nessun valore predittivo. È il problema delle **comparazioni multiple**: se tiri abbastanza freccette a caso, qualcuna colpirà il centro.

Il guaio è che spesso non ci si rende conto di quante ipotesi si sono testate. Ogni volta che si guarda un grafico e si pensa "se avessi usato la media a 15 invece che a 20 avrebbe funzionato meglio", si è aggiunto un test. Ogni volta che si esclude un periodo "anomalo" dal backtest, si è aggiunto un test. Alla fine, il sistema è stato ottimizzato su centinaia di decisioni successive, ciascuna delle quali ha usato informazione dal futuro.

**I gradi di libertà dei dati sono pochi.** Se si hanno 5.000 giorni di trading e 30 feature, il dataset ha circa 5.000 osservazioni indipendenti — ma in realtà meno, perché i giorni consecutivi sono correlati. I cicli indipendenti da 4-8 giorni in 20 anni sono forse 600-800. Sono pochi. Un modello complesso con centinaia di parametri può adattarsi perfettamente a 800 esempi senza aver imparato nulla di generalizzabile.

**Il feedback è ritardato e ambiguo.** Nel riconoscimento immagini, se il modello sbaglia, lo si sa immediatamente e con certezza. Nel trading, un singolo trade in perdita non dice se il modello è sbagliato — potrebbe essere varianza normale. Servono decine o centinaia di trade per avere una valutazione statistica affidabile.

## I backtest bugiardi: anatomia dell'inganno

Un backtest è una simulazione di come un sistema di trading avrebbe performato sui dati storici. In teoria è uno strumento di validazione essenziale. In pratica è il più grande generatore di false speranze nell'industria finanziaria.

Ecco i modi più comuni in cui un backtest mente — e la maggior parte sono errori involontari, non frodi deliberate.

**Look-ahead bias.** Il sistema usa, anche indirettamente, informazione che al momento del trade non era disponibile. Esempio classico: usare il prezzo di chiusura della giornata per decidere un trade che si sarebbe dovuto decidere durante la giornata. Nei sistemi complessi con pipeline di feature articolate, il look-ahead bias può infilarsi in modi sottilissimi.

**Survivorship bias.** Si testa il sistema solo su titoli che esistono oggi. Ma oggi esistono i titoli che sono sopravvissuti — quelli falliti, delistati, o fusi non ci sono più nel dataset. Questo gonfia i risultati perché si testa su un campione selezionato per il successo.

Per chi opera su indici come il MIB o il DAX, il survivorship bias è meno grave che per chi opera su singoli titoli, perché l'indice stesso incorpora i cambiamenti di composizione. Ma non è zero: la composizione dell'indice cambia nel tempo.

**Costi di transazione sottostimati o assenti.** Un backtest senza costi di transazione è fantasia. Ma anche includere i costi non basta se li si sottostima. Lo slippage dipende dalla liquidità del momento, dalla dimensione dell'ordine, dalla velocità di esecuzione. Su un singolo trade è poco. Su centinaia di trade è la differenza tra un sistema profittevole e uno che perde.

**Selezione del periodo.** Un sistema testato dal 2012 al 2024 sembra avere una storia lunga. Ma se quel periodo è stato prevalentemente rialzista, il sistema potrebbe semplicemente aver imparato a comprare. La vera domanda è: funziona anche in un bear market prolungato? In un mercato laterale di anni? Se il backtest non include almeno un ciclo completo di mercato, non sta dicendo nulla.

**Overfitting dei parametri.** Se si è testato l'RSI a 14 periodi e funziona, ma si sono anche provati 10, 12, 16, 18, 20 e si è scelto 14 perché funziona meglio nel backtest, si è overfittato il parametro. Il vero test sarebbe: la strategia funziona ragionevolmente bene con qualunque parametro RSI tra 10 e 20? Se funziona solo con 14 e crolla con 13 o 15, il risultato è fragile. La robustezza ai parametri è uno dei migliori indicatori di un edge genuino.

## I segnali di allarme

Ci sono dei pattern ricorrenti che segnalano quasi certamente un backtest inaffidabile.

**Equity line troppo bella.** Una curva dei profitti che sale dritta, senza drawdown significativi, senza periodi piatti prolungati, è quasi certamente overfittata. I mercati attraversano regimi diversi, e qualunque sistema reale soffre in almeno alcuni di essi.

**Sharpe ratio sopra 2 senza motivo strutturale.** Uno Sharpe ratio di 2 o superiore su base annuale è eccellente — e rarissimo in modo sostenibile. I migliori hedge fund al mondo hanno Sharpe intorno a 1-1.5 al netto di costi e fee. Se un sistema retail mostra Sharpe di 3 o 4, qualcosa non quadra.

**Nessuna analisi di robustezza.** Se chi presenta il sistema non mostra come variano i risultati cambiando leggermente i parametri, le finestre temporali, o i costi assunti, sta nascondendo qualcosa — forse anche a se stesso.

**Assenza di risultati live.** Un backtest senza track record live verificabile è un romanzo di fantascienza. L'unica prova che un sistema funziona è che ha funzionato — su soldi veri, in tempo reale, per un periodo significativo.

**Troppe regole.** Se un sistema ha 15 condizioni di ingresso, 8 filtri, 5 regole di uscita diverse a seconda del contesto, e 3 eccezioni — è overfittato. Un sistema con 3-4 regole chiare che funziona nel 55% dei casi è infinitamente più affidabile di un sistema con 30 regole che funziona nel 75% dei casi nel backtest.

## Come proteggersi: una checklist pratica

Se si sviluppa un sistema con ML — o se ne valuta uno — queste sono le domande da porsi.

Il sistema funziona su dati che non ha mai visto? Non il test set originale — un periodo completamente nuovo, successivo a tutto lo sviluppo.

I risultati sono robusti ai parametri? Cambia ogni parametro del 10-20% in entrambe le direzioni. Se la performance crolla, l'edge non è reale.

I costi di transazione sono realistici? Includi commissioni, spread, slippage, e aggiungici un margine di sicurezza.

Il periodo di test include regimi diversi? Almeno un bull market, un bear market, e un periodo laterale.

Il numero di trade è sufficiente? Con meno di 100 trade nel test set, la significatività statistica è quasi zero. Con meno di 30, si è nel territorio del lancio di moneta.

Il sistema è semplice? Meno regole, meno parametri, meno feature — meglio è. Se non si riesce a spiegare la logica del sistema in tre frasi, probabilmente è troppo complesso.

Si sono mostrati i risultati a qualcuno di scettico? Il confirmation bias è il peggior nemico. Mostra il sistema a qualcuno che ha interesse a smontarlo.

## La regola d'oro

**Se sembra troppo bello per essere vero, lo è.**

Non "forse lo è". Non "probabilmente lo è". **Lo è.** I mercati sono troppo competitivi, troppo rumorosi, troppo adattivi per produrre edge grandi e facili. Chi promette il contrario non ha capito il problema — o ha capito benissimo il problema e ha deciso che il prodotto da vendere sei tu.

---

# 14. Tirare le fila: un framework per decidere

## Cosa abbiamo costruito

In tredici capitoli abbiamo percorso un arco che va dalle basi del ML fino alla realtà cruda del trading. Ricapitoliamo i punti fermi — non come riassunto, ma come principi operativi da portarsi dietro.

Il Machine Learning è uno strumento per trovare pattern nei dati. Non è intelligenza, non è magia, non è una sfera di cristallo. È statistica applicata con molta potenza di calcolo. Funziona bene quando i dati sono abbondanti, il segnale è forte, e le regole del gioco non cambiano. I mercati finanziari violano tutte e tre queste condizioni, il che rende il ML applicabile ma con aspettative radicalmente ridimensionate rispetto a quello che il marketing racconta.

Il Deep Learning è un sottoinsieme del ML che eccelle su dati non strutturati — immagini, testo, audio — ma non aggiunge valore significativo sui dati tabulari che sono il pane quotidiano del trading. Per uno swing trader su dati giornalieri, il ML classico (Random Forest, XGBoost) è lo strumento giusto: più preciso, più interpretabile, più robusto, e infinitamente più pratico.

L'overfitting è il nemico esistenziale. Nei mercati è più grave che in qualunque altro campo perché il rapporto segnale/rumore è bassissimo, i dati sono pochi rispetto alla complessità del problema, e il sistema che si sta modellando si adatta ai modelli che ci si costruiscono sopra. Qualunque risultato che non è stato validato rigorosamente out-of-sample non vale niente.

Il ML non sostituisce l'esperienza del trader. La amplifica, la sistematizza, la testa — ma non la rimpiazza. Le applicazioni più solide sono il regime detection, il signal filtering e la costruzione di feature composite: tutti usi dove il ML lavora *al servizio* di una logica di trading esistente, non in sostituzione di essa.

## Il framework decisionale

Prima di aggiungere qualunque componente ML al proprio sistema, ci sono cinque domande da porsi in quest'ordine. Se la risposta a una qualunque è no, fermarsi lì.

**Ho un problema specifico da risolvere?** Il ML non è una soluzione in cerca di un problema. "Voglio usare l'AI nel mio trading" non è un problema — è una tentazione. "Voglio capire in quali regimi di mercato la mia strategia sui cicli smette di funzionare" è un problema. "Voglio filtrare i segnali a bassa qualità che mi generano le perdite più frequenti" è un problema.

**Ho abbastanza dati per affrontarlo?** Per un classificatore supervisionato servono almeno qualche centinaio di esempi nella classe minoritaria. Se si vogliono classificare i cicli in "buoni" e "cattivi" e in 20 anni se ne sono identificati 300 buoni e 150 cattivi, si è al limite ma ci si può lavorare. Se sono 30, lasciare perdere.

**Posso validare il risultato in modo onesto?** Bisogna poter mettere da parte un periodo di test che non si tocchi mai durante lo sviluppo. Se si hanno 20 anni di dati, i primi 14 per il training, i successivi 3 per la validazione, e gli ultimi 3 per il test. E bisogna resistere alla tentazione di "aggiustare il modello" dopo aver visto i risultati del test.

**Il beneficio giustifica la complessità?** Ogni componente ML che si aggiunge al sistema è un componente da mantenere. Pipeline di feature, monitoraggio, riaddestramento, debug. Se il filtro ML migliora il profit factor da 1.3 a 1.5, probabilmente vale la complessità. Se lo migliora da 1.3 a 1.35, probabilmente no.

**Sono disposto ad accettare che non funzioni?** Questa è la domanda più importante e quella a cui è più difficile rispondere onestamente. Dopo aver investito settimane o mesi nello sviluppo, la pressione psicologica a "farlo funzionare" è enorme. Ma se i risultati out-of-sample non ci sono, bisogna essere disposti a buttare tutto. Nel trading, l'attaccamento emotivo a un'idea è il modo più veloce per perdere soldi.

## Un percorso concreto

Se le cinque risposte sono sì, ecco un percorso sensato.

**Fase 1 — Regime detection.** Iniziare da qui perché è non supervisionato, non richiede labeling, e produce valore immediato anche senza un modello predittivo. Prendere le feature che descrivono lo stato del mercato e usare K-Means o Gaussian Mixture Models per identificare 3-4 regimi. Poi guardare come la strategia ha performato storicamente in ciascun regime. Se si trova un regime in cui la strategia perde consistentemente, si ha già un filtro operativo.

**Fase 2 — Labeling dei segnali storici.** Questo è il lavoro più noioso e più importante. Prendere tutti i segnali che il sistema ha generato storicamente e classificarli. Non con un criterio meccanico semplice come "ha guadagnato o ha perso" — ma con il giudizio di trader: era un segnale pulito? Il ciclo si è sviluppato come atteso? Questo labeling incorpora l'esperienza nel dataset e sarà la base per il modello di signal filtering.

**Fase 3 — Signal filtering.** Con i segnali etichettati, addestrare un classificatore (Random Forest per iniziare, XGBoost dopo) per prevedere la qualità del segnale a partire dalle feature di contesto. Validare rigorosamente walk-forward. Se il filtro migliora la performance out-of-sample in modo significativo, integrarlo. Se non la migliora, fermarsi.

**Fase 4 — Monitoraggio e iterazione.** Una volta in produzione, monitorare la performance del filtro su finestre mobili. Riaddestrare periodicamente. E tenere sempre attivo il sistema base senza filtro come benchmark — se il filtro ML smette di aggiungere valore, bisogna saperlo e poter tornare indietro senza danni.

## L'ultima parola

Il ML applicato al trading è uno strumento potente nelle mani di chi conosce i mercati e uno strumento pericoloso nelle mani di chi non li conosce. La differenza non è tecnica — è di esperienza, di onestà intellettuale, e di rispetto per l'incertezza.

Chi ha vent'anni di mercati reali, un sistema che gira in produzione da oltre un decennio, una teoria di riferimento testata nel tempo, e la mentalità giusta — quella di chi sa che il mercato ha sempre ragione e che il nostro lavoro è adattarci, non prevederlo — ha un vantaggio raro.

Il ML può rendere quell'adattamento più sistematico, più veloce, più misurabile. Non può renderlo automatico, non può renderlo infallibile, e non può sostituire il giudizio di chi ha pagato il prezzo di imparare dai propri errori.

Chi dice il contrario, sta vendendo qualcosa.
