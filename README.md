# Skills

Personal agent skills, one directory per skill with a `SKILL.md`, following the
[Agent Skills](https://agentskills.io/specification) standard.

## How these are used

This repository is the single management home for personal skills. `~/.claude/skills` is a
symlink to this clone, so every skill directory here is live at the personal level in Claude
Code. Personal skills override same-named project skills, so each skill stays project-agnostic
and defers to the project's own documents (CONTRIBUTING, testing docs, orchestration reference)
for concrete facts: build targets, gating job names, review requirements.

To add a skill: create `<skill-name>/SKILL.md`, commit, and it is live.

## Skills

- `adopt-orchestration`: onboard a repository to the orchestration process by auditing it
  against the contract the other skills defer to.
- `execute-runbook`: execute a project-local runbook step by step, stopping at the steps a
  human must own.
- `open-pull-request`: commit and PR procedure (signing, DCO, trailers, draft flow).
- `orientation`: bootstrap a cold task session by retrieval, not ingestion.
- `orchestrate-implementation`: the team-lead per-PR loop for dispatching implementer and
  reviewer subagents.
- `resume-orchestration`: restore the team-lead seat after compaction or restart.

## Origin

Ported on 2026-08-29 from the [Écluse](https://github.com/AlexaDeWit/npm-secure-proxy)
repository, then made project-agnostic. The Écluse-specific facts the originals carried now
live in that repository's own documents.
