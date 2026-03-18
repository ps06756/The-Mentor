# Problem Bank: Stacks, Queues & Monotonic Patterns

## Easy Problems

### 1. Valid Parentheses

**Statement**: Given a string containing just '(', ')', '{', '}', '[' and ']', determine if the input string is valid. An input string is valid if open brackets are closed by the same type of brackets in the correct order.

**Example**:
```
Input: s = "({[]})"
Output: true

Input: s = "([)]"
Output: false
```

**Optimal**: Stack, O(n) time, O(n) space
**Brute Force**: Repeatedly remove innermost pairs until empty, O(n^2) time

**Walkthrough**:
```
s = "({[]})"
stack = []

Step 1: '(' -> push    stack = ['(']
Step 2: '{' -> push    stack = ['(', '{']
Step 3: '[' -> push    stack = ['(', '{', '[']
Step 4: ']' -> matches '[', pop  stack = ['(', '{']
Step 5: '}' -> matches '{', pop  stack = ['(']
Step 6: ')' -> matches '(', pop  stack = []

Stack empty -> return True
```

**Follow-ups**:
- What if you only had one type of bracket? (Counter instead of stack)
- What if there were wildcard characters that could be any bracket?
- What is the minimum number of brackets to add to make the string valid?

---

### 2. Min Stack

**Statement**: Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.

**Example**:
```
MinStack()
push(-2)  -> stack: [-2], min_stack: [-2]
push(0)   -> stack: [-2, 0], min_stack: [-2]
push(-3)  -> stack: [-2, 0, -3], min_stack: [-2, -3]
getMin()  -> return -3
pop()     -> stack: [-2, 0], min_stack: [-2]
top()     -> return 0
getMin()  -> return -2
```

**Optimal**: Two stacks (main + min tracker), O(1) for all operations, O(n) space
**Alternative**: Store (value, current_min) tuples in a single stack

**Walkthrough**:
```
Two-stack approach:

push(-2): stack = [-2], min_stack = [-2]     min = -2
push(0):  stack = [-2, 0], min_stack = [-2]  min = -2 (0 > -2, don't push to min)
push(-3): stack = [-2, 0, -3], min_stack = [-2, -3]  min = -3

Wait - optimization: only push to min_stack if value <= current min

getMin(): return min_stack[-1] = -3
pop():    -3 == min_stack[-1], so pop both stacks
          stack = [-2, 0], min_stack = [-2]
getMin(): return min_stack[-1] = -2
```

**Follow-up**: Can you implement this with O(1) extra space? (Hint: store encoded values)

---

## Medium Problems

### 3. Daily Temperatures

**Statement**: Given an array of integers `temperatures`, return an array `answer` such that `answer[i]` is the number of days you have to wait after the ith day to get a warmer temperature. If there is no future day for which this is possible, keep `answer[i] == 0`.

**Example**:
```
Input: temperatures = [73, 74, 75, 71, 69, 72, 76, 73]
Output: [1, 1, 4, 2, 1, 1, 0, 0]
```

**Optimal**: Monotonic decreasing stack, O(n) time, O(n) space
**Brute Force**: For each day, scan forward, O(n^2) time

**Walkthrough**:
```
temps = [73, 74, 75, 71, 69, 72, 76, 73]
stack = []  (stores indices)
answer = [0, 0, 0, 0, 0, 0, 0, 0]

i=0 (73): stack empty, push 0
  stack = [0]

i=1 (74): 74 > temps[0]=73, pop 0, answer[0] = 1-0 = 1
  push 1. stack = [1]

i=2 (75): 75 > temps[1]=74, pop 1, answer[1] = 2-1 = 1
  push 2. stack = [2]

i=3 (71): 71 < temps[2]=75, push 3
  stack = [2, 3]

i=4 (69): 69 < temps[3]=71, push 4
  stack = [2, 3, 4]

i=5 (72): 72 > temps[4]=69, pop 4, answer[4] = 5-4 = 1
          72 > temps[3]=71, pop 3, answer[3] = 5-3 = 2
          72 < temps[2]=75, stop. push 5
  stack = [2, 5]

i=6 (76): 76 > temps[5]=72, pop 5, answer[5] = 6-5 = 1
          76 > temps[2]=75, pop 2, answer[2] = 6-2 = 4
          push 6. stack = [6]

i=7 (73): 73 < temps[6]=76, push 7
  stack = [6, 7]

Remaining indices in stack get 0 (no warmer day).
answer = [1, 1, 4, 2, 1, 1, 0, 0]
```

