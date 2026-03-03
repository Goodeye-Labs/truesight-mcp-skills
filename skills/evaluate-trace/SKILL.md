---
name: evaluate-trace
description: Evaluate one or more traces against an existing Truesight live evaluation. Use when a deployed live evaluation already exists and the user wants run outputs with optional handoff to review and promotion.
---

# Evaluate Trace

Use this skill when the user wants to evaluate traces with an existing live evaluation endpoint.

## Interactive Q&A protocol (mandatory)

If context does not make scope clear, ask one question at a time with lettered options.

Example:

```
Do you want to evaluate one trace or a batch?
A) One trace now
B) Small batch (up to 25)
C) Full batch loop
```

Rules:
- Ask exactly one clarifying question per message.
- Prefer lettered options.
- Ask a single follow-up if needed, then proceed.

## Workflow

1. Identify target live evaluation:
   - If live evaluation id is unknown, call `list_live_evaluations`.
   - Select `public_id` and verify required `input_columns`.
2. Prepare inputs:
   - Ensure `inputs` keys exactly match `input_columns`.
   - Include `media_url` for multimodal evaluations when needed.
3. Execute evaluation:
   - Use the `run_eval` tool with `live_evaluation_id` and `inputs` for each trace.
4. Return useful outputs:
   - `run_id`
   - per-judgment scores/outcomes
   - brief interpretation for next action
5. Optional handoff:
   - If human judgment is needed, route to `review-and-promote-traces`.

## Batch mode guidance

- Use deterministic trace ordering and log `run_id` for each input.
- Apply retries with stable idempotency context in caller logic if needed.
- Summarize failures by category or threshold, then propose review handoff.

## Scopes reference

- `list_live_evaluations` requires `live-evaluations:read`
- `run_eval` requires `live-evaluations:execute`

If a scope error occurs, ask the user to create an API key with the missing scope in Truesight Settings.
