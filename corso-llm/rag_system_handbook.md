# RAG System Development Journey

**Decisioni architetturali, problemi reali e soluzioni pratiche**

---

## Prefazione

Questo libro documenta il processo di costruzione di un sistema RAG per documenti finanziari, concentrandosi esclusivamente sulle **scelte architetturali**, i **dubbi emersi**, i **problemi incontrati** e le **soluzioni implementate**.

Non troverai:
- Descrizioni dettagliate del progetto
- Tutorial passo-passo
- Codice completo
- Spiegazioni di base su cos'è un RAG

Troverai:
- Discussioni sulle scelte tecnologiche
- Ragionamenti dietro ogni decisione
- Problemi reali affrontati
- Alternative considerate e scartate
- Lezioni apprese

L'approccio è pragmatico: quando una soluzione è "sufficiente" per lo scope lo diciamo chiaramente, quando qualcosa è over-engineering lo evidenziamo.

---

## Capitolo 1: Le Prime Decisioni Architetturali

### Il dilemma: LangChain o custom?

**Contesto iniziale**: Avevamo completato un modulo su LangChain e dovevamo decidere se usarlo per il progetto RAG.

**Dubbio espresso**: "Usiamo LangChain completo o andiamo custom? Non voglio sembrare uno che ha solo seguito tutorial, ma nemmeno voglio reinventare la ruota."

**Discussione**: 
- Pro LangChain: framework maturo, documentato, accelera sviluppo
- Contro LangChain: dipendenza pesante, rischio di sembrare "tutorial follower"
- Pro custom: dimostra capacità architetturali
- Contro custom: tempo sprecato a reimplementare cose già fatte

**Soluzione adottata**: Approccio ibrido
- Riutilizziamo `langchain-text-splitters` per chunking (testato, maturo)
- Custom code per tutto il resto (retrieval, generation, API)

**Razionale**: Dimostra pragmatismo professionale - riusare librerie mature per utilities, mantenere controllo sul core business logic.

### FAISS vs Qdrant: la questione dello scale

**Dubbio**: "FAISS è troppo semplice? Non dovremmo usare qualcosa di più enterprise come Qdrant?"

**Contesto**: Il progetto aveva scope limitato (~3 documenti per test/demo), ma doveva essere presentabile come portfolio professionale.

**Discussione sullo scale**:
- FAISS IndexFlatIP: 100% precision, zero infrastruttura, O(n) per query
- Qdrant: enterprise-grade, distributed, ma richiede Docker/cloud setup
- IVF/HNSW: necessari solo sopra 10k+ documenti

**Momento chiave**: "Per 3 documenti FAISS è perfetto. IVF sarebbe overkill."

**Proposta iniziale**: "Facciamo comunque Qdrant per imparare e mostrare capacità?"

**Risposta critica**: "No. Scegliere tecnologia appropriata al problema è più importante che flexare tech figa. Per portfolio, dimostra GIUDIZIO."

**Soluzione**: FAISS + documentazione chiara nel README su quando migrare a Qdrant (>10k docs, distributed deployment, multi-tenancy).

**Lezione**: Scope-appropriate > over-engineering per impressionare.

### Embeddings: pagare API o andare local?

**Domanda**: "OpenAI embeddings o SentenceTransformers locale?"

**Preferenza espressa**: "Non voglio dare soldi a OpenAI se posso evitare."

**Discussione sui trade-off**:

OpenAI embeddings:
- Pro: qualità alta, nessun setup
- Contro: costo per chiamata, latency API (100-200ms), 1536 dimensioni

SentenceTransformers (all-MiniLM-L6-v2):
- Pro: free, locale, privacy, bassa latency (10-50ms), 384 dimensioni
- Contro: qualità leggermente inferiore per edge cases

**Considerazione**: "384 dimensioni sono sufficienti? Non perdiamo troppa qualità?"

**Risposta**: "Per il 90% dei casi la qualità è più che sufficiente. Se servisse migliore, esistono modelli più grandi."

**Verifica**: Il modello è multilingual, quindi italiano supportato nativamente.

**Decisione**: SentenceTransformers con possibilità di upgrade a modelli migliori se necessario.

---

## Capitolo 2: Chunking - Il Problema del Contesto

### La scoperta del problema overlap

**Discussione iniziale**: "Perché serve l'overlap? Non basta spezzare il testo?"

