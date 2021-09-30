# Database System Note

Note to CMU 15-445 Database System

## Class 1 : Relation Model

**Relation Algebra**

- 𝜎 Select
- 𝛱 Projection
- ∪ Union
- ∩ Intersection
- − Difference
- × Product
- ⋈ Join

**Extra Operators of Relation Algebra:**

- 𝞺 Rename
- R<-S Assignment
- 𝝳 Duplicate Elimination
- 𝝪 Aggregation
- 𝛕 Sorting
- r÷S Division

## Class 2: Advanced SQL

Relational algebra not only define the data we want, but also defined the order we do query.

But we dont exactly need to define an specifc order. User oly needs to dpecify the answer that they want, not how to compute it. That why we need advanced 

The DBMS is responsible for efficent evaluation of the query:

- Query optimizer: re-orders operations and generatews query plan

**History of SQL:**

- "SEQUEL" (Structured English Query Language) from IBM's **System R** prototype
- Oracle Adopted SEQUEL and IBM's DB2 in 1983 supports SEQUEL, so it become de facto standard
- "SQL" (Structured Query Language) become Standard in 1986(ANSI) and 1987(ISO)

### Relational Languages:

- Data Definition Language (DDL)
  - 用于定义DB的三级结构，包括外模式、概念模式、内模式和互相之间的映像。定义数据的完整性、安全控制等约束
  - 如 CREATE ALTER DROP TRUNCATE COMMENT RENAME
- Data Manipulation Language (DML)
  - 由DBMS提供，让用户和程序员实现对数据库中数据的操作
  - SELECT INSERT UPDATE DELETE MERGE CALL (EXPLAIN PLAN) (LOCK TABLE) 等
- Data Control Language (DCL)
  - 权限控制
  - GRANT REVOKE

**Aggregates:**

Functions that return a singel value from a bag of tuples:

- AVG(col) : Return the average col value
- MIN(col) : Return the minimum col value
- MAX(col) : Return maximun col value
- SUM(col) : Return sum of value in col
- COUNT(col) : Return # of values for col

Eg. we can only select **GROUP BY ed** col if we used aggregating fuctions. We can filter aggregated col with **HAVING** but not WHERE.

``` SQL
SELECT AVE(s.gpa) as ave_gpa, e.cid
FROM enrolled AS e, student AS s
WHERE e.sid = s.sid
GROUP BY e.cid
HAVING avg_gpa > 3.9;
```

**String Operation:**

Only MySQL is Case **Insensitive**, Only MySQL/SQLite supports Double Quotes. All the other DB is Case Sensitive and using Single Only Quotes.

**Date/Time Operations:**

Date operation differs from different DB.

**Output Control:**

ORDER BY \<col\> [ASC|DESC]
LIMIT \<count\> [OFFSET \<count\>]

**Nested Queries:**

eg.

```
SELECT name FROM student WHERE sid IN (SELECT sid FROM enrolled);

# or

SELECT (SELECT S.name FROM student AS S WHERE S.sid = E.sid) AS sname
FROM enrolled AS E
WHERE cid = '15-445';
```
