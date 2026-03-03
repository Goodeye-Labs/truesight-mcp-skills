---
name: error-analysis
description: Systematically identify and categorize failure modes in evaluated traces using Truesight datasets and error-analysis tools. Use when quality issues are unclear, after major pipeline changes, or when incidents indicate drift.
---

# Error Analysis

Guide the user through trace-grounded failure analysis and dataset labeling.

## Interactive Q&A protocol (mandatory)

Ask one question at a time with lettered options whenever practical.

Example:

```
Which data source should we analyze first?
A) Existing Truesight dataset
B) New dataset to upload
C) Unsure, list datasets first
```

Rules:
- One question per message during setup.
- Prefer lettered options.
- Ask one follow-up if response is ambiguous.

## Core workflow

1. Select or create dataset:
   - If dataset exists, use `list_datasets`.
   - If not, use `upload_dataset`.
2. Collect representative traces:
   - Target approximately 100 traces when possible.
   - Use random plus stratified coverage when volume is high.
3. Analyze row by row:
   - Use `get_dataset_rows` with pagination.
   - For each row, call `suggest_error_notes`.
4. Persist annotations:
   - Save `_ts_error_notes` and `_ts_error_category` with `update_dataset_row`.
5. Consolidate categories:
   - Run `consolidate_error_categories`.
   - Review mapping proposals, then apply with `apply_category_mappings`.
6. Prioritize fixes:
   - Report most frequent categories first.
   - Recommend next skill based on failure type:
     - `create-evaluation` for new evaluation coverage
     - `review-and-promote-traces` for judgment backlog
     - `eval-audit` for broader process gaps

## Analysis heuristics

- Focus on first root failure in each trace, not every downstream symptom.
- Let categories emerge from observed traces, not pre-baked labels.
- Iterate categories after 20 traces, then relabel for consistency.
- Stop when recent traces no longer reveal new failure categories.

## Anti-patterns

- Defining categories before reading traces.
- Treating output quality labels as generic scores without concrete failure modes.
- Skipping relabel after category definitions change.
- Building new evaluators before fixing obvious prompt/tooling/engineering gaps.

## Scopes reference

- `list_datasets`, `get_dataset_rows` require `datasets:read`
- `upload_dataset`, `update_dataset_row`, `apply_category_mappings` require `datasets:write`
- `suggest_error_notes`, `consolidate_error_categories` require `error-analysis:execute`
