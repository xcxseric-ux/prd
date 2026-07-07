# PRD Skill — Stage 3 Executable PRDs for AI Coding Agents

A skill for AI coding assistants that produces **machine-executable PRDs** — every user story carries
`files_to_touch`, `commands_to_run`, `done_when`, and `verification` blocks so AI agents
can read, implement, and verify without human interpretation.

Works with Claude Code / Cursor / Codex CLI / Windsurf / OpenCode and similar tools.

## What makes this different

| Stage | PRD type | AI can... |
|-------|----------|-----------|
| 1 | Document PRD | Read it |
| 2 | Prompt PRD | Read + understand it |
| **3** | **Executable PRD** | **Read + implement + verify autonomously** |

Every user story includes an `agent_story` block:

```yaml
agent_story:
  files_to_touch: [gateway/platforms/feishu.py, gateway/delivery.py]
  files_to_NOT_touch: [gateway/bridge.py]  # prevents over-modification
  commands_to_run: ["pytest tests/gateway/test_feishu.py -v"]
  done_when: ["所有测试 PASS", "P95 延迟 < 5000ms"]
  verification:
    type: integration_test | unit_test | metric | manual
```

Downstream tools (ralplan, team, ralph, CI) consume these blocks directly —
no human needed to explain "how do I verify this?"

## Quick start

```bash
# Claude Code
git clone https://github.com/xcxseric-ux/prd.git .claude/skills/prd

# Cursor
git clone https://github.com/xcxseric-ux/prd.git .cursor/skills/prd

# Codex CLI / OpenCode / Windsurf
git clone https://github.com/xcxseric-ux/prd.git .agents/skills/prd
```

```bash
# Forward: idea → executable PRD
/prd "I want a feishu bot that responds to @mentions and runs terminal commands"

# Reverse: codebase → PRD (for undocumented projects)
/prd-reverse

# Supplement: add agent_story blocks to existing PRD
/prd-reverse --supplement docs/PRD.md
```

## Modes

### Forward (`/prd`)
Interactive Socratic interview → 16-section PRD → dual quality gates → write.

- 5-dimension questioning (users, features, data, constraints, metrics)
- Dual gate: structural (mechanical checks) + semantic (independent agent review)
- Session-resumable — interrupt and continue with `--resume`

### Reverse (`/prd-reverse`)
Parallel codebase scan → inference with confidence labels → user confirmation → write.

- All inferences tagged `high` / `medium` / `low` confidence
- `medium` items written with `[推断]` marker for human review
- `low` items go to Open Questions — never masquerade as facts

### Supplement (`/prd-reverse --supplement`)
Takes an existing Stage 2 PRD and adds `agent_story` blocks to every user story.
Your PRD is the anchor — only the executable layer gets added.

## PRD structure (16 sections)

1. Executive Summary
2. Problem Statement (with evidence)
3. Solution Overview + "What This Is NOT"
4. Target Users & Personas
5. User Stories (EARS format + agent_story blocks)
6. Functional Requirements (P0-P3 prioritized)
7. Non-Functional Requirements (baseline → target)
8. Success Metrics (baseline → 3mo → 12mo)
9. Edge Cases
10. Out of Scope (≥5 explicit exclusions)
11. Dependencies & Risks
12. Architecture Overview
13. Competitive Landscape
14. Release Plan
15. Open Questions & Approval Checklist
16. **Executable Acceptance Specification** (verification suite + matrix + CI spec)

## Methodology lineage

Synthesized from three industry frameworks after deep comparative analysis:

| Source | What we took |
|--------|-------------|
| **BHIL AI-First Toolkit** | EARS user stories, probabilistic AC, YAML traceability, approval checklist |
| **Vibe Coding PRD Template** | Ask-first philosophy, junior developer audience, explicit out-of-scope |
| **Nexus Harness** | Dual quality gates, Socratic 5-dimension questioning, session checkpoints |

## Integration

Works with existing Claude Code tooling — no conflicts:

- **ralplan**: Planner reads `files_to_touch` for module boundary analysis
- **team-exec**: Executor gets `commands_to_run` directly — no exploration needed
- **team-verify**: Verifier checks `done_when` item by item
- **ralph**: Autonomous loop: change → `commands_to_run` → check `done_when` → PASS or fix
- **gstack/spec**: `verification` blocks → GitHub issue checklists
- **CI**: PR touches `files_to_touch` → auto-triggers associated verification

## Author

Created by xcxs as part of the [omcharness](https://github.com/xcxs/omcharness) (养鹰) AI agent runtime project.
