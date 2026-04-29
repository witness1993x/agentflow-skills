# `agentflow-style` troubleshooting

## "no readable content from <path>"

The file is empty, has odd encoding, or pandoc/mammoth failed on a docx. Convert to UTF-8 markdown manually first.

## "URL fetch failed: 403 / 401"

Some platforms (Medium, paywalled Substack) reject scrapers. Save the article as markdown locally and pass `--file`.

## "LLM analysis returned empty"

Usually a transient LLM API error. Retry once. If persistent, check `~/.agentflow/logs/agentflow.log` for `agent_d0` errors. With `MOCK_LLM=true`, this should never fire — if it does, your fixtures are missing.

## "profile shows no taboos / one voice_principle"

Corpus too small. Ingest at least 3–5 articles before expecting useful aggregation. `af learn-style --recompute` after adding more sources will re-run the aggregation pass without re-fetching.

## "paragraph_preferences disagrees with my style_tuning.yaml"

The yaml is applied as a **bias**, not an override. The harvested median wins if it's clearly inconsistent with the bias; this is intentional so `learn-style` reflects what the user actually writes vs. what they aspire to. Edit the corpus if you want stronger control.

## "wrong language detected"

Profile's `output_language` is read from `topic_profiles.yaml`, not auto-detected here. Fix via `af topic-profile update --profile <id> --from-file <patch.yaml>` with the right `output_language`.

## Generic non-zero exit

```bash
tail -n 20 ~/.agentflow/logs/agentflow.log
```

Show output to user; don't auto-retry — the user usually needs to adjust paths or remove a problematic source.
