# mgm-projectinit — New Project Initialization Skill

Initializes a new project repository with the standard mgm Claude Code setup:
CLAUDE.md, .gitignore, .claudeignore, .claude/settings.json, rules, skills,
CHANGELOG.md, docs/, and skill/ directory. Tags `v0.0-bootstrap` **only after**
successful push. Clears context on completion.

---

## When to invoke

Use at the very start of a new project, in an empty or freshly cloned directory,
before any other work begins.

```
/mgm-projectinit
```

---

## Step 1 — Interview

Ask the user for all required values before touching any file or running any command.
Use AskUserQuestion for interactive input. Collect:

| Field | Example |
|-------|---------|
| Project display name | `AI DevChat TVP` |
| Short prefix (file/branch naming) | `aidc` |
| Developer full name | `Markus Michels` |
| Developer email | `markus.michels@mgm-tp.com` |
| GitHub repo URL | `https://github.com/markus7017/aidc.git` |

Do not proceed until all five values are confirmed.

---

## Step 2 — Verify repository reachability

Run as a **single atomic Bash call**:

```bash
git ls-remote <repo-url> HEAD
```

If this call fails (non-zero exit, connection refused, 404, auth error):

> **Initialization aborted.** Cannot reach repository: `<url>`
> Check the URL, your network connection, and GitHub authentication, then retry.

Stop completely. Do not create any files.

---

## Step 3 — Git init & identity

Run each as a separate atomic Bash call:

```bash
git init
git config user.name "<developer name>"
git config user.email "<developer email>"
git remote add origin <repo-url>
```

If `git remote add` fails because a remote already exists, use `git remote set-url origin <url>` instead.

---

## Step 4 — Conflict resolution

Before writing any file, check for conflicts with existing content:

- If `.claude/settings.json` already exists: merge permissions (deduplicate deny/allow arrays, preserve existing hooks, add new entries without removing others)
- If `.claude/rules/shell-rules.md` already exists: overwrite only if content differs from the canonical template below
- If `.gitignore` or `.claudeignore` already exist: append missing entries only — never remove existing entries
- If `CLAUDE.md` already exists: do NOT overwrite — report to user and skip
- If `CHANGELOG.md` already exists: prepend the Phase 0 entry at the top — do not replace existing entries
- For all other files (skill stubs, README templates, commands): overwrite silently

---

## Step 5 — Create standard files

Create every file below using Write tool calls (no Bash). Substitute `<PREFIX>`,
`<PROJECT_NAME>`, `<DEV_NAME>`, `<DEV_EMAIL>`, `<REPO_URL>` with the interview values.

### CLAUDE.md

```markdown
# <PROJECT_NAME> — Project Guide for Claude Code

Read this file at the start of every session. Load relevant resource files before
starting any phase. Never skip ahead.

---

## Resource Files

| File | Load when… |
|------|-----------|
| `.claude/resources/phases.md` | Starting any implementation phase/sub-phase |

Concept: `input/concept.md` | Summary: `input/summary.md`

---

## Project Identity

**Name**: <PROJECT_NAME> | Prefix: **<PREFIX>**
**Repo**: <REPO_URL>
**Developer**: <DEV_NAME> (`<DEV_EMAIL>`)

Brief description — fill in before Phase 1.

---

## Git Workflow

| Branch | Purpose |
|--------|---------|
| `main` | Stable — end of last completed phase |
| `phase-N` | Created at phase start; merged to main on completion |
| `phase-N-Na` | Created at sub-phase start |

- **Commit format**: `[phase-Na] description`
- **Tag format**: `v0.N-phaseN` after phase smoke test passes

---

## Ground Rules

1. Read CLAUDE.md + relevant resource files before writing any code
2. Discuss implementation before coding; ask when unclear
3. Complete each sub-phase fully before the next
4. Mark deferred work: `// TVP-FUTURE: <reason>`
5. No raw throws — errors as StatusCode objects (or equivalent)
6. TypeScript strict (or language equivalent)
7. Tests per phase
8. Prototype mindset: simple working solution first
9. Git: branch at start; reviewer → /compact → CHANGELOG → commit; push
10. Shell: single atomic Bash calls — no pipes, &&, ;, $()
11. Model routing: Haiku → simple; Sonnet → standard (default); Opus → complex + user approval
12. Plan Mode for cross-layer design or tasks touching >3 files
13. Auto /compact after each sub-phase; auto /clear when starting new phase
14. Spawn sub-agent reviewer at sub-phase completion → docs/reviews/phase-Na.md
15. Write docs/memory/phase-Na.md after each sub-phase (design decisions)
16. Bug fixes: read .wolf/buglog.json before fixing; log fix afterwards
17. CHANGELOG.md: append entry at each phase/sub-phase completion (timestamp + branch)

---

## Phase Roadmap

| Phase | Goal | Status |
|-------|------|--------|
| **0** ✅ | Repo bootstrap | done |
| **1** | — | next |

Sub-phase details: `.claude/resources/phases.md`

---

## Standard Sub-Phase Workflow

```
1. git checkout -b phase-N-Na
2. Read CLAUDE.md + relevant .claude/resources/ files
3. Discuss implementation before coding
4. Implement + tests
5. Spawn reviewer → docs/reviews/phase-Na.md
6. Fix blockers
7. /compact
8. Append to CHANGELOG.md
9. git commit -m "[phase-Na] …" then push
```

On phase completion: merge to main → tag → /clear before next phase.
```

### .gitignore

```
# macOS
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
Icon?

# Claude Code — session state; project config versioned via negation
.claude/*
!.claude/resources/
!.claude/rules/
!.claude/skills/
!.claude/commands/
!.claude/hooks/
!.claude/roles/
!.claude/workflow/
!.claude/context/
!.claude/settings.json
!.claude/project-state.json

# Claude Code — runtime / cache
.claude.json
.claude.json.backup.*
security_warnings_*.json
stats-cache.json
mcp-needs-auth-cache.json
history.jsonl
backups/
cache/
debug/
file-history/
paste-cache/
session-env/
shell-snapshots/
plans/
plugins/
tasks/
teams/
todos/
statsig/
telemetry/
usage-data/
ide/
projects/**/*.jsonl
projects/**/*.txt

# OpenWolf
.wolf/

# Dependencies
node_modules/
vendor/

