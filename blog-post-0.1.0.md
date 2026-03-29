# Your Encryption Has an Expiration Date. You Just Don't Know It Yet.

**Introducing quantum-secure v0.1.0 -- One Command to Protect Your Codebase From the Quantum Apocalypse**

---

Right now, as you read this, someone is recording your encrypted data.

Not because they can read it today. Because they know they'll be able to read it tomorrow.

It's called **Harvest Now, Decrypt Later**, and it's not a theoretical attack. Nation-states, intelligence agencies, and sophisticated adversaries are systematically intercepting and storing encrypted communications -- financial transactions, health records, trade secrets, classified government data, your users' private information -- with the explicit plan to decrypt all of it once quantum computers become powerful enough.

And that day is coming faster than most people realize.

---

## The Math is Settled. The Clock is Running.

In August 2024, NIST published the final post-quantum cryptography standards: **FIPS 203**, **FIPS 204**, and **FIPS 205**. Eight years of research. Thousands of cryptanalysts. Three algorithms that will replace everything you use today for public-key cryptography.

Here is what quantum computers break:

- **RSA** -- all key sizes. RSA-4096 offers zero additional protection against Shor's algorithm.
- **ECDSA** -- all curves. P-256, P-384, P-521, secp256k1. All of them.
- **ECDH** -- all curves. Every key exchange built on elliptic curves.
- **EdDSA** -- Ed25519, Ed448. The modern favorites. Equally vulnerable.
- **DH / DHE** -- classic Diffie-Hellman. Broken.
- **DSA** -- broken.

This is not a matter of using longer keys. The entire mathematical foundation -- integer factorization for RSA, discrete logarithm for DH, elliptic curve discrete logarithm for ECC -- collapses under Shor's algorithm. There is no classical fix. The only fix is new math.

NIST spent eight years finding that new math. Now it's your turn to deploy it.

---

## The Timeline That Should Terrify You

- **2024**: NIST finalizes post-quantum standards
- **2025**: Google Chrome, Cloudflare, Apple iMessage, Amazon, and Signal deploy PQC in production
- **2025-2026**: You are here
- **2028-2035**: Most estimates for cryptographically-relevant quantum computers
- **2033**: NSA deadline for full PQC migration of national security systems

Now consider this: enterprise cryptographic migrations historically take **5 to 15 years**. The window between "standards published" and "quantum computers break RSA" is estimated at **3 to 8 years**.

Do the math. If you haven't started migrating, you may already be too late to finish before the threat arrives.

The White House issued **National Security Memorandum 10** directing federal agencies to begin migration. CISA, NSA, and NIST published joint guidance urging critical infrastructure to act immediately. The NSA's **CNSA 2.0** mandates post-quantum algorithms for all national security systems starting 2025.

If the organizations responsible for protecting nuclear weapons and intelligence satellites have assessed this as urgent enough to mandate action, what exactly are you waiting for?

---

## Introducing quantum-secure v0.1.0

We built `quantum-secure` because the gap between "standards exist" and "codebases are migrated" is unacceptable. The tools exist. The knowledge exists. What's missing is the bridge from theory to your actual codebase.

`quantum-secure` is a Claude Code marketplace plugin that takes your codebase -- in any language -- and migrates your classical cryptography to NIST post-quantum standards. One command:

```
/quantum-secure full
```

That's it. Scan. Migrate. Validate. Done.

---

## How It Works

The plugin orchestrates three specialized AI agents, each with deep knowledge of cryptographic patterns across every major programming language.

### Phase 1: The Scanner

```
/quantum-secure scan
```

The scanner is **completely read-only** -- safe to run on any codebase, anywhere, right now. It performs a comprehensive sweep looking for classical cryptographic usage:

- **Import statements**: `from cryptography.hazmat.primitives.asymmetric import rsa`, `import "crypto/ecdsa"`, `use rsa::RsaPrivateKey`, `KeyPairGenerator.getInstance("RSA")`
- **Function calls**: `crypto.generateKeyPairSync('rsa')`, `RSA_generate_key_ex()`, `ECDsa.Create()`
- **Configuration strings**: TLS cipher suites, SSH key types, certificate configurations
- **Dependency manifests**: package.json, requirements.txt, Cargo.toml, pom.xml -- libraries that carry classical crypto

Every finding is classified:

