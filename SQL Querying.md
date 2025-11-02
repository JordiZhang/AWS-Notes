---

---
--- 
From LearnSQL the SQL basics course. Formatting conventions, CAPS aren't needed but are standard convention. Similarly number of spaces and line breaks doesn't matter and is mostly up to the user.
# Basics, Filtering and Sorting
- `SELECT`: Chooses rows to query. Use `AS` to give aliases to columns. Can also rename tables similarly by using `AS` in FROM or JOIN.
- `FROM`: From which database to query.
- `WHERE`: Conditionals for query, done before any grouping. Also for filtering.
- `ORDER BY`: Sorting queries, use `ASC` and `DESC`.
- `GROUP BY`: Groups query based on row/s.
- `HAVING`: Filters queries after grouping.
- `DISTINCT`: Removes duplicates.
- `LIMIT`: Limits the rows that are displayed. Used at the end of a query.

# Conditionals (From Mode)
We have used `WHERE` for certain simple conditions. What if we need if statements, we use `CASE`. `END` denotes end of the if statements. ELSE is optional.
```PostgreSQL
SELECT player_name, weight, 
	CASE WHEN weight > 250 THEN 'over 250' 
		WHEN weight > 200 THEN '201-250' 
		WHEN weight > 175 THEN '176-200' 
		ELSE '175 or under' END AS weight_group 
FROM benn.college_football_players
```
Since `COUNT` ignores nulls, you could use a `CASE` statement to evaluate the condition and produce null or non-null values depending on the outcome.
# Aggregation
- `COUNT()`: Counts number of rows.
- `SUM()`
- `MIN()`
- `MAX()`
- `AVG()`
# Joining
All `JOINS` are used with `ON`, which is the conditional for joining, for example: 
```PostgreSQL
-- It says PostgreSQL but this is standard SQL. PostgreSQL for better highlighting
SELECT *
FROM student
LEFT JOIN room
	ON student.room_id = room.id
```
- `JOIN`: Defaults to `INNER JOIN`. `AS` can be used to rename databases and joining same table twice.
- `INNER JOIN`: Joins tables, shows rows with matching columns. If one table has say room_id = NULL but the other table doesn't have any column matching it, ignores it. 
- `LEFT JOIN`: Returns all rows of Left (first) table and adds matching rows of the Right (second) one.
- `RIGHT JOIN`: Returns all rows of the Right (second) table and adds matching rows of the Left (first) one.
- `FULL JOIN`: Combines LEFT and RIGHT JOINS. Returns all rows of both tables and combines rows where there is a match. Some databases don't support this. Also called FULL OUTER JOIN sometimes?
- `NATURAL JOIN`: Joins two tables on the columns with same name. If two tables have a column named room_id, then NATURAL JOIN joins the tables at that column.

`LEFT`, `RIGHT` and `FULL` are actually also shortcuts for `OUTER JOINS`.
```PostgreSQL
SELECT *
FROM person
LEFT OUTER JOIN car
  ON person.id = car.owner_id;
```

# Subqueries
We can nest a query into another one. Can also be done using FROM ().
```PostgreSQL
SELECT * 
FROM trip
WHERE trip.price > (
  SELECT AVG(trip.price)
  FROM trip
);
```

- `IN`: Used with WHERE to specify a list of values. `WHERE` rating `IN` (1, 2, 3). Use this with subqueries that return multiple values.
- `ALL`
- `ANY`
- `EXISTS` ?

Correlated subqueries rule: subqueries can use tables from the main query, but the main query can't use tables from the subquery!
?

# Set Operations
- `UNION`: Combines 2 queries. By default it removes duplicate rows. Use UNION ALL to include them.
- `INTERSECT`: Shows rows that are present in both queries.
- `EXCEPT`: Shows first query except the rows that are also present in the second query. Some databases use `MINUS` instead.

# Challenge
--- 
The owner of the shop would like to see each customer's

- id (name the column `cus_id`).
- name (name the column `cus_name`).
- id of their latest purchase (name the column `latest_purchase_id`).
- the total quantity of all flowers purchased by the customer, in all purchases, not just the last purchase (name the column **all_items_purchased**).

Remember, you need not use all columns from all the tables here – choose them carefully.

My answer:
```PostgreSQL
-- That was hard.
SELECT 
	c.id AS cus_id,
	c.name AS cus_name,
	(SELECT MAX(p.id) 
  	 FROM purchase AS p
  	 WHERE p.customer_id = c.id) AS latest_purchase_id,
  	(SELECT SUM(i.quantity)
  	 FROM purchase_item AS i, purchase AS p
  	 WHERE i.purchase_id = p.id AND p.customer_id = c.id) AS all_items_purchased
FROM customer as c;
```

Official answer:
```PostgreSQL
SELECT
  c.id AS cus_id,
  c.name AS cus_name,
  (
    SELECT MAX(purchase.id)
    FROM purchase
    WHERE purchase.customer_id = c.id
  ) AS latest_purchase_id,
  (
    SELECT SUM(quantity)
    FROM purchase_item
    WHERE purchase_id IN ( -- Use of IN is much more readable than mine.
      SELECT id
      FROM purchase
      WHERE customer_id = c.id
    )
  ) AS all_items_purchased
FROM customer AS c;
```