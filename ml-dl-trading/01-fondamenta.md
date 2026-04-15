# Parte 1 — Le fondamenta

---

# 1. Cos'è il Machine Learning

## Programmare vs imparare

Per capire il Machine Learning bisogna partire da come funziona la programmazione tradizionale, quella che chi sviluppa software conosce da decenni.

Il modello classico è semplice: tu scrivi delle **regole**, dai al programma dei **dati**, e ottieni un **risultato**. Se vuoi un programma che decide se concedere un prestito, scrivi tu le condizioni: reddito sopra X, debiti sotto Y, anzianità lavorativa almeno Z. Il programmatore codifica la logica, il computer la esegue.

Il Machine Learning ribalta questo schema. Tu dai al programma i **dati** e i **risultati** (cioè esempi di decisioni già prese — prestiti concessi e rifiutati, con i relativi profili), e il programma **ricava da solo le regole**. Non le scrivi tu, le "impara" dai dati.

Tradotto nel mondo del trading: immagina di avere uno stagista a cui dici "ecco 10.000 grafici di cicli passati, in questi 6.000 il prezzo è salito, in questi 4.000 è sceso — trovami tu il pattern". Lo stagista non sa nulla di analisi tecnica, non ha una teoria. Ma se è bravo e ha abbastanza esempi, dopo un po' inizia a notare delle regolarità. Magari non le stesse che noterebbe un trader esperto, magari alcune migliori, magari alcune completamente sbagliate. Questo è il ML.

## Tre modi di imparare

Il ML si divide in tre grandi famiglie, e la distinzione è più semplice di quello che sembra.

**Apprendimento supervisionato** — è il caso che ho descritto sopra. Dai al modello degli esempi con l'etichetta giusta: "questo è spam, questa no", "questo cliente ha lasciato, questo è rimasto", "questo trade è andato in profitto, questo no". Il modello impara ad associare le caratteristiche dei dati con le etichette. È il tipo più usato in assoluto, e quello più rilevante per il trading.

**Apprendimento non supervisionato** — qui non ci sono etichette. Dai al modello solo i dati e gli chiedi "trovami dei gruppi, delle strutture, qualcosa che io non vedo". Esempio concreto: dai al modello cinque anni di dati di mercato e gli chiedi di raggruppare i giorni in categorie. Magari ti tira fuori quattro "regimi" — alta volatilità con trend, alta volatilità senza trend, bassa volatilità laterale, bassa volatilità con drift. Non glieli hai detti tu, li ha trovati lui. Può essere molto utile, ma è anche molto più difficile da validare: come fai a sapere se i gruppi che ha trovato hanno senso?

**Apprendimento per rinforzo** — il modello impara per tentativi, ricevendo una "ricompensa" quando fa bene e una "penalità" quando sbaglia. Come addestrare un cane: non gli spieghi la teoria, lo premi quando si siede al comando giusto. È quello che ha reso famosi AlphaGo e i modelli di gioco. Nel trading viene usato, ma con problemi enormi di cui parleremo.

## Cosa vuol dire "modello"

La parola "modello" nel ML è meno mistica di quanto sembri. Un modello è semplicemente una funzione matematica — un macchinario che prende dei numeri in ingresso e produce dei numeri in uscita. Prima dell'addestramento, questa funzione produce risultati casuali. Durante l'addestramento, i suoi parametri interni vengono aggiustati un po' alla volta finché i risultati non diventano ragionevolmente buoni sugli esempi che gli hai dato.

Pensa a un equalizzatore audio con cento manopole. All'inizio le manopole sono messe a caso e il suono è orribile. L'addestramento è il processo di girare quelle manopole, un po' alla volta, finché il suono non diventa buono. Le "manopole" sono i parametri del modello. Un modello semplice ne ha decine. Un modello di linguaggio come quelli alla base dei chatbot moderni ne ha miliardi.

## Feature: il vero lavoro

Un concetto fondamentale è quello delle **feature**, ovvero le caratteristiche dei dati che decidi di dare in pasto al modello. Questa è la parte in cui l'esperienza umana conta più di qualunque algoritmo.

Se vuoi prevedere se un titolo salirà nei prossimi 5 giorni, devi decidere *cosa* mostrare al modello. Il prezzo di chiusura degli ultimi 20 giorni? I volumi? La volatilità? Il rapporto prezzo/utili? L'RSI a 14 periodi? La posizione nel canale? Lo spread tra BTP e Bund?

