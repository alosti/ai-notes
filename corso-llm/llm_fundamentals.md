# Chat 1: LLM Fundamentals + API Setup

## Il Libro di Riferimento del Percorso AI Engineer

*Un viaggio pratico nel mondo degli LLM - Da zero a production-ready*

**Autore:** Ale + Claude  
**Data:** Dicembre 2025  
**Versione:** 1.0  

---

## Prefazione

Questo libro nasce da una conversazione reale, senza filtri, tra un senior backend developer con 25 anni di esperienza (Ale) e un LLM (Claude). 

Non troverai il solito marketing fluff dei tutorial online. Qui dentro c'è:

- Codice vero che funziona
- Errori veri e come risolverli
- Domande scomode e risposte oneste
- Pattern production-ready, non demo per principianti

Il tono è informale, diretto, a volte brutale. Esattamente come dovrebbe essere una conversazione tra engineer che vogliono capire come funzionano davvero le cose.

**Obiettivo della Chat 1:**

- Capire come funzionano gli LLM (no bullshit)
- Setup API pratico (Anthropic Claude)
- Primi script funzionanti
- Prompt engineering essenziale
- Considerazioni production (costi, errori, security)

Tempo stimato: 1-2 ore per leggere + provare codice

---

# PARTE 1: FONDAMENTI

## Capitolo 1: Come Funziona un LLM

### 1.1 - La Verità Scomoda: Non C'è Magia

Un LLM (Large Language Model) è un **predittore di parole estremamente sofisticato**. Niente intelligenza vera, niente coscienza, solo pattern matching su steroidi.

**Il concetto base:**

```
Input: "Il cielo è"

LLM calcola probabilità: 
- "blu" → 45%
- "azzurro" → 30%
- "nuvoloso" → 15%
- "verde" → 0.001%

Sceglie "blu" (o campiona probabilisticamente)
Output: "Il cielo è blu"
```

**Come ci arriva?**

**1. Training Phase** (quella che costa milioni di $):

- Prendi TONNELLATE di testo (web, libri, codice, etc.)
- Il modello legge tutto e impara pattern statistici
- "Dopo 'Il cielo è' di solito viene 'blu' o 'azzurro'"
- Training su GPU farm enormi per settimane/mesi

**2. Inference Phase** (quella che usi tu con le API):

- Mandi un prompt
- Il modello calcola probabilità token per token
- Genera risposta basandosi su pattern visti in training
- Veloce, economico, scalabile

### 1.2 - Tokens: L'Unità di Misura degli LLM

**I LLM non vedono lettere, vedono TOKENS.**

Un token è circa:

- 1 parola italiana = 1-2 tokens
- "Ciao" = 1 token
- "Buongiorno" = 2 tokens ("Buon" + "giorno")
- 1 carattere speciale = 1 token
- Numeri/codice = più tokens

**Esempio pratico:**

```python
Text: "Il mio trading system analizza 5000 operazioni"
Tokens: ["Il", "mio", "trading", "system", "analizza", "5000", "operazioni"]
Count: 7 tokens (circa)
```

**Perché ti interessa?**

- I costi API si pagano a token (input + output)
- I limiti di context window sono in tokens
- Le performance degradano con troppi tokens

### 1.3 - Context Window: La Memoria di Lavoro

Il **context window** è quanto testo il modello può "vedere" in una volta.

**Modelli attuali:**

- GPT-4o: 128k tokens (~100k parole)
- Claude Sonnet 3.5: 200k tokens (~150k parole)
- Claude Opus 3.5: 200k tokens

**Esempio pratico per il trading:**

```python
# Puoi mandargli:
# - 50 pagine di documentazione trading
# - Codice completo di un sistema (se <200k tokens)
# - Storico completo di una chat lunga

# E lui "ricorda" tutto per quella richiesta
```

**MA ATTENZIONE:**

- Il modello non ricorda tra chiamate diverse
- Ogni API call è isolata (stateless)
- Se vuoi "memoria", devi gestirla tu

### 1.4 - Come "Pensa" un LLM

Un LLM **non pensa**. Ma simula pensiero attraverso:

**1. Pattern Recognition:**

- Ha visto 1 milione di esempi di "calcolare Sharpe Ratio"
- Quando chiedi, ricombina pattern visti

**2. In-Context Learning:**

- Se nel prompt gli dai esempi, li usa
- "Few-shot prompting" = dare 2-3 esempi prima della domanda

**3. Reasoning Steps:**

- I modelli migliori possono fare "chain of thought"
- Se chiedi "pensaci step by step", ottieni risultati migliori

**Esempio:**

```
BAD PROMPT:
"Calcola Sharpe ratio di questo portfolio"

GOOD PROMPT:
"Calcola Sharpe ratio seguendo questi step:
1. Calcola return medio giornaliero
2. Calcola deviazione standard returns
3. Applica formula Sharpe = (Return - RiskFree) / StdDev
4. Mostrami calcoli intermedi

Portfolio: [dati]"
```

### 1.5 - Limiti Fondamentali (che DEVI conoscere)

**1. Hallucination (invenzione di cazzate):**

- LLM può inventare fatti con sicurezza assoluta
- Non sa distinguere vero da falso
- **Soluzione:** RAG (Retrieval Augmented Generation) - lo faremo Chat 3

**2. No real-time knowledge:**

- Training data ha una cutoff date
- Claude Sonnet: training fino ~gennaio 2025
- GPT-4o: training fino ~ottobre 2023
- Non sa notizie di oggi

**3. No determinismo:**

- Stessa domanda, risposte diverse (per design)
- Temperature parameter controlla questo

**4. Math/Logic non perfetta:**

- Bravo in pattern, meno in calcoli precisi
- Per math complessa: meglio scrivere codice Python

**5. Token limits:**

- Non puoi mandargli 1GB di dati
- Context window limitato

---

## Capitolo 2: Claude vs GPT - La Scelta

### 2.1 - Perché Abbiamo Scelto Claude

**Claude (Anthropic):**

**Strengths:**

- ✅ Più "honest" (ammette di non sapere)
- ✅ Meno propenso a hallucinations
- ✅ Migliore per reasoning lungo
- ✅ Context window gigante (200k tokens)
- ✅ Migliore con codice (opinione testata)
- ✅ Costi inferiori a parità di qualità

**Weaknesses:**

- ❌ Meno "creativo" (per design, più conservativo)
- ❌ Ecosystem tools meno maturo

**Modelli Claude:**

```
Claude Opus 3.5:    Top tier, costoso, best reasoning
Claude Sonnet 3.5:  Sweet spot, usalo per quasi tutto
Claude Haiku 3.5:   Veloce, economico, task semplici
```

**GPT (OpenAI):**

**Strengths:**

- ✅ Ecosystem più maturo (function calling, assistants API)
- ✅ Più "creativo" (bene per content generation)
- ✅ Integrations ovunque (tutti supportano GPT)
- ✅ o1 models per reasoning pesante

**Weaknesses:**

- ❌ Più propenso a hallucinations
- ❌ Costi più alti
- ❌ Context window più piccolo

### 2.2 - La Differenza nel Modo di Rispondere

**Osservazione di Ale:**

> "Le differenze che ho notato io tra Claude e ChatGPT sono anche nel modo di rispondere, a me ChatGPT mi sembra meno naturale, più artefatta e onestamente non mi piace affatto."

**Perché succede:**

GPT tende ad essere più "corporate-friendly", usa frasi più formali e a volte sembra che stia vendendo qualcosa. Claude è più diretto e ammette quando non sa cose. È design intenzionale di Anthropic - preferiscono "helpful, honest, harmless" anche se significa sembrare meno "impressionante".

### 2.3 - Raccomandazione Pratica per Trading/Fintech

**Primary:** `Claude Sonnet 3.5`

- Best price/performance
- Ottimo con codice Python/SQL
- Reasoning solido per logica trading

