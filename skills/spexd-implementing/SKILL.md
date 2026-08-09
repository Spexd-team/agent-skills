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

- **Given a task reference**, read it with both `getEntity` and
  `readDocument`, which answer different questions. `getEntity` is
  **content-free** — it gives the type, owning feature, lifecycle status,
  `viewUrl` and the ordered ancestor chain, and never the body.
  `readDocument` is the only tool that returns a body at all: the LIVE draft
  by default (ahead of the published version where there are unpublished
  edits), plus the current `status` and the published head `version`. Both take
  the bare reference and nothing else — no owning feature, no kind — so neither
  has to wait on the other.
- **Given a branch name or PR title**, `resolveGitHubBranch` turns it into the
  entity it names in one hop — no need to parse the reference out yourself. It
  reports an `outcome`: `resolved` (a task, on `task`), `out_of_scope` (the
  deepest thing named isn't a task — e.g. a branch naming only its feature),
  `ambiguous` (a tie, on `tied`), or `not_found`. A miss is an outcome, never
  an error, and the entities it did resolve come back either way.
- **Given a feature/requirement/design and asked for "what's next"**, use
  `getOutstandingWorkForEntity` — the bare reference, whichever of the three it
  is — to list the descendants whose status needs action, then `listChildren`
  on the design's reference to enumerate its tasks. Pick a task that is ready
  to build (see status rules below).
- **Asked for what's next with nothing to anchor on**, browse `listInbox`:
  the org-wide, status-filtered view of everything. Pick a `view` preset
  (`approved` is the one that lists work ready to build; also `all-work`,
  `ready-for-review`, `drafts`, `requires-changes`, `blocked`), narrow with
  `types: ["TASK"]`, and page with the `nextCursor` it returns. Each row
  carries its `viewUrl` and `outstandingCount`. This browses by status;
  `searchEntities` is the one to reach for when you have text to search on.

### 2. Orient — open with the entity, not with the work

Whenever you're asked to pick up, work on, or look at a Spexd entity — a task,
but equally a feature, requirement, AC or design — the **first thing you say
back** is what that entity actually is. Before any plan, any file reads, any
code, lead with:

1. **A link to the entity** — the `viewUrl` from the tool response, as a
   markdown link on the reference, e.g.
   `[TASK-12](https://www.spexd.com/e/TASK-12)`.
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

**An entity's address is its bare reference under `/e/`** — a reference is
org-unique, so `/e/TASK-12` locates it and the navigator derives the ancestors
itself. The older positional form (`/feature/FEAT-3/REQ-7/DES-9/TASK-12`) still
redirects, but don't write new links in it.

`viewUrl` comes back from `getEntity` / `getEntities`, `listFeatures`,
`listChildren`, `searchEntities`, `listInbox`, `resolveEntityReferences` and
`resolveGitHubBranch` — take it from the response rather than composing a URL
yourself. Two tools don't return one: `readDocument` (so keep the link from
whichever tool surfaced the entity), and `getOutstandingWorkForEntity`, whose
items are addressed by their own `reference` like anything else; the optional
`path` array they also carry is the ancestor chain, and its absence means that
row's chain is broken rather than that you can't address it. If the ask spans
several entities — "what's outstanding on FEAT-6" — give the same link + title
+ one-line summary for each, then say which one you're picking up.

Do this even when you're about to start work immediately: it's what lets a
human confirm you're on the right thing before you spend effort on it.

### 3. Gather the context — walk UP the chain

A task on its own is only a definition-of-done. Its *meaning* lives above it.
Before writing code, read the whole ancestor chain so you build the right thing:

- **Design** (`readDocument`) — the parent. This is where the architecture,
  boundaries, data model, contracts, security, configuration, deployment,
  failure modes, testing areas and trade-offs are. **This is your primary
  spec.** `getDesignCoverage` then tells you exactly which acceptance criteria
  it fulfils — that set, not the requirement's whole list, is what this design
  is answerable for.
- **Acceptance criteria** (`listAcceptanceCriteria` for a requirement's full
  set, each with its structured given/when/then and the designs that fulfil it;
  or `getRequirementCriteria` by `requirementRef` plus specific criterion
  `references`) — the testable conditions your implementation must satisfy.
  These are your acceptance tests in prose: your code is done when every one
  passes.
- **Requirement** (`readDocument`) — the behavioural rule the ACs prove.
- **Feature** (`readDocument`) — the product capability and why it matters; the
  scope boundaries that tell you what *not* to build.

