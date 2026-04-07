# Streaming HTTP e LLM: Guida Completa per Sistemi RAG Production-Ready

**Una guida pratica e informale su come implementare streaming in applicazioni AI reali**

---

## Indice

1. [Introduzione: Perché lo Streaming è Importante](#introduzione)
2. [HTTP Streaming: Le Basi](#http-streaming-basi)
3. [Server-Sent Events (SSE)](#server-sent-events)
4. [LLM Streaming: Come Funziona](#llm-streaming)
5. [SDK vs Raw HTTP: Il Confronto](#sdk-vs-raw)
6. [Pattern FastAPI per Streaming](#pattern-fastapi)
7. [Error Handling in Streaming](#error-handling)
8. [Request Cancellation](#request-cancellation)
9. [Production Checklist](#production-checklist)
10. [Lessons Learned](#lessons-learned)

---

## Introduzione: Perché lo Streaming è Importante {#introduzione}

Immagina questa situazione: hai costruito un sistema RAG per analizzare documenti finanziari. Un utente fa una domanda complessa tipo "Analizza il bilancio 2023 e confrontalo con 2022, evidenziando i KPI critici". Il sistema:

1. Cerca nei documenti (2 secondi)
2. Genera la risposta con l'LLM (10-15 secondi)
3. Ritorna tutto insieme

**Risultato:** L'utente guarda uno spinner per 15 secondi, poi riceve tutto in blocco.

**Con streaming:**

1. Cerca nei documenti (2 secondi)
2. Inizia a mostrare la risposta **parola per parola** mentre viene generata
3. L'utente vede immediatamente che qualcosa sta succedendo

La **latenza percepita** passa da 15 secondi a praticamente zero. Non è una questione tecnica, è **UX fondamentale**.

In applicazioni enterprise, specialmente nel fintech, dove le query possono essere lunghe e complesse, lo streaming non è un nice-to-have: è **standard di mercato**.

---

## HTTP Streaming: Le Basi {#http-streaming-basi}

### HTTP Tradizionale (Request-Response)

Il modello classico che tutti conosciamo:

```
Client → Server: POST /api/query
Server: [elabora... 15 secondi...]
Server → Client: 200 OK + JSON completo
```

**Caratteristiche:**

- Sincrono e bloccante
- Il server deve finire tutto prima di rispondere
- Il client aspetta in blocco

**Codice esempio:**

```python
@app.post("/query")
async def query(request: QueryRequest):
    # Search documenti (2 secondi)
    docs = rag_system.search(request.query)

    # LLM generation (10 secondi)
    answer = rag_system.generate(request.query, docs)

    # Ritorna TUTTO insieme dopo 12 secondi
    return {"answer": answer, "sources": docs}
```

### HTTP Streaming (Chunked Transfer)

Invece di aspettare tutto, il server invia **pezzi** man mano che li produce:

```
Client → Server: POST /api/query-stream
Server → Client: [chunk1] [chunk2] [chunk3] ... [chunkN]
                  ↑ Invio mentre elaboro
```

**Caratteristiche:**

- Connessione resta aperta
- Server invia dati incrementalmente
- Client riceve e visualizza in tempo reale

**Codice esempio:**

```python
@app.post("/query-stream")
async def query_stream(request: QueryRequest):
    async def event_generator():
        # Search documenti
        docs = rag_system.search(request.query)

        # LLM streaming - ogni token arriva subito
        async for token in llm_provider.stream(request.query, docs):
            yield f"data: {json.dumps({'token': token})}\n\n"

    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

### Le Tre Tecnologie per Streaming HTTP

Quando devi implementare streaming, hai tre opzioni:

#### 1. Chunked Transfer Encoding (HTTP/1.1)

```http
Transfer-Encoding: chunked

Response body arriva in chunks:
5\r\n
Hello\r\n
6\r\n
 World\r\n
0\r\n
\r\n
```

**Pro:** Standard HTTP, supportato ovunque  
**Contro:** Client deve gestire manualmente i chunk

#### 2. Server-Sent Events (SSE)

```http
Content-Type: text/event-stream

data: {"token": "Il"}

data: {"token": "bilancio"}

data: {"token": "2023"}
```

**Pro:** Standard W3C, auto-reconnect, struttura eventi  
**Contro:** Solo dal server → client (unidirezionale)

#### 3. WebSocket

```http
Upgrade: websocket
Connection: Upgrade

[Connessione bidirezionale full-duplex]
```

**Pro:** Comunicazione bidirezionale real-time  
**Contro:** Overkill per LLM streaming (non serve bidirezionale)

### Quale Scegliere per RAG?

**La risposta: Server-Sent Events (SSE)**

**Perché SSE e non WebSocket?**

1. **Non serve bidirezionale:** L'utente fa una domanda, l'LLM risponde. Fine. Non c'è ping-pong continuo.
2. **Auto-reconnect nativo:** Se la connessione cade, SSE riprende automaticamente.
3. **Più semplice:** HTTP standard, no handshake complesso.
4. **Firewall-friendly:** Passa attraverso proxy HTTP senza problemi.

**Quando usare WebSocket invece?**

- Chat collaborativa (più utenti in tempo reale)
- Gaming real-time
- Live trading feed (bid/ask che cambiano continuamente)
- Applicazioni dove **il client deve mandare messaggi durante lo streaming**

Per un sistema RAG dove l'utente fa una domanda e aspetta la risposta, SSE è la scelta perfetta.

---

## Server-Sent Events (SSE) {#server-sent-events}

### Mini-Esempio Standalone

Prima di applicarlo al RAG, vediamo come funziona SSE puro. Questo esempio simula un processo lungo (contare da 1 a 10) con streaming dei risultati.

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import asyncio
import json

app = FastAPI()

# ============== VERSIONE TRADIZIONALE (per confronto) ==============
@app.get("/count-sync")
async def count_sync():
    """
    Endpoint tradizionale: aspetta che finisca tutto, poi ritorna.
    Simula 10 secondi di elaborazione.
    """
    result = []
    for i in range(1, 11):
        await asyncio.sleep(1)  # Simula lavoro
        result.append(i)

    # Client aspetta 10 secondi, poi riceve tutto insieme
    return {"numbers": result}


# ============== VERSIONE STREAMING ==============
@app.get("/count-stream")
async def count_stream():
    """
    Endpoint streaming: invia ogni numero appena pronto.
    Client vede i numeri uno alla volta in tempo reale.
    """

    async def number_generator():
        """
        Generator che produce i numeri uno alla volta.
        Questo è il cuore dello streaming.
        """
        for i in range(1, 11):
            await asyncio.sleep(1)  # Simula lavoro

            # Formato SSE: "data: <JSON>\n\n"
            event_data = json.dumps({
                "number": i,
                "message": f"Processing step {i}/10"
            })

            yield f"data: {event_data}\n\n"

        # Evento finale per segnalare fine streaming
        yield f"data: {json.dumps({'done': True})}\n\n"

    # StreamingResponse + text/event-stream = SSE
    return StreamingResponse(
        number_generator(),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
        }
    )
```

**Come testare:**

```bash
# 1. Avvia server
uvicorn mini_streaming_example:app --reload

# 2. Test endpoint tradizionale
curl http://localhost:8000/count-sync
# Aspetta 10 secondi...
# {"numbers":[1,2,3,4,5,6,7,8,9,10]}

# 3. Test endpoint streaming
curl -N http://localhost:8000/count-stream
# Vedi arrivare un numero ogni secondo:
# data: {"number":1,"message":"Processing step 1/10"}
#
# data: {"number":2,"message":"Processing step 2/10"}
# ...
```

### Anatomia del Formato SSE

Ogni messaggio SSE ha questa struttura:

```
data: <payload>\n\n
```

**Regole fondamentali:**

- **`data:`** prefix obbligatorio
- **`\n\n`** (due newline) separa i messaggi
- Payload è tipicamente JSON stringificato

**Esempio completo:**

```
data: {"token": "Il"}\n\n
data: {"token": " bilancio"}\n\n
data: {"token": " 2023"}\n\n
data: {"done": true}\n\n
```

**Formato avanzato con eventi nominati (opzionale):**

```
event: message\n
data: {"text": "Hello"}\n\n

event: error\n
data: {"code": 500}\n\n
```

Per la maggior parte dei casi, il formato semplice `data:` è sufficiente.

### Come Funziona StreamingResponse in FastAPI

```python
from fastapi.responses import StreamingResponse

async def my_generator():
    for i in range(10):
        yield f"data: {i}\n\n"  # Ogni yield invia un chunk
        await asyncio.sleep(1)

return StreamingResponse(
    my_generator(),           # Generator/async generator
    media_type="text/event-stream",  # Tipo MIME per SSE
    headers={
        "Cache-Control": "no-cache",  # No caching (eventi real-time)
        "Connection": "keep-alive",   # Mantiene connessione aperta
    }
)
```

**Cosa succede sotto il cofano:**

1. FastAPI chiama `my_generator()`
2. Ad ogni `yield`, invia il chunk al client **immediatamente**
3. Il generator può fare `await` tra un yield e l'altro (non blocca)
4. Quando il generator finisce (o solleva eccezione), la connessione si chiude

### Differenza Chiave: Generator vs Return

```python
# ❌ SBAGLIATO - Non è streaming
async def fake_streaming():
    result = []
    for i in range(10):
        result.append(f"data: {i}\n\n")
    return "".join(result)  # Tutto insieme alla fine!

# ✅ CORRETTO - Vero streaming
async def real_streaming():
    for i in range(10):
        yield f"data: {i}\n\n"  # Ogni yield = invio immediato
```

**Regola d'oro:** Se usi `yield`, è streaming. Se usi `return`, aspetta tutto.

### Client-Side: Come Consumare SSE

**Browser JavaScript (EventSource API):**

```javascript
const eventSource = new EventSource('/count-stream');

eventSource.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('Received:', data);

    if (data.done) {
        eventSource.close();  // Chiudi connessione
    }
};

eventSource.onerror = (error) => {
    console.error('SSE error:', error);
    eventSource.close();
};
```

**Python (httpx per testing):**

```python
import httpx
import json

with httpx.stream("GET", "http://localhost:8000/count-stream") as response:
    for line in response.iter_lines():
        if line.startswith("data: "):
            data = json.loads(line[6:])  # Rimuove "data: "
            print(data)
```

**curl (da terminale):**

```bash
curl -N http://localhost:8000/count-stream
# -N disabilita buffering (vedi streaming real-time)
```

---

## LLM Streaming: Come Funziona {#llm-streaming}

### Come un LLM Genera Testo (Senza Streaming)

Quando chiami un LLM in modalità normale (non streaming):

```python
# Chiamata sincrona
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "Spiega il bilancio 2023"}],
    max_tokens=1000
)

# Aspetti 10 secondi...
# Poi ricevi TUTTO il testo insieme
print(response.content[0].text)
```

**Cosa succede sotto il cofano:**

1. Server LLM genera il testo **token by token** internamente
2. **Aspetta** che siano generati TUTTI i token
3. Manda la risposta completa al client

**Il problema:** Stai aspettando 10 secondi guardando uno spinner, quando in realtà il server ha già generato le prime 50 parole dopo 2 secondi.

### Come Funziona lo Streaming LLM

Con streaming abilitato:

```python
# Chiamata con streaming
stream = client.messages.create(
    model="claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "Spiega il bilancio 2023"}],
    max_tokens=1000,
    stream=True  # ← Magia qui
)

# Ricevi i token UNO ALLA VOLTA
for event in stream:
    if event.type == "content_block_delta":
        print(event.delta.text, end="", flush=True)
```

**Cosa cambia:**

1. Server LLM genera il testo **token by token**
2. **Ogni token viene inviato SUBITO** via SSE
3. Client visualizza in tempo reale

### Anatomia di uno Streaming Event (Anthropic)

Quando Anthropic fa streaming, manda **eventi tipizzati**. Ecco la sequenza completa:

```json
// 1. Inizio streaming
{
  "type": "message_start",
  "message": {
    "id": "msg_123",
    "role": "assistant",
    "content": []
  }
}

// 2. Inizio blocco contenuto
{
  "type": "content_block_start",
  "index": 0,
  "content_block": {
    "type": "text",
    "text": ""
  }
}

// 3. Token effettivi (questi sono tanti!)
{
  "type": "content_block_delta",
  "index": 0,
  "delta": {
    "type": "text_delta",
    "text": "Il"
  }
}

{
  "type": "content_block_delta",
  "index": 0,
  "delta": {
    "type": "text_delta",
    "text": " bilancio"
  }
}

{
  "type": "content_block_delta",
  "index": 0,
  "delta": {
    "type": "text_delta",
    "text": " 2023"
  }
}

// ... continua per centinaia di eventi ...

// 4. Fine blocco contenuto
{
  "type": "content_block_stop",
  "index": 0
}

// 5. Fine messaggio (con usage stats)
{
  "type": "message_delta",
  "delta": {
    "stop_reason": "end_turn"
  },
  "usage": {
    "output_tokens": 487
  }
}

// 6. Fine streaming
{
  "type": "message_stop"
}
```

**Eventi importanti:**

- `content_block_delta` → Contiene il testo da mostrare (questo è il 90% degli eventi)
- `message_delta` → Contiene usage/costi (arriva alla fine)
- Tutto il resto → Metadata (utili per debug ma puoi ignorarli)

### Differenza Anthropic vs OpenAI Streaming

#### Anthropic (eventi strutturati)

```python
with client.messages.stream(...) as stream:
    for event in stream:
        if event.type == "content_block_delta":
            token = event.delta.text
            print(token, end="")
```

**Caratteristiche:**

- Eventi **tipizzati** e strutturati
- Separazione chiara tra metadata e contenuto
- `usage` arriva alla fine nello specifico evento `message_delta`

#### OpenAI (struttura più semplice)

```python
stream = client.chat.completions.create(..., stream=True)

for chunk in stream:
    if chunk.choices[0].delta.content:
        token = chunk.choices[0].delta.content
        print(token, end="")
```

**Caratteristiche:**

- Struttura più semplice e diretta
- Ogni chunk contiene il delta del contenuto
- Meno metadata, più conciso

**Se hai una provider abstraction** (pattern comune in sistemi production), puoi supportare entrambi senza cambiare l'API esterna.

### Pattern di Integrazione con RAG

Ecco come lo streaming si inserisce nel pipeline RAG:

```
User Query: "Qual è l'EBITDA 2023?"
    ↓
[1] FastAPI Endpoint (/query-stream)
    ↓
[2] RAG System: Semantic Search
    ↓ (2 secondi - NON streaming)
    Retrieved Documents: [doc1, doc2, doc3]
    ↓
[3] Build Prompt con retrieved docs
    ↓
[4] LLM Provider Stream
    ↓ (streaming per 8 secondi)
    Token1 → yield → Client
    Token2 → yield → Client
    Token3 → yield → Client
    ...
    TokenN → yield → Client
    ↓
[5] Streaming finito
```

**Punti chiave:**

1. **Step [2] NON è in streaming** → Devi aspettare che la search finisca, ma è veloce (1-2 secondi), quindi OK

2. **Step [4] È in streaming** → Token arrivano immediatamente, questa è la parte lenta (8-15 secondi), quindi importante

3. **Cost tracking** → Arriva alla fine dello streaming (nel `message_delta` event di Anthropic)

### Eventi che Manderemo al Client

Per un sistema RAG, la sequenza di eventi SSE sarà:

```json
// 1. Documenti trovati (dopo la search)
{"type": "sources", "documents": [
  {"id": "doc1", "page": 5, "content": "..."},
  {"id": "doc2", "page": 12, "content": "..."}
]}

// 2. Token generati (streaming)
{"type": "token", "text": "Il"}
{"type": "token", "text": " bilancio"}
{"type": "token", "text": " 2023"}
{"type": "token", "text": " mostra"}
// ... continua per centinaia di eventi ...

// 3. Usage finale (costi e token)
{"type": "usage", "tokens": 487, "cost": 0.0024}

// 4. Fine
{"type": "done"}
```

**Perché questo ordine:**

- **Sources prima** → Client può mostrare "Sto analizzando doc X, Y, Z..." mentre aspetta i token
- **Tokens uno alla volta** → Streaming granulare, UX ottimale
- **Usage alla fine** → Come ritornano Anthropic/OpenAI nelle loro API
- **Done esplicito** → Client sa quando chiudere connessione (altrimenti resterebbe aperta)

---

## SDK vs Raw HTTP: Il Confronto {#sdk-vs-raw}

Una domanda importante: quando usi un SDK (tipo Anthropic Python SDK), cosa fa **veramente** per te? Vale la pena usarlo o è meglio fare tutto a mano?

Vediamo il confronto concreto.

### Streaming con SDK (High-Level)

Ecco come implementi streaming usando l'SDK Anthropic:

```python
from anthropic import Anthropic

def stream_with_sdk(self, messages, temperature=0.7, max_tokens=1000):
    """
    Streaming usando SDK Anthropic.
    Pulito, semplice, affidabile.
    """

    # Prepara messaggi
    system_msg = None
    conv_messages = []
    for msg in messages:
        if msg.role == "system":
            system_msg = msg.content
        else:
            conv_messages.append({
                "role": msg.role,
                "content": msg.content
            })

    call_kwargs = {
        "model": "claude-sonnet-4-20250514",
        "messages": conv_messages,
        "temperature": temperature,
        "max_tokens": max_tokens,
    }

    if system_msg:
        call_kwargs["system"] = system_msg

    # Streaming con context manager
    with self.client.messages.stream(**call_kwargs) as stream:
        # L'SDK filtra automaticamente gli eventi
        for text in stream.text_stream:
            yield text  # ← Solo il testo, niente eventi raw

        # Metadata finale
        final_message = stream.get_final_message()
        tokens_prompt = final_message.usage.input_tokens
        tokens_completion = final_message.usage.output_tokens
        finish_reason = final_message.stop_reason
```

**Linee di codice: ~30**

### Streaming Senza SDK (Raw HTTP)

Ecco come sarebbe lo stesso codice usando solo HTTP/SSE raw:

```python
import httpx
import json

def stream_raw(self, messages, temperature=0.7, max_tokens=1000):
    """
    Streaming implementato senza SDK - solo HTTP raw.
    Mostra ESATTAMENTE cosa fa l'SDK sotto il cofano.
    """

    # 1. Prepara payload
    system_msg = None
    conv_messages = []
    for msg in messages:
        if msg.role == "system":
            system_msg = msg.content
        else:
            conv_messages.append({
                "role": msg.role,
                "content": msg.content
            })

    # 2. Costruisci richiesta HTTP
    url = "https://api.anthropic.com/v1/messages"

    headers = {
        "x-api-key": self.api_key,
        "anthropic-version": "2023-06-01",
        "content-type": "application/json",
    }

    payload = {
        "model": "claude-sonnet-4-20250514",
        "messages": conv_messages,
        "temperature": temperature,
        "max_tokens": max_tokens,
        "stream": True,  # ← Abilita streaming
    }

    if system_msg:
        payload["system"] = system_msg

    # 3. Apri connessione HTTP streaming
    with httpx.Client(timeout=120.0) as client:
        with client.stream("POST", url, headers=headers, json=payload) as response:

            # Check HTTP status
            if response.status_code != 200:
                error_body = response.read().decode("utf-8")
                raise Exception(f"API Error {response.status_code}: {error_body}")

            # Accumula dati per metadata finale
            full_content = ""
            tokens_prompt = 0
            tokens_completion = 0
            finish_reason = None

            # 4. Processa eventi SSE riga per riga
            for line in response.iter_lines():

                # SSE format: "data: <json>\n\n"
                if not line.strip():
                    continue

                # Rimuovi "data: " prefix
                if not line.startswith("data: "):
                    continue

                json_str = line[6:]

                # Parse JSON event
                try:
                    event = json.loads(json_str)
                except json.JSONDecodeError:
                    continue

                # 5. Processa OGNI TIPO di evento
                event_type = event.get("type")

                # ========== INIZIO MESSAGGIO ==========
                if event_type == "message_start":
                    # Metadata iniziali (id, role, ecc)
                    pass

                # ========== INIZIO BLOCCO CONTENUTO ==========
                elif event_type == "content_block_start":
                    # Indica che inizia un nuovo blocco di testo
                    pass

                # ========== DELTA CONTENUTO (IL TOKEN!) ==========
                elif event_type == "content_block_delta":
                    # QUESTO è l'evento importante!
                    delta = event.get("delta", {})
                    delta_type = delta.get("type")

                    if delta_type == "text_delta":
                        # Estrai il token di testo
                        token = delta.get("text", "")
                        full_content += token

                        # YIELD il token al chiamante
                        yield token

                # ========== FINE BLOCCO CONTENUTO ==========
                elif event_type == "content_block_stop":
                    pass

                # ========== DELTA MESSAGGIO (USAGE!) ==========
                elif event_type == "message_delta":
                    # Contiene stop_reason e usage
                    delta = event.get("delta", {})
                    finish_reason = delta.get("stop_reason")

                    usage = event.get("usage", {})
                    tokens_completion = usage.get("output_tokens", 0)

                # ========== FINE STREAMING ==========
                elif event_type == "message_stop":
                    pass

                # ========== PING (keep-alive) ==========
                elif event_type == "ping":
                    # Anthropic manda ping per tenere connessione viva
                    pass

                # ========== ERRORE ==========
                elif event_type == "error":
                    error_msg = event.get("error", {}).get("message", "Unknown error")
                    raise Exception(f"Anthropic API Error: {error_msg}")

    # 6. Calcola metadata finale
    # (tokens_prompt è complicato da estrarre, usiamo approssimazione)
    tokens_prompt = self.count_tokens(system_msg or "") + sum(
        self.count_tokens(msg["content"]) for msg in conv_messages
    )

    # Salva per uso successivo
    self.last_tokens_prompt = tokens_prompt
    self.last_tokens_completion = tokens_completion
    self.last_finish_reason = finish_reason
```

**Linee di codice: ~150**

### Confronto Diretto

| Aspetto               | SDK                                   | Raw HTTP                                     |
| --------------------- | ------------------------------------- | -------------------------------------------- |
| **Linee di codice**   | ~30                                   | ~150                                         |
| **Gestione SSE**      | Automatica                            | Manuale (parse "data: ", newline)            |
| **Filtraggio eventi** | `.text_stream` fa tutto               | Devi controllare ogni `event.type`           |
| **Metadata finale**   | `.get_final_message()`                | Accumuli manualmente in variabili            |
| **Error handling**    | Gestisce automaticamente              | Devi controllare status code, eventi error   |
| **Context manager**   | Un solo `with`                        | Due `with` nested (client + stream)          |
| **Manutenibilità**    | Ottima (API changes gestiti dall'SDK) | Fragile (se Anthropic cambia formato eventi) |
| **Debugging**         | Più difficile (astrazione)            | Più facile (vedi tutto)                      |
| **Performance**       | Leggermente più lenta (overhead SDK)  | Leggermente più veloce (raw)                 |

### Cosa Fa l'SDK Che Non Vedi

#### 1. Gestione Protocollo SSE

```python
# Raw: devi parsare manualmente
for line in response.iter_lines():
    if line.startswith("data: "):
        json_str = line[6:]  # Rimuovi prefix

# SDK: fatto automaticamente
for text in stream.text_stream:
    # Already parsed and filtered!
```

#### 2. Filtraggio Eventi

```python
# Raw: devi controllare ogni tipo
if event_type == "content_block_delta":
    if delta_type == "text_delta":
        yield token

# SDK: filtra automaticamente
for text in stream.text_stream:
    yield text  # Solo testo, niente controlli
```

#### 3. Ricostruzione Messaggio Finale

```python
# Raw: accumuli manualmente
full_content = ""
tokens_prompt = 0
tokens_completion = 0
# ... per ogni evento, aggiorna le variabili

# SDK: tutto in una chiamata
final = stream.get_final_message()
tokens = final.usage.input_tokens
```

#### 4. Error Handling

```python
# Raw: devi controllare tutto
if response.status_code != 200:
    raise Exception(...)
if event_type == "error":
    raise Exception(...)

# SDK: gestisce automaticamente
# (solleva eccezioni appropriate)
```

### Quando Usare Raw HTTP

Ci sono casi in cui raw HTTP ha senso:

**✅ Usa raw HTTP quando:**

- **Debugging profondo** → Vuoi vedere esattamente quali eventi arrivano e quando
- **Provider senza SDK Python** → LLM custom, self-hosted, API non standard
- **Performance critiche** → Devi ottimizzare ogni millisecondo (molto raro)
- **Logging granulare** → Vuoi loggare ogni singolo evento SSE

**❌ Non usare raw HTTP quando:**

- **Production code normale** → Affidabilità > performance marginale
- **SDK disponibile** → Anthropic, OpenAI hanno ottimi SDK
- **Team development** → Codice più leggibile = meno bug

### La Lezione

**150 righe → 10 righe = 15x reduction**

Questo è il valore di un **buon SDK**. Non solo scrivi meno codice, ma:

- **Meno bug** → L'SDK è testato su milioni di chiamate
- **Più manutenibile** → API changes gestiti dall'SDK con backward compatibility
- **Più leggibile** → Intent chiaro, no boilerplate

**Il 99% dei casi: usa l'SDK.**

Ma è importante **capire cosa fa sotto il cofano** (il raw HTTP) per:

- Debugging efficace quando qualcosa va storto
- Sapere se l'SDK è appropriato per il tuo caso
- Fare scelte architetturali informate

---

## Pattern FastAPI per Streaming {#pattern-fastapi}

Ora che sai come funziona SSE e streaming LLM, vediamo i **pattern concreti** per integrarlo in FastAPI.

### Pattern 1: Basic Streaming Endpoint

Il pattern più semplice:

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import time
import json

app = FastAPI()

@app.post("/stream-basic")
async def stream_basic():
    """
    Pattern base: generator sincrono che yield stringhe.
    FastAPI gestisce automaticamente il threading.
    """

    def event_generator():
        for i in range(10):
            # Simula lavoro
            time.sleep(1)

            # Yield evento SSE
            event = {"number": i, "message": f"Processing {i}"}
            yield f"data: {json.dumps(event)}\n\n"

    return StreamingResponse(
        event_generator(),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
        }
    )
```

**Caratteristiche:**

- Generator sincrono (`def`, non `async def`)
- FastAPI wrappa in thread pool automaticamente
- OK per I/O blocking (chiamate API esterne)

**Quando usare:** Quando la tua logica è sincrona o quando chiami librerie che non supportano async.

### Pattern 2: Async Streaming

Versione async (più efficiente):

```python
@app.post("/stream-async")
async def stream_async():
    """
    Pattern async: generator asincrono.
    Più efficiente se fai async I/O.
    """

    async def event_generator():
        for i in range(10):
            # Async I/O non blocca event loop
            await asyncio.sleep(1)

            event = {"number": i}
            yield f"data: {json.dumps(event)}\n\n"

    return StreamingResponse(
        event_generator(),
        media_type="text/event-stream"
    )
```

**Quando usare:**

- Hai async I/O (httpx async, database async, ecc.)
- Vuoi massima concurrency
- Provider LLM supporta async

**Trade-off:**

- ✅ Più efficiente per alta concorrenza
- ❌ Richiede `async/await` dappertutto
- ❌ Debugging più complesso

### Pattern 3: Streaming con Business Logic Separation

Il pattern **production-grade** - separa business logic da HTTP layer:

```python
# services/my_service.py
class MyService:
    """
    Business logic separata dall'HTTP layer.
    Ritorna generator, non StreamingResponse.
    """

    def process_stream(self, query: str):
        """Questo metodo può essere testato senza FastAPI!"""

        # 1. Metadata iniziali
        yield {"type": "start", "query": query}

        # 2. Processing
        for i in range(10):
            result = self._do_work(i)  # Business logic pura
            yield {"type": "result", "data": result}

        # 3. Finale
        yield {"type": "done"}

    def _do_work(self, i):
        # Business logic pura - testabile separatamente
        return i * 2


# controllers/my_controller.py
@app.post("/stream-service")
async def stream_service(query: str):
    """
    Controller HTTP - thin wrapper.
    Converte generator in SSE format.
    """

    service = MyService()

    def event_generator():
        for event in service.process_stream(query):
            # Wrapper: dict → SSE format
            yield f"data: {json.dumps(event)}\n\n"

    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

**Vantaggi:**

- ✅ Business logic testabile separatamente (no HTTP mock)
- ✅ Controller sottile (solo HTTP concerns)
- ✅ Riusabile (stessa logica per CLI, API, job batch)
- ✅ Chiara separazione di responsabilità

**Questo è il pattern usato in sistemi RAG production!**

### Pattern 4: Streaming con Progress Tracking

Utile per elaborazioni lunghe:

```python
@app.post("/stream-progress")
async def stream_progress():
    """
    Invia progress updates durante elaborazione lunga.
    Utile per processing file, batch operations, ecc.
    """

    def event_generator():
        total_steps = 100

        for step in range(total_steps):
            # Simula lavoro
            time.sleep(0.1)

            # Progress update
            progress = {
                "type": "progress",
                "current": step + 1,
                "total": total_steps,
                "percent": ((step + 1) / total_steps) * 100
            }
            yield f"data: {json.dumps(progress)}\n\n"

        # Risultato finale
        result = {"type": "result", "data": "Processing complete"}
        yield f"data: {json.dumps(result)}\n\n"

    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

**Use case:** Processing file lunghi, batch operations, training ML models, analisi documenti complessi.

### Pattern 5: Streaming con Timeout Protection

Evita connessioni infinite:

```python
from fastapi import HTTPException
import time

@app.post("/stream-timeout")
async def stream_timeout():
    """
    Timeout per evitare connessioni infinite.
    """

    def event_generator():
        start_time = time.time()
        max_duration = 60  # 60 secondi max

        for i in range(1000):  # Potenzialmente molto lungo
            # Check timeout
            if time.time() - start_time > max_duration:
                error = {"type": "error", "message": "Timeout exceeded"}
                yield f"data: {json.dumps(error)}\n\n"
                break

            time.sleep(0.5)
            yield f"data: {json.dumps({'number': i})}\n\n"

    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

**Alternativa - Server timeout:**

```bash
# Configura uvicorn
uvicorn main:app --timeout-keep-alive 120
```

**Best practice:** Timeout server > Timeout applicativo (es. 120s server, 60s app).

### Architettura Completa: RAG con Streaming

Ecco come tutti i pattern si integrano in un sistema RAG:

```
┌──────────────────────────────────────────────┐
│  POST /query-stream                          │
│  FastAPI Endpoint (Controller)               │
│  - Validazione request                       │
│  - Error handling HTTP layer                 │
│  - StreamingResponse wrapper                 │
└──────────────┬───────────────────────────────┘
               │
               ↓
┌──────────────────────────────────────────────┐
│  RAGService.query_stream()                   │
│  Business Logic Layer                        │
│  - Dependency injection                      │
│  - Logging, metrics                          │
│  - Try/catch business errors                 │
└──────────────┬───────────────────────────────┘
               │
               ↓
┌──────────────────────────────────────────────┐
│  CompleteRAGSystem.query_stream()            │
│  RAG Orchestration                           │
│  1. Search documents (sync)                  │
│  2. Yield sources                            │
│  3. Generate answer (streaming)              │
│  4. Yield usage/cost                         │
└──────────────┬───────────────────────────────┘
               │
               ↓
┌──────────────────────────────────────────────┐
│  RAGGenerator.generate_stream()              │
│  Generation Logic                            │
│  - Build prompt                              │
│  - Call LLM provider stream                  │
│  - Yield tokens                              │
└──────────────┬───────────────────────────────┘
               │
               ↓
┌──────────────────────────────────────────────┐
│  LLMProvider.stream()                        │
│  Provider Abstraction                        │
│  - AnthropicProvider / OpenAIProvider        │
│  - Handle API streaming                      │
│  - Yield raw tokens                          │
└─────────────────────────────────────────────┘
```

**Flusso eventi reale:**

```
Client richiesta: "Qual è l'EBITDA 2023?"
    ↓ (2 secondi - search)
Event: {"type": "sources", "documents": [...]}
    ↓ (streaming LLM inizia)
Event: {"type": "token", "text": "Il"}
Event: {"type": "token", "text": " bilancio"}
Event: {"type": "token", "text": " 2023"}
Event: {"type": "token", "text": " mostra"}
... (100+ eventi in 8-10 secondi)
    ↓
Event: {"type": "usage", "tokens": 487, "cost": 0.002}
Event: {"type": "done"}
```

### Testing Streaming Endpoints

#### Test con httpx (Python)

```python
import httpx
import json
import pytest

def test_streaming_endpoint():
    """Test completo di un endpoint streaming."""

    with httpx.Client() as client:
        with client.stream(
            "POST",
            "http://localhost:8000/query-stream",
            json={"query": "Test query"}
        ) as response:

            events = []
            for line in response.iter_lines():
                if line.startswith("data: "):
                    event = json.loads(line[6:])
                    events.append(event)

            # Assertions
            assert events[0]["type"] == "sources"
            assert any(e["type"] == "token" for e in events)
            assert events[-1]["type"] == "done"
```

#### Test con Mock (Unit Testing)

```python
from unittest.mock import Mock

def test_rag_service_streaming():
    """Unit test del service senza HTTP."""

    # Mock LLM provider
    mock_provider = Mock()
    mock_provider.stream.return_value = iter(["token1", "token2", "token3"])

    # Test service
    service = RAGService(provider=mock_provider)
    events = list(service.query_stream("test query"))

    # Verify struttura eventi
    assert events[0]["type"] == "sources"
    assert sum(1 for e in events if e["type"] == "token") == 3
    assert events[-1]["type"] == "done"
```

---

## Error Handling in Streaming {#error-handling}

L'error handling nello streaming è **più complesso** rispetto a request-response tradizionale, perché gli errori possono succedere **durante** lo streaming, quando hai già mandato metà risposta al client.

### I Tre Tipi di Errori

#### 1. Pre-Streaming Errors

Errori che succedono **prima** di iniziare lo streaming:

```python
@app.post("/query-stream")
async def query_stream(request: QueryRequest):

    async def event_generator():
        try:
            # Validazione - può fallire PRIMA dello streaming
            if not request.query:
                yield f'data: {json.dumps({"type": "error", "message": "Empty query"})}\n\n'
                return

            # Search documenti - può fallire PRIMA dello streaming
            docs = rag_system.search(request.query)

        except ValueError as e:
            # Errore di validazione
            yield f'data: {json.dumps({"type": "error", "message": str(e)})}\n\n'
            return

        except Exception as e:
            # Errore generico pre-streaming
            yield f'data: {json.dumps({"type": "error", "message": "Search failed"})}\n\n'
            return

        # Se arriviamo qui, inizia lo streaming...
        async for token in llm_provider.stream(prompt, docs):
            yield f"data: {json.dumps({'type': 'token', 'text': token})}\n\n"

    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

**Caratteristiche:**

- Errore succede prima di mandare qualsiasi evento
- Client riceve subito l'errore
- Connessione si chiude pulita

#### 2. Mid-Streaming Errors

Errori che succedono **durante** lo streaming:

```python
async def event_generator():
    # Pre-streaming va bene
    docs = rag_system.search(query)
    yield f'data: {json.dumps({"type": "sources", "documents": docs})}\n\n'

    try:
        # Streaming - può crashare a metà!
        async for token in llm_provider.stream(prompt, docs):
            yield f"data: {json.dumps({'type': 'token', 'text': token})}\n\n"

    except Exception as e:
        # LLM è crashato a metà streaming!
        error_event = {
            "type": "error",
            "message": str(e),
            "partial_response": True  # Indica che era parziale
        }
        yield f"data: {json.dumps(error_event)}\n\n"
        # Connessione si chiude automaticamente dopo yield
```

**Caratteristiche:**

- Client ha già ricevuto parte della risposta (sources + alcuni token)
- Client riceve evento error
- Client deve gestire risposta parziale (es. mostrare warning "risposta incompleta")

#### 3. Client Disconnect

Client chiude la connessione (es. utente chiude tab):

```python
async def event_generator():
    try:
        async for token in llm_provider.stream(prompt, docs):
            yield f"data: {json.dumps({'type': 'token', 'text': token})}\n\n"
            # Se client disconnette, questa yield solleva CancelledError

    except asyncio.CancelledError:
        # Client ha chiuso connessione
        logger.info("Client disconnected during streaming")

        # Opzionale: cleanup (es. cancella LLM request)
        # await llm_provider.cancel_current_request()

        # Non fare raise se vuoi loggare senza errore
        # raise  # Oppure fai raise per propagare
```

**Caratteristiche:**

- FastAPI detecta chiusura TCP socket
- Solleva `asyncio.CancelledError` automaticamente
- Puoi fare cleanup prima di uscire

### Pattern Error Handling a Layers

Il pattern migliore è gestire errori su **tre livelli** distinti:

```python
# Layer 1: Business Logic (RAG System)
class RAGSystem:
    def query_stream(self, question: str):
        try:
            # Processing
            docs = self.search(question)
            yield {"type": "sources", "documents": docs}

            for token in self.generate_stream(question, docs):
                yield {"type": "token", "text": token}

        except ValueError as e:
            # Errore di business logic
            yield {"type": "error", "message": f"Invalid input: {e}"}

        except Exception as e:
            # Errore generico
            yield {"type": "error", "message": "Processing failed"}


# Layer 2: Service (orchestrazione)
class RAGService:
    def query_stream(self, query: str):
        try:
            for event in self.rag_system.query_stream(query):
                yield event

        except Exception as e:
            # Errori che sfuggono alla business logic
            logger.error(f"Service error: {e}", exc_info=True)
            yield {"type": "error", "message": "Service error"}


# Layer 3: Controller (HTTP)
@app.post("/query-stream")
async def query_stream(request: QueryRequest):

    async def event_generator():
        try:
            for event in rag_service.query_stream(request.query):
                yield f"data: {json.dumps(event)}\n\n"

        except asyncio.CancelledError:
            # Client disconnect
            logger.info("Client disconnected")
            # Non mandare evento - client ha già chiuso

        except Exception as e:
            # Errori HTTP layer (molto rari)
            logger.error(f"HTTP layer error: {e}", exc_info=True)
            yield f'data: {json.dumps({"type": "error", "message": "Internal error"})}\n\n'

    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

**Vantaggi:**

- Ogni layer gestisce i suoi errori
- Logging specifico per tipo di errore
- Client riceve sempre un messaggio utile
- Non esponi dettagli interni al client

### Quando NON Usare Streaming

Ci sono casi dove streaming è **overkill** o addirittura problematico:

❌ **Non usare streaming se:**

- Risposta è piccola (<1KB) → overhead inutile
- Devi fare post-processing sulla risposta completa
- Client non supporta SSE (molto raro ormai)
- Hai bisogno di transazionalità (o tutto o niente)

✅ **Usa streaming quando:**

- Elaborazione lunga (>2 secondi)
- Generazione incrementale (LLM, log analysis)
- Feedback immediato importante per UX
- Vuoi ridurre latenza percepita

---

## Request Cancellation {#request-cancellation}

Una feature avanzata ma importante: permettere all'utente di **fermare** la generazione a metà, come il pulsante "Stop" in Claude.ai o ChatGPT.

### Il Problema

```
User: "Analizza questi 50 bilanci..."
[LLM inizia a rispondere, token streaming...]

User: [clicca STOP]

Server: ??? Come fermo l'LLM a metà???
```

**Challenge:**

- Client può chiudere connessione SSE facilmente (`eventSource.close()`)
- Ma il **server continua a processare** e paghi token!
- LLM API non sa automaticamente che client ha disconnesso

### Approccio 1: Auto-Detection (Zero Changes)

**La buona notizia:** Funziona **gratis** con async generators!

```python
@app.post("/query-stream")
async def query_stream(request: QueryRequest):

    async def event_generator():
        try:
            async for event in rag_system.query_stream(request.query):
                yield f"data: {json.dumps(event)}\n\n"
                # ↑ Se client disconnette, questa yield solleva CancelledError

        except asyncio.CancelledError:
            logger.info("Client disconnected - request cancelled")
            raise  # Re-raise per cleanup

    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

**Come funziona:**

1. Client chiude connessione SSE (`eventSource.close()`)
2. FastAPI detecta chiusura TCP socket
3. Solleva `asyncio.CancelledError` nel generator
4. Generator esce dal loop

**Problema:** L'LLM continua a generare token in background se non lo fermi esplicitamente!

### Approccio 2: Propagare Cancellation al Provider

Per **veramente** fermare l'LLM e non pagare token inutili, devi propagare la cancellation:

```python
# Provider LLM con supporto cancellation
class AnthropicProvider:

    async def stream_async(self, messages, temperature=0.7, max_tokens=1000):
        """
        Async streaming con cancellation support.
        """

        # Prepara request...
        call_kwargs = self._prepare_request(messages, temperature, max_tokens)

        try:
            # Usa async context manager
            async with self.client.messages.stream(**call_kwargs) as stream:

                async for text in stream.text_stream:
                    yield text

                # Metadata finale
                final_message = stream.get_final_message()
                # ... salva tokens, cost, ecc.

        except asyncio.CancelledError:
            # Client ha cancellato!
            logger.info("LLM stream cancelled - stopping generation")

            # Anthropic SDK chiude automaticamente la connessione
            # quando esci dal context manager con eccezione

            raise  # Re-raise per cleanup stack
```

**Key point:** Quando esci dal `async with` a causa di `CancelledError`, l'SDK Anthropic **chiude la connessione HTTP** immediatamente. Questo ferma la generazione lato server Anthropic.

### RAG System con Cancellation

```python
class CompleteRAGSystem:

    async def query_stream(self, question: str):
        """
        Query con streaming, supporta cancellation.
        """

        try:
            # 1. Search documenti (veloce, non serve cancellation)
            docs = self.search(question, k=5)
            yield {"type": "sources", "documents": docs}

            # 2. Generate answer (STREAMING - cancellabile)
            async for token in self.rag_generator.generate_stream_async(question, docs):
                yield {"type": "token", "text": token}

            # 3. Usage finale (solo se completato)
            yield {"type": "usage", "tokens": self.last_tokens, "cost": self.last_cost}
            yield {"type": "done"}

        except asyncio.CancelledError:
            logger.info("RAG query cancelled by client")

            # Yield evento di cancellazione
            yield {"type": "cancelled", "message": "Request cancelled"}

            raise  # Propaga al layer sopra
```

### Client-Side: Come Cancellare

```javascript
let eventSource = null;

// Start streaming
function startQuery(query) {
    eventSource = new EventSource(`/query-stream?query=${query}`);

    eventSource.onmessage = (event) => {
        const data = JSON.parse(event.data);

        if (data.type === "token") {
            appendText(data.text);
        }
        else if (data.type === "done" || data.type === "cancelled") {
            eventSource.close();
            eventSource = null;
        }
    };

    eventSource.onerror = (error) => {
        console.error("SSE error:", error);
        eventSource.close();
        eventSource = null;
    };
}

// Stop button
document.getElementById("stop-btn").addEventListener("click", () => {
    if (eventSource) {
        eventSource.close();  // ← Chiude connessione SSE
        eventSource = null;
        console.log("Request cancelled");
    }
});
```

### Mini-Example: Cancellable Streaming

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import asyncio

app = FastAPI()

@app.get("/cancellable-stream")
async def cancellable_stream():
    """
    Demo di streaming cancellabile.
    Chiudi connessione client per vedere CancelledError in action.
    """

    async def event_generator():
        try:
            for i in range(100):  # Streaming lungo
                await asyncio.sleep(0.5)
                yield f"data: {i}\n\n"

                if i % 10 == 0:
                    print(f"Still streaming... at {i}")

        except asyncio.CancelledError:
            print("❌ Client disconnected! Stopping generation.")

            # Cleanup (es. cancella LLM request, salva stato)
            # await cleanup_resources()

            # Opzionale: manda evento finale
            yield f"data: CANCELLED\n\n"

    return StreamingResponse(event_generator(), media_type="text/event-stream")

# Test: curl http://localhost:8000/cancellable-stream
# Premi Ctrl+C dopo alcuni eventi → vedrai "Client disconnected"
```

### Costo/Beneficio della Cancellation

#### Pro:

- ✅ **UX migliore** - User può fermare risposta se va fuori strada
- ✅ **Risparmio token** - Non paghi token per risposte che user non vuole
- ✅ **Resource management** - Server può liberare risorse

#### Contro:

- ❌ **Complessità async** - Tutto deve essere async
- ❌ **Debugging harder** - CancelledError ovunque nei log
- ❌ **Edge cases** - Cosa succede se cancelli durante search? Durante embed?

### Raccomandazione Pragmatica

**Per progetti portfolio/demo:**

**Fase 1:** Implementa auto-detection (2 righe di codice)

```python
except asyncio.CancelledError:
    logger.info("Client disconnected")
```

**In colloquio puoi dire:**

> "Ho implementato auto-detection di client disconnect. Per production, renderei async tutto lo stack per propagare cancellation al provider LLM e risparmiare token."

**Questo dimostra:**

- ✅ Conosci il problema
- ✅ Sai la soluzione teorica
- ✅ Hai fatto trade-off pragmatico

**Fase 2 (solo se necessario):** Implementa async stack completo con propagazione

Richiede refactoring di tutto in async, ma è la soluzione production-grade vera.

### Alternative "Furbe" (Production Tricks)

#### Timeout Automatico

```python
async def event_generator():
    start_time = time.time()
    MAX_DURATION = 60  # 60 secondi max

    async for event in rag_system.query_stream(query):
        if time.time() - start_time > MAX_DURATION:
            yield f"data: {json.dumps({'type': 'timeout'})}\n\n"
            break
        yield f"data: {json.dumps(event)}\n\n"
```

#### Max Tokens Limit

```python
# Limita token generati
response = llm_provider.stream(messages, max_tokens=500)
```

#### Stop Sequences

```python
# LLM si ferma quando incontra sequenza
response = llm_provider.stream(messages, stop_sequences=["</answer>"])
```

---

## Production Checklist {#production-checklist}

Prima di deployare un sistema con streaming in production, verifica questi punti:

### Infrastructure

- [ ] **Timeout configurati correttamente**
  
  - Server timeout (uvicorn: `--timeout-keep-alive 300`)
  - LLM provider timeout (nel client: `timeout=120.0`)
  - Timeout server > Timeout LLM provider

- [ ] **Headers corretti**
  
  - `Cache-Control: no-cache` (no caching degli eventi)
  - `Connection: keep-alive` (mantiene connessione aperta)
  - `X-Accel-Buffering: no` (nginx compatibility)

- [ ] **Proxy/CDN configuration**
  
  - Alcuni proxy fanno buffering → disabilitalo
  - CloudFlare: Abilita "Rocket Loader off" per SSE
  - nginx: `proxy_buffering off;`

### Error Handling

- [ ] **3-layer error handling**
  
  - Pre-streaming errors (validazione, search fails)
  - Mid-streaming errors (LLM crashes)
  - Client disconnect handling

- [ ] **Logging appropriato**
  
  - Inizio streaming (user, query, timestamp)
  - Fine streaming (duration, tokens, cost)
  - Errori con stack trace
  - Client disconnect (non come error, info level)

- [ ] **Error messages utili al client**
  
  - Eventi `{"type": "error", "message": "..."}` chiari
  - Non esporre dettagli interni
  - Indicare se risposta è parziale

### Performance & Monitoring

- [ ] **Metrics tracking**
  
  - Latency primo token (time to first byte)
  - Durata totale streaming
  - Token/secondo (throughput)
  - Rate di cancellation (quanti user clickano stop)

- [ ] **Cost tracking**
  
  - Token usage in evento finale
  - Cost in USD nel response
  - Aggregazione per user/query type

- [ ] **Concurrency limits**
  
  - Max connessioni SSE simultanee
  - Rate limiting per user
  - Queue per evitare overload

### Testing

- [ ] **Unit tests business logic**
  
  - Test generator senza HTTP
  - Mock LLM provider
  - Verifica struttura eventi

- [ ] **Integration tests HTTP**
  
  - Test con httpx streaming
  - Verifica ordine eventi
  - Test error cases

- [ ] **Load testing**
  
  - Simula 100+ connessioni simultanee
  - Verifica memory leaks
  - Check timeout behavior

### Documentation

- [ ] **API documentation**
  
  - Esempio curl
  - Esempio JavaScript EventSource
  - Formato eventi SSE spiegato

- [ ] **Event types documented**
  
  - `{"type": "sources", ...}` - Documenti trovati
  - `{"type": "token", ...}` - Token generato
  - `{"type": "usage", ...}` - Costi finali
  - `{"type": "error", ...}` - Errore
  - `{"type": "done"}` - Fine streaming

- [ ] **Client integration guide**
  
  - Come consumare SSE
  - Error handling client-side
  - Reconnection logic
  - Cancellation (close EventSource)

### Security

- [ ] **Authentication**
  
  - Token/API key validation
  - Rate limiting per user
  - Abuse detection (query troppo lunghe)

- [ ] **Input validation**
  
  - Query length limits
  - Sanitize input
  - Prevent injection attacks

- [ ] **Resource limits**
  
  - Max concurrent streams per user
  - Max stream duration
  - Max tokens per response

---

## Lessons Learned {#lessons-learned}

### 1. Streaming è UX, Non Performance

**La lezione:** Lo streaming non rende l'LLM più veloce. Rende l'**attesa più sopportabile**.

La generazione LLM prende sempre 10 secondi. Ma con streaming:

- Latenza percepita: ~0 secondi (vedi subito il primo token)
- Senza streaming: 10 secondi di spinner

**Per l'utente, la differenza è enorme.** Nel fintech, dove le query sono complesse, streaming è standard di mercato.

### 2. SSE > WebSocket per LLM

**La lezione:** WebSocket è overkill per casi unidirezionali.

Per LLM streaming (server → client):

- ✅ SSE: Standard W3C, auto-reconnect, più semplice
- ❌ WebSocket: Bidirezionale, più complesso, nessun beneficio

Usa WebSocket **solo** quando hai vera comunicazione bidirezionale (chat multiplayer, trading real-time).

### 3. SDK Vale il Suo Peso in Oro

**La lezione:** 150 righe di codice → 10 righe con SDK.

Raw HTTP è utile per capire, ma in production:

- SDK gestisce edge cases testati su milioni di chiamate
- Meno bug, più manutenibile
- API changes gestiti con backward compatibility

**Eccezioni:** Debugging profondo, provider senza SDK, performance estreme.

### 4. Error Handling è Più Complesso

**La lezione:** Con streaming, gli errori possono succedere **durante** la risposta.

Pattern tradizionale (tutto o niente):

```python
try:
    result = process()
    return result
except Exception:
    return error
```

Pattern streaming (errori mid-stream):

```python
yield event1  # ✅ Già mandato
yield event2  # ✅ Già mandato
raise Exception  # ❌ Come gestisco? Client ha già ricevuto metà risposta
```

**Soluzione:** Yield eventi error invece di raise, gestisci 3 layer (pre/mid/post streaming).

### 5. Generator vs Return: Mente il Pattern

**La lezione:** `yield` = streaming real-time, `return` = aspetta tutto.

```python
# ❌ Fake streaming
def fake():
    result = []
    for i in range(10):
        result.append(i)
    return result  # Tutto insieme!

# ✅ Real streaming
def real():
    for i in range(10):
        yield i  # Uno alla volta!
```

Se accumuli in lista e poi ritorni → non è streaming, è buffering.

### 6. Cancellation è Hard ma Importante

**La lezione:** Client può disconnettere, ma server continua a processare (e paghi!).

Auto-detection è gratis:

```python
except asyncio.CancelledError:
    logger.info("Client gone")
```

Ma per **veramente** fermare l'LLM e risparmiare token, serve async stack completo con propagazione.

**Trade-off:** Complessità vs risparmio. Per portfolio, auto-detection basta. Per production, async propagation è giusto.

### 7. Business Logic Separation è Cruciale

**La lezione:** Separa generator puro da HTTP layer.

```python
# ✅ Business logic pura - testabile
class Service:
    def process(query):
        yield {"type": "result", "data": ...}

# ✅ HTTP wrapper - thin
@app.post("/stream")
def endpoint(query):
    for event in service.process(query):
        yield f"data: {json.dumps(event)}\n\n"
```

**Vantaggi:**

- Test senza HTTP mock
- Riusabile (CLI, API, job batch)
- Chiara separazione responsabilità

### 8. Formato Eventi Standardizzato

**La lezione:** Usa struttura consistente per tutti gli eventi.

```json
{"type": "sources", ...}
{"type": "token", ...}
{"type": "usage", ...}
{"type": "error", ...}
{"type": "done"}
```

Campo `type` permette al client di distinguere eventi facilmente. Molto meglio di eventi non tipizzati o formato custom.

### 9. Production ≠ Demo

**La lezione:** Per portfolio, pragmatismo > perfezione.

Demo project:

- ✅ Auto-detection cancellation
- ✅ Basic error handling
- ✅ Generator sincroni (se logica è sync)

Production system:

- ✅ Async stack completo
- ✅ Metrics, logging, monitoring
- ✅ Load testing, security, rate limiting

**Non serve tutto per dimostrare competenza.** Dimostra che **conosci** la soluzione completa e hai fatto **trade-off intelligenti**.

### 10. Headers Fanno la Differenza

**La lezione:** Tre headers critici per SSE:

```python
headers={
    "Cache-Control": "no-cache",     # No cache (real-time)
    "Connection": "keep-alive",      # Mantieni aperto
    "X-Accel-Buffering": "no"        # nginx compatibility
}
```

Senza questi, alcuni proxy/CDN fanno buffering e streaming non funziona. **Sempre includerli.**

### 11. Testing Richiede Approccio Diverso

**La lezione:** Non puoi testare streaming con `assert response.json()`.

Devi iterare sugli eventi:

```python
with client.stream(...) as response:
    events = []
    for line in response.iter_lines():
        if line.startswith("data: "):
            events.append(json.loads(line[6:]))

    assert events[0]["type"] == "sources"
    assert events[-1]["type"] == "done"
```

Unit test la business logic (generator puro), integration test l'HTTP (con streaming).

### 12. Timeout > Max Tokens

**La lezione:** Per evitare stream infiniti, timeout è meglio di max_tokens.

```python
# ❌ Max tokens troncano risposta
llm.stream(messages, max_tokens=500)  # Risposta può essere incompleta

# ✅ Timeout previene stream infiniti
start = time.time()
while time.time() - start < 60:
    yield token
```

Max tokens va usato per **cost control**, timeout per **resource protection**.

### 13. Async vs Sync: Non Sempre Async è Meglio

**La lezione:** Async è più efficiente ma più complesso.

Usa sync quando:

- Logica è sincrona (no await)
- SDK non supporta async
- Semplicità > performance marginale

FastAPI gestisce generator sincroni con thread pool → funziona comunque.

### 14. Log Everything, But Smart

**La lezione:** Logging è critico ma attenzione al rumore.

```python
# ✅ Log eventi importanti
logger.info(f"Stream started: user={user}, query={query[:50]}")
logger.info(f"Stream completed: duration={elapsed}s, tokens={tokens}")

# ❌ Non loggare ogni singolo token
for token in stream:
    logger.debug(f"Token: {token}")  # NO! Troppo rumore
```

Log inizio/fine, errori, metriche aggregate. Non ogni micro-evento.

### 15. La Regola del 80/20

**La lezione finale:** 80% del valore con 20% dello sforzo.

Per streaming production-grade:

- 20% sforzo: Basic streaming con SSE, error handling base
- 80% risultato: UX migliora drasticamente

Per ultimo 20% (async completo, cancellation propagation, monitoring avanzato):

- 80% sforzo
- 20% risultato incrementale

**Fai trade-off intelligenti.** Parti dal 20% che da 80% valore, aggiungi resto solo se serve.

---

## Conclusione

Streaming HTTP con SSE è una tecnica fondamentale per sistemi AI production-grade. Non è complessa da implementare (poche decine di righe di codice), ma richiede **attenzione ai dettagli**:

- Formato eventi consistente
- Error handling a tre livelli
- Testing appropriato
- Headers corretti
- Logging intelligente

La differenza tra "ho fatto un RAG" e "ho fatto un RAG production-ready" spesso sta proprio qui: nello streaming, nell'error handling, nella separazione di concerns.

**Non è teoria astratta.** È quello che distingue un progetto portfolio da un sistema che metteresti in production per davvero.

E soprattutto: **è UX**. Gli utenti lo noteranno subito.

---

*Documento creato come reference personale per implementazione streaming in sistemi RAG. Mantieni questo come guida per future implementazioni e per spiegare decisioni architetturali in colloqui tecnici.*
