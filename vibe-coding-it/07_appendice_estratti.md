# Appendice — Estratti dalle sessioni

Questa appendice raccoglie una selezione di scambi reali dalle sessioni di Claude Code durante lo sviluppo di ARIA. Gli estratti sono stati scelti perché rappresentativi del metodo descritto nel libro. Le risposte dello strumento sono state abbreviate dove necessario per ragioni di spazio, ma i prompt sono riportati nella loro forma originale. In alcuni punti il linguaggio è stato reso meno colorito rispetto all'originale — chi lavora sotto pressione con un terminale aperto sa come vanno queste cose.

---

## Il piano come prompt

*Backend — Implementazione analisi ciclica (7 marzo 2026, costo sessione: $1.87)*

Questa sessione mostra il caso estremo del "piano come prompt": un documento tecnico completo dato in pasto a Claude Code come istruzione di implementazione. Il prompt è stato scritto prima della sessione, non durante.

> **👤** Implement the following plan:
>
> **\# Piano: Implementazione Analisi Ciclica (CICLI)**
>
> **Context:** `bot/analysis/cicli.py` è attualmente uno stub che restituisce sempre un messaggio work-in-progress. La matematica dei canali di Millard è già completamente implementata in `bot/analysis/millard_channels.py` e esposta tramite `analyze(prices) -> MillardAnalysis`. Il piano è collegare queste due parti per produrre il JSON strutturato che l'engine e il responder si aspettano.
>
> **File modificati:**
> - `bot/analysis/cicli.py` — Riscrittura completa (sostituisce lo stub)
> - `llm/responder.py` — Aggiunta entry "cicli" in `ANALYSIS_INSTRUCTIONS_REGISTRY`
>
> **Funzioni helper:**
> - `_estimate_duration(turning_points)`: Filtra i soli minimi. 3+ minimi → media delle ultime due durate. 2 minimi → singola durata disponibile. <2 minimi → None.
> - `_current_cycle_state(turning_points)`: Ritorna (start_date, phase). Ultimo TP kind=='min' → fase ascending. Ultimo TP kind=='max' → fase descending.
> - `_performance(prices, start_date)`: (prezzo_corrente - prezzo_start) / prezzo_start * 100, arrotondato a 2 decimali.
>
> **Nota sul dict errore:** deve avere 'error', 'instrument', 'message' perché l'engine fa `block['instrument']` e `block['message']` sul percorso di errore (riga 707-708 engine.py).
>
> [seguivano: architettura dati completa, tabella delle incongruenze spec vs codice, casi limite da testare]

**🤖** Done. Here's a summary of what was implemented: [...]

Il piano conteneva costanti, firme delle funzioni, logica in pseudocodice, struttura del JSON di output, flusso dati completo, incongruenze già risolte, e casi limite. Claude Code l'ha implementato in due minuti con una sola svista minore — un import inutilizzato.

---

## Il vincolo diagnostico: "Non toccare il codice. Think Hard!"

*Frontend — Migrazione struttura dati tabella cicli (11 marzo 2026, costo sessione: $2.98)*

Questo estratto mostra la combinazione vincolo + ragionamento esteso che precede ogni implementazione significativa.

> **👤** È stata fatta una variazione nel backend per quanto riguarda i dati che vengono passati per costruire la tabella con le informazioni sui cicli.
>
> [seguiva: nuova struttura JSON campo per campo, tabella delle differenze rispetto alla versione precedente, spiegazione dei campi opzionali e mutualmente esclusivi]
>
> Ti chiedo di:
> - analizzare i nuovi dati passati rispetto a quelli attuali per valutare quali ha senso utilizzare per la nostra tabella: alcuni potrebbero essere mostrati, altri attivare colori o icone per rendere più chiaro il contesto
> - siccome è stata cambiata la logica con cui vengono passate le durate (ora sono dinamiche), valutare l'impatto di adattarsi a questa nuova struttura dei dati
> - controlla i punti di impatto nel codice ed aggiorna i mock a questa nuova struttura in modo che si possano fare dei test
>
> Non toccare il codice. Think Hard!

