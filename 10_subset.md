# 🧩 Archetype 10 — Subset / Power Set

* * *

# 0. Goal

## What problem class does this solve?

* Problems asking you to **enumerate all subsets, combinations, or partitions** of a collection
* Problems where you must **decide for each element: include it or exclude it**
* Problems requiring you to **find one valid configuration** that satisfies a constraint (boolean backtracking)
* Problems that decompose a sequence into **valid segments or partitions**
* Any problem where the solution space is a tree of choices with 2–k branches at each node

## What mastery looks like

* Recognizes the "include/exclude binary tree" vs. "for-loop + start index" split in < 15 seconds
* Can write both templates from scratch without bugs
* Knows exactly when to sort + prune for duplicates
* Can convert boolean reachability problems (Partition Equal Subset Sum) into backtracking
* Can explain WHY `start` prevents duplicate orderings and WHY sorting enables duplicate pruning
* Identifies when pruning is possible and writes the prune condition correctly

* * *

# 1. Recognition Signals

## Strong signals

* "Return **all subsets** / all combinations / all partitions"
* "Find **all ways** to decompose / split / partition"
* "Choose k elements from n" (combinations)
* "Can you reach target sum using subset of elements?"
* "Split array into two equal-sum halves"
* "List all valid expressions / IP addresses / palindrome cuts"
* Input size n ≤ 20–25 (exponential solution space is acceptable)

## Weak / disguised signals

* "Count the number of ways" → might be DP, but backtracking + memoization works too
* "Find if a partition exists" → Boolean backtracking (Partition Equal Subset Sum)
* "Word break" → String segmentation = for-loop subset on suffix
* "Restore IP addresses" → partitioning with fixed-segment constraint
* "Letter combinations of phone number" → multi-set Cartesian product via for-loop pattern

## Anti-signals (when NOT to use Subset backtracking)

* n > 25 with no memoization → exponential blowup, need DP
* Problem asks for **ordering** → use Permutation archetype (Archetype 11)
* Problem asks for **shortest / fewest** → likely BFS or DP
* Grid traversal with exploration → DFS Flood Fill (Archetype 12)

* * *

# 2. Cheat Sheet

```
WHEN TO USE:
- "All subsets", "all combinations", "all partitions", "all ways to split"
- Boolean: "can you partition / reach target sum?"
- Input size ≤ 20–25 (exponential space is OK)
- "Choose k from n" (combinations)
- String decomposition into valid segments

CORE IDEA:
- Every subset problem is a binary decision tree: include or exclude each element
- Every combination problem is a for-loop tree: try each candidate from start onward
- Both produce an O(2^n) or O(n! / k!) solution space
- Backtracking = DFS on this tree + undo choices on the way back

THE 4 PATTERNS:
1. Include/Exclude      → binary choice at each index, O(2^n) leaves
2. For-loop + Start     → try each candidate from start, O(n choose k) leaves
3. Duplicate Pruning    → sort + skip nums[i]==nums[i-1] at same depth
4. Boolean Backtracking → backtrack with a target; prune when target < 0 or visited

TEMPLATE SELECTOR:
- All subsets of set?               → Include/Exclude (Pattern 1): #78
- All combinations summing to X?    → For-loop + Start (Pattern 2): #39
- Duplicates in input?              → Sort + Prune (Pattern 3): #90, #40
- Fixed size k selections?          → For-loop + Start with size check: #77
- Partition array into equal halves?→ Boolean backtracking (Pattern 4): #416
- All palindrome cuts?              → For-loop + Start with inline check: #131

TIME / SPACE:
- All subsets:            O(n × 2^n) time, O(n) stack space
- All combinations:       O(n × 2^n) time in worst case, O(k) stack space
- Boolean backtracking:   O(2^n) worst case; O(n × target) with memoization
- Palindrome partitioning:O(n × 2^n) time, O(n) stack space

TOP 3 TRICKS:
1. The start index is what prevents [1,2] and [2,1] from both appearing
2. Sort before duplicate pruning — skip nums[i]==nums[i-1] only when i > start
3. For boolean backtracking, prune immediately when remaining < 0 (saves massive branches)

TOP 3 PITFALLS:
1. Forgetting to pass i vs i+1 to the next recursion (reuse vs no-reuse)
2. Pruning nums[i]==nums[i-1] when i==start — this removes valid first-choices
3. In Combination Sum II: forgetting to sort before pruning duplicates
```

