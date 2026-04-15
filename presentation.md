# Greedy Algorithms

**Computer Science Fundamentals Series**

Greedy choice property · Activity selection · Huffman coding · Fractional knapsack · Exchange arguments · Matroid theory

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [The Greedy Paradigm](#slide-02--the-greedy-paradigm)
2. [Greedy Choice Property & Optimal Substructure](#slide-03--greedy-choice-property--optimal-substructure)
3. [Activity Selection / Interval Scheduling](#slide-04--activity-selection--interval-scheduling)
4. [Activity Selection -- Proof & Implementation](#slide-05--activity-selection--proof--implementation)
5. [Huffman Coding -- Idea](#slide-06--huffman-coding--idea)
6. [Huffman Coding -- Algorithm](#slide-07--huffman-coding--algorithm)
7. [Fractional Knapsack](#slide-08--fractional-knapsack)
8. [Job Sequencing with Deadlines](#slide-09--job-sequencing-with-deadlines)
9. [Minimum Number of Coins](#slide-10--minimum-number-of-coins)
10. [Minimum Number of Platforms](#slide-11--minimum-number-of-platforms)
11. [Greedy Graph Algorithms -- Kruskal's MST](#slide-12--greedy-graph-algorithms--kruskals-mst)
12. [Greedy Graph Algorithms -- Prim's MST](#slide-13--greedy-graph-algorithms--prims-mst)
13. [Greedy Graph Algorithms -- Dijkstra's Shortest Path](#slide-14--greedy-graph-algorithms--dijkstras-shortest-path)
14. [Greedy vs Dynamic Programming](#slide-15--greedy-vs-dynamic-programming)
15. [When Greedy Fails](#slide-16--when-greedy-fails)
16. [Exchange Arguments for Proofs](#slide-17--exchange-arguments-for-proofs)
17. [Matroid Theory](#slide-18--matroid-theory)
18. [Common Greedy Patterns & Pitfalls](#slide-19--common-greedy-patterns--pitfalls)
19. [Summary & Further Reading](#slide-20--summary--further-reading)

---

## Slide 02 -- The Greedy Paradigm

### What is a greedy algorithm?

A greedy algorithm builds a solution piece by piece, always choosing the **locally optimal** option at each step, hoping that these local choices lead to a **globally optimal** solution.

- At each decision point, pick the choice that looks best *right now*
- Never reconsider previous choices -- no backtracking
- Hope that local optimality leads to global optimality
- Much simpler and faster than exhaustive search or dynamic programming

### The greedy template

```
GREEDY-SOLVE(problem):
    solution = empty
    while problem is not solved:
        candidate = SELECT best remaining option
        if candidate is FEASIBLE:
            solution = solution + candidate
    return solution
```

### When does it work?

A greedy strategy produces an optimal solution only when the problem exhibits two properties:

1. **Greedy choice property** -- a locally optimal choice is part of some globally optimal solution
2. **Optimal substructure** -- an optimal solution contains optimal solutions to its sub-problems

> Not every optimisation problem has these properties. Proving greedy correctness is the hard part -- the algorithm itself is usually simple.

---

## Slide 03 -- Greedy Choice Property & Optimal Substructure

### Greedy choice property

At every step, we can make a choice that is locally best without considering future sub-problems, and still reach a global optimum.

- The key insight: there exists an optimal solution that includes the greedy choice
- We do not need to solve all sub-problems first (unlike DP)
- Proof technique: assume an optimal solution that does *not* include the greedy choice, then show we can swap in the greedy choice without worsening the result

### Optimal substructure

After making the greedy choice, the remaining problem is a smaller instance of the same type, and its optimal solution combined with the greedy choice yields the overall optimum.

- Shared with dynamic programming -- both require this property
- The difference: greedy commits to one sub-problem; DP considers all

### Proof strategy

| Step | Description |
|------|------------|
| **1. Define** | Formalise the greedy choice |
| **2. Assume** | Suppose an optimal solution `OPT` exists that does not include the greedy choice |
| **3. Exchange** | Modify `OPT` by swapping in the greedy choice -- show the result is no worse |
| **4. Conclude** | Therefore, an optimal solution including the greedy choice exists |

> This "exchange argument" is the workhorse proof technique for greedy correctness.

---

## Slide 04 -- Activity Selection / Interval Scheduling

### Problem

Given `n` activities with start and finish times `(s_i, f_i)`, select the maximum number of non-overlapping activities.

| Activity | Start | Finish |
|----------|-------|--------|
| A | 0 | 6 |
| B | 1 | 4 |
| C | 3 | 5 |
| D | 5 | 7 |
| E | 3 | 9 |
| F | 5 | 9 |
| G | 6 | 10 |
| H | 8 | 11 |

### Greedy strategy

**Sort by finish time** (earliest first), then greedily select each activity whose start time is >= the finish time of the last selected activity.

### Why earliest finish time?

- Earliest start? Fails -- a long activity starting early blocks many short ones
- Shortest duration? Fails -- a short activity in the middle can block two non-overlapping ones
- Fewest conflicts? Works but is slower to compute
- **Earliest finish** leaves the most room for subsequent activities

> Earliest finish time is the canonical greedy choice for interval scheduling.

---

## Slide 05 -- Activity Selection -- Proof & Implementation

### Correctness proof sketch

1. Sort activities by finish time: `f_1 <= f_2 <= ... <= f_n`
2. Activity 1 (earliest finish) is in some optimal solution -- if not, swap the first activity in OPT with activity 1; it finishes no later, so no new conflicts arise
3. After selecting activity 1, the remaining problem is the same type -- select max non-overlapping activities from those starting after `f_1`
4. By induction, the greedy algorithm is optimal

### Implementation

```python
def activity_selection(activities):
    # Sort by finish time
    activities.sort(key=lambda x: x[1])
    selected = [activities[0]]
    last_finish = activities[0][1]

    for start, finish in activities[1:]:
        if start >= last_finish:
            selected.append((start, finish))
            last_finish = finish

    return selected
```

### Complexity

| Metric | Value |
|--------|-------|
| **Time** | `O(n log n)` -- dominated by sorting |
| **Space** | `O(1)` extra (if sorted in place) |

> The weighted variant (each activity has a profit) requires dynamic programming -- greedy no longer works.

---

## Slide 06 -- Huffman Coding -- Idea

### Problem

Given a set of characters with known frequencies, assign variable-length binary codes to minimise the total encoded length.

### Key insight

- Frequent characters get short codes; rare characters get long codes
- Must be a **prefix-free code** -- no code is a prefix of another -- to allow unambiguous decoding
- Huffman's algorithm builds an optimal prefix-free code using a greedy strategy

### Example

| Character | Frequency | Fixed (3-bit) | Huffman |
|-----------|-----------|---------------|---------|
| a | 45 | 000 | 0 |
| b | 13 | 001 | 101 |
| c | 12 | 010 | 100 |
| d | 16 | 011 | 111 |
| e | 9 | 100 | 1101 |
| f | 5 | 101 | 1100 |

- Fixed-length: `100 * 3 = 300` bits
- Huffman: `45*1 + 13*3 + 12*3 + 16*3 + 9*4 + 5*4 = 224` bits -- **25% smaller**

> Huffman coding is the foundation of data compression. Used in DEFLATE (gzip, PNG), JPEG, and many other formats.

---

## Slide 07 -- Huffman Coding -- Algorithm

### Algorithm

1. Create a leaf node for each character with its frequency
2. Insert all nodes into a **min-priority queue**
3. While the queue has more than one node:
   - Extract the two nodes with lowest frequency
   - Create a new internal node with these two as children; frequency = sum of children
   - Insert the new node back into the queue
4. The remaining node is the root of the Huffman tree

### Greedy choice

Always merge the two lowest-frequency nodes. This ensures the rarest characters end up deepest in the tree (longest codes).

### Implementation sketch

```python
import heapq

def huffman(freq):
    heap = [(f, i, char) for i, (char, f) in enumerate(freq.items())]
    heapq.heapify(heap)
    counter = len(heap)

    while len(heap) > 1:
        f1, _, left  = heapq.heappop(heap)
        f2, _, right = heapq.heappop(heap)
        merged = (left, right)
        heapq.heappush(heap, (f1 + f2, counter, merged))
        counter += 1

    return heap[0][2]   # root of the tree
```

### Complexity

- **Time:** `O(n log n)` -- each heap operation is `O(log n)`, performed `O(n)` times
- **Space:** `O(n)` for the tree

---

## Slide 08 -- Fractional Knapsack

### Problem

Given `n` items, each with weight `w_i` and value `v_i`, and a knapsack of capacity `W`, maximise total value. **Fractions of items are allowed.**

### Greedy strategy

1. Compute value-to-weight ratio `v_i / w_i` for each item
2. Sort items by ratio in decreasing order
3. Greedily add items: take the whole item if it fits; otherwise take the fraction that fills the remaining capacity

### Example

| Item | Value | Weight | Ratio |
|------|-------|--------|-------|
| A | 60 | 10 | 6.0 |
| B | 100 | 20 | 5.0 |
| C | 120 | 30 | 4.0 |

Capacity `W = 50`: take all of A (10), all of B (20), then 20/30 of C.

Total value: `60 + 100 + (20/30)*120 = 60 + 100 + 80 = 240`

### Why greedy works here

The fractional knapsack has the greedy choice property: the item with the highest ratio should be included as much as possible. If it were not, we could swap in more of it for less of a lower-ratio item, improving the total.

### Complexity

- **Time:** `O(n log n)` for sorting
- **Space:** `O(1)` extra

> **0/1 Knapsack** (no fractions allowed) does NOT have the greedy choice property. It requires dynamic programming.

---

## Slide 09 -- Job Sequencing with Deadlines

### Problem

Given `n` jobs, each with a deadline `d_i` and profit `p_i`, and each job takes one unit of time, schedule jobs on a single machine to maximise total profit. A job earns its profit only if completed by its deadline.

### Greedy strategy

1. Sort jobs by profit in decreasing order
2. For each job, schedule it in the latest available slot before its deadline
3. If no slot is available, skip the job

### Example

| Job | Deadline | Profit |
|-----|----------|--------|
| J1 | 2 | 100 |
| J2 | 1 | 19 |
| J3 | 2 | 27 |
| J4 | 1 | 25 |
| J5 | 3 | 15 |

Sorted by profit: J1(100), J3(27), J4(25), J2(19), J5(15)

- Slot 2: J1 (profit 100)
- Slot 1: J3 (profit 27) -- deadline 2, slot 2 taken, use slot 1
- J4: deadline 1, slot 1 taken -- skip
- J2: deadline 1, slot 1 taken -- skip
- Slot 3: J5 (profit 15)

Total profit: **142**

### Complexity

- **Time:** `O(n log n)` for sorting + `O(n * d)` for scheduling (can be `O(n log n)` with Union-Find)
- **Space:** `O(d)` for the slot array

---

## Slide 10 -- Minimum Number of Coins

### Problem

Given a set of coin denominations and a target amount, find the minimum number of coins that sum to the target.

### Greedy strategy

Always pick the largest denomination that does not exceed the remaining amount.

### Example -- standard denominations

Coins: `{1, 5, 10, 25}`, Target: `41`

- 25 -> remaining 16
- 10 -> remaining 6
- 5 -> remaining 1
- 1 -> remaining 0

Result: **4 coins** (optimal)

### When greedy fails

Coins: `{1, 3, 4}`, Target: `6`

- Greedy: 4 + 1 + 1 = **3 coins**
- Optimal: 3 + 3 = **2 coins**

The greedy approach fails because the coin system is not **canonical** -- the denominations do not have the property that greedy always works.

> US/UK/Euro denominations are canonical -- greedy works. For arbitrary denominations, use DP. Testing whether a denomination set is canonical is itself a non-trivial problem.

---

## Slide 11 -- Minimum Number of Platforms

### Problem

Given arrival and departure times of `n` trains, find the minimum number of platforms required so that no train waits.

### Greedy strategy

1. Sort all arrival times and all departure times separately
2. Use a two-pointer technique: sweep through events in chronological order
3. An arrival increments the platform count; a departure decrements it
4. Track the maximum concurrent platform count

### Example

| Train | Arrival | Departure |
|-------|---------|-----------|
| T1 | 09:00 | 09:10 |
| T2 | 09:40 | 12:00 |
| T3 | 09:50 | 11:20 |
| T4 | 11:00 | 11:30 |
| T5 | 15:00 | 19:00 |
| T6 | 18:00 | 20:00 |

Peak overlap: T2, T3, T4 all present between 11:00--11:20 -> **3 platforms**

### Implementation

```python
def min_platforms(arrivals, departures):
    arrivals.sort()
    departures.sort()
    plat_needed = 0
    max_plat = 0
    i = j = 0
    while i < len(arrivals):
        if arrivals[i] <= departures[j]:
            plat_needed += 1
            max_plat = max(max_plat, plat_needed)
            i += 1
        else:
            plat_needed -= 1
            j += 1
    return max_plat
```

- **Time:** `O(n log n)` -- **Space:** `O(1)` extra

---

## Slide 12 -- Greedy Graph Algorithms -- Kruskal's MST

### Minimum Spanning Tree (MST)

Given a connected, weighted, undirected graph, find a spanning tree with minimum total edge weight.

### Kruskal's algorithm

1. Sort all edges by weight (ascending)
2. For each edge in sorted order:
   - If adding this edge does not create a cycle, include it in the MST
3. Stop when `V - 1` edges are selected

### Greedy choice

Always pick the lightest edge that does not form a cycle. The cut property guarantees this is safe: the minimum-weight edge crossing any cut must be in some MST.

### Cycle detection -- Union-Find

```
KRUSKAL(G):
    sort edges by weight
    UF = UnionFind(V)
    mst = []
    for (u, v, w) in sorted edges:
        if UF.find(u) != UF.find(v):
            mst.append((u, v, w))
            UF.union(u, v)
    return mst
```

### Complexity

| Metric | Value |
|--------|-------|
| **Time** | `O(E log E)` -- dominated by sorting edges |
| **Space** | `O(V)` for the Union-Find structure |

> Kruskal's is edge-centric. Best for sparse graphs where `E` is close to `V`.

---

## Slide 13 -- Greedy Graph Algorithms -- Prim's MST

### Prim's algorithm

1. Start from any vertex; add it to the MST set
2. Repeatedly add the minimum-weight edge connecting an MST vertex to a non-MST vertex
3. Stop when all vertices are in the MST

### Greedy choice

Always pick the cheapest edge that expands the current tree. The cut property again guarantees optimality.

### Pseudocode (binary heap)

```
PRIM(G, start):
    key[v] = INF for all v; key[start] = 0
    parent[v] = NIL for all v
    Q = min-heap of all vertices keyed by key[v]
    while Q is not empty:
        u = EXTRACT-MIN(Q)
        for each neighbour v of u:
            if v in Q and w(u,v) < key[v]:
                parent[v] = u
                key[v] = w(u,v)
                DECREASE-KEY(Q, v, key[v])
    return parent
```

### Complexity

| Implementation | Time |
|---------------|------|
| Adjacency matrix | `O(V^2)` |
| Binary heap + adjacency list | `O((V + E) log V)` |
| Fibonacci heap + adjacency list | `O(E + V log V)` |

> Prim's is vertex-centric. Best for dense graphs where `E` is close to `V^2`.

---

## Slide 14 -- Greedy Graph Algorithms -- Dijkstra's Shortest Path

### Problem

Find the shortest path from a single source to all other vertices in a weighted graph with **non-negative** edge weights.

### Algorithm

1. Set `dist[source] = 0`, `dist[v] = INF` for all other vertices
2. Use a min-priority queue keyed by `dist`
3. Extract the vertex `u` with minimum `dist`, then relax all its neighbours: if `dist[u] + w(u,v) < dist[v]`, update `dist[v]`
4. Repeat until the queue is empty

### Greedy choice

Always process the unvisited vertex with the smallest known distance. Once a vertex is extracted, its distance is final.

### Why non-negative weights?

If edge weights can be negative, a vertex extracted from the queue might later be reachable via a cheaper path through a negative edge. This breaks the greedy invariant. Use **Bellman-Ford** for graphs with negative weights.

### Complexity

| Implementation | Time |
|---------------|------|
| Adjacency matrix | `O(V^2)` |
| Binary heap + adjacency list | `O((V + E) log V)` |
| Fibonacci heap | `O(E + V log V)` |

> Dijkstra's is the most widely used shortest-path algorithm in practice -- GPS routing, network protocols (OSPF), and game AI pathfinding.

---

## Slide 15 -- Greedy vs Dynamic Programming

### Similarities

- Both require **optimal substructure**
- Both solve optimisation problems
- Both decompose the problem into sub-problems

### Key differences

| Aspect | Greedy | Dynamic Programming |
|--------|--------|-------------------|
| **Choice timing** | Before solving sub-problems | After solving sub-problems |
| **Sub-problems** | One sub-problem remains | Multiple overlapping sub-problems |
| **Backtracking** | Never | Considers all options |
| **Proof** | Greedy choice property | Bellman equation / recurrence |
| **Typical complexity** | `O(n log n)` | `O(n^2)` or `O(nW)` |
| **Implementation** | Simple loop | Table filling / memoisation |

### Decision guide

- Can you prove the greedy choice property? -> **Greedy**
- Do you need to consider multiple choices per step? -> **DP**
- Is there a natural "exchange argument"? -> **Greedy**
- Does the problem have overlapping sub-problems? -> **DP**

> When in doubt, try greedy first. If you can prove it, you get a simpler, faster algorithm. If you cannot, switch to DP.

---

## Slide 16 -- When Greedy Fails

### Classic counter-examples

**0/1 Knapsack:** Items `{(60,10), (100,20), (120,30)}`, capacity 50. Greedy by ratio takes items 1 and 2 (value 160). Optimal: items 2 and 3 (value 220).

**Coin change (non-canonical):** Denominations `{1, 3, 4}`, target 6. Greedy: 4+1+1 = 3 coins. Optimal: 3+3 = 2 coins.

**Longest path in a DAG:** Greedy by heaviest next edge does not guarantee the longest total path.

**Travelling salesman (TSP):** Nearest-neighbour heuristic produces tours far from optimal.

### Why greedy fails

- The locally best choice locks out a combination of future choices that would have been globally better
- There is no "exchange" that can fix the solution -- swapping the greedy choice in breaks feasibility or worsens the objective
- The problem lacks the greedy choice property, even though it may have optimal substructure

### Greedy as approximation

Even when greedy is not optimal, it often provides a **good approximation** with a provable bound:

- Set cover: greedy is `O(ln n)` factor of optimal
- TSP nearest-neighbour: within factor 2 of optimal (for metric TSP)
- Scheduling to minimise makespan: within 4/3 of optimal (LPT rule)

---

## Slide 17 -- Exchange Arguments for Proofs

### The technique

An exchange argument proves greedy correctness by showing that any optimal solution can be transformed into the greedy solution without loss of quality.

### Template

1. Let `OPT` be an arbitrary optimal solution
2. Let `G` be the greedy solution
3. Find the first point where `OPT` and `G` differ
4. Modify `OPT` at that point to match `G` -- show the result is still feasible and no worse
5. Repeat until `OPT` has been transformed into `G`
6. Conclude: `G` is optimal

### Example -- activity selection

- `OPT` starts with activity `a_k` (not the earliest-finishing activity `a_1`)
- Replace `a_k` with `a_1` in `OPT`: since `f_1 <= f_k`, activity `a_1` finishes no later, so it does not conflict with any activity that `a_k` did not conflict with
- The modified solution has the same number of activities and is still valid
- By induction on the remaining activities, the greedy solution is optimal

### Tips

- The "exchange" must preserve feasibility -- this is where most proofs break down
- Usually argues about the first difference between OPT and G
- Induction handles the remaining sub-problem
- If you cannot construct a valid exchange, the greedy choice property likely does not hold

---

## Slide 18 -- Matroid Theory

### What is a matroid?

A matroid is a pair `(S, I)` where `S` is a finite ground set and `I` is a family of "independent" subsets satisfying:

1. **Hereditary:** if `A` is in `I` and `B` is a subset of `A`, then `B` is in `I`
2. **Exchange:** if `A, B` are in `I` and `|A| < |B|`, then there exists `x` in `B \ A` such that `A + {x}` is in `I`

### Why matroids matter for greedy

**Rado-Edmonds theorem:** a greedy algorithm produces an optimal solution for maximising a weighted sum over independent sets of a matroid -- and *only* for matroids.

If your problem can be cast as optimising over a matroid, greedy is guaranteed to work.

### Examples of matroids

| Matroid | Ground set | Independent sets |
|---------|-----------|-----------------|
| **Graphic** | Edges of a graph | Acyclic edge subsets (forests) |
| **Uniform** | Any set | Subsets of size <= k |
| **Partition** | Partitioned elements | At most one element per partition |
| **Linear** | Vectors | Linearly independent subsets |

> Kruskal's MST works because forests form the independent sets of a graphic matroid. The greedy algorithm on this matroid finds a maximum-weight basis (= MST).

---

## Slide 19 -- Common Greedy Patterns & Pitfalls

### Patterns

| Pattern | Examples |
|---------|---------|
| **Sort then scan** | Activity selection, fractional knapsack, job sequencing |
| **Priority queue** | Huffman coding, Dijkstra, Prim |
| **Sweep line** | Minimum platforms, interval scheduling |
| **Exchange argument** | Proving any greedy correct |

### Common pitfalls

- **Assuming greedy works without proof** -- the most dangerous mistake; always verify the greedy choice property
- **Wrong sorting criterion** -- activity selection by start time or duration fails; only finish time works
- **Confusing fractional and 0/1** -- fractional knapsack is greedy; 0/1 knapsack is DP
- **Ignoring edge cases** -- empty input, ties in sorting, all items identical
- **Negative weights** -- Dijkstra fails with negative edges; Kruskal/Prim handle them fine

### Interview strategy

1. Identify if the problem has greedy choice property
2. State the greedy criterion explicitly
3. Argue correctness (exchange argument or matroid)
4. Code the sort + scan / priority queue
5. State time and space complexity

---

## Slide 20 -- Summary & Further Reading

### Key takeaways

- Greedy algorithms make locally optimal choices and never backtrack -- simple and fast when correct
- Correctness requires the greedy choice property and optimal substructure -- prove it or switch to DP
- Activity selection, Huffman coding, and fractional knapsack are canonical greedy problems
- Kruskal's, Prim's, and Dijkstra's are greedy graph algorithms justified by the cut property or relaxation
- Exchange arguments are the standard proof technique -- show any OPT can be transformed to match greedy
- Matroid theory gives a complete characterisation of when greedy works
- When greedy fails (0/1 knapsack, non-canonical coins), it often still provides a useful approximation

### Recommended reading

| Source | Description |
|--------|------------|
| **CLRS** | *Introduction to Algorithms* -- Ch. 15-16: greedy algorithms and matroid theory |
| **Kleinberg & Tardos** | *Algorithm Design* -- Ch. 4: greedy algorithms, exchange arguments |
| **Erickson** | *Algorithms* -- [jeffe.cs.illinois.edu/teaching/algorithms](http://jeffe.cs.illinois.edu/teaching/algorithms/) -- free textbook, excellent greedy chapter |
| **MIT 6.046J** | *Design and Analysis of Algorithms* -- greedy algorithms lectures on MIT OCW |
| **Oxley** | *Matroid Theory*, 2nd ed. -- the definitive matroid reference |
