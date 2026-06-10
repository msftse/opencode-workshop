# OpenCode Flow

A structured, opinionated workflow for building software with [OpenCode](https://opencode.ai). It takes you from an idea, through a PRD and project rules, into context-rich plans and one-pass implementations — and gives every new session a clean way to load project context.

---

## TL;DR — The Flow

```
                           ┌────────────────────────┐
       Conversation ─────► │   /create-prd PRD.md   │  ── PRD.md
                           └─────────────┬──────────┘
                                         │
                                         ▼
                           ┌────────────────────────┐
       Existing codebase ► │     /create-rules      │  ── AGENTS.md
                           └─────────────┬──────────┘
                                         │
                                         ▼
                           ┌────────────────────────┐
       Feature request ──► │ /plan-feature <name>   │  ── .agents/plans/<name>.md
                           └─────────────┬──────────┘
                                         │
                                         ▼
                           ┌────────────────────────┐
                           │  /execute <plan-path>  │  ── code + tests
                           └─────────────┬──────────┘
                                         │
                                         ▼
                           ┌────────────────────────┐
                           │        /commit         │  ── atomic commit
                           └────────────────────────┘

       New session?  ───►  /prime   (load project context first)
```

---

## What's in this repo

```
.opencode/
├── AGENTS-template.md   # Template used by /create-rules
├── agents/              # Specialized subagents (codebase-mapper, researcher, security-auditor)
├── commands/            # Slash commands that drive the flow
│   ├── prime.md
│   ├── create-prd.md
│   ├── create-rules.md
│   ├── init-project.md
│   ├── plan-feature.md
│   ├── execute.md
│   └── commit.md
└── skills/              # Skills auto-loaded for matching tasks
    ├── agent-browser/
    ├── doc-coauthoring/
    ├── e2e-test/
    └── excalidraw-diagram/
```

Everything is defined as plain Markdown — commands, agents, and skills are all editable in place.

---

## Core Philosophy

1. **Context is king.** A plan must contain everything the implementation agent needs — patterns, files to read, validation commands — so it can succeed on the first attempt.
2. **Plan before code.** `/plan-feature` never writes implementation code. It produces a plan rich enough for `/execute` to follow blindly.
3. **Rules over reminders.** Project conventions live in `AGENTS.md`, not in every prompt.
4. **Prime every session.** New sessions start by loading context, not guessing.

---

## The Flow, Step by Step

### 1. `/create-prd [output-filename]` — Capture the product

Use this at the very start of a new product or major initiative.

- **Input:** the current conversation (requirements, goals, constraints).
- **Output:** a comprehensive PRD (default `PRD.md`) with 15 standard sections — executive summary, mission, target users, MVP scope, user stories, architecture, tech stack, security, success criteria, implementation phases, risks, etc.
- **When to use:** before any code exists, or when scoping a brand-new product surface.

```
/create-prd PRD.md
```

The command will ask clarifying questions if critical information is missing, then write a single Markdown file you can review and refine.

---

### 2. `/create-rules` — Generate `AGENTS.md`

Once a codebase (even a scaffold) exists, generate the global rules file that every future agent session will read.

- **Input:** the current codebase (configs, directory layout, existing patterns).
- **Output:** `AGENTS.md` at the project root, based on `.opencode/AGENTS-template.md`.
- **Covers:** project overview, tech stack, dev/build/test commands, structure, naming and error-handling patterns, key files.

```
/create-rules
```

`AGENTS.md` is the single source of truth for "how we work here." Keep it short and scannable — link out to deeper docs instead of duplicating them.

---

### 3. `/init-project` — Bootstrap the local environment

A project-specific runbook for getting the app running locally (env file, deps, database, migrations, dev server, health checks).

```
/init-project
```

Edit `.opencode/commands/init-project.md` to match your stack — it ships as a reference example.

---

### 4. `/plan-feature <feature description>` — Build a one-pass plan

This is where most of your time should be spent. The command runs a 5-phase planning process:

1. **Feature understanding** — problem, value, complexity, user story.
2. **Codebase intelligence** — structure, patterns, dependencies, tests, integration points (uses subagents in parallel).
3. **External research** — official docs, best practices, gotchas, with linked references.
4. **Strategic thinking** — trade-offs, edge cases, performance, security.
5. **Plan generation** — fills the full template: context references, phased implementation plan, step-by-step tasks with `IMPLEMENT / PATTERN / IMPORTS / GOTCHA / VALIDATE`, testing strategy, validation commands, acceptance criteria.

```
/plan-feature add user authentication
```

- **Output:** `.agents/plans/<kebab-case-name>.md` (e.g. `add-user-authentication.md`).
- **Rule:** no code is written in this phase. The plan is the deliverable.

A good plan passes the "no prior knowledge" test — a fresh agent should be able to implement it using only the plan.

---

### 5. `/execute <path-to-plan>` — Implement from the plan

Hand the plan to an execution session.

```
/execute .agents/plans/add-user-authentication.md
```

The execute command:

1. Reads the entire plan and understands task dependencies.
2. Works tasks in order, mirroring referenced patterns.
3. Implements the testing strategy.
4. Runs every validation command from the plan; fixes failures before moving on.
5. Reports completed tasks, created/modified files, test results, and validation output.

Deviations from the plan are documented in the report rather than silently applied.

---

### 6. `/commit` — Atomic commit

When the execute report says "Ready for Commit," wrap it up:

```
/commit
```

This inspects `git status` / `git diff HEAD`, stages the relevant files, and creates an atomic commit with a conventional tag (`feat`, `fix`, `docs`, …).

---

## Starting a New Session: `/prime`

Every new OpenCode session starts with **zero project context**. Run `/prime` first.

```
/prime
```

`prime` will:

- List tracked files (`git ls-files`) and show the directory tree.
- Read `PRD.md`, `AGENTS.md` (or `CLAUDE.md`), root and major-dir READMEs, architecture docs, and DB/schema configs.
- Read entry points and core configs (`package.json`, `pyproject.toml`, `tsconfig.json`, etc.).
- Check recent git activity (`git log -10 --oneline`, `git status`).
- Return a concise summary: project overview, architecture, tech stack, conventions, current state.

Use it whenever:

- You open a fresh session.
- You switch to a branch you haven't worked on recently.
- You hand off context to a teammate or a new agent.

---

## Recommended End-to-End Flow

**Greenfield project**

```
1. /create-prd PRD.md              # capture the product
2. /init-project                   # (after scaffold exists) verify local setup
3. /create-rules                   # generate AGENTS.md once the scaffold is in
4. /plan-feature <first slice>     # plan the first vertical slice
5. /execute .agents/plans/<...>.md # implement it
6. /commit                         # ship it
```

**Existing project, new feature**

```
1. /prime                          # load context
2. /plan-feature <feature>         # produce the plan
3. /execute .agents/plans/<...>.md # implement
4. /commit                         # ship it
```

**Coming back the next day**

```
1. /prime
2. continue from where /execute left off, or start /plan-feature for the next slice
```

---

## Working with Phases

`/plan-feature` always structures work into four implementation phases:

1. **Foundation** — schemas, types, configuration, helpers.
2. **Core Implementation** — business logic, services, models, endpoints.
3. **Integration** — wire into existing routers/handlers/middleware/config.
4. **Testing & Validation** — unit, integration, edge cases, acceptance.

Inside the phases, every task uses information-dense verbs (`CREATE`, `UPDATE`, `ADD`, `REMOVE`, `REFACTOR`, `MIRROR`) and includes:

- `IMPLEMENT` — what to do.
- `PATTERN` — the file:line to mirror.
- `IMPORTS` — required imports.
- `GOTCHA` — constraints to avoid.
- `VALIDATE` — an executable command.

`/execute` follows these top-to-bottom and validates as it goes. If validation fails, it fixes the issue and re-runs before moving on.

---

## Agents and Skills

Beyond the commands above, the flow uses:

**Subagents** (in `.opencode/agents/`)

- `codebase-mapper` — high-level structural overview of a repo.
- `researcher` — best-practice and library research.
- `security-auditor` — vulnerability and security review.

**Skills** (in `.opencode/skills/`) auto-load when a task matches their description.

- `agent-browser` — browser automation, screenshots, form filling, scraping.
- `doc-coauthoring` — structured doc-writing workflow.
- `e2e-test` — full end-to-end test runs with browser + DB validation.
- `excalidraw-diagram` — produce Excalidraw diagrams.

You don't normally invoke these directly — `/plan-feature` and `/execute` will pull them in as needed, and skills load automatically.

---

## Tips

- Keep `AGENTS.md` short. Link to deeper docs rather than duplicating them.
- Re-run `/create-rules` (or hand-edit `AGENTS.md`) when conventions drift.
- Treat plans in `.agents/plans/` as artifacts — commit them so future sessions can reference them.
- Don't skip `/prime` on new sessions. The minute you save is paid back many times over.
- If a plan feels thin, ask `/plan-feature` to revise — better to iterate on the plan than to debug a bad execution.
