Run a **three-role review** of the current plan or proposed changes. Each role evaluates independently with full depth, then synthesize.

## Step 0: Gather project context (do this FIRST, silently)
**Read past review memory FIRST**: Look for `review-product.md`, `review-design.md`, and `review-eng.md` in the project's auto-memory directory. If any exist, read them — they contain observations from previous reviews of this project. Use these to track whether past issues have been addressed.

Then:
- Read CLAUDE.md if it exists
- Run `git log --oneline -20` for recent changes
- Identify the tech stack, design system, and target users

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

## Step 4: Save observations (MANDATORY — do this after every review)

You MUST complete this step before finishing. Save project-specific observations from each role to their respective memory files in the project's auto-memory directory:
- `review-product.md` — 3-5 product observations
- `review-design.md` — 3-5 design observations
- `review-eng.md` — 3-5 engineering observations

Each file uses the same format:
```markdown
---
name: [Role] review observations
description: [Role-specific description]
type: project
---

- [observation 1]
- [observation 2]
```

If memory files already exist, update them — merge new observations, remove stale ones, don't duplicate.

If any observation seems universal, note it at the end and suggest running `/review-learn` to promote it.

## Step 5: Learn from feedback (AUTOMATIC)

If the user corrects, disagrees with, or refines any finding during this conversation, evaluate the correction:

1. **Is it universal?** Does this correction apply beyond this specific project?
2. **If universal**: Identify which role(s) it applies to (product, design, engineering) and append a concise lesson to the "Lessons learned" section of the corresponding file(s) under `~/.claude/commands/`. Verify it doesn't duplicate an existing lesson.
3. **If project-specific**: Update the corresponding project memory file (`review-product.md`, `review-design.md`, or `review-eng.md`) instead.

Do this automatically whenever feedback is received — do not wait for the user to run `/review-learn`.
