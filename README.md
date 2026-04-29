# agentflow — Skill Distribution

**Version:** see [`VERSION`](VERSION) (currently `1.0.3`) — release notes in [`CHANGELOG.md`](CHANGELOG.md). Versioning policy: skill API compatibility against the canonical runtime, not code parity. See the [Versioning](#versioning) section at the bottom.

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
├── agentflow/                     ← top-level workflow guide
│   ├── SKILL.md
│   ├── references/                  cli, gates, troubleshooting, examples,
│   │                                daemon-when-needed
│   └── assets/topic_profile.yaml    af topic-profile init --from-file
├── agentflow-hotspots/            ← /hotspots → Gate A
│   ├── SKILL.md
│   ├── references/                  cli, troubleshooting
│   └── assets/sources.yaml          af hotspots --from-file
├── agentflow-write/               ← /write   → Gate B
│   ├── SKILL.md
│   ├── references/                  cli, gates, troubleshooting
│   └── assets/edit_commands.yaml    af edit --from-file
├── agentflow-publish/             ← /publish → Gate C/D + dispatch
│   ├── SKILL.md
│   ├── references/                  cli, gates, troubleshooting
│   └── assets/platform_overrides.yaml  af preview --from-file
├── agentflow-tweet/               ← Twitter thread / single
│   ├── SKILL.md
│   ├── references/                  cli, troubleshooting
│   └── assets/thread_template.yaml  af tweet-draft --from-file
├── agentflow-newsletter/          ← Resend email digest
│   ├── SKILL.md
│   ├── references/                  cli, troubleshooting
│   └── assets/sections.yaml         af newsletter-draft --from-file
├── agentflow-style/               ← voice profile from samples
│   ├── SKILL.md
│   ├── references/                  cli, troubleshooting
│   └── assets/style_tuning.yaml     af learn-style --from-file
└── README.md                      ← skill-index / install hints

LICENSE
.gitignore
README.md  (this file)
```

Each skill: slim `SKILL.md` (≤100 lines, trigger + orchestration) + `references/` (deep-read on demand) + `assets/` (YAML templates consumed via `af … --from-file`).

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
af bootstrap --next-step --json
```

For Linux VM deploy: download a deploy bundle (`scripts/build_deploy_bundle.sh`
in the canonical runtime) and run `sudo bash deploy.sh` — handles venv +
systemd + chmod 600 in one shot. See `agentflow-deploy/INSTALL_LINUX.md`.

For a local mock/demo first run, use:

```bash
af bootstrap --mock --first-run --start-daemon
```

For real-key setup, follow `af bootstrap --next-step --json` one returned
`next_command` at a time. Credentials are entered in the terminal through
`af onboard` / `af onboard --section <id>`, not pasted into chat and not
hand-edited into `.env`.

After this, `af` is on `$PATH` or available at `backend/.venv/bin/af`.

### 2. Drop these skills into your workspace

Use the canonical runtime command:

```bash
af skill-install
```

Manual copy/symlink is a fallback only when the runtime CLI is not available.
The CLI path keeps host-specific directories and permissions consistent for
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
