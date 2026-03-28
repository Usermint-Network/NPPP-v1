# NPPP v1 Specification

## Status of This Document

This document defines **NPPP v1** as the normative specification for the Notarized Proof of Provenance Protocol.

This specification describes:

- proof grammar,
- field semantics,
- notarization behavior,
- replay verification behavior,
- proof-record semantics,
- protocol guarantees,
- protocol non-goals.

Implementation-specific concerns such as infrastructure, hosting, outer access control, and metering are not part of the normative protocol unless explicitly stated.

---

## 0. Normative Language

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT", and "MAY" in this document are to be interpreted as described in RFC 2119.

---

## 1. Purpose

NPPP v1 is a protocol for issuing and verifying deterministic cryptographic proofs of digital artifact integrity.

The protocol enables a conforming implementation to:

1. construct a deterministic bundle from input data,
2. compute a SHA-256 commitment over that bundle,
3. issue a canonical proof string,
4. persist an immutable proof record,
5. replay verification by recomputing the referenced bundle hash.

NPPP v1 is designed to minimize trust in stored verification state by making verification dependent on recomputation rather than mutable flags.

---

## 2. Design Principle

NPPP v1 does not assert semantic truth.

NPPP v1 proves that:

> a specific bundle of bytes hashes to a specific SHA-256 value, and that this equality can be deterministically recomputed at any later time.

Verification in NPPP v1 is therefore:

- deterministic,
- replayable,
- stateless.

## 2.1 Non-Authenticity Guarantee

NPPP v1 does not provide:

- authorship guarantees
- identity binding
- origin authenticity
- semantic correctness

A valid proof only demonstrates byte-level integrity.

Consumers of NPPP proofs MUST NOT interpret proof validity as a guarantee of trustworthiness or authorship.

---

## 3. Definitions

### 3.1 Artifact
A logical object submitted for notarization.

### 3.2 Bundle
The canonical byte representation persisted by the implementation and referenced by the proof.

### 3.3 Proof
A structured protocol string containing metadata and a SHA-256 commitment to a bundle.

### 3.4 Proof Record
An immutable stored attestation associated with a `proof_id`.

### 3.5 Replay Verification
The act of re-fetching the referenced bundle, recomputing SHA-256, and comparing the result to the proof commitment.

### 3.6 Stateless Verification
A verification model in which the verifier derives truth from recomputation rather than from a stored mutable verification flag.

---

## 4. Protocol Scope

NPPP v1 defines:

- proof format,
- field ordering,
- field constraints,
- verification semantics,
- result vocabulary,
- proof-record semantics.

NPPP v1 does **not** define:

- a canonical authentication system,
- a canonical cloud provider,
- a canonical identity provider,
- a canonical storage backend,
- a canonical transport beyond requiring a reachable interface,
- decentralized consensus,
- timestamp authority,
- semantic interpretation of submitted artifacts.

---

## 5. Transport and Access Neutrality

NPPP v1 is transport-agnostic and access-control neutral.

A conforming implementation MAY expose the protocol over HTTPS or any other reachable transport.

A conforming implementation MAY apply one or more outer access-control layers, including but not limited to:

- bearer tokens,
- API gateways,
- mTLS,
- private network controls,
- custom authorization systems.

Such controls are deployment-specific and are **not** part of the NPPP v1 normative protocol.

---

## 6. Canonical Proof Format

An NPPP v1 proof string MUST use the following exact structure and field order:

```text
NPPP:V1|project={project}|region={region}|service={service}|freshness={freshness}|bundle={bundle_uri}|sha256={bundle_sha256}|created={created_at}
```
### 6.1 Prefix
A proof MUST begin with:
```plain text
NPPP:V1|
```
### 6.2 Field Order
The proof fields MUST appear in exactly this order:
1. `project`
2. `region`
3. `service`
4. `freshness`
5. `bundle`
6. `sha256`
7. `created`

A proof with correct field names but incorrect ordering is invalid.
### 6.3 Delimiters
Fields MUST be separated by the pipe (|) character.
Each field after the prefix MUST use the format:
```plain text
key=value
```
### 6.4 Empty Values
Field values MUST NOT be empty.