Questa scelta si chiama **feature engineering**, e nel ML classico è dove si vince o si perde la partita. Un modello con feature scelte bene e un algoritmo semplice batte quasi sempre un modello con feature scelte male e un algoritmo sofisticatissimo. È un po' come la fotografia: la macchina fotografica conta, ma il fotografo conta molto di più.

## Il processo in sintesi

Il ciclo di lavoro del ML, semplificato al massimo:

1. **Raccogli i dati** e puliscili (il 70-80% del lavoro reale, sempre)
2. **Scegli le feature** — decidi cosa è rilevante
3. **Scegli un algoritmo** — Random Forest, SVM, regressione logistica, ecc.
4. **Addestra il modello** sui dati storici
5. **Valida** su dati che il modello non ha mai visto
6. **Metti in produzione** e monitora se continua a funzionare

Il punto 5 è quello che distingue un lavoro serio da una truffa, e ci dedicheremo un capitolo intero. Per ora basta sapere che un modello che funziona sui dati storici ma non su dati nuovi non vale niente. Questo si chiama **overfitting**, ed è il peccato originale del ML applicato ai mercati.

---

# 2. Cos'è il Deep Learning

## Dal ML classico alle reti neurali

Nel capitolo precedente abbiamo detto che il Machine Learning è un macchinario che impara le regole dai dati. Il Deep Learning è un sottoinsieme del ML — non qualcosa di diverso, ma un modo specifico di costruire quel macchinario. Il mattone fondamentale è la **rete neurale artificiale**.

Il nome è suggestivo e un po' fuorviante. Le reti neurali si ispirano *vagamente* al cervello umano, nel senso che sono fatte di tanti piccoli nodi collegati tra loro, come i neuroni sono collegati dalle sinapsi. Ma il paragone finisce lì. Nessuno oggi sostiene seriamente che funzionino come un cervello. Sono un costrutto matematico che si è rivelato straordinariamente efficace per certi compiti.

## Come funziona una rete neurale, senza equazioni

Immagina una catena di montaggio con diversi reparti. I dati grezzi entrano dal primo reparto (**layer di input**). Ogni reparto successivo (**layer nascosto**) prende il lavoro del reparto precedente, lo trasforma un po', e lo passa avanti. L'ultimo reparto (**layer di output**) produce il risultato finale.

Ogni reparto ha degli operai, e ogni operaio fa un lavoro semplicissimo: prende dei numeri in ingresso, li pesa (cioè dà più importanza ad alcuni e meno ad altri), li somma, e decide se passare avanti il risultato oppure no. Tutto qui. La magia sta nel fatto che milioni di operai che fanno questo lavoro banale, organizzati su molti reparti in sequenza, riescono a catturare relazioni nei dati che sono incredibilmente complesse.

La parola **deep** — profondo — si riferisce proprio al numero di reparti, cioè di layer. Una rete con 2-3 layer è una rete neurale "classica", roba degli anni '80. Una rete con decine o centinaia di layer è una rete **profonda**, ed è quello che ha cambiato tutto a partire dal 2012 circa.

## Cosa è cambiato: la rivoluzione della profondità

Le reti neurali esistono dagli anni '50. Per decenni sono rimaste una curiosità accademica perché erano lente da addestrare, servivano troppi dati, e i risultati non giustificavano lo sforzo. Tre cose sono cambiate quasi contemporaneamente:

**I dati sono esplosi.** Internet, smartphone, sensori, social media — improvvisamente c'erano quantità di dati impensabili anche solo dieci anni prima. E le reti neurali profonde, a differenza degli algoritmi classici, continuano a migliorare man mano che gli dai più dati. Non hanno un tetto visibile.

**Il calcolo è diventato accessibile.** Le GPU, nate per i videogiochi, si sono rivelate perfette per i calcoli paralleli delle reti neurali. Quello che prima richiedeva settimane su un supercomputer ora richiede ore su una scheda grafica da gaming.

**Qualche idea intelligente.** Tecniche come il dropout, la batch normalization, e architetture più furbe (ResNet, che ha permesso di addestrare reti con centinaia di layer senza che il segnale si perdesse strada facendo) hanno risolto problemi pratici che bloccavano tutto.

Il momento simbolico è il 2012: una rete neurale profonda chiamata AlexNet vince la competizione ImageNet per il riconoscimento di immagini, stracciando tutti gli approcci tradizionali. Da lì in poi è stata una valanga.

## La differenza chiave: le feature le trova da solo

Questo è il punto più importante per capire la differenza tra ML classico e DL, e vale la pena ripeterlo.

