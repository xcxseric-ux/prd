---
name: prd
description: |-
  PRD 生成与逆向工程（Stage 3 可执行层）——前向：想法→结构化可执行 PRD；反向：代码库→逆向 PRD。
  每个 User Story 附带 agent_story 子块（files_to_touch + commands_to_run + done_when + verification），
  使 AI agent 可自主读取、执行、验证，无需人工推断。内置双质量门禁（结构+语义+可执行性）。
  触发：/prd、/prd-forward、/prd-reverse、"写个PRD"、"生成需求文档"、"逆向PRD"、"从代码生成PRD"。
version: 2.0.0
author: omcharness
license: MIT
metadata:
  hermes:
    tags: [prd, product, requirements, spec, planning, reverse-engineering]
    related_skills: [plan, writing-plans, ralplan, team]
    related_agents: [product-manager, architect, planner]
---

# PRD Skill — 产品需求文档生成与逆向工程

> **方法论来源**：综合 BHIL AI-First Toolkit（结构+AI-native）、Vibe Coding PRD Template（交互哲学）、
> Nexus Harness（质量门禁+状态机）、omcharness 内置 product-manager agent（PM 领域知识）。
> 所有设计决策均有出处标注，如 `[BHIL]` `[VIBE]` `[NEXUS]` `[OMC]`。

---

## 1. 核心哲学（5 条铁律）

### 1.1 先追问，再落笔 `[VIBE]`

**拿到需求后第一件事不是写 PRD，是问问题。** 不问清楚就写的 PRD 是 AI 的幻觉——看起来专业，用起来空洞。

最少覆盖：
- 目标用户是谁？（不是"用户"，是具体角色）
- 解决什么问题？（不是"提高效率"，是可验证的痛点）
- 怎么算成功？（不是"用户满意"，是可测量的指标）
- 明确不做什么？（防止范围蔓延）

### 1.2 目标读者是初级开发者 `[VIBE]`

PRD 不是给 PM 看的故事，是给实现者看的**可执行契约**。每条需求都必须能让一个不熟悉业务的人理解并实现。

禁止词： "优化体验"、"提升性能"、"做得更好"——没有量化数字就不是需求。

### 1.3 PRD 是 artifacts 链的起点，不是终点 `[BHIL]`

```
PRD (what) → TRD/Spec (how) → ADR (why) → Tasks (steps) → Code → Review → Deploy
```

每个 artifact 通过 YAML frontmatter 的 `id` / `parent` / `children` 双向链接。PRD 不包含实现细节（那是 Spec 的事），但必须有足够信息让 Spec 可生成。

**artifact 链自动填充** `[STAGE-3]`：当 PRD 的 `children` 字段为空时，下游 ralplan/spec 生成 SPEC-NNN 后应自动回写 PRD frontmatter：
```yaml
children:
  - SPEC-042    # ralplan 生成技术方案时自动回写
  - TASK-042    # spec 拆任务时自动回写
```
PRD 永远知道自己被哪些下游 artifact 引用。不依赖人工维护。

### 1.4 质量门禁不可跳过 `[NEXUS]`

PRD 落盘前必须通过两道门禁：
1. **结构门禁**（机械）：必需章节齐全、frontmatter 完整、无占位符
2. **语义门禁**（子 Agent 审查）：独立 reviewer 逐项验证

任何一道不通过 → 修复 → 重检，不许跳过。

### 1.5 老项目先逆向，再迭代 `[OMC]`

对已有代码库，永远不要从空白 PRD 开始。"逆向模式"从代码/架构/文档中提取现状 PRD，然后在此基础上做增量。

---

## 2. PRD 标准结构（15 节）

以下为经过三大框架交叉验证的**最小完备 PRD 结构**。标注 `[BHIL]` `[VIBE]` `[NEXUS]` `[OMC]` 表示各框架的贡献。

### Frontmatter（YAML）`[BHIL]`

```yaml
---
id: PRD-NNN
title: "[Feature name — 5 words max]"
status: draft              # draft | in-review | approved | implemented | superseded
created: YYYY-MM-DD
updated: YYYY-MM-DD
author: [Name]
sprint: S-NN
priority: high             # high | medium | low
tags: [tag1, tag2]
children: []               # SPEC-NNN, etc. — populated downstream
adrs: []                   # ADR-NNN — populated as decisions are made
mode: forward | reverse    # [OMC] 新增：标注生成方向
---
```

