# NPPP v1 — Notarized Proof of Provenance Protocol

[![Protocol Version](https://img.shields.io/badge/protocol-v1.0.0-blue.svg)](spec/NPPP_v1.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Project Status](https://img.shields.io/badge/status-launch-brightgreen.svg)](#protocol-status)

Deterministic proof. Stateless verification. Minimal trust.

NPPP v1 is a protocol for issuing and verifying cryptographic proofs of data integrity through deterministic hashing and replayable verification.

---

## Overview

NPPP enables any system to:

- Generate a deterministic proof of an artifact.
- Store an immutable attestation record.
- Verify integrity through recomputation.
- Independently validate results without trusting stored state.

## Core Principle

NPPP does not store truth. It proves that a specific bundle of bytes hashes to a specific SHA-256 value and that this can be recomputed at any time.

## Protocol Status

| Property           | Value          |
| ------------------ | -------------- |
| Version            | 1.0.0          |
| State              | Launch         |
| Verification Model | Stateless      |
| Trust Model        | Replay-based   |

---

## Prerequisites

Before you begin, ensure you have the following:

- **`curl`**: A command-line tool for making HTTP requests.
- **Access Token**: You must have a valid access token. For the purposes of this guide, replace `<ACCESS_TOKEN>` with your token. (*Note: A section on how to acquire a token would typically be included here.*)

---

## Quick Start

This guide provides runnable `curl` examples to walk you through notarizing an artifact and verifying the proof.

### 1. Bootstrap a Developer Session

First, bootstrap a session to authenticate with the system. Replace `<ACCESS_TOKEN>` with your token.

```bash
curl -X POST https://api.usermint.network/v1/auth/bootstrap \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com"}'
```

### 2. Notarize an Artifact

Next, submit a request to notarize a digital artifact. This example sends the content directly in the request. Replace `<ACCESS_TOKEN>` and `<API_KEY>` with your credentials.

```bash
curl -X POST https://api.usermint.network/v1/notarize \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "X-API-Key: <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "service": "demo",
    "artifact_type": "json",
    "freshness": "now",
    "evidence": {
      "content": "{\"message\": \"Hello, NPPP!\"}"
    }
  }'
```

### 3. Verify a Proof

Once you have a proof string from the notarization step, you can verify its integrity. Replace `<ACCESS_TOKEN>` and the example proof string with your own.

```bash
curl -X POST https://api.usermint.network/v1/verify \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"proof":"NPPP:V1|project=usermint-network|region=us-central1|service=demo|freshness=now|bundle=gs://...|sha256=...|created=2026-03-28T05:58:21Z"}'
```

---

## How It Works

### Protocol Flow

The protocol follows a five-step process:
1.  **Bootstrap Session**: Authenticate and establish a developer session.
2.  **Submit Notarization Request**: Send the artifact to be notarized.
3.  **Receive Deterministic Proof**: The system returns a unique, deterministic proof string.
4.  **Replay Verification**: Independently recompute the proof to verify its integrity.
5.  **Retrieve Immutable Record**: Fetch the permanent proof record from the system.

### Example Proof String

An NPPP proof is a structured string containing all metadata required for verification.

`NPPP:V1|project=usermint-network|region=us-central1|service=demo|freshness=now|bundle=gs://...|sha256=...|created=2026-03-28T05:58:21Z`

### Core API Endpoints

The NPPP v1 protocol is exposed through the following core endpoints:

- `POST /v1/auth/bootstrap`
- `GET  /v1/credits/balance`
- `POST /v1/notarize`
- `POST /v1/verify`
- `GET  /v1/proof/{proof_id}`

---

## Key Concepts

### Stateless Verification

Verification is performed by parsing the proof, fetching the original data bundle, recomputing its SHA-256 hash, and comparing it against the commitment in the proof. No stored state is trusted, ensuring that verification is always deterministic and independent.

### Protocol Independence

NPPP v1 is designed to be agnostic to the underlying infrastructure:
- **Cloud-agnostic**: Works with any cloud provider.
- **Identity-agnostic**: Integrates with any identity system.
- **Transport-neutral**: Can be used over any network transport.

Access control and authentication are considered implementation-specific details that are layered on top of the core protocol.

---

## Documentation

For more detailed information, please refer to the following documents:

- 📜 **Protocol Specification**: `spec/NPPP_v1.md`
- ⚙️ **Runbook**: `docs/runbook.md`
- 🔐 **Trust Model**: `docs/trust-model.md`
- 🧠 **Adversary Model**: `docs/adversary-model.md`
- 🔑 **Authentication Model**: `docs/auth-model.md`
- 📄 **OpenAPI Schema**: `openapi/openapi.json`

## Contributing

Contributions are welcome! We follow a standard pull request process. Please see the `.github/pull_request_template.md` for guidelines. For major changes, please open an issue first to discuss what you would like to change.

## Support

If you have questions, find a bug, or need assistance, please [open an issue](https://github.com/Usermint-Network/NPPP-v1/issues) in this repository.

## Philosophy

> If a system can recompute the hash, the proof holds.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
