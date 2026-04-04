# NPPP v1 — Notarized Proof of Provenance Protocol

**Deterministic proof. Stateless verification. Minimal trust.**

## Status

| Property | Value |
|---|---|
| Version | 1.0.0 |
| State | Stable (Normative Specification Defined) |

The canonical protocol definition is located at:

- `spec/NPPP_v1.md`

If any document conflicts with the specification, the specification is authoritative.

## What Is NPPP?

NPPP v1 is a stateless cryptographic integrity protocol.

It allows any system to:

- construct a deterministic evidence bundle
- compute a SHA-256 commitment
- issue a canonical proof string
- verify integrity via replay (recomputation)

## Canonical Claim

NPPP does not claim:

> “this artifact is true”

NPPP claims:

> “this exact bundle hashes to this exact SHA-256 value, and that equality can be independently recomputed.”

## Core Properties

| Property | Description |
|---|---|
| Deterministic | Same input → same proof |
| Stateless | No stored verification truth |
| Replayable | Verification via recomputation |
| Minimal Trust | No reliance on mutable flags |

## Non-Goals

NPPP v1 does not provide:

- authorship guarantees
- identity verification
- semantic correctness
- timestamp authority
- decentralized consensus

## Protocol Overview

### Flow

1. Submit notarization request
2. Construct deterministic bundle
3. Compute SHA-256
4. Issue canonical proof string
5. Persist proof record
6. Verify via replay (recompute hash)

### Proof Format

```text
NPPP:V1|
project={project}|
region={region}|
service={service}|
freshness={freshness}|
bundle={bundle_uri}|
sha256={bundle_sha256}|
created={created_at}
```

**Requirements:**

- fixed field order (strict)
- pipe-delimited (|)
- no empty values
- SHA-256 must be 64 lowercase hex characters

## Evidence Bundle

The bundle is the source of truth for verification.

**Requirements:**

- deterministic serialization (UTF-8)
- stable structure and ordering
- immutable after proof issuance

**Example URI:**

`gs://usermint-network-notary-dev/bundles/example_bundle.tar.gz`

**Verification checks:**

`sha256(bundle_bytes) == proof.sha256`

## Verification Model

Verification is:

- stateless
- replay-based
- recomputed on demand

**Result Vocabulary:**

- `SHA Replay Verified: OK`
- `SHA Replay Verified: FAILED`

## Proof Record Semantics

A proof record MUST include:

- `proof_id`
- `proof`
- `bundle`
- `bundle_sha256`
- `created_at`
- `service`
- `artifact_type`
- `freshness`
- `verification`

**Required verification object:**

```json
{
  "status": "unverified",
  "model": "stateless"
}
```

This reflects the verification model, not stored truth.

## Reference HTTP Implementation

> **Important:** NPPP is transport-agnostic. The following API is a reference implementation, not part of the protocol.

**Base URL:** `https://api.usermintnetwork.com`

### System Endpoints

**Health:**
```bash
GET /system/health
```

**Liveness:**
```bash
GET /system/livez
```

### 1. Bootstrap (Implementation-Specific)

```bash
POST /v1/auth/bootstrap
```

```json
{
  "email": "developer@example.com"
}
```

**Returns:**

```json
{
  "user": {
    "id": "user_123",
    "email": "developer@example.com",
    "stripe_customer_id": null,
    "created_at": "2026-04-02T00:00:00Z"
  },
  "api_key": "usermint_sk_..."
}
```

### 2. Notarize

```bash
POST /v1/notarize
```

```json
{
  "service": "example-service",
  "artifact_type": "json",
  "freshness": "now",
  "evidence": {
    "path": "bundle/example.json"
  }
}
```

**Response:**

```json
{
  "proof_id": "nppp_abc123",
  "proof": "NPPP:V1|project=usermint-network|region=us-central1|service=example-service|freshness=now|bundle=gs://...|sha256=...|created=...",
  "bundle": "gs://...",
  "bundle_sha256": "10fa833533b76d3dcc74b06a07cdf44a778ef799fdfa34b87bc99b2500e7f148",
  "created_at": "2026-03-28T05:58:21Z"
}
```

### 3. Verify (Stateless Replay)

```bash
POST /v1/verify
```

```json
{
  "proof": "NPPP:V1|..."
}
```

**Response:**

```json
{
  "verified": true,
  "expected_sha256": "...",
  "computed_sha256": "...",
  "bundle": "gs://...",
  "status": "SHA Replay Verified: OK"
}
```

### 4. Retrieve Proof Record

```bash
GET /v1/proof/{proof_id}
```

**Example:**

```json
{
  "proof_id": "nppp_abc123",
  "proof": "NPPP:V1|...",
  "bundle": "gs://...",
  "bundle_sha256": "...",
  "created_at": "2026-03-28T05:58:21Z",
  "service": "example-service",
  "artifact_type": "json",
  "freshness": "now",
  "verification": {
    "status": "unverified",
    "model": "stateless"
  }
}
```

## Important Distinction

| Layer | Responsibility |
|---|---|
| NPPP v1 Spec | Defines proof + verification semantics |
| OpenAPI | Defines HTTP contract |
| Implementation (usermint-api) | Handles auth, billing, routing |

## Documentation

- 📜 `spec/NPPP_v1.md` — Normative protocol definition
- 📦 `docs/bundle-structure.md` — Canonical bundle design
- 📏 `docs/canonicalization-rules.md` — Deterministic encoding
- ⚙️ `docs/runbook.md` — Implementation walkthrough
- 🔐 `docs/trust-model.md` — Trust boundaries
- 🧠 `docs/adversary-model.md` — Threat analysis
- 📄 `openapi/openapi.json` — Reference HTTP contract

## Philosophy

> If a system can recompute the hash, the proof holds.

## Contributing

- Follow `.github/pull_request_template.md`
- Open an issue for major changes
- Do not modify protocol semantics without versioning

## License

MIT License — see `LICENSE`
