Style Guides
============

Guides for programming in good, consistent style

## Table of contents

- [Git](#git)
  - [How to make commits](#how-to-make-commits)
  - [How to write commit messages](#how-to-write-commit-messages)
  - [Suggested GitHub flow](#github-flow)
- [RSpec](#rspec)
- [Sidekiq](#sidekiq)
- [Rails and Ruby tips](#rails-and-ruby-tips)
  - [Method Objects](#method-objects)
  - [Method Objects Specs](#method-objects-specs)
  - [How to update records](#how-to-update-records)
  - [How to work with strings](#how-to-work-with-strings)
- [Ruby Code Style Guide](#ruby-code-style-guide)
- [Goodreads](#goodreads)
  - [Articles](#articles)
  - [Videos](#videos)
  - [Books](#books)

## Git

### How to make commits

- Commit small changes often instead of doing big commits every now
  and then
- Do not mix different issues in single commit
- If you spot any minor style issues while working on a feature commit
  them separately
- Always review changes before commiting to avoid accidental mistakes,
  and/or to avoid random files beeing added to the repository
- Do not rush your commits, review file changes and commit messages
  carefully before submitting

### How to write commit messages

    Capitalized, short (50 chars or less) summary

    More detailed explanatory text, if necessary. Wrap it to about 72
    characters.

    - If there is a need to explain the changes further use bullet
      points with dashes
    - Wrap the points at 72 characters too
    - Separate paragraphs with new line
    - Do not use dots in the title or bullet points
    - Write your commit message in the imperative (Fix bug, not Fixed
      bug, Fixes the bug)
    - Valid title should nicely end this sentence: "If submitted this
      commit will: Fix bug"
    - Do not repeat commit messages if you forget to include something.
      Either squash two commits if the changes are not pushed or just
      explicitly describe the second (missing) change.
    - Always focus the message on the actual change, not the big
      picture. Avoid "Setup base for new interface", use "Add routes
      for ...".

    http://project.management-system.com/ticket/123

Details:

- http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html
- https://github.com/thoughtbot/guides/blob/master/protocol/git/README.md

### Suggested GitHub flow

- Begin from updating a master branch:

  ```bash
  $ git checkout master
  $ git pull origin master
  ```

- Create a new branch for your new feature:

  ```bash
  $ git checkout -b new_feature_branch # use snake_case for a branch name
  ```

- Commit your changes often to allow earlier code review. After commiting very first changes, push your branch to remote and create a [draft pull request](https://github.blog/2019-02-14-introducing-draft-pull-requests/):

  ```bash
  $ git push origin new_feature_branch
  ```

  ![Github draft Pull Request](https://i1.wp.com/user-images.githubusercontent.com/3477155/52671177-5d0e0100-2ee8-11e9-8645-bdd923b7d93b.gif?resize=1024%2C512&ssl=1)

- Add a link to a newly created PR to the very end of a Trello ticket description.
- When ready, change the status to 'Ready for review' near the bottom of your pull request to remove the draft state.

### Method Objects

Encapsulate application logic inside Method Objects. Do not relay on class
methods to provide functionality. Work on class instances instead. In order to
simply API and testing always provide a shortcut/proxy class method.

It sole purpose should be to instantiate the class with given arguments and
then call the main instance method. In general case the main method should be
called #call.

Always make internal methods private. Ideal Method Object implementation should
have only one public method.

Even if the logic is extremely simple and it takes only one line, do not
use class methods to provide implementation. Create snippets and marcos in your
development evironment to make the process of createing new classes more easy
and frictionless.

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
def self.call(*args)
  instance = new(*args)
  instance.call
  instance
end
```

### Method Objects Specs

Always create spec files for every method object. If for some reason you can't
provide spec for given method object at the time create pending spec file with
the corresponding path.

```ruby
RSpec.describe Divider do
  describe ".call" do
    subject { described_class }

    it { should forward_to_instance(:call).with_2_args }
  end

  describe "#call" do
    let(:service) { described_class.new(10, arg) }

    context "with non-zero argument" do
      let(:arg) { 5 }

      it "returns division result" do
        expect(service.call).to eq 2
      end
    end

    context "with zero argument" do
      let(:arg) { 0 }

      it "returns 0" do
        expect(service.call).to eq 0
      end
    end
  end
end
```

If the logic provides vastly different paths based on the input always create
corresponding branches using contexts (zero / non zero input in the Divider
example). Use common sense for amount of context. In general use border
values and some middle values when dealing with range arguments.

Always provide isloated envorinment for method object specs. If there
are calls to external objects inside a method object - stub the calls with
resonable values. Always check if the calls was actually made in expected
conditions and check if the expected value was passed to the call.

Preffer stubs instead of dependency injection. Due to nature of Ruby semantics
heavy dependency injection makes the code unreadable hard to reason about.
Verified stubs also provide basic consistency checks without resorting to
integration testing.

```ruby
# Iterate over provided collection of orders and return
# only the orders which are finished.

class Order::Finished::Selector
  def self.call(*args)
    new(*args).call
  end

  def initialize(orders)
    self.orders = orders
  end

  def call
    orders.select do |order|
      Order::Finished::Verifier.call(order)
    end
  end

  private

  attr_accessor :orders
end
```

Even though orders in this method object are collection of objects the logic
doesn't interact with the objects at all. They are only passed to external
class and selected based on the result. When creating specs for such cases
there is no need to create those objects or even use doubles. In many cases
pure symbols representing the abstract list element will do just fine. It
will make the specs simpler and more clear. You could just as well use
doubles instead, but if you are just passing the entity around prefer
symbols over anything else.

```ruby
require "rails_helper"

RSpec.describe Order::Finished::Selector do
  describe ".call" do
    subject { described_class }

    it { should forward_to_instance(:call).with_1_arg }
  end

  describe "#call" do
    let(:orders) { %i[order1 order2 order3] }

    before do
      allow(Order::Finished::Verifier)
        .to receive(:call).and_return(true, false, true)
    end

    it "returns only finished orders" do
      expect(service.call).to eq [:order1, :order3]
    end

    it "calls the verifier" do
      service.call

      expect(Order::Finished::Verifier)
        .to have_received(:call)
        .with(:order1)
        .with(:order2)
        .with(:order3)
    end
  end
end
```

## RSpec

Every newly created file should be created with a corresponding spec file.
When staging changes for commit make sure that a staged file has a corresponding
spec file (if applicable):

```shell
app/services/some/example/object.rb # file
spec/services/some/example/object_spec.rb # and its spec
```

In general, always write spec for your code. Preferably write specs before
writing the implementation. If for some reason you cannot spec the current
logic, then at least create pending spec file. Always include comment
with explanation why the spec is missing.

```ruby
RSpec.describe Some::Example::Object do
  describe "#call" do
    pending __FILE__

    # FIXME I need help with this spec. I do not know how to test this
    # implementation. It uses new framework and there is no examples
    # for this kind of logic in the codebase yet.
  end
end
```

When passing block to single spec in general use multiline `do ...
end` notation. Always use double quotes around spec title. Do not use
parenthesis after `it` method name. Avoid condition in spec names.
If you need to describe what state is needed for given behaviour,
just wrap the text in `context` block. Use this practice even if there
is no more branches for this example and no other specs. More often
than not, it makes sence to add another variant, where opposite
condition is described.

```ruby

describe Contact do
  context "without firstname" do
    it "is invalid" do
      ...
    end
  end
end
```

In case of very trivial and repetable specs we do not use titles
at all. In such cases we use one line notation using `it { ... }`
syntax. Even though in general `should` notaion is discouraged in the
newest versions of RSpec we still use it for such trivial examples. It
reads better and it is much shorter. In general cases we use one-line
notation for various types of Shoulda Matchers.

```ruby
describe Contact do
  it { should validate_presence_of(:firstname) }
  it { should validate_uniqueness_of(:email) }
  it { should validate_length_of(:password).is_at_least(10) }
end
```

If a spec one-liner exceeds 80 character revert back to a typical multi
line notation. Never omit titles in multiline notation. If the spec is
stil very trivial just use simple `#method` notation.

```ruby
describe "definitions" do
  it "#kind" do
    should define_enum_for(:kind).with(bad: 0, average: 1, good: 2, awesome: 3)
  end

  it "#historic_levels_build_by_day" do
    should delegate_method(:build_by_day).to(:historic_levels).with_prefix
  end

  it "#amount" do
    # When breaking a chain of methods do so for each dot '.'
    expect(model)
      .to have_db_column(:uid)
      .of_type(:string)
      .with_options(null: false)
  end
end
```

Always put one empty line between two `do end` blocks. Do not place
space between oneline blocks. Always group onelines first, then place
multiline block entries. This rule applies to all kind of definitions,
including examples `it`, `before` blocks or `let` definitions as well.

```ruby
describe "#call" do
    # No spaces between one line blocks
    let(:order) { instance_double(Order) }
    let(:instance) { described_class.new(order, attributes) }
    let(:uuid) { "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" }
    let(:order_uuid) { "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy" }
    let(:price) { "137.00" }
    # Space here
    let(:attributes) do
      {
        uuid: uuid,
        order_uuid: "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy",
        amount: "%.2f".format(price),
        headers: { "HTTP_VERSION" => "1" },
      }
    end
    # And here
    let(:params) do
      # ...
    end
    # And obviously here
    context "" do
      # ...
    end
end
```

## Sidekiq

Do not pass complex object as a parameters to methods which schedule the jobs.
Sidekiq params are serialized and stored in Redis. When the job has to be
done in context of some Record, always pass just the id (number or uuid),
and then load the Object inside worker code.

Do not put any logic inside the typical Sidekiq worker code. Always
encapsulate expected logic into some separate Method Object. Call that
inside `#perform` method. Most prefered way is actually to load expected
object then pass that object to the Method Object.

Write a simple spec for this behavior. Check if the expected object is
loaded using preferred method and if the Method Object is called using
expected method with provided parameteres. Test the core logic in the
Method Object spec outside the worker code.

```
class Agent::Blocker::Worker
  include Sidekiq::Worker
  include RedisMutex::Macro

  auto_mutex :perform, on: [:agent_id]

  def perform(agent_id)
    agent = Agent.find(agent_id)
    Agent::Blocker.call(agent)
  end
end
```

## Rails and Ruby tips

### How to update records

Whenever record is being created or updated use `.create!` or `#update!` to prevent silent failing.
Use `#update` or `.create` only with corresponding `if` check.

### How to work with strings

Do not use string concatenation and interpolation. Use `Kernel.format` instead. Most of the time
we use pythonic version of `#format` available via
[powerpack](https://www.rubydoc.info/gems/powerpack/String#format-instance_method).

```ruby
# Add this to the Gemfile to enable Powerpack `#format`
gem "powerpack", require: "powerpack/string/format"
```

Check [ruby documentation](https://ruby-doc.org/core-2.6.1/Kernel.html#method-i-format) and
[examples](https://www.rubyguides.com/2012/01/ruby-string-formatting/).

## Ruby Code Style Guide

Projects related to style guide and code formatting:

- https://github.com/testdouble/standard
- https://github.com/samphippen/rubyfmt
- https://github.com/rubocop-hq/rubocop
- https://github.com/uohzxela/clean-code-ruby

## Goodreads

This section contains links to usefull articles, books, videos, podcasts and other resources.

### Articles

- [The worst mistake of computer science - Paul Draper](https://www.lucidchart.com/techblog/2015/08/31/the-worst-mistake-of-computer-science/)
- [7 Patterns to Refactor Fat ActiveRecord Models - Bryan Helmkamp (2012)](https://codeclimate.com/blog/7-ways-to-decompose-fat-activerecord-models/)

### Videos

- [Nothing is Something - Sandi Metz (2015)](https://www.youtube.com/watch?v=29MAL8pJImQ)
- [Boundaries - Gary Bernhardt (2012)](https://www.destroyallsoftware.com/talks/boundaries)
- [Y Not - Jim Weirch (2012)](https://www.youtube.com/watch?v=FITJMJjASUs)

### Books

- [Practical Object-Oriented Design (POODR) - Sandi Metz](https://www.poodr.com/)
- [Rails 5 Test Prescriptions - Noel Rappin](https://pragprog.com/book/nrtest3/rails-5-test-prescriptions)
- [Everyday Rails Testing with RSpec - Aaron Sumner](https://leanpub.com/everydayrailsrspec)
- [Exceptional Ruby - Avdi Grimm](http://exceptionalruby.com/)