Nel **ML classico**, sei tu a decidere quali caratteristiche dei dati sono rilevanti. Se vuoi riconoscere un gatto in una foto, devi dire tu al modello di guardare i bordi, le forme, i colori, le texture. Poi il modello usa quelle feature per classificare. Se scegli le feature sbagliate, il modello non funziona, punto.

Nel **Deep Learning**, dai al modello i dati grezzi — i pixel della foto, il testo grezzo, l'audio grezzo — e i layer successivi della rete imparano da soli a estrarre feature sempre più astratte. Il primo layer impara a riconoscere bordi e contrasti. Il secondo combina i bordi in forme semplici. Il terzo riconosce orecchie, occhi, baffi. Il quarto mette insieme il tutto e dice "gatto".

Nessuno ha programmato queste feature. La rete le ha scoperte da sola, perché quella organizzazione dei dati è quella che minimizza gli errori sugli esempi di addestramento.

Questo è potentissimo quando funziona. Ma ha un costo: servono molti più dati e molta più potenza di calcolo. E — punto cruciale — diventa molto più difficile capire *perché* il modello ha preso una certa decisione. Con un Random Forest puoi andare a guardare quali feature hanno contato di più. Con una rete profonda da 100 milioni di parametri, buona fortuna.

## Dove il DL ha dominato e dove no

Il Deep Learning ha letteralmente rivoluzionato alcuni campi:

**Visione artificiale** — riconoscimento immagini, guida autonoma, diagnostica medica da radiografie. Prima del 2012 era un problema semi-irrisolto. Oggi le macchine superano gli umani in diversi task specifici.

**Linguaggio naturale** — traduzione, riassunti, chatbot, e poi i modelli generativi. L'architettura Transformer (2017) ha reso possibile quello che fino a pochi anni fa sembrava fantascienza.

**Audio e voce** — riconoscimento vocale, sintesi vocale, generazione musicale.

**Giochi** — AlphaGo, AlphaZero, i bot di Dota e StarCraft. Qui l'apprendimento per rinforzo combinato con il DL ha prodotto risultati spettacolari.

Ma ci sono campi dove il DL **non** ha spazzato via il ML classico, e questo è un punto che spesso viene omesso dai venditori di hype. In molti problemi tabulari — dati organizzati in righe e colonne, come un database o un foglio Excel — un buon Random Forest o un XGBoost batte ancora regolarmente le reti neurali profonde, con meno dati, meno tempo e risultati più interpretabili.

Questo è molto rilevante per il trading, perché i dati di mercato sono in larga parte tabulari.

## Un riassunto delle differenze

Il **ML classico** richiede feature engineering manuale, funziona bene con dataset piccoli o medi (migliaia o decine di migliaia di esempi), si addestra su un computer normale in minuti o ore, e produce modelli che puoi ispezionare e capire. Funziona molto bene su dati strutturati — tabelle, numeri, categorie.

Il **Deep Learning** impara le feature da solo dai dati grezzi, ma ha bisogno di dataset grandi o enormi (centinaia di migliaia, milioni di esempi), richiede GPU e ore o giorni di addestramento, e produce modelli che sono in larga parte scatole nere. Domina su dati non strutturati — immagini, testo, audio, video.

Non è che uno sia "meglio" dell'altro. Sono strumenti diversi per problemi diversi. Un chirurgo non usa il bisturi per tutto — a volte serve la sega, a volte basta un cerotto. Il problema è che il marketing ha convinto il pubblico che "deep learning" sia sinonimo di "intelligenza artificiale avanzata" e che il ML classico sia roba vecchia. Non è così. Nel mondo del trading — dati di mercato, feature numeriche, dataset non enormi — il ML classico è spesso la scelta più sensata.

---

# 3. Come si addestra un modello — e perché quasi tutti sbagliano

## Il problema di fondo

Addestrare un modello di ML significa una cosa sola: trovare i valori dei parametri interni che producono le previsioni migliori. Nel capitolo 1 abbiamo usato la metafora dell'equalizzatore con cento manopole. Ora andiamo un passo più in là.

Il modello parte con parametri casuali. Gli mostri un esempio dei dati di addestramento — diciamo un set di feature di mercato e il risultato reale (il prezzo è salito). Il modello fa la sua previsione (magari dice "scende"). Sbaglia. A questo punto si calcola **quanto** ha sbagliato — questa misura si chiama **loss function**, funzione di errore. Poi i parametri vengono aggiustati leggermente nella direzione che riduce l'errore. Si ripete con l'esempio successivo. E il successivo. E così via, migliaia o milioni di volte.