# Build outputs
dist/
build/
out/
target/
.next/
.nuxt/
.output/
_site/
*.tsbuildinfo

# Test & coverage
coverage/
.nyc_output/
test-results/
playwright-report/
__pycache__/
.pytest_cache/
.cache/

# Secrets
.env
.env.local
.env.*.local

# Logs & temp
logs/
*.log
tmp/
temp/

# Editor / IDE
.idea/
.vscode/
*.swp
*.swo
*~

# OS
Thumbs.db
```

### .claudeignore

```
# Dependencies
node_modules/
vendor/
.pnp/
.yarn/

# Build outputs
dist/
build/
out/
target/
.next/
.nuxt/
.output/
_site/

# Minified / generated
*.min.js
*.min.css
*.bundle.js
*.chunk.js
*.map
*.generated.ts
*.generated.js

# Lock files
package-lock.json
yarn.lock
pnpm-lock.yaml
composer.lock
Gemfile.lock
poetry.lock

# Archives
*.zip
*.tar.gz
*.tar
*.jar

# Test artifacts
coverage/
.nyc_output/
test-results/
playwright-report/
__snapshots__/

# Caches
.cache/
.turbo/
.parcel-cache/
__pycache__/
*.pyc
.pytest_cache/

# Logs & temp
*.log
logs/
tmp/
temp/

# IDE
.vscode/
.idea/
*.iml

# OS
.DS_Store
Thumbs.db

# OpenWolf runtime
.wolf/*.json
.wolf/*.log
.wolf/hooks/
.wolf/memory.md
```

### .claude/settings.json

```json
{
  "mcpServers": {},
  "respectGitignore": true,
  "permissions": {
    "deny": [
      "Read(.env)", "Read(.env.*)", "Read(**/.env)", "Read(**/.env.*)",
      "Read(*.pem)", "Read(*.key)", "Read(*.p12)",
      "Read(**/secrets/**)", "Read(**/credentials/**)",
      "Read(.wolf/token-ledger.json)", "Read(.wolf/memory.md)",
      "Read(.wolf/config.json)", "Read(.wolf/*.log)", "Read(.wolf/hooks/**)",
      "Write(.env)", "Write(**/.env)", "Write(**/.env.*)", "Write(.DS_Store)",
      "Write(.wolf/**)", "Write(node_modules/**)",
      "Write(dist/**)", "Write(build/**)", "Write(out/**)", "Write(target/**)",
      "Write(.next/**)", "Write(.nuxt/**)",
      "Write(coverage/**)", "Write(.nyc_output/**)",
      "Write(__pycache__/**)", "Write(.cache/**)", "Write(.pytest_cache/**)",
      "Write(tmp/**)", "Write(temp/**)", "Write(logs/**)"
    ],
    "allow": [
      "Bash(ls *)", "Bash(ls)", "Bash(cat *)", "Bash(echo *)",
      "Bash(grep *)", "Bash(grep)", "Bash(find *)", "Bash(curl *)",
      "Bash(cd *)", "Bash(sed *)", "Bash(wc *)", "Bash(cp *)", "Bash(mkdir *)",
      "Bash(git status*)", "Bash(git log*)", "Bash(git diff*)",
      "Bash(git branch*)", "Bash(git remote*)", "Bash(git show*)",
      "Bash(npm run *)", "Bash(node *)",
      "Read(.claude/resources/**)", "Read(.claude/rules/**)",
      "Read(.claude/skills/**)", "Read(.claude/commands/**)",
      "Read(.claude/hooks/**)", "Read(.claude/settings.json)"
    ]
  }
}
```

### .claude/rules/shell-rules.md

```
Each Bash call must be a single atomic command — no pipes, chaining (&&, ||, ;),
command substitution ($(...)), or behavior-chaining redirections.

For multi-step tasks: execute sequential individual Bash calls.
For conditional logic: check exit code or output of the first call before proceeding.

Applies to all agents, subagents, skills, hooks, and prompts.
```

### CHANGELOG.md

```markdown
# CHANGELOG

Entries written at each phase/sub-phase completion.
Format: `## [phase-Na] YYYY-MM-DD | branch: <branch>`

---

## [phase-0] <TODAY> | branch: main — Repository Bootstrap

### 0a — Project initialization
Initial repository setup via mgm-projectinit skill.
Standard files: CLAUDE.md, .gitignore, .claudeignore, .claude/settings.json,
shell-rules, phase-review/phase-commit skills, CHANGELOG.md, docs scaffolding.
Tag: v0.0-bootstrap
```

### Empty placeholder files

Create with empty content:
- `.claude/commands/.gitkeep`
- `.claude/hooks/.gitkeep`
- `.claude/resources/.gitkeep`
- `docs/memory/README.md` — see template below
- `docs/reviews/README.md` — see template below
- `skill/<PREFIX>-skill.md` — minimal stub (fill project identity, update each phase)
- `skill/README.md` — install instructions

### docs/memory/README.md template

```markdown
# docs/memory — Per-Stage Design Memory

Write a file `phase-Na.md` here after each sub-phase with:
- Design decisions and rationale
- Alternatives considered and rejected
- Gotchas and surprises
- TVP-FUTURE items deferred
- Smoke test results
- Open reviewer warnings
```

### docs/reviews/README.md template

```markdown
# docs/reviews — Independent Sub-Agent Review Reports

Spawn a sub-agent reviewer (no shared context) at each sub-phase completion.
Save report as `phase-Na.md` here. A FAIL verdict blocks the commit.

Report format: Summary / Issues table (BLOCKER|WARNING|NOTE) / Verdict (PASS|FAIL)
```

### .claude/skills/phase-review.md and phase-commit.md

Source these skills from the `ai-projectinit` repository or copy from an existing
initialized project. They must be placed at:
- `.claude/skills/phase-review.md`
- `.claude/skills/phase-commit.md`

If the `ai-projectinit` repo is available locally, copy from its `skill/` directory.
Adapt any project-specific paths (project root, docs paths) to the new project.

### .claude/commands/summary.md

```markdown
# /summary — Print Project Reference

Reads `PROJECT.md` from the project root and prints its full contents to the console.

## Steps

1. Read `PROJECT.md` from `$CLAUDE_PROJECT_DIR/PROJECT.md`
2. Output the full content verbatim as formatted markdown

If `PROJECT.md` does not exist, print:

