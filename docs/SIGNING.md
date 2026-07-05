# DRP Signing — Normative Rules

Signing is a layer ABOVE the record format. Records are complete and
verifiable without signatures; signatures add non-repudiation.

## Rule 1 — Sign the record_hash, never a re-canonicalized body

A signature is Ed25519 over the ASCII bytes of the record's
`record_hash` (the 64-char lowercase hex string). Implementations MUST
NOT re-serialize the record to produce signing input. Rationale: two
runtimes canonicalizing "the same" JSON with different separators or
type coercion produce different digests; signing the already-sealed
record_hash makes the signature independent of any second
serialization path.

## Rule 2 — The chain link is inside the digest

`prev_hash` (decisions) and the decision-anchored `prev_hash`
(execution events) are part of the hashed payload. A conformant
verifier MUST reject any scheme that excludes chain-link fields from
the signed digest. Rationale: a signature that covers a record but not
its position in history permits silent re-ordering and re-chaining of
individually-valid records.

## Rule 3 — Signature envelope

A signature is carried in a detached envelope, never inside the
hashed payload:

    {
      "algorithm": "Ed25519",
      "signed_hash": "<record_hash of the signed record>",
      "signature": "<hex>",
      "public_key": "<hex>",
      "key_id": "<optional key identifier>"
    }

The signed record references its envelope via `receipt_ref` and a
`signed_as` edge. Envelope creation is the producer that activates
those fields; until a signer runs, they are null/absent per the
producer-gated field policy.

## Rule 4 — Verification order

1. Verify the record (hash recomputation, chain position) WITHOUT the
   signature.
2. Verify the signature over record_hash.
A signature over a record that fails step 1 is meaningless and MUST
be reported as a record failure, not a signature failure.

## Legacy formats

Pre-DRP receipt formats (line-concatenation chains, digests excluding
chain links, coercing canonicalizations) are non-conformant. Existing
ledgers in such formats should be marked with their legacy version
identifier and verified by their original tools; new records MUST be
drp/0.1.
