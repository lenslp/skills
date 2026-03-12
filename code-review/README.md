# Code Review

Comprehensive code review skill: analyze git diff, batch review large changesets (500+ lines), scan code quality, security vulnerabilities, and best practices compliance. Issues classified by P0-P3 priority with interactive fix workflow. Supports both frontend and backend codebases.

## Installation

```bash
npx skills add lenslp/skills --path code-review
```

## What It Checks

| Category | Checks |
|----------|--------|
| **Code Quality** | Correctness, readability, type safety, API design |
| **Security** | Injection, auth/authz, data exposure, input validation, dependency CVEs, frontend security |
| **Best Practices** | Architecture, error handling, testing, performance |
| **Frontend** | Component design, rendering performance, state management, a11y, bundle size, UX robustness |

## Priority Levels

| Priority | Definition | Action |
|----------|-----------|--------|
| **P0** | Security vulnerabilities, data loss, crash | Must fix immediately |
| **P1** | Logic errors, missing error handling, broken contracts | Fix before merge |
| **P2** | Code quality, readability, test coverage gaps | Should fix |
| **P3** | Style nits, naming, minor optimizations | Nice to have |

## Usage

```
Use $code-review to review my recent changes.
```

Review 完成后可以选择修复方式：

- 按优先级：`fix P0`、`fix P0 and P1`
- 按模块：`fix src/auth/`
- 按编号：`fix #1, #3, #5`
- 全部修复：`fix all`

## Files

```
code-review/
├── SKILL.md                          # Main workflow
├── agents/openai.yaml                # UI metadata
└── references/
    ├── security-checklist.md         # Detailed security checks by category & language
    ├── best-practices.md             # Backend + frontend best practices
    └── frontend-checklist.md         # Frontend-specific checks (50+ items)
```
