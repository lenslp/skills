# Security Scan Checklist

Detailed security checklist by vulnerability category and language.

## Table of Contents
- [Injection](#injection)
- [Authentication & Authorization](#authentication--authorization)
- [Data Exposure](#data-exposure)
- [Input Validation](#input-validation)
- [Dependency & Configuration](#dependency--configuration)
- [Language-Specific Checks](#language-specific-checks)

## Injection

| Check | Severity | What to Look For |
|-------|----------|------------------|
| SQL Injection | 🔴 Critical | String concatenation in SQL. Must use parameterized queries. |
| XSS | 🔴 Critical | User input in HTML without escaping. Check `innerHTML`, `dangerouslySetInnerHTML`. |
| Command Injection | 🔴 Critical | User input in `exec`, `spawn`, `system` without sanitization. |
| NoSQL Injection | 🟡 High | User objects passed to MongoDB `$where`, `$regex`. |
| Template Injection | 🟡 High | User input through Jinja2, EJS, Handlebars without sandboxing. |

## Authentication & Authorization

| Check | Severity | What to Look For |
|-------|----------|------------------|
| Missing Auth | 🔴 Critical | New endpoints without authentication middleware. |
| Broken Access Control | 🔴 Critical | IDOR — resources accessible by changing URL IDs without ownership check. |
| Hardcoded Credentials | 🔴 Critical | Passwords, API keys, tokens in source code. |
| Weak Password Handling | 🟡 High | Plaintext storage, MD5/SHA1 without salt. Use bcrypt/argon2. |
| Session Management | 🟡 High | No expiration, predictable IDs, no invalidation on logout. |

## Data Exposure

| Check | Severity | What to Look For |
|-------|----------|------------------|
| Secrets in Logs | 🔴 Critical | Logging passwords, tokens, PII. |
| Verbose Errors | 🟡 High | Stack traces or internal paths exposed in production. |
| Missing Encryption | 🟡 High | Sensitive data over HTTP or stored unencrypted. |
| Over-fetching | 🟡 Medium | API returning password hashes, internal IDs. |
| CORS Misconfiguration | 🟡 High | `Access-Control-Allow-Origin: *` on authenticated endpoints. |

## Input Validation

| Check | Severity | What to Look For |
|-------|----------|------------------|
| Path Traversal | 🔴 Critical | User input in file paths (`../../../etc/passwd`). |
| Missing Validation | 🟡 Medium | No schema validation on API request bodies. |
| File Upload | 🟡 High | No type/size validation, executable uploads. |
| Regex DoS | 🟡 Medium | Complex regex on user input causing backtracking. |

## Dependency & Configuration

| Check | Severity | What to Look For |
|-------|----------|------------------|
| Vulnerable Deps | 🟡 High | Run `npm audit`, `pip audit`, `cargo audit`. |
| Insecure Defaults | 🟡 Medium | Debug mode in production, default passwords. |
| Missing Headers | 🟡 Medium | No CSP, HSTS, X-Frame-Options. |
| Missing Rate Limiting | 🟡 Medium | No rate limiting on login, registration, API. |
| CSRF | 🟡 Medium | State-changing requests without CSRF tokens. |

## Frontend-Specific Security

| Check | Severity | What to Look For |
|-------|----------|------------------|
| localStorage Secrets | 🔴 Critical | JWT tokens, passwords, or sensitive data in localStorage/sessionStorage. Use httpOnly cookies. |
| dangerouslySetInnerHTML | 🔴 Critical | Rendering user HTML without DOMPurify or equivalent sanitization. |
| Third-Party Scripts | 🟡 High | External scripts without `integrity` (SRI) hash or from untrusted CDNs. |
| postMessage Validation | 🟡 High | Receiving `window.postMessage` without checking `event.origin`. |
| Open Redirect | 🟡 High | Redirecting via user-supplied URL parameter without allowlist. |
| Sensitive Data in URL | 🟡 High | Tokens, PII in query parameters (leaks via browser history, referrer headers). |
| iframe Sandboxing | 🟡 Medium | Third-party iframes without `sandbox` attribute. |
| CSP Compliance | 🟡 Medium | Inline scripts/styles that break Content Security Policy. Use nonces or hashes. |
| Client-Side Auth Logic | 🟡 High | Authorization decisions made only on the client without server-side enforcement. |

## Language-Specific Checks

### JavaScript / TypeScript
- `eval()`, `Function()`, `setTimeout(string)` with user input
- `__proto__` pollution via `Object.assign` or spread on user objects
- Dynamic `require()` with user-controlled paths
- Unvalidated `postMessage` origins

### Python
- `pickle.loads()` on untrusted data
- `yaml.load()` without `Loader=SafeLoader`
- `os.system()` / `subprocess.run(shell=True)` with user input
- `exec()` / `eval()` with user strings

### Go
- Unchecked error returns from I/O
- `fmt.Sprintf` in SQL instead of parameterized queries
- Missing `defer` for resource cleanup

### Java
- XXE in XML parsers without disabling external entities
- Insecure deserialization (`ObjectInputStream` on untrusted data)
- Missing `@PreAuthorize` / `@Secured` on controllers
