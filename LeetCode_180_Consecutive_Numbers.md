## 📝 LeetCode 180: Consecutive Numbers

  * **Date Solved:** September 25, 2025
  * **Link:** [Consecutive Numbers](https://leetcode.com/problems/consecutive-numbers/)

-----

### \#\# 🎯 Problem Summary

The goal is to find all numbers that appear at least three times in a row in the `Logs` table. The "consecutive" nature is determined by the `id` column.

-----

### \#\# 🧠 Key Concepts Tested

This problem is a classic test of **sequence analysis**. The key is to compare a row's value with the values in the rows immediately preceding it.

  * **Window Functions (`LAG()`/`LEAD()`):** These are the ideal tools for this job. They allow you to "peek" at previous or subsequent rows without using a costly self-join.
  * **Common Table Expressions (CTEs):** Used to create a temporary, readable dataset that includes the lagged values, which can then be easily filtered.
  * **Self-Joins:** An alternative, more traditional (and often less performant) method where you join the table to itself to align consecutive rows.
  * **`DISTINCT`:** Necessary to ensure the final output lists each qualifying number only once, even if it appears in a sequence longer than three (e.g., four or five times in a row).

-----

### \#\# ✅ Solution Approaches

#### \#\#\# Approach 1: Window Function (`LAG`) with a CTE (Optimal Solution)

This is the most efficient, readable, and scalable solution. The logic is to create two new columns representing the numbers from the previous one and two rows. Then, we simply filter for rows where all three numbers are the same.

```sql
WITH LaggedLogs AS (
    SELECT
        num,
        LAG(num, 1) OVER (ORDER BY id) AS lag1,
        LAG(num, 2) OVER (ORDER BY id) AS lag2
    FROM
        Logs
)
SELECT DISTINCT
    num AS ConsecutiveNums
FROM
    LaggedLogs
WHERE
    num = lag1 AND num = lag2;
```

  * **Pros:** Excellent performance (one scan over the table), easy to read, and simple to adapt for longer sequences (e.g., five in a row by adding `lag3` and `lag4`).
  * **Cons:** Requires understanding of window functions.

#### \#\#\# Approach 2: Multiple Self-Joins

This method works by joining the table to itself twice to line up three consecutive rows.

```sql
SELECT DISTINCT
    l1.num AS ConsecutiveNums
FROM
    Logs l1
JOIN
    Logs l2 ON l1.id = l2.id - 1
JOIN
    Logs l3 ON l1.id = l3.id - 2
WHERE
    l1.num = l2.num AND l2.num = l3.num;
```

  * **Pros:** Solves the problem using only basic `JOIN` logic.
  * **Cons:** Can be very slow on large tables due to the multiple joins. The logic is also less intuitive and harder to maintain compared to the window function approach.

-----

### \#\# 💡 Key Takeaways & Interview Prep Notes

1.  **`LAG()` and `LEAD()` are for Sequences:** When you see a problem that involves comparing a row to its neighbors (streaks, consecutive events, changes over time), your first thought should be `LAG()` or `LEAD()`. This is a very common and powerful pattern.

2.  **Compare Window Functions to Self-Joins:** This problem is a perfect example to discuss trade-offs in an interview. You can explain that while a self-join works, a window function is generally superior because it avoids the high cost of repeatedly joining a table to itself and represents a more modern, analytical approach to SQL.

3.  **Use CTEs to Build Logic Step-by-Step:** Don't try to solve everything at once. Use a CTE to prepare the data (e.g., adding the lagged columns) first. Then, use a simple final `SELECT` to query the prepared data. This makes your code clean, readable, and much easier to debug.
