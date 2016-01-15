---
layout: post
title: "In and out of PostgreSQL using COPY"
tags: [postgres, copy, csv]
---

I am pretty sure everyone of us has been in a situation where you needed to
generate a report and/or extract some data from a database and present it in a spreadsheet.
In many cases, our clients prefer Excel to handle spreadsheets/reports, because, duh, it's Excel.

So, how do you approach this problem? Do you copy and paste data? Or use a RDBMS GUI to
generate the report into a spreadsheet? Today, I'll show you a really small but convenient feature
of PostgreSQL - [COPY](http://www.postgresql.org/docs/9.2/static/sql-copy.html).


## Hello COPY

From the [COPY](http://www.postgresql.org/docs/9.2/static/sql-copy.html)
documentation: "COPY moves data between PostgreSQL tables and standard file-system files.
COPY TO copies the contents of a table to a file, while COPY FROM copies data from
a file to a table (appending the data to whatever is in the table already). COPY TO
can also copy the results of a SELECT query."

So, what does COPY do:

1. It can copy the contents of a file (data) to a table, or
2. It can copy the contents of a table (or a SELECT query result) into a file.

If a list of columns is specified, COPY will only copy the data in the specified
columns to or from the file. If there are any columns in the table that are not in the column list,
COPY FROM will insert the default values for those columns.

COPY with a file name instructs the PostgreSQL server to directly read from or write to a file.
The file must be accessible to the server and the name must be specified from the viewpoint of the server.
When STDIN or STDOUT is specified, data is transmitted via the connection between the client and the server.

Sounds good? Let's give it a try!

Disclaimer: COPY has the ability to read/write data from/to CSV and Binary files. Although I am
sure there are lots of usecases for using binary files, in this blogpost I will
only focus on using it for CSV files because, personally for me, they are the most
convenient for handing data sets.

## COPY TO

When you want to create a CSV file out of a SELECT query, or dump all of the contents of
a table in a CSV file, you can use the "COPY ... TO ..." command.

### Using a SELECT query

When you want to copy a result set to a CSV file, the format of the COPY command is:

{% highlight sql %}
COPY (<select-query-here>) TO <file-path>;
{% endhighlight %}

Or, a more real-life example:

{% highlight sql %}
COPY (SELECT * FROM people WHERE age > 21) TO '~/Desktop/adults.csv';
{% endhighlight %}

As you can see, we use the COPY command which copies the results into a CSV file on the local filesystem.
You can take the query a lot further. Here's a real life example of a project that I am
currently working on:

{% highlight sql %}
COPY (SELECT price_rules.* FROM quotes LEFT JOIN price_rules ON quotes.id = price_rules.chargeable_id where quotes.id = 437) TO '~Desktop/exports/price_rules.csv' CSV;
{% endhighlight %}

As you can see, you can use any SELECT query that can will return a data result set.
But, what's a CSV without headers, right? :-)

{% highlight sql %}
COPY (SELECT * FROM people WHERE age > 21) TO '~/Desktop/adults.csv' CSV HEADER;
{% endhighlight %}

Adding the keyword HEADER at the end will include headers in the CSV file, which
are the table column names.

Also, another key feature of CSV files are the delimiters. Depending on what character
delimiter you want the CSV file to have, you can specify the character in the command:

{% highlight sql %}
COPY (SELECT * FROM people WHERE age > 21) TO '~/Desktop/adults.csv' CSV DELIMITER ',' HEADER;
{% endhighlight %}

### Using a table name

When you want a whole table to be dumped into a CSV, the command is really simpler.
You just need to specify the table name and the target file:

{% highlight sql %}
COPY people TO '~/Desktop/people.csv' CSV DELIMITER ',' HEADER;
{% endhighlight %}

That's it.

## COPY FROM

Now, when you want to inject the data from the CSV file into a table, you can use the
"COPY ... FROM ..." command. The syntax is very simillar, with only one key difference:

{% highlight sql %}
COPY <table-name> FROM <file-path> DELIMITER ',' CSV HEADER;
{% endhighlight %}

Or, using a real life example:

{% highlight sql %}
COPY addresses FROM '~/Desktop/addresses.csv' DELIMITER ',' CSV HEADER;
{% endhighlight %}

## Outro

COPY is a really neat and cool feature of Postgres. For brewity, I tried to keep this
blogpost short and simple. If you have any thoughts and questions feel free to drop me a comment.
Or, if you don't feel chatty, you can head over to the [COPY documentation](http://www.postgresql.org/docs/9.4/static/sql-copy.html).
