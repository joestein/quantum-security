# Post-Quantum Migration Guide

A step-by-step guide to migrating your codebase from classical cryptography to NIST post-quantum standards using the `quantum-secure` plugin.

---

## Prerequisites

- **Claude Code** installed and configured
- **quantum-secure plugin** installed from the quantum-security marketplace
- A codebase with classical cryptographic usage (RSA, ECDSA, ECDH, DH, etc.)

### Installing the Plugin

Add the quantum-security marketplace to your Claude Code configuration:

```json
{
  "extraKnownMarketplaces": {
    "quantum-security": {
      "source": {
        "source": "github",
        "repo": "joestein/quantum-security"
      }
    }
  }
}
```

Then install the plugin:
```
/install quantum-secure@quantum-security
```

---

## Step 1: Scan Your Codebase

Run the scanner to understand your quantum exposure:

```
/quantum-secure scan
```

Or scan a specific directory:

```
/quantum-secure scan src/
```

### What the Scanner Does

The scanner is **completely read-only** -- it makes no modifications. It:

1. Detects which programming languages are in your codebase
2. Searches for classical crypto patterns: imports, function calls, config strings, constants
3. Classifies each finding by risk level:
   - **CRITICAL**: Asymmetric crypto broken by Shor's algorithm (RSA, ECDSA, ECDH, DH, DSA, EdDSA)
   - **HIGH**: Weak symmetric/hash weakened by Grover's algorithm (AES-128, DES, 3DES, MD5, SHA-1)
   - **MEDIUM**: Protocol/configuration level (TLS cipher suites, SSH config, certificates)
4. Recommends the specific PQC replacement for each finding
5. Checks for existing PQC usage (already-migrated code)
6. Produces a structured vulnerability report

### Reading the Scan Report

The report includes:

- **Executive Summary**: Overall quantum security posture in 2-3 sentences
- **Findings Table**: Every instance of classical crypto with file, line, algorithm, risk, and recommended replacement
- **Dependency Analysis**: Which package manager dependencies carry classical crypto
- **Existing PQC Usage**: Any post-quantum crypto already in the codebase
- **Recommendations**: Prioritized list of migration actions

### What to Look For

- **High finding count in CRITICAL category**: Your codebase is significantly exposed to quantum attack
- **Key exchange findings**: These are the highest priority -- HNDL attacks target encrypted communications
- **Signature findings**: Important for long-lived certificates and code signing
- **Multiple languages**: Each language may need different PQC libraries

---

## Step 2: Review the Report

Before migrating, review the scan report and consider:

1. **Which findings are highest priority?** Focus on CRITICAL findings first, especially key exchange used in data-in-transit encryption.

2. **Are there hybrid setups already?** If classical + PQC crypto coexist, the scanner notes this. Don't disrupt working hybrid configurations.

3. **What languages are involved?** Check the [Language Support](language-support.md) document to understand library availability for your languages.

4. **Are there custom or unusual crypto patterns?** The migrator handles standard patterns well. Unusual setups may need manual attention.

5. **Do you want hybrid mode?** For TLS/protocol contexts, hybrid is the default. For application-layer crypto, you can choose pure PQC or hybrid.

---

## Step 3: Run the Migration

Once you've reviewed the scan report:

```
/quantum-secure migrate
```

Or run the full pipeline (scan + migrate + validate) in one command:

```
/quantum-secure full
```

### What the Migrator Does

For each finding from the scan report:

1. **Checks library availability**: Consults the PQC library matrix for the target language
2. **Selects the replacement strategy**:
   - Mature PQC library exists → use it
   - No library but C FFI available → write FFI bindings to liboqs
   - No FFI → WASM binding or pure implementation (flagged for audit)
3. **Rewrites the code**:
   - Replaces import statements
   - Rewrites function calls to use PQC APIs
   - Updates type annotations and interfaces
   - Adjusts buffer sizes for larger PQC keys/signatures
4. **Updates dependencies**: Adds PQC libraries to package.json, requirements.txt, Cargo.toml, etc.
5. **Handles edge cases**:
   - TLS/protocol contexts → hybrid mode (classical + PQC)
   - JWT signing → flags for IETF JOSE PQC spec (deferred)
   - Database column sizes → flags for manual review
   - Fixed-size protocol fields → flags for manual review

### Migration Comments

The migrator adds concise comments to modified code:

```python
# QUANTUM-SECURE: Migrated from RSA-2048 to ML-DSA-65 (FIPS 204)
```

```javascript
// QUANTUM-SECURE: Migrated from ECDH P-256 to ML-KEM-768 (FIPS 203)
```

### Custom Implementations

For languages without PQC libraries (Ruby, PHP, etc.), the migrator writes FFI bindings to liboqs. These are marked:

