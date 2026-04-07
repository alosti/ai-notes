# AI Agents: Guida Pratica Senza Bullshit

Una guida onesta e pratica su quando (e quando NON) usare gli AI Agents, scritta per developer che vogliono capire davvero come funzionano le cose.

---

## Indice

1. [Introduzione](#introduzione)
2. [Cosa Sono gli Agents (Davvero)](#cosa-sono-gli-agents-davvero)
3. [Tool Use / Function Calling](#tool-use--function-calling)
4. [Pattern di Orchestrazione](#pattern-di-orchestrazione)
5. [Quando Usare Agents vs Chains](#quando-usare-agents-vs-chains)
6. [Limiti e Failure Modes](#limiti-e-failure-modes)
7. [Classificazione Intent per Routing](#classificazione-intent-per-routing)
8. [Function Calling: Come Funziona Sotto al Cofano](#function-calling-come-funziona-sotto-al-cofano)
9. [Lessons Learned](#lessons-learned)

---

## Introduzione

Gli AI Agents sono probabilmente la tecnologia più **sovra-venduta** del 2024. Il marketing AI ti fa credere che siano la soluzione a tutto, quando nella realtà dei fatti sono costosi, lenti, e spesso inutili per il 90% dei problemi.

Questa guida nasce da una domanda semplice: **quando ha davvero senso usare un agent invece di logica più semplice?**

La risposta breve: molto meno spesso di quanto pensi.

La risposta lunga: continua a leggere.

**Cosa troverai qui:**

- Spiegazioni tecniche senza marketing bullshit
- Esempi pratici con codice vero
- Numeri reali su costi e performance
- Decision tree chiare su quando usare cosa
- I failure modes che nessuno ti racconta

**Cosa NON troverai:**

- Hype su "AGI is coming"
- Tutorial LangChain copy-paste
- Promesse miracolose
- Soluzioni one-size-fits-all

Partiamo.

---

## Cosa Sono gli Agents (Davvero)

### La Differenza Fondamentale

Prima di tutto: un agent **NON è** un chatbot avanzato. È qualcosa di strutturalmente diverso.

**Chain (quello che probabilmente già fai):**

```python
def rag_query(question: str) -> str:
    """
    Flusso PREDEFINITO e LINEARE:
    1. Embedding della query
    2. Similarity search nel vector store
    3. Reranking (opzionale)
    4. Generate response con LLM
    """
    docs = vector_store.similarity_search(question)
    context = "\n".join([doc.page_content for doc in docs])
    response = llm.generate(f"Context: {context}\n\nQuestion: {question}")
    return response
```

**Agent (decisione dinamica):**

```python
def agent_query(question: str) -> str:
    """
    Flusso DINAMICO - L'agent DECIDE:
    - Quali tool usare
    - In che ordine
    - Quante volte iterare
    - Quando fermarsi
    """
    agent = create_agent(
        llm=llm,
        tools=[search_tool, calculator_tool, sql_tool]
    )
    # L'agent decide autonomamente il suo piano
    response = agent.run(question)
    return response
```

**La differenza chiave:**

- **Chain**: Il programmatore decide il flusso (if/else hardcoded)
- **Agent**: L'LLM decide il flusso (reasoning loop dinamico)

### Anatomia di un Agent

Un agent ha **4 componenti essenziali**:

```
┌─────────────────────────────────────────────────────────────┐
│                         AGENT                               │
│                                                             │
│  ┌──────────────┐                                           │
│  │   1. BRAIN   │  ← LLM che ragiona e decide               │
│  │  (LLM Core)  │    (GPT-4, Claude, Llama)                 │
│  └──────────────┘                                           │
│         │                                                   │
│         ↓                                                   │
│  ┌──────────────┐                                           │
│  │  2. MEMORY   │  ← Mantiene contesto conversazione        │
│  │  (Context)   │    (chat history, intermediate steps)     │
│  └──────────────┘                                           │
│         │                                                   │
│         ↓                                                   │
│  ┌──────────────┐                                           │
│  │  3. TOOLS    │  ← Azioni che può eseguire                │
│  │  (Functions) │    (API calls, DB queries, calculations)  │
│  └──────────────┘                                           │
│         │                                                   │
│         ↓                                                   │
│  ┌──────────────┐                                           │
│  │ 4. EXECUTOR  │  ← Loop che orchestra tutto               │
│  │ (ReAct Loop) │    (decide, agisce, osserva, ripete)      │
│  └──────────────┘                                           │
└─────────────────────────────────────────────────────────────┘
```

### Esempio Concreto: Agent vs Chain

Immagina questa domanda su un sistema di trading:

**Query:** *"Il mio Sharpe Ratio del 2024 è migliore della media del mercato? Se no, quali trade dovrei evitare per migliorarlo?"*

**Approccio CHAIN:**

```python
# Devi programmare ogni step
def analyze_sharpe():
    # Step 1: Query DB
    trades_2024 = db.query("SELECT * FROM trades WHERE year = 2024")

    # Step 2: Calcola Sharpe
    sharpe = calculate_sharpe_ratio(trades_2024)

    # Step 3: Cerca benchmark (web search? dove?)
    benchmark = get_market_benchmark()  # Da implementare manualmente

    # Step 4: Confronta
    comparison = compare(sharpe, benchmark)

    # Step 5: Identifica worst trades
    worst_trades = find_worst_trades(trades_2024)

    # Step 6: LLM per sintesi
    response = llm.generate(f"Analizza: {comparison}, {worst_trades}")
    return response
```

**Problemi:**

- ✗ Devi anticipare OGNI step
- ✗ Cosa fai se il DB non ha 2024? (errore hardcoded)
- ✗ Cosa fai se serve info aggiuntiva? (ri-programmare)
- ✗ Rigido: non si adatta a variazioni della query

**Approccio AGENT:**

```python
# L'agent decide autonomamente
def analyze_sharpe():
    agent = Agent(
        llm=gpt4,
        tools=[
            DatabaseTool(db_connection),    # Può fare SQL queries
            CalculatorTool(),               # Può fare math
            WebSearchTool(),                # Può cercare info
            PythonREPLTool()                # Può eseguire codice
        ]
    )

    response = agent.run(
        "Il mio Sharpe Ratio del 2024 è migliore della media del mercato? "
        "Se no, quali trade dovrei evitare per migliorarlo?"
    )
    return response
```

L'agent **potrebbe** decidere questo flusso autonomamente:

1. "Hmm, devo sapere Sharpe Ratio 2024" → Usa DatabaseTool
2. "Ok, ho i dati. Ora calcolo Sharpe" → Usa PythonREPLTool
3. "Sharpe è 1.8. Qual è il benchmark mercato 2024?" → Usa WebSearchTool
4. "Benchmark è 1.2. Sono sopra! Ma per rispondere completo..." → Query worst trades
5. "Identificati 3 worst trades. Simulo rimozione" → Ricalcola Sharpe
6. "Ho tutti i dati. Genero risposta finale" → Sintetizza

**Vantaggi:**

- ✓ Autonomo: decide il piano da solo
- ✓ Adattivo: se trova dati mancanti, cerca alternative
- ✓ Robusto: gestisce edge cases
- ✓ Flessibile: funziona con query simili ma diverse

### I 3 Tipi di Agents

**A) REACTIVE AGENTS (ReAct)**

Funzionamento: Loop Observation → Thought → Action → Repeat

```
User: "Quanti trade ho fatto a dicembre 2024?"

Agent reasoning:
Thought: "Devo interrogare il database"
Action: sql_tool("SELECT COUNT(*) FROM trades WHERE date >= '2024-12-01'")
Observation: "Result: 47"
Thought: "Ho la risposta, posso fermarmi"
Final Answer: "Hai fatto 47 trade a dicembre 2024"
```

**Quando usarlo:** Query che richiedono 1-5 step, tool use semplice, feedback immediato

**Limitazioni:** Non fa planning a lungo termine, può andare in loop, costoso

---

**B) PLAN-AND-EXECUTE AGENTS**

Funzionamento: Planning phase → Execution phase → Re-planning se serve

```
User: "Crea un report trimestrale performance Q4 2024 con grafici"

PLANNING PHASE:
1. Query DB per trades Q4
2. Calcola metriche (P&L, Sharpe, Drawdown)
3. Genera grafici Python (matplotlib)
4. Crea PDF report
5. Salva in outputs/

EXECUTION PHASE:
Step 1... ok
Step 2... ok
Step 3... ERRORE (missing library)

RE-PLANNING:
3bis. Installa matplotlib
3. Ri-genera grafici
4-5. Continua
```

**Quando usarlo:** Task complessi multi-step (>5 step), serve output strutturato, hai bisogno di "explainability"

**Limitazioni:** Lento (planning è pesante), rigido se il piano è sbagliato, overhead se task è semplice

---

**C) MULTI-AGENT SYSTEMS**

Funzionamento: N agents specializzati che collaborano

```
Sistema analisi trading:

DATA AGENT → Specialista queries DB
  ↓
ANALYSIS AGENT → Specialista calcoli finanziari
  ↓
REPORT AGENT → Specialista generazione testi

User: "Analizza Dicembre 2024"

DATA AGENT: "Ho estratto 47 trades, totale P&L +€12,450"
ANALYSIS AGENT: "Sharpe: 2.1, Max DD: 8%, Win Rate: 64%"
REPORT AGENT: "Dicembre è stato un mese eccellente..."
```

**Quando usarlo:** Problemi complessi che beneficiano di specializzazione, ogni sub-task ha expertise diversa, serve scalabilità

**Limitazioni:** Complessità architetturale enorme, coordinazione difficile, debug nightmare, molto costoso

---

## Tool Use / Function Calling

Questo è **IL** meccanismo che rende gli agents potenti. Senza tool use, un agent è solo un chatbot che parla.

### Cos'è il Function Calling

**Prima (pre-2023):** Dovevi spiegare nel prompt come chiamare funzioni, parsare l'output (sperando fosse valido JSON), eseguire la funzione, e re-inviare il risultato. Fragile e inaffidabile.

**Adesso (function calling nativo):** OpenAI, Anthropic, Google hanno aggiunto function calling nel protocollo API.

```python
# APPROCCIO MODERNO (robusto)
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather for a city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "City name, e.g. Milano"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"]
                    }
                },
                "required": ["city"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Che tempo fa a Milano?"}],
    tools=tools,
    tool_choice="auto"  # LLM decide se chiamare tool
)
```

**Vantaggi:**

- ✓ Formato garantito: JSON schema validation
- ✓ Affidabile: LLM è fine-tuned per chiamare funzioni
- ✓ Type-safe: parametri validati automaticamente
- ✓ Debuggable: structured tool call objects

### Come Funziona Tecnicamente

Il flow completo:

```
┌──────────────────────────────────────────────────────────┐
│ STEP 1: USER MESSAGE + TOOL DEFINITIONS                  │
└──────────────────────────────────────────────────────────┘
         │
         ↓
    ┌─────────┐
    │   LLM   │  ← "Vede" tools disponibili
    └─────────┘     Decide: chiamare tool O rispondere
         │
         ├─────→ CASO A: Risponde direttamente
         │       "Il tempo a Milano è..."
         │
         └─────→ CASO B: Chiama tool
                 tool_call = {
                   "name": "get_weather",
                   "args": {"city": "Milano"}
                 }
                      │
                      ↓
┌──────────────────────────────────────────────────────────┐
│ STEP 2: EXECUTOR ESEGUE LA FUNZIONE                      │
└──────────────────────────────────────────────────────────┘
                      │
         result = get_weather("Milano")
         # → "Soleggiato, 18°C"
                      │
                      ↓
┌──────────────────────────────────────────────────────────┐
│ STEP 3: RESULT TORNA ALL'LLM                             │
└──────────────────────────────────────────────────────────┘
         │
    ┌─────────┐
    │   LLM   │  ← Vede: "tool result = Soleggiato, 18°C"
    └─────────┘     Genera risposta finale
         │
         ↓
"A Milano oggi è soleggiato con 18°C"
```

**Nota critica:** L'LLM **non esegue** mai le funzioni. Solo **decide di chiamarle**. È il tuo codice (executor) che esegue la vera funzione Python e ritorna il risultato.

### Esempio Pratico: Tool per Database

```python
from typing import List, Dict
import mysql.connector

# STEP 1: Definisci la funzione Python
def get_trades_by_date_range(
    start_date: str,
    end_date: str,
    symbol: str = None
) -> List[Dict]:
    """
    Recupera trades da DB in un range di date.

    Args:
        start_date: Data inizio formato YYYY-MM-DD
        end_date: Data fine formato YYYY-MM-DD
        symbol: Opzionale - filtra per simbolo

    Returns:
        Lista di trade con campi: date, symbol, direction, pnl, points
    """
    conn = mysql.connector.connect(
        host="localhost",
        user="trader",
        password="***",
        database="trading_db"
    )

    query = """
        SELECT 
            trade_date,
            symbol,
            direction,
            entry_price,
            exit_price,
            pnl,
            points
        FROM trades
        WHERE trade_date BETWEEN %s AND %s
    """

    params = [start_date, end_date]

    if symbol:
        query += " AND symbol = %s"
        params.append(symbol)

    query += " ORDER BY trade_date DESC"

    cursor = conn.cursor()
    cursor.execute(query, params)

    trades = []
    for row in cursor.fetchall():
        trades.append({
            "date": row[0].isoformat(),
            "symbol": row[1],
            "direction": row[2],
            "entry": float(row[3]),
            "exit": float(row[4]),
            "pnl": float(row[5]),
            "points": float(row[6])
        })

    conn.close()
    return trades

# STEP 2: Definisci il tool per l'LLM (JSON schema)
trading_tool = {
    "type": "function",
    "function": {
        "name": "get_trades_by_date_range",
        "description": (
            "Query the trading database to retrieve trades "
            "within a specific date range. Useful for analyzing "
            "trading performance over time periods."
        ),
        "parameters": {
            "type": "object",
            "properties": {
                "start_date": {
                    "type": "string",
                    "description": "Start date in YYYY-MM-DD format",
                    "pattern": "^\\d{4}-\\d{2}-\\d{2}$"
                },
                "end_date": {
                    "type": "string",
                    "description": "End date in YYYY-MM-DD format",
                    "pattern": "^\\d{4}-\\d{2}-\\d{2}$"
                },
                "symbol": {
                    "type": "string",
                    "description": "Optional: Filter by symbol",
                    "enum": ["DAX", "FTSE", "CAC", "MIB"]
                }
            },
            "required": ["start_date", "end_date"]
        }
    }
}

# STEP 3: Executor agent loop
from openai import OpenAI
import json

client = OpenAI(api_key="...")

def run_agent(user_query: str):
    messages = [{"role": "user", "content": user_query}]

    # Loop fino a quando agent non finisce
    while True:
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=[trading_tool],
            tool_choice="auto"
        )

        message = response.choices[0].message

        # CASO 1: LLM vuole chiamare tool
        if message.tool_calls:
            messages.append(message)

            for tool_call in message.tool_calls:
                function_name = tool_call.function.name
                arguments = json.loads(tool_call.function.arguments)

                # Esegui la vera funzione Python
                if function_name == "get_trades_by_date_range":
                    result = get_trades_by_date_range(**arguments)

                # Ritorna il risultato all'LLM
                messages.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "name": function_name,
                    "content": json.dumps(result)
                })

            # Loop: ri-chiama LLM con i risultati
            continue

        # CASO 2: LLM ha finito e risponde
        else:
            return message.content

# USO:
response = run_agent(
    "Quanti trade ho fatto su DAX a dicembre 2024? "
    "Qual è stato il P&L totale?"
)

# Internamente agent fa:
# 1. LLM decide: "Devo chiamare get_trades_by_date_range"
# 2. Tool call: get_trades_by_date_range("2024-12-01", "2024-12-31", "DAX")
# 3. Result: [{"date": "2024-12-15", "pnl": 450, ...}, ...]
# 4. LLM vede result e genera: "Hai fatto 18 trade su DAX con P&L +€3,240"
```

### Tool Design Best Practices

**ERRORE COMUNE #1: Tool troppo granulari**

```python
# ❌ BAD: Un tool per ogni campo
def get_trade_symbol(trade_id): ...
def get_trade_pnl(trade_id): ...
def get_trade_date(trade_id): ...

# Agent deve fare 3 chiamate per un trade!
```

```python
# ✅ GOOD: Un tool che ritorna tutto
def get_trade_details(trade_id):
    return {
        "symbol": "DAX",
        "pnl": 450,
        "date": "2024-12-15",
        "direction": "LONG",
        # ... tutto insieme
    }
```

**ERRORE COMUNE #2: Tool troppo generici**

```python
# ❌ BAD: SQL generico (PERICOLOSO!)
def execute_sql(query: str):
    """Execute any SQL query"""
    return db.execute(query)

# Agent potrebbe fare DROP TABLE!
```

```python
# ✅ GOOD: Tool semantici specifici
def get_monthly_performance(year: int, month: int):
    """Get aggregated performance metrics for a specific month"""
    # Tu controlli la query SQL internamente
    # Agent può solo passare parametri sicuri
    ...
```

**ERRORE COMUNE #3: Descrizioni vaghe**

```python
# ❌ BAD
"description": "Gets data from database"
# Agent non sa QUANDO usarlo
```

```python
# ✅ GOOD
"description": (
    "Retrieves detailed information about a specific trade "
    "including entry/exit prices, P&L, timestamp, and strategy used. "
    "Use this when the user asks about a particular trade ID or "
    "wants to inspect trade execution details."
)
# Agent capisce: contesto + quando usare
```

---

## Pattern di Orchestrazione

### ReAct Pattern (Reason + Act)

È il pattern più comune e quello che usi al 90% dei casi.

**Come funziona:**

```
Loop infinito fino a "Final Answer":

1. THOUGHT: LLM ragiona su cosa fare
   ↓
2. ACTION: Decide quale tool chiamare
   ↓
3. OBSERVATION: Vede il risultato del tool
   ↓
4. THOUGHT: Ragiona sul risultato
   ↓
5. Ripete finché non ha abbastanza info
   ↓
6. FINAL ANSWER: Risponde all'utente
```

**Esempio trace reale:**

```
User: "Il mio ultimo trade è stato profittevole?"

--- ITERATION 1 ---
THOUGHT: Devo sapere qual è l'ultimo trade.
ACTION: get_trades_by_date_range(
    start_date="2024-01-01",
    end_date="2024-12-25",
    limit=1
)
OBSERVATION: [{"date": "2024-12-24", "pnl": 320, "symbol": "DAX"}]

--- ITERATION 2 ---
THOUGHT: L'ultimo trade è del 2024-12-24 con P&L di +320€.
         Questo è positivo, quindi sì, è stato profittevole.
ACTION: None (ho abbastanza info)

FINAL ANSWER: "Sì, il tuo ultimo trade (DAX del 24 dicembre) 
               è stato profittevole con +€320."
```

**Implementazione con LangChain:**

```python
from langchain.agents import create_react_agent, AgentExecutor
from langchain.tools import Tool
from langchain_openai import ChatOpenAI

# 1. Definisci tools
tools = [
    Tool(
        name="get_trades",
        func=get_trades_by_date_range,
        description="Get trades from database by date range"
    ),
    Tool(
        name="calculate_metrics",
        func=calculate_trading_metrics,
        description="Calculate Sharpe, Sortino, Max DD from trade list"
    )
]

# 2. Create agent
llm = ChatOpenAI(model="gpt-4o", temperature=0)
agent = create_react_agent(
    llm=llm,
    tools=tools,
    prompt=react_prompt_template
)

# 3. Create executor (gestisce il loop)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,  # Print reasoning steps
    max_iterations=5,  # Safety
    handle_parsing_errors=True
)

# 4. Run
result = agent_executor.invoke({
    "input": "Analizza performance dicembre 2024"
})
```

**Pro:** Semplice, trasparente, flessibile

**Contro:** Può essere verboso, costoso, può andare in loop

### Plan-and-Execute Pattern

Quando il task è complesso, ReAct può "vagare". Plan-and-Execute separa planning da execution.

**Come funziona:**

```
PHASE 1: PLANNING
User: "Crea report Q4 2024 con analisi risk-adjusted returns"

Planner Agent:
"Piano:
 1. Recupera tutti i trade Q4 2024
 2. Calcola metriche base (P&L, win rate, avg trade)
 3. Calcola Sharpe e Sortino Ratio
 4. Identifica periodi di high drawdown
 5. Genera grafici equity curve
 6. Compila report markdown
 7. Esporta PDF"

PHASE 2: EXECUTION
Executor:
  Step 1: ✅ Done → 87 trades recuperati
  Step 2: ✅ Done → P&L: +€18,450, WR: 67%
  Step 3: ✅ Done → Sharpe: 2.1, Sortino: 2.8
  Step 4: ✅ Done → Max DD: 12% (Nov 15-22)
  Step 5: ⚠️ Error → Missing matplotlib

PHASE 3: RE-PLANNING
Planner:
"Piano rivisto:
 5a. Installa matplotlib
 5b. Genera grafici
 6-7. Continua"
```

**Quando usarlo:** Task con >5 step sequenziali, serve trasparenza del piano, task che richiede "big picture thinking"

**Quando NON usarlo:** Query semplici, task dove il piano non è prevedibile, budget limitato

### Multi-Agent Pattern

Quando hai bisogno di specializzazione o parallelizzazione.

```python
# Agents specializzati
data_agent = create_react_agent(
    llm,
    tools=[db_tool],
    system_prompt="You are a data extraction specialist."
)

analyst_agent = create_react_agent(
    llm,
    tools=[calculator_tool, stats_tool],
    system_prompt="You are a quantitative analyst."
)

writer_agent = create_react_agent(
    llm,
    tools=[],
    system_prompt="You are a financial writer."
)

# Workflow
workflow = StateGraph(MessagesState)
workflow.add_node("data_agent", data_agent)
workflow.add_node("analyst_agent", analyst_agent)
workflow.add_node("writer_agent", writer_agent)

workflow.add_edge("data_agent", "analyst_agent")
workflow.add_edge("analyst_agent", "writer_agent")
```

**Pro:** Specializzazione, parallelizzazione possibile, isolation

**Contro:** Complessità alta, coordinazione difficile, debug complesso, molto costoso

---

## Quando Usare Agents vs Chains

Questa è la domanda da **€1000**. Ecco una decisione tree pratica.

### Decision Tree

```
START: Hai un task da automatizzare con LLM

│
├─→ Il flusso è COMPLETAMENTE prevedibile?
│   │
│   ├─→ SÌ → USA CHAIN
│   │         (es: "Traduci documento" → sempre load → translate → save)
│   │
│   └─→ NO → continua ↓
│
├─→ Serve prendere DECISIONI basate su dati runtime?
│   │
│   ├─→ SÌ → continua ↓
│   │
│   └─→ NO → USA CHAIN
│
├─→ Quante possibili azioni/tool?
│   │
│   ├─→ 1-2 tool → VALUTA Chain con conditional logic
│   ├─→ 3-10 tool → USA AGENT (ReAct)
│   └─→ >10 tool → USA AGENT (Plan-and-Execute o Multi-Agent)
│
├─→ Il task fallisce spesso e serve retry intelligente?
│   │
│   ├─→ SÌ → USA AGENT
│   └─→ NO → continua ↓
│
├─→ Budget / Latency è critico?
│   │
│   ├─→ SÌ → USA CHAIN (veloce/economico)
│   └─→ NO → USA AGENT (flessibile ma costoso)
│
└─→ Serve "reasoning" visibile?
    │
    ├─→ SÌ → USA AGENT
    └─→ NO → USA CHAIN
```

### Esempi Pratici

**CASO 1: RAG Q&A su documenti**

```
Task: "Rispondi a domande su bilanci aziendali"

Flusso:
1. Embed query
2. Similarity search
3. Rerank
4. Generate answer

Decisione: USA CHAIN ✅

Perché:
- Flusso sempre identico
- Nessuna decisione runtime
- Performance critica
- Costo contenuto
```

**CASO 2: Analisi multi-fonte**

```
Task: "Analizza se un trade è stato buono considerando condizioni mercato"

Flusso NON prevedibile:
- Recuperare trade (DB query)
- Verificare condizioni mercato (web search)
- Calcolare benchmark (web search)
- Confrontare performance
- Maybe: calcolare alternative

Decisione: USA AGENT ✅

Perché:
- Flusso DINAMICO
- Tool calling multiplo
- Decisioni runtime
- Reasoning trasparente utile
```

**CASO 3: Report generation**

```
Task: "Genera report PDF mensile performance"

Flusso:
1. Query DB
2. Calcola metriche (sempre le stesse)
3. Genera grafici
4. Compila template
5. Export PDF

Decisione: USA CHAIN ✅

Perché:
- Step SEMPRE gli stessi
- Nessuna decisione
- Deterministico
```

### Tabella Comparativa

| Criterio            | Chain              | Agent                |
| ------------------- | ------------------ | -------------------- |
| **Flusso**          | Fisso, prevedibile | Dinamico, adattivo   |
| **Tool calling**    | No o hardcoded     | Sì, dinamico         |
| **Decision making** | Zero (if/else)     | Sì (LLM decide)      |
| **Latency**         | Bassa (1 LLM call) | Alta (N LLM calls)   |
| **Costo**           | Basso              | Alto (N volte chain) |
| **Affidabilità**    | Alta               | Media                |
| **Debugging**       | Facile             | Difficile            |
| **Flessibilità**    | Bassa              | Alta                 |
| **Quando usare**    | Task ripetitivi    | Task complessi       |

---

## Limiti e Failure Modes

Gli agents **non sono magia**. Hanno limiti strutturali.

### Failure Mode #1: Infinite Loops

```python
# Agent bloccato in loop
User: "Calcola average P&L per simbolo"

Agent trace:
Thought: "Devo recuperare trades"
Action: get_all_trades()
Observation: [1000 trades...]

Thought: "Ok, ora devo calcolare media per simbolo. 
          Ma non ho ancora i simboli univoci..."
Action: get_all_trades()  # ← RICHIAMA STESSO TOOL!
Observation: [1000 trades...]

Thought: "Aspetta, forse devo prima..."
Action: get_all_trades()  # ← LOOP!
```

**Soluzione:**

```python
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    max_iterations=10,  # ← SEMPRE mettere limite!
    early_stopping_method="generate"
)
```

### Failure Mode #2: Tool Misuse

```python
# Agent usa tool sbagliato
User: "Quanti trade ho fatto oggi?"

Agent:
Action: web_search("what is today's date")  # ← ❌ Sbagliato!
# Doveva usare: datetime.now()

Observation: "Today is December 25, 2024..."
Action: get_trades_by_date_range("December 25 2024", ...)
# ← ❌ Formato data sbagliato! (serve YYYY-MM-DD)

ERROR: Invalid date format
```

**Soluzione:**

```python
# Migliora description
{
    "name": "get_trades_by_date_range",
    "description": (
        "Get trades between two dates. "
        "IMPORTANT: dates must be in YYYY-MM-DD format. "
        "To get today's date, use 'get_current_date' tool first."
    ),
    "parameters": {
        "start_date": {
            "type": "string",
            "pattern": "^\\d{4}-\\d{2}-\\d{2}$"  # ← Validazione
        }
    }
}

# Aggiungi tool helper
def get_current_date() -> str:
    """Returns today's date in YYYY-MM-DD format"""
    return datetime.now().strftime("%Y-%m-%d")
```

### Failure Mode #3: Hallucinated Tool Calls

```python
# Agent inventa tool che non esiste
User: "Plotta equity curve"

Agent:
Action: plot_equity_curve(data=[...])  # ← NON esiste!

ERROR: Tool 'plot_equity_curve' not found
```

**Soluzione:**

```python
system_prompt = """
You have access to these tools ONLY:
1. get_trades_by_date_range
2. calculate_metrics
3. save_to_file

You CANNOT plot charts (no plotting tools available).
If asked, explain the limitation politely.
"""
```

### Failure Mode #4: Context Length Exceeded

```python
# Agent accumula troppi tool results
User: "Analizza tutti i trade del 2024"

Agent:
Action: get_trades_by_date_range("2024-01-01", "2024-12-31")
Observation: [5000 trades, 500KB JSON...]

# Context pieno!
ERROR: Context length exceeded
```

**Soluzione:**

```python
# Tool che ritorna SUMMARY, non raw data
def get_trades_summary(start_date, end_date):
    """Returns AGGREGATED summary"""
    trades = db.query(...)
    return {
        "total_trades": len(trades),
        "total_pnl": sum(t.pnl for t in trades),
        "win_rate": calculate_win_rate(trades),
        "best_trade": max(trades, key=lambda t: t.pnl),
        # NO raw list di 5000 trade!
    }
```

### Failure Mode #5: Non-Deterministic Behavior

```python
# Stesso input, output diverso
query = "Qual è il mio miglior trade del 2024?"

# Run 1: calcola max(pnl) → "Trade del 15 marzo, +€2,340"
# Run 2: fa web search random → output diverso!
```

**Soluzione:**

```python
llm = ChatOpenAI(
    model="gpt-4o",
    temperature=0  # ← Deterministico
)
```

---

## Classificazione Intent per Routing

In un chatbot generico, non sai a priori se l'utente sta facendo una richiesta semplice (che puoi gestire con logica diretta) o complessa (che richiede un agent).

La soluzione: **classificare l'intent PRIMA di decidere il routing**.

### Il Problema

```python
# Senza classificazione
User: "Accendi la luce"
→ Lanci agent? (overkill, 2 secondi, $0.01)
→ Logica diretta? (come sai che è semplice?)

User: "Analizza performance e dimmi dove migliorare"
→ Logica diretta? (impossibile, troppo complesso)
→ Agent? (come sai che serve?)
```

Serve un **classifier** che decide il routing.

### Soluzione: Few-Shot LLM Classifier

**Perché questa soluzione:**

- ❌ Regex/Keywords → Troppo fragile
- ❌ Custom ML model → Serve training, maintenance
- ❌ BERT fine-tuned → Overkill, rigido
- ✅ LLM few-shot → Semantico, flessibile, zero training

**Numeri:**

- Costo: ~$0.0001 per classificazione (GPT-4o-mini)
- Latency: 100-300ms
- Accuracy: 90-95%

### Implementazione

```python
from openai import OpenAI
from pydantic import BaseModel
from typing import Literal

class IntentClassification(BaseModel):
    intent_type: Literal["simple_action", "complex_reasoning", "clarification_needed"]
    confidence: float
    reasoning: str
    suggested_route: Literal["direct_api", "chain", "agent"]

class IntentClassifier:
    def __init__(self, model: str = "gpt-4o-mini"):
        self.client = OpenAI()
        self.model = model

        # Few-shot examples
        self.examples = [
            {
                "input": "Accendi la luce del soggiorno",
                "classification": {
                    "intent_type": "simple_action",
                    "confidence": 0.95,
                    "reasoning": "Azione diretta, parametri espliciti",
                    "suggested_route": "direct_api"
                }
            },
            {
                "input": "Qual è il mio Sharpe Ratio di dicembre?",
                "classification": {
                    "intent_type": "simple_action",
                    "confidence": 0.9,
                    "reasoning": "Calcolo deterministico, parametri chiari",
                    "suggested_route": "chain"
                }
            },
            {
                "input": "Analizza se ho tradato bene considerando condizioni mercato",
                "classification": {
                    "intent_type": "complex_reasoning",
                    "confidence": 0.95,
                    "reasoning": "Richiede decisioni runtime, multi-step dinamico",
                    "suggested_route": "agent"
                }
            }
        ]

    def classify(self, user_input: str) -> IntentClassification:
        system_prompt = """Classifica il messaggio utente in:

1. simple_action: Azione diretta, parametri espliciti
2. complex_reasoning: Decisioni runtime, multi-step dinamico
3. clarification_needed: Troppo vago

Rispondi in JSON."""

        examples_text = "\n\n".join([
            f"Input: {ex['input']}\nOutput: {json.dumps(ex['classification'])}"
            for ex in self.examples
        ])

        response = self.client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": f"Esempi:\n{examples_text}\n\nInput: {user_input}\nOutput:"}
            ],
            temperature=0,
            response_format={"type": "json_object"}
        )

        result = json.loads(response.choices[0].message.content)
        return IntentClassification(**result)
```

### Integrazione nel Chatbot

```python
class SmartChatbot:
    def __init__(self):
        self.classifier = IntentClassifier()
        self.direct_handler = DirectAPIHandler()
        self.chain_handler = ChainHandler()
        self.agent_handler = AgentHandler()

    def handle_message(self, user_input: str) -> str:
        # STEP 1: Classifica intent
        classification = self.classifier.classify(user_input)

        # STEP 2: Route
        if classification.suggested_route == "direct_api":
            return self.direct_handler.execute(user_input)

        elif classification.suggested_route == "chain":
            return self.chain_handler.execute(user_input)

        elif classification.suggested_route == "agent":
            if classification.confidence < 0.75:
                return "Non sono sicuro di aver capito. Puoi riformulare?"
            return self.agent_handler.execute(user_input)
```

**Risultato:** Il 90% delle richieste usa il fast path (direct/chain), solo il 10% usa l'agent costoso.

---

## Function Calling: Come Funziona Sotto al Cofano

La domanda che molti non si fanno: **quando l'agent fa function calling, cosa pago esattamente?**

Risposta: **paghi TUTTO il contesto ogni volta**, non solo il risultato del tool.

### Flow Completo con Token Count

Esempio concreto: *"Qual è stato il mio miglior trade di dicembre 2024?"*

**ITERATION 1: Prima chiamata LLM**

```
Request → OpenAI:
{
  "messages": [
    {role: "system", content: "You are a trading assistant..."},
    {role: "user", content: "Miglior trade dicembre 2024?"}
  ],
  "tools": [/* definizione tool get_trades */]
}

Token count:
- System prompt: 15 tokens
- User message: 12 tokens
- Tool definitions: 120 tokens
─────────────────────────
TOTAL INPUT: 147 tokens

Response ← OpenAI:
{
  "tool_calls": [{
    "name": "get_trades_by_date_range",
    "arguments": '{"start_date": "2024-12-01", "end_date": "2024-12-31"}'
  }]
}

Token count:
- Tool call: 28 tokens
─────────────────────────
COST ITERATION 1: 147 input + 28 output = 175 tokens (~$0.00026)
```

**TUO CODICE esegue il tool (costo zero, è Python locale):**

```python
result = get_trades_by_date_range("2024-12-01", "2024-12-31")
# Returns: 18 trades (850 tokens when serialized to JSON)
```

**ITERATION 2: Seconda chiamata con risultato tool**

```
Request → OpenAI:
{
  "messages": [
    {role: "system", content: "..."},           // ← 15 tokens
    {role: "user", content: "Miglior trade?"}, // ← 12 tokens
    {role: "assistant", tool_calls: [...]},     // ← 28 tokens (RIMANDATO!)
    {role: "tool", content: "[18 trades...]"}   // ← 850 tokens (NUOVO!)
  ],
  "tools": [...]                                 // ← 120 tokens (RIMANDATO!)
}

Token count:
- System: 15
- User: 12
- Tool definitions: 120
- Assistant call: 28
- Tool result: 850
─────────────────────────
TOTAL INPUT: 1,025 tokens

Response ← OpenAI:
{
  "content": "Il miglior trade è stato DAX del 20 dic, +€2,100"
}

Token count:
- Final answer: 35 tokens
─────────────────────────
COST ITERATION 2: 1,025 input + 35 output = 1,060 tokens (~$0.00291)
```

**TOTAL COST per questa query:**

```
Iteration 1: 175 tokens   → $0.00026
Iteration 2: 1,060 tokens → $0.00291
──────────────────────────────────────
TOTAL: 1,235 tokens → $0.00317

Breakdown:
- User query: 12 tokens (1%)
- Tool definitions: 240 tokens (19% - ridondante!)
- Tool result: 850 tokens (69% - IL KILLER!)
- LLM reasoning: 63 tokens (5%)
```

### Il Problema: Context Explosion

```
Iteration N input = 
    system_prompt +
    user_query +
    tool_definitions +
    SUM(all previous assistant messages) +
    SUM(all previous tool results)
```

**Se agent fa 5 tool calls, paghi il contesto iniziale 5 volte!**

Esempio multi-iteration:

```
Query: "Analizza trade dicembre considerando mercato"

ITERATION 1: get_trades() → 200 tokens input, 30 output
ITERATION 2: web_search() → 1,080 tokens input (ACCUMULO!)
ITERATION 3: calculate() → 1,535 tokens input (ACCUMULO!)
ITERATION 4: final answer → 1,743 tokens input (ACCUMULO!)

TOTAL COST: $0.01307 (~$13 per 1,000 query!)
```

### Schema Visivo

```
USER QUERY
   ↓
┌─────────────────────────────────────────────────────────────┐
│ ITERATION 1                                                 │
│  INPUT: system + user + tools = 147 tokens                  │
│  OUTPUT: tool_call = 28 tokens                              │
│  💰 COST: $0.00026                                          │
└─────────────────────────────────────────────────────────────┘
   ↓
[TUO CODICE esegue tool → result = 850 tokens]
   ↓
┌─────────────────────────────────────────────────────────────┐
│ ITERATION 2                                                 │
│  INPUT: system + user + tools + prev_call + result          │
│         = 15 + 12 + 120 + 28 + 850 = 1,025 tokens ⚠️         │
│  OUTPUT: final_answer = 35 tokens                           │
│  💰 COST: $0.00291                                          │
└─────────────────────────────────────────────────────────────┘
```

### Ottimizzazioni

**1. Tool che ritorna SUMMARY**

```python
# ❌ BAD: Ritorna 500 trade (45,000 tokens)
def get_all_trades():
    return db.query("SELECT * FROM trades")

# ✅ GOOD: Ritorna aggregato (80 tokens)
def get_trades_summary():
    trades = db.query(...)
    return {
        "total": len(trades),
        "total_pnl": sum(t.pnl for t in trades),
        "win_rate": calculate_win_rate(trades),
        "best_trade": max(trades, key=lambda t: t.pnl)
        # NO lista completa!
    }

# Saving: 99.8%
```

**2. Prompt Caching (Anthropic/Gemini)**

```python
# Claude caching
response = anthropic.messages.create(
    system=[{
        "text": "System prompt...",
        "cache_control": {"type": "ephemeral"}  # ← Cache!
    }],
    tools=[...],  # Anche tools cachati
    messages=[...]
)

# Prima call: full price
# Successive (entro 5 min): 90% sconto su parte cachata
```

**3. Limiti Stretti**

```python
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    max_iterations=5,      # Max 5 tool calls
    max_cost_cents=10,     # Max $0.10
    timeout_seconds=30     # Max 30 secondi
)
```

### Perché l'API è Stateless?

Questa è una **scelta implementativa**, non una limitazione tecnica.

**Come funziona oggi (stateless):**

```
Client → Server: [tutto il context]
Server: processa, risponde, DISCARD tutto
Client → Server: [tutto il context + nuovo msg]
Server: processa TUTTO da zero, risponde, DISCARD
```

**Come POTREBBE funzionare (stateful):**

```
Client → Server: crea session
Server: alloca memoria, salva KV cache
Client → Server: nuovo msg (session_id)
Server: riusa KV cache, processa SOLO nuovo msg
```

**Perché non lo fanno?**

1. **Scalabilità:** Server stateless può load-balance facilmente
2. **Memoria:** KV cache = 5.9MB per token × 10K tokens = 59GB per session!
3. **Fault tolerance:** Server crash = perdi tutte le sessioni
4. **Business:** Token ridondanti = revenue

**Alternative esistenti:**

- Prompt caching (Anthropic/Gemini)
- ChatGPT UI è stateful, ma LLM sotto è stateless
- Gemini Extended Context

**La verità:** Non è limitazione tecnica, è scelta architetturale/business.

---

## Lessons Learned

### 1. Agents Sono Oversold

**Reality check:** 80% dei "use case agents" nel marketing AI sono bullshit. La maggior parte dei problemi si risolve meglio con:

- RAG semplice
- Chain deterministiche
- If/else intelligenti
- Classifier + routing

**Agents hanno senso SOLO quando:**

- ✅ Task è genuinamente imprevedibile
- ✅ Serve reasoning complesso
- ✅ Hai budget per costo/latency
- ✅ Puoi tollerare fallimenti

### 2. Tool Design È Critico

Il 70% dei problemi degli agents deriva da tool mal progettati:

```python
# ❌ Tool troppo granulari → troppe chiamate
# ❌ Tool troppo generici → agent confuso
# ❌ Tool che ritorna troppi dati → context explosion
# ❌ Descrizioni vaghe → agent usa tool sbagliato

# ✅ Tool semantici specifici
# ✅ Ritorna summary, non raw data
# ✅ Descrizioni MOLTO chiare
# ✅ Validazione parametri stretta
```

### 3. Context Management = Cost Management

La formula del costo agent:

```
cost = (system + user + tools) × iterations +
       sum(all_tool_results) +
       sum(all_outputs)
```

**Se tool ritorna 50K tokens e agent fa 5 iterations = 250K tokens solo per tool results!**

Ottimizzazioni critiche:

- Summary invece di raw data
- max_iterations limitato
- Prompt caching dove possibile
- LLM economici (GPT-4o-mini) dove appropriato

### 4. La Decisione Chain vs Agent È Economica

```
Chain:
- 1 LLM call
- ~500 tokens
- $0.001
- 500ms

Agent (3 iterations):
- 3 LLM calls
- ~3,000 tokens
- $0.008
- 3 secondi

Differenza: 8x costo, 6x latency
```

**Regola:** Usa agent SOLO quando chain proprio non basta.

### 5. Classificazione Intent È La Killer Feature

Pattern vincente per chatbot:

```python
# 90% richieste → fast path (direct/chain)
# 10% richieste → agent path

# Con few-shot classifier:
# - Cost: $0.0001 per classificazione
# - Latency: 100ms
# - Accuracy: 90%+

# Saving vs "agent sempre":
# - 80% reduction in cost
# - 70% reduction in latency
```

### 6. Guardrail Non Sono Opzionali

Agent senza limiti è una bomba:

```python
# SEMPRE implementare:
- max_iterations (evita loop infiniti)
- timeout (evita agent bloccati)
- max_cost (evita bill shock)
- allowed_tools whitelist (evita azioni pericolose)
- validation pre/post tool (evita dati corrotti)
```

### 7. Production vs Demo È Altro Mondo

Demo agent:

- Funziona 80% delle volte
- Failure → "riprova"
- Costo non importa

Production agent:

- Deve funzionare 99%+ delle volte
- Failure → logging, alerting, graceful degradation
- Costo × 1M users = importante
- Monitoring, metrics, A/B testing
- Human-in-the-loop per azioni critiche

### 8. Stateless API = Feature, Non Bug

Capire che l'API stateless è una scelta (non limitazione) ti aiuta a:

- Progettare meglio i tuoi agent
- Ottimizzare context management
- Sfruttare caching dove disponibile
- Non aspettarti magie che non arriveranno

### 9. Pattern di Orchestrazione Hanno Trade-off

```
ReAct:
+ Semplice, trasparente
- Può andare in loop, verboso

Plan-and-Execute:
+ Strutturato, explainable
- Lento, rigido se piano sbaglia

Multi-Agent:
+ Specializzazione, parallelizzazione
- Complessità, coordinazione difficile
```

Non c'è "il pattern migliore" – dipende dal task.

### 10. Use Case Reali Sono Pochi

Dove agents hanno DAVVERO senso in produzione:

- ✅ Customer support (KB grande, routing complesso)
- ✅ Code review (reasoning su codebase)
- ✅ Data analysis assistito (SQL + calcoli + visualizzazione)
- ✅ Research automation (multi-source, sintesi)

Dove sono OVERKILL:

- ❌ Comandi smart home semplici
- ❌ FAQ chatbot (RAG basta)
- ❌ Classificazione (batch processing più efficiente)
- ❌ Report generation deterministici

### 11. Tool Misuse È Il Failure Mode Più Comune

Agent confuso chiama tool sbagliato:

- Web search invece di datetime.now()
- SQL query con formato date sbagliato
- Tool che non esiste (hallucination)

**Soluzione:** Description tool MOLTO dettagliate + helper tools.

### 12. Non-Determinismo Va Gestito

Stesso input → output diverso è normale per agents.

Se serve determinismo:

- temperature=0
- Usa chain invece di agent
- Oppure accetta il non-determinismo e progetta di conseguenza

### 13. Debugging Agent È 10x Più Difficile

```python
# Chain: fallisce? Guardi il log, vedi esattamente dov'è l'errore

# Agent: fallisce?
# - Quale iteration?
# - Perché ha chiamato quel tool?
# - Il tool ha ritornato dati strani?
# - L'LLM ha interpretato male il risultato?
# - È andato in loop?
```

**Soluzione:** Logging verboso, tracing, tool per visualizzare reasoning.

### 14. Costo Hidden: Development Time

```
Chain semplice:
- 2 ore development
- 30 min debugging
- Deploy e funziona

Agent complesso:
- 8 ore development
- 4 ore debugging
- 2 ore tuning prompt
- 2 ore gestione edge cases
- Deploy e scopri nuovi problemi in production
```

**ROI non è sempre positivo!**

### 15. Il Futuro (Probabilmente)

Prossimi 2-3 anni:

- Caching più aggressivo (già happening)
- API ibride (stateless default, stateful opt-in)
- Context compression nativo
- Agent frameworks più maturi
- Costi in discesa (ma sempre > chains)

**Ma:** Pattern fondamentali non cambieranno. Chain vs Agent sarà sempre trade-off cost/flexibility.

---

## Conclusione

Gli AI Agents sono uno strumento potente **quando usati correttamente**. Il problema è che il 90% delle volte vengono usati scorrettamente.

**La regola d'oro:**

> **Default: NON usare agent. Usa agent solo quando logica semplice PROPRIO non basta.**

Se ti ritrovi a pensare "forse serve un agent", chiediti:

1. Posso fare lo stesso con una chain?
2. Posso fare lo stesso con if/else + LLM singolo?
3. Il costo/latency extra vale la flessibilità?

Se la risposta è "sì, sì, no" → non ti serve un agent.

**Happy coding, e ricorda:** essere scettico sugli agents ti rende un engineer migliore, non peggiore.

---

*Fine del documento. Buon lavoro!*
