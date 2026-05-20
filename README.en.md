# ClearFlow

ClearFlow is a guided development workflow skill for Codex. It helps you turn unclear ideas into precise code and debuggable systems through clear scope, BDD, test strategy, architecture decisions, observability, implementation handoff, review, regression capture, and progress recovery.

Core idea:

```text
Tests are executable requirements.
Logs are debugging evidence.
Progress is recovery context.
```

ClearFlow is not a replacement for coding skills, project instructions, or superpowers. It is a workflow guide that helps the user make key decisions and gives the agent structured artifacts to work from.

## When To Use

Use ClearFlow when you want to:

- Start a new feature with clear scope.
- Convert natural language requirements into PRD / BDD.
- Generate P0 / P1 / P2 test strategy.
- Use test strategy review, unit test review, and release acceptance review to find gaps.
- Design logs and debug artifacts before implementation.
- Prepare a handoff for a detailed superpowers implementation plan.
- Review a diff before PR with scope, tests, logs, and regression in mind, and decide before release whether the work can be delivered to users.
- Fix bugs with reproduction, regression tests, and progress recovery.

Example:

```text
$clearflow 带我从 Brief 开始做一个文生图功能。
```

Another example:

```text
$clearflow 把下面这个需求转成 PRD / BDD，并生成测试策略：
用户可以输入提示词生成一张图片，失败时要能定位失败阶段。
```

## Workflow

Default feature workflow:

```text
1. Brief
2. PRD / BDD
3. Test Strategy / Architecture Decision
4. Test Strategy Review, when risk justifies it
5. Plan / Task Handoff
6. superpowers Implementation Plan
7. Implementation
8. Unit Test Review, when unit-test changes are risk-relevant
9. Pre-PR Review / Acceptance Review / Regression Capture / Progress
```

Bug fix workflow:

```text
1. Brief
2. Test Strategy
3. Test Strategy Review, when the bug caused production failure or is regression-prone
4. Implementation
5. Unit Test Review, when unit-test changes are risk-relevant
6. Regression Capture
7. Progress
```

For complex projects, ClearFlow can expand the combined stages:

```text
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
```

## Stage 1: Brief

### What It Does

Brief defines the work boundary:

- What this project or feature will do.
- What it will not do.
- What success means.
- What risks are known.
- Which decisions still need the user.

### Why It Exists

AI coding goes wrong when scope is vague. The agent may implement extra behavior, skip important constraints, or make hidden assumptions. Brief prevents this by making the work boundary explicit before planning or coding.

### Example

```md
# Brief: Text-to-Image MVP

## Scope

- User enters a prompt.
- System creates one image.
- System stores the image URL and generation status.
- User can see success or failure result.

## Non-Scope

- Multi-image generation.
- Image editing.
- Prompt history.
- Payment limits.

## Success Criteria

- A valid prompt creates one image.
- Failed generation records a clear failure stage and error code.
- P0 tests pass.

## Risks

- OpenAI API failure.
- Image save failure.
- Database state inconsistency.
```

Prompt:

```text
$clearflow 帮我为文生图 MVP 写 Brief。请先问我必须确认的范围问题。
```

## Stage 2: PRD / BDD

### What It Does

PRD / BDD turns natural language into behavior:

- Standard PRD draft, when the user needs product-requirements documentation first.
- User flow.
- Success scenarios.
- Failure scenarios.
- Edge cases.
- Given / When / Then scenarios.
- Open questions.

ClearFlow includes a built-in PRD draft assistant. When standard product-requirements documentation is needed, use it directly inside the PRD / BDD stage; it does not depend on a separately installed `prd-writer` / `PRD-Writer` skill. It only helps structure PRD content; it does not replace ClearFlow confirmation, BDD scenarios, test strategy, architecture handoff, or release evidence.

### PRD Draft Assistant

What it does:

- Turns the Brief, raw user notes, business context, and constraints into a standard PRD draft.
- Fills the basic PRD structure first: problem / goal, users / personas, scope, non-scope, requirements, user flow, success scenarios, failure scenarios, edge cases, and open questions.
- Marks uncertain content as assumptions or questions instead of turning it into confirmed rules.

Why it exists:

- Early requirements are often fragmented. Going straight to BDD can miss why the feature exists, who it serves, what is in scope, and what is out of scope.
- The PRD draft clarifies product meaning before ClearFlow confirmation and BDD conversion, reducing silent business-rule invention by the agent.
- It separates document drafting from behavior acceptance: PRD explains product intent, while BDD verifies confirmed behavior.

What it improves:

- The user can review a fuller PRD draft before answering scattered requirement questions.
- BDD scenarios come from confirmed PRD behavior, making the later Test Strategy easier to trace back to real requirements.
- Plan / Task Handoff gets clearer scope, non-scope, and open questions, reducing implementation-stage omissions, overreach, and accidental scope expansion.

### Why It Exists

Natural language requirements are often ambiguous. BDD makes behavior testable and reviewable. It also gives the agent a precise target before implementation.

PRD explains why the feature exists, who it serves, what is in scope, and what is out of scope. BDD converts confirmed PRD behavior into testable acceptance scenarios. They do not conflict; PRD-Writer can draft the PRD, but it must not skip BDD.

### Example

```gherkin
Feature: Text-to-image generation

Scenario: User generates one image successfully
  Given the user submits a valid prompt
  When the image generation job completes successfully
  Then the job status should be succeeded
  And the generated image URL should be saved

Scenario: OpenAI image generation fails
  Given the user submits a valid prompt
  When the OpenAI image API returns an error
  Then the job status should be failed
  And the failure stage should be openai_generation
  And an error_code should be recorded
  And debug_artifacts should include the provider response summary
```

Prompt:

```text
$clearflow 把这个需求转成 BDD。不要替我拍板业务规则，把需要确认的问题列出来。
```

## Stage 3: Test Strategy / Architecture Decision

### What It Does

This stage defines how the behavior will be proven and how the system will be shaped.

Test Strategy decides:

- P0 tests: must pass before release.
- P1 tests: important failure and edge coverage.
- P2 tests: useful but deferrable.
- Unit / integration / e2e / regression split.
- Test data and fixtures.

Architecture Decision decides:

- Modules.
- Data flow.
- Dependency direction.
- State and storage.
- External dependency boundaries.
- Tradeoffs.

### Why It Exists

Tests make requirements executable. Architecture decisions prevent the agent from scattering logic across unrelated files. This stage also tells the agent what must be verified before code is considered done.

### Example

```md
## P0 Tests

- Valid prompt creates a succeeded job with one image URL.
- OpenAI failure marks the job failed with stage=openai_generation.
- Image save failure marks the job failed with stage=image_persistence.

## P1 Tests

- Empty prompt is rejected before calling OpenAI.
- OpenAI returns an empty image list.
- Database write fails after provider success.

## Test Type Split

Unit:
- prompt validation
- provider response parsing
- job state transition

Integration:
- generation service with mocked OpenAI client and test database

Regression:
- provider returns success response with empty images array
```

Prompt:

```text
$clearflow 基于这个 BDD 生成 Test Strategy，区分 P0/P1/P2，并说明哪些应该是单测、集成测试、回归测试。
```

## Review Agents

ClearFlow can use bounded review agents when risk justifies the extra pass. They are advisory by default: they may identify gaps, risks, and open questions, but they must not prescribe implementation steps or take over fixes.

Only these issues block by default:

- P0 behavior gaps.
- Security risk.
- Data risk.
- Production-failure risk.
- Issues the user explicitly accepts as blocking criteria.

P1 / P2 gaps are recorded as follow-ups by default unless they reveal P0, security, or data risk.

### Test Strategy Review Agent

Reviews whether the test strategy misses critical behavior or risk:

- P0 behavior from PRD / BDD has test evidence.
- The plan does not over-rely on E2E when unit or integration tests would localize failures better.
- Failure paths, permissions, data integrity, concurrency, idempotency, and rollback are considered.
- Fixtures and test data are realistic.
- Regression candidates that should become required tests are identified.

### Unit Test Review Agent

Reviews whether unit tests are trustworthy without reimplementing the feature:

- Tests assert behavior and business value, not private implementation details.
- Unit-level P0 / P1 coverage assigned by the Test Strategy is present.
- Assertions are strong enough to fail on broken behavior, not only smoke checks or broad snapshots.
- Mocks and stubs do not make the tests dishonest.
- Async work, timers, global state, and fixture cleanup are deterministic.

### Acceptance Review Agent

Acceptance Review is not a normal PR merge check. It decides before release whether the work can be delivered to users by reviewing the evidence package:

- The delivery boundary is clear: staging release, production release, or user handoff.
- Brief, BDD, Test Strategy, Architecture Decision, and Observability have corresponding verification evidence.
- P0 tests are present, meaningful, passing, and tied to accepted behavior.
- Production failures can be localized by the agreed stage.
- Fixed bugs have regression coverage and regression rules.
- Skipped checks, manual verification, environments, and known limitations are recorded.

## Stage 4: Observability

Observability can be part of Architecture Decision, but for async jobs, external APIs, image generation, database writes, and long-running workflows, it should be explicit.

