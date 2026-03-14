# vouch: A Reference Implementation Proposal

*A companion to [Nobody Owns This Code](README.md) — the VOUCH Framework and Protocol v0.1. This article assumes familiarity with the framework and protocol defined there. What follows is one concrete answer to the question: what does a tool built on that protocol actually look like?*

---

## 1. Design Decisions

Before the commands, a few choices worth stating explicitly.

**CLI-first.** A web app requires infra. An IDE plugin requires marketplace distribution. A GitHub Action runs only in CI. A CLI runs everywhere a developer has a terminal, integrates with any workflow, and can be scripted into hooks without dependencies. The protocol is tool-agnostic — this implementation starts with the simplest thing that gives you the full feedback loop.

**Git notes as storage.** Endorsement records need to travel with the repository, not require external infrastructure, and not create merge conflicts during normal development. Git notes satisfy all three: they're metadata attached to git objects, they don't appear in commit history or diffs, and multiple writers don't conflict because each note is per-object. The tradeoffs are real (discussed in Section 3) but manageable.

**Aggressive revocation by default.** When structural change invalidates an endorsement, the coverage drops immediately — no grace period, no `awaiting_review` limbo. The endorser discovers it on their next `vouch status`. This matches how developers actually work: you pull, you check what changed, you decide what to act on. Notification-driven collaborative revocation can be layered on top; the CLI implements the immediate strategy and leaves the collaborative layer to web extensions built over VCS primitives.

---

## 2. The CLI

Writing an endorsement:

```bash
vouch endorse src/payment/gateway.go
```

You're positioned at the current commit. You're telling the system: "I have reviewed this file. I understand what it does. I am responsible for it." Line ranges default to the full file; an agent-assisted review session can produce granular line-level records instead.

If you don't endorse, nothing breaks. The code ships. The committer is recorded as author (standard git behavior), the file is flagged as *unendorsed*. No gates, no blocks — just signal.

**Inspecting ownership — `vouch ls`:**

```
$ vouch ls src/payment/
ENDORSED  REVIEWED  UNKNOWN  FILE                   OWNERS
    87%       8%       5%    gateway.go             mibar, sarah
    45%       0%      55%    processor.go           mibar
     0%      23%      77%    webhook_handler.go     —
   100%       0%       0%    types.go               sarah
```

Defaults to CWD if no path is given. Use `--dir` (`-d`) to scope explicitly. Three columns map directly to the framework: `ENDORSED` (owned), `REVIEWED` (cognitive debt), `UNKNOWN` (alien code).

For line-level detail on a specific file:

```
$ vouch ls -a src/payment/gateway.go
LINES     STATE       OWNERS          AGE
1-89      endorsed    mibar, sarah    3w
90-102    reviewed    sarah           1w
103-147   unknown     —               —
148-200   endorsed    mibar           3w
```

**Scanning coverage — `vouch stats`:**

```bash
$ vouch stats --dir src/ --exclude "**/*.test.go"
Endorsed:        70%
Cognitive Debt:  18%  (reviewed, unendorsed)
Alien Code:      12%  (never seen)
Tier-1 threshold: 20%  ✓

$ vouch stats --config .vouchrc   # use project config for dirs, excludes, thresholds
```

`vouch stats` is the CI-facing command. `vouch ls` is the human-facing command. Same underlying data, different presentation.

**The communication protocol — `vouch status`:**

`git pull && vouch status`. This is the loop.

```
$ git pull
Updating abc123..def456
 src/core/query_optimizer.go  |  47 ++++---
 src/payment/processor.go     |  12 +-

$ vouch status
Your endorsements revoked by recent changes:
  (use "vouch endorse <file>" to re-endorse after reviewing the diff)
  (use "vouch retract <file>" to withdraw your endorsement)

        revoked: src/core/query_optimizer.go    commit abc123  by agent
        revoked: src/payment/processor.go       commit def456  by sarah

nothing else to act on.
```

Same grammar as `git status`: what changed, what you can do about it, the exact commands to do it. The post-merge hook has already run `vouch sync` — by the time you type the command, the state reflects reality.

Nobody touched your endorsements. The code changed under them. Now the decision is yours: re-endorse after reviewing the diff, or retract and let the lines sit as known cognitive debt.

**`vouch retract`** is the personal withdrawal command:

```bash
vouch retract src/payment/processor.go        # full file
vouch retract src/payment/processor.go 45 67  # specific line range
```

Retract is different from revocation: you're not being told your coverage lapsed, you're choosing to end it. The record is preserved with `retracted: true` and the reason is yours to add.

**`vouch transfer`** — designate a successor for your endorsements:

```bash
vouch transfer src/payment/gateway.go --to sarah         # full file to one person
vouch transfer src/auth/ --to sarah --to marco           # directory split across two
vouch transfer src/payment/ --to sarah --to marco \
  --note "sarah: gateway + processor; marco: webhooks"  # with a scope note
```

