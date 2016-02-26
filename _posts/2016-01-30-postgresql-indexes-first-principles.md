---
layout: post
title: "PostgreSQL Indexes: First Principles"
tags: [postgresql, indexes, databases]
image: psql-first-principles.png
---

We have all heard about indexes. Yeah, that thing that it's automatically added
to the Primary Key column that enables fast data retrieval and stuff. Sure, but
have you ever asked yourself if there are multiple types or implementations of
indexes? Or maybe, what type of indexes your favourite RDBMS implements? In this
blog post, we will take a step back to the beginning, exploring what indexes are,
what is their role, types of indexes, metrics and so on. And all of this in
PostgreSQL.

## What's the Problem?

When explaining indexes, everyone uses the phonebook analogy. The problem is that
the phonebook analogy is so good, thinking of any other one is a waste of time.
But, before we jump into indexes, we need to understand the problem that they
solve.

So, think of a phonebook. The phonebook is a book with all of the names and
phone numbers of the people in a city or a country. Now, let's say we want to
find John Doe's phone number. Knowing that the phone book is alphabetically
ordered we look for the page where the surname Doe is. There, we look for John
and his phone number. Good, that is efficient enough for a phonebook.

Now, imagine if our phonebook was not alphabetically sorted? Hell, right? We
would need to go through all of the pages, reading every name in the phonebook,
until we find our John Doe. This is called **sequential searching**. You go over
all of the records, until you find the person whose phone number you are looking
for.

One does not need to be a genius to understand that this is super inefficient.
The problem with database tables is that the data in them is unordered. If we
had a `people` table, containing all information about people, to lookup the
person with the full name John Doe, we would need to execute:

{% highlight sql %}
SELECT * FROM people WHERE first_name = "John" and last_name = "Doe";
{% endhighlight %}

Easy enough. But although this query is fast, under the hood, the database will
hit every row of the table, until it finds the record it is looking for. I am
sure your side-project won't have this problem soon, but imagine a table with
millions of records, without indexes. Data retrieval would take seconds, sometimes
maybe minutes. Imagine waiting 30 seconds for a list of videos on YouTube?

## Introducing Indexes

I guess the phonebook example painted the picture well for you. Now, what's
interesting is that the phonebooks, actually have indexes. But, they are
different. They are not like a book index, which shows what chapter starts on
which page. Or maybe they do, I haven't seen a phonebook in a while.

### Clustered Indexes

Nevertheless, indexes have different architectures/indexing methods. And the
phonebook has a **clustered index**. Clustering means that the data is in a
distinct order, resulting the row data to be stored in order. If this confuses
you, think again about the phonebook - the records in the book are ordered
alphabetically. Regardless of how simple this might seem to you, it's a clustered
index. The way the data is ordered (clusters) makes it really easy to search and
find the needed phone number. So, a clustered index is an index which
**physically** orders the data (the actual bits on disk) in a certain way, and
when new data arrives it is saved in the same order.

A caveat with the clustered indexes is that **only one** can be created on a
given database table. That occurs due to their nature - they enforce a data
order. Also, clustered indexes increase the write time, because when new data
arrives, all of the data has to be rearranged. But on the bright side - they can
greatly increase the reading speed of the table.

To summarize - clustered indexes **order** the data physically (on disk) in
clusters.

### Non-clustered Indexes

I know you guessed it - if there are clustered indexes, the opposite type has to
exist. And you are right, non-clustered indexes are a thing. They are the type
that we know to use most, but I guess not everyone knows how their implementation
works.

Non-clustered indexes are indexes that keep a separate ordering list that has
pointers to the physical rows. It's basically like a book index, it knows on
what page a certain chapter starts and ends. Now, unlike the clustered index, a
table can have many non-clustered indexes. But, the caveat with these indexes is
that each new index will increase the time it takes to write new records.

So, to summarize - non-clustered indexes **do not order** the data physically,
they just keep a list of the data order.

## Indexing effects

So, after we covered the index architecture, we can explore how indexes work in
PostgreSQL. But first things first - let's play with some indexes.

Indexes in PostgreSQL are manipulated via a set of commands. For this example,
I will use a table with 1000 records, representing 1000 users. The table schema
will look like this:

{% highlight bash %}
+-------------------------------------------------+
|             Table "public.users"                |
+-------------------------------------------------+
|  Column    |         Type           | Modifiers |
+------------+------------------------+-----------+
| id         | integer                | not null  |
| first_name | character varying(60)  |           |
| last_name  | character varying(60)  |           |
| email      | character varying(120) |           |
| gender     | character varying(6)   |           |
| created_at | date                   |           |
+-------------------------------------------------+
|Indexes:                                         |
|   "users_pkey" PRIMARY KEY, btree (id)          |
+-------------------------------------------------+
{% endhighlight %}

If you want to follow along with this tutorial, you can easily create this table
with the following query:

{% highlight sql %}
CREATE TABLE users (
  id INT PRIMARY KEY,
  first_name VARCHAR(60),
  last_name VARCHAR(60),
  email VARCHAR(120),
  gender VARCHAR(6),
  created_at DATE
);
{% endhighlight %}

