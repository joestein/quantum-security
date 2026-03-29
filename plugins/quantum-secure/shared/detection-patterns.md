# Classical Cryptography Detection Patterns

Language-specific patterns for detecting classical cryptographic usage. The scanner agent uses these patterns to identify code that needs post-quantum migration.

Each pattern includes:
- **Category**: key-exchange, signature, symmetric, hash, protocol
- **Risk**: critical (asymmetric crypto, directly broken by quantum), high (weak symmetric/hash), medium (configuration/protocol level)

---

## Python

### Imports
```regex
# Key exchange / asymmetric -- CRITICAL
from\s+cryptography\.hazmat\.primitives\.asymmetric\s+import\s+(rsa|ec|dh|x25519|x448|dsa)
from\s+Crypto\.PublicKey\s+import\s+(RSA|ECC|DSA|ElGamal)
from\s+Cryptodome\.PublicKey\s+import\s+(RSA|ECC|DSA)
import\s+rsa\b
from\s+ecdsa\s+import
from\s+nacl\.public\s+import\s+PrivateKey
from\s+paramiko.*import.*RSAKey|DSSKey|ECDSAKey

# Symmetric -- HIGH (weak ciphers)
from\s+Crypto\.Cipher\s+import\s+(DES3|DES|Blowfish|ARC4)
from\s+Cryptodome\.Cipher\s+import\s+(DES3|DES|Blowfish|ARC4)

# Hash -- HIGH (broken)
from\s+hashlib\s+import\s+(md5|sha1)
import\s+hashlib.*\.(md5|sha1)\(
```

### Function Calls
```regex
# Key generation -- CRITICAL
rsa\.generate_private_key\(
ec\.generate_private_key\(
dh\.generate_parameters\(
x25519\.X25519PrivateKey\.generate\(
dsa\.generate_private_key\(
RSA\.generate\(
RSA\.construct\(
ECC\.generate\(
generate_private_key.*key_size\s*=\s*(1024|2048|4096)

# Symmetric -- HIGH
DES3\.new\(
DES\.new\(
Blowfish\.new\(
ARC4\.new\(
AES\.new\(.*key.*(?:16|b'.{16}')  # AES-128 detection

# Hash -- HIGH
hashlib\.md5\(
hashlib\.sha1\(
hashlib\.new\(\s*['"]md5['"]\)
hashlib\.new\(\s*['"]sha1['"]\)
```

---

## JavaScript / TypeScript

### Imports & Requires
```regex
# Node.js crypto -- CRITICAL
require\(\s*['"]crypto['"]\)
import\s+.*\s+from\s+['"]crypto['"]
require\(\s*['"]node-rsa['"]\)
require\(\s*['"]elliptic['"]\)
require\(\s*['"]jsonwebtoken['"]\)
import\s+.*\s+from\s+['"]jose['"]

# Web Crypto API -- CRITICAL
crypto\.subtle\.(generateKey|importKey|deriveKey|deriveBits)
window\.crypto\.subtle
```

### Function Calls
```regex
# Key generation -- CRITICAL
crypto\.generateKeyPairSync\(\s*['"]rsa['"]
crypto\.generateKeyPairSync\(\s*['"]dsa['"]
crypto\.generateKeyPairSync\(\s*['"]ec['"]
crypto\.generateKeyPair\(\s*['"]rsa['"]
crypto\.createDiffieHellman\(
crypto\.createDiffieHellmanGroup\(
crypto\.createECDH\(
crypto\.createSign\(\s*['"]RSA
crypto\.createVerify\(\s*['"]RSA
subtle\.generateKey\(.*['"]RSA
subtle\.generateKey\(.*['"]ECDSA
subtle\.generateKey\(.*['"]ECDH

# Symmetric -- HIGH
crypto\.createCipheriv\(\s*['"]des
crypto\.createCipheriv\(\s*['"]aes-128
crypto\.createCipheriv\(\s*['"]bf
crypto\.createCipheriv\(\s*['"]rc4

# Hash -- HIGH
crypto\.createHash\(\s*['"]md5['"]\)
crypto\.createHash\(\s*['"]sha1['"]\)

# JWT -- CRITICAL (often RSA/ECDSA signed)
jwt\.sign\(.*algorithm.*['"]RS(256|384|512)['"]
jwt\.sign\(.*algorithm.*['"]ES(256|384|512)['"]
jwt\.sign\(.*algorithm.*['"]PS(256|384|512)['"]
```

---

## Go

### Imports
```regex
# Asymmetric -- CRITICAL
"crypto/rsa"
"crypto/ecdsa"
"crypto/ecdh"
"crypto/dsa"
"crypto/ed25519"
"crypto/elliptic"
"golang\.org/x/crypto/curve25519"
"golang\.org/x/crypto/ssh"

# Symmetric -- HIGH
"crypto/des"
"crypto/rc4"

# Hash -- HIGH
"crypto/md5"
"crypto/sha1"
```

