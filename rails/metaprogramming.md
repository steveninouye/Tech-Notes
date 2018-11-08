# Metaprogramming and Reflection

**Goals**:

* Learn how to use `send`
* Learn how to use `define_method`
* Learn how to use `method_missing`

## `send` and `define_method`

One of the powers of Ruby is **reflection** (also called
**introspection**): the ability for a program to examine itself.

For starters, we can ask an object what methods it will respond to:

```ruby
obj = Object.new
obj.methods
 => [:nil?, :===, :=~, :!~, :eql?, :hash, :<=>, :class, ...]
```

`Object#methods` returns an array of symbols, each the name of a
method that can be sent to the object. This is helpful for debugging,
but not super useful in production code.

More significantly, we can call a method by name:

```ruby
[].send(:count) # => 0
```

When is something like send useful? Why not just call the method the
normal way? Well, using **`send`** lets us write methods like this:

```ruby
def do_three_times(object, method_name)
  3.times { object.send(method_name) }
end

class Dog
  def bark
    puts "Woof"
  end
end

dog = Dog.new
do_three_times(dog, :bark)
```

We can even define new methods dynamically with **`define_method`**:

```ruby
class Dog
  # defines a class method that will define more methods; this is
  # called a **macro**.

  def self.makes_sound(name)
    define_method(name) { puts "#{name}!" }
  end

  makes_sound(:woof)
  makes_sound(:bark)
  makes_sound(:grr)
end

dog = Dog.new
dog.woof
dog.bark
dog.grr
```

**A couple notes:**

* The code inside `Dog` class is executed at the time Ruby defines the
  `Dog` class. `makes_sound` is called at class definition time,
  **not** each time a new `Dog` object is created.
* That makes sense, because the work of `makes_sound` sets up an
  instance method to be shared by **all** `Dog` objects. It's not
  instance-specific.
* Inside the definition of the `Dog` class, `makes_sound` knows to
  call the class method because `self == Dog` here.
* In the `makes_sound` macro, `self == Dog` because this is a `Dog`
  class method. `define_method` is implicitly called on `Dog`, adding
  a new method named `name`. The block is the code to run when the
  method is (later) called on an instance of `Dog`.

We don't write macros every single day, but they are frequently quite
useful. Some of the most famous macro methods are:

* `attr_accessor`: defines getter/setter methods given an instance variable name.
* `belongs_to`/`has_many`: defines a method to perform a SQL query to fetch associated objects.

## `method_missing`

When a method is called on an object, Ruby first looks for an existing
method with that name. If no such method exists, then it calls the
`Object#method_missing` method. It passes the method name (as a
symbol) and any arguments to `#method_missing`.

The default version simply raises an exception about the missing
method, but you may override `#method_missing` for your own purposes:

```ruby
class T
  def method_missing(*args)
    p args
  end
end
```

```ruby
T.new.adfasdfa(:a, :b, :c) # => [:adfasdfa, :a, :b, :c]
```

Here's a simple example:

```ruby
class Cat
  def say(anything)
    puts anything
  end

  def method_missing(method_name)
    method_name = method_name.to_s
    if method_name.start_with?("say_")
      text = method_name[("say_".length)..-1]

      say(text)
    else
      # do the usual thing when a method is missing (i.e., raise an
      # error)
      super
    end
  end
end

earl = Cat.new
earl.say_hello # puts "hello"
earl.say_goodbye # puts "goodbye"
```

Using `method_missing`, we are able to "define" an infinite number of
methods; we allow the user to call any method prefixed `say_` on a
`Cat`. This is very powerful; it isn't possible to do this using
`define_method` itself.

However, overriding `method_missing` can result in difficult to
understand/debug to code, and should not be your first resort when
attempting metaprogramming. Only if you want this infinite
expressability should you use `method_missing`; prefer a macro if the
user just wants to define a small set of methods.

### An Advanced Example: Dynamic Finders

What if we overrode `#method_missing` in `ActiveRecord::Base` to work for `#find_by_*` methods, so we could do the following:

```ruby
User.find_by_first_name_and_last_name("Ned", "Ruggeri")
User.find_by_username_and_state("ruggeri", "California")
```

We could do this by parsing the "missing" method name and combining the column names (separated by `and`s) with the given arguments. It might look something like this:

