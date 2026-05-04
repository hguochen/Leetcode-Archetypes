# 🌳 Archetype 7 — Tree DFS Postorder

* * *

# 0. Goal

## What problem class does this solve?

* Problems requiring information from both subtrees before computing a node's answer
* Problems that reduce to: "what does each subtree give me, and how do I combine them?"
* Tree structure validation, transformation, and path computation
* Problems where the answer at a node depends on the answers of its children (subtree DP)

## What mastery looks like

* Recognizes postorder vs preorder vs inorder in < 15 seconds from problem shape
* Can write all 4 core recursive patterns from scratch with no bugs
* Can handle dual-return problems (height + validity simultaneously)
* Knows when to use a global variable vs. return value for tracking best answer
* Can explain WHY the base case returns what it does (not just that it does)

* * *

# 1. Recognition Signals

## Strong signals

* Input is a `TreeNode` (binary tree)
* "Maximum / minimum depth" of a tree
* "Diameter", "longest path", "maximum path sum" through the tree
* "Is the tree balanced / symmetric / valid BST?"
* "Lowest common ancestor" of two nodes
* "Count nodes", "count good nodes", "count paths with sum X"
* "Invert / flatten / serialize" the tree
* "Same tree", "subtree of another tree"

## Weak / disguised signals

* "Distribute" something across a tree (House Robber III, Distribute Coins)
* "Camera / coverage" problem on a tree
* "Construct tree from traversal arrays" (preorder + inorder → tree)
* Anything asking for a property that requires knowing both children first

## Anti-signals (when NOT to use postorder)

* Need nodes in top-down order (level by level) → use BFS / level order instead
* Need to pass information *down* to children → use preorder DFS
* Need to find a path from root to leaf → preorder DFS with backtracking

* * *

# 2. Cheat Sheet

```
WHEN TO USE:
- Need info from both subtrees before computing node's answer
- "Height / depth / diameter / balance" of tree
- "Max/min path sum" through any node
- "LCA" (lowest common ancestor)
- "Validate" tree structure (BST, balanced, symmetric)
- "Transform" tree in-place (invert, flatten, prune)

CORE IDEA:
- Postorder = LEFT → RIGHT → ROOT
- Recurse into children FIRST, then combine at current node
- The call stack IS the traversal — no explicit stack needed
- Every problem asks: "what do I return up, and how do I combine it?"

THE 4 PATTERNS:
1. Aggregate Up         → return scalar (height, count, sum) from each subtree; combine at root
2. Subtree DP           → node's answer = f(left answer, right answer, root.val)
3. Dual Return          → return (height, isValid) or (max, min) together; avoid redundant passes
4. Global + Local       → recursion returns local info; global var tracks best seen so far

TEMPLATE SELECTOR:
- Max/min depth?                → Aggregate Up: max(left, right) + 1
- Diameter / longest path?      → Global + Local: update global with left + right; return max + 1
- Is balanced?                  → Dual Return: return (height, isBalanced)
- Max path sum?                 → Global + Local: update global; return root + max(left, right, 0)
- LCA?                          → Special: return node if found in subtree
- Invert / transform?           → Aggregate Up: recurse first, then rewire at root
- Same tree / subtree?          → Aggregate Up: return bool from both subtrees AND root comparison

TIME / SPACE:
- Most problems:  O(n) time, O(h) space (h = height = call stack depth)
- Balanced tree:  O(n) time, O(log n) space
- Skewed tree:    O(n) time, O(n) space (worst case stack depth)
- Serialization:  O(n) time, O(n) space

TOP 3 TRICKS:
1. Always define what null returns FIRST — it anchors the entire recursion
2. Max path sum: clamp negative contributions to 0 before combining
3. Diameter / path: return local (one-branch max + 1) UP; update global with two-branch sum

TOP 3 PITFALLS:
1. Forgetting to clamp negatives in path sum: left = max(left, 0), right = max(right, 0)
2. Returning height from an "is balanced" check but losing the validity bit
3. Confusing "path through root" (global update) with "path returned upward" (one-sided)
```

