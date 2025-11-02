--- 
From Mode.com

# Managing Tables
- `CREATE TABLE`: Creates a table. 
- `INSERT`: Insert new rows/entries.
```PostgreSQL
INSERT INTO tablename
(column1, column3) -- choose columns we want to add data to, or leave blank for all
VALUES (val1, val3), -- Adds the values of the row we want to add.
	   (more1, more3) -- Can add more than 1 row at a time
```
- `UPDATE`: Used to change or update records. Used with `SET` to set the record to whatever is needed. Must use `WHERE` to specify which records to update, otherwise all are updated.
```PostgreSQL
UPDATE Customers  
SET ContactName = 'Alfred Schmidt', City= 'Frankfurt'  
WHERE CustomerID = 1;
```
- `DELETE`: Deletes. 
 ```PostgreSQL
DELETE FROM _table_name_ WHERE _condition_;
  ```
- `DROP`: Used to delete entire table. `DROP tablename`.
