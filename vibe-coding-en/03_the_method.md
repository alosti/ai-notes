# 3. The Method

## Five phases, always the same

Analyzing the forty working sessions with Claude Code — nineteen on the backend and twenty-one on the frontend — a recurring pattern emerges. I didn't design it upfront; it grew on its own, session after session, until it became the natural way I approached every problem. Looking back, I can describe it as a sequence of five phases.

**Context.** Every session starts with a precise framing of the problem. Not a generic "I need to do this thing", but a specific indication of the files involved, the current situation, what works and what doesn't. The more precise the context, the less room for interpretation the tool has, and the lower the chance it heads in the wrong direction.

**Constraint.** Right after context comes the instruction on what not to do. "Don't touch the code." "Without modifying." "Analyze but don't implement." This is the step that separates conscious use of the tool from passive use. The constraint prevents Claude Code from acting before I've understood what's going on. It seems counterintuitive — you have a powerful tool and the first thing you do is stop it from doing something — but this is exactly the move that makes the difference.

**Diagnosis.** With context provided and action blocked, Claude Code is forced to think. If the prompt also contains "Think Hard", extended reasoning kicks in and the analysis it produces is more thorough. This is the phase where the tool gives me its understanding of the problem — which I then evaluate, challenge, correct. It's a dialogue, not an oracle.

**Implementation.** Only when the diagnosis is shared — when both the tool and I have an aligned understanding of the problem and the solution — does it become time to write code. At this point implementation is almost mechanical, because the important decisions were already made in the previous phase.

**Verification.** Implementation doesn't close the session. After every change, I verify the result. Sometimes verification is a test run, sometimes it's a code review, sometimes it's an explicit "check everything again." If verification fails, we go back to diagnosis — not patch blindly.

This sequence isn't rigid. In simple sessions, the diagnosis phase shrinks to a few lines. In complex sessions, the iteration between diagnosis and constraint can last longer than the implementation itself. But the underlying pattern is always the same: understand first, act second, always verify.

## "Don't touch the code"

Of all the phrases I used across the sessions, "don't touch" is the most frequent — sixteen times in the backend, seventeen in the frontend. That's not a coincidence. It's the phrase that activates the most important phase of the entire method: the separation between analysis and action.

When you're working with an LLM that has direct access to your code, the tool's natural instinct is to act immediately. You describe a problem and it starts modifying files. If the problem is simple and well-defined, this works. But if the problem is complex, involves multiple files, has non-obvious constraints or hidden dependencies, immediate action is almost always premature. The tool produces a solution that solves the problem as it understood it — which is not necessarily how you understood it.

"Don't touch the code" interrupts this automatism. It forces the tool to stay in analytical mode, to examine the problem without the urgency to solve it. And it forces me to read the analysis, evaluate it, decide whether it matches my understanding of the situation.

A concrete example comes from a frontend session where the backend had changed the data structure for the cycle table. The initial prompt was detailed: it included the new JSON structure field by field, a table with the differences from the previous version, and three specific requests — analyze the new data, assess the impact, update the mocks. But the phrase that closed the prompt was: "Don't touch the code. Think Hard!"

The result was a two-page analysis that mapped every old field to the new one, identified six impact points in the Vue component with line numbers, flagged potential crashes from missing null guards, and proposed design choices for the ambiguous cases — how to handle start_date on mobile, whether to separate or combine the Elapsed and Estimated columns, how to visually distinguish a point estimate from a range. All of this without touching a single line of code.

If I had asked directly "update the component to the new data structure", I'd have gotten an updated component. Probably working. But I wouldn't have had that impact map, those design questions that made me think about choices I hadn't considered. I wouldn't have discovered that `.toFixed()` crashed on optional fields before seeing it actually crash. The diagnostic phase, forced by the constraint, produced value that direct implementation wouldn't have produced.

## The art of the spec

There's an even deeper level of constraint that I used mainly on the backend where my technical depth was greater. Instead of giving a problem and asking for an analysis, I'd give the solution directly — in the form of a complete implementation plan.

The most representative session of this approach is the one where I implemented cycle analysis in the backend. The prompt wasn't a request — it was a two-page technical document. It contained the constants to use, the function signatures with their parameters, the logic of every helper function written in pseudocode, the JSON output structure, the data flow from the engine to the responder, a table of discrepancies between the original spec and the real code, and even the edge cases to test.

Claude Code received this plan and implemented it in two minutes. Two files modified, logic correct, one minor slip — an unused import. The session cost was under two dollars.

But that plan didn't appear from nowhere. Writing it took time — probably more time than it would have taken to implement the code directly. The difference is that I spent that time thinking, not writing code. Deciding how the duration estimate should work, defining what happens when the data isn't sufficient, verifying that the error structure was compatible with the engine that would consume it. By the time I pasted the plan into Claude Code, all the decisions had already been made. The tool just had to translate them into Python.

This is the architect's approach I mentioned in the first chapter. I wasn't writing code — I was writing a spec so precise that translating it into code became almost trivial. And if the translation is trivial, there's no point in me doing it when I have a tool that does it faster.

