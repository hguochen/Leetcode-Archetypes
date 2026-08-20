# 🛣️ Archetype 14 — Weighted Shortest Path

> ### 📇 QUICK REFERENCE CARD
>
> ⏭️ **FIRST READ? SKIP THIS — jump to §0.**
> This card compresses §3–§9. Reading it now will feel like understanding
> before it is understanding.
>
> ✅ **USE IT WHEN:** reviewing a solved archetype · warming up before a mock ·
> mid-problem when you've lost the frame.

```
WHEN TO USE:
- "minimum COST / price / weight / effort / time" and edges carry numbers
- "cheapest", "least total", "smallest maximum", "highest probability"
- DISQUALIFIER: every edge costs the same → archetype 13, plain BFS
- DISQUALIFIER: "how many paths", "longest path" → DP, archetype 23+

CORE FRAME (Dijkstra):
  dist[src] = 0;  pq.add({src, 0});
  while (!pq.isEmpty()) {
      {u, d} = pq.poll();
      if (d > dist[u]) continue;              // stale entry, skip
      for ({v, w} : adj[u])
          if (d + w < dist[v]) {              // RELAX
              dist[v] = d + w;
              pq.add({v, dist[v]});
          }
  }

THE KNOBS:
A COST COMBINE   → sum | max | min | product      how an edge joins a path
B STATE          → node | node + carried resource
C EXTRACTION     → min-heap | max-heap | layered (Bellman-Ford) | triple loop (Floyd-Warshall)

PATTERNS AS KNOB SETTINGS:
1. Dijkstra            = A:sum      B:node      C:min-heap
2. Non-additive cost   = A:max/min/product      C:min- or MAX-heap
3. State-augmented     = B:node + resource      C:min-heap
4. Bounded / all-pairs = C:layered  or  triple loop

TEMPLATE SELECTOR:
- "cheapest path, weights >= 0"        → Pattern 1
- "minimise the largest edge"          → Pattern 2, combine with max
- "maximise the smallest edge"         → Pattern 2, combine with min + MAX-heap
- "maximise a product of probabilities"→ Pattern 2, product + MAX-heap
- "at most K stops / within T time"    → Pattern 3
- "negative weights" or "exactly K edges" → Pattern 4, Bellman-Ford
- "distance between EVERY pair"        → Pattern 4, Floyd-Warshall

TIME / SPACE:
- Dijkstra:        O((V + E) log V) time, O(V + E) space
- Bellman-Ford:    O(V * E) time, O(V) space
- Floyd-Warshall:  O(V^3) time, O(V^2) space   (fine to about V = 400)

TOP 3 TRICKS:
1. The heap holds (node, dist) PAIRS, never bare nodes. Java has no decrease-key,
   so you push duplicates and discard stale ones on pop.
2. To MAXIMISE, negate the key or supply a reversed comparator — the relaxation
   test flips from < to >. Nothing else changes.
3. Dijkstra's greedy step only needs the cost function to be MONOTONE, not additive.
   That is why max, min and product all work.

TOP 3 PITFALLS:
1. Forgetting `if (d > dist[u]) continue;` → every stale heap entry re-expands.
   Correct answer, quadratic blow-up.
2. Comparator written as `a[1] - b[1]` → integer overflow on large weights.
   Use Integer.compare.
3. Running Dijkstra with negative weights → confident wrong answer. The greedy
   argument dies; use Bellman-Ford.
```

* * *

# 0. Goal & Readiness Contract

> 🎯 **Objective:** know exactly what "done" means before you read anything else.
> 🔗 **Builds on:** nothing.
> ⏱️ **Session 1**

## Where you'll be when you finish

You will read a problem, decide within **30 seconds** whether it is weighted
shortest path (and not plain BFS, not DP, not Union-Find), name which of the four
patterns applies and its knob settings within another **30 seconds**, and have
correct Java on screen inside **35 minutes** — with this document closed.

## The problem class this solves

* Problems asking for a **minimum total cost** between two points, where edges
  carry different weights
* Problems where the "cost" of a path is some monotone combination of its edges —
  a sum, a maximum, a minimum, or a product
* Problems adding a **budget** to that: at most K stops, within T minutes
* Problems asking for **all-pairs** distances on a small graph
* Key invariant: **once the cheapest unfinalised node is extracted, its distance is
  final.** Everything in this archetype is a consequence of that one fact.

## Prerequisites

| Archetype | What you need from it |
|---|---|
| 13 · BFS Shortest Path | The ordering theorem in its §2, and why it needs "every edge costs 1". This archetype is the repair. |
| 12 · DFS Flood Fill | Grid-as-graph, the `DIRS` array |

> 💡 **Read §2 of `archetypes/13_bfs_shortest_path.md` before starting.** It proves
> that BFS's queue holds only distances `d` and `d+1`, so first-arrival is shortest
> — and that proof uses "every edge costs 1" exactly once. Archetype 13's last
> problem, `#2290`, walks you to the precise point where it fails. You already know
> what Dijkstra is fixing; this archetype just shows you the fix.

## Exit criteria — you are done when (full detail in §11)

```
□ Write the Dijkstra frame cold, from a blank file, in < 4 min, zero bugs
□ Score 8/8 on the recognition test in < 90 sec
□ Name the knob settings for any in-archetype problem in < 30 sec
□ Solve one UNSEEN Medium cold in < 35 min, no doc, no hints
```

## Study path

```
SESSION 1  §0 → §4   then 🔓 GATE 1  · #743
SESSION 2  §5 → §6   then 🔓 GATE 2  · #787, #1631, #1514
SESSION 3  §7 → §8   then 🔓 GATE 3  · re-attempt your failures
SESSION 4  §9 → §11  then ✅ READINESS GATE
```

* * *

# 1. The Motivating Problem

> 🎯 **Objective:** feel the specific pain this archetype removes.
> 🔗 **Builds on:** nothing.
> ⏱️ **Session 1**

**The problem — #743 Network Delay Time.** A signal starts at node `k` and travels
along directed edges. `times[i] = [u, v, w]` means the signal takes `w` time to go
from `u` to `v`. Return the time for **all** `n` nodes to receive it, or `-1`.

```
        1
   2 ───────▶ 1
   │
   │ 1
   ▼
   3 ───────▶ 4
        1
```

**Your first instinct, honestly.** You just finished archetype 13. This is a graph,
it asks for a shortest path, so — BFS.

```java
// ❌ The instinct. It answers a different question.
public int networkDelayTime(int[][] times, int n, int k) {
    List<int[]>[] adj = new List[n + 1];
    for (int i = 1; i <= n; i++) adj[i] = new ArrayList<>();
    for (int[] t : times) adj[t[0]].add(new int[]{t[1], t[2]});

    int[] dist = new int[n + 1];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[k] = 0;

    Deque<Integer> q = new ArrayDeque<>();
    q.add(k);
    while (!q.isEmpty()) {                       // plain FIFO — archetype 13
        int u = q.poll();
        for (int[] e : adj[u]) {
            int v = e[0], w = e[1];
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                q.add(v);                        // no ordering by cost
            }
        }
    }
    int ans = 0;
    for (int i = 1; i <= n; i++) {
        if (dist[i] == Integer.MAX_VALUE) return -1;
        ans = Math.max(ans, dist[i]);
    }
    return ans;
}
```

**Where it dies.** Not where you expect. This actually produces the *right answer* —
it is Bellman-Ford in disguise, and we will meet it properly in §5. What it does not
do is finish quickly. A FIFO queue has no idea which node is cheapest, so it
re-relaxes nodes over and over as better routes trickle in.

Consider a graph shaped to punish it:

```
node 0 ──1──▶ 1 ──1──▶ 2 ──1──▶ ... ──1──▶ n-1        the cheap chain
   └──100──▶ 1 ──100──▶ 2 ──100──▶ ...                the expensive chain

Every node is first reached by an expensive route, then corrected by a cheaper
one, then corrected again — and each correction re-enqueues everything downstream.
```

The deeper problem is what BFS's ordering guarantee has become:

