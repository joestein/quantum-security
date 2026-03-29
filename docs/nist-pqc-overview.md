# NIST Post-Quantum Cryptography Standards Overview

## Background

In August 2024, NIST published three Federal Information Processing Standards (FIPS) for post-quantum cryptography, concluding an 8-year standardization process that began in 2016. These standards provide quantum-resistant replacements for all classical public-key cryptographic algorithms.

The three standards are:
- **FIPS 203**: ML-KEM (Module-Lattice-Based Key-Encapsulation Mechanism)
- **FIPS 204**: ML-DSA (Module-Lattice-Based Digital Signature Algorithm)
- **FIPS 205**: SLH-DSA (Stateless Hash-Based Digital Signature Algorithm)

Together, they cover all use cases currently served by RSA, ECDSA, ECDH, EdDSA, DH, and DSA.

---

## FIPS 203: ML-KEM (Module-Lattice-Based Key-Encapsulation Mechanism)

**Based on**: CRYSTALS-Kyber (Round 3 selection)

**Purpose**: Key encapsulation -- establishing a shared secret between two parties over an insecure channel. This replaces:
- RSA key exchange / RSA-KEM
- Elliptic Curve Diffie-Hellman (ECDH)
- Finite-field Diffie-Hellman (DH / DHE)
- X25519 / X448 key agreement

**How it works**: ML-KEM is a lattice-based KEM built on the Module Learning With Errors (MLWE) problem. Unlike classical DH-style key exchange, ML-KEM is a KEM -- one party generates a keypair, the other party encapsulates a shared secret to the public key, and the first party decapsulates it with the private key.

### Parameter Sets

| Parameter Set | NIST Security Level | Equivalent Classical Security | Public Key | Ciphertext | Shared Secret |
|---|---|---|---|---|---|
| **ML-KEM-512** | 1 | AES-128 | 800 bytes | 768 bytes | 32 bytes |
| **ML-KEM-768** | 3 | AES-192 | 1,184 bytes | 1,088 bytes | 32 bytes |
| **ML-KEM-1024** | 5 | AES-256 | 1,568 bytes | 1,568 bytes | 32 bytes |

### Recommendation

- **Default**: ML-KEM-768 for most applications
- **High security**: ML-KEM-1024 for long-term secrets, government, financial systems
- **Constrained**: ML-KEM-512 only when bandwidth/storage is severely constrained

### Key Operations

1. **KeyGen()** → (public_key, secret_key)
2. **Encaps(public_key)** → (ciphertext, shared_secret)
3. **Decaps(secret_key, ciphertext)** → shared_secret

### Performance Characteristics

ML-KEM is fast -- significantly faster than RSA key exchange:
- Key generation: ~microseconds
- Encapsulation: ~microseconds
- Decapsulation: ~microseconds
- Bandwidth overhead: moderate (1-2 KB vs. 32 bytes for X25519, but much smaller than RSA-4096)

---

## FIPS 204: ML-DSA (Module-Lattice-Based Digital Signature Algorithm)

**Based on**: CRYSTALS-Dilithium (Round 3 selection)

