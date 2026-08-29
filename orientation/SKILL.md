---
name: orientation
description: >-
  Bootstrap a cold task session with minimal context. Read the project entry
  point and the task contract, route to only the relevant sources, verify the
  necessary live state, and report the working picture. Do not use for an
  orchestration-seat resume when a resume skill applies.
---

# Orientation

Orient by retrieval, not ingestion. Repository files are durable memory. The conversation should
hold only the current task contract, the applicable invariants, and volatile evidence.

## Procedure

1. **Read the entry point:** read `README.md`. Agent harnesses load the project constitution
   (`AGENTS.md` or their native context file) at session startup. Do not reread it unless someone
   disputes an exact passage.
2. **Classify the task:** implementation, review, architecture, operations/configuration,
   documentation, release/CI, or orchestration. Identify the authoritative task contract: the user
   request, issue, or PR.
3. **Route context:** follow the project's context-routing table when it has one. Otherwise follow
   the references in `README.md`. Read only the files and sections the next decision needs. Design
   documents describe target intent. Code, git, and status records establish shipped state.
4. **Verify necessary live state:** inspect the branch, working tree, relevant recent commits, and
   relevant PR or issue state. Do not enumerate every issue, document, or worktree unless the task
   requires it. Do not checkout, pull, edit, or dispatch as part of orientation.
5. **Use memory selectively:** if `MEMORY.md` exists, read its index only when the task depends on
   prior user feedback or decisions. Follow only relevant links and verify volatile claims live.
6. **Report briefly:** state the objective, current shipped or in-flight state that matters,
   applicable invariants, sources selected, unresolved questions, and next action.

Before writing code or prose, retrieve the project's style guides. Before committing or opening a
PR, invoke `open-pull-request`. Do not preload its instructions at startup.

For an orchestration seat after restart or compaction, use `resume-orchestration` instead. Never
run both skills for the same startup.

This skill only builds a bounded working picture. It changes no files and dispatches no work.
