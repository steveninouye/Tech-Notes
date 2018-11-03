### Links

[Coding Horror - A Visual Explanation of SQL Joins](https://blog.codinghorror.com/a-visual-explanation-of-sql-joins/)
[SQL Bolt - SQL Topic: Subqueries](https://sqlbolt.com/topic/subqueries)
[PostgreSQL Tutorial - PostgreSQL CASE](http://www.postgresqltutorial.com/postgresql-case/)
[PostgreSQL Tutorial - PostgreSQL COALESCE](http://www.postgresqltutorial.com/postgresql-coalesce/)

# SQL

## A Brief History

SQL, short for Structured Query Language, was created in the early 1970s at IBM
to manipulate and retrieve data in their database management system. In the
late 1970s, Relation Software, Inc. (now Oracle Corporation) built on the SQL
concepts from IBM to develop their own SQL-based relational database management
system (RDBMS). In 1979, they introduced the first commercially-available
implementation of SQL, Oracle V2.

Today, Oracle remains one of the key players in SQL products, but there are
many different databases and implementations of SQL. Most of them don't follow
all of the SQL standard and are therefore inconsistent with one another; as a
result, it is rarely possible to port SQL code from one database to another
without modification. In particular, there is great variation in how different
implementations handle date and time syntax, `NULL`s, string concatenation, and
comparison case sensitivity. One notable exception is PostgreSQL, which strives
to comply with all of the SQL standard.

## SQL versus NoSQL

Non-relational databases have existed since the 1960s. They declined in
popularity with the advent of SQL from the early 1980s up to 2010, at which
time their approach to data storage and retrieval went through a resurgence. It
was at this time that the moniker "NoSQL" came about. NoSQL implementations
like MongoDB have been relatively popular since the 2000s.

There is quite a debate about whether SQL or NoSQL is better. Speaking
generally, there is not an answer; each is better suited to different purposes,
so the choice of database system should depend on the project. At the most
basic level, SQL and NoSQL differ in how they store data, and as a result,
operations like inserting, retrieving, and updating data happen very
differently.

SQL stores data in tables. NoSQL stores data in forms other than tables; these
forms can be graphs, key-value pairs, or one of many other options, but
most commonly they are documents. These documents are very similar to JSON
objects with field-value pairs. For example, our NoSQL database might store a
user like this:

```
{
  username: "mongoDB4ever",
  email: "alexa@gmail.com",
  password_digest: "Ke&63h1z$mK9jd37n"
}
```

So instead of rows, NoSQL has documents. These documents can be stored in
collections, which are similar to SQL's tables in that they hold and organize
related entries in the database. Beyond that, however, they are quite
different. Unlike tables, collections do not have schema. They are not
restricted to particular columns/fields. They do not require specific data
types, like strings, integers, or dates. They are much more flexible. This
means that the design does not have to be specified upfront.

It also means that joins are not possible in NoSQL. We have no guarantee of
specific columns to match user for key-matching (i.e. foreign key and primary
key) like we do in SQL, and even if we did give every document the same field,
there is no function built in to link them. As a result, NoSQL is often
implemented so that a single document holds any related information. Our user
document in NoSQL might look like this, then:

```
{
  username: "mongoDB4ever",
  email: "alexa@gmail.com",
  password_digest: "Ke&63h1z$mK9jd37n",
  featured_photo: {
    url: "https://imgur.com/FRK6meX",
    caption: "best pizza ever"
  }
}
```

In SQL, we would probably have stored this photo in a separate photos table,
and simply had the user store the photo's `id` to reference the photos table.
This technique of not storing duplicate information but instead storing a
reference to it is known as **normalization**. NoSQL typically uses
**denormalization**.

Denormalization is useful for speed in that a single simple query can retrieve
all of the information we need. For this reason, NoSQL is often cited as being
faster than SQL. However, this duplication of information means that any time
we want to change some data, we need to update the same thing in multiple
places, which is slower.

SQL also has the option of using transactions. We can ensure that an update to
two different tables happens successfully, or roll back all of the changes if
one of the tables fails to update. In NoSQL, there are ways to mimic
transactions, but they must be done manually.

Check out this [article][sql-no-sql] if you're interested in learning
about these differences in greater depth.

[sql-no-sql]: https://www.sitepoint.com/sql-vs-nosql-differences/

## Flavors of SQL / NoSQL

There are many different databases available. Many of the popular ones today
use SQL and are relational. A few (MongoDB, Redis) are not. Database
implementations vary (sometimes widely) in the following attributes:

**Licensing & Pricing**: Open source, free, paid, subscription, etc.

**Limits**: Maximum database size, table size, row size, etc.

**DB Capabilities**: Types of joins, storage of multimedia objects, etc.

**Supported Data Types**: `DATETIME` versus separate `DATE` and `TIME`,
number and string options, etc.

**Access Control Features**: Different rules and processes available for
maintaining security

**Ease of Setup**: How much configuration is necessary

### Specific SQL/NoSQL Implementations

NB: Some companies may use multiple implementations of SQL or NoSQL or both.
The best implementation for the job will depend on the type of tool or
application they are building and what their priorities for it are. For example,
a company might use MySQL for an internal messaging tool but use Redis for
their main customer-facing application.

#### PostgreSQL

Known for: being open-source, most standard-compliant, easy set up.

Used by: Instagram, Netflix, Uber, Postmates, Reddit, Spotify, 500px

#### SQLite

Known for: very easy setup, no separate server process, being lightweight and portable

Used by: Rumble, Empatica, Spire, Initia

#### MySQL

Known for: being open-source, wide usage, cross-platform support, ease of use

Used by: Twitter, Dropbox, Vine, 9GAG, Pinterest, Tumblr, Github

#### Oracle

Known for: reliability, enterprise scale

Used by: LinkedIn, Netflix, Ebay, HealthExpense, iFactor

#### MongoDB

Known for: Document-oriented storage (NoSQL), being open source, high performance,
ease of use, flexibility, easy maintenance

Used by: Hootsuite, Uber, Foursquare,

#### Redis

Known for: Performance, advanced key-value cache storage (NoSQL), easy
deployment, being open source, speed

Used by: Twitter, Instagram, 9GAG, Vine, Hootsuite, AirBnb, Uber, Medium

## Terminology

**SQL** - Structured Query Language

-   E.g. "There are many types of joins in SQL."

**RDBMS** - Relational Database Management System

-   E.g. "Oracle is a popular RDBMS because of its enterprise scale."

**Query** - An operation that retrieves data from one or more tables; describes
desired data, leaving the database management system to plan, optimize, and
carry out the operations necessary to produce the results

-   E.g. "Our query should retrieve all of the users and their photos."

**Three-Valued (Ternary) Logic** - A many-valued logic system with three truth
values: True, False, and Unknown (null)

-   E.g. "SQL's use of ternary logic means that we must explicitly check for NULL."

**Transaction** - A unit of work performed against a database that is treated
in a coherent and reliable way; all of the work within a transaction must
succeed, or it is rolled back entirely, i.e. "all or nothing"

-   E.g. "Because these operations are in a transaction, we can trust that
    our data will be consistent across both tables."

**Normalization** - An approach to database storage that practices storing
references to information that lives in a different location rather than duplicating it and
storing it in multiple places; antonym: denormalization

-   E.g. "SQL implementations are typically normalized and therefore avoid
    repetition of the same data in multiple tables."

    ***

    # Programming Paradigms

At this point, you may be wondering to yourself: _what in the world does
programming paradigm even mean?_ **You're in luck.** You're not the only one.
_Programming paradigms_ are just ways to classify programming languages according to
their style.

There are a lot of different styles of programming, and a programming language
isn't necessarily bound to one specific style. For now, let's talk about two
opposite styles in Ruby, imperative and declarative.

### Imperative Programming

The original style of high-level languages, imperative programming just feeds
step-by-step instructions for the computer to execute.

```ruby
def imperative_odds(array)
  idx = 0
  odds = []
  while (idx < array.length)
    if array[idx].odd?
      odds << array[idx]
    end
    i += 1
  end
  odds
end
```

### Declarative Programming

In contrast, declarative programming describes what you want to
achieve, without going into too much detail about how you're going to do it.

```ruby
def declarative_odds(array)
  odds = array.select { |el| el.odd? }
end
```

The given examples are functionally the same, but are fundamentally different in
style. Keep in mind, although programming languages like Ruby allow for different
styles, certain languages restrict themselves to the guidelines of a specific
programming paradigm. In the next reading, you'll find **SQL** is an example of
declarative programming.

### Additional Resources

-   [LMU overview of programming paradigms][reading]
-   [First part of Stanford lecture series][video]

[reading]: http://cs.lmu.edu/~ray/notes/paradigms/
[video]: https://www.youtube.com/watch?v=Ps8jOj7diA0

---

# SQL

## Databases

The Ruby objects you create during the lifetime of your program will
die when it closes. To save (or **persist**) data, you need to somehow
write the data to permanent storage, like the hard disk. We saw some
of this in the serialization chapter.

Applications usually also require rich relationships between pieces of
data. Consider a blogging system: users have many posts, posts have many
tags, users may be following other users, and so on.

Relational databases (also sometimes referred to as RDBMS, relational
database management systems) were developed to provide a means of
organizing data and their relationships, persisting that data, and
querying that data.

## Tables

Relational databases organize data in tables.

| id  | name  | age |
| --- | ----- | --- |
| 1   | John  | 22  |
| 2   | James | 24  |
| 3   | Sally | 54  |
| 4   | Bob   | 48  |
| 5   | Lucy  | 33  |
| 6   | Mary  | 98  |

Each row is a single entity in the table. Each column houses an
additional piece of data for that entity.

Every row in a database table will have a **primary key** which will
be its unique identifier in that table row. By convention, the primary
key is simply `id`. Most relational database systems have an
auto-increment feature to ensure that the primary keys are always
unique.

Breaking your domain down into database tables & columns is an
important part of developing any application. Each table will house
one type of resource: `people`, `houses`, `blog_posts`, etc. The
columns in the table will house the data associated with each instance
of the resource.

## Database Schemas

Your database **schema** is a description of the organization of
your database into tables and columns.

Designing your schema is one of the first and most important steps
when writing an application. It forces you to ask a basic but
essential question: what data does my application need to function?

When implementing a database schema, you must decide on three things:

-   the tables you will have
-   the columns each of those tables will have
-   and the data type of each of those columns

Schemas are mutable, so the decisions up front are not at all set
in stone. Still, you should spend time thinking about your schema at
the outset to avoid making major, avoidable mistakes.

The concept of **static typing** may be new to you. Ruby is **dynamically
typed** -- there is no need to specify in method parameters
or variables the class (also called **type**) of the data stored in
it. Ruby won't stop you even if you store something silly like a `Cat`
object in a variable named `favorite_dog`, or a `String` in a variable named `number`.

SQL is not quite so flexible; you must specify the type of data that
will go into each column.

Here are a few of the most common datatypes:

-   `BOOLEAN`
-   `INT`
-   `FLOAT` (stores "floating point" numbers)
-   `VARCHAR(255)` (essentialy a string with a length limit of 255
    chars)
-   `TEXT` (a string of unlimited length)
-   `DATE`
-   `DATETIME`
-   `TIME`
-   `BLOB` (non-textual, binary data; e.g., an image)

We'll see how exactly we create tables, include columns, and specify column
types in just a bit.

## Modeling Relationships

Now we have a way to store users and additional bits of data on them,
but how would we store associated entities like a blog post written
by a user?

We probably have the sense that they should be in their own
tables since they're not really additional attributes on a user (which
would call for additional columns), nor are they users themselves
(which would call for additional rows). But if posts were in their own
table, how would we know that they were associated with a
particular user?

We model these relationships between entries in separate tables
through **_foreign keys_**. A foreign key is a value in a database
table whose responsibility is to point to a row in a different
table. Check out the posts table below (and pretend that people were a
bit more creative in their titles and bodies).

```
posts table:

id |   title   |     body    |  user_id
---------------------------------------
1  |  'XXXX'   |   'xyz...'  |    3
2  |  'XXXX'   |   'xyz...'  |    5
3  |  'XXXX'   |   'xyz...'  |    7
4  |  'XXXX'   |   'xyz...'  |    10
5  |  'XXXX'   |   'xyz...'  |    2
6  |  'XXXX'   |   'xyz...'  |    5
```

The `user_id` column is a foreign key column. If we wanted to find all
the posts for the user with `id` 5, we'd look in the posts table and
retrieve all the posts where the `user_id` column had a value of 5. If
you already know a little SQL:

```sql
SELECT
  *
FROM
  posts
WHERE
  posts.user_id = 5
```

By convention, the foreign key in one table will reference the primary
key in another table. We usually call the column that houses the
foreign key `[other_table_name_singularized]_id`.

Foreign keys are how we model relationships between pieces of data
across multiple tables. This also allows us to ensure that data is not
duplicated across our database. Posts live in a single place, users in
another, and the foreign key (`user_id`) in `posts` expresses the
relation between the one and the other.

## Structured Query Language (SQL)

Now that we know what these tables look like and generally how
relationships are modeled between them, how do we actually get at the
data?

Enter SQL. SQL is a **domain-specific language** that's designed to
query data out of relational databases.

Here's a sample SQL query (we'll break it down in just a second):

```sql
-- Find crazy cat people
SELECT
  name, age, has_cats
FROM
  tenants
WHERE
  (has_cats = true AND age > 50)
```

SQL queries are broken down into clauses. Here, there is the `SELECT`
clause, the `FROM` clause, and the `WHERE` clause. `SELECT` takes a
list of comma-separated column names; only these columns of data will
be retrieved. `FROM` takes a table name to query. `WHERE` takes a list
of conditions separated by `AND` or `OR`; only rows matching these
conditions are returned..

SQL provides powerful filtering with `WHERE`; it supports the standard
comparison and equality operators (`<`, `>`, `>=`, `<=`, `=`, `!=`) as well as
boolean operators (`AND`, `OR`, `NOT`).

There are 4 main data manipulation operations that SQL provides:

-   `SELECT`: retrieve values from one or more rows
-   `INSERT`: insert a row into a table
-   `UPDATE`: update values in one or more existing rows
-   `DELETE`: delete one or more rows

Below are brief descriptions of each of the operators syntactical
signatures and a couple simple examples of their use:

### `SELECT`

**Structure:**

```sql
SELECT
  one or more columns (or all columns with *)
FROM
  one (or more tables, joined with JOIN)
WHERE
  one (or more conditions, joined with AND/OR);
```

**Examples:**

```sql
SELECT
  *
FROM
  users
WHERE
  name = 'Ned';

SELECT
  account_number, account_type
FROM
  accounts
WHERE
  (customer_id = 5 AND account_type = 'checking');
```

### `INSERT`

**Structure:**

```sql
INSERT INTO
  table name (column names)
VALUES
  (values);
```

**Examples:**

```sql
INSERT INTO
  users (name, age, height_in_inches)
VALUES
  ('Santa Claus', 876, 34);

INSERT INTO
  accounts (account_number, customer_id, account_type)
VALUES
  (12345, 76, 'savings');
```

### `UPDATE`

**Structure:**

```sql
UPDATE
  table_name
SET
  col_name1 = value1,
  col_name2 = value2
WHERE
  conditions
```

**Examples:**

```sql
UPDATE
  users
SET
  name = 'Eddard Stark', house = 'Winterfell'
WHERE
  name = 'Ned Stark';

UPDATE
  accounts
SET
  balance = 30
WHERE
  id = 6;
```

### `DELETE`

**Structure:**

```sql
DELETE FROM
  table_name
WHERE
  conditions
```

**Examples:**

```sql
DELETE FROM
  users
WHERE
  (name = 'Eddard Stark' AND house = 'Winterfell');

DELETE FROM
  accounts
WHERE
  customer_id = 666;
```

## Schema Definitions

Before basic querying can take place though, you need to actually
define your database schema. There are three operators that SQL
provides to manipulate a database schema:

-   `CREATE TABLE`
-   `ALTER TABLE`
-   `DROP TABLE`

Here's an example of creating a users table (we'll break it down
shortly):

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  birth_date DATE,
  house VARCHAR(255),
  favorite_food VARCHAR(20)
);
```

`CREATE TABLE` first specifies the name of the table, and then in
parentheses, the list of column names along with their data types.

## Querying across multiple tables (JOIN)

Similarly to the objects in a good Ruby program, a well-designed
database will split data into tables that each encapsulate some
object. Sometimes, we will want to access the data from more than
one of these tables at once, but so far we've only seen ways to
query a single table. How might we query across tables?

SQL provides a powerful facility: the `JOIN`. A `JOIN` will
do just what you'd expect it to do: join together two tables,
resulting in a temporary combined table that you can query just like
any other. `JOIN` clauses include an `ON` statement, in which you
specify how exactly those two tables relate to one another. This is
where foreign keys come into play. Check out the simple join below.

Let's write a query that returns the title of all the blog posts
written by each user:

```sql
SELECT
  users.name, posts.title
