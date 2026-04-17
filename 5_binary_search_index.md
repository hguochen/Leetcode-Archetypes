# 🧩 Archetype 5 Binary Search(index)

* * *

# 0. Goal

## What problem class does this solve?

* Problems where the answer lives in an ordered index space
* Problems where a test at mid lets you prove one half is impossible

## What mastery looks like

* Can identify binary search problems in <30 seconds
* Can derive optimal solution without memorization
* Can explain brute → optimal transition clearly
* Can define search space + invariant BEFORE coding
* Can derive:
    * exact match
    * lower/upper bound
    * rotated/peak patterns
* Can explain why half is eliminated each step

* * *


# 1. Recognition Signals (🚨 MOST IMPORTANT)

## Strong signals

* Sorted array
* Find in O(log n)
* "Find first/last occurrence
* Search Insert position
* Smallest index satisfying
* Find minimum / peak
* Search in rotated array
* Find boundary

## Weak / disguised signals

*     2D matrix that can be flattened or row-searched
*    Search over sorted auxiliary data, not original input
*    Latest value <= target
*    Mountain array / bitonic structure
*    Queries over version history
*    Right-interval style problems after preprocessing
*    Index-to-interval mapping where sort order differs from original order

## Anti-signals (when NOT to use)

* No monotonicity or ordering
* Need all combinations
* Need global aggregation over all elements
* Unsorted array where comparing to mid does NOT eliminate half

* * *


# 2. Core Mental Model

## Core Ideas

* Binary search is not "search sorted array"
* Binary search = eliminate half of an ordered search space
* **Every problem needs:**
    * **search space**
    * **invariant**
    * **shrink rule**
    * **return meaning**
* There are 3 main families:
    * exact match
    * boundary search
    * structural search

## Problem Transformation

Original Problem → What does it reduce to?

Most Binary Search(index) problems reduce to one of these 3 models.


### A. Exact Match

Find an index such that:

```
nums[i] == target
```

#### Invariant

```
answer ∈ [left, right]
```

This is the classic sorted lookup.


### B. Boundary Search

Find the first index where a condition becomes true.

#### Invariant

```
answer ∈ [left, right)
```


Example:

```
nums[i] >= target
```

This produces a monotonic boolean pattern:

```
F F F F T T T T
        ^
      answer
```

This is the deepest and most reusable model


### C. Structural Search

The array is not globally sorted, but still has enough structure to choose a direction.

Examples:

* rotated sorted array
* peak element
* mountain array
* single non-duplicate in sorted pairs

You are not asking whether the full left side is globally better. You are asking:

```
Which side is guaranteed to still contain the answer?
```



## Why Binary Search works

Binary search works because the search space has directional structure:

* sorted order
* monotonic predicate
* one sorted half in rotated array
* slope direction in peak problems
* sorted timestamps in history problems

One Check at mid is enough to prove an entire region impossible.
That is the real abstraction:

```
Binary Search = proof-based elimination
```

## 

## Visualization

### Boundary Search

```
index:      0 1 2 3 4 5 6 7
condition:  F F F F F T T T
                      ^
                    boundary
```

### Exact Match

```
[left ........ mid ........ right]
 if nums[mid] < target -> discard left half including mid
 if nums[mid] > target -> discard right half including mid
```

### Peak Search

```
nums[mid] < nums[mid + 1]  -> rising slope -> peak must exist on right
nums[mid] > nums[mid + 1]  -> falling slope -> peak exists at mid or left
```

* * *


# 3. Optimization Ladder (Interview Narrative)

## Step 1 — Brute Force

* Scan every index
* Check whether it matches target / boundary / structural rule



* Time: O(n)
* Why it's too slow
    * ignores sorted / monotonic structure
    * repeatedly examines elements in regions already implied impossible

## Step 2 — Optimization Insight

* What repeating work are we eliminating?

We are eliminating entire regions after one comparison.

Examples:

* If `nums[mid] < target`, everything at or left of mid is impossible for exact match in sorted array.
* if `nums[mid] >= target` and we want the first such index, the answer cannot be strictly right of mid only, so we keep mid and shrink right.
* if left half is sorted in a rotated array and target is not in that half, we discard it entirely.

Core Shift

```
from checking elements to discarding regions
```

