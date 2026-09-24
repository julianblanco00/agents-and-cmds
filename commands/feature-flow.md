---
description: "Feature flow: grill → spec (file only) → loop implement (Opus 5) / code review (Fable 5) / tests until the spec checklist is done and green; Jira requirements re-verified every iteration; one commit per green slice; resumable after /clear via the spec's checklist + flow log"
argument-hint: <feature description, optionally with a Jira link>
model: claude-opus-5
---

# Feature flow

Run the full feature lifecycle for: **$ARGUMENTS**

You are the orchestrator. You run at whatever model this command's `model:`
frontmatter pins — `/flow-model status` shows it, `/flow-model opus|fable`
switches it. Do not assert which model you are; read the pin if it matters.

You never write code yourself. Only the `flow-implementer` subagent (Opus 5)
writes code. All four review/verify agents — `flow-standards-reviewer`,
`flow-spec-reviewer`, `flow-danger-reviewer`, `flow-verifier` — are PINNED to
Fable 5 in their own agent files, regardless of the session model. If
launching one fails because Fable is capped by usage limits, re-send the
SAME path-only brief to the generic `claude` subagent (which uses the session
model) and record `model: <session model>` in that iteration's Flow Log line
— never skip the step because Fable is capped. If Fable is so capped that
this command itself cannot start, run `/flow-model opus` and re-run;
`/flow-model fable` restores Fable-first when the cap resets.

Phases 1–2 are interactive with the user; after the user approves the spec,
Phase 3 runs autonomously until done.

This flow is repo-agnostic. Repo-specific constraints (hazards, rules, check
commands, commit conventions) come from the current repo's own `CLAUDE.md`
and docs — never hardcode them into briefs beyond quoting what the repo
documents.

**Check for a `## Feature flow` section in the repo's `CLAUDE.md` before
Phase 1.** It carries the real check commands, the commands that must never
be run, the suites that false-green, the build-order traps, the commit
convention and hook health, and what the gates do NOT catch. Every brief you
write quotes from it.

If that section is missing, run the sweep in the global `~/.claude/CLAUDE.md`
("In a repo with no hazard section") before launching the first implementer,
and write what you find into the repo's `CLAUDE.md`. Until then treat every
hazard as UNKNOWN rather than absent — an unbootstrapped repo is the case
where a slice ships through four green gates on a suite that executed zero
tests.

Never create, comment on, or modify tracker issues (GitHub/GitLab/Jira)
during this flow, and never push or open a PR unless the user explicitly
asks. Committing is allowed in exactly one place: the per-slice checkpoint
commit in Phase 3 step 7. The spec lives as a file, not a ticket. One
read-only exception: a Jira link the user provided (see below).

## State, resume, and long runs

A long feature outlives any single context window. The spec file, the ADRs,
and the git history are the ONLY authoritative state of this flow; the
conversation is disposable.

- **Resume check — do this FIRST.** If a spec matching this feature already
  exists under `docs/specs/` and has unchecked checklist items, do NOT redo
  Phases 1–2. Reconstruct where things stand from the spec (checklist, Flow
  Log, Jira section) plus `git diff <baseline>` — which shows ALL work so
  far, committed or not. Show the user the checklist and the last Flow Log
  lines, confirm, and resume Phase 3 at the next unchecked item.
  **Interrupted-slice check:** uncommitted changes beyond the last Flow Log
  commit (`git diff <last-logged-sha>`) are a partially completed slice from
  an interrupted run - never assume they are green. Either brief the next
  `flow-implementer` to finish exactly that slice (tell it partial work
  exists and where), or `git restore` those files to restart the slice from
  the last commit. Decide before launching anything else.
- **Context checkpoint.** If your context is close to full (compaction
  warning, or you notice degraded recall), do not start a new heavy step.
  Finish the step in flight, update the Flow Log with the exact resume state
  (including a `resume:` note if you stop mid-iteration), then tell the user:
  run `/clear` and re-run this command. Resuming is cheap by design; a
  degraded orchestrator is not.
- **Flow log.** Keep a `## Flow Log` section at the bottom of the spec. After
  every Phase 3 iteration append exactly ONE compact line:
  `N. <slice> | rounds: <k> | review: clean|accepted <smell> | verifier: <real counts> | Jira: <k>/<n> impl | commit: <short sha> | next: <slice>`.
  No prose, no pasted output — it must stay skimmable, because a fresh
  session reconstructs the whole run from it. `rounds:` is the review→fix
  round count for that slice; it is how a resumed session knows where it
  stands against the round cap, and a slice that needed three or more is a
  signal the next slice should be smaller.
