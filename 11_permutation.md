# 🔀 Archetype 11 — Permutation

* * *

# 0. Goal

## What problem class does this solve?

* Problems requiring all ordered arrangements of a collection (distinct or with duplicates)
* Problems where **order matters** — unlike Subset where order does not
* In-place permutation manipulation: next permutation, previous permutation, kth permutation
* Frequency-based string checks: does string A contain a permutation of string B?
* Constrained arrangement: place elements where adjacent or structural rules must hold

## What mastery looks like

* Distinguishes permutation from combination/subset in < 10 seconds
* Writes the `used[]` boolean backtracking template from scratch with no bugs
* Adds duplicate pruning (`sort + skip if prev used`) without hesitation
* Knows when to swap-in-place vs. used-array vs. sliding-window for permutation problems
* Can implement next permutation in-place (the 3-step algorithm) under pressure

* * *

# 1. Recognition Signals

## Strong signals

* "All permutations / arrangements" of an array or string
* "Does string A contain a permutation / anagram of string B?" → sliding window
* "Next permutation", "previous permutation", "kth permutation"
* "Arrange elements such that no two adjacent are equal" → constraint backtracking
* "Count arrangements where …" (with structural constraint)
* Output contains ordered sequences, not just sets

## Weak / disguised signals

* "Largest number" → custom sort (ordering is a permutation problem in disguise)
* "Find missing / duplicate" with index cycling → implicit permutation on [1..n]
* "Palindrome permutation" → frequency analysis, not full generation
* "Wiggle sort" → partition / rearrange (permutation thinking without backtracking)

## Anti-signals (when NOT to use permutation)

* Order does not matter → Subset / Power Set (Archetype 10)
* Need level-by-level structure → BFS (Archetype 9)
* Looking for a contiguous window → Sliding Window (Archetype 2)
* Need to count paths in a grid → combinatorics math, not backtracking

* * *

# 2. Cheat Sheet

```
WHEN TO USE:
- "All permutations / arrangements" of input
- "Next / prev / kth permutation" (in-place or by rank)
- "Permutation in string" / anagram check in window
- Constrained placement where ordering matters
- Index-based problems on [1..n] range arrays

CORE IDEA:
- Permutation = ordered selection of ALL elements
- Backtracking with a used[] boolean array is the universal template
- Duplicate pruning: sort first, skip if nums[i] == nums[i-1] && used[i-1] == false
- At each depth, choose one unused element, recurse, un-choose

THE 4 PATTERNS:
1. Distinct Backtracking   → used[] boolean array; pick any unused at each level
2. Duplicate Pruning       → sort + used[]; skip nums[i] == nums[i-1] && !used[i-1]
3. In-Place Swap           → swap(nums, start, i); recurse; swap back — no extra space
4. In-Place Manipulation   → next permutation 3-step: find break, find swap, reverse suffix

TEMPLATE SELECTOR:
- All permutations, distinct?          → Distinct Backtracking (used[])
- All permutations, duplicates?        → Duplicate Pruning (sort + used[])
- Memory-critical, distinct?           → In-Place Swap (no used array)
- Next/prev permutation?               → In-Place Manipulation (3-step)
- Kth permutation?                     → Math: factoriadic + greedy digit selection
- Permutation in string?               → Sliding Window + frequency array

TIME / SPACE:
- Generate all: O(n! × n) time, O(n) space for used[] + recursion stack
- Next permutation: O(n) time, O(1) space
- Kth permutation: O(n²) time, O(n) space
- Permutation in string: O(n + k) time, O(1) space (26-char freq)

TOP 3 TRICKS:
1. Duplicate pruning: sort + skip if nums[i] == nums[i-1] && !used[i-1]
   → "!used[i-1]" ensures we always pick the leftmost copy first
2. Next permutation: find right-most i where nums[i] < nums[i+1], then find
   the smallest element to its right that is larger, swap, reverse the suffix
3. Kth permutation: use factorial system — greedily pick each digit by (k-1) / (n-1)!

TOP 3 PITFALLS:
1. Forgetting to mark used[i] = false after recursion (un-choose step)
2. In duplicate pruning: using !used[i-1] vs used[i-1] — flip causes duplicates or misses
3. Next permutation: reversing the entire suffix vs. from i+1 onward — must start from i+1
```

* * *

