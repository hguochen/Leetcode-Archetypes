# 🌊 Archetype 12 — DFS Flood Fill

* * *

# 0. Goal

## What problem class does this solve?

* Problems on a **grid or graph** where you must explore a connected region starting from a cell/node
* Problems that reduce to: "spread outward from here, marking everything reachable, and stop at boundaries"
* Counting connected components, measuring region size/shape, or transforming a whole region at once
* Key invariant: **every cell/node is visited at most once** — a `visited` marker turns exponential re-exploration into linear traversal

## What mastery looks like

* Recognizes flood fill in < 30 seconds: "grid + connectivity + regions" or "graph + components"
* Can write the recursive DFS grid template and the adjacency-list DFS template from scratch, no bugs
* Knows the four bug-killers cold: bounds check, visited check, mark-on-entry, 4 vs 8 directions
* Can compose flood fill with counting, area aggregation, boundary-anchoring, and multi-source seeding
* Can explain WHY marking visited *before* recursing (not after) prevents infinite loops and double counting

* * *

# 1. Recognition Signals

## Strong signals

* Input is a 2D grid (`char[][]`, `int[][]`) of land/water, 0/1, colors
* "Number of islands / regions / provinces / components"
* "Area / size / perimeter of the largest region"
* "Fill / flood / paint a connected region a new color"
* "Surrounded / enclosed / captured regions"
* Input is an adjacency list/matrix and you must count "connected components" or check reachability

## Weak / disguised signals

* "Cells connected to the border" (Pacific/Atlantic, Surrounded Regions) → reverse flood from edges
* "How many groups of friends / provinces" (adjacency matrix, not a grid)
* "Can you get from A to B" in a graph → single DFS reachability
* "Bipartite / 2-colorable" → DFS coloring is a flood fill with a color constraint
* "Detonate bombs / spread infection / reachable set" → DFS on an implicit graph

## Anti-signals (when NOT to use plain DFS flood fill)

* "**Shortest** path / fewest steps" in an unweighted grid → BFS (Archetype 13), not DFS
* "Minimum spanning / cheapest connection cost" → Union-Find / MST (later)
* "Order tasks with prerequisites" → Topological Sort (later)
* Weighted shortest path → Dijkstra (later)

* * *

# 2. Cheat Sheet

```
WHEN TO USE:
- 2D grid where connected cells form regions (islands, provinces, zones)
- "Count components", "measure area/shape", "fill a region", "enclosed regions"
- Graph given as adjacency list/matrix; need components or reachability
- You do NOT need the shortest path (if you do → BFS)

CORE IDEA:
- Pick a start cell/node, recurse to all valid neighbors, mark visited as you go
- The recursion stack IS the frontier — no explicit stack needed
- One flood = one connected component. Count how many floods you launch.
- visited[] (or in-place mutation) is what makes it O(V+E) instead of infinite

THE 4 PATTERNS:
1. Grid Component Count   → loop every cell; each unvisited start = +1 component, then flood it
2. Region Aggregation     → dfs returns a value (area/perimeter); combine over the 4 neighbors
3. Boundary-Anchored Fill → flood FROM the borders to mark "safe"; leftovers are enclosed
4. Graph DFS (adj list)   → same idea on nodes+edges: count components / reachability / coloring

TEMPLATE SELECTOR:
- "How many islands/provinces?"     → Grid Component Count / Graph DFS
- "Largest area / total perimeter?" → Region Aggregation (dfs returns int)
- "Regions NOT touching border?"    → Boundary-Anchored Fill (flood from edges first)
- "Recolor a connected blob?"       → single flood from the given seed
- "Is graph 2-colorable / bipartite?" → Graph DFS with color state

TIME / SPACE:
- Grid m x n:  O(m*n) time, O(m*n) space (recursion stack worst case)
- Graph:       O(V + E) time, O(V) space
- In-place marking saves the visited[] array (O(1) extra besides stack)

TOP 3 TRICKS:
1. Mark visited the INSTANT you enter a cell, before recursing — not after
2. Put the bounds + visited + validity checks at the TOP of dfs (base case guard)
3. To find "enclosed" regions, flood the complement from the border — invert the question

TOP 3 PITFALLS:
1. Marking visited too late → same cell enqueued by two neighbors → double count / stack overflow
2. Forgetting bounds check → ArrayIndexOutOfBounds at grid edges
3. Using 8 directions when the problem means 4 (or vice versa)
```

