---
name: code-review
description: Perform comprehensive code review on git changes including diff analysis, batch review for large changesets over 500 lines, code quality scanning, security vulnerability scanning, and best practices compliance checking. Use when the user asks for a code review, PR review, diff review, or wants to check code quality, security, or best practices on recent git changes. Triggers on requests like "review my code", "check this PR", "review my changes", "scan for security issues", or "check code quality".
---

# Code Review

Comprehensive code review workflow that analyzes git diffs, assesses code quality, scans for security vulnerabilities, and verifies best practices compliance.

## Workflow

```
git diff --stat → Assess scope → (>500 lines? batch by module) → Quality scan → Security scan → Best practices check → Report
```

## Step 1: Analyze Diff Scope

Run these commands to understand the changeset:

```bash
git diff --stat
git diff --shortstat
git diff --name-only
```

For staged changes use `git diff --cached`. For branch comparison use `git diff main...HEAD`.

Present a summary table:

```
| File                  | +Added | -Removed | Net |
|-----------------------|--------|----------|-----|
| src/auth/login.ts     | +45    | -12      | +33 |
| **Total**             | +320   | -180     | +140|
```

## Step 2: Determine Review Strategy

- **Total changed lines (additions + deletions) > 500**: Batch by module. Group files by top-level directory, review each batch separately, then summarize cross-module concerns (breaking interfaces, circular dependencies).
- **<= 500 lines**: Single-pass review of all changes.

## Step 3: Code Quality Scan

### Correctness
- Logic errors, off-by-one, missing null/undefined checks
- Unhandled edge cases and error paths
- Race conditions in async code
- Resource leaks (unclosed connections, file handles, listeners)

### Readability
- Functions > 50 lines or > 5 parameters
- Nesting > 3 levels deep
- Duplicated logic across files
- Vague names (`data`, `temp`, `result`, `handle`)
- Dead code, commented-out blocks, unused imports

### Type Safety & API
- Overly broad types (`any`, `object`, `{}`)
- Inconsistent error handling patterns
- Breaking public API changes without version bumps

## Step 4: Security Scan

### Critical (block merge)
- Hardcoded secrets, API keys, tokens, passwords
- SQL injection (string concatenation in queries)
- XSS / `dangerouslySetInnerHTML` without sanitization
- Command injection (unsanitized input in shell commands)
- Path traversal (user input in file paths)
- Sensitive data in localStorage/sessionStorage (tokens, passwords)

### High Priority
- Missing auth/authz on new endpoints
- Overly permissive CORS
- Sensitive data in logs, error messages, or URL query parameters
- Missing rate limiting on public endpoints
- Third-party scripts without SRI, unvalidated `postMessage` origins
- Open redirect via user-supplied URL

### Medium Priority
- Missing input validation
- Dependencies with known CVEs
- Missing CSRF protection
- Insecure defaults
- iframes without `sandbox`, inline scripts breaking CSP

For detailed language-specific and frontend checks, see [references/security-checklist.md](references/security-checklist.md).

## Step 5: Best Practices Check

### Architecture
- Single Responsibility: each function/class/component does one thing
- Proper separation of concerns (business logic vs I/O vs presentation)
- Dependency injection where appropriate

### Error Handling
- Errors caught at appropriate boundaries, never swallowed silently
- User-facing errors informative but don't leak internals
- Retry logic with backoff for transient failures

### Testing
- New code has corresponding tests
- Tests cover happy path AND edge cases
- No flaky assertions (timing-dependent, order-dependent)

### Performance (Backend)
- No N+1 queries
- Large collections paginated or streamed
- No synchronous blocking in async contexts

### Performance (Frontend)
- No unnecessary re-renders (objects/arrays/functions recreated every render)
- Large lists (100+ items) use virtualization
- Correct `useEffect` dependency arrays (no infinite loops or stale closures)
- Heavy images optimized (lazy loading, proper format, `srcset`)
- Animations use `transform`/`opacity` instead of layout-triggering properties

For detailed backend guidelines, see [references/best-practices.md](references/best-practices.md).

## Step 5b: Frontend-Specific Scan (if changeset includes frontend code)

Detect frontend code by file extensions (`.jsx`, `.tsx`, `.vue`, `.svelte`, `.css`, `.scss`, `.html`) or directory patterns (`src/components/`, `src/pages/`, `app/`). If frontend files are present, additionally check:

