# Frontend Review Checklist

Detailed frontend-specific checks for component design, rendering performance, state management, bundle size, accessibility, styling, and UX robustness.

## Table of Contents
- [Component Design](#component-design)
- [Rendering Performance](#rendering-performance)
- [State Management](#state-management)
- [Bundle Size & Loading](#bundle-size--loading)
- [Accessibility (a11y)](#accessibility-a11y)
- [CSS & Styling](#css--styling)
- [Frontend Security](#frontend-security)
- [UX Robustness](#ux-robustness)

## Component Design

| Check | Priority | What to Look For |
|-------|----------|------------------|
| Single Responsibility | P2 | Component doing too many things (fetching + rendering + business logic). Split into container/presentational or use hooks. |
| Props Design | P2 | More than 7 props, boolean prop explosion, passing entire objects when only 1-2 fields used. |
| Controlled vs Uncontrolled | P1 | Mixing controlled and uncontrolled patterns in the same component. Pick one. |
| Composition over Inheritance | P2 | Using inheritance or deep prop drilling instead of `children`, render props, or slots. |
| Key Usage | P1 | Using array index as `key` for dynamic lists that reorder/insert/delete. Use stable unique IDs. |
| Hardcoded Strings | P3 | UI text hardcoded instead of using i18n functions. |
| Component Size | P2 | Component file exceeds 300 lines. Extract sub-components or custom hooks. |

## Rendering Performance

| Check | Priority | What to Look For |
|-------|----------|------------------|
| Unnecessary Re-renders | P1 | New object/array/function created every render in props. Wrap with `useMemo`/`useCallback` or extract as constant. |
| Missing Memoization | P2 | Expensive computation in render path without `useMemo`. |
| React.memo Misuse | P3 | Applying `React.memo` everywhere vs. only on components that receive stable props but have expensive render. |
| Large List Rendering | P1 | Rendering 100+ items without virtualization. Use `react-window`, `react-virtuoso`, or equivalent. |
| Effect Dependencies | P1 | Missing or incorrect `useEffect` dependency array causing infinite loops or stale closures. |
| Event Handler Allocation | P2 | Creating arrow functions inside `map()` for event handlers on large lists. |
| Layout Thrashing | P1 | Reading DOM layout properties (offsetHeight) then immediately writing styles in a loop. |
| Image Optimization | P2 | Large unoptimized images, missing `width`/`height`, no lazy loading, no `srcset` for responsive. |

## State Management

| Check | Priority | What to Look For |
|-------|----------|------------------|
| State Proximity | P2 | State lifted too high or stored globally when only used by one subtree. Keep state close to where it's consumed. |
| Prop Drilling | P2 | Passing props through 3+ intermediate components. Use context, composition, or state library. |
| Global State Bloat | P2 | Storing server-cache data (API responses) in Redux/Zustand instead of using React Query/SWR/TanStack Query. |
| Derived State | P1 | Storing computed values as state and syncing with `useEffect`, instead of computing during render. |
| State Batching | P2 | Multiple sequential `setState` calls that could be combined (pre-React 18 issue or in async callbacks). |

## Bundle Size & Loading

| Check | Priority | What to Look For |
|-------|----------|------------------|
| Dynamic Import | P2 | Route-level components not using `React.lazy()` / dynamic `import()`. |
| Heavy Dependencies | P2 | Importing large libraries (moment.js, lodash full) when lighter alternatives exist (date-fns, lodash-es individual imports). |
| Tree Shaking | P2 | Barrel file re-exports (`export * from`) preventing tree shaking. Import directly from module path. |
| Unused Dependencies | P3 | Packages in `dependencies` that are no longer imported anywhere. |
| Asset Size | P2 | Uncompressed SVGs, PNGs where WebP/AVIF would suffice, fonts with unused character sets. |
| Vendor Chunk | P3 | Not splitting vendor chunk from app chunk, causing full re-download on every deploy. |

## Accessibility (a11y)

| Check | Priority | What to Look For |
|-------|----------|------------------|
| Semantic HTML | P1 | Using `<div>` / `<span>` for buttons, links, headings, lists. Use native elements. |
| ARIA Attributes | P2 | Custom interactive widgets without `role`, `aria-label`, `aria-expanded`, `aria-describedby`. |
| Keyboard Navigation | P1 | Interactive elements not reachable or operable via keyboard (Tab, Enter, Escape). |
| Color Contrast | P2 | Text-to-background contrast ratio below WCAG AA (4.5:1 normal text, 3:1 large text). |
| Focus Management | P2 | Modal/dialog opened without moving focus into it; focus not trapped inside; focus not restored on close. |
| Alt Text | P1 | Images missing `alt` attribute or using meaningless alt like "image". |
| Form Labels | P1 | Form inputs without associated `<label>` or `aria-label`. |
| Heading Hierarchy | P3 | Skipping heading levels (h1 → h3) or multiple h1 on a page. |

## CSS & Styling

| Check | Priority | What to Look For |
|-------|----------|------------------|
| Style Isolation | P2 | Global CSS selectors leaking across components. Use CSS Modules, Scoped styles, or CSS-in-JS. |
| !important Abuse | P3 | Excessive use of `!important` to override specificity. Fix the cascade instead. |
| Responsive Design | P2 | Fixed pixel widths on containers, no media queries / container queries for mobile. |
| Z-index Chaos | P3 | Arbitrary large z-index values (9999) without a defined scale/system. |
| CSS Unused | P3 | Large stylesheets with selectors that match nothing in the codebase. |
| Animation Performance | P2 | Animating `width`, `height`, `top`, `left` (triggers layout). Use `transform` and `opacity`. |
| Dark Mode | P3 | Hardcoded colors instead of using CSS variables / design tokens for theme support. |

## Frontend Security

| Check | Priority | What to Look For |
|-------|----------|------------------|
| localStorage Secrets | P0 | Storing JWT tokens, passwords, or sensitive data in localStorage/sessionStorage. Use httpOnly cookies. |
| dangerouslySetInnerHTML | P0 | Rendering user-supplied HTML without sanitization (DOMPurify or equivalent). |
| Third-Party Scripts | P1 | Loading external scripts without `integrity` (SRI) or from untrusted CDNs. |
| iframe Sandboxing | P2 | Embedding third-party content in iframes without `sandbox` attribute. |
| postMessage Validation | P1 | Receiving `window.postMessage` without checking `event.origin`. |
| Open Redirect | P1 | Redirecting based on user-supplied URL parameter without allowlist validation. |
| CSP Compliance | P2 | Inline scripts/styles that would break Content Security Policy. Use nonces or hashes. |
| Sensitive Data in URL | P1 | Tokens, passwords, or PII in URL query parameters (visible in browser history, logs, referrer). |

## UX Robustness

| Check | Priority | What to Look For |
|-------|----------|------------------|
| Error Boundaries | P1 | No React Error Boundary wrapping major sections. A single component crash takes down the whole page. |
| Loading States | P1 | Data-fetching components showing blank screen instead of skeleton/spinner. |
| Empty States | P2 | Lists/tables showing nothing when data is empty, instead of a helpful empty state message. |
| Error States | P1 | API errors silently swallowed or showing raw error objects to users. |
| Optimistic Updates | P2 | Optimistic UI not rolling back on server failure, leaving UI in inconsistent state. |
| Offline Handling | P3 | No indication when network is lost; forms silently fail to submit. |
| Form Validation | P2 | Only server-side validation, no client-side feedback. Or client-side only without server validation. |
| Debounce/Throttle | P2 | Search inputs or resize handlers firing on every keystroke/frame without debouncing. |