### What It Does

It defines:

- `job_id`
- `stage`
- `event`
- `error_code`
- `debug_artifacts`

### Why It Exists

When production fails, you need to locate the failure quickly. Logs are not just text output; they are debugging evidence. ClearFlow asks:

```text
If this fails in production, what must we know within 3 minutes?
```

Then it designs logs backwards from that question.

### Example

```md
## Stages

| Stage | Start Event | Success Event | Failure Event | Error Codes | Debug Artifacts |
|---|---|---|---|---|---|
| validation | generation.validation.started | generation.validation.passed | generation.validation.failed | PROMPT_EMPTY | request summary |
| openai_generation | generation.openai.started | generation.openai.succeeded | generation.openai.failed | OPENAI_ERROR, OPENAI_EMPTY_IMAGE_RESULT | provider response summary |
| image_persistence | generation.image_save.started | generation.image_save.succeeded | generation.image_save.failed | IMAGE_SAVE_FAILED | storage key, save error |
| database_write | generation.db_write.started | generation.db_write.succeeded | generation.db_write.failed | DB_WRITE_FAILED | job_id, intended state |
```

Prompt:

```text
$clearflow 带我为这个异步图片生成流程设计 Observability，重点是 stage、event、error_code 和 debug_artifacts。
```

## Stage 5: Plan / Task Handoff

### What It Does

Plan / Task Handoff converts confirmed workflow artifacts into input for a detailed implementation plan.

It is a ClearFlow handoff artifact, not the detailed implementation plan. ClearFlow does not define the internal format of superpowers plans or modify superpowers planning rules.

It includes:

- Goal.
- Scope / non-scope.
- Confirmed BDD scenarios.
- P0 / P1 / P2 tests.
- Architecture decisions.
- Observability requirements.
- Regression rules.
- User-confirmed decisions.
- Known files or modules.

### Why It Exists

ClearFlow does not try to duplicate superpowers `writing-plans`. Superpowers plans are more detailed and closer to code-level task execution. ClearFlow's handoff tells superpowers what must be preserved: behavior, tests, architecture, logs, and regression rules.

### Example

```md
# Plan / Task Handoff: Text-to-Image MVP

## Goal

Implement one-image text generation with reliable failure localization.

## Scope / Non-Scope

Scope:
- prompt validation
- image generation job
- OpenAI provider call
- image URL persistence
- failure stage and error code recording

Non-scope:
- multi-image generation
- image editing
- prompt history

## Confirmed BDD Scenarios

- valid prompt succeeds with one image URL
- OpenAI failure records stage=openai_generation
- image save failure records stage=image_persistence

## Observability Requirements

- every job must have job_id
- every failure must record stage and error_code
- OpenAI failures must preserve provider response summary as debug_artifacts

## Handoff to superpowers writing-plans

Use this document plus Brief, PRD/BDD, Test Strategy, and Architecture Decision to create a file-level implementation plan with failing tests, commands, and expected results.
```

Prompt:

```text
$clearflow 基于已确认的 Brief、BDD、测试策略和架构决策，生成 Plan / Task Handoff，准备交给 superpowers writing-plans。
```

## Stage 6: superpowers Implementation Plan

### What It Does

This optional stage hands the ClearFlow artifacts to superpowers `writing-plans`, which can produce a fine-grained implementation plan.

Superpowers plans usually include:

- Exact files to create or modify.
- Failing tests.
- Minimal implementation steps.
- Commands to run.
- Expected test output.
- Commit-sized tasks.

### Why It Exists

ClearFlow defines what must be true. Superpowers defines exactly how to implement it step by step. Keeping these separate prevents duplicate planning and makes both workflows stronger.

Prompt:

```text
Use superpowers writing-plans with this Plan / Task Handoff and the referenced ClearFlow artifacts.
Create a detailed implementation plan.
```

## Stage 7: Implementation

### What It Does

Implementation follows the Plan / Task Handoff or the superpowers implementation plan.

Rules:

- New behavior should have tests.
- Complex logic should use TDD.
- Bug fixes should reproduce first and add regression coverage when practical.
- Logs should identify the failing stage in multi-step workflows.

### Why It Exists

Implementation is where the code changes, but it should not invent scope or silently change behavior. The earlier artifacts provide constraints and evidence.

Prompt:

```text
$clearflow 进入 Implementation 阶段。请先读取 Plan / Task Handoff、Test Strategy 和 Observability，再说明将执行哪个最小任务。
```

## Stage 8: Pre-PR Review

### What It Does

Pre-PR Review checks the change against ClearFlow, not only code style.

