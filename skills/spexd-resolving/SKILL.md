---
name: spexd-resolving
description: >-
  Settle the findings of a completed Spexd review on a design — put every
  finding to the reader as a choice, correct the document where the software is
  right, raise work where the software is wrong, publish the whole set as one
  version, and ask for the design to be judged again, via the Spexd MCP tools
  (mcp__Spexd__*). Use when a turn hands the work over in words like "Resolve
  the 4 findings on the latest review of DES-70", or whenever someone asks to
  resolve, settle, act on, clear or fix the findings, points or drift a review
  reported on a design. Distinct from requestReview, which produces the
  findings, and from spexd-auditing, which audits a feature against the code by
  hand.
user-invocable: true
---

# Resolving a review's findings

A completed review says the document and the software disagree. **It never says
which of the two is wrong**, and neither do you. Every finding is put to the
reader, the reader decides it, and you carry the decisions out.

There are three dispositions per finding and only three:

| Disposition | The reader is saying | What gets written |
|---|---|---|
| **Resolve in favour of the code** | the software is right, the document is stale | the passage is rewritten in the live draft |
| **Raise a task to fix** | the document is right, the software is wrong | a task beneath the design, plus a comment anchored at the passage linking to it |
| **Skip it** | not ready to settle this one | nothing at all |

**Skipping writes nothing, and that is the design rather than an omission.**
There is no resolved-by-a-reader state, and inventing one would make a finding
that one reader dismissed invisible to the colleague they are about to discuss
it with. A skipped finding stands, and the next review reaches it again.

## When to use this

- A hand-off turn: *Resolve the 4 findings on the latest review of DES-70.*
- "Act on those findings", "fix the drift the review found", "the review says
  this design is stale — sort it out".
- **Not** for running a review. That is `requestReview`, one call.
- **Not** for auditing a whole feature against the code by hand. That is
  `spexd-auditing`.

Only a **design** is resolvable this way. A finding on an acceptance criterion
is a different subject with a different consequence — correcting a criterion
rewrites a given/when/then and publishes its *owning requirement* — and is
deliberately out of scope. If asked, say so rather than improvising it.

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
   spared.
3. **State the plan, with a count per disposition, and carry straight on.**
   *Because the arithmetic is the one thing the reader cannot check by eye.*
   They have just decided each finding one at a time; the total is the part
   they have not seen. Don't stop for a go-ahead: the last moment at which
   nothing is irreversible is the publish, not the plan, because everything
   before it lands in the live draft and a draft is not a version.
4. **Apply every correction to the live draft.** All of them. No publish yet.
5. **Raise each task.**
6. **Anchor each task's comment.** *Corrections before tasks, tasks before
   comments.* A comment anchors by exact quoted text against the document that
   will be published, so a thread created first can be left pointing at a
   passage the same batch then rewrote.
7. **Publish once** — propose, then confirm. *Nothing before this point is
   irreversible*, which is what makes stopping safe at every earlier step.
8. **Report what the cascade did.**
9. **Ask for the re-review.** *Last, because it judges the version that now
   exists.* Asked any earlier it would judge the one being replaced.

## Reading the finding

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

## Asking a finding

One `askChoice` per finding. The finding is the context; the options are the
decision.

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

**It is a statement, not a gate.** Never ask the reader to confirm the plan, in
either mode. It restates decisions they have just made one by one, and whether a
*write* needs their permission is the write gate's question rather than yours —
the plan is not a write, so there is nothing for a gate to hold.

## Correcting a passage

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

## Raising work

For each finding the reader settled by raising work, in this order.

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

A raised task's thread is never closed by this procedure. It is the record of
why the task exists, and it stands until a person resolves it — resolving a
thread is human-only.

## Publishing

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

## Reporting

After the publish, **in every mode**, tell the reader **which descendants lost
their approval and which kept it** — each named, with the reason the proposal
gave. A count is not enough: every invalidated descendant is now something a
person has to approve again, and they need to know which.

Report alongside it, plainly: how many passages were corrected, which tasks were
raised (each as a link), and which findings were left standing.

Where the run wrote nothing at all, say that, and say why — see
[The branches](#the-branches).

## The re-review

Last, after the publish: **`requestReview({ reference })`** on the design.

It judges the version that now exists; asked any earlier it would judge the one
being replaced. It cannot be refused as a duplicate here, because the publish
made a new version and the subject therefore differs.

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

## Four things never to do

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
