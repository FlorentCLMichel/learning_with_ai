# Data Structures and Algorithms: A Technical Primer

This document provides a brief overview of fundamental data structures and algorithms, including their properties, use cases, and trade-offs.

------

## Understanding Complexity

### Mathematical Definitions of Asymptotic Notations

#### Big-O Notation (O)
The Big-O notation describes the **upper bound** of the growth rate of a function. Formally, given a real-valued functions $f$ and a real-valued non-negative function $g$ of one real (or integer) variable, we say $f(x) \mathop{=}_{x \to \infty} O(g(x))$ if
$$
\exists c > 0 \quad \exists x_0 \in \mathbb{R} \quad \forall x \geq x_0 \quad \left\lvert f(x) \right\rvert \leq c \, g(x) .
$$
This notation is widely used in computer science to analyze the worst-case complexity of algorithms. Typically, $f$ will be a non-negative function describing the runtime, memory use, or power draw of running an algorithm.

For example, if $f(n) = 3n^2 + 2n + 1$, then $f(n) \mathop{=}_{n \to \infty} O(n^2)$. Notice that we also have $f(n) \mathop{=}_{n \to \infty} O(n^3)$ and $f(n) \mathop{=}_{n \to \infty} O(2^n)$—the ‘Big-O’ notation does not need to be tight. (However, $f(n) \mathop{=}_{n \to \infty} O(n^1)$ is incorrect.)

#### Little-o Notation (o)
The little-o notation describes a **strictly tighter upper bound** that is not asymptotically tight. Formally, with the same notations as above, $f(x) \mathop{=}_{x \to \infty} o(g(x))$ if
$$
\forall c > 0 \quad \exists x_0 \in \mathbb{R} \quad \forall x \geq x_0 \quad \left\lvert f(x) \right\rvert \leq c \, g(x) .
$$
Notice that the only (but important) difference is the quantifier of $c$.

For example, if $f(n) = 3n^2 + 2n + 1$, then $f(n) \mathop{=}_{n \to \infty} o(n^3)$. Notice that we also have $f(n) \mathop{=}_{n \to \infty} o(n^4)$ and $f(n) \mathop{=}_{n \to \infty} o(2^n)$—the ‘Little-o’ notation, like the ‘Big-O’ one, does not need to be tight. However, $f(n) \mathop{=}_{n \to \infty} o(n^2)$ is incorrect.

While less commonly used in practice, it is useful for more precise theoretical analysis.

#### Theta Notation (Θ)
The Theta notation provides a **tight bound** for the growth rate of a function. Formally, assuming that $f$ is non-negative, $f(x)  \mathop{=}_{x \to \infty} \Theta(g(x))$ if
$$
\exists c_1 > 0 \quad \exists c_2 > 0 \quad \exists x_0 \in \mathbb{R} \quad \forall x \geq x_0 \quad c_1 \, g(x) \leq f(x) \leq c_2 \, g(x) .
$$

For example, if $f(n) = 3n^2 + 2n + 1$, then $f(n) \mathop{=}_{n \to \infty} \Theta(n^2)$. Saying that $f(n) \mathop{=}_{n \to \infty} \Theta(n^3)$ or $f(n) \mathop{=}_{n \to \infty} \Theta(n)$ is incorrect. In this sense, the ‘Theta’ notation is tighter than ‘Big-O’.

This notation is useful for describing both the upper and lower bounds of an algorithm's complexity.

#### Notes on notations

When there is no reasonable ground for confusion, we may omit the index $x \to \infty$.

It is common to replace the functions $f$ and/or $g$ by their expressions in terms if their variable, usually denoted by $x$ if it takes continuous values or $n$ if it takes discrete values. 

#### Logarithmic complexity

An algorithm has **logarithmic complexity** if its time complexity is $O(\log n)$. 

Logarithmic-time algorithms are generally considered very efficient and scalable for large inputs.

#### Polynomial Complexity

An algorithm has **polynomial complexity** if its time complexity is $O(n^k)$ for some constant, positive $k$. Examples include:
- Linear time: $O(n)$
- Quadratic time: $O(n^2)$
- Cubic time: $O(n^3)$

Polynomial-time algorithms are generally considered relatively efficient and scalable for large inputs. However, how efficient they are in practice critically depends on the exponent.

#### Exponential Complexity