### Function Calls
```regex
# Key generation -- CRITICAL
rsa\.GenerateKey\(
ecdsa\.GenerateKey\(
ecdh\.P256\(\)|ecdh\.P384\(\)|ecdh\.P521\(\)
ed25519\.GenerateKey\(
dsa\.GenerateParameters\(
elliptic\.(P256|P384|P521)\(\)
x509\.CreateCertificate\(

# Symmetric -- HIGH
des\.NewCipher\(
des\.NewTripleDESCipher\(
rc4\.NewCipher\(
aes\.NewCipher\(.*\[\s*16\s*\]byte  # AES-128

# Hash -- HIGH
md5\.New\(\)
md5\.Sum\(
sha1\.New\(\)
sha1\.Sum\(
```

---

## Rust

### Imports (use statements)
```regex
# Asymmetric -- CRITICAL
use\s+rsa::
use\s+ring::agreement
use\s+ring::signature.*(rsa|ecdsa|ed25519)
use\s+p256::
use\s+p384::
use\s+x25519_dalek
use\s+ed25519_dalek
use\s+openssl::rsa
use\s+openssl::ec
use\s+openssl::dsa
use\s+openssl::dh

# Symmetric -- HIGH
use\s+des::
use\s+blowfish::

# Hash -- HIGH
use\s+md5::
use\s+sha1::
use\s+md-5::
use\s+sha-1::
```

### Function Calls
```regex
# Key generation -- CRITICAL
RsaPrivateKey::new\(
RsaKeyPair::generate\(
EcdsaKeyPair::generate\(
agreement::EphemeralPrivateKey::generate\(
SigningKey::generate\(
SecretKey::new\(  # x25519_dalek

# Cargo.toml dependencies -- CRITICAL
\b(rsa|p256|p384|x25519-dalek|ed25519-dalek|ecdsa)\s*=
```

---

## Java

### Imports
```regex
# Asymmetric -- CRITICAL
import\s+java\.security\.KeyPairGenerator
import\s+java\.security\.interfaces\.(RSA|EC|DSA)
import\s+javax\.crypto\.KeyAgreement
import\s+java\.security\.Signature
import\s+java\.security\.cert\.
import\s+org\.bouncycastle\.jce\.provider

# Symmetric -- HIGH
import\s+javax\.crypto\.spec\.DESKeySpec
import\s+javax\.crypto\.spec\.DESedeKeySpec

# Hash -- HIGH
import\s+java\.security\.MessageDigest
```

### Function Calls
```regex
# Key generation -- CRITICAL
KeyPairGenerator\.getInstance\(\s*["']RSA["']\)
KeyPairGenerator\.getInstance\(\s*["']EC["']\)
KeyPairGenerator\.getInstance\(\s*["']DSA["']\)
KeyPairGenerator\.getInstance\(\s*["']DiffieHellman["']\)
KeyAgreement\.getInstance\(\s*["']ECDH["']\)
KeyAgreement\.getInstance\(\s*["']DH["']\)
Signature\.getInstance\(\s*["']SHA\d+withRSA["']\)
Signature\.getInstance\(\s*["']SHA\d+withECDSA["']\)
Signature\.getInstance\(\s*["']SHA\d+withDSA["']\)

# Symmetric -- HIGH
Cipher\.getInstance\(\s*["']DES
Cipher\.getInstance\(\s*["']DESede
Cipher\.getInstance\(\s*["']Blowfish
Cipher\.getInstance\(\s*["']RC4
Cipher\.getInstance\(\s*["']AES/.*["']\).*128  # AES-128

# Hash -- HIGH
MessageDigest\.getInstance\(\s*["']MD5["']\)
MessageDigest\.getInstance\(\s*["']SHA-1["']\)
```

---

## C / C++

### Includes
```regex
# OpenSSL -- CRITICAL
#include\s*[<"]openssl/rsa\.h[">]
#include\s*[<"]openssl/ec\.h[">]
#include\s*[<"]openssl/ecdsa\.h[">]
#include\s*[<"]openssl/ecdh\.h[">]
#include\s*[<"]openssl/dsa\.h[">]
#include\s*[<"]openssl/dh\.h[">]
#include\s*[<"]openssl/evp\.h[">]
#include\s*[<"]openssl/pem\.h[">]

# Symmetric -- HIGH
#include\s*[<"]openssl/des\.h[">]
#include\s*[<"]openssl/blowfish\.h[">]
#include\s*[<"]openssl/rc4\.h[">]

# Hash -- HIGH
#include\s*[<"]openssl/md5\.h[">]
#include\s*[<"]openssl/sha\.h[">]
```

### Function Calls
```regex
# Key generation -- CRITICAL
RSA_generate_key(_ex)?\(
EC_KEY_generate_key\(
DH_generate_key\(
DH_generate_parameters(_ex)?\(
DSA_generate_key\(
DSA_generate_parameters(_ex)?\(
EVP_PKEY_keygen\(
EVP_PKEY_CTX_new_id\(\s*EVP_PKEY_(RSA|EC|DH|DSA)

# Signing -- CRITICAL
RSA_sign\(
ECDSA_sign\(
DSA_sign\(
EVP_DigestSign(Init|Update|Final)\(

# Symmetric -- HIGH
DES_ecb_encrypt\(
DES_ede3_cbc_encrypt\(
BF_encrypt\(
RC4\(
EVP_EncryptInit_ex\(.*EVP_(des|bf|rc4)

# Hash -- HIGH
MD5_Init\(|MD5_Update\(|MD5_Final\(|MD5\(
SHA1_Init\(|SHA1_Update\(|SHA1_Final\(|SHA1\(
```

