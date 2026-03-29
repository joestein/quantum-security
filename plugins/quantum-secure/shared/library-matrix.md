# Post-Quantum Cryptography Library Matrix

This document tracks PQC library availability per programming language. The migrator agent uses this to decide whether to use an existing library or write custom FFI bindings.

**Last updated**: 2026-03-28

---

## Library Availability Matrix

| Language | ML-KEM (FIPS 203) | ML-DSA (FIPS 204) | SLH-DSA (FIPS 205) | Primary Library | Install Command | Maturity |
|---|---|---|---|---|---|---|
| C/C++ | Yes | Yes | Yes | **liboqs** (Open Quantum Safe) | `apt install liboqs-dev` or build from source | Production |
| Rust | Yes | Yes | Yes | **pqcrypto** | `cargo add pqcrypto pqcrypto-mlkem pqcrypto-mldsa pqcrypto-sphincsplus` | High |
| Python | Yes | Yes | Yes | **oqs-python** | `pip install oqs` | High |
| Go | Yes | Yes | Partial | **circl** (Cloudflare) | `go get github.com/cloudflare/circl` | High |
| Java | Yes | Yes | Yes | **Bouncy Castle** | Maven: `org.bouncycastle:bcprov-jdk18on:1.78+` | Production |
| C# | Yes | Yes | Yes | **BouncyCastle.Cryptography** | `dotnet add package BouncyCastle.Cryptography` | High |
| JavaScript/TS | Partial | Partial | No | **crystals-kyber** (npm) | `npm install crystals-kyber` | Medium |
| Ruby | No | No | No | -- | FFI to liboqs | None |
| PHP | No | No | No | -- | FFI to liboqs | None |
| Swift | Partial | No | No | **CryptoKit** (Apple) | Built-in (iOS 17.4+, macOS 14.4+) | Medium |
| Kotlin/JVM | Yes | Yes | Yes | **Bouncy Castle** (via JVM) | Same as Java | Production |

---

## Detailed Library Information

### C/C++ -- liboqs

**Repository**: github.com/open-quantum-safe/liboqs

The reference implementation. All other language bindings typically wrap liboqs.

```c
#include <oqs/oqs.h>

// ML-KEM key encapsulation
OQS_KEM *kem = OQS_KEM_new(OQS_KEM_alg_ml_kem_768);
OQS_KEM_keypair(kem, public_key, secret_key);
OQS_KEM_encaps(kem, ciphertext, shared_secret, public_key);
OQS_KEM_decaps(kem, shared_secret, ciphertext, secret_key);

// ML-DSA signing
OQS_SIG *sig = OQS_SIG_new(OQS_SIG_alg_ml_dsa_65);
OQS_SIG_keypair(sig, public_key, secret_key);
OQS_SIG_sign(sig, signature, &sig_len, message, msg_len, secret_key);
OQS_SIG_verify(sig, message, msg_len, signature, sig_len, public_key);
```

**Algorithm identifiers**:
- `OQS_KEM_alg_ml_kem_512`, `OQS_KEM_alg_ml_kem_768`, `OQS_KEM_alg_ml_kem_1024`
- `OQS_SIG_alg_ml_dsa_44`, `OQS_SIG_alg_ml_dsa_65`, `OQS_SIG_alg_ml_dsa_87`
- `OQS_SIG_alg_sphincs_sha2_128s_simple`, `OQS_SIG_alg_sphincs_sha2_128f_simple`, etc.

---

### Rust -- pqcrypto

**Crates**: `pqcrypto-mlkem`, `pqcrypto-mldsa`, `pqcrypto-sphincsplus`

```rust
use pqcrypto_mlkem::mlkem768;
use pqcrypto_traits::kem::{PublicKey, SecretKey, SharedSecret, Ciphertext};

// ML-KEM key encapsulation
let (pk, sk) = mlkem768::keypair();
let (ss_sender, ct) = mlkem768::encapsulate(&pk);
let ss_receiver = mlkem768::decapsulate(&ct, &sk);

use pqcrypto_mldsa::mldsa65;
use pqcrypto_traits::sign::{SignedMessage, DetachedSignature};

// ML-DSA signing
let (pk, sk) = mldsa65::keypair();
let sm = mldsa65::sign(message, &sk);
let verified_msg = mldsa65::open(&sm, &pk).unwrap();
```