### §1 — Executive Summary `[OMC]` (综合 BHIL + VIBE)

**一句话**描述产品 + **一段话**（≤5 句）描述核心价值。读者 30 秒内能判断"这东西跟我有没有关系"。

### §2 — Problem Statement `[BHIL]`

```
格式：[User type] cannot [accomplish goal] because [root cause].

证据：
- 用户研究：[访谈发现, n=X]
- 行为数据：[指标显示问题]
- 支持工单：[ticket 量/主题]
- 竞品信号：[竞品做了什么/没做什么]
```

### §3 — Solution Overview `[OMC]` (综合 VIBE + BHIL)

- **方案描述**（2-4 段）：怎么解决问题
- **核心设计决策**：选 A 不选 B 的理由 + 权衡
- **What This Is NOT**（≥3 条显式排除）：防止范围蔓延和 AI 幻觉 `[VIBE]`

### §4 — Target Users & Personas `[OMC]` (综合 BHIL + VIBE)

| 角色 | 场景 | 痛点 | 使用频率 |
|------|------|------|---------|

每个角色有名字和上下文，不是"用户 A"。

### §5 — User Stories（EARS 格式 + Agent Story 可执行块）`[BHIL]` + `[STAGE-3]`

每条 User Story 由两部分组成：**人类可读的故事**（EARS 格式）+ **机器可执行的 agent_story**。

```
**US-001:** [一句话标题]
WHEN [trigger], the system SHALL [response].

Acceptance Criteria (人类可读):
  ✅ GIVEN [context], WHEN [action], THEN [expected outcome]
  ✅ GIVEN [edge case], WHEN [action], THEN [fallback behavior]
```

**agent_story 块**（每条 US 必含）`[STAGE-3]`：

```yaml
agent_story:
  id: "US-001"
  title: "[与上文一致]"
  context: |
    [1-2 句：本故事涉及的系统上下文。Agent 读完后知道"我现在在哪个子系统里"。]
    示例：消息处理入口在 gateway/platforms/feishu.py，投递路由在 gateway/delivery.py。

  files_to_touch:
    - path/to/module.py          # [为什么改这个文件 — 一行]
    - path/to/another.py         # [为什么改这个文件 — 一行]

  files_to_NOT_touch:            # [显式禁止碰的文件 — 防止过度修改]
    - path/to/do_not_touch.py    # [原因]
    - path/to/stable_api.py      # [原因：稳定接口，改了会破坏下游]

  commands_to_run:
    - "pytest tests/path/test_x.py -v -k test_name"
    - "pytest tests/path/test_y.py -v"
    # 每个命令必须是可直接复制粘贴运行的

  done_when:
    - "条件 1：可机器验证的判定"
    - "条件 2：可机器验证的判定"
    # 每个条件必须是二值的（过/不过），不需要人工判断

  verification:
    type: integration_test | unit_test | e2e_test | manual | metric
    metric: "[指标名 — 仅 type=metric 时填写]"
    threshold: "[阈值 — 仅 type=metric 时填写，如 'P95 < 5000ms']"
    measurement: "[测量方法 — 仅 type=metric 时填写，如 'grep e2e_latency gateway.log | tail -100']"
    eval_config: "[eval 配置文件路径 — 仅 AI/LLM 功能填写，如 'evals/us-001-factuality.yaml']"
```

**完整示例**：

```
**US-003:** 飞书消息桥接到微信
WHEN 用户在微信回复飞书消息，系统 SHALL 将回复内容桥接到飞书群。

Acceptance Criteria:
  ✅ GIVEN 飞书消息已投递到微信，WHEN 用户在微信回复，THEN 回复内容出现在飞书群
  ✅ GIVEN 回复含图片，WHEN 微信发送图片，THEN 飞书群收到同一图片

agent_story:
  id: "US-003"
  title: "飞书消息桥接到微信"
  context: |
    微信适配器在 gateway/platforms/wechat.py，跨平台桥接逻辑在 gateway/bridge.py。
    飞书端只负责接收桥接消息，不处理桥接方向判断。

  files_to_touch:
    - gateway/platforms/wechat.py      # 微信消息接收 → 调 bridge.send_to_feishu()
    - gateway/bridge.py                # 桥接方向判断 + 消息格式转换

  files_to_NOT_touch:
    - gateway/platforms/feishu.py      # 飞书端只做被动接收，不感知桥接方向
    - gateway/delivery.py              # 投递路由不变

  commands_to_run:
    - "pytest tests/gateway/test_bridge.py -v -k wechat_to_feishu"
    - "pytest tests/gateway/test_bridge.py -v -k image_passthrough"
    - "pytest tests/gateway/test_wechat.py -v"

  done_when:
    - "test_bridge.py:test_wechat_reply_bridges_to_feishu PASS"
    - "test_bridge.py:test_image_passthrough PASS"
    - "test_wechat.py 全量 PASS"
    - "回环检测：同一消息不重复桥接"

  verification:
    type: integration_test
```

