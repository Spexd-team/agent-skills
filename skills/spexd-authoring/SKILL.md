---
name: spexd-authoring
description: >-
  Author and curate the whole Spexd traceability chain — features,
  requirements, acceptance criteria, designs, and tasks — via the Spexd MCP
  tools (mcp__Spexd__*). Use when asked to populate Spexd, add or reorganise
  any entity, or import work from tickets/docs/code. Encodes the
  altitude/classification lens so each entity lands at the right level and
  implementation detail never leaks upward into a feature or requirement.
user-invocable: true
---

# Authoring the Spexd traceability chain

Spexd models a strict traceability chain: **Feature → Requirement →
Acceptance Criteria → Design → Task**. Getting content onto the right level
matters more than wording — classify first, write second. A misplaced
entity can be retired (cancelled), but retirement is irreversible, so it's
still cheaper to place things correctly the first time.

Each level answers a different question, and the answers get more concrete as
you descend:

| Level | Answers | Altitude |
|---|---|---|
| **Feature** | *What can someone do, and why does it matter?* | Product scope |
| **Requirement** | *What must be true within that capability?* | Behavioural rule |
| **Acceptance Criteria** | *How will we know it's true?* | Testable condition |
| **Design** | *How do we build it so those conditions hold?* | Architecture & mechanism |
| **Task** | *What work ships a slice of that design?* | Unit of work |

Put crudely: **Feature and Requirement are the *what* and the *why*; Design and
Task are the *how*.** Acceptance criteria sit on the hinge — still *what*, but
stated precisely enough to be proved.

## The one rule that governs everything: detail only descends

Implementation detail is introduced **at the Design level and nowhere
above it.** A feature or a requirement or an acceptance criterion describes
*what* and *why* in product terms; the Design entity is the first place that
decides *how* — and "how" here means **architecture**, not just UX. Data
models, transactions, concurrency, failure handling, and external services
all belong in Design, not only screen layouts.

So, above Design, keep out:

- **Vendors and products** — Clerk, Postgres, Redis, Railway, Stripe,
  Hocuspocus, an SMS provider. A capability is the requirement; the product
  that delivers it is design.
- **Libraries, frameworks, and technologies** — RLS, PgBouncer, Yjs, zod, H3
  cells, WebSockets.
- **Tables, columns, schemas, keys, indexes.**
- **Algorithms and data structures** — a reference-counter, a CAS on a
  version column, a finite-state machine, an outbox.
- **Code, interfaces, type shapes, HTTP contracts** — no `interface`, no
  route signatures, no request/response bodies.
- **Effort and sizing in engineering terms** — lines of code, story points,
  which service or repo it lands in.

If a technology name shows up in a feature or requirement title or body, the
entity is misplaced: the capability is the entity, the technology is a Design
detail. Worked example: *"Organization Data Isolation (Postgres RLS)"* was
raised as a feature, then correctly reworked as the requirement *"Organization
Data Isolation"* under the Authentication & Organizations feature, with RLS
demoted to a design footnote.

Measurable **thresholds** are the exception and are welcome above Design —
"within 1 second", "± 10% of final fare", "at least every 5 seconds". A
threshold is an observable outcome, not a mechanism. "Updates every 5 seconds"
is a requirement/AC statement; "polls an H3 index every 5 seconds" is design.

### What belongs where — at a glance

| Content | Feature | Req | AC | Design | Task |
|---|:---:|:---:|:---:|:---:|:---:|
| Product problem, value, persona, metrics | ✓ | | | | |
| Behavioural rule ("the rider can…", "every X must Y") | | ✓ | | | |
| Single testable pass/fail scenario | | | ✓ | | |
| Measurable threshold (500 ms, ±10%) | goal-level | ✓ | ✓ | ✓ | ✓ |
| Vendors / libraries / frameworks | ✗ | ✗ | ✗ | ✓ | ref |
| Data model, as a field/type/description table | ✗ | ✗ | ✗ | ✓ | ref |
| Endpoints and contracts, as a table | ✗ | ✗ | ✗ | ✓ | ref |
| Algorithms / data structures | ✗ | ✗ | ✗ | ✓ | ref |
| Security, configuration, deployment controls | ✗ | ✗ | ✗ | ✓ | ref |
| Existing code, quoted as evidence of the approach | ✗ | ✗ | ✗ | ✓ | ✓ |
| **New implementation code** | ✗ | ✗ | ✗ | ✗ | ✗ |
| Areas of testing that prove the covered ACs | ✗ | ✗ | ✗ | ✓ | ✓ |
| Architecture diagram | ✗ | ✗ | ✗ | ✓ | |
| Trade-offs / alternatives considered | ✗ | ✗ | ✗ | ✓ | |
| Definition of done, regressions, blocking tasks | | | | | ✓ |