| Risk Level | What It Means | Examples |
|------------|--------------|---------|
| **CRITICAL** | Directly broken by quantum computers via Shor's algorithm | RSA, ECDSA, ECDH, EdDSA, DH, DSA |
| **HIGH** | Weakened by Grover's algorithm to insufficient security levels | AES-128, 3DES, Blowfish, RC4, MD5, SHA-1 |
| **MEDIUM** | Protocol or configuration level exposure | TLS cipher suites, SSH config, certificate files |

The scanner produces a structured report telling you exactly how exposed you are. For most codebases, the answer is: very.

### Phase 2: The Migrator

```
/quantum-secure migrate
```

This is where it gets real. The migrator takes every finding and rewrites your code to use quantum-secure alternatives. For each piece of classical crypto, it follows a decision tree:

**1. What replaces it?**

| Your Current Crypto | NIST Replacement | Standard |
|---|---|---|
| RSA key exchange, ECDH, DH, X25519 | **ML-KEM** (Module-Lattice Key Encapsulation) | FIPS 203 |
| RSA signatures, ECDSA, EdDSA, DSA | **ML-DSA** (Module-Lattice Digital Signatures) | FIPS 204 |
| Signatures (conservative, hash-based) | **SLH-DSA** (Stateless Hash-Based Signatures) | FIPS 205 |
| AES-128, DES, 3DES, RC4 | **AES-256-GCM** | -- |
| MD5, SHA-1 | **SHA-256+** | -- |

**2. Is there a PQC library for this language?**

The migrator checks a comprehensive library matrix. If a mature PQC library exists -- it uses it and adds the dependency to your package manager. If no library exists -- it writes FFI bindings to `liboqs`, the C reference implementation maintained by the Open Quantum Safe project.

**3. Is this a protocol context (TLS, SSH)?**

For TLS and protocol-level crypto, the migrator defaults to **hybrid mode** -- combining classical and post-quantum algorithms together. This is exactly what Google Chrome and Cloudflare deploy in production: `X25519 + ML-KEM-768` for key exchange. You get quantum protection from the PQC component and backward compatibility from the classical component.

The migrator preserves your code style, adds concise migration comments, updates all dependency files, and flags anything that needs manual review (like database column sizes for larger PQC keys).

### Phase 3: The Validator

```
/quantum-secure validate
```

Trust, but verify. The validator:

1. **Re-scans** the codebase to confirm all classical crypto has been migrated (or justified as hybrid/deferred)
2. **Checks dependencies** resolve correctly -- no broken imports
3. **Builds the project** to catch compilation errors from the migration
4. **Runs your existing test suite** to verify nothing broke
5. **Flags every custom implementation** (FFI bindings, WASM shims) for mandatory third-party security audit
6. **Checks for common mistakes** -- hardcoded keys, non-cryptographic RNG, key material in logs

You get a clear **PASS**, **FAIL**, or **PASS WITH WARNINGS** verdict.

---

## Every Language. No Exceptions.

This is not a tool for one ecosystem. Classical crypto is everywhere, in every language, in every codebase. So quantum-secure works everywhere too.

### Tier 1: Full Native Library Support

These languages have mature, production-ready PQC libraries. The migrator drops in the right library and rewrites your code.

| Language | PQC Library | ML-KEM | ML-DSA | SLH-DSA |
|----------|-------------|:---:|:---:|:---:|
| **C/C++** | liboqs (Open Quantum Safe) | Yes | Yes | Yes |
| **Rust** | pqcrypto crate | Yes | Yes | Yes |
| **Python** | oqs-python | Yes | Yes | Yes |
| **Java** | Bouncy Castle | Yes | Yes | Yes |
| **Kotlin (JVM)** | Bouncy Castle | Yes | Yes | Yes |
| **C# (.NET)** | BouncyCastle.Cryptography | Yes | Yes | Yes |
| **Go** | circl (Cloudflare) | Yes | Yes | Partial |

### Tier 2: Partial Native Support

These languages have PQC libraries for some algorithms. The migrator uses what's available and fills gaps with WASM or FFI bindings.

| Language | Coverage | Gap Strategy |
|----------|----------|-------------|
| **JavaScript/TypeScript** | ML-KEM via npm | WASM liboqs for ML-DSA, SLH-DSA |
| **Swift** | ML-KEM via CryptoKit (iOS 17.4+) | FFI to liboqs for signatures |