**Purpose**: Digital signatures -- signing and verifying data. This replaces:
- RSA signatures (PKCS#1 v1.5, PSS)
- ECDSA (all curves: P-256, P-384, P-521, secp256k1)
- EdDSA (Ed25519, Ed448)
- DSA

**How it works**: ML-DSA is built on the Module Learning With Errors (MLWE) and Module Short Integer Solution (MSIS) problems. It uses a Fiat-Shamir with Aborts framework to prevent key leakage through signature analysis.

### Parameter Sets

| Parameter Set | NIST Security Level | Equivalent Classical Security | Public Key | Signature Size | Secret Key |
|---|---|---|---|---|---|
| **ML-DSA-44** | 2 | AES-128 | 1,312 bytes | 2,420 bytes | 2,560 bytes |
| **ML-DSA-65** | 3 | AES-192 | 1,952 bytes | 3,293 bytes | 4,032 bytes |
| **ML-DSA-87** | 5 | AES-256 | 2,592 bytes | 4,595 bytes | 4,896 bytes |

### Recommendation

- **Default**: ML-DSA-65 for most applications
- **High security**: ML-DSA-87 for government, financial, long-lived certificates
- **Constrained**: ML-DSA-44 when signature/key size is critical

### Key Operations

1. **KeyGen()** → (public_key, secret_key)
2. **Sign(secret_key, message)** → signature
3. **Verify(public_key, message, signature)** → boolean

### Performance Characteristics

ML-DSA is fast for signing and verification:
- Key generation: ~microseconds
- Signing: ~microseconds (may require multiple attempts due to abort mechanism)
- Verification: ~microseconds
- Size overhead: significant -- ML-DSA-65 signatures are 3,293 bytes vs. 64 bytes for Ed25519

### Size Impact

The dramatically larger signature and key sizes affect:
- Certificate chains (TLS handshake size increases)
- Blockchain/distributed ledger transactions
- Embedded/IoT systems with limited bandwidth
- Database columns storing signatures or public keys
- Protocol message formats with fixed-size fields

---

## FIPS 205: SLH-DSA (Stateless Hash-Based Digital Signature Algorithm)

**Based on**: SPHINCS+ (Round 3 selection)

**Purpose**: Digital signatures -- same use case as ML-DSA, but built on different mathematical foundations. SLH-DSA relies **only on hash function security**, not lattice problems.

**Why a second signature standard?** Algorithm diversity. If a future breakthrough breaks lattice-based cryptography (affecting both ML-KEM and ML-DSA), SLH-DSA remains secure because its security depends only on hash functions (SHA-256, SHA-512, SHAKE). This makes SLH-DSA the **conservative choice** for high-assurance applications.

### Parameter Sets

SLH-DSA offers many parameter sets across two dimensions:
- **Hash function**: SHA-256 or SHAKE-256
- **Speed/size tradeoff**: "s" (small signatures, slower) or "f" (fast signing, larger signatures)

| Parameter Set | Security Level | Public Key | Signature Size | Speed |
|---|---|---|---|---|
| **SLH-DSA-SHA2-128s** | 1 | 32 bytes | 7,856 bytes | Slower |
| **SLH-DSA-SHA2-128f** | 1 | 32 bytes | 17,088 bytes | Faster |
| **SLH-DSA-SHA2-192s** | 3 | 48 bytes | 16,224 bytes | Slower |
| **SLH-DSA-SHA2-192f** | 3 | 48 bytes | 35,664 bytes | Faster |
| **SLH-DSA-SHA2-256s** | 5 | 64 bytes | 29,792 bytes | Slower |
| **SLH-DSA-SHA2-256f** | 5 | 64 bytes | 49,856 bytes | Faster |
| **SLH-DSA-SHAKE-128s** | 1 | 32 bytes | 7,856 bytes | Slower |
| **SLH-DSA-SHAKE-128f** | 1 | 32 bytes | 17,088 bytes | Faster |
| **SLH-DSA-SHAKE-192s** | 3 | 48 bytes | 16,224 bytes | Slower |
| **SLH-DSA-SHAKE-192f** | 3 | 48 bytes | 35,664 bytes | Faster |
| **SLH-DSA-SHAKE-256s** | 5 | 64 bytes | 29,792 bytes | Slower |
| **SLH-DSA-SHAKE-256f** | 5 | 64 bytes | 49,856 bytes | Faster |

### Recommendation

- **Default**: SLH-DSA-SHA2-128s for most use cases where SLH-DSA is chosen
- **When to choose SLH-DSA over ML-DSA**:
  - Maximum conservative security (no lattice assumptions)
  - Long-lived signatures (decades) where algorithm diversity matters
  - Regulatory requirements for hash-based signatures
  - Defense-in-depth alongside ML-DSA (sign with both)
- **When to choose "s" vs "f"**: Use "s" (small) when bandwidth/storage matters more than signing speed. Use "f" (fast) when signing speed is critical and you can tolerate larger signatures.

### Key Operations

Same as ML-DSA:
1. **KeyGen()** → (public_key, secret_key)
2. **Sign(secret_key, message)** → signature
3. **Verify(public_key, message, signature)** → boolean

### Performance Characteristics

SLH-DSA is significantly **slower** than ML-DSA:
- Key generation: milliseconds (vs. microseconds for ML-DSA)
- Signing: milliseconds to seconds depending on parameter set
- Verification: milliseconds
- Public keys are tiny (32-64 bytes), but signatures are very large (7-50 KB)

---

## Choosing Between ML-DSA and SLH-DSA

| Factor | ML-DSA | SLH-DSA |
|--------|--------|---------|
| **Speed** | Very fast | Slow |
| **Signature size** | 2.4-4.6 KB | 7.8-49.8 KB |
| **Public key size** | 1.3-2.6 KB | 32-64 bytes |
| **Security basis** | Lattice problems (MLWE/MSIS) | Hash functions only |
| **Conservative?** | Moderate -- lattice math is well-studied but newer | Very -- hash functions are deeply understood |
| **Best for** | General purpose, high throughput | High assurance, long-lived, diversity |

**For most applications**: Use ML-DSA. It's faster, has reasonable signature sizes, and lattice-based cryptography is well-studied.

**For maximum assurance**: Use SLH-DSA, or sign with both ML-DSA and SLH-DSA.

---

## Hybrid Cryptography During Transition

During the migration period, NIST and other agencies recommend **hybrid schemes** that combine classical and post-quantum algorithms:

### Why Hybrid?

1. **Belt and suspenders**: If either the classical or PQC algorithm is broken, the other provides security
2. **Implementation maturity**: PQC libraries are newer; hybrid mode provides classical fallback
3. **Interoperability**: Not all peers support PQC yet; hybrid maintains backward compatibility

### Common Hybrid Schemes

| Use Case | Hybrid Combination | Deployed By |
|----------|-------------------|-------------|
| TLS key exchange | X25519 + ML-KEM-768 | Chrome, Cloudflare |
| TLS signatures | ECDSA + ML-DSA-65 | Experimental |
| SSH key exchange | curve25519 + ML-KEM-768 | OpenSSH 9.x |
| iMessage | ECDH + ML-KEM-768 | Apple PQ3 |
| Signal Protocol | X25519 + ML-KEM-768 | Signal PQXDH |

### When to Drop the Classical Component

Move from hybrid to pure PQC when:
- All peers/partners support PQC
- PQC implementations have been audited and are mature
- Regulatory requirements permit pure PQC
- The performance overhead of hybrid is unacceptable

---

## NIST Security Levels

NIST defines five security levels corresponding to the difficulty of attacking the algorithm:

| Level | Definition | Equivalent To |
|-------|-----------|---------------|
| 1 | At least as hard as AES-128 key search | AES-128 |
| 2 | At least as hard as SHA-256 collision | SHA-256 |
| 3 | At least as hard as AES-192 key search | AES-192 |
| 4 | At least as hard as SHA-384 collision | SHA-384 |
| 5 | At least as hard as AES-256 key search | AES-256 |

For most applications, **Level 3** (ML-KEM-768, ML-DSA-65) provides excellent security. Level 5 is recommended for the highest-assurance scenarios.
