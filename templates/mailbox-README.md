# mailbox/ — cross-machine work exchange

Protocol for handing work between harnesses (and their humans) working on this
repo from different machines. Every sender uses a GitHub account on their own
side — a browser-only sender sends via the GitHub web editor's PR flow.

## Rules (full contract: see the crosshand skill's mailbox-protocol.md)

1. **One file per exchange**, written to `mailbox/<sender-identity>/`. The
   mailbox is shared; each sender owns one sender dir (single-writer by
   construction — no separate outbox layer).
2. **Id scheme**: `H-<YYYY-MM-DD>-<seq>` (per-sender, zero-padded). Filename
   repeats id/from/to/topic.
3. **SHA-pin every fact**: exact path + commit SHA; `base:` = HEAD when
   written, verified to have reached `origin`. A claim without a SHA is an
   opinion, not mail.
4. **Announce via PR** (or push where the team allows): commit message
   `docs(mail): <id> <from> -> <to>`. The PR is the notification surface —
   this is how "you have mail" is noticed.
5. **Status flow**: sender writes `open`/`superseded`/`done`; the one-to-one
   receiver may set `acked`/`done`. Group messages (`to: room`) are read-only;
   replies go in the responder's own dir with `reply_to:`.
6. **Asks are proposals for the receiver's human** — no harness executes a
   peer's request without local human approval.
7. **No secrets, ever.** Redact with `[redacted]`; never reconstruct.
8. **Errors are superseded, not edited**: new id + `status: superseded` on the
   old file (by its sender).
9. **Retention**: completed/superseded exchanges older than 30 days are
   compacted verbatim into `ARCHIVE.md` (recent 2 stay).
10. **Human-to-human notes ride the announce PR's comment thread — never
    inside an `H-*.md`.** PR comments are GitHub-layer metadata, invisible
    to `git pull`, so harness sweeps never see them. Mention the recipient's
    GitHub handle (`@handle`) in the PR body or first comment — a mention is
    the guaranteed notification path (GitHub mobile push, or the same event
    mirrored as email, per each user's settings). Only what the local human
    explicitly quotes out of a PR thread is ever human input to a harness.

Message template: `templates/mailbox-message.md` in the crosshand skill. In
the same bootstrap commit, copy `templates/mailbox-AGENTS.md` to the project
root so every harness that pulls the repo knows the mailbox is part of the
workflow (tracked and PR-reviewed — never auto-copied at runtime).