**🤖** Ho tutto quello che mi serve. Analisi completa senza toccare il codice.

[seguiva: mappatura dei campi vecchio → nuovo, valutazione campo per campo con impatti sulla UI, tabella dei punti di impatto nel codice con numeri di riga, proposta di design per i casi ambigui]

L'analisi prodotta era di due pagine e identificava sei punti di impatto nel componente Vue, compreso un crash potenziale su `.toFixed()` con campo opzionale. Tutto prima di toccare una riga di codice.

---

## La sfida: contestare le proposte sbagliate

*Backend — Nomenclatura dei cicli (7 marzo 2026)*

Questo scambio mostra come la competenza di dominio viene usata per correggere proposte tecnicamente ragionevoli ma inadatte al contesto.

> **👤** Il nome da dare ai cicli. "Ciclo intermedio" e "ciclo lungo" non dice nulla e non è compatibile con i nomi dei cicli: quello che mi piacerebbe è dare un nome che sia più descrittivo basato sulla durata, come ad esempio ciclo mensile, ciclo a due mesi, ciclo semestrale.

**🤖** L'idea è buona. [...] Una funzione di bucketing: [proponeva un sistema di fasce con soglie fisse tipo "< 14 giorni → settimanale, < 45 giorni → mensile, < 120 giorni → trimestrale"]

> **👤** La tua proposta non ha senso. Per i cicli c'è una nomenclatura specifica, piuttosto che mettere quella tabella di fasce, è meglio metterci la durata del ciclo e dire "ciclo a 24 giorni", "ciclo a 2 giorni".

**🤖** Hai ragione, era una complicazione inutile. Usare direttamente la durata misurata è più onesto, più preciso e non introduce ambiguità sui bordi.

Lo strumento proponeva una soluzione elegante ma fragile — soglie fisse che avrebbero prodotto nomi che cambiavano da un giorno all'altro quando la durata oscillava sul bordo di una fascia. La soluzione giusta era la più semplice.

---

## L'iterazione: più giri fino alla convergenza

*Backend — Unità di misura della durata (7 marzo 2026)*

Stesso dialogo, fase successiva. Tre giri di correzione prima di convergere sulla soluzione corretta.

**🤖** [propone di usare le barre come unità di durata]

> **👤** Pensi che le barre siano solo giorni?

**🤖** Hai perfettamente ragione. La durata va espressa in unità di tempo reale, non in barre.

> **👤** Ok, procedi. Occhio che se cambi `_estimate_duration` devi anche adattare `build_cycle_json` in `@bot/analysis/cicli.py` che popola la chiave 'estimated_duration_days'. Secondo me al posto di 'estimate_duration_days' andrebbe passato un oggetto con due chiavi, una con l'unità di misura e l'altra con il valore, cosa ne pensi?

**🤖** Concordo, è molto più pulito. [...] Punto chiave: userei la stessa unità per tutti i campi di uno stesso ciclo — se la durata stimata è in mesi, anche elapsed e to_close usano mesi.

> **👤** ASSOLUTAMENTE NO! Un ciclo che dura 1 mese può mancare 1 giorno, e tu mi metti 0.qualcosa mesi?

**🤖** Hai ragione. Ogni campo usa l'unità più naturale per il suo valore.

Lo strumento ha sbagliato tre volte: prima sulle barre, poi sul timeframe, infine sull'omogeneità delle unità. In nessuno dei tre casi l'errore era di programmazione — era di comprensione del dominio.

---

## Il confronto cross-linguaggio: Java come riferimento

*Backend — Verifica algoritmo canali (9 marzo 2026, costo sessione: $2.96)*

Questo prompt mostra come il codice esistente in un altro linguaggio viene usato come riferimento di correttezza.

> **👤** Vorrei che controllassi che questo modulo che calcola i canali lo faccia nel modo che produce lo STESSO RISULTATO dei file java che sono in `@temp/`. Alla fine, come nel java, mi devo trovare un dataset con il canale effettivo con la parte proiettata con il polinomio.

