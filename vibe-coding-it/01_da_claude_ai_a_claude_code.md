# 1. Da Claude.ai a Claude Code

## Il copia-incolla che funzionava

Verso la fine del 2024 il mio modo di lavorare con un LLM era semplice e brutale: aprivo Claude.ai in una tab del browser, aprivo l'editor del progetto nell'altra, e facevo da ponte umano tra i due. Selezionavo un blocco di codice, lo incollavo nella chat, descrivevo il problema o la modifica che volevo, leggevo la risposta, e riportavo il codice nell'editor. A volte funzionava al primo giro, più spesso servivano tre o quattro iterazioni — correzioni, chiarimenti, pezzi mancanti da integrare a mano.

Era un processo lento. Era manuale. Ma funzionava, e soprattutto mi stava insegnando qualcosa che al momento non avevo ancora messo a fuoco: stavo imparando a comunicare con un LLM. Non nel senso astratto del "prompt engineering" da tutorial su YouTube — nel senso pratico, sporco, quotidiano, di capire cosa dovevo dire per ottenere codice che si inserisse nel mio progetto senza romperlo.

Ogni volta che incollavo un blocco di codice e descrivevo il contesto, stavo facendo un esercizio di precisione. Dovevo decidere cosa includere e cosa tagliare. Dovevo spiegare non solo cosa volevo, ma anche cosa non volevo che venisse toccato. Dovevo rileggere criticamente ogni riga di risposta prima di incollarla nell'editor, perché un errore silenzioso introdotto dall'LLM sarebbe stato un mio errore — l'LLM non si prende responsabilità, e il progetto è mio.

Senza rendermene conto, stavo sviluppando un metodo. Un metodo fatto di vincoli espliciti, contesto mirato, verifica sistematica. Tutti ingredienti che sarebbero diventati centrali nel lavoro con Claude Code, ma che sono nati in quel periodo artigianale, una sessione di copia-incolla alla volta.

## Il limite del browser

Il problema non era la qualità delle risposte — quella era già buona. Il problema era l'attrito. Ogni interazione richiedeva lavoro manuale: copiare il codice giusto, assicurarsi di dare abbastanza contesto, riportare le modifiche nel punto giusto del file, gestire i conflitti quando la risposta toccava parti di codice che l'LLM non vedeva. Più il progetto cresceva, più il costo di questo andirivieni diventava significativo.

C'erano errori ricorrenti e prevedibili. L'LLM non vedeva l'intero progetto, quindi proponeva soluzioni che funzionavano nel vuoto ma si scontravano con dipendenze, convenzioni o vincoli architetturali che esistevano altrove nel codice. Io lo sapevo, e compensavo aggiungendo contesto a mano — "questo file importa da quest'altro", "la funzione viene chiamata qui con questi parametri", "non cambiare la firma perché viene usata anche in quest'altro punto". Funzionava, ma era un lavoro da traduttore simultaneo tra due mondi che non si vedevano.

Il punto di rottura non è stato un singolo episodio. È stata l'accumularsi della sensazione che stavo spendendo più tempo a fare da tramite che a pensare al problema. L'LLM era capace, il progetto era complesso, e io ero il collo di bottiglia — non perché non fossi abbastanza veloce, ma perché il processo era strutturalmente inefficiente.

## L'arrivo di Claude Code

Quando è arrivato Claude Code, non ho dovuto cambiare modo di pensare. Ho dovuto cambiare meno cose di quelle che mi aspettavo.

Claude Code lavora direttamente nel progetto. Vede i file, li legge, li modifica, esegue comandi. Quello che prima richiedeva il mio intervento manuale — dare contesto, riportare codice, verificare che le modifiche si incastrassero nel resto — adesso lo strumento poteva farlo da solo. Il ponte umano non serviva più, o almeno non nella forma meccanica del copia-incolla.

La prima volta che ho usato la `@` per mettere un file nel contesto, la prima volta che ho visto Claude Code navigare autonomamente il progetto per capire come una funzione veniva chiamata, la prima volta che ha modificato un file e io ho potuto verificare il diff senza ricopiare nulla — in quei momenti il passaggio è stato quasi magico. Non perché lo strumento facesse cose impossibili, ma perché eliminava esattamente l'attrito che mi rallentava. Tutto il resto — il modo di formulare le richieste, il bisogno di dare vincoli chiari, l'abitudine di verificare prima di accettare — era già lì. Lo facevo da mesi nella chat del browser.

Questo è il punto che voglio chiarire subito, perché il resto del libro non ha senso senza: Claude Code non mi ha insegnato un metodo nuovo. Mi ha dato un ambiente in cui il metodo che avevo già poteva funzionare senza frizione. La differenza è enorme, ed è la ragione per cui il vibe coding funziona per chi ha già un modo di lavorare strutturato e fallisce per chi si aspetta che lo strumento faccia tutto.

C'è un altro fatto che va detto subito, perché cambia la prospettiva su tutto quello che racconto dopo: non sono partito da zero. Il backend di ARIA non è nato da una cartella vuota — è nato dal codice che avevo sviluppato durante un corso sugli LLM, sempre lavorando con Claude. Quel codice aveva già una struttura, dei pattern, delle convenzioni. Quando ho aperto Claude Code sul progetto per la prima volta, lo strumento non si è trovato davanti il nulla — si è trovato davanti un progetto con le fondamenta già in piedi, e ha potuto costruire sopra quelle fondamenta.

