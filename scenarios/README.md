# Workflow Scenarios

This directory contains reference n8n workflow JSONs for real-world use cases. These are **not published** to the docs site — they exist as reference material for AI agents building workflows and for developers to import into n8n.

## File naming convention

Each scenario has two files:
- `{use-case}.json` — the raw n8n workflow JSON (importable into n8n)
- `{use-case}.md` — business context, what the workflow does, and why each node is configured the way it is

## Available scenarios

| Scenario | Signal Type | Mode | Check Type | Description |
|----------|------------|------|------------|-------------|
| `obstruction` | Track State | Batch | Type I | Vehicles blocking factory aisles |
| `congestion` | Zone State | Streaming | Zone Activity | Multiple vehicles causing aisle congestion |

## Adding a new scenario

1. Export the workflow JSON from n8n (Settings → Export)
2. Save as `{use-case}.json`
3. Create `{use-case}.md` with:
   - Business use case description
   - Why this mode/signal type/check type was chosen
   - Key configuration decisions
   - Any expressions or metadata worth explaining
4. Update the table above
