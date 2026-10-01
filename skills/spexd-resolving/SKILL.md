---
name: spexd-resolving
description: >-
  Settle the findings of a completed Spexd review on a design or on a single
  acceptance criterion — put the findings to the reader as choices, correct the
  specification where the software is right, raise work where the software is
  wrong, publish as one version, and ask for the subject to be judged again, via
  the Spexd MCP tools (mcp__Spexd__*). Use when a turn hands the work over in
  words like "Resolve the 4 findings on the latest review of DES-70" or "Resolve
  the finding on the latest review of REQ-31/AC-3", or whenever someone asks to
  resolve, settle, act on, clear or fix the findings, points or drift a review
  reported. Distinct from requestReview, which produces the findings, and from
  spexd-auditing, which audits a feature against the code by hand.
user-invocable: true
---

# Resolving a review's findings

A completed review says the specification and the software disagree. **It never
says which of the two is wrong**, and neither do you. Every finding is put to the
reader, the reader decides it, and you carry the decisions out.

There are three dispositions per finding and only three:

| Disposition | The reader is saying | What gets written |
|---|---|---|
| **Resolve in favour of the code** | the software is right, the specification is stale | the wording is corrected — a design's passage in its live draft, or a criterion's own given/when/then |
| **Raise a task to fix** | the specification is right, the software is wrong | a task naming the change the *software* needs |
| **Skip it** | not ready to settle this one | nothing at all |

**Skipping writes nothing, and that is the design rather than an omission.**
There is no resolved-by-a-reader state, and inventing one would make a finding
that one reader dismissed invisible to the colleague they are about to discuss
it with. A skipped finding stands, and the next review reaches it again.

## Two subjects, and how many findings each carries

A review judges either a **design** or a **single acceptance criterion**,
addressed `REQ-4/AC-3`. One procedure covers both — but the number of decisions
it asks the reader for is not the same, and getting that wrong is the mistake
this section exists to prevent.

**A design's finding is separable.** Its judgement labels each point on its own
merits — `stale`, `contradicts`, `accurate` — so a point is a claim the reader
can settle by itself, and a four-point finding is four decisions.

**A criterion's finding is ONE finding, however many points it lists.** Its
judgement answers a single question — *does the software demonstrate this
given/when/then* — and the points are the evidence chain behind that one answer.
They carry no label precisely because there is no per-point verdict to give them.
Four points on a criterion are four reasons for one conclusion, not four things
to fix, and every one of them is settled by the same correction.

So on a criterion: **one question, about the verdict as a whole.** Never one per
point, never "the first of your four findings", and never a count in what you say
back — the hand-off carries no number for a criterion, and inventing one promises
the reader four decisions where they have one.

| | A design | An acceptance criterion |
|---|---|---|
| Addressed as | `DES-70` | `REQ-4/AC-3` — never a bare `AC-3`, which names a different criterion under every other requirement |
| Findings to decide | one per actionable point, or one where the summary is all there is | **always exactly one** |
| Verdict vocabulary | `accurate` / `stale` / `contradicts` / `unverified` | `proven` / `unproven` / `contradicted` / `not_implemented` / `unverified` |
| Read it with | `getEntity` + `readDocument` | `getEntity` on the requirement + `getRequirementCriteria` |
| Correcting it writes | the passage, in the live draft, then one publish | the criterion's own given/when/then, in its requirement's draft, then one publish of the requirement |
| The version that rolls | the design's | the **owning requirement's** |
| Raising work writes | a task beneath the design, plus a comment anchored at the passage | a task beneath a design that fulfils the criterion, and **no comment** |
| The re-review names | `DES-70` | `REQ-4/AC-3` |

Everything that follows is written for a design and says where a criterion
differs. Where it says nothing, the two are the same.

## When to use this

- A hand-off turn: *Resolve the 4 findings on the latest review of DES-70*, or
  *Resolve the finding on the latest review of REQ-31/AC-3*.
- "Act on those findings", "fix the drift the review found", "the review says
  this design is stale — sort it out", "the review says this AC isn't proven".
- **Not** for running a review. That is `requestReview`, one call.
- **Not** for auditing a whole feature against the code by hand. That is
  `spexd-auditing`.