### Tier 3: Custom Implementation

These languages lack PQC libraries entirely. The migrator writes FFI bindings to liboqs -- the battle-tested C reference implementation -- giving you quantum security today while the ecosystem catches up.

| Language | Strategy |
|----------|----------|
| **Ruby** | FFI gem binding to liboqs |
| **PHP** | PHP FFI extension binding to liboqs |
| **Elixir/Erlang** | NIF binding to liboqs |

Every custom implementation is clearly marked and flagged for third-party security audit. No exceptions. No shortcuts with custom crypto.

---

## The Three Algorithms That Save Everything

### ML-KEM (FIPS 203) -- Replacing Key Exchange

Every time two systems establish a shared secret -- TLS handshakes, API authentication, encrypted messaging -- they use key exchange. Today that's RSA-KEM, ECDH, or Diffie-Hellman. All broken by quantum.

**ML-KEM** (based on CRYSTALS-Kyber) is the replacement. It's a Key Encapsulation Mechanism built on the Module Learning With Errors problem. It's fast -- faster than RSA -- and the keys are compact enough for real-world deployment.

| Parameter Set | Security Level | Public Key | Ciphertext |
|---|---|---|---|
| ML-KEM-512 | AES-128 equivalent | 800 bytes | 768 bytes |
| **ML-KEM-768** | AES-192 equivalent | 1,184 bytes | 1,088 bytes |
| ML-KEM-1024 | AES-256 equivalent | 1,568 bytes | 1,568 bytes |

**Default recommendation**: ML-KEM-768. Google and Cloudflare are already using it in production TLS.

### ML-DSA (FIPS 204) -- Replacing Digital Signatures

Every certificate, every code signature, every signed JWT, every authentication token that uses RSA or ECDSA is vulnerable. ML-DSA (based on CRYSTALS-Dilithium) replaces all of them.

| Parameter Set | Security Level | Public Key | Signature |
|---|---|---|---|
| ML-DSA-44 | Level 2 | 1,312 bytes | 2,420 bytes |
| **ML-DSA-65** | Level 3 | 1,952 bytes | 3,293 bytes |
| ML-DSA-87 | Level 5 | 2,592 bytes | 4,595 bytes |

**Default recommendation**: ML-DSA-65. Fast signing, fast verification, reasonable sizes.

### SLH-DSA (FIPS 205) -- The Conservative Fallback

Here's the thing about ML-KEM and ML-DSA: they're both built on lattice mathematics. If someone finds a breakthrough against lattice problems, both fall. SLH-DSA is the insurance policy -- it relies **only on hash function security** (SHA-256, SHAKE). Hash functions are some of the most deeply studied primitives in cryptography.

SLH-DSA signatures are larger and slower, but the security is built on bedrock. Use it for:
- Long-lived signatures (certificates valid for decades)
- Maximum assurance environments (defense, financial infrastructure)
- Algorithm diversity (sign with ML-DSA AND SLH-DSA)

---

## Installation: 5 Minutes to Quantum Security

### Step 1: Add the Marketplace

Add to your Claude Code settings (`~/.claude/settings.json`):

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

### Step 2: Install the Plugin

```
/install quantum-secure@quantum-security
```

### Step 3: Know Your Exposure

```
/quantum-secure scan
```

This is read-only. It changes nothing. It just tells you the truth about your codebase. Run it right now.

### Step 4: Fix It

```
/quantum-secure full
```

Scan. Migrate. Validate. One command.

---

## "But Quantum Computers Are Years Away"

This is the most dangerous sentence in cybersecurity right now.

Yes, cryptographically-relevant quantum computers are likely years away. But consider what "years away" actually means:

**Your data doesn't expire.** A medical record encrypted with RSA-2048 today will still be a medical record in 2035. An intercepted trade secret is still valuable a decade from now. Attorney-client communications remain privileged indefinitely. The classified intelligence being transmitted right now has sensitivity measured in decades, not years.

**Your migration takes years too.** You don't flip a switch and become quantum-secure. You have to find every instance of classical crypto across every service, every library, every configuration file, every language in your stack. You have to replace it, test it, deploy it, and verify it didn't break anything. At enterprise scale, this is a multi-year initiative. The organizations that start in 2026 finish by 2030. The ones that start in 2029 finish by 2035 -- if they're lucky.