对于 AI/LLM 功能，**必须**包含概率化验收标准 `[BHIL]`：
```
  ✅ Factuality score ≥ 0.85 across 100 runs (LLM-judge)
  ✅ Zero toxicity violations across 100 runs

agent_story:
  # ... 同上结构，追加：
  verification:
    type: metric
    metric: "factuality_score"
    threshold: "≥ 0.85"
    measurement: "promptfoo eval --config evals/us-001-factuality.yaml"
    eval_config: "evals/us-001-factuality.yaml"
```

**agent_story 的设计约束**：
- `files_to_touch` 不是"可能相关的文件"，是**确定要改的文件**。不确定的放 `context` 里说明。
- `files_to_NOT_touch` 必须填。防止 AI agent 过度修改——这是 nexus-harness `[NEXUS]` 的 dependency boundary 思想。
- `commands_to_run` 的每个命令必须**可直接复制粘贴执行**，不依赖环境变量、不依赖当前目录（用绝对路径或相对于项目根）。
- `done_when` 的每个条件必须是**二值判断**（PASS/FAIL），不需要人工解读。
- `verification.type` 决定下游 Agent 怎么验证：`integration_test`→跑 commands_to_run；`metric`→跑 measurement 命令 + 对比 threshold；`manual`→阻塞等人工确认。

### §6 — Functional Requirements `[OMC]`

按模块（M1/M2/...）分组，每项标注优先级（P0/P1/P2/P3）和现状（✅/⚠️/❌）。

### §7 — Non-Functional Requirements `[BHIL]`

| 维度 | 要求 | 测量指标 | 基线 | 目标 |
|------|------|---------|------|------|
| 性能 | ... | P95 < Xms | 当前 | 目标 |
| 可用性 | ... | > 99.X% | 当前 | 目标 |
| 安全 | ... | ... | 当前 | 目标 |

### §8 — Success Metrics `[BHIL]` + `[OMC]`

| 指标 | 基线 | 3月目标 | 12月目标 | 测量方式 |
|------|------|--------|---------|---------|

对于 AI 功能追加 `[BHIL]`：
| AI 质量指标 | 阈值 | 运行次数 | 评估方法 |
|------------|------|---------|---------|

### §9 — Edge Cases `[VIBE]`

| 场景 | 预期行为 | 现状 |
|------|---------|------|
| 断连 | ... | ✅/⚠️/❌ |
| 超时 | ... | |
| 并发 | ... | |
| 空状态 | ... | |
| 权限边界 | ... | |

### §10 — Out of Scope `[BHIL]` + `[VIBE]`

**显式排除** ≥5 项。光有边界不行，必须说"容易混淆但明确不做"的。

### §11 — Dependencies & Risks `[BHIL]` + `[OMC]`

**外部依赖**：API/服务/库，风险等级，缓解措施
**技术风险**：概率 × 影响矩阵

### §12 — Architecture Overview `[OMC]`

仅保留 PRD 层面的架构约束（分层图 + 模块解耦矩阵）。详细设计指向 ARCHITECTURE.md。

### §13 — Competitive Landscape `[OMC]`

| 方案 | 定位 | 差异化 |

### §14 — Release Plan `[OMC]`

三阶段路线图（当前 → 下阶段 → 远期），每阶段有可验证的里程碑。

### §15 — Open Questions & Approval Checklist `[BHIL]`

