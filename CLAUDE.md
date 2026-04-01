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

## Lessons learned

Each agent accumulates lessons in its "Lessons learned" section. Use `/review-learn` to capture new lessons from a session. Lessons are:
- **Generalizable** -- applies beyond the current project
- **Actionable** -- tells you what to do, not just what to avoid
- **Earned** -- came from a real mistake or validated decision

## Editing agents

Edit the files in `commands/` directly. Changes take effect immediately in all projects since the global commands are symlinks. Commit changes here to track the evolution of the review system.

## Setup on a new machine

```bash
cd ~/Desktop/review-agents
for f in commands/review-*.md; do
  ln -sf "$(pwd)/$f" ~/.claude/commands/$(basename "$f")
done
```