* * *

# 3. Core Mental Model

## Core Ideas

* A tree is a recursive structure: every node is the root of its own subtree
* Postorder = children give you answers → you combine them at the current node
* The call stack handles traversal order automatically — trust it
* Every recursive function answers one question: "given the results from my children, what do I return?"
* All solutions fall into: **what do I return?** + **what do I do at this node?**

## The Node Contract

```
class TreeNode {
    int val;
    TreeNode left;   // <- left subtree root
    TreeNode right;  // <- right subtree root
}
```

## The Postorder Frame

```java
ReturnType solve(TreeNode root) {
    // 1. Base case — what does null give us?
    if (root == null) return BASE_CASE;

    // 2. Recurse — get answers from both subtrees FIRST
    ReturnType left  = solve(root.left);
    ReturnType right = solve(root.right);

    // 3. Combine — use left, right, and root.val to produce this node's answer
    return combine(left, right, root.val);
}
```

This frame solves ~80% of tree problems. The rest extend it with a global variable or dual return type.

## The 5 Patterns

```
┌─────────────────────────┬──────────────────────────────────────────────────┐
│ PATTERN                 │ ROLE                                             │
├─────────────────────────┼──────────────────────────────────────────────────┤
│ 1. Aggregate Up         │ Return scalar per subtree; combine at node       │
│ 2. Subtree DP           │ Node's value depends on both children's results  │
│ 3. Dual Return          │ Return multiple values to avoid second pass      │
│ 4. Global + Local       │ Global tracks best; local enables parent combine │
└─────────────────────────┴──────────────────────────────────────────────────┘
```

## Problem Transformation

```
A. Single pattern   → #104 Max Depth (Aggregate), #226 Invert (Aggregate)
B. Two patterns     → #543 Diameter (Global + Local), #110 Balanced (Dual Return)
C. Hard composition → #124 Max Path Sum (Global + Local + clamp)
                      #297 Serialize/Deserialize (preorder + reconstruction)
```

## Visualization

### Postorder Traversal Order

```
        1
       / \
      2   3
     / \
    4   5

Postorder visits: 4 → 5 → 2 → 3 → 1
Children always resolved BEFORE their parent.
```

### Aggregate Up (Max Depth)

```
null → 0        null → 0
       \               /
        1 → max(0,0)+1=1
         \
          2 → max(1,0)+1=2
           \
            3 → max(2,0)+1=3  ← answer bubbles up
```

### Global + Local (Diameter)

```
        1         local returned up = max(left,right) + 1
       / \         global updated   = left + right (through-node path)
      2   3
     / \
    4   5

At node 2:
  left=1 (depth of 4), right=1 (depth of 5)
  global = max(global, 1+1) = 2   ← diameter candidate
  return max(1,1) + 1 = 2         ← give parent the best one-sided depth
```

* * *

# 4. Optimization Ladder (Interview Narrative)

## Step 1 — Brute Force

* For height: recursively compute depth at every node, O(n²) for naive balanced check
* For diameter: at each node, compute left height + right height separately — redundant work
* For path sum: enumerate all root-to-leaf paths, collect sums, find max — O(n) paths × O(h) each

## Step 2 — Optimization Insight

What repeating work are we eliminating?

* We are computing subtree height multiple times for the same node
* We are making multiple passes when one postorder pass captures everything
* The key shift: instead of querying subtrees from the outside, let subtrees *report up*

Core shift:

```
from: call a helper to compute subtree property at each node (redundant traversals)
to:   let each node compute and return its property once; parent combines in O(1)
```

## Step 3 — Optimal Approach

* Key idea: one postorder pass — each node receives answers from children, combines in O(1), returns up
* Time: O(n) — every node visited exactly once
* Space: O(h) — call stack depth equals tree height

* * *

# 5. Core Patterns

