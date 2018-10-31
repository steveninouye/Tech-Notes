# RSpec Syntax & Mechanics

RSpec is distributed in a gem called 'rspec', which is actually a
meta-gem that packages three other gems: rspec-core, rspec-expectations,
and rspec-mocks. We'll spend most of our time on rspec-core and
rspec-expectations.

## File Organization

By convention, tests are kept in the `spec` folder and your
application code will be kept in a `lib` folder. Tests for `hello.rb`
will be written in a file called `hello_spec.rb`. Your directory
structure should look like this:

```
my_cool_project
  lib/
    hello.rb
  spec/
    hello_spec.rb
```

## Requiring Dependencies

Each spec will usually be limited to testing a single file and so
will require the file at the top of the spec. It will also have to
require the rspec gem.

```ruby
# hello_spec.rb

require 'rspec'
require 'hello'

describe '#hello_world' do

end
```

Note that RSpec will by default include the `lib/` folder in the
require path so that we can use `require` and not `require_relative`.
This is another reason to follow the convention of using `lib/` and
`spec/` for your code and your tests, respectively.

## Organization & Syntax

Here's what a simple 'Hello, World!' spec might look like.

```ruby
# hello_spec.rb

require 'rspec'
require 'hello'

describe "#hello_world" do
  it "returns 'Hello, World!'" do
    expect(hello_world).to eq("Hello, World!")
  end
end
```

And the code that would make it pass:

```ruby
# hello.rb

def hello_world
  "Hello, World!"
end
```

### `describe` and `it`

`it` is RSpec's most basic test unit. All of your actual individual
tests will go inside of an `it` block.

`describe` is RSpec's unit of organization. It gathers together
several `it` blocks into a single unit, and, as we'll see, allows you
to set up some context for blocks of tests.

Both `describe` and `it` take strings as arguments. For `describe`,
use the name of the method you're testing (use "#method" for instance
methods, and "::method" for class methods). For `it`, you should
describe the behavior that you're testing inside that `it` block.

`describe` can also take a constant that should be the name of the
class or module you're testing (i.e. `describe Student do`).

You can nest `describe` blocks arbitrarily deep. When nesting, also
consider the use of `context`, which is an alias for `describe` that
can be a bit more descriptive. Prefer `context` when it makes sense.

```ruby
describe Student do
  context 'when a current student' do
    ...
  end

  context 'when graduated' do
    ...
  end
end
```

### `expect`

`describe` and `it` organize your tests and give them descriptive
labels. `expect` will actually be doing the work of testing your
code.

Its task is to *match* between a value your code generates and an
expected value. You can specify the way in which it will match.

There are negative and positive constructions:

```ruby
expect(test_value).to ...
expect(test_value).to_not ...
```

There are two constructions to expect: with an argument and with a
block. We'll prefer the argument construction except when the block
construction is necessary.

```ruby
describe Integer do
  describe '#to_s' do
    it 'returns string representations of integers' do
      expect(5.to_s).to eq('5')
    end
  end
end
```

The block construction is necessary when you want to test that a
certain method call will throw an error:

```ruby
describe '#sqrt' do
  it 'throws an error if given a negative number' do
    expect { sqrt(-3) }.to raise_error(ArgumentError)
  end
end
```

RSpec comes with a variety of matchers that come after `expect().to`
or `expect().to_not`. The most common and staightforward matchers are
straight equality matchers.

`expect(test_value).to eq(expected_value)` will see if `test_value ==
expected_value`. `expect(test_value).to be(expected_value)` will test if
`test_value` is the same object as `expected_value`.

```
expect('hello').to eq('hello') # => passes ('hello' == 'hello')
expect('hello').to be('hello') # => fails (strings are different objects)
```

At this point, you know the absolute basics of RSpec's syntax. Head on
over to the GitHub pages and read both of the READMEs. This is required
reading.

* [rspec-core][rspec-core]
* [rspec-expectations][rspec-expectations] (note the variety of expectation
  matchers available to you)

Head back here once you're done.

### `before`

Welcome back! Hope you've learned a lot more about what RSpec allows
you to do.

One thing that we often want to do is set up the context in which our
specs will run. We usually do this in a `before` block.

```ruby
describe Chess do
  let(:board) { Board.new }

  describe '#checkmate?' do
    context 'when in checkmate' do
      before(:each) do
        board.make_move([3, 4], [2, 3])
        board.make_move([1, 2], [4, 5])
        board.make_move([5, 3], [5, 1])
        board.make_move([6, 3], [2, 4])
      end

      it 'should return true' do
        expect(board.checkmate?(:black)).to be true
      end
    end
  end
end
```