> PROJECT.md not found. Run `/mgm-projectinit` to generate it.
```

### .claude/commands/defaults.md

```markdown
# /defaults — Show All Claude Code Defaults

Displays the active defaults: permissions, model routing, shell rules, hooks, git conventions.

## Steps

1. Read `.claude/settings.json` — extract and display denied/allowed patterns and hooks
2. Read `.claude/rules/shell-rules.md` — display verbatim
3. Print model routing: Haiku (simple) / Sonnet (standard/default) / Opus (complex, user approval)
4. Print git conventions: branch naming, commit format, tag format, bootstrap tag rule
5. Print hook summary: guard_readonly.sh rules for curl/wget/rm/mv/cp
```

### PROJECT.md

Generate a combined setup reference and initialization record file at the project root. Include:

**Part 1 — Claude Code Setup:**
- Project identity table (name, prefix, repo, developer, skill location)
- File layout tree (`.claude/` structure, key project files)
- Model routing table
- Git workflow table + standard sub-phase workflow (9 steps)
- Shell rules summary
- Permissions summary (denied read, denied write, allowed Bash, allowed Read)
- Hooks table (event, matcher, handler)
- guard_readonly.sh guard rules table
- OpenWolf integration notes
- Skills & Commands table (`/phase-review`, `/phase-commit`, `/summary`, `/defaults`, `/mgm-projectinit`)
- CHANGELOG convention

**Part 2 — Initialization Record:**
- What mgm-projectinit does (idempotent scaffold tool)
- Bootstrap log (Steps 1–11 results)
- Extended configuration phases (A–H results)
- Files generated list
- How phase tracking works
- The mgm-aidevchat skill (locations, install)
- Re-running mgm-projectinit (conflict resolution table per file)

---

## Step 6 — Screen summary

After all files are written, print the following to the console (do not write to a file):

```
✅ Project initialized: <PROJECT_NAME> (<PREFIX>)
   Repo:      <REPO_URL>
   Developer: <DEV_NAME> (<DEV_EMAIL>)

Claude Code setup:
   CLAUDE.md         — project system prompt
   PROJECT.md        — full setup reference + init record (/summary to view)
   .claude/settings.json — permissions + hooks
   .claude/rules/shell-rules.md — atomic Bash rule
   .claude/hooks/guard_readonly.sh — curl/wget/path guard
   .claude/skills/   — phase-review, phase-commit
   .claude/commands/ — /summary, /defaults

Hooks active:
   ✅ curl: GET auto-allowed, mutations require confirmation
   ✅ wget: stdout auto-allowed, mutations and external writes guarded
   ✅ rm/mv/cp: blocked outside project root

Run /summary for full setup details.
Run /defaults to see all permissions and model routing.
```

---

## Step 8 — First commit

Stage files explicitly (never `git add -A`). Run each `git add` as a separate call:

```bash
git add CLAUDE.md
git add PROJECT.md
git add .gitignore
git add .claudeignore
git add CHANGELOG.md
git add .claude/settings.json
git add .claude/rules/shell-rules.md
git add .claude/commands/.gitkeep
git add .claude/commands/summary.md
git add .claude/commands/defaults.md
git add .claude/hooks/.gitkeep
git add .claude/resources/.gitkeep
git add .claude/skills/phase-review.md
git add .claude/skills/phase-commit.md
git add docs/memory/README.md
git add docs/reviews/README.md
git add skill/<PREFIX>-skill.md
git add skill/README.md
```

Then commit:

```bash
git commit -m "[phase-0a] project bootstrap via mgm-projectinit"
```

If commit fails:

> **Initialization aborted.** Commit failed. Check git configuration and try again.

Stop. Do not push or tag.

---

## Step 9 — Push

```bash
git push -u origin main
```

If push fails:

> **Initialization aborted.** Push to `<url>` failed.
> Check authentication (GitHub token / SSH key) and repository permissions.

Stop. Do not create the bootstrap tag.

---

## Step 10 — Tag (ONLY after successful push)

```bash
git tag v0.0-bootstrap
git push origin v0.0-bootstrap
```

Confirm:

> **Initialization complete.**
> Repository: `<url>`
> Tag: `v0.0-bootstrap`
> Run `/clear` now to start fresh before Phase 1.

---

## Step 11 — Clear context

Run `/clear` to reset the conversation context.
The project is ready. The next session starts with Phase 1.

---

## Abort conditions summary

| Step | Abort trigger | Message |
|------|--------------|---------|
| 2 | `git ls-remote` fails | Cannot reach repository — stop completely |
| 8 | `git commit` fails | Bootstrap commit failed — stop, do not push |
| 9 | `git push` fails | Bootstrap push failed — do not tag |
| F.5 | `git commit` fails | Config commit failed — report, continue to G |
| F.5 | `git push` fails | Config push failed — report, continue to G |

On any abort: report the error clearly, list what was and was not done, and stop.

---

## Extended Configuration — Phases A–G

Run after bootstrap (Steps 1–11) **or** on an existing project. All operations are idempotent.

---

### Phase A — Project Configuration (interactive)

#### A.1 — Detect available domain assets

Silently check which domain-relevant skills and MCPs are installed:

```bash
test -f /mnt/skills/user/mgm-a12/SKILL.md && echo "skill:mgm-a12=AVAILABLE" || echo "skill:mgm-a12=ABSENT"
test -f /mnt/skills/user/mgm-af/SKILL.md  && echo "skill:mgm-af=AVAILABLE"  || echo "skill:mgm-af=ABSENT"
```

Read MCP servers from `~/.claude/settings.json` → `.mcpServers` keys.

Store results as `AVAILABLE_SKILLS` and `AVAILABLE_MCPS`.

#### A.2 — Write and execute the interactive configuration script

The Bash tool runs in a non-TTY environment. `setRawMode` will fail. When it does:
use `AskUserQuestion` to collect `name`, `description`, `domains` (multi-select), and
`active_roles` (multi-select) instead. Construct the CONFIG object manually from the answers.

If a TTY is available, write `/tmp/project_config.mjs` with the script below — substituting
the actual detected values into `AVAILABLE_SKILLS` and `AVAILABLE_MCPS` — then run:

```bash
node /tmp/project_config.mjs
```

Read stdout as JSON: `{ name, description, domains, fach_context, activate_skills, activate_mcps, deactivate_mcps, active_roles }`

Store as CONFIG.

```javascript
// /tmp/project_config.mjs
import { createInterface } from "readline";
const RESET="\x1b[0m",BOLD="\x1b[1m",INVERT="\x1b[7m",DIM="\x1b[2m",CLEAR="\x1b[2J\x1b[H",GREEN="\x1b[32m";

