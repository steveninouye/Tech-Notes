# Migrations

## Overview

We've discussed SQL databases, so we know that programs can store and
pull out data from them. Data fetched from the DB can then be used to
populate the attributes of Ruby objects.

As a program is written, the structure of the database will evolve. We
would like some way to track the evolution of the database schema, so
that this is tracked along with our code in our git repository.

Additionally, because we develop our app on our own machine, with our
own local development database, but later deploy our application to a
server running a production database, we need a way to record the
transformations we've made locally, so that they may be "played back"
and performed on the server database when we deploy our code.

Database *migrations* are a solution to these problems. A migration is
a file containing Ruby code that describes a set of changes applied to
the database. It may create or drop tables; it may add or remove
columns from a table. Each new set of changes is written inside a new
migration file, which is checked into the repository. ActiveRecord
will take responsibility for performing the necessary migrations when
you ask it.

------

## Rolling back migrations

We'll start off this reading with a warning: **We strongly discourage students from ever rolling back. This is because rolling back is not an option when working on something in production.** That being said- this reading exists to teach you about what rolling back is and how it can be used. Occasionally you will make a mistake when writing a migration. If you
have already run the migration then you cannot just edit the migration
and run the migration again: Rails will think it has already run the
migration and so will do nothing when you run `rails
db:migrate`. That's because the timestamp is in `schema_migrations`.

You must first *roll back* the migration, which reverses the change (by
calling the `down`), if that is possible. This will undo the changes
and remove the timestamp from `schema_migrations`.

It is a common mistake
to begin editing the migration before rolling it back. Then, when you
try to roll back, ActiveRecord tries to rollback the migration **as it
is currently written**. This causes problems because the edited
migration does not correspond to the migration that was actually
previously run. You need to be careful of this: wait until after
rollback to edit.

To rollback the most recent migration, run `rails db:rollback`. You may
now edit the migration file and rerun.

-------

# Object Relational Mapping

## Overview

### Motivation

We've discussed how to manage changes to a SQL database through
migrations. Now we'd like to start using the records stored in that
database.

We've previously worked directly with a database by writing SQL. One
downside was that this embedded SQL code is in our Ruby code. Though this
works, it would be nice to use Ruby syntax as much as possible.

Also, when we fetched data from our SQL database, the data was
returned in generic `Hash` objects. For instance, if our database was
setup like this:

```sql
  CREATE TABLE cars (make VARCHAR(255), model VARCHAR(255), year INTEGER);
  INSERT INTO cars (model, make, year)
    ("Toyota", "Camry", 1997),
    ("Toyota", "Land Cruiser", 1989),
    ("Citroen", "DS", 1969);
```

And we wrote the following ruby code to fetch the data:

```ruby
require 'sqlite3'
db = SQLite3::Database.new('cars.db')
db.results_as_hash = true
db.type_translation = true

cars = db.execute('SELECT * FROM cars')
# => [
#  {"make" => "Toyota", "model" => "Camry", "year" => 1997},
#  {"make" => "Toyota", "model" => "Land Cruiser", "year" => 1989},
#  {"make" => "Citroen", "model" => "DS", "year" => 1969}
# ]
```

That works nicely, but what if we wanted to store and load objects of
a `Car` class? Instead of retrieving generic `Hash` objects, we want
to get back instances of our `Car` class. Then we could call `Car`
methods like `go` and `vroom`. How would we translate between the
world of Ruby classes and rows in our database?

### What is an ORM?

An *object relational mapping* is the system that translates between
SQL records and Ruby (or Java, or Lisp...) objects. The ActiveRecord
ORM translates rows from your SQL tables into Ruby objects on fetch,
and translates your Ruby objects back to rows on save. The ORM also
empowers your Ruby classes with convenient methods to perform common
SQL operations: for instance, if the table `physicians` contains a
foreign key referring to `offices`, ActiveRecord will be able to
provide your `Physician` class a method, `#office`, which will fetch
the associated record. Using ORM, the properties and relationships of
the objects in an application can be easily stored and retrieved from
a database without writing SQL statements directly and with less
overall database access code.

-----

# Validations

## Overview

