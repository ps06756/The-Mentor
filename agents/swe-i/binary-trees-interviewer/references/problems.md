# Problem Bank: Binary Trees

## Easy Problems

### 1. Maximum Depth of Binary Tree

**Statement**: Given the root of a binary tree, return its maximum depth.

**Example**:
```
Input:       3
            / \
           9  20
             /  \
            15   7
Output: 3
```

**Optimal**: Recursive DFS, O(n) time, O(h) space. **Alt**: BFS level counting, O(n) time, O(w) space.

**Walkthrough**:
```
maxDepth(3) = 1 + max(maxDepth(9), maxDepth(20)) = 1 + max(1, 2) = 3
maxDepth(9) = 1, maxDepth(20) = 1 + max(1,1) = 2
```

**Follow-ups**: Iterative solution. Skewed vs balanced space complexity. Minimum depth variant.

---

### 2. Invert Binary Tree

**Statement**: Given the root, invert (mirror) the tree and return its root.

**Example**:
```
Input:      4          Output:     4
           / \                    / \
          2   7                  7   2
         / \ / \                / \ / \
        1  3 6  9              9  6 3  1
```

**Optimal**: Recursive DFS, O(n) time, O(h) space

**Walkthrough**: At each node swap left/right, then recurse into both. Visit 4->swap(2,7), then recurse.

**Follow-ups**: Iterative BFS approach. Verify mirrors without modifying.

---

### 3. Same Tree

**Statement**: Given roots of two binary trees, check if they are structurally identical with same values.

**Example**:
```
Tree p:  1      Tree q:  1      Output: True
        / \             / \
       2   3           2   3
```

**Optimal**: Recursive DFS, O(n) time, O(h) space

**Walkthrough**:
```
isSame(1,1): vals match -> check children
  isSame(2,2): vals match -> both children null -> True
  isSame(3,3): vals match -> both children null -> True
-> True
```

**Follow-ups**: Check if two trees are mirrors (symmetric). Handle duplicate values.

---

### 4. Subtree of Another Tree

**Statement**: Given `root` and `subRoot`, return true if `root` contains a subtree matching `subRoot`.

**Example**:
```
root:       3       subRoot:  4
           / \               / \
          4   5             1   2
         / \
        1   2
Output: True
```

**Optimal**: DFS + Same Tree check, O(m * n) time, O(h) space

**Walkthrough**:
```
isSubtree(3, sub): isSame(3,4)? No -> check left
  isSubtree(4, sub): isSame(4,4)? Check: 4==4, isSame(1,1) True, isSame(2,2) True -> True!
```

**Follow-ups**: O(m + n) via tree serialization and string matching.

---

## Medium Problems

### 5. Validate Binary Search Tree

**Statement**: Determine if a binary tree is a valid BST (left subtree values strictly less, right strictly greater).

**Example**:
```
Valid:      5       Invalid:    5
           / \                 / \
          3   7               3   7
         / \                 / \
        1   4               1   6    <- 6 > 5, violates BST
```

**Optimal**: Recursive with min/max bounds, O(n) time, O(h) space

**Walkthrough**:
```
isValid(5, -inf, inf) -> isValid(3, -inf, 5) -> isValid(1, -inf, 3): ok
                                               -> isValid(4, 3, 5): ok
                      -> isValid(7, 5, inf): ok -> True

Common mistake: only checking immediate children. Node 6 as child of 1
passes parent check (6>1) but fails ancestor check (in 5's LEFT subtree, yet 6>5).
```

**Follow-ups**: Iterative inorder approach. Handle duplicate values. Stack-based solution.

---

### 6. Binary Tree Level Order Traversal

**Statement**: Return values level by level, left to right.

**Example**:
```
Input:       3          Output: [[3], [9, 20], [15, 7]]
            / \
           9  20
             /  \
            15   7
```

**Optimal**: BFS with queue, O(n) time, O(w) space

**Walkthrough**:
```
Queue: [3]         -> Level 0: [3]
Queue: [9, 20]     -> Level 1: [9, 20]
Queue: [15, 7]     -> Level 2: [15, 7]

Key: use len(queue) at each step to process one full level.
```

**Follow-ups**: Zigzag order. Bottom-up level order. Right side view of tree.

---

### 7. Lowest Common Ancestor of a Binary Tree

**Statement**: Given two nodes p and q, find their lowest common ancestor.

**Example**:
```
            3
           / \
          5   1
         / \ / \
        6  2 0  8

LCA(5, 1) = 3     LCA(5, 4) = 5     LCA(6, 4) = 5
```

**Optimal**: Recursive DFS, O(n) time, O(h) space

**Walkthrough**:
```
LCA(6, 4): lca(3) -> lca(5) finds 6 on left, finds 4 via lca(2)->right
  Both sides non-null at node 5 -> return 5. Answer: 5
```

**Follow-ups**: BST variant (use BST property). Confirm both nodes exist first. Iterative approach.

---

## Advanced Problems

### 8. Binary Tree Maximum Path Sum

**Statement**: Find the maximum sum path (any node to any node, not necessarily through root).

**Example**:
```
       -10
       /  \
      9   20         Max path: 15 -> 20 -> 7 = 42
         /  \
        15   7
```

**Approach**: At each node, compute max one-sided gain for parent; update global max with bridge path.

---

### 9. Serialize and Deserialize Binary Tree

**Statement**: Convert a binary tree to a string and reconstruct it.

**Approach**: Preorder traversal with null markers: "1,2,N,N,3,4,N,N,5,N,N"

---

## Sample Session Flows

### Session Flow 1: Candidate Struggles

**Interviewer**: "Can you tell me what a binary tree is?"
**Candidate**: "Like a linked list but with two pointers?"
**Interviewer**: "Great intuition! Each node has up to two children."
*[Stay with Max Depth, guide toward recursion]*

### Session Flow 2: Candidate Excels

**Candidate**: "BST: left subtree less, right greater. Inorder gives sorted. Search is O(log n) balanced, O(n) skewed."
*[Skip to Validate BST, then LCA or Max Path Sum]*

### Session Flow 3: Average Candidate

**Interviewer**: "Walk me through the three DFS traversal orders."
**Candidate**: "Inorder: 4,2,5,1,3. Preorder: 1,2,4,5,3. Postorder: 4,5,2,3,1."
*[Guide through Invert Tree, then Validate BST]*
