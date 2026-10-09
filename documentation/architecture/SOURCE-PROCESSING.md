# Independent Source Processing Groups

**Status:** Agreed isolation model; counts and scheduling algorithms remain under evaluation.

## Group ownership

Each source type is independently supervised and has its own bounded receive/collection path, RAM queue, processing worker pool, parsing state, overload behavior, health metrics, and shutdown lifecycle.

| Group | Input responsibilities | Failure behavior |
| --- | --- | --- |
| Native Windows Event Log | Windows Event Log API, channel position, bookmarks, original computer metadata | Pause/replay while records remain retained; detect gaps |
| WEC / WEF ForwardedEvents | Local forwarded-event stream; preserve original producer attribution | Pause/replay while local WEC channel retains records; monitor rollover |
| UDP syslog | Datagram read, bounds/framing, socket/peer metadata, loss exposure | No upstream backpressure; conditional best-effort spool and admission loss accounting |
| TCP syslog | Bounded accept, stream framing, connection management | TCP backpressure where supported; never presume application replay |
| TLS syslog | Dedicated TLS listener/session controls, framing and input health | TLS backpressure, protocol errors, certificate validation |

One collector and one processor per group are **not mandated**. Each group can have multiple processing workers under configured limits; per-connection handlers must also be bounded.

## Device vs group

A single UDP group may receive events from many devices. Device/source instances retain independent identities, measurements and optional limits without necessarily owning dedicated goroutines or queues. No per-event goroutine fanout.

## Scheduling and ordering

Shared CPU, disk and memory resources still require global ceilings. Within those ceilings, a flood in one group must not prevent others from receiving processing opportunities. Collection workers may process events concurrently; source-position checkpointing for replayable channels must only advance past contiguous completed dispositions. The exact fairness algorithm is open.

## Resource budgets

Bound by **bytes and records**, not counts alone. Limit maximum event length, active connections, buffered network frames, per-source queue residence, workers, spool admission, retry activity, and total memory. When limits are reached: UDP may drop with local accounting; TCP/TLS may backpressure; Windows sources may pause while their finite channel retention remains available.

## Worker supervision

Workers must stop predictably, release buffers on cancellation and emit stalled-worker and queue metrics. Process isolation within one Go service is logical, not isolation from shared host CPU or from a fatal process-level crash. Security fault containment may demand stronger boundaries in future design.