È come un arciere bendato che tira una freccia, qualcuno gli dice "troppo a destra e troppo in alto", lui corregge un po', tira di nuovo, e così via finché non si avvicina al centro. Non vede il bersaglio, ma ad ogni tiro riceve un feedback su direzione e distanza dell'errore.

Questo processo si chiama **ottimizzazione**, e nella sua forma più comune **gradient descent** — discesa del gradiente. Non serve capire la matematica che ci sta dietro. Serve capire che il modello non "capisce" i dati: cerca meccanicamente la configurazione di parametri che minimizza gli errori sugli esempi che gli hai dato. Niente di più, niente di meno.

## Il nemico: l'overfitting

Ed eccoci al concetto più importante di tutto il corso. Se c'è una sola cosa da portarsi a casa, è questa.

Un modello che funziona perfettamente sui dati di addestramento ma fa schifo su dati nuovi si dice **overfittato**. Ha memorizzato gli esempi invece di imparare le regole generali. È lo studente che impara a memoria le risposte del compito dell'anno scorso: prende 10 se il professore rimette le stesse domande, prende 2 se le cambia.

Un esempio concreto. Prendi 10 anni di dati giornalieri del FTSE MIB e addestra un modello per prevedere la direzione del giorno dopo. Se il modello è abbastanza complesso (cioè ha abbastanza parametri — abbastanza "manopole"), può arrivare a prevedere correttamente il 95% dei giorni nel dataset di addestramento. Sembra fantastico. Poi lo provi sull'anno successivo e il tasso di successo crolla al 51%. Praticamente un lancio di moneta.

Cosa è successo? Il modello ha trovato dei pattern nei dati storici che erano **rumore**, non segnale. Ha imparato che "il 14 marzo 2018, dopo una sequenza esatta di questi 5 prezzi, il mercato è salito" — un'informazione che è vera ma completamente inutile, perché quella sequenza esatta non si ripresenterà mai. Ha memorizzato le coincidenze del passato scambiandole per regole del futuro.

L'overfitting è un problema universale del ML, ma nei mercati finanziari è **devastante** per una ragione specifica: i mercati sono un sistema in cui il rapporto segnale/rumore è bassissimo. In un problema di riconoscimento immagini, un gatto è un gatto — le orecchie a punta, i baffi, la forma del muso sono segnali stabili e forti. Nei mercati, il segnale vero è sepolto sotto montagne di rumore casuale, e per di più il segnale stesso cambia nel tempo perché il mercato è un sistema adattivo: quando abbastanza persone scoprono un pattern, lo sfruttano, e il pattern sparisce.

## Come ci si difende: train, validation, test

La difesa standard contro l'overfitting è semplice nel principio e rigorosa nell'esecuzione: non valutare mai il modello sugli stessi dati che ha usato per imparare.

Si dividono i dati in tre gruppi:

**Training set** — tipicamente il 60-70% dei dati. È quello su cui il modello impara, dove i parametri vengono aggiustati.

**Validation set** — il 15-20%. Serve durante lo sviluppo per scegliere tra modelli diversi, regolare gli iperparametri (scelte di design come la complessità del modello), e capire quando fermarsi. Il modello non impara da questi dati, ma tu li usi per prendere decisioni sul modello, quindi c'è un rischio indiretto di adattamento.

**Test set** — il 15-20% restante. Si tocca una volta sola, alla fine di tutto, per avere una stima onesta di come il modello si comporterà nel mondo reale. Se lo usi più volte, stai barando con te stesso — e il test set diventa di fatto un secondo validation set.

Nel trading c'è una regola aggiuntiva fondamentale: la divisione deve rispettare il tempo. Non puoi mescolare i dati a caso e poi dividere, perché useresti il futuro per prevedere il passato. Il training set deve venire **prima** nel tempo, il validation **dopo**, e il test set deve essere il periodo più recente. Questo si chiama **walk-forward validation** e chi non lo fa sta ingannando se stesso.

## Perché il ML classico batte il Deep Learning sui dati tabulari

Adesso hai gli strumenti per capire questo punto, che è tutt'altro che marginale.

Quando si parla di "dati tabulari" si intendono dati organizzati in righe e colonne — esattamente come quelli in un sistema di trading. Ogni riga è un'osservazione (un giorno, un trade, un segnale), ogni colonna è una feature (prezzo, volume, volatilità, posizione nel canale, spread, ecc.). Questo è il formato di dati dominante nel trading, nella finanza, nel business in generale.

