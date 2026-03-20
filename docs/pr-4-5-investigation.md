# RMS-53 investigation: PR #4 and PR #5 stuck with `ready-to-merge`

Date: 2026-03-20
Repo: `dkarter/merge-train-sandbox`

## Scope

- Reviewed open PRs #4 and #5 with `ready-to-merge`.
- Checked mergeability, required-check state, and Merge Train behavior.
- Retriggered PR #4 by toggling the label.

## Findings

### PR #4 (`rms26-s3-rerun-success-v2`)

- Status: **OPEN**, `mergeStateStatus=BLOCKED`, `ready-to-merge` still present.
- Older Merge Train runs (action SHA `86b1a2b...`) were green at the workflow level, but the PR did not merge.
- Logs show reruns were skipped with `422 This check run is not rerequestable`, so required check `lightweight-ci` stayed failing.
- After retrigger, Merge Train used newer action SHA `328b2d3...`, confirming this is no longer an old-action-only issue.
- The newer run attempted safe `update branch` (PR was behind) and then timed out waiting for checks on updated head SHA `7ec7091...`.
- Result: PR #4 is still unmerged.

### PR #5 (`rms26-s4-rerun-still-fails`)

- Status: **OPEN**, `mergeStateStatus=BEHIND`, `ready-to-merge` still present.
- Required check `lightweight-ci` is still failing.
- Merge Train workflow is green, but action logs show a blocked outcome because required checks still fail after one-time rerun handling.
- This matches expected scenario 4 behavior (checks remain failing).

## Diagnosis summary

- A green Merge Train workflow does not guarantee merge; it only means the action finished.
- PR #4 is **not** blocked only by old action code anymore; retrigger used newer action SHA and still timed out after safe branch update, with required checks unresolved on the updated head.
- PR #5 remains correctly blocked because required checks are failing.

## Actions performed

- Retriggered PR #4 by removing and re-adding the `ready-to-merge` label.

Commands used:

```bash
gh pr edit 4 --repo dkarter/merge-train-sandbox --remove-label ready-to-merge
gh pr edit 4 --repo dkarter/merge-train-sandbox --add-label ready-to-merge
```

## Evidence URLs

### Pull requests

- PR #4: https://github.com/dkarter/merge-train-sandbox/pull/4
- PR #5: https://github.com/dkarter/merge-train-sandbox/pull/5

### PR #4 workflow runs and checks

- Retriggered Merge Train run (new action SHA `328b2d3...`): https://github.com/dkarter/merge-train-sandbox/actions/runs/23329269079
- Retriggered CI run (failed before safe update): https://github.com/dkarter/merge-train-sandbox/actions/runs/23329269064
- Prior Merge Train run (old action SHA `86b1a2b...`): https://github.com/dkarter/merge-train-sandbox/actions/runs/23327622582
- Prior CI run (failed): https://github.com/dkarter/merge-train-sandbox/actions/runs/23327622587

### PR #5 workflow runs and checks

- Merge Train run (blocked in action logic, workflow succeeded): https://github.com/dkarter/merge-train-sandbox/actions/runs/23327909530
- Latest CI failure: https://github.com/dkarter/merge-train-sandbox/actions/runs/23327909528
- Earlier CI failure: https://github.com/dkarter/merge-train-sandbox/actions/runs/23327908441
