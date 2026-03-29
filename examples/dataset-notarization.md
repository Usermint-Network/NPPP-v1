# Dataset Notarization (NPPP v1 Example)

## Status

This document is non-normative and provided for illustrative purposes only.

The normative protocol definition remains:

- spec/NPPP_v1.md

---

## Overview

This example demonstrates how NPPP v1 may be used to notarize a dataset.

---

## Use Case

A data pipeline produces a dataset and requires proof of:

- dataset integrity
- reproducibility
- independent verification

---

## Notarization Request

```bash
POST /v1/notarize
```

```json
{
  "service": "example-service",
  "artifact_type": "dataset",
  "freshness": "7d",
  "evidence": {
    "path": "bundle/"
  }
}
```

---

## Notarization Response

```json
{
  "proof_id": "<generated_proof_id>",
  "proof": "NPPP:V1|project=<project>|region=<region>|service=example-service|freshness=7d|bundle=<bundle_uri>|sha256=<sha256>|created=<timestamp>"
}
```

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

Verification is performed via stateless recomputation.

The verifier:
- retrieves the dataset bundle
- recomputes SHA-256
- compares with the proof

Verification:
- is independent of the original system
- requires no stored verification state
- is reproducible across implementations

## What Is Proven

- dataset bytes match the committed SHA-256 value
- dataset integrity is preserved
- proof is independently verifiable

## What Is NOT Proven

- dataset quality
- dataset correctness
- semantic meaning of the data

## Interpretation

NPPP v1 proves byte-level integrity and reproducibility.
It does not assert correctness or usefulness of the dataset.

---
