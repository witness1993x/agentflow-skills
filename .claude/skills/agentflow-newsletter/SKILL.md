---
name: agentflow-newsletter
description: |
  Turn an article draft (or scratch topic) into a newsletter email, test-send to yourself, and fan out to subscribers via Resend. The Telegram review daemon is optional and not used by this skill.

  TRIGGER: "/agentflow-newsletter", "af newsletter-*", "af notify", "newsletter", "email", "Resend", "邮件", "newsletter-correction".

  SKIP for: modifying the EmailPublisher internals (agent_d4/publishers/email.py); editing Resend audience configuration (in .env); modifying prompts/email_newsletter.md.
---

# agentflow-newsletter — ship the email version

Wraps `af newsletter-{draft,show,edit,preview-send,send,correction,list-show}`. Goal: deliver `<article_id>` (or scratch topic) as newsletter, with one hard stop before the irreversible blast.

- **Assets:** `assets/sections.yaml` — copy, fill, run `af newsletter-draft --from-file <path>`
- **References:** `references/cli.md` (full flags), `references/troubleshooting.md`

**Platform (v0.1):** Resend. Needs `RESEND_API_KEY`, `NEWSLETTER_FROM_EMAIL`, `NEWSLETTER_AUDIENCE_ID`. Mock mode works without keys. Emails **cannot be un-sent** — use `af newsletter-correction`.

## Input + setup

Expect `<article_id>` or `--from-scratch "<title>"`. If missing: ask.

```bash
cd <runtime_repo>/backend && source .venv/bin/activate
```

Prefix with `PYTHONPATH=.`.

## Step 1 — derive / create

```bash
PYTHONPATH=. af newsletter-draft <article_id> --json
PYTHONPATH=. af newsletter-draft --from-scratch "<title>" --json
PYTHONPATH=. af newsletter-draft <source> --from-file assets/sections.yaml --json
```

Capture `newsletter_id`. Tell user: "drafted `<id>` — subject `<S>` (`<N>` chars)".

## Step 2 — show

```bash
PYTHONPATH=. af newsletter-show <newsletter_id>
```

Read html_body / plain_text_body lengths, subject, preview_text. Print first ~20 lines of plain text.

## Step 3 — edit loop

Sections enum: `subject | preview_text | intro | body | closing`.

```bash
PYTHONPATH=. af newsletter-edit <newsletter_id> --section intro --command "<text>"
```

Loop until user says "good".

## Step 4 — pre-send overview (hard stop)

Skip only if user says "just send". Check subject length (≤ 45 en / ≤ 22 zh), preview text ≤ 90, images ≤ 2, `{unsubscribe_link}` present, audience configured, from-domain DKIM/SPF verified. Present as "Ready check"; pause: "Proceed to test-send? (yes / edit / cancel)".

## Step 5 — test-send to self

```bash
PYTHONPATH=. af newsletter-preview-send <newsletter_id> --to self --json
```

`MOCK_LLM=true` returns fake `resend_id`. Otherwise: "Test email sent. Confirm subject/preview/body/links/unsubscribe/reply-to. Reply 'looks good' or 'edit'."

## Step 6 — hard confirm

Print: `"You are about to send '<subject>' to audience <NEWSLETTER_AUDIENCE_ID>. This cannot be undone. To proceed, type: yes send"`. Only literal `yes send` proceeds. Optional dry-run: `af newsletter-send <id> --dry-run --json`.

## Step 7 — send + report

```bash
PYTHONPATH=. af newsletter-send <newsletter_id> --json
```

Success → "newsletter `<id>` sent — Resend id `<rid>`. Stats: https://resend.com/emails/<rid>". Failure → show `failure_reason`; don't auto-retry. Memory event `newsletter_sent` appended; metadata flips to `status: "sent"`.

## Correction flow

Triggered on "undo / send a correction". Does NOT retract original; sends follow-up. Steps: `af newsletter-edit` to fix → optional `--dry-run` → require `yes send` → `af newsletter-correction <id> --json`.

## `af notify`

`af notify "<msg>" [--event <type>]` — one-paragraph system email to `NEWSLETTER_REPLY_TO`. Use for completion signals on long-running jobs.

## NEVER

- Never skip Step 4 unless user says "just send".
- Never accept anything but literal `yes send` for Step 6.
- Never call `agent_d4.publishers.email` directly.
- Never email a real audience from mock mode (publisher short-circuits).

## MOCK_LLM

Fixtures + fake `resend_id`s, no network. Safe for rehearsal + CI.
