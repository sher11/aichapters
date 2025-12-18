---
layout: chapter
title: SQL Fundamentals
course_id: databases
chapter_number: 1
---

**One-liner:** Structured Query Language for relational databases - essential for data retrieval, manipulation, and schema design.

## Core SQL Operations

### SELECT (Query Data)
```sql
-- Basic SELECT
SELECT column1, column2 FROM table_name WHERE condition;

-- Aggregate functions
SELECT COUNT(*), AVG(salary), MAX(age) 
FROM employees 
WHERE department = 'Engineering';

-- GROUP BY with HAVING
SELECT department, COUNT(*), AVG(salary)
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

### Joins

| Join Type | Returns | Use Case |
|-----------|---------|----------|
| INNER JOIN | Matching rows from both | Most common |
| LEFT JOIN | All from left + matching right | Keep all left records |
| RIGHT JOIN | All from right + matching left | Keep all right records |
| FULL OUTER JOIN | All from both | Keep all records |

```sql
-- INNER JOIN
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;

-- LEFT JOIN
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;
```

### Window Functions
```sql
-- ROW_NUMBER (assign rank)
SELECT name, salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- Partition by department
SELECT name, dept_id, salary,
    AVG(salary) OVER (PARTITION BY dept_id) AS dept_avg
FROM employees;
```

### Common Table Expressions (CTE)
```sql
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 150000
)
SELECT d.name, COUNT(*) 
FROM high_earners h
JOIN departments d ON h.dept_id = d.id
GROUP BY d.name;
```

## Data Manipulation

### INSERT, UPDATE, DELETE
```sql
-- INSERT
INSERT INTO employees (name, salary, dept_id)
VALUES ('Alice', 120000, 5);

-- UPDATE
UPDATE employees
SET salary = salary * 1.1
WHERE performance_rating = 'excellent';

-- DELETE
DELETE FROM employees WHERE status = 'terminated';
```

## Schema Design

### CREATE TABLE
```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    salary DECIMAL(10, 2) CHECK (salary > 0),
    dept_id INTEGER REFERENCES departments(id),
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Constraints
- **PRIMARY KEY:** Unique identifier, NOT NULL
- **FOREIGN KEY:** Referential integrity
- **UNIQUE:** No duplicates
- **NOT NULL:** Must have value
- **CHECK:** Custom validation
- **DEFAULT:** Default value

## Normalization

### Normal Forms
- **1NF:** Atomic values, unique rows
- **2NF:** 1NF + no partial dependencies
- **3NF:** 2NF + no transitive dependencies

### Denormalization Trade-offs
- **Normalized:** Less redundancy, update safety, complex joins
- **Denormalized:** Faster reads, data duplication

## High-Frequency Interview Problems

1. **Second Highest Salary** - Subquery with LIMIT
2. **Department Top 3 Salaries** - Window function DENSE_RANK
3. **Duplicate Emails** - GROUP BY + HAVING COUNT(*) > 1
4. **Nth Highest Salary** - OFFSET or window function
5. **Delete Duplicates** - Keep MIN(id) with GROUP BY

## Performance Tips (L6/E6)

- **Use indexes** on WHERE, JOIN, ORDER BY columns
- **Avoid SELECT *** - specify needed columns only
- **Limit results** before joins
- **Analyze execution plans** (EXPLAIN)
- **N+1 queries:** Use joins instead of loops

## Common Gotchas

- **NULL behavior:** NULL != NULL (use IS NULL)
- **COUNT(*)** vs **COUNT(column)** - latter excludes NULLs
- **HAVING** filters groups, **WHERE** filters rows
- **JOIN order** matters for performance

## Key Takeaways

- SELECT with WHERE, JOIN, GROUP BY is 80% of SQL
- Window functions for ranking and analytics
- CTEs improve readability
- Normalize for transactional, denormalize for analytics
- Index on query patterns, not all columns
- For L6/E6: Discuss N+1 queries, execution plans, trade-offs