**Fallback:** `GPT-4o-mini`

- Se Claude ha downtime
- Per task semplici economici
- Compatibilità ecosystem

**Special use:** `Claude Opus 3.5`

- Solo per task critici complessi
- Es: architecting sistema RAG completo
- Quando costi non sono issue

---

# PARTE 2: SETUP PRATICO

## Capitolo 3: API Setup - Da Zero a Funzionante

### 3.1 - Setup Anthropic (Claude)

**Step 1: Account + API Key**

1. Vai su: https://console.anthropic.com
2. Sign up (usa Google/GitHub)
3. Vai su "API Keys" nel menu
4. "Create Key" → copia la key (tipo `sk-ant-...`)
5. **IMPORTANTE:** Salva in password manager (non la rivedi più)

**Step 2: Credits**

- Anthropic da $5 di crediti gratis
- Bastano per ~1M tokens (un sacco per imparare)
- Poi aggiungi carta se serve

**Step 3: Install Python SDK**

```bash
# Nel tuo ambiente Python
pip install anthropic
```

**Step 4: Test rapido**

Crea file `test_claude.py`:

```python
import anthropic
import os

# Metti API key in environment variable (SEMPRE)
# Mai hardcode in codice!
os.environ["ANTHROPIC_API_KEY"] = "sk-ant-TUA_KEY_QUI"

client = anthropic.Anthropic()

# Prima chiamata
message = client.messages.create(
    model="claude-sonnet-4-20250514",  # Latest Sonnet
    max_tokens=1000,
    messages=[
        {"role": "user", "content": "Ciao Claude, dimmi qualcosa sugli LLM"}
    ]
)

print(message.content[0].text)
```

Lancia:

```bash
python test_claude.py
```

Se funziona, sei setup!

### 3.2 - Best Practice: Environment Variables

**MAI hardcode API keys nel codice!**

Crea file `.env`:

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-tua_key
OPENAI_API_KEY=sk-tua_key
```

Crea file `.gitignore`:

```bash
# .gitignore
.env
__pycache__/
*.pyc
```

Usa nel codice:

```python
from dotenv import load_dotenv
import os

# Carica variables da .env
load_dotenv()

# Ora puoi fare
anthropic_key = os.getenv("ANTHROPIC_API_KEY")
```

---

## Capitolo 4: Primi Script Funzionanti

### 4.1 - Script Base: Chat Semplice

Questo è il pattern fondamentale - 90% di quello che farai parte da qui.

```python
# chat_simple.py
from anthropic import Anthropic
import os
from dotenv import load_dotenv

load_dotenv()

def chat_with_claude(user_message: str) -> str:
    """
    Funzione base per chattare con Claude.

    Args:
        user_message: Il messaggio dell'utente

    Returns:
        La risposta di Claude come stringa
    """
    client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,  # Limite output tokens
        messages=[
            {"role": "user", "content": user_message}
        ]
    )

    # Estrai il testo dalla risposta
    return response.content[0].text


if __name__ == "__main__":
    # Test rapido
    domanda = "Spiega in 3 righe cos'è lo Sharpe Ratio"
    risposta = chat_with_claude(domanda)
    print(f"Domanda: {domanda}\n")
    print(f"Risposta: {risposta}")
```

**Cosa succede sotto il cofano:**

1. `client.messages.create()` → API call HTTP a server Anthropic
2. Il tuo prompt viene tokenizzato
3. Claude genera risposta token per token
4. Response contiene: testo + metadata (tokens usati, etc)
5. Estrai il testo da `response.content[0].text`

### 4.2 - Script con Conversazione Multi-Turn

Claude è **stateless** - non ricorda chiamate precedenti. Devi gestire tu la "memoria".

```python
# chat_conversation.py
from anthropic import Anthropic
import os
from dotenv import load_dotenv

load_dotenv()

class ClaudeChat:
    """
    Gestisce una conversazione con memoria tra chiamate.
    """

    def __init__(self):
        self.client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))
        self.conversation_history = []  # Lista di messaggi

    def send_message(self, user_message: str) -> str:
        """
        Invia messaggio e mantiene storico conversazione.

        Args:
            user_message: Messaggio utente

        Returns:
            Risposta di Claude
        """
        # Aggiungi messaggio utente allo storico
        self.conversation_history.append({
            "role": "user",
            "content": user_message
        })

        # Chiamata API con TUTTO lo storico
        response = self.client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            messages=self.conversation_history  # Passa tutto lo storico
        )

        assistant_message = response.content[0].text

        # Aggiungi risposta Claude allo storico
        self.conversation_history.append({
            "role": "assistant",
            "content": assistant_message
        })

        return assistant_message

    def get_conversation_history(self) -> list:
        """Ritorna storico completo."""
        return self.conversation_history

    def reset(self):
        """Resetta conversazione."""
        self.conversation_history = []


if __name__ == "__main__":
    # Test conversazione multi-turn
    chat = ClaudeChat()

    # Primo messaggio
    resp1 = chat.send_message("Il mio portfolio ha fatto +15% quest'anno")
    print(f"User: Il mio portfolio ha fatto +15% quest'anno")
    print(f"Claude: {resp1}\n")

    # Secondo messaggio - Claude "ricorda" il primo
    resp2 = chat.send_message("È un buon risultato?")
    print(f"User: È un buon risultato?")
    print(f"Claude: {resp2}\n")

    # Terzo messaggio - Claude ricorda entrambi
    resp3 = chat.send_message("E se il benchmark ha fatto +20%?")
    print(f"User: E se il benchmark ha fatto +20%?")
    print(f"Claude: {resp3}")
```

**Concetto chiave:**

- Ogni chiamata API riceve **tutto lo storico**
- `messages = [{user}, {assistant}, {user}, {assistant}, ...]`
- Claude vede tutto e può fare riferimenti a messaggi precedenti
- Tu paghi tokens per TUTTO lo storico ogni volta

**Attenzione costi:**

```python
# Se hai conversazione di 100 messaggi:
# - Messaggio 1: paghi 10 tokens input
# - Messaggio 50: paghi 500 tokens input (storico completo)
# - Messaggio 100: paghi 1000 tokens input

# Soluzione: tronca storico vecchio, o usa summarization
```

### 4.3 - Script con System Prompt

Il **system prompt** è il modo per dare istruzioni "permanenti" a Claude.

```python
# chat_with_system.py
from anthropic import Anthropic
import os
from dotenv import load_dotenv

load_dotenv()

def chat_with_system_prompt(user_message: str, system_prompt: str) -> str:
    """
    Chat con system prompt custom.

    Args:
        user_message: Messaggio utente
        system_prompt: Istruzioni di sistema per Claude

    Returns:
        Risposta di Claude
    """
    client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        system=system_prompt,  # System prompt qui
        messages=[
            {"role": "user", "content": user_message}
        ]
    )

    return response.content[0].text


if __name__ == "__main__":
    # System prompt: trasforma Claude in analista trading
    system_prompt = """
    Sei un analista quantitativo senior specializzato in trading sistematico.

    Regole:
    - Rispondi sempre con dati numerici quando possibile
    - Usa terminologia tecnica (Sharpe, Drawdown, etc.)
    - Sii critico e onesto sui rischi
    - Non dare consigli generici, vai al punto
    """

    domanda = "Il mio sistema ha Sharpe 1.5 e max drawdown 20%. Valutazione?"

    risposta = chat_with_system_prompt(domanda, system_prompt)

    print(f"System Prompt: {system_prompt}\n")
    print(f"Domanda: {domanda}\n")
    print(f"Risposta: {risposta}")
```

**Quando usare system prompt:**

- Definire "personalità" di Claude per tutta la conversazione
- Dare regole di output (formato JSON, linguaggio, etc.)
- Limitare scope ("rispondi solo su trading, ignora altro")

### 4.4 - Script Production-Ready: Error Handling

Nel mondo reale, le API falliscono. Sempre. Devi gestirlo.

```python
# chat_production.py
from anthropic import Anthropic, APIError, APIConnectionError, RateLimitError
import os
from dotenv import load_dotenv
import time
from typing import Optional

