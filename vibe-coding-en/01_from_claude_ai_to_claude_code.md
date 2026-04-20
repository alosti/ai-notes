# 1. From Claude.ai to Claude Code

## The copy-paste that worked

Toward the end of 2024, my workflow with an LLM was simple and brutal: I'd open Claude.ai in one browser tab, open my project editor in another, and act as a human relay between the two. I'd select a block of code, paste it into the chat, describe the problem or the change I wanted, read the response, and carry the code back to the editor. Sometimes it worked on the first try, more often it took three or four iterations — corrections, clarifications, missing pieces to integrate by hand.

It was slow. It was manual. But it worked, and more importantly it was teaching me something I hadn't yet put into words: I was learning how to communicate with an LLM. Not in the abstract, tutorial-YouTube sense of "prompt engineering" — in the practical, messy, everyday sense of figuring out what I needed to say to get code that would fit into my project without breaking it.

Every time I pasted a block of code and described the context, I was doing a precision exercise. I had to decide what to include and what to cut. I had to explain not only what I wanted, but also what I didn't want touched. I had to critically read every line of the response before pasting it into the editor, because a silent error introduced by the LLM would be my error — the LLM doesn't take responsibility, and the project is mine.

Without realizing it, I was developing a method. A method built on explicit constraints, targeted context, systematic verification. All ingredients that would become central to working with Claude Code, but which were born in that artisanal period, one copy-paste session at a time.

## The limits of the browser

The problem wasn't the quality of the responses — that was already good. The problem was friction. Every interaction required manual work: copying the right code, making sure enough context was provided, carrying modifications back to the right place in the file, handling conflicts when a response touched parts of the code the LLM couldn't see. The larger the project grew, the more significant the cost of this back-and-forth became.

There were recurring, predictable errors. The LLM couldn't see the whole project, so it proposed solutions that worked in isolation but collided with dependencies, conventions, or architectural constraints that existed elsewhere in the codebase. I knew this, and I compensated by adding context by hand — "this file imports from this other one", "the function is called here with these parameters", "don't change the signature because it's used in this other place too". It worked, but it was simultaneous-interpreter work between two worlds that couldn't see each other.

The breaking point wasn't a single event. It was the accumulating feeling that I was spending more time shuttling information than thinking about the actual problem. The LLM was capable, the project was complex, and I was the bottleneck — not because I wasn't fast enough, but because the process was structurally inefficient.

## Claude Code arrives

When Claude Code arrived, I didn't have to change the way I thought. I had to change fewer things than I expected.

Claude Code works directly inside the project. It sees the files, reads them, modifies them, runs commands. What before required my manual intervention — providing context, carrying code back, making sure modifications fit the rest of the codebase — the tool could now handle on its own. The human relay was no longer needed, at least not in the mechanical copy-paste form.

The first time I used `@` to put a file into context, the first time I saw Claude Code navigate the project autonomously to understand how a function was being called, the first time it modified a file and I could review the diff without recopying anything — in those moments the transition felt almost magical. Not because the tool was doing impossible things, but because it eliminated exactly the friction that was slowing me down. Everything else — how to formulate requests, the need for clear constraints, the habit of verifying before accepting — was already there. I'd been doing it for months in the browser chat.

This is the point I want to be clear about upfront, because the rest of the book doesn't make sense without it: Claude Code didn't teach me a new method. It gave me an environment where the method I already had could work without friction. That's a significant difference, and it's why vibe coding works for people who already have a structured way of working and fails for those who expect the tool to do everything.

There's another fact that needs to be stated now, because it changes the perspective on everything I describe afterward: I didn't start from scratch. ARIA's backend wasn't born from an empty folder — it was born from code I had developed during an LLM course, also working with Claude. That code already had a structure, patterns, conventions in place. When I opened Claude Code on the project for the first time, the tool wasn't facing nothing — it was facing a project with its foundations already built, and it could build on top of them.

The same applies to the frontend. I didn't ask Claude Code to create a Vue application from zero. I started from a starter project of mine — a base template I use as the starting point for frontend projects, with routing, folder structure, configuration, and conventions already defined. Claude Code built ARIA's interface on top of that foundation, not in a vacuum.

This is not a minor detail. It's a fundamental part of the story. Anyone who thinks they can open a terminal, type "build me a financial analysis app" and get ARIA is mistaken. The tool needs foundations to work from — and those foundations you have to build yourself, or at least have built before. Vibe coding amplifies what's already there. If there's nothing there, it amplifies nothing.

There's another fact that needs to be stated here, because it changes the perspective on everything I describe afterward: I didn't start from zero. ARIA's backend wasn't born from an empty folder — it was born from code I had developed during an LLM course, also working with Claude. That code already had a structure, patterns, conventions. When I opened Claude Code on the project for the first time, the tool wasn't faced with nothing — it was faced with a project that already had its foundations in place, and it could build on top of them.

