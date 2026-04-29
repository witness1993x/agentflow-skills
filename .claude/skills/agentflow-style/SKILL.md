---
name: agentflow-style
description: |
  Teach or refresh the AgentFlow voice profile from sample past articles. The Telegram review daemon is optional and not used by this skill.

  TRIGGER: "/agentflow-style", "learn-style", "learn-from-handle", "style profile", "voice profile", "samples", "学风格".

  SKIP for: inspecting / editing an existing style_profile.yaml directly (raw yaml editing is out of skill scope); non-D0 topics.
---

# agentflow-style — learn or refresh the voice profile

Wraps `af learn-style`. Use when the user wants to (re)build the style profile at `~/.agentflow/style_profile.yaml` from past articles.

- **Assets:** `assets/style_tuning.yaml` — copy, fill, run `af learn-style --from-file <path>`
- **References:** `references/cli.md`, `references/troubleshooting.md`

## Decide the mode from the user's message

1. User provided file paths / URLs / a directory → use them directly with `--file` / `--url` / `--dir`.
2. "show the current profile" → `--show`.
3. "recompute from corpus" → `--recompute`.
4. "re-ingest my published drafts" → `--from-published`.
5. Otherwise ask: "Paste paths or URLs of 3–5 past articles (one per line), or say 'show' / 'recompute' / 'from-published'."

## Quick invoke

From the project root (venv at `backend/.venv/`):

```bash
cd <runtime_repo>/backend
source .venv/bin/activate
PYTHONPATH=. af learn-style --file /path/to/a.md --file /path/to/b.docx --url https://example.com/post
```

For tuning the style harvesting itself (paragraph length targets, banned phrases, must-haves), pass the asset:

```bash
PYTHONPATH=. af learn-style --from-file assets/style_tuning.yaml --file /path/to/a.md
```

See `references/cli.md` for all flags (`--dir`, `--show`, `--recompute`, `--from-published`).

## After the command succeeds

Run `af learn-style --show` and summarise in 6–10 lines from the YAML output:

- `voice_principles` — 3 bullets.
- `taboos` — 3–5 phrases the user wants to avoid.
- `paragraph_preferences` — typical min/max paragraph length.
- Number of corpus entries.
- Path to saved profile (`~/.agentflow/style_profile.yaml`).

Don't dump raw YAML — distil.

## Hand-off

After the profile is updated, suggest: "Run `/agentflow-hotspots` to scan for today's topics, or `/agentflow-write <hotspot_id>` if you already have one."

## MOCK_LLM note

`MOCK_LLM=true af learn-style --file ...` ingests real files but routes LLM calls (per-article analyses + aggregation) through fixtures. Real-key run on 2 samples ≈ 20-30s.

## NEVER

- Never import `agentflow.agent_d0` directly. Use `af learn-style`.
- Never write `~/.agentflow/style_profile.yaml` by hand.

## Error handling

If `af learn-style` exits non-zero, see `references/troubleshooting.md`. Quick check: `tail -n 20 ~/.agentflow/logs/agentflow.log`.