---

### Python -- oqs-python

**Package**: `oqs` (PyPI)

```python
import oqs

# ML-KEM key encapsulation
kem = oqs.KeyEncapsulation("ML-KEM-768")
public_key = kem.generate_keypair()
ciphertext, shared_secret_sender = kem.encap_secret(public_key)
shared_secret_receiver = kem.decap_secret(ciphertext)

# ML-DSA signing
sig = oqs.Signature("ML-DSA-65")
public_key = sig.generate_keypair()
signature = sig.sign(message)
is_valid = sig.verify(message, signature, public_key)
```

---

### Go -- circl (Cloudflare)

**Module**: `github.com/cloudflare/circl`

```go
import (
    "github.com/cloudflare/circl/kem/mlkem/mlkem768"
    "github.com/cloudflare/circl/sign/mldsa/mldsa65"
)

// ML-KEM key encapsulation
pk, sk, _ := mlkem768.GenerateKeyPair(rand.Reader)
ct, ssS, _ := mlkem768.Encapsulate(rand.Reader, pk)
ssR, _ := mlkem768.Decapsulate(sk, ct)

// ML-DSA signing
pub, priv, _ := mldsa65.GenerateKey(rand.Reader)
sig, _ := mldsa65.Sign(priv, message, nil)
valid := mldsa65.Verify(pub, message, nil, sig)
```

**Note**: SLH-DSA support in circl is partial. For full SLH-DSA, use `github.com/nicpottier/liboqs-go` or call liboqs via CGo.

---

### Java -- Bouncy Castle

**Maven**: `org.bouncycastle:bcprov-jdk18on:1.78+`

```java
import org.bouncycastle.pqc.jcajce.provider.BouncyCastlePQCProvider;
import org.bouncycastle.pqc.jcajce.spec.MLKEMParameterSpec;
import org.bouncycastle.pqc.jcajce.spec.MLDSAParameterSpec;

Security.addProvider(new BouncyCastlePQCProvider());

// ML-KEM key encapsulation
KeyPairGenerator kpg = KeyPairGenerator.getInstance("ML-KEM", "BCPQC");
kpg.initialize(MLKEMParameterSpec.ml_kem_768);
KeyPair kp = kpg.generateKeyPair();

KeyGenerator kg = KeyGenerator.getInstance("ML-KEM", "BCPQC");
kg.init(new KEMGenerateSpec(kp.getPublic(), "AES"));
SecretKeyWithEncapsulation enc = (SecretKeyWithEncapsulation) kg.generateKey();

// ML-DSA signing
KeyPairGenerator sigKpg = KeyPairGenerator.getInstance("ML-DSA", "BCPQC");
sigKpg.initialize(MLDSAParameterSpec.ml_dsa_65);
KeyPair sigKp = sigKpg.generateKeyPair();

Signature sig = Signature.getInstance("ML-DSA", "BCPQC");
sig.initSign(sigKp.getPrivate());
sig.update(message);
byte[] signature = sig.sign();
```

---

### C# -- BouncyCastle.Cryptography

**NuGet**: `BouncyCastle.Cryptography`

```csharp
using Org.BouncyCastle.Pqc.Crypto.MLKem;
using Org.BouncyCastle.Pqc.Crypto.MLDsa;

// ML-KEM key encapsulation
var kemGen = new MLKemKeyPairGenerator();
kemGen.Init(new MLKemKeyGenerationParameters(random, MLKemParameters.ML_KEM_768));
var kemKp = kemGen.GenerateKeyPair();

var encapsulator = new MLKemEncapsulator();
encapsulator.Init(kemKp.Public);
var encResult = encapsulator.Encapsulate();

// ML-DSA signing
var sigGen = new MLDsaKeyPairGenerator();
sigGen.Init(new MLDsaKeyGenerationParameters(random, MLDsaParameters.ML_DSA_65));
var sigKp = sigGen.GenerateKeyPair();

var signer = new MLDsaSigner();
signer.Init(true, sigKp.Private);
byte[] signature = signer.GenerateSignature(message);
```

---

### JavaScript/TypeScript

**npm**: `crystals-kyber` for ML-KEM

