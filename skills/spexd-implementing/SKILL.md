---
name: spexd-implementing
description: >-
  Implement software tasks using Spexd as the source of context — pull a task
  and its full traceability chain (task → design → acceptance criteria →
  requirement → feature) via the Spexd MCP tools (mcp__Spexd__*), open by
  reporting the entity's link, title and a concise summary, build to that
  spec, and reflect progress back onto the task's lifecycle status. Use when
  asked to pick up, build, or ship work that is tracked in Spexd, or to turn a
  Spexd task into code. The complement of spexd-authoring: authoring writes the
  spec, implementing executes it without changing it.
user-invocable: true
---

# Implementing Spexd tasks

Spexd models a strict traceability chain — **Feature → Requirement → Acceptance
Criteria → Design → Task** — and a **Task** is the leaf: the concrete unit of
implementation work decomposed from exactly one design. This skill is about
*executing* a task well, using everything above it in the chain as context. It
is the mirror of `spexd-authoring`: authoring decides *what* to build and *how*;
implementing carries out a decision that has already been made.

**The one rule that governs everything: implement the spec, don't rewrite it.**
A task executes a design; it never invents architecture. If, while building, you
find the design is wrong, missing, or under-specified, **stop and surface it** —
raise it with the humans or hand back to authoring to fix the *design* — rather
than quietly building something the chain doesn't describe. Code that diverges
from its design breaks traceability, which is the whole point of Spexd.

## When to use this

- "Pick up TASK-12 and implement it."
- "What's the next piece of outstanding work on FEAT-6, and build it."
- "Turn this design's tasks into a PR."
- Any request to build, ship, or code something that is tracked in Spexd.

If instead you're being asked to *write or restructure the spec itself* — add a
feature, split a requirement, draft acceptance criteria — that's
`spexd-authoring`, not this skill.

## The loop

Implementing a task is five steps: **find the work → orient → gather the
context → build → reflect status.** Never skip straight from "find" to
"build" — the context above the task is what makes the implementation correct.

### 1. Find the work

- **Given a task reference**, read it directly: `getTasks` (published version;
  pass one reference or several) and `readDocument` (the LIVE draft — it may be
  ahead of the published version, and it also returns the current lifecycle
  `status` and published head `version`).
- **Given a feature/requirement/design and asked for "what's next"**, use the
  outstanding-work tools — `getOutstandingWorkForFeature`,
  `getOutstandingWorkForRequirement`, `getOutstandingWorkForDesign` — to list the
  descendants whose status needs action, then `listChildren` (`kind: "design"`)
  to enumerate a design's tasks. Pick a task that is ready to build (see status
  rules below).

### 2. Orient — open with the entity, not with the work

Whenever you're asked to pick up, work on, or look at a Spexd entity — a task,
but equally a feature, requirement, AC or design — the **first thing you say
back** is what that entity actually is. Before any plan, any file reads, any
code, lead with:

1. **A link to the entity** — the `viewUrl` from the tool response, as a
   markdown link on the reference, e.g.
   `[TASK-12](https://www.spexd.com/feature/FEAT-3/REQ-7/DES-9/TASK-12)`.
2. **Its title**, verbatim.
3. **A concise summary** — two or three sentences on what it covers and what
   "done" means, in your own words rather than a paste of the body.

```markdown
**[TASK-12](https://www.spexd.com/feature/FEAT-3/REQ-7/DES-9/TASK-12) — Implement `POST /api/rides`**

Creates the ride-request endpoint and its state machine transitions per
[DES-9](https://www.spexd.com/feature/FEAT-3/REQ-7/DES-9), validating rider
location and payment method before persisting. Done when the endpoint accepts
a valid request, rejects the three failure cases in AC-4/AC-5, and the state
machine is covered by tests.
```

`viewUrl` comes back from `get{Entity}`, `listFeatures`, `listChildren` and
`searchEntities` — take it from the response rather than composing a URL
yourself. Two tools don't return one: `readDocument` (so keep the link from
whichever tool surfaced the entity), and the outstanding-work tools, which
return a `path` array instead — that path *is* the URL,
`https://www.spexd.com/feature/` + the path segments joined with `/`
(`["FEAT-15","REQ-64","DES-252"]` →
`https://www.spexd.com/feature/FEAT-15/REQ-64/DES-252`). If the ask spans
several entities — "what's outstanding on FEAT-6" — give the same
link + title + one-line summary for each, then say which one you're picking up.

Do this even when you're about to start work immediately: it's what lets a
human confirm you're on the right thing before you spend effort on it.

### 3. Gather the context — walk UP the chain

A task on its own is only a definition-of-done. Its *meaning* lives above it.
Before writing code, read the whole ancestor chain so you build the right thing:

- **Design** (`getDesigns` / `readDocument`) — the parent. This is where the
  architecture, data model, contracts, algorithms and trade-offs are. **This is
  your primary spec.** A design may fulfil several acceptance criteria; note them.
- **Acceptance criteria** (`getAcceptanceCriteria` — pass one reference or the
  whole set) — the testable conditions your implementation must satisfy. These
  are your acceptance tests in prose: your code is done when every one passes.
- **Requirement** (`getRequirements`) — the behavioural rule the ACs prove.
- **Feature** (`getFeatures`) — the product capability and why it matters; the
  scope boundaries that tell you what *not* to build.

Use the `get{Entity}` tools to pull one or several siblings at once, and `readDocument`
whenever you need the live draft rather than the last published version. Read
enough of the chain that you could explain *why* the task exists and *how* it's
meant to work before touching code.

### 4. Build

- Implement to the **design's** mechanism and the **task's** definition-of-done
  checklist. Satisfy **every** acceptance criterion in the design's AC set, golden
  path and edge cases alike.
- Follow the repository's own conventions and its `CLAUDE.md` / contributor rules
  — this skill governs *how to use Spexd for context*, not how to write code in a
  given codebase.
- Keep the change scoped to the task. Sibling tasks are separate units of work;
  don't fold them in unless asked.
- **If the design is wrong or silent** on something you need: don't guess an
  architecture. Surface the gap (to the user, or by flagging the design for
  authoring to amend). Update the *design* first, then implement — never the
  other way round.

### 5. Reflect status — keep the task's lifecycle honest

A task carries an implementation sub-flow that the rest of the chain reads, so
move it as the work moves. Use `transitionTaskStatus` (only legal manual
transitions are accepted; the server rejects illegal or system-only moves):

- **`APPROVED → IMPLEMENTATION_STARTED`** — when you begin building. (A task must
  be `APPROVED` before implementation can start; if it isn't yet, it's not ready
  to build — that's a spec/approval gap to raise, not to bypass.) Starting
  implementation `LOCKS` the whole ancestor chain, freezing the spec you're
  building against — which is exactly what you want.
- **`IMPLEMENTATION_STARTED → PR_RAISED`** — when the work moves into PR review.
  Reference the PR from the task so code and spec stay linked.
- **`→ COMPLETED`** is **system-driven** on PR merge — you do not set it by hand.

Never edit the task's *content* to record status ("done", "in progress") — status
is the lifecycle field, not the prose. And approval is a **human-only** action:
an agent can never approve a task or move anything to `APPROVED` (this is enforced
server-side; don't try to route around it).

## Operational notes (MCP surface)

- **Reads**: `get{Entity}` returns the last published version; `readDocument`
  returns the LIVE collaborative draft plus the current `status` and published head
  `version`. Prefer `readDocument` when you need the freshest content or the status.
- **`get{Entity}`** takes one or more references and pulls those entities of one
  kind at once (scoped to a feature for child kinds) — pass a single reference to
  read one, or a set to gather an AC set or a design's siblings in one call.
- **You generally don't author while implementing.** If you must correct the spec,
  edits are anchored, not whole-document: `readDocument` → `searchDocument` →
  `insertContent` / `replaceContent` / `deleteContent`, then a separate content-less
  `publishDocument` with the `baseVersion` from `readDocument` (a stale `baseVersion` is
  rejected with a conflict — re-read and retry). See `spexd-authoring` for the full
  authoring workflow.
- **References are feature-scoped** for child kinds: `DES-1` exists under many
  features, so always qualify a child reference with its `featureRef`.
- **Every entity you mention gets a link.** Tool responses carry a `viewUrl`;
  render references as markdown links to it — in your replies, in PR
  descriptions, and in commit messages where a reference appears. A bare
  `TASK-12` is something the reader has to go and find.
- **Never invent references, versions or URLs** — read them from the tool
  responses.

## Process checklist

1. Resolve the task (by reference, or by finding outstanding work).
2. Report it back first: **link (`viewUrl`) + title + concise summary**, before
   any plan or code.
3. Confirm it's ready to build (`APPROVED`); if not, surface the gap rather than
   forcing a transition.
4. Read the full ancestor chain (design → ACs → requirement → feature) before
   coding.
5. `transitionTaskStatus` → `IMPLEMENTATION_STARTED`.
6. Implement to the design and the task's definition-of-done; satisfy every AC.
7. Raise the PR; `transitionTaskStatus` → `PR_RAISED`; link the PR back, and
   link the task (and any entity you cite) from the PR description.
8. If the design proved wrong or incomplete, report it (or fix the *design* via
   authoring) — don't let the code and the spec diverge.
