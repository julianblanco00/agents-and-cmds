---
description: "Switch the feature-flow orchestrator model: sets the model pin in feature-flow.md and flow-review.md to Fable or Opus (e.g. when Fable runs out of credits), or shows the current pins. Agent pins are never touched — their fallback is automatic."
argument-hint: opus | fable | status
---

# Flow model switch

Target: **$ARGUMENTS**

This command manages ONLY the `model:` frontmatter line of
`~/.claude/commands/feature-flow.md` and `~/.claude/commands/flow-review.md`
(the flow ORCHESTRATOR pin). Never touch the agent files
(`~/.claude/agents/flow-*.md`): `flow-implementer` stays pinned to Opus, and
the four Fable-pinned agents — `flow-standards-reviewer`,
`flow-spec-reviewer`, `flow-danger-reviewer`, `flow-verifier` — already
degrade automatically per-launch and recover on their own when Fable's cap
resets.

## Behavior

- **`status`** (or empty $ARGUMENTS): print the current `model:` line of both
  command files and all five agent files (`~/.claude/agents/flow-*.md`, and
  any project-level `.claude/agents/flow-*.md` that shadow them), plus a
  one-line reading of what will run where. Change nothing.
- **`opus`**: in both command files, set the frontmatter line to
  `model: claude-opus-5`. Use for when Fable is out of credits: the
  orchestrator runs Opus; the four Fable-pinned review/verify agents keep
  trying Fable first and fall back to the session model automatically if it
  is still capped.
- **`fable`**: set the line back to `model: claude-fable-5` in both command
  files. Run this when credits reset.
- Anything else: say the valid arguments and stop.

## How to apply

Edit only the frontmatter `model:` line — one `sed` per file, e.g.:

```bash
sed -i '' 's/^model: claude-.*$/model: claude-opus-5/' ~/.claude/commands/feature-flow.md ~/.claude/commands/flow-review.md
```

Then print the resulting `model:` lines of both files as confirmation, and
remind the user: the change applies to the NEXT invocation of
`/feature-flow` / `/flow-review` (a new session picks it up; mid-session,
re-invoking the command is enough). After switching to opus, suggest
`/flow-model fable` for when the cap resets — Fable-first is the intended
steady state.
