NPPP v1 — Notarized Proof of Provenance Protocol



Deterministic proof. Stateless verification. Minimal trust.
What Is NPPP?
NPPP v1 is a stateless integrity protocol for issuing and verifying deterministic cryptographic proofs of data.
It enables any system to:
commit to a canonical bundle of bytes,
produce a deterministic proof string,
verify integrity through recomputation,
validate results without trusting stored verification state.
Canonical Claim
NPPP v1 does not claim:
“this artifact is true”
NPPP v1 claims:
“this exact bundle hashes to this exact SHA-256 value, and that equality can be independently recomputed.”
Protocol Status
Property
Value
Version
1.0.0
State
Launch
Verification
Stateless
Trust Model
Replay-based
Normative Specification
The authoritative definition of NPPP v1 is:
📜 spec/NPPP_v1.md
This specification defines:
proof grammar
field ordering
notarization procedure
verification procedure
result vocabulary
proof-record semantics
If any document conflicts with the spec, the spec wins.
Quick Start (Reference Deployment)
⚠️ This section demonstrates a reference implementation.
Authentication and access control are not part of the protocol.
1. Bootstrap Session
curl -X POST https://api.usermint.network/v1/auth/bootstrap 
  -H "Authorization: Bearer <ACCESS_TOKEN>" 
  -H "Content-Type: application/json" 
  -d '{"email": "you@example.com"}'
Returns:
{
  "api_key": "um_...",
  "credits": 20
}
2. Notarize
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
Returns:
proof_id
proof
bundle
bundle_sha256
3. Verify (Stateless Replay)
curl -X POST https://api.usermint.network/v1/verify 
  -H "Authorization: Bearer <ACCESS_TOKEN>" 
  -H "Content-Type: application/json" 
  -d '{"proof":"NPPP:V1|..."}'
Returns:
{
  "verified": true,
  "status": "SHA Replay Verified: OK"
}
4. Retrieve Proof Record
GET /v1/proof/{proof_id}
Example field:
"verification": {
  "status": "unverified",
  "model": "stateless"
}
This is expected.
It reflects:
verification is not persisted as truth
verification must be recomputed
How NPPP Works
Protocol Flow
Submit notarization request
Construct deterministic bundle
Compute SHA-256
Issue canonical proof string
Replay verification by recomputation
Proof Format
NPPP:V1|
project=...|
region=...|
service=...|
freshness=...|
bundle=...|
sha256=...|
created=...
fixed field order
deterministic structure
replay-verifiable
Key Concepts
Stateless Verification
Verification is not stored.
It is derived by recomputing:
SHA256(bundle_bytes) == proof.sha256
Protocol Independence
NPPP v1 is:
cloud-agnostic
identity-agnostic
transport-neutral
Authentication, hosting, and access control are deployment concerns, not protocol requirements.
Documentation
Document
Description
📜 spec/NPPP_v1.md
Normative protocol definition
⚙️ docs/runbook.md
Reference deployment walkthrough
🔐 docs/trust-model.md
Trust assumptions
🧠 docs/adversary-model.md
Threat model
🔑 docs/auth-model.md
Deployment access model
📄 openapi/openapi.json
Machine-readable API surface
Philosophy
If a system can recompute the hash, the proof holds.
Contributing
Contributions are welcome.
Follow .github/pull_request_template.md
Open an issue for major changes
Do not modify protocol semantics without versioning
License
MIT License — see LICENSE.