# RETRANS Engineering Roadmap

**Status: Proposed development sequence, not a release commitment.** No dates, throughput results, or completed implementations are asserted here.

## Goal

Deliver a lightweight, secure, operator-controlled Go service, initially for Microsoft Windows Server, that collects multiple log-source types, selects events independently for multiple receivers, and forwards only the records each receiver is configured to accept. It is neither a SIEM nor an event archive.

## Engineering stages

| Stage | Deliverable | Exit gate |
| --- | --- | --- |
| 0. Contracts | Architecture, decision register, failure semantics | Agreed behavior and unresolved issues explicitly identified |
| 1. Configuration | Versioned strict JSON model and validation CLI | Invalid and unsafe changes rejected without interrupting the active revision |
| 2. Windows runtime | Native service, bounded worker supervisor, health reporting | Clean startup/shutdown, constrained memory, worker isolation |
| 3. Source adapters | Windows Event Log/WEC, UDP syslog, TCP syslog, TLS syslog | Proven provenance, framing, source-specific overflow behavior |
| 4. Filtering and outputs | Two-stage matching, field extraction, receiver adapters | Independent allowlists; avoid unnecessary parsing and serialization |
| 5. Reliability | Conditional spooling, per-receiver state, quotas, retention, recovery | Crash tests, receiver outage, UDP eviction, source rollover tests |
| 6. Security and lifecycle | Service identity, trust material, installer/update/recovery | Least-privilege and credential rotation tests; rollback specification |
| 7. Performance and readiness | Long-running mixed input/output workloads and compatibility tests | Defined CPU/memory/latency/throughput objectives met on representative hardware |
| 8. Release | Operator guides, release artifacts, support policies | Repeatable build, installation, upgrade and acceptance procedures |

Stages may overlap where engineering dependencies permit, but reliability claims must not precede validation. Later platform support (including FreeBSD) remains a consideration, not an initial deliverable.

## Definition of an engineering-ready specification

A design section is ready for implementation only when it defines: input assumptions, state transitions, resource limits, measurable outcomes, crash/restart behavior, operational diagnostics, and acceptance tests. Example values do not become defaults merely because they appear in design discussions.

## Current documentation status

**Agreed direction:** memory-first processing, independent source groups and receivers, staged filtering, strict JSON configuration, bounded UDP retention, and encrypted outbound delivery.  
**Not yet finalized:** durable-spool physical format; exact failure/acknowledgment profiles; output protocol priority; service privilege model; installation and upgrade contract; numerical acceptance targets.