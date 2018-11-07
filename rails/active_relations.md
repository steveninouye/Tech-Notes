# Active Relation

## Queries are lazy

Querying methods like `group`, `having`, `includes`, `joins`, `select`
and `where` return an object of type `ActiveRecord::Relation`. The
`Relation` object looks a lot like an `Array`; like an `Array` you can
iterate through it or index into it.

There is one major difference from an `Array`: the contents of
`Relation` are not fetched until needed. This is called **laziness**:

```ruby
users = User.where('likes_dogs = ?', true) # no fetch yet!

# performs fetch here
users.each { |user| puts "Hello #{user.name}" }
# does not perform fetch; result is "cached"
users.each { |user| puts "Hello #{user.name}" }
```

The `Relation` is not evaluated (a database query is not fired) until
the results are needed. In this example, on the first line where
`users` is assigned, the result is not yet needed. Only when we call
`each` the first time do we actually need the results and the query is
forced.

### Caching

After the query is run, the results are **cached** by `Relation`; they
are stored for later re-use. Subsequent calls to `each` will not fire
a query; they will instead use the prior result. This is an advantage
because we can re-use the result without constantly hitting the
database over-and-over.

This can sometimes result in unexpected behavior. First, note that
when accessing a relation (e.g. `user1.posts`), a `Relation` object
is returned. The relation is itself cached inside model object (e.g.
`user1`) so that future invocations of the association will not hit
the DB.

```ruby
# Fires a query; `posts` relation stored in `user1`.
p user1.posts
# => []
p user1.posts.object_id
# => 70232180387400

Post.create!(
  user_id: user1.id, # THIS LINE IS KEY, BECAUSE IT TIES THE POST TO THE USER IN THE DATABASE
  title: 'Title',
  body: 'Body body body'
)

# Does not fire a query; uses cached `posts` relation, which itself
# has cached the results.
p user1.posts
# => []
p user1.posts.object_id
# => 70232180387400
```

Here `user1.posts` fires a query the first time. The second time,
`user1.posts` uses the prior result. The cached result, however, is
out of date; in between the first time we called `user1.posts` and the
second time, we added a new `Post` for the user. This is not reflected
in the `user1.posts` variable.

This behavior can be surprising, but it actually is not a common
issue. You can always force an association to be reloaded (ignoring
any cached value) by calling `user1.posts(true)`. You may also call
`user1.reload` to throw away all cached association
relations. However, this is seldom necessary.

## Laziness and stacking queries

Caching results makes sense: it saves DB queries. But why is laziness
a good thing?

Laziness allows us to build complex queries. Let's see an example:

```ruby
georges = User.where('first_name = ?', 'George')
georges.where_values_hash
# => {first_name: 'George'}

george_harrisons = georges.where('last_name = ?', 'Harrison')
george_harrisons.where_values_hash
# => {first_name: 'George', last_name: 'Harrison'}

p george_harrisons
```

In this somewhat silly example, we call `where` twice. The first call
to `where` returns a `Relation` which knows to filter by `first_name`
(the condition is stored in the `where_values_hash` attribute). Next, we
call `where` on this `Relation`; this produces a new `Relation` object
which will know to filter by both `first_name` and `last_name` (you
can see that `where_values_hash` was extended).

Note that the additional `where` created a new `Relation`; the
original `georges` is not changed.

The first `Relation` returned by the first `where` is never
evaluated. Instead, we build a second `Relation` from it. Here
laziness helps us; it lets us build up a query by chaining query
methods, none of which are executed until the chain is finished being
built and evaluated.

Just like `where` has a `where_values_hash` attribute, there are similar (private)
accessors for `includes_values`, `joins_values`, etc. You won't ever
access the attributes directly, but you can see how `Relation` builds
up the query by storing each of the conditions. When the `Relation`
needs to be evaluated, ActiveRecord looks at each of these values to
build the SQL to execute.

## Forcing evaluation

If you wish to force the evaluation of a `Relation`, you may call
`load`. This will force
evaluation if it hasn't been done already. Using `to_a` will return
an *actual* array.

Some other methods, like `count` will also force the evaluation of a query.

*Count vs Length vs Size*

