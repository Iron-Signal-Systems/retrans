# Storage Retention, Outage Pools, and Recovery

**Status:** Logical budget and UDP-loss policy agreed; exact sizing defaults and physical storage format remain open.

## Receiver storage budgets

Normal, burst and protected outage capacity are administrator-configurable. **Burst ceiling includes normal capacity**; outage reserve is additional protected capacity available only under a qualified receiver failure. The example values below are illustrative:

| Allocation | Example | Meaning |
| --- | ---: | --- |
| Normal | 1 GiB | Ordinary buffered backlog allowance |
| Burst ceiling | 1.5 GiB total | Adds 0.5 GiB above normal |
| Outage reserve | 2 GiB additional | Protected space used only after ordinary allowance fills and outage qualifies |
| Total illustrated ceiling | 3.5 GiB | Not a shipped default |

Logical quotas must be reconciled against actual physical payloads, indexes, journal metadata, recovery headroom and free volume capacity. A logical reservation does not guarantee actual disk space if other processes fill the volume.

**Proposed outage qualification:** network failure by default; receiver-service failure may be eligible if configured; TLS identity/certificate or local configuration faults do not qualify automatically. The actual failure thresholds remain open.

## Dedicated UDP-source retention

UDP-originated pending events require an independently bounded, administrator-configured retention policy. Two limits are enforced independently:

1. **Maximum age:** measured from original RETRANS receipt time, not retry time and not the sender-supplied clock.
2. **Maximum pool size:** pending data may never grow without bound.

When an incoming pending record would exceed available quota, evict the **oldest pending records by RETRANS ingestion sequence** until the new record fits. Records expire as they reach maximum age even if capacity remains. If one record is larger than the entire pool, reject that record without clearing the whole pool. Counts and bytes for age expiry, size eviction and failed admission must be reported distinctly.

Example only: `maximum_age = 7 days`, `maximum_pool = 2 GiB`. Seven days is a cap, **not a guarantee** that records survive seven days if the pool fills earlier.

The UDP bounded-retention policy is a deliberate loss-tolerant exception; it must not quietly delete another receiver's strict obligations to a shared payload.

## Full storage behavior

Strict-retention obligations are not automatically overwritten. Finite capacity can require source pause/backpressure, explicit failure or an administrator-authorized disposition. UDP-originated pending obligations under the approved loss policy may evict oldest. A UDP sender generally cannot be slowed reliably; report local admission loss without claiming visibility into packets never received.

## Recovery

After restart, RETRANS must distinguish committed disk-backed records from RAM-only records that cannot be recovered. Recover outstanding receiver obligations and retention clocks, then expire records already older than the configured limit. Do not treat torn or corrupt records as valid progress. Receiver drain rates must remain bounded; completed obligations release storage only when no other receiver requires the shared payload.

Receiver communication success is distinct from application ingestion confirmation. Retry after a crash can generate a duplicate if the transport send succeeded but the completion record was not durable.

## Open details

Exact interaction between UDP-specific quota and normal/burst/outage budgets, per-source versus per-receiver quota attribution, journal format, source checkpoint transaction, compaction and crash-synchronization procedures remain pending. Do not add illustrative budget values together as guaranteed physical reservations.