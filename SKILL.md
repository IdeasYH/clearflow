---
name: clearflow
description: Use when the user explicitly asks to use ClearFlow, their structured development workflow, Brief/PRD/BDD process, Test Strategy, Test Strategy Review Agent, Unit Test Review Agent, Acceptance Review Agent, Architecture Decision, Plan/Task Handoff, Pre-PR Review, Regression Capture, Progress recovery, or to guide a project through this workflow. Do not use for ordinary coding tasks unless the user explicitly asks for ClearFlow or this workflow.
---

# ClearFlow

ClearFlow guides a user through a structured software development workflow that uses clarity, standardization, tests, and observability to produce precise code and debuggable systems. It is a workflow coach and handoff layer, not a replacement for project instructions, superpowers, or normal coding judgment.

Core idea:

> Tests are executable requirements, logs are debugging evidence, and Progress is recovery context.

## Boundaries

- Use this skill only when the user explicitly asks for this workflow or one of its named stages.
- Do not force the full workflow for ordinary small changes.
- Do not override AGENTS.md, project conventions, or direct user instructions.
- Do not auto-enable open-ended multi-agent work, long-running loops, or external-model workflows.
- Do not require review agents for ordinary small changes; scale review depth to risk, behavior surface, and release impact.
- Do not duplicate superpowers implementation planning. Use this workflow to prepare better inputs for superpowers when available.

## Relationship With Superpowers

This workflow owns product behavior, test strategy, architecture decisions, observability, regression capture, and progress recovery.

Superpowers, when available and requested or appropriate, owns brainstorming discipline, TDD discipline, systematic debugging, detailed implementation plans, plan execution, code review, and branch finishing.

ClearFlow may request bounded review-agent passes for test strategy, unit tests, and acceptance readiness. These agents are independent reviewers with narrow prompts and explicit outputs, not owners of implementation. Use them when available and when the task is complex, risky, user-facing, security/data-sensitive, regression-prone, or when the user asks for extra confidence. If subagents are unavailable, perform the same review as checklist sections in the main session.

Review agents are advisory by default. They may block only for P0 behavior gaps, security risk, data risk, production-failure risk, or user-accepted blocking criteria. They must identify gaps, risks, and open questions; they must not prescribe implementation steps or take over fixes.

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
4. Test Strategy Review, when risk justifies it
5. Plan / Task Handoff
6. superpowers Implementation Plan
7. Implementation
8. Unit Test Review, when unit-test changes are risk-relevant
9. Pre-PR Review / Acceptance Review / Regression Capture / Progress

For bug fixes, use the shorter sequence:

1. Brief
2. Test Strategy
3. Test Strategy Review, when the bug caused production failure or is regression-prone
4. Implementation
5. Unit Test Review, when unit-test changes are risk-relevant
6. Regression Capture
7. Progress

For complex, risky, or multi-stage projects, expand the combined stages into:

1. Brief
2. PRD / BDD
3. Test Strategy
4. Test Strategy Review, normally included unless the user chooses a lightweight path
5. Architecture Decision
6. Plan / Task Handoff
7. superpowers Implementation Plan
8. Implementation
9. Unit Test Review
10. Pre-PR Review
11. Acceptance Review
12. Regression Capture
13. Progress

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
- Test Strategy Review result when used
- Architecture direction
- Observability requirements for failure localization
- Plan / Task Handoff before generating a detailed superpowers plan
- Unit Test Review result when it identifies blocking gaps
- Pre-PR Review result before merge or release
- Acceptance Review result before release on complex or high-risk work

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
docs/workflow/<feature-id>/acceptance-review.md
docs/workflow/<feature-id>/progress.md
tests/regression/
tests/fixtures/
```

Use the repository's existing documentation and test layout if it already has a clear convention.

Artifact filenames are recommended, not mandatory. When existing repositories use different names, identify ClearFlow artifacts by this priority:

1. Links in `docs/workflow/WORKFLOW.md`.
2. Top-level heading, for example `# Test Strategy: <feature>`.
3. Required template sections.
4. Optional frontmatter such as `artifact_type: test-strategy`.

When creating new durable ClearFlow artifacts, use the recommended filenames unless the repository already has a clear convention. These rules apply only to ClearFlow artifacts; do not change or redefine superpowers plan files or formats.

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

- PRD draft, when the user wants a standard product-requirements document
- User flow
- Success scenarios
- Failure scenarios
- Edge cases
- Given / When / Then scenarios
- Open questions

