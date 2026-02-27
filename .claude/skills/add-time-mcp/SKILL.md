---
name: add-time-mcp
description: Add time MCP server for date math, timezone conversions, and natural language date parsing. No credentials needed — pure computation. Gives agents resolve, convert, diff, and range tools so they never guess at date math.
---

# Add Time MCP

LLMs are unreliable at date math, day-of-week calculations, and timezone conversions. This skill adds a standalone MCP server with four tools that make "compute, don't guess" the path of least resistance.

## Tools Added

| Tool | Purpose |
|------|---------|
| `mcp__time__resolve` | NL → structured date ("next Thursday 2pm" → ISO + PT/ET) |
| `mcp__time__convert` | Timezone conversion ("2pm PT" → ET, UTC) |
| `mcp__time__diff` | Gap between dates ("how far is March 15?") |
| `mcp__time__range` | List date/time slots in a window ("weekdays next week 9-5") |

All tools default to Pacific Time via `NANOCLAW_PRIMARY_TIMEZONE` env var.

## Phase 1: Pre-flight

### Check if already applied

Read `.nanoclaw/state.yaml`. If `time-mcp` is in `applied_skills`, skip to Phase 3 (Verify).

## Phase 2: Apply Code Changes

### Apply the skill

```bash
npx tsx scripts/apply-skill.ts .claude/skills/add-time-mcp
```

This:
- Adds `container/agent-runner/src/time-mcp.ts` (standalone stdio MCP server)
- Modifies `container/agent-runner/src/index.ts` (adds `mcp__time__*` to allowed tools, wires time MCP server)
- Adds `luxon`, `chrono-node`, `@types/luxon` to container dependencies

### Install dependencies and rebuild

```bash
cd container/agent-runner && npm install
```

```bash
./container/build.sh
```

### Build and restart

```bash
npm run build
```

Linux:
```bash
systemctl --user restart nanoclaw
```

macOS:
```bash
launchctl kickstart -k gui/$(id -u)/com.nanoclaw
```

## Phase 3: Verify

Send test messages to the agent:

- "What day is March 15?" — should use `mcp__time__resolve`
- "Convert 2pm PT to ET" — should use `mcp__time__convert`
- "How far out is April 1?" — should use `mcp__time__diff`
- "What are the weekdays next week?" — should use `mcp__time__range`

Verify tool outputs include correct ISO timestamps, day names, and timezone labels.

## Customization

### Change default timezone

Edit the `NANOCLAW_PRIMARY_TIMEZONE` env var in `container/agent-runner/src/index.ts` where the time MCP server is configured:

```typescript
env: { NANOCLAW_PRIMARY_TIMEZONE: 'America/New_York' },
```

### Restrict to specific groups

Remove `'mcp__time__*'` from the `allowedTools` array and the `time` entry from `mcpServers` in the `query()` call in `container/agent-runner/src/index.ts`. Then add them only to specific groups' configurations.