```
Pattern 1 Aggregate Up   → return int (or boolean, TreeNode)
Pattern 2 Subtree DP     → return int[]  {include, exclude}
Pattern 3 Dual Return    → return int    (-1 as sentinel)
Pattern 4 Global + Local → instance variable int globalMax; return int (one side)
```

## Pattern 1 — Aggregate Up

**When to use:**

* Return a scalar (height, count, sum, bool) that combines results from both subtrees
* The answer at a node = f(left result, right result, node.val)
* No global state needed

**Mental model:**

```
null returns the identity value for the aggregation:
  - height → 0       (a null tree has depth 0)
  - count  → 0       (a null tree has 0 nodes)
  - sum    → 0       (a null tree contributes 0)
  - bool   → true    (a null tree satisfies most structural properties)
```

```java
// ─────────────────────────────────────────────
// PATTERN 1 — Aggregate Up
// When: return a scalar combining both subtrees
// No choice, no global state
// ─────────────────────────────────────────────
int dfs(TreeNode node) {
    if (node == null) return BASE_CASE;   // 0 for depth/count, true for bool, null for object
    int left  = dfs(node.left);
    int right = dfs(node.right);
    return combine(left, right, node.val); // e.g. Math.max(left, right) + 1
}
```

**Implementations**

```java
// Max depth template — Aggregate Up
int maxDepth(TreeNode root) {
    if (root == null) return 0;
    int left  = maxDepth(root.left);
    int right = maxDepth(root.right);
    return Math.max(left, right) + 1;
}
```

```java
// Invert binary tree template — Aggregate Up (transformation)
TreeNode invertTree(TreeNode root) {
    if (root == null) return null;
    TreeNode left  = invertTree(root.left);   // recurse first
    TreeNode right = invertTree(root.right);
    root.left  = right;                        // then rewire
    root.right = left;
    return root;
}
```

**Why null returns 0 (not -1):**
A null tree has depth 0. A leaf node (both children null) should return 1 (itself).
`max(0, 0) + 1 = 1` ✅. If null returned -1, a leaf would return 0 — off by one everywhere.

* * *

## Pattern 2 — Subtree DP

**When to use:**

* Node's local decision depends on what both subtrees return
* "Rob or not rob this node" style — choice affects what children can do
* Problems with optimal substructure on trees

**Mental model:**

```
At each node, make a decision based on children's results.
Return the best outcome of that decision upward.
```

```java
// ─────────────────────────────────────────────
// PATTERN 2 — Subtree DP
// When: node makes a choice; parent needs to know WHICH choice
// Return int[] {choiceA, choiceB}
// ─────────────────────────────────────────────
int[] dfs(TreeNode node) {
    if (node == null) return new int[]{0, 0};  // [include, exclude] base case
    int[] left  = dfs(node.left);
    int[] right = dfs(node.right);
    int include = node.val + left[1] + right[1];                          // A: constrains children
    int exclude = Math.max(left[0], left[1]) + Math.max(right[0], right[1]); // B: children free
    return new int[]{include, exclude};
}
// call: int[] res = dfs(root); return Math.max(res[0], res[1]);
```

**Implementation**

```java
// House Robber III template — Subtree DP
// Returns [robRoot, skipRoot] — max money for each choice
int[] rob(TreeNode root) {
    if (root == null) return new int[]{0, 0};
    int[] left  = rob(root.left);
    int[] right = rob(root.right);

    int robRoot  = root.val + left[1] + right[1];        // rob here → skip children
    int skipRoot = Math.max(left[0], left[1])             // skip here → children can choose
                 + Math.max(right[0], right[1]);
    return new int[]{robRoot, skipRoot};
}
// call: int[] res = rob(root); return Math.max(res[0], res[1]);
```

**Why return both choices:**
If you return only the max, the parent can't know whether this node was robbed or skipped.
It needs to know *which choice was made* to enforce the no-adjacent constraint.

* * *

## Pattern 3 — Dual Return

**When to use:**

* Need two pieces of information from each subtree simultaneously
* Computing one requires the other (height AND validity, max AND min)
* Avoids a second recursive pass

**Mental model:**

