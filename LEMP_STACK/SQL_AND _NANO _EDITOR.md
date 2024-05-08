
# SQL

<img width="350" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/3b402062-2390-45d7-a487-d1fefa5ea34b">

SQL (Structured Query Language) is a standardized programming language used for managing and manipulating relational databases. 

Summary:

SQL provides a set of commands for querying, updating, and managing relational databases. It allows users to interact with databases to perform tasks such as retrieving data, inserting records, updating existing data, and deleting records. SQL is widely used across industries for handling data-related tasks efficiently and securely.

SQL syntax consists of a set of rules and conventions for writing SQL statements to interact with databases. Here's a brief overview of the SQL syntax:

1\. **Keywords**: SQL statements are composed of keywords like SELECT, INSERT INTO, UPDATE, DELETE FROM, CREATE TABLE, ALTER TABLE, DROP TABLE, etc. These keywords indicate the type of operation to be performed.

2\. **Clauses**: SQL statements typically consist of various clauses, each serving a specific purpose. Common clauses include:

   - *SELECT*: Specifies the columns to be retrieved from a table.

   - *FROM* : Specifies the table from which to retrieve data.

   - *WHERE*: Filters rows based on specified conditions.

   - *GROUP BY*: Groups rows that have the same values into summary rows.

   - *HAVING*: Filters groups based on specified conditions.

   - *ORDER BY*: Sorts the result set based on specified columns.

   - *LIMIT OFFSET*: Limits the number of rows returned by a query.

   - *INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL JOIN*: Specifies how tables should be joined together.

Here's an example of a simple SQL SELECT statement:

```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

This statement selects specific columns (`column1`, `column2`) from a table (`table\_name`) based on a condition specified in the `WHERE` clause.

Here's an overview of some commonly used SQL commands and their syntax

1\. **SELECT**: Retrieves data from the "employees" table.

```sql
SELECT employee_id, first_name, last_name FROM employees WHERE department = 'HR';
```

2\. **INSERT INTO**: Adds a new record to the "customers" table.

````sql
INSERT INTO customers (name, email) VALUES ('John Doe', 'john@example.com');
````

3\. **UPDATE**: Modifies existing records in the "products" table.

````sql
UPDATE products SET price = 20.99 WHERE category = 'Electronics';
````

4\. **DELETE**: Removes records from the "orders" table.

````sql
DELETE FROM orders WHERE status = 'Cancelled';
````

5\. **CREATE TABLE**: Creates a new table named "students".

````sql
CREATE TABLE students (
       student_id INT PRIMARY KEY,

       first_name VARCHAR(50),

       last_name VARCHAR(50),

       age INT
   );
````

6\. **ALTER TABLE**: Adds a new column named "address" to the "customers" table.

````sql
ALTER TABLE customers ADD address VARCHAR(100);
````

7\. **DROP TABLE**: Deletes the "temp_data" table.

````sql
DROP TABLE temp_data;
````

8\. **CREATE INDEX**: Creates an index on the "username" column of the "users" table.

````sql
CREATE INDEX idx_username ON users (username);
````

9\. **DROP INDEX**: Deletes the index named "idx_email" from the "customers" table.

````sql
DROP INDEX idx_email;
````

10\. **JOIN**: Combines data from the "orders" and "customers" tables.

````sql
SELECT orders.order_id, customers.name 
FROM orders INNER JOIN customers ON orders.customer_id = customers.customer_id;
````

11\. **GROUP BY**: Groups orders by their status from the "orders" table.

````sql
SELECT status, COUNT(*) 
    FROM orders 
    GROUP BY status;
````

12\. **HAVING**: Filters orders with more than 5 items from the "orders" table.

````sql
SELECT customer_id, COUNT(*) as order_count 
    FROM orders 
    GROUP BY customer_id 
    HAVING COUNT(*) > 5;
````

# Nano Editor

Nano is a simple and user-friendly text editor commonly found on Unix-based systems. It offers basic text editing functionalities and is ideal for simple tasks such as editing configuration files, writing scripts, or taking quick notes. Nano features easy-to-remember keyboard shortcuts for tasks like saving files, searching for text, cutting and pasting, and navigating within the document. It provides a simple interface suitable for beginners and those who prefer simplicity over advanced features.


1\. **Opening a File:**

   `nano filename`

2\. **Saving a File:**

   - Press `Ctrl` + `O` (Write Out)
     Then press `Enter` to confirm the filename.

3\. **Exiting Nano:**

   - Press `Ctrl` + `X`.
     If there are unsaved changes, Nano will prompt you to save them.

4\. **Moving the Cursor:**

   - Use the arrow keys to move up, down, left, or right.

5\. **Cut, Copy, and Paste:**

   - `Ctrl` + `K`: Cut the current line.

   - `Ctrl` + `U`: Paste the cut text.

   - `Alt` + `6`: Copy the current line.

   - `Ctrl` + `Shift` + `6`: Paste the copied text.

6\. **Searching for Text:**

   - `Ctrl` + `W`: Search for text.

   - Enter the text to search and press `Enter`.

   - Use `Ctrl` + `W` to find the next occurrence.

7\. **Undo and Redo:**

   - `Ctrl` + `Shift` + `\_`: Undo.

   - `Alt` + `U`: Redo.

8\. **Delete:**

   - `Ctrl` + `D`: Delete the character under the cursor.

   - `Ctrl` + `K`: Delete the line after the cursor.

9\. **Inserting and Deleting:**

   - `Insert`: Toggle between insert and overwrite modes.

   - `Backspace` or `Delete`: Delete the character before or after the cursor, respectively.

10\. **Page Navigation:**

    - `Ctrl` + `V`: Move one page down.

    - `Ctrl` + `Y`: Move one page up.



