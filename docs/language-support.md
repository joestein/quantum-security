# Post-Quantum Cryptography Language Support

This document covers the current state of PQC library support across programming languages. The quantum-secure plugin uses this information to determine the best migration strategy for each language in your codebase.

---

## Support Matrix

| Language | ML-KEM (FIPS 203) | ML-DSA (FIPS 204) | SLH-DSA (FIPS 205) | Strategy | Maturity |
|----------|:-:|:-:|:-:|----------|----------|
| **C/C++** | Yes | Yes | Yes | liboqs (native) | Production |
| **Rust** | Yes | Yes | Yes | pqcrypto crate | High |
| **Python** | Yes | Yes | Yes | oqs-python | High |
| **Go** | Yes | Yes | Partial | circl (Cloudflare) | High |
| **Java** | Yes | Yes | Yes | Bouncy Castle | Production |
| **Kotlin (JVM)** | Yes | Yes | Yes | Bouncy Castle (JVM) | Production |
| **C# (.NET)** | Yes | Yes | Yes | BouncyCastle.Cryptography | High |
| **JavaScript/TS** | Partial | Partial | No | crystals-kyber + WASM | Medium |
| **Swift** | Partial | No | No | CryptoKit (Apple) | Medium |
| **Ruby** | No | No | No | FFI to liboqs | Custom |
| **PHP** | No | No | No | FFI to liboqs | Custom |
| **Elixir/Erlang** | No | No | No | NIF to liboqs | Custom |

---

## Tier 1: Full Native Support

These languages have mature, well-maintained PQC libraries covering all three NIST standards.

### C/C++ -- liboqs (Open Quantum Safe)

The reference implementation and the foundation most other language bindings build on.

