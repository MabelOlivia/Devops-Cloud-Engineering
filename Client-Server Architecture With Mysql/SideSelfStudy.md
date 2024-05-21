
# Ping and Traceroute:

#### Ping:
- **Purpose**: Ping is a networking utility used to test the reachability of a host on an Internet Protocol (IP) network.
- **How it works**: Ping sends ICMP echo request packets to the target host and waits for replies. The time taken for the round trip and any packet loss are measured.
- **Example Usage**:
  ```bash
  ping google.com
  ```
- **Interpretation**:
  - **Response Time (Round-Trip Time)**: 
    ```
    Round-trip min/avg/max = 20.2/30.5/50.8 ms
    ```
  - **Packet Loss**:
    ```
    0% packet loss
    ```

<img width="351" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/097f934d-2dce-4941-91a4-1ee0717cb2b2">


#### Traceroute:
- **Purpose**: Traceroute is used to trace the route that packets take to reach a destination.
- **How it works**: Traceroute sends packets with incrementally increasing TTL (Time To Live) values. Each router along the path decrements the TTL value, and when it reaches 0, the router discards the packet and sends an ICMP TTL Exceeded message back to the sender.
- **Example Usage**:
  ```bash
  traceroute google.com
  ```
- **Interpretation**:
  - **Hop-by-Hop Analysis**:
    ```
    1  192.168.1.1 (192.168.1.1)  1.067 ms  0.757 ms  0.869 ms
    2  10.20.30.1 (10.20.30.1)  10.231 ms  10.335 ms  10.164 ms
    ...
    10  108.170.251.129 (108.170.251.129)  23.824 ms  23.748 ms  23.761 ms
    ```
  - **Identifying Network Issues**:
    ```
    6   55.66.77.88  reports: Destination net unreachable
    ```
<img width="479" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/22ef2958-18c0-44fb-a972-f526cff097b8">


# SQL Basics:

#### SHOW:
- **Purpose**: SHOW is used to display various database objects or settings.
- **Explanation**:
  - `SHOW TABLES;`: Lists all tables in the current database.
  - `SHOW DATABASES;`: Lists all databases on the server.
  - `SHOW INDEX FROM table_name;`: Displays the indexes of a specific table.
- **Example Usage**:
  ```sql
  SHOW TABLES;
  SHOW DATABASES;
  SHOW INDEX FROM employees;
  ```

#### CREATE:
- **Purpose**: CREATE is used to create new database objects.
- **Explanation**:
  - `CREATE TABLE table_name (...)`: Creates a new table with specified columns and data types.
  - `CREATE DATABASE database_name;`: Creates a new database.
  - `CREATE INDEX index_name ON table_name (...);`: Creates an index on one or more columns of a table, which can improve query performance.
- **Example Usage**:
  ```sql
  CREATE TABLE employees (
      id INT PRIMARY KEY,
      name VARCHAR(50),
      age INT
  );
  
  CREATE DATABASE company;
  
  CREATE INDEX idx_name ON employees (name);
  ```

#### DROP:
- **Purpose**: DROP is used to delete database objects.
- **Explanation**:
  - `DROP TABLE table_name;`: Deletes a table and its data from the database.
  - `DROP DATABASE database_name;`: Deletes a database and all its tables and data.
  - `DROP INDEX index_name ON table_name;`: Deletes an index from a table.
- **Example Usage**:
  ```sql
  DROP TABLE employees;
  DROP DATABASE company;
  DROP INDEX idx_name ON employees;
  ```

#### SELECT:
- **Purpose**: SELECT is used to retrieve data from a database.
- **Explanation**:
  - `SELECT columns FROM table_name WHERE condition;`: Retrieves data from one or more columns of a table based on specified conditions.
  - `SELECT * FROM table_name;`: Retrieves all columns and rows from a table.
- **Example Usage**:
  ```sql
  SELECT name, age FROM employees;
  SELECT * FROM employees WHERE age > 30;
  ```

#### INSERT:
- **Purpose**: INSERT is used to add new records to a table.
- **Explanation**:
  - `INSERT INTO table_name (...) VALUES (...);`: Inserts new records into a table with specified values for each column.
- **Example Usage**:
  ```sql
  INSERT INTO employees (id, name, age) VALUES (1, 'John Doe', 35);
  ```

### Additional Tips:
- **Data Manipulation Language (DML)**: SQL commands like SELECT, INSERT, UPDATE, and DELETE are used to manipulate data in the database.
- **Data Definition Language (DDL)**: SQL commands like CREATE, DROP, ALTER, and TRUNCATE are used to define and manage database objects.
- **Referential Integrity**: Enforcing constraints like FOREIGN KEY ensures data consistency between related tables.
- **Normalization**: Organizing data into multiple tables to minimize redundancy and improve data integrity.
- **SQL Joins**: Combining data from multiple tables using JOIN operations like INNER JOIN, LEFT JOIN, and RIGHT JOIN.

Now with the explanations and code snippets combined, you have a comprehensive understanding of SQL basics and how to execute each command.
