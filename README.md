# Database System Note

Note to CMU 15-445 Database System

[Youtube to this Course](https://www.youtube.com/watch?v=1D81vXw2T_w&list=PLSE8ODhjZXjbohkNBWQs_otTrBTrjyohi&index=4)

[Chinese Translation Version](https://www.simtoco.com/#/albums/video?id=1000132)

## Class 1 : Relation Model

**Relation Algebra**

- ð Select
- ð± Projection
- âª Union
- â© Intersection
- â Difference
- Ã Product
- â Join

**Extra Operators of Relation Algebra:**

- ðº Rename
- R<-S Assignment
- ð³ Duplicate Elimination
- ðª Aggregation
- ð Sorting
- rÃ·S Division

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
  - ç¨äºå®ä¹DBçä¸çº§ç»æï¼åæ¬å¤æ¨¡å¼ãæ¦å¿µæ¨¡å¼ãåæ¨¡å¼åäºç¸ä¹é´çæ åãå®ä¹æ°æ®çå®æ´æ§ãå®å¨æ§å¶ç­çº¦æ
  - å¦ CREATE ALTER DROP TRUNCATE COMMENT RENAME
- Data Manipulation Language (DML)
  - ç±DBMSæä¾ï¼è®©ç¨æ·åç¨åºåå®ç°å¯¹æ°æ®åºä¸­æ°æ®çæä½
  - SELECT INSERT UPDATE DELETE MERGE CALL (EXPLAIN PLAN) (LOCK TABLE) ç­
- Data Control Language (DCL)
  - æéæ§å¶
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

``` SQL
SELECT name FROM student WHERE sid IN (SELECT sid FROM enrolled);

# or

SELECT (SELECT S.name FROM student AS S WHERE S.sid = E.sid) AS sname
FROM enrolled AS E
WHERE cid = '15-445';
```

## Class 3: Database Storage

**Different Level of Database System:**

- Query Planning
- Operator Execution
- Access Methods
- Buffer Pool Manager
- Disk Manager

**Disk-Oriented Architectur:** assumes that the primary storage location of database is on non-volatile disk. DBMS's components manage the movement of data between non-volatile and volatile storage.

**Time Cost of:**

- L1 Cache Ref: 0.5ns
- L2 Cache Ref: 7ns
- DRAM: 100ns
- SSD: 150,000ns
- HDD: 10,000,000ns
- Network Storage: 30,000,000ns

Some DB use **mmap** to use virtual memory (like levelDB LMDB). Some Partially use (like mongoDB SQLite). Moth main stream DB do not use (like MySQL, Oracle).

But, DBMS wants to control things itself and can do a better job (than OS) at it, like:

- Flushing ditry pages to disk in the correct order.
- Specialized prefetching.
- Buffer replacement plicy.
- Thread/process scheduling.

## For Database Storage

**Problem 1:** How the DBMS represents the database in files on disk.

**Problem 2:** How the DBMS manages its memory and move data back-and-forth from disk.

### File Storage

Most DB, espesically new DB in current years store database as one or more files on disk.

Early systems in 1980s use custom filesystems on raw storage.

### About Pages

There are three differeent notions of "pages" in a DBMS:

- Hardware Page (usually 4KB)
- OS Page (usually 4KB)
- Database Page (512B-16KB)
  - 4KB: SQLite DB2 Oracle
  - 8KB: SQL-Server PostgreSQL
  - 16KB: MySQL

### Tuple Page Storage

![tuple page](graphs/tuple_page.png)

For example, if database want to store tuple into a page, it can have a list of offsets at the front of a page **(we call it slot)**, and use the offset to find the tuples from back of the page.

In PostgresDB, we can get the page-id and slot id by:

``` SQL
SELECT r.ctid, r.* FROM r

ctid   id   val
(0,1)  101  aaa
(0,2)  102  bbb
```

### Tuple Layout

the order of tuples (usually) depends on the order of CREATE.

Some DBMS can automatically re-order the attributes.

``` SQL
CREATE TABLE foo (
  a INT PRIMARY KEY,
  b INT NOT NULL,
  c INT,
  d DOUBLE,
  e FLOAT
)

```

![tuple order](graphs/tuple_order.png)

We can also **denormalize (prejoin)** related tuples and store them together in same page.

e.g.: RethinkDB, CouchDB, RavenDB

``` SQL
CREATE TABLE foo (
  a INT PRIMARY KEY,
  b INT NOT NULL,
);

CREATE TABLE bar (
  c INT PRIMARY KEY,
  a INT REFERENCES foo(a),
);
```

![tuple denormalize](graphs/tuple_denormallize.png)

It (denormalize) can:

- Potentially reduces the amount of I/O for common workload patterns.
- Can make updates more expensive.

**Record IDs:**

Each tuple is assigned a unique **record identifier**

- Most common: page_id + offset/slot
- Can also contain file location info
  
e.g.: PostgreSQL(CTID 4-bytes) SQLite(ROWID 8-bytes) ORACLE (ROWID)

An application **cannot** rely on these ids to mean anything because it will re-arrange automaticallly be DBMS.

### Log-Structured File Organization

Instead of storing tuples in pages, the DBMS oly stores **log records**

### Decimal

Decimal runs faster than fixpoint number, and, decimal number will not have persicion issue.

**Outer Files:**

Store the path of file in tuple and store the file in file system outside the tuple.

Also can use BLOB(Binary Large Object) to store file inside pages. Usually we only treat files smaller than 256KB using this method.

### Metadata

Metadata stores the schema info of a table.

Databases can use SQL cmd to get the metadata of a table.

### Database Storage Models

![wikipedia_example](graphs/wikipedia_example.png)

**OLTP:** On-line Transaction Processing

- Simple queries that read/update a small amount of data that is related to a single entity in the database.

e.g.: ![OLTP](graphs/OLTP.png)

**OLAP:** On-line Analytical Processing

- Complex queries that read large portions of the database spanning multiple entities.

e.g.: ![OLAP](graphs/OLAP.png)

**Decomposition Storage Model (DSM):**

The DBMS stores the values of a single attribute for all tuples contiguously in a page. Known as a "column store".

![column store](graphs/column_store.png)

**Column Storge Tuple Identifications:**

Choice 1: Fixed-length Offsets

Each value have a same length.

![fixed offsets](graphs/fixed_offsets.png)

Choice 2: Embedded Ids

Each value is stored with its tuple id. (Rarely used recently)

![embedded ids](graphs/embedded_ids.png)

OLTP = Row Store

OLAP = Column Store

**Two main problem of Database Storageï¼**

- #1: How the DBMS represents the database in files on disk. (This section)
- #2: How the DBMS manage its memory and move data back-and-forth from disk. (Next section: Buffer Pools)

## Buffer Pools

We cannot read/write/modify data directly on Disk. We should move pages into memory.

So we have two main problem to think:

Spatial Control:

- Where to write pages on disk
- The goal is to keep pages that are used together often as physically close together as possible on disk

Temporal Control:

- When to read pages into memory, and when to write them to disk.
- The goal is minimize the number of stalls from having to read data from disk

### Bufer Pool Organization

We use **page table** to kepps track of pages that are currently in memory.

Also maintains additional meta-data per page:

- Dirty Flag (Modified, should flash to disk)
- Pin/Reference Counter (reading, cannot removed from memory)

If need to read a page not in memory, we should add a **latch** to Page Table in this page. In case of other people put this page in another **frame** in Buffer Pool.

![buffer pool](graphs/buffer_pool.png)

**Locks vs. Latches:**

Locks:

- Protects the database's logical contents from other transations.
- Held for transactions duration.
- Need to be able to rollback changes.

Latches (Mutex):

- Protects the critical sections of the DBMS's internal data structrue from other threads
- Held for operation duration
- Do not need to be able to roll back changes

**Page Table vs. Page Directory:**

**Page Directory** is the mapping from page ids to page locations in the database files.

- All changes must be recorded on disk to  allow the DBMS to find on restart.

**Page Table** is the mapping from page ids to a copy of the page in buffer pool frames.

- This is an in-memory data structure that does not need to be stored on disk.

### Buffer Pool Optimization

- Multiple Buffer Pools
- Pre-fetching
- Scan Sharing
- Buffer Pool Bypass

**Multiple Buffer Pools:**

The DBMS does not always have a single buffer pool for the entire system.

- Multiple buffer pool instance (maybe by hash)
- Per-database buffer pool
- Per-page type buffer pool

Using multiple buffer pool can lower the latch conflict, and can have better locality.

**Scan Sharing:**

If two Query scan same/simular pages, if they scan seprately, pages may read into buffer pool two/more times.

If we can share the scan, read pages into buffer pool and let multiple query to read them, then after it, switch them out and read new pages, we can lower the IO of disk.

**Buffer Pool Bypass:**

The sequential scan operator will not store fetched pages in the buffer pool to avoid overhead.

- Memory is local to running query.
- Works well if operator needs to read a large sequence of pages that are contiguous on disk.
- Can also be used for temporary data (sorting, joins).

Supported by most DBMS for example Oracle SQL-Server PostgreSQL.

Called "Light Scans" in Informix.

### Buffer Pool Replacement Policies

LRU: common

Clock: 

## Hash Table

**Difference speed to different Hash Functions**

like MurmurHash XXHash(Facebook) CityHash/FarmHash(Google)

(XXHash2.0/3.0 has to fastest score)

The throwput to hashed differ by the length of bit of keys, ï¼èä¸åç°å¨æè§å¾, å ä¸ºcacheçå®¹éå³ç³»ï¼å¦ææ°å¥½è½å¡«æ»¡cacheï¼éåº¦æå¿«)

**Way to solve hash collision and delete key-value after collision**

Two way:

- mark a tombStone on deleted slot
- move values after this slot up (very complicated)

**Non-Unique Keys**

- Store values in separete storage area for each key.
- Store duplicate keys entries together in the hash table.

### Static Hash Tables

**Robinhood Hashing**

to better solve the collision issue.

save a variable of **distance from value to its target slot** of every value.

When collision, scan down and find a slot with lower **distance**, swap into it and continue find a slot to save the value just swaped out.

**Cuckoo Hashing**

Another way to solve collision.

Use multiple hash tables with different hash function seeds.

- On insert, check every table and pick anyone that has a free slot.
- If no table has a free slot, evict the element from one of them and then re-hash it find a new location.

Look-ups and deletions are always O(1) because only one location per hash table is checked.

### Dynamic Hash Tables

## Chained Hashing

**Extenable Hashing**

Use a global counter to mark **how many bits we use to map hash(keys)**

![hash buckets](graphs/hash_buckets.png)

When buckets goes full, we can divide buckets.

**Linear Hashing**

## Tree Index

### B+ Tree

"Best index for DB"

How to insert/delete is very common so pass it

**In Practice:**

Typical Fill-Factor: 67%

Typical Capacities:

- Height 4: 1334 = 312,900,721 entries
- Height 3: 1333 = 2,406,104 entries

Pages per level:

- Level 1 =      1 pages =   8 KB
- Level 2 =    134 pages =   1 MB
- Level 3 = 17,956 pages = 140 MB

**Clustered Index**

The table is stored in the sort order specified by the primary key.

**Node Size of B+ Tree**

The slower the storage device, the larger the optimal node size for a B+ Tree.

- HDD: 1MB
- SSD: 10KB
- In-Memory: 512B

Optimal sizes can vary depending on the workload

- Leaf Node Scans vs. Root-to-Leaf Traversals

**Variable Length Key**

Approach #1: Printers (very costy, Use in a memory database in history)

- Store the keys as pointers to the tuple's attribute

Approach #2: Varaible Length Nodes 

- The size of each node in the index can vary
- Requires carefull memory management

Approach #3: Padding (easy and common)

- Always pad the key to be max length of the key type

Appraoch #4: Key Map / Indirection

- Embed an array of pointers that map to the key + value list within the node.

**Non-Unique Indexed**

Approach #1: same as Unique keys, just duplicate it.

Approach #2: key points to a list of values

**Intra-Node Search**

#1: Linear Search

#2: Binary Search

#3: Interpolation (calculate an approximate location and start from that)

**Prefix Compression**

Sorted keys in the same leaf node are likey to have the same prefix.

Instead of storing the entire key each time, extract common prefix and store only unique suffix for each key.

![prefix](graphs/prefix_compression.png)

**Suffix Truncation**

The keys in the inner nodes are only used to "direct traffix"

- We don't need the entire key.

Store a **minimun prefix** that is needed to correctly route probes into the index.

Origin:

![suffix](graphs/suffix_truncation_origin.png)

Suffix Truncation:

![suffix](graphs/suffix_truncation_after.png)

**Bulk Insert**

The fastest/best way to build a B+ Tree is to first sort the keys and then build the index from the bottom up.

**Pointer Swizzling**

Nodes use page ids to reference other noeds in the index. The DBMS must get the memory location from the page table during traversal.

If a page is pinned in the buffer pool, then we 



 
