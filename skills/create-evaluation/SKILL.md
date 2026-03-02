---
name: create-evaluation
description: Scope what quality should be measured, convert it into one or more actionable binary evaluations, deploy those evaluations through Truesight MCP, and generate a companion skill that applies them correctly. Use when a user wants to create new evals, quality checks, guardrails, or pass/fail criteria for AI outputs.
---

# Create Evaluation

Run this skill when a user asks to create evals for a task, workflow, or output type.

## Outcome

Produce all of the following in one flow:
1. Scoped evaluation dimensions with clear pass/fail boundaries
2. Deployed live eval endpoints
3. Full runnable cURL per endpoint (must include exact live eval ID and exact API key)
4. A generated companion skill that explains how to use the evals in the user's workflow

## Default behavior

- Prioritize non-technical scoping first.
- Use binary evaluations by default.
- Create separate evals per dimension by default.
- Avoid asking implementation-detail questions unless they change product intent.
- Infer technical defaults and execute.

## Start from a Template (Fastest Path)

Before going through the full from-scratch workflow below, first call `list_templates` to check whether a pre-built template already covers the use case.

**Template workflow:**

1. `list_templates` -- discover available templates; each result includes a `slug`, `name`, and `use_case` description
2. `provision_template(slug)` -- creates a private dataset copy for the user with pre-labeled rows and judgment configs already configured; returns `{ dataset_id }`
3. `create_and_deploy_evaluation(dataset_id)` -- deploys the live eval from that dataset; capture the returned `api_key` immediately (it is only shown once)
4. `run_eval` with `live_evaluation_id` and `inputs` -- start scoring right away

**Available templates** (as of early 2026):

- AI Writing Detection: `ai-vocabulary-overuse`, `em-dash-overuse`
- Code Quality: `function-naming`, `variable-naming`

The provisioned dataset is the user's own private copy -- they can add rows, tweak judgment criteria, and redeploy at any time to customize.

If no template matches the use case, proceed with the full from-scratch workflow below.

---

## Scoping workflow (high-information questions only)

Ask questions that define quality, not plumbing. Cover:
- What is being evaluated
- What "good" and "bad" look like
- Highest-cost failure modes
- Strictness preference (precision vs recall)
- How results should be used (gating, ranking, revision loop, monitoring, etc.)

Do not ask about dataset schema, API structure, key storage, or endpoint wiring unless the user explicitly wants custom handling.

## Synthesis step

After scoping, return:
- Proposed eval dimensions
- Recommended number of evals and why
- Criterion text for each eval with explicit pass/fail boundary
- Intended usage pattern for eval outputs in downstream workflow

Request one approval checkpoint before build.

## Build step (Truesight MCP)

Use Truesight MCP to implement approved evals.

For each eval:
1. Create/upload dataset with `upload_dataset` or `create_dataset`
   - Pass `input_columns` and `judgment_configs` inline to avoid separate configure calls
   - Use `idempotency_key` for safe retries in agentic loops
2. Deploy using `create_and_deploy_evaluation(dataset_id)`
   - **CRITICAL: the full `api_key` is ONLY returned at creation -- capture and store it immediately**
   - The live evaluation `public_id` is also needed for `run_eval` calls
3. Verify endpoint works with a real call

### judgment_configs reference

Each `judgment_configs` entry defines one scoring dimension. Pass as a list to `upload_dataset` or `create_dataset`.

**Binary (pass/fail) -- most common:**
```json
[{
  "judgment_column": "quality",
  "judgment_type": "binary",
  "criterion": "The response fully addresses the user's question without factual errors. Pass if it does, Fail if it does not."
}]
```

**Categorical (multiple labels):**
```json
[{
  "judgment_column": "tone",
  "judgment_type": "categorical",
  "options": ["professional", "neutral", "unprofessional"],
  "criterion": "Classify the tone of the response."
}]
```

**Continuous (numeric score):**
```json
[{
  "judgment_column": "relevance",
  "judgment_type": "continuous",
  "min_value": 0,
  "max_value": 10,
  "criterion": "Score how relevant the response is to the question, from 0 (irrelevant) to 10 (perfectly relevant)."
}]
```

**Multiple dimensions in one dataset:**
```json
[
  {"judgment_column": "accuracy", "judgment_type": "binary", "criterion": "..."},
  {"judgment_column": "tone", "judgment_type": "categorical", "options": ["formal", "casual"], "criterion": "..."}
]
```

Optional fields per config:
- `notes_column` (str): column for judge reasoning text. Highly recommended so the judge has the reasoning for why the judgment was made.

## cURL requirement (mandatory)

For every deployed eval, construct and store the full runnable cURL using:
- Live eval endpoint ID (`public_id`)
- Its corresponding API key (`api_key`)

Template:

```bash
curl -sS -X POST "https://api.truesight.goodeyelabs.com/api/eval/<public_id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-Type: application/json" \
  -d '{"inputs": { ... }}'
```

You must preserve exact endpoint IDs and keys returned from deployment. No placeholders in final delivered skill unless user asked for placeholders.

## Verification requirement (mandatory)

- Execute the exact cURL written into the companion skill for each eval.
- Confirm successful response and extractable judgment fields.
- Report verification evidence before claiming completion.

## Companion skill generation

Generate a new usage skill tailored to the scoped workflow.

The companion skill must include:
- Clear trigger description: what the eval suite does and when to use it
- Input contract: what inputs must be provided
- Eval execution instructions aligned to scoped usage (not hardcoded to one pattern)
- Output parsing guidance: how to read pass/fail and reasoning
- Full cURL blocks for every eval endpoint
- Operator loop logic for the approved usage pattern (for example: revise-until-pass, gate-on-fail, or monitor-only)

## Final delivery format

Return:
1. Scoping summary
2. Eval catalog (dimension + criterion + pass/fail boundary)
3. Deployment manifest (dataset IDs, eval IDs, live eval IDs, API keys)
4. Companion skill path
5. Verification results for every cURL

If any verification fails, stop and return a concrete fix plan instead of marking done.
