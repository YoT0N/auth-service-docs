# Auth Service — Documentation Overview

> **Single Source of Truth** for the Auth Service microservice documentation.  
> All documents are maintained in this repository as part of the Documentation as Code approach.

## 📁 Documentation Structure

```
docs/
├── README.md                        ← You are here (entry point)
├── architecture/
│   ├── SSD.md                       ← System Specification Document
│   ├── SDD.md                       ← Software Design Document
│   └── ISD.md                       ← Infrastructure Specification Document
├── quality/
│   ├── test-strategy.md             ← Test Strategy
│   └── traceability-matrix.md       ← Requirements Traceability Matrix
├── developer/
│   └── onboarding.md                ← Developer Onboarding Guide
└── mkdocs.yml                       ← MkDocs configuration
```

## 🔗 Navigation

### Architecture
| Document | Description |
|----------|-------------|
| [System Specification (SSD)](architecture/SSD.md) | Functional requirements, system boundaries, non-functional requirements |
| [Software Design (SDD)](architecture/SDD.md) | Architecture, components, API interfaces, data flow |
| [Infrastructure Specification (ISD)](architecture/ISD.md) | Deployment, scaling, infrastructure components |

### Quality Assurance
| Document | Description |
|----------|-------------|
| [Test Strategy](quality/test-strategy.md) | Testing approach, levels, tools |
| [Traceability Matrix](quality/traceability-matrix.md) | Requirements ↔ test cases mapping |

### Developer
| Document | Description |
|----------|-------------|
| [Onboarding Guide](developer/onboarding.md) | Setup, git flow, contribution rules |

## 🏛️ Single Source of Truth Rules

| Topic | Source Document | Referenced By |
|-------|----------------|---------------|
| Functional requirements | [SSD](architecture/SSD.md) | SDD, Test Strategy, Traceability Matrix |
| Architecture decisions | [SDD](architecture/SDD.md) | ISD, Onboarding |
| Infrastructure config | [ISD](architecture/ISD.md) | Onboarding (deploy section) |
| Test requirements | [Test Strategy](quality/test-strategy.md) | Traceability Matrix |

> ⚠️ **Rule:** If a requirement changes — update SSD **first**. All other documents must reference SSD identifiers (FR-XX, NFR-XX), not duplicate them.

## 📌 About the System

**Auth Service** is a microservice responsible for user authentication and authorization in a distributed system. It provides a secure REST API integrated with other microservices via JWT tokens.

**Tech stack:** Node.js · Express · PostgreSQL · Redis · Docker · Kubernetes

**Version:** `v1.0`  
**Last updated:** 2025  
**Author:** Ільків Богдан
