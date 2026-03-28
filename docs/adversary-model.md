# NPPP v1 Adversary Model

## Purpose

This document describes the adversaries NPPP v1 is designed to handle, the guarantees it provides, and the guarantees it does not provide.

This document is explanatory and should be read together with:

- `spec/NPPP_v1.md`
- `docs/trust-model.md`

---

## Core Observation

NPPP v1 is an integrity protocol, not a semantic-truth protocol.

Its goal is to make the following claim replayable:

> the referenced bundle bytes hash to the committed SHA-256 value.

---

## Trust Domains

NPPP v1 conceptually separates four domains:

### 1. Caller Domain

Provides the initial artifact description and metadata.

### 2. Notarization Service Domain

Constructs bundles, computes SHA-256, issues proofs, and stores proof records.

### 3. Storage Domain

Persists bundle bytes and makes them retrievable.

### 4. Verifier Domain

Fetches bundle bytes and recomputes SHA-256.

---

## Adversary Classes

### 1. Malicious Caller

**Capabilities**:

- submits dishonest metadata,
- mislabels artifacts,
- attempts to create misleading notarization requests.

**NPPP response**:

- protocol does not guarantee semantic truth,
- protocol guarantees only deterministic commitment to bundle bytes.

### 2. Storage Tampering Adversary

**Capabilities**:

- modifies stored bundle bytes after notarization.

**NPPP response**:

- replay verification recomputes SHA-256,
- altered bytes produce verification failure.

### 3. Replay Adversary

**Capabilities**:

- reuses proofs in other contexts,
- attempts to treat proof presence as semantic truth.

**NPPP response**:

- proof remains a commitment to specific bundle bytes,
- protocol does not claim broader semantic meaning.

### 4. Verifier Spoofing Adversary

**Capabilities**:

- attempts to assert a verification result without recomputation.

**NPPP response**:

- canonical verification is defined as replay computation,
- stored flags are not authoritative protocol truth.

### 5. Partial Infrastructure Compromise

**Capabilities**:

- compromises one layer of the deployment stack.

**NPPP response**:

- stateless replay reduces trust in stored verification flags,
- independent verifiers can recompute from bundle bytes and proof.

---

## What NPPP v1 Defends Against

NPPP v1 is designed to detect:

- bundle mutation,
- proof/commitment mismatch,
- tampering that changes committed bytes,
- false confidence based solely on stored mutable verification state.

---

## What NPPP v1 Does Not Defend Against

NPPP v1 does **not** by itself defend against:

- false semantic claims by the caller,
- authorship fraud,
- stolen identities,
- unavailable bundles,
- fraudulent timestamps,
- legal disputes about content meaning.

---

## Integrity Interpretation

A successful verification means:

```text
SHA256(bundle_bytes) == proof.sha256
```

A successful verification does **not** mean:

- the artifact is true,
- the artifact is lawful,
- the artifact is meaningful,
- the artifact was authored by a specific person,
- the bundle will remain available forever.

## Why Stateless Verification Matters

A mutable `verified` flag can be forged, copied, cached, or become stale.

NPPP v1 treats replay verification as the only canonical way to derive integrity truth.

This makes the protocol more resistant to trust inflation through stored state.

## Security Summary

NPPP v1 is strongest against:

- integrity drift,
- storage tampering,
- stale verification assumptions.

NPPP v1 is **not** a substitute for:

- identity systems,
- timestamp authority,
- semantic review,
- governance,
- legal adjudication.