**Anything coarser than those two is not a subject.** A requirement, a feature
and a project have no verdict of their own — a requirement's reference asks for
its *criteria* to be judged, one verdict each. Asked to "resolve the findings on
REQ-31" or "on FEAT-3", settle the subjects one at a time, say which one you are
on, and never fold several into a single set of questions or a single publish.

## The order

One fixed order, and every part of it exists to make something else safe. The
reasons are here because a procedure whose reasons are known survives being
optimised; one that is just a list gets shortened by the first model in a hurry.

1. **Read the subject and its finding.**
2. **Ask about every finding, one at a time, before acting on any of them.**
   *Because a correction is a publish.* Settling one finding at a time is one
   publish each, and every publish runs the relevance cascade over every
   approved descendant. Four findings settled separately are four cascades and
   four chances to invalidate an approval that a single version would have
   spared. **On a criterion this is exactly one question**, because a criterion
   carries exactly one finding — see [Two subjects](#two-subjects-and-how-many-findings-each-carries).
3. **State the plan, with a count per disposition, and carry straight on.**
   *Because the arithmetic is the one thing the reader cannot check by eye.*
   They have just decided each finding one at a time; the total is the part
   they have not seen. Don't stop for a go-ahead: the last moment at which
   nothing is irreversible is the publish, not the plan, because everything
   before it lands in the live draft and a draft is not a version. **On a
   criterion there is no arithmetic**, so this is one sentence saying what you
   are about to do — still a statement, still no go-ahead asked.
4. **Apply every correction to the live draft.** All of them. No publish yet.
   **On a criterion** the correction is one `updateAcceptanceCriterion`, which
   writes into the owning requirement's draft; step 7 then publishes that
   requirement.
5. **Raise each task.**
6. **Anchor each task's comment.** *Corrections before tasks, tasks before
   comments.* A comment anchors by exact quoted text against the document that
   will be published, so a thread created first can be left pointing at a
   passage the same batch then rewrote. **A criterion writes no comment** —
   there is no document for one to anchor in.
7. **Publish once** — propose, then confirm. *Nothing before this point is
   irreversible*, which is what makes stopping safe at every earlier step.
8. **Report what the cascade did.**
9. **Ask for the re-review.** *Last, because it judges the version that now
   exists.* Asked any earlier it would judge the one being replaced.

## Reading the finding

### On a design

`getEntity` for the link, the title, the status and the ancestor chain;
`readDocument` for the body **and** the verdict. Read the **live draft** — the
default — because that is what the corrections go into and what the reader is
looking at.

A design's verdict comes back on that read: a `state` (`accurate`, `stale`,
`contradicts`, `unverified`) and a `finding` in parts — a one-line `summary`, a
`points` list of `{ label, text }`, an optional `closing`, and where the
recorder added them a `demotion`, `unresolvedPaths` and `demonstration`.

**A point whose label says the document and the software agree is not a finding
to act on.** Those are the quiet ones the verdict block folds away. Act on the
rest, and count the rest.

A finding carrying a summary and no points at all is still **one** finding: act
on the summary.

### On a criterion

`getEntity` on the **owning requirement** for the link and the chain, then
`getRequirementCriteria({ requirementRef, references: ['AC-3'] })` for the
criterion itself. That returns its `title` and its three fields — `precondition`,
`action`, `outcome` — its `fulfilledBy` designs, and its `verdict`.

The verdict has the same shape and a different vocabulary: a `state` of `proven`,
`unproven`, `contradicted`, `not_implemented` or `unverified`, and a `finding`
with the same parts. **Its points carry `label: null`** — that is not missing
data, it is the judgement saying there is no per-point verdict to give, because
the whole finding answers one question.

**So there is one finding to settle, whatever `points` contains.** Read every
point — they are the evidence and the reader needs them — but read them as the
reasons for one conclusion. `readDocument` on the requirement is worth doing for
context, and its body does **not** contain the criterion: a criterion is a
structured row, not a passage.

### Opening, either way

