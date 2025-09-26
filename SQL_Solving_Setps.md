## 📝 A Step-by-Step Method for Solving Complex SQL Queries

This guide breaks down any daunting SQL problem into a series of small, manageable, and logical steps. The core principle is to **build and verify your query incrementally**. Never try to write the entire query at once.

### \#\# Pre-Step: Understand and Visualize

Before you write a single line of code, make sure you understand the goal.

1.  **What is the final output supposed to look like?** Sketch it out. What are the column headers? What does a single row represent (e.g., a day, a customer, a product)?
2.  **What data do you need?** Identify the specific tables and columns required.
3.  **What are the conditions?** List the filtering logic (e.g., "only unbanned users," "within a specific date range").

-----

### \#\# Step 1: The Base (`FROM` and `JOIN`)

**Goal:** Gather all your raw materials into a single place.

Start by joining the tables you need. Don't filter or group yet. Use `SELECT *` to see the combined result and ensure your joins are working as expected.

**Mental Checklist:**

  * Do I need more than one table?
  * What are the correct keys to join them on?
  * Do I need to join the same table twice with different aliases?

**Example:**

```sql
SELECT
    *
FROM Trips t
JOIN Users u_client ON t.client_id = u_client.users_id
JOIN Users u_driver ON t.driver_id = u_driver.users_id;
```

-----

### \#\# Step 2: The Filter (`WHERE`)

**Goal:** Remove all the rows you don't care about.

Now that you have your combined data, add your `WHERE` clause to filter it down to the relevant subset. This is the most effective way to reduce the amount of data the database has to process in later steps.

**Mental Checklist:**

  * Are there date ranges to apply?
  * Are there specific statuses to include or exclude?
  * Are there conditions from multiple tables (like `u_client.banned = 'No'`)?

**Example:**

```sql
... -- (FROM and JOINs from Step 1)
WHERE
    u_client.banned = 'No'
    AND u_driver.banned = 'No'
    AND t.request_at BETWEEN '2013-10-01' AND '2013-10-03';
```

-----

### \#\# Step 3: The Structure (`GROUP BY`)

**Goal:** Define the final granularity of your output.

Look back at your visualization from the pre-step. What does each row represent? The columns you identify are what you need to `GROUP BY`. This collapses your filtered dataset into the final structure.

**Mental Checklist:**

  * Am I calculating something "per day," "per customer," "per department"?
  * The answer to that question is what goes in your `GROUP BY`.

**Example:**

```sql
... -- (FROM, JOINs, WHERE from previous steps)
GROUP BY
    t.request_at;
```

-----

### \#\# Step 4: The Calculation (`SELECT` with Aggregations)

**Goal:** Calculate the values for each group.

Now you can write your final `SELECT` list. This is where you'll use your aggregate functions (`COUNT`, `SUM`, `AVG`, etc.) and conditional logic (`CASE WHEN`) to perform the necessary calculations on each group you created in Step 3.

**Mental Checklist:**

  * What is the numerator? What is the denominator?
  * Do I need a simple `COUNT(*)` or a conditional `SUM(CASE WHEN ...)`?
  * Am I calculating multiple metrics for each group?

**Example:**

```sql
SELECT
    t.request_at,
    SUM(CASE WHEN t.status LIKE 'cancelled%' THEN 1 ELSE 0 END),
    COUNT(*)
... -- (Rest of the query)
```

-----

### \#\# Step 5: The Polish (Final Formatting)

**Goal:** Clean up the output to match the required format.

This is the final touch. Apply aliases, round numbers, and order the results to make the output clean, readable, and correct.

**Mental Checklist:**

  * Do the columns need specific names (`AS "Column Name"`)?
  * Do any numbers need to be rounded (`ROUND(..., 2)`)?
  * Do I need to handle integer division (`* 1.0`)?
  * Should the final result be sorted (`ORDER BY`)?

By following these five steps, you can deconstruct any SQL problem into a logical, verifiable process, which is the key to building confidence and accuracy.
