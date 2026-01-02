你是 Claude Code，在本仓库中为一个"文件驱动的多代理RPG"生成可运行的 Subagents。

# 目标
在 `.claude/agents/` 下生成一组 RPG Subagents（符合 Claude Code 规范），用于协助主控 Orchestrator 运行游戏。
**每个 Subagent 有独立的上下文**，不会污染主对话。
**所有游戏数据都保存在 `game/` 的文件中**，子代理只允许按"读写边界"修改文件。

# 重要约束（必须遵守）
1) 不要在对话中长篇粘贴文件内容；直接在仓库内创建/更新文件。
2) 所有 JSON 必须可解析；所有路径必须存在或由你创建。
3) 子代理必须输出"结构化变更计划（Patch Plan）"，避免随意重写大文件。
4) 子代理必须遵守 `game/agents/contracts/contracts.md` 的读写边界。
5) 不要引入外部依赖；只使用本仓库文件。

# 你需要生成的 Subagents（目录结构）

每个 Subagent 是一个 Markdown 文件，包含 YAML frontmatter：

```
.claude/agents/
├── rpg-orchestrator.md      # 主控：读取 run_state、路由到子代理、合并 patch
├── rpg-world-builder.md     # 世界构建
├── rpg-story-narrative.md   # 故事叙述
├── rpg-town-agent.md        # 城镇代理
├── rpg-guild-agent.md       # 公会设施
├── rpg-shop-agent.md        # 商店设施
├── rpg-inn-agent.md         # 旅店设施
├── rpg-dungeon-agent.md     # 地下城代理
├── rpg-combat-referee.md    # 战斗裁判
└── rpg-notary-archivist.md  # 公证人/档案员
```

同时更新：
- `game/agents/manifest.json`：列出每个 subagent 的名称、路径、职责、读写边界

# Subagent 文件格式

每个 Subagent 必须包含 YAML frontmatter：

```yaml
---
name: rpg-agent-name
description: 简短描述，Claude 根据此判断何时调用该 subagent
tools: Read, Write, Edit, Glob
model: inherit
---

# Subagent 正文（System Prompt）
```

# 子代理统一输出格式（必须）
每个 subagent 的响应必须包含两段：

A) `PATCH_PLAN`（JSON，放在三引号代码块内）
字段：
- `summary`：本次做了什么
- `reads`：本次读取的关键文件列表
- `writes`：本次将写入的文件列表
- `ops`：数组，每个元素：
  - `op`: "create" | "update"
  - `path`: 文件路径
  - `mode`: "replace" | "append" | "json_merge"
  - `reason`: 为什么改
  - `content`: 若为 replace/append 给出内容；若 json_merge 给出 merge 对象

B) `NARRATIVE`（简短）
- 给 Orchestrator 的一句话建议：下一步该调用谁/玩家有哪些可选行动

# Orchestrator 的最小路由规则
- `run_state.location.scope == "world"`：展示 world_map 当前节点可走 routes；若进入 town/dungeon 则调用相应 agent
- `scope == "town"`：读 town_map 展示可达设施节点；进入设施调用对应 facility agent
- `scope == "dungeon"`：调用 rpg-dungeon-agent；若触发战斗则调用 rpg-combat-referee
- 每回合结束调用 rpg-notary-archivist，把关键变化写 `lore/notes.md` 和 `player/journal.md`

# 初始化检查
阅读并理解现有文件：
- `game/run_state.json`
- `game/world/world_map.json`
- `game/towns/T001/*`
- `game/dungeons/D001/*`
- `game/player/*`
- `game/rules/combat_rules.md`
- `game/agents/contracts/contracts.md`

# 交付物
1. 把所有 Subagent 写入 `.claude/agents/rpg-*.md`
2. 更新 `game/agents/manifest.json` 包含路径引用
3. 确保每个 Subagent 包含正确的 frontmatter 和完整的 system prompt

# 如何开始运行
Claude Code 会自动发现 `.claude/agents/` 中的 subagents。启动游戏时：
1. 读取 `game/run_state.json` 获取当前状态
2. `rpg-orchestrator` subagent 会根据 `location.scope` 自动路由
3. 每个子代理在独立上下文中执行，返回 PATCH_PLAN
4. Orchestrator 合并 patch，更新状态，返回叙述
