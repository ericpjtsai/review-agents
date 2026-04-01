Run a **three-role review** of the current plan or proposed changes. Each role evaluates independently with full depth, then synthesize.

## Step 0: Gather project context (do this FIRST, silently)
- Read CLAUDE.md if it exists
- Run `git log --oneline -20` for recent changes
- Identify the tech stack, design system, and target users
- Check for past review memory: look for `review-product.md`, `review-design.md`, and `review-eng.md` in the project's auto-memory directory. If any exist, read them — they contain observations from previous reviews of this project.

## Step 1: Product Director review
Read `~/.claude/commands/review-product.md` for the full evaluation framework.
Focus on: problem-solution fit, flow completeness, feedback loops, scope management, mental model, prioritization. Flag anti-patterns (over-engineering, admin UIs for rare changes). Apply lessons learned.

## Step 2: Design review
Read `~/.claude/commands/review-design.md` for the full evaluation framework.
Focus on: information architecture, interaction consistency, mobile responsiveness, loading/error/empty states, input validation, accessibility, visual consistency. Flag pattern violations. Apply lessons learned.

## Step 3: Engineering review
Read `~/.claude/commands/review-eng.md` for the full evaluation framework.
Focus on: data integrity (race conditions, dedup), error handling, performance, security (injection, auth), duplication, migration safety, type safety, observability. Flag anti-patterns. Apply lessons learned.

## Output format

For each role, provide:
- **Must fix** — blocking issues
- **Should fix** — important gaps
- **Consider** — future improvements

### Synthesis
The **5 most important changes** across all three roles, ranked by impact:
1. What the issue is
2. Which role(s) flagged it
3. The recommended fix
4. Effort estimate (trivial / small / medium / large)

### Verdict
- **Ship it** — plan is solid, proceed
- **Ship with fixes** — proceed but address Must Fix items first
- **Rethink** — fundamental issues need a different approach
