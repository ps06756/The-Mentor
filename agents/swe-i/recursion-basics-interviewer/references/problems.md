# Problem Bank: Recursion & Backtracking Basics

## Easy Problems

### 1. Fibonacci Number

**Statement**: Given `n`, return the n-th Fibonacci number. F(0) = 0, F(1) = 1, F(n) = F(n-1) + F(n-2).

**Example**:
```
Input: n = 5
Output: 5
```

**Naive Recursive**: O(2^n) time, O(n) space (call stack)
**Memoized**: O(n) time, O(n) space
**Iterative**: O(n) time, O(1) space

**Call Stack Walkthrough (fib(4))**:
```
fib(4)
  fib(3)
    fib(2)
      fib(1) -> 1  [base]
      fib(0) -> 0  [base]
    return 1
    fib(1) -> 1    [base]
  return 2
  fib(2)
    fib(1) -> 1    [base]
    fib(0) -> 0    [base]
  return 1
return 3
```

**Follow-ups**:
- Derive O(2^n) from the recursion tree. How does memoization reduce it to O(n)?
- Convert to iterative with O(1) space.

---

### 2. Factorial

**Statement**: Given non-negative integer `n`, compute n! recursively.

**Example**:
```
Input: n = 5
Output: 120
```

**Complexity**: O(n) time, O(n) space (call stack)

**Call Stack Walkthrough**:
```
factorial(5)
  5 * factorial(4)
    4 * factorial(3)
      3 * factorial(2)
        2 * factorial(1)
          return 1       [base case]
        return 2
      return 6
    return 24
  return 120
```

**Tail Recursive Version**:
```python
def factorial_tail(n, acc=1):
    if n <= 1:
        return acc
    return factorial_tail(n - 1, n * acc)
# factorial_tail(5,1) -> (4,5) -> (3,20) -> (2,60) -> (1,120) -> 120
```

**Follow-ups**:
- What happens with factorial(-1)? How would you guard against it?
- Does Python optimize tail calls? (No, but Scheme does.)

---

### 3. Generate Parentheses

**Statement**: Given `n` pairs of parentheses, generate all valid combinations.

**Example**:
```
Input: n = 3
Output: ["((()))","(()())","(())()","()(())","()()()"]
```

**Approach**: Backtracking, O(4^n / sqrt(n)) time (Catalan number)

**Decision Tree (n=2)**:
```
                    ""
                    |
                   "("
                  /    \
           "(("          "()"
            |                |
         "(()"            "()("
            |                |
        "(())"           "()()"
        [VALID]          [VALID]

Rules: add "(" if open < n; add ")" if close < open.
```

**Solution**:
```python
def generateParenthesis(n):
    result = []
    def backtrack(cur, open_c, close_c):
        if len(cur) == 2 * n:
            result.append(cur)
            return
        if open_c < n:
            backtrack(cur + "(", open_c + 1, close_c)
        if close_c < open_c:
            backtrack(cur + ")", open_c, close_c + 1)
    backtrack("", 0, 0)
    return result
```

**Follow-up**: How many valid combinations for n pairs? (Catalan number: C(2n,n)/(n+1))

---

### 4. Subsets (Power Set)

**Statement**: Given an integer array `nums` of unique elements, return all possible subsets.

**Example**:
```
Input: nums = [1, 2, 3]
Output: [[], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]]
```

**Approach**: Backtracking, O(n * 2^n) time and space

**Decision Tree**:
```
                      []
             /         |        \
          [1]         [2]      [3]
         /   \         |
      [1,2] [1,3]   [2,3]
        |
     [1,2,3]

Each node is added to results. Recurse with start = i + 1.
```

**Solution**:
```python
def subsets(nums):
    result = []
    def backtrack(start, current):
        result.append(current[:])
        for i in range(start, len(nums)):
            current.append(nums[i])
            backtrack(i + 1, current)
            current.pop()      # undo choice
    backtrack(0, [])
    return result
```

