# Independent Code Review: OpenWrt Cache Cleanup

**Initial codeQualityStatus:** WATCH
**Initial recommendation:** APPROVE
**Initial blockers:** None
**Final codeQualityStatus:** WATCH
**Final recommendation:** APPROVE
**Final blockers:** None

## Scope

Reviewed `.github/workflows/cleanup_cache.yml` together with the Build WRT and Release WRT cache producers. The goal is to run cleanup after successful manual Build WRT or Release WRT runs, retaining the newest cache for each cache ref, runner OS, and source-hash identity.

## Initial Finding and Resolution

The initial review identified one non-blocking MEDIUM finding: the jq selector sorted by `[ref, normalized-key]` immediately before `group_by` on the same expression. jq `group_by` performs that ordering itself, so the explicit sort was redundant.

The explicit sort was removed. The final selector now groups directly by `[ref, normalized-key]`, then sorts entries within each group by `[createdAt, id]` to retain the newest cache deterministically.

## Final Review

The exact `workflow_run` names match the producer workflows. The `completed` trigger and job condition restrict cleanup to successful `workflow_dispatch` runs, after the producer workflow and cache post-save complete. The cleanup workflow grants only `actions: write`, serializes runs without cancellation, and requires no checkout.

The REST cache listing is paginated with `per_page=100`; `set -euo pipefail` prevents listing, parsing, or selection failures from reaching deletion. Runtime guards pin both the exact cache family and anchored key pattern, rejecting broadened namespaces, broadened regexes, and malformed keys before any deletion. Selection groups by the API cache ref plus the key with only its numeric run and attempt suffix removed. It retains the maximum `(createdAt, id)` tuple, and deletes older entries using numeric cache IDs.

No test sources were added or changed. The fixture selector verifies an exact deleted-ID set across multiple pages, refs, runner OSes, and source hashes. Malformed-key and broadened-family mutation runs both fail closed without deleting entries.

## Programming Perspective

The workflow keeps the cleanup logic inline and introduces no new helper abstraction. It validates its fixed key namespace and full key pattern, runs with `set -euo pipefail`, paginates the API response, and deletes by numeric cache ID. No production language module or test file was changed.

## Remove-AI-Slops and QA Overfit Coverage

No test declarations were added, removed, or modified, so there are no deletion-only or tautological test changes to flag. The fixture uses multiple API pages, refs, operating systems, and source hashes with an exact expected deletion ID set. Malformed-key, broadened-family, and broadened-pattern mutations all must exit nonzero without deleting anything. The evidence does not claim a live GitHub API run or hosted cleanup execution.

## Final Verification

- Bun YAML parsing and workflow structure: PASS.
- Embedded shell `bash -n`: PASS.
- New-file whitespace check: PASS.
- Fixture selection retains the newest cache per group and selects only superseded cache IDs: PASS.
- Malformed namespace key mutation: exits nonzero and deletes nothing.
- Broadened cache family mutation: exits nonzero and deletes nothing.
- Broadened cache key regex mutation: exits nonzero and deletes nothing.
- The earlier non-blocking redundant-sort finding was removed; the final jq grouping output was replayed against the fixture and retained the expected ID selection.
- No live GitHub API cleanup or hosted workflow run was performed.

Final evidence: `/tmp/ulw-cache-cleanup-D1FFOi/workflow-structure.json`, `cleanup-selection.json`, `cleanup-mutation.json`, `cleanup-safety.json`, and `checks.json`.
