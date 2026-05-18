# Stage Templates

Use these templates as compact artifacts. Do not fill sections with speculation. Mark assumptions and ask the user to confirm key decisions.

## Brief

```md
# Brief: <feature>

## Scope

-

## Non-Scope

-

## Success Criteria

-

## Risks

-

## User Decisions

-
```

## PRD / BDD

````md
# PRD / BDD: <feature>

## User Flow

1.

## Success Scenarios

-

## Failure Scenarios

-

## Edge Cases

-

## BDD Scenarios

```gherkin
Feature: <feature>

Scenario: <specific behavior>
  Given <initial state>
  When <action>
  Then <observable result>
```

## Open Questions

-
````

## Test Strategy

```md
# Test Strategy: <feature>

## P0 Tests

Must pass before release.

-

## P1 Tests

Important edge and failure coverage.

-

## P2 Tests

Useful but deferrable.

-

## Test Type Split

Unit:
Integration:
E2E:
Regression:

## Test Data / Fixtures

-

## Regression Candidates

-

## Test Strategy Review

Use a bounded review-agent pass when risk justifies it.

Reviewer Input:
- Brief:
- PRD / BDD:
- Test Strategy:
- Architecture context:
- Risk rationale:

Decision:
-

Blocking Gaps:
-

P0 / P1 / P2 Changes:
-

Follow-ups Allowed:
- P1/P2 gaps without P0, security, or data risk:

Test Type Split Corrections:
-

Open Questions:
-
```

## Architecture Decision

```md
# Architecture Decision: <feature>

## Modules

-

## Data Flow

1.

## Dependency Direction

-

## State / Storage

-

## External Dependencies

-

## Tradeoffs

-
```

## Observability

```md
# Observability: <feature>

## Diagnostic Questions

If this fails in production, within 3 minutes we need to know:

1.

## Fields

job_id:
stage:
event:
error_code:
debug_artifacts:

## Stages

| Stage | Start Event | Success Event | Failure Event | Error Codes | Debug Artifacts |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

## Logging Rules

-
```

## Plan / Task Handoff

```md
# Plan / Task Handoff: <feature>

This is a ClearFlow handoff artifact, not the detailed implementation plan.

## Goal

-

## Scope / Non-Scope

Scope:
Non-scope:

## Confirmed BDD Scenarios

-

## Tests

P0:
P1:
P2:

Review agents, if used:
- Test Strategy Review: Used / Not used / N/A; reason:
- Unit Test Review: Used / Not used / N/A; reason:
- Acceptance Review: Used / Not used / N/A; reason:

## Architecture Decisions

-

## Observability Requirements

-

## Regression Rules

-

## User-Confirmed Decisions

-

## Known Files / Modules

-

## Handoff to superpowers writing-plans

Use this document plus the referenced workflow artifacts to create a detailed implementation plan. The superpowers plan should include file-level steps, failing tests, implementation steps, commands, and expected results.
```

## Pre-PR Review

```md
# Pre-PR Review: <feature>

## Scope Check

- [ ] Scope fulfilled
- [ ] Non-scope not accidentally implemented

## Behavior Check

- [ ] BDD scenarios covered
- [ ] P0 tests present and passing
- [ ] Test Strategy Review blocking gaps resolved or accepted
- [ ] Unit Test Review blocking gaps resolved or accepted

## Architecture Check

- [ ] Architecture decisions respected
- [ ] No unnecessary dependency direction changes

## Observability Check

- [ ] Failures can be localized by stage
- [ ] Required error codes exist
- [ ] Debug artifacts are captured where needed

## Regression Check

- [ ] Fixed bugs have regression coverage
- [ ] Regression rules updated

## Acceptance Review Summary

- [ ] Scope, BDD, tests, architecture, observability, and regression status reviewed
- [ ] Blocking gaps resolved or explicitly accepted
- [ ] Follow-ups recorded

## Result

Decision:
Blocking issues:
Follow-ups:
```

## Acceptance Review

```md
# Acceptance Review: <feature>

## Delivery Boundary

Staging release / production release / user handoff:

## Evidence Reviewed

- Brief:
- PRD / BDD:
- Test Strategy:
- Architecture Decision:
- Observability:
- Implementation Summary:
- Verification Results:
- Regression / Progress:

## Evidence Gaps

- Missing verification commands:
- Missing manual checks:
- Skipped checks and reasons:
- Unknown environment or data assumptions:

## Result

Decision: Release / Do not release / Needs user decision
Blocking gaps:
Only P0, security, data, production-failure, or user-accepted blocking criteria should block.
Accepted risks:
Follow-ups:
P1/P2 gaps without P0, security, or data risk may be accepted as follow-ups.
User decisions needed:
```

## Progress

```md
# Progress: <feature>

## Completed

-

## Confirmed Decisions

-

## Tests

-

## Regression Cases

-

## Observability Status

-

## Gaps / Blockers

-

## Next Smallest Task

-
```