**Key Insight**: Each element is pushed and popped at most once, so total work is O(n) despite the inner while loop.

**Follow-ups**:
- What if the array is circular (last day wraps to first)?
- What if you needed the next cooler day instead?
- Can you solve it with O(1) extra space? (Working backwards)

---

### 4. Basic Calculator

**Statement**: Implement a basic calculator to evaluate a simple expression string. The expression may contain '+', '-', '(', ')', digits, and spaces.

**Example**:
```
Input: s = "(1+(4+5+2)-3)+(6+8)"
Output: 23
```

**Optimal**: Stack for parentheses, O(n) time, O(n) space

**Walkthrough**:
```
s = "1 + (2 - 3)"
result = 0, sign = 1, stack = []

'1' -> result = 0 + 1*1 = 1
'+' -> sign = 1
'(' -> push result (1) and sign (1) to stack
       stack = [1, 1]
       reset: result = 0, sign = 1
'2' -> result = 0 + 1*2 = 2
'-' -> sign = -1
'3' -> result = 2 + (-1)*3 = -1
')' -> pop sign_before = 1, result_before = 1
       result = result_before + sign_before * result
       result = 1 + 1 * (-1) = 0

Final answer: 0
```

**Extended walkthrough with nested parentheses**:
```
s = "(1+(4+5+2)-3)"
result=0, sign=1, stack=[]

'(' -> push (0, 1), result=0, sign=1
'1' -> result=1
'+' -> sign=1
'(' -> push (1, 1), result=0, sign=1
'4' -> result=4
'+' -> sign=1
'5' -> result=9
'+' -> sign=1
'2' -> result=11
')' -> pop (1, 1): result = 1 + 1*11 = 12
'-' -> sign=-1
'3' -> result = 12 + (-1)*3 = 9
')' -> pop (0, 1): result = 0 + 1*9 = 9

But actual answer for "(1+(4+5+2)-3)+(6+8)" = 23
Full trace would continue with "+(6+8)" portion.
```

**Follow-ups**:
- What if the expression includes '*' and '/'? (Operator precedence)
- What if there are unary minus operators like "-(3+2)"?

---

### 5. Implement Queue using Stacks

**Statement**: Implement a first-in-first-out (FIFO) queue using only two stacks. The queue should support push, pop, peek, and empty.

**Example**:
```
MyQueue()
push(1)  -> input_stack: [1], output_stack: []
push(2)  -> input_stack: [1, 2], output_stack: []
peek()   -> move to output: input_stack: [], output_stack: [2, 1]
            return 1
pop()    -> output_stack: [2], return 1
empty()  -> false
```

**Optimal**: Two stacks, amortized O(1) per operation
**Key Insight**: Only transfer from input to output stack when output is empty

**Walkthrough**:
```
Two stacks: input_stack (for pushes), output_stack (for pops)

push(1): input = [1], output = []
push(2): input = [1, 2], output = []
push(3): input = [1, 2, 3], output = []

peek():
  output is empty, so transfer all from input:
  pop 3 from input, push to output: input = [1, 2], output = [3]
  pop 2 from input, push to output: input = [1], output = [3, 2]
  pop 1 from input, push to output: input = [], output = [3, 2, 1]
  return output top = 1

pop():
  output not empty, pop from output: output = [3, 2], return 1

push(4): input = [4], output = [3, 2]

pop():
  output not empty, pop from output: output = [3], return 2

pop():
  output not empty, pop from output: output = [], return 3

pop():
  output empty, transfer from input:
  output = [4], input = []
  pop from output: output = [], return 4
```

**Amortized Analysis**: Each element is moved at most twice (once into input, once into output), so n operations cost O(n) total = O(1) amortized per operation.

**Follow-ups**:
- Can you implement a stack using two queues?
- What is the worst-case time for a single pop operation?

---

## Advanced Problems

### 6. Sliding Window Maximum

**Statement**: Given an array `nums` and a sliding window of size `k`, return the maximum value in each window as it slides from left to right.

