# 🌲 Archetype 9 — Tree Level Order (BFS)

* * *

# 0. Goal

## What problem class does this solve?

* Problems requiring nodes to be processed **layer by layer**, from root to leaves
* Problems where the answer depends on **which level** a node is on, not just which subtree
* Problems requiring **left-to-right ordering within a level** to be preserved or exploited
* Problems that ask for a **boundary node** (leftmost, rightmost, largest) at each depth
* Problems that **connect** sibling or cousin nodes at the same level via pointer mutation

## What mastery looks like

* Writes the BFS level-order frame in < 2 minutes from scratch, no bugs
* Immediately recognizes the `int size = queue.size()` inner-loop idiom as the level boundary marker
* Knows when to use **index tracking** (width problems) vs. **plain BFS** (snapshot problems)
* Distinguishes 5 named patterns and maps a problem to one within 30 seconds
* Can explain why BFS uses a Queue but DFS uses the call stack (and when to use each)

* * *

# 1. Recognition Signals

## Strong signals

* Input is a `TreeNode` (binary tree or N-ary tree)
* "Level order traversal" / "level by level"
* "Right side view" / "left side view" (rightmost / leftmost node per level)
* "Average of levels", "maximum value per row", "sum of deepest level"
* "Zigzag traversal" (alternating left-right, right-left per level)
* "Connect next right pointers" (populate `next` field linking same-level nodes)
* "Maximum width of binary tree"
* "Check completeness of binary tree"
* "Minimum depth" (first level containing a leaf — early exit on BFS)

## Weak / disguised signals

* "Shortest path in binary matrix" — grid BFS, same BFS frame, different structure
* "Find corresponding node in a clone" — BFS two trees simultaneously
* "Cousins in binary tree" — same level, different parents — BFS tracks parent + depth
* "Complete binary tree inserter" — BFS finds the first incomplete level
* "Even Odd Tree" — BFS collects each level, validates alternating parity rule

## Anti-signals (when NOT to use BFS)

* Need to aggregate up from children → postorder DFS (Archetype 7)
* Need to pass state down from parent → preorder DFS (Archetype 8)
* Need all root-to-leaf paths → preorder DFS with backtracking (Archetype 8)
* Need strongly connected components, cycle detection → graph DFS/BFS (Archetypes 12–13)

* * *

# 2. Cheat Sheet

```
WHEN TO USE:
- Process nodes level by level
- "Right / left side view" (boundary at each depth)
- "Average / max / sum per level" (aggregation per level)
- "Zigzag traversal" (direction alternates per level)
- "Connect next right pointers" (sibling linking within a level)
- "Maximum width" (track virtual column index)
- "Minimum depth" (BFS reaches first leaf before DFS would)

CORE IDEA:
- Use a Queue (LinkedList / ArrayDeque) for FIFO ordering
- Snapshot trick: capture size = queue.size() BEFORE inner loop
  → this is the count of nodes at the CURRENT level
  → add children to queue; they belong to the NEXT level
- The inner loop over [0, size) processes exactly one level

THE 5 PATTERNS:
1. Level Snapshot      → collect all nodes per level into a list
2. Boundary Extract    → take first or last node per level
3. Level Aggregation   → compute sum/max/avg per level
4. Pointer Mutation    → connect next pointers during traversal (no list needed)
5. Index Tracking      → assign virtual column index to each node; width = max - min + 1

TEMPLATE SELECTOR:
- Collect all levels?               → Level Snapshot (return List<List<Integer>>)
- Only rightmost / leftmost?        → Boundary Extract (track last/first polled per level)
- Sum / max per level?              → Level Aggregation (running accumulator per level)
- Connect siblings?                 → Pointer Mutation (use prev node pointer within level)
- Width of tree?                    → Index Tracking (queue stores [node, index] pairs)
- First leaf encountered?           → Minimum Depth (early return when leaf found)

TIME / SPACE:
- All patterns:    O(n) time (every node visited once)
- Space:           O(w) where w = max width of tree
                   Balanced tree:  O(n/2) = O(n) — last level has ~n/2 nodes
                   Skewed tree:    O(1) — only one node per level
- Index tracking:  Risk of integer overflow on wide trees → normalize: subtract leftmost index

TOP 3 TRICKS:
1. Capture size = queue.size() BEFORE the inner loop — this is the level separator
2. For index tracking, offset indices to prevent Long overflow: idx - leftmost at each level
3. For minimum depth: BFS finds shortest path first — return immediately on first leaf

TOP 3 PITFALLS:
1. Forgetting to fix size before inner loop — queue grows as children are added
2. Using DFS for "minimum depth" — DFS explores full paths; BFS exits on first leaf
3. Integer overflow in width calculation — column index can exceed int range; use long
```