An algorithm has **exponential complexity** if its time complexity is $O(2^{n})$ or $O(k^n)$ for some constant $k > 1$. These algorithms quickly become impractical for large inputs due to their rapid growth.

### Time Complexity

- **Description:** Time complexity measures the amount of time an algorithm takes to complete as a function of the size of the input. It is expressed using Big-O notation, which describes the upper bound of the growth rate.
- **Common Notations:**
  - **$O(1)$:** Constant time. The algorithm takes the same amount of time regardless of input size.
  - **$O(\log n)$:** Logarithmic time. The time increases logarithmically with the input size.
  - **$O(n)$:** Linear time. The time increases linearly with the input size.
  - **$O(n \log n)$:** Linearithmic time. Common in efficient sorting algorithms like Merge Sort.
  - **$O(n^2)$:** Quadratic time. The time increases quadratically with the input size, often seen in nested loops.
  - **$O(2^n)$:** Exponential time. The time doubles with each additional input element, typical in recursive algorithms without optimization.

### Space Complexity

- **Description:** Space complexity measures the amount of memory an algorithm uses as a function of the input size. It is also expressed using Big-O notation.
- **Common Notations:** Similar to the above, replacing ‘time’ with ‘space’.

### Why Complexity Matters

- **Efficiency:** Understanding complexity helps in choosing the right data structure or algorithm for a given problem, ensuring optimal performance.
- **Scalability:** Algorithms with lower time complexity scale better with large input sizes, making them suitable for big data applications.
- **Resource Management:** Space complexity is crucial for memory-constrained environments, such as embedded systems or mobile devices.

Sometimes, optimizing for time complexity (e.g., using a hash table) may increase space complexity, and vice versa. Understanding these trade-offs is crucial for algorithm design.

------

## Common Data Structures

### Hash Table (Hash Map)

- **Description:** A data structure that implements an associative array abstract data type, mapping keys to values using a **hash function** to compute an index into an array of buckets or slots.
- **Main Properties:**
  - **Time Complexity:** $O(1)$ average case for Search, Insert, and Delete. $O(n)$ worst case (when many keys hash to the same index).
  - **Space Complexity:** $O(n)$.
  - **Collisions:** Handled via **Chaining** (linked lists in each bucket) or **Open Addressing** (finding the next empty slot).
- **Use-cases:** Database indexing, caching (LRU caches), unique element counting, and implementing Sets.
- **Example:**
  ```python
  cache = {}
  def get(key):
      return cache.get(key, None)
  def put(key, value):
      cache[key] = value
  ```

### Trie (Prefix Tree)

- **Description:** A tree-like data structure used for storing strings, where each node represents a character. It is efficient for prefix-based searches.
- **Main Properties:**
  - **Time Complexity:**
    - Insertion: $O(L)$, where $L$ is the length of the word.
    - Search: $O(L)$.
    - Prefix Search: $O(L)$.
- **Use-cases:** Autocomplete systems, spell checkers, and IP routing tables.
- **Example:**
  ```python
  trie = {}
  def insert(word):
      node = trie
      for char in word:
          if char not in node:
              node[char] = {}
          node = node[char]
      node["*"] = True
  ```

### Disjoint Set Union (DSU) or Union-Find

- **Description:** A data structure that tracks a partition of a set into disjoint subsets. It supports two operations: `find` (determine which subset an element is in) and `union` (merge two subsets).
- **Main Properties:**
  - **Time Complexity:** With path compression and union by rank: $O(\alpha(n))$, where $\alpha$ is the inverse Ackermann function (effectively constant time).
- **Use-cases:** Kruskal's algorithm for Minimum Spanning Tree (MST), network connectivity, and cycle detection in undirected graphs.

### Bloom Filter

- **Description:** A probabilistic data structure used to test whether an element is a member of a set. It may return false positives but never false negatives.
- **Main Properties:**
  - **Space Complexity:** $O(m)$, where $m$ is the size of the bit array.
- **Use-cases:** Spell checkers, caching systems, and network routers.

### Segment Tree

- **Description:** A binary tree used for storing intervals or segments. It allows for efficient range queries and updates.
- **Main Properties:**
  - **Time Complexity:**
    - Construction: $O(n)$.
    - Query: $O(\log n)$.
    - Update: $O(\log n)$.
- **Use-cases:** Range minimum/maximum queries, range sum queries, and interval scheduling.

