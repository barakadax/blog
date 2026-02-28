# Data structures
- [What is a data structure?](#what-is-a-data-structure)
- [Why do we need data structures?](#why-do-we-need-data-structures)
- [Types of data structures](#types-of-data-structures)
  - [Primitive](#primitive)
  - [Non-Primitive](#non-primitive)
- [Sources](#sources)

## What is a data structure?

Data structure is a specialized format for organizing, processing, and storing data so that it can be accessed and modified efficiently.

## Why do we need data structures?

Data structures serves as blueprints for how information is arranged in computer memory to solve specific problems.
For example seaching values or managing hierarchy.

## Types of data structures

![Data structures hierarchy](https://raw.githubusercontent.com/barakadax/blog/refs/heads/Master/Data%20structues/graph.png)

### Primitive
Primitive data structures are the basic data structures that are used to store single values, the actual value, directly in memory, always in fixed size:

- Int
- Float
- Double
- Char
- Boolean
- Fixed point
- Reference
- Symbol
- Enumerated type
- Complex

### Non-Primitive
Non-primitive data structures are complex data structures that are derived from primitive ones.
They are used to store multiple values and can be categorized based on how they organize and access data.

#### Linear Data Structures
In linear data structures, elements are arranged in a sequence, and each element is connected to its previous and next element.

##### Direct Access
These allow accessing elements directly using an index or key:
- **Array**: Fixed-size collection of similar elements stored in contiguous memory.
  - *When to use:* When you know the number of elements in advance and need fast access by index (e.g., storing RGB values of a pixel).
- **Matrix**: Two-dimensional array representing a grid or table.
  - *When to use:* For mathematical computations, image processing, or representing game boards (e.g., Sudoku or Chess).
- **Dynamic Array/Vector**: Arrays that can grow in size automatically when they run out of space.
  - *When to use:* When you need an array-like structure but don't know the capacity beforehand (e.g., a list of items in a shopping cart).
- **String**: A specialized array used for storing a sequence of characters.
  - *When to use:* For text processing, storing names, or any character-based data.
- **Tuple**: Fixed-length, immutable sequence of elements, often of different types.
  - *When to use:* To return multiple values from a function or store a fixed record (e.g., geographic coordinates `(lat, long)`).

##### Indirect Access
These require traversing through other elements to reach a specific one:
- **Linked Lists**: Elements (nodes) where each point to the next, allowing efficient insertion/deletion.
  - **Singly**: Each node points to the next.
  - **Doubly**: Each node points to both next and previous.
  - **Circular**: The last node points back to the first.
  - *When to use:* When frequent insertions/deletions at both ends are needed, or for implementing stacks/queues without size limits.
- **Stacks**: Follows LIFO (Last-In-First-Out) principle.
  - *When to use:* For undo mechanisms in software, expression evaluation (parsers), or backtracking algorithms.
- **Queues**: Follows FIFO (First-In-First-Out) principle.
  - **Simple Queue**: Standard insertion at back, removal from front.
  - **Circular Queue**: Last position is connected back to the first.
  - **Priority Queue**: Elements are removed based on priority rather than order.
  - **Deque (Double-ended queue)**: Insertion/removal allowed at both ends.
  - *When to use:* For task scheduling (CPU/Printer), handling asynchronous data (IO buffers), or BFS (Breadth-First Search).
##### Complexity Summary (Linear - Worst Case)


| Data Structure | Access | Search | Insert | Delete | Space |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Array** | $O(1)$ | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ |
| **Matrix** | $O(1)$ | $O(r \cdot c)$| $O(n)$ | $O(n)$ | $O(r \cdot c)$ |
| **Dynamic Array** | $O(1)$ | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ |
| **String** | $O(1)$ | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ |
| **Tuple** | $O(1)$ | $O(n)$ | $N/A$ | $N/A$ | $O(n)$ |
| **Singly Linked List** | $O(n)$ | $O(n)$ | $O(1)$ | $O(1)$ | $O(n)$ |
| **Doubly Linked List** | $O(n)$ | $O(n)$ | $O(1)$ | $O(1)$ | $O(n)$ |
| **Circular List** | $O(n)$ | $O(n)$ | $O(1)$ | $O(1)$ | $O(n)$ |
| **Stack** | $O(n)$ | $O(n)$ | $O(1)$ | $O(1)$ | $O(n)$ |
| **Simple Queue** | $O(n)$ | $O(n)$ | $O(1)$ | $O(1)$ | $O(n)$ |
| **Priority Queue** | $O(1)$ | $O(n)$ | $O(\log n)$ | $O(\log n)$ | $O(n)$ |
| **Deque** | $O(n)$ | $O(n)$ | $O(1)$ | $O(1)$ | $O(n)$ |

> [!NOTE]
> $O(1)^*$ for **Dynamic Array** indicates amortized time for insertion (grow calls are rare).
> **Tuples** are immutable, so insertion/deletion are not applicable.
> **Priority Queues** often use Heaps internally, giving $O(\log n)$ for structural changes.
> **Matrices** search assumes traversal over rows ($r$) and columns ($c$).
Elements in non-linear data structures are not arranged in a sequence. Instead, they form hierarchical or relational connections.

##### Hierarchical
- **Trees**: Hierarchical structures with a root node and child nodes.
  - **BST (Binary Search Tree)**: Each node has at most two children; left is smaller, right is larger.
    - *When to use:* For fast searching, insertion, and deletion in sorted data.
  - **AVL Tree**: Self-balancing BST where heights of subtrees differ by at most one.
    - *When to use:* When search operations are much more frequent than insertions/deletions.
  - **Red-Black Tree**: Self-balancing BST that uses "colors" to ensure balance.
    - *When to use:* For efficient insertion/deletion in maps and sets (used in many language libraries).
  - **B-Tree**: Balanced tree designed for systems that read/write large blocks of data.
    - *When to use:* For indexing in databases and file systems.
- **Heaps**: Complete binary tree where the parent is always greater/smaller than its children.
  - **Min-heap**: Parent is the minimum.
  - **Max-heap**: Parent is the maximum.
  - *When to use:* For implementing priority queues and the Heapsort algorithm.
- **Tries**: Prefix tree used to store a dynamic set of strings.
  - **Standard Trie**: Each node represents a character.
  - **Compressed Trie**: Merges nodes with only one child to save space.
  - *When to use:* For autocomplete features, spell checkers, and IP routing.

##### Complexity Summary (Hierarchical - Worst Case)


| Data Structure | Search | Insert | Delete | Space |
| :--- | :---: | :---: | :---: | :---: |
| **Binary Search Tree**| $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ |
| **AVL Tree** | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $O(n)$ |
| **Red-Black Tree** | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $O(n)$ |
| **B-Tree** | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $O(n)$ |
| **Heap** | $O(n)$ | $O(\log n)$ | $O(\log n)$ | $O(n)$ |
| **Standard Trie** | $O(L)$ | $O(L)$ | $O(L)$ | $O(AL)$ |
| **Compressed Trie** | $O(L)$ | $O(L)$ | $O(L)$ | $O(n)$ |

> [!NOTE]
> For **Tries**, $L$ is the length of the string and $A$ is the alphabet size.
> Self-balancing trees (AVL, RB) guarantee $O(\log n)$ search.

##### Relational/Unordered
- **Graphs**: A collection of nodes (vertices) and the connections (edges) between them.
  - **Directed**: Edges have a direction.
  - **Undirected**: Edges have no direction.
  - **Weighted**: Edges have assigned values (costs).
  - *When to use:* To represent social networks, maps/navigation systems, or dependency graphs (e.g., package managers).
- **Hashing**: Mapping data of arbitrary size to fixed-size values using a hash function.
  - **Hash Table/Map/Set**: Key-value pairs or unique sets for near-constant time operations.
  - *When to use:* For fast data retrieval, unique item tracking, and implementing caches.
- **Composite**: Data structures composed of primitive types or other containers.
  - **Record/Struct**: Groups related fields under one name.
  - **Union**: A structure that can store different types, but only one at a time.
  - *When to use:* To group related data (e.g., a `User` object with ID, name, and email).

##### Advanced & Specialized
- **Spatial**: Structures for organizing data associated with geometric space.
  - **Quadtree/Octree/R-tree**: Used for indexing geographic coords or 3D objects.
  - *When to use:* For collision detection in games or spatial indexing in GIS applications.
- **Range Queries**: Efficiently perform computations over a sub-range of elements.
  - **Segment Tree / Fenwick Tree**: Used for range sum, range min/max queries.
  - *When to use:* For competitive programming problems involving dynamic range updates.
- **Probabilistic**: Structures that provide approximate answers with very high efficiency.
  - **Bloom Filter**: Checks if an element is in a set (may return false positives).
  - **HyperLogLog**: Estimates the number of unique elements in a large dataset.
  - **Skip List**: Linked list with layers for faster search.
  - *When to use:* For large-scale web systems (e.g., checking if a URL is malicious or counting unique site visitors).
- **Disjoint Sets**: Keeps track of elements partitioned into non-overlapping sets.
  - **Union-Find**: Efficiently merges sets and finds the set an element belongs to.
  - *When to use:* For detecting cycles in graphs or finding connected components.
- **String Specialized**: Advanced structures for complex string pattern matching.
  - **Suffix Tree / Array**: Indexes all suffixes of a string for fast substring search.
  - **Aho-Corasick**: Matches multiple patterns simultaneously in a text.
  - *When to use:* For DNA sequence analysis, plagiarism detection, or search engine indexing.

##### Complexity Summary (Unordered & Advanced - Worst Case)


| Data Structure | Search | Insert | Delete | Space |
| :--- | :---: | :---: | :---: | :---: |
| **Hash Table/Map/Set** | $O(n)$ | $O(n)$ | $O(n)$| $O(n)$ |
| **Graphs (Adj. List)** | $O(V+E)$ | $O(1)$ | $O(1)$ | $O(V+E)$ |
| **Graphs (Adj. Matrix)** | $O(V)$ | $O(V^2)$ | $O(V^2)$ | $O(V^2)$ |
| **Record / Union** | $O(1)$ | $N/A$ | $N/A$ | $O(\sum S_i)$ |
| **Quad/Oct/R-Tree** | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ |
| **Segment/Fenwick Tree**| $O(\log n)$ | $O(\log n)$ | $N/A$ | $O(n)$ |
| **Bloom Filter** | $O(k)$ | $O(k)$ | $N/A$ | $O(m)$ |
| **Skip List** | $O(n)$ | $O(n)$ | $O(n)$ | $O(n \log n)$ |
| **HyperLogLog** | $N/A$ | $O(1)$ | $N/A$ | $O(\log \log n)$ |
| **Union-Find** | $O(\alpha(n))$ | $O(\alpha(n))$ | $N/A$ | $O(n)$ |
| **Suffix Tree** | $O(m)$ | $O(n)$ | $N/A$ | $O(n)$ |
| **Suffix Array** | $O(m \log n)$ | $O(n \log n)$ | $N/A$ | $O(n)$ |
| **Aho-Corasick** | $O(L + k)$ | $O(\sum L_i)$ | $N/A$ | $O(\sum L_i)$ |

> [!NOTE]
> **$\alpha(n)$** is the inverse Ackermann function, which is nearly $O(1)$.
> **HyperLogLog** is for cardinality estimation with minimal memory.

## Sources

- [Complexity](https://barakadax.github.io/blog?article=Complexity)
- [Computerphile](https://www.youtube.com/@Computerphile)
- [W3schools](https://www.w3schools.com/dsa/dsa_intro.php)
- [Wikipedia](https://en.wikipedia.org/wiki/List_of_data_structures)
- [Designgurus](https://www.designgurus.io/course-play/grokking-data-structures-for-coding-interviews/doc/types-of-data-structures?gad_source=1&gad_campaignid=23163907085&gbraid=0AAAAADME9yon1FEFCXDsEt4mt38UD6RPJ&gclid=Cj0KCQiAwYrNBhDcARIsAGo3u31JIUbwNNsWM3wmef4VAmelXrWLbb5WYsLNWK9f_EIsJag9V9XZsBAaAql4EALw_wcB)
- [Big O Cheat Sheet](https://www.bigocheatsheet.com/)
