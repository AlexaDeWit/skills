# CI gate mechanics

The procedure for driving a pull request's checks to a terminal result, and for trusting that
result afterwards. `orchestrate-implementation`, `implement-ticket`, and `resume-orchestration`
all defer here. The project's testing or contributing document names the gating jobs and the
informational contexts. It wins on any conflict.

## Before you wait

1. **Check that the pull request can build a merge ref.** Run `gh pr view <pr> --json mergeable`.
   A `CONFLICTING` pull request gets no `pull_request` workflow run at all, so only the DCO or
   equivalent commit-level check ever appears. Closing and reopening does nothing. Rebase it. Only
   DCO after a push is the signal to check `mergeable` first, before starting any watch.
2. **Check for an in-flight run before pushing.** Run `gh run list --branch <branch>`. A push
   cancels the branch's running workflow, and a cancelled run is not a signal about the code. When
   the push must happen anyway, for a rebase, a re-sign, or a fix, push at once. Waiting only adds
   delay.
3. **Check that the pull request is still open.** Run `gh pr view <pr> --json state,mergedAt`
   before any follow-up push. An owner who merges quickly can squash-merge between a review verdict
   and its fix commit, which strands the fix on a branch with no open pull request. Git's "Create a
   pull request for ..." hint after a push is the red flag. When it has merged, branch off the
   remote default branch, cherry-pick the fix with `-x`, confirm `git diff <default> HEAD` is
   exactly the intended change, and open a new pull request saying the content was already
   reviewed.

## Starting the watch

A watch started at the first push often returns before the checks exist. Guard the start, then
guard the loop.

- Poll plain `gh pr checks <pr>` about every 20 seconds while it answers "no checks reported".
  Only that message retries.
- Wait for the project's gating context to exist, not merely for some check to appear. A fresh
  push shows the commit-level checks first, and a watch started there exits at once with a
  meaningless pass.
- A mergeable pull request's run can still take 30 to 40 minutes to attach. Allow an hour before
  treating an absent run as a fault.
- Re-enter the watch while any check is `pending`, `in_progress`, or `queued`. `gh pr checks
  --watch` exits at push boundaries, and a watch running across a force-push usually exits instead
  of following the new head.
- A red exit is terminal. Do not retry it. Route the failing run's log back to whoever owns the fix.
- Verify the final state with `gh pr checks`, never with `gh run watch`'s exit code, which can exit
  0 on a failed run.

## Keeping the watch alive

A background watcher is not a durable signal. Host suspend, host memory pressure, and a dropped
API connection each kill one with no notification, and an agent that dies mid-call leaves no
journal entry. Silence is indistinguishable from progress.

- Prefer a session-owned persistent poll over a one-shot background watch when the harness offers
  one. Poll every few minutes, emit only terminal state changes, and ignore the informational
  contexts.
- Never depend on a single long-lived watcher, and never depend on a subagent's terminal report
  for timing. Both race the coordinator's own sweeps and lose.
- After any gap, a suspend, a restart, a session resume, or a long silence, sweep `gh pr checks`
  across every open pull request directly instead of waiting for a notification.
- Check liveness, not just results. When a dispatched agent looks slow, inspect its transcript or
  process modification time and read the tail of anything quiet for more than about 30 minutes. In
  Claude Code, recover a stalled workflow with `TaskStop` and then `resumeFromRunId`: unchanged
  prompts replay from cache and stalled calls re-run live. Tell the re-run agent that its worktree
  holds uncommitted partial work to keep and finish, and to amend rather than add a second commit.

## After a merge

A merge is a statement about the branch, never about CI. An owner with admin rights can merge
ahead of the gate, and the final run then reports failure minutes later against a default branch
nobody is watching.

- Check the conclusion of the merged head's run (`gh run list --commit <head>`) before calling a
  wave clean. A red there is a broken default branch that needs a fix pull request, not a rerun.
- Treat `BEHIND` as noise. A required up-to-date rule usually targets outside contributors, and an
  owner merges through it. It is never a hand-off blocker and never on its own a reason to rebase.
