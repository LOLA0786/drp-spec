# Agentic Trust Controls — Mapping (DRP Specification)

Self-assessment of the Decision Runtime Protocol (DRP) — this
public, open specification — against Vanta's Agentic Trust Controls
(Developer baseline). Every claim below is independently verifiable
by anyone who clones this repository alone, with no access to any
private runtime that implements DRP. That is the entire point of a
separate mapping for the spec: a runtime's mapping can cite its own
private tests; a spec's mapping can only cite what's actually public.

**Scope note:** most Agentic Trust Controls describe *runtime
behavior* (tool scoping, drift detection, multi-agent voting,
sandboxing) that a wire-format specification does not implement by
design — those controls are the responsibility of whatever runtime
conforms to DRP, not the spec itself. This document marks such
controls **OUT OF SCOPE FOR A SPEC**, not FAIL, and says so plainly
rather than either claiming them or hiding the distinction.

Status legend: **PASS** (verifiable today, by anyone, from this
repo alone) · **PARTIAL** · **OUT OF SCOPE FOR A SPEC** · **FAIL**.

---

## RBM — Runtime Behavioral Monitoring

The domain a decision-audit protocol most directly addresses.

| Control | Status | Evidence (public, in this repo) |
|---|---|---|
| RBM-03 Tamper-evident action logging | **PASS** | `schemas/decision_record.schema.json` + `schemas/execution_event.schema.json` define the sealed, hash-chained record format. `tools/verify_records.py` (standard-library-only, zero dependency on any producing runtime) independently checks it. Run it yourself: `python3 tools/verify_records.py test-vectors/tampered_field.jsonl` returns a record-hash-mismatch failure; `test-vectors/clean.jsonl` passes. |
| RBM-01 Behavioral telemetry generation | **PARTIAL** | The schema defines what a decision record *contains* (capability, decision, reason, evidence); whether a given runtime actually *generates* this telemetry in operation is a runtime property, not something the spec alone proves |
| RBM-04 Instrumentation for external enforcement | **OUT OF SCOPE FOR A SPEC** | This describes a runtime exposing an action stream; DRP defines the record format such a stream would use, not the stream itself |
| RBM-02 Behavioral drift detection | **OUT OF SCOPE FOR A SPEC** | Drift detection is a runtime capability; DRP has no field or mechanism for it |

## AID — Agent Identity & Authority

| Control | Status | Evidence |
|---|---|---|
| AID-01 Verifiable agent identity | **PASS** | `schemas/decision_record.schema.json` requires `agent_id` on every record, and the per-agent hash chain (`prev_hash` continuity, `GENESIS_HASH` anchor documented in `README.md`) structurally binds every record to a specific agent's history |
| AID-05 Authority attestation at execution | **PARTIAL** | The record format captures capability, decision, reason, and (per `docs/SIGNING.md`) an optional Ed25519 signature binding the record's origin — this is real attestation of *what was decided and by whom the record was sealed*, but the spec does not define a "granted authority" field distinct from the decision itself; that's a runtime-specific extension |
| AID-02, AID-03, AID-04 | **OUT OF SCOPE FOR A SPEC** | Credential rotation, tool scoping, and JIT privilege are runtime/deployment behaviors with no corresponding field or requirement in the record format |

## ADV — Adversarial Robustness & Testing

| Control | Status | Evidence |
|---|---|---|
| ADV-03 Re-test on system change | **PASS** | `test-vectors/` (`clean.jsonl`, `tampered_field.jsonl`, `deleted_record.jsonl`, `divergent.jsonl`) are canonical, pinned vectors any implementation must produce/reject correctly. Any change to a conforming implementation can be re-tested against these same four vectors, giving a stable regression baseline independent of the implementation's own internals |
| ADV-01 Adversarial red-teaming | **PARTIAL** | The four test vectors are a *deliberate, named set of adversarial cases* against the record format (tampering, deletion, forged execution, enforcement divergence), but this is a fixed, small set — not an open-ended red-teaming exercise, and it tests the format, not a full runtime's behavior |

## SCP — Supply Chain & Component Provenance

| Control | Status | Evidence |
|---|---|---|
| SCP-01 Model and tool provenance verification (adapted: *record* provenance) | **PASS** | Per `docs/SIGNING.md`, a signed record's Ed25519 signature covers the record hash directly, and the chain link is inside that hash — a signed record cannot be silently re-chained even if re-sealed validly, which is exactly a provenance-verification property, applied to decision records rather than models/tools |

## MAS — Multi-Agent Systems & Delegation

| Control | Status | Evidence |
|---|---|---|
| MAS-01, MAS-02 Inter-agent authentication / communication integrity | **OUT OF SCOPE FOR A SPEC** | DRP's record format has no multi-agent voting or inter-agent messaging construct; that is a runtime-layer capability, not part of this specification |

## Everything else (TUE, RII, MEM, HOA, GOV, OUT, RES)

**OUT OF SCOPE FOR A SPEC.** These domains describe tool execution,
sandboxing, memory systems, human oversight workflows, governance
processes, output generation, and resource limiting — all runtime or
organizational behaviors. DRP is a record format and a verification
tool; it does not execute, sandbox, remember, oversee, govern,
generate, or rate-limit anything. A runtime that conforms to DRP is
responsible for these controls, and should be assessed separately,
on its own merits, with its own evidence.

---

## Summary

**What this spec genuinely proves, checkable by anyone, right now:**
tamper-evident, hash-chained decision records; standard-library-only
independent verification requiring zero trust in any producing
runtime; a fixed, canonical adversarial test-vector set for
regression testing across implementations; optional Ed25519 signing
with chain-link-inside-the-digest, making history rewriting
detectable even when a record is validly re-sealed.

**What it deliberately does not claim:** anything about tool
execution, agent behavior, drift, delegation, or oversight — those
are runtime properties. A runtime implementing DRP should publish its
own, separate Agentic Trust Controls mapping, citing its own
(private or public) evidence, rather than inheriting claims from this
specification.

Try it yourself:
git clone https://github.com/LOLA0786/drp-spec
cd drp-spec
python3 tools/verify_records.py test-vectors/clean.jsonl        # VERDICT: PASS
python3 tools/verify_records.py test-vectors/tampered_field.jsonl  # VERDICT: FAIL