If a `prd-writer` / `PRD-Writer` skill is installed or explicitly provided, use it only as a drafting helper for the PRD sections. ClearFlow remains authoritative for workflow stage boundaries, user confirmation, BDD scenarios, test strategy, architecture handoff, and release evidence.

Do not invent business rules silently. Mark assumptions and ask the user to confirm them.
Do not let a PRD-writing helper overwrite ClearFlow's required BDD or testing responsibilities. Convert confirmed PRD behavior into Given / When / Then scenarios before moving to Test Strategy.

### Test Strategy / Architecture Decision

Generate and help the user choose:

- P0 / P1 / P2 test coverage
- Unit, integration, e2e, and regression test split
- Test data and fixtures
- Modules, data flow, dependency direction
- Required logs and diagnostic artifacts

For complex logic, prefer TDD. For bug fixes, reproduce first and capture a failing regression test when practical. For existing behavior changes, protect old behavior with tests before modifying it.

Use a Test Strategy Review Agent when the test plan is high impact or uncertain. Ask it to check for:

- Missing P0 behavior from PRD / BDD.
- Overreliance on E2E where unit or integration tests would localize failures better.
- Missing failure, permission, data integrity, concurrency, idempotency, or rollback cases.
- Missing fixtures or unrealistic test data.
- Regression candidates that should be promoted to required tests.

The review output must be concise:

- Decision: Pass / Needs revision / Blocked pending user decision.
- Blocking gaps.
- Suggested P0 / P1 / P2 changes.
- Test type split corrections.
- Open questions requiring user decision.

Do not let the reviewer invent business requirements. It may only identify gaps, assumptions, and test coverage risks.
P1/P2 suggestions are advisory and may be recorded as follow-ups unless the user explicitly promotes them or they reveal a P0, security, or data risk.

### Plan / Task Handoff

This is not a detailed code execution plan. It is a ClearFlow handoff artifact: structured input for superpowers `writing-plans` or the currently available detailed planning workflow. ClearFlow does not define the internal format of the downstream implementation plan.

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

Use a Unit Test Review Agent when unit tests are added or changed for complex logic, critical behavior, production-failure fixes, regression bugs, or shared modules. Ask it to inspect the tests, not reimplement the feature. It should check:

- Tests assert behavior and business value, not private implementation details.
- Required unit-level P0/P1 coverage from the Test Strategy is present.
- Assertions are strong enough to fail on broken behavior, not only smoke checks or broad snapshots.
- Core branches, boundary values, parameter combinations, and error paths assigned to unit tests are covered.
- Tests would fail for the original bug or likely regression.
- Mocks and stubs do not make the test dishonest.
- Test names, fixtures, setup, and teardown make failures diagnosable and isolated.
- Async tests await the real work and do not leave hanging timers, unhandled promises, or hidden races.
- Tests are deterministic and do not depend on timing, order, network, or hidden global state unless explicitly controlled.

The reviewer should not require new unit tests when the Test Strategy intentionally assigns the behavior to integration, e2e, or manual verification. Treat missing required P0 coverage assigned to unit tests by the Test Strategy, security/data risk in test coverage, non-deterministic tests, or tests that cannot fail for the intended behavior as blocking until addressed or explicitly accepted by the user. P1/P2 gaps should be recorded as follow-ups unless they expose P0, security, or data risk.

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

Use an Acceptance Review Agent before release for complex, high-risk, user-facing, or multi-stage work. It decides whether the work can be released or delivered to users, not merely whether a PR can be merged. Give it the Brief, PRD / BDD, Test Strategy, Architecture Decision, Observability requirements, implementation summary, verification results, regression status, and progress status. Ask it to decide whether the work is releasable from a workflow perspective by reviewing evidence, not redoing implementation.

The Acceptance Review Agent must check:

- The delivery boundary is clear: PR merge, staging release, production release, or user handoff.
- Scope is fulfilled and non-scope was not accidentally implemented.
- Confirmed BDD scenarios have implementation and verification evidence.
- P0 tests are present, meaningful, passing, and tied to accepted behavior.
- Architecture decisions and dependency direction are respected, or deviations are documented and accepted.
- Required observability can localize failures by stage within the agreed diagnostic window.
- Fixed bugs have regression coverage and recorded regression rules.
- Verification commands, manual checks, skipped checks, environments, and known limitations are recorded.
- Remaining gaps are classified as blocking, accepted risk, or follow-up with owner or next action.

If the acceptance review finds blocking gaps, update the Plan / Task Handoff or Progress with the next smallest task before continuing.
If it finds only P1/P2 gaps without P0, security, or data risk, record them as follow-ups and allow release when the user accepts the residual risk.

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