```
Pack multiple values into a single return (int[], boolean[], custom object).
Parent unpacks and uses both pieces.
```

```java
// ─────────────────────────────────────────────
// PATTERN 3 — Dual Return
// When: need 2 values simultaneously to avoid a second pass
// Encode invalidity as sentinel (-1)
// ─────────────────────────────────────────────
int dfs(TreeNode node) {
    if (node == null) return 0;            // 0 = height 0, valid
    int left  = dfs(node.left);
    if (left  == -1) return -1;            // short-circuit: already invalid
    int right = dfs(node.right);
    if (right == -1) return -1;

    if (Math.abs(left - right) > 1) return -1;      // invalid at this node
    return Math.max(left, right) + 1;               // valid: return height
}
// call: return dfs(root) != -1;
```

**Implementation**

```java
// Balanced binary tree template — Dual Return
// Returns height if balanced, -1 if not
int checkHeight(TreeNode root) {
    if (root == null) return 0;
    int left  = checkHeight(root.left);
    if (left == -1) return -1;             // short-circuit: already unbalanced
    int right = checkHeight(root.right);
    if (right == -1) return -1;

    if (Math.abs(left - right) > 1) return -1;   // unbalanced at this node
    return Math.max(left, right) + 1;             // balanced: return height
}
// call: return checkHeight(root) != -1;
```

**Why -1 as sentinel instead of a boolean:**
We need height to check balance, but also need to propagate failure up.
Using -1 encodes "invalid" in the same return channel — no extra boolean needed.
Short-circuit (`if left == -1 return -1`) prunes entire subtrees once invalid.

* * *

## Pattern 4 — Global + Local

**When to use:**

* The best answer spans *both* subtrees (e.g., a path through the current node)
* But you can only return *one* direction upward to the parent
* The answer is the max over all nodes, not just the root

**Mental model:**

```
Two roles for each node:
  LOCAL:  what do I return UP? (one-sided: best single branch)
  GLOBAL: what do I update?    (two-sided: left + right through me)

global = max(global, left + right + root)
return  max(left, right) + 1   ← one side only
```

```java
// ─────────────────────────────────────────────
// PATTERN 4 — Global + Local
// When: best answer spans BOTH subtrees (path through node)
//       but parent can only use ONE side
// ─────────────────────────────────────────────
int globalMax = Integer.MIN_VALUE;  // 0 if values guaranteed non-negative

int dfs(TreeNode node) {
    if (node == null) return 0;
    int left  = Math.max(dfs(node.left),  0);  // clamp negatives
    int right = Math.max(dfs(node.right), 0);
    globalMax = Math.max(globalMax, left + right + node.val); // TWO sides: global update
    return Math.max(left, right) + node.val;                  // ONE side: local return
}
// call: dfs(root); return globalMax;
```

**Implementation**

```java
// Diameter of Binary Tree template — Global + Local
int maxDiameter = 0;

int depth(TreeNode root) {
    if (root == null) return 0;
    int left  = depth(root.left);
    int right = depth(root.right);
    maxDiameter = Math.max(maxDiameter, left + right);  // global update
    return Math.max(left, right) + 1;                    // local return
}
```

```java
// Binary Tree Maximum Path Sum template — Global + Local + clamp
int maxSum = Integer.MIN_VALUE;

int gain(TreeNode root) {
    if (root == null) return 0;
    int left  = Math.max(gain(root.left),  0);  // clamp negatives
    int right = Math.max(gain(root.right), 0);
    maxSum = Math.max(maxSum, left + right + root.val); // global: path through root
    return Math.max(left, right) + root.val;            // local: best one-sided gain
}
```

**Why clamp negatives to 0:**
A negative subtree contribution makes the path *worse*. Clamping to 0 is equivalent to
"don't extend the path into this subtree" — effectively treating it as a new path start.

**Why return one-sided up:**
The parent node will extend the path in its own direction. If you returned both sides,
the parent would create a path that forks twice — invalid (a path has no branches).

* * *

