# Security Audit -- Inter-Component Communications & Authentication

**Audit Date:** 2026-03-19
**Auditor:** SEC (Security Auditor)
**Scope:** All inter-component communication channels and authentication mechanisms
**Codebase Revision:** main @ 3bb2c3b

---

## Executive Summary

**Overall Risk Level: HIGH**

The codebase implements a solid HMAC-SHA256 token scheme with nonce-based replay protection and secret rotation, which is well above average for internal tooling. However, there are **4 critical**, **5 high**, **3 medium**, and **3 low** severity findings. The most impactful issues are: (1) all HTTP channels run over plaintext with no TLS support, (2) the distributed dispatcher never attaches its configured auth secret to outbound requests, (3) WebSocket connections are completely unauthenticated, and (4) password hashing uses a single round of SHA-256 instead of a key derivation function.

---

## 1. Communication Channels Inventory

| # | Source | Destination | Protocol | Auth Mechanism | Encrypted | Notes |
|---|--------|-------------|----------|----------------|-----------|-------|
| 1 | CLI (clientshell) | Toolbox HTTP API | HTTP POST | HMAC-SHA256 Bearer token | **NO** | Plaintext HTTP, auto-detects localhost:9090 |
| 2 | CLI (clientshell) | Toolbox subprocess | exec pipe | CS_AUTH_TOKEN env var | N/A (local) | Token verified on startup by cs-toolbox |
| 3 | CLI (clientshell) | cs-server (web) | HTTP | Session cookie / Bearer token | **NO** | apiclient.Client, no TLS code |
| 4 | cs-server (web) | Browser clients | HTTP + WS | Session cookie (HTTP), **NONE** (WS) | **NO** | WebSocket is fully unauthenticated |
| 5 | Dispatcher | Remote agents | HTTP POST | **NONE** (auth secret stored but unused) | **NO** | Critical gap |
| 6 | Audit trail | Local filesystem | File I/O | flock advisory locking | N/A | SHA-256 hash chain for integrity |

---

## 2. Authentication Mechanisms

### 2.1 HMAC-SHA256 Token Auth (Toolbox API)

**Files:** `internal/toolbox/auth.go`, `internal/toolbox/server.go:199-221`, `internal/cli/toolbox_client.go:264-275`

**Strengths:**
- HMAC-SHA256 using `crypto/hmac` with constant-time comparison (`hmac.Equal`) -- prevents timing attacks (`auth.go:182`).
- 16-byte cryptographic nonce per token via `crypto/rand` (`auth.go:82-88`).
- Replay protection: nonces are tracked in `sync.Map` and rejected on reuse (`auth.go:214`).
- Token expiry enforced, with 5-second clock skew tolerance (`auth.go:204`).
- `maxAge` check (default 60s) prevents old tokens from being reused even before expiry (`auth.go:209`).
- Future-dated token detection (`auth.go:204`).
- Empty secret rejection (`auth.go:94`).

**Weaknesses:**
- **Nonce store is in-memory only.** On server restart, all nonces are lost, allowing replay of unexpired tokens issued before the restart. The `CleanupNonces()` method (`auth.go:222`) must be called externally; there is no automatic cleanup goroutine, leading to unbounded memory growth under sustained load. (See SEC-005)
- Token expiry is set to 5 minutes on the client side (`toolbox_client.go:269`) but `maxAge` on the verifier is 60 seconds (`auth.go:69`). This means tokens older than 60 seconds are rejected even though they have not "expired" per their claims. This is actually a defense-in-depth feature that limits the replay window, but the mismatch could cause legitimate requests to fail if there is clock drift or processing delay.

### 2.2 cs-server Authentication (Web API)

**Files:** `internal/web/middleware.go:173-210`, `internal/web/handlers.go:80-152`