* * *

# 3. Core Mental Model

## Core Ideas

* The solution space of any subset/combination problem is a **decision tree**
* At each node in the tree, you make a choice; you undo it (backtrack) before trying the next choice
* The recursion frame is: **make choice → recurse → undo choice**
* The two templates differ in how they **avoid duplicates**:
    * Include/Exclude: each *index* is visited once — position determines uniqueness
    * For-loop + Start: `start` advances forward — earlier elements can't be re-chosen

## The Two Core Templates

```java
// ─────────────────────────────────────────────
// TEMPLATE 1 — Include / Exclude
// When: collect all subsets (each element is in or out)
// O(2^n) subsets, each element = one binary decision
// ─────────────────────────────────────────────
void backtrack(int[] nums, int index, List<Integer> current, List<List<Integer>> result) {
    if (index == nums.length) {
        result.add(new ArrayList<>(current)); // base case: snapshot current subset
        return;
    }
    // Branch A: EXCLUDE nums[index]
    backtrack(nums, index + 1, current, result);

    // Branch B: INCLUDE nums[index]
    current.add(nums[index]);
    backtrack(nums, index + 1, current, result);
    current.remove(current.size() - 1); // undo
}
```

```java
// ─────────────────────────────────────────────
// TEMPLATE 2 — For-loop + Start Index
// When: collect combinations (choose any remaining candidates from start onward)
// O(2^n) worst case; prevents ordering duplicates via start
// ─────────────────────────────────────────────
void backtrack(int[] candidates, int start, int remaining, List<Integer> current, List<List<Integer>> result) {
    if (remaining == 0) {
        result.add(new ArrayList<>(current)); // found valid combination
        return;
    }
    for (int i = start; i < candidates.length; i++) {
        if (candidates[i] > remaining) break; // pruning (requires sorted input)
        current.add(candidates[i]);
        backtrack(candidates, i + 1, remaining - candidates[i], current, result); // i+1: no reuse
        current.remove(current.size() - 1); // backtrack
    }
}
// For reuse: pass i instead of i+1 (Combination Sum #39)
```

## The Decision Tree Visualization

```
nums = [1, 2, 3]  — Include/Exclude pattern

                     []
                /         \
          [1]              []
         /   \           /   \
      [1,2]  [1]      [2]    []
      /  \   / \      / \    / \
  [1,2,3][1,2][1,3][1] [2,3][2] [3] []
   ← all 8 subsets at the leaves (depth = n)
```

```
candidates=[1,2,3], target=3  — For-loop + Start pattern

                []  (remaining=3)
           /    |    \
          1     2     3
       /  |  \   |     \
      1   2   3  2      ← [1,2] sums to 3
     /|   |
    1  2   ← [1,1,1] and [1,2] found
```

## Key Invariants

```
Include/Exclude:
  - index advances by 1 regardless of choice
  - every leaf has index == n → exactly 2^n leaves
  - no duplicates possible (position determines which subset)

For-loop + Start:
  - i starts at `start`, not 0 → earlier elements never re-chosen
  - prevents [2,1] after [1,2] has been explored
  - combine with sort + prune to handle duplicate elements in input
```

* * *

# 4. Optimization Ladder (Interview Narrative)

## Step 1 — Brute Force

* Generate all 2^n binary strings; for each, collect elements where bit = 1
* Or: use `(1 << n)` integer loop — iterate all bitmask states
* Time: O(n × 2^n), Space: O(n × 2^n) for storing all results

## Step 2 — Optimization Insight

What are we eliminating?

* Bitmask brute force is implicit; recursive backtracking makes the decision tree **explicit**
* Key advantage of backtracking: **pruning** — when a constraint is violated (e.g. remaining < 0), entire subtree is cut
* We avoid generating and then discarding invalid combinations — we never enter them

Core shift:

```
from: generate all 2^n subsets, then filter by constraint (bitmask approach)
to:   build subsets incrementally; prune branches the moment constraint is violated
```

## Step 3 — Optimal Approach