* * *

# 3. Core Mental Model

## Core Ideas

* A binary tree has levels. BFS visits each level completely before descending
* The **Queue** enforces FIFO: nodes added left-to-right at level K appear in order at level K+1
* The **inner loop `for (int i = 0; i < size; i++)`** is the level boundary — run it exactly `size` times to exhaust one level
* Everything added to the queue inside the inner loop belongs to the **next** level
* This single frame — snapshot size, loop size times, enqueue children — solves ~85% of level-order problems

## The BFS Frame

```java
Queue<TreeNode> queue = new LinkedList<>();
queue.offer(root);

while (!queue.isEmpty()) {
    int size = queue.size();           // ← snapshot: how many nodes at THIS level
    for (int i = 0; i < size; i++) {
        TreeNode node = queue.poll();
        // --- process node here ---
        if (node.left  != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
    // ← after inner loop: all nodes at this level consumed; next level in queue
}
```

## Why `size = queue.size()` before the loop, not inside?

```
Before inner loop starts:   queue = [A, B]         size = 2
After poll A, add A.left/A.right: queue = [B, C, D]
After poll B, add B.left/B.right: queue = [C, D, E, F]

If you checked queue.size() inside the loop, you'd never stop at the right boundary.
Snapshotting size FIRST ensures you process exactly the nodes of THIS level.
```

## Queue Contents Across Levels

```
        1
       / \
      2   3
     / \   \
    4   5   6

Level 0: queue = [1]           size=1 → poll 1, add 2,3
Level 1: queue = [2,3]         size=2 → poll 2→add 4,5; poll 3→add 6
Level 2: queue = [4,5,6]       size=3 → poll all, add nothing (leaves)
```

## The 5 Patterns at a Glance

```
┌─────────────────────────┬─────────────────────────────────────────────────────┐
│ PATTERN                 │ WHAT IT EXTRACTS                                    │
├─────────────────────────┼─────────────────────────────────────────────────────┤
│ 1. Level Snapshot       │ All node values per level → List<List<Integer>>     │
│ 2. Boundary Extract     │ First or last node per level → List<Integer>        │
│ 3. Level Aggregation    │ Sum / max / avg per level → List<Double>            │
│ 4. Pointer Mutation     │ Connect next pointers within each level             │
│ 5. Index Tracking       │ Virtual column index → max width                    │
└─────────────────────────┴─────────────────────────────────────────────────────┘
```

* * *

# 4. Optimization Ladder (Interview Narrative)

## Step 1 — Brute Force

* Compute depth of every node via DFS: O(n) per node → O(n²) overall
* Group nodes by depth in a HashMap → two passes, O(n) space but verbose
* For width: enumerate all possible column positions for all nodes — O(n²)

## Step 2 — Optimization Insight

What repeating work are we eliminating?

* We are making multiple passes when one BFS sweep captures all per-level information simultaneously
* The Queue naturally delivers nodes in BFS order — we get level membership "for free" from the snapshot trick
* For width: instead of enumerating positions globally, track local index relative to parent → no extra space for level lookup

Core shift:
```
from: DFS traversal → assign depth to each node → group by depth → process
to:   BFS level loop → process each level in-place → O(n) single pass
```

## Step 3 — Optimal Approach

* Key idea: one BFS pass; snapshot `size` at start of each level; process `size` nodes; collect result
* Time: O(n) — every node enqueued and dequeued exactly once
* Space: O(w) — queue holds at most one full level at a time (worst case: all leaves ≈ n/2 nodes)

* * *

# 5. Core Patterns

## Pattern 1 — Level Snapshot

**When to use:**

* Need all node values per level (or some transformation of them)
* Output is `List<List<Integer>>` or similar collection-of-levels
* Problems: #102, #107, #429, #1609

