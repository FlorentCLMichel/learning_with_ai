# Multi-Party Computation (MPC) — A technical primer

## Introduction and Formal Definition

Multi-Party Computation (MPC) is a subfield of cryptography that allows a set of n parties to jointly compute a function $f(x_1, \dots, x_n)$ over their private inputs $x_1, \dots, x_n$. The fundamental goal is to provide the output of the function while ensuring that no party learns anything about the other participants' inputs beyond what is revealed by the output itself.

Formally, an MPC protocol satisfies two primary constraints:

* **Privacy:** For any party $P_i$, the information gained during the protocol (the "View") is effectively the same regardless of the other parties' specific private inputs, provided the output remains constant.
* **Correctness:** If the parties follow the protocol, the final output is guaranteed to be $f(x_1, \dots, x_n)$.

---

## Adversary Models and Security Guarantees

The security of an MPC protocol is defined by how many corrupt parties it can withstand and the behavior of those parties.

### Adversary Behavior

* **Semi-Honest (Passive):** Parties follow the protocol but attempt to learn private information from the intermediate data they see.
* **Malicious (Active):** Parties may deviate from the protocol, provide false inputs, or quit early to prevent others from getting the output.

### Threshold Structures

Protocols are generally categorized by the "threshold" t of corrupted parties they can tolerate:

* **Honest Majority:** Requires $t < n/2$. These often rely on **Information-Theoretic Security**, meaning they are secure even against an adversary with infinite computing power.
* **Dishonest Majority:** Secure even if $n-1$ parties are corrupt. These typically rely on **Computational Hardness** (e.g., the Learning With Errors (LWE) problem or Decisional Diffie-Hellman (DDH)).

---

## Fundamental Building Blocks

### Shamir’s Secret Sharing (SSS)

Used for $n$-party computation in honest-majority settings. It relies on the mathematical principle that $t+1$ points are required to uniquely define a polynomial of degree $t$.

To share a secret $s \in \mathbb{F}$:

1. Pick a random polynomial $q(x)$ of degree $t$ such that $q(0) = s$.


2. Give each party $P_i$ the share $y_i = q(i)$.
3. Any $t+1$ parties can use **Lagrange Interpolation** to reconstruct $q(0)$.

### Oblivious Transfer (OT)

The "atomic unit" of MPC. In a 1-out-of-2 OT, a sender has two messages $(m_0, m_1)$, and a receiver has a bit $b \in \{0, 1\}$. The receiver gets $m_b$ without the sender knowing which they chose, and without the receiver learning the other message. Modern protocols use **OT Extension** to generate millions of OTs using only a few expensive public-key operations.

### Garbled Circuits (Yao’s Protocol)

Primarily used for 2-party computation (2PC).

1. The function $f$ is represented as a Boolean circuit of AND and XOR gates.
2. The "Garbler" encrypts the truth table of each gate using random cryptographic keys (labels) for each wire.
3. The "Evaluator" receives the garbled tables and, through OT, the labels corresponding to their input.
4. The Evaluator decrypts the circuit gate-by-gate to find the final output.

---

## Arithmetic MPC and Multiplication

While addition is "free" in additive secret sharing (parties just add their local shares), multiplication is complex.

### Beaver Triples (Multiplication Gates)

To multiply two shared values $[a]$ and $[b]$, parties use precomputed "Beaver Triples" $([u], [v], [w])$ where $w = uv$.

1. Parties compute $[d] = [a] - [u]$ and $[e] = [b] - [v]$.

2. Parties "open" (reveal) $d$ and $e$.

3. The share of the product [ab] is computed locally as:
   $$
   [ab] = [w] + e[u] + d[v] + de
   $$

---

## Modern Frontiers and Optimization

| Technique | Description |
| --- | --- |
| **SPDZ** | A protocol for dishonest majority using a "preprocessing" phase to generate authenticated Beaver triples. |
| **FHE Integration** | Using Fully Homomorphic Encryption to offload heavy computation in MPC, reducing communication rounds. |
| **UC Security** | "Universally Composable" security, ensuring a protocol remains secure even when run alongside other instances of itself. |
| **MPC-in-the-Head** | A technique to build Zero-Knowledge Proofs (ZKPs) by simulating an MPC protocol internally. |

---

## Shamir Secret Sharing and Lagrange Interpolation

In MPC, we often use **Shamir’s Secret Sharing** to distribute a secret $S$ among $n$ parties such that any $t+1$ parties can reconstruct it, but $t$ or fewer parties learn absolutely nothing.

### The Setup: Encoding the Secret

To share a secret $S \in \mathbb{F}$ (where $\mathbb{F}$ is a finite field):

1. **Define the Degree:** We choose a threshold $t$. We want any $t+1$ people to be able to recover the secret.

2. Generate the Polynomial: We construct a random polynomial $q(x)$ of degree $t$:

   $$q(x) = a_0 + a_1x + a_2x^2 + \dots + a_tx^t$$

3. **Embed the Secret:** We set the constant term $a_0 = S$. All other coefficients $(a_1, \dots, a_t)$ are chosen uniformly at random from the field.

4. **Distribute Shares:** Each party $P_i$ is assigned a unique, public evaluation point $x_i$ (usually $x_i = i$). Their private share is the value $y_i = q(x_i)$.

### The Reconstruction: Lagrange Interpolation

The core "magic" is that a polynomial of degree $t$ is uniquely determined by $t+1$ distinct points. If a subset of $k = t+1$ parties pool their shares $(x_0, y_0), (x_1, y_1), \dots, (x_k, y_k)$, they can find $q(x)$ using the **Lagrange basis polynomials**.

