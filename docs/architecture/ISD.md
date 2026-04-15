# ISD — Infrastructure Specification Document

**Auth Service**  
**Version:** v1.0  
**Status:** Approved  
**Last updated:** 2025

> References: [SDD — Software Design](SDD.md) | [SSD — Non-functional requirements](SSD.md#4-non-functional-requirements)

---

## 1. Deployment Environment

Auth Service is deployed in a cloud environment (AWS / GCP) using Docker containerization and Kubernetes orchestration.

Three environments are supported:

| Environment | Purpose | Notes |
|-------------|---------|-------|
| `development` | Local development | Docker Compose, mocked external services |
| `staging` | Pre-production testing | Mirrors production config, limited resources |
| `production` | Live system | Full HA configuration, monitoring enabled |

---

## 2. Infrastructure Components

| Component | Technology | Configuration |
|-----------|-----------|---------------|
| Service container | Docker (Node.js 20 Alpine) | 2 replicas in Kubernetes Deployment |
| API Gateway | Nginx / Kong | SSL termination, rate limiting, routing |
| Database | PostgreSQL 15 | Primary + Read Replica, 20GB SSD |
| Session cache | Redis 7 (Cluster mode) | 3 nodes, maxmemory-policy allkeys-lru |
| Monitoring | Prometheus + Grafana | Latency, error rate, throughput metrics |
| Logging | ELK Stack | Structured JSON logs, 30-day retention |
| CI/CD | GitHub Actions | Build → Test → Docker push → K8s deploy |

---

## 3. Kubernetes Configuration

```yaml
# auth-service deployment (simplified)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-service
spec:
  replicas: 2
  selector:
    matchLabels:
      app: auth-service
  template:
    spec:
      containers:
        - name: auth-service
          image: auth-service:v1.0
          ports:
            - containerPort: 3000
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: auth-secrets
                  key: database-url
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
```

---

## 4. Scaling Principles

- **Horizontal scaling:** HPA (Horizontal Pod Autoscaler) increases replica count when CPU > 70%.
- **Stateless design:** all sessions are stored in Redis, allowing any replica to handle any request.
- **Redis Cluster:** provides horizontal cache scaling and fault tolerance.
- **Database Connection Pooling:** PgBouncer limits connection count and distributes load evenly.
- **Health checks:** `/health` endpoint for Kubernetes liveness and readiness probes.

### Autoscaling configuration

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: auth-service-hpa
spec:
  scaleTargetRef:
    name: auth-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## 5. CI/CD Pipeline

```
GitHub Push → GitHub Actions
    │
    ├── 1. Install dependencies (npm ci)
    ├── 2. Run tests (jest --coverage)
    ├── 3. Build Docker image
    ├── 4. Push to Container Registry
    └── 5. Deploy to Kubernetes (kubectl apply)
```

**Rollback strategy:** `kubectl rollout undo deployment/auth-service`

---

## 6. Monitoring and Alerting

| Metric | Alert Threshold | Action |
|--------|----------------|--------|
| Error rate | > 1% per 5 min | PagerDuty alert |
| Response time p95 | > 200 ms | Warning notification |
| CPU utilization | > 70% | Auto-scaling triggered |
| Redis memory | > 80% | Alert to DevOps team |

**Grafana dashboards:** Auth Service Overview, Token Operations, Database Performance

---

## 7. Security Infrastructure

- All secrets stored in **Kubernetes Secrets** (base64 encoded, RBAC restricted).
- Private JWT signing key stored in **AWS KMS / GCP KMS**.
- Network policies restrict pod-to-pod communication to required services only.
- TLS certificates managed by **cert-manager** (Let's Encrypt).

---

*Infrastructure decisions are aligned with scalability and availability requirements defined in [SSD NFR section](SSD.md#4-non-functional-requirements).*
