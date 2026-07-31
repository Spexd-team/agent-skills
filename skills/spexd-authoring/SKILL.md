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
| Tables / columns / indexes | ✗ | ✗ | ✗ | ✓ | ref |
| Algorithms / data structures | ✗ | ✗ | ✗ | ✓ | ref |
| Code / interfaces / HTTP contracts | ✗ | ✗ | ✗ | ✓ | ✓ |
| Architecture diagram | ✗ | ✗ | ✗ | ✓ | |
| Trade-offs / alternatives considered | ✗ | ✗ | ✗ | ✓ | |
| Definition-of-done checklist | | | | | ✓ |

Each entity may **reference** the level above ("see FEAT-2", "satisfies
AC-3" — always as a link to the entity, see *Content conventions*); none may
pre-empt the level below. A requirement that already names
the algorithm has done Design's job for it — and usually the wrong way.

---

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
architecture, vendors, libraries, schemas, algorithms, state machines, HTTP
contracts, code, and trade-offs are not just allowed but expected. It answers
*how do we build this so the ACs hold?*

This is **architectural design, not merely UX.** A design that only describes
screens and never the system beneath them — the data model, the transaction
boundaries, the concurrency and failure handling, the external services — is
incomplete. Screen layout can be part of a design, but it is never the whole
of one.

**The AC↔Design link is many-to-many** (the only fan-out/fan-in point in the
chain). One design can fulfil several ACs (shared infrastructure), and one AC
can need several designs (a frontend design *and* a backend design each cover
part of it). So split designs along **architectural seams**, not one-per-AC
by reflex.

**Design vs the levels above.** If you're naming a technology, drawing an
architecture diagram, choosing between approaches, or writing code — you're in
Design. Reference the ACs you satisfy; don't restate them in full.

**Design vs Task.** A design *decides* the approach; a task *executes* a
decision already made. Don't decompose a design into a checklist of work here
— that's the Task level.

**How to write a good one.** State the decision and the mechanism; show the
trade-offs and alternatives considered; diagram the architecture; include the
contracts and the key code where they clarify; call out failure modes and how
they're handled. Cite relevant ADRs and source tickets.

**Keep out — two things that creep into designs and don't belong.**

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

---

## Task

**What it is.** A concrete unit of implementation work decomposed from
exactly one design. It answers *what does someone actually do to ship a slice
of that design?* Ideally sized to a single reviewable change.

**Task vs Design.** A task never decides architecture — it carries out what
the design specified. If, mid-task, you discover the approach is wrong or
under-specified, update the **design**, don't quietly invent new architecture
in the task.

**How to write a good one.** An imperative title ("Implement `POST
/api/rides`"), a clear **definition of done**, and a subtask checklist. Point
back to the design it implements, as a link ("per [DES-9](…) — *Ride request
creation & state machine*"). Code snippets are fine here — a task executes design-level
decisions, so it inherits their implementation altitude.

**Keep out.** Don't re-argue the design; reference it. Don't introduce
architecture that isn't already in the design.

---

## Content conventions (all entities)

- **Don't repeat the title in the body.** The title is a separate field in
  Spexd, shown above the content — starting the body with an `# <Title>`
  heading just duplicates it. Begin the body with the sources line (below),
  then the content.
- **Don't indicate status in the content.** An entity's state lives in its
  Spexd lifecycle status field (managed via `transition*Status`), not in the
  body — no "Status:" line, no "shipped / planned / rejected" labels in the
  prose. Describe what the entity *is*, not what state it's in.
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
  satisfies [DES-9](https://www.spexd.com/feature/FEAT-3/REQ-7/DES-9)
  ```

  `getEntity` / `getEntities`, `listFeatures`, `listChildren` and
  `searchEntities` all return `viewUrl` alongside the reference — take the URL
  from the response rather than composing one yourself, and if you don't have
  it, read the entity rather than guessing. Two tools don't return it:
  `readDocument` (keep the link from whichever tool surfaced the entity) and
  the outstanding-work tools, whose items carry a `path` array that *is* the
  URL — `https://www.spexd.com/feature/` + the segments joined with `/`. That
  `path` is optional and dropped whenever an ancestor link is missing; when
  it's absent, pass the item's `reference` to `getEntity` and take the
  `viewUrl` from there rather than composing a partial path.

  An acceptance criterion is the one exception: its reference is numbered
  within its requirement, so `AC-3` is not addressable on its own and is
  linked as a query on its requirement's URL —
  `[AC-3](https://www.spexd.com/feature/FEAT-17/REQ-4?ac=AC-3)`.

  A bare `AC-3` or `DES-9` makes the reader go and find it. Link it.
- **For features and requirements**, a useful body shape is: `## Overview`
  (one or two sentences, product language) → behaviour/scope in product terms.
  Keep the mechanism out entirely rather than giving it a section — the "how"
  belongs in the Design entities the chain links automatically, not in a
  footnote on the feature or requirement. ACs are usually a single
  Given/When/Then line; designs are full technical documents; tasks are a
  definition of done plus a subtask checklist.

## Operational notes (MCP surface)

- **Editing is targeted and batched, not whole-document.** Entities are
  edited through the live-document tools, never by submitting a full body.
  Read first — `readDocument` (the live draft; it may be ahead of the
  published version `getEntity` returns) — then send **every** change for
  that document as a single `editDocument` call carrying an ordered `ops`
  array (1–50). Explicitly *not* one call per change. Your edits appear live
  to anyone editing the entity and merge with their concurrent changes.
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
  addressed by `kind` + `featureRef?` + `reference`, like `readDocument`)
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
- **Renaming**: pass the optional `title` on the `publishDocument` *propose*
  call — not on the confirm — to rename an entity alongside publishing.
- **Status transitions**: `transition*Status` tools move entities through
  the lifecycle (e.g. `DRAFT → CANCELLED`, `DRAFT → READY_FOR_REVIEW`).
  Only legal manual transitions are accepted; illegal moves and
  system-driven statuses are rejected.
- **"Deleting" = cancelling.** There is no hard delete; transition the
  entity to `CANCELLED` to retire it.
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

## Process

1. **Research before writing.** Build the inventory from real sources
   (tickets, ADRs, code, docs) and classify every candidate with the
   altitude lens above before the first create call. When a source mixes
   levels (a ticket that states a capability *and* how it's built), split it:
   the capability goes up the chain, the mechanism waits for Design.
2. **Check for an existing home** (`listFeatures`, then `listChildren` —
   which lists any entity's direct children by `kind` + `reference`: a
   feature's requirements, a requirement's acceptance criteria, an acceptance
   criterion's designs, a design's tasks) before creating, to avoid duplicates.
3. **Create top-down.** Feature first, then its requirements
   (`createRequirement` needs the `featureRef` from the create response),
   then acceptance criteria under each requirement — each a two-call step,
   `createAcceptanceCriterion` to propose and `confirmPublish` to write —
   then designs against the ACs (remember one design may satisfy several
   ACs), then tasks under each design.
4. **Move wording, don't duplicate.** When extracting a lower-level entity
   from a higher one (a requirement out of a feature, an AC out of a
   requirement), remove the moved text from the parent's draft — naturally a
   single `editDocument` call, whose `ops` delete the extracted passage and
   make any wording fixes the removal leaves behind — then publish the parent
   (propose, review the cascade, confirm). Spexd links the child to its
   parent automatically, so there's no need to list or point to it from the
   parent body.
5. **Verify at the end.** Walk the chain (`listFeatures` → `listChildren`
   at each level down) and confirm the created set matches the
   plan and that no implementation detail leaked above Design; report what was
   created and anything cancelled — each as a **link** (`viewUrl`) on its
   reference, with its title, never a bare list of references.
