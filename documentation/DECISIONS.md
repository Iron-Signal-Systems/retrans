# RETRANS Engineering Decision Register

This is a **design review record**, not a log of implemented features. Entries capture the latest agreed direction, including revisions that supersede earlier proposals. Dates reflect the discussion snapshot (2026-10-09).

| ID | Status | Decision / direction |
| --- | --- | --- |
| D-001 | Agreed | Product name RETRANS; Go implementation; initial Microsoft Windows Server target; Apache License 2.0. |
| D-002 | Agreed | One Windows service initially; independent Source Processing Groups for WEC, native Windows Event Log, syslog UDP, TCP and TLS. |
| D-003 | Agreed | Each group owns bounded queues and configurable collection/processing workers; **one worker per group is not a limit**. Reusable code does not imply shared execution. |
| D-004 | Agreed | Memory-first normal processing. Filtered and normally transmitted event payloads do not require a RETRANS disk write. Conditional persistence is used for delivery failure, congestion, source-retention risk or an explicitly stronger durability policy. |
| D-005 | Agreed | No mandatory raw-ingress journal or duplicate payload write on the healthy fast path. Earlier always-journaled ingress proposals are superseded. |
| D-006 | Agreed | Source capabilities differ: UDP generally cannot replay, TCP backpressure is not application replay, Windows channels can replay while records remain retained. |
| D-007 | Agreed | Each receiver has independent allowlists, output transformation, connection, backlog and health state. Interest is evaluated across all applicable receivers before discarding events. |
| D-008 | Agreed | Two-stage filtering, selective/lazy field extraction, and explicit filtered/rejected/unresolved dispositions. No arbitrary weights or opaque event scores. |
| D-009 | Agreed | Normal/burst/outage allocations are administrator-configurable; protected outage capacity is not ordinary burst space. Sample 1 GiB/1.5 GiB/additional 2 GiB figures are **examples**, not defaults. |
| D-010 | Agreed | UDP-originated events may use an independently bounded retention pool with both maximum age and size. When necessary, evict the **oldest pending** records; account for all deliberate losses. Example seven days/two GiB is not a fixed default. |
| D-011 | Agreed | Use JSON for human-edited configuration, strict schema validation, and immutable compiled in-memory policies. Durable spool/transport state is not stored in the configuration format. |
| D-012 | Agreed | Encrypted outbound transport and verified receiver identity are baseline requirements; no silent downgrade to plaintext. |
| D-013 | Agreed | Forwarding completion must never be misrepresented as receiver-confirmed ingestion when the protocol has no such acknowledgment. |
| D-014 | Agreed | GitHub README stays a high-level introduction; engineering contracts belong under `documentation/` and roadmap links may be included. |

## Working proposals — not yet implementation contracts

| ID | Status | Proposal |
| --- | --- | --- |
| P-001 | Working proposal | Single logical spool/journal authority with append-oriented segments, reusable references and independently recoverable receiver obligations. |
| P-002 | Working proposal | Native Windows event bookmarks tied to explicitly configured completion boundaries, never simply the last event read. |
| P-003 | Working proposal | Oldest-first ordering within a source-to-receiver stream by default; alternative live-priority delivery only by explicit policy. |
| P-004 | Working proposal | Network failures qualify for protected outage reserve by default; receiver-service failures may be separately eligible; security/configuration faults do not qualify by default. |
| P-005 | Working proposal | Single authoritative local configuration composed from separately organized source, receiver and policy JSON documents; validated and activated as one coherent revision. |
| P-006 | Working proposal | Bounded fair scheduling and receiver recovery throttling. |

## Superseded proposals and clarifications

1. **Always-journal-before-delivery** is not the general default. Memory-first processing was explicitly preferred. A stricter durable mode remains possible and must be accurately labeled.
2. **Mandatory ingress staging of all UDP/TCP records** was superseded by conditional spooling. This introduces documented crash-loss exposure while an event is only in RAM.
3. **One worker per source type** was illustrative; a Source Processing Group may contain multiple workers.
4. **No event loss ever** is not a universal contract. The administrator-authorized UDP retention policy explicitly permits age expiration and oldest-first capacity eviction.

## Change procedure

When a decision changes: record what it supersedes, update every affected contract, and add the failure and acceptance tests required by the new design. Do not claim an implementation is complete until validated.