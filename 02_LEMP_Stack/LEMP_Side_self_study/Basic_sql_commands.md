# Basic SQL Syntax and Commonly Used Commands

Structured Query Language (SQL) is the standard programming language used to manage and manipulate relational databases. In DevOps, SQL is essential for database migrations, state checks, and application troubleshooting.

## Core SQL Syntax Rules
    • Keywords: Commands like SELECT are case-insensitive but written in uppercase by convention.
    • Semicolons: Each SQL statement ends with a semicolon ; to signal the end of the command.
    • Strings: Text values must be enclosed in single quotes (e.g., 'DevOps').

### Most Commonly Used SQL Commands

#### 1. Data Definition Language (DDL):  
DDL commands define and modify the database structure.  

    • CREATE DATABASE: Initializes a new database instance. For example, CREATE DATABASE steghub_db;  

    • CREATE TABLE: Generates a new table with defined columns and data types. For example, CREATE TABLE trainees (
    id INT PRIMARY KEY, name VARCHAR(50), track VARCHAR(30));  

    • ALTER TABLE: Modifies an existing table structure, such as adding a column. For example, ALTER TABLE trainees ADD email VARCHAR(100);  

    • DROP TABLE: Permanently deletes a table and all its data. For example, DROP TABLE trainees;  

#### 2. Data Manipulation Language (DML)  

DML commands manage the data stored within the database objects.  

    • INSERT INTO: Adds new rows of data to a table. For example, INSERT INTO trainees (id, name, track) VALUES (1, 'Alex Doe', 'DevOps');  

    • UPDATE: Modifies existing records in a table based on a condition. For example, UPDATE trainees SET track = 'SRE' WHERE id = 1;  

    • DELETE: Removes specific rows from a table. For example, DELETE FROM trainees WHERE id = 1;  
    
#### 3. Data Query Language (DQL)

DQL is used to fetch and organize data from the database.  

    • SELECT: Retrieves data from one or more tables. For example, For example, SELECT name, track FROM trainees;  

    • WHERE: Filters query results to match specific criteria. For example, SELECT * FROM trainees WHERE track = 'DevOps';  

    • ORDER BY: Sorts the result set in ascending (ASC) or descending (DESC) order. For example, SELECT * FROM trainees ORDER BY name ASC;  

    • JOIN: Combines rows from two or more tables based on a related column. For example, SELECT trainees.name, Projects.project_name FROM trainees INNER JOIN Projects ON trainees.id = Projects.trainee_id;

