# Classical to Post-Quantum Algorithm Mapping

This is the canonical reference for which NIST PQC algorithm replaces which classical algorithm. Both the scanner and migrator agents reference this document.

## Key Encapsulation / Key Exchange

| Classical Algorithm | PQC Replacement | NIST Standard | Default Parameter Set | High Security Set | Notes |
|---|---|---|---|---|---|
| RSA key exchange (RSA-KEM) | ML-KEM | FIPS 203 | ML-KEM-768 | ML-KEM-1024 | RSA used for key transport |
| ECDH (P-256, P-384, P-521) | ML-KEM | FIPS 203 | ML-KEM-768 | ML-KEM-1024 | Elliptic curve key agreement |
| DH / DHE (finite field) | ML-KEM | FIPS 203 | ML-KEM-768 | ML-KEM-1024 | Classic Diffie-Hellman |
| X25519 (key agreement) | ML-KEM | FIPS 203 | ML-KEM-768 | ML-KEM-1024 | Hybrid X25519+ML-KEM-768 recommended during transition |
| X448 (key agreement) | ML-KEM | FIPS 203 | ML-KEM-1024 | ML-KEM-1024 | Higher security curve maps to higher parameter set |
| ElGamal | ML-KEM | FIPS 203 | ML-KEM-768 | ML-KEM-1024 | Rarely seen but still vulnerable |

### ML-KEM Parameter Sets

| Parameter Set | Classical Equivalent | Public Key Size | Ciphertext Size | Shared Secret |
|---|---|---|---|---|
| ML-KEM-512 | AES-128 | 800 bytes | 768 bytes | 32 bytes |
| ML-KEM-768 | AES-192 | 1,184 bytes | 1,088 bytes | 32 bytes |
| ML-KEM-1024 | AES-256 | 1,568 bytes | 1,568 bytes | 32 bytes |

**Default recommendation**: ML-KEM-768 for most applications. Use ML-KEM-1024 for long-term secrets or high-assurance contexts.

## Digital Signatures

| Classical Algorithm | PQC Replacement (Default) | PQC Replacement (Conservative) | NIST Standard | Default Parameter Set |
|---|---|---|---|---|
| RSA signatures (PKCS#1, PSS) | ML-DSA | SLH-DSA | FIPS 204 / FIPS 205 | ML-DSA-65 |
| ECDSA (all curves) | ML-DSA | SLH-DSA | FIPS 204 / FIPS 205 | ML-DSA-65 |
| EdDSA (Ed25519, Ed448) | ML-DSA | SLH-DSA | FIPS 204 / FIPS 205 | ML-DSA-65 |
| DSA | ML-DSA | SLH-DSA | FIPS 204 / FIPS 205 | ML-DSA-65 |

### ML-DSA Parameter Sets

| Parameter Set | NIST Security Level | Public Key Size | Signature Size |
|---|---|---|---|
| ML-DSA-44 | 2 (~ AES-128) | 1,312 bytes | 2,420 bytes |
| ML-DSA-65 | 3 (~ AES-192) | 1,952 bytes | 3,293 bytes |
| ML-DSA-87 | 5 (~ AES-256) | 2,592 bytes | 4,595 bytes |

**Default recommendation**: ML-DSA-65 for most applications. Use ML-DSA-87 for high-assurance (government, financial, long-lived certificates).

### SLH-DSA Parameter Sets

Use SLH-DSA when:
- You need a conservative fallback that does NOT rely on lattice hardness assumptions
- The application can tolerate larger signatures
- Long-lived signatures where algorithm diversity is desired

| Parameter Set | Security Level | Public Key | Signature Size | Speed |
|---|---|---|---|---|
| SLH-DSA-SHA2-128s | 1 | 32 bytes | 7,856 bytes | Slow, small sigs |
| SLH-DSA-SHA2-128f | 1 | 32 bytes | 17,088 bytes | Fast, larger sigs |
| SLH-DSA-SHA2-192s | 3 | 48 bytes | 16,224 bytes | Slow, small sigs |
| SLH-DSA-SHA2-192f | 3 | 48 bytes | 35,664 bytes | Fast, larger sigs |
| SLH-DSA-SHA2-256s | 5 | 64 bytes | 29,792 bytes | Slow, small sigs |
| SLH-DSA-SHA2-256f | 5 | 64 bytes | 49,856 bytes | Fast, larger sigs |
| SLH-DSA-SHAKE-128s | 1 | 32 bytes | 7,856 bytes | Slow, small sigs |
| SLH-DSA-SHAKE-128f | 1 | 32 bytes | 17,088 bytes | Fast, larger sigs |
| SLH-DSA-SHAKE-192s | 3 | 48 bytes | 16,224 bytes | Slow, small sigs |
| SLH-DSA-SHAKE-192f | 3 | 48 bytes | 35,664 bytes | Fast, larger sigs |
| SLH-DSA-SHAKE-256s | 5 | 64 bytes | 29,792 bytes | Slow, small sigs |
| SLH-DSA-SHAKE-256f | 5 | 64 bytes | 49,856 bytes | Fast, larger sigs |

**Default recommendation**: SLH-DSA-SHA2-128s for most use cases where SLH-DSA is chosen. Use SHA2 variants for compatibility; SHAKE variants where SHAKE is already in use.

## Symmetric Cryptography (Grover's Algorithm Impact)

Grover's algorithm effectively halves symmetric key lengths. Migrate to maintain equivalent post-quantum security.

| Classical | Issue | PQC Replacement | Notes |
|---|---|---|---|
| AES-128 | 64-bit effective security under Grover's | AES-256 | AES-256-GCM preferred |
| AES-192 | 96-bit effective security | AES-256 | Upgrade recommended |
| AES-256 | 128-bit effective security | AES-256 (keep) | Already sufficient |
| 3DES | 56-bit effective (already weak) | AES-256-GCM | Deprecated classically too |
| Blowfish | Variable, generally weak | AES-256-GCM | Deprecated classically too |
| RC4 | Broken | AES-256-GCM | Broken classically |
| ChaCha20-Poly1305 | 128-bit effective under Grover's | ChaCha20-Poly1305 (keep) | 256-bit key, sufficient |

## Hash Functions

| Classical | Issue | Recommendation | Notes |
|---|---|---|---|
| MD5 | Broken classically | SHA-256 minimum | Replace immediately regardless of quantum |
| SHA-1 | Broken classically | SHA-256 minimum | Replace immediately regardless of quantum |
| SHA-256 | 128-bit under Grover's | SHA-256 (adequate) or SHA-384/SHA-512 for margin | Generally sufficient for most applications |
| SHA-384 | 192-bit under Grover's | Keep | Sufficient |
| SHA-512 | 256-bit under Grover's | Keep | Sufficient |
| SHA-3 family | Same Grover's impact | Keep | Sufficient at 256+ bit variants |

## Hybrid Schemes (Transition Period)

During the transition period, combine classical + PQC for defense-in-depth:

| Context | Hybrid Scheme | Notes |
|---|---|---|
| TLS key exchange | X25519 + ML-KEM-768 | Matches Chrome/Cloudflare deployment |
| TLS signatures | ECDSA + ML-DSA-65 | Dual certificate chain |
| SSH key exchange | curve25519 + ML-KEM-768 | OpenSSH 9.x+ supports this |
| Code signing | ECDSA + ML-DSA-65 | Sign with both, verify either |
| Certificate chains | RSA/ECDSA + ML-DSA | Dual-algorithm certificates |

Hybrid mode should be used when:
- Interoperability with non-PQC systems is required
- The migration is incremental and classical support must be maintained
- Protocol specifications mandate backward compatibility
