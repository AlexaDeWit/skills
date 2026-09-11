---
name: orchestrate-implementation
description: >-
  Run the team-lead per-PR loop from a high-level strategy conversation.
  Dispatch fresh-context implementer and reviewer subagents, then coordinate
  evaluation, fixes, and the CI gate without filling the lead's context with
  implementation detail. Use when the user asks to dispatch implementation,
  coordinate multiple tickets in parallel, or take the orchestrator role. Also
  use after resume-orchestration restores that role. Do not infer orchestration
  from a single-ticket worktree, pull request, reviewer, or CI request. For
  direct ticket work, use implement-ticket.
---

# Orchestrate implementation

Use this skill only when the user assigns the current agent the orchestrator role. A restored
orchestration checkpoint also carries that assignment. The skill can coordinate one or more pull
requests.

Requests to dispatch implementation or coordinate multiple tickets in parallel assign that role.

Do not invoke it merely because a ticket request names a worktree, pull request, reviewer, or CI.
Those cues route to [`implement-ticket`](../implement-ticket/SKILL.md), where the current agent
writes the code.

The team lead's procedure for turning an approved design into merged PRs. The project's
orchestration reference owns the rationale and the project specifics (slice format, verification
targets, gate composition). This skill is the portable checklist. Read the project document when a
step needs depth.

Dispatch implementation only after the owner's explicit kickoff. Never merge and never push to the
default branch: all code lands through PRs the owner reviews.

## Publication authority

The owner's explicit kickoff for a named repository and scope authorises normal feature-branch
pushes, draft PR creation and updates, and reviewed fix follow-ups within that scope. Do not ask
again for each commit or push. Record the approved scope and repository destination in every
implementer brief and orchestration checkpoint.

This authority does not cover merges, pushes to the default branch, new destinations, secret
disclosure, releases, or deployments. Sandbox and tool approvals still apply. If a tool refuses
an action, verify its target and approved scope or ask the owner to clarify. Do not bypass the
refusal.

## Per-PR loop

Run this for each DAG node once its dependencies merge.

1. **Pick** a slice once every dependency has merged.
2. **Build.** Brief an implementer subagent in its own worktree. The implementer is never a fork
   of the team-lead thread: dispatch a fresh-context agent whose brief is self-contained,
   transcribing the conventions it must follow (commit and trailer format, prose register, pin and
   version policy) instead of inheriting them. A fork carries the lead's framing into the leaf,
   and the brief stops being auditable on its own. Carry the owner's acceptance
   criteria into the brief verbatim, plus the project's comment budget as a numbered criterion the
   reviewer can count. The brief also restates the owner's boy-scout rule for every file the slice
   edits: decide whether the file earns a rewrite, trim comment blocks over the cap and comments
   that restate the implementation, scoped to those files and behaviour-preserving, and name what
   was trimmed in the PR body. A minimal diff is not a virtue in a file that is a comment wall.
   The orchestrator sets the implementer's `model` per slice: higher for design-bearing or
   security-sensitive work, down to a lighter model (Sonnet in Claude Code, Terra in another
   ecosystem) for a genuinely trivial slice, such as a config toggle or a single-file mechanical
   change. Pick the verification mode for the host: an idle host runs the project's local check
   suite before pushing, a shared host runs the project's no-build floor (lint the changed files
   and fix what it reports, then the formatter last) and lets the PR's CI verify. Disjoint hunks
   across every open PR, and the later PR owns the rebase. The implementer opens the draft PR at
   its first push, reports the PR number and head SHA at once, and exits. It does not idle in a
   watch loop.
3. **Evaluate (mandatory).** At that first push, dispatch a fresh-context reviewer pinned to the
   head SHA and to Opus in Claude Code (Sol in another ecosystem), not the frontier tier: Fable in
   Claude Code hits its rate limit too easily under review load. The reviewer has no exposure to
   the implementer's reasoning. Fresh context means non-inherited: the reviewer is never a fork of
   the lead or the implementer thread. It runs beside CI and covers Stage A (requirements,
   gating-test evidence, scope) and Stage B (quality, security, and the comment count). Critical
   findings block. Route the fix as a distinct commit:
   resume the implementer, apply a small reviewer-specified fix directly, or brief a fresh agent.
   Then re-evaluate.
4. **Gate.** Own the watch: at the PR-open report, poll plain `gh pr checks` until it stops
   answering "no checks reported" (about 20 seconds between tries), then start one detached
   background watch on the PR (`gh pr checks <pr> --watch` in a background shell) and act on its
   terminal result. Only the no-checks message retries. A red exit is terminal. On a red,
   resume the implementer with the failing run's log. Review findings and CI reds land as one
   follow-up commit, both re-verify on the new head, and the watch restarts on it. Before every
   push, run `gh run list --branch <branch>`: a push cancels an in-flight run. After any gap,
   sweep `gh pr checks` on every open PR and restart the watches.
5. **Hand off.** Flip the PR ready only when the evaluation passed and the gate is green on the
   same head commit, and every required context passes (`gh pr checks`). Then `gh pr ready`, then
   report. A draft is never reported as done. The project's docs say which contexts are
   informational, and its definition of done may add gates (a perf reading, a doc
   reconciliation): read it before the flip.

## Always on

- Escalate, don't guess: a bounded attempt, then a decision-ready question with options and a
  recommendation.
- Surface decisions one at a time, paced by a task list. Record each ruling before the next
  question.
- Reference work by PR or issue number, never an internal task ID.
- After every merge, rebase the dependent worktrees and re-run their gate. Between waves, run the
  project's inter-wave quality pass when it defines one.
- Commit and PR mechanics: the [`open-pull-request`](../open-pull-request/SKILL.md) skill.