**Esempio pratico discusso**:
```
Senza overlap:
CHUNK 1: "Ricavi Q4 2023 €50M, +15% vs Q4 2022."
CHUNK 2: "Crescita dovuta a espansione Asia."

Query: "Perché ricavi Q4 cresciuti?"
Problema: I due pezzi di informazione sono separati
```

**Soluzione overlap**:
```
CHUNK 1: "...ricavi Q4 2023 €50M, +15% vs Q4 2022."
CHUNK 2: "...+15% vs Q4 2022. Crescita dovuta a espansione Asia."
```

**Momento di comprensione**: "Ah ok, l'overlap fa da 'colla' tra i chunk per mantenere il contesto."

### Quanto overlap è giusto?

**Domanda**: "Ma quanto overlap serve? Troppo non spreca risorse?"

**Discussione sui parametri**:
- Troppo poco (<5%): rischio di tagliare concetti critici
- Troppo (>50%): spreco computazionale e di memoria
- Sweet spot: 15-25%

**Decisione**: chunk_size=1000, overlap=200 (20%) come baseline.

**Nota pragmatica**: "Questi valori vanno tunati in base ai risultati dell'evaluation."

---

## Capitolo 3: Provider Abstraction - Evitare Lock-in

### Il problema del vendor lock-in

**Scenario presentato**: "Se hardcodiamo chiamate Anthropic ovunque e poi vogliamo passare a OpenAI?"

**Risposta**: "Dobbiamo riscrivere tutto."

**Obiezione**: "Ma tanto uso solo Claude, serve davvero astrarre?"

**Contro-argomento**: 
1. In produzione i costi possono cambiare
2. Un provider può avere downtime
3. Modelli diversi per use case diversi
4. Per portfolio dimostra design thinking

**Momento decisivo**: "Ok, facciamo l'astrazione. Anche perché nel progetto chatbot precedente l'avevamo già discussa."

### Riuso del lavoro precedente

**Proposta**: "Nel progetto chatbot abbiamo già fatto i provider. Possiamo riusarli?"

**Risposta entusiasta**: "Perfetto! È esattamente così che si fa - riutilizzare componenti tra progetti."

**Componenti riusati**:
- BaseLLMProvider (abstract interface)
- AnthropicProvider
- OpenAIProvider
- Message e CompletionResponse dataclasses

**Beneficio**: Zero vendor lock-in + cost tracking + unified interface.

---

## Capitolo 4: FastAPI Architecture - La Discussione Seria

### Primo tentativo: controller monolitico

**Approccio iniziale**: Controller/routes in pochi file con logica business inline.

**Reazione**: "Non mi piace. Voglio service/controller separation come si deve, non questa roba a metà."

**Obiezione**: "Per un progetto piccolo non è overkill?"

**Risposta ferma**: "No. Se devo metterlo su GitHub come portfolio, deve dimostrare capacità architetturali vere."

### Rifacimento completo

**Richiesta specifica**: "Architettura professionale con service e controller per separazione della logica."

**Ammissione onesta**: "Hai ragione - il codice che ti ho dato era un monolite da tutorial, non da backend developer serio."

**Nuova struttura discussa**:
- Controllers: thin HTTP layer
- Services: business logic (no HTTP knowledge)
- Models: Pydantic request/response
- Dependencies: dependency injection
- Exceptions: custom error handling

### Dependency injection

**Discussione sul pattern**:
- Singleton per servizi stateful (RAG system)
- FastAPI Depends per injection automatica
- Beneficio: testabilità senza HTTP

**Esempio pratico**:
```python
def get_rag_service() -> RAGService:
    # Singleton instance
    
async def query(service: RAGService = Depends(get_rag_service)):
    # FastAPI inietta automaticamente
```

### Custom exceptions

**Problema identificato**: "Come gestiamo errori in modo consistente?"

**Soluzione discussa**:
- RAGException base class
- Specifiche: SystemNotInitializedException, NoDocumentsException, ecc.
- Exception handler centralizzato

**Beneficio**: Errori consistenti, codice pulito nei controllers.

---

## Capitolo 5: Streamlit e FastAPI - Il Fraintendimento

### Il cambio improvviso

**Contesto**: Dopo aver discusso FastAPI in dettaglio, propongo improvvisamente Streamlit per l'interfaccia.

**Reazione immediata**: "Come mai prima dici FastAPI e poi di colpo Streamlit? C'è una motivazione o ti sei confuso?"

**Ammissione**: "Hai ragione - non ho spiegato. Ho tirato fuori Streamlit senza contesto."