**Strengths:**
- Session tokens are 32-byte cryptographic random (`middleware.go:78-79`).
- CSRF tokens are generated per session (`middleware.go:83-84`).
- Session cookies use `HttpOnly` and `SameSiteStrict` (`handlers.go:124-131`).
- RBAC-based permission checks via `RequirePermission` middleware (`middleware.go:213-224`).
- Security headers applied: X-Content-Type-Options, X-Frame-Options, CSP, Referrer-Policy (`middleware.go:267-276`).

**Weaknesses:**
- **Password hashing uses single-round SHA-256** (`middleware.go:244-248`). This is catastrophically weak against offline brute-force attacks. Should use bcrypt, scrypt, or Argon2. (See SEC-001)
- **Session cookie missing `Secure` flag** (`handlers.go:124-131`). Cookie will be transmitted over plaintext HTTP. (See SEC-003)
- **CSRF token is generated but never validated** in any handler. No middleware checks `X-CSRF-Token` header against the stored session CSRF token. (See SEC-006)
- CORS is set to `Access-Control-Allow-Origin: *` (`middleware.go:253`), allowing any origin to make credentialed requests. (See SEC-008)

### 2.3 Agent-to-Agent Authentication (Distributed Dispatcher)

**Files:** `internal/distributed/dispatcher.go:73-85, 255-311`

**Critical Finding:** The dispatcher accepts an `authSecret` via `WithAuthSecret()` option (`dispatcher.go:83-87`) and stores it, but **never uses it**. The `execOnAgent` method (`dispatcher.go:255-311`) sets only `Content-Type` on outbound requests -- no `Authorization` header, no token generation, no authentication whatsoever. Remote agents receive completely unauthenticated requests. (See SEC-002)

Similarly, `CheckHealth` (`dispatcher.go:178-206`) sends unauthenticated GET requests to agents.

---

## 3. Encryption Assessment

### 3.1 Shell <-> Toolbox

**Finding:** No TLS. The toolbox server uses `http.ListenAndServe()` (`server.go:186-187`), never `ListenAndServeTLS()`. The auto-detect probe hardcodes `http://localhost:9090` (`toolbox_client.go:175`). All traffic including auth tokens and command outputs is transmitted in cleartext.

**Risk:** On localhost, the risk is limited to local privilege escalation (any process on the host can sniff loopback traffic). On a network deployment, HMAC tokens and command outputs are fully exposed.

### 3.2 Shell <-> cs-server

**Finding:** No TLS. The web server uses `ListenAndServe()` (`web/server.go:221`). Session tokens, passwords (in login POST bodies), and all API data are transmitted in cleartext.

**Risk:** Session hijacking, credential theft on any network path between client and server.

### 3.3 Agent <-> Agent (Distributed)

**Finding:** No TLS. Agent URLs are stored as plain strings (`distributed/dispatcher.go:38`). No certificate validation or pinning.

**Risk:** Full MITM capability. Combined with the absent authentication (SEC-002), any attacker on the network can impersonate agents and execute arbitrary commands.

### 3.4 Audit Trail Storage

**Finding:** Audit log files are stored with mode `0640` (`audit/store.go:52`), directory with `0750` (`audit/store.go:27`). The SHA-256 hash chain (`audit/store.go:220-238`) provides tamper evidence but not confidentiality. File locking via `flock` provides process-level safety.

**Risk:** Any user in the file's group can read audit logs. No encryption at rest.

---

## 4. Vulnerability Assessment

### SEC-001 -- Weak Password Hashing
- **Severity:** CRITICAL
- **Component:** `internal/web/middleware.go:244-248`
- **Description:** `hashPassword()` uses a single SHA-256 iteration: `sha256(salt + password)`. Modern GPUs can compute billions of SHA-256 hashes per second. An attacker who obtains the in-memory user store can brute-force passwords trivially.
- **Recommendation:** Replace with `golang.org/x/crypto/bcrypt` -- but since stdlib-only is a project rule, implement PBKDF2 using `crypto/sha256` and `crypto/hmac` with at least 600,000 iterations per OWASP 2023 guidance, or use the stdlib `crypto/subtle` package with a custom Argon2-like construction.

