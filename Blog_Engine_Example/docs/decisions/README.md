# Architecture Decision Records (ADRs)

> **Purpose**: Document significant architectural decisions with context and rationale  
> **Pattern**: Michael Nygard ADR format  
> **Audience**: Architects, Developers, Technical Leadership

---

## What are ADRs?

Architecture Decision Records capture important architectural decisions made on this project, along with their context and consequences. Each ADR is immutable once accepted - if a decision changes, we create a new ADR that supersedes the old one.

---

## Active Decisions

| ID | Title | Status | Date | Impact | Supersedes |
|----|-------|--------|------|--------|------------|
| [ADR-001](adr-001-microservices.md) | Microservices Architecture | ✅ Accepted | November 1, 2024 | High | - |
| [ADR-002](adr-002-postgresql.md) | PostgreSQL as Primary Database | ✅ Accepted | November 2, 2024 | High | - |
| [ADR-003](adr-003-jwt-auth.md) | JWT for API Authentication | ✅ Accepted | November 5, 2024 | High | - |
| [ADR-004](adr-004-rabbitmq.md) | RabbitMQ for Event Bus | 🔄 Proposed | November 5, 2024 | Medium | - |
| [ADR-005](adr-005-redis-cache.md) | Redis for Caching | 🔄 Proposed | November 5, 2024 | Medium | - |

---

## Decision Status

- **✅ Accepted**: Decision is active and should be followed
- **🔄 Proposed**: Under review, not yet binding
- **⚠️ Deprecated**: No longer recommended, see superseding ADR
- **❌ Rejected**: Proposal was rejected, see alternatives

---

## How to Use ADRs

### For Developers
- **Starting a feature?** Check if relevant ADRs exist
- **Uncertain about approach?** Look for related decisions
- **Following a pattern?** Reference the ADR in code comments

### For Architects
- **New significant decision?** Create a new ADR using [template.md](template.md)
- **Changing a decision?** Create new ADR that supersedes the old one
- **Monthly review**: Check if any ADRs need updates

### For AI Assistants
- Load relevant ADRs when implementing features
- Follow architectural patterns defined in ADRs
- Reference ADR numbers in generated code comments

---

## Creating a New ADR

1. Copy [template.md](template.md)
2. Name it `adr-XXX-short-title.md` (next number in sequence)
3. Fill in all sections
4. Submit for architecture review
5. Once accepted, add to table above

---

## ADR Template

See [template.md](template.md) for the standard format.

**Required sections**:
- **Context**: What's the situation requiring a decision?
- **Decision**: What did we decide to do?
- **Consequences**: What are the positive and negative outcomes?
- **Alternatives Considered**: What other options did we evaluate?

---

## Related Documentation

- **System Architecture**: [/docs/architecture/system-overview.md](../architecture/system-overview.md)
- **Security Decisions**: [/docs/security/README.md](../security/README.md)
- **Implementation**: See individual features in [/features/](../../features/)

---

**Maintained By**: Architecture Team  
**Review Cycle**: Monthly  
**Last Review**: November 5, 2024