```ruby
class ActiveRecord::Base
  def method_missing(method_name, *args)
    method_name = method_name.to_s
    if method_name.start_with?("find_by_")
      # attributes_string is, e.g., "first_name_and_last_name"
      attributes_string = method_name[("find_by_".length)..-1]

      # attribute_names is, e.g., ["first_name", "last_name"]
      attribute_names = attributes_string.split("_and_")

      unless attribute_names.length == args.length
        raise "unexpected # of arguments"
      end

      search_conditions = {}
      attribute_names.length.times do |i|
        search_conditions[attribute_names[i]] = args[i]
      end

      # Imagine search takes a hash of search conditions and finds
      # objects with the given properties.
      self.search(search_conditions)
    else
      # complain about the missing method
      super
    end
  end
end
```

**NB:** Dynamic finders were actually a feature of Rails until just recently. Rails 4.2 deprecated (supported, but didn't recommend) dynamic finders, and as of Rails 5, they are no longer supported. Although, they are quite handy, they tend to lead to overly verbose code and are not very performant. For these reasons we also recommend against using them. Check out [this][defense-of-dynamic-finders] blog post if you'd like to learn more.

## Type Introspection

So far we focused on finding, defining, and calling methods at
runtime. We can also find class information:

```ruby
"who am i".class # => String
"who am i".is_a?(String) # => true
```

I commonly use `Object#class` when debugging or using pry to see what
kind of thing I'm dealing with, so that I can then know what class to
look up the documentation for.

Here we can see that even classes are objects in Ruby:

```ruby
Object.is_a?(Object) # => true
# such meta, wow
```

Deep. Let's dig deeper:

```ruby
Object.class # => Class
```

Okay, all classes are instances of a `Class` class.

```ruby
Class.superclass # => Module
Class.superclass.superclass # => Object
```

Classes are types of `Module`s (not important), which are
`Object`s. In Ruby everything is an `Object`, even `Class`es!

To summarize: `Object` is of type `Class`, which is a subclass of
`Object` itself. Whoa!

## Methods with Varying Argument Types

Say we have written a method `perform_get` that fetches a resource
over the internet. As a convenience to the user, we'd like
`perform_get` to take either a `String`, which is the literal URL to
fetch, or a hash, with the URL broken into parts

```ruby
perform_get("http://www.google.com/+")
perform_get(
  :scheme => :http,
  :host => "www.google.com",
  :path => "/+"
)
```

In the case where we give `perform_get` a hash, it's going to need to
do some extra work to construct the URL to get. How might this work?
Perhaps like so:

```ruby
def perform_get(url)
  if url.is_a?(Hash)
    # url is actually a hash of url options, call another method
    # to turn it into a string representation.
    url = make_url(url)
  end

  # ...
end
```

This is a quite common trick used by library writers to make their
methods much more flexible. You may not write a method like this
often, but as you grow more experienced, this kind of trick will come
in handy from time to time.

[defense-of-dynamic-finders]:http://chrisholtz.com/blog/in-defense-of-dynamic-finders

---------

# Class Instance Variables

You know all about instance variables:

```ruby
class Dog
  def initialize(name)
    @name = name
  end

  # could also use `attr_reader :name` to generate this.
  def name
    @name
  end
end
```

Inside a method, we can set an instance variable of the current
object. This is what we do inside the `initialize` instance method.

Recall that classes are objects, too. For instance, `Dog` itself is a
class. We can set instance variables on the `Dog` class object too:

```ruby
class Dog
  def self.all
    @dogs ||= []
  end
  
  def initialize(name)
    @name = name
    
    self.class.all << self
  end
end
```

In the class method `all`, we fetch/assign an instance variable
`dogs`. This stores an instance variable in the `Dog` object. As part
of the initialization of a `Dog` instance, we add the `Dog` instance
to the list of all `Dog`s. We can access all dogs through `Dog.all`:

```
d1 = Dog.new("Fido")
d2 = Dog.new("Fido 2.0")

p Dog.all
=> [#<Dog:0x007fe140a23928 @name="Fido">,
 #<Dog:0x007fe140a628d0 @name="Fido 2.0">]
```

Note that the `@dogs` variable in `Dog.all` works the same as any
other instance variable: setting or accessing `@dogs` will look inside
the current object (in this case, the `Dog` class object) and
set/fetch the instance variable.

When an instance variable is stored on a class, it is sometimes called
a **class instance variable**. Don't let the name wow you though;
we're just using a typical instance variable. This is similar to how
class methods are merely methods that are called on a `Class` object.

## Inheritance `@@`

For our purposes, the standard instance variable will typically be
enough. There is one downside: class instance variables don't interact
very nicely with inheritance. Let's take an example:

```ruby
class Corgi < Dog
end
```

Let's think what happens when we run `Corgi.new("Linus")`. Per the
definition of `initialize` in `Dog`, we will run `self.class.all <<
self`. `self.class` is `Corgi`; `Corgi` will have an `all` method by
virtue of inheriting from `Dog`.

The `all` method will look in `Corgi` for a `@dogs` instance
variable. Note that `Corgi` will not share the `@dogs` variable from
`Dog`. `Corgi` and `Dog` are different objects, so they do not share
instance variables. This means that `Corgi` will have its own `@dogs`
variable, and `Corgi`s will not be added to the `Dog`'s array of
`@dogs`.

That may not be what you want. Perhaps you would like that `Corgi`s be
added to the list of all `Dog`s. You can do this by switching from
`@dogs` to `@@dogs`; `@@dogs` is a **class variable**.

Class variables (not class **instance** variables) are shared between
super-class and subclass. Let's see this:

```ruby
class Dog
  def self.all
    @@dogs ||= []
  end
  
  def initialize(name)
    @name = name
    
    self.class.all << self
  end
end

class Husky < Dog
end

h = Husky.new("Rex")

Dog.all # => #<Husky:0x007f95421b5560 @name="Rex">
```

I should note: most of the classes you write won't be inherited
from. So you may want to eject the emotional baggage of `@@` and just
stick with the `@` variables you are familiar with, at least until `@`
doesn't work.

## Global variables

Bonus topic: buy two get one free! Global variables!

Global variables are prefixed with a `$`. Global variables are
top-level variables that live outside any class. They are accessible
anywhere:

```ruby
# this should have been a class variable though...
$all_dogs = []

class Dog
  def self.all
    $all_dogs
  end
  
  def initialize(name)
    @name = name
    
    $all_dogs << self
  end
end
```

Why couldn't we write `all_dogs = []`? The reason is that if we try to
create a **local** variable at the top level scope, it will be cleaned
up and removed when the source file is executed:

```
[1] pry(main)> require './dog'
=> true
[2] pry(main)> Dog.all
NameError: undefined local variable or method `all_dogs' for Dog:Class
        from: /Users/ruggeri/test.rb:5:in `all'
                from: (pry):2:in `__pry__'
```

Global variables, on the other hand, have permanence.

### Avoid global variables

Global variables are not very common to use; you should avoid them. I
rarely if ever use them. The reason is that since global variables
live outside any class, they aren't very object oriented. Data is
normally stored in one of two places:

* Inside an object (instance, class instance, and class variables)
* Inside a local variable; the local variable lives as long as the
  current method call.

If you need to access an object inside a method, it is typical to pass
the object into the method. If you need to return a result from a
method, it is typical to use `return` to pass it back. There is seldom
a reason to store things globally.

There are occasionally exceptions: sometimes an object will be useful
throughout your entire program, in which case you may want to make it
globally accessible. One classic example is the `$stdin` and `$stdout`
variables, which contain `File` objects (technically, `IO` objects,
but they're very similar) that you can use to read/write to the user.

Here's how `puts` and `gets` are defined:

```ruby
def puts(*args)
  $stdout.puts(*args)
end

def gets(*args)
  $stdin.gets(*args)
end
```

This eliminates most of the need to use these variables
explicitly. However, say you wanted to write your output differently
depending on whether the user was reading your output in a terminal or
dumping your output to a file. In Bash, they can specify this by either:

```
$ ruby program.rb # print to console
$ ruby program.rb > ./file_to_print_to # print to a file
```

You could use the `IO#isatty` method of `$stdout` to do this:

```ruby
if $stdout.isatty
  puts "I'm on a console!"
else
  puts "I'm on a file!"
end
```

Great. However, you're not likely to need to use global variables your
own self.

---------


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
- E.g. "There are many types of joins in SQL."

**RDBMS** - Relational Database Management System
- E.g. "Oracle is a popular RDBMS because of its enterprise scale."

**Query** - An operation that retrieves data from one or more tables; describes
desired data, leaving the database management system to plan, optimize, and
carry out the operations necessary to produce the results
- E.g. "Our query should retrieve all of the users and their photos."

**Three-Valued (Ternary) Logic** - A many-valued logic system with three truth
values: True, False, and Unknown (null)
- E.g. "SQL's use of ternary logic means that we must explicitly check for NULL."

**Transaction** - A unit of work performed against a database that is treated
in a coherent and reliable way; all of the work within a transaction must
succeed, or it is rolled back entirely, i.e. "all or nothing"
- E.g. "Because these operations are in a transaction, we can trust that
  our data will be consistent across both tables."

**Normalization** - An approach to database storage that practices storing
references to information that lives in a different location rather than duplicating it and
storing it in multiple places; antonym: denormalization
- E.g. "SQL implementations are typically normalized and therefore avoid
  repetition of the same data in multiple tables."
