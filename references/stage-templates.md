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

## Result

Decision:
Blocking issues:
Follow-ups:
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
