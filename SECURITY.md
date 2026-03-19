# Security Policy & Patch Log

## Reporting Vulnerabilities

Please report security vulnerabilities to **julien.rousselot@gmail.com**.
Do NOT open public issues for security vulnerabilities.

## Supported Versions

| Version | Supported |
|---------|-----------|
| v2.x    | Yes       |
| v1.x    | No        |

---

## Security Patch Log

### Sprint 16 — Security Hardening (2026-03-19)

Based on comprehensive pentest report (SEC-PENTEST-2026-03-19).

#### Critical Fixes

| ID | CVSSv3 | Vulnerability | Component | Fix |
|----|--------|---------------|-----------|-----|
| FINDING-001 | 9.1 | SSRF in web capture tool | `screenshot.go` | Block internal IPs (169.254/16, 10/8, 172.16/12, 192.168/16, 127/8), cloud metadata, file:// scheme |
| FINDING-002 | 8.8 | Path traversal in audit store | `store.go` | Sanitize client name with `filepath.Base()`, verify resolved path under baseDir |
| FINDING-003 | 8.6 | Unauthenticated WebSocket | `websocket.go` | Require token in query param or Authorization header, verify against active sessions |
| FINDING-004 | 8.2 | Dispatcher sends no auth to agents | `dispatcher.go` | Generate HMAC token per request, set Authorization: Bearer header |
| FINDING-005 | 7.5 | Single-round SHA-256 passwords | `middleware.go` | PBKDF2-like 100K iterations of HMAC-SHA256 with per-user salt |

#### High Fixes

| ID | CVSSv3 | Vulnerability | Component | Fix |
|----|--------|---------------|-----------|-----|
| FINDING-006 | 7.4 | Timing attack on API key | `gateway.go` | Use `crypto/subtle.ConstantTimeCompare()` |
| FINDING-007 | 7.0 | No body size limit on /exec | `server.go` | Add `http.MaxBytesReader` (10 MB max) |
| FINDING-008 | 6.5 | CORS Allow-Origin: * | `middleware.go` | Configurable origin whitelist, default deny |
| FINDING-009 | 6.0 | Nonce store unbounded growth | `auth.go` | Auto-cleanup expired nonces every 100 verifications |
| FINDING-010 | 6.5 | Arbitrary file write via --output | `screenshot.go` | Block `..`, system directories (/etc, /usr, /var, /sys, /proc) |
| FINDING-011 | 6.0 | All servers plaintext HTTP | `server.go`, `gateway.go` | Add `WithTLS(cert, key)` option for ListenAndServeTLS |
| FINDING-012 | 6.8 | Agent registration unauthenticated | `dispatcher.go` | Require shared secret for agent registration |

### Sprint 14 — E2EE Layer (2026-03-19)

| Feature | Component | Description |
|---------|-----------|-------------|
| Ed25519 identity | `crypto/identity.go` | Per-user/agent keypair with TOFU keystore |
| X25519 key exchange | `crypto/handshake.go` | Forward secrecy via ephemeral keys |
| AES-256-GCM encryption | `crypto/envelope.go` | End-to-end encrypted payloads |
| E2EE sessions | `crypto/session.go` | Automatic handshake + encrypted channel |

### Sprint 6 — Token Hardening (2026-03-18)

| Feature | Component | Description |
|---------|-----------|-------------|
| Nonce anti-replay | `toolbox/auth.go` | 16-byte crypto/rand nonce per token |
| IssuedAt + maxAge | `toolbox/auth.go` | 60s token lifetime enforcement |
| Rate limiting | `toolbox/ratelimit.go` | 100 req/min per user, 429 on excess |
| Secret rotation | `toolbox/secret_rotation.go` | Current + previous secret, auto-generate |

### Sprint 5 — Resilience (2026-03-18)

| Feature | Component | Description |
|---------|-----------|-------------|
| Circuit breaker | `cli/circuit_breaker.go` | 3 failures → 30s cooldown → half-open probe |
| Health endpoint | `toolbox/server.go` | Version, uptime, request counters |
| Auto-detection | `cli/toolbox_client.go` | Probe localhost:9090 with 500ms timeout |

### Sprint 1 — Foundation (2026-03-18)

| Feature | Component | Description |
|---------|-----------|-------------|
| HMAC-SHA256 auth | `toolbox/auth.go` | Token-based auth on all toolbox executions |
| Immutable audit trail | `audit/logger.go` | SHA-256 hash chain, tamper detection |
| Audit wiring | `cli/app.go` | All VM operations logged to audit trail |

---

## Audit Reports

| Date | Report | Findings |
|------|--------|----------|
| 2026-03-19 | [SEC-COMMS-AUDIT](docs/audit-reports/SEC-COMMS-AUDIT-2026-03-19.md) | 4 critical, 5 high (communications) |
| 2026-03-19 | [SEC-PENTEST](docs/audit-reports/SEC-PENTEST-2026-03-19.md) | 5 critical, 7 high (attack surface) |
| 2026-03-19 | [Automated Audit](docs/audit-reports/AUDIT-2026-03-19-888ca07.md) | 21/21 pass, 0 races, 0 deps |

## Security Architecture

```
Client ──── E2EE (X25519 + AES-256-GCM) ────── Agent
  │                                                │
  │ Ed25519 identity                               │ Ed25519 identity
  │ HMAC-SHA256 tokens                             │ Token verification
  │ Circuit breaker                                │ Rate limiting
  │ Nonce anti-replay                              │ Nonce store
  │                                                │
  └──────── SHA-256 Audit Trail (immutable) ───────┘
```
