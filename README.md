# NPPP v1 — Notarization Proof Packet Protocol

**Deterministic proof. Verifiable integrity. Minimal trust.**

NPPP (Notarization Proof Packet Protocol) is a lightweight protocol for generating, storing, and verifying cryptographic proof of data integrity.

---

## What NPPP Does

- Submit data for notarization  
- Receive a deterministic proof string  
- Retrieve proof metadata  
- Replay verification at any time  

---

## Base URL
https://api.usermint.network⁠

---

## Authentication
Authorization: Bearer <TOKEN>

---

## Core Endpoints

- `POST /v1/notarize`  
- `POST /v1/verify`  
- `GET /v1/proof/{proof_id}`  

---

## Quick Example

### Notarize

```json
{
  "service": "genesis-test",
  "artifact_type": "dataset",
  "freshness": "7d",
  "evidence": {
    "path": "."
  }
}
```

### Verify

```json
{
  "proof": "NPPP:V1|project=...|..."
}
```

### Proof Lookup

`GET /v1/proof/{proof_id}`

---

## Typical Flow

1.  **Submit** → `/v1/notarize`
2.  **Receive proof**
3.  **Verify** → `/v1/verify`
4.  **Lookup** → `/v1/proof/{proof_id}`

---

## Philosophy

If a system can deterministically reconstruct the input and reproduce the hash, the proof holds.

---

## Status

**NPPP v1 — Pre-Launch**