# 3. Core Mental Model

## Core Ideas

* Permutation = arrange ALL elements in every possible order → n! results for n distinct elements
* Unlike Subset: we must use every element exactly once; the START index trick does NOT apply
* Backtracking is the universal engine: place → recurse → un-place
* The `used[]` array tracks which elements are currently on the "path"
* Duplicate pruning requires: **sort first** (so duplicates are adjacent), then skip if the previous identical element is not in use

## Permutation vs. Subset Mental Model

```
Subset (Archetype 10):              Permutation (Archetype 11):
- Order does NOT matter             - Order MATTERS
- [1,2] == [2,1]                    - [1,2] ≠ [2,1]
- Use start index to avoid re-pick  - Use used[] to avoid re-pick
- 2^n possible outputs              - n! possible outputs
- Loop from `start`                 - Loop from 0, skip if used
```

## The Permutation Frame

```java
void backtrack(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> result) {
    // 1. Base case — path is complete
    if (path.size() == nums.length) {
        result.add(new ArrayList<>(path));
        return;
    }
    // 2. Try every element at this position
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;              // already on path
        used[i] = true;                     // choose
        path.add(nums[i]);
        backtrack(nums, used, path, result);
        path.remove(path.size() - 1);       // un-choose
        used[i] = false;
    }
}
```

## The Recursion Tree

```
Input: [1, 2, 3]
Depth 0 (pick position 0):    1           2           3
                             / \         / \         / \
Depth 1 (pick position 1): 2   3       1   3       1   2
                           |   |       |   |       |   |
Depth 2 (pick position 2): 3   2       3   1       2   1

Results: [1,2,3] [1,3,2] [2,1,3] [2,3,1] [3,1,2] [3,2,1]
```

## Duplicate Pruning — The Critical Insight

```
Input: [1, 1, 2]   (sorted)

Without pruning, both 1s at position 0 produce identical subtrees.
Pruning rule: skip nums[i] if nums[i] == nums[i-1] && !used[i-1]

Why !used[i-1]?
- If used[i-1] == true: nums[i-1] is on the current path — fine, different positions
- If used[i-1] == false: nums[i-1] was already tried at this level and backtracked
  → nums[i] would produce an identical sequence → SKIP

This ensures we always pick the leftmost copy of a value first at each level.
```

```java
for (int i = 0; i < nums.length; i++) {
    if (used[i]) continue;
    if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue; // ← prune duplicates
    used[i] = true;
    path.add(nums[i]);
    backtrack(nums, used, path, result);
    path.remove(path.size() - 1);
    used[i] = false;
}
```

* * *

# 4. Optimization Ladder (Interview Narrative)

## Step 1 — Brute Force

* Generate all n! permutations and store them all
* For duplicates: generate all, deduplicate at the end using a Set
* For kth permutation: generate all permutations, sort, pick index k-1 → O(n! × n log n)

eg.

My first instinct: since we need all permutations, I'll generate every possible ordering — that's n! results for n elements. If the input has duplicates, I'll throw everything into a HashSet to deduplicate at the end.

```
// BRUTE: generate all n! permutations, deduplicate after
List<List<Integer>> permuteUnique_brute(int[] nums) {
    Set<List<Integer>> result = new HashSet<>();
    boolean[] used = new boolean[nums.length];
    backtrack(nums, used, new ArrayList<>(), result);
    return new ArrayList<>(result);
}

void backtrack(int[] nums, boolean[] used,
               List<Integer> path, Set<List<Integer>> result) {
    if (path.size() == nums.length) {
        result.add(new ArrayList<>(path));  // HashSet deduplicates
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        used[i] = true;
        path.add(nums[i]);
        backtrack(nums, used, path, result);
        path.remove(path.size() - 1);
        used[i] = false;
    }
}

Time:  O(n! × n)  — n! permutations, O(n) to copy each into the set
Space: O(n! × n)  — storing all n! permutations in the HashSet
```

## Step 2 — Optimization Insight

What repeating work are we eliminating?

* Duplicate permutations: if we prune during recursion (not after), we cut identical subtrees before entering them — O(unique permutations) instead of O(n!)
* Kth permutation: we don't need all permutations. At each position, there are (n-1)! completions. We can greedily select which digit fills position 0, then position 1, etc.

