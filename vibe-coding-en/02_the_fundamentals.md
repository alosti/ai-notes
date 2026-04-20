# 2. The Fundamentals

## A four-hour course

Before touching Claude Code on a real project, I did something many people skip: I took a course. It was Anthropic's official course on DeepLearning.ai, dedicated to Claude Code. It wasn't long — you could finish it in a few hours — and it didn't promise to turn anyone into an expert. What it did, and did well, was show the basic tools and how to use them.

For someone with my development background, the course contained nothing revolutionary. It didn't teach software design, it didn't cover architecture, it didn't tackle complex problems. What it gave me was a set of practical mechanisms — the building blocks, so to speak — that I needed to get started without wasting time discovering them through trial and error. Knowing they exist and how they work is different from knowing how to combine them effectively, but without knowing them you can't even get off the ground.

I think it's right to acknowledge this debt before telling any other part of the story. The method I describe in this book is mine, but the building blocks it's constructed from came largely from that course.

## The @ — giving the tool eyes

The first practical mechanism I learned is the simplest: `@` followed by a filename. In Claude Code, writing `@filename` in a prompt means "put this file into the conversation context." The tool reads it, keeps it in mind, and uses it as a reference for everything that follows.

It sounds trivial, and in a way it is. But the difference from my old way of working with Claude.ai was enormous. Before, I had to manually copy the relevant code snippets and paste them into the chat, deciding myself what to include and what to cut. Now I could reference an entire file with a single character, and the tool had access to the complete content — no missing pieces, no partial context, no selection errors.

In practice, I quickly learned that `@` isn't just a way to provide information — it's a way to establish constraints. If a prompt says `@engine.py` and then "modify the error handling without changing the function signatures", Claude Code has the exact code in front of it. It doesn't have to guess, doesn't have to ask for clarification, doesn't have to invent a context it doesn't have. It sees the file, sees the signatures, sees the structure. The constraint becomes verifiable because the tool has everything it needs to respect it.

Over time I developed a specific habit: before writing any request, I stop and think about which files belong in the context. Not all of them — only those relevant to the specific request. Putting too many files in the context doesn't help; it dilutes the tool's attention and increases the chance it touches things it shouldn't. Context selection is a design act, not an automatic reflex.

## Think Hard — forcing the reasoning

The second mechanism I learned from the course is using "Think Hard" in prompts. In Claude Code, adding this phrase activates extended reasoning: the tool takes more time to analyze the problem before responding, explores more possibilities, and produces more considered answers.

The practical effect is visible. Without "Think Hard", Claude Code tends to jump straight to a solution — often good, but sometimes shallow, especially on complex problems where the first idea isn't necessarily the best. With "Think Hard", the tool slows down, considers more angles, and arrives at responses that account for more constraints simultaneously.

But the real value of "Think Hard" isn't something I learned from the course — I learned it by using it in combination with another constraint. In my working sessions, "Think Hard" almost always appears together with "don't touch the code." The two instructions together produce something specific: they force Claude Code to reason deeply about a problem without acting on it. It's the equivalent of telling a colleague "think about this carefully and tell me what you think, but don't change anything until we've talked it through." This creates a diagnosis phase that's separate from the implementation phase — a distinction that is fundamental when working with LLMs, and which I'll go into in the next chapter.

From the course I learned that "Think Hard" exists and what it does. From daily work I learned when to use it and what to pair it with. The difference between those two things is the difference between knowing a tool and knowing how to use it.

## CLAUDE.md — project memory

The third mechanism is the CLAUDE.md file. It's a text file you put in the project root that Claude Code reads automatically at the start of every session. It contains instructions, conventions, constraints — everything the tool needs to know about the project before you've asked it anything.

The course showed how to generate it with the `claude /init` command, which produces an initial version based on the project structure. But the auto-generated version is just a starting point. The value of CLAUDE.md emerges when you write it yourself, by hand, with the specific rules of your project — the ones no automation can guess.

In my case, CLAUDE.md contained things like: the directory structure and the role of each folder, naming conventions, architectural patterns to respect, files that couldn't be touched without explicit authorization. These were the same things I used to repeat by hand at the start of every Claude.ai session — "remember that the engine works like this", "LLM responses go through here", "don't change the signature of this function because it's used in three other places." With CLAUDE.md, that information was always present, automatically, without me having to remember to include it every time.

The course also taught me something else useful: the global CLAUDE.md, the one you place in your home directory and which applies to all projects. There I put the rules that always apply, regardless of project — comment style, how I want errors handled, the formatting conventions I prefer. This way, every time I open Claude Code on any project, the tool starts with a base of behaviors consistent with how I work.

## The blocks and the building

These three mechanisms — `@`, "Think Hard", and CLAUDE.md — are the fundamentals. They're simple to learn, understandable in a few minutes, and anyone who takes the course or reads the documentation can start using them right away. There's no secret, no hidden trick.

But there's a substantial difference between knowing these tools and using them effectively on a real project. `@` is useful only if you know which files to include in the context and which to leave out. "Think Hard" is useful only if you know when to force the reasoning and when to let the tool act directly. CLAUDE.md is useful only if you know what rules to write in it — and to know that, you need to have worked long enough on real projects to know which rules actually matter.

The fundamentals are the building blocks. The method is the building. And the building, as we'll see in the next chapter, is a different story entirely.
