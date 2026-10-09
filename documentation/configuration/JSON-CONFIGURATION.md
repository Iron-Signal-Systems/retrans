# JSON Configuration and Transactional Reload

**Status:** JSON selected; exact JSON field schema not finalized.

## Decisions

- Configuration is human-editable **JSON**, parsed with the Go standard library; no TOML/YAML parser dependency required.
- Keep human-edited configuration separate from event payloads, spool state, bookmarks, metrics and receiver progress.
- Keep one authoritative local configuration revision, logically divided into service, source groups, receivers and policy definitions.
- Validate the complete candidate, compile immutable receiver-policy lookup structures, and activate only after required resources can be prepared safely.
- Unknown settings, duplicates, malformed values, invalid cross-references and unsupported schema versions must fail validation; do not silently omit broken rules.

## Proposed file layout (not binding schema)

```text
config/
  retrans.json
  sources/
    windows.json
    wec.json
    syslog-udp.json
    syslog-tcp.json
    syslog-tls.json
  receivers/
    receiver-a.json
  policies/
    receiver-a-windows.json
```

The file names are examples, not required defaults. Administrator configuration must not directly contain raw private keys or password material in cleartext without an explicitly approved credential-handling mechanism.

## Policy compilation

At configuration activation, validate every rule and compile efficient in-memory structures. Processing workers use immutable snapshots. An event stays on a coherent policy revision; no record may be evaluated partly under one receiver list and partly under another.

Receiver policy semantics: allowlist per receiver, exclusion overrides selection for that receiver, parsing/field absence yields explicit unresolved handling, and no receiver gets to modify RETRANS policy remotely by default.

## Change classes

| Change | Proposed activation |
| --- | --- |
| Allowlist, exclusions, field rules | Immutable live snapshot cutover |
| Retry and non-structural operational values | Live update with bounds |
| Worker count, RAM queue size | Controlled source-group resize |
| Listener address or port | Controlled listener transition |
| WEC/native Windows subscription selector | Bookmark-aware controlled resubscription |
| Certificate or receiver identity | Security-validated handoff; queued obligations retained |
| Persistent spool path or format | Explicit maintenance/migration workflow |

A source-group transition can have its own completion boundary. Do not call a multi-resource reload atomically complete merely because one policy pointer changed.

## Configuration safety

Invalid candidates leave the active configuration intact. Only an authenticated and authorized local administrator can request configuration changes. Receiver deletion, strict-backlog abandonment and unsafe quota reduction require explicit validation and disposition. Last-known-good startup behavior must not override security revocation or an intentional disable action.

## Proposed CLI operations

`validate`, `diff`, `test-policy`, `apply`, `status`, `sources`, `receivers`, `backlog`, `history`, and `rollback`. These names are conceptual, not implemented command syntax.

## Open items

Define concrete JSON schema/version, multiple-file include/reference rules, secret-store integration, policy test input schema, and exact activation transaction/rollback semantics. The minimum Go version and standard-library JSON API are not yet committed.