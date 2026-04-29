# Install / self-deploy reference

This file is loaded **on demand** when the harness needs to drive a fresh install or finish an interrupted bootstrap. It is the agent-side companion to the canonical runtime install doc at `agentflow-article-publishing/INSTALL.md`.

The agent's job, in one sentence: **drive the operator from "just cloned the runtime repo" to `current_state: ready` without ever asking the operator to type anything except credentials in a terminal**.

## The loop (the only thing you really need)

```bash
af bootstrap --next-step --json
```

returns

```json
{
  "current_state": "...",
  "next_command": "...",
  "reason": "...",
  "stage": "init" | "ready",
  "mode": "harness" | "tg_review" | "unknown",
  "optional_next": "..."   // only present in some "ready" payloads
}
```

The agent does:

```
loop:
    res = af bootstrap --next-step --json
    if res.stage == "ready": break
    execute(res.next_command)
```

That's it. Every other thing in this document is just clarifying what each `current_state` means and how the agent should react.

## State table

| `current_state` | What it means | What the agent should do |
|---|---|---|
| `no_env` | No `~/.agentflow/secrets/.env` AND no legacy `backend/.env` | Run `af bootstrap` (no flags). It seeds the secrets file from `backend/.env.template` at `~/.agentflow/secrets/.env`. |
| `unknown` | Env file exists but couldn't be parsed | Run `af doctor` to surface the file/permission issue, then re-loop. |
| `skills_not_installed` | `~/.claude/skills` and `~/.cursor/skills` are both empty / missing | Run `af skill-install`. Prefer this over manual symlink/copy — it sets correct dirs + perms for the operator's harness. |
| `missing_real_keys` | The chosen LLM provider's key isn't set, AND `MOCK_LLM != true` | Run `af onboard --section <provider>`. **Operator types the key in the terminal**; the agent does NOT receive the key text and does NOT paste it into chat. |
| `missing_profile` | No `~/.agentflow/topic_profiles.yaml` or it has no profiles | Run `af topic-profile init -i --profile <id>` (interactive) OR `af topic-profile init --profile <id> --from-file assets/topic_profile.yaml`. The yaml route is faster and lets the operator pre-fill brand/voice/sources/keywords from the skill's asset template. |
| `incomplete_profile` | A profile exists but lacks brand / voice / 2× do / 2× dont / 3× product_facts / 3× keyword_groups | Same as above — `init -i` or `derive --profile <id>` (LLM reverse-engineers fields from a seed description). |
| `missing_chat_id` (Mode B/C only) | TG token set but bot hasn't received `/start` | Tell the operator: send `/start` to the bot in Telegram; the daemon auto-captures and writes `~/.agentflow/review/config.json`. |
| `daemon_not_running` (Mode B/C only) | Heartbeat missing or > 5min stale | Start the daemon: `af review-daemon &` (foreground, daemonize) OR `systemctl start agentflow-review` (Linux) OR equivalent launchd plist (macOS). |
| `ready` | All checks pass | Stop the loop. Move on to the daily flow (hotspots → write → publish). The payload may include `optional_next` describing how to upgrade Mode A → Mode B/C. |

## Mode resolution

The detector does **not** ask the operator which mode they want. It infers:

| Sees `TELEGRAM_BOT_TOKEN`? | Mode | Daemon? | Init states 5–7 enforced? |
|---|---|---|---|
| No (anywhere — file + process env) | `harness` (Mode A) | No | Skipped |
| Yes (any of: `~/.agentflow/secrets/.env`, `~/.agentflow/secrets/telegram.env`, `os.environ`, `backend/.env`) | `tg_review` (Mode B/C) | Yes | Enforced |

If the operator changes their mind later (Mode A → Mode B), the `ready` payload's `optional_next` field tells them exactly what to set + restart the loop. No code changes.

## Where the agent should NOT touch

- **Never write the operator's API keys into chat or any file the agent generates.** The wizard `af onboard` reads from `stdin` in the operator's terminal; the agent invokes the wizard and waits.
- **Never hand-edit `.env` / `~/.agentflow/secrets/.env`** from a chat session — use `af onboard --section <id>` or `af keys-edit <service>` (which opens `$EDITOR`).
- **Never push an env file to git.** The runtime repo's `.gitignore` covers `.env` / `.env_config*` / `secrets/` / `*.key` / `*.pem`. The deploy bundle's rsync excludes the same patterns + a post-build sanity guard rejects any file matching them.

## Mock-mode demo path

If the operator wants to validate the install end-to-end without burning real API quota:

```bash
af bootstrap --mock --first-run    # writes MOCK_LLM=true; runs the loop
af hotspots --gate-a-top-k 3       # populates ~/.agentflow/hotspots/
af write <hotspot_id> --auto-pick --json
af preview <article_id> --json
af publish <article_id> --force-strip-images --json
af memory-tail --limit 5 --json
```

All of these use the deterministic mocks under `backend/agentflow/shared/mocks/`. No network calls. Once the operator is happy, drop `--mock` from `bootstrap` (or run `af keys-edit <service>` to fill real keys), and re-run the loop — it'll move from `mock` to `ready` once `LLM_PROVIDER` is set + the matching key lands.

## Verification you've reached ready

```bash
af doctor        # readiness matrix per credential
af keys-where    # what file each var resolved from
af bootstrap --next-step --json | jq -r .current_state    # should print "ready"
```

If `ready`, the harness can dispatch to `/agentflow-hotspots`, `/agentflow-write`, `/agentflow-publish` — see those skills' `SKILL.md` for the daily flow.
