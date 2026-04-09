# Vio Skill

A portable skill for AI agents that provides conversational hotel search via the [Vio MCP server](https://mcp.vio.com/docs/). Search hotels, compare prices, read reviews, and explore room options through natural conversation.

## What It Does

The `/vio` skill teaches agents to use two MCP tools:

- `**search_hotels**` — Discover hotels by location, coordinates, or name. Supports filters (star rating, amenities, property type, price range), sorting, and pagination.
- `**get_hotels**` — Fetch detailed data for specific hotels: reviews, rooms, FAQ, policies, price analytics, and more.

## How It Works

The skill goes beyond basic search — it embeds product thinking into every interaction:

- **Intent-based ranking** — Hotels are ranked by how well they match what the user is actually looking for, not just by price or popularity. The agent evaluates location, reviews, price analytics, and amenities against the user's expressed intent.
- **Smart categorization** — Results are grouped into 2-3 meaningful categories (e.g., "Best Value", "Central Location", "Top Rated") with explanations of why each group stands out.
- **Booking funnel** — The agent actively guides users from search → narrowing → hotel deep-dive → room selection → booking, nudging toward the next step without being pushy.
- **Three modes** — Discovery (exploring options), Comparison (head-to-head evaluation), and Lookup (quick facts about a specific hotel), each with tailored behavior.
- **Intent persistence** — Context accumulates across the conversation. If the user mentioned "honeymoon" early on, that signal shapes every subsequent recommendation.
- **Proactive alternatives** — When criteria conflict with reality (e.g., cheap hotels in an expensive area), the agent reframes the search with context rather than just returning poor results.

## Prerequisites

- Access to a Vio MCP server endpoint (obtain an API key from Vio)
- An AI agent that supports the skill format (OpenClaw, Claude Code, or compatible)

## Download

Get the latest `vio.zip` from the [Releases page](https://github.com/viodotcom/mcp-skill/releases).

Or build locally: `mise run zip`

## Setup

### claude.ai

1. Download `vio.zip` from [Releases](https://github.com/viodotcom/mcp-skill/releases)
2. Go to **Customize → Skills** and upload `vio.zip`
3. Enable the skill
4. Add the Vio MCP server under **Customize → Connectors**:
  - **Type:** HTTP
  - **URL:** `https://mcp.vio.com/mcp?api_key=YOUR_API_KEY`
5. Start a conversation and try: "Find hotels in Amsterdam for next weekend"

### OpenClaw

1. Copy the skill into your personal skills directory:

```bash
mkdir -p ~/.agents/skills/vio
cp SKILL.md ~/.agents/skills/vio/
cp -r references ~/.agents/skills/vio/
```

2. Add the Vio MCP server to `~/.openclaw/openclaw.json`:

```json
{
  "mcp": {
    "servers": {
      "vio": {
        "transport": "streamable-http",
        "url": "https://mcp.vio.com/mcp?api_key=YOUR_API_KEY"
      }
    }
  }
}
```

3. Restart the gateway and verify:

```bash
openclaw gateway restart
openclaw skills list    # Should show "vio" as ready
openclaw mcp list       # Should show "vio" server
```

4. Launch and test:

```bash
openclaw tui
# Then type: /vio Amsterdam for next weekend
```

### Claude Code

Copy or symlink the skill into a plugin directory and use `--plugin-dir`:

```bash
mkdir -p my-plugin/.claude-plugin my-plugin/skills/vio
echo '{"name":"vio-hotels","description":"Hotel search via Vio MCP","version":"1.0.0"}' > my-plugin/.claude-plugin/plugin.json
cp SKILL.md my-plugin/skills/vio/
cp -r references my-plugin/skills/vio/
```

Configure the MCP server in `.mcp.json` at the plugin root:

```json
{
  "vio": {
    "type": "http",
    "url": "https://mcp.vio.com/mcp?api_key=${VIO_API_KEY}"
  }
}
```

Set the `VIO_API_KEY` environment variable and launch:

```bash
export VIO_API_KEY=your-api-key-here
claude --plugin-dir ./my-plugin
```

### Other Agents

Install `SKILL.md` and `references/` into your agent's skill discovery path, then configure the Vio MCP server connection:

```
https://mcp.vio.com/mcp?api_key=YOUR_API_KEY
```

## Usage

Invoke the skill directly:

```
/vio Paris, 3 nights in July, budget under 150 EUR
```

Or let the agent trigger it automatically when you mention hotel search:

```
Find me a 4-star hotel in Amsterdam with a pool
```

### Example Queries

- "Search hotels in Barcelona for next weekend"
- "Find family-friendly apartments in Rome with free cancellation"
- "Compare the Hilton and Marriott in London"
- "Show me reviews for that hotel"
- "What rooms are available? I need something with a king bed"
- "Sort by price, cheapest first"
- "Show me more results"

## File Structure

```
SKILL.md                        # Core skill definition
references/
  tool-reference.md             # Complete tool schemas and examples
README.md                       # This file
.mise.toml                      # Build commands (mise run zip)
```

## License

[BSD 3-Clause](LICENSE)