## Step 3 — Optimal Approach

* Key idea
    * maintain an interval guaranteed to contain the answer
    * use mid to shrink that interval by half
* Time: O(logn)
* Space: O(1)

* * *

# 4. Core Patterns

## Pattern 1 - Exact Match

Mental Model:

```
equal -> found
mid is smaller -> go right
mid is larger -> go left
```

```
int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (nums[mid] == target) return mid;
        if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;
}
```

When to use:

* sorted array, find target

Template idea:

* closed interval
* compare nums[mid] with target
* discard one side completely

* * *

## Pattern 2 - Boundary Search



### Pattern 2.1 - Lower Bound / First index with nums[i] >= target

Mental Model:

```
I am not searching for equality.
I am searching for the earliest index where the property becomes true.
```

```
int lowerBound(int[] nums, int target) {
    int left = 0, right = nums.length; // [left, right)
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        
        if (nums[mid] >= target) {
            right = mid; // keep mid
        } else {
            left = mid + 1; // discard mid
        }
    }
    return left;
}
```

When to use:

* insertion position
* first occurrence
* first bad version
* first index satisfying a predicate

Template idea:

* half-open interval
* if condition(mid) is true, keep mid and shrink right
* else discard mid and move left



### Pattern 2.2 - Upper Bound / First index with nums[i] > target

Mental Model:

```
right boundary = first greater - 1
```

```
int upperBound(int[] nums, int target) {
    int left = 0, right = nums.length; // [left, right)
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        
        if (nums[mid] > target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left;
}
```

When to use:

* first index > target
* last occurrence
* rightmost valid position

Template idea:

* search the first strictly greater value
* derive last occurrence as upperBound(target) - 1



### Pattern 2.3 - First / Last Occurrence

```
int first = lowerBound(nums, target);
if (first == nums.length || nums[first] != target) {
    return new int[]{-1, -1};
}
int last = upperBound(nums, target) - 1;
return new int[]{first, last};
```

* * *

## Pattern 3 - Structural Binary Search

Binary search on a space where:

* array is NOT globally sorted
* but has enough structure to eliminate half

Core Idea:

```
You don't need full monotonicity.
You only need a directional guarantee.

Use structure -> decide which side still contains answer
```



### Pattern 3.1 - Rotated Array Search

>This is a variant of Structural Binary Search

Mental model:

```
which half still has reliable structure?
```

```
int search(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (nums[mid] == target) return mid;
        
        if (nums[left] <= nums[mid]) { // left half sorted
            if (nums[left] <= target && target < nums[mid]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        } else { // right half sorted
            if (nums[mid] < target && target <= nums[right]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
    }
    return -1;
}
```

When to use:

* sorted array rotated around a pivot

Template idea:

* at least one half is sorted
* identify the sorted half first
* determine whether target/min lies inside it



### Pattern 3.2 - Peak Search

>This is a variant of Structural Binary Search

Mental Model:

```
I do not need global sorting.
I only need directional guarantee.
```

```
int findPeakElement(int[] nums) {
    int left = 0, right = nums.length - 1;
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        
        if (nums[mid] < nums[mid + 1]) { // peak on the right
            left = mid + 1; // rising slope → peak is to the right
        } else { // peak on the left
            right = mid; // falling slope → peak is at mid or left
        }
    }
    return left; // left == right, converged on peak
}
```

When to use:

* local directional information from neighbors

Template idea:

* compare nums[mid] with nums[mid + 1]
* rising slope means peak must be right
* falling slope means peak is at mid or left



### Pattern 3.3 - Sorted History Search

Mental model:

```
search the timeline, not the object
```

```
int rightmostLE(int[] arr, int target) {
    int left = 0, right = arr.length; // first index > target
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] > target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return left - 1;
}
```

When to use:

* timestamps
* snapshots
* historical versions

Template idea:

* binary search on sorted metadata
* usually rightmost value <= query

### Pattern 3.4 - Find min in sorted rotated array search

When to use

* finding minimum value in rotated array search

Template idea

* find high/low segment in the array
* minimum must lie in low segment