Each entity may **reference** the level above ("see FEAT-2", "satisfies
AC-3" — always as a link to the entity, see *Content conventions*); none may
pre-empt the level below. A requirement that already names
the algorithm has done Design's job for it — and usually the wrong way.

### The one thing that belongs at no level: the implementation itself

Detail descends, but it stops short of the code. **No entity — not even a
design or a task — contains the implementation code.** Writing that code is
the work the design and the task exist to commission; an entity that already
contains it has done the engineer's job for them, badly and without a compiler.

The test is direction, not syntax: **could an engineer paste this into the
repository as their implementation?** If yes, it doesn't belong. **Quoting code
that already exists is a different act and is welcome** at the Design and Task
levels — it is how a design shows the pattern it is conforming to and how a
task points at the thing that has to change. So a five-line excerpt of today's
repository layer, cited to its file, is good authoring; the new handler written
out ready to paste is not.

Where a contract or a schema needs pinning down precisely, express it as a
**table** — field, type, description — not as a type declaration, migration or
request body. That form is readable by the product members of the audience too,
which is why designs use it (see *Design*).

## Feature

**What it is.** The root of the chain: a top-level unit of product scope —
a surface, workflow, or capability a user (or agent) can point at or demo.
Ride booking, the spec navigator, the MCP server, commenting, GitHub
integration. Has no parent.

**Does this belong as a feature?** Ask: *could someone demo it or point at
it as a thing they can do?* If yes, feature. If it instead reads as a rule
*within* a capability ("every edit is a new version", "no data leaks across
orgs"), it's a requirement, not a feature. Two common misfires:

- **Cross-cutting concern raised as a sibling feature** — versioning, audit,
  numbering apply to every entity. These are requirements under the core
  model feature, not features of their own.
- **A behaviour of an existing surface raised as a new feature** — "create
  entities from the navigator" is a requirement under the navigator's
  feature.

When genuinely torn between feature and requirement, prefer **requirement**:
a requirement under a coarse feature is easy to live with; a stray top-level
feature adds noise until someone cancels it.

**How to write a good one.** Frame the problem and the value in product
language. A strong feature body covers: a one/two-sentence overview of the
capability and why it matters; goals and explicit non-goals; the primary
persona(s); scope boundaries (in scope / out of scope); and success metrics
where they exist. Write for a reader deciding *whether* to build it, not
*how*.

**Keep out.** Everything under "detail only descends" — no vendor, library,
table, algorithm, code, or architecture. Also no Given/When/Then acceptance
phrasing (that's an AC). A feature that mentions how it's built is doing three
levels' jobs at once.

---

## Requirement

**What it is.** A rule that must hold within exactly one feature. Reads as
"the rider can request a ride now", "every edit is a new immutable version",
"no data leaks across organizations". It constrains or specifies its feature
and can't be demoed on its own.

**Requirement vs Feature.** If it can't stand alone as a demoable capability
and instead qualifies one, it's a requirement. See the Feature section's
misfires — cross-cutting and surface-behaviour items land here.

**Requirement vs Acceptance Criteria.** The requirement states the rule in
prose; the AC is a *specific, testable scenario* that proves it. If what
you've written has a concrete precondition, a concrete action, and a single
observable outcome you could write a pass/fail test from, it's an AC, not a
requirement. "The rider can tip after a trip" is a requirement; "*Given* a
completed trip within the tipping window, *when* the rider adds a tip, *then*
the tip is charged as a separate transaction to the default method" is one of
its ACs.

**How to write a good one.** User stories ("as a rider standing on a corner,
I want… so that…"), scope in/out, and the behaviour described in product
terms. Note origin when extracted from another entity ("Formerly tracked as
feature FEAT-n"). Still no mechanism.

**Keep out.** Same prohibitions as a feature. In particular, no code or type
shapes: if you find yourself pasting an `interface` or an API body to pin
down "the request shape", stop — that contract is Design. (Seed/demo content
sometimes carries such shapes to exercise the renderer; don't treat it as a
model for real authoring.)

---

## Acceptance Criteria

**What it is.** A single testable condition that defines when its requirement
is satisfied. One AC = one scenario. The house style is **Given / When /
Then**: `**Given** <precondition>, **when** <action>, **then** <observable
outcome>.` Belongs to exactly one requirement.

**AC vs Requirement.** An AC is concrete and verifiable where the requirement
is general. If you cannot derive a pass/fail test from the sentence — because
it lacks a specific trigger or a specific observable result — it's still a
requirement. Measurable thresholds belong here ("within 500 ms", "within 1
second").

**AC vs Design.** An AC states what must be *observable*, never *how* it's
achieved. "The fare estimate updates within 500 ms" is an AC; "…by swapping
the carried estimate id locally against an H3 pricing cell" is design. If a
technique appears, move it down.

**How to write a good one.** Keep each AC atomic — one precondition, one
action, one outcome. Split any "and also…" into a second AC. Cover the golden
path *and* the important failure/edge paths as **separate** criteria (valid
request; no location permission; no payment method; each its own AC). Prefer
measurable outcomes over adjectives ("within 30 seconds", not "quickly").

**Keep out.** Mechanism and implementation. Thresholds yes; techniques no.

---

## Design

**What it is.** The first level where implementation lives — a technical
design/decision that fulfils one or more acceptance criteria. This is where
architecture, vendors, libraries, data models, algorithms, state machines,
contracts and trade-offs are not just allowed but expected. It answers *how do
we build this so the ACs hold?* — while stopping short of writing the code that
answer commissions.

This is **architectural design, not merely UX.** A design that only describes
screens and never the system beneath them — the data model, the transaction
boundaries, the concurrency and failure handling, the external services — is
incomplete. Screen layout can be part of a design, but it is never the whole
of one.

**Who reads it.** Two audiences at once: an engineer who will build from it,
and a product member who will not. That is the reason the mechanism is written
as prose and tables rather than as code — a data model given as field, type and
purpose is legible to both, where a migration or a type declaration is legible
to one. Write so a product reader can follow what the system does and why, and
an engineer can tell what has to change.

**A design's structural parent is its requirement.** `createDesign` takes a
`requirementRef`, and the owning feature comes from it. A design fulfils only
**its own requirement's** acceptance criteria — `addDesignCoverage` rejects a
criterion owned by any other requirement, and `moveEntity` onto a different
requirement drops the coverage the design can no longer hold (returned as
`droppedCoverage`, so you can see what the move cost).

Within that requirement **the AC↔Design link is many-to-many** (the only
fan-out/fan-in point in the chain). One design can fulfil several of its
requirement's ACs, and one AC can need several designs (a frontend design *and*
a backend design each cover part of it). So split designs along **architectural
seams**, not one-per-AC by reflex. Shared infrastructure serving several
requirements is a design under each — coverage never crosses a requirement
boundary.

**Design vs the levels above.** If you're naming a technology, drawing an
architecture diagram, tabulating a data model, or choosing between approaches —
you're in Design. Reference the ACs you satisfy; don't restate them in full.

**Design vs Task.** A design *decides* the approach; a task *executes* a
decision already made. A design describes the **target state of the
capability**; a task describes **a change** that moves the code toward it. Don't
decompose a design into a checklist of work here — that's the Task level.

**Ground it in the code, and follow the grain.** A design is bound by code
analysis: read the code that will host the capability before writing it, and
reflect what you find. The point is not only accuracy but **conformance** — a
design adopts the approach the codebase already takes. If business logic lives
in the repository layer behind thin controllers, the design puts its logic in
the repository layer; describing a logic-heavy controller instead is wrong here
even where it would be defensible in a greenfield codebase. Quote the existing
code where it pins the pattern down, and name the modules and layers involved.

Grounding a design in what exists does not turn it into a description of what
exists. It still states the capability's target state in the timeless present
(see *Keep out*) — never a diff, a migration path from today, or a note on how
much of it is already built. Getting there is the Task's job.

**What a good design covers.** Not every design needs every section — one that
adds no data and no endpoint shouldn't invent them — but a section should be
absent because it doesn't apply, not because nobody thought about it.

- **The decision and the mechanism.** What is being built and how it works,
  end to end, in enough detail that an engineer knows what has to change.
- **Boundaries.** How the functionality splits across the system's seams —
  services, layers, clients, and **repositories where more than one is
  involved**. A design spanning three repos says which part lands in each; that
  split is what its tasks decompose along. Link every repository it touches
  with `linkEntityRepository` (see *Operational notes*).
- **Data model.** Every entity it adds or changes, as a table — **field, type,
  description/purpose** — one row per field. Human-readable types (`uuid`,
  `timestamp`, `text`, `enum: draft | published`), not a schema dump.
- **Contracts.** Endpoints, events and interfaces as a table — method and path
  (or event name), purpose, request fields, response, error cases. Not a
  request body pasted as JSON.
- **Security controls.** Authentication and authorization, tenancy isolation,
  which data is sensitive and how it is protected, what an untrusted caller can
  and cannot reach.
- **Configuration controls.** Flags, environment variables and settings the
  behaviour depends on, their defaults, and who can change them.
- **Deployment.** Rollout and migration ordering, backwards compatibility,
  anything that has to ship or be enabled in a particular sequence.
- **Failure modes** and how each is handled.
- **Testing.** The areas of testing required to show the **acceptance criteria
  this design covers** are met — each linked, and each answered with the *kind*
  and *level* of test that would prove it (unit at this seam, integration
  across that boundary, an end-to-end path), plus the edge and failure cases
  that need their own coverage. Not test code, and not a restatement of the AC:
  the useful sentence names what an assertion would have to fail on if the
  criterion stopped holding.
- **Trade-offs and alternatives considered**, and the ADRs and tickets behind
  them.
- **A diagram**, where the architecture is easier seen than read. A diagram is
  an image in the body — upload it with `createAttachmentUpload` (see
  *Operational notes*) and embed the returned URL, rather than settling for
  ASCII art.

**Keep out — four things that creep into designs and don't belong.**

- **The implementation code.** A design commissions code; it doesn't contain
  it. No handler, migration, type declaration or query written out ready to
  paste — express the shape as a table instead. Quoting code that *already
  exists*, cited to its file, is the opposite move and belongs here (see *The
  one thing that belongs at no level*).
- **Delivery or build status.** A design describes the intended
  architecture, not how far it has been built. No "not built", "shipped",
  "in progress", "landed in wave N", no "Status:" note, and no references to
  what `main` currently does or doesn't do — the entity's Spexd lifecycle
  status carries all of that. Write the mechanism in the timeless present, as
  the design to build to, so it reads the same whether the work is unstarted
  or long shipped.
- **Source-code files in the sources line.** Cite sources as the documents,
  ADRs, tickets, and links a reader navigates to for intent and decisions —
  not a list of implementation files (`server/…/foo.ts`). A file path is an
  implementation pointer that goes stale, not a source of the design. (Naming
  a specific file *inside the body* to pin a mechanism is fine — the
  prohibition is on the sources line.)
- **Open questions you never asked.** An "Open questions" section is the
  easiest place to park a decision instead of making one, and it's designs
  that attract them. Before any such question goes into the document, **ask
  the user driving you** — most are answerable in a sentence, and the answer
  belongs in the design as a decision, not in a list of things nobody
  decided. Collect them and ask in one batch rather than trickling them out.
  A question survives into the entity only when the user couldn't resolve it
  either — then it's a real record of an undecided point, and say whose call
  it's waiting on. Writing a design "with open questions to follow up" without
  having put them to the user first is not allowed, however tidy the list looks.

---

## Task

**What it is.** A concrete unit of implementation work decomposed from
exactly one design. It answers *what does someone actually do to ship a slice
of that design?* **Change-level and single-repository**: one reviewable change,
in one codebase.

**Task vs Design.** A task never decides architecture — it carries out what
the design specified. Where the design is the capability's target state, the
task is the change that moves the code toward it. If, mid-task, you discover the
approach is wrong or under-specified, update the **design**, don't quietly
invent new architecture in the task.

**One repository, and say which.** Link it with `linkEntityRepository` — a task
carries **exactly one** repository, and linking a second replaces the first, so
set it once at creation. A design that spans repositories decomposes into
tasks along that boundary: one task per repository, never one task reaching
across two.

**Scope it so it can merge.** Size a task so that, once complete, it can be
merged to `main` on its own **without leaving dead paths behind** — no
half-wired screen, no route nothing reaches, no flag protecting an unfinished
journey. If a slice of the design can't be merged safely alone, it is the wrong
slice: redraw the boundary, or fold it into the task that completes the path.
This constrains decomposition more than size does — prefer a larger task that
lands whole over two that leave the UI stranded between them.

**Blocking comes first.** If a task can't start until other tasks are complete,
say so on the **first line of the body**, before the sources line and everything
else, so ordering is visible without reading the task:

```markdown
**Depends on:** [TASK-14](https://www.spexd.com/e/TASK-14), [TASK-15](https://www.spexd.com/e/TASK-15)
```

Linked references, never bare. Omit the line entirely when nothing blocks the
task — don't write "Depends on: none". This is a prose convention: a task has no
dependency field, and it is unrelated to `listInbox`'s `blocked` view, which
means *invalidated by an upstream publish*, not *waiting on a sibling task*.

**How to write a good one.** An imperative title ("Implement the ride-request
endpoint"). Like its design, a task is **bound by the code**: read what is
there and be explicit about how it has to change. The bar is that an engineer
can start from the task and its design without further analysis beyond
verifying and understanding the existing code. Cover:

- **What changes** — the modules, layers and files that have to change, and in
  what way. Quote the existing code where it makes the change unambiguous.
- **What good looks like** — the behaviour once the change lands, tied to the
  acceptance criteria it serves.
- **What tests should exist** — the tests to add or extend, and what each has
  to assert. The design names the areas of testing; the task names the tests.
- **Regressions to cover** — existing behaviour this change could break, and
  what must keep passing. Call these out explicitly rather than trusting the
  suite to notice.
- **What finished looks like** — the definition of done and a subtask
  checklist, including the merge-safety condition above.
- **References** — the design it implements, as a link ("per [DES-9](…) —
  *Ride request creation & state machine*"), and the acceptance criteria it
  contributes to, linked in the `[AC-3](…/e/REQ-4?ac=AC-3)` form. Reference
  them; don't restate them in full.

**Keep out.** Don't re-argue the design; reference it. Don't introduce
architecture that isn't already in the design. And **no implementation code** —
quoting what exists to show what has to change is right; writing out the code
the task exists to commission is not.

---

## Content conventions (all entities)

- **Don't repeat the title in the body.** The title is a separate field in
  Spexd, shown above the content — starting the body with an `# <Title>`
  heading just duplicates it. Begin the body with the sources line (below),
  then the content. A task with blockers is the one exception: its
  `**Depends on:**` line comes first, above the sources line.
- **Don't indicate status in the content.** An entity's state lives in its
  Spexd lifecycle status field (managed via `transitionEntityStatuses`), not in
  the body — no "Status:" line, no "shipped / planned / rejected" labels in the
  prose. Describe what the entity *is*, not what state it's in.
- **Resolve open questions with the user before writing them down.** This
  bites hardest in designs (see *Design → Keep out*) but holds for every
  entity: an unanswered question in an entity's body is a decision you didn't
  make and didn't ask about. Put it to the user first; only what they also
  couldn't settle stays in the document as an open question.
- **Cite sources as links.** Lead with a sources line — Linear URLs for
  tickets, repo links or paths for ADRs — so a reader can navigate straight
  to the source; quote key phrases where they capture intent.

  ```markdown
  **Sources:** <Linear ticket(s)>, <ADR(s)>
  ```
- **Always link a cross-reference, never leave it bare.** Whenever you name
  another entity — in an entity's body, in a comment, or in what you report
  back to the user — write it as a markdown link to that entity's `viewUrl`,
  not as plain text:

  ```markdown
  satisfies [DES-9](https://www.spexd.com/e/DES-9)
  ```

  **An entity's address is its bare reference under `/e/`**, not a positional
  path: a reference is org-unique, so `/e/DES-9` locates it and the navigator
  derives the ancestors itself. The older positional form
  (`/feature/FEAT-3/REQ-7/DES-9`) still redirects, but don't write new links in
  it.

  `getEntity` / `getEntities`, `listFeatures`, `listChildren`,
  `searchEntities`, `listInbox`, `resolveEntityReferences` and
  `resolveGitHubBranch` all return `viewUrl` alongside the reference — take the
  URL from the response rather than composing one yourself, and if you don't
  have it, read the entity rather than guessing. Two tools don't return one:
  `readDocument` (keep the link from whichever tool surfaced the entity) and
  `getOutstandingWorkForEntity`, whose items are addressed by their own
  `reference` like anything else. Those items also carry an optional `path`
  array of the ancestor chain; it is dropped when an ancestor link is broken,
  so read its absence as "this row's chain is broken" rather than as the
  address being unavailable.

  An acceptance criterion is the one exception: its reference is numbered
  within its requirement, so `AC-3` is not addressable on its own and is
  linked as a query on its requirement's URL —
  `[AC-3](https://www.spexd.com/e/REQ-4?ac=AC-3)`.

  A bare `AC-3` or `DES-9` makes the reader go and find it. Link it.
- **For features and requirements**, a useful body shape is: `## Overview`
  (one or two sentences, product language) → behaviour/scope in product terms.
  Keep the mechanism out entirely rather than giving it a section — the "how"
  belongs in the Design entities the chain links automatically, not in a
  footnote on the feature or requirement. ACs are usually a single
  Given/When/Then line; designs are full technical documents written for
  engineers and product readers alike; tasks state what changes, what good
  looks like, what tests should exist and what finished looks like.
- **Express structure as tables, not as code.** Data models, endpoint and event
  contracts, configuration and permission matrices all read better — and to a
  wider audience — as a table with a row per field and a plain-language
  description than as a schema, type or payload. See *The one thing that
  belongs at no level*.

## Operational notes (MCP surface)

- **`readDocument` is the only way to get a body at all.** Every other tool —
  `getEntity`, `getEntities`, `listChildren`, the `create*` tools, the publish
  and transition responses — is **content-free**: it carries the entity's
  metadata (type, status, owning feature, `viewUrl`, ancestors) and never its
  document. So don't reach for `getEntity` to "read" an entity; it tells you
  where a thing sits, not what it says.
  - It reads the **live draft** by default, unpublished edits included, which
    is what you want before editing. Pass `version` to read the record
    instead: `"latest-published"`, or a version number.
  - It **batches 1–10 references**, and every entry carries a whole body — so
    name the few documents you actually need rather than sweeping a subtree.
  - It is the one document tool allowed on a frozen (`LOCKED` / `COMPLETED` /
    `CANCELLED`) entity; `searchDocument`, `editDocument` and publishing are
    all refused there. Treat what you read as reference material.
  - It also returns `commentThreads` — every open and resolved thread, each
    with its `threadId`, status, anchored text and a `location.handle` you can
    pass straight to an `editDocument` op's `target.handle` without searching.
- **Editing is targeted and batched, not whole-document.** Entities are
  edited through the live-document tools, never by submitting a full body.
  Read first, then send **every** change for that document as a single
  `editDocument` call carrying an ordered `ops` array (1–50). Explicitly *not*
  one call per change. Your edits appear live to anyone editing the entity and
  merge with their concurrent changes.
  - **Target by text first.** An op names where it acts with `target.find`:
    the exact text to act on, which must occur **exactly once** in the
    document — include surrounding words when a short phrase repeats.
  - **`searchDocument` is the fallback, not a step in the chain.** It returns
    match handles for `target.handle`, which is *required* for anything with
    no text to match — an image, an attachment, a horizontal rule, an
    `@`-mention, a `#`-entity-reference, an empty bullet or table cell — and
    for any target you can't name uniquely as text. A stale handle (what it
    anchored changed under you) is rejected: re-run `searchDocument` and retry.
  - **Ops apply in order**, each seeing the result of the ones before it, so
    a later op can act on text an earlier op inserted.
  - **All-or-nothing.** If any op fails, the whole batch is rejected and the
    document is left untouched, with the error naming the op and why — fix
    that op and resend. Collaborators see the batch land as one change.
  - **Per-op shapes that are easy to get wrong:** `insert` takes either a
    `target` or `at: doc_start | doc_end` (mutually exclusive), plus
    `placement: before | after`; `replace` content must be a **single
    paragraph of inline markdown** — block content is rejected, so
    restructuring blocks is a `delete` op *plus* an `insert` op; `delete`
    takes an optional `through` second target to remove a range.
- **Publishing is two calls, and the first one writes nothing.** When the
  edits are complete, `publishDocument` (one generic tool for every kind —
  addressed by `reference` alone, like `readDocument`)
  **proposes** the publish: it returns the cascade the publish would perform
  — every approved descendant it would reach, what it would do to each and
  why — plus a single-use token valid for one hour. `confirmPublish({ token })`
  is the call that actually writes the new immutable version and applies
  those invalidations, in one transaction. It is **always two calls**, even
  when the proposal invalidates nothing. Don't publish after every micro-edit:
  finish a coherent set of changes, then publish once.
- **Not confirming is cancelling.** Let the token expire and nothing was
  written — there is nothing to discard and no cancel tool.
- **`baseVersion` is advisory.** Still required on the propose, and still the
  published head from `readDocument`, but it records what you edited against
  rather than gating the write — the publish targets the current head. The
  staleness contract lives on the *confirm*, which is rejected if the entity
  or any assessed descendant moved since you proposed; the fix is to
  re-propose (the same contract as a stale `searchDocument` handle).
- **Review the outcome set before confirming.** The confirm applies the
  proposed outcomes exactly as you reviewed them — the assessment is *not*
  re-run, which is what makes reviewing it meaningful. Read what each
  descendant got, and don't flatten the distinctions:
  - `not_assessed` — assessment was unavailable, so it is invalidated by
    default. Relevance was never established; this is not a judgement that
    your change affects it.
  - `invalidated` — judged to be affected by the change.
  - `pruned` — settled behind a spared ancestor, not a judgement of its own.
- **`skipAssessment` is the choice you have; overruling a sparing is not.**
  Passing `skipAssessment` on the propose skips the relevance pass and
  proposes **every** reached descendant for invalidation — the conservative,
  blanket outcome. It can only widen the blast radius, never narrow it.
  There is no `invalidateAnyway` tool and `confirmPublish` takes no
  overrides: invalidating something the assessment spared is **human-only**.
  If you disagree with a sparing, leave it in place and say so.
- **Acceptance-criterion writes are the other cascade trigger.**
  `createAcceptanceCriterion` / `updateAcceptanceCriterion` /
  `deleteAcceptanceCriterion` also write nothing on the first call: editing a
  criterion rolls its requirement's version, which can invalidate the designs
  and tasks approved against it, so each returns the same kind of proposal
  and redeems through the same `confirmPublish`. The confirm returns the
  **owning requirement** — the entity whose version moved — and, on a create
  or an update, the criterion it wrote alongside it in `criterion`. Read the
  new `AC-n` from there: it is claimed inside the confirm's transaction (which
  is why the proposal names a new criterion by title rather than by reference),
  so the confirm response is the only place it reaches you — don't re-list the
  requirement's criteria and match on title. A delete carries no `criterion`,
  having written no row. A frozen requirement is rejected with a 409 at the
  confirm.
- **Design coverage is a separate, cheap write.** Which ACs a design fulfils
  is *not* part of its document and does not roll a version or move it in the
  chain. Set it at creation with `createDesign`'s `acceptanceCriteriaRefs`, or
  afterwards with `addDesignCoverage` / `removeDesignCoverage` (one criterion
  at a time, idempotent) or `replaceDesignCoverage` (the whole set at once; an
  empty array clears it). Read it back with `getDesignCoverage`. Because a
  criterion's `AC-n` is numbered within its requirement, every one of these
  names the criterion as `requirementRef` + `criterionRef`, never `AC-3`
  alone — and the requirement must be the design's own.
- **Two views answer "is this covered?"** — `listAcceptanceCriteria` on a
  requirement returns each criterion with `fulfilledBy` (the designs that
  fulfil it), and `listDesignsForRequirement` gives the same relation from the
  other side, deduplicated. Use the first to find an AC with no design.
- **Set the repository on every design and task.** Where the work lives is a
  field, not a sentence in the body. `listGitHubConnections` lists the
  repositories connected to the org; `linkEntityRepository` attaches one by its
  `github_repo_id`; `listEntityRepositories` reads back what an entity carries.
  The cardinality mirrors the levels: **entities above a task accumulate any
  number of repositories** — so a design that spans repos links each one it
  touches — while **a task carries exactly one**, and linking another
  *replaces* it rather than adding. Set it as you create the entity; a
  design or task with no repository leaves an implementer guessing which
  codebase to open. Features and requirements may carry them too, but for those
  it's optional context rather than part of the spec.
- **Renaming**: pass the optional `title` on the `publishDocument` *propose*
  call — not on the confirm — to rename an entity alongside publishing.
- **Status transitions are a batch.** `transitionEntityStatuses` moves a
  **set** of entities through the lifecycle (e.g. `DRAFT → READY_FOR_REVIEW`,
  `DRAFT → CANCELLED`) — one tool for every kind, taking 1–25
  `{ reference, targetStatus }` pairs, each addressed by the bare reference.
  There is no singular `transitionEntityStatus`; raising ten drafts for review
  is one call, not ten. Every subject is decided on its own against the state
  machine: those that move come back under `transitioned`, and each one the
  machine refuses comes back under `failed` with its reason — so **check
  `failed`**, an illegal move doesn't fail the call or discard the legal moves
  batched with it. (A malformed reference, or the same reference twice, does
  reject the whole call before anything is written.) Only legal manual
  transitions are accepted; illegal moves and system-driven statuses are
  rejected, and **approving is human-only** — an agent can never move anything
  to `APPROVED`.
- **"Deleting" = cancelling, and it cascades.** There is no hard delete;
  transition the entity to `CANCELLED` to retire it. Cancelling is a
  soft-delete — the entity is hidden from the app UI but API and MCP reads
  still return it — and it **carries its descendants down with it**, returned
  as `cancelledDescendants` on the transition. Cancelling a feature therefore
  retires its whole subtree; check what you're about to take with you.
- **Terminal statuses are one-way and freezing.** `CANCELLED` and
  `COMPLETED` have no outgoing transitions, and they freeze content *and*
  title — publish is rejected once there. So when retiring an entity,
  **finish every edit first, transition last**:
  1. Move any content worth keeping to its new home (and update
     cross-references in other entities).
  2. Edit the retiring entity's draft down to its final content (e.g. a
     short pointer to where the content moved) via the document tools, then
     publish it — the final `title` goes on the `publishDocument` propose,
     and `confirmPublish` is what lands both the content and the rename.
  3. Transition it to `CANCELLED`.
  Cancelling before step 2 has *fully* completed — the confirm included, not
  just the propose — makes those edits permanently impossible.
- References are server-generated; never invent or assume the next number —
  read it from the create response.
- **Images and attachments are a two-step upload.** `createAttachmentUpload`
  does **not** transfer the file: compute the size and SHA-256 locally, call it
  to mint a one-shot ticket, then POST the bytes yourself with the `curl`
  command it hands back — never read the file into context or inline it as
  base64. Embed the returned absolute URL in the markdown (`![alt](url)` for an
  image, `[label](url)` otherwise). Tickets are single-use; mint a fresh one to
  retry.
- **Comments are how you talk about an entity without editing it.** Start a
  thread with `createCommentThread` — anchored by `quotedText` (an exact span
  occurring exactly once) or document-level when omitted — and reply with
  `addComment`. Read them with `listCommentThreads`, or fetch just the ones you
  care about by id with `getCommentThreads` (the `threadId`s come from
  `readDocument`, which already tells you each thread's status and location, so
  settled threads need never be fetched). Comments never change content and are
  accepted on an entity in any status. **Resolving, reopening or deleting a
  thread is human-only** — an agent may take part in a discussion but never
  close one down.
- **The record is readable.** `listVersions` gives an entity's timeline newest
  first — published versions interleaved with status transitions, approval
  decisions and approval retirements, each entry tagged with its `kind`, and
  each version carrying a `precis` of what it changed. It is **cursor**-paged:
  pass back the `nextCursor`, never compose one. `compareVersions` answers what
  changed between two published versions directly, so you never read both and
  diff them yourself.
- **Inline `:entityRef{reference=REQ-31}` directives** in a body you read back
  are cross-references whose labels resolve at read time. Batch them through
  `resolveEntityReferences` to get their current title, status and `viewUrl` —
  a renamed entity needs no edit to the documents referencing it.

## Process

1. **Research before writing.** Build the inventory from real sources
   (tickets, ADRs, code, docs) and classify every candidate with the
   altitude lens above before the first create call. When a source mixes
   levels (a ticket that states a capability *and* how it's built), split it:
   the capability goes up the chain, the mechanism waits for Design.
   Research surfaces gaps the sources don't close — keep a list of them and
   **put them to the user in one batch before you start writing**, so the
   answers land in the entities as decisions instead of as open questions.
   **Before any design or task, read the code as well.** Both levels are bound
   by code analysis: find the modules and layers the capability will live in,
   and learn the approach the codebase already takes there, so the design
   conforms to the grain rather than describing a system this one isn't.
2. **Check for an existing home** before creating, to avoid duplicates.
   `searchEntities` is the fastest first look — typo-tolerant full text across
   every entity, filterable by `types`, `statuses` and `feature`. Then walk
   structurally: `listFeatures` (optionally filtered to a project, or `"none"`
   for the unassigned), then `listChildren`, which lists any entity's direct
   children by `reference` alone — a feature's requirements, a requirement's
   designs, a design's tasks. Acceptance criteria are not a chain level, so
   list those with `listAcceptanceCriteria`; a project is not a chain parent,
   so list its features with `listProjectFeatures`.
3. **Create top-down.** Feature first (`createFeature`, optionally with
   `projectRefs`), then its requirements (`createRequirement` needs the
   `featureRef` from the create response), then acceptance criteria under each
   requirement — each a two-call step, `createAcceptanceCriterion` to propose
   and `confirmPublish` to write — then designs under each requirement
   (`createDesign` takes the `requirementRef`, plus `acceptanceCriteriaRefs`
   for the criteria it fulfils, which must be that requirement's own), then
   tasks under each design (`createTask` takes the `designRef`). **Link the
   repositories as you go** — `linkEntityRepository` on every design (each repo
   it touches) and every task (exactly one). Decompose a design into tasks along
   its repository and merge-safety boundaries, and give any task that can't
   start yet its `**Depends on:**` line.
4. **Move wording, don't duplicate.** When extracting a lower-level entity
   from a higher one (a requirement out of a feature, an AC out of a
   requirement), remove the moved text from the parent's draft — naturally a
   single `editDocument` call, whose `ops` delete the extracted passage and
   make any wording fixes the removal leaves behind — then publish the parent
   (propose, review the cascade, confirm). Spexd links the child to its
   parent automatically, so there's no need to list or point to it from the
   parent body.
5. **Verify at the end.** Walk the chain (`listFeatures` → `listChildren` at
   each level down, plus `listAcceptanceCriteria` on each requirement) and
   confirm the created set matches the plan, that every AC has a design against
   it (`fulfilledBy`), and that no implementation detail leaked above Design.
   Check the two things that are easy to leave half-done: **every design and
   task carries its repository** (`listEntityRepositories`), and **no entity
   contains implementation code** — an excerpt of existing code is fine, a
   ready-to-paste implementation is not. `listInbox` is the quick cross-cutting
   check that everything landed in the status you expected. Report what was created and anything cancelled — each
   as a **link** (`viewUrl`) on its reference, with its title, never a bare
   list of references.
