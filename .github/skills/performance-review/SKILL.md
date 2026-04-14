---
name: performance-review
description: Performance review checklist for tester-performance, covering both frontend (React) and backend (NestJS) concerns.
---

# Skill: Performance Review

## Purpose

Performance review checklist for `tester-performance`, covering both frontend (React) and backend (NestJS) concerns.

---

## Frontend Performance

### Rendering

- [ ] No unnecessary re-renders — components only re-render when their data changes
- [ ] Expensive computations wrapped in `useMemo`
- [ ] Stable callback references with `useCallback` on props passed to memoised children
- [ ] Lists use stable, unique `key` props — not array index
- [ ] No anonymous objects/arrays created inline in JSX (breaks memoisation)

### Bundle Size

- [ ] New dependencies are justified — check their size on `bundlephobia.com`
- [ ] Tree-shakeable imports used: `import { specific } from 'lib'` not `import lib from 'lib'`
- [ ] Route-level code splitting in place for large pages
- [ ] No duplicate dependencies (two versions of the same library)

### Asset Loading

- [ ] Images use appropriate formats (WebP/AVIF preferred)
- [ ] Images have explicit `width` and `height` to prevent layout shifts
- [ ] Images below the fold are lazy-loaded
- [ ] Fonts use `font-display: swap`

### Core Web Vitals

- [ ] **LCP** (Largest Contentful Paint) — critical images/content loads quickly; no blocking requests
- [ ] **CLS** (Cumulative Layout Shift) — no elements popping in and causing layout movement
- [ ] **INP** (Interaction to Next Paint) — UI responds quickly to user input

**Measurement tools:**

| Tool                        | How to use                                                                      |
| --------------------------- | ------------------------------------------------------------------------------- |
| **Lighthouse CI**           | Run `lhci autorun` in CI to enforce score thresholds per build                  |
| **`playwright-lighthouse`** | Attach Lighthouse to Playwright E2E runs for CWV measurement against real pages |
| **Chrome DevTools MCP**     | Capture reproducible traces, network data, console output, and memory evidence  |
| **WebPageTest**             | Real-device testing with waterfall views                                        |

### Network

- [ ] API calls are not made more than necessary (no polling when not needed)
- [ ] Paginated endpoints used for lists — not loading all records
- [ ] Loading states shown immediately so the UI doesn't feel frozen

## Chrome DevTools MCP Workflow

For browser-facing work, capture runtime evidence against a running target when the task calls for it:

- Performance trace for the primary user flow or screen
- Network waterfall or request summary for the critical API calls
- Console message snapshot for warnings and errors
- Lighthouse or Core Web Vitals snapshot for page-level changes
- Memory evidence when investigating suspected leaks or runaway listeners

Every performance finding should cite the artifact that supports it.

---

## Backend Performance

### Database

- [ ] New queries use indexes — check with `EXPLAIN ANALYZE` on Postgres
- [ ] No N+1 queries — use `JOIN` or `eager loading` appropriately
- [ ] Paginated queries for list endpoints — no `SELECT *` without a `LIMIT`
- [ ] No unnecessary columns selected (`SELECT *` replaced with explicit column list on large tables)
- [ ] Transactions used only where needed — they hold locks

### Application

- [ ] No synchronous blocking operations on the event loop (no `fs.readFileSync`, `execSync`)
- [ ] CPU-intensive work offloaded to a worker or queue
- [ ] No memory leaks — event listeners are cleaned up; subscriptions are closed

### Caching

- [ ] Expensive, frequently-read data is cached (in-memory, Redis)
- [ ] Cache invalidation strategy is correct — stale data won't be served after updates
- [ ] Repeated identical requests within a single request cycle are deduplicated

### Payload Size

- [ ] API responses return only the data the client needs — no over-fetching
- [ ] Large file uploads/downloads streamed — not buffered in memory
- [ ] Response compression enabled (`compression` middleware)

---

## Severity Levels

| Level      | Description                                            |
| ---------- | ------------------------------------------------------ |
| Blocking   | Causes visible latency, timeouts, or OOM in normal use |
| Warning    | Will cause issues at scale or with real data volumes   |
| Suggestion | Optimisation opportunity, not urgent                   |

---

## Finding Format

```
## Performance Review: {Task Title}

### Evidence Collected

### Frontend Findings
### Backend Findings

### Findings Table
| # | Severity | Area | Issue | Recommendation |
|---|---|---|---|---|

### Verdict: PASS / FAIL
```
