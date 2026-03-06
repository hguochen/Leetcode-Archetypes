# Cheat Sheet - Archetype 1 Pair Enumeration

# 1. Pattern Summary

## Core idea

>Iterate over all possible pairs(i, j) and check whether they satisfy a condition.


This is the fundamental brute force pattern before any optimization.

Typical complexity:

```
Time: O(n^2)
Space: O(1)
- where n is the size of input array
```

Upper bound guideline(Java):


|n	|n^2	|
|---	|---	|
|1000	|1,000,000	|
|2000	|4,000,000	|
|5000	|25,000,000	|

Acceptable in interviews if n <= 2000
* * *

# 2. Recognition Signals

Look for problem containing:


## Keywords

* “number of pairs“
* “count pairs”
* “two indices”
* “i < j”
* “absolute difference”
* “distance between elements”

## Input characteristics

```
n <= 2000
```

This is a big signal brute force may be acceptable.


## Structural hints

Problems that asks:

```
For every pair(i, j)
```

OR

```
Find two numbers satisfying condition
```

* * *

# 3. Core Template

## Template

```
Map<Integer, Integer> freq = new HashMap<>();
int result = 0;

for (int x : nums) {

    int needed = ...;   // value required to form pair

    result += freq.getOrDefault(needed, 0);

    freq.put(x, freq.getOrDefault(x, 0) + 1);
}

return result;
```

## Basic pair enumeration

```
for (int i = 0; i < n; i++) {
    for (int j = i + 1; j < n; j++) {

        if (condition(nums[i], nums[j])) {
            result++;
        }

    }
}
```

Key idea:

```
j starts from i + 1
```

Prevents duplicate pairs
* * *

# 4. Variations of Pair Enumeration

## Variation 1 - Count pairs

Example:

* Count pairs with difference k

```
if (Math.abs(nums[i] - nums[j]) == k)
```

* * *

## Variation 2 - Track best pair

Example:

* max difference
* max sum
* best score

```
best = Math.max(best, nums[i] + nums[j]);
```

* * *

## Variation 3 - Pair existence

Example:

* Two Sum(brute force)

```
if (nums[i] + nums[j] == target)
    return true;
```

* * *

## Variation 4 - Pair property

Example:

* same value
* gcd property
* divisible property

* * *

# 5. When This Pattern Appears

This archetype often appears when solving:

## Small input size

```
n <= 2000
```

* * *

## Interview trick

Sometimes the brute force solution is acceptable
But the interviewer wants to see if you can improve it to:

```
O(n logn)
OR
O(n)
```

* * *

# 6. Common Optimizations

Pair enumerations often leads to three optimizations.
* * *

## Optimization 1 - Hash Map

Convert

```
pair search
```

into

```
complement lookup
```

Example:

```
nums[i] + nums[j] = target
```

becomes

```
// lookup value
target - nums[i]
```

Example:

### Two Sum

```
time: O(n^2) -> O(n)
```

* * *

## Optimization 2 - Frequency Array

Example:

```
|nums[i] - nums[j]| = k
```

Use:

```
freq[x + k]
freq[x - k]
```

* * *

## Optimization 3 - Sorting + Two Pointers

Example:

```
find pair sum = target
```

Use:

```
sort + two pointers
```

* * *

# 7. Example Problems

## Essence Problems

These show the pure archetype


|Problem	|Concept	|
|---	|---	|
|LC1512	|Count equal pairs	|
|LC 2006	|Count pairs with difference k	|

## Variation Problems

|Problem	|Concept	|
|---	|---	|
|LC 1	|Two Sum	|
|LC 167	|Two Sum II	|
|LC 1099	|Two Sum Less Than K	|
|LC 1010	|Pairs divisible by 60	|
|LC 532	|K-diff pairs	|

* * *

# 8. Mental Model

Think of this as

```
Pair Graph
```

For an array:

```
[ a, b, c, d ]
```

Pairs are:

```
(a,b)
(a,c)
(a,d)
(b,c)
(b,d)
(c,d)
```

Total:

```
n(n-1)/2
```

* * *

# 9. Interview Thinking Process

When you see a problem?

## Step 1

```
Is this a pair problem?
```

## Step 2

```
n <= 2000 ?
```

## Step 3

```
Try brute force pair enumeration first
```

## Step 4

Ask yourself:

```
Can we reduce pair search to lookup?
```

Possible upgrades:

```
hash map
two pointers
frequency array
```

* * *

# 11 Typical Pitfalls

## Duplicate counting

Wrong:

```
for i
    for j
```

Correct:

```
for i
    for j = i + 1
```

* * *

## Negative values

If using frequency array:

```
value range must be small
```

* * *

## Overflow

Sometimes:

```
nums[i] + nums[j]
```

needs `long` to compute results
* * *

# 12. Implementation Variants

## Count pairs

```
result += 1
```

* * *

## Store pairs

```
List<int[]>
```

* * *

## Return first match

```
return new int[]{i, j};
```

* * *

# 13. Mastery signal

You know this archetype when you can instantly recognize these problems:

* Two Sum
* Pairs with difference
* Pairs divisible by k
* Pairs with equal values