### 6.5 Formal Grammar (ABNF)

The proof string MUST conform to the following grammar:
```abnf
proof = "NPPP:V1|"
        "project=" value "|"
        "region=" value "|"
        "service=" value "|"
        "freshness=" value "|"
        "bundle=" value "|"
        "sha256=" 64HEXDIG "|"
        "created=" value

value       = 1*(%x21-7E)  ; printable ASCII except space
```

In v1, proof field values are restricted to printable non-space ASCII to reduce encoding ambiguity and preserve deterministic interoperability.

---

## 7. Field Semantics
### 7.1 `project`
A deployment or environment project identifier. These fields are informational and do not establish trust or authority.
- **Type**: string
- **Constraint**: non-empty

### 7.2 `region`
A deployment execution region identifier. These fields are informational and do not establish trust or authority.
- **Type**: string
- **Constraint**: non-empty

### 7.3 `service`
A logical service identifier associated with the notarized artifact.
- **Type**: string
- **Constraint**: non-empty

### 7.4 `freshness`
A caller-supplied freshness descriptor.
- **Type**: string
- **Constraint**: non-empty

The `freshness` field is an opaque, caller-defined value.

NPPP v1 does not interpret or enforce freshness semantics.

### 7.5 `bundle`
A URI identifying the persisted canonical bundle.
- **Type**: string
- **Constraint**: MUST be a valid bundle URI for the implementation
In the reference implementation for v1, this is a gs:// URI.

### 7.6 `sha256`
The SHA-256 digest of the canonical bundle bytes.
- **Type**: string
- **Constraint**: MUST be exactly 64 lowercase hexadecimal characters

### 7.7 `created`
A UTC creation timestamp.
- **Type**: string
- **Constraint**: MUST match:
```plain text
YYYY-MM-DDTHH:MM:SSZ
```

---

## 8. Canonical Bundle Requirement
A conforming implementation MUST persist or otherwise reference a canonical bundle whose byte sequence is the source of the proof’s SHA-256 commitment.
The protocol requires that:
```plain text
sha256(bundle_bytes) == proof.sha256
```
when the bundle has not been modified.
NPPP v1 does not require a single canonical storage backend, but it requires that the verifier be able to retrieve the bundle bytes associated with the proof.

### 8.1 Deterministic Serialization Requirement

A conforming implementation MUST use UTF-8 encoding for bundle serialization.

## 8.2 Bundle Retrieval Semantics

A verifier MUST attempt to retrieve the bundle referenced by the proof.

If retrieval fails, the verifier MUST NOT return `SHA Replay Verified: FAILED`.

Instead, the verifier SHOULD return an implementation-defined retrieval error.

Examples include:

- `bundle_unreachable`
- `bundle_not_found`
- `access_denied`

Absence of the bundle does not imply proof invalidity.

## 8.4 Canonical Serialization Standard

A conforming implementation MUST serialize bundle data using a deterministic canonical encoding.

For JSON-based artifacts, the following rules MUST apply:

- Encoding MUST be UTF-8
- Object keys MUST be sorted lexicographically
- No insignificant whitespace is permitted
- Numbers MUST use canonical JSON representation
- Strings MUST be encoded without normalization changes

The resulting byte sequence MUST be identical for identical logical input.

Failure to enforce canonical serialization results in non-conforming implementations.

## 8.5 Bundle Immutability Requirement

A conforming implementation SHOULD ensure that the bundle referenced by the proof is immutable.

Recommended mechanisms include:

- content-addressed storage (e.g., hash-based paths)
- object versioning with retention locks
- write-once storage systems

If bundle immutability is not guaranteed, verification reliability is reduced but not invalidated.

A verifier MUST treat bundle mutation as a verification failure when detected.

---

## 9. Notarization Procedure
A conforming implementation MUST perform the following logical steps during notarization:
1. accept a notarization request,
2. validate required request fields,
3. derive or construct a canonical bundle payload,
4. serialize the bundle deterministically,
5. compute SHA-256 over the canonical bundle bytes,
6. persist the bundle or its immutable equivalent,
7. construct a proof string using the required field order,
8. construct a proof record,
9. persist the proof record,
10. return the notarization response.