# 6. Flagship Problem — Binary Tree Maximum Path Sum (#124)

This is the hardest single-tree problem and the capstone of this archetype.
It requires **Global + Local** with the **negative clamp** insight.

**Problem:** Find the maximum sum path between any two nodes in the tree (path can start and end anywhere).

```
Step 1: At each node, compute max gain from left subtree (clamp to 0 if negative)
Step 2: At each node, compute max gain from right subtree (clamp to 0 if negative)
Step 3: Update global max with left + right + root.val (path through this node)
Step 4: Return root.val + max(left, right) upward (one-sided — parent extends one way)
```

```java
private int maxSum = Integer.MIN_VALUE;

public int maxPathSum(TreeNode root) {
    gain(root);
    return maxSum;
}

private int gain(TreeNode root) {
    if (root == null) return 0;

    // clamp: only extend into subtree if it adds value
    int left  = Math.max(gain(root.left),  0);
    int right = Math.max(gain(root.right), 0);

    // global: best path that passes through this node (can't return this up)
    maxSum = Math.max(maxSum, left + right + root.val);

    // local: best single-branch extension for parent to build on
    return Math.max(left, right) + root.val;
}
```

**Why `Integer.MIN_VALUE` not `0`:**
All node values can be negative. If the best path is a single node with value -1,
initializing to 0 would incorrectly return 0 instead of -1.

**The two-role split at every node:**

```
GLOBAL role: left + right + root  ← this node is the "bend" in the path
LOCAL role:  max(left, right) + root  ← parent extends the path through one side
```

* * *

# 7. Key Tricks & Insights

## Trick 1 — Define null's return value first; everything else follows

```
Ask: "what is the answer for an empty tree?"
  max depth    → 0   (no nodes = depth 0)
  is balanced  → 0   (height 0, valid)
  path sum     → false
  invert       → null
  same tree    → true (two nulls are "same")
```

Getting this wrong cascades into off-by-one errors everywhere.

## Trick 2 — Clamp negative subtree contributions to 0

```java
int left = Math.max(gain(root.left), 0);
```

Equivalent to: "I'll extend into this subtree only if it helps."
If clamped, the path effectively starts fresh at this node.

## Trick 3 — Diameter: update global with two sides; return one side

```java
maxDiameter = Math.max(maxDiameter, left + right); // TWO sides — the path bends here
return Math.max(left, right) + 1;                  // ONE side — parent extends one way
```

This distinction is the core insight of every "longest path through a node" problem.

## Trick 4 — Use -1 as sentinel for invalid height

```java
if (Math.abs(left - right) > 1) return -1; // invalid
return Math.max(left, right) + 1;           // valid: propagate height
```

Packs validity and height into one integer. Avoids creating a wrapper class or pair.

## Trick 5 — LCA: return the node if found; null if not

```java
if (root == null || root == p || root == q) return root;
TreeNode left  = lca(root.left, p, q);
TreeNode right = lca(root.right, p, q);
if (left != null && right != null) return root; // p and q on opposite sides
return left != null ? left : right;             // both on same side
```

The LCA is the first node where p and q appear in different subtrees (or is p/q itself).

## Trick 6 — Path sum: subtract remaining rather than accumulate

```java
// Clean: converges toward 0
hasPathSum(root.left, remaining - root.val)

// Verbose: requires carrying both target and running sum
hasPathSum(root.left, running + root.val, target)
```

## Trick 7 — Same tree: ALL three conditions must hold

```java
if (p == null && q == null) return true;
if (p == null || q == null) return false;
return p.val == q.val
    && isSameTree(p.left, q.left)
    && isSameTree(p.right, q.right);
```

Both null → true. One null → false. Both non-null → check value AND recurse both sides.

* * *

# 8. Common Pitfalls (🚨 HIGH ROI)

## Pitfall 1 — Returning height from balanced check but losing validity

```java
// ❌ loses validity information
int height(TreeNode root) {
    if (root == null) return 0;
    return Math.max(height(root.left), height(root.right)) + 1;
    // can't detect imbalance — always returns a positive height
}
```

