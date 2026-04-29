# `agentflow-publish` troubleshooting

## "image-gate fails with Atlas/preflight error"

```bash
af doctor       # check probe 7 (Atlas) and 8 (Pillow)
```

If Atlas is down or Pillow can't open the cover template, offer `--mode none` (with user consent — they're publishing without generated images).

## "preview rejects with no platform versions found"

You skipped Step 2 image-gate, OR the previous `image-gate` run failed silently. Run `af image-gate <id> --mode <choice>` first.

## "publish rejects: image placeholder unresolved"

You ran `image-gate --mode none` but forgot `--force-strip-images` on `af publish`. Add the flag.

## "Medium publish always returns status=manual"

Expected. The Medium API was deprecated; AgentFlow only generates a paste package. Use `af review-publish-mark` to record the URL after manual paste.

## "Ghost publish 401 / 403"

Bad `GHOST_ADMIN_API_KEY`. Run `af onboard --section ghost` to re-enter; never hand-edit `.env`. Note Ghost Admin keys have a `<id>:<secret>` format — both halves required.

## "Ghost publish OK but post not visible"

`GHOST_STATUS=draft` was set in env (often a leftover from a recent rollback). Check `af prefs-show --json::publish.ghost_status_override`. Either accept (draft is the desired test mode) or unset:

```bash
unset GHOST_STATUS    # current shell only
af prefs-clear --key publish.ghost_status_override
```

## "LinkedIn publish 401"

`LINKEDIN_ACCESS_TOKEN` expired (LinkedIn tokens are short-lived, ~60 days). Refresh via the LinkedIn developer portal, then `af onboard --section linkedin`.

## "publish-rollback says no successful <platform> record found"

Article was never successfully published to that platform. Check `af memory-tail | grep <article_id>` to see what actually shipped.

## "publish-rollback says matching record has no platform_post_id"

Pre-fix history record (publish happened before the runtime started recording `platform_post_id`). Get the post id from Ghost admin URL (`/ghost/#/editor/post/<id>`) or `/tmp/` json dumps, then:

```bash
af publish-rollback <article_id> --post-id <platform_post_id> --json
```

## "Ghost DELETE 404"

Post is already gone (manually deleted via Ghost admin). Skill should treat this as success-equivalent: `publish_rolled_back` event still gets written, `metadata.json` cleaned up. If the CLI exited non-zero, run with `--ignore-404` if available, or manually edit metadata.

## "preferences forced wrong default platforms"

Inspect:

```bash
af prefs-show --json | jq .preview
```

Override per-run:

```bash
af preview <id> --platforms medium --ignore-prefs --json
af publish <id> --platforms medium --ignore-prefs --json
```

Or clear: `af prefs-clear --key preview.default_platforms`.

## Generic non-zero exit

```bash
tail -n 20 ~/.agentflow/logs/agentflow.log
```
