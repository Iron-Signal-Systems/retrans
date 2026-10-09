# Engineering Acceptance Criteria

**Status:** Planned verification contract; these tests have not yet been executed or passed.

## Source isolation and bounds

| Scenario | Required result |
| --- | --- |
| Flood UDP input while WEC and TCP run | No starvation of unrelated source groups within declared global host limits |
| Stall TCP clients or many TLS sessions | Bounded connections, handlers and buffers; unrelated groups continue |
| Malformed/oversized syslog | Explicit rejection or error classification; no unbounded allocation |
| Multiworker WEC processing finishes out of order | No checkpoint advances past an unresolved earlier source position |
| Sustained load over representative host profile | CPU, RAM, throughput and queue bounds measured; numerical goals must be defined before a release gate |

## Filtering and receiver independence

| Scenario | Required result |
| --- | --- |
| A accepts 15 IDs, B accepts a superset | Correct per-receiver selection; no premature global discard |
| A and B need same extracted field | Field extracted once per event context |
| No receiver interested | No unnecessary full normalization or RETRANS payload persistence |
| Receiver B offline, A online | A continues as allowed by source admission and finite disk limits; B's pending records follow its policy |
| Policy reload mid-event | Every event uses exactly one coherent policy revision |
| Transform fails for B | A remains unaffected where its output can be completed independently |

## Durability, loss and recovery

| Scenario | Required result |
| --- | --- |
| Crash during memory-only input processing | Expose/document unrecoverable non-replayable events; do not falsely claim durability |
| Crash after durable spool commit | Selected stored payload and obligations recovered |
| Crash during partial spool write | Incomplete transaction cannot be treated as committed |
| Retry after ambiguous send | Duplicate possible but no false exactly-once claim |
| UDP reaches size quota | Oldest pending events evicted and accurately counted |
| UDP reaches maximum retention age | Expire over-age records, independent of available capacity |
| An event exceeds entire UDP pool | Reject that event without erasing unrelated pending records |
| Strict receiver storage full | No silent eviction of strict obligations |
| WEC channel overwrites unread events | Detect and report the collection gap wherever OS information permits |
| Restart with expired UDP records | Recover state and enforce expiration before forwarding |

## Security and administration

| Scenario | Required result |
| --- | --- |
| Invalid or unknown JSON configuration key | Candidate rejected; running revision unchanged |
| Invalid Windows subscription selector | No partial silent activation |
| Receiver certificate identity mismatch | No plaintext fallback, clear security fault |
| Reduce pool below retained occupancy | Unsafe change rejected or requires deliberate explicit disposition |
| Disable/delete receiver with backlog | No accidental purge of strict outstanding obligations |

## Release gate prerequisites

Before claiming production readiness: fix the output protocol list and acknowledgment semantics; define exact resource and throughput targets; complete security privilege model and credential rotation behavior; test on supported Windows Server releases; perform long-duration outage, crash, disk-full, power-loss approximation and upgrade/rollback scenarios. Collect repeatable measured outputs. Avoid using simulated test results as product benchmarks.