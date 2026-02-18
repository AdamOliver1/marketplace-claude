---
name: gemini-search
description: "Standard web search via Gemini CLI — Tier 2 in the search hierarchy. Use for meaningful searches that need detail but don't require multi-source cross-referencing: documentation lookups, API changes, framework comparisons, technical how-tos. For trivial one-line lookups, prefer Claude's built-in WebSearch instead. For complex multi-source research, use /deep-search instead. Claude decides whether to add --deep flag based on query complexity."
---

# Gemini Search

Query: $ARGUMENTS

## Query Formulation

Read and follow `../shared/QUERY-FORMULATION.md` before constructing any search query.

## Availability Check

Before running gemini, verify it's installed:

```bash
which gemini 2>/dev/null
```

If not found (non-zero exit code), **do not attempt to run gemini**. Instead:
1. Fall back to Claude's built-in `WebSearch` tool to answer the query.
2. Notify the user: "Gemini CLI not installed. Using built-in WebSearch. Install with: `npm install -g @google/gemini-cli`"

If found, proceed with the command template below.

## Command Template

```bash
gemini -m <MODEL> -p "<PROMPT>" --output-format json --yolo
```

All three flags (`-p`, `--output-format json`, `--yolo`) are mandatory for every invocation.

## Model Selection

- **Default:** `gemini-2.5-flash` — fast lookups, docs, summaries
- **`--deep` flag or complex research:** `gemini-3-pro-preview` — multi-source synthesis, architecture analysis
- **`--model X` flag:** use exactly the model specified

## Patterns

**Web search:**
```bash
gemini -m gemini-2.5-flash -p "Search the web for: <QUERY>. Summarize technical details and provide source URLs." --output-format json --yolo
```

**Pipe local file for review:**
```bash
cat <FILE> | gemini -m gemini-2.5-flash -p "Check if this follows latest <FRAMEWORK> standards. Search for recent API changes." --output-format json --yolo
```

**URL analysis:**
```bash
gemini -m gemini-2.5-flash -p "Analyze and summarize key points from: <URL>" --output-format json --yolo
```

## Output

1. Extract the `response` field from JSON output
2. Present clean Markdown summary with **cited source URLs**
3. If code changes are suggested, verify against local project before applying

## Errors

- Command not found → fall back to WebSearch (see Availability Check above)
- Empty/failed response → retry once with simplified query
- Model unavailable → fall back to `gemini-2.5-flash`