load_dotenv()

def chat_with_retry(
    user_message: str,
    max_retries: int = 3,
    system_prompt: Optional[str] = None
) -> str:
    """
    Chat con retry logic e error handling production-grade.

    Args:
        user_message: Messaggio utente
        max_retries: Numero massimo tentativi
        system_prompt: System prompt opzionale

    Returns:
        Risposta di Claude

    Raises:
        Exception: Se tutti i retry falliscono
    """
    client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

    for attempt in range(max_retries):
        try:
            response = client.messages.create(
                model="claude-sonnet-4-20250514",
                max_tokens=1024,
                system=system_prompt,
                messages=[{"role": "user", "content": user_message}]
            )
            return response.content[0].text

        except RateLimitError as e:
            # Rate limit hit - aspetta e riprova
            wait_time = (attempt + 1) * 2  # Exponential backoff
            print(f"Rate limit hit, aspetto {wait_time}s...")
            time.sleep(wait_time)

        except APIConnectionError as e:
            # Network error - riprova
            print(f"Connection error (tentativo {attempt + 1}/{max_retries})")
            time.sleep(1)

        except APIError as e:
            # Generic API error
            print(f"API error: {e}")
            if attempt == max_retries - 1:
                raise  # Ultimo tentativo fallito, rilancia exception
            time.sleep(1)

    raise Exception(f"Failed dopo {max_retries} tentativi")


if __name__ == "__main__":
    try:
        risposta = chat_with_retry(
            "Spiega overfit in machine learning",
            max_retries=3
        )
        print(risposta)

    except Exception as e:
        print(f"Errore finale: {e}")
```

**Pattern production che devi sempre usare:**

1. **Try-Catch specifici** (RateLimitError, ConnectionError, etc.)
2. **Retry con exponential backoff** (aspetta 2s, poi 4s, poi 6s)
3. **Max retries limit** (non loopare infinito)
4. **Logging** (in prod aggiungi logging real)

---

# PARTE 3: PROMPT ENGINEERING

## Capitolo 5: Come Parlare agli LLM

### 5.1 - Principio Base: Clarity is King

LLM non indovina cosa vuoi. Devi essere **esplicito e dettagliato**.

**❌ BAD:**

```python
"Analizza questo trade"
```

**✅ GOOD:**

```python
"""
Analizza questo trade e fornisci:
1. Performance assoluta (%)
2. Confronto con benchmark MIB40
3. Risk metrics (Sharpe, max drawdown)
4. Tempo holding
5. Assessment qualitativo (good/bad/neutral con motivazioni)

Trade data:
- Entry: 2024-01-15, prezzo 35000
- Exit: 2024-02-20, prezzo 37500
- Size: 2 contratti FIB
- Benchmark MIB40 stesso periodo: +4%
"""
```

### 5.2 - Tecnica: Few-Shot Prompting

Dai esempi di input/output che vuoi. LLM impara al volo.

```python
prompt = """
Classifica sentiment di questi titoli di news trading.

Esempi:
Input: "MIB crolla -3%, panico sui mercati"
Output: NEGATIVO

Input: "Borsa Milano chiude piatta, scambi sottili"
Output: NEUTRO

Input: "Rally FIB: +2% su ottimismo export"
Output: POSITIVO

Ora classifica questo:
Input: "Volumi deboli, indici attendisti in vista BCE"
Output:
"""
```

Claude capirà il pattern e risponderà coerentemente.

### 5.3 - Tecnica: Chain of Thought

Chiedi a Claude di "pensare step by step". Migliora accuracy drammaticamente.

**❌ Senza CoT:**

```python
"Calcola se questo trade è profittevole:
Entry €50, Exit €47, commissioni €2"
```

Risposta: "Sì" (SBAGLIATO - non ha considerato commissioni bene)

**✅ Con CoT:**

```python
"Calcola se questo trade è profittevole seguendo questi step:
1. Calcola P&L lordo (Exit - Entry)
2. Sottrai commissioni
3. Calcola P&L netto
4. Determina se profittevole

Trade:
- Entry: €50
- Exit: €47
- Commissioni: €2
"
```

Risposta:

```
1. P&L lordo: €47 - €50 = -€3
2. Commissioni: -€2
3. P&L netto: -€3 - €2 = -€5
4. NON profittevole (loss di €5)
```

### 5.4 - Tecnica: Output Formatting

Specifica **esattamente** il formato output che vuoi.

```python
prompt = """
Analizza performance portfolio e ritorna JSON:

{
  "total_return": <float>,
  "sharpe_ratio": <float>,
  "max_drawdown": <float>,
  "rating": "GOOD" | "AVERAGE" | "POOR",
  "reasoning": "<string>"
}

Portfolio data:
Returns: [0.02, -0.01, 0.03, 0.01, -0.02]
Risk-free rate: 0.05
"""
```

Claude ritornerà JSON parsabile - ottimo per integrare in codice.

### 5.5 - Tecnica: Role Prompting

Assegna un "ruolo" a Claude nel system prompt.

```python
system_prompt = """
Sei un risk manager di un hedge fund quantitativo. 
Hai 20 anni di esperienza e sei estremamente conservativo.

Quando valuti trade:
- Focusati su downside risk prima di upside
- Evidenzia sempre worst-case scenarios
- Non accettare backtest senza walk-forward validation
- Richiedi sempre almeno 3 anni di dati
"""

# Ora Claude risponderà da quella prospettiva
```

### 5.6 - Pattern Avanzato: Prompt con Struttura

Per task complessi, usa struttura XML-like (Claude ama questo formato).

```python
prompt = """
<task>
Valuta robustezza di questo trading system
</task>

<data>
<backtest_period>2020-01-01 to 2024-12-01</backtest_period>
<total_trades>247</total_trades>
<win_rate>0.58</win_rate>
<sharpe_ratio>1.82</sharpe_ratio>
<max_drawdown>0.18</max_drawdown>
<recovery_time_avg>12 days</recovery_time_avg>
</data>

<evaluation_criteria>
1. Statistical significance (min 100 trades)
2. Sharpe ratio (target >1.5)
3. Drawdown management (<20%)
4. Recovery capability
5. Period coverage (min 3 years)
</evaluation_criteria>

<output_format>
Provide:
- PASS/FAIL per ogni criterio
- Overall assessment (ROBUST/DECENT/WEAK)
- Top 3 concerns
</output_format>
"""
```

Questo formato strutturato aiuta Claude a non perdere pezzi.

---

# PARTE 4: PRODUCTION

## Capitolo 6: Costi, Limiti e Security

### 6.1 - Costi Reali

**Claude Sonnet 3.5 pricing (Dicembre 2024):**

- Input: $3 per 1M tokens
- Output: $15 per 1M tokens

**Output costa 5x input!** Questo è importante.

**Esempio calcolo:**

```python
# Scenario: RAG system che risponde a 1000 domande/giorno

# Per domanda:
# - Input: 5000 tokens (context + prompt)
# - Output: 500 tokens (risposta)

# Al giorno:
input_tokens = 1000 * 5000 = 5M tokens
output_tokens = 1000 * 500 = 0.5M tokens

# Costi:
input_cost = (5M / 1M) * $3 = $15
output_cost = (0.5M / 1M) * $15 = $7.50