### Chiarimento

**Spiegazione**:
- FastAPI = backend serio, API REST, produzione
- Streamlit = demo veloce per README, screenshot, video

**Dubbio**: "Ma quindi quale facciamo?"

**Proposta dell'utente**: "Facciamo così: Streamlit per chi vuole provare al volo, FastAPI perché serve a me per studiare un sistema completo."

**Risposta**: "Ottima idea! È esattamente come si fa nei progetti veri."

**Soluzione finale**: Entrambi
- `api/` = FastAPI production-ready
- `app.py` = Streamlit per quick demo

---

## Capitolo 6: Evaluation - "È Una Pippa Mentale?"

### La domanda diretta

**Contesto**: Dopo aver presentato uno script di evaluation completo.

**Domanda senza filtri**: "Non ho capito a cosa serve. È una pippa mentale?"

**Risposta onesta richiesta**.

### Discussione sull'utilità reale

**Per uso quotidiano (3 documenti, personale)**: Un po' sì, è overkill.

**Ma ci sono 3 scenari dove serve**:

1. **Colloqui**
- Domanda: "Come hai validato la qualità?"
- Con metriche: "Retrieval success 87.5%, avg score 0.68"
- Senza metriche: "L'ho provato e funziona"

2. **Debugging**
```
Evaluation result:
avg_retrieval_score: 0.32  ← PROBLEMA!

Diagnosi: chunk_size troppo grande
Fix: ridurre da 1000 a 600
```

3. **Confronto provider**
- Claude vs GPT, quale costa meno/va meglio?
- Dati oggettivi per decidere

### Soluzione pragmatica

**Proposta**: "Invece dello script formale, facciamo una versione pratica."

**Risultato**: `quick_test.py`
- Testa query reali
- Output leggibile (GOOD/OK/BAD)
- Raccomandazioni concrete
- Usa 1 volta prima colloquio, copi numeri nel README

**Principio**: Utilità pratica > perfezione teorica.

---

## Capitolo 7: Il Problema delle Tabelle Finanziarie

### Il problema emerge

**Contesto**: "Caricando documenti vedo che ha difficoltà a decifrare le tabelle del conto economico e stato patrimoniale."

**Esempio fornito**: Script con pdfplumber per estrarre tabelle e convertirle in JSON.

### Prima reazione: debug del problema

**Proposta iniziale**: "Prima capiamo DOVE si rompe. Testiamo l'estrazione."

**Test richiesto**: Verificare se pypdf/pdfplumber estraggono le tabelle.

**Output del test**: 
- Tabelle trovate ma in formato markdown
- Markdown per tabelle larghe illeggibile
- Numeri ci sono ma struttura pessima

### L'escalation

**Dopo varie proposte incrementali**...

**Reazione**: "Invece di fare seghe, mi fai uno script che rende leggibili queste tabelle?"

**Risposta a questa richiesta**: "Hai ragione. Basta seghe."

**Primo script**: Estrazione standalone che crea file txt leggibile.

### Ancora non basta

**Feedback**: "Non riesci a fare un bel lavoro, invece che questa roba a metà?"

**Richiesta specifica**: "Parser del PDF specifico per bilanci che quando incontra tabelle le sostituisce con quelle leggibili."

**Comprensione**: Non serviva script standalone, serviva integrazione completa nel RAG.

### Soluzione finale

**FinancialPDFProcessor completo**:
- Rileva sezioni finanziarie (stato patrimoniale, conto economico)
- Estrae tabelle con settings ottimizzati
- Converte in formato natural language
- Mantiene testo narrativo
- Integrato nel flusso RAG

**Metodo dedicato**:
```python
rag.index_financial_document("bilancio.pdf")  # Parser specializzato
rag.index_documents(["report.pdf"])           # Parser standard
```

**Lezione appresa**: Per domain-specific documents servono parser specializzati, non soluzioni generiche.

---

## Capitolo 8: Postman Collection - Testing Pratico

### La richiesta

**Domanda**: "Mi scrivi la collection Postman per testare i servizi? Con esempi precompilati per modificare i dati."

**Obiettivo**: Testare API senza scrivere codice, avere esempi concreti.

### Organizzazione discussa

**Struttura proposta**:
- Health (health check)
- System (initialize, status)
- Documents (upload, list, clear)
- Query (esempi multipli in italiano)
- Complete Workflow (1-6 step by step)

