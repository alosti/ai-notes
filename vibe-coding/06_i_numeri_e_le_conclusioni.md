# 6. I numeri e le conclusioni

## Quaranta sessioni, centotrenta dollari

Prima di tirare le fila, vale la pena guardare i numeri nudi. Non per feticismo della metrica, ma perché i numeri raccontano una storia che le impressioni soggettive da sole non raccontano.

Il progetto ARIA — backend e frontend insieme — è stato sviluppato in quaranta sessioni di Claude Code. Diciannove per il backend Python, ventuno per il frontend Vue. Il costo totale in token è stato di circa centotrentaquattro dollari. Centotrenta dollari per un sistema che include un engine di analisi finanziaria con canali e analisi ciclica, un'integrazione con LLM per la generazione di risposte, un servizio di Speech-to-Text in streaming via WebSocket, un sistema di rate limiting e quote per tier, una console di amministrazione completa, il tutto con frontend responsive, localizzazione in due lingue, e deployment in produzione con Docker e systemd.

Non è un progetto giocattolo. È un sistema in produzione che serve utenti reali.

Il costo medio per sessione racconta la differenza tra i due progetti: circa 2.80 dollari per sessione nel backend, circa 3.90 nel frontend. Il frontend costa il quaranta per cento in più a parità di sessioni. Questo riflette quello che ho descritto nei capitoli precedenti: più iterazioni, più tentativi, più cicli di correzione visiva e di aggiustamento. Il prezzo della delega maggiore.

## Cosa dicono i numeri sulla realtà del vibe coding

Il primo dato che salta all'occhio è il tempo. Quaranta sessioni non sono quaranta giornate di lavoro — molte sessioni durano meno di un'ora, alcune si esauriscono in venti minuti. Il tempo totale di sviluppo è una frazione di quello che sarebbe stato necessario scrivendo tutto a mano. Non ho un conteggio preciso delle ore, ma la mia stima è che il rapporto sia almeno di uno a cinque — quello che ho fatto in una sessione di Claude Code mi avrebbe richiesto una giornata intera di lavoro manuale, forse di più.

Questo non significa che io abbia lavorato poco. Significa che il mio lavoro si è spostato. Meno tempo a scrivere codice, più tempo a pensare, specificare, verificare. Il tempo risparmiato sulla scrittura è stato reinvestito sulla qualità delle decisioni. Questo è il vero dividendo del vibe coding — non fare meno lavoro, ma fare lavoro diverso.

Il secondo dato è la distribuzione dei costi. Le sessioni più costose non sono quelle in cui il codice prodotto era più complesso — sono quelle in cui il dialogo era più lungo. La sessione più cara del frontend, quasi sei dollari, è stata quella in cui ho integrato lo Speech-to-Text: una funzionalità complessa che ha richiesto una lunga fase di analisi, una proposta dettagliata, e poi un'implementazione che toccava composable, componenti, proxy Vite, e gestione degli stati. Il costo riflette il pensiero, non la battitura.

Il terzo dato è il rapporto tra sessioni e feature. Alcune funzionalità hanno richiesto più sessioni — l'analisi ciclica ne ha richieste quattro tra backend e frontend, il sistema di admin ne ha richieste sei. Altre si sono risolte in una sessione singola. Non esiste una regola fissa, e questo è un punto importante: il vibe coding non rende prevedibile il costo di una feature. Lo rende più basso in media, ma la varianza resta alta perché dipende dalla complessità del problema, dalla chiarezza della specifica, e dalla quantità di iterazioni necessarie.

## Quello che i numeri non dicono

I numeri non misurano il valore delle decisioni prese durante il processo. Non misurano il bug evitato perché la fase di diagnosi ha identificato un crash su `.toFixed()` con campo opzionale prima che arrivasse in produzione. Non misurano la feature disegnata meglio perché ho avuto tempo di ragionarci invece di affogare nel codice. Non misurano l'architettura più pulita perché ho potuto permettermi di fare un revert completo e ricominciare da zero quando la direzione non mi convinceva.

