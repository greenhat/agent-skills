# Miden Core Library

The Miden Core Library provides optimized, battle-tested procedures for common operations.

## Usage

```masm
use miden::core::math::u64

begin
    push.5.0.3.0        # Two u64 values: 5 and 3
    exec.u64::wrapping_add
end
```

---

## Available Modules

### Math Operations

#### `miden::core::math::u64`

64-bit unsigned integers represented as two 32-bit limbs `[hi, lo]` on stack.

**Arithmetic:**
| Procedure | Stack Transition | Cycles | Notes |
| --------- | ---------------- | ------ | ----- |
| `overflowing_add` | `[b_hi, b_lo, a_hi, a_lo] -> [overflow, c_hi, c_lo]` | 6 | |
| `wrapping_add` | `[b_hi, b_lo, a_hi, a_lo] -> [c_hi, c_lo]` | 7 | |
| `overflowing_sub` | `[b_hi, b_lo, a_hi, a_lo] -> [underflow, c_hi, c_lo]` | 11 | |
| `wrapping_sub` | `[b_hi, b_lo, a_hi, a_lo] -> [c_hi, c_lo]` | 10 | |
| `overflowing_mul` | `[b_hi, b_lo, a_hi, a_lo] -> [c_hi_hi, c_hi_lo, c_lo_hi, c_lo_lo]` | 18 | |
| `wrapping_mul` | `[b_hi, b_lo, a_hi, a_lo] -> [c_hi, c_lo]` | 11 | |
| `div` | `[b_hi, b_lo, a_hi, a_lo] -> [c_hi, c_lo]` | 54 | a // b |
| `mod` | `[b_hi, b_lo, a_hi, a_lo] -> [c_hi, c_lo]` | 54 | a % b |
| `divmod` | `[b_hi, b_lo, a_hi, a_lo] -> [r_hi, r_lo, q_hi, q_lo]` | 54 | |

**Comparison:**
| Procedure | Stack Transition | Cycles |
| --------- | ---------------- | ------ |
| `lt` | `[b_hi, b_lo, a_hi, a_lo] -> [c]` | 11 |
| `gt` | `[b_hi, b_lo, a_hi, a_lo] -> [c]` | 11 |
| `lte` | `[b_hi, b_lo, a_hi, a_lo] -> [c]` | 12 |
| `gte` | `[b_hi, b_lo, a_hi, a_lo] -> [c]` | 12 |
| `eq` | `[b_hi, b_lo, a_hi, a_lo] -> [c]` | 6 |
| `neq` | `[b_hi, b_lo, a_hi, a_lo] -> [c]` | 6 |
| `eqz` | `[a_hi, a_lo] -> [c]` | 4 |
| `min` | `[b_hi, b_lo, a_hi, a_lo] -> [c_hi, c_lo]` | 23 |
| `max` | `[b_hi, b_lo, a_hi, a_lo] -> [c_hi, c_lo]` | 23 |

**Bitwise:**
| Procedure | Stack Transition | Cycles |
| --------- | ---------------- | ------ |
| `and` | `[b_hi, b_lo, a_hi, a_lo] -> [c_hi, c_lo]` | 6 |
| `or` | `[b_hi, b_lo, a_hi, a_lo] -> [c_hi, c_lo]` | 16 |
| `xor` | `[b_hi, b_lo, a_hi, a_lo] -> [c_hi, c_lo]` | 6 |
| `shl` | `[b, a_hi, a_lo] -> [c_hi, c_lo]` | 28 |
| `shr` | `[b, a_hi, a_lo] -> [c_hi, c_lo]` | 44 |
| `rotl` | `[b, a_hi, a_lo] -> [c_hi, c_lo]` | 35 |
| `rotr` | `[b, a_hi, a_lo] -> [c_hi, c_lo]` | 40 |
| `clz` | `[n_hi, n_lo] -> [clz]` | 43 |
| `ctz` | `[n_hi, n_lo] -> [ctz]` | 41 |
| `clo` | `[n_hi, n_lo] -> [clo]` | 42 |
| `cto` | `[n_hi, n_lo] -> [cto]` | 40 |

#### `miden::core::math::u256`

256-bit unsigned integers for large number operations.

---

### Memory Operations

#### `miden::core::mem`

| Procedure | Inputs | Outputs | Notes |
| --------- | ------ | ------- | ----- |
| `memcopy_words` | `[n, read_ptr, write_ptr]` | `[]` | Copies n words (word-aligned) |
| `memcopy_elements` | `[n, read_ptr, write_ptr]` | `[]` | Copies n elements |
| `pipe_double_words_to_memory` | `[C, B, A, write_ptr, end_ptr]` | `[C, B, A, write_ptr]` | From advice stack |
| `pipe_words_to_memory` | `[num_words, write_ptr]` | `[C, B, A, write_ptr']` | From advice stack |
| `pipe_preimage_to_memory` | `[num_words, write_ptr, COMMITMENT]` | `[write_ptr']` | With hash verification |

---

### Cryptographic Operations

#### `miden::core::crypto::hashes::rpo256`

Rescue Prime Optimized hash (native to Miden VM).

#### `miden::core::crypto::hashes::blake3`

BLAKE3 hash function.

#### `miden::core::crypto::hashes::sha256`

SHA-256 hash function.

#### `miden::core::crypto::hashes::sha512`

SHA-512 hash function.

#### `miden::core::crypto::hashes::keccak256`

Keccak-256 hash function.

#### `miden::core::crypto::aead`

Authenticated encryption with associated data using RPO hash.

---

### Digital Signatures

#### `miden::core::crypto::dsa::falcon512rpo`

RPO Falcon512 post-quantum signatures.

#### `miden::core::crypto::dsa::ecdsa_k256_keccak`

ECDSA on secp256k1 with Keccak256 hashing.

#### `miden::core::crypto::dsa::eddsa_ed25519_sha512`

EdDSA on Ed25519 with SHA512 hashing.

---

### Collections

#### `miden::core::collections::mmr`

Merkle Mountain Range operations.

#### `miden::core::collections::smt`

Sparse Merkle Tree with 4-element keys and values.

#### `miden::core::collections::sorted_array`

Searching in sorted word arrays.

---

### System Utilities

#### `miden::core::sys`

System-level utilities including `truncate_stack` for ensuring stack depth = 16 before returning from `call`/`syscall`.

#### `miden::core::sys::vm`

VM-facing utilities for recursive proof verification.

---

### Word Utilities

#### `miden::core::word`

Utilities for working with 4-element words.
