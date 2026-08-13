---
name: spexd-auditing
description: Audit a Spexd feature or requirement against the codebase — verdict every acceptance criterion, check each one against a real test, and flag designs whose prose no longer matches what shipped. Use this whenever someone asks whether a FEAT/REQ is done, complete, met, fully implemented, covered by tests, or still accurate; asks for a spec-compliance, traceability, coverage or merge-readiness check; or says things like "are all the ACs met", "is FEAT-3 actually finished", "do the designs still reflect reality", "what's left on REQ-28", or "review FEAT-1 against main". Reach for it even when the ask sounds casual or partial — a question about one requirement usually wants the same treatment as a whole feature. Distinct from code-review, which reviews a diff; this audits the written spec against the whole shipped codebase.
---

# Auditing a Spexd feature against the code

You are checking three separate things, and conflating them produces a useless report:

1. **Is each acceptance criterion actually true of `main`?**
2. **Does a test prove it** — one that asserts the criterion's own stated outcome?
3. **Do the designs still describe the system that shipped?**

A mature feature drifts in a specific way: the code moves forward, and the spec is left describing a system that no longer exists. So the most valuable findings are rarely "this is broken". They are "this criterion demands behaviour that was deliberately retired, and there is now a test asserting the opposite", and "this design's status table says six things haven't shipped that all shipped". Go in expecting that.

## Never infer a verdict from entity status

Everything in a mature feature tends to read `APPROVED`, including requirements with zero implementation and designs that describe a table dropped two migrations ago. Status tells you a human signed off once, at some version, on some content. It is not evidence. Every verdict must come from reading the source and the tests.

There is exactly one status that decides something, and it decides scope rather than verdict.

## `CANCELLED` entities are not in the audit at all

Cancelling is Spexd's soft delete: the entity is hidden from the app UI, but API and MCP reads still return it, so it will arrive in your `listChildren` and `readDocument` results looking like any other requirement or design. It is retired work. Someone decided it does not belong to the feature any more, and auditing it produces findings about spec nobody intends to honour.

So drop it, and drop everything under it — cancelling carries its descendants down with it, so a cancelled requirement takes its criteria and designs with it whether or not their own status says so. Filter on the status field as you walk the tree, before you read a single body: never verdict a criterion beneath a cancelled parent, never sort a cancelled design for accuracy, and never let either reach the report — not in a section, not in a table row, and above all not in the verdict counts. A count inflated by retired criteria misstates the health of the feature, which is the one number the reader most trusts.

The only acceptable mention is a single line noting how many cancelled entities you excluded, and only if there were enough that a reader comparing your report against the tree in the UI would otherwise wonder what happened to them. If a cancelled entity is genuinely load-bearing — a live criterion's `fulfilledBy` points at a cancelled design, say — that is a finding about the *live* entity's dangling reference, reported there.

## Gather the chain first, completely

Pull the whole tree before auditing anything. Judging one requirement in isolation makes you miss the cross-references that produce the best findings — a criterion under one requirement that duplicates one under another, or a design that was superseded by a sibling.

Use the Spexd MCP tools:

- `getEntity(FEAT-n)` and `readDocument([FEAT-n])` — the feature's own body drifts too, and it is the last thing anyone rereads.
- `listChildren(FEAT-n, pageSize: 100)` — the requirements. Discard the `CANCELLED` ones here, and don't descend into them.
- `readDocument([...])` for the requirement bodies. **Batch 5–6 at a time**; larger batches overflow and get spilled to a file you then have to re-read.
- `listAcceptanceCriteria(requirementRef, pageSize: 100)` for each surviving requirement — one call each, they parallelise well. Note each criterion's `fulfilledBy`.
- `listChildren(requirementRef, pageSize: 100)` for each surviving requirement — the designs. Discard the cancelled ones again; a live requirement can have cancelled designs beneath it.
- `readDocument([...])` for every remaining design, again in small batches.

If a `readDocument` result is spilled to a file, split it into per-reference files with a short Python one-liner rather than reading the whole blob back — you will want to grep individual designs later anyway.

## The five verdicts