```java
// ✅ encodes invalidity as -1
int checkHeight(TreeNode root) {
    if (root == null) return 0;
    int left = checkHeight(root.left);
    if (left == -1) return -1;
    // ...
}
```

## Pitfall 2 — Not clamping negative gains in path sum

```java
// ❌ including a negative subtree makes path worse
int left  = gain(root.left);
int right = gain(root.right);
maxSum = Math.max(maxSum, left + right + root.val); // wrong when left or right < 0
```

```java
// ✅ clamp: only extend path if subtree adds value
int left  = Math.max(gain(root.left),  0);
int right = Math.max(gain(root.right), 0);
```

## Pitfall 3 — Returning two-sided path sum upward (invalid path)

```java
// ❌ parent can't extend a path that already bends
return left + right + root.val; // wrong — this is the global update, not local return
```

```java
// ✅ return one-sided for parent to extend
return Math.max(left, right) + root.val;
```

## Pitfall 4 — Diameter off-by-one: forgetting +1 in local return

```java
// ❌ returns depth not including current node
return Math.max(left, right);

// ✅ current node adds 1 to the depth
return Math.max(left, right) + 1;
```

## Pitfall 5 — Initializing global max to 0 when values can be negative

```java
// ❌ fails when best path sum is negative
int maxSum = 0;

// ✅ handles all-negative trees
int maxSum = Integer.MIN_VALUE;
```

* * *

# 9. Edge Case Checklist

General tree edge cases to check before coding:

* `root == null` — empty tree (most problems return 0, false, or null)
* Single node — leaf with no children (diameter=0, depth=1, path sum = root.val)
* Single path (fully skewed left or right) — degenerates to a linked list
* All negative values — path sum max should still find correct answer
* p == q in LCA — LCA of a node with itself is the node itself
* p or q is the root in LCA — answer is root immediately
* All nodes same value — dedup problems must not over-remove
* Balanced vs. skewed tree — check space complexity (O(log n) vs O(n))

* * *

# 10. Decision Framework (🔥 INTERVIEW WEAPON)

## Step 1: Does the answer require info from BOTH subtrees?

→ YES → postorder DFS (children resolve first)
→ NO → consider preorder (pass info down) or BFS (level-by-level)

## Step 2: What does each recursive call return?

```
A single scalar?                    → Aggregate Up (Pattern 1)
A decision based on children?       → Subtree DP (Pattern 2)
Two values at once?                 → Dual Return (Pattern 3)
Local info + updating a global?     → Global + Local (Pattern 4)
State passed downward?              → ⛔ NOT postorder → see Archetype 10 (Path Tracking)
```

## Step 3: What does null return?

```
Depth / height problems?  → 0
Count problems?           → 0
Bool problems?            → true (empty tree satisfies most properties)
Sum problems?             → 0
Object problems?          → null
```

## Step 4: Is there a global max/min to track?

→ YES → declare instance variable; update inside recursion; query after
→ NO → the return value of `solve(root)` IS the answer

### Tree DFS Decision Tree

```
Input is a binary tree?
└── YES
    ├── Need info from both subtrees first?         → Postorder DFS
    │   ├── Return one scalar?                      → Aggregate Up
    │   │   ├── Height / depth?                     → max(left,right) + 1; null→0
    │   │   ├── Count / sum?                        → left + right + f(root); null→0
    │   │   └── Transform (invert)?                 → recurse, then rewire
    │   ├── Return depends on choice at node?       → Subtree DP (return array/pair)
    │   ├── Need height + validity together?        → Dual Return (-1 sentinel)
    │   ├── Longest path / max sum through node?    → Global + Local (clamp negatives)
    ├── Need to pass info DOWN to children?         → ⛔ NOT postorder → Archetype 10 (Path Tracking)
    └── Need level-by-level?                        → BFS / Level Order
```

* * *

# 11. Problem Map Quick Reference

