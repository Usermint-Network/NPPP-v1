# NPPP v1 Trust Model

## Purpose

This document explains how trust is minimized in NPPP v1 and clarifies what the protocol does and does not ask the verifier to trust.

This document is explanatory. The normative protocol definition is in `spec/NPPP_v1.md`.

---

## Core Principle

NPPP v1 minimizes trust by separating:

- immutable proof custody, and
- verification truth.

A proof record is an attestation.  
A verification result is a recomputation.

The protocol does not require the verifier to trust a stored mutable verification flag.

---

## What NPPP v1 Asks You to Trust

NPPP v1 asks you to trust only the following narrow claim:

> a specific bundle of bytes hashes to a specific SHA-256 value, and that equality can be independently recomputed.

When replay verification succeeds, the verifier can conclude:

```text
SHA256(bundle_bytes) == proof.sha256
```

Nothing stronger is implied by the protocol itself.

## What NPPP v1 Does Not Ask You to Trust

NPPP v1 does not require trust in:

- a stored `verified` flag,
- semantic correctness of artifact contents,
- the intentions of the caller,
- infrastructure branding,
- identity-provider branding,
- prior verification history.

Instead, it derives verification truth from deterministic recomputation.

## Trust Separation

### 1. Proof Custody

The proof record stores:

- `proof_id`
- `proof`
- `bundle`
- `bundle_sha256`
- protocol metadata
- verification model metadata

This stored record is an immutable attestation surface.

### 2. Verification Truth

Verification truth is derived by replay:

- parse proof
- resolve bundle
- fetch bundle bytes
- recompute SHA-256
- compare computed digest to proof digest

This verification result is computed at request time.

## Meaning of the Proof Record Verification Object

A conforming v1 proof record includes:

```json
{
  "verification": {
    "status": "unverified",
    "model": "stateless"
  }
}
```

This means:

- the proof record does not persist replay verification as canonical truth,
- the verification model is `stateless`,
- verification should be derived through recomputation.

It does **not** mean the proof is invalid, weak, or incomplete.

## Why Stateless Verification Matters

Mutable verification flags create avoidable trust problems:

- they can become stale,
- they can be cached incorrectly,
- they can be copied without replay,
- they can imply truth without recomputation.

NPPP v1 avoids this by making replay verification the source of truth.

## What the Protocol Guarantees

When implemented correctly, NPPP v1 provides:

- deterministic proof structure,
- deterministic replay verification,
- SHA-256–based tamper detection,
- reduced reliance on stored mutable state.

## What the Protocol Does Not Guarantee

NPPP v1 does **not** guarantee:

- semantic truth,
- authorship,
- real-world identity binding,
- permanent bundle availability,
- decentralized consensus,
- third-party timestamp authority.

These are outside the v1 protocol scope.

## Trust Summary

NPPP v1 is designed to reduce trust in systems and increase trust in recomputation.

Its trust model is simple:

- do not trust stored verification state,
- do trust deterministic cryptographic replay,
- do not infer semantics from integrity,
- do treat proof custody and verification truth as distinct.
