# Operational Health and Diagnostics

**Status:** Metric categories agreed; telemetry format and administrative APIs remain open.

RETRANS must report what is occurring without depending on a downstream SIEM to determine whether RETRANS is healthy. A running Windows service or established TCP/TLS connection is not proof of ongoing forwarding progress.

## Source metrics

Per Source Processing Group and identifiable source instance: messages received/observed, rate and sizes, parsing/identification failures, filtered and selected counts, unresolved policy decisions, RAM occupancy in bytes and records, admission drops, collection gap indicators, last input time, and worker stalls.

For native Windows/WEC sources: last checkpoint progress, oldest unread available event where determinable, channel retention/overwrite exposure, missing record/bookmark failures. For UDP: never claim to measure packets that never reached RETRANS.

## Receiver metrics

Per receiver: selected obligations, attempted transmissions, locally reported transport successes, actual receiver confirmations **only when protocol supports them**, retries, oldest pending age, bytes/records pending, normal/burst/outage/UDP pool use, certificate and routing errors, last successful progress and stalled-forwarding durations.

## Loss accounting

Separate source admission drops, locally detectable socket problems, UDP pool age expiration, UDP capacity eviction, oversized record rejection, deliberate administrator abandonment, channel rollover/source gap, and corrupt spool data. Do not collapse these into one generic dropped-events counter; do not count successful transmission as confirmed ingestion.

## Outage history

Record incident start, observed failure class, qualification and reserve activation, storage pressure, affected sources/receivers, recovery progress and ending condition. A service restart must not erase historical fault context needed for operational investigation.

## Performance metrics

Measure parsing cost, field extractions, policy evaluation time, allocations per event, queue wait, process CPU/RAM, disk bytes/latency, spool commits, TLS and network throughput, retry rates, and backlog drain. Attribute shared physical disk use separately from logical per-receiver quotas.

## Diagnostic security

Diagnostics must not emit private keys, passwords or unnecessary original log payloads. Administrator access and any remote access mechanism remain subject to the future Windows privilege/security contract. Design should favor bounded log volume so diagnostics do not become their own disk-exhaustion event.

## Operational acceptance

A receiver that is connected but not progressing must generate an actionable health state. A UDP receiver's successful local sends must display **remote delivery unknown** unless explicitly confirmed by a supported mechanism.