A binary met/not-met flattens exactly the distinctions the reader needs. Use these:

| Verdict | Meaning |
|---|---|
| **Proven** | Implemented, and a test asserts *this criterion's own stated outcome*. |
| **Proof gap** | Implemented, but nothing tests it — or only a helper beneath it is tested. |
| **Latent** | Implemented and tested, but gated off in every environment. |
| **Obsolete** | The behaviour it describes was deliberately retired. Cannot be met as written. |
| **Not met** | No implementation on `main`. |

**Latent** and **Obsolete** are what make the report honest. A feature-flagged cascade is not "met" — nothing in production does it — but calling it "not met" throws away the fact that it is built, tested and one env var from working. A criterion whose premise was removed by a later decision is not a failure; it is spec that outlived its decision, and saying so points at the fix (edit the AC) rather than at phantom work.

Distinguish **not met** from **vacuously true**. When a requirement is entirely unbuilt, most of its criteria have nothing to violate — "ownership confers no permissions" is trivially true when nothing can change ownership. Mark them not met, and say *why* they read as satisfied, or the reader will think seven of ten are fine.

## Reaching a verdict on a criterion

**Find the implementation.** Grep for the mechanism, not the words of the AC. An empty grep across `app/`, `server/` and `shared/` is decisive evidence of absence — run it before writing "not met", and again before writing "met".

**Then find the test, and hold it to the criterion's own outcome.** This is the bar that makes the audit worth reading. A requirement can be surrounded by fifty passing tests while the specific thing it promises is untested. Ask: does an assertion fail if this criterion stops holding? If the answer is "some test in the area would probably break", that is a proof gap.

Recurring shapes worth recognising:

- **Only the predicate is tested.** A pure helper (`transitionRequiresConfirmation`, `isSameInstant`) has thorough unit coverage while the component consuming it has none. The rule is proven; the behaviour is not.
- **The component is stubbed everywhere.** Grep the component name across the test suite. If every hit is `ComponentName: true` in a stubs map, nothing exercises it — every AC resting on its interaction is a proof gap. This is easy to miss because the grep *looks* like coverage.
- **The E2E takes the API shortcut.** A spec that drives a UI flow over `page.request.patch(...)` proves the server, not the dialog the AC is about.
- **A test asserts the opposite.** The strongest possible signal, and always worth quoting verbatim. A test named "…**is ACCEPTED** … (AC-136 retired)" settles the matter faster than any amount of reasoning.

Cite evidence as the test's own name — `publish-repository.test.ts` "absorbs a same-content publish, returning head without a new version". A reader can find and trust that; a bare file path proves nothing.

## Checks that pay off disproportionately

Run these deliberately; each one caught something real.

**Does the requirement body contradict its own criteria?** Requirement prose gets amended when a decision changes; the criteria beneath it often do not. When a body says a value is "advisory" and three of its ACs demand it be rejected as a conflict, that is the finding — and it means those ACs, not the code, are what needs fixing.

**Is anything gated behind a feature flag?** Grep for `process.env` reads in the repositories and for `vi.stubEnv` in the tests. A flag stubbed on in every test and unset in every environment means the criteria it serves are latent. Read the comment on the flag — it usually names the blocker, which belongs in the report.

**Look for designs that report their own status.** Sections headed "Delivery status", "Not yet built", "Status note", or a shipped?-column table are the highest-yield thing in the whole audit. They were written ahead of the work, and when the work lands nobody goes back. A table marking shipped items as ❌, or a design closing with "this is the missing code" about code that exists and has tests, will actively mislead whoever reads it next.

**Do the criteria's premises still exist?** A criterion predicated on an entity kind, column, table or status that a later decision removed cannot be met by anyone. Check the premise before checking the behaviour.

**Check the entity references resolve.** Renumberings leave dead links. Verify ADR paths against `docs/decisions/`, and treat a bare `AC-nnnn` as suspect wherever criteria are requirement-local. A design citing `docs/decisions/027-drop-acceptance-criteria…` when the file is `029-…` is a broken link a reader will chase.