While all three of these methods will return the number of records in the evaluated query, they do so in slightly different ways. When we call `count` on a relation, we will execute a SQL query (`SELECT COUNT(*) FROM ...`). Calling `length`, on the other hand, will load the entire collection into memory, convert it to an array, and then call `Array#length` on it. If used incorrectly, this could be an incredibly expensive operation (imagine if we have millions of users and call `User.all.length`).

If all we care about is returning the number of records, and nothing else, `count` will be preferred because it allows us to avoid loading the collection into memory. However, if we are going to load the collection into memory anyway, `length` will be preferred because we can avoid having to do another query altogether (since the collection will be cached in memory anyway). `size` is sort of a combination of both methods - if the collection is already loaded, it will count the elements without making a new query and, if not, will execute an additional query.




## References

* http://api.rubyonrails.org/classes/ActiveRecord/Relation.html
* http://api.rubyonrails.org/classes/ActiveRecord/QueryMethods.html
* http://blog.mitchcrowe.com/blog/2012/04/14/10-most-underused-activerecord-relation-methods/
*  https://mensfeld.pl/2014/09/activerecord-count-vs-length-vs-size-and-what-will-happen-if-you-use-it-the-way-you-shouldnt/

-------

# Joins in Active Record

This reading refers heavily to the [JoinsDemo][joins-demo]. Please download the zip file and follow along as you read.  

[joins-demo]: https://github.com/appacademy/curriculum/blob/master/sql/demos/joins_demo

## Schema overview:

Familiarize yourself with the schema for this project:

```ruby
# db/schema.rb
ActiveRecord::Schema.define(:version => 20130126200858) do
  create_table "comments", force: :cascade do |t|
    # force: true drops the table before creating it.
    t.text     "body"
    t.integer  "author_id"
    t.integer  "post_id"
    t.integer  "parent_comment_id"
    t.datetime "created_at",        null: false
    t.datetime "updated_at",        null: false
  end

  create_table "posts", force: :cascade do |t|
    t.string   "title"
    t.text     "body"
    t.integer  "author_id"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    # null: false means this field must be filled.
  end

  create_table "users", force: :cascade do |t|
    t.string   "user_name"
    t.string   "first_name"
    t.string   "last_name"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end
end
```

## `ActiveRecord::Base::joins`

When we were writing raw SQL we learned that the `JOIN` clause was a powerful tool that allowed us to combine rows from different tables to find information about related data in these tables. For example, if I wanted to find all of the comments made by a user with the user_name 'tamboer' I could make a query like this:

```SQL
SELECT
  comments.*
FROM
  comments
JOIN
  users ON users.id = comments.author_id
WHERE
  users.user_name = 'tamboer';
```

To fire the exact same query using ActiveRecord methods we would write `Comment.joins(:author).where(users: { user_name: 'tamboer' })`.

Let's break this line down. First we start with the `Comment` model. This already tells ActiveRecord that the `FROM` clause of our query should use the `comments` table. Then we call the `joins` method and pass in `:author` as an argument.

 ***NOTE:*** When you are using the `joins` method you do not pass in the name of a table. Instead, you are passing in the name of an association that has been defined. Since the association has already been defined with a class name, foreign key, and primary key, ActiveRecord can determine exactly how to join your two tables together.

 Then finally in our call to the `where` method we pass a hash of `{ users: { user_name: 'tamboer' } }`. We've seen simpler uses of `where` in which we just pass a key-value pair, with the key referencing a column name and the value being the value we want to match. When we pass a nested hash like this, the outermost key is the name of a table and the innermost hash has a key that is a column name in that table. When we join two tables together, we want to be specific about which table the column names we are matching come from. We could also replace our original call to `where` with `where("users.user_name = 'tamboer'")`. It's important to note that even though we used the name of the association in our call to `joins` we still use the real name of the table when we are referencing one of its columns.

### More complex joins
If you want to join more than one table to another, the syntax for the `ActiveRecord::Base::joins` method doesn't get much more complicated. If you have a through association defined on a model, it's as simple as just using the name of the association.

`User.joins(:post_feedback)` generates this query

```SQL
SELECT
  users.*
FROM
  users
JOIN
  posts ON posts.author_id = users.id
JOIN
  comments ON comments.post_id = posts.id
```

The `:post_feedback` association has already been set up to go from the users table, through the posts table, and to the comments table. If we didn't have this through association defined for us we could write `User.joins(posts: :comments)`. Check out the [Active Record Guides on joins](http://guides.rubyonrails.org/active_record_querying.html#joining-tables) for more info on this syntax!

