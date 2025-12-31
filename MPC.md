# Multi-Party Computation (MPC) — A technical primer

## Introduction and Formal Definition

**Multi-Party Computation (MPC)** is a subfield of cryptography that allows a set of $n$ parties (where $n$ is an integer larger than $1$) to jointly compute a function $f$ of $n$ variables (hereafter denoted by $x_1, x_2, \dots, x_n$) over their private inputs $x_1, x_2, \dots, x_n$. The fundamental goal is to provide the output of the function while ensuring that no party learns anything about the other participants' inputs beyond what is revealed by the output itself.

Formally, an MPC protocol satisfies two primary constraints:

* **Privacy:** For any party $P_i$ (for $i$ in $[\![1,n]\!]$), the information gained during the protocol (the “View”) is effectively the same regardless of the other parties' specific private inputs, provided the output remains constant.
* **Correctness:** If the parties follow the protocol, the final output is guaranteed to be $f(x_1, \dots, x_n)$.

### Adversary Models and Security Guarantees

The security of an MPC protocol is defined by how many corrupt parties it can withstand and the behavior of those parties.

#### Adversary Behavior

* **Semi-Honest (Passive):** Parties follow the protocol but attempt to learn private information from the intermediate data they see.
* **Malicious (Active):** Parties may deviate from the protocol, provide false inputs, or quit early to prevent others from getting the output.

#### Threshold Structures

Protocols are generally categorized by the **threshold** $t$ of corrupted parties they can tolerate:

* **Honest Majority:** Requires $t < n/2$. These often rely on **Information-Theoretic Security**, meaning they are secure even against an adversary with infinite computing power.
* **Dishonest Majority:** Secure even if $n-1$ parties are corrupt. These typically rely on **Computational Hardness** (e.g., the Learning With Errors (LWE) problem or Decisional Diffie-Hellman (DDH)).

#### A note on Correctness

To prevent a malicious party from corrupting the result, modern protocols use extra mathematical checks:

- **Information-Theoretic MACs:** In protocols like **SPDZ**, every secret share is “tagged” with a Message Authentication Code (MAC). If a party tries to change their share during a calculation, the MAC check will fail at the end, and the result will be discarded.
- **Zero-Knowledge Proofs (ZKPs):** A party can be required to provide a proof that says: *“I am not showing you my share, but here is a proof that this share was calculated correctly according to the protocol rules.”*
- **Error-Correcting Codes:** In honest-majority settings using Shamir's Secret Sharing, if a party provides an incorrect point, it becomes an “outlier”. If you have enough extra points, you can use **Reed-Solomon decoding** to identify and ignore the “noisy” (malicious) data point and still find the correct polynomial.

To take the example of a running competition, MPC can't stop you from lying about your personal best (*Input Truthfulness*), but it can ensure that once the race starts, no one can take a shortcut or jump ahead a mile without being disqualified (*Protocol Correctness*).

### Fundamental Building Blocks

Hereafter, $\mathbb{F}$ denotes a field.

#### Shamir’s Secret Sharing (SSS)

