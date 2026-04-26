# 🧩 Archetype 6 — Linked List

* * *

# 0. Goal

## What problem class does this solve?

* Problems on singly/doubly linked lists requiring pointer manipulation
* Problems where position relationships matter (middle, nth from end, cycle)
* Problems requiring structural transformation without extra space (reverse, reorder, merge)
* Problems that require detecting or resolving cycles in chains

## What mastery looks like

* Can identify which primitive(s) a problem needs in < 30 seconds
* Can write all 5 core primitives from scratch with no bugs
* Can compose multiple primitives in sequence (Reorder List, Palindrome Check)
* Can handle edge cases: empty list, single node, head deletion, cycle at entry
* Can explain WHY each pointer move is safe at each step

* * *

# 1. Recognition Signals

## Strong signals

* Input is a `ListNode` or singly/doubly linked list
* “Reverse”, “reorder”, “rotate” a linked list
* “Find the middle” of a list
* “Detect/find cycle” or “detect loop entry”
* “Merge two sorted lists”
* “Remove nth node from end”
* “Palindrome” on a list (can’t index backwards → must reverse half)
* “Copy list with random pointer”
* “Partition list around a value”

## Weak / disguised signals

* "Rearrange" elements with in-place constraint (can be list reorder)
* "Flatten" a nested structure into linear (multi-level list)
* "LRU Cache" (doubly linked list + hashmap)
* "Sort a linked list" (merge sort on list)

## Anti-signals (when NOT to use)

* Input is an array or string → use array-based two pointers instead
* No structural manipulation needed → just traverse and hash
* Problem asks for index access → wrong data structure entirely

* * *

# 2. Cheat Sheet

```
WHEN TO USE:
- Input is a listNode (not array)
- Need to find position: middle, nth from end, cycle
- Need to flip node order (whole or partial)
- Need to merge, interleave, or partition two lists
- Need to delete or insert at arbitrary position in-place

CORE IDEA:
- Linked list = rewiring arrows, not moving values
- You only control node.next
- Every problem composes from 5 atomic primitives

THE 5 PRIMITIVES:
1. Dummy Head         → head might change; return dummy.next
2. Fast/Slow Pointer  → position detection in one pass (2x speed differential)
3. In-place Reversal  → prev/curr/next dance; save next FIRST
4. Multi-ptr Weave    → dummy + tail; pick smaller, advance that list
5. Prev/Curr Tracking → delete needs predecessor; advance prev only on keep

TEMPLATE SELECTOR:
- Find middle?              → Fast/Slow, stop when fast hits end
- Find cycle?               → Fast/Slow, check slow == fast
- Find cycle entry?         → Fast/Slow phase 2: reset slow to head
- Kth from end?             → Gap trick: advance fast k steps, then both 1
- Reverse whole list?       → Reversal: prev=null, return prev
- Reverse sublist [L,R]?    → Dummy + front-insert loop, right-left iterations
- Merge two sorted?         → Dummy + tail weave
- Delete nodes?             → Dummy + prev/curr, advance prev only on keep
- Complex (reorder)?        → Fast/Slow → Reversal → Weave

TIME / SPACE:
- Most primitives:   O(n) time, O(1) space
- Merge k lists:     O(n log k) time, O(k) space (heap)
- Sort list:         O(n log n) time, O(log n) space (recursion stack)    

TOP 3 TRICKS:
1. Always save curr.next before curr.next = prev (or you lose the chain)
2. Always dummy.next = head when head might be deleted (return dummy.next)
3. Advance prev ONLY when NOT deleting curr (else consecutive dupes survive)

TOP 3 PITFALLS:
1. Missing null check: while (fast != null && fast.next != null) — BOTH required
2. Not cutting list after finding middle (creates cycle in Reorder / Palindrome)
3. Returning head instead of dummy.next after potential head deletion
```

* * *

# 3. Core Mental Model

## Core Ideas

* A linked list is a sequence of decisions made one step at a time
* You cannot index. You must traverse
* The only thing you ever control is `node.next`
* Every linked list problem is fundamentally about **rewiring arrows**, not moving values
* All solutions compose from **5 atomic primitives**

## The Node Contract

```
class ListNode {
    int val;
    ListNode next; // <- the ONLY lever you have
}
```



## The 5 Primitives

```
┌──────────────────────────┬─────────────────────────────────────────────┐
│ PRIMITIVE                │ ROLE                                        │
├──────────────────────────┼─────────────────────────────────────────────┤
│ 1. Dummy Head            │ Stable anchor when head might change        │
│ 2. Fast / Slow Pointer   │ Midpoint, cycle, kth-from-end in one pass   │
│ 3. In-place Reversal     │ Flip sequence of nodes without extra space  │
│ 4. Multi-pointer Weave   │ Merge, interleave, partition two lists      │
│ 5. Prev / Curr Tracking  │ Delete, insert, swap nodes safely           │
└──────────────────────────┴─────────────────────────────────────────────┘
```