Dire che il ML classico "è meglio" del DL su questi dati significa diverse cose concrete.

**Accuratezza delle previsioni.** Questo è il punto che sorprende di più. Su dati tabulari, algoritmi come XGBoost, LightGBM e Random Forest battono regolarmente le reti neurali profonde nelle competizioni e nei benchmark. Non è un'opinione: nel 2022 un paper molto citato di Grinsztajn, Oyallon e Varoquaux ha testato sistematicamente decine di dataset tabulari e il risultato è stato netto. I modelli ad albero vincono o pareggiano nella stragrande maggioranza dei casi. Altri studi hanno confermato risultati simili. Non è che il DL non funzioni — è che non aggiunge nulla per giustificare la complessità in più.

**Perché succede?** Il motivo è legato alla natura dei dati stessi. Le reti neurali profonde eccellono dove c'è una struttura spaziale o sequenziale nei dati grezzi — i pixel di un'immagine hanno relazioni di vicinanza, le parole di una frase hanno relazioni di ordine. Lì la capacità del DL di scoprire feature gerarchiche (bordi → forme → oggetti) è imbattibile. Ma in una tabella, le colonne non hanno una struttura spaziale intrinseca: la colonna "volume" non è "vicina" alla colonna "RSI" in nessun senso fisico. Mettere il volume nella colonna 3 o nella colonna 7 non cambia nulla. I modelli ad albero sono progettati esattamente per questo tipo di dati: fanno split su singole feature, le combinano naturalmente, e gestiscono bene le relazioni non lineari tra colonne senza bisogno che queste abbiano una struttura geometrica.

**Quantità di dati necessaria.** Una rete profonda ha milioni di parametri da regolare. Per farlo bene, servono molti dati — centinaia di migliaia o milioni di esempi. Nel riconoscimento immagini questo non è un problema: puoi generare milioni di foto di gatti. Nel trading, i dati sono quelli che sono. Se si tradano cicli di 4-8 giorni su un indice, in 20 anni ci sono circa 5.000 giornate di trading. Non 5 milioni, 5.000. Con 5.000 esempi un XGBoost può imparare qualcosa di utile. Una rete profonda con milioni di parametri farà overfitting come se non ci fosse un domani — troppe manopole, troppo pochi punti di riferimento.

**Velocità di addestramento e iterazione.** Un Random Forest o un XGBoost si addestra in secondi o minuti su un portatile normale. Una rete neurale profonda richiede GPU e ore. Questo sembra un dettaglio pratico, ma ha implicazioni enormi: nel ML applicato, la velocità di iterazione è tutto. Vuoi poter provare 50 combinazioni di feature, 10 configurazioni diverse, 5 finestre temporali — in un pomeriggio. Con il ML classico puoi farlo. Con il DL no, a meno di avere un cluster di GPU.

**Interpretabilità.** Un Random Forest ti dice: "questa previsione è 'rialzo' perché il volume è alto, il prezzo è nella parte bassa del canale, e la volatilità è in contrazione — e questi tre fattori pesano in questo ordine". Puoi guardare il modello e chiederti "ha senso?". Se il modello ti dice che il fattore più importante è il giorno della settimana, alzi un sopracciglio e indaghi. Con una rete profonda, hai un mucchio di matrici di numeri e sostanzialmente nessuna possibilità di capire il ragionamento. Nel trading, dove si rischiano soldi veri, sapere *perché* il modello ha preso una decisione non è un lusso — è una necessità. Ti permette di distinguere un pattern reale da un artefatto dell'addestramento, e ti dà la fiducia per seguire il segnale quando il mercato va contro.

**Robustezza.** I modelli ad albero sono naturalmente meno sensibili a variabili irrilevanti, a valori anomali, e a feature con scale diverse. Le reti neurali richiedono una preparazione dei dati molto più attenta — normalizzazione, gestione dei valori mancanti, bilanciamento delle classi — e sono più fragili quando qualcosa nei dati cambia. Nei mercati, qualcosa nei dati cambia *sempre*.

Messo tutto insieme, il quadro è chiaro: per dati tabulari con dataset di dimensioni normali (non enormi), i modelli di ML classico sono più precisi, più veloci, più comprensibili, più robusti, e più facili da mettere in produzione. Non è nostalgia per il passato — è la realtà dei benchmark e della pratica quotidiana.