---

## C# (.NET)

### Imports
```regex
# Asymmetric -- CRITICAL
using\s+System\.Security\.Cryptography\b
using\s+.*RSA\b
using\s+.*ECDsa\b
using\s+.*ECDiffieHellman\b
using\s+.*DSA\b
```

### Function Calls
```regex
# Key generation -- CRITICAL
RSA\.Create\(
ECDsa\.Create\(
ECDiffieHellman\.Create\(
DSA\.Create\(
RSACryptoServiceProvider\(
DSACryptoServiceProvider\(
CngKey\.Create\(.*ECDsa
CngKey\.Create\(.*ECDiffieHellman

# Symmetric -- HIGH
DES\.Create\(
TripleDES\.Create\(
RC2\.Create\(
DESCryptoServiceProvider\(
TripleDESCryptoServiceProvider\(
Aes\.Create\(.*128  # AES-128

# Hash -- HIGH
MD5\.Create\(
SHA1\.Create\(
MD5CryptoServiceProvider\(
SHA1CryptoServiceProvider\(
```

---

## Ruby

### Requires & Usage
```regex
# Asymmetric -- CRITICAL
require\s+['"]openssl['"]
OpenSSL::PKey::RSA\.new
OpenSSL::PKey::RSA\.generate
OpenSSL::PKey::EC\.new
OpenSSL::PKey::EC\.generate
OpenSSL::PKey::DH\.(new|generate)
OpenSSL::PKey::DSA\.(new|generate)

# Symmetric -- HIGH
OpenSSL::Cipher\.new\(\s*['"]DES
OpenSSL::Cipher\.new\(\s*['"]BF
OpenSSL::Cipher\.new\(\s*['"]RC4
OpenSSL::Cipher\.new\(\s*['"]AES-128

# Hash -- HIGH
Digest::MD5
Digest::SHA1
OpenSSL::Digest::MD5
OpenSSL::Digest::SHA1
```

---

## PHP

### Function Calls
```regex
# Asymmetric -- CRITICAL
openssl_pkey_new\(.*OPENSSL_KEYTYPE_(RSA|DSA|DH|EC)
openssl_sign\(
openssl_verify\(
openssl_dh_compute_key\(
openssl_pkey_get_details\(

# Symmetric -- HIGH
openssl_encrypt\(.*['"]DES
openssl_encrypt\(.*['"]BF
openssl_encrypt\(.*['"]RC4
openssl_encrypt\(.*['"]AES-128
mcrypt_encrypt\(  # deprecated but still found

# Hash -- HIGH
md5\(
sha1\(
hash\(\s*['"]md5['"]
hash\(\s*['"]sha1['"]
```

---

## Configuration & Protocol Files

### TLS Cipher Suites -- MEDIUM
```regex
# In any config file (.conf, .yml, .yaml, .json, .toml, .ini, .cfg)
TLS_RSA_WITH_
TLS_DHE_RSA_
TLS_ECDHE_RSA_
TLS_ECDHE_ECDSA_
ssl_ciphers\s+
cipher[_-]?suites?\s*[=:]
```

### SSH Configuration -- MEDIUM
```regex
# In sshd_config, ssh_config, known_hosts, authorized_keys
HostKeyAlgorithms.*ssh-rsa
PubkeyAcceptedAlgorithms.*ssh-rsa
ssh-rsa\s+AAAA
ssh-dss\s+AAAA
ecdsa-sha2-nistp(256|384|521)
```

### Certificate Files -- MEDIUM
```regex
# File extensions to flag
\.(pem|crt|cer|der|p12|pfx|key)$
# Content patterns in PEM files
BEGIN RSA PRIVATE KEY
BEGIN EC PRIVATE KEY
BEGIN DSA PRIVATE KEY
BEGIN DH PARAMETERS
```

### Docker / Infrastructure -- MEDIUM
```regex
# In Dockerfile, docker-compose.yml, nginx.conf, apache conf
ssl_protocols\s+
ssl_ciphers\s+
SSLProtocol\s+
SSLCipherSuite\s+
```

---

## Detection Priority

When scanning, process patterns in this order:
1. **Critical**: Asymmetric crypto (RSA, ECDSA, ECDH, DH, DSA, EdDSA) -- directly broken by quantum
2. **High**: Weak symmetric (DES, 3DES, Blowfish, RC4, AES-128) and broken hashes (MD5, SHA-1)
3. **Medium**: Protocol/configuration level (TLS suites, SSH config, certificates)

## False Positive Mitigation

To reduce false positives, the scanner should:
- Match import/require/include statements and function calls, NOT arbitrary string occurrences
- Skip comments and documentation strings when possible
- Verify the match is in actual code, not in test fixtures or mock data (unless scanning tests too)
- Weight import-level matches higher than string-level matches