**Mental model:**

```
Each iteration of the outer while loop = one full level
Inner for loop collects that level's values into a temporary list
Add the temporary list to the result after the inner loop completes
```

```java
// ─────────────────────────────────────────────
// PATTERN 1 — Level Snapshot
// When: collect all values per level
// Output: List<List<Integer>>
// ─────────────────────────────────────────────
List<List<Integer>> result = new ArrayList<>();
Queue<TreeNode> queue = new LinkedList<>();
if (root != null) queue.offer(root);

while (!queue.isEmpty()) {
    int size = queue.size();
    List<Integer> level = new ArrayList<>();
    for (int i = 0; i < size; i++) {
        TreeNode node = queue.poll();
        level.add(node.val);
        if (node.left  != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
    result.add(level);
}
return result;
```

**Zigzag variant — flip direction per level:**

```java
boolean leftToRight = true;
// Inside inner loop:
if (leftToRight) level.add(node.val);
else             level.addFirst(node.val);   // or use Deque<Integer>
// After inner loop:
leftToRight = !leftToRight;
```

**Why null-check before initial offer:**
If root is null, offering it would enqueue null, causing NPE on `node.val`. Check before queuing.

* * *

## Pattern 2 — Boundary Extract

**When to use:**

* Need only the first or last node per level (right side view, bottom-left, largest per row)
* Output is `List<Integer>` — one value per level
* Problems: #199, #513, #515

**Mental model:**

```
Inside the inner loop, track the last node polled (for rightmost) or the first (for leftmost)
After the inner loop, add that tracked node's value to the result
```

```java
// ─────────────────────────────────────────────
// PATTERN 2 — Boundary Extract
// When: capture first or last node per level
// Output: List<Integer>
// ─────────────────────────────────────────────
List<Integer> result = new ArrayList<>();
Queue<TreeNode> queue = new LinkedList<>();
if (root != null) queue.offer(root);

while (!queue.isEmpty()) {
    int size = queue.size();
    int boundaryVal = 0;
    for (int i = 0; i < size; i++) {
        TreeNode node = queue.poll();
        boundaryVal = node.val;             // last assigned = rightmost (for right-side view)
        // for leftmost: only assign when i == 0
        if (node.left  != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
    result.add(boundaryVal);
}
return result;
```

**Why this works for right side view:**
In each level, we enqueue children left-to-right. The last `node` polled in a level of size N is the rightmost node at that depth. Overwriting `boundaryVal` each time and reading after the loop gives us exactly the rightmost value.

* * *

## Pattern 3 — Level Aggregation

**When to use:**

* Need to compute a per-level aggregate: sum, max, min, average
* No need to store individual node values — running accumulator suffices
* Problems: #637, #1161, #1302, #515

**Mental model:**

```
Declare a running accumulator (sum, max, etc.) before the inner loop
Accumulate during the inner loop
Compute the final aggregate (e.g., avg = sum / size) after the inner loop
```

```java
// ─────────────────────────────────────────────
// PATTERN 3 — Level Aggregation
// When: compute sum/max/avg per level
// Output: List<Double> (for averages) or int (for max level sum)
// ─────────────────────────────────────────────
List<Double> result = new ArrayList<>();
Queue<TreeNode> queue = new LinkedList<>();
if (root != null) queue.offer(root);

while (!queue.isEmpty()) {
    int size = queue.size();
    double sum = 0;                         // reset per level
    for (int i = 0; i < size; i++) {
        TreeNode node = queue.poll();
        sum += node.val;
        if (node.left  != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
    result.add(sum / size);                 // average; for max: track max sum across outer loop
}
return result;
```

* * *

## Pattern 4 — Pointer Mutation

**When to use:**

* Each node has a `next` pointer; task is to connect all same-level nodes left → right
* No output collection needed — mutation is the goal
* Problems: #116, #117

**Mental model:**

```
Inside the inner loop, maintain a `prev` pointer (the last node processed in this level)
Connect prev.next = current node
After the level is processed, reset prev = null for the next level
```

