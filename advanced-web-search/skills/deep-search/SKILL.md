---
name: deep-search
description: "Tier 3 multi-source web search — queries Gemini and Perplexity in parallel, cross-references results, and delivers a unified summary with citations. Use when: (1) the user explicitly asks for /deep-search or says 'deep search', (2) the user has instructed Claude to 'use deep search when you need to search', (3) the topic is complex and benefits from cross-referencing multiple sources, (4) high confidence is needed (architectural decisions, technology comparisons). For simpler searches, use /gemini-search (Tier 2) or Claude's built-in WebSearch (Tier 1) instead."
---

# Deep Search

Query: $ARGUMENTS

## Query Formulation

Read and follow `../shared/QUERY-FORMULATION.md` before constructing any search query. Use the same condensed query for both Gemini and Perplexity.

## Workflow

### Step 0: Check Backend Availability

Before searching, determine which backends are available:

- **Gemini:** Run `which gemini 2>/dev/null` — if exit code is non-zero, Gemini is unavailable.
- **Perplexity:** Check if any `mcp__perplexity__*` tools are accessible. Attempt a minimal call to `mcp__perplexity__perplexity_search` — if it errors or the tool is not recognized, Perplexity is unavailable.

Based on availability:
- **Both available** → proceed to Step 1 with both backends in parallel (default behavior)
- **Only Gemini available** → run Gemini only, skip Perplexity. Notify: "Perplexity MCP not configured. Using Gemini only."
- **Only Perplexity available** → run Perplexity only, skip Gemini. Notify: "Gemini CLI not installed. Using Perplexity only. Install Gemini with: `npm install -g @google/gemini-cli`"
- **Neither available** → fall back to Claude's built-in `WebSearch` tool. Notify: "Neither Gemini nor Perplexity is available. Using built-in WebSearch."

Always state which backends were used at the top of the output.

### Step 1: Parallel Search

Run available search backends simultaneously using parallel tool calls in a single response.

**Gemini CLI** (via Bash) — skip if unavailable:
```bash
gemini -m gemini-2.5-flash -p "Search the web for: <QUERY>. Provide detailed technical information, key facts, and source URLs." --output-format json --yolo
```

CRITICAL: The Bash call for Gemini MUST run in **foreground** mode (do NOT set `run_in_background: true`). Set `timeout: 120000` (2 minutes) to allow Gemini enough time to complete. Background mode causes the output to be lost or inaccessible.

**Perplexity** (via MCP tool) — skip if unavailable:
Call the `mcp__perplexity__perplexity_search` tool with the same query.

Both calls MUST be made in the same response as parallel tool calls. The parallelism comes from issuing both tool calls in one message — NOT from backgrounding the Bash command.

### Step 2: Cross-Reference

After results return, analyze them:

1. **Agreements** — Facts confirmed by both sources (high confidence)
2. **Unique findings** — Information found in only one source (note which one)
3. **Contradictions** — Conflicting claims between sources (present both perspectives)

If only one backend was used, skip cross-referencing and present findings directly.

### Step 3: Synthesize

Present results using the output format below.

## Output Format

```markdown
## Deep Search: <query summary>
**Backends used:** Gemini, Perplexity | Gemini only | Perplexity only | WebSearch fallback

### Key Findings
<!-- Facts agreed upon by both sources (or all findings if single source) — high confidence -->
- ...

### Additional Details
<!-- Unique findings from a single source, tagged with origin -->
- [Gemini] ...
- [Perplexity] ...

### Contradictions
<!-- Only if sources disagree — present both sides -->
- Gemini says: ...
- Perplexity says: ...

### Sources
- [Title](URL) — via Gemini
- [Title](URL) — via Perplexity
```

Omit the **Contradictions** section if there are none or only one backend was used. Omit **Additional Details** if both sources fully agree.

## Deep Research Mode

When the user asks for deep/thorough research (e.g., "deep research", "thorough search", `--deep` flag), upgrade both backends:

- **Gemini:** Use `gemini-3-pro-preview` instead of `gemini-2.5-flash`
- **Perplexity:** Use `mcp__perplexity__perplexity_research` instead of `mcp__perplexity__perplexity_search`

## Fallback Chain

Handle failures gracefully:

1. **Perplexity fails** — Notify the user ("Perplexity unavailable, using Gemini only"), continue with Gemini results alone.
2. **Gemini fails** — Notify the user ("Gemini unavailable, using Perplexity only"), continue with Perplexity results alone.
3. **Both fail** — Fall back to Claude's built-in `WebSearch` tool. Notify the user that both primary sources are down.

Always state which sources were actually used at the top of the response, and note any that failed.

## Gemini CLI Notes

- All three flags (`-p`, `--output-format json`, `--yolo`) are mandatory.
- Append `2>/dev/null` to suppress stderr noise.
- Extract the `response` field from the JSON output for synthesis.
- If command not found → skip Gemini (see Step 0 above).
- If empty/failed response → retry once with a simplified query before falling back.
- If model unavailable → fall back to `gemini-2.5-flash`.
