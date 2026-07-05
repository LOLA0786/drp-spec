# DRP — Decision Runtime Protocol

An open wire format for tamper-evident, independently verifiable
AI-agent decision audit trails: what was requested, what was decided,
why, what actually executed, and proof that none of it was rewritten.

## The three questions

Agent runtimes converge on three questions, in order:

1. **Can this action ever be executed?** (deterministic authorization —
   resources, budgets, policy)
2. **Is this still the same trusted agent?** (behavioral identity —
   invariants, grants, drift)
3. **Can this decision be proven later?** (evidence — sealed records,
   hash chains, signatures, independent verification)

DRP standardizes the objects those questions produce, so any
enforcement runtime can emit them and any auditor can verify them.

## Record kinds

**DecisionRecord** — one per pre-execution authorization decision.
Sealed (SHA-256 over canonical JSON, sorted keys, compact separators),
per-agent hash-chained (genesis 64 zeros), chain link inside the
digest. Raw action arguments never stored — digest only.

**ExecutionEvent** — at most one per decision, reporting what actually
happened. Hash-anchored: its prev_hash is the decision's record_hash.

Records carry a producer-gated `edges` list (`follows`,
`resulted_in`; `signed_as` upon signing). Fields without a producing
component are null by policy, never fabricated.

## What verification proves

| Attack on the log                      | Caught by              |
|----------------------------------------|------------------------|
| Edit any field of any record           | record_hash mismatch   |
| Delete or reorder records              | per-agent chain break  |
| Forge an execution result              | anchor mismatch        |
| Runtime says BLOCK, action ran anyway  | ENFORCEMENT DIVERGENCE |

`tools/verify_records.py` is standard-library-only Python and does not
import any producing runtime — verification requires zero trust in the
code being audited.

## Test vectors

`test-vectors/` pins four canonical files: `clean` (MUST pass),
`tampered_field`, `deleted_record`, `divergent` (MUST fail, each for
its stated reason). A conformant implementation produces files the
verifier passes, and cannot produce tampering these checks miss.

## Signing

Ed25519 over `record_hash`, chain link inside the digest — see
`docs/SIGNING.md` for the normative rules and the attack class they
close.

## Status

drp/0.1. Produced by multiple independent runtimes (behavioral
identity engine, enterprise action-authorization gateway, evidence/
signing layer) that converged on hash-chained decision logs before
this format unified them.

## License

Code (verifier, generators): Apache-2.0. Specification text and
schemas: CC-BY-4.0.
