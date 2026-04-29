# Gate B — draft acceptance

Owner of Gate B. The Telegram review daemon is optional and not required; in Mode A the operator decides via chat reply. See `../agentflow/references/daemon-when-needed.md`.

## When Gate B fires

After `af fill` (manual path) or `af write --auto-pick` (auto path) completes, the article is in `STATE_DRAFT_FILLED`. Gate B asks: ship as-is, edit, rewrite, or reject?

## Decision matrix

Inspect `metadata.json::sections[].compliance_score` and `taboo_violations[]`:

| Signal | Action |
|---|---|
| Avg compliance ≥ 0.85, all sections clean | Approve → image-gate (`STATE_DRAFT_ACCEPTED`) |
| 1 section 0.5–0.85 with 1–2 violations | `af edit <id> --section N --command "<fix>"` |
| 1 section < 0.5 (taboo pattern hit / multi-paragraph too long) | `af fill <id>` rewrite (max 2 rounds → `STATE_DRAFTING_LOCKED_HUMAN`) |
| 2+ sections < 0.5 | Reject + reselect hotspot |
| `taboo_pattern: '首先...其次...最后'` | Edit, rewrite as declarative prose |
| LLM fabricated key fact (date / customer count / vuln) | Edit + add source URL OR remove the fabricated segment |

## Fact-check protocol (LLM hallucination)

The LLM may fabricate **outside** `product_facts`. Skill should remind user to check:

- Quantitative or temporal claims ("数百万笔交易" / "Q1 遭遇 X 漏洞") → must be human-verified.
- Cited projects / companies / people → grep `~/.agentflow/style_corpus/` or quick Google.
- Sales-y closing phrases ("加入我们 / 联系我们") → edit out (content_tone anti-pattern).

Specific numbers / dates / company names not declared in `product_facts` are **potential hallucinations**. Surface them.

## Ceiling on rewrite rounds

The CLI tracks rewrite count per article. After round 2:

```
STATE_DRAFTING_LOCKED_HUMAN
```

Means the LLM produced 3 unacceptable drafts in a row; human escalation required (manual edit pass or hotspot abandonment). Don't loop indefinitely.

## Gate B in TG-review mode (Mode B / C)

When the optional Telegram-review daemon is running, Gate B card lands on Telegram with buttons: `B:approve`, `B:edit`, `B:rewrite`, `B:reject`. The harness still drives `af write` / `af fill`; the optional daemon only mediates the approval signal.

In Mode A, just run `af edit` / `af fill` / `af image-gate` directly; the optional daemon stays unused.
