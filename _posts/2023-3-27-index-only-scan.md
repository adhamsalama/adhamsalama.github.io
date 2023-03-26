---
title: Improve PostgreSQL query performance by utilizing an index-only scan
date: 2023-3-27 12:12:12 +/-0800
categories: [Databases]
tags: [databases]
author: adham
---

## Indexes in PostgreSQL

All indexes in PostgreSQL are _secondary_ indexes, meaning that each index is stored separately from the table's main data area (the heap).  
This means that in an ordinary index scan, each row retrieval requires fetching data from both the index and the heap.

To solve this performance problem, PostgreSQL supports _index-only scans_, which can answer queries from an index alone without any heap access.  
The basic idea is to return values directly out of each index entry instead of consulting the associated heap entry.

There are two fundamental restrictions on when this method can be used:

1. The index type must support index-only scans. B-tree indexes always do.
2. The query must reference only columns stored in the index.

## Example

Let's take a look at this table called students.

```sql
create table students (
  id serial primary key,
  g int,
  firstname text,
  lastname text,
  middlename text,
  address text,
  bio text,
  dob date
)
```

Let's generate a big amount of data for this table.

```sql
insert into
  students (
    g,
    firstname,
    lastname,
    middlename,
    address,
    bio,
    dob
  )
select
  random() * 100,
  substring(md5(random() :: text), 0, floor(random() * 31) :: int),
  substring(md5(random() :: text), 0, floor(random() * 31) :: int),
  substring(md5(random() :: text), 0, floor(random() * 31) :: int),
  substring(md5(random() :: text), 0, floor(random() * 31) :: int),
  substring(md5(random() :: text), 0, floor(random() * 31) :: int),
  now()
from
  generate_series(0, 1000000);
```

### No Index

If we select the last name of students whose grade is equal to a specific grade (12 for example) and analyze the query execution, we get this:

```sql
explain analyze select lastname from students where g = 12;
```

| QUERY PLAN                                                                                                                   |
| ---------------------------------------------------------------------------------------------------------------------------- |
| Gather (cost=1000.00..26330.64 rows=10533 width=15) (actual time=1.610..146.335 rows=10078 loops=1)                          |
| Workers Planned: 2                                                                                                           |
| Workers Launched: 2                                                                                                          |
| \-&gt; Parallel Seq Scan on students (cost=0.00..24277.34 rows=4389 width=15) (actual time=0.362..124.327 rows=3359 loops=3) |
| Filter: (g = 12)                                                                                                             |
| Rows Removed by Filter: 329974                                                                                               |
| Planning Time: 0.145 ms                                                                                                      |
| Execution Time: 147.634 ms                                                                                                   |

Because there is no index, PostgreSQL did a sequential scan of the entire table to get the results.

It also did it in parallel so it was a bit faster.

If we want to make this query faster, we could add an index on g.

### Normal index on a column

```sql
create index g_idx on students(g);
```

If we execute the query again, we get this result:

| QUERY PLAN                                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------ |
| Bitmap Heap Scan on students (cost=118.06..16974.48 rows=10533 width=15) (actual time=13.775..33.405 rows=10078 loops=1) |
| Recheck Cond: (g = 12)                                                                                                   |
| Heap Blocks: exact=7829                                                                                                  |
| \-&gt; Bitmap Index Scan on g_idx (cost=0.00..115.42 rows=10533 width=0) (actual time=7.454..7.455 rows=10078 loops=1)   |
| Index Cond: (g = 12)                                                                                                     |
| Planning Time: 0.129 ms                                                                                                  |
| Execution Time: 35.000 ms                                                                                                |

We can see that the execution time went down from 66 ms to 35 ms.

Impressive. But can we do more? Yes!

### A covering index

This query can execute faster if it performed an index-only scan. i.e, if the lastname field was stored in the index.

How can we do that?  
Let's drop the index

```sql
drop index g_idx;
```

And create a new index on g that also includes lastname

```sql
create index g_idx on students(g) include (lastname);
```

Now if we execute the query again, we get this result:

| QUERY PLAN                                                                                                                    |
| ----------------------------------------------------------------------------------------------------------------------------- |
| Index Only Scan using g_idx on students (cost=0.42..384.75 rows=10533 width=15) (actual time=0.132..2.762 rows=10078 loops=1) |
| Index Cond: (g = 12)                                                                                                          |
| Heap Fetches: 0                                                                                                               |
| Planning Time: 2.493 ms                                                                                                       |
| Execution Time: 3.432 ms                                                                                                      |

We can see that the database performed an Index Only scan, so it didn't have to return to the heap and fetch the rows matched because lastname is already stored in the index.  
The execution time went down from 35 ms to 3 ms!

This is called a _covering index._  
A covering index is an index specifically designed to include the columns needed by a particular type of query that you run frequently.

## Note about non-key payload columns

However, it's wise to be conservative about adding non-key payload columns to an index, especially wide columns. If an index tuple exceeds the maximum size allowed for the index type, data insertion will fail. In any case, non-key columns duplicate data from the index's table and bloat the size of the index, thus potentially slowing searches.

Now you might ask, what is the difference between the last index we created and a multicolumn index? We can discuss that in the next article!

## Resources

- [https://www.udemy.com/course/database-engines-crash-course/](https://www.udemy.com/course/database-engines-crash-course/)
- [https://www.postgresql.org/docs/current/indexes-index-only-scans.html](https://www.postgresql.org/docs/current/indexes-index-only-scans.html)
