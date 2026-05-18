# 🌲 Archetype 8 — Tree DFS Preorder (Path Tracking)

* * *

# 0. Goal

## What problem class does this solve?

* Problems involving **root-to-leaf** paths (path must start at the root and end at a leaf)
* Problems where a decision is made **at the leaf**, not at internal nodes
* Problems requiring the current path to be **collected or accumulated** as you descend
* Problems where state must be **passed downward** from parent to child
* Path-encoded number/string problems (encode the root-to-leaf path as a single value)
* Top-down constraint propagation (pass a parent's value or constraint into children)

## What mastery looks like

* Distinguishes path tracking (top-down) from postorder (bottom-up) in < 15 seconds
* Can write the boolean variant and the collection variant from scratch with zero bugs
* Knows exactly why backtracking is required for mutable state but not for primitives or immutable strings
* Can explain why the leaf check (`left == null && right == null`) is mandatory
* Knows when to subtract vs. accumulate, and when to use XOR or bit-shift for path encoding

* * *

# 1. Recognition Signals

## Strong signals

* "Root-to-leaf path" anywhere in the problem statement
* "Does a path exist with sum X?" — boolean answer
* "Return all paths with sum X" — collection answer
* "Find all root-to-leaf paths" — enumerate all paths
* Path constraint starts at root and ends at a leaf
* "Construct a number from root to leaf" — digit encoding
* "Sum of all root-to-leaf numbers"
* "Minimum string from leaf to root" — reversed path accumulation

## Weak / disguised signals

* "Path sum" — check if it says root-to-leaf vs. any-node-to-any-node
  * Root-to-leaf → Path Tracking (this archetype)
  * Any node to any node → Postorder Global + Local (Archetype 7)
* "Max difference between ancestor and descendant" — top-down state (pass max/min seen so far)
* "Count nodes with even grandparent" — pass parent/grandparent down as state
* "Traverse" with an accumulator passed down

## Anti-signals (when NOT to use path tracking)

* Path can start and end **anywhere** → Postorder Global + Local (#124, #543)
* Answer requires info from **both subtrees** at each node → Postorder (#104, #110)
* No state needs to be passed down → Postorder
* Need level-by-level processing → BFS / Level Order (Archetype 9)

* * *

# 2. Cheat Sheet

```
WHEN TO USE:
- Path must START at root and END at a leaf
- State (running sum, current path list, encoded number) flows DOWN via parameters
- Decision is made when you REACH a leaf
- You need to track "what came before this node" (parent value, grandparent, max seen)

CORE IDEA:
- Preorder DFS: process node BEFORE recursing into children
- State accumulates as you go deeper
- For collection: backtrack (undo state) after both recursive calls return
- For boolean: no backtracking needed (primitive state passed by value)

THE 4 PATTERNS:
1. Boolean       → does a valid path exist? return true/false; no backtracking
2. Collection    → collect all valid paths; backtrack after each recursive call
3. Encoding      → encode path as number or string (multiply/shift/concat then check at leaf)
4. State Prop    → pass parent/ancestor info down; answer computable at any node

TEMPLATE SELECTOR:
- "Does a root-to-leaf path sum to X?"     → Pattern 1 (Boolean)
- "Return all root-to-leaf paths"          → Pattern 2 (Collection + backtrack)
- "Sum of all root-to-leaf numbers"        → Pattern 3 (Encoding)
- "Max diff between node and ancestor"     → Pattern 4 (State Propagation)

TIME / SPACE:
- Time:  O(n) — every node visited once
- Space: O(h) for call stack + O(h) for path list (h = tree height)
- Path collection: O(n·h) for storing all paths in worst case
- Balanced tree:  O(n) time, O(log n) stack space
- Skewed tree:    O(n) time, O(n) stack space

TOP 3 TRICKS:
1. Leaf check is ALWAYS required: node.left == null && node.right == null
2. Backtrack only when state is mutable (List, StringBuilder) — not for primitives or String
3. Snapshot before backtracking: result.add(new ArrayList<>(path)) — NOT result.add(path)

TOP 3 PITFALLS:
1. Missing leaf check: matching remaining == 0 at an internal node → false positive
2. Adding path reference instead of copy: path.add(path) vs path.add(new ArrayList<>(path))
3. Missing backtrack: path grows without bound; sibling subtrees see wrong prefix
```

* * *

# 3. Core Mental Model

## The Directional Split

```
POSTORDER (Archetype 7)              PATH TRACKING (Archetype 8)
─────────────────────────            ─────────────────────────────
Information flows UP                 Information flows DOWN
Children computed FIRST              Current node processed FIRST
Combine at parent                    Decide at leaf
No extra parameters                  State passed in parameters
T helper(node)                       void/bool helper(node, state, ...)

WHEN: need both children's results   WHEN: need "what came before me"
```

## Traversal Order

```
        1
       / \
      2   3
     / \
    4   5

Preorder  visits: 1 → 2 → 4 → 5 → 3   (root FIRST, then children)
Postorder visits: 4 → 5 → 2 → 3 → 1   (children FIRST, then root)

Preorder = you carry a backpack DOWN.
Postorder = children hand you things UP.
```

## The Preorder Frame

```java
// Boolean variant — state is primitive → no backtracking
void/bool dfs(TreeNode node, int remaining) {
    if (node == null) return false;
    if (node.left == null && node.right == null)    // leaf: make the decision HERE
        return node.val == remaining;
    return dfs(node.left,  remaining - node.val)
        || dfs(node.right, remaining - node.val);
}

// Collection variant — state is mutable → backtracking required
void dfs(TreeNode node, int remaining, List<Integer> path, List<List<Integer>> result) {
    if (node == null) return;
    path.add(node.val);                              // preorder: add BEFORE recursing
    if (node.left == null && node.right == null && node.val == remaining)
        result.add(new ArrayList<>(path));           // SNAPSHOT before backtrack
    dfs(node.left,  remaining - node.val, path, result);
    dfs(node.right, remaining - node.val, path, result);
    path.remove(path.size() - 1);                   // backtrack: UNDO after both calls
}
```

## Why the Leaf Check is Mandatory

```
If you check remaining == 0 at ANY node (not just leaf), you'll falsely match
internal nodes where the remaining path still has more nodes.

Example: target = 5, tree = [5, 4]
  - At root (val=5): remaining = 0 — but this path is NOT root-to-LEAF
  - The leaf is node 4 — path 5→4 sums to 9, not 5
  - Without the leaf check, root would incorrectly return true

Fix: check (node.left == null && node.right == null) BEFORE checking val == remaining
```

## Why Backtracking is Required for Collection

```
Boolean path: remaining is an int (primitive) → each call gets its own COPY
  → no mutation, nothing to undo

Collection path: path is a List (object reference) → all calls share ONE LIST
  → after recursing left subtree, the current node is still in the list
  → before recursing right subtree, the list already has wrong content
  → you must remove the current node: path.remove(path.size() - 1)

Backtracking = respecting the call stack:
  add node → recurse left → recurse right → remove node
  The node is "alive" exactly while this stack frame is active.
```

## Backtracking Visualization

```
Tree: root(5) → left(4,target=9) → left(11,target=5)
                               → right(8)

Call stack trace:
  dfs(5, 9)       → path=[5]
    dfs(4, 4)     → path=[5,4]
      dfs(11,-7)  → path=[5,4,11]  LEAF — 11 ≠ -7, no match
                    path=[5,4]     ← backtrack: remove 11
    path=[5]       ← backtrack: remove 4
    dfs(8, 1)     → path=[5,8]    LEAF — 8 ≠ 1, no match
                    path=[5]       ← backtrack: remove 8
```

## Pattern Decision Map

```
┌────────────────────────────┬─────────────────────────────────────────────────┐
│ PATTERN                    │ ROLE                                            │
├────────────────────────────┼─────────────────────────────────────────────────┤
│ 1. Boolean                 │ Root-to-leaf existence check; no backtrack      │
│ 2. Collection              │ All valid paths; backtrack on mutable state     │
│ 3. Encoding                │ Build number/string from path; check at leaf    │
│ 4. State Propagation       │ Pass ancestor info down; answer at any node     │
└────────────────────────────┴─────────────────────────────────────────────────┘
```

* * *

# 4. Optimization Ladder (Interview Narrative)

## Step 1 — Brute Force

* For boolean path: enumerate all root-to-leaf paths into a list, then check each sum — O(n·h)
* For collection: store all paths upfront, filter by sum — O(n·h) time + space
* For sum of numbers: convert each path to string, parse to int, sum all — O(n·h)

## Step 2 — Optimization Insight

What repeating work are we eliminating?

* We are re-traversing the tree once per path — unnecessary
* All paths share common prefixes; we should compute prefix sums once on the way down
* The key shift: carry the running sum/path downward instead of building all paths first

Core shift:

```
from: collect all root-to-leaf paths, then process each separately (O(n·h) passes)
to:   single DFS pass; carry state downward; make the decision at the leaf in O(1)
```

## Step 3 — Optimal Approach

* Key idea: one DFS pass — each node receives state from its parent, processes itself, passes state to children
* Boolean: O(n) time, O(h) space (call stack only)
* Collection: O(n) time, O(n·h) space (unavoidable — must store all paths in the result)
* Encoding: O(n) time, O(h) space

* * *

# 5. Core Patterns

## Pattern 1 — Boolean (Has Path Sum)

**When to use:** yes/no question about whether a valid root-to-leaf path exists.

**Mental model:** Subtract the current node's value from the remaining target as you descend. At the leaf, check if remaining equals the leaf's value (i.e., the running deficit is exactly paid off).

```java
// ─────────────────────────────────────────────
// PATTERN 1 — Boolean
// No mutation → no backtracking needed
// Primitive state: passed by value each call
// ─────────────────────────────────────────────
boolean hasPathSum(TreeNode root, int targetSum) {
    if (root == null) return false;
    if (root.left == null && root.right == null)   // leaf: make the decision
        return root.val == targetSum;
    return hasPathSum(root.left,  targetSum - root.val)
        || hasPathSum(root.right, targetSum - root.val);
}
```

**Why no backtracking:** `targetSum` is an int (primitive). Java passes it by value — each recursive call gets its own copy. Subtracting `root.val` only modifies that frame's local copy. Nothing to undo.

**Why subtract rather than accumulate:**

```java
// Subtract — converges toward 0; leaf check: val == remaining
hasPathSum(root.left, targetSum - root.val)

// Accumulate — requires two params (running sum + original target)
hasPathSum(root.left, running + root.val, targetSum) // more verbose; same O(n)
```

Subtraction keeps the function signature to one state parameter.

* * *

## Pattern 2 — Collection (All Valid Paths)

**When to use:** collect every root-to-leaf path that satisfies a condition.

**Mental model:** Same as boolean but with a shared mutable path list. Every node adds itself on the way in, and removes itself on the way out.

```java
// ─────────────────────────────────────────────
// PATTERN 2 — Collection + Backtracking
// Shared mutable path list → must backtrack
// ─────────────────────────────────────────────
List<List<Integer>> pathSum(TreeNode root, int targetSum) {
    List<List<Integer>> result = new ArrayList<>();
    dfs(root, targetSum, new ArrayList<>(), result);
    return result;
}

void dfs(TreeNode node, int remaining, List<Integer> path, List<List<Integer>> result) {
    if (node == null) return;
    path.add(node.val);                                           // add BEFORE recursing
    if (node.left == null && node.right == null && node.val == remaining)
        result.add(new ArrayList<>(path));                        // SNAPSHOT the path
    dfs(node.left,  remaining - node.val, path, result);
    dfs(node.right, remaining - node.val, path, result);
    path.remove(path.size() - 1);                                 // backtrack: remove AFTER both calls
}
```

**Why backtrack:** `path` is an object reference passed to all frames. After recursing left, the current node is still in the list. Without removing it, the right subtree will see a stale prefix. Removing at the end restores the list to its state before this call.

**Why snapshot:**

```java
result.add(path);                    // ❌ adds a REFERENCE — will be emptied by backtracking
result.add(new ArrayList<>(path));   // ✅ creates a COPY of the current path state
```

* * *

## Pattern 3 — Encoding (Path as Number or String)

**When to use:** the path from root to leaf encodes a value (number via digit shift, string concatenation, XOR accumulation).

**Mental model:** On the way down, transform the running encoded value by incorporating the current node. At the leaf, the encoded value is complete — record or sum it.

```java
// ─────────────────────────────────────────────
// PATTERN 3 — Encoding
// Build a number from root to leaf
// State: primitive (int) or immutable (String) → no backtrack
// ─────────────────────────────────────────────

// #129 Sum Root to Leaf Numbers — decimal encoding
int sumNumbers(TreeNode root) {
    return dfs(root, 0);
}

int dfs(TreeNode node, int currentNum) {
    if (node == null) return 0;
    currentNum = currentNum * 10 + node.val;              // encode: shift left + add digit
    if (node.left == null && node.right == null)
        return currentNum;                                 // leaf: return the complete number
    return dfs(node.left, currentNum) + dfs(node.right, currentNum);
}

// #1022 Sum of Root To Leaf Binary Numbers — binary encoding
int sumRootToLeaf(TreeNode root) {
    return dfs(root, 0);
}

int dfs(TreeNode node, int currentNum) {
    if (node == null) return 0;
    currentNum = (currentNum << 1) | node.val;            // binary: shift left 1 bit + add bit
    if (node.left == null && node.right == null)
        return currentNum;
    return dfs(node.left, currentNum) + dfs(node.right, currentNum);
}
```

**Why no backtrack:** `currentNum` is a primitive (int). Each call receives its own copy. The shift/add only modifies the local frame — nothing shared to undo.

**Key mental model difference from Pattern 1:** Pattern 1 checks a condition at the leaf. Pattern 3 *returns* the encoded value from the leaf and aggregates upward (sum or min).

* * *

## Pattern 4 — State Propagation (Top-Down Constraint)

**When to use:** the answer at a node depends on something from its ancestor (max seen so far, grandparent value, parity of depth). The state flows down and the decision happens at each node (not just leaves).

**Mental model:** Pass the ancestor's relevant state as a parameter. Each node uses that state for its computation and passes updated state to children.

```java
// ─────────────────────────────────────────────
// PATTERN 4 — State Propagation
// Pass ancestor state down; decision at each node
// ─────────────────────────────────────────────

// #1026 Maximum Difference Between Node and Ancestor
int maxAncestorDiff(TreeNode root) {
    return dfs(root, root.val, root.val);
}

int dfs(TreeNode node, int minSoFar, int maxSoFar) {
    if (node == null) return maxSoFar - minSoFar;        // leaf exhausted: return max diff seen
    minSoFar = Math.min(minSoFar, node.val);
    maxSoFar = Math.max(maxSoFar, node.val);
    int left  = dfs(node.left,  minSoFar, maxSoFar);
    int right = dfs(node.right, minSoFar, maxSoFar);
    return Math.max(left, right);
}

// #1315 Sum of Nodes with Even-Valued Grandparent
int sumEvenGrandparent(TreeNode root) {
    return dfs(root, -1, -1);                            // pass parent + grandparent down
}

int dfs(TreeNode node, int parent, int grandparent) {
    if (node == null) return 0;
    int sum = 0;
    if (grandparent % 2 == 0) sum += node.val;           // decision using ancestor state
    sum += dfs(node.left,  node.val, parent);
    sum += dfs(node.right, node.val, parent);
    return sum;
}
```

**Key difference from Pattern 1/2:** Decision is NOT restricted to leaves. Any node can contribute to the answer based on its ancestor's value.

**Why no backtrack:** All state parameters are primitives (int). Java passes by value — children get their own copies. The parent's values are unchanged when sibling's call runs.

* * *

# 6. Flagship Problem — Path Sum II (#113)

This is the canonical collection problem and the most important problem in this archetype. It forces mastery of all three mechanics simultaneously: leaf check, snapshot, and backtracking.

**Problem:** Given a binary tree and a target sum, return all root-to-leaf paths where the sum equals the target.

```
Tree:          target = 22
        5
       / \
      4   8
     /   / \
    11  13   4
   /  \     / \
  7    2   5   1

Valid paths: [5,4,11,2] and [5,8,4,5]
```

```
Step 1: At each node, add its value to the path list (preorder)
Step 2: At each node, subtract its value from remaining
Step 3: At a leaf, check if remaining == 0 (equivalently: node.val == remaining)
Step 4: If match → snapshot the path into result
Step 5: After both recursive calls → remove current node (backtrack)
```

```java
class Solution {
    public List<List<Integer>> pathSum(TreeNode root, int targetSum) {
        List<List<Integer>> result = new ArrayList<>();
        dfs(root, targetSum, new ArrayList<>(), result);
        return result;
    }

    private void dfs(TreeNode node, int remaining,
                     List<Integer> path, List<List<Integer>> result) {
        if (node == null) return;

        // Step 1: preorder — add current node to path BEFORE recursing
        path.add(node.val);

        // Step 2: check at leaf only
        if (node.left == null && node.right == null && node.val == remaining) {
            result.add(new ArrayList<>(path));    // Step 3: SNAPSHOT — not a reference
        }

        // Step 4: recurse into children with reduced remaining
        dfs(node.left,  remaining - node.val, path, result);
        dfs(node.right, remaining - node.val, path, result);

        // Step 5: backtrack — undo this node's addition before returning to parent
        path.remove(path.size() - 1);
    }
}
```

**The three mechanics and why each is necessary:**

```
1. LEAF CHECK (node.left == null && node.right == null)
   Why: an internal node with val == remaining is NOT a valid path endpoint.
   Path sum is root-TO-LEAF — the path must reach a leaf.

2. SNAPSHOT (new ArrayList<>(path))
   Why: path is a shared List object. result.add(path) adds a reference.
   By the time the caller reads the result, path has been backtracked to empty.
   Snapshot freezes the current state permanently.

3. BACKTRACK (path.remove(path.size() - 1))
   Why: after recursing into both subtrees, this node must be removed.
   Without it, the parent's path still has this node → wrong prefix for
   the parent's sibling subtree.
```

**Trace through the valid path [5,4,11,2]:**

```
dfs(5, 22)  path=[5]
  dfs(4, 17) path=[5,4]
    dfs(11, 13) path=[5,4,11]
      dfs(7, 2)  path=[5,4,11,7]  LEAF: 7≠2 → no match; backtrack → [5,4,11]
      dfs(2, 2)  path=[5,4,11,2]  LEAF: 2==2 → MATCH → snapshot [5,4,11,2]
                                  backtrack → [5,4,11]
    backtrack → [5,4]             (11 removed)
  backtrack → [5]                 (4 removed)
  dfs(8, 14) ... finds [5,8,4,5] similarly
backtrack → []                    (5 removed)
```

* * *

# 7. Key Tricks & Insights

## Trick 1 — Leaf check is always required for root-to-leaf problems

```java
if (root.left == null && root.right == null)  // leaf: both children null
```

An internal node with `val == remaining` is NOT a valid answer. The path must terminate at a leaf.
This is the #1 source of wrong answers on path sum problems.

## Trick 2 — Backtrack only when state is mutable and shared

| State type | Backtrack? | Why |
|---|---|---|
| `int remaining` | ❌ No | Primitive — passed by value; each frame has own copy |
| `List<Integer> path` | ✅ Yes | Object reference — shared across all recursive calls |
| `StringBuilder` | ✅ Yes | Mutable — must `sb.deleteCharAt(sb.length() - 1)` after recursion |
| `String` | ❌ No | Immutable — `+` creates a new String each time; nothing shared |

## Trick 3 — Snapshot before backtracking

```java
result.add(new ArrayList<>(path));   // copy BEFORE the path gets cleared
path.remove(path.size() - 1);       // then backtrack
```

The snapshot and the backtrack are always adjacent. If you see one without the other, something is wrong.

## Trick 4 — null check before leaf check

```java
if (node == null) return;                              // null check FIRST
if (node.left == null && node.right == null) {...}     // then leaf check is safe
```

Reversing the order causes a NullPointerException when accessing `node.left` on a null reference.

## Trick 5 — Encoding: shift × base + digit

```java
// Decimal: each new digit shifts existing number left one place
currentNum = currentNum * 10 + node.val;    // 1 → 12 → 129

// Binary: bit-shift
currentNum = (currentNum << 1) | node.val;  // 0 → 1 → 10 → 101

// XOR: useful for toggling parity or accumulated properties
currentNum = currentNum ^ node.val;
```

## Trick 6 — Top-down propagation: pass (min, max) together

```java
// For ancestor-descendant problems: carry both extremes
int dfs(TreeNode node, int minSoFar, int maxSoFar) {
    if (node == null) return maxSoFar - minSoFar;
    minSoFar = Math.min(minSoFar, node.val);
    maxSoFar = Math.max(maxSoFar, node.val);
    return Math.max(dfs(node.left, minSoFar, maxSoFar),
                    dfs(node.right, minSoFar, maxSoFar));
}
```

Pass both the min and max seen from root-to-current as state. When you hit null, the running difference is the max ancestor diff for this path.

## Trick 7 — String paths: use immutable String, avoid StringBuilder backtrack

```java
// StringBuilder approach — must backtrack
sb.append(node.val);
dfs(node.left, sb, result);
dfs(node.right, sb, result);
sb.deleteCharAt(sb.length() - 1);   // ← easy to forget

// String approach — no backtrack (new String created each call)
dfs(node.left,  path + "->" + node.val, result);   // immutable: no undo needed
```

String concatenation is O(n) per call but eliminates the backtrack bug. Use StringBuilder for performance in large inputs.

* * *

# 8. Common Pitfalls (🚨 HIGH ROI)

## Pitfall 1 — Missing leaf check (matches internal nodes)

```java
// ❌ matches any node where remaining == 0 (including internal nodes)
if (remaining == 0) return true;

// ✅ only matches leaves where remaining is exactly paid off
if (node.left == null && node.right == null) return node.val == remaining;
```

## Pitfall 2 — Adding path reference instead of snapshot

```java
// ❌ adds a reference — will be empty after backtracking
result.add(path);

// ✅ creates a copy of current path state
result.add(new ArrayList<>(path));
```

## Pitfall 3 — Missing backtrack

```java
path.add(node.val);
dfs(node.left,  ...);
dfs(node.right, ...);
// ❌ forgot: path.remove(path.size() - 1);
// path grows without bound — sibling subtrees accumulate stale nodes
```

```java
path.add(node.val);
dfs(node.left,  ...);
dfs(node.right, ...);
path.remove(path.size() - 1);  // ✅ always undo after both children
```

## Pitfall 4 — Null check order

```java
// ❌ crashes: node.left and node.right accessed on null
if (node.left == null && node.right == null) return node.val == remaining;
if (node == null) return false;

// ✅ null guard first
if (node == null) return false;
if (node.left == null && node.right == null) return node.val == remaining;
```

## Pitfall 5 — Off-by-one in encoding (missing the leaf's contribution)

```java
// ❌ checks currentNum before incorporating the leaf's digit
if (node.left == null && node.right == null) return currentNum;  // missing leaf digit

// ✅ incorporate the leaf's digit, then return
currentNum = currentNum * 10 + node.val;
if (node.left == null && node.right == null) return currentNum;
```

* * *

# 9. Edge Case Checklist

* `root == null` — empty tree: return false / 0 / empty list
* Single node — it IS a leaf; check `root.val == targetSum` directly
* Target sum = 0, root.val = 0 — valid match at a leaf with val 0
* Negative node values — subtraction still works; remaining can go negative (valid)
* Multiple valid paths — collection variant captures all; boolean short-circuits on first
* Path exists but doesn't reach leaf — leaf check prevents false positive
* Very large trees (skewed) — recursion depth equals n; stack overflow risk for n > 10^4

* * *

# 10. Decision Framework (🔥 INTERVIEW WEAPON)

## Step 1: Does the path start at the root?

→ YES → Path Tracking (this archetype)
→ NO → Postorder Global + Local (Archetype 7, any-node-to-any-node paths)

## Step 2: Does the path end at a leaf?

→ YES → root-to-leaf; use Pattern 1, 2, or 3
→ NO → any node; use Postorder

## Step 3: What is the output type?

```
Boolean (yes/no)?                 → Pattern 1 (no backtracking)
Collect all paths?                → Pattern 2 (backtracking required)
Aggregate path value (sum/min)?   → Pattern 3 (encoding, primitive state)
Pass ancestor state down?         → Pattern 4 (state propagation)
```

## Step 4: Is state mutable?

→ YES (List, StringBuilder) → backtracking required
→ NO (int, String) → no backtracking needed

### Tree DFS Preorder Decision Tree

```
Input is a binary tree?
└── YES
    ├── Path starts at root?                               → Path Tracking (Archetype 8)
    │   ├── Path ends at leaf?
    │   │   ├── YES — root-to-leaf path
    │   │   │   ├── Boolean output?                        → Pattern 1 (hasPathSum)
    │   │   │   ├── Collect all paths?                     → Pattern 2 (backtrack)
    │   │   │   └── Aggregate value (sum of numbers)?      → Pattern 3 (encoding)
    │   │   └── NO — path from any node                   → Postorder Global+Local
    │   └── State from ancestor needed?                    → Pattern 4 (state prop)
    │
    ├── Need info from both subtrees first?                → Postorder DFS (Archetype 7)
    └── Need level-by-level?                              → BFS / Level Order (Archetype 9)
```

* * *

# 11. Problem Map Quick Reference

```
PATTERN                    PROBLEMS
────────────────────────────────────────────────────────────────
Boolean (Has Path)         #112 Path Sum
Collection (All Paths)     #113 Path Sum II
Enumerate All Paths        #257 Binary Tree Paths
Decimal Encoding           #129 Sum Root to Leaf Numbers
Binary Encoding            #1022 Sum of Root To Leaf Binary Numbers
State Propagation          #1026 Max Difference Between Node and Ancestor
                           #1315 Sum of Nodes with Even-Valued Grandparent
Bit Masking + Path         #1457 Pseudo-Palindromic Paths (XOR bitmask)
Count Good Nodes           #1448 Count Good Nodes (pass max seen from root)
Ancestor Constraint        #863 All Nodes Distance K (preorder labeling)
                           #988 Smallest String Starting From Leaf
Advanced / Hybrid          #437 Path Sum III (prefix sum — NOT pure root-to-leaf)
                           #666 Path Sum IV (decode flat array into tree)
```

> **Note on #437 Path Sum III:** Paths can start from any node, not just root.
> Requires prefix sum + hashmap. NOT the pure preorder template — listed for contrast.

* * *

# 12. Drill Section (Mastery Check)

## Verbal Drill

Explain this archetype in 2 minutes.

Target structure:

1. Preorder DFS solves tree problems where state flows **down** from parent to child
2. The two core variants are: boolean (no backtrack, primitive state) and collection (backtrack, mutable state)
3. Backtracking requires: add before recursing → snapshot if match → recurse left → recurse right → remove after
4. The leaf check is mandatory for root-to-leaf problems — matching at internal nodes is a false positive
5. Encoding (Pattern 3) uses multiplication/bit-shift to build a number on the way down, no backtrack needed

## Recognition Drill

Given a problem, identify the pattern before coding:

* Does the path start at root? → preorder (path tracking)
* Does it end at a leaf? → leaf check required
* Boolean answer? → Pattern 1, no backtracking
* Collect paths? → Pattern 2, backtracking with snapshot
* Build a number/string from root to leaf? → Pattern 3, encoding
* Need info from an ancestor at each node? → Pattern 4, state propagation
* Path can start anywhere? → NOT this archetype → Postorder Global+Local (Archetype 7)

## Transformation Drill

Convert brute → optimal without coding:

* **Path Sum**
    * Brute: enumerate all root-to-leaf paths, sum each, check if any == target — O(n·h)
    * Optimal: single DFS, subtract at each node, check at leaf — O(n)

* **Path Sum II**
    * Brute: enumerate all paths into a list, filter by sum — O(n·h) time + space
    * Optimal: single DFS with backtracking, snapshot on match — O(n) time, O(h·n) space (unavoidable)

* **Sum Root to Leaf Numbers**
    * Brute: collect all paths as strings, parse, sum — O(n·h) + string parsing overhead
    * Optimal: single DFS, carry `currentNum = currentNum * 10 + node.val` downward — O(n)

* **Max Ancestor Diff**
    * Brute: for each node, walk back to root and find max/min ancestor — O(n²)
    * Optimal: single pass, carry `(minSoFar, maxSoFar)` down — O(n)

## Primitive Speed Test

Write each from scratch with no reference. Target: < 3 min each, zero bugs.

```
□ hasPathSum(TreeNode root, int targetSum)              ← boolean, leaf check, no backtrack
□ pathSum(TreeNode root, int targetSum)                 ← collection, snapshot, backtrack
□ binaryTreePaths(TreeNode root)                        ← all paths as strings
□ sumNumbers(TreeNode root)                             ← decimal encoding, no backtrack
□ maxAncestorDiff(TreeNode root)                        ← state propagation, min/max
```

* * *

# 13. Problem Approach Template

```
# Tree DFS Path Tracking — Problem Quick Drill

Signal check:
- Root-to-leaf path? (state passed DOWN, decision at LEAF?)
- OR: ancestor state needed at each node?

Variant:
- □ Boolean (no backtracking)
- □ Collection (backtracking required)
- □ Encoding (multiply/shift, no backtrack)
- □ State Propagation (pass min/max/ancestor down)

Null return?
- null → false / 0 / return early

Leaf check?
- node.left == null && node.right == null

State type?
- primitive (int remaining)    → no backtrack
- mutable object (List, SB)   → backtrack required
- immutable (String, int)     → no backtrack

Backtrack step (if needed):
- path.remove(path.size() - 1)

Snapshot step (if collection):
- result.add(new ArrayList<>(path))  ← BEFORE backtrack
```

## Problem Drills

```
1. #112 Path Sum
Pattern:
- Boolean (Pattern 1)

Null returns: false
Leaf check: root.left == null && root.right == null

Approach:
1. if null → return false
2. if leaf → return root.val == targetSum
3. return dfs(left, targetSum - root.val) || dfs(right, targetSum - root.val)

Invariant:
- remaining converges toward 0; at leaf, val must exactly equal remaining

Edge cases:
- null root → false ✓
- single node (leaf) → root.val == targetSum ✓
- negative values → remaining can go negative; still works ✓


2. #113 Path Sum II
Pattern:
- Collection (Pattern 2) + backtracking

State: List<Integer> path (mutable → must backtrack)
Null returns: return (void)
Leaf check: node.left == null && node.right == null && node.val == remaining

Approach:
1. if null → return
2. path.add(node.val)
3. if leaf && match → result.add(new ArrayList<>(path))  ← SNAPSHOT
4. dfs(left, remaining - node.val, path, result)
5. dfs(right, remaining - node.val, path, result)
6. path.remove(path.size() - 1)  ← BACKTRACK

Invariant:
- path contains exactly the nodes from root to current node at all times
- backtrack restores path to pre-call state before returning to parent

Edge cases:
- null root → return immediately, result empty ✓
- single node matching → snapshot [root.val] ✓
- multiple valid paths → both collected ✓


3. #257 Binary Tree Paths
Pattern:
- Encoding (Pattern 3) — string variant

State: String path (immutable → no backtrack)
Null returns: return (void)
Leaf check: node.left == null && node.right == null

Approach:
1. if null → return
2. if leaf → result.add(path + node.val)
3. dfs(left,  path + node.val + "->", result)
4. dfs(right, path + node.val + "->", result)

Invariant:
- String concatenation creates a new String each call; no shared state to undo
- path is built left-to-right as we descend

Edge cases:
- single node → path = "" + node.val → output "[root.val]" ✓
- null root → result stays empty ✓


4. #129 Sum Root to Leaf Numbers
Pattern:
- Encoding (Pattern 3) — decimal

State: int currentNum (primitive → no backtrack)
Null returns: 0

Approach:
1. if null → return 0
2. currentNum = currentNum * 10 + node.val
3. if leaf → return currentNum
4. return dfs(left, currentNum) + dfs(right, currentNum)

Invariant:
- currentNum at each node = the number formed by digits from root to this node
- sum all leaf currentNums

Edge cases:
- single node → return root.val ✓
- path 1→2→3 → currentNum = 1 → 12 → 123 ✓


5. #1026 Maximum Difference Between Node and Ancestor
Pattern:
- State Propagation (Pattern 4)

State: (int minSoFar, int maxSoFar) — primitives → no backtrack
Null returns: maxSoFar - minSoFar (max diff seen on this root-to-null path)

Approach:
1. if null → return maxSoFar - minSoFar
2. minSoFar = min(minSoFar, node.val)
3. maxSoFar = max(maxSoFar, node.val)
4. return max(dfs(left, min, max), dfs(right, min, max))

Invariant:
- minSoFar and maxSoFar = min/max values from root to current node
- at null, difference = the extreme diff on this root-to-leaf path

Edge cases:
- single node → dfs(null, root.val, root.val) → 0 ✓
- all same value → diff = 0 ✓
- decreasing tree → maxSoFar set at root; minSoFar updates at each level ✓
```
