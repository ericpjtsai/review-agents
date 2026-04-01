# Review Agents

A collection of Claude Code slash commands that act as specialized review agents for software projects.

## What this is

Five review agents that evaluate plans, code, and proposed changes from different expert perspectives. They work as global Claude Code slash commands (`/review-*`) and can be used in any project.

## Agents

| Command | Role | Focus |
|---------|------|-------|
| `/review-product` | Senior Product Director | Problem-solution fit, user flows, scope, prioritization |
| `/review-design` | Senior Product Designer | IA, interaction consistency, mobile, states, accessibility |
| `/review-eng` | Senior Full Stack Engineer | Data integrity, error handling, performance, security, types |
| `/review-all` | All three roles | Runs all three reviews, then synthesizes top 5 issues + verdict |
| `/review-learn` | Meta | Captures lessons from the current session into the appropriate agent |

## How it works

Each agent file lives in `commands/` and is symlinked into `~/.claude/commands/` so Claude Code picks them up as global slash commands. The source of truth is this repo.

```
~/.claude/commands/review-product.md  ->  ~/Desktop/review-agents/commands/review-product.md
~/.claude/commands/review-design.md   ->  ~/Desktop/review-agents/commands/review-design.md
~/.claude/commands/review-eng.md      ->  ~/Desktop/review-agents/commands/review-eng.md
~/.claude/commands/review-all.md      ->  ~/Desktop/review-agents/commands/review-all.md
~/.claude/commands/review-learn.md    ->  ~/Desktop/review-agents/commands/review-learn.md
```

## Output format

All agents output findings in three tiers:
- **Must fix** -- blocking issues
- **Should fix** -- important gaps
- **Consider** -- future improvements

`/review-all` adds a **Synthesis** (top 5 ranked issues) and a **Verdict** (Ship it / Ship with fixes / Rethink).

## Two-tier memory system

The agents grow smarter over time through two independent memory layers:

### Project-specific memory (automatic)
Each agent automatically saves 3-5 observations to the project's auto-memory directory after every review. These capture architecture patterns, known debt, recurring issues, and conventions specific to that project. Next time the agent reviews the same project, it reads these observations for context.

Memory files are stored at `~/.claude/projects/<project>/memory/review-{role}.md` and are scoped per-project -- no cross-contamination between projects.

### Universal lessons (manual promotion)
Lessons that apply across all projects live in the "Lessons learned" section of each agent file. Use `/review-learn` to promote a project-specific observation into a universal lesson. These are version-controlled in this repo.

**Flow**: Agent reviews project -> saves project observations (auto) -> flags universal candidates -> user runs `/review-learn` to promote (manual) -> hook auto-commits locally -> user pushes when ready.

## Editing agents

Edit the files in `commands/` directly. Changes take effect immediately in all projects since the global commands are symlinks. A PostToolUse hook auto-commits changes locally. Push manually after reviewing accumulated lessons.

## Setup on a new machine

```bash
cd ~/Desktop/review-agents
for f in commands/review-*.md; do
  ln -sf "$(pwd)/$f" ~/.claude/commands/$(basename "$f")
done
```
