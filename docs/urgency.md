# The Quantum Threat is Here -- Why You Must Migrate Now

## The Clock is Already Ticking

This is not a future problem. This is a **today** problem.

Adversaries -- nation-states, sophisticated criminal organizations, and intelligence agencies -- are executing **"Harvest Now, Decrypt Later" (HNDL)** attacks right now. They are intercepting and storing encrypted communications, financial transactions, health records, trade secrets, and classified data with the explicit intention of decrypting it once cryptographically-relevant quantum computers (CRQCs) become available.

Every piece of data you encrypt today with RSA, ECDSA, or ECDH can be stored by an adversary and decrypted in the future. The data doesn't expire. The threat doesn't wait.

## The Timeline

### What Has Already Happened

- **2016**: NIST launches the Post-Quantum Cryptography Standardization Process
- **2022**: White House issues **National Security Memorandum 10 (NSM-10)**, directing federal agencies to begin quantum migration planning
- **2022**: CISA, NSA, and NIST publish joint guidance urging critical infrastructure to start PQC migration immediately
- **2023**: NSA announces CNSA 2.0 Suite requiring PQC for national security systems by 2025-2033
- **August 2024**: NIST publishes final standards -- **FIPS 203 (ML-KEM)**, **FIPS 204 (ML-DSA)**, and **FIPS 205 (SLH-DSA)**
- **2024-2025**: Google Chrome, Cloudflare, and Amazon deploy hybrid PQC in production TLS
- **2025**: Apple iMessage adopts PQ3 protocol with ML-KEM
- **2026**: You are reading this document. The standards are finalized. The tools exist. **There are no more excuses.**

### What is Coming

- **2025-2030**: CNSA 2.0 mandates PQC for all US national security systems
- **2028-2030**: Most estimates for when quantum computers will threaten 2048-bit RSA
- **2030-2035**: Broader quantum computing availability; classical-only systems become indefensible
- **2033**: NSA deadline for full PQC migration of national security systems

The gap between "standards finalized" and "quantum computers breaking RSA" is estimated at **3-8 years**. Cryptographic migrations at enterprise scale historically take **5-15 years**. Do the math. If you haven't started, you may already be too late.

## What Shor's Algorithm Breaks

Shor's algorithm, running on a sufficiently powerful quantum computer, completely breaks:

| Algorithm | Used For | Status |
|-----------|----------|--------|
| **RSA** (all key sizes) | Key exchange, digital signatures, certificates | **BROKEN** by quantum |
| **ECDSA** (all curves) | Digital signatures, TLS, code signing | **BROKEN** by quantum |
| **ECDH** (all curves) | Key agreement, TLS handshake | **BROKEN** by quantum |
| **EdDSA** (Ed25519, Ed448) | Digital signatures, SSH, code signing | **BROKEN** by quantum |
| **DSA** | Digital signatures (legacy) | **BROKEN** by quantum |
| **DH / DHE** | Key exchange | **BROKEN** by quantum |
| **ElGamal** | Encryption | **BROKEN** by quantum |

This is not a matter of key size. RSA-4096 offers no additional protection against Shor's algorithm compared to RSA-2048. Elliptic curve cryptography is equally vulnerable regardless of curve choice. **The entire mathematical foundation of public-key cryptography as deployed today is broken by quantum computers.**

## What Grover's Algorithm Weakens

Grover's algorithm halves the effective security of symmetric algorithms:

| Algorithm | Classical Security | Post-Quantum Security | Action |
|-----------|-------------------|----------------------|--------|
| AES-128 | 128-bit | **64-bit** (insufficient) | Upgrade to AES-256 |
| AES-256 | 256-bit | 128-bit (sufficient) | Keep |
| SHA-256 | 256-bit | 128-bit (sufficient) | Keep (or upgrade to SHA-384) |
| 3DES | 112-bit | **56-bit** (broken) | Replace with AES-256 |