* * *

# 3. Core Mental Model

## Core Ideas

* A grid is just a graph in disguise: each cell is a node; edges connect orthogonal (or diagonal) neighbors
* Flood fill = DFS that **claims** every reachable cell exactly once by marking it visited
* You cannot un-visit a cell — that is the whole point; the mark is permanent within one traversal
* What you *can* control: the start seeds, the direction set, what the dfs returns, and what "valid neighbor" means
* Every problem composes from: **when do I launch a flood**, **what do I mark**, **what do I return/aggregate**

## The Grid Contract

```java
// The grid and the neighbor model
int m = grid.length, n = grid[0].length;
int[][] DIRS = {{-1,0},{1,0},{0,-1},{0,1}};   // 4-directional (up, down, left, right)
// For 8-directional add the diagonals:
// {{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}}
```

## The Core Flood Frame

```java
// The universal recursive grid flood — solves ~80% of grid problems
void dfs(int r, int c, int[][] grid, boolean[][] visited) {
    // 1. Base case guard: out of bounds, already visited, or not part of the region
    if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length) return;
    if (visited[r][c] || grid[r][c] == 0 /* water / wall */) return;

    // 2. Mark visited IMMEDIATELY (before recursing) — this is the safety latch
    visited[r][c] = true;

    // 3. Spread to all neighbors
    for (int[] d : DIRS) dfs(r + d[0], c + d[1], grid, visited);
}
```

And the outer driver that turns one flood into a component count:

```java
int components = 0;
for (int r = 0; r < m; r++)
    for (int c = 0; c < n; c++)
        if (!visited[r][c] && grid[r][c] == 1) {
            dfs(r, c, grid, visited);   // one flood drains one whole region
            components++;               // ...so bump the count once per launch
        }
```

## The 4 Patterns

```
┌────────────────────────────┬────────────────────────────────────────────────┐
│ PATTERN                    │ ROLE                                           │
├────────────────────────────┼────────────────────────────────────────────────┤
│ 1. Grid Component Count    │ Outer loop launches a flood per unvisited seed  │
│ 2. Region Aggregation      │ dfs returns area/perimeter; sum over neighbors  │
│ 3. Boundary-Anchored Fill  │ Flood from edges to mark "safe"; invert answer  │
│ 4. Graph DFS (adjacency)   │ Same flood on nodes+edges: components/reach/color│
└────────────────────────────┴────────────────────────────────────────────────┘
```

## Problem Transformation

```
A. Single pattern   → #200 Number of Islands (Grid Component Count)
                      #733 Flood Fill (single seed flood)
B. Two patterns     → #695 Max Area of Island (Component Count + Region Aggregation)
                      #130 Surrounded Regions (Boundary-Anchored Fill)
C. Hard composition → #417 Pacific Atlantic (multi-source Boundary Fill x2 + intersect)
                      #827 Making a Large Island (label regions + probe each 0-cell)
```

## Visualization

### One flood drains one component

```
grid:            flood launched at (0,0):        after flood:
1 1 0            (0,0)→(0,1)→(1,1)? no water     X X 0
1 0 1            stops at water & bounds          X 0 1
0 0 1                                             0 0 1

Component #1 = the top-left blob of 1s (all become X = visited).
Outer loop then finds (1,2) still unvisited → launches Component #2.
```

### Boundary-Anchored Fill (Surrounded Regions)

```
Original:      Flood 'O's connected to border → mark SAFE (#):
X X X X        X X X X
X O O X   →    X O O X   (inner O's never reached from border)
X X O X        X X O X
X O X X        X # X X   (this O touches the bottom border → safe)

Final pass: remaining O (not #) → flip to X; # → back to O.
```

### 4 vs 8 directions

```
   4-dir (orthogonal)        8-dir (with diagonals)
        · N ·                     NW N NE
        W C E                     W  C  E
        · S ·                     SW S SE
```

* * *

# 4. Optimization Ladder (Interview Narrative)