**The harvest is happening now.** The encrypted data being intercepted today doesn't need quantum computers today. It just needs to be stored. Storage is cheap. Patience is free. The adversary's bet is simple: collect everything, wait, decrypt later. The only way to defeat this bet is to encrypt with algorithms that quantum computers can't break. And those algorithms are available today.

**Your competitors are moving.** Google deployed PQC in Chrome. Apple deployed it in iMessage. Cloudflare deployed it across their entire network. Signal deployed it in their protocol. Amazon deployed it in AWS KMS. If these organizations -- who have some of the best security teams on the planet -- have decided the threat is real enough to act on today, your risk assessment should probably align with theirs.

---

## What Happens If You Do Nothing

Let's be specific about what "do nothing" looks like:

1. **Today**: Your API encrypts data in transit with ECDH P-256 and signs tokens with RSA-2048
2. **Today**: An adversary records this traffic at the network level (trivial for a nation-state, achievable for sophisticated criminals)
3. **2028-2035**: A quantum computer factors your RSA key in hours and solves your ECDH discrete log in minutes
4. **2028-2035**: Every API call, every authentication token, every encrypted message from 2026 is now plaintext

This is not science fiction. This is a straightforward application of an algorithm published in 1994 (Shor's algorithm) running on hardware that is advancing rapidly. The only variable is when, not if.

---

## What Happens If You Act Now

1. **Today**: You run `/quantum-secure scan` and discover 47 instances of classical crypto across your Python, TypeScript, and Go services
2. **Today**: You run `/quantum-secure full` and the migrator replaces them with ML-KEM-768, ML-DSA-65, and AES-256-GCM
3. **Today**: The validator confirms 100% migration, all tests pass, build succeeds
4. **2028-2035**: The quantum computer arrives. Your data -- past and present -- is unreadable. The adversary's harvest is worthless. Your users are protected.

That's the difference. One command. Run it today.

---

## FAQ

**Q: Is this safe to run on production code?**
A: The scan phase is completely read-only. The migrate phase modifies code and should be reviewed like any other code change. The validate phase runs builds and tests. We recommend running on a feature branch first.

**Q: What if my language doesn't have a PQC library?**
A: The migrator writes FFI bindings to liboqs, the C reference implementation. This covers any language that can call C functions. For browser JavaScript, it uses WASM. Every custom implementation is flagged for security audit.

**Q: Should I use hybrid mode or pure PQC?**
A: For TLS/protocol contexts, hybrid is the default and recommended approach (this is what Chrome and Cloudflare do). For application-layer crypto, pure PQC is fine if you don't need backward compatibility with non-PQC peers.

**Q: What about performance?**
A: ML-KEM is actually faster than RSA for key exchange. ML-DSA is fast but produces larger signatures (3 KB vs 64 bytes for Ed25519). SLH-DSA is slower and should only be used when conservative security is required. For most applications, the performance impact is negligible.

**Q: What about JWT/JOSE?**
A: The IETF is still standardizing PQC algorithm identifiers for JOSE/JWT. The migrator flags these for deferred migration with a TODO comment rather than making a breaking change. This is the responsible approach.

**Q: Is this a one-time thing?**
A: Run `/quantum-secure scan` regularly as your codebase evolves. New classical crypto can be introduced by new dependencies, new features, or new team members who haven't yet internalized the quantum threat.

---

## The Bottom Line

The post-quantum standards are published. The libraries exist. The tools exist. The threat is real and accelerating. Every line of classical crypto in your codebase is a liability. Every day without migration is another day of harvestable data in transit.

You don't need a quantum computing PhD. You don't need to understand lattice mathematics. You don't need to rewrite your crypto stack by hand. You need one command:

```
/quantum-secure full
```

**Your encryption has an expiration date. quantum-secure v0.1.0 removes it.**

Install it. Run it. Today. Not next quarter. Not after the next sprint. Today.

The quantum clock is ticking. Your adversaries are patient. Your data is not.

---

*quantum-secure v0.1.0 is open source under Apache 2.0. Available at [github.com/joestein/quantum-security](https://github.com/joestein/quantum-security).*

*Supports: Python, JavaScript/TypeScript, Go, Rust, Java, Kotlin, C/C++, C#, Ruby, PHP, Swift, Elixir/Erlang. Built on NIST FIPS 203 (ML-KEM), FIPS 204 (ML-DSA), FIPS 205 (SLH-DSA).*
