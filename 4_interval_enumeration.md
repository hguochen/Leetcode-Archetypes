# 🧩 Cheat Sheet - Archetype 4 Subarray / Interval Enumeration

Why

*    Extremely common in interviews (prefix sum + Kadane appear everywhere)
*    Converts O(n²) subarray enumeration → O(n) solutions
*    Foundation for many “count subarrays / maximize sum / range queries” problems
*    Frequently disguised as hashmap / modulo / difference problems

* * *

# **🚨 0.** Recognition Signals

* number of subarrays
* sum equals k
* continuous subarray
* negative numbers present
* modulo conditions

* * *

# 1. Core Documentation

## Mental Model

We are enumerating all subarrays:

```
for i in range(n):
    for j in range(i, n):
        subarray = nums[i...j]
```

Brute force = O(n^2) or O(n^3) (if sum computed repeatedly)


## Key Optimization: Prefix Sum

Define:

```
prefix[i] = sum of sums[0...i-1]
```

Then:

```
sum(i -> j) = prefix[j+1] - prefix[i]
```



## Core Transformation

Most problems reduce to:

```
prefix[j] - prefix[i] = target
-> prefix[i] = prefix[j] - target
```

So we use:

```
HashMap<prefixSum, frequency>
```



## Two Core problem Types

### 1. Counting Subarrays

```
count += map.get(prefix - target)
```

### 2. Maximum / Minimum Subarray

Kadane’s algorithm

```
current = max(num, current + num)
global = max(global, current)
```

### Special Pattern: Modulo Trick

Used when:

```
sum % k == target
```

Key Idea:

```
(prefix[j] - prefix[i]) % k == 0
→ prefix[j] % k == prefix[i] % k
```

* * *

# **🧠 2.** Core Patterns

## 2.1 Prefix Sum + HashMap

```
int count = 0;
int prefix = 0;
Map<Integer, Integer> map = new HashMap<>();
map.put(0, 1);

for (int num : nums) {
    prefix += num;
    count += map.getOrDefault(prefix - k, 0;
    map.put(prefix, map.getOrDefault(prefix, 0) + 1);
}
```

## 2.2 Kadane

```
int current = nums[0];
int max = nums[0];

for (int i = 1; i < nums.length; i++) {
    current = Math.max(nums[i], current + nums[i]);
    max = Math.max(max, current);
}
```

## 2.3 Modulo Pattern

```
int prefix = 0;
Map<Integer, Integer> map = new HashMap<>();
map.put(0, 1);

for (int num : nums) {
    prefix = (prefix + sum) % k;
    if (prefix < 0) prefix += k;
    
    count += map.getOrDefault(prefix, 0);
    map.put(prefix, map.getOrDefault(prefix, 0) + 1);
}
```