* One DFS pass; each node does O(n) work at worst
* Pruning reduces practical runtime dramatically for constraint problems
* For boolean problems (exists?), early termination as soon as answer found
* Time: O(n × 2^n) theoretical; often much faster in practice due to pruning

* * *

# 5. Core Patterns

```
Pattern 1 Include/Exclude    → binary choice at each index; base = index==n
Pattern 2 For-loop + Start   → for i in [start, n); recurse with i or i+1
Pattern 3 Duplicate Pruning  → sort + skip nums[i]==nums[i-1] when i > start
Pattern 4 Boolean Backtrack  → return boolean; prune on remaining < 0 or memoized
```

## Pattern 1 — Include / Exclude

**When to use:**

* "Return all subsets" — each element is independently in or out
* The number of valid leaves is exactly 2^n (every path to depth n is valid)
* No target constraint, no size constraint

**Mental model:**

```
At each index: make TWO recursive calls.
  Call A: skip this element (do NOT add to current)
  Call B: include this element (add, recurse, remove)
Collect a snapshot of `current` at every leaf (index == n).
```

```java
// ─────────────────────────────────────────────
// PATTERN 1 — Include / Exclude (Subsets #78)
// ─────────────────────────────────────────────
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

void backtrack(int[] nums, int index, List<Integer> current, List<List<Integer>> result) {
    if (index == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    // Exclude
    backtrack(nums, index + 1, current, result);
    // Include
    current.add(nums[index]);
    backtrack(nums, index + 1, current, result);
    current.remove(current.size() - 1);
}
```

**Why the exclude branch is called first:**
Either order works. Exclude-first generates the empty set first, which matches the natural
"start empty, add elements" mental model.

* * *

## Pattern 2 — For-loop + Start Index

**When to use:**

* "Find all combinations summing to target" — choices are candidates, reuse allowed or not
* "Choose k from n elements" — fixed selection size
* Any problem where you select from remaining candidates (not a fixed position)

**Mental model:**

```
At each level, try every candidate from `start` to end.
After choosing candidate i, recurse with start = i (reuse) or i+1 (no reuse).
The `start` pointer prevents choosing earlier candidates again → no ordering duplicates.
```

```java
// ─────────────────────────────────────────────
// PATTERN 2 — For-loop + Start Index (Combination Sum #39)
// ─────────────────────────────────────────────
public List<List<Integer>> combinationSum(int[] candidates, int target) {
    Arrays.sort(candidates); // enables early break pruning
    List<List<Integer>> result = new ArrayList<>();
    backtrack(candidates, 0, target, new ArrayList<>(), result);
    return result;
}

void backtrack(int[] candidates, int start, int remaining,
               List<Integer> current, List<List<Integer>> result) {
    if (remaining == 0) {
        result.add(new ArrayList<>(current));
        return;
    }
    for (int i = start; i < candidates.length; i++) {
        if (candidates[i] > remaining) break; // prune: sorted, so all further are too large
        current.add(candidates[i]);
        backtrack(candidates, i, remaining - candidates[i], current, result); // i: reuse allowed
        current.remove(current.size() - 1);
    }
}
```