`before` can be used as either `before(:each)` or `before(:all)`.
You'll almost always use `before(:each)`. `before(:each)` will execute
the block of code before each spec in that `describe` block. The nice
thing about it is that state is not shared - that is, you start fresh
on every spec, even if inside your spec (i.e. in your `it` block) you
manipulate some object that the `before` block set up for you.

`before(:all)` on the other hand does share state across specs and
for that reason, we avoid using it. It makes our tests a bit brittle by
making specs dependent on one another (and dependent on the order in
which specs are run).

There are also `after(:each)` and `after(:all)` counterparts.

### Pending Specs

Sometimes, you may want to write out a bunch of descriptions for specs
without actually writing the bodies of those specs. If you simply
leave the test bodies empty, it'll look like they're all passing. If
you fail them, then it'll look like you actually have test code
written that is currently failing.

What to do? Make the specs pending.

How?

Leave off the `do...end` from the `it`.

```ruby
describe '#valid_move?' do
  it 'should return false for wrong colored pieces'
  it 'should return false for moves that are off the board'
  it 'should return false for moves that put you in check'
end
```

## Additional Notes

Don't use `!=`.  Rspec does not support `expect(actual) != expected`.
Instead use `expect(actual).to eq expected` or `expect(actual).to_not eq
expected`.

On predicate syntatic sugar: With all predicates, you can strip off the
? and tack on a "be_" to make an expectation.  For example,
`expect(Array.empty?).to be true` is equivalent to `expect(Array).to be_empty`.

Note that RSpec can even conjugate verbs when necessary. For
instance, to test that a Hash `has_key?`, you can simplify:

    expect(my_hash.has_key?(my_key)).to eq(true)

to:

    expect(my_hash).to have_key(my_key)

## Intro Assessment Spec

Go back to the spec from the intro assessment. Read
the spec file and make sure everything makes sense to you.

## Additional Resources

* The [RSpec docs][rspec-docs] are a good resource. Knowing
  RSpec well will let you write beautiful specs. Note that the
  RSpec docs are specs themselves.

[rspec-docs]: https://www.relishapp.com/rspec/rspec-core/v/2-4/docs
[rspec-core]: https://github.com/rspec/rspec-core
[rspec-expectations]: https://github.com/rspec/rspec-expectations

-------------------------------
-------------------------------
# rspec-core [![Build Status](https://secure.travis-ci.org/rspec/rspec-core.svg?branch=master)](http://travis-ci.org/rspec/rspec-core) [![Code Climate](https://codeclimate.com/github/rspec/rspec-core.svg)](https://codeclimate.com/github/rspec/rspec-core)

rspec-core provides the structure for writing executable examples of how your
code should behave, and an `rspec` command with tools to constrain which
examples get run and tailor the output.

## Install

    gem install rspec      # for rspec-core, rspec-expectations, rspec-mocks
    gem install rspec-core # for rspec-core only
    rspec --help

Want to run against the `master` branch? You'll need to include the dependent
RSpec repos as well. Add the following to your `Gemfile`:

```ruby
%w[rspec rspec-core rspec-expectations rspec-mocks rspec-support].each do |lib|
  gem lib, :git => "https://github.com/rspec/#{lib}.git", :branch => 'master'
end
```

## Contributing

Once you've set up the environment, you'll need to cd into the working
directory of whichever repo you want to work in. From there you can run the
specs and cucumber features, and make patches.

NOTE: You do not need to use rspec-dev to work on a specific RSpec repo. You
can treat each RSpec repo as an independent project.

* [Build details](BUILD_DETAIL.md)
* [Code of Conduct](CODE_OF_CONDUCT.md)
* [Detailed contributing guide](CONTRIBUTING.md)
* [Development setup guide](DEVELOPMENT.md)

## Basic Structure

RSpec uses the words "describe" and "it" so we can express concepts like a conversation:

    "Describe an order."
    "It sums the prices of its line items."

```ruby
RSpec.describe Order do
  it "sums the prices of its line items" do
    order = Order.new

    order.add_entry(LineItem.new(:item => Item.new(
      :price => Money.new(1.11, :USD)
    )))
    order.add_entry(LineItem.new(:item => Item.new(
      :price => Money.new(2.22, :USD),
      :quantity => 2
    )))

    expect(order.total).to eq(Money.new(5.55, :USD))
  end
end
```