#### The Basis Polynomials

For each share $j$ in our subset, we define a basis polynomial $\ell_j(x)$ that is equal to $1$ at $x_j$ and $0$ at all other $x_m$ in our subset:

$$\ell_j(x) = \prod_{\substack{0 \le m \le k \\ m \neq j}} \frac{x - x_m}{x_j - x_m}$$

#### Finding the Secret

The full polynomial is the linear combination of these bases:

$$q(x) = \sum_{j=0}^{k} y_j \ell_j(x)$$

Since we only care about the secret $S$, which is $q(0)$, we don't even need to reconstruct the full polynomial. We just evaluate the basis polynomials at zero:

$$S = q(0) = \sum_{j=0}^{k} y_j \ell_j(0)$$

### Why This Works for Privacy

From a physics or information theory perspective, this is essentially about **entropy**.

- If you have $t+1$ points, the system of linear equations is determined; there is exactly one solution for $a_0$.
- If you only have $t$ points, for *every possible* secret $S' \in \mathbb{F}$, there exists exactly one polynomial of degree $t$ that passes through your $t$ points and has $q(0) = S'$.

Consequently, to an adversary with only $t$ shares, all possible secrets are equally likely. The "View" of the adversary provides zero information about the secret.

### Example: $t=1$ (2-out-of-$n$ sharing)

Imagine we want to share a secret $S = 5$ with a threshold $t=1$ (we need 2 people to recover it).

1. **Polynomial:** We pick a random $a_1 = 3$. Our polynomial is $q(x) = 5 + 3x$.

2. **Shares:** * Party 1 gets $(1, 8)$

   - Party 2 gets $(2, 11)$

3. Reconstruction: Using these two points to find the intercept:

   $$\ell_1(0) = \frac{0 - 2}{1 - 2} = 2$$

   $$\ell_2(0) = \frac{0 - 1}{2 - 1} = -1$$

   $$S = (8 \cdot 2) + (11 \cdot -1) = 16 - 11 = 5$$

---

## Operations on shares

In MPC, the ability to perform calculations on encrypted or shared data is what makes the technology powerful. Since you have a strong background in mathematics, you’ll find that the "linearity" of Shamir’s Secret Sharing (SSS) makes addition trivial, while multiplication requires a clever change in perspective.

### Addition: The "Free" Operation

Addition in Shamir’s Secret Sharing is **locally computable**. This means parties can add their shares together without sending any messages to each other.

Suppose we have two secrets, $A$ and $B$, shared using polynomials $q_A(x)$ and $q_B(x)$ of degree $t$.

- Party $P_i$ holds shares $y_{A,i} = q_A(i)$ and $y_{B,i} = q_B(i)$.

- To compute the shares of the sum $S = A + B$, the parties simply add their local shares:

  $$y_{S,i} = y_{A,i} + y_{B,i}$$

Why this works:

Because of the linearity of polynomials, the sum of two polynomials of degree $t$ is itself a polynomial of degree $t$:

$$Q(x) = q_A(x) + q_B(x)$$

The constant term of $Q(x)$ is $q_A(0) + q_B(0)$, which is exactly $A + B$. Since the degree remains $t$, the privacy threshold is preserved.

### Multiplication: The Challenge

Multiplication is significantly harder for two main reasons:

1. **Degree Doubling:** If you multiply two polynomials of degree $t$, the resulting polynomial has degree $2t$. To reconstruct a degree $2t$ polynomial, you would need $2t+1$ parties, which might exceed your available honest majority.
2. **Randomness Loss:** The product of two random polynomials is no longer a "random" polynomial (its coefficients are correlated), which can leak information.

#### The Solution: Beaver Triples

To solve this, modern MPC protocols (like **SPDZ**) use a technique called **Beaver Triples**, which shifts the heavy lifting to a "preprocessing" phase.

##### The Preprocessing Phase

Before the parties know their actual inputs $A$ and $B$, they (or a trusted setup) generate a set of random "triples" $([u], [v], [w])$ such that $w = u \cdot v$. These values are secret-shared among the parties and have no relation to the actual data.

##### The Online Phase

When it is time to multiply the real shares $[a]$ and $[b]$:

1. **Mask the inputs:** Parties compute and "open" (reconstruct) two values:

   - $d = a - u$

   - $e = b - v$

     Because $u$ and $v$ are random and secret, $d$ and $e$ reveal nothing about $a$ or $b$.

2. Compute the product share: Every party can now compute their share of the product $[ab]$ using only the public constants $d, e$ and their shares of the triple:

   

   $$[ab] = [w] + e[u] + d[v] + de$$

The Algebra behind it:

Mathematically, we are just expanding the identity:

$$ab = (d+u)(e+v) = de + dv + ue + uv$$

Since $uv = w$, this simplifies to the formula above. Every term is either a public constant ($de$) or a constant multiplied by a share ($e[u]$ and $d[v]$), both of which are locally computable.

### Comparison of Complexity

| **Operation**      | **Communication Cost**           | **Mathematical Tool**          |
| ------------------ | -------------------------------- | ------------------------------ |
| **Addition**       | Zero (Local)                     | Linearity of Polynomials       |
| **Multiplication** | High (Requires "Opening" $d, e$) | Beaver Triples / Preprocessing |

In a way, this is similar to **Perturbation Theory** in physics: we take a difficult non-linear problem (multiplication) and use a known reference state (the Beaver Triple) to linearize the calculation.
