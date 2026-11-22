# OpenWrt Cache Cleanup Gate Review

**Recommendation:** APPROVE

## Scope

Reviewed `.github/workflows/cleanup_cache.yml`, the Build WRT and Release WRT cache key producers, `.omo/evidence/cleanup-cache-code-review.md`, and the final fixture evidence under `/tmp/ulw-cache-cleanup-D1FFOi/`.

## Review

The workflow listens for `completed` runs of the exact `Build WRT` and `Release WRT` names and runs its job only for successful `workflow_dispatch` runs. This places cleanup after the producer workflow's `actions/cache` post-save phase. It uses `actions: write` and a non-cancelling concurrency group.

The REST API listing is paginated with `per_page=100`. The cleanup accepts only the fixed `openwrt-build-v1-` family and the exact anchored key pattern used by both producer workflows. It rejects malformed keys, a broadened family, or a broadened pattern before cache selection. Caches are grouped by their API `ref` and normalized source/runner key; the greatest `(created_at, id)` entry is retained, and older entries are deleted through numeric cache IDs.

## Evidence

- YAML structure and workflow trigger: PASS.
- Four-page fixture with 12 caches and four groups: PASS; IDs `200`, `201`, `203`, `204`, `206`, and `207` are selected, while newest entries, unrelated keys, and another ref remain.
- Malformed cache key mutation: PASS; exits nonzero and deletes nothing.
- Broadened cache family mutation: PASS; exits nonzero and deletes nothing.
- Broadened key-pattern mutation: PASS; exits nonzero and deletes nothing.
- Embedded shell syntax, YAML parsing, and new-file whitespace checks: PASS.
- Independent code review: APPROVE, with the redundant pre-group sort removed and the review report force-added to the staged change.

No live GitHub API deletion or hosted workflow run was performed; those remain unverified runtime operations rather than code-review blockers.