### 9.1 Determinism Requirement
The serialization used to derive the bundle hash MUST be deterministic.
If the same canonical input produces different bundle bytes across implementations or executions, the implementation is not conforming.

---

## 10. Verification Procedure
A conforming verifier MUST perform the following steps:
1. receive a proof string,
2. validate the proof prefix,
3. split the proof into fields,
4. validate field count,
5. validate field order,
6. parse field values,
7. validate `sha256` format,
8. validate `bundle` URI format according to implementation rules,
9. validate `created` format,
10. fetch the referenced bundle bytes,
11. compute SHA-256 over the fetched bundle bytes,
12. compare computed SHA-256 to the proof `sha256`,
13. return a verification result.

### 10.1 Proof Parsing Requirements

A conforming verifier MUST:

- split the proof string on `|`
- validate the prefix `NPPP:V1`
- parse each field into key-value pairs
- enforce field order before semantic validation

A verifier MUST NOT reorder fields prior to validation.

---

## 11. Proof Validation Rules
A verifier MUST reject a proof when any of the following is true:
- `prefix` is not `NPPP:V1|`
- field count is incorrect
- field ordering is incorrect
- any field lacks `=`
- any field value is empty
- `sha256` is not 64 lowercase hex characters
- `bundle` is not a valid URI for the implementation
- `created` is not in the required UTC format

---

## 12. Verification Result Vocabulary
A conforming verifier response in NPPP v1 MUST use one of the following status values:
- `SHA Replay Verified: OK`
- `SHA Replay Verified: FAILED`

### 12.1 Meaning of `SHA Replay Verified: OK`
The recomputed SHA-256 of the fetched bundle exactly equals the `sha256` value encoded in the proof.
### 12.2 Meaning of `SHA Replay Verified: FAILED`
The recomputed SHA-256 of the fetched bundle does not equal the `sha256` value encoded in the proof.

### 12.3 Verification Error Conditions

A verifier MAY return structured error conditions including:

- `invalid_proof_format`
- `invalid_field_order`
- `invalid_sha256_format`
- `bundle_unreachable`
- `bundle_corrupted`

These error conditions are implementation-specific but SHOULD NOT be conflated with verification success or failure.

### 12.4 Verification vs Retrieval

Verification result vocabulary applies only when bundle retrieval succeeds.

If bundle retrieval fails, the verifier MUST NOT return:

- `SHA Replay Verified: OK`
- `SHA Replay Verified: FAILED`

Instead, retrieval errors MUST be surfaced separately.

---

## 13. Proof Record Semantics
A conforming proof record MUST include the following fields:
- `proof_id`
- `proof`
- `bundle`
- `bundle_sha256`
- `created_at`
- `service`
- `artifact_type`
- `freshness`
- `verification`

### 13.1 `verification` Object
The `verification` object in a v1 proof record MUST be:
```json
{
  "status": "unverified",
  "model": "stateless"
}
```

### 13.2 Interpretation
This object does not mean the proof is invalid or incomplete.
It means:
- the proof record itself does not persist replay verification as canonical truth,
- replay verification remains a recomputation step,
- the record declares the verification model, not historical verification events.

---

## 14. Request and Response Concepts
NPPP v1 does not require a single transport binding, but the reference surface includes the following logical operations:
- `bootstrap developer session`
- `retrieve credits or metering balance`
- `notarize`
- `verify`
- `retrieve proof record`

These operations may be exposed differently by different implementations, but where they are exposed, they MUST preserve the proof and verification semantics defined in this specification.

---

## 15. Protocol Guarantees
NPPP v1 guarantees the following when implemented correctly:
### 15.1 Deterministic Proof Structure
Proof strings follow a fixed grammar and field ordering.
### 15.2 Replayable Verification
Any conforming verifier can recompute the verification result from the proof and bundle.
### 15.3 Hash Integrity
If verification returns `SHA Replay Verified: OK`, then the fetched bundle bytes match the committed SHA-256 value.
### 15.4 Tamper Detection
If the referenced bundle bytes are modified, replay verification MUST fail.
### 15.5 Reduced Trust in Stored State
Verification truth does not depend on a mutable stored “verified” flag.

