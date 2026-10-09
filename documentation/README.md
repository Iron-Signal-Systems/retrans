# RETRANS Engineering Documentation

**Reliable Event Transport, Routing, and Normalization System**  
**Project:** Iron Signal Systems  
**Status:** Engineering design; not a production release  
**Documentation snapshot:** 2026-10-09

This directory is the engineering reference for RETRANS. The repository-root [README](../README.md) remains the public project overview. The documents here distinguish **Agreed**, **Working proposal**, and **Open** statements:

- **Agreed:** An architectural direction selected during engineering review, subject to later controlled changes.
- **Working proposal:** A recommended implementation approach that still requires evaluation or a separate decision.
- **Open:** A design question without an approved contract.

No document here claims that the planned runtime has been implemented or tested. Sample numbers, event IDs, hostnames, and capacity values are illustrative unless explicitly identified as requirements.

## Navigation

### Product and decisions
- [Roadmap](ROADMAP.md) — work sequence and engineering gates; not delivery dates.
- [Decision register](DECISIONS.md) — current agreed design, proposals, and corrections.
- [Open questions](OPEN-QUESTIONS.md) — choices that still require review.

### Architecture
- [System overview](architecture/OVERVIEW.md) — scope, topology, governing invariants.
- [Source processing groups](architecture/SOURCE-PROCESSING.md) — independent workers, queues, and admission.
- [Event lifecycle](architecture/EVENT-LIFECYCLE.md) — states, acknowledgments, checkpoint semantics.
- [Filtering and normalization](architecture/FILTERING-AND-NORMALIZATION.md) — receiver-specific matching and lazy extraction.
- [Receiver delivery](architecture/RECEIVER-DELIVERY.md) — independent output workers and delivery contracts.

### Reliability and configuration
- [Memory-first and conditional spooling](reliability/MEMORY-FIRST-AND-SPOOLING.md) — fast path, optional disk path, explicit exposure.
- [Retention and recovery](reliability/RETENTION-AND-RECOVERY.md) — capacity tiers, UDP aging, crash handling.
- [JSON configuration](configuration/JSON-CONFIGURATION.md) — format and transactional activation.

### Operations and validation
- [Health and diagnostics](operations/HEALTH-AND-DIAGNOSTICS.md) — telemetry and truthful status reporting.
- [Acceptance criteria](testing/ACCEPTANCE-CRITERIA.md) — verification plan for future implementation.

## Documentation governance

Detailed technical contracts belong here rather than in the root README. Keep documentation organization-neutral: do not name a specific customer, government organization, operational deployment, or non-public infrastructure. Changes to agreed behavior must also update [DECISIONS.md](DECISIONS.md), affected specifications, and tests. Repository agents and contributors must also follow [AGENTS.md](../AGENTS.md).