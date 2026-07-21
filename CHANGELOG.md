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