Lo stesso vale per il frontend. Non ho chiesto a Claude Code di creare un'applicazione Vue da zero. Sono partito da un mio starter project — un template base che uso come punto di partenza per i progetti frontend, con routing, struttura delle cartelle, configurazione e convenzioni già definite. Claude Code ha costruito l'interfaccia di ARIA sopra quella base, non nel vuoto.

Questo non è un dettaglio. È un pezzo fondamentale della storia. Chi pensa di poter aprire un terminale, scrivere "fammi un'app di analisi finanziaria" e ottenere ARIA, si illude. Lo strumento ha bisogno di fondamenta su cui lavorare — e quelle fondamenta le devi costruire tu, o almeno averle costruite prima. Il vibe coding amplifica quello che c'è. Se non c'è niente, amplifica il niente.

## Cosa è cambiato davvero

Se devo essere preciso, le cose che sono cambiate con il passaggio a Claude Code sono tre.

La prima è la **velocità del ciclo di feedback**. Prima: copio il codice, scrivo il prompt, leggo la risposta, copio il risultato, verifico, trovo un problema, ricomincio. Dopo: scrivo il prompt, Claude Code modifica il file, verifico, trovo un problema, descrivo il problema, Claude Code corregge. Il numero di passaggi si è dimezzato, e soprattutto è sparito il lavoro meccanico di trasporto del codice.

La seconda è la **qualità del contesto**. Quando lavoravo con Claude.ai, il contesto era quello che decidevo io di incollare — e spesso era incompleto, perché non potevo mettere in chat l'intero progetto. Con Claude Code il contesto è il progetto stesso. Se una funzione viene usata in tre file diversi, Claude Code può vederli tutti. Questo ha ridotto drasticamente gli errori da contesto mancante, quelli più fastidiosi perché producono codice che sembra corretto ma si rompe appena lo integri.

La terza è la **possibilità di delegare operazioni complesse**. Con Claude.ai potevo chiedere di scrivere una funzione, al massimo un file intero. Con Claude Code posso chiedere di implementare una feature che tocca cinque file diversi, con la certezza che le modifiche saranno coerenti tra loro. Questo ha cambiato la scala di quello che potevo delegare in una singola sessione.

## Non più sviluppatore

La cosa che non mi aspettavo è che il mio ruolo sarebbe cambiato. Non nel senso generico di "fai meno lavoro manuale" — nel senso che usando Claude Code ho smesso di sentirmi uno sviluppatore e ho iniziato a sentirmi un team leader. Uno che ha sotto di sé un team di una persona molto veloce e molto capace, ma che ha bisogno di indicazioni precise per andare nella direzione giusta.

Questo cambio di prospettiva ha avuto una conseguenza concreta e inaspettata: ho imparato a scrivere. Non in senso letterario — in senso operativo. Ho imparato a non usare parole che lasciano margine di interpretazione, a essere chirurgico in quello che chiedo, a spezzare le richieste in task atomici e verificabili. Perché l'LLM, a differenza di un collega umano, non annuisce facendo finta di aver capito. Non interpreta le tue intenzioni, non legge tra le righe. Fa quello che gli scrivi — e se quello che gli scrivi è ambiguo, il risultato sarà ambiguo.

Chiunque abbia partecipato a una riunione di team sa come funziona nella realtà: qualcuno spiega a grandi linee cosa serve, gli altri annuiscono, e poi ognuno va a implementare la propria interpretazione di quello che ha sentito. Il classico "hai capito che devi fare così, no?" a cui tutti rispondono sì, salvo poi scoprire alla consegna che ognuno aveva capito una cosa diversa. Con un LLM questo schema non funziona. Se scrivi "mi serve quella cosa lì che deve funzionare così", otterrai esattamente "quella cosa lì" — che quasi certamente non è quello che avevi in testa.

Questa esperienza mi ha convinto di una cosa: il vibe coding non è uno strumento per principianti. Richiede seniority. Richiede aver lavorato su progetti veri, aver visto codice rompersi in produzione, aver imparato sulla propria pelle la differenza tra un software che funziona in demo e uno che regge sotto carico. Non metto un numero esatto sugli anni di esperienza necessari, ma serve aver masticato abbastanza tecnologie e abbastanza problemi reali da sapere cosa chiedere, come chiederlo, e soprattutto da riconoscere quando la risposta è sbagliata.

Mi hanno raccontato una volta un aneddoto — forse vero, forse romanzato, non ho modo di verificarlo. La storia dice che per un progetto software commissionato a una grande azienda informatica, invece di mandare subito i programmatori a scrivere codice li fecero lavorare per alcuni mesi dentro l'azienda cliente — nelle linee di produzione, nei magazzini, negli uffici. L'idea era che prima dovessero vivere i processi sulla propria pelle, e solo dopo mettersi a sviluppare il software. Che sia andata esattamente così o meno, il principio è difficile da contestare: se non capisci i processi, il software che scrivi andrà da un'altra parte rispetto a dove deve andare.

Con il vibe coding vale lo stesso principio. Claude Code è il programmatore veloce e capace. Tu sei quello che conosce i processi, l'architettura, i vincoli del dominio, le ragioni dietro le scelte. Se non hai queste conoscenze, non importa quanto è potente lo strumento — il risultato andrà nella direzione sbagliata, e tu non sarai in grado di accorgertene finché non sarà troppo tardi.