```
BFS on unweighted:  nodes leave the queue in order of DISTANCE   → first pop is final
BFS on weighted:    nodes leave the queue in order of HOP COUNT  → first pop means nothing
```

You are still popping in hop order, but you are measuring in cost. **The queue is
sorted by the wrong thing.**

> ✅ **Checkpoint:** state in one sentence what archetype 13's queue guaranteed, and
> which word in that guarantee stops being true here.
> Fail → reread §2 of `archetypes/13_bfs_shortest_path.md`.
>
> <sub>Answer: it guaranteed nodes are dequeued in non-decreasing DISTANCE order,
> because draining distance d appends only d+1. With varying weights, a one-hop
> neighbour can be more expensive than a five-hop one, so dequeue order no longer
> tracks cost.</sub>

* * *

# 2. Brute Force → The Core Insight

> 🎯 **Objective:** derive the fix yourself, rather than be told it.
> 🔗 **Builds on:** §1 (the ordering guarantee you just watched break).
> ⏱️ **Session 1**

## Step 1 — Brute force, precisely

* Enumerate every simple path from source to target and keep the cheapest.
* Time **O(V!)** in the worst case, Space **O(V)** for the recursion.
* Worth one sentence in an interview, then move on. It is the same trap as
  archetype 13's §1: correct, unaffordable, and it re-derives a value that is a
  property of the *node*, not of the path.

## Step 2 — The redundancy

The FIFO version above already fixes the exponential blow-up — it stores one
`dist[v]` per node instead of enumerating paths. What it wastes is **work spent on
nodes whose answer is not yet knowable.**

```
pop a node whose dist is still provisional  →  every edge you relax from it
                                               may need redoing later
```

## Step 3 — The shift

```
from: pop nodes in arrival order, and fix them up as better routes appear
to:   always pop the CHEAPEST unfinalised node, because its distance can
      never improve again
```

## Step 4 — Why the cheapest node is already final

This is the whole archetype, so do not skim it.

```
Claim:  if all weights are >= 0, and u is the unfinalised node with the smallest
        tentative dist, then dist[u] is FINAL.

Why:    suppose some cheaper route to u existed. It must leave the finalised set
        at some point, through a node x that is still unfinalised.
        That route's cost is at least dist[x], because reaching x costs at least
        dist[x] and the remaining edges are >= 0.
        But u was chosen as the SMALLEST unfinalised, so dist[u] <= dist[x].
        So the "cheaper" route costs at least dist[u]. Contradiction.
```

Notice what the proof leans on: **edges are non-negative**, so the remainder of a
path can never reduce its cost. That single assumption is the whole boundary of
Dijkstra.

* Key idea: replace the FIFO queue with a **priority queue keyed on cost**.
* Time **O((V + E) log V)** — each edge causes at most one push, each push costs
  `log`.
* Space **O(V + E)** — the adjacency list plus the heap.

> ⚠️ **Where this breaks — memorise it, it is the archetype's own boundary.** Allow
> one negative edge and "the remaining edges cannot reduce the cost" fails. A node
> popped as cheapest can still be improved later, and Dijkstra returns a confident
> wrong answer. That is what Bellman-Ford exists for — §5, Pattern 4.

> ✅ **Checkpoint:** say the "from → to" line out loud without looking, then state
> the one assumption the proof depends on.
> Fail → reread §1.

* * *

# 3. The Core Frame

> 🎯 **Objective:** write Dijkstra from memory and explain why each line is
> load-bearing.
> 🔗 **Builds on:** §2 (the greedy-extraction proof this code implements).
> ⏱️ **Session 1**

## Core ideas

* Nodes are **finalised in cost order**, cheapest first. Archetype 13 finalised them
  in hop order; that is the only change.
* `dist[]` is not a visited flag. It is a **best-known-so-far**, and it only ever
  decreases.
* What you control: how an edge combines with a path (§4 Knob A), what counts as a
  node (Knob B), and what discipline extracts the next one (Knob C).

## The contract

```java
// adjacency list: adj[u] holds {neighbour, weight} pairs
List<int[]>[] adj = new List[n];
for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
for (int[] e : edges) {
    adj[e[0]].add(new int[]{e[1], e[2]});
    adj[e[1]].add(new int[]{e[0], e[2]});   // omit this line for a DIRECTED graph
}
```

## The frame

```java
// Dijkstra. Solves ~80% of this archetype.
int[] dijkstra(List<int[]>[] adj, int n, int src) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    // (1) the heap holds (node, dist) PAIRS, ordered by dist
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
    pq.add(new int[]{src, 0});

    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        int u = cur[0], d = cur[1];

        // (2) stale entry: a better route to u was found after this was pushed
        if (d > dist[u]) continue;

        for (int[] e : adj[u]) {
            int v = e[0], w = e[1];
            // (3) RELAX: strictly cheaper, or do nothing
            if (d + w < dist[v]) {
                dist[v] = d + w;
                pq.add(new int[]{v, dist[v]});   // (4) push a duplicate, never update
            }
        }
    }
    return dist;
}
```

## Why each line is load-bearing

| # | Line | Remove or change it and... |
|---|---|---|
| 1 | heap holds **pairs**, not bare nodes | With bare nodes the comparator would read a `dist[]` that changes underneath the heap, corrupting its internal order. Java's `PriorityQueue` does not re-heapify on external mutation. |
| 2 | `if (d > dist[u]) continue;` | Every superseded push re-expands its whole neighbourhood. Still the right answer, but the heap can grow to `O(E)` and you do the work many times over. This is the most common Dijkstra bug. |
| 3 | strict `<` in the relax test | With `<=` you re-push on ties forever on any zero-weight edge, and the loop may not terminate. |
| 4 | push a duplicate rather than update | Java has no `decrease-key`. Lazy deletion (push a new pair, discard stale pops) is the standard workaround, and it is why line 2 exists. |

## Visualization

### Finalisation order is cost order, not hop order

```
        1        1        1
   S ───────▶ A ───────▶ B ───────▶ T          the long cheap way (cost 3)
   │
   └──────────────10──────────────▶ T          the short expensive way (cost 10)

pop order:  S(0)  A(1)  B(2)  T(3)
            T is pushed twice — once at 10 via the direct edge, once at 3 via B.
            The 3 pops first; when the 10 pops later, d(10) > dist[T](3) → skipped.
            That skip is line (2) earning its place.
```

### Why the heap holds pairs

```
push (T,10)                    heap: [(T,10)]
relax T via B, dist[T] = 3
push (T,3)                     heap: [(T,3), (T,10)]
pop  (T,3)   →  d == dist[T]   process it
pop  (T,10)  →  d >  dist[T]   STALE, discard

If the heap stored bare nodes, T would appear once and its priority would have
silently changed after insertion — the heap's ordering invariant is then broken.
```

> ✅ **Checkpoint:** close the doc and write the frame from a blank file. It must
> compile. Then check yourself against the four numbered lines — if you got (1) and
> (2) right, move on.
> Fail → reread §2. If the greedy-extraction proof is not yours, the code is shapes.

* * *

# 4. The Knobs

> 🎯 **Objective:** replace "memorise four algorithms" with "one frame + three
> dials", and read any problem as a knob setting.
> 🔗 **Builds on:** §3 (the frame the knobs turn).
> ⏱️ **Session 1**

Every pattern in §5 is the §3 frame with one knob turned. The `while` loop, the
stale check and the relaxation shape are identical in all four.

