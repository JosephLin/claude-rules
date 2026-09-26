# Project conventions

How code projects are built. These are defaults, not laws. Deviate when a project
calls for it, and note why in its `CLAUDE.md`. Section numbers are stable, so project
docs can link to them.

## 1. Repo shape

- Git from the first commit, even for a prototype.
- The baseline `.gitignore` covers `.DS_Store`, `.env`, `.env*.local`, `*.pem`,
  `.claude/settings.local.json` (allow rules can hold secrets) and
  `.claude/worktrees/`. Generated project files are ignored, with a comment naming
  the real source of truth (e.g. `project.yml`).
- Large binaries and client data live outside git, in the `data/` tree (§2).
  Untracked files inside a repo are one `git clean -xdf` from gone.

## 2. Code vs. assets

| | Can I lose it? | Where it lives |
|---|---|---|
| Code | no, it's in history | git → a hosted remote, never inside a cloud drive |
| `data/in/`: irreplaceable inputs | no | gitignored, backed up to a cloud drive |
| `data/out/`: outputs worth keeping | costly to redo | gitignored, backed up to a cloud drive |
| `data/ref/`: slow to re-fetch | costly to redo | gitignored, backed up to a cloud drive |
| `data/work/`: churn and caches | yes | neither |

- `.gitignore` has `/data/*` with `!/data/README.md` and `!/data/.syncignore`
  allowlisted.
- Back up with an explicit push script, never a sync daemon. Anything the push would
  overwrite is archived first, and restoring never deletes anything local.
- Backup is default-include. Forgetting to exclude something wastes a little space;
  forgetting to include something loses data.
- Secrets aren't assets: they live in a password manager.
- Don't move a directory the code reads by path; exclude it with `.syncignore`.
- A folder with no project doesn't belong in the projects folder.

## 3. Docs

- `README.md` covers what it is and why it's built this way.
- `CLAUDE.md` covers how to work on it: commands, layout, invariants and gotchas.
  Keep it short, so every line matters to a session starting cold.
- `ROADMAP.md` is the dated history, added once there's a second phase.
- `TASKS.md` is the live queue (see Working style in `CLAUDE.md`).
- Keep anything an agent should derive on its own, such as eval answers or ground
  truth, out of always-loaded files.
- Multi-repo products get a workspace-level `CLAUDE.md` that owns the cross-repo
  invariants.
- Outdated docs get a "historical" banner, not a delete.

## 4. `CLAUDE.md` contents

Include what an agent would otherwise get wrong:

- commands;
- invariants;
- **`## Verified findings — don't "fix" these`**, using that exact heading so
  it's findable across repos. Each finding states its measurement, what was
  tried and failed, and any deliberate absence;
- pointers to runtime checks (`/healthz`) rather than prose that will drift.

Leave out what an agent can read from the code or already knows.

## 5. Git

- Branch off the integration branch → PR → merge (see Working style in
  `CLAUDE.md`). Prefixes: `feat/`, `fix/`, `docs/`, `ship/`,
  `redesign/`; `worktree-` marks a parallel agent session.
- **Commit subjects are declarative sentences describing the world after the
  change** (`The preview stopped putting a white tile behind the logo`, not
  `fix: logo background`). No Conventional Commits, scopes or ticket IDs. The body
  records what was disproved, with numbers. One commit per change; amend
  follow-ups. Docs-only and negative-result commits are first-class.
- Trunk is fine for an unshipped prototype. Switch to PRs before the first
  deploy.
- Projects whose releases are cut by hand get one more long-lived branch,
  `release`, and nothing else (§7).
- Don't stack PRs. `gh pr merge --delete-branch` closes open PRs based on that
  branch instead of retargeting them, and they can't be reopened. Retarget
  first.

## 6. Testing

- Extract the logic that doesn't need UI or network, and test it against golden
  values from an independent source. Verify everything else by driving the real
  app and looking at it.
- For iOS UI, DEBUG launch arguments plus screenshots work better than XCUITest.
  Build signed, or the Keychain breaks.
- For LLM output, write deterministic checks of the invariants the prompt
  promises. Ship a fake-provider switch so the project runs end to end with no
  API keys.
- Having no test suite is fine if the replacement (e.g. lint + build + click
  through) is written down.
- Run the narrowest set of tests that could fail because of this change. Benches
  aren't tests: gate them on an explicit env var, never on a file being present.
  For Xcode, that means `TEST_RUNNER_FOO=1`, because a plain export doesn't reach
  the test process.
- When a tool can be installed more than once, name the exact one: a simulator
  UDID or pinned OS rather than a device name, and `DEVELOPER_DIR` rather than
  `xcode-select`. A name-matched simulator destination once took >600s against
  30s by UDID.
- Before crediting a fix, check that the before and after measurements are the
  same workload.

## 7. CI and deploys

- CI runs exactly the command sequence you run before calling work done. Projects
  with no tests still get CI (lint + build).
- Skip runs that can't tell you anything: docs-only changes, superseded pushes,
  merge commits.
- Gate an expensive job on its preconditions (secrets present, dev backend up)
  rather than deleting it. e2e hits a dev project, never production.
- Install from the lockfile. Pin the runtime in the file both CI and the host
  read (e.g. `.node-version`).
- On the free tier, use Xcode Cloud rather than GitHub macOS runners.
- A pipeline configured outside the repo is documented inside it. If build inputs
  are gitignored, CI regenerates them in a clone hook.
- **Releases advance a branch:** `git push origin main:release`, which fails if it
  isn't a fast-forward. `git log release..main` is what's merged but not shipped.
  Let the platform own the build number. This replaced release tags in 2026-08,
  after a tag trigger failed silently and a predicted build number collided.
- A green check isn't a deploy. Confirm with `/version` and `/healthz`.

## 8. Secrets and configuration

- Secrets live in a password manager, never in a cloud drive or git. Never prefix
  a secret with `NEXT_PUBLIC_`.
- `.env.example` explains each variable: what it's for, where to get it, and what
  breaks without it.
- Prefer a security model with no privileged key (RLS, constraints). Security
  changes go in a migration, not a component.
- Log which config values are set at boot, as presence only, never values.
  Expose `/healthz` (the actual fault) and `/version` (the deployed commit).
- A config file that isn't applied says so in its header.
- Pin dependencies.

## 9. LLM cost and safety controls

- Budgets pause rather than discard: an exceeded budget should be resumable.
- Cap spend at the account level, not only per call.
- Keep free-tier and paid work on separate keys.
- When a step's model is upgraded, record why inline, so it isn't "optimized"
  back down.
- Sequence multi-step agent flows by exposing only the current step's tool, not by
  prompt instructions.
- Compact long agent conversations from accepted structured state, not raw
  history.
- Anything with a real-world consequence goes propose → preview → confirm.

## 10. Agent hygiene

- Package a repeated workflow as a skill, including what it must not do.
- Prune `settings.local.json` when a project moves.
- A background worktree agent's `preview_start` runs the main checkout's
  `.claude/launch.json` in the main checkout, so it checks pre-edit code. Start the
  dev server in the worktree on a free port and navigate to it instead.

## 11. Architecture defaults

Choices that have worked, not rules:

- A thick backend and thin client when provider keys are involved. No backend at
  all when RLS can carry security.
- Serve catalogs, pricing and model names rather than baking them into the
  client. Bump the API version with the endpoint change.
- Scripts print artifacts to stdout for you to pipe.
- Keep research as data next to what it describes, including negative findings.
- US spelling. If UI copy isn't in English, code, comments and commits still are.
