# 6. The Numbers and the Conclusions

## Forty sessions, one hundred and thirty dollars

Before drawing conclusions, it's worth looking at the raw numbers. Not out of metric fetishism, but because the numbers tell a story that subjective impressions alone don't tell.

The ARIA project — backend and frontend combined — was developed across forty Claude Code sessions. Nineteen for the Python backend, twenty-one for the Vue frontend. The total token cost was approximately one hundred and thirty-four dollars. One hundred and thirty dollars for a system that includes a financial analysis engine with channels and cycle analysis, an LLM integration for response generation, a streaming Speech-to-Text service over WebSocket, a rate limiting and quota system organized by tier, a complete administration console, all with a responsive frontend, localization in two languages, and production deployment with Docker and systemd.

This is not a toy project. It's a production system serving real users.

The average cost per session reveals the difference between the two projects: approximately $2.80 per session on the backend, approximately $3.90 on the frontend. The frontend costs forty percent more per session. This reflects what I described in the previous chapter: more iterations, more attempts, more cycles of visual correction and adjustment. The price of greater delegation.

## What the numbers say about the reality of vibe coding

The first thing that stands out is time. Forty sessions aren't forty work days — many sessions last under an hour, some are done in twenty minutes. The total development time is a fraction of what would have been needed to write everything by hand. I don't have an exact hour count, but my estimate is a ratio of at least one to five — what I did in one Claude Code session would have taken me a full day of manual work, possibly more.

This doesn't mean I worked less. It means my work shifted. Less time writing code, more time thinking, specifying, verifying. The time saved on writing was reinvested in the quality of decisions. This is the real dividend of vibe coding — not doing less work, but doing different work.

The second figure is the cost distribution. The most expensive sessions aren't the ones that produced the most complex code — they're the ones where the dialogue was longest. The most expensive frontend session, nearly six dollars, was the Speech-to-Text integration: a complex feature that required a long analysis phase, a detailed proposal, and then an implementation that touched composables, components, the Vite proxy, and state management. The cost reflects the thinking, not the typing.

The third figure is the sessions-to-features ratio. Some features required multiple sessions — the cycle analysis took four sessions across backend and frontend, the admin system took six. Others were resolved in a single session. There's no fixed rule, and this is an important point: vibe coding doesn't make feature cost predictable. It makes it lower on average, but the variance remains high because it depends on problem complexity, spec clarity, and the number of iterations required.

## What the numbers don't say

The numbers don't measure the value of the decisions made during the process. They don't measure the bug avoided because the diagnosis phase identified a crash on `.toFixed()` with an optional field before it reached production. They don't measure the feature designed better because I had time to think about it instead of drowning in code. They don't measure the cleaner architecture because I was able to do a full revert and start over when the direction didn't convince me.

The numbers don't say how much of the final result is credit to the tool and how much is credit to me. It's a question that has no answer, because the result is the product of the interaction between the two. Claude Code without my domain expertise would have produced working code but probably wrong in its architectural choices. Me without Claude Code would have made the same architectural choices but in much more time and probably with more implementation errors.

It's not a question of who does more. It's an asymmetric collaboration where each side contributes what it does best.

## Competence is not optional

If there's one thesis that runs through this entire book, it's this: technical competence changes where you intervene in the process, but it doesn't change the fact that you have to intervene.

On the backend, where my experience is deep, I intervene at the start — I write the spec, design the solution, and delegate the translation into code. On the frontend, where my experience is thinner, I intervene at the end — I evaluate the result, request corrections, judge with the user's eye. Outside my area of competence, on devops, the method weakened because I knew neither what to ask nor what to verify.

Vibe coding amplifies what you have. If you have experience, it amplifies it into productivity — you do more in the same time, with fewer errors, with less mechanical work. If you have product sense and taste, it amplifies it into quality — you have time to think about choices instead of chasing the code. If you have method, it amplifies it into consistency — you apply the same rigor across multiple fronts simultaneously.

But if you have nothing to put into the process, what comes out is empty. Code that works but has no direction, architecture without reason, a product that does things but doesn't solve problems. I've heard people say that AI won't replace those who have something to say — it will help them say it better and faster. Those who have nothing to say will produce anonymous work, and that work won't interest anyone because anyone will be able to get at least that result. This is exactly my experience with vibe coding.

## The role that remains

At the end of this journey, what I take away isn't a five-phase method — even though the method works and I've described it as honestly as I can. What I take away is a clarity about my role.

I'm no longer a developer in the traditional sense of the term. I don't spend my days writing code line by line, hunting missing semicolons, debugging with print statements scattered through the codebase. But I'm not a manager who just delegates either. I'm something in between — an architect who knows how to build, a team lead who can read their team's code, a conductor who knows how to play the instruments.

Claude Code didn't replace me. It freed me from mechanical work and left me with the work that matters: deciding what to build, how to build it, and why. And then verifying that what was built matches what I had in mind.

That work requires experience. It requires having seen enough projects go wrong to know where to look. It requires having written enough bad code to recognize it when you see it. It requires having spent enough time in the problem domain to know what makes sense and what doesn't.

Vibe coding is not a shortcut. It's a different way of working that rewards those who have something to bring to the table. The technology is accessible to everyone. The difference is what you put into it.
