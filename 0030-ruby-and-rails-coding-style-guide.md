Style Guides
============

Guides for programming in good, consistent style

## Table of contents

- [Rails and Ruby coding guide](#rails-and-ruby-coding-guide)
  - [Method Objects](#method-objects)
  - [Method Objects Specs](#method-objects-specs)
  - [Custom controller actions](#do-not-add-custom-controller-actions)
  - [Desgin patterns in controllers](#design-patterns-in-controllers)
  - [How to update records](#how-to-update-records)
  - [How to work with strings](#how-to-work-with-strings)
  - [How to work with migrations](#how-to-work-with-migrations)
- [Ruby Code Style Guide](#ruby-code-style-guide)

## Rails and Ruby coding guide

### Service Objects

Encapsulate application logic inside Service Objects. Do not relay on class
methods to provide functionality. Work on class instances instead. In order to
simplify API and testing always provide a shortcut/proxy class method.

It sole purpose should be to instantiate the class with given arguments and
then call the main instance method. In general case the main method should be
called #call.

Always make internal methods private. Ideal Service Object implementation should
have only one public method.

Even if the logic is extremely simple and it takes only one line, do not
use class methods to provide implementation. Create snippets and macros in your
development environment to make the process of creating new classes more easy
and friction-less.

```ruby
# Divide two numbers or return zero if the second argument is
# equal to zero.

class Divider
  def self.call(*args)
    new(*args).call
  end

  def initialize(arg1, arg2)
    self.arg1 = arg1
    self.arg2 = arg2
  end

  def call
    return 0 if arg2.zero?

    arg1 / arg2
  end

  private

  attr_accessor :arg1, :arg2
end
```

If instead of return value you need to work on class instance just modify the
proxy class method to execute the logic and then return an instance instead.
This approach shouldn't be used often outside of the facade / action objects
used in controllers.

```ruby
def self.call(...)
  new(...).tap(&:call)

  # This is equivalent to:
  #
  # instance = new(...)
  # instance.call
  # instance
end
```

### Service Objects Specs

Always create spec files for every Service Object. If for some reason you can't
provide spec for given Service Object at the time create pending spec file with
the corresponding path.

```ruby
RSpec.describe Divider do
  describe ".#call" do
    context "with non-zero argument" do
      it "returns division result" do
        expect(described_class.call(10, 5)).to eq 2
      end
    end

    context "with zero argument" do
      it "returns 0" do
        expect(described_class.call(10, 0)).to eq 0
      end
    end
  end
end
```

If the logic provides vastly different paths based on the input always create
corresponding branches using contexts (zero / non zero input in the Divider
example). Use common sense for amount of context. In general use border
values and some middle values when dealing with range arguments.

Always provide isolated environment for Service Object specs. If there
are calls to external objects inside a Service Object - stub the calls with
reasonable values. Always check if the calls was actually made in expected
conditions and check if the expected value was passed to the call.

Prefer stubs instead of dependency injection. Due to nature of Ruby semantics
heavy dependency injection makes the code unreadable hard to reason about.
Verified stubs also provide basic consistency checks without resorting to
integration testing.

```ruby
# Iterate over provided collection of orders and return
# only the orders which are finished.

class Selector
  def self.call(...)
    new(...).call
  end

  def initialize(orders)
    self.orders = orders
  end

  def call
    orders.select do |order|
      Verifier.call(order)
    end
  end

  private

  attr_accessor :orders
end
```

Even though orders in this Service Object are collection of objects the logic
doesn't interact with the objects at all. They are only passed to external
class and selected based on the result. When creating specs for such cases
there is no need to create those objects or even use doubles. In many cases
pure symbols representing the abstract list element will do just fine. It
will make the specs simpler and more clear. You could just as well use
doubles instead, but if you are just passing the entity around prefer
symbols over anything else.

```ruby
require "rails_helper"

RSpec.describe Selector do
  describe ".call" do
    it "returns only finished orders" do
      orders = %i[order1 order2 order3]

      allow(Verifier)
        .to receive(:call)
        .and_return(true, false, true)

      expect(described_class.call)
        .to eq [:order1, :order3]
    end

    it "calls the verifier" do
      orders = %i[order1 order2 order3]

      allow(Verifier)
        .to receive(:call)
        .and_return(true, false, true)

      described_class.call

      expect(Verifier)
        .to have_received(:call)
        .with(:order1)
        .with(:order2)
        .with(:order3)
    end
  end
end
```

### Custom controller actions

Do not add custom (non-rest) controller actions. Say you have a
`OrdersController` and want to add a custom action to duplicate the Order:

```ruby
module Admin
  class OrdersController < BaseController
    before_action :authenticate_user!

    def new
      # ...
    end

    def create
      # ...
    end

    def clone
      # Logic to generate new order based on existing one
    end
  end
end
```

Instead of adding a custom method to an existing controller, create a new
controller with a REST action, that would correspond to `clone` action.

```ruby
module Admin
  module Orders
    class ClonesController < BaseController
      def show
        # Logic to generate new order based on existing one
      end
    end
  end
end
```

Note: Strongly consider moving the original controller and all related objects
into the new namespace as well.

```ruby
module Admin
  module Orders
    class OrdersController < BaseController
      def show
        # Logic to generate new order based on existing one
      end
    end
  end
end
```

Name duplication is fine in this case. If there is a strong desire to keep
more controllers in this namespace and have very good naming convetions
you can consider using `EntriesController` or `RecordsController` instead of
`OrdersController`, but this is not really necessary.

This would allow to use resourceful routes instead of defining a custom route:

```ruby
namespace :admin do
  namespace :orders do
    resources :orders, only: [:new, :show, :index, :create]
    resources :clones, only: [:show]
  end
end
```

Details:

- http://jeromedalbert.com/how-dhh-organizes-his-rails-controllers/

### Design patterns in controllers

Use `Facade` design pattern in all actions to prepare data you want to display in view.

```ruby
# Bad
module Admin
  class OffersController < BaseController
    def index
      @search = Offer.ransack(params[:q])
      @markets = UserLanguage.all
    end
  end
end

# Good
module Admin
  class OffersController < BaseController
    def index
      @facade = IndexFacade.new(params)
      # @facade.search
      # @facade.markets
    end
  end
end
```

If you need to execute a logic when preparing a `@facade` object, simply use the `.call` method instead.
Make sure to sill return the object itself.

```ruby
class OffersController
  class IndexFacade
    def self.call(...)
      new(...).tap(&:call)

      # This is equivalent to doing:
      #
      # facade = new(...)
      # facade.call
      # facade
    end
  end
end
```

Use `forms` in cases where you want to create or update an object (in `new`, `create` and `update` controller methods).

```ruby
module Admin
  class Offer
    class CreateForm
      include ActiveModel::Model

      # put here some validation, record saving etc.
    end
  end
end
```

### How to update records

Whenever record is being created or updated use `.create!` or `#update!` to
prevent silent failing. Use `#update` or `.create` only with corresponding `if`
check.

### How to work with hashes

Fetch value using `fetch` method. In case if key is missing `KeyError` will be raised. It will be much
easier to debug in opposite to fetching value by `[key]` and save our time for future debugging.
If you accept `nil` result use `fetch(key, nil)` instead to signal the intent.

### How to work with strings

Use string interpolation with `#{...}` only when there is no complex logic in
the method call within interpolation. Usually, when there is 1-2 chained calls,
interpolation is fine. If there is more calls, or method argument, use the
`format(...)` method instead. Also prefer this method for longer sting, which
needs to be broken into multiple lines. Do not concatenate strings over multiple
lines, prefer `heredocs` instead.

```ruby
# Bad
msg = "User #{user.name.truncate(15, ommision: '.')} has been created."

# Bad
msg = "User account is not active. Please check your email." +
 "If you didn't receive the email, please contact support."

# Good
msg = "User #{user.name} with email #{user.email.downcase} has been created."

# Good
def command
  format(
    <<~TXT.squish,
      ssh -o "StrictHostKeyChecking=no" -l account%<seat>s -T
      192.168.186.%<seat>s 'xwd -root -display :0|convert xwd:- png:-'
    TXT
    seat: user.seat,
  )
end
```

Details:

- https://ruby-doc.org/core-2.6.1/Kernel.html#method-i-format
- https://batsov.com/articles/2013/06/27/the-elements-of-style-in-ruby-number-2-favor-sprintf-format-over-string-number-percent/
- https://www.rubyguides.com/2012/01/ruby-string-formatting/

### How to work with migrations

One of the biggest pains when using `structure.sql` is ensuring that only the
required changes get committed to that file. When you pull someone’s branch
and run the migrations specific to that branch, your `structure.sql` will now
contain some changes. Say, you then go back to working on your own branch and
generate a new migration. Your `structure.sql` file will now contain both your
branch’s and the other branch’s changes. This only gets worse with growing
number of migrations from different not-yet-or-never-to-be merged branches.

Here is the strategy to ensure that `structure.sql` file only contains the
necessary changes to a specific branch. Once you are done working on a branch
that contains migrations, make sure you run rails `db:rollback STEP=n`, where
`n` is the number of migrations in that branch. This will ensure your database
structure reverts to its original state.

Note: Please make sure to always perform your own migration in the
`pg-local-dev` container. This is a clean database, which only contains the
schema. You should be able to generate a clean diff using this database. In
case of any issues, you can always pull the schema file from mainline branch
and run the migrations again. Please be extra careful when dropping/creating
databases locally. Make sure you are in the right container. If you are not
sure, ask for help. Dropping database from regular `dev` container will drop
the shared development database and disturb the work of other developers.

Details:

- https://blog.appsignal.com/2020/01/15/the-pros-and-cons-of-using-structure-sql-in-your-ruby-on-rails-application.html

## Ruby Code Style Guide

Projects related to style guide and code formatting:

- https://github.com/testdouble/standard
- https://github.com/samphippen/rubyfmt
- https://github.com/rubocop-hq/rubocop
- https://github.com/uohzxela/clean-code-ruby
- https://github.com/airbnb/ruby
- https://github.com/rubocop-hq/ruby-style-guide

## Our Guides for Ruby Code

See `rubocop.yml` on a specific project
