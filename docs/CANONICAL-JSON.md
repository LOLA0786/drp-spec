# Canonical JSON — the hash preimage

`record_hash` is SHA-256 over the canonical JSON serialization of the
record payload (every field except `record_hash`). Two conforming
implementations MUST produce byte-identical serializations for the same
record, or their chains fork silently: every record still verifies
locally, and no two runtimes agree.

The reference serialization is CPython's

```python
json.dumps(payload, sort_keys=True, separators=(",", ":"), allow_nan=False)
```

Below are the rules that produce it, stated normatively, because three
of them differ from the defaults of common JSON libraries.

## 1. Key order

Object keys sort by Unicode code point, ascending, at every level.

## 2. Separators

`,` between items, `:` between key and value. No whitespace anywhere.

## 3. Strings — ASCII-only escaping (`ensure_ascii`)

Every character above `U+007E` MUST be escaped as `\uXXXX`, lowercase
hex. Characters outside the BMP MUST be encoded as a UTF-16 surrogate
pair. Control characters below `U+0020` use the short forms where they
exist (`\b \f \n \r \t`) and `\u00XX` otherwise. `U+007F` (DEL) is
escaped.

| input | canonical output |
|---|---|
| `é` | `"\u00e9"` |
| `😀` (U+1F600) | `"\ud83d\ude00"` |
| `U+007F` | `"\u007f"` |
| `U+0001` | `"\u0001"` |

Most JSON libraries emit raw UTF-8 here. That is a fork.

## 4. Numbers — integers stay integers, floats use CPython `repr`

Integers serialize with no decimal point. Floats serialize using
CPython's shortest-round-trip `repr`, which uses fixed notation while
`-4 < decpt <= 16` and scientific notation otherwise, with a **signed,
at-least-two-digit exponent**, and always keeps a fractional part.

| value | canonical | common wrong output |
|---|---|---|
| `3` | `3` | |
| `3.0` | `3.0` | `3` |
| `0.0001` | `0.0001` | |
| `1e-5` | `1e-05` | `1e-5` |
| `1e16` | `1e+16` | `1e16` |
| `1e17` | `1e+17` | `1e17` |
| `1.5e300` | `1.5e+300` | `1.5e300` |
| `-0.0` | `-0.0` | `0.0` |

Rust's `ryu` (via `serde_json`) emits the right-hand column. That is a
fork, and it is invisible until a record happens to carry such a value.

## 5. Non-finite numbers are invalid

`NaN`, `Infinity`, `-Infinity` MUST NOT appear in a record. An
implementation that encounters one MUST refuse to seal (fail closed)
rather than emit a bare `NaN` token — which CPython's default
`allow_nan=True` would produce, yielding a record that verifies inside
that implementation and is unparseable by every strict JSON reader.

## 6. Null vs absent

Fields with no producing component are present and `null`. They are
never omitted and never populated with a placeholder. Both the field
set and the null values are inside the digest, so a stripped-and-
rehashed record fails the field-set check even though its hash is
internally consistent.

## Conformance

`test-vectors/clean.jsonl` exercises unicode and float cases above. An
implementation that reproduces every `record_hash` in that file is
byte-compatible with the reference; one that does not has forked, and
the divergence will be silent in production.

The reference implementation cross-checks a Rust and a Python
serializer against each other on exactly these vectors on every commit.
