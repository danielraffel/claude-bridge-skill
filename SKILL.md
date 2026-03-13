---
name: claude-bridge
description: Ask the local Claude CLI for a second opinion or structured output, or generate a deterministic Claude-ready prompt package. Use in non-Claude hosts that can run shell commands.
---

# Claude Bridge

This skill provides two workflows:
- **Bridge mode**: call the local Claude CLI non-interactively and return Claude's response.
- **Package mode**: generate a deterministic Claude-ready prompt package without calling Claude.

## Compatibility

- Requires `claude` in `PATH` and an authenticated Claude account.
- Intended for non-Claude hosts such as Codex, Gemini CLI, Cursor, GitHub Copilot, and OpenCode.
- If the current host is Claude Code, do not use bridge mode. Answer directly unless the user explicitly asks for a Claude prompt package.

## When to Use

Use this skill when:
- The user explicitly asks to ask Claude, use Claude, or get a Claude second opinion.
- The user asks for structured Claude output.
- The user asks to convert or package a request for Claude.
- The user explicitly names `claude-bridge`.

Do not use this skill for:
- General conversation with no Claude intent.
- Tasks that the current host can handle directly with no benefit from a Claude handoff.
- Claude Code bridge calls.

## Example CLI Calls

```bash
claude -p --output-format json "<prompt>"
claude -p --output-format json --json-schema '<schema>' "<prompt>"
claude -p --output-format json --resume <session_id> "<follow_up>"
```

## Mode Selection

Select mode deterministically:
- **Bridge mode** when the user wants a live Claude answer, structured Claude output, or session continuation.
- **Package mode** when the user explicitly wants a Claude-ready prompt package or prompt conversion.
- **Meta mode** for skill maintenance requests such as `help`, `review-skill`, or `literal`.

Routing precedence:
1. Meta intent
2. Continue or resume intent
3. Package or conversion intent
4. Bridge mode

Meta mode escape hatch:
- `help` explains how to use this skill.
- `review-skill` evaluates this skill and proposes improvements.
- `literal <text>` treats `<text>` as content, not a subcommand.

## Bridge Mode Contract

Bridge mode runs Claude CLI and returns Claude output plus minimal execution metadata.

Required parse order for CLI JSON output:
1. `structured_output` when `--json-schema` is used
2. `result`
3. Error state when `is_error == true` or the process exits non-zero

Always capture these fields when available:
- `session_id`
- `is_error`
- `stop_reason`
- `total_cost_usd`

Default bridge response formatting:
- Return Claude's substantive answer only.
- Omit execution metadata unless the user asks for it, session continuity matters, or an error occurred.

When metadata is included, keep it minimal by default:
- `session_id`
- `is_error`
- Add `stop_reason` and `total_cost_usd` only on request or for error analysis.

## Structured Response Schema

When bridge mode needs machine-readable output, use this JSON schema:

```json
{
  "type": "object",
  "properties": {
    "answer": { "type": "string" },
    "assumptions": {
      "type": "array",
      "items": { "type": "string" }
    },
    "risks": {
      "type": "array",
      "items": { "type": "string" }
    },
    "next_actions": {
      "type": "array",
      "items": { "type": "string" }
    },
    "needs_input": { "type": "boolean" },
    "questions": {
      "type": "array",
      "items": { "type": "string" }
    }
  },
  "required": [
    "answer",
    "assumptions",
    "risks",
    "next_actions",
    "needs_input",
    "questions"
  ],
  "additionalProperties": false
}
```

## Clarification Loop

Default behavior:
- Ask zero clarifying questions unless critical ambiguity exists.
- Ask at most 2 questions.
- Still provide best-effort output with explicit assumptions.

In bridge mode:
1. If Claude returns `needs_input: true` or non-empty `questions`, ask the user those questions.
2. Resume the same Claude session with `--resume <session_id>` after the user replies.
3. Continue until `needs_input: false` and no blocking questions remain.

## Package Mode Contract

Output is deterministic and copy-paste-ready with sections in this order:
1. `SYSTEM:`
2. `USER:`
3. `INPUTS:`
4. `CHECKLIST:`
5. `EDGE CASES:` optional

Never add text before `SYSTEM:` or after the final section.

`SYSTEM:`
- Keep it short and stable.
- Include role, priorities, and hard constraints.

`USER:`
- Restate the task, deliverables, constraints, and success criteria.
- Include relevant user-provided context.
- Include clarifying questions only when critical ambiguity exists.

`INPUTS:`
- List artifacts and intended use.
- If none exist, write exactly `- None.`

`CHECKLIST:`
- Use 6 to 12 bullets.
- Include correctness, completeness, constraints adherence, formatting compliance, scope control, and assumption transparency.
- Include edge-case handling only if `EDGE CASES:` is present.

`EDGE CASES:`
- Include only for high-risk or failure-prone conditions.
- Omit entirely if not needed.

## Context and Token Policy

For large inputs:
1. Include the user goal and hard constraints verbatim.
2. Include only directly relevant excerpts from files, logs, or specs.
3. If context is large, summarize first and then call Claude on the summaries.
4. Record assumptions about omitted context in the output.

## Error Handling

Common failures and handling:
- **CLI unavailable or auth issue**: report the failure. Do not silently switch to package mode unless the user asked for packaging.
- **Schema validation failure**: retry once with a clearer structured-output instruction. If it still fails, return raw `result` and flag the schema miss.
- **Permission or tool denial**: narrow the ask, disable tools, or propose an alternative.
- **Conflicting constraints**: preserve user constraints, state the conflict explicitly, and propose minimal resolution options.

## Build Procedure

1. Parse intent, deliverables, constraints, and success criteria.
2. Select `bridge`, `package`, or `meta` mode.
3. Gather only the context required for the chosen mode.
4. Detect critical ambiguity and ask up to 2 questions only if needed.
5. For package mode, emit the deterministic package.
6. For bridge mode, run Claude CLI, parse the JSON contract, and continue the clarification loop if needed.
7. Validate formatting, scope, assumptions, and constraint fidelity before returning.

## Final Check

- Mode selection is explicit and justified.
- Package headings are exact and in the required order.
- `INPUTS:` contains `- None.` when there are no artifacts.
- `CHECKLIST:` has 6 to 12 bullets with the required quality gates.
- `EDGE CASES:` appears only when warranted.
- Bridge output is concise by default.
- Session continuity is preserved when needed.