```
┌─────────────────┬──────────────────────────────┬────────────────────────────────────┐
│ KNOB            │ QUESTION IT ANSWERS          │ SETTINGS                           │
├─────────────────┼──────────────────────────────┼────────────────────────────────────┤
│ A COST COMBINE  │ how does an edge join a      │ sum      d + w      (default)      │
│                 │ path to make a new cost?     │ max      max(d, w)  minimax        │
│                 │                              │ min      min(d, w)  maximin        │
│                 │                              │ product  d * w      probabilities  │
├─────────────────┼──────────────────────────────┼────────────────────────────────────┤
│ B STATE         │ what makes two arrivals      │ node                               │
│                 │ "the same"?                  │ node + carried resource            │
├─────────────────┼──────────────────────────────┼────────────────────────────────────┤
│ C EXTRACTION    │ how is the next node chosen? │ min-heap    Dijkstra, minimising   │
│                 │                              │ max-heap    Dijkstra, maximising   │
│                 │                              │ layered     Bellman-Ford           │
│                 │                              │ triple loop Floyd-Warshall         │
└─────────────────┴──────────────────────────────┴────────────────────────────────────┘

Pattern 1  Dijkstra            = A:sum        B:node       C:min-heap
Pattern 2  Non-additive cost   = A:max/min/product         C:min- or MAX-heap
Pattern 3  State-augmented     = A:sum        B:node+res   C:min-heap
Pattern 4  Bounded / all-pairs = A:sum        B:node       C:layered | triple loop
```

> 📎 **Neighbours — explicit or implicit — is still a knob, but it is archetype
> 13's.** Whether `adj[u]` is a stored list or generated from a `DIRS` array or from
> a rule changes nothing here. That is why it is not one of these three.

## Knob A is the surprising one

Dijkstra is usually taught as "sum of weights". It is not. The greedy proof in §2
needs only that combining an edge with a path **never makes the path cheaper**:

```
sum      d + w          with w >= 0, never decreases      ✅
max      max(d, w)      never decreases                    ✅
min      min(d, w)      never INcreases — so MAXIMISE it   ✅ (flip the heap)
product  d * w          with 0 <= w <= 1, never increases  ✅ (flip the heap)
```

Any monotone combiner works. That is why `#1631` (minimise the largest step),
`#778` (minimise the highest water level), `#1514` (maximise a product) and `#2812`
(maximise the smallest safeness) are all the same algorithm.

## Knob B is archetype 13's Knob B, unchanged

Same three questions:

```
1. carried by the traveller?       stops used, time spent, discounts left
2. differs at the same node?
3. can a costlier arrival be BETTER because of it?
```

All three yes → the node is `(vertex, resource)`. `#1928` and `#2577` are exactly
`#1293`'s state augmentation with weights bolted on.

## Worked example — reading a problem as knob settings

**#1631 Path With Minimum Effort.** *A route's effort is the maximum absolute
difference in height between consecutive cells. Minimise it.*

| Knob | Sentence that decides it | Setting |
|---|---|---|
| **A** | "effort is the **maximum** absolute difference" — not the total | **max**, so `nd = Math.max(d, abs(h[nr][nc] - h[r][c]))` |
| **B** | You carry nothing — no budget, no counter | **node** |
| **C** | Minimising, weights non-negative | **min-heap** |

`A:max B:node C:min-heap` → **Pattern 2**, and the only line that differs from §3
is the combiner.

> ✅ **Checkpoint:** **#1514 Path with Maximum Probability** — *edges carry success
> probabilities; find the path maximising the product.* Name its three knob settings
> in under 30 seconds, before reading §5.
> Fail → reread §3.
>
> <sub>Answer: A:product. B:node — nothing is carried. C:MAX-heap, because you are
> maximising and the combiner decreases. → Pattern 2.</sub>

* * *

## 🔓 ATTEMPT GATE 1 — STOP READING

> You have the frame (§3) and the knobs (§4). That is everything needed for plain
> Dijkstra. Go prove it.

```
→ #743 Network Delay Time                       Medium

Rules: 25 min timer · no hints · no editorial · this doc CLOSED
       Nodes are 1-indexed — size your arrays n+1 or subtract 1 consistently.
       The answer is the MAXIMUM over all dist[], and -1 if any node is
       unreachable. Do not forget the unreachable check.
```

Come back either way, and log the outcome:

```
□ Solved clean          → continue to §5
□ Solved with bugs      → write the bug down in one line, here:
                          ______________________________________
                          It will appear by name in §7. Continue to §5.
□ Stuck > 10 min        → reread §3. The frame didn't land.
                          Do NOT read ahead; §5 is four variations on a
                          template you don't yet have.
```

* * *

# 5. Pattern Catalog

> 🎯 **Objective:** for each pattern, name the knob it turns and write it by
> modifying the §3 frame — not from memory.
> 🔗 **Builds on:** §3 (frame) · §4 (knobs).
> ⏱️ **Session 2**

All four are the same function from §3. Find what is *identical* first, then what
moved.

## Pattern 1 — Dijkstra

🔧 **Delta from §3:** none. This *is* the frame, plus whatever the problem wants
read out of `dist[]` at the end.

**When to use:** non-negative weights, minimise a sum, single source.

**Mental model:** finalise nodes cheapest-first; the first pop of a node is final.

```java
// #743 Network Delay Time — Pattern 1, unmodified frame.
public int networkDelayTime(int[][] times, int n, int k) {
    List<int[]>[] adj = new List[n + 1];
    for (int i = 1; i <= n; i++) adj[i] = new ArrayList<>();
    for (int[] t : times) adj[t[0]].add(new int[]{t[1], t[2]});

    int[] dist = new int[n + 1];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[k] = 0;

    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
    pq.add(new int[]{k, 0});

    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        int u = cur[0], d = cur[1];
        if (d > dist[u]) continue;
        for (int[] e : adj[u]) {
            int v = e[0], w = e[1];
            if (d + w < dist[v]) {
                dist[v] = d + w;
                pq.add(new int[]{v, dist[v]});
            }
        }
    }
    // the signal arrives when the LAST node hears it
    int ans = 0;
    for (int i = 1; i <= n; i++) {
        if (dist[i] == Integer.MAX_VALUE) return -1;
        ans = Math.max(ans, dist[i]);
    }
    return ans;
}
```

**Why it works:** §2's proof — the smallest unfinalised tentative distance cannot be
improved, because any alternative route must exit the finalised set through a node
that already costs at least as much.

**Problems:** #743, #1786, #2045, #2662

* * *

## Pattern 2 — Non-additive cost

🔧 **Delta from §3:** **KNOB A only** — one line, where the new cost is computed.
If the combiner *decreases* rather than increases, also flip the heap and the
comparison. Everything else is byte-identical.

**When to use:**
* "minimise the **maximum**" — bottleneck, effort, highest water level
* "maximise the **minimum**" — safeness, capacity, widest path
* "maximise a **product**" — probabilities, exchange rates

**Mental model:** Dijkstra never required addition. It requires that extending a
path moves the cost monotonically in one direction.

```java
// #1631 Path With Minimum Effort — Pattern 2, combiner = max.
public int minimumEffortPath(int[][] heights) {
    int m = heights.length, n = heights[0].length;
    int[][] DIRS = {{-1,0},{1,0},{0,-1},{0,1}};

    int[][] effort = new int[m][n];
    for (int[] row : effort) Arrays.fill(row, Integer.MAX_VALUE);
    effort[0][0] = 0;

    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[2], b[2]));
    pq.add(new int[]{0, 0, 0});

    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        int r = cur[0], c = cur[1], d = cur[2];
        if (d > effort[r][c]) continue;
        if (r == m - 1 && c == n - 1) return d;

        for (int[] dir : DIRS) {
            int nr = r + dir[0], nc = c + dir[1];
            if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
            // ⬅️ KNOB A: max instead of sum. This ONE line is the whole pattern.
            int nd = Math.max(d, Math.abs(heights[nr][nc] - heights[r][c]));
            if (nd < effort[nr][nc]) {
                effort[nr][nc] = nd;
                pq.add(new int[]{nr, nc, nd});
            }
        }
    }
    return 0;   // 1x1 grid: no moves, zero effort
}
```

And the maximising form — note the three coordinated flips:

