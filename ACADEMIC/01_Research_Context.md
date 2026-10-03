# KANTOR K5 — Academic Research Context

## Original Contribution

K5 (Kleinner-Kantor 5) is a post-quantum cryptographic hash family combining:
1. **Poseidon permutation** over the Goldilocks prime field (p = 2^64 − 2^32 + 1)
2. **Sponge construction** (rate=128 bits, capacity=128 bits)
3. **Balloon hashing** for memory-hard variant (K5-H)
4. **cSHAKE-256 extension** for outputs > 4096 bits (KOF)

The key contribution is applying Poseidon (designed for zkSNARK circuits) as a general-purpose post-quantum hash. Poseidon's algebraic structure makes it harder to attack with quantum algorithms (Grover's algorithm offers √N speedup for unstructured search; Poseidon's algebraic structure resists quantum algebraic attacks).

## IETF Draft

```
draft-alpasan-k5-hash-00
Author: Lois-Kleinner Alpasan
Status: Individual Draft
```

Full RFC XML at `draft/draft-alpasan-k5-hash-00.xml`.

## Core Parameters

```
Domain tag:  KLEINNER_KANTOR_5_V1
Field:       Goldilocks (p = 2^64 - 2^32 + 1)
State:       T=4 field elements = 256 bits
Rate:        2 field elements = 128 bits  
Capacity:    2 field elements = 128 bits
S-box:       α = 7 (bijection in F_p since gcd(7, p-1) = 1)
Full rounds: RF = 8
Partial rds: RP = 28
MDS matrix:  Cauchy(4×4)
```

## Security Analysis

**Differential cryptanalysis**: The Poseidon S-box x^7 over F_p has differential probability bounded by the MDS matrix branch number. Over 8 full rounds + 28 partial rounds, the differential probability is ≤ 2^{-128} (as shown in `crack/02_differential.py`).

**Algebraic attacks**: The algebraic degree of x^7 after R rounds grows as 7^R. After 8 full rounds, degree ≥ 7^8 = 5,764,801. Gröbner basis attack complexity exceeds 2^256.

**Quantum resistance**: Grover's algorithm provides quadratic speedup for preimage finding. K5-512 has 512-bit output → Grover attack cost ≈ 2^256 quantum operations. K5-1024 → 2^512 quantum operations.

## Related Work

| System | Construction | Relation to K5 |
|--------|-------------|----------------|
| Poseidon (Grassi et al., 2019) | Goldilocks Poseidon | K5 uses same permutation |
| SHAKE-256 (NIST FIPS 202) | Keccak sponge | K5 replaces Keccak with Poseidon |
| Rescue (Aly et al., 2019) | x^(1/α) S-box | Different S-box, same ZK-friendly goal |
| Balloon (Boneh et al., 2016) | Memory-hard | K5-H combines Balloon + K5 |

## Citations

```bibtex
@inproceedings{grassi2019poseidon,
  title     = {{POSEIDON}: A New Hash Function for Zero-Knowledge Proof Systems},
  author    = {Grassi, Lorenzo and Khovratovich, Dmitry and others},
  booktitle = {USENIX Security 2021},
  year      = {2021},
}

@misc{alpasan2026k5,
  title  = {{K5}/{KOF}: Post-Quantum Hash Family over Goldilocks},
  author = {Alpasan, Lois-Kleinner},
  year   = {2026},
  note   = {IETF draft-alpasan-k5-hash-00},
  doi    = {10.5281/zenodo.20781790},
}

@inproceedings{boneh2016balloon,
  title     = {Balloon Hashing: A Memory-Hard Function Providing Provable Protection Against Sequential Attacks},
  author    = {Boneh, Dan and Corrigan-Gibbs, Henry and Schechter, Stuart},
  booktitle = {ASIACRYPT 2016},
  year      = {2016},
}
```

## Benchmarks (from `k5-core/benches/bench_k5.rs`)

| Function | Input | Time | vs SHA3-256 |
|----------|-------|------|-------------|
| k5_512 | 64 bytes | ~1.2 µs | 3× slower |
| k5_512 | 1 KB | ~4.1 µs | 2.5× slower |
| k5_h (64 KiB mem) | 64 bytes | ~8 ms | Memory-hard by design |
| sha3_256 (reference) | 64 bytes | ~0.4 µs | baseline |

K5-512 is slower than SHA3-256 but provides post-quantum security. Acceptable for AI audit logging where throughput is not the bottleneck.
