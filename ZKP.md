# Zero-Knowledge Proofs: A Technical Primer

## Introduction

### What is a ZKP?
A **Zero-Knowledge Proof (ZKP)** is a cryptographic method where a **Prover** convinces a **Verifier** that a statement is true without revealing the underlying secret information (the **witness**).

* **Key elements:**
    * A public *statement* $x$, the claim being proved (*e.g.*, “I know a preimage of this hash value”).
    * A private *witness* $w$ known to the prover (*e.g.* the preimage itself).
    * A relation $R$ that takes the public statement $x$ and the private witness $w$ as inputs, and outputs a binary result (1 or 0) such that $R(x,w) = 1$ if and only if $w$ is a valid witness for $x$ (*e.g.* if applying the hash function to $w$ gives $x$).
    * The *language* $L$ for the relation $R$ is the set of all statements for which a valid witness exists (*e.g.* the set of all possible hashes).
* **Core Promise:** The verifier learns only that there exists a witness $w$ such that $R(x,w) = 1$, and nothing else. In particular, they so not learn $w$.
* **Key Use Cases:**
    * **Privacy:** Authenticating without revealing passwords, or private transactions on blockchains (*e.g.*, Zcash, Tornado Cash).
    * **Scalability:** In a blockchain context, verifying complex computations off-chain to save on-chain gas (*e.g.*, zk-Rollups).

### Fundamental Actors

* **Prover ($P$):** The party holding the secret witness $w$. They generate the proof.
* **Verifier ($V$):** The party checking the proof validity without access to $w$.
* **Simulator ($S$):** A theoretical construct used to prove the “Zero-Knowledge” property; if a simulator can generate a transcript that is indistinguishable from a real execution from the point of view of $V$ without knowing $w$, the protocol leaks no knowledge.

