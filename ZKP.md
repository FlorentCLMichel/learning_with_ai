# Basics of ZKPs

Here is a concise but technical summary of Zero-Knowledge Proofs (ZKP):

## Zero-Knowledge Proofs (ZKP)

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

### Knowledge-Soundness

Knowledge soundness (also called "proof of knowledge") in Zero-Knowledge Proofs refers to the property that if a prover can convince the
verifier to accept their proof, then they must actually know (not just prove the existence of) the witness for the statement being
proven. This is stronger than regular soundness which merely guarantees that false statements cannot be proven true.

Regular soundness only ensures: "No convincing proof exists for false statements"

Knowledge soundness additionally ensures: "A convincing proof implies the prover knows a witness/w secret"

This is formalized using an extractor—if the prover succeeds, there must exist an efficient algorithm (the knowledge extractor) that
can extract the witness from the prover.

### Zero-Knowledge

The Verifier learns nothing beyond the truth of the statement. Formally, there exists a Simulator $S$ that can produce transcripts indistinguishable from real interactions with any malicious Verifier ($V^*$):
$$ \forall V^* \exists S : \mathsf{View}_{V^*}(\langle P, V^* \rangle(x, w)) \approx S(x) $$
The indistinguishability can be:
- **Perfect**: Identical distributions
- **Statistical**: Negligible statistical distance
- **Computational**: Indistinguishable to polynomial-time algorithms

## Types of Proof Systems

### 1. SNARK (Succinct Non-interactive Argument of Knowledge)
- **Proof Size**: ~288 bytes (very small)
- **Verification Speed**: Milliseconds (fastest)
- **Setup**: Requires trusted setup ceremony
- **Security Assumptions**: Elliptic curves + pairings
- **Quantum Resistance**: No
- **Use Cases**: Blockchain rollups needing tiny proofs (e.g., Zcash)
- **Examples**: Groth16, PLONK

### 2. STARK (Scalable Transparent Argument of Knowledge)
- **Proof Size**: ~100KB (largest)
- **Verification Speed**: Slower than SNARKs but faster proving
- **Setup**: No trusted setup (transparent)
- **Security Assumptions**: Hash functions (post-quantum)
- **Quantum Resistance**: Yes
- **Use Cases**: High-throughput systems requiring quantum safety (e.g., StarkNet)
- **Examples**: StarkEx, StarkNet

### 3. SNARG (Succinct Non-interactive Argument)
- **Proof Size**: Similar to SNARKs (~288B) 
- **Verification Speed**: Fast
- **Setup**: May or may not need trusted setup
- **Security Assumptions**: Varies
- **Key Difference**: Weaker security than SNARKs (proof without knowledge)
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
| Assumptions       | EC Pairings  | Hashes      | Varies       | Discrete Log |
| Quantum Safe      | No           | Yes         | No           | No           |
| Scalability       | Medium       | High        | Medium       | Medium       |
| Best For          | Small proofs | High volume | Light proofs | Range proofs |

### Key Differences Summary

1. **Setup**: STARKs and Bulletproofs avoid trusted setup while SNARKs require it
2. **Security**: STARKs use hash-based security (post-quantum), others use number theory
3. **Performance**: SNARKs have smallest proofs, STARKs have fastest proving
4. **Specialization**: Bulletproofs excel at compact range proofs

## Key Techniques

- **Polynomial Commitments**: KZG (pairing-based), FRI (STARKs)
- **Arithmetization**: Convert computation to polynomial equations
  * R1CS (Rank-1 Constraint Systems) → QAP (Quadratic Arithmetic Program)
  * AIR (Algebraic Intermediate Representation) for STARKs
- **Cryptographic Primitives**:
  * Elliptic curve pairings (BLS12-381, BN254)
  * Fast Fourier Transforms (FFT) over finite fields
  * Merkle trees + Hash functions (Poseidon, Rescue)
- **Proof Composition**: 
  * Darlin (using Halo2 recursion)
  * Nova (folding schemes)

The mathematical foundations rely heavily on:
- **Hardness assumptions:** Discrete Log, Learning With Errors (LWE), Knowledge-of-Exponent
- **Algebraic techniques:** Multilinear polynomials, Reed-Solomon codes
- **Cryptographic accumulators:** Vector commitments, membership proofs
