---
name: agentflow-tweet
description: |
  Generate, review, publish a Twitter single or thread from a hotspot or article. The Telegram review daemon is optional and not used by this skill.

  TRIGGER: "/agentflow-tweet", "af tweet-*", "twitter", "thread", "推文", "X 平台".

  SKIP for: editing Twitter API credentials (.env); modifying the D3 twitter adapter internals.
---

# agentflow-tweet — from idea to a posted thread

Wraps `af tweet-draft`, `tweet-show`, `tweet-edit`, `tweet-publish`, `tweet-rollback`, `tweet-list`. Goal: take `<hotspot_id>` or `<article_id>` and ship a single or thread, with explicit review before publish. Separate flow from long-form `af publish`.

- **Assets:** `assets/thread_template.yaml` — copy, fill, run `af tweet-draft --from-file <path>`
- **References:** `references/cli.md` (full flags), `references/troubleshooting.md`

## Input + setup

Expect `<hotspot_id>` OR `<article_id>`. If missing: ask.

```bash
cd <runtime_repo>/backend && source .venv/bin/activate
```

Prefix with `PYTHONPATH=.`.

## Step 0 — detect form

"single / 一条推" → `single`. "thread / 串 / 长推" → `thread` (default). Unclear → ask.

## Step 1 — draft

```bash
PYTHONPATH=. af tweet-draft <source_id> --form <form> [--from-article] [--angle N] --json
PYTHONPATH=. af tweet-draft <source_id> --from-file assets/thread_template.yaml --json
```

Capture `tweet_id`. Keep for later steps.

## Step 2 — review

```bash
PYTHONPATH=. af tweet-show <tweet_id> --json
```

Summarise: lineage, form + tweet count, chars/tweet (flag > 275 or < 180), hook, refs used (resolve `source_refs` against hotspot's `source_references[]`), hallucination flag, image slots (warn > 4/tweet). End: "Looks good? (yes / edit / regen)".

## Step 3 — edit loop

`改短 tweet N` / `去AI味 tweet 0` → `af tweet-edit <tid> --index N --command "<cmd>"`. `把第 2 条拆成两条` → `--split 1`. `合并第 2 和第 3 条` → `--merge 1,2`. `重排 ...` → `--reorder 0,3,1,2,4`. `regen` → re-run Step 1 (new tweet_id). After each edit, show only changed tweets with old/new char counts. See `references/cli.md` for full flag list.

## Step 4 — pre-publish hard confirm

Print summary block (form, total chars, images, platform, expected rate-limit, refs alignment). Then literally: `"type 'yes send' to publish, or 'cancel'"`. Anything else cancels.

## Step 5 — publish

Optional dry-run: `af tweet-publish <tid> --dry-run --json`. Real:

```bash
PYTHONPATH=. af tweet-publish <tweet_id> --json
```

Parse `status`. `success` → show `published_url`. `partial_success` → don't auto-retry. `failed` → see `references/troubleshooting.md`.

## Step 6 — report

`Thread tw_... posted: <first_url>; thread_tweet_ids: [...]`.

## Rollback

Triggered on "delete / undo / 删掉 / 撤下".

```bash
PYTHONPATH=. af tweet-rollback <tweet_id> --json
```

Confirm first: "This deletes N tweets. Proceed? (yes / cancel)". Twitter delete is best-effort.

## State

- `~/.agentflow/tweets/<tweet_id>/` — metadata + tweets.json + tweets.md + images/
- `publish_history.jsonl` — one row per publish or rollback
- `memory/events.jsonl` — `tweet_draft_created` / `tweet_edited` / `tweet_published` / `tweet_rolled_back`

## NEVER

- Never publish without "type 'yes send'" hard confirm.
- Never silently drop tweets > 280 chars.
- Never auto-retry failed publish (rate-limit + spam ban risk).
- Never add `🧵` or `1/N` prefixes — Twitter natively numbers threads.

## MOCK_LLM

`MOCK_LLM=true` short-circuits LLM + tweepy. Fake URLs `https://twitter.com/<handle>/status/mock_...`.
