---
title: Multi-table Join Queries
---

# Multi-table Join Queries

Most of the time，we need to use data from multiple tables in one query. At this time, we can combine the data of two or more tables through the `JOIN` statement.

## Join type

### INNER JOIN

The join result of an inner join returns only rows that match the join condition.

For example, suppose if we want to know which authors have written the most books, we need to join the author base information table named `authors` with the book author table named `book_authors`.

<SimpleTab>
<div label="SQL" href="inner-join-sql">

In the following SQL statement, we use the keyword `JOIN` to declare that we want to join the rows of the left table `authors` and the right table `book_authors` as an inner join with the join condition `a.id = ba.author_id`, then the result set of the join will only contain rows that satisfy the join condition. If an author has not written any books, then his record in `authors` table will not satisfy the join condition and will therefore not appear in the result set.

{{< copyable "sql" >}}

```sql
SELECT ANY_VALUE(a.id) AS author_id, ANY_VALUE(a.name) AS author_name, COUNT(ba.book_id) AS books
FROM authors a
JOIN book_authors ba ON a.id = ba.author_id
GROUP BY ba.author_id
ORDER BY books DESC
LIMIT 10;
```

The query results are as follows:

```
+------------+----------------+-------+
| author_id  | author_name    | books |
+------------+----------------+-------+
|  431192671 | Emilie Cassin  |     7 |
|  865305676 | Nola Howell    |     7 |
|  572207928 | Lamar Koch     |     6 |
| 3894029860 | Elijah Howe    |     6 |
| 1150614082 | Cristal Stehr  |     6 |
| 4158341032 | Roslyn Rippin  |     6 |
| 2430691560 | Francisca Hahn |     6 |
| 3346415350 | Leta Weimann   |     6 |
| 1395124973 | Albin Cole     |     6 |
| 2768150724 | Caleb Wyman    |     6 |
+------------+----------------+-------+
10 rows in set (0.01 sec)
```

</div>
<div label="Java" href="inner-join-java">

{{< copyable "java" >}}

```java
public List<Author> getTop10AuthorsOrderByBooks() throws SQLException {
    List<Author> authors = new ArrayList<>();
    try (Connection conn = ds.getConnection()) {
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery("""
        SELECT ANY_VALUE(a.id) AS author_id, ANY_VALUE(a.name) AS author_name, COUNT(ba.book_id) AS books
        FROM authors a
        JOIN book_authors ba ON a.id = ba.author_id
        GROUP BY ba.author_id
        ORDER BY books DESC
        LIMIT 10;
        """);
        while (rs.next()) {
            Author author = new Author();
            author.setId(rs.getLong("author_id"));
            author.setName(rs.getString("author_name"));
            author.setBooks(rs.getInt("books"));
            authors.add(author);
        }
    }
    return authors;
}
```

</div>
</SimpleTab>

### LEFT OUTER JOIN

The left outer join will return all the data rows in the left table and the values ​​in the right table that can match the join condition. If no matching rows are found in the right table, it will be filled with `NULL`.

In some cases, we want to use multiple tables to complete the data query, but do not want the data set to become smaller because the join condition are not met. 

For example, on the homepage of the Bookshop app, we want to display a list of the latest books with average ratings. In this case, the latest books may not have been rated by anyone, and using inner joins will cause the information of these unrated books to be filtered out, which is not what we expect.

<SimpleTab>
<div label="SQL" href="left-join-sql">

In the following SQL statement, we use the `LEFT JOIN` keyword to declare that the left table `books` will be joined to the right table `ratings` in a left outer join, thus ensuring that all rows in the `books` table are returned.

{{< copyable "sql" >}}

```sql
SELECT b.id AS book_id, ANY_VALUE(b.title) AS book_title, AVG(r.score) AS average_score
FROM books b
LEFT JOIN ratings r ON b.id = r.book_id
GROUP BY b.id
ORDER BY b.published_at DESC
LIMIT 10;
```

The query results are as follows:

```
+------------+---------------------------------+---------------+
| book_id    | book_title                      | average_score |
+------------+---------------------------------+---------------+
| 3438991610 | The Documentary of lion         |        2.7619 |
| 3897175886 | Torey Kuhn                      |        3.0000 |
| 1256171496 | Elmo Vandervort                 |        2.5500 |
| 1036915727 | The Story of Munchkin           |        2.0000 |
|  270254583 | Tate Kovacek                    |        2.5000 |
| 1280950719 | Carson Damore                   |        3.2105 |
| 1098041838 | The Documentary of grasshopper  |        2.8462 |
| 1476566306 | The Adventures of Vince Sanford |        2.3529 |
| 4036300890 | The Documentary of turtle       |        2.4545 |
| 1299849448 | Antwan Olson                    |        3.0000 |
+------------+---------------------------------+---------------+
10 rows in set (0.30 sec)
```

It seems that the latest published book already has a lot of ratings. To verify the above, let's delete all the ratings of the book **The Documentary of lion** through the SQL statement:

{{< copyable "sql" >}}

```sql
DELETE FROM ratings WHERE book_id = 3438991610;
```

Query again, you will find that the book **The Documentary of lion** still appears in the result set, but the `average_score` column calculated from `score` of the right table `ratings` is filled with `NULL`.