# Totale giorno: $22.50
# Totale mese: $675
```

**Come abbattere costi:**

1. **Usa modelli più piccoli quando possibile**
   
   - Haiku per task semplici (10x più economico)

2. **Limita output tokens**
   
   - `max_tokens=200` invece di 1024 se basta

3. **Caching di risposte comuni**
   
   - Se domanda già vista, ritorna cached

4. **Prompt compression**
   
   - Manda solo context necessario, non tutto

### 6.2 - Rate Limits

**Claude tier gratuito:**

- 50 requests/minuto
- 40k tokens/minuto

**Se superi:**

- HTTP 429 error
- Devi implementare retry con backoff

**Per production:**

- Richiedi tier paid (aumenta limiti)
- Implementa queue system se carichi pesanti

### 6.3 - Latency Considerations

**Latency tipica Claude API:**

- Cold start: 1-3 secondi
- Risposta breve: 0.5-1 secondo
- Risposta lunga (1000 tokens): 3-5 secondi

**Se latency è critica:**

- Usa streaming (tokens arrivano incrementalmente)
- Cache aggressivo
- Considera self-hosted model (Llama) se <1s requirement

### 6.4 - Security Best Practices

**1. API Keys:**

- ❌ NEVER commit to git
- ✅ Use .env + gitignore
- ✅ Rotate periodicamente
- ✅ Use different keys dev/prod

**2. Input Sanitization:**

```python
def sanitize_user_input(text: str) -> str:
    """
    Pulisci input utente prima di mandarlo a LLM.
    Previene prompt injection attacks.
    """
    # Rimuovi caratteri pericolosi
    dangerous_chars = ['<', '>', '{', '}', 'script', 'eval']

    cleaned = text
    for char in dangerous_chars:
        if char in cleaned.lower():
            print(f"Warning: rilevato carattere sospetto: {char}")
            # Log per security monitoring

    # Limita lunghezza
    max_length = 5000
    if len(cleaned) > max_length:
        cleaned = cleaned[:max_length]

    return cleaned
```

**3. Output Validation:**

```python
def validate_llm_output(response: str) -> bool:
    """
    Verifica che output LLM sia sensato.
    """
    # Check lunghezza minima
    if len(response) < 10:
        return False

    # Check per hallucination comuni
    suspicious_patterns = [
        "As an AI",
        "I cannot",
        "I don't have access to"
    ]

    for pattern in suspicious_patterns:
        if pattern in response:
            print(f"Warning: pattern sospetto in output: {pattern}")

    return True
```

---

# PARTE 5: APPROFONDIMENTI

## Capitolo 7: Gestione Conversazioni Lunghe

### 7.1 - Il Problema

Conversazione lunga = troppi tokens = costi alti + rischio di superare context window.

### 7.2 - Soluzione: Summarization Progressiva

Strategia: quando lo storico diventa troppo lungo, usa Claude stesso per riassumere i messaggi vecchi, poi sostituisci i vecchi messaggi con il summary.

```python
# conversation_with_summarization.py
from anthropic import Anthropic
import os
from dotenv import load_dotenv

load_dotenv()

class SmartConversation:
    """
    Conversazione con summarization automatica dello storico.
    """

    def __init__(self, max_history_tokens=50000):
        self.client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))
        self.conversation_history = []
        self.max_history_tokens = max_history_tokens
        self.summary = None  # Summary dello storico vecchio

    def _estimate_tokens(self, text: str) -> int:
        """
        Stima approssimativa tokens (1 token ~= 4 caratteri per italiano).
        Per produzione usa tiktoken library.
        """
        return len(text) // 4

    def _count_history_tokens(self) -> int:
        """Conta tokens totali nello storico."""
        total = 0
        for msg in self.conversation_history:
            total += self._estimate_tokens(msg["content"])
        return total

    def _summarize_history(self):
        """
        Usa Claude per riassumere storico vecchio.
        Strategia: prendi primi N messaggi, riassumili, sostituiscili con summary.
        """
        if len(self.conversation_history) < 10:
            return  # Non serve summarization ancora

        # Prendi primi 6 messaggi (3 scambi user-assistant)
        messages_to_summarize = self.conversation_history[:6]

        # Costruisci prompt per summarization
        history_text = "\n\n".join([
            f"{msg['role'].upper()}: {msg['content']}"
            for msg in messages_to_summarize
        ])

        summarization_prompt = f"""
        Riassumi questa conversazione in modo conciso ma completo.
        Mantieni tutti i fatti importanti, decisioni prese, e contesto rilevante.

        Conversazione da riassumire:
        {history_text}

        Fornisci summary in 3-5 frasi.
        """

        # Chiamata separata per summary (non usa storico)
        response = self.client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=500,
            messages=[{"role": "user", "content": summarization_prompt}]
        )

        summary_text = response.content[0].text

        # Aggiorna summary cumulativo
        if self.summary:
            # Se già esiste un summary, combinalo con il nuovo
            self.summary = f"{self.summary}\n\n{summary_text}"
        else:
            self.summary = summary_text

        # Rimuovi messaggi vecchi dallo storico
        self.conversation_history = self.conversation_history[6:]

        print(f"[SUMMARIZATION] Rimossi 6 messaggi, summary aggiornato")
        print(f"[SUMMARY] {self.summary}\n")

    def send_message(self, user_message: str) -> str:
        """
        Invia messaggio con gestione automatica summarization.
        """
        # Check se serve summarization
        current_tokens = self._count_history_tokens()
        if current_tokens > self.max_history_tokens:
            self._summarize_history()

        # Aggiungi messaggio utente
        self.conversation_history.append({
            "role": "user",
            "content": user_message
        })

        # Prepara messaggi per API
        # Se c'è un summary, includilo come primo messaggio system-like
        messages_for_api = []

        if self.summary:
            # Inietta summary come contesto all'inizio
            messages_for_api.append({
                "role": "user",
                "content": f"[CONTEXT FROM PREVIOUS CONVERSATION]\n{self.summary}\n\n[CONTINUING CONVERSATION]"
            })
            messages_for_api.append({
                "role": "assistant",
                "content": "Ho letto il contesto precedente, continuo la conversazione."
            })

        # Aggiungi storico recente
        messages_for_api.extend(self.conversation_history)

        # Chiamata API
        response = self.client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            messages=messages_for_api
        )

        assistant_message = response.content[0].text

        # Aggiungi risposta allo storico
        self.conversation_history.append({
            "role": "assistant",
            "content": assistant_message
        })

        return assistant_message
```

**Come funziona:**

1. **Tracking tokens:** Stima tokens dello storico
2. **Threshold:** Quando supera soglia → trigger summarization
3. **Summarization:** Prende vecchi messaggi, li riassume con Claude
4. **Injection:** Summary viene iniettato come "contesto precedente"
5. **Cleanup:** Rimuove vecchi messaggi, mantiene solo recenti + summary

**Vantaggi:**

- ✅ Storico "infinito" senza esplosione costi
- ✅ Claude mantiene contesto generale
- ✅ Costi prevedibili

**Svantaggi:**

- ❌ Si perdono dettagli fini dei messaggi vecchi
- ❌ Costo extra per summarization (ma meno che mandare tutto)

---

## Capitolo 8: Semantic Cache - Cache Intelligente

### 8.1 - Il Problema del Caching Tradizionale

Problema: "Come calcolare Sharpe ratio?" vs "Cos'è lo Sharpe ratio?" → prompt diversi, risposta simile.

Cache tradizionale (exact match) non funziona.

### 8.2 - Soluzione: Semantic Caching con Embeddings

**Prima però: cosa sono gli EMBEDDINGS?**

#### 8.2.1 - Embeddings vs Bag of Words

**Bag of Words (BoW): Il Vecchio Modo**

```python
# Esempio BoW
documenti = [
    "Il trading algoritmico usa Python",
    "Python è ottimo per il trading",
    "Java non è ideale per trading"
]

# Passo 1: Crea vocabolario (tutte le parole uniche)
vocabolario = [
    "il", "trading", "algoritmico", "usa", "python", 
    "è", "ottimo", "per", "java", "non", "ideale"
]