## Problem Transformation

Original problem → Which primitives?

```
A. Single primitive   → e.g. Reverse List, Detect Cycle, Merge Two Lists
B. Two primitives     → e.g. Remove Nth (Fast/Slow + Prev/Curr), Reverse Sublist(Dummy + Reversal)
C. Three primitives   → e.g. Reorder List (Fast/Slow + Reversal + Weave)
```



## Visualization

### Dummy Head

```
dummy → [head] → [A] → [B] → null
  ↑
  anchor — never moves, protects against head deletion
```



### Fast / Slow (finding middle)

```
Step 0:  S         (slow, 1x)
         F         (fast, 2x)
         1 → 2 → 3 → 4 → 5

Step 1:  S=2, F=3
Step 2:  S=3, F=5   ← fast hits end, slow is at middle
```



### In-place Reversal (3-pointer dance)

```
Before:  null ← [prev]   [curr] → [next] → ...
After:   null ← [prev] ← [curr]   [next] → ...
```



### Multi-pointer Weave (merge)

```
l1: 1 → 3 → 5
l2: 2 → 4 → 6

tail picks smaller each step:
result: 1 → 2 → 3 → 4 → 5 → 6
```

* * *


# 4. Optimization Ladder (Interview Narrative)

## Step 1 — Brute Force

* Convert list to array
* Solve with array indexing
* Convert back to list

Time: O(n), Space: O(n) extra
Why it’s acceptable but not optimal:

* Allocates O(n) extra space
* Loses in-place requirement
* Does not demonstrate pointer fluency (what interviewers actually test)

## Step 2 — Optimization Insight

What repeating work are we eliminating?

* we are eliminating the extra space allocation
* we are proving that one traversal with the right pointer setup is sufficient
* The key shift: instead of materializing positions, we compute them relationally

Core shift:

```
from: indexing into a materialized array
to: maintaining pointer relationships during traversal
```

## Step 3 — Optimal Approach

* Key idea: All information can be derived in-place using pointer differentials(fast/slow), pointer direction reversal, or dual-list tracking
* Time: O(n) or O(nlogn) for sort
* Space: O(1) extra (except recursion stack if recursive)

* * *

# 5. Core Patterns

## Pattern 1 — Dummy Head Node

**When to use:**

* Head node might get deleted
* You’re inserting before the current head
* You want a uniform tail.next = ... loop without special casing

**Mental model:**

```
dummy.next = head
operate on dummy.next chain
return dummy.next as new head
```



```
// dummy head node template
ListNode dummy = new ListNode(0);
dummy.next = head;
ListNode curr = dummy;

// manipulate list via curr

return dummy.next // real head(even if original head was deleted)
```


**Why it works:**
`dummy` is never part of the logical list. it’s a sentinel that absorbs the edge case of head-change so your loop never needs a conditional for the first node.
* * *

## Pattern 2 — Fast / Slow Pointer

**When to use:**

* Find middle of list
* Detect cycle
* Find cycle entry point
* Find kth node from end

**Mental model:**

```
slow moves 1 step, fast moves 2 steps
-> when fast exhaust the list, slow has traveled half as far
-> when they meet inside a cycle, relative geometry pinpoints entry
```



### 2.1 — Find Middle

```
// Find middle template
ListNode slow = head, fast = head;

while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}
// slow is at middle (for even-length; left-middle)
return slow;
```

|Condition	|Odd result	|Even Result	|Use when	|
|---	|---	|---	|---	|
|`fast != null && fast.next != null`	|middle	|**2nd** middle	|LC 876 (return 2nd middle)	|
|`fast.next != null && fast.next.next != null`	|middle	|**1st** middle	|LC 141 (cycle), LC 142	|

**Boundary condition: `fast != null && fast.next != null`**
→ Never omit both checks. `fast.next.next` will NPE if `fast.next` is null.


### 2.2 — Detect Cycle

```
// detect cycle template
ListNode slow = head, fast = head;

while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow == fast) return true; // cycle confirmed
}
return false;
```



### 2.3 — Find Cycle Entry Point (Floyd's Phase 2)

```
// find cycle entry point template
// Phase 1: detect meeting point (same as 2.2)
// Phase 2: find entry
Node slow = head, fast = head;

while (fast.next != null && fast.next.next != null) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow == fast) break;
}
if (slow != fast) return null;

slow = head;
while (slow != fast) {
    slow = slow.next;
    fast = fast.next;
}
return slow; // slow == fast == cycle entry
```


**Why Phase 2 works (the math):**

