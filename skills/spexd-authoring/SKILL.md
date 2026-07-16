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
AC-3"); none may pre-empt the level below. A requirement that already names
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
back to the design it implements ("per the *Ride request creation & state
machine* design"). Code snippets are fine here — a task executes design-level
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
- **Cross-reference by reference** ("see FEAT-17", "satisfies AC-3"), and
  remember requirement/AC/design/task numbering is **feature-scoped**: REQ-1
  exists under many features, so always qualify a reference with its feature.
- **For features and requirements**, a useful body shape is: `## Overview`
  (one or two sentences, product language) → behaviour/scope in product terms
  → `## Design (out of scope here)` (one short paragraph pointing at
  ADRs/tickets for the mechanics, so the "how" has a home to point to without
  landing in the body). ACs are usually a single Given/When/Then line;
  designs are full technical documents; tasks are a definition of done plus a
  subtask checklist.

## Operational notes (MCP surface)

- **Editing is anchored, not whole-document.** Entities are edited through
  the live-document tools, never by submitting a full body: `readDocument`
  (the live draft — it may be ahead of the published version the `get*`
  tools return) → `searchDocument` (exact text → match handles with
  context) → `insertContent` / `replaceContent` / `deleteContent` (targeted
  edits at those handles, or at `doc_start`/`doc_end`). Your edits appear
  live to anyone editing the entity and merge with their concurrent
  changes. A stale handle (the matched text changed under you) is rejected
  — re-run `searchDocument` and retry.
- **Publishing is a separate, content-less step.** When the edits are
  complete, `publish*` flushes the entity's current shared draft to a new
  immutable version. It takes `baseVersion` (the published head from
  `readDocument`); a stale base returns 409 — your draft edits stay in
  place, so re-read and publish again. Don't publish after every micro-edit:
  finish a coherent set of changes, then publish once.
- **Renaming**: pass the optional `title` on any `publish*` call to rename
  an entity alongside publishing.
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
     publish it — with the final `title` on that `publish*` call.
  3. Transition it to `CANCELLED`.
  Cancelling before step 2 makes those edits permanently impossible.
- References are server-generated; never invent or assume the next number —
  read it from the create response.

## Process

1. **Research before writing.** Build the inventory from real sources
   (tickets, ADRs, code, docs) and classify every candidate with the
   altitude lens above before the first create call. When a source mixes
   levels (a ticket that states a capability *and* how it's built), split it:
   the capability goes up the chain, the mechanism waits for Design.
2. **Check for an existing home** (`listFeatures`,
   `listRequirementsForFeature`, `listAcceptanceCriteriaForRequirement`,
   `listDesignsForAcceptanceCriteria`, `listTasksForDesign`) before creating,
   to avoid duplicates.
3. **Create top-down.** Feature first, then its requirements
   (`createRequirement` needs the `featureRef` from the create response),
   then acceptance criteria under each requirement, then designs against the
   ACs (remember one design may satisfy several ACs), then tasks under each
   design.
4. **Move wording, don't duplicate.** When extracting a lower-level entity
   from a higher one (a requirement out of a feature, an AC out of a
   requirement), remove the moved text from the parent's draft
   (`searchDocument` → `deleteContent`) and publish the parent. Spexd links
   the child to its parent automatically, so there's no need to list or
   point to it from the parent body.
5. **Verify at the end.** Walk the chain (`listFeatures` →
   `listRequirementsForFeature` → …) and confirm the created set matches the
   plan and that no implementation detail leaked above Design; report
   references created and anything cancelled.
