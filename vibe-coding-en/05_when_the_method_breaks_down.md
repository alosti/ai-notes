# 5. When the Method Breaks Down

## The comfort zone has a boundary

Up to now I've described a method that works. It works on the backend where I have deep experience, it works on the frontend where I have less experience but compensate with a different form of control. The temptation would be to stop here and close the book with a reassuring message: follow these five phases and everything will be fine.

But that wouldn't be honest. The method has a precise limit, and that limit coincides with the boundary of competence. Not competence in the generic sense of "knowing how to program" — specific competence in the problem domain. When you step outside your comfort zone and enter territory you don't know well enough to challenge the tool's answers, the method weakens. And when it weakens, things go wrong.

## The Qdrant case

The most instructive moment in the entire project was deploying Qdrant to the production server. This wasn't programming — it was devops: systemd files, Docker, filesystem permissions, cgroups. Territory where I don't have the same command I have over Python or backend architecture.

The session started well. I asked Claude Code how to configure the service, received a systemd file, and deployed it. Then the disaster began.

The file contained hardening options — `ProtectSystem=strict`, `PrivateTmp=true`, `NoNewPrivileges=true` — that Claude Code had added because they're best practices for systemd services. The problem is that these options apply to the `docker run` process, not to the container, and they prevent Docker from accessing the socket and the system filesystem it needs to create the container. The service wouldn't start.

Then there were the CPU and memory limiting options — `--cpus` and `--memory` in the `docker run` command. Also reasonable in theory, but on the specific version of Docker on my server they didn't work as expected. Without them, the service started. With them, it broke.

And then there were the invisible spaces after the backslash continuation characters in the systemd file. A trivial error, almost invisible to the naked eye, that produced a cryptic "invalid reference format" pointing to no specific line.

Three overlapping problems, any one of them enough to prevent the service from running, and I didn't have the expertise to isolate them quickly. In a context I know well — a bug in the channel calculation, an error in the cycle logic — I'd have separated the causes in five minutes. Here I lost nearly an hour trying, failing, swearing, and trying again.

## What went wrong in the method

The interesting thing isn't that the service didn't start — technical problems happen. The interesting thing is how my behavior in that session was different from all the others.

I didn't use "don't touch the code." I didn't force a separate diagnosis phase before acting. I didn't write "Think Hard" before asking for the implementation. I took the systemd file Claude Code proposed and deployed it directly, trusting it was correct.

In other words, I skipped the method. I skipped the phases that had protected me from errors in every other session — the constraint, the diagnosis, the verification before action. I did it because I didn't have the knowledge to conduct that diagnosis. I didn't know what questions to ask. I didn't know what to check. And because I didn't know what I didn't know, I did the most natural and most dangerous thing: I trusted.

If I compare this session to the channels one — where I had the reference Java code, knew exactly what should come out, and challenged every single conclusion from Claude Code — the difference is obvious. On the channels I was the architect who verifies. On Qdrant I was the user who hopes.

## The documentation error

There's another episode worth telling, because it shows a different kind of failure — not of the method, but of the tool, with consequences amplified by my failure to verify.

When I asked Claude Code to update the STT documentation with the admin services, the tool did two things: it modified the code to add the missing fields to the services, and it updated the `STT_API.md` document with the new information. So far so good, apparently.

The problem is that Claude Code documented changes to the `GET /client/user/usage` service that it hadn't actually made in the code. The document said the service returned the new STT fields, but the code hadn't been touched. If I'd copied that document into the frontend folder and asked the frontend's Claude Code to implement the integration based on those specs, the frontend would have called a service expecting fields that didn't exist. A silent bug that would have surfaced only in production.

I discovered it by re-reading the document and comparing it with the code — the verification phase worked, but only because in this case I knew the code well enough to notice the discrepancy. If it had been a less familiar domain, I'd have taken the documentation at face value and the bug would have slipped through.

This episode adds an important nuance: the tool can be internally inconsistent. It can do one thing in the code and write another in the documentation. Not out of malice, but because the session context is long, the operations are multiple, and the tool has no automatic mechanism for verifying the consistency of its own actions. That mechanism is you.

## The pattern of misplaced trust

Looking at these episodes in hindsight, I see a common pattern. In both cases, the trigger wasn't a technical error — it was my decision to lower the level of control.

On the Qdrant devops, I accepted a systemd file without first asking for an analysis of the implications of each option. If I'd done what I always do — "tell me what each line of this file does, don't touch anything" — Claude Code would have explained that `ProtectSystem=strict` blocks filesystem access for the process, and I could have evaluated whether that restriction made sense for a Docker wrapper. I wouldn't have understood all the details, but I'd have had enough information to ask "are you sure these options work with Docker?" — and that question would probably have avoided the problem.

On the STT documentation, I verified the code but could have explicitly asked: "check that every change described in the document corresponds to a real change in the code." A cross-consistency request that the method fully supports but that I didn't make.

In both cases, the fix didn't require additional expertise — it required discipline. The method only works if you apply it, and the temptation to skip it is strongest precisely when you'd need it most: in the territories you don't know.

## What failure teaches

The lesson I took from these experiences isn't "learn devops" — though that wouldn't hurt. The lesson is more general and applies to anyone using vibe coding on real projects.

The method is robust as long as you maintain your role. Your role isn't to know everything — it's to know where you don't know, and use that awareness to increase control instead of decreasing it. When you enter unknown territory, the right response isn't to trust the tool more because "it knows more than I do about this." The right response is to slow down, ask for more explanations, force more diagnosis, verify more thoroughly. Do the opposite of what instinct suggests.

The tool doesn't become less reliable when you step outside your comfort zone — you become less capable of verifying its reliability. The difference is subtle but fundamental. Claude Code didn't make more mistakes on the Qdrant session than on the channels session. It made different mistakes, and I wasn't equipped to catch them in time.

This isn't a limitation of vibe coding. It's a human limitation that vibe coding amplifies. If you know little about a subject and you have a tool that produces convincing answers on that subject, the combination is dangerous — because the answer looks reasonable, you don't have the tools to contest it, and the result is that you accept something wrong with full conviction that it's right.

The solution isn't to stop using the tool outside your area of competence — that would throw away half its value. The solution is to be more methodical, not less, precisely when you feel least confident. More constraints, more diagnosis, more verification. And accept that in those cases the cycle will be slower and more expensive, because it will be.
