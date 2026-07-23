---
name: finn-review
description: Review open PRs against their linked Linear issues and required GitHub checks, then post a three-group verdict with Finn-loop labels. Use when asked to run Finn-loop's reviewer or review its PR queue. Designed for /loop; never merges or pushes code.
---

# Finn-loop reviewer

One pass = one PR reviewed. Under `/loop`, each iteration runs this skill once.

## 1. Find a PR needing review

```bash
gh pr list --state open --json number,title,labels,isDraft,headRefOid,updatedAt,url
```

Skip drafts. Skip every PR carrying `loop-stuck`: it has been stopped after
two failed fix rounds and stays out of the review queue until a human removes
the label. When a human removes `loop-stuck`, the fix-round count restarts
from zero — verdicts posted before the removal are never counted again
(step 4 enforces this via the label timeline).

For each remaining PR, find the latest comment whose first line is
`Finn-loop review of COMMIT_SHA`.

Skip a PR when that recorded SHA equals its current `headRefOid` and it already
has `loop-approved`, `loop-changes-requested`, or `needs-human-review`. Review
it again when new commits landed after the recorded SHA. If nothing needs
review, say so and end the pass.

## 2. Read the contract and code

- Parse the linked issue identifier from `Closes NEX-NNN` in the PR body and
  fetch the full Linear issue, including comments and relations. No linked
  issue is a must-fix finding.
- Read the full diff and every changed file in context.
- Review only against the linked issue: acceptance-criteria gaps, defects,
  broken data flow, unnecessary scope expansion, security problems, missing
  loading/error states, and code future agents will struggle to modify.
- Do not suggest unrelated improvements unless they are severe.

Every must-fix code finding starts with one of:

- `[AC-N]` — the PR does not satisfy that acceptance criterion
- `[DEFECT]` — the implementation is broken while staying inside scope
- `[SECURITY]` — a severe security issue blocks shipping
- `[CI]` — a required GitHub check failed

Non-goals are binding. If fixing a finding would require behavior excluded by
an `NG-N`, do not prescribe code. Record
`[SCOPE-CONFLICT AC-N ↔ NG-N]` with the exact contradiction and mark the PR for
human escalation.

## 3. Check merge evidence

Inspect the current PR head, mergeability, and required checks:

```bash
gh pr view NUMBER --json headRefOid,mergeable,mergeStateStatus
gh pr checks NUMBER --required --json bucket,name,state,link
```

- If required checks are pending or mergeability is still unknown, report that
  the PR is waiting and end without posting a verdict or changing labels. A
  later loop pass will retry it.
- Failed required checks are `[CI]` must-fix findings.
- A merge conflict is a `[DEFECT]` must-fix finding.
- If the repository has no required checks, mark the PR for human escalation;
  do not apply `loop-approved`. Finn-loop does not treat missing CI as green.

Review the exact `headRefOid` used for this evidence. Re-fetch it immediately
before posting. If it changed, discard the review and start again on a future
pass.

## 4. Loop-stuck brake (two failed fix rounds)

Runs only when the verdict about to be posted contains must-fix findings.
Count the earlier failed fix rounds on this PR:

1. Collect every existing comment whose first line is
   `Finn-loop review of COMMIT_SHA` and whose "Must fix before merge" section
   lists at least one finding.
2. Keep only verdicts recorded on a head SHA different from the current
   `headRefOid` — the same SHA is never counted twice.
3. Fetch the PR's label timeline
   (`gh api repos/OWNER/REPO/issues/NUMBER/timeline --paginate`) and find the
   most recent `unlabeled` event for `loop-stuck`. Keep only verdicts posted
   after that event; if `loop-stuck` was never removed, keep them all. A
   human's label removal therefore resets the count to zero.

If two such verdicts remain, the verdict about to be posted would be the third
must-fix in a row — two fix rounds have already failed. Stop the PR instead of
sending it back to the builder:

- Add `loop-stuck` and `needs-human-review`; remove `loop-approved` and
  `loop-changes-requested` (check existing labels first, as in step 5).
- Say in the verdict that the PR has been stopped and why: two fix rounds
  failed, listing the counted verdict SHAs. Set "Safe to merge" to
  `No — stopped after two failed fix rounds; a human must take over.`
- Post the Slack alert below, then end the pass.

Slack alert — uses the Slack connector (`slack_read_channel`,
`slack_send_message`). If the connector is unavailable in this session, skip
this alert silently, exactly like the other Slack steps. Channel:
`#notifs-factory`, ID `C0BK2JYC5GT`. Post one message:

```text
:warning: Fastnad: PR #NUMBER — TITLE
PR_URL
SHA: FULL_HEAD_SHA
Läs senaste verdiktet, åtgärda eller stäng PR:en, och ta bort `loop-stuck` + `needs-human-review` för att återuppta loopen.
```

Idempotency: first read the channel's recent history (`slack_read_channel`,
~100 messages). If a `Fastnad: PR #NUMBER` message with the same SHA already
exists, do not post again.

## 5. Post one verdict

Post one comment in this structure:

```md
Finn-loop review of COMMIT_SHA

CI: required checks passed | failed | not configured
Mergeability: clean | conflicting

## Review

Summary: one or two plain-language sentences on what this PR does.

## 1. Must fix before merge

None.

## 2. Should fix soon

None.

## 3. Safe to merge

Yes — automated review evidence is complete. A human still makes the merge decision.
```

Then set labels based on the verdict, checking existing labels before removing
them so an absent label does not fail the command:

- No must-fix and no new escalation: add `loop-approved`; remove
  `loop-changes-requested`. Preserve a pre-existing `needs-human-review` label
  because it may represent a separate high-risk human gate.
- Must-fix present and the loop-stuck brake (step 4) did not fire: add
  `loop-changes-requested`; remove `loop-approved`.
- Must-fix present and the brake fired: the labels set in step 4 stand
  (`loop-stuck` + `needs-human-review`); never add `loop-changes-requested`.
- Scope conflict or no required CI: add `needs-human-review`; remove both
  `loop-approved` and `loop-changes-requested`; set "Safe to merge" to
  `No — human decision required.`

The escalation path deliberately leaves the automated repair queue. A human
must resolve the reason, change the issue or repository configuration as
needed, and remove `needs-human-review` before Finn-loop reviews that unchanged
commit again.

## 6. Slack merge-ready notice

Runs only when the verdict just applied `loop-approved`. Uses the Slack
connector (`slack_read_channel`, `slack_send_message`). If the Slack
connector is unavailable in this session, skip this step silently — Slack is
a control surface, never a requirement for the review itself.

Merge channel: `#notifs-merge-ready`, channel ID `C0BKA99U8MP`.

Post one message to that channel with exactly this text shape (the merge
skill parses it):

```text
:rocket: Merge-ready: PR #NUMBER — TITLE
PR_URL
SHA: FULL_HEAD_SHA
Reagera med :rocket: så mergar jag. Jag svarar med :white_check_mark: när det är live.
```

Idempotency and supersession, in order:

1. Read the channel's recent history (`slack_read_channel`, ~100 messages)
   and find existing `Merge-ready: PR #NUMBER` messages.
2. If one exists with the same SHA, do not post again. End the step.
3. If one exists with an older SHA, reply in its thread with
   `Ersatt — ny head SHA: FULL_HEAD_SHA` before posting the new message, so a
   stale 🚀 can never authorize an outdated commit.

## 7. Hard limits

- Never merge or enable auto-merge.
- Never push commits to the PR branch.
- Never approve or request changes through a formal GitHub review. Use one
  comment plus labels because the loop may run on the PR author's token and
  GitHub rejects self-reviews.
- `loop-approved` is evidence for a human, not merge authorization.