## Harvest Now, Decrypt Later: A Concrete Scenario

Consider this scenario:

1. **Today**: Your application uses RSA-2048 to encrypt data in transit between services
2. **Today**: A state-sponsored adversary records this encrypted traffic (this is trivial at the network level)
3. **Today-2030**: The encrypted data sits in storage, waiting
4. **2030**: A quantum computer running Shor's algorithm factors your RSA-2048 key in hours
5. **2030**: Every "encrypted" message, every API call, every token exchange from today is now plaintext

If the data you're encrypting today has value for more than 3-5 years -- health records, financial data, legal communications, intellectual property, government secrets -- it is **already at risk**.

## The Cost of Waiting

### Technical Debt Compounds

Every line of classical crypto code written today is a line that must be migrated tomorrow. Every new service deployed with RSA or ECDSA is another migration target. The longer you wait:

- More code to migrate
- More developers to retrain
- More systems to test
- More interoperability issues to resolve
- Higher probability of rushed, error-prone migration

### Compliance is Tightening

- US federal agencies: NSM-10 mandates PQC migration planning now, full migration by 2033
- Financial sector: PCI DSS and banking regulators are beginning to require PQC readiness assessments
- Healthcare: HIPAA-covered entities handling long-lived patient data should consider HNDL exposure
- EU: ENISA has published recommendations for PQC transition timelines
- Defense/intelligence: CNSA 2.0 requires PQC for classified systems starting 2025

### Your Competitors are Moving

Google, Apple, Cloudflare, Amazon, Signal, and Microsoft have all deployed PQC in production. If they've assessed the threat as urgent enough to act on, what are you waiting for?

## What You Can Do Right Now

### 1. Assess Your Exposure (5 minutes)

Run the quantum-secure scanner on your codebase:

```
/quantum-secure scan
```

This is read-only and safe. It will tell you exactly how much classical crypto is in your code and what needs to change.

### 2. Start Migrating (varies)

Run the full migration pipeline:

```
/quantum-secure full
```

This will scan, migrate, and validate your codebase in one command.

### 3. Prioritize by Data Sensitivity

Not all data needs the same urgency:

| Priority | Data Type | Why |
|----------|-----------|-----|
| **Immediate** | Government/military secrets, long-term keys, CA certificates | HNDL exposure is existential |
| **High** | Health records, financial data, legal communications | Decades-long sensitivity |
| **Medium** | Authentication tokens, session keys, API keys | Short-lived but high-volume |
| **Lower** | Public data, ephemeral communications | Limited HNDL value |

### 4. Adopt Hybrid Mode During Transition

You don't have to go cold-turkey. Hybrid mode (classical + PQC) provides:
- Quantum protection from the PQC component
- Fallback security from the classical component (in case of PQC implementation bugs)
- Backward compatibility with non-PQC peers

### 5. Plan for the Long Term

- Add PQC to your security architecture documents
- Include PQC readiness in vendor assessments
- Train your team on post-quantum concepts
- Build PQC into new projects from day one

## The Bottom Line

The question is not **whether** you need to migrate to post-quantum cryptography. That is settled. The standards are published. The threat model is clear.

The question is **when**. And the answer is **now**.

Every day of delay is:
- Another day of HNDL-vulnerable data in transit
- Another day of technical debt accumulating
- Another day closer to quantum computers that break your encryption

**Run `/quantum-secure scan` today. Know your exposure. Start migrating.**

---

## Further Reading

- NIST FIPS 203, 204, 205: The official post-quantum cryptography standards
- NSM-10: White House National Security Memorandum on Promoting United States Leadership in Quantum Computing While Mitigating Risks to Vulnerable Cryptographic Systems
- CISA Post-Quantum Cryptography Initiative: Guidance for critical infrastructure
- CNSA 2.0: NSA's Commercial National Security Algorithm Suite for quantum resistance
- ETSI QSC: European standards body quantum-safe cryptography working group
