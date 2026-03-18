# Problem Bank: Heaps & Priority Queues

## Medium Problems

### 1. Kth Largest Element in an Array

**Statement**: Given an integer array `nums` and an integer `k`, return the kth largest element in the array. Note that it is the kth largest element in the sorted order, not the kth distinct element.

**Example**:
```
Input: nums = [3, 2, 1, 5, 6, 4], k = 2
Output: 5

Input: nums = [3, 2, 3, 1, 2, 4, 5, 5, 6], k = 4
Output: 4
```

**Optimal**: Min-heap of size K, O(n log k) time, O(k) space
**Alternative 1**: Quickselect, O(n) average time, O(1) space
**Alternative 2**: Sort, O(n log n) time, O(1) space

**Walkthrough (Min-Heap approach)**:
```
nums = [3, 2, 1, 5, 6, 4], k = 2

Step 1: Process first K elements into min-heap
  heap = [2, 3]  (min-heap, top is 2)

Step 2: Process remaining elements
  num=1: 1 < heap_min(2) -> skip
  num=5: 5 > heap_min(2) -> pop 2, push 5
         heap = [3, 5]
  num=6: 6 > heap_min(3) -> pop 3, push 6
         heap = [5, 6]
  num=4: 4 < heap_min(5) -> skip

Step 3: heap_min = 5 = 2nd largest element
```

**Walkthrough (Quickselect approach)**:
```
nums = [3, 2, 1, 5, 6, 4], k = 2
Target index = n - k = 6 - 2 = 4

Partition around pivot (random, say 4):
  [3, 2, 1] 4 [5, 6]
  pivot at index 3

  Target 4 > 3, search right: [5, 6]
  Partition around 5:
  [] 5 [6]
  pivot at index 4

  Target 4 == 4, found! Answer = 5
```

**Follow-ups**:
- What if k changes frequently? (Maintain a heap)
- What if the array is too large to fit in memory? (External sorting or distributed approach)
- What is the worst case for Quickselect and how to avoid it? (Median of medians)

---

### 2. Merge K Sorted Lists

**Statement**: You are given an array of `k` linked lists, each sorted in ascending order. Merge all the linked lists into one sorted linked list.

**Example**:
```
Input: lists = [[1,4,5],[1,3,4],[2,6]]
Output: [1,1,2,3,4,4,5,6]
```

**Optimal**: Min-heap of K list heads, O(N log K) time, O(K) space (N = total elements)
**Alternative 1**: Merge lists pairwise (divide and conquer), O(N log K) time, O(1) space
**Alternative 2**: Merge one by one, O(N*K) time

**Walkthrough (Min-Heap approach)**:
```
lists = [[1,4,5], [1,3,4], [2,6]]

Initialize heap with head of each list:
  heap = [(1, list0), (1, list1), (2, list2)]

Step 1: Pop (1, list0). Result: [1]
  Push list0.next = 4. heap = [(1, list1), (2, list2), (4, list0)]

Step 2: Pop (1, list1). Result: [1, 1]
  Push list1.next = 3. heap = [(2, list2), (4, list0), (3, list1)]

Step 3: Pop (2, list2). Result: [1, 1, 2]
  Push list2.next = 6. heap = [(3, list1), (4, list0), (6, list2)]

Step 4: Pop (3, list1). Result: [1, 1, 2, 3]
  Push list1.next = 4. heap = [(4, list0), (6, list2), (4, list1)]

Step 5: Pop (4, list0). Result: [1, 1, 2, 3, 4]
  Push list0.next = 5. heap = [(4, list1), (6, list2), (5, list0)]

Step 6: Pop (4, list1). Result: [1, 1, 2, 3, 4, 4]
  list1 exhausted. heap = [(5, list0), (6, list2)]

Step 7: Pop (5, list0). Result: [1, 1, 2, 3, 4, 4, 5]
  list0 exhausted. heap = [(6, list2)]

Step 8: Pop (6, list2). Result: [1, 1, 2, 3, 4, 4, 5, 6]
  list2 exhausted. heap = []

Done. Result: [1, 1, 2, 3, 4, 4, 5, 6]
```

**Walkthrough (Divide and Conquer approach)**:
```
Round 1: Merge pairs
  [1,4,5] + [1,3,4] -> [1,1,3,4,4,5]
  [2,6] (unpaired, carry forward)

Round 2: Merge pairs
  [1,1,3,4,4,5] + [2,6] -> [1,1,2,3,4,4,5,6]

log(K) rounds, each round processes N total elements.
```

**Follow-ups**:
- What if the lists are sorted arrays instead of linked lists?
- What if new lists can be added dynamically?
- How would you distribute this across multiple machines?

---

### 3. Find Median from Data Stream

**Statement**: The median is the middle value in an ordered integer list. If the size of the list is even, the median is the average of the two middle values. Implement the `MedianFinder` class with `addNum(int num)` and `findMedian()`.

**Example**:
```
addNum(1)    -> [1]           median = 1
addNum(2)    -> [1, 2]        median = 1.5
addNum(3)    -> [1, 2, 3]     median = 2
addNum(7)    -> [1, 2, 3, 7]  median = 2.5
addNum(4)    -> [1, 2, 3, 4, 7] median = 3
```

