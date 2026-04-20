# 4. Backend vs Frontend: Same Method, Different Levers

## Two projects, one person

ARIA is made up of two separate projects. The backend — the Python stack with FastAPI, MongoDB, the financial analysis engine, LLM integration — is the territory I know best. I've been working on it for years, I know the patterns, the pitfalls, the reasons behind every architectural choice. The frontend — Vue 3, Tailwind, JavaScript, the user interface the trader interacts with — is the territory where I'm less solid. I can read the code, understand what it does, but I don't have the same depth of experience I have on the backend.

This difference in competence produced two very different ways of using Claude Code, while maintaining the same base method. The session data tells the story with precision.

## The backend: the architect who dictates

On the backend, nineteen sessions, fifty-three dollars in cost. The dominant pattern is clear: I arrive with the problem already analyzed and often with the solution already designed.

The channel verification session is the most representative example. The opening prompt didn't ask "implement the channels" — it asked "verify that this module produces the same result as the Java files in this folder." I had the reference code in Java, I knew exactly what the output should be, and I used Claude Code as a cross-verification tool between two implementations. When the analysis showed that the algorithm was correct but the combined dataset was missing, I could immediately evaluate the proposed fix because I knew what "centered channel" vs "uncentered channel" meant in the context of cycle analysis.

In that same session, when it came time to choose a name for the variable — `full`, `channel`, `midline`, `values` — my answer was immediate. Not because I know naming conventions better than Claude Code, but because I knew how that variable would be used in the rest of the system. `ch51.channel` reads well. `ch51.full` doesn't.

This dynamic repeats across all backend sessions. I bring the data — numerical outputs, stack traces, comparisons with existing implementations — and I challenge the tool's conclusions until they're verified. I don't accept an "algorithm correct" without having compared it myself against the reference. I don't accept a refactoring proposal without having evaluated the impact on the function signatures I know.

The result is that on the backend the diagnosis phase is long and contentious, but implementation is fast. By the time we reach "ok, implement", the decisions are already made and the code that comes out is almost always correct on the first try.

## The frontend: the director who evaluates

On the frontend, twenty-one sessions, eighty-one dollars in cost. The higher cost is not a coincidence — it reflects a different way of working, with more iterations, more attempts, more code generated and revised.

The difference shows up in how sessions open. On the backend, eight out of nineteen sessions start with precise technical context — specific files, numerical data, algorithmic comparisons. On the frontend, eight out of twenty-one sessions start with a direct implementation request. Not because I'm less rigorous, but because on certain frontend aspects I don't have the depth to arrive with a solution already designed.

The user tier indicator session is revealing. The initial prompt contained an idea — "a thin colored border matching the tier color on the user button" — but it was a vague idea, and I knew it. I explicitly asked Claude Code to propose an alternative, something I almost never do on the backend. The tool analyzed the code, assessed that the border on the button risked looking like a focus state, and proposed the colored ring on the avatar instead. The proposal was better than my initial idea, and I acknowledged it.

But the method didn't change. "Don't touch the code, let's just discuss it. Think Hard!" — the constraint-diagnosis pattern was identical to the backend. What changed was the content of the diagnosis: on the backend I challenge technical details because I know them, on the frontend I evaluate design proposals because I don't have a solution ready.

## Where I move the control

The real difference isn't how much trust I give the tool — it's where I put it.

On the backend, my control is on the solution. I arrive with the plan, the function signatures, the edge cases. I control the input to the implementation phase.

On the frontend, my control is on the verification. I let Claude Code propose the solution, but then I examine it through the eyes of someone who knows the product and the end user. When in the cycles session Claude Code proposed separating the "Elapsed" and "Estimated" columns, I didn't have a technical opinion on the CSS choice — but I knew that on mobile every extra column is a problem, and I asked whether a responsive wrap could solve it. When it implemented the card layout for mobile, I looked at the result and asked for specific changes: the completion bar too short, the cycles not visually separated enough, the last border not closing the container.

These aren't technical interventions — they're user interventions. I look at the result through the eyes of the trader who'll use that interface and say what doesn't work. I don't always know how to fix it at the code level, but I know what's wrong.

It's a different form of control, not an inferior one. On the backend I control the how. On the frontend I control the what.

## The numbers speak

The data confirms this picture with a precision I didn't expect.

"Don't touch" appears sixteen times in the backend and seventeen in the frontend — nearly identical frequency. The constraint is constant, regardless of competence. This means that even in the territory where I feel less confident, I don't give up the separate diagnosis phase. The temptation to say "you handle it" is there, but the data says I don't give in.

"Think Hard" appears thirteen times in the backend and seventeen in the frontend. I use it more on the frontend — which makes sense: when I don't have a solution ready, I need the tool to reason deeply before acting.

"Implement" appears eleven times in the backend and thirteen in the frontend. Similar frequency, but with a different meaning. On the backend, the authorization to implement comes after a long diagnosis and a detailed spec. On the frontend, it comes after a shorter discussion but with a longer post-implementation iteration — visual adjustments, layout corrections, interaction details.

The files touched tell another story. On the backend, the most worked files are specific Python modules — `cicli.py`, `engine.py`, `settings.py` — each with a clear responsibility. On the frontend, the most touched files are the localization files — `it.json` and `en.json` appear in thirteen out of twenty-one sessions. Every interface change requires updating translated strings, which means the real cost of every frontend feature includes a systematic overhead that the backend doesn't have.

## The delegation trap

There's a specific risk in working on a domain where you have less expertise, and it would be dishonest not to name it.

In the tier indicator session, after agreeing on the avatar ring, I let Claude Code implement everything — store, component, style. The result looked like it was working. But when I checked more carefully, I found that the store wasn't being populated at the right time: the `userPlan` field stayed at the default value "Free" until the user opened the Settings page. The ring would have been green for everyone, regardless of their actual tier.

If I'd had the same depth of knowledge on the frontend that I have on the backend, I'd have anticipated the problem before implementation. I'd have known that the store wasn't persistent and that the data didn't come from the login service. Instead I discovered it afterward, while checking the result — which is still the method working, but with a higher iteration cost.

The solution was pragmatic: I asked for a full rollback, went into the backend to add the `plan` field to the `getUserInfo` service response, and then came back to the frontend with the data available from the right place. A detour that a frontend expert would have avoided, but one that the method still caught before it became a production bug.

The lesson is that the method — constraint, diagnosis, verification — works even when your competence is lower, but the feedback cycle is longer. You discover errors in the verification phase instead of the diagnosis phase, which means more iterations, more time, more cost. The numbers confirm it: eighty-one dollars for twenty-one frontend sessions versus fifty-three dollars for nineteen backend sessions. The difference is not accidental.

## Two modes, one principle

If I had to summarize the difference in one sentence, I'd say this: on the backend I'm the architect who designs and delegates the construction; on the frontend I'm the client who knows what they want, evaluates proposals, and requests changes until the result is convincing.

Both modes work because the underlying principle is the same: don't delegate the decisions. On the backend I make the decisions before implementation, because I have the expertise to do so. On the frontend I make them during and after implementation, because I can only recognize them when I see them realized. But in neither case do I let the tool decide for me.

This is the picture when the method works. But it doesn't always work.
