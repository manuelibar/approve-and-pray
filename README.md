# Approve and Pray. Halt and Pray. Cognitive Debt, Code Ownership, and the Tools We're Missing

*By [Manuel Ibar](https://github.com/mibar) — March 2026*

---

## 1. Two Fridays

Let me tell you two stories. They're unfolding simultaneously, in different corners of the industry, and they're both about the same pull request.

**The first story** takes place at a Series B startup. It's Friday at 4:47 PM. A PR landed an hour ago — 847 lines changed across 23 files, generated in about twelve minutes by an agent that never gets tired, never gets distracted, and never second-guesses itself. The CI is green. The senior engineer assigned to review it has been staring at it for forty minutes. They understand maybe 60% of what it does. The product owner pinged them twice asking about demo readiness. The team lead is asking why the sprint board still has three stories stuck "in review." They type "LGTM," hit merge, and feel a knot in the stomach they can't quite name.

They didn't rubber-stamp it. Rubber-stamping is lazy — it's the act of not caring. This was something more anxious and visceral. They *tried* to review it. Opened every file. Traced every function call they could follow. Cross-referenced the ticket. Checked the test assertions. And then hit the wall of their own cognitive bandwidth. They approved it anyway, because the alternative was blocking the entire team for another day, and there's a demo on Monday. They approved and prayed.

**The second story** takes place at a Fortune 500 that learned, through years of expensive incidents, not to ship code without process. The same PR — same 847 lines, same agent, same CI — enters the review queue on Monday. It needs a security sign-off first. The security review backlog is two days deep. On Wednesday, the security engineer reviews it for eleven minutes, checks for injection vulnerabilities and obvious secret exposure, finds nothing, approves. It still needs an architecture review: the board meets on Thursdays. Thursday comes; the architect has six PRs on the agenda and eight minutes for this one — design is consistent with existing patterns, good enough, approved. Still needs two senior engineer approvals, required by the team's branching policy. Both seniors are processing an average of eleven PRs per week on top of their own feature work. First senior approves Friday. Second senior asks two clarifying questions. The author — who has mentally shipped this code and moved on to the next sprint — answers on Tuesday. Final approval arrives Wednesday. Fifteen calendar days after the PR was opened. Six separate JIRA transitions.

Five people reviewed it. None of them understood it as a *system*. The security engineer understood the surface. The architect understood the structure. The seniors understood fragments of the logic. Nobody looked at the whole picture. Nobody talked to each other about this specific code. Nobody owns the mental model of what this module does, why it does it that way, or what breaks if the assumptions change.

The code ships, fifteen days late. They halted and prayed.

The cognitive debt is identical to the startup's.

Somewhere right now, a very confident AI is writing `UPDATE users SET role = 'admin'`. At the startup, a very tired human is approving it at 4:47 PM on a Friday — approved and prayed. At the corp, five moderately-attentive reviewers are halting it for fifteen calendar days across six JIRA transitions, then shipping it anyway — halted and prayed. The approvals don't compose into comprehension. They just add latency to the same outcome.

This is what makes cognitive debt different from every other software quality problem we've faced. It isn't solved by moving faster — the startup proves that. It isn't solved by adding more process — the corp proves that too. The startup's problem is too little friction: code ships before anyone understands it. The corp's problem is too much friction in the wrong places: code takes weeks to ship, and still nobody understands it at the end. Two completely different failure modes. One root cause.

The numbers tell both stories with uncomfortable precision. In 2025, [41% of code was already AI-generated](https://www.quantumrun.com/consulting/github-copilot-statistics/). By early 2026, teams using AI coding tools saw a [98% increase in PR volume with a 91% increase in review time](https://blog.logrocket.com/ai-coding-tools-shift-bottleneck-to-review/). Pull requests are [18% larger on average, and incidents per PR have climbed 24%](https://addyo.substack.com/p/code-review-in-the-age-of-ai). Senior engineers now spend an average of 4.3 minutes reviewing AI-generated code versus 1.2 minutes for human-written code — and that 4.3 minutes is being rationed across a review queue that never stops growing. The [review bottleneck is already here](https://levelup.gitconnected.com/the-ai-code-review-bottleneck-is-already-here-most-teams-havent-noticed-1b75e96e6781), and most teams haven't noticed it — because for startups the symptom looks like "going fast," and for corps the symptom looks like "being thorough."

Here's the asymmetry that neither approach addresses: AI generation speed has increased roughly 10x in the last two years. Human comprehension speed has not increased at all. It can't. It's a biological constant — bounded by working memory, attention span, and the speed at which the human brain builds mental models of complex systems. We are connecting a firehose to a garden hose and wondering why the garden is flooding. Adding more valves to the garden hose doesn't help. It just spreads the flooding across more people's calendars.

This is a backpressure problem. In distributed systems, when a producer overwhelms a consumer, backpressure builds until something breaks — the queue overflows, the consumer crashes, or data gets silently dropped. The same dynamics are playing out in engineering organizations right now. At the startup, the backpressure is explicit: the queue is visible, the pressure is felt, the "approved and prayed" is a conscious act of desperation. At the corp, the backpressure is diffuse: it spreads across review committees and architecture boards and compliance gates until the pressure per reviewer feels manageable — but the aggregate comprehension at the end of the pipeline is just as thin. There is no autoscaling for human cognition. You can't spin up another brain.

**Every approved-and-prayed PR permanently mints new cognitive debt — whether it took four minutes or fifteen days, whether one person approved it or five. We are trading system comprehension for delivery speed at the startup, and for process compliance at the corp. The exchange rate is getting worse every quarter, and neither model is paying it down.**

We have a name for code we don't maintain. We call it technical debt. But what do we call code we don't *understand*?

---

## 2. A Brief History of Owing the Compiler Money

In 1992, Ward Cunningham stood up at [OOPSLA and introduced the debt metaphor](https://c2.com/doc/oopsla92.html) while explaining the design decisions behind the WyCash portfolio management system. The idea was simple and powerful: sometimes you ship code you know isn't ideal because the business value of shipping now outweighs the cost of cleaning up later. You're borrowing against the future. As long as you pay it back — refactor, rewrite, clean up — the debt is manageable. If you don't, the interest compounds until the codebase becomes unmaintainable.

That metaphor changed how we talk about software. [Martin Fowler](https://martinfowler.com/bliki/TechnicalDebt.html) formalized it. Entire organizations built dashboards around it. "Technical debt" entered the vocabulary of product managers, CTOs, and board members who'd never written a line of code but intuitively understood what it meant to borrow against the future. For thirty years, it was the right abstraction.

But it carried an implicit assumption that nobody questioned because it was always true: *you understood the code you were choosing to defer*.

Technical debt is a conscious trade-off. You wrote the module. You know it's messy. You know the edge cases it doesn't handle. You ship it anyway because the deadline matters more than elegance right now, and you plan to come back. The debt is intentional, localized, and — critically — understood by the person who incurred it. That understanding is what makes it manageable. You know exactly what you owe and where the bodies are buried.

The vocabulary evolves because the practice evolves. In February 2025, Andrej Karpathy coined ["vibe coding"](https://x.com/karpathy/status/1886192184808149383) — describing the experience of programming by feel with AI, barely reading the generated code, just running it until it works. It captured something real: the seductive efficiency of not needing to understand every line. A year later, [he walked it back](https://thenewstack.io/vibe-coding-is-passe/): "Vibe coding is passé." The term served its moment. Now we talk about "agentic engineering" — autonomous agents writing, testing, and deploying code with minimal human intervention. The vocabulary shifted because the practice shifted from "human prompting AI for snippets" to "AI autonomously implementing features end-to-end."

And here's where Cunningham's metaphor reaches its breaking point. With agents, "refactoring later" becomes automated. An agent rewrites a legacy module overnight. Another agent picks up the tech debt it left behind. The TODO comments get resolved. The deprecated APIs get updated. The linter warnings disappear. The test coverage goes up. Technical debt drops to near zero on paper — the agents are tireless, thorough, and cheap. But the sheer volume of code implemented by this automation is staggering — and nobody reviewed any of it. The debt on the spreadsheet is gone. The understanding never existed. We've paid off the credit card by taking out a mortgage we don't fully understand the terms of.

Technical debt used to cause parallelization problems in a familiar way: accumulated cruft slowed everyone down. You couldn't add a feature without untangling the mess. Now we have a *worse* parallelization problem. During incidents, security audits, or onboarding, multiple teams hit the same wall at the same time — nobody understands the code. The cruft is gone. The system is clean, well-tested, beautifully formatted, and completely opaque to every human being responsible for it.

**We are trading debt we controlled — technical debt — for debt that accrues globally and lacks accountability.**

Technical debt is code you wrote and chose not to clean up. What we're accumulating now is fundamentally different: it's code you never understood in the first place.

---

## 3. Naming the Ghost

In June 2025, a team at the MIT Media Lab published a study titled ["Your Brain on ChatGPT"](https://www.media.mit.edu/publications/your-brain-on-chatgpt/) (Kosmyna et al., arXiv:2506.08872). They used functional near-infrared spectroscopy — essentially, they watched people's brains work — to measure brain connectivity in people completing cognitive tasks with and without LLM assistance. The finding was striking: participants who relied on LLMs showed the *weakest* brain connectivity patterns. They couldn't accurately recall or explain their own work. The researchers called this phenomenon *cognitive debt* — not as a software metaphor, but as a measurable neurological outcome. The tools that made us faster were also making our brains less engaged with the output. The debt wasn't metaphorical. It was happening in the prefrontal cortex.

In February 2026, [Margaret-Anne Storey](https://margaretstorey.com/blog/2026/02/09/cognitive-debt/) took the term and extrapolated it to software engineering. Drawing on Peter Naur's seminal insight that "a program is a theory" held in the programmer's mind — that the true nature of a software system lives not in the source code but in the mental model of the people who built it — and echoing Fred Brooks and [Kent Beck](https://tidyfirst.substack.com/p/augmented-coding-beyond-the-vibes), she argued that the real debt isn't in the code. It's in the developers' heads. When the human mental model of a system degrades, the system becomes ungovernable regardless of how clean the code looks. You can have zero technical debt and still be unable to operate your own software.

What happened next is telling: [at least five independent groups](https://byteiota.com/cognitive-debt-ai-coding-agents-outpace-comprehension-5-7x/) converged on the same concept within the same quarter. Different names, same pain. [CodeRabbit](https://www.coderabbit.ai/blog/2025-was-the-year-of-ai-speed-2026-will-be-the-year-of-ai-quality/) called 2025 "the year of AI speed" and 2026 "the year of AI quality." Others called it the "comprehension gap" or the "velocity-understanding divergence." The convergence itself is diagnostic — when that many people independently name the same problem, the problem is real and the pain is universal.

The term has neural origins. Let's give that its due and build on it.

For the purposes of this article — and for measurement — I'll define cognitive debt concretely:

> **Cognitive Debt** = the percentage of lines of code in production that have not been explicitly reviewed, understood, and endorsed by a human.

Think of it as the inverse of code coverage, but for human comprehension. We've spent decades measuring what machines have tested. We have never systematically measured what humans have understood. Code coverage asks: "Has a machine exercised this line?" Cognitive debt asks: "Has a human understood this line?" Both questions matter. Only one has tooling.

This gives us two distinct KPIs worth tracking separately:

**Cognitive Debt (Known Unknowns):** Code we *know* lacks a human endorser. It's on the balance sheet. We can see it, track it, and make conscious decisions about it. A new module written by an agent that nobody has reviewed yet — that's cognitive debt. It shows up in the dashboard. Someone can be assigned to review it. The team can decide whether it's worth the investment or whether they'll accept the risk. This is manageable debt.

**Alien Code (Unknown Unknowns):** Code that bypassed human mapping entirely. Autonomously generated end-to-end, never reviewed, perhaps never even surfaced for review. The agent committed it, the tests passed, and it went to production without a single human being aware of its specific logic. This is fine in your `go.sum` file or your auto-generated protobuf bindings. It is a nightmare in your payment gateway or your authentication module.

The distinction matters operationally because they require different responses. Cognitive debt is manageable — you know about it, you can prioritize it, you can assign someone to review it. It shows up in the dashboard with a number next to it. Alien code is dangerous precisely because you don't even know it's there. It doesn't show up on any dashboard because nobody knows to look for it. It exists in the gaps between what you track and what you assume. You discover it during an incident, and the discovery itself is the incident.

And then there's the illusion of green tests. Agents write syntactically flawless tests for logically flawed assumptions. The test suite passes brilliantly. The coverage number is pristine. The code does precisely the wrong thing, and the tests enthusiastically confirm it. That `UPDATE users SET role = 'admin'` has a beautifully written test asserting that it correctly updates all users to admin — because that's what the agent understood the requirement to be. The test is perfect. The assumption is catastrophic. And nobody caught it because the coverage report said 94%.

The cost of this illusion compounds in ways that aren't obvious at first. Generating those tests costs token money. Running them costs CI compute. Maintaining them costs human attention — except nobody is actually maintaining them because nobody understands them. We're paying real money to build and operate an increasingly elaborate illusion of safety. It's not just useless — it's actively harmful, because it gives us confidence where we should have doubt.

The way we think about testing is about to change fundamentally. When agents generate thousands of lines of opaque tests that nobody reads, maintains, or even understands — tests that cost real money to generate and real money to run — we need to ask whether the entire testing paradigm needs rethinking. What does "test coverage" even mean when the tests are as opaque as the code they cover? That's an entire subject on its own, and I'll probably write a dedicated article on it. For now, let's stay focused on the comprehension gap.

Having a name is step one. Step two is not repeating the mistakes we made with the last metric we invented.

---

## 4. The Coverage Trap

Before we build dashboards for cognitive debt, we need to talk about [Goodhart's Law](https://en.wikipedia.org/wiki/Goodhart%27s_law): *"When a measure becomes a target, it ceases to be a good measure."*

We've been here before, and the scars should still be fresh.

Code coverage was a great idea. You write tests, you measure how much of your code those tests exercise, and you use that number to identify gaps in your testing strategy. Simple, useful, actionable. Then someone put it on a dashboard. Then someone put a threshold on the dashboard. Then the threshold became a gate in CI. Then managers started asking why Team B had 73% coverage when Team A had 89%. Then developers started writing tests that technically touched every line but asserted nothing meaningful — tests whose sole purpose was to make a number go up. The metric became a doctrine. The dashboards were green. The code was still breaking in production. But the dashboards were green, so everything must be fine.

The same fate awaits cognitive debt if we're not careful. Three warnings for whatever measurement framework emerges:

**First: this is not a performance metric.** The moment you measure individual developers by their endorsement count, you've lost. People will script auto-endorsements faster than you can say "gamification." They'll bulk-approve code they haven't read, just like they wrote meaningless tests to hit coverage numbers. It's the same trap as measuring by PR count or commit count — the metric becomes the target, and the behavior it was meant to track goes underground. Cognitive debt should be a *system health indicator*, not a *developer productivity metric*. The same way you don't blame individual developers for low test coverage on a legacy module — you identify the gap and address it structurally.

**Second: zero cognitive debt is an anti-pattern.** If your cognitive debt is zero, something is wrong. It means you're manually reviewing every line of every generated module, which means you've eliminated the speed advantage of AI entirely. You've bought a Formula 1 car and you're driving it at 30 km/h because you want to read every road sign. Some cognitive debt is not just acceptable — it's *necessary*. The goal is to keep it *intentional and bounded*, not to eliminate it. The question isn't "do we have cognitive debt?" — it's "do we have it in the right places and at acceptable levels?"

**Third: strategic ignorance is engineering judgment, not negligence.** There's code you deliberately choose not to endorse because understanding it isn't worth the investment. A well-documented cryptographic library with solid test coverage and a decade of production use? You don't need to understand the internals of the elliptic curve implementation. You endorse its interface, its tests, its behavior — not its guts. A generated API client from an OpenAPI spec? Endorse the spec and the generator configuration, not the 15,000 lines of boilerplate output. A dependency's lock file that gets auto-updated weekly by Dependabot? That's Alien Code by design — it was never meant for human comprehension, and pretending otherwise just dilutes the metric. That's not laziness. That's an engineering decision about where human comprehension delivers the most value per hour invested. The term I'd use is *Strategic Ignorance* — the conscious, documented decision to exclude certain code from your cognitive debt calculation. It should have its own configuration file. More on that later.

Now, here's what happens when you *don't* measure it at all. These aren't hypothetical scenarios. These are the predictable consequences of flying blind on human comprehension:

**Silent data corruption.** The system runs perfectly — all tests pass, no errors, no alerts, metrics look normal — while subtly mutating data for months. An agent wrote a data transformation that technically satisfies every test case but introduces a rounding error in currency conversion that only manifests with certain decimal combinations. Nobody understood the code well enough to catch it. Nobody even looked at it, because the tests were green. By the time someone notices, six months of financial records have sub-cent discrepancies across millions of transactions.

**Incident response blackout.** It's 3 AM. A Tier-1 service is down. The on-call engineer opens the failing module and sees 2,000 lines of agent-generated code that nobody on the team has ever reviewed. There's no institutional knowledge to draw on. There's no "ask Maria, she wrote this" because Maria didn't write it — an agent did, three months ago, and the PR was approved in 90 seconds. MTTR doesn't just increase — it explodes. The engineer isn't debugging; they're *learning the system for the first time during a production incident*, under pressure, at 3 AM, with Slack notifications piling up and the incident commander asking for ETAs they can't give.

**Ownership vacuum.** An edge-case bug surfaces in a module that three different agents have modified over the past quarter. No human has endorsed any of those changes. Nobody on the current team can explain why the module is structured the way it is, what trade-offs were made, or what invariants it's supposed to maintain. The bug report turns into a full module re-learning exercise. What should take an hour takes a week. What should cost one engineer's afternoon costs three engineers' entire sprint.

**Distributed time bombs.** Individual agent-generated modules are locally elegant. They pass their own tests beautifully. But across the ecosystem, they create emergent behaviors that no single module's tests would catch. Fifty microservices all retrying with the same exponential backoff strategy that an agent chose because it's technically correct by the textbook — but collectively catastrophic when they all back off and retry in sync, creating thundering herds that take down shared infrastructure.

And the security dimension is getting harder to ignore. [One in five breaches are now attributed to AI-generated code](https://www.rg-cs.co.uk/ai-generated-code-blamed-for-1-in-5-breaches/), and [AI-generated code creates 1.7x more issues than human-written code](https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report). Not because AI writes obviously insecure code — it doesn't. It writes *subtle* code that passes surface-level review. The vulnerabilities are structural, not syntactic. They live in the gap between what the code does and what the reviewer thinks it does. And that gap is exactly what cognitive debt measures.

So how do you actually track who understands what? Start with the tools we already have — and understand why they're not enough.

---

## 5. From Writing Syntax to Governing Intent

The role of the software engineer is shifting beneath our feet. Not disappearing — shifting. We're moving from writing code to governing the intent of code written by machines. The lovers of code, the crafters of masterpieces who cannot let go of syntax perfection, are doomed to be left out if they do not adapt. The skill that matters now isn't typing speed or language fluency — it's the ability to read, evaluate, and take ownership of code you didn't write. That's always been a skill. Now it's *the* skill.

This isn't speculation. The data is clear. The [5-7x velocity-comprehension gap](https://byteiota.com/cognitive-debt-ai-coding-agents-outpace-comprehension-5-7x/) is real and widening: AI agents generate 140-200 lines per minute. Human comprehension sits at 20-40 lines per minute. No amount of "read faster" closes that gap. No amount of "be more disciplined about reviews" solves it either. The tooling must change because the biological constraint won't. We don't make pilots fly faster — we build better instruments.

[Gergely Orosz](https://newsletter.pragmaticengineer.com/p/the-future-of-software-engineering-with-ai) has been tracking this shift extensively. [Anthropic's own research](https://www.anthropic.com/research/AI-assistance-coding-skills) on how AI assistance impacts coding skills points to the same conclusion: the challenge isn't AI quality; it's human engagement. When the tool does the work for you, maintaining genuine understanding requires deliberate effort and structural support. Without that structure, comprehension atrophies — and with it, the ability to govern what the machines produce.

Let's look at the tools we have and why they fall short.

**`git blame`** tells you who committed a line, not who understood it. Run Prettier across a codebase and you now "own" 10,000 lines you never read. Rename a variable across 50 files with a search-and-replace and `git blame` says you're the expert on all of them. It tracks *authorship* — but authorship of a keystroke, not comprehension of a design. In the age of AI agents, `git blame` increasingly points at bots.

**`git bisect`** is brilliant for finding the commit that broke things. It tells you nothing about whether anyone understood that commit when it was merged. It's a forensic tool for incidents, not a governance tool for comprehension. Useful *after* the damage is done; useless for prevention.

**`CODEOWNERS`** routes review requests. That's it. It's a notification mechanism, not a comprehension signal. It operates at directory level — too coarse for meaningful ownership. It's notoriously stale: the person listed as owner left the company two quarters ago, and nobody updated the file because nobody wants to own the file that assigns ownership. Worse, being listed as a code owner doesn't mean you understand the code. It means someone put your name in a file once, probably during a reorg.

**Commit messages** are supposed to explain *why* a change was made. In practice, they're cryptic when written by hurried humans and increasingly generic when written by AI. "Refactor authentication module to improve maintainability" tells you nothing about what actually changed, what design decisions were made, or whether the person who wrote the message could explain the code in a whiteboard session.

All of these tools track **authorship** and **routing**. None of them track **endorsement** — the explicit, human assertion: "I have reviewed this code, I understand what it does, and I am willing to be the point of contact when it breaks."

That gap is what the VOUCH framework — and the VOUCH Protocol — aim to fill.

### The VOUCH Framework

**VOUCH** — **V**alidated **O**wnership and **U**nderstanding of **C**ode by **H**umans — is a conceptual framework for tracking human endorsement of code. It's not a standard from a standards body. It's a proposal for how we might start measuring and managing cognitive debt deterministically. Think of it as CODEOWNERS with teeth — granular, automated, AST-aware, and integrated into the development workflow rather than bolted onto it.

The core model rests on a few principles:

**Committer = Author by default.** When you or an agent commits code, git records authorship. Nothing changes here. No new concepts, no new ceremony. The git log works exactly as it always has. VOUCH doesn't replace git — it adds a layer on top.

**Authorship is not endorsement.** This is the key distinction. Code enters the repository *unendorsed* by default. It remains unendorsed until a human explicitly claims comprehension — not just "I looked at it" but "I understand what this does and I'm willing to own it." Endorsement is an affirmative act, separate from committing or reviewing. An agent can author code. A human endorses it — or doesn't.

**Escape hatches for noise.** Not every commit needs endorsement tracking. Formatter runs, CI configuration changes, dependency bumps — these create noise if you track them. The framework uses Conventional Commit prefixes as escape hatches: commits tagged `style:`, `chore:`, or `ci:` bypass endorsement tracking entirely. This prevents the absurdity of "inheriting ownership" of 10,000 lines because you ran `gofmt`.

**Community endorsement as exploration.** Here's where it gets interesting for team dynamics. New team members who explore and understand code can endorse it. A junior engineer who spends a week reading and understanding the payment module can endorse it — saying, in effect, "I own this now. I can be inquired about it." **Onboarding is not dead time — it is debt repayment.** Every hour a new hire spends genuinely understanding a module reduces the team's cognitive debt. That's not a cost center; it's an investment with measurable returns on the dashboard.

**AST-level resilience.** Endorsements must survive cosmetic changes but invalidate on structural ones. If someone reformats a file, renames a variable, or adjusts whitespace, the endorsement should stand — the logic hasn't changed. But if someone (or some agent) modifies the control flow, changes the algorithm, or alters the data model, the endorsement must be stripped. This requires tracking AST hashes rather than raw text. It's the difference between "the code looks different" and "the code *is* different."

**The living heatmap.** Imagine the codebase as a living heatmap of human comprehension. Endorsed areas glow green. Unendorsed areas are red. The heatmap shifts in real time as people join, leave, or as agents modify code. A module glowing green turns yellow when its primary endorser leaves the company. A red module turns green when a new team member finishes their deep-dive and endorses it. This is the dashboard that should sit alongside DORA metrics and uptime monitors — a real-time view of *where human knowledge lives* in your system.

**KPI target anchor.** As a starting point for discussion: aim for less than 20% cognitive debt on Tier-1 critical-path services. Track it over time on engineering dashboards alongside DORA metrics, test coverage, and incident rates. Adjust the target based on your organization's risk tolerance and the criticality of the service. A developer tools microservice can tolerate 60% cognitive debt. Your payment gateway cannot.

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

**It operates on metadata, not actual comprehension.** When someone endorses code, the framework records the endorsement. It cannot verify that the person actually understood what they endorsed. This is a trust-based system — exactly like git commit authorship itself. Nobody verifies that the person who committed the code actually wrote it. We trust the metadata because the alternative (verifying everything) is impractical. The protocol gives you the *mechanism*; the culture gives you the *integrity*.

**Endorsement is self-reported.** There is no comprehension quiz, no verification step, no proof of understanding. Someone can endorse code they don't understand, just like someone can approve a PR they didn't read. The framework provides the *infrastructure* for tracking; the culture provides the *incentive* for honesty.

**It only works if the culture supports it.** No tool can force genuine understanding. If the organization treats endorsement as a checkbox to be gamed — the same way some organizations treat code coverage — the metric becomes meaningless. The framework is a mirror; what it reflects depends on the organization looking into it.

The framework is conceptual. The protocol is formal. But a protocol without tooling is a whiteboard exercise.

---

## 6. A CLI Named `vouch`

This section is deliberately brief. The value of this article lives in Acts 1 through 5 — the problem definition, the conceptual framework, the cautionary tales, the protocol specification. What follows is a reference implementation sketch — a concrete starting point for the VOUCH Protocol. It's imperfect. Build something better.

### How it works

The simplest interaction:

```bash
vouch endorse src/payment/gateway.go
```

You're telling the system: "I have reviewed this file. I understand what it does. I'm willing to be the point of contact." You do this before pushing, as part of your workflow — not as a separate ceremony.

If you don't endorse, nothing breaks. The code ships normally. The committer is recorded as the author (standard git behavior), and the code is flagged as *unendorsed*. That's all. No gates, no blocks — just signal.

Query the debt:

```bash
vouch report src/payment/
# Cognitive Debt: 34% (src/payment/)
# 12 files, 8 endorsed, 4 unendorsed
# 2 endorsements invalidated (structural changes since last endorsement)
```

### Storage: git notes

Endorsement data lives in [git notes](https://git-scm.com/docs/git-notes) — metadata attached to git objects without modifying the commit history. No manifest files cluttering the repo. No merge conflicts from competing endorsement edits. The data travels with the repository.

Each endorsement is stored as a JSON record per the protocol spec. The `ast_hash` is computed via [tree-sitter](https://tree-sitter.github.io/) — a parser generator that supports 40+ languages and produces concrete syntax trees. By hashing AST nodes rather than full trees, cosmetic changes (formatting, whitespace, comment edits) produce the same hash while structural changes (logic, control flow, data model) produce a new one. When the hash changes, the endorsement is automatically invalidated. The code goes back to red on the heatmap.

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

### Agent integration

Agents are first-class participants in the protocol — as authors, never as endorsers:

- **Self-reporting.** An agent that generates code calls `vouch` to register itself as author. This creates a clear signal: the code is AI-generated, distinct from human-authored code that happens to use AI assistance.
- **Endorsement remains human.** Agents don't auto-endorse. They author commits, but the endorsement — the "I understand this and I own it" assertion — remains an irreducibly human act. This is the core invariant of the protocol.
- **Pre-modification risk check.** Before an agent rewrites a module, it queries the cognitive debt of that module. If it's about to rewrite a critical file that no human has endorsed, it flags the risk: "Warning: you're about to modify `src/payment/gateway.go`, which has no human endorser. Proceed?"

### `.vouchignore` — Strategic Ignorance, formalized

Not all code needs human endorsement. The `.vouchignore` file (included in this repository) declares patterns that are excluded from cognitive debt calculation entirely, using standard gitignore syntax. Lock files, generated code, build artifacts, massive test fixtures, vendored dependencies — these are Alien Code by design. They were never meant to be read by humans. Excluding them is not gaming the metric; it's being honest about where human comprehension matters.

The distinction is important: `.vouchignore` is not a loophole. It's a declaration of engineering intent. When your team puts `**/generated/**` in `.vouchignore`, they're saying: "We have decided, as a team, that generated code is outside our comprehension boundary. We trust the generator, not the output." That's a valid engineering decision — as long as it's conscious and documented. The `.vouchignore` file *is* the documentation.

### Agentic-assisted reviews

Looking further ahead, the next frontier isn't agents that *write* code but agents that *explain* it. Imagine an agent whose job isn't to implement a feature but to reverse-engineer the intent of code that another agent implemented — decompiling logic into human-readable explanations, annotating design decisions, surfacing implicit assumptions, pre-digesting comprehension so that humans can endorse faster and more confidently. Agents auditing agents. Not for correctness (we have tests for that) but for comprehensibility.

This is the natural course of this revolution. The same technology that created the comprehension gap will inevitably be part of closing it. But the endorsement — the human assertion of "I understand and I own this" — must remain the irreducible human element in the loop. An agent can help you understand. Only you can decide that you do.

---

## 7. Surfing the Wave Together

The 3 AM reality check is coming for everyone. Not *if* but *when*. A P1 hits a module that nobody on the team has ever reviewed. The on-call engineer stares at 3,000 lines of agent-generated code, and the incident timer is ticking, and Slack is blowing up, and the answer is somewhere in code that no human being has ever understood. MTTR doesn't just spike — it becomes unpredictable, because you can't estimate how long it takes to learn a system you've never seen.

This is not a problem we solve by "trying harder to read PRs." The volume has already exceeded human bandwidth. We don't solve it by banning AI tools either — that ship has sailed, and it shouldn't come back. We solve it by building systems that track human comprehension as a first-class engineering metric, the same way we track test coverage, deployment frequency, and change failure rate.

The conversation is already happening. [Storey](https://margaretstorey.com/blog/2026/02/09/cognitive-debt/) named the debt. [Osmani](https://addyo.substack.com/p/code-review-in-the-age-of-ai) documented the review bottleneck. [Beck](https://tidyfirst.substack.com/p/augmented-coding-beyond-the-vibes) articulated the skill shift. [Karpathy](https://thenewstack.io/vibe-coding-is-passe/) acknowledged the limits of vibes. [Orosz](https://newsletter.pragmaticengineer.com/p/the-future-of-software-engineering-with-ai) mapped the new landscape. The pieces are on the table. What's missing is the tooling to make it operational and the protocol to make it interoperable.

That's what VOUCH is trying to be — not the final answer, but the first formal attempt at an answer. A protocol that says: here's what an endorsement is, here's how it's stored, here's how it's invalidated, and here's how you measure the gap. v0.1. Incomplete by design. Built to be forked.

So here's the call to action, and it's simple:

Try measuring cognitive debt in your own codebase. Pick a critical service. Walk the code. Ask: "Who on this team can explain what this module does at 3 AM?" If the answer is "nobody" or "maybe Sarah, but she's on vacation," that's your cognitive debt, right there. No tooling required — just honest assessment.

Build better tools. The `vouch` CLI outlined above is a reference implementation sketch, not a finish line. If git notes are wrong, use something else. If AST hashing is too coarse, make it finer. If the protocol has blind spots, expose them. Fork the protocol, break it, submit PRs with improvements. Build a VS Code extension that shows the heatmap inline. Build a GitHub Action that gates PRs on cognitive debt thresholds. Build a Grafana dashboard that tracks endorsement coverage alongside uptime. The important thing is that *something* exists — some mechanism for humans to explicitly say "I own this" in a way that's tracked, queryable, and integrated into the development workflow.

Challenge the metric. If you think cognitive debt is measurable but this isn't the right way to measure it, write about that. If you think it's unmeasurable by nature — that any attempt to quantify human understanding is fundamentally flawed — write about that too. Push back. Poke holes. The VOUCH Protocol is v0.1 for a reason: it's designed to be wrong in instructive ways. The worst outcome isn't that we measure it wrong — it's that we don't measure it at all and discover the consequences during a production incident.

But above all: don't let it become another weaponized dashboard number. Don't tie it to performance reviews. Don't create leaderboards. Don't shame teams with high cognitive debt. The moment you do, people will game the metric, and you'll have gained nothing except a false sense of security on top of genuine ignorance — which is strictly worse than just the genuine ignorance.

The debt is already accruing. The question is whether we start measuring it before or after the next `UPDATE users SET role = 'admin'` ships on a Friday.

---

*This article proposes the VOUCH Protocol (v0.1) and the VOUCH Framework (Validated Ownership and Understanding of Code by Humans) as a starting point for managing cognitive debt. The term "cognitive debt" originates from [Kosmyna et al. at the MIT Media Lab](https://www.media.mit.edu/publications/your-brain-on-chatgpt/) (June 2025). [Margaret-Anne Storey](https://margaretstorey.com/blog/2026/02/09/cognitive-debt/) extrapolated it to software engineering (February 2026). Multiple groups converged independently on the same concept. This article's contribution is the VOUCH Protocol: the first formal specification for tracking human endorsement and ownership of code deterministically.*

*If you found this useful, the best thing you can do is challenge it. Open an issue, write a response, build a prototype, fork the protocol. The problem is real. The solution is unfinished.*
