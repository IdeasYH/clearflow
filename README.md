# ClearFlow

[English](README.en.md)

ClearFlow 是一个面向 Codex 的引导式开发工作流 skill。它通过清晰范围、BDD、测试策略、架构决策、可观测性、实现交接、审查、回归沉淀和进度恢复，把模糊想法转成更精准的代码和更容易 debug 的系统。

核心思想：

```text
测试是可执行需求。
日志是定位证据。
Progress 是恢复上下文。
```

ClearFlow 不是编码能力、项目说明或 superpowers 的替代品。它是一个流程向导：帮助使用者做关键判断，并给 Agent 提供结构化产物。

## 什么时候使用

当你想做这些事情时，使用 ClearFlow：

- 从清晰范围开始规划新功能。
- 把自然语言需求转换成 PRD / BDD。
- 生成 P0 / P1 / P2 测试策略。
- 使用测试策略审查、单元测试审查和发布验收审查来查缺补漏。
- 在实现前设计日志和 debug artifacts。
- 为 superpowers 的详细实现计划准备交接材料。
- 在 PR 前从范围、测试、日志和回归角度审查 diff，并在发布前判断是否可交付给用户。
- 用复现、回归测试和进度恢复来修 Bug。

示例：

```text
$clearflow 带我从 Brief 开始做一个文生图功能。
```

另一个示例：

```text
$clearflow 把下面这个需求转成 PRD / BDD，并生成测试策略：
用户可以输入提示词生成一张图片，失败时要能定位失败阶段。
```

## 工作流

默认新功能流程：

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

Bug 修复流程：

```text
1. Brief
2. Test Strategy
3. Test Strategy Review, when the bug caused production failure or is regression-prone
4. Implementation
5. Unit Test Review, when unit-test changes are risk-relevant
6. Regression Capture
7. Progress
```

复杂项目可以展开组合阶段：

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

## 阶段 1：Brief

### 做什么

Brief 定义工作边界：

- 这个项目或功能要做什么。
- 不做什么。
- 成功标准是什么。
- 已知风险有哪些。
- 还有哪些决策需要使用者确认。

### 为什么做

AI 编码经常出问题，是因为范围模糊。Agent 可能多做、不做关键约束，或者偷偷做假设。Brief 在计划和编码前把边界写清楚。

### 示例

```md
# Brief: 文生图 MVP

## Scope

- 用户输入提示词。
- 系统生成一张图片。
- 系统保存图片 URL 和生成状态。
- 用户可以看到成功或失败结果。

## Non-Scope

- 多图生成。
- 图片编辑。
- 提示词历史。
- 付费额度。

## Success Criteria

- 有效提示词能生成一张图片。
- 生成失败时记录明确的失败阶段和错误码。
- P0 测试通过。

## Risks

- OpenAI API 失败。
- 图片保存失败。
- 数据库状态不一致。
```

提示词：

```text
$clearflow 帮我为文生图 MVP 写 Brief。请先问我必须确认的范围问题。
```

## 阶段 2：PRD / BDD

### 做什么

PRD / BDD 把自然语言转成可验证行为：

- 用户流程。
- 成功场景。
- 失败场景。
- 边界场景。
- Given / When / Then 场景。
- 未确认问题。

### 为什么做

自然语言需求经常含糊。BDD 让行为变得可测试、可审查，也让 Agent 在实现前有明确目标。

### 示例

```gherkin
Feature: 文生图

Scenario: 用户成功生成一张图片
  Given 用户提交了有效提示词
  When 图片生成任务成功完成
  Then 任务状态应为 succeeded
  And 生成图片 URL 应被保存

Scenario: OpenAI 图片生成失败
  Given 用户提交了有效提示词
  When OpenAI 图片 API 返回错误
  Then 任务状态应为 failed
  And failure stage 应为 openai_generation
  And 应记录 error_code
  And debug_artifacts 应包含 provider response summary
```

提示词：

