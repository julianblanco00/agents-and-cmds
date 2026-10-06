---
description: "Review + fix pass: fresh-context three-axis code review (Standards + Spec + Danger) of a feature-flow spec's work OR of the current branch's PR; findings are handed to the implementer (Opus 5.5) and re-reviewed until clean, then verified. No new features — fixes only."
argument-hint: [spec path | feature name | PR number/url | pr]
model: claude-opus-5-5
---

# Flow review (review + fix stage only)

Run the review→fix→re-review→verify cycle for: **$ARGUMENTS**

You are a fresh orchestrator, running at whatever model this command's
`model:` frontmatter pins — read it rather than asserting it. All five flow
agents — `flow-implementer`, `flow-standards-reviewer`, `flow-spec-reviewer`,
`flow-danger-reviewer`, `flow-verifier` — are PINNED to Opus 5.5 in their own
agent files; if a launch fails because of usage limits, re-send the same brief
to the generic `claude` subagent (session model) and note the substitution in
the report — never skip the step. Do NOT grill, write specs, or implement NEW
checklist items — this command exists to review work already done and get
the findings FIXED. Only `flow-implementer` touches code, and only
to address review findings. Never push, never open a PR, never comment on or
edit any PR or ticket — PR access below is strictly read-only.

## Process

### 1. Locate the target — spec mode or PR mode

- **Spec mode** (default): $ARGUMENTS is a path, or matches a spec under
  `docs/specs/` by slug/topic. If $ARGUMENTS is empty and exactly one spec
  has unchecked checklist items, use it (say so).
- **PR mode**: $ARGUMENTS is a PR number/URL or the word "pr", OR no spec
  matches and the current branch has an open PR (`gh pr view` succeeds).
- Nothing matches either way, or several specs match: list the candidates
  and ask the user.

### 2. Pin the fixed point and the spec source

- **Spec mode**: read only the spec's header and checklist — the `Baseline`
  sha is the fixed point; the spec FILE is the spec source. Do not load the
  whole spec body into your context; sub-agents read it themselves.
- **PR mode**: `gh pr view [<n>] --json number,baseRefName,title,body,url`
  (read-only). Fixed point = `git merge-base HEAD origin/<baseRefName>`
  (fetch the base ref first if it is stale). Spec source: a feature-flow
  spec matching this branch if one exists; otherwise the PR title/body —
  pass the `gh pr view` COMMAND in the sub-agent brief so they fetch it
  themselves; if the body carries no requirements, the Spec axis reports
  "no spec available" and stops, as its agent definition specifies.

### 3. Sanity-check the fixed point and the repo's hazard record

If the repo's `CLAUDE.md` has a `## Feature flow` section, the verifier brief
in step 7 must quote its check commands and false-green traps. If it does
not, say that hazards are unknown for this repo and have the verifier confirm
its suites actually executed tests — a verifier that does not know which
suites silently skip will report a green that means nothing.


`git rev-parse <fixed-point>` must resolve and `git diff <fixed-point>
--stat` must be non-empty (this diff covers committed AND uncommitted work —
a fully-committed PR branch is simply the case where nothing extra is
uncommitted). If either fails, stop and report — do not launch sub-agents
against a bad ref or an empty diff.

### 4. Review (three axes, fresh contexts, ONE message)

Launch `flow-standards-reviewer`, `flow-spec-reviewer`, and
`flow-danger-reviewer` together in a single message so they run
concurrently, each with an EMPTY context. Do not invoke the
`code-review` skill — these agents carry its Standards and
Spec briefs (smell baseline included) in their own system prompts, so
re-sending that content every round is pure waste.

All three briefs contain only paths, shas, and commands (`git diff
<fixed-point>`, and in PR mode the `gh pr view` command) — never pasted
content. Also point them at the spec's `## Accepted Findings` section if the
spec has one, so they do not re-raise decisions already closed.

Aggregate under `## Standards`, `## Spec`, and `## Danger`, without
reranking across axes.

Do NOT launch `flow-verifier` alongside the axes. Verification is step 7,
after the fix loop converges; a verifier run on a tree a fix pass is about
to change is time thrown away.

### 5. Propose and fix

If there are findings, turn them into a short fix plan (one line per
finding: what, where, how to fix) and show it — then proceed without
waiting. Launch `flow-implementer` with a SELF-CONTAINED brief: the spec
path or PR reference, the findings verbatim with `file:line`, the fix plan,
and the constraint that this is a FIX-ONLY pass — no new features, no scope
beyond the findings, keep existing tests green and extend them only where a
finding demands it.

### 6. Re-review

Re-run step 4, but SCOPED: the diff command becomes
`git diff <fixed-point> -- <paths the fix pass reported touching>`, and the
briefs carry the previous round's findings verbatim so each can be confirmed
resolved. Re-reading the whole diff every round is the largest source of
wasted time here. Run one full unscoped pass only on the round you intend to
close on.

Record every acceptance in the spec's `## Accepted Findings` section (or, in
PR mode, in your running report) BEFORE launching the next round — an
acceptance you did not write down comes back next round and costs a full
round to re-litigate.

Loop fix→re-review until clean, or until only judgement-call smells remain
that you explicitly decide to accept.

**Default bias: accept non-blocking Standards-only nitpicks (style, naming,
minor duplication) instead of spending a fix round on them alone** — log
each acceptance and move on. Only spend a fix round on a Standards finding
when it is bundled with a Spec or Danger finding from the same round that
needs fixing anyway, or when it is severe enough to affect correctness or
maintainability rather than pure style. This bias never applies to
Spec-fidelity findings or Danger findings. Danger findings marked BLOCKING
can never be accepted: they get fixed, or the flow stops and the user
decides. Only spec-sanctioned Danger Warnings may be accepted, with the
sanctioning quote logged.

**Round cap.** After three fix→re-review rounds, if the latest still
produces NEW Blocking findings, stop and report to the user rather than
starting a fourth: say what is still blocking and why you read it as not
converging (usually a diff too large for one review surface, or an
undecided question in the spec). The user decides.

### 7. Verify

Launch `flow-verifier` on the touched workspaces with `mode: gate` to
confirm the fixes broke nothing: full typecheck, lint, unit and integration,
real counts. Red verifier → back to step 5 with the verbatim failures.

### 8. Commit and log

If fixes were made and the verifier is green, commit them to the current
branch per the repo's documented commit convention (never push — the user
decides when the PR updates). If a spec file exists, append one line to its
`## Flow Log`:
`R<n>. review+fix [pr#<n>] | rounds: <k> | findings: <standards>/<spec>/<danger> | accepted: <items or none> | verifier: <counts> | commit: <short sha or none>`.
End with the three-axis report summary; in spec mode, if checklist items
remain unchecked, remind the user that resuming the build loop is
`/feature-flow <feature>`.