```markdown
## Open Questions
- [ ] [Question] — Owner: [Name], Due: [Date]

## Approval Checklist
- [ ] Problem statement is one sentence with no solution hints
- [ ] All user stories use EARS format
- [ ] Success metrics are quantified
- [ ] Out-of-scope items explicitly listed (≥5)
- [ ] All open questions resolved
- [ ] No implementation details appear in this document
```

### §16 — Executable Acceptance Specification（可执行验收规范）`[STAGE-3]`

> **这是 PRD 从 Stage 2（AI 可读）升到 Stage 3（AI 可执行）的核心差异。**
> 本节内容被下游工具（ralplan/team/ralph/gstack）直接消费，驱动自动化验证。

#### 16.1 全局验证入口

每个 PRD 必须定义一个**一键验证脚本**，使任何人（或 AI agent）无需额外上下文即可验证全部 story：

```yaml
verification_suite:
  name: "PRD-NNN 验证套件"
  description: "[一句话：跑什么、验证什么]"
  entrypoint: |
    # 一键验证：所有 US 的 commands_to_run 的并集
    pytest tests/gateway/test_feishu.py -v
    pytest tests/gateway/test_bridge.py -v -k "wechat_to_feishu or image_passthrough"
  stories_covered: [US-001, US-003, US-005]
  expected_runtime: "~45s"
```

#### 16.2 验证矩阵

追踪每个 US 的验证覆盖状态。每行对应 §5 中的一个 US：

| User Story | 验证类型 | 命令/方法 | 阈值 | 覆盖状态 |
|-----------|---------|---------|------|---------|
| US-001 | integration_test | `pytest tests/gateway/test_feishu.py -v` | 全部 PASS | ✅ |
| US-003 | integration_test | `pytest tests/gateway/test_bridge.py -v -k wechat_to_feishu` | 全部 PASS | ✅ |
| US-003 | metric | `grep e2e_latency gateway.log \| tail -100` | P95 < 5000ms | ⚠️ 埋点已实现，未接入 CI |

状态图例：✅ 已验证 | ⚠️ 部分覆盖 | ❌ 未实现 | 🔄 待验证

#### 16.3 CI 集成规范

```yaml
ci:
  trigger: "PR 修改 files_to_touch 中任意文件 → 自动触发关联 US 的验证"
  gate: "所有关联 US 的 done_when 条件满足 → CI gate PASS → 允许 merge"
  skip_list: []  # 需人工验证的 US ID（例如 verification.type=manual）
  story_gate: "任一关联 US 的 done_when 不满足 → CI gate FAIL → block merge"
```

**工作流**：
```
PR 修改 gateway/platforms/wechat.py
  → CI 扫描 PRD §5，找 files_to_touch 含该文件的 US → 找到 US-003
  → 跑 US-003 的 commands_to_run
  → 检查 US-003 的 done_when
  → 全部 PASS → CI gate 通过
  → 任一 FAIL → block merge，失败原因自动回写到 PR 评论
```

#### 16.4 Eval 管道（AI/LLM 功能专用）

```yaml
evals:
  suite: "evals/PRD-NNN-suite.yaml"
  golden_dataset: "evals/golden/PRD-NNN.jsonl"
  min_examples: 20
  pass_threshold: "所有 US 的 verification.threshold 均满足"
  run_command: "promptfoo eval --config evals/PRD-NNN-suite.yaml"
  cadence: "PR 触发 + 每周定时全量"
```

#### 16.5 可执行验证完整性规则

以下三条规则进入**结构门禁**（§4.1）：

| 规则 | 说明 | 严重度 |
|------|------|--------|
| **覆盖完整性** | 每个 US 必须在 §16.2 验证矩阵中有对应行 | blocking |
| **入口一致性** | §16.1 `stories_covered` 必须与 §5 中所有含 `agent_story` 块的 US ID 集合一致 | blocking |
| **无未实现** | §16.2 中不允许存在 `❌ 未实现` 状态——新 PRD 至少需要 `⚠️ 部分覆盖` | blocking |

---

## 3. 两种工作模式

### 3.1 Forward Mode（新项目：想法 → PRD）`[NEXUS]` 哲学 + `[VIBE]` 交互

**触发**：`/prd "idea"` 或 `/prd-forward "idea"`

**流程**（状态机驱动，每步有 gate）：

