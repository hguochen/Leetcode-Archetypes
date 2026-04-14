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

## Anti-signals (when NOT to use)

* No monotonicity or ordering
* Need all combinations
* Need global aggregation over all elements
* Unsorted array where comparing to mid does NOT eliminate half

* * *


# 2. Core Mental Model

## Core Ideas

* Binary search is not “search sorted array”
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
            {
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
            left = mid + 1;
        } else { // peak on the left
            right = mid;
        }
    }
    return left;
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
        int mid = left + (right - most) / 2;
        
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

>“What ordered search space contains the answer, and what mid-test lets me eliminate half?”



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

* * *

# **11. Cheat Sheet (1-Pager)**

```
WHEN TO USE:
- 

CORE IDEA:
- 

INVARIANT:
- 

TEMPLATE:
- 

TIME/SPACE:
- 

TOP 3 TRICKS:
1.
2.
3.

TOP 3 PITFALLS:
1.
2.
3.
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

Before you start coding, write out the following details about the problem. If you can’t find a reasonable answer to either of these specifications, reconsider using binary search as an approach.


```
1. search space
2. invariant
3. shrink rule
4. return meaning
5. problem family
```

## 1.1 Predicate

Take a problem statement and rewrite it as:

```
What am i searching for?
What condition becomes true?
What is the search space?
```

Examples:

```
Search Insert Position
-> first index where nums[i] >= target

First Bad Version
-> first version where isBad(version) == true

Find first occurrence
-> first index where nums[i] >= target, then validate equality
```

## 1.2 Invariant

For any binary search problem, say:

```
The answer is always inside ______
```


Examples:

```
[left, right]
[left, right)
```

If you cannot state the invariant, you are coding too early.


## 1.3 Shrink rule

For each problem, force this question

```
Why is this half impossible?
```

NOT:

```
Which half feels right?
```

Binary search is proof, not intuition.


## 1.4 Return meaning

At the end of the loop, ask

```
What does left mean now?
```

Examples:

```
left = insertion position
left = first true index
left = peak index
left - 1 = rightmost value <= target
```

This single habit prevents many bugs.


## 1.5 Problem Family

Which problem family does the solution belong to?

```
- Exact Match
- Boundary Search
- Structural Search
```

```