# Passo 2: Rappresenta ogni documento come vettore
doc1_bow = [1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0]
doc2_bow = [1, 1, 0, 0, 1, 1, 1, 1, 0, 0, 0]
doc3_bow = [1, 1, 0, 0, 0, 1, 0, 1, 1, 1, 1]
```

**Problemi GIGANTESCHI di BoW:**

1. **Perde significato semantico:**
   
   ```python
   "Il gatto mangia il topo" 
   "Il topo mangia il gatto"
   # BoW: IDENTICI! (stesse parole, stesso count)
   # Ma significato OPPOSTO!
   ```

2. **Non capisce sinonimi:**
   
   ```python
   "Python è ottimo per trading"
   "Python è eccellente per trading"
   # BoW: completamente diversi (parole diverse)
   # Ma significato UGUALE!
   ```

**Embeddings: La Rivoluzione**

Gli **embeddings** sono rappresentazioni **dense** e **semantiche** del testo.

**Concetto chiave: SIGNIFICATO come VETTORE**

```python
# Embedding = vettore di numeri reali (tipicamente 384, 768, 1024 dimensioni)

text1 = "Python è ottimo per trading"
embedding1 = [0.23, -0.56, 0.89, 0.12, ..., 0.45]  # 384 numeri

text2 = "Python è eccellente per trading"
embedding2 = [0.21, -0.54, 0.87, 0.15, ..., 0.43]  # 384 numeri

# Embeddings SIMILI perché significato SIMILE!
# Anche se parole diverse ("ottimo" vs "eccellente")
```

**Cosine Similarity = misura quanto due vettori puntano nella stessa direzione**

```python
def cosine_similarity(vec1, vec2):
    """
    Similarità coseno: quanto due vettori sono allineati?

    Risultato:
      1.0  = identici (stesso significato)
      0.7+ = molto simili
      0.5  = moderatamente simili
      0.0  = completamente diversi
    """
    dot_product = np.dot(vec1, vec2)
    norm_a = np.linalg.norm(vec1)
    norm_b = np.linalg.norm(vec2)
    return dot_product / (norm_a * norm_b)
```

#### 8.2.2 - Semantic Cache: Implementazione

```python
# semantic_cache_production.py
from anthropic import Anthropic
from sentence_transformers import SentenceTransformer
import numpy as np
import os
from dotenv import load_dotenv

load_dotenv()

class ProductionSemanticCache:
    """
    Cache semantica production-ready con sentence-transformers.
    """

    def __init__(self, model_name='all-MiniLM-L6-v2', similarity_threshold=0.85):
        """
        Args:
            model_name: Modello sentence-transformers (locale, gratis)
            similarity_threshold: Soglia similarità per cache hit
        """
        self.claude_client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

        # Carica modello embeddings (locale, no API calls)
        print("[INIT] Caricamento modello embeddings...")
        self.embedding_model = SentenceTransformer(model_name)
        print("[INIT] Modello caricato")

        self.cache = []
        self.similarity_threshold = similarity_threshold

    def _get_embedding(self, text: str) -> np.ndarray:
        """Genera embedding usando sentence-transformers."""
        return self.embedding_model.encode(text)

    def _cosine_similarity(self, vec1: np.ndarray, vec2: np.ndarray) -> float:
        """Calcola similarità coseno."""
        dot_product = np.dot(vec1, vec2)
        norm1 = np.linalg.norm(vec1)
        norm2 = np.linalg.norm(vec2)
        return dot_product / (norm1 * norm2)

    def ask(self, prompt: str) -> dict:
        """
        Chiede a Claude con caching.

        Returns:
            dict con 'response', 'from_cache', 'similarity'
        """
        # Genera embedding
        prompt_embedding = self._get_embedding(prompt)

        # Cerca in cache
        best_match = None
        best_similarity = 0.0

        for cached_prompt, cached_embedding, cached_response in self.cache:
            similarity = self._cosine_similarity(prompt_embedding, cached_embedding)

            if similarity > best_similarity:
                best_similarity = similarity
                best_match = cached_response

        # Cache hit?
        if best_similarity >= self.similarity_threshold:
            return {
                'response': best_match,
                'from_cache': True,
                'similarity': best_similarity
            }

        # Cache miss - chiama Claude
        response = self.claude_client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            messages=[{"role": "user", "content": prompt}]
        )

        answer = response.content[0].text

        # Aggiungi a cache
        self.cache.append((prompt, prompt_embedding, answer))

        return {
            'response': answer,
            'from_cache': False,
            'similarity': 0.0
        }
```

**Vantaggi semantic cache:**

- ✅ Funziona con prompt variati ("Cos'è X", "Spiegami X", "Che cos'è X")
- ✅ Risparmio enorme (cache hit = $0)
- ✅ Latency bassissima (no API call)

#### 8.2.3 - SentenceTransformers: La Libreria

**Installazione:**

```bash
pip install sentence-transformers
```

**Uso base:**

```python
from sentence_transformers import SentenceTransformer

# Carica modello
model = SentenceTransformer('nome-modello')

# Genera embeddings
texts = ["Testo 1", "Testo 2"]
embeddings = model.encode(texts)
```

**Migliori modelli per ITALIANO:**

**TIER 1: Migliori Overall (Multilingual, supportano italiano bene)**

1. **`paraphrase-multilingual-mpnet-base-v2`**
   
   - Dimensioni: 768D
   - Performance: ★★★★★ (best quality)
   - Speed: ★★★☆☆ (medio)
   - **Best for:** Production chatbot, semantic search, RAG systems

2. **`paraphrase-multilingual-MiniLM-L12-v2`**
   
   - Dimensioni: 384D
   - Performance: ★★★★☆ (ottimo)
   - Speed: ★★★★★ (veloce)
   - **Best for:** Semantic cache (il tuo caso), recommendation, clustering

3. **`distiluse-base-multilingual-cased-v2`**
   
   - Dimensioni: 512D
   - Performance: ★★★★☆
   - **Best for:** General purpose

**Raccomandazioni per Use Case:**

```python
# Per Semantic Cache (tuo caso):
model = SentenceTransformer('paraphrase-multilingual-MiniLM-L12-v2')
# Perché: veloce, qualità ottima, memory-efficient

# Per Chatbot Production:
model = SentenceTransformer('paraphrase-multilingual-mpnet-base-v2')
# Perché: massima accuracy

# Per RAG System:
model = SentenceTransformer('paraphrase-multilingual-MiniLM-L12-v2')
# Perché: speed importante per embedding molti documenti
```

---

## Capitolo 9: Streaming - UX Migliore

### 9.1 - Perché Streaming

Streaming = tokens arrivano uno alla volta, come quando digiti. UX molto migliore.

### 9.2 - Implementazione Base

```python
# streaming_example.py
from anthropic import Anthropic
import os
from dotenv import load_dotenv
import sys

load_dotenv()

def stream_response(prompt: str):
    """
    Streaming di risposta Claude - tokens appaiono in real-time.
    """
    client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

    print("CLAUDE: ", end='', flush=True)

    # stream=True attiva streaming
    with client.messages.stream(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}]
    ) as stream:

        # Itera sui token che arrivano
        for text in stream.text_stream:
            print(text, end='', flush=True)  # Stampa token immediatamente
            # flush=True forza output immediato

    print()  # Newline finale
```

### 9.3 - Streaming con Metadata

```python
def stream_with_metadata(prompt: str):
    """
    Streaming con accesso a metadata completi.
    """
    client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

    print("CLAUDE: ", end='', flush=True)

    accumulated_text = ""

    with client.messages.stream(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}]
    ) as stream:

        for event in stream:
            if event.type == "content_block_delta":
                # Nuovo token arrivato
                if hasattr(event.delta, 'text'):
                    token = event.delta.text
                    accumulated_text += token
                    print(token, end='', flush=True)

            elif event.type == "message_stop":
                # Messaggio completato
                final_message = stream.get_final_message()

                print(f"\n\n[METADATA]")
                print(f"Input tokens: {final_message.usage.input_tokens}")
                print(f"Output tokens: {final_message.usage.output_tokens}")