```
Step 0: 收参
  ├─ 需求输入（文字/文件/草稿）
  ├─ 项目短名（英文小写+连字符，如 question-bank-admin）
  └─ 涉及端（backend / frontend / both）

Step 1: 判定入口模式
  ├─ 一句话/短文字（≤600 字，无 markdown 标题）→ 交互式追问
  └─ 完整 PRD 文件 / 多标题 markdown（>600 字）→ 快速校验落盘

Step 2: [仅交互模式] 复述理解 → 用户确认
  覆盖 5 点：目标用户 | 解决问题 | 核心功能 | 明确不做 | 成功标准
  未确认不进入追问

Step 3: 苏格拉底式追问（5 个维度，逐维收敛）[NEXUS]
  ① 用户与场景  ② 核心功能细节  ③ 数据与存储
  ④ 约束与边界  ⑤ 成功标准与度量
  规则：一次一个问题；3 轮无新信息→切换维度；全部维度完成→合成 PRD

Step 4: 生成 PRD（按 §2 的 15 节结构）

Step 5: 结构门禁（机械质检）
  - frontmatter 完整性（id/status/date/author）
  - 15 节全存在且无占位符 "[...]"
  - EARS 格式检查
  - 量化指标检查（禁止"fast"/"good"/"improve"）

Step 6: 语义门禁（子 Agent 审查）
  派 product-manager agent 或 critic 做独立审查
  不通过→修复→重检，最多 3 轮

Step 7: 落盘 + 衔接下游
  写入 docs/PRD.md 或指定路径
  提示是否进入 ralplan（共识规划）或 prd-to-spec
```

**关键差异**（vs 其他 PRD 生成工具）：
- 不是一次性生成。AI **必须先追问**再动笔。
- 有双门禁。机械缺陷（缺章节/占位符）自动修复；语义缺陷（逻辑矛盾/范围模糊）由独立 reviewer 捕获。
- Session 可恢复。追问中途断连，下次传 `--resume` 继续。

### 3.2 Reverse Mode（老项目：代码库 → PRD）`[OMC]`

**触发**：`/prd-reverse` 或 "逆向 PRD" 或 "从代码生成 PRD"

**场景**：已有成熟代码库，但没有 PRD（或 PRD 严重过时）。需要从代码/架构/文档中**逆向提取**一份反映"现状"的 PRD，作为后续迭代的基线。

**流程**：

```
Step 0: 收参
  ├─ 项目路径（默认 cwd）
  └─ 输出路径（默认 docs/PRD.md）

Step 1: 自动发现（并行扫描）
  ├─ 读 CLAUDE.md / AGENTS.md / README.md → 产品定位
  ├─ 读 ARCHITECTURE.md / docs/ → 架构 + 模块边界
  ├─ 读 pyproject.toml / package.json → 技术栈 + 依赖
  ├─ 扫描 agents/prompts/ → Agent 角色定义
  ├─ 扫描 gateway/platforms/ → 平台适配器
  ├─ 扫描 tools/ → 工具能力矩阵
  ├─ 扫描 skills/ → 技能清单
  └─ 扫描 plugins/ → 插件系统

Step 2: 推断与合成
  ├─ 从代码结构推断功能模块（M1-M12）
  ├─ 从 agent prompts 推断目标用户
  ├─ 从 config 推断非功能需求
  ├─ 从 git log 推断演进历史
  └─ 从 TODO/FIXME 推断已知限制

Step 3: 交互确认（关键：AI 推断的不等于事实）
  对每一项推断，标注 confidence（high/medium/low）：
  - High：直接写入 PRD，标注来源
  - Medium：写入 PRD，标注 "[推断]"，请用户确认
  - Low：列入 Open Questions，不写入正文
  用户逐项确认/纠偏

Step 4: 生成 PRD（按 §2 的 15 节结构）
  frontmatter 标注 mode: reverse

Step 5-7: 同 Forward Mode（结构门禁 → 语义门禁 → 落盘）
```

**逆向特有原则**：
- **所有推断必须标注来源和置信度**。不让 AI 的"理解"伪装成"事实"。
- **CLAUDE.md / AGENTS.md 是第一优先级来源**。这些是项目维护者写给 AI 的，权威性最高。
- **代码结构是第二来源，注释是第三来源**。代码不会说谎，但代码也不说"为什么"。
- **不确定的宁可少写**。一份缺 3 项但全对的 PRD 比一份 15 项全但错了 5 项的有用得多。

---