```javascript
import { KyberEncapsulate, KyberDecapsulate, KyberKeyGen } from 'crystals-kyber';

// ML-KEM (via Kyber)
const [publicKey, privateKey] = await KyberKeyGen(768);
const [ciphertext, sharedSecretSender] = await KyberEncapsulate(publicKey);
const sharedSecretReceiver = await KyberDecapsulate(ciphertext, privateKey);
```

**Note**: ML-DSA and SLH-DSA support in JavaScript is limited. Options:
1. Use `liboqs-node` (N-API bindings to liboqs) -- requires native compilation
2. Use WASM build of liboqs -- portable but larger bundle
3. For browser contexts, WASM is the only option

---

## Custom Implementation Strategy

When no library exists for a language, use this decision tree:

```
1. Does the language support C FFI?
   ├── YES: Write FFI bindings to liboqs
   │   ├── Ruby: Use `ffi` gem to call liboqs shared library
   │   ├── PHP: Use PHP FFI extension to call liboqs shared library
   │   ├── Lua: Use LuaJIT FFI
   │   └── Other: Language-specific C FFI mechanism
   └── NO: Can it load WASM?
       ├── YES: Use WASM build of liboqs
       │   ├── Browser JS: liboqs compiled to WASM
       │   └── Other WASM runtimes
       └── NO: Write a pure implementation
           └── WARNING: Custom crypto is dangerous.
               Flag for mandatory third-party audit.
               Include NIST test vectors for validation.
```

### FFI Binding Template (Ruby example)

```ruby
require 'ffi'

module LibOQS
  extend FFI::Library
  ffi_lib 'oqs'

  # KEM functions
  attach_function :OQS_KEM_new, [:string], :pointer
  attach_function :OQS_KEM_keypair, [:pointer, :pointer, :pointer], :int
  attach_function :OQS_KEM_encaps, [:pointer, :pointer, :pointer, :pointer], :int
  attach_function :OQS_KEM_decaps, [:pointer, :pointer, :pointer, :pointer], :int
  attach_function :OQS_KEM_free, [:pointer], :void

  # SIG functions
  attach_function :OQS_SIG_new, [:string], :pointer
  attach_function :OQS_SIG_keypair, [:pointer, :pointer, :pointer], :int
  attach_function :OQS_SIG_sign, [:pointer, :pointer, :pointer, :pointer, :size_t, :pointer], :int
  attach_function :OQS_SIG_verify, [:pointer, :pointer, :size_t, :pointer, :size_t, :pointer], :int
  attach_function :OQS_SIG_free, [:pointer], :void
end
```

### FFI Binding Template (PHP example)

```php
$ffi = FFI::cdef("
    typedef struct OQS_KEM OQS_KEM;
    OQS_KEM *OQS_KEM_new(const char *method_name);
    int OQS_KEM_keypair(const OQS_KEM *kem, uint8_t *public_key, uint8_t *secret_key);
    int OQS_KEM_encaps(const OQS_KEM *kem, uint8_t *ciphertext, uint8_t *shared_secret, const uint8_t *public_key);
    int OQS_KEM_decaps(const OQS_KEM *kem, uint8_t *shared_secret, const uint8_t *ciphertext, const uint8_t *secret_key);
    void OQS_KEM_free(OQS_KEM *kem);
", "liboqs.so");
```

---

## Dependency Installation Commands

Quick reference for the migrator agent when adding dependencies:

| Language | Package Manager | Command |
|---|---|---|
| C/C++ | apt / cmake | `apt install liboqs-dev` or add to CMakeLists.txt |
| Rust | Cargo | `cargo add pqcrypto-mlkem pqcrypto-mldsa pqcrypto-sphincsplus` |
| Python | pip | `pip install oqs` (add to requirements.txt / pyproject.toml) |
| Go | go modules | `go get github.com/cloudflare/circl@latest` |
| Java | Maven | Add `bcprov-jdk18on` and `bcpqc-jdk18on` to pom.xml |
| Java | Gradle | `implementation 'org.bouncycastle:bcprov-jdk18on:1.78+'` |
| C# | NuGet | `dotnet add package BouncyCastle.Cryptography` |
| JS/TS | npm | `npm install crystals-kyber` |
| Ruby | gem + system | `gem install ffi` + ensure liboqs installed on system |
| PHP | composer + system | Ensure PHP FFI extension enabled + liboqs installed |
