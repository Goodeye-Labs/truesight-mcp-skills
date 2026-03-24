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

## Interactive Q&A protocol (mandatory)

<HARD-GATE>
Do NOT call template provisioning tools, create datasets, deploy evaluations, generate cURLs, or produce a companion skill until scoping is complete and the user explicitly approves the scoped evaluation design.
</HARD-GATE>

<HARD-GATE>
BEFORE the first scoping question, search for a structured question tool (e.g., `AskUserQuestion` or similar interactive widget) and load it. Use that tool for EVERY scoping question. Fall back to plain-text lettered options ONLY if no such tool exists in the environment.
</HARD-GATE>

## Anti-pattern: "This is obvious, skip questions"

Do not skip the interactive scoping loop, even when the use case seems simple. Fast assumption-heavy execution creates weak criteria and poor downstream behavior. Keep the dialogue short when possible, but do not skip it.

## Checklist (complete in order)

You MUST complete each item in order:

1. **Initial framing.** Restate the use case and intended operator outcome.
2. **Clarifying dialogue.** Ask one question at a time; prefer multiple-choice when possible.
3. **Approach options.** Propose 2-3 decomposition options with trade-offs and recommendation.
4. **Design approval loop.** Present these sections and get approval after each section:
   - Quality dimensions
   - Pass/fail boundaries and strictness
   - Operational usage pattern (gate, rank, revise loop, monitor)
5. **Seed labeling.** Have the user label a small sample of traces to calibrate the LLM judge (see Seed labeling section below).
6. **Build authorization checkpoint.** Ask for explicit go-ahead before any MCP build or deploy action.
7. **Implementation and verification.** Execute from-scratch flow, verify, then deliver artifacts.

<HARD-GATE>
Every scoping question in the checklist above MUST be asked during the clarifying dialogue. The only exception: skip a question if the user has already explicitly answered it earlier in the conversation. Do not infer answers. Do not skip because the answer seems obvious.
</HARD-GATE>

## Dialogue rules

- Ask exactly one clarifying question per message during scoping.
- Use the structured question tool (loaded per the HARD-GATE above) for every scoping question. Structure each with a short header, 2-4 options with labels and descriptions, and place the recommended option first. Do not add "(Recommended)" or similar annotations to option labels.
- If the user response is ambiguous, ask one follow-up question before moving forward.
- Keep questions focused on quality intent, failure cost, and decision thresholds.

## Quick trial redirect

If the user wants a quick trial or does not yet have a strong evaluation concept, route to `bootstrap-template-evaluation` instead of running this skill.

Use `create-evaluation` for from-scratch evaluation design and deployment.

---

## Scoping workflow (high-information questions only)

Ask questions that define quality, not plumbing. Cover:
- What is being evaluated
- What "good" and "bad" look like
- Highest-cost failure modes
- Whether existing sample data or traces are available (if yes, read them early because they inform dimension selection, criterion wording, and borderline calibration)
- Strictness preference (precision vs recall)
- How results should be used (gating, ranking, revision loop, monitoring, etc.)

Do not ask about dataset schema, API structure, key storage, or endpoint wiring unless the user explicitly wants custom handling.

## Criterion quality standard

For each proposed quality dimension:

- Make it atomic: one dimension per criterion.
- Use strict binary pass/fail boundaries by default.
- Define explicit fail conditions, not just pass intent.
- Include at least one borderline example in scoping discussion when ambiguity risk is high.
- Prefer code-based checks for objective constraints and reserve LLM judgment for interpretive criteria.

Avoid holistic criteria like "is this good?" or "is this helpful?" without concrete boundaries.

## Real traces first, synthetic fallback via generate-synthetic-data

Default to real traces from user workflows whenever available.

If real traces are missing or too sparse to scope quality dimensions, invoke the `generate-synthetic-data` skill. Pass all scoping context already gathered (system type, trace structure, failure modes) so the user is not re-asked.

Synthetic traces are a bootstrap aid, not a replacement for production traces.

## Seed labeling

<HARD-GATE>
Before building the dataset, the user must label a small sample of traces. These labels improve evaluation accuracy and set the standard for how all remaining traces are labeled. The agent then uses those examples to label all remaining traces. No trace may be uploaded without a pre-filled label and reasoning for every judgment column. Skip this step ONLY if the traces already have both labels AND reasoning in every judgment column.
</HARD-GATE>

**Step 1: User labels seed traces**

Select the minimum number of traces needed to capture the labeling pattern. Start with 2-3. Only request more if the first batch does not cover enough variation to label the rest confidently. Absolute maximum: 10 traces.

Prioritize the highest-information traces:
- Borderline cases where pass/fail is genuinely ambiguous
- Traces that span different failure modes
- Cases where the criterion wording could be interpreted multiple ways

