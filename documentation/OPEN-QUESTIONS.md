# RETRANS Open Engineering Questions

Items listed here are **not resolved** by the current design review. They must not be presented as supported runtime functionality.

## Durability and delivery

1. Exact RAM queue admission, retry and RAM-to-disk handoff ordering; how in-flight records are handled when transport fails.
2. Which source/receiver combinations may opt into durable-before-send mode, and what its acknowledgment boundary is.
3. Physical spool representation, integrity checks, segment recovery, commit synchronization and compact/reclamation strategy.
4. Cross-receiver storage accounting and how normal/burst/outage budgets interact with the separately bounded UDP retention pool without charging a shared payload more than once physically.
5. What happens when a strict receiver is full while a healthy receiver still wants the same event; how to prevent source-wide head-of-line blocking where possible.
6. Which receiver protocols provide true application-level durable acknowledgment versus only transport-write completion.
7. Whether any outbound plaintext UDP syslog support will exist as an explicitly authorized exception to encrypted-outbound policy; current default prohibits plaintext.
8. Exact loop/replay identity and investigation behavior when original event provenance is incomplete.
9. Source-to-receiver ordering policy and permitted overtaking during backlog recovery.

## Windows / operations

10. Windows service identity, privileges, Windows Event Log access, service SID, ACLs, logon rights and credential handling.
11. Windows Certificate Store integration versus protected external key material; revocation, rotation and failure behavior.
12. Native Windows API selection and bookmark handoff after changing a subscription query.
13. Installer, upgrades, configuration migration, rollback, and recovery of spool state.
14. Global volume capacity enforcement, reserve guarantees, multi-volume support and disk failures.

## Configuration / validation

15. Precise JSON schema, versioning, per-file reference model, and portability requirements for the Go compiler and standard-library JSON implementation.
16. Exact operator vocabulary for policy rules and parse-error quarantine/rejection behavior.
17. Default worker counts, bounds, resource quotas, event size ceilings, retry windows, outage qualification and recovery rates.
18. Which receiver adapters and formats are included in the first release.
19. Numerical load, memory, CPU, storage latency and recovery-time acceptance targets on representative Windows Server hardware.

## Review guideline

No open item should acquire a fabricated default. Options and illustrative values are recorded as examples until expressly decided.