I numeri non dicono nemmeno quanto del risultato finale è merito dello strumento e quanto è merito mio. È una domanda che non ha risposta, perché il risultato è il prodotto dell'interazione tra i due. Claude Code senza la mia esperienza di dominio avrebbe prodotto codice funzionante ma probabilmente sbagliato nelle scelte architetturali. Io senza Claude Code avrei prodotto le stesse scelte architetturali ma in un tempo molto più lungo e probabilmente con più errori di implementazione.

Non è una questione di chi fa di più. È una collaborazione asimmetrica in cui ciascuna parte contribuisce con quello che sa fare meglio.

## La competenza non è opzionale

Se c'è una tesi che attraversa tutto questo libro, è questa: la competenza tecnica cambia dove intervieni nel processo, ma non cambia il fatto che devi intervenire.

Sul backend, dove la mia esperienza è profonda, intervengo all'inizio — scrivo la specifica, disegno la soluzione, e delego la traduzione in codice. Sul frontend, dove la mia esperienza è minore, intervengo alla fine — valuto il risultato, chiedo correzioni, giudico con gli occhi dell'utente. Fuori dalla mia competenza, sul devops, il metodo si è indebolito perché non sapevo né cosa chiedere né cosa verificare.

Il vibe coding amplifica quello che hai. Se hai esperienza, la amplifica in produttività — fai di più nello stesso tempo, con meno errori, con meno lavoro meccanico. Se hai gusto e sensibilità per il prodotto, la amplifica in qualità — hai il tempo di ragionare sulle scelte invece di correre dietro al codice. Se hai metodo, la amplifica in consistenza — applichi lo stesso rigore su più fronti contemporaneamente.

Ma se non hai nulla da mettere dentro il processo, quello che esce è vuoto. Codice che funziona ma che non ha una direzione, un'architettura senza ragione, un prodotto che fa cose ma non risolve problemi. Ho sentito qualcuno dire che l'AI non sostituirà chi ha qualcosa da dire — li aiuterà a dirlo meglio e più velocemente. Chi non ha nulla da dire produrrà lavoro anonimo, e quel lavoro non interesserà a nessuno perché chiunque sarà in grado di ottenere almeno quel risultato. È esattamente la mia esperienza con il vibe coding.

## Il ruolo che resta

Alla fine di questo percorso, quello che mi porto a casa non è un metodo in cinque fasi — anche se il metodo funziona e l'ho descritto nel modo più onesto possibile. Quello che mi porto a casa è una consapevolezza sul mio ruolo.

Non sono più uno sviluppatore nel senso tradizionale del termine. Non passo le giornate a scrivere codice riga per riga, a cercare punti e virgola mancanti, a debuggare con print sparse per il codice. Ma non sono nemmeno un manager che delega e basta. Sono qualcosa nel mezzo — un architetto che sa costruire, un team leader che sa leggere il codice del proprio team, un direttore d'orchestra che sa suonare gli strumenti.

Claude Code non mi ha sostituito. Mi ha liberato dal lavoro meccanico e mi ha lasciato con il lavoro che conta: decidere cosa costruire, come costruirlo, e perché. E poi verificare che quello che è stato costruito corrisponda a quello che avevo in testa.

Questo lavoro richiede esperienza. Richiede aver visto abbastanza progetti andare male da sapere dove guardare. Richiede aver scritto abbastanza codice cattivo da riconoscerlo quando lo vedi. Richiede aver passato abbastanza tempo nel dominio del problema da sapere cosa ha senso e cosa no.

Il vibe coding non è una scorciatoia. È un modo diverso di lavorare che premia chi ha qualcosa da portare al tavolo. La tecnologia è accessibile a tutti. La differenza la fa quello che ci metti dentro.
