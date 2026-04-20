# Preface

## Who's writing this

My name is Alessandro Osti. I've been a programmer for 25 years — not the conference-stage type, but the kind who writes code, ships it to production, and then lives with it. I work as a backend consultant in the banking and insurance sector, mainly on Java/Oracle stacks, and in recent years I've moved to Python for my personal projects.

The project I talk about in this book is called ARIA. It's a financial analysis system — a chatbot that performs cycle analysis, calculates channels, queries LLM models to generate responses, and serves everything through a web interface. The backend is in Python with FastAPI, the frontend in Vue 3 with Tailwind. This is not an academic exercise — it's running in production on maotrade.it, my personal trading platform that I use every day.

MAOTrade has been around since 2012. I built it together with my father, a bank director with a passion for cycle analysis. ARIA grew out of that foundation.

ARIA was developed largely with Claude Code. This book is about how.

I'm not an AI evangelist, I don't sell courses, I don't have a YouTube channel. I'm a developer who studies AI as a tool — first through a structured course on LLM fundamentals, then by building ARIA as a real-world case. I found a way of working that works, and I decided to document it starting from the data: real sessions, real prompts, real costs, real mistakes. Including the times things went wrong.

## What you'll find in this book

**Chapter 1 — From Claude.ai to Claude Code.** How I worked before Claude Code, copy-pasting code back and forth between a browser tab and my editor. What changed when the tool arrived, and what — surprisingly — didn't. Why my role shifted from developer to team lead. And a fact that needs to be said clearly: I didn't start from zero — neither on the backend nor on the frontend — and pretending otherwise would distort the whole story.

**Chapter 2 — The Fundamentals.** The basic building blocks from Anthropic's course on DeepLearning.ai: `@` for giving context, "Think Hard" to force deeper reasoning, CLAUDE.md to give the project persistent memory. What they are, how they work, and why they're not enough on their own.

**Chapter 3 — The Method.** The five phases that emerged from daily work — context, constraint, diagnosis, implementation, verification — illustrated with real examples from actual sessions. How to write an implementation plan that doubles as a prompt. How to challenge wrong answers. How to use documentation to bridge two projects that live in separate repositories.

**Chapter 4 — Backend vs Frontend: same method, different levers.** How the way you use Claude Code changes when your technical depth differs. On the backend I act as the architect. On the frontend I act as the client. The session data confirms the difference — and shows what delegation actually costs.

**Chapter 5 — When the method breaks down.** The sessions where things went wrong. A deployment that wouldn't start, documentation that didn't match the code. What happens when you let your guard down, and why the method matters most precisely when you feel least confident.

**Chapter 6 — The numbers and the conclusions.** Forty sessions, a hundred and thirty dollars, one system in production. What the data says about the reality of vibe coding, what it doesn't say, and why competence remains the ingredient that separates a product with direction from one that's just empty.

**Appendix — Session excerpts.** A selection of real exchanges from Claude Code sessions during ARIA's development. Real prompts, real responses, real mistakes. Including the architectural discussion where the tool's proposal and mine converged into something neither of us had thought of alone.