**Esempi query**:
- Revenue growth
- Customer satisfaction
- EBITDA margins
- Geographic expansion
- R&D investments
- Debt position
- ESG initiatives

**Ogni esempio**: Pre-configurato, user modifica valori e testa subito.

### Variables environment

**Discussione su configurazione**:
- `base_url`: per cambiare host/porta
- `anthropic_api_key`: per API key
- `openai_api_key`: alternativa

**Beneficio**: Cambi una volta, funziona ovunque nella collection.

---

## Capitolo 9: Documentazione e README

### Approach discusso

**Obiettivo**: README che vende il progetto a recruiter.

**Elementi essenziali**:
1. Features highlight
2. Quick start chiaro
3. Architecture diagram
4. Scelte architetturali spiegate
5. Migration paths (quando scalare)

### Il principio dell'onestà

**Discussione**: "Come presentiamo le scelte tecniche?"

**Approccio consigliato**: Onestà sulle limitazioni e alternative.

Esempio:
> "FAISS è appropriato per questo scope (<1000 documenti). Per scale enterprise (>10k documenti) considerare Qdrant con IVF indexing."

**Perché funziona**: Dimostra che SAI quando scalare, non che hai paura di ammettere limitazioni.

### Tone professionale ma accessibile

**Bilanciamento**:
- Non tutorial per principianti
- Non paper accademico
- Professional ma leggibile
- Decisioni spiegate, non solo "cosa"

---

## Capitolo 10: Il Libro - Meta-Documentazione

### La richiesta finale

**Domanda**: "Crea documento della chat in formato markdown: un libro che posso tenere come riferimento."

**Requisiti specifici**:
- Riorganizzare contenuti in modo organico
- Linguaggio informale come nella chat
- Includere domande e risposte
- Sezione lessons learned

### Primo tentativo

**Problema**: Libro con linguaggio troppo informale (parolacce).

**Feedback**: "Togli le parolacce, non posso presentare un libro così!"

### Secondo feedback

**Richiesta di focus**: "Includi SOLO concetti discussi, dubbi, problemi e soluzioni. Non mi interessa descrizione progetto o codice dettagliato."

**Comprensione**: Il valore è nelle DISCUSSIONI, non nel codice finale.

**Obiettivo**: Documentare il PROCESSO di pensiero, non il risultato.

---

## Lessons Learned

### 1. Scope Appropriato > Over-Engineering

**Situazione**: Discussione FAISS vs Qdrant per 3 documenti.

**Tentazione**: Usare Qdrant per sembrare più "enterprise".

**Lezione**: Scegliere tecnologia appropriata al problema dimostra più competenza che usare quella più "figa".

**In colloqui**: "Ho usato FAISS perché appropriato per questo scope; so quando migrare a Qdrant per scale maggiore."

### 2. Provider Abstraction è Investimento

**Situazione**: Dubbio se astrarre provider per progetto "piccolo".

**Resistenza iniziale**: "Tanto uso solo Claude."

**Lezione**: Astrazione costa poco upfront, evita riscritture massive dopo.

**Beneficio reale**: Permette confronti costo/performance tra provider.

### 3. Clean Architecture ≠ Over-Engineering

**Situazione**: Richiesta di service/controller separation.

**Prima reazione**: "Per progetto piccolo non è overkill?"

**Lezione**: Separation of concerns è professionalità, non spreco.

**Risultato**: Codice testabile, manutenibile, riusabile anche per progetti "piccoli".

### 4. Iterazione Rapida Batte Perfezione

**Situazione**: Problema tabelle finanziarie.

**Percorso**:
1. Markdown standard → fallito
2. Natural language format → meglio
3. Parser specializzato → ottimo

**Lezione**: Ship, test, improve > pensare settimane alla soluzione perfetta.

### 5. Domain Knowledge Matters

**Situazione**: Documenti finanziari con tabelle complesse.

**Tentazione**: Parser generico per tutti i PDF.

**Lezione**: Domain-specific documents richiedono parser specializzati.

**Generalizzabile**: Legal docs, medical records, technical specs → tutti domain-specific.

### 6. Feedback Diretto > Approvazione

**Situazione**: "Non riesci a fare un bel lavoro invece che questa roba a metà?"

**Risultato**: Discussione onesta → soluzione migliore.

**Principio**: Confronto diretto produce risultati migliori di assecondare sempre.

**Applicabile a**: Code review, design decisions, tech stack choices.

### 7. Documentare le Decisioni

**Situazione**: Ogni scelta tecnica discussa con pro/contro.