### SEC-002 -- Dispatcher Auth Secret Never Used
- **Severity:** CRITICAL
- **Component:** `internal/distributed/dispatcher.go:255-311`
- **Description:** `authSecret` is accepted and stored but never attached to outbound HTTP requests. All agent-to-agent communication is unauthenticated. Any network host can send POST /exec to agents and execute arbitrary commands.
- **Recommendation:** Generate HMAC tokens using the `authSecret` and attach them as `Authorization: Bearer <token>` headers in `execOnAgent()` and `CheckHealth()`. The `toolbox.GenerateToken()` function already exists and should be reused.

### SEC-003 -- No TLS on Any Communication Channel
- **Severity:** CRITICAL
- **Component:** `internal/toolbox/server.go:186`, `internal/web/server.go:221`
- **Description:** All three server components (toolbox API, web server, agent endpoints) use plaintext HTTP. Auth tokens, session cookies, passwords, and command outputs are transmitted unencrypted.
- **Recommendation:** Add TLS support via `http.ListenAndServeTLS()` with configurable cert/key paths. For localhost-only toolbox communication, consider Unix domain sockets as an alternative. At minimum, add the `Secure` flag to session cookies.

### SEC-004 -- WebSocket Endpoint Has No Authentication
- **Severity:** CRITICAL
- **Component:** `internal/web/websocket.go:148-231`, `internal/web/server.go:203`
- **Description:** The `/ws` endpoint is registered without any authentication middleware (`server.go:203` -- no `wrapAuth`). WebSocket connections accept `user`, `role`, and `session` as unauthenticated query parameters (`websocket.go:199-204`). An attacker can join any session room, impersonate any user, and broadcast arbitrary events to all participants.
- **Recommendation:** Wrap the WebSocket handler with `wrapAuth()` and validate the session before upgrade. Extract user/role from the authenticated session context rather than from query parameters.

### SEC-005 -- Nonce Store Unbounded Memory Growth
- **Severity:** HIGH
- **Component:** `internal/toolbox/auth.go:61, 214, 222-232`
- **Description:** Used nonces are stored in `sync.Map` indefinitely. `CleanupNonces()` exists but is never called automatically. Under sustained load, memory usage grows without bound. After server restart, all nonces are lost, enabling replay of any unexpired token.
- **Recommendation:** Start a periodic goroutine that calls `CleanupNonces()` (e.g., every 30 seconds). Consider a fixed-size LRU cache or time-bucketed cleanup. For persistence across restarts, either shorten the `maxAge` to minimize the replay window or persist nonces to disk.

### SEC-006 -- CSRF Token Generated But Never Validated
- **Severity:** HIGH
- **Component:** `internal/web/middleware.go:83-84`, `internal/web/handlers.go`
- **Description:** A CSRF token is generated for every session (`CSRFToken` field in `SessionData`) but no middleware or handler ever checks the `X-CSRF-Token` header against it. State-changing POST requests (login, logout, session operations, exec) are vulnerable to CSRF attacks.
- **Recommendation:** Add CSRF validation middleware for all state-changing endpoints. Verify that the request's `X-CSRF-Token` header matches the session's stored CSRF token.

### SEC-007 -- Secret Rotator Creates Fresh Verifiers (Nonce Bypass)
- **Severity:** HIGH
- **Component:** `internal/toolbox/secret_rotation.go:59-84`
- **Description:** `SecretRotator.Verify()` creates a new `HMACVerifier` on every call (`secret_rotation.go:71, 79`). Each new verifier has an empty nonce store, so the replay protection is completely bypassed when using the rotator. A token can be replayed indefinitely as long as it has not expired.
- **Recommendation:** Maintain persistent `HMACVerifier` instances for the current and previous secrets, and rotate them (not recreate them) when the secret changes. The nonce stores must persist across calls.