Transfer creates a `transfer_requested` record that appears in the target's `vouch status`. It does not confer endorsement — Sarah must still review and endorse. The transfer record is the institutional memory: what needs attention, who was asked to look at it, when the outgoing endorser created the request.

**`vouch risk`** — surface knowledge concentration:

```
$ vouch risk
KNOWLEDGE CONCENTRATION (Tier-1 critical path):
  mibar    78%   ⚠ single point of failure
  sarah    31%
  marco    12%

Simulated departure — mibar:
  Cognitive debt: 18% → 52%  on Tier-1
  Files with no remaining endorser: 6
  Pending transfer requests: 0

Run 'vouch transfer <path> --to <person>' to begin a handoff.
```

`vouch risk` answers the question no existing tool asks: what happens to comprehension coverage if this person leaves tomorrow? The concentration percentage is the bus factor made precise.

---

## 3. Storage and Hooks

**Storage: git notes.** Endorsement data lives in [git notes](https://git-scm.com/docs/git-notes) — metadata attached to git objects without modifying the commit history. No merge conflicts. The data travels with the repository. Each endorsement is stored as a JSON record per the protocol spec, anchored to a commit hash and scoped to line ranges. A separate `refs/notes/vouch-state` note on HEAD holds the materialized current state — what CI reads.

**Git hook — keeping the materialized state current:**

```bash
# .git/hooks/post-commit  (also post-merge, post-checkout)
vouch sync
```

`vouch sync` rebuilds the materialized state from endorsement records and writes it to `refs/notes/vouch-state`. With this hook in place, coverage is always fresh — no manual rebuild, no stale CI reads. It's the bridge between the append-only write path and the O(1) read that CI needs.

**Honest about the tradeoffs.** [Git notes](https://tylercipriani.com/blog/2022/11/19/git-notes-gits-coolest-most-unloved-feature/) are git's most underloved feature for good reasons:

- They don't auto-push or auto-pull. You need explicit `git push origin refs/notes/*` and corresponding fetch configurations. This is friction, and friction kills adoption.
- There's a ~1MB size limit per note per commit object. For large monorepos with thousands of files, the storage model needs careful design.
- Notes attach to commit objects, not file paths. Mapping "endorsement of a file" to "note on a commit" requires an indexing layer that doesn't exist in git natively.

These are real tradeoffs, not dealbreakers. If you have a better storage mechanism — a sidecar SQLite database, a lightweight service, a custom git ref namespace — build it. The VOUCH Protocol doesn't depend on git notes. It depends on metadata that travels with the repository and doesn't create merge conflicts.

---

## 4. The Agent Skill

The real power isn't the CLI alone — it's the `vouch` skill integrated into agentic coding tools. The agent itself participates in the endorsement workflow:

**Before modifying code**, the agent queries the endorsement state for the files it's about to touch. "I'm about to modify `src/payment/gateway.go`. Ana endorsed this file 3 weeks ago. If my changes are non-cosmetic, her endorsement will be revoked — she'll see it in `vouch status` on her next pull. Cognitive debt for the payment service will rise from 18% to 24%, above the 20% Tier-1 threshold. Proceed?"

**After generating code**, the agent self-reports as author. The code is flagged as AI-generated, unendorsed. The heatmap updates in real time.

**During review**, the agent helps the human *understand* — not just write. "You asked me to explain this module. Here's what it does, why it's structured this way, and what assumptions it makes." The human reads the explanation, traces the code, and when they genuinely understand it: `vouch endorse`.

---

## 5. Ana's Monday Morning

Ana opens her terminal and pulls.

```bash
git pull
vouch status
```

Two files come back: `src/core/query_optimizer.go` revoked three days ago by an agent, `src/payment/processor.go` revoked this morning by a teammate. She runs `vouch stats --config .vouchrc` — core data is at 31%, above the 20% Tier-1 threshold. That's the one that matters.

She runs `vouch ls -a src/core/query_optimizer.go` to see which line ranges went dark. She opens the file, asks her coding agent to walk her through what changed: "The refactor replaced the nested loop join selection with a cost-based optimizer. Core logic changed in three functions. Here's what each one does and why."

Ana reads the explanation, traces the code, runs the tests herself. She runs `vouch endorse src/core/query_optimizer.go`. `vouch stats` updates: core data 19%. Below threshold. Green.

It's 10:15 AM. She moves on to feature work. No PR queue. No blocked teammates. No Slack escalations.

---

## 6. Configuration and CI/CD

The `.vouchrc` defines directory scopes, exclusions, and thresholds in one place:

```yaml
# .vouchrc
tiers:
  - name: critical
    paths: [src/payment/, src/auth/]
    max_cognitive_debt: 20%
    max_alien_code: 5%
  - name: business
    paths: [src/]
    max_cognitive_debt: 40%
    max_alien_code: 20%

exclude:
  - "**/*.test.go"
  - "**/generated/**"
```

CI runs `vouch stats --config .vouchrc`. If any tier exceeds its threshold, the pipeline fails. The PR author sees: "This change introduces 450 lines of unendorsed code in a Tier-1 service. Cognitive debt: 23% (threshold: 20%). Alien code: 7% (threshold: 5%). Endorse or request endorsement before merging."

The `.vouchignore` file covers patterns excluded entirely from the cognitive debt calculation using standard gitignore syntax. If `.vouchrc` already has an `exclude` section, `.vouchignore` is redundant; keep whichever convention fits your team.

---

## 7. Knowledge Transfer Workflows

Two scenarios. One is a planned handoff. The other is damage control.

### The happy path: a departing endorser

Mibar is leaving the team in two weeks. He runs `vouch risk` to see what he owns on the critical path. 78% of Tier-1. He uses `vouch transfer` to create an endorsement plan:

```bash
vouch transfer src/payment/ --to sarah \
  --note "payment gateway: the retry logic on lines 45-89 is the tricky part, read the 2024 incident postmortem first"

vouch transfer src/auth/ --to marco \
  --note "auth module is stable, main risk is the token refresh edge case in jwt.go lines 112-130"

vouch transfer src/core/query_optimizer.go --to sarah --to marco \
  --note "split review: sarah takes cost-based optimizer (lines 1-200), marco takes join selection (201-400)"
```

Sarah and Marco each see the transfer requests on their next `vouch status`. The notes from Mibar are preserved — institutional memory, not just file paths. Over the next two weeks they review, ask Mibar questions while he's still reachable, and endorse. By his last day, `vouch risk` shows no single point of failure.

### The sad path: the endorser is already gone

Jordan left three months ago. Nobody ran `vouch transfer`. `vouch risk` now shows 43% of Tier-1 with no remaining endorser. `vouch status` shows a list of `at_risk` endorsements.

```
$ vouch risk --endorser jordan
COVERAGE LOST SINCE jordan's DEPARTURE:
  src/billing/reconciliation.go    no remaining endorsers   (jordan only)
  src/core/event_sourcing.go       no remaining endorsers   (jordan only)
  src/payment/gateway.go           sarah still endorses     (lines 1-89 only)
  ...4 more files
```

The recovery workflow:
1. `vouch ls -a src/billing/reconciliation.go` — see which line ranges went dark
2. Ask the agent to explain what Jordan understood: "explain src/billing/reconciliation.go — what does this module do, why is it structured this way, what are the edge cases?"
3. The agent analyzes the code, the git history, and any available context — not because it endorses, but because it pre-digests the comprehension work
4. The human reads the explanation, traces the code, satisfies themselves that they understand it
5. `vouch endorse src/billing/reconciliation.go`

The saddest thing about alien code is that it's recoverable — it just costs time. The agent skill turns a cold re-endorsement into a structured tutoring session. Jordan's code didn't disappear; it just lost its human champion. The heatmap shows where to start.

---

## 8. What's Missing

This is v0.1. Things that should exist but don't yet:

**Editor integration.** A VS Code extension that shows the heatmap inline — red/yellow/green per file in the explorer, line-level annotations in the gutter. `vouch endorse` as a right-click action after reviewing a diff.

**GitHub Action.** A ready-made Action that runs `vouch stats --config .vouchrc` in CI and posts a coverage delta as a PR comment: "This PR reduces endorsed coverage in `src/payment/` from 87% to 72%. 3 files need endorsement."

**Grafana dashboard.** Cognitive debt and alien code % tracked over time alongside DORA metrics. The trend line matters more than the current value.

**AST-aware revocation.** This implementation uses line ranges anchored to a commit — any change to an endorsed line triggers revocation. A smarter implementation would use AST-based structural comparison to distinguish "this function's logic changed" from "this function was reformatted." [Tree-sitter](https://tree-sitter.github.io/) supports 40+ languages and would be the natural choice.

**Monorepo scaling.** Git notes with a ~1MB limit per object work fine for small repos. A monorepo with thousands of endorsed files needs a different storage strategy — likely a sidecar SQLite database or a dedicated git ref namespace with indexed records.

**Collaborative revocation.** The CLI implements aggressive immediate revocation. The next layer is notification-driven: when your endorsement is revoked, a review request is automatically opened in your PR queue. This lives naturally as a GitHub App or GitLab integration.

---

*This article is the reference implementation companion to [Nobody Owns This Code](README.md), which defines the VOUCH Framework and Protocol v0.1. The framework is implementation-agnostic. This is one answer. There should be many.*