We can also call the `joins` method on an `ActiveRecord::Relation` object. For example:

```ruby
post = Post.find(1)
post.comments.joins(:author)
# The line above generates the query
# SELECT comments.* FROM comments JOIN users ON users.id = comments.author_id WHERE comments.post_id = 1
```

The `WHERE comments.post_id = 1` portion of that query came from the fact that we used the comments association on a post with an id of 1. This filtering remains even when we add our call to `joins` afterwards. Gotta love laziness and stacking!

## Using Select with Join

If we don't add a call to `select` on to our call to `joins` we'll only have access to columns from the primary table in our query.

```ruby
users = User.joins(:posts)
users[0].user_name # => 'ruggeri'
users[0].title # => undefined method 'title' for User object
```

This is because the select statement was defaulted to `users.*`. If we wanted the `User` objects we got back from our query to have access to attributes from the posts table we would have to explicitly say that.

```ruby
users = User.joins(:posts).select("users.*, posts.*")
users[0].user_name # => "ruggeri"
users[0].title # => "First post!"
```

If you just examine the user object, you won't see the title attribute.

```ruby
users[0]
=> id: 1,
 user_name: "ruggeri",
 first_name: "Ned",
 last_name: "Ruggeri",
 created_at: Tue, 03 Apr 2018 14:39:57 UTC +00:00,
 updated_at: Tue, 03 Apr 2018 14:39:57 UTC +00:00>
```

This is because the `inspect` method for `User` objects only show attributes that are column names from the users table. But if we use the `#attributes` method we can see all of the attributes that our user has access to.

```ruby
users[0].attributes
=>{"id"=>1,
   "user_name"=>"ruggeri",
   "first_name"=>"Ned",
   "last_name"=>"Ruggeri",
   "created_at"=>Tue, 03 Apr 2018 14:39:57 UTC +00:00,
   "updated_at"=>Tue, 03 Apr 2018 14:39:57 UTC +00:00,
   "title"=>"First post!",
   "body"=>"First posting is fun!",
   "author_id"=>1}
```

We can also alias attributes in a call to `select`. This is especially useful if we have columns from two different tables with the same name. Let's use created_at as a trivial example.

```ruby
Post.create!(author_id: 1, title: "Who loves active record?", body: "If you like active record say yeah!")
users = User.joins(:posts).select("users.*, posts.created_at as post_creation_time")

users[0].attributes
=> {"id"=>1,
 "user_name"=>"ruggeri",
 "first_name"=>"Ned",
 "last_name"=>"Ruggeri",
 "created_at"=>Tue, 03 Apr 2018 14:39:57 UTC +00:00,
 "updated_at"=>Tue, 03 Apr 2018 14:39:57 UTC +00:00,
 "post_creation_time"=>"2018-04-03 14:39:57.563795"}

 users[1].attributes
 {"id"=>1,
 "user_name"=>"ruggeri",
 "first_name"=>"Ned",
 "last_name"=>"Ruggeri",
 "created_at"=>Tue, 03 Apr 2018 14:39:57 UTC +00:00,
 "updated_at"=>Tue, 03 Apr 2018 14:39:57 UTC +00:00,
 "post_creation_time"=>"2018-04-03 20:04:13.596728"}
```

First we should note that we have an attribute called `"post_creation_time"` that we can now access on our user objects because that's what we gave as an alias in our call to `select`. Second we should note that users[0] and users[1] both have the attributes of our ruggeri user but they have different values for `post_creation_time`. Don't forget that if we're joining the users table to the posts table it is possible to have two rows that refer to the same user with different posts. In raw SQL the resulting table from selecting * and joining users to posts would look something like this:

<table>
  <tr>
    <td>id</td>
    <td>user_name</td>
    <td>first_name</td>
    <td>last_name</td>
    <td>created_at</td>
    <td>updated_at</td>
    <td>id</td>
    <td>title</td>
    <td>body</td>
    <td>author_id</td>
    <td>created_at</td>
    <td>updated_at</td>
  </tr>
  <tr>
    <td>1</td>
    <td>ruggeri</td>
    <td>Ned</td>
    <td>Ruggeri</td>
    <td>2018-04-03 14:39:57.486206</td>
    <td>2018-04-03 14:39:57.486206</td>
    <td>id</td>
    <td>First post!</td>
    <td>First posting is fun!</td>
    <td>1</td>
    <td>2018-04-03 14:39:57.563795</td>
    <td>2018-04-03 14:39:57.563795</td>
  </tr>
  <tr>
    <td>1</td>
    <td>ruggeri</td>
    <td>Ned</td>
    <td>Ruggeri</td>
    <td>2018-04-03 14:39:57.486206</td>
    <td>2018-04-03 14:39:57.486206</td>
    <td>id</td>
    <td>First post!</td>
    <td>First posting is fun!</td>
    <td>1</td>
    <td>2018-04-03 20:04:13.596728</td>
    <td>2018-04-03 20:04:13.596728</td>
  </tr>
</table>



The first row becomes the first element in our ActiveRecord::Relation object and the second row become the second element in our object.