```
let F = distance from head to cycle entry
let a = distance from entry to meeting point(inside cycle)
let C = cycle length

at meeting point:
    slow traveled: F + a
    fast traveled: F + a + n*C (lapped slow n times)
   
Since fast = 2 x slow:
 F + a + n*C = 2(F + a)
 F = n*C - a
 
 Resetting slow to head, both advance 1-step:
 - slow needs F more steps to reach entry
 - fast needs n*C - a = F more steps to wrap around to entry
 -> they converage at entry
```



### 2.4 — Find Kth Node From End (Gap Trick)

```
// find kth node from end template
ListNode slow = head, fast = head;

// advance fast k steps ahead
for(int i = 0; i < k; i++) {
    fast = fast.next;
}

// advance both until fast hits end
while (fast != null) {
    slow = slow.next;
    fast = fast.next;
}
// slow is now at kth node from end
return slow;
```


**Why it works:**


* fast is always k nodes ahead of slow

When fast = null(past the last node), slow is exactly k from end


## Pattern 3 — In-place Reversal

**When to use:**

* Reverse entire list
* Reverse a sublist [left, right]
* Any problem where order of nodes must be flipped in-place

**Mental model:**

```
Three-pointer dance: prev, curr, next
At each step: save next, rewrire current.next backward, advance both
```



### 3.1 — Reverse Entire List

```
// Reverse entire list template
Node prev = null;
Node curr = head;

while (curr != null) {
    Node next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
}
return prev;
```


**The one rule: always save `curr.next` BEFORE overwriting `curr.next = prev`**
Forgetting this severs the chain permanently 


### 3.2 — Reverse a Sublist [left, right]

```
// Reverse sublist template
Node dummy = new Node(0);
dummy.next = head;
Node prevM = dummy; // node just before m node

for(int i = 0; i < m - 1; i++) {
    prevM = prevM.next;
}
Node prev = null;
Node curr = prevM.next; // first node to reverse

for(int i = 0; i <= n - m; i++) {
    Node next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
}
prevM.next.next = curr;
prevM.next = prev;

return dummy.next;
```


**Mental model for sublist reversal:**


```
Each iteration takes the node immediately after curr and inserts it at the front
of the reversed segment. After `right - left` iterations, the segment is reversed
```



## Pattern 4 — Multi-pointer Weave

**When to use:**

* Merge two sorted lists
* Merge k sorted lists
* Partition list into two halves
* Interleave two halves(reorder list)

**Mental model:**

```
Build result by choosing which list to pull from at each step.
Use a dummy head + tail pointer that advances forward
```



### 4.1 — Merge Two Sorted Lists

```
// Merge two sorted lists template
Node l1 = head1, l2 = head2;
Node dummy = new Node(0);
Node tail = dummy;

while (l1 != null && l2 != null) {
    if (l1.val < l2.val) {
        tail.next = l1;
        l1 = l1.next;
    } else {
        tail.next = l2;
        l2 = l2.next;
    }
    tail = tail.next;
}

tail.next = l1 != null ? l1 : l2;

return dummy.next;
```



### 4.2 — Partition List(values < x left,  >= right)

```
// Parition list template

Node lessHead = new Node(0);
Node greaterHead = new Node(0);
Node lesser = lessHead;
Node greater = greaterHead;
Node curr = head;

while (curr != null) {
    if (curr.val < target) {
        lesser.next = curr;
        lesser = lesser.next;
    } else {
        greater.next = curr;
        greater = greater.next;
    }
    curr = curr.next;
}
greater.next = null;
lesser.next = greaterHead.next;

return lessHead.next;
```



### 4.3 — Merge K Sorted Lists (Heap approach)

```
// Merge K Sorted lists template
PriorityQueue<ListNode> heap = new PriorityQueue<>((a,b) -> a.val - b.val);

// seed heap with all list heads
for(ListNode node : lists) {
    if (node != null) heap.offer(node);
}

ListNode dummy = new ListNode(0);
ListNode tail = dummy;

while (!heap.isEmpty()) {
    ListNode curr = heap.poll(); // smallest head
    tail.next = curr;
    tail = tail.next;
    if (curr.next != null) heap.offer(curr.next); // advance that list
}
return dummy.next;
```


Time: O(n logk) where n = total nodes, k = number of lists

Each of the n nodes is pushed and popped exactly once.
Each push/pop costs O(log k) — heap size is bounded by k.
Total: O(n log k)


## Pattern 5 — Prev / Curr Tracking

**When to use:**

* Delete a node(need predecessor)
* Remove all occurrences of a value
* Swap pairs of nodes
* Any in-place edit requiring “the node before”

**Mental model:**

```
Singly linked list: you cannot go backward
To delete current, you need prev.next = current.next;
Always carry prev.
```



### 5.1 — Remove All Nodes with Target Value