The `describe` method creates an [ExampleGroup](http://rubydoc.info/gems/rspec-core/RSpec/Core/ExampleGroup).  Within the
block passed to `describe` you can declare examples using the `it` method.

Under the hood, an example group is a class in which the block passed to
`describe` is evaluated. The blocks passed to `it` are evaluated in the
context of an _instance_ of that class.

## Nested Groups

You can also declare nested groups using the `describe` or `context`
methods:

```ruby
RSpec.describe Order do
  context "with no items" do
    it "behaves one way" do
      # ...
    end
  end

  context "with one item" do
    it "behaves another way" do
      # ...
    end
  end
end
```

Nested groups are subclasses of the outer example group class, providing
the inheritance semantics you'd want for free.

## Aliases

You can declare example groups using either `describe` or `context`.
For a top level example group, `describe` and `context` are available
off of `RSpec`. For backwards compatibility, they are also available
off of the `main` object and `Module` unless you disable monkey
patching.

You can declare examples within a group using any of `it`, `specify`, or
`example`.

## Shared Examples and Contexts

Declare a shared example group using `shared_examples`, and then include it
in any group using `include_examples`.

```ruby
RSpec.shared_examples "collections" do |collection_class|
  it "is empty when first created" do
    expect(collection_class.new).to be_empty
  end
end

RSpec.describe Array do
  include_examples "collections", Array
end

RSpec.describe Hash do
  include_examples "collections", Hash
end
```

Nearly anything that can be declared within an example group can be declared
within a shared example group. This includes `before`, `after`, and `around`
hooks, `let` declarations, and nested groups/contexts.

You can also use the names `shared_context` and `include_context`. These are
pretty much the same as `shared_examples` and `include_examples`, providing
more accurate naming when you share hooks, `let` declarations, helper methods,
etc, but no examples.

## Metadata

rspec-core stores a metadata hash with every example and group, which
contains their descriptions, the locations at which they were
declared, etc, etc. This hash powers many of rspec-core's features,
including output formatters (which access descriptions and locations),
and filtering before and after hooks.

Although you probably won't ever need this unless you are writing an
extension, you can access it from an example like this:

```ruby
it "does something" do |example|
  expect(example.metadata[:description]).to eq("does something")
end
```

### `described_class`

When a class is passed to `describe`, you can access it from an example
using the `described_class` method, which is a wrapper for
`example.metadata[:described_class]`.

```ruby
RSpec.describe Widget do
  example do
    expect(described_class).to equal(Widget)
  end
end
```

This is useful in extensions or shared example groups in which the specific
class is unknown. Taking the collections shared example group from above, we can
clean it up a bit using `described_class`:

```ruby
RSpec.shared_examples "collections" do
  it "is empty when first created" do
    expect(described_class.new).to be_empty
  end
end

RSpec.describe Array do
  include_examples "collections"
end

RSpec.describe Hash do
  include_examples "collections"
end
```

## A Word on Scope

RSpec has two scopes:

* **Example Group**: Example groups are defined by a `describe` or
  `context` block, which is eagerly evaluated when the spec file is
  loaded. The block is evaluated in the context of a subclass of
  `RSpec::Core::ExampleGroup`, or a subclass of the parent example group
  when you're nesting them.
* **Example**: Examples -- typically defined by an `it` block -- and any other
  blocks with per-example semantics -- such as a `before(:example)` hook -- are
  evaluated in the context of
  an _instance_ of the example group class to which the example belongs.
  Examples are _not_ executed when the spec file is loaded; instead,
  RSpec waits to run any examples until all spec files have been loaded,
  at which point it can apply filtering, randomization, etc.

To make this more concrete, consider this code snippet:

``` ruby
RSpec.describe "Using an array as a stack" do
  def build_stack
    []
  end

  before(:example) do
    @stack = build_stack
  end

  it 'is initially empty' do
    expect(@stack).to be_empty
  end

  context "after an item has been pushed" do
    before(:example) do
      @stack.push :item
    end

    it 'allows the pushed item to be popped' do
      expect(@stack.pop).to eq(:item)
    end
  end
end
```

Under the covers, this is (roughly) equivalent to:

``` ruby
class UsingAnArrayAsAStack < RSpec::Core::ExampleGroup
  def build_stack
    []
  end

  def before_example_1
    @stack = build_stack
  end

  def it_is_initially_empty
    expect(@stack).to be_empty
  end

  class AfterAnItemHasBeenPushed < self
    def before_example_2
      @stack.push :item
    end

    def it_allows_the_pushed_item_to_be_popped
      expect(@stack.pop).to eq(:item)
    end
  end
end
```

To run these examples, RSpec would (roughly) do the following:

``` ruby
example_1 = UsingAnArrayAsAStack.new
example_1.before_example_1
example_1.it_is_initially_empty

example_2 = UsingAnArrayAsAStack::AfterAnItemHasBeenPushed.new
example_2.before_example_1
example_2.before_example_2
example_2.it_allows_the_pushed_item_to_be_popped
```

## The `rspec` Command

When you install the rspec-core gem, it installs the `rspec` executable,
which you'll use to run rspec. The `rspec` command comes with many useful
options.
Run `rspec --help` to see the complete list.

## Store Command Line Options `.rspec`

You can store command line options in a `.rspec` file in the project's root
directory, and the `rspec` command will read them as though you typed them on
the command line.

## Get Started

Start with a simple example of behavior you expect from your system. Do
this before you write any implementation code:

```ruby
# in spec/calculator_spec.rb
RSpec.describe Calculator do
  describe '#add' do
    it 'returns the sum of its arguments' do
      expect(Calculator.new.add(1, 2)).to eq(3)
    end
  end
end
```

Run this with the rspec command, and watch it fail:

```
$ rspec spec/calculator_spec.rb
./spec/calculator_spec.rb:1: uninitialized constant Calculator
```

Address the failure by defining a skeleton of the `Calculator` class:

```ruby
# in lib/calculator.rb
class Calculator
  def add(a, b)
  end
end
```

Be sure to require the implementation file in the spec:

```ruby
# in spec/calculator_spec.rb
# - RSpec adds ./lib to the $LOAD_PATH
require "calculator"
```

Now run the spec again, and watch the expectation fail:

```
$ rspec spec/calculator_spec.rb
F

Failures:

  1) Calculator#add returns the sum of its arguments
     Failure/Error: expect(Calculator.new.add(1, 2)).to eq(3)

       expected: 3
            got: nil

       (compared using ==)
     # ./spec/calcalator_spec.rb:6:in `block (3 levels) in <top (required)>'

Finished in 0.00131 seconds (files took 0.10968 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/calcalator_spec.rb:5 # Calculator#add returns the sum of its arguments
```

Implement the simplest solution, by changing the definition of `Calculator#add` to:

```ruby
def add(a, b)
  a + b
end
```

Now run the spec again, and watch it pass:

```
$ rspec spec/calculator_spec.rb
.

Finished in 0.000315 seconds
1 example, 0 failures
```

Use the `documentation` formatter to see the resulting spec:

```
$ rspec spec/calculator_spec.rb --format doc
Calculator
  #add
    returns the sum of its arguments

Finished in 0.000379 seconds
1 example, 0 failures
```

## Also see

* [https://github.com/rspec/rspec](https://github.com/rspec/rspec)
* [https://github.com/rspec/rspec-expectations](https://github.com/rspec/rspec-expectations)
* [https://github.com/rspec/rspec-mocks](https://github.com/rspec/rspec-mocks)
* [https://github.com/rspec/rspec-rails](https://github.com/rspec/rspec-rails)


# `subject` and `let`

## `subject` and `it`

To test a class, you will often want to instantiate an instance of the object to
test it out. In this case, you may want to define a `subject` for your tests.

```ruby
describe Robot do
  subject { Robot.new }

  it "satisfies some expectation" do
    expect(subject).to # ...
  end
end
```

You can also declare a `subject` with a name:

```ruby
describe Robot do
  subject(:robot) { Robot.new }

  it "position should start at [0, 0]" do
    expect(robot.position).to eq([0, 0])
  end

  describe "move methods" do
    it "moves left" do
      robot.move_left
      expect(robot.position).to eq([-1, 0])
    end
  end
end
```

In addition to the name, `subject` also accepts a block that constructs the
subject. You can do any necessary setup inside the block.

The `it` block is a test. It runs the code, and the test fails if the
`expect` fails. In the first test, we `expect` that the position is
`[0, 0]`. In the second test we move the robot, and then expect the
position to have changed.

## `let`

`subject` lets us define the subject of our tests. Sometimes we also want to
create other objects to interact with the subject. To do this, we use `let`.
`let` works just like `subject`, but whereas `subject` is the focus of the test,
`let` defines helper objects. Another difference is that there can only be one
(unnamed) `subject` (if you declare a second `subject`, the value of `subject`
inside of your `it` blocks will use the more recent definition). On the other
hand, you can define many helper objects through `let`.

```ruby
describe Robot do
  subject(:robot) { Robot.new }
  let(:light_item) { double("light_item", :weight => 1) }
  let(:max_weight_item) { double("max_weight_item", :weight => 250) }

  describe "#pick_up" do
    it "does not add item past maximum weight of 250" do
      robot.pick_up(max_weight_item)

      expect do
        robot.pick_up(light_item)
      end.to raise_error(ArgumentError)
    end
  end
end
```

`let` defines a method (e.g. `light_item`, `max_weight_item`) that
runs the block provided once for each spec in which it is called.

You may see that you have the option of using instance variables in a
`before` block to declare objects accessible to specs, but we'll
avoid defining instance variables in specs. Always prefer `let`.
Here's a [SO post][stack-overflow-let] that clearly describes why
that is.

Here's a [blog post][dry-up-rspec] with some nice examples using
`let` - note how the author uses it in conjunction with `subject`
(some fancy and clean stuff).

[stack-overflow-let]: http://stackoverflow.com/questions/5359558/when-to-use-rspec-let
[dry-up-rspec]:http://benscheirman.com/2011/05/dry-up-your-rspec-files-with-subject-let-blocks/

### `let` does not persist state

You might read that `let` memoizes its return value. Memoization means
that the first time the method is invoked, the return value is cached
and that same value is returned every subsequent time the method is
invoked within the same scope. Since every `it` is a different scope,
`let` does not persist state between those specs.

An example:

```ruby
class Cat
  attr_accessor :name

  def initialize(name)
    @name = name
  end
end

describe "Cat" do
  let(:cat) { Cat.new("Sennacy") }

  describe "name property" do
    it "returns something we can manipulate" do
      cat.name = "Rocky"
      expect(cat.name).to eq("Rocky")
    end

    it "does not persist state" do
      expect(cat.name).to eq("Sennacy")
    end
  end
end

# => All specs pass
```

# RSpec Order of Operations

RSpec is particular about the order in which we invoke its various
methods. Look at this code:

```ruby
RSpec.describe Deck do
  describe '#initialize' do
    it 'initializes with 52 cards' do
      subject(:deck) { Deck.new } # nope
      expect(deck.count).to eq(52)
    end
  end
end
```

Seems reasonable enough, right? But this won't run, and it doesn't give
us a very helpful error message, either:

```
1) Deck#initialize initializes with 52 cards
     Failure/Error: subject(:deck) { Deck.new }
     ArgumentError:
       wrong number of arguments (1 for 0)
```
The problem is that we are trying to declare our subject at the top of
our `it` block; RSpec requires that the subject be declared outside of
your `it` blocks. This test will run successfully:

```ruby
RSpec.describe Deck do
  describe '#initialize' do
    subject(:deck) { Deck.new } # yup

    it 'initializes with 52 cards' do
      expect(deck.count).to eq(52)
    end
  end
end
```

This sort of ordering requirement applies to all of RSpec's methods; you
can't just toss in a `describe`, `it`, `expect`, `subject`, `let`, or
`before` block wherever you might naturally want to put it. RSpec
enforces a hierarchy/ordering of its methods, and you need to arrange
your blocks within the context of that structure. If you simply keep
this in mind and emulate the patterns illustrated in previous chapters,
you will be fine.

Below is an example of RSpec written with the correct order of operations

```ruby
RSpec.describe Sloth do
  subject(:sloth) { Sloth.new("Herald") }

  describe "#run" do
    context "when a valid direction is given" do
      it "returns a string that includes the direction" do
        expect(sloth.run("north")).to include("north")
      end
    end

    context "when an incorrect direction is given" do
      it "raises ArgumentError" do
        expect { sloth.run("somewhere") }.to raise_error(ArgumentError)
      end
    end

  end
end
```

# Test Doubles

## A Preface of Great Interest

When we write unit tests, we want each of our specs to test just one
thing. This can be a little complicated when we write classes that
interact with other classes. For example, imagine

```ruby
class Order
  def initialize(customer)
    @customer = customer
  end

  def send_confirmation_email
    email(
      to: @customer.email_address,
      subject: "Order Confirmation",
      body: self.summary
    )
  end
end
```

Here an `Order` object needs a `Customer` object; the associated
`Customer` object is used, for instance, when we try to call the
`#send_confirmation_email`. In particular, if we want to test
`#send_confirmation_email`, it looks like we'll have to supply `Order`
a `Customer` object.

```ruby
RSpec.describe Order do
  subject(:order) do
    customer = Customer.new(
      :first_name => "Ned",
      :last_name => "Ruggeri",
      :email_address => "ned@appacademy.io"
    )
    Order.new(customer)
  end

  it "sends email successfully" do
    expect do
      subject.send_confirmation_email
    end.not_to raise_exception
  end
end
```

This is troublesome because a spec for `#send_confirmation_email`
should only test the `#send_confirmation_email` method, not
`Customer#email_address`. But the way we've written this spec, if
there's a problem with `Customer#email_address`, a spec for
`Order#send_confirmation_email` will also break, even though it should
have nothing to do with `Customer#email_address`. This will clutter up
your log of spec failures.

Another problem is if `Order` and `Customer` both have methods that
interact with the other. If we write the `Customer` specs and methods
first, then we'll need a functioning `Order` object first for our
`Customer` to interact with. But we're supposed to TDD `Order`; we'll
need to have written specs for `Order`, but this requires a
`Customer`...

Finally, it can be a pain to construct a `Customer` object; we had to
specify a bunch of irrelevant fields here. Other objects can be even
harder to construct, which means we can end up wasting a lot of time
building an actual `Customer`, when an object that merely "looks like"
a `Customer` would have been sufficient.

We want to write our tests in isolation of other classes: their bugs
or whether they've even been implemented yet. The answer to this is to
use **doubles**.

## Test doubles

A test double (also called a **mock**) is a fake object that we can
use to create the desired isolation. A double takes the place of
outside, interacting objects, such as `Customer`. We could write the
example above like so:

```ruby
#IMPLEMENTATION
class Order
    def initialize(customer, product)
        @customer = customer
        @product = product
    end

    def charge_customer
        @customer.debit_account(@product.cost)
    end
end

#RSPEC FILE
RSpec.describe Order do
  let(:customer) { double("customer") }
  subject(:order) { Order.new(customer) }

  it "sends email successfully" do
    allow(customer).to receive(:email_address).and_return("ned@appacademy.io")

    expect do
      order.send_confirmation_email
    end.to_not raise_exception
  end
end
```

We create the double by simply calling the `double` method (we give it
a name for logging purposes). This creates an instance of
`RSpec::Mocks::Mock`. The double is a blank slate, waiting for us to
add behaviors to it.

A method **stub** stands in for a method; `Order` needs `customer`'s
`email_address` method, so we create a stub to provide it. We do this
by calling `allow(double).to receive(:method)`, passing a symbol with
the name of the method that we want to stub. The `and_return` method
takes the return value that the stubbed method will return when called
as its parameter.

The `customer` double simulates the `Customer#email_address` method,
without actually using any of the `Customer` code. This totally
isolates the test from the `Customer` class; we don't use `Customer`
at all. We don't even need to have the `Customer` class defined.

The `customer` object is not a real `Customer`; it's an instance of
`Mock`. But that won't bother the `Order#send_confirmation_email`
method. As long as the object that we pass responds to an
`email_address` message, everything will be fine.

There's also a one-line version of creating a double and specifying
stub methods.

```ruby
let(:customer) { double("customer", :email_address => "ned@appacademy.io") }
```

## Method Expectations

If the tested object is supposed to call methods on other objects as
part of its functionality, we should test that the proper methods are
called. To do this, we use method expectations. Here's an example:

```ruby
RSpec.describe Order
  let(:customer) { double('customer') }
  let(:product) { double('product', :cost => 5.99) }
  subject(:order) { Order.new(customer, product) }

  it "subtracts item cost from customer account" do
    expect(customer).to receive(:debit_account).with(5.99)
    order.charge_customer
  end
end
```

Here we want to test that when we call `charge_customer` on an `Order`
object, it tells the `customer` to subtract the item price from the
customer's account. We also specify that we should check that we have
passed `#debit_account` the correct price of the product.

Notice that we set the message expectation before we actually kick off
the `#charge_customer` method. Expectations need to be set up in
advance.

## Integration tests

Mocks let us write unit tests that isolate the functionality of a
single class from other outside classes. This lets us live up to the
philosophy of unit tests: in each spec, test one thing only.

Unit tests specify how an object should interact with other
objects. For instance, our `Order#charge_customer` test made sure that
the order sends a `debit_account` message to its customer.

What if the `Customer` class doesn't have a `#debit_account` method?
Perhaps instead the method is called `Customer#subtract_funds`. Then
in real life, with a real `Customer` object, our
`Order#charge_customer` method will crash when it tries to call
`#debit_account`. What spec is supposed to catch this error?

The problem here is a mismatch in the interface expected by `Customer`
and the interface provided by `Order`. This kind of error won't be
caught by a unit test, because the purpose of unit test is to test
classes in isolation.

We need a higher level of testing that's intended to verify that
`Order` and `Customer` are on the same page: that `Order` tries to
call the right method on `Customer`, which does the thing that `Order`
expects.

This kind of test is called an **integration test**. In integration
tests, we use real objects instead of mocks, so that we can verify
that all the classes interact correctly. A thorough test suite will
have both unit and integration tests. The unit tests are very specific
and are meant to isolate logical problems within a class; the
integration tests are larger in scope and are intended to check that
objects interact properly.

## Resources

* The double facilities are provided in the submodule of RSpec called
  rspec-mocks. You can check out their [github][rspec-mocks-github]
  which has a useful README.

[rspec-mocks-github]: https://github.com/rspec/rspec-mocks
-------------------------------------
-------------------------------------

# RSpec Expectations [![Build Status](https://secure.travis-ci.org/rspec/rspec-expectations.svg?branch=master)](http://travis-ci.org/rspec/rspec-expectations) [![Code Climate](https://codeclimate.com/github/rspec/rspec-expectations.svg)](https://codeclimate.com/github/rspec/rspec-expectations)

RSpec::Expectations lets you express expected outcomes on an object in an
example.

```ruby
expect(account.balance).to eq(Money.new(37.42, :USD))
```

## Install

If you want to use rspec-expectations with rspec, just install the rspec gem
and RubyGems will also install rspec-expectations for you (along with
rspec-core and rspec-mocks):

    gem install rspec

Want to run against the `master` branch? You'll need to include the dependent
RSpec repos as well. Add the following to your `Gemfile`:

```ruby
%w[rspec-core rspec-expectations rspec-mocks rspec-support].each do |lib|
  gem lib, :git => "https://github.com/rspec/#{lib}.git", :branch => 'master'
end
```

If you want to use rspec-expectations with another tool, like Test::Unit,
Minitest, or Cucumber, you can install it directly:

    gem install rspec-expectations

## Contributing

Once you've set up the environment, you'll need to cd into the working
directory of whichever repo you want to work in. From there you can run the
specs and cucumber features, and make patches.

NOTE: You do not need to use rspec-dev to work on a specific RSpec repo. You
can treat each RSpec repo as an independent project.

- [Build details](BUILD_DETAIL.md)
- [Code of Conduct](CODE_OF_CONDUCT.md)
- [Detailed contributing guide](CONTRIBUTING.md)
- [Development setup guide](DEVELOPMENT.md)

## Basic usage

Here's an example using rspec-core:

```ruby
RSpec.describe Order do
  it "sums the prices of the items in its line items" do
    order = Order.new
    order.add_entry(LineItem.new(:item => Item.new(
      :price => Money.new(1.11, :USD)
    )))
    order.add_entry(LineItem.new(:item => Item.new(
      :price => Money.new(2.22, :USD),
      :quantity => 2
    )))
    expect(order.total).to eq(Money.new(5.55, :USD))
  end
end
```

The `describe` and `it` methods come from rspec-core.  The `Order`, `LineItem`, `Item` and `Money` classes would be from _your_ code. The last line of the example
expresses an expected outcome. If `order.total == Money.new(5.55, :USD)`, then
the example passes. If not, it fails with a message like:

    expected: #<Money @value=5.55 @currency=:USD>
         got: #<Money @value=1.11 @currency=:USD>

## Built-in matchers

### Equivalence

```ruby
expect(actual).to eq(expected)  # passes if actual == expected
expect(actual).to eql(expected) # passes if actual.eql?(expected)
expect(actual).not_to eql(not_expected) # passes if not(actual.eql?(expected))
```

Note: The new `expect` syntax no longer supports the `==` matcher.

### Identity

```ruby
expect(actual).to be(expected)    # passes if actual.equal?(expected)
expect(actual).to equal(expected) # passes if actual.equal?(expected)
```

### Comparisons

```ruby
expect(actual).to be >  expected
expect(actual).to be >= expected
expect(actual).to be <= expected
expect(actual).to be <  expected
expect(actual).to be_within(delta).of(expected)
```

### Regular expressions

```ruby
expect(actual).to match(/expression/)
```

Note: The new `expect` syntax no longer supports the `=~` matcher.

### Types/classes

```ruby
expect(actual).to be_an_instance_of(expected) # passes if actual.class == expected
expect(actual).to be_a(expected)              # passes if actual.kind_of?(expected)
expect(actual).to be_an(expected)             # an alias for be_a
expect(actual).to be_a_kind_of(expected)      # another alias
```

### Truthiness

```ruby
expect(actual).to be_truthy   # passes if actual is truthy (not nil or false)
expect(actual).to be true     # passes if actual == true
expect(actual).to be_falsy    # passes if actual is falsy (nil or false)
expect(actual).to be false    # passes if actual == false
expect(actual).to be_nil      # passes if actual is nil
expect(actual).to_not be_nil  # passes if actual is not nil
```

### Expecting errors

```ruby
expect { ... }.to raise_error
expect { ... }.to raise_error(ErrorClass)
expect { ... }.to raise_error("message")
expect { ... }.to raise_error(ErrorClass, "message")
```

### Expecting throws

```ruby
expect { ... }.to throw_symbol
expect { ... }.to throw_symbol(:symbol)
expect { ... }.to throw_symbol(:symbol, 'value')
```

### Yielding

```ruby
expect { |b| 5.tap(&b) }.to yield_control # passes regardless of yielded args

expect { |b| yield_if_true(true, &b) }.to yield_with_no_args # passes only if no args are yielded

expect { |b| 5.tap(&b) }.to yield_with_args(5)
expect { |b| 5.tap(&b) }.to yield_with_args(Integer)
expect { |b| "a string".tap(&b) }.to yield_with_args(/str/)

expect { |b| [1, 2, 3].each(&b) }.to yield_successive_args(1, 2, 3)
expect { |b| { :a => 1, :b => 2 }.each(&b) }.to yield_successive_args([:a, 1], [:b, 2])
```

### Predicate matchers

```ruby
expect(actual).to be_xxx         # passes if actual.xxx?
expect(actual).to have_xxx(:arg) # passes if actual.has_xxx?(:arg)
```

### Ranges (Ruby >= 1.9 only)

```ruby
expect(1..10).to cover(3)
```

### Collection membership

```ruby
expect(actual).to include(expected)
expect(actual).to start_with(expected)
expect(actual).to end_with(expected)

expect(actual).to contain_exactly(individual, items)
# ...which is the same as:
expect(actual).to match_array(expected_array)
```

#### Examples

```ruby
expect([1, 2, 3]).to include(1)
expect([1, 2, 3]).to include(1, 2)
expect([1, 2, 3]).to start_with(1)
expect([1, 2, 3]).to start_with(1, 2)
expect([1, 2, 3]).to end_with(3)
expect([1, 2, 3]).to end_with(2, 3)
expect({:a => 'b'}).to include(:a => 'b')
expect("this string").to include("is str")
expect("this string").to start_with("this")
expect("this string").to end_with("ring")
expect([1, 2, 3]).to contain_exactly(2, 3, 1)
expect([1, 2, 3]).to match_array([3, 2, 1])
```

## `should` syntax

In addition to the `expect` syntax, rspec-expectations continues to support the
`should` syntax:

```ruby
actual.should eq expected
actual.should be > 3
[1, 2, 3].should_not include 4
```

See [detailed information on the `should` syntax and its usage.](https://github.com/rspec/rspec-expectations/blob/master/Should.md)

## Compound Matcher Expressions

You can also create compound matcher expressions using `and` or `or`:

``` ruby
expect(alphabet).to start_with("a").and end_with("z")
expect(stoplight.color).to eq("red").or eq("green").or eq("yellow")
```

## Composing Matchers

Many of the built-in matchers are designed to take matchers as
arguments, to allow you to flexibly specify only the essential
aspects of an object or data structure. In addition, all of the
built-in matchers have one or more aliases that provide better
phrasing for when they are used as arguments to another matcher.

### Examples

```ruby
expect { k += 1.05 }.to change { k }.by( a_value_within(0.1).of(1.0) )

expect { s = "barn" }.to change { s }
  .from( a_string_matching(/foo/) )
  .to( a_string_matching(/bar/) )

expect(["barn", 2.45]).to contain_exactly(
  a_value_within(0.1).of(2.5),
  a_string_starting_with("bar")
)

expect(["barn", "food", 2.45]).to end_with(
  a_string_matching("foo"),
  a_value > 2
)

expect(["barn", 2.45]).to include( a_string_starting_with("bar") )

expect(:a => "food", :b => "good").to include(:a => a_string_matching(/foo/))

hash = {
  :a => {
    :b => ["foo", 5],
    :c => { :d => 2.05 }
  }
}

expect(hash).to match(
  :a => {
    :b => a_collection_containing_exactly(
      a_string_starting_with("f"),
      an_instance_of(Integer)
    ),
    :c => { :d => (a_value < 3) }
  }
)

expect { |probe|
  [1, 2, 3].each(&probe)
}.to yield_successive_args( a_value < 2, 2, a_value > 2 )
```

## Usage outside rspec-core

You always need to load `rspec/expectations` even if you only want to use one part of the library:

```ruby
require 'rspec/expectations'
```

Then simply include `RSpec::Matchers` in any class:

```ruby
class MyClass
  include RSpec::Matchers

  def do_something(arg)
    expect(arg).to be > 0
    # do other stuff
  end
end
```

## Also see

* [https://github.com/rspec/rspec](https://github.com/rspec/rspec)
* [https://github.com/rspec/rspec-core](https://github.com/rspec/rspec-core)
* [https://github.com/rspec/rspec-mocks](https://github.com/rspec/rspec-mocks)
* [https://github.com/rspec/rspec-rails](https://github.com/rspec/rspec-rails)
