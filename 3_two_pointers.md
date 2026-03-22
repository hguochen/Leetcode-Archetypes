# 🧩 Cheat Sheet - Archetype 3 Two Pointers

>Two pointers = using monotonic pointer movement to eliminate search space in O(n)

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

* symmetry or mirror comparison

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

Used when:

* you are making local optimal decisions
* usually involves smallest vs largest

Examples:

* Boats to save people
* Bag of Tokens
* Container with Most Water

Pattern:

```
if (can_use_smallest) -> use it
else -> target largest
```

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



## Two-Ends Fill-from-Back

Use this pattern when:

* array is sorted
* transformation can break ordering(eg. square, absolute, distance)
* you need the result sorted
* largest values come from either end

Signal:

* the max result comes from either the leftmost or rightmost element

Key Invariant:

```
result[idx+1 ... n-1] = already sorted largest elements
```

Core Idea

* maintain 2 pointers

```
left = 0
right = n - 1
```

* maintain a write pointer

```
// fill from the back
idx = n - 1
```

At each step:

* compare contribution from left vs right
* put the large one at result[idx]
* move that pointer inward
* decrement idx

template:

```
int[] result = new int[n];

int left = 0;
int right = n - 1;
int idx = n - 1;

while (left <= right) {
    int leftVal = transform(nums[left]);
    int rightVal = transform(nums[right]);

    if (leftVal > rightVal) {
        result[idx] = leftVal;
        left++;
    } else {
        result[idx] = rightVal;
        right--;
    }
    idx--;
}
```



## Chunk Processing

Process an array/string in fixed-size blocks, often with different logic per chunk.

Recognition signals:

```
- "for every k elements"
- "for every 2k block"
- "reverse first k, skip next k"
- "process in groups"
- "block-based transformation"
```

Common mistakes:

```
❌ Using cumulative step (e.g., start += 2*k*base)
✅ Always use fixed step: start += chunkSize

❌ Forgetting boundary checks
✅ Always clamp with Math.min(..., n - 1)

❌ Off-by-one errors on end index
✅ end = start + size - 1

❌ Using <= in reverse loop
✅ use while (left < right)
```


template 1: basic chunk iteration

```
for (int start = 0; start < n; start += chunkSize) {
    int end = Math.min(start + chunkSize - 1, n - 1);

    // process range [start, end]
}
```

template 2: process first k in Every 2k (LC541 pattern)

```
for (int start = 0; start < n; start += 2 * k) {
    int left = start;
    int right = Math.min(start + k - 1, n - 1);

    reverse(arr, left, right);
}
```

template 3: chunk conditional logic

```
for (int start = 0; start < n; start += chunkSize) {
    int remaining = n - start;

    if (remaining < k) {
        // case 1: less than k
    } else if (remaining < 2 * k) {
        // case 2: between k and 2k
    } else {
        // case 3: full chunk
    }
}
```

template 4: 2 phase chunk processing

```
for (int start = 0; start < n; start += chunkSize) {
    int mid = Math.min(start + k - 1, n - 1);
    int end = Math.min(start + chunkSize - 1, n - 1);

    // phase 1: process [start, mid]
    // phase 2: process [mid+1, end]
}
```

common helper(reverse)

```
private void reverse(char[] arr, int left, int right) {
    while (left < right) {
        char temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
        left++;
        right--;
    }
}
```


* * *

# 3. Pointer Rules (MOST IMPORTANT)

## Pointer Movements

At every step:

👉 You MUST be able to justify:
"Moving this pointer eliminates a portion of the search space safely"

If you cannot justify pointer movement → DO NOT use two pointers

Examples:


```
Sorted sum:
if (sum < target) → left++ (increase sum)
if (sum > target) → right-- (decrease sum)

Container:
move the smaller height (larger one is useless bottleneck)

Palindrome:
move both when equal, otherwise fail
```

## Loop Condition Patterns

Two common patterns:


### 1. left < right

Used when:

* comparing pairs
* need 2 distinct elements

Examples:

* 2Sum sorted
* container
* palindrome

### 2. left <= right

Used when:

* single element still needs processing
* middle element matters

Examples:

* bag of tokens
* watering plants II

## Dedup Handling

Deduplication. When problem requires unique results:

* skip duplicates AFTER using a value

```
Example (3Sum):

while (left < right && nums[left] == nums[left - 1]) left++;
while (left < right && nums[right] == nums[right + 1]) right--;
```

* skip duplicates BEFORE fixing i

```
if (i > 0 && nums[i] == nums[i - 1]) continue;
```

## Stable vs Unstable Partition

### Unstable (swap-based)

* changes relative order
* O(1) space

Examples:

* sort colors
* move zeros (optimized version)

### Stable

* preserves order
* usually requires extra space

Examples:

* pivot array
* rearrange by sign

* * *

# 4. Complexity

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

# 5. Pitfalls

* If partitioning must preserve relative order, be very suspicious of swap-based two pointers.
* Stable partition usually needs extra space.

