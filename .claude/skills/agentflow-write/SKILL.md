---
name: agentflow-write
description: |
  Turn a chosen hotspot into a finished draft via skeleton, fill, and an interactive edit loop. The Telegram review daemon is optional and not required for this skill.

  TRIGGER: "/agentflow-write", "af write", "af fill", "af edit", "write draft", "Gate B", "写稿", "draft", "section edit".

  SKIP for: modifying D2 prompt templates (agentflow/prompts/); running image-gate / Gate C/D (belongs to agentflow-publish).
---

# agentflow-write — from hotspot to finished draft

Wraps `af write`, `af fill`, `af edit`, `af draft-show`. Goal: produce a draft the user is happy with, then hand off to `agentflow-publish` for image-gate and Gate C/D.

- **Assets:** `assets/edit_commands.yaml` — copy, fill, run `af edit --from-file <path>`
- **References:** `references/cli.md`, `references/gates.md`, `references/troubleshooting.md`

## Inputs

Expect `<hotspot_id>` (and optional `--angle N`). If missing, tell user to run `/agentflow-hotspots` first. Hotspot ids resolve from both `~/.agentflow/hotspots/` and `~/.agentflow/search_results/`.

## Setup (every invocation)

```bash
cd <runtime_repo>/backend && source .venv/bin/activate
```

Prefix every `af` call with `PYTHONPATH=.` unless the package is globally installed.

## Step 1 — auto vs manual skeleton

Ask: "auto-write the full draft (recommended) or pick the skeleton yourself?"

**Auto path:**

```bash
PYTHONPATH=. af write <hotspot_id> --angle <N> --auto-pick --json
```

`--auto-pick` consults `~/.agentflow/preferences.yaml` (N≥3 past `fill_choices` events, confidence ≥ 0.5). JSON response includes `defaults_source` (`preferences` | `hardcoded`). Override with `--ignore-prefs` or explicit `--title N --opening N --closing N`. See `references/cli.md` for full flag list.

Parse `article_id` from the JSON; `Read ~/.agentflow/drafts/<article_id>/draft.md`. Jump to Step 3.

**Manual path:**

```bash
PYTHONPATH=. af write <hotspot_id> --angle <N> --json    # returns skeleton
# user picks indices
PYTHONPATH=. af fill <article_id> --title 2 --opening 0 --closing 1 --json
```

## Step 2 — show the draft (with compliance signals)

```bash
PYTHONPATH=. af draft-show <article_id> --json
```

Summarise: title, total word count, section count, **per-section compliance** (sections < 0.70 → call out), unresolved image placeholders.

Known limitation: Kimi often produces 140-180 char paragraphs against 150 target. D3 adapters mechanically split (Ghost 70w, LinkedIn 40w). Compliance < 1.00 on raw D2 is usually a signal not a blocker.

## Step 3 — interactive edit loop

Prompt user with the menu (`section N 改短` / `第 N 段 <cmd>` / `section N paragraph M <cmd>` / `done` / `show` / `regen`). Matching rules and CLI mapping live in `references/cli.md`. Five preset zh commands: `改短` `展开` `改锋利` `加例子` `去AI味`. Anything else passes through free-form. For batch edits use `assets/edit_commands.yaml` (`af edit --from-file`). After each edit, re-Read draft.md and show **only the edited section** with old → new word count.

## Step 3.5 — propose images (optional, default-on)

After `done`, ask: "Run image proposal? (yes — recommended / skip)".

```bash
PYTHONPATH=. af propose-images <article_id> --json
```

Inserts `[IMAGE: <desc>]` placeholders, writes `image_proposals.json`, fires `images_proposed` event. Show numbered placement list; offer accept-all / reject [N] / regenerate.

## Step 4 — hand off

`af draft-show <article_id> --json`. Then: "Draft `<article_id>` is ready. Run `/agentflow-publish <article_id>` to choose image mode and continue Gate C/D." Memory events: `article_created`, `fill_choices`, `section_edit`, `images_proposed`. Each `af write` creates a fresh `article_id`.

## NEVER

- Never call `agentflow.agent_d2.*` directly. Only via `af`.
- Never modify `draft.md` / `metadata.json` by hand.
- Never invoke another skill via Skill tool mid-loop.

## MOCK_LLM

`MOCK_LLM=true af write --auto-pick` → deterministic.
