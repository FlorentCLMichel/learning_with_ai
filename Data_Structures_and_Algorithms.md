# Data Structures and Algorithms: A Technical Primer

This document provides a brief overview of fundamental data structures and algorithms, including their properties, use cases, and trade-offs.

------

## Understanding Complexity

### Mathematical Definitions of Asymptotic Notations

#### Big-O Notation (O)
The Big-O notation describes an **upper bound** of the growth rate of a function. Formally, given a real-valued function $f$ and a real-valued non-negative function $g$ of one real (or integer) variable, we say $f(x) \mathop{=}_{x \to \infty} O(g(x))$ if
$$
\exists c > 0 \quad \exists x_0 \in \mathbb{R} \quad \forall x \geq x_0 \quad \left\lvert f(x) \right\rvert \leq c \, g(x) .
$$
This notation is widely used in computer science to analyze the worst-case complexity of algorithms. Typically, $f$ will be a non-negative function describing the runtime, memory use, or energy cost of running an algorithm. Intuitively, the notation $f(x) = O(g(x))$ may then be interpreted as ‘$f(x)$ does not grow faster than $g(x)$ (up to a multiplicative constant) as $x$ tends to infinity’.

For example, if $f(n) = 3n^2 + 2n + 1$, then $f(n) \mathop{=}_{n \to \infty} O(n^2)$. Notice that we also have $f(n) \mathop{=}_{n \to \infty} O(n^3)$ and $f(n) \mathop{=}_{n \to \infty} O(2^n)$—the ‘Big-O’ notation does not need to be tight. (However, $f(n) \mathop{=}_{n \to \infty} O(n^1)$ is incorrect.)

#### Little-o Notation (o)
The little-o notation describes a **strictly tighter upper bound** that is not asymptotically tight. Formally, with the same notations as above, $f(x) \mathop{=}_{x \to \infty} o(g(x))$ if
$$
\forall c > 0 \quad \exists x_0 \in \mathbb{R} \quad \forall x \geq x_0 \quad \left\lvert f(x) \right\rvert \leq c \, g(x) .
$$
Notice that the only (but important) difference is the quantifier of $c$.

For example, if $f(n) = 3n^2 + 2n + 1$, then $f(n) \mathop{=}_{n \to \infty} o(n^3)$. Notice that we also have $f(n) \mathop{=}_{n \to \infty} o(n^4)$ and $f(n) \mathop{=}_{n \to \infty} o(2^n)$—the ‘Little-o’ notation, like the ‘Big-O’ one, does not need to be tight. However, $f(n) \mathop{=}_{n \to \infty} o(n^2)$ is incorrect. Intuitively, the notation $f(x) = o(g(x))$ may often be interpreted as ‘$f(x)$ grows strictly more slowly than $g(x)$ as $x$ tends to infinity’.

While less commonly used in practice, it is useful for more precise theoretical analysis.

#### Theta Notation (Θ)
The Theta notation provides a **tight bound** for the growth rate of a function. Formally, assuming that $f$ is non-negative, $f(x)  \mathop{=}_{x \to \infty} \Theta(g(x))$ if
$$
\exists c_1 > 0 \quad \exists c_2 > 0 \quad \exists x_0 \in \mathbb{R} \quad \forall x \geq x_0 \quad c_1 \, g(x) \leq f(x) \leq c_2 \, g(x) .
$$

For example, if $f(n) = 3n^2 + 2n + 1$, then $f(n) \mathop{=}_{n \to \infty} \Theta(n^2)$. Saying that $f(n) \mathop{=}_{n \to \infty} \Theta(n^3)$ or $f(n) \mathop{=}_{n \to \infty} \Theta(n)$ is incorrect. In this sense, the ‘Theta’ notation is tighter than ‘Big-O’. Intuitively, the notation $f(x) = \Theta(g(x))$ may often be interpreted as ‘$f(x)$ and $g(x)$ grow with the same rate as $x$ tends to infinity’.

This notation is useful for describing both the upper and lower bounds of an algorithm's complexity. 

#### Notes on Notations

When there is no reasonable ground for confusion, we may omit the subscript $x \to \infty$. (The reason for this subscript is that, in Mathematics, one may want to compare the behavious of two functions in different limits, *e.g.* $x \to -\infty$ or $x \to x_l$ for some finits value $x_l$. For instance, we can write $(x + 1) / x^2 \mathop{=}_{x \to 0} \Theta(1 / x^2)$.)