```
// Remove all nodes with target value template
ListNode dummy = new ListNode(0);
dummy.next = head;
ListNode prev = dummy, curr = head;

while (curr != null) {
    if (curr.val == target) {
        prev.next = curr.next; // skip curr
    } else {
        prev = curr; // only advance prev when NOT deleting
    }
    curr = curr.next;
}
return dummy.next;
```


**Critical rule: advance `prev` only when you do NOT delete `curr`.**
if you delete and advance prev. you skip the check for consecutive duplicates.


### 5.2 — Swap Every Two Adjacent Nodes

```
// Swap every two adjacent nodes template
ListNode dummy = new ListNode(0);
dummy.next = head;
ListNode prev = dummy;

while (prev.next != null && prev.next.next != null) {
    ListNode first = prev.next;
    ListNode second = prev.next.next;
    
    first.next = second.next; // first skips over second
    second.next = first; // second points back to first
    prev.next = second; // prev connects to new front(second)
    
    prev = first; // advance past swapped pair
}

return dummy.next;
```


## Pattern 6 — Stack-based Arithmetic

**When to use:**

* Digits stored MSB-first, need to add from LSB
* Cannot reverse the list (follow-up constraint)
* Forward-order list but need backward processing

**Mental model:**

```
Push all nodes onto stack → pop in LSB-first order → sum with carry
Prepend result nodes (new ListNode(digit, head)) to avoid a reversal pass
```



```
// Stack arithmetic template (LC 2, LC 445)
Deque<Integer> s1 = new ArrayDeque<>(), s2 = new ArrayDeque<>();
while (l1 != null) { s1.push(l1.val); l1 = l1.next; }
while (l2 != null) { s2.push(l2.val); l2 = l2.next; }

ListNode head = null;
int carry = 0;
while (!s1.isEmpty() || !s2.isEmpty() || carry != 0) {
    int sum = carry;
    if (!s1.isEmpty()) sum += s1.pop();
    if (!s2.isEmpty()) sum += s2.pop();
    carry = sum / 10;
    head = new ListNode(sum % 10, head); // prepend — builds MSB-first directly
}
return head;
```


**Why prepend instead of append + reverse:**
Each new node becomes the new head. After the loop, the most recently computed digit
(most significant) is at the front — correct order without an extra pass.

* * *

## Pattern 7 — Make Circular + Cut (Rotate)

**When to use:**

* Rotate list right by k places
* Any problem where the tail should connect to the head

**Mental model:**

```
1. Find length + tail in one pass (tail.next == null)
2. Normalize: rotation = k % length
3. Make circular: tail.next = head
4. Walk to new tail: count - rotation - 1 steps from head
5. Cut: newHead = newTail.next; newTail.next = null
```



```
// Circular + cut template (LC 61)
int count = 1;
ListNode tail = head;
while (tail.next != null) { tail = tail.next; count++; }
int rotation = k % count;
if (rotation == 0) return head;

tail.next = head;                              // make circular
ListNode newTail = head;
for (int i = 0; i < count - rotation - 1; i++) newTail = newTail.next;
ListNode newHead = newTail.next;
newTail.next = null;                           // cut
return newHead;
```


**Why `count - rotation - 1`:**
The new tail is the node just before the new head. New head is at index `count - rotation`
(0-based), so new tail is at index `count - rotation - 1`.

* * *

## Pattern 8 — Two-pointer Intersection

**When to use:**

* Find where two lists share a node by reference (not value)
* No extra space allowed

**Mental model:**

```
Each pointer walks len(A) + len(B) total steps.
When a pointer exhausts its list, redirect it to the other list's head.
The length difference cancels — both arrive at intersection simultaneously.
If no intersection, both arrive at null simultaneously.
```



```
// Two-pointer intersection template (LC 160)
ListNode pA = headA, pB = headB;
while (pA != pB) {
    pA = (pA == null) ? headB : pA.next;
    pB = (pB == null) ? headA : pB.next;
}
return pA; // null if no intersection
```


**Why it works:**
`pA` travels `a + c + b` steps. `pB` travels `b + c + a` steps.
Same total distance → they sync at the intersection node (or null).
(`a` = unique prefix of A, `b` = unique prefix of B, `c` = shared suffix length)

* * *

## Pattern 9 — Bit-shift Accumulation

**When to use:**

* Binary or decimal number encoded MSB-first in a linked list
* Need to convert to integer in one pass without reversing

**Mental model:**

```
Reading MSB-first: each new digit shifts the accumulated result left
and ORs/adds the new digit in.
result = result * base + current_digit
```



```
// Bit-shift accumulation template (LC 1290)
int result = 0;
while (head != null) {
    result = (result << 1) | head.val;  // binary
    // result = result * 10 + head.val; // decimal
    head = head.next;
}
return result;
```


