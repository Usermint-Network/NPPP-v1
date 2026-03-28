# Contributing to NPPP

Thank you for your interest in contributing to the Notarization Proof Packet Protocol (NPPP).

We welcome improvements to documentation, examples, and reference implementations.

---

## Contribution Areas

Contributions are encouraged in the following areas:

- protocol documentation
- developer tooling
- verification tooling
- example integrations
- security analysis
- test vectors

---

## Pull Requests

When submitting a pull request:

1. fork the repository
2. create a feature branch
3. make focused changes
4. submit a pull request with a clear description

Small, well-scoped contributions are preferred.

---

## Protocol Changes

All changes to the protocol specification are subject to strict guidelines.

### Core Principles
- **Versioning**: Any change to the specification that alters behavior, adds fields, or changes semantics **must** be accompanied by a version bump.
- **Canonical Source**: The normative specification is `spec/NPPP_v1.md`. Implementation-specific documentation, diagrams, or examples must not contradict or override the canonical spec. If there is a conflict, the spec is the source of truth.

### Process
1.  Open an issue discussing the proposed change.
2.  Reference the impact on `spec/NPPP_v1.md`.
3.  All protocol changes will be debated and must be approved by the core maintainers.

---

## Code of Conduct

All contributors are expected to maintain a respectful and professional tone in discussions and reviews.

---

## Security Issues

If you discover a security issue related to the protocol or reference implementation, please report it privately according to:
SECURITY.md

---

## License

By contributing to this repository, you agree that your contributions will be licensed under the MIT License.