It is common to replace the functions $f$ and/or $g$ by their expressions in terms of their variable, usually denoted by $x$ if it takes continuous values or $n$ if it takes discrete values.

#### Logarithmic Complexity

An algorithm has **logarithmic complexity** if its time complexity is $\Theta(\log n)$. 

Logarithmic-time algorithms are generally considered very efficient and scalable for large inputs, as multiplying the input size by $2$ only adds (at least asymptotically) a constant to the runtime.

#### Polynomial Complexity

An algorithm has **polynomial complexity** if its time complexity is $\Theta(n^k)$ for some constant, positive $k$. Examples include:
- Linear time: $O(n)$
- Quadratic time: $O(n^2)$
- Cubic time: $O(n^3)$

Polynomial-time algorithms are generally considered relatively efficient and scalable for large inputs. However, how efficient they are in practice critically depends on the exponent and, to a lesser extent, the multiplicative constant not captured by the ‘Big-O’ notation.

#### Exponential Complexity

An algorithm has **exponential complexity** if its time complexity is $\Theta(k^n)$ for some constant $k > 1$. These algorithms quickly become impractical for large inputs due to their rapid growth.

### Types of complexity often used in Computer Science

#### Time Complexity

- **Description:** Time complexity measures the amount of time an algorithm takes to complete as a function of the size of the input. It is expressed using either the Big-O notation, which describes the upper bound of the growth rate, or te tighter Theta notation.
- **Common Notations:**
  - **$O(1)$:** Constant time. The algorithm takes the same amount of time regardless of input size.
  - **$O(\log n)$:** Logarithmic time. The time increases logarithmically with the input size.
  - **$O(n)$:** Linear time. The time increases linearly with the input size.
  - **$O(n \log n)$:** Linearithmic time. Common in efficient sorting algorithms like Merge Sort.
  - **$O(n^2)$:** Quadratic time. The time increases quadratically with the input size, often seen in nested loops.
  - **$O(2^n)$:** Exponential time. The time doubles with each additional input element, typical in recursive algorithms without optimization.

#### Space Complexity

- **Description:** Space complexity measures the amount of memory an algorithm uses as a function of the input size. It is also expressed using Big-O or Theta notations.
- **Common Notations:** Similar to the above, replacing ‘time’ with ‘space’.

### Why Complexity Matters

- **Efficiency:** Understanding complexity helps in choosing the right data structure or algorithm for a given problem, ensuring optimal performance.
- **Scalability:** Algorithms with lower time complexity scale better with large input sizes, making them suitable for big data applications.
- **Resource Management:** Space complexity is crucial for memory-constrained environments, such as embedded systems or mobile devices.

Sometimes, optimizing for time complexity (e.g., using a hash table) may increase space complexity, and vice versa. Understanding these trade-offs is crucial for algorithm design.

------

## Common Data Structures

In this section we present some common structures used to store a number $n$ of elements, Unless stated otherwise, the examples are written in Python.

### Hash Table (Hash Map)

- **Description:** A data structure that implements an associative array abstract data type, mapping keys to values using a **hash function** to compute an index into an array of buckets or slots.
- **Main Properties:**
  - **Time Complexity:** $O(1)$ average case for Search, Insert, and Delete. $O(n)$ worst case (when many keys hash to the same index).
  - **Space Complexity:** $O(n)$.
  - **Collisions:** Handled via **Chaining** (linked lists in each bucket) or **Open Addressing** (finding the next empty slot).
