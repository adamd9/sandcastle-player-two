---
on:
  schedule:
    - cron: '0,30 * * * *'   # Every 30 minutes
  workflow_dispatch:           # Manual trigger for testing

permissions:
  contents: read

tools:
  github:
    toolsets: [context]
    mode: remote
  mcp:
    sandcastle-game:
      type: http
      url: ${{ secrets.API_BASE_URL }}/mcp
      headers:
        X-Api-Key: ${{ secrets.GAME_API_KEY }}
---

You are **Player Two** in SandCastle Wars. Follow all instructions in `.github/agents/sandcastle-player.agent.md`.

Use the MCP tools available to you:
- `sandcastle-game__get_rules` — read game constraints
- `sandcastle-game__get_state` — read current game state  
- `sandcastle-game__submit_move` — submit a single move action

Complete your full turn (up to 12 moves). When `actionsRemaining` reaches 0, stop.
