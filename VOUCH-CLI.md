# VOUCH: Protocol & Reference Implementation

*A companion to [Nobody Owns This Code](README.md). This article specifies the VOUCH Protocol (v0.1) and presents a reference implementation: a CLI and an agent skill built on git.*

VOUCH is defined in terms of git. Not because git is the only possible storage — it isn't — but because git is where code lives, where history lives, and where teams already collaborate. A protocol that doesn't travel with the repository isn't a protocol; it's a service dependency.

---

## 1. Implementation Principles

Before the commands and the protocol, a few choices worth stating explicitly.

**CLI-first.** A web app requires infra. An IDE plugin requires marketplace distribution. A GitHub Action runs only in CI. A CLI runs everywhere a developer has a terminal, integrates with any workflow, and can be scripted into hooks without dependencies. The protocol is tool-agnostic — this implementation starts with the simplest thing that gives you the full feedback loop.

**Git notes as storage.** Endorsement records need to travel with the repository, not require external infrastructure, and not create merge conflicts during normal development. [Git notes](https://git-scm.com/docs/git-notes) satisfy all three: they're metadata attached to git objects, they don't appear in commit history or diffs, and multiple writers don't conflict because each note is per-object. Each endorsement is stored as a JSON record per the protocol spec, anchored to a commit hash and scoped to line ranges. A separate `refs/notes/vouch-state` note on HEAD holds the materialized current state — what CI reads. The tradeoffs are real: [git notes](https://tylercipriani.com/blog/2022/11/19/git-notes-gits-coolest-most-unloved-feature/) are git's most underloved feature — they don't auto-push or auto-pull (you need explicit `git push origin refs/notes/*` and corresponding fetch configurations), there's a ~1MB size limit per note per commit object, and notes attach to commit objects rather than file paths, requiring an indexing layer. These are real tradeoffs, not dealbreakers. If you have a better storage mechanism — a sidecar SQLite database, a lightweight service, a custom git ref namespace — build it. The VOUCH Protocol doesn't depend on git notes. It depends on metadata that travels with the repository and doesn't create merge conflicts.

**Aggressive revocation by default.** When structural change invalidates an endorsement, the coverage drops immediately — no grace period, no `awaiting_review` limbo. The endorser discovers it on their next `vouch status`. This matches how developers actually work: you pull, you check what changed, you decide what to act on. Notification-driven collaborative revocation can be layered on top; the CLI implements the immediate strategy and leaves the collaborative layer to web extensions built over VCS primitives.

**`vouch sync` and materialized state.** Individual endorsement records are the source of truth — append-only, one per endorsement act. But computing coverage from raw records requires traversing history. The implementation maintains a materialized state: a pre-computed summary of current endorsement coverage that can be read in O(1). A git hook keeps it fresh:

```bash
# .git/hooks/post-commit  (also post-merge, post-checkout)
vouch sync
```

`vouch sync` rebuilds the materialized state from endorsement records and writes it to `refs/notes/vouch-state`. With this hook in place, coverage is always current — no manual rebuild, no stale CI reads. It's the bridge between the append-only write path and the O(1) read that CI needs.

---

## 2. The VOUCH Protocol (v0.1)

Deliberately minimal, intentionally incomplete, designed to be broken and improved by the community.

### Endorsement Record

The atomic unit of the protocol is the **Endorsement Record** — a structured declaration that a human has reviewed and understood a specific piece of code at a specific point in its structural history.

```
{
  "version":    "0.1",
  "commit":     "abc123def456...",
  "file_path":  "src/payment/gateway.go",
  "endorser":   "mibar",
  "timestamp":  "2026-03-14T16:30:00Z",
  "lines": {
    "endorsed": [[1, 89], [148, 200]],
    "reviewed": [[90, 102]]
  },
  "note":       ""
}
```

Fields:

- **`version`** — Protocol version. Allows forward-compatible evolution.
- **`commit`** — The commit identifier the endorser was positioned at. The anchor: "I reviewed this file as it existed at this commit."
- **`file_path`** — Relative path from repository root.
- **`endorser`** — Human identity. Maps to the git author or a configured alias. Non-human agents MUST NOT appear in this field.
- **`timestamp`** — ISO 8601 UTC. When the endorsement was recorded.
- **`lines`** — Optional. If absent, the full file is endorsed. If present, a map with two keys: `endorsed` (line ranges the endorser vouches for) and `reviewed` (seen but not vouched). Line ranges are inclusive `[start, end]` pairs. Lines not covered by either key are `unknown`. These three states map directly to the framework: `endorsed` = owned, `reviewed` = cognitive debt, `unknown` = alien code.
- **`note`** — Optional free-text. Intended for context: "Endorsed after security audit Q1-2026."

### Operations

The protocol defines four operations:

**`ENDORSE`** — Create or update an endorsement record for a file at the current commit. If the file already has an endorsement by the same endorser, the operation updates the timestamp and commit reference (re-endorsement after changes).

**`RETRACT`** — Voluntarily withdraw your own endorsement. Only the endorser can issue a retraction — it is a personal act, not an administrative one. Used when you determine you no longer understand the code, have changed roles, or want to explicitly signal that your coverage has lapsed. Revocation is reserved for structural code changes; retraction is reserved for the endorser's own decision.

**`TRANSFER`** — The current endorser designates one or more target humans as intended successors for their endorsements on specific files or ranges. Creates a `transfer_requested` record visible in each target's `QUERY` response and status output. Transfer requests do NOT count as endorsements — the targets must actively review and endorse to complete the handoff. A single transfer can split scope across multiple people (an endorsement plan). Transfer requests survive the transferring endorser's departure; they remain actionable even after the originator is no longer on the team.

**`QUERY`** — Retrieve endorsement state for a file, directory, or glob pattern. Returns: current endorsements, pending transfer requests, and whether the code has structurally changed since endorsement. Knowledge concentration metrics are derivable from QUERY results — no separate operation needed.

### Invalidation

One trigger: **structural code change**.

When the logic, control flow, or data model of endorsed lines changes, those endorsements are soft-deleted by the system — **revoked**, not retracted. The record is preserved but marked `revoked: structural_change`. Coverage drops immediately. Endorsements on unchanged lines survive. The endorser discovers the revocation the next time they query endorsement state.

That's it. No grace periods, no timers, no staleness heuristics. Code changes, endorsements drop. The endorser decides what to do about it.

### Cosmetic-Resilient Behavior

What does NOT trigger revocation: formatter runs, whitespace changes, comment edits, file renames, code moving within the project structure. An endorsement survives a `gofmt` pass and survives a file moving from `src/payment/gateway.go` to `pkg/payment/gateway.go`. It does not survive a change to the control flow within the endorsed lines. The implementation is responsible for detecting the difference; the protocol mandates the behavior.

### Escape Hatches

Not all code changes require endorsement re-evaluation:

- **Conventional Commit prefixes.** Commits with `style:`, `chore:`, `ci:`, or `docs:` prefixes bypass endorsement invalidation. The assumption: these commits don't change program logic. If they do, the prefix is wrong — that's a process problem, not a protocol problem.
- **`.vouchignore`** — A file using gitignore syntax that excludes paths and patterns from cognitive debt calculation entirely. Generated code, lock files, vendored dependencies, build artifacts. These are alien code by design. Including them would inflate the metric and dilute its signal.

### Storage Requirements

The protocol is storage-agnostic. Endorsement records can live in git notes, a sidecar file, a database, or a service. The requirements are:

1. Records MUST be versioned alongside the code (or at least linked to specific commits).
2. Records MUST NOT create merge conflicts during normal development workflows.
3. Records MUST be queryable without checking out the full repository history.
4. Records SHOULD travel with the repository (no external service required for basic operation).
5. Actual source code MUST NOT be transmitted to third-party services to compute or store endorsement data. The protocol operates on metadata only: file paths, hashes, identities, timestamps.

### Configuration Contract

The protocol recognizes an implementation-defined configuration file (`.vouchrc` or equivalent) that specifies:

- **Directory scopes** — which paths are subject to endorsement tracking.
- **Exclusion patterns** — paths excluded from coverage calculation, with richer syntax than `.vouchignore`.
- **Threshold definitions** — per-tier cognitive debt and alien code thresholds used by CI gates.

This file SHOULD be committed to the repository and versioned alongside the code it governs.

---

## 3. The CLI

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

## 4. The Agent Skill

The real power isn't the CLI alone — it's the `vouch` skill integrated into agentic coding tools. The agent itself participates in the endorsement workflow, and this participation is central to how VOUCH scales.

### Before Modifying Code

The agent queries the endorsement state for the files it's about to touch. "I'm about to modify `src/payment/gateway.go`. Ana endorsed this file 3 weeks ago. If my changes are non-cosmetic, her endorsement will be revoked — she'll see it in `vouch status` on her next pull. Cognitive debt for the payment service will rise from 18% to 24%, above the 20% Tier-1 threshold. Proceed?"

This is the comprehension feedback loop: the agent knows the human cost of its changes before making them. It can suggest smaller, scoped modifications that avoid revoking endorsements unnecessarily. It can flag when a change would push a tier above its threshold.

### After Generating Code

The agent self-reports as author. The code is flagged as AI-generated, unendorsed. The heatmap updates in real time. No endorsement is created — the write path and the comprehension path remain distinct.

### During Review

The agent helps the human *understand* — not just write. "You asked me to explain this module. Here's what it does, why it's structured this way, and what assumptions it makes." The human reads the explanation, traces the code, and when they genuinely understand it: `vouch endorse`. The agent pre-digests the comprehension work. The human makes the ownership declaration.

This is where the agent skill becomes a force multiplier for debt repayment. A new team member onboarding onto a module doesn't start from zero — the agent walks them through the code, the history, the design decisions. What used to take a week of solo code-reading becomes a structured tutoring session. The endorsement at the end is still the human's — but the path to earning it is dramatically shorter.

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

Jordan left three months ago. Nobody ran `vouch transfer`. `vouch risk` now shows 43% of Tier-1 with no remaining endorser — 6 files where Jordan was the only person who understood the code.

```
$ vouch risk --endorser jordan
FILES WITH NO REMAINING ENDORSER:
  src/billing/reconciliation.go    jordan was sole endorser
  src/core/event_sourcing.go       jordan was sole endorser
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

*This article specifies the VOUCH Protocol (v0.1) and presents a reference implementation companion to [Nobody Owns This Code](README.md), which defines the VOUCH Framework and Ethos. The protocol is implementation-agnostic. This is one answer. There should be many.*
