# Appendix — Session Excerpts

This appendix collects a selection of real exchanges from Claude Code sessions during ARIA's development. The excerpts were chosen because they're representative of the method described in the book. The tool's responses have been shortened where necessary for space, but the prompts are reproduced in their original form. In a few places the language has been toned down from the original — anyone who works under pressure with a terminal open knows how these things go.

---

## The plan as prompt

*Backend — Cycle analysis implementation (March 7, 2026, session cost: $1.87)*

This session shows the extreme case of the "plan as prompt": a complete technical document handed to Claude Code as an implementation instruction. The prompt was written before the session, not during it.

> **👤** Implement the following plan:
>
> **\# Plan: Cycle Analysis Implementation (CICLI)**
>
> **Context:** `bot/analysis/cicli.py` is currently a stub that always returns a work-in-progress message. The Millard channel math is already fully implemented in `bot/analysis/millard_channels.py` and exposed via `analyze(prices) -> MillardAnalysis`. The plan is to connect these two parts to produce the structured JSON that the engine and responder expect.
>
> **Modified files:**
> - `bot/analysis/cicli.py` — Full rewrite (replaces the stub)
> - `llm/responder.py` — Add "cicli" entry in `ANALYSIS_INSTRUCTIONS_REGISTRY`
>
> **Helper functions:**
> - `_estimate_duration(turning_points)`: Filter minima only. 3+ minima → average of last two durations. 2 minima → single available duration. <2 minima → None.
> - `_current_cycle_state(turning_points)`: Returns (start_date, phase). Last TP kind=='min' → ascending phase. Last TP kind=='max' → descending phase.
> - `_performance(prices, start_date)`: (current_price - start_price) / start_price * 100, rounded to 2 decimals.
>
> **Note on error dict:** must have 'error', 'instrument', 'message' because the engine does `block['instrument']` and `block['message']` on the error path (lines 707-708 engine.py).
>
> [followed by: complete data architecture, spec vs code discrepancy table, edge cases to test]

**🤖** Done. Here's a summary of what was implemented: [...]

The plan contained constants, function signatures, pseudocode logic, JSON output structure, complete data flow, pre-resolved discrepancies, and edge cases. Claude Code implemented it in two minutes with one minor slip — an unused import.

---

## The diagnostic constraint: "Don't touch the code. Think Hard!"

*Frontend — Cycle table data structure migration (March 11, 2026, session cost: $2.98)*

This excerpt shows the constraint + extended reasoning combination that precedes every significant implementation.

