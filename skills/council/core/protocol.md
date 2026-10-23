# Protocol

## Token budget

- Brief by reference. Members can read the shared workspace: send diff (`git diff -U10`), file paths, and scope; paste only material they cannot read, such as logs, plan text, and external constraints. Each brief must stand alone because members see no prior history.
- Cap responses at 250 words, findings only. A SAFE or APPROVE with no findings takes three lines.
- Later rounds send deltas: new diff hunks, one-line status per prior finding, and open objections.
- Cross-examine only conflicting findings and only with members who disagree.
- For in-scope fixes under ten lines touching only flagged lines, recheck only with the members who requested them.
- Stop after three rounds and present all remaining answers to the user.

## Rounds and votes

- Round one is independent. No member sees another's answer.
- Anonymize later findings as Reviewer A/B/C so members discuss evidence.
- Gates require both core members to approve. Associates advise under [members.md](members.md).
- A failed, cancelled, interrupted, or unparseable response is a missing vote. Retry once; a missing core vote blocks.
- One strong, specific objection outweighs vague approvals. Never average away dissent or proceed through a gate on the lead's judgement alone.

## Scope

Keep repeated review from expanding the task. Sort each requested change:

- In scope and needed for correctness or safety: make the smallest fix.
- Valid but not needed now: record as a follow-up.
- Wrong or inapplicable: reject with a one-line reason.

Members read files and run read-only commands. They do not edit, push, merge, or grant approvals.

Final report: mode, tier, backend, rounds; one line per member per round; outcome; dissents and residual risks; out-of-scope follow-ups.