**Note criteria with no `fulfilledBy`.** Not damning on its own — a design may cover it without recording coverage — but an orphaned criterion is disproportionately often one that was superseded, duplicated from another requirement, or written for work nobody scheduled.

**Read the feature's own document last.** After auditing the requirements you know what shipped, and the feature body's summary claims are easy to check and frequently wrong.

## Auditing a design

Sort each design into: **accurate**, **stale in detail** (the argument holds, but names, signatures, line references or table names have moved), or **contradicts** (it asserts something false about `main` in a way that would cause a wrong decision).

That middle category matters. A design describing `metadata.status` when the column moved to `entity.status` is not wrong about anything that counts, and flagging it at the same severity as an inverted delivery table trains the reader to ignore you. Reserve **contradicts** for claims that would actually mislead.

Judge a design by whether its *reasoning* still holds, not whether its identifiers match. The most valuable designs are load-bearing arguments — why a constraint was given up, what must now be checked in code instead — and those survive renames. Say so when a document is stale in its particulars but right in its thinking; recommend an amendment, not a rewrite.

When you find a well-maintained design — one carrying an "Amended since" section reconciling itself with later decisions — name it as the house pattern the others should follow. That gives the recommendation a concrete model instead of an instruction.

## The report

Default to a **compliance register**: a per-requirement structure the reader can scan, then drill into.

Open with counts across the five verdicts, and immediately after them, **the two or three things actually worth acting on** — stated as findings, not categories. A reader who stops after that paragraph should still leave with the right conclusion.

Then one section per requirement, each with:

- A short lede in prose — what is genuinely in good shape, then the specific problem, in that order. Lead with the substance, not a restatement of the requirement's title.
- **A table of acceptance criteria**: Ref · Criterion · Verdict · Evidence or gap.
- **A table of designs**: Ref · Design · Accuracy · What has drifted.
- A recommendation callout only where there is a real decision to make. One per requirement at most; not every requirement earns one.

Close with feature-level drift and a short prioritised list of fixes.

Two things to hold to throughout. **Quote the code and the tests** — a line reference, a test name, a predicate — because the whole value of the report is that it is checkable. And **be fair to the code**: a feature whose real problem is six stale criteria and four out-of-date designs is in good shape, and the report should say so plainly rather than letting a wall of non-green verdicts imply otherwise.

If the audit runs to more than a couple of requirements, publish it as an Artifact — the tables are the point, and they need room. Load `artifact-design` first.

## The verdict file

The report is what a person reads. Emit a **verdict file** alongside it — the same audit in the form `pnpm review:import` loads into Spexd, so each finding reaches the subject's own page and the measures that count it. It lives under `reviews/` in the repository you audited.

### Where it goes

One directory per revision, named by the **full** commit hash you audited against, holding one file per scope you audited:

```
reviews/
  7b9279d346838b24dbeb2fecf89d021e9199983a/
    FEAT-32.json
    DES-227.json
```

Name each file after the scope you set out to audit — `FEAT-n.json` for a feature, `REQ-n.json` for one requirement's criteria, `DES-n.json` for a single design. That name is for whoever reads the directory; nothing in the store records a scope, so the file is judged only by the verdicts in it. A feature is still the natural unit for a *person* to audit. It is just no longer a unit the store knows about, which is why auditing a single design now produces a valid file.

A later sweep is a **new directory**, never an edit of an old one. A verdict travels with the revision it was formed at, so re-auditing at a later revision is a second finding rather than a correction — `accurate` at one revision and `stale` at the next *is* the drift signal, and editing the old file in place destroys the most useful thing a repeated audit has to say.

### The shape

A file is a flat set of verdicts at one revision. Nothing groups them: a verdict is keyed by the subject it judges, so there is no feature to name and no run to open or close.

| Field | Purpose |
|---|---|
| `revision` | The full commit hash judged. Applies to every verdict in the file, and travels with every measure derived from them |
| `entityVerdicts[]` | One per design you sorted: its reference, its verdict, and the evidence for it |
| `criterionVerdicts[]` | One per criterion you verdicted: the owning requirement, the criterion reference, its verdict, and the evidence for it |