### SEC-008 -- CORS Allows All Origins
- **Severity:** HIGH
- **Component:** `internal/web/middleware.go:253`
- **Description:** `Access-Control-Allow-Origin: *` allows any website to make API requests to the server. Combined with the session cookie authentication, this enables cross-origin credential theft if the `SameSite` policy is relaxed or the browser has specific configurations.
- **Recommendation:** Restrict CORS to known origins. Use a configurable allowlist. At minimum, do not reflect `*` when credentials are involved.

### SEC-009 -- Toolbox API Request Body Not Size-Limited
- **Severity:** HIGH
- **Component:** `internal/toolbox/server.go:284`
- **Description:** `json.NewDecoder(r.Body).Decode(&req)` reads the full request body without any size limit. An attacker can send a multi-gigabyte POST body to exhaust server memory (denial of service).
- **Recommendation:** Wrap `r.Body` with `http.MaxBytesReader(w, r.Body, maxSize)` before decoding. A 1MB limit is reasonable for command requests.

### SEC-010 -- Dispatcher Response Body Not Size-Limited
- **Severity:** MEDIUM
- **Component:** `internal/distributed/dispatcher.go:279`
- **Description:** `io.ReadAll(resp.Body)` reads unbounded response data from remote agents. A malicious or compromised agent can return an arbitrarily large response, exhausting dispatcher memory.
- **Recommendation:** Use `io.LimitReader(resp.Body, maxSize)` before reading.

### SEC-011 -- WebSocket Client ID Is Predictable
- **Severity:** MEDIUM
- **Component:** `internal/web/websocket.go:447-448`
- **Description:** `generateClientID()` uses `time.Now().Format(...)` which is predictable and not unique under concurrent connections. Two clients connecting in the same millisecond get the same ID.
- **Recommendation:** Use `crypto/rand` to generate client IDs (the `generateSecureToken` function already exists in the same package).

### SEC-012 -- Sensitive Data in Error Responses
- **Severity:** MEDIUM
- **Component:** `internal/toolbox/server.go:215-216, 321-323`
- **Description:** Authentication error messages include the specific failure reason (e.g., `authentication failed: nonce already used (replay attack)`). This leaks information about the auth mechanism to attackers. Tool execution errors (`err.Error()`) are returned directly and may contain filesystem paths or internal state.
- **Recommendation:** Return generic "authentication failed" for all auth errors. Sanitize tool execution errors to avoid leaking internal paths.

### SEC-013 -- Secret Passed via CLI Flag
- **Severity:** LOW
- **Component:** `cmd/cs-toolbox/main.go:1698-1703`
- **Description:** The `--secret` flag accepts the HMAC secret as a CLI argument, making it visible in `ps` output and shell history. The env var `CS_AUTH_SECRET` alternative exists but is not enforced.
- **Recommendation:** Deprecate the `--secret` flag. Accept secrets only via environment variable or file reference (`--secret-file`).

### SEC-014 -- WebSocket Frame Size Limit
- **Severity:** LOW
- **Component:** `internal/web/websocket.go:426`
- **Description:** WebSocket frame size limit is 1MB. While this prevents extreme abuse, it is generous for a control-plane WebSocket. Consider whether 64KB would suffice.
- **Recommendation:** Reduce the max frame size to the minimum required for the application's events.

### SEC-015 -- Session Cookie Not Domain-Scoped
- **Severity:** LOW
- **Component:** `internal/web/handlers.go:124-131`
- **Description:** The session cookie has no `Domain` attribute. In multi-subdomain deployments, the cookie may be sent to unintended subdomains.
- **Recommendation:** Set an explicit `Domain` attribute on the session cookie matching the deployment's domain.

---

## 5. Hardening Recommendations

### 5.1 Critical (implement immediately)

