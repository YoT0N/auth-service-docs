# SDD — Software Design Document

**Auth Service**  
**Version:** v1.0  
**Status:** Approved  
**Last updated:** 2025

> References: [SSD — System Specification](SSD.md)

---

## 1. General Architecture

Auth Service is implemented as a standalone microservice within the overall microservices architecture. Architecture style: **microservices with API Gateway** as the single entry point.

Internal architecture follows the **Layered Architecture** principle:

```
Client / API Gateway
        │
        ▼
┌─────────────────────┐
│   Controller Layer  │  ← HTTP request handling, input validation
├─────────────────────┤
│   Service Layer     │  ← Business logic, token management
├─────────────────────┤
│  Repository Layer   │  ← Data access abstraction
├─────────────────────┤
│ PostgreSQL │ Redis  │  ← Persistent storage + session cache
└─────────────────────┘
```

---

## 2. Core Components

| Component | Technology | Responsibility |
|-----------|-----------|----------------|
| API Layer (Controller) | Node.js / Express | HTTP request handling, input validation |
| Business Logic (Service) | Node.js | Authentication logic, token operations |
| Token Module | jsonwebtoken (RS256) | JWT generation and verification |
| Data Layer (Repository) | Prisma ORM | CRUD operations with PostgreSQL |
| Cache Layer | ioredis | Refresh token storage in Redis |
| Security Middleware | Helmet, express-rate-limit | Attack protection, rate limiting |

---

## 3. API Interfaces

All endpoints are available via HTTPS only.

| Method | Endpoint | Status Codes | Description |
|--------|----------|-------------|-------------|
| POST | `/api/auth/register` | 201, 400, 409 | Register a new user |
| POST | `/api/auth/login` | 200, 400, 401 | Login, return JWT pair |
| POST | `/api/auth/refresh` | 200, 401 | Refresh access token |
| POST | `/api/auth/logout` | 204, 401 | Logout, invalidate session |
| GET | `/api/auth/verify` | 200, 401 | Internal token verification |

### Request/Response Examples

**POST /api/auth/login — Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**POST /api/auth/login — Response (200 OK):**
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```
> Refresh token is returned as an `HttpOnly` cookie: `Set-Cookie: refresh_token=...; HttpOnly; Secure`

---

## 4. Component Interaction

### Login Flow

```
Client → API Gateway → Controller → AuthService → UserRepository → PostgreSQL
                                         │
                                         ├→ bcrypt.compare(password, hash)
                                         │
                                         ├→ TokenModule.generate(userId)
                                         │      ├── access_token (RS256, 15 min)
                                         │      └── refresh_token (7 days)
                                         │
                                         └→ Redis.set(refresh_token, userId, TTL=7d)
                                                    │
                                         Response ←─┘
```

**Step-by-step:**

1. API Gateway receives `POST /api/auth/login` from client.
2. Controller validates request body (email, password).
3. AuthService queries UserRepository for the user record.
4. Password is verified via `bcrypt.compare()`.
5. TokenModule generates `access_token` (RS256, 15 min) and `refresh_token`.
6. Refresh token is stored in Redis with TTL of 7 days.
7. Both tokens are returned to client (access in body, refresh in HttpOnly Cookie).

### Token Refresh Flow (Refresh Token Rotation)

1. Client sends refresh token via cookie.
2. Redis is checked for token validity.
3. Old token is deleted from Redis (rotation).
4. New token pair is generated and returned.

---

## 5. Database Schema

### Users table (PostgreSQL)

```sql
CREATE TABLE users (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email       VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(60) NOT NULL,   -- bcrypt hash
  created_at  TIMESTAMP DEFAULT NOW(),
  updated_at  TIMESTAMP DEFAULT NOW()
);
```

### Redis key structure

```
refresh_token:{token_value} → user_id   (TTL: 7 days)
rate_limit:{ip_address}     → attempts  (TTL: 1 minute)
```

---

## 6. Security Design

- **Password storage:** bcrypt with cost factor 12 — resistant to brute-force attacks.
- **JWT signing:** RS256 (asymmetric) — private key signs, public key verifies.
- **Refresh Token Rotation:** each token use invalidates the old one, preventing replay attacks.
- **Rate limiting:** 5 failed login attempts per IP per minute (see [NFR-02 in SSD](SSD.md#4-non-functional-requirements)).
- **Transport:** HTTPS/TLS 1.3 enforced at API Gateway level.
- **HttpOnly Cookie:** prevents JavaScript access to refresh token (XSS mitigation).

---

*Architecture decisions in this document are based on requirements defined in [SSD](SSD.md).*