## 4. 质量门禁详解 `[NEXUS]` 工程化

### 4.1 结构门禁（机械质检）

由主 Agent 在落盘前自动执行：

| 检查项 | 规则 | 严重度 |
|--------|------|--------|
| Frontmatter | id/status/date/author 齐全 | blocking |
| 章节完整性 | 16 节全存在（§1-§16） | blocking |
| 无占位符 | 无 `[...]` `[TBD]` `TODO` `placeholder` | blocking |
| EARS 格式 | 用户故事含 WHEN/WHILE/IF/WHERE 之一 | major |
| 量化指标 | 成功指标无"fast"/"good"/"improve"等模糊词 | blocking |
| Out of Scope | ≥5 条显式排除 | major |
| 优先级标注 | 功能需求含 P0/P1/P2/P3 | major |
| 概率化 AC | AI/LLM 功能含概率化验收标准 | major |
| **agent_story 完整性** `[STAGE-3]` | 每个 US 含 files_to_touch + commands_to_run + done_when + verification | **blocking** |
| **验证矩阵完整性** `[STAGE-3]` | §16.2 覆盖所有 US，无 ❌ 未实现，stories_covered 与 §5 一致 | **blocking** |
| **files_to_NOT_touch** `[STAGE-3]` | 每个 agent_story 至少声明 1 个 files_to_NOT_touch | major |

**修复策略**：
- blocking → 自动修复（脚本补全），修复后复检
- major → 自动修复，修复后复检
- 复检仍有 blocking → 不让落盘，回到用户补充信息

### 4.2 语义门禁（子 Agent 审查）

派独立 product-manager agent 或 critic 做语义审查：

| 检查维度 | 审查问题 |
|---------|---------|
| 问题-方案匹配 | 方案是否真正解决了 Problem Statement 里的痛点？ |
| 用户故事质量 | 每条故事是否可独立验证？验收标准是否二值化（过/不过）？ |
| 范围一致性 | Out of Scope 是否与 Functional Requirements 矛盾？ |
| 竞品诚实性 | 竞品定位是否真实？差异化是否自洽？ |
| 指标可测量 | 每个成功指标是否能用脚本/数据验证（不需要人工判断）？ |
| 边界覆盖 | Edge Cases 是否覆盖了断连/超时/并发/空状态/权限边界？ |

审查结果分为：`APPROVE` / `ITERATE`（有建议） / `REJECT`（有阻断问题）。

---

## 5. 与 omcharness 现有能力集成（Stage 3 升级版）

```
                         ┌──────────────────────┐
                         │   /prd (本技能)       │
                         │  PRD + agent_story    │
                         │  + §16 可执行验收规范  │
                         └──────────┬───────────┘
                                    │
              ┌──────────┬──────────┼──────────┬──────────┐
              │          │          │          │          │
              ▼          ▼          ▼          ▼          ▼
         ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
         │ralplan │ │ team   │ │ ralph  │ │gstack/ │ │gstack/ │
         │共识规划 │ │ 团队   │ │ 自循环 │ │ spec   │ │ review │
         └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘
             │          │          │          │          │
             ▼          ▼          ▼          ▼          ▼
         ┌─────────────────────────────────────────────────────┐
         │  下游工具消费 agent_story 的方式（不加新逻辑）：       │
         │                                                     │
         │  ralplan: Planner 用 files_to_touch 判断模块边界    │
         │  team-exec: executor 拿到 commands_to_run 直接跑    │
         │  team-verify: 对照 done_when 逐项判定               │
         │  ralph: 每轮结束自动跑 commands_to_run              │
         │          → PASS→下个 story; FAIL→修复→重试          │
         │  gstack/spec: verification 块→Issue checklist      │
         │  gstack/review: done_when→审查清单，不再凭感觉审     │
         │                                                     │
         │  CI: PR 改 files_to_touch 文件→自动触发验证          │
         └─────────────────────────────────────────────────────┘
```

### 5.1 与 ralplan 的关系

**ralplan**（`/oh-my-claudecode:ralplan`）是**技术规划**工具（Planner → Architect → Critic 共识循环）。它解决"怎么做"。

**prd** 是**产品需求**工具。它解决"做什么"。

**Stage 3 增强**：ralplan 的 Planner 可直接读取 `agent_story.files_to_touch` 判断模块边界，不再需要全仓探索。Architect 的审核范围也缩小到 `files_to_touch` + 依赖的模块。

