---
name: spexd-importing
description: >-
  Bootstrap a whole Spexd project from an existing codebase and its supporting
  sources — reverse-engineer the feature set, then fan out one sub-agent per
  feature to derive requirements and acceptance criteria, then design across the
  requirements — all via the Spexd MCP tools (mcp__Spexd__*). Use when asked to
  import, onboard, backfill, or reverse-engineer an existing project/repo into
  Spexd, or to stand up the traceability chain for software that already exists.
  Discovery-first and evidence-grounded: features, requirements, and designs
  come from what the sources actually show, never invention.
user-invocable: true
---

# Importing an existing project into Spexd

Spexd models a strict traceability chain — **Feature → Requirement →
Acceptance Criteria → Design → Task**. This skill is about *bootstrapping that
whole chain for software that already exists*: you point it at a codebase (and
whatever else documents the product — READMEs, ADRs, tickets, API specs, UI)
and it reverse-engineers a first draft of the spec, top-down, for humans to
review.

It builds on `spexd-authoring`: **that skill owns the altitude/classification
lens — read it and apply it here.** Every entity this skill creates must sit at
the right level (detail only descends: no vendor, table, or algorithm above the
Design level), and the guidance below assumes you are already classifying by
those rules. This skill adds the *import-specific* method on top: discovery,
grounding, and fan-out.

## The two rules that govern an import

1. **Ground everything in evidence — don't invent.** Features, requirements,
   and designs must be traceable to something you actually observed in the
   sources (a route, a model, a screen, a doc, a ticket). If you cannot point
   to where a capability lives in the code or docs, it is not a feature — leave
   it out. A wrong-but-plausible feature is worse than a missing one, because a
   reviewer will trust it.
   - **The one deliberate exception is acceptance criteria.** ACs describe how
     you'd *prove* a requirement holds, so they may reach past today's
     behaviour — cover the golden path plus the failure and edge cases a
     correct implementation *should* handle, even where the current code
     doesn't. Use judgement; ACs are the spec you'd want, not a transcript of
     what the code does now.

2. **Everything lands in DRAFT.** An import is a *proposal* for humans to
   review, not an approved spec. `create*` raises entities in `DRAFT` by
   default — so simply create and stop. Do **not** call any `transition*Status`
   tool to advance entities toward review/approval; leaving them in `DRAFT` is
   the point. (Approval is a human-only action regardless.)

## The shape of an import

An import is four phases, run top-down. Never jump ahead — each phase is the
grounded input to the next.

```
Phase 1  Discover  →  the feature set (+ a Project to hold it)      [orchestrator]
Phase 2  Fan out   →  requirements, one sub-agent per feature       [sub-agents]
Phase 3  (same fan-out) acceptance criteria for each requirement    [sub-agents]
Phase 4  Design    →  designs across the requirements, linked        [orchestrator]
```

Phases 2 and 3 are the *same* per-feature sub-agent doing two steps in
sequence; Phase 4 is orchestrator-led because designs cut across features.

---

## Phase 1 — Discover the feature set (orchestrator)

Do this yourself, before any fan-out, because the feature list is the spine
everything else hangs off.

1. **Survey every source you were given.** Read broadly: the repo (entry
   points, route tables, top-level modules, domain/model directories, public
   API surface, CLI commands, background jobs), plus README/docs, ADRs,
   `package.json`/manifests, OpenAPI/GraphQL schemas, UI screens, and any
   tickets or specs supplied. Use `Explore`/`general-purpose` sub-agents for a
   wide read if the codebase is large — the goal is a complete map of *what the
   system lets someone do*, not a line-by-line audit.
