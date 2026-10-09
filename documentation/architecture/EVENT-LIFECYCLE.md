# Event Lifecycle and Disposition Contract

**Status:** Agreed stage separation; exact persistence and acknowledgment models remain under design.

## Stages

1. **Observed / received:** A source adapter encounters the input. A UDP datagram merely reaching a socket is not a durable commitment.
2. **Identified:** Minimal provenance and type fields are captured without full normalization.
3. **Evaluated:** A consistent policy revision decides which receivers require the event. `FILTERED` means no selected receiver; `UNRESOLVED` means evaluation could not conclusively finish; `REJECTED` is an explicit policy/error disposition.
4. **Accepted for delivery:** The event enters per-receiver bounded delivery admission. The configured durability profile determines whether it is memory-only or disk-backed.
5. **Persisted (conditional):** When required, one retained payload and its outstanding receiver obligations are durably committed to the spool.
6. **Transmitted:** The local transport completes its send operation. It may *not* mean application-level ingestion.
7. **Confirmed / completed / expired / abandoned:** Each receiver obligation reaches its documented end state; expiration/eviction is a recorded loss, not success.

## Source-aware boundaries

- **UDP input:** no native acknowledgment or source replay; queued RAM records can be lost in a crash. Where conditional spool is active, durability begins after the spool commit, not before it.
- **TCP/TLS input:** byte-stream backpressure is available but application-level replay is not inherent. A TCP acknowledgment does not establish that RETRANS has durably retained a message.
- **Windows and WEC:** native records remain potentially replayable until channel overwrite, archive or removal. Bookmark advancement must follow a defined completion or spool boundary, never just the highest read event.

## Multiple receivers

An event selected by A and B produces independent obligations. A's progress cannot mark B complete or remove a payload still required by B. If A is healthy and B is offline, A may use the memory fast path while B uses conditional spool.

## Crash semantics

**Memory-only mode:** in-flight events can be lost; this risk must be surfaced. **Durable mode / conditional spool after commitment:** committed records must recover; incomplete writes must not create false completion. A receiver may see duplicates if transmission succeeded but its completion state was not durably recorded. Never promise exactly-once without source and receiver support.

## Replayable source position

The collection cursor is distinct from the last *committed, contiguous* progress position. Multiple workers may finish out of order; the checkpoint must wait for the earlier unfinished event. For native queries that suppress events, checkpoint semantics need separate documented treatment, particularly during query changes.

## Outstanding decision

The detailed transaction protocol for memory-to-disk transition, disk format, journal recovery and full-capacity strict-receiver admission remains open. See [Open Questions](../OPEN-QUESTIONS.md).