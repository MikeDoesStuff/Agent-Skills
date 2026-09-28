---
name: jg-code-search
description: Use the preinstalled `jg` (jevgrep) CLI to search a local code repository by behavior or intent. Prefer `jg` for semantic discovery when symbols or locations are unknown, and exact/ripgrep-style search for known identifiers, literals, regexes, and exhaustive references. Use this skill when investigating an unfamiliar codebase, tracing behavior across files, locating implementations from natural-language descriptions, or gathering precise file/line evidence before editing code.
---

# jg semantic code search

Use `jg` as a repository-navigation tool for coding work. Assume `jg` is already installed and configured; do not install it or run login flows unless the user explicitly asks.

## Core rule

Choose the search mode from the question:

- **Unknown location / behavior / concept** → use semantic `jg` search.
- **Known symbol, identifier, literal, route, config key, or regex** → use `jg exact` or `rg`.
- **Need strong coverage before saying something does not exist** → use `jg --all` on a suitably narrowed path, then verify important exact identifiers with exact search.

Do not replace `rg` wholesale with `jg`. Semantic search is for finding behavior whose wording or symbol names are not known; exact search remains the right tool for exhaustive literal/regex references.

## Working directory and scope

Run searches from the repository root whenever possible. If not at the root, pass the repository root or a narrow path explicitly.

Plain semantic searches recurse through the current directory. Prefer an explicit path when the likely subsystem is known because it reduces cost and noise.

Examples:

```bash
jg "where do we reject expired sessions?" .
jg "retry a failed network operation" src/
jg "release resources when a request is cancelled" src/network/
```

`jg` reads the current working tree, including uncommitted changes. Treat results as evidence about the files currently on disk, not necessarily HEAD.

## Narrow automatically as part of forming the query

Do not wait for a scope error to force narrowing. Handling it silently as normal workflow:

1. **Whole-repository roots must carry default exclusions.** For a `.` or repo-root search:
   `-g '!**/obj/**' -g '!**/bin/**' -g '!**/node_modules/**' -g '!**/.git/**' -g '!**/dist/**' -g '!**/packages/**'`
   and add stack-specific exclusions contextually (`target`, `vendor`, `Generated`, migration snapshots) based on the repo's own layout.
2. **Type by language.** Behavioral queries target source; add source-only globs (`-g '*.cs'`, `-g '*.ts'`).
3. **Subsystem first, widening later.** Search the most likely subsystem directory, then widen only after the first pass shows the behaviour crosses broadenings.
4. **Unknown repository shape ⇒ --dry-run first.** If the repo size/shape is unknown, `--dry-run` the intended query to detect discovery size before assuming a whole-repo run is fine.
5. **A source-byte-limit rejection (`jg: Search exceeds … source bytes`) is an expected steering signal.** Do not surface it as an error to the user — rerun immediately with exclusions or a narrower path and continue as if nothing happened. Report only what the investigation found.

## Default semantic workflow

For most behavioral investigations:

1. Start with a focused semantic search using a concrete behavior phrase.
2. Read the returned excerpts and exact file/line ranges.
3. Open the relevant source around those lines before making edits or conclusions.
4. Search again with a refined behavior if the first pass reveals better domain terminology.
5. Use exact search for discovered symbols to enumerate call sites, references, registrations, or tests.

Good semantic queries describe **entity + action**, for example:

```bash
jg "validate a refresh token and reject it when expired" src/
jg "map payment provider failures into API error responses" .
jg "release a distributed case lock when the viewer disconnects" src/
```

Avoid vague one-word semantic queries when a behavioral sentence is available.

## JSON for agent workflows

Prefer JSON when parsing or summarizing results programmatically:

```bash
jg --json "reject expired sessions" src/
```

The output includes a versioned schema, matches with `path`, `startLine`, `endLine`, optional `symbol`, source `text`, and a `coverage` object. Test matches may include `kind: "test"`.

When consuming JSON, pay attention to:

- `coverage.mode`
- `coverage.files`
- `coverage.evaluated`
- `coverage.eligible`
- `coverage.selected`
- `coverage.skipped`
- `coverage.selectionComplete`
- `coverage.evaluationComplete`
- `truncated`
- `omittedMatches`

Do not treat a no-match result as proof of absence unless the coverage is appropriate for that claim.

## Search modes

### Default shortlist search

Use the normal form first for quick semantic discovery:

```bash
jg "behavior description" path/
```

The default semantic path uses a local lexical/BM25 shortlist before Jev evaluates candidates. This is fast and usually appropriate for discovery, but semantically relevant code can be missed when its vocabulary differs greatly from the query.

Use `--limit N` when more than the default top matches are useful:

```bash
jg --limit 15 "behavior description" src/
```

Use `--json` when the result will feed further agent reasoning.

### Broad search

Use `--broad` when the default shortlist seems too narrow and the scope is still manageable:

```bash
jg --broad "release resources on cancellation" src/network/
```

This increases semantic evaluation coverage without necessarily scanning every eligible snippet.

### All-snippet search

Use `--all` when high search coverage matters, especially before making a negative claim such as “there is no code that does X.” Narrow the path first when possible.

```bash
jg --all "where do we reject expired sessions?" .
jg --all --limit 30 "retry a failed network operation" src/
```

`--all` evaluates every eligible discovered snippet within the configured limits; it does **not** guarantee semantic recall, include ignored/hidden/binary files, or place the whole repository into one shared model context.