**The general pattern:**
`result = result * base + digit` works for any base. Same logic used
when parsing integers digit by digit from left to right.

* * *

## Pattern 10 — Node Interleaving (Deep Copy)

**When to use:**

* Deep copy a list that has arbitrary cross-pointers (random/next)
* O(1) space required (no HashMap)

**Mental model:**

```
Step 1: Weave clone nodes between originals
  A → A' → B → B' → C → C'
Step 2: Set random pointers while lists are still woven
  A'.random = A.random.next  (A.random.next is the clone of A.random)
Step 3: De-interleave to restore original and extract clone list
```



```
// Node interleaving template (LC 138)

// Step 1: interleave clones
ListNode curr = head;
while (curr != null) {
    ListNode clone = new ListNode(curr.val);
    clone.next = curr.next;
    curr.next  = clone;
    curr = clone.next;
}

// Step 2: set random pointers
curr = head;
while (curr != null) {
    if (curr.random != null) curr.next.random = curr.random.next;
    curr = curr.next.next;
}

// Step 3: de-interleave
ListNode cloneHead = head.next;
curr = head;
while (curr != null) {
    ListNode clone = curr.next;
    curr.next  = clone.next;
    clone.next = clone.next != null ? clone.next.next : null;
    curr = curr.next;
}
return cloneHead;
```


**Why Step 2 works:**
While lists are woven, `original.random.next` is exactly the clone of that random node —
it's sitting right next to it in the interleaved structure.

* * *

# 6. Flagship Problem — Reorder List (#143)

This problem is the capstone of this archetype. It chains **3 primitives in sequence**.

**Problem:** Given 1→2→3→4→5, reorder to 1→5→2→4→3


```
Step 1: Find middle          [Fast/Slow Pointer]
Step 2: Reverse second half  [In-place Reversal]
Step 3: Weave two halves     [Multi-pointer Weave]
```



```
public void reorderList(ListNode head) {
    if (head == null || head.next == null) return;

    // Step 1: Find middle
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    // slow is at end of first half

    // Step 2: Reverse second half
    ListNode secondHalf = reverseList(slow.next);
    slow.next = null; // cut the list

    // Step 3: Weave
    ListNode first = head, second = secondHalf;
    while (second != null) {
        ListNode firstNext = first.next;
        ListNode secondNext = second.next;
        first.next = second;
        second.next = firstNext;
        first = firstNext;
        second = secondNext;
    }
}

private ListNode reverseList(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```


* * *

# 7. Key Tricks & Insights

## Trick 1 — Always use Dummy Head when head might change

If you delete or insert before the head without a dummy, you need special-case logic
for the head. The dummy absorbs all edge cases into the same uniform loop.


```
Rule: head might change? → dummy.next = head. Always return dummy.next.
```



## Trick 2 — The Fast/Slow termination condition is non-negotiable

```
while (fast != null && fast.next != null) // ✅ correct

while (fast.next != null)                  // ❌ NPE when fast = null
while (fast != null)                       // ❌ NPE on fast.next.next
```


Check BOTH. Never skip one.


## Trick 3 — Save next BEFORE rewiring in reversal

```
ListNode next = curr.next;  // ← this line must come first
curr.next = prev;           // now safe to overwrite
```



## Trick 4 — Advance prev ONLY when NOT deleting

```
if (delete condition) {
    prev.next = curr.next;  // skip
    // do NOT move prev
} else {
    prev = curr;            // advance prev only here
}
curr = curr.next;
```


Forgetting this causes consecutive duplicates to survive.


## Trick 5 — For Reorder List, cut the list after finding middle

```
ListNode secondHalf = reverseList(slow.next);
slow.next = null; // ← must cut before reversing / weaving
```


Without the cut, the reversal runs into the original list and creates cycles.


## Trick 6 — Copy List with Random Pointer: interleave trick avoids O(n) space

```
Phase 1: interleave clones between originals: 1 → 1' → 2 → 2' → ...
Phase 2: set clone.random = original.random.next
Phase 3: de-interleave
```


This works in O(n) time and O(1) extra space (no hashmap).


## Trick 7 — Sort List uses Merge Sort, not in-place quicksort

Merge sort on lists is naturally O(n log n) with O(log n) stack space.
Quicksort on lists is O(n²) worst case.


```
Split with Fast/Slow → recursively sort halves → Merge
```

* * *

# 8. Common Pitfalls (🚨 HIGH ROI)

## Pitfall 1 — Not saving next before rewiring

```
curr.next = prev; // ❌ you just lost the rest of the list
```


Fix: always `ListNode next = curr.next;` first.


## Pitfall 2 — Wrong fast/slow termination (NPE on fast.next.next)

```
while (fast.next != null)                  // ❌ NPE
while (fast != null && fast.next != null)  // ✅
```



