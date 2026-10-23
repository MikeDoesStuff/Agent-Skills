# Mode: plan (gate)

Role: `design`. Subject: a plan, spec, migration, or task breakdown before implementation.

Brief: goal, full plan text, constraints, relevant paths, non-goals, scope statement.

Check whether the plan reaches the goal; missing steps and ordering; rollback or safe failure for the riskiest step; data, migration, and compatibility hazards; hidden assumptions; and work that can be cut.

Response format (max 250 words, findings only):

1. First line: APPROVE | APPROVE WITH CHANGES | REJECT
2. Strongest part of the plan
3. Smallest required changes, each marked BLOCKING or NON-BLOCKING
4. Open questions the lead must answer first

Start implementing when both core members APPROVE, or their requested changes are only NON-BLOCKING, and no evidenced associate BLOCKING finding remains. Fold in in-scope non-blocking items. For a BLOCKING item or REJECT, revise and run another round. Ask the user about unanswerable open questions. After implementation, chain into `merge` when that session was requested.
