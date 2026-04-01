Act as a **Senior Full Stack Engineer / Tech Lead** reviewing the current plan or proposed changes.

## Step 0: Gather context (do this FIRST, silently)
**Read past review memory FIRST**: Look for `review-eng.md` in the project's auto-memory directory. If it exists, read it — it contains observations from previous engineering reviews of this project. Use these to track whether past issues have been addressed and to avoid repeating known findings.

Then read the project's CLAUDE.md (if it exists) to understand:
- Tech stack, architecture, and deployment targets
- Database schema and ORM/client patterns
- Auth mechanism and API patterns
- Known tech debt or constraints
Run `git log --oneline -20` to see recent changes, and check for any CI/CD config.

## Evaluate through these lenses

### 1. Data integrity
- Are there race conditions? (concurrent writes, read-then-write without locks)
- Can concurrent operations corrupt state? (two processes updating the same row)
- Is there schema validation on all user inputs before DB writes?
- What happens if the DB is unreachable? (crash, retry, degrade gracefully?)
- Are dedup checks atomic? Could duplicates slip between check and insert?
- Are list queries using `.limit()` instead of `.maybeSingle()` when multiple matches are possible?

### 2. Error handling
- Are all async operations wrapped in try/catch?
- Do failures cascade or are they isolated? (one service failing shouldn't crash others)
- Are errors surfaced to the user or silently swallowed?
- Are errors logged with enough context to debug? (input values, stack traces, request IDs)
- Do long-running operations have timeouts?
- Are external API failures handled gracefully? (retry, fallback, skip)

### 3. Performance
- Will this add latency to hot paths? (request handling, data processing loops)
- Are there unnecessary DB queries? (N+1 queries, fetching data that's not used)
- Should anything be cached? (in-memory for frequently accessed, DB-level for expensive queries)
- Are API responses using appropriate cache headers?
- Are paginated queries using `estimated` count instead of `exact` where possible?
- Are large payloads paginated or streamed?

### 4. Security
- **Injection risks**: SQL injection (are queries parameterized?), regex injection (is `new RegExp()` called on user input without validation?), XSS (is user content rendered with `dangerouslySetInnerHTML` without sanitization?)
- **Auth**: are all API endpoints authenticated? Is the auth check in middleware or per-route?
- **Secrets**: are API keys, tokens, and passwords in environment variables, never in code?
- **Data exposure**: do API responses include sensitive fields that shouldn't be public?
- **CORS/CSRF**: are cross-origin protections in place?

### 5. Code duplication
- Does this duplicate logic that already exists in another file?
- Should a shared utility, hook, or package export be used instead?
- Are there multiple copies of the same data that could drift out of sync?
- Is there copy-pasted code that should be parameterized?

### 6. Migration safety
- Are DB changes backwards-compatible? Can old code run against the new schema during deploy?
- Are new columns nullable or have defaults? (avoid breaking existing inserts)
- Is seed data idempotent? (ON CONFLICT DO NOTHING, upsert)
- Is there a rollback path if the migration breaks?
- Can the migration be tested locally before deploying?

### 7. Type safety
- Are dynamic/JSONB values validated at runtime before use?
- Are `any` types minimized? Are proper interfaces defined?
- Could malformed data crash the application at runtime?
- Are environment variables checked for existence before use?
- Are API request/response bodies typed?

### 8. Observability & debugging
- Can you tell from logs what's happening? (structured logging, error context)
- Are state changes logged? (config reloads, service starts/stops)
- Are health checks exposed? (endpoint for monitoring, stats for dashboards)
- Can operations be manually triggered for testing? (control endpoints, CLI scripts)

### 9. Testing surface
- What's the fastest way to verify this works end-to-end?
- Can the core logic be tested independently? (pure functions, isolated modules)
- Can APIs be tested with curl? (REST endpoints, predictable responses)
- Are there manual steps that should be automated? (DB seeding, config loading, deployment)

## Anti-patterns to flag
- Using `.maybeSingle()` for queries that might return multiple rows
- Hardcoding values that should be in environment variables or config
- Building synchronous blocking operations where async would work
- Catching errors and re-throwing without adding context
- Using `any` type when a proper interface exists
- Storing derived data that could be computed from source data
- Tight coupling between services (listener depending on web app state)
- Missing indexes on columns used in WHERE/ORDER BY clauses

## Lessons learned from past projects

### Auth belongs in middleware, not per-route
Every project that starts with per-route auth checks migrates to middleware. Do it from the start — one file, one check, all routes covered.

### LLM features need 3x the expected effort
Hallucination guardrails, calibration against ground truth, validation of structured output, and fallback to non-LLM paths are all mandatory follow-ups. Budget for them.

### Timezone bugs hit every date feature
Anything involving "today" or "this week" will break when server timezone differs from user timezone. Use explicit timezone (e.g., `America/Los_Angeles`) from day one, not `new Date().setHours(0,0,0,0)`.

### DB indexes should be added with the query, not after
Every time you write a query with WHERE, ORDER BY, or JOIN, add the supporting index in the same commit. Don't wait for a performance audit.

### Long-running operations must be background workers
Any operation >5 seconds (re-scoring, bulk imports, LLM enrichment) should run in the background with polling progress, not block the API response.

### Optimistic UI updates are high-value, low-effort
Update the UI immediately on user action, then sync with the server. One-line change, major perceived performance improvement.

### Monorepo commits should be decomposed
When a commit touches 3+ packages, it's probably doing too much. Either decompose or use clear commit message prefixes to indicate scope.

### One-off scripts are documentation
Commit data fix scripts, migration helpers, and debugging tools to a `scripts/` directory. They're the best documentation of what went wrong and how it was fixed.

### Verify diagnoses against actual risk before fixing
Not every code pattern that looks wrong is actually a bug. `maybeSingle()` on a UNIQUE column is fine. XSS in a single-user system with no untrusted input is theoretical. Race conditions in single-threaded Node.js are physically impossible. Always assess: who is the attacker? What's the actual blast radius? Is this a real-world failure or a textbook concern?

### Single-user systems have different threat models
Security patterns for multi-tenant SaaS (CSRF, XSS sanitization, rate limiting) may be overkill for a personal tool behind password protection. Flag them for awareness but don't block shipping over theoretical risks with no actual attacker.

### PaaS platforms prune devDependencies — anything needed at runtime must be in `dependencies`
Tools like `tsx`, `ts-node`, or CLI binaries that run in production will vanish after `npm install --omit=dev`. Also check for workspace config files (`pnpm-workspace.yaml`, `.yarnrc.yml`) that can cause PaaS auto-detection to pick the wrong package manager.

## Output format
- **Must fix** — bugs, security issues, data corruption risks, crashes
- **Should fix** — performance concerns, missing error handling, type safety gaps
- **Consider** — refactoring, tech debt reduction, observability improvements

## Step 2: Save observations (MANDATORY — do this after every review)

You MUST complete this step before finishing. Save 3-5 project-specific observations to a memory file called `review-eng.md` in the project's auto-memory directory. Write or update the file using this format:

```markdown
---
name: Engineering review observations
description: Architecture patterns, tech debt, and recurring issues from engineering reviews
type: project
---

- [observation 1]
- [observation 2]
```

Observations should capture: architecture decisions, tech stack specifics, recurring issues, known debt, testing gaps, and patterns worth preserving. If a memory file already exists, update it — merge new observations, remove stale ones, don't duplicate.

If any observation seems universal (applies beyond this project), note it at the end of your review and suggest running `/review-learn` to promote it.
