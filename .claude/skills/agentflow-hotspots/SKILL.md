---
name: agentflow-hotspots
description: |
  Scan trending topics, present them, and help the user pick one hotspot + angle. The Telegram review daemon is optional and not required for this skill.

  TRIGGER: "/agentflow-hotspots", "af hotspots", "热点", "topic discovery", "Gate A", "选题", "today's topic".

  SKIP for: modifying the hotspots scoring algorithm (D1 internals are not skill orchestration); editing publisher_account.keyword_groups (topic_profile domain).
---

# agentflow-hotspots — pick today's topic

Wraps `af hotspots` and `af hotspot-show`. Goal: leave the user with one `<hotspot_id>` + `<angle_index>` chosen, then hand off to `agentflow-write`.

- **Assets:** `assets/sources.yaml` — copy, fill, run `af hotspots --from-file <path>`
- **References:** `references/cli.md`, `references/troubleshooting.md`

## Step 0 — detect a topic intent in the user's message

- **Broad scan**: "跑一遍热点 / scan / today's hotspots" → no filter.
- **Topic-targeted**: "帮我找 X / 最近 Y 怎么说 / `--filter X`" → extract topic phrase as `--filter` regex.

For multi-word or OR alternatives: `MCP server` → `"MCP|server|agent"`. Err **broad** — too narrow returns 0.

## Step 0b — choose the topic profile id

Resolve in this order: explicit `--profile` → active TopicIntent in `~/.agentflow/intents/current.yaml` → `AGENTFLOW_DEFAULT_TOPIC_PROFILE` → ask. Never silently reuse another bot's profile.

## Step 1 — scan

Confirm profile is at least minimally usable first (`af topic-profile show --profile <id> --json`); see `references/troubleshooting.md` for what to do on missing fields.

```bash
cd <runtime_repo>/backend && source .venv/bin/activate
PYTHONPATH=. af hotspots --profile <id> --json                          # broad
PYTHONPATH=. af hotspots --profile <id> --filter "MCP|agent" --json     # targeted
PYTHONPATH=. af hotspots --from-file assets/sources.yaml --json         # custom sources
```

For large JSON, redirect and `Read /tmp/af_hotspots.json`.

## Step 2 — present numbered list

From `hotspots[]`:

```
[1] <topic_one_liner> — series:<recommended_series> freshness:<freshness_score>
    overlooked: <overlooked_angles[0]>
[2] ...
```

Keep top 5–8. If `filter` present, surface compact filter panel with `boundary.level`. Show `filtered_out_preview` only when level is `too_narrow` / `narrow` or user asks.

If `matched == 0`, tell user "0 matched out of N. Widen the query?" — never silently fall back.

## Step 3 — prompt for a pick

> "Pick a number (1–N), 'write all', or 'skip' to stop."

- Number → Step 4.
- "write all" → loop Step 4 (advise only with `MOCK_LLM=true`).
- "skip" → stop, no state mutation.

## Step 4 — show angles

```bash
PYTHONPATH=. af hotspot-show <hotspot_id> --json
```

Enumerate `suggested_angles[]`:

```
Angle [0] <angle> — depth:<depth> difficulty:<difficulty>
         fit: <fit_explanation>
```

Ask: "Which angle index? (default 0)"

## Step 5 — hand off

> "Run `/agentflow-write <hotspot_id> --angle <N>`."

If user says yes/go, continue write workflow inline (don't invoke another skill via Skill tool).

## Downstream + state

Each hotspot carries `source_references[]`; `agentflow-publish` Step 1b uses these for hallucination checks. Warn user if 0 references. `af hotspots` writes `~/.agentflow/hotspots/<YYYY-MM-DD>.json` (overwrites same-day); search archives in `~/.agentflow/search_results/`. Scans may surface config suggestions — review/apply via `af topic-profile`.

## NEVER

- Never import agentflow modules. Use the CLI.
- Never pick an angle silently — always surface options.
- Never proceed if `suggested_angles` empty; re-scan or pick another hotspot.

## MOCK_LLM

`MOCK_LLM=true af hotspots --json` returns 5 deterministic fixtures.