**Example**:
```
Input: nums = [1, 3, -1, -3, 5, 3, 6, 7], k = 3
Output: [3, 3, 5, 5, 6, 7]

Window positions:
[1  3  -1] -3  5  3  6  7  -> max = 3
 1 [3  -1  -3] 5  3  6  7  -> max = 3
 1  3 [-1  -3  5] 3  6  7  -> max = 5
 1  3  -1 [-3  5  3] 6  7  -> max = 5
 1  3  -1  -3 [5  3  6] 7  -> max = 6
 1  3  -1  -3  5 [3  6  7] -> max = 7
```

**Optimal**: Monotonic deque (decreasing), O(n) time, O(k) space
**Brute Force**: For each window, scan for max, O(n*k) time

**Walkthrough**:
```
nums = [1, 3, -1, -3, 5, 3, 6, 7], k = 3
deque = []  (stores indices, front is always the max of current window)
result = []

i=0 (1): deque empty, append 0
  deque = [0]

i=1 (3): 3 > nums[0]=1, pop 0 from back. Append 1.
  deque = [1]

i=2 (-1): -1 < nums[1]=3, append 2
  deque = [1, 2]
  Window [0..2] formed. Max = nums[deque[0]] = nums[1] = 3
  result = [3]

i=3 (-3): -3 < nums[2]=-1, append 3
  deque = [1, 2, 3]
  Check front: index 1 >= 3-3+1=1? Yes, still in window.
  Max = nums[1] = 3
  result = [3, 3]

i=4 (5): 5 > nums[3]=-3, pop 3
         5 > nums[2]=-1, pop 2
         5 > nums[1]=3, pop 1
  Append 4. deque = [4]
  Max = nums[4] = 5
  result = [3, 3, 5]

i=5 (3): 3 < nums[4]=5, append 5
  deque = [4, 5]
  Max = nums[4] = 5
  result = [3, 3, 5, 5]

i=6 (6): 6 > nums[5]=3, pop 5
         6 > nums[4]=5, pop 4
  Append 6. deque = [6]
  Max = nums[6] = 6
  result = [3, 3, 5, 5, 6]

i=7 (7): 7 > nums[6]=6, pop 6
  Append 7. deque = [7]
  Max = nums[7] = 7
  result = [3, 3, 5, 5, 6, 7]
```

**Key Insight**: The deque maintains a decreasing order. The front is always the index of the maximum in the current window. We remove from the front if it falls outside the window, and remove from the back if the new element is larger.

**Follow-ups**:
- What if you needed the sliding window minimum instead?
- What if the window size changes dynamically?
- Can you solve this using a heap? What's the time complexity difference?

---

## Sample Session Flows

### Session Flow 1: Candidate Struggles (Easy Path)

**Interviewer**: "Think of a stack of plates at a buffet. Which plate do you grab first?"
**Candidate**: "The one on top."
**Interviewer**: "Right — last one placed, first one grabbed. That's LIFO. Now, given the string '([)]', is this valid? Walk me through it using that plate analogy."
**Candidate**: "I'd push '(', then '[', then try to match ')' with '['... that doesn't match."
**Interviewer**: "Exactly! That mismatch tells us it's invalid. Let's code Valid Parentheses."

*[Stay at Easy level — guide through Valid Parentheses step by step]*

### Session Flow 2: Candidate Excels (Hard Path)

**Interviewer**: "What data structure powers your browser's back button?"
**Candidate**: "A stack. And the forward button is a second stack — when you go back, the page gets pushed onto the forward stack."
**Interviewer**: "Excellent — that's a classic two-stack pattern. Let's skip ahead. Given daily temperatures, how would you find how many days until a warmer day?"

*[Candidate quickly identifies monotonic stack. Move to Sliding Window Maximum.]*

**Interviewer**: "Now the window slides and you need the max at each position. A heap works, but can you do O(n)?"

### Session Flow 3: Average Candidate (Standard Path)

**Interviewer**: "What's the difference between a stack and a queue?"
**Candidate**: "Stack is LIFO, queue is FIFO."
**Interviewer**: "Perfect. When would you use each?"
**Candidate**: "Stack for undo operations, queue for processing things in order."

*[Start with Valid Parentheses]*

**Interviewer**: "Given '({[]})', is it valid? Don't code yet — just tell me what you'd push and pop."
**Candidate**: *[Traces through correctly]*
**Interviewer**: "Great! Now code it up. After that, let's try something harder — Daily Temperatures."

*[If time permits, introduce monotonic stack concept with Daily Temperatures]*
