---
name: truesight-mcp
description: Orchestration playbook for the Truesight MCP. Use when scoring inputs against a deployed live evaluation, running error analysis on a dataset, or managing the trace review and promote loop. For building a new evaluation from scratch, use the create-evaluation skill instead.
---

# Truesight MCP Playbook

## Decision tree -- start here

**Are you building a new live evaluation from scratch?**
- Yes -> use the `create-evaluation` skill instead of this one.

**Are you scoring inputs against an existing live evaluation?**
- Yes -> Workflow A below.

**Do you have a dataset of AI outputs and want to discover quality patterns?**
- Yes -> Workflow B below.

**Do you have review items that need judgment and should feed back into the dataset?**
- Yes -> Workflow C below.

---

## Workflow A: Score inputs against a live evaluation

This is the most common workflow. Use when a live evaluation already exists and you want to score one or more inputs.

### Step 1 -- Find the live evaluation

```
list_live_evaluations()
```

Returns a paginated list. Each item has `public_id` and `input_columns`. Pick the one matching your use case. If you already know the `public_id`, skip this step.

### Step 2 -- Score an input

```
run_eval(
  live_evaluation_id="live_abc123",
  inputs={"column_name": "the text to score"}
)
```

- `inputs` keys must match the live evaluation's `input_columns` exactly.
- For multimodal evals, also pass `media_url` with a public image URL.
- The response includes `run_id` and per-chain scores with reasoning.

### Step 3 -- Optional: flag low-scoring results for human review

If the score requires human review, pass the `run_id` to Workflow C.

---

## Workflow B: Error analysis on a dataset

Use when you have a dataset of AI outputs and want to categorize failures systematically. This workflow produces labeled rows with error notes and consolidated categories.

### Step 1 -- Upload or identify the dataset

If the data is not yet in Truesight, upload it:

```
upload_dataset(
  name="My error analysis dataset",
  rows=[{"input": "...", "output": "..."}, ...],
  input_columns="input"
)
```

If the dataset already exists, use `list_datasets()` to find its `public_id`.

### Step 2 -- Fetch rows page by page

```
get_dataset_rows(dataset_id="ds_abc123", page=1, page_size=25)
```

Response includes a `rows` array and a `row_ids` array aligned to it. Iterate through pages using `total_pages`.

### Step 3 -- Suggest error notes for each row

For each row, call:

```
suggest_error_notes(
  dataset_id="ds_abc123",
  row_id=<row_id from row_ids>,
  input="<the trace content to analyze>"
)
```

Returns `error_notes` (detailed explanation) and `error_category` (3-5 word label).

### Step 4 -- Save notes to the dataset

```
update_dataset_row(
  dataset_id="ds_abc123",
  row_id=<row_id>,
  row_data={"_ts_error_notes": "<notes>", "_ts_error_category": "<category>"}
)
```

Repeat steps 3-4 for all rows before consolidating.

### Step 5 -- Consolidate similar categories

```
consolidate_error_categories(dataset_id="ds_abc123")
```

Returns a `mappings` array of `{old_category, new_category}` suggestions. Review them before applying.

### Step 6 -- Apply the mappings

```
apply_category_mappings(
  dataset_id="ds_abc123",
  mappings=[{"old_category": "...", "new_category": "..."}, ...]
)
```

Returns `total_updated` (rows changed). The dataset now has clean, consolidated error categories.

---

## Workflow C: Review and promote labeled data

Use after `run_eval` produces outputs that need human judgment, or when review items already exist in the queue. Judged items are added back to the dataset to improve future evaluations.

### Step 1 -- Flag a run for review (if not already flagged)

```
flag_review_item(run_id="run_abc123")
```

Creates one review item per chain output in the run. `run_id` comes from `run_eval` or `list_results`.

### Step 2 -- List pending review items

```
list_review_items(page=1, page_size=50)
```

Each item has an integer `id`, the result data, and current review status.

### Step 3 -- Judge each item

```
judge_review_item(
  item_id=<integer id>,
  judgment_value="Pass",  # or "Fail", or the correct label for categorical evals
  notes="Optional explanation of the judgment"
)
```

Repeat for every item in the run. All items must have a `judgment_value` before promoting.

### Step 4 -- Promote reviewed items to the dataset

```
add_reviewed_items_to_dataset(run_id="run_abc123")
```

Adds all judged items as new labeled rows in the linked dataset. Returns confirmation with row count added.

After promoting, consider re-running evaluations on the updated dataset to measure improvement.

---

## Scopes reference

| Workflow | Required scopes |
|----------|----------------|
| A - Score inputs | `live-evaluations:read`, `live-evaluations:execute` |
| B - Error analysis | `datasets:read`, `datasets:write`, `error-analysis:execute` |
| C - Review and promote | `review:read`, `review:write` |

If a tool returns "Missing required scope", the API key used for the MCP connection needs those scopes added. Keys are managed in Truesight Settings.

---

## Pagination

All list tools (`list_datasets`, `list_evaluations`, `list_live_evaluations`, `list_results`, `list_review_items`, `get_dataset_rows`) accept `page` and `page_size`. Responses include `total_pages` -- iterate until you have all items.

## Idempotency

Mutating tools (`upload_dataset`, `create_dataset`, `create_and_deploy_evaluation`, `deploy_live_evaluation`, `add_reviewed_items_to_dataset`) accept an optional `idempotency_key`. Pass a stable unique string to make retries safe in agentic loops.
