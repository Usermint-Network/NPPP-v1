# NPPP v1 — Notarized Proof of Provenance Protocol

**Deterministic proof. Stateless verification. Minimal trust.**

## Latest Release

NPPP v1.0.0 is available:

https://github.com/Usermint-Network/NPPP-v1/releases/tag/v1.0.0

## What Is NPPP?

NPPP v1 is a stateless integrity protocol for issuing and verifying deterministic cryptographic proofs of data.

It enables any system to:

- commit to a canonical bundle of bytes
- produce a deterministic proof string
- verify integrity through recomputation
- validate results without trusting stored verification state

## Canonical Claim

NPPP v1 does not claim:

> “this artifact is true”

NPPP v1 claims:

> “this exact bundle hashes to this exact SHA-256 value, and that equality can be independently recomputed.”

## Implementation Boundary

The NPPP protocol defines proof structure and verification semantics.

Economic behavior, including pricing, billing, quotas, and access policies, is defined entirely by the implementing operator.

Multiple independent implementations may adopt different economic models while remaining fully conformant with NPPP v1.

## Protocol Status

| Property | Value |
|---|---|
| Version | 1.0.0 |
| State | Launch |
| Verification | Stateless |
| Trust Model | Replay-based |

## Normative Specification

The normative protocol definition remains:

- `spec/NPPP_v1.md`

This specification defines:

- proof grammar
- field ordering
- notarization procedure
- verification procedure
- result vocabulary
- proof-record semantics

If any document conflicts with the spec, the spec wins.

## Quick Start (Reference Deployment)

⚠️ This section demonstrates a reference implementation. Authentication and access control are not part of the protocol.

### 1. Bootstrap Session

```bash
curl -X POST https://api.usermint.network/v1/auth/bootstrap 
  -H "Authorization: Bearer <ACCESS_TOKEN>" 
  -H "Content-Type: application/json" 
  -d '{"email": "you@example.com"}'
```

Returns:

```json
{
  "api_key": "um_...",
  "credits": 20
}
```

### 2. Notarize

```bash
curl -X POST https://api.usermint.network/v1/notarize 
  -H "Authorization: Bearer <ACCESS_TOKEN>" 
  -H "X-API-Key: <API_KEY>" 
  -H "Content-Type: application/json" 
  -d '{
    "service": "demo",
    "artifact_type": "json",
    "freshness": "now",
    "evidence": {
      "path": "demo.json"
    }
  }'
```

Returns:

```text
proof_id
proof
bundle
bundle_sha256
```

### 3. Verify (Stateless Replay)

```bash
curl -X POST https://api.usermint.network/v1/verify 
  -H "Authorization: Bearer <ACCESS_TOKEN>" 
  -H "Content-Type: application/json" 
  -d '{"proof":"NPPP:V1|..."}'
```

Returns:

```json
{
  "verified": true,
  "status": "SHA Replay Verified: OK"
}
```

### 4. Retrieve Proof Record

```text
GET /v1/proof/{proof_id}
```

Example field:

```json
"verification": {
  "status": "unverified",
  "model": "stateless"
}
```

This is expected.
It reflects:
- verification is not persisted as truth
- verification must be recomputed

## How NPPP Works

### Protocol Flow

- Submit notarization request
- Construct deterministic bundle
- Compute SHA-256
- Issue canonical proof string
- Replay verification by recomputation

### Proof Format

```text
NPPP:V1|
project=...|
region=...|
service=...|
freshness=...|
bundle=...|
sha256=...|
created=...
```

- fixed field order
- deterministic structure
- replay-verifiable

## Key Concepts

### Stateless Verification

Verification of a proof is defined as a stateless recomputation process.
Verification does not rely on stored verification state and is derived solely from:
- the proof string
- the referenced bundle
- deterministic recomputation

### Protocol Independence

NPPP v1 is:

- cloud-agnostic
- identity-agnostic
- transport-neutral

Authentication, hosting, and access control are deployment concerns, not protocol requirements.

## Documentation

- 📜 `spec/NPPP_v1.md` — Normative protocol definition
- ⚙️ `docs/runbook.md` — Reference deployment walkthrough
- 🔐 `docs/trust-model.md` — Trust assumptions
- 🧠 `docs/adversary-model.md` — Threat model
- 🔑 `docs/auth-model.md` — Deployment access model
- 💰 `docs/economic-model.md` — Reference implementation economic model
- 📄 `openapi/openapi.json` — Machine-readable API surface

## Philosophy

> If a system can recompute the hash, the proof holds.

## Contributing

- Contributions are welcome.
- Follow `.github/pull_request_template.md`
- Open an issue for major changes
- Do not modify protocol semantics without versioning

## License

MIT License — see `LICENSE`.
