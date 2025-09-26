## 📝 LeetCode 262: Trips and Users

  * **Date Solved:** September 25, 2025
  * **Link:** [Trips and Users](https://leetcode.com/problems/trips-and-users/)

-----

### \#\# 🎯 Problem Summary

The goal is to calculate the daily cancellation rate for a specific date range ('2013-10-01' to '2013-10-03'). A key condition is that we must **exclude** any trips where **either the client or the driver** is a banned user. The final rate should be rounded to two decimal places.

-----

### \#\# 🧠 Key Concepts Tested

This is a fantastic, multi-step problem that mirrors real-world analytical tasks. It tests several important concepts:

  * **Conditional Aggregation:** This is the core pattern. It involves using a `CASE` statement inside an aggregate function (`SUM()` or `COUNT()`) to calculate different metrics within the same `GROUP BY` clause.
  * **Multiple `JOIN`s to the Same Table:** The requirement to check both client and driver status forces us to `JOIN` the `Users` table twice, using different aliases (`u_client`, `u_driver`) to distinguish them.
  * **`GROUP BY`:** Essential for aggregating the results for each distinct day.
  * **Data Type Handling:** The calculation of a rate requires careful handling of integer vs. floating-point division to avoid incorrect results (e.g., `5 / 10 = 0`).

-----

### \#\# ✅ Solution Approach

The most logical way to solve this is to build the query incrementally: join, filter, group, and then calculate.

```sql
-- Final query with all steps combined
SELECT
    t.request_at AS "Day",
    -- Step 4: Calculate the rate using the two aggregates,
    -- round to 2 decimal places, and handle integer division.
    ROUND(
        SUM(CASE WHEN t.status LIKE 'cancelled%' THEN 1 ELSE 0 END) * 1.0
        /
        COUNT(*)
    , 2) AS "Cancellation Rate"
FROM
    Trips t
-- Step 1: Join to Users table once for client info
JOIN
    Users u_client ON t.client_id = u_client.users_id
-- Step 1: Join to Users table a second time for driver info
JOIN
    Users u_driver ON t.driver_id = u_driver.users_id
WHERE
    -- Step 2: Filter out all trips with banned users (client OR driver)
    u_client.banned = 'No' AND u_driver.banned = 'No'
    -- Step 2: Filter for the specified date range
    AND t.request_at BETWEEN '2013-10-01' AND '2013-10-03'
-- Step 3: Group the results by day to perform daily calculations
GROUP BY
    t.request_at;
```

  * **Logic:** The query first creates a clean dataset by joining the tables and filtering out all irrelevant trips. Then, for each day, it calculates the total trips (`COUNT(*)`) and the canceled trips (`SUM(CASE WHEN...)`) and divides them to get the rate.

-----

### \#\# 💡 Key Takeaways & Interview Prep Notes

1.  **Conditional Aggregation is a Superpower:** This pattern is essential for any data role. When an interviewer asks you to calculate rates, percentages, or pivot data, think of `SUM(CASE WHEN...)`. It is far more efficient than multiple subqueries.
2.  **Joining a Table Twice is a Standard Pattern:** Don't be intimidated by joining to the same table multiple times. It's the standard way to handle records that have relationships with two or more other records of the same type (e.g., `client_id` and `driver_id` in `Trips`, or `sender_id` and `receiver_id` in a `Messages` table). The key is to use clear aliases.
3.  **Watch for Integer Division:** This is a classic SQL "gotcha." When dividing two integers, many SQL engines will return an integer. Multiplying the numerator by `1.0` or `CAST`ing it to a decimal/float is a simple, robust way to ensure you get a correct floating-point result.
4.  **Build Complex Queries in Steps:** The best way to tackle a daunting query is to build it incrementally. Start with your `JOIN`s, then add `WHERE` filters, then your `GROUP BY`, and finally, perfect the calculations in your `SELECT`. Verify your output at each step.
