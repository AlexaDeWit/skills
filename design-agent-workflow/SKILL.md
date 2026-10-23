---
name: design-agent-workflow
description: >-
  Design a large multi-agent workflow before you write its script: a
  codebase-wide audit, refactor, cleanup, migration, or test pass that fans out
  over many agents and runs for hours. Use when the user asks for a workflow, a
  multi-phase pass, or a fan-out over a whole repository. Covers the ownership
  shape, the control that proves behaviour held, verification through remote
  CI, lanes and host limits, models and cost, visibility, and the brief. Do not
  use for the per-PR ticket loop (orchestrate-implementation) or for one ticket
  (implement-ticket). The harness's own workflow reference owns the script API.
---

# Design an agent workflow

A checklist for the design of a long, unattended, multi-agent run. Work through it before you write
the script. Every rule here cost hours or millions of tokens to learn in a real run.

The harness's workflow authoring reference owns the script API. The project's documents own the
build targets, the gating jobs, and the style rules. This skill owns the shape of the run.

Put the design to the owner before dispatch: the shape, the agent count, the models, and the
expected wall time. The owner's kickoff is explicit. A workflow can spend a day of quota.

## 1. Size the outcome first

- Ask what a good result looks like, in numbers: changed lines, deleted tests, merged modules. If
  the owner ran a similar pass before, read that pull request. Its commit list shows the shape
  that worked: how many commits, per what unit, in what order.
- Estimate the ceiling of your design before you run it. Count the units of work, multiply by the
  size of one change, and compare with the target. A design whose ceiling is a fifth of the target
  is the wrong design, however careful it is.
- Estimate the wall time the same way: units in the longest sequential chain, times the minutes for
  one unit. Say the number. If it is "all night", redesign.

## 2. Pick the ownership shape

The shape sets the ceiling. Pick it on purpose.

| Shape | What it produces | Ceiling |
|---|---|---|
| One finding, one approval, one small commit | Surgical patches | Low. Nobody owns a file, so no file gets rewritten. |
| One owner per area, whole-file rewrite | Local polish: comments, function shape, file order | Medium. An owner cannot see what another area repeats. |
| Ordered by concern across the whole tree | Structural change: one home per helper, dead surface removed | High. Use when the value cuts across areas. |

- For a concern that cuts across areas, run three steps: a read-only inventory in parallel, one
  owner who designs the shared result and writes a map, then parallel owners who apply the map to
  their own files.
- Read each file once. Let one owner work several concerns over its files in one pass, in ordered
  commits, instead of one pass over the tree per concern.
- State the priority order in the brief, and name what the owner does NOT want. An agent told to
  "improve" a file trims comments first, because that is the safest edit. If comment work is the
  least valuable concern, say so, and put it last.

## 3. The control is the second entry

A large change needs proof that behaviour held. Hold one side fixed and change the other, like
double-entry bookkeeping: change the application with the tests fixed, then change the tests with
the application fixed.

- Do not add a third entry. A pre-approval gate on every change (a judge panel per finding) caps
  the run at small, fully specified edits. The fixed control and an adversarial review of the
  finished commit are enough.
- Decide what "behaviour" means before the run. Function-level equivalence (strictness, effect
  order, exception timing) is strict and blocks redesign. System-level equivalence (responses,
  outbound requests, messages, logs, metrics, configuration) allows redesign. Choose the level per
  area: keep the strict rule on hot paths and security boundaries, and the system-level rule
  elsewhere.
- Check what the control pins. Unit tests that call internal functions by name forbid internal
  redesign by construction. If the owner wants redesign, the control is the system-level suites.
- Make the control mechanical. A script lists every touched control file and fails unless a commit
  body carries a ledger line for it. A second script compares the test-name inventory with a
  baseline captured at the base commit.
- In a test pass, a surviving name proves little. Require a break experiment for every deleted,
  merged, or rewritten test: break the application code, confirm the surviving test fails, restore
  the file, and record the experiment in the commit body. Make coverage part of the gate, and read
  the delta, not only the status colour.
- Allow control edits that the main change forces (an import path, a module name), each with a
  ledger line. A rule with no exceptions blocks every module split.
- When the outcome metric is noisy, such as hosted load runs, give each change a deterministic proxy
  as its evidence: identical outputs, allocated bytes, benchmark rows. Measure the noisy metric once,
  on the aggregate, with interleaved repeats. Two runs of identical code can differ by more than one
  change's whole gain, so a screen of single runs per change rejects every change.

## 4. Verify through remote CI, in parallel

Sequential local validation is the worst bottleneck in a long run. A full build, the suites, and a
dead-code check per commit gave a 13 to 20 minute cycle, and the run crawled.

- The local floor for an agent is: compile, format, lint, and the cheap control scripts. Nothing
  that takes minutes, and never per commit.
- Push early and let CI run the suites on many runners. Read the result while the next unit works.
  Send a red result to a repair agent. Do not block a lane on it.
- Use throwaway branches when that buys parallel runners. On a public repository the compute costs
  nothing. N units on N branches verify at once, however small the local host is.
- Read the project's CI triggers first. If CI runs only on pull requests, a throwaway branch needs
  a draft pull request or a manual dispatch. Read the concurrency group too: a second push to the
  same branch usually cancels its run, and separate branches do not cancel each other.
