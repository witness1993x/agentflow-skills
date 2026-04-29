---
name: agentflow-publish
description: |
  Run image-gate, Gate C/D, preview, publish configured platforms, or prepare Medium manual fallback. The Telegram review daemon is optional and only needed for phone-based Gate C/D approval.

  TRIGGER: "/agentflow-publish", "af preview", "af publish", "Gate C", "Gate D", "image-gate", "publish-rollback", "medium-package", "review-publish-mark", "PR:mark", "I:cover_only", "PD:dispatch".

  SKIP for: modifying D3 platform adapter internals (agent_d3/adapters/); modifying D4 publisher internals (agent_d4/publishers/); editing platform-specific token / OAuth flows.
---

# agentflow-publish — image gate, preview, fan out, or Medium manual

Wraps `af draft-show`, `af image-gate`, `af preview`, `af publish`, `af medium-package`, `af review-publish-mark`, `af publish-rollback`. Goal: move an approved draft through Gate C/D, ship selected platforms.

- **Assets:** `assets/platform_overrides.yaml` — copy, fill, run `af preview --from-file <path>`
- **References:** `references/cli.md` (full flags), `references/gates.md` (Gate C/D logic), `references/troubleshooting.md`

**Platforms (v0.1):**
- `medium` — default manual package; close with `af review-publish-mark <id> <url>`.
- `ghost_wordpress` — optional, env-gated.
- `linkedin_article` — optional, env-gated.
- Twitter / newsletter use sibling skills, not `af publish`.

## Input + setup

Expect `<article_id>`. If missing: ask, or refer to `/agentflow-write`.

```bash
cd <runtime_repo>/backend && source .venv/bin/activate
```

Prefix `af` with `PYTHONPATH=.`.

## Step 1 — inspect

```bash
PYTHONPATH=. af draft-show <article_id> --json
```

Report: "Draft has X sections, Y words, Z unresolved images."

## Step 1b — pre-publish overview

Skip only if user says "just publish". Build:

1. **Lineage** — `metadata.json::{hotspot_id, chosen_angle_index, target_series}`.
2. **References** — top 3-5 `source_references[]` from hotspot.
3. **Compliance** — sections `< 0.85`; worst offender.
4. **Tag quality** — flag tags starting 的/在/对/和 or < 2 chars.
5. **Platform readiness** — Medium manual always ready; Ghost/LinkedIn env-gated.
6. **Preferences** — `af prefs-show --json`.

Pause: "Proceed to platform previews? (yes / edit first / cancel)". If central claim missing from references → flag hallucination. Run `af intent-check <id> --json` (or compute manually). See `references/gates.md` for thresholds.

## Step 2 — image gate (Gate C entry)

Telegram cards `[optional, only for phone-based review]`. In Mode A the operator reviews inline.

```bash
PYTHONPATH=. af image-gate <article_id> --mode cover-only --json   # default
PYTHONPATH=. af image-gate <article_id> --mode none --json         # skip → straight to Gate D
```

Interpret: `gate_c_short_id` = Gate C dispatched; `gate_d_short_id` = none-path → Gate D. If non-JSON suggests `--skip-images`, pass it to preview + `--force-strip-images` to publish.

## Step 3 — preview / 4 — present / 5 — confirm

```bash
PYTHONPATH=. af preview <article_id> --json
PYTHONPATH=. af preview <article_id> --from-file assets/platform_overrides.yaml --json
```

Add `--skip-images` if Step 2 was `--mode none`. For each `platform_versions/<platform>.md` produced, `Read` it; show front-matter + first ~30 lines. Then: "Dispatch selected platforms? (yes / list / cancel)".

## Step 6 — publish

```bash
PYTHONPATH=. af publish <article_id> --platforms <csv> [--force-strip-images] --json
```

For Medium: `manual` package — user pastes it, then `af review-publish-mark <article_id> <url>`.

## Step 7-8 — report + close

Print platform/status/url table. If any non-success: "Retry failed platforms? (yes / no)". Close: "Article `<id>` published to M/N platforms."

## Rollback

Triggered on "undo / 撤下 / takedown / rollback / 回滚". Only Ghost in v0.1. See `references/cli.md` for `af publish-rollback` flags. Always confirm: "This will permanently delete the Ghost post for `<article_id>`. Proceed? (yes / cancel)".

## NEVER

- Never call `agent_d3.*` / `agent_d4.*` directly.
- Never force-strip images silently — confirm.
- Never retry `success` — only retry failed platforms.
- Never publish without a preview.

## MOCK_LLM

Mock publishers return fake URLs. Safe for dry runs.