**推荐流程**：
1. `/prd "idea"` → 产出带 agent_story 的 PRD
2. `/ralplan --interactive "基于 docs/PRD.md"` → Planner 用 files_to_touch 直接出技术方案
3. `/team "实现 PRD-001"` → executor 拿 commands_to_run 直接开工

### 5.2 与 team pipeline 的 Stage 3 协作

```
team-prd 阶段：  调用 /prd → 产出 PRD + agent_story + §16 验证规范
team-exec 阶段： 每个 executor 拿到：
                 - 自己的 US 的 files_to_touch  → 知道改哪些文件
                 - 自己的 US 的 files_to_NOT_touch → 知道不能碰哪些
                 - 自己的 US 的 commands_to_run → 做完后跑什么验证
team-verify 阶段：对照每个 US 的 done_when 逐项检查
                 → 全部 PASS → 标记 US 完成
                 → 任一 FAIL → 回到 team-exec 修复
```

### 5.3 与 ralph 自循环的关系

**ralph** 自主循环实现 PRD。Stage 3 的 agent_story 使 ralph 实现真正的**无人值守闭环**：

```
ralph 读取 PRD
  → 找到第一个 done_when 不全 PASS 的 US
  → 读 agent_story.files_to_touch → 改代码
  → 跑 agent_story.commands_to_run
  → 检查 agent_story.done_when
  → PASS → 标记 US 完成 → 下一个 US
  → FAIL → 分析失败原因 → 修复 → 重跑 → 直到 PASS 或达到最大轮次
```

ralph 不需要人告诉它"怎么算通过"——`done_when` 就是通过标准。

### 5.4 与 gstack/spec 的关系

**gstack/spec** 将想法转为 GitHub Issue。

Stage 3 增强：spec 生成 Issue 时，自动将 `agent_story.verification` 转为 Issue checklist：
```
Issue: "US-003: 飞书消息桥接到微信"
  - [ ] files_to_touch: gateway/platforms/wechat.py, gateway/bridge.py
  - [ ] 验证: pytest tests/gateway/test_bridge.py -v -k wechat_to_feishu
  - [ ] done_when: 4 项条件全部 PASS
```

### 5.5 与 gstack/review 的关系

**gstack/review** 审查 PR diff。

Stage 3 增强：reviewer 对照 PRD 的 `agent_story` 做**契约式审查**——不是凭感觉看代码，而是逐项验证：
- `files_to_touch` 都被改了吗？（缺改 → 不完整）
- `files_to_NOT_touch` 被误改了吗？（改多了 → 范围越界）
- `commands_to_run` 都跑过了吗？结果贴了吗？

### 5.6 与 CI 的关系 `[STAGE-3]`

PR 修改的文件与 PRD 的 `files_to_touch` 做交集匹配：
```
CI 触发条件：PR 修改了 PRD 中任意 US 的 files_to_touch
CI 动作：跑对应 US 的 commands_to_run
CI gate：所有关联 US 的 done_when PASS → merge 允许
```

### 5.7 与 product-manager agent 的关系

**product-manager agent**（`agents/prompts/product-manager.md`）是 PM 角色定义，含 PRD 模板知识。

本 skill 在**语义门禁**阶段派 product-manager agent 作为独立 reviewer。Stage 3 新增审查维度：agent_story 块的完整性和合理性。

---

## 6. 使用示例

### Forward Mode

```bash
# 交互式（一句话→PRD）
/prd "我想做一个飞书 bot，能自动回复群消息并执行终端命令"

# 快速模式（已有完整 PRD 草稿）
/prd --file docs/draft-prd.md

# 指定输出路径
/prd "idea" --output workspace/my-project/PRD.md
```

### Reverse Mode

```bash
# 从当前目录逆向
/prd-reverse

# 指定项目路径
/prd-reverse --project /path/to/project

# 指定输出路径
/prd-reverse --output docs/reconstructed-PRD.md
```

### 协作模式

```bash
# PRD → 共识规划 → 实现
/prd "idea"
/ralplan --interactive "基于 docs/PRD.md 做技术方案"
/team "实现 PRD-001"
```

---

## 7. 与行业框架的对照表

