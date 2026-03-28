# NPPP Architecture

System Architecture for the Notarization Proof Packet Protocol

---

## Overview

NPPP provides infrastructure for generating and verifying replay-verifiable evidence packets.

The architecture consists of five primary components:

- Client Interfaces
- Notarization Service
- Evidence Storage
- Proof Ledger
- Verification System

Together these components allow systems to produce evidence that can be verified independently at any point in the future.

---

## Architectural Domains

To understand the NPPP architecture, it is helpful to distinguish between four key domains:

### 1. The Protocol

The **NPPP Protocol** is a set of rules. It is a specification that defines:
- The format of a proof (`spec/NPPP_v1.md`)
- The process for deterministic bundle creation
- The stateless verification logic

The protocol itself is just a document. It does not perform any actions.

### 2. A Reference Implementation

A **Reference Implementation** is software that implements the protocol. The documentation may refer to the **UserMint Network** as a reference implementation. An implementation is responsible for:
- Exposing an API (like `openapi/openapi.json`)
- Executing the notarization and verification logic
- Managing operational concerns like authentication and rate limiting

### 3. The Storage Domain

The **Storage Domain** is the system responsible for storing and retrieving the evidence bundles. This could be a cloud storage provider (like GCS or S3) or any other system that can store and retrieve files. The key requirements are durability and byte-level integrity.

### 4. The Verifier Domain

The **Verifier Domain** is the context in which verification occurs. Because verification is stateless, anyone can be a verifier. A verifier needs only:
- The NPPP proof string
- Access to the evidence bundle in the storage domain

The verifier re-runs the computation described in the protocol to confirm the integrity of the evidence.

---

## System Components

### Client Interfaces

Developers interact with NPPP through two primary interfaces:

- **CLI Interface**
`cli/README.md`

- **HTTP API**
`api/README.md`

These interfaces allow developers to submit artifacts for notarization and verify previously generated proofs.

---

### Notarization Service

The Notarization Service is responsible for producing NPPP proof packets.

Its responsibilities include:

- canonicalizing submitted artifacts
- generating deterministic evidence bundles
- computing SHA-256 integrity hashes
- constructing NPPP proof strings
- recording proof metadata

This service forms the core execution layer of the protocol.

---

### Evidence Storage

Evidence bundles are stored in durable object storage.

Example storage location:
```
gs://usermint-network-notary-dev/bundles/
```

Bundles contain the artifacts required to reproduce a proof.

Typical bundle contents include:
```
bundle/
  artifact/
  metadata/
  receipts/
```

Bundles must remain byte-identical over time to preserve replay verification.

---

### Proof Ledger

Each notarization event produces a ledger entry.

The ledger records:

- proof identifier
- bundle location
- SHA-256 bundle hash
- timestamp
- service metadata

The ledger allows clients to retrieve proof metadata and audit notarization activity.

---

### Verification System

Verification may occur through:

- the NPPP CLI
- the NPPP API
- independent third-party verifiers

Verification performs the following steps:

1. parse proof string
2. retrieve evidence bundle
3. compute SHA-256 hash
4. compare computed hash with proof hash

If the hashes match:
```
SHA Replay Verified: OK
```

Verification confirms the bundle has not been modified since the proof was created.

---

## Architectural Flow

The full system flow is:
```
Artifact
  ↓
Client (CLI or API)
  ↓
Notarization Service
  ↓
Deterministic Evidence Bundle
  ↓
SHA-256 Integrity Hash
  ↓
NPPP Proof String
  ↓
Evidence Storage
  ↓
Replay Verification
```

This process transforms arbitrary artifacts into reproducible cryptographic evidence.

---

## Architecture Diagram

The system architecture is illustrated in the diagram below.
See:

```
/diagrams/nppp_architecture.mmd
```

The diagram shows the relationships between:

- clients
- notarization services
- storage systems
- verification workflows

---

## Design Principles

NPPP architecture follows several core principles:

### Determinism

Identical evidence inputs must produce identical bundle bytes.

Determinism ensures replay verification succeeds across environments.

---

### Independence of Verification

Verification does not require trust in the original operator.

Any verifier capable of retrieving the evidence bundle can reproduce the integrity check.

---

### Durable Evidence

Evidence bundles must remain accessible and unchanged.

Loss or mutation of the bundle prevents verification.

---

### Protocol Stability

NPPP v1 defines a stable proof format.

Future protocol changes will require version increments.

---

## Deployment Model

The reference implementation is operated by the **UserMint Network**.

A typical deployment may include:

- API gateway
- notarization service
- object storage
- proof metadata database
- verification services

Additional infrastructure such as monitoring and logging may be deployed to support operational reliability.

---

## Summary

The NPPP architecture transforms digital artifacts into replay-verifiable evidence.

Key properties:

- deterministic bundle generation
- cryptographic integrity verification
- portable proof strings
- independent replay verification

These components together provide a foundation for verifiable digital evidence systems.
