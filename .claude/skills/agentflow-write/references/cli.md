# Write CLI reference

`af write`, `af fill`, `af edit`, `af draft-show`, `af propose-images`. All operate on `~/.agentflow/drafts/<article_id>/`.

## `af write <hotspot_id>`

Generate a skeleton (or full auto-pick draft) from a hotspot.

| Flag | Effect |
|---|---|
| `--angle N` | Choose angle from `suggested_angles[N]`. Required for non-auto runs. |
| `--auto-pick` | Run skeleton + fill in one shot using preferences-default indices. |
| `--ignore-prefs` | Force `0/0/0` defaults (skip preferences lookup). |
| `--title N --opening N --closing N` | Override defaults explicitly. |
| `--json` | JSON to stdout. Returns `{article_id, skeleton, defaults_source}`. |

Returns `article_id` — keep it for every later step. With `--auto-pick`, also returns full draft and the JSON includes `defaults_source` (`"preferences"` | `"hardcoded"`) and `defaults_source_events` (count of past runs).

## `af fill <article_id>`

Fill an empty skeleton into a draft.

| Flag | Effect |
|---|---|
| `--title N --opening N --closing N` | Required: indices into the skeleton's `*_candidates`. |
| `--json` | JSON output. |

Re-runnable: passing different indices regenerates `draft.md` and bumps `metadata.json::fill_history[]`.

## `af edit <article_id>`

Targeted edit of one section/paragraph/whole article.

| Flag | Effect |
|---|---|
| `--section N` | Section index. Required (use 0 for top-level). |
| `--paragraph M` | Optional paragraph within the section; null = whole section. |
| `--command "<text>"` | Edit instruction (free-form or one of the 5 presets). |
| `--from-file <yaml>` | Apply a batch of edits from `assets/edit_commands.yaml`. |
| `--json` | JSON output with new section content + word count delta. |

The 5 preset commands recognised by the runtime: `改短` `展开` `改锋利` `加例子` `去AI味`. Free-form anything else.

## `af draft-show <article_id>`

| Flag | Effect |
|---|---|
| `--json` | Full metadata. Includes `sections[].compliance_score`, `sections[].taboo_violations[]`, `image_placeholders[]`. |

## `af propose-images <article_id>`

LLM proposes image placements based on draft + hotspot's `source_references`.

| Flag | Effect |
|---|---|
| `--json` | Returns `{raw, placements_inserted}`. |
| `--max-images N` | Cap proposals at N. |

Side effects: rewrites `draft.md` with `[IMAGE: ...]` markers, writes `image_proposals.json`, syncs `metadata.json::image_placeholders[]`, appends `images_proposed` to events.jsonl.

## State written

- `~/.agentflow/drafts/<article_id>/skeleton.json` — stashed for re-fill.
- `~/.agentflow/drafts/<article_id>/metadata.json` — fill history, compliance scores, image placeholders.
- `~/.agentflow/drafts/<article_id>/draft.md` — assembled article.
- `~/.agentflow/memory/events.jsonl` — `article_created`, `fill_choices`, `section_edit`, `images_proposed`.