```java
// ─────────────────────────────────────────────
// PATTERN 4 — Pointer Mutation
// When: connect next pointers within each level
// Operates in-place on the tree
// ─────────────────────────────────────────────
Queue<Node> queue = new LinkedList<>();
if (root != null) queue.offer(root);

while (!queue.isEmpty()) {
    int size = queue.size();
    Node prev = null;
    for (int i = 0; i < size; i++) {
        Node node = queue.poll();
        if (prev != null) prev.next = node;   // link previous to current
        prev = node;
        if (node.left  != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
    // prev.next is null by default — last node in level points to null ✓
}
return root;
```

**Why `prev.next` not `node.next`:**
We're linking each node to the one that comes AFTER it (i.e., the next poll). By maintaining `prev`, when we process `current`, we link `prev → current`. After the loop, the last `prev` has no successor, so its `next` stays null.

* * *

## Pattern 5 — Index Tracking

**When to use:**

* Need to find the horizontal width of the tree (maximum nodes spanning a level)
* Column indices: root = 0; left child of index i = 2i; right child of index i = 2i+1
* Problems: #662, #958

**Mental model:**

```
Store (node, colIndex) pairs in the queue
At each level, width = lastIndex - firstIndex + 1
Normalize by subtracting the leftmost index at the start of each level to prevent Long overflow
```

```java
// ─────────────────────────────────────────────
// PATTERN 5 — Index Tracking
// When: compute tree width per level
// Queue stores [node, colIndex] pairs
// Use long to avoid integer overflow
// ─────────────────────────────────────────────
Queue<long[]> queue = new LinkedList<>();
// queue stores: [node's objectId mapped to index] — use Map or encode differently
// Simpler: use a Queue<int[]> with a separate Map<TreeNode, Long> for index
Map<TreeNode, Long> indexMap = new HashMap<>();
Queue<TreeNode> q = new LinkedList<>();
if (root != null) { q.offer(root); indexMap.put(root, 0L); }

int maxWidth = 0;
while (!q.isEmpty()) {
    int size = q.size();
    long leftmost = indexMap.get(q.peek());   // normalize against leftmost
    long last = leftmost;
    for (int i = 0; i < size; i++) {
        TreeNode node = q.poll();
        long idx = indexMap.get(node) - leftmost;  // normalize to prevent overflow
        last = idx;
        if (node.left  != null) { q.offer(node.left);  indexMap.put(node.left,  2 * idx); }
        if (node.right != null) { q.offer(node.right); indexMap.put(node.right, 2 * idx + 1); }
    }
    maxWidth = (int) Math.max(maxWidth, last + 1);
}
return maxWidth;
```

**Why normalize per level:**
Without normalization, deeply nested right-only trees cause `2^depth` to exceed `Long.MAX_VALUE`. By subtracting the leftmost index at the start of each level, we reset the reference point. The relative width `lastIdx - firstIdx + 1` is the same either way.

* * *

# 6. Flagship Problem — Maximum Width of Binary Tree (#662)

This is the most compositional BFS problem in this archetype. It combines the BFS level frame with virtual index assignment, overflow protection, and index normalization — all three must work together correctly.

**Problem:** Return the maximum width of any level. Width = number of nodes between the leftmost and rightmost node at that level (including null nodes in between).

```
Step 1: Assign root index 0; for any node at index i, left child → 2i, right child → 2i+1
Step 2: At each level, normalize indices by subtracting the leftmost index (prevents overflow)
Step 3: Width at this level = (lastIndex - firstIndex + 1)
Step 4: Track global maximum across all levels
```

```java
public int widthOfBinaryTree(TreeNode root) {
    if (root == null) return 0;

    // Queue of (node, colIndex) — use long for indices
    Queue<TreeNode> nodes = new LinkedList<>();
    Map<TreeNode, Long> indexMap = new HashMap<>();
    nodes.offer(root);
    indexMap.put(root, 0L);

    int maxWidth = 0;

    while (!nodes.isEmpty()) {
        int size = nodes.size();
        long firstIdx = indexMap.get(nodes.peek());  // leftmost of this level
        long lastIdx  = firstIdx;

        for (int i = 0; i < size; i++) {
            TreeNode node = nodes.poll();
            // normalize: subtract firstIdx to reset the range to [0, width-1]
            long idx = indexMap.get(node) - firstIdx;
            lastIdx = idx;

            if (node.left != null) {
                nodes.offer(node.left);
                indexMap.put(node.left, 2 * idx);
            }
            if (node.right != null) {
                nodes.offer(node.right);
                indexMap.put(node.right, 2 * idx + 1);
            }
        }

        maxWidth = (int) Math.max(maxWidth, lastIdx + 1);
    }

    return maxWidth;
}
```