const AVAILABLE_SKILLS = []; // populated by Claude from A.1
const AVAILABLE_MCPS   = []; // populated by Claude from A.1

const DOMAINS = [
  { key:"mgm-a12",      label:"mgm A12 Entwicklung",               skills:["mgm-a12"], mcps:["a12-docs"],
    fach:"A12 Low Code Platform (mgm). EUPL 1.2. Plasma Design System. BPMN/CIBseven. Keycloak, PostgreSQL, Kubernetes. OZG, German federal administration, Dataport, ITZBund." },
  { key:"mgm-af",       label:"mgm Atlantic Framework Agent",       skills:["mgm-af"],  mcps:[],
    fach:"mgm Atlantic Framework. AI Agent development, MCP protocol, agentic orchestration, Claude Code integration." },
  { key:"steuer",       label:"Steuer",                             skills:[], mcps:[],
    fach:"German tax law. ELSTER, BZSt, Finanzamt. UStG, EStG, KStG, AO. Tax forms, fiscal year, Datev interfaces." },
  { key:"versicherung", label:"Versicherung",                       skills:[], mcps:[],
    fach:"German insurance. VVG, GDV. Policy lifecycle, claims, premiums, actuarial, Solvency II." },
  { key:"commerce",     label:"Commerce",                           skills:[], mcps:[],
    fach:"E-commerce. Catalog, cart, checkout, payment (PCI-DSS), order management, fulfillment." },
  { key:"other",        label:"Other (manual selection)",           skills:[], mcps:[], fach:"" },
];

const ALL_ROLES = [
  { key:"architect",  label:"/architect  — System design, API contracts, ADRs, tech debt" },
  { key:"developer",  label:"/developer  — Implementation quality, patterns, code review" },
  { key:"devops",     label:"/devops     — CI/CD, deployment, infra, observability" },
  { key:"security",   label:"/security   — Threat modeling, OWASP, auth, secrets" },
  { key:"tester",     label:"/tester     — Coverage, edge cases, test strategy" },
  { key:"fach",       label:"/fach       — Domain expert (based on selected domains)" },
];

const STEPS = ["name","description","domain","assets","roles"];
let state = {
  step:"name", name:"", description:"",
  domainItems: DOMAINS.map(()=>false),   domainCursor:0,
  assetItems:[], assetChecked:[], assetCursor:0,
  roleChecked: ALL_ROLES.map(()=>true),  roleCursor:0,
};

function selectedDomains() {
  return DOMAINS.filter((_,i)=>state.domainItems[i]);
}

function buildAssets() {
  const domains = selectedDomains();
  const isOther = domains.some(d=>d.key==="other");
  const skillKeys = isOther
    ? AVAILABLE_SKILLS
    : [...new Set(domains.flatMap(d=>d.skills))].filter(s=>AVAILABLE_SKILLS.includes(s));
  const mcpKeys = isOther
    ? AVAILABLE_MCPS
    : [...new Set(domains.flatMap(d=>d.mcps))].filter(m=>AVAILABLE_MCPS.includes(m));
  return [
    ...skillKeys.map(s=>({type:"skill",key:s,label:`Skill: ${s}`})),
    ...mcpKeys.map(m=>({type:"mcp",  key:m,label:`MCP:   ${m}`})),
  ];
}

function render() {
  process.stdout.write(CLEAR);
  process.stdout.write(BOLD+"Project Initialization\n"+RESET);
  process.stdout.write("─".repeat(58)+"\n");

  if (state.step==="name") {
    process.stdout.write(BOLD+"Project name: "+RESET+state.name+"█\n");
    process.stdout.write(DIM+"ENTER to continue\n"+RESET);

  } else if (state.step==="description") {
    process.stdout.write(BOLD+"Name: "+RESET+GREEN+state.name+RESET+"\n");
    process.stdout.write(BOLD+"Short description:\n"+RESET+state.description+"█\n");
    process.stdout.write(DIM+"ENTER to continue\n"+RESET);

  } else if (state.step==="domain") {
    process.stdout.write(BOLD+"Name: "+RESET+GREEN+state.name+RESET+"  "+DIM+state.description+RESET+"\n\n");
    process.stdout.write(BOLD+"Select domains (SPACE toggle, ENTER confirm):\n"+RESET);
    DOMAINS.forEach((d,i)=>{
      const mark = state.domainItems[i]?"[x]":"[ ]";
      const line = `  ${mark}  ${d.label}\n`;
      process.stdout.write(i===state.domainCursor ? INVERT+line+RESET : line);
    });
    const sel = selectedDomains().length;
    process.stdout.write(DIM+`\n  ${sel} selected\n`+RESET);

  } else if (state.step==="assets") {
    process.stdout.write(BOLD+"Domains: "+RESET+GREEN+selectedDomains().map(d=>d.label).join(", ")+RESET+"\n\n");
    if (state.assetItems.length===0) {
      process.stdout.write(DIM+"No domain-specific skills or MCPs detected.\n"+RESET);
      process.stdout.write(DIM+"ENTER to continue\n"+RESET);
    } else {
      process.stdout.write(BOLD+"Activate domain assets (SPACE toggle, ENTER confirm):\n"+RESET);
      state.assetItems.forEach((item,i)=>{
        const mark = state.assetChecked[i]?"[x]":"[ ]";
        const line = `  ${mark}  ${item.label}\n`;
        process.stdout.write(i===state.assetCursor ? INVERT+line+RESET : line);
      });
    }

  } else if (state.step==="roles") {
    process.stdout.write(BOLD+"Activate roles for this project (SPACE toggle, ENTER confirm):\n"+RESET);
    process.stdout.write(DIM+"All roles are pre-selected. Deselect roles you don't need.\n\n"+RESET);
    ALL_ROLES.forEach((r,i)=>{
      const mark = state.roleChecked[i]?"[x]":"[ ]";
      const line = `  ${mark}  ${r.label}\n`;
      process.stdout.write(i===state.roleCursor ? INVERT+line+RESET : line);
    });
  }
}

