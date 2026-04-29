# `af hotspots` / `af hotspot-show` CLI reference

D1 topic discovery. Reads sources from `~/.agentflow/sources.yaml` (or `--from-file`), produces clusters, scores them, writes today's daily file.

## `af hotspots`

| Flag | Effect |
|---|---|
| `--profile <id>` | Use this topic profile's `keyword_groups` for scoring. Required when multiple profiles exist. |
| `--filter "<regex>"` | Case-insensitive regex matched against `topic_one_liner`, suggested angle titles, and source reference snippets. |
| `--from-file <yaml>` | Override scan sources (Twitter handles, RSS feeds, HN topic filters) for this run only. See `assets/sources.yaml`. |
| `--json` | Pure JSON to stdout; logs to stderr. |
| `--top-k <N>` | Override `top_k` from sources yaml. |
| `--scan-window-hours <N>` | Override scan window. |
| `--force-rescan` | Skip the same-day cache; always rescan. |

## Output schema (selected fields)

```json
{
  "hotspots": [
    {
      "hotspot_id": "hs_20260429_001",
      "topic_one_liner": "...",
      "freshness_score": 0.0,
      "recommended_series": "...",
      "overlooked_angles": ["..."],
      "source_references": [
        {"source": "twitter", "author": "...", "url": "...", "text_snippet": "...", "engagement": 0}
      ],
      "suggested_angles": [
        {"title": "...", "depth": 0, "difficulty": 0, "fit_explanation": "..."}
      ]
    }
  ],
  "filter": {
    "pattern": "...",
    "matched": 0,
    "total": 0,
    "boundary": {"level": "too_narrow|narrow|balanced|broad"},
    "filtered_out_preview": []
  }
}
```

## `af hotspot-show`

| Flag | Effect |
|---|---|
| `<hotspot_id>` | Required positional. |
| `--json` | Pure JSON output. |

Returns a single hotspot with full `suggested_angles[]` (typically 3–5 angles).

## Resolution sources

Hotspot ids resolve from both:

- `~/.agentflow/hotspots/<YYYY-MM-DD>.json` — daily D1 batches.
- `~/.agentflow/search_results/*.json` — archived search-result bundles (custom searches, manual filters).

`af hotspot-show` checks both.

## State side-effects

- Writes `~/.agentflow/hotspots/<YYYY-MM-DD>.json` (overwrites same-day).
- Archives search-result bundles to `~/.agentflow/search_results/`.
- Appends `hotspots_scanned` event to `memory/events.jsonl`.
- May surface `constraint_suggestions/*.json` if profile config drift detected — review via `af topic-profile review/apply`.