```java
// #1514 Path with Maximum Probability — combiner = product, so MAXIMISE.
public double maxProbability(int n, int[][] edges, double[] succProb,
                             int start, int end) {
    List<double[]>[] adj = new List[n];
    for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
    for (int i = 0; i < edges.length; i++) {
        adj[edges[i][0]].add(new double[]{edges[i][1], succProb[i]});
        adj[edges[i][1]].add(new double[]{edges[i][0], succProb[i]});
    }

    double[] best = new double[n];
    best[start] = 1.0;
    // ⬅️ flip 1: MAX-heap
    PriorityQueue<double[]> pq = new PriorityQueue<>((a, b) -> Double.compare(b[1], a[1]));
    pq.add(new double[]{start, 1.0});

    while (!pq.isEmpty()) {
        double[] cur = pq.poll();
        int u = (int) cur[0];
        double p = cur[1];
        if (p < best[u]) continue;               // ⬅️ flip 2: stale test reverses
        if (u == end) return p;
        for (double[] e : adj[u]) {
            int v = (int) e[0];
            double np = p * e[1];                // ⬅️ KNOB A: product
            if (np > best[v]) {                  // ⬅️ flip 3: relax reverses
                best[v] = np;
                pq.add(new double[]{v, np});
            }
        }
    }
    return 0.0;
}
```

**Why it works:** with `0 <= w <= 1` the product never grows, so the *largest*
unfinalised probability is final by the mirror image of §2's argument.

**Problems:** #1631, #1514, #2812 ⭐, #778

* * *

## Pattern 3 — State-augmented Dijkstra

🔧 **Delta from §3:** **KNOB B only.** `dist` gains a dimension, the heap entry
carries the resource, and there is a new affordability guard. The loop is untouched.

**When to use:** a budget rides along — "at most K stops", "within `maxTime`
minutes", "up to `discounts` free roads".

**Mental model:** the node was never the vertex. It was `(vertex, resource)` — in
Patterns 1 and 2 the resource was constant, so it factored out.

```java
// #1928 Minimum Cost to Reach Destination in Time — Pattern 3.
public int minCost(int maxTime, int[][] edges, int[] passingFees) {
    int n = passingFees.length;
    List<int[]>[] adj = new List[n];
    for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
    for (int[] e : edges) {
        adj[e[0]].add(new int[]{e[1], e[2]});
        adj[e[1]].add(new int[]{e[0], e[2]});
    }

    // ⬅️ KNOB B: the second index IS the pattern
    int[][] best = new int[n][maxTime + 1];
    for (int[] row : best) Arrays.fill(row, Integer.MAX_VALUE);
    best[0][0] = passingFees[0];

    // ordered by COST; time rides along as state, not as the key
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
    pq.add(new int[]{0, passingFees[0], 0});     // {node, cost, time}

    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        int u = cur[0], cost = cur[1], time = cur[2];
        if (cost > best[u][time]) continue;
        if (u == n - 1) return cost;

        for (int[] e : adj[u]) {
            int v = e[0], t = time + e[1];
            if (t > maxTime) continue;           // ⬅️ can't afford the time
            int nc = cost + passingFees[v];
            if (nc < best[v][t]) {
                best[v][t] = nc;
                pq.add(new int[]{v, nc, t});
            }
        }
    }
    return -1;
}
```

**Why it works:** §2's proof is about *nodes*, and a node here is a `(vertex, time)`
pair. Collapsing the pair would claim two genuinely different states are one — the
same error as a 2-D `visited` in archetype 13's `#1293`.

**Cost:** time and space multiply by the resource's range — `O(V * T)` states.

**Problems:** #787 (its hop bound is a resource), #1928, #2577

* * *

## Pattern 4 — Bounded relaxation & all-pairs

🔧 **Delta from §3:** **KNOB C only.** The heap disappears. Either relax every edge
in rounds (Bellman-Ford), or relax through every intermediate vertex
(Floyd-Warshall).

**When to use:**
* **negative weights** — Dijkstra's greedy proof is dead, use Bellman-Ford
* **exactly / at most K edges** — round `i` of Bellman-Ford is "paths of ≤ i edges"
* **all-pairs distances**, small `V` — Floyd-Warshall

**Mental model (Bellman-Ford):** after `i` rounds, `dist[v]` is the cheapest route
using at most `i` edges. The rounds *are* the hop count, which is why a hop bound
falls out for free.

```java
// #787 Cheapest Flights Within K Stops — Bellman-Ford, k+1 rounds.
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    for (int round = 0; round <= k; round++) {
        // ⬅️ the SNAPSHOT is the pattern: relax from the previous round only,
        //    so each round adds exactly one more edge to every path
        int[] next = dist.clone();
        for (int[] f : flights) {
            int u = f[0], v = f[1], w = f[2];
            if (dist[u] == Integer.MAX_VALUE) continue;
            if (dist[u] + w < next[v]) next[v] = dist[u] + w;
        }
        dist = next;
    }
    return dist[dst] == Integer.MAX_VALUE ? -1 : dist[dst];
}
```

```java
// #1334 Find the City With the Smallest Number of Neighbors — Floyd-Warshall.
public int findTheCity(int n, int[][] edges, int distanceThreshold) {
    int INF = 1_000_000_000;
    int[][] d = new int[n][n];
    for (int[] row : d) Arrays.fill(row, INF);
    for (int i = 0; i < n; i++) d[i][i] = 0;
    for (int[] e : edges) {
        d[e[0]][e[1]] = Math.min(d[e[0]][e[1]], e[2]);
        d[e[1]][e[0]] = Math.min(d[e[1]][e[0]], e[2]);
    }

    // ⬅️ k MUST be the OUTER loop: "paths allowed to pass through 0..k"
    for (int mid = 0; mid < n; mid++)
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                if (d[i][mid] + d[mid][j] < d[i][j])
                    d[i][j] = d[i][mid] + d[mid][j];

    int best = -1, bestCount = n + 1;
    for (int i = 0; i < n; i++) {
        int c = 0;
        for (int j = 0; j < n; j++)
            if (i != j && d[i][j] <= distanceThreshold) c++;
        if (c <= bestCount) { bestCount = c; best = i; }   // ties → larger index
    }
    return best;
}
```

**Why the snapshot matters in Bellman-Ford:** relaxing in place lets a path grow by
two edges inside one round, so "at most k+1 edges" stops being true. The `clone()`
is not defensive copying — it is the hop bound.

**Problems:** #787, #1334, #1462, #2976, #399

> ✅ **Checkpoint:** cover the code. For one problem per pattern, name the knob
> settings in under 10 seconds each. Then answer: which knob separates `#1631` from
> `#743`? (A.) Which separates `#1928` from `#743`? (B.)
> Fail → reread §4, not §5.

* * *

# 6. Composition — Flagship Problem: Find the Safest Path in a Grid (#2812)

> 🎯 **Objective:** see two archetypes chained, so composite problems stop looking
> like new ones.
> 🔗 **Builds on:** §5 Pattern 2 · archetype 13 Pattern 2 (multi-source BFS).
> ⏱️ **Session 2**

**Problem:** an `n x n` grid contains thieves (`1`). The *safeness* of a path is the
minimum Manhattan distance from any cell on it to any thief. Return the maximum
safeness achievable from `(0,0)` to `(n-1,n-1)`.

**Why this one:** it is two algorithms with a handoff, and the second one's input is
manufactured by the first — the exact shape of archetype 13's flagship `#934`. It
also needs *both* directions of Knob A in one problem.

```
Step 1: multi-source BFS from every thief → safeness[r][c]   [archetype 13, P2]
Step 2: max-min Dijkstra over that field                     [archetype 14, P2]
```

