# Security Review: teslatlas-edge

## Scope

Offline Standard scan of Teslatlas Edge current HEAD.

- Scan mode: repository
- Target kind: git_revision
- Target ID: target_sha256_a08bec564be5822bdde787c9842a3b7a491734c08c4ffd5701f5a4213a0cc727
- Revision: 9aa14d97d0e13a44064d60ec07beaf1a5661f81e
- Inventory strategy: repository
- Included paths: .
- Excluded paths: none
- Artifacts reviewed: src/admission.rs, src/delivery.rs, src/spool.rs, src/crypto.rs, src/credentials.rs, src/config.rs, src/runtime.rs, src/tls.rs, src/protocol.rs, packaging, docs/architecture

Limitations and exclusions:
- No builds, sidecar, Hub/vehicle network, private lab, deployed container or secret access.
- Large repository and packaging reviewed selectively; completeness partial.

### Scan Summary

| Field | Value |
| --- | --- |
| Scan outcome | completed |
| Reportable findings | 0 |
| Severity mix | none |
| Confidence mix | none |
| Coverage | partial |
| Validation mode | static_source_trace |

Canonical artifacts: `scan-manifest.json`, `findings.json`, and `coverage.json`. This report is a deterministic projection of those files.

## Threat Model

Teslatlas Edge is optional user-operated Fleet Telemetry ingress: a pinned receiver sidecar posts strict bounded JSON over loopback; Edge durably stores encrypted bounded records; a home Hub pulls and acknowledges batches over mTLS and bearer.

### Assets

- Telemetry envelopes containing VIN/location and private history
- Receiver bearer, Hub bearer, mTLS identities and spool encryption key
- Pending records, monotonic sequence, gap notices and acknowledgement receipts
- Operational readiness without silent data loss

### Trust Boundaries

- Vehicle mTLS to pinned sidecar, then loopback bearer JSON to Edge
- Unauthenticated local health/metrics versus receiver admission
- Hub mTLS client certificate and bearer to pull/ack API
- Encrypted on-disk spool and key across restart/migration/backup
- Configuration/installer paths to listener binding and secret files

### Attacker Capabilities

- A local process could connect to loopback but needs receiver bearer
- Network client may reach Hub listener if exposed but needs client cert and bearer
- Compromised sidecar/source may send malformed, repeated or large JSON within transport limits
- Actor with local filesystem access may tamper spool/key/config subject to OS permissions

### Security Objectives

- Reject unauthenticated admissions and Hub pulls/acks
- Bound request, spool, receipt and gap resource use
- Encrypt telemetry at rest and avoid logging private fields
- Preserve durable at-least-once identity/sequence semantics
- Fail closed on corrupt or mismatched credential/storage state

### Assumptions

- Offline read-only static review; no receiver, Hub, vehicle, network, build, installation or secret access.
- Deployment isolation differs between Linux service and per-user macOS LaunchAgent.
- No account credentials or vehicle-command authority in Edge source.

## Findings

### No findings

No reportable findings survived the canonical discovery, validation, and reportability gates.

## Reviewed Surfaces

| Surface | Risk Area | Outcome | Notes |
| --- | --- | --- | --- |
| Packaging and operational boundaries | Packaging and operational boundaries | Needs follow-up | Selected Linux, Docker and macOS packaging sampled; deployed sidecar and full installer matrix unverified. |
| Receiver admission | Receiver admission | No issue found | Loopback bind, constant-time bearer, strict 256 KiB JSON and concurrency cap reviewed. |
| Hub delivery and acknowledgements | Hub delivery and acknowledgements | No issue found | mTLS client CA plus independent bearer; v2 ACK identity/prefix and receipt-before-delete reviewed. |
| Encrypted spool and recovery | Encrypted spool and recovery | No issue found | XChaCha20-Poly1305, count/byte bounds, lock, sequence and fail-closed recovery sampled. |
| TLS, config and secret custody | TLS, config and secret custody | No issue found | Private files and intended separate mounts reviewed; read-before-cap requires trusted configured path. |

## Open Questions And Follow Up

- Architecture prose still describes spool FORMAT=3; runtime and upgrade guard use FORMAT=4. This is maintenance documentation, not a validated security bypass.
- If operator config points secrets/state into attacker-writable ancestors, check-then-path-read and chmod patterns deserve hardening; default documented private storage does not establish this prerequisite.
- Pinned receiver binary, actual deployment filesystem permissions, installer and matrix behavior were not executed/fully reviewed.
  - Follow-up prompt: Review deferred unit edge-deployment-followup and close its stated proof gap. Paths: tools, packaging, docs/operations.