2. **Derive candidate features from evidence.** A feature is a demoable unit of
   product scope (a surface, workflow, or capability someone can point at).
   Group the observed surfaces into capabilities — "user authentication",
   "checkout", "the admin dashboard", "the public API" — each backed by real
   code/docs. Apply the `spexd-authoring` feature-vs-requirement test: if a
   candidate is really a *rule within* a capability ("every request is rate
   limited", "no data leaks across tenants") or a cross-cutting concern
   (versioning, audit), it is a **requirement**, not a feature — hold it for
   Phase 2. When torn, prefer the coarser feature and push the detail down.
3. **Create a Project and the features under it.** `createProject` with a short
   product overview (what the system is, who it's for, how you scoped the
   import, and the sources surveyed). Then `createFeature` for each capability,
   passing `projectRefs: [<projectRef>]` so it links on creation (or
   `addFeatureToProject` after). Write each feature body in product language per
   `spexd-authoring` (overview, goals/non-goals, persona, scope, a `**Sources:**`
   line pointing at the files/docs that evidence it). Check `listFeatures` /
   `listProjectFeatures` first to avoid duplicating an existing home.

Capture the returned `featureRef` for every feature — those are the fan-out
work items for Phase 2.

## Phases 2 & 3 — Fan out: requirements, then acceptance criteria (sub-agents)

Delegate **one sub-agent per feature**, in parallel (launch them in a single
message so they run concurrently). Each sub-agent owns its feature's slice of
the chain end-to-end: it derives the requirements, then the ACs, and creates
them under that feature via the MCP tools. Because each sub-agent writes only
under its own `featureRef`, they work on disjoint subtrees and never collide —
references themselves are server-generated and unique across the whole org, so
nothing depends on the fan-out to keep them apart.

Give each sub-agent enough to work independently: the feature's `reference`,
title and body; the sources that back it; the pointer to load `spexd-authoring`
for altitude; and the two rules above. A prompt template:

> You are importing one feature of an existing project into Spexd. **First load
> the `spexd-authoring` skill and apply its altitude rules.** Feature:
> `<featureRef>` — "<title>". Its evidence: `<files/docs/routes>`.
>
> **Step A — Requirements.** Read the code and docs behind this feature and
> derive the behavioural rules it must uphold — what must be *true* within the
> capability, in product language ("the user can…", "every X must Y"). Ground
> each one in what you observe; don't invent capabilities the code doesn't
> show. For each, `createRequirement` under `<featureRef>` with a product-level
> body and a `**Sources:**` line. Keep mechanism out (no vendors, tables,
> algorithms, or code — those wait for Design). Check `listChildren` on
> `<featureRef>` first to avoid duplicates.
>
> **Step B — Acceptance criteria.** For each requirement you created, write the
> acceptance criteria that would prove it — atomic Given/When/Then scenarios,
> one condition each, golden path *and* the important failure/edge paths as
> **separate** criteria. Here you may use judgement and go beyond today's
> behaviour: describe what a correct implementation *should* guarantee, not only
> what the current code does. `createAcceptanceCriterion` under each requirement,
> then `confirmPublish` with the token it returns — the create alone writes
> nothing. Prefer measurable outcomes over adjectives.
>
> Create everything in **DRAFT** — do not transition any status. Return the
> requirement and AC references you created, plus any capability you noticed
> that seems to belong to a *different* feature (so the orchestrator can place
> it), and any architectural seams you spotted (to inform the design phase).

Collect each sub-agent's report: the created references and any surfaced
cross-feature notes. Reconcile — if a sub-agent flagged a rule that belongs to
another feature, place it there rather than duplicating.

## Phase 4 — Design across the requirements (orchestrator)

Designs are the first level where implementation lives, and the **AC↔Design
link is many-to-many** — one design can satisfy ACs across several requirements
and even several features (shared infrastructure), and one AC can need several
designs (e.g. a frontend and a backend design). So design **after** the fan-out,
with a view of the whole chain, and split designs along **architectural seams**,
not one-per-requirement.

1. **Re-read the real architecture** — this is where you *do* name the actual
   stack, because Design is where detail belongs and because an import must
   reflect how the system is genuinely built. Identify the seams from the code:
   services, data stores, auth boundary, background processing, external
   integrations, the client(s). The sub-agents' "architectural seams" notes from
   Phase 3 are a starting list.
2. **Create one design per seam**, each grounded in the codebase: state the
   mechanism as it actually exists (or is clearly intended) — data model,
   contracts, key flows, failure handling, trade-offs — citing the files/ADRs
   that evidence it in a `**Sources:**` line. Do **not** invent architecture the
   code doesn't support; where the code is silent on something an AC needs, note
   the gap rather than fabricating a mechanism.
3. **Link designs to the ACs they fulfil** across every feature, so each
   requirement's acceptance criteria are covered by at least one design. Use the
   `getOutstandingWorkForFeature` / `listAcceptanceCriteria` views to confirm no
   AC is left without a design. (Tasks are out of scope for an import
   — leave decomposition to `spexd-authoring`/`spexd-implementing` once humans
   have reviewed.)

You may fan out this phase too (a sub-agent per architectural seam), but keep
the *linking* coherent from the orchestrator so the many-to-many wiring is
consistent.

---

## Operational notes

- **Altitude is `spexd-authoring`'s job — defer to it.** Don't restate the full
  classification lens here; load that skill and use it. The failure mode unique
  to imports is letting the *code you just read* pull implementation detail
  upward — you were just looking at tables and vendors, so it's tempting to name
  them in a requirement. Don't. Keep them for the Phase 4 designs.
- **Grounding beats completeness.** A smaller set of well-evidenced entities is
  a better import than a sprawling one padded with guesses. If evidence is thin
  for a candidate, say so in the body or leave it out — don't manufacture
  detail to fill the shape.
- **Everything DRAFT, no transitions.** Reiterated because it's easy to slip:
  `create*` is enough; never call a `transition*Status` tool during an import.
- **MCP surface** (see `spexd-authoring` for the full editing/publishing model):
  entities are created with `create*`; edited by reading with `readDocument`
  and sending **every** change for that document as one `editDocument` call
  with an ordered `ops` batch (targeting exact text via `target.find`, falling
  back to a `searchDocument` handle only for something with no text to match);
  and published in **two** calls — a content-less `publishDocument`, which
  writes nothing and returns a cascade proposal with a one-hour token, then
  `confirmPublish({ token })`, which is the call that writes. It is always two
  calls, and during an import the proposal is usually empty — everything you
  create is `DRAFT`, so there is rarely an approved descendant to invalidate —
  but the confirm is still required, and skipping it means nothing was
  published. `createAcceptanceCriterion` proposes and confirms the same way.
  References are
  server-generated and **unique across the org** — read them from the create
  response and never invent them. A bare reference resolves on its own, so
  reading one back needs nothing else — `getEntity` (or `getEntities` for a
  batch), and the document tools and `listChildren` alike. Only writes name a
  parent, as `createRequirement` takes a `featureRef`.

## Process checklist

1. **Survey** all supplied sources (repo + docs + tickets); build a complete map
   of what the system does.
2. **Discover** the feature set from evidence; `createProject`; `createFeature`
   for each capability linked into the project. Capture every `featureRef`.
3. **Fan out** one sub-agent per feature (concurrently). Each: derive grounded
   **requirements**, then judgement-based **acceptance criteria**, created under
   its feature.
4. **Reconcile** the sub-agents' reports; place any cross-feature rules; collect
   the architectural-seam notes.
5. **Design** across the requirements along architectural seams, grounded in the
   real codebase; link each design to the ACs it fulfils; confirm every AC is
   covered.
6. **Verify** the chain (`listProjectFeatures`, then `listChildren` at each level
   down — feature → requirement → acceptance criterion → design):
   every entity is `DRAFT`, nothing was transitioned, no implementation detail
   leaked above Design, and nothing was invented beyond the AC exception. Report
   the project and references created for human review.
