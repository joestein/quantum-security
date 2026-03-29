# quantum-secure Plugin v0.1.0

**Post-quantum cryptographic migration for any codebase.**

`quantum-secure` is a Claude Code plugin that scans your codebase for classical cryptographic primitives vulnerable to quantum attack, migrates them to NIST post-quantum standards, and validates the results.

## The Threat is Real -- Act Now

Adversaries are already executing "Harvest Now, Decrypt Later" (HNDL) attacks -- collecting encrypted data today to decrypt once cryptographically-relevant quantum computers (CRQCs) arrive. NIST finalized three post-quantum cryptography standards in August 2024. The migration window is open. Every day you wait is another day of vulnerable data in transit.

## Quick Start

```
/quantum-secure full
```

That's it. The plugin will scan your entire codebase, migrate classical crypto to quantum-secure alternatives, and validate the changes.

## Commands

| Command | Description |
|---------|-------------|
| `/quantum-secure` | Run the full pipeline (scan + migrate + validate) |
| `/quantum-secure scan` | Scan only -- find classical crypto, output a report |
| `/quantum-secure migrate` | Migrate classical crypto to PQC alternatives |
| `/quantum-secure validate` | Validate migration completeness and run tests |
| `/quantum-secure full` | Explicit full pipeline |
| `/quantum-secure scan [path]` | Scan a specific directory or file |

## NIST Post-Quantum Standards

| Standard | Algorithm | Replaces | Use Case |
|----------|-----------|----------|----------|
| FIPS 203 | **ML-KEM** (Kyber) | RSA-KEM, ECDH, DH, X25519 | Key encapsulation / exchange |
| FIPS 204 | **ML-DSA** (Dilithium) | RSA sigs, ECDSA, EdDSA, DSA | Digital signatures (default) |
| FIPS 205 | **SLH-DSA** (SPHINCS+) | Same as ML-DSA | Signatures (conservative fallback) |

## Architecture

The plugin orchestrates three specialized agents:

### Scanner Agent (read-only)
- Detects classical crypto across 10+ programming languages
- Identifies imports, function calls, configuration strings, and constants
- Classifies findings by category (key exchange, signature, symmetric, hash, protocol)
- Assigns risk levels and recommends specific PQC replacements
- **Safe to run on any codebase** -- makes no modifications

### Migrator Agent (writes code)
- Rewrites classical crypto to use PQC libraries
- Checks language-specific library availability before choosing an approach
- Uses existing PQC libraries when available (liboqs, Bouncy Castle, pqcrypto, circl, etc.)
- Writes FFI bindings to liboqs for languages without native PQC libraries
- Supports hybrid mode (classical + PQC) for TLS/protocol contexts
- Adds dependencies to package managers automatically
- Preserves existing code style and conventions

### Validator Agent (read-only + test execution)
- Re-scans to verify migration completeness
- Checks dependency resolution (dry-run installs)
- Runs existing test suites
- Flags custom implementations for third-party security audit
- Produces a pass/fail report with migration completion percentage

## Supported Languages

| Language | PQC Library Support | Custom Implementation Strategy |
|----------|-------------------|-------------------------------|
| C/C++ | liboqs (production-ready) | N/A -- reference implementations available |
| Rust | pqcrypto crate | N/A -- mature wrappers available |
| Python | oqs-python | N/A -- FFI bindings available |
| Go | cloudflare/circl | liboqs-go for SLH-DSA |
| Java | Bouncy Castle | N/A -- full FIPS support |
| C# | BouncyCastle.Crypto | N/A -- full FIPS support |
| JavaScript/TypeScript | crystals-kyber (npm) | WASM liboqs bindings for ML-DSA/SLH-DSA |
| Ruby | -- | FFI bindings to liboqs |
| PHP | -- | FFI bindings to liboqs |

## Limitations (v0.1.0)

- Custom FFI implementations for Ruby/PHP require liboqs to be installed on the system
- WASM bindings for JavaScript PQC support are experimental
- Scanner may produce false positives on crypto-related comments or documentation strings
- Hybrid mode is automatic for TLS/protocol contexts but must be explicitly requested for application-layer crypto
- Custom implementations are flagged for audit -- they should not be deployed to production without third-party review

## License

Apache 2.0 -- See [LICENSE](../../LICENSE)
