# Gate C + Gate D — image and dispatch

Owner of Gate C (image / cover) and Gate D (platform dispatch). Telegram cards for both are **optional** and only emitted when the review daemon is running (Mode B / Mode C). In Mode A (default, harness-only) the operator decides inline. See `../agentflow/references/daemon-when-needed.md`.

## Gate C — image / cover

### Mode selection

| Mode | Effect | Next step |
|---|---|---|
| `cover-only` | Generate one cover image at the article's hero anchor. | `af preview` (no flag). |
| `cover-plus-body` | Generate cover + every non-cover `[IMAGE: ...]` placeholder. | `af preview`. |
| `none` | Skip generation entirely. State goes `image_skipped`. | `af preview --skip-images`. |

### Decision shortcuts

- New publisher / brand still being defined → `cover-only`. Cheapest, lowest risk.
- High-stakes article, lots of in-body diagrams → `cover-plus-body`.
- MOCK_LLM dry run / no Pillow / Atlas down → `none`.

### TG-review only buttons

`I:cover_only`, `I:cover_plus_body`, `I:none`. Mode A operator just passes `--mode <choice>` to `af image-gate` directly.

## Gate D — platform dispatch

### Default platform: medium

Manual package always available. Doesn't need any API token. After paste, close loop with `af review-publish-mark <id> <url>`.

### Optional platforms

Ghost / LinkedIn require env credentials and **opt-in** in `--platforms`. Never auto-include — user or preferences must select them.

### Ghost status override

Recent rollbacks bias the next publish toward `status=draft`:

```yaml
# ~/.agentflow/preferences.yaml (auto-managed)
publish:
  ghost_status_override: draft
  override_remaining_runs: 2
```

Surface this in Step 1b. Override decays after N runs unless re-triggered.

### Reference alignment check

Step 1b mandate. Pull top 3-5 `source_references[]` from the hotspot file. If the article's central claim doesn't map to any → flag potential hallucination. Don't auto-block — surface and let user decide.

### Intent alignment check

Read `~/.agentflow/intents/current.yaml::query.text`. Tokenize (lowercase, drop tokens < 2 chars, plus 2-char sliding windows for CJK). Check overlap with title + first heading + opening (or first 500 chars of section 0).

| Tokens matched | Action |
|---|---|
| 0 / N | Hard flag: `intent_drift`. Offer rewrite / clear-intent / proceed. |
| 1 to N/2 | Soft note: "intent partial match". |
| ≥ N/2 | Stay quiet. |

`af intent-check <article_id> --json` returns `alignment_score`; same thresholds at 0.1 / 0.5.

### TG-review only buttons

`PD:dispatch`, `PR:mark`. Mode A operator passes `--platforms <csv>` to `af publish` and runs `af review-publish-mark` for Medium manually.

## Rollback

Triggered post-publish. Only Ghost. Always confirm with the user before deleting; the API call is destructive and Ghost gives no undo. See `references/cli.md` `af publish-rollback` for flag reference.