**Two tools, two jobs.** `getEntity` is content-free: it gives the shape —
type, status, owning feature, `viewUrl`, and the **ordered ancestor chain** —
so reading the task already tells you the design, requirement and feature above
it without walking up one call at a time, and `getEntities` pulls that whole
set's metadata in one batched call, mixed kinds and all. But no body ever comes
back from it. For the actual spec text, `readDocument` takes **1–10 references
at once**, so the design, requirement and feature are a single call. Read the
live draft (the default) rather than a published version — it's what
collaborators currently see. Acceptance criteria are the exception to bare-
reference addressing: an AC's reference is numbered within its requirement
rather than org-wide, so criteria are fetched through their owning requirement.

`readDocument` also returns each document's `commentThreads`. Skim them before
building: an anchored discussion on the design is often where a caveat that
never made it into the body lives. Pull the ones that look relevant with
`getCommentThreads`.

Read enough of the chain that you could explain *why* the task exists and *how*
it's meant to work before touching code.

### 4. Build

- Implement to the **design's** mechanism and the **task's** definition of done.
  Satisfy **every** acceptance criterion in the design's AC set, golden path and
  edge cases alike.
- **The task tells you what to write; it never contains it.** A task states what
  changes, what good looks like, what tests should exist, what regressions to
  cover and what finished looks like — plus any `**Depends on:**` line naming
  tasks that must land first. Code in the entities is quoted evidence of what
  already exists, not a draft to paste. Writing the implementation is your job,
  in the repository's own idiom and along the grain the design describes.
- **Write the tests the task names**, including the regression cases it calls
  out. The design says which areas of testing prove its acceptance criteria; the
  task says which tests to add. A criterion with no assertion that fails when it
  stops holding is not implemented, however green the suite is.
- **Land it whole.** A task is scoped to merge to `main` on its own without
  leaving dead paths — no half-wired screen or unreachable route. If you find
  you can't finish the slice that way, say so rather than merging the stranded
  half; the task boundary is wrong and belongs back with authoring.
- Follow the repository's own conventions and its `CLAUDE.md` / contributor rules
  — this skill governs *how to use Spexd for context*, not how to write code in a
  given codebase.
- Keep the change scoped to the task. Sibling tasks are separate units of work;
  don't fold them in unless asked.
- **If the design is wrong or silent** on something you need: don't guess an
  architecture. Surface the gap (to the user, or by flagging the design for
  authoring to amend). Update the *design* first, then implement — never the
  other way round. `createCommentThread` on the design is the durable way to
  raise it — anchor the thread to the exact passage with `quotedText` so the
  question sits on the sentence that's wrong, and it stays there for a human
  after your session ends. Comments are accepted on an entity in any status, so
  this works even once implementation has locked the chain and edits are
  refused. You may start and reply to threads; **resolving or closing one is
  human-only**.

### 5. Reflect status — keep the task's lifecycle honest

A task carries an implementation sub-flow that the rest of the chain reads, so
it must move as the work moves:

