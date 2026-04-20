# 5. Quando il metodo salta

## La comfort zone ha un confine

Finora ho raccontato un metodo che funziona. Funziona sul backend dove ho esperienza profonda, funziona sul frontend dove ho meno esperienza ma compenso con un controllo diverso. La tentazione sarebbe fermarsi qui e chiudere il libro con un messaggio rassicurante: segui queste cinque fasi e tutto andrà bene.

Ma non sarebbe onesto. Il metodo ha un limite preciso, e quel limite coincide con il confine della competenza. Non la competenza nel senso generico di "saper programmare" — la competenza specifica nel dominio del problema. Quando esci dalla tua comfort zone e entri in un territorio che non conosci abbastanza da poter sfidare le risposte dello strumento, il metodo si indebolisce. E quando si indebolisce, le cose vanno male.

## Il caso Qdrant

Il momento più istruttivo dell'intero progetto è stato il deployment di Qdrant sul server di produzione. Non era programmazione — era devops: file systemd, Docker, permessi del filesystem, cgroup. Un territorio in cui non ho la stessa padronanza che ho su Python o sull'architettura del backend.

La sessione è cominciata bene. Ho chiesto a Claude Code come configurare il servizio, ho ricevuto un file systemd, e l'ho deployato. Poi è cominciato il disastro.

Il file conteneva opzioni di hardening — `ProtectSystem=strict`, `PrivateTmp=true`, `NoNewPrivileges=true` — che Claude Code aveva inserito perché sono best practice per i servizi systemd. Il problema è che queste opzioni si applicano al processo `docker run`, non al container, e impediscono a Docker di accedere al socket e al filesystem di sistema di cui ha bisogno per creare il container. Il servizio non partiva.

Poi c'erano le opzioni per limitare CPU e memoria — `--cpus` e `--memory` nel `docker run`. Anche queste ragionevoli in teoria, ma che nella versione specifica di Docker sul mio server non funzionavano come atteso. Rimuovendole, il servizio partiva. Rimettendole, si rompeva.

E poi c'erano gli spazi invisibili dopo i backslash di continuazione nelle righe del file systemd. Un errore banale, quasi invisibile a occhio nudo, che produceva un criptico "invalid reference format" che non puntava a nessuna riga specifica.

Tre problemi sovrapposti, ciascuno sufficiente a impedire il funzionamento del servizio, e io non avevo la competenza per isolarli rapidamente. In un contesto che conosco — un bug nel calcolo dei canali, un errore nella logica del ciclo — avrei separato le cause in cinque minuti. Qui ho perso quasi un'ora a provare, fallire, imprecare, e provare ancora.

## Cosa è andato storto nel metodo

La cosa interessante non è che il servizio non partisse — i problemi tecnici capitano. La cosa interessante è come il mio comportamento in quella sessione era diverso da tutte le altre.

Non ho usato "non toccare il codice". Non ho forzato una fase di diagnosi separata dall'azione. Non ho scritto "Think Hard" prima di chiedere l'implementazione. Ho preso il file systemd proposto da Claude Code e l'ho deployato direttamente, fidandomi che fosse corretto.

In altre parole, ho saltato il metodo. Ho saltato le fasi che in tutte le altre sessioni mi avevano protetto dagli errori — il vincolo, la diagnosi, la verifica prima dell'azione. L'ho fatto perché non avevo le conoscenze per condurre quella diagnosi. Non sapevo quali domande fare. Non sapevo cosa controllare. E siccome non sapevo cosa non sapevo, ho fatto la cosa più naturale e più pericolosa: mi sono fidato.

Se confronto questa sessione con quella sui canali — dove avevo il codice Java di riferimento, sapevo esattamente cosa doveva uscire, e ho sfidato ogni singola conclusione di Claude Code — la differenza è evidente. Sui canali ero l'architetto che verifica. Su Qdrant ero l'utente che spera.

## L'errore nella documentazione

C'è un altro episodio che vale la pena raccontare, perché mostra un fallimento diverso — non del metodo, ma dello strumento, con conseguenze aggravate dalla mia mancanza di verifica.

Quando ho chiesto a Claude Code di aggiornare la documentazione STT con i servizi admin, lo strumento ha fatto due cose: ha modificato il codice per aggiungere i campi mancanti ai servizi, e ha aggiornato il documento `STT_API.md` con le nuove informazioni. Fin qui tutto bene, apparentemente.