## Step 1 — Brute Force

* Naive "for each cell, re-scan the whole grid to find its group" → recomputes membership repeatedly
* Or: repeatedly sweep the grid flipping neighbors until nothing changes — O((m*n)²) in the worst case
* Acceptable to state, but wasteful: it revisits settled cells over and over

## Step 2 — Optimization Insight

What repeating work are we eliminating?

* Once a cell is claimed by a region, it never needs to be examined again
* The connectivity is transitive — a single DFS/BFS from any seed discovers the *entire* component in one shot

Core shift:

```
from: re-scan / re-sweep the grid to grow regions incrementally
to:   one DFS per component visits each cell exactly once; a visited mark makes re-entry a no-op
```

## Step 3 — Optimal Approach

* Key idea: sweep once; each unvisited land cell launches a single flood that claims its whole component
* Time: O(m*n) grid / O(V+E) graph — every cell/edge touched a constant number of times
* Space: O(m*n) worst-case recursion stack (a single snake-shaped region); O(V) for graphs

* * *

# 5. Core Patterns

```
Pattern 1 Grid Component Count → outer loop + void dfs; count++ per launch
Pattern 2 Region Aggregation   → int dfs returns area/perimeter; sum neighbors
Pattern 3 Boundary-Anchored    → seed floods from all border cells; invert
Pattern 4 Graph DFS (adj)      → void/boolean dfs over adjacency list; visited set
```

## Pattern 1 — Grid Component Count

**When to use:**

* "How many islands / regions / groups?"
* Each connected blob should contribute exactly 1 to the answer

**Mental model:**

```
Sweep every cell. The first time you touch an unvisited region member,
launch a flood that drowns the ENTIRE region, then increment the count once.
The flood guarantees you never launch twice inside the same region.
```

```java
// Number of Islands (#200) — Grid Component Count
public int numIslands(char[][] grid) {
    int m = grid.length, n = grid[0].length, count = 0;
    for (int r = 0; r < m; r++)
        for (int c = 0; c < n; c++)
            if (grid[r][c] == '1') {   // unvisited land
                dfs(grid, r, c);       // sink the whole island
                count++;
            }
    return count;
}

private void dfs(char[][] grid, int r, int c) {
    if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length) return;
    if (grid[r][c] != '1') return;     // water or already-sunk land
    grid[r][c] = '0';                  // MARK visited immediately (in-place)
    dfs(grid, r + 1, c);
    dfs(grid, r - 1, c);
    dfs(grid, r, c + 1);
    dfs(grid, r, c - 1);
}
```

**Why it works:**
Every land cell belongs to exactly one component. Because the flood marks cells `'0'` before recursing, once a region is entered it is fully consumed and can never trigger a second `count++`. The number of flood launches equals the number of components.

* * *

## Pattern 2 — Region Aggregation

**When to use:**

* You need a *measurement* of each region: area, perimeter, gold collected, sub-island match
* The region's value is a sum/combination of contributions from its cells

**Mental model:**

```
Make dfs RETURN a number. A single cell contributes its own value (e.g. 1 for area),
then adds whatever its neighbors return. null/out-of-region contributes the identity (0).
```

```java
// Max Area of Island (#695) — Region Aggregation
public int maxAreaOfIsland(int[][] grid) {
    int best = 0;
    for (int r = 0; r < grid.length; r++)
        for (int c = 0; c < grid[0].length; c++)
            if (grid[r][c] == 1)
                best = Math.max(best, area(grid, r, c));
    return best;
}

private int area(int[][] grid, int r, int c) {
    if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length) return 0;
    if (grid[r][c] != 1) return 0;      // water / visited → contributes 0
    grid[r][c] = 0;                     // mark visited
    return 1                            // this cell
         + area(grid, r + 1, c)
         + area(grid, r - 1, c)
         + area(grid, r, c + 1)
         + area(grid, r, c - 1);
}
```

**Why it works:**
Each cell adds `1` exactly once (it is zeroed on entry, so recursion never double-counts it), and the four recursive calls fan out over the rest of the component. The returned sum is the total area of the region rooted at the seed.

* * *

## Pattern 3 — Boundary-Anchored Fill

**When to use:**

