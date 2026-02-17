# Advanced Web Search

A Claude Code plugin providing tiered web search skills with graceful degradation.

| Tier | Skill | Backend | Use Case |
|:-----|:------|:--------|:---------|
| 1 | Built-in `WebSearch` | Claude | Trivial lookups, quick fact checks |
| 2 | `/advanced-web-search:gemini-search` | Gemini CLI | Docs, API changes, how-tos |
| 3 | `/advanced-web-search:deep-search` | Gemini + Perplexity | Complex research, cross-referenced results |

Both skills degrade gracefully — if a backend is unavailable, they fall back to whatever is available, down to Claude's built-in WebSearch.

## Quick Start

**Install the plugin:**
```bash
/plugin marketplace add AdamOliver1/marketplace-claude
/plugin install advanced-web-search@adamoliver1-plugins
```

**Install backends (optional — skills work without them):**
```bash
npm install -g @google/gemini-cli
gemini  # Authenticate
# For Perplexity: see "Perplexity MCP" section below (optional)
```

**Use in Claude Code:**
```
/advanced-web-search:gemini-search your query
/advanced-web-search:deep-search your query
```

## Installation

For general instructions on how to install Claude Code plugins and marketplaces, see the [official documentation](https://code.claude.com/docs/en/plugin-marketplaces).

### Option 1: Via Marketplace (Recommended)

1. Add the marketplace to Claude Code:
```bash
/plugin marketplace add AdamOliver1/marketplace-claude
```

2. Install the plugin:
```bash
/plugin install advanced-web-search@adamoliver1-plugins
```

Or use the interactive UI: `/plugin` → **Discover** tab → search for "advanced-web-search"

### Option 2: Direct GitHub Install

```bash
/plugin install https://github.com/AdamOliver1/marketplace-claude
```

### Option 3: Local Development

Clone and test locally:
```bash
git clone https://github.com/AdamOliver1/marketplace-claude.git
cd marketplace-claude/advanced-web-search
claude --plugin-dir .
```

## Prerequisites

Both backends are **optional** — skills work with any combination installed.

### Gemini CLI

```bash
npm install -g @google/gemini-cli
gemini  # Run once to authenticate
```

### Perplexity MCP (Optional)

Perplexity is **optional** — if not configured, the skills automatically fall back to Gemini only. The plugin does not bundle its own Perplexity MCP server to avoid conflicts with your existing setup. If you already have a Perplexity MCP server configured globally, the skills will use it automatically — no extra setup needed.

To add Perplexity support, configure the official MCP server in your global Claude config:

1. Run in Claude Code:
```
/mcp add perplexity -- npx -y @perplexity-ai/mcp-server
```

2. Set your API key in your shell profile (`~/.zshrc` or `~/.bashrc`):
```bash
export PERPLEXITY_API_KEY="your-api-key-here"
```

Get an API key at [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api).

## Usage

### Tier 2 — Gemini Search

```
/advanced-web-search:gemini-search how to configure ESLint flat config
/advanced-web-search:gemini-search --deep compare React Server Components vs Astro Islands
```

**Features:**
- Default model: `gemini-2.5-flash` (fast)
- `--deep` flag: upgrades to `gemini-3-pro-preview`
- Falls back to WebSearch if Gemini CLI is not installed
- Returns cited sources with technical details

### Tier 3 — Deep Search

```
/advanced-web-search:deep-search compare Bun vs Deno vs Node for production APIs
/advanced-web-search:deep-search --deep comprehensive analysis of WebTransport vs WebSockets
```

**Features:**
- Queries Gemini and Perplexity in parallel
- Cross-references results: agreements, unique findings, contradictions
- `--deep` flag: upgrades both backends to their research-grade models
- Gracefully handles missing backends (uses whatever is available)
- Shows which backends were actually used

## Optional: CLAUDE.md Snippet

Add this to your `~/.claude/CLAUDE.md` to give Claude context about when to use each tier:

```markdown
## Web Search: Tiered Search Strategy

Use the following hierarchy when you need to search the web. Choose the appropriate tier based on complexity — do NOT always escalate to higher tiers.

### Tier 1: Built-in WebSearch (Simple/Quick)
For trivial lookups, quick fact checks, or simple one-line answers — use Claude's built-in `WebSearch` tool directly. No skill invocation needed.
- Examples: "what version is X?", "is Y deprecated?", "what's the capital of Z?"

### Tier 2: `/advanced-web-search:gemini-search` (Standard)
For meaningful searches that need more detail — documentation lookups, API changes, framework comparisons, technical how-tos. Claude decides whether to use `--deep` flag or not based on query complexity.
- Examples: "how does X library handle Y?", "latest changes in React 19", "best practices for Z"

### Tier 3: `/advanced-web-search:deep-search` (Complex / Multi-Source)
For complex research requiring high-confidence, cross-referenced results from multiple sources (Gemini + Perplexity in parallel). Use when:
- The user explicitly asks for `/deep-search` or says "deep search"
- The user tells Claude to "use deep search when you need to search"
- The topic is complex and benefits from cross-referencing multiple sources
- High confidence is needed (architectural decisions, comparing technologies, etc.)

### Decision Guide
| Complexity | Tool | Example |
|:---|:---|:---|
| Trivial | `WebSearch` | "what's the latest Node.js LTS version?" |
| Standard | `/advanced-web-search:gemini-search` | "how to configure ESLint flat config" |
| Complex | `/advanced-web-search:deep-search` | "compare Bun vs Deno vs Node for production APIs in 2026" |
| User says so | `/advanced-web-search:deep-search` | user says "use deep search" or "search thoroughly" |
```

## Troubleshooting

### Skills not showing up
- Verify plugin is installed: `/plugin` → **Installed** tab
- Check for errors: `/plugin` → **Errors** tab
- Restart Claude Code if you just installed the plugin

### Gemini CLI not working
- Verify installation: `which gemini`
- Re-authenticate: `gemini` (run the CLI once)
- Check model access: some models require API keys or special access

### Perplexity not working
- Verify the MCP server is configured: `/mcp` in Claude Code
- Verify your API key is set: `echo $PERPLEXITY_API_KEY`
- Get a key at [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api)
- If not configured, the skills automatically fall back to Gemini only

## License

MIT