```

**Vantaggi streaming:**

- ✅ UX migliore (utente vede risposta subito)
- ✅ Perceived latency più bassa
- ✅ Utente può interrompere se risposta non utile

---

## Capitolo 10: Response Object - Proprietà Utili

### 10.1 - Cosa Contiene Response

L'oggetto `response` ha molto più del semplice testo.

```python
# response_inspection.py
from anthropic import Anthropic
import os
from dotenv import load_dotenv
import json

load_dotenv()

def inspect_response_object(prompt: str):
    """
    Esplora tutte le proprietà dell'oggetto response.
    """
    client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=500,
        messages=[{"role": "user", "content": prompt}]
    )

    print("RESPONSE OBJECT COMPLETO")
    print("="*60)

    # 1. ID univoco del messaggio
    print(f"\n1. MESSAGE ID:")
    print(f"   {response.id}")
    print("   → Usa per tracking/logging in produzione")

    # 2. Modello usato
    print(f"\n2. MODEL:")
    print(f"   {response.model}")

    # 3. Stop reason (IMPORTANTE)
    print(f"\n3. STOP REASON:")
    print(f"   {response.stop_reason}")
    print("   Possibili valori:")
    print("   - 'end_turn': risposta completa normale")
    print("   - 'max_tokens': hit limite max_tokens (risposta troncata!)")
    print("   - 'stop_sequence': hit stop sequence custom")
    print("   → CHECK SEMPRE in produzione!")

    # 4. Usage (CRITICO PER COSTI)
    print(f"\n4. USAGE (tokens):")
    print(f"   Input tokens:  {response.usage.input_tokens}")
    print(f"   Output tokens: {response.usage.output_tokens}")

    # Calcola costi
    input_cost = (response.usage.input_tokens / 1_000_000) * 3
    output_cost = (response.usage.output_tokens / 1_000_000) * 15
    total_cost = input_cost + output_cost

    print(f"\n   Costo stimato:")
    print(f"   - Input:  ${input_cost:.6f}")
    print(f"   - Output: ${output_cost:.6f}")
    print(f"   - Total:  ${total_cost:.6f}")
```

### 10.2 - Production Handler Pattern

```python
def production_response_handler(prompt: str) -> dict:
    """
    Pattern production per gestire response con tutti i check.
    """
    client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))

    try:
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            messages=[{"role": "user", "content": prompt}]
        )

        # Estrai informazioni chiave
        result = {
            'success': True,
            'text': response.content[0].text,
            'message_id': response.id,
            'model': response.model,
            'tokens_input': response.usage.input_tokens,
            'tokens_output': response.usage.output_tokens,
            'cost_usd': (
                (response.usage.input_tokens / 1_000_000) * 3 +
                (response.usage.output_tokens / 1_000_000) * 15
            ),
            'truncated': response.stop_reason == "max_tokens",
            'stop_reason': response.stop_reason
        }

        # Warning se troncato
        if result['truncated']:
            print(f"⚠️  WARNING: Response truncated")

        # Logging per monitoring
        print(f"[USAGE] Input: {result['tokens_input']} | "
              f"Output: {result['tokens_output']} | "
              f"Cost: ${result['cost_usd']:.6f}")

        return result

    except Exception as e:
        return {
            'success': False,
            'error': str(e),
            'text': None
        }
```

**Proprietà più importanti:**

1. **`response.usage`** → Monitoring costi
2. **`response.stop_reason`** → Detectare troncamenti
3. **`response.id`** → Logging/tracing
4. **`response.content[0].text`** → La risposta vera

---

# PARTE 6: LESSONS LEARNED

## Capitolo 11: Perché gli LLM Sbagliano i Calcoli

### 11.1 - La Domanda di Ale

> "Perchè gli LLM fanno male i calcoli? Se ti chiedo di calcolarmi i giorni di trading dell'operazione, in chat mi rispondi correttamente 27 giorni, mentre via API mi hai risposto 26. Quando in chat ti ho chiesto perchè mi hai detto che hai fatto calcolo diverso. Questo dimostra che sapete come fare i calcoli giusti, ma alle volte utilizzate formule che danno risultati sbagliati."

### 11.2 - La Verità: Chat vs API

**SCENARIO A: Claude in Chat (con tools)**

```
User: "Calcola giorni trading 2024-01-15 to 2024-02-20"
    ↓
Claude: "Serve calcolo preciso. Uso bash tool."
    ↓
[Esegue Python script reale]
    ↓
Risultato: 27 giorni (CORRETTO, calcolato deterministicamente)
```

**SCENARIO B: Claude via API (senza tools)**

```
User: "Calcola giorni trading 2024-01-15 to 2024-02-20"
    ↓
Claude (NO tools, solo generazione token):
    Token 1: "Calcolo"
    Token 2: "i"
    Token 3: "giorni"
    ...
    Token 50: "26"  ← QUESTO È PROBABILISTICO, NON CALCOLATO!
    Token 51: "giorni"
```

### 11.3 - Perché Sbagliano: Spiegazione Tecnica

**Gli LLM generano token per token:**

```python
# Pseudo-codice di come LLM genera risposta
def generate_response(prompt):
    tokens = []
    context = prompt

    while not done:
        # Per ogni token, calcola probabilità del PROSSIMO
        next_token_probs = model.predict_next_token(context)

        # Sample dal distribution
        next_token = sample(next_token_probs, temperature=0.7)

        tokens.append(next_token)
        context += next_token

    return tokens
```

**Il problema:**

```
Prompt: "Quanti giorni da 2024-01-15 a 2024-02-20?"

Token generation:
  "Ci"        prob: 0.95
  "sono"      prob: 0.92
  "36"        prob: 0.45  ← QUESTO È IL PROBLEMA
  "giorni"    prob: 0.89

Il "36" è una PREDIZIONE, non un CALCOLO.
Il modello ha visto pattern simili in training:
- "da gennaio a febbraio di solito ~30-40 giorni"
- Genera "36" perché statistically plausible
- Ma NON ha fatto: date_diff(2024-02-20, 2024-01-15)
```

### 11.4 - La Soluzione: Hybrid Approach

**Pattern corretto:**

```
User Query
    ↓
Python: Calcola metriche precise ← DETERMINISTICO
    ↓
LLM: Valuta + spiega + consiglia ← REASONING
    ↓
Return combined result
```

**Esempio implementazione:**

```python
# Layer 1: CALCOLI (Python)
def calculate_trade_metrics(entry_price, exit_price, size, 
                           entry_date, exit_date):
    """Calcoli PRECISI usando Python puro."""
    entry_dt = datetime.strptime(entry_date, "%Y-%m-%d")
    exit_dt = datetime.strptime(exit_date, "%Y-%m-%d")

    pnl = (exit_price - entry_price) * size
    return_pct = ((exit_price - entry_price) / entry_price) * 100

    calendar_days = (exit_dt - entry_dt).days

    # Calcolo trading days (escludi weekend)
    trading_days = 0
    current = entry_dt
    while current <= exit_dt:
        if current.weekday() < 5:  # Lun-Ven
            trading_days += 1
        current += timedelta(days=1)

    return {
        "pnl": pnl,
        "return_pct": return_pct,
        "calendar_days": calendar_days,
        "trading_days": trading_days
    }

# Layer 2: ASSESSMENT (LLM)
def get_claude_assessment(metrics):
    """Claude VALUTA dati già calcolati."""
    prompt = f"""Valuta questo trade:
    - Return: {metrics['return_pct']}%
    - P&L: €{metrics['pnl']}
    - Holding: {metrics['trading_days']} giorni trading

    Fornisci assessment qualitativo."""

    # Claude NON ricalcola, solo valuta
    return ask_claude(prompt)
