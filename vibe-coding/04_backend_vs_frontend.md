# 4. Backend vs Frontend: stesso metodo, leve diverse

## Due progetti, una persona

ARIA è composta da due progetti separati. Il backend — lo stack Python con FastAPI, MongoDB, l'engine di analisi finanziaria, l'integrazione con gli LLM — è il territorio che conosco meglio. Ci lavoro da anni, conosco i pattern, le trappole, le ragioni dietro ogni scelta architetturale. Il frontend — Vue 3, Tailwind, JavaScript, l'interfaccia utente con cui il trader interagisce — è il territorio in cui sono meno solido. So leggere il codice, capisco cosa fa, ma non ho la stessa profondità di esperienza che ho sul backend.

Questa differenza di competenza ha prodotto due modi molto diversi di usare Claude Code, pur mantenendo lo stesso metodo di base. I dati delle sessioni lo raccontano con precisione.

## Il backend: l'architetto che detta

Sul backend, diciannove sessioni, cinquantatré dollari di costo. Il pattern dominante è chiaro: arrivo con il problema già analizzato e spesso con la soluzione già disegnata.

La sessione di verifica dei canali è l'esempio più rappresentativo. Il prompt di apertura non chiedeva "implementa i canali" — chiedeva "controlla che questo modulo produca lo stesso risultato dei file Java che sono in questa cartella". Avevo il codice di riferimento in Java, sapevo esattamente cosa doveva uscire, e usavo Claude Code come strumento di verifica incrociata tra due implementazioni. Quando l'analisi ha mostrato che l'algoritmo era corretto ma mancava il dataset combinato, ho potuto valutare immediatamente la proposta di fix perché sapevo cosa significava "canale centrato" vs "canale non centrato" nel contesto dell'analisi ciclica.

In quella stessa sessione, quando è venuto il momento di scegliere un nome per la variabile — `full`, `channel`, `midline`, `values` — la mia risposta è stata immediata. Non perché sappia di naming convention meglio di Claude Code, ma perché sapevo come quella variabile sarebbe stata usata nel resto del sistema. `ch51.channel` si legge bene. `ch51.full` no.

Questa dinamica si ripete in tutte le sessioni backend. Porto i dati — output numerici, stack trace, confronti con implementazioni esistenti — e sfido le conclusioni dello strumento finché non sono verificate. Non accetto un "algoritmo corretto" senza averlo confrontato io stesso con il riferimento. Non accetto una proposta di refactoring senza aver valutato l'impatto sulle firme delle funzioni che conosco.

Il risultato è che sul backend la fase di diagnosi è lunga e conflittuale, ma l'implementazione è veloce. Quando si arriva al "ok, implementa", le decisioni sono già prese e il codice che ne esce è quasi sempre corretto al primo colpo.

## Il frontend: il regista che valuta

Sul frontend, ventuno sessioni, ottantuno dollari di costo. Il costo più alto non è un caso — riflette un modo di lavorare diverso, con più iterazioni, più tentativi, più codice generato e rivisto.

La differenza si vede già nell'apertura delle sessioni. Sul backend, otto sessioni su diciannove iniziano con contesto tecnico preciso — file specifici, dati numerici, confronti algoritmici. Sul frontend, otto sessioni su ventuno iniziano con una richiesta di implementazione diretta. Non perché sia meno rigoroso, ma perché su certi aspetti del frontend non ho la competenza per arrivare con una soluzione già disegnata.

La sessione dell'indicatore del tier utente è rivelatrice. Il prompt iniziale conteneva un'idea — "un bordino del colore del tier sul bottone dell'utente" — ma era un'idea vaga, e io lo sapevo. Ho chiesto esplicitamente a Claude Code di fare una proposta alternativa, cosa che sul backend non faccio quasi mai. Lo strumento ha analizzato il codice, ha valutato che il bordino sul bottone rischiava di sembrare un focus state, e ha proposto il ring colorato sull'avatar. La proposta era migliore della mia idea iniziale, e l'ho riconosciuto.

Ma il metodo non è cambiato. "Per ora non toccare il codice, discutiamo e basta. Think Hard!" — lo schema vincolo-diagnosi era identico al backend. Quello che è cambiato è il contenuto della diagnosi: sul backend sfido i dettagli tecnici perché li conosco, sul frontend valuto le proposte di design perché non ho una soluzione pronta.

## Dove sposto il controllo

La differenza vera non è quanta fiducia do allo strumento — è dove la metto.

Sul backend, il mio controllo è sulla soluzione. Arrivo con il piano, con le firme delle funzioni, con i casi limite. Controllo l'input della fase di implementazione.