## Challenging the answers

Diagnosis is not an act of faith. When Claude Code produces an analysis, that analysis needs to be verified — and the only way to verify it is to challenge it with data, questions, objections.

In the session on cycle naming, the dialogue was long and contentious. Claude Code proposed an initial solution — a bucketing system with fixed ranges that mapped durations to predefined names like "monthly cycle" or "quarterly cycle." The proposal had a logic to it, but it was fragile: what happens when a cycle sits on the border of a range? Today it measures 58 days and gets called "two-month cycle", tomorrow it measures 62 and becomes "quarterly cycle." For a user who follows the analysis over time, a name that changes for no apparent reason is pure confusion.

I challenged the proposal. Claude Code reformulated, but not in the right direction — it kept reasoning in terms of ranges and thresholds. The real solution was much simpler: use the measured duration directly as the name. "24-day cycle", "90-day cycle." No ranges, no ambiguity, no boundary problems.

The tool, in this case, was overcomplicating the problem. It was looking for an elegant solution where an honest one was needed. My contribution wasn't technical — the math it could handle fine — it was domain common sense. I knew how a trader reads that data, and I knew that a name that changes from one day to the next is worse than a plain name that says exactly what it measures.

This kind of interaction is perhaps the heart of the entire method. The tool is good at writing code, finding bugs, producing structured analyses. But design decisions — the ones about what's right for the end user, what makes sense in the specific domain, what's simple and what's unnecessarily complicated — those remain human. And the only way to surface them is to not accept the first answer, but to discuss it.

## One more round

There was another step in that same session worth describing, because it shows how the challenge can go several rounds before converging.

After agreeing to use the measured duration as the name, there was still the question of how to express units of time. Claude Code proposed using bars — which was correct for daily data, where one bar equals one day, but completely wrong for any other timeframe. On weekly data, 90 bars aren't 90 days, they're 630. On intraday data, 90 bars could be 90 hours or 90 fifteen-minute periods depending on the timeframe.

I pointed out the error. Claude Code corrected course and proposed converting bars to calendar days using datetime indices — correct in principle, but the proposed version used `.days` on the timedelta, which loses fractional days. On intraday data, turning points a few hours apart would produce `.days = 0`, losing the information entirely.

Two observations on this iteration cycle. First: the tool was wrong twice in a row, in two different ways. Second: in both cases, the error wasn't a programming error — it was a domain understanding error. Claude Code didn't know the data could be intraday, and didn't know `.days` truncates fractions. I knew both, and my job at that point was to bring that knowledge into the dialogue until the solution became correct.

We ended up using `total_seconds() / 60` as the internal unit, with a conversion function that picks the most natural unit for each value. But we got there after three rounds of objections and corrections — not after the first response.

## The plan as prompt

Going back to the cycles session, it's worth looking at what that implementation plan actually contained, because it's a concrete example of how domain expertise translates into an effective prompt.

The plan didn't just say "implement cycle analysis." It contained:

**Constants** with their values and the reason for those values — the minimum of 200 bars wasn't an arbitrary number, it was calculated from the warmup required for the 153-period channel.

**Function signatures** with parameter types and return values — not "make a function that estimates duration", but `_estimate_duration(turning_points) -> Optional[float]` with specific logic for three minima, two minima, fewer than two minima.

**The error structure** compatible with downstream code — the error dict needed to have keys `error`, `instrument`, `message` because the engine did `block['instrument']` and `block['message']` at lines 707-708.

**The complete data flow** from engine to responder, with every intermediate transformation.

**Discrepancies** between the original spec and the real code, already resolved in the plan — so Claude Code wouldn't get confused finding gaps between documentation and implementation.

**Edge cases** to test, written as condition and expected result.

None of this was the LLM's work — it was mine. I knew the codebase, knew where things could break, knew which design decisions I'd made and why. The plan was the translation of that knowledge into a format Claude Code could execute without ambiguity.

This approach works particularly well when the person has strong domain expertise. You don't need to write code — you need to write a spec so precise that the code becomes its natural consequence. It's different work from programming, but no less demanding. It requires the ability to think through the whole problem before writing a single line, to anticipate breaking points, to specify edge cases.

It requires, in other words, seniority.

## "Revert EVERYTHING"

Not every session goes well. The ability to recognize when a path is wrong and go back is an integral part of the method.

In one backend session, after several rounds of modifications to two files handling ticker downloads, I decided the direction we'd taken wasn't right. The changes were accumulating, the complexity was growing, and the solution was becoming more complicated than the problem it was supposed to solve. At that point the decision was clear: "revert EVERYTHING, bring the code back to the start of the chat."

Claude Code executed. It read the state of the files, identified all the changes made during the session, and brought both files back to the starting point. Clean operation, no residue.

This is one of the underappreciated advantages of vibe coding. When you write code by hand and realize after an hour that the direction is wrong, undoing the work is painful — psychologically, because you're "throwing away" work you did, and practically, because you have to remember what you changed and how it was before. With Claude Code, the revert is a command. There's no emotional attachment to the code written, because you didn't write it. And there's no problem remembering the previous state, because the tool knows it.