```text
$clearflow 把这个需求转成 BDD。不要替我拍板业务规则，把需要确认的问题列出来。
```

## 阶段 3：Test Strategy / Architecture Decision

### 做什么

这个阶段定义如何证明行为正确，以及系统结构如何组织。

Test Strategy 决定：

- P0 测试：发布前必须通过。
- P1 测试：重要边界和失败覆盖。
- P2 测试：有价值但可后置。
- 单测 / 集成测试 / E2E / 回归测试怎么分。
- 测试数据和 fixtures。

Architecture Decision 决定：

- 模块边界。
- 数据流。
- 依赖方向。
- 状态和存储。
- 外部依赖边界。
- 取舍。

### 为什么做

测试让需求变成可执行证据。架构决策避免 Agent 把逻辑散落到无关文件里。这个阶段也告诉 Agent：什么被验证过，代码才算完成。

### 示例

```md
## P0 Tests

- 有效提示词创建 succeeded 任务，并保存一张图片 URL。
- OpenAI 失败时任务 failed，stage=openai_generation。
- 图片保存失败时任务 failed，stage=image_persistence。

## P1 Tests

- 空提示词在调用 OpenAI 前被拒绝。
- OpenAI 返回空图片列表。
- provider 成功后数据库写入失败。

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

提示词：

```text
$clearflow 基于这个 BDD 生成 Test Strategy，区分 P0/P1/P2，并说明哪些应该是单测、集成测试、回归测试。
```

## Review Agents

ClearFlow 可以在风险足够高时启用有边界的审查 Agent。它们默认是顾问，只能指出缺口、风险和开放问题，不能给实现步骤，也不能接管修复。

只有这些问题默认阻塞：

- P0 行为缺口。
- 安全风险。
- 数据风险。
- 生产故障风险。
- 用户明确接受为阻塞条件的问题。

P1 / P2 缺口默认记录为 follow-up，除非它暴露 P0、安全或数据风险。

### Test Strategy Review Agent

审查测试策略是否漏掉关键行为和风险：

- PRD / BDD 中的 P0 行为是否都有测试证据。
- 是否过度依赖 E2E，而没有用单测或集成测试定位失败。
- 是否漏掉失败路径、权限、数据完整性、并发、幂等或回滚。
- fixture 和测试数据是否真实。
- 哪些回归候选应该升级为必测。

### Unit Test Review Agent

审查单元测试是否可信，而不是重新实现功能：

- 是否验证行为和业务价值，而不是私有实现细节。
- 是否覆盖 Test Strategy 分配给 unit 层的 P0 / P1。
- 断言是否足够强，不只是 smoke test 或宽泛快照。
- mock / stub 是否让测试失真。
- 异步、定时器、全局状态、fixture 清理是否稳定。

### Acceptance Review Agent

Acceptance Review 不是普通 PR 合并检查。它用于发布前判断是否可以交付给用户，重点审查证据包：

- 交付边界是否明确：staging release、production release 或 user handoff。
- Brief、BDD、Test Strategy、Architecture Decision、Observability 是否都有对应验证证据。
- P0 测试是否存在、有效、通过，并绑定到已确认行为。
- 生产失败是否能按约定 stage 定位。
- 已修 bug 是否有回归覆盖和回归规则。
- 跳过的检查、人工验证、环境和已知限制是否记录。

## 阶段 4：Observability

Observability 可以属于 Architecture Decision，但对异步任务、外部 API、图片生成、数据库写入、长流程任务，应该单独明确。

### 做什么

定义：

- `job_id`
- `stage`
- `event`
- `error_code`
- `debug_artifacts`

### 为什么做

线上失败时，你需要快速定位问题。日志不是“多打印点信息”，而是 debug 证据。ClearFlow 会问：

```text
如果这个功能线上失败，3 分钟内我们必须知道什么？
```

然后从这个问题反推日志和调试产物设计。

### 示例

```md
## Stages

