# mgm-projectinit

A Claude Code skill that bootstraps new software projects with a standard mgm setup:
structured CLAUDE.md, permissions policy, modular roles, workflow procedures, git discipline,
and session hooks — ready for Phase 1 in one command.

---

## Requirements

- [Claude Code](https://claude.ai/code) ≥ 1.0
- Node.js ≥ 18 (for hooks and interactive config script)
- `jq` (for session hooks)
- `bash` ≥ 3.2
- `git` with authenticated access to your target GitHub repository

---

## Installation

Copy the skill file to your Claude Code user skills directory:

```bash
mkdir -p ~/.claude/skills/mgm-projectinit
cp SKILL.md ~/.claude/skills/mgm-projectinit/SKILL.md
```

Claude Code picks up skills automatically from `~/.claude/skills/`.

---

## Usage

In any Claude Code session, run:

```
/mgm-projectinit
```

The skill runs in two stages:

**Bootstrap (Steps 1–11)** — for a new, empty repository:
- Interviews you for project identity (name, prefix, developer, repo URL)
- Verifies repository reachability before touching any file
- Writes `CLAUDE.md`, `.gitignore`, `.claudeignore`, `.claude/settings.json`, shell rules,
  skill stubs, docs scaffolding, `CHANGELOG.md`, and `CLAUDESETUP.md`
- Commits, pushes, and tags `v0.0-bootstrap`
- Clears context for Phase 1

**Extended configuration (Phases A–H)** — run on new or existing projects:

| Phase | What it does |
|-------|-------------|
| A | Interactive domain/role/MCP selection |
| B | Writes `.claude/project-state.json` |
| C | Activates skills and MCPs in settings |
| D | Creates modular `roles/`, `workflow/`, `context/` structure |
| E | Initializes `CHANGELOG.md` if absent |
| F | Installs `session_start.sh` hook (project state on every session) |
| F.5 | Commits configuration bootstrap |
| G | Prints project summary |
| H | Planning mode — generates and confirms `PHASES.md` with sub-phase breakdown |

---

---

## Phase & Sub-phase approach

The skill structures every project as a sequence of **phases**, each broken into **sub-phases**.

### Phases

A phase is a high-level milestone with a named goal — for example: _Foundation_, _UI_, _Integration_, _Hardening_. Phases are planned upfront in `PHASES.md` (Planning Mode, Phase H) and confirmed with you before implementation begins. On phase completion, the result is merged to `main`, tagged, and context is cleared for the next phase.

### Sub-phases

A sub-phase is the atomic unit of work within a phase — small enough to complete in one focused session (target: S < 2h, M 2–4h, L 4–6h). Each sub-phase has:

- A **scope** (one sentence — what changes)
- A **deliverable** (testable artifact: endpoint, component, passing test suite)
- **Acceptance criteria** (2–4 concrete checkboxes)
- Its own **git branch** (`phase-N-Na`)
- A full **review cycle** via an independent sub-agent before the commit

> **Why not "tasks"?**
> A task implies a granular to-do item — a single function, a test case, a config value.
> A sub-phase carries architectural weight: it has a branch, a reviewer, a commit, and a CHANGELOG entry.
> Within a sub-phase, Claude Code's built-in task tracking handles granular implementation steps.
> Hierarchy: **Phase → Sub-phase → Task** (internal session steps).

### Sub-phase lifecycle

```
1.  git checkout -b phase-N-Na
2.  Read CLAUDE.md + relevant .claude/resources/ files
3.  Discuss scope before writing any code — /architect if cross-layer
4.  Implement + tests (tasks tracked internally)
5.  /tester → prioritized test recommendations
6.  Spawn sub-agent reviewer (no shared context) → docs/reviews/phase-Na.md
7.  Resolve all BLOCKER findings
8.  /compact  — compress context before commit
9.  Append entry to CHANGELOG.md
10. git commit -m "[phase-Na] …"  then push
```

State is tracked in `.claude/project-state.json` — current phase, sub-phase, active role, pending reviews.

---

## Project roles

Each role gives Claude a focused lens and a specific activation checklist. Only roles selected during setup (Phase A) are active. The default role is `/developer`.

| Role | Focus | Activated automatically when… |
|------|-------|-------------------------------|
| `/architect` | System design, API contracts, ADRs, tech debt | New endpoints, schema changes, new modules, new dependencies, external integrations |
| `/developer` | Implementation quality, patterns, DRY, code review | Default — always active unless switched |
| `/devops` | CI/CD, deployment, infra, secrets, observability | Dockerfile changes, new env vars, new cloud resources |
| `/security` | Threat modeling, OWASP Top 10, auth/authz, data exposure | Auth changes, new data ingestion, file upload/download, sensitive data |
| `/tester` | Coverage gaps, edge cases, regression risk, boundary conditions | Sub-phase completion — recommendations only, not blockers |
| `/fach` | Domain terminology, process compliance, regulatory requirements | Domain-specific business logic, legal/regulatory constraints |

Roles are defined as separate files in `.claude/roles/`. Switching roles loads the file and adopts the lens — the previous role's context is not lost, just deprioritized. Roles can also be suggested proactively: Claude prompts for `/architect` before touching an API contract, for `/security` before auth code, and so on.

On activation, each role reads a specific context file:
- `/architect` → reads `.claude/context/decisions.md` (ADRs) — avoids re-debating settled choices
- `/developer` → reads `.claude/context/known-issues.md` — applies known fixes proactively
- `/fach` → reads `project-state.json → fach_context` — domain-specific constraints

---

## Token efficiency, security, and fewer interruptions

A long-running AI-assisted project accumulates context fast. This skill applies several complementary strategies to keep token consumption low, security high, and clarification prompts rare.

### Context management

| Mechanism | What it does | When it runs |
|-----------|-------------|-------------|
| `/compact` | Compresses conversation history to a dense summary | After each sub-phase, before commit |
| `/clear` | Resets context entirely | At phase start — previous phase context is no longer needed |
| On-demand resource files | `.claude/resources/` files are referenced but not auto-loaded | Only when the relevant area is touched |
| Modular role files | Role definitions live in `.claude/roles/` — not inlined in CLAUDE.md | On role activation only |
| Modular workflow files | Procedure files live in `.claude/workflow/` | On phase/sub-phase events |

CLAUDE.md itself is kept short — it is a dispatcher that points to on-demand files, not a monolith that loads everything at session start.

### Session continuity without re-explanation

| Mechanism | What it does |
|-----------|-------------|
| `session_start.sh` hook | Injects current role, phase, sub-phase, domains, and pending reviews at every session start |
| `project-state.json` | Single source of truth — Claude reads it instead of asking "where were we?" |
| `.claude/context/decisions.md` | Architectural decisions (ADRs) read on `/architect` activation — settled choices are not re-debated |
| `.claude/context/known-issues.md` | Known bug fixes read on `/developer` activation — the same bug is not diagnosed twice |
| `docs/memory/phase-Na.md` | Design decisions and gotchas written after each sub-phase — available in future sessions |

### Security

| Mechanism | What it protects |
|-----------|-----------------|
| `guard_readonly.sh` | Blocks `rm`/`mv`/`cp` outside the project root; blocks curl/wget mutations without confirmation; blocks file output to external paths |
| Permissions deny list | Blocks reads of `.env`, secrets, keys, wolf internals; blocks writes to build dirs, caches, and generated outputs |
| Shell rules (`shell-rules.md`) | Enforces single atomic Bash calls — no pipes, `&&`, `;`, `$()` — prevents command injection via chaining |
| No secrets in repo | `.gitignore` excludes `.env`, `*.key`, `*.pem`; deny list enforces this at tool level |

The `guard_readonly.sh` hook uses the correct Claude Code hook protocol: `exit 0` (no output) to allow, `{"decision":"block","reason":"..."}` to block. Outputting `{"decision":"allow"}` causes a validation error — this is a common pitfall documented explicitly in the skill.

### Fewer interruptions and back-and-forth

| Mechanism | How it reduces interruptions |
|-----------|----------------------------|
| Permissions allow list | `ls`, `grep`, `git log`, `git diff`, `git status`, `find`, `cat`, `node`, `npm run` are auto-approved — no prompts for read-only operations |
| Planning mode (Phase H) | Scope, acceptance criteria, and size estimates are agreed upfront — no mid-session scope debates |
| Role-specific checklists | Each role has a concrete activation checklist — Claude knows exactly what to read and check, without asking |
| `pending_reviews` field | Surfaces open review blockers at session start via the hook — not mid-task |
| `guard_readonly.sh` curl confirmation | Mutation requests prompt the user via `/dev/tty` with `[y/N]` — a clean decision point, not a tool failure |
| Independent sub-agent reviewer | The reviewer runs in a separate context with no shared conversation state — its findings are written to `docs/reviews/`, not injected back into the main context |
| Haiku/Sonnet/Opus routing | Simple/mechanical tasks use Haiku; standard development uses Sonnet; Opus only with explicit user approval — avoids over-spending tokens on routine work |

---

## What gets created

```
project/
├── CLAUDE.md                        # Project system prompt (dispatcher)
├── CLAUDESETUP.md                   # Setup reference (/summary to view)
├── CHANGELOG.md                     # Phase-level changelog
├── PHASES.md                        # Sub-phase plan (after Phase H)
├── .gitignore
├── .claudeignore
├── .claude/
│   ├── settings.json                # Permissions + hooks
│   ├── project-state.json           # Active role, phase, domains
│   ├── rules/
│   │   └── shell-rules.md           # Atomic Bash rule
│   ├── hooks/
│   │   ├── session_start.sh         # Project state on session start
│   │   └── guard_readonly.sh        # curl/wget/rm/mv/cp guard
│   ├── roles/                       # architect, developer, devops, security, tester, fach
│   ├── workflow/                    # ground-rules, checkpoints, changelog, subproject-workflow
│   ├── context/                     # decisions (ADRs), known-issues, project-plan
│   ├── skills/
│   │   ├── phase-review.md          # Independent sub-agent reviewer
│   │   └── phase-commit.md          # Pre-commit checklist
│   └── commands/
│       ├── summary.md               # /summary
│       └── defaults.md              # /defaults
├── docs/
│   ├── memory/                      # Per-phase design decisions
│   └── reviews/                     # Sub-agent review reports
└── skill/
    └── <prefix>-skill.md            # Project skill stub
```

---

## Hook protocol note

PreToolUse hooks must use `exit 0` (no output) to allow — not `{"decision":"allow"}`.
The `guard_readonly.sh` script enforces this protocol. All hook scripts must be `chmod 755`.
Homebrew binaries (`node`, etc.) require absolute paths in hook commands since Claude Code
runs hooks with a restricted PATH (`/usr/local/bin:/usr/bin:/bin`).

---

## License

MIT
