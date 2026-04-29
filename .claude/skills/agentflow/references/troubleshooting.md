# Troubleshooting — top-level

Common failure modes when bootstrapping or driving the pipeline. For platform-specific publish errors see `agentflow-publish/references/troubleshooting.md`.

## "af command not found"

The runtime repo is not installed, or the venv is not active.

```bash
cd <runtime_repo>/backend
python3 -m venv .venv
source .venv/bin/activate
pip install -e .
which af   # should now resolve
```

## "no such file: ~/.agentflow/"

Profile dir was never initialized. Run:

```bash
af review-init       # creates the dir layout
af bootstrap --next-step --json
```

## "af doctor probe N stale"

Probe 13 (heartbeat) is **only meaningful in Mode B / Mode C** where the optional Telegram-review daemon is supposed to be running. In Mode A (harness-only, default) probe 13 stale is expected and not blocking. Disable the warning by skipping the optional daemon entirely or check `references/daemon-when-needed.md` to confirm which mode you're in.

## "missing topic profile fields"

`af topic-profile show --profile <id> --json` lists missing fields. Fastest fix:

```bash
af topic-profile init --profile <id> --from-file <skill>/assets/topic_profile.yaml
```

Edit the yaml first to match your publisher.

## "MockLLM returns same skeleton every run"

Expected. `MOCK_LLM=true` is deterministic by design — drop the env var for real LLM.

## ".env credentials silently failing"

You probably edited `.env` by hand. Re-run `af onboard --section <id>` so the per-section probe runs and validates the credential. Skipping `af onboard` bypasses validation — `af doctor` may still report green while a section silently fails.

## "Article published but audit shows force=true"

`triggers.post_publish_ready` has a force-rewind guard. Any `force=True` from `STATE_PUBLISHED` requires explicit decision + audit log entry. Check `~/.agentflow/review/audit.jsonl`.

## "Telegram bot not responding"

Only relevant in Mode B / Mode C (optional Telegram-review setup). Check `af doctor` probe 13. Restart the optional daemon: `af review-daemon`. If `chat_id` is missing, send `/start` in Telegram so the optional daemon captures it. Never hand-edit `~/.agentflow/review/config.json`.

## "Hotspot generated but article drifts off-topic"

LLM hallucinated. Check `metadata.json::section[].compliance_score < 0.85` and the section's `taboo_violations`. Also check `references[]` — if the hotspot has 0 `source_references`, downstream alignment can't be verified and the LLM is more likely to wander.

## "Skill not registering in Claude Code"

```bash
af skill-install --copy --force
```

Then restart Claude Code session. Manual `ln -s` works as a fallback but may not respect host permission setup; prefer `af skill-install`.