### Binary Search Tree (BST) & Balanced Trees (AVL, Red-Black)

- **Description:** A node-based tree structure where each node has at most two children. For any node, the left subtree contains values less than the node, and the right subtree contains values greater.
- **Main Properties:**
  - **Time Complexity:** In a balanced tree (like an **AVL** or **Red-Black Tree**), Search, Insert, and Delete are $O(\log n)$.
  - **Traversal:** In-order traversal yields elements in sorted order.
- **Use-cases:** Implementing ordered sets and maps, file systems (B-Trees), and priority queues.

### Priority Queue

- **Description:** An abstract data type similar to a regular queue or stack, but where each element has a "priority" associated with it. In a priority queue, an element with high priority is served before an element with low priority. It is most commonly implemented using a **Heap** for efficiency.
- **Main Properties:**
  - **Insert (Push):** $O(\log n)$ — The element is added and the heap property is restored.
  - **Extract Max/Min (Pop):** $O(\log n)$ — The root is removed and the heap is "re-balanced."
  - **Peek:** $O(1)$ — Returns the highest priority element without removing it.
  - **Ordering:** Unlike a standard FIFO (First-In-First-Out) queue, the internal order depends entirely on the priority logic (e.g., a Min-Priority Queue vs. a Max-Priority Queue).
- **Use-cases:**
  - **Dijkstra's Algorithm:** Used to constantly pick the next node with the shortest tentative distance.
  - **Operating System Scheduling:** Managing processes where some tasks (like UI interrupts) have higher priority than background tasks.
  - **Bandwidth Management:** Prioritizing real-time data packets (like VoIP or video) over standard web traffic in routers.
  - **Huffman Coding:** Used in data compression algorithms to build the optimal prefix tree.

### Heaps (Binary Heap)

- **Description:** A complete binary tree that satisfies the **heap property**: in a Max-Heap, the parent is always greater than or equal to its children; in a Min-Heap, it is less than or equal.
- **Main Properties:**
  - **Access Max/Min:** $O(1)$.
  - **Insert/Delete (Extract):** $O(\log n)$.
  - **Heapify:** Building a heap from an unordered array takes $O(n)$.
- **Use-cases:** Priority queues, scheduling algorithms (like K8s pod scheduling), and finding the "top K" elements in a stream.

### Graphs (Adjacency List vs. Matrix)

- **Description:** A collection of vertices (nodes) connected by edges.
- **Main Properties:**
  - **Adjacency List:** $O(V + E)$ space. Better for sparse graphs. Fast to iterate over neighbors.
  - **Adjacency Matrix:** $O(V^2)$ space. Better for dense graphs. $O(1)$ to check if an edge exists between two specific nodes.
- **Use-cases:** Social networks, recommendation engines, pathfinding in maps, and dependency resolution in build systems.
- **Example:**
  ```python
  graph = {
      "Alice": ["Bob", "Charlie"],
      "Bob": ["Alice", "David"],
      "Charlie": ["Alice"],
      "David": ["Bob"]
  }
  ```

### Quick Reference: Time Complexity Table

| **Data Structure**        | **Access**  | **Search**  | **Insertion** | **Deletion** | **Notes**                          |
| ------------------------- | ----------- | ----------- | ------------- | ------------ | ---------------------------------- |
| **Array**                 | $O(1)$      | $O(n)$      | $O(n)$        | $O(n)$       | Fixed size, contiguous memory.     |
| **Hash Table**            | N/A         | $O(1)$      | $O(1)$        | $O(1)$       | Average case; worst case $O(n)$. |
| **Priority Queue (Heap)** | $O(1)$      | $O(n)$      | $O(\log n)$   | $O(\log n)$  | Min/Max access in $O(1)$.        |
| **BST (Balanced)**        | $O(\log n)$ | $O(\log n)$ | $O(\log n)$   | $O(\log n)$  | Requires balancing (AVL, Red-Black). |
| **Singly Linked List**    | $O(n)$      | $O(n)$      | $O(1)$        | $O(1)$       | Dynamic size, non-contiguous.      |
| **Trie**                  | N/A         | $O(L)$      | $O(L)$        | $O(L)$       | $L$ is the length of the word.   |
| **Bloom Filter**          | N/A         | $O(1)$      | $O(1)$        | N/A          | Probabilistic, false positives.    |

------

## Common Algorithms

### Dijkstra's Algorithm