Il problema è che Claude Code ha documentato modifiche al servizio `GET /client/user/usage` che in realtà non aveva fatto nel codice. Il documento diceva che il servizio restituiva i nuovi campi STT, ma il codice non era stato toccato. Se avessi copiato quel documento nella cartella del frontend e avessi chiesto a Claude Code del frontend di implementare l'integrazione basandosi su quelle specifiche, il frontend avrebbe chiamato un servizio aspettandosi campi che non esistevano. Un bug silenzioso che sarebbe emerso solo in produzione.

L'ho scoperto rileggendo il documento e confrontandolo con il codice — la fase di verifica ha funzionato, ma solo perché in questo caso conoscevo il codice abbastanza da accorgermi della discrepanza. Se fosse stato un dominio meno familiare, avrei preso per buona la documentazione e il bug sarebbe passato.

Questo episodio aggiunge una sfumatura importante: lo strumento può essere internamente incoerente. Può fare una cosa nel codice e scriverne un'altra nella documentazione. Non per malice, ma perché il contesto della sessione è lungo, le operazioni sono multiple, e lo strumento non ha un meccanismo automatico per verificare la coerenza tra le proprie azioni. Quel meccanismo sei tu.

## Il pattern della fiducia mal riposta

Guardando questi episodi con il senno di poi, vedo un pattern comune. In entrambi i casi, il fattore scatenante non è stato un errore tecnico — è stata una mia decisione di abbassare il livello di controllo.

Sul devops di Qdrant, ho accettato un file systemd senza chiedere prima un'analisi delle implicazioni di ogni opzione. Se avessi fatto quello che faccio sempre — "dimmi cosa fa ogni riga di questo file, non toccare nulla" — Claude Code mi avrebbe spiegato che `ProtectSystem=strict` blocca l'accesso al filesystem per il processo, e io avrei potuto valutare se quella restrizione aveva senso per un wrapper Docker. Non avrei capito tutti i dettagli, ma avrei avuto abbastanza informazioni per chiedere "sei sicuro che queste opzioni funzionino con Docker?" — e quella domanda avrebbe probabilmente evitato il problema.

Sulla documentazione STT, ho verificato il codice ma avrei potuto chiedere esplicitamente: "controlla che ogni modifica descritta nel documento corrisponda a una modifica reale nel codice". Una richiesta di coerenza incrociata che il metodo supporta perfettamente ma che non ho fatto.

In entrambi i casi, la correzione non richiedeva competenza aggiuntiva — richiedeva disciplina. Il metodo funziona solo se lo applichi, e la tentazione di saltarlo è più forte proprio quando ne avresti più bisogno: nei territori che non conosci.

## Cosa insegna il fallimento

La lezione che ho tratto da queste esperienze non è "impara il devops" — anche se non farebbe male. La lezione è più generale e riguarda chiunque usi il vibe coding su progetti reali.

Il metodo è robusto finché mantieni il tuo ruolo. Il tuo ruolo non è sapere tutto — è sapere dove non sai, e usare quella consapevolezza per aumentare il controllo invece di diminuirlo. Quando entri in un territorio sconosciuto, la risposta giusta non è fidarti di più dello strumento perché "lui ne sa più di me". La risposta giusta è rallentare, chiedere più spiegazioni, forzare più diagnosi, verificare più a fondo. Fare il contrario di quello che l'istinto suggerisce.

Lo strumento non diventa meno affidabile quando esci dalla tua comfort zone — tu diventi meno capace di verificarne l'affidabilità. La differenza è sottile ma fondamentale. Claude Code non ha sbagliato di più sulla sessione Qdrant rispetto alla sessione sui canali. Ha sbagliato in modo diverso, e io non ero attrezzato per accorgermene in tempo.

Questo non è un limite del vibe coding. È un limite umano che il vibe coding amplifica. Se sai poco di un argomento e hai uno strumento che produce risposte convincenti su quell'argomento, la combinazione è pericolosa — perché la risposta sembra ragionevole, tu non hai gli strumenti per contestarla, e il risultato è che accetti qualcosa di sbagliato con la piena convinzione che sia giusto.

La soluzione non è smettere di usare lo strumento fuori dalla propria competenza — sarebbe buttare via metà del suo valore. La soluzione è essere più metodici, non meno, proprio quando ci si sente meno sicuri. Vincolare di più, diagnosticare di più, verificare di più. E accettare che in quei casi il ciclo sarà più lento e più costoso, perché lo sarà.
