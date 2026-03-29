# NPPP v1 Economic Model

## Status of This Document

This document describes the economic model of a reference
implementation of NPPP v1.

The normative protocol definition remains:

- `spec/NPPP_v1.md`

This document is non-normative and does not modify or extend protocol semantics defined in the specification.

Economic models, pricing strategies, and billing mechanisms are
implementation-specific and are NOT part of the NPPP v1 protocol.

## Overview

The NPPP protocol separates proof creation from proof verification.

- **Proof creation** (notarization) may incur cost due to computation,
  storage, and infrastructure requirements.
- **Proof verification** is designed to remain freely accessible and
  independently executable.

This separation enables:

- open and permissionless verification
- predictable infrastructure cost modeling
- scalable and sustainable operation
- minimal friction for developers and integrators

## Implementation Boundary

The NPPP protocol defines proof structure and verification semantics.

Economic behavior, including pricing, billing, quotas, and access policies, is defined entirely by the implementing operator.

Multiple independent implementations may adopt different economic models while remaining fully conformant with NPPP v1.

Economic behavior MUST NOT alter the deterministic properties required for proof generation and verification.

## Core Principle

A typical NPPP-aligned implementation follows the principle:

> Proof creation MAY incur cost.  
> Proof verification SHOULD remain free.

This mirrors foundational internet infrastructure patterns:

| System                  | Paid Operation        | Free Operation      |
| :---------------------- | :-------------------- | :------------------ |
| Certificate Authorities | Certificate issuance  | TLS verification    |
| Blockchains             | Transaction inclusion | Block verification  |
| NPPP (reference model)  | Evidence notarization | Proof verification  |

Free verification ensures that proof validation remains independent,
reproducible, and universally accessible.

## Notarization Economics

In a reference implementation, a notarization operation may include:

- artifact canonicalization
- deterministic bundle construction
- SHA-256 integrity computation
- evidence bundle persistence
- proof string generation
- proof record persistence

These operations incur infrastructure costs related to:

- compute execution
- storage durability
- network access
- system monitoring

### Example Pricing (Non-Normative)

A reference implementation MAY adopt a pricing model for notarization operations.

Pricing is not defined by the protocol and MUST NOT affect proof structure, verification semantics, or interoperability.

## Developer Access Model

To encourage adoption and experimentation, implementations MAY provide:

- a limited number of free notarization operations
- test or sandbox environments
- development API access tiers

### Example Allocation (Non-Normative)

- **20 free notarizations per developer account**

This enables:

- rapid prototyping
- CI/CD integration testing
- verification workflow validation

## Verification Economics

Verification of a proof is defined as a stateless recomputation process.

Verification does not rely on stored verification state.

Verification MUST NOT require trusted stored verification flags.

Verification is derived solely from:

- the proof string
- the referenced bundle
- deterministic recomputation

A verification operation typically involves:

1. parsing the proof string
2. retrieving the referenced bundle
3. recomputing the SHA-256 hash
4. comparing computed and declared values

Because verification does not require privileged access or authoritative state:

- it MAY be performed by any independent verifier
- it SHOULD NOT require payment
- it SHOULD remain accessible without restriction

Open verification is essential to maintaining trust in the NPPP model.

Independent verifiers SHOULD be able to perform verification without interacting with the originating notarization service.
Verification MUST NOT depend on privileged access to implementation-specific infrastructure.

## Infrastructure Considerations

Operating an NPPP-compliant system may require:

- notarization services
- deterministic bundle processing pipelines
- durable object storage
- proof metadata storage
- API infrastructure
- verification tooling
- monitoring and observability systems

The economic model for notarization operations typically reflects these infrastructure requirements.

Verification operations, by contrast, are lightweight and may be performed independently of the originating system.

## Design Goals

The NPPP economic model is guided by the following principles:

### Predictability

Costs should be simple to estimate.
Flat, per-operation pricing provides
clear cost modeling for high-volume systems.

### Open Verification

Verification must remain accessible to any party.
Charging for verification would undermine independent validation
and reduce trust in the system.

### Minimal Friction

Notarization costs should remain low enough to support:

- automated CI/CD evidence generation
- AI system provenance tracking
- compliance and audit logging
- infrastructure state verification

### Sustainable Operation

The system must support:

- durable storage of evidence bundles
- reliable notarization services
- long-term operational stability

Charging for proof creation enables sustainable infrastructure without restricting verification.

## Separation of Protocol and Economics

NPPP v1 intentionally separates:

- protocol semantics (proof structure, verification model)
- economic models (pricing, billing, quotas)

The protocol:

- does not define pricing
- does not enforce payment
- does not require a specific billing model

Economic behavior is entirely defined by the implementing operator.

## Future Economic Extensions

Implementations MAY introduce additional economic layers, including:

- proof indexing or explorer services
- artifact lineage tracking
- compliance dashboards
- automated verification services
- enterprise service tiers

Such extensions MUST NOT alter the core protocol invariant:

> verification remains independently reproducible through deterministic recomputation.

## Economic Philosophy

The NPPP economic model preserves a critical asymmetry:

- proof creation reflects infrastructure cost
- proof verification remains universally accessible

This asymmetry ensures that trust in the system derives from independent recomputation rather than from privileged access or centralized authority.

## Summary

The NPPP economic model is intentionally minimal:

- proof creation MAY incur a cost
- proof verification SHOULD remain free
- developers MAY receive limited free usage
- pricing is implementation-defined

This model supports:

- open and independent verification
- sustainable infrastructure operation
- scalable adoption across systems and industries

The economic design reinforces the central goal of NPPP:

> enabling deterministic, replay-verifiable proof without requiring trust
in stored state or centralized custody.