| Stage | Start Event | Success Event | Failure Event | Error Codes | Debug Artifacts |
|---|---|---|---|---|---|
| validation | generation.validation.started | generation.validation.passed | generation.validation.failed | PROMPT_EMPTY | request summary |
| openai_generation | generation.openai.started | generation.openai.succeeded | generation.openai.failed | OPENAI_ERROR, OPENAI_EMPTY_IMAGE_RESULT | provider response summary |
| image_persistence | generation.image_save.started | generation.image_save.succeeded | generation.image_save.failed | IMAGE_SAVE_FAILED | storage key, save error |
| database_write | generation.db_write.started | generation.db_write.succeeded | generation.db_write.failed | DB_WRITE_FAILED | job_id, intended state |
```

提示词：

```text
$clearflow 带我为这个异步图片生成流程设计 Observability，重点是 stage、event、error_code 和 debug_artifacts。
```

## 阶段 5：Plan / Task Handoff

### 做什么

Plan / Task Handoff 把已确认的工作流产物转成详细实现计划的输入。

它是 ClearFlow 的交接产物，不是详细 implementation plan。ClearFlow 不定义 superpowers plan 的内部格式，也不修改 superpowers 的计划规则。

它包含：

- Goal。
- Scope / non-scope。
- 已确认 BDD 场景。
- P0 / P1 / P2 测试。
- 架构决策。
- 可观测性要求。
- 回归规则。
- 用户已确认决策。
- 已知文件或模块。

### 为什么做

ClearFlow 不重复 superpowers `writing-plans`。superpowers plan 更细，更接近代码级执行。ClearFlow 的 handoff 告诉 superpowers 必须保留什么：行为、测试、架构、日志和回归规则。

### 示例

```md
# Plan / Task Handoff: 文生图 MVP

## Goal

实现单图文生图，并能可靠定位失败阶段。

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

提示词：

```text
$clearflow 基于已确认的 Brief、BDD、测试策略和架构决策，生成 Plan / Task Handoff，准备交给 superpowers writing-plans。
```

## 阶段 6：superpowers Implementation Plan

### 做什么

这个可选阶段把 ClearFlow 产物交给 superpowers `writing-plans`，生成更细的实现计划。

superpowers plan 通常包括：

- 要创建或修改的准确文件。
- 失败测试。
- 最小实现步骤。
- 要运行的命令。
- 预期测试输出。
- commit 粒度任务。

### 为什么做

ClearFlow 定义“什么必须成立”。superpowers 定义“如何一步一步实现”。分开这两层，可以避免重复计划，并让两个工作流互相增强。

提示词：

```text
Use superpowers writing-plans with this Plan / Task Handoff and the referenced ClearFlow artifacts.
Create a detailed implementation plan.
```

## 阶段 7：Implementation

### 做什么

Implementation 按 Plan / Task Handoff 或 superpowers implementation plan 执行。

规则：

- 新行为应该有测试。
- 复杂逻辑应该用 TDD。
- Bug 修复应先复现，并在可行时加入回归覆盖。
- 多阶段流程的日志应该能定位失败 stage。

### 为什么做

Implementation 是实际改代码的阶段，但它不能自己发明范围，也不能悄悄改变行为。前面的产物提供约束和证据。

提示词：

```text
$clearflow 进入 Implementation 阶段。请先读取 Plan / Task Handoff、Test Strategy 和 Observability，再说明将执行哪个最小任务。
```

## 阶段 8：Pre-PR Review

### 做什么

Pre-PR Review 按 ClearFlow 检查改动，而不只是看代码风格。

它验证：

- Scope 是否完成。
- Non-scope 是否被误做。
- BDD 场景是否覆盖。
- P0 测试是否存在并通过。
- 架构决策是否被遵守。
- 日志是否能按 stage 定位失败。
- 相关 error_code 和 debug_artifacts 是否存在。
- 修复过的 Bug 是否沉淀为回归用例。
- Progress 是否更新。

