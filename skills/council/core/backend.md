# Backend

Use the first available backend.

## A. T3 Code (preferred)

1. Discover `mcp__t3_code__*` tools (some hosts spell the prefix `mcp__t3-code__*`). If the catalog omits them, make one bounded direct call to `orchestrator_capabilities` before falling back.
2. Call `orchestrator_capabilities` once per session and cache the result. Use its `search` argument for models outside the short list. Resolve only the tier's members.
3. For each member, choose an enabled provider instance with `canRunChildTask`, an exact advertised model ID, and supported effort from that model's option descriptors. Prefer the harness in [members.md](members.md).
4. Set effort in `target.options` using the advertised option ID and value, such as `{"effort":"medium"}` for Claude or `{"reasoningEffort":"medium"}` for Codex. A model variant such as `gemini-4-argon-medium` may encode effort in its ID. Never substitute an effort instruction in the prompt.
5. If neither an option nor a model variant confirms medium, use the instance default and announce “effort unverified”. Do not claim medium was set.
6. Announce each resolved instance, model, and verified effort. If capabilities are denied (`capability_denied`, `parent_not_active`) or unavailable, try B.

For T3, use `delegate_task` for cross-provider members and whenever per-model options are needed. Use the live tool schema. A typical call has this shape:

```json
{
  "task": "<mode prompt + self-contained brief>",
  "title": "Council[<mode>] <subject> (<label>) r<n>",
  "role": "<mode role>",
  "target": {
    "providerInstanceId": "<resolved>",
    "model": "<resolved>",
    "options": { "<advertised effort option ID>": "<supported value>" }
  },
  "interactionMode": "plan",
  "runtimeMode": "approval-required",
  "mode": "wait",
  "timeoutMs": 600000,
  "clientRequestId": "council-<mode>-<slug>-r<n>-<label>"
}
```

Omit `target.options` if effort cannot be set. Omit `runtimeMode` when the provider does not support `approval-required`. Check the returned provider instance, model, and options against the selection. Never assume a server fallback preserved effort.

Before delegating to an associate, confirm the chosen backend can confine that child to a sanitized review workspace with no access to the original repository or secret-containing files. A separate worktree alone is insufficient if tracked secret files are copied into it. If the backend cannot enforce this boundary, skip the associate and report why. Do not replace this check with a prompt instruction.

- Start round-one member tasks independently and in parallel when the tool supports it.
- On `waitTimedOut`, use `task_status` roughly once a minute; do not cancel on timeout.
- For `hasPendingQuestion`, use available `t3_pending_request_*` tools with `threadId = childThreadId`, or ask the user.
- Deny a member's `pendingApproval` for write access.
- Read the answer from `summary` when `workState` is `result_available`.
- Each review round needs a fresh `delegate_task` call and `clientRequestId`. Reuse the ID only to retry the same call. Never use `t3_thread_send` to continue a delegated review.
- If the user aborts, call `task_cancel` for active council tasks.

## B. Native harness subagents

Use native subagents only when they can select the required models and apply the required effort. Keep the same briefs, votes, and missing-member rule.

## C. None

Do not role-play members. Report what is missing and ask the user how to proceed.