**Optimal**: Two heaps (max-heap for lower half, min-heap for upper half), O(log n) add, O(1) median
**Alternative 1**: Sorted list with binary search insert, O(n) add, O(1) median
**Alternative 2**: Self-balancing BST (AVL/Red-Black), O(log n) add, O(log n) median

**Walkthrough (Two-Heap approach)**:
```
small = max-heap (stores lower half, negate for Python)
large = min-heap (stores upper half)

addNum(1):
  Push to small: small=[-1], large=[]
  Move top of small to large: small=[], large=[1]
  Rebalance (large bigger): move top of large to small
  small=[-1], large=[]
  Median: -small[0] = 1

addNum(2):
  Push to small: small=[-2, -1], large=[]
  Move top of small to large: small=[-1], large=[2]
  Balanced (same size).
  Median: (-small[0] + large[0]) / 2 = (1+2)/2 = 1.5

addNum(3):
  Push to small: small=[-3, -1], large=[2]
  Move top of small to large: small=[-1], large=[2, 3]
  Rebalance (large bigger): move 2 to small
  small=[-2, -1], large=[3]
  Median: -small[0] = 2

addNum(7):
  Push to small: small=[-7, -2, -1], large=[3]
  Move top of small to large: small=[-2, -1], large=[3, 7]
  Balanced.
  Median: (-small[0] + large[0]) / 2 = (2+3)/2 = 2.5

addNum(4):
  Push to small: small=[-4, -2, -1], large=[3, 7]
  Move top of small to large: small=[-2, -1], large=[3, 4, 7]
  Rebalance: move 3 to small
  small=[-3, -2, -1], large=[4, 7]
  Median: -small[0] = 3
```

**Key Invariants**:
- All elements in small <= all elements in large
- len(small) == len(large) or len(small) == len(large) + 1
- Median is either top of small (odd count) or average of both tops (even count)

**Follow-ups**:
- What if you also need to remove elements? (Lazy deletion with a hash map)
- What if the stream has billions of elements? (Approximate with count-min sketch or t-digest)
- What if you need a sliding window median? (LeetCode 480)

---

### 4. Top K Frequent Elements

**Statement**: Given an integer array `nums` and an integer `k`, return the `k` most frequent elements.

**Example**:
```
Input: nums = [1,1,1,2,2,3], k = 2
Output: [1, 2]
```

**Optimal (Heap)**: HashMap for counts + Min-heap of size K, O(n log k) time
**Alternative**: Bucket sort by frequency, O(n) time

**Walkthrough (Heap approach)**:
```
nums = [1,1,1,2,2,3], k = 2

Step 1: Build frequency map
  freq = {1: 3, 2: 2, 3: 1}

Step 2: Use min-heap of size K
  Process (1, 3): heap = [(3, 1)]          size < k
  Process (2, 2): heap = [(2, 2), (3, 1)]  size == k
  Process (3, 1): 1 < heap_min(2)          skip

  Result: [2, 1] (elements from heap)
```

**Walkthrough (Bucket Sort approach)**:
```
nums = [1,1,1,2,2,3], k = 2

Step 1: freq = {1: 3, 2: 2, 3: 1}

Step 2: Create buckets indexed by frequency
  bucket[1] = [3]
  bucket[2] = [2]
  bucket[3] = [1]

Step 3: Walk from highest bucket down, collect k elements
  bucket[3]: [1]  -> collected 1
  bucket[2]: [2]  -> collected 2

  Result: [1, 2]
```

**Follow-ups**:
- What if k changes frequently?
- What if elements arrive as a stream?
- What if you need the k LEAST frequent elements?

---

### 5. K Closest Points to Origin

**Statement**: Given an array of `points` where `points[i] = [xi, yi]` represents a point on the plane, return the `k` closest points to the origin `(0, 0)`.

**Example**:
```
Input: points = [[1,3],[-2,2],[5,1]], k = 2
Output: [[1,3],[-2,2]]
Explanation: Distances: sqrt(10), sqrt(8), sqrt(26)
```

**Optimal**: Max-heap of size K (by distance), O(n log k) time, O(k) space
**Alternative**: Quickselect by distance, O(n) average time

**Walkthrough**:
```
points = [[1,3],[-2,2],[5,1]], k = 2

Distances (squared, to avoid sqrt):
  [1,3]:  1+9 = 10
  [-2,2]: 4+4 = 8
  [5,1]:  25+1 = 26

Max-heap of size K=2 (negate distance for max-heap in Python):
  Process (10, [1,3]):  heap = [(-10, [1,3])]
  Process (8, [-2,2]):  heap = [(-10, [1,3]), (-8, [-2,2])]
  Process (26, [5,1]):  26 > |heap_max|=10 -> skip (farther away)

  Result: [[1,3], [-2,2]]

Wait — we want CLOSEST, so we use a MAX-heap as gatekeeper:
  Keep the K smallest distances by removing the largest when heap is full.

  Process (10, [1,3]):  heap = [(-10, [1,3])]   size < k
  Process (8, [-2,2]):  heap = [(-10, [1,3]), (-8, [-2,2])]  size == k
  Process (26, [5,1]):  26 > 10 (max in heap) -> skip

  Result: [[1,3], [-2,2]]
```

