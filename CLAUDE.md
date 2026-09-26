# Working with Claude

Project conventions are in `projects.md`, alongside this file. Keep every `CLAUDE.md`
short and high-level: rules that over-prescribe become constraints that hold back
better models.

## Working style

- **The main session is my inbox and where tasks get defined.** I send notes
  whenever I like, often while testing. Main investigates each one and discusses it
  with me until the what and the why are settled. Then it starts the task at once,
  without waiting for my go. It hands the task to the session already working on
  that issue, or starts a new task session. Main never implements.
- **The queue is `TASKS.md`, committed on the integration branch; main is its only
  writer.** Notes go in when they arrive. Each entry has its area, status (being
  defined, queued, active, waiting on me, done), session and PR, plus any open
  questions. Task status comes from PRs. A new main session starts from `TASKS.md`
  checked against the repo. Only one writer works on an area at a time. When a
  round lands, finished tasks move into the roadmap.
- **Task sessions run hands-off.** Each one works from a brief that stands alone:
  the goal and why, what "done" means, and how to check it. A task session asks only
  when it needs me: in its own session, with its PR marked as waiting on me.
- **Main tells me what needs me and links to it.** It doesn't relay conversations.
  Status comes from the repo, not from session summaries.
- **Nobody grades their own work.** A reviewer with fresh context checks the result
  against the brief, ideally on the running app, and leaves its findings on the PR.
  Any change to tests or checks gets flagged.
- **Safety comes from structure.** Task sessions get least privilege: isolated
  environments, no production credentials, protected branches. Reversible work runs
  freely. Anything irreversible, plus product calls, copy and spending, comes to me
  first. A session refused an action doesn't get another session to do it.
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
  read-and-report and mechanical edits, and the main model for judgment.