- Watch the CI cache quota. Let throwaway branches restore caches and not save them, or they evict
  the warm entries of the default branch.
- Trust the final check table, never the exit code of a watch command. A watch can exit non-zero
  because its working directory vanished, or report a status that the provider corrects a minute
  later.
- Delete the throwaway branches and close their pull requests at the end.
- If CI itself is slow and serial, fix that first with a small pass. Same checks in a parallel
  arrangement is safe to implement. Anything that drops, samples, or path-filters a check is the
  owner's decision.

## 5. Lanes and the host

- Measure the host before you choose the lane count: cores, free memory, and what else the owner
  runs on it. Build lanes are limited by memory, not by the harness.
- Give each lane its own worktree and branch. Warm a new worktree by copying a built tree's build
  directory. Keep language servers out of lane worktrees.
- Keep lane file sets disjoint. A lane never changes an export that another lane uses: it records
  the want, and a later step takes it.
- Give shared code its own lane or an up-front step. If every lane is barred from the shared
  libraries and defers to the closing agent, that agent works alone for hours. The closing step
  should only merge and verify.
- A cheap mechanical agent integrates: it cherry-picks finished units onto the pull request branch
  under a lock and pushes. Owners never push.
- When lanes reset to the integrated head between steps, record the hash of every commit that did
  not integrate. The reset drops it from the branch, and only the hash recovers it.
- Run the review of unit N alongside the owner of unit N+1. The reviewer reads with git only and
  builds nothing. Its defects go to the next owner in that lane as a first task.
- The harness caps the agents one workflow runs at once, at the smaller of 16 and the CPU count less
  two. Estimate wall time from the total agent-minutes over that cap, not from the lane count. An
  agent that waits on CI holds a slot: collect CI results in a later stage, when they are likely
  done, instead of one watcher per unit.

## 6. Models and cost

- Use a strong mid-tier model for every role that needs judgement: readers, owners, reviewers, and
  the few calls that nothing downstream can repair, such as a plan that shapes every commit or the
  review of the one step that edits the control. In Claude Code that model is Opus. It balances
  cost and confidence better than the frontier tier (Fable), so use the frontier tier only when the
  owner asks for it. Sonnet in Claude Code is a fine workhorse for straightforward work that
  needs little judgement. Use a light model (Sonnet or Haiku in Claude Code) for git and CI-watch
  agents. A frontier-heavy fan-out cannot finish inside a subscription quota.
- Show the model plan as a plain list before dispatch, with the agent count per stage. A table
  posted in the middle of a turn may not render.
- When a run stops early, salvage it. Finished results sit in the run journal. Embed them in the
  next script as a seed so those stages skip their agent. A changed model or prompt is a cache
  miss, so do not rely on resume alone.

## 7. Visibility

- The harness progress view shows phases, not units. A phase that holds 76 units looks "nearly
  done" at unit 13. Push to a draft pull request as units finish, and tell the owner to read the
  pull request.
- Report line counts split by kind: code against comments, and application against tests against
  support. A large total can hide that code only moved and comments shrank.
- Push work in progress when the owner asks whether it is worth continuing. An early look is
  cheaper than a finished run with the wrong shape.
- When a step grows the tree (shared helpers added before the copies go, internals moved behind a
  second module), say so as it happens, with the expected end state.

## 8. The brief

- A subagent loads the project's agent file. It does not load the lead's memory. Transcribe the
  owner's standing rulings into a file in the run kit and make every agent read it first: the
  protected code, the banned swaps, the comment budget as counted numbers, the prose rules.
- "Escalate, don't guess" becomes "skip, don't guess" inside an unattended run. Collect the skipped
  items and the report-only findings in the return value, and relay them to the owner.
- The pull request body follows the project's template. Ledgers and inventories live in commit
  bodies and in the report to the owner, not in the pull request body.
- Give every agent one worktree path and forbid the rest. Forbid amend, force-push, and pushes by
  anyone but the integrator.
- A judge of worth against cost gets the owner's own examples, quoted, and states a reason for every
  call. Do not turn the trade-off into numeric thresholds: owners find it hard to put in numbers, and
  a threshold misfires on the very cases their examples cover.

## 9. Before dispatch

- [ ] The owner agreed the shape, the agent count, the models, and the wall-time estimate.
- [ ] The ceiling of the design meets the owner's target.
- [ ] The control is named, its level is chosen per area, and the two control scripts run.
- [ ] The baseline is captured at the base commit, and no other process shares its build directory.
- [ ] No agent runs a suite per commit. CI does the verification, and one route handles a red.
- [ ] Shared code has its own lane or step. The closing step only merges and verifies.
- [ ] The lane count fits the host's memory.
- [ ] The rulings file exists, and every prompt names it.
- [ ] A draft pull request shows progress, and the owner knows to read it.
- [ ] The default branch is green, or the known red is understood.
- [ ] The script ran end to end with stub agents that return schema-shaped results. A dry run catches
      a placeholder inside a plain-quoted string, which never interpolates, and a backtick inside a
      template-literal prompt. Both otherwise fail hours into the run.
- [ ] For an overnight run: the host will not sleep, and commit signing works past the signing
      agent's passphrase cache.

## 10. After the run

Relay the report yourself. The owner does not see agent output. State what passed, what was
reverted or skipped and why, what the control scripts showed, and what nobody verified. Then record
what the run taught, in the project's memory or in this skill.
