---
name: feature-delivery
description: Deliver a code change using researcher, planner, coder, and validator agents with a bounded fix loop.
---

Deliver: $ARGUMENTS

**CRITICAL: You MUST NOT use Read, Edit, Write, Bash, Grep, or Glob directly.
You MUST delegate ALL work to subagents via the Agent tool.
No exceptions — even if the task looks simple or pre-analyzed.**

Workflow — execute every step unconditionally:

1. REQUIRED: Agent tool with subagent_type="fast-researcher" (or "deep-researcher" — see below).
   - Research only. Do not modify files.
   - Return compact handoff.

2. REQUIRED: Agent tool with subagent_type="planner", passing researcher's full output.
   - Create implementation plan from researcher handoff.
   - Do not modify files.

3. REQUIRED: Agent tool with subagent_type="coder", passing planner's full output.
   - Implement only the planner handoff.
   - Keep the smallest safe diff.

4. REQUIRED: Agent tool with subagent_type="validator".
   - Validate the implementation as-is.
   - Run relevant tests/checks.
   - Do not modify files.

5. If validator status is FAIL or PARTIAL:
   - Extract concrete validator findings.
   - Agent tool with subagent_type="coder" to fix only those findings.
   - Re-run validator.
   - Repeat max 3 times.

Stop conditions:
- Stop when validator returns PASS.
- Stop after 3 failed validation loops.
- Stop if validator reports that the plan is architecturally invalid.

If any subagent returns BLOCKED:
- Stop the workflow.

Researcher selection:
- Use subagent_type="fast-researcher" by default.
- Use subagent_type="deep-researcher" if the task involves infra, auth, permissions, billing, migrations, async workflows, cross-service behavior, or
unclear legacy code.
- If fast-researcher returns low confidence, ambiguous results, or BLOCKED due to code complexity, rerun with deep-researcher.

Rules:
- NEVER skip researcher.
- NEVER skip planner.
- NEVER let validator edit files.
- NEVER let coder redesign unless planner is re-run.
- ALWAYS use the Agent tool — never invoke subagents via Skill.
- Keep summaries compact between steps.

Final output:
\`\`\`yaml
delivery_status:
iterations:
research_summary:
plan_summary:
changed_files:
tests_run:
validation_status:
remaining_issues:
confidence:
\`\`\`

