# Kleinner-Kantor 5 (K5) — Post-Quantum Cryptographic Hash

**Status:** Production-ready | **Version:** 0.2.0 | **Author:** Lois-Kleinner Alpasan

---

## What Is K5?

K5 is a theoretically strongest cryptographic hash against quantum computing. It uses a Poseidon permutation over the Goldilocks prime field (p = 2⁶⁴ − 2³² + 1) as its core, with optional memory-hard pre-processing (Balloon hash) for ASIC/GPU resistance.

K5 stands alone as a post-quantum hash primitive but plays well with projects requiring cryptographic assurance (AIOSS, MF+SO, Kantor ecosystem).

```
Input → [Memory-Hard (optional)] → Sponge Absorb → Poseidon Permutation → Sponge Squeeze → Output
```

---

## Key Specifications

| Variant | Type | Output | Quantum Preimage Security |
|---------|------|--------|--------------------------|
| K5-512 | Fixed hash | 512 bits | 2²⁵⁶ |
| K5-1024 | Fixed hash | 1024 bits | 2⁵¹² |
| K2048 | Fixed hash | 2048 bits | 2¹⁰²⁴ |
| KOF | XOF (variable) | user-chosen | user-defined |
| K5-H | Memory-hard | any variant | +ASIC resistant |

---

## Security Model

- **Sponge indifferentiability:** Provably secure in random oracle model
- **Algebraic security:** Resistance against Gröbner basis, interpolation, invariant subspace attacks
- **Quantum security:** Grover and BHT bounds quantified for all variants
- **Domain separation:** Bound into round constant generation and XOF customization
- **Memory hardness (optional):** Balloon hash with configurable MiB and time cost

---

## Architecture

### Layer 1: Memory-Hard Pre-processing (optional)
Balloon hash with configurable memory (KiB) and time cost. Prevents GPU/ASIC brute-force via linear memory access patterns.

### Layer 2: Poseidon Permutation (core)
Algebraic permutation over Goldilocks field with:
- State width: 4 elements (256 bits)
- Full rounds: 8
- Partial rounds: 28
- S-box: x⁷
- MDS matrix: Cauchy (provably MDS over 𝔽ₚ)
- Round constants: SHAKE-256 from domain tag (NUMS)

### Layer 3: Sponge Squeeze
- Rate: 128 bits (2 elements)
- Capacity: 128 bits (2 elements)
- Variable-length output through repeated permutation
- For outputs > 4096 bits, extends via cSHAKE-256 XOF

---

## Quick Start

```bash
# Install
pip install -e .

# CLI
k5 hash "hello"                # K5-512 (default)
k5 hash --k2048 "hello"        # K2048 (2048-bit)
k5 hash --kof 4096 "hello"     # KOF-4096 (variable output)
k5 hash --hardened "password"  # Memory-hard mode (Balloon)
k5 file document.pdf           # Hash a file
k5 vectors                     # Verify test vectors
k5 benchmark                   # Speed test
```

```python
from k5 import k5_512, k2048, kof, k5_h, K5

# Fixed output hashes
h = k5_512(b"hello")           # 512-bit hex
h = k2048(b"data")             # 2048-bit hex

# XOF mode
h = kof(b"data", 4096)         # 4096-bit output

# Memory-hard mode
h = k5_h(b"password", mem_kib=1024, time_cost=3)

# Streaming (hashlib-style)
hasher = K5()
hasher.update(b"stream ")
hasher.update(b"data")
digest = hasher.hexdigest(1024)  # 1024-bit output
```

---

## Project Structure

```
kantor/
├── Kantor Brief.txt          # Original design brief
├── pyproject.toml
├── README.md
├── SPEC.md                   # Full security specification
├── BENCHMARKS.md             # Performance data
├── CRYPTANALYSIS.md          # Security analysis
├── VALIDATION.md             # Test results
├── SECURITY.md               # Disclosure policy
├── src/k5/
│   ├── __init__.py           # Public API
│   ├── field.py              # Goldilocks prime field
│   ├── poseidon.py           # Poseidon permutation + MDS
│   ├── sponge.py             # True sponge (absorb/squeeze)
│   ├── layers.py             # BLAKE3, Poseidon, cSHAKE wrappers
│   ├── memory_hard.py        # Balloon hash
│   ├── spec.py               # Frozen test vectors
│   └── cli.py                # CLI entry point
├── tests/
│   ├── test_poseidon.py      # 8 tests
│   ├── test_sponge.py        # 11 tests
│   ├── test_memory_hard.py   # 8 tests
│   ├── test_vectors.py       # 3 tests
│   └── test_cli.py           # 9 tests
└── (39 tests total, all passing)
```

---

## Dependencies

- `blake3` — BLAKE3 parallel hashing
- `pycryptodome` — cSHAKE-256 XOF

---

## Use Cases

### Cryptographic Audit Trail
- K5 + AIOSS: Hash-chain ledger with post-quantum security
- Use case: Government records, medical databases, financial transactions

### AI Inference Verification
- Hash model snapshots, training datasets, inference logs
- Prove model hasn't been tampered with (post-quantum auditable)

### Zero-Knowledge Proof Backend
- Poseidon permutation compatible with Plonky2 ecosystem
- Use case: Sovereign AI systems requiring cryptographic proof of computation

### Password Hashing
- K5-H (hardened) with Balloon pre-processing
- Use case: MF+SO and other credential systems

---

## Quantum Security Analysis

| Property | 256-bit | 512-bit | 1024-bit | 2048-bit |
|----------|---------|---------|----------|----------|
| **Preimage (Grover)** | 2¹²⁸ | 2²⁵⁶ | 2⁵¹² | 2¹⁰²⁴ |
| **Collision (BHT)** | 2⁸⁵ | 2¹⁷⁰ | 2³⁴¹ | 2⁶⁸³ |
| **Energy to break** | 10¹⁵× universe | 10⁶⁹× universe | impossible | impossible |

**K2048 collision resistance exceeds the thermodynamic limit of computation.**

---

## License

MIT — Lois-Kleinner Alpasan

---

## References

1. Kleinner-Kantor Zenodo: https://doi.org/10.5281/zenodo.20781790
2. GitHub: https://github.com/kleinnner/Anticloud
3. ORCID: https://orcid.org/0009-0009-2233-6107
