# Worked examples

Three end-to-end transcripts you can mimic.

## Example 1 — fresh mock demo (Mode A, harness-only)

```bash
# Operator just installed the runtime; wants to see the whole pipeline.
af bootstrap --mock --first-run     # daemon optional — harness handles approvals

# Skill walks through:
af topic-profile init --profile demo --from-file assets/topic_profile.yaml
MOCK_LLM=true af hotspots --profile demo --json
# operator: "pick 2, angle 0"
MOCK_LLM=true af write hs_mock_002 --angle 0 --auto-pick --json
# returns article_id ar_20260429_xxx
MOCK_LLM=true af draft-show ar_20260429_xxx --json
# Gate B: clean → approve in chat
MOCK_LLM=true af image-gate ar_20260429_xxx --mode none --json
# Gate C skipped → Gate D
MOCK_LLM=true af preview ar_20260429_xxx --platforms medium --skip-images --json
MOCK_LLM=true af publish ar_20260429_xxx --platforms medium --force-strip-images --json
# Medium manual: paste package then close loop
af review-publish-mark ar_20260429_xxx https://medium.com/@user/post-id
```

Total time with `MOCK_LLM=true`: ~2 minutes. Daemon is optional and not invoked here.

## Example 2 — real-key onboarding (Mode A)

```bash
# Iterative bootstrap, one step at a time.
af bootstrap --next-step --json
# returns: {"current_state": "missing_telegram_creds", "next_command": "af onboard --section telegram"}

af onboard --section telegram   # operator types token at terminal prompt
af bootstrap --next-step --json
# returns: {"current_state": "missing_llm_creds", "next_command": "af onboard --section llm"}

af onboard --section llm
af bootstrap --next-step --json
# ... continue until current_state == "ready"

af doctor --json   # confirm 13 probes (probe 13 stale is OK in Mode A)
```

## Example 3 — daily writing loop (already onboarded, Mode A)

```bash
af doctor                            # quick sanity
af topic-profile show --profile witness --json | jq .completeness
# 1.0 → ready

af hotspots --profile witness --filter "MCP|agent" --json
# operator picks: hotspot 3, angle 1

af write hs_20260429_003 --angle 1 --auto-pick --json
# article_id ar_20260429_001

af draft-show ar_20260429_001 --json
# Gate B: section 2 compliance 0.55 → edit
af edit ar_20260429_001 --section 2 --command "改短"
af draft-show ar_20260429_001 --json   # recheck

af propose-images ar_20260429_001 --json   # optional Step 3.5
af image-gate ar_20260429_001 --mode cover-only --json

af preview ar_20260429_001 --platforms medium --json
# Gate D: review preview, then dispatch
af publish ar_20260429_001 --platforms medium --json
af review-publish-mark ar_20260429_001 <medium_url>
```

Mode B operator does the same but TG cards arrive on phone for each Gate; the optional Telegram-review-only daemon mediates between `af` runs.