| 维度 | BHIL | Vibe Coding | Nexus Harness | **本技能（Stage 3）** |
|------|------|-------------|---------------|----------------------|
| PRD 结构 | 8 节 | 9 节 | 7 节 | **16 节**（三者交集 + OMC 扩展 + §16 可执行层） |
| 用户故事格式 | EARS | As a/I want/So that | 自然语言 | **EARS + agent_story** `[BHIL]`+`[STAGE-3]` |
| AI 可执行 | — | — | — | **agent_story + §16 验证矩阵** `[STAGE-3]` |
| CI 集成 | — | — | — | **PR 改 files_to_touch → 自动跑验证** `[STAGE-3]` |
| AI-native | ⭐⭐⭐ | — | — | **概率化 AC + eval 管道** `[BHIL]`+`[STAGE-3]` |
| 质量门禁 | Approval checklist | 无 | 三重（机械+修复+语义） | **双门禁 + 可执行性检查** `[NEXUS]`+`[STAGE-3]` |
| 追问 | 无（模板填空） | 先追问再写 | 苏格拉底 5 维度 | **苏格拉底 5 维度** `[NEXUS]` |
| 逆向工程 | 无 | 无 | 无 | **Reverse Mode（含 agent_story）** `[OMC]` |
| 状态管理 | YAML traceability | 无 | JSON checkpoint + session | **YAML frontmatter + children 自动回写** `[BHIL]`+`[STAGE-3]` |
| 下游衔接 | SPEC → TASK | Task List | TRD → Plan → Dev | **ralplan/team/ralph/review/spec/CI 全链路** `[OMC]`+`[STAGE-3]` |

---

## 8. 反模式（禁止行为）

- ❌ 不追问直接写 PRD——AI 幻觉的最大来源 `[VIBE]`
- ❌ 用"fast"/"good"/"improve"代替量化指标 `[BHIL]`
- ❌ 跳过 Out of Scope——这是范围蔓延的第一入口 `[BHIL]`
- ❌ 把 PRD 写成架构文档——PRD 是 what，架构是 how `[OMC]`
- ❌ 质量门禁不通过就落盘 `[NEXUS]`
- ❌ 逆向模式不标注置信度——推断伪装成事实 `[OMC]`
- ❌ 在 PRD 里写实现细节（DDL/API 签名/代码结构）——那是 Spec 的事 `[BHIL]`

---

## 9. Checklist（Agent 自检 — Stage 3 升级版）

每次产出一份 PRD 后自查：

**结构完整性**：
- [ ] Frontmatter 完整：id / status / date / author / mode
- [ ] 16 节全存在（§1-§16），§16 验证矩阵覆盖所有 US
- [ ] 全文无 "[...]" "[TBD]" "TODO" "placeholder"

**需求质量**：
- [ ] §2 Problem Statement 是一句话 + 带证据
- [ ] §3 Solution Overview 含"What This Is NOT"（≥3 条）
- [ ] §5 User Stories 使用 EARS 格式
- [ ] §5 AI 功能含概率化验收标准（如适用）
- [ ] §8 Success Metrics 含基线→目标→测量方式
- [ ] §9 Edge Cases 覆盖 ≥5 个场景
- [ ] §10 Out of Scope 显式排除 ≥5 项
- [ ] §15 Open Questions 全部解决或标注 owner+deadline
- [ ] 全文无 "fast" / "good" / "improve"（已替换为量化表述）

**Stage 3 可执行性 `[STAGE-3]`**：
- [ ] 每个 US 含完整的 `agent_story` 块
- [ ] 每个 `agent_story` 含 files_to_touch + commands_to_run + done_when + verification
- [ ] 每个 `agent_story` 至少声明 1 个 files_to_NOT_touch
- [ ] 每个 `commands_to_run` 命令可直接复制粘贴执行
- [ ] 每个 `done_when` 条件是二值判断（PASS/FAIL）
- [ ] §16.1 验证入口 `entrypoint` 可一键运行
- [ ] §16.2 验证矩阵的行数与 §5 的 US 数量一致
- [ ] §16.2 中无 `❌ 未实现` 状态
- [ ] §16.1 `stories_covered` 与 §5 的 US ID 集合一致

**质量门禁**：
- [ ] 语义门禁通过（reviewer 返回 APPROVE）
- [ ] 逆向模式：所有推断项标注来源和置信度
