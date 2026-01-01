# Miden VM Instruction Reference

## Table of Contents
- [Field Operations](#field-operations)
- [U32 Operations](#u32-operations)
- [Stack Manipulation](#stack-manipulation)
- [Input/Output Operations](#inputoutput-operations)
- [Cryptographic Operations](#cryptographic-operations)
- [Flow Control Operations](#flow-control-operations)
- [Events and Debugging](#events-and-debugging)

## Terms and Notations
- $p$ = modulus of VM's base field = $2^{64} - 2^{32} + 1$
- A *word* = group of four field elements
- Lower-case letters (a, b) = individual elements
- Upper-case letters (A, B) = words
- $a_0$ = first element of word A

---

## Field Operations

### Comparison Operations

| Instruction          | Stack Input   | Stack Output     | Cycles       | Notes                                        |
| -------------------- | ------------- | ---------------- | ------------ | -------------------------------------------- |
| `lte` / `lte.b`      | `[b, a, ...]` | `[c, ...]`       | 15 / 16      | c=1 if a <= b, else 0                        |
| `lt` / `lt.b`        | `[b, a, ...]` | `[c, ...]`       | 14 / 15      | c=1 if a < b, else 0                         |
| `gte` / `gte.b`      | `[b, a, ...]` | `[c, ...]`       | 16 / 17      | c=1 if a >= b, else 0                        |
| `gt` / `gt.b`        | `[b, a, ...]` | `[c, ...]`       | 15 / 16      | c=1 if a > b, else 0                         |
| `eq` / `eq.b`        | `[b, a, ...]` | `[c, ...]`       | 1 / 1-2      | c=1 if a = b, else 0                         |
| `neq` / `neq.b`      | `[b, a, ...]` | `[c, ...]`       | 2 / 2-3      | c=1 if a != b, else 0                        |
| `eqw`                | `[A, B, ...]` | `[c, A, B, ...]` | 15           | c=1 if all elements equal                    |
| `is_odd`             | `[a, ...]`    | `[b, ...]`       | 5            | b=1 if a is odd, else 0                      |

### Assertions

| Instruction  | Stack Input   | Stack Output | Cycles | Notes                                           |
| ------------ | ------------- | ------------ | ------ | ----------------------------------------------- |
| `assert`     | `[a, ...]`    | `[...]`      | 1      | Removes a if a=1. Fails if a!=1.                |
| `assertz`    | `[a, ...]`    | `[...]`      | 2      | Removes a if a=0. Fails if a!=0.                |
| `assert_eq`  | `[b, a, ...]` | `[...]`      | 2      | Removes a,b if a=b. Fails if a!=b.              |
| `assert_eqw` | `[B, A, ...]` | `[...]`      | 11     | Removes A,B if A=B. Fails if A!=B.              |

Assertions can be parameterized: `assert.err="Division by 0"`

### Arithmetic and Boolean

| Instruction              | Stack Input   | Stack Output | Cycles    | Notes                                    |
| ------------------------ | ------------- | ------------ | --------- | ---------------------------------------- |
| `add` / `add.b`          | `[b, a, ...]` | `[c, ...]`   | 1 / 1-2   | c = (a + b) mod p                        |
| `sub` / `sub.b`          | `[b, a, ...]` | `[c, ...]`   | 2 / 2     | c = (a - b) mod p                        |
| `mul` / `mul.b`          | `[b, a, ...]` | `[c, ...]`   | 1 / 2     | c = (a * b) mod p                        |
| `div` / `div.b`          | `[b, a, ...]` | `[c, ...]`   | 2 / 2     | c = (a * b^-1) mod p. Fails if b=0.      |
| `neg`                    | `[a, ...]`    | `[b, ...]`   | 1         | b = -a mod p                             |
| `inv`                    | `[a, ...]`    | `[b, ...]`   | 1         | b = a^-1 mod p. Fails if a=0.            |
| `pow2`                   | `[a, ...]`    | `[b, ...]`   | 16        | b = 2^a. Fails if a > 63.                |
| `exp.uxx` / `exp.b`      | `[b, a, ...]` | `[c, ...]`   | 9+xx      | c = a^b. `exp` is `exp.u64` (73 cycles). |
| `ilog2`                  | `[a, ...]`    | `[b, ...]`   | 44        | b = floor(log2(a)). Fails if a=0.        |
| `not`                    | `[a, ...]`    | `[b, ...]`   | 1         | b = 1 - a. Fails if a > 1.               |
| `and`                    | `[b, a, ...]` | `[c, ...]`   | 1         | c = a * b. Fails if max(a,b) > 1.        |
| `or`                     | `[b, a, ...]` | `[c, ...]`   | 1         | c = a + b - a*b. Fails if max(a,b) > 1.  |
| `xor`                    | `[b, a, ...]` | `[c, ...]`   | 7         | c = a + b - 2*a*b. Fails if max(a,b) > 1.|

### Extension Field Operations

Operations over quadratic extension field with modulus $p = 2^{64} - 2^{32} + 1$.

| Instruction | Stack Input             | Stack Output      | Cycles | Notes                     |
| ----------- | ----------------------- | ----------------- | ------ | ------------------------- |
| `ext2add`   | `[b1, b0, a1, a0, ...]` | `[c1, c0, ...]`   | 5      | Element-wise addition     |
| `ext2sub`   | `[b1, b0, a1, a0, ...]` | `[c1, c0, ...]`   | 7      | Element-wise subtraction  |
| `ext2mul`   | `[b1, b0, a1, a0, ...]` | `[c1, c0, ...]`   | 3      | Extension field multiply  |
| `ext2neg`   | `[a1, a0, ...]`         | `[a1', a0', ...]` | 4      | Negate both elements      |
| `ext2inv`   | `[a1, a0, ...]`         | `[a1', a0', ...]` | 8      | Inverse. Fails if a=0.    |
| `ext2div`   | `[b1, b0, a1, a0, ...]` | `[c1, c0, ...]`   | 11     | Division. Fails if b=0.   |

---

## U32 Operations

Operations on 32-bit integers. Most fail or have undefined behavior if inputs are not valid u32.

### Conversions and Tests

| Instruction  | Stack Input  | Stack Output  | Cycles | Notes                                    |
| ------------ | ------------ | ------------- | ------ | ---------------------------------------- |
| `u32test`    | `[a, ...]`   | `[b, a, ...]` | 5      | b=1 if a < 2^32, else 0                  |
| `u32testw`   | `[A, ...]`   | `[b, A, ...]` | 23     | b=1 if all elements < 2^32              |
| `u32assert`  | `[a, ...]`   | `[a, ...]`    | 3      | Fails if a >= 2^32                       |
| `u32assert2` | `[b, a,...]` | `[b, a,...]`  | 1      | Fails if a >= 2^32 or b >= 2^32          |
| `u32assertw` | `[A, ...]`   | `[A, ...]`    | 6      | Fails if any element >= 2^32             |
| `u32cast`    | `[a, ...]`   | `[b, ...]`    | 2      | b = a mod 2^32                           |
| `u32split`   | `[a, ...]`   | `[c, b, ...]` | 1      | b = a mod 2^32, c = floor(a / 2^32)      |

### Arithmetic

| Instruction                | Stack Input      | Stack Output  | Cycles | Notes                                    |
| -------------------------- | ---------------- | ------------- | ------ | ---------------------------------------- |
| `u32overflowing_add`       | `[b, a, ...]`    | `[d, c, ...]` | 1      | c=(a+b) mod 2^32, d=overflow flag        |
| `u32overflowing_add.b`     | `[a, ...]`       | `[d, c, ...]` | 2-3    | c=(a+b) mod 2^32, d=overflow flag        |
| `u32wrapping_add`          | `[b, a, ...]`    | `[c, ...]`    | 2      | c=(a+b) mod 2^32                         |
| `u32wrapping_add.b`        | `[a, ...]`       | `[c, ...]`    | 3-4    | c=(a+b) mod 2^32                         |
| `u32overflowing_add3`      | `[c, b, a, ...]` | `[e, d, ...]` | 1      | d=(a+b+c) mod 2^32, e=carry              |
| `u32wrapping_add3`         | `[c, b, a, ...]` | `[d, ...]`    | 2      | d=(a+b+c) mod 2^32                       |
| `u32overflowing_sub`       | `[b, a, ...]`    | `[d, c, ...]` | 1      | c=(a-b) mod 2^32, d=underflow flag       |
| `u32overflowing_sub.b`     | `[a, ...]`       | `[d, c, ...]` | 2-3    | c=(a-b) mod 2^32, d=underflow flag       |
| `u32wrapping_sub`          | `[b, a, ...]`    | `[c, ...]`    | 2      | c=(a-b) mod 2^32                         |
| `u32wrapping_sub.b`        | `[a, ...]`       | `[c, ...]`    | 3-4    | c=(a-b) mod 2^32                         |
| `u32overflowing_mul`       | `[b, a, ...]`    | `[d, c, ...]` | 1      | c=(a*b) mod 2^32, d=high bits            |
| `u32overflowing_mul.b`     | `[a, ...]`       | `[d, c, ...]` | 2-3    | c=(a*b) mod 2^32, d=high bits            |
| `u32wrapping_mul`          | `[b, a, ...]`    | `[c, ...]`    | 2      | c=(a*b) mod 2^32                         |
| `u32wrapping_mul.b`        | `[a, ...]`       | `[c, ...]`    | 3-4    | c=(a*b) mod 2^32                         |
| `u32overflowing_madd`      | `[b, a, c, ...]` | `[e, d, ...]` | 1      | d=(a*b+c) mod 2^32, e=high bits          |
| `u32wrapping_madd`         | `[b, a, c, ...]` | `[d, ...]`    | 2      | d=(a*b+c) mod 2^32                       |
| `u32div` / `u32div.b`      | `[b, a, ...]`    | `[c, ...]`    | 2 / 3-4| c=floor(a/b). Fails if b=0.              |
| `u32mod` / `u32mod.b`      | `[b, a, ...]`    | `[c, ...]`    | 3 / 4-5| c=a mod b. Fails if b=0.                 |
| `u32divmod` / `u32divmod.b`| `[b, a, ...]`    | `[d, c, ...]` | 1 / 2-3| c=floor(a/b), d=a mod b. Fails if b=0.   |

### Bitwise

| Instruction            | Stack Input   | Stack Output | Cycles | Notes                                     |
| ---------------------- | ------------- | ------------ | ------ | ----------------------------------------- |
| `u32and` / `u32and.b`  | `[b, a, ...]` | `[c, ...]`   | 1 / 2  | Bitwise AND. Fails if max(a,b) >= 2^32.   |
| `u32or` / `u32or.b`    | `[b, a, ...]` | `[c, ...]`   | 6 / 7  | Bitwise OR. Fails if max(a,b) >= 2^32.    |
| `u32xor` / `u32xor.b`  | `[b, a, ...]` | `[c, ...]`   | 1 / 2  | Bitwise XOR. Fails if max(a,b) >= 2^32.   |
| `u32not` / `u32not.a`  | `[a, ...]`    | `[b, ...]`   | 5 / 6  | Bitwise NOT. Fails if a >= 2^32.          |
| `u32shl` / `u32shl.b`  | `[b, a, ...]` | `[c, ...]`   | 18 / 3 | c=(a*2^b) mod 2^32. Undefined if b > 31.  |
| `u32shr` / `u32shr.b`  | `[b, a, ...]` | `[c, ...]`   | 18 / 3 | c=floor(a/2^b). Undefined if b > 31.      |
| `u32rotl` / `u32rotl.b`| `[b, a, ...]` | `[c, ...]`   | 18 / 3 | Rotate left. Undefined if b > 31.         |
| `u32rotr` / `u32rotr.b`| `[b, a, ...]` | `[c, ...]`   | 23 / 3 | Rotate right. Undefined if b > 31.        |
| `u32popcnt`            | `[a, ...]`    | `[b, ...]`   | 33     | Population count (Hamming weight).        |
| `u32clz`               | `[a, ...]`    | `[b, ...]`   | 42     | Count leading zeros.                      |
| `u32ctz`               | `[a, ...]`    | `[b, ...]`   | 34     | Count trailing zeros.                     |
| `u32clo`               | `[a, ...]`    | `[b, ...]`   | 41     | Count leading ones.                       |
| `u32cto`               | `[a, ...]`    | `[b, ...]`   | 33     | Count trailing ones.                      |

### U32 Comparison

| Instruction            | Stack Input   | Stack Output | Cycles | Notes                                     |
| ---------------------- | ------------- | ------------ | ------ | ----------------------------------------- |
| `u32lt` / `u32lt.b`    | `[b, a, ...]` | `[c, ...]`   | 3 / 4  | c=1 if a < b. Undefined if >= 2^32.       |
| `u32lte` / `u32lte.b`  | `[b, a, ...]` | `[c, ...]`   | 5 / 6  | c=1 if a <= b. Undefined if >= 2^32.      |
| `u32gt` / `u32gt.b`    | `[b, a, ...]` | `[c, ...]`   | 4 / 5  | c=1 if a > b. Undefined if >= 2^32.       |
| `u32gte` / `u32gte.b`  | `[b, a, ...]` | `[c, ...]`   | 4 / 5  | c=1 if a >= b. Undefined if >= 2^32.      |
| `u32min` / `u32min.b`  | `[b, a, ...]` | `[c, ...]`   | 8 / 9  | c=min(a,b). Undefined if >= 2^32.         |
| `u32max` / `u32max.b`  | `[b, a, ...]` | `[c, ...]`   | 9 / 10 | c=max(a,b). Undefined if >= 2^32.         |

---

## Stack Manipulation

Only top 16 elements are directly accessible.

| Instruction | Stack Input         | Stack Output        | Cycles | Notes                                                  |
| ----------- | ------------------- | ------------------- | ------ | ------------------------------------------------------ |
| `drop`      | `[a, ... ]`         | `[ ... ]`           | 1      | Deletes top item.                                      |
| `dropw`     | `[A, ... ]`         | `[ ... ]`           | 4      | Deletes top word (4 elements).                         |
| `padw`      | `[ ... ]`           | `[0,0,0,0, ... ]`   | 4      | Pushes four 0 values.                                  |
| `dup.n`     | `[ ..., a, ... ]`   | `[a, ..., a, ... ]` | 1-3    | Duplicates nth item (0-indexed). `dup` = `dup.0`.      |
| `dupw.n`    | `[ ..., A, ... ]`   | `[A, ..., A, ... ]` | 4      | Duplicates nth word (0-indexed). `dupw` = `dupw.0`.    |
| `swap.n`    | `[a, ..., b, ... ]` | `[b, ..., a, ... ]` | 1-6    | Swaps top with nth item (1-indexed). `swap` = `swap.1`.|
| `swapw.n`   | `[A, ..., B, ... ]` | `[B, ..., A, ... ]` | 1      | Swaps top word with nth word (1-indexed).              |
| `swapdw`    | `[D,C,B,A, ... ]`   | `[B,A,D,C ... ]`    | 1      | Swaps words: 1st with 3rd, 2nd with 4th.               |
| `movup.n`   | `[ ..., a, ... ]`   | `[a, ... ]`         | 1-4    | Moves nth item to top (2-indexed). n in 2..=15.        |
| `movupw.n`  | `[ ..., A, ... ]`   | `[A, ... ]`         | 2-3    | Moves nth word to top (2-indexed). n in 2..=3.         |
| `movdn.n`   | `[a, ... ]`         | `[ ..., a, ... ]`   | 1-4    | Moves top to nth position (2-indexed). n in 2..=15.    |
| `movdnw.n`  | `[A, ... ]`         | `[ ..., A, ... ]`   | 2-3    | Moves top word to nth position (2-indexed). n in 2..=3.|
| `reversew`  | `[a, b, c, d, ...]` | `[d, c, b, a, ...]` | 3      | Reverses the order of top 4 elements (a word).         |
| `reversedw` | `[a,b,c,d,e,f,g,h,...]` | `[h,g,f,e,d,c,b,a,...]` | 7  | Reverses the order of top 8 elements (double word).    |

### Conditional Manipulation

| Instruction | Stack Input       | Stack Output   | Cycles | Notes                                             |
| ----------- | ----------------- | -------------- | ------ | ------------------------------------------------- |
| `cswap`     | `[c, b, a, ... ]` | `[e, d, ... ]` | 1      | If c=1: d=b,e=a. If c=0: d=a,e=b. Fails if c > 1. |
| `cswapw`    | `[c, B, A, ... ]` | `[E, D, ... ]` | 1      | Word conditional swap. Fails if c > 1.            |
| `cdrop`     | `[c, b, a, ... ]` | `[d, ... ]`    | 2      | If c=1: d=b. If c=0: d=a. Fails if c > 1.         |
| `cdropw`    | `[c, B, A, ... ]` | `[D, ... ]`    | 5      | Word conditional drop. Fails if c > 1.            |

---

## Input/Output Operations

### Constant Inputs

| Instruction | Stack Input | Stack Output     | Cycles | Notes                                         |
| ----------- | ----------- | ---------------- | ------ | --------------------------------------------- |
| `push.a...` | `[ ... ]`   | `[c, b, a, ...]` | 1-2    | Push up to 16 field elements (decimal or hex).|

Hex words (32 bytes) are little-endian; short hex values are big-endian:
```
push.0x00001234.0x00005678.0x00009012.0x0000abcd
push.0x341200000000000078560000000000001290000000000000cdab000000000000
push.4660.22136.36882.43981
```

### Environment Inputs

| Instruction    | Stack Input  | Stack Output | Cycles | Notes                                      |
| -------------- | ------------ | ------------ | ------ | ------------------------------------------ |
| `clk`          | `[ ... ]`    | `[t, ... ]`  | 1      | Current clock cycle.                       |
| `sdepth`       | `[ ... ]`    | `[d, ... ]`  | 1      | Current stack depth.                       |
| `caller`       | `[A, b,...]` | `[H, b,...]` | 1      | Hash of function that syscall'd.           |
| `locaddr.i`    | `[ ... ]`    | `[a, ... ]`  | 2      | Absolute address of local at index i.      |
| `procref.name` | `[ ... ]`    | `[A, ... ]`  | 4      | MAST root of procedure `name`.             |

### Advice Provider (Nondeterministic Inputs)

#### Reading from Advice Stack

| Instruction  | Stack Input      | Stack Output     | Cycles | Notes                                       |
| ------------ | ---------------- | ---------------- | ------ | ------------------------------------------- |
| `adv_push.n` | `[ ... ]`        | `[a, ...]`       | n      | Pop n values from advice stack (1-16).      |
| `adv_loadw`  | `[0,0,0,0, ...]` | `[A, ...]`       | 1      | Pop word from advice stack, overwrite top.  |
| `adv_pipe`   | `[C,B,A,a,...]`  | `[E,D,A,a',...]` | 1      | Pop 2 words, write to memory at a and a+1.  |

#### System Events (Advice Injection)

| Instruction             | Stack Input       | Stack Output      | Notes                                       |
| ----------------------- | ----------------- | ----------------- | ------------------------------------------- |
| `adv.push_mapval`       | `[K, ... ]`       | `[K, ... ]`       | Push values from advice_map[K].             |
| `adv.push_mapval_count` | `[K, ... ]`       | `[K, ... ]`       | Push count of elements in advice_map[K].    |
| `adv.push_mapvaln`      | `[K, ... ]`       | `[K, ... ]`       | Push [n, elements...] from advice_map[K].   |
| `adv.push_mtnode`       | `[d, i, R, ... ]` | `[d, i, R, ... ]` | Push Merkle tree node.                      |
| `adv.insert_mem`        | `[K, a, b, ... ]` | `[K, a, b, ... ]` | advice_map[K] = mem[a..b].                  |
| `adv.insert_hdword`     | `[B, A, ... ]`    | `[B, A, ... ]`    | K=hash(A||B), advice_map[K]=[A,B].          |

### Random Access Memory

Memory is 0-initialized. Addresses: [0, 2^32). Locals at offset 2^31.

#### Absolute Addressing

| Instruction        | Stack Input          | Stack Output     | Cycles | Notes                                        |
| ------------------ | -------------------- | ---------------- | ------ | -------------------------------------------- |
| `mem_load.a`       | `[a, ... ]`          | `[v, ... ]`      | 1-2    | v = mem[a]. Fails if a >= 2^32.              |
| `mem_loadw_be.a`   | `[a, 0,0,0,0,...]`   | `[A, ... ]`      | 1-2    | Load word big-endian (mem[a+3] on top).      |
| `mem_loadw_le.a`   | `[a, 0,0,0,0,...]`   | `[A, ... ]`      | 4-5    | Load word little-endian (mem[a] on top).     |
| `mem_store.a`      | `[a, v, ... ]`       | `[ ... ]`        | 2-4    | mem[a] = v.                                  |
| `mem_storew_be.a`  | `[a, A, ... ]`       | `[A, ... ]`      | 1-3    | Store word big-endian (top at mem[a+3]).     |
| `mem_storew_le.a`  | `[a, A, ... ]`       | `[A, ... ]`      | 8-9    | Store word little-endian (top at mem[a]).    |
| `mem_stream`       | `[C, B, A, a, ... ]` | `[E,D,A,a',...]` | 1      | Read 2 words from a, a' = a+8.               |

#### Procedure Locals

Not 0-initialized. Max 2^16 locals per procedure.

| Instruction       | Stack Input      | Stack Output | Cycles | Notes                                        |
| ----------------- | ---------------- | ------------ | ------ | -------------------------------------------- |
| `loc_load.i`      | `[ ... ]`        | `[v, ... ]`  | 3-4    | v = local[i].                                |
| `loc_loadw_be.i`  | `[0,0,0,0, ...]` | `[A, ... ]`  | 3-4    | Load word big-endian. i must be multiple of 4.|
| `loc_loadw_le.i`  | `[0,0,0,0, ...]` | `[A, ... ]`  | 7-8    | Load word little-endian.                     |
| `loc_store.i`     | `[v, ... ]`      | `[ ... ]`    | 4-5    | local[i] = v.                                |
| `loc_storew_be.i` | `[A, ... ]`      | `[A, ... ]`  | 3-4    | Store word big-endian.                       |
| `loc_storew_le.i` | `[A, ... ]`      | `[A, ... ]`  | 11-12  | Store word little-endian.                    |

---

## Cryptographic Operations

Uses Rescue Prime Optimized hash function.

| Instruction    | Stack Input          | Stack Output     | Cycles | Notes                                     |
| -------------- | -------------------- | ---------------- | ------ | ----------------------------------------- |
| `hash`         | `[A, ...]`           | `[B, ...]`       | 20     | B = hash(A). 1-to-1 hash.                 |
| `hperm`        | `[B, A, C, ...]`     | `[F, E, D, ...]` | 1      | RPO permutation. C=capacity, E=digest.    |
| `hmerge`       | `[B, A, ...]`        | `[C, ...]`       | 16     | C = hash(A,B). 2-to-1 hash.               |
| `mtree_get`    | `[d, i, R, ...]`     | `[V, R, ...]`    | 9      | Get Merkle node at depth d, index i.      |
| `mtree_set`    | `[d, i, R, V', ...]` | `[V, R', ...]`   | 29     | Update Merkle node, return old and new root.|
| `mtree_merge`  | `[R, L, ...]`        | `[M, ...]`       | 16     | Merge two Merkle trees.                   |
| `mtree_verify` | `[V, d, i, R, ...]`  | `[V,d,i,R,...]`  | 1      | Verify Merkle path. Can add .err=code.    |

---

## Flow Control Operations

### Conditional Execution

```masm
if.true
    # instructions for true branch
else
    # instructions for false branch (optional)
end
```

- Pops condition (must be 0 or 1, else fails)
- `if.true`: executes first block if cond=1
- `if.false`: executes first block if cond=0
- Empty branches become `nop`

### Counter-Controlled Loops

```masm
repeat.COUNT
    # instructions
end
```

- COUNT must be > 0 (integer or constant)
- Unrolled at compile time (no runtime counting cost)

### Condition-Controlled Loops

```masm
while.true
    # instructions
end
```

- Pops condition before each iteration
- Continues while condition = 1
- Fails if condition not boolean

### No-Operation

```masm
nop
```

- 1 cycle, no effects except advancing cycle counter

---

## Events and Debugging

### Events

| Instruction        | Stack Input       | Stack Output      | Cycles | Notes                                   |
| ------------------ | ----------------- | ----------------- | ------ | --------------------------------------- |
| `emit.<event_id>`  | `[...]`           | `[...]`           | 3      | Emit event to host. Use `emit.event("name")`.|
| `emit`             | `[event_id, ...]` | `[event_id, ...]` | 1      | Emit event from stack (IDs 0-255 reserved).|
| `log_precompile`   | `[COMM,TAG,PAD,...]`| `[R1,R0,CAP',...]`| 1      | Absorbs precompile commitment into transcript.|
| `trace.<trace_id>` | `[...]`           | `[...]`           | 0      | Emit trace (only with -t/--trace flag). |

### Debugging

Active only in debug mode (0 cycles):

| Syntax                | Notes                              |
| --------------------- | ---------------------------------- |
| `debug.stack`         | Print entire stack.                |
| `debug.stack.N`       | Print top N items (0 < N < 256).   |
| `debug.mem`           | Print entire RAM.                  |
| `debug.mem.A`         | Print memory at address A.         |
| `debug.mem.A.M`       | Print memory from A to M.          |
| `debug.local`         | Print all procedure locals.        |
| `debug.local.I`       | Print local at index I.            |
| `debug.local.I.M`     | Print locals from I to M.          |