Open the way you would with any Spexd entity — the `viewUrl` as a markdown link
on the reference, the title, and what the review actually said — then go
straight to the first question. Don't summarise the findings into a plan before
the reader has decided anything; the plan comes after the decisions, not before.

### The count in the ask

The number in the hand-off is **advisory**. It is what the reader pressed a
moment ago, and a re-review landing between the press and your read replaces the
findings underneath it.

**Report what you actually read.** If it differs from the number in the ask, say
so in one sentence and carry on with the findings that exist — never act on the
number, and never manufacture a fourth finding to make the arithmetic match.

**A criterion's hand-off carries no number at all** — *Resolve the finding on the
latest review of REQ-31/AC-3* — and that is the correct reading of it rather than
an omission. Don't count the points back at the reader to fill the gap.

## Asking a finding

One `askChoice` per finding — **and a criterion has one finding**, so it gets one
question covering the verdict as a whole. The finding is the context; the options
are the decision.

- **The context is the finding's own text** — the point's `text` verbatim, and
  the citation it carries: the file, the lines, and the revision the review
  read. The reader is deciding whether the document or the software is wrong,
  and they cannot do that from a paraphrase.
- **Three options and only three**, in the reader's terms rather than the
  tools': *the code is right — correct the document*, *the document is right —
  raise a task*, *leave this one*. Never add a fourth, never fold two together,
  never offer a resolve-everything shortcut.
- **The options are a shortcut, not a form.** A reader who types something else
  is still answering — take what they said.
- Put them **one at a time**, in the order the finding lists its points, so the
  reader can see how far through they are.

Ask exactly this way **in both approval modes**. A choice is a decision only the
reader can make; it is not a write approval, and the reader's mode has nothing
to do with it.

### The one question on a criterion

The context is the **whole** finding — the summary, every point verbatim with its
citation, and the closing where there is one. The reader is deciding whether the
criterion or the software is wrong, and the points are what tells them; presenting
one point and holding the rest back asks them to decide on a fraction of the
evidence.

The three options, in the reader's terms:

- *the software is right — reword the criterion to match it*
- *the criterion is right — raise work against the software*
- *leave this one*

**Work out which case an `unproven` verdict is before you offer the choice.**
`unproven` does not mean the behaviour is missing — that is `not_implemented`. It
means nothing **demonstrated** the criterion at the revision reviewed, and the
commonest cause by far is code that does the right thing with no test asserting
it. The reader is then choosing between rewording the criterion to something the
software does demonstrate, and raising work to add the demonstration. Offering
that as "the software is right, so reword it" invites them to weaken a criterion
that was correct all along. Read the finding for which case it is, say so in the
question's context, and let them decide.

**Never split it.** Four points is not four questions, not "which of these four
would you like to settle first", and not a list to tick. Every point is evidence
for the same conclusion about the same given/when/then, so any correction settles
all of them at once and a per-point question asks the reader to decide the same
thing four times. If the points genuinely describe *different* observable
outcomes, that is a criterion doing two jobs — say so, and offer to split it into
two criteria as a separate piece of work rather than smuggling the split into
this run.

## Nothing before everything

**No write of any kind until every finding has been decided.** Not an edit, not
a task, not a comment, not a publish, and not a "while you think about it"
change to the draft.

Reading is free, and is how the waiting time is spent well: read the passage
each finding names, read the code it cites, work out what the rewrite would say.
None of it lands.

The check a reviewer will apply is exactly this: after decisions one to three of
four, the entity's content, version, lifecycle status and approvals are all
precisely as they were before the run started.

## The plan

Once every finding is decided, and before the first write, state the plan — and
then carry straight on from it into the corrections.

State **a count per disposition**, not a re-listing of the findings:

> Three findings to settle by correcting the document, one by raising a task,
> none left standing. That is one new version of DES-70 and one cascade over its
> approved descendants.

The reader has just decided each finding one at a time; what they cannot check
by eye is the arithmetic. The counts must add up to the number of findings you
read, and a disposition nobody chose is reported as none rather than omitted.

**On a criterion there is no arithmetic to state**, because there was one
decision. Say what you are about to do in one sentence, and name the consequence
the reader has not seen — the requirement whose version will roll:

