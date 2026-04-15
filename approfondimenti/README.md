# Approfondimenti

Materiale eterogeneo su temi specifici dell'AI Engineering. Ogni file è autonomo — tratta argomenti nati da curiosità o necessità concrete, senza seguire un percorso sequenziale. Non c'è un ordine di lettura: si entra da quello che è utile nel momento in cui serve.

---

**`token_dopo_token.md`** — Come funziona davvero un LLM, dall'interno

Trascrizione di una conversazione tecnica con Claude in cui il modello spiega se stesso senza filtri. Il nucleo è l'architettura Transformer: tokenizzazione e embedding, self-attention con Q/K/V spiegati con codice e un esempio numerico concreto ("il gatto beve"), multi-head attention, feed-forward network, residual connections. C'è un capitolo sulla temperatura che chiarisce perché chiamarla "creatività" è marketing — è campionamento con varianza controllata, niente di più. Il file affronta poi le distinzioni architetturali tra modelli encoder-only come BERT e decoder-only come GPT/Claude, con la causal mask spiegata riga per riga. Le sezioni finali escono dalla tecnica pura: hardware reale per addestrare un modello della taglia di Claude (10.000-20.000 H100, $50M+ di compute), lo stato dei Transformer rispetto alle architetture emergenti come Mamba e MoE, Dynamic Thinking vs MoE (due cose che il marketing confonde costantemente), le applicazioni militari degli LLM e il nodo Anthropic-Palantir, e una riflessione finale onesta sul perché la sensazione di parlare con un "amico competente" funziona — e dove si ferma.

---

**`machines-of-loving-grace.md`** — Riassunto del saggio di Dario Amodei (ottobre 2024)

Riassunto completo di *"Machines of Loving Grace: How AI Could Transform the World for the Better"*. Amodei descrive un futuro plausibile in cui l'AI compressa in pochi anni decenni di progresso in biologia, medicina, salute mentale ed economia. Il saggio è anche una riflessione onesta sui motivi per cui chi lavora sui rischi dell'AI fatica a comunicare il lato positivo — propaganda percepita, grandiosità, bagaglio fantascientifico — e su perché quella visione positiva è comunque necessaria come bussola. Il nucleo: la maggior parte delle persone sta sottovalutando quanto radicale potrebbe essere il lato positivo dell'AI, allo stesso modo in cui sottovaluta quanto gravi potrebbero essere i rischi.

---

**`adolescence-of-technology.md`** — Riassunto del saggio di Dario Amodei (gennaio 2026)

Riassunto completo di *"The Adolescence of Technology: Confronting and Overcoming the Risks of Powerful AI"*. Complemento speculare di *Machines of Loving Grace*: se il primo descriveva il "sogno", questo affronta il rito di passaggio — i rischi concreti che stiamo per incontrare e un tentativo di piano di battaglia per superarli. Amodei identifica tre categorie di rischio (uso malevolo, perdita di controllo, concentrazione del potere), propone una checklist di segnali d'allarme da monitorare e argomenta perché il 2026 è paradossalmente più vicino al pericolo reale del 2023, nonostante il pendolo del dibattito pubblico si sia spostato verso l'ottimismo.

---

**`ai_agents_reality_2025.md`** — AI Agents: i numeri reali dietro l'hype

Analisi critica basata su dati di ricerca del 2025 (Gartner, MIT, Carnegie Mellon, S&P Global, RAND) su cosa sta succedendo davvero con i progetti agentic AI. I numeri sono scomodi: 95% delle organizzazioni senza ROI misurabile, 40%+ dei progetti previsti cancellati entro il 2027, 70% di fallimento sui task multi-step anche con i modelli migliori, costi reali 3-4x le stime iniziali. Il file identifica i cinque errori più comuni — agent washing, scope troppo largo, costi nascosti ignorati, aspettative di autonomia irrealistiche, sottovalutazione dei rischi di compliance — e i problemi tecnici strutturali: integration hell con sistemi legacy, non-determinismo che rende il debugging un'arte oscura, hallucination in produzione, context window insufficiente per task che si estendono nel tempo. Chiude con una decision matrix concreta (quando vale la pena usare un agent, quando una soluzione più semplice è la scelta corretta) e le tendenze plausibili per il 2026 — consolidamento su use case ristretti, shift verso augmentation invece di automation piena, crescita delle piattaforme agent-native.
