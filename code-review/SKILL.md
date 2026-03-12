---
name: code-review
description: Perform comprehensive code review on git changes including diff analysis, batch review for large changesets over 500 lines, code quality scanning, security vulnerability scanning, and best practices compliance checking. Use when the user asks for a code review, PR review, diff review, or wants to check code quality, security, or best practices on recent git changes. Triggers on requests like "review my code", "check this PR", "review my changes", "scan for security issues", or "check code quality".
---

# Code Review

Comprehensive code review workflow that analyzes git diffs, assesses code quality, scans for security vulnerabilities, and verifies best practices compliance.

## Workflow

```
git diff → Scope → (>500 lines? batch) → SOLID → Quality → Security → Dead code → Best practices → Frontend → Report → Verdict → Confirm
```

**Important**: This is a review-only workflow by default. Do NOT implement any code changes until the user explicitly confirms.

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

- **No changes detected**: Inform user. Ask if they want to review staged changes (`--cached`) or a specific commit range.
- **Total changed lines (additions + deletions) > 500**: Batch by module. Group files by top-level directory, review each batch separately, then summarize cross-module concerns (breaking interfaces, circular dependencies).
- **<= 500 lines**: Single-pass review of all changes.
- **Mixed concerns**: Group findings by logical feature area, not just file order.

## Step 3: SOLID & Architecture Scan

Check all five SOLID principles and common code smells:

- **SRP**: Overloaded modules with unrelated responsibilities. Ask: "What is the single reason this module would change?"
- **OCP**: Frequent edits to switch/if blocks to add behavior instead of extension points. Ask: "Can I add a variant without touching existing code?"
- **LSP**: Subclasses that break expectations or require type checks. Ask: "Can I substitute any subclass without the caller knowing?"
- **ISP**: Wide interfaces with unused methods. Ask: "Do all implementers use all methods?"
- **DIP**: High-level logic tied to concrete I/O or infrastructure. Ask: "Can I swap the implementation without changing business logic?"

Also flag code smells: feature envy, data clumps, primitive obsession, shotgun surgery, speculative generality.

When proposing a refactor, explain *why* it improves cohesion/coupling. For non-trivial refactors, propose an incremental plan instead of a big rewrite.

For detailed prompts and heuristics, see [references/solid-checklist.md](references/solid-checklist.md).

## Step 4: Code Quality Scan

### Correctness
- Logic errors, off-by-one, missing null/undefined checks
- Unhandled edge cases and error paths
- Race conditions in async code (concurrent access, check-then-act, TOCTOU, missing locks)
- Resource leaks (unclosed connections, file handles, listeners)

### Boundary Conditions
- Null/undefined access without checks
- Empty array/object not handled (e.g., `arr[0]` without length check)
- Division by zero, integer overflow, floating point comparison
- Truthy/falsy confusion (`if (value)` when `0` or `""` are valid)
- Off-by-one in loops, slicing, pagination

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

## Step 5: Security Scan

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

## Step 6: Dead Code & Removal Candidates

Identify code that is unused, redundant, or feature-flagged off:
- Unreachable code paths, never-called functions, unused exports
- Deprecated APIs still present, old feature flag branches
- Commented-out code blocks with no context

Classify each as:
- **Safe delete now**: No references found, no external consumers
- **Defer with plan**: Has active consumers or needs migration

For non-trivial removals, use the template in [references/removal-plan.md](references/removal-plan.md).

## Step 7: Best Practices Check

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

## Step 7b: Frontend-Specific Scan (if changeset includes frontend code)

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
- **Verdict**: ✅ APPROVE / ⚠️ REQUEST_CHANGES / 💬 COMMENT
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

## Removal Candidates (if any)
- **Safe delete**: [list items]
- **Defer with plan**: [list items with migration notes]

## Module Reports (if batched)
### Module: src/auth/
...

## Cross-Cutting Concerns
- Interface compatibility
- Shared dependency changes
- Migration considerations
```

### Verdict Rules
- **APPROVE**: No P0 or P1 issues. P2/P3 only.
- **REQUEST_CHANGES**: Any P0 or P1 issues exist.
- **COMMENT**: No issues found, or only informational observations.

### Clean Review (no issues found)

If no issues are found, explicitly state:
- What was checked (list the scan steps completed)
- Areas not covered (e.g., "Did not verify database migrations" or "No test files in diff")
- Residual risks or recommended follow-up tests

## Step 8: Interactive Fix

**Important**: Do NOT implement any changes until the user explicitly confirms. Present the report first and wait.

After presenting the report, ask the user how to proceed:

```
Verdict: [APPROVE / REQUEST_CHANGES / COMMENT]
Found N issues (P0: X, P1: Y, P2: Z, P3: W).

How would you like to proceed?
1. Fix by priority: e.g. "fix P0" or "fix P0 and P1"
2. Fix by module: e.g. "fix auth module" or "fix src/api/"
3. Fix by issue number: e.g. "fix #1, #3, #5"
4. "fix all" to apply all fixes
5. "skip" to end without changes
```

When applying fixes:
- Process in priority order (P0 first, then P1, etc.) regardless of how user selected them.
- Show each fix before applying. Run linter/tests after each batch if the project has them configured.
- After completing the requested fixes, re-run diff to confirm changes and present remaining issues if any.

## Notes

- **Language matching**: Respond in the same language as the user's query. If the user asks in Chinese, output the entire review report in Chinese. If the user asks in English, output in English. This applies to all sections: summary, issue descriptions, suggested fixes, and interactive prompts.
- Always read actual diff content (`git diff`) before judging, not just file names.
- If the project has a linter config (ESLint, Prettier, Ruff, etc.), run it and include findings.
- Prioritize P0/P1 issues over P2/P3 in both reporting and fixing.
- If the project has existing `.editorconfig`, `tsconfig.json`, or similar configs, respect their conventions.
