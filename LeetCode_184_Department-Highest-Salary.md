# \#\# 📝 LeetCode 184: Department Highest Salary

  * **Date Solved:** September 23, 2025
  * **Link:** [Department Highest Salary](https://leetcode.com/problems/department-highest-salary/)

-----

### \#\# 🎯 Problem Summary

The goal is to find employees who have the highest salary in their respective departments. This problem can have multiple employees with the same top salary in a single department, and all should be returned.

-----

### \#\# 🧠 Key Concepts Tested

This problem is a classic test of "top-N-per-group" logic. It can be solved in several ways, each highlighting different SQL concepts:

  * **`JOIN`s:** To combine the `Employee` and `Department` tables.
  * **Subqueries:** Using a subquery in the `WHERE` clause to find the max salary for each department.
  * **Window Functions:** Using `RANK()` or `DENSE_RANK()` to rank employees by salary within each department. This is the modern, preferred approach.
  * **Common Table Expressions (CTEs):** To improve the readability of the window function solution.

-----

### \#\# ✅ Solution Approaches

#### \#\#\# Approach 1: Subquery in a `WHERE IN` Clause

This is a very intuitive approach. The logic is to first find the maximum salary for each department and then select the employees whose salary and department match these maximums.

```sql
SELECT
    d.name AS Department,
    e.name AS Employee,
    e.salary AS Salary
FROM
    Employee e
JOIN
    Department d ON e.departmentId = d.id
WHERE
    (e.departmentId, e.salary) IN (
        SELECT
            departmentId,
            MAX(salary)
        FROM
            Employee
        GROUP BY
            departmentId
    );
```

  * **Pros:** Easy to understand the logic.
  * **Cons:** Can be less performant on very large datasets compared to window functions.

#### \#\#\# Approach 2: Window Function (`DENSE_RANK`) with a CTE (Optimal Solution)

This is the most efficient and scalable solution. We use a window function to rank employees by salary within each department, then simply select everyone with a rank of 1.

```sql
WITH RankedSalaries AS (
    SELECT
        e.name AS employee_name,
        e.salary,
        d.name AS department_name,
        DENSE_RANK() OVER (PARTITION BY d.name ORDER BY e.salary DESC) as salary_rank
    FROM
        Employee e
    JOIN
        Department d ON e.departmentId = d.id
)
SELECT
    department_name AS Department,
    employee_name AS Employee,
    salary AS Salary
FROM
    RankedSalaries
WHERE
    salary_rank = 1;

```

  * **Why `DENSE_RANK()`?** Both `RANK()` and `DENSE_RANK()` work perfectly here. They correctly handle ties by giving the same rank to employees with the same salary. `DENSE_RANK()` is a great habit for ranking problems.
  * **Pros:** Highly readable, excellent performance, and easily adaptable for more complex "top-N" problems (e.g., "top 3 salaries").

-----

### \#\# 💡 Key Takeaways & Interview Prep Notes

1.  **Window Functions are Your Best Friend:** For any "per-group" ranking or ordering problem (top salary per department, second most recent order per customer, etc.), a window function is almost always the best answer. Be ready to explain **`PARTITION BY`** (how to group the data) and **`ORDER BY`** (how to rank within the group).

2.  **Know Multiple Approaches:** While the window function is optimal, being able to also write the `WHERE IN` subquery solution shows depth of knowledge. You can discuss the trade-offs in an interview, highlighting that the window function is generally superior in performance and scalability.

3.  **Handling Ties is Critical:** The problem requires returning all employees who share the top salary. This is where `RANK()` or `DENSE_RANK()` shine, as they are designed to handle ties gracefully. A solution using `ROW_NUMBER()` would be incorrect because it would arbitrarily pick only one of the tied employees.