FROM
  posts
JOIN
  users ON posts.user_id = users.id
```

This will return one row per post, with the user's name appearing next
to the title of the post they authored. By storing a `user_id` column in
the `posts` table, we can associate user data to posts without adding
columns that would duplicate data from other tables in the database and
other rows in the `posts` table; we just `JOIN` the tables as needed.

In this example, we joined two different tables using a foreign key
stored in a single column. This is the most common case we will see,
but `JOIN` is a flexible operator that can handle a variety of conditions.
Two variations we will use are self joins, in which we join a table to
itself (for example, if we have a table of employees, each of whom has
a supervisor in the same table) and joins that use multiple columns to
specify the `ON` condition (for example, if we have a bus timetable that
identifies routes by a company name and a route number).

In another case, we might have a many-to-many relationship; perhaps, in
the example above, `users` can "like" `posts`. In this case, including a
foreign key in one of the tables doesn't make sense; a user can like any
number of posts, and a post can be liked by any number of users. In
this case, we could use a **join table** that contains a foreign key for
each table, allowing us to represent each like with a row linking a user
to a post. We would then need two joins to associate users and liked
posts, like so:

```sql
SELECT
  users.name, posts.title
FROM
  posts
JOIN
  likes ON posts.id = likes.post_id
JOIN
  users ON likes.user_id = users.id