```
// canonical
// handles duplicates
int left = 0, right = nums.length - 1;

while (left < right) {
    int mid = left + (right - left) / 2;

    if (nums[mid] > nums[right]) {
        left = mid + 1;
    } else if (nums[mid] < nums[right]) {
        right = mid;
    } else {
        right--;
    }
}
return nums[left];
```

```
// handles duplicates
int left = 0, right = nums.length - 1;

while (left < right) {
    int mid = left + (right - left) / 2;

    if (nums[left] == nums[mid] && nums[mid] == nums[right]) {
        left++;
        right--;
        continue;
    }
    if (nums[mid] > nums[right]) {
        // right region is low segment, discard left, discard mid
        left = mid + 1;
    } else {
        // right region is high segment, discard right, preserve mid
        right = mid;
    }
}
return nums[left];
```

### Pattern 3.5 - Build sorted history per entity

Preprocess : build sorted list per key (timestamps always increasing → append-only)
Query : upper bound on timestamp → first entry past queryTime
Answer : left - 1 → last entry at or before queryTime
Guard : left == 0 → no valid entry exists

```
// Preprocess: build sorted history per entity (timestamps, snapshots, etc.)
// At query time:

int left = 0, right = history.size();

while (left < right) {
    int mid = left + (right - left) / 2;

    if (history.get(mid).timestamp > queryTime) {
        right = mid;
    } else {
        left = mid + 1;
    }
}

if (left == 0) return DEFAULT;      // nothing at or before queryTime
return history.get(left - 1).value; // last valid entry
```

### Pattern 3.6 - Index-Preserving Sort (➕ added)

Mental model:

```
sort by a derived key, but keep original index linkage intact
```

When sorting `intervals` directly would destroy original index information, use an auxiliary index array instead.

When to use:

* need sorted binary search access over a derived key (e.g. start values)
* answer must be returned as an original index
* LC 436 Find Right Interval, LC 2300 Successful Pairs of Spells and Potions

```
// Step 1: build aux array of original indices
Integer[] aux = new Integer[n];
for (int i = 0; i < n; i++) aux[i] = i;

// Step 2: sort aux by the derived key (e.g. interval start value)
// Note: Integer[] required — Arrays.sort with Comparator needs object array
Arrays.sort(aux, (a, b) -> intervals[a][0] - intervals[b][0]);

// After sort:
//   aux[k] = original index of the k-th interval by start value
//   starts in aux order are now monotonically non-decreasing → binary search works

// Step 3: lower bound binary search on aux
int left = 0, right = aux.length;
while (left < right) {
    int mid = left + (right - left) / 2;
    if (condition(intervals[aux[mid]])) {
        right = mid;
    } else {
        left = mid + 1;
    }
}

// left == aux.length  → no valid interval exists → return -1
// otherwise          → aux[left] is the original index of the answer
result[i] = (left == aux.length) ? -1 : aux[left];
```

Key detail — comparator mechanics:

```
(a, b) -> intervals[a][0] - intervals[b][0]

a, b    = values from aux = original indices into intervals
negative return  → a's start < b's start → a comes first
positive return  → a's start > b's start → b comes first
```

* * *

# **5. Problem Map (➕ added)**

Every problem from this archetype mapped to its pattern:

| # | Problem | Pattern |
|---|---|---|
| #704 | Binary Search | Exact Match |
| #35 | Search Insert Position | Lower Bound |
| #34 | First and Last Position of Element | Lower Bound + Upper Bound |
| #33 | Search in Rotated Sorted Array | Structural — Rotated |
| #81 | Search in Rotated Sorted Array II | Structural — Rotated (duplicates) |
| #153 | Find Minimum in Rotated Sorted Array | Structural — Find Min Rotated |
| #154 | Find Minimum in Rotated Sorted Array II | Structural — Find Min Rotated (duplicates) |
| #162 | Find Peak Element | Structural — Peak |
| #1901 | Find a Peak Element II | Structural — Peak (2D) |
| #278 | First Bad Version | Lower Bound |
| #69 | Sqrt(x) | Upper Bound |
| #540 | Single Element in a Sorted Array | Structural — Pair Alignment |
| #875 | Koko Eating Bananas | Lower Bound (Binary Search on Answer) |
| #702 | Search in Sorted Array of Unknown Size | Hidden — Exact Match |
| #1064 | Fixed Point | Lower Bound (hidden) |
| #1146 | Snapshot Array | Sorted History Search |
| #981 | Time Based Key-Value Store | Sorted History Search |
| #911 | Online Election | Sorted History Search + Precompute |
| #436 | Find Right Interval | Index-Preserving Sort + Lower Bound |
| #2300 | Successful Pairs of Spells and Potions | Sort + Lower Bound |

