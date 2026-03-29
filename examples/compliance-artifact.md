# Compliance Artifact Proof (NPPP v1 Example)

## Status

This document is non-normative and provided for illustrative purposes only.

The normative protocol definition remains:

- spec/NPPP_v1.md

---

## Overview

This example demonstrates how NPPP v1 may be used to notarize compliance-related artifacts.

---

## Use Case

An organization generates documents such as:

- audit reports
- financial statements
- regulatory filings

The organization requires verifiable proof of:

- document integrity
- existence at time of notarization
- independent auditability

---

## Notarization Request

```bash
POST /v1/notarize
```

```json
{
  "service": "example-service",
  "artifact_type": "compliance_document",
  "freshness": "90d",
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
  "proof": "NPPP:V1|project=<project>|region=<region>|service=example-service|freshness=90d|bundle=<bundle_uri>|sha256=<sha256>|created=<timestamp>"
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

Verification is performed through deterministic recomputation.

A verifier:
- retrieves the bundle
- recomputes SHA-256
- compares the result to the proof

Verification:
- is stateless
- does not rely on stored verification flags
- does not require privileged access

## What Is Proven

- the document bundle has not been altered
- the bundle corresponds to a specific SHA-256 commitment
- the proof can be independently verified

## What Is NOT Proven

- legal validity of the document
- regulatory compliance status
- authorship or authority

## Interpretation

NPPP v1 provides integrity guarantees.
It does not replace legal, regulatory, or audit frameworks.

---
