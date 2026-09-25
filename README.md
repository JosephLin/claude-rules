# claude-rules

The standing instructions I give Claude Code, written to be short and high-level so
they don't hold back newer models.

- [`CLAUDE.md`](CLAUDE.md): how work with Claude runs. A main session defines tasks,
  hands-off task sessions carry them out, there's a `TASKS.md` queue and one writer
  per area, reviews come from fresh context, and safety comes from structure.
- [`projects.md`](projects.md): how code projects are built: git, docs, testing,
  CI, secrets and cost controls.

Each rule exists because its absence cost something. The reasoning behind the
working style is summarized in the commit history.

## Using them

**Locally**, import them from your user-level `~/.claude/CLAUDE.md`, next to anything
private to your machine:

```markdown
@~/path/to/claude-rules/CLAUDE.md
```

For project conventions, import `projects.md` from a `CLAUDE.md` in your projects
folder.

**In Claude Code cloud sessions**, which see only the repo they clone, add this to
the cloud environment's setup script:

```bash
mkdir -p ~/.claude
{ curl -fsSL https://raw.githubusercontent.com/JosephLin/claude-rules/main/CLAUDE.md
  echo; curl -fsSL https://raw.githubusercontent.com/JosephLin/claude-rules/main/projects.md
} > ~/.claude/CLAUDE.md || true
```

Suggestions and disagreements are welcome as issues or PRs, especially ones backed
by a measurement.

## License

[CC BY 4.0](LICENSE): copy, adapt and use them, including commercially, as long as
you credit this repo and say whether you changed anything. Lifting a few bullets
into your own `CLAUDE.md` with a link back is enough.
