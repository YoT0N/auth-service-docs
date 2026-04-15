# SSD — System Specification Document

**Auth Service**  
**Version:** v1.0  
**Status:** Approved  
**Last updated:** 2025

---

## 1. Purpose of the System

Auth Service is a microservice responsible for secure user authentication and authorization in a distributed system. It serves as the single entry point for all identity-related operations and provides a standardized mechanism for issuing and verifying access tokens.

The service is part of a microservices architecture and interacts with:
- **User Service** — user profile data
- **Notification Service** — email confirmations (delegated)
- **API Gateway** — routing and SSL termination

---

## 2. System Boundaries

### Included in scope

- REST API for external clients and internal microservices
- User data storage (PostgreSQL)
- Active session cache (Redis)
- JWT token generation and validation module

### Excluded from scope

- Access rights management (RBAC) — responsibility of Authorization Service
- Email notifications — responsibility of Notification Service

---

## 3. Functional Requirements

### FR-01 — User Registration

The system accepts user data (email, password), verifies email uniqueness, hashes the password using bcrypt (cost factor 12), stores the record in the database, and returns a registration confirmation.

**Endpoint:** `POST /api/auth/register`  
**Success:** `201 Created`  
**Errors:** `400 Bad Request`, `409 Conflict` (duplicate email)

### FR-02 — Authentication (Login)

The system verifies credentials, generates a token pair (access token — TTL 15 min, refresh token — TTL 7 days), stores the refresh token in Redis, and returns both tokens to the client.

**Endpoint:** `POST /api/auth/login`  
**Success:** `200 OK`  
**Errors:** `400 Bad Request`, `401 Unauthorized`

### FR-03 — Token Refresh

The system accepts a refresh token, verifies its validity in Redis, invalidates the old token, and generates a new token pair (Refresh Token Rotation).

**Endpoint:** `POST /api/auth/refresh`  
**Success:** `200 OK`  
**Errors:** `401 Unauthorized`

### FR-04 — Logout

The system deletes the refresh token from Redis, preventing further session renewal.

**Endpoint:** `POST /api/auth/logout`  
**Success:** `204 No Content`  
**Errors:** `401 Unauthorized`

### FR-05 — Token Verification

An internal endpoint for verifying the validity of an access token by other microservices via API Gateway.

**Endpoint:** `GET /api/auth/verify`  
**Success:** `200 OK`  
**Errors:** `401 Unauthorized`

---

## 4. Non-Functional Requirements

| Category | Requirement | Metric |
|----------|-------------|--------|
| Performance | Throughput | At least 1000 requests/min |
| Performance | Response time | No more than 200 ms (95th percentile) |
| Availability | Uptime | At least 99.9% (SLA) |
| Security | Password encryption | bcrypt, cost factor 12 |
| Security | Transport layer | HTTPS/TLS 1.3 mandatory |
| Security | Brute-force protection | Rate limiting: 5 attempts/min per IP |
| Scalability | Horizontal scaling | Stateless design (Redis for state) |

> **NFR-01:** Performance — 1000 rpm, p95 < 200 ms  
> **NFR-02:** Rate limiting — block after 5 failed attempts per IP per minute

---

## 5. Constraints and Assumptions

- The system is designed exclusively for REST API interaction (no WebSocket or GraphQL).
- JWTs are signed using the RS256 algorithm (asymmetric cryptography).
- A separate PostgreSQL and Redis instance is assumed in the infrastructure.
- Clients are required to store the refresh token in an HttpOnly Cookie.
- The service does not handle RBAC or email delivery.

---

## 6. Target Audience

| Audience | Needs | Documents |
|----------|-------|-----------|
| Backend developers | API details, component architecture, DB schema | SDD, SSD, API Reference |
| QA engineers | Requirements, test scenarios, traceability | Test Strategy, Traceability Matrix |
| DevOps | Infrastructure, deployment schema, scaling | ISD, Deployment Guide |
| Business / Client | Functionality, constraints, SLA | SSD (requirements section) |

---

*This document is the Single Source of Truth for all functional requirements.*  
*Changes to requirements must be reflected here first, then propagated to SDD and Traceability Matrix.*
