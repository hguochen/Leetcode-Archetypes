# 🧩 Archetype 4 Subarray / Interval Enumeration - Cheat Sheet

Why

*    Extremely common in interviews (prefix sum + Kadane appear everywhere)
*    Converts O(n²) subarray enumeration → O(n) solutions
*    Foundation for many “count subarrays / maximize sum / range queries” problems
*    Frequently disguised as hashmap / modulo / difference problems

* * *

# **🚨 0.** Recognition Signals

## When to use this Archetype:

* subarray + sum / range accumulation
* sum equals k
* count subarrays
* modulo / divisible by k
* negative numbers break sliding window
* maximize / minimize subarray sum
* many range-sum queries

## When NOT to default to this archetype:

* all numbers positive AND constraint is monotonic over a moving window

  → consider Sliding Window

* fixed-size window

  → consider Fixed Sliding Window

* optimize max/min subarray sum directly

  → consider Kadane

* many range updates

  → consider Difference Array

## Subarray / Interval Decision Gate

```
1. Is it contiguous?
   - no → not this archetype
   - yes → continue

2. Is it a sum / range accumulation problem?
   - yes → continue
   - no → maybe other archetype

3. Are all values positive and condition monotonic?
   - yes → Sliding Window
   - no → continue

4. Is it asking for:
   - count / exact sum / modulo / longest-shortest with arbitrary integers?
     → Prefix Sum + HashMap
   - max/min subarray sum?
     → Kadane
   - many offline range queries?
     → Prefix Query
   - sum over all subarrays?
     → Contribution Counting
```

* * *

# 1. Core Documentation

```
This archetype has 4 internal engines:
- prefix + hashmap
- Kadane
- prefix query
- contribution counting
```

## 1.1 Core Insight (this unlocks everything)

Many subarray sum problems can be rewritten as pair problems on the prefix array

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
prefix[i] = sum of nums[0...i-1]
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

# 2. Problem Classes

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
→ (prefix[j] % k - prefix[i] % k) % k == 0
→ prefix[j] % k == prefix[i] % k
```

#### 

Example:

* #974 Subarray Sums Divisible by K

## **🟢** 2.4. Subarray sums

Question asks:

* sum of all subarrays
* sum of all odd/even subarrays
* sum over every interval

Ask:

```
Can i count how many times each element contributes?
```

Do this drill without jumping to the formula

Use:

```
arr = [1, 4, 2, 5, 3]
```

Focus only on:

```
index i = 1   value = 4
```

We need to derive how many odd-length subarrays include arr[i]

Follow these exact steps:

```
1. List all possible start indices for subarrays containing index 1
2. List all possible end indices for subarrays containing index 1
3. Count total number of subarrays containing index 1
4. Explicitly list all those subarrays
5. From that list, keep only the odd-length ones. (or what the question specifies as qualifying subarray)
6. Count them
7. Multiply that count by arr[1] to get the total contribution of value 4

```


Example:

* [#1588 Sum of all Odd Length Subarrays](https://leetcode.com/problems/sum-of-all-odd-length-subarrays/description/)

* * *

# **🧠 3.** Core Patterns

## **⭐** 2.1 Prefix Sum + HashMap (Count)

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

## **⭐** 2.2 Max Length Template (Max Length)

```
Map<Integer, Integer> map = new HashMap<>();
map.put(0, -1);

int prefix = 0;
int maxLen = 0;

for (int i = 0; i < nums.length; i++) {
    prefix += nums[i];
    
    if (map.containsKey(prefix - k)) {
        maxLen = Math.max(maxLen, i - map.get(prefix - k));
    }
    map.putIfAbsent(prefix, i);
}
```

## **⭐**2.2.1 Kadane

Maximum Subarray Variant

```
int current = nums[0];
int best = nums[0];

for (int i = 1; i < nums.length; i++) {
    current = Math.max(nums[i], current + nums[i]);
    best = Math.max(best, current);
}
```

Minimum Subarray Variant

```
int current = nums[0];
int best = nums[0];

for (int i = 1; i < nums.length; i++) {
    current = Math.min(nums[i], current + nums[i]);
    best = Math.min(best, current);
}
```

## **⭐** 2.3 Modulo Pattern

```
int prefix = 0;
Map<Integer, Integer> map = new HashMap<>();
map.put(0, 1);

for (int num : nums) {
    prefix = (prefix + num) % k;
    if (prefix < 0) prefix += k;
    
    count += map.getOrDefault(prefix, 0);
    map.put(prefix, map.getOrDefault(prefix, 0) + 1);
}
```

* * *

# 3. The 5 Recognition Signals (Train this reflex)

If you see ANY of these → this archetype


## **🚨** 3.1 Signal 1 - “subarray + sum/range accumulation”

→ consider prefix sum / Kadane / sliding window


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

* * *

# 5. PrefixSum + Hashmap solving framework

For subarray + prefix sum problems, force yourself to go through this exact sequence
We will use #1590 https://leetcode.com/problems/make-sum-divisible-by-p/description/ as illustration.

## Step 1: Write the brute-force condition as an equation

Not english. Equation.

```
(total - subArraySum) % p == 0
```

Then reduce it:

```
subArraySum % p == total % p
```

Let:

```
target = total % p
```

So:

```
subArraySum % p == target
```

## Step 2: Expand subarray sum using prefix sums

```
subArraySum(i..j) = prefix[j+1] - prefix[i]
```

Plug it in:

```
(prefix[j+1] - prefix[i]) % p == target
```



## Step 3: Rearrange until one side is “past thing”, the other is “current thing”

You want a lookup problem of the form:

```
past = function(current)
```

So rearrange:

```
prefix[i] % p = (prefix[j+1] % p - target + p) % p
```

This is the key step.

Now it is obvious:

* current thing = prefix[j+1] % p
* past thing to look up = needed prefix mod

So hashmap key must be:

```
prefixMod
```

Not subarray sum. Not removed sum. not target.


## Step 4: Ask: what is the question asking me to optimize?

This determines hashmap value.

Ask one of these:

```
- count how many?
- does one exist?
- longest length?
- shortest length?
```

For each:


### Count

Store frequency.

```
map[key] = count
```

### Existence

Store any index,, often earliest.


### Longest length

Store earliest index.

```
currentIndex - earliestIndex = largest
```

### Shortest length

Store latest index.

```
currentIndex - latestIndex = smallest
```



## The subarray hashmap cheat rule

Memorize this:

```
Hashmap key = the “past quantity” needed by the rearranged equation
Hashmap value = whatever helps optimize the objective
```

* * *

# 6. Solved problems canonical mappings

|Problem type	|Rearranged lookup	|Map Key	|Map value	|
|---	|---	|---	|---	|
|
Sum == k count	|
prefix[i] = current - k	|
prefix sum	|
frequency	|
|
Sum == k longest	|
prefix[i] = current - k	|
prefix sum	|
earliest index	|
|
Sum divisible by k count	|
prefix[i] % k = current % k	|
prefix mod	|
frequency	|
|
Sum divisible by k exists	|
same remainder	|
prefix mod	|
earliest index	|
|
Remove shortest subarray with mod target	|
prefix[i] % p = needed	|
prefix mod	|
latest index	|

* * *

# 7. Decision Template for Prefix Sum 

```
1. Write brute-force condition
2. Convert subarray to prefix difference
3. Rearrange into past = function(current)
4. Hashmap key = the past quantity
5. Hashmap value = determined by objective
   - count -> frequency
   - exists -> useful index
   - longest -> earliest index
   - shortest -> latest index
```