Core shift for kth permutation:
```
from: generate all n! permutations, sort, index into list
to:   factoriadic decomposition — at each step, compute which digit to pick using (k-1) / (n-1)!
```

eg.

The problem with this approach is that we're doing work we don't need to do. When the input has duplicates — say [1, 1, 2] — both 1s at position 0 generate completely identical subtrees. We recurse into them, generate the same permutations twice, then discard the duplicates after the fact. We're paying full price for work that produces nothing new.

The HashSet doesn't prevent the duplicate work — it only hides the duplicate output. We want to prevent the duplicate subtrees from being entered in the first place.

```
Input: [1a, 1b, 2]   (two identical 1s)

Subtree rooted at 1a (position 0):
    [1a, 1b, 2]
    [1a, 2, 1b]

Subtree rooted at 1b (position 0):   ← IDENTICAL WORK
    [1b, 1a, 2]  == [1,1,2]  already seen
    [1b, 2, 1a]  == [1,2,1]  already seen

We entered, computed, and discarded the entire right subtree.
```

## Step 3 — Optimal Approach

* Backtracking with `used[]`: O(n! × n) time, O(n) space
* Duplicate pruning: prunes redundant branches at each level — no set needed
* Next permutation: O(n) in-place, O(1) space — the standard library algorithm
* Kth permutation: O(n²) time (removing from list each step), O(n) space

eg.

If I sort the input first, all duplicate values become adjacent. Now I can detect at the point of choosing — before recursing — whether the current element would produce a subtree identical to one we already explored. I prune it right there. This eliminates duplicate subtrees entirely, not just duplicate outputs.

The condition !used[i-1] is the key. It means: the previous identical element is not currently on the path — it was tried at this level and already backtracked. Any subtree starting with nums[i] would be a carbon copy of that subtree. So we skip it before entering.

The two changes from brute force

```
Change 1: Arrays.sort(nums)                      ← puts duplicates adjacent
Change 2: if (i > 0 && nums[i] == nums[i-1]      ← same value as prior
             && !used[i-1]) continue;             ← prior already left this level
```

```
// OPTIMAL: prune duplicate subtrees during recursion
List<List<Integer>> permuteUnique(int[] nums) {
    Arrays.sort(nums);                            // ← Change 1
    List<List<Integer>> result = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    backtrack(nums, used, new ArrayList<>(), result);
    return result;
}

void backtrack(int[] nums, boolean[] used,
               List<Integer> path, List<List<Integer>> result) {
    if (path.size() == nums.length) {
        result.add(new ArrayList<>(path));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue; // ← Change 2
        used[i] = true;
        path.add(nums[i]);
        backtrack(nums, used, path, result);
        path.remove(path.size() - 1);
        used[i] = false;
    }
}

Time:  O(n! × n)  — worst case same (no duplicates), but prunes heavily with duplicates
Space: O(n)       — used[] array + recursion stack depth n
                    (no longer O(n! × n) — no HashSet storing all results in memory)
```
* * *

# 5. Core Patterns

```
Pattern 1 Distinct Backtracking   → used[] boolean; any unused element at each level
Pattern 2 Duplicate Pruning       → sort + used[]; skip identical-previous if not in use
Pattern 3 In-Place Swap           → swap elements into position; swap back on return
Pattern 4 In-Place Manipulation   → 3-step next permutation algorithm
```

## Pattern 1 — Distinct Backtracking

**When to use:**
* Input has no duplicate elements
* Need all n! permutations

**Mental model:**
```
At each recursive level, you fill one position of the output.
Try every element not currently on the path.
The used[] array is the "already chosen" guard.
```

```java
// ─────────────────────────────────────────────
// PATTERN 1 — Distinct Backtracking
// When: no duplicates, generate all n! permutations
// ─────────────────────────────────────────────
List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    backtrack(nums, used, new ArrayList<>(), result);
    return result;
}

void backtrack(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> result) {
    if (path.size() == nums.length) {
        result.add(new ArrayList<>(path));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        used[i] = true;
        path.add(nums[i]);
        backtrack(nums, used, path, result);
        path.remove(path.size() - 1);
        used[i] = false;
    }
}
```