```jsonc
{
  "revision": "7b9279d346838b24dbeb2fecf89d021e9199983a",
  "note": "spexd-auditing sweep over FEAT-32 at 7b9279d. Free-form provenance for a human reading the file; not stored.",
  "entityVerdicts": [
    {
      "entityReference": "DES-209",
      "verdict": "stale",
      "evidence": "The argument holds and the date guarantee it describes now has a real check behind it (tests/unit/terms-page.test.ts \"asserts the effective date\"), but it still names the column `metadata.status`, which moved to `entity.status`."
    }
  ],
  "criterionVerdicts": [
    {
      "requirement": "REQ-180",
      "criterionReference": "AC-1",
      "verdict": "proven",
      "evidence": "tests/unit/auth-guard.test.ts \"/terms — signed out — is not redirected\" drives the real middleware rather than a copy of its logic, and fails if the allowlist entry at app/middleware/auth.global.ts:11 is removed."
    }
  ]
}
```

Import it against the organization holding the specification:

```bash
pnpm review:import -- --org org_123 --dry-run reviews/<hash>   # parse and report, writes nothing
pnpm review:import -- --org org_123 reviews/<hash>
```

Two addressing rules, both of which a file can violate silently:

- **A criterion is addressed by the (requirement, criterion) pair**, never by a bare `AC-n`. `AC-1` restarts under every requirement, so a bare reference names a criterion under every requirement in the subtree — the verdict still lands, it is just somebody else's.
- **A file carries no organization id.** A verdict set is a statement about a revision of the software rather than about a tenant, and the same file is legitimately importable into a second organization holding the same specification. `--org` at import is the only thing that decides where the rows land.

A verdict carries no document version, and cannot: the importer records the version each subject stands at when the import runs, since that is the document the review read. So import a sweep promptly, against the specification it judged — a design published between the audit and the import records its verdict against the *new* version, which is the one claim the file cannot make true.

### Mapping your verdicts to the file's

Designs map straight across: **accurate** → `accurate`, **stale in detail** → `stale`, **contradicts** → `contradicts`.

The criteria do not. You reach five verdicts; the file has four, and the two with no value of their own are recorded by what they mean for the measure, with the distinction carried in `evidence`:

| Your verdict | In the file | Why |
|---|---|---|
| Proven | `proven` | — |
| Proof gap | `unproven` | Implemented, nothing asserts it |
| **Latent** | `unproven` | Built and tested but gated off in every environment, so no running system demonstrates it. Not `not_implemented` — the code exists and the open question is rollout |
| **Obsolete** | `contradicted` | The behaviour it describes was deliberately retired, so the software contradicts it as written |
| Not met | `not_implemented` | — |

Do not collapse or re-map these. Three of the four criterion verdicts count against proven identically, which makes merging them look free — but the summary buckets them apart, and that is what makes an unbuilt criterion read as backlog rather than as a specification that is wrong. Re-mapping one moves a measure in the product with nothing failing to say so.

`evidence` is what makes the file worth importing: it is the part a reader acts on. Carry over the citation you put in the report — the test's own name, the predicate, the line — not a restatement of the verdict.

### An audit that stops partway

There is no run to open, so there is none to mark failed, and nothing to abandon. Emit the verdicts you reached. Every subject you did not reach simply has no row and reads `unverified` in Spexd, which is a state of its own rather than a gap — so a half-finished sweep is a valid file, and every verdict in it is true of what it judged.

Say in the report which subjects you did not reach, and why. That is the one thing the file cannot carry: to the store, a subject you ran out of time for and one nobody ever intended to judge are the same absence.

## Scope

Audit the whole live feature unless the user names a narrower scope — cancelled entities are outside every scope, including one the user names explicitly, because the entity they pointed at may have been retired since they last looked. Say so and stop rather than auditing it.

If they ask about one requirement, still pull its siblings' criteria, because duplicated and superseded criteria only show up in comparison — then report on the one they asked about.

Do not fix anything you find. The deliverable is the assessment; edits to criteria and designs are the user's call, and several of the findings will be decisions (enable the flag, or retire the criteria?) rather than tasks.
