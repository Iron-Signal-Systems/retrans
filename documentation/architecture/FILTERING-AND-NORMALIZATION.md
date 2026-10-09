# Receiver-Specific Filtering and Selective Normalization

**Status:** Agreed pipeline; exact rule grammar and compiled indexing remain open.

## Central contract

Identify an event once inside its Source Processing Group, evaluate **every potentially interested receiver**, extract only fields needed by candidate policies, and transform only for selected output formats. An event filtered by all receivers should incur no RETRANS payload persistence.

## Two-stage selection

1. **Cheap identification:** Read available source metadata (for example Windows channel/provider/Event ID/version, or syslog facility/severity/hostname/app/message identifier), plus trusted collection provenance.
2. **Candidate union:** Determine every receiver whose allowlist could possibly match; do not short-circuit on the first receiver.
3. **Selective extraction:** Extract additional fields only if candidate rules require them; cache each extracted field within that event's processing context so multiple receivers do not trigger redundant parsing.
4. **Final rule evaluation:** Independently decide each receiver's selection. Preserve `MATCH`, `NO_MATCH`, and `UNRESOLVED` as different outcomes.
5. **Selective transformation:** Serialize or map only for selected output adapters. Transformation failure for one receiver must not automatically invalidate an unrelated output format for another.

## Policy behavior

Receiver policies are local, explicit allowlists: no allow rule matches -> filtered; allow and exclusion both match -> excluded for *that receiver*; missing or invalid required fields -> unresolved, processed according to a separately specified error policy. Do not silently interpret a parser failure as an ordinary no-match.

A policy rule can initially use typed equality, set membership, numeric comparisons, field presence and bounded string operations. Arbitrary scripting and unrestricted regex matching are not part of the initial agreed scope. Exact rule language is open.

## Source attribution

For WEC, the **collection host/channel** and the **original computer/provider/channel** are separate concepts. For syslog, the actual network peer is different from an asserted `HOSTNAME`. RETRANS must not silently merge or fabricate identities.

## Normalization and transformation

Keep a compact transient event envelope with timestamps, provenance, source/type identity, policy revision, and a reference to the original representation while needed. The universal envelope must not require every adapter to populate an enormous schema. Parse source-specific fields lazily. Selected receivers may need different output formats; share extracted fields but create separate output representations only as necessary.

Preserve source timestamps when present; preserve RETRANS reception time separately. Unknown, absent and malformed values must not be converted into invented defaults.

## Policy snapshots

Each event uses one immutable policy revision through its processing and routing decision. New revisions apply only at the declared cutover. Queued events cannot silently gain or lose obligations because an allowlist changed; if further encoding is deferred, retain sufficient mapping and policy version material to reproduce the required representation.

## Windows filtering

Use native event querying conservatively and benchmark it against Go-side filtering. Native selectors must not discard event types needed by any receiver. A subscription query change requires controlled cursor/overlap logic; native filtering's unseen events cannot be counted as individually processed.

## Validation examples

- A accepts 15 event IDs; B accepts those plus DNS/DHCP: A must not limit B's source events.
- No receiver wants an event: avoid full rendering, extra normalization and payload disk writes.
- A and B both need `TargetUserName`: extract once, evaluate separately.
- Output A is syslog and B is structured JSON: encode only required representations.
- One field cannot be parsed: record unresolved/error outcome, not false filtering.