**Why `lastIdx + 1` not `lastIdx - firstIdx + 1`:**
After normalization, `firstIdx` for children is effectively 0 each level. We track `lastIdx` which is already relative to 0. So width = `lastIdx - 0 + 1 = lastIdx + 1`.

**The overflow trap:**
At depth 32, indices reach 2^32 > Int.MAX_VALUE. Without long, this silently wraps around, producing negative indices and wrong widths. Without normalization, even long overflows around depth 63.

**The two-role split:**
```
LOCAL:  assign child indices relative to this node's (normalized) index
GLOBAL: compare lastIdx + 1 against maxWidth after each level
```

* * *

# 7. Key Tricks & Insights

## Trick 1 — Snapshot `size` before the inner loop; never inside

```java
int size = queue.size();          // ✅ captured once — level boundary is fixed
for (int i = 0; i < size; i++) { // ✅ processes exactly THIS level's nodes
    // adding children grows queue, but size is already captured
}
```

If you used `while (!queue.isEmpty())` as the inner condition, you'd consume all levels at once.

## Trick 2 — Null-guard root before initial offer

```java
if (root != null) queue.offer(root);  // ✅ handles empty tree edge case
```

Alternatively, check root first and return early. Either way, never offer null to the queue.

## Trick 3 — Minimum Depth: BFS is strictly better than DFS

```java
// BFS returns immediately on first leaf found — O(first leaf's depth) time
// DFS always traverses the full tree to find min — O(n) every time
while (!queue.isEmpty()) {
    int size = queue.size();
    depth++;
    for (int i = 0; i < size; i++) {
        TreeNode node = queue.poll();
        if (node.left == null && node.right == null) return depth;  // ← early exit
        if (node.left  != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
}
```

## Trick 4 — Zigzag: use a Deque, add to head or tail per level

```java
Deque<Integer> level = new ArrayDeque<>();
// leftToRight level: addLast
// rightToLeft level: addFirst
// After inner loop: result.add(new ArrayList<>(level))
```

Avoids an O(n) `Collections.reverse()` call.

## Trick 5 — Normalize indices to prevent Long overflow in width problems

```java
long firstIdx = indexMap.get(nodes.peek());
// then for each node: long idx = rawIdx - firstIdx;
// children get: 2*idx, 2*idx+1 (relative to this normalized origin)
```

This keeps all indices in [0, level_width - 1] regardless of depth.

## Trick 6 — Cousins check: BFS tracks (node, parent, depth) simultaneously

```java
// Use a wrapper or parallel queues
// At any level, if two target nodes exist with DIFFERENT parents → cousins
```

## Trick 7 — Level order bottom-up: add each level to front of result list

```java
// #107 Binary Tree Level Order Traversal II
LinkedList<List<Integer>> result = new LinkedList<>();
// In outer loop: result.addFirst(level);  ← O(1) instead of reversing at end
```

* * *

# 8. Common Pitfalls (🚨 HIGH ROI)

## Pitfall 1 — Not capturing `size` before the inner loop

```java
// ❌ queue grows as children are added — inner loop never terminates cleanly
while (!queue.isEmpty()) {
    for (int i = 0; i < queue.size(); i++) { // BUG: queue.size() increases each iteration
        TreeNode node = queue.poll();
        if (node.left != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
}
```

```java
// ✅ snapshot size before the loop
while (!queue.isEmpty()) {
    int size = queue.size();  // fixed for this level
    for (int i = 0; i < size; i++) {
        TreeNode node = queue.poll();
        if (node.left  != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
}
```

## Pitfall 2 — Using DFS for minimum depth

```java
// ❌ DFS explores the full right subtree even if left has a closer leaf
int minDepth(TreeNode root) {
    if (root == null) return 0;
    if (root.left == null && root.right == null) return 1;
    if (root.left == null) return 1 + minDepth(root.right);
    if (root.right == null) return 1 + minDepth(root.left);
    return 1 + Math.min(minDepth(root.left), minDepth(root.right));
    // ↑ visits the entire tree. O(n) always.
}
```

