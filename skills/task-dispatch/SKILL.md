---
name: task-dispatch
description: How the main session hands a settled task to a hands-off builder, gets it reviewed with fresh context, and merges it on green. Use when main starts, re-routes, reviews or merges a task from TASKS.md, or closes a round. Not for doing the task in main.
---

# Task dispatch

The rules and their reasons are in `CLAUDE.md` (Working style). This skill is
the how: templates and the loop. Project specifics (setup commands, data paths,
real inputs) come from the project's own `CLAUDE.md`, not from here.

## Before dispatching

- **The task is settled in `TASKS.md`**: area, why, done, check, status. Only one
  writer works on an area at a time. If two tasks touch the same code, queue the
  second behind the first's merge. Never stack PRs.
- **An integration branch exists and is pushed**, e.g. `milestone/<x>` off `main`.
  PRs need a hosted remote; creating one is the owner's call.
- **Price the fan-out first** (agents × cost per agent). Past the owner's
  threshold, give the options in one line and let them pick. Builders run on the
  main model for judgement or measurement work, and on a cheaper model for
  mechanical edits. A cheaper model is usually enough for reviewers. Calibration
  from one round (Oct 2026), so re-check with the owner's cost tool:

  | Agent | Cost |
  |---|---|
  | Builder on a measurement-heavy task (150-200 tool calls, ~300k context) | $25-30 |
  | Small, focused builder told to be economical | ~$7 |
  | Builder resumed with a large context for a fix round | ~$20+ |
  | Reviewer on a cheaper model | ~$1.50 |

  Cost is mostly cache reads of a long context, so lean briefs and lean runs
  matter more than which model you pick.

## Builder brief

Spawn it in the background, in an isolated worktree. Fill in every `<…>`. The
brief must stand alone: the builder has none of this conversation.

```
You are the task session for **<Tn: title>** in <project>, <what the project is, one line>.
Work hands-off from this brief until a PR is open. Read CLAUDE.md, README.md and the <Tn>
entry in TASKS.md first.

## Goal and why
<the problem, the evidence, who asked, and what they saw>

## Done means
<observable outcomes; what must not change (defaults, other modes, public interfaces);
which docs to update, including a measured design note and any failed attempts under
"Verified findings">

## How to check (numbers before eyeballing)
- <measurements on real data, which metric, which cases>
- Proposed quality bar: <bar>. If the data says the bar is wrong, say so with numbers
  instead of bending it.
- Add a test that fails on the old code. Run the full suite. Call out any change to an
  existing test, threshold or fixture in the PR body.
- <artifacts for the owner to judge, e.g. side-by-sides in an output folder>

## Environment
- The worktree lacks gitignored files (<data/, .venv, .env…>). Set up: <command>. Don't
  use the main checkout's environment: editable installs point at the main checkout's code.
- Real inputs: <absolute path>. Read-only: never write, move or delete there. Write runs
  to <the worktree's scratch dir>. Other sessions share this machine: <concurrency limit>.
- To drive the running app, start it from the worktree on a free port and navigate to
  it. Never use preview_start with a launch config, which runs the main checkout's code.
- Running in parallel: <other tasks and their files>. Keep changes localized: a new
  module plus small hooks.
- Be economical: run a few well-chosen measurements and keep the context lean.

## Git and PR
- Branch <prefix/name> off origin/<integration>. Open the PR into <integration>. Never
  push to main or <integration>, and never merge.
- Commits follow the project's conventions, one commit per change with follow-ups
  amended, ending with <attribution line>.
- PR body: what changed, the measurements, how to verify, any test changes, and
  <attribution line>.
- If you're blocked on a decision only the owner can make (a product call, copy,
  spending, anything irreversible), open the PR as a draft with a "## Waiting on
  <owner>" section stating the question and the options, then finish.

## Final report (short)
PR URL, key numbers, test count, test changes, open questions.
```

## Reviewer brief

When the PR opens, spawn a **fresh** agent, never the builder, in the background
in a worktree. Give it the builder's brief verbatim, not your opinion of the PR.

```
You are an independent reviewer with fresh context. Review <PR URL> (<branch> into
<integration>) for <project>. Check it against the brief below, not the PR's claims,
and re-measure instead of trusting numbers.

## The brief the builder was given
<Goal / Done / Check, verbatim>
The builder reports: <headline claims and numbers>.

## How to review
1. `git fetch origin && git checkout --detach origin/<branch>`. (`gh pr checkout`
   fails while the builder's worktree holds the branch.) Set up, then run the suite.
   Read the diff. Flag every change to pre-existing tests, thresholds or fixtures, and
   judge whether each is legitimate.
2. Read the code for <the specific risks you can name>.
3. Re-measure on real data (read-only) with your own computation. Include cases the
   builder didn't pick. For a UI, drive the running app.
4. Keep it proportionate. Blocking means wrong behaviour, a visible artifact, a broken
   brief item, an unjustified test change, or committed secrets or large binaries.
   Everything else is non-blocking.

## Output
Post one PR comment starting with `**Review: approve**` or `**Review: changes
requested**`. List blocking items, then non-blocking items, then what you verified,
with numbers. Don't push, merge or use `gh pr review` (it's the same account).
Final report: the verdict plus 3-6 lines.
```

## After the reviewer reports

- **Approve:** run `gh pr merge <n> --rebase --delete-branch`, pull the integration
  branch, and run the suite there. Mark the task done in `TASKS.md` with the PR link
  and a one-line result. Non-blocking items go to the Later list.
- **Changes requested:** resume the builder, which still has its context, with the
  comment link, the blocking items and any rebase needed. Then run a narrow
  re-review by resuming the same reviewer.
- **After every merge**, check the other open PRs (`gh pr view <n> --json
  mergeable`). For a conflicting PR, its builder rebases onto the integration branch,
  keeps both features, re-runs the suite and pushes with `--force-with-lease`. If
  that PR's review is running, wait and combine the rebase with the review fixes.
- **Main writes `TASKS.md` only on the integration branch** and commits each status
  change, so status lives in the repo.
- **Tell the owner** what landed (with numbers), what needs them (with a link), and
  spend against the estimate.

## Closing a round

The owner tests the integration branch. On "merge it", rebase-merge or
fast-forward `main` to it, so each PR stays its own commit. Add a dated
`ROADMAP.md` entry with PR links and headline numbers, and trim `TASKS.md` to the
open work. Remove finished worktrees and agent branches, after checking that
`git log main..<branch>` is empty for each.

## Must not

- Main writes the task's code. Setup and bookkeeping are fine.
- A builder or reviewer merges, pushes to `main` or the integration branch, writes
  to the owner's real inputs, or changes defaults or product copy unprompted.
- The builder reviews its own work, or a reviewer is told what the answer should be.
- A dependent task is stacked on an unmerged PR. Queue it instead.
- A session that was refused an action gets another session to do it.
- A large-context builder is resumed for a small unrelated task. Start a fresh one.