This guide teaches you how to validate that objects are correctly
filled out before they go into the database. Validations are used to
ensure that only valid data is saved into your database. For example,
it may be important to your application to ensure that every user
provides a valid email address and mailing address. Validations keep
garbage data out.

### Validations vs. Constraints

We need to make an important distinction here. Rails *validations* are **not** the same as database *constraints*, though they are conceptually similar. Both try to ensure data integrity and consistency, but *validations* operate in your Ruby code, while *constraints* operate in the database. So the basic rule is:

* *Validations* are defined inside **models**.
* *Constraints* are defined inside **migrations**.

#### Use Constraints

We've seen how to write some database constraints in SQL (`NOT NULL`,
`FOREIGN KEY`, `UNIQUE INDEX`). These are enforced by the database and
are very strong. Not only will they keep bugs in our Rails app from
putting bad data into the database, they'll also stop bad data coming
from other sources (SQL scripts, the database console, etc). We will
frequently use simple DB constraints like these to ensure data
consistency.

However, for complicated validations, DB constraints can be tortuous to
write in SQL. Also, when a DB constraint fails, a generic error is
thrown to Rails (`SQLException`). In general, Rails will not handle errors like these, and a web user's request will fail with an
ugly `500 Internal Server Error`.

#### Use Validations

For this reason, DB constraints are not appropriate for validating user
input. If a user chooses a previously chosen username, they should not
get a 500 error page; Rails should nicely ask for another name. This is
what *model-level validations* are good at.

Model-level validations live in the Rails world. Because we write them
in Ruby, they are very flexible, database agnostic, and convenient to
test, maintain and reuse. Rails provides built-in helpers for common
validations, making them easy to add. Many things we can validate in
the Rails layer would be very difficult to enforce at the DB layer.

#### Use Both!

Often you will use both together. For example, you might use a `NOT
NULL` constraint to guarantee good data while also taking advantage of
the user messaging provided by a corresponding `presence: true`
validation.

Perhaps a better example of this would be uniqueness. A `uniqueness:
true` validation is good for displaying useful feedback to users, but it
cannot actually guarantee uniqueness. It operates inside a single server
process and doesn't know what any other servers are doing. Two servers
could submit queries to the DB with conflicting data at the same time
and the validation would not catch it (This happens *surprisingly
often*). Because a `UNIQUE` constraint operates in the database and not
in the server, it will cause one of those requests to fail (albeit
gracelessly), preserving the integrity of your data.

## When does validation happen?

Whenever you call `save`/`save!` on a model, ActiveRecord will first
run the validations to make sure the data is valid to be persisted
permanently to the DB. If any validations fail, the object will be
marked as invalid and Active Record will not perform the `INSERT` or
`UPDATE` operation. This keeps invalid data from being inserted into
the database.

To signal success saving the object, `save` will return `true`;
otherwise `false` is returned. `save!` will instead raise an error if
the validations fail.

---------

# Indexing

## Overview

Suppose we have a simple query:

```sql
SELECT *
FROM Users
WHERE Name = 'Mike'
```

Now think back to our Big-O discussion from last week and consider the
time complexity for this query. If we have a table of 100 rows, it's
going to have to check every one of those rows for the name `'Mike'`.
This is referred to as a "table scan" and is `O(n)` time complexity. Now
imagine that we actually have ten million users, and that we're making
that query 100 times/sec. Our database is going to crash, our website
will go down in flames, and our bosses will fire our sorry,
non-optimizing behinds.

This is why it's important to index columns that are heavily used for
lookups in queries. When you index a column, it creates a sorted data
structure with pointers to the actual table. Since it's sorted, lookups
can use binary search, which as you recall runs in `O(log n)` time. Log
base 2 of 10 million is about 23, so as you can imagine, this improves
database performance (and our career prospects) *dramatically*.

Remember though, before you go all index happy, that indices do have a
cost. They make writes (`INSERT`s, `DELETE`s, and `UPDATE`s) a little
more taxing because the index must be updated. Furthermore,
optimizations made outside of the bottleneck are no optimizations at
all. So it's important to index the *right* things, or you might as well
have indexed *nothing*.

On that note, **foreign keys** are pretty much always a good choice for
indexing because they're frequently used in both `WHERE` clauses and in
`JOIN` conditions, both of which can be incredibly taxing when not
indexed.
