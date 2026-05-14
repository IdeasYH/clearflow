---
name: clearflow
description: Use when the user explicitly asks to use ClearFlow, their structured development workflow, Brief/PRD/BDD process, Test Strategy, Architecture Decision, Plan/Task Handoff, Pre-PR Review, Regression Capture, Progress recovery, or to guide a project through this workflow. Do not use for ordinary coding tasks unless the user explicitly asks for ClearFlow or this workflow.
---

# ClearFlow

ClearFlow guides a user through a structured software development workflow that uses clarity, standardization, tests, and observability to produce precise code and debuggable systems. It is a workflow coach and handoff layer, not a replacement for project instructions, superpowers, or normal coding judgment.

Core idea:

> Tests are executable requirements, logs are debugging evidence, and Progress is recovery context.

## Boundaries

- Use this skill only when the user explicitly asks for this workflow or one of its named stages.
- Do not force the full workflow for ordinary small changes.
- Do not override AGENTS.md, project conventions, or direct user instructions.
- Do not auto-enable multi-agent work, long-running loops, or external-model workflows.
- Do not duplicate superpowers implementation planning. Use this workflow to prepare better inputs for superpowers when available.

## Relationship With Superpowers

This workflow owns product behavior, test strategy, architecture decisions, observability, regression capture, and progress recovery.

Superpowers, when available and requested or appropriate, owns brainstorming discipline, TDD discipline, systematic debugging, detailed implementation plans, plan execution, code review, and branch finishing.

Recommended handoff:

```text
Brief / PRD / BDD / Test Strategy / Architecture Decision
  -> Plan / Task Handoff
  -> superpowers writing-plans
  -> Implementation
  -> Pre-PR Review / Regression Capture / Progress
```

## Default Workflow

Use this default sequence for feature work:

1. Brief
2. PRD / BDD
3. Test Strategy / Architecture Decision
4. Plan / Task Handoff
5. superpowers Implementation Plan
6. Implementation
7. Pre-PR Review / Regression Capture / Progress

For bug fixes, use the shorter sequence:

1. Brief
2. Test Strategy
3. Implementation
4. Regression Capture
5. Progress

For complex, risky, or multi-stage projects, expand the combined stages into:

1. Brief
2. PRD / BDD
3. Test Strategy
4. Architecture Decision
5. Plan / Task Handoff
6. superpowers Implementation Plan
7. Implementation
8. Pre-PR Review
9. Regression Capture
10. Progress

## Guided Mode

Guide the user stage by stage. At each stage:

1. Read available workflow artifacts before asking questions.
2. Convert the user's natural language into structured artifacts.
3. Surface ambiguity and risk.
4. Ask the user to make only the key decisions.
5. Record confirmed decisions in the relevant artifact.
6. State the next stage and required inputs.

Stop for user confirmation at these points:

- Brief scope and non-scope
- PRD / BDD behavior rules
- P0 test coverage
- Architecture direction
- Observability requirements for failure localization
- Plan / Task Handoff before generating a detailed superpowers plan
- Pre-PR Review result before merge or release

## Artifact Discovery

For repository work, prefer this index:

```text
docs/workflow/WORKFLOW.md
```

If it exists, read it first. It should point to current feature artifacts, test assets, regression cases, and observability rules.

If it does not exist and the user wants durable workflow artifacts, create it using `references/workflow-index-template.md`.

Recommended workflow paths:

```text
docs/workflow/WORKFLOW.md
docs/workflow/<feature-id>/brief.md
docs/workflow/<feature-id>/prd-bdd.md
docs/workflow/<feature-id>/test-strategy.md
docs/workflow/<feature-id>/architecture-decision.md
docs/workflow/<feature-id>/observability.md
docs/workflow/<feature-id>/plan-task-handoff.md
docs/workflow/<feature-id>/pre-pr-review.md
docs/workflow/<feature-id>/progress.md
tests/regression/
tests/fixtures/
```

Use the repository's existing documentation and test layout if it already has a clear convention.

## Stage Responsibilities

### Brief

Clarify:

- What is in scope
- What is out of scope
- Success criteria
- Key risks
- User decisions needed

Use superpowers brainstorming if the user wants deeper requirement discovery.

### PRD / BDD

Convert natural language into:

- User flow
- Success scenarios
- Failure scenarios
- Edge cases
- Given / When / Then scenarios
- Open questions

Do not invent business rules silently. Mark assumptions and ask the user to confirm them.

### Test Strategy / Architecture Decision

Generate and help the user choose:

- P0 / P1 / P2 test coverage
- Unit, integration, e2e, and regression test split
- Test data and fixtures
- Modules, data flow, dependency direction
- Required logs and diagnostic artifacts

For complex logic, prefer TDD. For bug fixes, reproduce first and capture a failing regression test when practical. For existing behavior changes, protect old behavior with tests before modifying it.

### Plan / Task Handoff

This is not a detailed code execution plan. It is the structured input for superpowers `writing-plans`.

Include:

- Goal
- Scope / non-scope
- Confirmed BDD scenarios
- P0 / P1 / P2 tests
- Architecture decisions
- Observability requirements
- Regression rules
- User-confirmed decisions
- Files or modules discovered so far, if known

If the user wants detailed execution, hand off to superpowers `writing-plans` and let it create the fine-grained implementation plan.

### Implementation

Follow the current Plan / Task Handoff or superpowers implementation plan.

Implementation must keep tests and observability aligned:

- New behavior should have tests.
- Complex logic should use Red / Green / Refactor.
- Bug fixes should include regression coverage when practical.
- Logs should identify the failing stage in multi-step workflows.

### Pre-PR Review

Review against the workflow, not only code style:

- Scope fulfilled
- Non-scope not accidentally implemented
- BDD scenarios covered
- P0 tests present and passing
- Architecture decisions respected
- Logs can locate failures by stage
- Relevant `error_code` and `debug_artifacts` exist
- Regression cases captured for fixed bugs
- Progress is updated

Use superpowers code review when available for code-level review, then apply this workflow review for product/test/observability/regression coverage.

### Regression Capture / Progress

For fixed bugs:

- Record reproduction data.
- Add or update regression tests.
- Mark the failing stage.
- Add a regression rule.

Progress must support recovery by another session or another agent:

- Completed work
- Confirmed decisions
- Existing tests and regression cases
- Known gaps
- Blockers
- Next smallest task

## Observability Standard

For async jobs, image generation, external APIs, database writes, background workers, or long-running workflows, help the user define:

- `job_id`
- `stage`
- `event`
- `error_code`
- `debug_artifacts`

The implementation is incomplete if a production failure cannot be localized to a concrete stage.

Ask:

```text
If this fails in production, what must we know within 3 minutes?
```

Then design logs and artifacts backwards from those questions.

## References

Load only the reference needed for the current stage:

- `references/workflow-index-template.md` for `docs/workflow/WORKFLOW.md`
- `references/stage-templates.md` for Brief, PRD / BDD, Test Strategy, Architecture Decision, Handoff, Review, and Progress templates
- `references/agent-md-snippet.md` for the optional AGENTS.md navigation rule