```java
class Solution {
    private static final int[][] DIRS = {{-1,0},{1,0},{0,-1},{0,1}};

    public int maximumSafenessFactor(List<List<Integer>> grid) {
        int n = grid.size();

        // ── Phase 1 (archetype 13): distance from every cell to the nearest thief
        int[][] dist = new int[n][n];
        for (int[] row : dist) Arrays.fill(row, -1);
        Deque<int[]> q = new ArrayDeque<>();
        for (int r = 0; r < n; r++)
            for (int c = 0; c < n; c++)
                if (grid.get(r).get(c) == 1) { dist[r][c] = 0; q.add(new int[]{r, c}); }

        while (!q.isEmpty()) {                       // multi-source BFS
            int[] cur = q.poll();
            for (int[] d : DIRS) {
                int nr = cur[0] + d[0], nc = cur[1] + d[1];
                if (nr < 0 || nr >= n || nc < 0 || nc >= n || dist[nr][nc] != -1) continue;
                dist[nr][nc] = dist[cur[0]][cur[1]] + 1;
                q.add(new int[]{nr, nc});
            }
        }

        // ── Phase 2 (archetype 14): maximise the MINIMUM safeness along a path
        int[][] best = new int[n][n];
        for (int[] row : best) Arrays.fill(row, -1);
        best[0][0] = dist[0][0];

        PriorityQueue<int[]> pq =
            new PriorityQueue<>((a, b) -> Integer.compare(b[2], a[2]));   // MAX-heap
        pq.add(new int[]{0, 0, dist[0][0]});

        while (!pq.isEmpty()) {
            int[] cur = pq.poll();
            int r = cur[0], c = cur[1], v = cur[2];
            if (v < best[r][c]) continue;                       // stale, reversed
            if (r == n - 1 && c == n - 1) return v;

            for (int[] d : DIRS) {
                int nr = r + d[0], nc = c + d[1];
                if (nr < 0 || nr >= n || nc < 0 || nc >= n) continue;
                int nv = Math.min(v, dist[nr][nc]);             // KNOB A: min
                if (nv > best[nr][nc]) {                        // maximise it
                    best[nr][nc] = nv;
                    pq.add(new int[]{nr, nc, nv});
                }
            }
        }
        return 0;
    }
}
```

**The non-obvious part.** Two different optimisations point in opposite directions
inside one problem, and it is easy to conflate them:

```
phase 1   MINIMISE   distance from each cell to the nearest thief   (BFS)
phase 2   MAXIMISE   the MINIMUM of those values along a path       (max-heap)
```

The instinct is to run a single search that "avoids thieves". That does not
work — safeness is a property of the *whole path's weakest point*, so the field has
to exist in full before the path search can start. The BFS output is the second
algorithm's edge weights.

**Why `min` and not `sum`:** the path is only as safe as its most exposed cell.
Summing would reward long detours through mildly-safe territory.

**Space:** `O(n²)` — the distance field, the best-so-far field, and the heap, which
can hold a whole layer.

> ✅ **Checkpoint:** explain in 60 seconds which phase produces the weights and which
> consumes them, and why they cannot be merged into one pass.
> Fail → reread §5 Pattern 2.

* * *

## 🔓 ATTEMPT GATE 2 — STOP READING

> You have all four patterns and one composition. This gate tests whether you can
> *derive* a variant rather than recall one.

```
→ #787  Cheapest Flights Within K Stops        Medium   ← Pattern 4 (or 3)
→ #1631 Path With Minimum Effort               Medium   ← Pattern 2, max
→ #1514 Path with Maximum Probability          Medium   ← Pattern 2, product + max-heap

Rules: 25 min each · no hints · no editorial · this doc CLOSED
       Before coding each one, write its three knob settings on paper.
       If you can't, that is the failure — not the code.
```

```
□ 3/3 clean             → continue to §7
□ Solved with bugs      → write each bug down in one line, here:
                          ______________________________________
                          ______________________________________
                          §7 exists to name them. Continue to §7.
□ Couldn't name the     → reread §4. That is a knob-reading failure, not a
  knobs before coding     coding failure, and §7 will not fix it.
```

* * *

# 7. Failure Modes

> 🎯 **Objective:** name the bug you produced at Gates 1–2, and know the input that
> catches each one.
> 🔗 **Builds on:** 🔓 Gates 1–2 (your own bugs).
> ⏱️ **Session 3**

> 📌 **Read this having ALREADY attempted Gates 1–2.** Cold, it is an inert list of
> warnings about mistakes you have not made. Warm, it is a diagnosis of code you
> wrote an hour ago.

## Failure 1 — Missing the stale-entry check

```java
// ❌ WRONG
while (!pq.isEmpty()) {
    int[] cur = pq.poll();
    int u = cur[0], d = cur[1];
    for (int[] e : adj[u]) { ... }        // no `if (d > dist[u]) continue;`
}
```

💥 **Symptom:** the right answer, arriving slowly. Every superseded push re-expands
its whole neighbourhood, and the effect compounds on dense graphs. TLE, not a wrong
answer — which is exactly why it survives your small tests.

```java
// ✅ CORRECT
if (d > dist[u]) continue;
```

🧪 **Catch it with:** a dense graph where many cheap routes are found late — e.g.
`n = 500` nodes with an edge between every pair. Correct code finishes instantly.

* * *

## Failure 2 — Comparator by subtraction

```java
// ❌ WRONG
new PriorityQueue<int[]>((a, b) -> a[1] - b[1]);
```

💥 **Symptom:** silently inverted ordering when the subtraction overflows. With
weights near `Integer.MAX_VALUE` — or an `INF` sentinel that leaks into the heap —
`a[1] - b[1]` wraps negative and the heap returns the *largest* element.

```java
// ✅ CORRECT
new PriorityQueue<int[]>((a, b) -> Integer.compare(a[1], b[1]));
```

🧪 **Catch it with:** two entries with costs `2_000_000_000` and `-2_000_000_000`.
The subtraction overflows; `Integer.compare` does not.

* * *

## Failure 3 — Running Dijkstra on negative weights

```java
// ❌ WRONG — a graph with any negative edge
if (d + w < dist[v]) { dist[v] = d + w; pq.add(...); }
```

💥 **Symptom:** a confident wrong answer, no crash, no slowness. §2's proof needs
"the remaining edges cannot reduce the cost". One negative edge and a node popped as
cheapest can still be improved later — but it has already been finalised.

```java
// ✅ CORRECT — Bellman-Ford, V-1 rounds over every edge
for (int i = 0; i < n - 1; i++)
    for (int[] e : edges)
        if (dist[e[0]] != INF && dist[e[0]] + e[2] < dist[e[1]])
            dist[e[1]] = dist[e[0]] + e[2];
```

🧪 **Catch it with:** `0 →(2)→ 1`, `0 →(5)→ 2`, `1 →(-4)→ 2`. True `dist[2]` is
`-2`; Dijkstra finalises `2` at `5`.

* * *

## Failure 4 — Relaxing in place inside Bellman-Ford's hop bound

```java
// ❌ WRONG — for #787, where the round count IS the hop limit
for (int round = 0; round <= k; round++)
    for (int[] f : flights)
        if (dist[f[0]] + f[2] < dist[f[1]]) dist[f[1]] = dist[f[0]] + f[2];
```

💥 **Symptom:** answers that are too small — routes using more than `k+1` edges
sneak in, because a value updated earlier in the same round is read later in that
same round.

```java
// ✅ CORRECT — snapshot the previous round
int[] next = dist.clone();
for (int[] f : flights)
    if (dist[f[0]] != INF && dist[f[0]] + f[2] < next[f[1]])
        next[f[1]] = dist[f[0]] + f[2];
dist = next;
```

🧪 **Catch it with:** `#787` example 2 — `n=3`, `[[0,1,100],[1,2,100],[0,2,500]]`,
`src=0 dst=2 k=0`. Expected `500`; in-place relaxation returns `200`.

* * *

## Failure 5 — Maximising without flipping all three places

```java
// ❌ WRONG — max-heap, but the rest still minimises
PriorityQueue<double[]> pq = new PriorityQueue<>((a,b) -> Double.compare(b[1], a[1]));
...
if (p > best[u]) continue;              // stale test not reversed
if (np < best[v]) { ... }               // relax not reversed
```

💥 **Symptom:** garbage, usually `0` or the first path found. Flipping the heap
alone is not enough — the comparison appears in **three** places and all three must
agree.

```java
// ✅ CORRECT — heap, stale test, and relax all reversed together
PriorityQueue<double[]> pq = new PriorityQueue<>((a,b) -> Double.compare(b[1], a[1]));
...
if (p < best[u]) continue;
if (np > best[v]) { ... }
```

🧪 **Catch it with:** `#1514` example 1 — expected `0.25`.

