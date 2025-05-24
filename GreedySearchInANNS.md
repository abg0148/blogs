# GreedySearch Algorithm for Approximate Nearest Neighbor Retrieval

The **GreedySearch** algorithm is a graph-based procedure designed for efficient retrieval of approximate nearest neighbors (ANNs). Each node in the graph corresponds to a data point, with edges indicating local neighborhood relationships. This method is central to systems such as **HNSW** \[1], **DiskANN** \[2], and **FreshDiskANN** \[3], which target scalable, low-latency ANN search in high-dimensional spaces.

---

## Objective

Given a query point $x_q$ (not necessarily present in the graph), the algorithm returns $k$ approximate nearest neighbors by traversing the graph in a greedy manner. Rather than scanning the entire dataset, GreedySearch prioritizes locally promising candidates based on geometric proximity to $x_q$.

---

## Inputs

* $G$: Directed graph over the dataset where each node has a set of out-neighbors.
* $s$: The starting node for the search.
* $x_q$: The query vector.
* $k$: Number of nearest neighbors to return.
* $L$: Search list limit; controls the number of candidates retained during search (analogous to beam width).

---

## Output

* $\ell$: List of $k$ approximate nearest neighbors to $x_q$.
* $\mathcal{V}$: Set of visited nodes during the traversal.

---

## Algorithm (Original Notation from the Literature)

```
Algorithm 1: GreedySearch
Input: G, s, x_q, k, L
Output: ℓ, 𝒱
Initialize ℓ ← {s}, 𝒱 ← ∅
while ℓ \ 𝒱 ≠ ∅ do
    p* ← argmin_{p ∈ ℓ \ 𝒱} ||x_p - x_q||
    ℓ ← ℓ ∪ N_out(p*)
    𝒱 ← 𝒱 ∪ {p*}
    if |ℓ| > L then
        ℓ ← top L nodes in ℓ closest to x_q
return top k nodes in ℓ closest to x_q, 𝒱
```

### Clarification

* The set difference symbol `\` refers to nodes in $\ell$ that have not yet been visited (i.e., $\ell \setminus \mathcal{V}$).

---

## Reformulated Python-style Pseudocode with Descriptive Variables

```python
# GreedySearch for approximate nearest neighbor retrieval in a proximity graph

def greedy_search(graph, start_node, query_vector, k, L):
    candidate_set = {start_node}         # Initialize search with starting node
    visited_set = set()                  # Track visited nodes to prevent revisits

    while candidate_set - visited_set:
        # Select the closest unvisited node to the query
        unvisited = candidate_set - visited_set
        current_node = min(unvisited, key=lambda n: distance(n, query_vector))

        visited_set.add(current_node)    # Mark as visited
        candidate_set.update(graph.neighbors(current_node))  # Explore neighbors

        if len(candidate_set) > L:
            # Retain only the L closest candidates
            candidate_set = set(sorted(candidate_set,
                                       key=lambda n: distance(n, query_vector))[:L])

    final_candidates = sorted(candidate_set, key=lambda n: distance(n, query_vector))[:k]
    return final_candidates, visited_set
```

---

## Toy Example

To illustrate the mechanics of GreedySearch, consider a graph with 5 nodes and their distances to a query vector $x_q$:

| Node | Distance to $x_q$ | Neighbors |
| ---- | ----------------- | --------- |
| A    | 10                | B, C      |
| B    | 7                 | A, D      |
| C    | 5                 | A, D      |
| D    | 3                 | B, C, E   |
| E    | 8                 | D         |

* **Start node**: A
* **Query**: $x_q$
* **$k = 2$** nearest neighbors
* **$L = 3$** (search list limit)

### Execution Trace

1. **Initialization**:

   * candidate\_set = {A}, visited = ∅

2. **Visit A (dist=10)** → Add B, C

   * candidate\_set = {A, B, C}, visited = {A}

3. **Visit C (dist=5)** → Add D

   * candidate\_set = {A, B, C, D} → Prune to {B, C, D}
   * visited = {A, C}

4. **Visit D (dist=3)** → Add E

   * candidate\_set = {B, C, D, E} → Prune to {D, C, B}
   * visited = {A, C, D}

5. **Visit B (dist=7)** → Neighbors already visited

   * visited = {A, B, C, D}

6. **Terminate**: No unvisited candidates

7. **Output**: top-2 closest nodes = \[D (3), C (5)]

---

## Conclusion

GreedySearch is a deterministic, local search heuristic that traverses proximity graphs to identify approximate nearest neighbors. Its use of a bounded search horizon (parameter $L$) allows it to balance computational efficiency and retrieval quality. The algorithm is especially effective in conjunction with graph construction techniques that enforce small-world properties and distance diversity among neighbors \[1, 2, 3].

---

## References

\[1] Yury A. Malkov and D. Yashunin, “Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs,” *IEEE Transactions on Pattern Analysis and Machine Intelligence (TPAMI)*, 2020. \[[arXiv:1603.09320](https://arxiv.org/abs/1603.09320)]

\[2] Tharun Medini and Anshumali Shrivastava, “DiskANN: Fast Accurate Billion-point Nearest Neighbor Search on a Single Node,” *Advances in Neural Information Processing Systems (NeurIPS)*, 2019. \[[arXiv:2002.08440](https://arxiv.org/abs/2002.08440)]

\[3] Aditi Singh et al., “FreshDiskANN: A Fast and Accurate Graph-Based ANN Index for Streaming Similarity Search,” *arXiv preprint arXiv:2105.09613*, 2021.
