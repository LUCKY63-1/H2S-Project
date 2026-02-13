# Suggestions for Next Iteration

## 1) Add an automated end-to-end smoke test for GitHub → DSPy
Create a lightweight test (Playwright or API-level integration) that:
1. fetches a small public repository,
2. confirms files land in `uploaded_files`, and
3. validates that analysis starts and produces at least one result artifact.

**Why:** The current status notes the critical path is fixed but still pending verification; automating this closes the highest-risk gap quickly.

## 2) Add session-state contract checks
Introduce a small validation utility that asserts required session keys and expected shapes at key transitions (mode switch, fetch completion, analysis start).

**Why:** The integration relies heavily on shared state. Contract checks reduce regressions from future refactors.

## 3) Improve observability with structured events
Emit structured logs/events for major milestones:
- repo fetch started/completed/failed
- file normalization completed
- analysis started/completed/failed
- result rendering completed

Include a correlation ID per run.

**Why:** Faster debugging when users report “analysis did not start” or “no results shown.”

## 4) Add a user-visible source badge and counts
In the dashboard file summary, show:
- source (`upload` vs `github`),
- total files fetched,
- analyzed files,
- skipped files (with reason).

**Why:** Makes the unified storage behavior explicit and reduces user confusion.

## 5) Complete and codify regression matrix
Document and run a minimal matrix before release:
- upload mode happy path
- github mode happy path
- switch modes repeatedly
- clear state + rerun
- large repo with unsupported files

**Why:** The current integration point can regress in mode-switch and cleanup flows.

## 6) Add timeout + retry UX for repository fetch
Add explicit timeout messaging and a one-click retry flow that preserves URL/input settings.

**Why:** Improves reliability perception and reduces abandonment on transient network issues.

## 7) Strengthen export validation
After analysis, validate export payload integrity (non-empty sections, timestamps, source metadata).

**Why:** Prevents silent failures where analysis appears successful but exported report is incomplete.

## 8) Add guardrails for very large repositories
Before full fetch/analysis, estimate repository size and warn users when limits may be exceeded. Offer path filters.

**Why:** Protects runtime performance and prevents stalled sessions.

## 9) Track a short list of quality metrics
Capture and monitor:
- fetch success rate,
- analysis start success rate after fetch,
- median fetch-to-analysis latency,
- result render completion rate.

**Why:** Gives objective evidence that integration quality is improving.

## 10) Update docs with one “golden flow” walkthrough
Create a concise single-page walkthrough from entering GitHub URL to exporting final report, with expected UI states/screens.

**Why:** Reduces support overhead and speeds onboarding for new contributors.