* * *

# **6. Key Tricks & Insights**

## Trick 1 - Convert the problem to boolean boundary search whenever possible.

Examples:

* insert position
* first occurrence
* first bad version
* first timestamp greater than query

All become:

```
find first true
```



## Trick 2 - Choose interval style first

* exact match → usually `[left, right]`
* boundary search → usually `[left, right)`

Do not mix them mid solution



## Trick 3 - Right boundary is easier through upper bound

```
last occurrence = upperBound(target) - 1
```

This is usually cleaner than inventing a second custom search.


## Trick 4 - In rotated arrays, discover structure before target logic

Order of reasoning:

1. which half is sorted?
2. does target/min lie there?
3. keep or discard



## Trick 5 - Peak problems are slope problems, not target problems.

You are not comparing with a target value.
You are comparing local direction around `mid`.


## Trick 6 - Binary search often lives on derived arrays.

Examples:

* timestamps list
* sorted interval starts
* snapshot ids
* auxiliary sorted values

## Trick 7 - When sorting destroys index info, sort indices instead. (➕ added)

If the answer requires original indices but binary search requires sorted order:

```
Integer[] aux = indices sorted by derived key
binary search on aux → aux[left] recovers original index
```

* * *

# **7. Common Pitfalls (🚨 HIGH ROI)**

## Pitfall 1 - Mixing interval styles.

Typical bug:

* writing `[left, right)`  updates
* but using `while(left <=right)`

This causes off-by-one errors or infinite loops


## Pitfall 2 - Not defining  what the returned index means

Before coding, know whether left means:

* target index
* insertion position
* first valid index
* first invalid index
* peak location
* rightmost valid history entry



## Pitfall 3 - Using binary search when the condition is not monotonic.

If the predicate behaves like:

```
F T F T F
```

binary search is invalid.


## Pitfall 4 - Duplicates can break clean structure.

Example:

* rotated array with duplicates
* `nums[left] == nums[mid] == nums[right]`

Now you may need cautious shrinking instead of direct half elimination.


## Pitfall 5 - Boundary search without post-validation

Lower bound gives a candidate index, not guaranteed existence.


## Pitfall 6 - Coding too early

If you cannot state:

* search space
* invariant
* shrink rule
* return meaning

the implementation will be unstable.

## Pitfall 7 - Integer overflow in product comparisons (➕ added)

When checking `spell * potion >= success` and values can reach 10^5:

```
spell * potion  →  up to 10^10  →  exceeds int range (2.1 × 10^9)
```

Fix: cast one operand to long before multiplying:

```java
(long) potions[mid] * spells[i] >= success   // ✅
potions[mid] * spells[i] >= success           // ❌ overflow
```

* * *

# **8. Edge Case Checklist**

These are the edge cases you must check for before coding

* empty input
* single element
* duplicates
* negative numbers
* overflow / large input
* boundaries (left / right)

Binary search specific additions:

* target smaller than all elements
* target larger than all elements
* all elements same
* pivot at edge in rotated array
* peak at index 0 or n - 1
* timestamp query earlier than all histories
* timestamp query later than all histories

* * *

# **9. Decision Framework (🔥 INTERVIEW WEAPON)**

## **Step 1: Is the data structure…**

Array / String → strong candidate
Tree → usually no, unless BST or sorted projection
Graph → usually no

Better question:

```
Can I define an ordered search space over indices or sorted metadata?
```



## **Step 2: Is the problem asking for…**

Contiguous → probably sliding window / prefix sum
Pair? → probably hashmap / two pointers
Optimization → maybe Binary Search on Answer
Exact index / boundary / insert position/peak/rotated minimum? → strong yes

## **Step 3: Key question**

For this archetype, refine it to:

>"What ordered search space contains the answer, and what mid-test lets me eliminate half?"



### Binary Search decision tree

if asking for exact value in sorted array
Use:

* exact match template

If asking for first/last/insert/bad version
Use:

* lower bound / upper bound template

If array is rotated
Ask:

1. which half is sorted?
2. is the answer inside that half?

If peak / mountain
Ask:

1. what does slope at mid say?
2. where must a peak exist?

If timestamps / snapshots
Ask:

1. are histories sorted?
2. do i need latest <= query?
3. binary search the rightmost valid history

If need sorted access but original indices matter
Ask:

1. can I sort an aux index array by the derived key?
2. binary search on aux → return aux[left] as original index

* * *

# **10. Problem Map Quick Reference (➕ added)**

```
PATTERN                     PROBLEMS
─────────────────────────────────────────────────────
Exact Match                 #704, #33 (with structure)
Lower Bound                 #35, #278, #34(first), #1064, #2300
Upper Bound                 #69, #34(last)
Rotated Search              #33, #81
Find Min Rotated            #153, #154
Peak Search                 #162, #1901
Pair Alignment              #540
Sorted History              #981, #1146, #911
Index-Preserving Sort       #436
Sort + Lower Bound          #2300
```

* * *

# **11. Cheat Sheet (1-Pager)**

```
WHEN TO USE:
- Sorted array + need index / boundary / threshold
- Monotonic predicate over indices
- Versioned / historical queries (timestamps, snapshots)
- Interval / pair problems after preprocessing
- Any problem where one comparison at mid eliminates half

CORE IDEA:
- Binary search = proof-based elimination
- Maintain invariant: answer always in current window
- One check at mid proves an entire region impossible

INVARIANT:
- Exact match:    answer in [left, right]   (closed)
- Boundary:       answer in [left, right)   (half-open)
- Structural:     answer in [left, right]   (closed, exact match style)

TEMPLATE SELECTOR:
- Find exact value?              → Exact Match   left <= right, return index/-1
- Find first/last/insert?        → Lower/Upper   left < right,  return left
- Rotated array?                 → Structural    left <= right, check sorted half first
- Peak / slope?                  → Structural    left < right,  right = mid (not mid-1)
- Timestamps / snapshots?        → History       left < right,  return left-1
- Sort + original index needed?  → Aux sort      Integer[] aux, return aux[left]

TIME/SPACE:
- Query:         O(log n),     O(1) space
- With sort:     O(n log n),   O(n) space (aux array)
- With precomp:  O(n) setup,   O(log n) query

TOP 3 TRICKS:
1. Convert to first-true boundary search whenever possible → [F,F,F,T,T,T]
2. Closed [left, right] for exact match; half-open [left, right) for boundaries — never mix
3. Use left-1 for rightmost valid value (history/snapshot queries); guard left == 0

TOP 3 PITFALLS:
1. Mixing interval styles → off-by-one or infinite loop
2. right = mid-1 in peak/boundary search → skips the answer at mid
3. Integer overflow in product comparisons → cast to (long) before multiplying
```

* * *

# **12. Drill Section (Mastery Check)**

## **Verbal Drill**

Explain this archetype in 2 minutes.

Target structure:

1. Binary search works on ordered search spaces
2. Main families are exact match, boundary, and structural search
3. The invariant is that the answer remains inside the current interval
4. Each mid-check discards half safely
5. Typical complexity is `O(logn)` and `O(1)` space

## **Recognition Drill**

Given a problem, justify why this applies

Ask yourself:

* is there a sorted /monotonic structure?
* Am i searching for an index or boundary?
* Can i rule out half after one comparison?
* Is this exact match or first true?

## **Transformation Drill**

Convert brute → optimal without coding
Examples:

* Search Insert Position
    * brute: scan utility first >= target
    * optimal: lower bound / first true
* First Bad Version
    * brute: test every version
    * optimal: boundary `F F F T T T`
* Peak Element
    * brute: scan all indices
    * optimal: slope comparison tells direction
* TimeMap get
    * brute: scan version history backward
    * optimal: binary search rightmost timestamp `<= query`

* * *

# **13. Problem Approach Template**

Use this template to approach every Binary Search problem

```
# Binary Search Quick Drill

Signal check:
- why BS?

Family:
- Exact / Boundary / Structural

Search space:
- 

Invariant:
- answer in 

Shrink rule:
- if ..., discard ...
- else, discard ...

Return meaning:
- 

Edge cases:
- 
```