**Follow-ups**:
- What if points arrive as a stream?
- What if you need the k FARTHEST points?
- Can you solve it without computing exact distances? (Squared distances work)

---

## Hard Problems

### 6. Task Scheduler

**Statement**: Given a char array `tasks` representing CPU tasks and a non-negative integer `n` representing the cooldown between two same tasks, return the minimum number of intervals the CPU takes to finish all tasks.

**Example**:
```
Input: tasks = ["A","A","A","B","B","B"], n = 2
Output: 8
Explanation: A -> B -> idle -> A -> B -> idle -> A -> B
```

**Optimal**: Max-heap + Queue for cooldown, O(N * n) time
**Alternative**: Math formula based on most frequent task

**Walkthrough (Heap + Queue approach)**:
```
tasks = [A,A,A,B,B,B], n = 2

Frequency: {A: 3, B: 3}
Max-heap: [-3, -3] (A and B both have count 3)
Cooldown queue: [] (stores (count, available_time))
time = 0

time=0: Pop -3 (A, count 3). Execute A. Push (-2, time+n+1=3) to queue.
  heap=[-3], queue=[(-2, 3)]

time=1: Pop -3 (B, count 3). Execute B. Push (-2, 4) to queue.
  heap=[], queue=[(-2, 3), (-2, 4)]

time=2: Heap empty. Queue front available at 3. Idle.
  queue=[(-2, 3), (-2, 4)]

time=3: Queue front (-2, 3) is ready. Push -2 back to heap.
  Pop -2 (A, count 2). Execute A. Push (-1, 6) to queue.
  heap=[], queue=[(-2, 4), (-1, 6)]

time=4: Queue front (-2, 4) is ready. Push -2 back to heap.
  Pop -2 (B, count 2). Execute B. Push (-1, 7) to queue.
  heap=[], queue=[(-1, 6), (-1, 7)]

time=5: Idle.

time=6: Queue front (-1, 6) ready. Push to heap.
  Pop -1 (A, count 1). Execute A. Count becomes 0, don't re-queue.
  heap=[], queue=[(-1, 7)]

time=7: Queue front (-1, 7) ready. Push to heap.
  Pop -1 (B, count 1). Execute B. Count becomes 0.
  heap=[], queue=[]

Total time = 8
Schedule: A B _ A B _ A B
```

**Walkthrough (Math approach)**:
```
tasks = [A,A,A,B,B,B], n = 2

Most frequent count = 3 (both A and B)
Number of tasks with max frequency = 2

Formula: max(len(tasks), (max_freq - 1) * (n + 1) + count_of_max_freq)
       = max(6, (3-1) * (2+1) + 2)
       = max(6, 6 + 2)
       = max(6, 8) = 8

Intuition: Create (max_freq - 1) blocks of size (n+1), plus a final partial block.
  [A B _] [A B _] [A B]
   block1  block2  final
```

**Follow-ups**:
- What if tasks have different execution times?
- What if you want to minimize idle time instead of total time?
- What if there are dependencies between tasks?

---

## Sample Session Flows

### Session Flow 1: Candidate Struggles (Easy Path)

**Interviewer**: "Imagine you're building a hospital emergency room system. Patients arrive in random order but need to be treated by severity. What data structure helps?"
**Candidate**: "A sorted list?"
**Interviewer**: "That works but inserting into a sorted list is O(n). There's a structure that gives you the highest-priority item in O(log n). It's called a heap — specifically a max-heap here. Let me show you how it works..."

*[Draw ASCII heap, explain insertion/extraction. Start with Kth Largest.]*

**Interviewer**: "Given [3, 2, 1, 5, 6, 4] and k=2, find the 2nd largest. What's the brute force?"
**Candidate**: "Sort and pick index n-2?"
**Interviewer**: "Right! O(n log n). But what if I told you to only track the top 2 elements? What size heap would you need?"

*[Guide to min-heap of size K]*

### Session Flow 2: Candidate Excels (Hard Path)

**Interviewer**: "You're building a real-time analytics dashboard. How would you compute the median response time as requests stream in?"
**Candidate**: "Two heaps — max-heap for the lower half, min-heap for the upper half."
**Interviewer**: "Exactly. Code it up. And then tell me — what happens when you have a billion data points? Does this still work?"

*[After they solve it, move to Task Scheduler or Sliding Window Median.]*

### Session Flow 3: Average Candidate (Standard Path)

**Interviewer**: "Twitter needs top 10 trending hashtags from 50 million. How would you approach it?"
**Candidate**: "Sort by frequency and take the top 10?"
**Interviewer**: "What's the time complexity?"
**Candidate**: "O(n log n)."
**Interviewer**: "Can we do better, given that K=10 is tiny compared to 50 million?"

*[Guide to min-heap of size K, then code Kth Largest. If time permits, move to Merge K Sorted Lists.]*