```
PATTERN                    PROBLEMS
────────────────────────────────────────────────────────────────
Aggregate Up               #104, #111, #226, #100, #101, #572,
                           #617, #700, #450
Subtree DP                 #337 (House Robber III), #968, #979
Dual Return                #110 (Balanced), #98 (Validate BST)
Global + Local             #543 (Diameter), #124 (Max Path Sum)
Path Tracking              → see Archetype 10
LCA Pattern                #235 (BST LCA), #236 (Binary Tree LCA)
Serialization              #297
3-Pattern Composition      #124 (Global + Local + clamp)
                           #297 (preorder + queue reconstruction)
                           #337 (Subtree DP + dual return)
```

* * *

# 12. Drill Section (Mastery Check)

## Verbal Drill

Explain this archetype in 2 minutes.

Target structure:

1. Postorder DFS solves tree problems where each node's answer depends on its children
2. The frame is: base case (null) → recurse left → recurse right → combine
3. Most problems are one of 4 patterns: aggregate up, subtree DP, dual return, global+local
4. The hardest problems (max path sum, diameter) use global+local: update global with two-sided path, return one-sided upward
5. Always define null's return value first — it anchors the entire solution

## Recognition Drill

Given a problem, identify the pattern before coding:

* Is info needed from both children? → postorder
* Return one number combining children? → aggregate up
* Node makes a choice based on children? → subtree DP
* Need two things returned at once? → dual return
* Best answer spans both subtrees but parent can only use one? → global + local

## Transformation Drill

Convert brute → optimal without coding:

* **Max Depth**
    * Brute: BFS layer by layer, count levels — O(n) time, O(n) space
    * Optimal: postorder `max(left, right) + 1` — O(n) time, O(h) space

* **Diameter**
    * Brute: at each node, compute left height + right height via separate calls — O(n²)
    * Optimal: one postorder pass, global updated at each node — O(n)

* **Balanced Binary Tree**
    * Brute: for each node, call height(left) and height(right) separately — O(n²)
    * Optimal: dual return — compute height and validity simultaneously in one pass — O(n)

* **Binary Tree Maximum Path Sum**
    * Brute: enumerate all paths, compute sums — O(n²) or worse
    * Optimal: global + local + clamp — one pass, O(n)

* **LCA**
    * Brute: for each node, find paths from root to p and q, compare paths — O(n²)
    * Optimal: postorder — if p and q found in different subtrees, current node is LCA — O(n)

## Primitive Speed Test

Write each from scratch with no reference. Target: < 3 min each, zero bugs.

```
□ maxDepth(TreeNode root)
□ invertTree(TreeNode root)
□ isSameTree(TreeNode p, TreeNode q)
□ isBalanced(TreeNode root)          ← use -1 sentinel
□ diameterOfBinaryTree(TreeNode root)
□ maxPathSum(TreeNode root)          ← hardest; clamp + global
□ lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q)
```

* * *

# 13. Problem Approach Template

```
1. Is this post-order? (need children before parent)
   → YES if: parent's computation depends on children's results

2. What does the parent need from each child?
   → List it out explicitly

3. Can one number capture it?
   → YES → single int + optional global
   → NO  → int[]{a, b} or custom class

4. What is the through-path formula?
   → Same property both sides: left + right (+ adjustment)
   → Different properties: property_A + property_B - 1
```

```
# Tree DFS Postorder — Problem Quick Drill

Signal check:
- Why postorder? (does answer need both subtrees first?)

Pattern:
- □ Aggregate Up
- □ Subtree DP
- □ Dual Return
- □ Global + Local

What does null return?
- null → ___

What does a leaf return?
- leaf → ___

Global variable needed?
- YES / NO

Return type:
- ___

Combine step (what does this node return to parent?):
- f(left, right, root.val) = ___

Edge cases:
- empty tree?
- single node?
- all negative values?
```

## Problem Drills

