This technical primer explores the **Transformer** architecture, which shifted the deep learning paradigm from sequential processing (RNNs) to parallelized attention mechanisms. Given your background in math and tech, we will focus on the linear algebra and vector space operations that make this architecture efficient.

------

## 1. High-Level Architecture

The Transformer is traditionally an **Encoder-Decoder** model. Unlike LSTMs or RNNs, it has no recurrence; instead, it processes the entire sequence simultaneously.

- **Encoder:** Maps an input sequence of symbol representations $(x_1, ..., x_n)$ to a sequence of continuous representations $z = (z_1, ..., z_n)$.
- **Decoder:** Generates an output sequence $(y_1, ..., y_m)$ of symbols one element at a time, attending to both the encoder output and its own previous outputs.

------

## 2. Input Representation: Embeddings & Position

Since Transformers lack an inherent sense of "order," they use **Positional Encodings** to inject sequence information into the input embeddings.

### Sinusoidal Encoding

The original paper uses sine and cosine functions of different frequencies:

$$
    PE_{(pos, 2i)} = \sin(pos / 10000^{2i/d_{model}})
$$

$$
    PE_{(pos, 2i+1)} = \cos(pos / 10000^{2i/d_{model}})
$$


This allows the model to learn to attend by relative positions since for any fixed offset $k$, $PE_{pos+k}$ can be represented as a linear function of $PE_{pos}$.

------

## 3. The Core: Scaled Dot-Product Attention

Attention allows the model to dynamically "weigh" the importance of different tokens in a sequence relative to a target token.

### The QKV Mechanism

Each input vector is projected into three distinct spaces using learned weight matrices $W^Q, W^K, W^V$:

1. **Query ($Q$):** What I am looking for.
2. **Key ($K$):** What I contain (to be matched against queries).
3. **Value ($V$):** The information I actually provide.

The attention output is calculated as:

$$
    \text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

- **Dot Product ($QK^T$):** Computes similarity scores between queries and keys.
- **Scaling ($1/\sqrt{d_k}$):** Prevents the dot product from growing too large in high dimensions, which would push the softmax into regions with extremely small gradients.
- **Softmax:** Normalizes scores into a probability distribution.

### Multi-Head Attention (MHA)

Rather than a single attention pass, MHA performs the operation $h$ times in parallel. This allows the model to attend to different types of information (e.g., one head for syntax, another for semantic reference) simultaneously.

------

## 4. The Transformer Block

Each layer in the stack consists of two primary sub-layers:

1. **Multi-Head Self-Attention:** Where tokens communicate.
2. **Feed-Forward Network (FFN):** A simple MLP applied to each position identically and independently. This is typically two linear transformations with a ReLU or GeLU activation.

Residual Connections & Layer Norm: Each sub-layer is wrapped in a residual connection followed by layer normalization:

$$
    \text{LayerOutput} = \text{LayerNorm}(x + \text{Sublayer}(x))
$$

------

## 5. Modern Evolutions (2025-2026)

While the 2017 "Attention is All You Need" architecture remains the foundation, several optimizations have become standard:

- **Rotary Positional Embeddings (RoPE):** Instead of adding absolute positions, RoPE rotates the $Q$ and $K$ vectors in complex space, naturally capturing relative distance.
- **Hyper-Connections:** Recent 2026 research (like DeepSeek's mHC) has begun replacing fixed residual connections with learnable, "double-stochastic" weightings to stabilize gradients in ultra-deep networks.
- **FlashAttention:** An IO-aware algorithm that speeds up the attention calculation by minimizing memory reads/writes.

------

