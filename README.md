# LLM-RPG (multi-agent, file-based) Starter Pack

This starter pack provides:
- A file/folder layout for a multi-agent RPG
- Minimal initial world/town/dungeon/player data
- A Claude Code prompt to generate subagents (stored in `claude_code_prompt.md`)

## Key idea
All game state is stored in files under `game/`.
AI agents should only read/write the files they are responsible for.

## Start here
1) Open `game/claude_code_prompt.md` and run it in Claude Code.
2) After subagents are generated, begin with `run_state.json` and drive actions via your orchestrator.