Questo non significa che il DL sia inutile nel trading. Ha applicazioni specifiche — ad esempio nell'analisi di testo per il sentiment dalle news, o nel processing di dati ad altissima frequenza dove le sequenze temporali sono molto lunghe. Ma per lo swing trading su dati giornalieri o settimanali, con feature numeriche costruite a mano da chi conosce il mercato, il ML classico è lo strumento giusto. Non il più figo, non il più nuovo — il più adatto.

---

# 4. Loss function, bias-variance, e come impedire al modello di barare

## La loss function: cosa stai ottimizzando conta più di come lo ottimizzi

Nel capitolo precedente abbiamo detto che l'addestramento consiste nel minimizzare una funzione di errore. Questo sembra un dettaglio tecnico, ma la scelta di *quale* errore minimizzare cambia radicalmente il comportamento del modello.

Facciamo un esempio fuori dal trading per rendere chiaro il concetto. Immagina di addestrare un modello per stimare il prezzo delle case. Puoi scegliere di minimizzare l'**errore medio assoluto** — cioè "in media, di quanti euro sbagli?". Oppure puoi minimizzare l'**errore quadratico medio** — che è simile, ma penalizza molto di più gli errori grandi. Con la prima scelta, il modello si impegna a essere ragionevolmente preciso su tutte le case. Con la seconda, il modello fa di tutto per evitare le cantonate grosse, a costo di sbagliare un po' di più su quelle facili.

Nessuna delle due è "giusta" in assoluto. Dipende da cosa ti importa.

Nel trading, questa scelta diventa ancora più critica perché la loss function standard del ML non è allineata con quello che vuoi davvero. Un modello di classificazione che prevede "sale" o "scende" cerca di massimizzare la percentuale di previsioni corrette. Ma a un trader non interessa avere ragione il 60% delle volte se il 40% di volte che sbaglia perde il triplo di quello che guadagna quando ha ragione. Quello che interessa è il rendimento aggiustato per il rischio — che è una cosa completamente diversa dalla percentuale di accuracy.

Questo disallineamento tra "quello che il modello ottimizza" e "quello che serve nella pratica" è una delle trappole più insidiose. Ci torneremo nella parte sul trading, perché è un punto dove molti sistemi automatici falliscono senza che il loro creatore capisca perché.

## Bias e variance: troppo semplice vs troppo complesso

C'è una tensione fondamentale in qualsiasi modello di ML, e capirla è essenziale per non farsi del male.

**Bias alto** significa che il modello è troppo semplice per catturare i pattern nei dati. Immagina di provare a descrivere il profilo delle Dolomiti con una linea retta. Per quanto aggiusti la pendenza, una retta non può seguire quelle curve. Il modello "non ci arriva" — sottostima la complessità del fenomeno. Questo si chiama **underfitting**.

**Varianza alta** significa che il modello è troppo sensibile ai dati specifici che ha visto. Immagina di descrivere il profilo delle Dolomiti passando esattamente per ogni singolo punto di misurazione, compresi gli errori del GPS. Il profilo risultante è frastagliato in modo assurdo, pieno di picchi e valli che non esistono nella realtà — sono artefatti della misurazione. Questo è l'**overfitting** che abbiamo già visto.

Il punto chiave è che non puoi eliminare entrambi contemporaneamente. Riduci il bias (rendi il modello più complesso) e la varianza aumenta. Riduci la varianza (semplifichi il modello) e il bias aumenta. Questo si chiama **bias-variance tradeoff** ed è una legge di natura del ML, non un problema risolvibile. Puoi solo cercare il punto di equilibrio migliore per il tuo problema specifico.

Nel trading, la tentazione è sempre verso la varianza alta — modelli troppo complessi che si adattano al rumore. Perché? Perché un modello complesso produce backtest bellissimi. I risultati sul training set sono spettacolari, i grafici dell'equity line salgono dritti come un fuso, e l'ego del creatore è soddisfatto. Poi arriva il mercato reale e la magia finisce.

## Regolarizzazione: le catene che salvano il modello da se stesso

La regolarizzazione è l'insieme di tecniche che impediscono al modello di diventare troppo complesso. È come mettere un limite di velocità: il modello potrebbe andare più forte, ma lo costringi a rallentare perché sai che a quella velocità finisce fuori strada.

Le forme più comuni, senza entrare nella matematica:

**Penalità sulla complessità.** Si aggiunge alla loss function un termine che cresce quando i parametri del modello diventano troppo grandi. In pratica si dice al modello: "sì, minimizza gli errori, ma fallo con parametri piccoli — non esagerare con nessun singolo fattore". Questo impedisce al modello di assegnare un peso enorme a una feature che per puro caso correla bene nel training set ma non ha valore predittivo reale. Nel linguaggio tecnico si chiamano regolarizzazione L1 e L2, ma il concetto è lo stesso: penalizzare l'eccesso.

