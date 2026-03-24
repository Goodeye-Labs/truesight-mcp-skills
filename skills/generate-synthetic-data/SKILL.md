---
name: generate-synthetic-data
description: Generate synthetic test data for LLM evaluations using dimension-based tuple expansion. Use when the user needs synthetic traces, test cases, eval datasets, or when create-evaluation needs synthetic fallback data.
---

# Generate Synthetic Data

Generate realistic synthetic traces for LLM evaluation datasets using dimension-based variation.

## When to use

- User needs test data for an evaluation but has no production traces.
- `create-evaluation` delegates here when real traces are unavailable.
- User wants to augment sparse real data with targeted synthetic examples.

<HARD-GATE>
Do NOT generate any synthetic data until scoping is complete and the user has approved the generation plan (dimensions, tuple count, trace structure, output destination).
</HARD-GATE>

<HARD-GATE>
BEFORE the first scoping question, search for a structured question tool (e.g., `AskUserQuestion` or similar interactive widget) and load it. Use that tool for EVERY scoping question. Fall back to plain-text lettered options ONLY if no such tool exists in the environment.
</HARD-GATE>

## Scoping protocol

Ask these five questions. Skip any already answered in the conversation (e.g., if `create-evaluation` already established the system type, do not re-ask).

1. **System type.** What kind of AI system produces the traces?
   - Simple RAG, tool-calling agent, multi-turn chat, support bot, classification pipeline, other
2. **Trace structure.** What columns does each trace contain?
   - Offer common patterns based on system type:
     - Simple RAG: `user_query`, `retrieved_context`, `response`
     - Tool-calling agent: `user_request`, `tool_calls`, `final_answer`
     - Multi-turn chat: `conversation_history`, `assistant_response`
     - Support bot: `customer_message`, `kb_lookup`, `agent_reply`
   - Let the user rename, add, or remove columns
3. **Dimensions of variation.** What axes should drive diversity?
   - Propose 3-5 starter dimensions based on system type and known failure modes
   - Each dimension needs 3-6 discrete values
   - Example for RAG: query complexity (simple factual, multi-hop, ambiguous, comparative), domain coverage (billing, technical support, account management), context quality (perfect match, partial match, irrelevant, missing)
4. **Dataset size.** How many final traces?
   - Default recommendation: 50-100 for initial eval scoping, 200+ for statistical significance
5. **Output destination.** Where should the data go?
   - Default: MCP upload when called from `create-evaluation`, file when standalone
   - Options: Truesight dataset (via MCP), JSONL file, CSV file, both

Use the structured question tool (loaded per the HARD-GATE above) for every question. One question per message.

## Core methodology

Follow this sequence exactly. Do not skip steps or combine them.

### Step 1: Draft seed tuples with the user

Generate ~20 tuples as dimension-value combinations. Each tuple is a row of dimension values that will become one trace.

Format tuples as a table:

| # | query_complexity | domain | context_quality | edge_case |
|---|-----------------|--------|-----------------|-----------|
| 1 | simple factual | billing | perfect match | none |
| 2 | multi-hop | technical support | partial match | none |
| 3 | ambiguous | compliance | irrelevant | non-English |

Rules:
- Cover every dimension value at least once across the ~20 tuples.
- Avoid uniform distribution. Weight toward failure-prone combinations.
- Present the table to the user and ask: "Do these combinations represent realistic scenarios your system encounters? Any to add, remove, or adjust?"

Do not proceed until the user validates the tuples.

### Step 2: LLM-expand tuples

After user approval of seed tuples:
- Generate 10+ additional tuples using the same dimensions.
- No duplicate dimension-value combinations allowed.
- Prioritize underrepresented dimension values and novel cross-dimension pairings.
- Present the expanded set for optional user review (do not block on this).

### Step 3: Convert tuples to natural language (two-step)

This is the key quality technique. Do NOT generate traces in a single step.

**Step 3a: Tuple to scenario sketch.**
For each tuple, write a 1-2 sentence scenario description that captures the dimension values in natural terms.

Example tuple: `(multi-hop, billing, partial match, none)`
Scenario: "A customer asks whether upgrading their plan mid-cycle affects their next invoice and any unused credits. The knowledge base has pricing docs but nothing about proration."

**Step 3b: Scenario to full trace.**
Convert each scenario sketch into the full trace structure (matching the columns from scoping).

Example trace for the above scenario:
```json
{
  "user_query": "If I upgrade from Basic to Pro halfway through my billing cycle, will my next invoice be higher? And what happens to the unused days on Basic?",
  "retrieved_context": "Pro plan costs $49/month. Basic plan costs $19/month. Upgrades take effect immediately.",
  "response": "When you upgrade mid-cycle, your next invoice will reflect the Pro plan price of $49/month. The remaining days on your Basic plan will be prorated as a credit on your next invoice."
}
```

Why two steps: single-step generation produces repetitive phrasing and shallow variation. The scenario sketch forces diverse framing before trace generation locks in wording.

### Step 4: Filter for quality

Review generated traces and remove:
- Awkward or unnatural phrasing
- Dimension-value mismatches (trace doesn't reflect its tuple)
- Near-duplicate traces (high textual similarity despite different tuples)
- Traces that could not plausibly come from the target system

Report how many traces survived filtering and the final count.

## Output

### MCP-first path (default when called from create-evaluation)

Invoke the `upload_dataset` tool with:
- `name` set to a descriptive dataset name
- `columns` set to an array of all column names (input columns + any judgment/notes columns if provided by the caller)
- `input_columns` set to the trace structure columns
- `rows` set to the generated trace data
- `idempotency_key` set to a unique string for safe retries

If the caller provided `judgment_configs`, pass them through to `upload_dataset`.

### File-first path (default when standalone)

Write traces to the user's chosen format:
- **JSONL** (default): one JSON object per line. Filename: `synthetic-traces-YYYY-MM-DD.jsonl`
- **CSV**: standard CSV with headers matching trace columns. Filename: `synthetic-traces-YYYY-MM-DD.csv`

After writing, offer to upload to Truesight via the `upload_dataset` tool.

## Optional: pipeline execution

If the user has a live system available, recommend running the synthetic inputs through it to get real outputs. Synthetic inputs with real outputs are more valuable than fully synthetic traces for evaluation scoping.

## Anti-patterns

- Generating traces without dimension-based variation. This produces clustered, non-diverse data.
- Single-step tuple-to-trace generation. This reduces phrasing diversity compared to two-step.
- Dimensions disconnected from actual failure modes. This generates variety without evaluation value.
- Skipping user validation of seed tuples. This risks generating unrealistic scenarios.
- Using synthetic data where real traces are available. Synthetic is a fallback, not a preference.
