# `agentflow-tweet` troubleshooting

## "missing TWITTER_CONSUMER_KEY" / similar 4xx

You set `TWITTER_BEARER_TOKEN` but not the user-context tokens. Bearer is read-only. Run:

```bash
af onboard --section twitter
```

Provide all four: `TWITTER_CONSUMER_KEY`, `TWITTER_CONSUMER_SECRET`, `TWITTER_USER_ACCESS_TOKEN`, `TWITTER_USER_ACCESS_SECRET`. Generate the user-context tokens from your Twitter Developer app's "Keys and tokens" tab.

## "tweepy not installed"

```bash
cd <runtime_repo>/backend
source .venv/bin/activate
pip install tweepy
```

Then re-run.

## "tweet exceeds 280 chars"

The runtime should split at `--form thread`. If it doesn't, the LLM probably ignored the cap. Re-run with explicit `--max-tweets` to push thread structure, or `af tweet-edit --index N --command "改短"` to shrink offenders.

## "rate-limited (429)"

Twitter's free tier rate-limits aggressively. Wait the indicated window (often 15 minutes), then re-run. Don't auto-retry from the skill — duplicate thread post = potential spam ban.

## "partial_success: posted 3 of 5"

Tweets 0-2 are live. Tweets 3-4 didn't post. The CLI returns the posted ids; surface them to the user. Manual options:
1. Compose tweets 3-4 manually as replies to the last posted id.
2. Roll back the partial thread (`af tweet-rollback`) and start over after rate-limit clears.

Don't run `af tweet-publish` again on the same `tweet_id` — it tries to post the entire thread fresh, creating duplicates.

## "tweet-rollback partial: 2 of 5 deleted"

Some tweets were deleted, some weren't. Common causes: tweet > 30 days old (Twitter restricts), tweet was already manually deleted (404 = success-equivalent), token lacks `tweet.write` scope (re-issue with full scopes).

## "image upload failed"

Twitter rejects > 5MB images. Check `~/.agentflow/tweets/<tid>/images/` sizes. Pre-resize before publishing, or `af tweet-edit <tid> --index N --command "drop image"`.

## "regen produces same draft"

LLM caching. Pass `--ignore-cache` if available, or modify the source angle (`--angle N+1`) to force a fresh prompt context.

## "tweet 0 hook claim flagged as not in refs"

The skill warned because the LLM made up a fact. Don't paper over — either:
1. Edit the hook to remove the unverified claim: `af tweet-edit <tid> --index 0 --command "rephrase without the X claim"`, or
2. Verify it externally and add the source URL explicitly.

## Generic non-zero exit

```bash
tail -n 20 ~/.agentflow/logs/agentflow.log
```
