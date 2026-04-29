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

[Unreleased]: https://github.com/witness1993x/agentflow-skills/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/witness1993x/agentflow-skills/releases/tag/v1.0.0
