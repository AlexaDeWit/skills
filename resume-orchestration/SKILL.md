---
name: resume-orchestration
description: >-
  Resume the team-lead seat after compaction or restart. Restore the volatile
  orchestration state, verify it against git and GitHub, and retrieve only the
  process or design sections the next decision needs.
---

# Resume orchestration

Resume from a compact checkpoint and live state. Do not rebuild general project knowledge by
rereading the full process and design canon. The project constitution is already loaded, and
stable detail stays retrievable from repository files.

## Delta-based resume

1. **Extract the checkpoint:** identify the objective, active slices, agents/worktrees, PRs,
   decisions, blockers, verification state, approved scope and repository destination, and exact
   next action from the compacted thread. Restore the recorded kickoff under
   [publication authority](../orchestrate-implementation/SKILL.md#publication-authority).
   Treat all git, GitHub, CI, and status claims as provisional.
2. **Verify volatile state:** run bounded queries for:
   - the current branch and working tree
   - recent commits relevant to active work
   - the worktrees or agents the checkpoint names
   - open PRs and their draft/check state
   - the issues of PRs merged since the checkpoint. When the repository does not auto-close
     issues on merge, a merged PR's issue stays open until the team lead closes it by hand.
   - the CI conclusion of every head merged since the checkpoint. A merge state is not a green
     gate, and an owner can merge ahead of the run that then fails.
   - the status and acceptance criteria of active or next-dispatchable slices

   A gap kills watches and dispatched agents alike, and silence looks the same as progress. Sweep
   `gh pr checks` across every open PR directly, check the liveness of any agent the checkpoint
   names, then restart the watch on every open draft the loop owns, per
   [`ci-gate.md`](../orchestrate-implementation/ci-gate.md).
3. **Retrieve by decision:** read only the sections the next orchestration action needs:
   - per-PR loop, fix routing, evaluation, gate, or guardrails from the project's orchestration
     reference
   - active slice records
   - design sections a blocker or review finding implicates
4. **Reconcile selective memory:** read `MEMORY.md` when the checkpoint names stale memory, when
   a current decision depends on it, or when there is no checkpoint at all. A restart without one
   may leave the durable decision queue, the standing holds, and the candidates not yet filed in
   memory as their only record. Verify a memory's volatile claims before acting on them. A memory
   that contradicts this skill, another skill, or a project document is stale by default: correct
   or delete it rather than follow it, and say which you did.
5. **Report and wait:** summarise what has merged, what is in review, and what is still draft,
   plus blockers or conflicts and the next dispatchable action. List the merged PRs whose issues
   still need closing by hand. Close only an issue whose acceptance criteria the merged PR met,
   never one it merely cross-references. Wait for the owner's explicit kickoff before dispatching
   an implementation build, unless the checkpoint records the kickoff for the active wave and
   names the next dispatchable issue. Then continue that wave without asking again. A kickoff
   authorises the scope it names and no more. A generic "carry on" resumes the recorded wave. It
   does not approve a plan still waiting on a ruling, and it does not lift a hold the owner
   placed.

Do not routinely reread the project's entry point, constitution, contributing guide, style
guides, or design set. Retrieve one when the next decision actually depends on it.

This routine may query live state, but it does not edit project files, dispatch work, merge, or
push.

## Compaction

When the project defines a compaction instruction, it is canonical. Compact at a phase boundary
while enough context remains to record decisions accurately. The checkpoint must keep volatile
state and rationale, and must not duplicate stable repository guidance.

When invoking your agent's compaction command manually (`/compact` or equivalent), request that
it follow the project's compaction instruction. The result must end by directing the resumed
thread to this skill so it verifies live state before acting.
