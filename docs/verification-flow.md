# NPPP Verification Flow

This document defines the **stateless replay verification flow** for **NPPP v1**.

Verification in NPPP is not about checking a stored status. It is about re-running the original computation to prove that an evidence bundle has not changed.

---

## Core Principles of Verification

### 1. Stateless Verification
Verification is a pure, stateless function. The outcome depends only on the inputs (the proof string and the retrieved bundle), not on any stored history, database flag, or pre-existing state. A verifier does not need to consult a central authority to determine validity.

### 2. Replay Semantics
Verification works by "replaying" the final step of the original notarization: hashing the canonical bundle. The verifier fetches the *exact same bundle* that was created during notarization and re-computes the SHA-256 hash. If the computed hash matches the hash in the proof string, integrity is proven.

### 3. No Stored Truth
The NPPP protocol **never** stores the *result* of a verification. A proof record does not say "this proof was successfully verified at this time." Instead, the system is designed so that anyone can re-derive the truth at any moment. The `verification` object in a proof record (`"status": "unverified"`) is a declaration of this principle.

---

## Verification Inputs

A verifier requires only two things:

1.  A valid NPPP v1 proof string.
2.  Network access to the evidence bundle referenced in the proof.

---

## The Stateless Verification Procedure

A verifier **MUST** perform the following steps in order.

### Step 1: Parse and Validate the Proof String
- The verifier parses the proof string according to the strict grammar in `spec/NPPP_v1.md`.
- It **MUST** reject the proof if the prefix, field order, field count, or format is invalid. This is a format check, not a verification failure.

### Step 2: Retrieve the Evidence Bundle
- The verifier fetches the byte-for-byte exact evidence bundle from the URI in the `bundle` field.
- If the bundle cannot be retrieved (e.g., due to a 404 error or a network failure), the process stops. This is a **retrieval error**, not a verification failure. The verifier should report that the bundle is unreachable.

### Step 3: Re-Compute the SHA-256 Hash
- The verifier computes the SHA-256 hash of the entire retrieved bundle's byte stream. No transformations are allowed.

### Step 4: Compare Hashes
- The verifier performs a byte-by-byte comparison of the newly computed hash and the `sha256` value from the proof string.

---

## Verification Outcomes

Based on Step 4, there are only two possible verification outcomes:

### `SHA Replay Verified: OK`
- **Meaning:** The recomputed SHA-256 hash of the fetched bundle *exactly* matches the hash in the proof.
- **Conclusion:** The evidence bundle has not been altered since it was notarized.

### `SHA Replay Verified: FAILED`
- **Meaning:** The recomputed hash *does not* match the hash in the proof.
- **Conclusion:** The evidence bundle has been altered, corrupted, or is not the correct bundle for this proof.

Any other error (invalid format, bundle unreachable) is an error in the process, not a verification outcome.

---
## Trust Model in Verification

This stateless, replay-based model is designed to minimize trust. A verifier does not need to trust:
- The system that originally created the proof.
- The client that submitted the artifact.
- Any database or stored "status".

Trust is placed only in the mathematics of SHA-256 and the ability to re-run the computation.
