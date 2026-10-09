# RETRANS Agent and Contributor Rules

## Purpose and source of authority

This file governs coding agents, contributors, review agents, and automation working in the RETRANS repository. It is an engineering and change-control contract, not a claim that the proposed runtime is implemented.

Read `README.md`, `documentation/README.md`, `documentation/DECISIONS.md`, `documentation/ROADMAP.md`, and `documentation/OPEN-QUESTIONS.md` before a material change. Read the applicable architecture, reliability, configuration, operations, and acceptance documents before changing a governed behavior. If documents and code disagree, explicitly identify and resolve the disagreement; do not silently select the easier interpretation. Distinguish **Agreed**, **Working proposal**, and **Open** design statements.

## Product boundaries

RETRANS is a lightweight Go event collection, selective processing, and forwarding service; initially a Windows Server service. It is not a SIEM, threat-detection engine, correlation platform, or long-term archive. It must not require a hosted ISS service, a third-party SIEM, ELK, Redis, Kafka, PostgreSQL, or an external database merely to operate.

Customer names, government organizations, public-safety deployments, and non-public infrastructure details must not appear in public repository documentation, examples, fixtures, or logs.

## Architectural invariants

- Independently supervised Source Processing Groups are required for native Windows Event Logs, WEC/WEF, syslog UDP, TCP, and TLS as applicable. Each has its own bounded queue, configured worker capacity, overload behavior, and health. More than one worker per group is permitted. Shared code does not imply a shared processing FIFO.
- Memory-first operation is the normal path: identify, filter, normalize selectively, and forward from bounded memory without routine event-payload disk writes. Persist payloads only when congestion, outage, retention risk, or an explicitly stronger durability policy requires it.
- All potentially interested receivers must be considered before an event is globally filtered. Receiver allowlists, exclusions, output transforms, connections, and delivery progress are independent.
- Filtering is staged. Avoid full event rendering, repeated parsing, unnecessary allocations, and preparing output encodings for destinations not selected by policy.
- Process each event under one immutable compiled policy revision. Do not silently change committed receiver obligations on reload.
- Preserve collection host and original source identity independently, especially for forwarded Windows events. Distinguish reported source timestamps from local receipt time.
- Native Windows/WEC event checkpoints must reflect explicitly defined processing/delivery boundaries and contiguous completed progress. A bookmark is not a guarantee that a source channel will retain the event indefinitely.
- Ordinary UDP syslog lacks delivery acknowledgments and source replay. An OS send success is not remote ingest confirmation. TCP/TLS transport success is also not necessarily durable application receipt. Never claim exactly-once or confirmed ingestion without real protocol support.
- UDP-originated outage retention is bounded by **both** maximum age and pool size. Administrator-authorized oldest-pending-first eviction/expiration is an accounted loss, not success; sample capacities and periods are not defaults.
- Normal, burst, and protected outage capacity settings are administrator-controlled. Ordinary bursts must not silently use outage-only reserves. Strict receiver obligations must not be silently overwritten when full.
- No receiving platform or source has authority to rewrite local configuration merely by being connected.

## Configuration and state

Configuration is **JSON**, using Go standard-library parsing with strict validation. Keep service, source, receiver, and policy definitions independently organized but activated as one coherent revision. Reject duplicate object members, unknown fields, invalid values, unsupported schema versions, inconsistent references, and unsafe resource reductions. Compile policies into immutable in-memory structures; never parse configuration per incoming event.

Runtime spool data, checkpoints, receiver obligations, and diagnostic state must not be stored as administrator-edited configuration files. The physical persistent format, recovery mechanics, security privilege model, and numerical performance thresholds remain open until specifically approved; agents must not invent defaults or treat proposals as final.

Invalid candidates must leave the current operational configuration intact. Changes requiring Windows subscription handoff, socket binding, quota migration, or TLS identity changes need controlled activation and explicit failure handling.

## Go and Windows coding rules

Use Go for production code unless a governing design explicitly approves another implementation. Prefer standard-library facilities and small, justified dependencies. Use `switch` for discrete states and `if` for guards, ranges, and compound conditions. Within coherent sections, keep functions and types alphabetized where it does not obscure lifecycle ordering. Use meaningful section comments, descriptive names, and explicit error context; do not silently discard errors or convert unknown into an invented value.

Implement Windows service and Event Log semantics through stable Windows APIs and carefully bounded wrappers. Do not make `powershell.exe`, `sc.exe`, `wevtutil`, or a shell command a production runtime dependency when a native API is the correct interface. Administrative scripts and test harnesses may use them explicitly.

Keep platform-specific Go source explicitly constrained with the appropriate `//go:build windows` (or future supported platform) build tag. Portable contracts may remain shared only when genuinely portable. Cross-compilation does not establish runtime support. FreeBSD is only a future consideration until approved, implemented, and tested.

## Security and operation

Encrypted outbound transport, receiver identity validation, supported external trust material, and no silent plaintext fallback are required. The existence of an incoming UDP syslog adapter does not authorize plaintext outbound UDP as an exception.

Use least privilege and avoid unnecessary Windows administrative rights. Do not add generic command execution, undocumented remote support access, hard-coded credentials, private keys in source control, unbounded network listeners, or insecure certificate bypasses. Do not log secret material or entire sensitive events simply for debug convenience. An unhealthy receiver must not prevent unrelated healthy receiver workers from making progress, subject to finite shared resources and explicit source admission policies.

Make source admission drops, parser failures, unresolved rules, spool evictions, age expirations, source rollover, certificate failures, and stalled delivery independently observable. Later recovery must not erase historical failure information. A running service or connected socket alone is not proof of useful delivery.

## Tests and documentation

Every material implementation change requires appropriate regression tests, error-path tests, and platform-representative validation. For applicable Go changes run `gofmt`, `go test ./...`, `go vet ./...`, and `git diff --check`; use supported Windows hosts for Windows runtime behavior. Crash consistency, disk exhaustion, rapid policy reload, UDP floods, WEC rollover, multiple receivers, and backlog recovery require targeted failure tests before product guarantees are claimed.

Do not invent passing tests, performance figures, platform support, or release readiness. Preserve failed test results alongside later successful reruns. Update `documentation/DECISIONS.md` when a design choice is approved or superseded, `documentation/OPEN-QUESTIONS.md` when an open issue is resolved, the affected specification, and the relevant acceptance criteria. Keep the root README a concise public overview; detailed contracts belong under `documentation/`.

## Repository hygiene and GitHub change control

Never commit private keys, credentials, local log data, spool contents, machine-specific state, unrelated experimental artifacts, or generated large test fixtures. Keep diffs narrow, inspect staged changes, and avoid unrelated refactors during fixes.

**Do not create branches, commit, push, open or merge PRs, change rulesets, alter settings, delete repository content, or create releases without explicit approval for the requested action.** Approval for one action is not blanket permission for later actions. Do not equate a request to review or prepare files with permission to push. An authorized documentation push does not authorize unrelated implementation changes.

## Final review questions

Before declaring work ready: Did we change source identity, a checkpoint, receiver selection, a durability or loss guarantee, a source/receiver resource boundary, a trust boundary, a supported platform, or an operational failure interpretation? Did we validate the correct source type and host? Are unknown outcomes reported as unknown? Did we avoid unnecessary disk I/O and parsing? Are docs, tests, and implementation in agreement? Was the exact repository action authorized?

RETRANS should remain understandable to an administrator diagnosing a forwarding failure: what was observed, what was selected, what was transmitted, what is still pending, and what is not known.