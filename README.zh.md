# PRD Skill — 可执行 PRD，给 AI Coding Agent 用

一个 AI 编码助手的技能，产出**机器可执行的 PRD**——每条用户故事自带 `files_to_touch`、`commands_to_run`、`done_when`、`verification` 块，AI agent 可以不经人类解释直接读取、实现、验证。

支持 Claude Code / Cursor / Codex CLI / Windsurf / OpenCode 等主流 AI 编码工具。

## 为什么不一样

| 阶段 | PRD 类型 | AI 能做什么 |
|------|---------|-----------|
| 1 | 文档 PRD | 阅读 |
| 2 | 提示词 PRD | 阅读 + 理解 |
| **3** | **可执行 PRD** | **阅读 + 实现 + 自主验证** |

每条用户故事都有一个 `agent_story` 块：

```yaml
agent_story:
  files_to_touch: [gateway/platforms/feishu.py, gateway/delivery.py]
  files_to_NOT_touch: [gateway/bridge.py]     # 防止过度修改
  commands_to_run: ["pytest tests/gateway/test_feishu.py -v"]
  done_when: ["所有测试 PASS", "P95 延迟 < 5000ms"]
  verification:
    type: integration_test | unit_test | metric | manual
```

下游工具（ralplan、team、ralph、CI）直接消费这些块——不需要人解释"怎么验证"。

## 快速开始

```bash
# Claude Code
git clone https://github.com/xcxseric-ux/prd.git .claude/skills/prd

# Cursor
git clone https://github.com/xcxseric-ux/prd.git .cursor/skills/prd

# Codex CLI / OpenCode / Windsurf
git clone https://github.com/xcxseric-ux/prd.git .agents/skills/prd
```

```bash
# 前向：想法 → 可执行 PRD
/prd "我想做一个飞书 bot，@ 它能自动回复消息并执行终端命令"

# 逆向：代码库 → PRD（给没有 PRD 的老项目）
/prd-reverse

# 补丁模式：给已有 PRD 追加 agent_story 块
/prd-reverse --supplement docs/PRD.md
```

## 三种模式

### Forward（`/prd`）
交互式苏格拉底追问 → 16 节标准 PRD → 双门禁质检 → 落盘。

- 5 维度追问（用户与场景、核心功能、数据与存储、约束与边界、成功标准）
- 双门禁：结构门禁（机械质检）+ 语义门禁（独立 agent 审查）
- 支持断点续传（`--resume`）

### Reverse（`/prd-reverse`）
并行扫描代码库 → 推断功能模块 → 置信度标注 → 用户确认 → 落盘。

- 所有推断标注 `high` / `medium` / `low` 置信度
- `medium` 写入时带 `[推断]` 标记，等人确认
- `low` 转 Open Questions——绝不伪装成事实

### Supplement（`/prd-reverse --supplement`）
拿现有 Stage 2 的 PRD 当锚点，只追加 `agent_story` 块。PRD 本体不动，只加可执行层。

## PRD 结构（16 节）

1. Executive Summary（一句话 + 核心价值）
2. Problem Statement（问题陈述 + 证据）
3. Solution Overview（方案 + "What This Is NOT"）
4. Target Users & Personas（用户画像）
5. User Stories（EARS 格式 + agent_story 块）
6. Functional Requirements（P0-P3 优先级）
7. Non-Functional Requirements（基线 → 目标）
8. Success Metrics（基线 → 3 月 → 12 月）
9. Edge Cases（边界情况）
10. Out of Scope（明确不做，≥5 条）
11. Dependencies & Risks（依赖 + 风险矩阵）
12. Architecture Overview（架构概览）
13. Competitive Landscape（竞品定位）
14. Release Plan（三阶段路线图）
15. Open Questions & Approval Checklist
16. **Executable Acceptance Specification**（验证套件 + 验证矩阵 + CI 规范）

## 方法论来源

深入分析三个行业框架后综合而成：

| 来源 | 吸收了什么 |
|------|-----------|
| **BHIL AI-First Toolkit** | EARS 用户故事、概率化验收标准、YAML 可追溯性、Approval Checklist |
| **Vibe Coding PRD Template** | 先追问再落笔、目标读者是初级开发者、显式 Out of Scope |
| **Nexus Harness** | 双质量门禁、苏格拉底 5 维度追问、Session 断点续传 |

## 下游集成

不冲突，是接力关系：

- **ralplan**：Planner 读 `files_to_touch` 做模块边界判断
- **team-exec**：executor 拿 `commands_to_run` 直接开工
- **team-verify**：verifier 逐条检查 `done_when`
- **ralph**：自主循环——改代码 → `commands_to_run` → 检查 `done_when` → PASS 或修复
- **gstack/spec**：`verification` 块 → GitHub Issue checklist
- **CI**：PR 改动 `files_to_touch` → 自动触发关联验证

## 作者

xcxs 创建，作为 [omcharness](https://github.com/xcxseric-ux/omcharness)（养鹰）AI Agent 运行时项目的一部分。
