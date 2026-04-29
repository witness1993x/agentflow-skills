# `agentflow-newsletter` troubleshooting

## "missing RESEND_API_KEY"

`af onboard --section resend`. Get the key from https://resend.com/api-keys. Never hand-edit `.env`.

## "domain not verified"

Resend rejects sends from unverified domains. Add DNS records (DKIM CNAMEs + SPF TXT) per the Resend dashboard, wait for verification, retry. The send will silently land in spam if you skip this; the publisher does NOT block sends pre-emptively.

## "audience_id not found"

`NEWSLETTER_AUDIENCE_ID` doesn't match an audience in your Resend account, or the API key lacks permission for that audience. Check https://resend.com/audiences. If you operate multiple audiences, override per-send:

```bash
af newsletter-send <id> --audience-id aud_xxx --json
```

## "subject too long (Gmail truncates)"

Run `af newsletter-edit <id> --section subject --command "shorten to 8 words"`. Mobile inbox preview cuts at ~45 chars (English) or ~22 chars (Chinese double-byte).

## "preview_text empty"

The runtime warns but doesn't block. Fix:

```bash
af newsletter-edit <id> --section preview_text --command "write a 70-char teaser"
```

Some clients (Apple Mail, Gmail) fall back to first body sentence — usually OK, but worth setting explicitly.

## "{unsubscribe_link} missing in plain text body"

The publisher injects a `mailto:` fallback if missing, but a real Audience unsub link is better for compliance. Run:

```bash
af newsletter-edit <id> --section closing --command "add the {unsubscribe_link} placeholder explicitly"
```

## "MOCK_LLM=true newsletter-send returned a real-looking resend_id"

Always check the prefix. Mock mode resend_ids start with `mock_`. Real ones are short alphanumerics. If you see a real-looking one in mock mode, the mock injection failed — treat it as a bug, don't trust the send happened.

## "test-send arrived but body looks broken in Gmail"

Gmail strips some CSS aggressively. The newsletter publisher uses safe inline styles, but custom edits via `newsletter-edit` may insert client-incompatible HTML. Try a plain-text-style edit if rendering keeps breaking.

## "newsletter-correction sent but recipients don't see it"

Check Resend dashboard — corrections are sent as a fresh email, not threaded. If subscribers filter by sender, they'll see it; if they filtered by subject, they may miss it. Use a clearly distinct subject line for corrections.

## "correction sent to wrong audience"

`af newsletter-correction` defaults to the same `audience_id` as the original. To target a subset, override with `--audience-id`.

## Generic non-zero exit

```bash
tail -n 20 ~/.agentflow/logs/agentflow.log
```
