# Mode: diagnose (advisory)

Role: `research`. Subject: a bug, incident, or flaky behavior.

Brief: symptoms, reproduction, logs, stack traces, recent changes, environment, ruled-out causes and evidence, relevant paths. Do not include your theory in round one.

Check ranked root-cause hypotheses, evidence for and against each, cheapest discriminating tests, and possible multiple causes.

Response format (max 250 words):

1. First line: TOP CAUSE: <one line> | CONFIDENCE: low/med/high
2. Ranked hypotheses with evidence
3. Cheapest discriminating checks first
4. Minimal fix if confident, described but not applied

Combine hypotheses, ranking independently shared causes highest. Run cheap checks yourself, then report. If fixing is authorized, apply the minimal fix and chain into `merge` when that session was requested.
