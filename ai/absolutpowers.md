# AbsolutPowers Skills

Plugin for Claude Code and Codex supporting the entire AI-assisted development lifecycle — from feature design through planning and implementation to code review.

Open-source repo: [github.com/AbsolutSystems/absolutpowers](https://github.com/AbsolutSystems/absolutpowers)

## Installation

### Claude Code

#### 1. Add the marketplace

```bash
/plugin marketplace add AbsolutSystems/absolutpowers
```

#### 2. Install the plugin

```bash
/plugin install absolutpowers@absolutpowers-skills
```

#### 3. Verify

Restart Claude Code and type `/` — autocomplete should show the plugin's commands (`/feature-discuss`, `/generate-tasks`, `/implement`, `/review`, `/debug`, `/update-ai-context`, `/tech-lead-advisor`), each labelled `(absolutpowers)` on the right. The label helps distinguish this plugin's commands from others.

#### Updating

```bash
/plugin install absolutpowers@absolutpowers-skills
```

### Codex

The repo exposes the plugin bundle at `./plugins/absolutpowers` and a local marketplace at `.agents/plugins/marketplace.json`. The repo marketplace is **repo-scoped** — visible only when you launch Codex inside the cloned repo. To make the plugin available **globally** (in any directory), add it to a **personal marketplace** at `~/.agents/plugins/marketplace.json`.

#### Requirements

- Codex CLI version with plugin support (0.125+ recommended)
- Git

#### 1. Clone the repo to a user-level location

```bash
mkdir -p ~/.agents/plugins
git clone https://github.com/AbsolutSystems/absolutpowers ~/.agents/plugins/absolutpowers
```

We clone directly into `~/.agents/plugins/absolutpowers/` — inside the personal marketplace root, so `source.path` in `marketplace.json` can be a relative path with `./` (per OpenAI's documentation).

#### 2. Create the personal marketplace

File `~/.agents/plugins/marketplace.json` with the following content:

```json
{
  "name": "absolutpowers-personal",
  "interface": {
    "displayName": "AbsolutPowers (personal)"
  },
  "plugins": [
    {
      "name": "absolutpowers",
      "source": {
        "source": "local",
        "path": "./absolutpowers/plugins/absolutpowers"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Productivity"
    }
  ]
}
```

#### 3. Verify the structure

```bash
ls ~/.agents/plugins/absolutpowers/plugins/absolutpowers/.codex-plugin/plugin.json
```

The file must exist.

#### 4. Launch Codex and install the plugin

In any directory (preferably a neutral one, e.g. `cd ~`):

```bash
codex
```

In the session:

```text
/plugins
```

In the Plugin Directory you'll see the **"AbsolutPowers (personal)"** marketplace — pick `absolutpowers` → Install. Restart Codex after the install completes.

#### 5. Invocation

In Codex, use the handle `$absolutpowers` followed by the skill name:

```text
$absolutpowers feature-discuss design a push notification system
$absolutpowers generate-tasks ./absolutpowers/feature/planning-push-notifications.md
$absolutpowers implement ./absolutpowers/feature/tasks-push-notifications.md
$absolutpowers review
$absolutpowers debug auth tests fail only in CI
$absolutpowers update-ai-context
```

The plugin works in **any directory** where you launch Codex.

#### Updating

The plugin is a git clone, so:

```bash
cd ~/.agents/plugins/absolutpowers
git pull
```

After `git pull`, restart Codex so it picks up the fresh skills. If Codex caches old versions, do Uninstall + Install again through `/plugins`.

#### Alternative: repo-scoped install

If you'd rather have the plugin available in a single repo only (e.g. for testing), skip steps 1–3 and launch Codex directly inside the cloned `absolutpowers` repo — the `.agents/plugins/marketplace.json` marketplace in the repo is picked up automatically. The plugin will then be available **only when you launch Codex in that repo**.

## Development Pipeline

Each skill produces output consumed by the next:

```
feature-discuss → generate-tasks → implement → review
     (WHAT?)         (HOW?)     (BUILD+VERIFY)  (AUDIT)
```

### `/feature-discuss` — feature discussion

Interactive Product Owner / Product Architect session. Claude doesn't write code — it drives the requirements conversation.

```bash
/feature-discuss I want a push notification system for users
```

- Asks one question per turn with selectable options (a/b/c/d), analyzes the codebase, proposes 2–3 alternatives with tradeoffs
- Creates an ADR for significant architectural decisions (not for file naming choices)
- Smart routing: micro-changes skip the planning doc; standard features → planning doc + suggestion to run `/generate-tasks`

**Output:** `./absolutpowers/feature/planning-{slug}.md`, optionally ADRs in `./docs/adr/`

### `/generate-tasks` — implementation plan

Creates a sequential plan from a planning document or a review report.

```bash
/generate-tasks @absolutpowers/feature/planning-push-notifications.md
/generate-tasks @absolutpowers/reviews/2026-04-21-feature-auth.md
```

- Reads the input doc, `./absolutpowers/patterns.md`, `./absolutpowers/rules.md`, ADRs
- Produces sequential tasks with exact file paths, method signatures, test cases
- Adds a final verification task with concrete project commands (build / typecheck / lint / formatter)

**Output:**
- From a planning doc: `./absolutpowers/feature/tasks-{slug}.md`
- From a review report: `./absolutpowers/feature/tasks-fix-{branch-slug}.md`

### `/implement` — task execution

Executes tasks sequentially with a TDD approach (where it makes sense).

```bash
/implement @absolutpowers/feature/tasks-push-notifications.md
```

- Picks the first pending task, implements it, and updates its status to `completed` in-place
- If it spots a better approach than the task — stops and asks before implementing
- Runs the final verification task before reporting completion (won't report completion if build/test fails)
- Updates `CLAUDE.md` / `AGENTS.md` for new modules, creates an ADR for significant deviations from the plan
- Reads `./absolutpowers/project-memory.md` on startup and may create memory candidates

**Output:** code + tests, updated tasks file

### `/review` — full code review

Four-phase review of current branch changes — always runs all phases, no partial reviews.

```bash
/review
/review develop    # custom base branch
```

| Phase | Focus |
|-------|-------|
| 1. Semantic Review | Behavior changes, blast radius, architectural decisions |
| 2. Edge Case Hunt | null, empty string, off-by-one, race conditions |
| 3. Rules Check | Compliance with `./absolutpowers/rules.md` (binary) |
| 4. Garbage Collection | Dead imports, debug logs, commented-out code |

Pulls the full picture of changes: diff vs base + staged + unstaged + untracked (new files read in full). Checks for evidence of final verification on executable code changes. Reads `./absolutpowers/project-memory.md` and may create memory candidates.

**Output:** `./absolutpowers/reviews/YYYY-MM-DD-{branch-slug}.md` (always, audit trail)

**Smart follow-up:** 0–2 problems — fix manually. 3+ problems — suggests `/generate-tasks` on the review report.

## Supporting Skills

### `/debug` — systematic debugging

Root cause investigation before any fix is attempted. **Iron Law:** no fixes without root cause investigation first.

```bash
/debug auth module tests fail in CI but pass locally
```

Four phases: Root Cause → Pattern Analysis → Hypothesis → Implementation. After 3+ failed fixes, it forces an architecture discussion (the problem is the foundation, not the hypothesis).

Includes supporting techniques: `root-cause-tracing.md` (trace the bug backward), `defense-in-depth.md` (validation across multiple layers), `condition-based-waiting.md` (eliminate flaky tests). Reads `./absolutpowers/project-memory.md` and may create memory candidates.

### `/update-ai-context` — AI documentation

Bootstraps or refreshes AI documentation for the project.

```bash
/update-ai-context
```

- **No CLAUDE.md** → Bootstrap: creates full documentation from scratch
- **CLAUDE.md exists** → Update: audits for drift, discovers new patterns

Manages: `CLAUDE.md`, mirrored `AGENTS.md`, `./absolutpowers/patterns.md`, `./absolutpowers/rules.md`

#### When to run it

Worth running regularly on existing codebases — e.g. every 1–2 weeks or after major changes (new module, architecture refactor, stack change).

**What it does on update:**
- **Drift detection** — finds discrepancies between `CLAUDE.md` and the current code: removed files, renamed components, new dependencies, stale paths
- **Pattern discovery** — scans the codebase for new patterns worth documenting in `patterns.md` (e.g. a new service type, a new error handling style)
- **Rules audit** — checks whether `rules.md` still reflects actual conventions in the code

**Why it matters:** a `CLAUDE.md` that lags behind the code is worse than no `CLAUDE.md` — the agent makes decisions based on stale information. Regular updates prevent that drift.

After updating the hierarchical `CLAUDE.md` files, the skill refreshes the sibling `AGENTS.md` mirrors.

## Strategic Support

### `/tech-lead-advisor`

Strategic guidance as an experienced software architect and tech lead — critical but constructive, opinionated but evidence-based.

```bash
/tech-lead-advisor should we migrate from REST to GraphQL for the mobile API?
```

Reads `CLAUDE.md` / `AGENTS.md`, ADRs, and `./absolutpowers/` before issuing a recommendation. Asks for missing context (scale, timeline, reversibility) before committing to a strong opinion. The response follows this structure: Quick Assessment → What Works → Concerns & Risks → Alternatives (with tradeoffs) → Recommendation.

Use it when you need a second opinion on an important technical choice — before a decision whose reversal would be costly.

## Shared AI Context

The project context workflow is shared across Claude Code and Codex:

- `CLAUDE.md` is the editable source of truth
- `AGENTS.md` is the generated mirror for Codex with the same directory scope
- `./absolutpowers/patterns.md` and `./absolutpowers/rules.md` are shared by both agents
- `./absolutpowers/project-memory.md` is optional operational memory for future coding work
- `./absolutpowers/memory-candidates/` is the approval queue for proposed durable memory entries

Helper script available in both plugin targets:

```bash
python3 scripts/sync_claude_to_agents.py /path/to/project
```

## File Conventions

All artifacts live under `./absolutpowers/` in the project root:

```
project/
├── absolutpowers/
│   ├── feature/
│   │   ├── planning-{slug}.md
│   │   └── tasks-{slug}.md
│   ├── memory-candidates/
│   │   └── memory-candidates-YYYY-MM-DD-{slug}.md
│   ├── project-memory.md
│   ├── reviews/
│   │   └── YYYY-MM-DD-{branch-slug}.md
│   ├── patterns.md
│   └── rules.md
├── docs/adr/
│   └── YYYY-MM-DD-{slug}.md
├── CLAUDE.md
└── AGENTS.md
```

## Project Memory

`project-memory.md` holds durable implementation knowledge meant to help future agents and developers avoid repeating the same mistakes.

**Use it for:**
- recurring traps
- non-obvious workarounds
- failure patterns with clear warning signs
- lessons that are likely to matter again in future tasks

**Don't use it for:**
- one-off ticket context
- temporary debugging notes
- branch-specific status
- facts that belong in `patterns.md`, `rules.md`, or ADRs instead

**Workflow:**
- `implement`, `debug`, and `review` read `./absolutpowers/project-memory.md` on startup if it exists
- if a session uncovers a durable lesson, the agent may create `./absolutpowers/memory-candidates/memory-candidates-YYYY-MM-DD-{slug}.md`
- the agent should ask the developer whether to promote the candidate into `project-memory.md`
- promotion requires explicit developer approval
- on promotion, update an existing matching memory entry instead of duplicating it
- after successful promotion, the candidate file is deleted

**Recommended `project-memory.md` structure:**

```markdown
# Project Memory

## src/auth

### Token refresh race in session bootstrap
- Problem: concurrent refresh paths invalidate each other
- Symptoms: flaky 401 on first page load, duplicate refresh requests
- Root cause: bootstrap and interceptor both refresh from stale state
- Resolution: gate refresh through a shared in-flight promise
- Warning signs:
  - intermittent auth failures only on cold start
  - duplicate refresh logs within one request cycle
- Affected paths:
  - `src/auth/bootstrap.ts`
  - `src/auth/refresh-token.ts`
```

**Recommended candidate structure:**

A candidate file (before promotion to `project-memory.md`) carries additional audit metadata — which skill produced it, in what context, and why it's worth promoting. This lets the developer make an informed decision: approve, reject, or modify.

```markdown
# Memory Candidate: Token refresh race in session bootstrap

## Status
Candidate — YYYY-MM-DD

## Source
- Skill: implement | debug | review
- Context: task / bug / branch being worked on

## Module
`src/auth`

## Problem / Symptoms / Root Cause / Resolution / Warning Signs / Affected Paths
[same as in project-memory.md entry]

## Why This May Matter Again
[justification — why this lesson is likely to recur]
```

After promotion to `project-memory.md`, only the concise operational part remains (Problem / Symptoms / Root Cause / Resolution / Warning Signs / Affected Paths). The metadata (Status, Source, Why This May Matter Again) is dropped — the candidate file is deleted.

## Typical Workflows

### New Feature

```bash
/feature-discuss "push notification system"
/generate-tasks @absolutpowers/feature/planning-push-notifications.md
/implement @absolutpowers/feature/tasks-push-notifications.md
/review
```

The generated tasks file should end with a final verification step, for example:
- backend compilation/build
- frontend production build
- typecheck
- formatter check (e.g. `spotlessCheck`)

`generate-tasks` emits this as a normal last task in the same task format, not as a loose reminder.

### Bug Fix

```bash
/debug "endpoint /api/users returns 500 on empty query param"
```

### Project Setup

```bash
/update-ai-context
```

## Repo Structure

```
absolutpowers/
├── .agents/
│   └── plugins/
│       └── marketplace.json    # Repo-local Codex marketplace
├── .claude-plugin/
│   ├── marketplace.json        # Marketplace manifest
│   └── plugin.json             # Plugin manifest
├── plugins/
│   └── absolutpowers/
│       ├── commands/
│       │   └── tech-lead-advisor.md
│       ├── .codex-plugin/
│       │   └── plugin.json     # Codex plugin manifest
│       └── skills/
│           └── ...             # Codex skill set
├── skills/
│   ├── debug/
│   ├── feature-discuss/
│   ├── generate-tasks/
│   ├── implement/
│   ├── review/
│   └── update-ai-context/
├── package.json
└── README.md
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or IDE extension, or
- Codex with local marketplace support

## License

MIT — Absolut Systems