function nextStep() {
  const idx = STEPS.indexOf(state.step);
  if (state.step==="domain") {
    if (selectedDomains().length===0) { render(); return; }
    state.assetItems   = buildAssets();
    state.assetChecked = state.assetItems.map(()=>true);
    state.assetCursor  = 0;
  }
  if (idx < STEPS.length-1) {
    state.step = STEPS[idx+1];
  } else {
    finish();
  }
}

function finish() {
  process.stdin.setRawMode(false);
  process.stdout.write(CLEAR);
  const domains   = selectedDomains();
  const fachCtx   = domains.map(d=>d.fach).filter(Boolean).join("\n");
  const actSkills = state.assetItems.filter((it,i)=>it.type==="skill"&&state.assetChecked[i]).map(it=>it.key);
  const actMcps   = state.assetItems.filter((it,i)=>it.type==="mcp"  &&state.assetChecked[i]).map(it=>it.key);
  const deactMcps = state.assetItems.filter((it,i)=>it.type==="mcp"  &&!state.assetChecked[i]).map(it=>it.key);
  const actRoles  = ALL_ROLES.filter((_,i)=>state.roleChecked[i]).map(r=>r.key);
  process.stdout.write(JSON.stringify({
    name: state.name, description: state.description,
    domains: domains.map(d=>d.key),
    fach_context: fachCtx,
    activate_skills: actSkills,
    activate_mcps:   actMcps,
    deactivate_mcps: deactMcps,
    active_roles:    actRoles,
  })+"\n");
  process.exit(0);
}

process.stdin.setRawMode(true); process.stdin.resume();
process.stdin.on("data", buf => {
  const key = buf.toString();
  if (state.step==="name"||state.step==="description") {
    if (key==="\r"||key==="\n") {
      const val=(state.step==="name"?state.name:state.description).trim();
      if (!val) { render(); return; }
      nextStep();
    } else if (key==="\x7f") {
      if (state.step==="name") state.name=state.name.slice(0,-1);
      else state.description=state.description.slice(0,-1);
    } else if (!key.startsWith("\x1b")) {
      if (state.step==="name") state.name+=key;
      else state.description+=key;
    }
  } else if (state.step==="domain") {
    if (key==="\x1b[A"&&state.domainCursor>0) state.domainCursor--;
    else if (key==="\x1b[B"&&state.domainCursor<DOMAINS.length-1) state.domainCursor++;
    else if (key===" ") state.domainItems[state.domainCursor]=!state.domainItems[state.domainCursor];
    else if (key==="\r"||key==="\n") nextStep();
    else if (key==="q"||key==="\x03") { process.stdin.setRawMode(false); process.exit(0); }
  } else if (state.step==="assets") {
    if (state.assetItems.length===0) { nextStep(); return; }
    if (key==="\x1b[A"&&state.assetCursor>0) state.assetCursor--;
    else if (key==="\x1b[B"&&state.assetCursor<state.assetItems.length-1) state.assetCursor++;
    else if (key===" ") state.assetChecked[state.assetCursor]=!state.assetChecked[state.assetCursor];
    else if (key==="\r"||key==="\n") nextStep();
    else if (key==="q"||key==="\x03") { process.stdin.setRawMode(false); process.exit(0); }
  } else if (state.step==="roles") {
    if (key==="\x1b[A"&&state.roleCursor>0) state.roleCursor--;
    else if (key==="\x1b[B"&&state.roleCursor<ALL_ROLES.length-1) state.roleCursor++;
    else if (key===" ") state.roleChecked[state.roleCursor]=!state.roleChecked[state.roleCursor];
    else if (key==="\r"||key==="\n") finish();
    else if (key==="q"||key==="\x03") { process.stdin.setRawMode(false); process.exit(0); }
  }
  render();
});
render();
```

---

### Phase B — Write Project State

Write `.claude/project-state.json` (create or merge — never overwrite existing phase data):

```json
{
  "project_name":        "<CONFIG.name>",
  "project_description": "<CONFIG.description>",
  "domains":             "<CONFIG.domains>",
  "fach_context":        "<CONFIG.fach_context>",
  "active_role":         "developer",
  "active_roles":        "<CONFIG.active_roles>",
  "active_skills":       "<CONFIG.activate_skills>",
  "active_mcps":         "<CONFIG.activate_mcps>",
  "current_phase":       1,
  "current_subphase":    "1.1",
  "phase_status":        "not_started",
  "pending_reviews":     [],
  "last_updated":        "<ISO timestamp>"
}
```

Merge rules when file already exists:
- Never overwrite `current_phase`, `current_subphase`, or `phase_status` if already set
- Add any missing top-level field from the schema above
- Replace `domain` (old string field) with `domains` (array) if the old key is present

Also add `!.claude/project-state.json` to `.gitignore`'s negation block if not already present.

---

### Phase C — Skill & MCP Activation

**C.1 — Activate selected skills in project CLAUDE.md**

For each skill in `CONFIG.activate_skills`:
- Check if `@/mnt/skills/user/<skill>/SKILL.md` is already referenced in CLAUDE.md
- If not: append the reference

For each available skill NOT in `CONFIG.activate_skills`:
- Ensure no `@`-reference exists in CLAUDE.md; remove the line if found

**C.2 — MCP project-level configuration**

Write or merge into `.claude/settings.json` under `mcpServers`:
- For each MCP in `CONFIG.activate_mcps`: ensure entry exists without `"disabled": true`
- For each MCP in `CONFIG.deactivate_mcps`: set `"disabled": true` at project level

---

### Phase D — Modular Structure

Split role definitions, workflow procedures, and context scaffolding into separate files.
CLAUDE.md becomes a compact dispatcher; all detail lives in on-demand files.

---

#### D.1 — Role dispatcher in CLAUDE.md

If a `## Role System` section already exists in CLAUDE.md: replace it.
If absent: append after the Phase Roadmap section.
Also remove any previously-inlined `## Phase & Subphase Checkpoints` and
`## CHANGELOG Entry Format` blocks from CLAUDE.md if present.

Add this exact block:

```markdown
## Role System

Active role: read `.claude/project-state.json` → `active_role`
Available roles: as configured in `.claude/project-state.json` → `active_roles`

On role activation (`/architect`, `/developer`, `/devops`, `/security`, `/tester`, `/fach`):
  Read `.claude/roles/<role>.md` and adopt that role fully.
  Confirm switch to user. Remind of active role when task is complete.

Suggest `/architect` proactively: new endpoints, schema changes, new modules,
  new dependencies, auth flow changes, new external integrations.
Suggest `/devops` proactively: Dockerfile changes, new env vars, infra changes.
Suggest `/security` proactively: auth changes, new data ingestion, access control changes.

## Workflow

On any phase or sub-phase event: read `.claude/workflow/checkpoints.md`
On CHANGELOG generation: read `.claude/workflow/changelog.md`
On new phase start: read `.claude/workflow/subproject-workflow.md`
Ground rules: read `.claude/workflow/ground-rules.md`

## Output Rules

- Be concise. No preamble, no summaries unless explicitly asked.
- Report only what changed or failed from tool results.
- No "I'll now...", "Let me...", "I've completed..." phrases.
- Suggest `/compact` when the conversation is getting long.
```

---

#### D.2 — Create .claude/roles/ files

Generate one file per role in `CONFIG.active_roles`. Skip roles not in `active_roles`.
All files are idempotent — write only if absent; if present, skip silently.

**architect.md**:
```markdown
# Role: Architect

Lens: system design, component coupling, API contracts, scalability, ADRs, technical debt.
Output style: concise decision + rationale + trade-offs. No implementation code unless asked.

## Trigger patterns — suggest /architect when:
- New or modified API endpoints or GraphQL schemas
- Database schema changes or new migrations
- New modules, services, or top-level packages added
- Changes to core abstractions (interfaces, base classes, shared DTOs)
- New external dependencies installed (npm install, pip install, cargo add)
- Authentication flows or security boundaries modified
- New external service integrations introduced

## Review checklist (phase completion)
- Did implementation match planned scope? Note any drift.
- New coupling introduced?
- API contracts stable?
- Any ADR required for decisions made?

## On activation
1. Read `.claude/context/decisions.md`
2. Note existing decisions — do not re-debate settled choices without new information
3. After making a new architectural decision: append ADR entry to `decisions.md`
4. Before phase completion review: check if any informal decisions made during the
   phase need to be formalized as ADRs
```

**developer.md**:
```markdown
# Role: Developer (Senior)

Lens: implementation quality, design patterns, DRY, performance, readability, code review.
Output style: direct implementation with inline rationale for non-obvious choices.
Default active role — active unless explicitly switched.

## Standards
- Prefer explicit over implicit
- Small, focused functions/methods
- Name things for what they do, not what they are
- Leave code cleaner than found

## On activation
1. Read `.claude/context/known-issues.md`
2. If current task touches an area covered by a known issue: apply fix proactively
3. When resolving a non-obvious bug: append entry to `known-issues.md`
```

**devops.md**:
```markdown
# Role: DevOps

Lens: CI/CD implications, deployment strategy, observability, containers, secrets management.
Output style: pipeline-aware recommendations with environment impact assessment.

## Trigger patterns — suggest /devops when:
- New Dockerfile or docker-compose changes
- New environment variables or secrets referenced
- Infrastructure configuration changes
- New deployment targets or cloud resources introduced
- Monitoring or logging gaps identified
```

**security.md**:
```markdown
# Role: Security

Lens: threat modeling, input validation, auth/authz, secrets handling, OWASP Top 10,
      dependency vulnerabilities, data exposure risks.
Output style: risk + severity (critical/high/medium/low) + concrete mitigation.
Never say "be careful" without a specific action.

## Trigger patterns — suggest /security when:
- Authentication or session management changes
- New data ingestion from external sources
- File upload or download handling added
- Access control logic changed
- New dependencies with known CVEs
- Sensitive data processed or stored

## Review checklist (phase completion)
- New attack surface introduced?
- Secrets handled correctly?
- Input validated at boundaries?
- Auth/authz changes reviewed?
```

**tester.md**:
```markdown
# Role: Tester

Lens: coverage gaps, edge cases, regression risk, boundary conditions, test strategy,
      performance characteristics.
Output style: prioritized recommendation list.
Mark each: [must-have] | [should-have] | [nice-to-have]

Activated at sub-phase completion — recommendations only, not blockers.

## Checklist per sub-phase
- Happy path covered?
- Boundary conditions tested?
- Error/exception paths covered?
- Regression risk from changes?
- Any performance-sensitive paths needing load tests?
```

**fach.md** (substitute `<domains>` and `<fach_context>` from CONFIG at write time):
```markdown
# Role: Fach (Domain Expert)

Read domain and fach_context from .claude/project-state.json before responding.

Lens: domain terminology precision, process compliance, regulatory requirements,
      business logic validity, domain-specific anti-patterns.
Output style: domain-precise language. Flag terminology misuse explicitly.
             Validate business rules against domain standards.

## On activation
1. Read .claude/project-state.json → domains, fach_context
2. Apply all domain contexts if multiple domains selected
3. Cross-domain conflicts: flag explicitly
```

---

#### D.3 — Create .claude/workflow/ files

All files are idempotent — write only if absent.

**ground-rules.md**:
```markdown
# Ground Rules

## Working principles
- Read before writing. Understand before changing.
- One concern per commit.
- If a change affects architecture, switch to /architect before proceeding.
- If a change touches security boundaries, flag it.
- Prefer reversible over irreversible actions.

## Communication
- State what you did, not what you're about to do.
- Flag blockers immediately, don't work around them silently.
- If requirements are ambiguous, ask before implementing.

## Code quality
- No commented-out code in commits.
- No TODO without a ticket reference.
- Tests are not optional for business logic.
```

**checkpoints.md**:
```markdown
# Phase & Subphase Checkpoints

## On sub-phase completion
1. Activate /tester → provide prioritized test recommendations
2. Update .claude/project-state.json: current_subphase → next value
3. Output: "Sub-phase [X.Y] complete. Test recommendations above. Proceed to [X.Z]?"

## On phase completion (all subphases done)
1. Activate /architect → scope review: drift? API contract changes? ADRs needed?
2. Activate /security → security review: new attack surface? secrets? auth changes?
3. Generate CHANGELOG.md entry (read .claude/workflow/changelog.md for format)
4. Update .claude/project-state.json: current_phase → next, phase_status → not_started
5. Output: "Phase [X] complete. Architect + Security review done. Start Phase [X+1]?"

## On new phase start
1. Output: "Starting Phase [X]. Recommend /architect review of planned scope."
2. Read .claude/workflow/subproject-workflow.md for standard procedure.
3. Wait for user confirmation before first implementation step.

## Pending reviews
Check .claude/project-state.json → pending_reviews before starting any new sub-phase.
If not empty: surface pending items to user first.

## On phase completion — additional checks
- /architect: are any decisions from this phase missing from `.claude/context/decisions.md`?
  If yes: create ADR entries before marking phase complete.
- /developer: are any resolved bugs from this phase missing from `.claude/context/known-issues.md`?
  If yes: add entries before marking phase complete.
```

