---
title: LeetCode48:Rotate Image(Medium)
published: 2025-04-03
tags: [LeetCode]
category: LeetCode
draft: false
---

# LeetCode 48. Rotate Image (Medium)

## 📝 Problem Statement

You are given an `n x n` 2D matrix representing an image, rotate the image by 90 degrees (clockwise).

You have to rotate the image **in-place**, which means you have to modify the input 2D matrix directly. **DO NOT** allocate another 2D matrix and do the rotation.

### Example 1:

**Input:**

```
matrix = [[1,2,3],
          [4,5,6],
          [7,8,9]]
```

**Output:**

```
[[7,4,1],
 [8,5,2],
 [9,6,3]]
```

### Example 2:

**Input:**

```
matrix = [[5,1,9,11],
          [2,4,8,10],
          [13,3,6,7],
          [15,14,12,16]]
```

**Output:**

```
[[15,13,2,5],
 [14,3,4,1],
 [12,6,8,9],
 [16,7,10,11]]
```

### Constraints:

- `n == matrix.length == matrix[i].length`
- `1 <= n <= 20`
- `-1000 <= matrix[i][j] <= 1000`

---

## 💡 Intuition & Approach

To rotate the matrix by 90 degrees clockwise **in-place**, we can use two steps:

### Step 1: Transpose the Matrix

Swap `matrix[i][j]` with `matrix[j][i]`. This converts rows to columns.

### Step 2: Reverse Each Row

After transposing, reversing each row gives the final 90-degree rotation.

### Why does this work?

Rotation 90° clockwise means:  
Element at position `(i, j)` → `(j, n - 1 - i)`

Transposing gives `(j, i)`, and then reversing row `j` moves the element to `(j, n - 1 - i)` — exactly what we want.

---

## ✅ Python Code

```python
def rotate(matrix):
    n = len(matrix)

    # Step 1: Transpose
    for i in range(n):
        for j in range(i + 1, n):
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]

    # Step 2: Reverse each row
    for row in matrix:
        row.reverse()
```

---

## 🧠 Complexity Analysis

- **Time Complexity:** O(n²)
- **Space Complexity:** O(1) — in-place rotation

---

## 🗣 Interview Tip

If asked why this approach works, you can say:

> “Rotating 90° clockwise is equivalent to transposing the matrix and then reversing each row. Transpose turns rows into columns, and reversing aligns the elements into their rotated positions.”

Alternatively, if you forget this method, you can describe rotating four elements at a time from each layer (top-left ↔ top-right ↔ bottom-right ↔ bottom-left), moving layer by layer inward.

This shows you understand both the logic and the implementation trade-offs.