**Early stopping.** Durante l'addestramento, si monitora l'errore non sul training set ma sul validation set. All'inizio entrambi gli errori scendono — il modello sta imparando pattern veri. A un certo punto, l'errore sul training continua a scendere ma quello sul validation inizia a risalire. Quello è il momento di fermarsi: da lì in poi il modello sta memorizzando, non imparando. È come cuocere la pasta: c'è un momento preciso in cui è pronta, e se la lasci troppo scuoce.

**Riduzione delle feature.** A volte il modo migliore per evitare l'overfitting è dare meno informazioni al modello, non di più. Se hai 200 feature e solo 20 sono davvero rilevanti, le altre 180 sono rumore che il modello cercherà di sfruttare trovandoci pattern inesistenti. Selezionare le feature giuste — eliminare il rumore prima che il modello lo veda — è una forma di regolarizzazione brutale ma efficacissima.

**Ensemble e bagging.** Invece di addestrare un solo modello, ne addestri molti, ciascuno su un sottoinsieme leggermente diverso dei dati, e poi prendi la media delle loro previsioni. I singoli modelli overfittano ciascuno a modo suo, ma gli errori tendono a cancellarsi nella media. Random Forest funziona esattamente così: è una foresta di alberi decisionali, ognuno imperfetto, ma la foresta nel suo insieme è più stabile di qualsiasi singolo albero. È lo stesso principio per cui la media delle stime di una folla è spesso più precisa della stima del singolo esperto — la cosiddetta "saggezza della folla".

## Il test finale: out-of-sample

Tutte queste tecniche servono a una cosa sola: far sì che il modello funzioni su dati che non ha mai visto. Il giudice ultimo è sempre la performance **out-of-sample** — fuori dal campione di addestramento.

Nel ML applicato ai mercati, questo è il momento della verità. Puoi aver scelto feature intelligenti, usato la regolarizzazione, fatto cross-validation impeccabile — ma se il modello non guadagna (o non produce segnali utili) su un periodo di test che non ha mai visto, non vale niente.

E qui c'è un'onestà intellettuale che pochi nel settore praticano: se testi il modello su 10 periodi diversi e ne mostri solo i 3 in cui ha funzionato, stai facendo overfitting sul test set. È una forma subdola dello stesso problema — non è il modello che overfittà, sei tu che scegli i risultati. Questo accade molto più spesso di quanto si ammetta, ed è il motivo per cui la maggior parte dei backtest pubblicati online non valgono la carta su cui sono stampati.

---

# 5. Misurare un modello — perché l'accuracy è una bugia

## Il problema del numero singolo

La tentazione più forte nel ML è ridurre la bontà di un modello a un solo numero. "Il mio modello ha il 75% di accuracy!" Sembra chiaro, sembra oggettivo, e sembra sufficiente. Non lo è quasi mai.

Partiamo da un esempio che rende il punto immediatamente. Immagina di voler prevedere se domani il mercato farà un movimento estremo — diciamo un calo superiore al 3%. Negli ultimi 20 anni di FTSE MIB, questo è successo in circa il 2% delle giornate. Ora, costruisco un "modello" semplicissimo: dice sempre "no, domani non ci sarà un calo del 3%". Sempre, qualunque cosa succeda. Questo modello idiota ha un'accuracy del 98%. Fantastico, no?

Ovviamente no. Il modello non ha imparato nulla — ignora completamente i casi che ti interessano. Ma se valuti solo l'accuracy, sembra un campione. Questo problema si chiama **class imbalance** — sbilanciamento delle classi — ed è endemico nel trading, dove gli eventi che vuoi prevedere (inversioni, breakout, crash) sono per definizione rari rispetto ai giorni "normali".

## Le metriche che servono davvero

Per capire se un modello funziona, devi guardare almeno quattro cose, e devi guardarle insieme.

**Precision** — di tutte le volte che il modello dice "sì, compra", quante volte aveva ragione? Se il modello genera 100 segnali di acquisto e 60 portano a un profitto, la precision è del 60%. Questa metrica ti dice quanto puoi fidarti quando il modello parla. Una precision bassa significa tanti falsi allarmi — e nel trading, ogni falso allarme costa commissioni e slippage.

