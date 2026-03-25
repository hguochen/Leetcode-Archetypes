# 🧩 Archetype 4 Subarray / Interval Enumeration - Cheat Sheet

Why

*    Extremely common in interviews (prefix sum + Kadane appear everywhere)
*    Converts O(n²) subarray enumeration → O(n) solutions
*    Foundation for many “count subarrays / maximize sum / range queries” problems
*    Frequently disguised as hashmap / modulo / difference problems

* * *

# **🚨 0.** Recognition Signals

## When to use this Archetype:

* number of subarrays
* sum equals k
* continuous subarray
* negative numbers present
* modulo conditions

## When NOT to use this Archetype:

❌ If:

*    strictly positive numbers
*    monotonic condition exists

→ Use Sliding Window instead
* * *

# 1. Core Documentation

## 1.1 Core Insight (this unlocks everything)

Every subarray problem is secretly a pair problem

```
nums[i...j]
↓
prefix[j] - prefix[i]
```

So instead of:

```
enumerating all subarrays
```

You are actually doing:

```
finding pairs (i, j) on prefix array
```

So our mental rewrite rule:

```
Subarray sum = target
↓
prefix[j] - prefix[i] = target
↓
prefix[i] = prefix[j] - target
```

This is literally Two Sum on prefix array.

## 1.2 Mental Model

We are enumerating all subarrays:

```
for i in range(n):
    for j in range(i, n):
        subarray = nums[i...j]
```

Brute force = O(n^2) or O(n^3) (if sum computed repeatedly)


## 1.3 Key Optimization: Prefix Sum

Define:

```
prefix[i] = sum of sums[0...i-1]
```

Then:

```
sum(i -> j) = prefix[j+1] - prefix[i]
```



## 1.4 Core Transformation

Most problems reduce to:

```
prefix[j] - prefix[i] = target
-> prefix[i] = prefix[j] - target
```

So we use:

```
HashMap<prefixSum, frequency>
```



## 1.5 Mental Compression (Memorize this)

```
Subarray Problem
-> prefix sum
-> convert to prefix difference
-> reduce to pair problem
-> hashmap lookup
```

# 2. 3 Problem Classes

Every subarray/interval enumeration question falls into ONE of these:

## **🟢 2.**1. Counting Subarrays

Question asks:

* “number of subarrays”
* “how many ways“

Pattern:

```
count += map.get(prefix - target)
```


Example:

* #560 Subarray Sum Equals K

## **🔵 2.**2. Max / Min Subarray  Optimization

Question asks:

* “maximum subarray”
* “longest subarray”
* “best score”

Pattern: 

Kadane’s algorithm OR prefix + index matching

```
current = max(num, current + num)
global = max(global, current)
```

Example:

* #53 Maximum Subarray

## **🟣 2**.3. Modulo / Remainder

Question asks:

* “divisible by k”
* “mod k”
* “multiple of k”

Pattern:

```
prefix[j] % k == prefix[i] % k
```

Used when:

```
sum % k == target
```

Key Idea:

```
(prefix[j] - prefix[i]) % k == 0
→ prefix[j] % k == prefix[i] % k
```

#### 

Example:

* #974 Subarray Sums Divisible by K

* * *

# **🧠 3.** Core Patterns

## **⭐** 2.1 Prefix Sum + HashMap

```
int count = 0;
int prefix = 0;
Map<Integer, Integer> map = new HashMap<>();
map.put(0, 1);

for (int num : nums) {
    prefix += num;
    
    count += map.getOrDefault(prefix - k, 0);
    
    map.put(prefix, map.getOrDefault(prefix, 0) + 1);
}
```

## **⭐** 2.2 Max Length Template / Kadane

```
Map<Integer, Integer> map = new HashMap<>();
map.put(0, 1);

int prefix = 0;
int maxLen = 0;

for (int i = 0; i < nums.length; i++) {
    prefix != nums[i];
    
    if (map.containsKey(prefix - k)) {
        maxLen = Math.max(maxLen, i - map.get(prefix - k));
    }
    map.putIfAbsent(prefix, j);
}
```

## **⭐** 2.3 Modulo Pattern

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

* * *

# 3. The 5 Recognition Signals (Train this reflex)

If you see ANY of these → this archetype


## **🚨** 3.1 Signal 1 - “subarray”

→ immediate prefix sum


## **🚨** 3.2 Signal 2 - “sum equals k”

→ prefix + hashmap


## **🚨** 3.3 Signal 3 - negative numbers

→ ❌ sliding window
→ ✅ prefix sum


## **🚨** 3.4 Signal 4 - “divisible by k”

→ modulo trick


## **🚨** 3.5 Signal 5 - “maximum subarray”

→ Kadane

* * *


# 4. The 3 Most Common Mistakes

## **❌** 4.1 Mistake 1 - Forgetting this

```
map.put(0, 1);
```

This handles subarrays starting at index 0


## **❌** 4.2 Mistake 2 - Wrong modulo handling

```
prefix = (prefix % k + k) % k;
```

+ k is to prevent negative numbers as a result of modulo negative numbers



## **❌** 4.3 Mistake 3 - Mixing index vs frequency

|Problem Type	|Map stores	|
|---	|---	|
|Counting	|frequency	|
|Max length	|index	|

## **❌** 4.4 Mistake 4 - Wrong subarray length calculation when using Prefix sum

Remember that 

```
prefix[j+1] - prefix[i] = k
prefix[i] = prefix[j+1] - k
```

When we want to find the length of a subarray involving prefix sums, it’s actually:

```
j - i
```

and NOT

```
j - i + 1
```

Because `prefix[i]` means sum of nums[0...i-1]. when we traverse the start to end, we are actually at:

```
start = i + 1
end = j
```

So calculating length is:

```
j - (i + 1) + 1
-> j - i
```