```
# Binary Search Problems Approach Template

# 0. Does the problem have any of these signals?

## Strong Signals

* Sorted array
* Find in O(log n)
* "Find first/last occurrence
* Search Insert position
* Smallest index satisfying
* Find minimum / peak
* Search in rotated array
* Find boundary

Consider binary search as an approach

## Anti Signals

* No monotonicity or ordering
* Need all combinations
* Need global aggregation over all elements
* Unsorted array where comparing to mid does NOT eliminate half

DO NOT use Binary Search approach


# 1. Binary Search Problem Drills

Before you start coding, write out the following details about the problem. If you can't find a reasonable answer to either of these specifications, reconsider using binary search as an approach.

1. search space
2. invariant
3. shrink rule
4. return meaning
5. problem family

## 1.1 Predicate

Take a problem statement and rewrite it as:

What am i searching for?
What condition becomes true?
What is the search space?

Examples:

Search Insert Position
-> first index where nums[i] >= target

First Bad Version
-> first version where isBad(version) == true

Find first occurrence
-> first index where nums[i] >= target, then validate equality

## 1.2 Invariant

For any binary search problem, say:

The answer is always inside ______

Examples:

[left, right]
[left, right)

If you cannot state the invariant, you are coding too early.


## 1.3 Shrink rule

For each problem, force this question

Why is this half impossible?

NOT:

Which half feels right?

Binary search is proof, not intuition.


## 1.4 Return meaning

At the end of the loop, ask

What does left mean now?

Examples:

left = insertion position
left = first true index
left = peak index
left - 1 = rightmost value <= target
left - 1 = last valid history entry (guard: left == 0 → no entry)

This single habit prevents many bugs.


## 1.5 Problem Family

Which problem family does the solution belong to?

- Exact Match
- Boundary Search
- Structural Search
```

In addition, when a binary search problem feels weird, use this checklist:

```
1. What pattern is regular? eg.LC540
2. Where does that pattern break?
3. Can mid tell me whether I'm before or after the break?
4. Do I need to normalize mid to inspect the right unit?
5. Once normalized, can I eliminate half safely?
```

```
// fill out this template to get a better idea 
Pattern:
Break:
Signal at mid:
Normalization needed?
Elimination rule:

// eg. LC540
Pattern: elements appear in pairs
Break: one single element shifts pair alignment
Signal at mid: does nums[mid] pair correctly with nums[mid+1]?
Normalization: make mid even
Elimination: valid pair -> go right, broken pair -> go left
```

Drills

