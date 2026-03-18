# Problem Bank: Linked Lists

## Easy Problems

### 1. Reverse Linked List

**Statement**: Given the head of a singly linked list, reverse the list and return the reversed list.

**Example**:
```
Input:  1 -> 2 -> 3 -> 4 -> 5 -> NULL
Output: 5 -> 4 -> 3 -> 2 -> 1 -> NULL
```

**Optimal**: Iterative with three pointers, O(n) time, O(1) space
**Alternative**: Recursive, O(n) time, O(n) space (call stack)

**Walkthrough**:
```
1 -> 2 -> 3 -> NULL
prev = NULL, curr = 1

Step 1: next = 2
        curr.next = NULL (prev)
        NULL <- 1    2 -> 3 -> NULL
        prev = 1, curr = 2

Step 2: next = 3
        curr.next = 1 (prev)
        NULL <- 1 <- 2    3 -> NULL
        prev = 2, curr = 3

Step 3: next = NULL
        curr.next = 2 (prev)
        NULL <- 1 <- 2 <- 3
        prev = 3, curr = NULL

Return prev -> 3 -> 2 -> 1 -> NULL
```

**Follow-ups**:
- Can you do it recursively?
- Can you reverse only a sub-section of the list (positions m to n)?
- Can you reverse in groups of k?

---

### 2. Linked List Cycle

**Statement**: Given the head of a linked list, determine if the list has a cycle in it. A cycle exists if some node's `next` pointer points back to a previously visited node.

**Example**:
```
Input:  1 -> 2 -> 3 -> 4 -> 5
                  ^              |
                  |______________|
Output: True (node 5 points back to node 3)
```

**Optimal**: Floyd's Cycle Detection (fast/slow pointers), O(n) time, O(1) space
**Alternative**: HashSet to track visited nodes, O(n) time, O(n) space