**Follow-ups**:
- What if nums has duplicates? (Sort first, skip consecutive duplicates.)
- Solve iteratively using bit manipulation.

---

### 5. Permutations

**Statement**: Given array `nums` of distinct integers, return all permutations.

**Example**:
```
Input: nums = [1, 2, 3]
Output: [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

**Approach**: Backtracking, O(n! * n) time

**Decision Tree**:
```
                      []
            /          |          \
         [1]          [2]         [3]
        /   \        /   \       /   \
    [1,2] [1,3]  [2,1] [2,3] [3,1] [3,2]
      |     |       |     |     |     |
  [1,2,3][1,3,2][2,1,3][2,3,1][3,1,2][3,2,1]

Unlike subsets, every element must appear exactly once.
```

**Solution**:
```python
def permute(nums):
    result = []
    def backtrack(current, remaining):
        if not remaining:
            result.append(current[:])
            return
        for i in range(len(remaining)):
            current.append(remaining[i])
            backtrack(current, remaining[:i] + remaining[i+1:])
            current.pop()
    backtrack([], nums)
    return result
```

**Follow-ups**:
- Subsets tree vs permutations tree: what is structurally different?
- Handle duplicates with sort + skip at the same recursion level.

---

### 6. Letter Combinations of a Phone Number

**Statement**: Given digits 2-9, return all letter combinations (phone keypad mapping).

**Example**:
```
Input: digits = "23"
Output: ["ad","ae","af","bd","be","bf","cd","ce","cf"]
Mapping: 2->"abc", 3->"def", ..., 7->"pqrs", 9->"wxyz"
```

**Approach**: Backtracking, O(4^n) time

**Decision Tree**:
```
digits = "23"
              ""
         /    |    \
       "a"   "b"   "c"        <- digit "2"
      /|\   /|\    /|\
    ad ae af bd be bf cd ce cf <- digit "3"
```

**Solution**:
```python
def letterCombinations(digits):
    if not digits:
        return []
    phone = {"2":"abc","3":"def","4":"ghi","5":"jkl",
             "6":"mno","7":"pqrs","8":"tuv","9":"wxyz"}
    result = []
    def backtrack(i, cur):
        if i == len(digits):
            result.append(cur)
            return
        for ch in phone[digits[i]]:
            backtrack(i + 1, cur + ch)
    backtrack(0, "")
    return result
```

---

## Sample Session Flows

### Session Flow 1: Candidate Struggles (Easy Path)

**Interviewer**: "In your own words, what is recursion?"
**Candidate**: "A function calling itself, but I don't really get when to use it."
**Interviewer**: "Let's make it concrete. 5! = 5 * 4!. And 4! = 4 * 3!. Same shape, just smaller."

*[Draw call stack for factorial(4), then move to Fibonacci]*

**Candidate**: "So fib(n) = fib(n-1) + fib(n-2), and fib(0)=0, fib(1)=1?"
**Interviewer**: "Exactly. Let me draw the call tree for fib(4) and we'll trace it together."

### Session Flow 2: Candidate Excels (Hard Path)

**Candidate**: "Recursion solves a problem via smaller instances of itself. Base case terminates it."
**Interviewer**: "What's the time complexity of naive recursive Fibonacci?"
**Candidate**: "O(2^n). With memoization, O(n)."
**Interviewer**: "Let's skip to backtracking. Generate all valid parentheses for n pairs."

*[Candidate writes solution, interviewer asks them to trace the decision tree]*

### Session Flow 3: Average Candidate (Standard Path)

**Interviewer**: "Write a recursive Fibonacci function."
**Candidate**: *[Writes correct naive solution]*
**Interviewer**: "Trace fib(4). Do you see fib(2) computed more than once?"
**Candidate**: "Maybe store the results?"
**Interviewer**: "That's memoization. Let's add it, then try Generate Parentheses."