```

### 11.5 - Tabella Decisionale

| Task                         | Via API (no tools) | In Chat (with tools) | **Best Approach**   |
| ---------------------------- | ------------------ | -------------------- | ------------------- |
| **Calcoli matematici**       | ❌ Inaffidabile     | ✅ Affidabile         | **Python sempre**   |
| **Date arithmetic**          | ❌ Spesso sbaglia   | ✅ Tool-based OK      | **Python datetime** |
| **Logica booleana semplice** | ✅ OK               | ✅ OK                 | LLM va bene         |
| **Reasoning qualitativo**    | ✅ Eccellente       | ✅ Eccellente         | **LLM perfetto**    |
| **Pattern recognition**      | ✅ Ottimo           | ✅ Ottimo             | **LLM ideale**      |

---

## Capitolo 12: Prompt By Example - L'Ambiguità

### 12.1 - La Domanda di Ale

> "Quando hai scritto il prompt by example, per questa tipologia di prompt io faccio fatica perchè non so mai quanto il mio esempio sia travisabile. Se invece di chiederti di mettere il vero score nella label score avessi voluto che tu scrivessi sempre il numero 8.5? Se dessero a me quell'esempio l'avrei inteso così!"

### 12.2 - Il Problema: Example Overfitting

```python
# ❌ ESEMPIO AMBIGUO
system_prompt = """
Output format:
{
  "score": 8.5,
  "rating": "GOOD"
}
"""

# Interpretazioni possibili:
# A) score è sempre 8.5 (hardcoded) ← Ale interpreterebbe così!
# B) score è esempio, calcola vero score
# C) score range è 0-10, 8.5 è buon valore

# LLM potrebbe interpretare qualsiasi!
```

### 12.3 - Soluzione 1: Example + Explicit Rules

```python
system_prompt = """
Output format JSON:
{
  "score": <float 0-10>,
  "rating": "EXCELLENT" | "GOOD" | "NEUTRAL" | "POOR" | "BAD"
}

RULES:
- score: CALCOLA valore 0-10 basato su performance
- rating: SCEGLI categoria appropriata

Example con valori VARIABILI:
- Trade +7% in 1 mese → score: 7.5, rating: "GOOD"
- Trade +15% in 1 mese → score: 9.0, rating: "EXCELLENT"
- Trade -5% in 1 mese → score: 3.0, rating: "POOR"
"""
```

**Cosa rende questo non ambiguo:**

- ✅ Tipo dato esplicito: `<float 0-10>`
- ✅ Enum esplicito: valori possibili
- ✅ RULES spiegano che calcolare
- ✅ Multiple examples con valori DIVERSI

### 12.4 - Soluzione 2: Schema + Description (no example)

```python
system_prompt = """
Output format JSON (NO examples, solo schema):

{
  "score": {
    "type": "number",
    "range": [0, 10],
    "description": "Calcola basandoti su: return%, holding time, risk"
  },
  "rating": {
    "type": "enum",
    "values": ["EXCELLENT", "GOOD", "NEUTRAL", "POOR", "BAD"],
    "description": "EXCELLENT: >12%, GOOD: 7-12%, etc."
  }
}

CRITICAL: Tutti i valori devono essere CALCOLATI, non hardcoded.
"""
```

### 12.5 - Checklist: Il Tuo Example È Chiaro?

Prima di usare example, chiediti:

- [ ] Ho usato `<placeholder>` invece di valori hardcoded?
- [ ] Ho dato 3+ examples con valori DIVERSI?
- [ ] Ho scritto explicit rule "calcola X, non copiare example"?
- [ ] Ho testato con edge cases?
- [ ] Example copre range completo di possibilità?

Se anche solo 1 checkbox è NO → il tuo example è ambiguo.

---

## Capitolo 13: L'Esercizio Pratico - Trade Analyzer

### 13.1 - Il Codice di Ale

Ale ha creato un trade analyzer con:

- ✅ Error handling production-grade
- ✅ Separazione concerns (ask_claude, analyze_trade, get_json_response)
- ✅ JSON cleaning robusto
- ✅ Type hints

**Il codice:**

```python
# trade_analyzer.py
import time
from anthropic import Anthropic, RateLimitError, APIConnectionError, APIError
import os
import json
from dotenv import load_dotenv

load_dotenv()

def ask_claude(system_prompt: str, user_message: str) -> str | None:
    """
    Ask data to claude and return response.
    """
    max_retries = 5
    client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))
    response = None

    for attempt in range(max_retries):
        try:
            response = client.messages.create(
                model="claude-sonnet-4-20250514",
                max_tokens=1024,
                system=system_prompt,
                messages=[{"role": "user", "content": user_message}]
            )
            return response.content[0].text

        except RateLimitError as e:
            wait_time = (attempt + 1) * 2
            print(f"Rate limit hit, aspetto {wait_time}s...")
            time.sleep(wait_time)

        except APIConnectionError as e:
            print(f"Connection error (tentativo {attempt + 1}/{max_retries})")
            time.sleep(1)

        except APIError as e:
            print(f"API error: {e}")
            if attempt == max_retries - 1:
                raise
            time.sleep(1)

    return response

def analyze_trade(entry_price, exit_price, size, entry_date, exit_date):
    """Usa Claude per analizzare un trade."""
    system_prompt = """Sei un analista quantitativo senior.

    CRITICAL: Risposta ESCLUSIVAMENTE JSON valido.
    NO markdown, NO testo extra.

    [... resto del prompt ...]
    """

    user_message = f"""Trade data:
    - Entry: {entry_date}, prezzo {entry_price}
    - Exit: {exit_date}, prezzo {exit_price}
    - Size: {size} contratti FIB
    """

    return ask_claude(system_prompt, user_message)

def get_json_response(response: str) -> dict:
    """Pulisce e parsa risposta JSON."""
    if not response:
        return {}

    try:
        cleaned = response.strip()

        # Rimuovi markdown
        if cleaned.startswith("```json"):
            cleaned = cleaned[7:]
        if cleaned.startswith("```"):
            cleaned = cleaned[3:]
        if cleaned.endswith("```"):
            cleaned = cleaned[:-3]

        cleaned = cleaned.strip()
        result_dict = json.loads(cleaned)
        return result_dict

    except json.JSONDecodeError as e:
        print(f"❌ Errore parsing JSON: {e}")
        return {}
```

### 13.2 - La Lezione Fondamentale

Ale ha scritto alla fine:

> "La risposta su quanto sono stato dentro nella posizione in termini di giorni di borsa aperta è errata. Da qui la lezione che ne deriva è: NON USARE CLAUDE PER CALCOLI CHE RICHIEDONO DATI ESTERNI, utilizza Claude per opinioni, classificazioni ecc, i calcoli è meglio farli con codice Python."

**QUESTA È LA LEZIONE PIÙ IMPORTANTE DEL CORSO.**

### 13.3 - La Soluzione: Hybrid Analyzer

```python
# Layer 1: CALCOLI (Python)
def calculate_trade_metrics(entry_price, exit_price, size, 
                           entry_date, exit_date):
    """Calcola metriche PRECISE usando Python."""
    # Calcoli deterministici
    pnl = (exit_price - entry_price) * size
    return_pct = ((exit_price - entry_price) / entry_price) * 100

    # Date arithmetic corretto
    entry_dt = datetime.strptime(entry_date, "%Y-%m-%d")
    exit_dt = datetime.strptime(exit_date, "%Y-%m-%d")
    calendar_days = (exit_dt - entry_dt).days

    # Trading days (escludi weekend)
    trading_days = 0
    current = entry_dt
    while current <= exit_dt:
        if current.weekday() < 5:
            trading_days += 1
        current += timedelta(days=1)

    return {
        "performance": {
            "absolute_return_pct": round(return_pct, 2),
            "total_pnl_eur": round(pnl, 2)
        },
        "holding": {
            "calendar_days": calendar_days,
            "trading_days": trading_days
        }
    }

