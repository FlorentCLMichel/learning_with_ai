# Basics

Here's a concise technical summary of Multi-Party Computation (MPC):

1. **Multi-Party Computation (MPC)**
   - Method where n parties compute a function f(x₁,...,xₙ) while keeping their inputs xᵢ private
   - Satisfies: ∀i, party Pᵢ learns only f(x₁,...,xₙ) and nothing else about xⱼ (j≠i)
   - Used for:
     * Private auctions/contracts
     * Privacy-preserving data analysis
     * Federated learning
     * Threshold cryptography

2. **Core Properties**
   - **Privacy**: No party learns more than specified by protocol
     ```∀i, Viewᵢ ≈ Viewᵢ' when xⱼ = xⱼ' ∀j≠i```
   - **Correctness**: Output matches f(x₁,...,xₙ) when parties honest
   - **Input Independence**: Adversary can't choose inputs after seeing others'
   - **Fairness/Guaranteed Output Delivery**: All get output or none do

3. **MPC Protocol Types**
   - **Honest Majority** (Adversaries < n/2): 
     * BGW (Ben-Or, Goldwasser, Wigderson) 
     * SPDZ (preprocessing model)
   - **Dishonest Majority** (Security Against Any n-1 Corruptions):
     * GMW (Goldreich-Micali-Wigderson)
     * Yao's Garbled Circuits (2-party)
   - **Threshold Adversary Structure**: (t,n)-threshold where t corruptions tolerated
   - **Synchronous vs Asynchronous**: Message delivery guarantees

4. **Key Techniques**
   - **Garbled Circuits** (2PC): 
     * AND-XOR gates encrypted with cryptographic labels
     * Point-and-permute optimizations
   - **Secret Sharing** (n-party): 
     * Shamir's SSS (threshold schemes)
     * Additive/Multiplicative sharing
   - **Oblivious Transfer (OT)**:
     * Correlated OT (cOT) for efficiency
   - **Preprocessing Models**: 
     * Beaver triples for multiplication
     * Function-dependent/independent setup
   - **MPC-in-the-Head**: Connection with ZKPs

Mathematical foundations rely on:
- Information-theoretic security (for honest majority)
- Computational hardness (LWE, DDH) for dishonest majority
Bundle: Number-Theoretic Transforms used in lattice-based MPC
Current frontiers include:
- Malicious security with abort vs fairness
- Protocol composition via UC security
- Integration with FHE (Fully Homomorphic Encryption)
