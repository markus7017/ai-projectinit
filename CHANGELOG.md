# CHANGELOG — mgm-projectinit

All notable changes to this skill are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.0.0] — 2026-05-22

### Added
- Bootstrap sequence (Steps 1–11): git init, identity, conflict resolution, standard file creation, commit, push, tag
- Extended configuration Phases A–H: interactive domain/role selection, project-state.json, modular structure, session hook, planning mode
- Modular `.claude/roles/` — architect, developer, devops, security, tester, fach
- Modular `.claude/workflow/` — ground-rules, checkpoints, changelog format, subproject-workflow
- Modular `.claude/context/` — ADR scaffold (decisions.md), known-issues.md, project-plan.md
- Interactive configuration script (`project_config.mjs`) with TTY/AskUserQuestion fallback
- `session_start.sh` hook — prints project state, ground rules, ADR/issue/phase counts on every session
- `guard_readonly.sh` PreToolUse hook — blocks rm/mv/cp outside project root, curl/wget mutations
- Claude Code hook protocol documentation (allow = silent exit 0, not `{"decision":"allow"}`)
- Phase H planning mode: constraint collection, phase confirmation loop, sub-phase generation with size estimates and acceptance criteria
- `.gitignore` with Claude Code runtime exclusions and negation block for versioned `.claude/` subdirectories
- `CLAUDESETUP.md` generation: compact setup reference with permissions, hooks, model routing, git conventions
- `/summary` and `/defaults` command stubs
- Abort conditions on repo reachability, commit failure, and push failure

### Notes
- Node.js ≥ 18 required for hooks and interactive config; use absolute paths for Homebrew binaries
- All extended configuration phases (A–H) are idempotent — safe to re-run on existing projects
