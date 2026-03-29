---
name: quantum-secure
description: Scan and migrate classical cryptography to NIST post-quantum standards (ML-KEM, ML-DSA, SLH-DSA)
argument-hint: <scan|migrate|validate|full> [path]
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, Agent]
user-invocable: true
---

# /quantum-secure -- Post-Quantum Cryptography Migration

Migrate your codebase from classical cryptography to NIST post-quantum standards. Supports any programming language.

## Arguments

The user invoked this with: $ARGUMENTS

Parse the arguments as follows:
- **No arguments** or **`full`**: Run the complete pipeline (scan → migrate → validate)
- **`scan`**: Run only the scanner agent -- find classical crypto, produce a report
- **`scan [path]`**: Scan a specific directory or file
- **`migrate`**: Run only the migrator agent (requires a prior scan report in the conversation)
- **`validate`**: Run only the validator agent (requires a prior migration in the conversation)
- **`full [path]`**: Run the complete pipeline on a specific path

## NIST Post-Quantum Standards

Three algorithms replace all classical public-key cryptography:

| Standard | Algorithm | Replaces |
|----------|-----------|----------|
| **FIPS 203** | ML-KEM | RSA key exchange, ECDH, DH, X25519 |
| **FIPS 204** | ML-DSA | RSA signatures, ECDSA, EdDSA, DSA |
| **FIPS 205** | SLH-DSA | Same as ML-DSA (conservative hash-based fallback) |

Additionally flags AES-128 (→ AES-256), broken hashes (MD5, SHA-1 → SHA-256+), and deprecated ciphers (DES, 3DES, RC4 → AES-256-GCM).

## Execution Protocol

### Mode: `scan`

Launch the **scanner** agent with the following prompt:

> Scan the codebase at `[path or current working directory]` for classical cryptographic usage vulnerable to quantum attacks. Follow your scanning procedure to identify all instances of RSA, ECC, DH, DSA, weak symmetric ciphers, and broken hashes across all programming languages present. Produce a structured Quantum Vulnerability Scan Report.

The scanner agent is **read-only** and safe to run on any codebase. It will:
1. Detect which programming languages are in use
2. Grep for classical crypto patterns (imports, function calls, config strings)
3. Classify each finding by category, risk level, and recommended replacement
4. Produce a structured findings report

After the scan completes, display the report to the user. If running in `full` mode, proceed to migration.

### Mode: `migrate`

Launch the **migrator** agent with the following prompt:

> Using the scan report below, migrate all classical cryptographic code to NIST post-quantum alternatives. For each finding, check the library matrix to determine the best PQC library for the target language. Use existing libraries when available; write FFI bindings to liboqs for languages without native PQC libraries. Use hybrid mode (classical + PQC) for TLS/protocol contexts. Produce a structured Quantum Migration Report.
>
> Scan Report:
> [paste the scanner's output]

The migrator agent will:
1. Process each finding in priority order (CRITICAL → HIGH → MEDIUM)
2. Select the appropriate PQC library per language from the library matrix
3. Rewrite code using Edit tool, preserving code style
4. Add PQC dependencies to package manager files
5. Write FFI bindings for languages without native libraries
6. Flag custom implementations for security audit
7. Produce a migration report

After migration completes, display the report to the user. If running in `full` mode, proceed to validation.

### Mode: `validate`

Launch the **validator** agent with the following prompt:

> Validate the post-quantum migration using the migration report below. Re-scan for remaining classical crypto, verify dependencies resolve, attempt to build the project, run existing tests, and check all custom implementations. Produce a structured Quantum Migration Validation Report.
>
> Migration Report:
> [paste the migrator's output]

The validator agent will:
1. Re-scan to verify all classical crypto was migrated (or justified as hybrid/deferred)
2. Verify PQC dependencies resolve correctly
3. Attempt to build the project
4. Run existing test suites
5. Flag all custom implementations for audit
6. Produce a validation report with overall PASS/FAIL status

### Mode: `full` (default)

Execute all three phases in sequence:
1. **Scan** → produce findings report
2. **Migrate** → using findings, rewrite code → produce migration report
3. **Validate** → verify migration → produce validation report

Between phases, briefly summarize the results to the user before proceeding to the next phase.

## Important Notes

- The **scan** phase is always safe to run -- it makes no modifications
- The **migrate** phase modifies source files and dependency manifests
- The **validate** phase may run build commands and test suites
- For TLS/protocol contexts, hybrid mode (classical + PQC) is the default
- Custom crypto implementations are always flagged for mandatory third-party audit
- This tool supports: Python, JavaScript/TypeScript, Go, Rust, Java, C/C++, C#, Ruby, PHP, Swift, Kotlin

## Why This Matters

Every day that classical cryptography remains in your codebase is another day that encrypted data can be harvested for future quantum decryption. NIST finalized these standards in August 2024. The migration window is now. Run `/quantum-secure scan` today to understand your exposure.
