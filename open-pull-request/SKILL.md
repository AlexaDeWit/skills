---
name: open-pull-request
description: >-
  Prepare a commit and pull request the portable way: GPG-signed, DCO-signed-off
  as the human author (never the AI), AI help disclosed with Assisted-by, the
  body short, and the PR opened as a draft. Invoke it before you commit and
  before you open or finalise a PR. The project's CONTRIBUTING file owns the
  standards and wins on any conflict.
---

# Open a pull request

The commit and PR mechanics that hold on every project. **The project's `CONTRIBUTING.md` (or
equivalent) owns the standards**: the commit format, the review process, and what must pass before
a PR is ready for review. Read it before you commit. Where this file and the project disagree, the
project wins.

## 1. Commits

Commit with both signing flags, every time. Sign off as you go, on every commit:

```
git commit -S -s -m "<subject>" -m "<body…>" -m "Assisted-by: <Agent Name> (<Vendor>)"
```

- **Do not hand-write a `Signed-off-by:` line.** `-s` derives it from the git identity, which is
  the human author. Hand-writing it is how an AI name slips in, and a DCO check gates on a trailer
  matching the commit author's email.
- **`Assisted-by:`, never `Co-Authored-By:`.** Git tooling and editor snippets reach for the
  latter by default, and it asserts joint authorship.
- **This machine's git has no `--trailer` flag.** Put trailers in literal `-m` lines, or in a `-F`
  message file. `--trailer` errors here.
- Follow the project's subject format (Conventional Commits where the project requires it).

## 2. The PR body

Follow the project's PR template. The body is three things, readable in seconds: the goal, the
motivation, the consequence. Lead with what a reviewer or operator gains or is protected from,
in words a reader outside the subsystem understands on their own.

- Never narrate the diff, and never restate a checklist or a gate CI already enforces.
- Never paste inventories, counts, tables, or verification appendices into the body. That audit
  trail belongs in the commit message of the change that produced it, or in the report to
  whoever dispatched the work.
- A deviation from the issue, or an operator-visible behaviour change, is one sentence each.
- A review or fix round rewrites the body only when the goal or a consequence changed. It never
  appends rounds, findings, or fix logs.
- The AI-assistance line names the actual tool (`Assisted-by: <Agent> (<Vendor>)`, filled in).
  Never leave a template's placeholder verbatim: a disclosure that names no tool discloses
  nothing, and reads as a template artefact to the reviewer.

## 3. Draft until hand-off

- **Open as a draft:** `gh pr create --draft …`. It stays draft while work or review is moving.
- **Pipe the body through stdin. Never write it to a repo-root file.** A root scratch file
  collides across concurrent agents and worktrees, and an agent stages it by accident:

  ```
  gh pr create --draft --title "<subject>" --body-file - <<'EOF'
  <body…>
  EOF
  ```

  `gh pr edit --body-file -` updates it the same way. If the body must exist as a file, put it in
  a gitignored scratch location with a branch-scoped basename (`pr-body-<branch>.md`). Concurrent
  agents share scratch space, so a bare `body.md` puts one PR's body on another and reports
  success. Never commit it.
- **Flip to ready only when the project's review requirement and its gating CI both pass.** The
  project's contributing or testing docs name the gating jobs and which contexts are
  informational. The instant both hold, run `gh pr ready`.
- **Verify the gate with `gh pr checks`, not with `gh run watch`'s exit code**, which can exit 0
  on failure.

## 4. If the DCO check goes red

Interactive rebase is unavailable here. Re-sign non-interactively with `git commit-tree -S`, and
preserve each commit's tree. Walk the branch commits. Re-create each one with the corrected
`Signed-off-by:` (and `Assisted-by:`) trailer and a GPG signature, then move the branch ref to the
new tip. Confirm that `git diff` against the old commits is empty before you force-push the
feature branch. Force-pushing a feature branch is fine. Never force-push the default branch.
