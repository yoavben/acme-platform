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

---

## Starting Position

This doc is written for the platform team member doing this setup. Before Step 1, none of this exists yet except the two demo projects:

| Piece | Status before this doc |
|-------|------------------------|
| `acme-platform` workgroup repo | **Doesn't exist** — created in Step 2 |
| `platform-ai-directives` fork | **Doesn't exist** — created in Step 1 by forking `tikalk/agentic-sdlc-team-ai-directives` |
| `acme-platform-demo-a` | **Already exists** — independent repo, owned/maintained by its own team, untouched by this doc except for submodule registration and spec-kit bootstrap |
| `acme-platform-demo-b` | **Already exists** — same as above |

The platform team's job is to (1) stand up the workgroup repo, (2) fork the team AI directives, and (3) register all three — the new fork plus the two existing projects — as submodules. Only once that's done does the team move on to bootstrapping spec-kit inside each existing project and running `/levelup.init`.

---

## Key Repositories

| Repo | What it is | How we use it here |
|------|-----------|---------------------|
| [Tikalk spec-kit](https://github.com/tikalk/agentic-sdlc-spec-kit) | A fork of [`github/spec-kit`](https://github.com/github/spec-kit) that adds a **team-directives layer** on top of the vanilla spec/plan/tasks/implement workflow — see below for what's actually different | Installed once in Prerequisites; used throughout via `specify init`, `specify run`, `specify self upgrade` |
| [Tikalk team-ai-directives starter](https://github.com/tikalk/agentic-sdlc-team-ai-directives) | Template repo of constitution/personas/rules/skills that AI agents follow | Forked in Step 1 as a **starting point** — `platform-ai-directives` is ACME's own copy, meant to be customized and grown into the team's real directives via `/levelup.*`, not used as-is |
| [Tikalk 12-factor SDLC](https://github.com/tikalk/agentic-sdlc-12-factors) | Conceptual methodology doc behind agentic SDLC | Background reading only — not cloned, installed, or referenced by any command in this doc |

### What's Different in Tikal's Spec Kit vs. Vanilla

Vanilla `github/spec-kit` is built for a single developer working alone in one repo — the base spec-driven slash commands (spec → plan → tasks → implement) and nothing more. Tikal's fork turns it into a team-wide system: every project shares one evolving directives repo instead of reinventing its own, and the machinery this entire onboarding doc depends on comes from that shift:

| Addition | What it does |
|----------|-------------|
| `specify init --team-ai-directives <url>` | Points a project at a shared directives repo (like `platform-ai-directives`) instead of each project reinventing its own constitution/rules |
| Extension system (`specify extension add/remove/search/catalog`) | Installs the `team-ai-directives` extension into `.specify/extensions/`, which is what makes `specify run adlc.team-ai-directives.*` commands available |
| Workflow engine (`specify workflow run/status/resume`) | Runs multi-step automations — `specify run adlc.team-ai-directives.verify` and `.discover` (Steps 6/7) are workflows under the hood |
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

For now, keep the fork's defaults as-is — don't hand-edit `constitution.md`, `personas/`, `rules/`, or `.skills.json` yet. Customizing these for ACME is what `/levelup.init` is for (see Step 9): it scans your codebases and proposes directive updates as CDRs, which get reviewed and merged back here via `/levelup.implement`.

Tag the defaults so downstream projects have something to pin to. Note: this repo already carries upstream version tags (`v1.0.0`–`v1.8.x`) from the template it was forked from, so reuse of `v1.0.0` will collide — use an `acme-` prefixed tag instead:

```bash
cd platform-ai-directives
git tag acme-v1.0.0
git push origin acme-v1.0.0
```

---

### Step 2 — Create the Workgroup Repository

The workgroup repo is the root umbrella. It doesn't contain source code — only submodule pointers and shared documentation.

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

### Step 3 — Register the Existing Projects and Directives Fork as Submodules

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

### Step 4 — Verify the Full Workgroup Structure

After Steps 1–3, the workgroup directory tree looks like this — note the two demo projects only have their pre-existing content so far; spec-kit hasn't been bootstrapped into them yet:

```
acme-platform/               ← root umbrella repo (acme/acme-platform)
├── .gitmodules               ← submodule registry
├── README.md                 ← workgroup overview
├── ONBOARDING.md              ← this file
├── platform-ai-directives/   ← acme/platform-ai-directives (submodule)
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

### Step 5 — Initialize Spec Kit and Team AI Directives in Each Project

Only now, with the workgroup wired up, do we bootstrap spec-kit inside each existing project and point it at the directives fork:

```bash
cd demo-a
specify init . \
  --ai claude \
  --team-ai-directives https://github.com/acme/platform-ai-directives.git@acme-v1.0.0
git add -A && git commit -m "chore: bootstrap spec-kit + team-ai-directives"
git push

cd ../demo-b
specify init . \
  --ai claude \
  --team-ai-directives https://github.com/acme/platform-ai-directives.git@acme-v1.0.0
git add -A && git commit -m "chore: bootstrap spec-kit + team-ai-directives"
git push

cd ..
```

This adds a `.specify/` directory to each project (see Step 10 for its layout).

---

### Step 6 — Verify Spec Kit Integration in Each Project

Run the built-in health check in each project:

```bash
cd demo-a
specify run adlc.team-ai-directives.verify

cd ../demo-b
specify run adlc.team-ai-directives.verify
```

Both should report all checks green (extension installed, skills loaded, constitution present).

---

### Step 7 — Daily Workflow (per project)

Inside any project directory, the typical spec-driven session looks like:

```bash
# 1. Discover relevant context before starting a feature
specify run adlc.team-ai-directives.discover

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

### Step 8 — Updating the Team AI Directives

When a project team discovers a useful pattern:
1. Use `/spec.levelup` inside the AI session to generate a knowledge packet.
2. PR the change into `platform-ai-directives`.
3. Tag a new release (e.g. `acme-v1.1.0`).
4. Update the `--team-ai-directives` reference in each project's `.specify/` config to point to the new tag.
5. Bump the submodule pointer in `acme-platform/platform-ai-directives`.

---

## Spec Kit CLI Reference (quick cheat-sheet)

| Command | What it does |
|---------|-------------|
| `specify version` | Print installed CLI version |
| `specify self check` | Check if a newer release is available |
| `specify self upgrade` | Upgrade CLI to latest stable release |
| `specify init <dir> --team-ai-directives <url>` | Bootstrap a project with spec-kit + directives |
| `specify run adlc.team-ai-directives.verify` | Health check — extension, skills, CDR, constitution |
| `specify run adlc.team-ai-directives.discover` | Auto-discover relevant context for a feature |

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

## Step 9 — LevelUp Init: Brownfield Codebase Analysis

After the initial directory structure is set up and spec-kit is initialized in each project, Tikalk recommends running `/levelup.init` to scan the existing code, discover reusable patterns, and generate **Context Directive Records (CDRs)** — the mechanism for contributing knowledge back to the shared team-ai-directives.

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

## Step 10 — Minimal Demo Project Structure (Already Created)

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
