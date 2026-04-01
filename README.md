# Review Agents

Self-improving Claude Code slash commands that review plans and code from three expert perspectives: **Product**, **Design**, and **Engineering**.

## Quick start

```bash
git clone https://github.com/ericpjtsai/review-agents.git ~/Desktop/review-agents
cd ~/Desktop/review-agents

# Symlink commands into Claude Code's global commands directory
mkdir -p ~/.claude/commands
for f in commands/review-*.md; do
  ln -sf "$(pwd)/$f" ~/.claude/commands/$(basename "$f")
done
```

Then in any Claude Code session:

```
/review-product   # Product Director review
/review-design    # Product Designer review
/review-eng       # Engineering / Tech Lead review
/review-all       # All three + synthesis + verdict
/review-learn     # Capture a universal lesson
```

## What these agents do

Each agent reads the project's `CLAUDE.md`, recent git history, and any past review observations, then evaluates through role-specific lenses:

| Agent | Role | Evaluates |
|-------|------|-----------|
| `review-product` | Senior Product Director | Problem-solution fit, user flows, scope, feedback loops, prioritization |
| `review-design` | Senior Product Designer | Information architecture, interaction consistency, mobile, states, accessibility |
| `review-eng` | Senior Full Stack Engineer | Data integrity, error handling, performance, security, types, observability |
| `review-all` | All three | Runs all three independently, synthesizes top 5 issues, delivers a verdict |
| `review-learn` | Meta | Promotes a universal lesson into the appropriate agent file |

### Output format

All agents output findings as:
- **Must fix** -- blocking issues
- **Should fix** -- important gaps
- **Consider** -- future improvements

`/review-all` adds a ranked **Synthesis** and a **Verdict** (Ship it / Ship with fixes / Rethink).

## How they grow

The agents use a two-tier memory system to get smarter over time without cross-project contamination.

### Tier 1: Project memory (automatic)

After every review, agents save 3-5 observations to the project's auto-memory directory:

```
~/.claude/projects/<project>/memory/review-product.md
~/.claude/projects/<project>/memory/review-design.md
~/.claude/projects/<project>/memory/review-eng.md
```

These observations capture architecture patterns, known debt, recurring issues, and conventions. On the next review of the same project, agents read these files first -- so they remember what they've seen before and can track whether past issues were addressed.

Project memory is scoped. Reviewing Project A never sees observations from Project B.

### Tier 2: Universal lessons (manual promotion)

Each agent file has a "Lessons learned" section containing rules that apply across all projects. These are version-controlled in this repo.

When an agent flags an observation as potentially universal, run `/review-learn` to evaluate and promote it. The lesson gets appended to the agent file, and a PostToolUse hook auto-commits the change locally.

```
Review project -> auto-save project observations -> flag universal candidates
                                                        |
                                        run /review-learn to promote
                                                        |
                                            hook auto-commits locally
                                                        |
                                         push when ready (manual)
```

## Auto-commit hook

Add this to `~/.claude/settings.json` to auto-commit agent file changes:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | { read -r f; resolved=$(readlink \"$f\" 2>/dev/null || echo \"$f\"); if [[ \"$resolved\" == /path/to/review-agents/commands/review-*.md ]] || [[ \"$f\" == /path/to/review-agents/commands/review-*.md ]]; then cd /path/to/review-agents && git add -A && git diff --cached --quiet || git commit -m \"Update $(basename \"$resolved\"): add lesson learned\"; fi; }",
            "timeout": 15
          }
        ]
      }
    ]
  }
}
```

Replace `/path/to/review-agents` with the actual path to your clone.

The hook commits but does not push -- you review accumulated lessons and push when ready.

## Customization

### Adding a new agent

1. Create `commands/review-{name}.md` following the pattern of existing agents
2. Include Step 0 (context gathering + memory read), evaluation lenses, anti-patterns, lessons learned, memory save, and output format sections
3. Symlink it: `ln -sf "$(pwd)/commands/review-{name}.md" ~/.claude/commands/review-{name}.md`

### Editing evaluation criteria

Edit files in `commands/` directly. Changes take effect immediately across all projects (symlinks). The hook auto-commits locally.

### Lessons format

Lessons in each agent file follow this structure:

```markdown
### [Concise rule title]
[Why it matters -- the consequence of ignoring it. When it applies -- the trigger condition.]
```

`/review-learn` enforces quality criteria: lessons must be generalizable, actionable, earned from real experience, and concise.

## File structure

```
review-agents/
  CLAUDE.md                        # Project instructions for Claude Code
  README.md                        # This file
  commands/
    review-product.md              # Product Director agent
    review-design.md               # Product Designer agent
    review-eng.md                  # Engineering / Tech Lead agent
    review-all.md                  # Combined review orchestrator
    review-learn.md                # Universal lesson capture
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or IDE extension
- `jq` (for the auto-commit hook)
