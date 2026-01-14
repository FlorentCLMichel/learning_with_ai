# Data Structures and Algorithms: A Technical Primer

This document provides a brief overview of fundamental data structures and algorithms.

------

## Understanding Complexity

### **Time Complexity**

- **Description:** Time complexity measures the amount of time an algorithm takes to complete as a function of the size of the input. It is expressed using Big-O notation, which describes the upper bound of the growth rate.
- **Common Notations:**
  - **$O(1)$:** Constant time. The algorithm takes the same amount of time regardless of input size.
  - **$O(\log n)$:** Logarithmic time. The time increases logarithmically with the input size.
  - **$O(n)$:** Linear time. The time increases linearly with the input size.
  - **$O(n \log n)$:** Linearithmic time. Common in efficient sorting algorithms like Merge Sort.
  - **$O(n^2)$:** Quadratic time. The time increases quadratically with the input size, often seen in nested loops.
  - **$O(2^n)$:** Exponential time. The time doubles with each additional input element, typical in recursive algorithms without optimization.

### **Space Complexity**

- **Description:** Space complexity measures the amount of memory an algorithm uses as a function of the input size. It is also expressed using Big-O notation.
- **Common Notations:**
  - **$O(1)$:** Constant space. The algorithm uses a fixed amount of memory regardless of input size.
  - **$O(n)$:** Linear space. The memory usage increases linearly with the input size.
  - **$O(n^2)$:** Quadratic space. The memory usage increases quadratically with the input size.

### **Why Complexity Matters**

- **Efficiency:** Understanding complexity helps in choosing the right data structure or algorithm for a given problem, ensuring optimal performance.
- **Scalability:** Algorithms with lower time complexity scale better with large input sizes, making them suitable for big data applications.
- **Resource Management:** Space complexity is crucial for memory-constrained environments, such as embedded systems or mobile devices.

------

## Common Data Structures

### **Hash Table (Hash Map)**

- **Description:** A data structure that implements an associative array abstract data type, mapping keys to values using a **hash function** to compute an index into an array of buckets or slots.
- **Main Properties:**
  - **Time Complexity:** $O(1)$ average case for Search, Insert, and Delete. $O(n)$ worst case (when many keys hash to the same index).
  - **Space Complexity:** $O(n)$.
  - **Collisions:** Handled via **Chaining** (linked lists in each bucket) or **Open Addressing** (finding the next empty slot).
- **Use-cases:** Database indexing, caching (LRU caches), unique element counting, and implementing Sets.

### **Binary Search Tree (BST) & Balanced Trees (AVL, Red-Black)**

- **Description:** A node-based tree structure where each node has at most two children. For any node, the left subtree contains values less than the node, and the right subtree contains values greater.
- **Main Properties:**
  - **Time Complexity:** In a balanced tree (like an **AVL** or **Red-Black Tree**), Search, Insert, and Delete are $O(\log n)$.
  - **Traversal:** In-order traversal yields elements in sorted order.
- **Use-cases:** Implementing ordered sets and maps, file systems (B-Trees), and priority queues.

### **Priority Queue**

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

### **Heaps (Binary Heap)**

- **Description:** A complete binary tree that satisfies the **heap property**: in a Max-Heap, the parent is always greater than or equal to its children; in a Min-Heap, it is less than or equal.
- **Main Properties:**
  - **Access Max/Min:** $O(1)$.
  - **Insert/Delete (Extract):** $O(\log n)$.
  - **Heapify:** Building a heap from an unordered array takes $O(n)$.
- **Use-cases:** Priority queues, scheduling algorithms (like K8s pod scheduling), and finding the "top K" elements in a stream.

### **Graphs (Adjacency List vs. Matrix)**

- **Description:** A collection of vertices (nodes) connected by edges.
- **Main Properties:**
  - **Adjacency List:** $O(V + E)$ space. Better for sparse graphs. Fast to iterate over neighbors.
  - **Adjacency Matrix:** $O(V^2)$ space. Better for dense graphs. $O(1)$ to check if an edge exists between two specific nodes.