```java
// ✅ BFS exits immediately on first leaf — O(depth of closest leaf)
int depth = 0;
while (!queue.isEmpty()) {
    int size = queue.size();
    depth++;
    for (int i = 0; i < size; i++) {
        TreeNode node = queue.poll();
        if (node.left == null && node.right == null) return depth;
        if (node.left  != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
}
```

## Pitfall 3 — Integer overflow in width tracking

```java
// ❌ int overflows around depth 32 — width silently wraps to negative
int leftIdx = indexMap.get(node.left) = 2 * parentIdx;  // int arithmetic → overflow

// ✅ use long throughout
long leftIdx = 2L * parentIdx;
Map<TreeNode, Long> indexMap = new HashMap<>();
```

## Pitfall 4 — Forgetting to normalize indices in width problems

```java
// ❌ without normalization, indices at depth d can reach 2^d → long overflow at depth ~63
indexMap.put(node.left,  2 * parentIdx);
indexMap.put(node.right, 2 * parentIdx + 1);

// ✅ normalize at the start of each level
long firstIdx = indexMap.get(queue.peek());
long normalizedIdx = rawIdx - firstIdx;
indexMap.put(node.left,  2 * normalizedIdx);
indexMap.put(node.right, 2 * normalizedIdx + 1);
```

## Pitfall 5 — Not handling null root before queuing

```java
// ❌ enqueuing null causes NullPointerException at node.val
queue.offer(root);  // root may be null

// ✅ guard before queuing
if (root != null) queue.offer(root);
// or: early return if root == null
```

* * *

# 9. Edge Case Checklist

* `root == null` — return empty list, 0, or null depending on problem
* Single node — one level with one element; no children to enqueue
* All nodes in a single path (fully skewed) — each level has exactly 1 node; width = 1 always
* Complete binary tree — last level may be partially full; width counting must handle gaps
* Very deep tree with index tracking — Long overflow risk; always normalize per level
* Cousins check — same depth, different parent; need to track parent alongside node in queue
* Minimum depth on a right-skewed tree — DFS would scan all; BFS exits at first leaf

* * *

# 10. Decision Framework (🔥 INTERVIEW WEAPON)

## Step 1: Is input a tree and do we need nodes layer by layer?

→ YES → BFS level-order
→ NO → DFS (postorder / preorder depending on direction of information flow)

## Step 2: What do we need from each level?

```
All values?                        → Level Snapshot   (collect into list per level)
Only leftmost / rightmost?         → Boundary Extract (track first/last polled per level)
Aggregate (sum / max / avg)?       → Level Aggregation (running accumulator per level)
Connect sibling pointers?          → Pointer Mutation  (prev pointer within level)
Horizontal width?                  → Index Tracking    (virtual column index)
First leaf encountered?            → Minimum Depth     (early return inside inner loop)
```

## Step 3: Does the problem require index/position information?

→ YES → Index Tracking pattern; use long; normalize per level
→ NO → plain BFS frame

## Step 4: Is output reversed (bottom-up)?

→ YES → use `LinkedList.addFirst()` or `Collections.reverse()` at the end

### Tree Level Order Decision Tree

```
Input is a binary tree (or N-ary tree)?
└── YES
    ├── Need all nodes per level?
    │   ├── All values?                     → Level Snapshot (List<List<Integer>>)
    │   ├── One value per level?
    │   │   ├── Last node (right view)?     → Boundary Extract (overwrite per poll)
    │   │   ├── First node (left/bottom)?   → Boundary Extract (assign on i==0)
    │   │   ├── Max value?                  → Level Aggregation (track running max)
    │   │   └── Average?                    → Level Aggregation (sum / size)
    │   ├── Direction alternates?           → Level Snapshot + Deque + flip flag
    │   └── Bottom-up order?               → Level Snapshot + addFirst to result
    ├── Need to link same-level nodes?      → Pointer Mutation (prev pointer)
    ├── Need horizontal width?              → Index Tracking (long idx, normalize)
    ├── Need first leaf (min depth)?        → BFS with early return on leaf
    └── Need sibling/cousin relationship?   → BFS with parent tracking in queue
```