The ASCII diagram below shows, schematically and in a simplified way, a protocol for a Non-Interactive Zero-Knowledge (NIZK) proof system, more specifically a [SNARK](#System Comparison) A ZK proof is said *Non-Interactive* if it requires only a single message from the prover to the verifier, rather than multiple rounds of back-and-forth communication. These systems are typically enabled by either a ["Trusted Setup"](#The Zero-Knowledge Trusted Setup (CRS Ceremony)) or the ["Fiat-Shamir heuristic"](#The Fiat-Shamir Heuristic) (which turns an interactive protocol into a non-interactive one). 

```
                                +-------------------+
                                |     ZKP System    |
                                |       (NIZK)      |
                                +-------------------+
                                          |
        +---------------------------------+---------------------------------+
        |                                 |                                 |
        v                                 v                                 v
   +----------+                  +-----------------+                  +----------+
   |  Prover  |                  |  Trusted Setup  |                  | Verifier |
   |   (P)    |                  |     (SNARK)     |                  |   (V)    |
   +----------+                  +-----------------+                  +----------+
        |                                 |                                 |
        |                                 |   Common Reference String (CRS) |
        | <-----------------------------------------------------------------|
        |                                 |                                 |
        | Witness (w) [Secret]            |                                 |
        | Statement (x) [Public]          |                                 |
        |                                 |                                 |
        | [Computes Proof]                |                                 |
        |                                 |                                 |
        | Proof (π)                       |                                 |
        |-----------------------------------------------------------------> |
        |                                 |                [Verifies Proof] |
        |                                 |                                 |
        |                                 |          Result (Accept/Reject) |
        |<------------------------------------------------------------------| 
```
### Core Properties
In the following, we will informally denote by $\langle P, V \rangle$ the interaction between the prover and verifier.

For a ZKP system to be valid, it must satisfy three properties:

#### 1. Completeness (Perfect or Statistical)
If the statement is true and the Prover is honest, the Verifier will be convinced with overwhelming probability (*statistical completeness*) or with certainty (*perfect completeness*). Completeness can be formalised as follows:

* **Statistical Completeness:** $R(x, w) = 1 \Rightarrow \Pr[\langle P, V \rangle(x, w) = 1] = 1 - \mathit{negl}(\lambda)$.
* **Perfect Completeness:** $R(x, w) = 1 \Rightarrow \Pr[\langle P, V \rangle(x, w) = 1] = 1$.

#### 2. Soundness (or Knowledge Soundness)

* **Soundness:** A malicious Prover cannot convince a Verifier of a false statement: $\forall P^*, x \notin L \Rightarrow \Pr[\langle P^*, V \rangle(x) = 1] \leq \mathit{negl}(\lambda)$.
  
* **Knowledge Soundness:** A stronger property. If a Prover convinces the Verifier, they must *know* the witness, not just that a witness exists. This is defined by the existence of an **Extractor ($\mathcal{E}$)** that can recover $w$ from a successful adversary: $\exists \mathcal{E} \; \forall P^* \; \Pr[w \leftarrow \mathcal{E}^{P^*}(x) : R(x,w)=1] \geq \Pr[\langle P^*, V \rangle(x) = 1] - \mathit{negl}(\lambda)$.

#### 3. Zero-Knowledge
The interaction reveals nothing but the validity of the statement. The verifier's view of the interaction can be simulated by a Simulator $S$ that does not know $w$: $\mathsf{View}_V(\langle P, V \rangle(x, w)) \approx S(x)$.
### Simple examples

#### Proving the color of the card drawn from a standard deck

**Protocol Decription:**

1. **Setup:** You have a standard deck of 52 cards (26 red, 26 black).
2. **The Claim:** The Prover ($P$) draws one card at random, looks at it, and places it face-down on the table. $P$ claims: "The card I have drawn is red."
3. **The Proof:** To prove the card is red without revealing its suit (Hearts/Diamonds) or its value (Ace, King, etc.), the Prover takes the remaining 51 cards in the deck.
4. **Verification:** The Prover systematically flips over and shows the Verifier ($V$) every single black card (all 26 of them: 13 Spades and 13 Clubs) from the remaining pile.

**Analyzing the properties:**

1. **Completeness:**
    * **Definition:** If the statement is true and both parties follow the rules, the Verifier will be convinced.
    * **In this example:** If the Prover actually drew a red card, then all 26 black cards must still be in the remaining pile of 51, the Prover successfully completes the protocol, and the Verifier is logically forced to accept that the hidden card is red.

2. **Soundness:**
    * **Definition:** If the statement is false, a cheating Prover cannot convince the Verifier.
    * **In this example:** Suppose the Prover lied and actually drew a black card (e.g., the Ace of Spades). In this case, there are only 25 black cards left in the remaining pile. No matter what the Prover does, she cannot produce 26 distinct black cards to show the Verifier. Since she cannot satisfy the requirement of the proof, the Verifier will not be convinced. This protects the Verifier from being "tricked" by a false claim.

3. **Zero-knowledge**
    * **Definition:** The Verifier learns nothing other than the fact that the statement is true.
    * **In this example:** After the proof, the Verifier knows with 100% certainty that the hidden card is red. However, the Verifier has no idea:
        * If it is a Heart or a Diamond.
        * What the Rank of the card is (e.g., is it a 5 or a Queen?).
        * The Verifier's "view" consists only of seeing the 26 black cards he already knew existed in a standard deck. He gained no "knowledge" about the secret witness ($w$, the specific card) itself.


**Comparison to Digital ZKPs:**

While this physical example uses a “process of elimination”, digital ZKPs like SNARKs use *Polynomial Commitments* to achieve the same effect.

* In the card game, “revealing the black cards” is like the *Proof* ($\pi$).
* The “hidden red card” is the *Witness* ($w$).
* The “standard 52-card deck” rules act as the *Relation* ($R$).


#### Sudoku board ([GNPR protocol](https://link.springer.com/article/10.1007/s00224-008-9119-9))

Imagine you (the Prover) claim to have solved a difficult Sudoku, and your friend (the Verifier) wants to be sure you aren't lying but doesn't want to see the solution yet. We shall use a physical card protocol.

1. **Setup (The Commitment):**
    * You take three identical decks of cards.
    * For every cell on the Sudoku board, you place three cards face-down. These cards represent the number in your solution (e.g., if a cell is a "5", you place three 5s there).
    * For the pre-filled "hint" numbers already on the board, you place those three cards face-up so the Verifier can see they match the puzzle. Once they are satisfied, you flip them face-down. 

2. **The Challenge:** The Verifier now chooses one of three "challenges" at random:
    1. **Rows:** Check if every row contains 1–9.
    2. **Columns:** Check if every column contains 1–9.
    3. **Subgrids:** Check if every 3x3 box contains 1–9.

3. **The Proof (The Shuffle):** If the Verifier chooses Rows (respectively Columns or Subgrids):
    * You take one of the three cards from every cell in the first row (resp. column or subgrid) and put them into a packet. You do this for all 9 rows (resp. columns or subgrids).
    * Crucially, you shuffle each packet thoroughly before handing it to the Verifier.
    * The Verifier opens the packets. They see nine cards numbered 1 through 9 in each packet

Steps 2 and 3 are repeated for the other two challenges, in an order chosen randomly. 

**Why this works:**

* **Completeness:** If you actually have the correct solution, you will always be able to produce packets containing 1–9.

* **Zero-Knowledge:** Because the packets were shuffled, the Verifier sees that the numbers 1–9 are present, but they have no idea which number came from which cell. The “where” information is destroyed by the shuffle.

* **Soundness:** If you cheated and have two "5s" in a row, one of your packets will eventually be missing a number. Since the Verifier can pick rows, columns, or subgrids, a cheater has a high chance of being caught. To make the proof “mathematically certain,” you simply repeat this process multiple times with fresh cards.

**Note:** It is critical that the challenges are chosen randomly and not communicated to the Prover before Setup; otherwise, the Prover could generate different solutions for the three types of challenges without being caught.

**Variation:** Alternatively, during the setup phase the Prover could put $n$ cards in each cell (where $n$ is a pre-determined number of iterations) and steps 2. and 3. could be repeated as is, with the challenge being chosen at random between the three possible ones at each iteration.

---

## The ZKP Pipeline: How it Works
ZKP systems generally follow a transformation pipeline to turn code into a proof.

### Step 1: [Arithmetization](#The Arithmetization Process)
[Arithmetization](#The Arithmetization Process) is the process of converting a computational statement (code) into a system of polynomial equations.

1.  **Computation Trace:** Define the execution as a table of register states.
2.  **Constraint System:**
    * **R1CS (Rank-1 Constraint System):** Converts logic into linear algebra ($A \cdot z \circ B \cdot z = C \cdot z$). Used heavily in SNARKs.
    * **AIR (Algebraic Intermediate Representation):** Expresses constraints as transition functions between rows of a trace table. Used in STARKs.
    * **Plonkish:** Uses custom gates and general constraints ($q_L a + q_R b + ... = 0$).

### Step 2: [Polynomial Commitments](#Polynomial Commitments)
Once the circuit is arithmetized into polynomials, the prover "commits" to them. The Verifier can later ask for evaluations of these polynomials at specific points to check the constraints.
* **KZG:** Constant-sized proofs, fast verification, requires Trusted Setup (Pairing-based).
* **FRI:** Used in STARKs. Hash-based, quantum-resistant, transparent, but larger proofs.
* **IPA:** Used in Bulletproofs. No trusted setup, slower verification.

---

## Types of Proof Systems

### Interactive vs. Non-Interactive (NIZK)
* **Interactive:** Requires multiple rounds of back-and-forth communication (*e.g.*, Schnorr).
* **Non-Interactive:** The prover sends a single message. This is achieved *via* a [trusted setup](The Zero-Knowledge Trusted Setup (CRS Ceremony)) or the [Fiat-Shamir Heuristic](), which replaces the Verifier's random challenges with the output of a cryptographic hash function.

### System Comparison
The three main families of ZKPs are *SNARKs*, *STARKs*, and *Bulletproofs*. Here is a brief comparison ($n$ denotes the number of gates):

| Property | SNARK | STARK | Bulletproofs |
| :--- | :--- | :--- | :--- |
| **Full Name** | Succinct Non-interactive Argument of Knowledge | Scalable Transparent Argument of Knowledge | N/A |
| **Trusted Setup** | **Required** (Usually) | **None** (Transparent) | **None** |
| **Proof Size** | Constant; ~250 B to 1kB | $\Theta(\log^2 n)$, ~45KB to >200kB in practice | $\Theta(\log n)$, ~1KB in practice |
| **Verifier Time** | Constant, Fastest (~1ms) | $\Theta(\log^2 n)$, Medium (a few ms) | $\Theta(n)$ Slow (15ms to seconds) |
| **Crypto Assumption**| Elliptic Curve Pairings + Knowledge-of-Exponent Assumption | Hash Functions (Post-Quantum) | Discrete Log |
| **Best Use Case** | L2 Rollups (ZkSync, Aztec), Mobile clients | High-throughput DEX (StarkNet), Data availability | Privacy coins (Monero), Range proofs |

#### **SNARKs: The "Gold Standard" for Size**

**Proof size:** ~250 bytes to 1 KB

SNARKs are designed to be "Succinct." Their proof size is constant, meaning the proof doesn't get bigger even if the computation you're proving is massive.

**Commonly-used setups:**

- **Groth16 (Smallest):** ~260–280 bytes. It consists of just 3 elliptic curve elements.
- **PLONK (Universal):** ~450–900 bytes. It is slightly larger than Groth16 but still fits comfortably in a single network packet.

#### **Bulletproofs: The "Middle Ground"**

**Proof size:** ~1 KB to 2 KB

Bulletproofs have a logarithmic proof size.

- **Small Circuits:** For a simple range proof (e.g., proving your age is > 18), the proof is about 700 bytes to 1 KB.
- **Scaling:** Even for very large computations, the logarithmic growth keeps the size manageable (usually under 2 KB). This is why they are popular for confidential transactions in Monero.

#### **STARKs: The “Heavylifters”

**Proof size:** ~45 KB to 200+ KB

STARKs are significantly larger because they don't use “compact” elliptic curve math; instead, they use Hash Functions and Merkle Trees to achieve quantum resistance and transparency (no trusted setup).

- **The Cost of Transparency:** Because you have to include many Merkle paths in the proof to convince the verifier, the size jumps to dozens or hundreds of kilobytes.
- **L2 Usage:** On networks like Starknet, these large proofs are “batched” so that one 100 KB proof covers 10,000 transactions, making the cost-per-transaction very low despite the large total size.

Think of a SNARK like a sealed wax signet: it's tiny and proves the king signed the letter, but you had to trust the person who made the signet ring (the Trusted Setup). A STARK is more like a massive legal binder: it contains so much evidence and cross-referenced footnotes (Merkle paths) that you don't need to trust anyone—the evidence is all right there in the binder.


---

## Advanced Concepts & Frontiers

### Recursion & Aggregation
To scale ZKPs, we can verify proofs *inside* other proofs.
* **Recursion:** A ZKP circuit that verifies a ZKP proof. Allows for infinite chains of computation *via* Incrementally Verifiable Computation (IVC) (e.g., Halo2):

  * IVC is a powerful technique that allows a Prover to prove the correctness of a long-running, multi-step computation by providing a proof for each step that “incorporates” the proof of the previous step. Instead of generating one massive proof for an entire 10-hour computation, IVC allows you to:

    1. Perform Step 1 and generate Proof 1.

    2. Perform Step 2 and generate Proof 2, which proves both Step 2 and that Proof 1 was valid.

    3. Continue this until the end.

    The final result is a single, small proof that convinces a Verifier the *entire* chain of computation was performed correctly. The Verifier only needs to check the very last proof in the chain to be sure that every preceding step—no matter how many there were—was correct.

* **Folding (Nova):** A newer technique to accumulate multiple instances of a computation into a single instance before proving.

  * The basic idea of folding is to take two mathematical problems (specifically [R1CS](#Program to Constraints) instances) and combine them into one. Imagine you have an existing proof of $N$ steps (the **accumulator**) and a new step you just performed. To “fold” them, the Verifier picks a random number $r$. The Prover then creates a linear combination of the two instances: $\text{New Instance} = \text{Instance}_1 + r \cdot \text{Instance}_2$. Because of the properties of the commitment scheme (which is *additively homomorphic*), the Verifier can verify this combination very cheaply without looking at the full witnesses.
  * **The Problem: The "Cross-Terms".** If you simply add two quadratic equations together, you get messy “cross-terms” ($2ab$) that break the original structure. Nova solves this by using a “relaxed” version of the math that includes an *error vector* ($E$) and a *scaling factor* ($u$). The folding process basically shuffles these cross-terms into the error vector $E$. The Verifier only needs to check a small commitment to this error vector rather than the whole thing.


The table below gives a brief conparison of traditional recursion and Nova:

| **Traditional Recursion**                                    | **Nova Folding**                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Heavy:** Each step verifies a full SNARK proof (FFTs, pairings). | **Light:** Each step only performs a few group additions and scalar multiplications. |
| **Complex:** Requires “Cycles of Curves” to avoid non-native arithmetic. | **Simple:** Works on any standard elliptic curve.            |
| **Slow Prover:** High memory and CPU usage.                  | **Fast Prover:** Up to 100x faster than traditional methods like Plonky2. |

### Mathematical Foundations
ZKPs rely on the hardness of specific problems:
1.  **Discrete Logarithm:** Given a finite group $G$, an element $g$ of $G$, and a natural integer $a$, it is (under some conditions) hard to find $a$ from $g^a$. It is worth noting that “hard” in this context usually means that the best-known classical algorithms (like Pollard's Rho) take exponential time in the size of group elements when represented in bits ($\Theta(|G|^{1/2})$). However, this is where quantum resistance comes in: Shor's Algorithm can solve the Discrete Logarithm Promlem (DLP) in polynomial time, which is why STARKs (which don't rely on the DLP) are considered "quantum-safe" while SNARKs and Bulletproofs are not. The conditions for this problem to be classically hard are: 

    1.  **Conditions on the Group $G$:**
        * **Order of the Group:** The number of elements in the group, denoted as $|G|$, must be a **large prime number** (or have a very large prime factor). If $|G|$ is small, an attacker can use brute force. If $|G|$ is a composite number with many small factors, an attacker can use the Pohlig-Hellman algorithm to solve the problem piece-by-piece.
        * **Cyclic Nature:** The group (or the subgroup used) must be **cyclic**, meaning every element in it can be expressed as a power of a single “generator” element $g$.
        * **Structure:** Common secure groups include:
          - **Multiplicative groups of finite fields:** $(\mathbb{Z}_p^\times)$, where $p$ is a large prime. (Note: for this group, specialised attacks based on the General Number Field Sieve give sub-exponential runtimes.)
          - **Elliptic Curve groups:** Points on a curve over a finite field. These are currently the standard for ZKPs because they offer the same security as finite fields but with much smaller keys.
    2.  **Conditions on the Element $g$ (The Generator):**
        - **High Order:** The element $g$ must be a **primitive root** or a generator of a large prime-order subgroup. If you accidentally pick a $g$ that generates a tiny subgroup (e.g., one that only produces 100 different values), an attacker can find $a$ in milliseconds.
        - **Publicly Known:** In ZKPs, $g$ is almost always a public constant defined by the protocol (e.g., the generator point $G$ in the Secp256k1 curve used by Bitcoin).
    3.  **Conditions on the Integer $a$ (The Secret):**
        - **Uniform Randomness:** The secret $a$ must be chosen **uniformly at random** from $G$. If $a$ is predictable (*e.g.*, “12345” or a low-entropy password), the group's strength doesn't matter.
        - **Large Size:** To resist the **Baby-step Giant-step** or **Pollard's Rho** algorithms, $a$ must be large enough. By 2025 standards, $a$ should typically be at least **256 bits** long (roughly $2^{256} \approx 10^{77}$, which is close to the estimated number of atoms in the observable universe, betwen $10^{78}$ and $10^{82}$).

2.  **Elliptic Curve Pairings:** Bilinear maps $G_1 \times G_2 \to G_T$ that allow checking multiplication relationships in encrypted data, where $G_1$, $G_2$, and $G_T$ are three distinct cyclic groups of the same prime order. 

    * $G_1$ is the **Base Field Group**: A subgroup of points on the elliptic curve whose coordinates are simple numbers in the base field $\mathbb{F}_p$. This is usually the “smallest” and fastest group to work with.
    * $G_2$ is the **Extension Field Group**: A subgroup of points on the same curve, but the coordinates are “complex” numbers in an extension field (like $\mathbb{F}_{p^2}$ or $\mathbb{F}_{p^{12}}$). Elements here are much larger and slower to compute than in $G_1$.
    * $G_T$ is the **Target Group**: This is not an elliptic curve group. It is a multiplicative subgroup of a large finite field (typically $\mathbb{F}_{p^{12}}$). The result of a pairing always lands here.

    Normally, with standard Elliptic Curve math, you can only add encrypted numbers: if you have $A = g^a$ and $B = g^b$, you can compute $g^{a+b}$ by adding the points; but you cannot compute $g^{ab}$ just by looking at $A$ and $B$. A pairing $e: G_1 \times G_2 \to G_T$ allows you to check if $a \times b = c$ without knowing $a, b,$ or $c$: $e(g_1^a, g_2^b) = e(g_1, g_2)^{ab}$. If you also have a value $C = g_1^c$, you can check if $e(g_1^a, g_2^b) = e(C, g_2)$. If the equation holds, then $ab = c$ must be true.

3.  **Knowledge of Exponent Assumption (KEA):** If you can output $g^a$ and $g^{\alpha a}$ for some secret $\alpha$, you must know $a$.

    * The Verifier (or the setup ceremony) publishes two values: $g$ and $g^\alpha$.
      * In a real SNARK, the setup provides many pairs: $(g^1, g^\alpha), (g^s, g^{s\alpha}), (g^{s^2}, g^{s^2\alpha}), \dots$.
    * They then destroy $\alpha$ so that nobody in the world knows its numerical value.
    * If the prover knows $a$, she can raise the two values to the power $a$ to get $g^a$ and $g^{\alpha a}$. 
      * In a real SNARK, they pick some coefficients $c_i$ and compute $A = \prod (g^{s^i})^{c_i} = g^{\sum c_i s^i}$, $B = \prod (g^{s^i \alpha})^{c_i} = g^{\alpha \sum c_i s^i}$.
    * The KEA states that if the prover can generate two values $A$ and $B$ such that $B = A^\alpha$, then must know an exponent $a$ such that $A = g^a$.
      * Or, in a real SNARK, the coefficients $c_i$.

---

## The Arithmetization Process

Arithmetization is the crucial first step in any Zero-Knowledge Proof system. It is the process of converting an arbitrary computation (like a program or a financial transaction) into a system of mathematical constraints that can be efficiently checked using polynomial evaluations. The goal is to translate the logic of a program into a set of equations where the solution space is the set of valid execution traces.

### Program to Constraints

The type of constraint system used depends on the ZKP scheme:

| Constraint System                               | Scheme Used In                | Description                                                  |
| ----------------------------------------------- | ----------------------------- | ------------------------------------------------------------ |
| **R1CS** (Rank-1 Constraint System)             | SNARKs (e.g., Groth16, PLONK) | This converts the computation into a system of linear algebra equations: $$A \cdot z \circ B \cdot z = C \cdot z$$ Where $A, B, C$ are matrices defining the circuit, $z$ is the wire vector (public inputs, private witness, and intermediate values), and $\circ$ is the Hadamard (element-wise) product. |
| **AIR** (Algebraic Intermediate Representation) | STARKs                        | This models the computation as an execution trace (a 2D table of register states). Constraints are expressed as transition functions between consecutive rows (steps) in the trace: $\forall i, F(\mathsf{trace}[i], \mathsf{trace}[i+1]) = 0$ |
| **Arithmetic Circuits** (often via R1CS) | Bulletproofs   | Models computation as a directed acyclic graph of addition and multiplication gates. Unlike SNARKs, Bulletproofs do not require a [QAP](#Constraints to Polynomials (The QAP Step)) or trusted setup; instead, they use a *recursive inner product argument* to prove that the input/output of the gates satisfy the circuit constraints. |

**R1CS (SNARKs):** Is like a **static snapshot** of a circuit. You define the whole “wiring” of the machine upfront as a set of matrices. It is very efficient but rigid.

**AIR (STARKs):** Is like a **physics simulation over time**. You don't define every wire; you define the “laws of motion” (the transition function) and apply them repeatedly to a state vector.

**Bulletproofs:** Is like a **pedantic accountant**. It walks through the circuit and proves that for every multiplication gate $a \cdot b = c$, the relationship holds, using a clever logarithmic-sized proof to compress all these checks into one.

### Constraints to Polynomials (The QAP Step)

For SNARKs using R1CS, the next step often involves the *Quadratic Arithmetic Program (QAP)*:

1. **R1CS to QAP:** The matrices $A, B, C$ (and the solution vector $z$) are encoded into a single set of polynomials. This is typically done using Lagrange interpolation.
2. **Quotient Check:** The entire R1CS constraint system is converted into a single polynomial check. The check ensures that the constraint polynomial $T(x)$ divides the polynomial derived from the solution: $T(x)$ must divide $P(x) = (A(x) \cdot z) \cdot (B(x) \cdot z) - (C(x) \cdot z)$.
3. **Low-Degree Check:** The core task for the Prover then becomes proving that $P(x)$ satisfies a low-degree check, meaning the polynomial has a restricted form.

### Modern Approaches (Plonkish & IOPs)

Modern systems, like PLONK and STARKs, often use more flexible arithmetization techniques:

* **Plonkish Arithmetization:** This uses a generic gate constraint that can be customized: $$q_L \cdot a + q_R \cdot b + q_O \cdot c + q_M \cdot a \cdot b + q_C = 0$$ This formula combines linear terms ($a, b, c$), a multiplication term ($a \cdot b$), and constant terms into one equation. It allows for the use of custom gates for optimized circuit implementations.
* **Polynomial IOPs (Interactive Oracle Proofs):** This is the high-level framework that underlies both Plonkish and STARKs. It separates the proof into:
    * **Oracles:** Prover provides polynomial data structures.
    * **Queries:** Verifier asks for evaluations at specific points.
    * **Consistency Checks:** Constraints between the polynomials are verified.

---

## Cryptographic Primitives

### Polynomial Commitments

A **Polynomial Commitment Scheme** is a cryptographic tool that allows a Prover to:

1. **Commit** to a polynomial $P(x)$ (or a set of polynomials) by generating a short, constant-sized commitment $C$. This is like digitally signing the polynomial.

2. Later, **Prove** that the polynomial evaluates to a specific value $y$ at a specific point $z$ (i.e., $P(z) = y$) by providing a compact proof $\pi$.

3. The Verifier can then check the commitment $C$, the evaluation point $z$, the claimed value $y$, and the proof $\pi$ to be convinced, without seeing the full polynomial $P(x)$ itself.

The choice of commitment scheme heavily influences the performance and security properties (like trusted setup, quantum resistance, and proof size) of the final ZKP system.

| Scheme | ZKP System Used In | Security Basis | Trusted Setup? | Proof Size / Verification Time |
| ------ | ------------------ | -------------- | -------------- | ------------------------------ |
| **KZG** (Kate-Zaverucha-Goldberg) | SNARKs (e.g., PLONK, Groth16) | Elliptic Curve Pairings (Bilinear Groups) | Yes (Requires CRS) | Constant-sized proofs, very fast verification |
| **FRI** (Fast Reed-Solomon IOPP) | STARKs	Hash Functions (collision resistance) | No (Transparent setup) | Larger proofs, medium verification |
| **IPA** (Inner Product Argument) | Bulletproofs | Discrete Logarithm Problem | No (Transparent setup) | Logarithmic proof size, slowest verification |
| **DARK** (Diophantine Arguments of Knowledge) | Varied | Groups of Unknown Order | Varies | Aimed at post-quantum security |

### Cryptographic Accumulators

These are compact data structures that represent a set of elements and allow for efficient, short proofs of membership or non-membership.

* **Merkle Trees:** Use hash nodes for logarithmic proof size.
* **RSA Accumulators:** Use the Strong RSA Assumption to represent the set $\{e_i\}$ as a single value: $acc = g^{\prod (e_i)} \mod N$.

### Elliptic Curves

These are the mathematical groups that provide the underlying security for most SNARKs:

* **Pairing-Friendly Curves:** (e.g., BLS12-381, BN254) are necessary to enable the bilinear maps (pairings) used by schemes like KZG.

* **Curve25519/Ed25519:** Used for faster operations in transparent schemes.

### ZK-Friendly Hash Functions

Traditional hash functions like SHA-256 are slow when translated into ZKP circuits. ZK-friendly hashes are designed to minimize the number of constraints in the arithmetization step, making proving faster.

**Examples:** Poseidon, Rescue, Pedersen.

---

## The Zero-Knowledge Trusted Setup (CRS Ceremony)

The **Trusted Setup** is a foundational component for many ZK-SNARK protocols, especially those using pairing-based polynomial commitments like KZG (e.g., Groth16, PLONK).

It is the process of generating the **Common Reference String (CRS)**, which consists of public parameters required by both the Prover and the Verifier to compute and check proofs.

### The Common Reference String (CRS)

The CRS is a set of encrypted values derived from a secret, random number (often called the $\tau$ or "toxic waste"). It looks like this:
$$
    \mathsf{CRS} = \left(\{g^{\tau^i}\}_{i=0}^d, \{g^{\alpha \cdot \tau^i}\}_{i=0}^d, ...\right)
$$

* $g$ is the generator of an elliptic curve group.
* $\tau$ and $\alpha$ are the random secret values used to create the parameters.
* $d$ is the maximum degree of the polynomial that the system can handle.

The crucial requirement for the system's security is that the secret values ($\tau$ and $\alpha$) must be destroyed immediately after the CRS is generated.

### The Knowledge-of-Exponent Assumption (KEA)

The security of SNARKs relies heavily on the Knowledge-of-Exponent Assumption (KEA). In the context of the CRS, KEA means:

> If the adversary can compute $g^a$ and $g^{\alpha a}$ using the public CRS, they must know the exponent $a$.

This assumption allows the Verifier to be confident that the Prover knows the witness ($w$) when they produce a valid proof, thereby enforcing the Knowledge-Soundness property.

### The Trusted Setup Risk ("Toxic Waste")

The major drawback of non-transparent setups is the security risk associated with the secret $\tau$ (the "toxic waste"):

* **Risk:** If a malicious party retains and uses the secret $\tau$ that generated the CRS, they could potentially forge valid proofs for false statements ($x \notin L$) without knowing the actual witness ($w$). This would break the Soundness of the system.

* **Mitigation:** To ensure the secret is destroyed, the setup is performed through a Multi-Party Computation (MPC) Ceremony.

### The MPC Ceremony

A setup ceremony involves many participants (often hundreds or thousands) collaborating to generate the CRS:

1. Each participant generates their own random secret and contributes to the CRS.

2. The final CRS is secure as long as at least one honest participant successfully destroys their individual secret share.

3. This dramatically minimizes the risk, as an attacker would need to compromise every single participant in the ceremony.

### Types of Trusted Setup

| Setup Type | Description | System Example |
| ---------- | ----------- | -------------- |
| Per-Circuit | The CRS must be generated specifically for every new computational circuit. | Groth16
| Universal | The CRS is generated once and can be reused for any circuit up to a certain size. | PLONK

---

## The Fiat-Shamir Heuristic

The **Fiat-Shamir heuristic** is a technique used to transform an **interactive** proof (where a Prover and Verifier talk back and forth) into a **non-interactive** proof (where the Prover sends a single, standalone message).

The Verifier's job is to provide a random "challenge." The Fiat-Shamir heuristic essentially replaces the human Verifier with a **cryptographic hash function**.

### **How the Transformation Works**

In a typical interactive "Sigma protocol," there are three steps:

1. **Commitment:** The Prover sends a message $t$ (a random "blinded" version of their secret).
2. **Challenge:** The Verifier sends a random value $c$.
3. **Response:** The Prover sends a response $z$ based on their secret, the randomness in $t$, and the challenge $c$.

**With Fiat-Shamir, the interaction is removed:**

- Instead of waiting for the Verifier to send $c$, the Prover computes the challenge themselves using a hash function: **$c = \text{Hash}(x, t)$**.
- Because a cryptographic hash function is "unpredictable," the Prover cannot "choose" a challenge that makes a fake proof work.14 They are effectively forced to use a random-looking value that depends entirely on their initial commitment.

### **Why It Matters**

- **Scalability:** The Prover can generate the proof once and post it on a blockchain or a public server.15 Anyone (the "Public") can verify it at any time without the Prover needing to be online.
- **Transferability:** Since the challenge is fixed by the hash of the data, the proof is "bound" to that specific statement and commitment. It can be verified by multiple parties without re-running the protocol.
- **Security:** This transformation is considered secure in the **Random Oracle Model (ROM)**, assuming the hash function behaves like a perfectly random function.

### **The "Sudoku" Connection**

In the variant of the above [Sudoku example](#Sudoku board), if you used Fiat-Shamir, you wouldn't wait for a friend to tell you "Check the Rows." Instead, you would take a photo of your face-down card layout, **hash that photo**, and use the resulting hash value to determine which constraint (Rows, Cols, or Grids) you must reveal. Because you committed to the cards *before* knowing the hash, you couldn't have cheated!

***

## Appendix: Notations Reference

| Symbol | Meaning |
| :--- | :--- |
| $\lambda$ | Security parameter (128-256 bits) |
| $L$ | Language of valid statements |
| $w$ | Witness (private input) |
| $x$ | Statement (public input) |
| $R$ | Relation, where $R(x,w)=1$ |
| $\mathbb{F}_p$ | Finite field of prime order $p$ |
| $zk$ | Zero-knowledge modifier |
| $\mathcal{E}$ | Knowledge Extractor |
| $\circ$ | Hadamard (element-wise) product |
