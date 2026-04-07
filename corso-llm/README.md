# Corso LLM

Questo è il materiale di studio accumulato durante lo sviluppo di ARIA, un assistente di analisi finanziaria costruito su MAOTrade. Il percorso non nasce da un piano didattico preconfezionato: ogni modulo è il risultato di una sessione di studio interattiva con un LLM, affrontando problemi concreti man mano che emergevano nel progetto. Il tono è diretto e a tratti informale, perché rispecchia il modo in cui le sessioni si sono svolte. Non si tratta di documentazione ufficiale né di un corso strutturato a priori — è una raccolta di note operative, decisioni architetturali, errori fatti e ragionamenti intermedi. Chi legge trovera sia la teoria che serve per capire perché le cose funzionano in un certo modo, sia il codice e i pattern che si usano effettivamente in produzione.

---

## Contenuto

**`llm_fundamentals.md`** — LLM Fundamentals + API Setup

Il punto di partenza del percorso. Spiega come funziona un LLM senza filtri: la predizione probabilistica token per token, la differenza tra training e inference, il concetto di context window e i suoi limiti pratici. C'è una sezione dedicata a tokens e costi — perché ogni chiamata API si paga e ignorare questi numeri porta dritti al budget esaurito. Il confronto tra Claude e GPT è fatto con dati concreti, non con preferenze di marketing. La parte finale copre prompt engineering essenziale (few-shot, chain of thought, role prompting) e i limiti fondamentali degli LLM che qualsiasi sistema serio deve tenere in conto: hallucination, nessun accesso a dati privati, nessuna conoscenza real-time.

---

**`ai_chat-production_ready_guide.md`** — AI Chat Package Production-Ready

Costruzione di un package `ai_chat` da zero, senza LangChain. Il file parte dall'analisi di LangChain — cosa fa, come funziona, la sua architettura — e spiega perché è stato scartato a favore di codice custom: instabilità delle API, dipendenza pesante, difficoltà di debug. Il package risultante supporta multipli provider LLM con interfaccia unificata, gestisce semantic caching per ridurre i costi, mantiene la conversation memory in modo persistente e include error handling serio con retry e rate limiting. Ogni scelta architetturale è accompagnata dal ragionamento che l'ha prodotta, incluso il costo di quella scelta in termini di complessità.

---

**`streaming_guide.md`** — HTTP Streaming e LLM: Guida Completa

Guida tecnica dedicata allo streaming, nata dal problema concreto di ARIA: query complesse su documenti finanziari che richiedono 10-15 secondi di generazione. Il file spiega le tre tecnologie di streaming HTTP (chunked transfer, Server-Sent Events, WebSocket) con i pro e contro reali di ciascuna, poi entra nel dettaglio di come funziona lo streaming negli SDK LLM e confronta l'approccio SDK con raw HTTP. La sezione su FastAPI copre i pattern pratici per implementare stream tipizzati con JSON, la gestione degli errori mid-stream e la cancellazione delle richieste. Include una production checklist con i problemi che si trovano solo quando si va live.

---

**`rag_theory_vector_databases.md`** — RAG Theory & Vector Databases

Copertura teorica completa di RAG: i tre limiti strutturali degli LLM che RAG risolve (knowledge cutoff, nessun dato privato, hallucination), il confronto onesto con le alternative (fine-tuning e long context), e l'architettura del sistema spiegata passo per passo. La parte centrale è un confronto tecnico approfondito tra i principali vector database — FAISS, Qdrant, Pinecone, pgvector — con tabelle di performance reali in funzione del numero di vettori, latenza, memoria e recall. Il file prosegue con Qdrant in dettaglio, una sezione sugli embeddings (compatibilità, dimensioni, modelli locali vs API) e i pattern avanzati di retrieval come il reranking.

---

**`rag_system_handbook.md`** — RAG System: Decisioni Architetturali e Problemi Reali

Diario di costruzione del sistema RAG per documenti finanziari. A differenza degli altri file, qui non c'è teoria: ci sono le decisioni prese, i dubbi che le hanno precedute e le ragioni che le hanno determinate. LangChain vs custom (soluzione ibrida, con langchain-text-splitters per il chunking e tutto il resto custom), FAISS vs Qdrant per lo scope del progetto, SentenceTransformers vs OpenAI embeddings. Un capitolo intero è dedicato al problema delle tabelle finanziarie nei PDF — un problema reale che ha richiesto tre iterazioni prima di arrivare a un parser specializzato per bilanci. Il file si chiude con 20 lessons learned che sintetizzano i principi emersi dal progetto: scope appropriato, iterazione rapida, documentazione delle decisioni, pragmatismo nei tool.

---

**`ai_agents_guida_pratica.md`** — AI Agents: Guida Pratica Senza Bullshit

Demistificazione degli AI Agents, partendo dalla differenza strutturale tra chain e agent: in una chain il programmatore decide il flusso, in un agent è l'LLM che decide dinamicamente quali tool usare e in che ordine. Il file spiega il meccanismo del function calling dall'interno — come l'LLM "vede" i tool, come costruisce le chiamate, come gestisce i risultati — e i pattern di orchestrazione principali (ReAct, Plan-and-Execute, Multi-Agent). La parte più utile è quella sui limiti e i failure modes: quando gli agents si inceppano, quanto costano in latenza e token, e in quali scenari una chain predefinita è la scelta corretta. Include un decision tree pratico e dati numerici su costi e performance.

---

## Ordine di lettura consigliato

1. `llm_fundamentals.md`
2. `ai_chat-production_ready_guide.md`
3. `streaming_guide.md`
4. `rag_theory_vector_databases.md`
5. `rag_system_handbook.md`
6. `ai_agents_guida_pratica.md`
