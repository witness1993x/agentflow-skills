# `af learn-style` CLI reference

Single command, multiple modes. All write to `~/.agentflow/style_profile.yaml` and append to `~/.agentflow/style_corpus/`.

## Basic forms

| Form | Effect |
|---|---|
| `af learn-style --file <path>` | Ingest one file (markdown / docx / txt). Repeat the flag for multiple files. |
| `af learn-style --url <url>` | Fetch + ingest one URL. Repeatable. |
| `af learn-style --dir <path>` | Ingest every supported file in a directory (non-recursive). |
| `af learn-style --show` | Print the current profile YAML to stdout. |
| `af learn-style --recompute` | Re-aggregate the profile from the existing `style_corpus/` without re-ingesting. |
| `af learn-style --from-published` | Ingest every draft marked as `published` in `~/.agentflow/drafts/*/metadata.json`. |
| `af learn-style --from-file <yaml>` | Apply a `style_tuning.yaml` to bias the harvest (paragraph targets, banned/must phrases, persona). |

Mix `--file`, `--url`, and `--from-file` in one call.

## Related (handle ingestion)

| Form | Effect |
|---|---|
| `af learn-from-handle <handle> --profile <id>` | Pull recent posts from a known platform handle (Twitter, Substack, Medium). Uses canonical runtime adapters. |

## Output flags

| Flag | Effect |
|---|---|
| `--json` | Machine-readable summary on stdout. |
| `--quiet` | Suppress per-article progress logs (default: on stderr). |

## What the profile contains

After ingest the YAML at `~/.agentflow/style_profile.yaml` includes:

- `voice_principles[]` — top 3 voice rules harvested across the corpus.
- `taboos[]` — banned phrases / patterns the user avoids.
- `paragraph_preferences` — `{ min_chars, max_chars, target_chars }` per language.
- `tone_tags[]` — short adjectives describing the prose.
- `corpus_size` — int, number of ingested sources.
- `last_updated` — ISO timestamp.

The harvest is LLM-driven. To bias it (e.g. "always punchy openings", "shorter paragraphs in zh-Hans"), pass `--from-file assets/style_tuning.yaml`.