Check out the [Active Record Guides on joins](http://guides.rubyonrails.org/active_record_querying.html#joining-tables) for more info!

----------

This reading refers heavily to the [JoinsDemo][joins-demo]. Please download the zip file and follow along as you read.  

[joins-demo]: http://assets.aaonline.io/fullstack/sql/demos/joins_demo.zip

## Schema overview:

Familiarize yourself with the schema for this project:

```ruby
# db/schema.rb
ActiveRecord::Schema.define(:version => 20130126200858) do
  create_table "comments", force: :cascade do |t|
    # force: true drops the table before creating it.
    t.text     "body"
    t.integer  "author_id"
    t.integer  "post_id"
    t.integer  "parent_comment_id"
    t.datetime "created_at",        null: false
    t.datetime "updated_at",        null: false
  end

  create_table "posts", force: :cascade do |t|
    t.string   "title"
    t.text     "body"
    t.integer  "author_id"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    # null: false means this field must be filled.
  end

  create_table "users", force: :cascade do |t|
    t.string   "user_name"
    t.string   "first_name"
    t.string   "last_name"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end
end
```

## The N+1 selects problem

Let's write some code to get the number of comments per `Post` by a
`User`:

```ruby
# app/models/user.rb
class User < ApplicationRecord
  # ...

  def n_plus_one_post_comment_counts
    posts = self.posts
    # SELECT *
    #   FROM posts
    #  WHERE posts.author_id = ?
    #
    # where `?` gets replaced with `user.id`

    post_comment_counts = {}
    posts.each do |post|
      # This query gets performed once for each post. Each db query
      # has overhead, so this is very wasteful if there are a lot of
      # `Post`s for the `User`.
      post_comment_counts[post] = post.comments.length
      # SELECT *
      #   FROM comments
      #  WHERE comments.post_id = ?
      #
      # where `?` gets replaced with `post.id`
    end

    post_comment_counts
  end
end
```

This code looks fine at the first sight. But the problem lies with the
total number of queries executed. The above code executes 1 query (to
find the user's posts) + "N" queries (one per post to find the
comments) for N+1 queries in total.

This is inefficient. Consider if a user had 10,000 posts: we'd make
10,000 queries for comments. We will next see a way to perform one
query for all the `comment`s, instead of N queries. Even though we
will receive the same number of `comments` rows total, doing all the
work in one query is much more efficient. Each query to the database
has some fixed overhead associated with it, so batching up work into
one query is a major efficiency win.

### Solution to N+1 queries problem

The solution to this problem is to fetch all the `Comment`s for all
the `Post`s in one go, rather than fetch them one-by-one for each
`Post`.

Active Record lets you specify associations to prefetch. When you use
these associations later, the data will already have been fetched and
won't need to be queried for. To do this, use the `includes`
method. If you use `includes` to prefetch data (e.g., `posts =
user.posts.includes(:comments)`), a subsequent call to access the
association (e.g., `posts[0].comments`) won't fire another DB query;
it'll use the prefetched data.

Revisiting the above case, we could rewrite `post_comment_counts` to
use eager loading:

```ruby
# app/model/user.rb
class User < ApplicationRecord
  # ...

  def includes_post_comment_counts
    # `includes` *prefetches the association* `comments`, so it doesn't
    # need to be queried for later. `includes` does not change the
    # type of the object returned (in this example, `Post`s); it only
    # prefetches extra data.
    posts = self.posts.includes(:comments)
    # Makes two queries:
    # SELECT *
    #   FROM posts
    #  WHERE post.author_id = ?
    #
    # where `?` is replaced with `user.id`.
    #
    # ...and...
    #
    # SELECT *
    #   FROM comments
    #  WHERE comments.post_id IN ?
    #
    # where `?` is replaced with `self.posts.map(&:id)`, the `Array`
    # of `Post` ids.

    post_comment_counts = {}
    posts.each do |post|
      # doesn't fire a query, since already prefetched the association
      # way better than N+1
      #
      # NB: if we write `post.comments.count` ActiveRecord will try to
      # be super-smart and run a `SELECT COUNT(*) FROM comments WHERE
      # comments.post_id = ?` query. This is because ActiveRecord
      # understands `#count`. But we already fetched the comments and
      # don't want to go back to the DB, so we can avoid this behavior
      # by calling `Array#length`.
      post_comment_counts[post] = post.comments.length
    end

    post_comment_counts
  end
end
```

The above code will execute just **2** queries, as opposed to **N+1**
queries. When there are many posts, this is a major win.

Normally you should wait until you see performance problems before
returning to optimize code ("premature optimization is the root of all
evil"). However, N+1 queries are so egregious that you should avoid
them from the beginning. Consider N+1 queries an error; they are never
the right solution.

### Complex includes

You can eagerly load as many associations as you like:

```ruby
comments = user.comments.includes(:post, :parent_comment)
```

Then both the `post` and `parent_comment` associations are eagerly
loaded. Neither `comments[0].post` nor `comments[0].parent_comment` will
hit the DB; they've been prefetched.

We can also do "nested" prefetches:

```ruby
posts = user.posts.includes(:comments => [:author, :parent_comment])
first_post = posts[0]
```

This not only prefetches `first_post.comments`, it also will prefetch
`first_post.comments[0]` and even `first_post.comments[0].author` and
`first_post.comments[0].parent_comment`.

## Joining Tables

Let's see how we can use `joins` to solve our N + 1 problem.

We've seen how to eagerly load associated objects to dodge the N+1
queries problem. There is another problem we may run into: `includes`
returns lots of data: it returns every `Comment` on every `Post` that
the `User` has written. This may be many, many comments. In the case
of counting comments per post, the `Comment`s themselves are useless,
we just want to count them.

We're doing too much in Ruby: we want to push some of the counting
work to SQL so that the database does it, and we receive just `Post`
objects with associated comment counts. This is another major use of
`joins`:

```ruby
# app/models/user.rb
class User
  # ...

  def joins_post_comment_counts
    # We use `includes` when we need to prefetch an association and
    # use those associated records. If we only want to *aggregate* the
    # associated records somehow, `includes` is wasteful, because all
    # the associated records are pulled down into the app.
    #
    # For instance, if a `User` has posts with many, many comments, we
    # would pull down every single comment. This may be more rows than
    # our Rails app can handle. And we don't actually care about all
    # the individual rows, we just want the count of how many there
    # are.
    #
    # When we want to do an "aggregation" like summing the number of
    # records (and don't care about the individual records), we want
    # to use `joins`.

    posts_with_counts = self
      .posts
      .select("posts.*, COUNT(*) AS comments_count") # more in a sec
      .joins(:comments)
      .group("posts.id") # "comments.post_id" would be equivalent
    # in SQL:
    #   SELECT posts.*, COUNT(*) AS comments_count
    #     FROM posts
    #    JOINS comments
    #       ON comments.post_id = posts.id
    #    WHERE posts.author_id = #{self.id}
    # GROUP BY posts.id
    #
    # As we've seen before using `joins` does not change the type of
    # object returned: this returns an `Array` of `Post` objects.
    #
    # But we do want some extra data about the `Post`: how many
    # comments were left on it. We can use `select` to pick up some
    # "bonus fields" and give us access to extra data.
    #
    # Here, I would like to have the database count the comments per
    # post, and store this in a column named `comments_count`. The
    # magic is that ActiveRecord will give me access to this column by
    # dynamically adding a new method to the returned `Post` objects;
    # I can call `#comments_count`, and it will access the value of
    # this column:

    posts_with_counts.map do |post|
      # `#comments_count` will access the column we `select`ed in the
      # query.
      [post.title, post.comments_count]
    end
  end
end
```

### OUTER JOINs

The default for `joins` is to perform an `INNER JOIN`. In the previous
example we will not return any posts with zero comments because there
will be no comment row to join the post against.

If we want to include posts with zero comments, we need to do an outer
join. We can do this like so:

```ruby
#In Rails 5+
posts_with_counts = self
  .posts
  .select('posts.*, COUNT(comments.id) AS comments_count')
  .left_outer_joins(:comments) #still the name of the association
  .group('posts.id') # "comments.post_id" would be equivalent

#Before Rails 5 (just for old time's sake)
posts_with_counts = self
  .posts
  .select('posts.*, COUNT(comments.id) AS comments_count') # more in a sec
  .joins('LEFT OUTER JOIN comments ON posts.id = comments.post_id') # raw sql, so comments here is the name of the table
  .group('posts.id') # "comments.post_id" would be equivalent
```

## References

* http://blog.arkency.com/2013/12/rails4-preloading/
* http://stackoverflow.com/questions/97197/what-is-the-n1-selects-problem
* http://stackoverflow.com/questions/6246826/how-do-i-avoid-multiple-queries-with-include-in-rails
* http://blog.bigbinary.com/2013/07/01/preload-vs-eager-load-vs-joins-vs-includes.html

----------

# Scopes

It's common to write commonly used queries as a **scope**. A scope is
just a fancy name for an `ActiveRecord::Base` class method that
constructs all or part of a query and then returns the resulting
[`Relation` object][relation-reading].
Remember, our models inherit from `ApplicationRecord`, which, in turn, inherits from `ActiveRecord::Base`.

Use scopes to keep your query code DRY: move frequently-used queries
into a scope. It will also make things much more readable by giving a
convenient name of your choosing to the query.

```ruby
class Post < ApplicationRecord
  def self.by_popularity
    self
      .select('posts.*, COUNT(*) AS comment_count')
      .joins(:comments)
      .group('posts.id')
      .order('comment_count DESC')
  end
end
```

We can now use `Post.by_popularity`:

```sql
irb(main):001:0> posts = Post.by_popularity
  Post Load (5.7ms)  SELECT posts.*, COUNT(*) AS comment_count FROM "posts" INNER JOIN "comments" ON "comments"."post_id" = "posts"."id" GROUP BY posts.id ORDER BY comment_count DESC
=> #<ActiveRecord::Relation [#<Post id: 12>, #<Post id: 5>, ...]>
irb(main):002:0> posts.first.comment_count
=> 45
```

Because it returns a `Relation` object and not just the results, we can
continue to tack query methods onto it. This makes scopes super flexible.
Suppose we only want the 5 most popular posts:

```sql
irb(main):003:0> posts = Post.by_popularity.limit(5)
  Post Load (1.4ms)  SELECT posts.*, COUNT(*) AS comment_count FROM "posts" INNER JOIN "comments" ON "comments"."post_id" = "posts"."id" GROUP BY posts.id ORDER BY comment_count DESC LIMIT 5
=> #<ActiveRecord::Relation [#<Post id: 12>, #<Post id: 5>, ...]>
irb(main):004:0> posts.count
=> 5
```

Another awesome thing about scopes is that you can use them with
associations. Through a bit of Rails magic, we can call
`user.posts.by_popularity`:

```sql
irb(main):005:0> posts = User.first.posts.by_popularity
  User Load (0.7ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT 1
  Post Load (28.7ms)  SELECT posts.*, COUNT(*) AS comment_count FROM "posts" INNER JOIN "comments" ON "comments"."post_id" = "posts"."id" WHERE "posts"."user_id" = $1 GROUP BY posts.id ORDER BY comment_count DESC  [["user_id", 1]]
=> #<ActiveRecord::AssociationRelation #<Post id: 1>, #<Post id: 7>, ...]>
irb(main):006:0> posts.first.comment_count
=> 8
```

Remember that `User#posts` returns a [`Relation`
object][relation-reading] too. `Relation` objects know what kind of
model objects they should contain. Because of this they will actually
assume the class methods (including scopes) that are available on that
model class. In this case, `User#posts` contains `Post` objects, so we
can chain scopes like `Post::by_popularity` directly on the result of
`User#posts`. Cool!

One final note: You will often see a shorthand syntax for defining
scopes using the `scope` method. Read more about this and other cool
stuff like scope chaining in [the docs][scope-docs].

[relation-reading]: https://github.com/appacademy/curriculum/blob/master/sql/readings/relation.md
[scope-docs]: http://apidock.com/rails/ActiveRecord/NamedScope/ClassMethods/scope

--------

# More on Querying

## Dynamic Finders

We've seen how to use `where` to retrieve an array of ActiveRecord objects
matching some conditions. Sometimes, you want to find the single
object that matches some criteria; you want to dispense with the array
(which in this case will be either empty, or length 1).
We use `::find` and `::find_by` for this:

```ruby
Application.find_by(email_address: 'ned@appacademy.io')
# returns the record whose email_address matches "ned@appacademy.io"

Application.find(4)
# returns the record with id 4
```

`::find` accepts a single argument: the id of the record you're looking for. `::find_by` accepts an options hash, which allows us to specify as many criteria as necessary.

An important difference to note is that `::find` will raise an `ActiveRecord::RecordNotFound` exception if you search for a nonexistent record, whereas `::find_by` will simply return `nil`. Don't let that scare you; just be aware of the difference! Prefer `::find` for looking up records by id.

## `order`, `group`, and `having`

### Ordering

To retrieve records from the database in a specific order, you can use
the `order` method.

```ruby
Client.order('orders_count ASC, created_at DESC').all
```

### Group, Having

You can apply `GROUP BY` and `HAVING` clauses.

```ruby
UserPost
  .joins(:likes)
  .group('posts.id')
  .having('COUNT(*) > 5')
```

### Aggregations

You can perform all the typical aggregations:

* `Client.count`
* `Orders.sum(:total_price)`
* `Orders.average(:total_price)`
* `Orders.minimum(:total_price)`
* `Orders.maximum(:total_price)`

## Finding by SQL

We've seen how to get ActiveRecord to do fairly advanced stuff for us.

By the time I'd need these methods, I'd probably just use
`find_by_sql`, honestly. ActiveRecord has its limits; it's great for
reducing boilerplate SQL queries, but after a certain point you should
drop down and just use SQL yourself. Be flexible; don't expect too
much from ActiveRecord. Even if you have to drop to SQL for a few
monster queries, ActiveRecord has saved you a lot of work on all the
easy queries.

The main problem with trying to take ActiveRecord too far is that it
can become difficult to understand what kind of query it will generate
or how it will do what you ask. The more you ask ActiveRecord to do,
the more you have to trust that you express yourself properly, and the
more you have to think about whether ActiveRecord will do the right
thing. Sometimes simpler is better.

If you'd like to use your own SQL to find records in a table you can
use `find_by_sql`. The `find_by_sql` method will return an array of
objects. For example you could run this query:

```ruby
Case.find_by_sql(<<-SQL)
  SELECT
    cases.*
  FROM
    cases
  JOIN (
    -- the five lawyers with the most clients
    SELECT
      lawyers.*
    FROM
      lawyers
    LEFT OUTER JOIN
      clients ON lawyers.id = clients.lawyer_id
    GROUP BY
      lawyers.id
    SORT BY
      COUNT(clients.*)
    LIMIT 5
  ) ON ((cases.prosecutor_id = lawyer.id)
         OR (cases.defender_id = lawyer.id))
SQL
```

Time to betray some ignorance: I don't know how I would do this with
ActiveRecord and not SQL (or if it's possible!). At the very least I don't know
how I'd do it in one query.

Even if I spent the time to torture myself and figure it out, I'd only
be punishing the next person to read my code. Even if they understand
my intent, it'd be a dog to figure out whether I'm constructing the
query correctly in ActiveRecord.

**NB:**

If you have a parameterized query that you need to pass values into,
you need to pass all the arguments, *including the query*, in an array
to `find_by_sql`. Consider this example from the
[`find_by_sql`][find-by-sql] API page:

```ruby
Post.find_by_sql([
  'SELECT title FROM posts WHERE author = ? AND created > ?',
  author_id,
  start_date
])
```

Notice that the query and the values to insert are all in one array argument.

## References

* http://guides.rubyonrails.org/active_record_querying.html

[find-by-sql]: http://api.rubyonrails.org/classes/ActiveRecord/Querying.html#method-i-find_by_sql
