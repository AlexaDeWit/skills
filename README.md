# Skills

Personal agent skills, one directory per skill with a `SKILL.md`, following the
[Agent Skills](https://agentskills.io/specification) standard.

## Origin and current state

These four skills are verbatim copies of the skills in the
[Écluse](https://github.com/AlexaDeWit/npm-secure-proxy) repository, ported here on
2026-08-29. They still carry Écluse-specific content: relative links into that repository,
`task` targets, CI job names, and project roles.

The plan is to make the general working style project-agnostic here, and leave only the
procedural project checks (build targets, gate names, repository rules) in each project.
That split has not happened yet.

## Skills

- `open-pull-request`: commit and PR procedure (signing, DCO, trailers, draft flow).
- `orientation`: bootstrap a cold task session by retrieval, not ingestion.
- `orchestrate-implementation`: the team-lead per-PR loop for dispatching implementer and
  reviewer subagents.
- `resume-orchestration`: restore the team-lead seat after compaction or restart.

## Use

Claude Code discovers personal skills from `~/.claude/skills/`. Symlink a skill directory
there to enable it:

```bash
ln -s ~/Workspace/skills/<skill-name> ~/.claude/skills/<skill-name>
```