**Walkthrough (Floyd's Algorithm)**:
```
1 -> 2 -> 3 -> 4 -> 5
               ^         |
               |_________|

slow and fast both start at node 1

Step 1: slow = 2, fast = 3
Step 2: slow = 3, fast = 5
Step 3: slow = 4, fast = 4  (5 -> 3 -> 4 for fast)

slow == fast -> cycle detected! Return True.

Without a cycle (1 -> 2 -> 3 -> NULL):
Step 1: slow = 2, fast = 3
Step 2: fast.next is NULL -> no cycle. Return False.
```

**Follow-ups**:
- Can you find the node where the cycle begins? (Floyd's Phase 2)
- What is the length of the cycle?

---

### 3. Merge Two Sorted Lists

**Statement**: Merge two sorted linked lists into one sorted list by splicing nodes together.

**Example**:
```
Input:  L1: 1 -> 3 -> 5 -> NULL
        L2: 2 -> 4 -> 6 -> NULL
Output: 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> NULL
```

**Optimal**: Iterative with dummy head, O(n + m) time, O(1) space
**Alternative**: Recursive, O(n + m) time, O(n + m) space (call stack)

**Walkthrough**:
```
L1: 1 -> 3 -> 5 -> NULL
L2: 2 -> 4 -> 6 -> NULL
dummy -> NULL, tail = dummy

Step 1: 1 <= 2 -> attach 1
  dummy -> 1, tail = 1
  L1 = 3

Step 2: 3 > 2 -> attach 2
  dummy -> 1 -> 2, tail = 2
  L2 = 4

Step 3: 3 <= 4 -> attach 3
  dummy -> 1 -> 2 -> 3, tail = 3
  L1 = 5

Step 4: 5 > 4 -> attach 4
  dummy -> 1 -> 2 -> 3 -> 4, tail = 4
  L2 = 6

Step 5: 5 <= 6 -> attach 5
  dummy -> 1 -> 2 -> 3 -> 4 -> 5, tail = 5
  L1 = NULL

Step 6: L1 is NULL -> attach remaining L2
  dummy -> 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> NULL

Return dummy.next
```

**Follow-ups**:
- What if the lists have different lengths?
- Can you merge k sorted lists efficiently? (Heap or divide-and-conquer)

---

## Medium Problems

### 4. Remove Nth Node From End of List

**Statement**: Given the head of a linked list, remove the nth node from the end and return the head.

**Example**:
```
Input:  1 -> 2 -> 3 -> 4 -> 5, n = 2
Output: 1 -> 2 -> 3 -> 5
(Removed node 4, which is 2nd from the end)
```

**Optimal**: Two pointers with n-gap, O(n) time, O(1) space, single pass
**Alternative**: Two passes (first to count length, second to remove), O(n) time, O(1) space

**Walkthrough**:
```
1 -> 2 -> 3 -> 4 -> 5, n = 2

Use a dummy node to handle edge case of removing the head:
dummy -> 1 -> 2 -> 3 -> 4 -> 5

Step 1: Advance fast pointer n+1 steps from dummy
  fast = dummy
  Move 1: fast = 1
  Move 2: fast = 2
  Move 3: fast = 3
  slow = dummy

Step 2: Move both until fast reaches NULL
  fast = 4, slow = 1
  fast = 5, slow = 2
  fast = NULL, slow = 3

Step 3: slow.next is the node to remove (4)
  slow.next = slow.next.next
  dummy -> 1 -> 2 -> 3 -> 5

Return dummy.next
```

**Follow-ups**:
- What if n equals the length of the list (remove the head)?
- Can you do it in one pass? (Yes, with the two-pointer gap technique)

---

### 5. Palindrome Linked List

**Statement**: Given the head of a singly linked list, determine if it is a palindrome.

**Example**:
```
Input:  1 -> 2 -> 3 -> 2 -> 1 -> NULL
Output: True
```

**Optimal**: Find middle, reverse second half, compare, O(n) time, O(1) space
**Alternative**: Copy to array, check palindrome, O(n) time, O(n) space

**Walkthrough**:
```
1 -> 2 -> 3 -> 2 -> 1 -> NULL

Step 1: Find the middle using slow/fast pointers
  slow = 1, fast = 1
  slow = 2, fast = 3
  slow = 3, fast = 1 (end)
  Middle = slow = 3

Step 2: Reverse the second half starting from middle.next
  Before: 3 -> 2 -> 1 -> NULL
  After:  3    1 -> 2 -> NULL
  (We reverse 2 -> 1 to get 1 -> 2)

Step 3: Compare first half with reversed second half
  First half:    1 -> 2 -> 3
  Reversed half: 1 -> 2 -> NULL

  1 == 1? Yes
  2 == 2? Yes
  Reversed half is NULL -> done

  Return True

Non-palindrome example: 1 -> 2 -> 3 -> NULL
  Middle = 2
  Reverse after middle: 3 -> NULL
  Compare: 1 vs 3 -> not equal -> False
```

**Follow-ups**:
- Can you restore the list to its original structure after checking?
- How would you handle an even-length list?

---

### 6. Reorder List

**Statement**: Given a singly linked list L0 -> L1 -> ... -> Ln-1 -> Ln, reorder it to L0 -> Ln -> L1 -> Ln-1 -> L2 -> Ln-2 -> ...

**Example**:
```
Input:  1 -> 2 -> 3 -> 4 -> 5 -> NULL
Output: 1 -> 5 -> 2 -> 4 -> 3 -> NULL
```

**Optimal**: Find middle + reverse second half + merge alternating, O(n) time, O(1) space

**Walkthrough**:
```
1 -> 2 -> 3 -> 4 -> 5 -> NULL

Step 1: Find middle (slow/fast pointers)
  slow = 1, fast = 1
  slow = 2, fast = 3
  slow = 3, fast = 5
  Middle = slow = 3

Step 2: Split into two halves
  First half:  1 -> 2 -> 3 -> NULL  (cut after middle)
  Second half: 4 -> 5 -> NULL

Step 3: Reverse second half
  5 -> 4 -> NULL

Step 4: Merge alternating
  Take from first:  1
  Take from second: 5
  Take from first:  2
  Take from second: 4
  Take from first:  3

  Result: 1 -> 5 -> 2 -> 4 -> 3 -> NULL
```

**Detailed merge step**:
```
first  = 1 -> 2 -> 3 -> NULL
second = 5 -> 4 -> NULL

Iteration 1:
  Save: f_next = 2, s_next = 4
  1 -> 5 -> 2 -> 3 -> NULL
  first = 2, second = 4

Iteration 2:
  Save: f_next = 3, s_next = NULL
  1 -> 5 -> 2 -> 4 -> 3 -> NULL
  first = 3, second = NULL

second is NULL -> done
Result: 1 -> 5 -> 2 -> 4 -> 3 -> NULL
```

**Follow-ups**:
- This problem combines three fundamental linked list operations. Can you identify them?
- What edge cases should you handle? (Empty list, single node, two nodes)

---

## Advanced Problems (For candidates who ace everything)

### 7. Reverse Nodes in k-Group

**Statement**: Given a linked list, reverse the nodes in groups of k. If the remaining nodes are fewer than k, leave them as-is.

**Optimal**: Iterative group reversal, O(n) time, O(1) space

**Walkthrough**:
```
Input: 1 -> 2 -> 3 -> 4 -> 5, k = 3

Group 1 (nodes 1,2,3): Reverse
  3 -> 2 -> 1

Group 2 (nodes 4,5): Only 2 nodes, fewer than k=3, leave as-is
  4 -> 5

Result: 3 -> 2 -> 1 -> 4 -> 5 -> NULL
```

### 8. Copy List with Random Pointer

**Statement**: A linked list where each node has an additional `random` pointer to any node or NULL. Create a deep copy of the list.

**Optimal**: Interleave copies then separate, O(n) time, O(1) space
**Alternative**: HashMap mapping original to copy, O(n) time, O(n) space

---

## Sample Session Flows

### Session Flow 1: Candidate Struggles (Easy Path)

**Interviewer**: "Welcome! Let's start with a quick warm-up. Can you tell me what a singly linked list is?"
**Candidate**: "It's like an array but with pointers?"
**Interviewer**: "That's a good start. Each node holds a value and a pointer to the next node. Unlike an array, the nodes are not stored in contiguous memory. What's the trade-off?"
**Candidate**: "Um, you can't access elements by index?"
**Interviewer**: "Exactly. Array access is O(1) by index, but linked list traversal is O(n). However, inserting at the head of a linked list is O(1). Let's try a problem."

*[Present Reverse a Linked List]*

**Interviewer**: "Given 1 -> 2 -> 3, reverse it to 3 -> 2 -> 1. What needs to happen?"
**Candidate**: *[Long pause]*
**Interviewer**: "Let me draw it out. Right now 1 points to 2 and 2 points to 3. After reversal, 3 should point to 2 and 2 should point to 1. What needs to change about each node?"
**Candidate**: "Each node's next pointer needs to point backwards."
**Interviewer**: "Exactly. Now, if you change 1's next pointer to NULL, how do you get to node 2? You need to save a reference to it first. That's the key insight -- you need three pointers."

*[Walk through step by step with ASCII diagrams]*

### Session Flow 2: Candidate Excels (Hard Path)

**Interviewer**: "What's a singly linked list and when would you prefer it over an array?"
**Candidate**: "A linked list is a chain of nodes where each node has data and a next pointer. You'd prefer it when you need frequent insertions and deletions at arbitrary positions since those are O(1) once you have a reference to the node, versus O(n) for arrays."
**Interviewer**: "Great nuance. Let's jump to a problem."

*[Present Reorder List directly]*

**Candidate**: *[Quickly identifies the three-step approach: find middle, reverse second half, merge]*
**Interviewer**: "Perfect. Can you code that up? And tell me -- what's the time and space complexity?"
**Candidate**: "O(n) time, O(1) space."
**Interviewer**: "Excellent. Follow-up: Can you generalize this to reverse nodes in groups of k?"

*[Challenge with Reverse Nodes in k-Group]*

### Session Flow 3: Average Candidate (Standard Path)

**Interviewer**: "Can you tell me the difference between a singly and doubly linked list?"
**Candidate**: "A singly linked list has one pointer to the next node. A doubly linked list also has a pointer to the previous node."
**Interviewer**: "Good. And what's the trade-off?"
**Candidate**: "Doubly linked lists use more memory but you can traverse backwards."
**Interviewer**: "Right. Let's start with a classic: reverse a singly linked list."

*[Candidate solves reversal with one hint about saving the next pointer]*

**Interviewer**: "Nice work. Now let's try something that builds on a different technique. Given a linked list, how would you detect if it has a cycle?"
**Candidate**: "I could use a set to track visited nodes?"
**Interviewer**: "That works. What's the space complexity?"
**Candidate**: "O(n)."
**Interviewer**: "Can we do it in O(1) space? Think about two people walking around a circular track at different speeds."

*[Guide toward Floyd's algorithm, then wrap up with feedback]*
