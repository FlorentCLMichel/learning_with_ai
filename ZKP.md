# Basics of ZKPs

Here is a concise but technical summary of Zero-Knowledge Proofs (ZKP):

## Zero-Knowledge Proofs (ZKP)

In this note, unless stated otherwise, 

* $P$ denotes a prover
* $V$ denotes a verifier
* $\lambda$ is a ‘small’ parameters
* $x$ is a statement
* $L$ is a set of ‘correct’ statements
* If $x \in L$, $w$ denotes a witness for $x$

### What is a ZKP?

- Method where a *prover* convinces a *verifier* they know a secret $w$ (witness) for some statement $x$ without revealing $w$
- Satisfies: there exists a protocol where the verifier learns only that $\exists w$ s.t. $R(x,w) = 1$ (for some relation $R$)
- Used for: 
  * Privacy-preserving authentication (*e.g.* Zcash)
  * Scalable computation verification (zk-Rollups)
  * Private smart contracts (*e.g.* Tornado Cash)
  * MPC-in-the-head protocols

### Key concepts

- **Prover**: The party that possesses secret information (witness) and generates cryptographic proofs to demonstrate knowledge of it without revealing the secret itself.

- **Verifier**: The party that examines proofs from the Prover, checking their validity according to the protocol's rules, without learning the secret information.

- **Statement (x)**: A public claim about some secret information, typically asserting membership in a language or satisfaction of a relation (e.g., "I know a preimage for this hash").

- **Witness (w)**: The private information that satisfies the statement, known only to the Prover (e.g., the preimage of a hash, a valid signature key).

## Core Properties

### Completeness

For any true statement with valid witness, an honest Prover will convince an honest Verifier with overwhelming probability:
$$ \Pr[\langle P, V \rangle(x, w) = 1] = 1 - \mathit{negl}(\lambda) $$
This ensures that valid proofs are almost always accepted when both parties follow the protocol. The probability approaches 1 as the security parameter $\lambda$ increases.

### Soundness

No malicious Prover ($P^*$) can convince the Verifier of a false statement, except with negligible probability:
$$ \Pr[\langle P^*, V \rangle(x) = 1] \leq \mathit{negl}(\lambda) \quad \text{for } x \notin L $$
Two main variants exist:
- **Statistical soundness**: Security holds against unbounded adversaries
- **Computational soundness**: Security relies on computational hardness assumptions (e.g., discrete logarithm, factoring)

**Verification Modes**:
- **Honest-Verifier ZK (HVZK)** - Only secure against verifiers following protocol
- **Malicious-Verifier ZK** - Security against arbitrary verification strategies

### Knowledge-Soundness

Knowledge soundness (also called "proof of knowledge") in Zero-Knowledge Proofs refers to the property that if a prover can convince the
verifier to accept their proof, then they must actually know (not just prove the existence of) the witness for the statement being
proven. This is stronger than regular soundness which merely guarantees that false statements cannot be proven true.

Regular soundness only ensures: "No convincing proof exists for false statements"

Knowledge soundness additionally ensures: "A convincing proof implies the prover knows a witness/w secret"

This is formalized using an extractor: For any prover $P^*$ that convinces $V$ with probability $\epsilon$, there exists an extractor $\mathcal{E}$ that runs in time $\mathit{poly}(1/\epsilon)$ and outputs $w$ such that $R(x,w) = 1$ with probability approaching 1.

We write this as:
$$ \forall P^* \exists \mathcal{E} : (w \leftarrow \mathcal{E}^{P^*}(x)) \land (R(x,w)=1) $$
with probability $1 - \mathit{negl}(\lambda)$

### Zero-Knowledge

The Verifier learns nothing beyond the truth of the statement. Formally, there exists a Simulator $S$ (that does not know $w$) that can produce transcripts indistinguishable from real interactions with any malicious Verifier ($V^*$):
$$ \forall V^* \exists S : \mathsf{View}_{V^*}(\langle P, V^* \rangle(x, w)) \approx S(x) $$
The indistinguishability can be:
- **Perfect**: Identical distributions
- **Statistical**: Negligible statistical distance
- **Computational**: Indistinguishable to polynomial-time algorithms

## Types of Proof Systems

### Interactive vs Non-Interactive
A key distinction in proof systems:

- **Interactive**: Multiple rounds of communication between $P$ and $V$ (e.g., classical ZK protocols like Schnorr's protocol)
- **Non-Interactive (NIZK)**: Single message proof (e.g., SNARKs/STARKs) enabled by:
  * Random oracle model (Fiat-Shamir heuristic)
  * Trusted setup parameters

### 1. SNARK (Succinct Non-interactive Argument of Knowledge)
- **Proof Size**: ~288 bytes (very small)
- **Verification Speed**: Milliseconds (fastest)
- **Proving Speed**: Moderate (depends on circuit size)
- **Setup**: Required
  * *CRS (Common Reference String)*: Public parameters generated in a setup phase that both prover/verifier use
  * *Groth16:* Per-circuit trusted setup
  * *PLONK:* Universal trusted setup (reusable)
  * *Trusted setup risks*: Requires secure parameter generation ceremony ("toxic waste" disposal)
- **Security Assumptions**: 
  * Elliptic curve pairings (bilinear groups)
  * Knowledge-of-Exponent Assumption (KEA)
  * Some variants require q-PKE or q-PDH
- **Quantum Resistance**: No
- **Use Cases**: Blockchain rollups needing tiny proofs (e.g., Zcash, Filecoin)
- **Examples**: 
  * Groth16: Most efficient proofs
  * PLONK: Flexible circuits with universal setup
  * Halo2: Recursive proofs without trusted setup

### 2. STARK (Scalable Transparent Argument of Knowledge)
- **Proof Size**: ~100KB (largest)
- **Verification Speed**: Slower than SNARKs
- **Proving Speed**: Faster than SNARKs for large computations due to parallelization
- **Setup**: No trusted setup (transparent)
- **Security Assumptions**: Hash functions (post-quantum)
- **Quantum Resistance**: 
  * Based on hash functions (e.g., SHA-3, Rescue)
  * Security reduces to finding collisions/preimages → quantum security requires 256+ bit security level
  * Grover's quantum algorithm provides square-root speedup → need larger security parameters
- **Use Cases**: High-throughput systems requiring quantum safety (e.g., StarkNet)
- **Examples**: StarkEx, StarkNet

### 3. SNARG (Succinct Non-interactive Argument)
- **Proof Size**: Similar to SNARKs (~288B) 
- **Verification Speed**: Fast
- **Setup**: May or may not need trusted setup
- **Security Assumptions**: Varies
- **Key Difference**: Weaker security guarantee than SNARKs since they only prove statement validity (not knowledge possession)
- **Security Implications**: 
  * Vulnerable to "proof of pre-processing" attacks (prover computes proof from leaked witness material) 
  * **SNARK** = **SNARG** + Knowledge soundness (proves actual witness possession)
- **Example Variants**: SNARK = SNARG + Knowledge soundness

### 4. Bulletproofs
- **Proof Size**: Logarithmic (medium)
- **Verification Speed**: Slowest
- **Setup**: No trusted setup
- **Security Assumptions**: Discrete logarithm problem
- **Quantum Resistance**: No
- **Specialty**: Efficient range proofs
- **Use Cases**: Confidential transactions (e.g., Monero)

### Comparison Table

| Property          | SNARK        | STARK       | SNARG        | Bulletproofs |
| ----------------- | ------------ | ----------- | ------------ | ------------ |
| Proof Size        | ~288B        | ~100KB      | ~288B        | Logarithmic  |
| Verification Time | Fastest      | Medium      | Fast         | Slowest      |
| Trusted Setup     | Required     | No          | Possibly     | No           |
| Assumptions       | EC Pairings + KEA | Hashes      | Varies       | Discrete Log |
| Quantum Safe      | No           | Yes         | No           | No           |
| Scalability       | Medium       | High        | Medium       | Medium       |
| Best For          | Small proofs | High volume | Light proofs | Range proofs |

### Key Differences Summary

1. **Setup**: STARKs and Bulletproofs avoid trusted setup while SNARKs require it
2. **Security**: STARKs use hash-based security (post-quantum), others use number theory
3. **Performance**: SNARKs have smallest proofs, STARKs have fastest proving
4. **Specialization**: Bulletproofs excel at compact range proofs

## Key Techniques

### Polynomial IOPs (Interactive Oracle Proofs)

Framework that separates proof construction into:
- **Oracles**: Prover provides oracle access to polynomials
- **Queries**: Verifier makes oracle queries during interaction
- **Consistency Checks**: Constraints between polynomials

Key advantages:
- Modularizes proof components
- Enables efficient composition via recursive usage
- Examples: Plonk, STARK, Sonic employ this framework

### Polynomial Commitments
Core primitive allowing a prover to commit to a polynomial and later prove evaluations of it. Several schemes exist:
- **KZG** (Kate-Zaverucha-Goldberg): Pairing-based with constant-sized proofs but requires trusted setup
- **FRI** (Fast Reed-Solomon IOPP): Used in STARKs, hash-based with transparent setup but larger proofs
- **IPA** (Inner Product Arguments): Used in Bulletproofs, based on discrete logarithm
- **DARK** (Diophantine Arguments of Knowledge): Uses groups of unknown order for post-quantum security
- **Hyrax**: Uses multilinear polynomials for commitments with logarithmic verifier time

### Arithmetization
Process of converting computational statements into systems of polynomial equations:
- **R1CS → QAP** (SNARK approach):
  * **R1CS**: Compile computation into linear algebra system: $A·z \circ B·z = C·z$
  * **QAP**: Encode constraints as quotient space membership using Lagrange interpolation
  * **Low-Degree Check**: Ensure compatibility between polynomials
- **AIR** (Algebraic Intermediate Representation - STARK approach):
  * Define execution trace as 2D table of register states
  * Express constraints as transition functions between rows: $\forall i,\ F(\text{trace}[i], \text{trace}[i+1]) = 0$
  * Apply boundary constraints on initial/final states
- **Plonkish Arithmetization**:
  * General-purpose constraints ($q_L·a + q_R·b + q_O·c + q_M·a·b + q_C = 0$)
  * Custom gates for optimized circuit implementations

### Cryptographic Primitives
Critical building blocks used across protocols:
- **Elliptic Curves**:
  * Pairing-friendly curves (BLS12-381, BN254) for efficient pairings in SNARKs
  * Curve25519/Ed25519 for faster operations in transparent schemes
- **Polynomial Transforms**:
  * FFT over finite fields for efficient polynomial multiplication/interpolation
  * Multiexponentiations (Pippenger) for fast MSM computations
- **Hash Functions**:
  * ZK-friendly designs (Poseidon, Rescue, Pedersen) minimize circuit complexity
  * Merkle trees (binary/poseidon) for commitment to large data structures
- **Zero-Knowledge Oracles**:
  * Random linear combination checks (PLONK)
  * Lookup arguments (Plookup, Halo2) for efficient table operations

### Proof Composition
Techniques to combine proofs for scalability and flexibility:
- **Recursion**:
  * Verify proofs inside bigger proofs (e.g., Halo2 recursion)
  * Requires special cycles of elliptic curves (for pairing-friendly curves)
- **Folding Schemes** (Nova):
  * Accumulate multiple instances into one
  * IVC (Incrementally Verifiable Computation) for long-running processes
- **Aggregation**:
  * Combine multiple proofs into single proof (SnarkPack, Bulletproofs)
  * Requires different techniques for different proof systems
- **Proof Carrying Data**:
  * Maintain integrity chain through distributed computations
  * Basis for zkBridge designs

## Mathematical Foundations

Zero-Knowledge Proof systems build upon several core mathematical concepts:

### Hardness Assumptions
Cryptographic security relies on computational problems believed to be hard:
- **Discrete Logarithm (DL)**: Given $g$ and $g^a$ in cyclic group $G$, find $a$
  * Basis for: Schnorr protocol, Bulletproofs
- **Learning With Errors (LWE)**: Solve noisy linear equations; basis for post-quantum schemes
  * Used in: lattice-based ZKPs
- **Knowledge-of-Exponent (KEA)**: If adversary computes $g^a$, they know $a$
  * Critical for: extractability in SNARKs
- **Elliptic Curve Pairings**: Bilinear maps $e: G_1 × G_2 → G_T$ enabling compact proofs
- **RSA Assumption**: Hardness of factoring large composites
- **Strong RSA Assumption**: Given $N$ and $y$, hard to find $x$ and $e>1$ s.t. $x^e ≡ y \mod N$ - used in accumulator constructions

### Algebraic Techniques
Efficient polynomial manipulation for arithmetization:
- **Multilinear Polynomials**: $p(x_1,...,x_k)$ linear in each variable
  * Enable: sum-check protocol, fast interactive proofs
- **Reed-Solomon Codes**: Error-correcting codes with polynomial evaluations
  * Core of: FRI protocol (STARKs), low-degree testing
- **Fast Fourier Transform (FFT)**: used for $\mathcal{O}(n \log n)$ polynomial multiplication
  * Optimizes: polynomial commitments, constraint systems
- **Elliptic Curve Cryptography**: 
  * Groups with efficient arithmetic & pairings (BLS12-381, BN254)

### Cryptographic Accumulators
Compact representations of sets with membership proofs:
- **Vector Commitments**: Binding to a vector $v_1,...,v_n$ with $\mathcal{O}(1)$ proofs
  * Example: polynomial commitments (KZG)
- **Membership Proofs**: 
  - **Merkle Trees**: Binary trees with hash nodes (logarithmic proof size)
  - **RSA Accumulators**: $acc = g^{\prod (e_i)} \mod N$ for set $\{e_i\}$
- **Universal Accumulators**: Support non-membership proofs
- **Zero-Knowledge Accumulators**: Hide set elements (e.g., using Pedersen commitments)
