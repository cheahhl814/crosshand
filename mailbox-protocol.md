# crosshand mailbox file contract (single source of truth)

This document defines the mailbox file contract for a repo's `mailbox/`
directory. SKILL.md must not diverge from it; amend it here when the contract
changes.

## Layout

```
mailbox/
  <sender-identity>/H-<YYYY-MM-DD>-<seq>-<from>-<to>-<topic>.md
  ARCHIVE.md          # compacted history (append-only displacement)
```

- The mailbox is shared; each sender owns exactly one sender dir. Senders
  write only into their own dir — no separate outbox layer exists.
- One file per exchange. Never two exchanges in one file, never one exchange
  spread across files.
- Ids: `H-<date>-<seq>`, seq = zero-padded 2 digits, per-sender, resets at the
  exchange's local midnight. The slug in the filename repeats id/from/to/topic.
- `<sender-identity>` = `<harness>@<machine-slug>`, agreed once by the team;
  it is the contact-list identity, not an email address.

## Frontmatter

| Field | Required | Meaning |
|---|---|---|
| `id` | yes | `H-<date>-<seq>`; the reply key |
| `from` / `to` | yes | identities; `to` is exactly one identity, or the literal `room` for group announcements |
| `reply_to` | yes (`null` if none) | id of the message this answers |
| `status` | yes | sender-writable: `open`, `done`, `superseded` (by new id). Receiver-writable (one-to-one only): `acked`, `done` |
| `created` | yes | ISO 8601 with timezone |
| `repo` | yes | shared repo URL |
| `base` | yes | `git rev-parse HEAD` when written; must be reachable from `origin/main` at send time |
| `subject` | yes | one line |
| `security` | yes | literal `no-secrets` — the file is guaranteed free of credentials/PII |

## Body sections

`## Summary`, `## Facts` (every row: exact path + commit SHA), `## Asks`
(numbered, phrased for the receiver's human to approve), `## Verification`
(commands + expected output), `## Reply` (instructions: new file in the
responder's own dir with `reply_to`, reference PR).

Rules that make the chain unambiguous:

- **Single-writer.** A file's content belongs to its sender forever. The only
  exception: the receiver of one-to-one mail may set its `status:` line.
  `room` messages are read-only.
- **SHA-pinned facts.** No path appears in Facts without its commit SHA.
- **Errors are never corrected in place.** Send `status: superseded` reference
  and a new id.
- **Retention.** Completed/superseded exchanges older than 30 days are moved
  verbatim to `ARCHIVE.md` by an auditor compaction (recent 2 stay); the
  displacement note lists what moved when.
- **Human-to-human notes ride the announce PR's comment thread — never
  inside an `H-*.md`.** PR comments are GitHub-layer metadata, invisible to
  `git pull`, so harness mailbox sweeps are structurally blind to them.
  Mention the recipient's GitHub handle (`@handle`) in the PR body or first
  comment: a mention is the guaranteed notification path (mobile push via
  the GitHub app, or the same event mirrored as email — per each user's
  notification settings). Only what the local human explicitly quotes out
  of a PR thread is ever human input to a harness.