* * *

## Failure 6 — Wrong loop order in Floyd-Warshall

```java
// ❌ WRONG — the intermediate vertex is not outermost
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++)
        for (int mid = 0; mid < n; mid++)
            d[i][j] = Math.min(d[i][j], d[i][mid] + d[mid][j]);
```

💥 **Symptom:** distances too large, and the error is data-dependent so small tests
often pass. The algorithm's meaning is "paths allowed to pass through vertices
`0..mid`" — that induction only holds if `mid` is the outer loop.

```java
// ✅ CORRECT
for (int mid = 0; mid < n; mid++)
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++) { ... }
```

🧪 **Catch it with:** `#1334` example 2 — expected city `0`.

* * *

## Failure 7 — INF arithmetic overflow

```java
// ❌ WRONG
Arrays.fill(dist, Integer.MAX_VALUE);
if (dist[u] + w < dist[v]) ...           // MAX_VALUE + w wraps NEGATIVE
```

💥 **Symptom:** unreachable nodes appear to have huge negative distances, and
everything downstream is wrong.

```java
// ✅ CORRECT — guard, or use a smaller sentinel
if (dist[u] == Integer.MAX_VALUE) continue;
// or
int INF = 1_000_000_000;                 // leaves headroom for INF + w
```

🧪 **Catch it with:** any graph with an unreachable node and a positive weight —
`n=2`, no edges, then relax from node 0.

* * *

## Failure 8 — Collapsing the state in Pattern 3

```java
// ❌ WRONG — #1928 with a 1-D best[]
int[] best = new int[n];
if (nc < best[v]) { ... }
```

💥 **Symptom:** wrong answer that passes the samples. A route reaching city `v`
cheaply but slowly blocks a costlier-but-faster one that was the only way to finish
inside `maxTime`.

```java
// ✅ CORRECT
int[][] best = new int[n][maxTime + 1];
if (nc < best[v][t]) { ... }
```

🧪 **Catch it with:** any `#1928` case where the cheapest route exceeds `maxTime` but
a pricier one fits.

* * *

## Edge case sweep

Every row names the line or Failure that handles it, plus an input that proves it.

| Edge case | Handled by | Verify with |
|---|---|---|
| Target unreachable | `dist[t] == INF` check before returning | `#743` `n=2, k=2, times=[[1,2,1]]` → `-1` |
| Source == target | `dist[src] = 0`, popped first | `#1631` on a 1×1 grid → `0` |
| Self-loop `u → u` | relaxation fails: `d + w < d` is false for `w ≥ 0` | any graph with `[u,u,5]` |
| Parallel edges | `Math.min` when building, or relaxation picks the cheaper | `#1334` with duplicated edges |
| Zero-weight edges | strict `<` in the relax test — frame line (3) | a chain of `w = 0` edges |
| Unreachable node in Bellman-Ford | the `dist[u] == INF` guard — Failure 7 | `#787` with a disconnected `dst` |
| Weights near `Integer.MAX_VALUE` | `Integer.compare` — Failure 2 | two costs at ±2e9 |
| Budget exactly exhausted at the goal | `t > maxTime` rejects only strict excess | `#1928` where the fastest route uses exactly `maxTime` |
| Grid with one cell | target test on pop, before any neighbour loop | `#2812` on `[[0]]` |

> ✅ **Checkpoint:** find the bug you logged at Gate 1 or 2 in the eight failures
> above. **Not there? Add it** — with its own ❌ / 💥 / ✅ / 🧪 entry. A bug you hit
> that this section does not name is a real gap, and you are the only person who can
> close it.

* * *

## 🔓 ATTEMPT GATE 3 — STOP READING

> You have now seen your own bugs named. Go close the loop.

```
→ Re-attempt EVERY problem you failed or bugged at Gates 1 and 2.
→ Then two new ones:  #1334 Find the City...              Medium  ← Floyd-Warshall
                      #1976 Number of Ways to Arrive...   Medium  ← Dijkstra + counting

Rules: 30 min each · no hints · this doc CLOSED
       Before you run the code, predict which Failure from §7 you are most
       likely to have committed, then check for it deliberately.
```

```
□ All clean this time   → continue to §8
□ Same bug recurred     → it is a habit, not a knowledge gap. Write the frame
                          out longhand three times before continuing.
□ #1334 wrong answer    → you almost certainly put `mid` in the inner loop.
                          Reread Failure 6.
```

* * *

# 8. Tricks & Insights

> 🎯 **Objective:** acquire the reframings that turn Hard problems into Medium ones.
> 🔗 **Builds on:** §5 (patterns) · §7 (what you now avoid).
> ⏱️ **Session 3**

> These are **reframings**, not bug guards. Every bug guard lives in §7 and appears
> nowhere else.

## Trick 1 — A* : Dijkstra with a compass

Dijkstra expands in every direction because it has no idea where the target is. If
you can cheaply *estimate* the remaining distance, add it to the priority:

```java
// order by  cost-so-far + heuristic(remaining)
pq.add(new int[]{r, c, d, d + manhattan(r, c, targetR, targetC)});
```

The heuristic must be **admissible** — never an over-estimate — or you lose
optimality. Manhattan distance on a 4-directional grid is admissible; straight-line
distance in Euclidean space is admissible.

**Buys you:** large sparse grids where the target is far away in a known direction.
On LeetCode it is rarely *needed*, but naming it is a strong signal.

## Trick 2 — Build the graph you need, not the one you were given

`#2662` gives you coordinates and a list of "special roads", not a graph. `#399`
gives you equations between strings. `#2976` gives you two parallel arrays of
characters. In each case the first move is the same: decide what a node is, then
manufacture the adjacency.

```java
Map<String, Integer> id = new HashMap<>();     // string → index
id.computeIfAbsent(name, x -> id.size());
```

**Buys you:** #2662, #399, #2976 — and it is the same inversion move as archetype
13's `#815` (nodes are routes) and `#1345` (value → index map).

## Trick 3 — Dijkstra also gives you a topological order for free

Nodes are finalised in non-decreasing distance order. If a DP over paths needs to
process nodes in that order, Dijkstra hands it to you — no separate sort.

```java
// #1786: run Dijkstra from n, then process nodes in increasing dist to count paths
```

**Buys you:** #1786, #1976 — anything that counts or accumulates *along* shortest
paths.

## Trick 4 — Counting shortest paths is one extra array

```java
if (nd < dist[v])       { dist[v] = nd; ways[v] = ways[u]; pq.add(...); }
else if (nd == dist[v]) { ways[v] = (ways[v] + ways[u]) % MOD; }
```

Strictly better → inherit the count. Exactly equal → add to it. Note the `else if`
does **not** re-push.

**Buys you:** #1976, #2045.

## Trick 5 — Waiting is a move

`#2577` lets you stand still. If the parity of arrival time is wrong, you can hop to
a neighbour and back, costing exactly 2. So an unreachable-looking time becomes
reachable with a `+2` adjustment.

```java
if ((t - dist[r][c]) % 2 == 1) t++;      // burn one round trip
```

**Buys you:** #2577, and any problem where time advances but you may idle.

## Trick 6 — Floyd-Warshall computes reachability, not just distance

Replace `min(+)` with `or(and)` and the same triple loop gives transitive closure.

```java
reach[i][j] = reach[i][j] || (reach[i][mid] && reach[mid][j]);
```

**Buys you:** #1462 — which asks only "is `u` a prerequisite of `v`", with no
weights at all.

> ✅ **Checkpoint:** name the trick that cracks `#1786`, and the one that makes
> `#399` tractable.

* * *

# 9. Recognition & Decision Framework

> 🎯 **Objective:** classify a problem into this archetype (or out of it) in under 30
> seconds, then pick the pattern.
> 🔗 **Builds on:** §4 (knobs) · §5 (patterns).
> ⏱️ **Session 4**

> Deliberately late. Recognition signals are compressed model knowledge — read
> before §3–§5 they are keywords to memorise; read now they are shorthand for
> machinery you own.

## Strong signals