- **Use-cases:** Database indexing, caching (Least Recently Used caches), unique element counting, and implementing Sets.
- **Example:**
  ```python
  # A node class with a key, a value, and pointers to the previous and next nodes.
  class Node:
      def __init__(self, key, value):
          self.key = key
          self.value = value
          self.prev = None
          self.next = None
  
  # We implement an LRU cache using a hash map for O(1) access and a doubly linked list
  # for O(1) eviction.
  class LRUcache:
      def __init__(self, capacity: int):
          self.capacity = capacity
          self.data = {}  # Maps keys to nodes
          self.head = Node(None, None)  # Dummy head
          self.tail = Node(None, None)  # Dummy tail
          self.head.next = self.tail
          self.tail.prev = self.head
  
      # To remove a node, simply link the previous and next nodes.
      def _remove_node(self, node):
          """Remove a node from the linked list."""
          prev_node = node.prev
          next_node = node.next
          prev_node.next = next_node
          next_node.prev = prev_node
  
      # To add a node to the front, put it just after the dummy head.
      def _add_to_front(self, node):
          """Add a node to the front of the linked list."""
          node.prev = self.head
          node.next = self.head.next
          self.head.next.prev = node
          self.head.next = node
  
      def get(self, key: int) -> int:
          """Get the value corresponding to a key; return None if the key is not in the cache."""
          if key in self.data:
              node = self.data[key]
              self._remove_node(node)
              self._add_to_front(node)
              return node.value
          else:
              return None
  
      def put(self, key: int, value: int):
          """Put a value in the cache. If the key already holds a value, it is updated. Otherwise, a
             new (key, value) pair is added. If the number of elements exceeds the capacity, the
             least recently used element is evicted."""
          if key in self.data:
              node = self.data[key]
              node.value = value
              self._remove_node(node)
              self._add_to_front(node)
          else:
              if len(self.data) >= self.capacity:
                  # Evict the least recently used node.
                  lru_node = self.tail.prev
                  self._remove_node(lru_node)
                  del self.data[lru_node.key]
              new_node = Node(key, value)
              self.data[key] = new_node
              self._add_to_front(new_node)
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
  - **Dijkstra's Algorithm:** Used to constantly pick the next node with the smallest tentative distance.
  - **Scheduling:** Prioritizing tasks in operating systems or job schedulers.
  - **Huffman Coding:** Building a Huffman tree for optimal prefix coding.

### Heap

- **Description:** A specialized tree-based data structure that satisfies the heap property. In a **Min-Heap**, for any given node, the value of the node is less than or equal to the values of its children. In a **Max-Heap**, the value of the node is greater than or equal to the values of its children.
- **Main Properties:**
  - **Time Complexity:**
    - Insert: $O(\log n)$.
    - Extract Min/Max: $O(\log n)$.
    - Peek: $O(1)$.
  - **Space Complexity:** $O(n)$.
- **Use-cases:** Priority queues, heap sort, and scheduling algorithms.

### Graph

- **Description:** A collection of nodes (vertices) connected by edges. Graphs can be directed or undirected, weighted or unweighted.
- **Main Properties:**
  - **Representation:** Adjacency matrix, adjacency list, or edge list.
  - **Algorithms:**
    - **Breadth-First Search (BFS):** Explores all neighbors at the present depth before moving on to nodes at the next depth level. Time complexity: $O(V + E)$.
    - **Depth-First Search (DFS):** Explores as far as possible along each branch before backtracking. Time complexity: $O(V + E)$.
    - **Dijkstra's Algorithm:** Finds the shortest path from a single source to all other nodes in a weighted graph. Time complexity: $O(E + V \log V)$ with a priority queue.
    - **Kruskal's Algorithm:** Finds a minimum spanning tree for a connected weighted graph. Time complexity: $O(E \log E)$ or $O(E \log V)$.
- **Use-cases:** Network routing, social network analysis, and dependency resolution.

### Stack

- **Description:** A linear data structure that follows the Last-In-First-Out (LIFO) principle. The last element added to the stack is the first one to be removed.
- **Main Properties:**
  - **Time Complexity:**
    - Push: $O(1)$.
    - Pop: $O(1)$.
    - Peek: $O(1)$.
  - **Space Complexity:** $O(n)$.
- **Use-cases:** Function call management (call stack), undo mechanisms, and expression evaluation.

### Queue

- **Description:** A linear data structure that follows the First-In-First-Out (FIFO) principle. The first element added to the queue is the first one to be removed.
- **Main Properties:**
  - **Time Complexity:**
    - Enqueue: $O(1)$.
    - Dequeue: $O(1)$.
    - Peek: $O(1)$.
  - **Space Complexity:** $O(n)$.
- **Use-cases:** Task scheduling, breadth-first search (BFS), and buffering.

### Linked List

- **Description:** A linear data structure where each element is a separate object (node) that contains a reference (or link) to the next node in the sequence.
- **Main Properties:**
  - **Time Complexity:**
    - Insertion/Deletion at head: $O(1)$.
    - Insertion/Deletion at tail: $O(1)$ if tail pointer is maintained, otherwise $O(n)$.
    - Search: $O(n)$.
  - **Space Complexity:** $O(n)$.
- **Use-cases:** Implementing stacks, queues, and adjacency lists for graphs.

### Array

- **Description:** A contiguous block of memory that stores elements of the same type. Elements are accessed using an index.
- **Main Properties:**
  - **Time Complexity:**
    - Access: $O(1)$.
    - Search: $O(n)$ (unless sorted and using binary search).
    - Insertion/Deletion: $O(n)$ (due to shifting elements).
  - **Space Complexity:** $O(n)$.
- **Use-cases:** Storing and accessing sequential data, matrices, and buffers.

------

## Common Algorithms

### Sorting Algorithms

#### Bubble Sort
- **Description:** Repeatedly steps through the list, compares adjacent elements, and swaps them if they are in the wrong order.
- **Time Complexity:**
  - Worst-case: $O(n^2)$.
  - Best-case: $O(n)$ (when the list is already sorted).
- **Space Complexity:** $O(1)$ (in-place sorting).
- **Use-cases:** Educational purposes, small datasets.

#### Merge Sort
- **Description:** A divide-and-conquer algorithm that divides the input array into two halves, recursively sorts them, and then merges the two sorted halves.
- **Time Complexity:** $O(n \log n)$ in all cases.
- **Space Complexity:** $O(n)$ (requires additional space for merging).
- **Use-cases:** Large datasets, external sorting.

#### Quick Sort
- **Description:** A divide-and-conquer algorithm that selects a 'pivot' element and partitions the array around the pivot.
- **Time Complexity:**
  - Average-case: $O(n \log n)$.
  - Worst-case: $O(n^2)$ (when the pivot is poorly chosen).
- **Space Complexity:** $O(\log n)$ (due to recursion stack).
- **Use-cases:** General-purpose sorting, libraries (e.g., C's `qsort`).

### Searching Algorithms

#### Linear Search
- **Description:** Sequentially checks each element in a list until it finds the target value.
- **Time Complexity:** $O(n)$.
- **Space Complexity:** $O(1)$.
- **Use-cases:** Small or unsorted datasets.

#### Binary Search
- **Description:** Efficiently locates a target value in a sorted array by repeatedly dividing the search interval in half.
- **Time Complexity:** $O(\log n)$.
- **Space Complexity:** $O(1)$ (iterative) or $O(\log n)$ (recursive).
- **Use-cases:** Searching in large, sorted datasets.

### Graph Algorithms

#### Breadth-First Search (BFS)
- **Description:** Explores all neighbors at the present depth before moving on to nodes at the next depth level.
- **Time Complexity:** $O(V + E)$.
- **Use-cases:** Shortest path in unweighted graphs, level-order traversal.

#### Depth-First Search (DFS)
- **Description:** Explores as far as possible along each branch before backtracking.
- **Time Complexity:** $O(V + E)$.
- **Use-cases:** Topological sorting, detecting cycles, solving puzzles.

#### Dijkstra's Algorithm
- **Description:** A greedy algorithm for finding the shortest paths from a single source node to all other nodes in a graph with non-negative edge weights.
- **Time Complexity:** $O((V + E) \log V)$, where $V$ is the number of vertices and $E$ is the number of edges.
- **Use-cases:** GPS navigation systems, network routing protocols, and finding the shortest path in a maze.
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

#### Kruskal's Algorithm
- **Description:** Finds a minimum spanning tree for a connected weighted graph by sorting all edges and adding them if they connect disjoint sets.
- **Time Complexity:** $O(E \log E)$ or $O(E \log V)$, where $E$ is the number of edges and $V$ is the number of vertices.
- **Use-cases:** Network design, clustering, and circuit design.

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

Understanding data structures and algorithms is fundamental to writing efficient and scalable code. By choosing the right data structure and algorithm for a given problem, you can optimize both time and space complexity, leading to better performance and resource utilization. This primer provides a foundation for further exploration into more advanced topics in computer science.