**Valore**: Non solo "cosa" ma "perché".

**Formato**: README + Architecture Decision Records.

**Beneficio**: Onboarding rapido, decisioni tracciabili, no "cargo cult".

### 8. Riuso Intelligente

**Situazione**: Provider abstraction già fatta in progetto chatbot precedente.

**Azione**: Riutilizzato invece di riscrivere.

**Lezione**: Componenti ben progettati si riusano tra progetti.

**Evita**: Not Invented Here syndrome.

### 9. Testing Con Dati Reali

**Situazione**: Test su PDF bilancio vero vs mock.

**Lezione**: Mock perfetti non riflettono problemi reali (tabelle complesse).

**Principio**: Test con dati rappresentativi sempre.

### 10. Evaluation per Decisioni Data-Driven

**Situazione**: "È una pippa mentale?" → discussione onesta sull'utilità.

**Chiarimento**: Per uso quotidiano forse sì, per colloqui/debugging no.

**Principio**: Metriche servono per decisioni oggettive, non per formalismo.

### 11. Onestà Sulle Limitazioni

**Situazione**: Come presentare scelte nel README.

**Approccio**: "FAISS appropriato qui, per scale maggiore Qdrant."

**Lezione**: Ammettere limitazioni e sapere alternative dimostra maturità.

**Red flag**: Pretendere che la propria soluzione sia perfetta per tutto.

### 12. Pragmatismo Nei Tool

**Situazione**: Usare langchain-text-splitters ma non tutto LangChain.

**Principio**: Riusa librerie mature per utilities, custom per core logic.

**Evita**: Purismo tecnologico che rallenta sviluppo.

### 13. Cost Tracking è Feature Core

**Situazione**: Ogni query traccia token e costo.

**Motivazione**: Budget control, provider comparison, optimization.

**Lezione**: Observability su costi è necessaria, non nice-to-have.

### 14. Separation of Concerns Sempre

**Situazione**: Controllers thin, services con business logic.

**Beneficio**: Testabilità, riusabilità (services in CLI, background jobs).

**Principio**: Anche per "progetti piccoli", mantenere layer separation.

### 15. Il Processo È Documentazione

**Situazione**: Questa conversazione diventa libro.

**Valore**: Decisioni + rationale + alternative considerate = oro.

**Formato**: ADR, design docs, postmortems.

**Beneficio**: Onboarding, knowledge sharing, evitare decisioni "cargo cult".

### 16. Portfolio ≠ Production

**Situazione**: FAISS per portfolio vs Qdrant per production scale.

**Lezione**: Portfolio dimostra giudizio, non solo capacità tecniche.

**In colloqui**: "Ho scelto X perché scope Y, so quando scalare a Z."

### 17. Dependencies Minimaliste

**Situazione**: Ogni dependency è debito tecnico.

**Principio**: Aggiungere solo il necessario.

**Evita**: "Uso framework X perché tutti lo usano" senza valutare alternative.

### 18. Multilingual Non È Gratis

**Situazione**: Verifica che all-MiniLM-L6-v2 supporti italiano.

**Lezione**: Sempre testare con lingua target, non assumere.

**Applicabile**: Timezone, currency, date format, ecc.

### 19. Chunking Influenza Qualità

**Situazione**: chunk_size e overlap critici per retrieval.

**Lezione**: Parametri vanno tunati con evaluation, non scelti random.

**Processo**: Default ragionevole → test → itera basato su metriche.

### 20. Comunicazione Diretta Accelera

**Situazione**: Feedback schietti su soluzioni incomplete.

**Risultato**: Meno iterazioni, soluzioni migliori più velocemente.

**Principio**: Onestà > diplomaticità quando si lavora su codice.

---

## Conclusione

Questo libro ha documentato non il sistema RAG finale, ma il **processo** per arrivarci:

- Le scelte architetturali e il loro perché
- I dubbi e come sono stati risolti  
- I problemi reali e le soluzioni iterative
- Le discussioni che hanno portato a decisioni migliori

Il valore non è nel codice (che cambierà), ma nei **principi** applicati:
- Scope-appropriate over over-engineering
- Iterazione rapida con feedback onesto
- Decisioni documentate con rationale
- Pragmatismo nei tool, rigore nell'architettura

Questi principi sono applicabili a qualsiasi progetto software, non solo sistemi RAG.

---

*Fine del libro. Il codice è su GitHub. Le decisioni sono qui.*
