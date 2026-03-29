# Quantum Security Marketplace for Claude Code

**Your codebase is not quantum-secure. Fix it today.**

Classical cryptography -- RSA, ECDSA, ECDH, DH -- is broken by quantum computers. Not theoretically. Not someday. The math is settled, the standards are published, and adversaries are already harvesting your encrypted data to decrypt later. NIST finalized post-quantum cryptography standards in August 2024. The migration window is open. **Start now.**

---

## The Threat

**Harvest Now, Decrypt Later (HNDL)**: Nation-state adversaries are intercepting and storing encrypted communications today, waiting for quantum computers to break the encryption. If your data has value for more than 3-5 years, it is already at risk.

**Timeline**: Most estimates place cryptographically-relevant quantum computers at 2028-2035. Enterprise cryptographic migrations historically take 5-15 years. **If you haven't started, you are already behind.**

Read the full analysis: [docs/urgency.md](docs/urgency.md)

---

## Available Plugins

### quantum-secure (v0.1.0)

One-command post-quantum migration for any codebase.

```
/quantum-secure full
```

**What it does**:
1. **Scans** your codebase for classical cryptography vulnerable to quantum attack
2. **Migrates** to NIST post-quantum standards (ML-KEM, ML-DSA, SLH-DSA)
3. **Validates** the migration -- checks completeness, builds, and runs tests

**Supports**: Python, JavaScript/TypeScript, Go, Rust, Java, C/C++, C#, Ruby, PHP, Swift, Kotlin -- and any future language via FFI bindings to liboqs.

**NIST Standards**:

| Standard | Algorithm | Replaces |
|----------|-----------|----------|
| FIPS 203 | **ML-KEM** | RSA key exchange, ECDH, DH |
| FIPS 204 | **ML-DSA** | RSA signatures, ECDSA, EdDSA |
| FIPS 205 | **SLH-DSA** | Signatures (conservative hash-based fallback) |

[Full plugin documentation](plugins/quantum-secure/README.md)

---

## Installation

### 1. Add the Marketplace

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

### 2. Install the Plugin

```
/install quantum-secure@quantum-security
```

### 3. Run It

```
/quantum-secure scan       # Assess your exposure (read-only, safe)
/quantum-secure full       # Full migration pipeline
```

---

## Documentation

| Document | Description |
|----------|-------------|
| [Urgency: Why Migrate Now](docs/urgency.md) | HNDL attacks, timelines, mandates, cost of delay |
| [NIST PQC Overview](docs/nist-pqc-overview.md) | Technical deep-dive into FIPS 203, 204, 205 |
| [Migration Guide](docs/migration-guide.md) | Step-by-step walkthrough of using the plugin |
| [Language Support](docs/language-support.md) | PQC library availability per language |
| [Plugin README](plugins/quantum-secure/README.md) | Plugin architecture, commands, and limitations |

---

## How It Works

The quantum-secure plugin orchestrates three specialized AI agents:

```
/quantum-secure full
        │
        ▼
┌──────────────┐     Findings     ┌──────────────┐     Report     ┌──────────────┐
│   SCANNER    │ ──────────────▶  │   MIGRATOR   │ ────────────▶  │  VALIDATOR   │
│  (read-only) │                  │ (writes code) │                │ (read + test) │
└──────────────┘                  └──────────────┘                └──────────────┘
       │                                 │                               │
  Detects RSA,                   Rewrites to ML-KEM,            Re-scans, builds,
  ECDSA, ECDH,                   ML-DSA, SLH-DSA.              runs tests, flags
  DH, AES-128,                   Adds PQC libraries.           custom code for
  MD5, SHA-1...                  Writes FFI bindings            security audit.
                                 when needed.
```

Each agent has deep knowledge of:
- **10+ programming languages** and their crypto idioms
- **NIST FIPS 203/204/205** algorithm specifications and parameter sets
- **PQC library ecosystem** across all major languages
- **Hybrid migration patterns** for TLS and protocol contexts

---

## Why This Marketplace Exists

The cryptographic community has done its part -- the standards are published, the reference implementations are available, and major tech companies are deploying PQC in production. But most codebases still run entirely on classical crypto. The gap is not knowledge -- it's action.

This marketplace exists to close that gap. One command. Any language. Quantum-secure.

**Every day you wait is another day of vulnerable data.**

---

## Contributing

Contributions welcome. If you'd like to:

- **Add PQC library support for a new language**: Submit a PR updating the library matrix and detection patterns
- **Improve detection patterns**: Add regex patterns for crypto usage we're not catching
- **Build new plugins**: This marketplace can host additional quantum-security tools
- **Report issues**: Open an issue with the scan report and language details

---

## License

Apache 2.0 -- See [LICENSE](LICENSE)

---

## References

- [NIST FIPS 203 - ML-KEM](https://csrc.nist.gov/pubs/fips/203/final)
- [NIST FIPS 204 - ML-DSA](https://csrc.nist.gov/pubs/fips/204/final)
- [NIST FIPS 205 - SLH-DSA](https://csrc.nist.gov/pubs/fips/205/final)
- [NSM-10 - National Security Memorandum on Quantum Computing](https://www.whitehouse.gov/briefing-room/statements-releases/2022/05/04/national-security-memorandum-on-promoting-united-states-leadership-in-quantum-computing-while-mitigating-risks-to-vulnerable-cryptographic-systems/)
- [CISA Post-Quantum Cryptography Initiative](https://www.cisa.gov/quantum)
- [Open Quantum Safe Project (liboqs)](https://openquantumsafe.org/)
- [Cloudflare circl Library](https://github.com/cloudflare/circl)