### 为什么做

一个功能可能“看起来能跑”，但仍然难以 debug，或者以后很容易改坏。Pre-PR Review 在合并前捕获缺失测试、缺失日志、范围蔓延和未沉淀回归。

提示词：

```text
$clearflow 对当前 diff 做 Pre-PR Review，重点检查范围、BDD、P0 测试、日志定位、回归用例和 Progress。
```

## 阶段 9：Regression Capture

### 做什么

对 Bug，Regression Capture 把一次失败转成可复用资产。

它记录：

- 复现数据。
- 失败 stage。
- 新增或更新的测试 / fixture。
- 回归规则。
- 相关 error_code 或 debug artifact。

### 为什么做

修 Bug 但不沉淀回归测试，意味着同类 Bug 可能再次回来。Regression Capture 把一次失败变成长期护栏。

### 示例

```md
## Regression Rule

When OpenAI returns a successful response with an empty images array:

- job status must be failed
- stage must be openai_generation
- error_code must be OPENAI_EMPTY_IMAGE_RESULT
- system must not save an empty image URL
```

提示词：

```text
$clearflow 这是刚修复的 bug，请帮我做 Regression Capture：复现数据、失败 stage、回归测试位置、Regression rule。
```

## 阶段 10：Progress

### 做什么

Progress 记录足够恢复上下文的信息。

它包括：

- 已完成工作。
- 已确认决策。
- 现有测试。
- 回归用例。
- 可观测性状态。
- 缺口和阻塞。
- 下一个最小任务。

### 为什么做

Progress 让另一个会话或另一个 Agent 可以继续做，不必重新发现所有上下文。它是长任务的恢复上下文。

提示词：

```text
$clearflow 更新 Progress，记录已完成内容、已确认决策、测试资产、回归 case、日志缺口和下一步最小任务。
```

## 推荐项目文件

如果项目需要长期保存 ClearFlow 产物，推荐使用：

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

`docs/workflow/WORKFLOW.md` 是索引。Agent 在处理结构化工作流任务时应该先读取它。

文件名是推荐，不是硬性要求。如果项目已有其他命名习惯，按以下顺序识别 ClearFlow 产物：

1. `docs/workflow/WORKFLOW.md` 中的链接。
2. 一级标题，例如 `# Test Strategy: <feature>`。
3. 模板中的必需章节。
4. 可选 frontmatter，例如 `artifact_type: test-strategy`。

这些识别规则只适用于 ClearFlow 产物，不改变 superpowers 的 plan 文件或格式。

## 可选 AGENTS.md 规则

如果仓库使用长期保存的 ClearFlow 产物，可以在仓库级 `AGENTS.md` 加一个简短导航规则：

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

## 常用提示词

开始新功能：

```text
$clearflow 带我从 Brief 开始规划这个功能：...
```

需求转 BDD：

```text
$clearflow 把下面自然语言需求转成 PRD / BDD，并列出需要我确认的问题：...
```

生成测试策略：

```text
$clearflow 基于这个 PRD / BDD 生成 Test Strategy，区分 P0/P1/P2，并建议测试类型和测试数据。
```

设计可观测性：

```text
$clearflow 帮我设计这个流程的 job_id、stage、event、error_code、debug_artifacts。
```

准备 superpowers 交接：

```text
$clearflow 生成 Plan / Task Handoff，准备交给 superpowers writing-plans。
```

PR 前审查：

```text
$clearflow 对当前 diff 做 Pre-PR Review。
```

沉淀回归：

```text
$clearflow 根据这个 bug 修复生成 Regression Capture 和 Progress 更新。
```

## 设计边界

ClearFlow 应该保持窄边界：

- 它引导流程。
- 它帮助结构化决策。
- 它准备工作流产物。
- 它检查测试、可观测性、回归和进度。
- 它不替代项目说明。
- 它不替代 superpowers 的详细实现计划。
- 它不为小任务强行套重流程。
