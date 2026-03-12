# Best Practices Reference

Detailed engineering best practices for code review.

## Table of Contents
- [Clean Code](#clean-code)
- [Architecture](#architecture)
- [Testing Standards](#testing-standards)
- [Performance](#performance)
- [Git Hygiene](#git-hygiene)

## Clean Code

### Functions
- Under 50 lines. Prefer <= 3 parameters; use options object for more.
- Either compute a value OR perform a side effect, not both.
- Verbs for functions (`getUserById`), nouns for variables (`activeUsers`), booleans as adjectives (`isValid`).

### Error Handling
- Never use empty `catch` blocks.
- Validate inputs at boundaries, fail fast.
- Use typed/custom error classes with context (what failed, with what input).

### Code Organization
- Group imports: stdlib → external → internal.
- Extract magic numbers into named constants.
- Prefer guard clauses over deep `if-else`.
- DRY threshold: extract only after 3+ duplications.

## Architecture

### Separation of Concerns
- **Controllers**: Parse input, format output. No business logic.
- **Services**: Business logic, framework-agnostic.
- **Repositories**: Data access only.
- **Utilities**: Pure functions, no application state.

### API Design
- PUT/DELETE must be idempotent.
- All list endpoints must paginate.
- Breaking changes require version bumps.
- Consistent error format across endpoints.

### Database
- Schema changes via migrations, never manual DDL.
- Index every `WHERE` column and foreign key.
- Multi-step mutations wrapped in transactions.
- Prevent N+1 with joins, eager loading, or DataLoader.

## Testing Standards

### Structure
- Arrange-Act-Assert pattern.
- One behavior per test case.
- Descriptive names: `should_return_404_when_user_not_found`.
- No logic (`if`, `for`) in test code.

### Coverage
- 80%+ branch coverage on new code.
- 100% on critical paths (auth, payments, data mutations).
- Always test: empty/null input, boundary values, concurrent access, network failures, malformed input.

### Quality
- No flaky tests. Remove timing dependencies.
- Mock only external boundaries (DB, network, filesystem).
- Use factories/fixtures, not hardcoded values.
- Tests clean up after themselves.

## Performance

### Anti-Patterns
- N+1 queries in loops.
- Loading entire tables into memory.
- Synchronous blocking in async contexts.
- `SELECT *` when only specific columns needed.
- Missing indexes on frequently-queried columns.

### Guidelines
- Measure before optimizing.
- Cache expensive computations and frequently-read data.
- Batch multiple small DB/API calls.
- Lazy load non-critical resources.
- Reuse connections via pooling.

## Frontend Practices

### Component Design
- Single Responsibility: one component = one concern. Split container (data) from presentational (UI).
- Prefer composition (`children`, render props, slots) over deep prop drilling.
- Limit props to 7 or fewer. Use context or composition to reduce.
- Compute derived values during render, not via `useEffect` + `setState`.
- Use stable unique IDs as `key`, not array index, for dynamic lists.

### Rendering Performance
- Wrap expensive computations in `useMemo`, stable callbacks in `useCallback`.
- Virtualize lists with 100+ items (`react-window`, `react-virtuoso`).
- Audit `useEffect` deps: missing deps = stale closures, extra deps = infinite loops.
- Animate only `transform` and `opacity`. Avoid animating layout properties (`width`, `top`).
- Lazy load images (`loading="lazy"`) and provide `width`/`height` to prevent layout shift.

### State Management
- Keep state as close to its consumers as possible.
- Use React Query / SWR / TanStack Query for server state, not Redux/Zustand.
- Minimize global state to truly app-wide concerns (auth, theme, locale).

### Accessibility
- Use native interactive elements (`<button>`, `<a>`, `<input>`) over styled `<div>`.
- Every `<img>` needs meaningful `alt`; every form input needs a `<label>`.
- Ensure keyboard navigation: Tab ordering, Enter/Space activation, Escape to close modals.
- Trap focus inside modals/dialogs; restore focus on close.
- Maintain WCAG AA color contrast (4.5:1 normal text, 3:1 large text).

### Bundle Optimization
- Lazy-load route-level components (`React.lazy`, dynamic `import()`).
- Import individual modules, not barrel files (`import debounce from 'lodash/debounce'`).
- Prefer lightweight alternatives: `date-fns` over `moment`, `clsx` over `classnames`.
- Audit bundle with `source-map-explorer` or `webpack-bundle-analyzer` periodically.

### UX Robustness
- Wrap major page sections in Error Boundaries.
- Every async data fetch must handle: loading, success, empty, and error states.
- Debounce search inputs and resize/scroll handlers (150-300ms).
- Validate forms on both client and server. Show inline field-level errors.

## Git Hygiene

### Commits
- Atomic: one logical change per commit, passes tests.
- Format: `type(scope): description` (e.g., `fix(auth): handle expired token`).
- No mixing refactoring with features.
- No generated files or binaries.

### PRs
- Under 500 lines when possible; split larger work into stacked PRs.
- Explain WHY, not just WHAT. Link issues.
- Self-review before requesting others.
- Include before/after screenshots for UI changes.
