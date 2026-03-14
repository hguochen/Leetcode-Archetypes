# Cheat Sheet - Archetype 2 Sliding Window

# 0. Cheat Sheet

Sliding Window Pattern Cheat Sheet

```
// longest valid window
for (right...) {
    add(right);
    while (invalid) remove(left++);
    updateMax();
}

// shortest valid window
for (right...) {
    add(right);
    while (valid) {
        updateMin();
        remove(left++);
    }
}

// count valid windows
for (right...) {
    add(right);
    while (invalid) remove(left++);
    count += right - left + 1;
}

// fixed-size window
for (right...) {
    add(right);
    if (window too big) remove(old);
    if (window size reached) check();
}
```

# 1. The Core Idea

Sliding window is an optimization for contiguous ranges.
Instead of checking all subarrays:

```
O(n^2)
```

we maintain a moving window:

```
[left, right]
```

**we expand and shrink the window dynamically** while maintaining a condition.
Core loop:

```
for right in array:
    add element to window
    
    while window invalid:
        remove left element
        left++
    
    update answer
```

Key principle:

>Every element enters the window once and leaves once.

so complexity becomes:

```
O(n)
```

in general, sliding window problems consists of:

* an expanding/shrinking window of elements
* a data structure to maintain state
* 1 or many conditions that must be satisfied

* * *

# 2. When Sliding Window Applies

Sliding window **ONLY works when the data must be contiguous.**

Typical signals in problems include words like:

* substring
* subarray
* contiguous
* longest
* shortest
* at most k
* at least k
* minimum window

Example prompts:

```
Longest substring without repeating characters
```

```
Minimum subarray length with sum >= target
```

```
Longest substring with at most K distinct characters
```

* * *

# 3. The Window Model

Visualize:

```
s = a b c a b b c
    L.    R
```

Window = [L, R]
We move:

* R → expand window
* L → shrink window

State is tracked using:

* frequency map
* count
* sum
* distinct element count

* * *

# 4. The 8 Sliding Window Patterns

you will see **8 major patterns**. think of these as the taxonomy of sliding window problems

```
Sliding Window
│
├─ 1. Longest Valid Window
├─ 2. Minimum Valid Window
├─ 3. Fixed Size Window
├─ 4. At Most K Constraint
├─ 5. Exactly K Constraint
├─ 6. Counting Subarrays
├─ 7. Window with Replacement / Budget
└─ 8. Permutation / Anagram Matching
```

|PATTERN	|SIGNAL WORDS	|
|---	|---	|
|Longest Window	|longest / max length	|
|Minimum Window	|smallest / minimum	|
|Fixed Size	|window size K	|
|At Most K	|<= K distinct / flips	|
|Exactly K	|exactly K	|
|Counting	|counting subarrays	|
|Budget	|replace / flip <= K	|
|Permutation	|anagram / permutation	|

Almost every sliding window problem becomes easy once you define:

    *    what the window means
    *    what state you maintain
    *    what makes the window invalid
    *    when to update the answer
    *    whether the problem wants:
    *    max length
    *    min length
    *    count
    *    exact count via atMost difference


When you see a new problem, ask:

1. **Is the range contiguous?**

if yes, sliding window may apply


1. **Is it asking for longest/shortest/count?**

That tells you which template to use.


1. **Is the condition:**

* at most K
* at least K
* exactly K
* fixed size

That tells you the invariant.


1. **Is it asking for the longest length?**

Use:

* longest valid window

1. **Is it asking for the shortest valid substring?**

Use:

* minimum valid window

1. **Is it asking for a count of valid subarrays?**

Use:

* counting windows

1. **Is it asking for exactly k?**

Use:

* atMost(k) - atMost(k - 1)

1. **Is the window size fixed?**

Use:

* fixed size window

1. **Is it about removing from ends?**

Try:

* transform into longest kept middle subarray

* * *

## Type 1 - Longest Valid Window

goal:

```
maximize window length
```

strategy:

* expand right
* if invalid → shrink left

example:

```
Longest substring without repeating characters
```

flow:

```
expand
expand
expand
invalid
shrink
expand
expand
```

code template:

```
int left = 0;
int answer = 0;

for(int right = 0; right < n; right++) {
    add(nums[right]);
    
    while(windowInvalid() {
        remove(nums[left]);
        left++;
    }
    answer = Math.max(answer, right - left + 1);
}
```

recognition phases:

* longest substring/subarray

typical problems:

*    LC 3 Longest Substring Without Repeating Characters
*    LC 424 Longest Repeating Character Replacement
*    LC 1004 Max Consecutive Ones III
*    LC 487 Max Consecutive Ones II
*    LC 1493 Longest Subarray of 1s After Deleting One Element
*    LC 340 At Most K Distinct
*    LC 159 At Most Two Distinct
*    LC 1208 Get Equal Substrings Within Budget
*    LC 1838 Frequency of the Most Frequent Element
*    LC 2024 Maximize the Confusion of an Exam

core invariant examples:

* zeroCount <= k
* distinctCount <= k
* totalCost <= maxCost
* min(countT, countF) <= k

* * *

## Type 2 - Minimum Valid Window

goal:

```
find smallest window satisfying condition
```

strategy:

* expand until valid
* then shrink aggressively

example:

```
Minimum Window substring
```

flow:

```
expand
expand
expand
valid
shrink
shrink
invalid
expand
```

code template:

```
int left = 0;

for (int right = 0; right < n; right++) {
    add(nums[right]);
    
    while (windowValid()) {
        answer = min(answer, right - left + 1);
        remove(nums[left]);
        left++;
    }
}
```

typical questions:

* #209 Minimum Size Subarray Sum
* #76 Minimum Window Substring

core invariant examples:

* missing == 0
* “window contains all required counts”

* * *

## Type 3 - Fixed Size Window

window size never changes.

```
window size = K
```

strategy:

```
expand
when size > k -> shrink
```

code template:

```
int left = 0;

for(int right = 0; right < n; right++) {
    add(nums[right]);
    
    if (right - left + 1 > k) {
        remove(nums[left]);
        left++;
    }
    if (right - left + 1 == k) {
        updateAnswer();
    }
}
```

recognition signals:

* permutation/anagram in string
* fixed size comparison
* every substring of length k

typical problems:

* LC567 Permutation in String
* LC438 Find all anagrams in a string(almost fixed)

* * *

## Type 4 - At Most K Constraint(EXTREMELY IMPORTANT)

this is the **most powerful sliding window trick.**

goal:

```
window must contain <= K something
```

examples:

```
<= K distinct characters
<= K zeros
<= K replacements
```

code template:

```
int left = 0;

for(int right = 0; right < n; right++) {
    add(nums[right]);
    
    while(constraint > K) {
        remove(nums[left]);
        left++;
    }
    updateAnswer();
}
```

recognition signals:

* at most K
* can flip K elements
* can replace K characters

typical problems:

*    #424 Longest Repeating Character Replacement
*    #1004 Max Consecutive Ones III
*    #340 Longest Substring with K Distinct Characters

* * *

## Type 5 - Exactly K Constraint

sliding window **cannot directly maintain exactly K**.

instead use the trick

```
exactly(K) = atMost(K) - atMost(K-1)
```

this is a very common interview trick.

example:

```
subarrays with exactly K distinct numbers
```

solution:

```
countAtMost(K) - countAtMost(K-1)
```

code template:

```
public int solution(nums, goal) {
    return atMostGoal(nums, goal) - atMostGoal(nums, goal - 1)
}

public int atMostGoal(nums, goal) {
    int left = 0;
    
    for(int right = 0; right < nums.length; right++) {
        addRight();
        
        while(invariantInvalid()) {
            removeLeft();
            left++;
        }
        updateAnswer();
    }
    return answer;
}
```

recognition phrases:

* exactly k distinct
* exactly k odds
* exactly sum goal in binary array
* exact all three chars, when alphabet is tiny

typical problems:

*    #992 Subarrays with K Different Integers
*    #1248 Count Number of Nice Subarrays
*    #930 Binary Subarrays with Sum
*    #1358 Number of Substrings Containing All Three Characters

* * *

## Type 6 - Counting Valid Windows

Instead of longest/minimum, we count valid subarrays.
use when the problem asks for the number of subarrays/substrings satisfying an at-most condition

key insight:
if [L, R] is valid, then

```
// all windows ending at R and starting from L..R are valid
[L, R]
[L+1, R]
[L+2, R]
...
[R, R]
```

why this works mathematically?
Once the largest window [left, right] satisfies invariant:

```
product < k
```

then any smaller suffix of this window also satisfies it. Because removing any elements can only reduce the product.

Essentially what we are saying is:

>For every right, we count all valid subarrays that END at right. Thus, summing across all right gives the total number of valid subarrays.


so we add:

```
count += right - left + 1
```

example:

```
Subarray product < K
```

code template:

```
int left = 0;

for (int right = 0; right < n; right++){
    add(nums[right]);
    
    while(invalid) {
        remove(nums[left]);
        left++;
    }
    count += right - left + 1;
}
```

recognition phrases:

* count subarray/substrings
* at most k
* product/sum/distinct odd count under a limit

typical problems:

*    #713 Subarray Product Less Than K
*    #930 Binary Subarrays With Sum
*    #1248 Nice Subarrays

* * *

## Type 7 - Window with Replacement / Budget

these track how much change you’re allowed to make.

example:

```
replace <= k characters
flip <= k zeros
cost <= budget
```

key variable:

```
usedBudget
```

window invariant:

```
windowSize - maxCharFrequency <= k
```

typical problems:

*    #1004 Max Consecutive Ones III
*    #424 Longest repeating Character Replacement
*    #1208 Equal Substrings Within Budget

* * *

## Type 8 - Permutation / Anagram Matching

goal:

```
check if a substring is permutation of another string
```

key idea:

```
frequency matching
```

usually window size = pattern length

code template:

```
int[] count = new int[26];

for(char c : pattern)
    count[c - 'a']++;
    
for(int right = 0; right < s.length(); right++;) {
    count[s.charAt(right) - 'a']--;

    if (right >= k)
        count[s.charAt(right - k) - 'a']++;
        
    if (allZero(count))
        foundMatch();
}
```

typical problems:

*    #567 Permutation in String
*    #438 Find All Anagrams in a String

* * *

# 5. Window State Tracking

you typically track one of these:

1️⃣ Frequency Map

used for strings

```
HashMap<Character, Integer>
```

or

```
// each index represents a character
// int[0] = frequency of a
// int[1] = frequency of b
// ...
int[26]
```

example:

```
Longest substring without repeating characters
```

* * *
2️⃣ Distinct Count

track how many unique elements

example:

```
At most K distinct characters
```

* * *
3️⃣ Running Sum

example:

```
minimum subarray sum >= target
```

* * *
4️⃣ Number of Violations

example:

```
Max consecutive ones with k flips
```

track:

```
zeros count
```

* * *

# 6. The Famous At-Most-K Trick

very important pattern.

many problems reduce to:

```
count(atMost(K)) - count(atMost(K-1))
```

used in:

```
Subarrays with K distinct integers
```

```
Nice subarrays
```

```
Binary subarrays with sum
```

this trick appears in many hard interview questions
* * *

# 7. Why Negative Numbers Break Sliding Window

sliding window relies on monotonicity.

>monotonicity: refers to a function or sequence that moves in a consistent direction - either never decreasing or never increasing. it is a fundamental concept used to analyze trends, optimize algorithms and ensure the stability of systems.


example:

```
sum increases as window grows
```

if numbers can be negative:

```
sub may decrease
```

then the window assumption breaks.
example:

```
subarray sum = k
```

in this case, we need prefix sum + hash map, NOT sliding window.
* * *

# 8. Sliding Window Recognition Cheat Sheet

if the problem says:

```
substring
subarray
contiguous
longest
shortest
at most k
minimum window
```

→ think sliding window immediately
* * *

# 9. The Master Template

memorize this skeleton

```
int left = 0;

for(int right = 0; right < nums.length; right++) {
    // 1. expand window
    add(nums[right]);
    
    // 2. shrink until valid
    while(windowInvalid()) {
        remove(nums[left]);
        left++;
    }
    
    // 3. update result
    updateAnswer();
}
```

this template solves 80% of sliding window problems
* * *

# 10. Mental Model of Interviews

always explain it like this:

## Step 1 - Brute Force

```
enumerate all subarrays
O(n^2)
```

## Step 2 - Observation

```
subarrays are contiguous
overlap heavily
```

## Step 3 - Optimization

```
maintain dynamic window
```

## Step 4 - Sliding Window

```
each element enters and leaves once
O(n)
```

interviewers love hearing this reasoning
* * *

# 11. The First Problems You Should Do

exactly in this order(best learning curve):

1️⃣ #3 Longest Substring Without Repeating Characters
2️⃣ #209 Minimum Size Subarray Sum
3️⃣ #424 Longest Repeating Character Replacement
4️⃣ #1004 Max Consecutive Ones III
5️⃣ #567 Permutation in String
6️⃣ #438 Find All Anagrams in a String
7️⃣ #76 Minimum Window Substring

these gradually teach:

```
basic window
sum windows
frequency windows
at-most-k trick
fixed-size windows
hard shrink logic
```

* * *

# 12. The Single Most Important Sliding Window Insight

sliding window is **state management.**
the real skill is maintaining the window invariant

example invariant

```
window has no duplicate characters
```

OR

```
window contains <= k zeros
```

OR

```
window sum >= target
```

the entire algorithm revolves around **maintaining this invariant**