## Pitfall 3 — Returning head instead of dummy.next when head was deleted

```
return head;        // ❌ if original head was the deleted node
return dummy.next;  // ✅ always safe
```



## Pitfall 4 — Not cutting the list before second half operations

In Reorder List and Palindrome, after finding the middle:

```
slow.next = null; // ← required cut
```

Without it, traversal of the first half will follow into the second half.


## Pitfall 5 — Advancing prev when deleting

```
// Deleting curr:
prev.next = curr.next;
prev = curr; // ❌ prev just jumped to the deleted node's position
```


Fix: only advance `prev` in the else branch.


## Pitfall 6 — Off-by-one in sublist reversal bounds

For Reverse Linked List II [left, right]:

```
for (int i = 1; i < left; i++)         // walk to node before left: loop runs left-1 times
for (int i = 0; i < right - left; i++) // reverse: loop runs right-left times
```


Count carefully. The reversal loop runs `right - left` times (not `right - left + 1`).


## Pitfall 7 — Forgetting to null-terminate the right partition

In Partition List:

```
rightTail.next = null; // ← required or the tail still points into original list
```

* * *

# 9. Edge Case Checklist

General linked list edge cases to check before coding:


* Empty list (`head == null`)
* Single node (`head.next == null`)
* Two nodes (many reversal/swap problems break here)
* All nodes have the same value (dedup/removal problems)
* Target is the head node (tests dummy head handling)
* Target is the tail node (tests prev/curr tracking)
* Cycle at the last node pointing back to head
* Cycle at the last node pointing back to itself (self-loop)
* k > list length (for kth-from-end problems: validate k)
* left == right for Reverse Sublist (no-op — code must not break)

* * *

# 10. Decision Framework (🔥 INTERVIEW WEAPON)

## Step 1: Is the input a linked list(not an array)?

→ YES → pointer manipulation, not indexing

## Step 2: Does the head node possibly change?

→ YES → use Dummy Head immediately 

## Step 3: What is the structural goal?

```
Find a position (middle, nth from end, cycle)?   → Fast/Slow Pointer
Flip order (whole or part)?                      → In-place Reversal
Combine two lists (sorted merge, interleave)?    → Multi-pointer Weave
Delete, insert, or swap a node?                  → Prev/Curr Tracking
```

## Step 4: Does the problem chain multiple goals?

```
Find middle + flip second half + interleave?  → Reorder List (3 primitives)
Find middle + flip second half + compare?     → Palindrome Check (3 primitives)
```

### LinkedList Decision Tree

```
Input is a linked list?
└── YES
    ├── Head might change?            → add Dummy Head
    ├── Find position?
    │   ├── Middle / cycle?           → Fast/Slow Pointer
    │   └── Kth from end?            → Fast/Slow Gap Trick
    ├── Flip order?
    │   ├── Whole list?               → In-place Reversal (prev/curr/next dance)
    │   └── Sublist [left, right]?   → Dummy + Sublist Reversal
    ├── Combine two lists?
    │   ├── Merge sorted?             → Dummy + Weave
    │   └── Merge k lists?           → Heap + Weave
    ├── Delete / insert / swap?       → Prev/Curr Tracking
    └── Complex (reorder, palindrome)?→ Compose 2-3 primitives in sequence
```


* * *

# 11. Problem Map Quick Reference

```
PRIMITIVE                   PROBLEMS
──────────────────────────────────────────────────────────
Dummy Head                  #21, #23 (setup), #24, #82, #83, #86, #147,
                            #148, #160, #203, #328, #445, #1669
Fast/Slow Pointer           #61, #141, #142, #143, #148, #234, #876, #2130
                            #19
In-place Reversal           #92, #143, #147, #206, #234, #2130
Multi-pointer Weave         #21, #23, #86, #143, #147, #1669
Prev/Curr Tracking          #19, #24, #82, #83, #203, #237, #328
Stack + Arithmetic          #2, #445
Circular + Cut              #61
Two-pointer Intersection    #160
Node Interleaving           #138
Bit-shift Accumulation      #1290
3-Primitive Composition     #143 (Fast/Slow + Reversal + Weave)
                            #234 (Fast/Slow + Reversal + Compare)
                            #148 (Fast/Slow + Merge recursion)
                            #2130 (Fast/Slow + Reversal + Twin compare)
```

* * *

# 12. Drill Section (Mastery Check)

## Verbal Drill

Explain this archetype in 2 minutes.

Target structure:


1. Linked list problems require pointer manipulation, not indexing
2. All problems compose from 5 primitives: dummy head, fast/slow, reversal, weave, prev/curr
3. Fast/slow works because 2x speed creates a position differential in one pass
4. Reversal requires 3 pointers and always saves next before overwriting
5. Dummy head eliminates head-change edge cases uniformly

## Recognition Drill