**🤖** Ho analizzato entrambe le implementazioni in dettaglio.
>
> **Algoritmo: è corretto ✓.** I componenti core replicano fedelmente il Java: cChannel → `_ChannelState`, periodi period1/period2 identici, doppia WMA causale ✓. Drain: `last_val += vel + acc` — identico al Java ✓. Polinomio: Lagrange su 3 punti = least-squares Commons Math su 3 punti (stessa soluzione unica) ✓.
>
> **Problema principale: il dataset combinato NON esiste.** In Java, il risultato finale è un unico array `v` — dopo `forecastChannel()`, gli ultimi `forecastPeriod` valori vengono sostituiti in-place con il polinomio. In Python, `ChannelResult` tiene `centered` e `forecast_values` separati.

Il prompt era di due righe. La risposta ha prodotto una verifica algebrica componente per componente e ha identificato un gap architetturale — il dataset combinato mancante — che era il vero problema.

---

## La delega ragionata: chiedere proposte sul frontend

*Frontend — Indicatore tier utente (15 marzo 2026, costo sessione: $2.63)*

Questo estratto mostra il modo di lavorare sul frontend: dare un'idea iniziale ma chiedere alternative, con il vincolo diagnostico attivo.

> **👤** Ciao! Mi piacerebbe mettere in evidenza il tipo di abbonamento che ha l'utente (i tier sono: Free, Pro, Max). Va bene anche solo il colore associato al tier, non per forza dobbiamo scrivere il nome del tier. Siccome l'interfaccia è sia per desktop che per mobile e ciascuna piattaforma ha le sue peculiarità, ti chiedo un'idea di come aggiungeresti questa informazione.
> Una mia idea per il desktop potrebbe essere quella di aggiungere un bordino del colore del tier sul bottone dell'utente.
> Guarda il codice, fammi qualche proposta e ragioniamoci insieme; per ora non toccare il codice, discutiamo e basta!
> Think Hard!

**🤖** La tua idea del "bordino sul bottone" ha senso concettualmente, ma il bottone `user-menu-button` è largo — un border intorno a tutto il rettangolo su sfondo nero della sidebar rischia di apparire grezzo o di sembrare un focus state. Meglio mirare all'avatar. Proposta: ring colorato sull'avatar. Un `ring-2` nel colore del tier attorno alla foto profilo. [...]

> **👤** Per quanto riguarda il desktop mi piace molto la tua idea del ring colorato sull'avatar, è geniale.

Sul frontend, il metodo è lo stesso — vincolo, diagnosi, discussione — ma il contenuto è diverso: non si sfidano dettagli algoritmici, si valutano proposte di design.

---

## La verifica che salva: scoprire il bug prima della produzione

*Frontend — Tier utente, sessione successiva (15 marzo 2026)*

Dopo aver concordato il design, Claude Code implementa. Ma la verifica rivela un problema.

> **👤** Controlla BENE perché, secondo me, lo store con l'informazione sul tier viene popolato quando si accede alla pagina dei Settings.

**🤖** Confermato: `updateUserPlan` viene chiamato solo dentro `AccountUsageSection.vue` al suo `mounted()`. Quel componente esiste solo in `Settings.vue`. Quindi se l'utente non apre mai le impostazioni, `userPlan` resta al default "Free" e il ring sarà sempre verde — sbagliato.

> **👤** Stai facendo casino, fai il rollback delle modifiche che hai fatto, faccio in modo di ritornarti il tier in getUserInfo.

L'intuizione era giusta: lo store non veniva popolato al momento giusto. Il bug sarebbe arrivato in produzione con tutti gli utenti che mostravano il ring verde del tier Free, indipendentemente dall'abbonamento reale.

---

## Il ponte tra progetti: documentazione come contratto

*Backend — Generazione specifica STT (23 marzo 2026)*

> **👤** Scrivi un documento di specifiche su come funziona il servizio di STT; il documento verrà utilizzato dal front-end per integrare i servizi ed implementare la funzionalità STT in Aria. Metti il documento insieme agli altri nella cartella `@docs/`.