* **Numbers on the edges.** Prices, times, distances, weights, probabilities. This
  is the single sharpest test.
* "minimum **cost** / total / price / effort / time" — a sum, not a count
* "cheapest", "least expensive", "smallest possible maximum"
* "maximum probability", "best exchange rate" — a product
* "at most K stops", "within T minutes" — a weighted problem with a budget
* Constraint sizes around `10^4–10^5` edges — too big for path enumeration

## Weak / disguised signals

* "Minimise the **largest** step / the **highest** water level" → Pattern 2, `max`
* "Maximise the **smallest** gap / the **widest** path" → Pattern 2, `min` + max-heap
* "Distance between **every** pair", `n ≤ 400` → Pattern 4, Floyd-Warshall
* Equations, exchange rates, or unit conversions → a weighted graph over strings
* "The signal reaches everyone" / "all nodes receive" → single-source, take the max

## Anti-signals — when this is the WRONG archetype

| Phrase in the problem | What it actually is | Archetype |
|---|---|---|
| every move costs the same; "fewest steps / moves" | plain BFS | **13** |
| "how many islands / regions / components" | DFS flood fill | **12** |
| "order the tasks given prerequisites", DAG | topological sort | **15** |
| "are these connected", repeated merges, no path needed | Union-Find | **16** |
| "how many **distinct paths**" (count, not cost) | DP | **23+** |
| "the **longest** path" in a general graph | NP-hard; on a DAG it is DP | **23+** |

> 🧠 The sharpest test in both directions: **does the problem put numbers on the
> edges?** If no, you are in archetype 13. If yes, you are here — no matter how much
> it looks like a grid walk.

## Decision tree

```
Are the edges WEIGHTED, and are you minimising a COST?
├── NO ──────────────────────────────────────────────────────────────┐
│    ├── all edges cost the same?          → BFS          (arch 13)  │
│    ├── counting components?              → DFS flood    (arch 12)  │
│    ├── ordering with prerequisites?      → topo sort    (arch 15)  │
│    ├── connectivity only?                → Union-Find   (arch 16)  │
│    └── counting paths, or a maximum?     → DP           (arch 23+) │
│                                                                    │
└── YES → you are in archetype 14. Now set the knobs: ───────────────┘
     │
     ├── KNOB C first · any NEGATIVE weights?
     │    └── YES                                    → Bellman-Ford   (P4)
     │    └── need ALL-PAIRS and n <= 400            → Floyd-Warshall (P4)
     │    └── a bound on the NUMBER of edges         → Bellman-Ford   (P4)
     │
     ├── KNOB B · do I carry a budget? ("at most K", "within T")
     │    └── YES                                    → state-augmented (P3)
     │
     ├── KNOB A · how does an edge join a path?
     │    ├── sum                                    → Pattern 1
     │    ├── max  (minimise the largest)            → Pattern 2, min-heap
     │    ├── min  (maximise the smallest)           → Pattern 2, MAX-heap
     │    └── product (maximise)                     → Pattern 2, MAX-heap
     │
     └── default: A:sum B:node C:min-heap            → Pattern 1
```

Check Knob C first — a negative weight disqualifies the heap no matter what the
other two say.

> ✅ **Checkpoint:** run the tree on `#2045`, `#2976` and `#778` without looking at
> their labels in §10. 3/3, or reread §4.

* * *

# 10. Problem Map Quick Reference

> 🎯 **Objective:** given a pattern, recall its problems — and vice versa.
> ⏱️ **Session 4**

## The 17 problems

Tick them off in `leetcode_study_roadmap.md`; this table is the working reference.
Priorities match `scripts/add_archetype14_reminders.sh`.

### 2 Essence · 🔴 high priority