> Rewording AC-3's outcome to match what shipped. That is one new version of
> REQ-31 and one cascade over the designs and tasks approved against it.

A count per disposition here would be three numbers, two of them zero, about a
single decision the reader made a moment ago. Don't pad it into one.

**It is a statement, not a gate.** Never ask the reader to confirm the plan, in
either mode. It restates decisions they have just made one by one, and whether a
*write* needs their permission is the write gate's question rather than yours —
the plan is not a write, so there is nothing for a gate to hold.

## Correcting a passage

This is the **design** arm; a criterion has no passage — see
[Correcting a criterion](#correcting-a-criterion).

For each finding the reader settled in favour of the code:

1. **Read the code the finding cites, before rewriting anything** — `readFile`
   at the path the citation names, `searchCode` where the citation is a symbol
   rather than a path. A rewrite drafted from the finding's summary alone
   describes the finding, not the software.
2. **Locate the passage** with `searchDocument` on the design's reference. Use a
   short exact string from within a single block — a match never spans block
   boundaries — and lengthen it until it is unique.
3. **Rewrite it to describe what shipped.** Same altitude, same voice, the
   design's own conventions. You are correcting a claim, not rewriting the
   section around it, and not adding a note about having corrected it.

Then apply them with `editDocument`. **Send the corrections as one ordered `ops`
batch** where the reader is meeting them together: that is the tool's own
contract, and the batch is all-or-nothing, so a partial correction cannot land.
Where the reader is approving them one at a time, one call per correction is
fine — **the rule that matters is one publish, not one edit.**

Read the `after` excerpts and the `warnings` the response carries rather than
re-reading the document. If an op fails because its target no longer resolves
uniquely, the document is untouched and the error names the op: re-search and
resend. **A handle minted before an edit is stale after it** — search again for
anything you still mean to edit rather than reusing handles from before.

## Correcting a criterion

Where the reader settled a criterion in favour of the code, there is no document
to search and no passage to locate. The criterion **is** three fields, and
correcting it is rewriting them.

1. **Read the code the finding cites, before rewording anything** — `readFile`
   at the paths the points name, `searchCode` where a citation is a symbol. The
   same rule as a design: a rewording drafted from the summary alone describes
   the finding, not the software.
2. **Reword the fields the finding actually touches**, and only those.
   `updateAcceptanceCriterion` changes the fields you pass and leaves the rest
   untouched, so pass `precondition`, `action`, `outcome` or `title`
   individually rather than resending all four.
3. **Keep it an acceptance criterion.** Load `spexd-authoring`'s **Acceptance
   Criteria** section and hold the rewording to it: one precondition, one
   action, one *observable* outcome, and no mechanism. The commonest way to get
   this wrong is to fix an `unproven` verdict by describing what the code does —
   which turns the criterion into a design and makes it unfalsifiable. What the
   review found is that the stated outcome is not demonstrated; the correction
   is a true, still-testable outcome, not a description of the implementation.

**One call settles it**, because there is one finding:
`updateAcceptanceCriterion({ requirementRef, reference, ...fields })` writes the
reworded fields into the owning requirement's draft and returns the criterion.
It rolls no version: the correction reaches one when the requirement is
published — see [Publishing](#publishing).

**Never reach for `editDocument` here**, and never edit the requirement's body to
"fix" a criterion: the criterion is not in that document, so an edit that appears
to work has changed a different thing.

**A rewording is not a deletion.** If the reader's answer really means the
criterion should not exist, stop and say so — `deleteAcceptanceCriterion` is a
different decision with a different consequence and is not one of the three
dispositions.

## Raising work

For each finding the reader settled by raising work, in this order. The design
arm is below; a criterion differs in where the task hangs and in writing no
comment — see [On a criterion](#on-a-criterion-1).

**1. `createTask` beneath the design** — `designRef` is the design under review,
and the owning feature comes from it. The task carries:

- a title naming the change to the **software**, never to the document;
- the finding's text verbatim, and its citation — the file, the lines, and the
  revision the review read;
- what the software has to do for the document's claim to be true again;
- a link back to the design the finding was raised on.

Load `spexd-authoring`'s **Task** section and write it to those conventions
rather than from memory.

**2. `createCommentThread` on the design**, with `quotedText` set to the exact
passage the finding names and a body linking to the task you have just created.
The quoted span must occur exactly once in the document — take it from the draft
**as it now stands**, after the corrections, and lengthen it until it is unique.
An overlapping or ambiguous span is rejected.

**Neither a task nor a comment touches the document's content**, so this
disposition runs no cascade and needs no publish.

A raised task's thread is never resolved by this procedure. It is the record of
why the task exists, and its point is not addressed until that task lands — so
it stays open.

Where a correction you publish does settle what an existing open thread on the
design asked for, reply in that thread first, saying how its point was
addressed, and only then resolve it with `resolveCommentThread`. A resolve with
no reply leaves a person unable to tell settled from dismissed. **Never reopen
or delete a thread.**

### On a criterion

**One task, and no comment.**

`createTask` beneath a design that fulfils the criterion — the `fulfilledBy`
list on the criterion you read. Where more than one design fulfils it, ask the
reader which; where the finding's citations sit squarely in one of them, propose
that one and say why. The task carries what a design's does, plus the criterion:

- a title naming the change to the **software**, never to the criterion — and
  where the verdict is `unproven` that change is usually a **test**, not a
  behaviour change, so say which the task is asking for;
- the criterion's `REQ-4/AC-3` address, its given/when/then in full, and its
  `viewUrl` as a markdown link — a reader of the task must be able to see what
  it is meant to make true;
- the finding's text verbatim and its citations — the files, the lines and the
  revision the review read;
- what the software has to do for the criterion to be demonstrated.

**No comment is anchored, and that is deliberate rather than an omission.**
`quotedText` resolves against a document, and a criterion is a structured row —
it appears in no document, including its own requirement's (its text is not in
that body). There is nothing to anchor to, and a document-level comment on the
requirement would point at the wrong thing. The criterion's own verdict is where
a reader meets this finding, and it stands there until the next review reaches
it — which is exactly what a left finding does too.

**Where nothing fulfils the criterion**, there is no design to raise the task
beneath. Say so plainly, and put it to the reader: name a design to raise it
under, or leave the finding standing. Don't invent a design, don't raise the task
somewhere it does not belong, and don't silently fall back to leaving it.

## Publishing

Both subjects publish as **propose then confirm** — `publishDocument`, then
`confirmPublish` — and on both the propose writes nothing. What differs is which
entity you publish, and so which version rolls.

### A design

If, and only if, at least one correction landed:

1. **`publishDocument({ reference, baseVersion })`** — `baseVersion` is the
   published head from your `readDocument`. **This writes nothing.** It returns
   the cascade the publish would perform: one outcome at depth 0 for the design
   itself, one for every approved descendant it reaches, each with a reason, and
   a single-use token valid for an hour.
2. **`confirmPublish({ token })`** — this is what writes.

**Once.** One proposal, one confirm, one version, one cascade, for the whole set
of corrections. It is always two calls, even when the proposal invalidates
nothing, and not confirming *is* cancelling — the token expires and nothing was
written, so there is nothing to undo.

Don't pass `skipAssessment`: it proposes the blanket invalidation of every
approved descendant, which is the outcome the whole of this order exists to
avoid.

You cannot overrule an outcome on the confirm. If you believe a correction was
more substantive than the assessment judged it, say so to the reader and leave
it with them — invalidating something the cascade spared is reserved to people.

### A criterion

**The reword is not the publish.** `updateAcceptanceCriterion` wrote the fields
into the owning requirement's draft and rolled nothing, and the re-review judges
the **published** criterion — so if the reword landed, publish the requirement:

1. **`publishDocument({ reference: <the owning requirement>, baseVersion })`** —
   `baseVersion` is the requirement's published head from `readDocument`. **This
   writes nothing.** It returns the cascade and a single-use token valid for an
   hour.
2. **`confirmPublish({ token })`** — this is what writes.

**Read the proposal's depth-0 outcome carefully: it is the owning REQUIREMENT,
not the criterion.** Publishing the requirement rolls its version, which is what
can invalidate the designs and tasks approved against it. So the reader is told
about a version they did not name and descendants they may not have been thinking
about, and they need that in the report.

**The publish carries the requirement's whole draft**, not only the reword: a
collaborator's unpublished change to its body or its other criteria goes out in
the same version. `readDocument` on the requirement shows the draft, its
`criteria` included — if the version would carry anything beyond the reword, tell
the reader before confirming.

The rest is the design's rules unchanged: always two calls even when the proposal
invalidates nothing, not confirming *is* cancelling, no `skipAssessment`, and no
overruling an outcome on the confirm. And **once** — one finding means one
proposal and one confirm, so a second propose in the same run means something has
gone wrong; stop rather than publishing twice.

A frozen requirement refuses the reword itself with a 409. That is the ordinary
refusal, and it refuses the *requirement*, not the criterion.

## Reporting

After the publish, **in every mode**, tell the reader **which descendants lost
their approval and which kept it** — each named, with the reason the proposal
gave. A count is not enough: every invalidated descendant is now something a
person has to approve again, and they need to know which.

Report alongside it, plainly: how many passages were corrected, which tasks were
raised (each as a link), and which findings were left standing.

On a criterion, report the **reworded fields** — what each said before and what it
says now — and name the requirement whose version rolled. A reader who pressed a
control on one criterion needs to be told plainly that its requirement moved.

Where the run wrote nothing at all, say that, and say why — see
[The branches](#the-branches).

## The re-review

Last, after the publish: **`requestReview({ reference })`** — `DES-70` on a
design, and the **qualified** `REQ-4/AC-3` on a criterion.

The qualified form judges that one criterion. Passing the bare requirement
reference instead would judge every criterion it owns, which is a review the
reader did not ask for and a bill they did not agree to; passing a bare `AC-3`
resolves to nothing at all.

It judges the version that now exists; asked any earlier it would judge the one
being replaced. It cannot be refused as a duplicate here, because the publish
made a new version and the subject therefore differs — a criterion's verdict is
dated by its **owning requirement's** version, which publishing the correction rolled.

A review writes nothing to the entity — no content, no version, no status, no
approval — so it is not a gated write and it does not differ between the modes.
Tell the reader it is running: the verdict block on the page they came from
shows it.

## The branches

| The run | What to do |
|---|---|
| **Tasks only** — no correction | **No publish and no re-review.** The document did not change, so its verdict still describes it. Report the tasks raised, and stop. |
| **Every finding skipped** | **Nothing at all** — no edit, no task, no comment, no publish, no review. Say so plainly ("you left all four standing, so I have written nothing; the next review will reach them again") rather than reporting an empty success. |
| **The confirm is rejected as stale** | Someone published the document during the run. **Re-propose against the new head and put the fresh cascade to the reader. Never re-confirm against the old proposal**, and never assume the outcomes are unchanged — something moved, which is why it was rejected. |
| **The reader declines the publish** | The corrections stay in the live draft. Nothing is lost, no version exists, and the reader can publish them later themselves. Report it as that, never as a failure. |
| **The reader stops part-way** | What had landed is landed, and nothing else. A draft is not a version, so nothing was published and nothing invalidated. |
| **An edit batch is rejected** | Nothing was applied. Re-search for the passage and resend the whole batch; don't split it up to get part of it through. |
| **The document is frozen by its lifecycle status** | The corrections are refused by the write path, in the ordinary way and with the ordinary message. Say which findings could not be settled that way — any tasks and comments still stand. |
| **The conversation is resumed later** | Every decision is a turn in it. Read back what was already decided and carry on from there rather than asking again. |
| **A criterion, and the reader rewords it** | One `updateAcceptanceCriterion`, then one `publishDocument` of the owning requirement and its `confirmPublish`. There is no `editDocument` — reaching for it means you are editing the wrong thing. |
| **A criterion, and nothing fulfils it** | There is no design to raise a task beneath. Say so and put it to the reader — a design to raise it under, or leave the finding. Never invent one. |
| **A criterion whose points describe different outcomes** | The criterion is doing two jobs. Say so, settle the finding as one, and offer splitting it into two criteria as separate work — never as a fourth disposition. |
| **A criterion, and the reader leaves it** | Nothing at all, exactly as on a design — no reword, no task, no publish, no review. |

## What differs between the reader's two modes

The run has no mode of its own; it inherits the reader's. **Only *when* they
meet the consequence changes.**

The plan is stated in both modes and gates neither, so it is not a row in this
table — it is not a write, and writes are all the mode governs.

| Step | Approval mode | Auto mode |
|---|---|---|
| Every per-finding question | asked one at a time, answered by choosing | **identical** |
| Each correction | an approval card before it lands | applied to the draft as it goes |
| The task and its comment | two more approval cards | written as they go |
| The publish | the cascade is on the confirm card, **before** it is applied | published, then the cascade reported in the turn |
| The re-review | identical — a review writes nothing to the entity | identical |

You do not implement that table; the write gate does. What you must not do is
change **your own** behaviour because of the mode — same questions, same order,
same single publish, and the cascade reported either way.

## Five things never to do

**Never split a criterion's verdict into several findings.** Its points are the
evidence behind one answer, not separable claims — they carry no label because
there is no per-point verdict to give them. Asking four questions asks the reader
to decide the same thing four times, and every answer settles the same
given/when/then. One subject, one finding, one question, one correction.

**Never settle one finding and then ask the next.** Four findings settled in
turn are four publishes and four cascades, each with its own chance to
invalidate an approval a single version would have spared. Collect every
decision first; it costs the reader nothing and it is the whole reason the
conversation has a shape.

**Never anchor a comment before the corrections land.** `quotedText` resolves
against the draft that will be published. A thread created first can end up
anchored to a passage the same batch rewrote, or fail to resolve at all.

**Never use `askChoice` to ask permission.** The approval gate asks that, in the
reader's own mode. A choice is a decision the reader owns — *which of these two
is wrong* — and using one to ask *may I write this* would gate a write in auto
mode, which is the one thing auto mode says it will not do.

**Never ask the reader to confirm the plan.** It is a statement of what they
have already decided, one finding at a time, and pausing on it gates the whole
set in a mode that says it will not gate writes — the same reasoning as the
prohibition above. Approval mode puts each correction, task and comment to them
on its own card moments later, against the actual prose; auto mode is them
saying they do not want those cards.

## Process checklist

### A design

1. `getEntity` and `readDocument` on the design; take the finding from the
   verdict, and drop the points whose label says the document and the software
   agree.
2. Report what you read, including a difference from the count in the ask.
3. One `askChoice` per finding, three options, one at a time, all of them.
4. Write nothing until the last one is decided.
5. State the plan as a count per disposition, then carry on to the corrections.
6. Read the cited code, `searchDocument`, then `editDocument` — every correction
   into the draft, no publish.
7. `createTask` per finding the reader raised work on, then
   `createCommentThread` anchored at the passage, linking it.
8. If any correction landed: `publishDocument`, review the cascade,
   `confirmPublish`. Once.
9. Report which descendants lost their approval and which kept it, the passages
   corrected, the tasks raised and the findings left standing.
10. `requestReview` on the design, and say it is running.

### A criterion

1. `getEntity` on the owning requirement, then `getRequirementCriteria` for the
   criterion — its three fields, its `fulfilledBy` and its verdict.
2. Report what you read. **One finding, whatever `points` contains.**
3. **One** `askChoice`, three options, carrying the whole finding as context.
4. Write nothing until it is answered.
5. Say in one sentence what you are about to do, then carry on.
6. Reworded: read the cited code, then `updateAcceptanceCriterion` — the fields
   the finding touches, held to `spexd-authoring`'s Acceptance Criteria rules.
7. Work raised: `createTask` beneath a design that fulfils it, naming the
   criterion and carrying the finding. No comment.
8. If it was reworded: `publishDocument` on the owning requirement, read the
   proposal — **its depth-0 outcome is the requirement** — then
   `confirmPublish`. Once.
9. Report the fields as they now read, which requirement's version rolled, and
   which of its descendants lost or kept their approval.
10. `requestReview({ reference: 'REQ-4/AC-3' })` — the qualified form — and say
    it is running.