Having the table now, we can dump any amount of data we want. I will use the
following [CSV file](https://dl.dropboxusercontent.com/u/1167828/dummy-users.csv),
which contains 1000 user records. Dumping the data in the table is easy using
PostgreSQL's `COPY` command. If you are unfamiliar with the command, check out
[my post](/postgresql-copy) which explains it in depth.

In our case, the actual command is:

{% highlight sql %}
COPY users (
  id,
  first_name,
  last_name,
  email,
  gender,
  created_at
)
FROM '/some/path/here/to/dummy-users.csv'
DELIMITER ',';

/* Output: COPY 1000 */
{% endhighlight %}

Having the data in the database now, let's see the real impact of the indexes.

### Analyzing Queries

PostgreSQL creates a query plan for each query it receives. When writing your
queries, it is very important to choose the best query plan, which adheres to the
query structure and the properties of the data. The query plans are really easy
to produce in PostgreSQL. It comes with a really neat command called `EXPLAIN`, which
shows the query plan, which contains all of the relevant info.

For example, let's see the query plan for the query that returns all of the users:

{% highlight sql %}
EXPLAIN SELECT * FROM users;
{% endhighlight %}

This will return the following output:

{% highlight text %}
                          QUERY PLAN
----------------------------------------------------------
 Seq Scan on users  (cost=0.00..20.00 rows=1000 width=48)
(1 row)
{% endhighlight %}

As you can notice, this simple query requires a **Seq**uential **Scan**, because
there is no `WHERE` clause in it. This means that it will have to scan each row
of the table and return it.

In the parentheses we notice couple of values. The cost is a range of arbitrary
units (that closely resemble disk page fetches) - starting from the expected
before the output phase can begin, to the estimated total cost of this query.
Then, the numbers of rows output that were estimated by the query planner. Last,
the width represents the average width of the rows, in bytes.

Let's add a `WHERE` clause to the query:

{% highlight sql %}
SELECT * FROM users WHERE email = 'phughes5m@nbcnews.com';
{% endhighlight %}

The query will return the user whose email is `phughes5m@nbcnews.com`. Perfect.
Let's `EXPLAIN` the query, and see its query plan:

{% highlight sql %}
EXPLAIN SELECT * FROM users WHERE email = 'phughes5m@nbcnews.com';
{% endhighlight %}

{% highlight text %}
                          QUERY PLAN
----------------------------------------------------------
 Seq Scan on users  (cost=0.00..22.50 rows=1 width=48)
   Filter: ((email)::text = 'phughes5m@nbcnews.com'::text)
{% endhighlight %}

As you can see, although the `WHERE` clause was added, and the result set was
reduced to only one row, the time didn't go down. Actually, it increased. This
happens because the query will issue a **Seq**uential **Scan** on the table.
Let's see if adding an index to the `email` column will decrease the cost of the
query.

### Adding an Index

Adding an index in PostgreSQL is done via the `CREATE INDEX` command:

{% highlight sql %}
CREATE INDEX <index name> ON <table name> USING <method>(<column name>);
{% endhighlight %}

Or, in our specific case:

{% highlight sql %}
CREATE INDEX email_idx ON users USING btree(email);
{% endhighlight %}

After we run this command, let's see a description of the table:
{% highlight bash %}
              Table "public.users"
   Column   |          Type          | Modifiers
------------+------------------------+-----------
 id         | integer                | not null
 first_name | character varying(60)  |
 last_name  | character varying(60)  |
 email      | character varying(120) |
 gender     | character varying(6)   |
 created_at | date                   |
Indexes:
    "users_pkey" PRIMARY KEY, btree (id)
    "email_idx" btree (email)
{% endhighlight %}

You can see in the "Indexes" section, the new index appears. After we added the
index, let's see the query plan for the `SELECT` query:

{% highlight sql %}
EXPLAIN SELECT * FROM users WHERE email = 'phughes5m@nbcnews.com';
{% endhighlight %}

{% highlight text %}
                              QUERY PLAN
----------------------------------------------------------------------
Index Scan using email_idx on users  (cost=0.28..8.29 rows=1 width=48)
  Index Cond: ((email)::text = 'phughes5m@nbcnews.com'::text)
{% endhighlight %}

Whoa! The projected query cost dropped from the original 22.50 to 8.29. So,
what changed? As you can notice, instead of a Sequential Scan, now PostgreSQL
will issue a **Index Scan**. We won't go into details here, because types and
approaches of scanning is topic for another blogpost. Simply said, Postgres
will issue a scan based on the indexes, therefore finding the exact location of
the user that we are looking for and returning it. This is a much cheaper
operation then using a sequential scan.

## Sequential isn't Always the Worst

Let's see a different example. Let's say, we want to return all of the users that
have the `id` between 100 and 200:

{% highlight sql %}
EXPLAIN SELECT * FROM users WHERE id > 100 AND id < 200;
{% endhighlight %}

{% highlight text %}
                          QUERY PLAN
----------------------------------------------------------------------
Index Scan using users_pkey on users (cost=0.28..11.28 rows=100 width=48)
  Index Cond: ((id > 100) AND (id < 200))
(2 rows)
{% endhighlight %}

Looks about right, right? It uses a Index Scan, with an Index Cond(itional)
searching for all `id`s between 100 and 200. Good enough.

How about we try to return all users with `id`s between 200 and 800?

{% highlight sql %}
EXPLAIN SELECT * FROM users WHERE id > 200 AND id < 800;
{% endhighlight %}

{% highlight text %}
                          QUERY PLAN
-------------------------------------------------------
Seq Scan on users  (cost=0.00..25.00 rows=600 width=48)
  Filter: ((id > 200) AND (id < 800))
(2 rows)
{% endhighlight %}

Hold on, what happened here? Although there is an index on the primary key
column, PostgreSQL decides that doing a sequential scan on the `user` table is
more performant than an index scan. An index scan would require finding 600
indexes and returning each and every one of those records whose index were found.
On the other hand, a sequential scan would just go over each of the records and
filter out the unwanted rows.

So, although this is a contrived example, you can see in some situations an
index scan will not be as performant as a sequential scan.

## Careful Usage

Indexes are a powerful way to improve the performance of your tables, but they
have to be used carefully. Often, indexes can actually stand in the way of your
queries, if not used properly.

Next time we will see what are the index types in PostgreSQL and how we can
leverage them.