```ruby
# QUANTUM-SECURE: Custom PQC implementation - requires third-party audit
# FFI binding to liboqs for ML-KEM-768. Ensure liboqs is installed on the system.
```

**These custom implementations MUST be reviewed by a security professional before production deployment.**

---

## Step 4: Run Validation

After migration, validate the results:

```
/quantum-secure validate
```

### What the Validator Checks

1. **Completeness**: Re-scans for remaining classical crypto. Every finding should be either migrated, in hybrid mode, or deferred with justification.

2. **Dependencies**: Verifies all new PQC libraries resolve correctly.

3. **Build**: Attempts to compile/build the project to catch errors introduced by migration.

4. **Tests**: Runs existing test suites to verify functionality is preserved.

5. **Security sanity**: Checks for common mistakes:
   - Hardcoded keys
   - Non-cryptographic RNG used for key generation
   - Key material logged
   - Improper key storage

6. **Custom implementation flags**: Every custom FFI/WASM/pure implementation is flagged for mandatory third-party audit.

### Reading the Validation Report

The report gives an overall **PASS**, **FAIL**, or **PASS WITH WARNINGS** status:

- **PASS**: All findings migrated, dependencies resolve, build succeeds, tests pass
- **PASS WITH WARNINGS**: Migration complete but some items need attention (custom implementations, deferred items)
- **FAIL**: Some findings were missed, build is broken, or tests fail

---

## Step 5: Manual Review

After automated validation, manually review:

1. **Custom implementations**: Any FFI bindings or WASM shims written by the migrator
2. **Deferred items**: JWT migrations, protocol-level changes awaiting spec finalization
3. **Size impact**: PQC keys and signatures are larger -- verify storage and bandwidth can handle them
4. **Performance**: Run benchmarks if crypto is in a hot path
5. **Interoperability**: Verify PQC-migrated services can communicate with their peers

---

## Common Scenarios

### Scenario: Python Web API with JWT Authentication

```
Scan findings:
- RSA-2048 for JWT signing (CRITICAL)
- ECDH P-256 for TLS (CRITICAL, protocol config)
- SHA-1 in legacy endpoint (HIGH)

Migration result:
- JWT: Deferred (awaiting IETF JOSE PQC spec), flagged with TODO
- TLS: Hybrid X25519 + ML-KEM-768 configuration
- SHA-1: Replaced with SHA-256
```

### Scenario: Go Microservice with gRPC

```
Scan findings:
- ECDSA P-256 for mTLS certificates (CRITICAL)
- X25519 for key agreement (CRITICAL)
- AES-128-GCM for payload encryption (HIGH)

Migration result:
- mTLS: Hybrid ECDSA + ML-DSA-65 certificates
- Key agreement: ML-KEM-768 using cloudflare/circl
- AES: Upgraded to AES-256-GCM
```

### Scenario: Multi-Language Monorepo (Python + TypeScript + Rust)

```
Scan findings:
- Python: RSA signatures, ECDH key exchange
- TypeScript: RSA JWT signing, AES-128
- Rust: ed25519-dalek for signing, x25519-dalek for key exchange

Migration result:
- Python: oqs-python for ML-DSA-65 and ML-KEM-768
- TypeScript: crystals-kyber for ML-KEM, WASM liboqs for ML-DSA
- Rust: pqcrypto crate for ML-DSA and ML-KEM
- All dependency files updated (requirements.txt, package.json, Cargo.toml)
```

---

## Troubleshooting

### Build failures after migration

- **Missing dependency**: Run the appropriate install command (npm install, pip install, cargo build)
- **Type mismatch**: PQC key types differ from classical. Check that all type annotations were updated.
- **Buffer size**: PQC keys/signatures are larger. Check any hardcoded buffer sizes.

### Tests failing after migration

- **Hardcoded test vectors**: Tests with hardcoded RSA/ECDSA key material need updated PQC key material
- **Size assertions**: Tests asserting key/signature sizes will fail with PQC
- **Mock crypto**: Mocked crypto operations may need updating to match PQC APIs

### liboqs not found (FFI bindings)

For Ruby, PHP, or other languages using FFI to liboqs:
1. Install liboqs: `apt install liboqs-dev` or build from source
2. Ensure the shared library is in the library path: `export LD_LIBRARY_PATH=/usr/local/lib`
3. Verify: `ls /usr/local/lib/liboqs.*`

### Performance regression

PQC algorithms have different performance profiles:
- ML-KEM is typically faster than RSA key exchange
- ML-DSA signing/verification is fast but signatures are 50x larger than ECDSA
- SLH-DSA is significantly slower than ML-DSA -- only use when conservative security is required
- If performance is critical, benchmark before and after migration