- **Description:** A greedy algorithm for finding the shortest paths from a single source node to all other nodes in a graph with non-negative edge weights.
- **Use-cases:** GPS navigation systems, network routing protocols, and finding the shortest path in a maze.
- **Time Complexity:** $O((V + E) \log V)$, where $V$ is the number of vertices and $E$ is the number of edges.
- **Example:**
  ```python
  import heapq
  def dijkstra(graph, start):
      distances = {node: float('infinity') for node in graph}
      distances[start] = 0
      priority_queue = [(0, start)]
      while priority_queue:
          current_distance, current_node = heapq.heappop(priority_queue)
          for neighbor, weight in graph[current_node].items():
              distance = current_distance + weight
              if distance < distances[neighbor]:
                  distances[neighbor] = distance
                  heapq.heappush(priority_queue, (distance, neighbor))
      return distances
  ```

### Kruskal's Algorithm

- **Description:** A greedy algorithm for finding the Minimum Spanning Tree (MST) of a graph. It uses the Union-Find data structure to efficiently manage and merge sets of nodes.
- **Use-cases:** Network design, clustering, and circuit design.
- **Time Complexity:** $O(E \log E)$ or $O(E \log V)$, where $E$ is the number of edges and $V$ is the number of vertices.

### Floyd-Warshall Algorithm

- **Description:** An algorithm for finding the shortest paths between all pairs of vertices in a weighted graph. It works for both directed and undirected graphs and can handle negative weights (but not negative cycles).
- **Use-cases:** Route planning, social network analysis, and detecting arbitrage in currency markets.
- **Time Complexity:** $O(V^3)$, where $V$ is the number of vertices.

### Topological Sorting (Kahn's Algorithm or DFS-based)

- **Description:** An algorithm for linearly ordering the vertices of a directed acyclic graph (DAG) such that for every directed edge $u \rightarrow v$, vertex $u$ comes before $v$ in the ordering.
- **Use-cases:** Task scheduling, dependency resolution, and build systems.
- **Time Complexity:** $O(V + E)$, where $V$ is the number of vertices and $E$ is the number of edges.

### Knapsack Problem (Dynamic Programming)

- **Description:** A classic optimization problem where the goal is to maximize the value of items in a knapsack without exceeding its weight capacity. It is solved using dynamic programming.
- **Use-cases:** Resource allocation, budget optimization, and financial portfolio selection.
- **Time Complexity:** $O(nW)$, where $n$ is the number of items and $W$ is the maximum weight capacity.

------

## Trade-offs and Practical Tips

### Time vs. Space Complexity

- **Hash Tables:** Offer $O(1)$ access but use more memory.
- **Graphs:** Adjacency lists are space-efficient for sparse graphs, while adjacency matrices are faster for edge lookups in dense graphs.

### Choosing the Right Data Structure

- Use a **hash table** for fast lookups and insertions.
- Use a **BST** for ordered data and efficient search, insertion, and deletion.
- Use a **priority queue** for tasks that require constant access to the highest or lowest priority element.

### Practical Tips

- **Hash Tables:** Use a good hash function to minimize collisions.
- **Graphs:** Use adjacency lists for sparse graphs and adjacency matrices for dense graphs.
- **Sorting:** Use quicksort for average-case performance and mergesort for stable sorting.

------

## Further Reading

- **Books:**
  - [*Introduction to Algorithms*](https://mitpress.mit.edu/9780262533058/introduction-to-algorithms/) by Thomas H. Cormen, Charles E. Leiserson, Ronald L. Rivest and Clifford Stein.
  - [*Algorithms*](https://algs4.cs.princeton.edu/home/) by Robert Sedgewick and Kevin Wayne.
- **Online Resources:**
  - [GeeksforGeeks](https://www.geeksforgeeks.org)
  - [LeetCode](https://leetcode.com)

------

## Visualizations

### Binary Tree Example

```
      A
     / \
    B   C
   / \   \
  D   E   F
```

This is a simple binary tree with root node `A`.

### Graph Example

```
  Alice -- Bob
   |      /  \
  Charlie   David
```

This is a simple undirected graph representing connections between people.

------

## Conclusion

This primer covers the fundamental data structures and algorithms that are essential for efficient problem-solving in computer science. Understanding these concepts will help you design scalable and optimized solutions for a wide range of problems. For further exploration, refer to the recommended books and online resources.
