# NPPP v1 Reference Deployment Runbook

## Purpose

This runbook proves the end-to-end flow of the current reference deployment.

It demonstrates:

- bootstrap,
- credits retrieval,
- notarization,
- replay verification,
- proof lookup,
- credits decrement.

This document is non-normative. It describes the current live reference deployment rather than the protocol definition itself.

For normative protocol behavior, see `spec/NPPP_v1.md`.

---

## Reference Deployment Base URL

```text
https://api.usermint.network
```

If using the direct service URL for the current deployment, substitute accordingly.

## Deployment Access Model

The reference deployment currently uses:

- an outer access credential,
- an application API key.

These are reference deployment concerns, not protocol requirements.

### Outer Access Header

```http
Authorization: Bearer <PLATFORM_ACCESS_TOKEN>
```

### Metering Header

```http
X-API-Key: <USERMINT_API_KEY>
```

## Step 1 — Acquire Outer Access Credential

Obtain a valid outer access token appropriate for the current deployment environment.

Example shell variable:

```bash
TOKEN="<PLATFORM_ACCESS_TOKEN>"
```

## Step 2 — Bootstrap Developer Session

```bash
BOOTSTRAP_RESP=$(curl -s -X POST 
  -H "Authorization: Bearer $TOKEN" 
  -H "Content-Type: application/json" 
  -d '{"email":"launch@usermintnetwork.com"}' 
  https://api.usermint.network/v1/auth/bootstrap)

echo "$BOOTSTRAP_RESP"
```

**Expected**:

- API key returned
- initial credits returned

## Step 3 — Extract API Key

```bash
API_KEY=$(printf '%s' "$BOOTSTRAP_RESP" | python3 -c 'import sys,json; print(json.load(sys.stdin)["api_key"])')
echo "$API_KEY"
```

## Step 4 — Check Credits Before Notarization

```bash
curl -i 
  -H "Authorization: Bearer $TOKEN" 
  -H "X-API-Key: $API_KEY" 
  https://api.usermint.network/v1/credits/balance
```

**Expected**:

- `HTTP 200`
- positive credits balance

## Step 5 — Submit Notarization Request

```bash
NOTARIZE_RESP=$(curl -s -X POST 
  -H "Authorization: Bearer $TOKEN" 
  -H "X-API-Key: $API_KEY" 
  -H "Content-Type: application/json" 
  -d '{
    "service":"demo",
    "artifact_type":"json",
    "freshness":"now",
    "evidence":{"path":"demo.json"}
  }' 
  https://api.usermint.network/v1/notarize)

echo "$NOTARIZE_RESP"
```

**Expected**:

- `proof_id`
- `proof`
- `bundle`
- `bundle_sha256`
- `created_at`
- developer quota information

## Step 6 — Extract Proof and Proof ID

```bash
PROOF_ID=$(printf '%s' "$NOTARIZE_RESP" | python3 -c 'import sys,json; print(json.load(sys.stdin)["proof_id"])')
PROOF=$(printf '%s' "$NOTARIZE_RESP" | python3 -c 'import sys,json; print(json.load(sys.stdin)["proof"])')

echo "$PROOF_ID"
echo "$PROOF"
```

## Step 7 — Replay Verification

```bash
curl -i -X POST 
  -H "Authorization: Bearer $TOKEN" 
  -H "Content-Type: application/json" 
  -d "{"proof":"$PROOF"}" 
  https://api.usermint.network/v1/verify
```

**Expected**:

- `HTTP 200`
- `verified: true` when bundle bytes match commitment
- `status: SHA Replay Verified: OK`

## Step 8 — Retrieve Proof Record

```bash
curl -i 
  -H "Authorization: Bearer $TOKEN" 
  "https://api.usermint.network/v1/proof/$PROOF_ID"
```

**Expected**:

- `HTTP 200`
- immutable proof record returned
- proof record includes:
  - `verification.status = "unverified"`
  - `verification.model = "stateless"`

This does not contradict successful replay verification. It reflects the stateless verification model.

## Step 9 — Check Credits After Notarization

```bash
curl -i 
  -H "Authorization: Bearer $TOKEN" 
  -H "X-API-Key: $API_KEY" 
  https://api.usermint.network/v1/credits/balance
```

**Expected**:

- `HTTP 200`
- credits reduced appropriately

## Success Criteria

The reference deployment flow is proven when all of the following succeed:

- bootstrap
- credits before
- notarize
- verify
- proof lookup
- credits decrement

## Interpretation

This runbook proves the reference deployment is functioning.

It does not redefine the protocol.

The protocol remains defined by:

- canonical proof grammar,
- deterministic bundle hashing,
- replay verification semantics,
- stateless proof-record interpretation.
