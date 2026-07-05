# DRP <-> AARM Crosswalk

How the Decision Runtime Protocol (DRP) record formats and verification
guarantees map onto the Autonomous Action Runtime Management (AARM)
specification (Errico, arXiv:2602.09433; CSAI Foundation, April 2026).

## Positioning

AARM is a system specification: it defines WHAT a runtime security
system for AI-driven actions must do. DRP is a wire format: it defines
the OBJECTS such a system emits and how anyone can verify them without
trusting the emitter. They are complementary — AARM-conformant systems
need concrete, interoperable object formats for receipts and forensic
reconstruction; DRP is a candidate for that layer.

## Component mapping

| AARM concern | DRP object / mechanism |
|---|---|
| Action interception & normalization to canonical schema | AgentAction / ActionRequest -> DecisionRecord with `capability`, `arguments_digest` (raw args never stored), optional `request_id` linkage |
| Policy & intent evaluation | `decision` (allow / require_approval / block) + `triggered_by` attributing WHICH layer decided (deterministic constraint, behavioral invariant, authorization, drift) |
| Action classification (forbidden / context-dependent deny / context-dependent allow) | Precedence levels: deterministic constraints and invariants -> BLOCK (forbidden); authorization and drift -> REQUIRE_APPROVAL (context-dependent); baseline -> ALLOW |
| Enforcement | HTTP status as signal in the reference implementation (200/202/403); `outcome: pending` const makes decision immutability machine-checkable |
| Tamper-evident receipts | Sealed records: SHA-256 over canonical JSON, per-agent hash chain, chain link INSIDE the digest; optional Ed25519 envelope over record_hash (docs/SIGNING.md) |
| Forensic reconstruction | Decision lineage (follows edges), execution anchoring (resulted_in edges), full-graph reload from the audit file alone |
| Session context accumulation | Per-agent chains; behavioral baseline advances only on ALLOW (denied probes cannot walk the baseline) |

## Threat model mapping

AARM formalizes prompt injection, confused deputy, data exfiltration,
and intent drift. DRP's verification guarantees address the AUDIT
side of each: whatever the attack, the log cannot be silently
rewritten to hide it. Four canonical test vectors pin the claims:

| Attack on the audit trail | Vector | Verifier check |
|---|---|---|
| Falsify a verdict | tampered_field | record_hash mismatch |
| Erase a refusal | deleted_record | per-agent chain break + orphaned execution |
| Forge/reorder history | (signing) | signature binds original hash, which binds prev_hash |
| Runtime lies about refusing | divergent | ENFORCEMENT DIVERGENCE: block + ok execution fails audit |

The divergence check is, to our knowledge, not present in comparable
receipt schemes: a runtime that CLAIMS to have blocked an action that
the executor reports as run is caught from the audit file alone, by a
standard-library-only verifier with zero trust in the runtime.

## Conformance offer

We propose DRP's schemas, test vectors, and independent verifier as
candidate conformance material for AARM's receipt and forensic
requirements. Three independently built runtimes (a behavioral
identity engine, an enterprise action-authorization gateway, and an
evidence/signing layer) converged on hash-chained decision logs before
DRP unified them — evidence that the format captures a real, shared
need rather than one vendor's internals.

Contact: PrivateVault AI (Pentaprime Solutions) — AARM WG contribution.
