# RAG in a Regulated Environment

## Overview

Production architecture for Retrieval-Augmented Generation in HIPAA/SOC2/FDA-regulated contexts.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Regulated Environment                               │
│                                                                             │
│  ┌─────────┐    ┌──────────────┐    ┌─────────────────────────────────────┐│
│  │  Users  │───▶│   Frontend   │───▶│           API Gateway               ││
│  └─────────┘    │  (SSO Auth)  │    │  (Rate Limit, Audit, WAF)          ││
│                 └──────────────┘    └──────────────┬──────────────────────┘│
│                                                    │                        │
│                                     ┌──────────────▼──────────────┐        │
│                                     │     Policy / Governance      │        │
│                                     │  ┌────────┐ ┌────────────┐  │        │
│                                     │  │ PII    │ │ Cost       │  │        │
│                                     │  │ Filter │ │ Guard      │  │        │
│                                     │  └────────┘ └────────────┘  │        │
│                                     └──────────────┬──────────────┘        │
│                                                    │                        │
│   ┌────────────────────────────────────────────────┼────────────────────┐  │
│   │                        RAG Service             │                     │  │
│   │  ┌────────────────┐    ┌───────────────────────▼────────────────┐   │  │
│   │  │   Retriever    │◀───│              Orchestrator               │   │  │
│   │  │                │    │  (Query Rewrite, Re-rank, Synthesize)   │   │  │
│   │  └───────┬────────┘    └────────────────────┬───────────────────┘   │  │
│   │          │                                  │                        │  │
│   │          ▼                                  ▼                        │  │
│   │  ┌───────────────┐                 ┌───────────────────┐            │  │
│   │  │  Vector DB    │                 │   LLM Provider    │            │  │
│   │  │  (Embeddings) │                 │   (via Gateway)   │            │  │
│   │  └───────┬───────┘                 └───────────────────┘            │  │
│   │          │                                                          │  │
│   │          ▼                                                          │  │
│   │  ┌───────────────┐                                                  │  │
│   │  │ Document Store│                                                  │  │
│   │  │ (Encrypted)   │                                                  │  │
│   │  └───────────────┘                                                  │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                      Audit & Observability                           │  │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐     │  │
│   │  │ Metrics  │  │ Traces   │  │  Logs    │  │  Audit Trail     │     │  │
│   │  │(Prometheus)│ │(Jaeger)  │  │ (ELK)   │  │ (Immutable)      │     │  │
│   │  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Key Components

| Component | Purpose | Compliance Role |
|-----------|---------|-----------------|
| API Gateway | Entry point | Rate limiting, WAF, audit logging |
| Policy Layer | Governance | PII detection, cost controls |
| Vector DB | Semantic search | Encrypted at rest |
| Document Store | Source docs | Encryption, access controls |
| LLM Provider | Generation | Via audited gateway |
| Audit Trail | Compliance | Immutable, queryable logs |

## Security Controls

- **Authentication**: SSO with MFA
- **Authorization**: RBAC with document-level permissions
- **Encryption**: TLS 1.3 in transit, AES-256 at rest
- **Audit**: Every query logged with user context
- **PII**: Automatic detection and masking
- **Network**: Private endpoints, egress controls

## Compliance Mapping

| Requirement | Control |
|-------------|---------|
| HIPAA §164.312(a) | Access controls, audit logs |
| HIPAA §164.312(e) | TLS encryption |
| SOC 2 CC6.1 | Authentication, authorization |
| SOC 2 CC7.2 | Monitoring, alerting |
