# Members

Members are defined by model and effort. Resolve provider instances at runtime using [backend.md](backend.md).

## Core (binding votes)

| Label | Model match | Preferred harness | Effort |
| --- | --- | --- | --- |
| Opus | Opus 5.5 | Claude | medium |
| Sol | GPT-6.1 Sol | Codex | medium |

- Core effort is medium in every tier and mode unless the user overrides it for the session.
- If the preferred harness is unavailable, another harness serving the same model may be used. Report the substitution.
- Gates require both core votes. Either member can block.

## Associate pool (advisory, no veto)

The user edits and orders this list. Pick the first available model that fits the mode and can be kept away from secret-containing files.

| Label | Model match | Good for |
| --- | --- | --- |
| Argon | Gemini 4 Argon | plan, decide (long context) |
| DeepSeek | deepseek-v4-pro | merge, diagnose |
| GLM | glm-5.3-flash | merge, consult |
| MiMo | mimo-v2.6-pro | diagnose, decide |

- Use at most one associate, only in heavy tier, and from a vendor different from both core vendors.
- Associates must not read secret-containing files. This includes `appsettings*.json`, `.env*`, credentials, private keys, tokens, and files identified as containing secrets regardless of name. Do not include their contents in briefs, diffs, logs, or follow-up rounds.
- A prompt prohibition alone does not restrict filesystem access. Use an enforceable isolated review workspace containing only sanitized material, or skip the associate. Never point the associate at the original workspace when it can read those files.
- Set medium effort if supported; otherwise use the provider default and report any uncertainty.
- Never choose an associate because it is cheap, fast, or popular.
- If none is available, proceed without one and say so. This does not block.

An associate cannot veto, but a specific evidenced BLOCKING finding (file, line, concrete failure) cannot be ignored. Fix it if in scope, or send it to both core members in the next round as “Reviewer C raised:”. If either core member agrees, it blocks.

For a missing core member, stop a gate and report why. An advisory session may use one core member only if the user agrees; label it “partial council”.