```

This query will give a list of user names and the posts they have liked.

## Resources

-   [Intro Video][youtube-dbclass]

[youtube-dbclass]: http://www.youtube.com/watch?v=wxFmiRwXcQY

---

# Self Joins

### What is a self join?

A self join is exactly what it sounds like: an instance of a table
joining with itself. The way you should visualize a self join for a
given table is by imagining a join performed between two identical
copies of that table.

Let's take a look at a classic self-join example. This returns each
employee's first and last name along with their manager's.

##### Employee Table

| id  | first_name | last_name | manager_id |
| :-- | :--------- | :-------- | :--------- |
| 1   | Kush       | Patel     | NULL       |
| 2   | Jeff       | Fiddler   | 1          |
| 3   | Quinn      | Leong     | 2          |
| 4   | Shamayel   | Daoud     | 2          |
| 5   | Robert     | Koeze     | 4          |
| 6   | Munyo      | Frey      | 3          |
| 7   | Kelly      | Chung     | 4          |

```sql
SELECT
  team_member.first_name, team_member.last_name,
   manager.first_name, manager.last_name
FROM
  employee AS team_member
JOIN
  employee AS manager ON manager.id = team_member.manager_id
```

| team_member.first_name | team_member.last_name | manager.first_name | manager.last_name |
| :--------------------- | :-------------------- | :----------------- | :---------------- |
| Jeff                   | Fiddler               | Kush               | Patel             |
| Quinn                  | Leong                 | Jeff               | Fiddler           |
| Shamayel               | Daoud                 | Jeff               | Fiddler           |
| Robert                 | Koeze                 | Shamayel           | Daoud             |
| Munyo                  | Frey                  | Quinn              | Leong             |
| Kelly                  | Chung                 | Shamayel           | Daoud             |

In all the examples you've covered thus far, JOINs were performed on two
different tables (presumably with two different names), which made it
easy to reference a specific column in a table. Since we only deal with
one table in a self join, we have to use **aliases**.

An alias is essentially a nickname for a table (or, in some cases, a
column). This is necessary because the query processor
needs to make a distinction between the duplicates of the same table to
JOIN them. Keep in mind, the keyword **AS** is not necessary to alias
tables or columns. The above SQL query could be rewritten:

```sql
SELECT
  team_member.first_name, team_member.last_name,
   manager.first_name, manager.last_name