Given a problem, identify the primitives before coding.

Ask yourself:


* Does the head possibly change? → dummy
* Am I finding a structural position? → fast/slow
* Am I flipping order? → reversal
* Am I combining two lists? → weave
* Am I deleting or inserting? → prev/curr tracking
* Does this chain multiple goals? → compose in sequence

## Transformation Drill

Convert brute → optimal without coding:


* **Reverse Linked List**
    * Brute: convert to array, reverse, rebuild list
    * Optimal: 3-pointer in-place dance, O(1) space
* **Detect Cycle**
    * Brute: HashSet of visited nodes, O(n) space
    * Optimal: fast/slow pointer, they meet iff cycle exists, O(1) space
* **Remove Nth From End**
    * Brute: traverse once to get length L, traverse again to L-n
    * Optimal: gap trick — fast pointer L steps ahead, both advance together
* **Merge Two Sorted Lists**
    * Brute: collect all nodes into array, sort, rebuild
    * Optimal: dummy + tail weave, never copy values
* **Reorder List**
    * Brute: convert to array, reindex, rebuild
    * Optimal: find middle → reverse second half → weave two halves

## Primitive Speed Test

Write each from scratch with no reference. Target: < 3 min each, zero bugs.


```
□ reverseList(ListNode head)
□ findMiddle(ListNode head)
□ hasCycle(ListNode head)
□ findCycleEntry(ListNode head)
□ mergeTwoSorted(ListNode l1, ListNode l2)
□ removeNthFromEnd(ListNode head, int n)
```

## Pointer Manipulation Primitive Drills (2026-04-23)

Write each from scratch with no reference. Target: < 8 min each.

```
✅ reverseSublist(ListNode head, int left, int right)
   — Bug caught: prevLeft = head → should be prevLeft = dummy (off-by-one anchor)

✅ partitionList(ListNode head, int x)
   — Bug caught: missing greater.next = null (cycle risk on greater tail)

✅ mergeKSorted(List<ListNode> lists)
   — Note: always declare comparator explicitly: new PriorityQueue<>((a,b) -> a.val - b.val)

✅ removeTarget(ListNode head, int target)
   — Clean first attempt

✅ swapAdjacent(ListNode head)
   — Clean first attempt
```


* * *

# 13. Problem Approach Template

```
# LinkedList Problem Quick Drill

Signal check:
- Why linked list primitives?

Primitives needed:
- □ Dummy Head
- □ Fast/Slow Pointer
- □ In-place Reversal
- □ Multi-pointer Weave
- □ Prev/Curr Tracking

Composition order (if multiple):
- Step 1:
- Step 2:
- Step 3:

Does head possibly change?
- YES / NO

Key pointer invariants:
-

Edge cases:
-
```

Drills:


```
1. #206 Reverse Linked List
Primitives:
- In-place Reversal

Approach:
1. prev = null, curr = head
2. save next, rewire curr.next = prev, advance both
3. return prev

Invariant:
- nodes left of curr are reversed; prev is the new tail of reversed portion

Return:
- prev (new head)

Edge cases:
- empty list: curr = null → while never runs → prev = null → return null ✓
- single node: curr = head, rewire done in 1 step, prev = head → return head ✓


2. #141 Linked List Cycle
Primitives:
- Fast/Slow Pointer

Approach:
1. slow = head, fast = head
2. advance slow 1, fast 2 each iteration
3. if slow == fast → cycle found

Invariant:
- if no cycle, fast reaches null; if cycle, fast laps slow and they meet

Return:
- true / false

Edge cases:
- null head: fast = null → while condition false → return false ✓
- single node, no cycle: fast.next = null → while exits ✓


3. #21 Merge Two Sorted Lists
Primitives:
- Dummy Head + Multi-pointer Weave

Approach:
1. dummy = new ListNode(0), tail = dummy
2. while both non-null: pick smaller, tail.next = it, advance that list, advance tail
3. attach remaining

Invariant:
- result[0..tail] = merged sorted prefix; l1 and l2 point to unprocessed remainders

Return:
- dummy.next

Edge cases:
- one list empty: while never runs, tail.next = other list ✓
- both empty: while never runs, tail.next = null ✓


4. #19 Remove Nth Node From End
Primitives:
- Fast/Slow Pointer (gap) + Dummy Head + Prev/Curr Tracking

Approach:
1. dummy.next = head; fast = dummy; slow = dummy
2. advance fast n+1 steps (so slow lands on node before target)
3. advance both until fast = null
4. slow.next = slow.next.next

Invariant:
- fast is always n+1 ahead of slow; when fast = null, slow is at predecessor of target

Return:
- dummy.next

Edge cases:
- removing head (n = length): fast lands at null after n+1 steps from dummy, slow stays at dummy ✓


5. #143 Reorder List
Primitives:
- Fast/Slow Pointer + In-place Reversal + Multi-pointer Weave

Approach:
1. find middle via fast/slow (use fast.next && fast.next.next for left-middle)
2. slow.next = null (cut list)
3. reverse second half
4. weave first and reversed second half

Invariant:
- after cut: two independent lists; after reversal: second half in correct pickup order; weave merges them

Return:
- void (in-place)

Edge cases:
- length 1 or 2: return early, no reorder needed ✓


6. #234 Palindrome Linked List
Primitives:
- Fast/Slow Pointer + In-place Reversal + comparison

Approach:
1. find middle via fast/slow
2. reverse second half
3. compare first half and reversed second half node by node

Invariant:
- second half reversed = correct order to compare against first half

Return:
- boolean

Edge cases:
- odd length: middle node is not compared (second half has fewer nodes than first) ✓
- single node: always palindrome ✓


7. #61 Rotate List
Primitives:
- Fast/Slow Pointer (find length + tail) + Circular + Cut

Approach:
1. walk to find length and tail in one pass
2. rotation = k % length; if 0, return head
3. tail.next = head (make circular)
4. walk count - rotation - 1 steps to find new tail
5. newHead = newTail.next; newTail.next = null

Invariant:
- after circular step, the list is a ring; new tail is exactly rotation nodes from the end

Return:
- newHead

Edge cases:
- k == 0: return head immediately ✓
- rotation == 0 (k % length == 0): return head unchanged ✓
- single node: return head ✓


8. #82 Remove Duplicates from Sorted List II
Primitives:
- Dummy Head + Prev/Curr Tracking (forward-look)

Approach:
1. dummy.next = head; prev = dummy
2. while prev.next != null:
   - if prev.next.val == prev.next.next.val: record dupVal, skip all nodes with dupVal via prev.next = prev.next.next
   - else: prev = prev.next

Invariant:
- everything before prev (inclusive) is de-duped; prev never points to a duplicate node

Return:
- dummy.next

Edge cases:
- all nodes duplicate: prev stays at dummy, all skipped ✓
- no duplicates: prev advances every step ✓


9. #160 Intersection of Two Linked Lists
Primitives:
- Two-pointer Intersection (list swap)

Approach:
1. pA = headA, pB = headB
2. while pA != pB: advance each; redirect to other list's head when null
3. return pA (intersection node or null)

Invariant:
- pA travels a + c + b steps; pB travels b + c + a; same total → sync at intersection

Return:
- intersection node by reference, or null

Edge cases:
- no intersection: both reach null simultaneously (null == null exits loop) ✓
- one list is prefix of other: handled by redirect ✓


10. #138 Copy List with Random Pointer
Primitives:
- Node Interleaving (O(1) space) or HashMap (O(n) space)

Approach (optimal — interleaving):
1. weave clone nodes between originals: A → A' → B → B'
2. set clone.random = original.random.next (while woven)
3. de-interleave: restore original, extract clone list

Invariant:
- during Step 2, A.random.next is always the clone of A.random (structural adjacency)

Return:
- cloneHead (head.next after interleaving)

Edge cases:
- null head: return null ✓
- random points to null: guard curr.random != null before accessing .next ✓
- random points to itself: handled naturally (self.next = self's clone) ✓


11. #2 / #445 Add Two Numbers (reverse / forward order)
Primitives:
- #2: Dummy Head + Prev/Curr (single pass, digits in reverse order)
- #445: Stack + Arithmetic + Prepend (digits in forward order)

Approach (#2):
1. single loop: while l1 || l2 || carry
2. sum = carry + (l1.val if l1) + (l2.val if l2)
3. carry = sum / 10; append sum % 10

Approach (#445):
1. push both lists onto stacks
2. pop and sum with carry; prepend each result node (head = new ListNode(digit, head))

Invariant:
- carry propagates forward; loop condition includes carry != 0 to handle final overflow

Return:
- dummy.next (#2) / head (#445)

Edge cases:
- different lengths: nulls handled by conditional addition ✓
- final carry (e.g. 999 + 1 = 1000): loop runs one extra iteration ✓


12. #2130 Maximum Twin Sum of a Linked List
Primitives:
- Fast/Slow Pointer + In-place Reversal + parallel walk

Approach:
1. fast/slow to find end of first half (use fast.next && fast.next.next condition)
2. reverse second half (slow.next onward); cut at slow.next = null
3. walk left and reversed-right in parallel; track max(left.val + right.val)

Invariant:
- after reversal, right[i] is the twin of left[i]; walking both together covers all twin pairs

Return:
- maxSum (int)

Edge cases:
- guaranteed even length per constraints; no empty/odd check needed ✓
- twin sums always ≥ 2 (val ≥ 1); maxSum = 0 init is safe ✓
```