**Why `start = i` for reuse vs `i+1` for no reuse:**
`i`: current candidate can be chosen again at next level (Combination Sum)
`i+1`: each candidate used at most once (Combinations #77, Combination Sum II #40)

* * *

## Pattern 3 — Duplicate Pruning

**When to use:**

* Input array has **duplicate elements** and you want **distinct** subsets/combinations
* Without pruning, you'd generate the same subset through different index orderings of equal values

**Mental model:**

```
Sort the input first (groups equal elements together).
In the for-loop, if nums[i] == nums[i-1] AND i > start: skip.
  → "i > start" means: we are NOT choosing this element as the FIRST at this level.
  → If i == start, we MUST allow it — this is the first pick at this level.
  → If i > start, we already explored a branch with the same value → skip duplicate.
```

```java
// ─────────────────────────────────────────────
// PATTERN 3 — Duplicate Pruning (Subsets II #90)
// ─────────────────────────────────────────────
public List<List<Integer>> subsetsWithDup(int[] nums) {
    Arrays.sort(nums); // REQUIRED: group duplicates together
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

void backtrack(int[] nums, int start, List<Integer> current, List<List<Integer>> result) {
    result.add(new ArrayList<>(current)); // add EVERY prefix, not just leaves
    for (int i = start; i < nums.length; i++) {
        if (i > start && nums[i] == nums[i - 1]) continue; // skip duplicate at same depth
        current.add(nums[i]);
        backtrack(nums, i + 1, current, result);
        current.remove(current.size() - 1);
    }
}
```

**Why `i > start` and not `i > 0`:**
If we wrote `i > 0`, we'd skip the duplicate even when it's the very first choice at this
recursion level — incorrectly eliminating valid subsets. The condition `i > start` means:
"we've already tried this same value as the first element at this level; skip."

* * *

## Pattern 4 — Boolean Backtracking

**When to use:**

* "Does a valid partition / assignment exist?" — return true/false, not the collection
* "Can you partition array into k equal subsets?" — assign each element to a bucket
* Boolean target-sum problems where you just need existence, not enumeration

**Mental model:**

```
Return true as soon as one valid configuration is found.
Return false only after exhausting all branches.
Prune immediately when constraint can no longer be satisfied (remaining < 0).
Use a visited/boolean array to avoid re-processing same element.
```

```java
// ─────────────────────────────────────────────
// PATTERN 4 — Boolean Backtracking (Partition Equal Subset Sum #416)
// ─────────────────────────────────────────────
public boolean canPartition(int[] nums) {
    int sum = Arrays.stream(nums).sum();
    if (sum % 2 != 0) return false; // odd total can't split evenly
    int target = sum / 2;
    Arrays.sort(nums); // optional: sort descending for earlier pruning
    return backtrack(nums, 0, target);
}

boolean backtrack(int[] nums, int start, int remaining) {
    if (remaining == 0) return true;     // found valid subset
    if (remaining < 0) return false;     // overshot
    for (int i = start; i < nums.length; i++) {
        if (i > start && nums[i] == nums[i - 1]) continue; // skip duplicates
        if (backtrack(nums, i + 1, remaining - nums[i])) return true; // early exit
    }
    return false;
}
```

**Practical note on memoization:**
For large inputs, pure backtracking TLEs on #416. The DP solution is preferred.
However, at interview time, backtracking demonstrates the pattern clearly and is the
logical stepping stone before the DP optimization.

* * *

# 6. Flagship Problem — Palindrome Partitioning (#131)

This is the canonical compositional problem for the Subset archetype. It requires:
- **For-loop + Start Index** template
- An **inline validity check** (is this segment a palindrome?)
- **Collecting all valid paths** (not just leaves that hit a target)

**Problem:** Given a string `s`, partition it so that every substring is a palindrome. Return all possible palindrome partitioning.

```
Step 1: Use for-loop + start index — try every possible first segment [start..i]
Step 2: Check if substring [start..i] is a palindrome before recursing
Step 3: If palindrome, add to current path and recurse from i+1
Step 4: When start == s.length(), we've consumed the whole string → add snapshot
Step 5: Backtrack by removing the last added segment
```

```java
public List<List<String>> partition(String s) {
    List<List<String>> result = new ArrayList<>();
    backtrack(s, 0, new ArrayList<>(), result);
    return result;
}

void backtrack(String s, int start, List<String> current, List<List<String>> result) {
    if (start == s.length()) {
        result.add(new ArrayList<>(current)); // consumed entire string → valid partition
        return;
    }
    for (int end = start; end < s.length(); end++) {
        if (isPalindrome(s, start, end)) {         // only recurse on valid segments
            current.add(s.substring(start, end + 1));
            backtrack(s, end + 1, current, result); // next segment starts after end
            current.remove(current.size() - 1);     // backtrack
        }
    }
}

boolean isPalindrome(String s, int left, int right) {
    while (left < right) {
        if (s.charAt(left++) != s.charAt(right--)) return false;
    }
    return true;
}
```

**Why `end + 1` not `end`:**
We've consumed `s[start..end]` as one segment. The next segment starts at `end + 1`.
Passing `end` would re-examine the current character → infinite recursion.

**Why collect at `start == s.length()` not at every level:**
We only have a valid partition when we've covered the ENTIRE string. Collecting at
intermediate states would give partial partitions (e.g. just the first cut).

**Optimization with precomputed palindrome table:**

```java
// Precompute dp[i][j] = true if s[i..j] is palindrome: O(n^2) preprocessing
boolean[][] dp = new boolean[n][n];
for (int i = n - 1; i >= 0; i--) {
    for (int j = i; j < n; j++) {
        dp[i][j] = s.charAt(i) == s.charAt(j) && (j - i <= 2 || dp[i + 1][j - 1]);
    }
}
// Then: if (dp[start][end]) instead of isPalindrome(...)
// Reduces isPalindrome from O(n) to O(1) per call
```

* * *

# 7. Key Tricks & Insights

## Trick 1 — The start index is the uniqueness guarantee

```
Without start:  try 1,2,3 at every level → [1,2] AND [2,1] both generated
With start:     at level 1, pick from [0..n); at level 2, pick from [i+1..n)
                → orderings are fixed; [2,1] is impossible if 1 comes before 2 in array
```

The `start` index is NOT about avoiding revisiting — it's about **fixing an order** on the choice sequence so that every distinct subset is generated exactly once.

## Trick 2 — Sort before duplicate pruning; the guard is i > start, not i > 0

```java
// ❌ wrong — skips first valid occurrence at this level
if (i > 0 && nums[i] == nums[i - 1]) continue;

// ✅ correct — only skip if not the first pick at this recursion level
if (i > start && nums[i] == nums[i - 1]) continue;
```

`i > start` means: "I've already explored a branch where this value was the FIRST pick here."
`i == start` means: "This is my first pick at this level — I must try it."

## Trick 3 — i vs i+1 controls reuse

```java
backtrack(candidates, i,     ...); // reuse: same element can be picked again (#39)
backtrack(candidates, i + 1, ...); // no reuse: each element used at most once (#40, #77)
```

This single character (`i` vs `i+1`) is the entire difference between Combination Sum I and II.

## Trick 4 — Collect at every level (subsets) vs. at leaves (combinations)

```java
// Subsets II — add current at every recursion entry (all prefixes are valid subsets)
result.add(new ArrayList<>(current)); // at the TOP of backtrack(), before the loop

// Combination Sum — add only at the base case (only leaves hitting target are valid)
if (remaining == 0) result.add(new ArrayList<>(current));
```

The question to ask: "Is a partial state already a valid answer?" For subsets: yes (empty set, single elements, etc. are all valid). For combinations: no (only complete sums count).

## Trick 5 — Sort + early break for combination sum pruning

```java
Arrays.sort(candidates);
for (int i = start; i < candidates.length; i++) {
    if (candidates[i] > remaining) break; // ALL subsequent candidates are also > remaining
    ...
}
```

Without sort, you can't break early — you'd have to continue checking all candidates.

## Trick 6 — Snapshot with `new ArrayList<>(current)`, never add `current` directly

```java
// ❌ wrong — all results point to the same mutable list
result.add(current);

// ✅ correct — snapshot the state at this point in time
result.add(new ArrayList<>(current));
```

`current` is mutated as backtracking continues. Adding a reference to it gives you the final (empty) state in all result entries.

* * *

# 8. Common Pitfalls (🚨 HIGH ROI)

## Pitfall 1 — Adding `current` directly instead of a snapshot

```java
// ❌ all entries in result are the same mutable list (will be empty at the end)
result.add(current);
```

```java
// ✅ snapshot: freeze the state at this moment
result.add(new ArrayList<>(current));
```

## Pitfall 2 — Pruning duplicates with `i > 0` instead of `i > start`

```java
// ❌ nums = [1, 2, 2]; subsets starting with 2 at first position are wrongly skipped
if (i > 0 && nums[i] == nums[i - 1]) continue;
```

```java
// ✅ only skip the duplicate if it's NOT the first choice at this level
if (i > start && nums[i] == nums[i - 1]) continue;
```

## Pitfall 3 — Forgetting to sort before duplicate pruning

```java
// ❌ nums = [2, 1, 2] → duplicates are not adjacent → pruning misses some, removes others
// without sort, nums[i] == nums[i-1] doesn't reliably identify duplicates
```

```java
// ✅ sort first: guarantees all equal values are adjacent → pruning is correct
Arrays.sort(nums);
```

## Pitfall 4 — Passing `i` instead of `i+1` when reuse is NOT intended

```java
// ❌ Combination Sum II (#40): elements must be used once, but i allows reuse
backtrack(candidates, i, remaining - candidates[i], current, result);
```

```java
// ✅ no-reuse: advance to i+1
backtrack(candidates, i + 1, remaining - candidates[i], current, result);
```

## Pitfall 5 — Collecting at the wrong point

```java
// ❌ for Subsets — collecting only at leaves (index==n) misses intermediate subsets
// [1], [2], [1,2] etc. would all be missed

// ❌ for Combinations — collecting at every level adds partial sums as results
if (remaining >= 0) result.add(new ArrayList<>(current)); // wrong
```

```java
// ✅ Subsets: collect at entry of backtrack (every state is a valid subset)
result.add(new ArrayList<>(current));

// ✅ Combinations: collect only at base case (target reached)
if (remaining == 0) { result.add(new ArrayList<>(current)); return; }
```

* * *

# 9. Edge Case Checklist

General edge cases for Subset/Combination problems:

* `nums = []` — empty input → result should be `[[]]` (one empty subset) for subset problems
* `nums = [0]` — zero in input → careful with target subtraction (0 can be infinitely included)
* All elements equal (e.g. `[2, 2, 2]`) — full duplicate pruning required
* Target = 0 — empty subset already satisfies it; return `[[]]` immediately
* Single element = target — valid combination of size 1
* No valid combination exists — return empty list `[]`
* Large n with tight constraint — consider whether memoization is needed
* Negative values in input — pruning `candidates[i] > remaining` no longer valid without sort; may need different approach

* * *

# 10. Decision Framework (🔥 INTERVIEW WEAPON)

## Step 1: Are we enumerating (collect all) or deciding (exists one)?

→ ENUMERATE → choose Include/Exclude or For-loop + Start
→ DECIDE    → use Boolean Backtracking (Pattern 4); return true on first valid path

## Step 2: Does order of elements in the result matter?

→ NO (subset {1,2} == {2,1}) → use For-loop + Start or Include/Exclude
→ YES (permutation (1,2) ≠ (2,1)) → ⛔ NOT this archetype → see Permutation (Archetype 11)

## Step 3: Can elements be reused?

→ YES → `backtrack(candidates, i, ...)` (Combination Sum #39)
→ NO  → `backtrack(candidates, i+1, ...)` (Subsets #78, Combination Sum II #40)

## Step 4: Are there duplicates in the input?

→ YES → sort + `if (i > start && nums[i] == nums[i-1]) continue`
→ NO  → no pruning needed

## Step 5: When do I collect a result?

→ Every state is valid (subsets) → collect at entry of backtrack
→ Only when target is met (combinations) → collect at base case when `remaining == 0`
→ Only when full string consumed (partitions) → collect when `start == s.length()`

### Subset / Combination Decision Tree

```
Problem asks for all subsets / combinations / partitions?
└── YES
    ├── Fixed binary in/out for each element?          → Include/Exclude (#78)
    ├── Choose from remaining candidates?              → For-loop + Start
    │   ├── Reuse allowed?                             → recurse with i (#39)
    │   ├── No reuse?                                  → recurse with i+1 (#40, #77)
    │   └── Duplicates in input?                       → sort + skip i > start (#90, #40)
    ├── Boolean: does valid partition exist?           → Boolean Backtracking (#416)
    └── String partitioning into valid segments?       → For-loop + inline check (#131)

Collect result at:
    ├── All subsets (every prefix valid)?              → top of backtrack function
    ├── Combination (only full sum valid)?             → base case remaining == 0
    └── Partition (only full string consumed valid)?   → base case start == s.length()
```

* * *

# 11. Problem Map Quick Reference

```
PATTERN                      PROBLEMS
─────────────────────────────────────────────────────────────────────
Include / Exclude             #78 Subsets
For-loop + Start (reuse)      #39 Combination Sum
For-loop + Start (no reuse)   #77 Combinations, #216 Combination Sum III
Duplicate Pruning             #90 Subsets II, #40 Combination Sum II
Boolean Backtracking          #416 Partition Equal Subset Sum, #698, #473
String Partition              #131 Palindrome Partitioning, #93 Restore IP
Segmentation (for-loop)       #139 Word Break, #140 Word Break II
Combo + expression            #22 Generate Parentheses, #241 Different Ways, #282 Expression Add Operators
Combo + constraint            #526 Beautiful Arrangement, #1239 Max Concat Length
Include/Exclude variant       #784 Letter Case Permutation, #320 Generalized Abbreviation
Target Sum (bool)             #494 Target Sum, #1049 Last Stone Weight II
Multi-set Cartesian           #17 Letter Combinations of a Phone Number
```

* * *

# 12. Drill Section (Mastery Check)

## Verbal Drill

Explain this archetype in 2 minutes.

Target structure:

1. Subset problems are decision trees: at each element, include or exclude (or: try each candidate from start)
2. Backtracking = DFS on this tree + undo choice before trying the next branch
3. Two templates: Include/Exclude (binary at each index) vs For-loop + Start (try remaining candidates)
4. Duplicate handling: sort + skip `nums[i] == nums[i-1]` when `i > start`
5. Reuse vs no-reuse: pass `i` vs `i+1` to next recursion level

## Recognition Drill

Given a problem, identify the pattern before coding:

* "All subsets" with no target? → Include/Exclude (#78)
* "All combinations summing to target" with reuse? → For-loop + Start, pass `i` (#39)
* "All combinations summing to target" no reuse, duplicates in input? → Sort + Prune, pass `i+1` (#40)
* "Choose exactly k from n"? → For-loop + Start with size limit (#77)
* "Can partition into equal halves?" → Boolean Backtracking (#416)
* "All palindrome decompositions"? → For-loop + Start with inline palindrome check (#131)

## Transformation Drill

Convert brute → optimal without coding:

* **Subsets #78**
    * Brute: iterate all 2^n bitmasks, collect elements where bit=1 — O(n × 2^n)
    * Optimal: Include/Exclude backtracking — same O(n × 2^n) but explicit, clean, extensible

* **Combination Sum #39**
    * Brute: generate all subsets, filter by sum — O(n × 2^n), generates many invalid
    * Optimal: For-loop + start + early break on sorted input — prunes invalid branches before entering

* **Subsets II #90 (with duplicates)**
    * Brute: generate all subsets, deduplicate with a HashSet — O(n × 2^n) + O(2^n) hash ops
    * Optimal: Sort + skip `nums[i]==nums[i-1] when i>start` — O(n × 2^n) no extra space

* **Partition Equal Subset Sum #416**
    * Brute: try all 2^n subsets, check sum — O(n × 2^n)
    * Backtracking with pruning: prune when remaining < 0 or no elements left — much faster in practice
    * DP optimization: O(n × target) — convert subset existence to knapsack boolean DP

## Primitive Speed Test

Write each from scratch with no reference. Target: < 3 min each, zero bugs.

```
□ subsets(int[] nums)                        ← Include/Exclude; collect at every entry
□ combinationSum(int[] candidates, int t)    ← For-loop + i (reuse); collect at remaining==0
□ subsetsWithDup(int[] nums)                 ← Sort + For-loop + i>start prune
□ combinationSum2(int[] candidates, int t)   ← Sort + For-loop + i+1 + i>start prune
□ combinations(int n, int k)                 ← For-loop + i+1 + size==k base case
□ partition(String s)                        ← For-loop + start + isPalindrome check
□ canPartition(int[] nums)                   ← Boolean backtracking + remaining==0 return true
```

* * *

# 13. Problem Approach Template

```
1. Is this enumerate (all results) or decide (exists one)?
   → ENUMERATE → backtracking with result collection
   → DECIDE    → boolean backtracking; return true on first valid

2. Template?
   → Binary in/out for each element?    → Include/Exclude
   → Choose from remaining candidates?  → For-loop + Start
   → Reuse elements?                    → pass i (not i+1)

3. Duplicates in input?
   → YES → sort first; add: if (i > start && nums[i] == nums[i-1]) continue;
   → NO  → no guard needed

4. When to collect?
   → All subsets?          → result.add() at TOP of backtrack (before loop)
   → Target-sum combo?     → result.add() when remaining == 0
   → String partition?     → result.add() when start == s.length()

5. Pruning?
   → Sorted + remaining < 0?    → break (not continue; all subsequent are larger)
   → Boolean found?             → return true immediately
```

```
# Subset / Combination — Problem Quick Drill

Signal check:
- Why backtracking? (enumerate all / find one valid configuration?)

Template:
- □ Include / Exclude
- □ For-loop + Start
- □ Duplicate Pruning (sort first?)
- □ Boolean Backtracking

Reuse elements?
- YES (pass i) / NO (pass i+1)

When to collect result?
- □ Every entry (subsets)
- □ Base case: remaining == 0 (combinations)
- □ Base case: start == s.length() (partitions)

Pruning?
- □ candidates[i] > remaining → break
- □ i > start && nums[i] == nums[i-1] → continue
- □ Early return true (boolean problems)

Edge cases:
- empty input?
- duplicates?
- target = 0?
- no valid answer?
```

## Problem Drills

```
1. #78 Subsets
Pattern: Include / Exclude

Null/empty returns: result gets [[]] at base

Approach:
1. if index == nums.length → snapshot current into result; return
2. Branch A: backtrack(index+1)           ← exclude nums[index]
3. Branch B: add nums[index]; backtrack(index+1); remove ← include

Invariant:
- at depth d: current holds d elements (all subsets of size d)
- every leaf generates exactly one unique subset

Edge cases:
- empty nums → backtrack(0) hits index==0==length immediately → result = [[]] ✓
- single element → result = [[], [nums[0]]] ✓


2. #39 Combination Sum
Pattern: For-loop + Start (reuse allowed)

Approach:
1. if remaining == 0 → snapshot; return
2. sort candidates (enables break pruning)
3. for i = start to end:
   a. if candidates[i] > remaining → break
   b. add candidates[i]
   c. backtrack(i, remaining - candidates[i])   ← i: reuse
   d. remove last

Invariant:
- start ensures no element before index start is re-chosen
- i (not i+1) allows same element multiple times

Edge cases:
- single candidate == target → [[candidate]] ✓
- all candidates > target → no results → return [] ✓


3. #90 Subsets II
Pattern: For-loop + Start + Duplicate Pruning

Approach:
1. sort nums FIRST
2. result.add(snapshot)    ← collect at every entry
3. for i = start to end:
   a. if i > start && nums[i] == nums[i-1] → continue
   b. add nums[i]; backtrack(i+1); remove

Invariant:
- sort groups equal values
- i > start guard prevents re-using same value as first pick at this level

Edge cases:
- nums = [1,1] → sorted = [1,1]; first pick i=0 (start=0, allowed); second pick i=1 (i>start, nums[1]==nums[0], skip) → result = [[], [1], [1,1]] ✓


4. #416 Partition Equal Subset Sum
Pattern: Boolean Backtracking

Approach:
1. sum = total; if sum % 2 != 0 → false
2. target = sum / 2; sort nums descending
3. backtrack(0, target):
   a. if remaining == 0 → return true
   b. if remaining < 0 → return false
   c. for i = start to end:
      - skip duplicates (i > start && nums[i]==nums[i-1])
      - if backtrack(i+1, remaining - nums[i]) → return true
   d. return false

Invariant:
- target == 0 means we've found a subset summing to half total
- pruning on remaining < 0 cuts branches early

Edge cases:
- total is odd → false immediately ✓
- single element == target → [element] is the partition ✓
- all elements > target → false ✓


5. #131 Palindrome Partitioning
Pattern: For-loop + Start + Inline Validity Check

Approach:
1. if start == s.length() → snapshot; return
2. for end = start to s.length()-1:
   a. if isPalindrome(s, start, end):
      - add s[start..end]
      - backtrack(end + 1)
      - remove last

Invariant:
- collect only when start reaches end of string (full partition consumed)
- inline palindrome check acts as prune: non-palindromes are never explored

Edge cases:
- single char always palindrome → every single-char partition is valid ✓
- entire string is palindrome → [[s]] is always in result ✓
```
