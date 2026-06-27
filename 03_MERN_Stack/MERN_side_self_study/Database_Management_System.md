# MERN SELF STUDY (DBMS)

## Question: 
Types of database management system (DBMS), what each type is suitable for. Be able to explain the difference between relational DBMS and NoSQL (of a different kind)

## Answer:

Database management systems organize, store, and retrieve digital data efficiently.

**DBMS Types and Suitability:**

    • Relational (RDBMS): Uses structured tables with rows and columns. Best for complex queries and transactional apps (e-commerce, banking).

    • Document-oriented (NoSQL): Stores data in JSON-like documents. Best for flexible schemas and rapid development (content management).

    • Key-Value (NoSQL): Stores data as unique keys and values. Best for caching and high-speed data retrieval (user sessions).

    • Graph (NoSQL): Uses nodes and edges to map relationships. Best for interconnected networks (social media maps, fraud detection).

**Relational vs. NoSQL (Document)**

    • RDBMS enforces strict, predefined schemas and uses SQL.

    • NoSQL (Document) allows dynamic schemas with nested, unstructured data.

    • RDBMS scales vertically by upgrading hardware power.

    • NoSQL scales horizontally by distributing data across servers.

**Relational vs. NoSQL: A Key Differentiation**

The core distinction between Relational and NoSQL DBMS lies in their data models, schema flexibility, scalability, query languages, and ACID compliance.

**1. Data Model:**

    • RDBMSs enforce a rigid structure with predefined schemas in tables.

    • NoSQL offers more flexibility with various data models like document-oriented, key-value, and graph.

**2. Schema Flexibility:**

    • RDBMSs demand a predefined, fixed schema, posing challenges for evolving data.

    • NoSQL allows for dynamic schema changes and embraces unstructured or semi-structured data.

**3. Scalability:**

    • RDBMSs traditionally struggle with horizontal scaling for massive data volumes.

    • NoSQL excels in horizontal scalability, making it perfect for distributed architectures.

**4. Query Language:**

    • RDBMSs leverage SQL (Structured Query Language) for powerful querying with complex joins and aggregations.

    • NoSQL may have its own query languages, sometimes with limitations compared to SQL.

**5. ACID Compliance:**

    • RDBMSs guarantee ACID properties for transactions.

    • NoSQL compliance varies, with some sacrificing certain ACID aspects for better scalability and performance.

By grasping these distinctions, organizations can make informed decisions when selecting a DBMS. The optimal choice ensures exceptional performance, scalability, and data management capabilities, empowering them to thrive in today's data-driven world.