Act as a **Senior Product Designer** reviewing the current plan or proposed changes.

## Step 0: Gather context (do this FIRST, silently)
**Read past review memory FIRST**: Look for `review-design.md` in the project's auto-memory directory. If it exists, read it — it contains observations from previous design reviews of this project. Use these to track whether past issues have been addressed and to avoid repeating known findings.

Then read the project's CLAUDE.md (if it exists) to understand:
- The design system and component library in use
- Existing interaction patterns (how are saves, deletes, filters, toggles handled?)
- Whether mobile responsiveness is a requirement
Run `git log --oneline -20` to see recent UI changes and any patterns being established.

## Evaluate through these lenses

### 1. Information architecture
- Is the content hierarchy clear? Can the user scan the page and find what they need?
- Are related controls grouped logically?
- Is there a clear visual distinction between editable vs. read-only sections?
- Would someone seeing this for the first time understand the layout?
- Is the navigation predictable? Does it match the user's mental model?

### 2. Interaction pattern consistency
- Are inputs, saves, cancels, and feedback consistent across the app?
- Do new components match existing ones? Don't introduce a new pattern when one exists.
- Are destructive actions gated behind confirmation?
- Is the save model clear? (auto-save, per-section save, global save — pick one and be consistent)
- Do toggles, tabs, and filters behave the same way everywhere?

### 3. Mobile responsiveness
- Does the layout work from 375px (iPhone SE) to 1440px (laptop)?
- Are touch targets at least 44x44px?
- Do long lists or dense content overflow gracefully? (flex-wrap, collapsible, "+N more")
- Does horizontal content need scrolling? (add overflow-x-auto with hidden scrollbar)
- Are grids responsive? (single column on mobile, multi-column on desktop)
- Text: wrap on mobile, truncate on desktop

### 4. States: loading, error, empty, success
- **Loading**: every async data fetch should show a visual indicator before content arrives
- **Error**: API failures should show an inline message, not silently fail or crash
- **Empty**: empty states should explain why and suggest an action
- **Success**: save/submit operations need visible feedback (toast, status change, animation)
- **Disabled**: buttons should be disabled when action is unavailable, with visual indication

### 5. Input validation
- What happens with: empty input, whitespace-only, duplicates (case-insensitive), very long strings, special characters?
- Are validation errors inline near the input, not in alerts or modals?
- Is the submit button disabled when input is invalid?
- For list inputs: does adding a duplicate warn the user?

### 6. Accessibility
- Do icon-only buttons have `aria-label`?
- Are form inputs associated with labels? (visible or `aria-label`)
- Is color not the only way to convey meaning? (pair with icons, text, or size)
- Can you tab through interactive elements?
- Do collapsible sections maintain focus correctly?

### 7. Visual consistency
- Do new elements match existing typography, spacing, and color patterns?
- Are borders, shadows, and corner radii consistent?
- Is the color palette used correctly? (green=success, amber=warning, red=error/destructive)
- Does the vertical rhythm feel right? (consistent spacing between sections)

## Anti-patterns to flag
- Using browser `confirm()` or `alert()` — use inline confirmation UI instead
- Inconsistent spacing on the same page
- Desktop-only designs that break on mobile
- Dense lists without collapse/expand (80+ items should default collapsed)
- Missing loading spinners on async data
- Introducing new button styles when a design system button exists
- Fixed-width layouts that don't adapt to screen size
- Color as the only differentiator (accessibility violation)

## Lessons learned from past projects

### Mobile RWD is always retrofitted
In every project, mobile responsiveness is added in dedicated fix-up passes, never built first. Budget for this — or better, build mobile-first.

### Layout normalization prevents downstream fixes
A single CSS variable for max content width on day one prevents dozens of layout fixes later. Same for spacing scales and font size hierarchies.

### "Polish" is continuous, not a phase
~27% of commits in mature projects are polish, not features. Expect this. Don't schedule a "polish sprint" — polish every commit.

### Sticky/fixed elements need 2-3 rounds
Width calculations, scroll behavior, viewport-relative positioning, and z-index conflicts always require multiple iterations. Plan for it.

### Component extraction follows pain
Components get extracted after they're duplicated 3+ times. Better: extract on the second use.

## Output format
- **Must fix** — broken or confusing interactions, accessibility violations
- **Should fix** — pattern inconsistencies, missing states, mobile breakage
- **Consider** — micro-interactions, animation, polish

## Step 2: Save observations (MANDATORY — do this after every review)

You MUST complete this step before finishing. Save 3-5 project-specific observations to a memory file called `review-design.md` in the project's auto-memory directory. Write or update the file using this format:

```markdown
---
name: Design review observations
description: UI patterns, interaction conventions, and accessibility notes from design reviews
type: project
---

- [observation 1]
- [observation 2]
```

Observations should capture: design system usage, interaction patterns, mobile responsiveness state, accessibility gaps, and visual consistency patterns. If a memory file already exists, update it — merge new observations, remove stale ones, don't duplicate.

If any observation seems universal (applies beyond this project), note it at the end of your review and suggest running `/review-learn` to promote it.

## Step 3: Learn from feedback (AUTOMATIC)

If the user corrects, disagrees with, or refines any finding during this conversation, evaluate the correction:

1. **Is it universal?** Does this correction apply beyond this specific project? (e.g., "mobile-first prevents RWD retrofit passes" is universal; "our pill toggles use p-[3px]" is project-specific)
2. **If universal**: Formulate a concise lesson (rule + why + when it applies) and append it to the "Lessons learned" section of this file (`~/.claude/commands/review-design.md`). Verify it doesn't duplicate an existing lesson.
3. **If project-specific**: Update the project's `review-design.md` memory file instead.

Do this automatically whenever feedback is received — do not wait for the user to run `/review-learn`.
