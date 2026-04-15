# Traceability Matrix — Auth Service

**Version:** v1.0  
**Last updated:** 2025

> Requirements source: [SSD](../architecture/SSD.md)  
> Test approach: [Test Strategy](test-strategy.md)

---

## Requirements ↔ Test Cases Mapping

| Requirement | Description | Test Case | Type | Status |
|-------------|-------------|-----------|------|--------|
| [FR-01](../architecture/SSD.md#fr-01--user-registration) | Register new user (success) | TC-01 | Integration | To be tested |
| [FR-01](../architecture/SSD.md#fr-01--user-registration) | Reject duplicate email | TC-02 | Integration | To be tested |
| [FR-02](../architecture/SSD.md#fr-02--authentication-login) | Successful authentication | TC-03 | Integration | To be tested |
| [FR-02](../architecture/SSD.md#fr-02--authentication-login) | Reject wrong password | TC-04 | Unit | To be tested |
| [FR-03](../architecture/SSD.md#fr-03--token-refresh) | Successful token refresh | TC-05 | Integration | To be tested |
| [FR-03](../architecture/SSD.md#fr-03--token-refresh) | Reject expired refresh token | TC-06 | Integration | To be tested |
| [FR-04](../architecture/SSD.md#fr-04--logout) | Successful logout | TC-07 | Integration | To be tested |
| [FR-04](../architecture/SSD.md#fr-04--logout) | Token invalidated after logout | TC-08 | Integration | To be tested |
| [FR-05](../architecture/SSD.md#fr-05--token-verification) | Verify valid token | TC-09 | Integration | To be tested |
| [NFR-01](../architecture/SSD.md#4-non-functional-requirements) | Performance: 1000 rpm, p95 < 200 ms | TC-10 | Performance | To be tested |
| [NFR-02](../architecture/SSD.md#4-non-functional-requirements) | Rate limiting: block after 5 attempts | TC-11 | Security | To be tested |

---

## Test Case Descriptions

### TC-01 — Register new user

**Requirement:** FR-01  
**Type:** Integration  
**Steps:**
1. Send `POST /api/auth/register` with valid email and password.
2. Verify response status is `201 Created`.
3. Verify user record exists in PostgreSQL.

**Expected result:** User is created, confirmation returned.

---

### TC-02 — Reject duplicate email

**Requirement:** FR-01  
**Type:** Integration  
**Steps:**
1. Register a user with email `test@example.com`.
2. Send second `POST /api/auth/register` with the same email.
3. Verify response status is `409 Conflict`.

**Expected result:** Error response with message about duplicate email.

---

### TC-03 — Successful authentication

**Requirement:** FR-02  
**Type:** Integration  
**Steps:**
1. Send `POST /api/auth/login` with valid credentials.
2. Verify response status is `200 OK`.
3. Verify `access_token` is present in response body.
4. Verify `refresh_token` is set as HttpOnly cookie.

**Expected result:** JWT pair returned, refresh token stored in Redis.

---

### TC-04 — Reject wrong password

**Requirement:** FR-02  
**Type:** Unit  
**Steps:**
1. Call `AuthService.login()` with correct email and wrong password.
2. Verify `UnauthorizedException` is thrown.

**Expected result:** 401 Unauthorized, no tokens issued.

---

### TC-05 — Successful token refresh

**Requirement:** FR-03  
**Type:** Integration  
**Steps:**
1. Login to obtain refresh token.
2. Send `POST /api/auth/refresh` with refresh token cookie.
3. Verify response status is `200 OK`.
4. Verify new `access_token` is returned.
5. Verify old refresh token is invalidated in Redis.

**Expected result:** New token pair issued, old refresh token deleted (Refresh Token Rotation).

---

### TC-06 — Reject expired refresh token

**Requirement:** FR-03  
**Type:** Integration  
**Steps:**
1. Use an expired or non-existent refresh token.
2. Send `POST /api/auth/refresh`.
3. Verify response status is `401 Unauthorized`.

**Expected result:** Error response, no new tokens issued.

---

### TC-07 — Successful logout

**Requirement:** FR-04  
**Type:** Integration  
**Steps:**
1. Login to obtain refresh token.
2. Send `POST /api/auth/logout` with valid access token.
3. Verify response status is `204 No Content`.

**Expected result:** Session ended, no response body.

---

### TC-08 — Token invalidated after logout

**Requirement:** FR-04  
**Type:** Integration  
**Steps:**
1. Login and save refresh token.
2. Logout.
3. Attempt `POST /api/auth/refresh` with the saved token.
4. Verify response status is `401 Unauthorized`.

**Expected result:** Refresh token no longer works after logout.

---

### TC-09 — Verify valid token

**Requirement:** FR-05  
**Type:** Integration  
**Steps:**
1. Login to obtain access token.
2. Send `GET /api/auth/verify` with `Authorization: Bearer <token>` header.
3. Verify response status is `200 OK`.

**Expected result:** Token is valid, user identity confirmed.

---

### TC-10 — Performance under load

**Requirement:** NFR-01  
**Type:** Performance  
**Tool:** k6  
**Steps:**
1. Run k6 load test: 50 virtual users, 1 minute duration.
2. Target endpoint: `POST /api/auth/login`.
3. Measure p95 response time and requests per minute.

**Expected result:** ≥ 1000 rpm, p95 < 200 ms.

---

### TC-11 — Rate limiting enforcement

**Requirement:** NFR-02  
**Type:** Security  
**Steps:**
1. Send 5 `POST /api/auth/login` requests with wrong password from the same IP.
2. Send 6th request.
3. Verify 6th request returns `429 Too Many Requests`.

**Expected result:** After 5 failed attempts, IP is temporarily blocked.

---

## Coverage Summary

| Requirement Group | Total Requirements | Test Cases | Coverage |
|------------------|--------------------|------------|----------|
| Functional (FR-01 to FR-05) | 5 | TC-01 to TC-09 (9 cases) | 100% |
| Non-Functional (NFR-01, NFR-02) | 2 | TC-10, TC-11 | 100% |
| **Total** | **7** | **11** | **100%** |
