# K5 Technical Specification

**Document Version:** 0.2.0  
**Author:** Lois-Kleinner Alpasan  
**Status:** Production  

---

## 1. Field Arithmetic

The Goldilocks prime field 𝔽ₚ with **p = 2⁶⁴ − 2³² + 1** is used for all algebraic operations.

### Properties
- Enables efficient 64-bit arithmetic on modern CPUs
- Native field of Plonky2 and Polygon Zero ecosystem
- Multiplicative order 2⁶⁴ − 2³² divisible by 7 (enabling x⁷ S-box bijection)
- Provides 128 bits of security against discrete-log based attacks

### Field Element Representation
All field elements are unsigned 64-bit integers in [0, p−1].

---

## 2. Poseidon Permutation Parameters

| Parameter | Value | Justification |
|-----------|-------|---------------|
| State width T | 4 | 2-element rate (128b) + 2-element capacity (128b) |
| Full rounds RF | 8 | Standard for 128-bit security with x⁷ S-box |
| Partial rounds RP | 28 | Conservative for 64-bit field size |
| S-box α | 7 | Bijection in 𝔽ₚ (gcd(7, p−1) = 1) |
| MDS matrix | Cauchy | Provably MDS over 𝔽ₚ |

---

## 3. Round Structure

Each round (0 ≤ r < RF + RP):

1. **ARK (Add Round Key)**: `state[i] += round_constants[r][i]` for all i
2. **S-box**: 
   - Full rounds (r < RF/2 or r ≥ RF/2 + RP): Apply x⁷ to all elements
   - Partial rounds: Apply x⁷ only to state[0]
3. **MDS multiplication**: `state = M × state` where M is Cauchy MDS matrix

---

## 4. Cauchy MDS Matrix

```
M[i][j] = 1 / (xᵢ + yⱼ) mod p
```

With x = [1, 2, 3, 4] and y = [5, 6, 7, 8].

**Property:** All square submatrices are non-singular (verified by exhaustive determinant computation over 𝔽ₚ).

---

## 5. Round Constant Generation

Round constants generated deterministically from domain tag using SHAKE-256:

```
seed = domain_tag || "POSEIDON_RC"
for each round r and element i:
    round_constants[r][i] = int(SHAKE256(seed, (r·t + i + 1)·8))[-8:] mod p
```

Nothing-up-my-sleeve (NUMS) generation with full transparency.

---

## 6. K5 Sponge Construction

### State Representation
4 Goldilocks field elements (256 bits total):
```
State = [s₀, s₁ | s₂, s₃]
         rate (128b) | capacity (128b)
```

### Padding Scheme
10*1 padding standard:
```
padded = data || 0x80 || 0x00*N || bit_length_64
```
Where N ensures `(len(padded) + 8) % 16 == 0`. Eight-byte big-endian bit length appended for domain separation.

### Absorb Phase
```
for each 16-byte block of padded input:
    (e₀, e₁) = bytes_to_field_elements(block)
    state[0] += e₀
    state[1] += e₁
    state = Poseidon(state)
```

### Squeeze Phase
```
output = []
while len(output) < requested_bytes:
    output += bytes(state[0]) + bytes(state[1])
    state = Poseidon(state)
return output[:requested_bytes]
```

For outputs > 4096 bits:
```
seed = bytes(state[0]) + bytes(state[1])
output = cSHAKE256(seed, domain_tag, requested_bits / 8)
```

---

## 7. Balloon Hash (Memory-Hard Layer)

### Parameters
| Parameter | Default | Range |
|-----------|---------|-------|
| mem_kib | 1024 | 0–2²⁰ |
| time_cost | 3 | 1–255 |

### Algorithm
```
1. state = BLAKE3(data)                    // 32 bytes
2. buffer[i] = BLAKE3(state || i)          // for i = 0..N-1, N = mem_kib × 64
3. For t = 0..time_cost-1:
     For i = 0..N-1:
       prev = buffer[(i-1) mod N]
       dep_idx = int(buffer[i]) mod N
       buffer[i] = BLAKE3(buffer[i] || prev || buffer[dep_idx])
     For i = 0..N-1:
       buffer[i] = BLAKE3(buffer[i] || buffer[(i+t) mod N][:8])
4. result = BLAKE3(XOR(all buffer bytes) || state)
5. Feed result into K5 sponge
```

### Security Properties
- Forces ~N bytes of RAM per evaluation
- Time cost T multiplies computation cost
- No known shortcut computation exists
- Default (1 MiB, 3 passes) provides strong ASIC/GPU resistance

---

## 8. Test Vectors

Frozen known-answer test vectors in `src/k5/spec.py`:

| Mode | Input | Output Size |
|------|-------|-------------|
| K5-512 | "", "abc", "hello" | 512 bits |
| K5-1024 | "", "abc", "hello" | 1024 bits |
| K2048 | "", "abc", "hello" | 2048 bits |
| KOF-256 | "", "hello" | 256 bits |
| KOF-4096 | "", "hello" | 4096 bits |

---

## 9. Domain Separation

Domain tag `domain_tag` is bound into:
- Poseidon round constant generation
- cSHAKE customization string (for XOF extension)

Ensures different domains produce entirely unrelated outputs.

---

## References

1. Grassi et al., "Poseidon: A New Hash Function for Zero-Knowledge Proofs" (2021)
2. Hamburg, "Ed448-Goldilocks, a new elliptic curve" (2015)
3. Boneh et al., "Balloon Hashing: A Memory-Hard Function Providing Provable Protection Against Sequential Attacks" (2016)
4. NIST, "SHA-3 Derived Functions: cSHAKE, KMAC, TupleHash, and ParallelHash" (SP 800-185)
