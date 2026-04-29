# `af` top-level CLI reference

Commands reachable from the top-level skill. Sibling skills cover their own subcommands in their own `references/cli.md`.

## Bootstrap & onboarding

| Command | Purpose | Common flags |
|---|---|---|
| `af bootstrap --next-step --json` | Iterative real-key onboarding. Returns `next_command` to run. | `--next-step`, `--json` |
| `af bootstrap --mock --first-run` | One-shot mock setup (harness flow, no daemon). | `--mock`, `--first-run`, `--start-daemon` (optional, TG-review only) |
| `af onboard` | Interactive credential capture (terminal hides input). | `--section <id>` |
| `af doctor` | 13-probe health check. | `--json` |
| `af doctor --json` | Machine-readable doctor output. | — |
| `af skill-install` | Symlink/copy skills into `~/.claude/skills/`. | `--cursor`, `--copy`, `--force` |
| `af review-init` | Create `~/.agentflow/` data dir. | — |

## Topic profile

| Command | Purpose |
|---|---|
| `af topic-profile show --profile <id> --json` | Inspect a profile (missing fields, completeness). |
| `af topic-profile init -i --profile <id>` | Interactive profile creation. |
| `af topic-profile init --profile <id> --from-file assets/topic_profile.yaml` | Non-interactive from yaml. |
| `af topic-profile update --profile <id> --from-file <patch.yaml>` | Patch a subset of fields. |
| `af topic-profile derive --profile <id>` | LLM-derive keyword_groups / do / dont from default_description. |
| `af topic-profile suggestion-list --profile <id>` | List unreviewed suggestions. |
| `af topic-profile review <suggestion_id>` | Show one suggestion. |
| `af topic-profile apply <suggestion_id>` | Approve and write into profile. |

## Memory & state

| Command | Purpose |
|---|---|
| `af memory-tail` | Show last N events from `memory/events.jsonl`. |
| `af intent-clear` | Clear active TopicIntent. |
| `af intent-check <article_id> --json` | Score article-vs-intent alignment. |
| `af prefs-show --json` | Show learned preferences (default platforms, override flags). |

## Optional Telegram review daemon (opt-in)

These are **only** for Mode B / Mode C (phone-based approval, optional daemon). See `references/daemon-when-needed.md` first.

| Command | Purpose |
|---|---|
| `af review-daemon` | Start the optional daemon (foreground / systemd / launchd). |
| `af review-cron-install --times "..."` | Install scheduled hotspot scans. |

## --json contract

All `af <cmd> --json` write pure JSON to stdout. Logs (collector progress, LLM calls, SSL warnings) go to **stderr**. To pipe: `af hotspots --json 2>/dev/null | jq .`.

## MOCK_LLM toggle

Prefix any command with `MOCK_LLM=true` to use canned fixtures (deterministic, instant). `.env` ships with `MOCK_LLM=false`; per-command override is safe.