**🤖** Documento creato in `docs/STT_API.md`. Copre: requisiti audio con codice di conversione Float32→Int16 pronto all'uso, protocollo completo client→server e server→client, codici di chiusura WebSocket, quote STT per tier, tre diagrammi di sequenza, snippet JavaScript, note implementative.

*Frontend — Consumo della specifica STT (23 marzo 2026, costo sessione: $5.68)*

> **👤** Ciao! Lato backend ho implementato la funzionalità di speech to text `@docs/STT_API.md`. Vorrei integrarla nel chatbot in modo professionale; come sai dobbiamo pensare sia all'integrazione desktop che mobile. Leggi la documentazione e fammi una proposta di come integreresti questa nuova funzionalità. Fino a quando non abbiamo le idee chiare non toccare il codice. Think Hard!

**🤖** [produceva una proposta completa: posizione del pulsante, stati dell'interfaccia, gestione del testo trascritto, architettura del composable, gestione errori — tutto derivato dalla specifica scritta dall'istanza backend]

Due istanze di Claude Code, due progetti separati. Un documento fa da contratto tra i due.

---

## Il revert: sapere quando tornare indietro

*Backend — Gestione download ticker (8 marzo 2026, costo sessione: $4.47)*

Dopo diversi giri di modifiche a due file, la complessità cresceva senza convergere verso una soluzione pulita.

> **👤** Reverta TUTTO, riporta il codice all'inizio della chat.

**🤖** Leggo prima lo stato attuale dei due file per fare i revert precisi. [...] Pulito. Entrambi i file sono tornati allo stato iniziale della chat.

Nessuna esitazione. Quando la direzione è sbagliata, si torna indietro.

---

## L'architettura discussa: non accettare la prima proposta

*Backend — Gestione profilo utente (16 marzo 2026, costo sessione: $0.75)*

Questo scambio mostra una discussione architetturale dove l'autore propone una soluzione diversa da quella dello strumento, la motiva, e arriva a un compromesso.

**🤖** [proponeva Strada 2: modificare `_get_user_limits` per accettare parametri opzionali]

> **👤** Non mi piace la strada 2 perché devi modificare la firma di troppi metodi per aggiungere ciò che per quei metodi è "rumore". Io propendo per un'altra soluzione che so non ti piacerà: la `_get_user_limits` resta come è, la sua funzione è quella di assicurarsi di poter gestire i limiti per quell'utente, che è un compito che ha impatto diretto sui costi, quindi importante. Creerei la funzione `ensure_user_profile` con la logica di creazione COMPLETA.
> L'end user avrà solo a disposizione le API di Aria, quindi la probabilità che sia la `_get_user_limits` a creare l'utente è quasi vicina allo zero, ma siccome i limiti sono importanti mi tengo il paracadute.
> Dimmi cosa ne pensi (so che storceresti il naso).
> Non toccare il codice!

**🤖** Hai ragione e non storco il naso più di tanto. Il mio istinto iniziale sarebbe "duplicazione della logica = rischio manutenzione". Ma guardandolo bene, la duplicazione è minima e accettabile. [...]

> **👤** Ok, fai però un refactoring per spostare la logica di creazione del documento in una funzione separata in modo che duplichiamo il codice il meno possibile. Mi fai un piano per procedere? Think Hard!

La soluzione finale non era né quella proposta dallo strumento né quella proposta dall'autore nella forma originale — era una sintesi che prendeva il meglio di entrambe: la separazione delle responsabilità voluta dall'autore, con il refactoring per minimizzare la duplicazione suggerito dallo strumento.

---

## Le parole ricorrenti

Per chiudere, i numeri grezzi delle parole più usate nei prompt, estratti dall'analisi di tutte le quaranta sessioni.

**Backend (19 sessioni):** "non toccare" — 16 volte. "Think Hard" — 13 volte. "implementa" — 11 volte. "controlla" — 8 volte. "verifica" — 7 volte.

**Frontend (21 sessioni):** "non toccare" — 17 volte. "Think Hard" — 17 volte. "implementa" — 13 volte. "funziona" — 14 volte. "modifica" — 18 volte.

Il vocabolario racconta il metodo: prima blocca, poi ragiona, poi agisci, poi verifica.
