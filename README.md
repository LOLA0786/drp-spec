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

DRP standardizes the objects the third question produces, so any
enforcement runtime can emit them and any auditor can verify them.

## Record kinds

**DecisionRecord** — one per pre-execution authorization decision.
Sealed with SHA-256 over canonical JSON (see
[docs/CANONICAL-JSON.md](docs/CANONICAL-JSON.md) — the rules are exact
and three of them differ from common library defaults). Per-agent hash
chain; the chain link is inside the digest, so a signature over
`record_hash` covers history. Raw action arguments are never stored —
digest only.

**ExecutionEvent** — at most one per decision, reporting what actually
happened. Hash-anchored: its `prev_hash` is the decision's
`record_hash`.

Records carry a producer-gated `edges` list (`follows`, `resulted_in`;
`signed_as` upon signing). Fields without a producing component are
`null` by policy, never fabricated.

## Chain origin and external anchors

An agent's first record establishes its origin. `prev_hash` of 64 zeros
is an unanchored genesis; any other value is an **external provenance
anchor** binding the chain to state outside the file, typically the head
of a prior export. The file proves continuity *from* the origin; it
cannot prove the origin's legitimacy — that is an out-of-band
attestation. Conforming verifiers accept anchored first records and MUST
report which agents are anchored.

## What verification proves

| Attack on the log | Caught by |
|---|---|
| Edit any field of any record | `record_hash` mismatch |
| Strip fields and re-hash | exact field-set check |
| Delete or reorder records | per-agent chain break |
| Fabricate a parent decision | lineage check (parent must exist earlier, same agent, be the immediate prior record) |
| Duplicate a decision id | uniqueness check |
| Report an outcome on another agent's decision | execution/decision agent binding |
| Forge an execution result | anchor mismatch |
| Two executions for one decision | one-execution-per-decision check |
| Runtime says BLOCK, action ran anyway | ENFORCEMENT DIVERGENCE |

`tools/verify_records.py` is standard-library-only Python and imports
no producing runtime — verification requires zero trust in the code
being audited.

## Test vectors

`test-vectors/` pins four canonical files: `clean` (MUST pass),
`tampered_field`, `deleted_record`, `divergent` (MUST fail, each for
its stated reason). `clean` deliberately includes the unicode and float
cases that silently fork naive serializers. A conformant implementation
produces files the verifier passes, and cannot produce tampering these
checks miss.

## Precedence contract

`contracts/precedence-order.json` pins the order in which a conforming
runtime evaluates decision levels, and which levels may block versus
escalate. It is hash-pinned: the reference implementation's CI fails if
the code and the contract disagree in either direction.

## Signing

Ed25519 over `record_hash`, chain link inside the digest — see
[docs/SIGNING.md](docs/SIGNING.md) for the normative rules and the
attack class they close.

## Status

`drp/0.1.1`. The format was extracted after three of my own runtime
prototypes independently converged on hash-chained decision logs. The
reference implementation is **PrivateVault Agent DNA** (commercial;
access on request).

There are no third-party implementations yet. If you build one, the
test vectors are the conformance bar and I would like to hear about it —
particularly if you find a divergence the vectors do not catch.

## License

Code (verifier, generators): Apache-2.0.
Specification text and schemas: CC-BY-4.0.

---

Chandan Galani · [privatevault.ai](https://privatevault.ai) ·
[github.com/LOLA0786](https://github.com/LOLA0786)
