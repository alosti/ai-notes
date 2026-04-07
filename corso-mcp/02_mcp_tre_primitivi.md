# Modulo 2 — I Tre Primitivi: Tools, Resources, Prompts

### *Il vocabolario del protocollo*

---

## Indice

1. [Tools](#1-tools)
   
   - Cos'è un tool in MCP
   - Anatomia
   - Discovery
   - Il ciclo di vita di una tool call
   - Il campo `content` nella risposta
   - Error handling
   - Best practice sulle description

2. [Resources](#2-resources)
   
   - Cos'è una resource in MCP
   - URI scheme
   - Anatomia
   - MIME types
   - Statiche vs dinamiche
   - Subscribe e notifiche
   - Come le resource entrano nel flusso
   - I pattern di caricamento

3. [Prompts](#3-prompts)
   
   - Cos'è un prompt in MCP
   - Anatomia
   - Quando usarli vs system prompt hardcoded
   - Usi oltre la scorciatoia utente

4. [Confronto diretto: quando usare cosa](#4-confronto-diretto-quando-usare-cosa)

5. [Massima da ricordare](#massima-da-ricordare)

---

## 1. Tools

### Cos'è un tool in MCP

Un tool è una **funzione che il modello può chiedere di eseguire**. Il concetto è lo stesso del function calling di OpenAI o del tool use di Anthropic — la differenza è che in MCP il tool vive nel server, non nell'applicazione host.

Prima di analizzare la struttura, è utile fissare la distinzione fondamentale tra i tre primitivi, perché tutto il resto dipende da questo:

```
Tool      → il modello decide di usarlo             (model-driven)
            "ho bisogno del prezzo storico di ENI, chiamo get_price_history"

Resource  → l'host decide di caricarlo              (host-driven / app-driven)
            "prima di rispondere, inietto il contesto del portafoglio"

Prompt    → l'utente sceglie di attivarlo           (user-driven)
            "usa il template 'analisi_tecnica' per questo ticker"
```

Questa divisione non è stilistica. Ha implicazioni concrete su chi controlla il flusso, chi consuma token nel decision loop e quali effetti collaterali sono possibili.

---

### Anatomia di un tool

Un tool MCP è composto da tre campi obbligatori:

```json
{
  "name": "get_price_history",
  "description": "Restituisce la serie storica OHLCV per un titolo quotato su MIB o DAX,
                  con granularità configurabile. Usa questo tool quando l'utente chiede
                  dati storici, vuole fare un'analisi tecnica o ha bisogno di dati per
                  costruire un grafico. Non usare per prezzi in tempo reale.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "ticker": {
        "type": "string",
        "description": "Simbolo del titolo (es. 'ENI', 'BMW'). Senza suffisso di mercato."
      },
      "period": {
        "type": "string",
        "enum": ["1d", "1w", "1m", "3m", "1y"],
        "description": "Periodo da coprire a ritroso dalla data odierna"
      },
      "granularity": {
        "type": "string",
        "enum": ["1min", "5min", "1h", "1d"],
        "description": "Granularità delle candele. Default: 1d"
      }
    },
    "required": ["ticker", "period"]
  }
}
```

**`name`** — identificatore usato nella tool call. Deve essere univoco nel server. Convenzione: `snake_case`, senza spazi.

**`description`** — il campo più importante. Non è documentazione per sviluppatori: è l'unica informazione che il modello usa per **decidere se e quando chiamare questo tool**. Se la description è vaga, il modello la chiama quando non dovrebbe, o non la chiama quando dovrebbe. Vedi la sezione dedicata più avanti.

**`inputSchema`** — JSON Schema che descrive i parametri. Il modello usa questo schema per generare la chiamata; l'SDK MCP lo usa per validarla prima di eseguire il tool.

---

### Discovery

La discovery avviene al momento della connessione tra client e server. Il client manda una request `tools/list` e il server risponde con la lista completa:

```
Client → Server:  { "method": "tools/list", "params": {} }

Server → Client:  {
  "tools": [
    { "name": "get_price_history", "description": "...", "inputSchema": {...} },
    { "name": "get_portfolio",     "description": "...", "inputSchema": {...} }
  ]
}
```

Questo avviene **una volta sola** (salvo ri-connessioni o notifiche di cambio lista). L'host inietta poi questi tool nelle chiamate API successive all'LLM, tradotti nel formato che il modello capisce (Anthropic tool use, OpenAI function calling, ecc.).

Se il server aggiorna la lista (per esempio perché l'utente si è autenticato e ora ha accesso a più funzioni), può inviare una notifica. Il client risponde con una nuova `tools/list`:

```
Server → Client (notifica):
  { "method": "notifications/tools/list_changed" }

Client → Server (conseguenza):
  { "method": "tools/list", "params": {} }
```

---

### Il ciclo di vita di una tool call

```
               ┌────────────────────────────────────────────┐
               │                    HOST                    │
Utente         │                                            │
 │             │  ┌──────────┐        ┌──────────────────┐  │   ┌────────┐
 │ "analizza"  │  │          │  API   │                  │  │   │        │
 └────────────►│  │   LLM    │◄──────►│   MCP Client     │◄─┼──►│  MCP   │
               │  │          │ call   │                  │  │   │ Server │
               │  └──────────┘        └──────────────────┘  │   │        │
               │       │                       ▲            │   └────────┘
               │       │ tool_use              │            │
               │       │ {name, input}         │ tools/call │
               │       └───────────────────────┘            │
               │                                            │
               └────────────────────────────────────────────┘
```

**1. L'LLM decide:** ricevuto il messaggio utente (con la lista tool iniettata dall'host), l'LLM risponde con un blocco `tool_use` invece che con testo:

```json
{
  "type": "tool_use",
  "id": "toolu_01abc",
  "name": "get_price_history",
  "input": { "ticker": "ENI", "period": "3m" }
}
```

**2. L'host intercetta:** il client MCP invia una `tools/call` al server:

```json
{
  "method": "tools/call",
  "params": {
    "name": "get_price_history",
    "arguments": { "ticker": "ENI", "period": "3m" }
  }
}
```

**3. Il server esegue e risponde:**

```json
{
  "content": [
    {
      "type": "text",
      "text": "ENI OHLCV ultimi 3 mesi: [{date:'2025-01-06', o:14.2, ...}, ...]"
    }
  ],
  "isError": false
}
```

**4. L'host riprende la conversazione:** manda all'LLM un messaggio `tool_result` con il contenuto restituito dal server. L'LLM ora ha i dati e genera la risposta finale per l'utente.

---

### Il campo `content` nella risposta

La risposta di un tool non è una stringa: è un array di blocchi di contenuto. Questo permette di restituire tipi diversi nello stesso tool:

```json
{
  "content": [
    {
      "type": "text",
      "text": "Caricati 63 campioni giornalieri"
    },
    {
      "type": "text",
      "text": "[{\"date\":\"2025-01-06\",\"open\":14.2,...}]"
    }
  ]
}
```

In teoria si può restituire anche `image` (base64) o `resource` (URI a una resource MCP). In pratica, per la maggior parte dei casi reali si usa `text`.

---

### Error handling

MCP distingue due tipi di errore che è importante non confondere:

**Errore di protocollo** — qualcosa è andato storto nel layer MCP (tool non trovato, schema non valido, timeout di connessione). Il server risponde con un errore JSON-RPC:

```json
{
  "error": {
    "code": -32601,
    "message": "Method not found"
  }
}
```

**Errore applicativo** — il tool è stato trovato e chiamato, ma la logica interna ha fallito (ticker non valido, database non raggiungibile, API esterna down). Il server risponde con `isError: true` nel corpo normale:

```json
{
  "content": [
    {
      "type": "text",
      "text": "Errore: ticker 'XYZ' non trovato nel database MAOTrade"
    }
  ],
  "isError": true
}
```

La distinzione è concreta: l'LLM vede solo il secondo tipo. Con `isError: true` il modello riceve il messaggio di errore come contesto e può rispondere di conseguenza all'utente. Con un errore JSON-RPC il client solleva un'eccezione e l'LLM non vede nulla.

**Regola pratica:** usa errori JSON-RPC solo per problemi tecnici del layer MCP. Usa `isError: true` per qualsiasi errore di business logic che vuoi che il modello possa gestire.

---

### Best practice sulle description

La description è il punto dove la maggior parte delle implementazioni sbaglia. Non è documentazione per sviluppatori: è ciò che il modello legge per capire **quando** usare il tool.

```
❌ Descrizione da documentazione tecnica:
"Fetches OHLCV data from MAOTrade database using the provided ticker 
symbol."

✅ Descrizione pensata per il modello:
"Restituisce la serie storica OHLCV per un titolo. Usa questo tool quando
l'utente chiede dati storici di prezzo, vuole analizzare un trend, 
confrontare performance tra periodi, o ha bisogno di dati per un'analisi 
tecnica. Non usare per prezzi in tempo reale (usa get_realtime_price 
per quelli)."
```

La struttura che funziona meglio:

1. **Cosa fa** — una frase secca
2. **Quando usarlo** — i trigger che devono far scattare la chiamata
3. **Quando NON usarlo** — i confini, specialmente se hai tool simili

Il terzo punto è il più trascurato. Se hai `get_price_history` e `get_realtime_price`, senza indicazioni esplicite il modello può confonderli. Le description devono disambiguare attivamente, non solo descrivere.

Contano anche i `description` dei singoli parametri nello schema:

```json
"ticker": {
  "type": "string",
  "description": "Simbolo nel formato MIB/DAX (es. 'ENI', 'BMW'). Non includere il suffisso di mercato (.MI, .DE)."
}
```

Il modello legge questi campi per capire come valorizzare i parametri. Descrizioni vaghe producono chiamate con valori sbagliati.

---

## 2. Resources

### Cos'è una resource in MCP

Una resource è un **dato che il server espone per essere letto**. Non è un'azione — è un contenuto: un file, un documento, i risultati di una query, lo stato corrente di un sistema.

La differenza fondamentale con i tool:

```
Tool:     il modello chiede di eseguire qualcosa → possibili effetti collaterali
Resource: qualcuno legge qualcosa                → nessun effetto collaterale
```

Le resource sono **idempotenti** per definizione. Non modificano stato. Questa proprietà ha una conseguenza importante: l'host può caricarle automaticamente come contesto prima che il modello inizi a ragionare, cosa che non farebbe mai con un tool (troppo rischio di side effects indesiderati).

---

### URI scheme

Ogni resource è identificata da un URI. Il formato è libero ma deve essere un URI valido. Convenzioni comuni:

```
file:///home/ale/maotrade/config.yaml       # file locale
db://maotrade/portfolio/current             # dato da database
aria://context/user_preferences             # schema custom del server
aria://market/MIB/summary                   # dato di dominio
aria://portfolio/positions/current          # stato applicativo
```

Non esiste uno standard imposto su come costruire gli URI — è responsabilità del server definirli in modo sensato. L'unica regola: devono essere univoci all'interno del server.

---

### Anatomia di una resource

La discovery funziona con `resources/list`, analogo a `tools/list`:

```json
{
  "resources": [
    {
      "uri": "aria://market/MIB/summary",
      "name": "Riepilogo MIB corrente",
      "description": "Stato attuale dell'indice: variazione, volume, breadth",
      "mimeType": "application/json"
    },
    {
      "uri": "file:///data/trading/session_notes.md",
      "name": "Note sessione corrente",
      "description": "Appunti e osservazioni sulla sessione di trading in corso",
      "mimeType": "text/markdown"
    }
  ]
}
```

Per leggere una resource, il client manda `resources/read`:

```json
{ "method": "resources/read", "params": { "uri": "aria://market/MIB/summary" } }
```

La risposta contiene i contenuti:

```json
{
  "contents": [
    {
      "uri": "aria://market/MIB/summary",
      "mimeType": "application/json",
      "text": "{\"variation\": -0.42, \"volume\": 1243000, \"advancing\": 18, \"declining\": 22}"
    }
  ]
}
```

Per contenuti binari (immagini, PDF) si usa `blob` invece di `text`, con il contenuto in base64.

---

### MIME types

Il `mimeType` serve all'host per capire come trattare il contenuto. I tipi più comuni in contesti reali:

```
text/plain            → testo semplice
text/markdown         → Markdown
application/json      → dati strutturati
text/html             → HTML
application/pdf       → documento PDF (come blob base64)
image/png, image/jpeg → immagini (come blob base64)
```

Non è obbligatorio, ma omettere il mimeType significa che l'host deve indovinare.

---

### Resource statiche vs dinamiche

Le resource **statiche** esistono in lista con URI fisso e il loro contenuto non cambia frequentemente. Sono censite in `resources/list` e l'host può fare cache aggressiva.

Le resource **dinamiche** — che MCP chiama *resource templates* — sono URI parametrici che seguono il pattern RFC 6570:

```json
{
  "resourceTemplates": [
    {
      "uriTemplate": "aria://market/{exchange}/ticker/{symbol}",
      "name": "Dati ticker",
      "description": "Dati correnti per un titolo specifico",
      "mimeType": "application/json"
    }
  ]
}
```

Con questo template, l'host può costruire URI a runtime:

```
aria://market/MIB/ticker/ENI
aria://market/DAX/ticker/BMW
```

I template **non compaiono in `resources/list`**, compaiono in `resources/templates/list`. Se l'host non supporta i template, li ignora. È una feature opzionale del protocollo.

---

### Subscribe e notifiche

Le resource possono essere sottoscrivibili: il client si iscrive a un URI e il server notifica quando il contenuto cambia.

```
Client → Server:
  { "method": "resources/subscribe", "params": { "uri": "aria://market/MIB/summary" } }

[... il mercato si muove ...]

Server → Client (notifica):
  {
    "method": "notifications/resources/updated",
    "params": { "uri": "aria://market/MIB/summary" }
  }

Client → Server (va a rileggerla):
  { "method": "resources/read", "params": { "uri": "aria://market/MIB/summary" } }
```

La notifica **non porta il nuovo contenuto** — dice solo "questa resource è cambiata". Il client deve fare esplicitamente una nuova `resources/read`. Scelta di design che costringe un round-trip extra, ma ha senso se il client vuole ignorare la notifica o fare throttling prima di ricaricare.

Per disiscriversi:

```json
{ "method": "resources/unsubscribe", "params": { "uri": "aria://market/MIB/summary" } }
```

**Attenzione:** subscribe funziona solo con transport SSE o Streamable HTTP. Con stdio il server non può inviare notifiche spontanee in modo affidabile. Questo è uno dei motivi per cui SSE esiste come transport alternativo.

---

### Come le resource entrano nel flusso

Il punto che non è ovvio: se il modello non richiede le resource e l'utente non le attiva esplicitamente come i prompt, chi le mette nel flusso e quando?

**L'host.** Prima di chiamare l'LLM, l'host decide quali resource caricare, le legge dal server via `resources/read`, e le inietta nel messaggio come contesto. Il modello le riceve già pronte — dal suo punto di vista sono testo nel contesto, non sa nemmeno che vengono da un server MCP.

Il flusso concreto:

```
1. Utente scrive: "come sta andando ENI oggi?"

2. HOST (prima di chiamare l'LLM):
   → resources/read "aria://portfolio/positions/current"
   → resources/read "aria://context/user_preferences"
   → assembla il messaggio:

     [system]:  "Sei ARIA..."
     [context]: "Posizioni correnti: ENI 500 azioni a 14.20..."
     [context]: "Preferenze: timeframe giornaliero..."
     [user]:    "come sta andando ENI oggi?"

3. LLM risponde con il contesto già disponibile
   → magari chiama anche un tool get_realtime_price
   → ma il portafoglio era già lì, senza un turno extra
```

L'analogia più precisa è **allegare un file a Claude.ai**: quando alleghi un PDF non è il modello che ha "chiesto" il file — l'hai messo nel contesto prima che il modello rispondesse. Le resource in MCP sono esattamente questo, con il processo automatizzato e standardizzato tramite protocollo.

Il vantaggio rispetto a un tool che legge gli stessi dati: se il dato è sempre necessario come contesto, non ha senso sprecare un intero turno di conversazione (modello chiede tool → esegui → restituisci risultato → modello risponde) quando puoi caricarlo prima.

---

### I pattern di caricamento

L'host deve sempre avere una logica per decidere quali resource caricare. Esistono tre pattern con complessità crescente.

**Pattern 1 — Caricamento statico incondizionato**

L'host non deve "sapere" niente. Carica sempre le stesse resource, a prescindere dalla query:

```
Ogni richiesta → carica sempre:
  aria://context/user_preferences
  aria://portfolio/positions/current
  aria://market/session/status
```

Nessun routing, nessuna classificazione. Le resource vengono iniettate nel contesto di sistema a ogni chiamata. Copre una fetta ampia di use case reali: dati che sono sempre utili come contesto di base. Il costo è il consumo di token — si pagano anche quando non servono. Per resource piccole e dati sempre rilevanti, il trade-off è quasi sempre favorevole.

**Pattern 2 — Routing deterministico senza LLM**

L'host usa regole semplici: keyword matching, stato dell'applicazione, ora del giorno:

```python
# nessun LLM per decidere questo
if "portafoglio" in query or "posizioni" in query:
    load_resource("aria://portfolio/positions/current")

if session_state.active_ticker:
    load_resource(f"aria://market/MIB/ticker/{session_state.active_ticker}")

if time_is_market_hours():
    load_resource("aria://market/session/live_summary")
```

Funziona bene per domini limitati dove il vocabolario della query è prevedibile.

**Pattern 3 — Routing semantico**

Per domini ampi dove il matching su keyword non basta, l'host usa un LLM leggero per classificare la query prima di caricare le resource:

```
Query → LLM fast (Haiku/Flash) → intent JSON → resource selection → LLM principale
```

Questo è esattamente il pattern del router di ARIA. È la scelta giusta quando le query sono varie e non riducibili a keyword matching — come in un dominio finanziario dove "cosa pensi di ENI", "analizza il mio portafoglio" e "quando conviene uscire da BMW" richiedono contesti diversi.

Vale la pena dire con chiarezza che stai usando un LLM per decidere quali dati passare a un altro LLM. Ha un costo (latenza + token del pre-processing). Il vantaggio è che il contesto del modello principale rimane pulito e focalizzato. Se il dominio è abbastanza complesso da giustificarlo, il trade-off regge.

**Pattern 4 — Resource come output di un tool**

Meno documentato ma potente: un tool esegue, produce dati, li salva come resource sul server e restituisce l'URI. Il turno successivo l'host carica quella resource come contesto:

```
Tool get_analysis → salva risultati come resource
                  → restituisce URI
                  → host carica la resource al turno successivo come contesto
```

È un modo per gestire stato persistente tra turni di conversazione, più pulito che passare grandi payload JSON come tool result diretto.

---

## 3. Prompts

### Cos'è un prompt in MCP

Un prompt MCP è un **template di messaggio con argomenti dinamici**, esposto dal server e attivabile esplicitamente. La spec lo definisce "user-driven": a differenza del tool (che il modello attiva) o della resource (che l'host carica), il prompt è qualcosa che l'utente o l'interfaccia sceglie di usare.

In Claude Desktop si manifesta come slash commands nella chat: l'utente digita `/analisi_tecnica ENI 3m` e quello attiva un prompt MCP con argomenti `ticker=ENI, period=3m`.

---

### Anatomia di un prompt

Discovery con `prompts/list`:

```json
{
  "prompts": [
    {
      "name": "analisi_tecnica",
      "description": "Analisi tecnica completa: trend, supporti/resistenze, volume, canali",
      "arguments": [
        {
          "name": "ticker",
          "description": "Simbolo del titolo",
          "required": true
        },
        {
          "name": "period",
          "description": "Periodo di analisi (1m, 3m, 1y)",
          "required": false
        }
      ]
    }
  ]
}
```

Per ottenere il prompt con argomenti valorizzati, il client manda `prompts/get`:

```json
{
  "method": "prompts/get",
  "params": {
    "name": "analisi_tecnica",
    "arguments": { "ticker": "ENI", "period": "3m" }
  }
}
```

Il server risponde con la lista di messaggi che compongono il prompt:

```json
{
  "description": "Analisi tecnica ENI su 3 mesi",
  "messages": [
    {
      "role": "user",
      "content": {
        "type": "text",
        "text": "Analizza tecnicamente ENI su 3 mesi. Considera trend primario,
                 supporti e resistenze, volume nelle fasi di avanzamento e ritiro,
                 e costruisci i canali se i dati lo consentono.
                 Riassumi in un giudizio operativo finale."
      }
    }
  ]
}
```

I messaggi possono includere anche contenuto di tipo `resource`, con i dati già embeddati:

```json
{
  "role": "user",
  "content": {
    "type": "resource",
    "resource": {
      "uri": "aria://market/MIB/ticker/ENI",
      "mimeType": "application/json",
      "text": "{...dati ENI pre-caricati dal server...}"
    }
  }
}
```

In questo caso il server ha già embeddato il contenuto della resource nel prompt — non serve un round-trip separato per leggerla.

---

### Quando usarli vs system prompt hardcoded

La documentazione ufficiale non risponde bene a questa domanda, quindi vale la pena essere diretti.

**I prompt MCP hanno senso quando:**

```
✅ Il template deve essere riutilizzabile da più host diversi
   → Lo scrivi nel server MCP e Claude Desktop, Cursor e la tua app
     lo usano tutti allo stesso modo, senza duplicazione

✅ Vuoi che l'utente possa scoprirli e attivarli a runtime
   → L'host mostra la lista, l'utente sceglie quale workflow avviare

✅ Il template ha logica di composizione non banale
   → Il server assembla dati da più resource in base agli argomenti,
     non è una semplice sostituzione di variabili

✅ Team separati: chi costruisce il server è diverso da chi costruisce 
    l'host
   → I prompt diventano un'interfaccia contrattuale tra i due
```

**I prompt MCP sono overkill quando:**

```
❌ Hai una singola applicazione con system prompt fisso
   → Hardcodalo nell'API call, zero vantaggio a metterlo nel server

❌ I "template" sono f-string con sostituzione di variabili
   → Un dizionario di stringhe Python fa lo stesso con meno round-trip

❌ L'host li applica sempre in modo silenzioso senza che l'utente scelga
   → Stai usando i prompt come resource, non ha senso il round-trip extra

❌ Prototipazione o applicazione piccola
   → La complessità aggiuntiva non porta nessun beneficio concreto
```

---

### Usi oltre la scorciatoia utente

La spec definisce i prompt come user-driven, ma questa è una limitazione d'uso, non una limitazione tecnica. `prompts/get` è una chiamata che può fare chiunque — l'host non deve necessariamente aspettare un input esplicito dell'utente.

**Uso dentro un tool:** un tool con logica complessa può richiamare `prompts/get` internamente per costruire il proprio messaggio all'LLM invece di avere il template hardcoded. Permette di aggiornare il template nel server senza toccare il codice del tool.

**Uso in architetture multi-step:** un host orchestratore che gestisce workflow a più fasi può chiamare `prompts/get` per ogni fase, permettendo di configurare il comportamento di ogni step a runtime. Il template di ogni step sta nel server, non nell'host.

Detto questo, in entrambi i casi si tratta di over-engineering se controlli sia il server che l'host. Un template Jinja o una stringa Python fa lo stesso lavoro con meno dipendenze e meno round-trip. Il vantaggio reale dei prompt MCP emerge solo quando il server è sviluppato e deployato indipendentemente dall'host.

---

## 4. Confronto diretto: quando usare cosa

### La mappa decisionale

```
                CHI CONTROLLA L'ATTIVAZIONE?
                            │
       ┌────────────────────┼────────────────────┐
       │                    │                    │
    MODELLO                HOST                UTENTE
       │                    │                    │
       ▼                    ▼                    ▼
      TOOL               RESOURCE             PROMPT

 "ho bisogno di         "prima di rispondere  "l'utente ha scelto
  fare qualcosa"         carica questo        questo workflow"
                         contesto"

  Side effects           Idempotente           Configura il comportamento
  possibili              Solo lettura          Argomenti espliciti

  Query DB               Portafoglio corrente  /analisi_tecnica
  Scrivi file            Config utente         /confronto_titoli
  Chiama API             Note sessione         /report_settimanale
  Calcola indicatori     Stato mercato         /backtest_setup
```

### Domande da farsi per scegliere

**Produce effetti collaterali?**
→ Sì: Tool, sempre.
→ No: vai alla domanda successiva.

**Chi decide quando usarlo?**
→ Il modello in autonomia: Tool (anche senza side effects, se è computazione che il modello innesca).
→ L'applicazione in modo automatico: Resource.
→ L'utente esplicitamente: Prompt.

**Deve essere riusabile su più host?**
→ Sì: qualsiasi primitivo MCP ha senso.
→ No: valuta se MCP serve davvero o bastano pattern più semplici.

### Casi concreti su MAOTrade/ARIA

```
SCENARIO                               PRIMITIVO    PERCHÉ
────────────────────────────────────────────────────────────────────────
Leggere prezzi storici da MySQL        Tool         Il modello decide quando
                                                    serve; query DB è un'azione

Portafoglio corrente come contesto     Resource     Iniettato automaticamente,
                                                    nessun side effect

Calcolare canali su dati               Tool         Computazione che il modello
forniti                                             attiva quando serve

Config utente (preferenze, ticker)     Resource     Letta all'avvio, caricata
                                                    a ogni sessione

Analisi tecnica completa               Prompt       Workflow con argomenti
con un comando dell'utente                          ticker/period scelti dall'utente

Alert su soglie di prezzo              Tool         Scrive stato, produce
                                                    effetti (notifiche)

Ultima analisi salvata su DB           Resource     Dato da leggere, idempotente

Report settimanale standard            Prompt       Workflow ripetibile che
                                                    l'utente attiva esplicitamente
```

### L'errore più comune

L'errore che si vede più spesso nelle implementazioni MCP: **mettere tutto come tool**.

È comprensibile — i tool sono il primitivo più familiare (function calling esiste da prima di MCP) e sembrano abbastanza generici da fare tutto. Ha però un costo reale:

```
1. Context window sprecata
   → I tool consumano token nel prompt (schema + description)
   → Più tool = prompt più lungo = più costoso e più lento

2. Il modello viene sovraccaricato di scelte
   → Con 20 tool, il modello valuta 20 opzioni a ogni turno
   → Aumenta la probabilità di chiamare il tool sbagliato
     o di non chiamare quello giusto

3. Le resource non vengono mai usate
   → Dati che sono solo contesto non dovrebbero passare
     per il decision loop del modello

4. I prompt non vengono mai usati
   → Workflow ripetibili vengono reimplementati ogni volta
     invece di essere esposti come template riutilizzabili
```

La configurazione ottimale bilancia i tre primitivi: tool per le azioni, resource per iniettare contesto in modo efficiente, prompt per esporre workflow standard. Nessuno dei tre è universale.

---

## Riepilogo

```
TOOLS
  Chi decide: il modello (model-driven)
  Side effects: possibili
  Campo critico: description — orienta il modello su quando e come usarli
  Error handling: JSON-RPC per errori MCP, isError:true per business logic
  Discovery: tools/list + notifica tools/list_changed
  Validazione: inputSchema applicato prima dell'esecuzione

RESOURCES
  Chi decide: l'host (app-driven)
  Side effects: nessuno — solo lettura, idempotenti
  Identificate da URI con schema custom per il proprio dominio
  Statiche (in resources/list) o dinamiche (resourceTemplates con URI template)
  Sottoscrivibili: notifications/resources/updated → round-trip resources/read
  Subscribe funziona solo con SSE/HTTP, non con stdio
  Pattern di caricamento: statico, deterministico, semantico (router), o prodotto da tool

PROMPTS
  Chi decide: l'utente (user-driven)
  Template con argomenti espliciti
  Utili quando il server è separato dall'host e i workflow devono essere condivisi
  Overkill per applicazione singola con system prompt fisso
  Tecnicamente chiamabili anche dall'host, ma il vantaggio reale è nell'uso esplicito

QUANDO USARE COSA
  Effetti collaterali o computazione model-driven? → Tool
  Contesto da caricare automaticamente, solo lettura? → Resource
  Workflow che l'utente sceglie esplicitamente? → Prompt
  Controlli sia server che host, team unico, app singola? → Valuta se MCP serve davvero
```

---

> **Da ricordare**
> *Il primitivo sbagliato non rompe il sistema — lo rende semplicemente più costoso, più lento e più fragile di quanto dovrebbe essere. Scegliere tra tool, resource e prompt non è una questione di stile: è una decisione su chi detiene il controllo del flusso e a quale prezzo.*