> **👤** A change was made in the backend regarding the data passed to build the table with cycle information.
>
> [followed by: new JSON structure field by field, table of differences from the previous version, explanation of optional and mutually exclusive fields]
>
> I'm asking you to:
> - analyze the new data passed compared to the current ones to evaluate which make sense to use in our table: some could be displayed, others could activate colors or icons to make the context clearer
> - since the logic for passing durations has changed (they're now dynamic), evaluate the impact of adapting to this new data structure
> - check the impact points in the code and update the mocks to this new structure so tests can be run
>
> Don't touch the code. Think Hard!

**🤖** I have everything I need. Full analysis without touching the code.

[followed by: old → new field mapping, field-by-field evaluation with UI impact, impact point table in the code with line numbers, design proposals for ambiguous cases]

The resulting analysis was two pages and identified six impact points in the Vue component, including a potential crash on `.toFixed()` with an optional field. All before touching a single line of code.

---

## The challenge: contesting wrong proposals

*Backend — Cycle naming (March 7, 2026)*

This exchange shows how domain expertise is used to correct technically reasonable but contextually wrong proposals.

> **👤** The name to give cycles. "Intermediate cycle" and "long cycle" says nothing and isn't compatible with cycle names: what I'd like is a more descriptive name based on duration, like monthly cycle, two-month cycle, semi-annual cycle.

**🤖** Good idea. [...] A bucketing function: [proposed a fixed-range system with thresholds like "< 14 days → weekly, < 45 days → monthly, < 120 days → quarterly"]

> **👤** Your proposal makes no sense. For cycles there's a specific nomenclature — rather than that range table, it's better to use the cycle duration and say "24-day cycle", "2-day cycle".

**🤖** You're right, it was unnecessary complexity. Using the measured duration directly is more honest, more precise, and introduces no boundary ambiguity.

The tool was proposing an elegant but fragile solution — fixed thresholds that would have produced names changing from one day to the next when duration oscillated on a range boundary. The right solution was the simplest one.

---

## The iteration: multiple rounds to convergence

*Backend — Duration unit of measure (March 7, 2026)*

Same dialogue, next phase. Three rounds of correction before converging on the correct solution.

**🤖** [proposes using bars as the duration unit]

> **👤** You think bars are only days?

**🤖** You're absolutely right. Duration should be expressed in real time units, not bars.

> **👤** Ok, proceed. Note that if you change `_estimate_duration` you also need to adapt `build_cycle_json` in `@bot/analysis/cicli.py` which populates the 'estimated_duration_days' key. In my opinion instead of 'estimated_duration_days' we should pass an object with two keys, one for the unit and one for the value — what do you think?

**🤖** Agreed, much cleaner. [...] Key point: I'd use the same unit for all fields of the same cycle — if the estimated duration is in months, elapsed and to_close also use months.

> **👤** ABSOLUTELY NOT! A cycle that lasts 1 month might be missing 1 day, and you give me 0.something months?

**🤖** You're right. Each field uses the most natural unit for its value.

The tool was wrong three times: first on bars, then on timeframe, then on unit homogeneity. In none of the three cases was the error a programming error — it was a domain understanding error.

---

## The cross-language comparison: Java as reference

*Backend — Channel algorithm verification (March 9, 2026, session cost: $2.96)*

This prompt shows how existing code in another language is used as a correctness reference.

> **👤** I'd like you to verify that this module which calculates the channels does so in a way that produces THE SAME RESULT as the Java files in `@temp/`. In the end, as in the Java, I need to end up with a dataset containing the actual channel with the projected part using the polynomial.

**🤖** I've analyzed both implementations in detail.
>
> **Algorithm: correct ✓.** The core components faithfully replicate the Java: cChannel → `_ChannelState`, identical period1/period2 periods, causal double WMA ✓. Drain: `last_val += vel + acc` — identical to Java ✓. Polynomial: Lagrange on 3 points = least-squares Commons Math on 3 points (same unique solution) ✓.
>
> **Main issue: the combined dataset does NOT exist.** In Java, the final result is a single array `v` — after `forecastChannel()`, the last `forecastPeriod` values are replaced in-place with the polynomial. In Python, `ChannelResult` keeps `centered` and `forecast_values` separate.

The prompt was two lines. The response produced a component-by-component algebraic verification and identified an architectural gap — the missing combined dataset — which was the real problem.

---

## Reasoned delegation: asking for proposals on the frontend

*Frontend — User tier indicator (March 15, 2026, session cost: $2.63)*

This excerpt shows the frontend working style: give an initial idea but ask for alternatives, with the diagnostic constraint active.

> **👤** Hi! I'd like to highlight the type of subscription the user has (tiers are: Free, Pro, Max). Just the color associated with the tier is fine, we don't necessarily need to show the tier name. Since the interface is for both desktop and mobile and each platform has its own characteristics, I'd like an idea of how you'd add this information.
> One idea I had for desktop would be to add a thin border in the tier color on the user button.
> Look at the code, give me some proposals and let's think about it together; don't touch the code for now, let's just discuss!
> Think Hard!

**🤖** Your "border on the button" idea makes conceptual sense, but the `user-menu-button` is wide — a border around the whole rectangle on the sidebar's black background risks looking rough or like a focus state. Better to target the avatar. Proposal: colored ring on the avatar. A `ring-2` in the tier color around the profile photo. [...]

> **👤** For desktop I really like your ring idea on the avatar, it's brilliant.

On the frontend, the method is the same — constraint, diagnosis, discussion — but the content is different: you're not challenging algorithmic details, you're evaluating design proposals.

---

## The verification that saves: catching the bug before production

*Frontend — User tier, follow-up session (March 15, 2026)*

After agreeing on the design, Claude Code implements. But verification reveals a problem.

> **👤** Check CAREFULLY because in my opinion the store with the tier information gets populated when you access the Settings page.

**🤖** Confirmed: `updateUserPlan` is only called inside `AccountUsageSection.vue` at its `mounted()`. That component only exists in `Settings.vue`. So if the user never opens settings, `userPlan` stays at the default "Free" and the ring will always be green — wrong.

> **👤** You're making a mess, do a rollback of the changes you made, I'll make sure to return the tier in getUserInfo.

The instinct was right: the store wasn't being populated at the right moment. The bug would have reached production with all users showing the Free tier's green ring, regardless of their actual subscription.

---

## The bridge between projects: documentation as contract

*Backend — STT spec generation (March 23, 2026)*

> **👤** Write a spec document on how the STT service works; the document will be used by the frontend to integrate the services and implement the STT feature in ARIA. Put the document with the others in the `@docs/` folder.

**🤖** Document created in `docs/STT_API.md`. Covers: audio requirements with ready-to-use Float32→Int16 conversion code, complete client→server and server→client protocol, WebSocket close codes, STT quotas by tier, three sequence diagrams, JavaScript snippets, implementation notes.

*Frontend — Consuming the STT spec (March 23, 2026, session cost: $5.68)*

> **👤** Hi! On the backend I've implemented the speech-to-text feature `@docs/STT_API.md`. I'd like to integrate it into the chatbot professionally; as you know we need to think about both desktop and mobile integration. Read the documentation and give me a proposal for how you'd integrate this new feature. Until we have clear ideas don't touch the code. Think Hard!

**🤖** [produced a complete proposal: button placement, interface states, handling of transcribed text, composable architecture, error handling — all derived from the spec written by the backend instance]

Two Claude Code instances, two separate projects. One document acts as the contract between them.

---

## The revert: knowing when to go back

*Backend — Ticker download management (March 8, 2026, session cost: $4.47)*

After several rounds of modifications to two files, complexity was growing without converging on a clean solution.

> **👤** Revert EVERYTHING, bring the code back to the start of the chat.

**🤖** Reading the current state of both files first to make precise reverts. [...] Clean. Both files are back to their initial state at the start of the chat.

No hesitation. When the direction is wrong, go back.

---

## Architecture under discussion: don't accept the first proposal

*Backend — User profile management (March 16, 2026, session cost: $0.75)*

This exchange shows an architectural discussion where the author proposes a different solution from the tool's, justifies it, and reaches a synthesis.

**🤖** [proposed Option 2: modify `_get_user_limits` to accept optional parameters]

> **👤** I don't like option 2 because you have to change the signature of too many methods to add what for those methods is "noise." I lean toward a different solution that I know you won't like: `_get_user_limits` stays as it is, its job is to make sure user limits can be handled for that user, which has a direct impact on costs — important. I'd create an `ensure_user_profile` function with the COMPLETE creation logic.
> The end user will only have access to ARIA's APIs, so the probability of `_get_user_limits` being the one to create the user is close to zero, but since limits matter I keep the safety net.
> Tell me what you think (I know you'd grimace).
> Don't touch the code!

**🤖** You're right and I'm not grimacing too much. My initial instinct was "logic duplication = maintenance risk." But looking at it properly, the duplication is minimal and acceptable. [...]

> **👤** Ok, but do a refactor to move the document creation logic into a separate function so we duplicate code as little as possible. Can you make a plan to proceed? Think Hard!

The final solution was neither the tool's original proposal nor the author's in its original form — it was a synthesis taking the best of both: the separation of responsibilities the author wanted, with the refactoring to minimize duplication the tool suggested.

---

## The recurring words

To close, the raw word counts from the most-used phrases in prompts, extracted from analysis of all forty sessions.

**Backend (19 sessions):** "don't touch" — 16 times. "Think Hard" — 13 times. "implement" — 11 times. "check" — 8 times. "verify" — 7 times.

**Frontend (21 sessions):** "don't touch" — 17 times. "Think Hard" — 17 times. "implement" — 13 times. "works" — 14 times. "modify" — 18 times.

The vocabulary tells the method: first brake, then reason, then act, then verify.
