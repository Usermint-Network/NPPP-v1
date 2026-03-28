# NPPP Canonicalization Rules

This document defines the canonicalization principles for **NPPP v1**.

Canonicalization is the process of transforming notarization inputs into a stable representation so that identical logical artifacts produce identical evidence bundles.

---

## Purpose

Canonicalization exists to guarantee determinism.

Without canonicalization, semantically identical artifacts may produce different bytes, causing verification failures even when the underlying evidence has not meaningfully changed.

---

## Canonicalization Goal

The goal of canonicalization is:

> identical inputs should produce identical bundle bytes

This is required for replay verification to function reliably.

---

## Canonicalization Scope

Canonicalization applies to any input that influences the final evidence bundle, including:

- files
- metadata
- structured records
- service-generated context
- archive ordering

---

## Core Rules

NPPP v1 canonicalization **MUST** enforce the following principles to guarantee byte-for-byte deterministic output.

### 1. Character Encoding
- **Text:** All textual data, including file contents, metadata, and structured data, **MUST** be encoded as **UTF-8**. No other text encodings are permitted.

### 2. JSON Serialization
- **Deterministic Serialization:** All structured metadata represented as JSON **MUST** be serialized deterministically.
- **Stable Key Ordering:** JSON object keys **MUST** be sorted lexicographically (by Unicode code point) before serialization.
- **Whitespace Rules:** JSON output **MUST** be compact, with no insignificant whitespace (e.g., newlines or spaces between keys, values, and separators). The output should be a single line.
- **Example:** A JSON object `{"c": 3, "a": 1, "b": 2}` MUST be serialized as `{"a":1,"b":2,"c":3}`.

### 3. File and Directory Ordering
- **Stable File Ordering:** Files included in an evidence bundle **MUST** be ordered deterministically. The required method is a simple lexicographical sort of their full path strings.
- **Stable Directory Layout:** The directory structure within any archive (e.g., `.tar.gz`) **MUST** be consistent across all runs for the same logical input.

### 4. Timestamps
- **Intentional Timestamps Only:** Only timestamps that are an explicit part of the notarized evidence may be included. Incidental timestamps (e.g., file access times, archive creation times) **MUST NOT** be included in the data used to generate the bundle hash.

### 5. Archive Generation
- **Deterministic Archives:** If an archive format (like TAR) is used to create the bundle, its generation **MUST** be deterministic. This means identical inputs must produce identical archive bytes. Tools and flags should be chosen to support this (e.g., `tar --sort=name`).

---

## Non-Canonical Sources of Variance
Implementations SHOULD eliminate or normalize sources of accidental variation such as:
- filesystem-dependent ordering
- unstable metadata ordering
- environment-specific line endings
- runtime-generated temporary values
- archive timestamps not intended as notarized evidence

---

## Canonicalization of Structured Data
For structured data formats, implementations SHOULD ensure consistent serialization.
Examples of consistency include:
- stable object key ordering
- normalized whitespace
- consistent numeric representation where relevant
The exact encoding strategy may vary by implementation, but the result MUST remain deterministic.

---

## Canonicalization of Binary Artifacts
Binary artifacts SHOULD be included as-is unless the protocol or application explicitly defines a canonical binary normalization process.
The key requirement is that the bytes included in the bundle remain stable and intentional.

---

## Relationship to Bundling
Canonicalization precedes hashing.
The canonicalized inputs are assembled into the evidence bundle, and the final serialized bundle is hashed using SHA-256.
If canonicalization is inconsistent, bundle hashes will diverge.

---

## Failure Mode
If identical logical artifacts do not produce identical bundle bytes, then:
- the resulting proof hashes may differ
- replay verification may fail unexpectedly
- the notarization process loses determinism

For this reason, canonicalization is not optional in NPPP v1 systems.

---

## Canonicalization and Trust
Canonicalization does not make evidence true.
It makes evidence stable.
This distinction is critical.
NPPP relies on canonicalization to ensure that cryptographic verification is meaningful across repeated verification attempts.

---

## Versioning
These rules define canonicalization expectations for NPPP v1.
Future protocol versions may introduce stricter normalization requirements, artifact-specific canonical forms, or mandatory serialization standards.
