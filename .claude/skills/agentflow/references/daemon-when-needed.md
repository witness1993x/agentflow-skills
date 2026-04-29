# Do I need the optional review daemon?

**TL;DR:** No, by default. The daemon is opt-in and only adds phone-based Telegram approval. If you read all the skill prose and saw "ensure daemon running" — that's a Mode B / Mode C concern (Telegram-review-only), not a hard prerequisite.

Pick one of three modes within 60 seconds. Most users want **Mode A**.

## Mode A — harness-only (default, recommended)

**Who:** Operator works inside Claude Code or Cursor. Drives `af` directly from the chat session. Wants approve/reject as a chat reply, not a phone notification.

**Daemon required?** No — daemon is optional in Mode A.

**Setup:**

```bash
af bootstrap --mock --first-run     # mock demo
# or
af bootstrap --next-step --json     # iterative real-key
```

Note the absence of `--start-daemon` (the optional flag). `af doctor` probe 13 (heartbeat stale) is **expected** in Mode A and not blocking.

**How approvals work:** at each gate, the harness shows the decision options inline; the operator types `B:approve`, `I:cover_only`, `PD:dispatch` etc. as a normal chat reply. No phone, no Telegram, the optional daemon is not started.

**When to switch up:** if you find yourself wanting to approve articles from your phone while away from the desk → upgrade to Mode B. Otherwise, stay here.

## Mode B — Telegram-review (phone-based approval)

**Who:** Operator wants to approve/reject from a phone (often after stepping away from the laptop). Cards arrive on Telegram with inline buttons.

**Daemon required?** Yes — Mode B is the **only** mode where the optional Telegram-review daemon becomes mandatory.

**Setup:**

```bash
af bootstrap --next-step --json     # ensure TG section is configured
af review-daemon                    # optional component; foreground / systemd / launchd
# in Telegram, send /start to your bot to capture chat_id
```

For systemd:

```bash
sudo systemctl enable --now agentflow-review.service
```

**How approvals work:** when the harness drives `af` past a gate, the optional Telegram-review daemon detects the state transition, posts a Gate A/B/C/D card to your Telegram chat with `Approve/Reject` buttons, and waits for your tap. Tap → state advances → optional Telegram-review daemon writes back into `~/.agentflow/`.

**Health check:** `af doctor --json` probe 13 (heartbeat) must report fresh (< 60s). If stale → optional Telegram-review daemon died → restart it.

## Mode C — hybrid (harness + optional daemon mirror)

**Who:** Power user. Wants the harness to drive the pipeline (so the IDE always knows current state) AND wants gates mirrored to Telegram (so they can approve from phone if convenient).

**Daemon required?** Yes — same setup as Mode B (optional Telegram-review daemon required only in Mode B/C).

**How approvals work:** any gate can be resolved from EITHER the chat reply OR the Telegram tap. First one wins; the other side reflects the resolution. Useful when you sometimes work at the desk and sometimes from your phone.

## Decision flow

```
Do you want to approve articles from your phone?
├── No  → Mode A (optional daemon stays unused). Stop reading.
└── Yes → Are you also driving from a desktop IDE?
           ├── No  → Mode B (optional daemon becomes mandatory).
           └── Yes → Mode C (harness + optional daemon).
```

## Common pitfall

`af doctor` probe 13 reports "stale heartbeat" in Mode A. That's normal — the optional daemon was never started, so there's nothing to write the heartbeat. Don't "fix" it by force-starting an optional daemon you don't need. Just ignore the warning, or filter it: `af doctor --json | jq '.probes | map(select(.id != 13))'`.