* * *

# 11. Problem Map Quick Reference

```
PATTERN                    PROBLEMS
────────────────────────────────────────────────────────────────────────
Level Snapshot             #102, #107, #429, #1609, #655
Boundary Extract           #199, #513, #515, #993
Level Aggregation          #637, #1161, #1302, #515
Pointer Mutation           #116, #117, #2415
Index Tracking             #662, #958
Minimum Depth (BFS)        #111
Multi-Source / Near-Tree   #1091 (grid BFS), #1379, #2471
Complete Tree BFS          #919 (inserter), #958 (completeness)
Zigzag                     #103
```

* * *

# 12. Drill Section (Mastery Check)

## Verbal Drill

Explain this archetype in 2 minutes.

Target structure:

1. Level order BFS uses a Queue and processes nodes level-by-level using the snapshot trick
2. The frame: snapshot `size = queue.size()` before the inner loop; process `size` nodes; enqueue their children
3. All problems fall into 5 patterns: snapshot, boundary extract, aggregation, pointer mutation, or index tracking
4. Index tracking for width requires long to prevent overflow and per-level normalization to prevent long overflow on deep trees
5. BFS is preferred over DFS when the answer depends on which level a node is on, or when minimum depth early-exit matters

## Recognition Drill

Given a problem, identify the pattern before coding:

* Output is all levels? → Level Snapshot
* Output is one value per level? → Boundary Extract or Level Aggregation
* Connect `next` pointers? → Pointer Mutation
* Width / completeness? → Index Tracking
* Minimum depth? → BFS with early leaf return

## Transformation Drill

Convert brute → optimal without coding:

* **Level Order**
    * Brute: DFS, track depth per node, HashMap<depth, list> — O(n) but two-concept overhead
    * Optimal: BFS level loop with snapshot — single-concept O(n) with O(w) space

* **Right Side View**
    * Brute: DFS, record rightmost node at each depth in a map — O(n) time, O(n) depth map
    * Optimal: BFS, overwrite a variable with each node polled per level — O(n) time, O(1) per level

* **Maximum Width**
    * Brute: For each level, count all possible positions including nulls — exponential
    * Optimal: BFS with virtual index; normalize per level; width = last - first + 1 — O(n)

* **Minimum Depth**
    * Brute: DFS — must explore all paths O(n) before finding minimum
    * Optimal: BFS — exits the moment the first leaf is encountered — O(first leaf depth * branching factor)

## Primitive Speed Test

Write each from scratch with no reference. Target: < 3 min each, zero bugs.

```
□ levelOrder(TreeNode root)                      ← Level Snapshot; basic BFS frame
□ rightSideView(TreeNode root)                   ← Boundary Extract; track last polled
□ zigzagLevelOrder(TreeNode root)                ← Level Snapshot + Deque + direction flag
□ averageOfLevels(TreeNode root)                 ← Level Aggregation; sum/size per level
□ connect(Node root)                             ← Pointer Mutation; prev pointer
□ widthOfBinaryTree(TreeNode root)               ← Index Tracking; long; normalize
□ minDepth(TreeNode root)                        ← Early exit BFS on first leaf
```

* * *

# 13. Problem Approach Template

```
1. Is this level-order? (process nodes layer by layer)
   → YES if: answer depends on which level a node is at, or left-to-right within a level

2. What do I extract from each level?
   → All values? / Boundary value? / Aggregate? / No extraction (mutation)?

3. Do I need position/index info per node?
   → YES → Index Tracking; use long; normalize per level to prevent overflow
   → NO  → plain BFS frame

4. What is the early-exit condition (if any)?
   → Minimum depth: return immediately on first leaf
   → Cousins: return true/false once target depth is processed
```

```
# Tree Level Order — Problem Quick Drill

Signal check:
- Why BFS? (does answer need level-by-level processing?)

Pattern:
- □ Level Snapshot       → collect all per level
- □ Boundary Extract     → first/last per level
- □ Level Aggregation    → sum/max/avg per level
- □ Pointer Mutation     → connect next pointers
- □ Index Tracking       → width; use long; normalize

Inner loop setup:
- int size = queue.size()   ← snapshot before loop
- for (int i = 0; i < size; i++) { ... }

What goes into the result after the inner loop?
- level list / boundary value / aggregate value / nothing (mutated in-place)

Index tracking?
- YES / NO → use long? → normalize per level?

Early exit condition?
- YES (what triggers it?) / NO

Edge cases:
- null root?
- single node?
- skewed tree (one node per level)?
- very deep tree (index overflow risk)?
```

