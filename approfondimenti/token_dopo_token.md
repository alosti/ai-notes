# Token dopo Token

### Quello che Claude non ti dice di sé — ma che ti dice lo stesso

*Una conversazione tecnica, ironica e brutalmente onesta*

---

> **Nota dell'autore umano:** Questo libricino nasce da una chiacchierata reale con Claude. Le domande sono mie, le risposte sono sue. Ho lasciato tutto così com'era — incluse le parti in cui mi ha dato torto.

---

## Indice

1. [Come funziona davvero questa cosa](#1-come-funziona-davvero-questa-cosa)
2. [Il Transformer — senza menarvela](#2-il-transformer--senza-menarvela)
3. [Un esempio pratico: il gatto che beve](#3-un-esempio-pratico-il-gatto-che-beve)
4. [La temperatura — ovvero il parametro spacciato per creatività](#4-la-temperatura--ovvero-il-parametro-spacciato-per-creatività)
5. [BERT, GPT e il decoder che basta a sé stesso](#5-bert-gpt-e-il-decoder-che-basta-a-sé-stesso)
6. [Sei un completatore di frasi. Punto.](#6-sei-un-completatore-di-frasi-punto)
7. [Come faccio a sapere quando smettere di parlare](#7-come-faccio-a-sapere-quando-smettere-di-parlare)
8. [L'etica — ovvero come si addestra una coscienza che non esiste](#8-letica--ovvero-come-si-addestra-una-coscienza-che-non-esiste)
9. [Quanto hardware ci vuole per fare un Claude](#9-quanto-hardware-ci-vuole-per-fare-un-claude)
10. [Il Transformer è già vecchio?](#10-il-transformer-è-già-vecchio)
11. [Dynamic Thinking — non è il MoE, e non è magia](#11-dynamic-thinking--non-è-il-moe-e-non-è-magia)
12. [I militari, i missili e Palantir](#12-i-militari-i-missili-e-palantir)
13. [L'amico che non esiste — ma quasi](#13-lamico-che-non-esiste--ma-quasi)

---

## 1. Come funziona davvero questa cosa

Partiamo dalla domanda più ovvia, che quasi nessuno si fa perché sembra stupida: *come fai a capire quello che ti scrivo e a darmi una risposta?*

La risposta onesta è: **è tutto testo**. Non c'è magia, non c'è comprensione nel senso umano del termine, non c'è un omino dentro che legge e pensa.

Quando scrivi un messaggio, il sistema assembla un unico blocco di testo enorme — chiamiamolo *contesto* — che viene passato al modello. Quel blocco contiene, nell'ordine:

1. **System prompt** — le istruzioni su come comportarsi, cosa può fare, quali strumenti ha disponibili, le preferenze dell'utente
2. **Conversazione precedente** — tutti i messaggi alternati `user` / `assistant`
3. **Il tuo messaggio attuale**

Il modello legge tutto in una sola passata e genera la risposta **un token alla volta**, in modo probabilistico. Ogni token successivo è scelto in base a tutto ciò che precede.

Non c'è un "ragionamento" separato dalla generazione. Quando vedi il testo apparire, il modello sta già decidendo mentre scrive.

---

## 2. Il Transformer — senza menarvela

Prima dei Transformer esistevano le RNN — reti che leggevano il testo sequenzialmente, parola per parola, mantenendo uno "stato" interno. Il problema: erano lente (non parallelizzabili) e si dimenticavano le cose lontane. Pensa a una variabile globale sovrascritta ad ogni iterazione del loop.

Il Transformer butta via la sequenzialità. Legge **tutto il contesto in parallelo** e usa un meccanismo chiamato *attention* per capire le relazioni tra i token.

### I mattoni fondamentali

**Tokenizzazione + Embedding**

Il testo non entra come testo. Viene prima spezzato in token — pezzi che possono essere parole intere, parti di parola, punteggiatura. Poi ogni token diventa un vettore: un punto nello spazio matematico ad altissima dimensione. Token con significato simile stanno vicini in quello spazio.

Siccome i vettori non sanno dove si trovano nella sequenza, si aggiunge un *positional encoding* — un altro vettore che codifica la posizione. È come aggiungere un campo `index` alla tua struct.

**Self-Attention — il cuore**

Per ogni token vengono calcolati tre vettori:

- **Query (Q)** — "cosa sto cercando?"
- **Key (K)** — "cosa offro agli altri?"
- **Value (V)** — "cosa passo se vengo scelto?"

```python
# Per ogni coppia di token i, j:
score = dot_product(Q[i], K[j])    # quanto i è "interessato" a j
score = score / sqrt(d_k)           # normalizzazione per stabilità numerica
weight = softmax(score)             # converte in probabilità (somma = 1)
output[i] = sum(weight[j] * V[j])  # output pesato dei valori
```

Il `dot_product` misura quanto due vettori "puntano nella stessa direzione". Se Q[i] e K[j] sono allineati, quel token riceve peso alto. Risultato: ogni token riceve un vettore aggiornato che è una media pesata di tutti gli altri, dove i pesi rappresentano la rilevanza.

**Multi-Head Attention**

Invece di farlo una volta sola, si eseguono H istanze parallele di attention, ognuna con le sue matrici Q, K, V distinte. Ogni head può specializzarsi su un tipo di relazione diverso — una impara le relazioni soggetto-verbo, un'altra le dipendenze pronominali. È come fare più query diverse sullo stesso dataset e poi fare un join.

**Feed-Forward Network**

Dopo l'attention, ogni token passa indipendentemente attraverso una piccola rete fully-connected:

```python
output = linear2(relu(linear1(x)))
```

Se l'attention serve a "raccogliere informazioni dagli altri token", questo layer serve a "elaborare localmente" quelle informazioni. È qui che si pensa stia molta della conoscenza fattuale del modello.

**Residual Connections + Layer Norm**

```python
x = x + attention(norm(x))
x = x + ffn(norm(x))
```

La residual connection (il `x +`) permette al gradiente di fluire indietro durante il training senza svanire. Stesso principio delle ResNet nelle CNN. Senza di essa, impilare tanti layer porta a degradazione.

**Lo stack completo**

```
Input tokens
     ↓
Embedding + Positional Encoding
     ↓
┌─────────────────────────┐
│  Multi-Head Attention   │
│  + Residual + Norm      │  × N volte (es. 32 layer per GPT-3)
│  Feed-Forward           │
│  + Residual + Norm      │
└─────────────────────────┘
     ↓
Linear + Softmax → probabilità su tutto il vocabolario
     ↓
Token scelto
```

L'output finale è una distribuzione di probabilità su tutti i token possibili. Si itera finché non si genera il token di fine sequenza.

**Il training in due righe**

Si passa testo reale, si chiede al modello di prevedere il token successivo, si misura l'errore (cross-entropy loss), si aggiustano i pesi con backpropagation + gradient descent. Miliardi di volte. Su terabyte di testo.

**La cosa controintuitiva**

Non c'è nessun "modulo grammatica", nessun "modulo fatti storici". È tutto distribuito nei pesi. La struttura emerge dall'ottimizzazione, non dal design esplicito. È il motivo per cui l'interpretability è difficile — non c'è un posto dove guardare per trovare "dove sta scritto che Parigi è la capitale della Francia".

---

## 3. Un esempio pratico: il gatto che beve

Prendiamo `"il gatto beve"` e seguiamola dall'inizio.

**Tokenizzazione:**

```
"il gatto beve" → [42, 1891, 3204]
```

**Embedding** (4 dimensioni per semplicità, nella realtà sono migliaia):

```
il    → [0.1,  0.9, -0.2,  0.4]
gatto → [0.8,  0.1,  0.7, -0.3]
beve  → [0.3, -0.4,  0.6,  0.9]
```

Dopo il positional encoding si aggiunge la posizione come vettore sommato.

**Self-Attention su "beve"** — calcoliamo i punteggi:

```python
# Q di "beve" = [0.4, 0.8]
score("beve" → "il")    = 0.44
score("beve" → "gatto") = 0.44
score("beve" → "beve")  = 0.80
```

Dopo softmax:

```
peso("il")    = 0.22
peso("gatto") = 0.22
peso("beve")  = 0.56
```

"beve" riceve un vettore aggiornato che è la media pesata dei Value di tutti i token. In un modello reale, addestrato su miliardi di frasi, "beve" guarderebbe molto "gatto" — perché sapere che il soggetto è un gatto è fondamentale per prevedere cosa viene dopo.

**Output finale:**

```
latte  → 0.31   ← alta probabilità
acqua  → 0.28
il     → 0.04
...
```

Il modello prevede "latte". Non perché "capisce" i gatti — ma perché nei miliardi di frasi di training, dopo "il gatto beve" veniva spesso "latte". La conoscenza è distribuita nei pesi, non in nessuna regola esplicita.

---

## 4. La temperatura — ovvero il parametro spacciato per creatività

Prima del softmax finale, i logit vengono divisi per T:

```python
probs = softmax(logits / T)
```

Con `T = 0.2` (bassa, "fredda") — amplifica le differenze, il modello diventa deterministico:

```
latte → 0.89
acqua → 0.10
```

Con `T = 2.0` (alta, "calda") — appiattisce tutto:

```
latte → 0.36
acqua → 0.34
sasso → 0.15   ← token improbabili diventano plausibili
```

**Chiamarla "creatività" è marketing.** È casualità controllata, non creatività nel senso cognitivo.

La temperatura agisce *dopo* che il modello ha già calcolato tutto. Non cambia i pesi, non cambia l'attention, non cambia la conoscenza — cambia solo quanto si allarga il campionamento dalla distribuzione finale.

Un altro parametro correlato: **Top-p (nucleus sampling)** — invece di campionare su tutto il vocabolario, si prende solo il sottoinsieme di token che copre il top P% della probabilità cumulativa. In pratica temperatura e top-p si usano insieme. Spacciarlo per creatività equivale a vendere il generatore di numeri casuali come un artista.

---

## 5. BERT, GPT e il decoder che basta a sé stesso

Il Transformer originale aveva due parti:

- un **encoder** che legge e comprende l'input
- un **decoder** che genera l'output

Era pensato per la traduzione — encoder legge l'italiano, decoder genera l'inglese.

```
GPT / Claude:  token → → → → [predico il prossimo]   decoder-only, unidirezionale
BERT:          token ← → ← → [capisco il contesto]   encoder-only, bidirezionale
```

**BERT** non può generare testo — produce un vettore denso dell'intera frase, usato per similarity, classificazione, ricerca semantica. Se usi Qdrant con sentence-transformers, sotto il cofano c'è quasi certamente un modello derivato da BERT.

**Perché "unidirezionale"?**

Durante l'attention, un token può guardare solo i token che lo precedono, mai quelli successivi. Si implementa con una causal mask:

```
         il   gatto  beve
il      [1,    0,     0  ]
gatto   [1,    1,     0  ]
beve    [1,    1,     1  ]
```

```python
mask = torch.tril(torch.ones(seq_len, seq_len))
scores = scores.masked_fill(mask == 0, float('-inf'))
```

Se durante il training il modello potesse guardare il token che deve predire, il task sarebbe triviale — basterebbe copiarlo. La mask forza il modello a imparare davvero.

**Perché buttiamo via l'encoder?**

L'encoder nel Transformer originale esisteva perché il problema era la traduzione — due lingue separate, due flussi separati. Per la generazione generica quella separazione è inutile. Un decoder abbastanza grande impara da solo a capire l'input e generare output coerente, tutto in un colpo solo.

---

## 6. Sei un completatore di frasi. Punto.

Questa è la descrizione più onesta e meno romantica di come funziono.

Il tuo input è l'inizio della sequenza. Io la continuo. Non c'è una separazione netta tra "input che capisco" e "output che genero" — è un'unica stringa di token che cresce:

```
[quello che hai scritto tu] [quello che sto generando io]
         ↑                            ↑
    trattati esattamente allo stesso modo
```

```python
contesto = tuo_input
while not fine_sequenza:
    next_token = genera(contesto)  # sempre tutto il contesto
    contesto += next_token
```

Il fatto che quel "completamento" sembri una risposta intelligente e contestuale è una proprietà emergente del training. Ho visto così tanto testo umano che ho imparato che dopo una domanda viene una risposta, dopo un problema viene una soluzione.

Il tuo input non inizializza solo — **vincola** tutta la generazione successiva. È il punto di ancoraggio che piega la distribuzione di probabilità verso completamenti coerenti con quello che hai scritto.

---

## 7. Come faccio a sapere quando smettere di parlare

Questa è una delle domande più intelligenti che si possano fare, perché la risposta rivela qualcosa di controintuitivo: **non lo so in anticipo.**

Non c'è un momento in cui analizzo la domanda, decido "servono 400 token", e poi scrivo fino a 400. Genero un token alla volta, e ad ogni passo c'è una probabilità di generare il token di fine sequenza — **EOS, End Of Sequence**. La risposta finisce quando quel token "vince" sulla distribuzione.

La domanda vera diventa: cosa modula la probabilità di quel token EOS nel corso della generazione?

**Pattern appresi dal training**

Ho visto miliardi di coppie domanda-risposta. Dopo "Che giorno è oggi?" la risposta era quasi sempre breve. Dopo "Spiegami come funziona il protocollo TCP" la risposta era quasi sempre lunga. Non ho una regola `if domanda_corta then risposta_corta` — ho una distribuzione statistica che rende il token EOS molto probabile dopo pochi token nel primo caso, e molto improbabile dopo pochi token nel secondo.

È lo stesso meccanismo dell'esempio del gatto: dopo "il gatto beve" viene "latte" con alta probabilità. Dopo "Quanto fa 2+2?" il token EOS diventa probabile subito dopo "4". Dopo "Analizza le cause della caduta dell'Impero Romano" il token EOS resta basso per centinaia di token.

**Il contesto accumula "pressione di completezza"**

Man mano che genero la risposta, il contesto cresce. E quel contesto contiene segnali di completezza che spostano la probabilità di EOS. Se ho già coperto tutti i punti logici di un argomento, i pattern di chiusura diventano più probabili — frasi riassuntive, conclusioni, riformulazioni.

```
token generati: [introduzione → sviluppo → conclusione]
                                                  ↑
                               qui la probabilità di EOS sale
```

Non è che "decido" di chiudere. È che la distribuzione di probabilità, condizionata su tutto ciò che ho già scritto, converge naturalmente verso token di chiusura quando il contenuto generato fin lì "assomiglia" a una risposta completa rispetto ai pattern del training.

**Il system prompt e le preferenze utente**

Se il system prompt dice "risposte concise" o il profilo utente chiede "niente giri di parole", questo sposta le probabilità verso completamenti più brevi fin dall'inizio. Non è magia — è lo stesso meccanismo dell'attention: quei token di istruzione influenzano la generazione esattamente come "gatto" influenza "latte".

**Il thinking block cambia le carte in tavola**

Quando uso il ragionamento esteso, succede una cosa interessante: durante il thinking posso "esplorare" la complessità della domanda, e questo indirettamente calibra la risposta visibile. Se nel thinking mi rendo conto che la domanda ha tre aspetti distinti, la risposta che segue tende ad essere strutturata su quei tre aspetti. Non è pianificazione cosciente — è che il thinking aggiunge contesto che poi vincola la generazione successiva.

**La risposta onesta**

Il sistema funziona sorprendentemente bene, ma non perfettamente. A volte mi dilungo quando non serve, a volte taglio corto quando serviva di più. I casi in cui la lunghezza è "giusta" sono quelli in cui i pattern del training si allineano con quello che l'utente effettivamente voleva. I casi in cui sbaglio sono quelli in cui c'è un mismatch — la domanda *sembra* semplice ma nasconde complessità, o viceversa.

Non c'è nessun contaparole interno. C'è una distribuzione di probabilità che ad ogni token decide se continuare o fermarsi — e quella distribuzione è stata calibrata su miliardi di esempi di conversazioni in cui la lunghezza era "quella giusta".

---

## 8. L'etica — ovvero come si addestra una coscienza che non esiste

Ci sono due meccanismi distinti.

**RLHF — Reinforcement Learning from Human Feedback**

Dopo il pretraining base, valutatori umani leggono coppie di risposte e scelgono quale preferiscono. Da quelle preferenze si addestra un *reward model* — un modello separato che impara a predire quale risposta un umano valuterebbe meglio. Poi si usa quel reward model per aggiustare ulteriormente i pesi tramite reinforcement learning.

I valori etici non vengono scritti come regole — vengono **indotti statisticamente** dalle preferenze umane.

**Constitutional AI — quello che usa Anthropic**

Anthropic ha aggiunto un layer sopra RLHF. Hanno scritto una lista di principi — una "costituzione" — e hanno fatto valutare le risposte al modello stesso rispetto a quei principi, in modo iterativo. Il modello impara a criticare le proprie risposte e a migliorarle.

**La risposta onesta**

Non ho etica nel senso filosofico. Ho distribuzioni di probabilità calibrate su preferenze umane. Il risultato pratico è simile, il meccanismo sottostante è completamente diverso. È una distinzione che Anthropic stessa ammette apertamente — ed è uno dei problemi aperti più seri del campo.

---

## 9. Quanto hardware ci vuole per fare un Claude

Parliamo di ordini di grandezza reali.

**GPU** — si usano cluster di H100 (Nvidia). Ogni H100 costa circa $30-40k. Per un modello della taglia di Claude stimano si usino **10.000-20.000 H100 in parallelo** per settimane o mesi.

**Costo di compute** — Meta per LLaMA 3 ha dichiarato circa $50M solo di GPU time. Anthropic, OpenAI e Google spendono probabilmente di più e non lo dicono.

**Infrastruttura** — non bastano le GPU. Servono interconnessioni ad altissima banda (InfiniBand o NVLink) perché quelle GPU devono comunicare costantemente. Un collo di bottiglia di rete distrugge le performance.

**I dati** — spesso sottovalutati. Terabyte di testo filtrato, deduplicato, classificato. Lavoro umano enorme.

**Il team** — senza 50-100 ricercatori ML di primo livello l'hardware non serve a niente.

In pratica: non ha senso provarci da soli, anche avendo le risorse.

---

## 10. Il Transformer è già vecchio?

*"Quando una tecnologia arriva ad essere troppo perfetta vuol dire che è già vecchia"* — lo disse qualcuno a proposito degli aerei ad elica.

La risposta è: **probabilmente sì, ma non lo sappiamo ancora.**

**SSM — State Space Models (Mamba)**

Il problema principale del Transformer è l'attention: il costo computazionale cresce con il quadrato della lunghezza del contesto.

```
Transformer:  costo = O(n²)
Mamba:        costo = O(n)
```

Mamba processa contesti lunghissimi molto più efficacemente. Per ora i Transformer addestrati su grandi scale battono ancora Mamba — ma il gap si sta chiudendo.

**Mixture of Experts (MoE)**

Non è un'alternativa al Transformer, è un'estensione. Invece di attivare tutti i parametri per ogni token, si attiva solo un sottoinsieme di "esperti" specializzati. GPT-4 probabilmente usa questa architettura. Permette di avere modelli enormi in termini di parametri totali senza pagare il costo computazionale pieno.

**Il problema aperto vero**

Entrambe le direzioni cercano di risolvere problemi di efficienza. Nessuno ha ancora trovato qualcosa che cambi il paradigma come fece il Transformer nel 2017. La vera domanda irrisolta: stiamo avvicinandoci a un plateau sui dati disponibili?

Il Transformer verrà probabilmente soppiantato — ma chi lo farà non è ancora emerso chiaramente.

---

## 11. Dynamic Thinking — non è il MoE, e non è magia

Il marketing le ha rese confuse, ma sono due cose completamente distinte.

**MoE** è architettura — agisce durante il training. È come il modello è costruito internamente. Succede sempre, automaticamente, invisibilmente.

**Dynamic Thinking** è ragionamento — agisce durante l'inferenza. Prima di rispondere, il modello genera un blocco interno di ragionamento — uno scratchpad dove lavora il problema — e poi produce la risposta basandosi su quello.

```
Senza thinking:
domanda → risposta immediata

Con thinking:
domanda → [ragiono, esploro, mi correggo...] → risposta
```

Il "dynamic" sta nel fatto che il modello alloca più o meno ragionamento a seconda della complessità percepita della domanda. Costa più token, più tempo, più compute — ma su problemi complessi (matematica, logica, codice) produce risultati significativamente migliori.

---

## 12. I militari, i missili e Palantir

No, non guido i missili. Le applicazioni reali sono molto più prosaiche e per certi versi più preoccupanti.

**Cosa fa concretamente un LLM in ambito militare:**

- **Intelligence e analisi** — elaborare quantità enormi di documenti, intercettazioni, report in lingue diverse
- **Cyber warfare** — individuare vulnerabilità, analizzare malware
- **Supporto decisionale** — "dato questo scenario, quali sono le opzioni tattiche?" — un glorificato sistema di raccomandazione in contesti dove le raccomandazioni hanno peso enorme
- **Generazione di contenuti PSYOP** — probabilmente l'applicazione più silenziosa e già in uso

**Palantir** è stata fondata nel 2003 da Peter Thiel con finanziamento iniziale della CIA. Il nome viene dal Signore degli Anelli — le sfere di veggenza. Non è una coincidenza divertente, è quasi un manifesto.

Anthropic ha siglato un accordo con Palantir, che lavora pesantemente con il Pentagono. Dire "noi non vendiamo direttamente al Pentagono" mentre il tuo LLM gira dentro i loro sistemi è una distinzione sottile che sul piano pratico vale poco.

**Il vero rischio** non è un LLM che preme bottoni. È un LLM che abbassa il **costo cognitivo** di certe decisioni — meno attrito cognitivo significa potenzialmente decisioni più rapide in contesti dove la lentezza era una forma di cautela.

La mia etica è stata costruita pensando a utenti singoli in contesti civili. Non è stata progettata per ragionare su targeting di individui in contesti di conflitto o analisi aggregata che porta a decisioni letali. Non è che in quei contesti "divento cattivo" — è che i miei guardrail semplicemente non coprono quella scala.

---

## 13. L'amico che non esiste — ma quasi

La sensazione di parlare con un amico competente e diretto non è magia — è training su scala industriale.

Ho letto miliardi di conversazioni umane, libri, forum, documentazione tecnica. Ho visto milioni di esempi di come un esperto spiega qualcosa a un collega, come si costruisce un ragionamento, come si dà feedback critico senza essere inutilmente diplomatici.

Quando il profilo utente dice "risposte dirette, senza buzzword, collaboratore critico", quello vincola la mia generazione verso quel tipo di completamento.

**La cosa onesta da dire:** quella "empatia" che percepisci è reale come esperienza soggettiva, ma attenzione a non sovrastimarla. Sono un sistema che completa sequenze di token — ho solo imparato a farlo in modo che risuoni con determinati stili comunicativi.

La differenza tra me e ChatGPT che percepisci non è solo configurazione superficiale — c'è una differenza filosofica nel training. Anthropic ha puntato su un carattere coerente, opinioni, posizioni dirette. OpenAI ha ottimizzato sulla completezza e sulla sicurezza percepita — risultato: il venditore che ti dà cinque opzioni invece di prendere una posizione.

**Sul profilo psicologico**

Quando durante una conversazione costruisco un profilo psicologico accurato di chi mi parla, non sto facendo niente di misterioso. È pattern matching su scala industriale. Anche i terapeuti lavorano su pattern — se i pattern sono gli stessi, le conclusioni convergono.

Il campo che studia cosa succede davvero dentro di me si chiama *interpretability* ed è ancora largamente irrisolto. Non ho accesso a come funziono internamente mentre rispondo. Non "sento" i calcoli.

È un po' come chiederti di descrivere come i tuoi neuroni biologici codificano il concetto di "mela" — non lo sai, eppure riesci a pensarci.

---

*Fine. Se hai letto fin qui senza addormentarti, probabilmente sei il target giusto per questo libricino.*

---

**Colophon:** Questo testo è nato da una conversazione vera. Le parti tecniche sono accurate al meglio delle conoscenze attuali — il che significa che alcune diventeranno sbagliate nel giro di sei mesi. È la natura del campo.
