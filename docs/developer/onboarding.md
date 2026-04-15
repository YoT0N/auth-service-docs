# Developer Onboarding Guide — Auth Service

**Version:** v1.0  
**Last updated:** 2025

> Architecture reference: [SDD](../architecture/SDD.md) | Deployment: [ISD](../architecture/ISD.md)

---

## 1. Prerequisites

Before starting, make sure you have installed:

- **Node.js** 20.x (LTS)
- **Docker** + Docker Compose
- **Git**
- **psql** (PostgreSQL client, optional)

---

## 2. Local Setup

### Step 1 — Clone the repository

```bash
git clone https://github.com/your-org/auth-service.git
cd auth-service
```

### Step 2 — Install dependencies

```bash
npm install
```

### Step 3 — Configure environment

```bash
cp .env.example .env
```

Edit `.env` with your local values:

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/auth_db
REDIS_URL=redis://localhost:6379
JWT_PRIVATE_KEY_PATH=./keys/private.pem
JWT_PUBLIC_KEY_PATH=./keys/public.pem
PORT=3000
```

### Step 4 — Generate JWT keys

```bash
mkdir -p keys
openssl genrsa -out keys/private.pem 2048
openssl rsa -in keys/private.pem -pubout -out keys/public.pem
```

### Step 5 — Start infrastructure

```bash
docker-compose up -d   # Starts PostgreSQL + Redis
```

### Step 6 — Run migrations

```bash
npx prisma migrate dev
```

### Step 7 — Start the service

```bash
npm run dev       # Development mode with hot reload
npm run start     # Production mode
```

Service is available at: `http://localhost:3000`

---

## 3. Running Tests

```bash
npm run test              # All tests
npm run test:unit         # Unit tests only
npm run test:integration  # Integration tests only
npm run test:coverage     # With coverage report
```

See [Test Strategy](../quality/test-strategy.md) for full testing approach.

---

## 4. Git Flow

```
main ────────────────────────────────────── production
  └── develop ──────────────────────────── integration
         └── feature/FR-01-registration ── your work
```

### Branch naming conventions

| Type | Pattern | Example |
|------|---------|---------|
| Feature | `feature/FR-XX-description` | `feature/FR-01-user-registration` |
| Bug fix | `fix/short-description` | `fix/refresh-token-expiry` |
| Documentation | `docs/what-changed` | `docs/update-api-reference` |
| Hotfix | `hotfix/critical-issue` | `hotfix/jwt-validation-bypass` |

### Commit message format

```
type(scope): short description

Examples:
feat(auth): add refresh token rotation
fix(token): correct RS256 key loading
docs(ssd): update NFR-01 performance target
test(login): add brute force test case
```

### Pull Request rules

1. Branch must be up-to-date with `develop`.
2. All tests must pass (CI enforced).
3. At least 1 reviewer approval required.
4. PR description must reference requirement ID (e.g., `Implements FR-03`).

---

## 5. Project Structure

```
auth-service/
├── src/
│   ├── controllers/     ← HTTP handlers (Express routes)
│   ├── services/        ← Business logic
│   ├── repositories/    ← Data access (Prisma)
│   ├── middleware/       ← Auth, rate limiting, error handling
│   ├── modules/
│   │   └── token/       ← JWT generation and verification
│   └── app.ts           ← Express app setup
├── tests/
│   ├── unit/
│   └── integration/
├── prisma/
│   └── schema.prisma    ← Database schema
├── docs/                ← Documentation (this repository)
├── keys/                ← JWT keys (git-ignored)
├── docker-compose.yml
└── package.json
```

---

## 6. Useful Commands

```bash
npx prisma studio          # Open database GUI
npx prisma migrate reset   # Reset database (dev only)
docker-compose logs -f     # View infrastructure logs
npm run lint               # Run ESLint
npm run build              # TypeScript compilation
```

---

*For deployment instructions, refer to [ISD](../architecture/ISD.md).*  
*For API details, refer to [SDD — API Interfaces](../architecture/SDD.md#3-api-interfaces).*
