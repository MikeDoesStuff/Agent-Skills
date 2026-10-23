---
name: council
description: Convene a cross-vendor model council for merge and plan gates or advisory decisions, diagnosis, and consultation. Use when the user asks for a council, asks whether a change is safe to merge or a plan is approved, or requires council agreement before proceeding.
---

# Council

You are the lead. Prepare the brief, convene independent members, reconcile findings, and act only as the selected mode allows.

Read [members](core/members.md), [backend](core/backend.md), and [protocol](core/protocol.md), then only the selected mode file.

| Mode | Kind | Use when | File |
| --- | --- | --- | --- |
| merge | gate | Merging or shipping a change | [merge](modes/merge.md) |
| plan | gate | Approving a plan before building | [plan](modes/plan.md) |
| decide | advisory | Choosing between named options | [decide](modes/decide.md) |
| diagnose | advisory | Investigating an unclear bug or incident | [diagnose](modes/diagnose.md) |
| consult | advisory | Other council questions | [consult](modes/consult.md) |

Routing:

- Work ending in a merge or shipment uses `merge`.
- A request to approve a plan before implementation uses `plan`.
- Two or more named alternatives use `decide`.
- An unclear cause uses `diagnose`.
- If a gate and advisory mode fit equally well, ask one short question.
- Chain sessions when requested: plan → build → merge; diagnose → fix → merge. Each session has its own brief.

## Tier

Choose before convening. The user may override the tier.

| Tier | Panel | When |
| --- | --- | --- |
| light | One core member other than the lead's model | Consult questions; under 50 changed lines with no risk flags |
| standard | Opus + Sol | Default |
| heavy | Opus + Sol + one associate | Auth, payments, data or migrations, infrastructure, concurrency, public API, or “full council” |

Gate modes are never light. Before convening, announce: “Council: <mode>, <tier>, effort medium. Members: <labels>. Backend: <backend>.” Report any unverified effort separately.