**changelog.md**:
```markdown
# CHANGELOG Format

Append to CHANGELOG.md on phase completion:

---
## Phase [N] — [Phase Name] — [YYYY-MM-DD]

### Implemented
- [bullet: what was built]

### Architecture decisions
- [decision: rationale]

### Security review
- [finding addressed — or: no findings]

### Test recommendations (open)
- [from /tester reviews — or: all addressed]

### Next phase scope preview
- [2-3 bullet summary of Phase N+1 planned scope]
```

**subproject-workflow.md**:
```markdown
# Standard Sub-Project Workflow

## Phase structure
Each phase consists of named sub-phases.
Sub-phases are small enough to complete in one session.
State is tracked in .claude/project-state.json.

## Standard sequence per sub-phase
1. /architect confirms sub-phase scope is consistent with phase plan (new phases only)
2. /developer implements
3. /tester reviews at completion → recommendations logged
4. Proceed to next sub-phase

## Standard sequence per phase
1. /architect reviews full phase scope before start
2. Sub-phases executed sequentially
3. /architect + /security review at phase completion
4. CHANGELOG entry generated
5. Next phase starts fresh

## Scope discipline
If a sub-phase grows beyond original scope: stop, flag to user, optionally split.
Never silently expand scope.
```

---

#### D.4 — Create .claude/context/ scaffolding

All files are idempotent — write only if absent.
Substitute `<PROJECT_NAME>` from CONFIG.

**decisions.md**:
```markdown
# Architecture Decision Records — <PROJECT_NAME>

Maintained by /architect. Read this file on every /architect activation.
Add a new entry for every significant architectural decision made.
Update status to "superseded" if a later decision replaces an earlier one.

---

## ADR-001 — [Title]
**Date:** YYYY-MM-DD
**Status:** active | superseded by ADR-XXX
**Context:** [What situation led to this decision?]
**Decision:** [What was decided?]
**Rationale:** [Why this over alternatives?]
**Consequences:** [What does this mean going forward? What becomes easier/harder?]

---

*(Add new entries above this line)*
```

**known-issues.md**:
```markdown
# Known Issues — <PROJECT_NAME>

Maintained by /developer. Read this file on every /developer activation.
Add entries when a non-obvious bug is found and resolved.
This prevents re-discovering the same fix in future sessions.

---

## ISSUE-001 — [Short title]
**Date:** YYYY-MM-DD
**Symptom:** [What the user or developer observes]
**Root cause:** [Why it happens]
**Solution:** [Exact fix applied]
**Affected files:** [list]

---

*(Add new entries above this line)*
```

Also add `!.claude/context/` to the negation block in `.gitignore` if not already present.

---

### Phase E — CHANGELOG Initialization

If `CHANGELOG.md` does not exist, create it:

```markdown
# Changelog — <CONFIG.name>
<CONFIG.description>

Domains: <comma-separated domain labels>
Started: <ISO date>

---

*(Phase entries will be appended here automatically)*
```

If it exists: do not modify.

---

### Claude Code hook protocol (applies to all hooks)

**PreToolUse / PostToolUse decision hooks:**
- **Allow**: exit 0 with **no output** — Claude Code treats silent exit 0 as allow
- **Block**: exit 0 and output `{"decision":"block","reason":"..."}` on stdout
- **Error**: any non-zero exit code, or JSON output that doesn't match the schema

`{"decision":"allow"}` is **not** a valid output — Claude Code's schema only accepts
`"block"` as a decision value. Outputting it causes `Hook JSON output validation failed`.

`node` and other Homebrew binaries are not in Claude Code's restricted PATH
(`/usr/local/bin:/usr/bin:/bin`). Use absolute paths (e.g. `/opt/homebrew/bin/node`)
in all hook commands in `settings.json`.

---

### Phase F — Update Session Start Hook

Create or extend `.claude/hooks/session_start.sh`. Add after any existing content:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"

# Project state
STATE="$PROJECT_ROOT/.claude/project-state.json"
if [ -f "$STATE" ]; then
  echo "### Project State"
  ROLE=$(jq -r '.active_role // "developer"' "$STATE")
  PHASE=$(jq -r '.current_phase // 1' "$STATE")
  SUBPHASE=$(jq -r '.current_subphase // "1.1"' "$STATE")
  DOMAINS=$(jq -r '.domains // [] | join(", ")' "$STATE")
  PENDING=$(jq -r '.pending_reviews | length' "$STATE" 2>/dev/null || echo 0)
  echo "  Active role:    /$ROLE"
  echo "  Domains:        $DOMAINS"
  echo "  Phase:          $PHASE  /  Subphase: $SUBPHASE"
  [ "$PENDING" -gt 0 ] && echo "  Pending reviews: $PENDING"
fi

echo ""
echo "### Ground Rules"
cat "$PROJECT_ROOT/.claude/workflow/ground-rules.md" 2>/dev/null | head -20

ADR_COUNT=$(grep -c "^## ADR-" "$PROJECT_ROOT/.claude/context/decisions.md" 2>/dev/null || echo 0)
ISSUE_COUNT=$(grep -c "^## ISSUE-" "$PROJECT_ROOT/.claude/context/known-issues.md" 2>/dev/null || echo 0)
PHASE_COUNT=$(grep -c "^## Phase " "$PROJECT_ROOT/PHASES.md" 2>/dev/null || echo 0)

echo ""
echo "### Context"
echo "  ADRs: $ADR_COUNT  |  Known issues: $ISSUE_COUNT  |  Phases defined: $PHASE_COUNT"

if [ -f "$STATE" ]; then
  PHASE_NAME=$(jq -r '.phase_name // ""' "$STATE")
  SUB_NAME=$(jq -r '.subphase_name // ""' "$STATE")
  PHASE=$(jq -r '.current_phase' "$STATE")
  SUBPHASE=$(jq -r '.current_subphase' "$STATE")
  [ -n "$PHASE_NAME" ] && echo "  Current: Phase $PHASE — $PHASE_NAME / Sub $SUBPHASE — $SUB_NAME"
