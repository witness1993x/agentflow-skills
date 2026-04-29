# Gate definitions (A → B → C → D)

AgentFlow's pipeline has four review checkpoints. Each gate corresponds to a state machine transition and an optional Telegram review card. The daemon is **only required for phone-based approval** — in harness mode (Mode A) the operator approves/rejects in chat reply.

## Gate A — topic selection

- **Owner skill:** `agentflow-hotspots`
- **State transition:** `STATE_HOTSPOTS_FRESH` → `STATE_HOTSPOT_PICKED`
- **Inputs:** scan output at `~/.agentflow/hotspots/<YYYY-MM-DD>.json`
- **Decision artefact:** `<hotspot_id>` + `<angle_index>`
- **TG-review only:** `A:approve`, `A:reject`. In Mode A, just say "pick 2, angle 0".

## Gate B — draft acceptance

- **Owner skill:** `agentflow-write`
- **State transition:** `STATE_DRAFT_FILLED` → `STATE_DRAFT_ACCEPTED`
- **Inputs:** filled draft at `~/.agentflow/drafts/<article_id>/draft.md`, `metadata.json::sections[].compliance_score`, `taboo_violations[]`
- **Decision matrix:**
  - clean (avg ≥ 0.85, 0 violations) → approve → image-gate
  - 1 section degraded → `af edit` that section
  - 1 section broken (compliance < 0.5) → `af fill` rewrite (max 2 rounds)
  - multiple sections broken → reject + reselect hotspot
  - LLM fabricated key fact → edit + add source URL or remove
- **TG-review only:** `B:approve`, `B:edit`, `B:rewrite`, `B:reject`.

## Gate C — image / cover

- **Owner skill:** `agentflow-publish`
- **State transition:** `STATE_DRAFT_ACCEPTED` → `STATE_IMAGES_RESOLVED` (or `STATE_IMAGES_SKIPPED`)
- **Inputs:** `metadata.json::image_placeholders[]`
- **Modes:** `cover-only` (default) / `cover-plus-body` / `none` (skip → straight to Gate D)
- **TG-review only:** `I:cover_only`, `I:cover_plus_body`, `I:none`. In harness mode, pass `--mode` to `af image-gate`.

## Gate D — platform dispatch

- **Owner skill:** `agentflow-publish`
- **State transition:** `STATE_IMAGES_RESOLVED` → `STATE_PREVIEW_READY` → `STATE_PUBLISHED`
- **Inputs:** `~/.agentflow/drafts/<article_id>/d3_output.json`, per-platform `platform_versions/<platform>.md`
- **Default platform:** `medium` (manual package, no creds needed). Optional configured channels: `ghost_wordpress`, `linkedin_article`.
- **TG-review only:** `PD:dispatch`, `PR:mark`. In harness mode, pass `--platforms <csv>` to `af publish` and run `af review-publish-mark <article_id> <url>` for Medium.

## State machine reference

The full state list (14 STATE_*) lives in the runtime repo at `backend/agentflow/agent_review/state.py`. The earlier 5-state model ("approved/skeleton/draft/preview/published") is obsolete.