**Recall** — di tutti i movimenti favorevoli che *tu hai definito come operabili*, quanti il modello li ha effettivamente catturati? Questa metrica ti dice quanto il modello è completo. Un recall basso significa opportunità perse — ma nel trading, perdere un'opportunità non costa nulla, mentre entrare su un segnale falso sì. Questa asimmetria è fondamentale.

Nota importante: il recall si misura sempre rispetto ai *tuoi* positivi — cioè rispetto a come hai etichettato i dati. Se hai correttamente etichettato come "non prendere" un movimento rialzista che non rispetta i tuoi criteri di ingresso (setup non pulito, posizione nel canale sbagliata, volumi incoerenti), quel movimento per il modello è un negativo. Se il modello lo ignora, sta facendo il suo lavoro. Se lo prende, è un falso positivo — un errore di precision, non di recall. Questo è esattamente il motivo per cui la fase di **labeling** — decidere cosa è un positivo e cosa no — è forse il lavoro più importante e più sottovalutato di tutto il processo. Il modello può solo essere buono quanto le etichette che gli dai. Se etichetti come positivo qualunque giorno in cui il prezzo è salito, stai insegnando al modello a essere superficiale. Se etichetti come positivo solo i movimenti che avresti preso tu, con i tuoi criteri, stai trasferendo al modello il tuo giudizio di trader. La qualità dell'etichettatura è la qualità del modello — garbage in, garbage out.

**F1 score** — è una media armonica tra precision e recall, un modo per bilanciare le due. È utile come numero di sintesi, ma nasconde il tradeoff tra le due metriche, che nel trading vuoi gestire consapevolmente.

**Matrice di confusione** — è una tabella 2x2 che mostra tutte le combinazioni: quante volte il modello ha detto "sale" ed è salito (vero positivo), ha detto "sale" ed è sceso (falso positivo), ha detto "scende" ed è sceso (vero negativo), ha detto "scende" ed è salito (falso negativo). Guardare questa tabella ti dà un'immagine immediata di dove il modello sbaglia e di che tipo di errori fa. E nel trading, i tipi di errore hanno costi molto diversi.

## Precision vs recall: il tradeoff che cambia tutto

Questi due numeri sono in tensione tra loro, e dove metti il cursore dipende interamente dal tuo stile di trading.

Se alzi la soglia del modello — cioè gli dici "segnalami un'operazione solo quando sei molto sicuro" — la precision sale (meno falsi allarmi) ma il recall scende (perdi più opportunità). Se abbassi la soglia — "segnalami anche i casi dubbi" — il recall sale ma la precision crolla.

Per uno swing trader che fa poche operazioni mirate su cicli di diversi giorni, la precision è quasi sempre più importante del recall. Preferisci fare 20 operazioni all'anno con il 65% di successo piuttosto che 200 operazioni con il 52%. Meno rumore, più segnale, meno costi di transazione, meno stress.

Per un sistema ad alta frequenza che fa migliaia di operazioni al giorno, il recall diventa più importante: puoi permetterti una precision più bassa perché il vantaggio statistico si manifesta sul volume.

Questa è una scelta di design, non un parametro tecnico. E se chi ti vende un sistema non ti spiega dove ha messo il cursore e perché, non sa cosa sta facendo — oppure lo sa e non te lo dice.

## Metriche da ML vs metriche da trading

C'è un ultimo punto che collega questo capitolo alla parte sul trading. Le metriche che abbiamo visto — precision, recall, F1, accuracy — sono metriche **del modello**. Misurano quanto bene il modello fa previsioni. Ma nel trading, la previsione è solo il primo passo. Quello che conta è il **risultato economico** della strategia costruita su quelle previsioni.

Un modello con il 55% di accuracy che entra sempre con lo stop loss stretto e il target largo può essere molto profittevole. Un modello con il 70% di accuracy che entra su ogni segnale senza gestione del rischio può distruggere il conto. Le metriche che contano alla fine sono altre: lo Sharpe ratio, il maximum drawdown, il profit factor, il rapporto rendimento/rischio per trade — tutte cose che un trader esperto conosce molto meglio di qualunque data scientist.

Questo è il ponte fondamentale tra il mondo del ML e il mondo del trading reale, ed è dove la maggior parte dei progetti fallisce. Il data scientist ottimizza l'accuracy. Il trader ottimizza il rendimento aggiustato per il rischio. Se i due non parlano la stessa lingua — o se sono la stessa persona ma con il cappello sbagliato — il risultato è un modello che funziona da manuale ma perde soldi.

---