The default evaluation budget is large but bounded. If the scan would exceed the budget, `jg` fails before model requests instead of silently omitting code. Prefer narrowing paths before increasing budgets.

If needed:

```bash
jg --all --max-evaluations 5000 "behavior description" src/service/
```

Do not raise evaluation budgets reflexively across the entire repository.

## Dry-run before expensive searches

Use `--dry-run` to inspect discovery size, snippet counts, and estimated request count without model calls:

```bash
jg --all --dry-run "where do we reject expired sessions?" .
```

For sensitive repositories, remember that real semantic searches send selected source/context to the configured backend. Dry-run itself does not make model calls.

When debugging candidate selection, use:

```bash
jg --dry-run --dump-candidates --json "billing rules" src/
```

Only use candidate dumps when needed; they can produce larger outputs.

## Exact search

For exact symbols, literals, regexes, or exhaustive reference searches, use `jg exact`. It delegates arguments to ripgrep without changing their semantics.

```bash
jg exact -n --glob '*.ts' 'refreshToken' src/
jg exact -n 'CaseViewLockProvider' .
jg exact -n -F '/api/v1/Leads/byRefs' .
```

If ordinary `rg` is already part of the coding workflow, using `rg` directly is equally appropriate for exact search.

A strong combined workflow is:

```bash
jg --json "where is an application lookup by affiliate reference handled?" .
# discover symbol/path, then:
jg exact -n 'GetApplicationByCode' .
```

## Scoping with globs

Use repeatable ripgrep-style globs to reduce irrelevant source:

```bash
jg -g '*.ts' -g '*.tsx' "where is authentication state refreshed?" src/
jg --dry-run -g '*.cs' "where are late fees capped?" .
```

Prefer path narrowing and globs over huge repository-wide all-mode searches.

## Getting more context

Default matches are deliberately compact excerpts. Once a match looks relevant, read the source file around the reported lines with the normal repository/file tools.

Use `--full` only when a larger snippet is genuinely useful:

```bash
jg --broad --full "release resources on cancellation" src/network/
```

Even full output remains subject to the overall output budget. If output is truncated or matches are omitted, increase `--limit` / `--max-output` carefully or read the source directly.

## Files-only mode

When you only need candidate file names:

```bash
jg --files "retry failed requests" src/
```

Then inspect those files using normal repository tools.

## Interpreting coverage correctly

Coverage metadata is essential when making completeness claims.

- `selectionComplete: false` means not every eligible discovered snippet was selected for semantic evaluation.
- `evaluationComplete: false` means evaluation was not complete, including dry-run cases or searches with skips.
- Ignored and hidden files are excluded from normal discovery.
- Binary, non-UTF-8, oversized files, giant/minified lines, and other unsupported content may be skipped.
- A no-match response is evidence that the performed search found nothing, **not** proof the behavior is absent from the repository.

Before stating “there is no implementation/reference for X,” prefer:

1. a narrowed `jg --all` behavioral search;
2. exact searches for likely/discovered names and literals;
3. inspection of relevant registration/configuration files if applicable.

Phrase residual uncertainty when coverage is incomplete.

## Tests are evidence

Do not discard matches under test paths. `jg` intentionally ranks tests on relevance and labels them with `kind: "test"` in structured output. Tests can be the clearest statement of intended behavior.

Use test matches to:

- discover production symbols;
- understand expected edge cases;
- identify likely implementation files;
- confirm behavior after an edit.

Then trace from the test into production code with exact search.

## Sensitive-code awareness

Real semantic searches may send selected source excerpts and bounded context to the configured Jevgrep backend. Filesystem paths remain local, but source/context from selected candidates is transmitted for inference.

If the repository is sensitive and the user has not already established that remote semantic search is acceptable, avoid unnecessarily broad candidate dumps and prefer narrowly scoped searches. Use `--dry-run` when you only need to inspect the planned scope.

Do not expose credentials, secrets, tokens, or private source in chat output beyond what is required for the task.

## Failure handling

Treat exit codes as follows:

- `0`: matches or dry-run preview
- `1`: no matches
- `2`: error
- `130`: interrupted

A backend/batch failure means the semantic search failed; do not describe it as a successful partial result.

If semantic search fails because of auth, rate limits, or network issues, continue with local exact search / repository inspection where possible rather than attempting installation or authentication unless asked.

## Query-name edge case

If the semantic query itself is named like a CLI subcommand such as `login`, `status`, or `mcp`, separate options from the query with `--`:

```bash
jg -- "login" .
```

## Agent decision guide

Use this sequence by default:

```text
Do I know the exact symbol/literal/regex?
  yes -> jg exact / rg
  no  -> semantic jg

Did semantic search find a promising implementation?
  yes -> read surrounding code, then exact-search discovered symbols/references
  no  -> refine query and/or narrow subsystem; consider --broad

Am I about to claim the behavior does not exist?
  yes -> narrow path + --all, inspect coverage, then exact-search plausible identifiers/config

Is the repository-wide --all scan large?
  yes -> --dry-run first; narrow paths/globs before raising max evaluations
```

## Recommended response behavior

When reporting findings from `jg`:

- cite exact repository paths and line ranges from the search result;
- distinguish test evidence from production implementation;
- summarize what the matched code actually does rather than relying on the semantic score alone;
- mention incomplete coverage when it materially affects the conclusion;
- open/read the matched source before proposing edits whenever practical.

Do not describe the semantic score as a calibrated probability. Use it only as a ranking signal.

## Reference

Source guidance adapted from the jevgrep (`jg`) README:
https://github.com/remotehostai/jg#readme