Avoid obvious pass or obvious fail examples. They add no labeling signal.

For each selected trace, present it to the user using the structured question tool (loaded per the AskUserQuestion HARD-GATE above). For each judgment dimension, ask:
- The label (Pass/Fail for binary, the category for categorical, the score for continuous)
- A 1-2 sentence reason explaining why that label applies

Present one trace per message. Do not batch them.

**Step 2: Agent labels remaining traces**

Using the user's seed labels as examples, label all remaining traces with both the judgment value and reasoning for every judgment column. Match the user's labeling style, strictness, and reasoning depth.

After labeling, present a summary to the user for approval:
- Total traces labeled per judgment value (e.g., "62 Pass, 25 Fail")
- 2-3 example auto-labeled traces so the user can spot-check quality

If the user flags issues, adjust the labeling approach and re-label. Do not upload until the user approves the distribution and spot-check.

**Step 3: Record labels**

Write all labels (user seed labels and agent-generated labels) into the `judgment_column` and `notes_column` fields for their respective rows.

## Synthesis step

After scoping, return:
- Proposed eval dimensions
- Recommended number of evals and why
- Criterion text for each eval with explicit pass/fail boundary
- Intended usage pattern for eval outputs in downstream workflow

Get explicit user approval on the scoped design before build.

## Build step (Truesight MCP)

Use Truesight MCP to implement approved evals.

For each eval:
1. Create/upload dataset with `upload_dataset` or `create_dataset`
   - Pass `input_columns` and `judgment_configs` inline to avoid separate configure calls
   - The `columns` array MUST include all `judgment_column` and `notes_column` names from `judgment_configs`, in addition to your input columns. The API will reject the request if judgment/notes columns are missing from `columns`.
   - Use `idempotency_key` for safe retries in agentic loops

   Example (text input):
   ```python
   create_dataset(
       name="My Eval",
       columns=["conversation", "quality", "quality_reasoning"],  # includes judgment + notes columns
       input_columns=["conversation"],
       judgment_configs=[{
           "judgment_column": "quality",
           "notes_column": "quality_reasoning",
           "judgment_type": "binary",
           "criterion": "..."
       }]
   )
   ```

   Example (image-only input). Use `media_url_column` with `input_columns=[]`. The image column cannot also be an input column:
   ```python
   create_dataset(
       name="My Image Eval",
       columns=["image_url", "quality", "quality_reasoning"],
       input_columns=[],
       media_url_column="image_url",
       judgment_configs=[{
           "judgment_column": "quality",
           "notes_column": "quality_reasoning",
           "judgment_type": "binary",
           "criterion": "..."
       }]
   )
   ```
   At `run_eval` time, pass `inputs={}` and provide the image via the `media_url` parameter.
2. Deploy using `create_and_deploy_evaluation(dataset_id)`
   - **CRITICAL: the full `api_key` is ONLY returned at creation.** Capture and store it immediately.
   - The live evaluation `public_id` is also needed for `run_eval` calls
3. Verify endpoint works with a real call

### judgment_configs reference

Each `judgment_configs` entry defines one scoring dimension. Pass as a list to `upload_dataset` or `create_dataset`.

**Binary (pass/fail), the most common type:**
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

### File conventions

- **Directory:** `.claude/skills/<skill-name>/SKILL.md` (the file MUST be named `SKILL.md` in all caps)
- **Frontmatter:** Every companion skill MUST start with YAML frontmatter:

```yaml
---
name: <kebab-case-name>
description: <1-2 sentence description>. Use when <trigger phrases>.
---
```

The `description` field drives skill discovery. Include explicit "Use when..." trigger phrases that match how users will ask for this skill.

### Required content

The companion skill must include:
- Clear trigger description: what the eval suite does and when to use it
- Input contract: what inputs must be provided
- Eval execution instructions aligned to scoped usage (not hardcoded to one pattern)
- Output parsing guidance: how to read pass/fail and reasoning
- Full cURL blocks for every eval endpoint
- Operator loop logic for the approved usage pattern (for example: revise-until-pass, gate-on-fail, or monitor-only)

**IMPORTANT:** Document MCP tool calls in natural language with exact parameter names and values. Never use function-call syntax with parentheses. Example: "Invoke the `run_eval` tool with `live_evaluation_id` set to `\"live_xxx\"` and `inputs` set to `{...}`." Parenthesized call syntax triggers security hooks.

## Final delivery format

Return:
1. Scoping summary
2. Eval catalog (dimension + criterion + pass/fail boundary)
3. Deployment manifest (dataset IDs, eval IDs, live eval IDs, API keys)
4. Companion skill path
5. Verification results for every cURL

If any verification fails, stop and return a concrete fix plan instead of marking done.
