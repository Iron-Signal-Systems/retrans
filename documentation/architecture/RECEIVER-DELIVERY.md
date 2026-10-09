# Independent Receiver Delivery

**Status:** Agreed independence and security goals; adapters, acknowledgment and scheduling details under design.

## Per-receiver ownership

Each configured receiver independently owns its event-selection policy, output format, credentials and TLS settings, connection behavior, retry/backoff, bounded in-memory queue, conditional disk-spool obligations, storage accounting, and health reporting.

One source event may be selected by multiple receivers. Delivery to a healthy receiver must continue independently of an unavailable receiver within source-admission and physical resource limits.

## Normal fast path

Selected payloads remain in bounded RAM, are transformed only as required, and are sent directly. No event-payload disk write is mandatory for the healthy fast path. Ordinary transmission does not automatically imply durable capture, and a crash can lose unsynchronized in-flight records.

## Failure / congestion path

When a receiver is unavailable or bounded queues cannot progress, conditionally spool selected pending records according to the applicable source/receiver durability policy. A shared original payload may be referenced by more than one pending obligation rather than copied for each. Disk-backed obligations remain until independently completed or explicitly disposed of under their retention contract.

A blocked destination should not stall other destinations indefinitely. However, finite storage plus strict obligations imposes real admission limits, particularly when a single source event is required by both full and healthy receivers. See [Open Questions](../OPEN-QUESTIONS.md).

## Outage classifications

- Network transport failure: protected outage capacity eligible by default under the proposed policy.
- Receiver service failure: may be independently eligible if configured.
- Degraded forwarding: report stalled progress; qualification details open.
- Certificate validation / identity / configuration failure: report distinctly; do not silently label a network outage or fall back to plaintext.

## Acknowledgment vocabulary

- **Transmission attempted:** a send was tried.
- **Transport transmitted:** local transport reported success; receiver application acceptance not proven.
- **Receiver confirmed:** the adapter has a documented application-level confirmation, if supported.
- **Pending / retry:** the obligation is still outstanding under its chosen contract.
- **Expired / evicted / abandoned:** data was not delivered under the strict success definition; count and report loss.

UDP syslog has no general application acknowledgment. Conventional TCP/TLS syslog likewise cannot prove that the destination durably stored an event merely because a write succeeded. Do not advertise exactly-once delivery without protocol support.

## Recovery and ordering

Receiver recovery must be bounded by CPU, disk and network budgets. Oldest-first within a given source-to-receiver stream is the working default proposal; live-priority/out-of-order recovery is a possible explicit policy, not yet decided. Avoid global ordering claims across unrelated input streams.

## Encrypted outbound baseline

Approved outbound network delivery requires encryption, receiver certificate/identity validation and no automatic plaintext fallback. Support operator-provisioned CA/trust and optional mTLS when required. Native plaintext UDP output has **not** been approved; if ever added it requires an explicit and documented exception.