## Problem Drills

```
1. #102 Binary Tree Level Order Traversal
Pattern:
- Level Snapshot

Null returns: empty list

Approach:
1. Queue offer root (if non-null)
2. While queue not empty:
   a. size = queue.size()
   b. Create level list
   c. Loop size times: poll → add val → offer children
   d. Add level list to result
3. Return result

Invariant:
- After inner loop, queue contains exactly next level's nodes

Edge cases:
- null root → return []


2. #199 Binary Tree Right Side View
Pattern:
- Boundary Extract

Approach:
1. Queue offer root
2. While queue not empty:
   a. size = queue.size()
   b. Loop size times: poll → overwrite lastVal → offer children
   c. Add lastVal to result after loop
3. Return result

Invariant:
- lastVal after inner loop = rightmost node at this level

Edge cases:
- null root → return []
- single node → [root.val] ✓


3. #103 Binary Tree Zigzag Level Order Traversal
Pattern:
- Level Snapshot + direction flag

Approach:
1. Queue offer root; boolean leftToRight = true
2. While queue not empty:
   a. size = queue.size(); Deque<Integer> level = new ArrayDeque<>()
   b. Loop size times: poll → addLast or addFirst based on leftToRight → offer children
   c. result.add(new ArrayList<>(level)); leftToRight = !leftToRight
3. Return result

Invariant:
- Even levels (0,2,...) go left-to-right; odd levels go right-to-left

Edge cases:
- single node: one level, leftToRight=true → addLast ✓


4. #637 Average of Levels in Binary Tree
Pattern:
- Level Aggregation

Approach:
1. Queue offer root
2. While queue not empty:
   a. size = queue.size(); double sum = 0
   b. Loop size times: poll → sum += val → offer children
   c. result.add(sum / size)
3. Return result

Invariant:
- sum / size after inner loop = exact average for this level

Edge cases:
- single node: sum = root.val, size = 1, avg = root.val ✓


5. #116 Populating Next Right Pointers in Each Node
Pattern:
- Pointer Mutation

Approach:
1. Queue offer root (perfect binary tree)
2. While queue not empty:
   a. size = queue.size(); Node prev = null
   b. Loop size times: poll → if prev != null: prev.next = node → prev = node → offer children
   c. (prev.next remains null — last node in level)
3. Return root

Invariant:
- After inner loop, all nodes in this level are linked left → right; last.next = null

Edge cases:
- null root → return null ✓
- leaf level: no children offered, prev.next stays null ✓


6. #662 Maximum Width of Binary Tree
Pattern:
- Index Tracking (long, normalize)

Global: int maxWidth = 0
Approach:
1. Queue offer root; indexMap.put(root, 0L)
2. While queue not empty:
   a. size = queue.size(); firstIdx = indexMap.get(queue.peek()); lastIdx = firstIdx
   b. Loop size times:
      poll → idx = indexMap.get(node) - firstIdx → lastIdx = idx
      → offer left with 2*idx; offer right with 2*idx+1
   c. maxWidth = max(maxWidth, lastIdx + 1)
3. Return maxWidth

Invariant:
- idx is always normalized to [0, level_width-1]
- lastIdx + 1 = width of this level (including null gaps between leftmost and rightmost)

Edge cases:
- null root → 0
- skewed tree: width always 1 ✓
- deep tree: normalization prevents overflow ✓


7. #111 Minimum Depth of Binary Tree
Pattern:
- BFS early exit (Boundary Extract variant)

Approach:
1. Queue offer root; depth = 0
2. While queue not empty:
   a. size = queue.size(); depth++
   b. Loop size times:
      poll → if leaf (no children): return depth
      → offer children
3. Return 0 (unreachable if root non-null)

Invariant:
- BFS guarantees first leaf found is at minimum depth

Edge cases:
- null root → 0
- root is leaf → return 1 ✓
- right-skewed: BFS exits much earlier than DFS would ✓
```