```
+------------+---------------------------------+---------------+
| book_id    | book_title                      | average_score |
+------------+---------------------------------+---------------+
| 3438991610 | The Documentary of lion         |          NULL |
| 3897175886 | Torey Kuhn                      |        3.0000 |
| 1256171496 | Elmo Vandervort                 |        2.5500 |
| 1036915727 | The Story of Munchkin           |        2.0000 |
|  270254583 | Tate Kovacek                    |        2.5000 |
| 1280950719 | Carson Damore                   |        3.2105 |
| 1098041838 | The Documentary of grasshopper  |        2.8462 |
| 1476566306 | The Adventures of Vince Sanford |        2.3529 |
| 4036300890 | The Documentary of turtle       |        2.4545 |
| 1299849448 | Antwan Olson                    |        3.0000 |
+------------+---------------------------------+---------------+
10 rows in set (0.30 sec)
```

What happens if instead you use the inner join through `JOIN`? It's up to you to try.

</div>
<div label="Java" href="left-join-java">

{{< copyable "java" >}}

```java
public List<Book> getLatestBooksWithAverageScore() throws SQLException {
    List<Book> books = new ArrayList<>();
    try (Connection conn = ds.getConnection()) {
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery("""
        SELECT b.id AS book_id, ANY_VALUE(b.title) AS book_title, AVG(r.score) AS average_score
        FROM books b
        LEFT JOIN ratings r ON b.id = r.book_id
        GROUP BY b.id
        ORDER BY b.published_at DESC
        LIMIT 10;
        """);
        while (rs.next()) {
            Book book = new Book();
            book.setId(rs.getLong("book_id"));
            book.setTitle(rs.getString("book_title"));
            book.setAverageScore(rs.getFloat("average_score"));
            books.add(book);
        }
    }
    return books;
}
```

</div>
</SimpleTab>

### RIGHT OUTER JOIN

A right outer join returns all the records in the right table and the values ​​in the left table that match the join condition. If there is no matching value, it is filled with `NULL`.

### FULL OUTER JOIN

The full outer join is based on all the records in the left table and the right table. If no value that meets the join condition is found in the another table, it is filled with `NULL`.

### CROSS JOIN

When the join condition is constant, the inner join between the two tables is called a [cross join](https://en.wikipedia.org/wiki/Join_(SQL)#Cross_join). A cross join joins every record of the left table to all the records of the right table. If the number of records in the left table is m and the number of records in the right table is n, then m \* n records will be generated in the result set.

### LEFT SEMI JOIN

TiDB does not support `LEFT SEMI JOIN table_name` at the SQL syntax level, but at the execution plan level, [subquery-related optimizations](https://docs.pingcap.com/tidb/stable/subquery-optimization/) will use `semi join` as the default join method for rewritten equivalent JOIN queries.

## Implicit join

Before the `JOIN` statement that explicitly declared a join appeared as an SQL standard, it was possible to join two or more tables in a SQL statement using the `FROM t1, t2` clause, using `WHERE t1.id = t2.id` clause to specify the conditions for the join. You can think of it as an implicitly declared join, which uses an inner join way to join tables.

## Join related algorithms

TiDB supports the following three general table join algorithms and the optimizer will select the appropriate join algorithm to execute based on factors such as the amount of data in the joined table. You can see which algorithm the query uses for Join by using the `EXPLAIN` statement.

- [Index Join](https://docs.pingcap.com/tidb/stable/explain-joins#index-join)
- [Hash Join](https://docs.pingcap.com/tidb/stable/explain-joins#hash-join)
- [Merge Join](https://docs.pingcap.com/tidb/stable/explain-joins#merge-join)

If it is found that the optimizer of TiDB does not execute according to the optimal join algorithm. You can also use [Optimizer Hints](https://docs.pingcap.com/tidb/stable/optimizer-hints) to force TiDB to use a better join algorithm for execution.

For example, assuming the example SQL for the left join query above executes faster using the Hash Join algorithm, which is not chosen by the optimizer, you can add hint `/*+ HASH_JOIN(b, r) */` the `SELECT` keyword (Note: If the table name is aliased, the table alias should also be used in hint).

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT /*+ HASH_JOIN(b, r) */ b.id AS book_id, ANY_VALUE(b.title) AS book_title, AVG(r.score) AS average_score
FROM books b
LEFT JOIN ratings r ON b.id = r.book_id
GROUP BY b.id
ORDER BY b.published_at DESC
LIMIT 10;
```

Hints related to join algorithm:

- [MERGE_JOIN(t1_name [, tl_name ...])](https://docs.pingcap.com/tidb/stable/optimizer-hints#merge_joint1_name--tl_name-)
- [INL_JOIN(t1_name [, tl_name ...])](https://docs.pingcap.com/tidb/stable/optimizer-hints#inl_joint1_name--tl_name-)
- [INL_HASH_JOIN(t1_name [, tl_name ...])](https://docs.pingcap.com/tidb/stable/optimizer-hints#inl_hash_join)
- [HASH_JOIN(t1_name [, tl_name ...])](https://docs.pingcap.com/tidb/stable/optimizer-hints#hash_joint1_name--tl_name-)

## Join order

In actual business scenarios, join statements of multiple tables are very common, and the execution efficiency of join is related to the order in which each table participates in join. TiDB uses the Join Reorder algorithm to determine the order in which multiple tables are joined.

When the join order selected by the optimizer is not good enough, you can use the `STRAIGHT_JOIN` syntax to make TiDB enforce the join query in the order of the tables used in the FROM clause.

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT *
FROM authors a STRAIGHT_JOIN book_authors ba STRAIGHT_JOIN books b
WHERE b.id = ba.book_id AND ba.author_id = a.id;
```

You can learn about the implementation details and limitations of this algorithm by checking the [Introduction to Join Reorder Algorithm](https://docs.pingcap.com/tidb/stable/join-reorder) chapter.

## Read more

- [Explain Statements That Use Joins](https://docs.pingcap.com/tidb/stable/explain-joins)
- [Introduction to Join Reorder](https://docs.pingcap.com/tidb/stable/join-reorder)