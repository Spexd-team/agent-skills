# Spexd Agent Skills

Agent skills for working with [Spexd](https://www.spexd.com) — the
specification and traceability platform that models product work as a strict
chain of **Feature → Requirement → Acceptance Criteria → Design → Task**.

These skills teach AI coding agents (Claude Code, and any agent that supports
the [`skills`](https://github.com/vercel-labs/skills) format) how to read from
and write to Spexd through its MCP server, so the spec and the code stay in
sync.

## What's in here

| Skill | What it's for |
|---|---|
| [`spexd-authoring`](skills/spexd-authoring/SKILL.md) | **Building the spec.** Author and curate the whole traceability chain — features, requirements, acceptance criteria, designs, and tasks — placing each entity at the right altitude so implementation detail never leaks upward into a feature or requirement. Use it to populate Spexd, reorganise entities, or import work from tickets, docs, and code. |
| [`spexd-importing`](skills/spexd-importing/SKILL.md) | **Bootstrapping the spec from an existing project.** Reverse-engineer a whole Spexd project from a codebase and its supporting sources: discover the feature set from evidence, fan out one sub-agent per feature to derive requirements and acceptance criteria, then design across the requirements — all raised as `DRAFT` for human review. Discovery-first and evidence-grounded, building on `spexd-authoring` for altitude. |
| [`spexd-implementing`](skills/spexd-implementing/SKILL.md) | **Executing the spec.** Pull a task and its full ancestor chain (task → design → acceptance criteria → requirement → feature) as context, build to that spec without changing it, and reflect progress back onto the task's lifecycle status. Use it to pick up, build, or ship work that's tracked in Spexd. |
| [`spexd-auditing`](skills/spexd-auditing/SKILL.md) | **Checking the spec against the code.** Audit a feature or requirement on `main`: verdict every acceptance criterion, hold each one to a test that asserts its own stated outcome, and flag designs whose prose no longer matches what shipped. Use it to answer "is FEAT-3 actually done", "are all the ACs met", or "do the designs still reflect reality". |

The four are complements: `spexd-importing` bootstraps a spec from a project
that already exists; `spexd-authoring` decides *what* to build and *how*;
`spexd-implementing` carries out a decision that has already been made; and
`spexd-auditing` checks afterwards whether the spec and the shipped code still
agree. All operate over the Spexd MCP tools (`mcp__Spexd__*`).

## Requirements

- An agent that supports installable skills (e.g. [Claude Code](https://www.anthropic.com/claude-code)).
- The **Spexd MCP server** connected to your agent, so the `mcp__Spexd__*`
  tools are available. See [spexd.com](https://www.spexd.com) for access and
  setup.

## Installation

Install the skills into your project (or agent) with the
[`skills`](https://www.npmjs.com/package/skills) CLI:

```bash
# Install everything in this repo
npx skills add spexd-team/agent-skills

# List the available skills without installing
npx skills add spexd-team/agent-skills --list

# Or install a single skill
npx skills add spexd-team/agent-skills --skill spexd-authoring
npx skills add spexd-team/agent-skills --skill spexd-importing
npx skills add spexd-team/agent-skills --skill spexd-implementing
npx skills add spexd-team/agent-skills --skill spexd-auditing
```

Once installed, invoke a skill by name (for example `/spexd-authoring`) or let
your agent select it automatically when a task matches — every skill is
`user-invocable`.

## The core idea: detail only descends

Spexd's traceability chain works because each level answers a different
question, and the answers get more concrete as you go down:

| Level | Answers | Altitude |
|---|---|---|
| **Feature** | *What can someone do, and why does it matter?* | Product scope |
| **Requirement** | *What must be true within that capability?* | Behavioural rule |
| **Acceptance Criteria** | *How will we know it's true?* | Testable condition |
| **Design** | *How do we build it so those conditions hold?* | Architecture & mechanism |
| **Task** | *What work ships a slice of that design?* | Unit of work |

Feature and Requirement are the *what* and the *why*; Design and Task are the
*how*. Implementation detail — vendors, libraries, data models, algorithms — is
introduced at the **Design** level and nowhere above it. These skills encode
that classification lens so entities land where they belong and stay
traceable from product intent all the way down to shipped code.

The implementation itself is the exception that stops at the bottom: **no
entity contains the code**, not even a design or a task. Designs express data
models and contracts as human-readable tables — they are read by product and
engineering alike — and both designs and tasks are grounded in analysis of the
codebase they will change, quoting the code that already exists rather than
writing the code that doesn't yet.

## Contributing

Each skill lives in its own directory under [`skills/`](skills/) with a
`SKILL.md` that carries YAML front matter (`name`, `description`,
`user-invocable`) followed by the skill body. To add a skill, create a new
directory following the same structure.

## License

Contact the [Spexd team](https://www.spexd.com) regarding usage.
