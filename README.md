# agentflow — Skill Distribution

> ⚠ **READ THIS FIRST — what this repo is and isn't**
>
> This repo is **skill instructions only** — 7 `SKILL.md` files describing how
> Claude Code / Cursor should drive the AgentFlow content pipeline.
>
> **It contains NO Python code, NO `pyproject.toml`, NO `requirements.txt`,
> NO `.service` unit, NO `agentflow-deploy/` tarball, NO `backend/`, NO
> `prompts/`, NO `agentflow/shared/mocks/` fixtures.**
>
> Don't review this repo for "missing dependencies" / "deployment gaps" /
> "venv setup". Those things live in the **canonical runtime repo** —
> [`agentflow-article-publishing`](#canonical-runtime-repo) — which is
> deployed independently to a Linux VM and exposes the `af` CLI on PATH.
>
> When a SKILL.md says `run af hotspots`, that command is provided by the
> canonical runtime, not by this repo.

---

## What's actually here

```
.claude/skills/
├── agentflow/SKILL.md             ← top-level workflow guide
├── agentflow-hotspots/SKILL.md    ← /hotspots → Gate A
├── agentflow-write/SKILL.md       ← /write   → Gate B
├── agentflow-publish/SKILL.md     ← /publish → Gate C/D + dispatch
├── agentflow-tweet/SKILL.md       ← Twitter thread / single
├── agentflow-newsletter/SKILL.md  ← Resend email digest
├── agentflow-style/SKILL.md       ← voice profile from samples
└── README.md                      ← skill-index / install hints

LICENSE
.gitignore
README.md  (this file)
```

**Total size: ~90 KB / 11 files.** That's the entire repo.

## Canonical runtime repo

The runtime (D0/D1/D2/D3/D4 Python pipeline, Telegram review daemon, prompts,
config-examples, deploy assets, full test suite) lives **here**:

> [`witness1993x/agentflow-article-publishing`](https://github.com/witness1993x/agentflow-article-publishing) (separate public repo)

Deploy + dependencies + Pillow + service unit + .env layout etc. are all
documented there, not here.

## How the two repos talk to each other

```
┌─────────────────────────────────────────────────────────────┐
│  THIS REPO (openclaw, ~90 KB)                               │
│  ┌────────────────────────────────────┐                     │
│  │ .claude/skills/agentflow-write/    │                     │
│  │   SKILL.md  ← "run af write …"    │                     │
│  └─────────────────┬──────────────────┘                     │
│                    │ Claude Code / Cursor                   │
│                    │ shells out via Bash tool               │
└────────────────────┼────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  CANONICAL RUNTIME (agentflow-article-publishing)           │
│  ┌────────────────────────────────────┐                     │
│  │ af  ← installed CLI (in PATH)     │                     │
│  │  → agentflow.cli.commands         │                     │
│  │  → D0 / D1 / D2 / D2.5 / D3 / D4  │                     │
│  │  → LLM / Atlas / Telegram /       │                     │
│  │    Ghost / LinkedIn / Twitter /   │                     │
│  │    Webhook / Resend               │                     │
│  └────────────────────────────────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

Real work happens entirely on the runtime side. This repo just ships the
prompts that tell Claude / Cursor how to orchestrate the runtime.

## Setup

### 1. Install the canonical runtime once (separate repo)

```bash
git clone https://github.com/witness1993x/agentflow-article-publishing.git
cd agentflow-article-publishing/backend
python3 -m venv .venv && source .venv/bin/activate
pip install -e .
af doctor                 # verify keys
```

For Linux VM deploy: download a deploy bundle (`scripts/build_deploy_bundle.sh`
in the canonical runtime) and run `sudo bash deploy.sh` — handles venv +
systemd + chmod 600 in one shot. See `agentflow-deploy/INSTALL_LINUX.md`.

After this, `af` is on `$PATH`.

### 2. Drop these skills into your workspace

Either:
- Copy `.claude/skills/` into the project where you want skills active, or
- Symlink: `ln -s /path/to/openclaw/agentflow-1.0.0/.claude/skills <project>/.claude/skills`

The canonical runtime ships `af skill-install` which automates this and is
the recommended path — it handles the symlink + permissions for both
Claude Code and Cursor configs.

### 3. Use

Type `/agentflow-hotspots` (or any of the 7 skill commands) in Claude Code;
the skill body tells Claude to shell out to `af` for you.

## A note on path references inside SKILL.md

Some SKILL.md docs say things like `backend/.venv/` or
`backend/agentflow/shared/mocks/email_newsletter.json`. **Those paths point
into the canonical runtime install dir, not into this repo.** If you're
chasing one of them, look in your local clone of `agentflow-article-publishing`,
not here.

## Versioning

This distribution tracks the canonical runtime by **skill API
compatibility** — i.e. by the `af <subcommand>` interface — not by line-for-line
code parity. A SKILL.md change happens only when the underlying CLI surface
changes meaningfully, which is rare.
