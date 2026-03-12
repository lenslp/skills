[English](README.md) | [中文](README.zh-CN.md)

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

## Usage in Cursor

The skill is automatically registered after installation. Use it in Cursor Chat (Agent mode).

### Auto Trigger

The skill activates when your prompt contains keywords like:

- "review my code"
- "check this PR"
- "scan for security issues"
- "check code quality"

### Manual Trigger

Example prompts:

```
Review my recent code changes.
```

```
Review the changes on this branch against main.
```

```
Check my staged code for security issues.
```

### After Review

The report lists all issues sorted by P0-P3 priority. Choose how to fix:

- By priority: `fix P0`, `fix P0 and P1`
- By module: `fix src/auth/`
- By issue number: `fix #1, #3, #5`
- Fix everything: `fix all`
- Skip: `skip`

## Files

```
code-review/
├── SKILL.md                          # Main workflow
├── agents/agent.yaml                # UI metadata
└── references/
    ├── security-checklist.md         # Detailed security checks by category & language
    ├── best-practices.md             # Backend + frontend best practices
    └── frontend-checklist.md         # Frontend-specific checks (50+ items)
```
