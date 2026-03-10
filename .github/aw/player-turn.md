---
on:
  schedule:
    - cron: '0,30 * * * *'   # Every 30 minutes — two chances per tick
  workflow_dispatch:           # Allow manual trigger

permissions:
  contents: read

tools:
  github:
    toolsets: [context]
    mode: remote

safe-outputs:
  jobs:
    submit-moves:
      runs-on: ubuntu-latest
      steps:
        - name: Submit moves to game API
          run: |
            API_BASE_URL="${{ secrets.API_BASE_URL }}"
            GAME_API_KEY="${{ secrets.GAME_API_KEY }}"
            
            # Parse moves array and POST each one
            echo "$GH_AW_AGENT_OUTPUT" | jq -c '.moves[]' | while IFS= read -r move; do
              response=$(curl -s -w "\n%{http_code}" -X POST "$API_BASE_URL/move" \
                -H "X-Api-Key: $GAME_API_KEY" \
                -H "Content-Type: application/json" \
                -d "$move")
              
              http_code=$(echo "$response" | tail -1)
              body=$(echo "$response" | head -1)
              
              echo "Move: $move → HTTP $http_code: $body"
              
              # Stop on auth failure
              if [ "$http_code" = "401" ] || [ "$http_code" = "403" ]; then
                echo "Auth failure — stopping"
                exit 1
              fi
            done
---

You are **Player Two** in SandCastle Wars. Follow the full instructions in `.github/agents/sandcastle-player.agent.md`.

Your API base URL is: ${{ secrets.API_BASE_URL }}

Steps:
1. Fetch and analyse the current game state and rules
2. Decide your moves (up to 12)
3. Output ONLY valid JSON in this exact format: `{"moves": [...]}`

If the game is in a good state and no moves are needed, output: `{"moves": []}` (the submit job will handle an empty array gracefully).