- **`APPROVED → IMPLEMENTATION_STARTED`** — when building begins. (A task must
  be `APPROVED` before implementation can start; if it isn't yet, it's not ready
  to build — that's a spec/approval gap to raise, not to bypass.) Starting
  implementation `LOCKS` the whole ancestor chain, freezing the spec you're
  building against — which is exactly what you want.
- **`IMPLEMENTATION_STARTED → PR_RAISED`** — when the work moves into PR review.
- **`→ COMPLETED`** on PR merge. This one is **system-driven** and there is no
  manual route to it — never try to set it by hand.

**Name the task in the branch and the PR, and the lifecycle mostly drives
itself.** Where the repository is connected to Spexd, GitHub activity moves the
task for you: a branch naming an `APPROVED` task moves it to
`IMPLEMENTATION_STARTED`, marking its PR ready for review moves it to
`PR_RAISED`, and merging moves it to `COMPLETED` — and a hop that never fired
(a PR opened ready with no branch event seen) is applied on the way rather than
skipped. So the naming convention is not bookkeeping, it *is* the mechanism:

- **Put the reference in the branch name**, e.g.
  `claude/task-12-ride-request-endpoint`. A trailing slug is ignored and the
  scan is case-insensitive; `resolveGitHubBranch` is what reads it.
- **Always title the PR `{reference}: {title}`** — the task's bare reference, a
  colon and a space, then the change description, e.g. `TASK-12: Implement POST
  /api/rides`. Never raise a PR for Spexd-tracked work without the reference
  leading the title: it's what lets a reviewer trace a PR back to its spec from
  the PR list alone, and what the Spexd–GitHub link keys off. Keep it a plain
  unlinked reference (a PR title is plain text) — the markdown link goes in the
  description.
- **Check rather than assume.** The transitions ride on webhook delivery, so
  they can lag or, on an unconnected repository, never arrive.
  `listTaskPullRequests` shows the PRs Spexd has actually observed for a task,
  and `listEntityRepositories` shows whether the task is linked to a repository
  at all. If the task hasn't moved, transition it yourself.

**Transitioning by hand** is `transitionEntityStatuses` — one tool for every
kind, taking a batch of 1–25 `{ reference, targetStatus }` pairs addressed by
bare reference. There is no singular `transitionEntityStatus`. Each subject is
decided independently: the moves that land come back under `transitioned` and
the refusals under `failed` with a reason, so **read `failed`** rather than
assuming success — a task the webhook already moved will refuse the same hop,
which is a no-op worth recognising rather than an error to retry. Only legal
manual transitions are accepted; illegal moves and system-driven statuses are
rejected.

Never edit the task's *content* to record status ("done", "in progress") — status
is the lifecycle field, not the prose. And approval is a **human-only** action:
an agent can never approve a task or move anything to `APPROVED` (this is enforced
server-side; don't try to route around it).

## Operational notes (MCP surface)

- **Reads**: `getEntity` is **content-free** — metadata only, never a document
  body. `readDocument` is the only tool that returns content, and it returns the
  LIVE collaborative draft plus the current `status` and published head
  `version`; pass `version` (`"latest-published"` or a number) to read the
  record instead. `readDocument` batches 1–10 references and every entry carries
  a whole body, so name the documents you'll actually read. It is also the one
  document tool allowed on a frozen (`LOCKED` / `COMPLETED` / `CANCELLED`)
  entity — `searchDocument`, `editDocument` and publishing are all refused
  there, which matters precisely when you're implementing, since starting
  implementation locks the chain above you.
- **`getEntity` addresses any entity by its bare reference** — no kind, no owning
  feature. The type token is already in the reference and a reference is unique
  across the org, so `DES-9` on its own resolves to exactly one entity. The result
  is discriminated by `type` and carries the owning `feature`, the ancestor chain
  and the `viewUrl`. **`getEntities`** is the batched sibling: 1–100 references of
  any mix of kinds in one call, with unmatched references simply omitted from the
  result rather than failing the batch.
- **Reading the history** when you need to know what changed and why:
  `listVersions` gives the entity's timeline — published versions interleaved
  with status transitions and approval decisions, each version carrying a
  `precis` of what it changed — and `compareVersions` gives the difference
  between two published versions directly. Worth a look when a task's design
  was invalidated and re-approved and you need to know what moved.
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
  `getEntity` (or, from the raw branch/PR text, `resolveGitHubBranch`). The
  *document* tools (`readDocument`, `searchDocument`, `editDocument`,
  `publishDocument`), `listChildren`, `transitionEntityStatuses`, `moveEntity`,
  `listVersions` and `getOutstandingWorkForEntity` address the same way — the
  bare reference, no kind and no owning feature. Only a parent is ever named:
  `createRequirement` takes a `featureRef`, `createDesign` a `requirementRef`,
  `createTask` a `designRef`, and `moveEntity` takes the new parent as
  `parentRef`. Acceptance criteria are the exception throughout: `AC-3` is
  numbered within its requirement, so it is always addressed as `requirementRef`
  + criterion reference.
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
3. Confirm it's ready to build (`APPROVED`), and that nothing on its
   `**Depends on:**` line is still outstanding; if either is unmet, surface the
   gap rather than forcing a transition or building out of order.
4. Read the full ancestor chain (design → ACs → requirement → feature) before
   coding — one batched `readDocument` for the bodies, `getDesignCoverage` for
   the exact AC set you owe.
5. Branch with the reference in its name; confirm the task reached
   `IMPLEMENTATION_STARTED`, and `transitionEntityStatuses` it there yourself
   if the GitHub automation didn't.
6. Implement to the design and the task's definition of done; write the tests it
   names and cover the regressions it calls out; satisfy every AC.
7. Raise the PR, titled `{reference}: {title}` (e.g. `TASK-12: Implement POST
   /api/rides`); confirm the task reached `PR_RAISED` (or transition it); link
   the task and any entity you cite from the PR description.
8. If the design proved wrong or incomplete, report it (or fix the *design* via
   authoring) — don't let the code and the spec diverge.
