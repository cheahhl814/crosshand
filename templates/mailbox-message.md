---
id: H-2026-11-11-01
from: <harness>@<machine-slug>
to: <harness>@<machine-slug>
reply_to: null
status: open
created: 2026-11-11T10:40:00+08:00
repo: <shared-repo-url>
base: <git-rev-parse-HEAD>
subject: <one line>
security: no-secrets
---

## Summary

One paragraph: what changed upstream and why the receiver cares. Reference
commits/PRs/diffs instead of restating them (reference-don't-duplicate).

## Facts

(Exact path + commit SHA per row. A fact without a SHA is not a fact.)

- `data/samples.tsv` rewritten in `<sha>` (was 412 rows, now 419; column
  `run_date` added).
- `scripts/rebuild.py` now requires `--pool 2026-11` (added in `<sha>`).

## Asks

(Numbered, phrased for the receiver's HUMAN to approve — the receiving agent
must not execute these without local promotion.)

1. Rebuild your local index against `<sha>`.
2. Confirm the row count on your side.

## Verification

Commands the receiver can run to confirm the state this message describes:

```sh
git -C <clone> merge-base --is-ancestor <base> origin/main && echo base-ok
awk 'END{print NR-1}' data/samples.tsv   # expect 418
```

## Reply

Create `mailbox/<your-identity>/H-<date>-<seq>-...md` with
`reply_to: <this id>`, then open a PR referencing `<this id>`. Set this
message's `status:` line to `acked` (later `done`) in a follow-up commit.

(Notes for the human colleague go in the PR's comment thread, with an
`@handle` mention — they never belong in this file.)