The lesson is simple: if the direction is wrong, stop and go back. Don't try to salvage the work done by adapting it, don't add complexity to compensate for a bad initial choice. The cost of starting over is low — much lower than the cost of dragging a solution forward that you're not convinced by.

## The bridge between two projects

There's a technique I developed while working on ARIA that deserves its own section, because it solved a problem that anyone working on a multi-project system knows well: how to transfer knowledge between a backend and a frontend when the two live in separate repositories, and the two Claude Code instances can't talk to each other.

The problem is concrete. When I implement a new feature in the backend — a new endpoint, a change in the data structure, a communication protocol — the frontend needs to know exactly what changed in order to adapt. In a traditional team, this information travels through meetings, messages, hand-written documentation, and some piece of it often gets lost. With two Claude Code instances on two separate projects, the risk is the same: the frontend instance doesn't know what the backend instance did.

The solution I found is simple and works well: I have Claude Code write the documentation itself.

When I implemented the Speech-to-Text service in the backend, once the code was working, I asked the backend's Claude Code: "write a spec document on how the STT service works; the document will be used by the frontend to integrate the services and implement the STT feature in ARIA." The document was saved in the `docs/` folder of the project.

The result was a complete spec file — audio requirements with ready-to-use conversion code, WebSocket protocol with all client-server and server-client message types, close codes, tier quotas, sequence diagrams for the main use cases, JavaScript snippets, implementation notes on non-obvious points. The backend's Claude Code wrote this documentation with a precision I couldn't have matched by hand, because it had just written that code and knew every detail of it.

The document was then copied to the `docs/` folder of the frontend project. At that point, opening Claude Code on the frontend, the prompt became: "On the backend I've implemented the speech-to-text feature. Read the documentation and give me a proposal for how you'd integrate this new feature. Don't touch the code until we have clear ideas. Think Hard!" — with a reference to the file `@docs/STT_API.md`.

The frontend's Claude Code read the document, analyzed the existing code, and produced a detailed integration proposal — button placement, interface states, handling of transcribed text, composable architecture, error handling. All based on the spec written by the other instance.

The most powerful thing about this technique is that the document lives and evolves. When I added the STT quota fields to the admin services in the backend, I asked Claude Code to update the same document: "add the admin services section to `@docs/STT_API.md` so the frontend can handle the new fields." The document was updated, copied to the frontend, and there the prompt was: "The service changes with field names are in `@docs/STT_API.md`" — and the frontend's Claude Code knew exactly what had changed and where to intervene.

In practice, I created a communication protocol between two tool instances that can't talk to each other directly. The document acts as a contract between the two sides of the system: the backend writes it, the frontend reads it, and I act as the messenger. The nice part is that the document is written by whoever knows the code most intimately — the instance that just implemented it — and read by whoever needs that knowledge to do their job.

This isn't a technique I read somewhere. It grew from the practical necessity of keeping two projects aligned as they evolved together, and it turned out to be one of the most effective tools in the entire process.

One practical note for anyone who wants to adopt this approach: in my case, transferring documents between the two projects was a simple `cp` from one folder to another. Both projects have a `docs/` folder in their git repository, and the spec files live there alongside the code. This choice is deliberate. Keeping documentation inside the repository gives the project history and substance — documents evolve with the code, are versioned with the same tool, and anyone who clones the repository has everything needed to understand how the system works.

In a company environment with multiple teams, documentation could also live on platforms like SharePoint or Teams — which have the advantage of more accessible versioning for people who don't work with git, and immediate availability without needing to clone a repository. But my recommendation is still to copy the documents into the project folder too. The reason is simple: when Claude Code opens a session, the files in the repository are its natural context. A document in the `docs/` folder is one `@docs/STT_API.md` away. A document on SharePoint requires an extra step — downloading it, pasting it, or describing its content by hand — and every extra step is friction, and friction is exactly what this method tries to eliminate.

## The method is not a formula

I want to be clear on one point: the sequence I've described — context, constraint, diagnosis, implementation, verification — is not a formula to apply mechanically. I never opened a session thinking "now I do phase one, then phase two." I worked, and the method emerged from the work.

The reason I describe it as a method is that when analyzing the sessions in retrospect, the pattern is evident. The phases are there, the sequence repeats, the patterns are measurable — sixteen "don't touch", thirteen "Think Hard", eleven "implement", eight "check." The numbers tell a consistent story: first brake, then analyze, then act, then verify.

But the numbers also tell something else. Across forty sessions, there were sessions where the constraint wasn't needed because the problem was simple and well-defined. Sessions where diagnosis came down to one line because the cause was obvious. Sessions where implementation was immediate and verification was a test that passed first try. The method adapts to the problem, not the other way around.

What stays constant is the attitude: don't trust blindly, don't delegate decisions, don't accept the first answer just because it sounds reasonable. The tool is powerful, but the brain is mine.
