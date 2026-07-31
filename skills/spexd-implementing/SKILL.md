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

- **Given a task reference**, read it directly: `getEntity` (published version;
  the bare reference is enough — no owning feature, no kind) and `readDocument`
  (the LIVE draft — it may be ahead of the published version, and it also returns
  the current lifecycle `status` and published head `version`). `getEntity`
  returns the owning `feature`, which is what `readDocument` needs for a child
  kind, so read it first and pass that through.
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
   markdown link on the reference, e.g. `[TASK-12](https://www.spexd.com/e/TASK-12)`.
2. **Its title**, verbatim.
3. **A concise summary** — two or three sentences on what it covers and what
   "done" means, in your own words rather than a paste of the body.

```markdown
**[TASK-12](https://www.spexd.com/e/TASK-12) — Implement `POST /api/rides`**

Creates the ride-request endpoint and its state machine transitions per
[DES-9](https://www.spexd.com/e/DES-9), validating rider location and payment
method before persisting. Done when the endpoint accepts a valid request,
rejects the three failure cases in AC-4/AC-5, and the state machine is covered
by tests.
```

`viewUrl` comes back from `getEntity` / `getEntities`, `listFeatures`,
`listChildren` and `searchEntities` — take it from the response rather than
composing a URL yourself. Two tools don't return one: `readDocument` (so keep
the link from whichever tool surfaced the entity), and the outstanding-work
tools, whose items carry a `reference` — and that reference *is* the address:
`https://www.spexd.com/e/` + it (`DES-252` → `https://www.spexd.com/e/DES-252`).
Don't build the link out of the `path` of ancestors those items also carry: it
is optional and dropped whenever an ancestor link is missing, and the ancestors
are what the view derives from the leaf rather than part of the address.
If the ask spans several entities — "what's outstanding on FEAT-6" —
give the same link + title + one-line summary for each, then say which one
you're picking up.

Do this even when you're about to start work immediately: it's what lets a
human confirm you're on the right thing before you spend effort on it.

### 3. Gather the context — walk UP the chain

A task on its own is only a definition-of-done. Its *meaning* lives above it.
Before writing code, read the whole ancestor chain so you build the right thing:

- **Design** (`getEntity` / `readDocument`) — the parent. This is where the
  architecture, data model, contracts, algorithms and trade-offs are. **This is
  your primary spec.** A design may fulfil several acceptance criteria; note them.
- **Acceptance criteria** (`getRequirementCriteria`, by `featureRef` +
  `requirementRef` — pass one reference or the whole set; or
  `listAcceptanceCriteria` for a requirement's full set) — the testable
  conditions your implementation must satisfy. These are your acceptance tests in
  prose: your code is done when every one passes.
- **Requirement** (`getEntity`) — the behavioural rule the ACs prove.
- **Feature** (`getEntity`) — the product capability and why it matters; the
  scope boundaries that tell you what *not* to build.

`getEntity` returns the entity's **ordered ancestor chain** alongside it, so
reading the task already tells you the design, requirement and feature above it
— then `getEntities` pulls that whole set in one batched call, mixed kinds and
all. Acceptance criteria are the exception: an AC's reference is numbered within
its requirement rather than org-wide, so it is fetched through its owning
requirement, not by `getEntity`. Use `readDocument` whenever you need the live
draft rather than the last published version. Read enough of the chain that you
could explain *why* the task exists and *how* it's meant to work before touching
code.

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

- **Reads**: `getEntity` returns the last published version; `readDocument`
  returns the LIVE collaborative draft plus the current `status` and published head
  `version`. Prefer `readDocument` when you need the freshest content or the status.
- **`getEntity` addresses any entity by its bare reference** — no kind, no owning
  feature. The type token is already in the reference and a reference is unique
  across the org, so `DES-9` on its own resolves to exactly one entity. The result
  is discriminated by `type` and carries the owning `feature`, the ancestor chain
  and the `viewUrl`. **`getEntities`** is the batched sibling: 1–100 references of
  any mix of kinds in one call, with unmatched references simply omitted from the
  result rather than failing the batch.
- **You generally don't author while implementing.** If you must correct the spec:
  `readDocument`, then one `editDocument` carrying **every** change as an ordered
  `ops` batch — not a call per change — each op targeting `target.find` (exact
  text, occurring exactly once) or, for something with no text to match, a
  `target.handle` from `searchDocument`. If any op fails the whole batch is
  rejected and the document is untouched. Publishing is then **two** calls:
  a content-less `publishDocument` (`baseVersion` from `readDocument`, advisory)
  which writes nothing and returns a cascade proposal plus a one-hour token, and
  `confirmPublish({ token })`, which is what actually writes — always two, even
  when nothing would be invalidated, and not confirming is simply cancelling. A
  confirm rejected as stale means re-propose. See `spexd-authoring` for the full
  authoring workflow.
- **A reference is org-unique, so nothing has to be qualified to read it back.**
  `TASK-991` pasted from a branch name or a PR title resolves on its own through
  `getEntity`. The *document* tools (`readDocument`, `searchDocument`,
  `editDocument`, `publishDocument`) still address by `kind` + `reference` and, for
  child kinds, `featureRef` — take that feature from the `getEntity` result rather
  than hunting for it.
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
