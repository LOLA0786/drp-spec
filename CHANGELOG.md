## drp/0.1 — test-vector regeneration (2026-07-21)

The four canonical test vectors were regenerated and re-pinned. Cause:
a conformance test in the reference runtime invoked the vector
generator against the canonical directory on every run, silently
rewriting the vectors; one regenerated set had propagated before the
bug was found. The producing bug is fixed and regression-locked in
the reference runtime (generator writes to the canonical directory
only on deliberate invocation; committed vectors are SHA-256-pinned
by a test). The wire format, record semantics, and verifier are
UNCHANGED — only the concrete example records (UUIDs, timestamps,
derived hashes) differ. `test-vectors/SHA256SUMS` now pins the
canonical generation; implementations verifying against the old
vector bytes should refresh from this commit.

## drp/0.1 — verifier strict mode (2026-07-21)

`tools/verify_records.py` is upgraded from 6 to 9 check families,
synced from the reference runtime where these checks were developed
and test-hardened. New: exact per-kind field-set enforcement (unknown
AND missing fields fail — closes the strip-and-rehash attack, where a
record is reduced to a subset of fields and resealed into an
internally consistent but non-conformant file), file-wide
decision_id/event_id uniqueness, 64-lowercase-hex validation of all
hashes, the origin/anchor rule (first record per agent: 64 zeros =
genesis, anything else = reported external provenance anchor), full
lineage binding (parent_decision must be the immediate prior decision
of the same agent, matched by prev_hash, with exactly one "follows"
edge), enum validation of decision and execution status, execution
agent_id must match its decision, and strict JSON (NaN/Infinity
rejected, per docs/CANONICAL-JSON.md).

This is a tightening: files that passed the 6-check verifier may fail
strict mode if they carry extra/missing fields or malformed
references. drp/0.1 has no known external adopters, so the
tightening lands in-place rather than as a version bump; from the
next change onward, verifier semantics changes will version. The
four canonical vectors pass strict mode; SHA256SUMS unchanged.
