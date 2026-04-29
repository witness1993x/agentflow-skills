# `agentflow-write` troubleshooting

## "Hotspot not found"

The id may live in `~/.agentflow/search_results/` rather than `~/.agentflow/hotspots/`. Both are checked, but if neither has it the user typo'd. Check:

```bash
ls ~/.agentflow/hotspots/ ~/.agentflow/search_results/ | grep <hotspot_id>
```

If empty, re-run `af hotspots` or `af search`.

## "IndexError: section out of range" / paragraph out of range

The `--section N` or `--paragraph M` is past the actual count. Run:

```bash
af draft-show <article_id> --json | jq '.sections | length'
```

to see real bounds.

## "compliance_score reports 0 across the board"

The compliance scorer needs `topic_profiles.yaml::publisher_account.do/dont/product_facts` populated. Likely your profile is incomplete. See `../agentflow/references/troubleshooting.md` ("missing topic profile fields").

## "all sections come back at 140-180 chars when target was 150"

Known limitation of Kimi / Moonshot v1 LLM. The mechanical paragraph splitter in D3 (`agent_d3/adapters`) handles platform-specific limits at publish time:

- Ghost: max 70 words per paragraph.
- LinkedIn: max 40 words per paragraph.
- Medium: no enforced max.

So compliance < 1.00 on raw D2 is usually a signal, not a blocker. Still investigate sections < 0.70.

## "af fill returns same draft with same indices"

Cached. Bump indices (try a different `--title` / `--opening` / `--closing` combo) or pass `--force-regen`.

## "af edit silently succeeds but draft.md unchanged"

Check the section index — you may have edited a section but a different section is being shown. `af draft-show --json` to see the actual content per section.

## "image proposal placed [IMAGE: ...] in the wrong place"

Re-run `af propose-images --json` (it overwrites). Or manually delete the bad placeholder from `draft.md` and re-`Read` it.

## "draft missing image_placeholders after propose-images"

The metadata-resync step failed. Run `af draft-show --json` — if `image_placeholders[]` is empty but `draft.md` has `[IMAGE: ...]` lines, manually re-run `af propose-images <article_id>` (idempotent regeneration).

## Generic non-zero exit

```bash
tail -n 20 ~/.agentflow/logs/agentflow.log
```
