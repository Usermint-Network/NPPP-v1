# AI Model Proof (NPPP v1 Example)

## Status

This document is non-normative and provided for illustrative purposes only.

The normative protocol definition remains:

- spec/NPPP_v1.md

---

## Overview

This example demonstrates how NPPP v1 may be used to notarize an AI model artifact.

The goal is to produce a deterministic proof that can be independently verified without relying on stored verification state.

---

## Use Case

An AI system produces a trained model and associated artifacts.

The system may require a verifiable proof of:

- model integrity
- training artifact consistency
- reproducibility of outputs

---

## Notarization Request

```bash
POST /v1/notarize
```

```json
{
  "service": "example-service",
  "artifact_type": "ai_model",
  "freshness": "30d",
  "evidence": {
    "path": "bundle/"
  }
}
```

**Notes**

The evidence.path represents a canonical bundle containing model artifacts (e.g., weights, configuration, metadata). The exact bundle structure is implementation-defined but MUST be deterministic.

---

## Notarization Response

```json
{
  "proof_id": "<generated_proof_id>",
  "proof": "NPPP:V1|project=<project>|region=<region>|service=example-service|freshness=30d|bundle=<bundle_uri>|sha256=<sha256>|created=<timestamp>"
}
```

**Notes**

Field values shown are illustrative. The proof string MUST conform to the canonical grammar defined in the specification.

---

## Verification

```bash
POST /v1/verify
```

```json
{
  "proof": "NPPP:V1|..."
}
```

## Verification Model

Verification is performed through stateless recomputation.

A verifier:
- parses the proof string
- retrieves the referenced bundle
- recomputes the SHA-256 hash
- compares it to the proof

Verification:
- does not rely on stored verification state
- does not require access to the original notarization system
- is independently reproducible

## What Is Proven

- the canonical bundle has not changed
- the bundle hashes to the committed SHA-256 value
- the proof can be independently verified

## What Is NOT Proven

- model correctness
- model performance
- semantic validity of outputs

## Interpretation

NPPP v1 proves integrity and reproducibility of artifact bytes. It does not assert correctness, authorship, or semantic truth.

---