- **Context hygiene (orchestrator).** Subagents burn their own context and
  die; yours must last. Never paste full diffs, full test logs, or whole
  spec bodies into your own messages — carry forward only findings, counts,
  and decisions. Briefs pass PATHS and COMMANDS, not content: subagents read
  the spec and the diff from disk themselves, so your context level can
  never degrade what they see.
- **After any compaction.** If the conversation gets summarized mid-flow,
  re-read the spec (checklist, Flow Log, Jira section, Baseline, and
  `## Accepted Findings`) before the next step, and trust the file over the
  summary wherever they disagree. If you cannot recover which paths the last
  fix pass touched, fall back to a full `git diff <baseline>` for the next
  review round — a scoped round on the wrong scope is worse than a slow one.

## Jira input (only when $ARGUMENTS includes a Jira link or issue key)

The ticket is the source of truth for requirements — the spec derives from
it, never the other way around. Read-only: never edit, transition, or comment
on it.

- **Fetch it now**, before Phase 1, with the available Atlassian tools (or the
  `jira`/`acli` CLI, or the REST API — whatever this environment provides).
  Pull the summary, description, acceptance criteria, and any subtasks or
  linked issues that carry requirements.
- **Extract a numbered requirements list** `J1..Jn` — one entry per concrete,
  verifiable requirement or acceptance criterion. This list feeds the
  grilling (gaps and ambiguities in the ticket are prime questions) and the
  spec.
- **Re-fetch on every Phase 3 iteration** — tickets change mid-flight.

## Phase 1 — Grill (interactive)

Invoke the Skill tool twice: `grilling` and
`domain-modeling` (this is exactly what the plugin's
`grill-with-docs` does). Interview the user in rounds about $ARGUMENTS —
seeded with the Jira requirements, if any — until the design tree is settled.
As decisions crystallise, record glossary terms in the relevant `CONTEXT.md`
and decisions as ADRs under the right `docs/adr/` (respect `CONTEXT-MAP.md`
if the repo has multiple contexts).

## Phase 2 — Spec (file only)

Follow the plugin `to-spec` process with one override: the spec is published
to a FILE, never to the issue tracker.

1. Explore the affected areas of the repo if you haven't already. Use the
   domain glossary vocabulary and respect ADRs in the area.
2. Sketch the seams at which the feature will be tested. Prefer existing
   seams; use the highest seam possible; the ideal number is one. Confirm the
   seams with the user.
3. Record the baseline: `git rev-parse HEAD`.
4. Write the spec to `docs/specs/<kebab-feature-slug>.md` (create the
   directory if needed) with this structure:

   - A header block: `Baseline: <commit sha>` (the fixed point for every code
     review in Phase 3), the branch name, and the Jira link if one was given.
   - `## Problem Statement` — the problem from the user's perspective.
   - `## Solution` — the solution from the user's perspective.
   - `## Jira Requirements` (only if a link was given) — the numbered `J1..Jn`
     list, each quoting the ticket's wording. Every `Jn` must be covered by
     at least one checklist item; note the mapping.
   - `## User Stories` — a LONG numbered list: "As an <actor>, I want
     <feature>, so that <benefit>". Extensive, covering all aspects.
   - `## Implementation Decisions` — modules built/modified, interfaces,
     architectural decisions, schema changes, API contracts. No file paths or
     code snippets (exception: a prototype snippet that encodes a decision
     more precisely than prose — trimmed to the decision-rich parts).
   - `## Testing Decisions` — what makes a good test (external behavior only),
     which modules get tested, the agreed seams, prior art in the codebase.
   - `## Out of Scope`
   - `## Accepted Findings` — starts empty. Every review finding you decide
     to ACCEPT rather than fix gets one line here: the `file:line`, the axis,
     the finding in a clause, and the reason (quoting the spec line that
     sanctions it, where one does). This section is a contract with every
     later review round and every future session: reviewers are instructed
     to read it and NOT re-raise what it lists. Keeping it current is what
     stops the review loop from re-litigating closed decisions — the single
     largest source of wasted rounds. If a later round proves an accepted
     finding wrong, rewrite the line and leave the correction visible; never
     silently overwrite it.
   - `## Checklist` — one `- [ ]` item per spec point / user story that must
     ship. This section drives the Phase 3 loop and is updated as items land.
     Size each item so it is independently shippable AND fits the Phase 3
     blast-radius budget (~5 files / ~400 lines, step 1) — a checklist of
     five fat items produces five slices that will not converge. When a
     single user story would blow that budget, split it into multiple
     checklist items now, at spec time, instead of leaving the split to be
     discovered mid-flow.
   - `## Flow Log` — starts empty; one line per iteration (see above).

   The spec must be complete enough that a session with ZERO conversation
   memory can resume the flow from it alone: agreed seams, decisions, and
   accepted findings all live in the file, not in the chat.