# Layer 2: ASSESSMENT (Claude)
def get_claude_assessment(metrics):
    """Claude VALUTA dati già calcolati."""
    prompt = f"""Valuta questo trade:

    PERFORMANCE:
    - Return: {metrics['performance']['absolute_return_pct']}%
    - P&L: €{metrics['performance']['total_pnl_eur']}

    HOLDING:
    - Giorni trading: {metrics['holding']['trading_days']}

    Fornisci assessment JSON:
    {{
      "rating": "EXCELLENT" | "GOOD" | "NEUTRAL" | "POOR",
      "score": <float 0-10>,
      "strengths": [<list>],
      "concerns": [<list>],
      "recommendation": "<string>"
    }}
    """

    return ask_claude("Sei analista quantitativo", prompt)

# Layer 3: ORCHESTRAZIONE
def analyze_trade_complete(entry_price, exit_price, size, 
                          entry_date, exit_date):
    """Analisi completa: Python calcola + Claude valuta."""

    # Step 1: Calcoli precisi (Python)
    metrics = calculate_trade_metrics(
        entry_price, exit_price, size, entry_date, exit_date
    )

    # Step 2: Assessment qualitativo (Claude)
    assessment = get_claude_assessment(metrics)

    # Step 3: Combina
    return {
        "metrics": metrics,
        "assessment": assessment
    }
```

---

# PARTE 7: LESSONS LEARNED

## Le 15 Lezioni Chiave di Questa Chat

### 1. **LLM ≠ Calcolatrice**

> "NON USARE CLAUDE PER CALCOLI CHE RICHIEDONO DATI ESTERNI"

**Perché:** LLM genera token probabilisticamente, non esegue calcoli deterministici.

**Soluzione:** Python per calcoli, LLM per reasoning.

---

### 2. **Hybrid Approach = Best**

```
Python calcola → LLM valuta → Combined power
```

**Pattern da seguire sempre:**

- Dati precisi → Python
- Interpretazione/assessment → LLM
- Output combinato → User value

---

### 3. **Error Handling Non È Opzionale**

Nel mondo reale:

- API falliscono
- Network è instabile
- Rate limits esistono

**Must have:**

- Try-catch specifici
- Exponential backoff
- Max retries
- Logging

---

### 4. **Output Tokens Costano 5x Input**

$3/1M input vs $15/1M output

**Implicazioni:**

- Limita `max_tokens` quando possibile
- Cache aggressivo
- Prompt compression

---

### 5. **Context Window Non È Infinito**

200k tokens sembra tanto, ma:

- Conversazione lunga = explosion costi
- Performance degrada con troppo context

**Soluzione:** Summarization progressiva

---

### 6. **Semantic Cache > Exact Match Cache**

"Come calcolare Sharpe?" ≈ "Cos'è lo Sharpe?"

**Tools:**

- SentenceTransformers per embeddings
- Cosine similarity per matching
- Threshold 0.85+ per cache hit

---

### 7. **Embeddings > Bag of Words**

BoW: perde semantica, non capisce sinonimi
Embeddings: cattura significato, funziona con variazioni

**Best model per italiano:**
`paraphrase-multilingual-MiniLM-L12-v2`

---

### 8. **Streaming Migliora UX**

User vede risposta incrementalmente:

- Perceived latency più bassa
- Può interrompere se non utile
- Feel più "naturale"

---

### 9. **Example Prompts Possono Essere Ambigui**

Single example → LLM può copiare invece di generalizzare

**Soluzione:**

- 3+ examples con valori DIVERSI
- Explicit rules
- Type annotations
- CRITICAL notes

---

### 10. **System Prompt = Personalità Permanente**

Definisci una volta, vale per tutta conversazione:

- Ruolo (analista, developer, etc.)
- Regole output
- Tone/style
- Limitazioni scope

---

### 11. **Conversazioni Lunghe = Gestione Memoria**

LLM è stateless, devi gestire tu:

- Storia completa ogni call
- Summarization quando troppo lungo
- Trade-off costi vs contesto

---

### 12. **Response Object Ha Info Preziose**

Non solo `.text`, anche:

- `usage.input_tokens` → costi
- `usage.output_tokens` → costi
- `stop_reason` → troncamenti
- `id` → tracing

**Monitora sempre in production!**

---

### 13. **Claude vs GPT: Honest vs Creative**

Claude:

- Più onesto (ammette non sapere)
- Meno hallucinations
- Migliore per reasoning

GPT:

- Più creativo
- Più integrations
- A volte troppo "confident"

**Per fintech/trading:** Claude wins

---

### 14. **API Keys = Crown Jewels**

- Mai in git
- Usa .env + gitignore
- Rotate periodicamente
- Dev keys ≠ prod keys

---

### 15. **LLM Sono Tools, Non Soluzioni**

LLM è un componente, non l'intera soluzione:

```
Good System = Python + LLM + DB + Cache + Monitoring
```

Non cadere nel trap di "LLM fa tutto".

---

# APPENDICE A: QUICK REFERENCE

## Patterns Da Ricordare

### Pattern 1: Simple Chat

```python
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{"role": "user", "content": prompt}]
)
text = response.content[0].text
```

### Pattern 2: Chat con System Prompt

```python
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    system="Sei un analista...",
    messages=[{"role": "user", "content": prompt}]
)
```

### Pattern 3: Error Handling

```python
for attempt in range(max_retries):
    try:
        response = client.messages.create(...)
        return response.content[0].text
    except RateLimitError:
        time.sleep((attempt + 1) * 2)
    except APIError as e:
        if attempt == max_retries - 1:
            raise
```

### Pattern 4: Streaming

```python
with client.messages.stream(...) as stream:
    for text in stream.text_stream:
        print(text, end='', flush=True)
```

### Pattern 5: Hybrid (Python + LLM)

```python
# 1. Calcola (Python)
metrics = calculate_with_python(data)

# 2. Valuta (LLM)
assessment = ask_llm(metrics)

# 3. Combina
return {"metrics": metrics, "assessment": assessment}
```

---

# APPENDICE B: Comandi Utili

## Setup

```bash
pip install anthropic python-dotenv sentence-transformers
```

## .env File

```bash
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

## .gitignore

```bash
.env
__pycache__/
*.pyc
```

## Test Quick

```bash
python test_claude.py
```

---

# CONCLUSIONE

Questa è stata la **Chat 1: LLM Fundamentals + API Setup**.

Hai imparato:

- ✅ Come funzionano gli LLM (no bullshit)
- ✅ Setup API pratico
- ✅ Pattern production-ready
- ✅ Prompt engineering essenziale
- ✅ Quando usare LLM vs Python
- ✅ Error handling, caching, streaming
- ✅ Le lezioni più importanti (hybrid approach!)

**Next Steps:**

**Chat 2: LangChain Basics**

- Architettura LangChain
- Chains, Memory, Prompts
- Primo mini-progetto (chatbot con memoria)

**Chat 3-4: RAG Systems**

- Cos'è RAG e perché serve
- Embeddings + Vector DB
- Progetto: Financial Document Q&A

**Chat 5+: AI Agents**

- Agent orchestration
- Tool use
- Progetto: Trading Analysis Agent

---

**Keep this book.** Tornerai a consultarlo spesso.

Ogni volta che hai dubbi:

- "Come gestisco errori API?" → Capitolo 4.4
- "Perché LLM sbaglia calcoli?" → Capitolo 11
- "Come faccio semantic cache?" → Capitolo 8
- "Quale modello embeddings uso?" → Capitolo 8.2.3

**Ale + Claude**  
*Dicembre 2024*

---

*"Il miglior modo per imparare è fare. Il secondo miglior modo è leggere un libro scritto da qualcuno che ha fatto."*

*Questo libro è entrambi.*
