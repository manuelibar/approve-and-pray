# Nobody Owns This Code

<p align="center">
  <img src="https://github.com/user-attachments/assets/2d01f20f-aee6-46e0-a605-0f2981b711ae" alt="Nobody Owns This Code" width="100%" />
</p>

*By [Manuel Ibar](https://github.com/mibar) — March 2026*

---

## 1. Three Fridays

Let me tell you about three Fridays.

**Ana** leads the platform team at a Fortune 500 that adopted agents aggressively — same tools as everyone else, same velocity promises. Implementation is blazing fast. The problem is everything else.

Mandatory human review on every PR. Cross-review required for shared infrastructure. Branch protection: two approvals minimum. The process was designed for human-speed implementation, and nobody updated it when the implementation speed jumped 10x.

The math is killing her. Agents ship 40+ PRs a week. Ana can review maybe 20 if she does nothing else. The queue grows every single day. Her Slack is a timeline of escalating politeness: "Hey, any chance you can look at #342?" becomes "Just pinging again, no rush!" becomes "Hi Ana, the feature freeze is tomorrow."

The irony is sharp: she's the best reviewer on the team. That's why she's the bottleneck. Her competence is her curse. She doesn't approve bad code. She doesn't approve *any* code fast enough.

Ana hadn't approved a single thing she didn't understand. She also hadn't gone home before 8 PM in three weeks.

---

**Marcus** runs a team of three — himself and two engineers — backed by a fleet of agents. They ship more features per week than teams of twenty. They are, by every metric that matters to leadership, the most productive team in the company.

Andrej Karpathy [called it](https://x.com/karpathy/status/1886192184808149383) "vibe coding" in February 2025. Marcus calls it Tuesday. He sets direction, reviews outcomes: does the feature work? Does the dashboard look right? Does the customer flow make sense?

He doesn't read implementation code. Not because he's lazy — because 12,000 lines were written this week and there are three of them. He resolved the same math that's crushing Ana, just differently: instead of adding reviewers, he removed the review.

Everything works. No bottlenecks. No review queues. No Slack escalations. The dashboards are green. He might write a blog post about how to 10x engineering velocity with three engineers.

The discourse says vibe coding is dead, that you need a human in the loop, that agents need harnesses. Marcus's dashboards disagree. For now.

Marcus slept well that Friday night. The code he'd never read was working perfectly. He had no way of knowing it wouldn't be by Monday.

---

It's Friday at 4:47 PM. A PR landed an hour ago — 847 lines changed across 23 files, generated in about twelve minutes by an agent that never gets tired, never gets distracted, and never second-guesses itself. The CI is green. You've been staring at it for forty minutes. You understand maybe 60% of what it does. The product owner pinged you twice about demo readiness. The team lead is asking why the sprint board still has three stories stuck "in review."

You type "LGTM," hit merge, and feel a knot in your stomach you can't quite name.

You didn't rubber-stamp it. Rubber-stamping is lazy — it's the act of not caring. This was something more anxious and visceral. You *tried* to review it. Opened every file. Traced every function call you could follow. Cross-referenced the ticket. Checked the test assertions. And then hit the wall of your own cognitive bandwidth. You approved it anyway, because the alternative was blocking the entire team for another day, and there's a demo on Monday.

You approved and prayed.

Of the three, you're the closest to getting it right. You have the speed. You have some review. You have the instinct that something is wrong — that's what the knot is. What you don't have is a way to turn that instinct into a system. You're managing risk by prayer, and prayer is not an engineering practice.

But what if you could keep the speed, keep the review where it matters, and replace the prayer with something you can actually measure?

---

There's a module in each of these codebases that will cause their next P1. When it breaks, someone will ask: "Who owns this?"

Ana will raise her hand — along with 400 other files she's responsible for. Marcus won't even be in the room. You'll hesitate, because you approved it but you're not sure you understood it.

The numbers tell all three stories with uncomfortable precision. In 2025, [41% of code was already AI-generated](https://www.quantumrun.com/consulting/github-copilot-statistics/). By early 2026, teams using AI coding tools saw a [98% increase in PR volume with a 91% increase in review time](https://blog.logrocket.com/ai-coding-tools-shift-bottleneck-to-review/). Pull requests are [18% larger on average, and incidents per PR have climbed 24%](https://addyo.substack.com/p/code-review-in-the-age-of-ai). The [review bottleneck is already here](https://levelup.gitconnected.com/the-ai-code-review-bottleneck-is-already-here-most-teams-havent-noticed-1b75e96e6781), and most teams haven't noticed it — because for Ana the symptom looks like "being thorough," for Marcus it looks like "going fast," and for you it looks like "being pragmatic."

AI generation speed has increased roughly 10x in the last two years. Human comprehension speed has not increased at all. It can't. It's a biological constant — bounded by working memory, attention span, and the speed at which the human brain builds mental models of complex systems. We are connecting a firehose to a garden hose, and the three stories are three ways of dealing with the flood:

Ana added valves, filters, and quality controls to the garden hose. The water barely trickles. The garden is dying of thirst. But hey, no contamination.

Marcus disconnected the hose and installed sprinklers that run themselves. The garden looks lush. He has no idea what's in the water.

You opened the valve most of the way and you're watching carefully. The garden is getting watered. Some of it is flooding. You know which parts — you just don't have a way to fix it without shutting off the valve entirely.

**Every approved-and-prayed PR permanently mints new cognitive debt — whether it took four minutes or fifteen days, whether one person approved it or five, whether the review was exhaustive or absent. Ana trades system comprehension for process compliance. Marcus trades it for velocity. You trade it for pragmatism. The exchange rate is getting worse every quarter, and none of you are paying it down.**

What do we call code we don't *understand*?

---

## 2. A Brief History of Owing the Compiler Money

In 1992, Ward Cunningham stood up at [OOPSLA and introduced the debt metaphor](https://c2.com/doc/oopsla92.html) while explaining the design decisions behind the WyCash portfolio management system. The idea was simple and powerful: sometimes you ship code you know isn't ideal because the business value of shipping now outweighs the cost of cleaning up later. You're borrowing against the future. As long as you pay it back — refactor, rewrite, clean up — the debt is manageable. If you don't, the interest compounds until the codebase becomes unmaintainable.

That metaphor changed how we talk about software. [Martin Fowler](https://martinfowler.com/bliki/TechnicalDebt.html) formalized it. Entire organizations built dashboards around it. "Technical debt" entered the vocabulary of product managers, CTOs, and board members who'd never written a line of code but intuitively understood what it meant to borrow against the future. For thirty years, it was the right abstraction.

But it carried an implicit assumption that nobody questioned because it was always true: *you understood the code you were choosing to defer*.

Technical debt is a conscious trade-off. You wrote the module. You know it's messy. You know the edge cases it doesn't handle. You ship it anyway because the deadline matters more than elegance right now, and you plan to come back. The debt is intentional, localized, and — critically — understood by the person who incurred it. That understanding is what makes it manageable. You know exactly what you owe and where the bodies are buried.

The vocabulary evolves because the practice evolves. In February 2025, Andrej Karpathy coined ["vibe coding"](https://x.com/karpathy/status/1886192184808149383) — describing the experience of programming by feel with AI, barely reading the generated code, just running it until it works. It captured something real: the seductive efficiency of not needing to understand every line. Marcus would recognize this description instantly — it's not a weekend experiment for him; it's his production workflow. A year later, Karpathy [walked it back](https://thenewstack.io/vibe-coding-is-passe/): "Vibe coding is passé." The industry is converging on "harness engineering" — the idea that agents need guardrails, not freedom. Marcus hasn't read the memo yet.

And here's where Cunningham's metaphor reaches its breaking point. With agents, "refactoring later" becomes automated. An agent rewrites a legacy module overnight. Another agent picks up the tech debt it left behind. The TODO comments get resolved. The deprecated APIs get updated. The linter warnings disappear. The test coverage goes up. Technical debt drops to near zero on paper — the agents are tireless, thorough, and cheap. But the sheer volume of code implemented by this automation is staggering — and nobody reviewed any of it. The debt on the spreadsheet is gone. The understanding never existed. We've paid off the credit card by taking out a mortgage we don't fully understand the terms of.

Technical debt used to cause parallelization problems in a familiar way: accumulated cruft slowed everyone down. You couldn't add a feature without untangling the mess. Now we have a *worse* parallelization problem. During incidents, security audits, or onboarding, multiple teams hit the same wall at the same time — nobody understands the code. The cruft is gone. The system is clean, well-tested, beautifully formatted, and completely opaque to every human being responsible for it.

**We are trading debt we controlled — technical debt — for debt that accrues globally and lacks accountability.**

You'd recognize this trade-off from 4:47 PM last Friday. You're doing what Cunningham described — shipping now, planning to come back later. But the code you're deferring isn't messy code you wrote and understood. It's clean code you never understood in the first place. Cunningham's metaphor assumed comprehension. That assumption just broke.

---

## 3. Naming the Ghosts

Remember those three Fridays? Each team was dealing with the same explosion of AI-generated code. Each chose a different strategy. Let's name what's happening to each of them.

In June 2025, a team at the MIT Media Lab published a study titled ["Your Brain on ChatGPT"](https://www.media.mit.edu/publications/your-brain-on-chatgpt/) (Kosmyna et al., arXiv:2506.08872). They used functional near-infrared spectroscopy — essentially, they watched people's brains work — to measure brain connectivity in people completing cognitive tasks with and without LLM assistance. The finding was striking: participants who relied on LLMs showed the *weakest* brain connectivity patterns. They couldn't accurately recall or explain their own work. The researchers called this phenomenon *cognitive debt* — not as a software metaphor, but as a measurable neurological outcome. The tools that made us faster were also making our brains less engaged with the output. The debt wasn't metaphorical. It was happening in the prefrontal cortex.

In February 2026, [Margaret-Anne Storey](https://margaretstorey.com/blog/2026/02/09/cognitive-debt/) took the term and extrapolated it to software engineering. Drawing on Peter Naur's seminal insight that "a program is a theory" held in the programmer's mind — that the true nature of a software system lives not in the source code but in the mental model of the people who built it — and echoing Fred Brooks and [Kent Beck](https://tidyfirst.substack.com/p/augmented-coding-beyond-the-vibes), she argued that the real debt isn't in the code. It's in the developers' heads. When the human mental model of a system degrades, the system becomes ungovernable regardless of how clean the code looks. You can have zero technical debt and still be unable to operate your own software.

What happened next is telling: [at least five independent groups](https://byteiota.com/cognitive-debt-ai-coding-agents-outpace-comprehension-5-7x/) converged on the same concept within the same quarter. Different names, same pain. [CodeRabbit](https://www.coderabbit.ai/blog/2025-was-the-year-of-ai-speed-2026-will-be-the-year-of-ai-quality/) called 2025 "the year of AI speed" and 2026 "the year of AI quality." Others called it the "comprehension gap" or the "velocity-understanding divergence." The convergence itself is diagnostic — when that many people independently name the same problem, the problem is real and the pain is universal.

Now let's return to those three Fridays.

**Ana's team is the tragic case.** They don't have cognitive debt — they review everything. Ana hasn't approved a line she didn't understand. And her reward is a Slack full of blocked teammates, a sprint velocity that's dropping despite record implementation speed, and a growing suspicion from leadership that her team "isn't adapting fast enough." "Who owns this code?" — Ana does. All of it. That's the problem. The question her organization needs to answer isn't "how do we review faster?" — it's "what genuinely needs human understanding and what doesn't?"

**Marcus doesn't have a backlog.** He doesn't have blocked teammates. He also doesn't have a knot in his stomach — and that's the problem. His code didn't pass through human review at all. It's not on any balance sheet. He can't point at the PRs he rushed because he didn't rush them — he never entered the loop. 12,000 lines running in production, and Marcus reviews outcomes, not implementations. "Who owns this code?" Marcus would say he does. He'd be wrong. He owns the *outcomes*. Nobody owns the *code*. This is **alien code** — code that bypassed human mapping entirely. Unknown unknowns. He'll discover it the way you discover a gas leak: when something explodes.

**Remember that knot in your stomach? It has a name now.** Your 4:47 PM LGTM created **cognitive debt** — code you *know* you didn't fully understand, sitting in production. Known unknowns. You can point at the PRs you rushed. You remember which ones made you uneasy. You can measure it. You can choose to pay it down by going back, this time with coffee and no deadline, and actually reading what you approved. It's on the balance sheet. That's the one good thing about it — you know it's there. "Who owns this code?" You're not sure. But at least you know you're not sure.

Cognitive debt is what you get when "who owns this?" has an uncertain answer. Alien code is what you get when the question has no answer at all.

For the purposes of this article — and for measurement — I'll define cognitive debt concretely:

> **Cognitive Debt** = the percentage of lines of code in production that have not been explicitly reviewed, understood, and endorsed by a human.

Think of it as the inverse of code coverage, but for human comprehension. We've spent decades measuring what machines have tested. We have never systematically measured what humans have understood. Code coverage asks: "Has a machine exercised this line?" Cognitive debt asks: "Has a human understood this line?" Both questions matter. Only one has tooling.

The distinction between cognitive debt and alien code matters operationally because they require different responses. Cognitive debt is manageable — you know about it, you can prioritize it, you can assign someone to review it. It shows up in the dashboard with a number next to it. Alien code is dangerous precisely because you don't even know it's there. It doesn't show up on any dashboard because nobody knows to look for it. You discover it during an incident, and the discovery itself is the incident.

And then there's the illusion of green tests. Agents write syntactically flawless tests for logically flawed assumptions. The test suite passes brilliantly. The coverage number is pristine. The code does precisely the wrong thing, and the tests enthusiastically confirm it. A data transformation introduces a subtle rounding error in currency calculations — and the agent-generated test suite asserts, correctly, that the function returns exactly the wrong value. The test is perfect. The assumption is catastrophic. And nobody caught it because the coverage report said 94%.

The cost of this illusion compounds in ways that aren't obvious at first. Generating those tests costs token money. Running them costs CI compute. Maintaining them costs human attention — except nobody is actually maintaining them because nobody understands them. We're paying real money to build and operate an increasingly elaborate illusion of safety. It's not just useless — it's actively harmful, because it gives us confidence where we should have doubt.

The way we think about testing is about to change fundamentally. When agents generate thousands of lines of opaque tests that nobody reads, maintains, or even understands — tests that cost real money to generate and real money to run — we need to ask whether the entire testing paradigm needs rethinking. That's an entire subject on its own. For now, let's stay focused on the comprehension gap — and talk about how not to repeat the mistakes we made with the last metric we invented.

---

## 4. The Coverage Trap

Before we build dashboards for cognitive debt, we need to talk about [Goodhart's Law](https://en.wikipedia.org/wiki/Goodhart%27s_law): *"When a measure becomes a target, it ceases to be a good measure."*

We've been here before, and the scars should still be fresh.

Code coverage was a great idea. You write tests, you measure how much of your code those tests exercise, and you use that number to identify gaps in your testing strategy. Simple, useful, actionable. Then someone put it on a dashboard. Then someone put a threshold on the dashboard. Then the threshold became a gate in CI. Then managers started asking why Team B had 73% coverage when Team A had 89%. Then developers started writing tests that technically touched every line but asserted nothing meaningful — tests whose sole purpose was to make a number go up. The metric became a doctrine. The dashboards were green. The code was still breaking in production. But the dashboards were green, so everything must be fine.

The same fate awaits cognitive debt if we're not careful. Three warnings for whatever measurement framework emerges:

**First: this is not a performance metric.** The moment you measure individual developers by their endorsement count, you've lost. People will script auto-endorsements faster than you can say "gamification." They'll bulk-approve code they haven't read, just like they wrote meaningless tests to hit coverage numbers. It's the same trap as measuring by PR count or commit count — the metric becomes the target, and the behavior it was meant to track goes underground. Cognitive debt should be a *system health indicator*, not a *developer productivity metric*. The same way you don't blame individual developers for low test coverage on a legacy module — you identify the gap and address it structurally.

**Second: zero cognitive debt is an anti-pattern.** This is what's happening to Ana right now. Her team is trying to drive cognitive debt to zero through exhaustive review. She's succeeding — every line reviewed, every PR understood. And her team can't ship. She bought a Formula 1 car and she's driving it at 30 km/h because she wants to read every road sign. The goal isn't zero. The goal is *intentional and bounded*. Some cognitive debt is not just acceptable — it's *necessary*.

**Third: strategic ignorance is engineering judgment, not negligence.** Here's what nobody has told Ana yet: she doesn't need to review everything. A generated API client from an OpenAPI spec? Endorse the spec, not the 15,000 lines of output. A lock file updated by Dependabot? That's alien code by design — it was never meant for human eyes. A well-documented cryptographic library with a decade of production use? Endorse its interface and behavior, not its internals. *Strategic Ignorance* — the conscious, documented decision to exclude certain code from your comprehension boundary. It's not negligence. It's triage. And it's the difference between Ana going home at 6 PM or 8 PM.

Now think about Marcus. He's been sleeping well for months. The dashboards are green. The agents are shipping. And somewhere in those 12,000 lines nobody read, a data transformation has been introducing a rounding error in currency conversions — just enough to be invisible on any single transaction, just enough to be catastrophic across millions of them over six months. Marcus isn't paying interest on cognitive debt. He doesn't even know he has a loan.

The consequences of flying blind don't stop at silent data corruption. At 3 AM, a Tier-1 service goes down. The on-call engineer opens the failing module and sees 2,000 lines of agent-generated code that nobody on the team has ever reviewed. There's no institutional knowledge to draw on. There's no "ask Maria, she wrote this" because Maria didn't write it — an agent did, three months ago, and the PR was approved in 90 seconds. MTTR doesn't just increase — it explodes. The engineer isn't debugging; they're *learning the system for the first time during a production incident*, under pressure, at 3 AM, with Slack notifications piling up and the incident commander asking for ETAs they can't give.

An edge-case bug surfaces in a module that three different agents have modified over the past quarter. No human has endorsed any of those changes. Nobody on the current team can explain why the module is structured the way it is, what trade-offs were made, or what invariants it's supposed to maintain. The bug report turns into a full module re-learning exercise. What should take an hour takes a week. What should cost one engineer's afternoon costs three engineers' entire sprint.

Fifty microservices all retrying with the same exponential backoff strategy that an agent chose because it's technically correct by the textbook — but collectively catastrophic when they all back off and retry in sync, creating thundering herds that take down shared infrastructure. Individual modules are locally elegant. Collectively, they create emergent behaviors that no single module's tests would catch.

And the security dimension is getting harder to ignore. [One in five breaches are now attributed to AI-generated code](https://www.rg-cs.co.uk/ai-generated-code-blamed-for-1-in-5-breaches/), and [AI-generated code creates 1.7x more issues than human-written code](https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report). Not because AI writes obviously insecure code — it doesn't. It writes *subtle* code that passes surface-level review. The vulnerabilities are structural, not syntactic. They live in the gap between what the code does and what the reviewer thinks it does. And that gap is exactly what cognitive debt measures.

---

## 5. The Middle Path

The article has diagnosed three anti-patterns. Time to take a stance.

Full ownership is a delusion. It was a delusion before AI — nobody ever understood every line. CODEOWNERS was already a convenient fiction. The difference is that pre-AI, the fiction was close enough to reality that it worked. Now it's not.

Total ignorance — Marcus's model — is unacceptable for anything on the critical path. Alien code in your lock files is fine. Alien code in your payment gateway is a ticking bomb.

The answer is in the middle: **Strategic Ignorance with Endorsement Tracking.** Choose what to understand. Choose what to skip. Track both decisions. Set thresholds so the blind spots don't grow silently. Use tooling to make it sustainable.

Four principles:

**1. Accept cognitive debt as a tool, not a failure.** Like financial debt: taking a mortgage isn't irresponsible. Not knowing you have a mortgage is. Your problem at 4:47 PM wasn't that you had cognitive debt — it's that you didn't know how much or where.

**2. Draw the comprehension boundary deliberately.** Your Tier-1 critical path — payments, auth, core data: less than 20% cognitive debt. Business logic and integrations: up to 40%. Internal tools and utilities: 70% is fine. Generated code, lock files, build artifacts: excluded entirely. This is Strategic Ignorance — conscious triage, not negligence.

**3. Eliminate alien code on the critical path.** Marcus's model works for experiments, prototypes, internal tools. It does NOT work for anything that handles money, user data, or system integrity. If code is on the critical path, a human must have endorsed it. Period.

**4. Make endorsement the unit of comprehension, not review.** Ana's problem is that "review" is tied to the PR cycle — every PR, every time, blocking the pipeline. Endorsement is different: it's a durable declaration — "I understand this module" — that survives until the code structurally changes. Ana reviews once, endorses, and the endorsement persists. She's freed from the PR treadmill.

What does this look like for each of them?

You keep your speed. You accept some cognitive debt. But you *know* where it is, you track it, and you set limits. Your heatmap has honest patches of red and green. The red is intentional.

Ana stops reviewing every PR and starts endorsing modules. Her review queue unblocks. The pipeline gates on cognitive debt thresholds, not on her personal approval. She goes home at 6 PM. Her Slack is quiet.

Marcus can't keep running blind. He starts with his `.vouchignore` — excluding the stuff that genuinely doesn't need human eyes. Everything else needs at least one human endorser. His heatmap goes from solid red to "red with intent" — and the intent is documented.

The goal isn't that everyone owns everything. The goal is that "who owns this?" always has at least one honest answer for anything on the critical path.

The philosophy is clear. But a philosophy without tools is a motivational poster. Let's build the tools.

---

## 6. From Writing Syntax to Governing Intent

The role of the software engineer is shifting. Not disappearing — shifting. We're moving from writing code to governing the intent of code written by machines. The skill that matters now isn't typing speed or language fluency — it's the ability to read, evaluate, and take ownership of code you didn't write. That's always been a skill. Now it's *the* skill.

This isn't speculation. The [5-7x velocity-comprehension gap](https://byteiota.com/cognitive-debt-ai-coding-agents-outpace-comprehension-5-7x/) is real and widening: AI agents generate 140-200 lines per minute. Human comprehension sits at 20-40 lines per minute. No amount of "read faster" closes that gap. The tooling must change because the biological constraint won't. We don't make pilots fly faster — we build better instruments.

[Gergely Orosz](https://newsletter.pragmaticengineer.com/p/the-future-of-software-engineering-with-ai) has been tracking this shift extensively. [Anthropic's own research](https://www.anthropic.com/research/AI-assistance-coding-skills) on how AI assistance impacts coding skills points to the same conclusion: maintaining genuine understanding requires deliberate effort and structural support.

Let's look at the tools we have and why they fall short. **`git blame`** tells you who committed a line, not who understood it. In the age of AI agents, it increasingly points at bots. **`git bisect`** finds the commit that broke things; it tells you nothing about whether anyone understood that commit. **`CODEOWNERS`** routes review requests — it's a notification mechanism, not a comprehension signal, and it's notoriously stale. **Commit messages** are cryptic when written by hurried humans and generic when written by AI.

All of these tools track **authorship** and **routing**. None of them track **endorsement** — the explicit, human assertion: "I have reviewed this code, I understand what it does, and I am willing to be the point of contact when it breaks."

That gap is what the VOUCH framework — and the VOUCH Protocol — aim to fill.

### The VOUCH Framework

**VOUCH** — **V**alidated **O**wnership and **U**nderstanding of **C**ode by **H**umans — is a conceptual framework for tracking human endorsement of code, and the implementation of the middle path described above. Think of it as CODEOWNERS with teeth — granular, automated, AST-aware, and integrated into the development workflow rather than bolted onto it.

The core model rests on a few principles:

**Committer = Author by default.** When you or an agent commits code, git records authorship. Nothing changes here. VOUCH doesn't replace git — it adds a layer on top.

**Authorship is not endorsement.** This is the key distinction. Code enters the repository *unendorsed* by default. It remains unendorsed until a human explicitly claims comprehension — not just "I looked at it" but "I understand what this does and I'm willing to own it." An agent can author code. A human endorses it — or doesn't.

**Escape hatches for noise.** Not every commit needs endorsement tracking. Formatter runs, CI configuration changes, dependency bumps — these create noise if you track them. Commits tagged `style:`, `chore:`, or `ci:` bypass endorsement tracking entirely.

**Community endorsement as exploration.** A new engineer onboarding onto Ana's team spends a week reading the payment module. Under the current model, that's ramp-up cost. Under VOUCH, that's *debt repayment*. They endorse the module. Two people understand it now. **Onboarding is not dead time — it is debt repayment.** Every hour a new hire spends genuinely understanding a module reduces the team's cognitive debt. That's not a cost center; it's an investment with measurable returns on the dashboard.

**AST-level resilience.** Endorsements must survive cosmetic changes but invalidate on structural ones. If someone reformats a file, renames a variable, or adjusts whitespace, the endorsement should stand — the logic hasn't changed. But if someone (or some agent) modifies the control flow, changes the algorithm, or alters the data model, the endorsement must be stripped. This requires tracking AST hashes rather than raw text. It's the difference between "the code looks different" and "the code *is* different."

**The living heatmap.** Imagine the codebase as a living heatmap of human comprehension. Endorsed areas glow green. Unendorsed areas are red. The heatmap shifts in real time as people join, leave, or as agents modify code. Your codebase: honest patches of red and green. Ana's: mostly green, finally. Marcus's: still red, but now he *knows* it's red. This is the dashboard that should sit alongside DORA metrics and uptime monitors — a real-time view of *where human knowledge lives* in your system.

**KPI target anchor.** As a starting point for discussion: aim for less than 20% cognitive debt on Tier-1 critical-path services. Track it over time on engineering dashboards alongside DORA metrics, test coverage, and incident rates. A developer tools microservice can tolerate 60% cognitive debt. Your payment gateway cannot.

### The VOUCH Protocol (v0.1)

The framework needs a protocol — a formal specification that implementations can target. What follows is v0.1: deliberately minimal, intentionally incomplete, designed to be broken and improved by the community.

#### Endorsement Record

The atomic unit of the protocol is the **Endorsement Record** — a structured declaration that a human has reviewed and understood a specific piece of code at a specific point in its structural history.

```
{
  "version":    "0.1",
  "file_path":  "src/payment/gateway.go",
  "ast_hash":   "sha256:a7f3b2c1d4e5...",
  "endorser":   "mibar",
  "timestamp":  "2026-03-14T16:30:00Z",
  "scope":      "full",
  "note":       ""
}
```

Fields:

- **`version`** — Protocol version. Allows forward-compatible evolution.
- **`file_path`** — Relative path from repository root.
- **`ast_hash`** — Hash of the file's abstract syntax tree (not its text). Algorithm and granularity defined by the implementation; the protocol requires only that cosmetic changes (whitespace, formatting, comments) MUST NOT invalidate the hash, and structural changes (logic, control flow, data model) MUST invalidate it.
- **`endorser`** — Human identity. Maps to the git author or a configured alias. Non-human agents MUST NOT appear in this field.
- **`timestamp`** — ISO 8601 UTC. When the endorsement was recorded.
- **`scope`** — One of `full` (entire file), `partial` (specific functions/blocks — future extension), or `interface` (public API surface only, internals deliberately unendorsed).
- **`note`** — Optional free-text. Intended for context: "Endorsed after security audit Q1-2026" or "Interface only — internals are vendored crypto."

#### Operations

The protocol defines four operations:

**`ENDORSE`** — Create or update an endorsement record for a file. The implementation MUST compute the current AST hash and store it with the endorsement. If the file already has an endorsement by the same endorser, the operation updates the timestamp and AST hash (re-endorsement after changes).

**`REVOKE`** — Explicitly remove an endorsement. Used when an endorser leaves the team, changes roles, or determines they no longer understand the code sufficiently. Revocation is an affirmative act — endorsements don't just expire, but implementations MAY support time-based staleness warnings (e.g., "this endorsement is 18 months old").

**`QUERY`** — Retrieve endorsement status for a file, directory, or glob pattern. Returns the list of current endorsements, their staleness, and whether the AST hash still matches (i.e., whether the code has changed since endorsement).

**`REPORT`** — Compute cognitive debt metrics for a scope (file, directory, service, entire repository). Returns percentage of lines covered by valid endorsements, broken down by endorsement scope and staleness.

#### Invalidation Rules

Endorsements are not permanent. The protocol defines three invalidation triggers:

1. **Structural change.** When the AST hash of an endorsed file changes, all endorsements for that file are invalidated. The code has structurally changed; the endorser's understanding may no longer be accurate. The endorsement record is preserved but marked `invalidated: structural_change`, allowing the endorser to re-endorse quickly if the change was minor.

2. **Endorser departure.** When an endorser is removed from the team roster (however that's managed), their endorsements SHOULD be marked `at_risk` rather than immediately invalidated. The knowledge may still be reachable (the person might be in another team, or contactable), but the endorsement's operational value is degraded.

3. **Staleness.** The protocol does NOT mandate automatic expiration. Implementations MAY surface warnings for endorsements older than a configurable threshold, but the endorsement remains valid until the code changes or the endorser revokes it. Understanding doesn't expire on a timer — but it does atrophy, and surfacing staleness helps teams make conscious decisions about re-endorsement.

#### Escape Hatches

Not all code changes require endorsement re-evaluation:

- **Conventional Commit prefixes.** Commits with `style:`, `chore:`, `ci:`, or `docs:` prefixes bypass endorsement invalidation. The assumption: these commits don't change program logic. If they do, the prefix is wrong — that's a process problem, not a protocol problem.
- **`.vouchignore`** — A file using gitignore syntax that excludes paths and patterns from cognitive debt calculation entirely. Generated code, lock files, vendored dependencies, build artifacts. These are Alien Code by design. Including them would inflate the metric and dilute its signal.

#### Storage Requirements

The protocol is storage-agnostic. Endorsement records can live in git notes, a sidecar file, a database, or a service. The requirements are:

1. Records MUST be versioned alongside the code (or at least linked to specific commits).
2. Records MUST NOT create merge conflicts during normal development workflows.
3. Records MUST be queryable without checking out the full repository history.
4. Records SHOULD travel with the repository (no external service required for basic operation).
5. Actual source code MUST NOT be transmitted to third-party services to compute or store endorsement data. The protocol operates on metadata only: file paths, hashes, identities, timestamps.

The reference implementation described in the next section uses git notes. It's a reasonable starting point with real tradeoffs.

### What the framework doesn't do

Intellectual honesty demands acknowledging the limitations:

**It operates on metadata, not actual comprehension.** When someone endorses code, the framework records the endorsement. It cannot verify that the person actually understood what they endorsed. This is a trust-based system — exactly like git commit authorship itself. The protocol gives you the *mechanism*; the culture gives you the *integrity*.

**Endorsement is self-reported.** There is no comprehension quiz, no verification step, no proof of understanding. Someone can endorse code they don't understand, just like someone can approve a PR they didn't read. The framework provides the *infrastructure* for tracking; the culture provides the *incentive* for honesty.

**It only works if the culture supports it.** No tool can force genuine understanding. If the organization treats endorsement as a checkbox to be gamed — the same way some organizations treat code coverage — the metric becomes meaningless. The framework is a mirror; what it reflects depends on the organization looking into it.

---

## 7. A CLI Named `vouch` (and a Skill Named `vouch`)

### The CLI

The simplest interaction:

```bash
vouch endorse src/payment/gateway.go
```

You're telling the system: "I have reviewed this file. I understand what it does. I'm willing to be the point of contact." You do this as part of your workflow — not as a separate ceremony.

If you don't endorse, nothing breaks. The code ships normally. The committer is recorded as the author (standard git behavior), and the code is flagged as *unendorsed*. That's all. No gates, no blocks — just signal.

Query the debt:

```bash
vouch report src/payment/
# Cognitive Debt: 34% (src/payment/)
# 12 files, 8 endorsed, 4 unendorsed
# 2 endorsements invalidated (structural changes since last endorsement)
```

**Storage: git notes.** Endorsement data lives in [git notes](https://git-scm.com/docs/git-notes) — metadata attached to git objects without modifying the commit history. No manifest files cluttering the repo. No merge conflicts from competing endorsement edits. The data travels with the repository. Each endorsement is stored as a JSON record per the protocol spec. The `ast_hash` is computed via [tree-sitter](https://tree-sitter.github.io/) — a parser generator that supports 40+ languages and produces concrete syntax trees. By hashing AST nodes rather than full trees, cosmetic changes produce the same hash while structural changes produce a new one.

### The Agent Skill

The real power isn't the CLI alone — it's the `vouch` skill integrated into agentic coding tools. The agent itself participates in the endorsement workflow:

**Before modifying code**, the agent queries cognitive debt. "I'm about to modify `src/payment/gateway.go`. This file has one endorser (Ana, 3 weeks ago). Cognitive debt for the payment service is 18%. Proceeding will invalidate Ana's endorsement and push it to 24%, above the 20% threshold for Tier-1. Do you want to proceed?"

**After generating code**, the agent self-reports as author. The code is flagged as AI-generated, unendorsed. The heatmap updates in real time.

**During review**, the agent helps the human *understand* — not just write. "You asked me to explain this module. Here's what it does, why it's structured this way, and what assumptions it makes." The human reads the explanation, traces the code, and when they genuinely understand it: `vouch endorse`.

### Ana's Monday Morning

Ana opens her terminal. She runs `vouch report --tier 1`. The dashboard shows: Payment service: 22% cognitive debt. Auth service: 14%. Core data: 31%.

Core data is above threshold. She checks why: an agent refactored the query optimizer last Thursday. The change invalidated her endorsement and her colleague Marco's. Nobody has re-endorsed yet.

She opens the module. It's 400 lines. She asks her coding agent: "explain src/core/query_optimizer.go". The agent walks her through the changes: "The refactor replaced the nested loop join selection with a cost-based optimizer. The core logic changed in three functions. Here's what each one does and why."

Ana reads the explanation, traces the code, runs the tests herself, and satisfies herself that the logic is sound. She runs `vouch endorse src/core/query_optimizer.go`. The dashboard updates: Core data: 19%. Below threshold. Green.

It's 10:15 AM. She moves on to feature work. No PR queue. No blocked teammates. No Slack escalations.

### `.vouchignore` — Strategic Ignorance, formalized

Not all code needs human endorsement. The `.vouchignore` file (included in this repository) declares patterns that are excluded from cognitive debt calculation entirely, using standard gitignore syntax. Lock files, generated code, build artifacts, massive test fixtures, vendored dependencies — these are alien code by design. They were never meant to be read by humans. Excluding them is not gaming the metric; it's being honest about where human comprehension matters.

The distinction is important: `.vouchignore` is not a loophole. It's a declaration of engineering intent. When your team puts `**/generated/**` in `.vouchignore`, they're saying: "We have decided, as a team, that generated code is outside our comprehension boundary. We trust the generator, not the output." That's a valid engineering decision — as long as it's conscious and documented. The `.vouchignore` file *is* the documentation.

### Honest about the tradeoffs

[Git notes](https://tylercipriani.com/blog/2022/11/19/git-notes-gits-coolest-most-unloved-feature/) are git's most underloved feature for good reasons:

- They don't auto-push or auto-pull. You need explicit `git push origin refs/notes/*` and corresponding fetch configurations. This is friction, and friction kills adoption.
- There's a ~1MB size limit per note per commit object. For large monorepos with thousands of files, the storage model needs careful design.
- Notes attach to commit objects, not file paths. Mapping "endorsement of a file" to "note on a commit" requires an indexing layer that doesn't exist in git natively.

These are real tradeoffs, not dealbreakers. If you have a better storage mechanism — a sidecar SQLite database, a lightweight service, a custom git ref namespace — build it. The VOUCH Protocol doesn't depend on git notes. It depends on metadata that travels with the repository and doesn't create merge conflicts.

### CI/CD integration

Where `vouch` becomes operationally powerful is as a CI/CD gatekeeper. Define cognitive debt thresholds per service tier:

```yaml
# .vouch.yml
thresholds:
  tier-1:  # Payment, auth, core data
    max_cognitive_debt: 20%
  tier-2:  # Business logic, integrations
    max_cognitive_debt: 40%
  tier-3:  # Internal tools, dev utilities
    max_cognitive_debt: 70%
```

If a PR drops human endorsement of a Tier-1 critical service below the threshold, the pipeline fails. Same principle as coverage gates, but measuring comprehension, not test execution. The PR author sees: "This change introduces 450 lines of unendorsed code in a Tier-1 service. Current cognitive debt: 23% (threshold: 20%). Please endorse or request endorsement before merging."

### Agentic-assisted reviews

The next frontier isn't agents that *write* code but agents that *explain* it. Imagine an agent whose job isn't to implement a feature but to reverse-engineer the intent of code that another agent implemented — decompiling logic into human-readable explanations, annotating design decisions, surfacing implicit assumptions, pre-digesting comprehension so that humans can endorse faster and more confidently. Agents auditing agents. Not for correctness (we have tests for that) but for comprehensibility.

This is the natural course of this revolution. The same technology that created the comprehension gap will inevitably be part of closing it. But the endorsement — the human assertion of "I understand and I own this" — must remain the irreducible human element in the loop. An agent can help you understand. Only you can decide that you do.

---

## 8. Surfing the Wave Together

When the P1 hits at 3 AM — and it will — someone will ask "who owns this?"

Ana will raise her hand. She endorsed this module. She can explain it. But the fix touches a module that's been in her review queue for two weeks — one she hasn't endorsed yet.

You open the failing module and recognize the PR you approved at 4:47 PM. The knot in your stomach was right.

Marcus opens a module no human has ever seen and starts learning the system during a production incident.

This is not a problem we solve by "trying harder to read PRs." The volume has already exceeded human bandwidth. We don't solve it by banning AI tools either — that ship has sailed, and it shouldn't come back. We solve it by building systems that track human comprehension as a first-class engineering metric, the same way we track test coverage, deployment frequency, and change failure rate.

The conversation is already happening. [Storey](https://margaretstorey.com/blog/2026/02/09/cognitive-debt/) named the debt. [Osmani](https://addyo.substack.com/p/code-review-in-the-age-of-ai) documented the review bottleneck. [Beck](https://tidyfirst.substack.com/p/augmented-coding-beyond-the-vibes) articulated the skill shift. [Karpathy](https://thenewstack.io/vibe-coding-is-passe/) acknowledged the limits of vibes. [Orosz](https://newsletter.pragmaticengineer.com/p/the-future-of-software-engineering-with-ai) mapped the new landscape. The pieces are on the table. What's missing is the tooling to make it operational and the protocol to make it interoperable.

That's what VOUCH is trying to be — not the final answer, but the first formal attempt at an answer. A protocol that says: here's what an endorsement is, here's how it's stored, here's how it's invalidated, and here's how you measure the gap. v0.1. Incomplete by design. Built to be forked.

So here's the call to action, and it's simple:

Try measuring cognitive debt in your own codebase. Pick a critical service. Walk the code. Ask: "Who on this team can explain what this module does at 3 AM?" If the answer is "nobody" or "maybe Sarah, but she's on vacation," that's your cognitive debt, right there. No tooling required — just honest assessment.

Build better tools. The `vouch` CLI outlined above is a reference implementation sketch, not a finish line. If git notes are wrong, use something else. If AST hashing is too coarse, make it finer. If the protocol has blind spots, expose them. Fork the protocol, break it, submit PRs with improvements. Build a VS Code extension that shows the heatmap inline. Build a GitHub Action that gates PRs on cognitive debt thresholds. Build a Grafana dashboard that tracks endorsement coverage alongside uptime. The important thing is that *something* exists — some mechanism for humans to explicitly say "I own this" in a way that's tracked, queryable, and integrated into the development workflow.

Challenge the metric. If you think cognitive debt is measurable but this isn't the right way to measure it, write about that. If you think it's unmeasurable by nature — that any attempt to quantify human understanding is fundamentally flawed — write about that too. Push back. Poke holes. The VOUCH Protocol is v0.1 for a reason: it's designed to be wrong in instructive ways. The worst outcome isn't that we measure it wrong — it's that we don't measure it at all and discover the consequences during a production incident.

But above all: don't let it become another weaponized dashboard number. Don't tie it to performance reviews. Don't create leaderboards. Don't shame teams with high cognitive debt. The moment you do, people will game the metric, and you'll have gained nothing except a false sense of security on top of genuine ignorance — which is strictly worse than just the genuine ignorance.

Start asking the question. Not in a postmortem — in standup. Not "who committed this" or "who reviewed this" or "whose name is in CODEOWNERS." Who *understands* this? If nobody raises their hand — now you know where to start. And you can stop praying.

---

*This article proposes the VOUCH Protocol (v0.1) and the VOUCH Framework (Validated Ownership and Understanding of Code by Humans) as a starting point for managing cognitive debt. The term "cognitive debt" originates from [Kosmyna et al. at the MIT Media Lab](https://www.media.mit.edu/publications/your-brain-on-chatgpt/) (June 2025). [Margaret-Anne Storey](https://margaretstorey.com/blog/2026/02/09/cognitive-debt/) extrapolated it to software engineering (February 2026). Multiple groups converged independently on the same concept. This article's contribution is the VOUCH Protocol: the first formal specification for tracking human endorsement and ownership of code deterministically.*

*If you found this useful, the best thing you can do is challenge it. Open an issue, write a response, build a prototype, fork the protocol. The problem is real. The solution is unfinished.*