| # | Problem | Diff | Knobs | Pattern | Gate |
|---|---|---|---|---|---|
| 743 | [Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Med | A:sum B:node C:min-heap | P1 | 🔓 1 |
| 787 | [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | Med | A:sum B:node C:layered | P4 | 🔓 2 |

### 5 Variations · 🟡 medium priority

| # | Problem | Diff | Knobs | Pattern | What it adds |
|---|---|---|---|---|---|
| 1631 | [Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/) | Med | A:**max** B:node C:min-heap | P2 | the combiner stops being a sum |
| 1514 | [Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/) | Med | A:**product** B:node C:**MAX**-heap | P2 | all three comparisons flip |
| 1334 | [Find the City With the Smallest Number of Neighbors…](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/) | Med | C:**triple loop** | P4 | all-pairs; `mid` must be outermost |
| 1976 | [Number of Ways to Arrive at Destination](https://leetcode.com/problems/number-of-ways-to-arrive-at-destination/) | Med | A:sum B:node C:min-heap | P1 | counting on top of distances (Trick 4) |
| 1368 | [Minimum Cost to Make at Least One Valid Path in a Grid](https://leetcode.com/problems/minimum-cost-to-make-at-least-one-valid-path-in-a-grid/) | Hard | A:sum B:node C:**deque** | P1 | 0-1 weights — archetype 13's `#2290` again |

### 10 Mixed · ⚪ normal priority

| # | Problem | Diff | Pattern | What it is FOR |
|---|---|---|---|---|
| 1786 | [Number of Restricted Paths From First to Last Node](https://leetcode.com/problems/number-of-restricted-paths-from-first-to-last-node/) | Med | P1 | Dijkstra hands DP a processing order — Trick 3 |
| 2045 | [Second Minimum Time to Reach Destination](https://leetcode.com/problems/second-minimum-time-to-reach-destination/) | Hard | P1 | the SECOND shortest, so first-arrival is not the answer |
| 2662 | [Minimum Cost of a Path With Special Roads](https://leetcode.com/problems/minimum-cost-of-a-path-with-special-roads/) | Med | P1 | the graph is implicit: nodes are coordinates you invent |
| **2812** | [**Find the Safest Path in a Grid**](https://leetcode.com/problems/find-the-safest-path-in-a-grid/) | Med | **P2 ⭐** | **FLAGSHIP** — arch-13 BFS builds the weights an arch-14 max-min Dijkstra consumes |
| 778 | [Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/) | Hard | P2 | minimise the MAXIMUM — the cleanest non-additive combiner |
| 1928 | [Minimum Cost to Reach Destination in Time](https://leetcode.com/problems/minimum-cost-to-reach-destination-in-time/) | Hard | P3 | a budget makes the node a pair — Knob B with weights |
| 2577 | [Minimum Time to Visit a Cell In a Grid](https://leetcode.com/problems/minimum-time-to-visit-a-cell-in-a-grid/) | Hard | P3 | waiting is legal; parity turns "unreachable" into "+2" |
| 1462 | [Course Schedule IV](https://leetcode.com/problems/course-schedule-iv/) | Med | P4 | Floyd-Warshall as transitive closure, with no weights at all |
| 2976 | [Minimum Cost to Convert String](https://leetcode.com/problems/minimum-cost-to-convert-string/) | Med | P4 | a 26-node alphabet graph makes `O(V³)` trivially fine |
| 399 | [Evaluate Division](https://leetcode.com/problems/evaluate-division/) | Med | P4 | the graph is built from strings, and the cost is a product |

**Totals:** 17 problems · 12 Medium / 5 Hard · flagship `#2812`

* * *

```
PATTERN                          PROBLEMS
──────────────────────────────────────────────────────────────────────
Essence                          #743, #787
Variations                       #1631, #1514, #1334, #1976, #1368

Pattern 1 · Dijkstra             #1786, #2045, #2662
Pattern 2 · Non-additive cost    #2812 ⭐, #778
Pattern 3 · State-augmented      #1928, #2577
Pattern 4 · Bounded / all-pairs  #1462, #2976, #399

Composite (2+ archetypes)        #2812 (arch 13 BFS + P2), #1786 (P1 + DP),
                                 #1368 (arch 13 0-1 BFS + P1)
```

## By difficulty

```
MEDIUM  #743 #787 #1631 #1514 #1334 #1976 #1786 #2662 #2976 #399 #1462 #2812
HARD    #1368 #2045 #1928 #2577 #778
```

## Reverse lookup — problem → knobs

```
#743   A:sum      B:node        C:min-heap    P1   the canonical Dijkstra
#787   A:sum      B:node        C:layered     P4   hop bound = round count
#1631  A:max      B:node        C:min-heap    P2   minimise the largest step
#1514  A:product  B:node        C:MAX-heap    P2   all three comparisons flip
#1334  A:sum      B:node        C:triple loop P4   all-pairs
#1976  A:sum      B:node        C:min-heap    P1 + Trick 4 (counting)
#1368  A:sum      B:node        C:deque       P1   0-1 weights: arch-13's #2290 again
#2812  A:min      B:node        C:MAX-heap    P2   ⭐ arch-13 BFS builds the weights
#1928  A:sum      B:node+time   C:min-heap    P3   the budget is the dimension
#2976  A:sum      B:node        C:triple loop P4   26-node alphabet graph
```

## What each Mixed problem is FOR

```
#1786  Dijkstra hands DP a processing order — Trick 3
#2045  the SECOND shortest, so first-arrival is not the answer
#2662  the graph is implicit: nodes are coordinates you invent
#2812  ⭐ two archetypes with a handoff, and both directions of Knob A
#778   minimise the MAXIMUM — the cleanest non-additive combiner
#1928  a budget makes the node a pair — Knob B with weights
#2577  waiting is a legal move; parity turns "unreachable" into "+2"
#1462  Floyd-Warshall as transitive closure, with no weights at all
#2976  a tiny fixed vertex set (26 letters) makes O(V^3) trivially fine
#399   the graph is built from strings, and the cost is a product
```

## Cut, with reasons

Restore any of these if the archetype feels thin:

```
#847   Shortest Path Visiting All Nodes    needs bitmask state    → archetype 20
#864   Shortest Path to Get All Keys       needs bitmask state    → archetype 20
#2258  Escape the Spreading Fire           binary search on answer→ archetype 21
#2290  Minimum Obstacle Removal            already archetype 13's boundary problem
#1102  Path With Maximum Minimum Value     🔒 Premium; #778 teaches the same idea
#505   The Maze II                         🔒 Premium
#2093  Minimum Cost to Reach City...       🔒 Premium; #1928 covers state-augmented
#2473  Minimum Cost to Buy Apples          🔒 Premium
#2699  Modify Graph Edge Weights           rare shape, high plumbing, thin lesson
#2192  All Ancestors of a Node             reachability only, no weights
```

> ⏭️ **Deliberately excluded — come back after archetype 20.** `#847` and `#864` are
> the canonical next-level shortest-path problems, but both need **bitmask state** in
> the key. They are Pattern 3 with `B: node + bitmask` — a pattern you already know,
> waiting on one missing tool.

* * *

# 11. Readiness Gate

> 🎯 **Objective:** produce a verdict — ready, or ready-except-X.
> 🔗 **Builds on:** everything.
> ⏱️ **Session 4**

> You are ready when you can pick up an **unseen** problem in this archetype and make
> credible progress in 5 minutes with this document closed.
> **All four tests must pass. No partial credit.**

## The four tests

| # | Test | Task | Pass bar | If you fail |
|---|---|---|---|---|
| 1 | **RECALL** | Blank file → write the §3 Dijkstra frame. Must compile. | < 4 min, 0 bugs | → §3 |
| 2 | **RECOGNIZE** | The 8 titles below. In-archetype or not? If not, which archetype? | 8/8, < 90 sec | → §9 |
| 3 | **MAP** | The 3 problems below → knob settings + pattern. | < 30 sec each | → §4 |
| 4 | **SOLVE COLD** | One unseen Medium from §10. No doc, no hints, no editorial. | Accepted, < 35 min | → §5 + §7 |

### Test 1 — recall

Write from a blank file, no reference:

```
□ the Dijkstra frame, with the stale check and Integer.compare
□ the Bellman-Ford round loop, with the snapshot
□ the Floyd-Warshall triple loop, with mid outermost
□ the three flips needed to turn minimise into maximise
```

### Test 2 — recognize

Three of these eight are **not** archetype 14. Name which, and which archetype each
actually belongs to.

```
1. Network Delay Time                              (#743)
2. Rotting Oranges                                 (#994)
3. Path With Minimum Effort                        (#1631)
4. Course Schedule                                 (#207)
5. Cheapest Flights Within K Stops                 (#787)
6. Number of Provinces                             (#547)
7. Swim in Rising Water                            (#778)
8. Minimum Cost to Convert String                  (#2976)
```

<details>
<summary>Answer key</summary>

Decoys: **#994** → archetype 13, every minute costs the same, plain multi-source BFS.
**#207** → archetype 15, prerequisites over a DAG, no weights.
**#547** → archetype 12, counting components, no distance asked.
The rest are archetype 14: #743 P1, #1631 P2, #787 P4, #778 P2, #2976 P4.

</details>

### Test 3 — map

Fill these in, out loud, timed:

```
#2045 Second Minimum Time to Reach Destination   A:___ B:___ C:___  Pattern ___
#399  Evaluate Division                          A:___ B:___ C:___  Pattern ___
#2577 Minimum Time to Visit a Cell In a Grid     A:___ B:___ C:___  Pattern ___
```

<details>
<summary>Answer key</summary>

`#2045` A:sum B:node C:min-heap (or plain BFS — every edge costs the same `time`,
so this is genuinely archetype 13 with a twist; the interesting part is tracking the
*second* distance). → **P1 + Trick 4**.
`#399` A:product B:node C:min-heap or triple loop; the graph is built from strings.
→ **P4** (Floyd-Warshall) or P2 with DFS.
`#2577` A:max-with-adjustment B:node C:min-heap, plus Trick 5 for parity.
→ **P2 / P3 boundary** — worth saying aloud that it is a hybrid.

</details>

### Test 4 — solve cold

Pick a problem from §10 you have **not** attempted. Fill this in before writing any
code — if you cannot fill it in 3 minutes, you are not ready to code yet.

```
Signal check  — why archetype 14 and not 13 / 15 / 16 / 23?
Any negative weights?  — if yes, the heap is out before anything else
Pattern       — □ P1  □ P2  □ P3  □ P4
Knobs         — A: ______  B: ______  C: ______
Node          — what exactly is one node? (vertex? vertex + budget?)
Combiner      — how does an edge join a path? sum / max / min / product
Direction     — minimise or maximise? if maximise, WHICH THREE lines flip?
Termination   — return on pop, or read dist[] at the end?
Edge cases    — unreachable? source == target? INF overflow?
```

## Verdict

```
4/4  → ✅ CONFIDENT.
        Work the remaining §10 problems at interview pace:
        Medium ≤ 25 min, Hard ≤ 40 min. Then move to archetype 15.

3/4  → ⚠️  Route to the failed test's section in the table above.
        Retest ONLY that test. Do not re-read the whole document.

≤2/4 → ❌ Do not proceed to archetype 15.
        Re-run Part B (§2–§6) including Gates 1 and 2. The gap is in the
        frame or the knobs, and no amount of extra problems will fix it.
```

## Verbal drill — do this at 4/4, not before

Explain the archetype in 2 minutes, out loud, to an empty room:

1. **What it is** — Dijkstra finalises nodes in cost order, and the cheapest
   unfinalised node is already final, because any alternative route must exit the
   finalised set through something at least as expensive.
2. **The frame** — a priority queue of `(node, dist)` pairs, a stale check on pop,
   and a strict relaxation.
3. **The knobs** — how an edge joins a path, whether you carry a budget, and what
   discipline extracts the next node. Four patterns, three dials, one loop.
4. **The hardest variant** — non-additive costs. Dijkstra never needed addition,
   only monotonicity, which is why minimax, maximin and products all work.
5. **The boundary** — one negative edge and the greedy proof dies. That is where
   Bellman-Ford starts.

If you can say all five without notes, you can teach this. That is the bar.

* * *

*Archetype 14 · Weighted Shortest Path · v2 format · 17 problems (2 + 5 + 10)*
