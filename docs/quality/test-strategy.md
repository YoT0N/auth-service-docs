# Test Strategy — Auth Service

**Version:** v1.0  
**Status:** Approved  
**Last updated:** 2025

> References: [SSD — Functional requirements](../architecture/SSD.md) | [Traceability Matrix](traceability-matrix.md)

---

## 1. Testing Objectives

Ensure correct operation of all authentication functions, security of data transmission, and compliance with non-functional requirements defined in [SSD](../architecture/SSD.md):

- All functional requirements (FR-01 to FR-05) are covered by tests.
- Non-functional requirements (NFR-01, NFR-02) are verified by performance and security tests.
- Test coverage ≥ 80% for unit tests.

---

## 2. Testing Levels

### Unit Testing

**Goal:** Verify isolated logic components.  
**Scope:** TokenModule, password hashing, input validation, business logic functions.  
**Coverage target:** ≥ 80%  
**Tool:** Jest

```bash
npm run test:unit
npm run test:coverage
```

### Integration Testing

**Goal:** Verify component interaction.  
**Scope:** API → Service → Repository → Database chain, Redis integration.  
**Tool:** Jest + Supertest + test database (PostgreSQL in Docker)

```bash
npm run test:integration
```

### End-to-End (E2E) Testing

**Goal:** Verify complete user scenarios via HTTP.  
**Scenarios:**
- Register → Login → Access protected resource
- Login → Refresh → Logout
- Brute-force attempt → Rate limit trigger

**Tool:** Supertest / Postman Collections

### Security Testing

**Goal:** Verify protection against common attack vectors.  
**Checks:**
- Brute-force protection (rate limiting after 5 attempts)
- SQL injection resistance
- JWT manipulation (tampering, expired tokens, wrong signature)
- XSS protection (HttpOnly cookie validation)

**Tool:** OWASP ZAP, manual testing

### Performance Testing

**Goal:** Verify compliance with [NFR-01](../architecture/SSD.md#4-non-functional-requirements).  
**Target:** 1000 requests/min, p95 response time < 200 ms.  
**Tool:** k6

```javascript
// k6 load test scenario
export const options = {
  vus: 50,
  duration: '1m',
  thresholds: {
    http_req_duration: ['p(95)<200'],
    http_reqs: ['rate>16.7'], // 1000 rpm = 16.7 rps
  },
};
```

---

## 3. Tools Summary

| Tool | Purpose |
|------|---------|
| Jest | Unit and integration tests |
| Supertest | HTTP API testing |
| k6 | Load/performance testing |
| OWASP ZAP | Security scanning |
| GitHub Actions | CI — automatic test execution on push |

---

## 4. Test Environment

- **Unit/Integration:** Local or CI (Docker Compose with PostgreSQL + Redis)
- **E2E:** Staging environment
- **Performance:** Isolated staging environment (to avoid affecting production metrics)
- **Security:** Staging environment (OWASP ZAP automated scan)

---

## 5. Definition of Done

A feature is considered tested when:

1. Unit tests pass with ≥ 80% coverage.
2. Integration tests pass for happy path and error cases.
3. Corresponding test cases are linked in [Traceability Matrix](traceability-matrix.md).
4. No Critical or High severity security issues found.

---

*This document describes testing approach. Specific test cases and requirement mapping are in the [Traceability Matrix](traceability-matrix.md).*
