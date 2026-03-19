# 🧩 Cheat Sheet - Archetype 3 Two Pointers

# 0. Should you use Two Pointers?

## **❌ When NOT to Use Two Pointers**

### 1. Unsorted + No Ordering Property

* no monotonic(consistently moving in 1 direction) behavior
* moving pointers gives no guarantee

Bad fit

* Two Sum(unsorted) → use hashmap instead

Red flag:

* moving pointer doesn’t eliminate search space

### 2. Requires All Combinations (not prunable)

* not prunable means we cannot remove something without harming the overall system
* need exhaustive enumeration

Bad fit

* Subsets/ permutations

Use:

* Backtracking / DFS

### 3. Non-contiguous Relationships

* Problem depends on scattered indices

Bad fit

* Longest subsequence(not substring)

Use:

* DP/Hashing

### 4. No monotonic condition

* Cannot decide pointer movement direction

example:

```
if sum < target → ??? move left or right?
```

if unclear → do not use two pointers


### 5. Sliding Window Instead

if problem involves:

* subarray/substring
* continuous region
* dynamic expansion/shrinking

use:

* sliding window(not static two pointers)

### 6. Graph / Tree Problems

* Structure is not linear

Use:

* BFS/DFS

## **✅ When to Use Two Pointers**

### 1. Sorted Input(or can be sorted)

Strongest signal

* array is already sorted OR sorting doesn’t break the problem
* you need to:
    * find pairs/triplets
    * compare extremes
    * shrink search space

Examples

* Two Sum II
* 3Sum / 4Sum
* Container with most water

Mental model:

* i can eliminate part of the search space deterministically.

### 2. Opposite-End Convergence

* Start from both ends and move inward
* Usually optimizing something (max/min)

Examples

* Container with most water
* Valid Palindrome

Signal:

* Decision at left/right lets me discard one side safely

### 3. Partitioning/In-place rearrangement

* Reorganizing array into regions

```
[ valid | invalid | unknown ]
```

Examples

* move zeros
* remove element
* sort colors

Signal:

* i’m rewriting the array in-place with constraints

### 4. Fast & Slow Pointer (Compression / Dedup)

* Build result in-place
* Maintain processed prefix

Examples

* Remove Duplicates(I & II)
* Remove Element

Signal:

* Ouput overwrites input progressively

### 5. Fix + Two Pointers (k-sum pattern)

* Fix 1 element → reduce to 2-pointer problem

Examples:

* 3Sum
* 3Sum Closest
* 4Sum

Signal:

* Nested reduction _> 2Sum with pointers

### 6. Comparison / Matching problems

* compare characters or elements from both ends

Examples

* valid palindrome
* backspace string compare

Signal

* symmetry or mirrore comparison

* * *

# 1. Decision Framework

Ask these in order:


### Step 1: is input sorted OR can i sort it?

→ YES → consider two pointers

### Step 2: can i eliminate search space based on comparison?

→ YES → strong candidate


### Step 3: is this about contiguous range?

→ YES → use sliding window instead


### Step 4: is this about in-place transformation?

→ YES → fast/slow pointers
* * *

# 2. Core Patterns

## Opposite Ends

Used for:

* pair sum
* palindrome
* greedy comparisons

template:

```
int left = 0;
int right = nums.length - 1;

while (left < right) {
    if (condition) {
        left++;
    } else {
        right++;
    }
}
```

## 

## Fast/Slow Pointer

Used for:

* filtering
* deduplication
* in-place compaction

template:

```
int slow = 0;

for(int fast = 0; fast < nums.length; fast++) {
    if (valid(nums[fast]) {
        nums[slow] = nums[fast];
        slow++;
    }
}
```



## Deduplication in Sorted Array

```
int slow = 1; 

for (int fast = 1; fast < nums.length; fast++) {
    if (nums[fast] != nums[fast - 1]) {
        nums[slow] = nums[fast];
        slow++;
    }
}
```



## Multi-Pointer Expansion (3 Sum Style)

```
for (int i = 0; i < nums.length; i++) {
    int left = i + 1;
    int right = nums.length - 1;
    
    while (left < right) {
        int sum = nums[i] + nums[left] + nums[right];
        
        if (sum == target) {
            result.add(...);
            left++;
            right--;
        }
        else if (sum < target) left++;
        else right--;
    }
}
```

## 

## Greedy Two Pointers

example: Container with most water

rule:

```
move the limiting side
```

example logic:

```
if (height[left] < height[right]) 
    left++;
else
    right--;
```

* * *

# 3. Complexity

two pointers works because each pointer moves monotonically

```
time: O(n)
space: O(1)
```

each pointer traverses the array at most once

this archetype is one of the highest frequency interview patterns and combines heavily with:

* sorting
* greedy
* prefix sum
* binary search