* "Regions **not** touching the border", "surrounded / enclosed / captured cells"
* The property is defined by connection to the edge → flood from the edge, then invert

**Mental model:**

```
Directly finding "enclosed" regions is awkward. Invert it:
flood inward from EVERY border cell to mark everything reachable-from-edge as SAFE.
Whatever is left unmarked is, by definition, enclosed.
```

```java
// Surrounded Regions (#130) — Boundary-Anchored Fill
public void solve(char[][] board) {
    int m = board.length, n = board[0].length;
    // 1. Flood 'O's connected to the border → temporarily mark '#'
    for (int r = 0; r < m; r++) { mark(board, r, 0); mark(board, r, n - 1); }
    for (int c = 0; c < n; c++) { mark(board, 0, c); mark(board, m - 1, c); }
    // 2. Remaining 'O' are enclosed → flip to 'X'; restore '#' back to 'O'
    for (int r = 0; r < m; r++)
        for (int c = 0; c < n; c++) {
            if (board[r][c] == 'O') board[r][c] = 'X';
            else if (board[r][c] == '#') board[r][c] = 'O';
        }
}

private void mark(char[][] b, int r, int c) {
    if (r < 0 || r >= b.length || c < 0 || c >= b[0].length) return;
    if (b[r][c] != 'O') return;         // wall 'X' or already marked '#'
    b[r][c] = '#';                      // safe (border-connected)
    mark(b, r + 1, c); mark(b, r - 1, c);
    mark(b, r, c + 1); mark(b, r, c - 1);
}
```

**Why it works:**
An `'O'` survives as `'O'` only if no path of `'O'`s connects it to the border. By flooding from all border seeds first, every edge-connected `'O'` becomes `'#'`. Any `'O'` still standing afterward is provably enclosed, so flipping it to `'X'` is correct.

* * *

## Pattern 4 — Graph DFS (Adjacency)

**When to use:**

* Input is an adjacency matrix/list (provinces, friend circles, network nodes)
* Count connected components, test reachability, or 2-color (bipartite) via DFS

**Mental model:**

```
Same flood, but "neighbors" come from the adjacency structure instead of grid deltas.
A visited[] set replaces the grid mutation. One DFS launch = one component.
```

```java
// Number of Provinces (#547) — Graph DFS component count on adjacency matrix
public int findCircleNum(int[][] isConnected) {
    int n = isConnected.length, provinces = 0;
    boolean[] visited = new boolean[n];
    for (int i = 0; i < n; i++)
        if (!visited[i]) { dfs(isConnected, visited, i); provinces++; }
    return provinces;
}

private void dfs(int[][] adj, boolean[] visited, int i) {
    visited[i] = true;                  // mark on entry
    for (int j = 0; j < adj.length; j++)
        if (adj[i][j] == 1 && !visited[j])
            dfs(adj, visited, j);
}
```

**Why it works:**
`visited[]` guarantees each node is expanded once. Every node reachable from seed `i` gets marked in a single DFS, so the outer loop launches exactly one DFS per connected component — the launch count is the component count.

* * *

# 6. Flagship Problem — Making A Large Island (#827)

This is the most compositional flood-fill problem: it chains **region labeling**, **area aggregation**, and a **probe step** that reasons about merging.

**Problem:** In a binary grid you may flip at most one `0` to `1`. Return the size of the largest island possible afterward.

```
Step 1: Flood every island, giving each a unique id (>=2), record id → area    [Component Count + Aggregation]
Step 2: For each 0-cell, look at its 4 neighbors' distinct island ids            [Probe]
Step 3: Candidate size = 1 + sum of areas of the DISTINCT neighboring islands    [Merge reasoning]
Step 4: Answer = max candidate; handle the all-1s grid (no 0 to flip) edge case  [Edge case]
```

