Style Guides
============

Guides for programming in good, consistent style

Git commits
-------------------

- Commit small changes often instead of doing big commits every now
  and then
- Do not mix different issues in single commit
- If you spot any minor style issues while working on a feature commit
  them separately
- Always review changes before commiting to avoid accidental mistakes,
  and/or to avoid random files beeing added to the repository
- Do not rush your commits, review file changes and commit messages
  carefully before submitting

Git commit messages
-------------------

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

http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html
https://github.com/thoughtbot/guides/blob/master/protocol/git/README.md

Method Objects
-------------------

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

Method Objects Specs
--------------------

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
        expect(service.all).to eq 2
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

RSpec
-------------------

Block implementation
If you are passing block RSpec will use it as implementation of the method. All arguments provided by caller will be yielded to your block. 
  
  Examples:
  - allow(sth).to receive(:foo) { do_something }
  - allow(sth).to receive(:foo).with("args") { do_something }
  - allow(sth).to receive(:foo).once { do_something }
  - allow(sth).to receive(:foo).order { do_something }

```ruby

describe Contact do
  it "is invalid without a firstname" do
    contact = Contact.create(first_name: nil, email: 'ted@example.com', password: 'empirestatebuilding')
    contact.valid?
    expect(contact.errors[:firstname]).to include("can't be blank")
  end

  it "is invalid with a duplicate email address" do
    Contact.create(first_name: 'Ted', email: 'ted@example.com', password: 'empirestatebuilding')
    contact = Contact.new(first_name: 'Robin', email: 'ted@example.com', password: 'montrealcanadiens')
    contact.valid?
    expect(contact.errors[:emal]).to include("has already been taken")
  end

  it "is invalid without a minimum password length" do
    contact_one = Contact.create(first_name: 'Ted', email: 'ted@example.com', password: 'empire')
    contact_one.valid?
    expect(Contact.errors[:password]).to include("password is too short")
  end
end
```

This feature supports many use cases and is very flexible. But if we don't need a comment or your tests are short and simple then it's better to use one-liner implementation.


One-Liner implementation
Is used for setting an expectation of the subject. One-liner implementation is faster in writing and still very readable. Using the example above if we decided to write tests for Contact with One-liner then it will looks like this

```ruby
describe Contact do
  it { is_expected.to validate_presence_of(:firstname) }
  it { is_expected.to validate_uniqueness_of(:email) }
  it { is_expected.to validate_length_of(:password).is_at_least(10) }
end
```

As we can see Block implementation needs much more steps and much more code to deal with the same job as One-liner. Of course not everything probably can be done without Block implementation but if it's only possible then it will be good to use One-liner implementation for writing tests and save codking time.