- **Use-cases:** Social networks, recommendation engines, pathfinding in maps, and dependency resolution in build systems.

### Quick Reference: Time Complexity Table

| **Data Structure**        | **Access**  | **Search**  | **Insertion** | **Deletion** |
| ------------------------- | ----------- | ----------- | ------------- | ------------ |
| **Array**                 | $O(1)$      | $O(n)$      | $O(n)$        | $O(n)$       |
| **Hash Table**            | N/A         | $O(1)$      | $O(1)$        | $O(1)$       |
| **Priority Queue (Heap)** | $O(1)$      | $O(n)$      | $O(\log n)$   | $O(\log n)$  |
| **BST (Balanced)**        | $O(\log n)$ | $O(\log n)$ | $O(\log n)$   | $O(\log n)$  |
| **Singly Linked List**    | $O(n)$      | $O(n)$      | $O(1)$        | $O(1)$       |

------

## Common Algorithms

### **Dijkstra's Algorithm**

- **What it is for:** Finding the shortest path from a single source node to all other nodes in a weighted graph with non-negative edge weights.
- **Description:** It maintains a set of visited nodes and a set of unvisited nodes. It repeatedly picks the unvisited node with the smallest tentative distance, updates the distances of its neighbors, and marks it as visited.
- **Complexity Analysis:**
  - **Runtime:** $O((V + E) \log V)$ when using a Fibonacci heap or a standard Binary Priority Queue.
  - **Memory:** $O(V + E)$ to store the graph and distances.
- **Variations:**
  - **A\* Search:** Uses heuristics to speed up the search (common in AI/Games).
  - **Bellman-Ford:** Can handle negative edge weights ($O(VE)$).
- **Use-cases:** GPS navigation, network routing protocols (OSPF).

### **Binary Search**

- **What it is for:** Finding the position of a target value within a **sorted** array.
- **Description:** It compares the target value to the middle element of the array. If they are not equal, the half in which the target cannot lie is eliminated, and the search continues on the remaining half.
- **Complexity Analysis:**
  - **Runtime:** $O(\log n)$.
  - **Memory:** $O(1)$ for iterative implementation; $O(\log n)$ for recursive due to stack depth.
- **Use-cases:** Searching in databases, finding roots of functions (Numerical Analysis), and debugging (Git Bisect).

### **Merge Sort**

- **What it is for:** A stable, comparison-based sorting algorithm.
- **Description:** A **Divide and Conquer** algorithm. It divides the array into two halves, recursively sorts them, and then merges the two sorted halves back together.
- **Complexity Analysis:**
  - **Runtime:** $O(n \log n)$ in all cases (best, average, worst).
  - **Memory:** $O(n)$ auxiliary space for the merging process.
- **Variations:** **Timsort** (used in Python/Java), which optimizes Merge Sort for real-world data that often contains pre-sorted runs.
- **Use-cases:** Sorting linked lists (where $O(1)$ extra space is possible), external sorting (sorting data too large for RAM).

### **Breadth-First Search (BFS) & Depth-First Search (DFS)**

- **What it is for:** Traversing or searching tree or graph data structures.
- **Description:**
  - **BFS:** Explores neighbors level-by-level using a **Queue**.
  - **DFS:** Explores as far as possible along each branch before backtracking using a **Stack** (or recursion).
- **Complexity Analysis:**
  - **Runtime:** $O(V + E)$.
  - **Memory:** $O(V)$. BFS memory is proportional to the "width" of the graph; DFS to the "depth."
- **Use-cases:**
  - **BFS:** Finding the shortest path in an unweighted graph, peer-to-peer networks.
  - **DFS:** Topological sorting (build systems), solving puzzles (mazes), cycle detection in graphs.

------
