---
description: |-
  PRD 生成（Stage 3 可执行层）——从想法/一句话生成结构化可执行 PRD。
  交互式追问 → 15 节 PRD + agent_story 块 → 双门禁质检 → 落盘。
argument-hint: "[idea | --file <path> | --resume]"
allowed-tools: [Bash, Read, Grep, Glob, Write, Edit, AskUserQuestion, Agent]
---

请按 `skills/product/prd/SKILL.md` 的完整规范执行前向 PRD 生成。

用户输入：$ARGUMENTS

核心流程：
0. 收参（需求输入 + 项目短名 + 涉及端）
1. 判定入口模式（一句话 ≤600 字 → 交互追问 / 完整 PRD → 快速校验）
2. [交互模式] 复述理解 → 用户确认（5 点）
3. 苏格拉底式追问（5 维度，逐维收敛）
4. 生成 PRD（16 节标准结构 + agent_story 块）
5. 结构门禁（机械质检）
6. 语义门禁（派 product-manager agent 审查）
7. 落盘 + 衔接下游

先 Read skills/product/prd/SKILL.md 加载完整方法论，再执行。
