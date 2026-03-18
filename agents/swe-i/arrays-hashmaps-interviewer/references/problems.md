# Problem Bank: Arrays & HashMaps

## Easy Problems

### 1. Two Sum

**Statement**: Given `nums` and `target`, return indices of two elements that add to target.

**Example**:
```
Input: nums = [2,7,11,15], target = 9
Output: [0,1]
Explanation: nums[0] + nums[1] = 2 + 7 = 9
```

**Optimal**: HashMap, O(n) time, O(n) space
**Brute Force**: Nested loops, O(n^2) time, O(1) space

**Walkthrough**:
```
nums = [2, 7, 11, 15], target = 9
map = {}

Step 1: num = 2, complement = 9 - 2 = 7
  7 not in map -> map = {2: 0}

Step 2: num = 7, complement = 9 - 7 = 2
  2 IS in map at index 0 -> return [0, 1]
```

**Follow-ups**:
- What if the array is sorted? (Two pointers, O(n) time, O(1) space)
- What if you need to return the values instead of indices?
- What if there are multiple valid pairs?

---

### 2. Contains Duplicate

**Statement**: Return true if any value appears at least twice.

**Optimal**: HashSet, O(n) time, O(n) space
**Alternative**: Sort then check adjacent, O(n log n) time, O(1) space

**Walkthrough**:
```
nums = [1, 2, 3, 1]
seen = set()

Step 1: 1 not in seen -> seen = {1}
Step 2: 2 not in seen -> seen = {1, 2}
Step 3: 3 not in seen -> seen = {1, 2, 3}
Step 4: 1 IS in seen -> return True
```

**Follow-up**: Can you do it with O(1) extra space without modifying the array? (Very hard - requires Pigeonhole Principle for integers in range)

---

### 3. Valid Anagram

**Statement**: Given two strings, determine if one is an anagram of the other.

**Optimal**: HashMap frequency count, O(n) time, O(1) space (max 26 for lowercase letters)
**Alternative**: Sort and compare, O(n log n) time

**Walkthrough**:
```
s = "anagram", t = "nagaram"

Count s: {a:3, n:1, g:1, r:1, m:1}
Count t: {n:1, a:3, g:1, r:1, m:1}

Maps are equal -> return True
```

**Optimized single-pass approach**:
```
s = "anagram", t = "nagaram"
count = {}

For each char in s: count[char] += 1
For each char in t: count[char] -= 1
If all values in count == 0: return True
```

**Follow-up**: What if strings contain Unicode characters? (HashMap still works, but space is no longer O(1))

---

### 4. Two Sum II (Sorted Array)

**Statement**: Array is sorted in ascending order. Find two numbers that add to target.

**Optimal**: Two pointers from both ends, O(n) time, O(1) space
**Why it works**: If sum is too small, need larger number (move left). If sum too large, need smaller (move right).

**Walkthrough**:
```
nums = [2, 7, 11, 15], target = 9
left = 0, right = 3

Step 1: nums[0] + nums[3] = 2 + 15 = 17 > 9 -> move right
left = 0, right = 2

Step 2: nums[0] + nums[2] = 2 + 11 = 13 > 9 -> move right
left = 0, right = 1

Step 3: nums[0] + nums[1] = 2 + 7 = 9 == 9 -> return [0, 1]
```

---

## Medium Problems

### 5. Group Anagrams

**Statement**: Given array of strings, group anagrams together.

**Optimal**: HashMap with sorted string as key, O(n * k log k) where k is max string length
**Alternative**: HashMap with character count tuple as key, O(n * k)

**Example**:
```
Input: ["eat","tea","tan","ate","nat","bat"]
Output: [["bat"],["nat","tan"],["ate","eat","tea"]]
```

**Walkthrough**:
```
map = {}

"eat" -> sorted = "aet" -> map = {"aet": ["eat"]}
"tea" -> sorted = "aet" -> map = {"aet": ["eat", "tea"]}
"tan" -> sorted = "ant" -> map = {"aet": ["eat", "tea"], "ant": ["tan"]}
"ate" -> sorted = "aet" -> map = {"aet": ["eat", "tea", "ate"], "ant": ["tan"]}
"nat" -> sorted = "ant" -> map = {"aet": ["eat", "tea", "ate"], "ant": ["tan", "nat"]}
"bat" -> sorted = "abt" -> map = {"aet": ["eat", "tea", "ate"], "ant": ["tan", "nat"], "abt": ["bat"]}

Return values: [["eat", "tea", "ate"], ["tan", "nat"], ["bat"]]
```

---

### 6. Top K Frequent Elements

**Statement**: Given integer array and integer k, return k most frequent elements.

**Optimal**: HashMap for counts + Min Heap of size k, O(n log k)
**Alternative**: Bucket sort by frequency, O(n)

**Walkthrough (Bucket Sort approach)**:
```
nums = [1,1,1,2,2,3], k = 2

Step 1: Count frequencies
  count = {1: 3, 2: 2, 3: 1}

Step 2: Create frequency buckets (index = frequency)
  buckets[0] = []
  buckets[1] = [3]
  buckets[2] = [2]
  buckets[3] = [1]
  buckets[4] = []
  buckets[5] = []
  buckets[6] = []

Step 3: Walk buckets from highest to lowest, collect k elements
  buckets[3] -> [1]  (collected 1)
  buckets[2] -> [2]  (collected 2)
  k reached -> return [1, 2]
```

