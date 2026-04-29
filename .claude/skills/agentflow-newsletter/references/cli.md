# Newsletter CLI reference

`af newsletter-*` subcommands. All operate on `~/.agentflow/newsletters/<newsletter_id>/`.

## `af newsletter-draft <source>`

Generate an email from an article (or a scratch title).

| Flag | Effect |
|---|---|
| `<source>` | Either `<article_id>` or omitted with `--from-scratch`. |
| `--from-scratch "<title>"` | Generate from scratch using the title only as seed. |
| `--from-file <yaml>` | Apply `assets/sections.yaml` (subject_template, intro, body sections, etc). |
| `--audience-id <id>` | Override `NEWSLETTER_AUDIENCE_ID` for this draft. |
| `--json` | Returns `{newsletter_id, subject, preview_text, html_body_length, plain_body_length}`. |

## `af newsletter-show <newsletter_id>`

| Flag | Effect |
|---|---|
| (no flags) | Prints subject + preview + plain body to stdout. |
| `--json` | Full draft (html + plain + metadata). |

## `af newsletter-edit <newsletter_id>`

| Flag | Effect |
|---|---|
| `--section <enum>` | Required: `subject` / `preview_text` / `intro` / `body` / `closing`. |
| `--command "<text>"` | Free-form edit instruction. |
| `--json` | Output. |

Idempotent — re-editing a section just replaces it. Old versions stay in `metadata.json::edit_history[]`.

## `af newsletter-preview-send <newsletter_id>`

Test-send to a single address before the audience blast.

| Flag | Effect |
|---|---|
| `--to self` | Send to `NEWSLETTER_REPLY_TO`. |
| `--to <email>` | Send to an arbitrary address. |
| `--json` | Returns `{resend_id, to}`. |

`MOCK_LLM=true` short-circuits — fake `resend_id`, no email leaves the machine.

## `af newsletter-send <newsletter_id>`

The irreversible blast.

| Flag | Effect |
|---|---|
| `--dry-run` | Validate creds + audience + content; don't send. |
| `--audience-id <id>` | Override default. |
| `--json` | Returns `{platform_post_id, status, audience_size}`. |

## `af newsletter-correction <newsletter_id>`

Send a follow-up correction to the same audience. Use after the original has gone out and a fix is needed.

| Flag | Effect |
|---|---|
| `--dry-run` | Validate without sending. |
| `--json` | Returns `{platform_post_id, status}`. |

## `af newsletter-list-show`

| Flag | Effect |
|---|---|
| `--limit N` | Default 20. |
| `--status <name>` | `draft` / `sent` / `correction_sent`. |
| `--json` | Output. |

## `af notify`

System notification email — not a newsletter.

```bash
af notify "<message>" [--event hotspots_done|publish_failed|...]
```

Sends one paragraph to `NEWSLETTER_REPLY_TO`. Useful for long-running job completion. `MOCK_LLM=true` mocks silently.

## Credentials

`.env` requires:

```
RESEND_API_KEY
NEWSLETTER_FROM_EMAIL          # must be on a verified domain
NEWSLETTER_REPLY_TO            # operator's inbox; receives test-sends + notify
NEWSLETTER_AUDIENCE_ID         # Resend audience id
```

Configure via `af onboard --section resend`. Domain DKIM/SPF must be verified in Resend dashboard before sending to real subscribers — unverified mail lands in spam.
