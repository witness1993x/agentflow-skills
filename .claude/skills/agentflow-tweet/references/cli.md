# Tweet CLI reference

`af tweet-*` subcommands. All operate on `~/.agentflow/tweets/<tweet_id>/`.

## `af tweet-draft <source_id>`

Generate a tweet or thread from a hotspot or article.

| Flag | Effect |
|---|---|
| `--form single` / `--form thread` | Required (or comes from yaml). |
| `--from-article` | Treat `<source_id>` as `article_id` instead of `hotspot_id`. |
| `--angle N` | Pick angle index from the source's `suggested_angles[]`. |
| `--from-file <yaml>` | Apply `assets/thread_template.yaml` (max_tweets, tone, pin, etc). |
| `--max-tweets N` | Cap thread length. |
| `--tone <name>` | sarcastic / punchy / factual / explainer. |
| `--json` | Returns `{tweet_id, tweets: [...]}`. |

Returns a fresh `tweet_id` each call. Old drafts persist on disk.

## `af tweet-show <tweet_id>`

| Flag | Effect |
|---|---|
| `--json` | Full draft + char counts + image slots + source_refs. |

## `af tweet-edit <tweet_id>`

| Flag | Effect |
|---|---|
| `--index N` | Target tweet index. |
| `--command "<text>"` | Edit instruction (preset or free-form). |
| `--split N` | Split tweet at index N into two. |
| `--merge a,b[,c...]` | Merge listed indices into one. |
| `--reorder a,b,c,...` | Reorder tweets by indices (length must match). |
| `--json` | Output. |

Presets recognised: `改短`, `展开`, `改锋利`, `加例子`, `去AI味`. Free-form passes through.

## `af tweet-publish <tweet_id>`

| Flag | Effect |
|---|---|
| `--dry-run` | Validate creds + content; don't post. |
| `--json` | Returns `{status, published_url, thread_tweet_ids[], failure_reason?}`. |

`status`: `success` / `partial_success` / `failed`. Never auto-retries — Twitter's rate limit is aggressive and a duplicate thread risks a spam ban.

## `af tweet-rollback <tweet_id>`

| Flag | Effect |
|---|---|
| `--json` | Output. |

Calls Twitter delete API for each posted tweet. Best-effort; old tweets or account-transferred ones may refuse.

## `af tweet-list`

| Flag | Effect |
|---|---|
| `--limit N` | Default 20. |
| `--status <name>` | Filter by `draft` / `published` / `rolled_back`. |
| `--json` | Output. |

## Credentials

`.env` needs all four for real publishing:

```
TWITTER_CONSUMER_KEY
TWITTER_CONSUMER_SECRET
TWITTER_USER_ACCESS_TOKEN
TWITTER_USER_ACCESS_SECRET
TWITTER_HANDLE       # optional, prettier URLs
```

The read-only `TWITTER_BEARER_TOKEN` is **not enough**. User-context tokens come from the Twitter Developer app's "Keys and tokens" tab. Configure via `af onboard --section twitter`; never hand-edit `.env`.
