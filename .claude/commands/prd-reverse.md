---
description: |-
  逆向 PRD（Stage 3 可执行层）——从现有代码库逆向提取结构化可执行 PRD。
  自动发现代码结构 → 推断功能模块 → 标注置信度 → 用户确认 → 落盘。
  支持 --supplement 模式：拿现有 PRD 当锚点，只补 agent_story 块。
argument-hint: "[--project <path>] [--output <path>] [--supplement <existing-prd.md>]"
allowed-tools: [Bash, Read, Grep, Glob, Write, Edit, AskUserQuestion, Agent]
---

请按 `skills/product/prd/SKILL.md` 的完整规范执行逆向 PRD 生成。

用户输入：$ARGUMENTS

核心流程：
Step 0: 收参（项目路径默认 cwd，输出路径默认 docs/PRD.md）
Step 1: 自动发现（并行扫描 CLAUDE.md → ARCHITECTURE.md → 代码结构 → agent prompts → 工具注册表）
Step 2: 推断与合成（标注 confidence: high/medium/low）
Step 3: 交互确认（high 直接写入，medium 标注 [推断] 请确认，low 转 Open Questions）
Step 4: 生成 PRD（16 节标准结构 + agent_story 块，frontmatter 标注 mode: reverse）
Step 5-7: 结构门禁 → 语义门禁 → 落盘

若传了 --supplement <path>：不生成完整 PRD，只读现有 PRD 的 §4 User Stories，逐个逆向 agent_story 块并追加。

先 Read skills/product/prd/SKILL.md 加载完整方法论，再执行。