**Why loop from 0 (not `start`):**
In subset/combination, `start` prevents picking earlier elements again (order doesn't matter).
In permutation, [1,2] and [2,1] are DIFFERENT — we must be able to pick any element at any position.
The `used[]` array prevents picking the *same element twice*, not picking an *earlier index*.

* * *

## Pattern 2 — Duplicate Pruning

**When to use:**
* Input contains duplicate values
* Need unique permutations only

**Mental model:**
```
Sort the input first (puts duplicates adjacent).
At each level, if we see nums[i] == nums[i-1] and nums[i-1] was NOT used
(meaning it already completed a subtree at this level), skip nums[i].
```

```java
// ─────────────────────────────────────────────
// PATTERN 2 — Duplicate Pruning
// When: input has duplicates, need unique permutations
// Sort first; skip if nums[i] == nums[i-1] && !used[i-1]
// ─────────────────────────────────────────────
List<List<Integer>> permuteUnique(int[] nums) {
    Arrays.sort(nums); // MUST sort first
    List<List<Integer>> result = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    backtrack(nums, used, new ArrayList<>(), result);
    return result;
}

void backtrack(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> result) {
    if (path.size() == nums.length) {
        result.add(new ArrayList<>(path));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue; // prune
        used[i] = true;
        path.add(nums[i]);
        backtrack(nums, used, path, result);
        path.remove(path.size() - 1);
        used[i] = false;
    }
}
```

**Why `!used[i-1]` (not `used[i-1]`):**
`used[i-1] == false` means nums[i-1] was tried at this recursive level and already returned.
Any subtree starting with nums[i] (identical value) would produce the exact same permutations.
So we skip it. If `used[i-1] == true`, nums[i-1] is on the current path at a different depth — nums[i] at THIS position forms a different arrangement, so we allow it.

* * *

## Pattern 3 — In-Place Swap

**When to use:**
* Distinct elements only
* Space-constrained — want to avoid the `used[]` array
* Often used for interview-style "generate all" questions to show O(1) extra space

**Mental model:**
```
Treat nums[start..end] as "remaining elements to place."
Swap nums[start] with nums[i] to "choose" element i.
Recurse with start+1.
Swap back to restore the array.
```

```java
// ─────────────────────────────────────────────
// PATTERN 3 — In-Place Swap
// When: distinct elements, want O(1) extra space
// ─────────────────────────────────────────────
void backtrack(int[] nums, int start, List<List<Integer>> result) {
    if (start == nums.length) {
        List<Integer> perm = new ArrayList<>();
        for (int n : nums) perm.add(n);
        result.add(perm);
        return;
    }
    for (int i = start; i < nums.length; i++) {
        swap(nums, start, i);          // choose: bring nums[i] to position start
        backtrack(nums, start + 1, result);
        swap(nums, start, i);          // un-choose: restore
    }
}
```

**Caveat:** This generates permutations but not in lexicographic order. Use Pattern 1 (with `used[]`) if lexicographic order matters.

* * *

## Pattern 4 — In-Place Next Permutation (3-Step Algorithm)

**When to use:**
* "Next permutation" / "previous permutation" problem
* Modify array in-place, O(n) time, O(1) space
* The same algorithm underlies #31, #556, #1053

**Mental model:**
```
A permutation in lexicographic order: to get the NEXT one,
find the rightmost "ascending break" and make the smallest valid swap.

Three steps:
1. Find rightmost i where nums[i] < nums[i+1]  ← the "break point"
2. Find rightmost j where nums[j] > nums[i]     ← smallest element larger than nums[i]
3. Swap nums[i] and nums[j]
4. Reverse nums[i+1..end]                        ← suffix becomes smallest possible
```

```java
// ─────────────────────────────────────────────
// PATTERN 4 — Next Permutation (In-Place)
// O(n) time, O(1) space
// ─────────────────────────────────────────────
void nextPermutation(int[] nums) {
    int n = nums.length;

    // Step 1: Find rightmost ascending break
    int i = n - 2;
    while (i >= 0 && nums[i] >= nums[i + 1]) i--;

    if (i >= 0) {
        // Step 2: Find rightmost element greater than nums[i]
        int j = n - 1;
        while (nums[j] <= nums[i]) j--;
        // Step 3: Swap
        swap(nums, i, j);
    }
    // Step 4: Reverse the suffix (i+1 to end)
    reverse(nums, i + 1, n - 1);
}
```

**Why reverse the suffix after swap:**
After swapping at position i, the suffix nums[i+1..end] is still in descending order.
The next permutation requires the smallest suffix — reverse makes it ascending (smallest).

* * *

# 6. Flagship Problem — Permutations II (#47)

This is the core problem of the archetype. Mastering duplicate pruning here unlocks every constrained-permutation variant.

**Problem:** Given a collection of numbers that might contain duplicates, return all possible unique permutations.

```
Step 1: Sort nums so duplicate values are adjacent
Step 2: Use used[] boolean array, same as distinct permutation
Step 3: Add pruning condition: skip nums[i] if nums[i] == nums[i-1] && !used[i-1]
Step 4: Collect when path.size() == nums.length
```

```java
class Solution {
    public List<List<Integer>> permuteUnique(int[] nums) {
        Arrays.sort(nums);                        // Step 1: sort
        List<List<Integer>> result = new ArrayList<>();
        boolean[] used = new boolean[nums.length];
        backtrack(nums, used, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] nums, boolean[] used,
                           List<Integer> path, List<List<Integer>> result) {
        if (path.size() == nums.length) {
            result.add(new ArrayList<>(path));    // snapshot
            return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (used[i]) continue;                // already on path
            // DUPLICATE PRUNING:
            // nums[i-1] identical to nums[i] and already returned at this level
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;

            used[i] = true;
            path.add(nums[i]);
            backtrack(nums, used, path, result);
            path.remove(path.size() - 1);         // un-choose
            used[i] = false;
        }
    }
}
```

**Trace: Input [1, 1, 2]:**
```
Sorted: [1, 1, 2]
Level 0:
  i=0: pick nums[0]=1 → used=[T,F,F], path=[1]
    Level 1:
      i=0: used → skip
      i=1: nums[1]=1, nums[0]=1, !used[0]=false → allowed (nums[0] IS used)
           pick 1 → path=[1,1]
        Level 2: only i=2 free → pick 2 → result [1,1,2]
      i=2: pick 2 → path=[1,2]
        Level 2: only i=1 free → pick 1 → result [1,2,1]
  i=1: nums[1]=1 == nums[0]=1, !used[0]=true → PRUNE ← prevents [1,...] duplicate tree
  i=2: pick nums[2]=2 → path=[2]
    Level 1:
      i=0: pick 1 → path=[2,1] → pick 1 → [2,1,1]
      i=1: nums[1]=1==nums[0]=1, !used[0]=true → PRUNE
Result: [[1,1,2], [1,2,1], [2,1,1]]
```

* * *

# 7. Key Tricks & Insights

## Trick 1 — Sorting unlocks duplicate pruning

Without sorting, identical values aren't adjacent, so you can't compare nums[i] to nums[i-1].
Always sort before any permutation problem that has potential duplicates.

## Trick 2 — The `!used[i-1]` direction is non-negotiable

```java
// ❌ wrong direction — prunes too aggressively (some valid perms missed)
if (i > 0 && nums[i] == nums[i-1] && used[i-1]) continue;

// ✅ correct — prunes only when prior identical element has already been tried at this level
if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;
```

Mnemonic: "Skip if the sibling already left the room" — `!used[i-1]` means it came, tried, and went.

## Trick 3 — Next permutation suffix must be reversed, not re-sorted

```java
// ❌ O(n log n) — works but slow
Arrays.sort(nums, i + 1, nums.length);

// ✅ O(n) — after the swap, suffix is always descending, so reverse = ascending
reverse(nums, i + 1, n - 1);
```

The suffix from i+1 onward is always in descending order before reversal, because you found i by walking from the right until you hit an ascending pair. Everything to the right of i was already descending.

## Trick 4 — Kth permutation: factoriadic decomposition

```
k = 1-indexed rank of desired permutation.
At each position, there are (n-1)! permutations starting with any given digit.
Digit index = (k-1) / (n-1)!
New k = (k-1) % (n-1)! + 1

Repeat for n positions, removing chosen digit from available list each time.
```

```java
String getPermutation(int n, int k) {
    int[] factorial = new int[n + 1];
    factorial[0] = 1;
    List<Integer> digits = new ArrayList<>();
    for (int i = 1; i <= n; i++) {
        factorial[i] = factorial[i - 1] * i;
        digits.add(i);
    }
    k--;                                    // convert to 0-indexed
    StringBuilder sb = new StringBuilder();
    for (int i = n; i >= 1; i--) {
        int idx = k / factorial[i - 1];
        sb.append(digits.get(idx));
        digits.remove(idx);
        k %= factorial[i - 1];
    }
    return sb.toString();
}
```

## Trick 5 — Permutation in String = sliding window on frequency

```java
// Don't backtrack — use a sliding fixed-size window of length s1.length()
// Compare frequency arrays: if they match, s2 contains a permutation of s1
boolean checkInclusion(String s1, String s2) {
    int[] freq = new int[26];
    for (char c : s1.toCharArray()) freq[c - 'a']++;
    int left = 0, count = s1.length();
    for (int right = 0; right < s2.length(); right++) {
        if (freq[s2.charAt(right) - 'a']-- > 0) count--;
        if (count == 0) return true;
        if (right - left + 1 == s1.length()) {
            if (freq[s2.charAt(left++) - 'a']++ >= 0) count++;
        }
    }
    return false;
}
```

* * *

# 8. Common Pitfalls (🚨 HIGH ROI)

## Pitfall 1 — Forgetting the un-choose step (used[i] = false)

```java
// ❌ never resets used — all indices eventually blocked, only first permutation found
used[i] = true;
path.add(nums[i]);
backtrack(nums, used, path, result);
path.remove(path.size() - 1);
// missing: used[i] = false;
```

```java
// ✅ always restore state — backtracking requires symmetric choose/un-choose
used[i] = true;
path.add(nums[i]);
backtrack(nums, used, path, result);
path.remove(path.size() - 1);
used[i] = false;     // ← the un-choose step
```

## Pitfall 2 — Wrong pruning direction for duplicates

```java
// ❌ used[i-1] — prunes when the prior IS on the path, missing valid arrangements
if (i > 0 && nums[i] == nums[i-1] && used[i-1]) continue;

// ✅ !used[i-1] — prunes when the prior already COMPLETED at this level
if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;
```

## Pitfall 3 — Forgetting to sort before duplicate pruning

```java
// ❌ nums not sorted — nums[i] == nums[i-1] check is meaningless
int[] nums = {2, 1, 1};  // duplicates not adjacent
// pruning fires incorrectly or not at all
```

```java
// ✅ always sort first
Arrays.sort(nums);
// now duplicates are adjacent → nums[i] == nums[i-1] is a safe identity check
```

## Pitfall 4 — Using start index in permutation (subset pattern)

```java
// ❌ subset pattern — misses permutations where later elements come first
for (int i = start; i < nums.length; i++) {  // wrong for permutations
```

```java
// ✅ permutation pattern — always iterate from 0, use used[] to prevent re-pick
for (int i = 0; i < nums.length; i++) {
    if (used[i]) continue;
```

## Pitfall 5 — Next permutation: reversing from wrong index

```java
// ❌ reverses from index i instead of i+1 — corrupts the swap position
reverse(nums, i, n - 1);

// ✅ reverse the suffix that starts AFTER the break point
reverse(nums, i + 1, n - 1);
```

* * *

# 9. Edge Case Checklist

* Empty array or single element → trivially one permutation (itself)
* All elements identical (e.g., [1,1,1]) → exactly one unique permutation
* k = 1 in kth permutation → return sorted order (lexicographically smallest)
* k = n! in kth permutation → return reverse-sorted order (lexicographically largest)
* Already last permutation in next-permutation → i never found; reverse entire array
* s1.length() > s2.length() in permutation-in-string → impossible, return false immediately
* Two strings with different character sets → impossible, frequency check catches it

* * *

# 10. Decision Framework (🔥 INTERVIEW WEAPON)

## Step 1: Does order matter?

→ YES → permutation (not subset/combination)
→ NO  → Subset / Power Set (Archetype 10)

## Step 2: What type of permutation problem?

```
Generate all arrangements?            → Backtracking (Pattern 1 or 2)
In-place modify to next/prev?         → 3-Step In-Place Algorithm (Pattern 4)
Find kth permutation?                 → Factoriadic Math
Check if string contains permutation? → Sliding Window + freq array (not backtracking)
Count valid arrangements with constraint? → Backtracking with pruning
```

## Step 3: Does input have duplicates?

→ YES → sort first, add `!used[i-1]` pruning condition (Pattern 2)
→ NO  → plain `used[]` array (Pattern 1)

## Step 4: Memory constraint?

→ O(1) extra → In-Place Swap (Pattern 3, distinct only)
→ No constraint → `used[]` boolean array

### Permutation Decision Tree

```
Problem involves ordering of elements?
└── YES
    ├── Generate all orderings?
    │   ├── No duplicates?            → Distinct Backtracking (used[])
    │   └── Duplicates present?       → Sort + Duplicate Pruning (!used[i-1])
    │
    ├── Modify in-place to next/prev?
    │   └──                           → 3-Step: find break, swap, reverse suffix
    │
    ├── Find kth arrangement?
    │   └──                           → Factoriadic: (k-1) / (n-1)! per position
    │
    ├── Check if window contains permutation?
    │   └──                           → Sliding Window + freq count (not backtracking)
    │
    └── Count arrangements with constraint?
        └──                           → Backtracking + pruning at constraint check
```

* * *

# 11. Problem Map Quick Reference

```
PATTERN                          PROBLEMS
────────────────────────────────────────────────────────────────────────
Distinct Backtracking            #46 Permutations
                                 #784 Letter Case Permutation
Duplicate Pruning                #47 Permutations II
                                 #996 Number of Squareful Arrays
In-Place Swap                    #46 (alternate implementation)
In-Place Next/Prev               #31 Next Permutation
                                 #556 Next Greater Element III
                                 #1053 Previous Permutation With One Swap
Kth Permutation (Factoriadic)    #60 Permutation Sequence
Sliding Window (freq check)      #567 Permutation in String
                                 #438 Find All Anagrams in a String
Constrained Backtracking         #526 Beautiful Arrangement
                                 #291 Word Pattern II
                                 #351 Android Unlock Patterns
Permutation in Disguise          #41 First Missing Positive
                                 #299 Bulls and Cows
                                 #179 Largest Number (sort = permutation)
```

* * *

# 12. Drill Section (Mastery Check)

## Verbal Drill

Explain this archetype in 2 minutes.

Target structure:
1. Permutation = ordered arrangement of ALL elements; order matters, unlike subset
2. Universal engine: backtracking with `used[]` — pick any unused element at each depth
3. Duplicate handling: sort first, then skip nums[i] == nums[i-1] && !used[i-1]
4. Special cases: next permutation uses 3-step in-place; kth uses factoriadic
5. String "contains permutation" → sliding window on freq array, not backtracking

## Recognition Drill

Given a problem, identify the pattern before coding:

* "All arrangements / all permutations" → backtracking + used[]
* Input has duplicates? → sort + !used[i-1] pruning
* "Next permutation" / "previous permutation" → 3-step in-place
* "Kth permutation" → factoriadic digit selection
* "Does string contain permutation of s1?" → sliding window, fixed-size window = len(s1)
* "Count valid arrangements where ..." → backtracking + constraint check at each step

## Transformation Drill

Convert brute → optimal without coding:

* **All Permutations with Duplicates**
    * Brute: generate all n! permutations → deduplicate via HashSet → O(n! × n) + memory
    * Optimal: prune at each level during recursion → only unique permutations generated

* **Kth Permutation**
    * Brute: generate all permutations, sort, return index k-1 → O(n! × n log n)
    * Optimal: factoriadic — O(n²) by selecting one digit per position

* **Permutation in String**
    * Brute: generate all permutations of s1, check if any is a substring → O(n! × m)
    * Optimal: sliding window of fixed size len(s1) on s2, compare freq arrays → O(n + m)

## Primitive Speed Test

Write each from scratch with no reference. Target: < 3 min each, zero bugs.

```
□ permute(int[] nums)                      ← distinct, used[] array
□ permuteUnique(int[] nums)                ← sort + !used[i-1] pruning
□ nextPermutation(int[] nums)              ← 3-step in-place
□ getPermutation(int n, int k)             ← factoriadic
□ checkInclusion(String s1, String s2)    ← sliding window freq
```

* * *

# 13. Problem Approach Template

```
1. Is this a permutation? (order matters, use ALL elements)
   → YES if: "all arrangements", "next/kth permutation", "permutation in string"
   → NO  → Subset if order doesn't matter (Archetype 10)

2. What variant?
   → Generate all?         → backtracking + used[]
   → In-place next/prev?   → 3-step algorithm
   → Kth?                  → factoriadic math
   → Check substring?      → sliding window

3. Duplicates?
   → YES → sort first + !used[i-1] pruning
   → NO  → plain used[] boolean array

4. What does the base case look like?
   → path.size() == nums.length → snapshot current path

5. What is the constraint check inside the loop?
   → if (used[i]) continue
   → if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue   [duplicates only]
   → additional problem-specific constraint (e.g., adjacency check for #996)
```

```
# Permutation — Problem Quick Drill

Signal check:
- Why permutation? (order matters + all elements used)

Pattern:
- □ Distinct Backtracking (used[])
- □ Duplicate Pruning (sort + !used[i-1])
- □ In-Place Swap
- □ In-Place Next Permutation (3-step)
- □ Factoriadic (kth)
- □ Sliding Window (permutation in string)

Duplicates in input?
- YES / NO (if YES: sort first)

Base case:
- path.size() == ___

Global variable needed?
- YES / NO

Constraint inside loop:
- skip if: ___

Edge cases:
- empty / single element?
- all identical?
- k = 1 or k = n!?
```

## Problem Drills

```
1. #46 Permutations
Pattern: Distinct Backtracking (used[])

Null/base returns: path.size() == nums.length → snapshot

Approach:
1. for i in 0..n-1: if used[i] skip
2. used[i] = true; path.add(nums[i])
3. recurse
4. path.remove(); used[i] = false

Invariant:
- path at any depth contains distinct elements (enforced by used[])
- exactly one copy of each element on any root-to-leaf path in recursion tree

Edge cases:
- n=1: loop runs once, base case immediately → [[nums[0]]] ✓
- n=0: base case at depth 0 → [[]] ✓


2. #47 Permutations II
Pattern: Duplicate Pruning (sort + !used[i-1])

Pre-step: Arrays.sort(nums)

Approach:
1. if used[i] → skip
2. if i > 0 && nums[i] == nums[i-1] && !used[i-1] → skip (PRUNE)
3. choose, recurse, un-choose

Invariant:
- for any set of identical values, we always pick the leftmost one first at each level
- each unique value-sequence appears exactly once

Edge cases:
- all same [1,1,1]: prune fires at i=1 and i=2 at every level → only [1,1,1] ✓
- two duplicates [1,1,2]: 3 unique permutations ✓


3. #31 Next Permutation
Pattern: In-Place 3-Step

Approach:
1. Find i from right where nums[i] < nums[i+1]   (break point)
2. If i found: find j from right where nums[j] > nums[i]; swap(i,j)
3. Reverse nums[i+1..n-1]
4. If i not found (entire array descending): reverse entire array

Invariant:
- After step 1+2: nums[i] is now the next-larger value
- After step 3: suffix is minimized (ascending order)

Edge cases:
- [3,2,1]: i never found → reverse entire array → [1,2,3] ✓
- [1,2,3]: i=1, j=2, swap → [1,3,2], reverse [2] → [1,3,2] ✓


4. #60 Permutation Sequence
Pattern: Factoriadic

Pre-step: build factorial[] and digits list [1..n]; k-- (0-indexed)

Approach:
1. for i = n down to 1:
   idx = k / factorial[i-1]
   append digits[idx]
   digits.remove(idx)
   k %= factorial[i-1]

Invariant:
- At each position, (n-1)! permutations start with each digit
- idx selects which digit fills current position
- k updated to rank within remaining digits

Edge cases:
- k=1 → k-- = 0 → idx=0 always → sorted order ✓
- k=n! → k-- = n!-1 → idx=n-1 always → reverse sorted ✓


5. #567 Permutation in String
Pattern: Sliding Window + freq count

Approach:
1. Build freq[26] from s1
2. count = s1.length() (target matches needed)
3. Slide window of size s1.length() over s2:
   - if freq[right]-- > 0: count-- (matched a needed char)
   - if count == 0: return true
   - if window full: if freq[left]++ >= 0: count++ (un-match leaving char)

Invariant:
- count == 0 iff the window is a permutation of s1
- freq tracks "still needed" — negative means over-supplied

Edge cases:
- s1.length() > s2.length(): window never fills → return false ✓
- s1 == s2: single window covers both → match at step n ✓
```