```java
public int largestIsland(int[][] grid) {
    int n = grid.length;
    int[][] DIRS = {{-1,0},{1,0},{0,-1},{0,1}};
    Map<Integer, Integer> areaById = new HashMap<>();
    int id = 2;                                  // start ids at 2 (0=water, 1=unlabeled)

    // Step 1: label each island with a unique id and store its area
    for (int r = 0; r < n; r++)
        for (int c = 0; c < n; c++)
            if (grid[r][c] == 1) {
                areaById.put(id, label(grid, r, c, id, DIRS));
                id++;
            }

    // Step 2-3: try flipping each water cell; merge distinct neighbor islands
    int best = areaById.values().stream().max(Integer::compare).orElse(0); // all-1s / no-flip case
    for (int r = 0; r < n; r++)
        for (int c = 0; c < n; c++)
            if (grid[r][c] == 0) {
                Set<Integer> seen = new HashSet<>();
                int size = 1;                    // the flipped cell itself
                for (int[] d : DIRS) {
                    int nr = r + d[0], nc = c + d[1];
                    if (nr < 0 || nr >= n || nc < 0 || nc >= n) continue;
                    int nid = grid[nr][nc];
                    if (nid > 1 && seen.add(nid)) // DISTINCT island only → avoid double add
                        size += areaById.get(nid);
                }
                best = Math.max(best, size);
            }
    return best;
}

// paint the whole island with `id`, return its area
private int label(int[][] grid, int r, int c, int id, int[][] DIRS) {
    if (r < 0 || r >= grid.length || c < 0 || c >= grid.length) return 0;
    if (grid[r][c] != 1) return 0;               // water or already labeled
    grid[r][c] = id;                             // mark with unique id
    int sum = 1;
    for (int[] d : DIRS) sum += label(grid, r + d[0], c + d[1], id, DIRS);
    return sum;
}
```

**Why label with ids instead of a plain visited flag:**
A single `0`-cell can border two *different* islands. If cells only stored "1 = land", you could not tell whether two neighbors belong to the same island (add once) or two islands (add both). Unique ids + a `Set` of seen ids let you sum each distinct island exactly once.

**The two-role split at every 0-cell:**

```
ROLE 1 (probe):  gather the distinct island ids touching this cell
ROLE 2 (merge):  candidate = 1 + Σ area(distinct neighbor islands)
```

* * *

# 7. Key Tricks & Insights

## Trick 1 — Mark visited on ENTRY, never on exit

```java
grid[r][c] = 0;                 // do this BEFORE the 4 recursive calls
dfs(r+1,c); dfs(r-1,c); ...     // neighbors now see it as claimed
```

If you mark after recursing, a neighbor can re-enter the current cell before it is claimed → infinite recursion or double counting.

## Trick 2 — Guard clause at the top absorbs all invalid cases

```java
if (r < 0 || r >= m || c < 0 || c >= n) return;  // bounds
if (grid[r][c] != TARGET) return;                // water / wall / visited
```

Putting bounds + validity at the top means the *caller* never has to pre-check neighbors — you can blindly recurse in all 4 directions.

## Trick 3 — In-place mutation replaces the visited[] array

Overwriting `'1'→'0'` (or land→id) uses the grid itself as the visited set → O(1) extra space besides the stack. Only use a separate `visited[][]` when you are forbidden to mutate the input.

## Trick 4 — Invert "enclosed" into "reachable from border"

Hard: find cells with no escape to the edge. Easy: flood from the edge and mark the escapees; the rest are enclosed. This one inversion cracks #130, #1020, #417.

## Trick 5 — Multi-source flood: seed the queue/stack with many starts