```
1. #704 Binary Search - Find target in sorted array
Pattern:
- array is globally sorted in non-decreasing order

Break:
- none; this is direct exact-match search, not structure-break search

Mid Signal:
- nums[mid] == target
- nums[mid] < target
- nums[mid] > target

Normalization:
- none

Elimination:
- nums[mid] < target -> discard left half including mid
- nums[mid] > target -> discard right half including mid

2. #35 Search Insert Position - Find index where target should be inserted
Pattern:
- monotonic predicate: [F, F, F, T, T]

Break:
- first index where nums[i] >= target

Mid Signal:
- nums[mid] >= target
- nums[mid] < target

Normalization:
- none

Elimination:
- nums[mid] >= target -> keep mid, discard right side beyond mid
- nums[mid] < target -> discard left side including mid

3. #278. First Bad Version - Find first version where isBadVersion = true
Pattern:
- monotonic predicate: [F, F, F, T, T]

Break:
- first true version

Mid Signal:
- isBadVersion(mid) == false
- isBadVersion(mid) == true

Normalization:
- none

Elimination:
- if isBadVersion(mid) == false
    - discard mid and everything before it
- if isBadVersion(mid) == true
    - keep mid
    - discard everything after it

4. #540 Single element in a sorted array - Pairs + one single element
Pattern:
- before the single element, pairs start at even indices

Break:
- the single element breaks pair alignment
- after the single element, pairs start at odd indices

Mid Signal:
- after making mid even:
    - nums[mid] == nums[mid+1] -> pair structure is still valid
    - nums[mid] != nums[mid+1] -> pair structure breaks here

Normalization:
- if mid is odd, shift it left by 1 so mid points to the first index of a pair

Elimination:
- valid pair -> single element must be on the right
- broken pair -> single element is at mid or on the left

5. #153. Find Minimum in Rotated Sorted Array - Rotated sorted array, no duplicates
Pattern:
- rotated array consists of a high segment followed by a low segment
- the minimum is the first element of the low segment

Break:
- the pivot is where the array transitions from high segment to low segment

Mid Signal:
- nums[mid] > nums[right]
- nums[mid] <= nums[right]

Normalization:
- none

Elimination:
- nums[mid] > nums[right]
    -> mid is in the high segment
    -> minimum must be on the right
    -> discard left side and mid
- nums[mid] <= nums[right]
    -> mid is in the low segment, or mid is the minimum
    -> minimum is at mid or on the left
    -> keep mid, discard right side beyond mid

6. #33. Search in Rotated Sorted Array - Search target in rotated array
Pattern:
- at any step, at least one half of the rotated array is sorted

Break:
- global sortedness is broken by the rotation pivot
- but one half remains locally sorted at each step

Mid Signal:
- if nums[left] <= nums[mid], left half is sorted
- else, right half is sorted

Normalization:
- none

Elimination:
- if left half is sorted:
    - if nums[left] <= target < nums[mid], keep left half
    - else, discard left half and go right
- if right half is sorted:
    - if nums[mid] < target <= nums[right], keep right half
    - else, discard right half and go left

7. #162. Find Peak Element - Find any peak in 1D array
Pattern:
- array has a rising slope and a falling slope

Break:
- the peak is the turning point between the rising and falling slopes

Mid Signal:
- nums[mid] < nums[mid + 1]

Normalization:
- none

Elimination:
- if nums[mid] < nums[mid + 1]
    -> we are on a rising slope
    -> peak must be on the right
    -> discard left and mid
    -> left = mid + 1
- else
    -> we are on a falling slope
    -> peak is at mid or on the left
    -> keep mid
    -> right = mid

8. #1901. Find a Peak Element II - Find peak in 2D matrix
Pattern:
- binary search on columns
- for the maximum value in a chosen column, vertical neighbors do not matter because it is already >= top and bottom
- only left and right neighbors matter

Break:
- when the column maximum is greater than both left and right neighbors, it is a valid 2D peak

Mid Signal:
- let mat[row][midCol] be the maximum value in the middle column
- compare it with mat[row][midCol - 1] and mat[row][midCol + 1]

Normalization:
- for each chosen middle column, first find the row index of the maximum value in that column

Elimination:
- if current > left neighbor and current > right neighbor
    -> found a peak
- if left neighbor > current
    -> a peak must exist in the left half
    -> discard right half of columns
- if right neighbor > current
    -> a peak must exist in the right half
    -> discard left half of columns

9. #69. Sqrt(x) - Find floor(sqrt(x))
Pattern:
- monotonic predicate: mid^2 > x → [F, F, F, T, T]

Break:
- first index where mid^2 > x

Mid Signal:
- mid^2 > x
- mid^2 <= x

Normalization:
- none

Elimination:
- if mid^2 > x
    - mid is a candidate
    - keep mid
    - right = mid
- if mid^2 <= x
    - mid is valid
    - discard mid
    - left = mid + 1

Answer:
- return left - 1

10. #875. Koko Eating Bananas - Find minimum eating speed to finish within h hours (➕ added)
Pattern:
- monotonic predicate: canFinish(speed) = [F, F, F, T, T, T]
- as speed increases, total hours required decreases → easier to finish within h

Break:
- first speed where totalHours(speed) <= h

Mid Signal:
- totalHours(mid) <= h  → speed is sufficient
- totalHours(mid) > h   → speed is too slow

Normalization:
- none
- search space is [1, max(piles)] — not array indices

Elimination:
- if totalHours(mid) <= h
    -> speed mid qualifies, but we want the minimum
    -> keep mid as candidate
    -> right = mid
- if totalHours(mid) > h
    -> speed mid is too slow
    -> discard mid
    -> left = mid + 1

Answer:
- return left (first speed where canFinish is true)
- Note: this is Binary Search on Answer — search space is the answer range, not array indices
```