**Follow-up**: What if k is close to n? What if elements are unique?

---

### 7. Product of Array Except Self

**Statement**: Return array where each element is product of all other elements (without division).

**Optimal**: Two-pass approach with prefix/suffix products, O(n) time, O(1) extra space (excluding output)

**Walkthrough**:
```
Input:  [1, 2, 3, 4]

Pass 1 (Left to Right - Prefix Products):
  output[0] = 1         (nothing to the left)
  output[1] = 1         (product of elements to the left: 1)
  output[2] = 1 * 2 = 2 (product of elements to the left: 1*2)
  output[3] = 2 * 3 = 6 (product of elements to the left: 1*2*3)
  output = [1, 1, 2, 6]

Pass 2 (Right to Left - Multiply by Suffix Products):
  suffix = 1
  output[3] = 6 * 1 = 6,  suffix = 1 * 4 = 4
  output[2] = 2 * 4 = 8,  suffix = 4 * 3 = 12
  output[1] = 1 * 12 = 12, suffix = 12 * 2 = 24
  output[0] = 1 * 24 = 24, suffix = 24 * 1 = 24
  output = [24, 12, 8, 6]

Verification:
  [24, 12, 8, 6]
  24 = 2*3*4, 12 = 1*3*4, 8 = 1*2*4, 6 = 1*2*3  Correct!
```

---

## Advanced Problems (For candidates who ace everything)

### 8. Longest Consecutive Sequence

**Statement**: Given an unsorted array of integers, find the length of the longest consecutive elements sequence. Must run in O(n) time.

**Optimal**: HashSet, O(n) time, O(n) space

**Walkthrough**:
```
nums = [100, 4, 200, 1, 3, 2]
num_set = {100, 4, 200, 1, 3, 2}

For each num, only start counting if (num - 1) is NOT in set:
  100: 99 not in set -> start! 100 (length 1)
  4: 3 IS in set -> skip (not the start of a sequence)
  200: 199 not in set -> start! 200 (length 1)
  1: 0 not in set -> start! 1, 2, 3, 4 (length 4)
  3: 2 IS in set -> skip
  2: 1 IS in set -> skip

Max length = 4
```

### 9. Minimum Window Substring

**Statement**: Given strings s and t, find the minimum window in s which contains all characters of t.

**Optimal**: Sliding window + HashMap, O(n) time

**Walkthrough**:
```
s = "ADOBECODEBANC", t = "ABC"
need = {A:1, B:1, C:1}, have = 0, need_count = 3

Expand right pointer:
  A -> need[A] met, have=1
  D -> skip
  O -> skip
  B -> need[B] met, have=2
  E -> skip
  C -> need[C] met, have=3  -> Window "ADOBEC" (len=6)

Shrink left pointer:
  Remove A -> have=2, window invalid

Continue expanding...
Eventually find "BANC" (len=4) -> minimum window
```

---

## Sample Session Flows

### Session Flow 1: Candidate Struggles (Easy Path)

**Interviewer**: "Let's start. What's the time complexity of accessing array[5]?"
**Candidate**: "O(n)?"
**Interviewer**: "Close! Array access by index is actually O(1) - constant time. The array knows exactly where index 5 is in memory. Let me explain..."

*[Switch to Contains Duplicate - simplest problem]*

**Interviewer**: "Given [1, 2, 3, 1], does any value appear twice?"
**Candidate**: "I'd use two loops..."
**Interviewer**: "That works! What's the time complexity?"
**Candidate**: "O(n^2)?"
**Interviewer**: "Right. Can we do better? What if we had a way to instantly check if we've seen a number before?"

*[Guide to HashSet solution]*

### Session Flow 2: Candidate Excels (Hard Path)

**Interviewer**: "What's the time complexity of HashMap lookup?"
**Candidate**: "Amortized O(1), but O(n) worst case with hash collisions."
**Interviewer**: "Impressive! You mentioned collisions - how are they typically resolved?"

*[Skip to Product of Array Except Self]*

**Interviewer**: "Can you solve this without division?"
**Candidate**: *[Quickly arrives at prefix/suffix approach]*
**Interviewer**: "Great. Follow-up: what if some elements are zero? And can you do it with O(1) extra space?"

*[Challenge with Minimum Window Substring if time permits]*

### Session Flow 3: Average Candidate (Standard Path)

**Interviewer**: "What's the time complexity of accessing an element in an array by index?"
**Candidate**: "O(1)."
**Interviewer**: "Perfect. When would you choose a HashMap over an array?"
**Candidate**: "When I need to look things up by a key that isn't a number."

*[Start with Two Sum]*

**Interviewer**: "Given an array [2, 7, 11, 15] and target 9, find two numbers that sum to 9. What approach comes to mind?"
**Candidate**: "I could check every pair..."
**Interviewer**: "That's a valid brute force. What's the time complexity?"
**Candidate**: "O(n^2)."
**Interviewer**: "Can we do better? For each number, what are you looking for?"
**Candidate**: "The complement... target minus the current number."
**Interviewer**: "And how can you quickly check if that complement exists?"

*[Guide toward HashMap solution, then move to Group Anagrams]*
