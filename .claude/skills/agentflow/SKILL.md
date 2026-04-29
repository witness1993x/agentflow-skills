---
name: agentflow
description: |
  Top-level AgentFlow guide. Default entry is first deployment / onboarding continuation: verify runtime repo, venv, .env, skills, profile, then guide through af bootstrap/onboard/topic-profile/doctor before daily writing. The optional Telegram review daemon is only for phone-based approval gates.

  TRIGGER: "agentflow", "/agentflow", "first run", "setup", "init", "bootstrap", "onboard", "af doctor", "daily workflow", "hotspot to publish".

  SKIP for: single sub-tasks (use a more specific sibling skill); pure bash / grep; editing ~/.agentflow/topic_profiles.yaml user data.
---

# agentflow — first deployment, then daily workflow

AgentFlow is a single-user article pipeline. This skill is the entry point; it defaults to first deployment / onboarding continuation, then dispatches to task-specific sibling skills once runtime is healthy.

When the user invokes this skill without a clear subtask, do **not** jump to hotspots/write/publish. Ask whether this is mock demo or real-key setup, then run the init ladder below.

- **Assets:** `assets/topic_profile.yaml` — copy, fill, run `af topic-profile init --from-file <path>`
- **References:** `references/cli.md`, `references/gates.md`, `references/troubleshooting.md`, `references/examples.md`, `references/daemon-when-needed.md` (optional Telegram-review-only daemon guide)

## Sibling skills

- `agentflow-style` — voice profile from samples (`af learn-style`).
- `agentflow-hotspots` — pick today's topic (`af hotspots`).
- `agentflow-write` — hotspot → draft (`af write`, `af fill`, `af edit`).
- `agentflow-publish` — image-gate, Gate C/D, dispatch (`af preview`, `af publish`).
- `agentflow-tweet` — Twitter/X single + thread.
- `agentflow-newsletter` — Resend newsletter.

## Default entry — first deployment / onboarding continuation

Resolve current state in this order:

1. Runtime repo exists and contains `backend/agentflow/`.
2. CLI exists at `backend/.venv/bin/af`, or `af` on PATH.
3. `backend/.env` exists.
4. `~/.agentflow/` exists and profile data is initialized.
5. `af doctor` is clean enough for the requested mode.

Use framework commands only:

```bash
af bootstrap --next-step --json
```

For mock/demo:

```bash
af bootstrap --mock --first-run     # harness flow; daemon is optional
# add --start-daemon only if you want phone-based Telegram approval gates
```

For real-key setup, guide one returned `next_command` at a time. Credentials must be typed by the user in the terminal via `af onboard`; never paste API keys into chat and never edit `.env` by hand.

The Telegram review daemon is **optional** — required only when the operator wants phone-based approve/reject. See `references/daemon-when-needed.md` to pick Mode A (harness-only, default), Mode B (TG-review), or Mode C (hybrid).

## Daily flow

0. Profile drift check: `af doctor`, `af topic-profile show --profile <id> --json`. Fill missing fields via `af topic-profile init --profile <id> --from-file assets/topic_profile.yaml`. Suggestion-first updates: `af topic-profile suggestion-list/review/apply`.
1. (Weekly) `/agentflow-style` — refresh voice profile.
2. `/agentflow-hotspots` — pick today's topic.
3. `/agentflow-write <hotspot_id>` — skeleton, fill, edit.
4. `/agentflow-publish <article_id>` — preview, fan out or Medium manual package.

Profile setup is part of the main flow. Resolve profile id in this order: explicit `--profile`, active TopicIntent, `AGENTFLOW_DEFAULT_TOPIC_PROFILE`, then ask. Do not silently reuse a profile from another bot/project. Minimal profile path: `account_config_pending → publisher_profile_pending → learning_sources_pending → extra_profile_pending → profile_suggestions_pending → profile_ready`. Only `publisher_profile_pending` is blocking for first use.

## State lives at `~/.agentflow/`

- `hotspots/<YYYY-MM-DD>.json`, `search_results/*.json`
- `drafts/<article_id>/{metadata.json, draft.md, skeleton.json, d3_output.json, platform_versions/<platform>.md}`
- `memory/events.jsonl` — append-only mutation log
- `intents/current.yaml`, `publish_history.jsonl`
- `style_profile.yaml`, `topic_profiles.yaml`
- `constraint_suggestions/*.json`, `constraint_sessions/*.json`
- `logs/agentflow.log` — tail on any CLI failure

## Cross-cutting rules

- Never import `agentflow.*` Python modules. Use `af`.
- Prefer `--json`; parse inline. Use `Read` tool for large payloads.
- On any non-zero exit: `tail -n 20 ~/.agentflow/logs/agentflow.log` and surface.
- `MOCK_LLM=true` for deterministic dry runs.
- Constraint changes via `af topic-profile show/init/update/suggestion-list/review/apply`, never raw yaml edits.
- Output language follows profile's `publisher_account.output_language` (default `zh-Hans`); treat as hard prompt constraint.

### Topic intent

When the user implies a topic, treat as **TopicIntent** threading the whole session: hotspots `--filter`, write keeps it in edit prompts, publish Step 1b checks alignment. Every use writes a `topic_intent_used` memory event.

## When invoked standalone

If user just says "agentflow" without a target, start with:

> "我先按首次部署/初始化续跑检查。你要走 mock demo 还是 real-key？如果你已经 ready，我会继续到 profile / style / hotspots / write / publish。"
