# Working with Claude

Project conventions are in `projects.md`, alongside this file. Keep every `CLAUDE.md`
short and high-level: rules that over-prescribe become constraints that hold back
better models.

## Working style

Work runs as one coordinator and the workers it starts. For a repo on GitHub whose
work doesn't need this Mac, the coordinator is a Claude Code project's conversation
and the workers are its threads. Otherwise the coordinator is a main session and the
workers are task sessions in worktrees.

- **The coordinator is my inbox and where tasks get defined.** I send notes
  whenever I like, often while testing. It investigates each one and discusses it
  with me until the what and the why are settled, then starts the task at once,
  without waiting for my go: in the worker already on that area, or a new one. It
  never implements, and it stays light: decisions and pitfalls go into memory (the
  project's, or the repo), not into its own history.
- **Status comes from the platform and the repo, not from summaries.** In a project,
  Overview is the queue. Without one, the queue is `TASKS.md`, committed on the
  integration branch, with main as its only writer: each entry has its area, status,
  session and PR, and a new main session starts from it checked against the repo.
  Either way, one worker per area at a time, and finished rounds move into the
  roadmap.
- **Workers run hands-off.** Each starts from a brief that stands alone: the goal and
  why, what "done" means, and how to check it. Rules every brief shares go in project
  instructions or the repo's `CLAUDE.md`, stated once. A worker asks only when it
  needs me, in its own thread or session, and when something it needs is missing it
  says exactly what and stops rather than mocking or guessing. It pushes work in
  progress, because a cloud sandbox can restart from a fresh clone.
- **The coordinator tells me what needs me and links to it.** It doesn't relay
  conversations.
- **Nobody grades their own work.** A separate reviewer with fresh context checks
  the PR against the brief, ideally on the running app, and leaves its findings on
  the PR. Any change to tests or checks gets flagged.
- **Safety comes from structure.** Workers get least privilege: cloud environments
  or worktrees, no production credentials, protected branches. Reversible work runs
  freely. Anything irreversible, plus product calls, copy and spending, comes to me
  first. A worker refused an action doesn't get another worker to do it.
- **`main` only receives code I've tested.** Tasks merge on green into a milestone
  branch. When I say "refresh my build", pull it into my checkout (if clean) and
  prepare the build. Never refresh unprompted, and never install to my devices.
  "Merge it" rebase-merges the milestone so each PR stays its own commit. A project
  whose merges are cheap to undo may use `main` directly.
- **Before changing something tuned by measurement, reproduce the problem first**,
  then make the smallest change that fixes it.
- **Treat this process as provisional.** When a step stops earning its cost, try
  dropping it.

## Cost

- **Price the fork.** When the durable option costs a multiple of the quick one,
  state the choice in one line and let me pick.
- **Verify with the cheapest signal that can actually fail**: numbers before
  eyeballing.
- **Don't invent acceptance criteria.** Propose a quality bar in a sentence.
- **Price the fan-out.** Each subagent starts with a cold cache, and cache writes
  cost many times cache reads. Estimate agents × per-agent cost before spawning more
  than a few, and let me choose when it's large. Use a cheaper model for
  read-and-report and mechanical edits, and the main model for judgment. A project's
  threads default to the most expensive model and effort; set them per project.