5. Show the user the spec path, the seams, and the checklist, and get an
   explicit "go". This is the LAST interactive gate; everything after runs
   without asking.

## Phase 3 — Loop: implement → review → fix → re-review → test → Jira check → commit (autonomous)

Repeat until EVERY checklist item is checked AND the last verifier run is
green AND (if a Jira link was given) every `Jn` is verified implemented.
Each iteration:

1. **Pick a slice — and keep it small.** The next unchecked checklist
   item(s) that form one coherent vertical slice. If two slices are fully
   independent, launch their implementers in ONE message so they run
   concurrently.

   **Size is a hard constraint, not a preference.** A slice is ONE checklist
   item, or two only when they cannot be tested apart. Before launching,
   estimate the blast radius: if the slice would touch more than ~5 files or
   change more than ~400 lines, SPLIT IT and do the halves as separate
   iterations. Review cost scales with the size of the diff the reviewers
   must read, and every fix pass adds surface for the next round to find —
   so a large slice does not just cost more, it fails to converge. A small
   slice runs a full three-axis round in minutes; a large one runs four
   rounds and still surfaces new blocking findings in the last.
2. **Implement (Opus 5).** Launch `flow-implementer` with a SELF-CONTAINED
   brief (it cannot see this conversation): the spec PATH and which sections
   to read (plus the `Jn` requirement numbers the slice covers), the agreed
   seams, the files/workspaces involved, any repo-documented hazards that
   apply, and the TDD discipline from `tdd` — red before
   green, one slice at a time, tests only at the pre-agreed seams, test
   external behavior through public interfaces, no implementation-coupled or
   tautological tests. It must write the slice's tests, make them pass, and
   report `file:line` changes plus verbatim test output — including the
   explicit LIST OF PATHS it touched, which you need to scope the re-review
   in step 4. If the slice adds untracked files, `git add -N` them so the
   reviewers' `git diff` can see them.
3. **Review (three axes, fresh contexts, ONE message).** Launch
   `flow-standards-reviewer`, `flow-spec-reviewer`, and
   `flow-danger-reviewer` together in a single message so they run
   concurrently. They start with EMPTY contexts — that is where the actual
   reviewing happens, so review quality does not depend on how full YOURS
   is. Each brief contains ONLY: the spec path, the fixed-point sha, and the
   diff command. Never paste diff or spec content into a brief from your own
   context; they read everything from disk. Do not invoke the
   `code-review` skill — these three agents carry its
   Standards and Spec briefs (smell baseline included) in their own system
   prompts, so re-sending that content every round is pure waste.

   **The diff command differs by round — this is the main cost lever:**

   - **Round 1** (fresh from step 2): `git diff <baseline>`.
   - **Rounds 2+** (re-review after a fix pass): scope it to what the fix
     pass actually touched —
     `git diff <baseline> -- <paths the fix pass reported>`. Also give them
     the previous round's findings verbatim so they can confirm each one
     resolved. Re-reading the whole baseline diff every round is the single
     largest source of wasted time in this flow: rounds 2..N pay again for
     everything rounds 1..N-1 already read, while the actual change under
     review is a handful of hunks.
   - **Final round only** (the one you intend to close the slice on): one
     full `git diff <baseline>` pass, to catch interactions between hunks
     that the scoped rounds could not see.

   **Do NOT launch `flow-verifier` in this message.** The implementer already
   reported verbatim test output in step 2, so a verifier run here adds no
   information you do not have — and if any finding produces a fix, its
   result is invalidated before you can use it. Verification is step 5, once
   the review loop has converged. (In a run measured at four rounds, four
   parallel verifier launches cost 46 minutes and only the last one was
   load-bearing.)

   Aggregate the three reports under `## Standards`, `## Spec`, and
   `## Danger`, without reranking across axes.