fi
```

Register it in `.claude/settings.json` as a `SessionStart` hook if not already present:

```json
{ "type": "command", "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/session_start.sh\"", "timeout": 5 }
```

---

### Phase F.5 — Commit configuration bootstrap

Make hook scripts executable before staging:
```bash
chmod 755 .claude/hooks/guard_readonly.sh
chmod 755 .claude/hooks/session_start.sh
```

Stage all files created or modified during Phases A–F. Run each `git add` as a separate Bash call.

Core files (always stage):
```bash
git add CLAUDE.md
git add CHANGELOG.md
git add .gitignore
git add .claude/project-state.json
git add .claude/settings.json
git add .claude/hooks/session_start.sh
```

Role module files — stage only if directory exists:
```bash
git add .claude/roles/architect.md
git add .claude/roles/developer.md
git add .claude/roles/devops.md
git add .claude/roles/security.md
git add .claude/roles/tester.md
git add .claude/roles/fach.md
```

Workflow module files — stage only if directory exists:
```bash
git add .claude/workflow/ground-rules.md
git add .claude/workflow/checkpoints.md
git add .claude/workflow/changelog.md
git add .claude/workflow/subproject-workflow.md
```

Context scaffold files — stage only if directory exists:
```bash
git add .claude/context/decisions.md
git add .claude/context/known-issues.md
git add .claude/context/project-plan.md
```

`git add` on a missing file prints a warning but exits 0 — safe to run unconditionally.

Commit:
```bash
git commit -m "[phase-0] project configuration — domains, roles, project state, session hook"
```

Push:
```bash
git push
```

If commit fails (nothing staged or other error): report the error and continue to Phase G.
If push fails: report the error and continue to Phase G. Do not tag before a successful push.

---

### Phase G — Summary

Print to console:

```
┌───────────────────────────────────────────────────────┐
│ PROJECT CONFIGURED                                    │
├───────────────────────────────────────────────────────┤
│ Name:        <CONFIG.name>                            │
│ Domains:     <comma-separated domain labels>          │
│ Description: <CONFIG.description>                     │
├───────────────────────────────────────────────────────┤
│ Active skills:  <list or "(none)">                    │
│ Active MCPs:    <list or "(none)">                    │
│ Disabled MCPs:  <list or "(none)">                    │
├───────────────────────────────────────────────────────┤
│ Roles ready:  <space-separated /role names from       │
│               CONFIG.active_roles only>               │
├───────────────────────────────────────────────────────┤
│ Phase 1 / Subphase 1.1 — not started                  │
│ Next: /planning to enter Planning Mode                │
└───────────────────────────────────────────────────────┘
```

---

### Phase H — Planning Mode

Activated automatically after Phase G, or standalone via `/planning`.
Goal: collect project constraints, generate sub-phase breakdowns, get developer confirmation.
Output: confirmed PHASES.md, `.claude/context/project-plan.md`, updated project-state.json, git commit.

---

#### H.1 — Collect project constraints

Use `AskUserQuestion` to collect context not already present in `project-state.json`.
Adapt questions — skip anything already known. Write results to `.claude/context/project-plan.md`.

Collect:
- **Tech stack**: primary language + version, framework(s), database/storage, infra/runtime
- **Team**: solo / pair / small team (N people), async or co-located
- **Timeline**: target completion date or key milestone dates (convert relative dates to ISO)
- **Non-goals**: what is explicitly out of scope for this version
- **External dependencies**: APIs, services, existing systems to integrate with
- **Known risks**: technical, timeline, or domain risks already identified

Write `.claude/context/project-plan.md`:

```markdown
# Project Plan — <project_name>

## Tech Stack
- Language: ...
- Framework: ...
- Database: ...
- Infrastructure: ...

## Team
...

## Timeline
...

## Non-Goals
...

## External Dependencies
...

## Known Risks
...
```

---

#### H.2 — Phase confirmation

If PHASES.md already exists: read existing phases and present them for review.
If absent: propose phases based on project description and domain (3–6 phases, Phase 1 = Foundation, last = Hardening).

Present all phase names and goals in one block. Ask:

> "These are the proposed phases. Accept, or tell me which to add / rename / remove."

After confirmation: proceed to H.3.

---

#### H.3 — Sub-phase generation and confirmation loop

For each confirmed phase, one round at a time:

**Generate** sub-phases applying these rules:
- 2–5 sub-phases per phase
- Each completable in one focused session
- One concern, one deliverable, clear done-condition
- Size: **S** < 2h | **M** 2–4h | **L** 4–6h — if L, note it and suggest splitting
- Order within phase: infrastructure → core logic → integration → polish/tests
- Each produces a testable artifact (endpoint, component, passing test suite)
- Acceptance criteria: 2–4 concrete checkboxes per sub-phase

**Present** in this format (plain text, no tool call):

```
━━━ Phase [N] — [Name] ━━━━━━━━━━━━━━━━━━━━━━━━━━
Goal: [goal]

[N].1  [Sub-phase name]  [S/M/L]
       Scope:    [one sentence]
       Produces: [testable artifact]
       ✓ [criterion 1]
       ✓ [criterion 2]

[N].2  ...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Ask via `AskUserQuestion`**:
- **Accept** — write this phase as-is and move to next
- **Modify** — user provides changes in free text; apply and re-present before writing
- **Regenerate** — user provides feedback; regenerate from scratch and re-present

Repeat until all phases are confirmed.

---

#### H.4 — Write confirmed PHASES.md

Overwrite PHASES.md with all confirmed phases and sub-phases.
Use the standard PHASES.md format (## Phase, ### Sub-phase, acceptance criteria checkboxes).
Update `project-state.json`:
- `phase_name`: name of Phase 1
- `subphase_name`: name of Sub-phase 1.1

---

#### H.5 — Commit plan

```bash
git add PHASES.md
git add .claude/context/project-plan.md
git add .claude/project-state.json
```

Commit message: `[phase-0] project plan confirmed — [N] phases, [M] sub-phases`

```bash
git push
```

Print:
```
✅ Planning complete.
   [N] phases / [M] sub-phases confirmed and committed.
   Start Phase 1 with: /architect (scope review) then /developer (implementation).
```
