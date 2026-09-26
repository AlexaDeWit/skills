---
name: adopt-orchestration
description: >-
  Onboard a repository to the orchestration process, or re-verify one after a
  process change. Audit the repo against the contract the other skills defer
  to, verify each fact live, surface gaps one ruling at a time, and scaffold
  the agreed files in a PR. Invoke before the first orchestrate-implementation
  loop in a repository.
---

# Adopt orchestration

The other skills stay project-agnostic by deferring to the host repository's documents. This
skill makes sure those documents answer. Run it before the first `orchestrate-implementation`
loop in a repository, and again after a process change: a new deferred fact, a renamed gate, a
changed verification floor.

## The contract

One row per fact a skill defers to. The home column names the conventional place. A different
home is fine when the constitution's routing table points to it.

| Project fact | Conventional home | Consumer |
| --- | --- | --- |
| Entry point: what the project is, the architecture map | `README.md` | orientation |
| Constitution: invariants and a context-routing table | `AGENTS.md` | orientation |
| Commit rules: format, signing, sign-off, AI disclosure | `CONTRIBUTING.md` | open-pull-request |
| PR rules: template, body register, draft flow | `.github/PULL_REQUEST_TEMPLATE.md` and `CONTRIBUTING.md` | open-pull-request |
| Gating CI jobs by display name, and the informational contexts | testing doc or orchestration reference | open-pull-request, implement-ticket, orchestrate-implementation |
| Local verification: the check suite and the no-build floor, as runnable commands | `AGENTS.md` or orchestration reference | implement-ticket, orchestrate-implementation |
| Worktree lifecycle commands | `AGENTS.md` or orchestration reference | implement-ticket, orchestrate-implementation |
| Runbook location: where step-by-step operator procedures live | `runbooks/`, named in the `README.md` architecture map | execute-runbook |
| Work decomposition: where slices live (in-repo slice files, or GitHub issues with milestones as waves), the status convention, the dependency order | orchestration reference | orchestrate-implementation, resume-orchestration |
| Definition of done, comment budget included when counted | orchestration reference | implement-ticket, orchestrate-implementation |
| Compaction instruction | `.agents/compact-prompt.md` or equivalent | resume-orchestration |
| Merge posture: who merges, whether merges land ahead of the gate, whether the required up-to-date rule binds them | orchestration reference | orchestrate-implementation, resume-orchestration |
| Host limits: which test tiers must not run in a subagent or concurrently across worktrees | `AGENTS.md` or orchestration reference | implement-ticket, orchestrate-implementation |
| Operator conventions a brief must transcribe: banned terms, punctuation, spelling register, the boy-scout rule | the operator's own harness-level instruction file, outside the repository | orchestrate-implementation |
| Issue auto-close on merge: on or off | orchestration reference | resume-orchestration |
| Decision records: where a ruling is recorded, and the label for a deferred decision | orchestration reference or `CONTRIBUTING.md` | run-decision-queue |

## Procedure

1. **Audit read-only.** Locate each row's fact. A row whose home sits outside the repository is
   still audited: confirm the file exists and holds the fact, and never let a repository document
   claim that agents load a rule the repository does not carry. Verify live, not by prose: run
   the named commands at the `--list` level, read the CI workflow's terminal gate job and its
   `needs:`, and check `gh` for branch protection, the merge method, and auto-close behaviour. A
   comment about a setting is not the setting.
2. **Choose the operating mode with the owner.** Direct mode (the owner commits, PRs at their
   discretion) needs only the first six rows. Orchestrated mode (the per-PR loop with
   subagents) needs every row. The mode is the owner's call per repository and can change
   later. Repo age does not decide it: a day-old repo with a milestone plan and an issue
   backlog adopts orchestrated mode at once.
3. **Report gaps one ruling at a time,** per
   [`run-decision-queue`](../run-decision-queue/SKILL.md).
4. **Scaffold.** Write the smallest file that answers each agreed row, in the repository's own
   register. Do not copy another project's canon: point at a reference example only when the
   owner asks. Land the changes as one PR, per `open-pull-request`.
5. **Re-run the audit** on the PR branch. Adoption is done when every row of the chosen mode
   resolves against the live repository.

## When a skill changes

A skill that starts deferring to a new project fact adds a row here in the same PR. This table
is the single audit home. The consuming skill keeps only its "the project's docs name X"
phrasing.
