# Changelog

All notable changes to the **agentflow skill distribution** are recorded here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This repo follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html),
but versions track **skill API compatibility against the canonical runtime**
(`af <subcommand>` interface) — not line-for-line code parity. A version bump
happens only when the SKILL.md surface that Claude Code / Cursor sees changes
in a way an integrator must notice.

The version number lives in [`VERSION`](VERSION). The directory name
`agentflow-<version>/` is expected to match `VERSION` and the latest git tag
`v<version>`.

## [Unreleased]

- _no changes yet_

## [1.0.3] — 2026-04-30

Adds the agent-driven install reference. A Claude Code / Cursor / OpenClaw
harness loading this skill can now drive a fresh install end-to-end by
looping on `af bootstrap --next-step --json` and reacting to the per-state
table.

### Added

- **`agentflow/references/install.md`** — single source of truth on what
  the agent should do for each `current_state` returned by
  `af bootstrap --next-step --json`. Covers Mode A (harness-only) and
  Mode B/C (Telegram-review) paths, including the auto-resolution rule
  for picking a mode from credential presence.

### Changed

- **`agentflow/SKILL.md` "Default entry" block** rewritten as a tight
  reference to the loop pattern + `references/install.md`. State table
  moved out of `SKILL.md` (kept lean at ≤100 lines) into the new
  reference file so updates to the install path don't bloat every
  harness session.

### Pairs with sibling repo

- `witness1993x/agentflow-article-publishing v1.0.5` fixes the matching
  bootstrap-detector bug (Mode A operators were being forced into Mode
  B/C / daemon-required). Same release window; install the matching
  pair.

## [1.0.2] — 2026-04-29

A pure-skill-surface restructure release: every public skill now follows
the standard layout (`SKILL.md` + `references/` + `assets/`), and every
mention of the review daemon is explicitly marked optional.

### Changed

- **All 7 skills restructured** to standard layout. Each `SKILL.md` is
  now a slim trigger + orchestration file (≤100 lines); long-form CLI
  reference, gate definitions, and troubleshooting moved into per-skill
  `references/`. Affects `agentflow`, `agentflow-hotspots`,
  `agentflow-write`, `agentflow-publish`, `agentflow-style`,
  `agentflow-tweet`, `agentflow-newsletter`.
- **Daemon explicitly marked OPTIONAL** across all skills. The review
  daemon is required only for **Mode B (phone-based Telegram approval)**
  or **Mode C (hybrid)**. **Mode A** — harness-only, where Claude Code /
  Cursor drives `af` directly — needs no daemon. The `af bootstrap
  --start-daemon` flag is documented as opt-in, not part of the canonical
  first-run example.

### Added

- **`assets/<topic>.yaml`** for each skill — CLI-consumable templates
  the operator copies → fills → passes via the matching runtime
  `af … --from-file` flag (added in `agentflow-article-publishing v1.0.4`).
  - `agentflow/assets/topic_profile.yaml` → `af topic-profile init --from-file`
  - `agentflow-style/assets/style_tuning.yaml` → `af learn-style --from-file`
  - `agentflow-hotspots/assets/sources.yaml` → `af hotspots --from-file`
  - `agentflow-write/assets/edit_commands.yaml` → `af edit --from-file`
  - `agentflow-publish/assets/platform_overrides.yaml` → `af preview --from-file`
  - `agentflow-tweet/assets/thread_template.yaml` → `af tweet-draft --from-file`
  - `agentflow-newsletter/assets/sections.yaml` → `af newsletter-draft --from-file`
- **`agentflow/references/daemon-when-needed.md`** — single page
  explaining Modes A / B / C so an operator knows in <1 minute whether
  they need to run a daemon at all.
- **Per-skill `references/` content** — `cli.md`, `gates.md`,
  `troubleshooting.md`, `examples.md` (where applicable). Loaded
  on-demand by Claude Code / Cursor when the skill is in scope.

### Pairs with sibling repo

- **`witness1993x/agentflow-article-publishing v1.0.4`** — secrets
  relocate to `~/.agentflow/secrets/`, deploy-bundle leak plugged,
  `--from-file` flag on 6 CLI commands consuming this repo's `assets/`
  YAMLs, +21 TG operator-completeness commands. The two releases ship
  together.

## [1.0.1] — 2026-04-29

### Changed

- Make the top-level `agentflow` skill default to first deployment /
  onboarding continuation instead of the daily writing loop when the user's
  intent is ambiguous.
- Point first-run guidance at `af bootstrap --next-step --json`,
  `af bootstrap --mock --first-run --start-daemon`, `af onboard`,
  `af topic-profile`, and `af doctor` instead of manual `.env` edits or
  direct symlink setup.
- Update publish guidance so the primary image path is
  `af image-gate <article_id> --mode ...`, with `--mode none` explicitly
  routing to Gate D.

### Fixed

- Reduce stale `image-resolve` emphasis in writing/publishing skill text now
  that image generation and Gate C/D routing are owned by `af image-gate`.

## [1.0.0] — 2026-04-28

First public skill distribution. Pure prompt artifacts (~90 KB / 11 files);
the `af` CLI runtime is shipped separately by
[`agentflow-article-publishing`](https://github.com/witness1993x/agentflow-article-publishing).

### Added

- `.claude/skills/agentflow/SKILL.md` — top-level workflow guide (profile →
  hotspots → write → publish).
- `.claude/skills/agentflow-style/SKILL.md` — wraps `af learn-style` /
  `af learn-from-handle` for voice profile bootstrap and refresh.
- `.claude/skills/agentflow-hotspots/SKILL.md` — wraps `af hotspots` /
  `af hotspot-show` for daily topic discovery (Gate A).
- `.claude/skills/agentflow-write/SKILL.md` — wraps `af write` / `af fill` /
  `af edit` / `af image-resolve` for the skeleton-fill-edit loop (Gate B).
- `.claude/skills/agentflow-publish/SKILL.md` — wraps `af preview` /
  `af publish` / `af medium-package` / `af review-publish-mark` (Gate C/D
  + dispatch + Medium browser-ops fallback).
- `.claude/skills/agentflow-tweet/SKILL.md` — Twitter/X single + thread.
- `.claude/skills/agentflow-newsletter/SKILL.md` — Resend email digest +
  preview-send / send / correction.
- `.claude/skills/README.md` — skill index, install hints (symlink / copy /
  cwd auto-register), shared rules.
- Top-level `README.md` declaring the repo as **skills-only** and pointing
  at the canonical runtime repo for the actual `af` CLI implementation.
- `LICENSE`.

### Compatibility

- Targets the `af` CLI surface as documented in
  `agentflow-article-publishing` at the time of release.
- Claude Code and Cursor are both supported invocation hosts.

[Unreleased]: https://github.com/witness1993x/agentflow-skills/compare/v1.0.2...HEAD
[1.0.2]: https://github.com/witness1993x/agentflow-skills/compare/v1.0.1...v1.0.2
[1.0.1]: https://github.com/witness1993x/agentflow-skills/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/witness1993x/agentflow-skills/releases/tag/v1.0.0