1. **SEC-002:** Wire the `authSecret` into `execOnAgent()` and `CheckHealth()` in the dispatcher. Generate HMAC bearer tokens for all inter-agent requests.
2. **SEC-004:** Add authentication to the WebSocket endpoint. Validate session before upgrade; derive user/role from authenticated session context.
3. **SEC-001:** Replace single-round SHA-256 password hashing with PBKDF2-HMAC-SHA256 (600k+ iterations) using only stdlib packages.
4. **SEC-003:** Add TLS configuration options to all HTTP servers. Use Unix domain sockets for localhost-only toolbox communication.

### 5.2 High (implement in next sprint)

5. **SEC-007:** Fix `SecretRotator.Verify()` to reuse persistent `HMACVerifier` instances instead of creating new ones per call.
6. **SEC-005:** Add automatic nonce cleanup goroutine in the toolbox server startup.
7. **SEC-006:** Implement CSRF validation middleware for all state-changing endpoints.
8. **SEC-008:** Replace wildcard CORS with a configurable origin allowlist.
9. **SEC-009:** Add `http.MaxBytesReader` to the toolbox API `/exec` handler.

### 5.3 Medium (planned improvement)

10. **SEC-010:** Add `io.LimitReader` to dispatcher response reading.
11. **SEC-011:** Use `crypto/rand` for WebSocket client ID generation.
12. **SEC-012:** Sanitize error messages in authentication and tool execution responses.

### 5.4 Low (nice to have)

13. **SEC-013:** Deprecate `--secret` CLI flag in favor of env var or file-based secret input.
14. **SEC-014:** Reduce WebSocket max frame size.
15. **SEC-015:** Set explicit `Domain` on session cookies.

---

## 6. Compliance Check -- OWASP Top 10 (2021) Mapping

| OWASP ID | Category | Status | Findings |
|----------|----------|--------|----------|
| A01:2021 | Broken Access Control | **FAIL** | SEC-004 (unauthenticated WebSocket), SEC-002 (unauthenticated agent dispatch), SEC-006 (no CSRF validation) |
| A02:2021 | Cryptographic Failures | **FAIL** | SEC-001 (weak password hashing), SEC-003 (no TLS), SEC-007 (nonce bypass via rotator) |
| A03:2021 | Injection | PASS | JSON decoding is used throughout; no SQL or command injection vectors identified. Tool args are passed as string slices, not shell-interpolated. |
| A04:2021 | Insecure Design | **WARN** | SEC-002 (auth secret accepted but unused is a design gap), SEC-006 (CSRF tokens generated but not checked) |
| A05:2021 | Security Misconfiguration | **FAIL** | SEC-008 (CORS *), SEC-013 (secret in CLI args), SEC-015 (cookie scoping) |
| A06:2021 | Vulnerable and Outdated Components | PASS | No external dependencies (stdlib only). Go stdlib is maintained. |
| A07:2021 | Identification and Authentication Failures | **FAIL** | SEC-001 (weak password hash), SEC-004 (no WS auth), SEC-002 (no agent auth) |
| A08:2021 | Software and Data Integrity Failures | PASS | Audit trail uses SHA-256 hash chain with integrity verification. HMAC token signing prevents tampering. |
| A09:2021 | Security Logging and Monitoring Failures | PASS | Comprehensive audit logging with immutable hash chain. All API requests are logged with user, action, and timestamp. |
| A10:2021 | Server-Side Request Forgery (SSRF) | **WARN** | The dispatcher sends HTTP requests to agent URLs stored in the registry. If the registry is writable without proper authorization, an attacker could register agents pointing to internal services. The registry file is written with mode 0600, mitigating this. |

---

**Summary:** The authentication framework (HMAC-SHA256 with nonces) is architecturally sound, but implementation gaps -- particularly the dispatcher not using its configured secret, unauthenticated WebSocket, and the absence of TLS -- create exploitable attack surfaces. The four critical findings should be addressed before any network-facing deployment.