Sul frontend, il mio controllo è sulla verifica. Lascio che Claude Code proponga la soluzione, ma poi la esamino con l'occhio di chi conosce il prodotto e l'utente finale. Quando nella sessione dei cicli Claude Code ha proposto di separare le colonne "Trascorso" e "Stimato", io non avevo un'opinione tecnica sulla scelta CSS — ma sapevo che su mobile ogni colonna in più è un problema, e ho chiesto se un wrap responsive potesse risolvere. Quando ha implementato il layout a card per mobile, ho guardato il risultato e ho chiesto modifiche specifiche: la barra di completamento troppo corta, i cicli non separati visivamente a sufficienza, l'ultimo bordo che non chiudeva il contenitore.

Questi non sono interventi tecnici — sono interventi da utilizzatore. Guardo il risultato con gli occhi del trader che userà quella interfaccia e dico cosa non funziona. Non so sempre come correggerlo a livello di codice, ma so cosa non va.

È una forma di controllo diversa, non inferiore. Sul backend controllo il come. Sul frontend controllo il cosa.

## I numeri parlano

I dati confermano questo quadro con una precisione che non mi aspettavo.

"Non toccare" compare sedici volte nel backend e diciassette nel frontend — frequenza quasi identica. Il vincolo è costante, indipendentemente dalla competenza. Questo significa che anche nel territorio dove mi sento meno sicuro, non rinuncio alla fase di diagnosi separata. La tentazione di dire "fai tu" c'è, ma i dati dicono che non ci cado.

"Think Hard" compare tredici volte nel backend e diciassette nel frontend. Sul frontend lo uso di più — il che ha senso: quando non ho una soluzione pronta, ho più bisogno che lo strumento ragioni in profondità prima di agire.

"Implementa" compare undici volte nel backend e tredici nel frontend. Frequenza simile, ma con un significato diverso. Sul backend, l'autorizzazione all'implementazione arriva dopo una diagnosi lunga e una specifica dettagliata. Sul frontend, arriva dopo una discussione più breve ma con un'iterazione post-implementazione più lunga — aggiustamenti visivi, correzioni di layout, dettagli di interazione.

I file toccati raccontano un'altra storia. Sul backend, i file più lavorati sono moduli Python specifici — `cicli.py`, `engine.py`, `settings.py` — ciascuno con una responsabilità chiara. Sul frontend, i file più toccati sono i file di localizzazione — `it.json` e `en.json` compaiono in tredici sessioni su ventuno. Ogni modifica all'interfaccia richiede l'aggiornamento delle stringhe tradotte, il che significa che il costo reale di ogni feature frontend include un overhead sistematico che il backend non ha.

## La trappola della delega

C'è un rischio specifico del lavorare su un dominio in cui si è meno competenti, e sarebbe disonesto non parlarne.

Nella sessione del tier utente, dopo aver concordato il ring sull'avatar, ho lasciato che Claude Code implementasse tutto — store, componente, stile. Il risultato sembrava funzionare. Ma quando ho controllato più a fondo, ho scoperto che lo store non veniva popolato al momento giusto: il campo `userPlan` restava al valore di default "Free" finché l'utente non apriva la pagina Settings. Il ring sarebbe stato sempre verde per tutti, indipendentemente dal tier reale.

Se avessi avuto la stessa profondità di conoscenza che ho sul backend, avrei previsto il problema prima dell'implementazione. Avrei saputo che quello store non era persistente e che il dato non arrivava dal servizio di login. Invece l'ho scoperto dopo, controllando il risultato — che è comunque il metodo che funziona, ma con un costo di iterazione più alto.

La soluzione è stata pragmatica: ho chiesto il rollback completo, sono andato nel backend ad aggiungere il campo `plan` nella risposta del servizio `getUserInfo`, e poi sono tornato nel frontend con il dato disponibile dal punto giusto. Un giro che un esperto frontend avrebbe evitato, ma che il metodo ha comunque intercettato prima che diventasse un bug in produzione.

La lezione è che il metodo — vincolo, diagnosi, verifica — funziona anche quando la competenza è minore, ma il ciclo di feedback è più lungo. Scopri gli errori nella fase di verifica invece che nella fase di diagnosi, il che significa più iterazioni, più tempo, più costo. I numeri lo confermano: ottantuno dollari per ventuno sessioni frontend contro cinquantatré per diciannove sessioni backend. La differenza non è casuale.

## Due modalità, un principio

Se dovessi riassumere la differenza in una frase, direi questo: sul backend sono l'architetto che disegna e delega la costruzione; sul frontend sono il committente che sa cosa vuole, valuta le proposte, e chiede modifiche finché il risultato non lo convince.

Entrambe le modalità funzionano perché il principio di fondo è lo stesso: non delegare le decisioni. Sul backend le decisioni le prendo prima dell'implementazione, perché ho la competenza per farlo. Sul frontend le prendo durante e dopo l'implementazione, perché le riconosco solo quando le vedo realizzate. Ma in nessuno dei due casi lascio che sia lo strumento a decidere al posto mio.

Questo è il quadro quando il metodo funziona. Ma non funziona sempre.
