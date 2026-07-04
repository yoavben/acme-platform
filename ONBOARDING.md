# ACME Platform Team — Agentic SDLC Workgroup Onboarding

> **Goal:** Demonstrate how to use the Tikalk Spec Kit and Team AI Directives in a real, multi-project environment.
> **Company:** ACME (placeholder — replace with your actual org name)
> **Team:** Platform (placeholder — replace with your actual team name). Naming convention: workgroup = `acme-platform` (`{company}-{team}`), directives fork = `platform-ai-directives` (`{team}-ai-directives`).
> **Current spec-kit release:** `agentic-sdlc-v0.12.4+adlc9`

---

## Overview

The workgroup is a **GitHub repository that acts as an umbrella** for three sub-projects registered as git submodules:

| # | Repo name | Type | Origin |
|---|-----------|------|--------|
| 1 | `platform-ai-directives` | AI directive library | Fork of [`tikalk/agentic-sdlc-team-ai-directives`](https://github.com/tikalk/agentic-sdlc-team-ai-directives), created by the platform team as part of this setup |
| 2 | `acme-platform-demo-a` | Python project | Existing repo, already owned/maintained by its own team |
| 3 | `acme-platform-demo-b` | Python project | Existing repo, already owned/maintained by its own team |

The workgroup repo itself (`acme-platform`) lives at the root of this directory and wires everything together via submodules.

**Why submodules?** A git submodule is a pointer from one repo to a specific commit in another repo — the umbrella repo doesn't contain the projects' code, just a reference to each one's URL and current commit. That's the right fit here because `demo-a`, `demo-b`, and `platform-ai-directives` are independent repos with their own history, owners, and release cadence; submodules let `acme-platform` track and pin specific versions of each (e.g. the directives tag a project is bootstrapped against) without merging their code or history into one repo.

---

## Starting Position

This doc is written for the platform team member doing this setup. Before Step 1, none of this exists yet except the two demo projects:

| Piece | Status before this doc |
|-------|------------------------|
| `acme-platform` workgroup repo | **Doesn't exist** — created in Step 3 |
| `platform-ai-directives` fork | **Doesn't exist** — created in Step 1 by forking `tikalk/agentic-sdlc-team-ai-directives`, trimmed in Step 2 |
| `acme-platform-demo-a` | **Already exists** — independent repo, owned/maintained by its own team, untouched by this doc except for submodule registration and spec-kit bootstrap |
| `acme-platform-demo-b` | **Already exists** — same as above |

The platform team's job is to (1) fork and trim the team AI directives, (2) stand up the workgroup repo, and (3) register all three — the new fork plus the two existing projects — as submodules. Only once that's done does the team move on to bootstrapping spec-kit inside each existing project and running `/levelup.init`.

---

## Key Repositories

| Repo | What it is | How we use it here |
|------|-----------|---------------------|
| [Tikalk spec-kit](https://github.com/tikalk/agentic-sdlc-spec-kit) | A fork of [`github/spec-kit`](https://github.com/github/spec-kit) that adds a **team-directives layer** on top of the vanilla spec/plan/tasks/implement workflow — see below for what's actually different | Installed once in Prerequisites; used throughout via `specify init`, `specify extension`, `specify self upgrade` |
| [Tikalk team-ai-directives starter](https://github.com/tikalk/agentic-sdlc-team-ai-directives) | Template repo of constitution/personas/rules/skills that AI agents follow | Forked in Step 1 as a **starting point** — `platform-ai-directives` is ACME's own copy, trimmed of irrelevant stack content in Step 2, then meant to be grown into the team's real directives via `/levelup.*`, not used as-is |
| [Tikalk 12-factor SDLC](https://github.com/tikalk/agentic-sdlc-12-factors) | Conceptual methodology doc behind agentic SDLC | Background reading only — not cloned, installed, or referenced by any command in this doc |

### What's Different in Tikal's Spec Kit vs. Vanilla

Vanilla `github/spec-kit` is built for a single developer working alone in one repo — the base spec-driven slash commands (spec → plan → tasks → implement) and nothing more. Tikal's fork turns it into a team-wide system: every project shares one evolving directives repo instead of reinventing its own, and the machinery this entire onboarding doc depends on comes from that shift:

| Addition | What it does |
|----------|-------------|
| `specify init --team-ai-directives <url-or-path>` | Points a project at a shared directives repo (like `platform-ai-directives`) instead of each project reinventing its own constitution/rules. Accepts a remote URL (optionally `@tag`) or a local path — this doc uses the local path, since `platform-ai-directives` is already checked out as a sibling submodule (Step 4) |
| Extension system (`specify extension add/remove/search/catalog`) | Installs the `team-ai-directives` extension into `.specify/extensions/`, which is what registers `/team.verify`, `/team.discover`, `/team.constitution`, and the other `adlc.team-ai-directives.*` slash commands (see `.specify/extensions/.registry` for the alias list). These run inside your AI agent session, not the `specify` CLI |
| Workflow engine (`specify workflow run/status/resume`) | Runs multi-step CLI automations defined as YAML — e.g. the bundled `speckit` workflow (`specify workflow list`) chains specify → plan → tasks → implement with review gates. Not the mechanism behind `/team.verify` or `/team.discover`, which are agent-side commands, not workflows |
| `/levelup.*` + CDR loop | `/levelup.init` → `/levelup.clarify` → `/levelup.implement` → `/levelup.validate` — the discovery-to-publish pipeline for contributing patterns back to `platform-ai-directives` (Step 9). No equivalent exists in vanilla spec-kit |
| Presets & bundles (`specify preset`, `specify bundle`) | Packaged, installable template/config sets (e.g. a `healthcare-compliance` preset) — not used in this doc, but part of the same extended toolchain |

If you ever revert to vanilla spec-kit (see Appendix), all of the above — the directives extension, `/levelup.*`, and workflow verification — go away with it.

---

## Prerequisites

Install these once on your machine before starting:

```bash
# 1. uv — fast Python package manager, required to install the Specify CLI.
#    Checks first so re-running this script is safe (skips download if already on PATH).
#    If uv is already installed but outdated, run: uv self update
command -v uv &>/dev/null || curl -LsSf https://astral.sh/uv/install.sh | sh

# 2. Tikalk Agentic SDLC Specify CLI (pin to current stable release)
#    To check the tag below is still the latest release, run:
#      gh release list --repo tikalk/agentic-sdlc-spec-kit --limit 1
#    or: gh api repos/tikalk/agentic-sdlc-spec-kit/releases/latest --jq .tag_name
uv tool install agentic-sdlc-specify-cli \
  --from git+https://github.com/tikalk/agentic-sdlc-spec-kit.git@agentic-sdlc-v0.12.4+adlc9

# Verify
specify version

# 3. GitHub CLI — used later in this doc to fork/create repos (`gh repo fork`,
#    `gh repo create`) and to check releases (`gh release list`/`gh api`).
#    Don't want to install it? Skip this step and do the equivalent manually:
#    fork/create repos via https://github.com and clone with plain `git`;
#    check releases via https://github.com/<org>/<repo>/releases instead of `gh release list`.
command -v gh &>/dev/null || brew install gh   # or see https://cli.github.com/manual/installation
gh auth login
```

---

## Step-by-Step Setup

### Step 1 — Fork the Team AI Directives

The team-ai-directives repo is your team's AI rulebook — scoped to this team, not the whole company. "Platform" here is just an example team name; substitute your actual team throughout (e.g. `payments-ai-directives`, `mobile-ai-directives`). A company with multiple teams would have one such fork per team. Fork it under your GitHub org.

```bash
# Option A: fork via GitHub UI at https://github.com/tikalk/agentic-sdlc-team-ai-directives
# Option B: fork via CLI
gh repo fork tikalk/agentic-sdlc-team-ai-directives \
  --org acme \                        # replace "acme" with your target org/company
  --clone \
  --fork-name platform-ai-directives  # --remote is unsupported here since a repo arg is given;
                                       # `--clone` already points `origin` at your fork
```

---

### Step 2 — Trim the Fork to a Clean Starting Point

The starter ships generic, multi-stack example content — Java personas, Spring Boot rules, JUnit testing patterns, Airbyte/Airflow orchestration rules, and more — that may not match your team's actual stack. Leaving it in isn't harmless: `/team.discover` and `/spec.constitution` surface directives by relevance match, and a mismatch could nudge an agent toward the wrong conventions later (e.g. Java style guidance leaking into a Python project). Keep the stack-agnostic pieces and reset the rest to an empty skeleton, so `/levelup.init` (Step 9) repopulates it later from your own codebases instead of inheriting content you'll never use.

**Keep as-is:** `context_modules/constitution.md` (generic but solid, language-agnostic principles), `AGENTS.md`, `README.md` (schema/structure docs).

**Empty out:** `context_modules/personas/`, `context_modules/rules/`, `context_modules/examples/`, `skills/` — and reset `.skills.json` and `CDR.md` to their bare schema, since both currently reference the files you're deleting.

```bash
cd platform-ai-directives

rm -rf context_modules/personas/* context_modules/rules/* context_modules/examples/* skills/*

cat > .skills.json <<'EOF'
{
  "version": "1.0.0",
  "source": "team-ai-directives",
  "description": "Team skills manifest for the Skills Package Manager. Defines required, recommended, internal, and blocked skills with policy settings.",
  "skills": {
    "required": {},
    "recommended": {}
  }
}
EOF

cat > CDR.md <<'EOF'
# Context Directive Records

Context Directive Records (CDRs) track approved contributions to team-ai-directives from various projects. CDRs document patterns, practices, and knowledge extracted from real-world implementations.

> **Memory Engineering**: This index tracks directive freshness. Run `/levelup.validate` to update verification timestamps.

## CDR Index

| ID | Target Module | Type | Status | Created | Verified | Age | Descriptor |
|----|---------------|------|--------|---------|----------|-----|------------|
EOF

git add -A && git commit -m "chore: trim starter to a clean baseline, keep constitution and schema"
```

This is a one-time manual exception to the "don't hand-edit yet" rule below — it applies to `personas/`, `rules/`, `examples/`, and `skills/`, not to `constitution.md`'s actual principles. Those stay as the inherited starting point until amended later through the CDR loop (Step 9), not by hand-editing now.

Tag the cleaned baseline so downstream projects have something to pin to. Note: this repo already carries upstream version tags (`v1.0.0`–`v1.8.x`) from the template it was forked from, so reuse of `v1.0.0` will collide — prefix with your team name instead (matching the `{team}-ai-directives` naming convention), not the company name, since this fork is team-scoped:

```bash
git tag platform-v1.0.0
git push origin platform-v1.0.0
```

---

### Step 3 — Create the Team's Workgroup Repository

The workgroup repo is scoped to this one team, not the whole company — it's the root umbrella for the Platform team's projects specifically. A company with multiple teams would have one workgroup repo per team (e.g. `acme-platform`, `acme-payments`, `acme-mobile`), each wired to its own directives fork from Step 1, not a single shared company-wide umbrella. It doesn't contain source code — only submodule pointers and shared documentation.

```bash
# "acme" stands for your target org/company throughout this doc. Replace it with:
#   - an org name, if you have repo-creation rights there (e.g. a real "acme" org
#     with a "platform" team would run this exactly as written), or
#   - your own GitHub username, if you're following along solo (e.g. `yoavben/acme-platform`,
#     or just `acme-platform` with no owner prefix — that defaults to your account)

# Create empty repo on GitHub
gh repo create acme/acme-platform --public --description "ACME multi-project agentic SDLC workgroup"

# Initialize locally (this directory)
cd /path/to/acme-platform    # or your workgroup directory
git init
git remote add origin git@github.com:acme/acme-platform.git

# Minimal root files
touch README.md .gitmodules
git add . && git commit -m "chore: initialize workgroup"
git push -u origin main
```

---

### Step 4 — Register the Existing Projects and Directives Fork as Submodules

Demo Project A and Demo Project B already exist as independent repos — they're not created here, just wired into the workgroup. The directives fork from Step 1 gets registered the same way.

```bash
cd acme-platform

# Existing projects, owned by their own teams — just add as submodules
git submodule add git@github.com:acme/acme-platform-demo-a.git demo-a
git submodule add git@github.com:acme/acme-platform-demo-b.git demo-b

# Directives fork created in Step 1
git submodule add git@github.com:acme/platform-ai-directives.git platform-ai-directives

git commit -m "chore: register demo-a, demo-b, and platform-ai-directives as submodules"
git push
```

---

### Step 5 — Verify the Full Workgroup Structure

After Steps 1–4, the workgroup directory tree looks like this — note the two demo projects only have their pre-existing content so far; spec-kit hasn't been bootstrapped into them yet:

```
acme-platform/               ← root umbrella repo (acme/acme-platform)
├── .gitmodules               ← submodule registry
├── README.md                 ← workgroup overview
├── ONBOARDING.md              ← this file
├── platform-ai-directives/   ← acme/platform-ai-directives (submodule, trimmed in Step 2)
│   ├── context_modules/
│   │   ├── constitution.md
│   │   ├── personas/
│   │   └── rules/
│   └── skills/
├── demo-a/                    ← acme/acme-platform-demo-a (submodule, pre-existing)
│   ├── src/
│   └── pyproject.toml
└── demo-b/                    ← acme/acme-platform-demo-b (submodule, pre-existing)
    ├── src/
    └── pyproject.toml
```

Clone everything fresh with:

```bash
git clone --recurse-submodules git@github.com:acme/acme-platform.git
```

---

## Modes of Operation

Before bootstrapping anything, it's worth naming the choice Step 6 onward makes implicitly: there are two ways to scope spec-kit and the AI agent across a multi-project workgroup like this one.

| Mode | How it works | Status here |
|-------|--------------|--------------|
| **Split (per-project)** | `specify init`, specs, and the AI agent are each scoped to one project directory (`demo-a`, `demo-b`) at a time. Cross-repo features are split into paired per-project specs. `/levelup.init` runs sequentially, one project at a time, with the directives ref refreshed between runs (see the Optional Workflow note after Step 12). | **What Steps 6–12 implement.** |
| **Root-aware (unified)** | Spec-kit and the AI agent are bootstrapped at the workgroup root, with specs also living at the root, so the agent is aware of all submodules at once and can work a genuine cross-repo spec directly. | Not implemented yet — flagged as a follow-up to evaluate (see the note after Step 12). |

This doc follows the **split mode** from here on. Treat the root-aware mode as a separate thing to try and compare, not an assumption baked into Steps 6–12.

---

### Step 6 — Initialize Spec Kit and Team AI Directives in Each Project

Only now, with the workgroup wired up, do we bootstrap spec-kit inside each existing project and point it at the directives fork. Since `platform-ai-directives` is already checked out as a sibling submodule at the workgroup root (Step 4), we point `--team-ai-directives` at it via a local relative path rather than the remote URL — no network fetch, and it always reflects whatever commit the submodule is currently pinned to:

```bash
cd demo-a
specify init . \
  --integration claude \
  --team-ai-directives ../platform-ai-directives
git add -A && git commit -m "chore: bootstrap spec-kit + team-ai-directives"
git push
cd ..
```

Repeat for every project repository in the workgroup (`demo-a`, `demo-b`, ...) — not `platform-ai-directives` itself.

This adds a `.specify/` directory to each project (see Step 13 for its layout).

> The local path means `demo-a` and `demo-b` share one directives checkout — they always see whatever commit `platform-ai-directives` is pinned to at the workgroup root, and can't independently pin to different tags the way a remote `@tag` URL would allow. See the trade-off note in the Optional Workflow section after Step 12.

---

### Step 7 — Establish the Project Constitution (per project)

`specify init` (Step 6) only drops in the unfilled `constitution-template.md` — it doesn't write real principles into `.specify/memory/constitution.md`. Do this next, before verifying or doing any spec/plan work: `/spec.plan` checks specs against the constitution via a "Constitution Check" gate, and Step 8's health check specifically inspects this file — running it against the raw template just produces a warning you'd have to come back and clear.

Inside the project directory, in your AI agent session:

```
/spec.constitution
```

This first runs the `team-ai-directives` `before_constitution` hook, which loads `platform-ai-directives/context_modules/constitution.md` into context so the project constitution inherits the team's principles instead of starting from scratch. Answer the prompts (or pass principles as arguments) to fill in the project-specific sections, then commit the result:

```bash
git add .specify/memory/constitution.md
git commit -m "docs: establish project constitution"
git push
```

Repeat for each project (`demo-a`, `demo-b`).

This is distinct from `/levelup.init`'s "Constitution CDR" (Step 9): that one scans whatever code *already exists* in the project for cross-cutting patterns and proposes *amendments* back to the shared `platform-ai-directives` constitution. This step is what gives the project a real constitution to begin with; Step 9 refines it later with project-specific evidence.

---

### Step 8 — Verify Spec Kit Integration in Each Project

Run the built-in health check in each project. This is a slash command provided by the `team-ai-directives` extension, not a `specify` CLI command — invoke it inside your AI agent session (Claude Code), from within each project directory:

```
/team.verify
```

(fully-qualified form: `/adlc.team-ai-directives.verify` — see the registered alias in `.specify/extensions/.registry`)

Run it once in `demo-a` and once in `demo-b`. Both should report all checks green (extension installed, skills loaded, constitution present and inheriting team principles — this is why Step 7 comes first).

---

### Step 9 — LevelUp Init: Brownfield Codebase Analysis (per project, once)

Run this **once per project**, right after Step 8's verification passes — not as part of ongoing daily work, and not gated on it. `/levelup.init` is a **brownfield** scanner: it mines whatever code *already exists* in the project for reusable patterns and proposes **Context Directive Records (CDRs)**, the mechanism for contributing knowledge back to the shared team-ai-directives. Since `demo-a`/`demo-b` are pre-existing repos (Step 4), not created by this doc, there's already real code to mine here — it doesn't need Step 11's daily workflow to produce new features first.

### What `/levelup.init` Does

It runs a **three-phase multi-agent pipeline** inside your AI session:

| Phase | Agent | Work done |
|-------|-------|-----------|
| 1 — Discovery | Discovery Agent | Scans each sub-system, finds raw patterns |
| 2 — Pattern Analysis | Pattern Agent | Classifies patterns, scores reusability (0–1), checks for duplicates vs. existing directives |
| 3 — Synthesis | Synthesis Agent | Cross-sub-system analysis, detects inconsistencies, generates CDRs |

It also generates a **Constitution CDR** if new principles emerge from the cross-cutting patterns.

### How to Run It (per project)

Open your AI coding agent (e.g. Claude Code) inside the project directory and invoke the slash command:

```
# Scan entire codebase (default)
/levelup.init

# Focus on a specific concern
/levelup.init "Python FastAPI patterns"

# Focus only on rules
/levelup.init --focus rules

# Resume an interrupted run
/levelup.init --resume
```

The command is interactive — it will propose detected sub-systems and ask you to confirm before proceeding.

### Output

All output is written to `.specify/drafts/cdr.md`. The CDRs are **not** pushed to team-ai-directives automatically — they stay local until reviewed.

### After `/levelup.init`: The Review → Publish Loop

```
/levelup.init        ← scan & discover CDRs
     ↓
/levelup.clarify     ← resolve ambiguities and inconsistencies
     ↓
Review cdr.md        ← mark CDRs "Accepted" after human review
     ↓
/levelup.implement   ← compiles accepted CDRs into a PR to platform-ai-directives
```

### Signal Gate (quality filter)

Before a CDR can be published via `/levelup.implement`, it must pass a strict quality check:

| Check | Requirement |
|-------|------------|
| Team-wide | Pattern applies across multiple projects, not just this one |
| High Value | Saves >30 min per future use |
| Unique | Not already covered by existing directives |
| Evidence | Has concrete commits and/or file paths as proof |

CDRs that fail the gate stay in local drafts for refinement.

### Published Directive Format

Every directive published to team-ai-directives includes YAML frontmatter for freshness tracking:

```yaml
---
id: rule-python-error-handling
cdr_ref: CDR-2026-001
created: 2026-04-15
modified: 2026-05-18
verified: 2026-05-18
age_days: 33
evidence:
  - commit: abc123
    file: src/error_handler.py
---
```

Directives older than 30 days without re-verification show a staleness warning to AI agents.

### Validate Existing Directives

Use this periodically to check for rule conflicts and stale directives:

```
/levelup.validate
```

---

### Step 10 — Re-sync the Project Constitution (per project)

If Step 9 generated a Constitution CDR and it's since been accepted and published (merged into `platform-ai-directives`, tagged, and the submodule pointer bumped — see Step 12), the project's own `.specify/memory/constitution.md` doesn't pick that up automatically; it was only inherited once, back in Step 7.

Re-run both commands to catch it up:

```
/spec.constitution
/team.verify
```

`/spec.constitution` re-triggers the `before_constitution` hook, merging the team's newly-amended principles into the project constitution; `/team.verify` re-confirms Check 6 (Constitution Alignment) now reports `[OK]` against the updated version. Repeat for each project (`demo-a`, `demo-b`).

Skip this step if Step 9 didn't produce a constitution amendment, or if the resulting CDR wasn't accepted and published.

---

### Step 11 — Daily Workflow (per project)

Inside any project directory, the typical spec-driven session looks like — all of these are slash commands run inside your AI agent session, not `specify` CLI invocations:

```
# 1. Discover relevant context before starting a feature
/team.discover

# 2. Create a spec for your feature
# (launch your AI agent — e.g. Claude Code — and use /spec.new)

# 3. Generate implementation plan from spec
# /spec.plan  (inside your AI agent session)

# 4. Execute tasks
# /spec.execute

# 5. After a significant session, capture learnings back to directives
# /spec.levelup  — creates a knowledge packet + suggests directives updates
```

---

### Step 12 — Updating the Team AI Directives

When a project team discovers a useful pattern:
1. Use `/spec.levelup` inside the AI session to generate a knowledge packet.
2. PR the change into `platform-ai-directives`.
3. Tag a new release (e.g. `platform-v1.1.0`).
4. Bump the submodule pointer in `acme-platform/platform-ai-directives` to that tag, then `git submodule update --remote platform-ai-directives`.

Because `--team-ai-directives` points at the local path rather than a remote `@tag`, there's no separate per-project `.specify/` config to update — every project sees the new version as soon as the submodule pointer is bumped and updated locally.

---

## Spec Kit CLI Reference (quick cheat-sheet)

| Command | What it does |
|---------|-------------|
| `specify version` | Print installed CLI version |
| `specify self check` | Check if a newer release is available |
| `specify self upgrade` | Upgrade CLI to latest stable release |
| `specify init <dir> --team-ai-directives <url-or-path>` | Bootstrap a project with spec-kit + directives |

These next two are **not** `specify` CLI commands — they're slash commands from the `team-ai-directives` extension, run inside your AI agent session:

| Command | What it does |
|---------|-------------|
| `/team.verify` | Health check — extension, skills, CDR, constitution |
| `/team.discover` | Auto-discover relevant context for a feature |

---

## Upgrading the Spec Kit CLI

```bash
# Check for updates (read-only)
specify self check

# Preview the upgrade
specify self upgrade --dry-run

# Upgrade to latest
specify self upgrade

# Pin a specific version
specify self upgrade --tag agentic-sdlc-v0.12.4+adlc9
```

---

## Optional Workflow: Sequential Multi-Project LevelUp + Split Specs

> **Status: optional, unvalidated.** Try it on a real cross-project feature and confirm it holds up before treating it as standard practice.

For a workgroup like this one — multiple projects sharing one `platform-ai-directives` — where a feature spans more than one project:

- **Specs stay per-project.** `.specify/specs/` lives inside each project (Step 13), not at the workgroup root. A feature touching both `demo-a` and `demo-b` gets split into two specs, one per project, instead of one spec spanning both repos.
- **`/levelup.init` runs one project at a time, sequentially — not in parallel.** Recommended order:
  1. Run `/levelup.init` → `/levelup.clarify` → review → `/levelup.implement` in `demo-a`. This merges a PR into `platform-ai-directives` and cuts a new tag (e.g. `platform-v1.1.0`).
  2. Before starting in `demo-b`, bump the shared `platform-ai-directives` submodule pointer at the workgroup root to that tag and run `git submodule update --remote platform-ai-directives` (Step 12, item 4). Because `demo-a` and `demo-b` both reference the directives via the same local path, this one bump is enough — there's no separate per-project ref to update — and `demo-b`'s Pattern Agent phase then sees what `demo-a` already contributed and doesn't re-propose it.
  3. Run `/levelup.init` in `demo-b`. Its output should then only cover what's new or `demo-b`-specific.

Each `specify init` and each `/levelup.init` run stays scoped to one project directory, matching Step 6 and Step 9 as written — Claude/the AI agent is initialized per project, not at the workgroup root.

**Trade-off of the shared local checkout:** because both projects point at the same `platform-ai-directives` directory, they can't independently pin to different directive versions — bumping the submodule pointer for `demo-b`'s sake also moves `demo-a` onto the new tag, even if `demo-a` wasn't ready to move. That's fine for this sequential workflow (both projects want the latest), but if a project ever needs to stay pinned to an older directives version while another moves ahead, that requires going back to a remote `@tag` URL for that project's `--team-ai-directives` instead of the local path.

### Follow-up to evaluate: root-aware multi-project setup

A different mode, not yet adopted here: bootstrap spec-kit and the AI agent **at the workgroup root** instead of per-project, with specs also living at the root, so the agent is aware of all submodules (`demo-a`, `demo-b`, `platform-ai-directives`) at once instead of being scoped to a single project directory. This would presumably support a genuine cross-repo spec instead of split per-project specs. Flagged here as a follow-up to investigate, not a current recommendation.

---

## Step 13 — Minimal Demo Project Structure (Already Created)

The demo projects at the workgroup root are scaffolded as minimal Python packages:

```
demo-a/
├── pyproject.toml
├── src/demo_a/
│   ├── __init__.py
│   └── main.py
└── tests/
    └── test_main.py
demo-b/
├── pyproject.toml
├── src/demo_b/
│   ├── __init__.py
│   └── main.py
└── tests/
    └── test_main.py
```

After running `specify init` in each project directory, each will also have:

```
demo-a/
└── .specify/
    ├── commands/          ← AI agent slash commands
    ├── memory/            ← project constitution lives here
    ├── specs/             ← feature specs created during /spec.specify
    ├── templates/
    ├── drafts/
    │   └── cdr.md         ← generated by /levelup.init
    └── extensions/
        └── team-ai-directives/   ← installed from platform-ai-directives
```

---

## Appendix: Reverting to Vanilla Spec Kit

If you installed the Tikal version (`agentic-sdlc-specify-cli`) during this workgroup and want to return to the upstream `github/spec-kit`:

**Before switching**, note your vanilla version (or check `uv tool list` — it stays registered even after the Tikal version takes over the `specify` binary):

```bash
uv tool list
# specify-cli v0.11.1   ← this is your vanilla version
```

**Revert:**

```bash
uv tool install specify-cli \
  --from git+https://github.com/github/spec-kit.git@v<your-version> \
  --reinstall
```

Replace `<your-version>` with the version noted above (e.g. `v0.11.1`).

**Verify:**

```bash
specify version   # should show the vanilla GitHub Spec Kit banner and version
```

> Both tools share the `~/.local/bin/specify` binary. `--reinstall` forces the binary to be re-linked to the vanilla installation. No uninstall is required.

---

## Sources

- [tikalk/agentic-sdlc-spec-kit](https://github.com/tikalk/agentic-sdlc-spec-kit)
- [tikalk/agentic-sdlc-team-ai-directives](https://github.com/tikalk/agentic-sdlc-team-ai-directives)
- [tikalk/agentic-sdlc-12-factors](https://github.com/tikalk/agentic-sdlc-12-factors)
- [Tikal Knowledge GitHub org](https://github.com/tikalk)
