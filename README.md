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
