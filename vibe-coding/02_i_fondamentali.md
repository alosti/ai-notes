# 2. I fondamentali

## Un corso di quattro ore

Prima di mettere le mani su Claude Code per un progetto vero, ho fatto una cosa che probabilmente molti saltano: ho seguito un corso. Era il corso ufficiale di Anthropic su DeepLearning.ai, dedicato a Claude Code. Non era lungo — si completava in poche ore — e non prometteva di trasformare nessuno in un esperto. Quello che faceva, e lo faceva bene, era mostrare gli strumenti di base e come usarli.

Per qualcuno con la mia esperienza di sviluppo, il corso non conteneva concetti rivoluzionari. Non spiegava come progettare software, non insegnava architettura, non affrontava problemi complessi. Quello che mi ha dato è stato un insieme di meccanismi pratici — i mattoni, per così dire — che mi servivano per iniziare a lavorare senza perdere tempo a scoprirli per tentativi. Sapere che esistono e come funzionano è diverso dal saperli combinare in modo efficace, ma senza conoscerli non si parte nemmeno.

Credo sia giusto riconoscere questo debito prima di raccontare qualsiasi altra cosa. Il metodo che descrivo in questo libro è mio, ma i mattoni con cui è costruito vengono in buona parte da lì.

## La @ — dare occhi allo strumento

Il primo meccanismo pratico che ho imparato è il più semplice: la `@` seguita dal nome di un file. In Claude Code, scrivere `@nome_file` nel prompt significa dire "metti questo file nel contesto della conversazione". Lo strumento lo legge, lo tiene presente, e lo usa come riferimento per tutto quello che segue.

Sembra una cosa banale, e in un certo senso lo è. Ma la differenza rispetto al mio vecchio modo di lavorare con Claude.ai era enorme. Prima dovevo copiare manualmente i pezzi di codice rilevanti e incollarli nella chat, decidendo io cosa includere e cosa tagliare. Adesso potevo indicare un file intero con un carattere, e lo strumento aveva accesso al contenuto completo — niente pezzi mancanti, niente contesto parziale, niente errori di selezione.

Nella pratica, ho imparato velocemente che la `@` non è solo un modo per dare informazioni — è un modo per dare vincoli. Se nel prompt scrivo `@engine.py` e poi dico "modifica la gestione degli errori senza cambiare le firme delle funzioni", Claude Code ha sotto gli occhi esattamente il codice di cui sto parlando. Non deve indovinare, non deve chiedere chiarimenti, non deve inventare un contesto che non ha. Vede il file, vede le firme, vede la struttura. Il vincolo diventa verificabile perché lo strumento ha tutti gli elementi per rispettarlo.

Con il tempo ho sviluppato un'abitudine precisa: prima di scrivere qualsiasi richiesta, mi fermo e penso a quali file servono nel contesto. Non tutti — solo quelli rilevanti per la richiesta specifica. Mettere troppi file nel contesto non aiuta, anzi: diluisce l'attenzione dello strumento e aumenta la possibilità che tocchi cose che non dovrebbe toccare. La selezione del contesto è un atto di design, non un automatismo.

## Think Hard — forzare il ragionamento

Il secondo meccanismo che ho imparato dal corso è l'uso di "Think Hard" nei prompt. In Claude Code, aggiungere questa frase attiva il ragionamento esteso: lo strumento si prende più tempo per analizzare il problema prima di rispondere, esplora più possibilità, e produce risposte più ragionate.

L'effetto pratico è visibile. Senza "Think Hard", Claude Code tende a partire subito con una soluzione — spesso buona, ma a volte superficiale, soprattutto su problemi complessi dove la prima idea non è necessariamente la migliore. Con "Think Hard", lo strumento rallenta, considera più angolazioni, e arriva a risposte che tengono conto di più vincoli contemporaneamente.

Ma il valore reale di "Think Hard" non l'ho capito dal corso — l'ho capito usandolo in combinazione con un altro vincolo. Nelle mie sessioni di lavoro, "Think Hard" compare quasi sempre insieme a "non toccare il codice". Le due istruzioni insieme producono qualcosa di specifico: forzano Claude Code a ragionare in profondità su un problema senza agire. È l'equivalente di dire a un collega "pensaci bene e dimmi cosa ne pensi, ma non cambiare nulla finché non ne parliamo". Questo crea una fase di diagnosi separata dalla fase di implementazione — una distinzione che nel lavoro con gli LLM è fondamentale e che approfondirò nel prossimo capitolo.

Dal corso ho imparato che "Think Hard" esiste e cosa fa. Dal lavoro quotidiano ho imparato quando usarlo e con cosa combinarlo. La differenza tra le due cose è la differenza tra conoscere uno strumento e saperlo usare.

## CLAUDE.md — la memoria del progetto

Il terzo meccanismo è il file CLAUDE.md. È un file di testo che si mette nella root del progetto e che Claude Code legge automaticamente all'inizio di ogni sessione. Contiene istruzioni, convenzioni, vincoli — tutto quello che lo strumento deve sapere sul progetto prima ancora che tu gli chieda qualcosa.

Il corso mostrava come generarlo con il comando `claude /init`, che produce una versione iniziale basata sulla struttura del progetto. Ma la versione generata automaticamente è solo un punto di partenza. Il valore del CLAUDE.md emerge quando lo scrivi tu, a mano, con le regole specifiche del tuo progetto — quelle che nessun automatismo può indovinare.

Nel mio caso, il CLAUDE.md conteneva cose come: la struttura delle directory e il ruolo di ciascuna, le convenzioni di naming, i pattern architetturali da rispettare, i file che non dovevano essere toccati senza autorizzazione esplicita. Erano le stesse informazioni che prima dovevo ripetere a mano in ogni sessione di Claude.ai — "ricordati che l'engine funziona così", "le risposte dell'LLM passano da qui", "non modificare la firma di questa funzione". Con il CLAUDE.md, queste informazioni erano sempre presenti, automaticamente, senza che dovessi ricordarmi di includerle ogni volta.

Il corso mi ha insegnato anche un'altra cosa utile: il CLAUDE.md generico, quello che si mette nella home dell'utente e si applica a tutti i progetti. Lì ho messo le regole che valgono sempre, indipendentemente dal progetto — lo stile dei commenti, il modo in cui voglio che vengano gestiti gli errori, le convenzioni di formattazione che preferisco. In questo modo, ogni volta che apro Claude Code su qualsiasi progetto, lo strumento parte già con una base di comportamenti coerenti con il mio modo di lavorare.

## I mattoni e la costruzione

Questi tre meccanismi — la `@`, "Think Hard", e il CLAUDE.md — sono i fondamentali. Sono semplici da imparare, si capiscono in pochi minuti, e chiunque segua il corso o legga la documentazione può iniziare a usarli subito. Non c'è nessun segreto, nessun trucco nascosto.

Ma c'è una differenza sostanziale tra conoscere questi strumenti e usarli in modo efficace su un progetto reale. La `@` è utile solo se sai quali file mettere nel contesto e quali escludere. "Think Hard" è utile solo se sai quando forzare il ragionamento e quando lasciare che lo strumento agisca direttamente. Il CLAUDE.md è utile solo se sai quali regole scriverci — e per saperlo devi aver lavorato abbastanza a lungo su progetti veri da sapere quali sono le regole che contano.

I fondamentali sono i mattoni. Il metodo è la costruzione. E la costruzione, come vedremo nel prossimo capitolo, è tutta un'altra storia.
