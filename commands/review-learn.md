Capture a **universal** lesson from the current session and append it to the appropriate review agent.

Note: This command is for lessons that apply across all projects. Project-specific observations are saved automatically by each review agent to the project's auto-memory directory — you don't need `/review-learn` for those.

## Process

1. **Ask**: What happened? What was the mistake, correction, or validated decision?

2. **Classify** which role(s) the lesson applies to:
   - **Product** — scope, prioritization, user flow, feature decisions
   - **Design** — UI patterns, interaction consistency, mobile, accessibility
   - **Engineering** — bugs, architecture, performance, security, tech debt

3. **Formulate** the lesson as:
   - A concise rule (1 sentence)
   - Why it matters (the consequence of ignoring it)
   - When it applies (the trigger condition)

4. **Append** to the "Lessons learned" section of the appropriate file(s):
   - `~/.claude/commands/review-product.md`
   - `~/.claude/commands/review-design.md`
   - `~/.claude/commands/review-eng.md`

5. **Verify** the lesson doesn't duplicate an existing one. If it refines an existing lesson, update it instead of adding a new one.

## Example lessons

**Product**: "Features that show the same data on two pages always get merged. Plan for co-location from the start."
→ Trigger: whenever a new page is proposed that displays data already shown elsewhere.

**Design**: "Sticky/fixed elements need 2-3 rounds of fixes for width, scroll, and z-index. Budget for iteration."
→ Trigger: whenever a sticky header, fixed nav, or floating element is proposed.

**Engineering**: "Timezone bugs hit every date feature. Use explicit timezone, not server-local time."
→ Trigger: whenever a feature involves "today", "this week", or date-relative queries.

## Quality criteria for a good lesson
- **Generalizable** — applies to future projects, not just this one. If it only applies to the current project, it belongs in the project's auto-memory, not here.
- **Actionable** — tells you what to DO, not just what to avoid
- **Earned** — came from a real mistake or validated decision, not theory
- **Concise** — one sentence for the rule, one sentence for why
