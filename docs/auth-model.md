# NPPP v1 Reference Deployment Access Model

## Purpose

NPPP v1 does not define a canonical authentication mechanism.

This document describes the access and metering model used by the current reference deployment.

This document is non-normative. Authentication and access control are deployment-specific concerns, not protocol requirements.

See `spec/NPPP_v1.md` for the normative protocol definition.

---

## Overview

The current reference deployment uses two distinct layers:

1.  an outer access-control layer to reach the service,
2.  an application-level metering key for selected operations.

These layers solve different problems and must not be conflated.

---

## Layer 1 — Outer Access Layer

The current deployment applies platform-level access control before the request reaches the NPPP application.

Example header:

```http
Authorization: Bearer <PLATFORM_ACCESS_TOKEN>
```

This header is:

- deployment-specific,
- implementation-specific,
- not part of the NPPP v1 protocol.

Its purpose is to determine whether a caller may reach the service at all.

## Layer 2 — Metering Layer

The current deployment uses an application-issued API key for metered developer operations.

Header:

```http
X-API-Key: <USERMINT_API_KEY>
```

This header is also:

- deployment-specific,
- implementation-specific,
- not part of the normative protocol.

Its purpose is to:

- identify a developer session,
- meter compute usage,
- expose credits balance.

## Why These Layers Are Separate

The outer access token answers:

> may this request reach the service boundary?

The application API key answers:

> which developer session is consuming metered operations?

These are distinct concerns and should not share a single semantic meaning.

## Current Reference Deployment Flow

### Bootstrap

The reference deployment exposes a `bootstrap` operation:

- caller presents outer access credential,
- caller submits email,
- service returns an application API key and initial credits.

Example response:

```json
{
  "api_key": "um_...",
  "credits": 20
}
```

### Metered Operations

The current reference deployment expects `X-API-Key` for:

- `POST /v1/notarize`

### Non-Metered Operations

The current reference deployment does not require `X-API-Key` for:

- `POST /v1/verify`
- `GET /v1/proof/{proof_id}`

These operations may still be behind the outer deployment access layer.

## Failure Modes

### Missing Outer Access Credential

The request may be rejected by the deployment platform before it reaches the NPPP application.

### Missing `X-API-Key`

The application may return:

- `401 missing api key`

### Invalid `X-API-Key`

The application may return:

- `403 invalid key`

## Canonical Protocol Note

NPPP v1 itself does not require:

- bearer tokens,
- any specific identity provider,
- any specific API key model,
- any specific metering strategy.

Any conforming implementation may replace the current deployment access model with another access model while still remaining protocol-conformant.

## Summary

The current deployment uses:

- an outer platform access layer,
- an inner application metering layer.

Neither is part of the normative protocol.

They are reference deployment choices.
