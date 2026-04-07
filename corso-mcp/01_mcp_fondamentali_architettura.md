# Corso MCP — Model Context Protocol

## Modulo 1: Fondamentali e Architettura

### *"Perché esiste MCP e come è fatto"*

---

> **Prerequisiti consigliati**
> Conoscenza base di come funzionano le API degli LLM (Anthropic / OpenAI),
> familiarità con JSON, nozioni di architettura client/server.

---

## Indice

1. [Il problema che MCP risolve](#1-il-problema-che-mcp-risolve)
2. [Architettura Host / Client / Server](#2-architettura-host--client--server)
3. [Il flusso dinamico: come l'LLM usa i tool MCP](#3-il-flusso-dinamico-come-lllm-usa-i-tool-mcp)
4. [JSON-RPC 2.0 come protocollo base](#4-json-rpc-20-come-protocollo-base)
5. [Transport layer: stdio, SSE, HTTP](#5-transport-layer-stdio-sse-http)
6. [I tre primitivi: Tools, Resources, Prompts](#6-i-tre-primitivi-tools-resources-prompts)
7. [Confronto con le alternative](#7-confronto-con-le-alternative)
8. [Riepilogo](#8-riepilogo)

---

## 1. Il problema che MCP risolve

### 1.1 — Il caos delle integrazioni

Prima di MCP, integrare un LLM con strumenti esterni era un problema risolto in modo diverso
da ogni provider, ogni framework, ogni team. Il risultato era questo:

```python
    # Il mondo prima di MCP: stesso tool, quattro formati diversi

    # OpenAI function calling
    tools = [{
        "type": "function",
        "function": {
            "name": "get_stock_price",
            "parameters": { "ticker": "string" }
        }
    }]

    # Anthropic tool use (pre-MCP)
    tools = [{
        "type": "tool_use",
        "name": "get_stock_price",
        "input": { "ticker": "AAPL" }
    }]

    # LangChain tools
    class StockTool(BaseTool):
        name = "get_stock_price"
        def _run(self, ticker: str): ...

    # AutoGPT plugins
    # (formato proprietario completamente diverso)
```

Se costruisci un tool — per esempio un accesso al database di MAOTrade — devi reimplementarlo
quattro volte per quattro sistemi diversi. E quando un provider cambia le API, riscrivi tutto.

Questo è esattamente il problema che MCP vuole risolvere: **un protocollo standard,
indipendente dal modello, per connettere AI a strumenti e dati.**

---

### 1.2 — L'analogia con LSP

L'analogia che Anthropic usa è quella con il **Language Server Protocol (LSP)**, il protocollo
che ha standardizzato il supporto ai linguaggi negli editor (VS Code, IntelliJ, Neovim):

```
Prima di LSP (pre-2016):
    IntelliJ → aveva il suo sistema per Java
    Eclipse  → aveva il suo sistema per Java
    VS Code  → aveva il suo sistema per JavaScript

    Ogni IDE reimplementava autocompletamento, go-to-definition, refactoring
    per ogni linguaggio separatamente.
    Risultato: N IDE × M linguaggi = N×M integrazioni da mantenere.

Dopo LSP:
    Java Language Server   → parla LSP
    Python Language Server → parla LSP
    Rust Language Server   → parla LSP

    Ogni IDE implementa LSP una sola volta.
    Risultato: N IDE + M linguaggi = N+M integrazioni.
```

MCP fa la stessa cosa per AI e strumenti:

```
Senza MCP:  N modelli × M strumenti  =  N×M integrazioni
Con MCP:    N modelli + M server MCP =  N+M integrazioni
```

Ogni tool viene scritto **una volta**, come MCP server, e funziona con qualsiasi host
che supporta MCP: Claude Desktop, Cursor, la tua applicazione.

---

## 2. Architettura Host / Client / Server

MCP divide il mondo in tre ruoli distinti. Capirli bene è fondamentale perché la confusione
su questi termini è la fonte numero uno di incomprensioni nel protocollo.

```
┌─────────────────────────────────────────────────────────────────┐
│                             HOST                                │
│      (Claude Desktop, Cursor, la tua applicazione ARIA)         │
│                                                                 │
│   ┌────────────────────┐      ┌────────────────────┐            │
│   │    MCP CLIENT A    │      │    MCP CLIENT B    │            │
│   │   (1 per server)   │      │   (1 per server)   │            │
│   └─────────┬──────────┘      └─────────┬──────────┘            │
└─────────────┼───────────────────────────┼───────────────────────┘
              │ MCP Protocol              │ MCP Protocol
              │ (JSON-RPC 2.0)            │ (JSON-RPC 2.0)
              ▼                           ▼
  ┌───────────────────────┐    ┌───────────────────────┐
  │     MCP SERVER A      │    │     MCP SERVER B      │
  │  (accesso database)   │    │  (filesystem)         │
  │                       │    │                       │
  │  Tools, Resources,    │    │  Tools, Resources,    │
  │  Prompts              │    │  Prompts              │
  └───────────────────────┘    └───────────────────────┘
```

---

### HOST

È l'applicazione che l'utente usa direttamente. Contiene il modello LLM o si connette
ad esso via API. È responsabile di:

- gestire il ciclo di vita dei MCP client
- decidere quali server connettere
- coordinare le richieste tra LLM e server MCP
- gestire sicurezza e permessi

Claude Desktop è un host. Se costruisci un'applicazione con ARIA che usa MCP,
quella applicazione è l'host.

---

### CLIENT

Vive *dentro* l'host. Ogni client mantiene una connessione **1:1** con un singolo server MCP.
Non è un'applicazione separata: è un componente dell'host. L'host istanzia un client
per ogni server MCP che vuole usare.

---

### SERVER

È il processo esterno che espone le capacità (tools, resources, prompts). Può essere:

- un processo locale sulla stessa macchina (comunicazione via stdio)
- un servizio remoto (comunicazione via HTTP/SSE)
- un processo in un container Docker

Un server MCP è tipicamente piccolo e focalizzato: un server per il database,
uno per i file, uno per un'API esterna. Non è un monolite.

---

### Il punto critico: chi controlla cosa

```
LLM decide: "voglio chiamare questo tool"
    ↓
HOST riceve la richiesta, valuta se è permessa
    ↓
CLIENT traduce in JSON-RPC e manda al server
    ↓
SERVER esegue e risponde
    ↓
CLIENT riceve la risposta
    ↓
HOST la passa al LLM
```

**Il modello non parla mai direttamente con il server MCP.** L'host fa da mediatore
e può bloccare, loggare, o modificare qualsiasi chiamata. Questo è il design
di sicurezza intenzionale di MCP.

---

## 3. Il flusso dinamico: come l'LLM usa i tool MCP

La struttura statica — host istanzia client, client parla con server — è abbastanza intuitiva.
Quello che non è immediato è capire *quando e come* l'LLM entra in gioco. La distinzione
tra host e client sembra chiara finché non si prova a spiegarla: l'host è l'applicazione
che ha bisogno dei dati del server e che è responsabile di istanziare il client. Fin qui
tutto regge. Il punto che sfugge è il collegamento con il modello: deve essere l'host
a configurare le chiamate all'LLM per informarlo che esistono dei server MCP connessi,
e a quel punto il modello, se lo ritiene utile, li usa?

Sì, è esattamente così. E capire questo meccanismo è la chiave per capire MCP davvero,
non solo sulla carta.

---

### Fase 1 — Startup: l'host scopre cosa offrono i server

Quando l'host si avvia, per ogni server MCP configurato esegue un handshake:

```
Host
  │
  ├── avvia Server A (es. stdio: lancia il processo Python)
  │     └── Client A si connette
  │           └── chiama: initialize          → handshake
  │           └── chiama: tools/list          → "ho: [get_candles, get_indicators]"
  │           └── chiama: resources/list      → "ho: [config, last_analysis]"
  │
  ├── avvia Server B (es. filesystem)
  │     └── Client B si connette
  │           └── tools/list                  → "ho: [read_file, write_file, list_dir]"
  │
  └── HOST ora sa tutto quello che è disponibile
        → Tool pool:     get_candles, get_indicators, read_file, write_file, list_dir
        → Resource pool: config, last_analysis
```

A questo punto i server stanno girando e i client sono connessi, ma l'LLM non è ancora
stato coinvolto. L'host ha solo fatto l'inventario.

---

### Fase 2 — L'utente fa una richiesta

L'utente scrive: *"Analizza FTSEMIB su timeframe settimanale"*

L'host costruisce la chiamata all'LLM **iniettando i tool MCP** nel formato
che il modello capisce:

```python
    # L'host prende i tool dal pool MCP e li inserisce nella chiamata API

    available_tools = [
        {
            "name": "get_candles",
            "description": "Recupera dati OHLCV per un ticker nel periodo specificato",
            "input_schema": {
                "type": "object",
                "properties": {
                    "ticker":    { "type": "string" },
                    "timeframe": { "type": "string", "enum": ["1d", "1w", "1m"] },
                    "limit":     { "type": "integer", "default": 100 }
                },
                "required": ["ticker", "timeframe"]
            }
        },
        # ... tutti gli altri tool dai server connessi
    ]

    response = anthropic.messages.create(
        model="claude-opus-4-5",
        system="Sei ARIA, assistente per analisi finanziaria...",
        messages=[
            {"role": "user", "content": "Analizza FTSEMIB su timeframe settimanale"}
        ],
        tools=available_tools    # ← qui l'LLM "vede" i tool MCP
    )
```

**L'LLM non sa nulla di MCP.** Dal suo punto di vista sta vedendo una lista di tool
nel formato standard che già conosce. Il fatto che quei tool vengano da un server MCP
è completamente trasparente al modello.

---

### Fase 3 — L'LLM risponde con una tool call

Il modello analizza la richiesta, decide che ha bisogno di dati, e risponde:

```json
    {
        "role": "assistant",
        "content": [
            {
                "type": "text",
                "text": "Per analizzare FTSEMIB ho bisogno dei dati delle candele recenti."
            },
            {
                "type": "tool_use",
                "id": "toolu_01XY",
                "name": "get_candles",
                "input": {
                    "ticker": "FTSEMIB",
                    "timeframe": "1w",
                    "limit": 52
                }
            }
        ]
    }
```

A questo punto **l'LLM si ferma**. Non ha eseguito niente. Ha solo detto
"vorrei chiamare questo tool con questi argomenti". La palla torna all'host.

---

### Fase 4 — L'host esegue la tool call via MCP

```
Host riceve la tool call dall'LLM
  │
  ├── Identifica quale client gestisce "get_candles"
  │     → è il Client A (connesso al server MAOTrade)
  │
  ├── Client A invia al Server A (via JSON-RPC):
  │     {
  │       "jsonrpc": "2.0",
  │       "id": 1,
  │       "method": "tools/call",
  │       "params": {
  │         "name": "get_candles",
  │         "arguments": { "ticker": "FTSEMIB", "timeframe": "1w", "limit": 52 }
  │       }
  │     }
  │
  ├── Server A esegue la query sul database MySQL di MAOTrade
  │
  └── Server A risponde con i dati delle candele
```

---

### Fase 5 — L'host riprende la conversazione con l'LLM

```python
    # L'host aggiunge il risultato alla conversazione e richiama l'LLM

    messages.append({
        "role": "assistant",
        "content": [tool_use_block]         # la richiesta del modello (fase 3)
    })
    messages.append({
        "role": "user",
        "content": [{
            "type": "tool_result",
            "tool_use_id": "toolu_01XY",
            "content": "[{open: 28450, high: 28920, low: 28100, close: 28680}, ...]"
        }]
    })

    # Seconda chiamata: adesso il modello ha i dati e può fare l'analisi
    response = anthropic.messages.create(
        model="claude-opus-4-5",
        messages=messages,
        tools=available_tools
    )
```

---

### Il ciclo completo

```
UTENTE       HOST/ARIA         LLM              CLIENT A      SERVER A
  │               │               │                 │               │
  ├─ "Analizza ──►│               │                 │               │
  │   FTSEMIB"    │               │                 │               │
  │               ├─ tools ──────►│                 │               │
  │               │   nel contesto│                 │               │
  │               │               ├─ "chiama ───────│               │
  │               │               │  get_candles"   │               │
  │               │◄── tool_call ─┤                 │               │
  │               │               │                 │               │
  │               ├─ tools/call ───────────────────►│               │ 
  │               │               │                 ├── JSON-RPC ──►│
  │               │               │                 │◄── result ────┤
  │               │◄── result ──────────────────────│               │
  │               │               │                                 │
  │               ├─ tool_result ►│                                 │
  │               │               │                                 │
  │               │◄── analisi ───┤                                 │
  │               │    completa                                     │
  │◄── risposta ──┤                                                 │
```

---

## 4. JSON-RPC 2.0 come protocollo base

MCP usa JSON-RPC 2.0 come formato dei messaggi. È un protocollo per fare chiamate
a procedure remote usando JSON: semplice, bidirezionale, ben specificato.

### 4.1 — Struttura base

```json
    // REQUEST (client → server)
    {
        "jsonrpc": "2.0",
        "id": 1,
        "method": "tools/call",
        "params": {
            "name": "get_candles",
            "arguments": {
                "ticker": "FTSEMIB",
                "timeframe": "1w"
            }
        }
    }

    // RESPONSE — caso successo (server → client)
    {
        "jsonrpc": "2.0",
        "id": 1,
        "result": {
            "content": [
                { "type": "text", "text": "[{open: 28450, ...}]" }
            ]
        }
    }

    // RESPONSE — caso errore
    {
        "jsonrpc": "2.0",
        "id": 1,
        "error": {
            "code": -32603,
            "message": "Ticker not found"
        }
    }

    // NOTIFICATION (nessuna response attesa, non ha "id")
    {
        "jsonrpc": "2.0",
        "method": "notifications/progress",
        "params": {
            "progressToken": "abc123",
            "progress": 50,
            "total": 100
        }
    }
```

---

### 4.2 — Perché JSON-RPC e non REST?

REST sarebbe stata la scelta ovvia per chiunque arrivi dal web. Ma ci sono ragioni concrete
per cui JSON-RPC è più adatto qui:

**Il paradigma è diverso.** REST è pensato per risorse (`GET /candles/FTSEMIB`).
JSON-RPC è pensato per azioni (`call get_candles con ticker=FTSEMIB`). Un LLM che usa
tool sta eseguendo azioni, non navigando risorse.

**Bidirezionalità.** JSON-RPC funziona in entrambe le direzioni sullo stesso canale.
Il server può mandare notifiche al client senza che il client le abbia richieste.
Con REST richiederebbe polling o una connessione separata.

**Formato unico.** Un singolo schema per request, response ed errori.
Nessun verbo HTTP, status code, o header da gestire.

---

### 4.3 — I metodi MCP principali

```
Inizializzazione:
    initialize              → handshake, negozia capabilities
    initialized             → conferma (notification)

Tools:
    tools/list              → elenca i tool disponibili
    tools/call              → chiama un tool

Resources:
    resources/list          → elenca le risorse disponibili
    resources/read          → leggi una risorsa
    resources/subscribe     → iscriviti a cambiamenti (se supportato)

Prompts:
    prompts/list            → elenca i template disponibili
    prompts/get             → recupera un template

Utility:
    ping                    → keepalive
    logging/setLevel        → imposta livello log
    notifications/...       → varie notifiche async
```

---

## 5. Transport layer: stdio, SSE, HTTP

Il transport layer è il mezzo fisico attraverso cui viaggiano i messaggi JSON-RPC.
MCP ne supporta tre, ciascuno adatto a scenari diversi.

---

### 5.1 — stdio (Standard Input/Output)

Il client lancia il server come **processo figlio** e comunicano attraverso stdin/stdout.

```
Host Process
  │
  ├── fork/spawn ──► Server Process
  │                       │
  │   client writes       │
  │   ──────────────────► stdin del server
  │                       │
  │   server writes       │
  │   ◄────────────────── stdout del server
  │
  │   stderr del server ──► log separati (non fa parte del protocollo MCP)
```

**Quando si usa:** tool locali — file system, database locale, script Python, processi
sulla stessa macchina. È il transport più comune per i server MCP in locale.

**Pro:** semplice, nessuna porta di rete aperta, sicuro per default, latenza minima.

**Contro:** solo locale, il server muore quando muore il client, non scalabile su
più macchine.

```python
    # Avviare un server MCP via stdio (lato host/client)
    import asyncio
    from mcp import ClientSession, StdioServerParameters
    from mcp.client.stdio import stdio_client

    server_params = StdioServerParameters(
        command="python",
        args=["maotrade_mcp_server.py"],
        env={"DB_HOST": "localhost", "DB_NAME": "maotrade"}
    )

    async def main():
        async with stdio_client(server_params) as (read, write):
            async with ClientSession(read, write) as session:
                await session.initialize()
                # da qui puoi chiamare tools, leggere resources, ecc.
                tools = await session.list_tools()
```

---

### 5.2 — SSE (Server-Sent Events)

Il client si connette a un endpoint HTTP del server. Il server tiene aperta la connessione
e manda eventi via SSE. Per i messaggi dal client al server si usa un endpoint POST separato.

```
CLIENT                              SERVER (HTTP)
  │                                       │
  ├── GET /sse ──────────────────────────► (connessione tenuta aperta)
  │   ◄────────────────── events ─────────┤
  │                                       │
  ├── POST /message ─────────────────────► (requests del client)
  │   ◄────────────────── 202 Accepted ───┤
  │                                       │
  │   ◄────────────────── response SSE ───┤ (risposta arriva sullo stream)
```

**Quando si usa:** server remoti accessibili via HTTP, ambienti cloud, quando più host
devono connettersi allo stesso server.

**Pro:** HTTP standard (firewall-friendly), server su macchine diverse, più client
contemporaneamente.

**Contro:** più complesso di stdio, la connessione SSE può avere problemi con certi proxy.

---

### 5.3 — Streamable HTTP

Il transport più recente, pensato per rimpiazzare SSE nei deployment cloud. Usa HTTP puro
con streaming, più compatibile con infrastrutture serverless e load balancer.

```
CLIENT                              SERVER
  │                                      │
  ├── POST /mcp ─────────────────────────► (ogni request)
  │   ◄───────────── streaming response──┤ (risposta streamata via HTTP)
```

**Quando si usa:** deployment cloud, serverless, situazioni dove SSE ha problemi
di compatibilità con proxy o load balancer.

---

### 5.4 — Quale scegliere

```
┌──────────────────────────────────────────────────────────────────┐
│  Scenario                              Transport consigliato     │
├──────────────────────────────────────────────────────────────────┤
│  Tool locale (script, DB locale)       stdio                     │
│  Servizio sul tuo server OVH           stdio o SSE               │
│  Servizio cloud / SaaS                 Streamable HTTP           │
│  Development / testing                 stdio (più semplice)      │
└──────────────────────────────────────────────────────────────────┘
```

Per ARIA: se esponi le capacità di MAOTrade come MCP server sullo stesso server OVH
dove gira tutto il resto, **stdio** è la scelta più semplice e sicura. SSE o HTTP
solo se hai bisogno di accedervi da macchine diverse.

---

## 6. I tre primitivi: Tools, Resources, Prompts

Sono le tre astrazioni attraverso cui un server MCP espone le sue capacità.
Ogni primitivo ha un ruolo specifico e non sono intercambiabili.

---

### 6.1 — Tools (Azioni)

Un tool è una funzione che il modello può **chiamare per fare qualcosa**. Ha effetti:
legge dati, scrive dati, chiama API, esegue calcoli.

**Caratteristica chiave:** il modello decide quando chiamarlo e con quali argomenti.
L'esecuzione avviene nel server.

```python
    from mcp.server import Server
    from mcp.types import Tool, TextContent
    import mcp.types as types

    app = Server("maotrade-server")

    @app.list_tools()
    async def list_tools() -> list[Tool]:
        return [
            Tool(
                name="get_candles",
                description="Recupera i dati OHLCV per un ticker nel periodo specificato",
                inputSchema={
                    "type": "object",
                    "properties": {
                        "ticker": {
                            "type": "string",
                            "description": "Simbolo del ticker (es. FTSEMIB, DAX)"
                        },
                        "timeframe": {
                            "type": "string",
                            "enum": ["1d", "1w", "1m"],
                            "description": "Timeframe delle candele"
                        },
                        "limit": {
                            "type": "integer",
                            "description": "Numero di candele da recuperare",
                            "default": 100
                        }
                    },
                    "required": ["ticker", "timeframe"]
                }
            )
        ]

    @app.call_tool()
    async def call_tool(name: str, arguments: dict) -> list[TextContent]:
        if name == "get_candles":
            # qui va la logica reale che interroga MySQL di MAOTrade
            data = await fetch_candles_from_db(
                ticker=arguments["ticker"],
                timeframe=arguments["timeframe"],
                limit=arguments.get("limit", 100)
            )
            return [TextContent(type="text", text=str(data))]
```

**Regola pratica:** se stai pensando "voglio che il modello possa fare X", stai pensando
a un tool.

---

### 6.2 — Resources (Dati)

Una resource è un **dato che il modello può leggere**, identificato da una URI.
Non fa azioni, non ha effetti collaterali, è read-only. Pensa a file, documenti,
configurazioni, snapshot di dati.

**Caratteristica chiave:** le resource sono esposte proattivamente dal server, non vengono
"chiamate" con argomenti arbitrari. È l'host a decidere quali includere nel contesto.

```python
    from mcp.types import Resource

    @app.list_resources()
    async def list_resources() -> list[Resource]:
        return [
            Resource(
                uri="maotrade://config/channels",
                name="Configurazione canali Millard",
                description="Parametri dei canali Millard attivi (periodi 9, 51, 153)",
                mimeType="application/json"
            ),
            Resource(
                uri="maotrade://analysis/latest",
                name="Ultima analisi ciclica",
                description="Report dell'ultima analisi ciclica eseguita",
                mimeType="text/plain"
            )
        ]

    @app.read_resource()
    async def read_resource(uri: str) -> str:
        if uri == "maotrade://config/channels":
            config = load_channels_config()
            return json.dumps(config)

        if uri == "maotrade://analysis/latest":
            return load_latest_analysis_report()
```

**Differenza tools vs resources:** se il dato è statico o quasi-statico e vuoi che il modello
lo abbia come contesto, è una resource. Se il modello deve interrogare dati con parametri
variabili (es. "dammi le candele di DAX degli ultimi 52 periodi"), è un tool.

---

### 6.3 — Prompts (Template)

I prompts sono **template di messaggi riutilizzabili** che il server espone. Permettono
al server di definire modi standardizzati di interagire con l'LLM per compiti specifici.
Possono accettare argomenti.

```python
    from mcp.types import Prompt, PromptArgument, GetPromptResult, PromptMessage

    @app.list_prompts()
    async def list_prompts() -> list[Prompt]:
        return [
            Prompt(
                name="analisi_ciclica",
                description="Template per analisi ciclica di un titolo con canali Millard",
                arguments=[
                    PromptArgument(
                        name="ticker",
                        description="Titolo da analizzare",
                        required=True
                    ),
                    PromptArgument(
                        name="timeframe",
                        description="Orizzonte temporale (es. settimanale, mensile)",
                        required=False
                    )
                ]
            )
        ]

    @app.get_prompt()
    async def get_prompt(name: str, arguments: dict) -> GetPromptResult:
        if name == "analisi_ciclica":
            ticker    = arguments["ticker"]
            timeframe = arguments.get("timeframe", "settimanale")

            return GetPromptResult(
                description=f"Analisi ciclica di {ticker}",
                messages=[
                    PromptMessage(
                        role="user",
                        content=TextContent(
                            type="text",
                            text=f"""Esegui un'analisi ciclica completa di {ticker}
                            sul timeframe {timeframe}.
                            Considera: fase del ciclo, posizione nei canali Millard,
                            volume relativo, divergenze degli indicatori."""
                        )
                    )
                ]
            )
```

**Quando usare i prompts:** sono meno usati di tools e resources. Servono quando vuoi
che l'host possa offrire all'utente comandi predefiniti, come slash commands in una chat.

---

### 6.4 — Riepilogo visivo dei tre primitivi

```
┌────────────────────────────────────────────────────────────────────┐
│  Primitivo    Chi decide         Direzione          Uso tipico     │
├────────────────────────────────────────────────────────────────────┤
│  Tools        LLM                LLM → Server       Eseguire       │
│               (model-driven)                        azioni,        │
│                                                     query dinamiche│
│                                                                    │
│  Resources    Host               Server → LLM       Contesto       │
│               (host-driven)                         statico,       │
│                                                     configurazioni │
│                                                                    │
│  Prompts      Utente             Server → Host      Template       │
│               (user-driven)                         riutilizzabili,│
│                                                     slash commands │
└────────────────────────────────────────────────────────────────────┘
```

---

## 7. Confronto con le alternative

Vale la pena capire perché MCP e non qualcos'altro, con una valutazione onesta dei trade-off.

---

### 7.1 — Function calling puro (OpenAI / Anthropic)

```python
    # Function calling senza MCP: tutto in-process
    tools = [{
        "type": "function",
        "function": {
            "name": "get_candles",
            "description": "...",
            "parameters": { "ticker": "string", "timeframe": "string" }
        }
    }]

    response = openai.chat.completions.create(
        model="gpt-4",
        messages=messages,
        tools=tools
    )

    # Se il modello vuole usare il tool, lo esegui tu in-process
    tool_call = response.choices[0].message.tool_calls[0]
    result     = execute_locally(tool_call.function.name, tool_call.function.arguments)
```

**Pro:** semplice, nessuna dipendenza esterna, nessun processo aggiuntivo, massimo controllo.

**Contro:** i tool sono definiti in-process e non sono riusabili tra applicazioni diverse.
Se vuoi usare lo stesso tool con Claude e con GPT-4, lo scrivi due volte in formati diversi.
Se hai tre applicazioni che accedono allo stesso database, hai tre copie della logica.

**Quando ha senso invece di MCP:** quando hai una singola applicazione con tool strettamente
accoppiati all'app stessa. Per ARIA standalone, function calling puro è più semplice.
MCP diventa interessante quando vuoi riusare i server tra applicazioni diverse.

---

### 7.2 — LangChain Tools

```python
    from langchain.tools import BaseTool

    class GetCandlesTool(BaseTool):
        name        = "get_candles"
        description = "Recupera dati OHLCV..."

        def _run(self, ticker: str, timeframe: str) -> str:
            return fetch_from_db(ticker, timeframe)

        async def _arun(self, ticker: str, timeframe: str) -> str:
            return await async_fetch_from_db(ticker, timeframe)
```

**Pro:** integrato nell'ecosistema LangChain, facile se lo usi già.

**Contro:** lock-in su LangChain. Se esci dal framework riscrivi tutto. I tool non sono
riusabili fuori da LangChain. Il framework ha avuto una storia di API instabili e
breaking changes frequenti.

**Quando ha senso:** solo se sei già profondamente dentro LangChain e non hai intenzione
di uscire.

---

### 7.3 — OpenAPI / REST

```yaml
    # openapi.yaml
    paths:
      /candles/{ticker}:
        get:
          operationId: get_candles
          parameters:
            - name: ticker
              in: path
              required: true
              schema:
                type: string
```

**Pro:** standard aperto, enorme ecosistema, documentazione automatica, usabile da
qualsiasi client HTTP.

**Contro:** non pensato per l'uso da parte di LLM. Il modello deve capire lo schema
OpenAPI, gestire autenticazione HTTP, interpretare status code. Non c'è un modo
standardizzato per il server di inviare notifiche al client. Manca il concetto
di "resources" e "prompts".

**Quando ha senso:** se stai costruendo API che devono essere usate sia da LLM sia
da altri client HTTP (app web, mobile, ecc.). Non è una scelta esclusiva rispetto a MCP.

---

### 7.4 — MCP: quando vale davvero

```
MCP è la scelta giusta quando:
    ✓ Vuoi scrivere un tool una volta e usarlo con Claude Desktop,
      Cursor, la tua app, e qualsiasi altro host MCP
    ✓ Stai costruendo un ecosistema di strumenti riusabili
    ✓ Vuoi separare la logica dei tool dall'applicazione host
    ✓ Hai un team che costruisce tool separato da chi costruisce l'app

MCP è overkill quando:
    ✗ Hai una singola applicazione con tool strettamente accoppiati
    ✗ Non hai bisogno di riusabilità cross-application
    ✗ Sei in fase di prototipazione rapida
    ✗ Il tuo team è piccolo e la complessità aggiuntiva non vale il vantaggio
```

Per ARIA così com'è oggi, MCP è probabilmente overkill. Il valore reale arriva se vuoi
esporre le capacità di MAOTrade (dati, analisi, canali Millard) a strumenti esterni
come Claude Desktop o Cursor, o se vuoi che altri sistemi possano usare le stesse
capacità senza reimplementarle.

---

## 8. Riepilogo

```
IL PROBLEMA
    N modelli × M strumenti = N×M integrazioni  (prima di MCP)
    N modelli + M server    = N+M integrazioni   (con MCP)

ARCHITETTURA (tre ruoli distinti)
    Host   → l'applicazione che usa l'LLM, gestisce i client, controlla la sicurezza
    Client → componente dentro l'host, connessione 1:1 con un server
    Server → processo esterno che espone capacità (tools, resources, prompts)
    LLM    → non parla MAI direttamente con il server, passa sempre dall'host

FLUSSO A RUNTIME
    1. Startup: host scopre i tool disponibili dai server (tools/list)
    2. Richiesta utente: host inietta i tool nella chiamata API all'LLM
    3. LLM risponde con tool call (si ferma, non esegue niente)
    4. Host intercetta, esegue via MCP, prende il risultato
    5. Host riprende la conversazione con l'LLM con il risultato

PROTOCOLLO BASE
    JSON-RPC 2.0: request / response / notification in JSON
    Semplice, bidirezionale, un unico schema per tutto

TRANSPORT (il mezzo fisico)
    stdio          → locale, processo figlio, semplice e sicuro
    SSE            → remoto via HTTP, multi-client
    Streamable HTTP → cloud/serverless, il più recente

TRE PRIMITIVI
    Tools      → azioni, il modello decide quando e come (model-driven)
    Resources  → dati read-only, l'host decide cosa includere (host-driven)
    Prompts    → template riutilizzabili, slash commands (user-driven)

QUANDO USARE MCP
    Sì: riusabilità cross-application, ecosistemi di tool condivisi
    No: singola app, prototipo rapido, team piccolo
```

---

> **Da ricordare**
> *MCP non aggiunge intelligenza al modello — aggiunge mani.
> Il modello ragiona sempre, ma senza un host che traduca le sue intenzioni in chiamate reali, quelle intenzioni restano testo su uno schermo.*