[Shamir's Sceret Sharing](#Shamir Secret Sharing and Lagrange Interpolation) is used for $n$-party computation in honest-majority settings. It relies on the mathematical principle that $t+1$ points are required to uniquely define a polynomial of degree $t$.

To share a secret $s \in \mathbb{F}$:

1. Pick a random polynomial $q(x)$ of degree $t$ such that $q(0) = s$.
2. For $i \in [\![1, n]\!]$, give the party $P_i$ the share $y_i = q(i)$.
3. Any $t+1$ parties can use **Lagrange Interpolation** to reconstruct $q(0)$.

#### Oblivious Transfer (OT)

[Oblivious Transfer](#Oblivious Transfer) (OT) is the “atomic unit” of MPC. In a 1-out-of-2 OT, a sender has two messages $(m_0, m_1)$, and a receiver has a bit $b \in \{0, 1\}$. The receiver gets $m_b$ without the sender knowing which they chose, and without the receiver learning the other message. Modern protocols use [OT Extension](OT Extension) to generate millions of OTs using only a few expensive public-key operations.

#### Garbled Circuits (Yao’s Protocol)

Primarily used for 2-party computation (2PC).

1. The function $f$ (known to both parties) is represented as a Boolean circuit of AND and XOR gates.
2. The *Garbler* encrypts the truth table of each gate using random labels for the two possible values $0$ and $1$, chosen independently for each wire of the circuit:
   1. The Garbler chooses labels for the values $0$ and $1$ for each wire.
   2. For each column of each gate, (two input columns and one output column), each value ($0$ or $1$) is replaced by its label.
   3. The output entries of each table are encrypted with a key dependent on the two input labels.
   4. The output values of each table are shuffled.

3. The *Evaluator* receives 
   * the garbled tables,
   * the encrypted inputs from the Garbler,
   * through OT, the labels corresponding to their input.
4. The Evaluator decrypts the circuit gate-by-gate to find the label of the final output. 
5. She sends it to the Garbler, who is able to retrive the corresponding value.

### Arithmetic MPC and Multiplication

While [addition is simple](#Addition: The “Free” Operation) in additive secret sharing (parties just add their local shares), multiplication is more complex.

#### Beaver Triples (Multiplication Gates)

To multiply two shared values $[a]$ and $[b]$, parties use precomputed [Beaver Triples](#Multiplication: The Challenge) $([u], [v], [w])$ where $w = uv$.

1. Parties compute $[d] = [a] - [u]$ and $[e] = [b] - [v]$.

2. Parties *open* (reveal) $d$ and $e$.

3. The shares of the product $[ab]$ are computed locally as:
   $$
   [ab] = [w] + e[u] + d[v] + de
   $$

### Modern Frontiers and Optimization

| Technique | Description |
| --- | --- |
| **SPDZ** | A protocol for dishonest majority using a preprocessing phase to generate authenticated Beaver triples. |
| **FHE Integration** | Using Fully Homomorphic Encryption to offload heavy computation in MPC, reducing communication rounds. |
| **UC Security** | “Universally Composable” security, ensuring a protocol remains secure even when run alongside other instances of itself. |
| **MPC-in-the-Head** | A technique to build Zero-Knowledge Proofs (ZKPs) by simulating an MPC protocol internally. |

---

## Shamir Secret Sharing and Lagrange Interpolation

In MPC, we often use **Shamir’s Secret Sharing** to distribute a secret $s$ among $n$ parties such that any $t+1$ parties can reconstruct it, but $t$ or fewer parties learn absolutely nothing.

### The Setup: Encoding the Secret

To share a secret $s \in \mathbb{F}$ (where $\mathbb{F}$ is a finite field):

1. **Define the Degree:** We choose a threshold $t$. We want any $t+1$ people to be able to recover the secret.

2. **Generate the Polynomial:** We construct a random polynomial $q(x)$ of degree at most $t$:

   $$q(x) = a_0 + a_1x + a_2x^2 + \dots + a_tx^t$$

3. **Embed the Secret:** We set the constant term $a_0 = s$. All other coefficients $(a_1, \dots, a_t)$ are chosen uniformly at random from the field. (The polynomial $q$ has degree $t$ with probability $1 - 1 / \lvert F \rvert$.)

4. **Distribute Shares:** Each party $P_i$ is assigned a unique, public evaluation point $x_i$ (usually $x_i = i$). Their private share is the value $y_i = q(x_i)$.

### The Reconstruction: Lagrange Interpolation

The core “magic” is that a polynomial of degree $t$ is uniquely determined by $t+1$ distinct points. If a subset of $t+1$ parties pool their shares $(x_1, y_1), (x_2, y_2), \dots, (x_{t+1}, y_{t+1})$, they can find $q(x)$ using the **Lagrange basis polynomials**.

#### The Basis Polynomials

For each share $j$ in our subset, we define a basis polynomial $\ell_j(x)$ that is equal to $1$ at $x_j$ and $0$ at all other $x_m$ in our subset:
$$
\ell_j(x) = \prod_{\substack{1 \le m \le t+1 \\ m \neq j}} \frac{x - x_m}{x_j - x_m}
$$

#### Finding the Secret

The full polynomial is the linear combination of these bases:
$$
q(x) = \sum_{j=0}^{t+1} y_j \ell_j(x)
$$
Since we only care about the secret $s$, which is $q(0)$, we don't even need to reconstruct the full polynomial. We just evaluate the basis polynomials at zero:
$$
s = q(0) = \sum_{j=0}^{t+1} y_j \ell_j(0)
$$

### Why This Works for Privacy

- If you have $t+1$ points, the system of linear equations is determined; there is exactly one solution for $a_0$.
- If you only have $t$ points, for *every possible* secret $s' \in \mathbb{F}$, there exists exactly one polynomial of degree $t$ that passes through your $t$ points and has $q(0) = s'$.

Consequently, to an adversary with only $t$ shares, all possible secrets are equally likely. The “View” of the adversary provides zero information about the secret.

### Example: $t=1$ (2-out-of-$n$ sharing)

Imagine we want to share a secret $s = 5$ with a threshold $t=1$ (we need 2 people to recover it).

1. **Polynomial:** We pick a random $a_1 = 3$. Our polynomial is $q(x) = 5 + 3x$.

2. **Shares:**

   * Party 1 gets $(1, 8)$

   - Party 2 gets $(2, 11)$

3. **Reconstruction:** Using these two points to find the intercept: $\ell_1(0) = (0 - 2) / (1 - 2) = 2$, $\ell_2(0) = (0 - 1) / (2 - 1) = -1$, and $S = (8 \cdot 2) + (11 \cdot -1) = 16 - 11 = 5$.

---

## Operations on shares

In MPC, the ability to perform calculations on encrypted or shared data is what makes the technology powerful. The linearity of Shamir’s Secret Sharing (SSS) makes addition trivial, while multiplication requires a change in perspective.

### Addition: The “Free” Operation

Addition in Shamir’s Secret Sharing is **locally computable**. This means parties can add their shares together without sending any messages to each other.

Suppose we have two secrets, $A$ and $B$, shared using polynomials $q_A(x)$ and $q_B(x)$ of degree $t$.

- Party $P_i$ holds shares $y_{A,i} = q_A(i)$ and $y_{B,i} = q_B(i)$.

- To compute the shares of the sum $S = A + B$, the parties simply add their local shares: $y_{S,i} = y_{A,i} + y_{B,i}$.


**Why this works:**

Because of the linearity of polynomials, the sum of two polynomials of degree at most $t$ is itself a polynomial of degree at most $t$: the polynomial $q(x)$ defined by
$$
q(x) = q_A(x) + q_B(x)
$$
has degree at most $t$ with its $t$ non-constant coefficients (statistically) uniformly sampled from $\mathbb{F}$, so the privacy threshold is preserved. The constant term of $q(x)$ is $q_A(0) + q_B(0)$, which is exactly $A + B$. 

### Multiplication: The Challenge

Multiplication is significantly harder for two main reasons:

1. **Degree Doubling:** If you multiply two polynomials of degree $t$, the resulting polynomial has degree $2t$. To reconstruct a degree $2t$ polynomial, you would need $2t+1$ parties, which might exceed your available honest majority.
2. **Randomness Loss:** The product of two random polynomials is no longer as “random” (its coefficients are correlated), which can leak information.

#### The Solution: Beaver Triples

To solve this, modern MPC protocols (like **SPDZ**) use a technique called **Beaver Triples**, which shifts the heavy lifting to a “preprocessing” phase. 

In the following, for an element $x$ of $\mathbb{F}$, $[x]$ denotes shares of $x$.

##### The Preprocessing Phase

Before the parties know their actual inputs $A$ and $B$, they (or a trusted setup) generate a set of random “triples” $([u], [v], [w])$ such that $u$ and $v$ are sampled uniformly and independently from $\mathbb{F}$, and $w = u \cdot v$. These values are secret-shared among the parties and have no relation to the actual data.

##### The Online Phase

When it is time to multiply the real shares $[a]$ and $[b]$:

1. **Mask the inputs:** Parties compute and *open* (reconstruct) two values:

   - $d = a - u$
   - $e = b - v$

   Because $u$ and $v$ are secret and uniformaly sampled from $\mathbb{F}$, $d$ and $e$ reveal nothing about $a$ or $b$.

2. Compute the product share: Every party can now compute their share of the product $[ab]$ using only the public constants $d, e$ and their shares of the triple: $[ab] = [w] + e[u] + d[v] + de$.


##### The Algebra behind it:

Mathematically, we are just expanding the identity:

$$ab = (d+u)(e+v) = de + dv + ue + uv$$

Since $uv = w$, this simplifies to the formula above. Every term is either a public constant ($de$) or a constant multiplied by a share ($e[u]$ and $d[v]$), both of which are locally computable.

### Comparison of Complexity

| **Operation**      | **Communication Cost**           | **Mathematical Tool**          |
| ------------------ | -------------------------------- | ------------------------------ |
| **Addition**       | Zero (Local)                     | Linearity of Polynomials       |
| **Multiplication** | High (Requires “opening” $d, e$) | Beaver Triples / Preprocessing |

***

## Oblivious Transfer

The most intuitive way to understand **1-out-of-2 Oblivious Transfer (OT)** is through a protocol based on the **Diffie-Hellman** exchange.

In this scenario, we have a **Sender** (Alice) and a **Receiver** (Bob). Alice has two messages, $m_0$ and $m_1$. Bob has a selection bit $b \in \{0, 1\}$ and wants to receive $m_b$ without Alice knowing which one he picked, and without Bob learning the other message.

### The Conceptual Setup

Imagine Alice has two locked boxes. Bob wants the key to one of them, but:

1. Alice must not know which key Bob took.
2. Bob must only be able to open the box he chose.

### A Mathematical Example (Bellare-Micali Protocol)

This protocol relies on the **Discrete Logarithm Problem**, which is a staple in the computational hardness foundations of MPC.

#### Step 1: Initialization

Alice and Bob agree on a cyclic group $G$ (like a prime-order subgroup of $(\mathbb{Z}/p\mathbb{Z})^\times$ for some positive integer $p$) with generator $g$. Alice also picks a random element $c \in G$ and sends it to Bob.

#### Step 2: Bob’s Choice

Bob wants message $m_b$. He picks a random exponent $k$:

- If he wants $m_0$ ($b=0$), he sets his first public key $PK_0 = g^k$ and his second $PK_1 = c / g^k$.
- If he wants $m_1$ ($b=1$), he sets $PK_0 = c / g^k$ and $PK_1 = g^k$.

Bob sends $PK_0$ to Alice.

**The "Oblivious" Part:** Alice receives $PK_0$. She can calculate $PK_1 = c / PK_0$. However, because $c$ was random, $PK_0$ looks like a random element of the group to her. She has no way of knowing if Bob knows the discrete log of $PK_0$ or the discrete log of $PK_1$.

#### Step 3: Alice’s Encryption

Alice treats $PK_0$ and $PK_1$ as public keys. She picks two random exponents $r_0, r_1$ and sends two pairs (essentially ElGamal encryptions) to Bob:

1. **For $m_0$:** $(g^{r_0}, m_0 \cdot PK_0^{r_0})$
2. **For $m_1$:** $(g^{r_1}, m_1 \cdot PK_1^{r_1})$

#### Step 4: Bob’s Decryption

Bob receives both pairs. Since he knows $k$ such that $PK_b = g^k$, he can decrypt the message he chose:
$$
m_b = \frac{m_b \cdot (g^k)^{r_b}}{(g^{r_b})^k}
$$
Because he does *not* know the discrete log of the other public key ($PK_{1-b}$), the other message remains computationally hidden from him.

### Why This Matters for MPC

OT is considered the foundation of MPC because you can build any complex logic gate (like an AND gate) using multiple OTs.

- **For 2-Party Computation:** It’s how the “Evaluator” in a Garbled Circuit gets the specific keys for their input wires without the “Garbler” knowing what those inputs are.
- **Efficiency:** While the math above involves expensive modular exponentiation, modern *OT Extension* allows us to perform a few of these “base OTs” and then use much faster symmetric-key operations (like AES) to generate millions of OTs.

You can think of OT as the ultimate “privacy-preserving API call”: a way to query a database where the server doesn't know what you asked for, and you only get exactly the record you paid for.

### OT Extension

The transition from **Base OT** to **OT Extension** is one of the most significant breakthroughs in practical MPC. It represents a shift from “public-key” complexity to “symmetric-key” efficiency, allowing us to generate millions of OTs per second.

#### The "Why": The Bottleneck of Base OT

As we saw with the Diffie-Hellman example, Base OT requires modular exponentiations or elliptic curve operations.

- These are computationally expensive and thus slow in practice.
- In a complex circuit with millions of gates, performing a Base OT for every single wire would make the protocol unusable for real-world tech applications.

**OT Extension** allows us to perform a small, fixed number of Base OTs (e.g., 128) and extend them into an essentially unlimited number of OTs using only fast symmetric-key operations like **XOR** and **AES-based PRGs** (Pseudo-Random Generators).

#### The IKNP Protocol (The Matrix Trick)

The most famous construction for this is the **IKNP protocol** (named after Ishai, Kilian, Nisim, and Petrank). The core of the protocol involves a clever mathematical “swap” of roles and a matrix transposition.

##### Step 1: Small Base OTs

Alice (the Sender) and Bob (the Receiver) start by performing $k$ Base OTs (where $k$ is a security parameter, typically 128).

**The Twist:** In these base OTs, the roles are reversed. Bob acts as the “Sender” with random seeds, and Alice acts as the “Receiver” picking which seeds she gets based on a secret bitstring $s$.

1. Alice generates a random secret $s \in \lbrace 0, 1 \rbrace^k$.
2. Bob generates $k$ pairs of random seeds $(k_{1,0}, k_{1,1}), (k_{2,0}, k_{2,1}), \dots, (k_{k,0}, k_{k,1})$. 
3. Alice receives one seed from each pair: $q_j = k_{j, s_j}$ *via* base OTs.

##### Step 2: Encoding the Choice Bits

Now, Bob wants to receive $m$ messages, so he has a choice bitstring $r \in \{0,1\}^m$. For each index $j \in \{1, \dots, k\}$:

1. He expands both seeds $k_{j,0}$ and $k_{j,1}$ using a PRG, hereafter called $G$, to a length of $m$ bits:
   - $G(k_{j,0})$
   - $G(k_{j,1})$
2. He computes a column $u_j$ that “masks” his choice bits $r$: $u_j = G(k_{j,0}) \oplus G(k_{j,1}) \oplus r$.
3. Bob sends all $k$ of these $u_j$ vectors to Alice.

##### Step 3: The Symmetry Trick

Alice now has the vectors $u_j$ from Bob and her $k$ seeds ($q_j$). She computes her own matrix columns $t_j$: $t_j = G(q_j) \oplus (s_j \cdot u_j)$.

If you substitute the value of $u_j$ into Alice's equation, you see the magic of the selection bit $s_j$:

- if $s_j = 0$, $t_j = G(k_{j,0})$
- if $s_j = 1$: $t_j = G(k_{j,0}) \oplus r$

In either case, $t_j = G(k_{j,0}) \oplus (s_j \cdot r)$.

Let us call $T$ the matrix whose columns are the vectors $t_j$ (known to Alice) and $V$ the matrix whose columns are the vectors $G(k_{j,0})$ (known to Bob). Then, for each $i$ in $\lbrace 1, 2, \dots, m \rbrace$, $T_i = V_i \oplus (r_i \cdot s)$.

##### Step 4: Turning Rows into Cryptographic Keys

To ensure Bob can only learn the message he chose, Alice uses a **Random Oracle** (a cryptographic hash function like SHA-256), hereafter called $H$, to turn these rows into encryption keys.

For the $i$-th oblivious transfer, Alice prepares two keys:

- $K_{i,0} = H(i, T_i)$
- $K_{i,1} = H(i, T_i \oplus s)$

Alice then encrypts her two messages ($m_{i,0}, m_{i,1}$) using these keys and sends them to Bob: $y_{i,0} = m_{i,0} \oplus K_{i,0}$, $y_{i,1} = m_{i,1} \oplus K_{i,1}$.  Because $H$ (usually based on SHA-256 or AES) is incredibly fast, we can now process millions of these "Extended OTs" in the time it would take to do just a few dozen Base OTs.

Now consider Bob's perspective. He knows $V_i$ and his choice bit $r_i$. He computes his key as $K_{i, \text{Bob}} = H(i, V_i)$.

- if $r_i = 0$: Bob's key $H(i, V_i)$ is exactly $H(i, T_i)$, which is $K_{i,0}$. He decrypts $m_{i,0}$.
- if $r_i = 1$ Bob's key $H(i, V_i)$ is $H(i, T_i \oplus s)$, which is $K_{i,1}$. He decrypts $m_{i,1}$.

Because Bob does not know Alice's secret string $s$, he cannot compute $T_i \oplus s$ if $r_i=0$, and he cannot compute $T_i$ if $r_i=1$. Therefore, he is computationally blocked from the message he didn't choose.

### Summary of the Flow

1. **Seeds & Base OT:** Alice gets $k$ seeds; Bob keeps all $2k$.
2. **Expansion:** Both parties expand their seeds into large matrices.
3. **Transposition:** They “flip” the matrices to look at rows.
4. **Hashing:** The rows become keys for symmetric encryption (XOR).

This process is effectively “recycling” the expensive public-key entropy from the Base OTs to create millions of secure channels using only fast symmetric-key operations.

#### Efficiency Comparison

| **Feature**            | **Base OT (e.g., DH)**                   | **OT Extension (IKNP)**                               |
| ---------------------- | ---------------------------------------- | ----------------------------------------------------- |
| **Cryptography**       | Public-key (Asymmetric)                  | Symmetric-key (AES/SHA)                               |
| **Typical Throughput** | ~Thousands per second                    | ~Millions per second                                  |
| **Core Operation**     | Modular Exponentiation                   | XOR & PRG expansion                                   |
| **Scaling**            | Large linear cost (in the number of OTs) | Large constant cost (Base OTs) + marginal linear cost |

#### Modern Varieties: cOT and OLE

**Correlated OT (cOT)** is a further optimization where the two messages $m_0, m_1$ are not independent but have a fixed correlation, like $m_1 = m_0 + \Delta$ for some fixed global constant $\Delta$. This is even faster and is the engine behind protocols like **SPDZ** used for privacy-preserving data analysis.

**Oblivious Linear Evaluation (OLE)** is the arithmetic generalization of OT, where a sender provides a linear function $f(x) = ax + b$ and a receiver provides an input $x$, allowing the receiver to obtain the result $f(x)$ without learning the individual coefficients $a$ or $b$, and without the sender learning the input $x$. It serves as a fundamental building block for performing secure multiplications over large finite fields (arithmetic MPC) in the same way that OT serves as the foundation for Boolean circuits.

***

## Lattice-Based MPC

### The Geometric Foundation: What is a Lattice?

In mathematics, a lattice $\mathcal{L}$ is a discrete subgroup of $\mathbb{R}^n$ generated by a set of linearly independent basis vectors.

- Imagine a crystal structure in physics. A 2D lattice is a grid of points formed by integer combinations of two vectors.
- **The Hard Problem:** In low dimensions, it's easy to find the point in the lattice closest to a random target. In high dimensions (e.g., $n=512$ or $1024$), finding the **Shortest Vector** or the **Closest Vector** is computationally "hard"—even for a quantum computer.

### The Engine: Learning With Errors (LWE)

Most lattice-based MPC protocols are built on the **Learning With Errors (LWE)** problem.

#### The "Noisy" Equation

If I give you a system of linear equations like $Ax = b$, you can solve for $x$ easily using Gaussian elimination. However, LWE adds a small amount of "noise" or "error" ($e$) to each equation: $b = Ax + e \pmod q$.

- **Security:** Without knowing the specific noise values, finding $x$ becomes practically impossible.
- **Physics Analogy:** It’s like trying to reconstruct a signal from a very low Signal-to-Noise Ratio (SNR) observation where the noise follows a specific Gaussian distribution.

### The MPC Strategy: Threshold FHE

Lattice-based MPC typically utilizes **Fully Homomorphic Encryption (FHE)** in a "Threshold" setting.

#### Step 1: Distributed Key Generation

Instead of one person owning the secret key, $n$ parties jointly generate a **Public Key ($PK$)**.

- The **Secret Key ($SK$)** is never fully reconstructed; instead, each party $P_i$ receives a "share" $SK_i$ using a technique like Shamir’s Secret Sharing.

#### Step 2: Encrypted Evaluation

Parties encrypt their private inputs $x_i$ using the joint $PK$. Because lattice-based schemes are naturally homomorphic:

- **Addition:** Adding two ciphertexts results in the sum of the plaintexts.
- **Multiplication:** Multiplying ciphertexts (with some extra "relinearization" steps) results in the product.

#### Step 3: Distributed Decryption

Once the function $f(x_1, \dots, x_n)$ is computed homomorphically, the result is still a ciphertext. To see the answer, a quorum of parties must each provide a "partial decryption" using their $SK_i$. These partial results are combined to reveal the final output without ever exposing individual inputs or the full secret key.

### Efficiency and the NTT

One major bottleneck in lattice schemes is multiplying large polynomials. To solve this, protocols use the **Number-Theoretic Transform (NTT)**.

- **The Math:** Just as the **Fast Fourier Transform (FFT)** turns a complex convolution in the time domain into a simple point-wise multiplication in the frequency domain, the NTT does the same for polynomials over finite fields.
- **Performance:** This reduces the complexity from $O(n^2)$ to $O(n \log n)$, making lattice-based MPC fast enough for real-time applications.

### Why Use Lattices?

1. **Post-Quantum Security:** Unlike RSA or Elliptic Curves, there are no known quantum algorithms (like Shor's) that can efficiently solve lattice problems.
2. **Dishonest Majority:** Lattice-based MPC is the gold standard for protocols where you cannot trust a majority of the participants.
3. **Low Communication:** Once inputs are encrypted, the "server" can do the heavy lifting, meaning the parties don't need to send millions of messages back and forth during the calculation.

***

