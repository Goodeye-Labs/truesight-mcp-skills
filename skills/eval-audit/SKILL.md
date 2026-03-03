---
name: eval-audit
description: Audit an existing evaluation workflow and produce severity-ranked findings with concrete next actions. Use when inheriting an eval setup, diagnosing quality regressions, or checking LLM evaluation process maturity.
---

# Eval Audit

Audit LLM evaluation practice and route gaps to the right skills.

## Interactive Q&A protocol (mandatory)

Ask one question at a time with lettered options whenever practical.

Example:

```
What should this audit prioritize first?
A) Live evaluation quality and coverage
B) Error analysis maturity
C) Review and promotion loop health
D) End-to-end process health
```

Rules:
- One question per message.
- Prefer lettered options.
- Ask one follow-up only if ambiguity remains.

## Inputs and evidence

Collect available evidence from Truesight first:
- datasets and dataset rows
- live evaluations
- evaluation runs/results
- review queue items
- existing evaluation criteria and deployment patterns

If evidence is missing, record that as a finding.

## Diagnostic areas

1. Evaluation coverage and quality dimensions
2. Error analysis practice and category quality
3. Review and promotion workflow discipline
4. Template usage versus custom needs
5. Operational hygiene (verification, reruns, iteration cadence)

## Report format (mandatory)

For each finding, include:

```
### <Finding title>
Status: Problem exists | OK | Cannot determine
Evidence: <specific evidence from Truesight context>
Severity: critical | high | medium | low
Recommended skill: <one of current skill set>
Next command: <concrete instruction to run next>
```

Order findings by severity and impact.

## Severity rubric

- critical: likely causes incorrect go/no-go decisions or severe user harm
- high: frequent quality failures or missing control loops
- medium: meaningful process weakness with moderate impact
- low: optimization opportunity, documentation, or ergonomics issue

## Handoff map

- Missing or weak failure taxonomy -> `error-analysis`
- Missing live evaluation coverage -> `create-evaluation` or `bootstrap-template-evaluation`
- Review backlog or low judgment throughput -> `review-and-promote-traces`
- Unclear starting path -> `truesight-workflows`

## Guardrails

- Keep scope within current Truesight MCP capabilities.
