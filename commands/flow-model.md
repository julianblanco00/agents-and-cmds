---
description: "Switch the feature-flow orchestrator model: sets the model pin in feature-flow.md and flow-review.md to Opus 5.5, or shows the current pins. Agent pins are never touched — their fallback is automatic."
argument-hint: opus | status
---

# Flow model switch

Target: **$ARGUMENTS**

This command manages ONLY the `model:` frontmatter line of
`~/.claude/commands/feature-flow.md` and `~/.claude/commands/flow-review.md`
(the flow ORCHESTRATOR pin). Never touch the agent files
(`~/.claude/agents/flow-*.md`): all five flow agents are pinned to Opus 5.5
and fall back to the session model per-launch on usage limits.

## Behavior

- **`status`** (or empty $ARGUMENTS): print the current `model:` line of both
  command files and all five agent files (`~/.claude/agents/flow-*.md`, and
  any project-level `.claude/agents/flow-*.md` that shadow them), plus a
  one-line reading of what will run where. Change nothing.
- **`opus`**: in both command files, set the frontmatter line to
  `model: claude-opus-5-5` — the default steady state. Use it to restore
  the pin if it drifted.
- Anything else: say the valid arguments and stop.

## How to apply

Edit only the frontmatter `model:` line — one `sed` per file, e.g.:

```bash
sed -i '' 's/^model: claude-.*$/model: claude-opus-5-5/' ~/.claude/commands/feature-flow.md ~/.claude/commands/flow-review.md
```

Then print the resulting `model:` lines of both files as confirmation, and
remind the user: the change applies to the NEXT invocation of
`/feature-flow` / `/flow-review` (a new session picks it up; mid-session,
re-invoking the command is enough).