The same goes for the frontend. I didn't ask Claude Code to create a Vue application from scratch. I started from my own starter project — a base template I use as a starting point for frontend work, with routing, folder structure, configuration and conventions already defined. Claude Code built ARIA's interface on top of that foundation, not in a vacuum.

This is not a minor detail. It's a fundamental part of the story. Anyone who thinks they can open a terminal, type "build me a financial analysis app" and get ARIA is fooling themselves. The tool needs foundations to work on — and those foundations you have to build yourself, or at least have built before. Vibe coding amplifies what's there. If there's nothing, it amplifies nothing.

There's another fact that needs to be said here, because it changes the perspective on everything that follows: I didn't start from zero. ARIA's backend wasn't born from an empty folder — it was born from code I had developed during an LLM course, also working with Claude. That code already had a structure, patterns, conventions. When I opened Claude Code on the project for the first time, the tool didn't find nothing — it found a project with its foundations already in place, and it was able to build on top of them.

The same is true for the frontend. I didn't ask Claude Code to create a Vue application from scratch. I started from an existing starter project of mine — a base template I use as a starting point for frontend projects, with routing, folder structure, configuration, and conventions already defined. Claude Code built ARIA's interface on top of that foundation, not in a vacuum.

This is not a minor detail. It's a fundamental piece of the story. Anyone who thinks they can open a terminal, type "build me a financial analysis app" and get ARIA is fooling themselves. The tool needs foundations to work from — and those foundations have to be built by you, or at least have been built before. Vibe coding amplifies what's already there. If there's nothing, it amplifies nothing.

## What actually changed

To be precise, three things changed when I moved to Claude Code.

The first is **feedback loop speed**. Before: copy the code, write the prompt, read the response, copy the result, verify, find a problem, start over. After: write the prompt, Claude Code modifies the file, verify, find a problem, describe the problem, Claude Code fixes it. The number of steps was cut in half, and the mechanical work of transporting code disappeared entirely.

The second is **context quality**. When I worked with Claude.ai, the context was whatever I decided to paste — and it was often incomplete, because I couldn't put the entire project into a chat. With Claude Code the context is the project itself. If a function is used in three different files, Claude Code can see all of them. This dramatically reduced errors caused by missing context — the most annoying kind, because they produce code that looks correct but breaks the moment you integrate it.

The third is **the ability to delegate complex operations**. With Claude.ai I could ask for a function, at most an entire file. With Claude Code I can ask for a feature that touches five different files, with the assurance that the changes will be consistent across all of them. This changed the scale of what I could delegate in a single session.

## No longer a developer

What I didn't expect is that my role would change. Not in the generic sense of "do less manual work" — in the sense that using Claude Code I stopped feeling like a developer and started feeling like a team lead. Someone with a single team member who is extremely fast and capable, but who needs precise direction to go in the right direction.

This shift in perspective had one concrete, unexpected consequence: I learned to write. Not in the literary sense — in the operational sense. I learned to avoid words that leave room for interpretation, to be surgical in what I ask for, to break requests into atomic, verifiable tasks. Because an LLM, unlike a human colleague, doesn't nod along pretending to understand. It doesn't interpret your intentions, it doesn't read between the lines. It does what you write — and if what you write is ambiguous, the result will be ambiguous.

Anyone who's been in a team meeting knows how it actually works: someone gives a rough explanation of what's needed, everyone nods, and then each person goes off to implement their own interpretation of what they heard. The classic "you got that I need it done this way, right?" to which everyone says yes — only to discover at delivery that everyone understood something different. With an LLM this doesn't work. If you write "I need that thing there that should work like this", you'll get exactly "that thing there" — which is almost certainly not what you had in mind.

This experience convinced me of one thing: vibe coding is not a tool for beginners. It requires seniority. It requires having worked on real projects, having seen code break in production, having learned firsthand the difference between software that works in a demo and software that holds up under load. I won't put an exact number on the years of experience required, but you need to have digested enough technologies and enough real problems to know what to ask for, how to ask for it, and above all to recognize when the answer is wrong.

I once heard an anecdote — maybe true, maybe embellished, I have no way to verify it. The story goes that for a software project commissioned by a large IT company, instead of sending programmers straight to write code, they had them work inside the client's company for a few months first — on the production lines, in the warehouses, in the offices. The idea was that they had to experience the processes firsthand, and only then start developing the software. Whether that's exactly how it happened or not, the principle is hard to argue with: if you don't understand the processes, the software you write will go somewhere other than where it needs to go.

The same principle applies to vibe coding. Claude Code is the fast, capable programmer. You are the person who knows the processes, the architecture, the domain constraints, the reasons behind every decision. Without that knowledge, it doesn't matter how powerful the tool is — the result will go in the wrong direction, and you won't be able to tell until it's too late.