### 15.6 Integrity Scope

NPPP v1 guarantees only:

- integrity of the bundle bytes referenced by the proof
- reproducibility of SHA-256 equality

NPPP v1 does not guarantee:

- availability of the bundle
- immutability of the bundle
- persistence of storage

---

## 16. Non-Guarantees
NPPP v1 does not guarantee:
### 16.1 Semantic Truth
The protocol does not prove that the artifact content is meaningful, accurate, or honest.
### 16.2 Identity Authenticity
The protocol does not cryptographically bind a proof to a real-world identity.
### 16.3 Permanent Availability
The protocol does not guarantee indefinite storage or reachability of the bundle.
### 16.4 Trusted Time Authority
The `created` timestamp is implementation-generated and is not, by itself, a third-party notarized timestamp.
### 16.5 Decentralization
NPPP v1 does not require decentralized consensus or blockchain anchoring.

---

## 17. Security Interpretation
NPPP v1 is an integrity protocol.
It provides a deterministic mechanism for proving that:
- a canonical bundle was created,
- a SHA-256 commitment was issued,
- the equality between commitment and bundle bytes can be replayed.
It is not a protocol for:
- authorship proof,
- semantic validation,
- legal truth,
- decentralized settlement.

---

## 18. Trust Boundary Summary

## 18.1 Storage Trust Model

The integrity of NPPP verification depends on the ability to retrieve the correct bundle bytes.

NPPP v1 assumes:

- storage may be untrusted
- storage may be mutable
- storage may become unavailable

Trust is therefore derived from:

- deterministic recomputation
- SHA-256 collision resistance

Implementations SHOULD document:

- storage guarantees
- immutability guarantees
- retention guarantees

NPPP itself does not enforce these properties.

NPPP v1 divides responsibility conceptually into:
- caller domain,
- notarization service domain,
- storage domain,
- verifier domain.

A conforming implementation SHOULD document these boundaries separately in implementation or trust-model documentation.
The normative protocol itself remains focused on proof construction and replay verification behavior.

The protocol assumes no trust in:

- storage persistence guarantees
- infrastructure providers
- identity providers

Trust is derived solely from deterministic recomputation.

---

## 19. Extensibility

## 19.1 Content-Addressable Storage (Recommended)

Implementations SHOULD use content-addressable storage where:

bundle_uri = location derived from sha256(bundle_bytes)

This ensures:

- inherent immutability
- verifiable storage integrity
- reduced trust in storage systems

Future versions MAY extend NPPP with:
- signature layers,
- external timestamp authority,
- alternate storage backends,
- multi-party attestation,
- alternate hash algorithms,
- zero-knowledge verification mechanisms,
- richer verification model vocabularies.
Such extensions MUST NOT be represented as NPPP v1 unless they preserve v1 semantics and wire format.

---

## 20. Versioning
This document defines NPPP v1.
Any change to:
- proof grammar,
- field ordering,
- required fields,
- verification result vocabulary,
- verification object semantics,
SHOULD be treated as a versioned protocol change.

---

## 21. Minimal Conformance
An implementation is minimally conformant with NPPP v1 if it:
- issues proofs that conform to the required proof grammar,
- computes SHA-256 over deterministic bundle bytes,
- exposes a replay verification process,
- uses the required verification result vocabulary,
- exposes or persists proof records whose `verification` object is:
  - `status` = `"unverified"`
  - `model` = `"stateless"`

---

## 22. Canonical Summary
NPPP v1 is a stateless, replay-based integrity protocol.
Its canonical claim is not:

> “this artifact is true”

Its canonical claim is:

> “this exact bundle hashes to this exact SHA-256 value, and that equality can be independently recomputed.”

## 23. Reference Implementation

A reference implementation of NPPP v1 is available and demonstrates conformance to:

- proof grammar
- deterministic bundle hashing
- replay verification semantics

This implementation is informative and non-normative, and serves as a practical guide to protocol conformance.