Pacific/Atlantic (#417): instead of asking "can this cell reach the ocean?", flood *from* each ocean's border and record reach. Cells reached by both floods are the answer. Seeding all border cells at once is a multi-source DFS.

## Trick 6 — Direction array keeps the code symmetric

```java
int[][] DIRS = {{-1,0},{1,0},{0,-1},{0,1}};
for (int[] d : DIRS) dfs(r + d[0], c + d[1]);
```

Swapping to 8-directional is a one-line change. It also prevents copy-paste sign bugs across four hand-written calls.

## Trick 7 — Perimeter without a visited array

For Island Perimeter (#463) you can skip DFS entirely: each land cell contributes 4, minus 2 for every shared edge with a right/down land neighbor. But the DFS version returns `4 - (land neighbors)` per cell — a clean Region Aggregation.

* * *

# 8. Common Pitfalls (🚨 HIGH ROI)

## Pitfall 1 — Marking visited too late

```java
// ❌ neighbors re-enter before this cell is claimed → double count / overflow
void dfs(int r, int c) {
    if (invalid(r, c)) return;
    dfs(r+1, c); dfs(r-1, c); dfs(r, c+1); dfs(r, c-1);
    grid[r][c] = 0;   // marked AFTER recursion — too late
}
```

```java
// ✅ claim first, then spread
void dfs(int r, int c) {
    if (invalid(r, c)) return;
    grid[r][c] = 0;   // mark on entry
    dfs(r+1, c); dfs(r-1, c); dfs(r, c+1); dfs(r, c-1);
}
```

## Pitfall 2 — Missing / misordered bounds check

```java
// ❌ reads grid[r][c] before checking bounds → ArrayIndexOutOfBounds
if (grid[r][c] != 1 || r < 0 || c < 0) return;
```

```java
// ✅ bounds FIRST (short-circuit), then it's safe to index
if (r < 0 || r >= m || c < 0 || c >= n) return;
if (grid[r][c] != 1) return;
```

## Pitfall 3 — Counting the component instead of launching one flood

```java
// ❌ increments per land cell, not per island
for (r,c) if (grid[r][c] == 1) { count++; dfs(r,c); }  // count++ fires for every cell? no—
// subtler bug: forgetting dfs entirely, so every cell counts as its own island
for (r,c) if (grid[r][c] == 1) count++;
```

```java
// ✅ one flood per unvisited region, count once per launch
for (r,c) if (grid[r][c] == 1) { dfs(r,c); count++; }
```

## Pitfall 4 — 4 vs 8 directions

```java
// ❌ using diagonals when the problem defines islands as 4-connected
int[][] DIRS = {{-1,-1},{-1,1},{1,-1},{1,1}, ...};  // wrong connectivity → merges separate islands
```

```java
// ✅ match the problem's definition (Number of Islands = 4-directional)
int[][] DIRS = {{-1,0},{1,0},{0,-1},{0,1}};
```

## Pitfall 5 — Double-adding the same island when merging

```java
// ❌ two neighbors belong to the SAME island → area counted twice
int size = 1;
for (int[] d : DIRS) size += areaById.getOrDefault(idOf(nr,nc), 0);
```

```java
// ✅ dedup with a Set of island ids
Set<Integer> seen = new HashSet<>();
for (int[] d : DIRS) if (seen.add(idOf(nr,nc))) size += areaById.get(idOf(nr,nc));
```

## Pitfall 6 — Recursion stack overflow on huge regions

```java
// ❌ a 200x200 all-land grid = 40k-deep recursion → StackOverflowError on some judges
```

```java
// ✅ convert to an explicit stack (iterative DFS) or BFS queue when grids are large
Deque<int[]> stack = new ArrayDeque<>();
stack.push(new int[]{r, c}); grid[r][c] = 0;
while (!stack.isEmpty()) { int[] cell = stack.pop(); /* push valid neighbors, mark on push */ }
```

* * *

# 9. Edge Case Checklist

* Empty grid / `grid.length == 0` or `grid[0].length == 0` → return 0 / no-op
* Single cell grid → island of size 1 if land, 0 if water
* Entire grid is water → 0 components
* Entire grid is land → 1 component (and for #827, no `0` to flip — return total area)
* Region shaped like a long snake → deepest recursion; watch for stack overflow
* Cell on the border → its off-grid neighbors must be handled by the bounds guard, not a crash
* Disconnected single cells → each is its own component
* 4-directional vs 8-directional connectivity — re-read the problem statement
* Must-not-mutate input → use a separate `visited[][]` instead of overwriting

* * *

# 10. Decision Framework (🔥 INTERVIEW WEAPON)

## Step 1: Is the answer about connectivity of a grid or graph?

→ YES → flood fill / graph DFS territory
→ NO → different archetype

## Step 2: Do you need the SHORTEST path or fewest steps?

→ YES → ⛔ NOT DFS flood fill → use BFS (Archetype 13)
→ NO → DFS flood fill is fine (any traversal order works)

## Step 3: What does each flood need to produce?

```
Just "was it reached"?            → void dfs + visited (Component Count / Reachability)
A measurement (area/perimeter)?   → int dfs that sums neighbor returns (Region Aggregation)
A property tied to the border?    → seed floods from edges, invert (Boundary-Anchored)
A color/label per cell?           → dfs carries a color/id argument (Labeling / Bipartite)
```

### DFS Flood Fill Decision Tree

```
Input is a grid or graph with connected regions?
└── YES
    ├── Need shortest path / min steps?           → ⛔ BFS (Archetype 13)
    ├── Count regions / components?               → Grid Component Count / Graph DFS
    │   ├── Grid (char/int matrix)?               → outer loop + void dfs, mark in-place
    │   └── Adjacency list/matrix?                → visited[] + dfs over neighbors
    ├── Measure a region (area/perimeter/gold)?   → Region Aggregation (int dfs)
    ├── Property defined by touching the border?  → Boundary-Anchored Fill (flood from edges)
    │   ├── "surrounded / captured"?              → #130 pattern
    │   ├── "can't walk off the grid"?            → #1020 enclaves
    │   └── "reaches both oceans"?                → #417 multi-source x2, intersect
    ├── Recolor one connected blob?               → single flood from given seed (#733)
    └── 2-colorable / bipartite?                  → Graph DFS carrying a color
```

* * *

# 11. Problem Map Quick Reference

```
PATTERN                     PROBLEMS
────────────────────────────────────────────────────────────────
Grid Component Count        #200, #1254, #1905, #1971, #323
Region Aggregation          #695, #463, #1219, #694, #711
Boundary-Anchored Fill      #130, #417, #1020, #1034
Graph DFS (adjacency)       #547, #841, #1319, #785, #797, #2101, #1466
Single-seed / recolor       #733, #529, #1391, #1559
Compositional / flagship    #827 (label + aggregate + merge)
```

* * *

# 12. Drill Section (Mastery Check)

## Verbal Drill

Explain this archetype in 2 minutes.

Target structure:

1. Flood fill is DFS on a grid/graph that claims every reachable cell exactly once via a visited mark
2. The frame is: guard (bounds + visited + validity) → mark on entry → recurse to all neighbors
3. Four patterns: component count, region aggregation, boundary-anchored fill, and graph DFS on adjacency
4. The hardest variant (Making a Large Island) labels each island with an id + area, then probes each 0-cell to merge distinct neighbors
5. Key insight: mark visited *before* recursing, and invert "enclosed" questions into "reachable from the border"

## Recognition Drill

Given a problem, identify the pattern before coding:

* Grid + "how many regions"? → Grid Component Count
* "Largest / total size of a region"? → Region Aggregation (dfs returns int)
* "Enclosed / surrounded / can't reach edge"? → Boundary-Anchored Fill (flood from border)
* Adjacency matrix + "groups / provinces"? → Graph DFS component count
* "Shortest / fewest steps"? → STOP, that's BFS not this archetype
* "2-colorable / bipartite"? → Graph DFS carrying a color

## Transformation Drill

Convert brute → optimal without coding:

* **Number of Islands**
    * Brute: repeatedly sweep, merging neighbor labels until stable — O((mn)²)
    * Optimal: one DFS per unvisited land cell, mark in place — O(mn)

* **Max Area of Island**
    * Brute: for each cell, BFS its region from scratch every time — redundant re-floods
    * Optimal: single sweep, dfs returns area, mark on entry — O(mn)

* **Surrounded Regions**
    * Brute: for each inner 'O', DFS to test if it can escape to the border — O((mn)²)
    * Optimal: flood once from all border 'O's, invert — O(mn)

* **Number of Provinces**
    * Brute: transitively expand friend sets pairwise until stable — O(n³)
    * Optimal: DFS component count over the adjacency matrix — O(n²)

* **Making a Large Island**
    * Brute: flip each 0, recount the largest island from scratch — O((mn)²)
    * Optimal: label islands + areas once, probe each 0's distinct neighbors — O(mn)

## Primitive Speed Test

Write each from scratch with no reference. Target: < 3 min each, zero bugs.

```
□ numIslands(char[][] grid)              ← Grid Component Count, mark in place
□ maxAreaOfIsland(int[][] grid)          ← Region Aggregation, int dfs
□ floodFill(int[][] image, sr, sc, color)← single seed, guard on original color
□ solve(char[][] board)                  ← Surrounded Regions, border flood + invert
□ findCircleNum(int[][] isConnected)     ← Graph DFS on adjacency matrix
□ dfs grid template with int[][] DIRS    ← direction-array flood
```

* * *

# 13. Problem Approach Template

```
# DFS Flood Fill — Problem Quick Drill

Signal check:
- Why flood fill? (grid/graph connectivity, NOT shortest path)

Pattern:
- □ Grid Component Count
- □ Region Aggregation
- □ Boundary-Anchored Fill
- □ Graph DFS (adjacency)

Grid setup:
- Directions: 4-dir or 8-dir?
- Mark visited: in-place mutation or separate visited[][]?
- What is a "valid neighbor"? (in bounds AND target value AND not visited)

dfs contract:
- Return type: void (mark only) or int (aggregate)?
- What does out-of-bounds / wrong-value return? (usually 0 / no-op)
- When do I mark visited? (ON ENTRY, always)

Edge cases:
- empty grid?
- all water / all land?
- single cell?
- huge region → iterative stack?
```

## Problem Drills

```
1. #200 Number of Islands
Pattern:
- Grid Component Count

Mark: in-place ('1' → '0')

Approach:
1. sweep every cell
2. if grid[r][c] == '1' → dfs to sink the island, count++
3. dfs: bounds guard → if not '1' return → set '0' → recurse 4 dirs

Invariant:
- each launch drains exactly one island; launch count = island count

Return:
- count of flood launches

Edge cases:
- empty grid → 0 ✓
- all water → 0 ✓
- single land cell → 1 ✓


2. #695 Max Area of Island
Pattern:
- Region Aggregation

Mark: in-place (1 → 0)

Approach:
1. sweep; for each land cell best = max(best, area(r,c))
2. area: bounds/water → return 0; mark 0; return 1 + sum of 4 recursions

Invariant:
- each cell adds 1 once; returned value = area of the region

Return:
- max area over all regions

Edge cases:
- no land → 0 ✓
- one big island → its full area ✓


3. #733 Flood Fill
Pattern:
- Single-seed flood

Mark: overwrite with new color (guard against original == new)

Approach:
1. remember startColor = image[sr][sc]
2. if startColor == newColor → return (avoid infinite loop)
3. dfs: if in bounds AND image[r][c] == startColor → set newColor, recurse 4 dirs

Invariant:
- only cells of the original color, 4-connected to seed, get repainted

Return:
- mutated image

Edge cases:
- startColor == newColor → no-op (CRITICAL, else infinite recursion) ✓
- single pixel → just that pixel ✓


4. #130 Surrounded Regions
Pattern:
- Boundary-Anchored Fill

Mark: border-connected 'O' → '#'

Approach:
1. flood from every border cell, 'O' → '#'
2. final sweep: 'O' → 'X' (enclosed), '#' → 'O' (restore safe)

Invariant:
- after step 1, '#' = escapes to border; leftover 'O' = enclosed

Return:
- mutated board

Edge cases:
- all 'O' touching border → nothing captured ✓
- 1-row / 1-col board → every cell is border ✓


5. #547 Number of Provinces
Pattern:
- Graph DFS (adjacency matrix)

Mark: visited[] boolean array

Approach:
1. for each unvisited node i → dfs(i), provinces++
2. dfs(i): visited[i]=true; for j where adj[i][j]==1 && !visited[j] → dfs(j)

Invariant:
- one dfs launch per connected component

Return:
- number of launches = provinces

Edge cases:
- no edges → n provinces ✓
- fully connected → 1 province ✓


6. #827 Making A Large Island
Pattern:
- Label + Region Aggregation + Merge probe

Mark: unique id per island (>= 2), store id → area

Approach:
1. label each island with id, record areaById
2. for each 0-cell: sum areas of DISTINCT neighbor ids + 1
3. answer = max(candidate, largest existing island)

Invariant:
- ids let a 0-cell tell apart same vs different neighboring islands

Return:
- max achievable island size after ≤ 1 flip

Edge cases:
- all 1s (no 0 to flip) → total area ✓
- all 0s → flipping one → size 1 ✓
```