- **Library**: [liboqs](https://github.com/open-quantum-safe/liboqs)
- **Coverage**: ML-KEM (all parameter sets), ML-DSA (all), SLH-DSA (all)
- **Install**: `apt install liboqs-dev` or CMake build from source
- **Maturity**: Production -- used as the reference by NIST
- **Integration**: Add `liboqs` to CMakeLists.txt or link with `-loqs`
- **Also provides**: OQS-OpenSSL fork for TLS integration

### Rust -- pqcrypto

- **Crates**: `pqcrypto-mlkem`, `pqcrypto-mldsa`, `pqcrypto-sphincsplus`
- **Coverage**: ML-KEM (all), ML-DSA (all), SLH-DSA (all)
- **Install**: `cargo add pqcrypto-mlkem pqcrypto-mldsa pqcrypto-sphincsplus`
- **Maturity**: High -- wraps C reference implementations with safe Rust APIs
- **Note**: Also available: `oqs-rs` for direct liboqs bindings

### Python -- oqs-python

- **Package**: `oqs` on PyPI
- **Coverage**: ML-KEM (all), ML-DSA (all), SLH-DSA (all)
- **Install**: `pip install oqs`
- **Maturity**: High -- FFI bindings to liboqs
- **Requires**: liboqs shared library installed on the system
- **Alternative**: `pqcrypto` package (pure Python, slower)

### Java -- Bouncy Castle

- **Maven**: `org.bouncycastle:bcprov-jdk18on:1.78+` and `org.bouncycastle:bcpqc-jdk18on:1.78+`
- **Coverage**: ML-KEM (all), ML-DSA (all), SLH-DSA (all)
- **Install**: Add Maven/Gradle dependency
- **Maturity**: Production -- Bouncy Castle is the de facto standard Java crypto library
- **Note**: Requires registering `BouncyCastlePQCProvider` as a JCE provider

### C# (.NET) -- BouncyCastle.Cryptography

- **NuGet**: `BouncyCastle.Cryptography`
- **Coverage**: ML-KEM (all), ML-DSA (all), SLH-DSA (all)
- **Install**: `dotnet add package BouncyCastle.Cryptography`
- **Maturity**: High -- mirrors the Java Bouncy Castle implementation
- **Namespace**: `Org.BouncyCastle.Pqc.Crypto.MLKem`, `...MLDsa`, `...SphincsPlus`

---

## Tier 2: Partial Native Support

These languages have PQC libraries for some but not all NIST standards.

### Go -- circl (Cloudflare)

- **Module**: `github.com/cloudflare/circl`
- **Coverage**: ML-KEM (all parameter sets), ML-DSA (all parameter sets), SLH-DSA (limited)
- **Install**: `go get github.com/cloudflare/circl@latest`
- **Maturity**: High -- maintained by Cloudflare, used in production TLS
- **Gap**: SLH-DSA support may be incomplete. Fallback: `liboqs-go` (CGo bindings)
- **Note**: circl is also used in the Go standard library's experimental PQC support

### JavaScript/TypeScript

- **ML-KEM**: `crystals-kyber` (npm) -- medium maturity
- **ML-DSA**: Limited -- `liboqs-node` (N-API) or WASM liboqs build
- **SLH-DSA**: No native library -- WASM liboqs required
- **Install**: `npm install crystals-kyber` for ML-KEM
- **Maturity**: Medium -- Kyber has decent npm packages; ML-DSA and SLH-DSA require native/WASM bindings
- **Browser support**: WASM liboqs is the primary option
- **Note**: The Web Crypto API does not yet support PQC algorithms. W3C/IETF work is in progress.

### Swift -- CryptoKit (Apple)

- **Framework**: CryptoKit (built into iOS/macOS)
- **Coverage**: ML-KEM (iOS 17.4+, macOS 14.4+), no ML-DSA or SLH-DSA yet
- **Install**: Built-in -- no package needed
- **Maturity**: Medium -- Apple's first-party implementation but limited algorithm coverage
- **Gap**: ML-DSA and SLH-DSA require third-party libraries or C FFI

---

## Tier 3: Custom Implementation Required

These languages lack PQC libraries. The quantum-secure plugin writes FFI bindings to liboqs.

### Ruby

- **Native PQC library**: None
- **Strategy**: FFI to liboqs using the `ffi` gem
- **Requirements**:
  - `gem install ffi`
  - liboqs installed on the system (`apt install liboqs-dev`)
- **What the plugin generates**: A `LibOQS` Ruby module with FFI bindings for KEM and signature operations
- **Limitation**: Requires liboqs as a system dependency

### PHP

- **Native PQC library**: None
- **Strategy**: FFI to liboqs using PHP's built-in FFI extension
- **Requirements**:
  - PHP 7.4+ with FFI extension enabled (`extension=ffi` in php.ini)
  - liboqs installed on the system
- **What the plugin generates**: PHP FFI class wrapping liboqs KEM and signature functions
- **Limitation**: PHP FFI has performance overhead; not ideal for high-throughput crypto

### Elixir/Erlang

- **Native PQC library**: None
- **Strategy**: NIF (Native Implemented Function) calling liboqs via C
- **Requirements**: C compiler, liboqs headers and library
- **What the plugin generates**: A NIF module wrapping liboqs operations
- **Limitation**: NIFs can crash the BEAM VM if implemented incorrectly -- extra audit scrutiny needed

---

## System Dependency: liboqs

Many language bindings depend on **liboqs** being installed at the system level. Here's how to install it:

### Debian/Ubuntu
```bash
apt install liboqs-dev
```

### macOS (Homebrew)
```bash
brew install liboqs
```

### From Source
```bash
git clone https://github.com/open-quantum-safe/liboqs.git
cd liboqs
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr/local ..
make -j$(nproc)
sudo make install
```

### Docker
```dockerfile
FROM ubuntu:24.04
RUN apt update && apt install -y liboqs-dev
```

### Verification
```bash
# Check library is installed
ls /usr/local/lib/liboqs.* || ls /usr/lib/x86_64-linux-gnu/liboqs.*

# Check headers are available
ls /usr/local/include/oqs/oqs.h || ls /usr/include/oqs/oqs.h
```

---

## Language-Specific Migration Notes

### Python
- If using `cryptography` library (hazmat primitives), the PQC migration is straightforward with oqs-python
- If using `PyCryptodome`, switch to oqs-python for PQC operations
- Virtual environments: add `oqs` to requirements.txt or pyproject.toml

### JavaScript/TypeScript
- Server-side (Node.js): prefer `liboqs-node` N-API bindings for performance
- Client-side (browser): must use WASM build of liboqs
- The `crystals-kyber` npm package covers ML-KEM but not signatures
- JWT libraries (jsonwebtoken, jose) don't yet support PQC algorithms -- flag for deferred migration

### Go
- Cloudflare's `circl` is the go-to library, used in production at scale
- For SLH-DSA, consider `liboqs-go` which provides full algorithm coverage via CGo
- The Go standard library may add native PQC support in future releases

### Rust
- The `pqcrypto` family of crates provides comprehensive coverage
- For integration with `ring` or `rustls`, check compatibility -- PQC support is being added
- The Rust ecosystem has strong compile-time safety that helps prevent crypto implementation errors

### Java
- Bouncy Castle is the standard choice and covers all three NIST algorithms
- Register the PQC provider: `Security.addProvider(new BouncyCastlePQCProvider())`
- Works seamlessly with JCA/JCE patterns familiar to Java developers
- Android: Check Bouncy Castle version compatibility with your min SDK

### C#
- BouncyCastle.Cryptography NuGet package mirrors the Java BC implementation
- Integrates with standard .NET patterns
- .NET MAUI/Xamarin: verify NuGet package compatibility with target platforms

### C/C++
- liboqs is the canonical implementation -- no wrappers needed
- For OpenSSL-based applications, use the OQS-OpenSSL provider for transparent PQC
- CMake integration: `find_package(liboqs REQUIRED)` then `target_link_libraries(... OQS::oqs)`

---

## Future Outlook

The PQC library ecosystem is rapidly maturing:

- **2024-2025**: Initial library releases aligned with final NIST standards
- **2025-2026**: Major language runtimes integrating PQC (Go stdlib, .NET, Java JDK)
- **2026-2027**: Web platform support (Web Crypto API PQC extensions)
- **2027+**: Legacy languages (Ruby, PHP) likely to get native PQC libraries

**Don't wait for perfect library support.** FFI bindings to liboqs work today and can be replaced with native libraries as they become available. The important thing is to start the migration now.
