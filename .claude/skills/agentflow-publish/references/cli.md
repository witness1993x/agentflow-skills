# Publish CLI reference

D3 (preview / adapters) + D4 (publishers) + Gate C/D entrypoints. All operate on `~/.agentflow/drafts/<article_id>/`.

## `af image-gate <article_id>`

Gate C entry. Generates cover image (or skips), then posts the appropriate gate card. Telegram cards are **only** dispatched in Mode B / Mode C (optional, opt-in daemon-based review).

| Flag | Effect |
|---|---|
| `--mode cover-only` | Generate cover; post Gate C. Default. |
| `--mode cover-plus-body` | Generate cover + body images; post Gate C. |
| `--mode none` | Skip generation; transition to `image_skipped`; post Gate D directly. |
| `--json` | JSON return: `{gate_c_short_id?, gate_d_short_id?}`. |

## `af preview <article_id>`

D3 adapters: convert one draft into per-platform markdown.

| Flag | Effect |
|---|---|
| `--platforms <csv>` | Comma list: `medium,ghost_wordpress,linkedin_article`. Defaults to learned preferences or `medium`. |
| `--skip-images` | Strip image placeholders before adapting. Required when image-gate was `--mode none`. |
| `--from-file <yaml>` | Apply per-platform overrides from `assets/platform_overrides.yaml`. |
| `--json` | JSON output. |

Writes:
- `~/.agentflow/drafts/<id>/d3_output.json` — per-platform structured form.
- `~/.agentflow/drafts/<id>/platform_versions/<platform>.md` — human-readable preview.

## `af publish <article_id>`

D4 dispatch. Sends previewed content to each platform's API (or generates manual package for Medium).

| Flag | Effect |
|---|---|
| `--platforms <csv>` | Required. Subset of platforms previewed in Step 3. |
| `--force-strip-images` | Required when `image-gate --mode none` was used; otherwise CLI rejects. |
| `--json` | JSON: `{results: [{platform, status, url?, failure_reason?, platform_post_id?}]}`. |

For Medium, `status: "manual"` and a `package` field — user pastes manually.

## `af medium-package <article_id>`

Build the Medium manual paste package (markdown + front-matter). Idempotent.

```bash
PYTHONPATH=. af medium-package <article_id>
# writes ~/.agentflow/drafts/<id>/medium/package.md
```

## `af review-publish-mark <article_id> <url>`

Close the loop after manual Medium paste. Records the published URL into `metadata.json` and `publish_history.jsonl`. Required when Medium is in `--platforms`; otherwise the article stays in `STATE_PREVIEW_READY`.

## `af publish-rollback <article_id>`

Take down a published post. Only Ghost in v0.1.

| Flag | Effect |
|---|---|
| `--platform <name>` | Default `ghost_wordpress`. |
| `--post-id <id>` | Override platform_post_id (legacy fallback when history record predates the field). |
| `--json` | JSON output. |

Side effects: appends `status=rolled_back` to `publish_history.jsonl`, writes `publish_rolled_back` to `memory/events.jsonl`, removes platform from `metadata.json::published_platforms` (reverts `status` to `preview_ready` if last).

## `af draft-show <article_id>`

Inherited from write skill — see `agentflow-write/references/cli.md`. Used here for the Step 1 inspection.
