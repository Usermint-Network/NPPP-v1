# NPPP v1 Canonicalization Rules

## Status of This Document

This document defines the canonicalization profile used to satisfy the deterministic bundle requirements of NPPP v1.

The normative protocol definition remains:

- `spec/NPPP_v1.md`

If this document conflicts with the normative specification, the specification governs.

This document operationalizes the deterministic serialization requirements defined in Section 8 of the NPPP v1 specification.

## Purpose

Canonicalization exists to guarantee determinism.

Without canonicalization, semantically identical artifacts may produce different bundle bytes, causing divergent SHA-256 commitments and unreliable replay verification.

## Canonicalization Goal

The goal of canonicalization is:

> identical logical inputs should produce identical bundle bytes

This property is required for replay verification to function reliably.

## Scope

Canonicalization applies to any input that influences the final serialized bundle bytes, including:

- files
- metadata
- structured records
- service-generated context
- archive ordering

## Required Canonical Properties

The following properties are required for a conforming NPPP v1 implementation:

1.  textual data MUST use UTF-8 encoding
2.  structured data MUST be serialized deterministically
3.  file ordering MUST be deterministic and stable
4.  incidental timestamps MUST NOT affect bundle bytes
5.  identical logical inputs MUST yield identical bundle bytes

This requirement applies to the exact byte sequence used as input to SHA-256.

Determinism is defined at the boundary of the canonicalized bundle bytes.

Any transformation applied prior to canonicalization is outside the scope of NPPP v1,
but the resulting canonicalized byte sequence MUST be stable and reproducible.

## Deterministic Implementation Strategies (Non-Normative)

The following strategies are recommended for achieving the required canonical properties:

### 1. JSON Serialization

-   object keys MUST be sorted lexicographically using a deterministic ordering
-   remove insignificant whitespace
-   use a single-line compact representation

Example:

```json
{"a":1,"b":2,"c":3}
```

### 2. File and Directory Ordering

-   sort files by stable full path
-   preserve a deterministic directory layout

### 3. Archive Generation

-   archive generation MUST be deterministic if archives are used as bundle containers
-   eliminate archive timestamps not intended as notarized evidence
-   choose tooling and flags that preserve stable output

---

## Non-Canonical Sources of Variance
Implementations SHOULD eliminate or normalize sources of accidental variation such as:
- filesystem-dependent ordering
- unstable metadata ordering
- environment-specific line endings
- runtime-generated temporary values
- archive timestamps not intended as notarized evidence

---

## Canonicalization of Structured Data
For structured data formats, implementations SHOULD ensure consistent serialization.
Examples of consistency include:
- stable object key ordering
- normalized whitespace
- consistent numeric representation where relevant

The exact encoding strategy may vary by implementation, but the resulting bundle bytes MUST remain deterministic.

---

## Canonicalization of Binary Artifacts
Binary artifacts SHOULD be included as-is unless a canonical binary normalization is explicitly defined.

If binary normalization is applied, it MUST be deterministic and reproducible across implementations.

The key requirement is that the byte sequence included in the bundle remains stable and intentional.

---

## Relationship to Bundling
Canonicalization precedes hashing.
The canonicalized inputs are assembled into the evidence bundle, and the final serialized bundle is hashed using SHA-256.
If canonicalization is inconsistent, bundle hashes will diverge.

The SHA-256 commitment in the proof is computed over the fully canonicalized bundle bytes.

The SHA-256 input MUST be exactly the canonicalized bundle byte sequence, with no additional framing, encoding, or transformation.

## Failure Consequences

If identical logical artifacts do not produce identical bundle bytes, then:

-   independent verifiers may produce inconsistent results for the same proof
-   proof hashes may diverge
-   replay verification may fail unexpectedly
-   interoperability between implementations may degrade
-   the notarization process loses determinism

For this reason, canonicalization is not optional in NPPP v1 systems.

## Trust Interpretation

Canonicalization does not make evidence true or trustworthy.

Canonicalization makes evidence stable enough to support deterministic hashing and reproducible replay verification.

NPPP relies on canonicalization to ensure that cryptographic verification remains meaningful across repeated verification attempts.

Canonicalization reduces ambiguity in input representation but does not protect against maliciously crafted inputs that are intentionally identical at the byte level.

---

## Versioning
These rules define canonicalization expectations for NPPP v1.
Future protocol versions may introduce stricter normalization requirements, artifact-specific canonical forms, or mandatory serialization standards.

Canonicalization is a prerequisite for interoperability across independent NPPP implementations.

Independent implementations that conform to these canonicalization rules SHOULD produce identical bundle bytes for identical logical inputs, enabling cross-system verification interoperability.

See also:
- spec/NPPP_v1.md (normative protocol definition)
- docs/bundle-structure.md (bundle construction model)