It verifies:

- Scope fulfilled.
- Non-scope not accidentally implemented.
- BDD scenarios covered.
- P0 tests present and passing.
- Architecture decisions respected.
- Logs can locate failures by stage.
- Relevant error codes and debug artifacts exist.
- Regression cases captured for fixed bugs.
- Progress is updated.

### Why It Exists

A feature can look correct but still be hard to debug or unsafe to change later. Pre-PR Review catches missing tests, missing logs, hidden scope creep, and missing regression capture before the work is merged.

Prompt:

```text
$clearflow 对当前 diff 做 Pre-PR Review，重点检查范围、BDD、P0 测试、日志定位、回归用例和 Progress。
```

## Stage 9: Regression Capture

### What It Does

For bugs, Regression Capture turns the failure into a reusable asset.

It records:

- Reproduction data.
- Failing stage.
- Test or fixture added.
- Regression rule.
- Related error code or debug artifact.

### Why It Exists

Fixing a bug without capturing a regression test means the same bug can return. Regression Capture converts a one-time failure into a permanent guardrail.

### Example

```md
## Regression Rule

When OpenAI returns a successful response with an empty images array:

- job status must be failed
- stage must be openai_generation
- error_code must be OPENAI_EMPTY_IMAGE_RESULT
- system must not save an empty image URL
```

Prompt:

```text
$clearflow 这是刚修复的 bug，请帮我做 Regression Capture：复现数据、失败 stage、回归测试位置、Regression rule。
```

## Stage 10: Progress

### What It Does

Progress records enough context for recovery.

It includes:

- Completed work.
- Confirmed decisions.
- Existing tests.
- Regression cases.
- Observability status.
- Gaps and blockers.
- Next smallest task.

### Why It Exists

Progress lets another session or agent continue without rediscovering everything. It is the recovery context for long-running work.

Prompt:

```text
$clearflow 更新 Progress，记录已完成内容、已确认决策、测试资产、回归 case、日志缺口和下一步最小任务。
```

## Recommended Repository Files

For durable project workflows, ClearFlow uses:

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

`docs/workflow/WORKFLOW.md` is the index. Agents should read it first when working on structured workflow tasks.

Filenames are recommended, not mandatory. If a project already has another naming convention, identify ClearFlow artifacts in this order:

1. Links in `docs/workflow/WORKFLOW.md`.
2. Top-level heading, for example `# Test Strategy: <feature>`.
3. Required template sections.
4. Optional frontmatter, for example `artifact_type: test-strategy`.

These identity rules apply only to ClearFlow artifacts. They do not change superpowers plan files or formats.

## Optional AGENTS.md Rule

If a repository uses durable ClearFlow artifacts, add a short navigation rule to its `AGENTS.md`:

```md
## Workflow Artifacts

When working on structured workflow tasks, first read:

- `docs/workflow/WORKFLOW.md`

This file is the index for active features, workflow artifacts, stage status, test assets, regression cases, and observability rules.

Do not assume workflow artifact paths. Read `docs/workflow/WORKFLOW.md` first.

For implementation plans generated by superpowers, check:

- `docs/superpowers/plans/`

Workflow artifacts and superpowers plans serve different purposes:

- `docs/workflow/` defines product behavior, test strategy, architecture decisions, observability, regression capture, and progress.
- `docs/superpowers/plans/` contains detailed implementation plans for execution.
```

## Common Usage Prompts

Start a feature:

```text
$clearflow 带我从 Brief 开始规划这个功能：...
```

Convert requirements to BDD:

```text
$clearflow 把下面自然语言需求转成 PRD / BDD，并列出需要我确认的问题：...
```

Generate test strategy:

```text
$clearflow 基于这个 PRD / BDD 生成 Test Strategy，区分 P0/P1/P2，并建议测试类型和测试数据。
```

Design observability:

```text
$clearflow 帮我设计这个流程的 job_id、stage、event、error_code、debug_artifacts。
```

Prepare superpowers handoff:

```text
$clearflow 生成 Plan / Task Handoff，准备交给 superpowers writing-plans。
```

Review before PR:

```text
$clearflow 对当前 diff 做 Pre-PR Review。
```

Capture regression:

```text
$clearflow 根据这个 bug 修复生成 Regression Capture 和 Progress 更新。
```

## Design Boundary

ClearFlow should stay narrow:

- It guides the workflow.
- It helps structure decisions.
- It prepares artifacts.
- It checks tests, observability, regression, and progress.
- It does not replace project instructions.
- It does not replace superpowers detailed implementation plans.
- It does not force heavy process for small tasks.