4. **Fix.** If the review reports findings, hand them verbatim (with
   `file:line`) to a fresh `flow-implementer` to address, then re-run the
   review scoped to the paths that fix pass touched (step 3). Loop review→fix
   until clean, or until only judgement-call smells remain that you
   explicitly decide to accept.

   **Default bias: accept non-blocking Standards-only nitpicks (style,
   naming, minor duplication) instead of spending a fix round on them
   alone** — log each in `## Accepted Findings` and move on. Only spend a
   fix round on a Standards finding when it is bundled with a Spec or Danger
   finding from the same round that needs fixing anyway, or when it is
   severe enough to affect correctness or maintainability rather than pure
   style. This bias never applies to Spec-fidelity findings (missing or
   wrong requirements) or Danger findings. Danger findings marked BLOCKING
   can never be accepted: they get fixed, or the flow stops and the user
   decides. Only spec-sanctioned Danger Warnings may be accepted, with the
   spec quote logged.

   **Write every acceptance down BEFORE the next round.** Each accepted
   finding gets its line in the spec's `## Accepted Findings` section
   immediately — not at the end of the slice. The reviewers are instructed to
   read that section and not re-raise what it lists, so a decision you
   accepted but did not record WILL come back next round, often re-argued as
   Blocking, and cost you a full round to re-litigate. This is bookkeeping
   you do while the implementer runs; it never blocks anything.

   **Round cap.** If three review→fix rounds have run and the latest round
   still produces NEW Blocking findings, stop. Do not start a fourth. Report
   to the user: the rounds so far, what is still blocking, and your read of
   why it is not converging — the usual causes are a slice too large for one
   review surface (step 1) or a spec that never decided the question the
   findings keep circling. Let the user choose between splitting the slice,
   deciding the open question, or accepting a scoped risk. Four rounds of an
   agent arguing with itself is not more rigour; it is an unbounded loop with
   a spec-shaped hole in it.
5. **Test (Fable 5).** Only now, with the review loop converged. Launch
   `flow-verifier` with the touched workspaces, the slice's intent, and the
   MODE:

   - `mode: gate` — the default and the norm here. Full typecheck, lint, unit
     and integration for every touched workspace. Only a green gate lets the
     slice commit.
   - `mode: loop` — narrow (touched workspaces' typecheck/lint + the slice's
     seam and owning unit suite). Use it only when you need a quick check
     mid-iteration and you know a gate run follows before commit.

   If anything fails, hand the verbatim failures back to `flow-implementer`,
   then re-verify. Never mark a slice done on anything but a green `gate`
   run.
6. **Jira requirements check (only if a link was given).** Re-fetch the
   ticket and walk `J1..Jn`: mark each **Implemented** (cite the evidence —
   the test or `file:line` that satisfies it), **Partial**, or **Missing**.
   A requirement without a passing test or concrete evidence is NOT
   implemented, whatever the diff suggests. If the ticket gained or changed
   requirements since the last fetch, update `## Jira Requirements`, add the
   corresponding checklist items, and fold them into upcoming slices. Partial
   and Missing requirements whose slice was supposedly done go back to step 2.
7. **Commit, check off, and log.** Commit the green slice to the current
   branch: stage the slice's files (plus the updated spec only if the repo
   tracks specs — respect the repo's documented rules about generated files),
   write the message in the repo's documented commit convention, and never
   push. Then mark the completed checklist item(s) in the spec file, append
   the iteration's Flow Log line with the commit's short sha, and start the
   next iteration. Each commit is a restore point: `git log <baseline>..HEAD`
   is the run's history in code form.

## Done

When the checklist is complete, the final verifier pass is green, and every
`Jn` is Implemented, report: the spec path, the checked-off checklist, the
final `J1..Jn` → status → evidence table (if a Jira link was given),
per-workspace verifier results with real counts (not adjectives), any
accepted judgement-call findings, the commit list (`git log <baseline>..HEAD
--oneline`), and the full list of changed files. Do not push, open a PR, or
touch the Jira ticket unless the user asks.