### Component Design
- Component exceeds 300 lines — extract sub-components or hooks
- Props explosion (7+ props) — consider composition or context
- Array index used as `key` in dynamic lists
- Derived state stored with `useEffect` sync instead of computing in render

### State Management
- State lifted too high or stored globally when only one subtree uses it
- Prop drilling through 3+ intermediate components
- Server data cached in Redux/Zustand instead of React Query/SWR

### Accessibility (a11y)
- `<div>`/`<span>` used for interactive elements instead of native `<button>`/`<a>`
- Missing `alt` on images, missing `<label>` on form inputs
- Interactive elements not keyboard-navigable
- Color contrast below WCAG AA (4.5:1 normal, 3:1 large text)

### Bundle Size
- Route components not lazy-loaded (`React.lazy` / dynamic `import()`)
- Heavy deps imported fully (moment.js, lodash) when tree-shakeable alternatives exist
- Barrel file re-exports (`export *`) preventing tree shaking

### UX Robustness
- Missing Error Boundary on major page sections
- No loading / empty / error states for async data
- Search/resize handlers without debounce/throttle

For the complete frontend checklist, see [references/frontend-checklist.md](references/frontend-checklist.md).

## Output Format

All issues must be classified into exactly one priority level:

| Priority | Definition | Action |
|----------|-----------|--------|
| **P0** | Security vulnerabilities, data loss, crash. Blocks merge. | Must fix immediately |
| **P1** | Logic errors, missing error handling, broken contracts. | Fix before merge |
| **P2** | Code quality, readability, test coverage gaps. | Should fix |
| **P3** | Style nits, naming, minor optimizations. | Nice to have |

Structure the review report as:

```markdown
# Code Review Report

## Summary
- **Files changed**: X
- **Lines changed**: +Y / -Z
- **Review mode**: Single pass / Batched by module
- **Overall risk**: 🔴 High / 🟡 Medium / 🟢 Low
- **Issue count**: P0: N / P1: N / P2: N / P3: N

## P0 — Critical (must fix immediately)
| # | File:Line | Module | Issue | Suggested Fix |
|---|-----------|--------|-------|---------------|
| 1 | src/auth/login.ts:42 | auth | Hardcoded JWT secret | Move to env variable |

## P1 — Important (fix before merge)
| # | File:Line | Module | Issue | Suggested Fix |
|---|-----------|--------|-------|---------------|
| 2 | src/api/users.ts:87 | api | Missing null check on user query | Add guard clause |

## P2 — Improvement (should fix)
| # | File:Line | Module | Issue | Suggested Fix |
|---|-----------|--------|-------|---------------|
| 3 | src/utils/format.ts:15 | utils | Function exceeds 80 lines | Extract helper functions |

## P3 — Minor (nice to have)
| # | File:Line | Module | Issue | Suggested Fix |
|---|-----------|--------|-------|---------------|
| 4 | src/api/users.ts:12 | api | Variable name `d` is vague | Rename to `userData` |

## Module Reports (if batched)
### Module: src/auth/
...

## Cross-Cutting Concerns
- Interface compatibility
- Shared dependency changes
- Migration considerations
```

## Step 6: Interactive Fix

After presenting the report, ask the user how to proceed. Provide these options:

1. **Fix by priority**: "Fix all P0", "Fix P0 and P1", etc.
2. **Fix by module**: "Fix all issues in src/auth/", etc.
3. **Fix specific issues**: "Fix #1, #3, #5" (by issue number from the table)
4. **Fix everything**: Apply all suggested fixes at once
5. **Skip**: End the review without making changes

Prompt format:

```
Review complete. Found N issues (P0: X, P1: Y, P2: Z, P3: W).

How would you like to proceed?
- By priority: e.g. "fix P0" or "fix P0 and P1"
- By module: e.g. "fix auth module" or "fix src/api/"
- By issue number: e.g. "fix #1, #3, #5"
- "fix all" to apply all fixes
- "skip" to end without changes
```

When applying fixes:
- Process in priority order (P0 first, then P1, etc.) regardless of how user selected them.
- Show each fix before applying. Run linter/tests after each batch if the project has them configured.
- After completing the requested fixes, re-run diff to confirm changes and present remaining issues if any.

## Notes

- Always read actual diff content (`git diff`) before judging, not just file names.
- If the project has a linter config (ESLint, Prettier, Ruff, etc.), run it and include findings.
- Prioritize P0/P1 issues over P2/P3 in both reporting and fixing.
- If the project has existing `.editorconfig`, `tsconfig.json`, or similar configs, respect their conventions.