FROM
  employee team_member
JOIN
  employee manager ON manager.id = team_member.manager_id
```

[query-pro]: https://github.com/appacademy/curriculum/blob/master/sql/readings/db-stack.md

---

# Formatting SQL Code

### SQL Conventions

Different programmers use different SQL conventions, but in
preparation for ActiveRecord and Rails, which have their own
conventions, you should:

-   Always name SQL tables **snake_case** and
    **pluralized**. (e.g., `musical_instruments`, `favorite_cats`)
-   If a `musician` belongs to a `band`, your `musicians` table will
    need to store a foreign key that refers to the `id` column in the
    `bands` table. The foreign key column should be named `band_id`.
-   Always have a column named `id`, and use it as the primary key for a
    table.

You must not write SQL all on a single line. It will be impossible to
read:

```sql
SELECT * FROM table_one LEFT OUTER table_two ON table_one.column_one = table_two.column_x WHERE (table_one.column_three > table_two.column_y ...
```

Here's an example of some well formatted SQL code:

```sql
SELECT
  table_two.column_one,
  table_two.column_two,
  table_two.column_three
FROM
  table_one
LEFT OUTER JOIN
  table_two ON table_one.column_one = table_two.column_x
WHERE
  (table_one.column_three > table_two.column_y
    AND another_condition IS NULL)
GROUP BY
  table_two.column_four
ORDER BY
  table_two.column_four
```

Notice that each component of the SQL statement starts with the
keyword aligned left. The body of each component is indented two
spaces. Complex `WHERE` clauses are parenthesized and indented two
spaces on the following line.

## Subqueries

Life gets complicated when you make subqueries. Here's how I do it:

```sql
SELECT
  bands.*
FROM
  bands
JOIN (
  SELECT
    albums.*
  FROM
    albums
  WHERE
    album.type = "POP"
  GROUP BY
    album.band_id
  HAVING
    COUNT(*) > 3
  ) AS pop_group_albums ON bands.id = pop_group_albums.band_id
WHERE
  band.leader_id IN (
    SELECT
      musicians.id
    FROM
      musicians
    WHERE
      musicians.birth_yr > 1940
  )
```

I put the leading paren on the prior line, indent the query two
spaces, and close with a trailing paren at the start of a new line. I
put the `ON` of a `JOIN` right after the closing paren.

## References

-   Based on the style guide [How I Write SQL][how-i-write-sql].

[how-i-write-sql]: http://www.craigkerstiens.com/2012/11/17/how-i-write-sql

---

## NULL and Ternary Logic in SQL

SQL uses **ternary logic**. This means that a conditional statement can
evaluate to `TRUE`, `FALSE` or `NULL` (unknown). Whaaaa? :open_mouth: And somehow `NULL`
is still 'falsy'? Unfortunately, this won't be the only time you run into logic
that defies intuition. _Stay tuned for Javascript quirks._

If we ask if a `NULL` value `== NULL`, we will always get false. This is
because `NULL` was derived to represent an unknown value. How can we know if
two unknowns are the same? We can't. Given that this sort of comparison doesn't
yield any useful information, always use `IS NULL` or `IS NOT NULL` in place of
the traditional (`==` or `!=`) comparisons.

---

# PostgreSQL Setup

## Installing Postgres

First, download and install [Postgres.app][postgres-app]. This is
already installed on the class Mac minis.

Follow the directions to add the
[command line tools][pg-command-line-tools] to your `$PATH` so that
you may use them in the terminal. This will involve adding
`PATH="/Applications/Postgres.app/Contents/Versions/9.4/bin:$PATH"` to your
`~/.bashrc` file. For older versions of Postgres.app, you may have to
add `PATH="/Applications/Postgres93.app/Contents/MacOS/bin:$PATH"` or `"PATH=/Applications/Postgres.app/Contents/MacOS/bin:$PATH"`.

This will allow programs like Rails to access your Postgres database.

You will have to rerun `.bashrc` for the `$PATH` to be updated. The
easiest way to do this is to restart the terminal (alternatively run
`source ~/.bashrc` to force `$PATH` to be reloaded). If for some
reason putting it in `.bashrc` didn't help, put it in `.bash_profile`;
that should work.

**Linux users:** The [Ubuntu wiki][pg-linux] can help. See especially
"Alternative Server Setup".

[postgres-app]: http://postgresapp.com/
[pg-command-line-tools]: http://postgresapp.com/documentation/cli-tools.html
[pg-windows]: http://netpie.wordpress.com/2011/03/17/setting-up-rails-3-with-postgresql-on-windows/
[pg-linux]: https://help.ubuntu.com/community/PostgreSQL

## Creating the DB

Make sure Postgres.app is running (it's in your Applications). You
should see a little elephant icon in your top bar.

First, we need to create the database. Postgres can support multiple
applications, each of which might be storing different kinds of data
in their own **database**. Applications shouldn't have access to each
other's databases, even though they are all managed by a single
**database server** (Postgres is the database server).

Let's create a blank database named `bank`:

```
~$ psql
psql (9.2.2)
Type "help" for help.

ruggeri=# CREATE DATABASE bank;
CREATE DATABASE
ruggeri=# \q
```

We can connect to our bank DB. We run the `psql` program, giving it
the name of the DB we want to connect to.

```
~$ psql bank
psql (9.2.2)
Type "help" for help.

bank=# \d
No relations found.
```

### Importing the data

There's no data imported yet, so our db is a lonely place (`\d` is a
special Postgres command to list the tables). Let's run the import
script (which sets up the tables and adds the initial
records). Download my version of the
[LearningSQLExample.sql][learning-sql-example] script (or [legacy version][learning-sql-example-legacy] for the 1st Edition); I've modified
it to work with Postgres.

[learning-sql-example]: http://assets.aaonline.io/fullstack/sql/demos/learning_sql_example_postgres.sql
[learning-sql-example-legacy]: http://assets.aaonline.io/fullstack/sql/demos/learning_sql_example_postgres_1st_ed.sql

The `learning-sql-example-postgres.sql` file contains SQL commands. To
run them we need to "pipe them in" to `psql bank`. This is what the
shell's `|` operation does: `cat learning-sql-example-postgres.sql | psql bank` takes the _output_ of the first command (`cat learning-sql-example-postgres.sql`, which just outputs the contents of
the file) and uses it as the _input_ of the command on the right
(`psql bank`, which runs SQL commands).

There will probably be some notices about creating implicit sequences
and indexes; you may ignore these.

## Making your first query

Let's check on the data:

```
~$ psql bank
psql (9.2.2)
Type "help" for help.

bank=# \d
                  List of relations
 Schema |          Name          |   Type   |  Owner
--------+------------------------+----------+---------
 public | account                | table    | ruggeri
 public | account_account_id_seq | sequence | ruggeri
 public | branch                 | table    | ruggeri
 public | branch_branch_id_seq   | sequence | ruggeri
 public | business               | table    | ruggeri
 public | customer               | table    | ruggeri
 public | customer_cust_id_seq   | sequence | ruggeri
 public | department             | table    | ruggeri
 public | department_dept_id_seq | sequence | ruggeri
 public | employee               | table    | ruggeri
 public | employee_emp_id_seq    | sequence | ruggeri
 public | individual             | table    | ruggeri
 public | officer                | table    | ruggeri
 public | officer_officer_id_seq | sequence | ruggeri
 public | product                | table    | ruggeri
 public | product_type           | table    | ruggeri
 public | transaction            | table    | ruggeri
 public | transaction_txn_id_seq | sequence | ruggeri
(18 rows)

bank=# SELECT * FROM customer LIMIT 2;
 cust_id |   fed_id    | cust_type_cd |       address       |   city    | state | postal_code
---------+-------------+--------------+---------------------+-----------+-------+-------------
       1 | 111-11-1111 | I            | 47 Mockingbird Ln   | Lynnfield | MA    | 01940
       2 | 222-22-2222 | I            | 372 Clearwater Blvd | Woburn    | MA    | 01801
(2 rows)
```

Yay! You're good to go!

## Potential Pitfalls

### `DYLD_LIBRARY_PATH`

**First, check with a TA or instructor**.

In order to access postgres from your Rails application, you may need
to add this line:

`export DYLD_LIBRARY_PATH="/Applications/Postgres.app/Contents/MacOS/lib:$DYLD_LIBRARY_PATH"`

Please do not add it unless (a) you have problems connecting from a
Rails app, (b) you made sure that running `psql` in the command line
**did work** (otherwise something else is the problem), and (c) your
TA gives you the okay.

This is usually a result of installing an old version of Postgres.app;
more recent versions shouldn't suffer this problem. You should first
try reinstalling Postgres.app.

### Shared Memory Problem

**First, check with a TA or instructor**.

There is another problem where postgres may fail to launch properly;
you won't be able to connect with `psql` (much less from Rails). To
diagnose the error, invoke postgres manually:

```
$ postgres -D ~/Library/Application\ Support/Postgres/var/ -p5432
7/7/12 8:24:23.017 PM com.heroku.postgres-service: server starting
7/7/12 8:24:23.018 PM Postgres: 75469 /Applications/Postgres.app/Contents/MacOS/bin/pg_ctl: Status 0
7/7/12 8:24:23.117 PM com.heroku.postgres-service: FATAL:  could not create shared memory segment: Invalid argument
7/7/12 8:24:23.117 PM com.heroku.postgres-service: DETAIL:  Failed system call was shmget(key=5432001, size=14499840, 03600).
7/7/12 8:24:23.117 PM com.heroku.postgres-service: HINT:  This error usually means that PostgreSQL's request for a shared memory segment exceeded your kernel's SHMMAX parameter.  You can either reduce the request size or reconfigure the kernel with larger SHMMAX.  To reduce the request size (currently 14499840 bytes), reduce PostgreSQL's shared memory usage, perhaps by reducing shared_buffers or max_connections.
7/7/12 8:24:23.117 PM com.heroku.postgres-service: 	If the request size is already small, it's possible that it is less than your kernel's SHMMIN parameter, in which case raising the request size or reconfiguring SHMMIN is called for.
7/7/12 8:24:23.117 PM com.heroku.postgres-service: 	The PostgreSQL documentation contains more information about shared memory configuration.
```

If you have the SHM error messages, first, see if this solution fixes things:

```
sudo sysctl -w kern.sysv.shmall=65536
sudo sysctl -w kern.sysv.shmmax=16777216
```

This is only a temporary fix; these settings will be lost when you
restart. For this reason, you want to edit the `/etc/sysctl.conf`
system file. Add these lines:

```
kern.sysv.shmall=65536
kern.sysv.shmmax=16777216
```

Now Postgres should work in the future.
