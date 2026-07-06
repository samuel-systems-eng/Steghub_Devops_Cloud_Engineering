# SQL Documentation: Foundational SQL Commands

## Introduction

Structured Query Language (SQL) is the standard language used to manage and manipulate relational databases.  
This documentation provides a structured overview of essential SQL commands. It covers how to discover infrastructure, define structures, and perform core data operations.

## SQL Command Categories

To manage databases effectively, commands are grouped by their operational scope:

    • Data Definition Language (DDL): Commands that define, alter, or destroy the structure of database objects (e.g., CREATE, DROP).

    • Data Manipulation Language (DML): Commands used to manage and manipulate the actual data stored within those structures (e.g., SELECT, INSERT, UPDATE, DELETE).

### 1. System Exploration Commands 

**`(a) SHOW`**  

    • Category: Database Exploration (Sub-dialect specific, primarily MySQL/MariaDB)

    • Purpose: Displays structural metadata from the database server, such as available databases, tables, or column schemas.

**Syntax (sql)**

`SHOW DATABASES;`  
`SHOW TABLES;`  
`SHOW COLUMNS FROM table_name;`  

**Examples (sql)**

-- View all databases on the current server  
`SHOW DATABASES;`

-- View all tables inside the currently selected database   
`SHOW TABLES;`

-- Display column names, data types, and null constraints for a specific table  
`SHOW COLUMNS FROM employees;`

**`(b) USE`**

    • Category: Session Management

    • Purpose: Switches the active context to a specific database so subsequent commands execute against it.

**Syntax & Example (sql)**

`USE database_name;`  
`USE my_database;`  


### 2. Data Definition Language (DDL)

**`(a) CREATE`**

    • Category: DDL

    • Purpose: Establishes a brand-new database or a structurally defined table.

**Core Concepts to Know**  

    • Data Types: Tells the database what kind of data to expect (e.g., INT for integers, VARCHAR(N) for text up to N characters, DECIMAL(P, S) for precise monetary figures).

    • Constraints: Rules applied to columns to enforce data integrity (e.g., PRIMARY KEY guarantees uniqueness, NOT NULL prevents blank entries).  

**Syntax (sql)**

`CREATE DATABASE database_name;`

CREATE TABLE table_name (  
    column1 data_type constraint,  
    column2 data_type,
    ...  
);`

**Examples (sql)**

-- Create a blank database container  
`CREATE DATABASE my_database;`

-- Create an organized table layout with constraints  
`CREATE TABLE employees (  
    id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    position VARCHAR(50),
    salary DECIMAL(10, 2)
);`

**`(b) DROP`**

    • Category: DDL  

    • Purpose: Permanently deletes an entire database or table, including all underlying data. This action cannot be undone.

**Syntax (sql)**

`DROP DATABASE database_name;`
`DROP TABLE table_name;`

**Examples (sql)**

-- Deletes the database and everything inside it  
`DROP DATABASE my_database;`

-- Erases the table schema and its rows entirely  
`DROP TABLE employees;`


### 3. Data Manipulation Language (DML)

**`(a) SELECT`**

    • Category: DML

    • Purpose: Retrieves and displays records from one or more tables.

**Syntax (sql)**

`SELECT column1, column2, ...  
FROM table_name
WHERE condition;`

**Examples (sql)**

-- Retrieve every column and row from the table (Wildcard '*')  
`SELECT * FROM employees;`

-- Extract only specific columns to reduce data overhead  
`SELECT name, position FROM employees;`

-- Filter records out using conditional statements  
`SELECT name, salary FROM employees WHERE position = 'Manager';`

**`(b) INSERT`**

    • Category: DML

    • Purpose: Adds new data rows into an existing table structure.

**Syntax (sql)**

`INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);`

**Examples (sql)**

-- Insert a single, complete record  
`INSERT INTO employees (id, name, position, salary)
VALUES (1, 'John Doe', 'Manager', 75000.00);`

-- Batch insert multiple rows simultaneously  
`INSERT INTO employees (id, name, position, salary)
VALUES
(2, 'Jane Smith', 'Developer', 65000.00),
(3, 'Emily Johnson', 'Designer', 60000.00);`

**`(c) UPDATE`**

    • Category: DML

    • Purpose: Modifies existing data records within a table.

    • Critical Warning: Always include a WHERE clause. Omitting it will change the specified value across every single row in the table.

**Syntax (sql)**

`UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;`

**Example (sql)

-- Give John Doe a salary raise  
`UPDATE employees
SET salary = 80000.00
WHERE id = 1;`

**`(d) DELETE`**

    • Category: DML

    • Purpose: Removes specific rows from a table based on a condition.

    • Critical Warning: Always include a WHERE clause. Omitting it will wipe out all data in the table, though it leaves the table structure intact (unlike DROP).

**Syntax (sql)**

`DELETE FROM table_name
WHERE condition;`

**Example (sql)**

-- Remove a specific employee record safely  
`DELETE FROM employees
WHERE id = 3;`


**Command Reference Summary**

|**Command** |**Category** |**Target** |**Safe Without "WHERE" command**|  
-------------|-------------|-----------|-----------------------|
|SHOW	|Exploration	|Metadata |	Yes (N/A) |
|CREATE	|DDL	|Structures	|Yes (N/A)
|DROP |	DDL |	Structures	|Destructive (Deletes Table/DB)
|SELECT	|DML	|Data Rows	|Yes (Returns all rows)
|INSERT	|DML	|Data Rows	|Yes
|UPDATE	|DML	|Data Rows	|Dangerous (Alters all rows if omitted)
|DELETE	|DML	|Data Rows	|Dangerous (Trashes all rows if omitted)