```
1. #104 Maximum Depth of Binary Tree
Pattern:
- Aggregate Up

Null returns: 0

Approach:
1. if null → return 0
2. left  = maxDepth(root.left)
3. right = maxDepth(root.right)
4. return max(left, right) + 1

Invariant:
- return value = height of subtree rooted at current node

Edge cases:
- null root → return 0 ✓
- single node → max(0,0)+1 = 1 ✓


2. #226 Invert Binary Tree
Pattern:
- Aggregate Up (transformation)

Null returns: null

Approach:
1. if null → return null
2. left  = invertTree(root.left)
3. right = invertTree(root.right)
4. root.left = right; root.right = left
5. return root

Invariant:
- after call, entire subtree rooted at root is mirrored

Edge cases:
- null → return null ✓
- leaf → swap null/null, no-op ✓


3. #543 Diameter of Binary Tree
Pattern:
- Global + Local

Global: int maxDiameter = 0
Null returns: 0 (depth of null tree)

Approach:
1. if null → return 0
2. left  = depth(root.left)
3. right = depth(root.right)
4. maxDiameter = max(maxDiameter, left + right)  ← TWO sides
5. return max(left, right) + 1                    ← ONE side

Invariant:
- return value = max depth of subtree (one branch)
- global = longest path seen (through any node)

Edge cases:
- single node: left=0, right=0 → diameter=0 ✓
- path goes through root: left+right captured at root ✓


4. #110 Balanced Binary Tree
Pattern:
- Dual Return (-1 sentinel)

Null returns: 0 (height 0, valid)

Approach:
1. if null → return 0
2. left  = checkHeight(root.left);  if left == -1 → return -1
3. right = checkHeight(root.right); if right == -1 → return -1
4. if |left - right| > 1 → return -1 (unbalanced here)
5. return max(left, right) + 1

Invariant:
- return value = height if subtree is balanced, -1 if not
- short-circuit prunes entire subtrees once invalid found

Edge cases:
- empty tree: checkHeight(null) = 0 → valid ✓
- skewed tree: every node triggers |left-right| > 1 check ✓


5. #124 Binary Tree Maximum Path Sum
Pattern:
- Global + Local + clamp

Global: int maxSum = Integer.MIN_VALUE
Null returns: 0

Approach:
1. if null → return 0
2. left  = max(gain(root.left),  0)  ← clamp
3. right = max(gain(root.right), 0)  ← clamp
4. maxSum = max(maxSum, left + right + root.val)  ← global
5. return max(left, right) + root.val              ← local

Invariant:
- clamping ensures negative subtrees are ignored
- global updated with best "bend" at current node
- local returns best one-sided contribution for parent

Edge cases:
- all negative: min value node is the answer; Integer.MIN_VALUE init handles ✓
- single node: left=0, right=0 → maxSum = root.val ✓


6. #236 Lowest Common Ancestor of Binary Tree
Pattern:
- Aggregate Up (special node-return variant)

Null returns: null

Approach:
1. if null || root == p || root == q → return root
2. left  = lca(root.left,  p, q)
3. right = lca(root.right, p, q)
4. if left != null && right != null → return root  (p and q on opposite sides)
5. return left != null ? left : right

Invariant:
- return non-null iff p or q found in subtree
- root is LCA when p and q split across left and right subtrees

Edge cases:
- p is ancestor of q: p found first → returned up before reaching q ✓
- p == q: first match returned immediately ✓


7. #337 House Robber III
Pattern:
- Subtree DP (dual return)

Null returns: [0, 0]

Approach:
1. if null → return [0, 0]
2. left  = rob(root.left)   → [robLeft, skipLeft]
3. right = rob(root.right)  → [robRight, skipRight]
4. robRoot  = root.val + left[1] + right[1]
   skipRoot = max(left[0], left[1]) + max(right[0], right[1])
5. return [robRoot, skipRoot]

Invariant:
- [0] = best when this node IS robbed
- [1] = best when this node is SKIPPED (children can choose freely)
- parent uses both values to make its own choice

Edge cases:
- leaf: left=[0,0], right=[0,0] → rob=root.val, skip=0 ✓
- single path: adjacent nodes cannot both be robbed ✓
```
